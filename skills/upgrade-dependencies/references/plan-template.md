# Plan template

Read when drafting the upgrade plan, completeness ledger, or PR-body skeleton. Keep the working copy outside the repo unless required.

## Plan

```markdown
# Upgrade plan

## Mode
semi-auto | full-auto

## Scope
- Ecosystems:
- Packages / skills in scope:
- Out of scope (explicit):

## Provenance
| Package | From (lock/pin) | To | Evidence (lock path / diff) |
|---------|-----------------|----|-------------------------------|
|         |                 |    |                               |

## Transitive majors
| Package | From | To | Why pulled |
|---------|------|----|------------|

## Sequence
1. …
2. …

## Codemods
| Package | Command | Covers | Follow-ups |
|---------|---------|--------|------------|

## Ledger summary
- Required open (`breaking` / `required-config`): N
- Behavioral (HIGH RISK): N
- Deferred optional (`optional-feature`, user opt-in only): N
- Advisory notes: N
- Blocked / unknown: …

## Behavioral (HIGH RISK) — do not collapse into “types still pass”
| ID | Package | Change | Codebase | Status |
|----|---------|--------|----------|--------|
|    |         |        |          |        |

## Optional recommendations (do not implement until user opts in)
| ID | Package | Feature | Why deferred |
|----|---------|---------|--------------|
|    |         |         |              |

## Risks / blocks
- …

## Acceptance gates
- Frozen/immutable install:
- Project commands:
- verify-before-done handoff: yes (if available)
```

## Ledger rows

One row per upstream required/behavioral/advisory/optional item (see SKILL.md columns). Example:

```text
ID: react-19-jsx-runtime
Package: react (npm) 18.3.1 → 19.0.0
Source: https://…/migrating-to-react-19#… 
Severity: breaking
Codebase: hit — src/ui/Button.tsx (classic runtime import)
Action: migrate
Status: open
```

## PR-body skeleton

Fill after required work; paste into the PR when asked to open one.

```markdown
## Summary
- Upgraded: <pkg> <from> → <to> (and …)
- Required migrations: <short list>
- Behavioral notes: <short list or “none”>
- Deferred optionals: <short list or “none”>
- Advisories in range: <ids or “none material”>

## Test plan
- [ ] <frozen/immutable install>
- [ ] <project build>
- [ ] <project lint>
- [ ] <project test>
- [ ] <extra gates if any>
```
