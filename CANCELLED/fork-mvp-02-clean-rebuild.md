# FORK-MVP-02: Clean fork tool rebuild on current main

## Goal
Replacement for superseded PR #267. Rebuild the fork tool from the authoritative current main tree, porting only the final product behavior and the minimum generic child-run seams required by fork. The old branch remains read-only forensic source; incoming/current main behavior is authoritative everywhere.

Approved architecture:
- fork is a main-agent child run using the existing durable deferred child-run lifecycle;
- no separate synchronous polling/finalization/cancellation machinery;
- create an immutable fork-local copy of the parent session at the fork boundary;
- invoke the canonical configured /compact operation on that isolated copy, with normal threshold/no-op/model/thinking/prompt/boundary semantics;
- never use VirtualCompactionOrchestrator, forced compaction, fork-specific summarization retries/max_tokens, or raw-history fallback;
- parent RunStore/EventStore/session files remain untouched;
- structural compaction no-op still persists the sanitized fork-local launch snapshot through the generic run_messages_replaced replay event;
- continuation is durable via run_control and exactly-once phase guards;
- fork execution returns DeferredToolCompletionOutcome and reuses current generic deferred child lifecycle;
- explicit fork model/thinking override precedence is persisted through reservation/projection/redelivery;
- fork child has canonical main-agent message/context semantics, excludes fork tool, retains subagent/MCP tools according to exact active toolset;
- no TUI/live-view/export/context-statistics work in this replacement unless already present on main and directly required for the tool to function.

Branch hygiene:
- start from exact origin/main tree/ancestry; integration main currently has two duplicate merge commits but an identical tree, so reset the new unpublished task branch to origin/main before implementation;
- do not merge/cherry-pick the old branch wholesale;
- do not port old tests blindly or delete current-main tests;
- port in small reviewable commits with RED test commit before each production contract;
- old branch/PR #267 stays untouched until replacement validation is complete.

Mandatory workflow: read full AGENTS/testing/task-workflow/session-storage/compaction docs; Castor only; each test <15s; no sleeps/test-only production conditionals; commit+push each slice; parent manual smoke before reviewer/CODE-REVIEW.

## Acceptance criteria
- Replacement PR diff contains only fork tool behavior and minimum reusable deferred child-run seams; no legacy TUI/live-view/export changes or obsolete test deletions
- Branch starts at exact current origin/main ancestry and treats main behavior as authoritative
- Fork tool definition/handler returns DeferredToolCompletionOutcome and uses the durable child-run batch lifecycle
- Fork-local parent copy is isolated; parent state/events/messages remain byte-identical
- Canonical AgentRunner::compact runs on fork-local session; normal structural no-op is accepted; no custom/forced fork compaction stack exists
- Sanitized fork-local snapshot is replayable through generic run_messages_replaced semantics
- Continuation message is routed to run_control and cannot execute before ReadyForChildLaunch
- Model and reasoning overrides survive durable reservation, projection, continuation, and redelivery
- Startup migrations are registered and proven from an empty database
- Focused Castor tests, deptrac, phpstan, cs-check, then full Castor gate pass with zero forbidden cache growth
- User manually validates a simple fork before reviewer launch or CODE-REVIEW
- PR #267 is closed as superseded only after replacement branch is validated and replacement PR is available

## Workflow metadata
Status: CANCELLED
Branch: task/fork-mvp-02-clean-rebuild
Worktree:
Fork run:
PR URL:
PR Status:
Started: 2026-07-16T13:41:54.744Z
Completed: 2026-07-18T23:06:45Z

## Work log
- Created: 2026-07-16T13:41:48.166Z

## Task workflow update - 2026-07-16T13:41:54.744Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fork-mvp-02-clean-rebuild.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fork-mvp-02-clean-rebuild.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fork-mvp-02-clean-rebuild.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fork-mvp-02-clean-rebuild.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fork-mvp-02-clean-rebuild.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fork-mvp-02-clean-rebuild.
- Summary: User approved replacing unreviewable PR #267 with a clean rebuild from current main. Worktree must be rebased/reset to exact origin/main before any implementation because local integration main contains two duplicate merge commits with an identical tree.

## Task workflow update - 2026-07-16T13:42:09.936Z
- Summary: Clean replacement worktree created and unpublished task branch reset to exact origin/main at 7ee0d3c64. Tree and ancestry match origin/main exactly; status clean. Old PR #267 branch remains untouched/read-only.

## Task workflow update - 2026-07-16T15:42:48.294Z
- Summary: Architecture simplified by user: fork is the existing subagent child lifecycle with a custom launch preparation only. Fork-specific differences are limited to inherited parent context at start, custom system/contract prompt and handoff format, and configured model/thinking settings. Fork child tool policy excludes BOTH `fork` and `subagent`; nested child agents are unsupported. No nested registry discovery, child-depth exception, background catalog polling, TUI nested routing, generic renaming of the subagent subsystem, or separate fork lifecycle machinery. PRs #267 and #293 are closed; branches preserved only for selective salvage.

## Task workflow update - 2026-07-18T23:06:45Z
- Cancelled as superseded by completed FORK-MVP-03 / PR #295. Empty stale worktree removed.
