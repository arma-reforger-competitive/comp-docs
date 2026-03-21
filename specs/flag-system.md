# Flag System Spec

Technical specification for the flag capture and objective system.

Audience: Developers implementing `arma-comp-core/src/AAS/`.

---

## Overview

The flag system manages capturable objective points in AAS mode. Flags follow linear progression — teams can only interact with the next flag in the chain. Each flag is a state machine driven by player presence in a capture zone.

---

## Data Model

### Flag

```
Flag {
  id:            int           // Unique, matches layer order (0-indexed)
  name:          string        // Display name from layer config
  position:      vector3       // World coordinates (center of capture zone)
  radius:        float         // Capture zone radius in meters
  state:         FlagState     // Current state (see State Machine)
  owner:         TeamID | null // Team that owns this flag (null if Neutral)
  captureProgress: float       // 0.0 to 1.0 (0 = no progress, 1 = captured)
  capturingTeam: TeamID | null // Team currently making capture progress
}
```

### FlagState

```
enum FlagState {
  Locked,      // Not active in progression
  Neutral,     // Active, no owner
  Capturing,   // Active, capture in progress
  Secured      // Active, owned by a team
}
```

### TeamID

```
enum TeamID {
  Team1,
  Team2
}
```

---

## State Machine

```
                    Match Start
                        |
                        v
    [Locked] -----(progression activates)-----> [Neutral]
                                                   |
                                        (team enters zone)
                                                   |
                                                   v
                                              [Capturing]
                                              /         \
                              (progress = 1.0)           (all attackers leave,
                                    |                     decay to 0.0)
                                    v                         |
                               [Secured]                      v
                                    |                    [Neutral]
                          (enemy begins capture)
                                    |
                                    v
                               [Capturing]
                              /         \
                (progress = 1.0)         (all attackers leave,
                      |                   decay to 0.0)
                      v                       |
                 [Secured]                    v
                (new owner)            [Secured]
                                    (original owner)
```

### Transition Rules

| From | To | Trigger |
|------|----|---------|
| Locked → Neutral | Adjacent flag in the chain is captured, making this flag the new front line |
| Neutral → Capturing | At least one player from a team is inside the capture zone with no enemies present |
| Capturing → Secured | `captureProgress` reaches 1.0 |
| Capturing → Neutral | All capturing players leave the zone and `captureProgress` decays to 0.0 |
| Capturing → Capturing (paused) | Both teams have players in the zone (Contested). Progress is frozen, not reset. |
| Secured → Capturing | Enemy team has players in the zone with no defenders present. `captureProgress` resets to 0.0 and begins for the attacking team. |

### Contested State

Contested is not a distinct `FlagState` — it is a condition evaluated during the `Capturing` state:

```
isContested = (team1PlayersInZone > 0) AND (team2PlayersInZone > 0)
```

When contested:
- `captureProgress` is **frozen** (no increase, no decay)
- Spawning on this flag is disabled for both teams
- HUD shows contested indicator

When no longer contested (one side cleared):
- If the capturing team remains, progress resumes
- If only the defending team remains, progress begins decaying

---

## Capture Mechanics

### Player Counting

Count players inside the capture zone per team every server tick:

```
team1PlayersInZone = count of Team1 alive infantry within flag.radius of flag.position
team2PlayersInZone = count of Team2 alive infantry within flag.radius of flag.position
```

**Vehicle rules for counting:**
- Players inside Transport or Light Armor vehicles count as infantry for presence (they are "in the zone")
- Heavy Armor vehicle occupants **do not count** toward capture presence
- The vehicle hull itself never adds capture weight — only the player count matters

### Capture Speed

Base capture time is 90 seconds (progress 0.0 → 1.0). Scaling by player count:

| Players in Zone | Capture Time | Progress per Second |
|----------------|-------------|-------------------|
| 1 | 90s | 0.0111 |
| 2 | 60s | 0.0167 |
| 3 | 45s | 0.0222 |
| 4+ | 35s | 0.0286 |

Formula:

```
captureRate(n) = where n = number of capturing team players in zone
  if n == 0: 0
  if n == 1: 1/90
  if n == 2: 1/60
  if n == 3: 1/45
  if n >= 4: 1/35
```

Progress is capped at 4 players — additional players beyond 4 do not speed up capture. This prevents "zerg rush" strategies from being disproportionately rewarded.

### Capture Decay

When all attackers leave the zone and the flag is in `Capturing` state with no defenders:

```
decayRate = 1/45  // 45 seconds from any progress back to 0.0
```

If defenders (the owning team) are present, decay rate doubles:

```
decayRateWithDefenders = 1/22.5
```

When `captureProgress` reaches 0.0 during decay:
- If the flag had a previous owner: state returns to `Secured` (original owner)
- If the flag was Neutral: state returns to `Neutral`

---

## Linear Progression

### Initial Setup

Flags are ordered by `id` (from layer config). At match start:

1. Flags owned by Team1 (from their side of the map) are `Secured` with `owner = Team1`
2. Flags owned by Team2 (from their side) are `Secured` with `owner = Team2`
3. The center flag (if neutral in layer config) is `Neutral`
4. Only the **active front** flags are unlocked; all others are `Locked`

### Active Front Calculation

```
getActiveFront():
  for each flag in order:
    if flag.owner == Team1 and nextFlag.owner != Team1:
      nextFlag is Team1's attack target (must be Neutral, Capturing, or Secured by Team2)
    if flag.owner == Team2 and prevFlag.owner != Team2:
      prevFlag is Team2's attack target

  Active flags = all Secured flags + attack targets for both teams
  Everything else = Locked
```

After a flag changes ownership, recalculate the active front immediately. Newly active flags transition from `Locked` to `Neutral`.

### Edge Case: Final Flag

When a team captures the enemy's last flag:
- Match ends immediately (win condition)
- No further state transitions occur

---

## Tick Processing

Every server tick (order matters):

```
1. Count players in each flag's capture zone
2. For each non-Locked flag:
   a. If Contested → freeze progress, skip
   b. If capturing team has players → advance captureProgress by captureRate * deltaTime
   c. If no capturing team players → decay captureProgress by decayRate * deltaTime
   d. If captureProgress >= 1.0:
      - Set state = Secured, owner = capturingTeam
      - Fire EVENT_FLAG_CAPTURED(flag, newOwner, previousOwner)
      - Recalculate active front (may unlock/lock other flags)
   e. If captureProgress <= 0.0 during decay:
      - Reset state to Secured (original owner) or Neutral
      - Clear capturingTeam
3. Update HUD for all players
```

---

## Events Emitted

| Event | Payload | Consumed By |
|-------|---------|-------------|
| `EVENT_FLAG_CAPTURED` | flagId, newOwner, previousOwner | Ticket system (-20 to loser, +30 to winner), match flow (win condition check), spawn system (update valid spawns) |
| `EVENT_FLAG_CONTESTED` | flagId | Spawn system (disable spawns on this flag), HUD |
| `EVENT_FLAG_UNCONTESTED` | flagId, remainingTeam | Spawn system (re-enable if applicable), HUD |
| `EVENT_FLAG_PROGRESS` | flagId, progress, capturingTeam | HUD (capture bar update) |
| `EVENT_FLAG_UNLOCKED` | flagId | HUD (new objective), spawn system |
