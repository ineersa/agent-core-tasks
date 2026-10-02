# Fix in-flight Codex WebSocket cancellation

## Goal
## Problem

Session 75 stalled during a Codex WebSocket request on 2026-09-30. The request was prepared at 22:33:13 UTC. Five Escape cancellations reached the backend at 22:43–22:44, but the run remained `cancelling` with an active LLM step and a claimed, unacknowledged delivery. The user recovered the session by restarting with `/repair`. Cancellation did not finish on its own.

The HTTP transport checks the run cancellation token through progress callbacks, including while waiting for stream data. The current Codex WebSocket transport does not have equivalent cancellation checks during pending I/O. The adapter checks cancellation between deltas, but it cannot run that check while the stream iterator waits for the next delta.

This is a confirmed gap in the current source. The exact reason the session's request outlasted its I/O timeout remains unproven. Do not treat timeout cancellation as run cancellation.

## Scope

Make active Codex WebSocket requests observe run cancellation while waiting, including before the first delta and between later deltas. Release the cancelled request's transport resources and let the existing aborted-result path finish cancellation and acknowledge the execution delivery.

Use existing cancellation and async facilities. Verify the owning provider-package integration and make any package changes through its supported source and dependency workflow, not permanent edits under `vendor/`.

Do not change model settings, switch transports, increase timeouts, add worker-kill escalation, or implement automatic abandoned-delivery recovery.

## Entry points

- `src/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapter.php`: invocation scope and stream cancellation checks.
- `src/AgentCore/Infrastructure/SymfonyAi/RunCancellationToken.php`: persisted run cancellation status.
- `src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmCancelAwareHttpClient.php`: existing HTTP cancellation behavior.
- Provider package `ineersa/symfony-ai-openai-codex-platform`: `CodexWebSocketModelClient`, `RawWebSocketResult`, and `AmpCodexWebSocketConnector`.
- `src/AgentCore/Application/Handler/ExecuteLlmStepWorker.php` and `src/AgentCore/Application/Pipeline/LlmStepResultHandler.php`: aborted result and terminal cancellation.

## Validation

Follow the testing skill and `tests/AGENTS.md`. Add deterministic regression proof at the lowest layer that exercises the pending WebSocket operation, rather than only checking a token between mocked deltas. Use explicit readiness barriers, owned resources, and deterministic teardown. Cases must finish within 10 seconds without arbitrary sleeps or timing-window assertions. Run targeted Castor validation during implementation. The CODE-REVIEW transition owns the full `castor check` gate.

## Related work

Keep this separate from `2026-09-29-recover-a-claimed-run-control-delivery-when-its-worker-crashes.md` and `2026-09-05-revisit-safe-automatic-recovery-for-abandoned-messenger-deliveries.md`. This task fixes cancellation of an active provider operation, not recovery after worker death.

## Acceptance criteria
- Cancelling a Codex WebSocket request before its first delta interrupts pending I/O without waiting for another provider event or the idle timeout.
- Cancellation also interrupts waits between deltas. Audit pending connect, send, receive, and message-buffer operations so none bypass run cancellation.
- The worker reports an aborted LLM result through the existing flow, the run reaches `cancelled`, and the execution delivery is acknowledged without `/repair`.
- Cancelled requests release owned sockets and pending async work. Cached connections are not reused after cancellation, and partial output follows the existing aborted-result contract.
- Deterministic regression coverage exercises cancellation during a pending WebSocket wait, with explicit readiness and teardown. Existing HTTP cancellation and successful WebSocket requests remain working.

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
- Created: 2026-09-30T22:55:25+00:00
