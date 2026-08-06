# Codex provider reliability: test + port meaningful pi-mono codex transport features (no websockets)

## Goal
## Context

After merging pi-mono `origin/main` into `develop` (commit `054f5065`), the **codex provider catalog is unchanged** (gpt-5.5/5.4/5.4-mini — same prices/context hatfield already has). But the pi-mono codex **implementation** (`packages/ai/src/api/openai-codex-responses.ts`, 1568 lines) gained real reliability logic that develop did not have. Hatfield has its **own** PHP codex client and **no websockets**, so nothing auto-ports — but several features are worth mirroring.

User note: codex hasn't been tested in hatfield for a long time. So this is **exploratory + test first, then port what's meaningful**.

## pi-mono codex impl features gained vs pre-merge develop

| Feature | Port to hatfield? |
|---|---|
| zstd request compression (`zstdCompressSync`, level 3) on SSE responses endpoint | Candidate — check if hatfield's HTTP client can send compressed bodies + if z.ai/openai-codex backend honors it |
| Retry/backoff: terminal rate-limit detection, retryable status, `Retry-After` header parsing, capped exponential | **Likely worth mirroring** — reliability baseline |
| SSE header stall protection (develop had custom 20s `createSSEHeaderTimeout`; main refactored to unified `httpTimeoutMs` via `AbortSignal.timeout`) | Verify hatfield has equivalent stall guard; add if missing |
| user-agent race fix | Already in develop's base; verify hatfield equivalent |
| Error-body passthrough (`formatProviderError`/`normalizeProviderError` — surface provider HTTP error body instead of opaque message) | **Worth mirroring** — better failure messages |
| `before_provider_headers` / `ProviderEnv` / `ProviderHeaders` hooks | Extension hook — out of scope unless hatfield has an equivalent seam |
| getBuiltinModule (node 23.6 compat) | N/A — pi-mono/TS-only |
| WebSocket connection-limit reconnect (`websocket_connection_limit_reached`) | **N/A** — hatfield has no websockets |
| Stale session rotation (`SESSION_WEBSOCKET_MAX_AGE_MS` 55min reaping) | **N/A** — hatfield has no websockets |

## Work to do

1. **Test codex first** (it's been a while). Live smoke a real codex run (gpt-5.5 or gpt-5.4) through hatfield's `openai-codex` provider: confirm auth (OAuth PKCE via `bin/console auth:codex`), a request reaches the model, tool/text output returns. Record whether it currently works end-to-end.
2. For each candidate feature above, decide: **already have / port / N/A**. Hatfield already has a custom SSE header timeout in develop's equivalent code — confirm it survived and is still wired. Prioritize: retry/backoff with rate-limit + Retry-After handling, and error-body passthrough (biggest UX win for debugging).
3. Port chosen features with focused tests. Skip websocket-only features entirely.
4. Confirm codex models (5.5/5.4/5.4-mini) prices/context unchanged vs pi-mono.

## Reference — pi-mono codex impl structure
`packages/ai/src/api/openai-codex-responses.ts`: `isTerminalRateLimitError`, `isRetryableError`, `getRetryAfterDelayMs`, `capRetryDelayMs`, `compressRequestBodyZstd`, `buildRequestBody`, `resolveCodexServiceTier`, `processStream`, `mapCodexEvents`, `parseSSE`. Service-tier cost multiplier + pricing applied (`applyServiceTierPricing`).

## Related
Sister task `2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge` owns the z.ai/GLM resolver work. This task owns codex only.

## Acceptance criteria
- Codex (openai-codex provider, gpt-5.5/5.4) smoke-tested live: a real run reaches the model and returns tool/text output (or a clear failure is recorded with root cause)
- Decision documented per pi-mono reliability feature (zstd compression, retry/backoff for retryable status + terminal rate-limit, SSE header timeout, user-agent) on whether hatfield already has it / should add it / is N/A (hatfield has no websockets so connection-limit-reconnect and stale-session rotation are N/A)
- Any reliability gaps hatfield lacks that pi-mono gained are either ported with a focused test, or explicitly scoped out with rationale
- Codex prices/context (gpt-5.5/5.4/5.4-mini) confirmed unchanged vs pi-mono catalog
- Focused Castor validation passes for any code change (castor test + castor phpstan + castor cs-check)

## Workflow metadata
Status: DONE
Resolution: Cancelled (no-op) — explain-phase exploration found pi-mono reliability features already exist in Hatfield's shared LlmHttp layer; nothing meaningful to port. See work log. Real gap (401 token-refresh) filed as separate task `2026-07-10-codex-401-token-refresh-gap`.
Branch: (none — read-only exploration)
Worktree: (none)
Fork run: (read-only explain-phase scout fork, 2026-07-10)
PR URL: (none — no code change)
PR Status: n/a
Started:
Completed: 2026-07-10

## Work log
- Created: 2026-07-07T22:45:47.755Z
- 2026-07-10 (explain phase, read-only scout fork): Traced the full codex transport wiring. **Conclusion: codex does NOT bypass Hatfield's reliability layer** — `CodexModelClient` is wrapped by the shared `LlmRetryingHttpClient` + `LlmHttpRetryPolicy` in production wiring (`SymfonyAiProviderFactory.php:152`, `getHttpClient()`). Most pi-mono features are ALREADY implemented there.

  **Feature matrix (pi-mono → Hatfield codex):**
  - Retry/backoff + `Retry-After` + terminal rate-limit + capped backoff → **ALREADY-HAVE** (`LlmRetryingHttpClient` + `LlmHttpRetryPolicy`; retries 408/425/429/5xx; parses `retry-after-ms`/seconds/HTTP-date; backoff capped 60s; well-tested).
  - Error-body passthrough → **PARTIAL (by design)** — provider JSON errors land in exception message + structured `response_error_*` diagnostics; raw body hidden from user (privacy). Intentional, not a gap.
  - SSE header/stall timeout → **PARTIAL** — Symfony `timeout` 30s + `max_duration` 120s; no separate first-byte timer; codex buffers the full SSE body before parse (`CodexSseStream`).
  - zstd compression → **GAP** (deferred per user — real win for large bodies, backend honors it per pi-mono comment `openai-codex-responses.ts:62`, but needs ext-zstd/composer dep; not in scope this round).
  - User-Agent → **GAP** (deferred per user — trivial one-liner; pi-mono's "race fix" is Node-async-specific, no PHP analog).
  - WebSocket reconnect + stale session rotation → **N/A** (Hatfield codex = HTTP POST + SSE-body parse only).

  **One real gap (NOT a pi-mono port):** 401 token-refresh — OAuth refresh happens once at provider build time (`SymfonyAiProviderFactory::buildCodexProvider`), token frozen into `CodexModelClient` for provider lifetime; a 401 mid-run throws `AuthenticationException` (`ResultConverter.php:51-52`) with no refresh-and-retry. Bites long-lived messenger workers. Filed as separate task `2026-07-10-codex-401-token-refresh-gap`.

  **Decision:** Cancelled as no-op per user. No branch/worktree/PR (exploration was read-only).
