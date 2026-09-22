# Make LLM request and stream retries reliable and visible in TUI

## Goal
Audit all LLM call paths (main, children, compaction, HTTP/SSE and Codex websocket) for bypassed retry behavior. Incident: reviewer e0fe4617-8310-5cdd-9886-3251a3d2c56f failed on Z.ai idle timeout after 30 seconds; classifier falsely claimed retry exhaustion. User requests retries for HTTP failures, failed chunks and timeouts, default limit five, and visible TUI retry progress. Preserve cancellation and prevent duplicate tool execution/partial response duplication. Existing staged catalog change on main is unrelated and must remain untouched.

## Acceptance criteria
- Default five retries after initial LLM attempt; retry provider/request/stream failures across supported paths, not gated off by retryable:false.
- Each retry is visible live in TUI with attempt/budget, reason and delay; final failure reports actual exhaustion accurately.
- One bounded retry budget, no multiplying transport and application retry loops; user cancellation stops attempts and backoff.
- Failed partial streams are discarded/replaced safely; tool side effects are not replayed.
- Deterministic tests cover HTTP errors, idle timeout, failed chunks, recovery/exhaustion, cancellation and TUI event integration; targeted live provider proof where required.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/488
PR Status: merged
Started: 2026-09-10T13:13:25+00:00
Completed: 2026-09-10T16:42:29+00:00

## Work log
- Created: 2026-09-10T13:13:09+00:00

## Task workflow update - 2026-09-10T13:13:25+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui.

## Task workflow update - 2026-09-10T13:15:17+00:00
- Routing: LlmPlatformAdapter catches stream exceptions into PlatformInvocationResult errors; LlmProviderErrorClassifier hardcodes retryable=false and exhaustion text for transport errors; HTTP RetryableHttpClient passes timeout chunks through and ceases retry interception after response starts. ExecuteLlmStepWorker additionally has a separate thinking-only retry. Existing stream observer -> LlmStreamDispatchObserver provides lifecycle seam for retry/reset notifications.
- Ownership: owner=fork; fork_run=none; revision=fea3ae62f; scope=cohesive LLM retry policy and runtime/TUI visibility with deterministic proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-10T14:07:06+00:00
- Validation: castor test: 4941 tests / 20760 assertions passed; castor test:controller-replay: 11 tests / 175 assertions passed; castor test:tui --filter=TuiProviderErrorE2eTest passed; Targeted PHPStan, deptrac, docs:validate and cs-check passed
- Summary: Implemented bounded application retry budget (five retries), HTTP including 400/auth failures, timeout/stream recovery, runtime/TUI retry notifications and sanitized exhaustion. Corrected fork regressions and validated complete default test lane and controller replay. Uncommitted; live provider proof and transition-owned full gate outstanding.
- Ownership: owner=fork; fork_run=agent_69707f667e13be2e; revision=5a04835f5; scope=initial retry and TUI implementation; outcome=blocked; commit=none
- Ownership: owner=fork; fork_run=agent_90a42315610ed3e5; revision=5a04835f5; scope=takeover after first owner context limit; correct clocks, privacy, compaction isolation and cancellation; outcome=completed; commit=none

## Task workflow update - 2026-09-10T15:16:40+00:00
- Summary: Reviewer agent_37df405547229273 requests changes on dirty revision vs 5a04835f5: remove dead Messenger provider retry path (stacking trap), add moved thinking-only recovery proof, remove unsupported Retry-After parser. Review also identified empty successful stream bypass and stale comments. Assigning focused corrections to existing owner before re-review.
- Ownership: owner=fork; fork_run=agent_90a42315610ed3e5; revision=5a04835f5 plus dirty retry diff; scope=review-requested retry cleanup and missing regression proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-10T15:26:20+00:00
- Validation: Focused revised tests:43 tests242 assertions passed; castor test:llm-real:5 tests30 assertions passed; Focused PHPStan, deptrac and cs-check passed
- Summary: Reviewer agent_37df405547229273 APPROVE WITH SUGGESTIONS after fixes, reviewed dirty revision committed unchanged as 29c4bd312. No blockers. Removed dead Messenger retry path, speculative Retry-After parsing; added thinking-only/empty stream adapter proofs.
- Ownership: owner=fork; fork_run=agent_90a42315610ed3e5; revision=29c4bd312; scope=review corrections and focused proof; outcome=completed; commit=29c4bd312

## Task workflow update - 2026-09-10T15:28:31+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui.md.
- Session/run: 30.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui/var/reports/qa-20260910-152632-20259-a7e75e95.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-10T15:31:23+00:00
- Validation: Focused DurableFinishReason and empty-stream exhaustion cases passed 2 tests9 assertions
- Summary: Gate failure fixed: removed obsolete finish-only-success test contradicting new empty-stream recovery. Retained neighboring test proves finish metadata; new adapter exhaustion test proves empty contract. Reviewer agent_37df405547229273 approved correction at 80f14e83f. No unresolved blockers.
- Ownership: owner=main; fork_run=none; revision=80f14e83f; scope=gate test-contract correction; outcome=completed; commit=80f14e83f

## Task workflow update - 2026-09-10T15:32:30+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (51.7s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui/var/reports/qa-20260910-153138-23189-04646d88.
- Session/run: 30.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-10T15:32:32+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui/var/reports/qa-20260910-153138-23189-04646d88.
- Session/run: 30.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-10T15:32:35+00:00
- castor check passed (51.7s).
- Pushed task/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui to origin.
- Created PR: <url>
- Session/run: 30.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-10T15:32:35+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (51.7s).
- Pushed task/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/488

## Task workflow update - 2026-09-10T15:45:43+00:00
- Moved CODE-REVIEW → IN-PROGRESS.

## Task workflow update - 2026-09-10T15:45:51+00:00
- Ownership: owner=main; fork_run=none; revision=80f14e83f; scope=remove disabled HTTP retry wrapper and unused policy strategy; outcome=assigned; commit=none

## Task workflow update - 2026-09-10T15:52:01+00:00
- Validation: Focused tests12/44 passed; phpstan/deptrac/docs/cs passed
- Summary: Cleanup 4a37ecdf0 approved by reviewer agent_37df405547229273. Removed disabled RetryableHttpClient and obsolete strategy/status/backoff; timeout-only options preserve settings. No blockers.
- Ownership: owner=fork; fork_run=agent_672ca6fd1768f165; revision=4a37ecdf0; scope=HTTP retry scaffolding cleanup after previous owner context exhaustion; outcome=completed; commit=4a37ecdf0

## Task workflow update - 2026-09-10T15:53:00+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (52.5s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui/var/reports/qa-20260910-155207-29038-d311772e.
- Session/run: 30.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-10T15:53:02+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui/var/reports/qa-20260910-155207-29038-d311772e.
- Session/run: 30.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-10T15:53:03+00:00
- castor check passed (52.5s).
- Pushed task/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui to origin.
- PR already exists: <url>
- Session/run: 30.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-10T15:53:03+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (52.5s).
- Pushed task/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/488

## Task workflow update - 2026-09-10T16:42:29+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui: ide_close_project returned isError.
- Merged task/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                                                                       |  15 ++++++
 docs/settings-models.md                                                                    |  15 ++++--
 src/AgentCore/Application/AGENTS.md                                                        |   2 +-
 src/AgentCore/Application/Handler/ExecuteLlmStepWorker.php                                 |  82 ++-------------------------------
 src/AgentCore/Application/Handler/RetryableLlmStepFailureException.php                     |  34 --------------
 src/AgentCore/Contract/Hook/LlmRequestRetryObserverInterface.php                           |  29 ++++++++++++
 src/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapter.php                              |  68 ++++++++++++++++++++++++++-
 src/AgentCore/Infrastructure/SymfonyAi/LlmProviderErrorClassifier.php                      |  63 ++++++++++++++++++-------
 src/AgentCore/Infrastructure/SymfonyAi/Retry/LlmRequestRetryExecutor.php                   | 217 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 src/AgentCore/Infrastructure/SymfonyAi/Retry/LlmRequestRetryPolicy.php                     |  58 +++++++++++++++++++++++
 src/CodingAgent/Config/Ai/AiHttpConfig.php                                                 |  12 +++--
 src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpClientOptions.php                     |  46 +++++++++++++++++++
 src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpRetryPolicy.php                       |  75 ------------------------------
 src/CodingAgent/Infrastructure/SymfonyAi/Retry/LlmRequestRetryPolicyFactory.php            |  29 ++++++++++++
 src/CodingAgent/Infrastructure/SymfonyAi/SymfonyAiProviderFactory.php                      |  23 +++-------
 src/CodingAgent/Runtime/AGENTS.md                                                          |   1 +
 src/CodingAgent/Runtime/Messenger/LlmWorkerFailedEventSubscriber.php                       | 100 ++++++----------------------------------
 src/CodingAgent/Runtime/ProjectionPipeline/LlmRequestRetryProjectionSubscriber.php         |  28 ++++++++++++
 src/CodingAgent/Runtime/Protocol/RuntimeEventTypeEnum.php                                  |   6 +++
 src/CodingAgent/Runtime/Stream/LlmRequestRetryDispatchObserver.php                         |  55 ++++++++++++++++++++++
 src/Tui/Listener/TickPollListener.php                                                      |   6 +--
 src/Tui/Runtime/SubagentLiveChildViewPoller.php                                            |   4 ++
 src/Tui/Runtime/SubagentLiveViewState.php                                                  |   5 ++
 src/Tui/Runtime/TuiRuntimeEventApplier.php                                                 |  23 ++++++++++
 src/Tui/Runtime/TuiSessionState.php                                                        |   6 +++
 tests/AgentCore/Application/Handler/ExecuteLlmStepWorkerTest.php                           | 128 +++++++++++----------------------------------------
 tests/AgentCore/Application/Pipeline/LlmStepResultHandlerTest.php                          |   6 ++-
 tests/AgentCore/Infrastructure/SymfonyAi/DurableFinishReasonPlatformIntegrationTest.php    |  16 -------
 tests/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapterTest.php                        |  25 ++++++----
 tests/AgentCore/Infrastructure/SymfonyAi/LlmProviderErrorClassifierTest.php                |  75 ++++++++++++++++++++----------
 tests/AgentCore/Infrastructure/SymfonyAi/PlatformIntegrationTest.php                       | 143 +++++++++++++++++++++++++++++++++++++++++++++++++++++++--
 tests/AgentCore/Infrastructure/SymfonyAi/Retry/LlmRequestRetryExecutorTest.php             | 314 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/AgentCore/Infrastructure/SymfonyAi/Retry/LlmRequestRetryPolicyTest.php               |  31 +++++++++++++
 tests/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpClientOptionsTest.php               |  35 ++++++++++++++
 tests/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpRetryPolicyTest.php                 |  99 ----------------------------------------
 tests/CodingAgent/Infrastructure/SymfonyAi/Retry/LlmRequestRetryPolicyFactoryTest.php      |  35 ++++++++++++++
 tests/CodingAgent/Infrastructure/SymfonyAi/SymfonyAiProviderFactoryTest.php                |  12 +++--
 tests/CodingAgent/Runtime/Controller/E2E/ControllerReplayLlmRequestRetryVisibilityTest.php | 149 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Runtime/Messenger/LlmWorkerFailedEventSubscriberTest.php                 | 162 ++++++----------------------------------------------------------
 tests/CodingAgent/Runtime/ProjectionPipeline/LlmRequestRetryProjectionSubscriberTest.php   |  52 +++++++++++++++++++++
 tests/CodingAgent/Runtime/RuntimeEventTypeTest.php                                         |   3 ++
 tests/CodingAgent/Runtime/Stream/LlmRequestRetryDispatchObserverTest.php                   |  38 ++++++++++++++++
 tests/Tui/E2E/TuiProviderErrorE2eTest.php                                                  |  10 ++--
 tests/Tui/Listener/TickPollListenerTest.php                                                |  51 +++++++++++++++++++++
 tests/Tui/Runtime/TuiRuntimeEventApplierTest.php                                           |  32 +++++++++++++
 45 files changed, 1687 insertions(+), 731 deletions(-)
 delete mode 100644 src/AgentCore/Application/Handler/RetryableLlmStepFailureException.php
 create mode 100644 src/AgentCore/Contract/Hook/LlmRequestRetryObserverInterface.php
 create mode 100644 src/AgentCore/Infrastructure/SymfonyAi/Retry/LlmRequestRetryExecutor.php
 create mode 100644 src/AgentCore/Infrastructure/SymfonyAi/Retry/LlmRequestRetryPolicy.php
 create mode 100644 src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpClientOptions.php
 delete mode 100644 src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpRetryPolicy.php
 create mode 100644 src/CodingAgent/Infrastructure/SymfonyAi/Retry/LlmRequestRetryPolicyFactory.php
 create mode 100644 src/CodingAgent/Runtime/ProjectionPipeline/LlmRequestRetryProjectionSubscriber.php
 create mode 100644 src/CodingAgent/Runtime/Stream/LlmRequestRetryDispatchObserver.php
 create mode 100644 tests/AgentCore/Infrastructure/SymfonyAi/Retry/LlmRequestRetryExecutorTest.php
 create mode 100644 tests/AgentCore/Infrastructure/SymfonyAi/Retry/LlmRequestRetryPolicyTest.php
 create mode 100644 tests/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpClientOptionsTest.php
 delete mode 100644 tests/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpRetryPolicyTest.php
 create mode 100644 tests/CodingAgent/Infrastructure/SymfonyAi/Retry/LlmRequestRetryPolicyFactoryTest.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/E2E/ControllerReplayLlmRequestRetryVisibilityTest.php
 create mode 100644 tests/CodingAgent/Runtime/ProjectionPipeline/LlmRequestRetryProjectionSubscriberTest.php
 create mode 100644 tests/CodingAgent/Runtime/Stream/LlmRequestRetryDispatchObserverTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-10-make-llm-request-and-stream-retries-reliable-and-visible-in-tui.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-09-10T16:43:58+00:00
- Updated PR Status: merged
- Validation: Post-merge castor check passed qa-20260910-164234-34118-d3ad345d; all ten lanes, artifact integrity and leak check OK
- Summary: PR #488 merged; task DONE. Integration checkout clean and task worktree removed. Post-merge gate completed successfully; supervisor lost status initially but final captured log confirms all lanes and quality: ok.

## Task workflow update - 2026-09-10T22:49:42+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
