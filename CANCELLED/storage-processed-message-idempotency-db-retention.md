# STORAGE- Move processed-message idempotency from session files to SQLite

## Goal
Replace the append-only per-run `idempotency.jsonl` implementation with a database-backed processed-message ledger and define a provably safe retention boundary. Current `JsonlIdempotencyStore` creates top-level `sessions/<childRunId>/idempotency.jsonl` directories independently of canonical parent/child storage, scans from byte zero on every lookup, and retains every handled run-control message forever. Measured snapshot: 27,044 records across 219 files; 22,633 records are in 214 UUID child-run directories. Observed scopes are result.tool, command.advance, result.llm, command.apply, command.start, and command.compact.

This task must distinguish run-control message idempotency from execution-worker/tool-batch deduplication. A single latest hash per session/scope must not be assumed safe: parallel/out-of-order results and delayed Messenger redelivery need proof. Before implementation, establish the exact boundary from Messenger ACK/redelivery, RunState/event completion evidence, tool-batch settlement, terminal/quiescent state, child-parent ownership, and explicit session deletion. Do not use TTL, terminal status alone, or “latest key” pruning without deterministic safety evidence.

Source audit: active task `2026-08-21-audit-session-storage-file-io`; report `.pi/reports/session-storage-file-io-audit.md` on its task branch.

## Acceptance criteria
- Produce a per-scope matrix for command.start, command.apply, command.apply_shell, command.advance, result.llm, result.tool, command.compact, and result.compaction covering key construction, concurrency/order, durable completion evidence, duplicate behavior, and earliest provably safe pruning boundary.
- Document the crash windows between state/event commit, processed-message marking, Messenger ACK, retry/redelivery, and separately enqueued duplicate envelopes. Retention must preserve correctness across process crash/restart.
- Finalize with the user one deterministic retention policy. Explicit session deletion is the conservative baseline; any earlier pruning must prove queue/in-flight quiescence or authoritative stale-message rejection. TTL-only and latest-hash-only policies are forbidden.
- Replace `JsonlIdempotencyStore` with an existing-DB-backed implementation using a uniqueness constraint on `(run_id, scope, idempotency_key)` or an equally strong finalized schema. Do not add another database/file when the existing project SQLite ownership boundary applies.
- Associate child-run processed messages with their canonical parent session/artifact lifecycle so child UUIDs no longer create top-level session directories and cleanup can cascade from the parent.
- Define atomic/concurrency behavior for lookup/marking under the existing per-run lock and database uniqueness. Do not claim exactly-once guarantees across file state/event commits and DB writes unless transaction boundaries actually provide them.
- Implement deterministic cleanup at the finalized safe boundary, including crash-recoverable cleanup where lifecycle hooks are not guaranteed to run.
- Address existing `idempotency.jsonl` files with an explicit migration/compatibility decision approved by the user; do not silently delete the nine unreferenced UUID candidates or legacy ledgers.
- Add lowest-layer deterministic tests for concurrent duplicate processing, delayed/out-of-order redelivery, parallel tool results, child-run ownership/cleanup, terminal/resume behavior, and crash-window recovery where representable. No timing-window sleeps or retry-until-green.
- Update session-storage documentation and rerun the privacy-safe storage audit to prove no new idempotency-only top-level directories are created and linear JSONL scans are removed.
- Run focused Castor validation and full `castor check`; inspect JUnit for individual cases over 10 seconds.

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
- Created: 2026-08-25T15:49:52.498Z

## Task workflow update - 2026-08-25T15:54:39.531Z
- Validation: Three read-only scouts traced Messenger ACK/redelivery, per-scope duplicate behavior, and parent/child lifecycle.; No application event fires after Doctrine receiver ACK; WorkerMessageHandledEvent fires before ACK.; Current queue count has no indexed run_id filter and does not prove no worker holds an unACKed envelope.; Per-scope audit found only result.tool has a strong settled-batch completion model; start/apply/shell/advance/compact and compaction result remain unsafe to prune from terminal/latest boundaries.; Explicit session deletion is the only immediately provable semantic boundary, but current deletion must be strengthened because it does not drain queues/deferred/background records and RunMessageProcessor can synthesize queued state when state is missing.
- Summary: Safe-boundary investigation result: terminal session state alone is unsafe, and observed Messenger queue count 0 is also unsafe. Queue count is queue-wide, run_id is buried in serialized envelopes and not indexed, received/in-flight envelopes are not proven absent, and separately enqueued duplicates may share the same application idempotency key. Current handlers for most scopes are not independently duplicate-safe after key pruning. Conservative V1 should retain DB processed-message rows for the canonical parent session lifetime and delete only through a strengthened explicit session teardown that prevents new dispatch, drains/purges parent+child queued/in-flight/deferred work, and prevents late-message resurrection; then cascade rows by parent session. No active-session pruning initially.

## Task workflow update - 2026-08-25T15:56:18.235Z
- Validation: Session terminal alone is insufficient because late/deferred producers can enqueue afterward.; Queue count alone is insufficient because current count is queue-wide/available-oriented and run_id is not indexed.; Terminal generation + producers closed + zero physical queued/delivered rows removes the source of future redelivery, allowing immediate pruning without waiting for session deletion.; Cleanup must be generation-scoped so a persisted session can accept later user turns without retaining prior-turn message identities.
- Summary: Corrected practical retention target after user challenge: do not retain processed keys until session deletion. Target cleanup at a quiescent terminal run/turn generation: first close that generation to further internal production, ensure parent/child/deferred producers for it are settled, then prove the Doctrine transport contains zero physical rows for that run family (including delivered/in-flight rows, not merely available-message count), and delete processed keys for the settled generation. Future user turns use new identities. This requires indexed run-family/generation ownership for queue/in-flight tracking or an equivalent dispatch/ACK ledger; current queue-wide MessageCountAware count and serialized body cannot prove it. Explicit session deletion remains only a fallback, not the desired retention policy.

## Task workflow update - 2026-08-25T17:09:53.855Z
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Superseded by the finalized design: do not move the append-only processed-message receipt ledger to SQLite. Replace global per-message receipts with bounded authoritative RunState/command/tool-batch transition guards; permit normal Messenger redelivery of unfinished execution and use /repair for genuinely stranded in-progress transitions. Replacement task: STORAGE-state-transition-idempotency.
