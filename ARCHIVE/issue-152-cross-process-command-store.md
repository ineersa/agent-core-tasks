# #151/#152 residuals: cross-process CommandStore (queue stranding) + sticky Cancelling TUI status (cancel flicker)

## Goal
## Origin

Residual of GitHub issue #152 "Steering/followup queue seems to be broken" (cross-linked to #134 scheduling invariant). The acute symptoms of #152 are already fixed and tested (see "Already fixed" below). This task is the remaining **cross-process** failure that single-process unit tests cannot see.

## What CommandStoreInterface holds

`Ineersa\AgentCore\Contract\CommandStoreInterface` stores queued user-input commands (`PendingCommand`): `runId`, `kind` (steer | follow_up | cancel | human_response | continue), `idempotencyKey`, `payload` (array), `CommandCancellationOptions`. Plus per-command status (`pending | applied | rejected | superseded`) and an insertion-order FIFO index per run. Mailbox cap is 100 per run.

Interface (`src/AgentCore/Contract/CommandStoreInterface.php`):
`enqueue`, `has`, `pending`, `countPending`, `rejectPendingByKind`, `markApplied`, `markRejected`, `markSuperseded`.

## The bug

`CommandStoreInterface` is aliased to `InMemoryCommandStore` (`config/packages/agent_core.yaml:17-18`) — a **per-process PHP array**. But the runtime is multi-process:

- `HeadlessController.php:134-141` launches `run_control`, `llm`, `tool` as **separate** `messenger:consume` child processes (confirmed).
- Transport routing (`config/packages/messenger.yaml:61-65`):
  - `ApplyCommand` → `run_control` → `ApplyCommandHandler` (enqueue site)
  - `ExecuteLlmStep` → `llm` → `LlmStepResult` is handled **sync within the llm worker** (`messenger.yaml:23-25`) → `LlmStepResultHandler` → `CommandMailboxPolicy::applyPendingStopBoundaryCommands` (**stop-boundary drain** site)
  - `AdvanceRun` → `run_control` → `AdvanceRunHandler` (turn-start drain)

So the **enqueue** (run_control process) and the **stop-boundary drain** (llm process) read from **different in-memory stores**. Two failure modes:

1. **Stranding** — A follow_up queued while a run is actively streaming sits in run_control's store. When the LLM stops naturally (no tool calls), the stop-boundary drain runs in the llm process, reads that process's **empty** store → nothing drained → run goes Idle and the queued message is stranded until the user sends another input. This reproduces #152's "I sent message, but run actually took previous one from steering queue" symptom.
2. **Loss on consumer restart** — run_control consumers run with `--time-limit=3600` and have crash-restart via `ConsumerSupervisor` (exponential backoff, up to 3 restarts). Any restart wipes the in-memory queue → queued commands gone forever.

The **turn-start drain** (`AdvanceRunHandler`, also run_control) is fine because it's the same process as enqueue. Only the mid-run-queue → natural-stop path is broken.

## Already fixed (do NOT redo)

- `ApplyCommandHandler::applyCancelCommand` rejects all pending steer/follow_up/continue on cancel — `src/AgentCore/Application/Pipeline/ApplyCommandHandler.php:214-228` (comment cites #152).
- Boundary guard: no `AdvanceRun` dispatched while Running/Cancelling — `ApplyCommandHandler.php:141-158`.
- TUI `queuedFollowUp` grace window during Cancelling + auto-dispatch on RunCancelled — `src/Tui/Runtime/RuntimeEventPoller.php:102-117`.
- Existing tests: `testCancelRejectsPendingSteerAndFollowUp`, `testFollowUpAfterCancelledStillDispatchesAdvanceRun`, etc.

## Fix shape

Swap the `CommandStoreInterface` alias from `InMemoryCommandStore` to a new cross-process-safe implementation. The infrastructure already exists:

- `config/packages/framework.yaml:61` → `cache.app: cache.adapter.doctrine_dbal` — the app cache is **already Doctrine DBAL-backed (shared SQLite), cross-process safe**.
- `DoctrineBundle` registered (`config/bundles.php:9`), DBAL configured (`config/packages/doctrine.yaml`).
- `CacheInterface` autowires `cache.app`; nothing in `src/` uses it yet.
- `Symfony\Component\Lock\LockFactory` (FlockStore) is already wired and used by `SessionRunStore` for atomic CAS — reuse the same pattern.

Implementation notes:
- One cache item per run holding the ordered command list + statuses (small: ≤100 tiny DTOs).
- Guard enqueue/drain/reject with the existing `LockFactory` (FlockStore) keyed by run, mirroring `SessionRunStore::compareAndSwap` (`src/CodingAgent/Session/SessionRunStore.php:79-107`).
- Swap alias in `config/packages/agent_core.yaml:17-18`. Keep `InMemoryCommandStore` available (instantiate directly) for single-process unit tests that don't need cross-process semantics.
- **No backward-compatibility shims, dual-format readers, or migration paths** (per AGENTS.md). Replace the alias; update tests/docs.

## Test thesis

Existing unit tests pass because they're single-process (one `InMemoryCommandStore` instance). They **cannot** catch this bug. The contract to protect: **a command enqueued in one process is visible to pending()/rejectPendingByKind()/mark* in another process.**

Minimal failing repro: instantiate two store instances backed by the same Doctrine DBAL cache pool with separate run-state; enqueue in instance A; assert `pending()` returns it from instance B (currently fails for InMemory, must pass for the new impl). Add a cross-instance reject/mark visibility assertion.

This is runtime/scheduling, not TUI-rendered, so no TmuxHarness E2E proof is required. Default budget: 1 focused contract test for cross-process visibility + keep the existing single-process tests green.

## #151 residual: sticky `Cancelling` status (cosmetic flicker)

The acute #151 "cancel must be instant" is concluded **acceptable** and functional cancel is NOT in scope here: the file-backed `SessionRunStore` re-reads `state.json` on every `get()` (no cache), so the `llm` consumer's `RunCancellationToken` sees `Cancelling` and `LlmPlatformAdapter::consumeStream()` aborts the HTTP stream on the next delta (`src/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapter.php:261-264, 287-289`). BashTool checks the token at the top of every poll iteration (`src/CodingAgent/Tool/BashTool.php:132, 235`) — effectively instant. HTTP cancel is accepted as delta-bound.

The remaining cosmetic glitch: `ActivityStateMachine::transition()` (`src/Tui/Runtime/ActivityStateMachine.php`, ~lines 36-59) is a **pure event-type → state mapper** with no state awareness. Late mid-turn deltas (`AssistantTextDelta`, `TurnStarted`, thinking/tool-call deltas) arriving during the abort window flip `activity` from `Cancelling` back to `Running`, so the TUI flickers "Cancelling" → "Working" → back. Functionally harmless (the stream already aborts), but it is a state-machine correctness smell, and it hardens the UI for any future async-cancel work.

### Fix shape
- Make `Cancelling` sticky: once `activity === Cancelling`, ignore mid-turn streaming deltas (`AssistantTextDelta`, `AssistantThinkingDelta`, tool-call arg/delta events, `TurnStarted`); only cancel-class / terminal events (`OperationCancelled`, `RunCancelled`, `RunFailed`, `RunIdle`) move out of `Cancelling`.
- Must keep the new-run path working: after cancel completes, the queued-follow_up auto-dispatch on `RunCancelled` (`src/Tui/Runtime/RuntimeEventPoller.php:102-117`) starts a fresh run. Gate the stickiness on **event class**, not a blanket lock, so a clean `RunStarted`/`TurnStarted` for the *new* run still transitions `Cancelling` → `Starting`/`Running`.
- The implementer MUST read `src/Tui/Runtime/ActivityStateMachine.php` and `RuntimeEventPoller.php` to confirm the exact transition function and event-type set before changing it. This change is within the TUI layer (no AgentCore dependency), so Deptrac is unaffected.

### Test thesis
Feed `Cancelling` then an `AssistantTextDelta`; assert state stays `Cancelling` (this is the regression). Then feed `RunCancelled`; assert terminal. Then feed a fresh `RunStarted`/`TurnStarted`; assert it moves to `Starting`/`Running` (proving the new-run path is not blocked). This is a TUI state-machine correctness fix, but the status/footer is TUI-rendered, so the mandatory TUI E2E gate applies: add a `#[Group('tui-e2e-replay')]` TmuxHarness proof asserting the footer/status does NOT flip back to "Working" after "Cancelling" during a streamed cancel.

## Out of scope

- #151 **functional** cancel — concluded acceptable: file-backed SessionRunStore re-reads disk on every get(); cancel reaches the llm consumer's RunCancellationToken and aborts the stream on the next delta; BashTool checks the token every poll interval. The cosmetic flicker IS in scope (section above).
- Async / signal-based HTTP cancel (cancel lands *between* deltas, e.g. a Revolt `EventLoop::repeat` ticker or a real HTTP client AbortSignal) — separate future task if delta-bound latency ever becomes a problem.

## Notes

- Related: #134 (scheduling invariant). This task closes the cross-process half of that invariant.
- `RunStoreInterface` and `EventStoreInterface` are already file-backed (SessionRunStore / SessionRunEventStore). `CommandStoreInterface` is the last per-process store.

## Acceptance criteria
- New cross-process CommandStore implementation using cache.app (Doctrine DBAL adapter) or a dedicated table, with atomicity via LockFactory (FlockStore) mirroring SessionRunStore's CAS pattern
- `CommandStoreInterface` alias swapped to the new impl in config/packages/agent_core.yaml; InMemoryCommandStore retained for single-process tests
- No backward-compat shims / dual-format readers / migration paths (per AGENTS.md) — replace the alias, update tests/docs
- Contract test proving cross-process visibility: enqueue in store instance A, pending()/rejectPendingByKind()/mark* visible from a fresh store instance B sharing the same backing store (would fail for InMemoryCommandStore)
- Existing single-process ApplyCommandHandler / AdvanceRunHandler / CommandMailboxPolicy tests remain green
- castor deptrac, castor phpstan, castor cs-check, and castor test (focused) pass
- `ActivityStateMachine::transition()` makes `Cancelling` sticky: mid-turn streaming deltas (`AssistantTextDelta`, `TurnStarted`, thinking/tool-call deltas) no longer regress `activity` to `Running` while a run is aborting
- New-run start after cancel still transitions `Cancelling` → `Starting`/`Running` on a fresh `RunStarted`/`TurnStarted` (stickiness gates on event class, not a blanket lock)
- State-machine unit test: `Cancelling` + `AssistantTextDelta` stays `Cancelling`; + `RunCancelled` → terminal; + fresh `RunStarted` → `Starting`/`Running`
- TUI E2E proof (TmuxHarness replay, `#[Group('tui-e2e-replay')]`) asserting the footer/status does not flip back to "Working" after "Cancelling" during a streamed cancel, per the mandatory TUI E2E gate

## Workflow metadata
Status: DONE
Branch: task/issue-152-cross-process-command-store
Worktree: /home/ineersa/projects/agent-core-worktrees/issue-152-cross-process-command-store
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/171
PR Status: merged
Started: 2026-06-18T21:26:34.282Z
Completed: 2026-06-19T00:41:28.255Z

## Work log
- Created: 2026-06-18T21:00:58.429Z

## Task workflow update - 2026-06-18T21:26:34.282Z
- Moved TODO → IN-PROGRESS.
- Created branch task/issue-152-cross-process-command-store.
- Created worktree /home/ineersa/projects/agent-core-worktrees/issue-152-cross-process-command-store.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/issue-152-cross-process-command-store.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/issue-152-cross-process-command-store.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/issue-152-cross-process-command-store.

## Task workflow update - 2026-06-18T22:13:43.901Z
- Validation: castor test: 2775 tests, 8255 assertions, 0 failures, 0 risky, exit 0 (independently verified by orchestrator); castor phpstan: 0 errors, 0 file_errors; castor deptrac: 0 violations, 0 errors; castor cs-check: clean (0 fixes); castor test:tui --filter=CancelStickiness: 8/8 consecutive passes (independently verified by orchestrator via 8x loop); TUI E2E proof uses real TmuxHarness + replay fixture (tui-tool-call-bash-sleep.json), isolated var/tmp project dir — meets AGENTS.md hard gate
- Summary: Implemented both scopes via 3 forks + 1 orchestrator hygiene fix. 5 commits on branch task/issue-152-cross-process-command-store.

SCOPE 1 — cross-process CommandStore (#152 residuals):
- New CacheCommandStore (src/AgentCore/Infrastructure/Storage/CacheCommandStore.php) backed by cache.app (Doctrine DBAL, shared SQLite) with LockFactory CAS per runId, mirroring SessionRunStore. PendingCommand/CommandCancellationOptions round-trip through the cache pool serializer.
- agent_core.yaml alias swapped InMemoryCommandStore → CacheCommandStore. Existing scheduling tests construct InMemoryCommandStore directly, so they're immune.
- CacheCommandStoreTest: cross-instance visibility contract (enqueue in A, observable from B) — the regression guard that would FAIL for InMemoryCommandStore.

SCOPE 2 — sticky Cancelling footer (#151 cosmetic). Two-layer fix (both required):
- ActivityStateMachine: Cancelling is now sticky — mid-turn deltas (AssistantTextDelta, ToolCallStarted, TurnStarted, etc.) no longer regress activity to Running. Only terminal/cancel events exit Cancelling. New-run path verified safe (RuntimeEventPoller sets activity=Starting on RunCancelled follow-up dispatch).
- TickPollListener: the REAL footer fix. Previously mapped ALL active states (incl. Cancelling) to 'Working...', overwriting CancelListener's 'Cancelling...' on the next tick. Now renders 'Cancelling...' when activity is Cancelling.
- Deterministic TUI E2E proof (CancelStickinessE2eTest): Escape sent after the ToolResult 'Running…' block appears (tool_execution_started boundary), not after '◐ Working' (which appears during the instant-replay LLM step). sleep 15 fixture for a wide window. 8/8 consecutive passes.

Hygiene: CacheCommandStoreTest tearDown now calls restore_exception_handler() (mirrors TraceReplayTest) — cleared 10 PHPUnit 'risky' flags.

NOTE on scout accuracy: the original scout #1 claimed RunCancellationToken always returns false cross-process because of InMemoryRunStore — WRONG (the alias is SessionRunStore, file-backed). Functional cancel already works via SessionRunStore + delta-bound stream abort + BashTool foreground supervision. #151 was cosmetic (footer flicker), #152's acute symptoms were already fixed; this task addressed the residual cross-process CommandStore gap + the footer correctness bug.

## Task workflow update - 2026-06-18T22:14:48.023Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (46.0s).
- Pushed task/issue-152-cross-process-command-store to origin.
- branch 'task/issue-152-cross-process-command-store' set up to track 'origin/task/issue-152-cross-process-command-store'.
- Created PR: https://github.com/ineersa/agent-core/pull/171

## Task workflow update - 2026-06-18T22:21:46.218Z
- Moved CODE-REVIEW → IN-PROGRESS.

## Task workflow update - 2026-06-18T22:21:57.869Z
- Summary: REVIEWER (1st pass): REQUEST CHANGES — 2 bugs in CacheCommandStore. (1) TOCTOU: idempotency has() check is outside the lock in enqueue(); two concurrent enqueues with same idempotency key can both pass has()→false then overwrite+duplicate under lock. Fix: move authoritative has() check inside lock after load(). (2) Silent save failure: pool->save() return value unchecked; enqueue returns true even on persistence failure → silent data loss (the contract violation this store exists to prevent). Fix: check return, throw RuntimeException. Both defeat the cross-process correctness guarantee. Everything else APPROVED (layer boundaries clean, no compat shims, E2E proof real, test hygiene correct, ActivityStateMachine/TickPollListener logic correct). Nits skipped (provider simplification, @covers redundancy) per AGENTS.md no-broaden rule. Moving to task-review-iterate: CODE-REVIEW→IN-PROGRESS, fork the 2 fixes, re-review.
- task-to-pr step 2 (reviewer subagent) was SKIPPED before initial CODE-REVIEW move — caught by user. Ran reviewer after the fact on worktree.
- Reviewer verdict: REQUEST CHANGES — CacheCommandStore TOCTOU on idempotency check + unchecked save() return. Both are concurrency/silent-failure bugs invisible to single-threaded unit tests (hence 2775 green didn't catch them).

## Task workflow update - 2026-06-18T22:36:42.608Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (44.0s).
- Pushed task/issue-152-cross-process-command-store to origin.
- branch 'task/issue-152-cross-process-command-store' set up to track 'origin/task/issue-152-cross-process-command-store'.
- PR already exists: https://github.com/ineersa/agent-core/pull/171
- Summary: REVIEWER 2nd pass: APPROVED. Both blockers resolved in commit 50c593fa3. (1) TOCTOU fixed: authoritative idempotency check now inside the LockFactory lock after load(); pre-lock has() retained as documented non-authoritative hint; order[] write only reached when key absent under lock; return semantics preserved. (2) Silent save fixed: pool->save() return checked, throws RuntimeException; all 5 mutating methods route through single save() chokepoint; try/finally releases lock on throw. Regression test testPersistenceFailureThrowsRuntimeException is well-formed contract test (orchestrator-independently verified it FAILS on reverted code). Reviewer confirmed no lock leak, no remaining dup path, 0 risky. Moving back to CODE-REVIEW (castor check + push + update PR #171).

## Task workflow update - 2026-06-19T00:41:28.255Z
- Moved CODE-REVIEW → DONE.
- Merged task/issue-152-cross-process-command-store into integration checkout.
- Merge made by the 'ort' strategy.
 config/packages/agent_core.yaml                    |   2 +-
 .../Infrastructure/Storage/CacheCommandStore.php   | 266 ++++++++++++++++
 src/Tui/Listener/TickPollListener.php              |  15 +-
 src/Tui/Runtime/ActivityStateMachine.php           |  30 ++
 .../Storage/CacheCommandStoreTest.php              | 343 +++++++++++++++++++++
 tests/Tui/E2E/CancelStickinessE2eTest.php          | 254 +++++++++++++++
 .../Tui/E2E/fixtures/tui-tool-call-bash-sleep.json |  34 ++
 tests/Tui/Runtime/ActivityStateMachineTest.php     | 126 +++++++-
 8 files changed, 1056 insertions(+), 14 deletions(-)
 create mode 100644 src/AgentCore/Infrastructure/Storage/CacheCommandStore.php
 create mode 100644 tests/AgentCore/Infrastructure/Storage/CacheCommandStoreTest.php
 create mode 100644 tests/Tui/E2E/CancelStickinessE2eTest.php
 create mode 100644 tests/Tui/E2E/fixtures/tui-tool-call-bash-sleep.json
- Removed worktree /home/ineersa/projects/agent-core-worktrees/issue-152-cross-process-command-store.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/issue-152-cross-process-command-store.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: Merged by user; smoke-tested; confirmed working. Task complete.
