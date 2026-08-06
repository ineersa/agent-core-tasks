# Make TUI QA tmux teardown timeout-safe and attributable

## Goal
Follow-up from PR #284 deterministic-gate investigation. Current TUI tests clean detached tmux sessions only through PHPUnit tearDown/TmuxHarness destructor. When castor check hard-times out the ParaTest lane, TERM→KILL bypasses both; detached tmux server sessions survive. HATFIELD_QA_RUN_ID is encoded in session names but not propagated into pane/controller descendants, so post-run leak detection cannot attribute them. This task intentionally excludes the immediate timeout budget increase from 120s to 180s, which is being handled in PR #284 to unblock the gate. Do not solve this with broad preflight deletion of all hatfield/tui sessions.

## Acceptance criteria
- Propagate the current HATFIELD_QA_RUN_ID into TUI tmux pane commands and descendants without changing normal runtime sessions
- Introduce exact QA-run-owned timeout teardown for detached tmux sessions; never target unrelated sessions or root-owned processes
- Post-run deterministic leak assertion detects QA-run-owned tmux sessions in addition to QA-tagged processes
- Add a focused regression proving an externally timed-out TUI-like lane removes only its own detached tmux session
- Preserve normal TmuxHarness tearDown/destructor behavior and active HATFIELD_SESSION_ID safety rules
- Document ownership and abnormal-timeout teardown rationale in non-obvious lifecycle comments
- All QA uses Castor; focused tests, deptrac, phpstan, and cs-check pass

## Workflow metadata
Status: DONE
Branch: task/tui-qa-timeout-safe-tmux-teardown
Worktree: /home/ineersa/projects/agent-core-worktrees/tui-qa-timeout-safe-tmux-teardown
Fork run: jpshhzt9tqp6
PR URL: https://github.com/ineersa/agent-core/pull/292
PR Status: merged
Started: 2026-07-15T22:11:49.831Z
Completed: 2026-07-16T00:42:09.607Z

## Work log
- Created: 2026-07-14T20:30:45.000Z

## Task workflow update - 2026-07-15T22:11:49.832Z
- Moved TODO → IN-PROGRESS.
- Created branch task/tui-qa-timeout-safe-tmux-teardown.
- Created worktree /home/ineersa/projects/agent-core-worktrees/tui-qa-timeout-safe-tmux-teardown.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/tui-qa-timeout-safe-tmux-teardown.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/tui-qa-timeout-safe-tmux-teardown.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/tui-qa-timeout-safe-tmux-teardown.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/tui-qa-timeout-safe-tmux-teardown.
- Summary: User approved implementation. Existing task already captures the requested timeout-safe QA tmux ownership, exact-run teardown, leak assertion, and regression proof; reusing it rather than creating a duplicate.

## Task workflow update - 2026-07-15T22:12:58.219Z
- Recorded fork run: macpozqalu5s
- Summary: Implementation fork launched in task worktree. Scope: explicit QA ownership propagation into tmux panes, exact-run Castor tmux finalization, tmux-aware leak assertion, and forced-timeout regression proof. Timeout budget changes and unrelated runtime cleanup are explicitly out of scope.

## Task workflow update - 2026-07-15T22:21:13.407Z
- Recorded fork run: macpozqalu5s
- Validation: Fork confirmed full read of .agents/skills/testing/SKILL.md and tests/AGENTS.md; castor test:timeout-hardstop — PASS (including new Test H); castor test:tui — PASS (37 tests, 193 assertions, ~129s); castor deptrac — PASS (0 violations); castor phpstan — PASS (0 errors); castor cs-check — PASS after castor cs-fix; git status — clean; Commit — 04173add6
- Summary: Implementation completed and committed as 04173add6. Added exact QA-run tmux ownership via @hatfield_qa_run_id plus pane env propagation; bounded Castor inventory/finalization for exact-owned sessions; tmux-aware post-run leak assertion; standalone test:tui ownership/finalization; and timeout-hardstop Test H proving owned cleanup preserves an unrelated control session. Timeout budgets were not changed. Worktree is clean.

## Task workflow update - 2026-07-15T23:45:54.571Z
- Recorded fork run: 7tbhjrnl8h42
- Validation: Reviewer confirmed testing skill and tests/AGENTS.md read; Reviewer verdict — APPROVE WITH SUGGESTIONS; No critical issues; Security review — exact tmux option match and shell quoting approved; Test H contract and fixture cleanup approved
- Summary: Task-to-PR reviewer verdict: APPROVE WITH SUGGESTIONS. No critical/security issues; exact ownership, quoting, active-session safety, and Test H were approved. One lifecycle edge case is being fixed before PR: move exact-run tmux finalization into the lane orchestration finally block so a later llama-proxy cache assertion cannot bypass teardown. Also making process.php explicitly require qa_tmux.php.

## Task workflow update - 2026-07-15T23:59:37.688Z
- Validation: Final reviewer verdict — APPROVE WITH SUGGESTIONS (no issues; only non-blocking observations); castor test — PASS (4421 tests, 14646 assertions); castor deptrac — PASS (0 violations); castor phpstan — PASS (0 errors); castor cs-check — PASS; castor test:tui initial run — transient TuiToolOutputE2eTest startup timeout; exact-run finalizer left no new QA tmux sessions or stale worker candidates; castor test:tui --filter=TuiToolOutputE2eTest — PASS (2 tests, 13 assertions); castor test:tui final full rerun — PASS (37 tests, 193 assertions); Worktree git status — clean; Final HEAD — 286ddaa06
- Summary: Reviewer follow-up commit 286ddaa06 moved exact-run tmux finalization into the lane orchestration finally block, preventing cache-guard failure from bypassing teardown, and made process.php explicitly require qa_tmux.php. Final re-review found no actionable issues and approved the lifecycle ordering, load safety, exact ownership, and scope.

## Task workflow update - 2026-07-16T00:02:32.646Z
- Validation: move_task deterministic castor check — FAIL only test:tui exit 124 at old 120s budget; Post-failure castor clean:cleanup:workers:list — no stale candidates; Post-failure exact QA-run tmux inventory — no surviving sessions for qa-20260715-235946-535312-f50aaeb9; PR #284 — confirmed MERGED; upstream diff includes test:tui timeout 120s→180s
- Summary: First CODE-REVIEW transition gate failed only because this worktree was created from a stale integration base with the old 120s TUI lane timeout. The exact-run tmux finalizer worked: no new QA-owned tmux sessions or stale worker candidates remained after exit 124. PR #284, already merged on GitHub, contains the approved 120s→180s TUI gate budget; the task branch now needs to merge latest origin/main rather than duplicating that out-of-scope change.

## Task workflow update - 2026-07-16T00:02:48.941Z
- Recorded fork run: jpshhzt9tqp6
- Summary: Launched integration fork to merge latest origin/main (including merged PR #284's approved 180s TUI gate budget) into the task branch, preserve timeout-safe tmux changes, and revalidate focused Castor lanes.

## Task workflow update - 2026-07-16T00:16:11.287Z
- Recorded fork run: jpshhzt9tqp6
- Validation: Final post-merge reviewer verdict — APPROVE WITH SUGGESTIONS (no issues; non-blocking observations only); castor test — PASS (4457 tests, 15041 assertions); castor deptrac — PASS (0 violations); castor phpstan — PASS (0 errors); castor cs-check — PASS; castor test:tui — PASS (37 tests, 193 assertions, 126.6s); castor test:timeout-hardstop — PASS (fork validation, Tests A–H); Worktree git status — clean; Final HEAD — c6e5996ed; origin/main ancestry — confirmed
- Summary: Merged latest origin/main non-destructively as c6e5996ed, incorporating merged PR #284's approved 180s TUI gate budget while preserving all timeout-safe tmux lifecycle changes. Final post-merge reviewer found no issues or conflict artifacts. Worktree is clean and origin/main is an ancestor.

## Task workflow update - 2026-07-16T00:18:30.923Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (126.9s).
- Pushed task/tui-qa-timeout-safe-tmux-teardown to origin.
- branch 'task/tui-qa-timeout-safe-tmux-teardown' set up to track 'origin/task/tui-qa-timeout-safe-tmux-teardown'.
- Created PR: https://github.com/ineersa/agent-core/pull/292
- Validation: Reviewer — no actionable issues; castor test — PASS (4457 tests); castor test:tui — PASS (37 tests); castor test:timeout-hardstop — PASS; castor deptrac — PASS; castor phpstan — PASS; castor cs-check — PASS
- Summary: Final implementation at c6e5996ed is synced with latest origin/main, reviewer-approved with no actionable issues, and focused Castor validation is green. Retrying deterministic gate with merged PR #284's 180s TUI lane budget.

## Task workflow update - 2026-07-16T00:18:36.440Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/292
- Updated PR Status: open
- Validation: move_task deterministic castor check — PASS (126.9s); Branch pushed to origin/task/tui-qa-timeout-safe-tmux-teardown; PR created: https://github.com/ineersa/agent-core/pull/292
- Summary: PR #292 created after deterministic CODE-REVIEW gate passed. Final HEAD c6e5996ed; reviewer found no actionable issues.

## Task workflow update - 2026-07-16T00:42:09.607Z
- Moved CODE-REVIEW → DONE.
- Merged task/tui-qa-timeout-safe-tmux-teardown into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/e2e.php               |  12 ++-
 .castor/helpers.php           |  41 +++++++---
 .castor/process.php           |  69 +++++++++++++++-
 .castor/qa_tmux.php           | 179 ++++++++++++++++++++++++++++++++++++++++++
 .castor/tasks.php             |   5 ++
 castor.php                    |   1 +
 phpstan.dist.neon             |   1 +
 tests/AGENTS.md               |   3 +-
 tests/Tui/E2E/TmuxHarness.php |  31 +++++++-
 9 files changed, 324 insertions(+), 18 deletions(-)
 create mode 100644 .castor/qa_tmux.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/tui-qa-timeout-safe-tmux-teardown.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/tui-qa-timeout-safe-tmux-teardown.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #292 state — MERGED; Merge commit — 7ee0d3c64167d3875b7954b68654f71a9964d65e; Pre-merge integration checkout — clean
- Summary: PR #292 confirmed merged on GitHub at 2026-07-16T00:41:39Z with merge commit 7ee0d3c64167d3875b7954b68654f71a9964d65e. Moving task to DONE, syncing integration checkout, and removing task worktree.

## Task workflow update - 2026-07-16T00:49:46.462Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/292
- Updated PR Status: merged
- Validation: PR #292 merged at 2026-07-16T00:41:39Z — merge commit 7ee0d3c64167d3875b7954b68654f71a9964d65e; Post-merge check attempt 1 — only test:tui BashCancelFollowUpE2eTest timing failure; leak check clean; castor test:tui --filter=BashCancelFollowUpE2eTest — PASS (1 test); Post-merge check attempt 2 — only test:controller-replay 90s timeout; leak check clean; castor test:controller-replay — PASS (8 tests, 112 assertions); Final LLM_MODE=true castor check qa-20260716-004726-621951-d5f7008d — PASS (301.4s); Final check lanes: test 4453 PASS; controller-replay 8 PASS; TUI 37 PASS; llm-real 10 PASS; deptrac/phpstan/cs-check PASS; llama-proxy cache guard — stable 356→356; QA artifact integrity — PASS; QA process/tmux leak check — PASS; Integration checkout — clean, main ahead of origin/main by 2 local workflow merge commits; Task worktree directory — removed
- Summary: Task fully closed. PR #292 merged, integration checkout synced, task worktree and IDEA exclusions removed. Final post-merge deterministic gate passed. Two earlier post-merge attempts exposed unrelated timing flakes (BashCancelFollowUp TUI settle timeout, then controller-replay 90s lane timeout); focused reruns passed, both failed QA runs reported no owned process/tmux leaks, and the final full gate was green.
