# CLAUDE.md — arma-comp-docs

## What This Repo Is

Documentation-only repository for the ARC (Arma Reforger Competitive) ecosystem. Contains rulesets, system designs, specs, philosophy, and tournament operations. No code.

## Repository Structure

```
/rulesets     → Competitive mode rules (player/tournament-facing)
/systems      → System design docs (roles, squads, layers, admin protocol)
/specs        → Developer-facing technical specifications (state machines, data models, event flows)
/terminology  → Canonical term definitions (glossary)
/philosophy   → Competitive design principles
/tournament   → Tournament operations, formats, procedures
/server-setup → Server installation & configuration guides (planned)
```

## Document Audiences

- **Rulesets** — Players and tournament organizers. Simple language, enforceable rules, no implementation details.
- **Systems** — Designers and developers. How systems are designed, why decisions were made, cross-cutting effects.
- **Specs** — Developers only. State machines, data models, pseudocode, event flows. Enough detail to implement without guessing.
- **Terminology** — Everyone. Single source of truth for all terms.
- **Philosophy** — Everyone. The "why" behind design decisions.
- **Tournament** — Tournament organizers. Operational procedures, formats, conduct rules.

## Writing Conventions

- Use specific numbers, never vague language ("90 seconds", not "a reasonable amount of time")
- Every number must have a rationale — if you can't explain why, question the number
- Rules must be enforceable by the server. If a server can't enforce it, it belongs in tournament conduct rules, not the ruleset.
- Use terms exactly as defined in `terminology/glossary.md`. If a new term is needed, add it to the glossary first.
- Cross-reference other documents by relative path (e.g., "see `systems/squads.md`")
- No emojis in documents

## Versioning

- Rulesets use semantic versioning: MAJOR.MINOR.PATCH
  - MAJOR: Rule changes affecting match outcomes
  - MINOR: Clarifications, new sections
  - PATCH: Typos, formatting
- All documents are living drafts — propose changes with rationale

## Naming Conventions

- File names: lowercase, hyphens for spaces (e.g., `roles-and-classes.md`)
- Directory names: lowercase, no hyphens (e.g., `rulesets`, `systems`, `specs`)
- Layer IDs: `{map}-{mode}-{variant}-v{version}` (e.g., `arland-aas-standard-v1`)

## Key Terminology

- MSP = Mobile Spawn Point (NOT MCU)
- SL = Squad Leader
- AT = Anti-Tank
- MG = Machine Gunner
- AAS = Advance and Secure
- ARC = Arma Reforger Competitive

## Commit Messages

Format: `[area] Brief description`

Areas: `ruleset`, `system`, `spec`, `terminology`, `philosophy`, `tournament`, `meta`

Examples:
- `[ruleset] Add disconnect handling to AAS v1`
- `[spec] Define flag capture decay formula`
- `[terminology] Add MSP and HUB definitions`
- `[meta] Update README structure`
