# agent-core-04: Consolidate execution and storage scaffolding

## Goal
## Goal
Bundle the report's low-risk mechanical cleanup into one final AgentCore task, applying only extractions that delete meaningful repetition and preserve lifecycle/error semantics.

## Full report
[`/home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md`](file:///home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md) — AgentCore candidates 8–10 and smaller confirmed items.

## Architecture report evidence

### RunOrchestrator wrappers (candidate 8)
Eight `#[AsMessageHandler]` methods repeat `RunLogContext`, optional tracer checks, root-span setup, attributes, and `RunMessageProcessor::process()`. Shell and compaction-result paths differ in tracer usage and must be verified as intentional before consolidation. Public Messenger entry methods remain required, but can delegate to one private dispatch envelope.

### ToolBatchCollector durable/in-memory branches (candidate 9)
`admitHumanInputSuspension`, `resumeHumanInputAnswer`, and `redriveHumanInputAnswer` each repeat durable-store `mutate()` versus load/null-check/apply/save logic. One private operation can own batch lookup/mutation while preserving both store modes and missing-batch errors.

### Store/observer scaffolding (candidate 10)
- `CacheCommandStore` repeats lock creation, blocking acquisition, load, mutate, save, and release across enqueue/reject/apply/reject operations.
- `LlmPlatformAdapter` repeats guarded stream observer notification, try/catch, and degradation logging for start/delta/end/error.

### Additional confirmed repetition to include only when naturally removed
- Three worker envelopes repeat `RunLogContext` → optional trace span → command-bus dispatch → Messenger exception wrapping; consolidate only if the RunOrchestrator envelope provides a real shared owner.
- `CommandMailboxPolicy` has two near-identical malformed/missing envelope rejection blocks.
- `LlmPlatformAdapter::extractResponseDiagnostics()` contains swallowed `Throwable` catches. Root policy requires explicit logging of intentional local degradation; fix this without logging raw responses, headers, prompts, credentials, or session content.

## Smallest viable direction
- Private methods/callables inside the existing owner are preferred.
- Do not create an execution framework, generic lock transaction abstraction, repository base class, observer bus, or worker superclass.
- Preserve `finally`-based lock release, durable/in-memory behavior, tracer optionality, span names/attributes, command ordering, observer isolation, and degradation logs.
- If an extraction does not reduce net LOC or makes control flow less obvious, record KEEP and leave it.

## Scope boundaries
- No command/event/storage schema, queue, retry, tracing, metrics, observer, model, or user-visible behavior changes.
- No new dependency, public API, interface, abstract base, setting, or compatibility path.
- Do not touch RunState copying or handler notification/callback work owned by agent-core-01/02.

## Test thesis
Retain existing orchestrator, durable/in-memory collector, cache store, and adapter behavior tests. Add only the smallest regression for shared failure/cleanup behavior that is currently uncovered; do not add tests for trivial private delegation.

## Acceptance criteria
- RunOrchestrator public Messenger handlers delegate repeated logging/tracing/processing mechanics to one clear private envelope while preserving every span and context difference.
- ToolBatchCollector durable and in-memory mutation paths share one owner and retain identical missing-batch/error/save behavior.
- CacheCommandStore lock/load/mutate/save/release scaffolding and LlmPlatformAdapter observer notification scaffolding are consolidated where net code decreases.
- Worker/rejection duplication is removed only when it naturally reuses the same owner; otherwise it is recorded as KEEP.
- All caught diagnostic exceptions are explicitly logged as intentional local degradation without exposing sensitive content.
- No generic transaction/observer/execution framework, base class, new interface, dependency, setting, or public API is introduced.
- The final production diff removes more code than it adds and keeps failure/cleanup paths obvious.
- The fork and handoff state that `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` were read and followed.
- Focused Castor tests, `castor test`, `castor test:controller-replay`, `castor deptrac`, `castor phpstan`, `castor cs-check`, and final `castor check` pass.

## Workflow metadata
Status: ARCHIVE
Branch: task/agent-core-04-consolidate-execution-and-storage-scaffolding
Worktree: /home/ineersa/projects/agent-core-worktrees/agent-core-04-consolidate-execution-and-storage-scaffolding
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/395
PR Status: merged
Started: 2026-08-16T18:11:58.758Z
Completed: 2026-08-16T20:07:50.776Z

## Work log
- Created: 2026-08-15T23:28:18.390Z

## Task workflow update - 2026-08-16T18:11:58.758Z
- Moved TODO → IN-PROGRESS.
- Created branch task/agent-core-04-consolidate-execution-and-storage-scaffolding.
- Created worktree /home/ineersa/projects/agent-core-worktrees/agent-core-04-consolidate-execution-and-storage-scaffolding.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/agent-core-04-consolidate-execution-and-storage-scaffolding.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/agent-core-04-consolidate-execution-and-storage-scaffolding.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/agent-core-04-consolidate-execution-and-storage-scaffolding.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/agent-core-04-consolidate-execution-and-storage-scaffolding.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/agent-core-04-consolidate-execution-and-storage-scaffolding/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/agent-core-04-consolidate-execution-and-storage-scaffolding.
- Summary: Starting the next sequential AgentCore architecture task. Scope is the final low-risk mechanical cleanup batch; only extractions with clear net deletion and preserved lifecycle/error semantics will be implemented.

## Task workflow update - 2026-08-16T18:18:19.327Z
- Validation: All three scouts read/followed root/nested AGENTS.md, `.agents/skills/testing/SKILL.md`, and `tests/AGENTS.md`.; Scouts traced production callers, registrations, stores/observer implementations, concurrency/locking semantics, logging fields, and existing tests.; Specification gate: no public/user-facing surface; only required safe diagnostic degradation warnings are newly observable. No unresolved user decision.
- Summary: Three scouts completed the required design gate. Implement: one private RunOrchestrator dispatch envelope preserving the shell no-trace path and current compaction-result trace; one ToolBatchCollector durable/in-memory mutation helper preserving atomic store mutate and three exact errors; narrow CacheCommandStore status helper only (keep enqueue/reject-by-kind inline to preserve race/FIFO clarity); one LlmPlatformAdapter observer guard/isolation helper preserving all existing fields/order; safe warning logs for five swallowed diagnostic stages including Retry-After date parsing; small CommandMailbox rejection helper. KEEP all worker envelopes because they have different owners, nullable outcomes, spans, context, result dispatch, and exception text. Production diff must remain net-negative.

## Task workflow update - 2026-08-16T18:33:13.409Z
- Validation: Fork read/followed root/nested AGENTS.md, `.agents/skills/testing/SKILL.md`, and `tests/AGENTS.md`.; Focused changed tests — PASS (45 tests, 282 assertions).; castor test — PASS (4493 tests, 17659 assertions).; castor test:controller-replay — PASS (12 tests, 165 assertions).; castor test:llm-real — PASS (13 tests, 144 assertions).; castor deptrac — PASS (0 violations, 0 errors).; castor phpstan — PASS (0 errors after fixing one iterable value-type annotation).; castor cs-check — PASS (0 files fixed after one cs-fix formatting pass).; git diff --check — clean.; castor check intentionally deferred to task-to-pr.
- Summary: Implementation committed as c3feea487 on task/agent-core-04-consolidate-execution-and-storage-scaffolding. Consolidated RunOrchestrator dispatch envelope, ToolBatchCollector durable/in-memory human-input mutation, CacheCommandStore status mutation, CommandMailboxPolicy rejection construction, LlmPlatformAdapter observer isolation, and safe diagnostic degradation logging. Worker envelopes, enqueue, and rejectPendingByKind deliberately kept. 9 files changed +582/-301; production +249/-270 (net -21), tests +333/-31. Worktree clean, not pushed, no PR. IDE diagnostics report 0 errors in RunOrchestrator, ToolBatchCollector, and LlmPlatformAdapter.

## Task workflow update - 2026-08-16T19:29:57.586Z
- Validation: Initial and final reviewers read/followed testing skill and tests/AGENTS.md.; Final reviewer verdict: APPROVED.; CacheCommandStoreTest run twice consecutively — PASS both times (11 tests, 28 assertions each).; Final castor test — PASS (4493 tests, 17663 assertions).; Final castor test:controller-replay — PASS (12 tests, 165 assertions).; Final castor test:llm-real — PASS (13 tests, 144 assertions).; Final castor deptrac — PASS (0 violations, 0 errors).; Final castor phpstan — PASS (0 errors).; Final castor cs-check — PASS (0 files fixed).; git diff --check — clean; worktree clean.
- Summary: task-to-pr review complete after two fix iterations. Initial reviewer requested a nonblocking lock-release probe, a smaller RunOrchestrator dispatch signature, and removal of a self-fulfilling observer-order assertion; fixed in 99c1199f7. Re-review found a persistent DBAL cache key making repeated focused runs fail; fixed in 58105c986 and proven by two consecutive passes. Final reviewer verdict: APPROVED. Cumulative commits c3feea487 + 99c1199f7 + 58105c986; production diff remains net-negative.

## Task workflow update - 2026-08-16T19:32:30.693Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (140.4s).
- Pushed task/agent-core-04-consolidate-execution-and-storage-scaffolding to origin.
- branch 'task/agent-core-04-consolidate-execution-and-storage-scaffolding' set up to track 'origin/task/agent-core-04-consolidate-execution-and-storage-scaffolding'.
- Created PR: https://github.com/ineersa/agent-core/pull/395
- Validation: Full unit, controller replay, llm-real, deptrac, phpstan, cs-check, focused rerun-stability checks, IDE diagnostics, and diff-check passed.; Worktree clean.
- Summary: Reviewer APPROVED cumulative commits c3feea487, 99c1199f7, and 58105c986. Consolidated private execution/storage/observer scaffolding with preserved behavior, safe diagnostic degradation logs, net-negative production diff, and explicit worker/enqueue KEEP decisions.

## Task workflow update - 2026-08-16T20:07:50.776Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/agent-core-04-consolidate-execution-and-storage-scaffolding.
- Merged task/agent-core-04-consolidate-execution-and-storage-scaffolding into integration checkout.
- Merge made by the 'ort' strategy.
 .../Application/Handler/ToolBatchCollector.php     | 103 ++++++------
 .../Application/Pipeline/CommandMailboxPolicy.php  |  64 +++-----
 .../Application/Pipeline/RunOrchestrator.php       | 174 ++++++++------------
 .../Infrastructure/Storage/CacheCommandStore.php   |  24 +--
 .../SymfonyAi/LlmPlatformAdapter.php               | 142 +++++++++-------
 .../Handler/ToolBatchCollectorDurableTest.php      |  71 ++++++++
 .../Pipeline/CommandMailboxPolicyTest.php          |  26 +++
 .../Storage/CacheCommandStoreTest.php              | 109 +++++++++----
 .../SymfonyAi/LlmPlatformAdapterTest.php           | 180 ++++++++++++++++++++-
 9 files changed, 592 insertions(+), 301 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/agent-core-04-consolidate-execution-and-storage-scaffolding.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/agent-core-04-consolidate-execution-and-storage-scaffolding.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User confirmed PR #395 merged. Moving task to DONE and syncing integration checkout.

## Task workflow update - 2026-08-16T20:10:40.062Z
- Validation: Post-merge `LLM_MODE=true castor check` PASS: qa-20260816-200814-549290-366420c8.; All 8 lanes green: deptrac; test (4493 tests, 17663 assertions); controller replay (12 tests, 165 assertions); TUI (37 tests, 292 assertions); llm-real (13 tests, 144 assertions); phpstan (0 errors); cs-check; docs:validate.; QA artifact integrity and leak checks passed; llama-proxy cache stable at 326 entries.
- Summary: Post-merge integration validation completed successfully after PR #395 merge. Integration checkout synced and task worktree removed.

## Task workflow update - 2026-08-18T00:06:36.221Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
