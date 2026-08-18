# Spike: migrate Hatfield TUI toward first-class Symfony TUI widgets

## Goal
Follow-up to DONE/evaluate-stable-symfony-tui-widget-lifecycle-for-transcript-rendering.md and GitHub issue #243.

The prior investigation was scoped narrowly around transcript per-block render caching/performance and concluded that the RENDER-05 rendered-line cache was sufficient. New motivation: Hatfield's current hybrid architecture wraps most UI regions in `LiveTextWidget` producers that render project-native `TuiWidget` objects to strings, and transcript blocks build detached/offscreen Symfony widget trees. This bypasses Symfony TUI `WidgetContext` for inner MarkdownWidget instances, prevents stylesheet-driven Markdown sub-element theming, and preserves a fragile custom rendering/layout layer on top of Symfony TUI.

Current shape to investigate:
- Real Symfony TUI tree owns outer lifecycle/event loop/layout.
- ChatScreen uses many `LiveTextWidget` wrappers for header/transcript/pending/status/footer/extension regions.
- TranscriptBlockWidget renders `TranscriptBlock[] -> list<string>` and internally creates temporary Symfony widgets per block via standalone Renderer.
- This means large regions are pre-rendered strings rather than first-class Symfony `AbstractWidget` trees.

Goal: perform a proper architecture spike for leaning into Symfony TUI instead of maintaining two rendering systems. Determine whether ChatScreen regions, transcript blocks, extension slots, and Markdown rendering can become first-class Symfony widgets while preserving Hatfield's visual contract, extension API compatibility, and testability.

Suggested investigation areas:
1. Map current `TuiWidget`/`LiveTextWidget` regions to possible Symfony `AbstractWidget` equivalents.
2. Prototype transcript rendering as a real Symfony widget subtree mounted in the live widget tree, with direct `MarkdownWidget` usage for user/assistant/thinking markdown blocks.
3. Verify whether Markdown sub-element stylesheet tokens work naturally when widgets are attached to Symfony `WidgetContext`.
4. Evaluate migration paths that avoid a big-bang rewrite: adapter layer, optional Symfony-widget-producing path, transcript-first migration, or slot-by-slot migration.
5. Identify blockers around scrolling/viewport behavior, dynamic block updates, finalized block caching, streaming tail invalidation, preview expansion, tool-call/result pairing, extension slots, and virtual/TUI E2E testing.
6. Decide whether to file an upstream Symfony TUI feature request only after confirming the limitation remains relevant under proper first-class widget usage.

Non-goals for the spike:
- Do not perform a production migration without an accepted plan.
- Do not open a Symfony TUI issue until the spike decides whether Hatfield's architecture is the real blocker.
- Do not remove existing `TuiWidget` extension API without a compatibility strategy.

## Acceptance criteria
- Architecture report compares current hybrid `TuiWidget` + `LiveTextWidget` + offscreen Renderer model with a first-class Symfony TUI widget-tree model.
- Prototype or proof demonstrates whether MarkdownWidget sub-element theming works when mounted in a real Symfony `WidgetContext`.
- Prototype or proof covers at least transcript markdown blocks and one non-markdown block/tool exchange path, or documents why that is blocked.
- Migration plan proposes incremental phases, test thesis/layers, and rollback strategy; explicitly addresses extension slot compatibility.
- Decision recorded: migrate toward first-class Symfony widgets, keep current hybrid model, or file upstream Symfony TUI request with clarified scope.
- If migration is recommended, follow-up implementation tasks are created/proposed with focused validation strategy (`castor test`, relevant virtual tests, `castor test:tui` only where needed).

## Workflow metadata
Status: CANCELLED
Branch: task/tui-native-symfony-widget-architecture-spike
Worktree: /home/ineersa/projects/agent-core-worktrees/tui-native-symfony-widget-architecture-spike
Fork run:
PR URL:
PR Status:
Started: 2026-07-19T00:17:10.152Z
Completed:

## Work log
- Created: 2026-07-05T22:33:02.402Z

## Task workflow update - 2026-07-11T03:15:33.359Z
- Summary: Performance finding from fork MVP manual session 7: transcript is represented as one monolithic LiveTextWidget. Ordinary live polling reads controller stdout into the in-memory projector (not events.jsonl), but every changed transcript block causes TickPollListener to pass the full transcript array to ChatScreen::setTranscriptBlocks(), which unconditionally invalidates the transcript widget. TranscriptBlockWidget has per-block render caches, so unchanged Markdown/widgets are not fully rebuilt, but it still traverses all blocks, recomposes all rendered lines, and LiveTextWidget wraps the combined output before Symfony TUI frame diff/render. Mutable subagent/fork progress (elapsed/tokens/cost) can therefore trigger whole-transcript recomposition up to the 50ms poll cadence. Session 7 had a 943KB canonical event log and individual message records up to ~150KB, exposing severe lag. Full events.jsonl reads are separate and occur at resume/initialization, RunLeafChanged reconstruction, and child replay-on-enter—not ordinary progress ticks.
- Architecture spike should evaluate independently updateable transcript block widgets and/or visible-region virtualization while preserving natural scrolling. Required goals: block-level invalidation, avoid full transcript line recomposition/wrapping for one mutable card, stable cursor/scroll behavior, and correct Symfony TUI screen diff integration. Measure operation/invalidation counts and bytes/lines processed rather than wall-clock-only tests.
- Short-term metric-only progress coalescing was considered but intentionally deferred: user accepts current slowness as architecture debt as long as rendering/replay correctness is restored. Do not hide or stale live metrics in fork MVP merely to mask the monolithic widget architecture.

## Task workflow update - 2026-07-19T00:17:10.153Z
- Moved TODO → IN-PROGRESS.
- Created branch task/tui-native-symfony-widget-architecture-spike.
- Created worktree /home/ineersa/projects/agent-core-worktrees/tui-native-symfony-widget-architecture-spike.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/tui-native-symfony-widget-architecture-spike.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/tui-native-symfony-widget-architecture-spike.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/tui-native-symfony-widget-architecture-spike.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/tui-native-symfony-widget-architecture-spike.
- Summary: User approved task start as a simple production-path POC in this task worktree: prototype actual mounted Symfony TUI widgets, keep aggregate telemetry, do not add tests, do not add fallback/compatibility modes, and avoid viewport virtualization. Use findings to inform a later tui-rework epic branch and architecture report.

## Task workflow update - 2026-07-19T00:29:27.349Z
- Validation: castor deptrac: passed (0 violations); focused castor phpstan for Transcript, ChatScreen, and InteractiveMode: passed; castor cs-check: passed after castor cs-fix; Production ChatScreen + Symfony VirtualTerminal manual smoke: 2/2 Markdown widgets attached; paired tool exchange visible; Tests and castor check intentionally not run per user-approved POC scope; Real interactive castor run:agent not run because fork session was non-interactive/TTY-blocking
- Summary: Production-path POC implemented and committed in task worktree. ChatScreen now mounts a first-class Symfony transcript subtree with live WidgetContext Markdown theming, stable visual-node reconciliation, paired tool exchange rendering, and aggregate debug telemetry. Architecture report records scout findings, implementation discoveries, final tui-rework/no-viewport decisions, and candid limitations. No tests were written or run per user request; interactive real-TTY feel remains unverified.
- POC commit: 1ec2e2867da3be431690d70b2b74cf58ed2bfcdc
- Report correction commit: f6b02d57b63f16ed22df691da8aeb6288df8299a
- Worktree: /home/ineersa/projects/agent-core-worktrees/tui-native-symfony-widget-architecture-spike
- Next required product evidence: user/parent runs the worktree TUI in a real terminal to assess look and feel.

## Task workflow update - 2026-07-19T02:53:22.781Z
- Validation: castor deptrac: passed (0 violations); focused castor phpstan for Tui Terminal telemetry, TranscriptRenderTelemetry, and InteractiveMode: passed; castor cs-check: passed after castor cs-fix; In-process classification smoke: full_initial terminal INFO summary and transcript INFO summary emitted without content; Tests, castor check, and interactive TUI intentionally not run in this iteration
- Summary: Added default-info aggregate telemetry for transcript reconciliation/render work and terminal redraw classification. Production Tui now uses a TerminalInterface decorator that infers full-initial, full-clear, differential, and non-frame writes from Symfony TUI v8.1 ANSI output without logging terminal content. Detailed events remain debug; summaries emit after the first frame/render then at most every 10 seconds. Reconciler behavior intentionally unchanged pending evidence.
- Telemetry iteration commit: 63fd4303b
- Next evidence: run a new real-TTY session at default info logging and inspect tui.terminal.redraw.summary plus tui.transcript.render.summary.
- Known limitation: session-switch direct STDOUT clear remains uncounted; terminal mode classification is documented ANSI inference.

## Task workflow update - 2026-07-19T03:09:06.186Z
- Summary: Session 2 telemetry analyzed. The transcript used 1,031 snapshots/reconciles and 1,012 renders, with 1,006 mutable invalidations and 421 structure-change resyncs. Terminal emitted 1,243 frames: 1 initial, 10 full-clear, and 1,232 differential. The strongest full-clear burst aligned with large structure-change/add-remove bursts, supporting clear()+add() resynchronization as a flicker contributor; however full clears were rare, so persistent bounce likely also comes from frequent ancestor invalidation, complete layout, and differential row movement.
- Session 2 safe aggregate telemetry: transcript added=349 updated=996 removed=315 reused=45,399; reconciliation duration=906.950ms total, max single=7.757ms; final 32 blocks/34 visual nodes.
- Session 2 terminal telemetry: 4,424 writes, 1,243 frames, 2,195,783 frame bytes; full_initial=1, full_clear=10, differential=1,232, unknown=0.
- Recommended next POC iteration: replace whole-transcript clear()+add() with stable visual-node wrappers and granular append/remove/update, reserving full resync for true non-local reorder; then rerun telemetry.

## Task workflow update - 2026-07-19T03:17:11.640Z
- Validation: castor deptrac: passed (0 violations); castor phpstan --path=src/Tui/Transcript: passed; castor cs-check: passed after scoped castor cs-fix; Production VirtualTerminal smoke: streaming wrappers stable; tool wrapper stable across result; Markdown 2/2 attached; direct remove and tail append granular; outer resync only on non-tail insertion; Tests, castor check, and interactive TTY intentionally not run
- Summary: Granular mounted transcript reconciliation implemented. Stable outer visual-node wrappers now survive ordinary streaming and card updates; Markdown mutates in place, tool result arrival replaces only wrapper content using a stable exchange key, tail nodes append directly, and obsolete nodes remove individually. Whole outer clear()+add() is reserved for genuine non-local insertion or relative-order change. New info telemetry counters support comparison with session 2.
- Granular reconciliation commit: 1742e1360
- New telemetry: outer_resync_total, wrapper_content_replace_total, tail_append_total, direct_remove_total.
- Next evidence: run session 3 and compare outer resync/full clear/differential counters against session 2 baseline (421 structure changes, 10 full clears).

## Task workflow update - 2026-07-19T03:25:26.672Z
- Summary: Analyzed resumed session 2 after granular-wrapper commit. Over ~78 seconds there were 223 reconciles, 31 logical structure changes, but 0 outer transcript resyncs. Changes were handled as 98 wrapper-content replacements, 62 tail appends, and 2 direct removals. Terminal telemetry recorded 1 initial frame, 1 full clear, and 347 differential frames. Whole-container churn and full-clear rate fell materially versus the pre-fix session 2 run, while differential frame/write/byte rates increased, leaving ancestor invalidation/layout as the likely remaining source if bounce persists.
- Resumed session 2 post-fix telemetry: snapshots/reconciles/renders=222/223/218; mutable invalidations=215; added/updated/removed/reused=62/192/2/9,818; outer_resync=0; wrapper_replace=98; tail_append=62; direct_remove=2.
- Resumed terminal telemetry: full_initial/full_clear/differential=1/1/347; writes=1,217; frame bytes=585,770; unknown=0.
- Normalized comparison: structure changes 64.1→23.8/min; full clears 1.52→0.77/min. Differential frames 187.6→266.0/min and terminal bytes 337→453 KB/min increased. Final partial interval is not flushed, so totals are lower bounds.

## Task workflow update - 2026-07-19T03:52:12.106Z
- Validation: castor deptrac: passed; castor phpstan --path=src/Tui/Transcript: passed; castor cs-check: passed; Production VirtualTerminal smoke: grow/shrink/complete reservation stable; terminal resize rebases; wrapper identity and WidgetContext preserved; finalized never-streamed content has no fabricated reservation; Tests, castor check, and interactive TTY intentionally not run
- Summary: Added monotonic streaming Markdown height stabilization. Active streaming Markdown still renders every delta but cannot shrink for the current terminal width; shorter reflow is padded to the maximum observed height, and the reservation is retained after completion to avoid suffix/final-settlement bounce. Width changes reset the reservation. Aggregate geometry counters were added for growth, prevented shrink, and padding.
- Streaming height stabilization commit: 2758158a0
- New telemetry: streaming_height_samples_total, streaming_height_growth_events_total, streaming_height_prevented_shrink_events_total, streaming_padded_rows_total/max.
- Known limit: growth still moves bottom chrome; only shrink/reflow oscillation is prevented.

## Task workflow update - 2026-07-19T03:55:31.559Z
- Validation: castor deptrac: passed (0 violations); castor phpstan --path=src/Tui/Footer: passed; castor cs-check: passed after scoped castor cs-fix; Non-committed footer render smoke at widths 10, 20, 40, 80, and 120: exactly 2 rows, no row exceeded terminal width, empty content preserved second row; Tests, castor check, and interactive TTY intentionally not run
- Summary: Stabilized the default footer at exactly two physical terminal rows. Footer segments retain priority/order, pack into at most two rows, and overflow is ANSI-safely ellipsized instead of creating a third row. Short or empty content uses a non-empty reserved blank second row so LiveTextWidget cannot collapse the footer height. Footer separator remains separate.
- Fixed two-row footer commit: 12afce158
- Overflow uses Symfony AnsiUtils::truncateToWidth with Unicode ellipsis and ANSI-safe reset handling.
- Known scope: default FooterBarWidget only; custom setFooter replacements are not forced to two rows.

## Task workflow update - 2026-07-19T23:06:19.129Z
- Validation: castor deptrac: passed (0 violations); castor phpstan on InteractiveMode, TuiTickDispatcher, and AlwaysActiveInteractiveTickCadencePolicy: passed; castor cs-check: passed; Focused non-committed policy smoke: all handlers dispatched; installed wrapper always returns busy=true; raw dispatcher semantics unchanged; policy does not request render; Tests, castor check, and interactive TTY intentionally not run
- Summary: Investigated footer/editor cadence and implemented an always-active interactive tick policy. There was no separate 250ms footer/editor loop: Symfony uses one adaptive ticker at 10ms while busy and 250ms while idle. InteractiveMode now floors that shared ticker to 10ms for the whole interactive run while preserving request/invalidation-driven rendering. This targets idle editor/footer responsiveness; streaming was already on 10ms and its remaining bounce is transcript row growth/layout movement.
- Always-active interactive 10ms cadence commit: dfae9d28a
- RuntimeEventPoller remains capped at 50ms and was intentionally unchanged.
- Tradeoff: idle interactive sessions now wake about every 10ms instead of 250ms, increasing idle CPU while avoiding forced renders.
- Latest telemetry before cadence change showed streaming height growth and zero prevented shrink/outer resync, confirming the remaining streaming bounce is growth-side geometry rather than footer wrapping or reconciliation churn.

## Task workflow update - 2026-07-19T23:10:18.470Z
- Validation: castor deptrac: passed (0 violations); castor phpstan --path=src/Tui/Application/InteractiveMode.php: passed; castor phpstan --path=src/Tui/Runtime/TuiTickDispatcher.php: passed; castor cs-check: passed; No residual AlwaysActiveInteractiveTickCadencePolicy / tui.tick_cadence.policy references; Tests and castor check not run (task scope)
- Summary: Reverted only the always-active 10ms cadence POC. InteractiveMode again uses Symfony default adaptive busy-hint semantics: ~10ms while any tick handler returns true (runtime active), ~250ms idle. Footer/transcript POC commits preserved.
- Clean git revert of dfae9d28a as b77b31d95 — no history rewrite
- Deleted AlwaysActiveInteractiveTickCadencePolicy.php; restored InteractiveMode onTick to ticks->dispatch only
- Preserved earlier footer hard-fix (12afce158) and streaming monotonic height (2758158a0)

## Task workflow update - 2026-07-19T23:46:12.760Z
- Validation: Ticker revert commit b77b31d95; castor deptrac passed; scoped phpstan passed; castor cs-check passed; Pi scout read coding-agent README, docs/tui.md, linked TUI README, implementation sources and renderer tests; read-only, no build/test/edit
- Summary: Reverted the always-active 10ms ticker experiment and studied Pi's TUI architecture. Hatfield is back to Symfony's default 10ms busy / 250ms idle cadence. Pi's stability comes primarily from explicit terminal-height viewport bookkeeping and append-scroll behavior: it renders a flat full buffer, tracks the visible bottom window, advances that viewport as tail rows append, coalesces renders to ~16ms, diffs changed rows, and wraps writes in synchronized-output mode. Stable bottom chrome therefore requires a viewport/clipping or equivalent terminal scroll-region boundary; it does not require transcript node virtualization.
- Pi root order: header/resources/chat/pending/status/widgets/editor/widgets/footer. Streaming mutates the latest component in place.
- Pi TUI tracks previousViewportTop=max(0, bufferLength-terminalHeight); appending past viewport bottom emits terminal newlines and advances the logical viewport, keeping tail chrome visible.
- Pi requestRender coalesces at ~16ms; differential writes use changed-row ranges and CSI 2026 synchronized output.
- Architectural conclusion: fixed footer height, stable wrapper identity, and monotonic message height cannot prevent downstream row movement in a purely unclipped vertical stack. Smallest viable Hatfield change is a non-virtualized bottom-following transcript viewport: reserve bottom chrome rows, render only transcript's fitting tail slice, retain all mounted transcript nodes.

## Task workflow update - 2026-07-20T00:13:11.507Z
- Validation: castor deptrac: passed (0 violations); castor phpstan scoped to BottomFollowingViewportWidget, ViewportRenderTelemetry, and ChatScreen: passed; castor cs-check: passed; Non-committed VirtualTerminal smoke passed: underfill and overfill exact terminal height; footer row stable; latest transcript retained/oldest clipped; 80/80 Markdown widgets attached; cursor marker present pre-ScreenWriter; before/after overlays ordered/focused/removable; resize recomputed; tiny-terminal chrome suffix retained; telemetry emitted; Tests, castor check, and interactive TTY intentionally not run
- Summary: Implemented a production-path non-virtualized bottom-following viewport. ChatScreen now mounts one custom screen compositor: it fully renders all mounted top-flow/transcript widgets, measures fixed editor/footer chrome first, tail-clips only the resulting top-flow lines to the remaining terminal rows, pads underfill, and appends chrome. The final frame is exactly terminal height, keeping editor/footer fixed while transcript grows. Overlay order/focus was moved into fixed chrome and preserved.
- Bottom-following viewport commit: fda6f2969
- New production classes: src/Tui/Layout/BottomFollowingViewportWidget.php and ViewportRenderTelemetry.php.
- New events: tui.viewport.frame.completed (debug) and tui.viewport.render.summary (coalesced info).
- Limitations: transcript scrollback not yet implemented; descendant WidgetRect coordinates are pre-clip; tiny terminals can clip editor/cursor; full transcript tree still renders before line clipping, so this stabilizes geometry but not CPU cost.

## Task workflow update - 2026-07-20T00:34:24.319Z
- Validation: Exact revert commit 6625560b4; Viewport classes, telemetry, docs, and ChatScreen compositor changes removed with no residual symbols; castor deptrac: passed (0 violations); castor phpstan --path src/Tui/Screen/ChatScreen.php: passed; castor cs-check: passed; Working tree clean; tests/castor check intentionally not run
- Summary: Corrected an unacceptable scope violation: the bottom-following fixed viewport POC was fully reverted because it broke native terminal scrolling and contradicted the task/user constraint against viewport-based behavior. ChatScreen is restored to the native flat Symfony root and ScreenWriter/terminal scrolling. No replacement scrolling system will be proposed or implemented unless the user explicitly reopens that constraint.
- Hard constraint reaffirmed: preserve native terminal scrollback/scroll behavior; no fixed viewport, clipping compositor, custom transcript scrolling, terminal scroll region, or equivalent replacement.
- The Pi comparison was misapplied: borrowing its viewport mechanics violated the explicit Hatfield task constraint even though it explained Pi's stability. Do not repeat.
- Reverted viewport POC fda6f2969 via 6625560b4; earlier mounted transcript, granular reconciliation, monotonic streaming height, fixed two-row footer, and default adaptive ticker remain.

## Task workflow update - 2026-07-20T16:12:45.760Z
- Validation: castor deptrac: passed (0 violations); castor phpstan scoped to PiStyleScreenWriter, alias installer, and InteractiveMode: passed; castor cs-check: passed; Non-committed smokes passed: alias installed before vendor class and Tui private property instantiated app writer; vendor-first load fails loudly; over-height growth preserved full logical frame with no full clears during pure growth; underfill/shrink/resize/positive scrollOffset/cursor shape/width guard exercised; production ChatScreen retained flat root and 2/2 mounted Markdown contexts; Tests, castor check, and interactive TTY intentionally not run
- Summary: Implemented the approved production POC replacing Symfony's private/final ScreenWriter through an early fail-fast class_alias. The app-owned writer preserves the full flat widget frame, mounted Markdown styling, native terminal scrolling/scrollback, synchronized output, and existing public behavior while adding Pi-style persistent viewport and physical-screen cursor bookkeeping. Interactive feel remains the decisive evidence before any upstream work.
- Pi-style ScreenWriter alias POC commit: ed7c86d25
- New classes: src/Tui/Terminal/PiStyleScreenWriter.php and PiStyleScreenWriterAliasInstaller.php.
- One-shot info event: tui.screen_writer.alias.activated with algorithm=pi_native_viewport_bookkeeping.
- Hard constraints preserved: no fixed viewport, clipping compositor, custom transcript scrolling, scroll region, virtualization, fallback, feature flag, vendor edit, or ticker change.
- Risks: process-global alias; real terminal behavior still needs human validation; positive scrollOffset retains Symfony slice semantics; forced full redraws still clear scrollback as stock Symfony does.

## Task workflow update - 2026-07-20T16:40:21.392Z
- Validation: castor deptrac: passed (0 violations); castor phpstan scoped to PiStyleScreenWriter, telemetry, and alias installer: passed; castor cs-check: passed after scoped cs-fix; Non-committed smokes passed: blank/nonblank append normalization; insertion before trailing chrome; over-height differential growth; native-scroll metric path; shrink/no-change/full-render reasons; cursor distance; positive scrollOffset; exact-width counts; first info summary; runtime alias origin; Tests, castor check, and interactive TTY intentionally not run
- Summary: Added the missing Pi appended-line diff normalization and privacy-safe ScreenWriter algorithm telemetry. Blank-only appends now enter the differential changed range instead of being mistaken for no change. No padding, sentinel, or layout behavior was added. Writer summaries now expose logical growth, changed ranges, viewport movement, native-scroll rows, cursor distance, exact-width rows, and full-render reasons for the next session-2 continuation.
- Append normalization and writer telemetry commit: 2eb9dbf45
- New events: tui.screen_writer.frame.completed (debug) and tui.screen_writer.summary (coalesced info).
- No bottom padding/sentinel was added; telemetry will determine whether an exact-width pending-wrap/off-by-one exists.
- Next run: inspect growth/append-start, line delta, changed rows, viewport delta, native scroll rows, exact-width row count, cursor distance, and full reasons alongside subjective bounce.

## Task workflow update - 2026-07-20T17:20:05.115Z
- Validation: castor deptrac: passed (0 violations); castor phpstan --path src/Tui/Terminal/PiStyleScreenWriter.php: passed (0 errors); whole Terminal path only pre-existing AliasInstaller ReflectionClass generics warning; castor cs-check: passed; Non-committed smokes 9/9: ANSI order one content write; erase before CRLF; repeated growth; footer-only; shrink; scrollOffset; exact-width; 0J classified differential; flat ChatScreen/mounted transcript preserved; Tests, castor check, interactive TTY intentionally not run
- Summary: Implemented Fable/persistent-prompt growth tail-rewrite in PiStyleScreenWriter: on visible logical-frame growth that rewrites an existing tail row before trailing chrome, move to firstChanged, CSI 0J erase visible suffix, print full changed tail+chrome, embed hardware cursor when possible, all inside one CSI 2026 synchronized write. Telemetry outcome tail_rewrite + erase/implicit-scroll/embedded-cursor counters added. No layout/padding/viewport changes.
- Growth tail-rewrite commit: 9ce467f3c on top of 2eb9dbf45
- Branch condition: appended && !appendStart && !scrollOffsetSliced && firstChanged in previous viewport and before previous line end
- Sequence: 2026h → move → CSI 0J → print firstChanged..end → embed cursor → 2026l → one write
- Telemetry: outcome=tail_rewrite, erase_to_end, tail_rewrite_rows, implicit_scroll_rows, embedded_cursor; summary counters added
- TerminalRedrawTelemetry treats CSI 0J/J as differential not full_clear
- Next: session-2 interactive — inspect tail_rewrite_frame_count / erase_to_end_count / implicit_scroll_rows and subjective bounce

## Task workflow update - 2026-07-20T17:21:00.208Z
- Validation: castor deptrac: passed (0 violations); castor phpstan --path src/Tui/Terminal/PiStyleScreenWriter.php: passed; castor cs-check: passed; Non-committed 9-case smoke passed: tail-rewrite branch; ANSI order sync-begin < CSI 0J < changed tail/footer < embedded cursor < sync-end; one content write; no CSI 2J/3J; full logical line count retained; next-frame state; repeated growth/exact-width/cursor; old branches for shrink/no-change/footer-only/resize/scrollOffset; telemetry/classification; Tests, castor check, and interactive TTY intentionally not run
- Summary: Implemented erase-before-scroll growth tail rewriting in the Pi-style writer. For visible growth that changes an existing tail row before trailing chrome, the writer now begins synchronized output, moves to the first changed visible row, clears the visible suffix with CSI 0J before any native scroll, prints the changed transcript tail plus all trailing chrome, embeds cursor positioning, ends synchronization, and performs one terminal content write. Flat layout, full logical frame, native scrolling, and scrollback remain intact.
- Erase-before-scroll tail rewrite commit: 9ce467f3c
- New telemetry outcome/counters: tail_rewrite, tail_rewrite_frame_count, erase_to_end_count, tail_rewrite_rows_total/max, implicit_scroll_rows_total/max, embedded_cursor_count.
- Branch is growth-only when firstChanged is inside the previous visible viewport, appendStart=false, and positive scrollOffset is not slicing. Other paths remain unchanged.
- No padding, sentinel, layout, compact-header placement, viewport, vendor, or ticker changes.

## Task workflow update - 2026-07-20T17:24:03.809Z
- Summary: Interactive user validation after commit 9ce467f3c: erase-before-scroll tail rewriting materially improved streaming stability. User reports behavior now resembles the old renderer and is slightly better overall; remaining bounce/flash/tear exists but the TUI is "way way more stable" and looks better.
- Human TTY verdict confirms the persistent-prompt sequence is effective: clear the visible transcript suffix/trailing chrome before native-scroll-producing output, then repaint transcript tail and chrome in one synchronized write.
- Residual minor bounce/flash/tear remains; no further layout, padding, viewport, or footer-placement changes were made.

## Task workflow update - 2026-07-20T17:42:45.070Z
- Validation: castor test --filter=FrameRateLimitedTuiVirtualTest: passed (3 tests, 15 assertions); castor deptrac: passed (0 violations); castor phpstan scoped to FrameRateLimitedTui and InteractiveMode: passed (0 errors); castor cs-check: passed; castor check and interactive TTY intentionally not run in implementation phase
- Summary: Implemented production render-only max-60-FPS POC in commit e3c3a1ffa. New FrameRateLimitedTui keeps Symfony's 10ms active tick/input/runtime cadence and 250ms idle behavior, while coalescing ordinary terminal paints to at most one per 16.67ms. Initial and forced renders bypass the cap; latest pending state is retained.
- Render-only 60 FPS cap commit: e3c3a1ffa29f93f73f39c6ff4c94e59abe996ebd
- Added virtual proof that within-budget requests coalesce, latest state renders after budget, force bypasses, and empty processRender does not advance the frame deadline.
- Added low-frequency tui.frame_rate.summary and DEBUG tui.frame_rate.rendered telemetry. Actual cadence may be ~50–60 FPS because the unchanged active ticker samples at 10ms.
- No AdaptativeTicker alias/vendor edit/reflection, and no viewport/layout/padding/footer changes.

## Task workflow update - 2026-07-20T17:47:09.881Z
- Validation: castor deptrac: passed (0 violations); castor phpstan scoped to InteractiveMode and PiStyleScreenWriter: passed (0 errors); castor cs-check: passed; Verified FrameRateLimitedTui, its virtual test, telemetry, and docs were removed; PiStyleScreenWriter tailRewriteRender remains
- Summary: Rolled back the max-60-FPS render-cap experiment after human TTY validation found no improvement and worse behavior. Revert commit ecc265492 restores stock Symfony Tui render scheduling while preserving the successful erase-before-scroll PiStyleScreenWriter path.
- User verdict: 60 FPS render coalescing made behavior worse and should not be pursued.
- Reverted e3c3a1ffa non-destructively with ecc2654928559dc4456d957903de9e066d9d11f0.
- Current baseline remains erase-before-scroll commit 9ce467f3c with default adaptive 10ms active / 250ms idle Symfony scheduling.

## Task workflow update - 2026-07-20T18:29:18.578Z
- Validation: castor test --filter=PiStyleScreenWriterScrollAheadTest: passed (3 tests, 40 assertions); castor deptrac: passed (0 violations); castor phpstan scoped to PiStyleScreenWriter: passed (0 errors); castor phpstan scoped to PiStyleScreenWriterTelemetry: passed (0 errors); castor cs-check: passed; Full castor check and interactive TTY intentionally not run in implementation phase
- Summary: Implemented narrow scroll-ahead ScreenWriter POC in commit ac1cdcada. For eligible overheight growth, the writer now erases old trailing chrome, performs all required full-screen native scrolling upfront, then repaints the changed tail/chrome without additional scroll in one synchronized write. Existing erase-before-scroll tail rewrite remains the semantic fallback.
- Scroll-ahead commit: ac1cdcada9704b33c03059802ba946f6fedd6bc6
- Added test-local VT interpreter proof: final visible grid, native scrollback contents, scroll timing after CSI 0J and before repaint, cursor mapping, exact-width/no trailing CRLF, and stale-history guard fallback.
- Added scroll_ahead telemetry counters, rows/repaint/frame bytes, and fallback-reason histogram. No DECSTBM, viewport, clipping, alternate screen, padding, ticker, or layout changes.
- No visual improvement claimed pending human TTY A/B.

## Task workflow update - 2026-07-20T18:54:07.722Z
- Summary: Human TTY validation after scroll-ahead commit ac1cdcada: user reports no bouncing. The successful sequence is erase old visible suffix/chrome, perform exact native full-screen growth scrolling upfront, then repaint the changed transcript tail and trailing chrome without further scrolling in one synchronized write.
- Interactive verdict: "no bouncing!"
- Scroll-ahead is now the successful production POC baseline. Preserve guarded fallback to erase-before-scroll for frames where pre-scrolling could write stale changed rows into native scrollback.
- Rejected 60 FPS coalescing remains reverted; no DECSTBM, fixed viewport, clipping, scroll region, padding, or layout workaround was required.

## Task workflow update - 2026-07-20T19:34:34.594Z
- Validation: castor test --filter=PiStyleScreenWriterScrollAheadTest: passed (3 tests, 43 assertions); castor deptrac: passed (0 violations); castor phpstan scoped to PiStyleScreenWriter, PiStyleScreenWriterTelemetry, and TerminalRedrawTelemetry: passed (0 errors); castor cs-check: passed; Full castor check and human TTY intentionally not run in implementation phase
- Summary: Implemented stale-aware row repaint for guarded scroll-ahead in commit 306986cd9. Broad CSI 0J was removed only from the successful scroll-ahead path; native scrolling now occurs first, followed by row-by-row stale-to-fresh repaint. Existing erase-before-scroll tail rewrite retains CSI 0J as the safety fallback.
- Stale-aware repaint commit: 306986cd9b3ad9681c8ffdcb0cb543699e2200b4
- Scroll-ahead row policy skips byte-identical shifted rows, directly overwrites known-width rows that fully cover stale content, clears shorter/unknown rows with CSI 2K immediately before repaint, and avoids clearing newly blank bottom rows.
- Added telemetry for equal rows skipped, newly blank rows, direct overwrites, cleared rows, repaint bytes, and broad erase=false; updated terminal classifier for synchronized no-erase frames.
- Expected degraded-terminal transient is stale chrome shifted upward then corrected row-by-row, not a blank bottom block. No visual claim pending interactive validation.

## Task workflow update - 2026-07-20T20:08:59.580Z
- Validation: castor test --filter=PiStyleScreenWriterScrollAheadTest: passed (3 tests, 57 assertions); castor deptrac: passed (0 violations); castor phpstan scoped to PiStyleScreenWriter and PiStyleScreenWriterTelemetry: passed (0 errors); castor cs-check: passed; Full castor check, tmux E2E, and human TTY intentionally not run in implementation phase
- Summary: Implemented bottom-to-top absolute-CUP repaint for guarded scroll-ahead in commit f5ac94ab7. After native pre-scroll, the writer now corrects bottom chrome first and then walks upward, minimizing footer displacement on the focused tmux client that lacks synchronized-output support.
- Bottom-to-top repaint commit: f5ac94ab7
- Guarded scroll-ahead still uses one write: pre-scroll first, then absolute-CUP stale-aware repaint from bottom to top, then absolute editor cursor. No repaint CRLF and no broad CSI 0J.
- Added telemetry for bottom-to-top order, absolute-CUP repaint rows, and bottom-three completion byte offset.
- Virtual proof verifies bottom three rows become final before transcript repaint and within 1KB, final grid/history/cursor remain correct, exact-width rows are followed by CUP, and unsafe fallback retains CSI 0J.
- No visual claim pending human test on the focused non-Sync tmux client.

## Task workflow update - 2026-07-20T20:18:58.531Z
- Validation: castor test --filter=PiStyleScreenWriterScrollAheadTest: passed (3 tests, 67 assertions); castor deptrac: passed (0 violations); castor phpstan scoped to PiStyleScreenWriter, PiStyleScreenWriterTelemetry, TerminalRedrawTelemetry: passed (0 errors); castor cs-check: passed; Full castor check and human TTY intentionally not run
- Summary: Implemented known-shorter print-then-CSI-0K suffix erase on guarded bottom-to-top scroll-ahead in commit 4fea82a5b. Working/prompts-style short rows no longer clear wholly before print; unknown/image rows keep CSI 2K clear-first; tail-rewrite fallback still uses CSI 0J.
- Print-then-0K commit: 4fea82a5b43bdaadebce616de00eb9b39c9ce5de
- Known-shorter path: CUP → print → CSI 0K; unknown/image: CUP → CSI 2K → print; equal-skip/overwrite/newly-blank/bottom-to-top/pre-scroll/one-write unchanged.
- Telemetry: scroll_ahead_rows_suffix_erased_after_print and scroll_ahead_rows_cleared_before_print (compat rows_cleared = clear-before-print). TerminalRedrawTelemetry treats CSI 0K as differential.
- Virtual proof covers shorter-row prefix snapshots (stale after CUP, not blank after print, final after 0K), exact-width never 0K, fallback still 0J.
- No visual claim pending human TTY on Working/prompts flicker.

## Task workflow update - 2026-07-20T20:19:40.392Z
- Validation: castor test --filter=PiStyleScreenWriterScrollAheadTest: passed (3 tests, 67 assertions); castor deptrac: passed (0 violations); castor phpstan scoped to PiStyleScreenWriter, PiStyleScreenWriterTelemetry, and TerminalRedrawTelemetry: passed (0 errors); castor cs-check: passed; Full castor check, tmux E2E, and human TTY intentionally not run in implementation phase
- Summary: Implemented print-first suffix erasure for known-shorter scroll-ahead rows in commit 4fea82a5b. Working/prompts-style rows now render their new content before CSI 0K removes stale trailing cells, avoiding a fully blank intermediate row on the focused non-Sync tmux client.
- Print-first suffix erase commit: 4fea82a5b43bdaadebce616de00eb9b39c9ce5de
- Known shorter rows use CUP → new row → CSI 0K; unknown/image widths retain CUP → CSI 2K → row. Exact-width rows cannot enter the 0K path.
- Virtual prefix snapshots prove the row remains stale until new content arrives, never becomes wholly blank, and reaches correct final state after suffix erase.
- Added telemetry distinguishing suffix-erased-after-print from conservative clear-before-print. If material flicker remains after human validation, stop writer-level iteration.

## Task workflow update - 2026-07-20T20:57:43.169Z
- Summary: Created Hatfield follow-up issue #303 documenting the mounted-widget spike outcome, the injectable Symfony ScreenWriter prerequisite, the non-Sync presentation limitation, migration constraints, and acceptance criteria. The issue also contains a complete ready-to-copy Symfony TUI issue draft; no Symfony issue was created.
- Hatfield issue: https://github.com/ineersa/agent-core/issues/303
- Upstream Symfony issue body is embedded under 'Draft body for an upstream Symfony TUI issue' and explicitly marked draft-only.
- Confirmed no existing Symfony issue/PR specifically requests ScreenWriter injection before drafting.

## Task workflow update - 2026-08-15T01:46:32.763Z
- Moved IN-PROGRESS → CANCELLED.
- Worktree path missing (/home/ineersa/projects/agent-core-worktrees/tui-native-symfony-widget-architecture-spike); skipping git worktree remove.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/tui-native-symfony-widget-architecture-spike.
- Summary: Cancelled at user request; the spike is complete and no further task workflow is needed.
