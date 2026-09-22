# Revisit safe automatic recovery for abandoned Messenger deliveries

## Goal
Deferred by user after repeated SIGALRM keepalive failures and duplicate LLM/tool execution. Current direction removes keepalive and elapsed-time reclaim, accepting explicit manual recovery. Related tasks: 2026-08-24-fix-out-of-order-deepseek-stream-deltas and 2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry. Symfony #64912/#64917 fixed a transaction race but later lease-expiry duplicates remain confirmed. Do not re-enable the old mechanism without proof. No implementation requested now.

## Acceptance criteria
- Define abandoned-delivery recovery that cannot mistake a slow live worker for a dead worker.
- Prove concurrent delivery safety and crash recovery deterministically, including controller restarts and long-running LLM/tool calls.
- Avoid async-signal database work; explicitly document external-side-effect crash limitations.
- Preserve manual recovery and require explicit approval before restoring automatic reclaim.

## Workflow metadata
Status: TODO
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-09-05T16:14:13+00:00

## Task workflow update - 2026-09-11T01:57:48+00:00
- 2026-09-11 investigation (keepalive root cause + repro): mechanism proven — Console `--keepalive` is a one-shot `pcntl_alarm()`; `Application::doRunCommand()` re-arms via `scheduleAlarm()` only AFTER `ConsoleEvents::SIGNAL`/`ALARM` dispatch and `Command::handleSignal()` (no `try/finally`, `vendor/symfony/console/Application.php` ~1089-1091). Any Throwable in that callback permanently disarms the heartbeat while the process stays alive and keeps consuming. Plain-PHP probes confirm: throwing callback (caught by workload) → no further alarms; re-arm before the throw → alarms keep firing. Probes: /tmp/alarm-probe-1.php, /tmp/alarm-probe-2.php.
- Field evidence (Datadog, 2026-09-05): PID 25 last keepalive 15:42:50.351Z for message 30242 (which then completed fine); next 9 deliveries (47s, 89s among them) emitted zero keepalives while PID 25 stayed alive and acked; message 30834 was reclaimed by PID 27 at 15:47:08.016Z (+60s lease) and executed concurrently. No exception logged anywhere on the keepalive path — Console/Worker/transport log nothing on failure and `logging.level: info` drops debug breadcrumbs. 2026-08-24/25 logs are not retained in Datadog.
- Repro built (no LLM): `var/tmp/keepalive-repro/` — `./run.sh control|fault|sqlite-lock`. Control passes (keepalives every 2s, no reclaim). Fault: injected keepalive `TransportException` → keepalives freeze, consumer stays alive and finishes the in-flight job, second consumer reclaims after `redeliver_timeout` and the side effect runs twice. SQLite-lock variant reproduces the same with a genuine `SQLSTATE[HY000]: General error: 5 database is locked` from `Connection::keepalive()`. Verified by main independently.
- Upstream issue: no existing symfony/symfony issue for the re-arm lifecycle (checked via `gh search issues` 2026-09-11; related-but-distinct #64912/#64917 Doctrine transaction race and #64763 process interaction). Draft written per Symfony bug template plus verified minimal Console-only reproducer: `.hatfield/tmp/symfony-console-alarm-rearm-issue.md`, `.hatfield/tmp/symfony-console-alarm-repro.php`. Filing tracked in new task `2026-09-11-upstream-symfony-console-sigalrm-rearm-issue`; the old CANCELLED upstream task (2026-08-25) is superseded, not reopened.
- Implication for this task: automatic reclaim must not rely on the old process-local one-shot heartbeat. Safe direction to evaluate: reclaim only when the owning process is provably dead (pid + process start time / boot id recorded with the claim), keep manual recovery as the baseline, document external-side-effect crash limits. Status stays TODO — investigation only, no code changes.

## Task workflow update - 2026-09-11T15:49:06+00:00
- 2026-09-11 follow-up: telemetry + in-app reproduction work split out to TODO/2026-09-11-messenger-keepalive-telemetry-in-app-repro (keepalive-boundary telemetry, diagnostic --keepalive mode + strace recipe, deterministic timer-lifecycle and SQLite BEGIN IMMEDIATE contention tests, root-cause report). Advisory review accepted: the Console rearm defect is real but rearming in finally is a partial fix; the production trigger remains unproven — Datadog shows pid 25 logged only INFO that day (zero warnings/errors), no observer/stream/llm.request.failed records, and no SQLSTATE/locked/busy/TransportException records at all, so the swallowed exception left no trace. Do not restore automatic reclaim until telemetry and a heartbeat-failure policy exist.
