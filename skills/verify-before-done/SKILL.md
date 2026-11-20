---
name: verify-before-done
description: "Gates task completion on scoped acceptance evidence. Use before declaring done, finishing a PR, or claiming tests/CI passed; distinguishes fresh execution from cached, skipped, or historical evidence and stops unnecessary polling."
license: MIT
metadata:
  author: thedevdavid
  version: "1.0"
---

# Verify Before Done

Do not claim done, fixed, or green without scoped acceptance evidence. Complements `skills/prune-tests/` verify/stop language — same vocabulary, for any task.

No new runners, wrappers, or CI redesign by default. Use maintained project commands and hooks.

## 1. Scope the claim

Before any success claim, name: what must be true, which commands/checks prove it, and what is in scope. If the user limited verification, treat that as a bound and disclose gaps — do not silently widen or invent runners.

## 2. Gather evidence (label every source)

| Label | Meaning |
| --- | --- |
| **fresh** | Full command/check run in this turn against current relevant inputs |
| **cached** | Reused result from an earlier run still valid for unchanged relevant inputs |
| **skipped** | Intentionally not run (permission, env, user stop, optional integration) |
| **historical** | Past run or green CI from another revision — not proof for this claim alone |

Required hooks that already ran count as evidence; do not duplicate them. One complete affected run per relevant change. Report/PR wording-only edits do not invalidate execution evidence. Prefer disposable resources for authorized integrations; clean up, including on failure.

Read [evidence.md](references/evidence.md) when labeling is ambiguous or deciding whether to rerun.

## 3. Gate the claim

Only claim what the evidence supports. “Should pass”, confidence, partial logs, or agent self-reports are not evidence. If checks fail or are skipped, state actual status with labels and gaps.

## 4. Stop

Bounded waits for required in-flight checks; react to status changes. Once evidence is sufficient — or the user says stop — finish and disclose limits. Do not poll CI forever or re-run unchanged suites to look busy.
