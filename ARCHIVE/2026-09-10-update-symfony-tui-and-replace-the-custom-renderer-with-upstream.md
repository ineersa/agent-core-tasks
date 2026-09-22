# Update Symfony TUI and replace the custom renderer with upstream

## Goal
Follow-up to https://github.com/ineersa/agent-core/issues/467 and archived task 2026-09-05-investigate-large-resumed-session-tui-input-and-streaming-lag.

User approved updating Symfony TUI, swapping to the original upstream renderer, and testing the result. Current composer.json requires 8.1.x-dev; composer.lock pins 94c5b0ecce2c5ce41ab9460b84f48b2ab1366baf. A development-branch constraint does not automatically advance the installed revision.

Current upstream 8.2 checkout uses LineBufferInterface and cached maximum visible widths, which may supersede CachedWidthValidationRenderer's per-widget WeakMap of validated rows. First determine whether these changes are available on 8.1; do not assume updating the existing constraint will install 8.2 changes. Keep dependency changes limited to TUI and required dependencies. If a branch change is required, document it before proceeding.

Remove CachedWidthValidationRenderer, its alias installer, wiring, drift guard, dead-code exceptions, and obsolete copy-specific tests once upstream behavior is verified. Preserve relevant behavior coverage using the upstream renderer. Adapt callers to upstream line-buffer APIs without compatibility shims.

Preserve the ScreenWriter cursor-hiding, synchronized restoration, and deferred cursor workaround unless the selected upstream revision includes the complete replacement. PR https://github.com/symfony/symfony/pull/65966 is separate. Updating TUI may require adapting our writer to writeFrame/LineBufferInterface; account for this explicitly.

Measure cached large-transcript prompt edits before and after at identical geometry, including width validation, render latency, and retained memory. Historical downstream evidence: about 18,099 logical rows at 224x60; render p50 66.029ms to 2.746ms with local cache. Do not compare fresh timings directly with historical instrumented numbers as controlled proof.

Use normal task worktree workflow. No PR or merge until requested.

## Acceptance criteria
- Record old and new Composer versions/revisions and confirm whether 8.1 contains the needed upstream optimization.
- Use upstream Renderer without the custom renderer alias; remove superseded implementation and registration artifacts.
- Retain cursor workaround behavior and adapt to new TUI APIs where necessary.
- Add or retain minimal deterministic behavior tests for rendering, changed rows, resize, and over-wide output.
- Run targeted Castor checks and a controlled large-transcript comparison; report correctness, performance, and memory results without claiming emulator proof from unit tests.
- Provide a build for manual prompt/streaming/resize validation. Record remaining blockers; full castor check belongs to CODE-REVIEW transition.
- Update issue #467 with evidence; close it only when its upstream optimization/removal goal is satisfied.

## Workflow metadata
Status: DONE
Branch: task/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/491
PR Status: merged
Started: 2026-09-10T00:12:39+00:00
Completed: 2026-09-10T23:50:37+00:00

## Work log
- Created: 2026-09-10T00:10:56+00:00

## Task workflow update - 2026-09-10T00:11:51+00:00
- Summary: User clarified target branch: 8.2. Verified GitHub default branch is 8.2 for both symfony/symfony and symfony/tui. Target composer constraint is 8.2.x-dev, replacing 8.1.x-dev; record resolved lock revision during implementation. This supersedes the earlier instruction to first seek the optimization on 8.1 or leave the branch choice unresolved.

## Task workflow update - 2026-09-10T00:12:39+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream/.idea.
- JetBrains project open degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream: HTTP request failed: Amp\Internal\FutureIterator::consume(): Return value must be of type ?array, Mcp\Schema\JsonRpc\Error returned. Filesystem tools remain available.

## Task workflow update - 2026-09-10T00:37:44+00:00
- Validation: Final castor test --suite=tui: 1216 tests, 5342 assertions, exit 0.; Final focused picker/writer/upstream renderer: 25 tests, 117 assertions passed.; Final castor cs-check exit 0; castor phar:build passed.; Earlier migration phpstan TUI, dead-code, deptrac passed; focused writer PHPStan passed after shrink refinement.; Full castor check not run; CODE-REVIEW transition owns gate.
- Summary: Implemented migration in task worktree, uncommitted. Upgraded only symfony/tui to 8.2.x-dev@11815c044d4f43a7773719e98d6901e9a7976a9f; removed custom renderer/alias and registration/dead-code remnants. Rebased retained cursor workaround onto LineBuffer APIs. BC audit addressed renderFrame/renderWidgetLines/cache types, shared dispatcher test setup, persistent detach listeners in SetupScreen, and SelectList constructor named arguments. Preserved #477 picker no-clear/no-transcript-replay assertions; prefix-aware overheight shrink gate in local writer required to retain behavior. Synthetic 2000-row 224x60 comparison: custom p50 2.80ms/p95 3.03ms, upstream p50 4.04ms/p95 4.14ms; cold 8.28ms→5.47ms. This is not proof of historical 18k-row workload or memory equivalence. Large-session manual trial and retained-memory comparison remain pending. PHAR rebuilt. No commits/push/PR. Fork posted migration evidence on issue #467, left open.
- Ownership: owner=fork; fork_run=agent_30e7882692f3303f; revision=task worktree baseline; scope=TUI 8.2 migration and BC validation; outcome=completed; commit=none

## Task workflow update - 2026-09-10T22:58:59+00:00
- Validation: TUI suite: 1216 tests / 5342 assertions passed; Focused final: 54 tests / 309 assertions passed; PHPStan, Deptrac, dead-code, cs-check, docs validation passed
- Summary: Final scope confirmed: upstream renderer, retained custom ScreenWriter. Removed temporary instrumentation from main and worktree; ignored measurement evidence preserved. Manual memory comparison was cold-vs-warm confounded; cold startup allocations matched. Reviewer agent_71dd065de424a274 APPROVE WITH SUGGESTIONS on dirty f6f85d652; comment-only suggestions applied. No unresolved blockers.
- Ownership: owner=fork; fork_run=agent_f144e198585fb0b2; revision=f6f85d652; scope=migration telemetry cleanup and focused validation; outcome=completed; commit=none
- Reviewer: agent_71dd065de424a274; target=dirty migration against f6f85d652; scope=specification fidelity and full migration; verdict=APPROVE WITH SUGGESTIONS

## Task workflow update - 2026-09-10T23:00:24+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream.md.
- Session/run: 30.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream/var/reports/qa-20260910-225918-5264-f7b05bd6.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-10T23:23:21+00:00
- Validation: Focused autoload/Footer/resources:16 tests142 assertions passed; Independent autoload proof2 tests6 assertions passed
- Summary: Gate failure root cause fixed: extensions autoloader prepended old TUI8.1 ahead of host8.2. Config now appends; generated loader verified. Reviewer agent_71dd065de424a274 approved fix with suggestions. Commit44a24db47; focused16/142 and reviewer2/6 pass. Retrying mandatory gate.

## Task workflow update - 2026-09-10T23:24:22+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream.md.
- Session/run: 30.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream/var/reports/qa-20260910-232331-11898-51753637.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-10T23:26:24+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (84.1s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream/var/reports/qa-20260910-232533-16693-9d021887.
- Session/run: 30.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-10T23:26:26+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream/var/reports/qa-20260910-232533-16693-9d021887.
- Session/run: 30.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-10T23:26:29+00:00
- castor check passed (84.1s).
- Pushed task/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream to origin.
- Created PR: <url>
- Session/run: 30.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-10T23:26:29+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (84.1s).
- Pushed task/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/491
- Summary: CS-only helper ordering fixed with Castor at fe2f2cf74. Behavior unchanged from approved autoload correction.

## Task workflow update - 2026-09-10T23:50:37+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream: ide_close_project returned isError.
- Merged task/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream into integration checkout.
- Auto-merging tests/Tui/Listener/PreviewExpansionInputListenerTest.php
Merge made by the 'ort' strategy.
 .hatfield/extensions/composer.json                                |   3 +-
 composer.json                                                     |   2 +-
 composer.lock                                                     |  15 ++++---
 docs/settings-agents.md                                           |   3 ++
 docs/tui-architecture.md                                          |   4 ++
 src/CodingAgent/Extension/ExtensionManager.php                    |   7 +--
 src/Tui/Application/InteractiveMode.php                           |   2 -
 src/Tui/Setup/SetupScreen.php                                     |  16 ++++---
 src/Tui/Terminal/CachedWidthValidationRenderer.php                | 372 ------------------------------------------------------------------------------------------------------------------------------------------------------
 src/Tui/Terminal/CachedWidthValidationRendererAliasInstaller.php  |  38 ----------------
 src/Tui/Terminal/SynchronizedCursorScreenWriter.php               | 309 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++--------------------------------------------------------------
 src/Tui/Terminal/SynchronizedCursorScreenWriterAliasInstaller.php |   2 +-
 tests/CodingAgent/Extension/ExtensionManagerTest.php              | 102 +++++++++++++++++++++++++++++++++++++++++
 tests/Tui/CompactHeader/CompactHeaderWidgetTest.php               |   2 +-
 tests/Tui/Footer/FooterBarWidgetTest.php                          |   2 +-
 tests/Tui/Listener/PreviewExpansionInputListenerTest.php          |   2 +-
 tests/Tui/Picker/PickerOverlayTest.php                            |   5 ++-
 tests/Tui/Question/QuestionControllerTest.php                     |   2 +-
 tests/Tui/Screen/ChatScreenTest.php                               |   5 ++-
 tests/Tui/Startup/LoadedResourcesWidgetTest.php                   |   2 +-
 tests/Tui/Terminal/CachedWidthValidationRendererTest.php          | 103 ------------------------------------------
 tests/Tui/Terminal/SynchronizedCursorScreenWriterTest.php         |  57 +++++++++++++++++++----
 tests/Tui/Terminal/UpstreamRendererBehaviorTest.php               |  60 +++++++++++++++++++++++++
 tests/Tui/Transcript/HotkeyTableWidgetTest.php                    |   2 +-
 tests/Tui/Transcript/SubagentResultRendererTest.php               |   2 +-
 tools/phpstan/DeadCode/HatfieldDeadCodeUsageProvider.php          |   5 ---
 26 files changed, 412 insertions(+), 712 deletions(-)
 delete mode 100644 src/Tui/Terminal/CachedWidthValidationRenderer.php
 delete mode 100644 src/Tui/Terminal/CachedWidthValidationRendererAliasInstaller.php
 delete mode 100644 tests/Tui/Terminal/CachedWidthValidationRendererTest.php
 create mode 100644 tests/Tui/Terminal/UpstreamRendererBehaviorTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-10-update-symfony-tui-and-replace-the-custom-renderer-with-upstream.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: Confirmed PR491 merged on GitHub at3b6f4d41fbff5fd8331fd9e6d22840c19e1a43ed. User requested completion.

## Task workflow update - 2026-09-10T23:53:06+00:00
- Validation: Post-merge castor check qa-20260910-235044-21724-ed1f8c65 passed all10 lanes including4941 tests/20757 assertions, controller replay11/175, TUI9/64, live5/30. Artifact integrity/leak/cache checks passed.
- Summary: Integration checkout clean; dependencies installed and extension autoload regenerated; worktree removed. Post-merge validation complete.
