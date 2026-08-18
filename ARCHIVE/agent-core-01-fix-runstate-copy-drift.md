# agent-core-01: Fix RunState copy drift and retry-attempt resets

## Goal
## Goal
Fix the proven auto-retry accounting bug caused by field-dropping `RunState` reconstruction, then eliminate the copy-drift pattern without changing intentional state transitions.

## Full report
[`/home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md`](file:///home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md) — AgentCore candidate 1.

## Architecture report evidence
`RunState::with()` already exists in `src/AgentCore/Domain/Run/RunState.php` and explicitly says to prefer it when changing only a subset of fields. The report found 44 manual `new RunState(...)` constructions in AgentCore; 38 omit `retryAttempts`.

The three proven dangerous constructions are:
- two in `Application/Pipeline/RunCommit.php` (assigned-sequence/CAS bump and failure state),
- one in `Domain/Event/EventFactory.php::incrementStateVersion()`.

They preserve `retryableFailure` but implicitly reset `retryAttempts` to zero. The resulting failure chain is:
1. `LlmStepResultHandler` commits `retryableFailure: true` with attempt N,
2. `RunCommit` sequence reconciliation or `EventFactory` stale-result version bump reconstructs state and drops N,
3. the follow-up `Continue` reads zero,
4. retry accounting restarts, delaying or preventing `retriesExhausted`.

Replay can then diverge from the live bumped state because the predicted/reduced state and post-commit reconstructed state preserve different fields.

## Smallest viable direction
- First replace the three proven bug sites with `RunState::with()` and add one regression that fails on the old behavior.
- Audit every remaining manual construction. Convert copy-style state transitions to `with()`.
- Keep true creation boundaries as constructors.
- Where a transition intentionally resets retry state, make `retryAttempts: 0` and related retry fields explicit so future fields cannot silently change semantics.
- Delete field-by-field copy boilerplate; do not introduce a builder, mutable state, reflection copier, or compatibility path.

## Scope boundaries
- No retry-count, retry-delay, exhaustion threshold, event schema, storage, command, or user-visible behavior changes beyond preserving the already-recorded attempt count.
- Do not conflate this with callback/helper cleanup from agent-core-02.
- Preserve immutable state semantics and named-argument readability.

## Test thesis
One focused boundary regression must exercise a retry episode through the assigned-sequence/CAS bump and stale-result/version-bump paths and prove `retryAttempts` survives. Existing handler/reducer tests should continue to prove unchanged state shape; do not add one test per mechanical call-site conversion.

## Acceptance criteria
- The two `RunCommit` constructions and `EventFactory::incrementStateVersion()` preserve `retryAttempts` through `RunState::with()` or an equally direct existing primitive.
- All AgentCore `new RunState(...)` call sites are classified as fresh construction, copy-style transition, or intentional reset.
- Copy-style transitions use `RunState::with()`; intentional retry resets set the relevant values explicitly.
- Live post-commit state and replay-reduced state no longer diverge because a sequence/version bump drops retry accounting.
- A runnable regression fails against the old three-site implementation and proves CAS/assigned-sequence plus stale-result bump preservation.
- No builder, mutable state object, reflection copier, new setting, schema change, or compatibility shim is introduced.
- The fork and handoff state that `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` were read and followed.
- Focused Castor tests, `castor test`, `castor test:controller-replay`, `castor deptrac`, `castor phpstan`, `castor cs-check`, and final `castor check` pass.

## Workflow metadata
Status: ARCHIVE
Branch: task/agent-core-01-fix-runstate-copy-drift
Worktree: /home/ineersa/projects/agent-core-worktrees/agent-core-01-fix-runstate-copy-drift
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/390
PR Status: merged
Started: 2026-08-15T23:31:23.162Z
Completed: 2026-08-16T00:22:29.732Z

## Work log
- Created: 2026-08-15T23:28:18.389Z

## Task workflow update - 2026-08-15T23:31:23.162Z
- Moved TODO → IN-PROGRESS.
- Created branch task/agent-core-01-fix-runstate-copy-drift.
- Created worktree /home/ineersa/projects/agent-core-worktrees/agent-core-01-fix-runstate-copy-drift.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/agent-core-01-fix-runstate-copy-drift.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/agent-core-01-fix-runstate-copy-drift.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/agent-core-01-fix-runstate-copy-drift.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/agent-core-01-fix-runstate-copy-drift.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/agent-core-01-fix-runstate-copy-drift/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/agent-core-01-fix-runstate-copy-drift.
- Summary: Starting agent-core-01 first, sequentially. Scope is the proven retry-attempt preservation bug plus an audited RunState copy-style transition sweep; intentional resets must remain explicit.

## Task workflow update - 2026-08-15T23:59:10.712Z
- Validation: Fork confirmed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` were read and followed.; castor test --filter='RunCommitAfterTurnCommitPersistedSeqTest\|EventFactoryTest\|CommandMailboxPolicyTest' — PASS (14 tests, 84 assertions).; castor test — PASS (4476 tests, 17514 assertions).; castor test:controller-replay — PASS (12 tests, 165 assertions).; castor deptrac — PASS (0 violations, 0 errors).; castor phpstan — PASS (0 errors).; castor cs-check — PASS.; git diff --check HEAD^..HEAD — PASS; worktree clean.; castor check intentionally not run during task-start.
- Summary: Implementation committed as f9ef99882. Audited all production RunState constructions across AgentCore and CodingAgent seams: retained 3 true creation boundaries, converted 57 copy/reset transitions to RunState::with(), made 27 episode resets explicit, preserved 25 counters, and used 5 explicit conditional reset/preserve transitions. Fixed the three reported loss sites plus CommittedRunEventAppender::syncParentLastSeq. Deleted CommandMailboxPolicy::copyState(). Worktree is clean; 18 files, +443/-950. IDE diagnostics report no errors in RunCommit, RunStateReducer, or EventFactory. Stopping at task-start; not pushed and no PR.

## Task workflow update - 2026-08-16T00:17:28.655Z
- Validation: Reviewer read and followed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`.; Final reviewer verdict: APPROVED after commit f60e2abae.; castor test — PASS (4476 tests, 17514 assertions).; castor deptrac — PASS (0 violations, 0 errors).; castor phpstan — PASS (0 errors).; castor cs-check — PASS (0 files fixed).
- Summary: task-to-pr review complete. Initial reviewer requested deletion of two no-op retryAttempts self-overrides in AdvanceRunHandler; follow-up commit f60e2abae removes exactly those lines. Re-review verdict: APPROVED. Final cumulative diff is 18 files, +441/-950, with no external API/schema/settings/command additions.

## Task workflow update - 2026-08-16T00:19:56.771Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (136.0s).
- Pushed task/agent-core-01-fix-runstate-copy-drift to origin.
- branch 'task/agent-core-01-fix-runstate-copy-drift' set up to track 'origin/task/agent-core-01-fix-runstate-copy-drift'.
- Created PR: https://github.com/ineersa/agent-core/pull/390
- Validation: Final focused task-to-pr validation passed: castor test, castor deptrac, castor phpstan, castor cs-check.; Earlier implementation validation passed: castor test:controller-replay.; Worktree clean and git diff --check passed.
- Summary: Reviewer APPROVED final commits f9ef99882 + f60e2abae. RunState copy-style transitions now use the existing with() primitive, intentional retry resets are explicit, and regressions cover assigned-sequence and stale/version bumps.

## Task workflow update - 2026-08-16T00:22:29.732Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/agent-core-01-fix-runstate-copy-drift.
- Merged task/agent-core-01-fix-runstate-copy-drift into integration checkout.
- Auto-merging tests/AgentCore/Application/Pipeline/RunCommitAfterTurnCommitPersistedSeqTest.php
Merge made by the 'ort' strategy.
 .../Application/Pipeline/AdvanceRunHandler.php     | 161 +++------
 .../Application/Pipeline/ApplyCommandHandler.php   | 277 +++++----------
 .../Pipeline/ApplyShellCommandHandler.php          |  28 +-
 .../Application/Pipeline/CommandMailboxPolicy.php  |  32 +-
 .../Application/Pipeline/LlmStepResultHandler.php  | 140 +++-----
 src/AgentCore/Application/Pipeline/RunCommit.php   |  47 +--
 .../Application/Pipeline/ToolCallResultHandler.php |  79 ++---
 .../Application/Replay/RunStateReducer.php         | 383 +++++----------------
 src/AgentCore/Domain/Event/EventFactory.php        |  22 +-
 .../Application/Pipeline/CompactRunHandler.php     |  50 +--
 .../Pipeline/CompactionStepResultHandler.php       |  49 +--
 .../Messenger/WorkerFailedEventSubscriber.php      |  28 +-
 .../Session/CommittedRunEventAppender.php          |  22 +-
 .../Session/Repair/SessionRepairService.php        |  22 +-
 .../Pipeline/CommandMailboxPolicyTest.php          |  31 +-
 .../RunCommitAfterTurnCommitPersistedSeqTest.php   |   9 +
 tests/AgentCore/Domain/Event/EventFactoryTest.php  |   2 +
 .../AgentCore/Support/Builder/RunStateBuilder.php  |   9 +
 18 files changed, 441 insertions(+), 950 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/agent-core-01-fix-runstate-copy-drift.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/agent-core-01-fix-runstate-copy-drift.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR https://github.com/ineersa/agent-core/pull/390 state: MERGED.; Pre-merge integration checkout clean.
- Summary: PR #390 confirmed merged on GitHub at merge commit 6ddcf5012cbcaa85c58db0a58933226fc4e98219. Moving task to DONE and syncing integration checkout.

## Task workflow update - 2026-08-16T00:24:57.156Z
- Validation: LLM_MODE=true castor check — PASS (QA run qa-20260816-002233-211969-ffd49f38).; All eight lanes passed: deptrac, test, controller-replay, TUI replay, llm-real, phpstan, cs-check, docs validation.; QA leak check passed; llama-proxy cache remained stable at 282 entries.
- Summary: Post-merge integration validation complete. Worktree cleanup was performed by the DONE transition.

## Task workflow update - 2026-08-18T00:06:36.247Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
