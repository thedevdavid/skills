# David conventions (quality bar)

Personal engineering skills in `thedevdavid/skills` encode judgment, not generic advice. Use this when Discovery asks “is this dense enough?” and when Implementation decides what to keep.

## Source of truth for density

Read `skills/prune-tests/SKILL.md` (and its judgment reference) as the bar:

- **Gates over essays** — retention/completion/verification as decision gates with required answers, not motivational sections.
- **Evidence language** — fresh vs cached vs skipped vs historical; disclose limits; do not equate green output with correct judgment.
- **Case-by-case** — shared rationales only after individual examination; group slogans fail.
- **Anti-filler** — delete instructional text the model already knows; keep corrections, invariants, and non-obvious constraints.
- **Temporary records** — decision ledgers live outside the repo unless the user/project requires a committed artifact.
- **Stop conditions** — explicit completion criteria; no polling theater; no quota-driven churn.

A new skill that cannot point to comparable gates is not ready.

## What to extract from a session

Prefer:

- Corrections (“use X not Y”, “never Z”)
- Ordering that actually mattered
- Defaults with a named escape hatch
- Boundaries: what looks in-scope but must not be done
- Failure modes the agent hit before being steered

Discard:

- Restating language/framework fundamentals
- Exhaustive option menus without a default
- Process theater (status dashboards, ceremonial checklists with no gate)
- Host-specific invocation folklore presented as the skill’s core

## Scope discipline for this monorepo

| In | Out |
| --- | --- |
| Thin engineering / meta skills with portable bodies | Product packs (capture, PARA, handoff, onboarding → second-skills) |
| Judgment that differs from commodity public skills | Clones of crowded upstream skills; link users instead |
| One coherent unit per skill | Mega SDLC / “process OS” frameworks |
| Author under `skills/<name>/` | Authoring into `~/.cursor/skills-cursor/` or other host built-in caches |

## Overlap checks

Before Design locks:

1. Does `second-skills` (or another product pack) already own this?
2. Is there a well-maintained upstream skill the user should install instead?
3. Does an existing skill in this repo already cover the when-to-use?

Overlap is a non-merge finding unless the user intentionally wants a differentiated variant — document the difference in non-goals.

## Anti-patterns that fail review

- Description without trigger terms or written in second person
- Nested reference chains required to perform the happy path
- Tests inside the skill folder
- “Works in Cursor” as a hard requirement in the body
- Prose that sounds professional but asserts no gate (“ensure high quality”, “follow best practices”)
- Manufacturing replacement coverage or scaffolding to justify keeping weak content

## After draft

Run this skill’s Verification checklist, then prefer `review-skill` (when available) for an audit with evidence cites. Feed dry-run mistakes into gotchas — that loop is how prune-tests-level density is earned.
