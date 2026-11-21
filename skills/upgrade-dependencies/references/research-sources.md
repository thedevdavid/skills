# Research sources

Read when choosing where to look for from→to changes, classifying severity, finding codemods, or skimming advisories.

## Priority order (per package)

1. Official migration / upgrade guide for the major (or named “Migrating to vN”).
2. `CHANGELOG.md` / release notes for **every intermediate** version between from and to.
3. GitHub/GitLab Releases and compare view (`/compare/vFrom...vTo`).
4. Official docs “breaking changes” / upgrade pages.
5. Maintainer blog posts or announced migration threads linked from releases.
6. Codemod / migrate CLI docs shipped by the same project.
7. Advisory databases for the crossed range (GitHub Advisories, OSV, `npm audit` / ecosystem equivalent) — skim only.

Do not stop at the endpoint release. A 3.1 → 3.4 jump can break in 3.2. Never omit a required/breaking/behavioral item because it looks small.

## What to extract

Severities must match the ledger 1:1:

| Severity | Capture |
| --- | --- |
| breaking | Removed/renamed exports, signature changes, import path moves, deprecations that now error, hard minimum runtime/tooling |
| behavioral | Default flips, changed return shapes that still typecheck, timing/retry/error-mapping shifts |
| required-config | Env keys, feature flags, bundler plugin names, peer ranges the app must satisfy |
| optional-feature | New APIs or patterns that are not needed to compile/run as before |
| advisory | ID, severity, whether to-version remediates, link |

## Codemod discovery

Search the migration guide and repo for `codemod`, `migrate`, `jscodeshift`, `rsx`, `eslint --fix`, official CLIs (`npx <pkg> migrate`, framework upgrade tools). Record the exact command and supported from→to. Prefer the official codemod; note manual follow-ups it does not cover.

## Skill / plugin packages

For Agent Skills and editor plugins: read the skill’s own changelog or release notes when present; otherwise diff `SKILL.md` / package manifests between versions and treat instruction or interface changes as required if consumers depend on them.

## Gotchas

- **No changelog:** use Releases compare + public API/docs diff; remaining gaps stay `unknown` on the ledger — do not invent API shapes.
- **`@types/*` and peers:** bump DefinitelyTyped / peer packages when the runtime major requires it; record as `required-config` or `breaking` as sources say.
- **Lockfile-only shifts:** a transitive major with no manifest edit is still in scope — research it like a direct bump.
- **Unreachable sources:** if guides/compare views cannot be fetched, mark rows `unknown` and block unqualified completion.
- **Workspace / catalog pins:** resolve from→to from the workspace protocol, catalog entry, or shared lock — do not flatten `workspace:` / `catalog:` to a registry version in the plan’s provenance table.

## Anti-patterns

- Relying on training memory for API shape
- Reading only the latest release notes when multiple versions were crossed
- Treating “miss” in the codebase as license to skip documenting the upstream break
- Expanding into supply-chain policy setup (already skipped in the skill description)
