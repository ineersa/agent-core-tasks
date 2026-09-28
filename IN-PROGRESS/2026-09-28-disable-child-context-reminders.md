# Allow disabling context reminders for forks and subagents

## Goal
Add context_budget_reminders.disable_for_forks and disable_for_subagents, both default false. Each suppresses both early and urgent reminder injection for its child type. Parent reminders and context-limit enforcement remain unchanged. Separate from token usage investigation #536.

## Acceptance criteria
- Both boolean settings load through existing configuration and default to false.
- Fork and subagent suppression are independent and cover early and urgent reminders.
- Parent behavior and context-limit enforcement remain unchanged.
- Update defaults, project settings example, documentation, and focused regression tests.

## Workflow metadata
Status: IN-PROGRESS
Branch: task/2026-09-28-disable-child-context-reminders
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-28-disable-child-context-reminders
Fork run:
PR URL:
PR Status:
Started: 2026-09-28T15:08:30+00:00
Completed:

## Work log
- Created: 2026-09-28T15:08:22+00:00

## Task workflow update - 2026-09-28T15:08:30+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-28-disable-child-context-reminders.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-28-disable-child-context-reminders.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-28-disable-child-context-reminders.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-28-disable-child-context-reminders.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-28-disable-child-context-reminders.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-28-disable-child-context-reminders/.idea.

## Task workflow update - 2026-09-28T15:09:48+00:00
- Ownership: owner=main; fork_run=none; revision=811587184; scope=child reminder settings, hook, documentation and focused regression tests; outcome=assigned; commit=none

## Task workflow update - 2026-09-28T15:13:22+00:00
- Validation: castor test --filter='ContextBudgetReminderHookSubscriberTest|AppConfigTest': PHPUnit report confirms 50 tests, 124 assertions passed. Tool supervision lost final status, so inspected completed lane log and JUnit instead of rerunning. Slowest case 0.019s.; castor phpstan --path=src/CodingAgent/ContextBudget: zero errors.; castor cs-check: no changes required.; castor docs:validate: passed.; git diff --check: passed. Full castor check deferred to CODE-REVIEW transition as required.
- Summary: Implemented both opt-out flags with false defaults. Hook uses existing run_started session kind/child_kind metadata and reuses the existing first-event lookup. Early and urgent reminders are independently suppressed for forks and named subagents. Parent reminders and context-limit enforcement are unchanged. Updated defaults, project example, docs, and regression tests. Committed 307751ede; clean worktree. Ready for task-to-pr, not yet full-gate validated.
- Ownership: owner=main; fork_run=none; revision=811587184; scope=child reminder settings, hook, documentation and focused regression tests; outcome=completed; commit=307751ede
