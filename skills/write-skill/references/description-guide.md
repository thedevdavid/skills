# Description guide

The `description` field is the routing key. Many hosts load only `name` + `description` until the skill activates. A weak description means the skill never loads — or steals unrelated turns.

## Required shape

Write in **third person**. Cover **what** the skill does and **when** to use it. Include concrete **trigger terms** a user or agent would actually say.

| Constraint | Rule |
| --- | --- |
| Length | 1–1024 characters; prefer one tight paragraph |
| Person | Third person (“Guides…”, “Audits…” — not “I help you…”) |
| What | Observable outcome or procedure the skill owns |
| When | Situations, verbs, and nouns that should activate it |
| Anti-vague | Ban “helps with X”, “best practices”, “various tasks” |

## Pattern

```text
<what it does, including key verbs/objects>. Use when <situation A>, <situation B>, or <situation C>.
```

Good (this skill):

> Guides creation of a new Agent Skill from a real workflow or session. Use when extracting a skill, writing SKILL.md, naming or describing a skill, structuring references/, or turning corrections into durable agent instructions.

Good (prune-tests):

> Audit existing test suites case by case and delete or consolidate low-value checks. Use for deliberate test-value audits or suite pruning, including orphan cleanup; honor read-only requests.

Bad:

> Helps you write better skills.
> A comprehensive guide to Agent Skills best practices for any project.

## Trigger stress test

Before locking Design, test mentally (or with sample prompts):

1. **Should fire** — “extract a skill from this session”, “write SKILL.md for X”, “name and describe this skill”, “structure references for a new skill”.
2. **Must not steal** — ordinary code review, test pruning, “skill” used as synonym for ability, product onboarding unrelated to Agent Skills packages.
3. **Boundary** — reviewing an existing skill folder is `review-skill` once it exists; writing/extracting is this skill. Keep verbs distinct in both descriptions.

If false positives are likely, tighten when-clause nouns (SKILL.md, references/, Agent Skill) rather than lengthening with adjectives.

## After edits

Re-run the stress test whenever the skill’s purpose or non-goals change. Description drift is a common regression: body evolves, description stays generic.
