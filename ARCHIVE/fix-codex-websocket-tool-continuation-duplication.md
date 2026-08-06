# Fix cached Codex WebSocket tool continuation input duplication

## Goal
Session 33 failed after a successful Codex WebSocket tool call with `invalid_request_error` on `input`. The cached continuation sent `previous_response_id` together with both the provider-generated `function_call` and the new `function_call_output`, duplicating an item already owned by the previous response.

Evidence points to `RawWebSocketResult::commitContinuationIfSuccessful()`: continuation response items are cached only from terminal `response.output`, with no fallback to completed `response.output_item.done` items. Request-count arithmetic in session 33 shows zero response items entered the cached baseline, causing the previous function call to be treated as new input and replayed.

Relevant areas:
- `src/Platform/Bridge/OpenAICodex/RawWebSocketResult.php`
- `src/Platform/Bridge/OpenAICodex/CodexWebSocketContinuationState.php`
- `src/Platform/Bridge/OpenAICodex/CodexWebSocketModelClient.php`
- cached WebSocket client/result tests under `tests/Platform/Bridge/OpenAICodex/`

Test thesis: when a streamed response emits a function call through output-item events but the terminal response omits/empties `response.output`, the next cached continuation must send only the matching `function_call_output` delta alongside `previous_response_id`; it must not resend the provider-generated `function_call`.

## Acceptance criteria
- Cache completed streamed response output items for continuation even when terminal `response.output` is absent or empty.
- A tool-result continuation with `previous_response_id` sends only newly added input and never replays the preceding provider `function_call`.
- Add a focused regression test reproducing the session 33 event shape and proving the outgoing continuation delta.
- Preserve correct behavior for terminal responses that already include `response.output` without duplicating cached items.
- Run focused validation through Castor only; because this changes provider/LLM-visible continuation behavior, include the relevant unit tests and `castor test:llm-real` when available.

## Workflow metadata
Status: ARCHIVE
Branch: task/fix-codex-websocket-tool-continuation-duplication
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-codex-websocket-tool-continuation-duplication
Fork run: 94e6zfxu30oh
PR URL: https://github.com/ineersa/agent-core/pull/314
PR Status: merged
Started: 2026-07-22T20:38:51.567Z
Completed: 2026-07-23T13:44:08.820Z

## Work log
- Created: 2026-07-22T20:38:44.473Z

## Task workflow update - 2026-07-22T20:38:51.567Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-codex-websocket-tool-continuation-duplication.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-codex-websocket-tool-continuation-duplication.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-codex-websocket-tool-continuation-duplication.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-codex-websocket-tool-continuation-duplication.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-codex-websocket-tool-continuation-duplication.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-codex-websocket-tool-continuation-duplication.
- Summary: Task created from session 33 diagnosis. Root cause is cached Codex WebSocket continuation replaying a provider function_call with previous_response_id because streamed output-item completion was not included in the continuation baseline.

## Task workflow update - 2026-07-22T20:39:25.525Z
- Recorded fork run: 94e6zfxu30oh
- 2026-07-22: Launched implementation fork 94e6zfxu30oh in the task worktree. Scope: focused regression for streamed function-call output-item continuation caching, production fix, Castor-only focused validation, and commit.

## Task workflow update - 2026-07-22T20:45:34.067Z
- Recorded fork run: 94e6zfxu30oh
- Validation: Read and followed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`.; Focused regression/authority/existing continuation tests: PASS (4 tests, 22 assertions).; `castor test --filter='CodexWebSocket|RawWebSocketResult|CodexWebSocketContinuation'`: PASS (34 tests, 149 assertions).; llama-proxy health on port 9052: PASS.; `castor test:llm-real`: PASS (13 tests, 173 assertions, 34.9s).; Full `castor check` intentionally not run during task-start; it remains the CODE-REVIEW transition gate.
- Summary: Implementation complete and committed as b8396f2056e101e2db4fbb4d7abc16cd896b32f4. RawWebSocketResult now records completed `response.output_item.done` items and uses them only when terminal `response.output` is absent/empty; populated terminal output remains authoritative. Added a session-33-shaped wire regression proving a tool continuation sends only `function_call_output`, plus coverage preventing terminal/stream item duplication. Worktree is clean; branch is not pushed and no PR exists yet.

## Task workflow update - 2026-07-22T20:56:35.575Z
- Validation: Reviewer verdict: APPROVED; mandatory testing skill and tests/AGENTS.md were read.; `castor test`: PASS (4466 tests, 15449 assertions).; `castor deptrac`: PASS (0 violations, 0 errors).; `castor phpstan`: PASS (0 errors).; `castor cs-check`: PASS (0 files requiring fixes).; `castor test:llm-real`: PASS (13 tests, 173 assertions, 32.8s).; Worktree clean at HEAD 1105bde08d083c8ed09222144a08c0d27667f4f2.
- Summary: Task-to-PR review approved with no blocking findings. Reviewer confirmed continuation item capture is per-result lifecycle-safe, terminal output remains authoritative, both terminal event variants are covered, and the wire regression fails without the fix. Initial task-to-PR CS check found docblock alignment only; fork corrected it in commit 1105bde08d083c8ed09222144a08c0d27667f4f2. Current branch HEAD is clean with implementation commit b8396f205 and style commit 1105bde08.

## Task workflow update - 2026-07-22T20:59:31.716Z
- Validation: First deterministic gate attempt: FAILED cache-growth guard (143 → 144 entries).; Warmup `castor test:llm-real`: PASS (13 tests, 173 assertions, 31.7s).; llama-proxy stats stable: 144 entries / 11,352,879 bytes before and after warmup.
- Summary: First CODE-REVIEW transition gate failed only because llama-proxy cache grew from 143 to 144 entries during `castor check`; no code/test lane failure was reported. Re-ran the prescribed warmup and confirmed proxy entries stayed stable at 144 before and after `castor test:llm-real`.

## Task workflow update - 2026-07-22T21:01:44.605Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (118.6s).
- Pushed task/fix-codex-websocket-tool-continuation-duplication to origin.
- branch 'task/fix-codex-websocket-tool-continuation-duplication' set up to track 'origin/task/fix-codex-websocket-tool-continuation-duplication'.
- Created PR: https://github.com/ineersa/agent-core/pull/314
- Validation: Reviewer: APPROVED.; `castor test`: PASS (4466 tests, 15449 assertions).; `castor deptrac`: PASS.; `castor phpstan`: PASS.; `castor cs-check`: PASS.; `castor test:llm-real`: PASS; proxy cache stable at 144 entries after prescribed warmup.
- Summary: Reviewer approved; all focused validation passes. After the first gate's proxy cache-growth guard failure, the live lane was warmed and proxy stats remained stable at 144 entries. Retrying deterministic gate, push, and PR creation.

## Task workflow update - 2026-07-23T13:44:08.820Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-codex-websocket-tool-continuation-duplication into integration checkout.
- Merge made by the 'ort' strategy.
 .../Bridge/OpenAICodex/RawWebSocketResult.php      |  51 +++++++-
 .../CodexWebSocketCachedModelClientTest.php        | 135 +++++++++++++++++++++
 .../Bridge/OpenAICodex/RawWebSocketResultTest.php  |  98 +++++++++++++++
 3 files changed, 279 insertions(+), 5 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-codex-websocket-tool-continuation-duplication.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-codex-websocket-tool-continuation-duplication.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR merge confirmation supplied by user. GitHub CLI verification was unavailable because local `gh` authentication returned HTTP 401.; Integration checkout clean before DONE transition (`main...origin/main [ahead 2]`).
- Summary: User confirmed PR #314 is merged. Completing task workflow and syncing the integration checkout.

## Task workflow update - 2026-07-23T13:49:05.790Z
- Validation: First post-merge `LLM_MODE=true castor check`: all lanes passed except transient `ShellFollowUpLiveE2eTest::testShellThenFollowUpOnCompletedRun` missing command.ack.; `castor clean:cleanup:workers:list`: no stale QA worker candidates.; `castor test:llm-real --filter=ShellFollowUpLiveE2eTest`: PASS (2 tests, 21 assertions).; Post-merge retry `LLM_MODE=true castor check`: PASS (all 7 lanes; cache stable 144 → 144; artifact integrity and leak checks passed; 287.4s reported aggregate).; Integration checkout clean (`main...origin/main [ahead 2]`); task worktree removed.
- Summary: DONE transition completed: task branch merged into integration checkout, remote changes pulled, task worktree removed, and IDEA exclusions cleaned. Post-merge full validation passed on retry. The first post-merge gate had a transient live ShellFollowUp test failure (missing follow-up command ack while controller remained running); no stale workers were found, the focused live test passed immediately, and the full deterministic gate then passed.

## Task workflow update - 2026-08-06T20:58:59.881Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
