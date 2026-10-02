# Hatfield: owner-local execution state and bounded event replay

**Status:** Proposed implementation specification, version 1.0.

**Review baseline:** `ineersa/agent-core`, PR #541, commit `932ce85002171172385a72ad3fdb84aaaad6682c`, compared with `4f107b0340b52f4837b66fd5cf562f6ecd2f9dff`.

This is a proposed replacement design, not a statement that the reviewed implementation provides these guarantees. Numbers below are initial engineering defaults to validate, not measured safe operating limits.

## 1. Decisions and scope

The run-control worker owns current execution state. Other processes receive immutable invocation inputs, narrow operational metadata, or transcript updates. Full `RunState` and transcript graphs must not be stored in `cache.app`.

Completed children must not remain resident in the run-control worker. Opening a child view reconstructs a bounded transcript on demand; it does not reconstruct execution state or restart the child. Compaction must release obsolete message and rendering graphs after the transition finishes.

### Required invariants

1. Only the session's exclusively owned run-control worker appends canonical parent and child run events. Other processes submit commands/results. Maintenance tools acquire the same ownership with the runtime stopped.
2. `ActiveRunContext::requireLoaded()` is an in-memory lookup: no I/O, normalization, TTL, or implicit recovery.
3. A missing in-memory state means not loaded, intentionally released, or recovery required. It never means a new run.
4. A full archive reconstruction is allowed at owner startup/recovery and explicit history/view operations, not on ordinary state or metadata access.
5. A cold archive with no valid index may require two streaming passes. An arbitrary rewind is not solved by retaining full snapshots for every turn.
6. Memory consumption is bounded by admitted active contexts, bounded UI views, and bounded work buffers—not total event count or completed-child count.
7. Canonical event acceptance precedes execution authorization. Persisted results precede acknowledgement of execution requests.
8. A delivery retry is not an execution retry. Transport delivery count never creates a new operation identity.
9. Uncertain external side effects are surfaced for explicit repair, not automatically repeated.
10. Durable pending work has no access-based expiry. Disk quota exhaustion applies backpressure; it does not evict unconsumed work.
11. The history index is disposable. Pending-transition intents, execution authorizations, and unconsumed results are not disposable caches.
12. No uncapped queues, event arrays, progress histories, completed-child caches, or ever-growing in-memory deduplication sets are introduced.

### Non-goals

Do not replace Messenger, introduce another broker/daemon, redesign tool business logic, implement a user-visible branch tree, or add full-state-per-turn checkpoints. Preserve existing history, human-input, and explicit `/repair` semantics unless a change is explicitly identified here. Do not change worker memory limits to make acceptance tests pass.

The dispatch/recovery protocol in sections 5–6 is a deliberate durability hardening change. Implement and review it separately from the reader/rendering optimizations; it is not a free consequence of switching to local state.

## 2. Ownership and service boundaries

| Owner | Retains | Must not retain |
|---|---|---|
| `run_control` | Parent current state; admitted nonterminal child states; narrow current identities and cursors | Historical state versions, completed children, full transcript after bootstrap |
| LLM/tool/agent worker | Its current immutable request and bounded result working set | A general run-state cache or unrelated runs |
| Controller | Process supervision, session ownership, bounded forwarding buffers, high-water notifications | Full execution context or a complete transcript copy |
| TUI | Bounded parent transcript, at most one child view, bounded visible rendering cache | Execution state, all child transcripts, every progress revision |
| History index | Disk-backed scalar locations and logical history metadata | Message bodies, decoded events, serialized `RunState` |
| Durable work storage | Unfinished transition intent; request/result references and operation status | A hot read-through history cache |

Existing immutable LLM invocation envelopes are the model for other execution inputs. [S2]

### Refactor existing responsibilities

- Restore owner-local `ActiveRunContext`. Expose `createNew`, `loadRecovered`, `requireLoaded`, `replaceCurrent`, and `release`. Creation requires a successful durable run-creation/reservation operation, not an empty registry slot.
- Keep `RunMessageProcessor`, existing handlers, and reducers. Move irreversible coordination mutations out of pre-commit handler code into the recoverable commit/finalization protocol.
- Restrict `CommittedRunEventAppender` to owner-side use or replace execution-side uses with command submission. This includes progress, failure, history, compaction, and extension-originated canonical writes.
- Prepare fork/subagent invocation context in the owner. Execute expensive model work in an execution worker, without holding an owner transition lock.
- Keep narrow cancellation, question, operation, batch, and child-status lookups in their appropriate stores. Do not expose a `stateFor()` service over RPC.
- Extend existing tool-batch and deferred-batch storage rather than adding parallel authorities for the same operation. [S3]

Service names in this document denote responsibilities, not a requirement to create an interface and implementation for every name.

## 3. Data and lifetime contracts

### State registry

The parent remains resident while its session is attached, including long idle/tool waits. It is loaded once per owner lifetime unless an explicit recovery or history selection replaces it.

Children remain resident only while needed for admitted, nonterminal execution or terminal finalization. All nonterminal children—including waiting children—count toward admission limits. Admission is checked before constructing or queueing another full child context. Excess launches are deferred using bounded durable descriptors, not resident prepared context graphs.

Use a bounded registry, not an eviction cache. Never silently evict an executing run and assume the next getter will reconstruct it. When admission cannot fit another context, defer/reject new work explicitly.

### Persistent data classes

| Data | Location/lifetime |
|---|---|
| Canonical events | Existing parent/child `events.jsonl`, retained as today |
| History index | Run-local `history-index.sqlite`, rebuildable |
| Pending transition | Run-local `runtime/pending-transition/`, at most one published intent per run |
| Request/result payloads | Private durable runtime payload storage, retained until their references are no longer needed |
| Operation authorization/status | Narrow durable rows; extend existing operation/batch stores or add one table for uncovered effect types |
| Bootstrap transfer | Private bounded temporary spool, deleted after transfer acknowledgement or abandoned-bootstrap cleanup |

Use existing parent/child path resolution. Never put child runtime files under a fabricated top-level child session directory.

Execution payload storage is **not** the existing ephemeral output-cap directory. That directory has startup/shutdown cleanup and is not a safe home for unconsumed execution requests or results. [S1]

Large full output may remain an optional artifact under its existing policy. The bounded model-visible result and any bytes necessary to replay canonical state must survive independently of that optional artifact.

## 4. Parent startup: one reconstruction, one handoff

### Protocol

Use distinct `runtime.ready` and `session.ready` meanings. The first announces a usable controller transport. The second announces that the owner has recovered the session and the TUI has mounted a coherent bootstrap. Starting a process is not evidence that its recovery has completed. The reviewed controller currently emits runtime readiness after consumer launch. [S4]

1. The controller acquires exclusive session ownership. It starts the owner worker in `Bootstrapping`, and establishes bounded command/response routing before waiting for worker output.
2. Only the owner performs execution recovery. Controller session-start listeners and `SessionInitializer` must not independently reconstruct and publish `RunState`.
3. The owner reconciles unfinished transition intents, validates the archive/index cut, and constructs parent state plus bounded transcript/resume products through the same replay coordinator.
4. Recover child execution state only for children proven nonterminal by durable batch/operation records. Do not hydrate the completed-child catalog.
5. Apply existing attach policy—such as cancelling recovered human-input waits and refreshing generated instructions—through owner transitions before freezing the bootstrap cut. Feed those newly committed events into the temporary bootstrap projector without a second archive scan. Attach alone does not start a new model turn. [S1]
6. Freeze a cut `(run_id, bootstrap_id, view_epoch, canonical_seq, end_offset, selected_anchor)`. `end_offset` is a complete committed-event boundary, not current file size or `sequence.cursor`.
7. Stream the bounded transcript/resume products into a private spool. Seal it with counts, bytes, schema version, and checksum. The spool contains no `RunState`. Immediately reset the temporary projector and release reconstruction products not owned by the registry.
8. Send `bootstrap.available` containing the spool token and cut. The controller transfers bounded frames; it does not deserialize and retain the whole spool. Tokens resolve only inside the session's private spool directory—never accept arbitrary file paths from the UI.
9. The TUI incrementally validates and builds its bounded view. `bootstrap.end` supplies the final checksum/count. It atomically mounts the new view, sets its durable cursor to the cut, and sends `bootstrap.applied` with the bootstrap ID and cursor.
10. Only a matching acknowledgement releases the spool. Superseded bootstrap IDs and old view epochs cannot update the screen or its cursor.
11. Catch up committed events after `end_offset`, then enable normal session interaction. Canonical sequence gaps are valid. Advance the cursor to the last event actually applied, never by incrementing it by one.

### Events arriving during transfer

Do not buffer an unbounded suffix while the TUI mounts. Retain only the latest committed high-water marker; catch up from the archive's unread suffix in bounded batches. Lost notifications are harmless because an acknowledged cursor and current high-water marker remain available.

Transient sequence-zero deltas may be dropped/coalesced during bootstrap; they are not used to establish the durable cursor. A completed message always comes from canonical history. Normal active forwarding can send committed events directly; bounded suffix reads are for bootstrap catch-up/resynchronization, not repeated full replay.

Transfers have a bounded timeout and byte quota. Slow or disconnected clients cannot hold the owner lock or prevent durable result processing. Cancelling a transfer closes handles and releases buffers. Failure before mounting leaves the UI visibly not attached; never substitute an empty usable session for reconstruction failure.

## 5. Canonical append and dispatch: explicit crash protocol

A JSONL append and an unrelated SQLite/transport transaction are not one atomic transaction. SQLite atomicity does not extend to an arbitrary application-managed log file. [S6]

### Narrow pending-transition intent

Introduce one recoverable pending intent per run, **not a full-state snapshot or permanent receipt log**. It records:

- Transition/source-message identity and expected predecessor identities.
- Archive identity, predecessor committed offset/sequence, and prefix verification data.
- Exact already-serialized planned event bytes in a bounded staging file, including allocated sequences.
- Replayable effect descriptors with stable effect IDs and immutable request references.
- Idempotent coordination-finalization actions, including mailbox consumption and tool-batch changes where relevant.

Allocate sequences under owner serialization before publishing the intent. Allocation can leave holes if the process stops before an intent becomes durable; holes are not committed events.

All nondeterministic values used in the transition—IDs, timestamps, prepared requests—are captured before publishing the intent. Recovery must not rerun model/tool code or generate different events. Existing post-commit callbacks that dispatch essential work must become persisted, replayable descriptors; arbitrary closures are not a recovery format.

### Commit algorithm

1. Require a loaded, non-recovering state. Validate the source command and operation identity. Check memory/disk admission and request/result capacity.
2. Compute the transition without external execution. Stage exact event bytes, request payloads, and effect descriptors. Validate maximum record and transition sizes before touching the canonical archive.
3. Atomically publish the pending manifest after its referenced files are complete. Effects remain unauthorized. Do not acknowledge the source message.
4. Verify the expected predecessor offset. Append the staged bytes using checked writes, handling short writes. Flush/check the promised file durability boundary. Only one owner may append; no later transition starts while this intent is unfinished.
5. Idempotently finalize coordination metadata and arm its pending effects. Do not authorize effects unless the complete planned event suffix has been verified as appended. Update local state from the same committed event batch; never reread the full archive to perform this update.
6. Update history-index metadata incrementally. If index publication fails, mark the index unusable. Execution may proceed only where correctness does not depend on that index; commands requiring a historical/idempotency lookup wait for index repair.
7. Publish the committed cut for observers. Finish/remove the pending intent only after required coordination finalization is durable. A removed intent must not be needed to rediscover pending effects: armed operation rows retain that responsibility.
8. Acknowledge the source control delivery. Dispatch pending effects outside the transition lock. A broker outage leaves durable pending effects, not a best-effort warning that loses work.

Do not let `StreamingCommittedRuntimeEventStore` or another append wrapper emit supposedly committed runtime events as each raw line is appended. Publish the finalized batch/cut only after the transition protocol permits observation. Readers must honor that cut even when the physical file already contains additional staged bytes.

The source message's accepted identity must also have durable committed evidence. Reuse current operation/mailbox guards; use a disk-indexed canonical command identity when current bounded state is insufficient. Never add a forever-growing in-memory key set.

### Recovery of an unfinished append

Before normal replay, compare the actual tail at the intent's starting offset with the staged expected bytes, in bounded chunks.

- No appended bytes: append the planned batch.
- A byte-for-byte matching prefix, including a partial final JSON line: append the missing bytes from the staged suffix; do not allocate new sequences.
- The whole batch is present: do not append it again; repeat only idempotent coordination finalization.
- Different bytes, an impossible offset, or unexpected extra tail: stop and surface corruption/concurrent-writer evidence. Do not silently skip, truncate, or invent events.

This permits repair of a partial write only because its exact intended bytes are available. An unexplained malformed legacy tail remains an explicit repair case. Normal readers never read beyond an advertised committed cut.

### Durability boundary

The mandatory first release is tested against process death, worker death, interrupted writes, duplicate delivery, and database/transport failures. Do not advertise power-loss guarantees merely because files were renamed. File synchronization, directory-entry durability, database configuration, and storage behavior require separate platform validation. PHP exposes `fsync()`; its result must be checked when that durability mode is promised. [S7]

## 6. Effect execution, result persistence, and ambiguity

Messenger can redeliver a successfully processed message when the worker dies before acknowledgement. Stable delivery identities alone do not make an arbitrary external action exactly once. [S5]

### Operation authorization

Use stable logical identity including the existing run, turn, step, attempt, idempotency key, and tool/batch identity where applicable. Add an owner-assigned execution generation when needed to distinguish superseded work; do not use a UI view epoch or transport retry count for this purpose.

Persist a narrow state machine:

`Prepared -> Armed -> Running -> ResultReady -> Consumed`

`Running -> OutcomeUnknown` is an explicit recovery branch. Cancellation may invalidate authorization or cause a later result to be recorded as stale; it is not proof an external action did not occur.

Only the owner creates/arms an authorization record. An execution delivery with no matching record is rejected as stale; it must not recreate the record.

Before invoking the external action, the worker performs an atomic `Armed -> Running` claim against the expected operation and claim token. A duplicate delivery cannot claim again. A worker lease/heartbeat helps observation but expiry alone does not prove that execution stopped or is safe to repeat.

### Dispatch rules

A dispatcher republishes unclaimed `Armed` work using the same effect ID when delivery is uncertain. The gate makes duplicate envelopes safe. A `Running` operation is not redispatched for execution merely because a dispatch acknowledgement or worker heartbeat is missing.

For a provider/tool with its own verified idempotency facility, pass the same logical idempotency key. Do not assume an internal Hatfield key is enforced by the external system.

### Result rules

1. Normalize/cap the model-visible result and persist it atomically under deterministic `(effect_id, claim_token)` identity.
2. Publish a `ResultReady` reference with size/hash and ownership checks, then notify the owner. A scan of pending result-ready operations can repeat notifications after a crash.
3. Acknowledge the execution request only after its result is durable. Missing/broken notification delivery does not erase the result.
4. The owner applies the result through the normal pending-transition commit protocol. Mark `Consumed` only after the corresponding canonical transition and coordination finalization commit.
5. Repeated result delivery is a no-op after committed evidence matches. Before parent/child generation validation, do not publish new progress or terminal state.
6. If cancellation/rewind superseded the operation, persist its stale-result disposition and retain evidence as required; do not apply it to current context. Never re-execute it to obtain a different result.

If a worker dies after saving its deterministic result file but before updating `ResultReady`, recovery validates and adopts that file using the stored claim token. If no valid result exists after a `Running` worker is confirmed dead, enter `OutcomeUnknown`. A crash immediately before the external call can also be indistinguishable from one immediately afterward; conservative repair is intentional.

**Guarantee:** no automatic repeat of an ambiguously executed arbitrary tool; no loss of an already-durable result because a notification/acknowledgement was lost. No general guarantee of recovering a result that never reached durable storage. Unknown work can block automatic progress and must be visible to the user.

Explicit `/repair` may authorize another attempt after confirming the old execution cannot still run and explaining potential duplicate side effects. This is compatible with the documented warning that ordinary tools are not exactly once, but automatic execution retry paths must be audited to respect the new gate. [S1]

### Crash matrix

| Crash boundary | Required recovery |
|---|---|
| Before pending intent | Source delivery remains unacknowledged; validate/reprocess normally |
| After intent, before/during append | Complete the exact staged bytes; no effects authorized yet |
| After append, before arming | Verify suffix, finalize coordination, arm once |
| After broker acceptance, before dispatch acknowledgement | Republish same identity; execution gate prevents a second claim |
| After claim, before durable result | Keep live owner if present; otherwise explicit unknown unless result file can be adopted |
| After result persistence, before owner notification | Repeat result notification; never execute again |
| After canonical result commit, before acknowledgement | Detect committed identity; no duplicate messages or follow-up effects |
| Cache/index disappears | Rebuild disposable data; do not initialize a new run or erase durable work |

### Garbage collection

Delete request/result payloads and authorization records only after consumed/stale disposition is durable and old deliveries cannot regain authorization. Missing records reject old deliveries. Only newly validated owner transitions create records. Retain any minimal canonical command-identity index needed to stop an old source command from generating a new effect.

Batch-level result files may need to remain until the batch's canonical tool-message commit; receipt of one member is not enough. Sweep disk records in bounded pages. An unresolved operation may remain on disk indefinitely; it must not force an unbounded resident graph. Quota exhaustion refuses additional work rather than deleting this evidence.

## 7. Completed children and live-view entry

### Release boundary

At child termination, the owner prepares a bounded terminal handoff containing child/batch/operation identities, final canonical cut, terminal status, summary/result reference, and any required capped failure/cancellation context.

Commit that handoff and its pending parent-delivery intent with the child's terminal transition. Once these are durable, immediately remove the child from the in-memory registry and release its tool/message maps, listeners, promises, captured callbacks, and temporary projectors.

**Do not wait for the parent to consume the handoff before freeing the child's state.** Parent delivery retries use the durable handoff, not the child's graph. Distinguish durable handoff availability, notification enqueued, and parent consumption; a single `terminalCompletionEnqueuedAt` timestamp is not evidence of all three. [S8]

While durable finalization fails, the run is finalizing/recovery-required, not a completed child admitted into a permanent cache. Persist enough bounded terminal evidence to permit unloading as soon as possible. A failed finalization must backpressure new child admission.

On restart, enumerate nonterminal execution records and outstanding terminal handoff references in pages. Outstanding handoffs are delivered without rehydrating the corresponding completed children.

### Entering a child view

1. Obtain a descriptor `(child_id, view_epoch, canonical_seq, end_offset)` from the controller/owner. For completed children, use durable terminal metadata; no execution-state load is needed.
2. `ChildRunTranscriptSnapshotProvider` builds a transcript-only projection from that stable archive cut, using the index when available. No `RunStateReducer`, context refresh, execution recovery, or shared child-transcript cache.
3. Consume records incrementally with the UI's view-build work/byte budget. Cancellation checks occur between bounded records/batches. The builder may be an existing read-only view task; it must not monopolize the canonical transition owner.
4. Mount at the supplied cut; catch up the bounded unread suffix for an active child. Discards/repair invalidate the old view epoch and trigger explicit resynchronization.
5. Keep only the bounded parent view plus one bounded visible child view. Entering another child or leaving the view closes readers and releases blocks, widgets, ASTs, listeners, and pending callbacks. No hidden LRU of completed-child transcripts.

Re-entering a completed child may reread its archive. This is an explicit, user-selected history read, not a violation of the ordinary execution-read budget.

## 8. History index and replay semantics

### Index schema (logical)

A separate run-local SQLite index avoids putting its growing historical tables on the hot application-cache path. It contains scalar rows only:

- `index_meta`: schema version, archive identity, indexed complete byte offset, last actual indexed sequence, selected anchor, retained tip.
- `event_location`: actual sequence, start offset, byte length, event type, context-anchor identity, and indexed source/operation identity where needed.
- `turn_anchor`: immutable anchor ID, displayed turn number, predecessor, creation sequence, retained status, bounded prompt preview/reference.
- `history_change`: sequence, selection/discard kind, resolved anchor identity.
- `context_checkpoint_location`: compaction event locations and the history identity to which they apply; no full state bodies.

Use immutable `TurnAdvanced` event identity/sequence as the anchor key. A displayed turn number must not be assumed globally unique after rewind. Keep the UI's linear-history model; these identities are implementation bookkeeping, not a new branch-tree feature.

### Incremental update

After canonical append, apply the committed batch and its new watermark in one index transaction. On index failure, roll back and mark it unavailable. A later explicit catch-up reads only the bytes after its last valid committed watermark.

Selection moves the selected anchor; it does not discard later retained anchors. Discard removes logical retained descendants of the selected anchor according to the existing history policy; it does not delete archive bytes. New anchors are linked to the surviving predecessor. Previously discarded anchors stay addressable as physical history but are never included in the active context merely because a displayed number is reused.

Command events that seed a future turn are assigned logical anchors by the same policy as retained-history replay. Store unresolved event locations on disk until resolved; do not retain their decoded bodies in an ever-growing list. Queued, applied, rejected, suppressed, and discarded commands are not interchangeable. Preserve the existing filter's tested semantics.

### Incomplete index, sequence holes, and corruption

- Advance the watermark only through complete committed records. Appending the log and updating its index are separate steps; an index behind the log is expected and recoverable.
- Validate strict sequence increase, not contiguity. A gap does not mean a missing event.
- Use byte offsets and the last actual event sequence for catch-up. `sequence.cursor` is an allocation counter, not a committed-data watermark. [S1]
- Reject an index with the wrong version/archive identity, offsets beyond the committed cut, or a mismatching boundary record. Rebuild explicitly.
- Incomplete writes are handled by the pending-transition protocol before indexing. A line with no validated complete commit boundary is never decoded as committed history.
- In-place edits/replacement while owned are unsupported. Record identity and boundary checks detect ordinary replacement/truncation; they are not advertised as full cryptographic detection of arbitrary interior edits. An offline full verification can scan the archive when that guarantee is required.

### Cold reconstruction

With no valid index, pass A streams the archive to build logical history/index metadata. It retains bounded parser state and disk-backed unresolved references—not decoded history, all-turn checkpoints, or all seen-sequence IDs.

Pass B iterates the resolved active records through bounded readers to build current state and, when requested, a bounded transcript. Release each raw event after its reducers have consumed it. Read full payloads only for products that need them. This may reread archive bytes; count and report it honestly as a two-pass cold rebuild.

With a valid index, resolve the needed ranges first and perform one coordinated replay for the requested products. Per-record seeks or multiple relevant ranges do not justify pretending the whole archive was read only once; record physical bytes and access reason.

A `ContextCompacted` event can replace prompt messages, but is not automatically a full execution checkpoint. Operational identity, pending tools, mailbox coordination, and execution receipts are reconstructed from their proper canonical/durable sources. Never skip pre-compaction execution metadata merely because prompt context can start later.

Historical conversation reconstruction and current execution authorization are distinct products. A history selection must not revive an old pending tool or LLM operation merely because replay at that anchor contains it. Apply the existing selection/cancellation preconditions under owner serialization; fence superseded operations with current durable identities, then publish the selected conversation and history-position event. Old results cannot attach to a newly selected branch. Selection itself executes no tools and preserves forward retained history until the normal context-mutating discard.

Avoid a new ever-growing `retainedTurnNos` array in the hot registry. Query historical anchor pages on demand; retain only current selected/tip identities and bounded necessary working metadata.

## 9. Byte bounds, admission, and backpressure

These starting values must be measured against the packaged application and adjusted through explicit configuration review—not hidden clamps or higher PHP limits.

| Budget | Initial value | At limit |
|---|---:|---|
| Encoded current context, per run | 6 MiB soft / 8 MiB hard | Safe compaction before dispatch; otherwise reject/defer oversized input |
| Encoded current contexts, owner aggregate | 16 MiB | Defer new child/context admission |
| Model-visible tool result | 256 KiB | Produce explicit bounded excerpt/overflow notice; retain full output only according to artifact policy |
| Transcript payload, each parent/visible-child view | 4 MiB and 2,000 blocks | Evict oldest complete display groups; provide explicit historical navigation |
| Render cache, across visible views | 2 MiB plus a node/widget cap | Release offscreen rendering products |
| IPC frame, encoded | 64 KiB | Chunk before framing; reject malformed oversized input |
| Pending IPC bytes per peer | 1 MiB | Coalesce transient deltas or switch durable delivery to cursor-based resync |
| Archive read batch | 256 KiB and 128 records | Yield between batches; an admitted larger single record is processed alone |
| Single canonical JSONL record | 16 MiB | Reject before append; oversized legacy record produces a typed compatibility error |
| Archive-decoded records awaiting reduction | 1 | Do not collect the complete input batch while reducing |
| One staged transition's event bytes | 32 MiB on disk | Reject invalid transition size before canonical append |
| Runtime payload spool, per session | 256 MiB | Reserve capacity before execution; stop admitting work rather than evicting unfinished data |
| Bootstrap spool | 8 MiB | Keep product within transcript/resume budgets; never truncate required execution state |

All count limits are additional to byte limits. Count messages, tool calls, nesting depth, and decoded nodes to bound pathological JSON/object overhead. Include content, tool arguments/results, reasoning/encrypted content, metadata, and encoded attachment references in context accounting. Oversized inlined attachments must be rejected or represented using the existing supported reference mechanism.

Do not repeatedly serialize a whole run just to measure it. Maintain sizes on immutable messages/content nodes as they are constructed; accumulate the total when messages are added/replaced. Validate actual wire size during encoding. References have both an envelope-byte cost and a resolved-payload budget—the latter cannot be bypassed by making the envelope tiny.

A byte quota is not a PHP heap guarantee. Parsing, object graphs, copy-on-write separation, serializer buffers, callbacks, and allocator retention all matter. Admission must reserve headroom for the largest accepted event and old/new contexts during compaction. Validate the heap-amplification envelope with real structures and structural caps. If the configured combination cannot meet the existing hard process limit, reject it or defer work; do not continue on optimistic byte arithmetic.

### Oversize behavior

Do not silently trim user/system messages or sever assistant-tool/result groups. Compaction is attempted at the existing safe boundary. If no safe boundary exists or mandatory retained content alone exceeds the hard budget, return `ContextTooLarge` with a clear recovery action before dispatch.

For tool output, stream capture through the output limiter; never collect unlimited output and truncate afterward. Preserve structured success/error status and artifact/truncation notices. Reserve enough disk space for the bounded terminal result before external execution. Unexpected storage failure afterward is reported as result-persistence/unknown work, not retried external execution.

### Transport backpressure

Use length-limited framing before JSON decoding, including on consumer stdout and controller stdin. An incomplete line buffer has the same hard limit. Do not allow unlimited concatenation while waiting for a newline.

Transient output is coalesced per active operation and may be dropped; durable results are stored, not dropped. When a durable observer falls behind, store only its acknowledged cursor and latest high-water marker, then resynchronize. A disconnected TUI does not prevent result persistence.

Disk queues and unresolved operations also need quotas and admission checks. A bounded process with an unbounded spool is not a complete resource policy.

## 10. Compaction and actual object release

Compaction creates an immutable successor context, commits its canonical transition, and swaps the registry entry. After the commit frame returns, old messages must not remain reachable from checkpoints, hooks, log contexts, pending callbacks, observers, tool-batch snapshots, completed-results maps, serializers, or UI projectors.

The temporary old-plus-new peak is expected and included in admission; indefinite historical retention is not. A live LLM/fork operation may legitimately own an immutable earlier request until it finishes. That ownership must be explicit, bounded by the worker/request budget, and independent of an owner-local history cache.

After terminal execution/handoff, release both the owner reference and all worker job references. After each worker task, clear/reset non-owned entity-manager results and service accumulators using the existing worker reset lifecycle. Prefer paged scalar/DTO queries for catalogs and recovery enumeration to avoid retaining thousands of hydrated entities.

Apply the existing transcript compaction retention policy, additionally bounded by bytes/count. Release corresponding TUI widgets and Markdown ASTs. Shared parser/highlighter infrastructure may remain resident, but not parsed documents for evicted blocks.

Memory telemetry records only scalar counts/bytes/cursors; no prompt/output payloads. Measure quiescent memory after cleanup as well as the transient peak. `gc_collect_cycles()` is not a substitute for removing strong references, and allocator-reserved memory need not immediately return to its pre-task value.

## 11. Failure and compatibility rules

- Recovery-required is an explicit operational state. Do not consume ordinary mutating deliveries for that run until reconciliation succeeds; preserve their durable inputs.
- An ordinary projection/index failure must not make a failure-handler append through a different, unguarded path. Failure transitions use the same canonical commit machinery.
- Do not reset a known run to queued/empty on any cache/index miss.
- Do not use process IDs alone as durable ownership identity. Pair claim identity with a process-instance token and actual supervised ownership evidence.
- No live execution claim is stolen on a timeout. Unknown external execution stays explicit.
- Existing archives remain readable through the bounded legacy path when records meet configured limits. For older runs without prepared-transition evidence, an ambiguous tail is an explicit repair case—not something the new journal can retroactively prove safe.
- Disposable PR #541 cache entries can be ignored/removed under quiesced ownership. Do not delete operational queues, unfinished batch snapshots, pending results, or canonical events during migration.
- Keep the existing attach/human-input policy and test it explicitly; a cache refactor is not permission to silently revive old human questions. [S1]

## 12. Suggested implementation sequence

### Change 1 — independent memory wins

Extract streaming JSONL reads, remove decoded archive caching, retain shared Markdown parser/highlighter construction, and add honest physical-read/retained-memory counters. No message-delivery semantics changes.

### Change 2 — owner-local state and immutable execution inputs

Restore a bounded explicit registry. Inventory every cross-process full-state reader and canonical writer. Convert fork/subagent inputs and side-writer commands. Remove shared full-state/history/transcript caches only after consumers have correct replacement paths.

### Change 3 — recovery correctness

Implement the pending-transition append/finalization protocol, execution authorization gate, durable result-ready delivery, and explicit unknown-execution status. Extend existing stores where their identities already fit. Do not advertise automatic append/dispatch recovery before this gate passes failure-injection tests.

### Change 4 — index and coordinated parent bootstrap

Implement the scalar history index, two-pass cold fallback, product-specific replay, bootstrap spool/acknowledgement, and bounded suffix catch-up. Remove duplicate TUI/controller/worker reconstruction entry points.

### Change 5 — child and compaction lifetimes

Implement durable handoff-before-release, transcript-only child views, disposal on exit, admission budgets, and release audits. Enforce budgets on every ingress, not only archive replay.

Each change is independently reviewable. Maintain a call-site inventory and delete obsolete adapters/tests; do not keep both architectures through layers of fallback getters.

## 13. Required acceptance tests

Use real JSONL files, real serialization, the real configured SQLite adapter where appropriate, and separate processes for ownership/crash tests. Stub-only tests cannot establish process-boundary or recovery guarantees.

| Scenario | Required assertion |
|---|---|
| Parent startup with valid index | One coordinated selected-record replay; TUI/controller do not independently reconstruct execution state |
| Missing/corrupt index | Explicit bounded two-pass fallback; no all-turn snapshots; indexed results match reference history semantics |
| Long parent/tool/human wait | No access expiry; subsequent valid delivery can proceed after normal prerequisite checks |
| Completed children accumulated over hundreds/thousands of runs | Registry contains no completed children; quiescent memory does not grow with completed-child count |
| Pending parent terminal delivery | Child state is already released; delivery still succeeds from durable handoff after restart |
| Child view enter/exit/re-enter | No execution recovery or invocation; bounded transcript; all view-owned references released on exit |
| Repeated compactions across many distinct turns | Old graph weak-reference probes become unreachable after the legitimate transition/job lifetime; memory plateaus |
| Huge single record/output/metadata tree | Limits apply before unbounded decode/capture; typed error or explicit output-cap behavior |
| Slow/disconnected TUI | Pending bytes stay below cap; durable suffix is eventually applied once; no terminal event lost |
| Crash at each append/finalization step | Exact staged events completed once, unchanged allocated sequences, no early effects |
| Duplicate execution message | Only one worker claims the authorization; a stale/missing record does not recreate permission |
| Crash after possible side effect before result persistence | Explicit unknown state; no automatic external repeat |
| Result persisted, notification lost | Owner eventually consumes the original result; tool is not rerun |
| Result committed, ACK lost | Redelivery is a no-op; no repeated follow-up effect |
| Rewind/discard with delayed old result | Old result does not mutate selected context; disposition is durable |
| Reused displayed turn number | Distinct immutable anchors; discarded content stays excluded |
| Sequence holes and partial suffix | Actual committed offsets/sequences determine replay; no fictitious missing-event failure |
| Cache/index cleared during known run | Never publishes empty queued state over existing canonical history |
| Disk quota/storage failure | Backpressure before execution where possible; no deletion of unfinished data, no false completion |
| Resume with pending human input | Existing cancel/refresh behavior preserved; attach does not silently execute a new turn |

For history correctness, compare selected messages, pending operations, tool-call/result relationships, transcript, and selected position against established fixtures and a simple slow test-only reference implementation. Generate adversarial sequences of selection, discard, compaction, queued/applied commands, and late results. Avoid using the new implementation itself as the sole oracle.

For memory tests, compare equal active-window workloads with increasing archived event counts, distinct turns, completed children, and progress revisions. Include many turns—not only repeated events under a single turn. Define baseline-to-quiescent tolerances from repeated packaged-process runs; do not assert a universal byte-perfect allocator value.

Track `archive_bytes_read`, `full_scan_count`, read reason, index catch-up bytes, decoded-record peak, active-run count, current-context bytes, temporary compaction bytes, view bytes/blocks, IPC pending bytes, pending payload disk bytes, and result-ready/unknown counts. Ordinary getters must have zero archive reads, zero full graph serialization, and zero cache writes.

## 14. Done criteria

The replacement is done when the acceptance matrix passes at the existing process limits, completed children and pre-compaction graphs demonstrably disappear, ordinary execution uses no full-state shared cache, and recovery does not silently rerun uncertain tools or discard durable results.

Document the intentional exceptions to the archive-read goal: cold index rebuild, explicit history selection, child-view entry, and bounded observer suffix resynchronization. Do not call a method-count assertion proof of one physical archive traversal.

## Sources and baseline references

Source facts are distinct from the proposed contracts above.

- **S1:** `docs/session-storage.md` at the baseline commit. Canonical archive/sequence distinction, child and attach policy, DB-only durable records, repair safety, and ephemeral output-cap lifecycle.
- **S2:** `src/AgentCore/Domain/Message/ExecuteLlmStep.php` at the baseline commit. Existing immutable invocation boundary for prompt history.
- **S3:** `src/CodingAgent/Session/SessionToolBatchStore.php` at the baseline commit. Existing durable run-scoped tool-batch snapshots and lock ordering.
- **S4:** `src/CodingAgent/Runtime/Controller/HeadlessController.php` at the baseline commit. Controller ownership, consumer launch, readiness, and forwarding responsibilities.
- **S5:** Symfony Messenger documentation, “Writing Idempotent Handlers,” retrieved 2026-10-01: `https://symfony.com/doc/current/messenger.html`.
- **S6:** SQLite, “Atomic Commit In SQLite,” retrieved 2026-10-01: `https://www.sqlite.org/atomiccommit.html`. SQLite transactions are a database guarantee, not a transaction over arbitrary application-managed external files.
- **S7:** PHP manual, `fsync`, retrieved 2026-10-01: `https://www.php.net/manual/en/function.fsync.php`.
- **S8:** `src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Lifecycle/DeferredSubagentBatchLifecycleDeliveryService.php` and deferred-batch entity at the baseline commit. Existing progress/terminal delivery orchestration and enqueue marker; proposed consumption protocol must not conflate these stages.
