# Reduce hardware cursor jumps during transcript rendering

## Goal
User reports the hardware cursor visibly jumping and blinking outside the editor during streaming, typing, and transcript additions. Research vendor Symfony TUI and Hatfield's rendering path, then implement a focused fix or experimental prototype. User explicitly requests no questions and permits a prototype without tests or review while the visual root cause remains uncertain.

Research at main 20e041cea: Symfony Tui::processRender passes composed lines to ScreenWriter. InteractiveMode installs Hatfield's DeferredCursorCommitScreenWriter alias before constructing Tui. Full, differential, and deletion rendering end DEC synchronized output before positionHardwareCursor restores the editor cursor. Terminal::write flushes each write. This exposes the last painted row as a frame boundary while the hardware cursor remains visible. Cursor position/style and cursor visibility then arrive as separate flushed writes. Hatfield repeats the cursor commit next event-loop turn for overheight frames to address an earlier partial-presentation defect.

Scope: preserve the hardware cursor, existing focus and cursor shape, differential rendering, native scrollback, and the deferred overheight workaround. Prototype changes only at the existing app-owned ScreenWriter. No vendor edits, new settings, renderer framework, ticker delays, or picker repaint optimizations. Related but separate tasks: 2026-09-04-upstream-overheight-screenwriter-cursor-fix and 2026-09-06-stabilize-picker-frames-and-reduce-full-repaints.

Record raw output evidence before and after. Byte ordering can establish the presentation gap, but cannot prove that every real terminal's visible blinking is fixed. Leave IN-PROGRESS with that limitation. No PHPUnit additions/runs, independent review, PR, or full gate in this prototype pass.

## Acceptance criteria
- Document the vendor and Hatfield render/cursor paths and distinguish proven output ordering from terminal-specific visual uncertainty.
- Prototype preventing the hardware cursor from becoming visible at intermediate repaint positions. Restore final cursor position and visibility before releasing synchronized output.
- Keep the deferred next-turn overheight commit and preserve reset/shutdown cancellation and existing rendering semantics.
- Capture before/after output evidence using a disposable Castor-run probe. Leave tests and review for the next phase; do not claim visual validation or full completion.

## Workflow metadata
Status: DONE
Branch: task/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/475
PR Status: merged
Started: 2026-09-06T23:34:32+00:00
Completed: 2026-09-07T02:15:49+00:00

## Work log
- Created: 2026-09-06T23:34:13+00:00

## Task workflow update - 2026-09-06T23:34:32+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering.

## Task workflow update - 2026-09-06T23:35:03+00:00
- Ownership: owner=main; fork_run=none; revision=20e041cea; scope=research and prototype frame/cursor output ordering in existing app-owned ScreenWriter; outcome=assigned; commit=none
- Main owns this cohesive writer change. No independent implementation slice would justify a fork. Testing skill and tests/AGENTS.md read. This pass stops at a prototype with raw-output investigation and focused static checks, not tests or review.

## Task workflow update - 2026-09-06T23:44:42+00:00
- Validation: castor --castor-file=var/tmp/cursor-probe.castor.php cursor-probe --label=before and --label=after: captured 12 steps for each writer, including streaming, simultaneous typing, growth, deletion, reset, and overheight/deferred output.; castor --castor-file=var/tmp/cursor-probe.castor.php cursor-compare: final ScreenBuffer text unchanged in all captured steps. All 8 app-owned repaint cases now release synchronization last. Cursor-only, identical, hidden-cursor-only, and deferred output remain byte-identical. Vendor output remains byte-identical.; castor phpstan --path=src/Tui/Terminal/DeferredCursorCommitScreenWriter.php: errors=0, file_errors=0.; castor cs-check: passed, files_fixed=0. No castor check, PHPUnit, tmux, live LLM, or independent review in this prototype pass.
- Summary: Research found a concrete output-ordering gap shared by vendor ScreenWriter and Hatfield's copy. Implemented a one-file prototype that hides the hardware cursor during painting and releases synchronized output only after restoring final cursor position and visibility. No renderer replacement, vendor edit, new setting, ticker delay, or changes to the deferred overheight workaround.
- Render path: src/Tui/Application/InteractiveMode.php:193 installs the app-owned writer alias before new Tui. TickPollListener.php:103 polls runtime events and :133-136 applies transcript changes. Symfony Tui.php:227-257 serializes ticks and requests a render when the widget revision changes. Tui.php:470-478 renders the widget tree into ScreenWriter. Input follows Tui.php:483-502. Editor/EditorRenderer.php:231 embeds a cursor marker only for the focused editor; it does not move the terminal cursor directly.
- Root-cause evidence: vendor/symfony/tui/Render/ScreenWriter.php:246, :321, and :416 release DEC mode 2026 before restoring the editor cursor. Terminal/Terminal.php:156-159 performs fwrite plus fflush for every write; positionHardwareCursor then writes movement/style and showCursor separately. The captured stream-only frame ends synchronization on the assistant row, then moves down two rows to the editor. This is a proven presentation gap, not proof of the entire terminal-specific visual symptom.
- Prototype scope: fullRender, handleDeletedLines, and differentialRender now begin with synchronized output plus cursor hide, then repaint, position/show or hide the cursor, and end synchronization. Cursor-only and identical-frame output are unchanged. Existing overheight next-turn cursor commit bytes are unchanged. No cursor-shape caching or redundant-commit suppression was added because those would broaden this experiment.
- Disposable investigation artifacts in worktree var/tmp/: cursor-probe.castor.php, cursor-probe.php, cursor-before.json, cursor-after.json. Probe compares vendor and app-owned writer output through Symfony VirtualTerminal and ScreenBuffer. No terminal emulator presentation was measured, no live Hatfield workers or PHAR were touched, and no PHPUnit tests were added or run.
- Remaining uncertainty: real-terminal blink phase, behavior through multiplexers, and the previous overheight partial-presentation workaround need visual evaluation before treating this as a finished fix. Raw frame order and virtual final-screen equality are not sufficient completion proof. Keep IN-PROGRESS. Tests, independent review, and CODE-REVIEW gate remain deferred.
- Skill defect noted, not edited: .agents/skills/testing/SKILL.md documents `castor phpstan [path]`, but this checkout requires `castor phpstan --path=...`. The initial positional invocation failed at argument parsing; the corrected scoped command passed.

## Task workflow update - 2026-09-06T23:45:06+00:00
- Summary: Prototype committed as 834953c35 in the task worktree. Tracked working tree is clean. Task remains IN-PROGRESS pending real-terminal evaluation and later regression tests/review; no push or PR.
- Ownership: owner=main; fork_run=none; revision=20e041cea; scope=research and prototype frame/cursor output ordering in existing app-owned ScreenWriter; outcome=completed; commit=834953c35

## Task workflow update - 2026-09-07T01:22:45+00:00
- User reports the synchronized-frame prototype looks solid in manual use and requests a temporary experiment without the deferred cursor commit. This supersedes the preserve-defer constraint for this experiment only. Keep frame-ordering changes, remove deferred scheduling/lifecycle code, leave uncommitted, no tests or review.
- Ownership: owner=main; fork_run=none; revision=834953c35; scope=temporary removal of deferred cursor commit for manual comparison; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T01:23:57+00:00
- Validation: Disposable Castor cursor probe --label=no-defer: hatfield deferred output is empty; repaint output still restores cursor before ending synchronized output.; castor phpstan --path=src/Tui/Terminal/DeferredCursorCommitScreenWriter.php: errors=0, file_errors=0. No tests, review, or PHAR rebuild run.
- Summary: Temporary no-defer variant is ready in the same task worktree, uncommitted atop 834953c35. Removed deferred scheduling, watcher state, cancellation calls, and EventLoop import. Kept synchronized cursor restoration intact. No rename or test changes for this manual experiment; existing deferred-specific tests intentionally still describe the committed baseline.
- Ownership: owner=main; fork_run=none; revision=834953c35; scope=temporary removal of deferred cursor commit for manual comparison; outcome=completed; commit=none

## Task workflow update - 2026-09-07T01:31:21+00:00
- Finalized scope supersedes prototype-only instructions: user reports no-defer version is rock solid and authorizes minimal regression tests, permanent removal of the deferred workaround, independent review, and CODE-REVIEW transition. Update GitHub #460 for later upstream work; do not upstream in this task.
- Ownership: owner=main; fork_run=none; revision=834953c35 plus no-defer working diff; scope=finalize no-defer writer naming and minimal output-ordering regressions; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T01:44:52+00:00
- Validation: castor test --filter=SynchronizedCursorScreenWriter: 8 tests, 20 assertions passed, PHPUnit 0.395s. Includes 5 frame shapes plus focus loss, drift pin, alias installer.; castor cs-check: passed, files_fixed=0.; Manual user testing of both original synchronized prototype and no-defer variant repeatedly reported stable cursor and no partial rendering; terminal versions unspecified. No universal-emulator claim.
- Summary: Final no-defer implementation committed as ec590f39b. Renamed writer/installer to SynchronizedCursorScreenWriter, removed superseded deferred tests, retained upstream drift guard and alias wiring proof, added minimal data-driven ANSI frame-order regression and focus-loss regression. Independent reviewer approved with non-blocking suggestions.
- Ownership: owner=main; fork_run=none; revision=834953c35 plus no-defer working diff; scope=finalize no-defer writer naming and minimal output-ordering regressions; outcome=completed; commit=ec590f39b
- Review: role=reviewer; artifact=agent_b2d4a08f52ce52e5; revision=ec590f39b; base=20e041cea; scope=specification fidelity, writer/error ordering, aliases, minimal regression adequacy; outcome=APPROVE WITH SUGGESTIONS; blockers=none. Active Hatfield procedures confirmed in follow-up. Reviewer initially used raw php -l in error; excluded from evidence and corrected, no product change needed.
- Deleted-test mapping: deferred-repeat and cancellation cases described the removed workaround, not a remaining contract. Full/delta/deletion and overheight ANSI-order cases now prove immediate restoration before the one sync release; focus loss proves hidden visibility. No new timer or process test.
- Related picker task is paused pending this task landing and already records that its deferred-specific changes/tests must be removed rather than reintroduced after integration.

## Task workflow update - 2026-09-07T01:47:28+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (133.5s).
- Pushed task/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering to origin.
- branch 'task/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering' set up to track 'origin/task/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering'.
- Created PR: https://github.com/ineersa/agent-core/pull/475
- Summary: Submitted ec590f39b after independent approval and minimal focused tests. Run transition-owned full QA gate; keep upstream issue #460 open.

## Task workflow update - 2026-09-07T01:48:26+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/475
- Updated PR Status: open
- Validation: move_task CODE-REVIEW full castor check: passed in 133.5s. Reports: var/reports/qa-20260907-014511-2118-aff023ca.; Worktree clean after transition and issue update.
- Summary: CODE-REVIEW complete at ec590f39b. PR #475 created, full deterministic castor check passed in 133.5s. Updated GitHub #460 title/body to the corrected synchronized cursor ordering and no-defer evidence, keeping upstream follow-up open. Removed accidental shell-wrapper garbage from the old issue body as part of its rewrite.

## Task workflow update - 2026-09-07T01:52:30+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User requests merging origin/main to resolve PR conflict and checking docs/architecture references. Performance investigation of previous gate found tests complete at 76.5s; dead-code finished about 46s later. Main owns merge and minimal documentation update.

## Task workflow update - 2026-09-07T01:52:57+00:00
- Ownership: owner=main; fork_run=none; revision=ec590f39b; scope=merge origin/main a8d6e6c8c, resolve obsolete defer-test modify/delete conflict, check docs/architecture cursor references; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T01:58:30+00:00
- Validation: castor test --filter=SynchronizedCursorScreenWriter: 8 tests / 20 assertions, 0.400s, passed.; castor docs:validate and git diff checks passed.
- Summary: Merged origin/main a8d6e6c8c at 6f8add188. Only conflict was modify/delete of obsolete deferred-writer tests; retained deletion. Preserved incoming session refresh work. Added current writer ordering/alias/upstream removal explanation to docs/tui-architecture.md and architecture/context-and-projection.md; diagrams unchanged.
- Ownership: owner=main; fork_run=none; revision=ec590f39b; scope=merge origin/main a8d6e6c8c, resolve obsolete defer-test modify/delete conflict, check docs/architecture cursor references; outcome=completed; commit=6f8add188
- Prior gate timing investigation from saved artifacts, no rerun: unit 76.543s, TUI 33.179s, controller replay 32.211s, live 18.666s. Dead-code log finalized 01:47:24 UTC versus last test 01:46:38 UTC, about 46s later. PHPStan and dead-code resultCache.php lastFullAnalysisTime both fall within the run, unlike warm main cache. Exact cache invalidation reason not recorded. No case exceeded 10s; max session-switch 8.269s, new cursor cases total 0.005281s.

## Task workflow update - 2026-09-07T01:58:47+00:00
- Summary: Prior reviewer re-reviewed merge/docs revision 6f8add188 and approved with non-blocking suggestion. Merge preserves both reviewed cursor code and incoming session-refresh code byte-for-byte. Issue #460 link already verified by main during prior update; no blocker.
- Review: role=reviewer; artifact=agent_b2d4a08f52ce52e5; revision=6f8add188; scope=merge resolution, parent-diff integrity, docs accuracy/links, specification fidelity; outcome=APPROVE WITH SUGGESTIONS; blockers=none.

## Task workflow update - 2026-09-07T02:00:36+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (83.5s).
- Pushed task/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering to origin.
- branch 'task/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering' set up to track 'origin/task/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering'.
- PR already exists: https://github.com/ineersa/agent-core/pull/475
- Summary: Resubmit 6f8add188 after origin/main merge, obsolete timer-test conflict resolution, documentation updates, focused tests, and independent re-review. Existing PR #475 remains open.

## Task workflow update - 2026-09-07T02:03:05+00:00
- Validation: Reports var/reports/qa-20260907-015910-7269-d219307d: 4797 cases across all lanes, zero >10s. Maximum TuiJourneyE2eTest 9.442s.; Test lanes: unit 72.233s, controller replay 30.663s, TUI 31.305s, live 18.172s. Dead-code finished about 25s after lane start rather than about 122s on previous full-analysis run.
- Summary: Merged revision 6f8add188 pushed to PR #475, now MERGEABLE. Transition-owned check passed in 83.5s, back to expected range. Worktree clean.
- Performance conclusion: previous 133.5s gate was limited by dead-code full analysis, not slow cursor tests. Cached metadata dates full PHPStan/dead-code analysis to previous gate; latest normal gate took 83.5s without QA configuration changes. Exact prior cache invalidation trigger is not logged.

## Task workflow update - 2026-09-07T02:15:49+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering: ide_close_project returned isError.
- Merged task/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering into integration checkout.
- Merge made by the 'ort' strategy.
 architecture/context-and-projection.md                                                                                              |   7 ++++++
 docs/tui-architecture.md                                                                                                            |  17 +++++++++++++
 src/Tui/Application/InteractiveMode.php                                                                                             |   4 ++--
 src/Tui/Setup/SetupScreen.php                                                                                                       |   4 ++--
 src/Tui/Terminal/DeferredCursorCommitScreenWriterAliasInstaller.php                                                                 |  35 ---------------------------
 src/Tui/Terminal/{DeferredCursorCommitScreenWriter.php => SynchronizedCursorScreenWriter.php}                                       |  69 +++++++++++------------------------------------------
 src/Tui/Terminal/SynchronizedCursorScreenWriterAliasInstaller.php                                                                   |  35 +++++++++++++++++++++++++++
 tests/Tui/Terminal/DeferredCursorCommitScreenWriterTest.php                                                                         | 109 ------------------------------------------------------------------------------------
 tests/Tui/Terminal/{DeferredCursorCommitScreenWriterAliasInstallerTest.php => SynchronizedCursorScreenWriterAliasInstallerTest.php} |  10 ++++----
 tests/Tui/Terminal/SynchronizedCursorScreenWriterTest.php                                                                           |  83 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tools/phpstan/DeadCode/HatfieldDeadCodeUsageProvider.php                                                                            |   4 ++--
 11 files changed, 167 insertions(+), 210 deletions(-)
 delete mode 100644 src/Tui/Terminal/DeferredCursorCommitScreenWriterAliasInstaller.php
 rename src/Tui/Terminal/{DeferredCursorCommitScreenWriter.php => SynchronizedCursorScreenWriter.php} (89%)
 create mode 100644 src/Tui/Terminal/SynchronizedCursorScreenWriterAliasInstaller.php
 delete mode 100644 tests/Tui/Terminal/DeferredCursorCommitScreenWriterTest.php
 rename tests/Tui/Terminal/{DeferredCursorCommitScreenWriterAliasInstallerTest.php => SynchronizedCursorScreenWriterAliasInstallerTest.php} (58%)
 create mode 100644 tests/Tui/Terminal/SynchronizedCursorScreenWriterTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-06-reduce-hardware-cursor-jumps-during-transcript-rendering.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User merged PR #475; GitHub confirms MERGED at 7620c42fb47281012da8546d6b08e01be676f0c5. Integrate task branch, remove worktree, then run required post-merge gate.

## Task workflow update - 2026-09-07T02:18:39+00:00
- Updated PR Status: merged
- Validation: Post-merge reports: /home/ineersa/projects/agent-core/var/reports/qa-20260907-021557-11766-774d0a79. All 10 lanes passed; artifact integrity, owned-resource leak check, cache guard passed.; 4797 cases across all test lanes, zero >10s; maximum TuiJourneyE2eTest 8.700s.; Unit lane 78.4s; dead-code 118.8s was critical path. Summary quality time 302.5s is aggregate lane time, not wall duration.; Initial bash tool returned unclean while original QA process was still active. Did not rerun or signal it. Waited on original shell PID via pidfd and recovered complete log plus exit status 0 from .hatfield/tmp/bg/266fa71848016036.log/.status. Early worker-list candidate was still running in original gate; final leak assertion passed.
- Summary: DONE. Integration revision c7fbddc90 contains merged PR #475. Required post-merge LLM_MODE=true castor check passed with original shell exit 0. Integration checkout clean; task worktree removed.
- Worktree removal succeeded. JetBrains close reported degraded isError during task transition; no filesystem cleanup blocker remained.
