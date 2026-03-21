# Squad System

Defines squad structure, creation, leadership, and disconnect handling.

Referenced by: `rulesets/aas-v1.md`, `systems/roles-and-classes.md`, `systems/layers.md`

---

## Squad Structure

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| **Max squad size** | 6 | Small enough to require multiple squads for coordination. Large enough to field a meaningful fireteam with role diversity. Fits common team sizes (18v18 = 3 squads, 24v24 = 4 squads). |
| **Min squad size** | 1 | SL can exist alone (e.g., creating a squad before others join). |
| **Max squads per team** | No artificial cap | Derived naturally from team size. A 24-player team could have 4 full squads or 6 partial squads — both are valid tactical choices. |
| **Squad naming** | Auto-assigned: Alpha, Bravo, Charlie, Delta, Echo, Foxtrot, Golf, Hotel | No custom names. Keeps comms clean and prevents abuse. |

---

## Squad Creation & Joining

1. Any player without a squad can **create** a squad. They become Squad Leader automatically.
2. Squads are **open join** by default — any team member can join until the squad is full.
3. SL can **lock** the squad to prevent new joins. Locked squads show as locked in the squad list.
4. SL can **kick** a player from the squad. Kicked players can join other squads or create their own.
5. Players can **leave** a squad voluntarily at any time.

---

## Squad Leader Succession

When a Squad Leader leaves (disconnect, kick, voluntary leave):

1. If the squad has other members, the **longest-serving member** becomes SL automatically.
2. The new SL retains their current role and gains SL tools (markers, orders). They do **not** get the SL loadout until they respawn and select it.
3. If the last player leaves, the squad is dissolved.

**Rationale:** Automatic succession prevents orphaned squads. Longest-serving avoids randomness. Not forcing an immediate loadout swap prevents disrupting a live firefight.

---

## Squad Leader Tools

| Tool | Description |
|------|-------------|
| **Map Markers** | Place attack, defend, move, and observe markers visible to the squad and all SLs. |
| **Squad Lock/Unlock** | Toggle squad open or closed for new joins. |
| **Kick** | Remove a squad member. |
| **SL Channel** | Voice/text channel shared among all Squad Leaders on the team. Enables inter-squad coordination without cluttering squad comms. |

---

## Disconnect Handling

| Scenario | Behavior |
|----------|----------|
| **Player disconnects mid-match** | 90-second reconnect window. Slot is held. If they reconnect within 90s, they rejoin their squad with the same role. After 90s, slot is released. |
| **SL disconnects** | Same 90s window. If they don't return, succession activates per rules above. |
| **Entire squad disconnects** | Squad is dissolved after the last member's 90s window expires. Any deployed Small Radio from that squad is destroyed. |

**Rationale:** 90-second window prevents abrupt punishments for connection issues while limiting exploit potential (intentional disconnect to reset position or avoid death).

---

## Impact on Other Systems

| System | Effect |
|--------|--------|
| **Spawn** | "Squad-only" spawn (Small Radio) means only members of the squad whose Radio Operator placed it. |
| **Roles** | Per-squad role limits are enforced against squad membership. Changing squads may require changing role if the new squad already has that role filled. |
| **Match Flow** | Warm-up phase is used for squad formation and role selection. |
| **Tickets** | No direct ticket impact. Squad wipes are not penalized beyond individual death tickets. |
