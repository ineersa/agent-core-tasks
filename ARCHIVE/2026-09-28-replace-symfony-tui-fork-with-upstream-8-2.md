# Replace Symfony TUI fork with upstream 8.2

## Goal
User approved switching from ineersa/tui to official symfony/tui 8.2 after testing Nicolas's fixes in #66423 and #66431. Official 8.2 head b3a05d5b777cb9f12b10d417e847d90024df7b14 includes both. Accepted behavior: temporary history/viewport overlap on shrink that disappears on regrowth, and in-place viewport repaints instead of the fork's differential-only shrink. Keep main unchanged until the normal merge workflow.

## Acceptance criteria
- Remove ineersa/tui VCS repository and inline fork alias; lock official symfony/tui 8.2 containing both fixes.
- Update fork-specific repaint tests without losing no-whole-screen-clear and visible content correctness checks.
- Validate dependency origin, focused TUI tests and PHAR smoke checks; record accepted tradeoff.

## Workflow metadata
Status: DONE
Branch: task/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/539
PR Status: merged
Started: 2026-09-28T21:34:36+00:00
Completed: 2026-09-28T22:02:12+00:00

## Work log
- Created: 2026-09-28T21:34:28+00:00

## Task workflow update - 2026-09-28T21:34:36+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2/.idea.

## Task workflow update - 2026-09-28T21:35:03+00:00
- Summary: Routing: cohesive dependency switch with known CompletionListenerTest assertions. Main owns implementation. No production renderer or runtime edits. In-process physical ScreenBuffer proof, focused picker/progress tests and PHAR smoke checks; full check belongs to later CODE-REVIEW transition.
- Ownership: owner=main; fork_run=none; revision=95ab949b9; scope=official Composer dependency and fork-specific repaint regression adaptation; outcome=assigned; commit=none

## Task workflow update - 2026-09-28T21:36:59+00:00
- Validation: castor test --filter='CompletionListenerTest|PickerOverlayTest|SubagentProgressCardViewportRenderTest|QuestionControllerTest': 77 tests, 351 assertions passed.; castor cs-check: clean.; Automatic PHAR build and smoke checks passed.; composer validate --no-check-publish: valid with unbounded/exact development constraint warnings.; composer show symfony/tui confirms official github.com/symfony/tui b3a05d5; git diff --check clean.
- Summary: Implemented upstream switch at 99252a343. Removed fork repository/alias; official symfony/tui 8.2 locked and installed at b3a05d5b777cb9f12b10d417e847d90024df7b14. Adapted composition regression to accepted full-viewport in-place repaint, retaining content assertions and adding no-CSI-3J plus Tab checks. Main unchanged. Ready for task-to-pr; full gate not yet run.
- Ownership: owner=main; fork_run=none; revision=99252a343; scope=official Composer dependency and fork-specific repaint regression adaptation; outcome=completed; commit=99252a343

## Task workflow update - 2026-09-28T21:57:05+00:00
- Summary: Independent reviewer APPROVE at 99252a343; artifact agent_c373387047e70647, read-only specification-fidelity, lock/source identity and regression review. No blockers. Reusing implementation focused proof for unchanged revision. Full gate delegated to CODE-REVIEW transition.
- Review: role=reviewer; artifact=agent_c373387047e70647; revision=99252a343; scope=specification fidelity, official upstream identity, Composer lock consistency, retained rendering proof; verdict=APPROVE

## Task workflow update - 2026-09-28T21:59:01+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (102.8s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2/var/reports/qa-20260928-215718-2777-d3481810.
- Session/run: 59.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-28T21:59:03+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2 to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2/var/reports/qa-20260928-215718-2777-d3481810.
- Session/run: 59.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-28T21:59:06+00:00
- castor check passed (102.8s).
- Pushed task/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2 to origin.
- Created PR: <url>
- Session/run: 59.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-28T21:59:06+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (102.8s).
- Pushed task/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2 to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/539

## Task workflow update - 2026-09-28T22:02:12+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2 into integration checkout.
- Merge made by the 'ort' strategy.
 composer.json                                 |  6 +-----
 composer.lock                                 | 45 +++++++++++++++++++++++++++++----------------
 tests/Tui/Listener/CompletionListenerTest.php | 13 ++++++++-----
 3 files changed, 38 insertions(+), 26 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-28-replace-symfony-tui-fork-with-upstream-8-2.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR #539 merged. Integrating official Symfony TUI switch; post-merge dependency install and full validation follow.

## Task workflow update - 2026-09-28T22:03:47+00:00
- Updated PR Status: merged
- Validation: composer install completed; source github.com/symfony/tui b3a05d5b777cb9f12b10d417e847d90024df7b14 verified.; castor check qa-20260928-220219-7815-287dbcf9 passed all 11 lanes, artifact integrity and leak checks. Unit 4879/22078, controller replay 14/253, TUI 6/40, LLM-real 5/30.; PHAR rebuild and smoke checks passed. git status --short empty; task worktree absent.
- Summary: Integrated PR #539, installed official symfony/tui b3a05d5 on main and rebuilt PHAR. Post-merge full check passed at 5fb7da562c8c6f5a5a7c6dfc9f3ccc9fd1bee6a8. Working tree clean and task worktree removed.
