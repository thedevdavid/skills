---
name: prune-tests
description: "Audit existing test suites case by case and delete or consolidate low-value checks. Use for deliberate test-value audits or suite pruning, including orphan cleanup; honor read-only requests."
license: MIT
metadata:
  author: thedevdavid
  version: "1.0"
---

# Prune Tests

Remove tests whose maintenance and execution costs exceed their distinct ability to detect consequential defects. When implementation is requested, deliver justified deletions or consolidations and orphan cleanup. Test counts, coverage percentages, and green results are neither success criteria nor deletion quotas.

## Establish scope and evidence

Use the requested scope and applicable project instructions. Discover maintained runners, workspace boundaries, hooks, optional integrations, and existing acceptance evidence. Respect read-only requests, planning restrictions, and execution permissions. Historical instructions and the skill itself do not authorize unrelated actions.

Account for **every individual test and expanded parameterized case** in scope: retained and unchanged suites, newly added tests, nested parameterization, loops/generated cases, skipped/conditional cases, type tests, and tests outside normal discovery. Combine source inspection with existing runner discovery or structured reports. Executed cases alone omit disabled and environment-dependent checks. Do not activate integrations merely to enumerate them. Identify duplicate titles by declaration and occurrence or another stable identity.

For generated tests, inspect the generator and each concrete expanded case in the scoped corpus. A property or generator review cannot substitute for examining its concrete cases; finite observations do not exhaust an unbounded input domain. Record unenumerated or inaccessible cases as gaps rather than certifying completeness.

Keep a temporary decision record outside the repository unless the user or project requires a committed artifact. Record each case's identity, disposition, rationale, evidence, and gaps. Read [judgment-guide.md](references/judgment-guide.md) when constructing the record or judging difficult guards, parameter predicates, modeled evidence, or contracts. Shared rationale references are allowed only after individual examination; a group/file justification is insufficient.

Reuse trustworthy evidence tied to relevant source, runner settings, environment, and cache state. Establish a new baseline only when required evidence is missing or relevant inputs changed. Separate superseded rationale from current decisions and static fault candidates from faults actually exercised.

## Challenge each case

Read its actual assertions, fixtures, mocks, and reached production branches. Require all four answers for retention:

1. **Fault:** What plausible production defect or change would it detect?
2. **Consequence:** What practical user, security, data, or operational harm follows?
3. **Oracle:** What independently justified observable outcome distinguishes correct behavior?
4. **Distinct value:** Why would stronger existing checks miss the same defect?

An oracle can be a requirement, justified fixed fixture, independent reader/reference, persisted state, externally meaningful output, or documented safety invariant. Copying the production algorithm or observing a preset mock response does not establish independence. A fixed expected value or third-party reader is not automatically filler.

**Retention is a decision gate, not a writing exercise.** For each retained case, trace the concrete chain: fixture and mocked inputs → production decision reached → asserted consequence → source of the expected outcome. Identify a plausible faulty implementation that the assertion distinguishes, beyond deleting an arbitrary return or changing a literal. If the mock supplies the claimed behavior or the expected result repeats the implementation, a serious-sounding fault description does not satisfy this gate. See the evidence examples in [judgment-guide.md](references/judgment-guide.md#trace-what-supplies-the-result).

Trace earlier guards and validate fixture assumptions. Rejection by authentication does not establish tenant isolation; malformed data rejected during decoding does not establish a downstream resource bound. Multiply-invalid inputs often prove only the first rejection. A fault probe is meaningful only if it reaches the claimed decision and fails for the intended consequence.

Delete implementation echoes, source/configuration assertions, trivial glue/library demonstrations, forwarding/mock-only checks, duplicated behavior, cosmetic checks, and repetitive valid/default/optional-field matrices without distinct protection. Naming a serious consequence does not rescue an otherwise trivial check. Critical filenames, different inputs, labels, repeated error messages, or previous retention do not earn exemptions. Preserve independently removable predicates and distinct consequences even when their assertions look similar.

Judge mocks and stateful fakes by whether real application decisions produce independently meaningful outcomes. Prefer stronger existing real flows that detect the same fault. Observing modeled residue can establish a cleanup decision; it does not establish real provider cleanup, database semantics, device behavior, or production acceptance. Disclose that boundary. Preserve valuable independent readers, justified fixtures, real persistence checks, and explicit type-contract constraints ordinary builds do not enforce.

Apply this judgment to the remaining test, not just its original title. Renaming a persistence test to a mapping test, documenting its limitations, or removing a call-count assertion does not earn retention for the leftover mapping check. Reassess its distinct consequence and the cost of its entire fixture/mock scaffold. A different function, layer, input label, or named field is not itself distinct protection. Compare the observable fault against stronger checks; distinguish repeatable automated coverage from revision-specific manual acceptance without claiming either proves more than it does.

Apply an explicit project UI/snapshot policy when present. **If none is explicitly defined, no snapshot or rendered UI tests are allowed, including interaction tests.** This is this skill's default policy. Nonvisual data/document integrity, sanitization, serialized native payloads, and application logic remain eligible; DOM tooling alone is not a UI test. A justified known-answer fixture is not inherently a snapshot assertion.

If value remains materially uncertain after reasonable investigation, leave the case temporarily unchanged, mark it unresolved, and state what evidence is missing. Preserve a valuable failing test and disclose the defect; never remove it just to obtain green results. Distinguish invalid test setup from a runtime fault. Make small directly related runtime fixes only when authorized, keep them identifiable, and record larger defects or uncovered gaps.

Use unresolved only for a concrete unanswered question that could change the disposition. An established mock echo or trivial forwarding check is low-value, not unresolved merely because deleting it exposes a coverage gap. Conversely, a user challenging over-retention does not authorize a blanket purge of mocks: preserve independently demonstrated critical decisions and real integrity checks.

## Coordinate and challenge retention

When parallel work is authorized, give workers ownership and the retention gate above. The coordinating agent must inspect the assertion chain and reached production code for each retained mocked/model-based case; a worker's rationale, inventory, or passing run is not substitute evidence. Challenge claims such as “different layer,” “data integrity,” “boundary mock,” and “could lose data” against what the assertion actually observes. If a rationale fails, check other retentions using the same rationale before declaring completion.

When the user identifies missed filler after an audit, treat the affected retention judgments as unverified. Reopen the actual source and assess the shared failure pattern; do not merely expand the ledger, rename tests, repeat green checks, or defend the prior disposition. Correct unsupported coverage claims in task-owned reports as well as the tests.

## Implement decisions and clean up

Make supported deletions and consolidations when authorized for implementation. Remove redundant portions of mixed cases while preserving their independently valuable assertions. Strengthening an existing consequential assertion may be appropriate; manufacturing replacement coverage to compensate for deleted filler is not.

Before deleting fixtures, mocks, helpers, snapshots, dependencies, or runner configuration, inspect **all remaining consumers**, including application tooling, optional integrations, and disabled operational workflows. Remove only orphaned support. Use existing maintained runners and workspace capabilities. Connecting useful undiscovered checks through existing tooling is scope-dependent; report newly activated costs separately.

Keep the work within pruning and authorized related fixes. Do not introduce custom runners, wrappers, replacement suites, E2E scaffolding, enforcement systems, or CI redesign by default. Gaps exposed by deleting false confidence are findings, not permission to rebuild the test strategy.

## Verify once per relevant change

After relevant edits, run the complete affected suites and project-required acceptance checks once. Let required hooks perform their checks and count that evidence instead of duplicating them manually. Rerun only for relevant changes, failures, or unresolved concerns; report or PR wording changes alone do not invalidate execution evidence. Honor explicit instructions to stop verification or CI waiting and disclose resulting limits.

Distinguish fresh execution, skipped integrations, cached results, and historical evidence. Use newly provisioned disposable resources for necessary authorized integration checks and preserve cleanup, including on failure. Avoid substituting existing application or hosted resources.

For important uncertain mock-dependent safeguards, use a small representative sample of temporary production faults where feasible and authorized. Record the pre-probe source, introduce one bounded fault, confirm the intended assertion failure through the maintained runner, and restore every probe change exactly while preserving preexisting work. Check restoration. Do not add a mutation framework, mutation-score target, or claim that all static candidates were exercised.

When cost measurements are requested, compare equivalent source/settings where applicable and the same measurement conditions. Separate ordinary tests, optional integrations, newly activated checks, and cache effects. Label individual timings as observations; fewer tests or a faster observation does not establish a stable speedup.

## Complete and stop

Reconcile decisions with actual edits and available runner case identities once. Completion requires scoped cases to be accounted for, justified changes and orphan cleanup to be finished, authorized acceptance evidence to be sufficient, temporary faults/resources to be restored or cleaned up, and consequential gaps to be disclosed. Before declaring completion, inspect the remaining mocked/model-based cases and their shared scaffolding against the retention gate; do not stop after confirming the deletions. Materially unresolved cases, unchecked retention claims, or inaccessible inventory prevent an unqualified claim of audit completion.

Report actual deletions/consolidations, why retained protection earns its cost, cleanup, acceptance evidence, and remaining defects or gaps. Include the temporary record's location when useful. Distinguish completed pruning from unresolved judgments or failed acceptance. Passing tests and inventory completeness do not prove every retention judgment correct.

Stop when these conditions are met. Do not keep pruning to meet a count, repeatedly reconcile unchanged state, or repeatedly poll/wait for CI. For a running required check, use bounded waits and react to changed status; once sufficient evidence exists, finish.
