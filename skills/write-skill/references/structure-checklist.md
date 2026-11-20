# Structure checklist

Use while implementing or before Verification. Binary where possible.

## Package shape

- [ ] Folder is `skills/<name>/` with `name` matching directory
- [ ] `SKILL.md` present with YAML frontmatter (`name`, `description`)
- [ ] Soft metadata when shipping: `license: MIT`, `metadata.author`, `metadata.version`
- [ ] Optional: `README.md` with multi-host install (`npx skills`, `~/.agents/skills`, Claude/Codex/Grok notes)
- [ ] Optional: `agents/openai.yaml` for Codex UI only — not required to run
- [ ] Optional: `references/` and/or `scripts/` as justified below
- [ ] **No** `tests/`, fixtures-as-evals, or sample corpora inside the skill directory

## Progressive disclosure

- [ ] `SKILL.md` body under ~500 lines (and prefer well under 5k tokens of core instructions)
- [ ] Deep examples, long matrices, and rare edge cases live in `references/`
- [ ] Every reference link is **one level deep** from `SKILL.md` (no reference→reference chains required to operate)
- [ ] Each reference has a **load cue** in the body (“Read X when …”), not a bare “see references/”
- [ ] Gotchas that the agent will hit without warning stay **inline** in `SKILL.md`

## Scripts policy

- [ ] No `scripts/` unless agents reinvented the same logic across real runs
- [ ] Scripts are portable (documented deps; shell/Python preferred); no host-private binaries required
- [ ] Skill still useful if scripts are unavailable (prose fallback or clear failure)

## Portability

- [ ] Body instructions make sense without Cursor-only tools, paths, or MCP names as requirements
- [ ] Host-specific install or UI tips live in README / `agents/`, not as the only procedure
- [ ] No required marketplace wrapper, plugin manifest, or single-host packaging to use the skill
- [ ] Compatible with Agent Skills directory layout (`SKILL.md` + optional refs/scripts/assets)

## Monorepo hygiene

- [ ] Root README catalog row added/updated
- [ ] Does not write into built-in host skill caches as the source of truth
- [ ] MIT license consistent with repo
- [ ] Overlap with product packs / upstream commodity skills checked and disclosed

## Done when

All boxes pass or failures are explicitly waived by the user with rationale. Do not merge with silent checklist gaps.
