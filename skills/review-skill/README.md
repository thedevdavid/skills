# Review Skill

Portable Agent Skills package for auditing an existing skill’s triggers, structure, installer safety, portability, and judgment density.

## Install

```sh
npx skills add thedevdavid/skills --skill review-skill
```

Or copy into `~/.agents/skills/review-skill` (shared by Codex / Cursor / Grok-compatible layouts). Claude Code: symlink or copy under `~/.claude/skills/`.

## Use

`$review-skill` / `/review-skill` with a skill name or path. Default is audit + recommended edits — not a rewrite. See [SKILL.md](SKILL.md).
