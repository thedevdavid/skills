---
name: upgrade-dependencies
description: "Researches and applies required migrations for dependency and Agent Skill version bumps. Use when upgrading packages, lockfiles, majors, or Agent Skills/plugin packages. Skip when the request is supply-chain hardening only (Socket, ignore-scripts, Renovate security presets), or Renovate/Dependabot triage-only with no migration."
license: MIT
metadata:
  author: thedevdavid
  version: "1.0"
---

# Upgrade Dependencies

Turn version bumps into a researched, ledger-backed migration. Detect or perform bumps, research every required change across releases and guides, implement **required** migrations, and list optional new features as recommendations until the user opts in.

Default scope: package-manager dependencies **and** Agent Skills / plugin skill packages. Language runtimes, Docker base images, and CI action pins only when the user asks or they clearly belong to the requested bump set. Do not configure supply-chain tooling.

## Modes

| Mode | When | Bump step |
| --- | --- | --- |
| **Semi-auto** | Bumps expected from user/bot/PR, or already in the tree | Detect from→to in worktree, manifests, lockfiles, and skill packages; **do not invent bumps** |
| **Full-auto** | User asks to upgrade named packages or “latest” / a target | Order: pre-pin → bump/resolve → research → ledger → plan → implement required |

**Semi-auto wait:** If no version diffs are visible yet, poll at most ~3 times over ~10–15 minutes for diffs under manifests, lockfiles, and skill paths only. React only to those diffs; never invent bumps or switch to full-auto while waiting. If the bound expires with still no bumps, stop and ask whether to full-auto or widen detection.

If mode is unclear, prefer semi-auto when the worktree already shows version diffs; otherwise ask once. After the plan exists, **implement every required ledger row** unless the user limited the run to plan-only. Optional new functionality stays recommendation-only until explicit yes.

**Fail-closed:** If a required major (or tightly coupled group) fails mid full-auto after partial bumps, halt. Mark ledger rows `blocked`, leave the tree in a disclosed state, and prefer reverting that bump set if the tree is red. Do not skip-and-continue remaining required majors or treat optional deferrals as covering required unknowns.

## Establish scope and provenance

Use the requested packages, PR, or worktree diff. Discover the ecosystem(s), workspace boundaries, and maintained install/build/test commands. Read [ecosystem-commands.md](references/ecosystem-commands.md) when mapping manifests, lockfiles, skill packages, install commands, frozen verify, peer conflicts, or full-auto discover order.

Record **from** and **to** from the lockfile (or skill package pin/manifest), not a loose manifest range. For semi-auto, derive pairs from `git diff` (and status) of manifests, lockfiles, and skill folders/packages. For full-auto, record pre-bump lock pins, then target versions after resolution. Treat unexpected **transitive major** lockfile shifts as first-class research targets.

Confirm baseline acceptance when practical (project build/lint/test). A red baseline is a disclosed bound: later failures may not be attributable to the upgrade. Respect read-only / plan-only requests and execution permissions.

Keep the temporary ledger and plan outside the repository unless the user or project requires a committed artifact.

## Research every required change

For each direct target and each transitive major in scope, research the path from→to — **every intermediate release**, not only the endpoints. Prefer live sources over memory: GitHub/GitLab releases and compare views, official docs, migration/upgrade guides, maintainer blogs linked from releases, and changelogs/release notes. Read [research-sources.md](references/research-sources.md) when choosing sources, classifying severity, finding codemods, or skimming advisories. When Next.js, React, TypeScript, or ESLint are in scope, read [framework-upgrade-patterns.md](references/framework-upgrade-patterns.md) before planning.

Extract and classify using the same severities as the ledger:

1. **breaking** — removed or renamed APIs, signature or import path changes, deprecations that now error, hard minimum runtime/tooling the project must meet.
2. **behavioral** — same compile surface, different runtime defaults or results (highest risk; always ledger).
3. **required-config** — env keys, feature flags, bundler plugin names, peer ranges the app must satisfy.
4. **optional-feature** — additive APIs or patterns not required for compatibility.
5. **advisory** — CVEs/GHSAs in the crossed version range (skim; record IDs and whether the bump remediates).

Discover official **codemods** or migrate CLIs; prefer them over hand edits when they match the from→to range. Peer conflicts: upgrade constraining packages first, then intentional overrides, then risk-accepted mutes only with user acceptance — mute ≠ fixed. Do not `--force` / legacy-peer-deps unless accepted. Halt and report irresolvable constraint chains. See [ecosystem-commands.md](references/ecosystem-commands.md).

## Completeness ledger (never skip required)

Every `breaking`, `behavioral`, and `required-config` item from sources gets a row. Nothing “small” may be omitted. Shared rationale is allowed only after individual examination.

| Column | Content |
| --- | --- |
| ID | Stable short id |
| Package | Name (+ ecosystem) |
| From → To | Provenance versions |
| Source | URL or path + section |
| Severity | breaking / behavioral / required-config / advisory / optional-feature |
| Codebase | hit / miss / unknown (with grep or path evidence) |
| Action | migrate / codemod / config / test-audit / defer-optional / n/a-miss |
| Status | open / done / blocked / deferred-optional |

**Completion gate:** no open `breaking`, `behavioral`, or `required-config` rows with `hit` or `unknown`. `miss` still needs an explicit “no call sites” note. `unknown` blocks unqualified completion. Optional-feature rows stay `deferred-optional` unless the user opted in.

## Plan

Before editing application code (full-auto may already have bumped manifests), produce a plan from [plan-template.md](references/plan-template.md). Sequence into verifiable units: after each major or tightly coupled group, run relevant project checks before the next. Converge on required compatibility; do not keep throwaway shims for a smoother intermediate.

## Implement required only

Execute open required ledger rows. Prefer upstream codemods, then mechanical call-site updates from documented APIs. Grep for every changed symbol/config key before claiming a row done. Migrate callers and remove obsolete usage in the same wave when the old API is internal to this repo.

Keep the diff to what the upgrade requires. Do not adopt optional new features, drive-by refactors, or reformatting. If a build error is not in the ledger, return to research for that symbol — do not invent an API shape.

Skill/plugin packages: apply required instruction or API migrations inside consuming projects; do not silently rewrite upstream skill sources unless that repo is the workspace.

## Verify

After bumps, run the ecosystem’s **frozen/immutable** install check before claiming the lock is CI-safe (see [ecosystem-commands.md](references/ecosystem-commands.md); Bun needs an explicit `--frozen-lockfile`). Then hand off to `verify-before-done` (if available): scope the acceptance claim, run maintained project gates once per relevant change, label evidence fresh/cached/skipped/historical, and stop polling once evidence is sufficient. Do not invent runners or CI redesign.

## Complete and stop

Reconcile the ledger with the diff and lockfile pins once. Report: from→to table, required migrations done, deferred optionals, advisory notes, evidence labels, blocks/gaps, and the PR-body skeleton filled from the plan. Majors may still need human review when unknowns remain. Stop when required rows are closed or honestly blocked; do not keep churning optionals or unrelated deps.
