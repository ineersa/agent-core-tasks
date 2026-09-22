# File upstream symfony/console issue: SIGALRM alarm not re-armed when alarm callback throws

## Goal
Goal: publish the drafted upstream issue on symfony/symfony (component: Console, label: Bug). The user will publish it manually; this task tracks the filing and must end with the issue URL recorded.

## Artifacts (in this checkout)

- Draft issue (paste-ready, follows `.github/ISSUE_TEMPLATE/1_Bug_report.yaml`):
  `.hatfield/tmp/symfony-console-alarm-rearm-issue.md`
- Verified minimal reproducer (Console-only, no Messenger): `.hatfield/tmp/symfony-console-alarm-repro.php`
  Runnable copy with vendor symlink: `.hatfield/tmp/alarm-repro-run/` (`php repro.php`)
- End-to-end no-LLM Messenger repro (worker stays alive, keepalives stop, second consumer reclaims and executes twice): `var/tmp/keepalive-repro/` (`./run.sh control|fault|sqlite-lock`)

## Defect summary

`Application::setAlarmInterval()` arms a one-shot `pcntl_alarm()`. In `Application::doRunCommand()` (dispatcher branch) the SIGALRM closure re-arms via `scheduleAlarm()` only after dispatching `ConsoleEvents::SIGNAL` + `ConsoleEvents::ALARM` and calling `Command::handleSignal()`, with no `try/finally`. Any Throwable on that path permanently disarms the alarm while the process stays alive and keeps working. The non-dispatcher branch re-arms before `handleSignal()` and does not have this defect.

Impact: `messenger:consume --keepalive=N` silently loses its lease heartbeat after one transient keepalive failure (`Connection::keepalive()` deliberately throws `TransportException`); a long handler then exceeds `redeliver_timeout`, a second worker reclaims the row, and the message runs twice concurrently.

## Evidence

- Verified reproducer output (symfony/console 8.1.5, PHP 8.5.5): ticks 1,2,3; tick 3 throws; Throwable surfaces in the interrupted `usleep()` and is caught; no further ticks for the remaining 6 s; process exits 0.
- Plain-PHP probes: a throwable from an async SIGALRM callback (caught by the workload) permanently stops the alarms; re-arming before the throw keeps them firing (`/tmp/alarm-probe-1.php`, `/tmp/alarm-probe-2.php`).
- Field signature (Datadog, 2026-09-05): PID 25's last keepalive 15:42:50.351Z (message 30242), then 9 further deliveries (47 s and 89 s among them) with zero keepalives while the process stayed alive and acked; PID 27 reclaimed `30834` exactly at +60 s (15:47:08.016Z) and both acked. No exception was logged: nothing on the path (Console → `Worker::keepalive()` → transport) logs keepalive failures, and logging level is `info`.

## Duplicate check (2026-09-11)

No existing issue found for the alarm re-arm lifecycle (`gh search issues` for SIGALRM / pcntl_alarm / keepalive / setAlarmInterval). Related but distinct: symfony/symfony#64912 + #64917 (Doctrine keepalive transaction race), #64763 (alarm interval vs symfony/process).

## Supersedes

`CANCELLED/2026-08-25-upstream-symfony-console-failure-safe-sigalrm-rearming.md`. That was cancelled because a *different* upstream bug had been fixed (Doctrine keepalive transaction race); it does not cover this defect.

## Follow-up (not part of this task)

The recovery design for abandoned Messenger deliveries (TODO/2026-09-05-revisit-safe-automatic-recovery-for-abandoned-messenger-deliveries) must not rely on this process-local one-shot heartbeat; owner liveness (pid + process start time) and manual recovery remain the safe baseline until the re-arm is fixed upstream.

## Acceptance criteria
- Issue published on symfony/symfony (Console) using the draft and verified reproducer
- Issue URL recorded in this task's metadata or work log
- Draft path .hatfield/tmp/symfony-console-alarm-rearm-issue.md and repro path .hatfield/tmp/symfony-console-alarm-repro.php updated if the published issue differs

## Workflow metadata
Status: CANCELLED
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-09-11T01:57:47+00:00

## Task workflow update - 2026-09-11T12:32:45+00:00
- 2026-09-11 revision: issue draft now leads the Description with the concrete Hatfield production failure (setup: HTTP-less Symfony 8.1 CLI, Messenger + Doctrine transport on SQLite, `--keepalive=5`, `redeliver_timeout=60`; PID 25 timeline 15:42:50→15:47:08 duplicate with no keepalives and no error; only fallible work on the callback path is the transport keepalive, which throws TransportException by design) before the generic root-cause section. Repro code and Additional Context unchanged.

## Task workflow update - 2026-09-14T20:37:53+00:00
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Cancelled by user: SIGALRM heartbeat issue figured out upstream; no upstream filing needed for now.
