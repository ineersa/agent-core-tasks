# Reduce picker overlay flicker by avoiding unnecessary full TUI redraws

## Goal
User observed a brief full-screen flicker when opening bottom picker overlays, including /resume. This appears separate from session-08b file rewind: file rewind/tree now work correctly after using Symfony SelectListWidget selection and keeping pickers below editor, but picker overlay mount/close still likely force full render.

Context from session-08b:
- `PickerOverlay::mount()` currently calls `$tui->requestRender(true)`.
- `PickerOverlay::close()` calls `$this->screen->requestRender(true)` when requestRender is true.
- `SessionPickerController` and `SubagentLivePickerController` also have `requestRender(true)` paths.
- CompletionMenu uses `ChatScreen::insertOverlayAfterEditor()` without an explicit forced render and may be a useful comparison point.
- The previous stale overlay corruption was likely caused by /tree and /rewind rebuilding SelectListWidget items on every selection change; that was fixed by commit f9d4e6e90 in session-08b by using static list items and letting Symfony TUI own selection.

Hypothesis:
- The remaining flicker is caused by forced/full frame renders (`requestRender(true)`) during generic picker mount/close or picker-specific selection changes.
- We should prefer incremental render (`requestRender()`) for picker open/close and only use forced render where a test/live repro proves it is required.

Important constraints:
- This is a general TUI picker lifecycle task, not part of file rewind semantics.
- Use Symfony TUI's built-in widget rendering/selection behavior; do not invent per-arrow item rebuild or custom repaint machinery unless necessary.
- Keep picker overlays below editor unless a specific picker has a proven reason not to.
- Follow AGENTS.md: load testing skill and tests/AGENTS.md before TUI/test work; all QA through Castor.

## Acceptance criteria
- Identify all picker overlay paths that force full renders on mount/close/navigation and explain which are necessary vs unnecessary.
- Reduce visible flicker for /resume and other bottom pickers by replacing unnecessary forced renders with normal incremental renders or a narrower invalidation strategy.
- Do not regress previously fixed /tree and /rewind overlay behavior: no footer/model bleed, no duplicate/stale rows, no raw multiline picker labels.
- Add/update lowest-correct-layer TUI tests proving picker mount/close/render behavior and guarding against stale overlay artifacts without relying on full redraws where possible.
- Run focused Castor validation for affected TUI tests plus `castor test:tui`, `castor deptrac`, `castor phpstan`, and `castor cs-check`; run live tmux smoke if virtual/replay tests cannot prove flicker/stale-paint behavior.

## Workflow metadata
Status: TODO
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-07-04T18:32:04.426Z

## Task workflow update - 2026-07-21T17:09:55.313Z
- Summary: Cancellation requested on 2026-07-21 because the picker-overlay flicker has not been observed since subsequent TUI fixes. The task was hypothesis-driven and has no current reproducible symptom, so speculative render-path changes are not warranted. Reopen only if flicker returns with a reproducible picker/action and terminal capture.
- 2026-07-21: Treated as fixed or superseded based on the absence of recurring picker-overlay flicker. Cancel rather than changing full-render behavior without a current reproduction.
