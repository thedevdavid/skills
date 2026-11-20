# Prune Tests

A portable Agent Skills package for auditing existing tests individually and implementing justified deletions or consolidations.

Lives in [`thedevdavid/skills`](https://github.com/thedevdavid/skills) at `skills/prune-tests/`.

## Install

```sh
npx skills add thedevdavid/skills --skill prune-tests
```

Or manually:

```sh
mkdir -p ~/.agents/skills
git clone --depth 1 https://github.com/thedevdavid/skills.git /tmp/thedevdavid-skills
cp -R /tmp/thedevdavid-skills/skills/prune-tests ~/.agents/skills/prune-tests
```

For Claude Code:

```sh
mkdir -p ~/.claude/skills
ln -s ~/.agents/skills/prune-tests ~/.claude/skills/prune-tests
```

## Use

Invoke `$prune-tests` in Codex or `/prune-tests` in compatible hosts, and specify the repository and audit scope.

Workflow: [SKILL.md](SKILL.md). Judgment examples: [references/judgment-guide.md](references/judgment-guide.md).
