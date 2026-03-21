# ARC Terminology Glossary

Canonical definitions for all terms used across ARC documents.
Every ruleset, spec, and guide references these definitions.

---

## General

| Term | Definition |
|------|-----------|
| **ARC** | Arma Reforger Competitive — the competitive ecosystem including core mod, map layers, rulesets, and tooling. |
| **Layer** | A specific competitive configuration for a map. Defines flag positions, flag order, vehicle pools, spawn locations, team assignments, and match settings. A single map can have multiple layers. |
| **Match** | A single competitive game between two teams, played on one layer, from warm-up through end. |
| **Round** | Synonym for match in single-round formats. Reserved for future multi-round formats. |
| **Server Authority** | Design principle: the server is the sole source of truth for all game state. Clients display and request — they never decide. |

---

## Teams & Players

| Term | Definition |
|------|-----------|
| **Team** | One of two opposing factions in a match. Each team has its own ticket pool, spawn points, and vehicle pool. |
| **Squad** | A subgroup within a team. Fixed maximum size (defined per layer, default 6). Has a Squad Leader and optional specialist roles. |
| **Squad Leader (SL)** | The player who creates or inherits a squad. Can place markers, issue orders, and approve squad join requests. Cannot be removed from the role except by leaving the squad or disconnecting. |
| **Radio Operator** | A squad role. Can deploy one Small Radio for the squad. Only one Radio Operator per squad. |
| **Role** | A player's class selection that determines loadout and capabilities. See `/systems/roles-and-classes.md`. |

---

## Objectives

| Term | Definition |
|------|-----------|
| **Flag** | A capturable objective point on the map. Defined by a geographic zone with a center point and capture radius. |
| **Flag State: Locked** | The flag is not yet active in the linear progression. Cannot be captured, spawned on, or interacted with competitively. Displayed as inactive on HUD. |
| **Flag State: Neutral** | The flag is active but not owned by either team. Can be captured. |
| **Flag State: Capturing** | A team has players inside the capture zone and is actively converting ownership. Shows capture progress. |
| **Flag State: Secured** | A team has completed capture and owns the flag. Contributes to flag count for bleed calculations. |
| **Contested** | A flag where both teams have players inside the capture zone simultaneously. Capture progress is paused. No spawning is allowed on a contested flag. |
| **Linear Progression** | Flags must be captured in order. A team can only attack the next flag in the chain. Flags beyond the active front are Locked. |
| **Active Flag** | The one flag each team is currently attacking or defending — the front line of the linear chain. Each team has at most one active attack target. |
| **Capture Zone** | The geographic area around a flag where player presence counts toward capture. Defined per flag in the layer. |
| **Capture Radius** | The radius of the capture zone, measured from the flag center. |

---

## Tickets

| Term | Definition |
|------|-----------|
| **Tickets** | A team's resource pool. Depleted by deaths, vehicle losses, and flag losses. Replenished by flag defenses. When tickets reach 0, the team loses. |
| **Ticket Bleed** | Passive ticket drain applied to a team that controls fewer flags than the opponent. Rate: -1 per 30 seconds. |
| **Heavy Bleed** | Accelerated ticket bleed (-3 per 30s) applied when a team has lost all flags except their final one. |
| **Ticket Event** | Any game event that modifies a team's ticket count. All ticket events are server-authoritative and logged. |

---

## Spawn System

| Term | Definition |
|------|-----------|
| **Spawn Point** | A location where dead players can re-enter the match. Each type has different placement rules, access restrictions, and persistence. |
| **Main Base** | A fixed, indestructible spawn point per team. Located at the team's map edge. All team members may spawn here. Always active. |
| **Small Radio** | A deployable squad-only spawn point. Placed by the Radio Operator. Destructible. Only the deploying squad can spawn on it. One per squad. |
| **MSP (Mobile Spawn Point)** | A truck-mounted team-wide spawn point. Mobile and relocatable. Destructible. All team members can spawn while it is deployed. |
| **HUB** | A stationary team-wide spawn point deployed from a Construction Truck. Destructible. All team members can spawn while it is intact. |
| **Spawn Timer** | Server-controlled cooldown between a player's death and their availability to respawn. Prevents instant redeployment. |
| **Spawn Wave** | A fixed-interval respawn system where dead players spawn together in groups rather than individually. Interval is server-configured. |

---

## Vehicles

| Term | Definition |
|------|-----------|
| **Vehicle Pool** | The set of vehicles available to a team, defined per layer. Specifies type, quantity, and respawn behavior. |
| **Vehicle Class** | A category of vehicle determining its ticket cost, combat role, and restrictions. See `/systems/roles-and-classes.md`. |
| **Vehicle Stacking** | Concentrating multiple vehicles on a single flag. Prohibited to prevent overwhelming a capture zone with armor. |
| **Vehicle Respawn Timer** | Time after destruction before a replacement vehicle becomes available. Server-controlled, defined per vehicle class in the layer. |

---

## Match Flow

| Term | Definition |
|------|-----------|
| **Warm-up** | Pre-match phase (60 seconds). Players can move within Main Base area only. No combat, no flag captures, no ticket changes. Used for squad organization and role selection. |
| **Live** | The active match phase. All systems active. Maximum duration defined per layer (default: 75 minutes). |
| **Overtime** | Triggered when a team's tickets reach 0 while the final flag is Contested or Capturing. Match continues until the flag resolves (Secured by either team) or a configurable overtime timer expires (default: 5 minutes). |
| **Match End** | Terminal state. Triggered by: tickets reaching 0, final flag secured, time expiring, or overtime expiring. Winner is declared and the match is locked — no further game state changes. |

---

## Administration

| Term | Definition |
|------|-----------|
| **Admin / Referee** | A designated match official with server commands to pause, restart, or intervene. Not a player. |
| **Pause** | Admin-initiated suspension of all match timers, captures, and bleed. Players remain in position but cannot fire or interact. |
| **Layer Configuration** | The server-side file or settings that define a layer's parameters: flags, vehicles, teams, timers, ticket counts. |
