# Bound observational-memory queries with safe event chunks

## Goal
## Context

Observational memory currently reads a committed source range for an entire logical turn/boundary (user request → agent/tool loop → final assistant response) and then materializes that range into one OM model request. Large agent turns produced 130k+ token OM queries in session 45, causing excessive latency and extension-agent peak memory/recycling.

The active task `2026-08-24-fix-critical-runtime-event-performance-and-delivery` added bounded streaming `EventStoreInterface::rangeFor()` reads, which removes the duplicate full-history `RunEvent[]`/cache layer but intentionally does not solve OM's own complete-range DTO/block/model-input accumulation. This task owns that remaining problem.

Relevant flow:

`ObserveBoundaryJobHandler` → `ObserverPipeline::observeThrough()` → `SessionEventReaderInterface::readRange()` → `OmSourceBlockBuilder` → OM model request.

Relevant files:

- `.hatfield/extensions/observational-memory/src/Observer/ObserveBoundaryJobHandler.php`
- `.hatfield/extensions/observational-memory/src/Observer/ObserverPipeline.php`
- `.hatfield/extensions/observational-memory/src/Observer/OmSourceBlockBuilder.php`
- `src/CodingAgent/Extension/Session/ExtensionSessionEventReader.php`
- `src/AgentCore/Contract/EventStoreInterface.php`
- `.pi/reports/runtime-events-performance.md`

## Product requirement

Process very large observation ranges as multiple bounded OM requests rather than one whole-turn request. Split only at semantically safe event/block boundaries, preserve canonical order and required context, and make the chunk limit configurable through the existing observational-memory extension configuration surface. Do not add a second event store, queue, cache, or generic chunking framework.

Before implementation, finalize the chunk unit/default and the exact safe-boundary rules from the existing OM block model; do not silently invent them.

## Acceptance criteria
- Map the current event → source-block → model-input flow and define which existing block/event boundaries are safe to split without cutting an atomic tool call/result, assistant message, or required user/final-response context.
- A source range that would exceed the configured OM chunk limit is processed incrementally as ordered chunks; no single OM request contains the entire oversized turn.
- Use the existing observational-memory settings/configuration mechanism for one documented chunk-limit setting; finalize its unit and default before implementation and add no unrelated knobs.
- Cursor/checkpoint advancement is failure-safe: advance only for successfully observed chunks, and retries produce no gaps or duplicate persisted observations.
- Small turns retain current one-request behavior; chunking does not change normal observation semantics unnecessarily.
- Deterministic lowest-layer tests cover a synthetic oversized turn, safe-boundary splitting, ordering, retry after a middle-chunk failure, and normal small-turn behavior; no timing/memory assertions.
- Measure the session-45-style 130k+ token case before and after, recording request token/size distribution and extension-agent live/allocated memory by PID without logging source content.
- Full focused Castor validation passes for the observational-memory extension and extension-session integration; PHPStan, Deptrac, CS, and applicable catalog/docs validation pass.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-24-bound-observational-memory-queries-with-safe-chunks
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-24-bound-observational-memory-queries-with-safe-chunks
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/442
PR Status: merged
Started: 2026-08-29T16:46:52.105Z
Completed: 2026-08-29T17:17:09.416Z

## Work log
- Created: 2026-08-24T19:37:17.791Z

## Task workflow update - 2026-08-24T21:06:09.051Z
- Summary: Live experiment evidence: while a disposable Symfony SSE polling patch caused DeepSeek reconnect/replay corruption, the TUI emitted `Extension background job failed after retrying.` Code trace confirms this event only represents an exhausted `ExtensionAgentJobMessage`; current production dispatchers are OM observe-boundary/reflect-generation. Thus an OM background job failed during the experiment. Stock PHAR is restored, but verify failed-boundary recovery/backfill and expose handler/job correlation in diagnostics when implementing this task.

## Task workflow update - 2026-08-29T16:46:52.105Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-24-bound-observational-memory-queries-with-safe-chunks.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-24-bound-observational-memory-queries-with-safe-chunks.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-24-bound-observational-memory-queries-with-safe-chunks.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-24-bound-observational-memory-queries-with-safe-chunks.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-24-bound-observational-memory-queries-with-safe-chunks.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-24-bound-observational-memory-queries-with-safe-chunks.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-24-bound-observational-memory-queries-with-safe-chunks/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-24-bound-observational-memory-queries-with-safe-chunks.
- Summary: Finalized minimal scope with user: keep existing observer.context_window_ratio as the sole request-budget setting; retain default 0.65 but set project OM ratio to 0.50; change OM estimation from 4 to 3.25 characters/token; keep tool results in full through 5,000 estimated tokens and use the existing head/tail/hash digest above that threshold; preserve existing ordered chunk persistence and retry behavior; add no tokenizer dependency or new setting.

## Task workflow update - 2026-08-29T16:47:03.301Z
- Ownership: owner=main; fork_run=none; revision=d3c879397ad9a802a80359b025b935568d64086e; scope=minimal OM estimator, tool-result digest threshold, project ratio configuration, focused tests and docs consistency; outcome=assigned; commit=none

## Task workflow update - 2026-08-29T16:53:27.812Z
- Validation: castor test --filter=ObserverChunkAndToolTest — OK (6 tests, 142 assertions); castor test --suite=extensions — OK (143 tests, 885 assertions); castor test --filter=ExtensionSessionEventReaderTest — OK (2 tests, 10 assertions); castor phpstan --path=.hatfield/extensions/observational-memory — 0 errors; castor cs-check — 0 files fixed; castor deptrac — 0 violations, 0 errors; castor docs:validate — OK; git diff --check — OK
- Summary: Implemented the finalized minimal OM fix. OM now estimates at 3.25 Unicode characters/token, uses that same ratio when splitting oversized source blocks, counts source-message framing against the existing context-window envelope, preserves tool results through 5,000 estimated tokens before applying the existing head/tail/hash digest, and configures this project’s observer ratio at 0.50. No tokenizer dependency or new setting was added.
- Ownership: owner=main; fork_run=none; revision=d3c879397ad9a802a80359b025b935568d64086e; scope=minimal OM estimator, tool-result digest threshold, project ratio configuration, focused tests and docs consistency; outcome=completed; commit=4da94da5a

## Task workflow update - 2026-08-29T17:09:23.963Z
- Validation: Final reviewer run 8edeef0b — APPROVE at a1082cd435093bd89890d9af9ae775554d71da36; artifact /home/ineersa/.pi/agent/sessions/--home-ineersa-projects-agent-core--/subagent-artifacts/8edeef0b_reviewer_output.md; castor test --filter=ObserverChunkAndToolTest — OK (6 tests, 142 assertions); castor test --suite=extensions — OK (143 tests, 885 assertions); castor deptrac — 0 violations, 0 errors; castor phpstan --path=.hatfield/extensions/observational-memory — 0 errors; castor cs-check — 0 files fixed
- Summary: PR gate review completed at a1082cd435093bd89890d9af9ae775554d71da36. Final independent reviewer verdict: APPROVE, with no blocking correctness, security, specification-fidelity, dead-code, or proof findings. Latest explicit user clarification intentionally supersedes the task file’s older broad chunk-setting/measurement acceptance wording; this PR is the finalized minimal estimator, ratio, framing-budget, and tool-result threshold change.
- Review: role=reviewer; run_id=21c0e676; target=4da94da5a; scope=specification fidelity, OM size accounting, tests, callers, security; verdict=APPROVE WITH SUGGESTIONS; artifact=/home/ineersa/.pi/agent/sessions/--home-ineersa-projects-agent-core--/subagent-artifacts/21c0e676_reviewer_output.md
- Review follow-up: owner=main; revision=4da94da5a; outcome=updated two stale test comments only; commit=a1082cd43
- Review: role=reviewer; run_id=8edeef0b; target=a1082cd435093bd89890d9af9ae775554d71da36; scope=final origin/main diff, specification fidelity, arithmetic invariants, tests, security; verdict=APPROVE; blockers=none; artifact=/home/ineersa/.pi/agent/sessions/--home-ineersa-projects-agent-core--/subagent-artifacts/8edeef0b_reviewer_output.md

## Task workflow update - 2026-08-29T17:12:13.734Z
- CODE-REVIEW transition attempt: deterministic gate and branch push completed; PR creation blocked by GitHub GraphQL HTTP 504 while checking existing PR. REST verification found no PR; branch origin/task/2026-08-24-bound-observational-memory-queries-with-safe-chunks is pushed at a1082cd435093bd89890d9af9ae775554d71da36. Retrying transition because failure is isolated to the external PR API, not a QA flake.

## Task workflow update - 2026-08-29T17:13:36.355Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (69.3s).
- Pushed task/2026-08-24-bound-observational-memory-queries-with-safe-chunks to origin.
- branch 'task/2026-08-24-bound-observational-memory-queries-with-safe-chunks' set up to track 'origin/task/2026-08-24-bound-observational-memory-queries-with-safe-chunks'.
- Created PR: https://github.com/ineersa/agent-core/pull/442
- Validation: Reviewer 8edeef0b — APPROVE; castor test --filter=ObserverChunkAndToolTest — OK (6 tests, 142 assertions); castor test --suite=extensions — OK (143 tests, 885 assertions); castor deptrac — 0 violations, 0 errors; castor phpstan --path=.hatfield/extensions/observational-memory — 0 errors; castor cs-check — 0 files fixed
- Summary: Prepared for code review at a1082cd435093bd89890d9af9ae775554d71da36. Independent Pi reviewer run 8edeef0b approved the finalized minimal scope with no blockers. OM now uses a 3.25 chars/token estimate consistently for sizing and splitting, counts request framing against the envelope, preserves tool results through 5,000 estimated tokens before deterministic digesting, and configures the project observer ratio at 0.50 while leaving the built-in 0.65 default unchanged.

## Task workflow update - 2026-08-29T17:17:09.416Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-24-bound-observational-memory-queries-with-safe-chunks: ide_close_project returned isError.
- Merged task/2026-08-24-bound-observational-memory-queries-with-safe-chunks into integration checkout.
- Merge made by the 'ort' strategy.
 .../src/Observer/OmChunkPacker.php                 | 14 ++++--
 .../src/Observer/OmSourceBlockBuilder.php          |  8 +--
 .../src/Observer/OmTokenEstimator.php              | 11 +++-
 .../tests/ObserverChunkAndToolTest.php             | 58 +++++++++++++++++++---
 .../tests/OmQueryServiceTest.php                   |  4 +-
 .../tests/ReflectGenerationJobHandlerTest.php      |  4 +-
 .hatfield/settings.yaml                            |  2 +-
 7 files changed, 78 insertions(+), 23 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-24-bound-observational-memory-queries-with-safe-chunks.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-24-bound-observational-memory-queries-with-safe-chunks.
- Pulled integration checkout: Merge made by the 'ort' strategy.
 .castor/distribution.php                           |   3 +-
 .castor/e2e.php                                    |   3 +
 .castor/phpunit.php                                |   1 +
 .castor/tasks.php                                  |   3 +-
 .hatfield/agents/datadog-logs.md                   |   2 +
 .pi/reports/agent-core-architecture.md             |   6 +-
 .pi/reports/coding-agent-architecture.md           |   4 +-
 .pi/reports/session-storage-file-io-audit.md       |  59 ++-
 ...e-redesign-run-operational-state-in-database.md | 243 ++++-----
 .pi/reports/tests-architecture.md                  |   2 +-
 AGENTS.md                                          |   2 +-
 composer.json                                      |   5 +-
 config/migrations/messenger_transport.yaml         |   7 +
 config/packages/agent_core.yaml                    |  15 -
 config/packages/doctrine.yaml                      |   3 +
 config/packages/doctrine_migrations.yaml           |   2 +-
 config/packages/messenger.yaml                     |  53 +-
 config/services.yaml                               |  49 +-
 config/services_test.yaml                          |   7 +
 docs/session-storage.md                            |  24 +-
 migrations/Version20260617141000.php               |  56 --
 .../{ => application}/Version20260601152619.php    |   0
 .../{ => application}/Version20260606140000.php    |   0
 .../{ => application}/Version20260607000000.php    |   0
 .../{ => application}/Version20260608162000.php    |   0
 .../{ => application}/Version20260617141001.php    |   0
 .../{ => application}/Version20260617141002.php    |   0
 .../{ => application}/Version20260628140000.php    |   0
 .../{ => application}/Version20260710120000.php    |   0
 .../{ => application}/Version20260713120000.php    |   0
 .../{ => application}/Version20260713130000.php    |   0
 .../{ => application}/Version20260713140000.php    |   0
 .../{ => application}/Version20260713160000.php    |   0
 .../{ => application}/Version20260714140000.php    |   0
 .../{ => application}/Version20260715120000.php    |   0
 .../{ => application}/Version20260720120000.php    |   0
 .../{ => application}/Version20260723230000.php    |   0
 .../{ => application}/Version20260813031629.php    |   0
 migrations/application/Version20260828223857.php   |  65 +++
 .../messenger_transport/Version20260828224203.php  |  32 ++
 src/AgentCore/Application/AGENTS.md                |  18 +-
 src/AgentCore/Application/Dto/ReplayIntegrity.php  |  21 -
 .../Application/Dto/ResolvedReplayEvents.php       |  19 -
 .../Handler/ExecuteCompactionStepWorker.php        |   3 -
 .../Application/Handler/ExecuteLlmStepWorker.php   |  23 +-
 .../Application/Handler/ExecuteToolCallWorker.php  |  19 +-
 .../Application/Handler/LatencyHistogram.php       |  92 ----
 src/AgentCore/Application/Handler/RunMetrics.php   | 214 --------
 .../Application/Handler/StepDispatcher.php         |  14 +-
 .../Application/Pipeline/AdvanceRunHandler.php     |  12 +-
 .../Application/Pipeline/ApplyCommandHandler.php   |  16 +
 .../Pipeline/ApplyShellCommandHandler.php          |  14 +-
 .../Pipeline/CasRetryExhaustedException.php        |  17 -
 .../Application/Pipeline/LlmStepResultHandler.php  |  37 +-
 src/AgentCore/Application/Pipeline/RunCommit.php   | 198 ++------
 .../Application/Pipeline/RunMessageProcessor.php   | 190 ++-----
 .../Application/Pipeline/RunOrchestrator.php       |  24 +-
 .../Application/Pipeline/ToolCallResultHandler.php | 109 +++-
 .../Replay/PromptStateReplayService.php            |  22 -
 .../Application/Replay/RunStateReducer.php         | 111 +++-
 .../Contract/ActiveRunContextInterface.php         |  21 +
 .../Contract/PromptStateStoreInterface.php         |  22 -
 .../Replay/HotPromptIntegrityVerifierInterface.php |  12 -
 .../Replay/HotPromptStateRebuilderInterface.php    |  13 -
 src/AgentCore/Contract/RunOperationalStatusDTO.php |  19 +
 .../RunOperationalStatusReaderInterface.php        |  14 +
 src/AgentCore/Contract/RunStoreInterface.php       |  21 -
 .../Extension/AfterTurnCommitHookContext.php       |   2 +
 src/AgentCore/Domain/Message/AGENTS.md             |   5 +-
 src/AgentCore/Domain/Message/AdvanceRun.php        |   2 +-
 src/AgentCore/Domain/Message/CompactRun.php        |   2 +-
 src/AgentCore/Domain/Message/ExecuteLlmStep.php    |   7 +-
 .../Domain/Message/ExecuteShellToolCall.php        |  14 +-
 .../Domain/Message/InvalidateRunContext.php        |  25 +
 .../RunControlTransitionMessageInterface.php       |  12 +
 src/AgentCore/Domain/Run/CurrentToolCallDTO.php    |  26 +
 .../Domain/Run/PendingHumanInputRequestDTO.php     |  12 +
 src/AgentCore/Domain/Run/PromptState.php           |  45 --
 .../Run/RunOperationalToolCallStatusEnum.php       |  16 +
 src/AgentCore/Domain/Run/RunState.php              |   5 +
 src/AgentCore/Domain/Run/ToolBatchIdentity.php     |  14 +
 .../Infrastructure/Storage/CacheCommandStore.php   |   4 +-
 .../Storage/InMemoryPromptStateStore.php           |  33 --
 .../Infrastructure/Storage/InMemoryRunStore.php    |  57 ---
 .../SymfonyAi/LlmPlatformAdapter.php               |  50 +-
 .../SymfonyAi/RunCancellationToken.php             |   6 +-
 .../Agent/Artifact/ActiveRunContext.php            |  64 +++
 .../Agent/Artifact/AgentArtifactPathsDTO.php       |   4 -
 .../Agent/Artifact/AgentArtifactRegistry.php       |   2 -
 .../Artifact/AgentArtifactRetrievalService.php     |  25 +-
 .../Agent/Artifact/AgentChildRunStore.php          | 180 -------
 .../Agent/Artifact/AgentChildRunStoreFactory.php   |  42 --
 .../Agent/Artifact/ChildAwareEventStore.php        |   3 +-
 .../Agent/Artifact/ChildAwareRunStore.php          | 108 ----
 .../Execution/AgentResumeExecutionService.php      |  36 +-
 .../DeferredSubagentBatchChildOutcomeFactory.php   |  18 +-
 .../SubagentChildLaunchInputFactory.php            |   9 +-
 .../Agent/Fork/ForkExecutionService.php            |  14 +-
 src/CodingAgent/CLI/AgentCommand.php               |   4 +-
 .../Compaction/AutoCompactionHookSubscriber.php    |  29 +-
 .../ContextBudgetReminderHookSubscriber.php        |  11 +-
 src/CodingAgent/Entity/DeferredSubagentBatch.php   |   5 +-
 src/CodingAgent/Entity/DeferredToolCompletion.php  |   4 +-
 src/CodingAgent/Entity/HatfieldSession.php         |   3 +-
 .../Entity/RunOperationalHumanInput.php            |  61 +++
 src/CodingAgent/Entity/RunOperationalState.php     | 116 +++++
 src/CodingAgent/Entity/RunOperationalToolCall.php  |  61 +++
 src/CodingAgent/Entity/ToolQuestion.php            |   7 +-
 .../Migrations/ApplicationMigrationExecutor.php    |  79 +--
 .../Migrations/MessengerTransportSchemaEnsurer.php | 104 ----
 .../Migrations/SqliteConnectionHardener.php        |  47 ++
 .../Migrations/StartupDatabaseMigrator.php         |   8 +-
 .../RunOperationalProjectionRepository.php         | 229 +++++++++
 .../CommandHandler/ExecuteShellToolCallWorker.php  | 171 ++-----
 .../Runtime/Controller/HeadlessController.php      |   4 +-
 .../Messenger/WorkerFailedEventSubscriber.php      |  58 +--
 .../Session/CommittedRunEventAppender.php          |  40 +-
 ...rojectionControllerSessionLifecycleListener.php |  25 +
 src/CodingAgent/Session/HatfieldSessionStore.php   |  13 +-
 .../Session/History/HistorySelectionService.php    |  17 +-
 .../Repair/SessionRepairRefusalReasonEnum.php      |   1 -
 .../Session/Repair/SessionRepairService.php        |  51 +-
 .../Replay/SessionHotPromptReplayService.php       | 100 ----
 .../Session/SessionAgentArtifactPathResolver.php   |  10 -
 src/CodingAgent/Session/SessionRunStore.php        | 164 ------
 src/CodingAgent/Session/SessionToolBatchStore.php  |   2 +-
 src/Tui/Listener/RepairCommandHandler.php          |   1 -
 .../Handler/DeferredToolCompletionRuntimeTest.php  |  46 +-
 .../Handler/ExecuteLlmStepWorkerTest.php           |  25 +
 .../Handler/ExecutionFailureDrillTest.php          |   3 +-
 .../Application/Handler/ExecutionWorkerTest.php    |  67 +--
 .../Handler/HookDispatcherContractTest.php         |   4 +
 .../Application/Handler/RunMetricsTest.php         |  53 --
 .../Application/Handler/StepDispatcherTest.php     |  33 ++
 .../Handler/ToolCallHumanInputSuspensionTest.php   | 162 +++---
 .../Application/Pipeline/AdvanceRunHandlerTest.php |  26 +-
 .../Pipeline/ApplyCommandHandlerTest.php           | 100 +++-
 .../Pipeline/ApplyShellCommandHandlerTest.php      |  17 +-
 .../Pipeline/CommandMailboxPolicyTest.php          |  48 +-
 .../Pipeline/LlmStepResultHandlerTest.php          |  34 +-
 .../RunCommitAfterTurnCommitPersistedSeqTest.php   |  34 +-
 .../Application/Pipeline/RunCommitLoggingTest.php  | 269 ++--------
 .../RunMessageProcessorLogComponentTest.php        | 246 +--------
 .../RunOrchestratorInvalidateRunContextTest.php    |  27 +
 .../Pipeline/ToolCallResultHandlerTest.php         | 132 ++++-
 .../Domain/Run/CurrentToolCallDTOTest.php          |  33 ++
 .../Storage/InMemoryRunStoreCasTest.php            |  78 ---
 .../DurableFinishReasonPlatformIntegrationTest.php |   3 +-
 .../Infrastructure/SymfonyAi/LlamaCppSmokeTest.php |  32 +-
 .../SymfonyAi/LlmPlatformAdapterTest.php           |   5 +-
 .../SymfonyAi/PlatformIntegrationTest.php          | 181 +++----
 .../SymfonyAi/Replay/ReplayRecordingTest.php       | 281 -----------
 .../Infrastructure/SymfonyAi/Replay/ReplayTest.php | 395 ---------------
 .../Infrastructure/SymfonyAi/TraceReplayTest.php   | 561 ++++++---------------
 .../Support/NullRunOperationalStatusReader.php     |  17 +
 tests/AgentCore/Support/TestActiveRunContext.php   |  41 ++
 tests/AgentCore/Tools/SessionStorageAuditTest.php  | 130 -----
 .../Agent/Artifact/ActiveRunContextTest.php        |  73 +++
 .../Agent/Artifact/AgentArtifactRegistryTest.php   |   3 -
 .../Artifact/AgentArtifactRetrievalServiceTest.php |  75 +--
 .../Agent/Artifact/AgentChildRunDirectoryTest.php  | 214 --------
 .../Agent/Artifact/AgentChildRunStoreTest.php      | 405 ---------------
 .../Agent/Artifact/ChildAwareRunStoreTest.php      |  64 ---
 .../Execution/AgentResumeExecutionServiceTest.php  |  53 +-
 ...05BareAgentsEffectiveContextIntegrationTest.php |  87 ++--
 .../Launch/DeferredSubagentBatchLaunchTest.php     |  15 +-
 .../DeferredSubagentBatchLifecycleTest.php         |  88 +++-
 ...redSubagentBatchChildTurnHookSubscriberTest.php |   3 +
 .../SubagentChildLaunchModelInheritanceTest.php    |  37 +-
 .../Progress/SubagentProgressEventAppenderTest.php |  83 +--
 .../Execution/SubagentExecutionServiceTest.php     |  11 +-
 .../SubagentPromptUserContextContractTest.php      |  92 +---
 .../Support/PipelineCapturingAgentRunner.php       |  26 +-
 .../Support/ProviderBoundaryCaptureSupport.php     |   3 +-
 .../Support/SubagentExecutionServiceFactory.php    |  10 +-
 .../Fork/ForkChildStartRunInputCompositionTest.php |  25 -
 .../Agent/Fork/ForkExecutionServiceTest.php        |  73 +--
 .../ForkSnapshotCompactionBeforeLaunchTest.php     | 115 ++---
 .../AutoCompactionHookSubscriberTest.php           | 114 +----
 .../ContextBudgetReminderHookSubscriberTest.php    |  47 +-
 .../MessengerTransportSqliteIsolationTest.php      |   7 +-
 ...engerSqliteImmediateTransactionKernelWorker.php |   9 -
 .../Extension/ChildExtensionOmIsolationTest.php    |   3 +
 .../ExtensionAfterTurnCommitHookSubscriberTest.php |   9 +-
 .../MessengerDoctrineRedeliverTimeoutLeaseTest.php |   7 +-
 .../RunResultMessagesRouteToRunControlTest.php     |  76 +++
 .../ApplicationMigrationExecutorTest.php           |  32 +-
 .../MessengerTransportMigrationExecutorTest.php    |  82 +++
 .../MessengerTransportSchemaEnsurerTest.php        |  70 ---
 .../Migrations/StartupDatabaseMigratorTest.php     |  61 +++
 tests/CodingAgent/Phar/PharSmokeTest.php           |   3 +-
 .../RunOperationalProjectionRepositoryTest.php     | 183 +++++++
 ...ocessControllerSessionLifecycleListenerTest.php |  16 +-
 .../ExecuteShellToolCallWorkerTest.php             | 124 ++---
 .../Runtime/Controller/ConsumerSupervisorTest.php  | 213 +++-----
 .../Controller/E2E/ControllerE2eTestCase.php       |   4 +-
 ...storyTurnEmitsRunHistoryPositionChangedTest.php | 285 +++++------
 .../ParentPromptUserContextRegressionTest.php      |  28 +-
 .../Messenger/WorkerFailedEventSubscriberTest.php  | 301 ++++-------
 tests/CodingAgent/Session/AggregateResumeTest.php  | 167 ------
 ...RunEventAppenderLiveProgressIntegrationTest.php |  57 +--
 .../Session/CommittedRunEventAppenderTest.php      |  89 +++-
 ...ctionControllerSessionLifecycleListenerTest.php |  37 ++
 .../Session/HatfieldSessionStoreTest.php           |   2 +-
 .../History/HistorySelectionServiceTest.php        |  52 +-
 .../Session/Repair/SessionRepairServiceTest.php    | 139 ++---
 .../Replay/SessionHotPromptReplayServiceTest.php   |  58 ---
 .../Replay/SessionRunStateReplayServiceTest.php    |  97 +++-
 tests/CodingAgent/Session/SessionRunStoreTest.php  | 416 ---------------
 .../ToolBatchSnapshotCleanupHookSubscriberTest.php |  76 +--
 tests/CodingAgent/Tool/ViewImageToolTest.php       |   2 +-
 tests/Tui/Listener/RepairCommandHandlerTest.php    |  12 +-
 .../SubagentChildLiveViewFixtureSupport.php        |   1 -
 .../Tui/Support/SubagentProgressEventsFixture.php  |  19 -
 tests/paratest-bootstrap.php                       |   9 +-
 tools/session-storage-audit.py                     | 299 -----------
 216 files changed, 4238 insertions(+), 7814 deletions(-)
 create mode 100644 config/migrations/messenger_transport.yaml
 delete mode 100644 migrations/Version20260617141000.php
 rename migrations/{ => application}/Version20260601152619.php (100%)
 rename migrations/{ => application}/Version20260606140000.php (100%)
 rename migrations/{ => application}/Version20260607000000.php (100%)
 rename migrations/{ => application}/Version20260608162000.php (100%)
 rename migrations/{ => application}/Version20260617141001.php (100%)
 rename migrations/{ => application}/Version20260617141002.php (100%)
 rename migrations/{ => application}/Version20260628140000.php (100%)
 rename migrations/{ => application}/Version20260710120000.php (100%)
 rename migrations/{ => application}/Version20260713120000.php (100%)
 rename migrations/{ => application}/Version20260713130000.php (100%)
 rename migrations/{ => application}/Version20260713140000.php (100%)
 rename migrations/{ => application}/Version20260713160000.php (100%)
 rename migrations/{ => application}/Version20260714140000.php (100%)
 rename migrations/{ => application}/Version20260715120000.php (100%)
 rename migrations/{ => application}/Version20260720120000.php (100%)
 rename migrations/{ => application}/Version20260723230000.php (100%)
 rename migrations/{ => application}/Version20260813031629.php (100%)
 create mode 100644 migrations/application/Version20260828223857.php
 create mode 100644 migrations/messenger_transport/Version20260828224203.php
 delete mode 100644 src/AgentCore/Application/Dto/ReplayIntegrity.php
 delete mode 100644 src/AgentCore/Application/Dto/ResolvedReplayEvents.php
 delete mode 100644 src/AgentCore/Application/Handler/LatencyHistogram.php
 delete mode 100644 src/AgentCore/Application/Handler/RunMetrics.php
 delete mode 100644 src/AgentCore/Application/Pipeline/CasRetryExhaustedException.php
 delete mode 100644 src/AgentCore/Application/Replay/PromptStateReplayService.php
 create mode 100644 src/AgentCore/Contract/ActiveRunContextInterface.php
 delete mode 100644 src/AgentCore/Contract/PromptStateStoreInterface.php
 delete mode 100644 src/AgentCore/Contract/Replay/HotPromptIntegrityVerifierInterface.php
 delete mode 100644 src/AgentCore/Contract/Replay/HotPromptStateRebuilderInterface.php
 create mode 100644 src/AgentCore/Contract/RunOperationalStatusDTO.php
 create mode 100644 src/AgentCore/Contract/RunOperationalStatusReaderInterface.php
 delete mode 100644 src/AgentCore/Contract/RunStoreInterface.php
 create mode 100644 src/AgentCore/Domain/Message/InvalidateRunContext.php
 create mode 100644 src/AgentCore/Domain/Message/RunControlTransitionMessageInterface.php
 create mode 100644 src/AgentCore/Domain/Run/CurrentToolCallDTO.php
 delete mode 100644 src/AgentCore/Domain/Run/PromptState.php
 create mode 100644 src/AgentCore/Domain/Run/RunOperationalToolCallStatusEnum.php
 create mode 100644 src/AgentCore/Domain/Run/ToolBatchIdentity.php
 delete mode 100644 src/AgentCore/Infrastructure/Storage/InMemoryPromptStateStore.php
 delete mode 100644 src/AgentCore/Infrastructure/Storage/InMemoryRunStore.php
 create mode 100644 src/CodingAgent/Agent/Artifact/ActiveRunContext.php
 delete mode 100644 src/CodingAgent/Agent/Artifact/AgentChildRunStore.php
 delete mode 100644 src/CodingAgent/Agent/Artifact/AgentChildRunStoreFactory.php
 delete mode 100644 src/CodingAgent/Agent/Artifact/ChildAwareRunStore.php
 create mode 100644 src/CodingAgent/Entity/RunOperationalHumanInput.php
 create mode 100644 src/CodingAgent/Entity/RunOperationalState.php
 create mode 100644 src/CodingAgent/Entity/RunOperationalToolCall.php
 delete mode 100644 src/CodingAgent/Migrations/MessengerTransportSchemaEnsurer.php
 create mode 100644 src/CodingAgent/Migrations/SqliteConnectionHardener.php
 create mode 100644 src/CodingAgent/Repository/RunOperationalProjectionRepository.php
 create mode 100644 src/CodingAgent/Session/Event/RunOperationalProjectionControllerSessionLifecycleListener.php
 delete mode 100644 src/CodingAgent/Session/Replay/SessionHotPromptReplayService.php
 delete mode 100644 src/CodingAgent/Session/SessionRunStore.php
 delete mode 100644 tests/AgentCore/Application/Handler/RunMetricsTest.php
 create mode 100644 tests/AgentCore/Application/Handler/StepDispatcherTest.php
 create mode 100644 tests/AgentCore/Application/Pipeline/RunOrchestratorInvalidateRunContextTest.php
 create mode 100644 tests/AgentCore/Domain/Run/CurrentToolCallDTOTest.php
 delete mode 100644 tests/AgentCore/Infrastructure/Storage/InMemoryRunStoreCasTest.php
 delete mode 100644 tests/AgentCore/Infrastructure/SymfonyAi/Replay/ReplayRecordingTest.php
 delete mode 100644 tests/AgentCore/Infrastructure/SymfonyAi/Replay/ReplayTest.php
 create mode 100644 tests/AgentCore/Support/NullRunOperationalStatusReader.php
 create mode 100644 tests/AgentCore/Support/TestActiveRunContext.php
 delete mode 100644 tests/AgentCore/Tools/SessionStorageAuditTest.php
 create mode 100644 tests/CodingAgent/Agent/Artifact/ActiveRunContextTest.php
 delete mode 100644 tests/CodingAgent/Agent/Artifact/AgentChildRunDirectoryTest.php
 delete mode 100644 tests/CodingAgent/Agent/Artifact/AgentChildRunStoreTest.php
 delete mode 100644 tests/CodingAgent/Agent/Artifact/ChildAwareRunStoreTest.php
 create mode 100644 tests/CodingAgent/Migrations/MessengerTransportMigrationExecutorTest.php
 delete mode 100644 tests/CodingAgent/Migrations/MessengerTransportSchemaEnsurerTest.php
 create mode 100644 tests/CodingAgent/Migrations/StartupDatabaseMigratorTest.php
 create mode 100644 tests/CodingAgent/Repository/RunOperationalProjectionRepositoryTest.php
 delete mode 100644 tests/CodingAgent/Session/AggregateResumeTest.php
 create mode 100644 tests/CodingAgent/Session/Event/RunOperationalProjectionControllerSessionLifecycleListenerTest.php
 delete mode 100644 tests/CodingAgent/Session/Replay/SessionHotPromptReplayServiceTest.php
 delete mode 100644 tests/CodingAgent/Session/SessionRunStoreTest.php
 delete mode 100644 tools/session-storage-audit.py.
- Validation: Independent reviewer 8edeef0b — APPROVE; CODE-REVIEW transition castor check — passed (69.3s)
- Summary: Merged after explicit user authorization. PR #442 was open at a1082cd435093bd89890d9af9ae775554d71da36; independent Pi reviewer run 8edeef0b had approved with no blockers and the CODE-REVIEW transition full castor check passed.

## Task workflow update - 2026-09-06T15:40:33+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
