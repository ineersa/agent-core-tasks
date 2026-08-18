# tui-01: Close the TUI architecture gap and DRY session UI helpers

## Goal
## Goal
Combine the report's immediate TUI governance fix with its mechanical DRY/dead-surface cleanup. No user-visible behavior changes.

## Architecture report evidence

### Missing ImagePaste layer (candidate 1)
`depfile.yaml` references `TuiImagePaste` in the ruleset but does not define its collector. The 11 files under `src/Tui/ImagePaste/` are therefore reported as uncovered despite owning the OS boundary (`Symfony Process`, clipboard commands, temporary files). The intended ruleset already permits `AppConfig`, `AppSession`, `TuiRuntime`, `TuiScreen`, `TuiTranscript`, and `SymfonyProcess`.

The architect ran raw `vendor/bin/deptrac`, contrary to repository policy. Treat that result as non-authoritative: reproduce through `castor deptrac` before changing anything. The expected minimal fix is the four-line directory collector for `src/Tui/ImagePaste/.*`.

### DRY batch (candidate 5)
- Identical model-selection/footer synchronization appears in `ModelControlListener`, `ModelPickerController`, and `ModelCommandHandler`; `FooterStateInitializer` already owns initialization and is the natural reuse point.
- Subagent live status → activity mapping is repeated in three `TickPollListener` branches.
- Identical SelectList keybindings and `maxVisible: 10` setup appears in `QuestionController`, five picker controllers, and `CompletionMenu`.
- `SubagentLiveChildViewPoller` duplicates `RuntimeEventPoller`'s nullable callbacks, dispatch, callback invocation, and runtime-event extraction.

### Small verified cleanup from the report
Audit and remove only if still unused/redundant:
- dead `QuestionController::setRuntimeRefs()` context argument,
- stale by-reference state captures in `CtrlCInputInterceptor` and `LoadedResourcesStartupRegistrar`,
- per-render sorting in `TuiSlotRegistry::getWidgetsByPlacement()` when ordering can be maintained on mutation,
- unused/test-shaped `TuiRenderContext::terminalHeight` and production constructor defaults, subject to extension/API reference proof.

## Smallest viable direction
- Add the missing Deptrac collector.
- Put each duplicated policy in the narrow existing owner: footer initializer, status DTO/enum mapper, one standard SelectList keybinding provider, and one callback dispatcher.
- Do not create a generic UI utilities package or broad event framework.
- Delete dead surface only after semantic reference checks; do not reshape production APIs solely for cleaner tests.

## Scope boundaries
- No picker, question, completion, transcript, extension, or runtime behavior changes.
- Preserve key actions, priorities, visible row count, footer values, status transitions, callback order/failure behavior, and slot ordering.
- Do not start the per-session scope redesign in this task.

## Acceptance criteria
- `TuiImagePaste` is defined as a Deptrac layer and all ImagePaste production files are governed by the intended existing ruleset.
- Model-selection footer synchronization, subagent status mapping, SelectList keybinding setup, and runtime-event callback dispatch each have one narrow owner.
- All call sites preserve current values, callback ordering, errors, key actions, and picker/question behavior.
- Confirmed dead/test-shaped surfaces are deleted or recorded as KEEP with exact references; no public ExtensionApi contract is changed.
- No speculative utility framework, event bus, setting, or compatibility shim is introduced.
- Virtual/in-process tests cover the shared behavior at the lowest correct layer without duplicating framework tests.
- The fork and handoff state that `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` were read and followed.
- Focused `castor test`, `castor deptrac`, `castor phpstan`, `castor cs-check`, `castor test:tui`, and final `castor check` pass.

## Workflow metadata
Status: ARCHIVE
Branch: task/tui-01-close-architecture-gap-and-dry-session-ui-helpers
Worktree: /home/ineersa/projects/agent-core-worktrees/tui-01-close-architecture-gap-and-dry-session-ui-helpers
Fork run: ww28sm4ag6sh
PR URL: https://github.com/ineersa/agent-core/pull/391
PR Status: merged
Started: 2026-08-15T23:52:42.213Z
Completed: 2026-08-16T02:44:59.467Z

## Work log
- Created: 2026-08-15T23:16:16.558Z

## Task workflow update - 2026-08-15T23:19:03.254Z
- Full architect report: [`/home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md`](file:///home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md) — TUI report, candidates 1 and 5.

## Task workflow update - 2026-08-15T23:52:42.213Z
- Moved TODO → IN-PROGRESS.
- Created branch task/tui-01-close-architecture-gap-and-dry-session-ui-helpers.
- Created worktree /home/ineersa/projects/agent-core-worktrees/tui-01-close-architecture-gap-and-dry-session-ui-helpers.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/tui-01-close-architecture-gap-and-dry-session-ui-helpers.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/tui-01-close-architecture-gap-and-dry-session-ui-helpers.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/tui-01-close-architecture-gap-and-dry-session-ui-helpers.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/tui-01-close-architecture-gap-and-dry-session-ui-helpers.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/tui-01-close-architecture-gap-and-dry-session-ui-helpers/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/tui-01-close-architecture-gap-and-dry-session-ui-helpers.
- Summary: Claimed for implementation. Main agent will orchestrate only; a fork will implement in the task worktree. TUI proof must include a replay-backed real TmuxHarness E2E path per task-start request.

## Task workflow update - 2026-08-16T00:00:01.019Z
- Recorded fork run: zhxrgwft2sm0
- Summary: Launched implementation fork in `/home/ineersa/projects/agent-core-worktrees/tui-01-close-architecture-gap-and-dry-session-ui-helpers`. Scope includes missing TuiImagePaste collector, four narrow DRY owners, reference-proven cleanup, virtual behavior coverage, and user-required replay-backed real TmuxHarness completion proof. Fork instructed to run focused Castor lanes except `castor check`, then commit.

## Task workflow update - 2026-08-16T00:19:50.332Z
- Recorded fork run: zhxrgwft2sm0
- Validation: Baseline pre-change `castor deptrac`: PASS (0 violations; uncovered 2250).; `castor test`: PASS (4477 tests, 17536 assertions).; `castor deptrac`: PASS (0 violations; uncovered reduced to 2213).; `castor phpstan`: PASS (0 errors).; `castor cs-check`: PASS.; `castor test:tui`: PASS (37 tests, 294 assertions; real tmux, including new slash-completion journey proof).; `castor clean:cleanup:workers:list`: PASS (no stale QA worker candidates).; `castor check`: intentionally not run in task-start; task-to-pr owns the final gate.
- Summary: Implementation fork completed and committed as `513ee461e` (`refactor(tui): close ImagePaste deptrac gap and DRY session UI helpers`). Verified clean task worktree, 32 expected files changed (576 insertions, 294 deletions), including the missing TuiImagePaste collector, narrow footer/status/SelectList/runtime-callback owners, reference-proven cleanup, and tests. Verified `tests/Tui/E2E/TuiJourneyE2eTest.php` adds a real replay-backed TmuxHarness slash-completion path (`/he` → Completions `/help` → Tab accept → overlay closes). KEEP decisions: CtrlCInputInterceptor by-reference capture is load-bearing; SubagentLiveViewState fresh-enter Completed fallback remains to avoid changing failed/cancelled behavior. Testing skill and `tests/AGENTS.md` were read and followed. No PR/reviewer/check transition performed.

## Task workflow update - 2026-08-16T02:20:07.213Z
- Summary: task-to-pr reviewer decision: REQUEST CHANGES. Blocking: deduplicate repeated RuntimeEventCallbacks construction in SubagentLiveChildViewPoller with one private factory/helper; delete empty RuntimeEventCallbacksTest::setUp(); delete trivial MAX_VISIBLE constant-equals-10 test. Specification fidelity, behavior preservation, Deptrac edges, ExtensionApi boundary, and real TmuxHarness proof otherwise approved.

## Task workflow update - 2026-08-16T02:20:23.660Z
- Recorded fork run: ww28sm4ag6sh
- Summary: Launched focused fix fork for the three reviewer blockers only. Fork will run focused Castor tests/phpstan/cs-check and commit; then reviewer re-review is required before full task-to-pr validation.

## Task workflow update - 2026-08-16T02:23:01.038Z
- Recorded fork run: ww28sm4ag6sh
- Validation: Focused `castor test --filter='RuntimeEventCallbacksTest|SelectListKeybindingsTest|SubagentLiveChildViewPollerReplayTest'`: PASS (8 tests, 38 assertions).; `castor phpstan --path=src/Tui/Runtime`: PASS (0 errors).; `castor cs-check`: PASS.
- Summary: Reviewer blockers fixed in commit `9a91b035b`: deduplicated child-poller RuntimeEventCallbacks construction; removed empty test setup and trivial constant assertion. Verified clean worktree and exact 3-file diff (23 insertions, 27 deletions).

## Task workflow update - 2026-08-16T02:30:46.734Z
- Summary: Re-review after `9a91b035b`: APPROVED. Reviewer confirmed all prior blockers resolved, full specification-fidelity gate passes, no behavior/public/ExtensionApi drift, no per-session redesign, and replay-backed TmuxHarness proof remains appropriate. Proceeding with required focused Castor lanes before CODE-REVIEW transition.

## Task workflow update - 2026-08-16T02:35:25.611Z
- Validation: `castor test`: PASS (4476 tests, 17535 assertions).; `castor deptrac`: PASS (0 violations, 0 errors).; `castor phpstan`: PASS (0 errors).; `castor cs-check`: PASS (0 files fixed).; `castor test:tui`: PASS (37 tests, 290 assertions; replay-backed real tmux).; `castor test:llm-real`: PASS (13 tests, 144 assertions; proxy warmup before check gate).; `castor clean:cleanup:workers:list`: PASS (no stale QA worker candidates).; Worktree clean at `9a91b035b`; reviewer decision APPROVED.
- Summary: task-to-pr review APPROVED at HEAD `9a91b035b`. Full focused validation rerun in clean worktree; real TmuxHarness lane passed and no stale workers remain. llama-proxy/live lane warmed successfully before deterministic CODE-REVIEW gate.

## Task workflow update - 2026-08-16T02:37:41.762Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (123.1s).
- Pushed task/tui-01-close-architecture-gap-and-dry-session-ui-helpers to origin.
- branch 'task/tui-01-close-architecture-gap-and-dry-session-ui-helpers' set up to track 'origin/task/tui-01-close-architecture-gap-and-dry-session-ui-helpers'.
- Created PR: https://github.com/ineersa/agent-core/pull/391
- Validation: Reviewer: APPROVED.; castor test: PASS.; castor deptrac: PASS.; castor phpstan: PASS.; castor cs-check: PASS.; castor test:tui: PASS.; castor test:llm-real warmup: PASS.; No stale QA workers.
- Summary: Reviewer APPROVED after one focused fix iteration. HEAD `9a91b035b`; all required focused Castor lanes passed, including replay-backed TmuxHarness E2E, and worktree is clean.

## Task workflow update - 2026-08-16T02:44:59.467Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/tui-01-close-architecture-gap-and-dry-session-ui-helpers.
- Merged task/tui-01-close-architecture-gap-and-dry-session-ui-helpers into integration checkout.
- Merge made by the 'ort' strategy.
 depfile.yaml                                       |  10 ++
 src/Tui/Layout/TuiSlotRegistry.php                 |  15 +--
 src/Tui/Listener/CompletionMenu.php                |   7 +-
 src/Tui/Listener/CtrlCInputInterceptor.php         |   5 +-
 src/Tui/Listener/FooterStateInitializer.php        |  22 ++++
 .../Listener/LoadedResourcesStartupRegistrar.php   |   2 +-
 src/Tui/Listener/ModelCommandHandler.php           |   6 +-
 src/Tui/Listener/ModelControlListener.php          |   4 +-
 src/Tui/Listener/SubmitListener.php                |   2 +-
 src/Tui/Listener/TickPollListener.php              |  23 +---
 src/Tui/Picker/FavoritePickerController.php        |  14 +--
 src/Tui/Picker/HistoryPickerController.php         |  14 +--
 src/Tui/Picker/ModelPickerController.php           |  21 +---
 src/Tui/Picker/SessionPickerController.php         |  14 +--
 src/Tui/Picker/SubagentLivePickerController.php    |  14 +--
 src/Tui/Question/QuestionController.php            |  20 +---
 src/Tui/Runtime/RuntimeEventCallbacks.php          | 117 +++++++++++++++++++++
 src/Tui/Runtime/RuntimeEventPoller.php             |  79 +++-----------
 src/Tui/Runtime/SubagentLiveChildViewPoller.php    |  85 ++++-----------
 src/Tui/Runtime/SubagentLiveStatusEnum.php         |  20 ++++
 src/Tui/Runtime/SubagentLiveViewState.php          |  19 ++--
 src/Tui/Widget/SelectListKeybindings.php           |  35 ++++++
 src/Tui/Widget/TuiRenderContext.php                |  13 +--
 .../Tui/CompactHeader/CompactHeaderWidgetTest.php  |   1 -
 tests/Tui/E2E/TuiJourneyE2eTest.php                |  68 +++++++++++-
 tests/Tui/Footer/FooterBarWidgetTest.php           |  21 +++-
 tests/Tui/Question/QuestionControllerTest.php      |   5 +-
 tests/Tui/Runtime/RuntimeEventCallbacksTest.php    | 105 ++++++++++++++++++
 tests/Tui/Runtime/SubagentLiveStatusEnumTest.php   |  46 ++++++++
 tests/Tui/Startup/LoadedResourcesWidgetTest.php    |   2 +-
 .../Tui/Transcript/SubagentResultRendererTest.php  |   2 +-
 tests/Tui/Widget/SelectListKeybindingsTest.php     |  35 ++++++
 32 files changed, 562 insertions(+), 284 deletions(-)
 create mode 100644 src/Tui/Runtime/RuntimeEventCallbacks.php
 create mode 100644 src/Tui/Widget/SelectListKeybindings.php
 create mode 100644 tests/Tui/Runtime/RuntimeEventCallbacksTest.php
 create mode 100644 tests/Tui/Runtime/SubagentLiveStatusEnumTest.php
 create mode 100644 tests/Tui/Widget/SelectListKeybindingsTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/tui-01-close-architecture-gap-and-dry-session-ui-helpers.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/tui-01-close-architecture-gap-and-dry-session-ui-helpers.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #391 state: MERGED.; Integration checkout clean before transition.
- Summary: PR #391 confirmed merged on GitHub at 2026-08-16T02:44:32Z (merge commit a224eb78262cc25764a1d5b51d2f5c67ec07e043). Proceeding with integration merge, pull, and worktree cleanup.

## Task workflow update - 2026-08-16T02:47:40.212Z
- Validation: Post-merge `LLM_MODE=true castor check`: PASS (all 8 lanes).; Unit/integration: 4476 tests, 17541 assertions.; Controller replay: 12 tests, 165 assertions.; Replay-backed TUI/tmux: 37 tests, 290 assertions.; Live LLM: 13 tests, 144 assertions.; Deptrac, PHPStan, CS check, docs validation: PASS.; QA artifact integrity, leak check, and llama-proxy cache guard: PASS.; Task worktree removed; integration checkout clean.
- Summary: task-done complete: PR #391 merged, task branch integrated locally, remote merge pulled, exact task worktree/IDE project/exclusions cleaned up, and integration checkout remains clean.

## Task workflow update - 2026-08-18T00:06:36.251Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
