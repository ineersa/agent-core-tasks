# Fix agents-live view showing the main transcript after Ctrl+O preview toggle

## Goal
In a child live view (`/agents-live`), pressing Ctrl+O to toggle preview expansion swaps the displayed transcript to the main agent's while the child view still appears active.

## Bug

While `subagentLiveView->active` is true, ChatScreen is fed `subagentLiveView->childTranscript`. Ctrl+O unconditionally re-pushes the parent transcript, so the screen shows the main transcript even though live view state (status, picker, footer, input routing) is still on the child. Not severe: `/agents-main` or Ctrl+\\ then re-entering the child view clears it, which is likely why it went unnoticed.

## Reproduction

1. Open a session that has at least one child run (subagent or fork).
2. Enter the child live view with `/agents-live`.
3. Press Ctrl+O to toggle preview expansion.
4. Observe: the content shown is the main/parent transcript, not the child's, while the view still behaves as agents-live.
5. Press Ctrl+\\ (or `/agents-main`) and re-enter `/agents-live` to confirm it clears.

## Suspected root cause

`PreviewExpansionInputListener::register()` always calls `$screen->setTranscriptBlocks($state->transcript)` after toggling `previewableBlocksExpanded` — `src/Tui/Listener/PreviewExpansionInputListener.php:44`. Live-view paths instead feed the screen from `subagentLiveView->childTranscript` (`src/Tui/Listener/SubmitListener.php:696,712`; `src/Tui/Picker/SubagentLivePickerController.php:426`), and `SubagentLiveMainReturn::returnToMain()` is what pushes `$state->transcript` when leaving live view (`src/Tui/Runtime/SubagentLiveMainReturn.php:36`). The toggle bypasses that ownership check.

## Expected behavior

Ctrl+O toggles preview expansion for whichever transcript is currently displayed: the child transcript while live view is active, the main transcript otherwise. Live-view state must not change.

## Notes

- Keep this a small bug fix; do not add new view modes or restructure live-view ownership.
- Reported by the user on 2026-09-10. Whether any other always-push-main listener shares this defect is unverified.

## Acceptance criteria
- Ctrl+O during an active agents-live child view re-renders the child transcript with preview expansion toggled; the main transcript is not shown.
- Ctrl+O with no active live view keeps current behavior (main transcript, expansion toggled).
- Live-view state (active, selected, child transcript/seq, activity) is unchanged by the toggle.
- Deterministic regression coverage at the lowest correct layer (virtual TUI/live-view test), with no sleeps or timing-window assertions.
- castor check passes and the TUI lane requirements for touched TUI runtime code are met.

## Workflow metadata
Status: DONE
Branch: task/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/490
PR Status: merged
Started: 2026-09-10T23:00:30+00:00
Completed: 2026-09-10T23:24:31+00:00

## Work log
- Created: 2026-09-10T22:58:43+00:00

## Task workflow update - 2026-09-10T23:00:30+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview.

## Task workflow update - 2026-09-10T23:01:06+00:00
- Ownership: owner=main; fork_run=none; revision=9f559f20b; scope=Ctrl+O preview toggle in agents-live view (PreviewExpansionInputListener); outcome=assigned; commit=none

## Task workflow update - 2026-09-10T23:03:33+00:00
- Validation: castor test --filter=PreviewExpansionInputListenerTest -> OK (7 tests, 56 assertions, 0.7s); Regression proof: with PreviewExpansionInputListener reverted to HEAD (fix patch saved and re-applied), the new case ctrlOTogglesLiveViewChildTranscriptWithoutSwappingToMain FAILED and rendered MAIN_TRANSCRIPT_SHOULD_NOT_RENDER in place of the child transcript.; castor deptrac -> violations=0, errors=0, uncovered=2700; castor phpstan -> errors=0, file_errors=0; castor cs-check -> files_fixed=0
- Ownership: owner=main; fork_run=none; revision=9f559f20b; scope=Ctrl+O preview toggle in agents-live view (PreviewExpansionInputListener); outcome=completed; commit=f7cbd1433

## Task workflow update - 2026-09-10T23:12:36+00:00
- Reviewer: role=reviewer; artifact=agent_f3d782ccf1f02000; revision=f7cbd1433; scope=specification fidelity + correctness of PreviewExpansionInputListener live-view branch and its virtual regression test; verdict=APPROVE WITH SUGGESTIONS (no CRITICAL/BUG/SEC findings)
- Reviewer verified subagentLiveView->active is the same ownership flag used by TickPollListener/SubmitListener/FooterStateSegmentProvider/CompactHeaderRegistrar, that entry pushes childTranscript and only SubagentLiveMainReturn::returnToMain() pushes the parent transcript after active flips false, and that the new case fails on the pre-fix listener. No unresolved blockers.
- Reviewer NTH follow-up (out of scope, not fixed here): ImagePasteInputListener.php:209 and :265 unconditionally push $state->transcript and are reachable while subagentLiveView->active, so Ctrl+V system notices can swap the screen to the main transcript in the same way. Candidate for a separate task with the same guard.

## Task workflow update - 2026-09-10T23:14:40+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (118.0s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview/var/reports/qa-20260910-231242-2694-eb84c736.
- Session/run: 38.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-10T23:14:42+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview/var/reports/qa-20260910-231242-2694-eb84c736.
- Session/run: 38.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-10T23:14:45+00:00
- castor check passed (118.0s).
- Pushed task/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview to origin.
- Created PR: <url>
- Session/run: 38.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-10T23:14:45+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (118.0s).
- Pushed task/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/490

## Task workflow update - 2026-09-10T23:24:31+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview: ide_close_project returned isError.
- Merged task/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Listener/PreviewExpansionInputListener.php       |  12 +++++++++---
 tests/Tui/Listener/PreviewExpansionInputListenerTest.php | 110 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 2 files changed, 119 insertions(+), 3 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-10-fix-agents-live-view-showing-the-main-transcript-after-ctrl-o-preview.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #490 merged on GitHub at 2026-09-10T23:24:00Z (merge commit bc013bac37f283c3d94fdc14280aa00b31034629). Post-merge castor check follows in the integration checkout.

## Task workflow update - 2026-09-10T23:26:29+00:00
- Validation: Post-merge castor check in integration checkout: quality: ok (147.8s). QA report var/reports/qa-20260910-232436-7380-b18f2667.; All 10 lanes OK: deptrac, test (4939 tests, 20745 assertions), test:controller-replay (11/175), test:tui (9/64), test:llm-real (5/30), phpstan (errors=0), dead-code (errors=0), cs-check, docs:validate, catalog:version-check.; QA artifact integrity: ok (10 lane logs). QA run leak check: ok (no processes or tmux sessions owned by the run). llama-proxy cache guard: ok (entries 397 -> 397).; Integrated revision: merge commit bc013bac3 (PR #490) merged locally; integration checkout git status clean at cdc276ed7.; Task worktree unregistered from git worktree list and its directory absent; merged code present at src/Tui/Listener/PreviewExpansionInputListener.php:47.; Note: JetBrains project close degraded (ide_close_project isError) during DONE cleanup; filesystem worktree removal succeeded.
- Post-merge validation complete. PR #490 merged, integration gate passed, worktree removed, integration checkout clean.
