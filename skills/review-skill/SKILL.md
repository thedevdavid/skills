---
name: review-skill
description: "Audits an Agent Skill for description triggers, SKILL.md length, progressive disclosure, installer safety (no tests inside the skill directory), portability, and judgment quality. Use when reviewing a skill PR, tightening SKILL.md, or checking portability before publishing."
license: MIT
metadata:
  author: thedevdavid
  version: "1.0"
---

# Review Skill

Audit an existing skill folder. Default output: structured findings + recommended edits. Do **not** rewrite the skill unless asked. Ordinary code/diff review is out of scope (that is `review-diff` when it exists); authoring a new skill is `write-skill`.

Density bar: `skills/prune-tests/` and `skills/write-skill/`. Challenge “sounds professional” filler the same way prune-tests challenges mock echoes.

## 1. Resolve

Accept a skill name, path to `SKILL.md`, or skill directory. If empty, ask which skill. Confirm `SKILL.md` exists; read frontmatter, body, and one-level companions actually linked.

## 2. Audit

Score each item **pass / fail / unresolved** with evidence (path + line cites). Vibes are not findings.

| Axis | Fail when |
| --- | --- |
| Description | Missing what+when, no concrete triggers, vague slogans, or would steal unrelated tasks |
| Structure | Body ≫~500 lines without disclosure; nested refs; missing load cues; tests/fixtures inside skill dir |
| **Portability** | **Hard fail:** required Cursor-only (or any single-host) APIs/paths/tools; install that only works on one product; body assumes one host |
| Judgment | No gates/gotchas; generic best-practices essay; density far below prune-tests |
| Overlap | Duplicates `second-skills` / commodity upstream without intentional differentiation — **non-merge** unless user accepts |

Unresolved only for a concrete missing fact that could change the verdict.

Read [checklist.md](references/checklist.md) for the binary list and failure patterns.

## 3. Report

Emit:

```text
VERDICT: pass | fail | unresolved
SUMMARY: <one line>

## Findings
- [fail|pass|unresolved] <axis>: <evidence cite> — <why> — <recommended edit>

## Clean
- <specific rule that passed, with cite>
```

Empty findings on a true pass still need a populated Clean section. Do not invent nits to look thorough. Portability fail ⇒ overall **fail** even if other axes pass.
