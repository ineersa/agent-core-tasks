# Upstream failure-safe Symfony Console SIGALRM rearming

## Goal
Prepare and submit an upstream Symfony issue and, if maintainers agree, a minimal PR for Symfony Console's one-shot SIGALRM rearm gap. Hatfield observed live Messenger consumers permanently losing keepalive alarms while remaining alive; Doctrine then reclaimed active LLM messages at redeliver_timeout and multiple workers executed the same row. The dispatcher path in Symfony Console Application rearms only after ConsoleSignalEvent, ConsoleAlarmEvent, and signalable command handling return, without failure-safe finally. Use `.pi/reports/messenger-keepalive-redelivery-root-cause.md` and Hatfield task `2026-08-24-fix-out-of-order-deepseek-stream-deltas` as source evidence. Do not disclose prompts/session contents or local environment secrets. This is follow-up only; Hatfield will ship a local pre-arm listener first.

## Acceptance criteria
- Create a concise upstream Symfony issue with a minimal framework-level reproducer and clear expected/actual alarm lifecycle.
- Propose the smallest Symfony Console fix: rearm SIGALRM before fallible callback work or unconditionally in finally, preserving exit/signal semantics.
- Add deterministic upstream tests proving an exception/interruption cannot leave the one-shot alarm permanently disarmed; no timing-window sleeps.
- If accepted, open a focused PR and record issue/PR URLs and target Symfony release.
- After an upstream release is adopted, remove Hatfield's local workaround and retain regression coverage.

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
- Created: 2026-08-25T01:40:45.433Z

## Task workflow update - 2026-08-25T01:47:39.439Z
- Summary: Upstream-readiness clarification: do not file a Symfony issue based only on Hatfield's correlation and structural code reading. First capture the exact production callback boundary that fails during the tmux-hidden reproduction. Instrument deterministic phase markers around: signal-closure entry, ConsoleSignalEvent completion, ConsoleAlarmEvent completion, ConsumeMessagesCommand::handleSignal entry, Worker::keepalive completion/error, and final rearm. Preserve exception class/message safely. Then pair that field evidence with a minimal Symfony-only subprocess reproducer. An artificial throwing listener/handleSignal can prove the failure-unsafe one-shot lifecycle, but it must not be presented as Hatfield's actual trigger unless field evidence matches it.

## Task workflow update - 2026-08-25T13:29:40.380Z
- Summary: Do not file a new Symfony Console issue yet. Symfony issue #64912 and PR #64917 already identify/fix a concrete async SIGALRM Doctrine Messenger keepalive race in symfony/doctrine-messenger v8.1.0; fix released in v8.1.2. Agent-core is locked to v8.1.0. Upgrade and rerun production reproduction first. The field capture also showed an idle worker's final callback reaching normal final rearm before alarms ceased, so a separate Console/pcntl issue remains possible but is not yet reproducible or suitable for upstream filing.

## Task workflow update - 2026-08-25T14:36:35.927Z
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Cancelled by user. The concrete production failure was fixed upstream in symfony/doctrine-messenger and adopted via dependency update; no separate Symfony Console upstream issue should be pursued.
