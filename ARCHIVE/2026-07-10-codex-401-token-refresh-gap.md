# Codex 401 token-refresh: refresh-and-retry on expired OAuth mid-run

## Goal
## Gap (found during codex-reliability explain phase — task `2026-07-07-codex-provider-reliability-after-pi-mono-merge`, cancelled no-op)

Hatfield's codex (openai-codex) provider refreshes its OAuth access token **once at provider build time**, then freezes the token string into `CodexModelClient` for the provider instance's lifetime. A 401 mid-run throws `AuthenticationException` with no refresh-and-retry — so a long-lived `messenger:consume` worker (or any process holding the provider past the token lifetime) fails codex requests even though `CodexAuthStorage` knows how to refresh.

This is **Hatfield auth architecture**, NOT a pi-mono port.

## Evidence (from read-only exploration)
- `src/CodingAgent/Infrastructure/SymfonyAi/SymfonyAiProviderFactory.php` `buildCodexProvider()`: single `codexAuth->loadCredentials($authKey)` call; the returned `$record->access` token string is passed into `CodexModelClient` constructor (static for provider lifetime).
- `CodexAuthStorage::loadCredentials()` already auto-refreshes when `CodexAuthRecord::isExpired(60s buffer)` under flock — but only when *called*. The provider instance never calls it again after construction.
- `src/Platform/Bridge/OpenAICodex/ResultConverter.php:51-52`: a 401 throws `AuthenticationException` — no refresh-and-retry loop.
- `SymfonyAiProviderRegistry` / `ConfiguredSymfonyAiPlatformFactory` cache providers (and tokens) for the container/request lifetime → a long-lived worker holds a stale token.

## Approaches (decide during task-start)
- **Option A (preferred):** refresh in `CodexModelClient` on 401 — inject `CodexAuthStorage` + the provider `auth_key`, and on a 401 response call `loadCredentials()` (which refreshes if expired), update the held token, and retry once. Keep the retry bounded (single retry; avoid loops). Mirror the existing `LlmRetryingHttpClient` "one extra attempt" pattern.
- **Option B:** shorten provider TTL — rebuild providers periodically / per-run so the token is always fresh. Simpler but coarser; doesn't cover a token expiring mid-run.
- Option A gives true request-time resilience; Option B is a cheaper mitigation. Could combine (B as a safety net + A for correctness).

## Scope / non-goals
- Do NOT change the retry/backoff layer (`LlmRetryingHttpClient` / `LlmHttpRetryPolicy`) — that's provider-agnostic and already correct. The refresh belongs in the codex-specific path (it's OAuth/PKCE, codex-only).
- Do NOT port pi-mono transport features (zstd, UA) here — those were explicitly deferred by the user out of the cancelled reliability task.

## Acceptance criteria
- A codex provider whose access token is expired/stale at request time refreshes via `CodexAuthStorage` and retries the request (bounded, no infinite loop) instead of surfacing a hard `AuthenticationException`.
- Focused test proving the 401 → refresh → retry behavior (mock the HTTP layer; assert refresh called once + request retried once with the new token; assert no retry loop on persistent 401). Test thesis: a long-lived worker recovers from an expired token without a manual `auth:codex --refresh`.
- `castor test` + `castor phpstan` + `castor cs-check` green.
- Decision documented: Option A vs B vs combined, with rationale.

## Related
- Parent exploration: `DONE/2026-07-07-codex-provider-reliability-after-pi-mono-merge.md` (cancelled no-op; this task is the one real gap it surfaced).

## Acceptance criteria
- Expired/stale codex OAuth token at request time triggers refresh via CodexAuthStorage + bounded single retry (no hard AuthenticationException, no infinite loop)
- Focused test: 401 → loadCredentials (refresh) called once → request retried once with new token; persistent 401 does not loop
- castor test + castor phpstan + castor cs-check green
- Approach decision (Option A refresh-in-client vs B provider-TTL vs combined) documented with rationale

## Workflow metadata
Status: DONE
Branch: task/2026-07-10-codex-401-token-refresh-gap
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-10-codex-401-token-refresh-gap
Fork run: el3yguan1uiy
PR URL: https://github.com/ineersa/agent-core/pull/277
PR Status: merged
Started: 2026-07-10T21:09:09.220Z
Completed: 2026-07-12T21:31:04.806Z

## Work log
- Created: 2026-07-10T20:37:32.110Z

## Task workflow update - 2026-07-10T21:09:09.221Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-07-10-codex-401-token-refresh-gap.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-07-10-codex-401-token-refresh-gap.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-07-10-codex-401-token-refresh-gap.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-07-10-codex-401-token-refresh-gap.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-07-10-codex-401-token-refresh-gap.
- Summary: task-start. Approach: Option A — refresh-in-client on 401 with bounded single retry (force-refresh via CodexOAuthService::refreshCredentials). New bridge interface AccessTokenRefresherInterface (decouples bridge from App\Auth); App adapter wraps CodexOAuthService; CodexModelClient::request() detects 401, force-refreshes, retries once. Force-refresh (not load-if-expired) chosen because a 401 means the token is bad regardless of local clock-expiry estimate (handles revoked-before-expiry). Option B (provider TTL) rejected — doesn't recover mid-run.

## Task workflow update - 2026-07-10T21:15:30.340Z
- Validation: castor test OK 4217 tests / 13720 assertions (~25.9s); castor test --filter=CodexModelClientTest OK 19/19 (4 new: 401→refresh→retry 200; 401→refresh null→original 401; persistent 401→no loop, 2 requests exactly; no-refresher→401 passthrough backward-compat); castor phpstan 0 errors; castor cs-check 0 issues (after castor cs-fix on 2 files); bin/console lint:container OK (compiles, type-compatible injections); DI verified: debug:container SymfonyAiProviderFactory Arguments[5] = Service(CodexOAuthService) non-null — feature NOT dormant; minor open items for task-to-pr: (1) refreshAndRetryOnce() has unused $failedResponse param (cleanliness); (2) optional integration test asserting container wires CodexOAuthService→factory→refresher (manually verified via debug:container; unit tests cover CodexModelClient behavior)
- Summary: task-start COMPLETE — implementation done, awaits user task-to-pr. Commit 141298c89 on task/2026-07-10-codex-401-token-refresh-gap (6 files, +288/-2). Approach: Option A (refresh-in-client on 401, bounded single retry, force-refresh). New bridge interface AccessTokenRefresherInterface (decouples bridge from App\Auth); App adapter CodexAccessTokenRefresher wraps CodexOAuthService::refreshCredentials() (force-refresh — handles both clock-expired AND revoked-before-expiry, since a 401 means token is bad regardless of local expiry); CodexModelClient::request() detects 401 → refreshAndRetryOnce() (one refresh, one retry, new auth_bearer + new x-client-request-id; guards null/same-token/throw → keeps original 401 for ResultConverter); Factory threads the dep; SymfonyAiProviderFactory wires CodexAccessTokenRefresher when CodexOAuthService present (nullable guard). DI WIRING VERIFIED LIVE: debug:container shows SymfonyAiProviderFactory arg 6 = Service(CodexOAuthService) non-null → CodexAccessTokenRefresher constructed in production. LlmRetryingHttpClient + ResultConverter untouched (401 stays non-retryable in retry layer; ResultConverter 401 throw is the clean fallback).

## Task workflow update - 2026-07-10T21:57:16.740Z
- Summary: task-to-pr: Reviewer APPROVE WITH SUGGESTIONS (no blockers) on commit 0540a35b6. All findings addressed in review-iterate (amended → 96ff4875d). deptrac 0 violations (App→bridge direction allowed), privacy-clean logs, backward-compat verified, reviewer confirmed NO container-wiring integration test needed (debug:container + lint:container + unit tests suffice). 

Review-iterate fixes (fork): (1) SUBSTANTIVE — removed silent try/catch in CodexAccessTokenRefresher (was swallowing refresh failures with no log, making CodexModelClient's codex.token.refresh_failed catch dead code, violating AGENTS.md); adapter now throws, CodexModelClient's catch/log is the LIVE diagnostic path; adapter return narrowed to string (satisfies ?string interface, phpstan). (2) removed redundant $retryOptions['headers']= copy line. (3) renamed $oAuthService→$oAuth. (4) added throw-path test (refresher throws → 1 request, 401 surfaces, codex.token.refresh_failed logged via TestLogger). Interface docblock updated (impls SHOULD throw on failure; caller catches+logs).

Validation: castor test 4218/13727 OK; CodexModelClientTest 20/20 (5 codex-401 tests now); castor phpstan 0; castor cs-check 0; castor deptrac 0 violations; bin/console lint:container OK; DI verified live (SymfonyAiProviderFactory arg 6 = Service(CodexOAuthService) non-null). Approaching move_task CODE-REVIEW.

## Task workflow update - 2026-07-10T21:59:23.423Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (105.0s).
- Pushed task/2026-07-10-codex-401-token-refresh-gap to origin.
- branch 'task/2026-07-10-codex-401-token-refresh-gap' set up to track 'origin/task/2026-07-10-codex-401-token-refresh-gap'.
- Created PR: https://github.com/ineersa/agent-core/pull/277

## Task workflow update - 2026-07-11T02:11:11.157Z
- Synced with main: merged origin/main into task branch (brought in single-writer ToolBatch rework — DBAL→session-scoped SessionToolBatchStore, migration Version20260710120000, cleanup hook subscriber; 55 files). Clean merge, no conflicts.
- Full castor check green (265.3s): deptrac OK, test 4221/13756, controller-replay 8/112, tui 32/163, llm-real 10/121, phpstan 0 errors, cs-check clean, cache guard ok (291→291), no leaks.
- Pushed: 8e679bd97..28af766fe (fast-forward). PR #277 updated with merged state; diff now shows only codex-401-specific changes (401 refresh-and-retry + builder extraction).

## Task workflow update - 2026-07-12T18:21:49.092Z
- Recorded fork run: 6ks4dle9uvag
- Review iteration started for PR #277 comments: approved simplification is to replace AccessTokenRefresherInterface + CodexAccessTokenRefresher adapter with an optional Closure seam, and make CodexSymfonyAiProviderBuilder's CodexAuthStorage/CodexOAuthService dependencies required/non-nullable. Implementation fork 6ks4dle9uvag launched; no push until validation/review.

## Task workflow update - 2026-07-12T18:35:31.225Z
- Validation: Focused Codex client/builder/factory tests: OK (41 tests, 163 assertions).; castor check: OK (251.1s) — test 4220/13754; controller-replay 8/112; TUI 32/163; llm-real 10/121; deptrac, phpstan, cs-check green; cache guard 292→292; no QA process leaks.
- Summary: PR #277 review iteration complete: removed AccessTokenRefresherInterface and CodexAccessTokenRefresher adapter; replaced bridge seam with optional Closure supplied directly by CodexSymfonyAiProviderBuilder; made CodexAuthStorage and CodexOAuthService required/non-nullable builder dependencies; retained missing-credentials and bounded 401 retry behavior. Pushed commit b97f901ac and replied to all four inline review threads.
- Review commit b97f901ac pushed to PR #277 (28af766fe..b97f901ac). All four review threads replied to: stale general-factory coupling comment documented as resolved by 8e679bd97; interface/adapter comments resolved by Closure seam; nullable builder-service comment resolved by required dependencies.

## Task workflow update - 2026-07-12T19:55:35.477Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR review iteration: fix Codex streaming error propagation so non-2xx responses (observed HTTP 404 in session 1) surface as provider errors instead of being consumed as an empty SSE stream. Auth/profile configuration is explicitly out of scope; prior account-selection inference was retracted.

## Task workflow update - 2026-07-12T19:56:20.224Z
- Summary: Expanded current PR review iteration with user-approved Codex HTTP parity: preserve `originator: hatfield`, add `Accept: text/event-stream` and explicit Hatfield `User-Agent`, plus propagate generic non-2xx HTTP responses before SSE conversion. Regression thesis: the observed streaming HTTP 404 must surface as a provider exception with safe diagnostics, never as `empty_response`. Auth/profile configuration remains out of scope.

## Task workflow update - 2026-07-12T20:04:30.003Z
- Summary: Reviewer verdict on a164f9665: REQUEST CHANGES. Blocking privacy issue: generic non-2xx exception currently includes arbitrary provider-controlled message/body content, and LlmPlatformAdapter response diagnostics can independently expose the same field. Required iteration: keep HTTP status and allowlisted structural metadata only; add secret-marker regression. Non-blocking suggestions: verify retry preserves new headers and avoid duplicated `HTTP 404: HTTP 404`.

## Task workflow update - 2026-07-12T20:58:11.102Z
- Validation: A/B minimal Pi-equivalent SSE request with exact Pi bearer: HTTP 404 invalid_request_error param=model; Same SSE request with exact Hatfield bearer: identical HTTP 404; Pi bearer + native Node zstd-compressed identical SSE body: identical HTTP 404; openai-codex/gpt-5.6-luna scout succeeds through Pi default auto transport (WebSocket first)
- Summary: Root cause of GPT-5.6 Luna HTTP 404 isolated: Codex backend rejects gpt-5.6-luna on SSE transport, independent of account/token, OAuth originator, request identity, session headers, request body, and zstd compression. Pi succeeds because openai-codex transport defaults to WebSocket; Hatfield currently supports SSE/HTTP only. Temporary 3-file Pi identity/session diagnostic remains uncommitted and should be reverted after user approval. PR-scope c6021e9f1 generic non-2xx/error privacy/header fixes remain valid.

## Task workflow update - 2026-07-12T21:18:29.559Z
- Recorded fork run: el3yguan1uiy
- Validation: castor test --filter='CodexModelClientTest|ResultConverterTest|CodexSymfonyAiProviderBuilderTest|SymfonyAiProviderFactoryTest': 91 tests, 400 assertions OK; castor deptrac: 0 violations, 0 errors; castor phpstan: 0 errors; castor cs-check: 0 files
- Summary: Temporary three-file Pi identity/OAuth/session diagnostic reverted exactly to committed HEAD c6021e9f1. Worktree is clean. The WebSocket requirement is tracked separately in TODO/2026-07-12-add-codex-websocket-transport.md; PR #277 remains scoped to bounded 401 refresh, Codex builder decoupling, SSE headers, and privacy-safe non-2xx error propagation.

## Task workflow update - 2026-07-12T21:20:19.967Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (101.0s).
- Pushed task/2026-07-10-codex-401-token-refresh-gap to origin.
- branch 'task/2026-07-10-codex-401-token-refresh-gap' set up to track 'origin/task/2026-07-10-codex-401-token-refresh-gap'.
- PR already exists: https://github.com/ineersa/agent-core/pull/277
- Validation: Focused Codex/provider tests: 91 tests, 400 assertions OK; Deptrac: 0 violations/errors; PHPStan: 0 errors; CS check: clean
- Summary: Review iteration cleaned and ready: reverted all temporary Pi parity diagnostics; retained committed 401 refresh, architecture decoupling, SSE headers, and privacy-safe generic non-2xx propagation through c6021e9f1. WebSocket support split to separate TODO task.

## Task workflow update - 2026-07-12T21:31:04.806Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-07-10-codex-401-token-refresh-gap into integration checkout.
- Auto-merging config/services.yaml
Auto-merging src/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapter.php
Merge made by the 'ort' strategy.
 config/services.yaml                               |   7 +
 .../SymfonyAi/LlmPlatformAdapter.php               |  12 +-
 .../Codex/CodexSymfonyAiProviderBuilder.php        | 109 +++++++++++
 .../SymfonyAiProviderBuilderInterface.php          |  22 +++
 .../SymfonyAi/SymfonyAiProviderFactory.php         | 107 ++---------
 .../Bridge/OpenAICodex/CodexModelClient.php        |  59 ++++++
 src/Platform/Bridge/OpenAICodex/Factory.php        |   6 +-
 .../Bridge/OpenAICodex/ResultConverter.php         |  56 ++++++
 .../SymfonyAi/LlmPlatformAdapterTest.php           |  38 ++++
 .../CodexSymfonyAiProviderBuilderTest.php}         |  98 +++++++---
 .../SymfonyAi/SymfonyAiProviderFactoryTest.php     |  81 ++++----
 .../SymfonyAi/SymfonyAiProviderRegistryTest.php    |   6 +-
 .../Bridge/OpenAICodex/CodexModelClientTest.php    | 214 +++++++++++++++++++++
 .../Bridge/OpenAICodex/ResultConverterTest.php     |  47 +++++
 14 files changed, 695 insertions(+), 167 deletions(-)
 create mode 100644 src/CodingAgent/Infrastructure/SymfonyAi/Codex/CodexSymfonyAiProviderBuilder.php
 create mode 100644 src/CodingAgent/Infrastructure/SymfonyAi/SymfonyAiProviderBuilderInterface.php
 rename tests/CodingAgent/Infrastructure/SymfonyAi/{SymfonyAiProviderFactoryCodexAuthTest.php => Codex/CodexSymfonyAiProviderBuilderTest.php} (82%)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-07-10-codex-401-token-refresh-gap.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-07-10-codex-401-token-refresh-gap.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #277 state MERGED at 2026-07-12T21:30:37Z; Pre-merge deterministic castor check passed in 101.0s; Focused Codex/provider tests: 91 tests, 400 assertions OK; Deptrac, PHPStan, and CS check clean
- Summary: PR #277 merged as a1eb995bdc371a06af124a2f5f9c2b95a2b36abd. Delivered bounded request-time Codex OAuth refresh on 401, Codex-specific provider builder decoupling, explicit SSE Accept/User-Agent headers, and privacy-safe generic non-2xx error propagation. GPT-5.6 WebSocket transport remains separately tracked in TODO/2026-07-12-add-codex-websocket-transport.md.
