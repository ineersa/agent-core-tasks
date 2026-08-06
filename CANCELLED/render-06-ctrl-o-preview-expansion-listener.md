# RENDER-06: Ctrl+O preview expansion listener — SUPERSEDED BY RENDER-07

> Superseded by RENDER-07 (`render-07-docs-snapshots-product-validation`).
> The Ctrl+O listener/hotkey/runtime behavior was implemented there because RENDER-07 validation depended on it and RENDER-06 had not been started as a standalone task.
> Keep this file as historical planning context; do not start a separate RENDER-06 implementation unless the RENDER-07 work is explicitly reverted or split.

## Goal
Part of `.pi/plans/tui-rich-transcript-blocks-plan.md`, revised for the current TUI architecture.

Add the session-local TUI input behavior that toggles transcript preview expansion for previewable blocks. The listener must follow the existing `TuiListenerRegistrar` pattern and operate on `TuiSessionState` only.

Order: depends on RENDER-01 for display state, RENDER-04 for normal tool result preview behavior, and RENDER-05 for diff preview behavior.

Scope:
- Add listener-owned `Ctrl+O` handling using the existing TUI listener registrar style.
- Ensure the listener receives `Ctrl+O` before the editor consumes it; validate the real input path rather than assuming the Symfony `EditorWidget` binding is inert.
- Toggle only `TranscriptDisplayState::previewableBlocksExpanded`.
- Keep state session-only; do not persist to settings, session metadata, runtime events, or transcript events.
- Invalidate/re-render the transcript immediately after toggling.
- Update hotkey metadata so `/hotkeys` documents the preview expansion key.

Non-goals:
- No persisted user setting mutation.
- No canonical event mutation.
- Only `Ctrl+O` is introduced for preview expansion in v1.
- No effect on non-previewable block kinds.

Glyph/testing stability:
- `Ctrl+O` tests should prove preview expansion behavior without asserting unrelated transcript glyphs.
- `/hotkeys` documentation may assert the keybinding text, but should not require broad snapshot rewrites.

## Acceptance criteria
- `Ctrl+O` toggles session-only preview expansion state.
- The `Ctrl+O` listener is registered through a `TuiListenerRegistrar` service.
- The real TUI input path proves the listener receives the key before the editor consumes it.
- The toggle does not mutate Hatfield settings, session metadata, runtime commands, or canonical transcript events.
- Normal tool result previews from RENDER-04 respond to the toggle.
- Diff previews from RENDER-05 respond to the toggle.
- User, assistant, thinking, system, error, progress, question, approval, cancelled, and tool-call blocks are unaffected.
- `/hotkeys` includes the preview expansion binding.
- Focused Castor validation is reported at the lowest correct TUI layer for the input behavior, plus `castor phpstan` and `castor deptrac`.

## Workflow metadata
Status: SUPERSEDED by RENDER-07 (left in TODO directory only as historical planning context)
Branch: task/render-07-docs-snapshots-product-validation
Worktree: /home/ineersa/projects/agent-core-worktrees/render-07-docs-snapshots-product-validation
Fork run: x2y6y124ztz0 (initial incomplete implementation); direct finish authorized after fork failures
PR URL:
PR Status:
Started: folded into RENDER-07 on 2026-07-01
Completed: superseded by RENDER-07 commit c09494f0b (`feat(tui): document and validate rich transcript previews`)

## Work log
- Created: 2026-05-22T19:09:14.549Z
- Revised: 2026-06-29 — Made RENDER-06 depend on actual preview renderers.
- Revised: 2026-06-29 — Added keybinding test-churn mitigation requirements.
- Superseded: 2026-07-01 — Folded into RENDER-07 because RENDER-07 acceptance/docs/product-validation required the Ctrl+O runtime toggle. Implemented in RENDER-07 commit c09494f0b with `PreviewExpansionInputListener`, `/hotkeys` metadata, session-only `TranscriptDisplayState` toggling, virtual input tests, and replay-backed TmuxHarness product validation. Focused validation passed there: `castor test --filter='PreviewExpansionInputListenerTest|TuiVirtualInputTest'`, `castor test:tui --filter='TuiRichTranscriptProductValidationE2eTest'`, `castor deptrac`, `castor phpstan`, `castor cs-check`.
