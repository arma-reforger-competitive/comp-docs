# Match Flow Spec

Technical specification for the match state machine, phase transitions, and win condition evaluation.

Audience: Developers implementing `arma-comp-core/src/Match/`.

---

## Overview

The match flow system is the top-level state machine that controls the lifecycle of a competitive match. It orchestrates all other systems (flags, tickets, spawns) by enabling/disabling them based on the current match phase.

---

## Data Model

```
MatchState {
  phase:          MatchPhase
  layer:          LayerConfig     // Loaded from layer file
  timer:          float           // Countdown timer for current phase (seconds remaining)
  pauseState:     PauseState | null
  result:         MatchResult | null
  pausesUsed:     map<TeamID, int>  // Pauses consumed per team
}

enum MatchPhase {
  Loading,
  Warmup,
  Live,
  Overtime,
  Ended
}

PauseState {
  requestedBy:  TeamID
  timeRemaining: float   // Max 600s (10 min)
  pausedDuring: MatchPhase  // Phase that was paused (to resume correctly)
}

MatchResult {
  winner:       TeamID | null  // null = draw
  reason:       WinReason
  team1Tickets: int
  team2Tickets: int
  duration:     float          // Total match time (excluding pauses)
  flagState:    list<FlagSnapshot>
}

enum WinReason {
  TicketsZero,
  FinalFlagCaptured,
  TimeExpired,
  OvertimeExpired,
  Forfeit,
  Draw
}
```

---

## State Machine

```
[Loading] ---(layer loaded, players connected)---> [Warmup]
                                                      |
                                              (warmup timer = 0)
                                                      |
                                                      v
                                                   [Live]
                                                  /   |   \
                               (tickets = 0,     /    |    \  (tickets = 0 or
                                final flag       /     |     \  final flag or
                                contested)      /      |      \ time expired,
                                               /       |       \ NOT contested)
                                              v        |        v
                                        [Overtime]     |     [Ended]
                                            |          |
                                   (flag resolved      |
                                    or OT timer = 0)   |
                                            |          |
                                            v          |
                                         [Ended] <-----/
```

---

## Phase Details

### Loading

**Purpose:** Server initializes the match from a layer configuration.

**Actions:**
1. Parse and validate layer file (see `systems/layers.md` for validation rules)
2. Initialize flag system with flag positions, radii, and initial ownership
3. Initialize ticket pools with starting tickets
4. Initialize vehicle pools
5. Set spawn points (Main Base per team)
6. Wait for all players to connect (or timeout after configurable wait, default 120s)

**Transitions:**
- → `Warmup`: All players connected OR connection timeout reached

**System states during Loading:**
- Flags: Initialized but no captures
- Tickets: Set but no events processed
- Spawns: Disabled
- Vehicles: Not spawned
- Player input: Disabled

### Warmup

**Purpose:** Players organize squads, select roles, prepare.

**Duration:** 60 seconds (configurable per layer)

**Actions:**
1. All players spawn at Main Base
2. Start warmup countdown timer
3. Display warmup HUD overlay

**Player permissions:**
- CAN: Select roles, form/join squads, communicate, view map, look around, move within Main Base protection zone
- CANNOT: Leave Main Base protection zone, fire weapons, deploy spawn points, enter vehicles, deal/take damage

**Transitions:**
- → `Live`: Warmup timer reaches 0

**System states during Warmup:**
- Flags: Visible on HUD, states set, no captures possible
- Tickets: Set, no events processed
- Spawns: Disabled (players are alive at Main Base)
- Vehicles: Spawned at Main Base but not enterable
- Player input: Movement only within Main Base zone

**On transition to Live:**
1. Remove Main Base movement restriction
2. Enable all player input (weapons, vehicles, deployables)
3. Enable flag capture processing
4. Enable ticket event processing
5. Enable spawn system
6. Start match timer (default 4500s / 75 min)
7. Broadcast "MATCH LIVE" to all players

### Live

**Purpose:** Active competitive play.

**Duration:** Up to 75 minutes (configurable per layer)

**All systems active:**
- Flag capture processing: ON
- Ticket events: ON
- Bleed timer: ON
- Spawn waves: ON
- Vehicle spawns/respawns: ON

**Transitions:**
- → `Overtime`: `EVENT_TICKETS_ZERO` received AND final flag is Contested or Capturing
- → `Ended`: `EVENT_TICKETS_ZERO` received AND final flag is NOT Contested/Capturing
- → `Ended`: `EVENT_FLAG_CAPTURED` where captured flag is the enemy's last flag
- → `Ended`: Match timer reaches 0

**Win condition evaluation on `EVENT_TICKETS_ZERO(teamId)`:**

```
onTicketsZero(teamId):
  enemyTeam = getOpponent(teamId)
  finalFlag = getLastFlag(teamId)  // The flag closest to teamId's Main Base

  if finalFlag.state == Capturing and finalFlag.capturingTeam == enemyTeam:
    transitionTo(Overtime)
  else if finalFlag.state == Contested:
    transitionTo(Overtime)
  else:
    endMatch(winner = enemyTeam, reason = TicketsZero)
```

**Win condition evaluation on `EVENT_FLAG_CAPTURED`:**

```
onFlagCaptured(flagId, newOwner, previousOwner):
  if previousOwner != null:
    remainingFlags = count of flags owned by previousOwner
    if remainingFlags == 0:
      endMatch(winner = newOwner, reason = FinalFlagCaptured)
```

**Win condition on timer expiry:**

```
onTimerExpired():
  team1Tickets = getTickets(Team1)
  team2Tickets = getTickets(Team2)

  if team1Tickets > team2Tickets:
    endMatch(winner = Team1, reason = TimeExpired)
  else if team2Tickets > team1Tickets:
    endMatch(winner = Team2, reason = TimeExpired)
  else:
    // Equal tickets at time expiry
    team1Flags = countFlags(Team1)
    team2Flags = countFlags(Team2)
    if team1Flags > team2Flags:
      endMatch(winner = Team1, reason = TimeExpired)
    else if team2Flags > team1Flags:
      endMatch(winner = Team2, reason = TimeExpired)
    else:
      endMatch(winner = null, reason = Draw)
```

### Overtime

**Purpose:** Resolve a match where tickets reached 0 while the final flag was being contested.

**Duration:** Maximum 5 minutes (configurable per layer)

**On enter:**
1. Start overtime timer (default 300s)
2. Suspend bleed timer (no bleed ticks during overtime)
3. Suspend all ticket events EXCEPT flag capture events
4. Broadcast "OVERTIME" to all players
5. Spawning remains active

**Active systems:**
- Flag capture processing: ON
- Ticket events: OFF (except flag capture: -20 / +30)
- Bleed timer: OFF
- Spawn waves: ON
- Vehicle spawns/respawns: ON

**Transitions:**
- → `Ended`: Flag is captured by either team (resolves the contest)
- → `Ended`: Overtime timer reaches 0

**Win condition on flag captured during overtime:**

```
onFlagCapturedDuringOvertime(flagId, newOwner, previousOwner):
  endMatch(winner = newOwner, reason = FinalFlagCaptured)
```

**Win condition on overtime timer expiry:**

```
onOvertimeExpired():
  team1Flags = countFlags(Team1)
  team2Flags = countFlags(Team2)

  if team1Flags > team2Flags:
    endMatch(winner = Team1, reason = OvertimeExpired)
  else if team2Flags > team1Flags:
    endMatch(winner = Team2, reason = OvertimeExpired)
  else:
    team1Tickets = getTickets(Team1)
    team2Tickets = getTickets(Team2)
    if team1Tickets > team2Tickets:
      endMatch(winner = Team1, reason = OvertimeExpired)
    else if team2Tickets > team1Tickets:
      endMatch(winner = Team2, reason = OvertimeExpired)
    else:
      endMatch(winner = null, reason = Draw)
```

### Ended

**Purpose:** Match is over. Display results, prevent further play.

**On enter:**
1. Disable all player input (weapons, movement, vehicles, deployables)
2. Stop all timers (match, bleed, spawn wave)
3. Stop flag capture processing
4. Build and store `MatchResult`
5. Broadcast result to all players
6. Write match result to server log
7. Display scoreboard for 30 seconds
8. After scoreboard, server returns to lobby or loads next match

**System states during Ended:**
- All gameplay systems: OFF
- Player input: Disabled (view-only)
- HUD: Shows final result overlay

---

## Pause System

A pause can be requested during `Live` or `Overtime` phases only.

```
requestPause(teamId, refereeId):
  if matchState.phase not in [Live, Overtime]:
    reject("Cannot pause outside of Live or Overtime")
  if matchState.pauseState != null:
    reject("Already paused")
  if matchState.pausesUsed[teamId] >= 3:
    reject("Team has used all 3 pauses")

  matchState.pauseState = PauseState {
    requestedBy: teamId,
    timeRemaining: 600,  // 10 minutes max
    pausedDuring: matchState.phase
  }
  matchState.pausesUsed[teamId] += 1

  // Freeze everything
  freezeAllTimers()      // Match timer, bleed timer, spawn waves, capture progress
  disablePlayerActions() // Cannot fire, move, interact. Can look around.
  broadcast("MATCH PAUSED by Referee")
```

```
requestUnpause(refereeId):
  if matchState.pauseState == null:
    reject("Not paused")

  broadcast("MATCH RESUMING IN 5 SECONDS")
  startCountdown(5 seconds)
  // After countdown:
  matchState.pauseState = null
  unfreezeAllTimers()
  enablePlayerActions()
```

**Pause timer:** If `pauseState.timeRemaining` reaches 0, the match auto-unpauses with the 5-second countdown.

---

## Simultaneous Events

When both teams' tickets reach 0 in the same server tick:

```
onBothTeamsTicketsZero():
  team1Flags = countFlags(Team1)
  team2Flags = countFlags(Team2)

  if team1Flags > team2Flags:
    endMatch(winner = Team1, reason = TicketsZero)
  else if team2Flags > team1Flags:
    endMatch(winner = Team2, reason = TicketsZero)
  else:
    endMatch(winner = null, reason = Draw)
```

This should be extremely rare due to the ticket processing priority order (see `specs/ticket-system.md`), but the system must handle it.

---

## Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `EVENT_TICKETS_ZERO` | Ticket system | Evaluate win condition or transition to Overtime |
| `EVENT_FLAG_CAPTURED` | Flag system | Check for final flag win condition |
| Pause request | Admin/Referee command | Enter pause state |

## Events Emitted

| Event | Payload | Consumed By |
|-------|---------|-------------|
| `EVENT_MATCH_PHASE_CHANGED` | newPhase, previousPhase | All systems (enable/disable), HUD |
| `EVENT_MATCH_ENDED` | MatchResult | Server log, HUD, scoreboard |
| `EVENT_MATCH_PAUSED` | pauseState | HUD, all timers |
| `EVENT_MATCH_UNPAUSED` | resumePhase | HUD, all timers |
