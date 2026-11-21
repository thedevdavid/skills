# Ecosystem commands

Read when detecting manifests/lockfiles, performing bumps, mapping install/gate commands, running frozen verify, resolving peers, or choosing full-auto discover order. Prefer the repo’s scripts and CI over generic defaults.

## Package managers

| Ecosystem | Manifest / lock (from-version) | Bump / install (typical) | Frozen / immutable verify (after bumps) |
| --- | --- | --- | --- |
| npm | `package.json` / `package-lock.json` | edit manifest; `npm install` / `npm ci` after pin | `npm ci` |
| pnpm | `package.json` / `pnpm-lock.yaml` | `pnpm up <pkg>@<ver>` or edit + `pnpm install` | `pnpm install --frozen-lockfile` |
| yarn | `package.json` / `yarn.lock` | `yarn up <pkg>@<ver>` (Berry) or edit + install | `yarn install --immutable` |
| bun | `package.json` / `bun.lock` | `bun add <pkg>@<ver>` | `bun install --frozen-lockfile` (not default in CI — pass explicitly) |
| Python | `pyproject.toml` / `uv.lock` / `poetry.lock` / `requirements*.txt` | `uv lock` / `poetry add` / edit requirements + install | `uv lock --check` / `uv sync --locked` (or Poetry equivalent) |
| Rust | `Cargo.toml` / `Cargo.lock` | edit + `cargo update -p <pkg> --precise <ver>` | `cargo … --locked` |
| Go | `go.mod` / `go.sum` | `go get <module>@<ver>` | `go mod verify` (+ tidy diff if the project uses it) |
| Ruby | `Gemfile` / `Gemfile.lock` | edit + `bundle update <gem>` | `bundle lock --check` / CI install with frozen bundle |
| PHP | `composer.json` / `composer.lock` | `composer require <pkg>:<ver>` | `composer install --no-update` / lock-committed CI install |
| JVM | `pom.xml` / Gradle files | edit coordinates; use project wrapper | project wrapper with lockfile / dependency verification if present |

Dry-run when available (`npm install --dry-run`, `cargo update --dry-run`, `poetry update --dry-run`). Do not hand-edit lockfiles. Confirm the lock pin equals the intended **to** after install, then run the frozen/immutable verify for that ecosystem before claiming CI-safe.

## Full-auto discover ladder

Prefer this order unless the user named exact targets:

1. **Outdated** — ecosystem outdated/list command (`npm outdated`, `pnpm outdated`, `yarn up --mode=report`, etc.).
2. **In-range** — refresh lock within existing manifest ranges when that is the ask.
3. **Major** — intentional manifest range / major bump + full research → ledger path.

Do not use blind mass `--latest` / `ncu -u` as the strategy. Separate in-range lock refresh from intentional majors.

## Workspace / catalog gotchas

- **Yarn Berry `up`:** often project-wide; confirm which workspaces change before treating the bump set as single-package.
- **pnpm:** use `-r` / `--filter` deliberately; catalogs rewrite catalog entries — regenerate the shared lock, do not hand-flatten.
- **Do not flatten** `workspace:` or `catalog:` protocol pins to bare registry versions in manifests unless that is an explicit, researched migration.
- Prefer the repo’s existing workspace tooling over inventing a root-only upgrade path.

## Peer conflicts

Playbook (in order):

1. **Upgrade constraints** — bump the package that declares the peer / the peer itself so ranges satisfy.
2. **Intentional overrides** — `overrides` / `resolutions` / pnpm peer overrides only when documented and accepted; record on the ledger.
3. **Risk-accepted mutes** — `legacy-peer-deps`, warning-only peer rules, etc. only with explicit user acceptance.

**Mute ≠ fixed.** Warning suppression does not close a peer contract. If the lock was shaped with install flags, CI/frozen install must use the **same** flags or the verify is invalid.

## Agent Skills / plugins

Detect bumps via:

- Skill folders under `skills/<name>/`, `~/.agents/skills/`, host skill paths, or plugin skill trees
- Package manifests that vend skills (`package.json` of a skills plugin, marketplace lock, `skills-lock` / similar if present)
- `git diff` on `SKILL.md`, skill `package.json`, or plugin version fields

**From/to:** version field in the skill package metadata when present; otherwise commit/tag or content hash before vs after. Research the skill changelog or diff instructions consumers rely on.

## Optional broader pins (only if in scope)

| Kind | Typical files |
| --- | --- |
| Runtime | `.nvmrc`, `.node-version`, `engines`, `.python-version`, `rust-toolchain.toml` |
| Docker | `Dockerfile`, `*.dockerfile`, compose `image:` |
| CI actions | `.github/workflows/*.yml` `uses: org/action@vN` |

Same ledger rules: research required migrations for the crossed range; do not expand into these unprompted.

## Gates

Discover from `package.json` scripts, `Makefile`, `justfile`, or CI. Common pattern: frozen install verify → build → lint → test. Hand off labeling to `verify-before-done` (if available); do not invent alternate runners.
