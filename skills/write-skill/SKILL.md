---
name: write-skill
description: "Guides creation of a new Agent Skill from a real workflow or session. Use when extracting a skill, writing SKILL.md, naming or describing a skill, structuring references/, or turning corrections into durable agent instructions."
license: MIT
metadata:
  author: thedevdavid
  version: "1.1"
---

# Write Skill

Turn real expertise into a portable Agent Skills package (`SKILL.md` + optional `references/`, `scripts/`, README). Do not invent from model priors alone.

Density bar: `skills/prune-tests/` — gates and gotchas, not essays. Cut prose the agent already knows.

## 1. Discover

Require source: a steered session/corrections, project artifacts that encode judgment, or an explicit user statement. If missing, stop and ask.

Lock purpose, when-to-use (trigger phrases), non-goals, and overlap (this monorepo, product packs like second-skills, better upstream installs). Ask the user only when a choice blocks Design.

## 2. Design

- **`name`**: kebab-case; equals folder; ≤64 chars; `[a-z0-9-]` only.
- **`description`**: third person; what + when; concrete triggers; ≤1024 chars. Routing key — vague slogans fail. Prefer pushy when-clauses so hosts that under-trigger still select it; add Skip-when if false positives are likely.
- **Outline**: what stays in `SKILL.md` vs `references/` with “read when …” cues.
- **Path**: author under `skills/<name>/`. Never into host built-in skill caches.

Reject mega process frameworks, commodity upstream clones, and product-pack content.

## 3. Implement

```
skills/<name>/
├── SKILL.md
├── README.md              # thin multi-host install
├── agents/openai.yaml     # optional
├── references/…           # only if needed; one level deep
└── scripts/…              # only if reinvented across runs
```

Body rules: imperative procedures; gotchas inline; under ~500 lines; host-agnostic (no required Cursor-only APIs); soft metadata `license: MIT`, `metadata.author: thedevdavid`, `metadata.version`; **no tests/fixtures** in the skill folder. Update the root README catalog.

Temporary decision notes stay outside the repo unless required.

## 4. Verify

Before done: description would select on the right prompts and skip unrelated ones; frontmatter valid; refs one-level with load cues; portable package; no overlap surprises; density matches prune-tests; prefer one real-task dry run and fold mistakes into gotchas.

Read [checklist.md](references/checklist.md) when shipping or unsure a rule applies. Hand off to `review-skill` when it exists.
