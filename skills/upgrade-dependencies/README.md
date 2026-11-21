# Upgrade Dependencies

Portable Agent Skills package for researching dependency bumps and applying **required** migrations (packages + Agent Skills/plugins), with optional new features listed as recommendations.

Lives in [`thedevdavid/skills`](https://github.com/thedevdavid/skills) at `skills/upgrade-dependencies/`.

## Install

```sh
npx skills add thedevdavid/skills --skill upgrade-dependencies
```

Or manually:

```sh
mkdir -p ~/.agents/skills
git clone --depth 1 https://github.com/thedevdavid/skills.git /tmp/thedevdavid-skills
cp -R /tmp/thedevdavid-skills/skills/upgrade-dependencies ~/.agents/skills/upgrade-dependencies
```

For Claude Code:

```sh
mkdir -p ~/.claude/skills
ln -s ~/.agents/skills/upgrade-dependencies ~/.claude/skills/upgrade-dependencies
```

## Use

Invoke `$upgrade-dependencies` in Codex or `/upgrade-dependencies` in compatible hosts. Say whether bumps are already in the tree (semi-auto) or should be performed (full-auto), and name the packages or skill scope.

Workflow: [SKILL.md](SKILL.md). Sources: [references/research-sources.md](references/research-sources.md). Plan/ledger/PR skeleton: [references/plan-template.md](references/plan-template.md). Commands: [references/ecosystem-commands.md](references/ecosystem-commands.md). Framework sequencing: [references/framework-upgrade-patterns.md](references/framework-upgrade-patterns.md).

## Not in v1

Renovate/Dependabot **triage-only** (review/group/merge bot PRs with no migration work), batch “patch Friday” with bisect-on-red, baked-in framework compatibility matrices, and supply-chain hardening (Socket, ignore-scripts, bot security presets) are out of scope. Use this skill when a bump needs researched call-site / config migration.
