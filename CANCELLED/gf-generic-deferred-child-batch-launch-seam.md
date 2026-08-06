# GF-07: Extract generic deferred child batch launch seam

## Goal
First prerequisite PR for clean fork rebuild. Refactor current main's subagent-only deferred batch launch path into a kind-agnostic reusable launch coordinator without adding fork behavior.

Scope:
- introduce AgentChildLaunchTaskInterface;
- introduce generic deferred child launch plan/intent/failure DTOs and preparation interface;
- extract durable reserve→prepare→ordered runtime start→persist-success/failure orchestration into DeferredAgentChildBatchLaunchCoordinator and DeferredAgentChildBatchRuntimeStartService;
- make DeferredSubagentBatchLaunchService a thin adapter and DeferredSubagentBatchPreparationService the subagent preparation implementation;
- preserve all current subagent runtime, timeout, cancellation, artifact, progress, recovery, and result behavior exactly;
- no artifact-kind DB changes yet; subagent remains the only kind in this PR;
- no fork files, config, prompt, compaction, TUI, export, migrations, or wording changes.

Work from exact origin/main. RED architecture contract before production; existing subagent deferred launch/lifecycle/recovery tests remain authoritative. Castor only, tests <15s, no sleeps/test-only production branches.

## Acceptance criteria
- PR contains only reusable deferred launch seam and current subagent adapter changes
- No fork, TUI, prompt, config, migration, artifact-kind, reasoning, export, or compaction behavior changes
- Subagent execution façade remains thin and public behavior is unchanged
- Reservation, preparation failure, partial start abort, runtime start failure, success persistence, and idempotent redelivery remain covered
- Focused Castor tests, deptrac, phpstan, cs-check pass
- Branch starts at exact origin/main and has a reviewable diff

## Workflow metadata
Status: CANCELLED
Branch: task/gf-generic-deferred-child-batch-launch-seam
Worktree:
Fork run: ky?
PR URL: https://github.com/ineersa/agent-core/pull/293
PR Status: closed
Started: 2026-07-16T13:47:17.209Z
Completed: 2026-07-18T23:06:45Z

## Work log
- Created: 2026-07-16T13:47:11.058Z

## Task workflow update - 2026-07-16T13:47:17.209Z
- Moved TODO → IN-PROGRESS.
- Created branch task/gf-generic-deferred-child-batch-launch-seam.
- Created worktree /home/ineersa/projects/agent-core-worktrees/gf-generic-deferred-child-batch-launch-seam.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/gf-generic-deferred-child-batch-launch-seam.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/gf-generic-deferred-child-batch-launch-seam.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/gf-generic-deferred-child-batch-launch-seam.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/gf-generic-deferred-child-batch-launch-seam.
- Summary: Prerequisite PR 1 for clean fork rebuild. Extract only the generic deferred child batch launch seam; no fork behavior.

## Task workflow update - 2026-07-16T13:51:59.865Z
- Recorded fork run: ky?
- Summary: Implementation Slice A pushed at 522131a7d (RED 23f5c313f). Diff is 17 files, +623/-235: generic launch coordinator/contracts/runtime-start extraction plus thin current-subagent adapter; no fork/config/DB/TUI/compaction changes. Parent diff audit confirms scope except one non-behavioral PHPDoc phrase mentioning 'fork later', which should be removed before review completion. Focused Castor tests and static checks passed per handoff. Full mandatory docs were not read by implementation fork, so parent is not moving task to CODE-REVIEW yet.

## Task workflow update - 2026-07-16T13:52:05.148Z
- Correction: prior update recorded placeholder forkRun `ky?`; synchronous fork response did not expose a run ID. Ignore that placeholder and use commit SHAs 23f5c313f/522131a7d as provenance.

## Task workflow update - 2026-07-16T13:58:47.249Z
- Validation: Reviewer: APPROVE WITH SUGGESTIONS (no blockers); Focused launch/subagent: 11 tests, 82 assertions; Deferred lifecycle/recovery/observation: 21 tests, 298 assertions; castor deptrac: 0 violations; Scoped castor phpstan: 0 errors; Scoped castor cs-check: 0 fixable; Branch local=remote at e48fd0f86; exact origin/main base; clean diff of 17 files +623/-235
- Summary: Reviewer verdict APPROVE WITH SUGGESTIONS, no blockers. Reviewer traced semantic parity for idempotency, context/deadline, reserve ordering, preparation/runtime failure abort+persistence+logging, empty-prepared reconciliation, clock semantics, exception chain, typing, DI and deptrac. Two wording-only suggestions were resolved in e48fd0f86; PR diff now contains no fork references. Remaining suggestion (definition catalog is re-resolved during prepare rather than cached in plan) is non-blocking because current catalog is immutable; no behavior change requested.

## Task workflow update - 2026-07-16T14:04:34.341Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (119.9s).
- Pushed task/gf-generic-deferred-child-batch-launch-seam to origin.
- branch 'task/gf-generic-deferred-child-batch-launch-seam' set up to track 'origin/task/gf-generic-deferred-child-batch-launch-seam'.
- Created PR: https://github.com/ineersa/agent-core/pull/293

## Task workflow update - 2026-07-16T15:42:48.290Z
- Updated PR Status: closed
- Summary: PR #293 closed as superseded by user-approved simpler fork design. The generic extraction renamed/moved too much of the stable subagent launcher. Branch and commits remain available as reference only; do not merge.

## Task workflow update - 2026-07-18T23:06:45Z
- Cancelled as superseded/rejected: PR #293 is closed and the merged fork implementation uses the approved simpler launch path. Worktree removed; branch retained for forensic history.
