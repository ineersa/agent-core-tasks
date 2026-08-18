# Simplify verified CodingAgent config indirection and dead APIs

## Goal
Deep ponytail audit of `src/CodingAgent/Config` verified these cuts with IDE references plus YAML string-consumer checks. Estimated net reduction: ~220 lines, no dependency changes.

Scope:
- Fold the production-single-caller `ModelSettingsPersister` into `ModelSelectionService`; remove the dedicated class and redundant dedicated test coverage while preserving persistence behavior in `ModelSelectionServiceTest`.
- Remove deprecated, test-only `TuiConfig::fromArray()` and update tests to use the production constructor.
- Remove test-only `SettingsPathResolver::resolveList()` and `getAppRoot()` plus implementation-mirroring tests.
- Remove unreferenced `AppResourceLocator::getAppRoot()`.
- Remove unused `ModelSelectionService::cycleReasoning()` and tests-only `ModelResolver::cycleReasoning()` plus its test block; preserve `ModelResolver::LEVELS` and `cycleReasoningForCurrentModel()`.

Explicit exclusions:
- Do not modify any Subagent or Fork production/test path; another task is active there.
- Do not change model selection, persistence, settings resolution, TUI, or provider behavior.
- Do not pursue rejected audit candidates: keep `HatfieldModelCatalog::config()` because `AppConfig::$ai` and `$catalog` can be independently supplied; keep the two private numeric/env parsers rather than adding a shared helper abstraction.
- Do not touch generated `config/reference.php` or Composer dependencies.

## Acceptance criteria
- `ModelSettingsPersister` is removed and its behavior is owned directly by `ModelSelectionService` with no persistence or validation change.
- Deprecated/test-only and unreferenced methods listed in scope are removed, with callers/tests minimally updated.
- No file whose path or subject is Subagent/Fork is modified.
- Existing rationale comments for persistence, in-memory config synchronization, path resolution, and model-level handling are preserved or updated.
- Focused Castor tests for CodingAgent config/model selection pass; `castor deptrac`, `castor phpstan`, and `castor cs-check` pass.
- Final branch diff is behavior-preserving and removes approximately 200 lines without adding abstractions, settings, APIs, or dependencies.

## Workflow metadata
Status: ARCHIVE
Branch: task/simplify-coding-agent-config-indirection
Worktree: /home/ineersa/projects/agent-core-worktrees/simplify-coding-agent-config-indirection
Fork run: swhytwvd8rro
PR URL: https://github.com/ineersa/agent-core/pull/380
PR Status: merged
Started: 2026-08-13T16:49:57.157Z
Completed: 2026-08-13T17:28:45.584Z

## Work log
- Created: 2026-08-06T14:57:22.257Z

## Task workflow update - 2026-08-13T16:49:57.157Z
- Moved TODO → IN-PROGRESS.
- Created branch task/simplify-coding-agent-config-indirection.
- Created worktree /home/ineersa/projects/agent-core-worktrees/simplify-coding-agent-config-indirection.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/simplify-coding-agent-config-indirection.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/simplify-coding-agent-config-indirection.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/simplify-coding-agent-config-indirection.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/simplify-coding-agent-config-indirection.
- Summary: Starting implementation of verified CodingAgent Config deletions. Finalized scope is behavior-preserving removal/folding only: no Subagent/Fork paths, generated reference config, dependencies, provider/TUI behavior, or rejected audit candidates.

## Task workflow update - 2026-08-13T16:56:24.208Z
- Summary: Scout verified all finalized dead-API cuts against current HEAD and identified exact constructor/test call sites. ModelSettingsPersister has one production owner and no explicit DI registration; fold its three persistence operations into ModelSelectionService while preserving validation, write order, metadata shape, user-layer empty cwd, atomic writer boundary, and AppConfig sync comments. All removed TuiConfig/path/reasoning APIs are test-only or unreferenced after listed updates. No YAML/service/reflection/string consumers found. No TUI behavior proof or llm-real lane is required because behavior and provider-visible flow are unchanged.

## Task workflow update - 2026-08-13T17:04:38.036Z
- Recorded fork run: swhytwvd8rro
- Validation: Focused config/model/provider tests: 146 tests, 409 assertions PASS.; PharSmokeTest: 7 tests, 131 assertions PASS.; castor test: 4408 tests, 16671 assertions PASS.; castor deptrac: 0 violations.; castor phpstan: 0 errors.; castor cs-check: clean.; IDE diagnostics for ModelSelectionService.php: 0 problems.
- Summary: Implementation complete and committed as 098d832d1. Folded ModelSettingsPersister into ModelSelectionService with identical validation/write ordering/session metadata and in-memory AppConfig synchronization, removed deprecated/test-only and unreferenced config APIs, updated constructor/test call sites, and deleted redundant dedicated persister coverage while retaining high-value persistence assertions in ModelSelectionServiceTest. Diff: 23 files, +138/-393, net -255. Exclusion/dead-reference audit clean: no Subagent/Fork paths, generated reference config, Composer, docs/plans/reports, or stale removed-symbol references. Worktree clean.

## Task workflow update - 2026-08-13T17:23:00.394Z
- Validation: Reviewer: APPROVED, no blockers.; castor test: 4408 tests, 16671 assertions PASS.; castor deptrac: 0 violations/errors.; castor phpstan: 0 errors.; castor cs-check: clean.
- Summary: Final reviewer subagent APPROVED at 098d832d1 with no blocking findings. Default reviewer model was rate-limited (429), so the reviewer agent completed using deepseek/deepseek-v4-pro. Review verified persistence-fold equivalence, autowiring/manual constructors, dead-symbol searches, behavioral test coverage, exclusion compliance, and no conflict with current origin/main task-workflow-only changes.

## Task workflow update - 2026-08-13T17:25:20.646Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 240s)...
- castor check passed (125.9s).
- Pushed task/simplify-coding-agent-config-indirection to origin.
- branch 'task/simplify-coding-agent-config-indirection' set up to track 'origin/task/simplify-coding-agent-config-indirection'.
- Created PR: https://github.com/ineersa/agent-core/pull/380
- Validation: Focused config/model/provider tests: 146 tests, 409 assertions PASS.; PharSmokeTest: 7 tests, 131 assertions PASS.; castor test: 4408 tests, 16671 assertions PASS.; castor deptrac: 0 violations/errors.; castor phpstan: 0 errors.; castor cs-check: clean.; Reviewer: APPROVED.
- Summary: Behavior-preserving CodingAgent config cleanup complete at 098d832d1: folded single-caller persister into ModelSelectionService, removed dead/test-only APIs, preserved persistence semantics and model-aware reasoning behavior, net -255 lines. Final reviewer APPROVED.

## Task workflow update - 2026-08-13T17:28:45.584Z
- Moved CODE-REVIEW → DONE.
- Merged task/simplify-coding-agent-config-indirection into integration checkout.
- Merge made by the 'ort' strategy.
 src/CodingAgent/Config/AppResourceLocator.php      |   8 --
 src/CodingAgent/Config/ModelResolver.php           |  16 ---
 src/CodingAgent/Config/ModelSelectionService.php   |  85 +++++++++---
 src/CodingAgent/Config/ModelSettingsPersister.php  |  66 ----------
 src/CodingAgent/Config/SettingsPathResolver.php    |  23 ----
 src/CodingAgent/Config/TuiConfig.php               |  23 ----
 .../Infrastructure/SymfonyAi/LlamaCppSmokeTest.php |   4 +-
 .../Infrastructure/SymfonyAi/TraceReplayTest.php   |   6 +-
 .../Application/Pipeline/CompactRunHandlerTest.php |   9 +-
 .../SnapshotCompactionExtensionHookTest.php        |   7 +-
 tests/CodingAgent/Config/Ai/AiConfigTest.php       |   2 +-
 tests/CodingAgent/Config/ModelResolverTest.php     |  23 +---
 .../Config/ModelSelectionServiceTest.php           |  55 ++++++--
 .../Config/ModelSettingsPersisterTest.php          | 146 ---------------------
 .../Config/SessionAwareModelResolverTest.php       |   5 +-
 .../Config/SettingsPathResolverTest.php            |  21 ---
 .../ContextBudgetReminderHookSubscriberTest.php    |   2 +-
 .../Codex/CodexSymfonyAiProviderBuilderTest.php    |   2 +-
 .../SymfonyAi/SymfonyAiProviderFactoryTest.php     |   6 +-
 .../SymfonyAi/SymfonyAiProviderRegistryTest.php    |   2 +-
 tests/CodingAgent/Phar/PharSmokeTest.php           |   4 +-
 tests/Tui/Listener/ModelCommandHandlerTest.php     |  11 +-
 tests/Tui/Picker/ModelPickerControllerTest.php     |   5 +-
 23 files changed, 138 insertions(+), 393 deletions(-)
 delete mode 100644 src/CodingAgent/Config/ModelSettingsPersister.php
 delete mode 100644 tests/CodingAgent/Config/ModelSettingsPersisterTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/simplify-coding-agent-config-indirection.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/simplify-coding-agent-config-indirection.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #380 state: MERGED at 2026-08-13T17:28:29Z.; Integration checkout clean on main before merge.
- Summary: PR #380 merged on GitHub as 13b24dee8ce727310068e428c39bb4f419fd870c. Moving task to DONE and cleaning task worktree.

## Task workflow update - 2026-08-13T17:31:10.571Z
- Validation: LLM_MODE=true castor check PASS: 4408 unit/integration tests (16671 assertions), 12 controller-replay tests (165 assertions), 36 TUI tests (286 assertions), 13 llm-real tests (144 assertions), deptrac 0 violations, PHPStan 0 errors, cs-check clean.; llama-proxy cache stable 244→244; QA artifact integrity and process/tmux leak checks clean.
- Summary: Post-merge integration validation passed on main after PR #380. Task worktree and IDEA exclusions cleaned.

## Task workflow update - 2026-08-14T19:53:42+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
