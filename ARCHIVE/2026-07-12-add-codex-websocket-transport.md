# Add Codex WebSocket transport for GPT-5.6

## Goal
## Context

Hatfield's OpenAI Codex bridge currently supports HTTP/SSE only. GPT-5.6 models (`gpt-5.6-luna`, `gpt-5.6-sol`, `gpt-5.6-terra`) are rejected by the Codex SSE endpoint with HTTP 404 `invalid_request_error` / `param=model`, while the same model/account succeeds through pi-mono's default WebSocket transport.

Diagnostic evidence (2026-07-12):
- Exact Pi bearer + minimal Pi-equivalent uncompressed SSE body → HTTP 404 `invalid_request_error/model`.
- Exact Hatfield bearer + identical SSE body → same 404.
- Exact Pi bearer + native Node zstd-compressed identical SSE body → same 404.
- Pi itself with `transport: sse` reports `Model not found gpt-5.6-luna`.
- Pi scout using `openai-codex/gpt-5.6-luna` succeeds under default `auto` transport (WebSocket first).

This rules out account/profile selection, bearer/account pairing, OAuth client/scope/originator, request originator/User-Agent, Accept header, session/request IDs, prompt cache key, body shape, and zstd. GPT-5.6 requires WebSocket transport in the observed backend behavior.

## Feasibility

PHP can implement this. Preferred dependency candidate: `amphp/websocket-client` (PHP 8.1+, maintained, WSS, custom handshake headers, cancellation, iterable messages). The repo already has `revolt/event-loop`, which Amp 3 uses, but does not currently have a WebSocket client dependency. Confirm Composer compatibility before implementation.

## Proposed scope

1. Add a Codex-specific transport abstraction under the OpenAICodex bridge/Codex app namespace; general Symfony AI factory remains implementation-agnostic.
2. Add WebSocket handshake to the resolved Codex URL (`https`→`wss`) with bearer, `chatgpt-account-id`, `originator`, User-Agent, `session-id`, and `x-client-request-id` headers. Do not send `OpenAI-Beta` on the WS handshake, matching pi-mono.
3. Send the request as one JSON frame: `{ "type": "response.create", ...requestBody }`.
4. Consume Codex response events until `response.completed`; map the same event objects through the existing result conversion path rather than duplicating response semantics.
5. Propagate structured provider errors, handshake failures, premature close, malformed frames, cancellation, connect timeout, and idle timeout. Never convert them to `empty_response`.
6. Add provider transport setting with at least `auto`, `websocket`, and `sse`; default `auto`. In `auto`, attempt WebSocket first and fall back to SSE only when failure occurs before any response event. Never replay after partial output.
7. Preserve bounded 401 refresh semantics for authentication failures where the WebSocket library exposes handshake status; refresh once and retry once, never loop.
8. Initial implementation may use one WebSocket connection per LLM request. Session-level connection caching, continuation via `previous_response_id`, proxy support, and WebSocket reconnect-after-partial-stream are explicitly deferred unless separately approved.
9. Update `.hatfield/settings.yaml` and `docs/settings.md` together for new settings.
10. Remove/revert temporary Pi-identity/session diagnostic edits from the 401 task separately; do not build the feature on diagnostic identity changes.

## Architecture risks / decisions

- The current Symfony AI bridge returns `RawHttpResult`; WebSocket events need a clean bridge result/stream boundary without pretending they are an HTTP response.
- Amp/Revolt integration must not leave event-loop watchers, sockets, fibers, or workers alive after success, error, cancellation, or timeout.
- Cancellation must bridge Hatfield's cancellation token to Amp cancellation without adding app-specific dependencies to the generic Symfony AI factory.
- Decide whether `auto` is provider-global or per-provider config; prefer per-provider compatibility with future transports.

## Validation thesis

The fix is proven only if a live Hatfield request to `openai-codex/gpt-5.6-luna` succeeds over WebSocket. Mocked/replay tests alone cannot prove backend compatibility.

Related task/PR: `2026-07-10-codex-401-token-refresh-gap`, PR #277.

## Acceptance criteria
- A live Hatfield `openai-codex/gpt-5.6-luna` request succeeds over WebSocket and returns assistant output or tool calls instead of HTTP 404/model.
- Codex `transport: sse` reproduces the current model-not-found behavior clearly; `transport: websocket` uses only WebSocket; `transport: auto` attempts WebSocket first.
- Auto fallback to SSE occurs only before the first WebSocket response event; no duplicate request is sent after partial output.
- WebSocket handshake uses the exact current bearer/account pair and required Codex headers; request frame is `response.create` with the existing normalized body.
- Existing Codex event conversion is reused for text, reasoning, function calls, usage, completion, and structured errors without a second divergent mapper.
- Connect timeout, idle timeout, cancellation, provider error event, malformed frame, and premature close surface as provider errors with privacy-safe diagnostics.
- Any 401 refresh/retry remains bounded to one refresh and one retry; no retry loops.
- No WebSocket connection, event-loop watcher, fiber, or timer leaks after success/failure/cancellation.
- General `SymfonyAiProviderFactory` has zero Codex/WebSocket implementation knowledge; Codex specifics remain in Codex namespaces.
- `.hatfield/settings.yaml` and `docs/settings.md` document transport behavior and defaults.
- Focused tests follow `tests/AGENTS.md` and the testing skill; provider-visible changes pass focused Castor validation and `castor test:llm-real`, then full `castor check` before review.

## Workflow metadata
Status: DONE
Branch: task/2026-07-12-add-codex-websocket-transport
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-12-add-codex-websocket-transport
Fork run: a0k6c6nkslf7
PR URL: https://github.com/ineersa/agent-core/pull/285
PR Status: merged
Started: 2026-07-12T21:44:19.359Z
Completed: 2026-07-13T17:36:26.847Z

## Work log
- Created: 2026-07-12T21:12:51.567Z

## Task workflow update - 2026-07-12T21:43:43.843Z
- Summary: Task-explain scope revision approved by user: support explicit `websocket | sse` only; remove `auto` mode and all WS→SSE fallback behavior. Default Codex transport should be `websocket` so the tracked GPT-5.6 catalog works; `sse` remains an explicit diagnostic/legacy path. Initial WebSocket implementation opens one connection per request. Connection caching and `previous_response_id` continuation moved to separate follow-up task `2026-07-12-codex-websocket-session-cache-continuation`. Correct current pi-mono handshake parity: WebSocket MUST send `OpenAI-Beta: responses_websockets=2026-02-06` and MUST NOT send SSE headers `Accept: text/event-stream`, `Content-Type: application/json`, or `OpenAI-Beta: responses=experimental`. Keep bounded pre-output error handling, but with no transport fallback: connect/upgrade/timeout/protocol failures surface directly; structured provider errors and any post-event failure also surface directly. Temporary diagnostics were already removed in completed PR #277.
- 2026-07-12 task-explain decisions: user chose `websocket | sse` without `auto`; one connection per request; caching/continuation split out; privacy/error classification accepted; current WebSocket beta header parity approved. Existing task-body references to `auto`, SSE fallback, and 'Do not send OpenAI-Beta' are superseded by this scope revision.

## Task workflow update - 2026-07-12T21:44:19.359Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-07-12-add-codex-websocket-transport.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-07-12-add-codex-websocket-transport.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-07-12-add-codex-websocket-transport.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-07-12-add-codex-websocket-transport.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-07-12-add-codex-websocket-transport.
- Summary: Starting approved implementation with revised scope: explicit `websocket | sse`, WebSocket default, one connection per request, no auto fallback, current Codex WebSocket beta-header parity. Connection caching and previous_response_id continuation remain separate follow-up task.

## Task workflow update - 2026-07-12T21:48:27.993Z
- Recorded fork run: 4q3mtppfolv2
- Summary: Implementation fork launched in task worktree with revised explicit websocket|sse scope, Amp v2 dependency verification, transport-neutral request mapping, non-HTTP RawResult boundary, generic cancellation seam, focused tests/docs, and no auto/cache/continuation. No push/PR/full gate authorized in task-start.

## Task workflow update - 2026-07-12T21:58:11.015Z
- Recorded fork run: otglx8z81wx9
- Summary: Parent verification rejected first implementation handoff pending narrow fixes: terminal response must stop receiving immediately (server may keep WS open); send failure must close socket; cleanup catches need structured logging and idempotence; idle timeout/non-text frames need explicit errors; stream provider messages must be sanitized; project-owned cancellable result contract must use Ineersa namespace. Follow-up fork launched to amend unpushed commit.

## Task workflow update - 2026-07-12T23:15:41.624Z
- Recorded fork run: zaqzh3ewq8jx
- Summary: Final task-start cleanup fork launched after parent verification: remove environment-dependent real localhost:9 WebSocket timeout test and stale duplicate ResultConverter docblock, then amend unpushed commit with focused Castor QA.

## Task workflow update - 2026-07-12T23:17:31.038Z
- Recorded fork run: zaqzh3ewq8jx
- Validation: Focused Codex WebSocket/ResultConverter tests: 59 tests, 269 assertions — OK; Earlier broader Codex-focused suite after implementation: 60 tests, 271 assertions — OK; castor deptrac — 0 violations; castor phpstan — 0 errors; castor cs-check — 0 fixable files; castor test:llm-real — 10 tests, 121 assertions — OK; Worktree clean; commit ef89b5c1509d0486a112c13c946347919dcdc8dc verified; Not run by design in task-start: full castor check; production openai-codex/gpt-5.6-luna WebSocket smoke
- Summary: Task-start implementation complete and parent-verified at commit ef89b5c1509d0486a112c13c946347919dcdc8dc; worktree clean. Added explicit websocket|sse Codex transports (websocket default), Amp v2 one-connection-per-request transport, shared SSE/WS request-body factory, current Codex WS handshake/frame protocol, bounded handshake 401 refresh, streaming RawWebSocketResult with terminal/timeout/protocol/resource cleanup, generic cancellable raw-result contract, privacy-safe stream diagnostics, provider config/docs, and focused tests. Removed environment-dependent localhost connector test. No push/PR/full castor check or production GPT-5.6 live proof in task-start.

## Task workflow update - 2026-07-12T23:53:12.832Z
- Summary: Reviewer verdict on ef89b5c15: REQUEST CHANGES. Actionable blockers: restore/update non-obvious rationale comments lost during Codex request-body extraction; force response.create frame type to override any body type; close RawWebSocketResult deterministically if never iterated; add bounded WebSocket handshake 401 refresh/retry tests; remove dead Symfony AI Result PSR-4 mapping. Also address reasonable cleanup: single parse_url, final WS client, dead test variable, default-websocket selection proof, remove meaningless always-true request log flag. Privacy/resource/Amp API architecture otherwise reviewed positively.
- task-to-pr reviewer read AGENTS.md, testing skill, tests/AGENTS.md, task file, and Amp v2 vendor sources; returned REQUEST CHANGES on ef89b5c15. Live GPT-5.6 proof remains manual acceptance, not an automated-test blocker.

## Task workflow update - 2026-07-12T23:57:12.630Z
- Recorded fork run: apunmvsyga23
- Validation: Review-fix focused tests: 98 tests, 421 assertions — OK; castor deptrac — 0 violations; castor phpstan — 0 errors; castor cs-check — 0 fixable files; Worktree reported clean at 60bbdf7b4
- Summary: Review-fix fork completed at new commit 60bbdf7b4cc868075b8d9f9f4b535b6c66597619: restored request/retry/privacy rationale comments; forced response.create type; added deterministic unconsumed-result close; added bounded WS handshake 401 tests; removed dead PSR-4 mapping; simplified logging and removed dead test code. Default-selection NTH intentionally skipped because no public provider inspection API and enum/default mirror tests are prohibited. Awaiting reviewer re-check.

## Task workflow update - 2026-07-13T00:09:03.727Z
- Recorded fork run: 7pa1dpewksw9
- Validation: Reviewer re-check of 60bbdf7b4: APPROVED; no blockers; Final current-HEAD reviewer at 1c958145b: APPROVED; exact two-line comment-only delta verified; castor test initial run: 1 unrelated SubagentLivePickerControllerTest failure (3808 tests/12578 assertions); exact focused rerun passed 1 test/5 assertions; castor test clean rerun: 4349 tests, 14364 assertions — OK; castor deptrac — 0 violations, 0 errors; castor phpstan — 0 errors; castor cs-check — 0 fixable files; castor test:llm-real — 10 tests, 121 assertions — OK; Git worktree clean at 1c958145b388cdc18bae729c0a99e678570de040; Production openai-codex/gpt-5.6-luna WebSocket manual smoke remains unrun
- Summary: Final reviewer verdict on current HEAD 1c958145b388cdc18bae729c0a99e678570de040: APPROVED with no blockers. All prior review findings resolved; final comment-only amend restored accurate bare-HttpClient/CodexSseStream rationale. Worktree clean and ready for deterministic CODE-REVIEW gate.

## Task workflow update - 2026-07-13T00:14:06.219Z
- Validation: First move_task deterministic castor check: FAILED only test:tui exit 124 (lane timeout); Standalone castor test:tui rerun: 36 tests, 186 assertions — OK in 107.8s; No process/session cleanup performed; timed-out gate leak recorded
- Summary: First CODE-REVIEW transition gate failed only because test:tui lane hit exit 124 at the 120s lane cap. Gate log showed ParaTest had progressed into final tree-rewind session; standalone isolated `castor test:tui` then passed 36 tests/186 assertions in 107.8s. The timed-out gate left its current-user tmux/controller/worker tree alive under session `tui-tree-rewind-07c-qa-20260713-000912-451541-4fe468fd-451858-0`; it was recorded and deliberately not signaled/cleaned without user authorization. Retrying deterministic gate with warm caches.

## Task workflow update - 2026-07-13T00:16:14.663Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (116.2s).
- Pushed task/2026-07-12-add-codex-websocket-transport to origin.
- branch 'task/2026-07-12-add-codex-websocket-transport' set up to track 'origin/task/2026-07-12-add-codex-websocket-transport'.
- Created PR: https://github.com/ineersa/agent-core/pull/285

## Task workflow update - 2026-07-13T00:16:20.562Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/285
- Updated PR Status: open
- Validation: Deterministic castor check retry — PASSED in 116.2s; Branch task/2026-07-12-add-codex-websocket-transport pushed to origin; PR #285 created: https://github.com/ineersa/agent-core/pull/285
- Summary: Moved to CODE-REVIEW after final reviewer APPROVED and focused validation. Second deterministic castor check passed in 116.2s; branch pushed and PR #285 created. Production openai-codex/gpt-5.6-luna manual WebSocket smoke remains an acceptance follow-up.

## Task workflow update - 2026-07-13T01:19:56.163Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Review iteration authorized by user after live A/B isolated Codex routing requirement: UUIDv4 correlation/cache IDs return invalid_request_error/model with internal alias gpt-5.6-luna-free-1p-codexswic-ev3; UUIDv7 succeeds. Implement focused UUIDv7 correction, regression coverage, and live proof.

## Task workflow update - 2026-07-13T02:01:52.382Z
- Recorded fork run: tzsdf85e8hpr
- Validation: Focused Codex tests: 33 tests, 189 assertions OK; castor phpstan: 0 errors; castor cs-check: 0 fixable files; castor deptrac: 0 violations; castor test:llm-real: 10 tests, 121 assertions OK (after initial UUIDv7 fix); Live openai-codex/gpt-5.6-luna WebSocket diagnostic: success, assistant text OK, response.completed; Final reviewer at 2a494a4d6: APPROVED
- Summary: UUIDv7 live-routing iteration completed across commits fad28127e, 25ad1023e, and 2a494a4d6. Generated Codex correlation IDs are UUIDv7 and align session-id/x-client-request-id/prompt_cache_key; generated IDs rotate coherently on bounded 401 retry; explicit run_id/prompt_cache_key values remain stable; empty prompt_cache_key falls back to generated UUIDv7. Privacy-safe live production WebSocket smoke for gpt-5.6-luna returned assistant text OK and response.completed. Final reviewer APPROVED with no blockers.

## Task workflow update - 2026-07-13T02:04:04.208Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (120.0s).
- Pushed task/2026-07-12-add-codex-websocket-transport to origin.
- branch 'task/2026-07-12-add-codex-websocket-transport' set up to track 'origin/task/2026-07-12-add-codex-websocket-transport'.
- PR already exists: https://github.com/ineersa/agent-core/pull/285
- Validation: Focused Codex tests 33/189 OK; phpstan 0 errors; cs-check 0 fixable; deptrac 0 violations; test:llm-real 10/121 OK; Live gpt-5.6-luna WebSocket smoke OK; Reviewer APPROVED
- Summary: Final UUIDv7 routing iteration approved and ready to update PR #285. Generated IDs align across headers/body and retry; explicit IDs preserved; live gpt-5.6-luna proof succeeded.

## Task workflow update - 2026-07-13T02:07:01.301Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User authorized another PR #285 review iteration for prompt-cache stability. Root cause: LlmPlatformAdapter supplies stable run_id, but DynamicToolDescriptionProcessor removes it after toolset resolution, causing Codex to synthesize a new prompt_cache_key per turn and defeating cross-turn provider prompt caching.

## Task workflow update - 2026-07-13T02:29:12.966Z
- Recorded fork run: op0g6hp2g7pw
- Validation: Focused processor + Codex tests: 45 tests, 237 assertions OK; castor phpstan: 0 errors; castor cs-check: 0 fixable files; castor deptrac: 0 violations; Filtered castor test:llm-real LlamaCppSmokeTest: 1 test, 8 assertions OK; Full castor test:llm-real attempt failed in live controller E2E with partial event streams/missing Messenger DB; deterministic full gate will rerun during CODE-REVIEW transition; Final reviewer at 7584e08c3: APPROVED
- Summary: Prompt-cache stability iteration committed as 04e52d676 plus generic rationale correction 7584e08c3. DynamicToolDescriptionProcessor now preserves stable run_id after both active and empty toolset resolution while stripping resolver-only tools_ref/turn_no. Codex derives stable cross-turn prompt_cache_key and strips run_id from wire JSON. Final reviewer APPROVED; generic-provider unknown run_id behavior classified pre-existing/nonblocking.

## Task workflow update - 2026-07-13T14:32:29.686Z
- Recorded fork run: prwjm72vc1v9
- Validation: Focused Castor tests: 58 tests, 263 assertions OK; Migration executor tests: 5 tests, 25 assertions OK; castor phpstan: 0 errors; castor deptrac: 0 violations; castor cs-check: OK; castor test:llm-real run 1: exit 0; castor test:llm-real run 2: exit 0; llama-proxy cache entries stabilized 325→326→326; Reviewer on HEAD 201610657: APPROVE WITH SUGGESTIONS, no blockers
- Summary: Final prompt-cache/correlation iteration complete at HEAD 201610657. Persisted Hatfield sessions now have immutable UUIDv7 provider_cache_key metadata; Codex uses it consistently for session-id, x-client-request-id, and prompt_cache_key while generic providers strip both run_id and provider_cache_key. Existing sessions are backfilled with distinct UUIDv7 values. Ephemeral child/default runs use UUIDv7 run IDs without requiring a DB session row. Final reviewer APPROVE WITH SUGGESTIONS; no blockers.

## Task workflow update - 2026-07-13T14:40:14.742Z
- Recorded fork run: si0stn7t1xn8
- Validation: Focused TuiRepairCommandE2eTest: 1 test, 3 assertions OK; Full castor test:tui: 36 tests, 186 assertions OK; castor cs-check: 0 fixable files
- Summary: Deterministic TUI gate fixture failure fixed in 28e2f0aa3: post-migration manual hatfield_session insert now supplies valid UUIDv7 provider_cache_key, preserving the non-null ORM invariant. Production behavior unchanged.

## Task workflow update - 2026-07-13T14:42:16.637Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (111.2s).
- Pushed task/2026-07-12-add-codex-websocket-transport to origin.
- branch 'task/2026-07-12-add-codex-websocket-transport' set up to track 'origin/task/2026-07-12-add-codex-websocket-transport'.
- PR already exists: https://github.com/ineersa/agent-core/pull/285
- Validation: Focused tests 58/263 OK; Migration tests 5/25 OK; phpstan/deptrac/cs-check green; llm-real twice green; cache stable 325→326→326; Full TUI 36/186 OK; Reviewer APPROVE WITH SUGGESTIONS; no blockers
- Summary: PR #285 iteration complete: Codex WebSocket transport, UUIDv7 provider cache identity, stable cross-turn caching, generic-provider sanitation, migration/backfill, ephemeral child-run support, and corrected post-migration TUI fixture.

## Task workflow update - 2026-07-13T15:09:21.533Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Addressing PR #285 inline feedback: simplify Codex transport parsing and replace loose session metadata arrays with direct typed HatfieldSession reads, while keeping all production mutation/flush behavior inside HatfieldSessionStore.

## Task workflow update - 2026-07-13T15:16:49.137Z
- Recorded fork run: o3uwfsjvaxx2
- Validation: Focused Castor tests: 96 tests, 296 assertions OK; Focused regression pair: 2 tests, 4 assertions OK; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: 0 fixable; castor test:tui: 36 tests, 186 assertions OK
- Summary: PR review iteration commit bf5ff2bf0: centralized nullable/blank Codex transport parsing; replaced loose session metadata array reads with direct typed HatfieldSession findSession APIs across production, tests, and docs. Production mutation remains routed through stores; no DTO/compatibility alias.

## Task workflow update - 2026-07-13T15:28:25.291Z
- Recorded fork run: ue6pavsyhyug
- Validation: Focused Castor tests: 40 tests, 151 assertions OK; castor phpstan: 0 errors; castor cs-check: 0 fixable
- Summary: Reviewer follow-up commit 83b3dd83b fixes stale TraceReplayTest array assertions, removes duplicate session-id assertion, adds narrow non-null test guards, and updates session-storage docs for direct ?HatfieldSession reads. No production changes.

## Task workflow update - 2026-07-13T15:36:10.858Z
- Summary: Reviewer re-review still REQUEST CHANGES: production design and prior fixes are sound, but several tests accessing ?HatfieldSession remain without explicit assertNotNull guards. A final systematic test-only guard pass is required for clean failure diagnostics.

## Task workflow update - 2026-07-13T15:37:43.686Z
- Recorded fork run: a0k6c6nkslf7
- Validation: Focused Castor tests: 71 tests, 288 assertions OK; Focused castor test:llm-real LlamaCppSmokeTest: 1 test, 9 assertions OK; castor phpstan: 0 errors; castor cs-check: 0 fixable
- Summary: Final test-only guard audit commit d4ab37df5 adds explicit assertNotNull before all changed findSession entity property accesses. No production/docs behavior changes. Parent notes fork handoff did not fully satisfy mandatory docs-read reporting; final reviewer is instructed to verify the tiny diff against full testing conventions before acceptance.

## Task workflow update - 2026-07-13T15:46:05.289Z
- Summary: Final reviewer APPROVED commits bf5ff2bf0 + 83b3dd83b + d4ab37df5. Verified direct entity-read design, store-controlled mutations, nullable Codex transport parsing, complete old API removal, docs alignment, and systematic test guards. No blockers; only NTH consistency suggestions.

## Task workflow update - 2026-07-13T15:48:11.256Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (115.4s).
- Pushed task/2026-07-12-add-codex-websocket-transport to origin.
- branch 'task/2026-07-12-add-codex-websocket-transport' set up to track 'origin/task/2026-07-12-add-codex-websocket-transport'.
- PR already exists: https://github.com/ineersa/agent-core/pull/285
- Validation: Reviewer final verdict: APPROVED, no blockers; Focused session/model/Codex tests: 96 tests, 296 assertions OK; Review fixes: 40 tests, 151 assertions OK; Systematic guard audit: 71 tests, 288 assertions OK; Focused llm-real LlamaCppSmokeTest: 1 test, 9 assertions OK; castor test:tui: 36 tests, 186 assertions OK; castor deptrac/phpstan/cs-check green
- Summary: Addressed PR #285 inline feedback: Codex transport parsing is centralized and readable; session metadata reads now return typed HatfieldSession entities directly with no DTO conversion or associative-array mapping; production mutations remain store-controlled; tests/docs migrated and reviewed.

## Task workflow update - 2026-07-13T17:36:26.847Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-07-12-add-codex-websocket-transport into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/settings.yaml                            |    1 +
 composer.json                                      |    3 +-
 composer.lock                                      | 1503 +++++++++++++++++++-
 docs/session-storage.md                            |   67 +-
 docs/settings.md                                   |   10 +
 migrations/Version20260713120000.php               |   42 +
 src/AgentCore/Application/Pipeline/AgentRunner.php |    4 +-
 .../SymfonyAi/DynamicToolDescriptionProcessor.php  |    6 +-
 .../SymfonyAi/LlmPlatformAdapter.php               |    5 +-
 .../Agent/Execution/SubagentExecutionService.php   |    6 +-
 src/CodingAgent/Config/Ai/AiProviderConfig.php     |    3 +
 src/CodingAgent/Config/ModelResolver.php           |    8 +-
 .../Config/SessionAwareModelResolver.php           |   37 +
 src/CodingAgent/Config/SessionMetadataStore.php    |    9 +-
 src/CodingAgent/Entity/HatfieldSession.php         |   13 +-
 .../Codex/CodexSymfonyAiProviderBuilder.php        |    4 +
 .../SymfonyAi/SymfonyAiProviderFactory.php         |   11 +-
 .../Migrations/ApplicationMigrationExecutor.php    |    1 +
 src/CodingAgent/Session/HatfieldSessionStore.php   |   56 +-
 .../Generic/GenericProviderInternalOptionKeys.php  |   33 +
 .../Bridge/Generic/SanitizedGenericModelClient.php |   54 +
 .../OpenAICodex/AmpCodexWebSocketConnector.php     |   32 +
 .../OpenAICodex/CodexCorrelationProvenance.php     |   23 +
 .../OpenAICodex/CodexCorrelationRequestId.php      |   57 +
 .../OpenAICodex/CodexCorrelationResolution.php     |   38 +
 .../Bridge/OpenAICodex/CodexModelClient.php        |  111 +-
 .../Bridge/OpenAICodex/CodexRequestBodyFactory.php |   97 ++
 .../Bridge/OpenAICodex/CodexTransportEnum.php      |   36 +
 .../CodexWebSocketConnectorInterface.php           |   18 +
 .../CodexWebSocketHandshakeHeadersFactory.php      |   33 +
 .../OpenAICodex/CodexWebSocketModelClient.php      |  235 +++
 .../OpenAICodex/CodexWebSocketResultHandle.php     |   14 +
 .../OpenAICodex/CodexWebSocketUrlResolver.php      |   27 +
 src/Platform/Bridge/OpenAICodex/Factory.php        |   68 +-
 .../Bridge/OpenAICodex/RawWebSocketResult.php      |  134 ++
 .../Bridge/OpenAICodex/ResultConverter.php         |  100 +-
 .../Result/CancellableRawResultInterface.php       |   17 +
 src/Tui/Application/InteractiveMode.php            |    6 +-
 src/Tui/Listener/ExportCommandHandler.php          |    9 +-
 src/Tui/Listener/FooterStateInitializer.php        |   10 +-
 src/Tui/Listener/RenameSessionCommandHandler.php   |    4 +-
 .../DynamicToolDescriptionProcessorTest.php        |   22 +-
 .../Infrastructure/SymfonyAi/LlamaCppSmokeTest.php |   43 +-
 .../Infrastructure/SymfonyAi/TraceReplayTest.php   |   29 +-
 .../CodingAgent/Config/Ai/AiProviderConfigTest.php |   10 +
 .../Config/ModelSelectionServiceTest.php           |   30 +-
 .../Config/ModelSettingsPersisterTest.php          |   14 +-
 .../Config/SessionAwareModelResolverTest.php       |   51 +-
 .../Codex/CodexSymfonyAiProviderBuilderTest.php    |   52 +
 .../ApplicationMigrationExecutorTest.php           |   56 +
 .../InProcess/StartRunPersistsSessionModelTest.php |   53 +-
 .../Session/HatfieldSessionStoreTest.php           |  167 ++-
 .../GenericCompletionsWireSanitizationTest.php     |   55 +
 .../Generic/SanitizedGenericModelClientTest.php    |   85 ++
 .../Bridge/OpenAICodex/AssertUuidV7Trait.php       |   17 +
 .../OpenAICodex/CodexCorrelationRequestIdTest.php  |  110 ++
 .../Bridge/OpenAICodex/CodexModelClientTest.php    |  138 +-
 .../CodexWebSocketHandshakeHeadersFactoryTest.php  |   27 +
 .../OpenAICodex/CodexWebSocketModelClientTest.php  |  332 +++++
 .../OpenAICodex/CodexWebSocketUrlResolverTest.php  |   21 +
 .../Bridge/OpenAICodex/RawWebSocketResultTest.php  |  116 ++
 .../Bridge/OpenAICodex/ResultConverterTest.php     |   40 +-
 .../OpenAICodex/ResultConverterWebSocketTest.php   |   32 +
 tests/Tui/E2E/TuiRepairCommandE2eTest.php          |    5 +-
 64 files changed, 4048 insertions(+), 402 deletions(-)
 create mode 100644 migrations/Version20260713120000.php
 create mode 100644 src/Platform/Bridge/Generic/GenericProviderInternalOptionKeys.php
 create mode 100644 src/Platform/Bridge/Generic/SanitizedGenericModelClient.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/AmpCodexWebSocketConnector.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexCorrelationProvenance.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexCorrelationRequestId.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexCorrelationResolution.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexRequestBodyFactory.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexTransportEnum.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexWebSocketConnectorInterface.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexWebSocketHandshakeHeadersFactory.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexWebSocketModelClient.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexWebSocketResultHandle.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexWebSocketUrlResolver.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/RawWebSocketResult.php
 create mode 100644 src/Platform/Result/CancellableRawResultInterface.php
 create mode 100644 tests/Platform/Bridge/Generic/GenericCompletionsWireSanitizationTest.php
 create mode 100644 tests/Platform/Bridge/Generic/SanitizedGenericModelClientTest.php
 create mode 100644 tests/Platform/Bridge/OpenAICodex/AssertUuidV7Trait.php
 create mode 100644 tests/Platform/Bridge/OpenAICodex/CodexCorrelationRequestIdTest.php
 create mode 100644 tests/Platform/Bridge/OpenAICodex/CodexWebSocketHandshakeHeadersFactoryTest.php
 create mode 100644 tests/Platform/Bridge/OpenAICodex/CodexWebSocketModelClientTest.php
 create mode 100644 tests/Platform/Bridge/OpenAICodex/CodexWebSocketUrlResolverTest.php
 create mode 100644 tests/Platform/Bridge/OpenAICodex/RawWebSocketResultTest.php
 create mode 100644 tests/Platform/Bridge/OpenAICodex/ResultConverterWebSocketTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-07-12-add-codex-websocket-transport.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-07-12-add-codex-websocket-transport.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Final reviewer APPROVED; Deterministic castor check passed in 115.4s before final push; PR merged: https://github.com/ineersa/agent-core/pull/285
- Summary: PR #285 merged on GitHub as 223ffd867c6bb5ecdbb720773cdffa5bc22e6544. Delivered Codex WebSocket transport, UUIDv7 request/cache correlation, persisted provider cache identity, stable cross-turn prompt caching, generic-provider option sanitation, bounded 401 refresh, privacy-safe stream errors, direct typed session entity reads, and synchronized tests/docs.

## Task workflow update - 2026-07-13T17:41:35.887Z
- Validation: Initial post-merge LLM_MODE=true castor check: failed only because integration vendor lacked newly locked amphp/websocket-client classes; composer install synchronized 20 locked Amp/WebSocket dependencies; Post-sync LLM_MODE=true castor check: quality OK in 314.7s; Unit/integration: 4369 tests, 14476 assertions OK; Controller replay: 8 tests, 112 assertions OK; TUI: 36 tests, 186 assertions OK; llm-real: 10 tests, 122 assertions OK; deptrac/phpstan/cs-check green; cache stable 326→326; no leaked QA processes
- Summary: Post-merge integration validation completed. Initial check exposed stale integration vendor dependencies after composer.lock merge; installed locked dependencies, then full gate passed.
