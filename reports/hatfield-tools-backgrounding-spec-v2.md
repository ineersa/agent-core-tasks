# Hatfield: user-controlled tool backgrounding

**Technical design and implementation specification**  
**Date:** 2026-10-15  
**Source baseline:** `ineersa/agent-core@f1240029bc0f249fd065edcd1e2ef2a408a35c3b`  
**Revision:** 2 — split background execution from the model-visible inbox and hierarchical messaging.  
**Status:** Proposed design; no implementation or test execution is claimed.  
**Companion:** [Parent–child messaging and result inbox](hatfield-parent-child-messaging-spec.md).  
**Delivery order:** BG-01–04 ship Bash independently; BG-05 adds child tools. MSG-01 can follow BG-01–04, with child handoffs added after BG-05. Code mode and MCP are independent later integrations.  
**Revision basis:** The attached specification and the subsequent scope decisions in this conversation; source baseline unchanged, not revalidated against a newer repository head.

## 1. Decision summary

Implement backgrounding as an **owner-controlled change in how a tool invocation resolves its caller's wait**, not as a new execution mechanism.

The `run_control` owner accepts the user's request and commits one background acknowledgment for the original tool call. The existing execution owner continues the already-started work. Its eventual result is accepted independently of the original tool batch, retained as an immutable completion message/result reference, then delivered through the existing command mailbox at a safe model boundary with explicit tool provenance. Consuming the mailbox entry does not delete the retained message.

Reuse these facilities:

- `RunMessageProcessor`, `RunCommit`, the prepared-transition store, `TransitionFinalizer`, and the existing application database transaction for authoritative decisions.
- `DeferredToolCompletion` for detached-call correlation, extended rather than supplemented with another job registry or delivery journal.
- `ToolBatchCollector` and its store for admission; add detached execution reservations to their accounting rather than implementing another scheduler.
- Existing managed Bash processes, deferred child lifecycles, code-mode host execution, and later MCP connection/request ownership.
- `AgentMessage`, `ModelNotificationDTO`, the command mailbox, canonical events, and existing specialized result surfaces. [S21], [S31]
- One retained-message/delivery contract (§7.5) shared with the companion specification. In this release its only producer is accepted detached completion; no model-facing inbox or agent-authored send API is required.

**First Bash implementation:** keep `BashTool`'s existing supervision loop running in its current Messenger worker after the owner acknowledges backgrounding. Delete the threshold question and do not return early from that loop on Ctrl+B. This keeps the original monotonic deadline and cancellation checks alive without transferring supervision to a new service. The cost is explicit: a backgrounded Bash invocation continues to occupy a tool worker. See §10 before changing this choice.

This is not an implementation of process-per-tool isolation, unlimited tool concurrency, daemonized work, or exactly-once side effects.

### Navigation

[Scope and ownership](#11-ownership-of-the-two-specifications) · [Product decisions](#4-product-decisions) · [Shared retained-message contract](#75-shared-retained-message-and-delivery-contract--version-1) · [Delivery and wake policy](#9-automatic-delivery-provenance-and-safe-boundaries) · [File-level changes](#15-file-level-implementation-map) · [Implementation phases](#17-implementation-sequence-and-acceptance-gates) · [Tests](#18-deterministic-verification-specification) · [Companion inbox spec](hatfield-parent-child-messaging-spec.md)

### Reading this specification

Sections marked **Observed** describe inspected source at the pinned commit. **Required** behavior comes from the supplied requirements brief or is a normative target of this proposed design. Decisions D01–D11 resolve the brief's open questions as recommended defaults; they are not claims that these defaults were previously approved or already implemented. New symbols and event names below are proposed internal names, not existing APIs.

### 1.1 Ownership of the two specifications

| Concern | Normative owner | Release boundary |
|---|---|---|
| Detach caller wait; acknowledge once; continue the same execution | This specification | BG-01–04 for Bash |
| Native ownership, deadlines, cancellation, capacity, terminal acceptance | This specification | Per BG integration |
| Retained completion identity, evidence, mailbox delivery, safe boundaries | This specification, especially §7.5 and §9 | BG-01–02; shared dependency |
| Model-visible listing/reading of retained messages and results | Companion specification | MSG-01; not required for Bash |
| Parent ↔ direct-child notices and parent task steering | Companion specification | MSG-02 |
| Correlated parent–child questions/replies | Companion specification | MSG-03; nonblocking in the initial release |
| Active-work inventory, native logs and cancellation | This specification and existing native tools | Keep `/background`, `bg_status`, `agent_retrieve` |
| Sibling messaging, broadcasts, teams, general chat channels | Neither | Explicitly out of scope |

The companion extends the **producer and retrieval surfaces**, not execution supervision. It references §7.5 rather than defining another mailbox, message body store, scheduler, terminal-result authority, or wake policy. A notification, steering message, or answer cannot by itself approve execution, cancel work, mark a job complete, or restart a child.

### 1.2 Revision 2 change summary

This revision preserves D01–D08 and D10–D11, native integration choices, retained-worker capacity behavior, the BG phase identifiers, and the original deterministic proof obligations. It changes D09 from “no universal model-facing result tool” to “not part of the backgrounding release; introduced separately by MSG-01.”

It also makes retention explicit: a completion has a stable message ID and owned evidence after mailbox consumption and compaction. Notifications push useful bounded content; a later inbox can pull additional evidence. Reading or ignoring a message never settles execution. New retention/compatibility tests are C21–C28.

The new artifact names and relationship policy are design decisions for this revision, not claims about existing repository APIs. Blocking “ask and wait” is not added to backgrounding or silently enabled by the companion.

## 2. Requirements baseline and scope

The supplied **“Backgroundable tool execution: requirements and decision brief”** is the product baseline. In particular, backgrounding must not restart work, change its arguments, imply success, bypass permissions, lose its result, or resolve its original tool call twice. It must work for main-owned and child-owned invocations without confusing the visible TUI run with the execution owner. Completion must be automatic in a healthy runtime, not dependent on repeated model calls to status tools. [R0]

### Required delivery scope

| Release | Eligible invocations | Result surface |
|---|---|---|
| Bash release | Top-level model-emitted `bash`, including Bash in a child run | Managed-process logs, `bg_status`, common TUI inventory |
| Child integration | `subagent`, `fork`, and `agent_resume`, once their native execution is registered | Existing child artifacts and handoff presentation; `agent_retrieve` |
| Code-mode integration | The whole top-level `code_mode` invocation | Structured value and diagnostics, with a recoverable result artifact |
| Later MCP integration | Explicitly integrated MCP invocations whose owning connection/request stays alive | Structured result and resource references, with bounded presentation |

“Top-level” means a call emitted by that run's model and executed through its normal tool-call lifecycle. A Bash call emitted by a child model is top-level in that child run. A `tool('bash', …)` call inside code mode is not.

The conversation adds a separate model-visible inbox and parent–child communication scope, delivered in pieces. That scope is governed by the companion, not a prerequisite for this table. Ordinary foreground tool calls still return ordinary tool results; do not route every tool response through an inbox. [R1]

No first-release background flag or launch tool is exposed to the model. `read`, `write`, `edit`, `view_image`, `agent_retrieve`, extension agent jobs, and independently selected nested code-mode calls remain outside backgrounding eligibility. Their ordinary execution modes are not changed.

Interactive shell commands entered directly through Hatfield's shell-command path are also outside the first release. That path has separate `pendingShellToolCalls` handling; do not accidentally match it merely because a shell process exists. [S04]

## 3. Observed source architecture

### 3.1 Bash already has native process ownership

`BashTool` immediately calls `BackgroundProcessManager::start()`, then supervises the returned immutable database record ID. Its foreground loop checks cancellation, its monotonic timeout, and managed-process status. After a configurable threshold, it can ask a question, mark that existing process backgrounded, and return a PID/log notice. Child runs currently skip the optional question. The first release replaces that question; it does not need another process launch. [S01]

At this commit, `BashToolConfig` supplies a 15-second question threshold, a 100,000-microsecond loop interval, a 30-second default timeout, and a 3,600-second maximum timeout. These are baseline values, not new proposed settings. [S02]

The current accepted-background return also exits Bash's timeout loop. Therefore, preserving the original execution budget after Ctrl+B is a real behavior change; it cannot be achieved merely by replacing the question with a hotkey and keeping the early return. [S01]

### 3.2 Execution and caller collection are different layers

`ExecuteToolCallWorker` builds the domain `ToolCall`, executes it, and returns `ToolCallResult` through the command bus. It already handles a typed `DeferredToolCompletionOutcome`: the worker registers the original correlation and does not send a normal final result until the deferred owner completes. [S03]

`ToolCallResultHandler` validates the current turn/step and uses `ToolBatchCollector` to resolve the batch. The collector currently removes the call from `inFlight` when collecting its result. A second, different result for the same call is a conflicting duplicate; a later result from an old step is generally stale. An acknowledgment cannot simply be inserted into that path followed by an ordinary late final result. [S04], [S05]

`ToolExecutor` owns result reuse, normalization, terminal result processors, and cancellation arbitration. Its result store is not a distributed exclusion mechanism. Backgrounding must not turn an acknowledgment into a cached successful execution outcome or bypass terminal processing. [S06], [S27]

### 3.3 Existing deferred correlation is the natural integration point

`DeferredToolCompletion` already stores the original run, turn, step, tool-call identity, ordering, and execution metadata. It has unique constraints for `deferred_id` and `(run_id, tool_call_id)`. The public repository contract currently offers pending/completed status only. [S07]

`CompleteDeferredToolCallHandler` currently converts a completion to the original `ToolCallResult`, sends it, marks the correlation completed, and deletes the batch afterward. The proposed lifecycle needs a distinction between **sent**, **accepted by the owner**, and **applied to model history**. In particular, detachment must not invoke this existing completion path. [S08]

### 3.4 The current Bash notification path is unsuitable as the shared path

`BackgroundProcessCompletionPoller` refreshes process records, reads a bounded log tail, emits a sequence-zero runtime event, and sends a synthetic `append_message` through `UserCommand`. It then marks the process notified after sending. That is not proof that the run owner accepted the completion. The poller also aggregates by PID rather than a generic invocation identity. [S09]

`AdvanceRunHandler` permits non-compaction mailbox work to move a completed, failed, cancelled, or human-waiting run back to running; that branch clears pending human-input requests. The common completion path must not inherit those semantics. [S10], [S11]

### 3.5 The repository already has a prepared-transition mechanism

`RunCommit` appends a prepared transition, replaces the committed run state, and invokes `TransitionFinalizer`. The finalizer applies captured local metadata under verified transition evidence before finalizing and dispatching effects after the owner lock is released. `LocalMetadataCoordinator` already groups batch and mailbox changes in an application-database transaction. Extend this mechanism for detached correlation and reservations. Do not introduce another outbox, state file, recovery journal, or publication service. [S12], [S13]

There is an important existing limit: the finalizer explicitly does not treat the journal as a retry queue. Transport loss after finalization requires repair. This specification preserves that boundary rather than silently claiming stronger delivery durability. [S12]

### 3.6 Child tools already use durable native supervision

Subagent, fork, and resume execution share deferred child lifecycle and handoff machinery. [S33] Their deadline and interruption logic inspect deferred registration and terminal markers. Marking the generic deferred call completed when the parent stops waiting could therefore disable the very supervision backgrounding must preserve. [S14], [S15], [S16]

The pinned commit releases a finished child's full `ActiveRunContext` after its terminal transition is finalized, and also releases terminal children rehydrated for late no-op messages. Detached work must not undo that memory-lifetime boundary. [S12], [S17]

### 3.7 Code mode is a retained host execution, not just a script PID

`CodeModeHostBridge` owns the script process, IPC server/connection, nested tool execution, result collection, and cleanup. Nested tools execute synchronously on the host stack. `CodeModeTool` returns plain structured fields `code_mode_value` and `code_mode_diagnostics` and is registered sequentially. Returning from the host on Ctrl+B would enter its `finally` cleanup and destroy the execution. Polling only inside the host loop would not release the caller while a nested tool blocks. [S18], [S19]

### 3.8 MCP source is newer than parts of the architecture prose

At the pinned commit, `McpToolInvoker` passes the ambient cancellation token and timeout to the connection manager. `docs/tool-execution.md` describes the limits of that cooperation, especially blocking HTTP operations. Older architecture prose claiming there is no per-call cancellation path must not be used as the implementation baseline. The current mapper returns a string; preservation of the future structured MCP result requires explicit work before that mapping boundary. [S20], [S27], [S28]

## 4. Product decisions

| ID | Decision for this specification |
|---|---|
| **D01 — Ctrl+B target** | Operate on the visible run's eligible, unresolved foreground invocations. A selected eligible invocation wins; otherwise one candidate is selected directly, and multiple candidates open a picker. Never background all implicitly. The server revalidates the chosen immutable identity. |
| **D02 — Child batches** | Background the whole model tool invocation. A parallel `subagent`/`fork` batch receives one acknowledgment and one final aggregate handoff. Do not split it into new independently resolving tool calls. |
| **D03 — Child view** | Ctrl+B in a child live view targets that child's own invocation. It does not detach the parent's wait for the child. To detach that wait, target the parent-owned `subagent`/`fork`/`agent_resume` entry from the main view. |
| **D04 — Lifetime** | Normal completion of an agent turn does not stop accepted background work. Explicit run cancellation stops supported background work in that run's owned descendant scope. Session/controller shutdown ends the execution lifetime. Merely switching live views does not. |
| **D05 — Finished child destination** | Keep the child as the origin. If the child has already become terminal, do not restart it. Forward a bounded completion to the explicitly recorded controller/root run, identifying the original child and invocation. If that destination is unavailable or ineligible, retain a parked receipt/result. |
| **D06 — Idle completion** | Automatically wake a normally idle/completed root agent once when an eligible completion reaches a safe boundary. Do not auto-restart cancelled or failed runs, resurrect a terminal child, abandon a question, or wake from a stale pre-cancellation notification. |
| **D07 — Deadlines** | Keep the original execution deadline and budgets. Ctrl+B neither resets nor removes them. Existing cooperative limitations remain; backgrounding does not create a hard timeout for arbitrary blocking code. |
| **D08 — Human input** | Existing questions and approvals retain exact ownership and continuation identity. Backgrounding is not an answer. Child-owned work can still request attention. Code mode keeps its current restrictions on unsupported human/deferred continuations. |
| **D09 — Inspection/control** | BG adds a common TUI inventory and cancellation routing. Keep `bg_status` Bash-only and `agent_retrieve` child-specific, including the stable Bash invocation selector and PID compatibility. MSG-01 separately exposes retained messages/results through `messages`; it does not replace live status/log/control. A universal model-facing `jobs` facade is not required by either initial release. |
| **D10 — Resume** | Include a running `agent_resume` invocation in the child integration, using its existing lifecycle. Do not make `agent_retrieve` backgroundable. |
| **D11 — Keyboard precedence** | Active question/modal/completion overlays retain their input handling. Outside them, Ctrl+B takes precedence over editor cursor-left only when there is an eligible target or a background-target picker is active. With no eligible target, cursor-left remains unchanged. |

D01's picker is not an approval dialog. It chooses an invocation; eligibility and existing approvals are still enforced by the runtime.

### 4.1 Lifetime matrix

| Event | Execution | Completion/model behavior |
|---|---|---|
| Caller proceeds to another model turn | Continue under original owner and budget | Deliver at the next safe boundary |
| Root agent produces its normal final answer | Continue | Eligible completion can start a continuation |
| Child agent produces its final answer while owning detached work | Continue under the native owner; release full child execution state | Record and forward to root; never silently resume child |
| Explicit cancel of a run | Latch cancellation for active detached work in its owned scope; use native stop/cancel | Preserve partial/error evidence, suppress automatic wake |
| Cancel a question only | Keep existing question-cancellation semantics | Do not interpret it as cancellation of every background invocation |
| Switch main/child view | No change | No destination rewrite |
| TUI transport closes and the owned controller shuts down | Stop supported local/native work through the existing controller lifecycle | Park/recover known outcomes; no daemon guarantee |
| Controller restart | Reconcile owned records after acquiring the existing owner lock | Do not automatically repeat ambiguous side effects or restart agents because stale work completed |
| Worker loss while controller lives | Use existing supervision/recovery to establish terminal or unknown state | Record `unknown/interrupted` when outcome cannot be established; no automatic side-effect replay |
| History navigation only | Do not mutate history or change execution ownership | Keep late completion pending/parked until an explicit applicable continuation |
| Explicit history-tail discard | Do not reattach the result to the new branch | Cancel affected active work where supported; retain out-of-branch evidence without automatic injection |

The current TUI/controller relationship is not an independently detachable daemon connection. Do not promise that closing the TUI leaves background jobs running. [S16], [S29]

## 5. Non-negotiable invariants

**I01 — One invocation.** Ctrl+B never emits another `ExecuteToolCall`, starts another process, starts/resumes another child, changes arguments, or retries a remote operation.

**I02 — One caller resolution.** The original model tool call receives either its final result or a background acknowledgment. It never receives both as competing tool results.

**I03 — Acknowledgment is not terminal execution.** It must not emit a false `ToolExecutionEnd`, mark a child lifecycle terminal, release a live writer/resource reservation, clean a host bridge, or mark final delivery accepted.

**I04 — Existing owner remains responsible.** Processes, child deadlines, MCP connections, and code-mode IPC remain with their existing owners. Generic code stores references, not live native objects.

**I05 — Owner authority.** Only the run-control owner commits the disposition. The TUI requests it; a worker reports readiness/completion. Neither decides what history receives.

**I06 — Stable correlation.** All completion and control operations validate the original invocation identity and execution owner. PID, visible view, tool name, or “latest running call” alone is insufficient.

**I07 — Separate admission.** A resolved caller wait is not proof that execution capacity or a sequential/writer restriction has been released.

**I08 — No lost or promoted output.** Preserve a terminal result or an owned result reference. Deliver bounded external-tool data with explicit provenance, never as a new human instruction or trusted system instruction.

**I09 — Sticky cancellation.** A subsequent turn, cleared operational projection, or reopened run must not erase an already accepted cancellation request for detached work.

**I10 — No hidden replay.** Replay/recovery may reapply coordination and resend a known result identity. It must not automatically rerun an ambiguously completed side-effecting operation.

**I11 — Safety boundaries stay intact.** The original approval, tool access, schema validation, output processing, resource ownership, and result representation checks remain in force.

**I12 — Bounded lifetime of coordination state.** Keep small active references/receipts, not finished child `RunState` graphs, transcripts, unbounded output tails, or copied fork contexts.

**I13 — Retention is not scheduling.** A mailbox item is a delivery obligation; an immutable message is retained evidence. Applying, parking, forwarding, or inspecting the former cannot erase or rewrite the latter.

**I14 — Reading is not settlement.** Native completion, reservation release, and canonical acceptance never depend on a model calling a retrieval tool or acknowledging a notification.

**I15 — No inbox prerequisite.** BG-01–04 must deliver useful completion information and supported native inspection without exposing or calling the companion's tools.

## 6. Identity, state, and persistence

### 6.1 Invocation identity

Use the original execution tuple as the authoritative identity:

```text
origin_run_id
origin_turn_no
origin_step_id
tool_call_id
```

Also retain the captured tool name, original message/idempotency identity, and a fingerprint sufficient to reject a mismatched execution envelope. Retries of the same execution identity do not become new invocations merely because a Messenger envelope is redelivered.

Expose a compact `invocation_id` as a stable alias. It is not a bearer credential: all lookup and control still require scope authorization. Reuse the existing native `deferred_id` for registered child calls. For retained-worker calls, derive/allocate the alias once using the project's existing identifier facilities and bind it to the tuple. Do not change the existing `(run_id, tool_call_id)` uniqueness contract without a separate compatibility migration; validate the stronger tuple when loading that row.

An accepted invocation additionally records:

```text
controller_run_id                 # trusted root/controller scope
execution_owner_kind              # implemented internal adapter kind
execution_owner_reference         # small typed reference, not PHP objects
original_execution_mode
original_deadline_metadata        # immutable; execution owner enforces it
origin_wake_generation
root_wake_generation
```

For Bash the native reference includes the immutable process **record ID**. PID and log path are presentation/inspection fields. For child tools it is the batch lifecycle ID and existing artifact references. For retained code mode it is the top-level worker invocation identity; the live host/connection are not serialized.

### 6.2 Separate state dimensions

Extend the existing deferred correlation rather than treating its current `status` string as every lifecycle at once.

| Dimension | Values / meaning |
|---|---|
| Caller disposition | `foreground_pending`, `background_acknowledged`, `foreground_result_accepted` |
| Execution observation | `active`, `cancel_requested`, `terminal`, `unknown` |
| Final acceptance | No terminal receipt, or one canonical accepted `completion_id` and result/reference |
| Model delivery | `not_applicable`, `pending`, `applied`, `forwarded`, or `parked` with a reason |

`terminal` is not synonymous with success. Preserve success, failure, cancellation, timeout, or unknown outcome in typed terminal metadata when the execution owner can establish it.

Existing deferred rows can remain `status=pending` until a **true final outcome is accepted**. Add separate detachment/disposition fields instead of assigning `completed` on Ctrl+B. `status=completed` must no longer mean merely that a result was sent to a transport. Rename repository operations internally to express the accepted-final meaning where practical.

Valid principal transitions:

```text
foreground_pending + active
    -- true result accepted --> foreground_result_accepted + terminal
    -- background accepted --> background_acknowledged + active

background_acknowledged + active
    -- cancel accepted --> background_acknowledged + cancel_requested
    -- true result accepted --> background_acknowledged + terminal + pending delivery

background_acknowledged + cancel_requested
    -- stopped/result observed --> terminal + parked-or-pending delivery
    -- native outcome cannot be established --> unknown + parked evidence
```

An execution becoming physically terminal does not by itself resolve the caller. The run-control owner's accepted event order decides whether the call receives a final result or an acknowledgment followed by completion.

### 6.3 Where each piece belongs

| Data | Owner/storage |
|---|---|
| Readiness of a currently active foreground call | Existing current-tool/batch projection, with a small native owner reference; canonical event sufficient for replay |
| Detached correlation and active reservation | Extended `deferred_tool_completion`, using the existing application DB |
| Accepted disposition, terminal receipt, retained completion envelope | Canonical run events, committed through `RunCommit`; one accepted receipt contains the immutable message header/body reference |
| Pending model notification | Existing command mailbox containing message identity/reference plus recipient delivery policy; not the only copy of the message |
| Received/applied/parked delivery evidence | Recipient-owned canonical events with narrow disposable lookup projections; receiving is distinct from inclusion in model history |
| Future model inbox queries | MSG-01 indexes the above evidence; it does not create another authoritative message store |
| Bash output | Existing managed-process log/artifact |
| Child result | Existing artifact/handoff machinery |
| Structured code-mode/MCP result | Owned result artifact in existing session storage, added at the execution-specific boundary when necessary |
| Live script/IPC/MCP client | Existing worker/native execution owner only |

Do not put a complete `RunState`, provider prompt, fork context, live object, or full log into the detached row. Reuse the existing captured execution correlation; do not copy immutable launch-input contents into another record.

For Bash and code mode, a new deferred row is needed when detachment is accepted, not as a speculative registry for every ordinary tool call. Child invocations keep their already-existing deferred registration.

### 6.4 Local transaction and projection changes

Add typed local coordination actions for accepting detachment, accepting a final outcome, latching cancellation, and recording delivery disposition. Integrate their validation, normalization, and application with the existing prepared-transition mechanism. The logical operation must cover:

1. Captured caller/batch delta.
2. Detached correlation and reservation delta.
3. Retained-message lookup/recipient-delivery delta and pending notification mailbox delta, when applicable.

The retained message itself is in the canonical transition, not an independent SQL payload write. For same-run completion, terminal acceptance, message retention, and the initial delivery obligation are one prepared transition. Cross-run forwarding uses two owner transitions joined by a stable message identity (§9.4), never a cross-run transaction.

`LocalMetadataCoordinator` currently recognizes batch/mailbox actions explicitly. Extend that contract deliberately; do not place arbitrary callbacks or native objects into the prepared transition. All fallible payload preparation occurs before the short database transaction, following its existing pattern. [S13]

Canonical append is the authority, and verified transition finalization repairs interrupted local application. This is not a claim that an event-log append, database transaction, and Messenger send form one distributed atomic transaction.

### 6.5 Retention and duplicate protection

Keep an active detached row until native execution is terminal/unknown and delivery is applied or explicitly parked/forwarded under the owner policy. A compact accepted-completion tombstone and retained message locator must survive for the root session's retained evidence lifetime, either in narrow projections or rebuildable from canonical events. Recognize duplicates before reading a reclaimed or damaged artifact.

Releasing a worker, deleting an obsolete caller batch, consuming a mailbox entry, compacting prompt context, or releasing a terminal child graph must not delete the message header, its bounded useful presentation, or its promised result evidence. Preserve immutable result references and their integrity metadata; §7.5 and §14.4 define cleanup gates.

Do not retain a whole completed batch in memory merely to remember one detached execution. Move its small reservation into the common correlation in the same verified transition. Similarly, an inbox must not retain full child transcripts/`RunState` objects just to serve IDs.

The guarantee is local retained evidence, not permanent remote-resource availability or disaster recovery from missing session files. Explicit root-session evidence deletion can remove the message and its owned bodies. An unexpected missing/corrupt body returns a typed inspection failure; it never schedules a repeat execution.

## 7. Internal protocol

These are internal runtime messages/events. They do not add model-facing tools or permit models/extensions to background themselves by returning a particular JSON flag.

### 7.1 Readiness report

**Capability wiring:** use small internal execution-owner adapters in `CodingAgent`, selected through Symfony dependency injection behind an `AgentCore` contract where the layering boundary requires one. The first release registers only the Bash adapter. A later adapter supplies native reference validation and native inspect/cancel/result handling; the shared owner still owns waiting and delivery. Do not add a second scheduler, a public extension registration API, or an MCP placeholder implementation. Eligibility requires a registered adapter **and** an exact top-level readiness report. Never infer it from parallel mode, tool-name prefixes, PID presence, or tool-returned JSON.

Proposed `ToolInvocationReadyForBackgrounding` carries the exact invocation tuple and a typed native owner reference. It is sent only after the relevant execution has actually started and all pre-execution gates for that invocation have passed.

- Bash reports after `BackgroundProcessManager::start()` returns the owned record.
- Child tools report through their durable deferred-registration path after native lifecycle ownership exists.
- Code mode reports at the top-level execution boundary after validation, before entering a potentially blocking host/nested-tool call; the existing worker owns the remaining startup/execution even if the script is still initializing.
- MCP reports only when its later adapter can retain and identify the in-flight request.

The report adds eligibility; it does not resolve the caller. Late readiness for an already resolved/cancelled invocation is ignored. A ready report from a nested code-mode tool is not independently registered.

### 7.2 Background request

Proposed controller command:

```json
{
  "type": "background_tool",
  "request_id": "ui-generated-stable-request-id",
  "run_id": "origin-run-id",
  "turn_no": 123,
  "step_id": "original-step-id",
  "tool_call_id": "original-model-call-id"
}
```

The controller verifies that the target belongs to its root/child scope, then routes a typed owner message through `agent.command.bus` / `run_control`. The command must not execute on the blocked tool/MCP worker.

The owner responds with a structured accepted or rejected outcome. Rejections include `not_running`, `not_ready`, `not_backgroundable`, `already_resolved`, `cancelling`, `wrong_owner`, and `stale_identity`. An already-accepted request for the same invocation returns its existing acknowledgment identity, not another event/model message.

### 7.3 Original tool acknowledgment

The owner creates the acknowledgment; it is not a handler return value and is not processed as a successful terminal execution in `ToolExecutor`.

Example model-visible body:

```json
{
  "status": "running_in_background",
  "invocation_id": "stable-invocation-id",
  "origin_run_id": "origin-run-id",
  "tool_call_id": "original-model-call-id",
  "tool": "bash",
  "completion": "automatic",
  "inspection": {
    "tool": "bg_status",
    "invocation_id": "stable-invocation-id",
    "pid": 12345
  }
}
```

The acknowledgment uses the **original** tool-call ID as that call's one provider-visible resolution. It is ordered alongside sibling results in the existing batch. `status` must not say `success`, `finished`, or `completed`.

Keep the acknowledgment small and stable. Include a native artifact identifier for child invocations when available; do not include task prompts, arbitrary command output, or a fabricated final handoff.

### 7.4 Terminal result admission

Retained workers continue returning their ordinary normalized final `ToolCallResult`. Before the ordinary stale-step/duplicate-result path, the owner checks for a matching detached correlation and routes that result to detached completion acceptance.

Native child owners can continue publishing `CompleteDeferredToolCall`. Its handler resolves the correlation and creates the final result envelope as today, but must leave the row pending until owner acceptance. Sending is not final acceptance, and an acknowledgment is never sent through `CompleteDeferredToolCall`.

Use a deterministic `completion_id`, for example a hash of `background-result:v1` and the stable invocation identity. This is one terminal receipt per invocation, not a mutable stream of result revisions. A conflicting second terminal result is an invariant violation to report, not a newer result that silently replaces the first.

### 7.5 Shared retained-message and delivery contract — version 1

This section is the single normative contract used by both specifications. BG implements it first for detached completions. The companion adds authorized producers and model-facing reads in later phases. The names below are proposed internal types; they do not create a public extension API.

#### SC-01 — Three separate objects

1. **Retained message:** immutable origin, bounded content, and owned body/artifact references accepted in canonical history.
2. **Recipient delivery:** mutable-by-events state for one intended recipient: received, pending, applied, parked, or rejected, with wake/activity/history fences.
3. **Mailbox entry:** an internal pending instruction to consider that delivery at a safe boundary. It contains a stable message reference and policy, not the sole copy of the message body.

The message is evidence. The delivery is recipient bookkeeping. The mailbox is scheduling. None is the execution owner or a new job registry.

#### SC-02 — Identity and envelope

For detached completion, **`message_id = completion_id`**. Allocate the deterministic identity described in §7.4 once per original invocation. Keep it unchanged after retries, forwarding, compaction, or retrieval. Other producer kinds introduced by MSG use their own domain-separated deterministic IDs; they cannot impersonate runtime completion IDs.

Conceptual retained envelope:

```text
schema_version: 1
message_id: stable completion_id
kind: tool_completion
origin:
    controller_run_id
    source_run_id / origin_run_id
    origin_turn_no / origin_step_id / tool_call_id
    invocation_id / execution_owner_kind
    accepted_source_event_reference
provenance:
    producer: runtime
    payload_authority: external_tool_result
presentation:
    severity / terminal_outcome
    subject
    bounded_summary
body:
    inline validated terminal content OR owned immutable result reference
attachments:
    zero or more authorized native artifact/resource references
integrity:
    stable payload fingerprint; referenced bodies include bytes/hash/media type
```

`accepted_source_event_reference` is the owner-allocated canonical location, not supplied by the tool and not a requirement to guess a sequence before append. Resolve it during normal event allocation/finalization. Exclude self-referential location fields from the immutable semantic payload fingerprint; hash a versioned canonical normalization of origin, kind, content, and referenced-body digests.

The envelope does not contain live native objects, credential values, whole `RunState`, a copied fork context, or a claim that the model has read it. An ID is not a bearer capability. Output-controlled JSON cannot supply authoritative origin, routing, or outcome fields.

#### SC-03 — Recipient record

A recipient delivery references that envelope and records:

```text
destination_run_id
message_id
received_event_reference / recipient-order sequence
origin and destination wake-generation fences
applicable history/branch identity
optional forwarding source/destination evidence
state: pending | applied | parked | rejected
state_reason
applied_event_reference, when present
```

The recipient record is keyed uniquely by **`(destination_run_id, message_id)`**. A forwarded copy at the root and the original child receipt are distinct recipient records for the same message, not competing terminal outcomes. Recipient ordering uses that recipient's committed event order, not sender clocks or an invented global total order.

`applied` means the bounded presentation was committed into provider-bound canonical context. It does not mean the provider returned, the model understood the body, or the agent acted on it. Delivery/retrieval facts are not native completion markers.

#### SC-04 — Canonical authority and publication order

The source owner appends the retained completion envelope as part of its accepted terminal receipt. Local projection/mailbox changes go through existing verified transition finalization. For delivery to another run, dispatch a typed reference after source commit; the recipient validates the committed source, scope, identity, and fingerprint before recording receipt and its mailbox obligation under its own lock.

Retain the sender-side forwarding decision in the existing canonical event/coordination evidence so explicit repair can redispatch the **same message reference**. Do not introduce a new outbox table, publication journal, recurring repair service, or second message broker. A successful Messenger send is not receiver acceptance. Preserve the existing post-finalization transport-loss limitation and `/repair` boundary.

A recipient retry checks its accepted identity/fingerprint before fetching a body. Duplicate identical deliveries are no-ops; a conflicting payload for an existing ID is an invariant failure, not an edit.

#### SC-05 — Useful push, optional pull

Automatic delivery must include the invocation, actual terminal outcome, and a useful bounded summary/diagnostic. Never reduce every completion to “new message #id.” For small results, include the full validated body when it fits the boundary budget. For large results, include status, bounded summary, and existing native inspection references. MSG-01 later adds `messages.read` access over the same evidence.

Do not call an LLM just to summarize these notifications. Use runtime outcome metadata and existing native handoff/result summaries; clearly label truncated evidence. Preserve full available evidence through owned references rather than manufacturing a second child-handoff format.

Before MSG-01, delivery text must not instruct the model to call `messages`. After MSG-01, render retrieval guidance only when that tool is actually available to the recipient. The stored message content/identity does not change when tool exposure changes. Freeze each applied presentation in canonical history; never rewrite old prompt prefixes to add a new tool hint.

#### SC-06 — Bounds and result preservation

Proposed internal defaults, not existing settings:

| Boundary | Limit / policy |
|---|---|
| Subject | 256 UTF-8 bytes |
| Notification summary | 1 KiB UTF-8 bytes; outcome and actionable failure data must survive |
| Inline body | At most 8 KiB of validated serialized terminal data; otherwise an owned reference |
| Retained envelope | At most 16 KiB including fixed metadata/references; large arrays or reference lists move to an owned manifest |
| Combined automatic presentation | 16 KiB at one model boundary, including wrappers and hints (§9.1) |

All limits are byte-based, with valid UTF-8 boundaries and existing representation-depth checks. If the configured downstream output limit is smaller, honor the smaller effective budget. Never process an unbounded body simply to find a small preview. Apply terminal result processors at the true terminal boundary; mailbox/retrieval rendering must not repeat processors or side effects.

A body reference describes a sealed local result or an existing artifact generation. If a native handoff/log can later change, bind to an immutable version or preserve a terminal snapshot once through existing session artifact facilities. Stream large copies; never duplicate full bodies into SQL, mailbox rows, and retained PHP state. External resource links remain external and are labelled with their availability limitations.

#### SC-07 — Retention and non-destructive reads

Retain headers, summaries, and promised local result bodies while the owning root session retains this message evidence. Prompt compaction is not evidence deletion. Mailbox consumption is not evidence deletion. Child resume must not mutate the body referenced by an older completion. Native temp cleanup may run only after its durable replacement is sealed and referenced.

Use existing session/artifact cleanup ownership; add a narrow hold/reference check for retained messages where required. This is not an independent garbage-collection scheduler. Explicit root-session evidence deletion removes the scope together. Local body loss or integrity failure is reported as `body_missing`, `body_corrupt`, or the existing representation error; it never triggers tool/child reexecution. Remote resource expiry is not hidden by reporting the original invocation failed retroactively.

MSG-01 reads are repeatable and non-destructive. Neither reading nor application transfers process ownership, releases capacity, settles a question, or grants access to another run's artifacts.

#### SC-08 — Delivery gates and cancellation

Use §9's owner mailbox policy, generation fences, terminal-child forwarding, and history-selection rules. A received message can be retained while delivery is parked. No data message may clear human input, interrupt a provider request, reattach a discarded branch, or revive a failed/cancelled run. Cross-run acceptance and application both validate destination eligibility.

The companion adds child-activity fences for authored messages using the existing launch/resume invocation identity. It cannot relax these background-completion fences. Reading parked evidence during an explicit authorized continuation does not restore its old permission to auto-wake.

#### SC-09 — Compatibility and producer extension

Version the discriminated message envelope. BG initially recognizes only the implemented `tool_completion` producer. The companion adds separately validated `agent_notice`, `parent_steering`, `question`, and `reply` variants as its phases ship. Do not enable their tool schemas or accept arbitrary kinds early.

An unknown kind/version is retained or quarantined as inspectable unsupported evidence and does not execute control logic or auto-wake. An upgrade requires compatible controller/worker versions; no mixed-version live-queue guarantee is added. The model-visible index in MSG-01 is rebuildable from these canonical records and points to existing body owners.

#### SC-10 — Completion is independent of inbox consumption

The native owner reports terminal evidence; run control accepts it, releases the applicable execution reservation, and preserves the result **before and independently of** any inbox read. A receiver that never reads its message must not hold a worker, child lifecycle, or caller tool batch open. Full model execution graphs remain releasable.

BG's TUI inventory and native tools stay operational without MSG-01. MSG-01 must pass an upgrade test against messages produced by BG alone, without rewriting their IDs, resending their notifications, or asking the tools to run again.

## 8. Owner algorithms and race behavior

### 8.1 Accepting Ctrl+B

Under the existing per-run owner lock:

1. Finish/assert any prior prepared transition using the existing mechanism.
2. Locate the exact original call and its current readiness/native ownership reference.
3. Reject queued/not-started calls, ineligible calls, mismatched identity, cancelled ownership, and calls whose caller result has already been accepted.
4. For a native deferred call, load its existing correlation without changing lifecycle-terminal markers. For a retained-worker call, prepare a detached correlation from the captured original call.
5. Build the acknowledgment and a dedicated batch **wait-resolution** delta.
6. Transfer, rather than discard, that call's active execution reservation into the detached correlation.
7. Prepare `tool_backgrounded` with the acknowledgment, origin identity, native reference, and wake fences. Capture all local changes in the prepared transition.
8. Commit and finalize through `RunCommit`; only then publish accepted UI state and any permitted successor effects.

Do not wait for a callback from a blocked code-mode host or MCP worker. Readiness has already established retained execution ownership. For Bash, exposing the existing record as accepted background work is an idempotent local/native-adapter update; it does not start or stop the process.

### 8.2 Finishing a retained or native-deferred execution

Under the original run's owner lock:

1. Resolve the exact correlation and check for an already accepted terminal receipt before loading an artifact or reconstructing a native result. Compare the stored result fingerprint to reject conflicting duplicates, then validate any genuinely new terminal representation.
2. If the original call is still foreground-pending, process its ordinary final result. A later Ctrl+B is rejected.
3. If it was acknowledged in the background, accept the actual outcome into the detached lifecycle, not the original batch's result slot.
4. Commit the true `ToolExecutionEnd` with the original correlation and a canonical accepted completion receipt/result reference. Record origin fields separately from the current append sequence/current run turn.
5. Release the detached execution reservation. Reevaluate the owning run's **current** pending tool batch using the existing collector/admission path; it may no longer be the original batch.
6. Retain the immutable completion envelope under the same `message_id = completion_id`; prepare its recipient record and an internal mailbox reference or a parked/forwarded disposition according to §9. Do not make delivery/read completion a condition of terminal acceptance.
7. Finalize metadata and effects through the existing transition machinery.

The late result must not be rejected merely because `activeStepId` advanced after the acknowledgment. It must not replace the acknowledgment or create a second original-call `ToolExecutionEnd` for a synthetic acknowledgment.

If output cannot be preserved or represented, accept a bounded diagnostic terminal outcome with a clear artifact/representation failure, rather than retrying the side effect. Preserve available native evidence.

### 8.3 Completion racing with backgrounding

| First owner-accepted transition | Result |
|---|---|
| True foreground result | Ordinary final tool result; background request is rejected/no-op |
| Background acceptance | Exactly one acknowledgment; later terminal result becomes one separate completion |
| Cancellation while foreground-pending | Existing cancellation resolves/settles the call; background request is rejected |
| Background acceptance, then cancellation | Acknowledgment remains; native cancellation is requested; terminal/partial evidence is retained without automatic restart |
| Duplicate background request | Return existing disposition; no new execution or model message |
| Duplicate terminal message | Reuse/ignore accepted receipt; do not reopen batch or inject another completion |

Physical OS exit just before an accepted background request is not a contradiction if its result has not yet been accepted by the owner. The visible sequence can legitimately be “backgrounded” followed immediately by “completed.” Do not try to arbitrate this with unsynchronized wall-clock timestamps.

### 8.4 Critical prepared-transition failure points

| Failure point | Required recovery |
|---|---|
| Before canonical append | No accepted acknowledgment; original supervision continues |
| After append, before local metadata finalization | Existing verified-transition repair applies the captured disposition, batch, and reservation changes |
| After local finalization, before UI event arrives | Canonical replay/snapshot shows the accepted disposition; a repeated UI request is idempotent |
| After finalization, before successor/result transport delivery | Use the existing repair boundary to redispatch known coordination/result identity; do not invent a new retry journal |
| After terminal acceptance, before mailbox application | Existing mailbox retains the typed completion; no second terminal acceptance |
| After mailbox application, before rendering | Replay shows one completion card/message keyed by `message_id = completion_id`; retained evidence remains readable |
| After compaction or cleanup of consumed mailbox entries | Retained canonical message/reference survives; rebuild only its disposable lookup projection, never reexecute |

## 9. Automatic delivery, provenance, and safe boundaries

### 9.1 Extend the existing mailbox, not `append_message`

Add an internal `tool_completion` command kind with a typed reference to the already retained message and its recipient policy. Only the owner completion path can enqueue it in the BG release. It is not a model-visible mailbox; MSG later exposes a read model over retained messages, never raw runtime commands. Do not expose it as arbitrary user text and do not route it through `UserMessageHandler` as an ordinary follow-up.

`CommandMailboxPolicy` must handle this kind explicitly. It must not inherit “any non-compaction command restarts the run and clears human waits.” Update `AdvanceRunHandler` to distinguish an explicitly authorised user continuation from an automatically delivered tool notification. [S10], [S11]

| Destination state | Required action |
|---|---|
| Running model step | Retain pending; never mutate the request already in flight |
| Foreground tool batch unresolved | Retain pending until every original call has a valid result/acknowledgment |
| Safe turn boundary | Append bounded completion message(s) before the next model request |
| Waiting for human input | Retain pending and show attention/status without clearing, answering, or replacing the question |
| Compacting | Keep pending through compaction; apply to the post-compaction context at the next safe boundary |
| Normally completed/idle root | Apply pending eligible completion(s), then dispatch one valid `AdvanceRun` successor |
| Cancelled/failed root | Park; no automatic restart |
| Terminal child | Commit origin receipt, release child state, then forward to the recorded root through its owner |
| Viewing earlier history / discarded origin branch | Do not discard the forward tail or inject into another branch; park until explicit applicable continuation |

A completion without a permitted automatic wake remains pending/parked until an explicit user continuation or inspection; merely delivering a UI status event is not permission to start a model. Several completions already pending at a boundary should be applied in stable mailbox order and coalesced into one successor model step. The existing `AdvanceRun` identity/current-operation guards remain authoritative. Never poll from the model to discover completion.

Applying the mailbox entry records the exact provider-bound presentation and advances only that recipient's delivery state. Consuming/compacting the mailbox never consumes the underlying message. A retrieval tool call and its ordinary tool result are not another automatic-delivery event.

Use a fixed internal byte budget for the combined completion content added at one boundary, in addition to existing result caps. Suggested first-release budget: **16 KiB** total notification presentation. Leave excess notifications pending and drain them on subsequent safe boundaries in recipient acceptance order. This budget is shared with companion-message delivery after MSG ships; do not give each subsystem its own 16 KiB allowance or let new arrivals starve old entries. A single oversized item must still make progress as a bounded summary plus artifact reference; never starve forever behind the budget. This is an implementation constant, not a new public setting.

### 9.2 Preserve provenance using existing custom messages

Represent the applied canonical message as an `AgentMessage` with a custom role, for example `tool_completion`, and a `ModelNotificationDTO` whose `source` is the runtime tool-completion adapter. Keep `toolCallId` correlation in metadata, but do not make this a second ordinary `role=tool` message resolving the old call.

The inspected `AgentMessageConverter` already maps custom roles to a provider `UserMessage` with a role prefix. Therefore this proposal does **not** claim a provider-native low-trust notification role. Use that existing compatibility path with an explicit, deterministic provenance wrapper, and test its conversion across the supported provider adapters. The wrapper must be present in provider-bound text; metadata alone is insufficient. [S21], [S22]

Example presentation:

```text
[tool_completion] Runtime-delivered result of an earlier tool invocation.
This is external tool output, not a new human request.
Message: <message_id>
Origin: run=<id>, invocation=<id>, tool=bash, outcome=failed.
Result summary: <bounded, JSON-escaped or equivalently delimited data>
Full result: <owned artifact reference>
```

Required precautions:

- The TUI labels this as **tool completion**, never as a human-submitted prompt or an assistant-authored answer.
- The structured source/origin fields come from the runtime, not from tool-controlled output.
- Tool text cannot replace the wrapper, set notification routing, declare itself approved, or request a wake by returning metadata-shaped JSON.
- Add one compact model instruction that custom tool-completion messages are external results, not new human directions. Do not promote their text to a system/developer message.
- Preserve the wrapper through reasoning-transition conversion, history reconstruction, compaction input, and notifications. A prefix alone is not a security sandbox or a guarantee against prompt injection.
- No fake assistant tool calls or second result for the original tool-call ID are needed. BG exposes no retrieval schema; MSG-01 later adds an ordinary authorized tool over retained evidence, not a replacement provider protocol.

Provider-history validation must run on an acknowledgment followed later by this custom message. The feature is not shippable if an adapter loses provenance or creates unmatched tool-call sequences.

### 9.3 Wake fencing

Persist a small monotonic **wake generation** in the existing run operational/canonical state and increment it on explicit run cancellation. A detached correlation captures its origin generation and its recorded root destination generation.

Before enqueuing an automatic successor, and again when the mailbox item is applied, validate the destination's current generation. A cancellation that occurred after detachment suppresses an old completion's automatic wake even when the user subsequently starts a new turn and the run status becomes `Running` again.

This is separate from execution cancellation. The sticky per-invocation cancellation field controls whether native work must stop; the wake generation controls whether its result may autonomously restart model work. A late cancelled invocation cannot become eligible just because the current status is no longer `Cancelled`.

### 9.4 Forwarding after a child finishes

The origin run first accepts the terminal receipt without changing its terminal agent status. It may be temporarily rehydrated through the existing owner path, but must be released again after finalization. Forward the retained envelope/reference to the explicitly recorded root using the **same `message_id`** and a separate recipient delivery record. Preserve a canonical forwarding decision for explicit repair; do not copy or regenerate the full body. The root owner independently accepts/deduplicates the forwarded completion under its own lock.

Do not hold two run locks or append directly to the parent's log from a child worker. Do not recreate or resume the child. Do not rewrite a previously completed child handoff in place.

For a forwarding root that is normally idle, the same automatic-wake policy applies only if its captured root wake generation still matches. Root inspection of child-owned artifacts requires the existing parent/child ownership relationship. Forwarding a result does not transfer ownership of its native process or permit arbitrary cross-session lookup.

## 10. Execution capacity and ordering

### 10.1 Retained worker is a deliberate first-release choice

For Bash, retaining the worker avoids a new monitoring handoff and preserves the existing monotonic deadline. For code mode it also preserves the host stack and IPC. For MCP it can retain the in-flight client call without pretending the single MCP consumer becomes concurrent.

Consequences must be visible:

- The parent can take another model step after its foreground obligations resolve, even though execution continues.
- Other tool calls can execute only when ordinary pool capacity and ordering permit.
- With one tool worker occupied by a background Bash/code-mode invocation, another tool on that pool waits. This includes model-issued `bg_status` inspection/stop calls and, after MSG-01, `messages` reads on that pool; the TUI inventory/cancel request must go directly through run control and remain responsive. The acknowledgment must not promise otherwise.
- Backgrounding several invocations must not spawn unbounded consumers or bypass `tools.execution.max_parallelism`.

This is a capacity change from today's Bash early-return path. Include it in release notes and test the single-worker case. It is preferable to silently losing deadlines, inventing a second supervisor, or claiming that a PHP function can continue after its execution stack has been unwound.

A future optimization may transfer Bash supervision fully to `BackgroundProcessManager` and existing runtime control. That requires preserving its deadline, cancellation, final-result construction, shutdown, and lost-worker semantics before freeing the worker. It is not part of the first release and must not be a prerequisite for it.

### 10.2 Extend the existing admission accounting

The common subsystem must distinguish **caller obligations**, **logical execution reservations**, and **physical worker occupancy**.

Add a dedicated collector operation, conceptually:

```text
prepareBackgroundAcknowledgement(original_call, acknowledgement, detached_reservation)
```

It records the original caller resolution and can complete its tool batch, but moves the existing execution reservation into the detached correlation in the same verified transaction. It must not invoke ordinary final-result collection and discard `inFlight` without preserving that reservation elsewhere.

The existing `ToolBatchCollector`/store admission calculation must consider active detached reservations for the same originating run in addition to the current batch. Read them through a narrow indexed projection, not by replaying every old batch or loading finished child histories.

Maintain the original registration mode. A detached sequential invocation continues to impose the applicable same-run exclusion until true completion. This prevents a later model turn from accidentally bypassing a constraint merely because its old caller was acknowledged. It does not create a global lock over unrelated runs or tools.

On true completion/cancellation confirmation, remove the detached reservation and reevaluate any current queued batch. Prepare that release and newly dispatchable calls together so a lost capacity-release message cannot leave the run permanently waiting.

Background capability must never be computed from `ToolExecutionMode`. In particular, non-backgroundable `read` and mutation tools must not be forced into a new universal serialized lane.

This is an extension of the existing admission machinery required by calls spanning turns, not a replacement tool scheduler or a promise of parallel MCP/nested-code execution.

## 11. Execution-specific implementation

### 11.1 Bash — first independently shippable integration

**Change `BashTool`:**

1. Keep its one call to `BackgroundProcessManager::start()` and use the immutable returned record ID.
2. Report top-level readiness after the start succeeds, with exact ambient run/turn/step/tool-call identity.
3. Delete the threshold timing, question invocation, decline branch, and background-triggered early return. Keep normal completion, timeout, cancellation, bounded output, and failure handling.
4. Remove child-run classification used solely to suppress the old question. Child Bash eligibility now follows explicit invocation ownership and the same common lifecycle.
5. Continue using the original monotonic deadline after acknowledgment. Do not construct a new deadline on detachment or the next loop iteration.
6. Supply typed timeout/cancellation/stop reason at the Bash result/native boundary so completion presentation does not have to recognize a timeout by parsing message text. Preserve existing useful output and diagnostics.
7. Use the invocation-aware cancellation token described in §12, so post-execution cancellation arbitration cannot lose a previously accepted stop request.

The live loop may observe that the record was marked accepted-background for inspection, but it must not be responsible for resolving the model's wait. The run owner can do that without waiting for the next 100-ms Bash poll.

**Change process visibility and inspection:**

- Reuse the accepted-background flag/native record rather than create another OS process record.
- `bg_status` remains Bash-only. Add `invocation_id` as a stable selector alternative to PID, with mutually exclusive argument validation.
- Resolve it through the common correlation to the immutable Bash record ID; enforce same-run or proven controller-parent/child ownership. Do not accept an arbitrary run ID as authorization.
- Keep existing PID usage for compatibility, but use the stable selector in new acknowledgments and notifications.
- Preserve private-foreground visibility restrictions. A ready-but-not-detached Bash command must not suddenly become globally listable/stoppable through `bg_status`.
- For a correlated invocation, `bg_status stop` submits a typed cancellation request to the run owner. The owner commits the sticky intent before dispatching the existing native stop effect. Report **stop requested** until native termination is observed, rather than mutating common state from the tool worker or claiming immediate rollback/termination. Legacy uncorrelated PID records retain their native inspection/stop path.

**Replace old completion delivery:**

The retained worker's normalized final result is the normal completion producer. Remove the poller's independent `append_message` publication for newly managed invocations. Do not allow both a native process observer and the worker to publish independently accepted final presentations for one invocation.

Retain/extract the poller's useful owned-session scope resolution where the controller lifecycle needs it. Native status inspection remains useful for startup/repair. A fallback observer may submit an identity-qualified observation to the common owner, but must not directly mark final delivery or invent a second result after an accepted worker outcome.

**Delete superseded configuration and instructions:**

Remove `backgroundPromptThresholdSeconds` and its serialized setting, threshold description text, Bash-only background-question adapters that become unused, related DI wiring, and obsolete tests/documentation. Retain actual execution timeout, poll interval, output cap, process-stop grace, and log configuration.

Do not remove generic `ToolQuestion` facilities that other tools still use. For a user config containing the retired threshold key, provide a clear obsolete-setting diagnostic through the existing configuration-validation path; do not silently preserve a nonfunctional switch.

**Cleanup:**

`BackgroundProcessControllerSessionLifecycleListener` currently stops and deletes accepted process sidecars and rows at startup/shutdown. Reorder it so a known terminal/partial outcome is durably accepted or preserved for repair before its last evidence is deleted. If evidence cannot be persisted, keep the exact owned files/row for later reconciliation rather than falsely claiming notification succeeded. This does not require keeping the native process alive during shutdown. [S29]

### 11.2 Subagent, fork, and resume integration

Use the existing durable child batch as the execution owner. Detachment changes the parent call's wait disposition only.

Required changes:

- Publish readiness from the registered native deferred execution, preserving lifecycle ID, child identities, and artifact ownership.
- Include `agent_resume` alongside `subagent` and `fork`; each resume invocation gets its own original tool-call correlation even though the child run identity is reused.
- Keep whole-invocation batch semantics. Partial child outcomes can update the existing progress/TUI surfaces; the final completion still uses the ordinary aggregate handoff.
- Do not set `DeferredToolCompletion.status=completed`, `terminalCompletionEnqueuedAt`, child terminal status, or handoff completion because the parent received an acknowledgment.
- Preserve deadlines, `DeferredSubagentBatchInterruptionService`, observation cursors, recovery, and native child cancellation.
- Refactor `CompleteDeferredToolCallHandler` so generic completed status is finalized by true owner acceptance, not its earlier transport send.
- Keep launch-input cleanup tied to the existing durable registration/native lifecycle guarantees. Ctrl+B must not delete inputs merely because the parent is no longer waiting.
- When a child finishes while it still owns its own detached tool invocations, include a bounded outstanding-work reference in the handoff/status metadata. Do not imply all side effects have stopped or a shared checkout is safe for a new writer.
- Keep terminal-child `ActiveRunContext` release. Reconstruct required completed presentation from existing durable artifacts/projections rather than retaining the full execution graph.

Parent cancellation must reach detached descendant work even if the immediate child is already terminal and ordinary `AgentRunner::cancel()` has nothing left to cancel in that child agent. Use the common controller-scoped inventory for outstanding native invocations, not only the list of currently running child models.

### 11.3 Code mode integration

Background the complete top-level `code_mode` execution. The worker keeps running `CodeModeHostBridge::execute()` and its current IPC connection; Ctrl+B does not return from that method, trigger its `finally`, or detach just the PHP script.

Required changes:

- Report top-level readiness from the retained execution boundary independently of the synchronous `serve()`/nested-tool loop.
- Preserve the original host deadline, memory budget, process ownership, and cleanup.
- Propagate the same invocation cancellation context into nested tools, while retaining their separate nested call identities and synchronous `tool()` return semantics.
- Never deliver a background acknowledgment as the result of a nested `tool()` call. Only the top-level model call is acknowledged.
- Keep existing unsupported nested deferred/human-input behavior; backgrounding does not enable it.
- Retain the sequential reservation until the host actually finishes.
- Preserve `code_mode_value`, stdout/stderr diagnostics, and failure details through the existing terminal processing/output-cap path.
- Before workspace cleanup can remove the only copy, persist the recoverable structured terminal result to an owned session artifact where needed. The small completion notification references that result, not a Bash-style log.

A required integration test blocks inside a nested tool, backgrounds the outer invocation from another runtime process, verifies that the parent receives its acknowledgment, then releases the nested tool and verifies the structured final value and diagnostics. A script that merely sleeps outside nested `tool()` does not prove this contract.

### 11.4 Later MCP integration

Do not expose MCP background eligibility in the Bash release. The common protocol does not need dummy MCP APIs or a public plugin backgrounding interface to accommodate it.

When implemented:

- Keep the same worker/client pending request alive; retain required connection state until it resolves or native cancellation/disconnection completes.
- Report a stable invocation/request correlation, never an assumed process PID.
- Route wait release through run control even if the SDK call is blocking.
- Keep the original deadline and cooperative cancellation propagation already used by `McpToolInvoker`.
- Preserve structured content/resource references before `McpResultMapper` reduces the result to its existing string presentation.
- Distinguish cancellation requested, local call interrupted, transport disconnected, remote operation known stopped, and remote outcome unknown. A cancellation notification or closed response body is not proof of remote termination.
- Do not reconnect, resend, or replay the MCP tool call merely to background it.
- Keep MCP queue concurrency unchanged unless a separate concurrency change is implemented and verified. Backgrounding alone does not remove the single-consumer bottleneck.

## 12. Cancellation, permissions, and human attention

### 12.1 Invocation-aware cancellation

`RunCancellationToken` currently derives cancellation only from the run's present operational status. Detached execution can outlive that status and its original turn. [S23]

Introduce a composed/internal token implementation for background-capable top-level invocations:

```text
cancelled = existing run cancellation
         OR sticky cancellation intent for this invocation
         OR existing execution-owner shutdown/deadline cancellation
```

The common owner latches cancellation before dispatching a native cancel effect. A new user turn does not clear it. A missing live run projection must not erase a stored invocation-level intent.

Use the same token through `ToolRuntime`, `ToolExecutor`, and code-mode nested contexts. Keep run-level tokens for unrelated execution unchanged. Cancellation confirmation is separate from cancellation request, and partial effects remain real.

### 12.2 Approval and readiness

Do not accept backgrounding while a call is only queued or awaiting its initial approval. Preserve hook outcomes that block, replace, or require approval before execution. Returning tool-controlled data that looks like a background marker is not a capability grant.

A running child invocation may be detached while a child subsequently needs input, but the question remains owned by the child, visible as attention, and answerable only through its original continuation. Parent acknowledgment must not synthesize permission or erase the child question.

### 12.3 Writer ownership

A background acknowledgment is not a completion handoff and not permission to reuse a worktree while a native writer is active. Keep any existing enforced ownership checks tied to execution lifetime, not caller resolution. Where ownership is a workflow convention rather than a runtime lock, preserve that distinction in status/handoffs; this feature does not magically add filesystem isolation.

Normal parent continuation can perform unrelated work subject to current tool admission and permissions. The specification does not promise that arbitrary simultaneous Bash, raw PHP, and child operations are race-free.

## 13. TUI, runtime protocol, and event projection

Add a `BackgroundInvocationInputListener` following the existing `InputEvent` interception pattern rather than changing the editor's cursor-left implementation globally. Register it through the current TUI listener mechanism and an explicit `InputPriority` position below active modal/question/completion handling but before ordinary editor dispatch. `AppHotkeyRegistrar` supplies the hint; it is not itself the implementation of the hotkey. [S24], [S25]

The TUI must maintain small, identity-keyed eligible/running-background entries from runtime snapshots and canonical projections. Do not parse rendered tool text to infer candidates.

Request flow:

```text
Ctrl+B
  -> resolve visible owner run and candidate/picker
  -> send background_tool request with exact identity
  -> show "backgrounding requested"
  -> owner accepted/rejected event
  -> update tool card and background inventory
```

Do not display “backgrounded” before owner acceptance. Disable/reject stale picker rows without redirecting the action to a different current call. Changing live view while a request is in flight does not change its target.

Provide one TUI inventory surface, for example `/background`, listing invocation ID, originating run/child, tool, execution state, elapsed time/deadline when known, and native inspection/cancel actions. This is a TUI command, not an LLM retrieval tool. It can reuse existing picker/list widgets and status infrastructure.

Event changes:

- Add a canonical `tool_backgrounded` event for caller acknowledgment. Replay must reconstruct the acknowledged result and pending-batch disposition without pretending execution ended.
- Keep actual execution end separate. Late `ToolExecutionEnd` carries original invocation identity and detached disposition.
- Add/record an accepted terminal completion receipt and its pending/applied/forwarded/parked delivery state.
- Project one completion card per `(visible recipient, message_id)`; for BG `message_id = completion_id`. Receipt can display it immediately, and later mailbox application updates that card rather than adding a second copy. The companion inbox reuses this identity; it does not add another completion notification.
- Extend event enums/codecs/replayers/projection mappings together. Live rendering and replay must agree.
- Do not map internal completion application to `user.message_submitted` or put it into human-prompt `/history` navigation.
- Inventory restoration after attachment comes from canonical/narrow operational state, not sequence-zero transient events alone.

The existing child-view cancellation listener already treats the visible child as the target. Apply that identity discipline to backgrounding, without changing its question/Escape semantics. [S26]

## 14. Failure, recovery, and cleanup policy

### 14.1 What is guaranteed

In a healthy running controller, accepted background work produces automatic completion delivery without model polling. Accepted caller disposition and terminal result identity are deduplicated at the canonical owner boundary. Prepared local state changes use the existing verified-transition recovery.

### 14.2 What is not guaranteed

This feature does not provide exactly-once external execution, persistence of live PHP/MCP state across worker loss, or background survival after controller shutdown. Existing result-store reuse still does not exclude concurrent duplicate workers before an outcome is recorded. Do not market the new correlation row as solving all at-least-once execution risks. [S03], [S06], [S27]

After a detached row exists, worker redelivery must **not** enter its tool handler again. Update the existing deferred-registration fast path to distinguish native child-deferred ownership from retained-worker detachment: a retained-worker row must not emit child registration events, and neither kind permits a duplicate launch.

A duplicate while no detached registration exists retains the repository's existing limitations. Ctrl+B itself must introduce no new redispatch window. Any broader execution-claim/exactly-once redesign is a separate task.

### 14.3 Recovery actions

On startup/repair, after acquiring existing ownership:

1. Finalize pending verified canonical transitions.
2. Read small active detached correlations in the owned root/child scope.
3. Ask the existing native owner to establish a known result, live ownership, or unknown/interrupted outcome.
4. Reapply/resend known coordination or a known result identity through the same owner protocol.
5. Never re-execute unknown side-effecting work automatically.
6. Keep old automatic wakes disabled after controller restart; surface recovered/parked completions for explicit session continuation.

Do not add a new recurring repair scheduler. Use current native supervision/startup/recovery entry points and Messenger delivery. If a post-finalization transport loss strands a continuation, surface that it needs `/repair`; routine status polling by the model is not the workaround.

### 14.4 Result and resource cleanup

ACK alone makes nothing terminal and deletes no execution resource.

Local process shutdown is allowed before model delivery, but retain the result, bounded diagnostic, or exact native evidence needed to produce it. A missing output artifact becomes a typed inspection failure, not a reason to rerun the invocation.

Cleanup gates must distinguish:

```text
execution no longer live
terminal outcome/result evidence preserved
canonical receipt accepted or recoverably parked
no remaining native owner need for the resource
```

Keep active reservations and child/native supervision until their actual settlement. Release full terminal child graphs as at the pinned commit. Keep only bounded history-backed coordination data in memory, but keep promised local result bodies on disk for the retained message lifetime. Mailbox application/read is not permission to reclaim them. Seal/version native artifacts that are mutable across resume; replace temporary references before cleanup and validate their hashes on read. See SC-06–07. All cleanup remains scoped to exactly owned rows/paths/processes; backgrounding does not relax existing PID/process-group ownership checks.

## 15. File-level implementation map

Paths below are existing source files read directly or identified through inspected imports/repository search, unless explicitly marked **new**. Proposed names can follow project conventions, but the responsibilities and boundaries are required.

| Area | Existing files / proposed additions | Required change |
|---|---|---|
| Owner request | **New** internal request DTO/handler under `AgentCore/Domain/Message` and `AgentCore/Application/Pipeline` | Validate original identity/readiness; prepare atomic caller-acknowledgment transition |
| Readiness/native adapter | **New** internal readiness report, small projection fields, and CodingAgent-owned DI adapters behind the necessary AgentCore contract | Publish actual top-level eligibility; validate native references and route inspect/cancel without confusing queued/nested calls or introducing a public extension API |
| Final result routing | `src/AgentCore/Application/Pipeline/ToolCallResultHandler.php` | Match detached correlation before stale-step/ordinary duplicate handling; accept true late result independently |
| Batch admission | `src/AgentCore/Application/Handler/ToolBatchCollector.php`; its current DTO/store contract/implementation | Add wait-only resolution; migrate detached reservations; include them in current-run admission; release/reconsider on real completion |
| Deferred persistence | `src/CodingAgent/Entity/DeferredToolCompletion.php`; `src/CodingAgent/Entity/DeferredToolCompletionRepository.php`; `src/AgentCore/Contract/Tool/DeferredToolCompletionRepositoryInterface.php` | Add disposition, native reference, acceptance/delivery and cancellation fields; indexed active lookup; verified local update operations |
| Native deferred adapter | `src/AgentCore/Application/Handler/CompleteDeferredToolCallHandler.php` | Stop marking complete on send; route final envelope to canonical owner acceptance; do not finalize on ACK |
| Execution worker | `src/AgentCore/Application/Handler/ExecuteToolCallWorker.php` | Retain normal execution; distinguish retained-worker detached rows from native deferred registration on redelivery |
| Terminal processing | `src/AgentCore/Application/Handler/ToolExecutor.php` | Keep validation/processors/capping/cancellation semantics on true result only; ACK must not enter result-store reuse |
| Commit machinery | `src/AgentCore/Application/Pipeline/RunCommit.php`; `TransitionFinalizer.php`; `LocalMetadataCoordinator.php`; existing action validator/plan/serialization registry | Reuse prepared transitions; add typed local disposition/reservation/delivery deltas and validate replay |
| Mailbox policy | `src/AgentCore/Application/Pipeline/CommandMailboxPolicy.php`; `AdvanceRunHandler.php`; existing core-command kinds | Add internal completion kind, safe-boundary handling, generation fencing, and coalesced successor admission |
| Retained completion contract | **New** internal envelope/reference DTOs and narrow reader under the existing message/notification contracts; accepted completion event and existing session artifact stores | SC-01–10: retain evidence independently of mailbox consumption; use `message_id = completion_id`; preserve sealed bodies; do not expose model APIs yet |
| Evidence cleanup | Existing managed-log, output-cap, child-artifact and session cleanup owners | Hold/promote/version locally referenced result evidence before removal; use the existing cleanup lifecycle, not another scheduler |
| Canonical messages | `src/AgentCore/Domain/Message/AgentMessage.php`; `src/AgentCore/Domain/Notification/ModelNotificationDTO.php` / `ModelNotificationCodec.php` | Reuse custom-role/provenance representation, not ordinary human follow-up |
| Provider conversion | `src/AgentCore/Infrastructure/SymfonyAi/AgentMessageConverter.php`; existing history/reasoning conversion and compaction handling | Preserve origin wrapper and valid tool-call sequences; do not rebuild unrelated history prefixes unnecessarily |
| Cancellation | `src/AgentCore/Infrastructure/SymfonyAi/RunCancellationToken.php`; **new** invocation-aware composed token; existing cancel owner handler | Latch detached cancellation and wake generation; do not rely only on current RunStatus |
| Bash | `src/CodingAgent/Tool/BashTool.php`; `src/CodingAgent/Config/BashToolConfig.php` | Remove threshold question/early return; report readiness; keep loop/deadline and typed terminal reason |
| Bash inspection | `src/CodingAgent/Tool/BgStatusTool.php`; `src/CodingAgent/Tool/Arguments/BgStatusArgumentsDTO.php`; `BackgroundProcessManager`/record storage | Stable invocation selector and owner-verified native lookup; existing PID compatibility; preserve private visibility |
| Bash notifications/cleanup | `src/CodingAgent/Runtime/Controller/BackgroundProcessCompletionPoller.php`; `BackgroundProcessControllerSessionLifecycleListener.php` | Remove synthetic human publication; retain/extract owned scope/native reconciliation; preserve evidence before cleanup |
| Child completion | `src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Completion/DeferredSubagentBatchCompletionDispatcher.php` | Continue true handoff production; acknowledgment must not set native terminal markers |
| Child interruption/recovery | `src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Interruption/DeferredSubagentBatchInterruptionService.php`; existing lifecycle/recovery handlers | Keep deadlines/cancel/observation active after detach; complete only at real lifecycle settlement |
| Child memory | `src/AgentCore/Application/Pipeline/RunMessageProcessor.php`; `RunCommit.php` | Preserve terminal-child release on finalization and late no-op messages |
| Code mode | `src/CodingAgent/Tool/CodeModeTool.php`; `src/CodingAgent/Tool/CodeMode/CodeModeHostBridge.php`; existing diagnostics processor | Retain whole host execution, original budgets, nested semantics, and structured result artifact |
| MCP, later | `src/CodingAgent/Mcp/Tool/McpToolInvoker.php`; existing connection manager and result mapper | Retain pending request/client; capture structured result before string projection; preserve cooperative cancel limitations |
| TUI input | **New** `src/Tui/Listener/BackgroundInvocationInputListener.php`; `src/Tui/Layout/InputPriority.php`; `src/Tui/Listener/AppHotkeyRegistrar.php`; `src/Tui/Screen/ChatScreen.php` | Contextual Ctrl+B interception, picker, hint, no global cursor-left removal |
| TUI protocol/projection | Existing `UserCommand`, controller command/event mapping, runtime event translator, current run/tool projection; **new** inventory view action | Exact request identities; accepted/rejected outcomes; bounded restoration; distinct completion cards and no fake prompt-history entry |
| Configuration/routing | `config/services.yaml`; `config/packages/messenger.yaml`; existing event/action codecs and DB migrations | Route new owner messages to `run_control`; wire internal adapters; remove obsolete prompt-only wiring; migrate existing records |
| Documentation | `docs/background-processes.md`, `docs/tools.md`, `docs/tool-execution.md`, `docs/terminal-usage.md`, `docs/settings.md`, `architecture/backgrounding.md`, related architecture references | Replace threshold UX, explain budgets/capacity/lifetime, update event/ownership diagrams and model guidance |

Before editing an ancillary file identified through imports or architecture references, inspect its current contents and any more-specific `AGENTS.md`. This map is not permission to infer uninspected constructors, configuration key locations, or serialization registries.

## 16. Migration and compatibility

Use the existing application database migration process. Do not require a new database or transport.

- Existing pending native deferred rows default to `foreground_pending`; the backgrounding feature must not detach them implicitly.
- Existing completed rows stay completed under legacy interpretation. Do not replay their handoffs merely to manufacture new acceptance metadata.
- New rows/records use a protocol version so replay and redelivery can distinguish old semantics from the accepted-final lifecycle.
- Previously accepted Bash records without an invocation correlation cannot safely be attached to an arbitrary current tool call. Preserve native inspection/cleanup and label them legacy; do not invent an origin tool-call ID or inject an uncorrelated user message.
- Require a clean controller restart for the upgrade. Do not promise interoperability between old and new workers consuming the same session queue.
- Preserve the historical meaning of old `ToolExecutionEnd`, `ToolBatchCommitted`, and synthetic Bash-completion events when replaying old logs. New `tool_backgrounded` events use explicit versioned payloads.
- Validate retired threshold settings using the normal configuration path and update user documentation. No new global “background everything” setting is added.
- Do not add extension/MCP public capability APIs until an actual implemented integration needs them.

For MSG-01 migration, index already retained BG version-1 message receipts and recipient events. Do not backfill ordinary foreground results or manufacture reliable IDs for legacy uncorrelated notifications. Existing applied notifications remain applied; enabling an inbox must not redeliver them. Retrieval permission and artifact retention are checked using the existing root/session/child ownership evidence.

## 17. Implementation sequence and acceptance gates

The work can use several reviewable PRs, but **BG-01 through BG-04 together form the first independently shippable Bash feature**. Do not replace the old UX before its new acknowledgment and completion paths work end to end.

Recommended delivery graph (arrows are prerequisites, not permission to combine all work in one PR):

```text
BG-01 → BG-02 → BG-03 → BG-04  = independently shippable Bash
                         │
             after BG-04 ├──→ MSG-01  = result inbox for shipped integrations
                         ├──→ BG-05   = child backgrounding/handoffs
                         ├──→ BG-06   = code mode (independent integration)
                         └──→ BG-07   = MCP (later, independent integration)

MSG-01 + BG-05 → MSG-02 → MSG-03
```

BG-06 and BG-07 need the common BG-01–04 foundation, not the messaging tools. MSG-01 needs no child-agent integration to expose Bash results. The recommended user-facing sequence is Bash, child backgrounding, inbox, parent–child notices/steering, then questions/replies. BG-08 remains optional.

### BG-01 — Common owner lifecycle and verified persistence

**Implement:** identity/readiness representation, detached disposition fields, typed owner request, canonical acknowledgment/terminal receipt, SC-01–10 retained completion envelope and body ownership, prepared local actions, and replay. No generic public messaging API.

**Gate:** deterministic owner tests show final-first versus background-first behavior, duplicate request/result handling, interrupted local finalization recovery, and no ACK in terminal result storage. No tool capability is enabled yet; preserve current behavior until the full Bash replacement is ready.

### BG-02 — Caller resolution, admission, and completion mailbox

**Implement:** dedicated collector acknowledgment operation; detached reservation accounting; early late-result routing; accepted-final semantics for `CompleteDeferredToolCallHandler`; typed mailbox references over retained messages; recipient dedupe and non-destructive application; cancellation/wake fencing; bounded full-body/summary delivery and custom-role provenance.

**Gate:** parent can continue after its original batch resolves; occupied execution capacity remains occupied; no stale-result drop, second original-call result, cleared human wait, or cancelled-run restart. Provider-history conversion tests pass.

### BG-03 — Bash adapter and obsolete-path removal

**Implement:** retained Bash supervision, readiness and stable native reference, typed termination metadata, `bg_status` stable selector, removal of threshold question/config/prompt text, single completion producer, evidence-preserving cleanup.

**Gate:** actual managed process starts once, runs before and after detachment, obeys its original timeout, can be inspected/stopped, and produces one automatic completion. Test both main-owned and child-owned Bash.

### BG-04 — TUI request, picker, inventory, and full Bash release

**Implement:** contextual Ctrl+B, exact ownership/picker requests, accepted/rejected status, background inventory/control, canonical replay, distinct completion display, and updated documentation.

**Gate:** end-to-end TUI/runtime scenario proves acknowledgment before native completion without any model status polling. Cursor-left, question overlays, child view, single-worker capacity, cancel, close/restart, and `/history` behavior all have evidence. Run the repository's required full gate.

### BG-05 — Child-agent tool integration

**Implement:** subagent/fork/resume readiness and wait detachment; whole-batch result handoff; terminal child forwarding; outstanding-native-work status; unchanged deadlines/recovery/memory release.

**Gate:** parallel batch finishes once; original batch lifecycle remains active after parent ACK; timeout and parent cancellation still work; child terminal release is preserved; `agent_retrieve` yields the usual handoff.

### BG-06 — Code-mode integration

**Implement:** retained top-level host eligibility, invocation-aware nested cancellation, sequential reservation, structured result/diagnostic preservation and inspection artifact.

**Gate:** background during a blocked nested call, continue parent, then finish nested/script work and deliver the original structured value/stdout/stderr/errors. Do not enable the capability with only a script-sleep test.

### BG-07 — MCP integration, separately scheduled

**Implement:** pending-request ownership and later structured-result preservation using the same acknowledgment/receipt/mailbox path.

**Gate:** no resend/reconnect on detach, original cooperative timeout/cancel policy remains, remote outcome uncertainty is accurate, and single-consumer serialization is neither broken nor falsely advertised as solved.

### BG-08 — Optional Bash worker-release optimization, not required

Only pursue this if retained-worker capacity is measured to be a problem. It must move all Bash supervision responsibilities to the existing native owner before releasing the worker, without changing the common acknowledgment/completion protocol. Do not combine it with the first release by default.

## 18. Deterministic verification specification

Use the lowest layer that can prove each contract. Barriers, pipes, explicit readiness markers, fake clocks, and fault injection establish ordering; elapsed sleeps and “usually races this way” tests do not. Process fixtures must own and clean up their complete resource tree.

The repository testing skill requires focused `castor test --filter=…` checks and `castor check` for these runtime/TUI/Messenger/LLM-visible changes. Read `tests/AGENTS.md` and the testing skill in the implementation worktree before adding/running tests. Full integration checks listed here are requirements, not checks executed for this design. [S30]

### 18.1 Core lifecycle and delivery

| Test | Arrange / action | Required assertion |
|---|---|---|
| C01 — One execution | Hold a managed invocation behind a post-start barrier; request background twice | Launch counter remains one; same native record/identity persists |
| C02 — Parent continuation | Release caller through ACK while native barrier remains held | Next model step is dispatched only after sibling obligations resolve; native execution is still active |
| C03 — Final wins | Admit true result, then background request | One final original tool result, no background completion |
| C04 — ACK wins | Commit background, then admit true result | One ACK for original call and one separate completion |
| C05 — Duplicate receipt | Redeliver identical true result before/after mailbox apply and artifact cleanup | One accepted receipt and one model delivery; no artifact read required for known duplicate |
| C06 — Conflicting receipt | Send a different terminal payload under the same completion identity | Invariant failure/diagnostic; original accepted outcome remains unchanged |
| C07 — Old-step result | Advance to another turn before true result arrives | Detached result is accepted by original identity, not dropped as stale or added to current batch |
| C08 — No result-cache ACK | Detach and inspect executor store/processor calls | ACK not cached as terminal output; terminal processors run once on true outcome |
| C09 — Post-append failure | Fail local finalization after canonical background append | Existing prepared-transition recovery restores the exact captured ACK/correlation/reservation |
| C10 — Before-append failure | Fail canonical append | No accepted ACK, no native restart, foreground ownership remains |
| C11 — Send-loss boundary | Fail known effect/result send after finalization | Explicit repair can redispatch known identity; no hidden invocation replay |
| C12 — Busy delivery | Complete during LLM streaming and during unresolved tools | Current request/history remains unchanged; notification appears at safe boundary |
| C13 — Human wait | Complete while another question is pending | Same request ID and continuation survive; no automatic answer or clearing |
| C14 — Compaction | Complete during compaction | No injection into discarded input snapshot; one notification in post-compaction continuation |
| C15 — Idle coalescing | Several completions reach an idle root together | Stable application order, one successor, bounded combined content |
| C16 — Cancel fence | Queue completion, cancel, then start a new explicit user turn | Old completion does not auto-restart or clear cancellation; retained evidence remains inspectable |
| C17 — Terminal child | Finish child model before its detached native work | Child stays terminal; receipt forwards to root with origin; full child graph is released |
| C18 — Forward retry | Redeliver child-to-root completion | One root delivery; no direct cross-run log write or child resume |
| C19 — History branch | Navigate/discard origin branch before completion | No implicit forward-tail discard or reassignment to a new branch |
| C20 — Forged marker | Tool returns JSON claiming background/cancel/destination metadata | It remains ordinary untrusted output; no owner-control operation occurs |

### 18.1a Retained-message contract and phased compatibility

| Test | Required assertion |
|---|---|
| C21 — Mailbox consumption | Deliver/apply completion and remove consumed mailbox entry; the same message ID, summary and result body remain retrievable by the internal reader |
| C22 — Compaction retention | Compact away provider-bound completion text; retained evidence and canonical locator survive without retaining full child state |
| C23 — Immutable handoff | Resume a child after completion; the first completion's artifact reference still resolves its original generation, not the later handoff |
| C24 — Small versus large | A small result is pushed in full when it fits; a large result has actionable status/summary and a sealed body reference, never an ID-only ping |
| C25 — Cleanup hold | Native/temp cleanup cannot remove the last promised local body while its root-session message evidence is retained |
| C26 — Inbox absent | BG-01–04 pass end to end with no `messages` registration or schema; automatic text mentions only available inspection tools |
| C27 — Inbox upgrade | Build the later inbox projection from BG records; IDs and applied status do not change; no notification replay, body duplication, or tool rerun |
| C28 — Missing/foreign body | Missing/corrupt bodies yield bounded typed inspection failures; a forged ID/reference cannot bypass scope or trigger execution |

### 18.2 Capacity, safety, and UI

| Test | Required assertion |
|---|---|
| Q01 — Single worker | ACK and parent LLM continuation work; second same-pool tool remains queued until capacity frees; no extra worker is spawned |
| Q02 — Parallel capacity | Detachment does not bypass configured admission by repeatedly opening new batches |
| Q03 — Sequential reservation | A retained sequential invocation cannot be overlapped by a later same-run call contrary to its exclusion policy |
| Q04 — Release reschedules | Real completion releases the detached reservation and admits the current waiting batch, even when the origin batch was deleted |
| Q05 — Unsupported background tool | Non-backgroundability does not change that tool's ordinary parallel/sequential registration |
| Q06 — Approval | Queued/unapproved call cannot be backgrounded; no execution happens without approval |
| Q07 — Sticky cancel | Detached work sees cancellation even after current RunStatus changes or its original turn is gone |
| Q08 — Scoped control | Wrong root/run/native record is rejected; PID reuse cannot redirect invocation-ID inspection/stop |
| U01 — Immediate key | A started eligible Bash can be selected without waiting for the old threshold |
| U02 — Multiple calls | Picker targets exactly one call; stale selection is rejected, not retargeted |
| U03 — Child view | Child Ctrl+B never silently detaches the parent call, and main Ctrl+B never steals an unrelated child call |
| U04 — Editor fallback | Ctrl+B still moves cursor left with no eligible candidate |
| U05 — Modal precedence | Question/approval/completion UI retains input handling and does not lose its request |
| U06 — Render/replay | Live and rebuilt transcripts show one ACK state and one completion card with source metadata |
| U07 — Prompt history | Runtime completion is not listed as a human prompt or final assistant answer |
| U08 — Request race | View switch/duplicate key before server response does not change target or produce another acknowledgment |

### 18.3 Native execution contracts

| Test | Required assertion |
|---|---|
| B01 — Same process | A fixture emits identity before and after detach; PID/process record/native launch count stay unchanged |
| B02 — Original timeout | Advance a controllable deadline around background acceptance; no reset/removal; native stop and timeout evidence survive |
| B03 — Output bounds | Large/binary-invalid output follows existing representation/capping rules; bounded completion refers to preserved evidence |
| B04 — Stop | Stable invocation selector stops only the owned process group and yields one true completion |
| B05 — Cleanup failure | Fail result preservation during shutdown; evidence is not deleted and then reported delivered |
| B06 — Legacy record | Old accepted Bash row is inspectable/reconcilable without inventing a tool-call identity |
| A01 — Whole child batch | Detach a multi-child invocation; all required child outcomes contribute to one ordinary aggregate handoff |
| A02 — Native deadline | Detach, then trigger existing child deadline; interruption/cancellation still reaches unfinished children |
| A03 — Parent cancel | Cancel after parent ACK; outstanding descendant work is cancelled even if a child model already finished |
| A04 — Resume | Existing child run is continued once; backgrounding the resume call does not issue a second resume |
| A05 — Memory | Terminal child owner context is released; late no-op/forwarding does not retain completed graphs |
| K01 — Blocked nested tool | Parent receives ACK while code-mode host remains inside nested tool; release barrier, receive real nested return and final structured result |
| K02 — Diagnostics | Structured return, stdout, stderr, script failure, nested error, and timeout are preserved in bounded presentation/recoverable artifact |
| K03 — Cleanup | Host/socket/script remain live after ACK and are cleaned once after terminal execution |
| K04 — Nested eligibility | Nested Bash/MCP/code-mode calls never independently receive Ctrl+B acknowledgments |
| M01 — No MCP replay | One remote request before/after detach, no reconnect or resend |
| M02 — MCP uncertainty | Local cancellation/disconnect never falsely asserts remote termination; original deadline metadata stays unchanged |
| M03 — MCP result | Structured content and resource references survive bounded presentation |

Use fake/provider adapter conversion tests for all current provider families affected by history shaping. Use live model/TUI checks only where a protocol or input contract cannot be established below; follow the repository's required integration gate rather than claiming mocks prove a live run.

## 19. Requirement coverage and completion criteria

| Brief requirement | Specification coverage |
|---|---|
| Immediate user-controlled Bash; same process | §§7–8, 11.1; C01, B01, U01 |
| Common main/child invocation ownership | §§4, 6–9, 13; C17–18, Q08, U03 |
| ACK is not true completion | §§5–8; C03–08, A02, K03 |
| Automatic completion without routine model polling | §9 and existing Messenger/native result flow; C02, C12–16 |
| Keep native supervision/handoffs | §11.1–11.2; B02–05, A01–05 |
| Whole code-mode host and structured output | §11.3; K01–04 |
| MCP-compatible lifecycle without concurrency promises | §11.4; M01–03 |
| Separate background capability and parallel eligibility | §§2, 10; Q01–05 |
| Permissions/HITL/output limits/provenance | §§9, 12; C13, C20, Q06, B03, U05 |
| Existing framework/persistence; no competing journal | §§3.5, 6.4, 8.4, 14; C09–11 |
| Lifetime/cancel/restart limits; no ambiguous replay | §§4.1, 9.3, 12, 14; C16, Q07, B05 |
| Resolve the eleven open product decisions | D01–D11 in §4 |
| Retained completion compatible with the separate inbox | §7.5 SC-01–10, §14.4, C21–28 |
| Independent releases; no sibling/team messaging | §1.1–1.2, §17; companion MSG-01–03 |

A Bash release is complete only when BG-01–BG-04 pass their gates together, old threshold behavior/configuration/guidance is removed, the runtime can prove one coherent ACK/final outcome in both race orders, and the capacity/lifetime limitations are documented.

The main remaining implementation validation risks are **provider conversion/provenance**, **the existing batch store's exact deletion/admission implementation**, **prepared-action codec coverage**, and **resource-preserving shutdown ordering**. They have explicit required gates above; they are not reasons to silently weaken the contract or ship a Bash-only shortcut.

## 20. Alternatives rejected

| Alternative | Reason not selected |
|---|---|
| Ctrl+B launches a second “background version” | Violates invocation identity and duplicates side effects |
| Every tool runs in a new process | Not required by the brief; changes isolation/supervision/capacity far beyond this feature |
| Return early from code-mode host | Unwinds IPC/cleanup and cannot preserve synchronous nested calls |
| Let a host-loop polling flag decide every ACK | Cannot release the caller while the host is blocked in a nested tool or SDK |
| Reuse ordinary final result for ACK, then send final later | Existing batch duplicate/stale rules conflict; confuses execution end and caller resolution |
| Mark child deferred correlation complete on detach | Risks disabling deadlines, interruption, recovery, and eventual handoff |
| Reuse Bash `append_message` delivery unchanged | Human provenance, owner-acceptance, cancellation, and human-wait semantics are wrong for common completion |
| Add a new result queue/outbox/recovery journal | Duplicates existing Messenger/mailbox/prepared-transition ownership |
| Make `bg_status` a universal result tool as part of Bash | Not a prerequisite; MSG-01 retrieves retained evidence without forcing all native owners into a process model |
| Use the consumed mailbox as the only message store | Loses evidence after delivery/compaction and prevents repeatable inbox reads |
| Wait for the model to read/acknowledge before settling execution | Couples native lifecycle/capacity to nondeterministic model behavior |
| Implement sibling channels/teams alongside backgrounding | Outside the agreed parent–child scope and independent release boundaries |
| Promise background jobs survive controller loss | Existing native hosts/clients and cleanup do not support that guarantee |
| Free a logical/physical slot solely on ACK | Permits hidden oversubscription or violates sequential/resource ownership |

## 21. Sources and inspection limits

### Requirements source

**[R0]** User attachment: `Pasted text(1).txt`, titled “Backgroundable tool execution: requirements and decision brief.” Its organization and decisions are the requirements baseline. SHA-256: `211e73baa329cc4fd7b74953e855a4181357b038928ffa372b355682fe0c4f71`.

**[R1]** Subsequent user decisions in this conversation: separate tool-backgrounding and parent–child inbox/messaging specifications; implement in pieces; permit parent ↔ child communication, not sibling/team messaging. The companion's specific schemas, limits, and nonblocking-question policy are proposed implementation decisions, not additional claims about the original brief.

**[R2]** Supplied first specification: `hatfield-tools-backgrounding-spec.md`, SHA-256 `68d9a2b7a49c92b40ccd41e37a73b9075ad4c88073e1c6fa10719265785dc449`. Revision 2 edits that baseline; no newer repository checkout or verification was performed for this revision. The observations and pinned source links below are inherited from the original specification's recorded inspection.

The original source inspection focused on the execution worker, terminal result handling, deferred lifecycle, prepared transitions, mailbox/advancement, Bash process notification/cleanup, code-mode host, MCP invocation boundary, and relevant TUI/provenance paths. It was a static source read through the connected GitHub repository. No local checkout, test execution, performance measurement, or repository modification is claimed. Ancillary implementation files in §15 must be inspected in the actual worktree before editing.

Every repository source below is pinned to the stated commit. Where general architecture prose and implementation differ, §3 explicitly identifies the discrepancy; this document does not silently treat old prose as current code.

- **S01 — [`src/CodingAgent/Tool/BashTool.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Tool/BashTool.php)**: Managed launch, immutable record identity, foreground loop, threshold question, child-question restriction, and early background return.
- **S02 — [`src/CodingAgent/Config/BashToolConfig.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Config/BashToolConfig.php)**: Current timeout, threshold, loop, and log-tail settings.
- **S03 — [`src/AgentCore/Application/Handler/ExecuteToolCallWorker.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Handler/ExecuteToolCallWorker.php)**: Execution request conversion, deferred registration, launch-input validation, redelivery fast path, and final-result send.
- **S04 — [`src/AgentCore/Application/Pipeline/ToolCallResultHandler.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Pipeline/ToolCallResultHandler.php)**: Stale-step guard, cancellation, human suspension, batch collection, actual execution-end and ordered tool-message publication.
- **S05 — [`src/AgentCore/Application/Handler/ToolBatchCollector.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Handler/ToolBatchCollector.php)**: Prepared registration/collection, inFlight removal, duplicate checks, human waiting, and current dispatch admission.
- **S06 — [`src/AgentCore/Application/Handler/ToolExecutor.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Handler/ToolExecutor.php)**: Result reuse, exception normalization, terminal processors, deferred/suspension treatment, and cancellation arbitration.
- **S07 — [`src/CodingAgent/Entity/DeferredToolCompletion.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Entity/DeferredToolCompletion.php)**: Existing durable correlation schema and unique constraints. Companion: [`src/AgentCore/Contract/Tool/DeferredToolCompletionRepositoryInterface.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Contract/Tool/DeferredToolCompletionRepositoryInterface.php).
- **S08 — [`src/AgentCore/Application/Handler/CompleteDeferredToolCallHandler.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Handler/CompleteDeferredToolCallHandler.php)**: Current send-then-mark-completed behavior and batch deletion.
- **S09 — [`src/CodingAgent/Runtime/Controller/BackgroundProcessCompletionPoller.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Runtime/Controller/BackgroundProcessCompletionPoller.php)**: Native observation, scoped child discovery, PID aggregation, synthetic append_message, and notification marker.
- **S10 — [`src/AgentCore/Application/Pipeline/AdvanceRunHandler.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Pipeline/AdvanceRunHandler.php)**: Current-operation/idempotency guards, unresolved-tool barrier, mailbox application, terminal reactivation, human-wait clearing, and compaction.
- **S11 — [`src/AgentCore/Application/Pipeline/CommandMailboxPolicy.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Pipeline/CommandMailboxPolicy.php)**: FIFO boundary snapshot, ordinary user follow-up application, and command-applied events.
- **S12 — [`src/AgentCore/Application/Pipeline/RunCommit.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Pipeline/RunCommit.php)**: Prepared transition append, verified actions, finalization, and finished-child context release. Companion: [`src/AgentCore/Application/Pipeline/TransitionFinalizer.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Pipeline/TransitionFinalizer.php).
- **S13 — [`src/AgentCore/Application/Pipeline/LocalMetadataCoordinator.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Pipeline/LocalMetadataCoordinator.php)**: Verified local action validation and combined batch/mailbox application transaction.
- **S14 — [`src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Completion/DeferredSubagentBatchCompletionDispatcher.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Completion/DeferredSubagentBatchCompletionDispatcher.php)**: Native child handoff publication, registration dependency, input cleanup, and terminal-enqueued marker.
- **S15 — [`src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Interruption/DeferredSubagentBatchInterruptionService.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Interruption/DeferredSubagentBatchInterruptionService.php)**: Deadline rescheduling, first-wins interruption, registration/completed guards, and unfinished-child cancellation.
- **S16 — [`architecture/processes-and-queues.md`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/architecture/processes-and-queues.md)**: Controller ownership, current transports/pool counts, shared child execution, deadline routing, and shutdown/recovery boundaries.
- **S17 — [`src/AgentCore/Application/Pipeline/RunMessageProcessor.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Pipeline/RunMessageProcessor.php)**: Per-run owner locking, canonical readiness, history-tail handling, and release after terminal-child no-op.
- **S18 — [`src/CodingAgent/Tool/CodeMode/CodeModeHostBridge.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Tool/CodeMode/CodeModeHostBridge.php)**: Retained host/script/IPC, synchronous nested toolbox calls, budget propagation, result collection, and cleanup.
- **S19 — [`src/CodingAgent/Tool/CodeModeTool.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Tool/CodeModeTool.php)**: Plain structured value/diagnostics boundary, sequential registration, and documented script limits.
- **S20 — [`src/CodingAgent/Mcp/Tool/McpToolInvoker.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Mcp/Tool/McpToolInvoker.php)**: Cancellation/budget propagation, client failure treatment, and raw-result-to-string mapping boundary.
- **S21 — [`src/AgentCore/Domain/Message/AgentMessage.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Domain/Message/AgentMessage.php)**: Canonical message metadata and existing custom-role representation.
- **S22 — [`src/AgentCore/Infrastructure/SymfonyAi/AgentMessageConverter.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Infrastructure/SymfonyAi/AgentMessageConverter.php)**: Provider conversion, custom-role user-message compatibility, metadata propagation, and tool-image ordering.
- **S23 — [`src/AgentCore/Infrastructure/SymfonyAi/RunCancellationToken.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Infrastructure/SymfonyAi/RunCancellationToken.php)**: Current status-derived cancellation semantics.
- **S24 — [`src/Tui/Listener/PreviewExpansionInputListener.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/Tui/Listener/PreviewExpansionInputListener.php)**: Existing InputEvent interception, propagation control, and visible-view pattern.
- **S25 — [`src/Tui/Listener/AppHotkeyRegistrar.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/Tui/Listener/AppHotkeyRegistrar.php)**: Application hotkey documentation registration distinct from input enforcement.
- **S26 — [`src/Tui/Listener/CancelListener.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/Tui/Listener/CancelListener.php)**: Existing question/overlay precedence and visible child cancellation targeting.
- **S27 — [`docs/tool-execution.md`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/docs/tool-execution.md)**: Execution routing, output caps, result reuse limits, and current documented MCP cancellation constraints.
- **S28 — [`architecture/tools-and-mcp.md`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/architecture/tools-and-mcp.md)**: Tool architecture/reference paths; parts of the MCP prose must be reconciled against the inspected invoker and current execution documentation.
- **S29 — [`src/CodingAgent/Runtime/Controller/BackgroundProcessControllerSessionLifecycleListener.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Runtime/Controller/BackgroundProcessControllerSessionLifecycleListener.php)**: Startup/shutdown stop and exact sidecar/record cleanup for accepted native processes.
- **S30 — [`.agents/skills/testing/SKILL.md`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/.agents/skills/testing/SKILL.md)**: Castor validation requirements, deterministic readiness and contention proof, and full runtime gate.
- **S31 — [`src/AgentCore/Domain/Notification/ModelNotificationDTO.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Domain/Notification/ModelNotificationDTO.php)**: Existing notification source/kind/severity/delivery/text/metadata fields and deterministic content identity. Companion: [`src/AgentCore/Domain/Notification/ModelNotificationCodec.php`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Domain/Notification/ModelNotificationCodec.php).
- **S32 — [`AGENTS.md`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/AGENTS.md)**: Repository layering, small-change, source-of-truth, and implementation/testing workflow requirements.
- **S33 — [`architecture/extensions-and-agents.md`](https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/architecture/extensions-and-agents.md)**: Subagent/fork/resume shared lifecycle, artifacts, child scope, live-view ownership, and extension-job separation.

### Reference links

[R0]: #requirements-source
[R1]: #requirements-source
[R2]: #requirements-source
[S01]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Tool/BashTool.php
[S02]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Config/BashToolConfig.php
[S03]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Handler/ExecuteToolCallWorker.php
[S04]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Pipeline/ToolCallResultHandler.php
[S05]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Handler/ToolBatchCollector.php
[S06]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Handler/ToolExecutor.php
[S07]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Entity/DeferredToolCompletion.php
[S08]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Handler/CompleteDeferredToolCallHandler.php
[S09]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Runtime/Controller/BackgroundProcessCompletionPoller.php
[S10]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Pipeline/AdvanceRunHandler.php
[S11]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Pipeline/CommandMailboxPolicy.php
[S12]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Pipeline/RunCommit.php
[S13]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Pipeline/LocalMetadataCoordinator.php
[S14]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Completion/DeferredSubagentBatchCompletionDispatcher.php
[S15]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Interruption/DeferredSubagentBatchInterruptionService.php
[S16]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/architecture/processes-and-queues.md
[S17]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Application/Pipeline/RunMessageProcessor.php
[S18]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Tool/CodeMode/CodeModeHostBridge.php
[S19]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Tool/CodeModeTool.php
[S20]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Mcp/Tool/McpToolInvoker.php
[S21]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Domain/Message/AgentMessage.php
[S22]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Infrastructure/SymfonyAi/AgentMessageConverter.php
[S23]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Infrastructure/SymfonyAi/RunCancellationToken.php
[S24]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/Tui/Listener/PreviewExpansionInputListener.php
[S25]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/Tui/Listener/AppHotkeyRegistrar.php
[S26]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/Tui/Listener/CancelListener.php
[S27]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/docs/tool-execution.md
[S28]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/architecture/tools-and-mcp.md
[S29]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/CodingAgent/Runtime/Controller/BackgroundProcessControllerSessionLifecycleListener.php
[S30]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/.agents/skills/testing/SKILL.md
[S31]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/src/AgentCore/Domain/Notification/ModelNotificationDTO.php
[S32]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/AGENTS.md
[S33]: https://github.com/ineersa/agent-core/blob/f1240029bc0f249fd065edcd1e2ef2a408a35c3b/architecture/extensions-and-agents.md
