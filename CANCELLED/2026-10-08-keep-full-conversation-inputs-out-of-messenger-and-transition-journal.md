# Keep full conversation inputs out of Messenger and transition journals

## Goal
Follow-up to PR #555, review finding #8. ExecuteLlmStep currently embeds the complete current conversation context. Messenger serializes it, and the transition journal serializes it again into .append.work with a 32 MiB encoded-work limit. This remains deferred so PR #555 can merge.

Keep run_control as the sole owner of live RunState. Freeze invocation-specific input in an immutable file using existing project storage and serialization facilities. Send a small identity/checksum reference through Messenger and the journal. The execution worker resolves that exact input without rebuilding RunState or replaying the archive. Do not create a generic execution ledger, receipt protocol, outbox, or a second worker-owned conversation state.

Cover normal LLM launch and explicit repair. Audit compaction and child-start paths for remaining full-history Messenger payloads. Preserve invocation identity and established cleanup boundaries. Measure append latency, serialized journal/message bytes, and peak memory with a representative large context; report the fixture size, method, and before/after measurements.

## Acceptance criteria
- LLM messages and journal work carry only a small immutable-input reference, not conversation history.
- Fresh and restarted execution workers resolve the frozen input without accessing an owner RunState or scanning canonical history.
- Repair preserves the invocation identity and does not accidentally substitute a later conversation context.
- Missing, corrupt, or mismatched input fails visibly; file lifetime and cleanup do not strand unfinished work.
- Audit identifies and removes other full-history execution payloads, including compaction where applicable, without introducing generic receipts or an outbox.
- Focused Castor validation proves correctness and reports large-context append latency, serialized bytes, and peak memory.

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
- Created: 2026-10-08T23:38:57+00:00

## Task workflow update - 2026-10-15T18:34:03+00:00
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Deferred, not implemented. User requested ARCHIVE, but the task tool allows ARCHIVE only from DONE; user approved CANCELLED as the deferred status instead. Conversation payload duplication remains, but its practical cost has not been measured. Built-in image paths carry file references and metadata through Messenger, not image bytes. Reopen if measured queue/journal latency or peak memory warrants the extra file lifecycle, or encoded journal work approaches the 32 MiB limit.
