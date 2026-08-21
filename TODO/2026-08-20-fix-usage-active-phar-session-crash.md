# [LOW] Prevent /usage from crashing active PHAR sessions when lazy classes become unavailable

## Goal
Bug report from dogfood session #41 on 2026-08-20. Running `/usage` terminated the active agent/TUI instead of degrading locally.

Observed evidence:

- `.hatfield/logs/agent-2026-08-20.log:32947` at `2026-08-20T17:05:13.751711+00:00`: provider quota probe degraded while loading OpenAI credentials, `exception_class=Error`.
- Immediately after, log line 32948 records an unhandled `Error`: `Class "Revolt\\EventLoop\\UncaughtThrowable" not found`, originating from the active session PHAR at `vendor/revolt/event-loop/src/EventLoop/Internal/AbstractDriver.php:428`, through `DriverSuspension.php`, `Symfony Tui.php:196`, `InteractiveMode.php:298`, and `AgentCommand.php`.
- At line 32949 the controller sees stdin EOF and shuts down session #41.
- Two seconds earlier, agent consumer `agent#0` exited 255 (line 32941) with PHAR autoload failures: `Symfony\\Component\\Console\\Event\\ConsoleErrorEvent`, then `Symfony\\Component\\ErrorHandler\\ThrowableUtils`; Composer reported failed includes from `var/tmp/phar/sessions/hatfield.phar`.
- Session state remained `status=running`, turn 1501, version 831 after the parent died, so persisted state did not reflect the terminal failure.
- The active runtime uses the fixed mutable path `var/tmp/phar/sessions/hatfield.phar`. Multiple unrelated lazy classes becoming unavailable across TUI and worker processes strongly suggests the active PHAR was removed, replaced, truncated, or otherwise made inconsistent while still in use. `/usage` exposed the problem by entering the Revolt render/suspension and credential-probe path; do not assume the quota probe itself is the sole root cause.
- Related historical evidence exists in `ARCHIVE/make-castor-qa-safe-to-run-from-hatfield-sessions.md`: deleting an in-use session PHAR previously caused lazy-load failures. The current fixed-path strategy intentionally overwrites on a new build under a serialized-launch assumption; investigate whether builds/cleanup/task workflow can still mutate that path during an active session.

Investigate and fix the root lifecycle/package-integrity issue. Also ensure `/usage` failures remain local and cannot tear down the agent command. Do not merely catch the specific missing-class error or special-case these class names.

## Acceptance criteria
- Reproduce the failure at the correct integration layer: an active PHAR-backed TUI/session invokes `/usage` while the relevant PHAR build/cleanup/replacement path occurs, proving whether fixed-path mutation causes lazy autoload failures.
- An active PHAR-backed session and its controller/consumers continue operating when another build/ensure/cleanup workflow runs; no in-use PHAR file is deleted, truncated, or overwritten.
- `/usage` provider credential/probe degradation renders a safe local result with session totals and does not terminate the TUI, disconnect controller stdin, or leave the session falsely marked running due to parent-process death.
- Add automated regression proof at the lowest layer that can exercise the real PHAR/lazy-autoload lifecycle; virtual handler-only tests are not sufficient as sole proof for this incident.
- Follow the mandatory testing skill and `tests/AGENTS.md`; because this touches TUI runtime/controller lifecycle, complete validation requires `castor check`.

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
- Created: 2026-08-20T17:15:37+00:00
