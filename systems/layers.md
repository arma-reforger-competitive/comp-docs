# Layer System

Defines what a competitive layer is, what it contains, and how it interfaces with the core mod.

Referenced by: `rulesets/aas-v1.md`, `systems/roles-and-classes.md`

---

## What Is a Layer

A layer is a **complete match configuration** for a specific map. It contains everything the server needs to run a competitive match beyond what the core mod provides.

Layers live in **map-specific mods** (e.g., `comp-arland`), not in `arma-comp-core`. The core mod provides the layer **parser and validator**.

---

## Layer Contents

| Section | What It Defines | Required |
|---------|----------------|----------|
| **Meta** | Layer ID, display name, map, mode, version, compatible core version | Yes |
| **Teams** | Faction assignments (e.g., Team 1 = US, Team 2 = USSR), team size | Yes |
| **Flags** | Ordered list of flag positions (coordinates + capture radius), flag names, initial ownership | Yes |
| **Tickets** | Starting tickets per team | No (defaults to 400) |
| **Vehicle Pool** | Per-team vehicle list: class, type, quantity, respawn timer | No (defaults to none = infantry only) |
| **Spawn Points** | Main Base positions per team, allowed spawn types (Small Radio, MSP, HUB) | Yes |
| **Role Overrides** | Changes to default role/class limits | No (defaults to standard) |
| **Timers** | Match duration, warm-up, overtime, spawn wave interval, bleed interval | No (defaults to core) |
| **Restrictions** | Layer-specific restrictions (e.g., "no Heavy Armor", "infantry only") | No |

---

## Naming Convention

```
{map}-{mode}-{variant}-v{version}
```

**Examples:**
- `arland-aas-standard-v1` — Arland, AAS, standard vehicle mix
- `arland-aas-infantry-v1` — Arland, AAS, infantry only
- `arland-aas-heavy-v1` — Arland, AAS, full vehicle roster

---

## Example Layer

```
Layer: arland-aas-standard-v1

Meta
  id:           arland-aas-standard-v1
  name:         Arland AAS Standard
  map:          Arland
  mode:         AAS
  version:      1.0.0
  core_version: >=1.0.0

Teams
  team_1:       { faction: US, size: 24 }
  team_2:       { faction: USSR, size: 24 }

Flags (ordered)
  1: Lumber Mill     pos: [x,y,z]  radius: 50  initial: team_1
  2: Crossroads      pos: [x,y,z]  radius: 40  initial: team_1
  3: Church          pos: [x,y,z]  radius: 35  initial: neutral
  4: Factory         pos: [x,y,z]  radius: 40  initial: team_2
  5: Radar Station   pos: [x,y,z]  radius: 50  initial: team_2

Tickets
  team_1: 400
  team_2: 400

Vehicle Pool
  team_1:
    Transport:    M998 HMMWV x3         respawn: 180s
    Light Armor:  M2A3 Bradley x1       respawn: 300s
    Transport:    M939 (Construction) x1 respawn: 300s
  team_2:
    Transport:    UAZ-469 x3            respawn: 180s
    Light Armor:  BMP-2 x1             respawn: 300s
    Transport:    Ural (Construction) x1 respawn: 300s

Spawn Points
  team_1: main_base: [x,y,z]  allowed: [Small Radio, MSP, HUB]
  team_2: main_base: [x,y,z]  allowed: [Small Radio, MSP, HUB]

Timers
  match_duration:  4500s (75 min)
  warmup:          60s
  overtime:        300s (5 min)
  spawn_wave:      30s
  bleed_interval:  30s

Restrictions
  (none — standard rules apply)
```

---

## Validation Rules

The core mod validates a layer on load and rejects invalid configurations:

1. **Flags must have at least 3 entries** — fewer than 3 makes linear progression meaningless.
2. **Flags must have an odd initial split or a neutral center** — ensures both teams have something to fight for immediately.
3. **Team sizes must be equal** — asymmetric team sizes are not supported in AAS v1.
4. **Vehicle pool per team must not exceed class limits** — e.g., cannot assign 3 Heavy Armor if the class limit is 1.
5. **Main Base positions must be outside all capture zones** — prevents spawn camping by design.
6. **Core version compatibility must be satisfied** — layer declares minimum core version.

---

## Where Layers Live

```
comp-arland/
  /layers
    arland-aas-standard-v1.conf
    arland-aas-infantry-v1.conf
    arland-aas-heavy-v1.conf
```

- Each map mod contains a `/layers` directory with one file per layer.
- `arma-comp-core` provides the layer parser and validator — it does not contain layers.
- Tournament organizers reference layers by ID in match configuration.
- Map veto/pick in tournaments = layer veto/pick from a defined pool.

---

## Default Values

When a layer omits optional sections, the core mod applies these defaults:

| Parameter | Default |
|-----------|---------|
| Tickets per team | 400 |
| Match duration | 75 minutes |
| Warm-up | 60 seconds |
| Overtime | 5 minutes |
| Spawn wave interval | 30 seconds |
| Bleed interval | 30 seconds |
| Vehicle pool | Empty (infantry only) |
| Role limits | Standard (see `systems/roles-and-classes.md`) |
| Allowed spawn types | All (Main Base, Small Radio, MSP, HUB) |
