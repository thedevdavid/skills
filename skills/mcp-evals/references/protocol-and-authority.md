# Protocol Conformance, Authority Fences & Live Inspection

Read when implementing Tier 1 local/runtime authority and `@modelcontextprotocol/conformance` tests, Tier 2 non-TTY `mcpdo` / `mcp-inspector` checks, OAuth 2.1 challenge verification, or `ui://` MCP Apps conformance.

## 1. Local `@modelcontextprotocol/conformance` + Auth Forwarder

The official `@modelcontextprotocol/conformance` runner does not drive custom OAuth consent screens. Test authenticated endpoints against the real runtime (e.g. `workerd`/Miniflare or local HTTP server + real DB migrations) by placing a minimal loopback forwarder in front of the worker:

```ts
function forwardTo(handleFetch: (req: Request) => Promise<Response>, accessToken: string) {
  return async (incoming: IncomingMessage, outgoing: ServerResponse) => {
    const headers = new Headers();
    for (const [k, v] of Object.entries(incoming.headers)) {
      if (v !== undefined) headers.set(k, Array.isArray(v) ? v.join(", ") : v);
    }
    if (!headers.has("authorization")) {
      headers.set("authorization", `Bearer ${accessToken}`);
    }
    const target = new URL(incoming.url ?? "/", "https://mcp.internal");
    const hasBody = incoming.method !== "GET" && incoming.method !== "HEAD";
    const res = await handleFetch(
      new Request(target, {
        body: hasBody ? (Readable.toWeb(incoming) as ReadableStream) : undefined,
        duplex: hasBody ? "half" : undefined,
        headers,
        method: incoming.method,
      } as RequestInit),
    );
    outgoing.writeHead(res.status, Object.fromEntries(res.headers.entries()));
    if (res.body) {
      for await (const chunk of res.body as unknown as AsyncIterable<Uint8Array>) {
        outgoing.write(chunk);
      }
    }
    outgoing.end();
  };
}
```

- **Isolate rate limits per revision run.** Mint a fresh workspace/actor grant per `conformance` invocation so per-connection rate limits never bleed across protocol revisions (`2025-11-25`, `2026-07-28`, etc.).
- **Close keep-alive sockets in `afterAll`.** Call `server.closeAllConnections()` alongside `server.close()` so lingering SSE/HTTP clients from the conformance runner do not hang Vitest.
- **Keep `--expected-failures` bidirectional.** Commit `mcp-conformance/<revision>.yml` grouping expected skips under `# fixture` (reference `test_*` tools your server intentionally omits), `# not served` (prompts, completions, logging, subscriptions), and `# stateless` (`server-sse-multiple-streams:server-sse-multiple-streams-session` on stateless Streamable HTTP). Fail CI both when a new check breaks **and** when a listed entry starts passing.
- **Pair with strict Inspector schema check.** In the same suite, run `mcp-inspector --cli <forwarderUrl> --transport http --stored-auth-only --method tools/list --strict` to catch non-portable JSON Schema constructs before any vendor SDK sees them.

## 2. Authority, Scope & Mutation Invariants (Tier 1)

Test these deterministically against the real request handler (`POST /mcp` or `/api/mcp`), never through a model:

| Boundary | What to assert |
| --- | --- |
| **Transport & batch guards** | Reject cross-origin browser `Origin` headers (`403 invalid_origin`), unsupported `Mcp-Protocol-Version` (`400`), oversized bodies (`413`), and JSON-RPC batch arrays when single-message Streamable HTTP is required (`400`). Return `202` empty body on notifications-only payloads. |
| **Granular OAuth scopes** | Separate read (`*.read`, `*.search`), narrow write/capture (`*.save`, `drafts.write`, `*.triage`), broad mutation (`*.write`, `*.organize`), and outbound/irreversible (`mail.send`, `share`). Verify `findMissingMcpScope` per method and tool name. |
| **Selected-root & Inbox non-disclosure** | When a grant scopes access to selected roots/workspaces with `inbox: false`, reading an outside-root or Inbox ID returns a **non-disclosing** error (`Item not found`)—never `403 Forbidden on <secret title>`. Verify descendant ancestry is rechecked on every call. |
| **Signed continuation cursors** | HMAC-sign pagination and binary/extraction cursors over `(version, operationId, itemId, attachmentId, contentHash, contentLength, etag, offset)`. Assert tampering `offset` or swapping `itemId` returns `null`/error, and UTF-8 multi-byte boundaries page cleanly without corrupting `contentHash`. |
| **Immediate grant revocation & workspace fencing** | Revoking a client grant or removing a workspace immediately rejects subsequent reads, saves, and in-flight cursor continuations on that token without waiting for JWT expiry. |
| **Command attribution & pending-send hold** | Every MCP mutation writes through the same command/audit log as first-party clients with MCP client attribution + undo support. High-risk actions (e.g. `mail.send`) default to creating an owner-reviewable `pending_send` (plus frozen-send undo window) unless the client grant explicitly enables direct send. |
| **Untrusted content marking** | Tool outputs carrying third-party text (emails, web captures, shared notes) fence/mark payloads as untrusted data; server handlers never interpret content fields as instructions. |

## 3. Breadth-First Probe & Three-Flavor OAuth Challenge Inspection (Tier 2)

Separate primary wire/RPC evidence from aggregated `doctor` convenience summaries. Probe transport and OAuth discovery metadata first (`/.well-known/oauth-protected-resource`, AS metadata, `S256` PKCE, and client registration support: CIMD, DCR, or preregistered; test DCR redirect-mismatch and invalid-bearer rejection when DCR is exposed).

MCP hosts differ on which wire challenge triggers sign-in or OAuth 2.1 step-up. Verify the exact wire contract your target hosts consume:

1. **HTTP `401 Unauthorized` (unauthenticated / expired / revoked token):**
   ```http
   HTTP/1.1 401 Unauthorized
   WWW-Authenticate: Bearer resource_metadata="https://example.com/.well-known/oauth-protected-resource"
   ```
2. **HTTP `403 Forbidden` (authenticated token missing required tool scope — RFC 6750 step-up):**
   ```http
   HTTP/1.1 403 Forbidden
   WWW-Authenticate: Bearer error="insufficient_scope", error_description="This tool requires the second:save scope", scope="second:save", resource_metadata="https://example.com/.well-known/oauth-protected-resource"
   ```
3. **Tool-level `isError` challenge (ChatGPT / lazy mixed-auth servers):**
   Tool declares `oauth2` in `securitySchemes` and returns a `200` JSON-RPC tool result with `isError: true` and `_meta["mcp/www_authenticate"]` containing `error` and `error_description`.
   - **Gotcha:** Inspection CLIs often attach a synthetic `_authChallenge` sibling field for convenience. Assert against raw HTTP headers and `_meta["mcp/www_authenticate"]`, never `_authChallenge`.
   - Missing `scopes_supported` or optional `scope` hints without a spec `MUST` is an interop note, not a security vulnerability.

## 4. Non-TTY Staging Smoke (`mcpdo` & `mcp-inspector --cli`)

Probe live servers through `mcpdo` and `mcp-inspector --cli` (avoid ad-hoc client scripts):

- **Agent / interactive debugging (`mcpdo`):**
  - Run `mcpdo connect <entry-or-url>` **alone** (never chained with `&&`), because non-TTY OAuth exits `0` with `pendingAuth: true` and `authUrl` before the connection is usable.
  - Always pass `@<name>` or `--connection <name>` in non-TTY shells.
  - Diagnose `401`/`403` token audience, scope, or expiry mismatches via `mcpdo auth/show <name>` (prints unverified claims + relative expiry without leaking the raw token).
  - Force clean re-consent tests with `mcpdo disconnect <name> --clear-auth`.
- **Automated CI post-deploy check (`mcp-inspector --cli`):**
  ```ts
  if (process.env.GITHUB_ACTIONS === "true") {
    console.log(`::add-mask::${accessToken}`);
  }
  const ran = spawnSync(
    inspectorBin,
    [
      "--cli", endpointUrl,
      "--transport", "http",
      "--stored-auth-only",
      "--header", `Authorization: Bearer ${accessToken}`,
      ...args,
    ],
    { encoding: "utf-8", env: { ...process.env, CI: "true" }, stdio: ["ignore", "pipe", "pipe"], timeout: 120_000 },
  );
  ```
  - Assert both structural presence (`tools/list` contains expected tools) and **downstream binding health** on read calls (e.g. vector embedding binding answered rather than silently degrading to keyword search).

## 5. `ui://` MCP Apps Conformance & Bounded Host Qualification

When a server exposes interactive widgets (`@modelcontextprotocol/ext-apps` / `ui://` resources):
1. **Server-side checks:** Verify every UI tool sets `_meta.ui.resourceUri` (avoiding deprecated `_meta["ui/resourceUri"]` unless backward interop is required), `resources/read` serves bundled HTML with valid MIME, `_meta.ui.csp` and `_meta.ui.domain` restrict external origins, app-only helper tools set `visibility: ["app"]`, and the tool result includes a meaningful text/structured fallback for non-UI hosts.
2. **Runtime boundary:** Passing server-side `ui://` checks proves resource metadata and payload shape only—not host sandbox proxy, CSP enforcement, or `postMessage` lifecycle.
3. **Bounded live-host receipts:** When qualifying real hosts (e.g. ChatGPT Web selected-root denial and live grant revocation), use dedicated disposable accounts, apply analytics-isolation blocks before browser navigation, verify an in-scope positive control first, then test outside-root denial, Inbox denial, and post-revocation retry (stopping immediately if the host silently auto-reconnects into a new grant). Record dated, `(hostId, surfaceId, capability)`-scoped observations with sanitized IDs—never upgrade overall host certification from one surface check.
