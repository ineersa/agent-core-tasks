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
Status: ARCHIVE
Branch: task/2026-08-17-fix-subagent-live-idle-child-spinner
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-17-fix-subagent-live-idle-child-spinner
Fork run: 5a61de07-7269-51b1-ae71-4da329f9ef30
PR URL: https://github.com/ineersa/agent-core/pull/456
PR Status: merged
Started: 2026-09-02T21:57:02+00:00
Completed: 2026-09-03T02:56:18+00:00

## Work log
- Created: 2026-08-17T23:56:35.111Z

## Task workflow update - 2026-08-17T23:58:20.984Z
- User confirmed: message text is fine; the loader widget itself must swap to its static idle indicator. Verified ChatScreen::syncWorkingSlot (main checkout lines 720-745): empty message -> setFinishedIndicator('●') + 'idle' + stop(); non-empty -> spinner start(). So the widget lifecycle is already correct; only the live-view branch keeps the message non-empty when both parent and child are idle.
- Confirmed fix site: TickPollListener live-active branch (~lines 210-228) — when parent idle/terminal AND child idle (no WaitingHuman/Cancelling), resolve the working message to '' so the existing syncWorkingSlot static path renders, instead of the literal 'Child agent idle' string driving the spinner. All other states keep the spinner. Single decision point, no widget change.

## Task workflow update - 2026-09-02T21:56:09+00:00
- Summary: Additional user report from 2026-09-02: during a resumed reviewer run, the main view correctly showed the child as running and the live child view worked. After returning to the main view with Ctrl+\, the `/agents-live` picker showed the same child as completed. Reopening the picker several times kept showing completed while the resumed child was still running. This suggests the picker seeds or refreshes from a stale pre-resume child-status snapshot rather than the current resumed-run state, and may share the same live-state synchronization fault as the idle-spinner bug.
- Observed flow: resume a completed reviewer child; main view shows running; child live view is usable; press Ctrl+\ to return; open `/agents-live`; picker reports completed; repeated picker opens remain completed while the child run is active.
- Investigation requirement: trace resumed-child status updates into `SubagentLiveCatalog`, `SubagentLiveView`, and picker seeding/refresh. Fix the stale snapshot at its source rather than overriding the picker label.
- Acceptance addition: while a resumed child is active, the picker must show its current active status after leaving live view and on repeated reopen; it must switch to completed only after the resumed run actually completes.
- Proof addition: add deterministic lowest-layer coverage for completed child → resume/running update → leave live view → open/reopen picker, alongside the existing idle-versus-active spinner proof.

## Task workflow update - 2026-09-02T21:57:02+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-17-fix-subagent-live-idle-child-spinner.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-17-fix-subagent-live-idle-child-spinner.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-17-fix-subagent-live-idle-child-spinner.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-17-fix-subagent-live-idle-child-spinner.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-17-fix-subagent-live-idle-child-spinner.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-17-fix-subagent-live-idle-child-spinner.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-17-fix-subagent-live-idle-child-spinner/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-17-fix-subagent-live-idle-child-spinner.

## Task workflow update - 2026-09-02T22:00:39+00:00
- Summary: Routing pass found two related state faults. First, `TickPollListener` maps a fully idle live view to the non-empty text `Child agent idle`, which starts `LoaderWidget`; the existing empty-message path already renders the static two-line idle indicator. Second, `SubagentLiveCatalog` allows a changed-task running snapshot to reopen a completed artifact on resume, but a late terminal snapshot from the prior task can overwrite that reopened row because terminal updates are accepted without checking the task generation. The picker correctly rebuilds from the catalog on each open, so repeated opens faithfully show the catalog's stale completed state.
- Ownership: owner=main; fork_run=none; revision=e0d40a394; scope=fix idle live-view loader state and prevent prior-task progress snapshots from overwriting a resumed child catalog row; outcome=assigned; commit=none
- Data shape: `SubagentLiveCatalog` is the artifact-indexed authoritative collection of immutable `SubagentLiveChildDTO` rows. `taskSummary` identifies the current child invocation on a reused artifact/run. Status updates for another task generation are stale after a resume rebind and must not replace the current row.
- Validation plan: deterministic `TickPollListener` working-slot render coverage for idle and active states; catalog and virtual picker coverage for completed task A → running task B → late completed task A → picker close/reopen; focused `castor test`, `castor test:tui`, `castor deptrac`, `castor phpstan`, and `castor cs-check`. Full `castor check` is reserved for task-to-pr.

## Task workflow update - 2026-09-02T22:00:50+00:00
- Summary: Ownership reconsidered after user feedback. A fork is worthwhile because the slice combines two linked lifecycle-state bugs in unfamiliar TUI runtime code and requires focused virtual and terminal-lane validation. Main completed the routing pass and will review the fork's diff and evidence.
- Ownership: owner=main; fork_run=none; revision=e0d40a394; scope=fix idle live-view loader state and prevent prior-task progress snapshots from overwriting a resumed child catalog row; outcome=blocked; commit=none
- Ownership: owner=fork; fork_run=pending; revision=e0d40a394; scope=implement both linked subagent live-state fixes with deterministic virtual proof and focused Castor validation; outcome=assigned; commit=none

## Task workflow update - 2026-09-02T22:10:42+00:00
- Recorded fork run: 6d1e6289-47ab-536b-9453-987691a3ad66
- Ownership: owner=fork; fork_run=6d1e6289-47ab-536b-9453-987691a3ad66; revision=e0d40a394; scope=implement both linked subagent live-state fixes with deterministic virtual proof and focused Castor validation; outcome=completed; commit=c774998c384524a7c3e816ea49f98810fed6d197

## Task workflow update - 2026-09-02T22:12:01+00:00
- Summary: Main review found that the picker symptom comes from stale per-child live-view cache state, not a late parent progress snapshot. `SubagentLiveViewState::$childCaches` is keyed by a reused `agentRunId` and restores the prior invocation's terminal activity. The next tick reconciles that stale local terminal state into the reopened catalog row through `applyChildStatus`, so the picker then reads completed. The first fork's cross-task catalog guard is broader than the demonstrated bug and needs correction.
- Ownership: owner=fork; fork_run=6d1e6289-47ab-536b-9453-987691a3ad66; revision=c774998c384524a7c3e816ea49f98810fed6d197; scope=first implementation superseded after review identified stale child-cache generation as the picker root cause; outcome=blocked; commit=c774998c384524a7c3e816ea49f98810fed6d197
- Ownership: owner=fork; fork_run=pending; revision=c774998c384524a7c3e816ea49f98810fed6d197; scope=correct resumed-child cache generation handling, remove speculative catalog behavior, retain idle-spinner fix, and add end-to-end virtual state regression; outcome=assigned; commit=none

## Task workflow update - 2026-09-02T22:19:41+00:00
- Recorded fork run: 5a61de07-7269-51b1-ae71-4da329f9ef30
- Ownership: owner=fork; fork_run=5a61de07-7269-51b1-ae71-4da329f9ef30; revision=c774998c384524a7c3e816ea49f98810fed6d197; scope=correct resumed-child cache generation handling, remove speculative catalog behavior, retain idle-spinner fix, and add end-to-end virtual state regression; outcome=completed; commit=a24472e1f4eccd7618352fc589d8f47544553c12

## Task workflow update - 2026-09-02T22:20:52+00:00
- Validation: `castor test --filter='SubagentLiveViewStateTest|SubagentLiveCatalogTest|TickPollListenerSubagentLiveTest|SubagentLivePickerControllerTest'` passed: 36 tests, 188 assertions, PHPUnit time 1.048s.; `castor test:tui` passed: 8 tests, 59 assertions.; `castor deptrac` passed with 0 violations.; `castor phpstan` passed with 0 errors.; `castor cs-check` passed with no files changed.; JetBrains diagnostics found no problems in changed production and test files.; Worktree is clean and `git diff --check` passed.
- Summary: Implemented both fixes. Idle child live views now use the existing static `● idle` working slot instead of starting the loader. Resumed child cache entries now carry `taskSummary` as invocation identity; when `agent_resume` reuses a run ID for a new task, the TUI keeps transcript/sequence continuity but resets stale terminal activity and transient queued/replay state from the prior invocation. This prevents the next tick from marking the reopened catalog row completed, so repeated picker opens remain running until the current task completes.
- Review: role=main; revision=a24472e1f4eccd7618352fc589d8f47544553c12; scope=idle loader lifecycle, resumed invocation cache identity, catalog reconciliation, picker reopen regression, and focused validation; verdict=accepted

## Task workflow update - 2026-09-02T23:04:40+00:00
- Summary: Reviewer approved with suggestions at `a24472e1f`, but main is applying three small review corrections before transition: preserve the existing `Working... | Child agent idle` text when only the parent is active, map a new cache generation's Failed/Cancelled status exactly instead of collapsing it to Completed, and remove an unreachable compatibility fallback from the in-memory cache shape.
- Review: role=reviewer; artifact=agent_9ca9241b4d17fd82; revision=a24472e1f4eccd7618352fc589d8f47544553c12; scope=specification fidelity, idle loader lifecycle, resumed cache generation, picker projection, architecture, and proof quality; verdict=APPROVE WITH SUGGESTIONS
- Ownership: owner=main; fork_run=none; revision=a24472e1f4eccd7618352fc589d8f47544553c12; scope=apply reviewer corrections for message-text fidelity, exact terminal activity mapping, and dead cache fallback removal; outcome=assigned; commit=none

## Task workflow update - 2026-09-02T23:06:08+00:00
- Validation: `castor test --filter='SubagentLiveViewStateTest|TickPollListenerSubagentLiveTest'` passed: 8 tests, 41 assertions, PHPUnit time 0.476s.; `castor cs-check` passed with no files changed.; `git diff --check` passed.
- Summary: Applied reviewer suggestions at `99d3cddbd`: parent-active plus child-idle keeps the existing combined message text, while fully idle still uses the static slot; resumed Failed/Cancelled catalog states now retain their exact activity; removed the unreachable cache-shape fallback.
- Ownership: owner=main; fork_run=none; revision=a24472e1f4eccd7618352fc589d8f47544553c12; scope=apply reviewer corrections for message-text fidelity, exact terminal activity mapping, and dead cache fallback removal; outcome=completed; commit=99d3cddbd

## Task workflow update - 2026-09-02T23:08:34+00:00
- Validation: Reviewer reran focused coverage at final revision: 37 tests, 190 assertions, 1.0s, all passed.; Reviewer confirmed JetBrains diagnostics clean, `git diff --check` clean, and worktree clean.
- Summary: Reviewer re-approved final revision `99d3cddbd` with no blocking findings. Aggregate behavior matches the task: only the fully idle child live view changes to the static indicator; existing active-state text and spinner behavior remain intact; resumed cache generations retain exact lifecycle state.
- Review: role=reviewer; artifact=agent_9ca9241b4d17fd82; revision=99d3cddbd; scope=final specification fidelity, working-message truth table, exact resumed terminal mapping, cache shape, and proof quality; verdict=APPROVE

## Task workflow update - 2026-09-02T23:09:32+00:00
- Validation: `castor test --filter='SubagentLiveViewStateTest|SubagentLiveCatalogTest|TickPollListenerSubagentLiveTest|SubagentLivePickerControllerTest'` passed: 37 tests, 190 assertions, PHPUnit time 1.004s.; `castor test:tui` passed: 8 tests, 59 assertions in 23.173s.; `castor deptrac` passed with 0 violations and 0 errors.; `castor phpstan` passed with 0 errors.; `castor cs-check` passed with no files changed.; `git status --short` returned no changes and `git diff --check` passed.
- Summary: Final task-to-pr focused validation passed at `99d3cddbd`. No unresolved blockers remain before the CODE-REVIEW transition gate.

## Task workflow update - 2026-09-02T23:11:54+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (123.3s).
- Pushed task/2026-08-17-fix-subagent-live-idle-child-spinner to origin.
- branch 'task/2026-08-17-fix-subagent-live-idle-child-spinner' set up to track 'origin/task/2026-08-17-fix-subagent-live-idle-child-spinner'.
- Created PR: https://github.com/ineersa/agent-core/pull/456
- Validation: Final focused virtual tests passed: 37 tests, 190 assertions, PHPUnit time 1.004s.; `castor test:tui` passed: 8 tests, 59 assertions in 23.173s.; `castor deptrac` passed with 0 violations and 0 errors.; `castor phpstan` passed with 0 errors.; `castor cs-check` passed with no files changed.; Independent reviewer verdict: APPROVE at `99d3cddbd`.; Worktree clean and `git diff --check` passed.
- Summary: Fixed two linked subagent live-state bugs. Fully idle child live views now use the existing static `● idle` slot instead of a spinner. Live-view caches now carry task-generation identity so `agent_resume` cannot restore the prior invocation's terminal activity and incorrectly mark the resumed child completed in `/agents-live`. Existing active-state messages and spinners remain unchanged.

## Task workflow update - 2026-09-03T00:10:33+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User live-tested PR #456. The idle indicator is correct, but returning with Ctrl+\ and opening the live-agent picker now corrupts the TUI layout. Reopening investigation at the real terminal interaction boundary before changing code.

## Task workflow update - 2026-09-03T00:11:09+00:00
- Summary: User live validation found a terminal regression after the idle fix: Ctrl+\ returns from the child correctly, but opening the live-agent picker after that corrupts the TUI. Treating the live terminal result as authoritative over the passing virtual regression. Investigation will reproduce the exact live-view → Ctrl+\ → picker sequence and trace overlay/render state rather than guessing from the status-cache fix.
- Ownership: owner=main; fork_run=none; revision=99d3cddbd; scope=reproduce and fix TUI corruption in live-view → Ctrl+\ return → live-agent picker sequence, preserving the confirmed idle indicator and resumed-status fixes; outcome=assigned; commit=none

## Task workflow update - 2026-09-03T01:43:46+00:00
- Summary: Separated the newly observed partial-frame defect from PR #456. User confirmed the idle indicator works. The malformed frame clears on any later keypress, matching tracked task `2026-08-22-fix-tui-persistent-partial-frame-until-keypress`; evidence and a picker-round-trip proof requirement were added there. PR #456 remains scoped to idle-state rendering and resumed-child status synchronization, with no further code changes.
- Ownership: owner=main; fork_run=none; revision=99d3cddbd; scope=reproduce and fix TUI corruption in live-view → Ctrl+\ return → live-agent picker sequence, preserving the confirmed idle indicator and resumed-status fixes; outcome=blocked; commit=none
- Routing decision: the picker symptom is a non-input render-completion failure that clears on keypress and matches TODO task `2026-08-22-fix-tui-persistent-partial-frame-until-keypress`. No speculative redraw workaround will be added to PR #456.

## Task workflow update - 2026-09-03T01:45:21+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (86.1s).
- Pushed task/2026-08-17-fix-subagent-live-idle-child-spinner to origin.
- branch 'task/2026-08-17-fix-subagent-live-idle-child-spinner' set up to track 'origin/task/2026-08-17-fix-subagent-live-idle-child-spinner'.
- PR already exists: https://github.com/ineersa/agent-core/pull/456
- Validation: Revision remains reviewer-approved `99d3cddbd`.; Worktree is clean.; No temporary reproduction code remains.; CODE-REVIEW transition runs the deterministic `castor check` gate.
- Summary: Returned PR #456 to CODE-REVIEW without code changes. Live validation confirmed the idle-state fix. The separate frame that remains malformed until a keypress is now tracked with concrete picker evidence under `2026-08-22-fix-tui-persistent-partial-frame-until-keypress`.

## Task workflow update - 2026-09-03T02:56:18+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-17-fix-subagent-live-idle-child-spinner: ide_close_project returned isError.
- Merged task/2026-08-17-fix-subagent-live-idle-child-spinner into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Listener/TickPollListener.php                   |   4 ++-
 src/Tui/Picker/SubagentLivePickerController.php         |   3 +-
 src/Tui/Runtime/SubagentLiveViewState.php               |  37 +++++++++++++++++++------
 tests/Tui/Listener/TickPollListenerSubagentLiveTest.php | 243 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++-
 tests/Tui/Runtime/SubagentLiveViewStateTest.php         |  55 ++++++++++++++++++++++++++++++++++--
 5 files changed, 327 insertions(+), 15 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-17-fix-subagent-live-idle-child-spinner.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-17-fix-subagent-live-idle-child-spinner.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub reports PR #456 state MERGED.; Merged PR URL: https://github.com/ineersa/agent-core/pull/456
- Summary: PR #456 merged on GitHub at 2026-09-03T02:55:49Z with merge commit 6b0e967c535c90d3f9783e39a390269bb6d11cc2. Moving the approved task to DONE and integrating the branch into main.

## Task workflow update - 2026-09-03T03:04:01+00:00
- Validation: `LLM_MODE=true castor check` passed all 10 lanes in 162.4s after commit `16ddf4ba0`: 4693 tests, 19195 assertions; 6 controller replay tests; 8 TUI tests; 5 llm-real tests; deptrac, phpstan, dead-code, cs-check, docs, and catalog checks passed.; QA artifact integrity, worker leak check, cache cleanup, and llama-proxy cache guard passed.; `doctrine:migrations:version 'DoctrineMigrations\Version20260617141000' --delete` removed the obsolete row from `var/test/app_test.sqlite`.; `doctrine:migrations:status` now reports `Executed Unavailable: 0`.; Task worktree was removed and integration checkout is clean.
- Summary: Post-merge integration validation initially exposed a test constructor drift after PR #457 changed `SessionEventsExportService`. Updated the merged regression test to use the shared `SessionEventsExportServiceFactory` in commit `16ddf4ba0`; the subsequent full gate passed. Also removed stale migration metadata for deleted migration `DoctrineMigrations\Version20260617141000` from the persistent `var/test/app_test.sqlite`; migration status now reports `Executed Unavailable: 0`.

## Task workflow update - 2026-09-06T15:40:32+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
