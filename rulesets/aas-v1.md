# ARC Competitive AAS v1 Ruleset

**Mode:** Advance and Secure (AAS)
Infantry and vehicles, ticket-based, linear objectives

---

## 1. Purpose & Scope

This document defines the **official rules** for the competitive AAS mode in Arma Reforger.

It establishes:

- Match structure and flow
- Objective and ticket mechanics
- Spawn and vehicle rules
- Win conditions
- Restrictions for fair, balanced competitive play

This ruleset is suitable for **tournament play**.

For term definitions, see `terminology/glossary.md`.
For role and vehicle class details, see `systems/roles-and-classes.md`.
For squad rules, see `systems/squads.md`.
For layer configuration, see `systems/layers.md`.

---

## 2. Match Overview

- **Teams:** 2 teams of **24 players** (adjustable per layer; teams must be equal size)
- **Squads:** Max 6 players per squad (see `systems/squads.md`)
- **Objectives:** 5-7 linear capture points (flags)
- **Tickets:** 400 per team (adjustable per layer)
- **Match duration:** 75 minutes maximum (adjustable per layer)
- **Focus:** Infantry and vehicle combat, tactical coordination

---

## 3. Flag / Objective Rules

- Flags follow **linear progression** (cannot be skipped)
- States: **Locked -> Neutral -> Capturing -> Secured**
- Capture speed scales with players in the zone (each additional player reduces capture time, minimum cap at 4 players)
- Base capture time: **90 seconds** (1 player in zone)
- Capture scaling: 2 players = 60s, 3 players = 45s, 4+ players = 35s
- Flags can be **occupied by infantry or vehicles**, but Heavy Armor **does not accelerate capture**
- A flag becomes **Contested** when both teams have players in the capture zone — capture progress is **paused** until one side is cleared
- If a capturing team is fully cleared from the zone, capture progress **decays** back to neutral over 45 seconds

---

## 4. Ticket System

| Event | Ticket Change |
|---|---|
| Player death | -1 |
| Transport destroyed | -5 |
| Light Armor destroyed | -10 |
| Heavy Armor destroyed | -15 |
| Flag lost | -20 (losing team) |
| Flag defended | +30 (holding team) |
| Ticket bleed | -1 every 30s |
| Final flag bleed | -3 every 30s |

Vehicle ticket costs correspond to vehicle classes defined in `systems/roles-and-classes.md`.

### Bleed Rules

- Bleed starts when a team controls **fewer flags than the enemy**
- Bleed rate: **-1 ticket per 30 seconds**
- **Final flag bleed:** When a team controls only their last flag, bleed increases to **-3 per 30 seconds**
- Bleed is evaluated every 30 seconds on a server tick — not continuously

### Flag Defended

- **+30 tickets** are awarded when a team successfully defends a flag that was being captured by the enemy (enemy capture progress resets to 0 while the defending team holds the flag)

---

## 5. Vehicle Rules

- Vehicles are **limited per team** by class (configurable per layer)
- Default limits: Transport x4, Light Armor x2, Heavy Armor x1
- **No vehicle stacking:** Maximum 2 vehicles per team within a single capture zone
- Vehicle respawn timer starts on destruction (Transport: 3 min, Light Armor: 5 min, Heavy Armor: 8 min)
- Destroyed vehicles respawn at **Main Base only**
- Vehicle classes and ticket costs are defined in `systems/roles-and-classes.md`

---

## 6. Spawn Rules

Respawns are strictly limited to **designated team-controlled locations** to maintain competitive fairness.

### Allowed Spawn Points

1. **Main Base** - Fixed map location; all team members may respawn here. Indestructible.
2. **Small Radio (Forward Outpost)** - Deployable by a squad's **Radio Operator**; only that squad may respawn here. One per squad. Destructible.
3. **MSP (Mobile Spawn Point)** - Truck-mounted mobile spawn; all team members may respawn while it is deployed. Destructible.
4. **HUB** - Stationary spawn point deployed by the **Construction Truck**; all team members may respawn while it remains intact. Destructible.

### Spawn Restrictions

- Players **cannot respawn** at:
  - Contested flags (both teams present in capture zone)
  - Locked flags (not yet active in linear progression)
  - Destroyed or disabled spawn points

- **Small Radio:**
  - Only active while deployed and powered by the Radio Operator
  - Only the deploying squad may use it
  - Destroyed if the Radio Operator dies without another squad member within 50m
  - One per squad; deploying a new one destroys the previous

- **MSP:**
  - Mobile and can be relocated by driving
  - All team members may use it while deployed
  - Must be stationary and deployed (not moving) to allow spawns
  - Disabled if an enemy is within 30m

- **HUB:**
  - Stationary, must be deployed at a suitable location
  - All team members may use it while intact
  - Disabled if an enemy is within 30m

### Spawn Timing

- Spawns use a **wave system**: dead players spawn in groups every **30 seconds** (configurable per layer)
- Players who die between waves wait for the next wave
- Spawn wave timer is global per team, not per player

### Anti-Spawn Camping

- **Main Base protection zone:** A 150m radius around each Main Base. Enemy players **cannot enter** the Main Base protection zone. Players who enter are warned for 5 seconds, then killed by the server.
- **Spawn point proximity disable:** Small Radio, MSP, and HUB are **disabled** when an enemy player is within **30m**. The spawn point remains visible but cannot be used until cleared.
- **Spawn protection:** Players have **3 seconds of spawn protection** after respawning (cannot take damage, cannot deal damage). Protection ends immediately if the player fires, aims down sights, or enters a vehicle.

---

## 7. Match Flow

### Phase 1: Warm-up (60 seconds)

- All players spawn at Main Base
- Players **may:** select roles, form squads, communicate, view the map
- Players **may not:** leave the Main Base protection zone, fire weapons, deploy spawn points, enter vehicles
- Warm-up timer is displayed on HUD
- When warm-up ends, all systems activate simultaneously

### Phase 2: Live Match (up to 75 minutes)

- All systems active: captures, tickets, bleed, spawns, vehicles
- Match timer counts down from 75:00
- Live match continues until a win condition is met or the timer expires

### Phase 3: Overtime

Overtime is triggered when **all** of the following are true:
- At least one team's tickets reach 0
- The final active flag is **Contested** or **Capturing** (not Secured)

Overtime rules:
- Match continues for a maximum of **5 minutes** (configurable per layer)
- Ticket bleed is **suspended** during overtime
- No new tickets are awarded or deducted except through flag capture resolution
- Overtime ends when:
  - The contested flag is **Secured** by either team (that team wins)
  - The overtime timer expires (see Win Conditions)

### Phase 4: Match End

- All player input is disabled
- Final scores are displayed
- Server logs the match result

---

## 8. Win Conditions

Evaluated in this priority order:

1. **Tickets reach 0** - The team whose tickets reach 0 loses. If both teams reach 0 in the same server tick, the team that **controls more flags** wins.
2. **Final flag secured** - If a team captures the enemy's last flag, that team wins immediately regardless of ticket count.
3. **Time expires** - When the 75-minute timer runs out, the team with **more tickets** wins.
4. **Overtime expires** - When overtime ends without flag resolution, the team that **controls more flags** wins. If flag count is equal, the team with **more tickets** wins.
5. **Draw** - Only possible if tickets are equal, flag count is equal, and no flag is actively being captured. This is extremely rare by design.

---

## 9. Restrictions

### Role & Class Limits

Enforced per squad and per team as defined in `systems/roles-and-classes.md`:

- **Per squad:** Max 1 each of Medic, AT, MG, Marksman, Radio Operator. 1 Squad Leader required.
- **Per team:** Max 4 AT, 4 MG, 2 Marksman. No team-wide cap on Medic or Rifleman.
- Rifleman is the default role. Players who do not select a role are Rifleman.
- Loadouts are **fixed per role** — no customization.

### Vehicle Limits

Enforced per team as defined in `systems/roles-and-classes.md`:

- **Transport:** Max 4 per team (5 ticket cost)
- **Light Armor:** Max 2 per team (10 ticket cost)
- **Heavy Armor:** Max 1 per team (15 ticket cost)
- Layers may override these defaults (see `systems/layers.md`)

### Prohibited Actions

- **Intentional teamkilling** - Server-tracked. Excessive TKs result in automatic kick.
- **Glitching or exploiting** - Using unintended map geometry, clipping through walls, or abusing game engine bugs. Enforced by admin; results in disqualification in tournament play.
- **Stream sniping** - Using an opponent's broadcast to gain positional information. Enforced by tournament admin on evidence.

---

## 10. Disconnects & Pauses

### Player Disconnect

- A disconnected player has a **90-second reconnect window**
- During this window, their squad slot and role are held
- If they reconnect within 90s, they rejoin their squad with the same role
- After 90s, the slot is released and Squad Leader succession triggers if applicable (see `systems/squads.md`)
- A disconnected player's body remains in the world for 10 seconds, then despawns (counts as a death, -1 ticket)

### Admin Pause

- Only a designated **Admin / Referee** can pause the match
- Pause **freezes:** all match timers, capture progress, bleed ticks, and spawn waves
- Pause **does not freeze:** player positions (players remain where they are and can look around but cannot fire, move, or interact)
- Maximum pause duration: **10 minutes** per pause, **3 pauses per team** per match
- A team requests a pause through their captain to the referee. The referee executes the pause at the next safe moment (not mid-capture).
- Pauses may not be used tactically (e.g., pausing to scout enemy positions while frozen). Abuse results in forfeit of remaining pauses.

### Server Crash

- If the server crashes and cannot recover the match state, the match is **replayed** from the beginning
- If the server crashes and match state can be restored, play resumes from the restored state after a 60-second warm-up

---

## 11. Versioning & Authority

- File: `aas-v1.md`
- Version: **v1.0.0**
- Semantic versioning: MAJOR (rule changes affecting outcomes), MINOR (clarifications, new sections), PATCH (typos, formatting)
- This ruleset is the **official Phase 1+ competitive standard**
- Servers running ARC competitive AAS **must** follow this ruleset
- Layer configurations may adjust numerical values (tickets, timers, team size) but **may not** contradict rules
- Tournament organizers may add supplementary rules (scheduling, conduct) but **may not** override this ruleset
