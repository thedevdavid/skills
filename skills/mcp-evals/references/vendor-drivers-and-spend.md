# Vendor MCP Drivers, Gateway Isolation & Spend Guards

Read when implementing Tier 3 real-model vendor connectors (Claude, OpenAI, Google Gemini), isolated AI Gateway routing, PKCE eval actor sign-in, `--dry-run` preflight disclosure, or `SpendGuard` budget enforcement.

## 1. Unified Driver Contract

Every vendor driver takes the same `TrialContext` and returns a `DriverOutcome` separating gradable completions from infrastructure/provider failures:

```ts
export type EvalVendor = "claude" | "gpt" | "gemini";

export interface TrialContext {
  readonly accessToken: string;
  readonly attempt: number;
  readonly caseKey: string;
  readonly endpointUrl: string;
  readonly evalRunId: string;
  readonly hostTools?: readonly string[]; // Subset offered by restricted hosts; all when omitted
  readonly prompt: string;
}

export type DriverOutcome =
  | {
      readonly kind: "completed";
      readonly inputTokens: number;       // Total input tokens INCLUDING cache reads & writes
      readonly cachedInputTokens: number;
      readonly cacheWriteInputTokens: number;
      readonly outputTokens: number;
      readonly stopReason: string | null;
      readonly toolCalls: readonly ObservedToolCall[];
      readonly transcript: unknown;
    }
  | {
      readonly kind: "provider_failure";
      readonly errorClass: string;        // "connection" | "rate_limited" | "overloaded" | "server_error_5xx"
      readonly transcript: unknown;
    };
```

Set `maxRetries: 0` on SDK clients so rate limits and `5xx`/`529` overloads classify cleanly as `provider_failure` (`inconclusive`) instead of burning hidden retries. Any `4xx` request-shape error (bad parameter, invalid schema) must **rethrow**—it is a runner defect, not a provider outage.

## 2. First-Party Remote MCP Connectors (All Three Vendors)

### Claude — Anthropic Messages API (`@anthropic-ai/sdk`)

- Pass `betas: ["mcp-client-2025-11-20"]`, `mcp_servers: [{ type: "url", name, url, authorization_token }]`, and `tools: [{ type: "mcp_toolset", mcp_server_name: name }]`.
- For restricted-host cases (`trial.hostTools`), set `default_config: { enabled: false }` and enable only listed tools in `configs`.
- **Gotcha (`pause_turn`):** Long server-side MCP turns return `stop_reason === "pause_turn"`. Loop up to `MAX_TURN_RESUMES` (e.g. `4`), appending `{ role: "assistant", content: answer.content }` to `messages` until `stop_reason !== "pause_turn"`.
- **Gotcha (token accounting):** Anthropic's `usage.input_tokens` **excludes** `cache_read_input_tokens` and `cache_creation_input_tokens`. Sum all three into `inputTokens` for accurate cost pricing.
- Extract calls from `block.type === "mcp_tool_use"` (`id`, `name`, `input`) and match results/errors from `block.type === "mcp_tool_result"` (`tool_use_id`, `is_error`, `content`).

### OpenAI — Responses API (`openai`)

- Call `client.responses.create` with `store: false` (do not retain eval transcripts on OpenAI) and:
  ```ts
  tools: [
    {
      type: "mcp",
      server_label: SERVER_NAME,
      server_url: trial.endpointUrl,
      authorization: trial.accessToken,
      require_approval: "never",
      ...(trial.hostTools ? { allowed_tools: [...trial.hostTools] } : {}),
    },
  ]
  ```
- **Gotcha (token accounting):** OpenAI's `usage.input_tokens` **already includes** `usage.input_tokens_details.cached_tokens`. Do not double-add cached tokens.
- Extract calls from `answer.output` items where `item.type === "mcp_call"`:
  - Parse `item.arguments` with `JSON.parse`; if parsing throws, keep the raw string so `schemaProblems` fails it cleanly.
  - Mark `isError: Boolean(item.error) || item.status === "failed"`, and capture `item.output ?? JSON.stringify(item.error)`.

### Google Gemini — Interactions API (`@google/genai` >= 3.5.0)

- Use current Gemini 4 models (e.g. `gemini-4-hydrogen` or `gemini-4-argon`; never deprecated `gemini-2.5-*`/`2.0-*` or legacy `@google/generative-ai`).
- Call `client.interactions.create` with `store: false` (Interactions default to `store: true` for 55 days; opt out for evals) and `mcp_server` tools:
  ```ts
  const interaction = await client.interactions.create({
    model,
    input: trial.prompt,
    store: false,
    generation_config: { max_output_tokens: 4096 },
    tools: [
      {
        type: "mcp_server",
        name: SERVER_NAME,
        url: trial.endpointUrl,
        headers: { Authorization: `Bearer ${trial.accessToken}` },
        ...(trial.hostTools ? { allowed_tools: [...trial.hostTools] } : {}),
      },
    ],
  });
  ```
- Extract remote MCP calls from `interaction.steps`:
  - Pair each `step.type === "mcp_server_tool_call"` (`id`, `name`, `arguments`) with the matching `step.type === "mcp_server_tool_result"` (`call_id === call.id`, `result`, error status).
  - Read token counts from `interaction.usage` (`input_tokens`, `output_tokens`, `cached_tokens`; verify whether the active SDK version's `input_tokens` includes `cached_tokens` against your pricing table).

## 3. Isolated AI Gateway Routing (`copilot-evals` Pattern)

Isolate eval model keys and spend limits from production traffic using a dedicated gateway (Cloudflare AI Gateway shown below, or an equivalent eval-only proxy):

1. **Dedicated gateway (`copilot-evals`).** Configure provider keys as gateway-stored BYOK secrets with gateway-level per-run and daily caps. Refuse eval traffic in runtime adapters if the eval gateway ID is unset rather than falling back to the production gateway.
2. **Strip SDK placeholder keys on egress.** Provider SDKs require a non-empty `apiKey` at construction (`"eval-gateway-stored-key"`), which they attach on wire headers. Wrap `fetch` (or `httpOptions` on `GoogleGenAI` targeting `.../google-ai-studio`) to delete **all** SDK credential headers before forwarding:
   ```ts
   const PROVIDER_CREDENTIAL_HEADERS = ["authorization", "x-api-key", "x-goog-api-key"] as const;
   ```
   Then set `cf-aig-authorization: Bearer <gatewayToken>`, `cf-aig-collect-log-payload: "false"`, and `cf-aig-metadata`.
3. **Respect the 5-entry metadata limit.** Cloudflare AI Gateway accepts at most 5 metadata keys per request. Put `evalRunId` in the workspace slot (`{ evalRunId, caseKey, attempt, surface: "ai_client", vendor }`) so gateway per-run spend rules split accurately on `evalRunId`.

## 4. Eval Actor Sign-In & Scope-Keyed Tokens

- **Real PKCE flow, staging-only bootstrap.** Exchange `EVAL_DRIVER_SECRET` at a non-production bootstrap route for a short-lived session cookie, drive the real OAuth 2.1 `/authorize` (with `S256` PKCE + `resource` indicator) → `/consent` → `/token` flow, sign out the bootstrap session in `finally`, and keep only the scoped access token.
- **Cache tokens by sorted scope key.** Re-mint a token only when a case specifies narrowed `entry.scopes` (e.g. omitting `download` or `mail.send` to test scope refusal).
- **Pre-validate `hostTools` before spending.** Fetch `tools/list` once at run start. If any selected case's `hostTools` names a tool absent from `tools/list`, abort immediately before creating the run or billing a single token.

## 5. Pre-Run Disclosure (`--dry-run`) & Spend Guard

Support `--dry-run` (and print a preflight header before the first paid run in interactive sessions):

```text
PREFLIGHT (no model calls made)
  server:          https://api-staging.example.com/mcp
  commit:          a1b2c3d4
  suite:           ai-client-regression@2026-11-19.6
  toolListVersion: 9f8e7d6c5b4a3210 (18 tools)
  drivers:         claude:claude-haiku-5-5, gpt:gpt-6-luna, gemini:gemini-4-hydrogen
  planned calls:   38 cases × 3 trials × 3 drivers = 342 trials
  spend caps:      $5.00/run · $20.00/day ($3.12 spent today UTC)
```

### `SpendGuard` invariants (`spend.ts`)

- Check `await guard.beforeTrial()` **before every trial** in a sequential `for` loop:
  1. `EVAL_STOP_AT` (ISO timestamp set slightly ahead of CI job timeout) → stop cleanly with `{ kind: "out_of_time" }` so the `finally` block closes the run record instead of SIGTERM leaving an orphan open run.
  2. `runCostUsd >= RUN_COST_CAP_USD` → stop with `{ kind: "run_cap" }`.
  3. `await evalSpendTodayUsd(db) >= DAILY_COST_CAP_USD` (queried live from the shared store so concurrent jobs halt within one trial of each other) → stop with `{ kind: "daily_cap" }`.
- After each completed trial, compute exact USD cost from token counts. If `costUsd === null` (model missing from pricing table), immediately trip `{ kind: "unpriced", model }`—caps cannot guard uncounted spend.
- **Exit code precedence (`exitCodeOf`):**
  - Any case verdict `=== "fail"` → `1` (`EXIT.failed`)
  - Else `stop?.kind === "daily_cap"` → `3` (`EXIT.dailyCap`)
  - Else any trial `=== "inconclusive"` or `stop !== null` → `2` (`EXIT.inconclusive`)
  - Else → `0` (`EXIT.passed`)
