# GF-03: Stabilize agents-live picker rows during navigation

## Goal
Extract the `/agents-live` picker row-stability fix from the abandoned fork MVP branch into a fresh task based on current origin/main.

Observed historical bug: `SubagentLivePickerController::openWithChildren()` registered an `onSelectionChange` callback that rebuilt the entire item list via `setItems()` on every arrow-key movement. `SelectListWidget` already owns selection styling; rebuilding resets selection/invalidation and can leave duplicated or stale rows under incremental rendering.

Historical reference only:
- `9f971b378` — removed per-arrow list rebuild.

This is a small existing-subagent TUI bug fix. Do not include fork behavior, context statistics, export, nested discovery, or broader picker redesign.

## Acceptance criteria
- First commit/push contains only a reviewed RED virtual TUI specification test; no production implementation. The accepted test is immutable during implementation.
- Red test opens `/agents-live` with multiple existing subagents and performs repeated up/down navigation through the real picker/widget input path.
- Rendered picker contains exactly one row per child after every navigation step; no duplicated/stale selection rows accumulate.
- Selection moves correctly without rebuilding/resetting the item list.
- Dismiss and selection behavior remain unchanged.
- Implementation is the smallest production correction; no picker redesign or fork-specific code.
- All QA uses Castor; no historical branch tests are copied.

## Workflow metadata
Status: ARCHIVE
Branch: (deleted)
Worktree: (removed)
Fork run: fork-2nd-launch
PR URL:
PR Status:
Started: 2026-07-11T20:14:41.861Z
Completed: 2026-07-11

## Resolution
**Duplicate.** The same fix was independently applied in `atomic-per-run-file-sequence-allocation` (commit `c67241263` — "fix(tui): keep agents live picker rows stable on arrow navigation"). The core fix (removing `onSelectionChange` callback that rebuilt items via `setItems()` on every arrow key) is identical. The `atomic-per-run-file-sequence-allocation` branch already has its own tests for this behavior.

GF-03 went slightly further (also removed accent-color ANSI embedding from `buildItems()`), but that is a cosmetic cleanup, not required for the row-stability fix.

**Branch `task/gf-agents-live-picker-row-stability` deleted. Worktree removed. No PR created.**

GF-03 attempted commits (discarded):
- `3945a7dcc` RED: agents-live picker row stability spec test
- `490d0740c` Fix: remove per-arrow item rebuild from agents-live picker

## Work log
- Created: 2026-07-11T17:43:23.429Z

## Task workflow update - 2026-07-11T20:14:41.861Z
- Moved TODO → IN-PROGRESS.
- Created branch task/gf-agents-live-picker-row-stability.
- Created worktree /home/ineersa/projects/agent-core-worktrees/gf-agents-live-picker-row-stability.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/gf-agents-live-picker-row-stability.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/gf-agents-live-picker-row-stability.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/gf-agents-live-picker-row-stability.

## Task workflow update - 2026-07-11T20:26:18.662Z
- Recorded fork run: fork-2nd-launch
- Validation: castor test --filter=SubagentLivePickerOverlayVirtualTest: OK (2 tests, 24 assertions); castor test --filter=SubagentLivePickerControllerTest: OK (14 tests, 45 assertions); castor test: OK (4235 tests, 13811 assertions); castor deptrac: 0 violations; castor phpstan: OK; castor cs-check: OK
- Summary: Implementation complete.

## Task workflow update - 2026-07-11
- Marked DONE as duplicate. Same fix exists in `atomic-per-run-file-sequence-allocation` (`c67241263`).
- Deleted branch `task/gf-agents-live-picker-row-stability`.
- Removed worktree `/home/ineersa/projects/agent-core-worktrees/gf-agents-live-picker-row-stability`.

## Task workflow update - 2026-08-06T20:59:11.359Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
