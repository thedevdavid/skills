# Evidence labels and stop rules

Read when unsure whether to rerun, reuse, or disclose a gap.

## Labels

| Label | Use when | Not enough alone for |
| --- | --- | --- |
| fresh | You ran the full scoped check now; exit/output inspected | — |
| cached | Same relevant source/settings/env; earlier trustworthy result | Claims after relevant code/config/env change |
| skipped | User/project forbade it, missing auth/env, or optional path | Unqualified “all checks pass” |
| historical | Other commit, prior CI, or earlier session | This revision’s done claim |

Hooks that enforced the same acceptance bar count as fresh/cached like any other check — note they ran via hook, not a second manual duplicate.

## Rerun vs reuse

- Rerun after relevant edits, failures, or unresolved concerns.
- Do not invalidate solely for report/PR wording changes.
- Do not substitute a different weaker check (linter ≠ tests ≠ build) and call it the same proof.

## Stop conditions

- Sufficient scoped evidence exists → claim with labels, stop.
- User says stop waiting/verifying → stop; disclose what remains unverified.
- Required check still running → bounded wait; on terminal status, finish.
- Never: endless CI poll, quota-driven re-runs, or “one more time” without a relevant change.
