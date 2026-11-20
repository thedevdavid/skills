# Write Skill

A portable Agent Skills package for extracting a new skill from a real workflow or session: naming, description triggers, progressive disclosure, and a high-bar judgment layout.

Lives in [`thedevdavid/skills`](https://github.com/thedevdavid/skills) at `skills/write-skill/`.

Intended to auto-select from description triggers (“extract a skill”, “write SKILL.md”, …) on hosts that route by metadata. Optional host-only frontmatter is not required to use the package.

## Install

```sh
npx skills add thedevdavid/skills --skill write-skill
```

Or manually:

```sh
mkdir -p ~/.agents/skills
git clone --depth 1 https://github.com/thedevdavid/skills.git /tmp/thedevdavid-skills
cp -R /tmp/thedevdavid-skills/skills/write-skill ~/.agents/skills/write-skill
```

For Claude Code:

```sh
mkdir -p ~/.claude/skills
ln -s ~/.agents/skills/write-skill ~/.claude/skills/write-skill
```

Shared path `~/.agents/skills` is also the usual install target for Codex and Grok-compatible layouts.

## Use

Invoke `$write-skill` in Codex or `/write-skill` in compatible hosts when extracting or authoring a skill. Provide the source session, corrections, or artifacts to encode.

Workflow: [SKILL.md](SKILL.md). References: [description-guide.md](references/description-guide.md), [structure-checklist.md](references/structure-checklist.md), [david-conventions.md](references/david-conventions.md).
