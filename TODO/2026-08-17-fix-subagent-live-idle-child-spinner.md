# fix-subagent-live-idle-child-spinner

## Goal
User-reported (2026-08-17, manual test during tui-05 PR #403 review): in the subagent live view, an idle child agent shows a spinner, implying activity when there is none.

Mechanism (TickPollListener.php:185-189, 210-228): whenever the subagent live view is open, setWorkingVisible(true) is unconditional (only the main-view pending-question path hides it). The working message can be 'Child agent idle' (parent idle/terminal + child idle), which renders next to ChatScreen's always-spinning LoaderWidget — idle text with an activity spinner.

Likely smallest fix: while the live view is open and the combined working state is idle (child idle and parent idle/terminal — i.e. lastLiveWorkingMessage resolves to just the idle text), use the existing LoaderWidget idle/finished lifecycle (ChatScreen::syncWorkingSlot already models active/idle/hidden with a stable two-line footprint) instead of keeping the spinner running. WaitingHuman and Cancelling keep the spinner. Do not change the message text semantics or attention footer.

Scope guards: no new public API/setting/command; keep spinner for all active/cancelling/waiting states; keep two-line footprint stable; reuse existing RunActivityStateEnum-driven match in TickPollListener.

## Acceptance criteria
- Subagent live view open with child idle (and parent idle/terminal) shows an idle/finished indicator, not a spinning loader
- Working/cancelling/waiting/active states keep the spinner exactly as today
- Two-line working-slot footprint and idle→active transitions remain stable (no layout jump)
- Virtual test at the lowest correct layer (TickPollListener/harness) covering idle vs active child states; focused castor test + test:tui for the touched lane; no new setting/command
- Public ExtensionApi untouched

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
- Created: 2026-08-17T23:56:35.111Z

## Task workflow update - 2026-08-17T23:58:20.984Z
- User confirmed: message text is fine; the loader widget itself must swap to its static idle indicator. Verified ChatScreen::syncWorkingSlot (main checkout lines 720-745): empty message -> setFinishedIndicator('●') + 'idle' + stop(); non-empty -> spinner start(). So the widget lifecycle is already correct; only the live-view branch keeps the message non-empty when both parent and child are idle.
- Confirmed fix site: TickPollListener live-active branch (~lines 210-228) — when parent idle/terminal AND child idle (no WaitingHuman/Cancelling), resolve the working message to '' so the existing syncWorkingSlot static path renders, instead of the literal 'Child agent idle' string driving the spinner. All other states keep the spinner. Single decision point, no widget change.
