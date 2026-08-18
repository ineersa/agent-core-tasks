# Disable context compaction for fork and subagent runs

## Goal
Fork/subagent run `687c45f3-c6f1-52ab-87d3-2f921199f092` in parent session 37 timed out at 1800s while an automatic compaction raced with cancellation. Its artifact recorded `context_compaction_started` at ~119,199 estimated tokens and later `context_compaction_failed(reason=stale_result)`, which replayed as a misleading resume error.

Product invariant: child agent runs must never perform context compaction, automatically or manually. Parent sessions retain existing compaction behavior. Enforce this at the compaction policy/orchestration boundary using run kind; do not merely hide events in the TUI. Coordinate scope with the existing compaction-boundary and DRY orchestration tasks without absorbing their broader refactors.

## Acceptance criteria
- Automatic compaction is never scheduled or started for fork or subagent child runs.
- Manual/API-triggered compaction is rejected or unavailable for fork and subagent child runs.
- Parent-session automatic and manual compaction behavior remains unchanged.
- Child cancellation/timeout cannot emit context_compaction_started, context_compaction_failed, or stale-result compaction noise.
- A focused regression reproduces a child run above the normal auto-compaction threshold and proves no compaction request/event occurs.
- Documentation states that child runs complete, hand off, or fail at their context limit rather than compacting.

## Workflow metadata
Status: ARCHIVE
Branch: task/disable-compaction-for-fork-and-subagent-runs
Worktree: /home/ineersa/projects/agent-core-worktrees/disable-compaction-for-fork-and-subagent-runs
Fork run: xm2h3utuya4m
PR URL: https://github.com/ineersa/agent-core/pull/372
PR Status: merged
Started: 2026-08-07T02:50:41.815Z
Completed: 2026-08-13T00:11:23.505Z

## Work log
- Created: 2026-08-07T02:39:24.715Z

## Task workflow update - 2026-08-07T02:50:41.815Z
- Moved TODO → IN-PROGRESS.
- Created branch task/disable-compaction-for-fork-and-subagent-runs.
- Created worktree /home/ineersa/projects/agent-core-worktrees/disable-compaction-for-fork-and-subagent-runs.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/disable-compaction-for-fork-and-subagent-runs.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/disable-compaction-for-fork-and-subagent-runs.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/disable-compaction-for-fork-and-subagent-runs.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/disable-compaction-for-fork-and-subagent-runs.
- Summary: Claimed for implementation. Scope is limited to enforcing the child-run no-compaction invariant at the existing policy/orchestration boundary; broader ownership and DRY orchestration refactors remain in their separate tasks.

## Task workflow update - 2026-08-07T03:00:41.804Z
- Recorded fork run: sv8jyzjbatdf
- Summary: Implementation fork launched in `/home/ineersa/projects/agent-core-worktrees/disable-compaction-for-fork-and-subagent-runs`. Instructions constrain scope to a minimal shared compaction-eligibility policy, all automatic scheduling paths, a defensive CompactRun handler gate, focused regressions, and docs; broader compaction refactors are excluded.

## Task workflow update - 2026-08-07T03:09:44.855Z
- Recorded fork run: sv8jyzjbatdf
- Validation: Filtered 4 new child-run regressions: OK (4); Focused compaction/LLM step tests: OK (69); castor test: OK (4397 tests, 16453 assertions); castor deptrac: OK (0 violations); castor phpstan: OK (0 errors); castor cs-fix followed by castor cs-check: OK; castor check intentionally not run during task-start phase
- Summary: Implementation completed and committed as `3ba0245a45e90693d9472e90ba67325195472c71`. Verified the worktree is clean, HEAD is on `task/disable-compaction-for-fork-and-subagent-runs`, and the 12-file diff matches scope: shared eligibility contract/implementation and DI alias; gates for after-turn, pre-LLM, overflow recovery, and CompactRun handling; focused regressions; compaction docs. Parent and parent-side fork snapshot compaction remain unchanged. Fork confirmed it read root AGENTS.md, the testing skill, and tests/AGENTS.md before test work.

## Task workflow update - 2026-08-07T16:26:52.646Z
- Validation: Reviewer: APPROVED; castor test: OK (4397 tests, 16453 assertions); castor deptrac: OK (0 violations); castor phpstan: OK (0 errors); castor cs-check: OK (0 files fixed)
- Summary: Reviewer APPROVED commit `3ba0245a45e90693d9472e90ba67325195472c71`. Specification fidelity passed: no unmapped external surface or unnecessary complexity; shared policy contract is the minimal AgentCore/CodingAgent boundary seam. Reviewer confirmed all child scheduling/orchestration paths are gated, parent and fork-snapshot behavior is preserved, and focused regressions are adequate. Non-blocking notes: `docs/settings.md` could repeat the canonical child caveat, and child manual command application remains a documented/unavailable silent no-op rather than a new error protocol.

## Task workflow update - 2026-08-07T16:28:59.522Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (113.8s).
- Pushed task/disable-compaction-for-fork-and-subagent-runs to origin.
- branch 'task/disable-compaction-for-fork-and-subagent-runs' set up to track 'origin/task/disable-compaction-for-fork-and-subagent-runs'.
- Created PR: https://github.com/ineersa/agent-core/pull/372
- Validation: Reviewer APPROVED; castor test: OK (4397 tests, 16453 assertions); castor deptrac: OK (0 violations); castor phpstan: OK (0 errors); castor cs-check: OK
- Summary: Reviewer approved and focused Castor validation passed. Moving to CODE-REVIEW for deterministic castor check, push, and PR creation.

## Task workflow update - 2026-08-12T20:16:29.316Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR review iteration started for comments questioning the AgentCore eligibility interface/service wiring and whether the overflow-recovery gate is needed. Will simplify while preserving the finalized invariant that child automatic compaction is never scheduled or started.

## Task workflow update - 2026-08-12T20:21:29.438Z
- Summary: Classified both PR comments as actionable simplification feedback. The child gate in overflow recovery cannot simply disappear because it dispatches `CompactRun` and the acceptance criterion says child automatic compaction is never scheduled; removing parent overflow recovery would also violate unchanged parent behavior. Planned simplification: remove the new eligibility interface, adapter service, and alias; inject the existing `SubagentRunMetadataReader` directly in CodingAgent consumers, and use the existing AgentCore `EventStoreInterface` in the Core overflow scheduler for the minimal child metadata check. No new public surface or behavior.

## Task workflow update - 2026-08-12T20:21:54.549Z
- Recorded fork run: ciigt2v3twaz
- Summary: Review-fix fork launched to remove the eligibility interface/adapter/DI alias, directly inject the canonical child metadata reader in CodingAgent consumers, and retain the required no-scheduling overflow guard using existing AgentCore event-store data.

## Task workflow update - 2026-08-12T22:06:12.701Z
- Recorded fork run: ciigt2v3twaz
- Validation: Re-review: APPROVED; castor test: OK (4397 tests, 16453 assertions); castor deptrac: OK (0 violations); castor phpstan: OK (0 errors); castor cs-check: OK
- Summary: Review feedback addressed in commit `0d74dcb04abe583a3cff1c9ae7ad403de4cd1c52`: deleted the new eligibility interface, adapter service, and DI alias; CodingAgent gates now use the existing `SubagentRunMetadataReader` directly; AgentCore overflow scheduling uses its existing `EventStoreInterface` to avoid dispatching child `CompactRun`. Re-review APPROVED and confirmed both PR comments are substantively resolved. Overflow gate remains because that path truly schedules CompactRun, while the questioned machinery is gone.

## Task workflow update - 2026-08-12T22:10:56.496Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (115.0s).
- Pushed task/disable-compaction-for-fork-and-subagent-runs to origin.
- branch 'task/disable-compaction-for-fork-and-subagent-runs' set up to track 'origin/task/disable-compaction-for-fork-and-subagent-runs'.
- PR already exists: https://github.com/ineersa/agent-core/pull/372
- Validation: Re-review APPROVED; castor test: OK (4397 tests, 16453 assertions); castor deptrac: OK (0 violations); castor phpstan: OK (0 errors); castor cs-check: OK
- Summary: PR feedback addressed and re-review approved. Removed the eligibility interface/service machinery as requested while retaining the overflow no-scheduling gate through existing EventStore data. Focused validation passed; retrying after unrelated repository castor-check lock holder completed.

## Task workflow update - 2026-08-12T22:20:50.603Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User clarified the compaction contract: compaction occurs only manually or automatically at the configured threshold. Provider context-limit rejection is a real visible failure and must never trigger automatic compaction/retry. Remove overflow-recovery compaction entirely, including AgentCore scheduling/classification plumbing and tests; this supersedes the earlier requirement to preserve parent overflow-recovery behavior.

## Task workflow update - 2026-08-12T22:21:16.088Z
- Recorded fork run: xm2h3utuya4m
- Summary: Fork launched to delete overflow-recovery compaction rather than move it across boundaries. Target contract: manual or threshold-based compaction only; provider context-limit errors remain normal visible failures. AgentCore child metadata parsing and all overflow compaction scheduling/retry machinery will be removed.

## Task workflow update - 2026-08-12T22:31:53.333Z
- Recorded fork run: xm2h3utuya4m
- Validation: Re-review: APPROVED; castor test: OK (4396 tests, 16455 assertions); castor deptrac: OK (0 violations); castor phpstan: OK (0 errors); castor cs-check: OK; Repository search: no overflow-recovery scheduling or AgentCore child-compaction symbols remain
- Summary: Latest clarification implemented in `2d40a7271`: overflow-recovery compaction was deleted entirely rather than moved. AgentCore no longer parses child metadata or schedules CompactRun after provider rejection. Context-limit errors follow the ordinary visible non-retryable LLM failure path. Manual/threshold compaction and CodingAgent child gates remain. Re-review APPROVED; both PR concerns are fully resolved.

## Task workflow update - 2026-08-12T22:34:05.694Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (121.4s).
- Pushed task/disable-compaction-for-fork-and-subagent-runs to origin.
- branch 'task/disable-compaction-for-fork-and-subagent-runs' set up to track 'origin/task/disable-compaction-for-fork-and-subagent-runs'.
- PR already exists: https://github.com/ineersa/agent-core/pull/372
- Validation: Re-review APPROVED; castor test: OK (4396 tests, 16455 assertions); castor deptrac: OK (0 violations); castor phpstan: OK (0 errors); castor cs-check: OK
- Summary: Overflow-recovery compaction deleted per final user clarification. Re-review approved and focused validation passed. Updating PR with the simpler contract: manual/threshold compaction only; provider context-limit rejection fails visibly.

## Task workflow update - 2026-08-13T00:11:23.505Z
- Moved CODE-REVIEW → DONE.
- Merged task/disable-compaction-for-fork-and-subagent-runs into integration checkout.
- Auto-merging docs/settings.md
Merge made by the 'ort' strategy.
 docs/compaction.md                                 |  8 +-
 docs/settings.md                                   |  2 +-
 .../Application/Pipeline/LlmStepResultHandler.php  | 73 +-----------------
 .../Application/Pipeline/CompactRunHandler.php     | 10 +++
 .../Compaction/AutoCompactionHookSubscriber.php    | 10 +++
 .../CodingAgentPreLlmCompactionGuard.php           |  7 ++
 .../Pipeline/LlmStepResultHandlerTest.php          | 26 +++++--
 .../Application/Pipeline/CompactRunHandlerTest.php | 90 ++++++++++++++++++++++
 .../AutoCompactionHookSubscriberTest.php           | 75 ++++++++++++++++++
 .../CodingAgentPreLlmCompactionGuardTest.php       | 46 +++++++++++
 10 files changed, 266 insertions(+), 81 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/disable-compaction-for-fork-and-subagent-runs.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/disable-compaction-for-fork-and-subagent-runs.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #372 state: MERGED; Pre-merge deterministic castor check: OK (121.4s); Reviewer: APPROVED; castor test: OK (4396 tests, 16455 assertions); castor deptrac/phpstan/cs-check: OK
- Summary: PR #372 confirmed merged on GitHub at 2026-08-13T00:10:51Z (merge commit `8af433809d0ef24ce734b1a668c3ae0c1f7b309c`). Final behavior: child runs never compact; compaction is manual or threshold-triggered only; provider context-limit rejection remains a visible non-retryable failure.

## Task workflow update - 2026-08-13T00:13:32.998Z
- Validation: LLM_MODE=true castor check: OK; deptrac OK; unit/integration 4410 tests / 16559 assertions; controller replay 12 tests / 165 assertions; TUI 35 tests / 237 assertions; llm-real 13 tests / 144 assertions; phpstan/cs-check OK; QA artifact integrity, leak check, cache cleanup, and llama-proxy cache guard all OK
- Summary: Post-merge validation passed in the integration checkout. Worktree and IDEA exclusions were removed by the DONE transition.

## Task workflow update - 2026-08-14T19:53:39+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
