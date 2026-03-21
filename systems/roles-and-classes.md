# Role & Class System

Defines all infantry roles, vehicle classes, their loadouts, and per-squad/per-team limits.

Referenced by: `rulesets/aas-v1.md`, `systems/squads.md`, `systems/layers.md`

---

## Design Principles

- Roles restrict loadout, not movement or positioning — every player can go anywhere.
- Every role serves a clear competitive function — no filler roles.
- Restrictions force squad composition decisions, not punish players.
- Role count stays low in Phase 1 — complexity grows with modes, not within a mode.

---

## Infantry Roles

| Role | Function | Loadout Core | Per-Squad Limit | Per-Team Limit |
|------|----------|-------------|-----------------|----------------|
| **Rifleman** | Baseline infantry | Assault rifle, grenades, bandages | Unlimited | Unlimited |
| **Medic** | Sustain squad health | Assault rifle, medical kit (full heal + revive) | 1 | -- |
| **Anti-Tank (AT)** | Counter vehicles | Assault rifle, AT launcher (limited ammo) | 1 | 4 |
| **Machine Gunner (MG)** | Area suppression, denial | Light machine gun, bipod | 1 | 4 |
| **Marksman** | Long-range precision | Designated marksman rifle, optic | 1 | 2 |
| **Radio Operator** | Deploy squad spawn point | Assault rifle, Small Radio backpack | 1 | -- |
| **Squad Leader (SL)** | Command, coordination | Assault rifle, binoculars, squad markers | 1 (required) | -- |

### Notes

- **Per-Squad Limit** of 1 means only one player in a squad can select that role. "--" means the per-team limit is sufficient or the role is structurally limited (one SL per squad by definition).
- **Per-Team Limit** caps the total across all squads. "--" means no team-wide cap beyond per-squad limits.
- Rifleman is the default. If a player doesn't select a role, they are Rifleman.
- SL is automatically assigned to the squad creator. If SL disconnects, see `systems/squads.md` for succession rules.
- Loadouts are fixed per role. No customization beyond what the role defines. This is a deliberate competitive constraint — no loadout meta-gaming.

---

## Limit Rationale

| Limit | Why |
|-------|-----|
| 1 Medic per squad | Prevents squads from becoming unkillable through heal stacking. Forces the squad to protect their medic as a high-value target. |
| 1 AT per squad, 4 per team | Ensures vehicles remain viable but can be countered. More than 4 AT across a team would make vehicles unplayable. |
| 1 MG per squad, 4 per team | MG is area denial. Stacking MGs on a flag makes it uncapturable. Team limit prevents all-MG cheese. |
| 1 Marksman per squad, 2 per team | Marksman has range advantage. More than 2 per team turns the game into a sniper fest — incompatible with objective play. |
| 1 Radio Operator per squad | Each squad gets exactly one forward spawn. Multiple radios per squad would make squads too resilient to wipes. |

---

## Vehicle Classes

| Class | Examples | Ticket Cost | Per-Team Limit | Respawn Timer |
|-------|----------|-------------|----------------|---------------|
| **Transport** | Trucks, unarmored cars | 5 | 4 | 3 min |
| **Light Armor** | APCs, IFVs | 10 | 2 | 5 min |
| **Heavy Armor** | Tanks, heavy IFVs | 15 | 1 | 8 min |

### Notes

- Vehicle classes and limits are defined per layer and can override these defaults. These are the reference values for standard AAS.
- **Construction Truck** is Transport-class but its primary purpose is HUB deployment. Costs 5 tickets on destruction.
- **MSP (Mobile Spawn Point)** is Transport-class but functions as a mobile spawn. Costs 5 tickets on destruction.
- Large vehicles (Heavy Armor) do not accelerate flag capture. Light Armor and Transport count as 1 player for capture purposes (the driver/dismounts count, the vehicle hull does not add extra capture weight).

---

## Layer Overrides

Layers can override any default limit. Common overrides:

- **Infantry-only layers:** Vehicle pool is empty. AT and MG limits may be adjusted.
- **Heavy vehicle layers:** Heavy Armor limit increased to 2. AT per-team limit increased to 6.
- **Asymmetric layers:** One team may have different vehicle classes than the other (e.g., more transports vs. fewer but heavier vehicles), as long as total ticket value is balanced.

Override rules are defined in `systems/layers.md`.
