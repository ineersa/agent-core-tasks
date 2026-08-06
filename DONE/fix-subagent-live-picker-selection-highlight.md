# Fix subagent live picker selection highlight

## Goal
The `/agents-live` subagent picker shows two competing selection styles: the first row remains highlighted with one color while arrow navigation moves a second, differently colored highlight. The visible selected row therefore appears stuck on the first item even though the native arrow selection changes.

Likely root cause already identified in `src/Tui/Picker/SubagentLivePickerController.php`: `buildItems(..., selectedIndex: 0)` manually wraps the first label with `ThemeColorEnum::Accent`, while Symfony TUI's `SelectListWidget` independently renders its native selected-row highlight. The dismiss/rebuild path also pre-colors an item before calling `setSelectedIndex()`.

Fix this by reusing the same native picker selection behavior used by the other Hatfield pickers (`PickerOverlay` + `SelectListWidget`) rather than maintaining a second manual label color/state. Remove obsolete custom selected-index coloring/signatures where possible; do not introduce another picker implementation.

## Acceptance criteria
- With at least three subagents in `/agents-live`, exactly one row is visually highlighted at a time.
- Up/down arrow navigation moves the visible highlight to the native selected row; the first row immediately loses highlight when selection moves away.
- The selected-row color/style matches the shared `SelectListWidget` picker behavior used by model/session/tree/favorite pickers rather than a subagent-specific accent.
- Opening and rebuilding the picker, including after dismissing a finished child, does not add a second manually colored row and preserves a valid native selection index.
- Enter, export, and dismiss actions continue to target the row selected by the native `SelectListWidget`.
- Add a virtual/in-process TUI regression proof that renders the real picker with multiple rows, sends arrow input, and asserts the screen-buffer highlight follows selection. A label-only/buildItems unit test is insufficient; no new tmux test is needed for this purely local picker behavior.
- Relevant Castor tests, `castor deptrac`, `castor phpstan`, and `castor cs-check` pass; full `castor check` passes before CODE-REVIEW because TUI runtime behavior is touched.

## Workflow metadata
Status: DONE
Branch: task/fix-subagent-live-picker-selection-highlight
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-subagent-live-picker-selection-highlight
Fork run: uk3g0jonpuna
PR URL: https://github.com/ineersa/agent-core/pull/350
PR Status: merged
Started: 2026-08-01T01:32:34.930Z
Completed: 2026-08-01T02:58:09.756Z

## Work log
- Created: 2026-07-23T20:55:33.770Z

## Task workflow update - 2026-08-01T01:32:34.930Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-subagent-live-picker-selection-highlight.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-subagent-live-picker-selection-highlight.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-subagent-live-picker-selection-highlight.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-subagent-live-picker-selection-highlight.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-subagent-live-picker-selection-highlight.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-subagent-live-picker-selection-highlight.
- Summary: Task claimed. Latest task-start instruction extends the task acceptance criteria: implementation must include both the requested virtual/in-process picker regression and a real replay-backed TmuxHarness E2E proof exercising the user-visible /agents-live navigation path.

## Task workflow update - 2026-08-01T01:44:07.402Z
- Recorded fork run: 2twlkdt801ju
- Summary: Implementation fork launched in /home/ineersa/projects/agent-core-worktrees/fix-subagent-live-picker-selection-highlight. Scope: remove manual Accent/selected-index label state, keep native SelectListWidget selection/action ownership, add both real-controller virtual ScreenBuffer regression and latest-user-required replay-backed TmuxHarness /agents-live three-row arrow/style proof.
- Scouts confirmed root cause in SubagentLivePickerController::buildItems/open/dismiss rebuild; shared PickerOverlay + SelectListWidget already owns selection. Existing unit arrow test only inspects item count and existing Tmux journey has one child, so both require focused regression upgrades. No external research needed.

## Task workflow update - 2026-08-01T01:52:22.650Z
- Recorded fork run: 2twlkdt801ju
- Validation: Fork confirmed it read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before test work and used Castor only.; `castor test --filter=SubagentLivePickerControllerTest` — OK (17 tests, 99 assertions).; `castor test:tui --filter=TuiSubagentLiveViewE2eTest` — OK (2 tests, 21 assertions).; `castor deptrac` — 0 violations.; `castor phpstan` — 0 errors.; `castor cs-check` — clean after Castor cs-fix.; Parent verified clean worktree at cbd70892a and 5 expected changed files (+521/-49).; Parent verified the TmuxHarness test exercises the actual replay-backed create-session → canonical three-child events → `/resume` → `/agents-live` → Down path, scopes ANSI/style assertions to picker rows, rejects stale Accent, saves snapshots, uses no live LLM, and relies on existing harness teardown.
- Summary: Implementation complete in commit cbd70892adb7e19eb7fffb453c0cb2734a21deed. Removed manual Accent/selected-index coloring from SubagentLivePickerController, left native SelectListWidget ownership for highlighting and actions, simplified dismiss rebuild to plain items plus valid index 0, and updated all buildItems call sites. Added the required real-controller virtual ScreenBuffer proof and latest-user-required replay-backed TmuxHarness /agents-live proof with three rows, Down input, ANSI native-bold/arrow versus stale-Accent assertions, snapshots, and standard cleanup.

## Task workflow update - 2026-08-01T02:10:19.787Z
- Recorded fork run: uk3g0jonpuna
- Summary: Initial reviewer verdict at cbd70892a: APPROVED WITH SUGGESTIONS; specification fidelity PASS; virtual TUI proof PASS; replay-backed TmuxHarness proof PASS; no correctness, security, integration, or production-design blockers. Cleanup fork launched for two sensible ponytail findings only: delete unused child-export files from the three-row picker fixture and replace two manual native-row count loops with typed stdlib sums.
- Reviewer explicitly confirmed reading root AGENTS.md, testing skill, tests/AGENTS.md, and task file. Skipped subjective/non-actionable NTHs: preserve duplicate-row assertion required by latest acceptance, avoid broader tmux capture/harness changes, no shared ANSI parser for two differing contexts, no cosmetic timestamp/octal churn. Re-review is required after cleanup commit.

## Task workflow update - 2026-08-01T02:18:58.548Z
- Recorded fork run: uk3g0jonpuna
- Validation: Cleanup fork confirmed root AGENTS.md, `.agents/skills/testing/SKILL.md`, and `tests/AGENTS.md` were read and followed.; `castor test --filter=SubagentLivePickerControllerTest` — OK (17 tests, 99 assertions).; `castor test:tui --filter=TuiSubagentLiveViewE2eTest` — OK (2 tests, 21 assertions).; `castor phpstan` — 0 errors.; `castor cs-check` — clean.; Parent full focused gate: `castor test` — OK (4405 tests, 16317 assertions; 30.4s).; Parent full focused gate: `castor test:tui` — OK (31 tests, 189 assertions; 95.9s; replay-backed, no live LLM).; Parent full focused gate: `castor deptrac` — 0 violations.; Parent full focused gate: `castor phpstan` — 0 errors.; Parent full focused gate: `castor cs-check` — clean (0 files fixed).
- Summary: Final reviewer verdict at HEAD 410e85913c9d2a2f3288ca787d9953330f4fd02a: APPROVED. Specification fidelity PASS; real VirtualTuiHarness proof PASS; replay-backed no-live-LLM TmuxHarness proof PASS; cleanup correctness PASS; security/minimality PASS; ready for CODE-REVIEW.
- Reviewer cleanup commit 410e85913 (`test: drop unused three-child export fixtures and simplify native-row counts`) changed 3 test files (+2/-32), removing unused child export setup and replacing manual count loops with stdlib sums. Final branch diff: 5 files, +491/-49; worktree clean.

## Task workflow update - 2026-08-01T02:22:47.868Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (106.4s).
- Pushed task/fix-subagent-live-picker-selection-highlight to origin.
- branch 'task/fix-subagent-live-picker-selection-highlight' set up to track 'origin/task/fix-subagent-live-picker-selection-highlight'.
- Created PR: https://github.com/ineersa/agent-core/pull/350
- Validation: castor test — 4405 tests, 16317 assertions OK; castor test:tui — 31 tests, 189 assertions OK; castor deptrac — 0 violations; castor phpstan — 0 errors; castor cs-check — clean; Reviewer — APPROVED; specification fidelity, virtual TUI proof, TmuxHarness proof, security, and minimality all PASS; First move gate: no branch failure; lock acquisition timed out after 60s behind sibling castor check PID 763892; process subsequently exited and retry proceeded
- Summary: Reviewer APPROVED HEAD 410e85913. Removed competing manual Accent highlight and retained native SelectListWidget selection ownership. Added both required real-controller virtual proof and replay-backed TmuxHarness `/agents-live` three-row Down/style proof. Full focused Castor validation passed. First CODE-REVIEW transition attempt was blocked only by the repository-wide Castor check lock held by sibling task fix-runaway-recursive-search-output-and-token-estimates; holder exited normally, so transition was retried without code changes.

## Task workflow update - 2026-08-01T02:58:09.756Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-subagent-live-picker-selection-highlight into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Picker/SubagentLivePickerController.php    |  34 ++--
 tests/Tui/E2E/TuiSubagentLiveViewE2eTest.php       | 203 +++++++++++++++++++++
 .../Picker/SubagentLivePickerControllerTest.php    | 177 +++++++++++++++---
 tests/Tui/Support/SubagentLiveScenarioHarness.php  |   2 +-
 .../Tui/Support/SubagentProgressEventsFixture.php  | 124 +++++++++++++
 5 files changed, 491 insertions(+), 49 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-subagent-live-picker-selection-highlight.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-subagent-live-picker-selection-highlight.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #350 state: MERGED; PR merge commit: 33eb2b4e2443029c8cd8b58e958ee115418422d1; Integration checkout clean before merge
- Summary: PR #350 merged on GitHub at 2026-08-01T02:57:41Z (merge commit 33eb2b4e2443029c8cd8b58e958ee115418422d1). Reviewer approved and all pre-merge Castor gates passed. Moving task to DONE and cleaning the task worktree.

## Task workflow update - 2026-08-01T03:00:19.485Z
- Validation: `LLM_MODE=true castor check` — quality OK (QA run qa-20260801-025813-817213-777fb3d2).; PHAR build and smoke — OK.; castor check test lane — 4418 tests, 16358 assertions OK.; castor check controller replay lane — 11 tests, 160 assertions OK.; castor check TUI replay lane — 31 tests, 191 assertions OK.; castor check live LLM lane — 13 tests, 144 assertions OK.; deptrac — 0 violations; phpstan — 0 errors; cs-check — clean.; QA artifact integrity — OK (7 lane logs); QA process/tmux leak check — OK; exact-run cache cleanup — OK.; llama-proxy cache guard — stable at 223 entries.; Integration checkout clean; task worktree removed.
- Summary: Post-merge validation complete. Integration checkout contains PR #350, task is DONE, worktree and IDEA exclusions removed, and the integration checkout is clean.
