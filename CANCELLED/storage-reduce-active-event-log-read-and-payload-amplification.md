# STORAGE- Reduce active event-log reads and canonical payload amplification

## Goal
Follow-up to `2026-08-21-audit-session-storage-file-io`. The current audit task only aligns child `latestSequenceFor()` / `reverseFor()` with the existing parent reverse-tail implementation. This task owns the broader work deliberately excluded from that small change: remove full canonical replay from in-process active polling, eliminate child recovery scans from byte zero, and redesign oversized canonical event payloads without compromising resume/repair/history correctness.

Measured snapshot: child logs are 280,042,430 bytes (267.1 MiB) across 229 files. The historical nonterminal-progress persistence bug contributed zero child records. Four canonical types consume 95.02% of child bytes: `message_end` 148,795,187; `tool_execution_end` 48,497,156; `run_started` 43,141,809; `llm_step_completed` 25,664,707.

Structural measurement shows 11,117 child tool results are represented repeatedly: `tool_execution_end.result` totals 44,573,403 bytes; tool `message_end.message.content` totals 44,916,502; `message_end.message.details` totals 96,413,545, including nested result content and raw details. Child `run_started` events total 43,141,809 bytes, of which initial `messages` are 39,947,463 bytes (92.6%); system prompts are only 2,017,805 bytes. Do not attribute this footprint to transient progress or remove data without defining the authoritative replay representation.

Desired invariant: canonical parent/child `events.jsonl` is append-only persistence and is not fully reread during active execution. Full replay remains allowed at resume, repair, explicit history/inspection/export, catalog recovery, and one-time child-live-view snapshot entry.

## Acceptance criteria
- Measure and document actual opens, bytes physically read, decoded records, duration, and cache hit/miss for parent versus child event-store operations before choosing implementation priority. Metrics must be privacy-safe and must not record run IDs, paths, prompts, tool output, or payloads.
- Rework `InProcessAgentSessionClient::events()` so active polling does not call `allFor()` or repeatedly decode canonical history. Use existing in-memory/runtime event delivery and a durable sequence watermark; preserve transient/canonical ordering and resume backfill semantics.
- Replace child `readAfterSeq()` zero-offset recovery scanning with a bounded durable mechanism. Do not use `sequence.cursor` as event-tail truth because allocation can leave valid gaps. Define crash/restart behavior and avoid a new index/sidecar unless measurement proves existing JSONL capabilities insufficient.
- Define the authoritative canonical representation for tool completion lifecycle versus model-visible tool messages. Remove repeated storage of the same logical tool output across `tool_execution_end.result`, `message_end.message.content`, and `message_end.message.details` only after mapping every replay, repair, transcript, export, attachment, notification, and provider-conversion consumer.
- Analyze child `run_started` initial-message retention separately. Preserve exact child resume/replay semantics while avoiding unnecessary duplication of launch context; do not assume the system prompt is the dominant field—the measured initial `messages` field is 92.6% of child run-start bytes.
- Analyze `llm_step_completed` payload retention and duplication with assistant/message state before changing its schema.
- Preserve readability/resume/repair of existing session event logs unless the user explicitly approves dropping legacy sessions. Any schema evolution must use the existing versioning/normalization seams rather than an ad hoc payload walker.
- Do not delete canonical history as a storage optimization. Any compaction/blob/reference strategy requires an explicit durability, portability, cleanup, and corruption-recovery design approved by the user.
- Keep process-mode live-child behavior at one canonical snapshot on entry followed by runtime-pipe events; do not reintroduce repeated child history reads.
- Add deterministic lowest-layer tests for cursor/watermark recovery, append races, sequence gaps, legacy event replay, tool-result replay equivalence, attachments/notifications, repair, and one-time live-view snapshot ordering. No timing windows, arbitrary sleeps, retries-until-green, test-only production APIs, or cases over 10 seconds.
- Update `.pi/reports/session-storage-file-io-audit.md`, `docs/session-storage.md`, and the reusable audit tool with before/after byte and read-amplification evidence.
- Because this touches runtime polling, event storage/replay, and Messenger-visible flow, run focused Castor validation and full `castor check`; inspect JUnit and remediate any individual case over 10 seconds.

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
- Created: 2026-08-25T17:48:06.437Z

## Task workflow update - 2026-08-25T17:57:30.620Z
- Summary: Scope split after user prioritized child payload bloat as ASAP. Canonical payload ownership/deduplication for child `message_end`, `tool_execution_end`, `run_started`, and `llm_step_completed` now belongs exclusively to TODO/storage-optimize-child-canonical-event-payloads.md. This task retains only: reworking InProcessAgentSessionClient active polling so it does not repeatedly call allFor(), and replacing child readAfterSeq() zero-offset recovery scanning with a bounded durable mechanism. Ignore the earlier payload-schema acceptance bullets here in favor of the dedicated task.

## Task workflow update - 2026-08-25T20:35:55.828Z
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Superseded by the narrower task STORAGE-eliminate-remaining-active-event-log-full-scans. Child latestSequenceFor()/reverseFor() were already fixed in PR #430, and canonical payload amplification is owned by STORAGE-optimize-child-canonical-event-payloads. The only remaining concerns are InProcessAgentSessionClient active polling via allFor() and child readAfterSeq() scanning from byte zero.
