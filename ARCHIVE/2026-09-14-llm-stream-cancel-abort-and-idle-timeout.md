# LLM runtime: abort in-flight stream on cancel + stall idle timeout

## Goal
## Evidence (worktree session 2, 2026-09-14 18:31–18:33 UTC)

Test-agent run via PHAR. The multiline picker (task 2026-08-22-upstream-multiline-select-options) is exonerated — the answer was submitted cleanly; the failure is underneath it.

Timeline:
- 18:31:38 human-response step calls `api.deepseek.com` (`deepseek/deepseek-flash`, reasoning=high, 39 tools)
- 18:31:51 HTTP 200 `text/event-stream` arrives — then ZERO chunks ever arrive (provider-side stall)
- 18:32:21 user ESC → `Handling cancel command`, run status → `cancelling` in 7ms
- 18:32:37 two more ESCs — same result; UI stuck in "cancelling"
- 18:33:38 Symfony HttpClient `TransportException: "Max duration was reached"` (max_duration ≈ 120s is the only armed timer) → `llm_step_aborted` → `agent_end: cancelled`
- 18:33:38 `llm.request.retrying` → fallback provider `192.168.2.38:8052` answered in 22ms — too late, run already aborted

## Root cause

`LlmPlatformAdapter::consumeStream()` (src/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapter.php, ~line 408) evaluates `RunCancellationToken` ONLY: before the first delta, between deltas, and after the last delta. A stalled stream yields no deltas, so the `foreach ($deferredResult->asStream() ...)` body never runs; the single-threaded worker sits blocked inside the HttpClient read and never re-polls run status. `abortConnection()` already implements transport teardown (`$response->cancel()` / `CancellableRawResultInterface::abort()`) but is only reachable from the between-chunk/exception paths — it has no trigger during silence.

## Fix direction (not a mandated design)

1. **Cancel-aware stream abort**: move the cancellation check inside the transport wait — e.g. Symfony HttpClient `on_progress` callback (fires on curl loop wake-ups even during a stalled read) with a token check that calls `abortConnection()`-equivalent teardown. ESC should kill the connection within milliseconds regardless of chunk arrival.
2. **Stream idle timeout**: for the no-cancel case, arm an idle/SSE-silence timeout (seconds, not the full ~120s `max_duration`) that aborts and fails over through the existing `LlmRequestRetryExecutor` retry policy. Locate where max_duration is configured for the LLM HTTP client and keep the hard cap as the outer bound.

## Notes

- Owner boundary: AgentCore LLM platform adapter + HTTP client options; no TUI changes required (TUI already delivers cancel promptly — 7ms).
- Relevant symbols: `LlmPlatformAdapter::consumeStream/abortConnection/cancellationToken`, `RunCancellationToken`, `LlmRequestRetryExecutor`, `ExecuteLlmStepWorker`.
- Do not widen scope to provider-availability handling beyond the retry path that already exists.

## Acceptance criteria
- A cancel applied while an LLM stream is silent (no chunks ever arriving) aborts the in-flight HTTP request and lands the run in `cancelled` within a bounded short window (sub-second to low single-digit seconds), not up to max_duration
- A stalled stream (HTTP 200 + no SSE bytes) is detected by an idle timeout and fails over through the existing retry executor without waiting for max_duration
- Both behaviors proven by deterministic tests at the lowest correct layer (controlled fake/stalled stream server or HttpClient stub; no real provider calls, no arbitrary sleeps, no retry-until-green)
- Normal streaming path (chunks arriving) keeps today's between-chunk cancel semantics and is regression-covered
- Validation per testing skill + tests/AGENTS.md: targeted Castor checks during implementation; full `castor check` at CODE-REVIEW transition (LLM-visible flow)

## Workflow metadata
Status: DONE
Branch: task/2026-09-14-llm-stream-cancel-abort-and-idle-timeout
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-14-llm-stream-cancel-abort-and-idle-timeout
Fork run: 0b46293c-1596-5401-9254-3aac24fd0e90
PR URL: https://github.com/ineersa/agent-core/pull/502
PR Status: merged
Started: 2026-09-14T20:31:56+00:00
Completed: 2026-09-17T17:51:34+00:00

## Work log
- Created: 2026-09-14T18:43:37+00:00

## Task workflow update - 2026-09-14T20:31:56+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-14-llm-stream-cancel-abort-and-idle-timeout.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-llm-stream-cancel-abort-and-idle-timeout.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-14-llm-stream-cancel-abort-and-idle-timeout.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-14-llm-stream-cancel-abort-and-idle-timeout.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-14-llm-stream-cancel-abort-and-idle-timeout.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-14-llm-stream-cancel-abort-and-idle-timeout/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-llm-stream-cancel-abort-and-idle-timeout.

## Task workflow update - 2026-09-16T23:07:51+00:00
- Summary: Fetched origin and fast-forwarded task branch from f731e9a1e to origin/main 6b4b902c0 at user request; clean checkout, no conflicts. Refreshed routing: upstream added explicit provider HTTP budgets, which must remain intact. Prior claim that incident binary lacked curl is retracted; exception message is not sufficient transport evidence.
- Ownership: owner=fork; fork_run=none; revision=6b4b902c0; scope=LLM silent-stream cancellation and idle-timeout implementation with deterministic regression tests; outcome=assigned; commit=none

## Task workflow update - 2026-09-16T23:57:36+00:00
- Summary: Parent verification found custom SSE/model-client replacements contrary to existing-facilities rule and use of logging context for a cancellation token. Implementation remains IN-PROGRESS pending corrections; focused tests alone do not establish acceptance.
- Ownership: owner=fork; fork_run=agent_598659a077d5f3de; revision=24dd202aa; scope=Revise cancellation implementation to remove custom framework replacements and logging-context token coupling; outcome=assigned; commit=24dd202aa

## Task workflow update - 2026-09-17T00:24:31+00:00
- Recorded fork run: 0b46293c-1596-5401-9254-3aac24fd0e90
- Ownership: owner=main; fork_run=none; revision=c708751f6; scope=Final test-server cleanup and focused validation after fork context limit; outcome=assigned; commit=none

## Task workflow update - 2026-09-17T00:27:28+00:00
- Recorded fork run: 0b46293c-1596-5401-9254-3aac24fd0e90
- Validation: Parent: castor test --filter='LlmCancelAwareHttpClientTest|LlmPlatformAdapterTest|PlatformIntegrationTest|SymfonyAiProviderFactoryTest' PASS: 40 tests, 263 assertions.; Parent: castor cs-fix --path=tests/CodingAgent/Infrastructure/SymfonyAi/Http/LlmCancelAwareHttpClientTest.php PASS, no formatting changes.; Parent: castor phpstan --path=tests/CodingAgent/Infrastructure/SymfonyAi/Http/LlmCancelAwareHttpClientTest.php FAIL: 42 findings including dynamic static assertion calls, object-shape writable-property annotations, redundant type assertions.; Fork reported production-scoped PHPStan and deptrac PASS. Full castor check not run; reserved for CODE-REVIEW transition.
- Summary: Implementation committed through 2c71d114b after origin/main fast-forward to 6b4b902c0. Vendor SSE framing retained; request-scoped cancellation intercepts progress/stream failures before EventSource reconnect hides cancellation. Existing idle timeout retained with retry regression coverage; no new settings. Parent hardened server protocol framing, removed dead RELEASE path, made accepts query fail loudly. Worktree clean. Additional explicit PHPStan analysis of new test file reports 42 findings; not resolved, so not claiming readiness for CODE-REVIEW.
- Ownership: owner=fork; fork_run=0b46293c-1596-5401-9254-3aac24fd0e90; revision=6b4b902c0; scope=Silent-stream cancellation implementation and deterministic regressions; outcome=completed; commit=c708751f6
- Ownership: owner=main; fork_run=none; revision=c708751f6; scope=Final test-server cleanup and focused validation; outcome=blocked; commit=2c71d114b

## Task workflow update - 2026-09-17T00:59:30+00:00
- Summary: Independent reviewer agent_42a273c7ed79af82 reviewed 2c71d114b. Report says APPROVE WITH SUGGESTIONS but includes BUG findings; workflow requires treating BUG as REQUEST CHANGES. Fixing misleading empty_response logs for cancelled zero-delta results and deterministic retry clock for existing 31s integration test before gate. Reviewer confirmed tests/ is outside normal PHPStan scope; prior 42 explicit test-analysis findings are non-gating.
- Ownership: owner=fork; fork_run=none; revision=2c71d114b; scope=Review fixes for cancelled empty-response logging and deterministic integration retry delay; outcome=assigned; commit=none

## Task workflow update - 2026-09-17T01:07:02+00:00
- Validation: Focused review-fix tests PASS: 23 tests, 121 assertions; prior 31-second case now deterministic.; Source-scoped PHPStan PASS; style PASS.; castor test:llm-real --filter=LlamaCppSmokeTest PASS: 1 test, 8 assertions.
- Summary: Reviewer agent_42a273c7ed79af82 re-reviewed d00b4f01e with specification fidelity: APPROVE WITH SUGGESTIONS, prior BUGs fixed. Remaining suggestions are nonblocking; full transition gate pending. Correction: new cancellation duration_ms is total invocation duration, NOT detection-to-abort latency as suggested by reviewer. Explicit test-file PHPStan findings outside configured analysis paths are non-gating.
- Ownership: owner=fork; fork_run=agent_e203d656f848dfc1; revision=2c71d114b; scope=Cancelled result/log correctness and deterministic integration backoff; outcome=completed; commit=d00b4f01e
- Review: reviewer=agent_42a273c7ed79af82; revision=d00b4f01e; scope=full implementation plus review fixes and specification fidelity; decision=APPROVE WITH SUGGESTIONS; blockers=none before mandatory gate

## Task workflow update - 2026-09-17T01:10:02+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-14-llm-stream-cancel-abort-and-idle-timeout.md.
- Session/run: 48.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-llm-stream-cancel-abort-and-idle-timeout/var/reports/qa-20260917-010720-7777-6410e763.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-17T01:22:24+00:00
- Validation: castor phpstan PASS, 0 errors; castor dead-code PASS, 0 errors; Prior gate other lanes reported PASS; not reused as full gate proof for new revision
- Summary: Gate failures corrected in d1d43ba1f and 821297dea. Full standalone PHPStan and dead-code now pass. Reviewer agent_42a273c7ed79af82 APPROVE at 821297dea, verified narrow existing vendor usage-provider registration, no baseline changes. Remaining datadog cancellation event doc suggestion nonblocking. Full transition gate must rerun for this revision.
- Ownership: owner=fork; fork_run=agent_e203d656f848dfc1; revision=d00b4f01e; scope=Gate PHPStan and vendor dynamic usage corrections; outcome=completed; commit=821297dea
- Review: reviewer=agent_42a273c7ed79af82; revision=821297dea; decision=APPROVE; scope=gate fixes and specification fidelity

## Task workflow update - 2026-09-17T01:24:56+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (135.6s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-llm-stream-cancel-abort-and-idle-timeout/var/reports/qa-20260917-012241-13463-06827d93.
- Session/run: 48.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-17T01:24:58+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-14-llm-stream-cancel-abort-and-idle-timeout to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-llm-stream-cancel-abort-and-idle-timeout/var/reports/qa-20260917-012241-13463-06827d93.
- Session/run: 48.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-17T01:25:01+00:00
- castor check passed (135.6s).
- Pushed task/2026-09-14-llm-stream-cancel-abort-and-idle-timeout to origin.
- Created PR: <url>
- Session/run: 48.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-17T01:25:01+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (135.6s).
- Pushed task/2026-09-14-llm-stream-cancel-abort-and-idle-timeout to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/502
- Summary: Reviewer approved revision 821297dea after gate PHPStan and dead-code corrections; standalone failed lanes now pass.

## Task workflow update - 2026-09-17T16:18:22+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Address advisor P2: preserve reasoning-only partial messages and null error on cancellation.

## Task workflow update - 2026-09-17T16:18:36+00:00
- Ownership: owner=fork; fork_run=agent_e203d656f848dfc1; revision=821297dea; scope=Advisor reasoning-only cancellation guard and companion regression in previously owned worker; outcome=assigned; commit=none

## Task workflow update - 2026-09-17T16:26:50+00:00
- Validation: Regression failed before fix on non-null error; worker suite passed after fix: 8 tests, 46 assertions.; Implementation owner reported scoped source PHPStan and style pass; full transition gate pending.
- Summary: Advisor P2 fixed in 96d7d25a6. Aborted reasoning-only results now bypass both normalizations, retaining original message and null error. Reviewer agent_de8f0a3378bb805b APPROVE with specification fidelity. Prior children could not resume across parent lifetime; replacement owner agent_e4962c1fdbe95e5c completed bounded fix.
- Ownership: owner=fork; fork_run=agent_e4962c1fdbe95e5c; revision=821297dea; scope=Advisor reasoning-only cancellation guard and regression; outcome=completed; commit=96d7d25a6
- Review: reviewer=agent_de8f0a3378bb805b; revision=96d7d25a6; scope=advisor P2 correction/specification fidelity; decision=APPROVE

## Task workflow update - 2026-09-17T16:27:54+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (54.1s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-llm-stream-cancel-abort-and-idle-timeout/var/reports/qa-20260917-162700-1414-2c7aca3c.
- Session/run: 48.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-17T16:27:57+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-14-llm-stream-cancel-abort-and-idle-timeout to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-llm-stream-cancel-abort-and-idle-timeout/var/reports/qa-20260917-162700-1414-2c7aca3c.
- Session/run: 48.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-17T16:27:58+00:00
- castor check passed (54.1s).
- Pushed task/2026-09-14-llm-stream-cancel-abort-and-idle-timeout to origin.
- PR already exists: <url>
- Session/run: 48.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-17T16:27:58+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (54.1s).
- Pushed task/2026-09-14-llm-stream-cancel-abort-and-idle-timeout to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/502
- Summary: Advisor P2 corrected in 96d7d25a6; independent review approved; thinking-only aborted results preserve partial message and null error, with red-green regression coverage.

## Task workflow update - 2026-09-17T17:51:34+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-14-llm-stream-cancel-abort-and-idle-timeout: ide_close_project returned isError.
- Merged task/2026-09-14-llm-stream-cancel-abort-and-idle-timeout into integration checkout.
- Merge made by the 'ort' strategy.
 docs/settings-models.md                                                          |   2 +
 src/AgentCore/Application/Handler/ExecuteLlmStepWorker.php                       |  19 ++++++-
 src/AgentCore/Infrastructure/SymfonyAi/LlmInvocationCancelScope.php              |  91 ++++++++++++++++++++++++++++++
 src/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapter.php                    | 111 +++++++++++++++++++++++++++----------
 src/AgentCore/Infrastructure/SymfonyAi/LlmStreamCancelledException.php           |  22 ++++++++
 src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmCancelAwareHttpClient.php       | 108 ++++++++++++++++++++++++++++++++++++
 src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpClientOptions.php           |   3 +-
 src/CodingAgent/Infrastructure/SymfonyAi/SymfonyAiProviderFactory.php            |   7 ++-
 tests/AgentCore/Application/Handler/ExecuteLlmStepWorkerTest.php                 |  91 ++++++++++++++++++++++++++++++
 tests/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapterTest.php              | 134 ++++++++++++++++++++++++++++++++++++++++++++
 tests/AgentCore/Infrastructure/SymfonyAi/PlatformIntegrationTest.php             |   5 ++
 tests/CodingAgent/Infrastructure/SymfonyAi/Http/LlmCancelAwareHttpClientTest.php | 413 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tools/phpstan/DeadCode/HatfieldDeadCodeUsageProvider.php                         |   7 ++-
 13 files changed, 979 insertions(+), 34 deletions(-)
 create mode 100644 src/AgentCore/Infrastructure/SymfonyAi/LlmInvocationCancelScope.php
 create mode 100644 src/AgentCore/Infrastructure/SymfonyAi/LlmStreamCancelledException.php
 create mode 100644 src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmCancelAwareHttpClient.php
 create mode 100644 tests/CodingAgent/Infrastructure/SymfonyAi/Http/LlmCancelAwareHttpClientTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-llm-stream-cancel-abort-and-idle-timeout.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-llm-stream-cancel-abort-and-idle-timeout.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User merged PR #502; GitHub confirms merge ae94f197699f40139978ed7df4dd2839bc9ea5e7. Post-merge integrated gate to follow.

## Task workflow update - 2026-09-17T17:53:09+00:00
- Updated PR Status: merged
- Validation: Post-merge castor check FAIL at prerequisite: test model endpoint responds to health/models but generation timed out. QA run var/reports/qa-20260917-175201-6829-26c5b0b4. PHAR build/smoke passed before preflight failure.; git status --short clean; task worktree removal confirmed.
- Summary: DONE; PR #502 merged, integration checkout clean at 7cbebaedd, task worktree removed. Created requested upstream assessment issue https://github.com/ineersa/agent-core/issues/503. Post-merge validation INCOMPLETE: castor check stopped during llama.cpp generation preflight (HTTP 0, curl 28); no blind retry or worker restart.

## Task workflow update - 2026-09-17T19:32:31+00:00
- Validation: castor check PASS: all 10 lanes; 5082 unit/integration tests, 11 controller-replay, 9 TUI, 5 llm-real. Static analysis, dead-code, style, docs, deptrac and catalog checks passed. Leak check and cache guard passed.; Reports: var/reports/qa-20260917-193010-8213-b14b6723
- Summary: Post-merge validation now complete. User identified changed server IP as cause of prior generation timeout. Requested rerun of castor check passed on integration revision 6151d2d83073280153eaa3b99982f4bb5afef6b9.
