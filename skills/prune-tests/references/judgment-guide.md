# Case judgment and decision records

## Lightweight record

Use a temporary table, structured list, or existing audit format. No permanent inventory system is required. Give every concrete case an entry; reusable rationale IDs can avoid repeated prose without hiding separate judgments.

| Field | What to record |
| --- | --- |
| Identity | Relative path, declaration/title, expanded parameters or generated input, and occurrence when names repeat. Preserve old/new identities for consolidation or renaming. |
| Disposition | Retain, delete, consolidate into an identified case, or unresolved. |
| Judgment | Retention: concrete production decision, faulty alternative, asserted consequence, source of expected outcome, distinct value. Deletion: why protection is low-value, unsupported, or redundant. |
| Comparison | Identify the stronger existing check when one exists. Otherwise disclose the gap; do not invent substitute protection. |
| Evidence | Relevant source revision/settings, reached branch, fixture assumptions, and supporting evidence reference. Label static reasoning, exercised fault, fresh/cached execution, skips, or historical evidence. |
| Limit | Modeled boundary, remaining defect, unexpanded cases, missing resources, or superseded rationale. |

For an unresolved case, record the missing fact and the smallest useful next investigation. Do not repeatedly re-open settled cases merely to fill a record. An existing ledger is evidence to inspect, not authority to trust blindly.

Keep an execution note with command, relevant source/settings, environment, cache state, results, skips, and cleanup. Update it only when evidence changes. This supports reuse without repeated suite runs, source hashing, reconciliation, or CI polling.

## Trace what supplies the result

Follow the data through the test and production code before accepting a rationale. A test invoking real code can still have its claimed outcome supplied entirely by its own fixture. Ask which production decision separates the correct outcome from a plausible harmful one, and whether the assertion would notice that decision being wrong.

| Pattern | Judgment |
| --- | --- |
| Mock insert returns its input; test calls the result “persisted” and compares it with that input. | The asserted value comes from the mock. It establishes neither persistence nor an independent application decision. Delete unless a separate consequential assertion earns retention; remove exclusive insertion fixtures too. |
| Fake collections use Maps, ignore query predicates, and immediately resolve transaction promises. | They cannot establish selection predicates, durable recovery, rollback, or database races. A pure application guard may still be tested, but identify that guard and its independent consequence; reassess whether the scaffold is worth keeping for it. Renaming the case is insufficient. |
| A loopback server receives a literal body constructed inside the test's publish callback. | Real HTTP does not make the body an application-generated artifact. This observes callback gating at most. Judge the gate against stronger checks and its scaffold cost; do not claim request construction, identity preservation, upload ordering, or integrated recovery. |
| Expected digest/ID/output is calculated using the same production helper as the code under test. | Equality alone cannot detect a shared error in that helper. A separate outer decision can still be useful, but its expected outcome must be justified independently; do not label helper self-comparison an independent oracle. |
| A stub returns admitted external data, and real code must reject a forbidden transition or reconstruct a verifiable capture. | Potentially valuable: establish fixture validity, the reached guard/transformation, and the independently specified denial or integrity consequence. Compare stronger flows and exercise a representative fault where warranted. Do not discard it merely for using a mock. |

A generic hypothetical such as “a field might be omitted” does not establish why a thin projection deserves its own test. Conversely, a fixed fixture for a nontrivial historical-value, authorization, encoding, or integrity contract can establish useful behavior. Judge the concrete contract and consequence, not the presence of literals or mocks.

If a case's original claim fails, the remaining narrower assertion must earn retention from scratch. “This only tests the decision, not persistence” is a limitation, not a reason to keep it. A large scaffold may have no worthwhile consumers after its unsupported cases are removed. Do not manufacture new fake state, wrappers, or cases to justify preserving that scaffold.

## Earlier guards and invalid fixtures

Suppose `removeFile` first checks authentication, then tenant ownership, then retention. A test named "rejects a foreign retained file" supplies an unauthenticated caller and asserts 403. It establishes authentication rejection; it proves neither ownership nor retention.

Challenge its actual value as an authentication case against existing checks. Delete it if redundant; retain or consolidate only its independently useful protection and correct misleading claims. Do not automatically add new downstream cases. Record the ownership/retention gap if no existing check establishes it.

A fake query returning a preset tenant-B record while production predicates require tenant A can make a saved-value assertion impressive but unreachable. Inspect whether the setup represents an admitted production state. A test that previously detected a temporary fault can still be invalid because of fabricated setup.

If two production guards contain the same fault, removing only one may correctly leave the suite green. A surviving redundant guard is not proof that the test is useless. Define the consequential fault and observed path before interpreting a probe.

## Similar parameters, different protection

A formula-injection sanitizer handles leading `=`, `+`, `-`, and `@` through separately enumerated predicates. Each row can detect removal of its own predicate while the other rows remain green. Similar assertions therefore do not justify deleting three rows as duplicates.

Conversely, ten labels passing equivalent values through the same predicate and observing the same outcome may earn one representative case. Inspect the implementation and consequence rather than counting labels. Similar inputs may also cross distinct boundaries, encodings, state transitions, or consumers; explain the distinction concretely.

Nested parameterization expands combinations. Do not treat reviewing the shared body as reviewing every row: determine which combinations reach distinct decisions and which merely duplicate a rejection. For randomized/property-based tests, inspect the generator and scoped concrete corpus without claiming exhaustive review of every possible input.

## Stateful models and cleanup

A cleanup test starts a fake provider's object set with stale bytes, returns "multipart missing," and lets production control flow decide whether to remove the object. Asserting an empty set can detect an early return that leaks the bytes. The fake state is an independently meaningful oracle for the application's deletion decision.

A companion test that only expects `delete(key)` to be called adds little when the state-based test detects the same omission. A test asserting a preset successful provider response does not prove cleanup.

Compare against stronger existing real flows before retention. Even a useful model does not prove actual provider deletion, persisted retries, transaction rollback, or absence of hosted residue. State the supported invariant narrowly.

Do not confuse stateful evidence with automatic independence. Replacing `expect(delete).toHaveBeenCalled()` with a Set mutation controlled by the same mock is not by itself an improvement. Establish that real application control flow chooses a consequential action under a valid failure scenario, the model does not implement that decision, and no stronger retained check detects the same omission. Never add this replacement merely to keep a weak test during pruning.

## Independent readers, fixtures, and contracts

An encoder checked by an independent decoder against externally justified byte fixtures can catch interoperability defects that its own reader shares. Inspect fixture provenance and observable output; do not delete it merely because it calls a library. A fixture is useful only while it represents a justified expectation and has a consumer.

An independent reader is not itself a retention exemption. Parsing an object literal emitted by a thin standard JSON wrapper usually demonstrates serializer behavior and obvious field mapping, rather than a distinct application interoperability risk. Contrast that with independently reading a nontrivial byte layout, normalization, escaping, framing, or compatibility contract. Establish the application's distinct decision and oracle before retaining the reader check; a generic claim that a field could be omitted is insufficient by itself.

An explicit type assertion may enforce "untrusted identifiers cannot enter this API" or "this public result remains a discriminated union." Ordinary production compilation can pass after such a constraint weakens, particularly when test files are excluded. Inspect what the assertion uniquely enforces and how it is executed.

A type test recreating a retired schema solely to compare two copies may be compatibility scaffolding with no current contract. Neither "typecheck covers it" nor "type contracts are critical" is sufficient without examining the constraint.

## UI, snapshots, and nonvisual integrity

Discover the project's explicit policy first. In its absence, this skill excludes snapshot and rendered UI tests, including interactions. Project permission to use them does not exempt them from individual value judgment.

Parsing a document to ensure tenant data is absent, sanitizing executable markup, or checking the identity in a serialized print payload can be consequential nonvisual integrity. Counting CSS classes, matching layout, or snapshotting a rendered tree is presentation evidence. Classify the assertion and consequence, not the testing library.

Known-answer input/output fixtures can be independent oracles. Snapshot assertions that merely approve current output are different. Where an existing mixed test has consequential assertions plus prohibited presentation checks, preserve its useful nonvisual assertions through scoped consolidation where appropriate.

## Valuable failing cases and gaps

A real persistence test demonstrates that an accepted write loses the customer's value after reopening storage. If the expectation comes from the product contract and the setup is valid, the failure is evidence of a runtime defect. Keep it. Report failed acceptance or make only an authorized small related fix.

A failure caused by an impossible fixture, obsolete API, or missing authorized test resource needs a different diagnosis. Repair setup only where that preserves an independently valuable case; otherwise delete the invalid check and disclose unsupported behavior. Never reinterpret a correct expectation simply to make it pass.

Removing fake autosave coverage does not mean autosave works, and a lower-level artifact race does not replace integrated autosave ordering. It is valid to record "no retained check establishes this behavior." Keep current claims separate from withdrawn claims.

## Evidence reuse and completion

If the tested runtime, tests, dependencies, runner settings, and relevant environment are unchanged, a documentation correction does not require another test run. Hooks that already executed acceptance checks count as execution evidence. Task-cache reuse is different from fresh execution; optional integrations that skipped remain skipped evidence.

Use a new disposable resource when an integration is necessary and authorized. Keep its cleanup observable. If acceptance fails, report the specific failure; do not delete the meaningful failing test or repeatedly run an unchanged command hoping for green.

Finish with reconciled case decisions, actual edits, sufficient authorized evidence, restored probes/resources, and disclosed gaps. Check the retained set, not only the removed set. Report what mocks/models remain and the specific decisions they establish; disclose any unresolved retention instead of presenting case counts or green CI as proof of quality. Unknown case identities or materially unresolved judgments prevent an unqualified completeness claim, even when other pruning is finished.
