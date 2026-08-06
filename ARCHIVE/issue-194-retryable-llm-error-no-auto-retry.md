# Retryable LLM provider error (HTTP 5xx/429/timeout) never auto-retries despite "Will retry automatically" message (#194)

## Goal
## Problem

When an LLM provider returns a retryable error (HTTP 500/502/503/504, 429 rate limit, 408/425 timeout), `LlmProviderErrorClassifier` marks it `retryable: true` and emits a user-facing message promising **"Will retry automatically."** But **no agent-level retry ever fires.** The run transitions to `Failed` with `retryableFailure: true` and the user must manually re-prompt. The message is a lie.

Only the HTTP transport layer retries (`LlmRetryingHttpClient`, `maxRetries` from `ai.http` settings). Once those are exhausted, the error surfaces to the LLM step, gets classified retryable, and... nothing dispatches a retry. The agent-level "auto retry" the message promises does not exist.

Ref: https://github.com/ineersa/agent-core/issues/194

## Root cause (VERIFIED against current code)

1. **Classifier** (`src/AgentCore/Infrastructure/SymfonyAi/LlmProviderErrorClassifier.php:167-168`) marks these as `retryable: true` with "Will retry automatically.":
   - 429 (non-billing) → `CATEGORY_RATE_LIMIT`
   - 408, 425 → `CATEGORY_TIMEOUT`
   - 500, 502, 503, 504 → `CATEGORY_SERVER`

2. **Handler error branch** (`src/AgentCore/Application/Pipeline/LlmStepResultHandler.php`, `if (null !== $message->error)` branch ~line 159-200): emits `LlmStepFailed`, sets `status: Failed`, `retryableFailure: $retryable`, and returns `postCommit: $this->turnCompletedCallbacks(...)` — which **only records metrics**. Compare the no-tool-calls success branch, which calls `$this->followUpAdvanceCallback($runId, 'stop-boundary-follow-up')` to dispatch an `AdvanceRun` via `commandBus`. The error branch has **no** equivalent retry dispatch.

3. **`continueRejectionReason()`** (`src/AgentCore/Application/Pipeline/CommandMailboxPolicy.php:49-72`) only *gates* an externally-issued `continue`: it requires `status === Failed` AND `retryableFailure === true` AND last-message role user/tool. It is a validator, not an auto-dispatcher. Nothing calls `AgentRunner::continue()` automatically after a retryable failure.

4. **`AgentRunner::continue()`** (`src/AgentCore/Application/Pipeline/AgentRunner.php:57-60`) is the public retry entrypoint (`applyCoreCommand($runId, CoreCommandKind::Continue, [])`) but is only invoked manually (TUI/user).

**Conclusion:** the issue's root cause is correct. There is no code path that auto-queues a `continue`/`AdvanceRun` after a retryable `llm_step_failed`.

## Evidence (from issue, session 5 in comp-06 worktree)

```
seq=24 turn=3 LLM_STEP_FAILED
  retryable=True
  error_type=Symfony\Component\HttpClient\Exception\ServerException
  http_status=500
  error_category=server
  user_message=LLM provider server error (HTTP 500 — retryable). Will retry automatically.
  response_error=Context size has been exceeded.

seq=25 turn=3 COMMAND_QUEUED kind=follow_up    ← user manual re-prompt, NOT auto-retry
```

No `COMMAND_QUEUED kind=continue` between seq=24 and seq=25. Replay report: `.hatfield/tmp/reports/20260622-replay.txt`.

## Expected behavior

After a retryable LLM provider error, the system should automatically dispatch a retry (via `AgentRunner::continue()` / a `continue` core command, or an `AdvanceRun` like `followUpAdvanceCallback` does), with bounded retry attempts and backoff — without user intervention. The retry count must be capped to avoid infinite loops.

## Related/secondary: context-overflow misclassified as retryable

The captured error was `"Context size has been exceeded."` surfaced as HTTP 500 by llama.cpp. This is **not** a transient server error — retrying hits the identical error. The classifier currently bucket it as `CATEGORY_SERVER` / `retryable: true` purely from the 500 status. The issue suggests this should either:
- **Not** be marked retryable (retry is futile), OR
- Trigger compaction-based recovery instead of a blind retry.

Note: `LlmStepResultHandler::maybeScheduleOverflowRecovery()` exists only on the `comp-06-auto-compaction-reserve-token-policy` branch (NOT on `main`), per `ide_search_text` for `maybeScheduleOverflowRecovery`. Coordinate/depend on comp-06 (PR #192) if the recovery path needs to land together. On `main` there is currently no overflow recovery either.

The PRIMARY fix (#1 below) is the missing auto-retry. The misclassification (#2) is secondary — at minimum the classifier should not promise auto-retry for conditions where retry is futile, but the deeper compaction-recovery coupling can be scoped separately if needed.

## Suggested fix locations (per issue)

1. **`LlmStepResultHandler`** error branch — after setting `retryableFailure: true`, auto-dispatch a retry command (with backoff + attempt cap) via `postCommit`, mirroring `followUpAdvanceCallback`'s `commandBus->dispatch(new AdvanceRun(...))` pattern.
2. **`CommandMailboxPolicy`** or a new handler — detect the retryable-failed state and auto-queue a `continue`.

## Design questions for the implementing fork (NOT blockers — surface to orchestrator)

- **Retry attempt tracking:** where is the retry count stored? A new `RunState` field (e.g. `retryAttempts`), the command store, or idempotency in the dispatched command? Must survive cross-process boundary (Process runtime uses separate consumer processes — see issue #152 / #183 precedent that in-process state is NOT shared across consumers).
- **Attempt cap + backoff:** reuse `LlmHttpRetryPolicy`-style semantics or a distinct agent-level policy? Settings key under `ai.`? Default cap (e.g. 3) with exponential backoff.
- **Dispatch shape:** `AdvanceRun` (like `followUpAdvanceCallback`) vs `AgentRunner::continue()` (`CoreCommandKind::Continue`). Note `continueRejectionReason` already permits `continue` only from `Failed`+`retryableFailure`+last-role user/tool — the auto-retry must satisfy these or bypass them intentionally. Check the last-message-role guard: after `LlmStepFailed`, is the last message role still `user`? (The failed assistant message is NOT appended on error — verify the guard won't reject the auto-continue.)
- **Futile-retry avoidance:** once #2 is considered, ensure "Context size exceeded"-type 500s do not enter an infinite retry loop. Even before full compaction coupling, the classifier or handler should detect context-overflow signals in the error body and either skip retry or cap at 0.
- **State on final exhaustion:** when retries are exhausted, transition to a terminal `Failed` and ensure the user-facing message no longer says "Will retry automatically."

## Areas to inspect

- `src/AgentCore/Infrastructure/SymfonyAi/LlmProviderErrorClassifier.php` (retryable classification + user_message)
- `src/AgentCore/Application/Pipeline/LlmStepResultHandler.php` (error branch — no retry dispatch; success branch `followUpAdvanceCallback` is the template)
- `src/AgentCore/Application/Pipeline/CommandMailboxPolicy.php` (`continueRejectionReason` gating — lines 49-72)
- `src/AgentCore/Application/Pipeline/AgentRunner.php` (`continue()` — line 57-60)
- `src/AgentCore/Application/Pipeline/AdvanceRunHandler.php` (how `AdvanceRun` is consumed; resume branch + unresolved-tool-calls guard)
- `src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmRetryingHttpClient.php` + `LlmHttpRetryPolicy.php` (existing transport-level retry policy to mirror/reuse)
- `src/AgentCore/Domain/Run/RunState.php` (`retryableFailure` field; candidate for retry-attempt counter)
- Settings: `.hatfield/settings.yaml` + `docs/settings.md` under `ai.http` (existing retry knobs) for new agent-level retry settings

## Out of scope

- Full compaction-recovery-on-overflow wiring (depends on / duplicates comp-06 PR #192 work). Keep the secondary classifier fix minimal unless comp-06 has merged.
- Reworking the HTTP transport retry policy itself (it works; the gap is the agent layer above it).

## Notes on validation (per AGENTS.md)

- This touches the **LLM-visible runtime flow / pipeline** → `castor check` is mandatory (deterministic, replay-backed).
- Per "Replay tests vs. live reproduction": replay-backed tests may pass while the real retry path is broken (replay does not enforce the same run-state transitions / cross-process topology). A retryable-error scenario that the replay marks green but live does not retry is the exact failure mode to watch for. Consider a live `#[Group('llm-real')]` controller E2E that forces a retryable error (e.g. a provider that returns 500/429) and asserts a `continue`/`AdvanceRun` is dispatched and the run recovers — but only if a deterministic proof is insufficient. Surface the tradeoff to the orchestrator.
- Live LLM smoke (`castor test:llm-real`, `castor test:controller`) is opt-in; do NOT add it to default `castor check`.

## Acceptance criteria
- A retryable LLM provider error (5xx/429/408/425) triggers an automatic retry at the agent level (a `continue`/`AdvanceRun` is dispatched) WITHOUT user intervention — the 'Will retry automatically.' message becomes truthful.
- Retry attempts are capped (bounded, configurable) with backoff; no infinite retry loop is possible.
- Retry attempt count survives the Process-runtime cross-process boundary (separate consumer processes) — verified, not assumed in-process.
- When retries are exhausted, the run reaches a terminal `Failed` state and the user-facing message no longer claims automatic retry.
- Context-overflow errors surfaced as HTTP 500 (e.g. llama.cpp 'Context size has been exceeded.') do NOT enter a futile infinite retry loop — either not classified retryable, retry-capped at 0, or routed to overflow recovery (coordinate with comp-06 PR #192 if applicable).
- `continueRejectionReason` last-message-role guard does not reject the auto-continue (last role after `LlmStepFailed` is still `user` since the failed assistant message is not appended — verified, or the guard is adjusted intentionally).
- Smallest failing repro test added first (contract/regression), then fix — e.g. asserts a retry command is dispatched from the `LlmStepResultHandler` error branch when `retryable: true`, and is NOT dispatched when `retryable: false`.
- `castor check` passes in the worktree (deterministic, replay-backed). If replay cannot prove the live retry path, a live `#[Group('llm-real')]` controller E2E is added and passes `castor test:controller`/`test:llm-real` — decision surfaced to orchestrator.
- Settings additions (if any) added to `.hatfield/settings.yaml` AND `docs/settings.md` in sync.

## Workflow metadata
Status: DONE
Branch: task/issue-194-retryable-llm-error-no-auto-retry
Worktree: /home/ineersa/projects/agent-core-worktrees/issue-194-retryable-llm-error-no-auto-retry
Fork run: typydif8p5yy
PR URL: https://github.com/ineersa/agent-core/pull/203
PR Status: merged
Started: 2026-06-23T15:25:39.884Z
Completed: 2026-06-24T00:26:37.918Z

## Work log
- Created: 2026-06-22T20:57:05.265Z

## Task workflow update - 2026-06-23T15:25:39.884Z
- Moved TODO → IN-PROGRESS.
- Created branch task/issue-194-retryable-llm-error-no-auto-retry.
- Created worktree /home/ineersa/projects/agent-core-worktrees/issue-194-retryable-llm-error-no-auto-retry.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/issue-194-retryable-llm-error-no-auto-retry.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/issue-194-retryable-llm-error-no-auto-retry.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/issue-194-retryable-llm-error-no-auto-retry.
- Summary: Claiming task for implementation. Loaded task-workflow skill, testing skill, and tests/AGENTS.md before preparing runtime/test work.

## Task workflow update - 2026-06-23T15:31:28.629Z
- Orchestrator context gathered: loaded task-workflow skill, testing skill, tests/AGENTS.md, task file, and scout outputs. Worktree: /home/ineersa/projects/agent-core-worktrees/issue-194-retryable-llm-error-no-auto-retry.
- Scout findings: retry gap is LlmStepResultHandler error branch; ExecuteLlmStep/LlmStepResult already carry attempt; AdvanceRunHandler hardcodes attempt=1; ApplyCommand continue path clears retryableFailure and dispatches AdvanceRun; RunStateReplayService restores retryableFailure from LlmStepFailed events. Suggested minimal durable retry count: use LlmStepResult/ExecuteLlmStep attempt values carried through Messenger rather than in-process counters.
- Scout findings: context overflow patterns are already recognized by LlmProviderErrorClassifier::isContextOverflow but 500/503 status classification still marks retryable and says 'Will retry automatically'; adjust classifier/handler so overflow is not blindly retried and final exhausted errors do not promise retry. Existing tests to extend: LlmStepResultHandlerTest, LlmProviderErrorClassifierTest, AiConfig/AiHttp config-style tests, maybe LlmRetryingHttpClient tests.

## Task workflow update - 2026-06-23T15:43:27.319Z
- Recorded fork run: lbmcnwrukngk
- Validation: PASS castor test --filter=LlmStepResultHandlerTest (8 tests); PASS castor test --filter=AiAgentRetryConfigTest (5 tests); PASS focused classifier + replay test filters; PASS castor deptrac; PASS castor phpstan; PASS castor cs-check; PARTIAL castor check: unit test/deptrac/phpstan/cs-check lanes passed; castor test:controller-replay failed in ControllerReplaySmokeTest stuck after run.started; castor test:tui exited 1 and was not fully diagnosed by fork; NOT RUN castor test:llm-real (opt-in)
- Summary: Implementation fork completed and committed 44dfdef8b on task/issue-194-retryable-llm-error-no-auto-retry. Implemented bounded agent-level retry with RunState retryAttempts, ai.agent_retry config/docs, delayed ApplyCommand(continue) auto-retry, exhaustion messaging without auto-retry promise, context-overflow 500 non-retryable classification, replay preservation, and focused tests. Worktree is clean. Note: diff against origin/main currently shows unrelated prior main/compaction changes; HEAD commit itself touches 15 expected files.

## Task workflow update - 2026-06-23T15:52:09.917Z
- Summary: Post-fork scout audit found a blocking defect before CODE-REVIEW: autoRetryContinueCallback dispatches ApplyCommand Continue with options auto_retry/retry_attempt, but CommandRouter::validateOptionKeys only permits cancel_safe. Result: auto-retry command would be rejected before ApplyCommandHandler. Scout recommended keeping auto_retry metadata in payload only, removing from options, and adding a focused routing/handler test. Separate check-failure scout found controller-replay/TUI failures likely environmental/flaky, with both passing in isolation after stale-process cleanup, but this does not clear the blocking router defect.

## Task workflow update - 2026-06-23T15:54:49.783Z
- Recorded fork run: uvmr48sqpyku
- Validation: PASS castor test --filter=LlmStepResultHandlerTest (8 tests, 81 assertions); PASS castor test --filter=CommandRouterContractTest (8 tests, 19 assertions); PASS castor phpstan; PASS castor cs-check; FAIL castor check: controller-replay timed out waiting for runtime.ready in ControllerReplayAutoCompactionMultiTurnTest; TUI failed in TmuxHarness/TuiJourneyE2eTest. Fork reports likely unrelated flakes; not resolved in this scope.
- Summary: Fix fork completed and committed dcc6cb9e3 on top of 44dfdef8b. It removed auto_retry/retry_attempt from ApplyCommand.options in LlmStepResultHandler auto-retry callback, keeping metadata in payload only so CommandRouter accepts Continue. Added router/handler regression coverage in CommandRouterContractTest and LlmStepResultHandlerTest. Also fixed exhausted retry payload to explicitly set nested error.retryable=false. Worktree status is clean; HEAD is dcc6cb9e3.

## Task workflow update - 2026-06-23T19:42:30.229Z
- Recorded fork run: 5tv2l77dgabf
- Launched fork 5tv2l77dgabf per user request to fetch/merge origin/main into task branch and run castor check after stale-worker cleanup. Fork instructed not to push/PR/move status and to stop on unexpected uncommitted changes or non-trivial conflicts.

## Task workflow update - 2026-06-23T19:44:31.626Z
- Recorded fork run: 5tv2l77dgabf
- Validation: PASS castor check after origin/main merge (~167.5s); PASS check lane deptrac (0 violations); PASS check lane unit/integration: 3494 tests, 11025 assertions; PASS check lane controller-replay: 7 tests, 97 assertions; PASS check lane TUI replay E2E: 16 tests, 140 assertions; PASS check lane phpstan: no errors; PASS check lane cs-check: 674/674 files clean; NOT RUN castor test:llm-real (opt-in, not required for deterministic gate)
- Summary: Merge/validation fork completed. It fetched origin and merged origin/main into task/issue-194-retryable-llm-error-no-auto-retry with no conflicts, creating merge commit 4c713ad9f. No manual code edits were made by the fork. Worktree is clean at HEAD 4c713ad9ffd8076d09b2752a0b024303894e8167.

## Task workflow update - 2026-06-23T23:39:44.627Z
- Recorded fork run: k6igpp74t2s3
- Launched fork k6igpp74t2s3 per user request to fetch/merge latest origin/main test improvements into the issue-194 task branch and run castor check after current-user stale-worker cleanup. Fork instructed not to push/PR/move status and to stop on unexpected uncommitted changes or non-trivial conflicts.

## Task workflow update - 2026-06-23T23:41:12.280Z
- Recorded fork run: k6igpp74t2s3
- Validation: PASS castor check after latest origin/main merge (~156.1s); PASS check lane deptrac: 0 violations; PASS check lane unit/integration: 3494 tests, 11025 assertions; PASS check lane controller-replay: 7 tests, 97 assertions; PASS check lane TUI replay E2E: 16 tests, 140 assertions; PASS check lane llm-real: 8 tests, 92 assertions; llama.cpp generation preflight OK; PASS check lane phpstan: no errors; PASS check lane cs-check: 674/674 files clean
- Summary: Second origin/main merge fork completed. It fetched latest origin/main through 78d54a8d3 / PR #202, merged into task/issue-194-retryable-llm-error-no-auto-retry with no conflicts, creating merge commit faa4aef77. No manual code edits were made. Worktree is clean at HEAD faa4aef77fd839b6d9ef0a104022be9a097d3c10.

## Task workflow update - 2026-06-24T00:06:45.395Z
- Summary: Reviewer subagent returned REQUEST CHANGES on HEAD faa4aef77. Blocking/actionable findings: RunStateReplayService::applyTurnAdvanced drops retryAttempts causing live/replay divergence and possible retry counter reset after restart; replay test fixture incorrectly uses auto_retry in options instead of production-correct payload-only/options=[]; CommandMailboxPolicy::copyState drops retryAttempts for boundary commands; dead fallback reads auto_retry from options even though CommandRouter rejects those keys; docs say max_attempts is per overflow episode though overflow is excluded; minor redundant context-overflow pattern/noisy naming docs. Need fork remediation and re-review before CODE-REVIEW.

## Task workflow update - 2026-06-24T00:10:34.293Z
- Recorded fork run: typydif8p5yy
- Validation: PASS castor test --filter=RunStateReplayServiceTest (43 tests, 190 assertions); PASS castor test --filter=CommandRouterContractTest (8 tests, 19 assertions); PASS castor test --filter=LlmStepResultHandlerTest (8 tests, 81 assertions); PASS castor test --filter=CommandMailboxPolicyTest::testCopyStatePreservesRetryAttempts (1 test, 1 assertion); PASS castor phpstan; PASS castor cs-check (674/674); PARTIAL castor check: deptrac/unit/controller-replay/TUI/phpstan/cs-check lanes passed; llm-real CompactionLiveSmokeTest failed waiting for compaction.completed after command.ack/status.updated/compaction.started; suspected live/proxy flake unrelated to issue-194 diff
- Summary: Reviewer-fix fork completed and committed 3bb1fe9d7 on top of faa4aef77. Fixes include preserving retryAttempts through RunStateReplayService::applyTurnAdvanced, production-correct payload-only replay fixture/options with turn_advanced coverage, CommandMailboxPolicy::copyState retryAttempts preservation, payload-only auto_retry checks, docs wording, redundant classifier pattern trim, RunState docblock, and null-safe agent_retry DI expressions. Worktree reported clean.

## Task workflow update - 2026-06-24T00:19:59.141Z
- Summary: Reviewer re-review at HEAD 3bb1fe9d7 returned APPROVED. Reviewer verified previous blockers resolved: retryAttempts preserved through applyTurnAdvanced and replay test covers production-shaped llm_step_failed -> agent_command_applied(payload auto_retry/options=[]) -> turn_advanced; CommandMailboxPolicy::copyState preserves retryAttempts; auto_retry is payload-only; docs wording fixed; null-safe agent_retry DI acceptable. No critical/issues remaining; only non-blocking NTH about import style.

## Task workflow update - 2026-06-24T00:21:14.427Z
- Validation: PASS castor test: 3499 tests, 11040 assertions (~15.1s); PASS castor deptrac: 0 violations; PASS castor phpstan: errors=0, file_errors=0; PASS castor cs-check: files_fixed=0 / clean; PASS castor test:llm-real: llama.cpp generation preflight OK; 8 tests, 92 assertions (~29.3s)
- Summary: Task-to-pr local review/validation complete at HEAD 3bb1fe9d7. Reviewer returned APPROVED after remediation. Focused validation run by orchestrator after safe stale-worker cleanup passed, including opt-in llm-real because this is LLM-visible runtime/provider-adjacent behavior.

## Task workflow update - 2026-06-24T00:21:31.792Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (0.2s).
- Pushed task/issue-194-retryable-llm-error-no-auto-retry to origin.
- branch 'task/issue-194-retryable-llm-error-no-auto-retry' set up to track 'origin/task/issue-194-retryable-llm-error-no-auto-retry'.
- Created PR: https://github.com/ineersa/agent-core/pull/203
- Validation: Reviewer: APPROVED at HEAD 3bb1fe9d7; PASS castor test: 3499 tests, 11040 assertions; PASS castor deptrac: 0 violations; PASS castor phpstan: errors=0; PASS castor cs-check: clean; PASS castor test:llm-real: llama.cpp generation OK; 8 tests, 92 assertions
- Summary: Prepared for code review at HEAD 3bb1fe9d73366b2586ddb5e45efb4c6adaf03030. Reviewer subagent approved after remediation. Implementation adds bounded agent-level auto-retry for retryable LLM failures, durable retryAttempts across hot state/replay, payload-only auto_retry Continue routing, exhaustion messaging without auto-retry promise, context-overflow 500 non-retryable classification with overflow recovery, ai.agent_retry config/docs, and focused tests.

## Task workflow update - 2026-06-24T00:26:37.918Z
- Moved CODE-REVIEW → DONE.
- Merged task/issue-194-retryable-llm-error-no-auto-retry into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/settings.yaml                            |   6 +
 config/services.yaml                               |   6 +
 docs/settings.md                                   |  24 +++
 .../Application/Handler/RunStateReplayService.php  |  16 +-
 .../Application/Pipeline/AdvanceRunHandler.php     |   2 +
 .../Application/Pipeline/ApplyCommandHandler.php   |   5 +
 .../Application/Pipeline/CommandMailboxPolicy.php  |   1 +
 .../Application/Pipeline/LlmStepResultHandler.php  | 138 ++++++++++--
 src/AgentCore/Domain/Run/RunState.php              |   2 +
 .../SymfonyAi/LlmProviderErrorClassifier.php       |   7 +
 src/CodingAgent/Config/Ai/AiAgentRetryConfig.php   |  86 ++++++++
 src/CodingAgent/Config/Ai/AiConfig.php             |   3 +
 .../Handler/CommandRouterContractTest.php          |  56 +++++
 .../Handler/RunStateReplayServiceTest.php          |  58 +++++
 .../Pipeline/CommandMailboxPolicyTest.php          |  22 ++
 .../Pipeline/LlmStepResultHandlerTest.php          | 239 ++++++++++++++++++++-
 .../SymfonyAi/LlmProviderErrorClassifierTest.php   |  15 ++
 .../Config/Ai/AiAgentRetryConfigTest.php           |  53 +++++
 18 files changed, 715 insertions(+), 24 deletions(-)
 create mode 100644 src/CodingAgent/Config/Ai/AiAgentRetryConfig.php
 create mode 100644 tests/CodingAgent/Config/Ai/AiAgentRetryConfigTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/issue-194-retryable-llm-error-no-auto-retry.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/issue-194-retryable-llm-error-no-auto-retry.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #203 reported merged by user
- Summary: User reported PR #203 merged. Moving task to DONE and cleaning worktree.
