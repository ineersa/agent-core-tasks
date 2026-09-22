# Stabilize picker frames and reduce unnecessary full repaints

## Goal
User reports next-turn cursor commit improves normal TUI rendering but persistent partial frames still occur around pickers. Pickers also visibly force full repaints. Improve existing picker rendering without redesigning UX or adding settings.

Initial research: PickerOverlay mount/close both force render; historical removal caused stale-row regressions, so do not blanket-remove force. SessionPickerController rebuilds list labels on every selection. SubagentLivePickerController feedback and SessionPickerController confirmation updates force rendering. Some close/select paths request force twice. DeferredCursorCommitScreenWriter currently schedules only for overheight frames and reset cancels pending callback; inspect hidden-cursor picker and overheight-to-short transitions, callback freshness, and teardown before changing policy.

Sequence: reproduce at the lowest correct layer; remove demonstrated redundant invalidation/list rebuilding while preserving selection visuals; fix cursor commit lifecycle only where reproduction proves a gap. Retain required structural cleanup. Use existing Symfony widget/context/render facilities. No new renderer framework, periodic redraws, arbitrary delays, or Datadog probes. No full Castor gate during implementation. Avoid overwriting the live session PHAR.

Scout evidence: agent_cfac7cf6a94a4f79, initial read-only picker lifecycle research. Entry points: src/Tui/Picker/PickerOverlay.php, SessionPickerController.php, SubagentLivePickerController.php, src/Tui/Terminal/DeferredCursorCommitScreenWriter.php, Symfony Tui requestRender/processRender.

## Acceptance criteria
- Picker navigation and feedback avoid unnecessary full-screen resets; preserve current labels, focus, actions and theme.
- Open/close and changing-height picker transitions leave no stale rows or stranded cursor; preserve native scrollback and editor/footer behavior.
- Pending deferred cursor commits cannot apply obsolete geometry or cursor visibility after newer frames, reset, or teardown.
- Add focused deterministic virtual/writer regressions and one minimal terminal proof when physical presentation requires it; state remaining live-only uncertainty honestly.
- Run focused Castor validation; no speculative settings or broad test refactors.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/477
PR Status: merged
Started: 2026-09-06T23:32:15+00:00
Completed: 2026-09-07T16:49:36+00:00

## Work log
- Created: 2026-09-06T23:28:34+00:00

## Task workflow update - 2026-09-06T23:32:15+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints.

## Task workflow update - 2026-09-06T23:33:46+00:00
- Summary: Main routing complete. Data shape: last rendered frame carries row count plus nullable cursor target; one cancellable deferred commit belongs to that frame. Picker list owns focus and normally has no cursor marker. Existing scheduler excludes null targets and reset cancels pending work. Investigate that gap without speculative viewport rewrite. Retain historical full-clear safeguards unless reproduction proves removal safe.
- Research: role=scout; artifact=agent_cfac7cf6a94a4f79; revision=20e041cea; scope=picker invalidation lifecycle; outcome=completed. Main confirmed shared mount/close force, session selection setItems churn, feedback force, and deferred commit null-cursor early return.
- Ownership: owner=fork; fork_run=pending; revision=20e041cea; scope=picker invalidation and cursor-commit lifecycle with focused terminal proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-06T23:49:20+00:00
- Summary: First implementation d7f5293f8 covers null-cursor overheight deferred commits, session native selected styling, and in-place differential requests. Main review found test proof weaknesses: force-reset assertion inspected state after helper forcibly repainted, E2E sampled scrollback and sent Down then Escape without asserting navigation; empty catch copied from legacy tests. These must be corrected before accepting. No claim yet of physical GNOME/Kitty symptom elimination.
- Ownership: owner=fork; fork_run=artifact agent_6a2a1ae766953dd4; revision=20e041cea; scope=picker invalidation and cursor lifecycle; outcome=completed; commit=d7f5293f8
- Ownership: owner=fork; fork_run=pending; revision=d7f5293f8; scope=correct deterministic picker proof and remove redundant style/reset additions; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T00:05:40+00:00
- Summary: Main accepted production delta and mutation-sensitive no-clear proof. Finishing two small test corrections under main ownership; focused checks only.
- Ownership: owner=fork; fork_run=artifact agent_27b3d76cfc213afe; revision=d7f5293f8; scope=correct deterministic picker proof and remove redundant style/reset additions; outcome=completed; commit=40217e7a8
- Ownership: owner=main; fork_run=none; revision=40217e7a8; scope=apply output delta to prior virtual screen and remove legacy welcome fallback/manual directory setup in new test; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T00:11:16+00:00
- Validation: Fork: 44 focused tests / 189 assertions passed; PHPStan, Deptrac, dead-code, style passed. Corrected focused subset 8/39 passed.; Mutation proof: restoring force on confirm made no-clear test fail as expected.; Main final writer/navigation subset: 8 tests / 25 assertions passed.; Final minimal tmux: castor test:tui --filter=TuiResumePickerOverlayE2eTest exit0, 1 test / 8 assertions, PHPUnit4.883s; waits for index3, current-pane close and clean exit. Log var/reports/picker-castor-exit.log.; Final cs-check and git diff --check passed; task worktree clean.; Earlier parent shell calls exited unclean despite green JUnit; added explicit graceful pane-exit wait and captured final Castor exit0, no cause claimed beyond observed result.
- Summary: Implementation complete at ee11dd3dc. Overheight picker frames without a cursor marker now receive a deferred hide-cursor commit; session arrow selection uses native accent style without list rebuilds; in-place feedback/confirmation avoids forced clears. Mount/close and main/child structural force safeguards retained. GNOME/Kitty symptom elimination still needs user manual confirmation; tmux and writer proof do not establish all emulator presentation behavior. No full gate or reviewer run in task-start.
- Research: role=scout; artifact=agent_cfac7cf6a94a4f79; revision=40217e7a8; scope=post-test unclean invocation diagnosis; outcome=completed; exact failure root cause unproven.
- Ownership: owner=main; fork_run=none; revision=40217e7a8; scope=current-frame proof and deterministic test teardown; outcome=completed; commit=ee11dd3dc

## Task workflow update - 2026-09-07T01:35:04+00:00
- Summary: User manual testing rejects the current revision as a fix for the reported picker symptoms: pickers still flash, repaint the transcript, and leave partially rendered state. Earlier focused checks establish only narrower contracts, not acceptance. Paused at user request pending 2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering landing. No further picker edits or tests now.
- Dependency inspected read-only: cursor task prototype 834953c35 restores cursor position/visibility before releasing synchronized output. Latest task notes report the no-defer variant is rock solid in manual use and is being finalized as SynchronizedCursorScreenWriter. Its worktree has active uncommitted edits; do not copy or modify them.
- Integration plan after cursor task lands: merge landed main, remove superseded deferred-specific picker changes/tests rather than reintroducing the removed workaround, preserve useful picker invalidation/style improvements, and re-evaluate picker open/navigation/close flashing and partial presentation on the new writer.
- Ownership: owner=main; fork_run=none; revision=ee11dd3dc; scope=picker symptom reassessment after failed manual validation; outcome=blocked; commit=none

## Task workflow update - 2026-09-07T02:25:58+00:00
- Summary: User resumes task after cursor fix #475 landed. Partial rendering on picker close is resolved; remaining symptom is transcript-wide flashing/repaint. User authorizes rolling back prior picker experiment and merging main, then investigating whether a minimal safe repaint reduction is possible. Supersedes preserve-prior-picker-improvements/deferred-commit requirements.
- Ownership: owner=main; fork_run=none; revision=ee11dd3dc; scope=revert prior 3-commit experiment, merge origin/main, reproduce picker full repaint on synchronized writer and implement only demonstrated safe reduction; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T02:36:59+00:00
- Validation: Red/green reproduction: new physical-viewport-mode in-process case failed on main because opening emitted ESC[2J/ESC[3J plus all transcript sentinels; passed after dropping forced resets.; Final castor test --filter='PickerOverlayTest|TuiHistoryPickerOverlayVirtualTest|TuiResumeSessionVirtualTest|SynchronizedCursorScreenWriter': 23 tests, 129 assertions, 0.807s, passed. New two-case regression drives Down/Escape across repeated 8/1/5-item overlays with short and 60-line transcript; asserts no transcript replay/clear and correct retained screen/editor.; castor phpstan --path=src/Tui/Picker/PickerOverlay.php: passed. cs-check and git diff --check passed.; Supplementary throwaway Castor tmux probe: 65-line canonical answer, /model → /history → /model open/down/Escape/editor input all passed; visible transcript tail retained, no stale picker rows. Snapshots var/tmp/tui-e2e-picker-smoke-86872a7c66f68a1f. Not sole automated proof and not an emulator-wide flash claim.; castor clean:cleanup:workers:list: no stale candidates. Task worktree clean. No full gate, push, PR, or review in task-start.
- Summary: Resumed on landed synchronized writer. Reverted previous experiment non-destructively at 883c8d2ef, merged origin/main at dd5d26542 with identical tree to origin/main, then committed minimal picker fix fe2f70589. Shared PickerOverlay no longer forces ScreenWriter reset on mount/close. It requests normal differential output and leaves genuine geometry fallbacks to writer. Only 2 production call changes plus explanatory comment and a regression in existing PickerOverlayTest.
- Ownership: owner=main; fork_run=none; revision=ee11dd3dc; scope=revert prior 3-commit experiment, merge origin/main, reproduce picker full repaint on synchronized writer and implement only demonstrated safe reduction; outcome=completed; commit=fe2f70589
- Remaining scope/limits: file-rewind extension has its own forced-reset PickerOverlay, unchanged in this narrow candidate. Session delete/rename and subagent feedback retain explicit force calls; actual session/history/model changes can still repaint. User manual comparison of common picker open/close is next before broadening.
- Validation tooling observations: testing skill documents positional phpstan path, but command requires --path. Scoped test-file PHPStan flags existing project dynamic PHPUnit calls; normal PHPStan excludes tests and CS fixer requires dynamic style, so retained repository style and used normal production scope. Throwaway probe initially loaded project DI into Castor PHAR and hit incompatible class versions; corrected to separate source PHP subprocess via Castor, not a product defect.

## Task workflow update - 2026-09-07T03:25:14+00:00
- Summary: User confirms /model, /history, /resume transitions work nicely at fe2f70589. Extends scope to /agents-live flashing, duplicate rewind picker lifecycle, and stuck one-shot history/rewind notices. User finalized notice lifetime: clear on next submitted prompt or command, not typing or timers. Native Symfony lists already own every inspected picker; remove duplicate wrapper/reset policy rather than invent a rendering framework.
- Ownership: owner=main; fork_run=none; revision=fe2f70589; scope=remaining agents-live force resets and removal of rewind custom overlay wrapper using native Symfony widgets with focused render proof; outcome=assigned; commit=none
- Ownership: owner=fork; fork_run=pending; revision=fe2f70589; scope=one-shot history and rewind notices cleared on next nonempty submission while preserving persistent statuses and existing reasoning lifetime; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T03:43:49+00:00
- Validation: castor test --filter=testEmptyPickerAndFeedbackRetainTranscriptFrame failed before reset removal with ESC[2J/ESC[3J and retained transcript replay. After removing the three resets, agents-live and rewind focused paths passed 4 cases/30 assertions.; Intermediate wider focused Castor subset passed 49 tests/284 assertions; scoped PHPStan and Deptrac passed. Final test consolidation and formatting pending.
- Summary: Transient-notice fork committed fa11cde46. Parent review found persistent same-text writes retained transient membership and new test duplicated existing setup. Parent now owns corrections: reuse setStatus, clear membership before equality shortcut, remove unrequested empty-string rejection, merge submission tests into existing reasoning notice suite using actual Tui::handleInput rather than reflection dispatch. Rewind custom PickerOverlay removed in favor of native ContainerWidget composition. Agents-live empty catalog, feedback, and last-dismiss render paths reproduced full transcript clear; removed those three forced resets, preserving actual child-view transition reset.
- Ownership: owner=fork; fork_run=artifact agent_e37b410d8f55ff32; revision=fe2f70589; scope=transient notices; outcome=completed; commit=fa11cde46
- Ownership: owner=main; fork_run=none; revision=fa11cde46; scope=parent corrections to transient lifetime/test proof plus remaining native picker render work; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T03:52:18+00:00
- Validation: Final focused Castor suite: 70 tests, 362 assertions, PHPUnit 5.583s; includes shared overlay short/overheight physical-viewport cases, agents-live empty/rejection/last-dismiss, rewind native open/up/Escape/editor focus, session native accent navigation/confirm cancel/rename, real editor submission for notices, unchanged reasoning and OM paths.; Scoped PHPStan passed for src/Tui, extension-api/src/Tui, file-rewind/src; Deptrac 0 violations/errors; docs:validate passed; final cs-check and git diff --check passed.; castor clean:cleanup:workers:list: no stale candidates. No full castor check, review, push or PR in task-start. User real-terminal comparison of the follow-up remains pending.
- Summary: Completed follow-up at 64c010a23, clean worktree. History/rewind one-shot notices now clear on next nonempty submitted prompt or command, before routing; typing/empty submission and persistent statuses are unaffected. Fixed same-text persistent-write lifetime during parent review and removed unsupported empty-text validation. Deleted rewind's duplicate PickerOverlay; controller composes native Symfony ContainerWidget/TextWidget/SelectListWidget through existing extension bridge. Agents-live empty catalog/feedback/last-dismiss no longer clear transcript. Session picker uses Symfony ::selected style instead of rebuilding items on arrows and no longer forces resets for rename/deletion confirmation feedback. Genuine main/child transcript switches retain reset. No broad widget framework or settings added.
- Ownership: owner=main; fork_run=none; revision=fa11cde46; scope=parent corrections to transient lifetime/test proof plus remaining native picker render work; outcome=completed; commit=64c010a23
- Proof mapping: removed fork-created SubmitListenerTransientStatusClearTest and redundant empty-history visibility test. Existing SubmitListenerReasoningNoticeClearTest now drives actual Tui::handleInput for empty submit, typing, /history, next command and prompt. ChatScreenTest proves persistent same-text takeover. TuiFileRewindPickerExtensionVirtualTest proves actual missing-session caller uses transient mechanism.
- Proof mapping: deleted two implementation-specific SessionPickerControllerTest label-accent cases. TuiSessionPickerDeleteVirtualTest now checks one native selected marker and magenta style while moving Down/Up, retained screen during confirmation cancel and editor command insertion. Session labels no longer embed selected styling.
- Public contract addition limited to setTransientStatus on existing TuiExtensionContextInterface for the finalized one-shot notice behavior; existing setStatus remains persistent. Rewind stays extension-owned by architecture but no duplicate picker wrapper/render code remains. Shared host PickerOverlay is lifecycle glue around native Symfony widgets, not a custom renderer.
- origin/main advanced independently during implementation; no merge performed in this phase. Refresh branch against latest main before task-to-pr/full gate.

## Task workflow update - 2026-09-07T16:07:47+00:00
- Validation: 47 focused tests / 207 assertions passed, including writer, picker, notice, ChatScreen, rewind, jbcontext warning and OM tests; cs-check and docs:validate passed.; Initial targeted run found missing newly merged jbcontext class due to stale Composer autoload. Regenerated with composer dump-autoload --no-scripts; targeted suite then passed.
- Summary: Per user request pushed main through 635873fa6 to origin/main and merged it into picker branch at a89f7d6b5. Deferred overheight cursor commit now present alongside synchronized restoration in picker worktree. Resolved three additive conflicts by preserving both transient-status API and incoming extension-warning API, bridge methods, and documentation. Working tree clean; no status transition or picker PR.
- Ownership: owner=main; fork_run=none; revision=64c010a23; scope=merge pushed main including deferred cursor restoration into picker branch; outcome=completed; commit=a89f7d6b5

## Task workflow update - 2026-09-07T16:16:32+00:00
- Validation: castor test --filter='QaSessionEnvSanitizationTest|QaStandaloneTestCacheIsolationTest|CastorCommandRewriterTest|CastorLlmModeToolCallHookTest': 32 tests / 129 assertions passed, 0.591s PHPUnit; no explicit LLM_MODE prefix, output had no progress/colors or Xdebug warning.; Regression executes PHP children through both QA prefixes and asserts effective Xdebug mode off by default and debug/coverage when explicitly requested.; castor cs-check and git diff --check passed. No full gate or status transition; worktree clean.
- Summary: User-approved Castor follow-up committed as 295ca9667 in picker worktree. Shared QA home/env prefix now defaults XDEBUG_MODE=off for QA subprocesses including test migrations; explicit XDEBUG_MODE values remain honored. Removed misleading parent extension_loaded warning. Filtered castor test now honors LLM_MODE compact flags and writes phpunit-filter.junit.xml. Verified castor extension automatically exports LLM_MODE=true with an actual command; previous attribution to omitted env was incorrect, filtered builder was ignoring mode.
- Ownership: owner=main; fork_run=none; revision=a89f7d6b5; scope=user-approved QA Xdebug default and filtered LLM output correction; outcome=completed; commit=295ca9667

## Task workflow update - 2026-09-07T16:21:55+00:00
- Validation: Xdebug off: castor test passed 4865 tests / 20091 assertions, 16.9s runner (PHPUnit 16.765s), 16 workers.; XDEBUG_MODE=develop castor test: same 4865 tests / 20091 assertions passed, 32.9s runner (PHPUnit 32.799s), same 16 workers. Single sequential comparison, approximately 49% less runner time with Xdebug off; not directly comparable to full check's 4-worker concurrent lane.; castor cs-check and git diff --check passed; worktree clean.
- Summary: Ran user-requested full standalone Castor unit suite comparison in picker worktree. First run found merge omission: incoming jbcontext registration test's anonymous TUI context lacked setTransientStatus. Added missing no-op method, committed 6f79bc047; no production change. Worker diagnostic showed no stale QA candidates.

## Task workflow update - 2026-09-07T16:40:44+00:00
- Summary: User smoke tests positive; requested CODE-REVIEW. Independent reviewer APPROVE on 6f79bc047 vs origin/main 635873fa6, no blocking findings. Reviewed full 24-file scope including native picker updates, transient notice lifetime, extension boundaries, QA Xdebug/LLM output and test mapping. Testing prerequisites confirmed. Reusing same-revision full standalone unit evidence; transition owns mandatory full check.
- Reviewer: role=reviewer; artifact=agent_56e7936beae840d4; revision=6f79bc047; scope=full picker task specification fidelity and correctness; outcome=APPROVE. No unresolved blockers. Optional notes only; external interface implementors must add setTransientStatus.
- Ownership: owner=main; fork_run=none; revision=6f79bc047; scope=task-to-pr review and transition; outcome=assigned; commit=6f79bc047

## Task workflow update - 2026-09-07T16:42:55+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (105.3s).
- Pushed task/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints to origin.
- branch 'task/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints' set up to track 'origin/task/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints'.
- Created PR: https://github.com/ineersa/agent-core/pull/477
- Summary: User smoke approved. Independent review approved 6f79bc047, no blockers. Keep deferred cursor workaround from main; picker changes avoid unnecessary resets without changing real transcript-switch rendering.

## Task workflow update - 2026-09-07T16:49:36+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints: ide_close_project returned isError.
- Merged task/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/helpers.php                                                             |  4 +++-
 .castor/phpunit.php                                                             |  8 ++------
 .hatfield/extensions/extension-api/docs/extension-api-tui.md                    |  1 +
 .hatfield/extensions/extension-api/src/Tui/TuiExtensionContextInterface.php     |  8 ++++++++
 .hatfield/extensions/file-rewind/src/FileRewindPickerController.php             | 31 ++++++++++++++++++-------------
 .hatfield/extensions/file-rewind/src/PickerOverlay.php                          | 57 ---------------------------------------------------------
 .hatfield/extensions/jbcontext/tests/JbcontextExtensionRegistrationTest.php     |  4 ++++
 .hatfield/extensions/observational-memory/tests/OmSessionContextCommandTest.php |  8 ++++++++
 docs/tui-architecture.md                                                        |  8 ++++++++
 src/Tui/Listener/SubmitListener.php                                             |  4 ++++
 src/Tui/Picker/HistoryPickerController.php                                      |  2 +-
 src/Tui/Picker/PickerOverlay.php                                                |  8 ++++----
 src/Tui/Picker/SessionPickerController.php                                      | 76 ++++++++++++++++++----------------------------------------------------------
 src/Tui/Picker/SubagentLivePickerController.php                                 |  6 +++---
 src/Tui/Runtime/BridgeTuiExtensionContext.php                                   |  5 +++++
 src/Tui/Screen/ChatScreen.php                                                   | 36 +++++++++++++++++++++++++++++++++++-
 tests/CodingAgent/Castor/QaSessionEnvSanitizationTest.php                       | 19 +++++++++++++++++++
 tests/Tui/Listener/SubmitListenerReasoningNoticeClearTest.php                   | 77 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++-----
 tests/Tui/Picker/PickerOverlayTest.php                                          | 85 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Picker/SessionPickerControllerTest.php                                | 46 ++--------------------------------------------
 tests/Tui/Picker/SubagentLivePickerControllerTest.php                           | 50 ++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Screen/ChatScreenTest.php                                             | 16 ++++++++++++++++
 tests/Tui/Screen/TuiFileRewindPickerExtensionVirtualTest.php                    | 59 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++-
 tests/Tui/Screen/TuiSessionPickerDeleteVirtualTest.php                          | 51 +++++++++++++++++++++++++++++++++++++++++++++++----
 24 files changed, 471 insertions(+), 198 deletions(-)
 delete mode 100644 .hatfield/extensions/file-rewind/src/PickerOverlay.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-06-stabilize-picker-frames-and-reduce-full-repaints.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User merged PR #477; GitHub confirms MERGED at 98c025fc10437c384c9db7c50de8d88a2618f245. Integrate and remove task worktree; post-merge full gate follows.

## Task workflow update - 2026-09-07T16:50:55+00:00
- Updated PR Status: merged
- Validation: LLM_MODE=true castor check passed all 10 lanes plus artifact integrity, leak and cache guards. Reports: var/reports/qa-20260907-164945-19083-8460a3fc.; 4887 JUnit cases, none over 10s; max TuiJourneyE2eTest 7.037s. Unit lane 36.0s, controller replay 31.2s, TUI 22.1s, live smoke 9.3s.; git status --short empty; task worktree absence verified.
- Summary: Post-merge integration validation complete at 794b592c8. PR #477 merged, task DONE, worktree removed, integration Git status clean. Tool reported JetBrains project-close degradation (ide_close_project isError), but worktree and IDEA exclusions were removed.

## Task workflow update - 2026-09-10T22:49:40+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
