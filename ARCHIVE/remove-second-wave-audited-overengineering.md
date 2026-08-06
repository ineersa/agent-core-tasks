# Remove second-wave audited over-engineering and dead code

## Goal
Apply the verified follow-up ponytail-audit cuts as a behavior-preserving cleanup. Reconfirm references on the task worktree before each deletion because active TUI/compaction branches may have changed callers. Do not add compatibility shims or replacement abstractions.

Ranked scope:
1. Delete the test-only transcript rendering stack and its self-test; production uses `TranscriptMountedWidget`: `src/Tui/Transcript/{TranscriptBlockWidget,TranscriptBlockRenderer,SymfonyTuiWidgetRenderer}.php`, `tests/Tui/Transcript/TranscriptBlockRendererTest.php` (~2,190 lines).
2. Delete the self-referential lifecycle event hierarchy, validator, contract test, related `.toon` docs, and now-dead `RunEventTypeEnum::isLifecycleType()`; production emits `RunEvent`: `src/AgentCore/Domain/Event/Lifecycle/`, `LifecycleOrderValidator.php`, `LifecycleEventContractTest.php` (~710 lines).
3. Delete the flat TUI layout subsystem used only by its test; production uses `ChatScreen`: `ChatLayout`, `TranscriptWidget`, `TranscriptEntry`, `PromptEditorWidget`, `ChatLayoutTest` (~620 lines).
4. Delete `DomainMessageBuildersTest`; retain builders used by behavioral tests (~284 lines).
5. Replace duplicated test temp-directory setup/cleanup with `TestDirectoryIsolation` across `tests/CodingAgent/` (~195 lines).
6. Delete unreferenced Castor helpers `dev_php_exec`, `persist_process_output`, `phpunit_inputs_available`, `write_empty_junit_report`, and `phpunit_risky_summary` (~106 lines).
7. Delete `HomeSettingsWriter`; use `SettingsOverrideWriter::set(User, …)` (~70 lines).
8. Remove `CommandHandlerRegistry` and `HookSubscriberRegistry`; inject tagged iterables directly (~55 lines).
9. Remove write-only batch supervision fields, unused completion enum cases, and discarded abort results; pass item lists directly (~50 lines).
10. Delete `ReadonlyFooterDataProvider` and `FooterDataProvider::readonly()` (~41 lines).
11. Delete `ChildRunTerminalFinalizationResultDTO`; make terminal finalization return `void` (~35 lines).
12. Delete test-only `HotPromptStateStore`; use production-wired `InMemoryPromptStateStore` (~37 lines).
13. Delete test-only `TuiRenderContext::withWidth()`, `withHeight()`, and `withTheme()` (~36 lines).
14. Delete duplicate `ActiveModelResolverInterface` and deprecated `getActiveModel()` alias; use `RunModelResolverInterface::resolveActiveModel()` (~20 lines).
15. Delete unwired `ToolIdempotencyKeyResolverInterface` and optional `ToolExecutor` plumbing (~19 lines).
16. Delete unwired SafeGuard `allow_destructive_in_paths` setting/config/policy/test/docs state (~18 lines).
17. Replace no-argument `SubagentChildRunBatchLifecyclePolicyFactory` with a static default constructor or direct DTO construction (~14 lines).
18. Delete stateless `LogReaderFactory`; wire `LogReader` through Symfony DI (~14 lines).
19. Delete empty, unreferenced `SessionStore` (~9 lines).
20. Delete unused legacy `app.default_model` parameter and stale comments (~6 lines).
21. Remove unused direct `symfony/expression-language` dependency.

Estimated opportunity: approximately 4,500 lines and one direct dependency.

Before touching tests/runtime/TUI or running QA, load `.agents/skills/testing/SKILL.md` and read `tests/AGENTS.md`. Use Castor exclusively for QA and Composer/tooling operations.

## Acceptance criteria
- Every listed candidate is revalidated against current production references, callers, DI wiring, and active architecture before removal; changed/live candidates are documented and skipped rather than forced.
- Verified dead/test-only classes, methods, fields, configuration, helpers, and tests are removed without replacement abstractions or compatibility paths.
- Production behavior remains on the canonical paths: `TranscriptMountedWidget`/`ChatScreen`, plain `RunEvent`, `SettingsOverrideWriter`, `InMemoryPromptStateStore`, `RunModelResolverInterface`, and Symfony DI.
- Tests use shared `TestDirectoryIsolation`; tests that only exercise removed APIs are deleted, while user-visible and stable-contract proofs remain.
- `symfony/expression-language` is removed from `composer.json` and the lockfile is updated through Castor tooling if no current dependency or configuration requires it.
- Comments, docs, service wiring, Deptrac rules, and stale symbol references are updated while preserving non-obvious rationale comments unrelated to removed logic.
- Focused Castor validation passes: `castor test`, `castor deptrac`, `castor phpstan`, and `castor cs-check`.
- Because TUI/runtime paths are touched, deterministic `castor check` passes with no leaked workers before CODE-REVIEW.

## Workflow metadata
Status: ARCHIVE
Branch: task/remove-second-wave-audited-overengineering
Worktree: /home/ineersa/projects/agent-core-worktrees/remove-second-wave-audited-overengineering
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/328
PR Status: merged
Started: 2026-07-27T23:36:18.821Z
Completed: 2026-07-28T01:42:38.409Z

## Work log
- Created: 2026-07-27T23:35:38.082Z

## Task workflow update - 2026-07-27T23:36:18.821Z
- Moved TODO → IN-PROGRESS.
- Created branch task/remove-second-wave-audited-overengineering.
- Created worktree /home/ineersa/projects/agent-core-worktrees/remove-second-wave-audited-overengineering.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/remove-second-wave-audited-overengineering.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/remove-second-wave-audited-overengineering.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/remove-second-wave-audited-overengineering.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/remove-second-wave-audited-overengineering.
- Summary: Claimed for implementation after verified repository-wide ponytail audit. Main agent will orchestrate scouts and an implementation fork; no direct edits in integration checkout.

## Task workflow update - 2026-07-28T00:02:26.423Z
- Validation: castor test — PASS (4402 tests, 15476 assertions); castor deptrac — PASS (0 violations); castor phpstan — PASS (0 errors); castor cs-check — PASS; Commit 3d049cad25293497e3d07a3101b84f69c4310d23 exists; worktree clean; 133 files changed, +389/-5031
- Summary: Implementation committed as 3d049cad25293497e3d07a3101b84f69c4310d23. Implemented audit items 1–20, removing obsolete TUI render/layout paths, lifecycle event island, redundant tests/helpers/wrappers/config, and simplifying child-run/model/log/settings wiring. Item 21 was correctly skipped after revalidation: `symfony/expression-language` is live because `config/services.yaml` contains many `@=` expressions. Existing virtual mounted-transcript/ChatScreen tests remain the lowest-layer TUI behavior proof. Fork confirmed it read `.agents/skills/testing/SKILL.md`, `tests/AGENTS.md`, and `.agents/skills/castor/SKILL.md`. Worktree is clean; diff is 133 files, +389/-5031.

## Task workflow update - 2026-07-28T00:34:41.987Z
- Validation: Reviewer final verdict — APPROVED (HEAD 9387497b1c1f39a4df0452f85211b539017f72b2); castor test — PASS (4404 tests, 15481 assertions); castor test:tui — PASS (37 tests, 190 assertions); castor deptrac — PASS (0 violations, 0 errors); castor phpstan — PASS (0 errors); castor cs-check — PASS (0 files fixed); Worktree clean; 134 files changed, +436/-5029 versus origin/main
- Summary: Task-to-PR review completed. Initial reviewer approved the deletion pass but identified a lost intermediate-map validation contract and two stale comments. Fix commits ec7184701533d0dd0868c687389c85ff1a013b3c and 9387497b1c1f39a4df0452f85211b539017f72b2 restore fail-closed scalar/list/null parent validation with focused tests and remove stale comments. Final reviewer APPROVED HEAD 9387497b1 with no blockers and confirmed full testing-skill/tests-AGENTS compliance. `symfony/expression-language` remains correctly required by active `@=` DI expressions. Worktree clean; branch diff 134 files, +436/-5029.

## Task workflow update - 2026-07-28T00:37:01.998Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (125.2s).
- Pushed task/remove-second-wave-audited-overengineering to origin.
- branch 'task/remove-second-wave-audited-overengineering' set up to track 'origin/task/remove-second-wave-audited-overengineering'.
- Created PR: https://github.com/ineersa/agent-core/pull/328
- Validation: castor test — PASS (4404 tests, 15481 assertions); castor test:tui — PASS (37 tests, 190 assertions); castor deptrac — PASS; castor phpstan — PASS; castor cs-check — PASS
- Summary: Reviewer APPROVED HEAD 9387497b1. Focused unit/static/TUI replay validation passed; transitioning through deterministic castor check gate, push, and PR creation.

## Task workflow update - 2026-07-28T01:42:38.409Z
- Moved CODE-REVIEW → DONE.
- Merged task/remove-second-wave-audited-overengineering into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/helpers.php                                |   57 -
 .castor/shared.php                                 |   43 -
 AGENTS.md                                          |    2 +-
 config/hatfield.defaults.yaml                      |    4 -
 config/services.yaml                               |   27 +-
 docs/settings.md                                   |    1 -
 docs/tui-architecture.md                           |    3 -
 .../Application/Handler/CommandHandlerRegistry.php |   34 -
 .../Application/Handler/CommandRouter.php          |   24 +-
 .../Application/Handler/HookDispatcher.php         |    8 +-
 .../Application/Handler/HookSubscriberRegistry.php |   31 -
 src/AgentCore/Application/Handler/ToolExecutor.php |    6 +-
 .../Tool/ToolIdempotencyKeyResolverInterface.php   |   12 -
 .../Event/Lifecycle/AbstractLifecycleRunEvent.php  |   34 -
 .../Domain/Event/Lifecycle/AgentEndEvent.php       |   12 -
 .../Domain/Event/Lifecycle/AgentStartEvent.php     |   12 -
 .../Domain/Event/Lifecycle/MessageEndEvent.php     |   12 -
 .../Domain/Event/Lifecycle/MessageStartEvent.php   |   12 -
 .../Domain/Event/Lifecycle/MessageUpdateEvent.php  |   12 -
 .../Event/Lifecycle/ToolExecutionEndEvent.php      |   12 -
 .../Event/Lifecycle/ToolExecutionStartEvent.php    |   12 -
 .../Event/Lifecycle/ToolExecutionUpdateEvent.php   |   12 -
 .../Domain/Event/Lifecycle/TurnEndEvent.php        |   12 -
 .../Domain/Event/Lifecycle/TurnStartEvent.php      |   12 -
 .../Domain/Event/LifecycleOrderValidator.php       |  161 --
 src/AgentCore/Domain/Event/RunEventTypeEnum.php    |   21 -
 .../Infrastructure/Storage/HotPromptStateStore.php |   37 -
 .../SymfonyAi/LlmPlatformAdapter.php               |    2 +-
 .../Contract/ChildRunBatchCompletionKindEnum.php   |   16 -
 .../Contract/ChildRunBatchSupervisionResultDTO.php |    3 -
 .../ChildRunTerminalFinalizationResultDTO.php      |   27 -
 .../Lifecycle/ChildRunBatchLaunchService.php       |   18 +-
 .../ChildRunBatchLifecycleListenerInterface.php    |    3 +-
 ...erredSubagentBatchTerminalCompletionService.php |    4 -
 ...dSubagentBatchInterruptionCompletionService.php |    2 -
 .../DeferredSubagentBatchInterruptionService.php   |    6 +-
 .../DeferredSubagentBatchRuntimeStartService.php   |    7 +-
 .../SubagentChildRunBatchLifecycleListener.php     |   54 +-
 ...SubagentChildRunBatchLifecyclePolicyFactory.php |   21 -
 src/CodingAgent/CLI/Log/LogClearCommand.php        |    6 +-
 src/CodingAgent/CLI/Log/LogFilesCommand.php        |    6 +-
 src/CodingAgent/CLI/Log/LogSearchCommand.php       |    6 +-
 src/CodingAgent/CLI/Log/LogTailCommand.php         |    6 +-
 .../Compaction/ActiveModelResolverInterface.php    |   23 -
 .../Compaction/AutoCompactionHookSubscriber.php    |    5 +-
 .../CodingAgentPreLlmCompactionGuard.php           |    5 +-
 .../ModelSelectionActiveModelResolver.php          |    8 +-
 src/CodingAgent/Config/HomeSettingsWriter.php      |  139 --
 src/CodingAgent/Config/ModelSelectionService.php   |    4 +-
 src/CodingAgent/Config/ModelSettingsPersister.php  |    9 +-
 src/CodingAgent/Config/SettingsOverrideWriter.php  |   33 +
 .../Builtin/SafeGuard/Policy/SafeGuardPolicy.php   |    7 -
 .../Builtin/SafeGuard/SafeGuardConfig.php          |    3 -
 src/CodingAgent/Logging/LogReaderFactory.php       |   34 -
 src/CodingAgent/Session/SessionStore.php           |   10 -
 src/Tui/Editor/PromptEditor.php                    |    3 +-
 src/Tui/Editor/PromptEditorWidget.php              |   51 -
 src/Tui/Footer/FooterDataProvider.php              |    5 -
 src/Tui/Footer/ReadonlyFooterDataProvider.php      |   35 -
 src/Tui/Layout/ChatLayout.php                      |  140 --
 src/Tui/Layout/TuiSlotRegistry.php                 |    2 +-
 src/Tui/Status/StatusPanelWidget.php               |    2 +-
 src/Tui/Transcript/SymfonyTuiWidgetRenderer.php    |   84 -
 src/Tui/Transcript/TranscriptBlockRenderer.php     |   52 -
 src/Tui/Transcript/TranscriptBlockWidget.php       |  304 ----
 .../Transcript/TranscriptBlockWidgetFactory.php    |    2 +-
 src/Tui/Transcript/TranscriptEntry.php             |   60 -
 src/Tui/Transcript/TranscriptGlyphs.php            |    2 +-
 src/Tui/Transcript/TranscriptWidget.php            |   54 -
 src/Tui/Widget/TuiRenderContext.php                |   36 -
 src/Tui/Widget/TuiWidget.php                       |    4 +-
 .../Handler/CommandRouterContractTest.php          |   23 +-
 .../Handler/HookDispatcherContractTest.php         |    5 +-
 .../Handler/ToolCallHumanInputSuspensionTest.php   |    8 +-
 .../Application/Pipeline/AdvanceRunHandlerTest.php |   31 +-
 .../Pipeline/ApplyCommandHandlerTest.php           |   51 +-
 .../Pipeline/CommandMailboxPolicyTest.php          |    9 +-
 .../Pipeline/LlmStepResultHandlerTest.php          |   23 +-
 .../PendingHumanInputAnswerValidationTest.php      |    3 +-
 .../RunCommitAfterTurnCommitPersistedSeqTest.php   |    3 +-
 .../Pipeline/RunStateModelIdentityTest.php         |    7 +-
 .../Contract/LifecycleEventContractTest.php        |  289 ----
 .../Infrastructure/SymfonyAi/LlamaCppSmokeTest.php |    6 +-
 .../SymfonyAi/PlatformIntegrationTest.php          |    4 +-
 .../Infrastructure/SymfonyAi/TraceReplayTest.php   |    8 +-
 .../Support/Builder/DomainMessageBuildersTest.php  |  284 ----
 .../Launch/DeferredSubagentBatchLaunchTest.php     |    2 +-
 .../DeferredSubagentBatchLifecycleTest.php         |    4 +-
 .../Execution/SubagentExecutionServiceTest.php     |   23 -
 .../Support/PipelineCapturingAgentRunner.php       |    2 +-
 .../Support/SubagentExecutionServiceFactory.php    |   12 +-
 tests/CodingAgent/Auth/CodexAuthStorageTest.php    |    3 +-
 tests/CodingAgent/Auth/CodexOAuthServiceTest.php   |    3 +-
 .../CLI/CompletionFileIndexRefreshCommandTest.php  |   33 +-
 .../CLI/FileMentionIndexBuilderTest.php            |   34 +-
 .../AutoCompactionHookSubscriberTest.php           |   54 +-
 .../CodingAgentPreLlmCompactionGuardTest.php       |   12 +-
 .../Config/ModelSelectionServiceTest.php           |   10 +-
 .../Config/ModelSettingsPersisterTest.php          |    8 +-
 .../Config/SessionAwareModelResolverTest.php       |    6 +-
 ...st.php => SettingsOverrideWriterHomeAiTest.php} |   57 +-
 .../SafeGuard/Policy/SafeGuardPolicyTest.php       |   16 -
 .../Builtin/SafeGuard/SafeGuardConfigTest.php      |    3 -
 .../CodingAgent/Extension/ExtensionManagerTest.php |   27 +-
 .../Codex/CodexSymfonyAiProviderBuilderTest.php    |    3 +-
 tests/CodingAgent/Logging/LogReaderTest.php        |    3 +-
 .../BackgroundProcessCompletionPollerTest.php      |    4 +-
 .../ExecuteShellToolCallWorkerTest.php             |    2 +-
 .../ParentPromptUserContextRegressionTest.php      |    4 +-
 tests/CodingAgent/Session/AggregateResumeTest.php  |    3 +-
 .../Replay/SessionHotPromptReplayServiceTest.php   |   20 +-
 .../Session/SessionRunEventStoreTest.php           |    3 +-
 tests/CodingAgent/Session/SessionRunStoreTest.php  |    3 +-
 .../ToolBatchSnapshotCleanupHookSubscriberTest.php |    5 +-
 tests/CodingAgent/Skills/SkillDiscoveryTest.php    |   27 +-
 tests/CodingAgent/Skills/SkillRegistryTest.php     |   28 +-
 .../Skills/SkillsContextBuilderTest.php            |   27 +-
 .../SystemPrompt/AgentsContextDiscoveryTest.php    |   27 +-
 .../SystemPrompt/SystemPromptBuilderTest.php       |   28 +-
 .../Tool/BackgroundProcessManagerTest.php          |    5 +-
 .../ImageAttachmentProcessorTest.php               |    4 +-
 .../Tool/OutputCapLlmTransformHookTest.php         |   27 +-
 tests/CodingAgent/Tool/OutputCapTest.php           |   27 +-
 tests/CodingAgent/Tool/ReadFileToolTest.php        |    4 +-
 tests/CodingAgent/Tool/ViewImageToolTest.php       |    4 +-
 tests/CodingAgent/Tool/WriteFileToolTest.php       |    4 +-
 tests/Tui/E2E/TuiJourneyE2eTest.php                |    2 +-
 tests/Tui/Footer/FooterBarWidgetTest.php           |   17 -
 tests/Tui/Layout/ChatLayoutTest.php                |  315 ----
 tests/Tui/Listener/ModelCommandHandlerTest.php     |   12 +-
 tests/Tui/Picker/ModelPickerControllerTest.php     |    6 +-
 .../Tui/Transcript/SubagentResultRendererTest.php  |   54 +-
 .../Tui/Transcript/TranscriptBlockRendererTest.php | 1765 --------------------
 tests/Tui/Widget/TuiRenderContextTest.php          |   41 -
 134 files changed, 436 insertions(+), 5029 deletions(-)
 delete mode 100644 src/AgentCore/Application/Handler/CommandHandlerRegistry.php
 delete mode 100644 src/AgentCore/Application/Handler/HookSubscriberRegistry.php
 delete mode 100644 src/AgentCore/Contract/Tool/ToolIdempotencyKeyResolverInterface.php
 delete mode 100644 src/AgentCore/Domain/Event/Lifecycle/AbstractLifecycleRunEvent.php
 delete mode 100644 src/AgentCore/Domain/Event/Lifecycle/AgentEndEvent.php
 delete mode 100644 src/AgentCore/Domain/Event/Lifecycle/AgentStartEvent.php
 delete mode 100644 src/AgentCore/Domain/Event/Lifecycle/MessageEndEvent.php
 delete mode 100644 src/AgentCore/Domain/Event/Lifecycle/MessageStartEvent.php
 delete mode 100644 src/AgentCore/Domain/Event/Lifecycle/MessageUpdateEvent.php
 delete mode 100644 src/AgentCore/Domain/Event/Lifecycle/ToolExecutionEndEvent.php
 delete mode 100644 src/AgentCore/Domain/Event/Lifecycle/ToolExecutionStartEvent.php
 delete mode 100644 src/AgentCore/Domain/Event/Lifecycle/ToolExecutionUpdateEvent.php
 delete mode 100644 src/AgentCore/Domain/Event/Lifecycle/TurnEndEvent.php
 delete mode 100644 src/AgentCore/Domain/Event/Lifecycle/TurnStartEvent.php
 delete mode 100644 src/AgentCore/Domain/Event/LifecycleOrderValidator.php
 delete mode 100644 src/AgentCore/Infrastructure/Storage/HotPromptStateStore.php
 delete mode 100644 src/CodingAgent/Agent/Execution/ChildRun/Contract/ChildRunBatchCompletionKindEnum.php
 delete mode 100644 src/CodingAgent/Agent/Execution/ChildRun/Contract/ChildRunTerminalFinalizationResultDTO.php
 delete mode 100644 src/CodingAgent/Agent/Execution/Subagent/SubagentChildRunBatchLifecyclePolicyFactory.php
 delete mode 100644 src/CodingAgent/Compaction/ActiveModelResolverInterface.php
 delete mode 100644 src/CodingAgent/Config/HomeSettingsWriter.php
 delete mode 100644 src/CodingAgent/Logging/LogReaderFactory.php
 delete mode 100644 src/CodingAgent/Session/SessionStore.php
 delete mode 100644 src/Tui/Editor/PromptEditorWidget.php
 delete mode 100644 src/Tui/Footer/ReadonlyFooterDataProvider.php
 delete mode 100644 src/Tui/Layout/ChatLayout.php
 delete mode 100644 src/Tui/Transcript/SymfonyTuiWidgetRenderer.php
 delete mode 100644 src/Tui/Transcript/TranscriptBlockRenderer.php
 delete mode 100644 src/Tui/Transcript/TranscriptBlockWidget.php
 delete mode 100644 src/Tui/Transcript/TranscriptEntry.php
 delete mode 100644 src/Tui/Transcript/TranscriptWidget.php
 delete mode 100644 tests/AgentCore/Contract/LifecycleEventContractTest.php
 delete mode 100644 tests/AgentCore/Support/Builder/DomainMessageBuildersTest.php
 rename tests/CodingAgent/Config/{HomeSettingsWriterTest.php => SettingsOverrideWriterHomeAiTest.php} (70%)
 delete mode 100644 tests/Tui/Layout/ChatLayoutTest.php
 delete mode 100644 tests/Tui/Transcript/TranscriptBlockRendererTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/remove-second-wave-audited-overengineering.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/remove-second-wave-audited-overengineering.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: gh pr view 328 — state MERGED, merge commit 15f6336deb2edd2ffd309141daeff18de481e6db; Integration checkout clean before DONE transition
- Summary: PR #328 confirmed merged on GitHub at 2026-07-28T01:41:50Z (merge commit 15f6336deb2edd2ffd309141daeff18de481e6db). Merging/syncing task branch into integration checkout and cleaning worktree.

## Task workflow update - 2026-07-28T01:45:11.721Z
- Validation: PR #328 — MERGED (15f6336deb2edd2ffd309141daeff18de481e6db); Post-merge `LLM_MODE=true castor check` — PASS (372.5s): test 4399/15424, controller replay 10/135, TUI 37/190, llm-real 13/175, deptrac/phpstan/cs-check green; llama-proxy cache guard — PASS (181 → 181 entries); QA artifact integrity — PASS (7 lane logs); QA leak check — PASS (no owned processes/tmux sessions); Integration checkout clean; task worktree removed
- Summary: Task completed. PR #328 merged; task branch merged/synced into integration checkout; task worktree and IDEA exclusions removed. Post-merge deterministic gate passed with cache guard, artifact integrity, and leak checks green. Integration checkout clean.

## Task workflow update - 2026-08-06T20:59:22.882Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
