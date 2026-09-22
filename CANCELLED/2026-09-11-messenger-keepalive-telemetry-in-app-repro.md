# Hatfield: keepalive-boundary telemetry + in-app reproduction of SIGALRM heartbeat loss (root cause)

## Goal
Goal: instrument the real keepalive boundary in Hatfield, reproduce the heartbeat loss in-app with the real timer, and establish the actual root cause. The upstream `finally` rearm fix is a correct partial fix but only symptom treatment; do not stop there, and do not restore automatic reclaim in production until telemetry and a failure policy exist.

## Current state

- `ConsumerSupervisor` intentionally launches without `--keepalive`; transports use a ~10-year `redeliver_timeout`; abandoned deliveries require manual recovery (umbrella: TODO/2026-09-05-revisit-safe-automatic-recovery-for-abandoned-messenger-deliveries).
- Upstream filing tracked in TODO/2026-09-11-upstream-symfony-console-sigalrm-rearm-issue (draft: `.hatfield/tmp/symfony-console-alarm-rearm-issue.md`).
- Advisory review of the diagnosis: timer-lifecycle defect confirmed (callback rearm skipped when anything throws; non-dispatcher branch rearms first); the SQLite/`database is locked` trigger is plausible but **unproven as the production cause**; rearming in `finally` and heartbeat-failure handling are separate concerns.

## Evidence status (2026-09-11)

- Datadog (2026-09-05, pid 25, full day): 2150 records, **all INFO** — zero warnings/errors; no `LlmStreamObserver::*` / `onDelta threw` / `llm.request.failed` / `llm.provider.stream_error`; zero SQLSTATE/locked/busy/TransportException records for any pid that day. Between the last keepalive (15:42:50.351Z, msg 30242) and the normal completion (15:42:53.579Z) there is exactly one record — the keepalive itself.
- Therefore the production trigger is still unidentified: either the Throwable was swallowed by a catch that does not log, or the failure was not a Throwable at all. Both need observable telemetry.

## Deliverables

1. **Keepalive-boundary telemetry** (info-level, written outside the Messenger SQLite file): callback entry, rearm performed, receiver-call start, receiver-call success, receiver-call exception (full exception chain), callback completion. Fields: pid, process start time, run/session id, message id, queue/transport, sequence number, monotonic duration, DBAL transaction nesting level. Do **not** treat `ConsoleAlarmEvent` as a successful renewal — it is dispatched before the command calls keepalive. Log attempt and outcome separately.
2. **Diagnostic run mode**: isolated session that re-enables `--keepalive` with a short `redeliver_timeout` (and does not disturb normal supervision), plus the strace recipe for the consumer pid: `strace -tt -T -p $PID -e trace=alarm,setitimer,rt_sigaction,rt_sigprocmask -e signal=SIGALRM`.
3. **Deterministic in-app reproductions** (two layers):
   a. Timer lifecycle: one *real* keepalive call throws once while a handler runs; assert the exception follows the expected application path and later ticks still occur (regression for the upstream fix). Use the real alarm timer, never a manually sent SIGALRM (a manual signal can leave the original timer pending and mask a skipped rearm).
   b. SQLite contention: disposable transport DB; a second process holds `BEGIN IMMEDIATE` beyond the configured busy timeout, synchronized by an explicit "lock acquired" acknowledgment; let a real scheduled heartbeat hit it; release; assert later renewals and whether a second consumer receives the same long-running message.
4. **Root-cause report** with the telemetry evidence and a policy recommendation: supervision aware of the last confirmed renewal (not just process liveness); note Doctrine's keepalive UPDATE is keyed by message id with no ownership-generation check, and `Worker::stop()` only sets a stop flag rather than aborting the current handler.

## Constraints

Lowest-layer tests, real timers, no arbitrary sleeps, individual cases ≤10s under Castor load; diagnostics must not depend on the DB under test; no production behavior change (keepalive stays disabled in supervision) until telemetry + policy land. Forks must read the testing skill and tests/AGENTS.md before touching tests.

## Acceptance criteria
- Keepalive-boundary telemetry implemented and observed in a real consumer run (attempt, success, exception with full chain, rearm)
- Both reproductions deterministic under Castor (timer lifecycle; SQLite BEGIN IMMEDIATE contention) with no manual SIGALRM injection
- Root-cause findings recorded: what killed the alarm, or documented evidence that the production trigger remains unidentified
- Policy recommendation for heartbeat-aware supervision + ownership caveats recorded as input for the recovery task

## Workflow metadata
Status: CANCELLED
Branch: task/2026-09-11-messenger-keepalive-telemetry-in-app-repro
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-11-messenger-keepalive-telemetry-in-app-repro
Fork run:
PR URL:
PR Status:
Started: 2026-09-11T15:57:28+00:00
Completed:

## Work log
- Created: 2026-09-11T15:49:06+00:00

## Task workflow update - 2026-09-11T15:57:28+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-11-messenger-keepalive-telemetry-in-app-repro.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-11-messenger-keepalive-telemetry-in-app-repro.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-11-messenger-keepalive-telemetry-in-app-repro.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-11-messenger-keepalive-telemetry-in-app-repro.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-11-messenger-keepalive-telemetry-in-app-repro.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-11-messenger-keepalive-telemetry-in-app-repro/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-11-messenger-keepalive-telemetry-in-app-repro.

## Task workflow update - 2026-09-11T16:00:42+00:00
- Ownership: owner=main; fork_run=none; revision=task/2026-09-11-messenger-keepalive-telemetry-in-app-repro; scope=keepalive-boundary telemetry (4 new classes + DI decoration + diagnostic keepalive flag + redeliver timeout override); outcome=assigned; commit=none
- Ownership: owner=main; fork_run=none; revision=task/2026-09-11-messenger-keepalive-telemetry-in-app-repro; scope=keepalive-boundary telemetry (4 new classes + DI decoration + diagnostic keepalive flag + redeliver timeout override); outcome=completed; commit=none
- Implemented: KeepaliveTelemetry (tick/attempt/renewed/failed/state records, exception chain, DBAL nesting level), KeepaliveTelemetrySubscriber (ConsoleAlarmEvent tick + WorkerMessageReceivedEvent state), KeepaliveTelemetryTransport/TransportFactory (decorates messenger.transport_factory, wraps keepalive-capable transports, logs the real keepalive call and rethrows unchanged), ConsumerSupervisor --keepalive from HATFIELD_MESSENGER_KEEPALIVE, JsonlProcessAgentSessionClient redeliver_timeout from HATFIELD_MESSENGER_REDELIVER_TIMEOUT (stock default unchanged).
- Validation: prod cache:clear OK; debug:container shows messenger.transport_factory = KeepaliveTelemetryTransportFactory; debug:event-dispatcher lists KeepaliveTelemetrySubscriber::onConsoleAlarm; live idle consumer (messenger:consume llm --keepalive=1, probe DSN, 6s) logged messenger.keepalive.tick x5 with ~1000ms cadence. Not yet exercised: attempt/renewed/failed records need an in-flight message — user will reproduce (recipe in task/summary). No tests, no reviewer, per user instruction.

## Task workflow update - 2026-09-11T16:54:35+00:00
- Added keepalive diagnostics to the castor launcher: .castor/run.php now prefixes run:agent with HATFIELD_MESSENGER_KEEPALIVE=2 HATFIELD_MESSENGER_REDELIVER_TIMEOUT=30 (override via exported env; 0 restores stock behavior), prints the effective values, and keeps run:agent-test/run:agent-capture unchanged. Verified: php -l clean; helper returns default/override/disabled values correctly.

## Task workflow update - 2026-09-14T20:50:16+00:00
- Moved IN-PROGRESS → CANCELLED.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-11-messenger-keepalive-telemetry-in-app-repro: ide_close_project returned isError.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-11-messenger-keepalive-telemetry-in-app-repro.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-11-messenger-keepalive-telemetry-in-app-repro.
- Summary: Cancelled by user: SIGALRM heartbeat issue figured out; task no longer needed. Uncommitted telemetry implementation discarded with explicit user approval; worktree removed.
