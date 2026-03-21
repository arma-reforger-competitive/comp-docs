# Spawn System Spec

Technical specification for the spawn point and respawn system.

Audience: Developers implementing `arma-comp-core/src/GameMode/` (spawn logic).

---

## Overview

The spawn system controls where and when dead players can re-enter the match. It manages four spawn point types, a wave-based respawn timer, and validation rules that prevent spawning at invalid locations.

---

## Data Model

```
SpawnPoint {
  id:          int
  type:        SpawnType
  teamId:      TeamID
  position:    vector3
  state:       SpawnState
  squadId:     int | null    // Only for SmallRadio (squad-locked)
  entityId:    int | null    // Linked vehicle/object entity (for MSP, HUB, SmallRadio)
}

enum SpawnType {
  MainBase,
  SmallRadio,
  MSP,
  HUB
}

enum SpawnState {
  Active,      // Available for spawning
  Disabled,    // Exists but cannot be used (enemy nearby, contested)
  Destroyed    // Entity destroyed, spawn point removed
}

SpawnWave {
  teamId:      TeamID
  timer:       float         // Countdown to next wave (seconds)
  interval:    float         // Wave interval (default 30s, from layer)
  queue:       list<PlayerID>  // Players waiting to spawn
}

RespawnRequest {
  playerId:    PlayerID
  spawnPointId: int
  timestamp:   float
}
```

---

## Spawn Point Types

### Main Base

```
Properties:
  type:        MainBase
  permanent:   true (indestructible)
  access:      All team members
  count:       1 per team
  position:    Fixed (from layer config)
  protectionZone: 150m radius — enemies cannot enter
```

- Always `Active` (never disabled or destroyed)
- Position defined in layer config, validated to be outside all capture zones
- Enemy players entering the 150m protection zone receive a 5-second warning, then are killed by the server

### Small Radio

```
Properties:
  type:        SmallRadio
  permanent:   false (destructible)
  access:      Deploying squad only
  count:       1 per squad
  deployedBy:  Radio Operator role
  health:      Destructible (takes damage from weapons, explosives)
  disableRadius: 30m (enemy proximity)
```

**Deployment rules:**
- Only a Radio Operator can deploy
- Deploying a new Small Radio destroys the squad's previous one
- Cannot be placed within 100m of Main Base (prevents redundant spawn stacking)
- Cannot be placed within 30m of another team spawn point (prevents spawn clustering)
- Cannot be placed inside a capture zone that is Locked
- Placement requires the Radio Operator to be stationary for 5 seconds (interruptible by damage or movement)

**Destruction conditions:**
- Destroyed by enemy fire (has a health pool)
- Destroyed if Radio Operator dies and no squad member is within 50m (auto-destroy after 30 seconds, not instant)
- Destroyed if the squad is dissolved (all members disconnect)
- Destroyed if a new Small Radio is deployed by the same squad

**Disable conditions:**
- Enemy player within 30m → state = `Disabled`
- Re-enabled when no enemy is within 30m
- Check runs every 2 seconds (not every tick, for performance)

### MSP (Mobile Spawn Point)

```
Properties:
  type:        MSP
  permanent:   false (destructible vehicle)
  access:      All team members
  count:       Per layer vehicle pool (typically 1)
  vehicle:     Dedicated MSP truck entity
  disableRadius: 30m (enemy proximity)
```

**State rules:**
- `Active` when: vehicle is stationary, engine off (deployed state), no enemy within 30m
- `Disabled` when: vehicle is moving, engine on, or enemy within 30m
- `Destroyed` when: vehicle entity is destroyed

**Deployment:** Driver stops the vehicle and activates deploy mode (engine off, deploy action). Undeploying requires the same action (takes 5 seconds, interruptible).

**Movement:** When undeployed, the MSP is a normal Transport-class vehicle. It can be driven and relocated. Spawning is disabled while moving/undeployed.

### HUB

```
Properties:
  type:        HUB
  permanent:   false (destructible structure)
  access:      All team members
  count:       Per layer (typically 1 Construction Truck can build 1 HUB)
  builtBy:     Construction Truck
  health:      Higher than Small Radio (requires significant effort to destroy)
  disableRadius: 30m (enemy proximity)
```

**Construction rules:**
- A Construction Truck driver activates build mode at the desired location
- Building takes 60 seconds and requires the truck to remain stationary
- Building is interrupted if the truck takes damage or moves
- Once built, the Construction Truck is consumed (removed from play)
- The team does not get the Construction Truck back until the HUB is destroyed and the vehicle respawn timer elapses

**Destruction conditions:**
- Destroyed by enemy fire (higher health pool than Small Radio)
- Cannot be self-destructed by the owning team (prevents exploit: destroy and rebuild in a better position for free)

**Disable conditions:**
- Same as MSP: enemy within 30m → `Disabled`

---

## Spawn Wave System

Respawns are not instant. Dead players join a queue and spawn together in waves.

```
processSpawnWave(teamId):
  wave = getSpawnWave(teamId)
  wave.timer -= deltaTime

  if wave.timer <= 0:
    wave.timer += wave.interval  // Add interval (preserves remainder)

    for each player in wave.queue:
      spawnPoint = player.selectedSpawnPoint
      if validateSpawnRequest(player, spawnPoint):
        spawnPlayer(player, spawnPoint)
        remove player from queue
      else:
        // Invalid spawn point selected — player stays in queue
        // UI notifies player to select a different spawn point
        notifyInvalidSpawn(player, spawnPoint)
```

### Wave Timing

- Wave interval: **30 seconds** (configurable per layer)
- The wave timer is **global per team** — both teams share the same interval but their timers are independent
- Timer starts at match start (phase = Live) and ticks continuously
- Timer is frozen during pauses

### Death-to-Spawn Flow

```
1. Player dies
   → Player enters spectator/death cam (10 seconds minimum)
   → Ticket event fired (-1 to team)

2. After 10-second death cam:
   → Player enters spawn selection screen
   → Player sees available spawn points (Active only)
   → Player selects a spawn point

3. Player joins their team's spawn wave queue
   → Waits for next wave tick

4. Wave ticks
   → If selected spawn point is still valid: player spawns
   → If selected spawn point is now invalid (destroyed, disabled): player stays in queue, notified to reselect
```

### Minimum Death Time

Players must be dead for at least **10 seconds** before they can enter the spawn queue. This is independent of the wave timer. A player who dies 1 second before a wave must wait for the *next* wave (10s death cam means they miss the immediate wave).

---

## Spawn Validation

Every spawn request is validated server-side before execution:

```
validateSpawnRequest(playerId, spawnPointId) -> bool:
  sp = getSpawnPoint(spawnPointId)
  player = getPlayer(playerId)

  // Basic checks
  if sp == null: return false
  if sp.state != Active: return false
  if sp.teamId != player.teamId: return false

  // Squad check for Small Radio
  if sp.type == SmallRadio:
    if sp.squadId != player.squadId: return false

  // Flag proximity check
  for each flag in getAllFlags():
    if flag.state == Locked and isInsideZone(sp.position, flag):
      return false  // Cannot spawn inside a Locked flag zone
    if flag.state == Capturing and isInsideZone(sp.position, flag):
      if flag.capturingTeam != player.teamId:
        return false  // Cannot spawn inside a flag being captured by the enemy

  // Match phase check
  if matchPhase != Live and matchPhase != Overtime:
    return false

  return true
```

---

## Enemy Proximity Check

For Small Radio, MSP, and HUB — the 30m disable radius:

```
// Runs every 2 seconds for each deployable spawn point (not every tick)
updateSpawnPointStates():
  for each sp in getAllDeployableSpawnPoints():
    if sp.state == Destroyed: continue

    enemyTeam = getOpponent(sp.teamId)
    nearestEnemy = getNearestPlayerDistance(sp.position, enemyTeam)

    if nearestEnemy <= 30.0:
      sp.state = Disabled
    else:
      if sp was previously Disabled:
        sp.state = Active
```

The 2-second check interval is a performance optimization. A 30m radius with 2-second polling means an enemy sprinting at ~6 m/s covers 12m between checks — still well within the disable zone before any spawn could occur (spawns only happen on wave ticks).

---

## Vehicle Spawning

Vehicles respawn independently from player spawns:

```
VehicleSpawn {
  vehicleClass:  VehicleClass
  vehicleType:   string         // Specific vehicle type from layer
  teamId:        TeamID
  spawnPosition: vector3        // Main Base vehicle pad
  respawnTimer:  float          // Countdown after destruction
  isAlive:       bool
}
```

**Rules:**
- Vehicles spawn only at **Main Base** at match start
- After destruction, respawn timer starts (Transport: 180s, Light Armor: 300s, Heavy Armor: 480s)
- Respawned vehicles appear at Main Base vehicle pad
- Vehicles **do not respawn** at MSP or HUB (correction from ruleset — vehicles are assets that return to base)
- Vehicle respawn is disabled during Overtime

---

## Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| Player death | Game engine | Start death cam, add to spawn queue after 10s |
| `EVENT_FLAG_CAPTURED` | Flag system | Recalculate valid spawn points (flags may now be Locked/Contested) |
| `EVENT_FLAG_CONTESTED` | Flag system | Disable spawns on affected flag area |
| `EVENT_MATCH_PHASE_CHANGED` | Match flow | Enable/disable spawn system |
| `EVENT_MATCH_PAUSED` | Match flow | Freeze wave timer |
| Entity destroyed | Game engine | Mark spawn point as Destroyed, notify queued players |

## Events Emitted

| Event | Payload | Consumed By |
|-------|---------|-------------|
| `EVENT_PLAYER_SPAWNED` | playerId, spawnPointId, position | HUD, logging |
| `EVENT_SPAWN_POINT_CREATED` | SpawnPoint | HUD (add to map), spawn validation |
| `EVENT_SPAWN_POINT_DESTROYED` | spawnPointId | HUD (remove from map), spawn queue (notify affected players) |
| `EVENT_SPAWN_POINT_STATE_CHANGED` | spawnPointId, newState | HUD (grayed out if disabled), spawn validation |
| `EVENT_SPAWN_WAVE` | teamId, playerCount | Logging |
