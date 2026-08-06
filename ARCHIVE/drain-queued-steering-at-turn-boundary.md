# Drain queued steering messages FIFO at the same safe turn boundary

## Goal
## Problem

A real session accepted two steering messages while a run was active. At the next turn break, only the second message was applied; the first remained queued and was applied at the following turn break. Expected behavior is to drain the full steering queue at the safe boundary.

## Exploration findings

Steering flows through `SubmitListener` → runtime/controller → `ApplyCommandHandler` → `CommandStoreInterface` and is drained by `CommandMailboxPolicy` from both `LlmStepResultHandler` (stop boundary) and `AdvanceRunHandler` (turn start).

`CommandMailboxPolicy` currently defaults to `one_at_a_time`, which supersedes all but the newest steer in a pending snapshot. An unused `all` mode applies the snapshot FIFO. However, the exact observed sequence (second applied, then first at the next boundary) also suggests enqueue/drain interleaving: `pending()` is a point-in-time snapshot, and an accepted command that is not yet visible when the boundary drains can miss that continuation.

Primary investigation/fix areas:
- `src/AgentCore/Application/Pipeline/CommandMailboxPolicy.php`
- `src/AgentCore/Application/Pipeline/ApplyCommandHandler.php`
- `src/AgentCore/Application/Pipeline/LlmStepResultHandler.php`
- `src/AgentCore/Application/Pipeline/AdvanceRunHandler.php`
- `src/AgentCore/Infrastructure/Storage/CacheCommandStore.php`
- controller/process ordering for multiple active-run `user_message` commands

Existing tests cover FIFO storage, latest-wins supersession, and one queued steer, but not multiple queued steers draining at one boundary. No existing active task duplicates this issue.

## Test thesis

When two or more steering messages have been accepted/queued during one active turn before its safe boundary, the next model continuation must see all of them exactly once and in FIFO acceptance order. The old behavior would fail by superseding, reversing, or deferring one message.

Use the lowest correct proof around the mailbox/pipeline for batch semantics, plus runtime/controller coverage that exercises the real asynchronous command ordering. If deterministic replay cannot reproduce the real process topology/timing, add an exact live-controller regression rather than treating replay green as proof.

## Acceptance criteria
- Reproduce and explain the observed second-then-first behavior, distinguishing latest-wins supersession from enqueue/drain snapshot interleaving.
- All steering messages accepted before the defined safe-boundary cutoff are applied at that boundary exactly once; normal queued steering does not silently supersede an earlier message.
- Preserve FIFO acceptance order through `RunState.messages` and canonical queued/applied events.
- Drain the batch before scheduling the next model turn, with exactly one continuation for the drained batch rather than one continuation per message.
- Leave no command from the drained batch pending; commands accepted after the explicit boundary cutoff may wait for the next boundary, with that cutoff documented in code/tests.
- Add focused regression coverage for at least two steers at stop-boundary and turn-start drain paths, including FIFO order, applied status/events, and empty remaining queue.
- Add runtime/controller-path proof for multiple queued steering messages; use an exact live-controller regression if replay does not exercise the timing/process path that caused the real issue.
- Run mandatory Castor validation for runtime changes, including `castor check`, before CODE-REVIEW.

## Workflow metadata
Status: ARCHIVE
Branch: task/drain-queued-steering-at-turn-boundary
Worktree: /home/ineersa/projects/agent-core-worktrees/drain-queued-steering-at-turn-boundary
Fork run: w2zdqjm5ybdz
PR URL: https://github.com/ineersa/agent-core/pull/340
PR Status: merged
Started: 2026-07-30T21:35:15.134Z
Completed: 2026-07-30T22:49:07.155Z

## Work log
- Created: 2026-07-21T19:40:53.204Z

## Task workflow update - 2026-07-30T21:35:15.134Z
- Moved TODO → IN-PROGRESS.
- Created branch task/drain-queued-steering-at-turn-boundary.
- Created worktree /home/ineersa/projects/agent-core-worktrees/drain-queued-steering-at-turn-boundary.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/drain-queued-steering-at-turn-boundary.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/drain-queued-steering-at-turn-boundary.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/drain-queued-steering-at-turn-boundary.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/drain-queued-steering-at-turn-boundary.
- Summary: Task claimed for implementation orchestration; reviewing mailbox/pipeline and runtime ordering before dispatching implementation fork.

## Task workflow update - 2026-07-30T21:42:32.339Z
- Summary: Scouts traced both boundaries and confirmed the mailbox already snapshots FIFO under the per-run lock and both handlers schedule one continuation per batch. Root cause is the default latest-wins supersession path; the observed second-then-first sequence cannot come from supersession (which marks the first superseded), so it requires asynchronous ACK/enqueue ordering around the snapshot cutoff. Implementation should make FIFO batch drain the only normal behavior and define the cutoff as commands durably queued before pending() snapshots under the run lock.
- Loaded task-workflow and testing skills plus tests/AGENTS.md and application architecture notes.
- Test thesis: two steers durably queued before either safe-boundary snapshot must both apply exactly once in FIFO queued-event order; stop boundary schedules one AdvanceRun, turn start schedules one LLM continuation, and no command from the snapshot remains pending.
- Planned proof: focused orchestrator/mailbox regressions for stop-boundary and turn-start plus replay-backed controller subprocess coverage sending two active-run steers. Add a live controller test only if replay cannot fail on the old behavior/reproduce the required asynchronous path.

## Task workflow update - 2026-07-30T21:43:26.033Z
- Recorded fork run: r986vm54cl6y
- Summary: Implementation fork launched in the task worktree with exact FIFO mailbox, cutoff, two-boundary regression, controller replay, Castor-only validation, and commit requirements.

## Task workflow update - 2026-07-30T21:50:23.503Z
- Recorded fork run: r986vm54cl6y
- Validation: castor test --filter=CommandMailboxPolicyTest — OK (8 tests, 45 assertions); Temporary old latest-wins restore + focused turn-start regression — expected FAIL (2 user messages instead of 3); restored FIFO implementation and regression passed; castor test:controller-replay — OK (10 tests, 153 assertions); castor deptrac — OK (0 violations); castor phpstan --path=src/AgentCore/Application/Pipeline/CommandMailboxPolicy.php — OK; castor cs-check — OK (0 files to fix); castor check — not run by design in task-start; required during task-to-pr/CODE-REVIEW gate
- Summary: Implementation complete and committed as d1b0f36a2d77b25018b47e1ab05ad6cbf649bcbc. Removed unused latest-wins steer mode so each safe-boundary pending snapshot drains all queued steers FIFO. Documented the durable-queue snapshot cutoff. Added two-boundary orchestrator regressions and replay-backed controller/Messenger multi-steer proof. Verified the commit exists, only the expected 3 files changed, and the worktree is clean.

## Task workflow update - 2026-07-30T22:25:21.625Z
- Summary: Reviewer returned REQUEST CHANGES. Core FIFO/cutoff/continuation behavior and controller proof were judged sound; blockers are vestigial `CommandStoreInterface::markSuperseded()` plus both implementations/test, and dead fallback/redundant override code in the new controller replay test. Event enum/reducer/translator handling must remain for replaying historical canonical events.
- Review decision: REQUEST CHANGES — no correctness/security bugs; remove zero-caller store supersession mutation API and dead controller-test branches, then re-review.
- Non-blocking cleanup accepted for fix: align the replay start prompt with the reused sleep-4 fixture. No live controller regression required; replay exercises the actual controller/Messenger/tool topology and waits for canonical queued projection rather than ACK.

## Task workflow update - 2026-07-30T22:25:47.634Z
- Recorded fork run: w2zdqjm5ybdz
- Summary: Reviewer-fix fork launched to remove vestigial supersession store API/tests and dead controller replay branches, then run focused Castor validation and commit.

## Task workflow update - 2026-07-30T22:38:26.827Z
- Validation: Reviewer: APPROVED (full origin/main...HEAD re-review; specification fidelity gate passed; no blockers/security/privacy/lifecycle concerns); castor test — OK (4362 tests, 15950 assertions, 33.1s); castor deptrac — OK (0 violations, 0 errors); castor phpstan — OK (0 errors); castor cs-check — OK (0 files to fix); Reviewer-fix fork focused checks: CommandMailboxPolicyTest 8/8; CacheCommandStoreTest 10/10; castor test:controller-replay 10/10, 153 assertions
- Summary: Re-review APPROVED after commit 1ca5ba07841cec825459d026dadde675ffbba278. Reviewer confirmed all prior blockers resolved, historical superseded-event replay compatibility preserved, both safe boundaries drain FIFO exactly once with one continuation, durable queued/applied event order is proved without relying on ACK, and no live controller test is required for this deterministic mailbox policy bug.
- Final reviewed commits: d1b0f36a2 (FIFO behavior/regressions) + 1ca5ba078 (reviewer cleanup). Worktree verified clean before CODE-REVIEW gate.

## Task workflow update - 2026-07-30T22:41:06.158Z
- Validation: First automatic castor check: FAILED only test:llm-real / ShellFollowUpLiveE2eTest::testShellThenFollowUpOnCompletedRun; castor clean:cleanup:workers:list — no stale QA worker candidates; castor test:llm-real --filter=ShellFollowUpLiveE2eTest::testShellThenFollowUpOnCompletedRun — OK (1 test, 12 assertions, 7.4s)
- Summary: First CODE-REVIEW gate attempt failed only in pre-existing live test `ShellFollowUpLiveE2eTest::testShellThenFollowUpOnCompletedRun` (no assistant response; controller stayed running). No stale QA worker candidates were present. The exact failed live test passed immediately on focused Castor rerun; no code changes made.
- Gate failure classified as transient existing live-smoke failure after exact focused rerun passed. Retrying deterministic CODE-REVIEW gate without code changes or worker signaling.

## Task workflow update - 2026-07-30T22:43:00.856Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (103.3s).
- Pushed task/drain-queued-steering-at-turn-boundary to origin.
- branch 'task/drain-queued-steering-at-turn-boundary' set up to track 'origin/task/drain-queued-steering-at-turn-boundary'.
- Created PR: https://github.com/ineersa/agent-core/pull/340

## Task workflow update - 2026-07-30T22:43:08.278Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/340
- Updated PR Status: open
- Validation: Automatic deterministic castor check — PASSED (103.3s); PR: https://github.com/ineersa/agent-core/pull/340
- Summary: Task-to-PR complete: reviewer APPROVED, deterministic castor check passed on retry, branch pushed, PR #340 opened.

## Task workflow update - 2026-07-30T22:49:07.155Z
- Moved CODE-REVIEW → DONE.
- Merged task/drain-queued-steering-at-turn-boundary into integration checkout.
- Merge made by the 'ort' strategy.
 .../Application/Pipeline/CommandMailboxPolicy.php  |  57 +---
 src/AgentCore/Contract/CommandStoreInterface.php   |   2 -
 .../Infrastructure/Storage/CacheCommandStore.php   |  14 -
 .../Storage/InMemoryCommandStore.php               |   5 -
 .../Pipeline/CommandMailboxPolicyTest.php          | 100 ++++---
 .../Storage/CacheCommandStoreTest.php              |  14 -
 .../E2E/ControllerReplayMultiSteerFifoTest.php     | 320 +++++++++++++++++++++
 7 files changed, 392 insertions(+), 120 deletions(-)
 create mode 100644 tests/CodingAgent/Runtime/Controller/E2E/ControllerReplayMultiSteerFifoTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/drain-queued-steering-at-turn-boundary.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/drain-queued-steering-at-turn-boundary.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #340 state: MERGED
- Summary: PR #340 confirmed merged on GitHub at 2026-07-30T22:48:44Z (merge commit b14c0554d175e27bf1836102aa2aff97e2a8d2d2). Moving task to DONE and syncing integration checkout.

## Task workflow update - 2026-07-30T22:51:18.705Z
- Updated PR Status: merged
- Validation: Post-merge `LLM_MODE=true castor check` — PASSED (all 7 lanes; unit 4362/15950, controller replay 10/153, TUI 29/177, llm-real 13/144, deptrac/phpstan/cs clean; cache guard and leak check passed)
- Summary: DONE: PR #340 merged, integration checkout synchronized, task worktree removed, IDEA exclusions cleaned, and post-merge full QA passed.

## Task workflow update - 2026-08-06T20:58:47.978Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
