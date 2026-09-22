# Keep Symfony TUI native scrollback boundary non-overlapping

## Goal
Follow-up to issue #512 and PR #515. The existing signed fork fix prevents unbounded viewport snapshots but still displays rows already committed to native scrollback when an overheight tail shrinks. The user explicitly accepts blank rows after shrink so the history/view boundary can remain monotonic. Update both the Symfony monorepo 8.2 branch and standalone TUI fork, then refresh Agent Core's Composer lock to the corrected signed fork commit.

## Acceptance criteria
- For a stable committed prefix, each content row belongs to either native scrollback or the live viewport, never both.
- Tail shrink keeps the committed history/view boundary fixed and clears newly unused bottom rows instead of pulling archived rows back into the viewport.
- Tail regrowth within the committed boundary fills blank rows without scrolling; genuine growth beyond the previous maximum commits each newly archived prefix row once.
- Changes inside the committed prefix, resize, scroll offset, cursor placement, deletion handling, and end-of-content positioning retain explicit correct behavior.
- Regression tests assert combined scrollback plus viewport uniqueness after every long/short frame and after appending the exact completion output; existing shell output remains preserved.
- The standalone 100x20 tmux reproduction finishes with one occurrence of Transcript line 04 and without repeated history growth.
- Symfony monorepo and standalone TUI branches contain equivalent tested changes; Agent Core Composer lock resolves the corrected signed fork commit.

## Workflow metadata
Status: DONE
Branch: task/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/516
PR Status: merged
Started: 2026-09-20T01:02:46+00:00
Completed: 2026-09-20T20:59:56+00:00

## Work log
- Created: 2026-09-20T01:02:37+00:00

## Task workflow update - 2026-09-20T01:02:46+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping/.idea.

## Task workflow update - 2026-09-20T01:02:56+00:00
- Ownership: owner=main; fork_run=none; revision=df32269abb22c28293fe201ecb77b8850b4ce9ca/e682d162f31aa82832d8255f4c9c3fd17977e903; scope=implement monotonic non-overlapping native scrollback boundary in Symfony monorepo and standalone TUI, add combined-state proofs, refresh Agent Core Composer lock; outcome=assigned; commit=none

## Task workflow update - 2026-09-20T01:16:27+00:00
- Validation: Symfony TUI full suite: 2,047 tests, 6,385 assertions passed.; Standalone TUI full suite: 2,047 tests, 6,385 assertions passed (one existing PHPUnit deprecation).; PHP CS Fixer dry-run: 0 of 2 files need changes.; Real isolated 100x20 tmux reproduction: history_size=5; Transcript line 04 occurrences=1 after 20 oscillations and completion output.; Agent Core PickerOverlayTest: 12 tests, 139 assertions passed.; Agent Core PHAR build and smoke passed; TuiSubagentProgressE2eTest: 1 test, 18 assertions passed.
- Summary: Implemented the user-approved fixed native-scrollback boundary. Stable tail shrink leaves blank rows rather than redisplaying committed history; regrowth fills them without scrolling; genuine growth advances the boundary; committed-prefix edits and terminal dimension changes rebuild explicitly. Updated equivalent Symfony and standalone TUI branches with unsigned commits, then refreshed Agent Core's lock to the new fork revision.
- Ownership: owner=main; fork_run=none; revision=2e4f2c1eadc1d6ecea9286fa9cb9f9da26bed0f7; scope=implement monotonic non-overlapping native scrollback boundary in Symfony monorepo and standalone TUI, add combined-state proofs, refresh Agent Core Composer lock; outcome=completed; commit=93ada00d20ebf2ca17e084c2e1aa11bdbb945f2b/ea5ecb829f55de239ad55cd0e5fb9faab4f5f914/2e4f2c1eadc1d6ecea9286fa9cb9f9da26bed0f7

## Task workflow update - 2026-09-20T02:06:38+00:00
- Summary: Implementation and focused validation are complete, but the required independent reviewer is temporarily blocked: reviewer artifact agent_f34adb2251f0b3ea failed three attempts because its provider reached maximum request duration. No review verdict was produced, so the task has not been advanced to CODE-REVIEW or merged into main.
- Review: role=reviewer; artifact=agent_f34adb2251f0b3ea; revision=93ada00d20/ea5ecb8/2e4f2c1ea; scope=ScreenWriter fixed-boundary behavior, combined-state tests, Composer lock; verdict=BLOCKED_PROVIDER_TIMEOUT

## Task workflow update - 2026-09-20T20:52:48+00:00
- Validation: Independent reviewer artifact agent_7cc60a5f134232db reviewed Symfony e7c2e5fbab, TUI 5818dc2888, and Agent Core 69c07688ae: behavior and lock correctness approved; signature requirement waived by user.; Symfony TUI full suite: 2,050 tests, 6,392 assertions passed.; Focused staged-shrink regression: 2 tests, 46 assertions passed.; Agent Core PickerOverlayTest: 12 tests, 139 assertions passed; PHAR build and smoke passed.; Standalone 100x20 tmux reproduction on final rendering code: history_size=5 and Transcript line 04 occurs once.
- Summary: Final independent review found the rendering behavior, regression coverage, cross-repository parity, and Composer lock exactness correct. The only blocking finding was that the latest fork commits are unsigned; the user explicitly waived the signed-commit acceptance criterion and directed merging the unsigned revision.
- Review: role=reviewer; artifact=agent_7cc60a5f134232db; revision=e7c2e5fbab/5818dc2888/69c07688ae; scope=fixed history boundary, staged shrink/regrowth, resize cursor, lock exactness; verdict=REQUEST_CHANGES only for unsigned commit provenance.
- User decision: explicitly waived the signed-commit criterion and approved merging the unsigned revision.

## Task workflow update - 2026-09-20T20:55:10+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (127.8s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping/var/reports/qa-20260920-205302-1920-372b4836.
- Session/run: 59.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-20T20:55:11+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping/var/reports/qa-20260920-205302-1920-372b4836.
- Session/run: 59.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-20T20:55:14+00:00
- castor check passed (127.8s).
- Pushed task/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping to origin.
- Created PR: <url>
- Session/run: 59.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-20T20:55:14+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (127.8s).
- Pushed task/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/516
- Validation: Symfony TUI: 2,050 tests, 6,392 assertions passed.; Agent Core PickerOverlayTest: 12 tests, 139 assertions passed.; PHAR build and smoke passed.; Standalone tmux reproduction: history_size=5; boundary line occurs once.; Independent reviewer verified rendering behavior and lock exactness; user waived signed-commit requirement.
- Summary: Update Agent Core's Composer lock to the corrected TUI fork revision after final rendering and regression validation.

## Task workflow update - 2026-09-20T20:59:56+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping into integration checkout.
- Merge made by the 'ort' strategy.
 composer.lock | 8 ++++----
 1 file changed, 4 insertions(+), 4 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-20-keep-symfony-tui-native-scrollback-boundary-non-overlapping.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: CODE-REVIEW castor check passed in 127.8s.; GitHub PR #516 confirmed MERGED.; Integration checkout dirty files were inspected and are limited to unrelated observational-memory source/test changes; requireCleanMain=false used intentionally.
- Summary: PR #516 was already merged on GitHub. Updating the integration checkout while preserving two unrelated unstaged observational-memory changes; they do not overlap the Composer lock update.

## Task workflow update - 2026-09-20T21:01:18+00:00
- Validation: Post-merge castor check passed in 145.0s on QA run qa-20260920-210004-7191-d314eee8.; All 11 lanes passed: 5,123 unit tests/22,207 assertions; controller replay 11/175; TUI 9/69; LLM-real 5/30; deptrac, PHPStan, LSP, dead-code, CS, docs, and catalog checks passed.; QA artifact integrity, process leak check, cache cleanup, and llama-proxy cache guard passed.; Task worktree removed. Integration checkout is intentionally not clean only because unrelated observational-memory source/test edits predated this merge and were preserved.
- Summary: Post-merge validation passed on Agent Core main at db1ae939f2. PR #516 is merged, the task worktree was removed, and main now locks symfony/tui to 5818dc28888921d87e7b4696335ca60152305a1e. Two pre-existing unrelated observational-memory files remain modified and were preserved.
