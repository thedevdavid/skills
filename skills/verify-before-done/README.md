# Verify Before Done

Portable Agent Skills package that gates “done” on labeled acceptance evidence (fresh / cached / skipped / historical) and stops unnecessary polling.

## Install

```sh
npx skills add thedevdavid/skills --skill verify-before-done
```

Or copy into `~/.agents/skills/verify-before-done` (shared by Codex / Cursor / Grok-compatible layouts). Claude Code: symlink or copy under `~/.claude/skills/`.

## Use

`$verify-before-done` / `/verify-before-done` before claiming done, green tests/CI, or finishing a PR. See [SKILL.md](SKILL.md).
