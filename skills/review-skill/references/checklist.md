# Audit checklist

Read when scoring a skill or unsure a rule applies. Portability failures are **hard fails**.

## Description / routing

- [ ] Third person; what + when; concrete triggers; ≤1024 chars
- [ ] Would select on real phrases; would not steal code review or unrelated “skill” chat
- [ ] Does not summarize a multi-step workflow so agents skip the body

## Structure / installer

- [ ] `name` matches folder; soft metadata when shipping (MIT, author, version)
- [ ] `SKILL.md` lean (~500 lines max); gotchas inline
- [ ] References one-level-deep from `SKILL.md` with “read when …” cues
- [ ] **No** tests/fixtures under the skill directory
- [ ] Scripts only if justified and portable

## Portability (hard fail)

- [ ] Plain Agent Skills package; body usable on Cursor, Codex, Claude, Grok, peers
- [ ] No required single-host APIs, MCP names, or built-in cache paths as the only procedure
- [ ] Install tips (if any) live in README / optional `agents/`, not as body lock-in

## Judgment / overlap

- [ ] Density comparable to `skills/prune-tests/` (gates, evidence, anti-filler)
- [ ] Sourced judgment — not LLM-generic best practices
- [ ] No accidental overlap with second-skills or better-upstream installs

## Failure patterns (quick)

| Pattern | Usually |
| --- | --- |
| “Helps with X” / no triggers | Description fail |
| Nested `references/a → b → c` required for happy path | Structure fail |
| “Only works in Cursor” / required Cursor MCP | **Portability hard fail** |
| Tests inside skill folder | Installer fail |
| Motivational essay, no gates | Judgment fail |
| Clone of crowded upstream / second-skills topic | Overlap non-merge |
