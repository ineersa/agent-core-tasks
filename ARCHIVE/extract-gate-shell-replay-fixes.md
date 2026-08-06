# Extract gate-discovered shell and replay lifecycle fixes

## Goal
Split two valid but OM-unrelated fixes out of PR #335's effective diff by landing them independently on main before OM-05 completes.

Scope:
- Standalone shell completion must deterministically wake run_control after AgentEnd so queued follow-ups drain through the existing AdvanceRun path; retain live follow-up phase correlation proof.
- Controller replay E2E ephemeral session IDs must be nonnumeric so they cannot be misclassified as persisted session IDs.
- Reuse the already-reviewed commits from `task/om-05-status-view-settings-docs`: `655410ee6` and `70d53be72`; do not redesign or broaden.
- No OM code, ExtensionApi changes, TUI feature work, timeout changes, or unrelated test cleanup.

Dependency: merge this task before updating OM-05 with latest main, so these patches disappear from PR #335's tree diff while remaining in the product.

## Acceptance criteria
- Standalone shell AgentEnd dispatches an idempotent AdvanceRun wake; queued follow-up live regression remains green
- Replay E2E generated session IDs cannot satisfy ctype_digit
- Focused worker/live/controller-replay tests pass
- Full deterministic castor check passes before CODE-REVIEW
- No OM or unrelated feature files changed

## Workflow metadata
Status: ARCHIVE
Branch: task/extract-gate-shell-replay-fixes
Worktree: /home/ineersa/projects/agent-core-worktrees/extract-gate-shell-replay-fixes
Fork run: lopkxmnkvng3
PR URL: https://github.com/ineersa/agent-core/pull/343
PR Status: merged
Started: 2026-07-31T00:10:34.416Z
Completed: 2026-07-31T02:59:17.142Z

## Work log
- Created: 2026-07-31T00:10:26.867Z

## Task workflow update - 2026-07-31T00:10:34.416Z
- Moved TODO → IN-PROGRESS.
- Created branch task/extract-gate-shell-replay-fixes.
- Created worktree /home/ineersa/projects/agent-core-worktrees/extract-gate-shell-replay-fixes.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/extract-gate-shell-replay-fixes.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/extract-gate-shell-replay-fixes.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/extract-gate-shell-replay-fixes.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/extract-gate-shell-replay-fixes.
- Summary: Claimed as prerequisite for PR #335 review cleanup. Reuse commits 655410ee6 and 70d53be72 only.

## Task workflow update - 2026-07-31T00:18:57.679Z
- Recorded fork run: lopkxmnkvng3
- Validation: castor test --filter=ExecuteShellToolCallWorkerTest — OK 2 / 29; castor test:llm-real --filter=ShellFollowUpLiveE2eTest — OK 2 / 21; castor test:controller-replay — OK 10 / 153; castor test — OK 4362 / 15958; castor deptrac — 0 violations; castor phpstan — 0 errors; castor cs-check — clean
- Summary: Implemented prerequisite by exact cherry-pick onto current local main: shell follow-up wake/correlation at 66d2c4b40 and nonnumeric replay IDs at eebcc89c3. Five expected files only (+181/-10), no OM changes, clean worktree.
- 2026-07-30 — Implementation complete; per workflow stop before reviewer/PR. Branch base is local main 7273be8a2, which is ahead of origin/main by two local merge commits; verify integration/base state during task-to-pr.

## Task workflow update - 2026-07-31T02:30:36.902Z
- Validation: Reviewer verified recorded focused Castor evidence and judged safe for deterministic CODE-REVIEW gate; Worktree clean; diff against task base 7273be8a2 is 5 files +181/-10; Base note: local main has two merge-only commits over origin/main with equivalent tree content; no source diff
- Summary: Fresh-context reviewer APPROVED at eebcc89c3 with zero critical issues and zero issues. Spec fidelity PASS: five expected files only; shell wake, live phase correlation, and replay nonnumeric IDs map exactly. Ponytail: Lean already. Ship. One NTH constant extraction deferred as no value.

## Task workflow update - 2026-07-31T02:32:31.498Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (103.4s).
- Pushed task/extract-gate-shell-replay-fixes to origin.
- branch 'task/extract-gate-shell-replay-fixes' set up to track 'origin/task/extract-gate-shell-replay-fixes'.
- Created PR: https://github.com/ineersa/agent-core/pull/343
- Summary: Exact split of two gate-discovered fixes from OM-05. Reviewer APPROVED; focused unit/live/controller replay/full unit/deptrac/phpstan/cs validation green. No OM changes.

## Task workflow update - 2026-07-31T02:59:17.142Z
- Moved CODE-REVIEW → DONE.
- Merged task/extract-gate-shell-replay-fixes into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                               |   8 ++
 .../CommandHandler/ExecuteShellToolCallWorker.php  |  36 +++++++
 .../ExecuteShellToolCallWorkerTest.php             |  29 +++++-
 .../Controller/E2E/ControllerReplayE2eTestCase.php |   4 +-
 .../Controller/E2E/ShellFollowUpLiveE2eTest.php    | 114 +++++++++++++++++++--
 5 files changed, 181 insertions(+), 10 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/extract-gate-shell-replay-fixes.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/extract-gate-shell-replay-fixes.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #343 state MERGED, mergedAt 2026-07-31T02:58:50Z; Deterministic castor check passed in 103.4s before PR creation
- Summary: PR #343 merged on GitHub at 8ac4fb65cd9b0eb6c53cce89dc0f881587d3741f. Merge into integration checkout and sync main.

## Task workflow update - 2026-08-06T20:58:59.489Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
