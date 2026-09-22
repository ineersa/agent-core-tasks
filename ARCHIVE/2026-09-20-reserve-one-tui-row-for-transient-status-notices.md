# Reserve one TUI row for transient status notices

## Goal
The status panel currently renders zero rows when empty and one or more rows when statuses such as `om-background` appear. On the normal terminal screen, this frame expansion and shrink moves the editor/footer, exposes fixed-history blank rows, and triggers stale cursor/partial rendering on older synchronized-output clients. Reserve one row as the status panel's minimum footprint. Showing or clearing one status must replace that row without changing total frame height. Preserve existing behavior for multiple simultaneous status rows.

## Acceptance criteria
- An empty status panel renders one reserved blank row.
- Showing or clearing a single keyed status does not move the editor or footer.
- Multiple simultaneous status entries remain visible and may use additional rows.
- Add deterministic virtual TUI proof for the empty, shown, and cleared lifecycle.
- Run focused Castor validation at the lowest correct TUI layer.

## Workflow metadata
Status: DONE
Branch: task/2026-09-20-reserve-one-tui-row-for-transient-status-notices
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-20-reserve-one-tui-row-for-transient-status-notices
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/517
PR Status: merged
Started: 2026-09-20T22:18:21+00:00
Completed: 2026-09-21T00:25:20+00:00

## Work log
- Created: 2026-09-20T22:18:03+00:00

## Task workflow update - 2026-09-20T22:18:21+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-20-reserve-one-tui-row-for-transient-status-notices.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-20-reserve-one-tui-row-for-transient-status-notices.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-20-reserve-one-tui-row-for-transient-status-notices.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-20-reserve-one-tui-row-for-transient-status-notices.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-20-reserve-one-tui-row-for-transient-status-notices.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-20-reserve-one-tui-row-for-transient-status-notices/.idea.
- Validation: Loaded root and TUI instructions, testing skill, tests/AGENTS.md, task-start procedure, ownership rules, specification-fidelity rules, and TUI proof requirements.
- Summary: Starting the approved TUI change. Main owns the cohesive StatusPanelWidget and virtual-render proof slice.

## Task workflow update - 2026-09-20T22:18:30+00:00
- Ownership: owner=main; fork_run=none; revision=4a3e62443f10f0791adc13603f10b2fbd8004c97; scope=reserve StatusPanelWidget minimum row and add deterministic virtual lifecycle proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-20T22:21:25+00:00
- Validation: Pre-fix focused test failed because the footer moved from row 19 to row 20 when the first status appeared.; Post-fix `castor test --filter=ChatScreenStatusRowVirtualRenderTest`: 2 tests, 33 assertions passed.; `castor cs-check`: 0 files need changes.
- Summary: StatusPanelWidget now renders one blank row when it has no entries. A single status such as om-background replaces that row instead of changing frame height. Multiple statuses retain their existing multi-row behavior. Added a virtual ChatScreen lifecycle proof that editor and footer anchors remain fixed across empty, shown, and cleared states.
- Ownership: owner=main; fork_run=none; revision=4cca74d5519f09c39bbc00aefe2e6588da685825; scope=reserve StatusPanelWidget minimum row and add deterministic virtual lifecycle proof; outcome=completed; commit=4cca74d5519f09c39bbc00aefe2e6588da685825

## Task workflow update - 2026-09-20T22:41:03+00:00
- Validation: Reviewer found no correctness, security, architecture, specification-fidelity, or test-quality issues. Optional suggestions were not required for approval.
- Summary: Independent reviewer approved revision 4cca74d5519f09c39bbc00aefe2e6588da685825 with suggestions only. The reviewer confirmed that `['']` reserves a real layout row, the virtual lifecycle test is deterministic at the lowest correct layer, multiple and wrapped statuses retain existing behavior, and the change adds no unsupported API or complexity.
- Review: role=reviewer; artifact=agent_bdd1ba5a25b87a83; revision=4cca74d5519f09c39bbc00aefe2e6588da685825; scope=specification fidelity, TUI layout correctness, regressions, deterministic proof, architecture, and complexity; verdict=APPROVE WITH SUGGESTIONS

## Task workflow update - 2026-09-20T22:43:21+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-20-reserve-one-tui-row-for-transient-status-notices.md.
- Session/run: 59.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-20-reserve-one-tui-row-for-transient-status-notices/var/reports/qa-20260920-224115-3552-25a5ee75.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-20T22:51:05+00:00
- Validation: Failed full gate: test lane clipped the `Keyboard shortcuts` title because the new reserved row consumed one of the test's 80 rows.; Post-fix focused validation: `castor test --filter='TuiVirtualInputTest|ChatScreenStatusRowVirtualRenderTest'` passed 8 tests and 233 assertions.; Post-fix `castor cs-check`: 0 files need changes.; Final reviewer verdict: APPROVE WITH SUGGESTIONS; no blocking findings.
- Summary: The first CODE-REVIEW gate exposed a stale 80-row hotkeys test assumption. The test lane stopped at the first clipped title. Follow-up review found the sibling resize test had the same assumption. Both tests now share `HOTKEYS_VIEWPORT_ROWS = 81`, including the width-only resize. Final reviewer verdict is APPROVE WITH SUGGESTIONS at revision 598ff56c7da41ebdc883dc4fe64b4786002ab04f.
- Review: role=reviewer; artifact=agent_bdd1ba5a25b87a83; revision=d6f440ea3d80b62c30e543484a530f040fa390d8; scope=post-gate-failure hotkeys viewport test adjustment; verdict=REQUEST CHANGES
- Ownership: owner=main; fork_run=none; revision=598ff56c7da41ebdc883dc4fe64b4786002ab04f; scope=update both hotkeys viewport tests for the reserved status row; outcome=completed; commit=598ff56c7da41ebdc883dc4fe64b4786002ab04f
- Review: role=reviewer; artifact=agent_bdd1ba5a25b87a83; revision=598ff56c7da41ebdc883dc4fe64b4786002ab04f; scope=final specification-fidelity and focused-proof review after gate fix; verdict=APPROVE WITH SUGGESTIONS

## Task workflow update - 2026-09-20T22:52:07+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (50.8s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-20-reserve-one-tui-row-for-transient-status-notices/var/reports/qa-20260920-225116-9032-b66c20bc.
- Session/run: 59.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-20T22:52:08+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-20-reserve-one-tui-row-for-transient-status-notices to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-20-reserve-one-tui-row-for-transient-status-notices/var/reports/qa-20260920-225116-9032-b66c20bc.
- Session/run: 59.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-20T22:52:12+00:00
- castor check passed (50.8s).
- Pushed task/2026-09-20-reserve-one-tui-row-for-transient-status-notices to origin.
- Created PR: <url>
- Session/run: 59.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-20T22:52:12+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (50.8s).
- Pushed task/2026-09-20-reserve-one-tui-row-for-transient-status-notices to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/517
- Validation: Focused virtual proof: 8 tests, 233 assertions passed.; PHP CS Fixer: 0 files need changes.; Independent reviewer verdict: APPROVE WITH SUGGESTIONS, no blocking findings.
- Summary: Submitting revision 598ff56c7da41ebdc883dc4fe64b4786002ab04f after correcting both hotkeys viewport proofs exposed by the first full gate and follow-up review.

## Task workflow update - 2026-09-21T00:25:20+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-20-reserve-one-tui-row-for-transient-status-notices into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Status/StatusPanelWidget.php                      |  4 +++-
 tests/Tui/Screen/ChatScreenStatusRowVirtualRenderTest.php | 40 ++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Screen/TuiVirtualInputTest.php                  | 11 ++++++-----
 3 files changed, 49 insertions(+), 6 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-20-reserve-one-tui-row-for-transient-status-notices.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-20-reserve-one-tui-row-for-transient-status-notices.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #517 state: MERGED.
- Summary: GitHub confirms PR #517 merged at 2026-09-21T00:24:48Z with merge commit f3417985e45be87de03eb267fe182f0294d5954b. Integrating the merged task into main before post-merge validation.

## Task workflow update - 2026-09-21T00:26:53+00:00
- Validation: `castor check` passed in 163.5s for QA run `qa-20260921-002530-15013-5e83faed`.; All 11 lanes passed: test 5124 tests/22219 assertions; controller replay 11/175; TUI 9/69; LLM-real 5/30; deptrac, PHPStan, LSP, dead-code, CS, docs, and catalog checks passed.; QA artifact integrity and leak checks passed.; Task worktree removed; integration working tree clean.
- Summary: Post-merge validation passed on integration revision cc66209c36919765af07e68d815a217db207366b. The task worktree was removed and the integration working tree is clean.
