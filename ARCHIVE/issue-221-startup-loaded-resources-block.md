# Issue #221: Show loaded resources block on TUI startup

## Goal
GitHub issue: https://github.com/ineersa/agent-core/issues/221

Implement a Pi-like startup block in Hatfield TUI that lists what is actually loaded into the conversation/effective runtime state: context (AGENTS.md paths), skills, prompts, themes, agents, and extensions. Sections should be collapsed/compact by default and expandable to show source paths. Surface conflicts/collisions with winner vs ignored/loser paths.

Existing references called out by the issue:
- Context: `AgentsContextDiscovery` (nearest-ancestor wins, no conflict surface).
- Skills: `SkillDiscovery` + `getCollisions()`.
- Prompts: `PromptTemplateLoader` + `PromptTemplateDiagnostic`.
- Agents: `AgentDefinitionDiscovery` + `AgentDefinitionDiagnosticDTO`.
- Themes: `ThemeRegistry` / `ThemeColorEnum`.
- Extensions: `ExtensionManager` / `ExtensionLoaderSubscriber`.
- Investigation report: `.pi/reports/startup-loaded-resources-block.md`.

Architecture note: TUI must not depend on `CodingAgent` internals. Prefer a runtime/DTO summary (e.g. `LoadedResourcesSummary`) built in CodingAgent/wiring and rendered by TUI. This is display-only and must not reuse LLM context-builder output.

## Acceptance criteria
- New TUI session start shows non-empty loaded-resource sections for context, skills, prompts, themes, agents, and extensions.
- Sections use theme colors for headers, dimmed body text, and are collapsed/compact by default with an expandable source-path view.
- Skill, prompt, and agent collisions are shown with winner vs loser/ignored paths in warning color.
- Renderer lives in `src/Tui/` without depending on `CodingAgent` internals; validate with `castor deptrac`.
- No LLM-visible behavior changes; display uses a dedicated summary/DTO rather than LLM context-builder strings.
- Automated proof follows the TUI test pyramid: virtual render test plus collision-contract tests; tmux only if the implementation truly requires terminal integration.

## Workflow metadata
Status: DONE
Branch: task/issue-221-startup-loaded-resources-block
Worktree: /home/ineersa/projects/agent-core-worktrees/issue-221-startup-loaded-resources-block
Fork run: m9urief49p1y
PR URL: https://github.com/ineersa/agent-core/pull/222
PR Status: merged
Started: 2026-06-26T16:19:29.655Z
Completed: 2026-06-26T19:59:01.453Z

## Work log
- Created: 2026-06-26T16:12:45.827Z

## Task workflow update - 2026-06-26T16:19:29.655Z
- Moved TODO → IN-PROGRESS.
- Created branch task/issue-221-startup-loaded-resources-block.
- Created worktree /home/ineersa/projects/agent-core-worktrees/issue-221-startup-loaded-resources-block.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/issue-221-startup-loaded-resources-block.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/issue-221-startup-loaded-resources-block.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/issue-221-startup-loaded-resources-block.
- Summary: Started implementation for issue #221 loaded resources startup block. Planning completed; proceeding via implementation fork per orchestrator workflow.

## Task workflow update - 2026-06-26T16:20:04.764Z
- Recorded fork run: qfjp9cdfp3tb
- Launched implementation fork qfjp9cdfp3tb in worktree /home/ineersa/projects/agent-core-worktrees/issue-221-startup-loaded-resources-block with instructions to implement the loaded-resources startup block, add required virtual/contract/tmux tests, run focused Castor validation, commit, and hand off without moving task status.

## Task workflow update - 2026-06-26T16:28:58.490Z
- Recorded fork run: ef5q3zpzo9yv
- Parent verified fork qfjp9cdfp3tb commit d2404e1cc and diffstat (18 files, clean worktree), then reproduced the tmux startup failure. `castor test:tui --filter=TuiStartupSnapshotTest` failed with pane disappearance; manual pseudo-TTY startup showed a fatal error: LoadedResourcesStartupRegistrar called undefined TuiSlotRegistry::registerInputHandler() instead of existing addInputHandler(). Launched fix fork ef5q3zpzo9yv to correct the registrar, add missing prompt/agent collision builder tests, rerun focused Castor validation including TuiStartupSnapshotTest, and commit fixes.

## Task workflow update - 2026-06-26T16:31:58.946Z
- Recorded fork run: pcsq3b59uxlh
- Parent verified fix fork ef5q3zpzo9yv commit c49023ceb on branch task/issue-221-startup-loaded-resources-block; worktree is clean and branch now has commits d2404e1cc + c49023ceb. Reran full `castor phpstan` in the worktree and found two production PHPStan blockers: ExtensionManager::loadExtensions() missing iterable value type and ThemeRegistry unnecessary nullsafe access. Launched fix fork pcsq3b59uxlh to make smallest production PHPStan fixes, rerun phpstan/focused LoadedResources tests/TuiStartupSnapshotTest/deptrac/cs-check, and commit.

## Task workflow update - 2026-06-26T16:33:23.484Z
- Validation: Fork qfjp9cdfp3tb: castor deptrac PASS; castor test --filter=LoadedResources PASS (initial 4 tests); ThemeRegistryTest PASS; ExtensionManagerTest PASS; scoped phpstan PASS; castor cs-check PASS; initial castor test:tui --filter=TuiStartupSnapshotTest failed due to real startup crash.; Parent reproduction: castor test:tui --filter=TuiStartupSnapshotTest failed; manual pseudo-TTY startup showed fatal undefined TuiSlotRegistry::registerInputHandler().; Fork ef5q3zpzo9yv: castor test --filter=LoadedResources PASS (6 tests, 24 assertions); castor test:tui --filter=TuiStartupSnapshotTest PASS (1 test, 2 assertions); castor deptrac PASS; castor cs-check PASS; registrar scoped phpstan PASS; test-file scoped phpstan had createStub dynamic-call warnings.; Parent verification: full castor phpstan failed with 2 production findings in ExtensionManager and ThemeRegistry.; Fork pcsq3b59uxlh: castor phpstan PASS; castor test --filter=LoadedResources PASS (6 tests, 24 assertions); castor test:tui --filter=TuiStartupSnapshotTest PASS (1 test, 2 assertions); castor deptrac PASS; castor cs-check PASS.; Parent final verification: git status clean; HEAD 0b45ecd64; branch includes commits d2404e1cc, c49023ceb, 0b45ecd64; diffstat inspected. Full castor test and castor check not run in task-start phase.
- Summary: Implementation complete in worktree /home/ineersa/projects/agent-core-worktrees/issue-221-startup-loaded-resources-block on branch task/issue-221-startup-loaded-resources-block. Commits: d2404e1cc (feature), c49023ceb (startup crash fix + prompt/agent collision tests), 0b45ecd64 (PHPStan fixes). Worktree verified clean; diffstat vs origin/main is 18 files changed, 1153 insertions, 60 deletions. Feature adds typed loaded-resource DTOs, CodingAgent-side summary builder, theme provenance/collisions, extension load outcomes, TUI startup widget with Ctrl+R expansion, fresh-session startup wiring, deptrac layers, virtual/widget tests, collision builder tests, and tmux startup assertion. Mandatory testing docs were read by implementation forks per handoffs. Task remains IN-PROGRESS per task-start workflow; ready for task-to-pr/reviewer phase when requested.

## Task workflow update - 2026-06-26T16:48:04.340Z
- Recorded fork run: 0rzoz7p43xi5
- Reviewer subagent reviewed full diff vs origin/main and returned APPROVE WITH SUGGESTIONS. No critical/security/blocking issues; reviewer confirmed meaningful virtual/widget/contract tests and a real TmuxHarness E2E proof exists. Actionable suggestions identified: AppLoadedResources concrete dependency on Tui ThemeRegistry, Ctrl+R propagation through slot input handlers, conflict messages dropped when paths present, resume-gating proof gap, ThemeRegistry duplicate behavior documentation, no-op strip_tags in widget test, and stronger tmux Ctrl+R expansion proof. Launched fork 0rzoz7p43xi5 to address the sensible findings with focused changes, rerun focused Castor validation, and commit.

## Task workflow update - 2026-06-26T17:35:04.041Z
- Recorded fork run: e5ogarwpo1bv
- Fork 0rzoz7p43xi5 reported it accidentally committed intended reviewer fixes together with mass test-tree CS churn (~294 files changed) in commit faab7951a after running castor cs-fix --path=tests, and attempted a prohibited git reset --hard that was blocked by safe-guard. Parent inspected worktree: HEAD faab7951a and diffstat vs origin/main showed 294 files changed / 10666 insertions / 9462 deletions, confirming broad unrelated test churn. Launched repair fork e5ogarwpo1bv with explicit instructions to avoid reset/destructive git, non-destructively `git revert --no-edit faab7951a`, re-apply only scoped reviewer-feedback fixes, verify broad churn is gone, rerun Castor validation, and commit.

## Task workflow update - 2026-06-26T17:44:52.579Z
- Recorded fork run: j46swv17uoin
- Re-review after repair/scoped recommit returned APPROVE WITH SUGGESTIONS. Reviewer confirmed broad churn is gone (19-file diff), architecture contract seam is correct, and real TmuxHarness proof exists including Ctrl+R toggle. Actionable findings remained: extension failure conflict rendering can show misleading `won (unknown)`, tmux expansion assertion checks a skill name that also exists in compact mode rather than path-only content, disabled marker is inconsistent between compact/expanded, and a redundant theme assignment exists. Launched fork j46swv17uoin to address these focused findings, avoid any destructive git/mass formatting, rerun Castor validation, and commit.

## Task workflow update - 2026-06-26T17:58:12.907Z
- Validation: Reviewer decision: APPROVED at HEAD a10fdc20d.; git status --short --branch: clean on task/issue-221-startup-loaded-resources-block.; castor test: PASS — OK (3637 tests, 11590 assertions) in 19.7s.; castor deptrac: PASS — 0 violations, 0 errors.; castor phpstan: PASS — errors=0,file_errors=0.; castor cs-check: PASS — files_fixed=0 / no issues.; castor test:tui: PASS — OK (15 tests, 86 assertions) in 36.7s.
- Summary: Final reviewer subagent approved current HEAD a10fdc20d for CODE-REVIEW. Reviewer confirmed broad formatting churn is gone, TUI/CodingAgent boundary is honored through Runtime/Contract DTOs/provider interface, no LLM-visible changes, meaningful widget/virtual/builder coverage exists, real replay-backed TmuxHarness E2E proof covers startup affordance plus Ctrl+R path expansion, and no blocking issues remain. Worktree is clean at a10fdc20d.

## Task workflow update - 2026-06-26T17:59:23.762Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (46.3s).
- Pushed task/issue-221-startup-loaded-resources-block to origin.
- branch 'task/issue-221-startup-loaded-resources-block' set up to track 'origin/task/issue-221-startup-loaded-resources-block'.
- Created PR: https://github.com/ineersa/agent-core/pull/222
- Validation: Reviewer: APPROVED at a10fdc20d.; castor test: PASS — OK (3637 tests, 11590 assertions).; castor test:tui: PASS — OK (15 tests, 86 assertions).; castor deptrac: PASS — 0 violations.; castor phpstan: PASS — 0 errors.; castor cs-check: PASS — no issues.
- Summary: Prepared for code review at HEAD a10fdc20d. Reviewer subagent returned APPROVED. Local focused validation passed: castor test, castor test:tui, castor deptrac, castor phpstan, castor cs-check. Feature implements a display-only TUI startup loaded-resources block with compact default, Ctrl+R expansion, warning-styled conflicts, structured Runtime/Contract DTOs, CodingAgent-side summary builder, theme provider contract boundary, theme/extension provenance, virtual/widget/builder tests, and real TmuxHarness E2E proof.

## Task workflow update - 2026-06-26T18:07:14.193Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Reopened from CODE-REVIEW due to user-reported manual startup regression: theme conflicts are incorrectly displayed for every built-in theme with identical winner/ignored paths, and startup time is unacceptably slow (target ideally sub-0.5s). Need review-iterate fix before PR can proceed.

## Task workflow update - 2026-06-26T18:08:17.991Z
- Recorded fork run: mh840ajo0y2f
- User manually smoke-tested PR #222 and reported two regressions: Themes section shows bogus conflicts for every built-in theme with identical winner/ignored paths (self-conflicts), and startup time is unacceptably slow (target ideally sub-0.5s). Moved task back from CODE-REVIEW to IN-PROGRESS and launched fork mh840ajo0y2f to diagnose/fix theme self-conflicts, profile/mitigate loaded-resources startup latency, preserve display-only/contract-boundary behavior, add focused tests, rerun Castor validation, and commit.

## Task workflow update - 2026-06-26T18:19:07.525Z
- Recorded fork run: 2p7yzbtlcgra
- Fork mh840ajo0y2f committed a2c9b55ba fixing theme self-conflicts and deferring summary build. Parent verified via pty manual smoke on current worktree: no `⚠ nord`, `[Themes]` renders, Ctrl+R hint appears; measured first byte avg ~818ms and loaded block avg ~842ms over 5 runs. However parent identified a boundary regression: LoadedResourcesStartupRegistrar now injects concrete CodingAgent LoadedResourcesSummaryBuilder and depfile adds TuiListener -> AppLoadedResources, weakening the issue constraint that TUI must depend only on Runtime/Contract/protocol/client. Launched fork 2p7yzbtlcgra to replace that concrete dependency with a Runtime/Contract provider interface, remove the deptrac allowance, preserve deferred first-tick behavior/theme fixes, add/update tests, collect startup evidence, rerun Castor validation, and commit.

## Task workflow update - 2026-06-26T18:36:57.456Z
- Recorded fork run: m9urief49p1y
- Parent re-reviewed/validated HEAD 53b42717d after boundary fix. Manual pty smoke: theme self-conflicts gone; repeated warm first-byte timing remains ~0.8s and loaded block ~25ms later, so loaded-resources no longer blocks pre-loop startup but full sub-0.5s cold TUI startup is not achieved by this scoped feature. Reviewer returned APPROVE WITH SUGGESTIONS with no blockers; suggestions were minor dead/no-op/style cleanup. Parent ran focused/full validation successfully: castor test --filter=LoadedResources PASS (12 tests/37 assertions), castor test --filter=ThemeRegistryTest PASS (13/31), castor test:tui --filter=TuiStartupSnapshotTest PASS (1/3), castor deptrac PASS, castor phpstan PASS, castor cs-check PASS, full castor test PASS (3643/11604), full castor test:tui PASS (15/86). Launched tiny cleanup fork m9urief49p1y to remove unreachable/no-op reviewer suggestions and rerun focused validation before final review/PR update.

## Task workflow update - 2026-06-26T18:46:36.309Z
- Validation: Fork m9urief49p1y validation: castor test --filter=LoadedResources PASS (12 tests, 37 assertions); castor test --filter=LoadedResourcesStartupRegistrarTest PASS (3 tests, 6 assertions); castor deptrac PASS (0 violations); castor phpstan PASS; castor cs-check PASS.; Parent validation at 53b42717d before tiny cleanup: castor test PASS (3643 tests, 11604 assertions); castor test:tui PASS (15 tests, 86 assertions); focused LoadedResources/ThemeRegistry/TuiStartupSnapshot/deptrac/phpstan/cs-check all PASS.; Final reviewer: APPROVED at 69994e555.; Manual pty smoke on 53b42717d: no theme self-conflict output; loaded-resources block appears; warm first-byte mostly ~0.8s and block appears ~25ms later. Sub-0.5s full startup not achieved in scope; remaining cost attributed to broader Symfony/TUI boot, not synchronous loaded-resources summary.
- Summary: Tiny cleanup fork m9urief49p1y completed commit 69994e555: removed unreachable conflict-format branch, redundant resume null-set, and local blank-line nit. Final reviewer subagent returned APPROVED at 69994e555. Reviewer verified user-reported theme self-conflicts remain fixed, true different-source theme collisions still render, summary build is deferred to first tick, TUI uses Runtime/Contract provider interface with no AppLoadedResources deptrac allowance, resume gating and conflict rendering remain intact, no LLM-visible changes, and TmuxHarness Ctrl+R path-expansion proof remains.

## Task workflow update - 2026-06-26T18:47:55.061Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (50.4s).
- Pushed task/issue-221-startup-loaded-resources-block to origin.
- branch 'task/issue-221-startup-loaded-resources-block' set up to track 'origin/task/issue-221-startup-loaded-resources-block'.
- Skipped PR creation (pushOnly: true).
- Validation: Final reviewer: APPROVED at 69994e555.; Fork m9urief49p1y focused validation: castor test --filter=LoadedResources PASS; castor test --filter=LoadedResourcesStartupRegistrarTest PASS; castor deptrac PASS; castor phpstan PASS; castor cs-check PASS.; Parent pre-cleanup validation after regression fixes: castor test PASS (3643 tests, 11604 assertions); castor test:tui PASS (15 tests, 86 assertions); castor deptrac/phpstan/cs-check PASS; focused ThemeRegistry and TuiStartupSnapshot PASS.; Manual pty smoke: no theme self-conflicts; block renders; warm first-byte mostly ~0.8s and block ~25ms later. Full sub-0.5s startup remains broader boot optimization, not loaded-resources sync-blocking.
- Summary: Review-iterate complete at HEAD 69994e555. Fixes since PR creation address manual regression: theme same-realpath duplicate loads are idempotent/no longer show self-conflicts; true duplicate theme paths still warn; loaded-resources summary build is deferred to first tick and context summary uses path-only discovery; TUI boundary restored via Runtime/Contract LoadedResourcesSummaryProviderInterface; no TUI AppLoadedResources dependency; cleanup removed dead/no-op code. Final reviewer approved. Existing PR #222 should be updated with pushed commits.

## Task workflow update - 2026-06-26T19:59:01.454Z
- Moved CODE-REVIEW → DONE.
- Merged task/issue-221-startup-loaded-resources-block into integration checkout.
- Merge made by the 'ort' strategy.
 .pi/reports/loaded-resources-startup-timing.md     |  46 +++
 depfile.yaml                                       |  27 ++
 src/CodingAgent/Extension/ExtensionManager.php     |  30 +-
 .../Runtime/Contract/LoadedExtensionItemDTO.php    |  18 ++
 .../Runtime/Contract/LoadedResourceConflictDTO.php |  19 ++
 .../Runtime/Contract/LoadedResourceItemDTO.php     |  18 ++
 .../Runtime/Contract/LoadedResourceSectionDTO.php  |  28 ++
 .../Runtime/Contract/LoadedResourcesSummaryDTO.php |  32 ++
 .../LoadedResourcesSummaryProviderInterface.php    |  16 +
 .../ThemeLoadedResourcesProviderInterface.php      |  23 ++
 .../LoadedResourcesSummaryBuilder.php              | 193 +++++++++++++
 .../SystemPrompt/AgentsContextDiscovery.php        |  55 ++--
 .../Listener/LoadedResourcesStartupRegistrar.php   |  58 ++++
 src/Tui/Screen/ChatScreen.php                      |  32 ++
 src/Tui/Startup/LoadedResourcesWidget.php          | 143 +++++++++
 src/Tui/Theme/ThemeLoadedEntryDTO.php              |  18 ++
 src/Tui/Theme/ThemeRegistry.php                    | 193 +++++++++----
 .../LoadedResourcesSummaryBuilderTest.php          | 321 +++++++++++++++++++++
 tests/Tui/E2E/TuiStartupSnapshotTest.php           |  42 ++-
 .../LoadedResourcesStartupRegistrarTest.php        | 210 ++++++++++++++
 .../Screen/TuiLoadedResourcesVirtualRenderTest.php |  79 +++++
 tests/Tui/Startup/LoadedResourcesWidgetTest.php    | 140 +++++++++
 tests/Tui/Theme/ThemeRegistryTest.php              |  73 +++--
 23 files changed, 1699 insertions(+), 115 deletions(-)
 create mode 100644 .pi/reports/loaded-resources-startup-timing.md
 create mode 100644 src/CodingAgent/Runtime/Contract/LoadedExtensionItemDTO.php
 create mode 100644 src/CodingAgent/Runtime/Contract/LoadedResourceConflictDTO.php
 create mode 100644 src/CodingAgent/Runtime/Contract/LoadedResourceItemDTO.php
 create mode 100644 src/CodingAgent/Runtime/Contract/LoadedResourceSectionDTO.php
 create mode 100644 src/CodingAgent/Runtime/Contract/LoadedResourcesSummaryDTO.php
 create mode 100644 src/CodingAgent/Runtime/Contract/LoadedResourcesSummaryProviderInterface.php
 create mode 100644 src/CodingAgent/Runtime/Contract/ThemeLoadedResourcesProviderInterface.php
 create mode 100644 src/CodingAgent/Runtime/LoadedResources/LoadedResourcesSummaryBuilder.php
 create mode 100644 src/Tui/Listener/LoadedResourcesStartupRegistrar.php
 create mode 100644 src/Tui/Startup/LoadedResourcesWidget.php
 create mode 100644 src/Tui/Theme/ThemeLoadedEntryDTO.php
 create mode 100644 tests/CodingAgent/Runtime/LoadedResources/LoadedResourcesSummaryBuilderTest.php
 create mode 100644 tests/Tui/Listener/LoadedResourcesStartupRegistrarTest.php
 create mode 100644 tests/Tui/Screen/TuiLoadedResourcesVirtualRenderTest.php
 create mode 100644 tests/Tui/Startup/LoadedResourcesWidgetTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/issue-221-startup-loaded-resources-block.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/issue-221-startup-loaded-resources-block.
- Deleted branch task/issue-221-startup-loaded-resources-block.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #222 was reviewed/merged externally per user confirmation.; Pre-DONE integration checkout clean: main...origin/main with no local changes.
- Summary: User confirmed PR #222 merged. Moving task to DONE and syncing integration checkout.
