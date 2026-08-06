# Issue #258: Make prompt history Up/Down navigation fast in long TUI sessions

## Goal
GitHub issue: https://github.com/ineersa/agent-core/issues/258

Open bug: prompt history Up/Down is extremely slow in long TUI sessions due to linear scans over full transcript, full editor `setText()` rebuilds/undo clearing, and full-screen renders on every navigation step.

Likely areas: `src/Tui/Listener/PromptHistoryNavigator.php`, `src/Tui/Listener/PromptHistoryListener.php`, `src/Tui/Screen/ChatScreen.php`, and Symfony TUI editor integration constraints. Implementation must follow the TUI behavior proof pyramid and preserve draft/history semantics.

## Acceptance criteria
- History navigation avoids per-keypress full-transcript scans in long sessions (for example via an indexed user-message history refreshed from transcript changes).
- Current draft text is preserved across history navigation and returning to the draft; undo/redo loss is avoided or explicitly justified if Symfony editor constraints make it unavoidable.
- Rendering work per Up/Down press is minimized to the lowest practical invalidation path without reintroducing stale/corrupt TUI output.
- Automated proof covers long-session history navigation latency/behavior at the lowest correct TUI layer, plus focused Castor validation.
- No regression to completion-menu Up/Down routing or existing prompt history behavior.

## Workflow metadata
Status: DONE
Branch: task/issue-258-prompt-history-navigation-lag
Worktree: /home/ineersa/projects/agent-core-worktrees/issue-258-prompt-history-navigation-lag
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/269
PR Status: merged
Started: 2026-07-08T19:03:46.695Z
Completed: 2026-07-09T15:25:26.168Z

## Work log
- Created: 2026-07-07T18:08:35.713Z

## Task workflow update - 2026-07-08T19:03:46.695Z
- Moved TODO → IN-PROGRESS.
- Created branch task/issue-258-prompt-history-navigation-lag.
- Created worktree /home/ineersa/projects/agent-core-worktrees/issue-258-prompt-history-navigation-lag.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/issue-258-prompt-history-navigation-lag.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/issue-258-prompt-history-navigation-lag.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/issue-258-prompt-history-navigation-lag.

## Task workflow update - 2026-07-08T19:11:43.647Z
- Validation: castor test OK (4220 tests, 13875 assertions); castor phpstan: no errors; castor cs-check: OK (after castor cs-fix on 3 files); castor deptrac: 0 violations; not run (per task-start scope): castor check, castor test:tui, castor test:llm-real; verified: 3 append sites at SubmitListener L142/153/172; seedFrom at PromptHistoryListener L50; navigator no longer imports TranscriptBlock
- Summary: Implementation complete in worktree. Commit 0b87d3480 on branch task/issue-258-prompt-history-navigation-lag.

Design (locked): new autowireable PromptHistory service (src/Tui/Listener/PromptHistory.php) holds list<string> of user prompts. Seeded once per session from projected transcript UserMessage blocks via PromptHistoryListener::register() (fires on start/resume/switch). Appended on live submit by SubmitListener at the 3 real-prompt dispatch points (bang->originalText, DispatchRuntime->$text, normal->$text); slash/local-commands excluded by routing. PromptHistoryNavigator rewritten to O(1) cursor indexing over prompts() (no longer scans transcript; previous()/next() take no args). Activation model UNCHANGED (empty-only). Rewind: intentional no-op.

Files (4 prod + 6 tests): PromptHistory.php (new), PromptHistoryNavigator.php (rewrite), PromptHistoryListener.php (DI+seed), SubmitListener.php (3 append sites); PromptHistoryTest.php (new), PromptHistoryNavigatorTest.php (+large-session 150x Up walk), PromptHistoryListenerTest.php (DI+seed), SubmitListenerDispatchRuntimeTest.php (append+slash exclusion), SubmitListenerSubagentLiveInputTest.php + SubagentLiveScenarioHarness.php (DI fixture). No services.yaml or depfile.yaml changes.
- task-start: implementation fork (foreground) completed, commit 0b87d3480, all focused gates green
- next: orchestrator/user runs task-to-pr (reviewer subagent + move_task CODE-REVIEW -> castor check in worktree -> PR)

## Task workflow update - 2026-07-09T02:21:08.746Z
- Validation: reviewer subagent (1st pass): REQUEST CHANGES - 1 CONVENTION stale docblock + DEAD CODE reset() + SIMPLIFY test re-seeds + NTH cursor-stability test; all runtime/DI/deptrac/semantics confirmed correct; reviewer subagent (re-review after 5fb89973b): APPROVED - all 4 findings verified resolved; castor test: OK (4220 tests, 13885 assertions); castor phpstan: 0 errors; castor cs-check: 0 files to fix (committed clean); castor deptrac: 0 violations; not run separately (covered by move_task castor check): castor test:tui, castor test:llm-real
- Summary: task-to-pr: reviewer APPROVED after fix iteration. Branch task/issue-258-prompt-history-navigation-lag now has 3 commits: 0b87d3480 (impl), c6d3840ec (cs-fix), 5fb89973b (review fixes). Runtime logic was confirmed correct on first review; fix commit addressed doc/test-only findings: rewrote stale docblock in PromptHistoryListener (was falsely describing the old transcript-scan), removed test-only PromptHistory::reset(), de-duplicated seedFrom() in two navigator walk tests, added cursor-stability-across-append test. Reviewer re-review: APPROVED (all 4 findings verified resolved).
- task-to-pr: inspected worktree - found+committed uncommitted cs-fix residue (c6d3840ec)
- task-to-pr: reviewer 1st pass REQUEST CHANGES (doc/test-only) -> fork fix commit 5fb89973b -> reviewer re-review APPROVED
- task-to-pr: local main is 6 commits ahead of origin/main (pre-existing repo state); PR vs origin/main will temporarily include those commits but auto-cleans when origin/main catches up
- next: move_task to CODE-REVIEW (deterministic castor check + push + PR), then user reviews/merges -> task-done

## Task workflow update - 2026-07-09T02:23:31.833Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (106.2s).
- Pushed task/issue-258-prompt-history-navigation-lag to origin.
- branch 'task/issue-258-prompt-history-navigation-lag' set up to track 'origin/task/issue-258-prompt-history-navigation-lag'.
- Created PR: https://github.com/ineersa/agent-core/pull/269

## Task workflow update - 2026-07-09T02:23:48.605Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/269
- Updated PR Status: open
- Validation: move_task castor check (deterministic, replay-backed): PASSED (106.2s); castor test: OK (4220 tests, 13885 assertions); castor phpstan: 0 errors; castor cs-check: clean; castor deptrac: 0 violations; reviewer: APPROVED
- Summary: task-to-pr complete. Deterministic castor check passed (106.2s) in worktree. Branch pushed to origin. PR created: https://github.com/ineersa/agent-core/pull/269
- task-to-pr: move_task to CODE-REVIEW succeeded - castor check passed (106.2s), branch pushed, PR #269 created

## Task workflow update - 2026-07-09T02:50:41.619Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: review-iterate: PR #269 feedback. Comments #1/#4 (issue-245 files in diff) resolved by pushing local main to origin/main (PR diff now clean). Acting on comment #3: fold PromptHistoryNavigator cursor+methods into PromptHistory (delete the navigator class), and add the clarifying comment from comment #2 (seedFrom runs once per session iteration).

## Task workflow update - 2026-07-09T15:07:29.702Z
- Validation: reviewer subagent (fold re-review, commit 2bbc38a37): APPROVED — behavior-preserving, byte-equivalent navigation logic, seedFrom cursor reset correct for production, tests not weakened, navigator symbol gone; castor test: OK (4220 tests, 13885 assertions) at 2bbc38a37; castor test --filter=PromptHistory: OK (38 tests, 232 assertions) at bd0313cd6; castor phpstan: 0 errors; castor cs-check: 0 files to fix (clean); castor deptrac: 0 violations; rg 'PromptHistoryNavigator' src/ tests/: empty; rg -i 'navigator' in PromptHistoryListener.php + SubmitListener.php: empty (stale refs cleaned)
- Summary: review-iterate complete for PR #269. All 4 inline review comments addressed:
- #1 (PatchApplier.php "why in this PR?") + #4 (EditFileToolTest.php "why here?"): issue-245 base-tracking artifact; RESOLVED by pushing local main to origin/main (fast-forward ec5e02bf9..25889d055); PR diff now contains only prompt-history files.
- #2 (PromptHistoryListener:52 "Will it execute once or every tick?"): verified via InteractiveMode::run() — register() runs once per session-switch loop iteration (line 234, before $tui->run() at line 256 which blocks for the whole session); seedFrom is NOT per-tick/per-submit. Added clarifying comment.
- #3 (PromptHistoryNavigator:18 "Do we still need it even? just use PromptHistory?"): FOLDED the navigator class into PromptHistory. Deleted PromptHistoryNavigator.php; cursor + previous()/next()/isNavigating()/exitNavigation()/cursor() now live on PromptHistory; seedFrom() resets cursor (same per-session-iteration reset as the old per-register new navigator); append() untouched (cursor-stability preserved). Reviewer re-review of the fold: APPROVED (behavior-preserving, byte-equivalent navigation, tests not weakened). Also cleaned 2 stale 'navigator' comment references (PromptHistoryListener docblock + SubmitListener:429).

Branch now has 5 task commits: 0b87d3480, c6d3840ec, 5fb89973b, 2bbc38a37 (fold), bd0313cd6 (docs). All gates green: castor test (4220/13885), phpstan 0, cs-check clean, deptrac 0.

Note: integration checkout had a stray untracked PromptHistory.php left by a fork (wrote to integration checkout instead of worktree) — cleaned up (deleted; canonical version committed in worktree). Integration main is ahead-2 of origin/main (benign: compaction-template PR #270 merged both on GitHub and locally via different merge commits); git pull will reconcile at DONE.
- review-iterate: PR #269 comments #1/#4 resolved by pushing local main (issue-245 files dropped from diff)
- review-iterate: comment #2 answered — seedFrom runs once per session iteration (InteractiveMode register() before $tui->run()); clarifying comment added
- review-iterate: comment #3 — fork 2bbc38a37 folded PromptHistoryNavigator into PromptHistory (class deleted); reviewer re-review APPROVED
- review-iterate: fork bd0313cd6 cleaned 2 stale 'navigator' comment refs (docs-only)
- cleanup: deleted stray untracked PromptHistory.php from integration checkout (fork artifact)
- next: move_task to CODE-REVIEW (castor check + push + update PR), then reply to 4 PR comment threads

## Task workflow update - 2026-07-09T15:09:34.919Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (97.8s).
- Pushed task/issue-258-prompt-history-navigation-lag to origin.
- branch 'task/issue-258-prompt-history-navigation-lag' set up to track 'origin/task/issue-258-prompt-history-navigation-lag'.
- PR already exists: https://github.com/ineersa/agent-core/pull/269
- Summary: review-iterate complete: folded PromptHistoryNavigator into PromptHistory (comment #3), added seedFrom clarifying comment (comment #2), cleaned stale navigator refs. Re-review APPROVED. Moving back to CODE-REVIEW to re-gate and push the fold+docs commits to PR #269.

## Task workflow update - 2026-07-09T15:10:37.592Z
- Updated PR Status: open
- Summary: Back in CODE-REVIEW. Deterministic castor check passed (97.8s) on the fold+docs branch; pushed fast-forward (5fb89973b→bd0313cd6) to origin; PR #269 auto-updated. Replied inline to all 4 review threads (comment ids 3552701654, 3552701777, 3552701920, 3552702109) explaining: #1/#4 = issue-245 base artifact resolved by pushing main; #2 = seedFrom runs once per session iteration (not per tick); #3 = navigator folded into PromptHistory.
- review-iterate DONE: move_task to CODE-REVIEW — castor check passed (97.8s), branch pushed (5fb89973b->bd0313cd6), PR #269 updated
- replied inline to all 4 PR review threads (3552701654/777/920/109)
- awaiting user re-review of PR #269 -> then task-done to merge

## Task workflow update - 2026-07-09T15:25:26.168Z
- Moved CODE-REVIEW → DONE.
- Merged task/issue-258-prompt-history-navigation-lag into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/issue-258-prompt-history-navigation-lag.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/issue-258-prompt-history-navigation-lag.
- Pulled integration checkout: Already up to date..
- Summary: PR #269 merged on GitHub (merge-commit 1c7eab0f4). Pre-synced local main with origin/main (merge 46aa04e5f) so the task branch is already present locally — move_task DONE merge should be a no-op. Local main ahead 4 (z.ai docs 9efbb32f3 + compaction merges + sync merge), behind 0; will push after DONE. Worktree cleanup expected.

## Task workflow update - 2026-07-09T15:27:12.785Z
- Validation: PR #269: MERGED on GitHub (merge-commit 1c7eab0f4, merge-commit method not squash); move_task DONE: task-branch merge = Already up to date (no-op); worktree removed; IDEA exclusions cleaned; pull = Already up to date; git push origin main: 1c7eab0f4..46aa04e5f (local main now in sync with origin/main, ahead 0 / behind 0); post-merge castor test: OK (4220 tests, 13885 assertions, 22.0s); post-merge castor deptrac: 0 violations, 0 errors; post-merge castor phpstan: 0 errors, 0 file_errors; post-merge castor cs-check: 0 files fixed; branch castor check (pre-merge, on task branch): passed 97.8s
- Summary: DONE. PR #269 merged on GitHub (merge-commit 1c7eab0f4). Pre-synced local main with origin/main (merge 46aa04e5f, clean ort merge — brought prompt-history, preserved unique local z.ai docs commit 9efbb32f3) so move_task's task-branch merge was a clean no-op. move_task(DONE) removed the worktree + IDEA exclusions. Pushed local main to origin (1c7eab0f4..46aa04e5f) — now in sync, no ahead/behind. Post-merge focused validation on integration checkout all green: castor test OK (4220/13885), deptrac 0 violations, phpstan 0 errors, cs-check 0 files. Full castor check had previously passed on the branch (97.8s).
- task-done: confirmed PR #269 merged (1c7eab0f4)
- task-done: pre-synced local main with origin/main (merge 46aa04e5f, clean) to avoid redundant task-branch merge — preserved z.ai docs 9efbb32f3
- task-done: move_task(DONE) — no-op merge, worktree removed, IDEA exclusions cleaned
- task-done: pushed local main to origin (1c7eab0f4..46aa04e5f), in sync
- task-done: post-merge focused validation green (test 4220/13885, deptrac 0, phpstan 0, cs-check 0)
- TASK COMPLETE — issue #258 fixed and merged
