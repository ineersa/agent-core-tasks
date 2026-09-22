# Keep live subagent and fork cards compact and height-stable

## Goal
Live single, fork, and parallel subagent cards currently render variable-length task, identity, tool-history, excerpt, metrics, and hint sections. In an overheight normal-screen session, mutable card rows enter native scrollback. Later progress updates then force the scrollback-safe ScreenWriter to repaint the full viewport with per-row erases, which old iTerm2 exposes as severe flashing. Raw capture from session 60 showed four 50-row repaints during subagent state transitions, while ordinary spinner frames repainted one row. Make live cards compact and keep their row count stable through progress updates. Detailed activity remains available through `/agents-live`.

## Acceptance criteria
- A live single subagent or fork card has a compact, fixed row count across ordinary progress, tool, and status updates.
- A live parallel card uses a predictable fixed number of rows derived from the declared child count; each child has the same compact row allocation.
- The main card keeps essential identity, task, status, progress metrics, and current activity. Detailed tool history and assistant excerpts remain available through `/agents-live` instead of expanding the transcript card.
- Optional or not-yet-populated fields use reserved rows rather than changing the card height.
- Terminal status and handoff guidance do not unexpectedly grow or shrink the existing progress card.
- Add deterministic virtual rendering proofs for single and parallel card height stability across representative lifecycle snapshots.
- Add the lowest-correct terminal proof that representative live updates no longer trigger whole-viewport repaint output when the compact card remains in the live viewport.

## Workflow metadata
Status: DONE
Branch: task/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/519
PR Status: merged
Started: 2026-09-21T01:03:59+00:00
Completed: 2026-09-21T02:00:40+00:00

## Work log
- Created: 2026-09-21T01:03:49+00:00

## Task workflow update - 2026-09-21T01:03:59+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable/.idea.
- Validation: Observed session 60 raw pane output: four 50-row per-line viewport repaints during subagent state transitions; ordinary spinner frames repainted one row.; Loaded task-start, implementation ownership, specification fidelity, TUI proof, testing, root tests, and TUI instructions.
- Summary: Starting the approved compact-card change. Main owns the cohesive SubagentProgressCardWidget rendering and proof slice.

## Task workflow update - 2026-09-21T01:04:17+00:00
- Ownership: owner=main; fork_run=none; revision=cc66209c36919765af07e68d815a217db207366b; scope=compact height-stable single/fork/parallel progress cards and deterministic rendering/terminal proofs; outcome=assigned; commit=none

## Task workflow update - 2026-09-21T01:14:15+00:00
- Validation: `castor test --filter='SubagentResultRendererTest|SubagentProgressCardViewportRenderTest'`: 27 tests, 218 assertions passed.; Physical non-virtual ScreenWriter proof confirms representative parallel running and completion updates erase at most the card's 8 rows, with no CSI 2J or CSI 3J and no whole-viewport repaint.; `castor test:tui --filter=TuiSubagentProgressE2eTest`: 1 test, 18 assertions passed.; `castor cs-check`: 0 files need changes.; `castor phpstan --path=src/Tui/Transcript/SubagentProgressCardWidget.php`: 0 errors.
- Summary: Live single/fork cards now use a fixed nine-row footprint. Parallel cards use `2 * total children + 4` rows, reserve slots before child snapshots arrive, and show one compact status row plus one activity/task row per child. Single cards retain task, artifact/run identity, current activity, metrics, context usage, and guidance. Tool history is reduced to the latest entry and assistant excerpts stay in `/agents-live`. Handoff guidance has a reserved row so terminal completion does not resize the card.
- Ownership: owner=main; fork_run=none; revision=53df8759472e3049d2d7372c71608945f2e6a43a; scope=compact height-stable single/fork/parallel progress cards and deterministic rendering/terminal proofs; outcome=completed; commit=53df8759472e3049d2d7372c71608945f2e6a43a

## Task workflow update - 2026-09-21T01:17:28+00:00
- Summary: Starting independent specification-fidelity review for revision 53df8759472e3049d2d7372c71608945f2e6a43a. Review scope covers card-height invariants, retained user-visible information, terminal repaint proof, regressions, architecture, and unnecessary complexity.

## Task workflow update - 2026-09-21T01:32:55+00:00
- Summary: Independent reviewer artifact agent_4ddd7470b41b6c0a returned APPROVE WITH SUGGESTIONS for revision 53df8759472e3049d2d7372c71608945f2e6a43a. Before submission, main will remove vestigial child-header state, correct the stale formatter synchronization comment, and apply the small reachable-invariant cleanups noted in review.
- Ownership: owner=main; fork_run=none; revision=53df8759472e3049d2d7372c71608945f2e6a43a; scope=review follow-ups for dead render state, formatter contract comment, aggregate guidance symmetry, index contract, and whitespace-only task summaries; outcome=assigned; commit=none

## Task workflow update - 2026-09-21T01:35:33+00:00
- Validation: Review-follow-up focused suite: 28 tests, 230 assertions passed.; Focused replay-backed TUI E2E: 1 test, 18 assertions passed.; CS check passed; scoped PHPStan passed for both changed production files.
- Summary: Applied reviewer follow-ups at revision 543f7676e6e4c557713211448f154d4ab59ccb97: removed dead child-header state, documented compact-versus-full formatter responsibilities, made parallel guidance unconditional, documented one-based child indices, handled whitespace-only tasks, and covered failed/cancelled lifecycle states.
- Ownership: owner=main; fork_run=none; revision=543f7676e6e4c557713211448f154d4ab59ccb97; scope=review follow-ups for dead render state, formatter contract comment, aggregate guidance symmetry, index contract, and whitespace-only task summaries; outcome=completed; commit=543f7676e6e4c557713211448f154d4ab59ccb97

## Task workflow update - 2026-09-21T01:37:58+00:00
- Validation: Independent specification-fidelity review: APPROVE at 543f7676e6e4c557713211448f154d4ab59ccb97.
- Summary: Independent reviewer artifact agent_4ddd7470b41b6c0a APPROVED revision 543f7676e6e4c557713211448f154d4ab59ccb97. The reviewer confirmed the fixed single and parallel row formulas, reserved missing-child rows, differential ANSI proof, retained detail paths, and all review follow-ups. No blockers remain.

## Task workflow update - 2026-09-21T01:40:17+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (127.8s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable/var/reports/qa-20260921-013809-5397-7ed7ea83.
- Session/run: 59.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-21T01:40:19+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable/var/reports/qa-20260921-013809-5397-7ed7ea83.
- Session/run: 59.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-21T01:40:21+00:00
- castor check passed (127.8s).
- Pushed task/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable to origin.
- Created PR: <url>
- Session/run: 59.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-21T01:40:21+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (127.8s).
- Pushed task/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/519

## Task workflow update - 2026-09-21T02:00:40+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable into integration checkout.
- Merge made by the 'ort' strategy.
 src/CodingAgent/Runtime/Projection/SubagentProgressDisplayFormatter.php |   5 ++--
 src/Tui/Transcript/SubagentProgressCardWidget.php                       | 116 +++++++++++++++++++++++++++++++++++++++++++-----------------------------------------
 tests/Tui/Transcript/SubagentProgressCardViewportRenderTest.php         | 120 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Transcript/SubagentResultRendererTest.php                     | 125 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++--
 4 files changed, 306 insertions(+), 60 deletions(-)
 create mode 100644 tests/Tui/Transcript/SubagentProgressCardViewportRenderTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-21-keep-live-subagent-and-fork-cards-compact-and-height-stable.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #519 is merged on GitHub at merge commit 35125ca37ff107f723e116bbb698ee2a85b8c4db. Proceeding with integration checkout update and worktree cleanup.

## Task workflow update - 2026-09-21T02:02:05+00:00
- Validation: `castor check` passed all 11 lanes on integration revision 1d307a3f53d04d460e7d637528fe8db6c399ce8d.; Unit suite: 4,991 tests and 21,879 assertions passed.; Controller replay: 13 tests and 218 assertions passed.; TUI E2E: 5 tests and 25 assertions passed.; LLM-real: 5 tests and 30 assertions passed.; Deptrac, PHPStan, LSP diagnostics, dead-code analysis, CS check, docs validation, catalog version check, QA artifact integrity, process leak check, and llama-proxy cache guard passed.; Integration checkout has no uncommitted changes; task worktree removal confirmed.
- Summary: Post-merge validation completed on integration revision 1d307a3f53d04d460e7d637528fe8db6c399ce8d. PR #519 is merged, the task worktree is removed, and the integration checkout is clean.
