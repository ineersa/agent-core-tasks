# Fix cancelled tool-batch terminal session corruption

## Goal
A live manual run in the tui-02 worktree exposed a pre-existing AgentCore cancellation race. During a two-tool batch, one tool durably emitted `tool_call_result_received` + `tool_execution_end`, but cancellation arrived before its `message_start`/`message_end`. The sibling tool emitted a cancelled tool message, then `tool_batch_committed(count=1)` and `agent_end(reason=cancelled)` were persisted. Every later follow-up fails `MalformedToolCallSequenceException` because the assistant tool call remains unmatched.

Observed canonical shape in session 2: assistant tool calls at seq 117; first tool start/result/end at 118/120/121 with no tool message; cancellation at 122; sibling cancelled result/end/message at 123-126; batch commit at 127; terminal agent end at 128. Missing tool result ID: `chatcmpl-tool-86ac71396b23bc7c`.

This is separate from tui-02: `TuiSessionSwitchService::cancelCurrentRun()` semantics are unchanged versus its parent, although `/new` and `/resume` use the same cancellation path and can expose the race while tools are active.

Current recovery is also insufficient: `SessionRepairService` returns “No repairable corruption detected” immediately when any terminal `agent_end` exists, so `/repair` cannot recover this exact terminal-but-malformed history.

Fix once at the shared cancellation/tool-batch commit boundary. Keep canonical repair append-only; do not edit or reorder existing events. Do not add a new command or setting.

## Acceptance criteria
- Cancelling during a multi-tool batch cannot leave any assistant tool call without exactly one matching durable tool message before the batch is committed/terminalized.
- The exact observed race—one tool completed before cancellation but its message projection was interrupted while a sibling is cancelled—has the smallest deterministic regression proof at the correct runtime layer.
- `SessionRepairService` recognizes and safely repairs the existing terminal-cancelled malformed sequence when prevention was not present; repair remains append-only, idempotent, and refuses ambiguous pending work.
- After repair, transcript reconstruction passes `AgentMessageToolCallSequenceValidator` and a subsequent user follow-up can proceed.
- Cancellation ordering and batch counts remain correct for successful, failed, and cancelled sibling tools; no duplicate tool results are introduced.
- No ExtensionApi, setting, command, storage schema, or unrelated TUI behavior changes.
- Testing skill and `tests/AGENTS.md` are followed; required focused lanes and final `castor check` pass with no leaked workers.

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
- Created: 2026-08-17T00:21:02.249Z
