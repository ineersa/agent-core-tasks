# TUI: bound question overlay to available rows so oversized multiline options can't push the editor off-screen

## Goal
## Evidence (worktree try-out of multiline picker, 2026-09-14)

User snapshot: `.hatfield/tmp/tui/snapshots/snapshot-ansi-20260914-145147.ansi` — a ~29-row wrapped option ("Option Omega") inside the question picker; the full overlay content ran ~40 rows and the input row ("Type your answer") landed past the visible window. User report: picker "dies" when one option is larger than the window (~10+ wrapped rows in Hatfield's pane).

Probes (var/tmp/trial/giant_fullframe_probe.php):
- Overlay rendered alone with a bounded budget: emits exactly 12 rows at a 12-row RenderContext — the upstream multiline widget clamps correctly when the budget is real (43/43 widget tests green).
- Full ChatScreen frame (banner + transcript + status + overlay + editor + footer): emits **38 rows identically at 80x24 and 80x16** — the vertical stack renders every child at natural height with no terminal constraint. The overlay is inserted via `insertOverlayBeforeEditor()` with no row budget, so `SelectListWidget` never sees real available rows and its budgeting never engages.
- `SynchronizedCursorScreenWriter` bottom-aligns taller-than-viewport frames (`$firstVisibleRow = max(0, count($lines) - rows)`), so the failure is a scrolled/mangled screen with the focused editor off-screen — not a crash. Symptom reads as "picker died".

## Root cause

Hatfield composition, not Symfony TUI: the ChatScreen stack and overlay insertion never derive a row budget from the terminal size. Upstream Symfony TUI treats taller-than-viewport frames as by-design (scroll into scrollback, upstream b6057de); the app must compose within the viewport. Pre-existing in weakened form (single-line picker with `maxVisible: 10` could already overflow short terminals); multiline options amplify it to a single option blowing the window.

## Fix direction (not a mandated design)

Derive the overlay's real row budget at render time: terminal rows minus the chrome above (banner/header/status/compact header as currently visible) minus the editor block below (editorSep + editor + footerSep + footer). Pass that budget down so the question container — and the SelectListWidget inside it — renders against bounded rows. No upstream changes; no editor redesign.

## Relevant code

- `src/Tui/Screen/ChatScreen.php` — `mount()` stack composition, `insertOverlayBeforeEditor()`
- `src/Tui/Question/QuestionController.php` — `open()`/`addSelectList()` (listWidget construction, `multiline: true`, `maxVisible: SelectListKeybindings::MAX_VISIBLE`)
- `src/Tui/Terminal/SynchronizedCursorScreenWriter.php` — taller-than-viewport handling (bottom-align) to keep consistent with
- Upstream dependency: symfony/tui `multiline` patch (ineersa/tui `multiline-select-options` @ 3baefb2) already wired in the try-out; the widget-side budgeting exists and works — this task only supplies the budget

## Constraints

- Do not regress the existing picker for short options (byte-identical rendering when nothing overflows).
- Deterministic virtual/full-frame tests only; tmux smoke at most as a final sanity check per tui-testing lane rules.
- No new user-visible settings or API surface beyond the internal layout budget.

## Acceptance criteria
- A question overlay containing an option whose wrapped label exceeds the available space renders clamped to the rows actually available above the editor — the editor row stays on-screen and focusable at e.g. 80x16, 80x24, and the user's tall-window case
- The SelectListWidget receives a RenderContext whose rows equal the real budget (terminal rows − chrome above − editor block below), so the upstream multiline window-fitting (leading trim, clamp-to-first-rows, conditional indicator) engages
- Normal-height options render byte-identically to today (no visual/behavior regression in the standard picker), including the no-scroll exact-fit case
- Proven deterministically at the lowest correct layer: virtual full-frame probes (ChatScreen mounted + question open) asserting emitted frame rows ≤ terminal rows; existing QuestionControllerTest suite stays green; no tmux-only proof
- Full `castor check` at CODE-REVIEW transition (TUI runtime flow)

## Workflow metadata
Status: CANCELLED
Branch: task/2026-09-14-tui-question-overlay-row-budget
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-question-overlay-row-budget
Fork run:
PR URL:
PR Status:
Started: 2026-09-17T16:26:36+00:00
Completed:

## Work log
- Created: 2026-09-14T19:52:29+00:00

## Task workflow update - 2026-09-17T16:26:36+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-14-tui-question-overlay-row-budget.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-question-overlay-row-budget.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-question-overlay-row-budget.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-question-overlay-row-budget.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-question-overlay-row-budget.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-question-overlay-row-budget/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-question-overlay-row-budget.

## Task workflow update - 2026-09-17T16:26:49+00:00
- Summary: User approves fix plus moving question above prompts/skills/agents/MCP compact header. Whole question area capped at12 rows, constrained by real terminal room with editor/footer visible. Latest placement requirement supersedes byte-identical old placement. Delegate cohesive layout implementation to prior investigating fork because full-frame layout iteration/context substantial.
- Ownership: owner=fork; fork_run=agent_891aef8aed4ef454; revision=task-start baseline; scope=question placement and bounded layout with full-frame proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-17T16:28:20+00:00
- Moved IN-PROGRESS → CANCELLED.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-question-overlay-row-budget: ide_close_project returned isError.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-question-overlay-row-budget.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-question-overlay-row-budget.
- Summary: User requests cancellation of separate task/worktree. Implement approved row budget and placement in existing upstream-multiline-select-options trial worktree instead. No edits in this worktree.
