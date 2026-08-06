# Include tests in Castor CS fixer/checker scope

## Goal
Context: `.php-cs-fixer.dist.php` already configures `php_unit_test_case_static_method_calls => ['call_type' => 'this']`, but the default Finder only scans `src`, `.castor`, `.hatfield/extensions`, `.php-cs-fixer.dist.php`, and `castor.php`. As a result, default `castor cs-check` / `castor check` can pass while tests still contain `self::assert*`; running `castor cs-fix tests` exposes broad test style drift. User wants this fixed on main as a separate task, and `cs-fix` should include tests by default too. Keep separate from PR #254/subagent-live work.

## Acceptance criteria
- Default `castor cs-check` scans test files as well as existing source/tooling paths.
- Default `castor cs-fix` applies the same test scope as `castor cs-check` (no need to pass `tests` manually).
- Run `castor cs-fix` once and commit the resulting test formatting churn in this task branch, including `self::assert*` → `$this->assert*` where required by the existing rule.
- `castor cs-check` passes cleanly after the repo-wide formatting commit.
- Focused validation uses Castor only; no raw `vendor/bin/*` unless isolating a Castor failure.
- Keep this as a separate PR from subagent-live PR #254.

## Workflow metadata
Status: DONE
Branch: task/include-tests-in-castor-cs-scope
Worktree: /home/ineersa/projects/agent-core-worktrees/include-tests-in-castor-cs-scope
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/261
PR Status: merged
Started: 2026-07-04T23:20:37.903Z
Completed: 2026-07-05T00:27:59.436Z

## Work log
- Created: 2026-07-04T23:15:09.794Z

## Task workflow update - 2026-07-04T23:20:37.904Z
- Moved TODO → IN-PROGRESS.
- Created branch task/include-tests-in-castor-cs-scope.
- Created worktree /home/ineersa/projects/agent-core-worktrees/include-tests-in-castor-cs-scope.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/include-tests-in-castor-cs-scope.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/include-tests-in-castor-cs-scope.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/include-tests-in-castor-cs-scope.

## Task workflow update - 2026-07-04T23:22:46.316Z
- Validation: castor cs-fix (default scope incl. tests): exit 0; castor cs-check (default scope): exit 0, 0 fixable violations across 1223 files; git: 323 files changed in commit f4d9a3649 (+10718/-10733); src/ changed = 0; .php-cs-fixer.dist.php changed = 1; tests/ changed = 322; git status clean post-commit
- Summary: Implementation complete on task branch `task/include-tests-in-castor-cs-scope` @ `f4d9a3649`.

Single atomic commit: config change + repo-wide test style churn.
- `.php-cs-fixer.dist.php`: added `__DIR__.'/tests'` to Finder `->in([...])` so default `castor cs-fix`/`cs-check` cover tests. No Castor task edits.
- 322 test files reformatted (self::assert* → $this->assert*, @Symfony risky rules, declare_strict_types, ordering, fopen b_mode, \LOCK_EX, Yoda, concat spacing).
- 0 `src/` files touched; working tree clean.

Mechanical style-only; no behavioral test changes flagged on spot-check.
- IN-PROGRESS claimed; worktree /home/ineersa/projects/agent-core-worktrees/include-tests-in-castor-cs-scope
- Fork implemented: config edit + castor cs-fix run + verified castor cs-check; committed f4d9a3649
- Verification from integration checkout: branch + commit confirmed; config diff is the intended single-line addition; src/ untouched

## Task workflow update - 2026-07-05T00:22:07.537Z
- move_task -> CODE-REVIEW: castor check gate FAILED only on test:llm-real lane (1 test: SubagentRetrieveLiveE2eTest::testSubagentThenAgentRetrieveChain). All deterministic lanes passed: cs-check (0 violations), deptrac (0), phpstan (0 errors), test (4064 OK), controller-replay (8 OK), tui (28 OK).
- Root-caused llm-real failure as a transient live-LLM flake (parent model emitted text instead of calling agent_retrieve on a cache miss during the gate). PROOF task changes are unrelated: (1) 0 src/ files changed -> production prompts identical; (2) test-file diff is purely mechanical (self::->\$this::, ordered_class_elements) with NO string-literal content changes; (3) focused re-run on main passes 7.8s/23 assertions; (4) focused re-run in worktree passes 7.8s/23 assertions (shared warm proxy cache on :9052).
- Retrying move_task -> CODE-REVIEW with warm cache.

## Task workflow update - 2026-07-05T00:23:48.562Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (90.5s).
- Pushed task/include-tests-in-castor-cs-scope to origin.
- branch 'task/include-tests-in-castor-cs-scope' set up to track 'origin/task/include-tests-in-castor-cs-scope'.
- Created PR: https://github.com/ineersa/agent-core/pull/261

## Task workflow update - 2026-07-05T00:23:56.718Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/261
- Updated PR Status: open
- move_task -> CODE-REVIEW (retry): castor check PASSED (90.5s, all lanes green with warm cache). Branch pushed, PR #261 created.

## Task workflow update - 2026-07-05T00:27:59.436Z
- Moved CODE-REVIEW → DONE.
- Merged task/include-tests-in-castor-cs-scope into integration checkout.
- Merge made by the 'ort' strategy.
 .php-cs-fixer.dist.php                             |   2 +-
 .../AfterTurnCommitSerializerRegressionTest.php    |   1 -
 .../Handler/CommandRouterContractTest.php          |  39 +-
 .../ExecuteCompactionStepSerializerTest.php        |  50 +--
 .../Handler/ExecuteCompactionStepWorkerTest.php    |  67 +--
 .../Handler/ExecuteLlmStepWorkerTest.php           |  54 +--
 .../Handler/ExecutionFailureDrillTest.php          |  24 +-
 .../Handler/HookDispatcherContractTest.php         |  14 +-
 .../Handler/InMemoryToolBatchStoreTest.php         |  14 +-
 .../Application/Handler/ReplayServiceTest.php      | 114 ++---
 .../Application/Handler/RunLockManagerTest.php     |  20 +-
 .../Application/Handler/RunMetricsTest.php         |  28 +-
 .../Application/Handler/RunTracerTest.php          |  62 ++-
 .../Handler/ToolBatchCollectorDurableTest.php      |  80 ++--
 .../Application/Handler/ToolBatchCollectorTest.php |  76 ++--
 .../Handler/ToolExecutionPolicyResolverTest.php    |  16 +-
 .../Application/Handler/ToolExecutorTest.php       |  35 +-
 .../Application/Pipeline/AdvanceRunHandlerTest.php |   2 -
 .../Pipeline/ApplyCommandHandlerTest.php           |   4 +-
 .../Pipeline/CommandMailboxPolicyTest.php          |  44 +-
 .../Pipeline/LlmStepResultHandlerTest.php          |  17 +-
 .../Application/Pipeline/StartRunHandlerTest.php   |   5 +-
 .../Application/Pipeline/ToolCallExtractorTest.php |  80 ++--
 .../Pipeline/ToolCallResultHandlerTest.php         |  30 +-
 .../Replay/TurnTreeReplayFilterTest.php            |  46 +-
 .../Tool/StackToolExecutionContextAccessorTest.php |  25 +-
 .../Contract/LifecycleEventContractTest.php        |  14 +-
 .../Domain/Command/CommandBoundaryTest.php         |  58 +--
 tests/AgentCore/Domain/Event/EventFactoryTest.php  |  50 +--
 tests/AgentCore/Domain/Event/RunEventTest.php      |  20 +-
 .../Domain/Message/AgentBusMessageContractTest.php |  81 ++--
 .../AgentCore/Domain/Message/AgentMessageTest.php  |  90 ++--
 .../Domain/Model/ModelInvocationContractTest.php   |  58 +--
 tests/AgentCore/Domain/Run/RunStateTest.php        |  53 ++-
 .../AgentCore/Domain/Run/TurnTreeProjectorTest.php | 156 +++----
 tests/AgentCore/Domain/Tool/ToolBoundaryTest.php   |  62 +--
 .../AgentCore/Infrastructure/RunLogContextTest.php |  32 +-
 .../Storage/CacheCommandStoreTest.php              |   2 +-
 .../Storage/InMemoryRunStoreCasTest.php            |  20 +-
 .../DynamicToolDescriptionProcessorTest.php        |  72 +--
 .../Infrastructure/SymfonyAi/LlamaCppSmokeTest.php |  26 +-
 .../SymfonyAi/LlmPlatformAdapterTest.php           |  15 +-
 .../SymfonyAi/LlmProviderErrorClassifierTest.php   | 138 +++---
 .../SymfonyAi/PlatformIntegrationTest.php          |  44 +-
 .../ReasoningContentFeatureShaperTest.php          |  77 ++--
 .../SymfonyAi/Replay/ReplayRecordingTest.php       |   6 +-
 .../Infrastructure/SymfonyAi/Replay/ReplayTest.php |  20 +-
 .../Infrastructure/SymfonyAi/TraceReplayTest.php   |  26 +-
 .../Support/Builder/DomainMessageBuildersTest.php  |   1 -
 .../AgentCore/Support/Builder/RunStateBuilder.php  |  12 +-
 tests/AgentCore/Support/Fake/FakePlatform.php      |   5 +-
 tests/AgentCore/Support/Fake/FakeToolExecutor.php  |   5 +-
 tests/AgentCore/Support/TestLogger.php             |   3 +-
 tests/AgentCore/Support/TestSerializerFactory.php  |   2 +-
 .../Agent/Artifact/AgentArtifactKindEnumTest.php   |  34 +-
 .../Agent/Artifact/AgentArtifactRegistryTest.php   | 163 ++++---
 .../Artifact/AgentArtifactRetrievalServiceTest.php | 106 +++--
 .../Artifact/AgentArtifactSessionListingTest.php   |  10 +-
 .../Agent/Artifact/AgentChildRunDirectoryTest.php  |  36 +-
 .../Agent/Artifact/AgentChildRunEventStoreTest.php |  34 +-
 .../Agent/Artifact/AgentChildRunStoreTest.php      |  56 +--
 .../Agent/Artifact/ChildAwareEventStoreTest.php    |   9 +-
 .../Agent/Artifact/ChildAwareRunStoreTest.php      |  13 +-
 .../Agent/Context/AgentContextRendererTest.php     |  18 +-
 .../Agent/Context/AgentsContextBuilderTest.php     |  20 +-
 .../Definition/AgentDefinitionCatalogTest.php      |  63 ++-
 .../Definition/AgentDefinitionDiscoveryTest.php    | 214 ++++-----
 .../Agent/Definition/AgentDefinitionParserTest.php | 399 +++++++++--------
 .../Agent/Execution/AgentDepthGuardTest.php        |  10 +-
 .../Agent/Execution/AgentMcpToolsResolverTest.php  |  26 +-
 .../Execution/AgentToolPolicyResolverTest.php      |  12 +-
 .../SubagentChildProgressSummaryBuilderTest.php    |  26 +-
 .../Execution/SubagentExecutionServiceTest.php     | 491 ++++++++++----------
 .../Execution/SubagentToolSetResolverTest.php      | 120 ++---
 .../Agent/Fork/ForkContextBuilderTest.php          | 164 +++----
 .../Agent/Fork/ForkHandoffValidatorTest.php        |  64 +--
 .../CodingAgent/Agent/Fork/ForkLevelConfigTest.php | 100 ++---
 .../Agent/Fork/ForkRunMetadataDTOTest.php          |  80 ++--
 .../Agent/Fork/ForkSnapshotCompactorTest.php       | 148 +++---
 .../Agent/Fork/ForkSnapshotSanitizerTest.php       | 110 ++---
 .../Agent/Fork/ForkTaskPromptBuilderTest.php       |  32 +-
 .../Agent/Tool/AgentRetrieveToolTest.php           |  15 +-
 .../Tool/SubagentToolDefinitionBuilderTest.php     |   4 +-
 tests/CodingAgent/Agent/Tool/SubagentToolTest.php  |  31 +-
 .../Application/Pipeline/CompactRunHandlerTest.php |   5 +-
 .../Pipeline/CompactionStepResultHandlerTest.php   | 494 +++++++++++----------
 .../Auth/CodexAccountIdExtractorTest.php           |  34 +-
 tests/CodingAgent/Auth/CodexAuthRecordTest.php     |   8 +-
 tests/CodingAgent/Auth/CodexAuthStorageTest.php    |  34 +-
 tests/CodingAgent/Auth/CodexOAuthProviderTest.php  |   8 +-
 tests/CodingAgent/Auth/CodexOAuthServiceTest.php   |  28 +-
 tests/CodingAgent/Auth/ManualCodeParserTest.php    |   1 -
 .../CLI/AgentCommandPromptTemplatesOptionsTest.php |  22 +-
 .../CLI/CompletionFileIndexRefreshCommandTest.php  |  45 +-
 .../CLI/FileMentionIndexBuilderTest.php            |  61 +--
 .../AutoCompactionHookSubscriberTest.php           | 287 ++++++------
 .../CodingAgentPreLlmCompactionGuardTest.php       |  84 ++--
 .../Compaction/CompactionHookDispatcherTest.php    |  41 +-
 .../Compaction/CompactionTokenEstimatorTest.php    |  10 +-
 .../ProviderContextUsageResolverTest.php           | 220 ++++-----
 .../Compaction/ToolResultDigestServiceTest.php     |  64 +--
 tests/CodingAgent/Config/AgentsConfigTest.php      |  49 +-
 .../Config/Ai/AiAgentRetryConfigTest.php           |  18 +-
 .../CodingAgent/Config/Ai/AiCostCalculatorTest.php |  14 +-
 tests/CodingAgent/Config/Ai/AiHttpConfigTest.php   |  94 ++--
 .../CodingAgent/Config/Ai/AiModelReferenceTest.php |  28 +-
 .../CodingAgent/Config/Ai/AiProviderConfigTest.php |   1 -
 .../Config/Ai/HatfieldModelCatalogTest.php         | 210 ++++-----
 tests/CodingAgent/Config/AppConfigLoaderTest.php   | 107 ++---
 tests/CodingAgent/Config/AppConfigTest.php         |  38 +-
 tests/CodingAgent/Config/CompactionConfigTest.php  |  71 ++-
 .../CodingAgent/Config/HomeSettingsWriterTest.php  |  95 ++--
 tests/CodingAgent/Config/ModelResolverTest.php     |  98 ++--
 .../Config/ModelSelectionServiceTest.php           | 444 +++++++++---------
 .../Config/ModelSettingsPersisterTest.php          |  29 +-
 tests/CodingAgent/Config/PromptsConfigTest.php     |  26 +-
 .../Config/ReasoningOptionsResolverTest.php        | 167 ++++---
 .../Config/SessionAwareModelResolverTest.php       |  38 +-
 .../Config/SettingsPathResolverTest.php            |  16 +-
 .../MessengerTransportSqliteIsolationTest.php      |   2 +-
 .../Doctrine/SqliteConnectionConfigTest.php        |   1 -
 .../Classifier/SafeGuardCommandMatcherTest.php     |  44 +-
 .../Classifier/SafeGuardPathMatcherTest.php        |  26 +-
 .../SafeGuard/SafeGuardToolCallHookTest.php        |  99 ++---
 .../CodingAgent/Extension/ExtensionManagerTest.php |   3 +-
 .../ExtensionToolHookEventSubscriberTest.php       | 128 +++---
 .../Extension/ExtensionToolRegistryBridgeTest.php  | 216 ++++-----
 .../Extension/InMemoryExtensionApiBridge.php       |   2 +-
 .../Extension/NoninteractiveChildRunProbeTest.php  |   4 +-
 .../TaskWorkflowExtensionIntegrationTest.php       |   2 +-
 .../SymfonyAi/Http/LlmHttpRetryPolicyTest.php      |  96 ++--
 .../SymfonyAi/Http/LlmRetryingHttpClientTest.php   |  54 +--
 .../SymfonyAi/ProjectedSymfonyModelCatalogTest.php |  78 ++--
 .../SymfonyAiProviderFactoryCodexAuthTest.php      |  74 +--
 .../SymfonyAi/SymfonyAiProviderFactoryTest.php     | 155 ++++---
 .../SymfonyAi/SymfonyAiProviderRegistryTest.php    |  18 +-
 .../Logging/LogContextProcessorTest.php            |  34 +-
 tests/CodingAgent/Logging/LogContextTest.php       |   2 +-
 tests/CodingAgent/Logging/LogFilterTest.php        |  52 +--
 tests/CodingAgent/Logging/LogParserTest.php        |  71 ++-
 tests/CodingAgent/Logging/LogReaderTest.php        |  69 ++-
 .../Mcp/Catalog/McpToolNameMapperTest.php          |  22 +-
 .../Catalog/SessionFileMcpToolCatalogStoreTest.php |  80 ++--
 .../Mcp/Client/McpConnectionManagerTest.php        | 245 +++++-----
 .../CodingAgent/Mcp/Config/McpConfigLoaderTest.php |   1 -
 .../CodingAgent/Mcp/Fixtures/http-echo-server.php  |  10 +-
 .../CodingAgent/Mcp/Fixtures/stdio-echo-server.php |   6 +-
 .../Handler/McpInitializeSessionHandlerTest.php    | 175 ++++----
 .../McpExecuteToolCallRoutingMiddlewareTest.php    |  17 +-
 .../McpCatalogRegisteringToolSetResolverTest.php   |   4 +-
 tests/CodingAgent/Mcp/Tool/McpToolHandlerTest.php  |  58 ++-
 .../CodingAgent/Mcp/Tool/McpToolRegistrarTest.php  |  36 +-
 .../MessengerTransportSchemaEnsurerTest.php        |   2 +-
 .../CodingAgent/Phar/KernelCacheIsolationTest.php  |   2 +-
 tests/CodingAgent/Phar/PharSmokeTest.php           | 123 ++---
 .../PromptTemplateArgumentParserTest.php           |  34 +-
 .../PromptTemplateFrontmatterParserTest.php        |  76 ++--
 .../PromptTemplate/PromptTemplateLoaderTest.php    | 148 +++---
 .../PromptTemplate/PromptTemplateServiceTest.php   |  54 +--
 .../PromptTemplateSubstitutorTest.php              |  50 +--
 .../BackgroundProcessCompletionPollerTest.php      | 105 ++---
 .../CommandHandler/AnswerHumanHandlerTest.php      |  64 +--
 .../AnswerToolQuestionHandlerTest.php              |  51 ++-
 .../CommandHandler/CompactHandlerTest.php          |  18 +-
 .../ExecuteShellToolCallWorkerTest.php             |  36 +-
 .../CommandHandler/ResumeHandlerTest.php           |  16 +-
 .../CommandHandler/ShellCommandHandlerTest.php     |  55 ++-
 .../Controller/E2E/ControllerE2eTestCase.php       | 170 ++++---
 ...ControllerReplayAutoCompactionMultiTurnTest.php | 334 +++++++-------
 ...ReplayAutoCompactionRepeatedReplicationTest.php | 358 +++++++--------
 ...ControllerReplayAutoCompactionToolCycleTest.php | 355 ++++++++-------
 .../E2E/ControllerReplayBashCancelFollowUpTest.php | 176 ++++----
 .../Controller/E2E/ControllerReplayE2eTestCase.php |  85 ++--
 .../Controller/E2E/ControllerReplaySmokeTest.php   |  66 +--
 .../E2E/ControllerReplaySummaryOnlyGuardTest.php   | 381 ++++++++--------
 .../Runtime/Controller/E2E/ControllerSmokeTest.php |  18 +-
 .../E2E/OutputCapReadFileControllerTest.php        |  73 ++-
 .../Replay/ControllerReplayHttpClientFactory.php   |   3 +-
 .../E2E/SafeGuardApprovalControllerReplayTest.php  | 114 ++---
 .../Controller/E2E/ShellFollowUpLiveE2eTest.php    |  59 ++-
 .../Controller/E2E/SubagentParallelLiveE2eTest.php | 107 +++--
 .../Controller/E2E/SubagentRetrieveLiveE2eTest.php | 124 +++---
 .../Controller/E2E/ViewImageToolE2eTest.php        |  58 +--
 .../Controller/E2E/WriteFileToolE2eTest.php        |  28 +-
 .../Runtime/Controller/LlmStdoutPollerTest.php     |  48 +-
 .../Runtime/Controller/RuntimeEventEmitterTest.php |   6 +-
 .../Runtime/Controller/ToolQuestionPollerTest.php  |  71 ++-
 .../InProcess/InMemoryRuntimeEventSinkTest.php     |  18 +-
 .../InProcessAttachDoesNotContinueTest.php         |   6 +-
 .../InProcessRewindEmitsRunLeafChangedTest.php     |  92 ++--
 .../PromptTemplateExpansionInProcessTest.php       | 164 ++++---
 .../InProcess/StartRunPersistsSessionModelTest.php | 191 ++++----
 tests/CodingAgent/Runtime/JsonlCodecTest.php       |  64 +--
 ...onlProcessAgentSessionClientEventBufferTest.php |  27 +-
 .../JsonlProcessPromptTemplateOptionsTest.php      |  33 +-
 .../JsonlProcessShellStandalonePayloadTest.php     |  17 +-
 .../Projection/SubagentProgressProjectionTest.php  |  81 ++--
 .../Runtime/Projection/TranscriptProjectorTest.php |   5 +-
 .../CompactionProjectionSubscriberTest.php         |  39 +-
 .../CodingAgent/Runtime/RuntimeEventMapperTest.php | 350 ++++++++-------
 tests/CodingAgent/Runtime/RuntimeEventTypeTest.php |   1 -
 .../Runtime/Stream/StreamDeltaSubscriberTest.php   | 108 ++---
 tests/CodingAgent/Session/AggregateResumeTest.php  |  38 +-
 .../Session/HatfieldSessionStoreTest.php           | 235 +++++-----
 .../Session/SessionRunEventStoreTest.php           |   2 +-
 .../Session/SessionTurnTreeProviderTest.php        |  78 ++--
 .../Skills/SkillsContextBuilderTest.php            |   8 +-
 .../CodingAgent/Support/TestDirectoryIsolation.php |  21 +-
 .../Support/TestDirectoryIsolationTest.php         |   2 +-
 .../SystemPrompt/SystemPromptBuilderTest.php       |   1 -
 .../TestCase/IsolatedKernelTestCase.php            | 112 ++---
 .../TestCase/PerMethodIsolatedKernelTestCase.php   |  26 +-
 tests/CodingAgent/Tool/AskHumanToolTest.php        |   7 -
 tests/CodingAgent/Tool/BashToolTest.php            |  31 +-
 tests/CodingAgent/Tool/BgStatusToolTest.php        |   8 +-
 tests/CodingAgent/Tool/EditFileToolTest.php        |   7 +-
 .../Tool/OutputCapLlmTransformHookTest.php         |   2 +-
 .../CodingAgent/Tool/RegistryBackedToolboxTest.php |   8 +-
 .../Store/DbalToolBatchStoreConcurrencyTest.php    |  34 +-
 .../BackgroundProcessStatusCheckerTest.php         |  10 +-
 .../RuntimeBashBackgroundPromptAdapterTest.php     | 117 +++--
 .../ToolQuestionAnswerResolverTest.php             |   4 +-
 .../Tool/ToolQuestion/ToolQuestionFactoryTest.php  |   8 +-
 .../Tool/ToolQuestion/ToolQuestionStoreTest.php    | 176 ++++----
 .../Bridge/Generic/DurableResultConverterTest.php  | 233 +++++-----
 .../Bridge/OpenAICodex/CodexContractTest.php       |  44 +-
 .../Bridge/OpenAICodex/CodexModelClientTest.php    |  22 +-
 .../Bridge/OpenAICodex/CodexSseStreamTest.php      |  42 +-
 .../Bridge/OpenAICodex/ResultConverterTest.php     |   3 +-
 .../Application/SessionInitializerReplayTest.php   |   5 +-
 tests/Tui/Application/SessionInitializerTest.php   | 143 +++---
 tests/Tui/Application/SessionSwitchServiceTest.php | 137 +++---
 .../TranscriptDisplayConfigMapperTest.php          |  30 +-
 tests/Tui/Command/CommandParserTest.php            | 112 ++---
 tests/Tui/Command/EchoHandler.php                  |   4 +-
 tests/Tui/Command/FixedMessageTestHandler.php      |   5 +-
 tests/Tui/Command/Hotkey/HotkeyRegistryTest.php    |  46 +-
 tests/Tui/Command/NoOpTestHandler.php              |   2 +-
 tests/Tui/Command/SlashCommandRegistryTest.php     | 180 ++++----
 tests/Tui/Command/SubagentLiveInputPolicyTest.php  |  30 +-
 tests/Tui/Command/SubmissionRouterTest.php         |  73 ++-
 .../CompactHeader/CompactHeaderPinnedOrderTest.php |  12 +-
 .../CompactHeaderSnapshotProviderTest.php          |  71 ++-
 .../Tui/CompactHeader/CompactHeaderWidgetTest.php  |  56 +--
 .../Completion/CompletionProviderRegistryTest.php  |   4 +-
 .../FileMentionCompletionProviderTest.php          |   2 +-
 .../Completion/SessionIdCompletionProviderTest.php |  60 +--
 tests/Tui/E2E/BashBackgroundAcceptE2eTest.php      |   4 +-
 tests/Tui/E2E/SafeGuardApprovalTuiE2eTest.php      | 153 ++++---
 tests/Tui/E2E/TmuxHarness.php                      | 417 +++++++++--------
 tests/Tui/E2E/TmuxPane.php                         |   3 +-
 .../Tui/E2E/TuiAskHumanOverlayMarkdownE2eTest.php  |  40 +-
 tests/Tui/E2E/TuiAutoCompactionCancelE2eTest.php   |  86 ++--
 tests/Tui/E2E/TuiAutoCompactionE2eTest.php         | 203 +++++----
 tests/Tui/E2E/TuiJourneyE2eTest.php                |  50 +--
 tests/Tui/E2E/TuiMarkdownRenderE2eTest.php         |  42 +-
 tests/Tui/E2E/TuiOutputCapNoticeE2eTest.php        |  53 ++-
 tests/Tui/E2E/TuiProviderErrorE2eTest.php          |  40 +-
 tests/Tui/E2E/TuiResumeModelRestoreE2eTest.php     |  62 +--
 .../TuiRichTranscriptProductValidationE2eTest.php  |  56 +--
 .../Tui/E2E/TuiStartupTranscriptConfigE2eTest.php  |  10 +-
 tests/Tui/E2E/TuiSubagentLiveViewE2eTest.php       |  20 +-
 tests/Tui/E2E/TuiSubagentProgressE2eTest.php       |  28 +-
 tests/Tui/E2E/TuiToolOutputE2eTest.php             |  76 ++--
 tests/Tui/E2E/TuiTranscriptRenderE2eTest.php       |  32 +-
 tests/Tui/E2E/TuiTreeCommandE2eTest.php            |   8 +-
 .../Extension/SlotBasedTuiExtensionContextTest.php |  34 +-
 .../Extension/TuiCommandRegistryAdapterTest.php    |  20 +-
 tests/Tui/Footer/FooterBarWidgetTest.php           |  30 +-
 tests/Tui/Layout/ChatLayoutTest.php                |  76 ++--
 tests/Tui/Layout/TuiSlotRegistryPriorityTest.php   |  12 +-
 tests/Tui/Layout/TuiSlotRegistryTest.php           |  75 ++--
 tests/Tui/Listener/CancelListenerTest.php          |   7 +-
 tests/Tui/Listener/CompactHeaderRegistrarTest.php  |  18 +-
 tests/Tui/Listener/CompletionListenerTest.php      |  16 +-
 tests/Tui/Listener/CopyCommandRegistrarTest.php    |   6 +-
 .../Listener/FooterStateSegmentProviderTest.php    |  71 ++-
 .../LoadedResourcesStartupRegistrarTest.php        |  19 +-
 tests/Tui/Listener/ModelCommandHandlerTest.php     | 377 ++++++++--------
 .../Tui/Listener/NewSessionCommandHandlerTest.php  |   6 +-
 .../Listener/PreviewExpansionInputListenerTest.php |  36 +-
 tests/Tui/Listener/PromptHistoryListenerTest.php   |   6 -
 .../PromptTemplateCommandRegistrarTest.php         |  74 +--
 .../Listener/RenameSessionCommandHandlerTest.php   | 124 +++---
 .../Listener/ResumeSessionCommandHandlerTest.php   | 156 +++----
 tests/Tui/Listener/SessionCommandRegistrarTest.php | 106 ++---
 .../Listener/SubmitListenerDispatchRuntimeTest.php |  82 ++--
 .../SubmitListenerSubagentLiveInputTest.php        |  44 +-
 .../Listener/TickPollListenerSubagentLiveTest.php  |  56 ++-
 tests/Tui/Listener/TickPollListenerTest.php        | 175 ++++----
 tests/Tui/Listener/TreeCommandHandlerTest.php      |   6 +-
 tests/Tui/Picker/ModelPickerControllerTest.php     |  34 +-
 tests/Tui/Picker/PickerOverlayTest.php             |  37 +-
 tests/Tui/Picker/SessionPickerControllerTest.php   |  50 +--
 tests/Tui/Picker/TreePickerControllerTest.php      | 191 ++++----
 tests/Tui/Question/QuestionCoordinatorTest.php     | 225 +++++-----
 tests/Tui/Question/QuestionRequestTest.php         |  66 ++-
 tests/Tui/Runtime/ActivityStateMachineTest.php     |   1 -
 tests/Tui/Runtime/RuntimeEventPollerTest.php       | 294 ++++++------
 tests/Tui/Runtime/SubagentLiveCatalogTest.php      |  18 +-
 tests/Tui/Runtime/TuiRuntimeEventApplierTest.php   |  39 +-
 .../Runtime/TuiSessionLifecycleDispatcherTest.php  |  76 ++--
 tests/Tui/Runtime/UsageProjectionTest.php          | 155 ++++---
 tests/Tui/Screen/ChatScreenTest.php                | 134 +++---
 tests/Tui/Screen/TuiExportCommandVirtualTest.php   |  22 +-
 tests/Tui/Screen/TuiFileCompletionRenderTest.php   |   8 +-
 .../Tui/Screen/TuiModelInteractionVirtualTest.php  |  19 +-
 tests/Tui/Screen/TuiReasoningCycleTest.php         |  22 +-
 tests/Tui/Screen/TuiStartupVirtualRenderTest.php   |   8 +-
 .../TuiTranscriptBlocksVirtualRenderTest.php       | 206 +++++----
 tests/Tui/Screen/TuiVirtualInputTest.php           |  30 +-
 tests/Tui/Startup/LoadedResourcesWidgetTest.php    |   1 -
 .../ResumeSessionInitializerTestFactory.php        |   2 +-
 .../Tui/Support/SubagentProgressEventsFixture.php  |   1 -
 .../Tui/Support/TuiRuntimeContextBuilderTrait.php  |   5 -
 tests/Tui/Support/VirtualTuiHarness.php            |   1 -
 tests/Tui/Theme/DefaultThemeTest.php               |  46 +-
 tests/Tui/Theme/ThemePaletteTest.php               |  41 +-
 tests/Tui/Theme/ThemeRegistryTest.php              |  19 +-
 tests/Tui/Transcript/HotkeyTableRendererTest.php   |  95 ++--
 .../Tui/Transcript/SubagentResultRendererTest.php  |  50 +--
 tests/Tui/Widget/TuiRenderContextTest.php          |  34 +-
 tests/paratest-bootstrap.php                       |  13 +-
 323 files changed, 10718 insertions(+), 10733 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/include-tests-in-castor-cs-scope.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/include-tests-in-castor-cs-scope.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-07-05T00:28:42.311Z
- Updated PR Status: merged
- move_task -> DONE: PR #261 merged (2026-07-05T00:27:45Z). Task branch merged into main (commit f4d9a3649 via merge 2816e133c). Integration checkout pulled, working tree clean, .php-cs-fixer.dist.php on main now includes tests/. Worktree removed and IDEA exclusions cleaned up.
