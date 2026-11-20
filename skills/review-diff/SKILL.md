---
name: review-diff
description: "Reviews a diff on two axes: standards compliance and intent match to the request or spec. Use for PR review, change-set critique, or checking whether implementation matches what was asked."
license: MIT
metadata:
  author: thedevdavid
  version: "1.0"
---

# Review Diff

Two-axis review of a change set. Keep axes separate so green standards cannot mask wrong intent (and the reverse). Do not rubber-stamp green CI. Challenge unsupported claims with hunk evidence.

Not for auditing Agent Skills packages (`review-skill`) or specialized security/bug hunts — hand those off.

## 1. Pin the diff and ask

Resolve the base (branch, SHA, PR base, or user-supplied fixed point). Confirm a non-empty diff. Identify the ask/spec: user request, issue/PR body, or linked doc. If none, say so — Intent may be unresolved, not invented.

## 2. Standards axis

Against project rules when present (`CONTRIBUTING`, coding standards, AGENTS/rules). Repo docs override generic smells. Skip what tooling already enforces. Flag clear defects and high-confidence smells with file/hunk cites; judgement calls stay labelled as such.

Read [axes.md](references/axes.md) for the thin smell baseline and Intent checklist.

## 3. Intent axis

Does the diff match the ask? Report: missing/partial requirements, wrong behaviour vs the ask, and drive-by scope not requested. Quote the ask/spec line beside each finding.

## 4. Parallelism (optional)

When authorized, run Standards and Intent workers in parallel with fresh context, then aggregate. The coordinating agent still inspects hard claims — worker rationale is not substitute evidence. Otherwise run both axes sequentially in one pass.

## 5. Report

```text
## Standards
- [severity] path:line — finding — evidence

## Intent
- [severity] path:line — finding — ask/spec cite — evidence

## Summary
Standards: N · Intent: N · worst per axis (no cross-axis winner)
```

Severity: defect / judgement / note. Empty axis → explicit “no material issues” plus what was inspected. Do not merge axes into one ranked list.
