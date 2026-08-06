# Add session-cached Codex WebSocket connections and previous-response continuation

## Goal
## Context

Follow-up to `2026-07-12-add-codex-websocket-transport`.

The initial WebSocket transport intentionally opens one connection per LLM request and sends the full request context. This task adds the performance/context optimizations separately after the baseline WebSocket protocol is proven live with GPT-5.6.

Reference behavior: pi-mono caches Codex WebSocket connections per session, leases them for one request at a time, expires idle/old connections, and tracks the prior request/response so subsequent turns can send only the input delta with `previous_response_id`.

## Proposed scope

1. Add an explicit `websocket-cached` Codex transport mode. Preserve baseline `websocket` as one connection per request and `sse` as the existing HTTP path.
2. Cache connections by stable Hatfield session/run ID plus provider/auth identity so credentials and accounts can never cross sessions or profiles.
3. Use exclusive lease/busy semantics: one in-flight request per cached connection. Define deterministic behavior for concurrent requests in the same session (new connection or bounded wait; decide during task-explain).
4. Track continuation state only after a terminal successful response:
   - last full request body/input,
   - last response ID,
   - canonical response output items needed to establish the next baseline.
5. On the next compatible turn, compute a safe input delta and send `previous_response_id` with only the new input. If the current input is not a strict extension of the stored baseline, send full context and reset continuation rather than guessing.
6. Invalidate and close cached state on auth refresh/account change, provider/model mismatch, malformed/incomplete/error response, cancellation, premature close, timeout, session cleanup, idle TTL, or maximum connection age.
7. Integrate cache cleanup with Hatfield session lifecycle/resource cleanup; no static process-global resources without an explicit cleanup owner.
8. Add privacy-safe structured metrics/logging for connection created/reused/expired/reset and full-context vs delta requests. Do not log prompts, tool output, tokens, account IDs, or full payloads.
9. Session cache and continuation remain Codex-specific; general Symfony AI factory and core model abstractions must not gain Codex knowledge.
10. Update `.hatfield/settings.yaml` and `docs/settings.md` for `websocket-cached` behavior.

## Reference defaults to evaluate

Pi-mono currently uses approximately 5 minutes idle TTL and 55 minutes maximum connection age. Confirm these values against Hatfield's long-lived Messenger/session topology rather than copying blindly.

## Dependency

Blocked by successful completion and live validation of `2026-07-12-add-codex-websocket-transport`.

## Acceptance criteria
- `transport: websocket-cached` reuses a healthy connection for subsequent requests in the same Hatfield session; plain `websocket` still opens one connection per request.
- Connections and continuation state never cross session ID, provider ID, auth profile/account identity, or incompatible model boundaries.
- A compatible follow-up sends `previous_response_id` and only the new input delta; behavior is proven against captured outgoing frames.
- A non-extension/divergent transcript falls back to a full-context request and resets continuation without dropping or duplicating input.
- Continuation state is committed only after a terminal successful response; errors, incomplete responses, cancellation, malformed frames, and premature close cannot poison the next turn.
- 401 credential refresh/account identity change closes and invalidates the old connection before retrying with new credentials.
- Idle TTL and maximum-age expiry close sockets and remove continuation state deterministically.
- Concurrent same-session requests cannot interleave frames or responses on one socket.
- Session completion/removal and process shutdown close owned connections; focused tests prove no socket/timer/watcher leaks.
- Structured diagnostics report create/reuse/reset/expiry and full-vs-delta decisions without sensitive content.
- General Symfony AI factory remains unaware of Codex connection caching or continuation.
- Focused Castor tests, `castor test:llm-real`, and full `castor check` pass before review; live multi-turn GPT-5.6 validation demonstrates connection reuse and continuation.

## Workflow metadata
Status: DONE
Branch: task/2026-07-12-codex-websocket-session-cache-continuation
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-12-codex-websocket-session-cache-continuation
Fork run: cecd29im2o57
PR URL: https://github.com/ineersa/agent-core/pull/287
PR Status: merged
Started: 2026-07-13T19:01:10.703Z
Completed: 2026-07-13T23:05:38.714Z

## Work log
- Created: 2026-07-12T21:43:35.652Z

## Task workflow update - 2026-07-13T19:01:10.703Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-07-12-codex-websocket-session-cache-continuation.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-07-12-codex-websocket-session-cache-continuation.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-07-12-codex-websocket-session-cache-continuation.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-07-12-codex-websocket-session-cache-continuation.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-07-12-codex-websocket-session-cache-continuation.
- Summary: Starting follow-up after PR #285 completion. First phase will inspect the merged WebSocket transport and resolve cache ownership, same-session concurrency, continuation baseline, TTL/max-age, and lifecycle cleanup decisions before implementation.

## Task workflow update - 2026-07-13T19:13:35.309Z
- Summary: Design approved with simplicity/reliability priority: explicit websocket-cached mode; busy cache uses fresh one-shot socket; process-scoped shared DI cache (no static globals); 5m idle/55m max-age defaults; strict-extension continuation only; divergence/error/abort/auth change resets; continuation stays connection-bound; plain websocket and SSE remain unchanged.

## Task workflow update - 2026-07-13T19:21:07.125Z
- Recorded fork run: qrtqgltrgdbf
- Validation: Focused Codex cache/continuation/client/builder tests: 31 tests, 100 assertions OK; castor phpstan: 0 errors; castor cs-check: 0 fixable; castor deptrac: 0 violations; castor test:llm-real: 10 tests, 122 assertions OK
- Summary: Implemented explicit websocket-cached transport in dacd454f9: shared DI-owned per-process connection cache, busy one-shot fallback, strict same-socket previous_response_id continuation, 5m idle/55m max-age policy, privacy-safe compatibility fingerprints/logs, worker shutdown cleanup, provider-local settings/docs, and preserved plain websocket/SSE behavior. Live GPT-5.6 multi-turn proof remains before PR.

## Task workflow update - 2026-07-13T19:55:30.909Z
- Summary: Reviewer on dacd454f9 returned REQUEST CHANGES. Confirmed blockers: busy one-shot send failure leaks its connection; cached Generated-ID 401 retry can diverge handshake/body correlation IDs; cached identity remains based on the stale pre-refresh token; busy one-shot 401/identity mismatch can disrupt the active cached stream; project settings example is not synchronized. Additional cleanup: dead method/docblock/filter, restore privacy rationale, remove redundant assignment, document lease flag combinations, add error/abort/expiry safety proofs.
- Reviewer zai/glm-5.2 reviewed full origin/main...dacd454f9 diff after reading AGENTS/testing/task docs; verdict REQUEST CHANGES. Preparing focused fix fork for all actionable findings plus token-fingerprint and busy-identity lifecycle issues discovered during parent analysis.

## Task workflow update - 2026-07-13T20:02:38.813Z
- Recorded fork run: zd1zoyvcfjgu
- Validation: Focused Codex tests: 19 tests, 89 assertions OK; castor test: 4386 tests, 14536 assertions OK; castor phpstan: 0 errors; castor cs-check: 0 fixable; castor deptrac: 0 violations; castor test:llm-real: 10 tests, 122 assertions OK
- Summary: Review-fix commit 0b9fc633b addresses all concrete findings: closes one-shot send failures, bypasses cache for Generated IDs, keeps post-401 IDs aligned, removes rotating bearer token from stable compatibility fingerprint, makes busy leases non-disruptive, hardens terminal/result close-once behavior, synchronizes project settings, simplifies worker cleanup, and adds deterministic max-age/error/401 regressions.

## Task workflow update - 2026-07-13T20:18:43.348Z
- Validation: Orchestrator castor test: 4386 tests, 14536 assertions OK; Orchestrator castor deptrac: 0 violations; Orchestrator castor phpstan: 0 errors; Orchestrator castor cs-check: 0 fixable; Orchestrator castor test:llm-real: 10 tests, 122 assertions OK
- Summary: Reviewer re-review on HEAD 0b9fc633b returned APPROVED with zero actionable findings. All previous lifecycle, 401, identity, concurrency, terminal-event, settings, and test findings verified fixed. Orchestrator focused validation is green; live GPT-5.6 two-turn websocket-cached acceptance proof is now running in an isolated controller harness.
- Reviewer zai/glm-5.2 re-reviewed full origin/main...0b9fc633b diff after required docs; verdict APPROVED, no actionable findings. NTH only: optional future cache size cap and idle-timer callback test.

## Task workflow update - 2026-07-13T20:21:44.315Z
- Recorded fork run: gjxkx4qune4c
- Validation: Live GPT-5.6 turn 1: run.completed, non-empty assistant text, cache.created=1, websocket-cached transport; Live GPT-5.6 turn 2: run.completed, non-empty assistant text, cache.reused=1, continuation.delta=1, delta_input_count=3, has_previous_response_id=true; Live proof provider errors: 0; tool executions: 0; processes after graceful shutdown: 0; Artifacts: var/tmp/codex-ws-cached-live-proof-proof-c669d36f
- Summary: Live acceptance proof succeeded on real openai-codex/gpt-5.6-luna using an isolated two-turn controller session with transport websocket-cached. Turn 1 created the cache and correctly sent full context; turn 2 reused the same cached WebSocket, sent previous_response_id with a 3-item delta, produced non-empty text, and completed without provider errors. Harness workers exited cleanly. The harness exit code 1 was a false-negative assertion that counted the expected first-turn no_continuation full-context event; substantive turn-2 acceptance passed.

## Task workflow update - 2026-07-13T20:23:53.611Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (110.2s).
- Pushed task/2026-07-12-codex-websocket-session-cache-continuation to origin.
- branch 'task/2026-07-12-codex-websocket-session-cache-continuation' set up to track 'origin/task/2026-07-12-codex-websocket-session-cache-continuation'.
- Created PR: https://github.com/ineersa/agent-core/pull/287
- Validation: castor test: 4386 tests, 14536 assertions OK; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: 0 fixable; castor test:llm-real: 10 tests, 122 assertions OK; Reviewer zai/glm-5.2: APPROVED, zero actionable findings; Live openai-codex/gpt-5.6-luna: two runs completed; one cache create, second-turn cache reuse, previous_response_id accepted, delta_input_count=3, zero provider errors, clean shutdown
- Summary: Ready for code review at HEAD 0b9fc633b. Reviewer APPROVED after one fix iteration. Delivers explicit websocket-cached transport with session-scoped connection reuse, strict same-socket previous_response_id continuation, busy one-shot isolation, bounded TTL/max-age, privacy-safe identity isolation/logging, and deterministic worker cleanup. Real GPT-5.6 two-turn acceptance proved cache reuse and continuation delta.

## Task workflow update - 2026-07-13T20:24:06.691Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/287
- Updated PR Status: open
- Validation: Deterministic castor check: passed in 110.2s
- Summary: PR #287 created after deterministic castor check passed in 110.2s. Branch pushed at 0b9fc633b; task is ready for human review.

## Task workflow update - 2026-07-13T20:42:08.096Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User requested PR iteration: switch the tracked project openai-codex setting from websocket to websocket-cached so caching/continuation is the default for this project and can be tested directly. Keep plain websocket transport available as an explicit fallback.

## Task workflow update - 2026-07-13T20:43:17.338Z
- Recorded fork run: v3v5050yav0x
- Validation: castor test --filter CodexSymfonyAiProviderBuilderTest: 16 tests, 26 assertions OK; castor cs-check: 0 fixable
- Summary: User-requested config iteration committed as 0823ec349: tracked project openai-codex transport now defaults to websocket-cached, while plain websocket remains documented as the one-shot fallback. PHP API fallback is unchanged. Ready for user restart/live test before re-review and PR update.

## Task workflow update - 2026-07-13T20:51:21.966Z
- Validation: Session 2 turns 1-4: websocket-cached, cache created then reused, continuation deltas accepted; Session 2 turn 5 after ~308s idle: cache reused, delta prepared, then stream_failure / connection closed before response.completed; No codex.websocket idle_ttl expiration event occurred before reuse, confirming timer callback did not execute
- Summary: User live session 2 exposed an idle-reuse bug on HEAD 0823ec349. Four cached turns succeeded, then after ~308 seconds idle the fifth request reused the socket and immediately failed because the peer had closed it. Root cause: idle expiry relies on Revolt EventLoop::delay, but the Messenger worker blocks between messages so the timer callback does not run. Fix will replace background timer reliance with synchronous idle-age/closed-socket validation on acquire, reconnecting with full context after expiry.

## Task workflow update - 2026-07-13T20:55:25.899Z
- Recorded fork run: cecd29im2o57
- Validation: Focused cache/client/result tests: 21 tests, 78 assertions OK; castor test: 4390 tests, 14553 assertions OK; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: 0 fixable; castor test:llm-real: 10 tests, 122 assertions OK
- Summary: Idle reuse regression fixed and committed as f9ee640de. Cache no longer depends on Revolt timers while Messenger is blocked; successful release records idleSince, and acquire synchronously discards closed, mismatched, max-age, or >=TTL idle sockets before reuse. Post-expiry requests reconnect with full context and no previous_response_id. Ready for user restart/live retest before re-review and PR update.

## Task workflow update - 2026-07-13T22:22:48.753Z
- Summary: User live retest after f9ee640de succeeded: send one turn, idle beyond five minutes, then follow up in the same session completed without the prior closed-before-response.completed failure. Proceeding to final reviewer and PR update.

## Task workflow update - 2026-07-13T22:27:24.905Z
- Validation: Reviewer zai/glm-5.2 on 0823ec349..f9ee640de: APPROVED; Live session 3: cache.expired idle_ttl → cache.created → full_context/no previous_response_id → success
- Summary: Reviewer checked only latest commit f9ee640de as requested and returned APPROVED with zero actionable findings. Live session 3 confirmed the exact fix: after idle expiry it logged cache.expired reason=idle_ttl, created a fresh connection, sent full context without previous_response_id, and completed successfully.

## Task workflow update - 2026-07-13T22:31:08.309Z
- Validation: castor clean:cleanup:workers:list: no stale QA worker candidates; castor test:controller-replay rerun: 8 tests, 112 assertions OK
- Summary: First deterministic CODE-REVIEW gate failed only in unrelated controller replay timing: auto-compaction test observed context_compaction_started at seq 20 but stopped before compacted/failed terminal event. No stale QA workers were present. Immediate full `castor test:controller-replay` rerun passed 8 tests / 112 assertions in 65.7s; retrying deterministic gate.

## Task workflow update - 2026-07-13T22:33:24.102Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (121.7s).
- Pushed task/2026-07-12-codex-websocket-session-cache-continuation to origin.
- branch 'task/2026-07-12-codex-websocket-session-cache-continuation' set up to track 'origin/task/2026-07-12-codex-websocket-session-cache-continuation'.
- PR already exists: https://github.com/ineersa/agent-core/pull/287
- Validation: castor test: 4390 tests, 14553 assertions OK; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: 0 fixable; castor test:llm-real: 10 tests, 122 assertions OK; castor test:controller-replay rerun: 8 tests, 112 assertions OK; Reviewer latest commit f9ee640de: APPROVED; Live session 3 idle retest: cache.expired idle_ttl → fresh connection/full context → success
- Summary: PR #287 iteration ready: project defaults to websocket-cached and f9ee640de synchronously enforces idle TTL. Latest-commit reviewer APPROVED; user live session 3 proved idle expiry reconnect/full-context success. First gate's unrelated controller-replay timing failure passed on immediate isolated rerun.

## Task workflow update - 2026-07-13T22:33:28.113Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/287
- Updated PR Status: open
- Validation: Deterministic castor check retry: passed in 121.7s
- Summary: PR #287 updated with commits 0823ec349 and f9ee640de. Deterministic castor check passed in 121.7s; branch pushed and ready for human review.

## Task workflow update - 2026-07-13T23:05:38.714Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-07-12-codex-websocket-session-cache-continuation into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/settings.yaml                            |   4 +-
 config/services.yaml                               |   3 +
 docs/settings.md                                   |  10 +-
 src/CodingAgent/Config/Ai/AiProviderConfig.php     |  38 +--
 .../Codex/CodexSymfonyAiProviderBuilder.php        |  11 +
 .../CodexWebSocketWorkerShutdownSubscriber.php     |  38 +++
 .../Bridge/OpenAICodex/CodexTransportEnum.php      |   4 +-
 .../OpenAICodex/CodexWebSocketCacheEntry.php       |  28 ++
 .../OpenAICodex/CodexWebSocketCacheLease.php       |  27 ++
 .../OpenAICodex/CodexWebSocketCacheSettings.php    |  26 ++
 .../CodexWebSocketCachedStreamContext.php          |  21 ++
 .../CodexWebSocketCompatibilityFingerprint.php     |  42 +++
 .../OpenAICodex/CodexWebSocketConnectionCache.php  | 232 ++++++++++++++++
 .../CodexWebSocketContinuationComparator.php       |  47 ++++
 .../CodexWebSocketContinuationState.php            |  88 +++++++
 .../OpenAICodex/CodexWebSocketModelClient.php      | 285 +++++++++++++++-----
 src/Platform/Bridge/OpenAICodex/Factory.php        |  18 +-
 .../Bridge/OpenAICodex/RawWebSocketResult.php      | 127 ++++++++-
 .../Codex/CodexSymfonyAiProviderBuilderTest.php    |   4 +
 .../CodexWebSocketCachedModelClientTest.php        | 291 +++++++++++++++++++++
 .../CodexWebSocketConnectionCacheTest.php          | 225 ++++++++++++++++
 .../CodexWebSocketContinuationStateTest.php        |  58 ++++
 .../Bridge/OpenAICodex/RawWebSocketResultTest.php  |  49 ++++
 23 files changed, 1586 insertions(+), 90 deletions(-)
 create mode 100644 src/CodingAgent/Messenger/CodexWebSocketWorkerShutdownSubscriber.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexWebSocketCacheEntry.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexWebSocketCacheLease.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexWebSocketCacheSettings.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexWebSocketCachedStreamContext.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexWebSocketCompatibilityFingerprint.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexWebSocketConnectionCache.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexWebSocketContinuationComparator.php
 create mode 100644 src/Platform/Bridge/OpenAICodex/CodexWebSocketContinuationState.php
 create mode 100644 tests/Platform/Bridge/OpenAICodex/CodexWebSocketCachedModelClientTest.php
 create mode 100644 tests/Platform/Bridge/OpenAICodex/CodexWebSocketConnectionCacheTest.php
 create mode 100644 tests/Platform/Bridge/OpenAICodex/CodexWebSocketContinuationStateTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-07-12-codex-websocket-session-cache-continuation.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-07-12-codex-websocket-session-cache-continuation.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #287 state: MERGED at 2026-07-13T23:05:14Z
- Summary: User merged PR #287 on GitHub. Merge commit bb3df3fbb2c393b8e2296a3b02b383f83a995324. Moving task to DONE and cleaning its worktree.

## Task workflow update - 2026-07-13T23:10:49.904Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/287
- Updated PR Status: merged
- Validation: GitHub PR #287 merged as bb3df3fbb2c393b8e2296a3b02b383f83a995324; Initial post-merge gate: all lanes green except transient test:llm-real SubagentRetrieveLiveE2eTest; castor clean:cleanup:workers:list: no stale QA worker candidates; LLM_MODE=true castor test:llm-real --filter=SubagentRetrieveLiveE2eTest: 1 test, 23 assertions OK; LLM_MODE=true castor check retry: passed in 310.8s; Unit/integration: 4386 tests, 14541 assertions OK; Controller replay: 8 tests, 112 assertions OK; TUI: 36 tests, 186 assertions OK; LLM-real: 10 tests, 122 assertions OK; Deptrac/phpstan/cs-check: green; llama-proxy cache stable: 327→327; QA leak check: no processes for run qa-20260713-230838-470288-a4ce2764
- Summary: Post-merge validation completed on integration checkout. First LLM_MODE=true castor check had one transient llm-real failure in SubagentRetrieveLiveE2eTest (numeric run temporarily lacked session metadata); no stale QA workers were found, and an immediate focused rerun passed 1/23. Full deterministic gate retry then passed all lanes in 310.8s with stable llama-proxy cache and zero leaked QA processes. Worktree cleanup confirmed.
