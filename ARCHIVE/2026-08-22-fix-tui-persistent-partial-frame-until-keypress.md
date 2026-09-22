# Fix TUI getting stuck on a partial frame until keypress

## Goal
User-reported intermittent TUI rendering failure observed in Hatfield session 43 after a long-running session (~12h29m). The terminal suddenly shows mostly horizontal separator lines with only part of the normal frame/footer visible. The cursor moves into the malformed/partial frame. It resembles a render flicker, but the broken frame remains indefinitely instead of recovering on the next render tick. Pressing any key causes the TUI to repaint and return to normal.

Captured visible footer context at occurrence: model `gpt-5.6-sol`, project `projects/agent-core`, branch `main`, session `43`; usage footer showed `15847.9k/65.7k`, `$15.29`, `↻ 93%`, `42%`, `114.8k/272.0k`, `7.4 t/s`. The display above it consisted predominantly of full-width horizontal rules and one shorter rule, suggesting an interrupted, stale, or incorrectly projected render frame rather than normal transcript content.

Investigate the actual render/invalidation lifecycle before choosing a fix. Correlate session 43 events and TUI/application logs around occurrences where possible. Consider asynchronous runtime events, terminal size/cursor state, partial writes, renderer exceptions/degradation, output interleaving, long-session transcript/layout dimensions, frame scheduling/coalescing, and whether only input events request the repaint that repairs the screen. Do not paper over the issue with periodic blind redraws unless evidence proves that is the smallest owning-boundary fix.

## Acceptance criteria
- Establish a reproducible or evidence-backed root cause for the persistent partial-frame state, including why the screen remains broken indefinitely and why any keypress repairs it.
- Inspect session 43 canonical events/state and relevant structured logs without exposing prompts or session content; identify timestamps/correlation evidence if available.
- Trace TUI render scheduling and invalidation from runtime events, timers, input, resize, widget/layout changes, terminal writes, and exception paths. Identify whether a render is interrupted, omitted, interleaved, or incorrectly considered complete.
- Verify terminal cursor positioning and frame write behavior: a failed/partial render must not leave the cursor in visible transcript/chrome content indefinitely.
- Fix at the smallest owning lifecycle boundary. Do not add arbitrary sleeps, unconditional high-frequency repaint loops, or keypress simulation.
- Ensure non-input state changes that require repaint reliably schedule and complete one, and renderer/write failures either recover on the next safe tick or intentionally force a complete redraw.
- Add regression proof at the lowest correct layer. If the defect depends on real PTY/cursor/frame behavior, use a minimal replay-backed TmuxHarness test rather than service-only mocks.
- Regression proof must demonstrate that the malformed partial frame recovers without user input and that normal typing/input behavior remains unchanged.
- Load and follow `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before investigation, test work, or validation.
- Because this touches TUI runtime/render lifecycle, run focused virtual/controller-replay/Tmux tests as appropriate and complete `castor check` before CODE-REVIEW. If required tmux proof cannot run, leave the task IN-PROGRESS with the blocker.
- Preserve structured log privacy: do not log raw prompts, transcript contents, terminal output, API credentials, or environment values. Include correlation fields such as run_id/session_id, component, and event_type for any new degradation telemetry.
- Never signal, kill, or otherwise modify root-owned workers or processes tagged with `HATFIELD_SESSION_ID` during diagnosis.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-22-fix-tui-persistent-partial-frame-until-keypress
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-22-fix-tui-persistent-partial-frame-until-keypress
Fork run: 68ce843b-0ee1-59b2-8aa4-6f22827204c0
PR URL: https://github.com/ineersa/agent-core/pull/459
PR Status: merged
Started: 2026-09-03T16:27:35+00:00
Completed: 2026-09-04T16:48:31+00:00

## Work log
- Created: 2026-08-22T02:57:09+00:00

## Task workflow update - 2026-08-22T02:59:49+00:00
- Summary: Attached two user-captured ANSI terminal snapshots from session 43 showing consecutive occurrences/states of the persistent partial-frame TUI defect. Preserve and compare both during investigation rather than relying only on the textual report.
- Evidence snapshot 1: `/home/ineersa/projects/agent-core/.hatfield/tmp/tui/snapshots/snapshot-ansi-20260821-225731.ansi` — captured 2026-08-21 22:57:31 local, 10,513 bytes.
- Evidence snapshot 2: `/home/ineersa/projects/agent-core/.hatfield/tmp/tui/snapshots/snapshot-ansi-20260821-225851.ansi` — captured 2026-08-21 22:58:51 local, 9,729 bytes.
- Treat snapshots as potentially session-sensitive terminal evidence: inspect locally, do not paste transcript contents into logs/PR/task comments. Compare ANSI cursor movement, erase/clear sequences, terminal dimensions, visible partial frame, and state before/after keypress recovery.

## Task workflow update - 2026-09-03T01:43:25+00:00
- Summary: Added a second concrete trigger for the same persistent partial-frame symptom. While testing PR #456 at a wide terminal, opening `/agents-live`, entering a child, returning with Ctrl+\, and showing or closing the picker can leave the frame mostly as duplicated horizontal separators and duplicated footer content. The malformed frame remains until any keypress, including Down, immediately causes a correct redraw. State and input still work; the missing step is a completed repaint after the non-input picker transition.
- Evidence from 2026-09-03 live test: after a resumed scout completed, `/agents-live` and the child live view worked. Ctrl+\ returned to the parent. The picker transition then left duplicated footer rows and separator-only content. Pressing any key repaired the screen immediately.
- Investigation addition: include PickerOverlay mount/close, `SubagentLiveMainReturn::returnToMain()`, forced `requestRender(true)`, ScreenWriter scheduling/coalescing, and the difference between picker-triggered renders and the repaint triggered by the next input event. Reproduce at a wide terminal comparable to the supplied capture, not only the existing 120-column virtual case.
- Proof addition: a minimal real-tmux picker round trip should assert that the final frame settles correctly without another keypress. Keep this proof in the partial-frame task because it concerns terminal render completion, not subagent status or idle-state semantics.

## Task workflow update - 2026-09-03T16:05:28+00:00
- Summary: Additional live evidence: after footer elapsed time changed from second precision to minute precision, the persistent partial-frame defect became substantially less frequent in the current session, though not confirmed eliminated. This strengthens the hypothesis that render frequency/timing exposes the defect; the minute footer likely masks or reduces the trigger rather than proving the render path is correct.
- 2026-09-03: User reports the issue is now less prominent after footer updates dropped from every second to every minute. During task-start, compare behavior with a synthetic changing footer versus a stable footer and inspect render scheduling/coalescing; do not treat reduced frequency as resolution.

## Task workflow update - 2026-09-03T16:07:50+00:00
- Summary: Fresh reproduction captured at `.hatfield/tmp/tui/snapshots/snapshot-ansi-20260903-120547.ansi` during an ordinary main-session question/answer with no subagents running. The defect is therefore generic TUI frame corruption, not a subagent-state defect; `/agents-live` remains only one deterministic transition that can expose it. The 224x60 capture shows duplicated bottom structure: four separator rows near the footer, including one truncated-width separator, followed by one footer row.
- 2026-09-03: Scope correction from fresh live reproduction: no subagents were running and no live-agent transition occurred. Investigate shared render/write/viewport behavior around normal assistant/tool completion and footer updates. Preserve `/agents-live` only as a reproduction harness if it remains deterministic; do not root the fix in subagent code without new evidence.
- 2026-09-03: Snapshot evidence retained at `.hatfield/tmp/tui/snapshots/snapshot-ansi-20260903-120547.ansi` (11,113 bytes, 60 captured rows, apparent width 224). Plain structural analysis finds separator rows at lines 17, 56, 57, 58, and 59; line 57 is only 155 columns while adjacent separators are 224, then the footer appears at line 60. This confirms a persistent partial bottom-frame/row-clearing failure.

## Task workflow update - 2026-09-03T16:27:18+00:00
- Summary: User approved a sequential hypothesis-driven implementation. Start with physical viewport/cursor-row accounting by recovering the smallest applicable behavior from the historical Pi-style ScreenWriter. Stop after that isolated change for manual reproduction testing before attempting later candidates.
- 2026-09-03 implementation order agreed with user: P1 physical viewport/cursor-row accounting, reusing the smallest proven part of the historical Pi-style ScreenWriter. P2 write-all handling for partial `fwrite()` only if P1 does not resolve the defect or instrumentation proves a short write. P3 exact-width/autowrap handling. P4 lost or coalesced final render request. P5 DEC synchronized-output behavior, now lower priority because the user's tmux configuration removed most bouncing/flicker. Width-transition theory is ruled out for the fresh reproduction because the user confirms no resize occurred. Keep widget duplication as a cheap diagnostic, not an implementation candidate.
- Implement one candidate at a time. Each candidate needs its own evidence, commit, focused automated proof, and manual worktree checkpoint. Do not combine speculative fixes, periodic repainting, render-twice behavior, sleeps, or keypress simulation.

## Task workflow update - 2026-09-03T16:27:35+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-22-fix-tui-persistent-partial-frame-until-keypress.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-22-fix-tui-persistent-partial-frame-until-keypress.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-22-fix-tui-persistent-partial-frame-until-keypress.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-22-fix-tui-persistent-partial-frame-until-keypress.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-22-fix-tui-persistent-partial-frame-until-keypress.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-22-fix-tui-persistent-partial-frame-until-keypress.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-22-fix-tui-persistent-partial-frame-until-keypress/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-22-fix-tui-persistent-partial-frame-until-keypress.
- Validation: Task-start prerequisites loaded: task workflow router and task-start procedure, implementation ownership, specification fidelity, TUI proof, TUI AGENTS.md, testing skill, and tests/AGENTS.md.
- Summary: Started sequential diagnosis and implementation. First slice is physical viewport/cursor-row accounting, recovered minimally from the historical Pi-style ScreenWriter. Later candidates remain explicitly deferred until manual testing of this slice.

## Task workflow update - 2026-09-03T16:29:33+00:00
- Summary: Assigned the first implementation slice to a fork. The slice is limited to restoring physical viewport and logical cursor-row accounting from the initial historical Pi-style writer experiment, with current-vendor compatibility and deterministic writer-level proof. Partial-write handling and all later hypotheses are excluded.
- Ownership: owner=fork; fork_run=pending; revision=16ddf4ba0; scope=P1 app-owned ScreenWriter with physical viewport/cursor-row accounting only, plus alias activation and focused deterministic proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-03T16:38:52+00:00
- Recorded fork run: 47a7a10e-aa83-51e4-b271-603f2442403c
- Ownership identity update: implementation fork artifact=agent_bea9dd524f5a0ee4; fork_run=47a7a10e-aa83-51e4-b271-603f2442403c; target revision=16ddf4ba0; scope=P1 physical viewport/cursor-row accounting.

## Task workflow update - 2026-09-03T16:41:42+00:00
- Recorded fork run: 47a7a10e-aa83-51e4-b271-603f2442403c
- Validation: Prototype fork ran focused checks before the user requested manual-first iteration: castor test --filter='PiStyleScreenWriter' passed (3 tests, 23 assertions); castor deptrac passed; scoped phpstan passed; cs-check passed.; Parent review confirmed the production diff is limited to the app-owned writer, fail-fast pre-Tui alias installation, and its documentation/tests. No partial-write, autowrap, render-coalescing, synchronized-output, or periodic-redraw changes are present.; Manual root-cause discrimination remains pending. The current finite-terminal viewport tests exercise the implementation but do not reproduce the intermittent live failure against stock Symfony, so they are not being treated as proof that P1 fixes the reported bug.
- Summary: P1 prototype is ready for manual reproduction testing at revision 31974e466. It restores the initial historical Pi-style physical viewport and logical hardware-cursor accounting only. No later hypotheses were included.
- Ownership: owner=fork; fork_run=47a7a10e-aa83-51e4-b271-603f2442403c; revision=16ddf4ba0; scope=P1 app-owned ScreenWriter with physical viewport/cursor-row accounting only; outcome=completed; commit=31974e4663bf0378ea7ae0c78a5ebd62824a0553
- Manual checkpoint: user requested prototype-first iteration and permits deferring proof/polish while testing candidate root causes one by one. Worktree `/home/ineersa/projects/agent-core-worktrees/2026-08-22-fix-tui-persistent-partial-frame-until-keypress` is clean at 31974e466. Wait for live reproduction results before P2.

## Task workflow update - 2026-09-03T16:47:11+00:00
- Summary: Manual live testing reproduced the persistent partial-frame defect with P1 revision 31974e466 active. Physical viewport and logical cursor-row accounting alone do not fix the reported failure. Proceeding to the next isolated candidate, P2 short-write handling, without adding autowrap, render-coalescing, synchronized-output, periodic redraws, sleeps, or keypress simulation.
- Ownership: owner=main; fork_run=none; revision=31974e4663bf0378ea7ae0c78a5ebd62824a0553; scope=P2 terminal frame write-all handling for partial fwrite only, with focused deterministic proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-03T16:48:57+00:00
- Summary: Manual testing reproduced the same persistent partial-frame defect in the P1 worktree at revision 31974e466. Snapshot: `.hatfield/tmp/tui/snapshots/snapshot-ansi-20260903-124606.ansi` (112x60, 7,960 bytes). P1 physical viewport/cursor accounting is rejected as the root-cause fix and will be reverted before P2.
- P1 manual outcome: rejected. User reproduced without a width change while the Pi-style viewport writer was active. Snapshot structural evidence includes a 20-column stale/truncated separator among current 112-column editor/footer rows near the bottom. Proceed sequentially by reverting commit 31974e466, then prototype P2 write-all handling for partial terminal writes only.

## Task workflow update - 2026-09-03T16:50:41+00:00
- Ownership: owner=fork; fork_run=pending; revision=20a076be0; scope=P2 prototype only: guarantee complete STDOUT frame writes while preserving stock Symfony ScreenWriter and terminal behavior; outcome=assigned; commit=none

## Task workflow update - 2026-09-03T16:55:25+00:00
- Recorded fork run: 68ce843b-0ee1-59b2-8aa4-6f22827204c0
- Validation: P2 production paths pass PHP syntax checks, scoped phpstan, deptrac, and cs-check. No automated regression test was added for this manual root-cause checkpoint.; Parent review confirms P2 contains no viewport accounting, autowrap, render-coalescing, synchronized-output, or repeated-render changes.
- Summary: P1 was reverted in commit 20a076be0. P2 prototype is ready at ed1e986a4: stock Symfony ScreenWriter now writes through CompleteWriteTerminal, which loops until the complete frame reaches STDOUT or fails on zero/false progress.
- Ownership identity update: implementation fork artifact=agent_a6d6dae34a419f42; fork_run=68ce843b-0ee1-59b2-8aa4-6f22827204c0; target revision=20a076be0; scope=P2 complete STDOUT frame writes.
- Ownership: owner=fork; fork_run=68ce843b-0ee1-59b2-8aa4-6f22827204c0; revision=20a076be0; scope=P2 prototype only: guarantee complete STDOUT frame writes while preserving stock Symfony ScreenWriter and terminal behavior; outcome=completed; commit=ed1e986a4c8d5e20e54bb9cd43af46c290ee989d
- Manual checkpoint: test revision ed1e986a4 in `/home/ineersa/projects/agent-core-worktrees/2026-08-22-fix-tui-persistent-partial-frame-until-keypress`. If the defect reproduces, capture a new ANSI snapshot before reverting P2 and moving to exact-width/autowrap.

## Task workflow update - 2026-09-03T17:00:46+00:00
- Summary: Manual testing reproduced the defect with P2 complete-STDOUT-write prototype active at ed1e986a4. Snapshot: `.hatfield/tmp/tui/snapshots/snapshot-ansi-20260903-125804.ansi` (112x60, 7,428 bytes). P2 is rejected and will be reverted before P3.
- P2 manual outcome: rejected. Reliable trigger is now simple: ask `hello`, let the assistant stream, then do nothing. Cursor jumps several times and the bottom frame becomes partial or duplicated. It can recover later without input, or immediately on keypress. This adds evidence that successive scheduled streaming paints can repair the bad physical frame, and weakens the theory that only user input requests recovery.
- P2 snapshot structural evidence: current 112-column frame contains a stale 20-column separator at row 52, followed by several full-width 112-column separators and footer/chrome. No synchronized-output markers appear in the captured visible snapshot. Proceed by reverting P2, then isolate P3 exact-width/autowrap behavior.

## Task workflow update - 2026-09-03T17:01:42+00:00
- Ownership: owner=main; fork_run=none; revision=f97702b14; scope=P3 prototype only: remove the guaranteed exact-width separator rows by rendering all ChatScreen separators at terminal width minus one; outcome=assigned; commit=none

## Task workflow update - 2026-09-03T17:02:23+00:00
- Validation: P3 prototype: PHP syntax passed, git diff check passed, and JetBrains diagnostics report no problems in ChatScreen.php. Full test work remains deferred until manual root-cause discrimination.
- Summary: P2 was reverted in f97702b14. P3 exact-width/autowrap prototype is ready at 81ad58a1d. All three ChatScreen separator rows now render at terminal width minus one, leaving stock Symfony ScreenWriter and every other render path unchanged.
- Ownership: owner=main; fork_run=none; revision=f97702b14; scope=P3 prototype only: remove the guaranteed exact-width separator rows by rendering all ChatScreen separators at terminal width minus one; outcome=completed; commit=81ad58a1d
- Manual checkpoint: reproduce with `hello`, allow streaming to finish, then leave input untouched and watch for cursor jumps or malformed bottom chrome. The deliberate one-column separator gap distinguishes this P3 build. Capture a new ANSI snapshot if it fails.

## Task workflow update - 2026-09-03T17:22:17+00:00
- Summary: Manual testing reproduced the defect with P3 exact-width/autowrap prototype active at 81ad58a1d. Snapshot: `.hatfield/tmp/tui/snapshots/snapshot-ansi-20260903-131917.ansi` (112x60, 8,204 bytes). P3 is rejected and will be reverted before P4 render scheduling/coalescing.
- P3 manual outcome: rejected. The fresh snapshot proves the width-minus-one separators were active at rows 53 and 57, yet the malformed bottom frame still includes stale short row 52 and duplicated 112-column rows. Exact-width separator autowrap is not the root cause.
- Ownership: owner=main; fork_run=none; revision=81ad58a1d; scope=P4 prototype only: coalesce rapid ordinary render requests to a bounded frame cadence while preserving latest state and immediate forced renders; outcome=assigned; commit=none

## Task workflow update - 2026-09-03T17:25:06+00:00
- Validation: P4 prototype production files pass PHP syntax, git diff checks, and JetBrains diagnostics. Automated proof remains deferred for manual root-cause discrimination.
- Summary: P3 was reverted in 64a598c76. P4 render-coalescing prototype is ready at 28ac7cf34. Ordinary paints now retain only the latest requested frame and emit at most every 100ms; forced renders bypass the interval. Runtime polling and input cadence remain unchanged.
- Ownership: owner=main; fork_run=none; revision=81ad58a1d; scope=P4 prototype only: coalesce rapid ordinary render requests to a bounded frame cadence while preserving latest state and immediate forced renders; outcome=completed; commit=28ac7cf34
- Manual checkpoint: use the same `hello` streaming reproduction at revision 28ac7cf34 and leave input untouched. This prototype deliberately caps ordinary terminal paints at 10 FPS to make the timing hypothesis easy to distinguish. Capture a new ANSI snapshot if the frame still corrupts.

## Task workflow update - 2026-09-03T17:38:54+00:00
- Summary: P4 changed the symptom but did not eliminate it. At revision 28ac7cf34, snapshot `.hatfield/tmp/tui/snapshots/snapshot-ansi-20260903-133432.ansi` shows the bottom chrome intact, but skill-list row 47 is truncated to `pri`; the hardware cursor remains in that list until input triggers another paint. P4 will be reverted. Next probe bypasses differential rendering by forcing a full repaint for every requested frame.
- P4 manual outcome: rejected as a fix, informative as an exposure probe. Reducing ordinary paints to 10 FPS moved/reduced corruption rather than eliminating it, so render cadence affects where the incomplete frame becomes visible but is not sufficient to restore correctness.
- Next prototype: force every render request through ScreenWriter reset/full repaint. This directly discriminates differential ScreenWriter state/cursor movement from renderer/widget-state corruption. No rate cap or other hypothesis will remain active.

## Task workflow update - 2026-09-03T17:39:37+00:00
- Validation: Full-repaint prototype passes PHP syntax, git diff checks, and JetBrains diagnostics. Automated proof remains deferred during manual root-cause discrimination.
- Summary: P4 reverted in 319ae94c4. Differential-render bypass probe is ready at ac08ea1ae: every render request resets ScreenWriter, so every frame uses the full repaint path.
- Ownership: owner=main; fork_run=none; revision=28ac7cf34; scope=full-repaint diagnostic prototype; outcome=completed; commit=ac08ea1ae
- Manual checkpoint: repeat the same streaming scenario at ac08ea1ae without pressing keys. Report whether any partial row or cursor stranded inside transcript/skills remains. A clean result isolates the defect to differential rendering/cursor bookkeeping; reproduction implicates the renderer/full-frame terminal path.

## Task workflow update - 2026-09-03T20:36:58+00:00
- Summary: Root-cause discrimination phase verdict: the full-repaint control at ac08ea1ae eliminated all persistent partial-frame and stranded-cursor artifacts, while causing expected severe flicker. Therefore renderer/widget state and terminal writes can produce a complete frame; corruption is specific to Symfony ScreenWriter's differential-render path. P1 viewport prototype was insufficient, but the broader differential-render boundary is now proven. Revert the diagnostic and continue with a targeted differential algorithm fix rather than testing unrelated hypotheses.
- Full-repaint manual outcome: clean frames and correct cursor placement, with unacceptable rapid flicker on typing/updates. This is diagnostic success, not a shippable implementation.
- Phase verdict: P1 physical viewport prototype rejected; P2 complete writes rejected; P3 exact-width separators rejected; P4 cadence coalescing changed symptom but did not fix; full repaint eliminated defect. Root-cause boundary is differential ScreenWriter cursor/row/update accounting.
- Next phase should capture the exact differential transition that diverges—especially content-growth/scroll and cursor repositioning—then correct or selectively fall back for that transition. Do not ship full repaint or generic frame throttling.

## Task workflow update - 2026-09-03T20:55:54+00:00
- Summary: Beginning targeted differential-render instrumentation after full-repaint control isolated the fault. User explicitly approved dirty prototypes, vendor edits, and Datadog logging. The probe will emit content-free per-frame ScreenWriter decisions and predicted cursor/viewport state into a Datadog-collected worktree log.
- Instrumentation data shape: one JSON event per ScreenWriter frame containing frame id, chosen render path, terminal dimensions, old/new line counts, changed range, predicted logical hardware cursor before/after, viewport top, movement deltas, growth/shrink counts, buffer bytes, and changed-row visible widths. No rendered text, prompts, tool output, or environment values.
- Probe location is vendor/symfony/tui/Render/ScreenWriter.php and is intentionally dirty/uncommitted because vendor is ignored. Full-repaint prototype has already been reverted in 8e43b566e.

## Task workflow update - 2026-09-03T21:17:34+00:00
- Validation: `castor phar:build` passed; inspected built PHAR and confirmed `tui.screen_writer.probe` instrumentation plus idempotent alias installer are embedded.
- Summary: First instrumentation run produced no probe data because `castor run:agent` launches the packaged PHAR and ignored the direct vendor edit. Corrected by adding a temporary app-owned instrumented ScreenWriter alias and rebuilding the PHAR; probe is now embedded and writes to the worktree Datadog-collected log.
- Session 6 reproduction cannot be analyzed: it ran the non-instrumented artifact, and no `tui-screen-writer-probe.log` was produced. This is a probe setup failure, not evidence about the bug.
- Corrected probe artifact is now ready from the same worktree. A new `castor run:agent` launch will materialize the rebuilt PHAR and emit `/home/ineersa/projects/agent-core-worktrees/2026-08-22-fix-tui-persistent-partial-frame-until-keypress/.hatfield/logs/tui-screen-writer-probe.log`.

## Task workflow update - 2026-09-03T21:19:34+00:00
- Summary: Session 7 probe captured 73 frames: 68 differential, 1 initial full, 4 unchanged. Terminal was 224x60 and rendered content remained below viewport height (48→57 rows; viewport top estimate stayed 0), ruling out scrolling/viewport drift for this reproduction. The strongest divergence is differential updates that repeatedly end on exact-width 224-column rows before a separate relative cursor-position write. This can leave the terminal in wrap-pending state, explaining cursor stranding and why full repaint succeeds.
- Session 7 key frames: frame 18 grew 48→52 and rewrote rows 34–51 (7,674 bytes); frame 29 grew 52→53 and rewrote rows 46–52; frame 61 grew 53→55 and rewrote rows 38–54; frame 73 grew 56→57 and rewrote rows 41–56. Nearly all streaming differential frames rewrote an exact-width 224-column row, then repositioned from the changed row to editor cursor using a separate write.
- P1 viewport bookkeeping was not merely incomplete for this trace: no frame exceeded the 60-row viewport. Next targeted prototype disables terminal autowrap only while emitting a differential frame, then restores it before cursor positioning. This preserves full-width layout and tests the wrap-pending mechanism directly.

## Task workflow update - 2026-09-03T21:22:59+00:00
- Summary: Additional user-observed precursor: immediately before corruption, the visible hardware cursor consistently leaves the editor and jumps into another content row. This matches the session 7 trace: differential paint moves from the editor cursor to the first changed row, writes exact-width rows, then relies on a separate relative move to restore the editor cursor. The visible jump means restoration is delayed or misinterpreted, not merely stale text.
- Root-cause signal strengthened: cursor departure from the editor precedes every visible failure. The autowrap-disabled differential prototype directly tests whether exact-width delayed-wrap state causes the subsequent relative cursor restoration to target the wrong physical row.

## Task workflow update - 2026-09-03T21:24:32+00:00
- Summary: Session 8 reproduced with autowrap disabled during every differential frame, rejecting delayed-wrap as the root cause. Trace remained below viewport (48→57 rows, 224x60). The remaining protocol flaw is now explicit: ScreenWriter ends DEC synchronized output and writes the changed frame, then performs editor-cursor restoration in a second terminal write. The cursor can visibly remain at the changed row between those writes—the exact user-observed precursor.
- Session 8 autowrap outcome: rejected. 61 frames captured (57 differential, 1 full, 3 unchanged); all differential events carried autowrap-disabled marker and corruption still occurred.
- Next targeted prototype makes each differential paint and final editor cursor placement one atomic synchronized-output write. It removes the autowrap experiment and avoids the separate post-frame cursor write.

## Task workflow update - 2026-09-03T22:31:06+00:00
- Summary: Session 9 reproduced despite atomic paint-plus-cursor restoration, rejecting the two-write boundary as the root cause. The trace confirms all 92 differential frames included cursor restoration before synchronized-output end. Remaining distinction from the successful full-repaint control is relative cursor addressing versus a known home anchor.
- Session 9 atomic-cursor outcome: rejected. Cursor jump/corruption returned even though each differential frame included its final editor row/column movement before DEC synchronized output ended.
- Next prototype anchors the initial frame at terminal home and uses absolute CUP addressing for differential paint start and editor cursor restoration while content fits the viewport. This tests relative cursor-position drift without introducing full-frame repaint flicker.

## Task workflow update - 2026-09-03T22:33:42+00:00
- Summary: Session 10 reproduced with absolute addressing only at the start/end of each differential frame. It also exposed an expected prototype flaw: homing the initial frame without clearing left prior terminal content duplicated. This rejects accumulated between-frame cursor drift but does not yet test within-frame row advancement, because changed rows were still traversed with CRLF.
- Session 10: 70 frames (61 differential, 1 initial, 8 unchanged), all below viewport. Cursor jump/corruption persisted with absolute frame start and final editor CUP.
- Next prototype clears once on initial render and uses absolute CUP for every changed row, eliminating CRLF/autowrap row advancement inside differential frames. This targets the remaining place where ScreenWriter infers physical row movement.

## Task workflow update - 2026-09-03T23:39:21+00:00
- Summary: Session 11 reproduced even with absolute CUP addressing for every differential row. The captured frame shows overlapping/concatenated resource rows and bottom chrome, so relative cursor movement, CRLF advancement, autowrap, and split cursor restoration are all ruled out. The remaining difference from the successful full-repaint control is the selected-row diff itself versus repainting the complete row array.
- Session 11: 160 frames (156 differential), content stayed at or below 59 of 60 rows. Every differential row and final cursor used absolute addressing, yet corruption persisted.
- Next discriminating probe keeps synchronized output and absolute addressing but repaints every logical row each frame without clearing the screen. If stable, changed-range selection/state is wrong; if it still corrupts, the required behavior is the full repaint path's screen clear/reset.

## Task workflow update - 2026-09-03T23:52:38+00:00
- Summary: The per-row absolute-addressing/all-row repaint probe failed immediately and produced pervasive malformed output. Repeated CUP+erase for every row is not viable. This does not isolate changed-range selection cleanly because it introduced a radically different terminal protocol.
- Replace the failed probe with a narrower no-clear full-frame write: home once, emit the entire row array contiguously with the same CRLF structure as Symfony fullRender, erase any old trailing rows, and restore the cursor atomically. This keeps the successful full-render protocol and removes only the screen clear.

## Task workflow update - 2026-09-04T02:03:36+00:00
- Summary: Stopped the ScreenWriter mutation sequence after the user's reassessment. Restored the worktree to stock Symfony TUI, removed all uncommitted instrumentation/aliases, deleted probe logging, and rebuilt the clean PHAR. The branch is clean at 8e43b566e.
- Reassessment: P5–P8 changed too many terminal-protocol variables and did not increase confidence. Discarded them rather than preserving speculative code.
- Current evidence retained: full repaint suppresses corruption but flickers; P4 10 FPS coalescing does not fix corruption but uniquely preserves bottom rows and moves the stranded cursor/artifact into the skill list; changing footer cadence from 1s to 1m substantially reduced real-world frequency. These two cadence experiments are the strongest causal signal.
- Next investigation should leave ScreenWriter untouched and instrument Tui request/process scheduling: request source/time, render revision before/after tick, pending duration, processRender start/end, and whether a request arrives during rendering. Then replay stock cadence and P4 cadence against identical runtime events.

## Task workflow update - 2026-09-04T02:09:37+00:00
- Summary: User reproduced the persistent partial-frame/cursor-stranding defect directly in GNOME Terminal without tmux. tmux and `.tmux.conf` are therefore excluded as necessary causes. Investigation remains focused on the application/Symfony TUI render cadence and terminal-write scheduling path.
- Environment discrimination: direct GNOME Terminal reproduction succeeded without tmux. Remove tmux configuration and tmux DEC synchronized-output handling from the active hypothesis set. Testing another terminal emulator can still distinguish GNOME/VTE-specific behavior from a general Symfony TUI defect.

## Task workflow update - 2026-09-04T02:11:46+00:00
- Summary: User reproduced the defect directly in Kitty as well as GNOME Terminal, and previously without tmux. Terminal-emulator-specific behavior, tmux, and `.tmux.conf` are excluded as necessary causes. The common fault is now localized to Hatfield/Symfony TUI rendering or scheduling.
- Cross-terminal discrimination: reproduced in GNOME/VTE and Kitty with no tmux dependency. Stop terminal-specific protocol experiments. Resume from the strongest causal evidence: render cadence changes exposure and corruption location (P4), while reducing footer invalidation frequency substantially reduces incidence.

## Task workflow update - 2026-09-04T02:41:41+00:00
- Summary: New evidence: disabling OM `om-background` status-row updates suppresses one frequent trigger but `/hotkeys` can still break the bottom frame. OM was an exposure source, not root cause. Temporarily disabling PromptEditor synthetic Down/End reduced visible cursor jumping but was rolled back; it is not required for the defect. Shared trigger now appears to be a render that changes the number/layout of rows above the focused editor (status row insertion/removal or large `/hotkeys` transcript insertion), potentially through wrapping/line accounting in differential rendering.
- Restored PromptEditor synthetic Down/End behavior after diagnostic; only OM status-row setStatus remains disabled in current probe build.
- Investigate row-count/layout transitions above the focused editor, especially wrapped semantic widgets and ScreenWriter differential handling when content height changes or exceeds the viewport.

## Task workflow update - 2026-09-04T02:44:05+00:00
- Summary: Refined root-cause candidate: the shared data shape is an ordered logical row list rendered into a finite terminal viewport. `/hotkeys` adds a large semantic transcript block; OM status insertion/removal changes rows above the editor. Stock Symfony ScreenWriter clips over-height content only when scrollOffset > 0. At normal scrollOffset=0 it writes every logical row, lets the terminal physically scroll, but keeps `hardwareCursorRow` in logical-row coordinates. Once logical rows exceed terminal height, physical cursor row and tracked logical row can diverge; later relative differential movement can repaint/restore the wrong rows. This directly matches bottom corruption and keypress repair.
- PromptEditor diagnostic fully rolled back. Current build has normal editor cursor behavior and only OM `setStatus` suppressed.
- Next discriminating prototype should not revisit old cursor escape variants: clip the composed frame to the bottom terminal-height rows before differential rendering, keeping editor/footer visible and keeping ScreenWriter row coordinates inside the physical viewport. Test `/hotkeys` and OM status-row growth at 60 rows, then remove the OM suppression to verify both triggers.

## Task workflow update - 2026-09-04T02:56:12+00:00
- Ownership: owner=main; fork_run=none; revision=8e43b566e; scope=bounded viewport-clipping ScreenWriter prototype with OM status updates re-enabled; outcome=assigned; commit=none

## Task workflow update - 2026-09-04T02:57:43+00:00
- Validation: PHP syntax passed for ViewportClippingScreenWriter, alias installer, and InteractiveMode.; Alias activation smoke resolved Symfony ScreenWriter to app-owned ViewportClippingScreenWriter.; castor phar:build passed all PHAR smoke checks; built artifact contains viewport clipping probe.; git diff --check passed.
- Summary: Viewport-clipping prototype ready. App-owned ScreenWriter now clips every composed frame to the final terminal-height rows before raw-frame comparison and differential rendering. This keeps editor/footer rows and keeps logical row coordinates bounded to the physical viewport. OM `om-background` status updates are re-enabled so both known triggers can be tested.
- Prototype ready for manual discrimination at baseline revision 8e43b566e with uncommitted app-owned ScreenWriter files. Test `/hotkeys`, then a normal stream ending while OM activity appears/disappears.

## Task workflow update - 2026-09-04T03:00:43+00:00
- Summary: Manual viewport-clipping result: `/hotkeys` no longer causes the old bouncing/partial-bottom corruption, but the blunt final-N-row slice visibly drops the beginning of the composed frame and produces an incomplete `/hotkeys` view. This is expected from the diagnostic and is not acceptable as the final implementation. The result strongly implicates overflow transitions where the logical frame exceeds terminal height and the terminal scrolls during differential rendering.
- Viewport-clipping probe changed the failure mode from unstable/bouncing bottom corruption to stable but truncated output. Treat this as positive root-cause discrimination, not a shippable fix.
- Next implementation must preserve the full transcript/native scrollback while keeping ScreenWriter's differential comparison and cursor bookkeeping aligned to the physical viewport. Do not ship array_slice-based frame truncation.

## Task workflow update - 2026-09-04T03:01:55+00:00
- Validation: Structural-repaint classes and InteractiveMode pass PHP syntax checks.; Alias activation resolves Symfony ScreenWriter to StructuralRepaintScreenWriter.; castor phar:build and PHAR smoke checks passed; artifact contains structural-repaint probe.; git diff --check passed.
- Summary: Replaced the truncating viewport-slice probe with a structural-repaint probe. It preserves the entire composed transcript and native scrollback. When the rendered logical row count changes, ScreenWriter clears and fully repaints; content-only frames still use stock differential rendering. OM status updates remain enabled.
- Viewport clipping was intentionally removed after manual result proved overflow involvement but visibly truncated `/hotkeys`. New manual probe should test `/hotkeys` and OM status appear/disappear for stable complete output.

## Task workflow update - 2026-09-04T03:03:37+00:00
- Validation: git status clean after rollback.; castor phar:build passed all smoke checks from clean revision 8e43b566e.
- Summary: Structural full-repaint-on-row-count-change probe failed: `/hotkeys` left only the top/resource area plus a short separator and omitted editor/footer. Rejected. All current prototype and diagnostic changes are now rolled back, including OM suppression; worktree and PHAR are back to clean revision 8e43b566e.
- Ownership: owner=main; fork_run=none; revision=8e43b566e; scope=bounded viewport-clipping/structural-repaint ScreenWriter prototype with OM enabled; outcome=blocked; commit=none
- Stop applying ScreenWriter algorithm guesses. Both bounded clipping and structural clears changed the failure destructively. Next step must capture renderer output row count, cursor-marker row, physical terminal rows, and exact write sequence for the deterministic `/hotkeys` trigger before another mutation.

## Task workflow update - 2026-09-04T03:10:30+00:00
- Research: agent=researcher; artifact=agent_3a3238705e33216f; target=Symfony issue #64941 and repository ScreenWriter history; outcome=completed. Issue requested writer injection but shipped no Symfony patch. Repository history contains the real custom implementation, culminating at 4fea82a5b.

## Task workflow update - 2026-09-04T03:11:21+00:00
- Ownership: owner=main; fork_run=none; revision=8e43b566e; scope=restore the complete final historical PiStyleScreenWriter implementation from commit 4fea82a5b for manual A/B testing, without its separate Terminal decorator; outcome=assigned; commit=none

## Task workflow update - 2026-09-04T03:12:04+00:00
- Validation: All three restored ScreenWriter classes pass PHP syntax.; Early class alias smoke resolves Symfony ScreenWriter to PiStyleScreenWriter.; castor phar:build passed all PHAR smoke checks.; Built PHAR contains final historical scrollAheadRender implementation.; git diff --check passed.
- Summary: Restored the complete final historical Pi-style ScreenWriter from commit 4fea82a5b for manual A/B testing. This is not the earlier P1 subset: it includes append-diff normalization, viewport bookkeeping, guarded growth tail rewrite, native scroll-ahead, stale-aware bottom-to-top absolute-CUP repaint, and print-then-CSI-0K handling for shorter rows. It preserves full logical output and native scrollback. The separate TelemetryTerminal decorator was omitted because it is not part of rendering behavior.
- Historical evidence: cancelled tui-native-symfony-widget-architecture-spike recorded commit ac1cdcada as a successful human TTY baseline with 'no bouncing'; final refinement was 4fea82a5b. Symfony issue #64941 itself contained no implementation, only the injection-seam request and class_alias workaround discussion.

## Task workflow update - 2026-09-04T03:18:55+00:00
- Summary: Final historical writer 4fea82a5b failed `/hotkeys`. Its own telemetry captured the trigger at 224x60 with a 216-row logical frame growing to 227 rows via 6 native scroll-ahead frames, followed by shrink to 226. The visible result retained stale loaded-resource rows below the hotkey block and cut the footer. This points at scroll-ahead's logical-to-physical row mapping and stale-row selection, not width calculation: terminal width stayed 224 and no resize/full-clear occurred.
- Reject final 4fea82a5b custom writer as-is. The newer scroll-ahead refinements are now suspect because the writer skipped/repainted selected shifted rows while the actual viewport retained stale rows. Next A/B should use the earlier 9ce467f3c erase-before-scroll tail-rewrite implementation, which historical manual testing called materially more stable, without scroll-ahead row selection.

## Task workflow update - 2026-09-04T03:19:33+00:00
- Validation: Restored writer classes pass PHP syntax.; Confirmed source and built PHAR contain tailRewriteRender and do not contain tryScrollAheadRender.; castor phar:build passed all smoke checks.; git diff --check passed.
- Summary: Prepared the earlier 9ce467f3c historical writer for the next A/B. It retains Pi viewport bookkeeping, append normalization, and erase-before-scroll tail rewriting, but removes the later guarded scroll-ahead and selective stale-row repaint algorithm that failed `/hotkeys`.
- Ownership handoff within main: final 4fea82a5b writer rejected; now testing earlier 9ce467f3c tail-rewrite-only writer against deterministic `/hotkeys` trigger.

## Task workflow update - 2026-09-04T03:27:22+00:00
- Validation: git status clean after historical-writer rollback.; castor phar:build passed all smoke checks from clean revision 8e43b566e.
- Summary: Earlier 9ce467f3c erase-before-scroll writer also failed `/hotkeys` with the same cut footer. Historical-writer family is rejected. All custom writer files and alias wiring have been removed again; worktree and PHAR are clean at 8e43b566e.
- Ownership: owner=main; fork_run=none; revision=8e43b566e; scope=restore final and earlier historical PiStyleScreenWriter implementations for deterministic `/hotkeys` A/B; outcome=blocked; commit=none
- Key renderer/layout clue: `/hotkeys` virtual tests deliberately use rows=80 with comment 'Tall virtual screen so the full hotkeys catalog stays in the viewport'. Real reproduction is 60 rows while the composed logical frame exceeds 200 rows. The failure is tied to unbounded root layout overflowing RenderContext rows. Investigate how Symfony Renderer/LayoutEngine allocates finite height and why ChatScreen's transcript is not the vertically expandable region before touching ScreenWriter again.

## Task workflow update - 2026-09-04T03:28:55+00:00
- Validation: `castor test --filter=testHotkeysSlashCommandRoutesLocallyAndRendersKeyboardShortcutsTable` failed deterministically at rows=60 because the final ScreenBuffer omitted the top of `/hotkeys` and showed duplicated bottom separators.; Restored diagnostic test edit; git status and git diff --check clean.
- Summary: Deterministic reproduction achieved at the virtual terminal's real 60-row height with stock code. Changing only `TuiVirtualInputTest` from rows=80 to rows=60 made `/hotkeys` fail: the final screen starts mid-table, omits the `Keyboard shortcuts` heading, and shows idle plus duplicate separators/footer at the bottom. This reproduces the user's cut-over shape without timing, tmux, OM, or a custom writer. The diagnostic test edit was restored afterward; worktree remains clean.
- Root-cause boundary is now finite-height rendering of an overheight root frame. Existing test had explicitly hidden it with rows=80. Next implementation should fix root layout/viewport semantics so the transcript occupies the available middle region while editor/footer stay visible; ScreenWriter replacement is no longer the primary path.

## Task workflow update - 2026-09-04T03:30:59+00:00
- Ownership: owner=main; fork_run=none; revision=8e43b566e; scope=behavior-identical stock ScreenWriter copy with content-free frame geometry logging for `/hotkeys`; outcome=assigned; commit=none

## Task workflow update - 2026-09-04T15:22:30+00:00
- Validation: Captured 17 stock-equivalent frames with geometry-only logging. Final frame: terminal=224x60, previous=143, new=216, changed=129–215, cursor marker=212, final tracked hardware row=212.; Final tail geometry: separators at logical rows 210, 211, 213, 214; blank editor row 212; footer content row 215. This matches ChatScreen's intended editor/footer structure.; No changed-row frame followed the user's Down key, yet the visible terminal corrected.
- Summary: Stock-writer geometry capture completed. `/hotkeys` frame 17 grew 143→216 logical rows in a 60-row terminal; the editor/footer tail remained internally correct at rows 210–215 and cursor marker row 212. The later Down key fixed the visible layout without any changed-row frame being emitted after frame 17. This weakens the root-layout hypothesis: the renderer produced correct bottom chrome, while terminal presentation remained stale until input. The 60-row virtual failure is bottom-window cropping, not by itself proof of corrupted renderer output.
- Ownership: owner=main; fork_run=none; revision=8e43b566e; scope=behavior-identical stock ScreenWriter copy with content-free frame geometry logging for `/hotkeys`; outcome=completed; commit=none
- Ownership: owner=main; fork_run=none; revision=8e43b566e; scope=disable DEC synchronized-output framing while retaining stock differential algorithm and geometry logging; outcome=assigned; commit=none

## Task workflow update - 2026-09-04T15:27:33+00:00
- Validation: No-sync PHAR ran stock differential rendering at 224x60; final `/hotkeys` frame still grew 143→216 with correct logical tail geometry and reproduced the defect.
- Summary: No-synchronized-output A/B failed: `/hotkeys` produced the same corruption with all DEC 2026 begin/end markers removed. Synchronized output is not the cause.
- Ownership: owner=main; fork_run=none; revision=8e43b566e; scope=disable DEC synchronized-output framing while retaining stock differential algorithm and geometry logging; outcome=blocked; commit=none
- Next diagnostic data shape: `TerminalCursorState`, separating ScreenWriter's logical hardware row from the finite terminal's physical row and viewport top. Trace each terminal write to locate the first bookkeeping divergence.

## Task workflow update - 2026-09-04T15:31:38+00:00
- Summary: No-sync result confirmed the same corruption. Current evidence now proves the full failure chain: Symfony Renderer emits an unbounded 216-row root frame into a 60-row terminal; ScreenWriter allows terminal scrolling but records `hardwareCursorRow=212` in logical coordinates; the physical cursor is at row 56. Subsequent relative moves are computed from 212, not 56, so terminals clamp them and repaint the wrong physical rows. The Down key triggers another cursor action that happens to redraw/correct the visible bottom.
- The earlier Pi writers tracked a logical viewport but did not repair this exact contract; they still fed overheight roots to terminal writes and failed `/hotkeys`. Correct fix belongs at the finite screen-frame boundary: preserve the full transcript model, but give ScreenWriter only the terminal-height viewport with the editor/footer tail retained. `/hotkeys` top cropping at 60 rows is expected unless a separate scroll model is added; duplicated/cut chrome is not.

## Task workflow update - 2026-09-04T15:33:55+00:00
- Summary: The earlier geometry logger intentionally skipped ScreenWriter's unchanged-frame early return. Therefore the absence of a logged changed frame after Down does not mean no terminal write occurred: unchanged frames still call `positionHardwareCursor()`. Added logging around that path and restored stock DEC synchronized output. Next run will show whether Down recovery is only cursor repositioning against stale logical row state.
- Ownership: owner=main; fork_run=none; revision=8e43b566e; scope=instrument ScreenWriter unchanged-frame early return to trace Down-key recovery; outcome=assigned; commit=none

## Task workflow update - 2026-09-04T15:35:38+00:00
- Validation: Frame 21: changed rows 129–215, 216 total, cursor marker/hardware row 212.; Frame 22 after Down: unchanged_before/unchanged_after, 216 rows, first/last changed=-1, hardware row remained 212; ScreenWriter wrote no content.
- Summary: Down-key recovery captured. Frame 21 wrote the 216-row `/hotkeys` update and ended with tracked cursor row 212. Down produced frame 22 through ScreenWriter's unchanged-frame path: no content repaint, row delta 0, only cursor-column/style/visibility sequences were emitted. The visible layout nevertheless corrected. Therefore Down does not repair application or renderer state. It causes the terminal to present an already-written frame.
- Ownership: owner=main; fork_run=none; revision=8e43b566e; scope=instrument ScreenWriter unchanged-frame early return to trace Down-key recovery; outcome=completed; commit=none

## Task workflow update - 2026-09-04T15:36:10+00:00
- Ownership: owner=main; fork_run=none; revision=8e43b566e; scope=prototype one deferred cursor-only commit after an overheight changed frame, matching the proven Down-key recovery write on the next event-loop turn; outcome=assigned; commit=none

## Task workflow update - 2026-09-04T15:41:55+00:00
- Validation: Manual `/hotkeys` reproduction at 224x60 with 216 logical rows: no persistent partial frame and no keypress needed to repair layout.; Residual: visible cursor movement during the large differential update remains; user described it as less severe and acceptable compared with the broken frame.
- Summary: User confirmed the deferred cursor-commit prototype fixes the persistent broken frame: no broken or stuck content remained during `/hotkeys`. Some transient cursor movement remains, but it is much less severe and does not leave the layout corrupted. Treat the root defect as fixed by yielding one event-loop turn and repeating the final cursor commit after an overheight update.
- Ownership: owner=main; fork_run=none; revision=8e43b566e; scope=prototype one deferred cursor-only commit after an overheight changed frame, matching the proven Down-key recovery write on the next event-loop turn; outcome=completed; commit=none

## Task workflow update - 2026-09-04T16:01:00+00:00
- Validation: `castor test --filter='DeferredCursorCommitScreenWriter'`: 3 tests, 4 assertions passed.; `castor test:tui --filter=testJourneyCoversCoreTuiBehavior`: 1 test, 17 assertions passed in 5.0s.; `castor deptrac`: 0 violations.; `castor phpstan`: 0 errors.; `castor cs-check`: 0 files fixed.; Manual 224x60 `/hotkeys` validation: persistent broken frame eliminated without follow-up input; only transient cursor movement remains.
- Summary: Clean implementation committed as 44507230a. Added app-owned copy of Symfony ScreenWriter pinned to upstream revision 94c5b0ec with one narrow delta: after an overheight changed frame, cancel any older pending commit and repeat the final cursor positioning on the next Revolt event-loop turn. Installed through class_alias because Symfony TUI 8.1 constructs a final ScreenWriter internally. Removed all probes and telemetry. Added unit proof for deferred vs in-viewport behavior, alias installation proof, and a real tmux `/hotkeys` overheight journey assertion.
- Ownership: owner=main; fork_run=none; revision=8e43b566e; scope=productionize deferred cursor commit, remove probes, add lowest-layer and tmux terminal proof; outcome=completed; commit=44507230a

## Task workflow update - 2026-09-04T16:17:04+00:00
- Summary: Reviewer agent_6b60b48e6f55d230 requested changes at 44507230a. Blocking fixes: enroll src/Tui/Terminal in Deptrac; correct tmux proof wording because tmux verifies end-to-end wiring but cannot reproduce emulator presentation latency; then run full castor check. Reviewer also identified deferred callback teardown and moving Symfony 8.1.x-dev drift risks, which will be fixed before re-review.
- Review: role=reviewer; artifact=agent_6b60b48e6f55d230; revision=44507230a; scope=specification fidelity, terminal lifecycle, architecture, test quality; verdict=REQUEST CHANGES
- Ownership: owner=main; fork_run=none; revision=44507230a; scope=address reviewer findings: Deptrac layer, accurate proof claims, deferred teardown, upstream drift guard, exact delta docs; outcome=assigned; commit=none

## Task workflow update - 2026-09-04T16:30:19+00:00
- Validation: `castor test --filter='DeferredCursorCommitScreenWriter|SetupScreen'`: 33 tests, 190 assertions passed.; `castor deptrac`: 0 violations.; `castor phpstan`: 0 errors.; `castor dead-code`: 0 errors.; `castor cs-check`: 0 files fixed.; `git diff --check`: passed.
- Summary: Addressed reviewer findings in a806066b3. TuiTerminal is now covered by Deptrac; alias installation covers both interactive and setup TUI entry points; getState cancels pending deferred callbacks before teardown; a source hash test detects Symfony ScreenWriter drift; dead-code analysis knows the alias-driven vendor contract; tmux proof wording now states its actual packaged integration scope. Replaced raw cursor escape-sequence assertions with Symfony TerminalInterface interaction expectations.
- Ownership: owner=main; fork_run=none; revision=44507230a; scope=address reviewer findings: Deptrac layer, accurate proof claims, deferred teardown, upstream drift guard, exact delta docs; outcome=completed; commit=a806066b3

## Task workflow update - 2026-09-04T16:33:53+00:00
- Validation: `castor test:llm-real`: 5 tests, 30 assertions passed in 9.5s.; `castor cs-check`: 0 files fixed.; `git diff --check`: passed.
- Summary: Re-review at a806066b3 found only the mandatory full gate and one incomplete delta comment outstanding. Updated the comment in f44e60b12. Warmed the live test proxy successfully; ready for the CODE-REVIEW transition's deterministic castor check.
- Review: role=reviewer; artifact=agent_6b60b48e6f55d230; revision=a806066b3; scope=verify round-one blockers and TerminalInterface interaction tests; verdict=REQUEST CHANGES pending mandatory castor check and one delta-doc line
- Ownership: owner=main; fork_run=none; revision=a806066b3; scope=complete ScreenWriter delta inventory; outcome=completed; commit=f44e60b12

## Task workflow update - 2026-09-04T16:34:55+00:00
- Summary: Final reviewer approval at f44e60b12. All code, architecture, lifecycle, proof-scope, and maintenance findings are resolved. Approval is subject only to the mandatory castor check performed by the CODE-REVIEW transition.
- Review: role=reviewer; artifact=agent_6b60b48e6f55d230; revision=f44e60b12; scope=final review of all prior findings; verdict=APPROVE WITH SUGGESTIONS subject to transition castor check

## Task workflow update - 2026-09-04T16:36:56+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (97.0s).
- Pushed task/2026-08-22-fix-tui-persistent-partial-frame-until-keypress to origin.
- branch 'task/2026-08-22-fix-tui-persistent-partial-frame-until-keypress' set up to track 'origin/task/2026-08-22-fix-tui-persistent-partial-frame-until-keypress'.
- Created PR: https://github.com/ineersa/agent-core/pull/459
- Validation: Manual GNOME Terminal and Kitty validation at 224x60: `/hotkeys` no longer leaves a persistent broken frame or requires follow-up input.; Focused unit/setup tests: 33 tests, 190 assertions passed.; Focused TUI journey: 1 test, 17 assertions passed in 5.0s.; Standalone llm-real warmup: 5 tests, 30 assertions passed in 9.5s.; Deptrac, PHPStan, dead-code, CS check, and git diff check passed.; Final reviewer verdict: APPROVE WITH SUGGESTIONS at f44e60b12; only mandatory transition castor check remained.
- Summary: Fixed persistent partial TUI frames by repeating the final cursor commit on the next Revolt event-loop turn after overheight ScreenWriter updates. Added alias installation at both TUI entry points, teardown cancellation, Symfony source drift detection, Deptrac coverage, deterministic writer tests, and packaged TUI integration coverage. Independent reviewer approved revision f44e60b12 subject to this transition gate.

## Task workflow update - 2026-09-04T16:48:31+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-22-fix-tui-persistent-partial-frame-until-keypress: ide_close_project returned isError.
- Merged task/2026-08-22-fix-tui-persistent-partial-frame-until-keypress into integration checkout.
- Merge made by the 'ort' strategy.
 depfile.yaml                                                              |  10 +++
 src/Tui/Application/InteractiveMode.php                                   |   2 +
 src/Tui/Setup/SetupScreen.php                                             |   2 +
 src/Tui/Terminal/DeferredCursorCommitScreenWriter.php                     | 572 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 src/Tui/Terminal/DeferredCursorCommitScreenWriterAliasInstaller.php       |  35 +++++++++
 tests/Tui/E2E/TuiJourneyE2eTest.php                                       |  29 +++++++-
 tests/Tui/Terminal/DeferredCursorCommitScreenWriterAliasInstallerTest.php |  26 +++++++
 tests/Tui/Terminal/DeferredCursorCommitScreenWriterTest.php               |  93 +++++++++++++++++++++++
 tools/phpstan/DeadCode/HatfieldDeadCodeUsageProvider.php                  |   5 ++
 9 files changed, 772 insertions(+), 2 deletions(-)
 create mode 100644 src/Tui/Terminal/DeferredCursorCommitScreenWriter.php
 create mode 100644 src/Tui/Terminal/DeferredCursorCommitScreenWriterAliasInstaller.php
 create mode 100644 tests/Tui/Terminal/DeferredCursorCommitScreenWriterAliasInstallerTest.php
 create mode 100644 tests/Tui/Terminal/DeferredCursorCommitScreenWriterTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-22-fix-tui-persistent-partial-frame-until-keypress.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-22-fix-tui-persistent-partial-frame-until-keypress.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #459 state: MERGED.; Merge commit: 9df8cc00c564a5679e2d5f2c4b142ecb5fceff17.
- Summary: PR #459 merged at 2026-09-04T16:47:33Z with merge commit 9df8cc00c564a5679e2d5f2c4b142ecb5fceff17. The persistent partial-frame defect is fixed by a deferred cursor commit after overheight ScreenWriter updates.

## Task workflow update - 2026-09-04T16:52:40+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/459
- Updated PR Status: merged
- Validation: PR #459 merged at 2026-09-04T16:47:33Z; merge commit 9df8cc00c564a5679e2d5f2c4b142ecb5fceff17.; `LLM_MODE=true castor check`: passed all 10 lanes in 177.1s under QA run qa-20260904-165106-29136-232b56b5.; Unit/integration: 4704 tests, 19214 assertions.; Controller replay: 6 tests, 88 assertions.; TUI: 8 tests, 61 assertions.; LLM real: 5 tests, 30 assertions.; Deptrac, PHPStan, dead-code, CS check, docs validation, and catalog version check passed.; QA leak check and llama-proxy cache guard passed; cache remained 393 entries.; Integration checkout has no uncommitted changes; task worktree removed.
- Summary: Post-merge validation completed. `LLM_MODE=true castor check` passed all 10 lanes on the integration checkout in 177.1s. The task worktree was removed and the integration checkout has no uncommitted changes.

## Task workflow update - 2026-09-06T15:40:32+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
