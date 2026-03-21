# Tournament Operations Guide

Defines the operational structure for running ARC competitive tournaments. Covers scheduling, check-in, layer selection, match execution, and dispute handling.

This guide supplements the AAS v1 ruleset — it does not override it.

---

## Tournament Roles

| Role | Responsibility |
|------|---------------|
| **Tournament Organizer (TO)** | Owns the tournament. Sets format, schedule, rules supplements. Final authority on appeals. |
| **Server Admin** | Operates the server infrastructure. Loads layers, manages crashes, provides logs. |
| **Referee** | Enforces rules during matches. See `systems/admin-and-referee.md`. |
| **Caster** | Broadcasts matches with commentary. Operates on delayed spectator feed. |
| **Team Captain** | Represents the team in all official communications. Point of contact for the TO and referee. |

One person may hold multiple roles (e.g., TO + Server Admin) but a referee **must not** be a player in the match they officiate.

---

## Tournament Formats

### Recommended Formats

| Format | Best For | Structure |
|--------|----------|-----------|
| **Single Elimination** | Quick events, 4-8 teams | Loss = out. Fast, high stakes. |
| **Double Elimination** | Standard tournaments, 8-16 teams | Two brackets (winners/losers). Must lose twice to be eliminated. |
| **Round Robin** | Leagues, 4-6 teams | Every team plays every other team. Most matches, fairest seeding. |
| **Swiss** | Large fields, 8-32 teams | Teams with similar records play each other. Efficient ranking without full round robin. |

### Match Series

- **Group stage:** Single match (best of 1)
- **Playoffs:** Best of 1 (single elimination) or best of 3 (double elimination finals)
- **Grand final:** Best of 3

In best-of-3 series:
- Layer veto is performed once before the series. Each team bans, then picks, alternating.
- Teams **switch sides** (faction) between matches on the same layer.
- If a third match is needed, the remaining layer from the veto pool is played. Side selection is given to the team with fewer total tickets lost across the first two matches.

---

## Registration

### Team Requirements

- **Roster size:** Minimum 24 players, maximum 30 (24 starters + up to 6 substitutes)
- **Team name:** Unique, non-offensive. TO reserves the right to reject names.
- **Captain:** One designated captain per team, registered by name
- **Roster lock:** Rosters are locked **24 hours before** the team's first match. No changes after lock except for emergency substitutions approved by the TO.

### Emergency Substitution

If a rostered player cannot attend after roster lock:
1. Captain notifies the TO at least **1 hour before** match time
2. TO approves a substitute from the team's registered substitute pool
3. The substitute must already be on the team's 30-player roster
4. Maximum **3 emergency substitutions** per tournament

---

## Match Day Flow

### Pre-Match (30 minutes before)

| Time | Action |
|------|--------|
| **T-30 min** | Server is online. Layer is loaded. Referee and casters connect. |
| **T-15 min** | Team check-in opens. Captains confirm roster in the tournament channel. |
| **T-10 min** | Check-in deadline. Teams that haven't checked in receive a warning. |
| **T-5 min** | Final warning. Unchecked teams have 5 minutes before forfeit. |
| **T-0** | Match starts. Players connect. Warm-up phase begins. |

### Check-In Rules

- Both captains must confirm their team is ready in the designated tournament communication channel
- A team that fails to check in by **T-5 min** forfeits the match
- If both teams fail to check in, the TO decides: reschedule or double forfeit
- A team may check in with fewer than 24 players only if the opposing captain agrees. If the opponent does not agree, the short team has until T-0 to fill their roster or forfeit.

### During the Match

1. Warm-up begins (60 seconds)
2. Live match plays out per ruleset
3. Referee monitors via spectator mode
4. Pauses handled per `systems/admin-and-referee.md`
5. Match ends via win condition
6. Referee confirms result and reports to TO

### Post-Match

| Action | Deadline |
|--------|----------|
| Referee submits match result to TO | Immediately after match end |
| Dispute window opens | Match end |
| Dispute window closes | **30 minutes** after match end |
| TO publishes result | After dispute window closes (or after dispute resolution) |

---

## Layer Selection & Veto

### Layer Pool

The TO defines a **layer pool** for the tournament before registration opens. The pool must:
- Contain an **odd number** of layers (minimum 3, recommended 5 or 7)
- Include only layers compatible with the tournament's core mod version
- Be published and available for teams to review before registration

### Veto Process (Best of 1)

1. Coin flip determines which team vetoes first (Team A)
2. **Team A bans** one layer from the pool
3. **Team B bans** one layer from the pool
4. Repeat until **one layer remains** — that layer is played
5. The team that did **not** make the last ban chooses their starting side (faction)

### Veto Process (Best of 3)

1. Coin flip determines which team acts first (Team A)
2. **Team A bans** one layer
3. **Team B bans** one layer
4. **Team A picks** one layer (this is Match 1 — Team B chooses side)
5. **Team B picks** one layer (this is Match 2 — Team A chooses side)
6. If a Match 3 is needed, the remaining unpicked/unbanned layer is played. Side selection goes to the team with fewer total tickets lost across Matches 1 and 2.

### Veto Timing

- Each team has **60 seconds** per ban/pick decision
- If a team exceeds the timer, the TO or referee makes a random selection on their behalf
- Veto is conducted in a private channel with both captains and the TO/referee present

---

## Server Configuration

### Required Settings

| Setting | Value | Notes |
|---------|-------|-------|
| **ARC Core mod** | Required, version matching layer compatibility | Must be loaded before any layer mod |
| **Tick rate** | Server default (highest stable) | Do not reduce for performance — reschedule if hardware can't handle it |
| **Max players** | Team size x2 + spectator slots (e.g., 52 for 24v24 + 4 spec) | |
| **Password** | Required | Prevents unauthorized joins |
| **Auto-kick idle** | Disabled during tournament | Prevents accidental kicks during pauses |
| **Friendly fire** | Enabled | Required by ruleset. TK tracking handles abuse. |

### Pre-Match Server Checklist

- [ ] Correct ARC Core version loaded
- [ ] Correct layer mod loaded
- [ ] Correct layer selected
- [ ] Server password set and distributed to teams and officials only
- [ ] Spectator slots reserved (not counted as player slots)
- [ ] Server logging enabled (required for dispute resolution)
- [ ] Test connection verified by referee

---

## Conduct Rules

These are **tournament-level** rules administered by humans. They supplement the server-enforced rules in the AAS v1 ruleset.

### Player Conduct

- Treat all players, opponents, officials, and casters with respect
- No hate speech, slurs, or targeted harassment — in-game, in voice, or in tournament channels
- No impersonation of officials or other players
- No discussion of ongoing match details in public channels until the match is complete

### Team Conduct

- Captains are responsible for their team's behavior
- A team whose players accumulate **3 or more kicks** across a tournament is subject to disqualification review by the TO
- Teams must play to win — intentional match throwing (e.g., to manipulate bracket seeding) results in disqualification

### Penalties

| Offense | Penalty |
|---------|---------|
| First conduct violation (minor) | Warning to player via captain |
| Second conduct violation | Player banned for next match |
| Third conduct violation | Player banned from tournament |
| Team-level conduct violation | TO review — ranges from match forfeit to tournament disqualification |
| Cheating (confirmed) | Immediate player ban from tournament. Team disqualified if evidence suggests team knowledge. |

---

## Recording & Evidence

### Match Recording

- The **server log** is the primary official record of every match
- Tournament organizers **should** record all matches from spectator view (referee or caster POV)
- Teams are encouraged but not required to record their own POV
- All recordings used as evidence in disputes must be **unedited** and include timestamps

### Evidence Standards for Disputes

| Evidence Type | Weight |
|---------------|--------|
| Server logs (timestamps, events, commands) | Highest — server-authoritative, tamper-proof |
| Referee spectator recording | High — official perspective |
| Caster broadcast recording (with delay) | Medium — delayed, may miss context |
| Player POV recording | Medium — subjective, but useful for intent |
| Screenshots | Low — lack context, easily manipulated |
| Testimony without evidence | Not accepted on its own |

---

## Results & Standings

### Match Result Format

```
Match: [Tournament Name] - [Round/Stage]
Layer: [Layer ID]
Date:  [YYYY-MM-DD HH:MM UTC]

Team 1: [Name] — [Faction]
Team 2: [Name] — [Faction]

Result: [Team Name] wins
  Final tickets: [Team 1 tickets] - [Team 2 tickets]
  Win condition:  [tickets/flag/time/forfeit]
  Duration:       [MM:SS]

Referee: [Name]
Disputes: [None / Pending / Resolved — summary]
```

### Standings Calculation (Round Robin / Swiss)

1. **Match wins** (primary)
2. **Ticket differential** (total tickets remaining minus tickets lost across all matches — tiebreaker)
3. **Head-to-head result** (if two teams are tied on wins and ticket differential)
4. **Coin flip** (absolute last resort)
