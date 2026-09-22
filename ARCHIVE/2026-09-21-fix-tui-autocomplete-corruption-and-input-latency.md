# Fix Symfony TUI autocomplete corruption and input latency after fixed scrollback boundary

## Goal
The fixed native-scrollback boundary ScreenWriter patch at Symfony d0c6e96eaf / standalone TUI 5818dc2888 corrupts Hatfield autocomplete and makes Tab selection and editor newlines take seconds in long sessions. The failure reproduces in Ghostty without tmux; upstream Symfony TUI renders the same interaction correctly. First capture the actual previous/current frames and ScreenWriter transition state to identify the selected render path and invalid invariant. Do not replace the fixed-boundary design or add a public API until evidence shows that a local rendering/accounting fix is insufficient.

## Acceptance criteria
- Deterministically reproduce the stale editor borders, duplicated footer, or equivalent wrong physical viewport from the actual Hatfield autocomplete/editor frame transition.
- Capture frame counts, terminal dimensions, changed range, historyCommittedThrough, hardware cursor row, selected render path, emitted byte count, and render duration without logging transcript content.
- Explain the proven cause before choosing the fix; distinguish a local rendering/accounting defect from a mutable-suffix or immutable-prefix contract violation.
- Autocomplete open, query updates, Tab selection, close, and editor newline render the correct physical viewport without stale editor, overlay, status, or footer rows.
- Supported autocomplete and editor transitions do not fall back to work proportional to the full transcript; focused proof must show bounded emitted output or render work for long history.
- Preserve the original combined native-history and viewport uniqueness behavior, including deep and staged shrink cases and pre-existing shell history.
- Add deterministic lowest-layer regression coverage that consumes the ScreenWriter's actual ANSI output and checks physical viewport plus native history; add only the minimum terminal integration proof unavailable below.
- Keep Symfony and standalone TUI fork implementations and tests synchronized, then update Agent Core's Composer pin and rebuild the PHAR.
- If the reproduction proves that mutable UI exceeds the viewport or crosses an irreversible history boundary, stop for a product decision instead of silently clearing scrollback or inventing an overflow policy.

## Workflow metadata
Status: DONE
Branch: task/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/521
PR Status: merged
Started: 2026-09-21T16:03:42+00:00
Completed: 2026-09-21T17:39:52+00:00

## Work log
- Created: 2026-09-21T16:03:33+00:00

## Task workflow update - 2026-09-21T16:03:42+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency/.idea.
- Summary: Starting evidence-first reproduction of direct-terminal autocomplete corruption and long-session input latency. Main will own capture, root-cause analysis, minimal ScreenWriter fix, synchronized upstream/fork changes, and focused validation.

## Task workflow update - 2026-09-21T16:03:51+00:00
- Ownership: owner=main; fork_run=none; revision=1d307a3f53d04d460e7d637528fe8db6c399ce8d; scope=capture actual autocomplete/editor frame transitions, prove ScreenWriter render path and invariant, implement the smallest supported fix across Symfony/TUI fork and Agent Core pin, and run focused TUI validation; outcome=assigned; commit=none

## Task workflow update - 2026-09-21T16:13:27+00:00
- Validation: castor test --filter=longTranscriptAutocompleteUpdatesThePhysicalViewportWithoutStaleChrome: focused probe executes in 0.13s; current ScreenBuffer assertions pass after diagnostic correction.; Captured per-key frame counts, changed ranges, history boundary, cursor row, output bytes, and erase sequence counts without transcript content.
- Summary: Deterministic non-virtual ScreenWriter probe now reproduces the real Hatfield autocomplete frame shape with a 100-line transcript at 224x60 and records transition state. Opening '/' changes 120→132 lines, firstChanged=116, commits history 60→72 through differentialRender, writes 2,582 bytes and 16 line clears. Refinement that shrinks results uses redrawViewport, writes 5.2–5.8 KB and clears all 60 rows. The ANSI ScreenBuffer remains correct, so the final stale-footer symptom is not explained by frame slices alone; physical cursor divergence or a real-session-specific transition remains to prove.

## Task workflow update - 2026-09-21T17:22:47+00:00
- Validation: Standalone TUI full suite: 2051 tests, 6411 assertions passed.; Standalone TUI PHPStan: no errors for Render/ScreenWriter.php.; Agent Core targeted test: 76 tests, 319 assertions passed (CompletionListenerTest, QuestionControllerTest, PickerOverlayTest).; Agent Core castor cs-check: clean.; PHAR build and smoke tests passed.; User live-tested successfully in Ghostty, GNOME Terminal, and tmux.; Independent reviewer APPROVE for Agent Core fd9dfca8f, TUI 0ed99e7, Symfony 99816cced1.
- Summary: Implemented and live-tested the differential shrink fix. Standalone TUI revision 0ed99e761763a22120e040e4554a1f9525ffc58c and Symfony mirror 99816cced1 remove whole-viewport redraws for shrinking frames, simplify the committed-history invariant, and add an exact autocomplete/editor/footer regression. Agent Core revisions 0994cfb18 and fd9dfca8f pin the fork, add a 224x60 physical-terminal regression, and refresh the stable question-overlay rationale. Independent reviewer approved after follow-up. Symfony PR #66187 description updated to document differential tail erasure and the fixed-boundary tradeoff.

## Task workflow update - 2026-09-21T17:36:52+00:00
- Summary: Task-to-PR review approved at Agent Core revision fd9dfca8f6307fa6602b8046d5bd730870bbf9ef; reviewer artifact agent_19f68ee8ed06518b. User requested merging the unsquashed Hatfield branch to main for large-transcript testing. External TUI/Symfony commits remain as reviewed and signed.
- Ownership: owner=main; fork_run=none; revision=fd9dfca8f6307fa6602b8046d5bd730870bbf9ef; scope=Composer pin, physical-terminal autocomplete regression, and stale renderer comment update; outcome=completed; commit=fd9dfca8f6307fa6602b8046d5bd730870bbf9ef
- Review: role=independent reviewer; artifact=agent_19f68ee8ed06518b; revision=fd9dfca8f6307fa6602b8046d5bd730870bbf9ef; scope=specification fidelity, renderer boundary correctness, regression quality, lock exactness; verdict=APPROVE

## Task workflow update - 2026-09-21T17:39:14+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (124.1s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency/var/reports/qa-20260921-173710-6570-10287eb7.
- Session/run: 59.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-21T17:39:16+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency/var/reports/qa-20260921-173710-6570-10287eb7.
- Session/run: 59.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-21T17:39:18+00:00
- castor check passed (124.1s).
- Pushed task/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency to origin.
- Created PR: <url>
- Session/run: 59.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-21T17:39:18+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (124.1s).
- Pushed task/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/521

## Task workflow update - 2026-09-21T17:39:52+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency into integration checkout.
- Merge made by the 'ort' strategy.
 composer.lock                                 |  8 ++++----
 src/Tui/Question/QuestionController.php       |  7 +++----
 tests/Tui/Listener/CompletionListenerTest.php | 98 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++-----
 3 files changed, 100 insertions(+), 13 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-21-fix-tui-autocomplete-corruption-and-input-latency.
- Pulled integration checkout: Already up to date..
- Summary: User explicitly approved merging PR #521 to main for large-transcript integration testing. CODE-REVIEW castor check passed before merge; independent reviewer approved revision fd9dfca8f.

## Task workflow update - 2026-09-21T17:43:29+00:00
- Validation: Initial post-merge castor check qa-20260921-174001-11441-e21159d3: expected failure in longTranscriptAutocompleteUpdatesThePhysicalViewportWithoutStaleChrome because installed vendor remained at old 5818dc2 (60 clear-line repaint).; composer install upgraded installed symfony/tui 5818dc2 -> 0ed99e761763a22120e040e4554a1f9525ffc58c; installed ScreenWriter.php matches fork.; Final post-merge castor check qa-20260921-174139-15113-000b5fb0: all 11 lanes passed; unit 4993 tests/21908 assertions, controller replay 13/218, TUI 6/40, LLM-real 5/30; deptrac, PHPStan, LSP, dead-code, CS, docs, catalog, artifact integrity, and leak checks passed.
- Summary: Post-merge validation completed on main. The first post-merge castor check exposed that the integration checkout still had symfony/tui 5818dc2 installed even though composer.lock pinned 0ed99e7; the new regression correctly failed with a 60-row repaint. Ran composer install, verified installed symfony/tui source 0ed99e761763a22120e040e4554a1f9525ffc58c and byte-identical ScreenWriter.php, then reran castor check successfully. Pulled the remotely merged PR #521 into local main.
