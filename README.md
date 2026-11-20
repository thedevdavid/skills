# Skills

Personal Agent Skills collection for Cursor, Claude Code, Codex, and other Agent Skills–compatible hosts.

Layout follows the Agent Skills monorepo convention (`skills/<name>/SKILL.md`), as used by collections like [anthropics/skills](https://github.com/anthropics/skills).

This repo is the **personal engineering skill monorepo**. Product packs (capture, PARA, handoff, onboarding) live in a separate repo: [thedevdavid/second-skills](https://github.com/thedevdavid/second-skills).

## Catalog

| Skill | Description |
|-------|-------------|
| [prune-tests](skills/prune-tests/) | Audit existing test suites case by case and delete or consolidate low-value checks |

## Install

### Skills CLI

```sh
npx skills add thedevdavid/skills
# or a single skill:
npx skills add thedevdavid/skills --skill prune-tests
```

### Manual

| Host | Typical path |
|------|----------------|
| Shared (Codex / Cursor / Grok) | `~/.agents/skills/<skill-name>/` |
| Cursor (project) | `.cursor/skills/<skill-name>/` |
| Claude Code | `~/.claude/skills/<skill-name>/` |

```sh
mkdir -p ~/.agents/skills
git clone --depth 1 https://github.com/thedevdavid/skills.git /tmp/thedevdavid-skills
cp -R /tmp/thedevdavid-skills/skills/prune-tests ~/.agents/skills/prune-tests
```

## Repository layout

```
skills/
└── prune-tests/
    ├── SKILL.md
    ├── README.md
    ├── agents/
    └── references/
```

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE).
