# Remove audited dead code and thin wrappers

## Goal
Apply the validated ponytail-audit cuts as one behavior-preserving cleanup. Do not introduce compatibility shims or replacement abstractions.

Scope:
- Delete `RunDebugService` and its four snapshot DTOs; nothing replaces them until an inspect command exists.
- Delete unreferenced `RunReadService`.
- Delete unfinished, unreferenced `AgentProcessSupervisor`; existing runtime supervisors remain canonical.
- Remove concrete `SessionMetadataStore`; inject `HatfieldSessionStore` directly into its consumers.
- Delete superseded, unreferenced `JsonlRuntimeEventSink`.
- Remove test-only `CompactionConfig::resolveModelReference()` and tests that solely exercise it; production continues using `resolveRuntimeSettings()`.
- Delete unreferenced `RunAccessScope`.
- Remove `ThemeFactory`; inline its minimal theme construction into `InteractiveMode`.
- Delete empty `SkillsProvider`, `FindTool`, and `GrepTool` placeholders.
- Remove `MessageIdempotencyService`; inject/use `IdempotencyStoreInterface` directly in `RunMessageProcessor`.
- Delete unused TUI container parameters in `config/packages/tui.yaml`.
- Remove backward-compatibility-only `RuntimeEventMapper::toRunEventData()` and its isolated test.
- Delete unused `BusNames` constants class.
- Remove legacy `cs_fixer` Castor alias; retain canonical `cs-fix`.
- Remove broken Castor `audit` task referencing the uninstalled security-checker.
- Replace `symfony/test-pack` with direct `phpunit/phpunit`, removing unused browser-testing packages from the lockfile.

Before touching tests/runtime/TUI or running QA, load `.agents/skills/testing/SKILL.md` and read `tests/AGENTS.md`. All QA and Composer/tooling operations must go through Castor.

## Acceptance criteria
- All listed dead classes, methods, DTOs, placeholders, parameters, and Castor aliases/tasks are removed, with no new replacement abstractions or compatibility paths.
- `SessionMetadataStore`, `ThemeFactory`, and `MessageIdempotencyService` consumers use the existing concrete/service contracts directly while preserving behavior.
- Tests that exist only for removed APIs are deleted or updated; no production API is added solely for tests.
- `composer.json` directly requires PHPUnit instead of `symfony/test-pack`, and the lockfile no longer retains browser-testing packages unless Composer proves another dependency needs them.
- Comments/docs/service wiring referencing removed symbols are updated or deleted, while non-obvious rationale comments unrelated to removed logic are preserved.
- Focused Castor validation passes (`castor test`, `castor deptrac`, `castor phpstan`, `castor cs-check`).
- Because runtime and TUI wiring are touched, deterministic `castor check` passes before transition to CODE-REVIEW, with no leaked workers.

## Workflow metadata
Status: DONE
Branch: task/remove-audited-dead-code-thin-wrappers
Worktree: /home/ineersa/projects/agent-core-worktrees/remove-audited-dead-code-thin-wrappers
Fork run: 9a27mlwext9y
PR URL: https://github.com/ineersa/agent-core/pull/324
PR Status: merged
Started: 2026-07-27T15:40:02.603Z
Completed: 2026-07-27T16:34:46.073Z

## Work log
- Created: 2026-07-27T03:00:53.951Z

## Task workflow update - 2026-07-27T15:40:02.603Z
- Moved TODO → IN-PROGRESS.
- Created branch task/remove-audited-dead-code-thin-wrappers.
- Created worktree /home/ineersa/projects/agent-core-worktrees/remove-audited-dead-code-thin-wrappers.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/remove-audited-dead-code-thin-wrappers.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/remove-audited-dead-code-thin-wrappers.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/remove-audited-dead-code-thin-wrappers.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/remove-audited-dead-code-thin-wrappers.
- Summary: Claimed for implementation via orchestrated fork.

## Task workflow update - 2026-07-27T15:48:06.713Z
- Recorded fork run: r0vuf7njgk1f
- Summary: Implementation fork launched in task worktree. Scope: audited deletions/direct wiring, dependency cleanup, focused Castor validation, and existing replay-backed TmuxHarness startup proof. Full castor check intentionally deferred to task-to-pr.
- Scouts mapped production references, test constructor fallout, Castor aliases/tasks, and Composer lock removals. Testing skill and tests/AGENTS.md were read before test planning.

## Task workflow update - 2026-07-27T15:54:58.480Z
- Recorded fork run: r0vuf7njgk1f
- Validation: castor test --filter=... — PASS (213 tests); castor test — PASS (4527 tests, 15798 assertions); castor test:tui --filter=TuiStartupSnapshotTest — PASS (1 test, 3 assertions; real TmuxHarness replay proof); castor deptrac — PASS after explicit AppRuntimeInternals→AppSession allowance; castor phpstan — PASS (0 errors); castor cs-check — PASS; Commit ffc9a60a67f1047d31d31993d2f5b4dade9feb5f exists; worktree clean; removed-symbol reference scan clean apart from retained php-cs-fixer summary helper names
- Summary: Implementation committed as ffc9a60a67f1047d31d31993d2f5b4dade9feb5f. Verified clean worktree and expected 52-file deletion/rewire diff (+63/-1349). Removed audited symbols, wrappers, Castor aliases/tasks, test-pack/browser lock dependencies; rewired session metadata, idempotency, and theme construction. Added AppRuntimeInternals→AppSession Deptrac allowance required by direct HatfieldSessionStore injection. Existing replay-backed TmuxHarness startup test proves InteractiveMode/default-theme boot. Full castor check remains deferred to task-to-pr.
- Fork confirmed it read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before test/runtime/TUI work and used Castor-only QA/tooling.
- Temporary Castor Composer removal task was removed before commit. No castor check, push, PR, or reviewer step was run.

## Task workflow update - 2026-07-27T16:03:34.847Z
- Recorded fork run: 9a27mlwext9y
- Summary: Reviewer returned APPROVE WITH SUGGESTIONS on ffc9a60a6. Code/architecture/TmuxHarness proof approved; sole actionable suggestion is removal of one stale config/services.yaml section header. Narrow fix fork launched. Subjective Deptrac rationale comment and cosmetic test-helper renames intentionally skipped.
- Reviewer confirmed testing skill/tests AGENTS conventions, accepted direct AppRuntimeInternals→AppSession dependency as required by task, and verified replay-backed TmuxHarness startup proof. Deterministic castor check remains the CODE-REVIEW transition gate.

## Task workflow update - 2026-07-27T16:10:47.619Z
- Recorded fork run: 9a27mlwext9y
- Validation: castor test — PASS (4527 tests, 15796 assertions); castor deptrac — PASS (0 violations, 0 errors); castor phpstan — PASS (0 errors); castor cs-check — PASS (0 files fixed); castor test:tui — PASS (37 tests, 190 assertions; full replay-backed TmuxHarness lane); Focused validation log: /tmp/remove-audited-dead-code-focused-validation.log; Reviewer verdict on HEAD dcfe8e0c3: APPROVED
- Summary: Reviewer follow-up APPROVED current HEAD dcfe8e0c339fadee9a77c503fe1a7aff01f5f550 with no remaining actionable findings. Reviewer verified architecture, direct AppSession dependency, stale-reference cleanup, and real replay-backed TmuxHarness startup proof. Fix commit dcfe8e0c3 removed the sole stale services.yaml header suggestion; branch is clean.
- Reviewer suggestion fix committed as dcfe8e0c339fadee9a77c503fe1a7aff01f5f550 (`Remove stale session-store section header in services.yaml`).
- Skipped only reviewer items explicitly deemed subjective/cosmetic/non-actionable: optional Deptrac rationale comment and local test-helper renames.

## Task workflow update - 2026-07-27T16:16:04.058Z
- Validation: castor clean:cleanup:workers:list — no stale QA worker candidates; castor test:controller-replay — PASS (10 tests, 135 assertions) after gate-only failure; First deterministic castor check attempt — FAIL only test:controller-replay; all other lanes completed
- Summary: First CODE-REVIEW gate attempt failed only in test:controller-replay: ControllerReplaySmokeTest observed transient missing session metadata under parallel check load. No leaked worker candidates were present. Immediate isolated Castor replay rerun passed all 10 tests; no code changed, so reviewer APPROVED verdict remains current.

## Task workflow update - 2026-07-27T16:18:15.844Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (118.9s).
- Pushed task/remove-audited-dead-code-thin-wrappers to origin.
- branch 'task/remove-audited-dead-code-thin-wrappers' set up to track 'origin/task/remove-audited-dead-code-thin-wrappers'.
- Created PR: https://github.com/ineersa/agent-core/pull/324
- Validation: castor test — PASS (4527 tests, 15796 assertions); castor deptrac — PASS; castor phpstan — PASS; castor cs-check — PASS; castor test:tui — PASS (37 tests, 190 assertions); castor test:controller-replay — PASS (10 tests, 135 assertions) after initial gate-only failure
- Summary: Reviewer APPROVED HEAD dcfe8e0c3. Focused validation and full TUI replay passed. Initial gate-only controller replay failure passed immediately in isolated rerun with no stale workers; retrying deterministic gate.

## Task workflow update - 2026-07-27T16:18:21.994Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/324
- Updated PR Status: open
- Validation: Deterministic castor check — PASS (118.9s); PR: https://github.com/ineersa/agent-core/pull/324
- Summary: Prepared for code review. Reviewer APPROVED current HEAD dcfe8e0c339fadee9a77c503fe1a7aff01f5f550. Deterministic castor check passed on retry (118.9s), branch pushed, PR #324 created.

## Task workflow update - 2026-07-27T16:34:46.073Z
- Moved CODE-REVIEW → DONE.
- Merged task/remove-audited-dead-code-thin-wrappers into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/tools.php                                  |  23 --
 composer.json                                      |   1 -
 composer.lock                                      | 266 +--------------------
 config/packages/tui.yaml                           |  14 +-
 config/services.yaml                               |   2 -
 depfile.yaml                                       |   1 +
 docs/session-storage.md                            |  11 +-
 docs/tui-architecture.md                           |   4 +-
 src/AgentCore/Application/AGENTS.md                |   1 -
 .../Application/Dto/HotPromptStateSnapshot.php     |  43 ----
 .../Application/Dto/PendingCommandSnapshot.php     |  19 --
 src/AgentCore/Application/Dto/RunDebugSnapshot.php |  23 --
 src/AgentCore/Application/Dto/RunStateSnapshot.php |  20 --
 .../Handler/MessageIdempotencyService.php          |  34 ---
 .../Application/Handler/RunDebugService.php        | 216 -----------------
 .../Application/Pipeline/RunMessageProcessor.php   |   6 +-
 src/AgentCore/Application/RunReadService.php       | 223 -----------------
 src/AgentCore/Domain/Run/RunAccessScope.php        |  35 ---
 .../Infrastructure/Messenger/BusNames.php          |  11 -
 src/CodingAgent/Config/CompactionConfig.php        |  17 --
 src/CodingAgent/Config/ModelResolver.php           |   5 +-
 src/CodingAgent/Config/ModelSettingsPersister.php  |   8 +-
 .../Config/SessionAwareModelResolver.php           |   3 +-
 src/CodingAgent/Config/SessionMetadataStore.php    |  59 -----
 .../Controller/CommandHandler/StartRunHandler.php  |   4 +-
 .../InProcess/InProcessAgentSessionClient.php      |   6 +-
 src/CodingAgent/Runtime/Process/AGENTS.md          |   2 -
 .../Runtime/Process/AgentProcessSupervisor.php     |  80 -------
 .../Runtime/Process/JsonlRuntimeEventSink.php      |  47 ----
 .../Runtime/Protocol/RuntimeEventMapper.php        |  19 --
 src/CodingAgent/Session/SkillsProvider.php         |  10 -
 src/CodingAgent/Tool/FindTool.php                  |  10 -
 src/CodingAgent/Tool/GrepTool.php                  |  10 -
 src/Tui/Application/InteractiveMode.php            |   6 +-
 src/Tui/Application/ThemeFactory.php               |  40 ----
 .../Handler/DeferredToolCompletionRuntimeTest.php  |   3 +-
 .../Handler/InMemoryIdempotencyStore.php           |   3 +-
 .../Handler/ToolCallHumanInputSuspensionTest.php   |   6 +-
 .../Pipeline/CommandMailboxPolicyTest.php          |   3 +-
 .../Infrastructure/SymfonyAi/LlamaCppSmokeTest.php |   5 +-
 .../Infrastructure/SymfonyAi/TraceReplayTest.php   |   5 +-
 .../Support/PipelineCapturingAgentRunner.php       |   3 +-
 .../Application/Pipeline/CompactRunHandlerTest.php |   8 +-
 tests/CodingAgent/Config/CompactionConfigTest.php  |  34 ---
 tests/CodingAgent/Config/ModelResolverTest.php     |  10 +-
 .../Config/ModelSelectionServiceTest.php           |   5 +-
 .../Config/ModelSettingsPersisterTest.php          |   7 +-
 .../Config/SessionAwareModelResolverTest.php       |   3 +-
 .../ParentPromptUserContextRegressionTest.php      |   3 +-
 .../InProcess/StartRunPersistsSessionModelTest.php |   6 +-
 .../CodingAgent/Runtime/RuntimeEventMapperTest.php |  23 --
 tests/Tui/Listener/ModelCommandHandlerTest.php     |   5 +-
 tests/Tui/Picker/ModelPickerControllerTest.php     |   3 +-
 53 files changed, 63 insertions(+), 1351 deletions(-)
 delete mode 100644 src/AgentCore/Application/Dto/HotPromptStateSnapshot.php
 delete mode 100644 src/AgentCore/Application/Dto/PendingCommandSnapshot.php
 delete mode 100644 src/AgentCore/Application/Dto/RunDebugSnapshot.php
 delete mode 100644 src/AgentCore/Application/Dto/RunStateSnapshot.php
 delete mode 100644 src/AgentCore/Application/Handler/MessageIdempotencyService.php
 delete mode 100644 src/AgentCore/Application/Handler/RunDebugService.php
 delete mode 100644 src/AgentCore/Application/RunReadService.php
 delete mode 100644 src/AgentCore/Domain/Run/RunAccessScope.php
 delete mode 100644 src/AgentCore/Infrastructure/Messenger/BusNames.php
 delete mode 100644 src/CodingAgent/Config/SessionMetadataStore.php
 delete mode 100644 src/CodingAgent/Runtime/Process/AgentProcessSupervisor.php
 delete mode 100644 src/CodingAgent/Runtime/Process/JsonlRuntimeEventSink.php
 delete mode 100644 src/CodingAgent/Session/SkillsProvider.php
 delete mode 100644 src/CodingAgent/Tool/FindTool.php
 delete mode 100644 src/CodingAgent/Tool/GrepTool.php
 delete mode 100644 src/Tui/Application/ThemeFactory.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/remove-audited-dead-code-thin-wrappers.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/remove-audited-dead-code-thin-wrappers.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: gh pr view 324 — state MERGED, merge commit fdcb698000c0cd1aff2ffa6ac8be6d8ba111eec2
- Summary: PR #324 confirmed merged on GitHub at 2026-07-27T16:33:25Z (merge commit fdcb698000c0cd1aff2ffa6ac8be6d8ba111eec2). Proceeding with task workflow merge/sync and worktree cleanup.

## Task workflow update - 2026-07-27T16:39:32.313Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/324
- Updated PR Status: merged
- Validation: PR #324 — MERGED (fdcb698000c0cd1aff2ffa6ac8be6d8ba111eec2); Initial post-merge LLM_MODE=true castor check — FAIL only ShellFollowUpLiveE2eTest; cache guard/artifact integrity/leak checks passed; castor clean:cleanup:workers:list — no stale QA worker candidates; castor test:llm-real --filter=ShellFollowUpLiveE2eTest — PASS (2 tests, 21 assertions); Post-merge LLM_MODE=true castor check retry — PASS (311.0s): test 4522/15739, controller replay 10/135, TUI 37/190, llm-real 13/175, deptrac/phpstan/cs-check green, cache guard/integrity/leak check green; Integration checkout clean; task worktree removed
- Summary: Task completed. PR #324 merged; task branch merged/synced into integration checkout; task worktree and IDEA exclusions removed. Post-merge deterministic gate passed on retry after a transient live ShellFollowUp test failure reproduced green in focused rerun.
