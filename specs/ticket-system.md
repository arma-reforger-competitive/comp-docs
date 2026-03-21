# Ticket System Spec

Technical specification for the team ticket and bleed system.

Audience: Developers implementing `arma-comp-core/src/Tickets/`.

---

## Overview

Tickets are each team's primary resource. Every competitive action costs or rewards tickets. When a team's tickets reach 0, they lose. The system is entirely server-authoritative — clients receive ticket counts for display but cannot modify them.

---

## Data Model

```
TicketPool {
  teamId:     TeamID
  tickets:    int        // Current ticket count (never negative, clamped to 0)
  maxTickets: int        // Starting tickets (from layer config, default 400)
}

TicketEvent {
  timestamp:  float      // Server time
  teamId:     TeamID     // Affected team
  delta:      int        // Change amount (negative = loss, positive = gain)
  reason:     EventType  // What caused the change
  sourceId:   int | null // Player or entity that caused the event (null for bleed)
}

enum EventType {
  PlayerDeath,
  VehicleDestroyed_Transport,
  VehicleDestroyed_LightArmor,
  VehicleDestroyed_HeavyArmor,
  FlagLost,
  FlagDefended,
  Bleed,
  FinalFlagBleed
}
```

---

## Ticket Costs

| Event | Delta | Applied To |
|-------|-------|-----------|
| Player death | -1 | Dead player's team |
| Transport destroyed | -5 | Vehicle owner's team |
| Light Armor destroyed | -10 | Vehicle owner's team |
| Heavy Armor destroyed | -15 | Vehicle owner's team |
| Flag lost | -20 | Team that lost the flag |
| Flag defended | +30 | Team that held the flag |
| Bleed tick | -1 | Team with fewer flags |
| Final flag bleed tick | -3 | Team on their last flag |

### Player Death Rules

- A death is counted when a player enters the dead state (health reaches 0)
- Revived players: if a Medic revives a player before they respawn, the death ticket **still counts**. The ticket was spent on the death event, not on respawn. This prevents revive mechanics from making deaths free.
- Teamkills: the **team of the killed player** loses the ticket (not the killer's team). Teamkills are punished administratively, not via ticket manipulation on the killing team.
- Suicide / fall damage / environmental death: -1 to the player's team. No source player.
- Disconnect body despawn: -1 to the disconnecting player's team (treated as a death).

### Vehicle Destruction Rules

- Ticket cost is applied when the vehicle entity is destroyed (health reaches 0), not when it is disabled or abandoned
- Vehicle ticket cost is charged to the **team that owns the vehicle**, not the team that destroyed it
- If a team destroys their own vehicle (teamkill), the ticket cost still applies to that team
- Unoccupied vehicle destruction: same cost. Vehicles are team assets regardless of occupancy.

### Flag Event Rules

- **Flag lost (-20):** Fired when `EVENT_FLAG_CAPTURED` is received and `previousOwner != null`. Applied to `previousOwner`.
- **Flag defended (+30):** Fired when an enemy's capture progress on a Secured flag decays back to 0.0 while the owning team has players in the zone. Applied to the flag owner.
- A flag changing from Neutral to Secured (first capture at match start) does **not** trigger Flag Lost because there was no previous owner.

---

## Bleed System

### Bleed Evaluation

Bleed is evaluated on a fixed server timer, not continuously:

```
bleedInterval = 30 seconds (configurable per layer)
bleedTimer = 0

onServerTick(deltaTime):
  bleedTimer += deltaTime
  if bleedTimer >= bleedInterval:
    bleedTimer -= bleedInterval  // Subtract, don't reset (preserves remainder)
    evaluateBleed()
```

### Bleed Logic

```
evaluateBleed():
  team1Flags = count of flags where owner == Team1
  team2Flags = count of flags where owner == Team2

  // Standard bleed: team with fewer flags bleeds
  if team1Flags < team2Flags:
    applyTicketEvent(Team1, -1, Bleed)
  else if team2Flags < team1Flags:
    applyTicketEvent(Team2, -1, Bleed)
  // Equal flags: no bleed

  // Final flag bleed: stacks with standard bleed
  totalFlags = team1Flags + team2Flags + neutralFlags
  if team1Flags == 1 and team2Flags >= (totalFlags - 1):
    applyTicketEvent(Team1, -3, FinalFlagBleed)  // On top of the -1 above
  if team2Flags == 1 and team1Flags >= (totalFlags - 1):
    applyTicketEvent(Team2, -3, FinalFlagBleed)
```

**Final flag bleed stacks with standard bleed.** A team on their last flag loses -4 total per bleed tick (-1 standard + -3 final flag).

### Bleed During Overtime

Bleed is **suspended** during overtime. The bleed timer stops and does not tick.

---

## Ticket Modification

All ticket changes go through a single function:

```
applyTicketEvent(teamId, delta, reason, sourceId = null):
  pool = getTicketPool(teamId)

  // Clamp: tickets cannot go below 0
  newTickets = max(0, pool.tickets + delta)
  actualDelta = newTickets - pool.tickets
  pool.tickets = newTickets

  // Log the event
  event = TicketEvent {
    timestamp: serverTime,
    teamId: teamId,
    delta: actualDelta,  // Actual applied amount (may differ from requested if clamped)
    reason: reason,
    sourceId: sourceId
  }
  logTicketEvent(event)

  // Notify systems
  emit EVENT_TICKETS_CHANGED(teamId, pool.tickets, actualDelta, reason)

  // Check win condition
  if pool.tickets == 0:
    emit EVENT_TICKETS_ZERO(teamId)
```

### Ordering and Race Conditions

Multiple ticket events can occur in the same server tick (e.g., two players die simultaneously). Rules:

1. All events in the same tick are processed sequentially in this priority order:
   - Flag events (captured/defended) — highest priority
   - Vehicle destruction
   - Player death
   - Bleed — lowest priority
2. Within the same priority, events are processed in the order they were received by the server
3. If tickets reach 0 partway through processing, remaining events for that team are skipped
4. `EVENT_TICKETS_ZERO` is emitted once, after all tick processing is complete

This prevents a scenario where a flag defense (+30) and deaths (-1 each) happen in the same tick — the flag defense is processed first, potentially keeping the team alive.

---

## Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| Player death | Game engine | applyTicketEvent(team, -1, PlayerDeath, playerId) |
| Vehicle destroyed | Game engine | applyTicketEvent(team, -cost, VehicleDestroyed_*, vehicleId) |
| `EVENT_FLAG_CAPTURED` | Flag system | applyTicketEvent(previousOwner, -20, FlagLost); if defended, applyTicketEvent(owner, +30, FlagDefended) |
| Bleed timer | Internal | evaluateBleed() |

## Events Emitted

| Event | Payload | Consumed By |
|-------|---------|-------------|
| `EVENT_TICKETS_CHANGED` | teamId, currentTickets, delta, reason | HUD (ticket counter update), match flow (logging) |
| `EVENT_TICKETS_ZERO` | teamId | Match flow (triggers win condition or overtime check) |
