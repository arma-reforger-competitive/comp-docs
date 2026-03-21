# Admin & Referee Protocol

Defines the role, authority, and tools available to match officials in ARC competitive play.

Referenced by: `rulesets/aas-v1.md` (Section 10: Disconnects & Pauses)

---

## Roles

### Referee

A designated match official responsible for enforcing rules during a competitive match. Not a player. Required for all tournament matches. Optional for ranked or organized play.

**Authority:**
- Pause and unpause the match
- Restart a match (requires both team captains' consent unless a server crash occurred)
- Kick a player for rule violations
- Issue warnings for conduct violations
- Disqualify a team for repeated or severe violations
- Make final rulings on disputes during a match

**Limitations:**
- Cannot modify tickets, flag ownership, or match timers (no manual score manipulation)
- Cannot spawn items, vehicles, or players
- Cannot override ruleset rules — only enforce them
- Cannot pause for personal reasons — only for rule enforcement or team-requested pauses

### Server Admin

The person with full server access. May or may not be the referee. In tournament play, the server admin operates the infrastructure while the referee handles the match.

**Authority (beyond referee):**
- Start and stop the server
- Load and change layers
- Configure server settings
- Restore match state after a crash
- Access server logs

**Restriction:** In tournament play, the server admin **must not** be a player on either team.

### Team Captain

One designated player per team who is the point of contact for the referee. Does not need to be a Squad Leader.

**Responsibilities:**
- Request pauses on behalf of the team
- Communicate team decisions to the referee (e.g., consent to restart)
- Receive warnings and rulings from the referee
- Participate in map/layer veto (see Tournament Operations)

---

## Referee Commands

All referee actions are executed through server commands and logged automatically.

| Command | Effect | Restrictions |
|---------|--------|-------------|
| **Pause** | Freezes match timers, captures, bleed, spawn waves. Players cannot move, fire, or interact. | Max 10 min per pause. Max 3 pauses per team per match. |
| **Unpause** | Resumes match with a 5-second countdown displayed to all players. | Must follow a pause. |
| **Warn** | Issues a formal warning to a player. Displayed to the player and logged. | 3 warnings to the same player = automatic kick. |
| **Kick** | Removes a player from the match. They cannot rejoin. | Logged with reason. Team plays short-handed (no substitute). |
| **Restart** | Ends the current match and starts fresh on the same layer. | Requires both captains' consent, except after server crash. |
| **End Match** | Ends the match immediately with a declared winner. | Only for unresolvable situations (e.g., team refuses to play, mass disconnect with no recovery). Must state reason. |

### Command Logging

Every referee command is written to the server log with:
- Timestamp
- Command executed
- Target player (if applicable)
- Reason (mandatory for Warn, Kick, End Match)
- Referee identity

Logs are the official record for post-match disputes.

---

## Violations & Penalties

### Severity Levels

| Level | Examples | Penalty |
|-------|----------|---------|
| **Minor** | Accidental teamkill, spawning on wrong point due to UI error | Verbal warning (not formal). No log entry required. |
| **Moderate** | Repeated teamkilling (3+ in a match), intentional glitch use, exploiting spawn mechanics | Formal warning. 3rd formal warning = kick. |
| **Severe** | Confirmed cheating (aimbot, wallhack, speed hack), intentional match throwing, harassment | Immediate kick. Disqualification from tournament. Reported to tournament organizer for further ban. |
| **Team-level** | Coordinated exploit abuse, refusal to follow referee instructions, stalling (not playing the objective to run out clock without engaging) | Team forfeit. Referee declares opposing team the winner. |

### Teamkill Tracking

- The server tracks teamkills per player per match
- **Auto-kick threshold:** 5 teamkills in a single match (configurable by tournament organizer)
- Referee can kick before the threshold if teamkills are clearly intentional
- Reflected damage is **not** used — it changes combat dynamics. Punishment is administrative, not mechanical.

---

## Dispute Resolution

### During a Match

1. Team captain requests a pause and states the dispute to the referee
2. Referee reviews available information (server logs, game state)
3. Referee makes a ruling
4. Ruling is final for the duration of the match — play resumes
5. Teams may file a post-match appeal if they disagree with the ruling

The referee's in-match ruling **cannot be overturned during the match**. This prevents infinite disputes from stalling play.

### Post-Match

1. The disputing team submits a written appeal to the tournament organizer within **30 minutes** of match end
2. The appeal must reference specific rules and include evidence (timestamps, screenshots, recordings)
3. The tournament organizer reviews the appeal, server logs, and any available recordings
4. The organizer may uphold, modify, or overturn the referee's ruling
5. The organizer's decision is final

---

## Spectator Mode

Referees and authorized observers (casters, analysts) use spectator mode.

### Spectator Rules

- Spectators have **full map visibility** — they can see all players, vehicles, and objectives from both teams
- Spectators **cannot interact** with the game world — no placing markers, no spawning, no chat to players
- Spectators are on a **separate communication channel** — they cannot be heard by players and players cannot hear them
- **Broadcast delay:** Tournament streams must have a minimum **90-second delay** to prevent stream sniping. The referee's spectator view has **no delay**.
- Maximum **4 spectator slots** per match (1 referee + up to 3 observers/casters). Spectator slots do not count toward team size.

### Spectator Access

- Referee: always has spectator access
- Casters: granted access by the tournament organizer before match start
- Team coaches: **not permitted** in spectator mode during the match. Coaching happens between matches only.
- Public: no spectator access. Public viewing is through the delayed broadcast only.
