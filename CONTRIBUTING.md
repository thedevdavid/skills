# Contributing

## Layout

Each skill is self-contained under `skills/`:

```
skills/<skill-name>/
├── SKILL.md           # Required — YAML frontmatter + instructions
├── README.md          # Optional — human install/use notes
├── agents/            # Optional — host UI metadata (e.g. Codex)
├── references/        # Optional — deep-dive docs
├── scripts/           # Optional — helpers
└── assets/            # Optional — templates/fixtures
```

`SKILL.md` frontmatter must include `name` and `description`.

## Adding a skill

1. Create `skills/<name>/` with a complete `SKILL.md`.
2. Keep long examples in `references/`.
3. List the skill in the root `README.md` catalog.
4. Open a PR against `main`.
