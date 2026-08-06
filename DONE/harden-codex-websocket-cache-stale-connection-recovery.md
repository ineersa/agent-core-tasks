# Harden cached Codex WebSocket recovery without disabling caching

## Goal
Session 33 hung again after the continuation-duplication fix, but on a different path. The LLM worker claimed turn 171 and reused a cached Codex WebSocket after continuation compatibility fell back to full context. The request used no `previous_response_id`, both preceding tools had completed, and no human input remained. Logging stopped immediately before `WebsocketConnection::sendText()`; the worker's socket was observed in `CLOSE-WAIT` and no provider completion/error was surfaced.

The active Amp send path has no cancellation parameter and may wait indefinitely. The 120-second timeout currently begins only later in `RawWebSocketResult::receive()`, while `WebsocketMessage::buffer()` is also unbounded. Cache stale detection relies on `isClosed()`, which may lag a kernel half-close.

This task must KEEP the `websocket-cached` transport and its continuation/performance behavior. Do not disable caching, change the default transport, or treat transport switching as the fix. Harden cached connection lifecycle and bounded I/O so stale/half-closed sockets fail and recover instead of hanging.

Relevant areas:
- `src/Platform/Bridge/OpenAICodex/CodexWebSocketModelClient.php`
- `src/Platform/Bridge/OpenAICodex/RawWebSocketResult.php`
- `src/Platform/Bridge/OpenAICodex/CodexWebSocketConnectionCache.php`
- Amp WebSocket/byte-stream send and message buffering behavior
- tests under `tests/Platform/Bridge/OpenAICodex/`

Live evidence is authoritative: canonical state remained `running`, with no pending tool/human input; Messenger row was delivered; no completion/failure result followed the cached full-context request. Root-owned processes were observed and must remain untouched.

## Acceptance criteria
- Retain `websocket-cached` transport, connection reuse, and compatible `previous_response_id` continuation behavior; do not remove or disable caching.
- A stale or half-closed cached connection cannot block an LLM worker indefinitely during outbound send, response receive, or message buffering.
- On bounded cached-connection I/O failure, invalidate and close the affected cache entry and propagate or safely recover according to explicit at-most-once/retry semantics; never silently duplicate a request that may already have reached the provider.
- A subsequent request can establish and use a fresh WebSocket connection after stale-entry invalidation.
- Add deterministic regression proof using the real Amp WebSocket/stream path or an equivalently faithful local peer that reproduces the blocked/half-closed cached connection; a mock that immediately throws is not sufficient as the only proof.
- Preserve structured, privacy-safe diagnostics that identify the failed I/O phase and cache lifecycle without logging prompts, tool output, environment values, or full request content.
- Run all QA through Castor. Because this touches provider/runtime behavior, include focused transport tests, `castor test:llm-real` when available, and the full `castor check` gate before CODE-REVIEW.

## Workflow metadata
Status: DONE
Branch: task/harden-codex-websocket-cache-stale-connection-recovery
Worktree: /home/ineersa/projects/agent-core-worktrees/harden-codex-websocket-cache-stale-connection-recovery
Fork run: 9pqd4ms38k9g
PR URL: https://github.com/ineersa/agent-core/pull/317
PR Status: merged
Started: 2026-07-23T15:54:43.283Z
Completed: 2026-07-23T17:23:21.259Z

## Work log
- Created: 2026-07-23T15:54:23.005Z

## Task workflow update - 2026-07-23T15:54:43.283Z
- Moved TODO → IN-PROGRESS.
- Created branch task/harden-codex-websocket-cache-stale-connection-recovery.
- Created worktree /home/ineersa/projects/agent-core-worktrees/harden-codex-websocket-cache-stale-connection-recovery.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/harden-codex-websocket-cache-stale-connection-recovery.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/harden-codex-websocket-cache-stale-connection-recovery.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/harden-codex-websocket-cache-stale-connection-recovery.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/harden-codex-websocket-cache-stale-connection-recovery.
- Summary: Created and started from the second live session-33 hang. Scope explicitly retains websocket caching and continuation behavior; the fix must bound cached WebSocket I/O, invalidate stale entries, preserve safe request semantics, and add a faithful local Amp regression rather than switching transports.

## Task workflow update - 2026-07-23T16:06:45.644Z
- Recorded fork run: 9pqd4ms38k9g
- Summary: Task started with explicit user constraint to retain websocket caching and continuation. Implementation targets bounded send/receive/message-buffer phases, stale-entry invalidation, no automatic replay after ambiguous send delivery, fresh connection on later request, structured diagnostics, and a faithful real-Amp regression.
- 2026-07-23: Production-design scout confirmed `sendText()` has no cancellation and Future-await cancellation alone cannot stop the write; socket/cache invalidation must close the underlying stream. `WebsocketMessage::buffer()` accepts cancellation and should be bounded. Existing `isClosed()` remains only an early stale check.
- 2026-07-23: Launched implementation fork 9pqd4ms38k9g in task worktree. Fork instructed to preserve caching, use at-most-once semantics for ambiguous send failure, add faithful local Amp proof, run Castor-only focused validation, and commit without PR.

## Task workflow update - 2026-07-23T16:18:52.209Z
- Recorded fork run: 9pqd4ms38k9g
- Validation: Fork read and followed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`.; Narrow stale-send/buffer regressions: PASS (6 tests, 46 assertions).; Focused OpenAICodex suite: PASS (36 tests, 177 assertions).; `castor deptrac`: PASS (0 violations).; `castor phpstan`: PASS (0 errors).; `castor cs-check`: PASS after scoped Castor formatting fix.; `castor test:llm-real`: PASS (13 tests, 173 assertions).; Full `castor check` intentionally deferred to CODE-REVIEW transition.
- Summary: Implementation complete and committed as 4b4f04ef7352891d9d313e58feb15f13ced5e796. `websocket-cached` remains enabled. Send, receive, and message buffering are bounded by the existing idle timeout; send timeout/failure closes and invalidates the cache without auto-resending an ambiguously delivered request; subsequent requests acquire a fresh connection and full context. Added real Amp socket-pair backpressure coverage and fragmented-message buffer timeout coverage with privacy-safe phase diagnostics. Worktree is clean; no push or PR yet.

## Task workflow update - 2026-07-23T17:17:37.569Z
- Validation: Reviewer verdict: APPROVE WITH SUGGESTIONS; no blockers. Reviewer read testing skill and tests/AGENTS.md.; `castor test`: PASS (4468 tests, 15477 assertions).; `castor deptrac`: PASS (0 violations, 0 errors).; `castor phpstan`: PASS (0 errors).; `castor cs-check`: PASS (0 files requiring fixes).; `castor test:llm-real`: PASS (13 tests, 173 assertions, 30.5s).; Worktree clean at 4b4f04ef7352891d9d313e58feb15f13ced5e796.
- Summary: Task-to-PR review approved with suggestions and no blocking findings. Reviewer verified send timeout closes the underlying Amp socket to wake blocked writes, ambiguous delivery is not replayed, cache/one-shot cleanup and receive/buffer finalization are correct, successful continuation reuse is unchanged, logging is privacy-safe, and the real Amp tests exercise the pre-fix hang paths. Branch diff is clean against refreshed origin/main and contains only four task files.

## Task workflow update - 2026-07-23T17:19:47.173Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (115.8s).
- Pushed task/harden-codex-websocket-cache-stale-connection-recovery to origin.
- branch 'task/harden-codex-websocket-cache-stale-connection-recovery' set up to track 'origin/task/harden-codex-websocket-cache-stale-connection-recovery'.
- Created PR: https://github.com/ineersa/agent-core/pull/317
- Validation: Reviewer: APPROVE WITH SUGGESTIONS, no blockers.; `castor test`: PASS (4468 tests, 15477 assertions).; `castor deptrac`: PASS.; `castor phpstan`: PASS.; `castor cs-check`: PASS.; `castor test:llm-real`: PASS (13 tests, 173 assertions).
- Summary: Reviewer found no blockers and all focused Castor validation passes. Moving to CODE-REVIEW for deterministic full gate, push, and PR creation. `websocket-cached` remains enabled and compatible continuation reuse is preserved.

## Task workflow update - 2026-07-23T17:23:21.259Z
- Moved CODE-REVIEW → DONE.
- Merged task/harden-codex-websocket-cache-stale-connection-recovery into integration checkout.
- Merge made by the 'ort' strategy.
 .../OpenAICodex/CodexWebSocketModelClient.php      |  93 ++++++--
 .../Bridge/OpenAICodex/RawWebSocketResult.php      |  32 ++-
 .../CodexWebSocketCachedModelClientTest.php        | 246 +++++++++++++++++++++
 .../Bridge/OpenAICodex/RawWebSocketResultTest.php  |  81 ++++++-
 4 files changed, 435 insertions(+), 17 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/harden-codex-websocket-cache-stale-connection-recovery.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/harden-codex-websocket-cache-stale-connection-recovery.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR merge confirmation supplied by user.; Integration checkout clean before DONE transition (`main...origin/main [ahead 2]`).
- Summary: User confirmed PR #317 is merged. Completing task workflow, syncing integration checkout, and cleaning the task worktree.

## Task workflow update - 2026-07-23T17:29:20.756Z
- Validation: Post-merge `LLM_MODE=true castor check` attempt 1: controller-replay timed out at 90.1s (exit 124); every other lane passed; cache/artifact/leak guards passed.; `castor clean:cleanup:workers:list`: no stale QA worker candidates.; `castor test:controller-replay`: PASS (9 tests, 127 assertions, 71.7s).; Post-merge `LLM_MODE=true castor check` attempt 2: controller-replay again timed out at 90.0s (exit 124); every other lane passed; cache/artifact/leak guards passed.; Second worker diagnostic: no stale QA worker candidates.; Integration checkout clean (`main...origin/main [ahead 4]`); task worktree removed.
- Summary: DONE transition completed: branch merged into integration checkout, remote changes pulled, task worktree removed, and IDEA exclusions cleaned. Post-merge functional validation is green, but the full integration `castor check` did not complete because the deterministic controller-replay lane hit its fixed 90s lane timeout twice under parallel gate load. The same controller-replay suite passes focused in 71.7s; all other full-gate lanes passed on both attempts, including unit/integration, TUI replay, llm-real, Deptrac, PHPStan, CS, cache guard, artifact integrity, and leak checks. No stale workers were found.
