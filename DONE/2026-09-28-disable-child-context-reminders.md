# Allow disabling context reminders for forks and subagents

## Goal
Add context_budget_reminders.disable_for_forks and disable_for_subagents, both default false. Each suppresses both early and urgent reminder injection for its child type. Parent reminders and context-limit enforcement remain unchanged. Separate from token usage investigation #536.

## Acceptance criteria
- Both boolean settings load through existing configuration and default to false.
- Fork and subagent suppression are independent and cover early and urgent reminders.
- Parent behavior and context-limit enforcement remain unchanged.
- Update defaults, project settings example, documentation, and focused regression tests.

## Workflow metadata
Status: DONE
Branch: task/2026-09-28-disable-child-context-reminders
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-28-disable-child-context-reminders
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/537
PR Status: merged
Started: 2026-09-28T15:08:30+00:00
Completed: 2026-09-28T15:30:23+00:00

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

## Task workflow update - 2026-09-28T15:22:47+00:00
- Summary: Reviewer agent_5c740974fbf16050 reviewed 307751ede and approved with nonblocking suggestions. Confirmed specification fidelity, launch metadata classification, config hydration, and early/urgent independence tests. Updated stale AppConfig comment in 6017e0f27; no runtime or test changes after reviewed revision. Reviewer explicitly did not require another review for that comment fix. Existing focused validation applies. No unresolved blockers; proceeding to transition-owned full gate.
- Review: role=reviewer; artifact=agent_5c740974fbf16050; revision=307751ede; scope=origin/main...HEAD child context reminder settings and tests; verdict=APPROVE WITH SUGGESTIONS. Final revision 6017e0f27 differs only by the requested AppConfig docblock correction.

## Task workflow update - 2026-09-28T15:24:39+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (100.5s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-28-disable-child-context-reminders/var/reports/qa-20260928-152259-3614-c75e1cf9.
- Session/run: 74.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-28T15:24:41+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-28-disable-child-context-reminders to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-28-disable-child-context-reminders/var/reports/qa-20260928-152259-3614-c75e1cf9.
- Session/run: 74.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-28T15:24:44+00:00
- castor check passed (100.5s).
- Pushed task/2026-09-28-disable-child-context-reminders to origin.
- Created PR: <url>
- Session/run: 74.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-28T15:24:44+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (100.5s).
- Pushed task/2026-09-28-disable-child-context-reminders to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/537

## Task workflow update - 2026-09-28T15:30:23+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-28-disable-child-context-reminders into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/settings.yaml                                                     |  2 ++
 config/hatfield.defaults.yaml                                               |  3 +++
 docs/settings.md                                                            |  4 ++++
 src/CodingAgent/Config/AppConfig.php                                        |  2 +-
 src/CodingAgent/Config/ContextBudgetReminderConfig.php                      |  2 ++
 src/CodingAgent/ContextBudget/ContextBudgetReminderHookSubscriber.php       | 24 +++++++++++++++++++++---
 tests/CodingAgent/Config/AppConfigTest.php                                  | 15 +++++++++++++++
 tests/CodingAgent/ContextBudget/ContextBudgetReminderHookSubscriberTest.php | 49 +++++++++++++++++++++++++++++++++++++++++++++++--
 8 files changed, 95 insertions(+), 6 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-28-disable-child-context-reminders.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-28-disable-child-context-reminders.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR #537 merged as d13bbeb52e86d0644a450a185f7b915bd6d49b9c. Updating integration checkout and cleaning task worktree before post-merge validation.

## Task workflow update - 2026-09-28T15:31:50+00:00
- Updated PR Status: merged
- Validation: castor check: all 11 lanes passed, 162.1s. QA reports: var/reports/qa-20260928-153030-8745-47e3e5a6. Artifact integrity, process leak check, and llama-proxy cache guard passed.; Confirmed clean integration checkout and removed task worktree.
- Summary: Post-merge castor check passed on integration revision 6065c980a7d50cd563c7cc471ec28fd9a03c915e. Git status clean; task worktree removed.
