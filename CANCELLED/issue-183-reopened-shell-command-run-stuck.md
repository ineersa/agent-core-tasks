# Issue #183: Reconcile reopened TUI shell-command stuck/run-startup failure

## Goal
GitHub issue: https://github.com/ineersa/agent-core/issues/183

Open issue reports `!` shell commands can leave the interactive TUI/runtime session stuck after tool approval/startup timeout (`Agent process did not emit run_started event within 15s`), with cancellation/state cleanup problems and leaked current-user controller/consumer processes.

There is historical task context in `ARCHIVE/issue-183-shell-command-run-stuck.md`, but no active TODO/IN-PROGRESS/CODE-REVIEW/DONE task. Treat this as a reopened/current-state reconciliation task: reproduce on current main, compare against the archived fix/residual notes, then either fix the remaining bug or close the GitHub issue with evidence if already resolved.

## Acceptance criteria
- Current-main behavior for issue #183 is reproduced or disproven with a real runtime/TUI path appropriate to the bug, not only replay tests.
- If the bug persists, shell-only and follow-up-after-shell flows reach terminal/recoverable state and do not leave unresolved pending tool calls or stuck cancellation.
- Controller/consumer lifecycle is correct; no leaked current-user Hatfield workers after the focused reproduction/validation.
- Automated regression proof is added at the lowest correct layer; for live runtime discrepancies, include the appropriate live/controller/TUI validation per project testing rules.
- GitHub issue #183 is updated or closed with the validation evidence.

## Workflow metadata
Status: TODO
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-07-07T18:08:35.713Z

## Task workflow update - 2026-07-22T15:07:46.319Z
- Summary: Cancelled by user on 2026-07-22. Do not start or implement this task. It remains physically in TODO only because the current task workflow has no CANCELLED/ARCHIVE transition; move it once `task-workflow-toon-archive-cancelled` adds that status.
- User decision: cancel this task; the reopened issue investigation is no longer planned.
- Workflow limitation: `move_task` currently supports only TODO, IN-PROGRESS, CODE-REVIEW, and DONE. Do not misclassify cancellation as DONE or manually move the external task file.
