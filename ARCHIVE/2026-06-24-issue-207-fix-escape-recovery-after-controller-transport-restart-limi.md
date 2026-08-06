# ISSUE-207 Fix Escape recovery after controller transport restart-limit failure

## Goal
GitHub issue: https://github.com/ineersa/agent-core/issues/207

After `Runtime transport error: Controller process has crashed too many times (3 restarts in 60s)`, Escape silently no-ops because RuntimeEventPoller sets activity to `Failed`, `RunActivityStateEnum::Failed::isActive()` is false, and CancelListener falls through to clear-editor instead of cancelling or surfacing recovery.

Distinct from #205. Start points: `src/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClient.php`, `src/Tui/Runtime/RuntimeEventPoller.php`, `src/Tui/Runtime/RunActivityStateEnum.php`, `src/Tui/Listener/CancelListener.php`, `src/Tui/Listener/TickPollListener.php`.

## Acceptance criteria
- After fatal runtime transport error, Escape has deterministic visible behavior: cancel fallback, dismiss/recover, or explicit cannot-cancel message; it must not silently no-op.
- If an active run handle exists but controller stdin is unavailable, the TUI surfaces a clear recovery path/error instead of swallowing ESC.
- Regression coverage exists for the Failed/transport-error state; if TUI-visible, a real TmuxHarness E2E or justified runtime/TUI test proves the behavior.
- Castor validation passes; runtime/TUI changes receive `castor check` before CODE-REVIEW.

## Workflow metadata
Status: DONE
Branch: task/2026-06-24-issue-207-fix-escape-recovery-after-controller-transport-restart-limi
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-207-fix-escape-recovery-after-controller-transport-restart-limi
Fork run: tul8q47g6kfb
PR URL: https://github.com/ineersa/agent-core/pull/210
PR Status: merged
Started: 2026-06-24T18:18:51.910Z
Completed: 2026-06-24T19:48:03.101Z

## Work log
- Created: 2026-06-24T18:13:01.279Z

## Task workflow update - 2026-06-24T18:18:51.910Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-06-24-issue-207-fix-escape-recovery-after-controller-transport-restart-limi.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-207-fix-escape-recovery-after-controller-transport-restart-limi.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-207-fix-escape-recovery-after-controller-transport-restart-limi.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-207-fix-escape-recovery-after-controller-transport-restart-limi.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-207-fix-escape-recovery-after-controller-transport-restart-limi.
- Summary: Claimed task-start for orchestration. Loaded task-workflow skill and testing skill; read tests/AGENTS.md before planning runtime/TUI validation.

## Task workflow update - 2026-06-24T18:23:10.405Z
- Validation: Read .agents/skills/task-workflow/SKILL.md; .agents/skills/testing/SKILL.md, and tests/AGENTS.md before planning.; Ran scout subagents for production and test context; output artifact: /home/ineersa/.pi/agent/tmp/2026-06--e9b58f28.txt
- Summary: Scout context gathered. Core bug: RuntimeEventPoller sets activity=Failed and lastRuntimePollError on fatal transport/restart-limit errors; CancelListener only cancels active/Compacting states, so Failed falls through to clearEditor. Existing CancelListenerTest currently asserts Failed does not call cancel. RuntimeEventPollerTest has generic fatal pipe coverage but no controller restart-limit message. TUI E2E patterns available in tests/Tui/E2E/* with TmuxHarness; task is TUI-visible so fork must add a real TmuxHarness proof (replay-backed, no live LLM) or keep task blocked if tmux cannot validate.

## Task workflow update - 2026-06-24T18:24:11.673Z
- Recorded fork run: tul8q47g6kfb
- Summary: Launched implementation fork tul8q47g6kfb in worktree /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-207-fix-escape-recovery-after-controller-transport-restart-limi. Fork instructions require targeted CancelListener fix, CancelListener/RuntimeEventPoller regressions, real TmuxHarness E2E proof for transport restart-limit Escape behavior, focused Castor validation, and a commit. No PR/task transition requested.

## Task workflow update - 2026-06-24T18:46:58.162Z
- Recorded fork run: tul8q47g6kfb
- Validation: Fork reported reading .agents/skills/testing/SKILL.md and tests/AGENTS.md and following Castor/test conventions.; Verified git status clean in worktree.; Verified git show --stat c68349707: 5 files changed, 384 insertions(+), 2 deletions(-).; Verified focused validation reported by fork: castor test --filter=CancelListenerTest OK (14); castor test --filter=RuntimeEventPollerTest OK (19); castor test:tui --filter=TuiTransportRestartLimitEscapeE2eTest OK (1); castor deptrac OK; castor cs-check OK.; Parent did not run castor check and did not move task to CODE-REVIEW, per task-start instructions.
- Summary: Implementation fork completed and was verified by parent. Commit c68349707 (fix(tui): Escape recovery after controller transport restart limit) is HEAD of task branch. Worktree is clean. Commit changes expected files only: src/Tui/Listener/CancelListener.php, tests/Tui/Listener/CancelListenerTest.php, tests/Tui/Runtime/RuntimeEventPollerTest.php, tests/Tui/E2E/TmuxHarness.php, and new tests/Tui/E2E/TuiTransportRestartLimitEscapeE2eTest.php. Verified TUI proof exists: TuiTransportRestartLimitEscapeE2eTest is a real TmuxHarness #[Group('tui-e2e-replay')] E2E that starts process transport, kills current-user controller descendants under the tmux pane until restart-limit, presses Escape, and asserts visible Cancel failed / restart guidance. Note for task-to-pr: diff against origin/main also includes pre-existing commit 91650cd59 (composer.json php-http discovery plugin change) already in local integration history; implementation commit itself does not modify composer.json.

## Task workflow update - 2026-06-24T19:02:46.024Z
- Validation: Review decision before fixes: APPROVED WITH SUGGESTIONS, no blocking correctness issues, TmuxHarness E2E proof accepted as satisfying the TUI proof gate.; Fork validation for hygiene fixes: castor test --filter=CancelListenerTest OK (14); castor test:tui --filter=TuiTransportRestartLimitEscapeE2eTest OK (1); castor cs-check OK.; Parent verified git diff --stat origin/main...HEAD now lists only src/Tui/Listener/CancelListener.php, tests/Tui/E2E/TmuxHarness.php, tests/Tui/E2E/TuiTransportRestartLimitEscapeE2eTest.php, tests/Tui/Listener/CancelListenerTest.php, tests/Tui/Runtime/RuntimeEventPollerTest.php.
- Summary: Reviewer returned APPROVED WITH SUGGESTIONS. Launched implementation fork to address actionable hygiene items. Fork added commits 17fe52280 (non-destructive revert of unrelated composer.json change from PR diff) and 680eadc3c (comment clarity + E2E hygiene). Parent verified worktree remains clean and diff vs origin/main now excludes composer.json: 5 ISSUE-207 files changed.

## Task workflow update - 2026-06-24T19:10:46.689Z
- Validation: Reviewer subagent final verdict: APPROVED.; Focused local validation on worktree: castor test OK (3537 tests, 11171 assertions, 18.7s).; castor deptrac OK (violations=0, errors=0).; castor phpstan OK (errors=0, file_errors=0).; castor cs-check OK (files_fixed=0).; castor test:tui OK (17 tests, 141 assertions, 82.9s).; Worktree clean after validation; no current-user stale worktree workers found before CODE-REVIEW move.; Current HEAD: 680eadc3c; diff vs origin/main: 5 ISSUE-207 files, no composer.json.
- Summary: Final reviewer decision: APPROVED on current HEAD 680eadc3c. Reviewer confirmed no issues, accepted the real TmuxHarness replay-backed E2E proof, verified process-kill scoping respects current-user/root-owned process constraints, and confirmed PR diff is clean of unrelated composer.json after non-destructive revert.

## Task workflow update - 2026-06-24T19:13:53.065Z
- Validation: move_task(to=CODE-REVIEW) automatic castor check failed: test:llm-real exit code 1; failure was Ineersa\CodingAgent\Tests\Runtime\Controller\E2E\CompactionLiveSmokeTest::testCompactionLiveSmoke expected compaction.completed, observed command.ack/status.updated/compaction.started.; Focused rerun: castor test:llm-real --filter=CompactionLiveSmokeTest OK (1 test, 20 assertions, 50.9s).
- Summary: First CODE-REVIEW transition attempt failed during automatic castor check in the test:llm-real lane: CompactionLiveSmokeTest did not observe compaction.completed within that run. No code changes made. Focused rerun of the failed live smoke test passed, indicating transient/live-lane flake rather than an implementation regression.

## Task workflow update - 2026-06-24T19:15:32.077Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (88.9s).
- Pushed task/2026-06-24-issue-207-fix-escape-recovery-after-controller-transport-restart-limi to origin.
- branch 'task/2026-06-24-issue-207-fix-escape-recovery-after-controller-transport-restart-limi' set up to track 'origin/task/2026-06-24-issue-207-fix-escape-recovery-after-controller-transport-restart-limi'.
- Created PR: https://github.com/ineersa/agent-core/pull/210

## Task workflow update - 2026-06-24T19:15:39.612Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/210
- Updated PR Status: open
- Validation: move_task(to=CODE-REVIEW) castor check passed (88.9s).; Pushed branch task/2026-06-24-issue-207-fix-escape-recovery-after-controller-transport-restart-limi to origin.; Created PR #210: https://github.com/ineersa/agent-core/pull/210
- Summary: Task moved to CODE-REVIEW. Automatic deterministic castor check passed on retry (88.9s), branch pushed to origin, PR created: https://github.com/ineersa/agent-core/pull/210.

## Task workflow update - 2026-06-24T19:48:03.101Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-06-24-issue-207-fix-escape-recovery-after-controller-transport-restart-limi into integration checkout.
- Merge made by the 'ort' strategy.
 composer.json                                      |   7 +-
 src/Tui/Listener/CancelListener.php                |  13 +-
 tests/Tui/E2E/TmuxHarness.php                      |  23 ++
 .../E2E/TuiTransportRestartLimitEscapeE2eTest.php  | 291 +++++++++++++++++++++
 tests/Tui/Listener/CancelListenerTest.php          |  39 ++-
 tests/Tui/Runtime/RuntimeEventPollerTest.php       |  25 ++
 6 files changed, 390 insertions(+), 8 deletions(-)
 create mode 100644 tests/Tui/E2E/TuiTransportRestartLimitEscapeE2eTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-207-fix-escape-recovery-after-controller-transport-restart-limi.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-207-fix-escape-recovery-after-controller-transport-restart-limi.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Verified PR #210 state=MERGED via gh pr view; mergedAt=2026-06-24T19:47:29Z.
- Summary: PR #210 was merged on GitHub (merge commit ff69d131af72271059f917a96d5cc27b9e288877). Moving task to DONE and syncing integration checkout.
