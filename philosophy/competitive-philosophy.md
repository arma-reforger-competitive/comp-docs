# ARC Competitive Philosophy

This document defines the design principles behind the Arma Reforger Competitive ecosystem. Every ruleset, system, and decision is filtered through these principles.

This is not a rulebook. For rules, see `rulesets/`. For system designs, see `systems/`.

---

## The Problem

Arma Reforger is a military sandbox. Sandboxes optimize for freedom — any playstyle, any approach, any outcome. That freedom is the game's strength for casual and milsim play. It is its weakness for competitive play.

Competitive play requires:

- Both teams start with equal opportunity
- Outcomes are determined by skill and coordination, not by exploiting ambiguity
- Every rule is enforceable, every edge case has an answer
- Spectators and players can understand who is winning and why

Arma's sandbox provides none of this by default. ARC exists to build it.

---

## Core Principles

### 1. Competitive Integrity Above All

Every design decision is filtered through one question: **does this make fair, balanced play?**

If a mechanic is realistic but exploitable, it gets restricted. If a feature is popular but creates ambiguity, it gets removed or redesigned. Realism and freedom are not goals — they are tools that serve competition when they align, and are discarded when they don't.

This is not an anti-realism stance. Arma's core gunplay, movement, ballistics, and vehicle handling are realistic and competitive. ARC preserves these. What ARC restricts is the sandbox layer on top: unlimited loadouts, unstructured spawning, undefined win conditions, and uncontrolled game flow.

### 2. Server Authority

The server is the sole source of truth for all game state. Clients display information and send input — they never decide outcomes.

This means:

- **Captures** are validated server-side. A client cannot claim a flag.
- **Tickets** are tracked server-side. A client cannot modify ticket counts.
- **Spawns** are authorized server-side. A client cannot force a spawn at an invalid location.
- **Timers** run server-side. A client cannot manipulate match flow.

If the server doesn't confirm it, it didn't happen. This eliminates entire categories of exploits and ensures every player in the match is subject to the same rules enforced by the same authority.

### 3. Determinism Over Randomness

Competitive outcomes should be predictable from inputs. When a player makes a decision, the result should follow from that decision — not from a dice roll.

In practice:

- **No random spread** beyond the game's base weapon mechanics
- **No random spawn locations** — all spawn points are deliberate and placed by players or the layer
- **No random ticket events** — every ticket change is caused by a specific, trackable action
- **No random flag behavior** — capture times are formulaic, not variable

Where Arma's engine introduces randomness (e.g., bullet dispersion at extreme range), ARC accepts it as a skill-testing mechanic. But ARC never adds randomness on top.

### 4. Clarity of State

At any moment during a match, every player, spectator, and referee should be able to answer:

- Who is winning?
- What is the current score?
- What objectives are active?
- What needs to happen for either team to win?

This drives several design choices:

- **Linear flag progression** — the front line is always clear, never scattered across the map
- **Visible ticket counts** — both teams' tickets are displayed at all times
- **Explicit match phases** — warm-up, live, overtime, and end are distinct states with clear transitions
- **No hidden mechanics** — every system that affects the match outcome is documented and visible to players

Ambiguity is the enemy of competitive play. If a player doesn't know why they lost, the system has failed.

### 5. Enforceable Rules Only

If the server can't enforce a rule, it's not a rule — it's a suggestion. Suggestions create disputes. Disputes destroy competitive integrity.

Every rule in ARC must be:

- **Detectable** — the server can identify when it's being followed or violated
- **Preventable or punishable** — the server can either prevent the violation (preferred) or automatically apply consequences
- **Unambiguous** — there is no gray area about whether the rule was broken

Rules that require human judgment to enforce (e.g., "don't play in an unsportsmanlike manner") are reserved for tournament-level conduct rules administered by referees. They are never embedded in the core ruleset.

### 6. Constraints Create Depth

ARC intentionally limits Arma's sandbox freedom. This is a feature, not a compromise.

- **Fixed loadouts per role** force players to work within constraints and coordinate with squadmates instead of everyone carrying the optimal all-purpose kit
- **Squad size limits** force multiple squads to coordinate instead of one mega-squad steamrolling
- **Vehicle class limits** force teams to decide how to deploy limited armor instead of spamming tanks
- **Linear flag progression** forces head-to-head engagements instead of backcapping empty objectives

Unlimited freedom leads to solved metas where the optimal strategy is discovered and repeated. Constraints create decision points. Decision points create depth. Depth is what makes competitive play rewarding across hundreds of matches.

### 7. Every Number Has a Rationale

No value in ARC is arbitrary. Every timer, limit, cost, and threshold exists for a stated reason.

Examples:

- **400 starting tickets** — enough for a 75-minute match with active combat without running out from deaths alone. Forces ticket discipline around vehicles and flags.
- **90-second base capture time** — long enough that a single player can't ninja-cap, short enough that a coordinated push is rewarded. Scaling down to 35s at 4+ players rewards team play.
- **-20 tickets for losing a flag** — severe enough to make flag defense a priority, not so severe that losing one flag is unrecoverable.
- **+30 tickets for defending a flag** — net positive over losing one (+30 vs -20) to reward teams that contest rather than concede.
- **6-player squads** — forces 4 squads at 24v24, requiring inter-squad coordination. Larger squads would reduce the need for SL-to-SL communication.

When a number changes, its rationale must be updated. If a number can't be justified, it should be questioned.

---

## What ARC Is Not

**ARC is not milsim.** Milsim prioritizes immersion, role-playing, and realistic procedures. ARC borrows Arma's realistic mechanics but discards milsim conventions that don't serve competition (mandatory radio protocols, realistic convoy procedures, etc.).

**ARC is not casual.** There are no mechanics designed to help losing teams catch up (rubber-banding), no matchmaking adjustments mid-game, and no participation rewards. Better teams win. The system's job is to ensure "better" means more skilled and coordinated, not more exploitative.

**ARC is not a mod showcase.** Features are not added because they're technically possible. Every feature must justify its existence through competitive value. If it doesn't make matches more fair, more clear, or more skillful — it doesn't ship.

**ARC is not finished.** Every document is a living draft. Numbers will change as playtesting reveals balance issues. Systems will be added as competitive needs evolve. The principles in this document are the stable foundation — everything else is subject to evidence-based revision.

---

## How Decisions Are Made

When a design question arises, evaluate it against the principles in order:

1. Does it preserve competitive integrity?
2. Is it server-authoritative?
3. Is it deterministic?
4. Is the resulting game state clear to all participants?
5. Can it be enforced by the server?
6. Does the constraint create meaningful decisions?
7. Can every number in the design be justified?

If a proposed feature fails any of these checks, it needs to be redesigned or rejected. The principles are not negotiable — they are the reason ARC exists.
