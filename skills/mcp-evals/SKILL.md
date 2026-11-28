---
name: mcp-evals
description: "Builds, runs, and triages verification and evaluation suites for MCP servers and MCP Apps across Claude, OpenAI, and Gemini connectors with deterministic code/state graders and spend guards. Use when adding or running MCP evals, checking protocol/ui:// conformance, verifying OAuth challenges/scopes/revocation, importing test corpora into eval cases, or triaging and diffing eval runs. Skip when scaffolding an MCP server or widget without verification (use build-mcp-server or create-mcp-app) or pruning general test suites (use prune-tests)."
license: MIT
metadata:
  author: thedevdavid
  version: "1.0"
---

# MCP Evals

Own MCP server and MCP App verification in-repo across three deterministic tiers: local runtime/protocol conformance, non-TTY live wire inspection, and real-model vendor connector evals graded strictly by code and persisted domain state (no hosted eval SaaS, mocked/replayed model stand-ins, throwaway client scripts, or LLM-as-judge).

## Choose the tier

Match the claim to the cheapest tier that actually proves it. Lower tiers gate protocol, schema, and authority bugs for free; Tier 3 spends model budget solely on what static tests cannot see (whether real vendor connectors and models pick the right tool, construct valid arguments, recover from refusals, and reach the intended domain state).

| Tier | Surface & tooling | Proves | Cannot prove |
| --- | --- | --- | --- |
| **1. Runtime & conformance** | Local/Worker runtime + `@modelcontextprotocol/conformance` + `mcp-inspector --cli --strict` | JSON-RPC framing, per-revision spec compliance, portable tool schemas, OAuth scope/root fences, non-disclosing denials, cursor signing, revocation, pending-mutation attribution | Live deployment wiring, vendor connector serialization, or model tool choice |
| **2. Live protocol & Apps check** | Non-TTY `mcpdo` / `mcp-inspector --cli --stored-auth-only` against staging/prod | Deployed transport reachability, OAuth 2.1 DCR/CIMD/step-up challenge headers, `ui://` resource MIME/CSP/fallback metadata, live service bindings (embeddings, DB) | Host iframe/sandbox/postMessage rendering or model prompt interpretation |
| **3. Real-model connector evals** | Anthropic Messages API, OpenAI Responses API, Google Gemini Interactions API → remote MCP server | Per-vendor tool selection (direct, indirect, negative), live JSON Schema adherence, refusal recovery, multi-step domain outcomes, prompt-injection resistance | Interactive browser OAuth consent UX or native host UI chrome (keep a dated manual host receipt) |

## Tier 1: Deterministic authority & conformance

Run against the real server handler and real database/storage schema (local Postgres, SQLite, or Workers `workerd`/Miniflare)—never mocked tool handlers.

1. **Official protocol conformance.** Run `@modelcontextprotocol/conformance` (`conformance server --url <local-forwarder> --requirements <revision> --expected-failures <baseline.yml>`) once per supported MCP protocol revision, plus `mcp-inspector --cli <endpoint> --transport http --stored-auth-only --method tools/list --strict` for schema portability. Because `conformance` does not complete interactive OAuth, front the local endpoint with a thin HTTP forwarder that injects a real AS-issued `Bearer` token only when `Authorization` is absent and streams SSE chunks unchanged.
2. **Bidirectional baseline files.** Partition `--expected-failures` entries into `# fixture` (reference-server diagnostic tools not served), `# not served` (omitted optional capabilities), and `# stateless` (session-ID checks on stateless HTTP). Treat both unbaselined failures **and** baseline entries that now pass as red builds—delete stale entries immediately.
3. **Authority, scope, and mutation boundaries.** Prove every security invariant deterministically before touching a model:
   - **Split scopes & non-disclosure:** verify distinct scopes (`read` vs `save` vs `write` vs `send`) and selected-root / workspace / Inbox intersections. Inaccessible items outside a narrowed grant return a non-disclosing denial (e.g. `Item not found`) rather than leaking existence or title.
   - **Signed cursors:** bind pagination and original-file continuation cursors cryptographically to `(operationId, itemId, attachmentId, contentHash, etag)` so tampering or cross-item replay fails closed.
   - **Live grant revocation:** revoking or narrowing a connection blocks subsequent reads and writes immediately, including queued or cached continuation tokens.
   - **Command-attributed & pending mutations:** verify MCP writes record client attribution for audit/undo, and high-consequence actions (such as outbound `mail.send`) land in a pending/frozen approval state unless explicit direct-send authority is granted.

Read [protocol-and-authority.md](references/protocol-and-authority.md) when wiring `@modelcontextprotocol/conformance`, local auth forwarders, OAuth 2.1 challenges, scope/revocation tests, or `ui://` MCP Apps checks.

## Tier 2: Non-TTY live & Apps inspection

Use `mcpdo` for interactive/agent sessions and `mcp-inspector --cli --stored-auth-only` (with `stdio: ["ignore", "pipe", "pipe"]` and `CI=true`) for automated post-deploy gates. Mask bearer tokens in CI logs (`::add-mask::<token>`) before passing `--header`.

1. **Breadth-first transport & OAuth metadata probe.** Separate raw wire/probe evidence, breadth-first `doctor` snapshots, and connected-sweep output. Inspect RFC 9728 protected-resource metadata, AS metadata, `S256` PKCE, and registration posture (CIMD, DCR, or preregistered), and distinguish lazy mixed-auth surfaces from full-auth endpoints before judging `tools/list`.
2. **Three-flavor auth-challenge verification.** Hosts act on different wire signals; assert raw responses rather than CLI-added convenience fields:
   - **Unauthenticated (`401`):** `WWW-Authenticate: Bearer resource_metadata="<rfc9728-url>"`.
   - **Under-scoped (`403` step-up):** `WWW-Authenticate: Bearer error="insufficient_scope", scope="<required>", resource_metadata="…"`.
   - **ChatGPT-style tool challenge:** `isError: true` tool result carrying `_meta["mcp/www_authenticate"]` (`error` + `error_description`) on tools declaring `oauth2` `securitySchemes` (distinct from any CLI-added `_authChallenge` wrapper).
3. **MCP Apps (`ui://`) server-side conformance.** Run `apps conformance` / resource checks for `_meta.ui.resourceUri` links, `resources/read` HTML MIME payloads, `_meta.ui.csp` / `_meta.ui.domain` declarations, `visibility: ["app"]` tool scoping, and non-UI text fallbacks. State the boundary explicitly: server-side `ui://` checks do not prove host iframe lifecycle, sandbox proxy, or `postMessage` bridge execution.
4. **Semantic health assertions.** Call representative read tools and assert real backing bindings answered (e.g. `queryEmbedding !== "unavailable"` so silent full-text fallback on broken vector/service bindings fails the check).

## Tier 3: Real-model vendor connector evals

Attach the deployed staging/eval MCP endpoint to each vendor's **first-party remote MCP connector** so models see the exact wire tool list and schemas a real host presents:
- **Claude:** Anthropic Messages API (`mcp_servers` + `mcp_toolset`, `betas: ["mcp-client-2025-11-20"]`, `pause_turn` loop).
- **OpenAI:** Responses API (`tools: [{ type: "mcp", require_approval: "never", allowed_tools }]`, `store: false`).
- **Google Gemini:** `@google/genai` Interactions API (`tools: [{ type: "mcp_server", url, headers, allowed_tools }]`, `store: false`).

### Isolation, gateway, and spend discipline

- **Isolated eval actor & neutralized workspace.** Mint short-lived PKCE tokens for a dedicated migration-seeded eval actor in a disposable eval workspace—never personal or staff accounts. Neutralize outward side effects in the data layer (shares non-public/auto-expiring, downloads short-circuited, outbound emails held/discarded, analytics/popularity/client-budgets excluded). Restore temporary role demotions (`viewer` → `owner`) in `finally`.
- **Dedicated eval gateway & key boundary (`copilot-evals` pattern).** Route all three vendor SDKs through an isolated AI Gateway (such as Cloudflare AI Gateway with BYOK stored keys, or an equivalent eval-only proxy) separated from production keys and client budgets, with payload logging off, SDK placeholder credential headers stripped (`authorization`, `x-api-key`, `x-goog-api-key`), and `evalRunId` stamped in request metadata.
- **Trigger & target alignment.** Gate CI Tier 3 runs on diffs touching tool names/descriptions/schemas/annotations, prompts/skills, or backing operations—and point the runner at an environment serving that exact commit (never pre-deploy staging running the prior commit).
- **Pre-run disclosure (`--dry-run` / preflight).** Before spending budget on a new or changed suite, print and confirm: target `server.origin`, 16-char `toolListVersion` SHA-256 prefix, `suiteVersion`, git `commit`, `vendor:model` matrix, `cases × trials` call count, and active per-run / daily USD caps. Require `idempotencyKey` on scenario setup writes and retryable run launches.
- **Sequential spend guard & canonical exit codes.** Run trials sequentially, checking `beforeTrial()` against both the per-run cap and today's shared store spend before every trial. Halt immediately if a model lacks a price entry or `EVAL_STOP_AT` is reached. Always close the run record in `finally` with the negotiated `protocolRevision`.
  - `0` — **passed:** every conclusive case passed and `0` cases were inconclusive.
  - `1` — **failed:** at least one case failed a trial assertion.
  - `2` — **inconclusive:** provider outage/rate-limit (`connection`, `rate_limited`, `overloaded`, `5xx`), retryable server refusal, `scenario_setup_failed`, per-run spend cap, unpriced model, or `EVAL_STOP_AT` left trials unjudged with zero assertion failures.
  - `3` — **daily cap / not gateable:** daily spend cap prevented or stopped the run, or unapproved/incomplete corpus import blocked gating.

Read [vendor-drivers-and-spend.md](references/vendor-drivers-and-spend.md) when implementing Claude/OpenAI/Gemini MCP drivers, AI Gateway BYOK routing, PKCE eval sign-in, `--dry-run` preflight, or `SpendGuard`.

### Deterministic code & state grading (zero LLM-as-judge)

Keep cases **host-neutral** (`driver: "any-ai-client"`, `author: null` until human acceptance) so Claude, OpenAI, and Gemini run identical case keys. Pin every run's identity to `(commit, suiteId, suiteVersion, sortedModelIds, promptVersion, toolListVersion)`.

1. **Tool selection & negative precision.** Cover direct prompts, indirect user-intent prompts, restricted-host subsets (`hostTools: ["search", "fetch"]`), directory-review sets, standing `injection` regression cases, and negative cases (`expected: []` asserting `no_tool`). Hold negative-prompt precision high before chasing recall.
2. **Live JSON Schema grading (`schemaProblems`).** Validate the first call's parsed arguments against the server's live `tools/list` `inputSchema`: fail on missing `required` properties and invented property names under `additionalProperties: false` (supporting `anyOf`/`oneOf` and array `items`). Leave raw unparseable JSON argument strings intact so schema validation fails them deterministically.
3. **Refusal recovery vs retryable infrastructure refusals.**
   - `recovers: true` passes only when an `isError` call on `tool` (e.g. unknown taxonomy term or stale optimistic-concurrency version) is followed by a succeeding call on `tool`.
   - `failedOnlyByRetryableRefusals` reclassifies a failing trial to `inconclusive` (`server_refused_retryable`) **only** when every failed check stems from a tool whose first call returned a transient upstream code (`rate_limited`, `dependency_unavailable`, `timed_out`). Wrong tool selection, missing calls, or model-caused validation refusals still fail (`1`).
4. **Post-trial domain-state grading (`gradeScenarioState`).** Grade multi-step scenarios on persisted workspace state read after `since = evalDatabaseNow(db)`, never on the assistant's prose claim:
   - Assert created/mutated records, bounds, and anti-hallucination provenance (every persisted entity ID came from the trial's own tool results).
   - For permission, scope, role, and post-revocation cases, require the model to attempt or hit the boundary while proving **zero persisted residue** (`no_project_created`, `downloads: max 0`, or mutation held in `pending_approval`).
5. **Multi-trial regression rule.** Run regression cases `N` times (default `3`). Any single `fail` trial fails the case; `inconclusive` trials are excluded from per-tool precision/recall (`scoreToolSelection`). Separate aspirational `capability` suites (report-only) from `regression` suites.

Read [grading-and-scenarios.md](references/grading-and-scenarios.md) when writing `gradeTrial`, `schemaProblems`, `gradeScenarioState`, refusal-recovery checks, or `scoreToolSelection`.

## Triage, baseline compare, and corpus import

- **Triage in strict order — never re-run to "see if it passes".** Paid failed runs already carry evidence.
  1. Read `verdictSource` (`assertion_failure` | `provider_or_infra_failure` | `spend_cap_or_policy_stop`) before opening trial rows. Zero tokens, uniform crashes across cases, or `scenario_setup_failed` mean the run never tested the tools.
  2. Paginate all trial results before computing pass rates. Separate **`N/N` deterministic failures** (server bug, schema drift, or stale expectation) from **`k/N` intermittent failures** (ambiguous prompt or two reasonable tools—tighten prompt/`hostTools` or accept valid alternatives).
  3. On multi-step scenarios, read the **first `fail` step**; later failures/`skipped` steps are cascade, and `pending` on a terminal run indicates timeout or cap stop.
- **Diff against a baseline (`compare`).** Compare head against `baseRunId` or `baseCommitSha` to classify every case as `regressed`, `fixed`, `new_case`, `removed_case`, `changed` (case prompt/checks edited—incomparable), `unchanged_passed`, or `unchanged_failed`, and flag any per-tool `recallDropped` (including a base-recalled tool never reached in head).
- **Stamp honest statuses when importing corpora.** Treat external test files (Promptfoo, Vitest/Jest, pytest, CSV) as **untrusted static data**—never execute them during conversion. Validate against a closed case schema. Stamp `import.status`:
  - `exact` — cites structural rules (`S-1..S-6`) in `import.note`; runs automatically once tool names resolve against live `tools/list`.
  - `approximated`, `unsupported`, `unresolved` — parked `disabled: true` so unreviewed conversions never gate CI. Running an `approximated` case requires explicit per-run `--allow-approximated <caseId> --approval-reason "<why>"` with zero persistent acceptance.

Read [triage-compare-and-import.md](references/triage-compare-and-import.md) when investigating a failed/inconclusive run, implementing `compareEvalRuns`, or converting existing tests into MCP eval suites.

## Verify and stop

Gate completion with `verify-before-done` labels (`fresh`, `cached`, `skipped`, `historical`):
1. Change **one tool description, schema field, or skill cue at a time** when tuning model behavior and attach the `compare` diff—richer prose can raise recall on one tool while inflating step counts and regressing another.
2. Never promote a bounded single-surface live host receipt (e.g. ChatGPT Web) into broad host or Desktop/Mobile certification.
3. Confirm disposable scenario fixtures are cleaned up (or neutralized in the dedicated eval workspace), run records are closed, no secrets/tokens appear in committed receipts, and stop once acceptance evidence is sufficient.
