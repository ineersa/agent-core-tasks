# Hatfield: parent–child messaging and result inbox

**Technical design and implementation specification**  
**Revision:** 1 — companion to tool-backgrounding revision 2.  
**Date:** 2026-10-15  
**Source baseline:** `ineersa/agent-core@f1240029bc0f249fd065edcd1e2ef2a408a35c3b`, inherited from the attached backgrounding specification.  
**Status:** Proposed design; no implementation, new source checkout, performance result, or test execution is claimed.  
**Companion and shared contract:** [Tool backgrounding, revision 2](hatfield-tools-backgrounding-spec-v2.md), especially §7.5 SC-01–SC-10 and §9.  
**Delivery order:** MSG-01 result inbox; MSG-02 parent–child notices and parent steering; MSG-03 correlated, nonblocking questions/replies. No sibling/team messaging.

### Navigation

[Scope and phases](#2-scope-non-goals-and-release-boundaries) · [Tool contract](#7-model-facing-tool-contract) · [Authorization](#8-authorization-and-activity-fencing) · [Delivery policy](#10-automatic-delivery-provenance-and-wake-policy) · [Questions/replies](#12-questions-replies-and-deadlock-avoidance) · [File-level changes](#16-file-level-implementation-map) · [Release gates](#18-implementation-sequence-and-acceptance-gates) · [Tests](#19-deterministic-verification-specification)

## 1. Decision summary

Build a **model-visible read surface over retained runtime messages**, then add communication between a parent and its direct child. Reuse the backgrounding delivery contract, the existing run-control owner, canonical events, Messenger, and the existing command mailbox.

The inbox is not the runtime command queue. Reading a result does not complete a job. A message saying “finished” does not mark a child complete. A message saying “approved” does not answer a human approval. Sending to a child does not launch or resume it.

Keep three separate surfaces:

| Surface | Responsibility |
|---|---|
| **Messages / result inbox** | What happened, what information another authorized endpoint sent, and where retained evidence can be read |
| **Active-work inventory and control** | What is running, its native capabilities and ownership, whether cancellation can be requested |
| **Artifacts and logs** | Full native handoffs, structured result bodies, and bounded/paged log inspection |

This specification implements the first surface and references the other two. It does not replace all artifacts with a message table or all execution types with a Bash process model.

**Small messages are pushed in full when they fit. Large results push useful status/summary and can be pulled by ID.** The model must not need routine polling to discover completion or incoming communication.

### 1.1 Requirements and design status

The source materials are the supplied requirements brief, the attached backgrounding specification, and the subsequent conversation. The explicit user scope is **parent ↔ child**, including parent steering, child notifications/questions to parent, and information in the reverse direction. General sibling communication, broadcasts and full teams are excluded. The user requested separate revised specifications and incremental implementation. [R0], [R1], [BG]

Specific schemas, bounds, release identifiers, and refusal policies below are proposed normative implementation choices. They are not assertions that these APIs already exist or were specified verbatim in the requirements brief. Repository observations are inherited from the earlier source read; implementers must inspect the current worktree and local `AGENTS.md` before modifying the identified seams.

### 1.2 Reading order and contract ownership

Read BG §§1, 4, 6–9, 10 and 14 first. **BG §7.5 is the normative retained-message/delivery contract.** This specification adds producer-specific authorization, inbox queries, parent–child activity fencing, and questions/replies. It does not redefine BG terminal acceptance, background acknowledgement, cancellation, worker reservations, or automatic completion wake behavior.

When both specifications touch a source file, land the BG contract first and extend it once. Do not create independent `BackgroundMailbox` and `AgentMailbox` queues, duplicate output cap processors, or separately competing `AdvanceRun` producers.

## 2. Scope, non-goals and release boundaries

### 2.1 In scope

| Phase | Model-visible behavior | Prerequisites |
|---|---|---|
| **MSG-01** | List and read retained runtime messages/results by stable ID; preserve automatic completion presentation | BG-01–04. Bash alone is enough. Native child results become available after BG-05 |
| **MSG-02** | Parent sends notices or task steering to an active direct child; child sends notices to its parent | MSG-01 + BG-05; existing tool access and child authorization |
| **MSG-03** | Either side asks a correlated question and the other replies, without pausing a model/tool execution | MSG-02; detached relationship for child→parent questions; question expiry and activity fences |

A single eligible runtime can ship each phase without enabling later actions. A later integration such as code mode or MCP contributes results through BG's adapter and envelope; it does not need another inbox implementation.

### 2.2 Explicitly out of scope

- Sibling-to-sibling messages, broadcasts, team membership, general channel discovery, cross-session conversations, or shared mutable task-board state.
- Automatic child creation, resurrection or resume on receipt. Existing explicit `subagent`, `fork` and `agent_resume` remain the launch/resume operations.
- Model-controlled background launch/detachment, process-per-tool isolation, a new execution scheduler, guaranteed remote cancellation, or unlimited worker capacity.
- A blocking `ask_and_wait`, `await_message`, or arbitrary nested yield protocol. MSG-03 questions are nonblocking. No `wait: true` placeholder is exposed.
- Turning every ordinary foreground tool result into a separately notified inbox entry.
- Replacing `bg_status`, `agent_retrieve`, or all live logs. A future `jobs` facade can delegate to existing owners but is not a dependency or deliverable of MSG-01–03.
- Automatic conversational acknowledgments, generated “thanks” responses, or LLM-generated notification summaries.
- A standalone authoritative message database, a new outbox/recovery journal, or a new periodic message-polling service.

The model may communicate about work; only explicit authorized runtime operations can control that work.

## 3. Relevant baseline and required changes

The earlier specification records these implementation seams:

| Inherited observation | Consequence for this design |
|---|---|
| Run transitions commit through `RunMessageProcessor` / `RunCommit` and verified local metadata actions | Message acceptance, recipient receipt/application, and question state use the same owner transition path |
| The old Bash completion poller sends synthetic `append_message` | BG replaces it; MSG must not revive it as a second completion producer |
| Ordinary mailbox advancement can have user-continuation semantics, including human-wait changes | Internal messages need the typed safe-boundary policy from BG, not an ordinary user command |
| `AgentMessage` permits custom roles; the converter maps those to prefixed provider user messages | Provenance must be in provider-bound text; there is no claim of a provider-native notification authority |
| Child tools use durable deferred correlation, supervision and artifacts | Child handoffs stay native; inbox references them instead of generating another result format |
| Terminal child context is released; resumed children can reuse a run identity | Serve retained messages without keeping child graphs; fence authored messages by launch/resume activity, not run ID alone |
| Code mode uses synchronous nested tool calls and rejects deferred/human continuations | Owner-bound message mutations must not be launched through a nested bridge that cannot settle them |

These are inherited baseline observations, not a fresh audit of repository HEAD. See BG §3 and its pinned sources S03–S17, S18–S23 and S31–S33. Any newly proposed class names below are implementation targets, not discovered APIs. [BG]

## 4. Product decisions

| ID | Decision |
|---|---|
| **MSG-D01 — Topology** | Directed communication only across an authenticated direct parent–child edge. The current prohibition on nested agent launch is unchanged. Runtime forwarding of a child-owned completion to the recorded root remains BG's special case, not a general ancestor messaging capability |
| **MSG-D02 — Tools** | One host-owned `messages` tool, with only phase-implemented actions visible. MSG-01: `list`, `read`; MSG-02 adds `send`; MSG-03 adds `ask`, `reply`. No `ack`, destructive pop, edit, delete, arbitrary publish, or raw runtime command action |
| **MSG-D03 — Retention** | Retained messages outlive mailbox consumption and prompt compaction. Reads are repeatable; referenced local bodies remain for the owning root-session evidence lifetime under BG SC-07 |
| **MSG-D04 — Push/pull** | Push useful bounded content automatically. Push small bodies in full when possible; pull large evidence on demand. No obligatory read call for every small notice |
| **MSG-D05 — Authored authority** | Parent `steer` is a task-owner instruction within an existing child's task, not a human/system permission change. Child→parent is notice, question or reply, never a privileged steering/control command |
| **MSG-D06 — Terminal destinations** | Do not resume a terminal child. Reject a new send addressed to an already terminal child; a destination that becomes terminal after source acceptance retains/parks the message. A normally idle root may wake only under the authorized active detached-child policy in §10 |
| **MSG-D07 — Activity identity** | Bind child-directed instructions and all questions/replies to the exact current parent launch/resume invocation, plus wake/history fences. An old message must not become an instruction for a newly resumed task |
| **MSG-D08 — Read semantics** | `applied` means bounded text committed to model context. `read` means an ordinary retrieval call returned evidence. Neither means understood, acted on, job finished, or question answered |
| **MSG-D09 — Questions** | MSG-03 asks return immediately with a question ID. There is no blocking wait. Reject child→parent questions while the parent still waits on that child's foreground invocation; suggest backgrounding or normal handoff instead |
| **MSG-D10 — Receipts** | Source acceptance is distinguishable from recipient acceptance and model-context application. A successful `send`/`ask`/`reply` result promises only the owner's persisted submission, not that another model saw it |
| **MSG-D11 — Compatibility** | Keep `bg_status`, `agent_retrieve`, existing TUI inventory and native logs. An inbox must not erase their permissions, controls or specialized result semantics |
| **MSG-D12 — Delivery ownership** | All message kinds share BG's byte budget, mailbox policy and successor admission. There is no second wake loop or automatic message-response agent |

These choices are binding for the proposed implementation. Broader blocking-conversation or job-facade work would require a separately reviewed amendment rather than implicit scope growth.

## 5. Invariants

**MI01 — One accepted identity.** Retrying an identical submission or delivery produces the same message and no new automatic presentation. Conflicting payloads for a stable ID are rejected.

**MI02 — Retained evidence is immutable.** Edits, later child resumes and mailbox cleanup cannot rewrite the body or provenance of an already accepted message. A correction is a new message.

**MI03 — Only owners write canonical state.** Workers report requests/outcomes. The source owner accepts a message; the destination owner accepts/applies delivery. Neither appends directly to another run's log.

**MI04 — No implicit execution control.** Message text cannot grant approval, cancel work, mark native status, bypass tool access, adjust deadlines, free reservations, resume an agent, or transfer a worktree.

**MI05 — Authenticated direct edge.** A guessed ID, supplied sender field, artifact title, or shared controller is not authorization. Ordinary authored communication is parent ↔ direct child only.

**MI06 — Non-destructive consumption.** Listing, reading, automatic application, parking and forwarding do not remove the retained message. Full body reads are not required for native lifecycle settlement.

**MI07 — Safe model boundaries.** Delivery does not mutate an in-flight provider request, break an unresolved tool-call sequence, replace a compaction snapshot, or clear human input.

**MI08 — Sticky fences.** Cancellation, history-tail changes and child activity changes prevent stale automatic application even after run status becomes running again.

**MI09 — Bounded work.** List/read/page/automatic delivery sizes are bounded before loading bodies. No full event-log replay per message read or permanent retention of completed child graphs.

**MI10 — Explicit question semantics.** A question is data with correlation and expiry, not a suspended tool call. A reply resolves only that question's state; it is not a tool approval or deferred child completion.

**MI11 — No invented durability.** Source commit is not a distributed transaction with recipient receipt or model invocation. Post-finalization transport failure follows the existing explicit repair boundary, not a hidden new resend service.

**MI12 — One completion producer.** A child/worker handoff reaches the inbox through BG terminal acceptance. Agents are not required to send an additional “finished” message to complete their task.

## 6. Data model and canonical records

### 6.1 Reuse the shared envelope

Use BG §7.5's version-1 retained envelope. MSG-01 reads the existing `tool_completion` variant unchanged. Later phases extend the discriminator with producer-specific fields:

| Kind | Phase | Producer / allowed direction | Additional semantics |
|---|---|---|---|
| `tool_completion` | BG / MSG-01 reader | Runtime, using original invocation ownership | `message_id = completion_id`; status and final body are native evidence |
| `agent_notice` | MSG-02 | Parent→child or child→parent | Bounded agent-authored information, no mandatory response |
| `parent_steering` | MSG-02 | Parent→active direct child only | Task guidance; retains parent provenance, recipient activity and original launch permission scope |
| `question` | MSG-03 | Parent↔child subject to §12's liveness checks | `question_id = message_id`, requester, designated responder, activity identity and expiry |
| `reply` | MSG-03 | Designated responder→requester | References one accepted question; no implicit authority outside that request |

Do not trust a model-supplied `kind`. The tool action, authenticated origin, and validated direction determine it. Runtime status fields are accepted only from BG's native completion path. A child's plain text stating “status: succeeded” remains plain text.

Agent-authored variants use closed typed fields, not arbitrary metadata maps:

```text
origin: source run, original tool-call tuple, authenticated parent/child relation
message_id: deterministic, domain-separated from runtime completion IDs
kind: assigned by runtime from action and authorized direction
presentation: subject, bounded summary, optional severity assigned/validated by host
body: bounded validated text (inline) or an explicitly owned immutable reference
parent_child_activity_id: exact native launch/resume invocation being addressed
source/destination wake and history fences
optional in_reply_to: another visible message in the same direct edge/activity
question fields, only for question/reply variants
```

MSG-02 text is capped and stored inline unless an existing authorized artifact is attached. The runtime does not summarize agent text with another model; use an explicit bounded subject/preview. Full body and author are preserved.

### 6.2 Identity and retry semantics

For `send`, `ask` and `reply`, derive message identity from a versioned namespace plus the originating `(run, turn, step, tool_call_id)` and action. Do not include the Messenger delivery-attempt number or a random retry ID. The tool invocation's canonical input fingerprint distinguishes an identical retry from an incompatible reuse.

A new model tool call with a new tool-call ID is a new intentional submission, even if its body happens to match an earlier message. Do not deduplicate based only on text or subject. Otherwise legitimate repeated task guidance could vanish.

Bind question and reply IDs to the same accepted activity. A reply has its own message ID; the question ID remains the first ask's ID. There are no in-place reply edits.

### 6.3 Source and recipient events

Event/type names are proposed; follow existing enum/codec conventions:

| Record | Owner | Purpose |
|---|---|---|
| Accepted BG terminal receipt with retained envelope | Original execution run | Canonical runtime result/message; do not duplicate it as an additional MSG source event |
| `runtime_message_recorded` | Authored-message source run | Accept immutable message, intended target and captured delivery action |
| `runtime_message_received` | Destination run | Validate source reference, record recipient acceptance/order and pending/parked disposition |
| `runtime_message_applied` | Destination run | Persist exact bounded presentation, model-history inclusion, and consumed mailbox instruction |
| `runtime_message_delivery_changed` | Relevant recipient owner | Park/reject/expire delivery with a structured reason; do not mutate source text |
| `parent_child_question_closed` | Requester run | First accepted answer, expiry, cancellation or activity closure, with the relevant answer ID/reason |

BG's existing accepted/applied completion events can carry the shared shapes rather than being renamed merely to match this table. The important contract is one canonical producer record, one recipient identity and one application record, not a required migration of historical event names.

`ModelNotificationDTO.id` can carry the retained message ID for UI correlation, but its pre-existing content-ID semantics alone are not receiver deduplication. Use the canonical `(recipient_run_id, message_id)` receipt/application contract.

### 6.4 Read index: a projection, not another authority

MSG-01 adds a narrow `RunMessageIndex` read model in the existing application database or the repository's established indexed session projection facilities. Its canonical source is BG's retained receipts and recipient events. It contains bounded headers and locators only, not full transcripts or result bodies.

Minimum conceptual columns:

```text
recipient_run_id + message_id        # unique inbox identity
source_run_id + source_event_ref
recipient_received_seq              # stable recipient order/cursor
kind / subject / bounded summary
origin_invocation_id, when present
parent_child_activity_id, after MSG-02
body locator + digest/bytes/type (not body payload)
delivery disposition + applied_event_ref
question_id / question_state, after MSG-03
```

A source/sent lookup is indexed by `(source_run_id, source_event_seq, message_id)`. This is a **sent-message view**, not a transport outbox. Cross-run receiver state is queried through authorized narrow metadata, never by writing the receiver's state back into the sender's log from a foreign worker.

Apply projection deltas through verified local actions with the accepting owner transition. A missing index is rebuildable using the existing canonical streaming/index path; do not scan every root/child transcript for each list/read call. Record a versioned rebuild watermark and use bounded pagination. A read failure cannot turn an unverified SQL row into authoritative provenance.

### 6.5 Body and artifact adapters

Reuse specialized owners:

| Message body | Reader / retained evidence |
|---|---|
| Bash completion | Sealed terminal result and managed log generation retained under BG cleanup rules |
| Child handoff | Existing artifact/handoff, pinned to that launch/resume generation |
| Code mode | Existing BG structured-value and diagnostic artifact; not flattened to one Bash log |
| MCP | BG-preserved structured content and resource references; remote resource availability remains explicit |
| Agent-authored text | Canonically accepted bounded body |

Where current native artifacts are mutable or temporary, BG must first preserve an immutable version. The reader cannot recover the correct old handoff by silently returning the latest child artifact. If a remote link expires, report availability separately from the completed operation's original status.

Readers accept internal reference types, not model-supplied filesystem paths or URLs. A model may select an attachment ID from a returned message manifest; the server resolves and authorizes it before reading bytes.

## 7. Model-facing tool contract

### 7.1 Registration, phase exposure and execution mode

Proposed tool name: **`messages`**. Use the project's existing provider/DTO/toolbox registration and validation pattern. Only implemented actions appear in its active schema. Input schemas are closed: reject unknown fields and action-inapplicable combinations rather than ignoring control-looking keys.

The tool requires a real authorized run context. It is not callable as an unscoped admin transport, extension-job side channel, or arbitrary MCP producer. Respect current run tool allowlists/exclusions; do not give a restricted child the tool by bypassing its tool policy. Document the required child-definition/toolset updates when MSG-02 is enabled.

Reads can use ordinary parallel execution. Message mutations serialize canonical acceptance at the source owner and do not require a new global tool-exclusive lane. Use `ToolExecutionMode::Parallel` for the single registration because these actions have no filesystem mutation semantics; it must still honor existing admission limits and same-run sequential reservations. Parent steering in the same assistant batch has no guaranteed send order beyond source-owner acceptance; use subsequent calls/turns when instructions depend on an earlier one.

Initially the tool is **top-level only**, including top-level calls from child runs. Add it to code mode's unsupported nested operations before invocation. This avoids a nested send launching a deferred owner operation that the bridge cannot settle. Do not reject only after side effects have been submitted. Supporting nested read-only use later is optional, not a requirement to implement nested communication now.

### 7.2 `list` — bounded discovery and recovery

Illustrative invocation:

```json
{
  "action": "list",
  "view": "inbox",
  "limit": 20
}
```

| Field | Contract |
|---|---|
| `action` | `list` |
| `view` | `inbox` by default; `sent` becomes available in MSG-02 |
| `cursor` | Optional opaque continuation bound to authenticated run/view/filter/snapshot |
| `limit` | Default 20, integer 1–50; byte limits can return fewer entries |
| `kind` | Optional implemented kind filter; no arbitrary SQL or run filter |

The default inbox is the current run's accepted recipient records, including explicitly parked records with their reason. It does not enumerate all controller/child inboxes. A parent's own sent view is available after MSG-02; parenthood alone is not blanket authority to inspect a different child's inbox.

Response includes bounded entries, receipt/application state, available body/attachment summary, and `next_cursor`. List cursors use recipient acceptance order for inbox and source acceptance order for sent records. A first-page query fixes a high-water sequence so subsequent pages do not duplicate/skip old entries as new messages arrive. Reissuing a query without a cursor obtains a new snapshot.

No destructive pop or read-state mutation occurs. “Available” and “applied to context” are meaningful labels; do not claim that an agent comprehended something. Explicit recovery after compaction or inspection of parked evidence can use `list`; normal completion discovery remains push-driven.

### 7.3 `read` — non-destructive, bounded retrieval

Illustrative invocation:

```json
{
  "action": "read",
  "ids": ["m42", "m43"],
  "max_bytes": 8192
}
```

| Field | Contract |
|---|---|
| `action` | `read` |
| `ids` | 1–8 message IDs; IDs are opaque strings, not assumed numeric or sortable |
| `part` | `body` by default; `attachment` requires exactly one ID and an `attachment_id` returned by that message |
| `attachment_id` | Required only for `part=attachment`; never a raw path/URL |
| `cursor` | Optional body/manifest pagination cursor; accepted only with exactly one ID |
| `max_bytes` | Integer 1024–16384; default 8192 bytes, including returned wrapper/metadata. Lower configured output limits still apply; an insufficient effective envelope budget returns a bounded budget error rather than an endless empty page |

Validate access before resolving a body. Return unknown and unauthorized IDs using one `not_found_or_not_authorized` category to avoid cross-session enumeration. Author or designated-recipient access is allowed only inside the established root scope and relevant artifact policy.

Each entry has message ID, authenticated provenance, original outcome where applicable, current delivery state, bounded selected content, attachment/part descriptors, and an explicit continuation/truncation indicator. Return results in requested ID order. If the aggregate budget is exhausted, report remaining IDs as `not_returned_budget` rather than silently claiming they were read; the model can make a focused follow-up call.

Cursors bind to immutable body generation/digest, selected part and authorized run. For text, use valid UTF-8 byte windows. For structured results, page native parts or a bounded manifest before materializing a large value. When a JSON artifact must be returned as text chunks, label it `serialized_json_fragment`; do not pretend a truncated fragment is a complete decoded object. Code-mode values and diagnostics remain distinguishable. Images/binary/remote resources retain existing capability and access rules; the inbox does not embed unbounded bytes or automatically fetch arbitrary links.

A read result is an ordinary tool result for the **read call's** own tool-call ID. It never reuses the original background tool-call ID, changes an old acknowledgement, or re-enqueues the message notification. Replaying that tool call can reuse its ordinary result, but no `mark_read` callback controls delivery or native completion.

### 7.4 `send` — notices or parent steering, MSG-02

Parent-to-child example:

```json
{
  "action": "send",
  "to": "child:<artifact-id>",
  "intent": "steer",
  "subject": "Keep this change API-only",
  "text": "Do not modify the database schema. Keep the existing persistence interface and report any blocker."
}
```

Child-to-parent example:

```json
{
  "action": "send",
  "to": "parent",
  "intent": "notify",
  "subject": "Found a related cancellation edge case",
  "text": "The current test fixture also exercises late completion after cancellation. I will keep the fix within the assigned files."
}
```

| Field | Contract |
|---|---|
| `to` | Exactly `parent` or `child:<existing artifact-id>`; server resolves the direct edge/current activity |
| `intent` | `notify` by default; `steer` only parent→active direct child |
| `subject` | Required nonblank, at most 256 UTF-8 bytes |
| `text` | Required nonblank, at most 8 KiB UTF-8 bytes |
| `in_reply_to` | Optional visible message ID in the same two-endpoint activity; a conversational reference, not a correlated answer |
| `attachment_ids` | Optional, at most 8 existing owned artifact IDs authorized for both endpoints; reject arbitrary paths and missing authority |

The model cannot set sender, root, target run generation, runtime kind, job outcome, wake override, raw command, tool approval, permissions, or a destination outside the direct edge. Address resolution freezes the activity in the accepted envelope; it is not rerun later against whichever resume happens to be current.

The tool completes after source-owner acceptance, not after recipient reading:

```json
{
  "submission": "accepted",
  "message_id": "m42",
  "recipient": "child:<artifact-id>",
  "recipient_acceptance": "not_confirmed",
  "delivery": "safe_boundary",
  "resumed_recipient": false
}
```

This is a submission receipt, not a promise that the target is still active or has seen it. Receiver rejection/parking is visible through receipt state/TUI and later explicit inspection. Failure notifications must be bounded and must not create an acknowledgment-response loop.

### 7.5 `ask` — nonblocking correlated question, MSG-03

```json
{
  "action": "ask",
  "to": "parent",
  "subject": "Expected behavior for an unknown ID",
  "text": "Should this endpoint return null or raise NotFound? I can continue the unrelated validation tests meanwhile.",
  "expires_in_seconds": 300
}
```

`to`, `subject`, `text` and attachment validation follow `send`. Additional field `expires_in_seconds` is an integer 1–3600, default 300. The owner sets absolute expiry from its acceptance clock and caps it by any applicable earlier native task deadline. These are proposed initial limits, not a change to existing execution budgets.

The accepted response includes `question_id = message_id`, captured responder/activity, absolute expiry, and `waiting: false`. No run enters `WaitingHuman`; no tool call remains suspended for the responder; no budget is extended. The requester may continue independent work and receive the answer later. Absence of an answer is not permission to guess a required decision.

Child→parent `ask` is refused while the parent still waits on that child's foreground invocation (§12). `send(intent=notify)` is still allowed in that situation because it makes no response/liveness promise.

### 7.6 `reply` — explicit correlated response, MSG-03

```json
{
  "action": "reply",
  "question_id": "q17",
  "text": "Raise NotFound, using the existing exception type. Do not change the public error format."
}
```

There is no arbitrary `to`: derive the requester from the validated question. The caller must be its designated responder in the same activity. Text is nonblank and bounded as above; optional attachment IDs require the same access checks.

A successful source submission returns a new reply message ID plus `question_acceptance: not_confirmed`. Only the requester's owner can mark the question answered. Source-side validation rejects a known expired/closed/foreign question immediately; receiver acceptance rechecks all conditions because they may change in transit.

Only an explicit `reply` can answer a question. A normal notice with `in_reply_to`, or text saying “yes,” does not silently resolve it. A reply does not answer a native human approval or call `CompleteDeferredToolCall` on the questioner's background task.

## 8. Authorization and activity fencing

### 8.1 Direction matrix

| Authenticated caller | Operation | Allowed target |
|---|---|---|
| Root/parent run | Read/list | Its own accepted inbox and, after MSG-02, its own sent records |
| Child run | Read/list | Its own accepted inbox/sent records only |
| Parent run | Notify | Its active direct child |
| Parent run | Steer | Its active direct child, within existing task/tool/approval scope |
| Child run | Notify | Its recorded parent |
| Either endpoint | Ask/reply | The other endpoint, subject to current activity and §12 liveness/expiry rules |
| Child run | `steer` parent | Rejected |
| Any run | Send to sibling/unrelated/root by guessed ID | Rejected |
| Runtime BG producer | Forward completed child-owned native work | BG's recorded root destination only, preserving origin and message ID |

The root's TUI/operator inventory can retain BG's authorized controller-tree visibility. That does not grant every child model equivalent lookup or send authority. Existing tool and artifact authorization remains the final gate even when a message reference is visible.

### 8.2 Capture identities once; verify again on receipt

Use native parent/child metadata and the launch/resume invocation as the **activity ID**. Do not introduce a second team or conversation-membership registry.

At submission acceptance, capture:

```text
source run + source model tool invocation
recorded parent run + direct child run/artifact
parent launch/resume invocation identifying this child activity
source and destination wake generations
current applicable history/branch evidence
original parent authorization generation for this activity
```

Check parent authorization from the activity's launch/resume evidence, not merely the parent's current generation at send time. A child from a cancelled task must not regain permission by observing that the user has since started a new root turn. A new explicit authorized resume can establish a new activity.

At destination receipt and again before context application, verify that the activity and wake/history fences still apply. Incompatible messages are parked/rejected as stale; they do not become instructions for the next task. Original evidence remains inspectable by authorized endpoints.

### 8.3 What steering may and may not do

Steering updates task guidance at a permitted model boundary. It does not mutate original tool-call arguments already executing, change the child's model/tool allowlist, add filesystem access, alter the sandbox, waive approvals, reset a timeout, or release another writer's resource claim.

An active child can receive new parent instructions before its next model request, using the same legitimate continuation semantics as existing steering where compatible. Do **not** route it through an ordinary synthetic human `append_message` or a generic restart branch that clears `WaitingHuman`. The parent-task-owner label must survive into provider text. Current human/system instructions and actual authorization still outrank it.

MSG-02 does not require cancellation of the current provider/tool operation in order to steer. Mid-flight interruption or forced rewind is a separate operation and must remain explicitly authorized.

### 8.4 History and inactive endpoints

Selecting an earlier history point is not permission for a runtime message to discard the forward tail. Use BG's history gate. Tail-discarded activity messages remain out-of-branch evidence and are not reassigned to a new branch merely because the same run is active.

New parent sends to an already terminal child return `recipient_inactive`; no implicit `agent_resume` occurs. If termination races an accepted source submission, the destination may accept it as parked evidence without rehydrating a persistent active agent graph. Return/replay explicit `recipient_terminal` status.

Old parked messages are not automatically included when the user later resumes a child. The new explicit task can refer to selected prior evidence through authorized reads, but it does not re-arm old steering/wake permissions.

## 9. Submission and receipt implementation

### 9.1 Read-only actions

MSG-01 `list` and `read` use the ordinary validated toolbox path and the authenticated ambient run context. The read model and native body readers are read-only. Retrieval does not write delivery flags from a tool worker or require a run-control round trip merely to label a message read.

Reads return bounded ordinary tool data, preserving native result/provenance metadata. Tool result normalization, malformed UTF-8/depth handling, output limits and current error semantics still apply. Raw body checks are performed before data can be hidden by presentation conversion.

### 9.2 Owner-accepted authored submissions

Use the existing **deferred tool completion mechanism for the short source-owner acceptance handshake**, not a blocking worker RPC or a new queue. This deferral ends when the source owner accepts/rejects the submission; it never waits for another agent to respond.

Proposed flow:

```text
messages.send / ask / reply on ordinary execution worker
    → usual access/schema/hooks/approval gates
    → typed native DeferredToolCompletionOutcome for this exact submission
    → existing worker registers durable correlation and releases its PHP stack
    → messaging-specific registration listener schedules AcceptParentChildMessage
    → source run_control validates and commits source acceptance or rejection
    → normal deferred terminal result resolves the messages tool call
    → after source commit, deliver typed source reference to destination run_control
    → recipient owner records receipt + mailbox obligation or refusal/parking
```

Implementation requirements:

1. The handler returns the supported typed coordination object, not model-controlled JSON flags. Derive a deterministic submission/deferred identity from the original messages tool call. It remains non-backgroundable and is not a long-running job in `/background`.
2. The existing durable deferred registration must exist before the messaging registration listener schedules owner acceptance. Retried registration re-arms the **same** owner submission. Dispatch nothing to the recipient before canonical source acceptance.
3. Filter the listener by exact host-owned tool/action and correlation. It must not enter child-batch lifecycle handlers or synthesize a child launch. Child registration listeners must ignore messaging submissions safely.
4. `AcceptParentChildMessage` is a proposed typed owner message. It carries the exact correlation/reference, not arbitrary authority. Under the source lock, verify the current authorized original call, direct edge, immutable body/ref bounds, activity, and any question state visible at acceptance.
5. Source acceptance commits `runtime_message_recorded` and captures the recipient delivery effect plus the small source-tool submission receipt. Reuse `RunCommit`, prepared actions and existing finalization. No new journal/table promises delivery exactly once.
6. Complete the source `messages` call through the normal deferred result factory and canonical terminal-result admission path. BG's accepted-final semantics apply to this short correlation; it is not marked finished merely on transport send.
7. A redelivery first looks up the accepted source decision by original invocation/message ID and reuses it. It cannot append another authored message or re-run a side effect to recover a lost submission receipt.
8. If source submission was rejected before acceptance, complete the tool with a bounded typed error and no recipient effect. If source acceptance succeeded but a later result-send fails, repair redrives the recorded receipt/delivery identity; it must not report that the accepted message never existed.

This mechanism extends an existing supported coordination value rather than inventing `MessageSubmissionOutcome` alongside unrelated competing executors. The concrete listener/correlation routing must be verified against the current worktree before implementation.

### 9.3 Why the worker must not wait for the receiver

A synchronous tool-worker wait for another model would consume capacity and can deadlock a parent/child using the same pool. The submission tool returns once its **own owner** has durably accepted it. Its local deferred wait is only the ordinary owner handshake; there is no blocking receive loop, permanent worker thread, or nested tool promise.

Actual physical pool limits still apply before the send/read tool is admitted. With one tool worker occupied by retained Bash/code mode, a `messages` call can wait for that worker. The inbox is not an undeclared priority execution lane. An active same-run sequential reservation can likewise defer a parent model's message tool, even when a physical worker is free. MSG-02's live interaction gate must use actual child registration/admission behavior and prove that the tested configuration has available capacity; document configurations in which model steering is delayed rather than bypassing reservations. Automatic delivery and TUI attention use run control and remain independent of that pool; native inspection/stop remains available through BG's TUI route.

### 9.4 Receiver acceptance

Under only the destination's owner lock:

1. Verify the canonical source record/reference and its stable semantic fingerprint. Resolve body refs only after scope checks; no eager large-body load is needed to accept the header.
2. Deduplicate `(destination_run_id, message_id)` before body retrieval; reject conflicting identities.
3. Check the direct edge or BG-authorized runtime forwarding rule, destination activity, wake generation, history position and resource authorization.
4. Prepare a received record in canonical recipient order and a pending mailbox reference, or an explicit parked/rejected disposition.
5. For a reply, validate and transition the requester-owned question under this same lock (§12); retain the reply as stale/parked evidence if it cannot answer the question.
6. Commit/finalize using the existing owner transition. Release a terminal child context after administrative receipt/no-op as required by BG's memory-lifetime contract.
7. Request normal mailbox reconsideration/`AdvanceRun` only if the shared policy permits. No receiver directly calls a model from inside message acceptance.

Do not hold source and destination locks simultaneously. The intended recipient in a source record is not proof of receipt; the received event is. TUI “received” must not mean “model saw the body.”

### 9.5 Ordering and fairness

Mailbox application follows recipient acceptance order at each boundary, within the shared byte budget. No cross-run wall-clock ordering is promised. System/human cancellation and existing control-command priority remain authoritative; authored messages cannot starve explicit user commands.

A burst already queued at a boundary may be represented by one bounded presentation batch and one successor model step. Preserve each message ID and provenance separately within the batch. Do not synthesize agent replies to drain an inbox. Old eligible entries must make progress before an unbounded stream of new notices.

## 10. Automatic delivery, provenance and wake policy

### 10.1 One policy with BG

All kinds use BG §9 and SC-08. MSG does not add a poller that concurrently appends user text or starts another `AdvanceRun`. Capture exact presentation at application time and append it only at a valid boundary.

| Destination condition | Runtime completion | Notice / steering / question / reply |
|---|---|---|
| LLM request in flight | Pending | Pending; never mutate request or interrupt stream |
| Foreground tool batch unresolved | Pending | Pending; keep original tool/result sequence valid |
| Safe boundary | Apply bounded presentation | Apply bounded presentation if direct-edge/activity fences still match |
| Waiting for human/approval | Pending with TUI attention | Pending with TUI attention; never answer/replace the human request |
| Compaction active | Pending to post-compaction boundary | Same; no insertion into the discarded input snapshot |
| Normally idle/completed root | BG's matching-generation completion wake | May wake only for a message from its still-authorized active **detached** direct-child activity, with matching launch-time root generation/history; coalesce through the same successor gate |
| Cancelled/failed root | Park; no automatic wake | Park; no automatic wake |
| Terminal child | BG forwards to recorded root | Do not revive; reject new targeted sends or park in-flight receipts |
| Stale child activity/history/wake fence | BG's parked/forwarded rules | Park/reject; no automatic application to a new task |

The extra root wake permission for authored child messages is narrow: the runtime verifies an active supervised detached relationship that the parent already authorized. It is not permission for a completed unrelated child, arbitrary peer, or cancelled old task to restart a root. Root `send` to a child does not make that child detached.

A known inactive responder causes `ask` refusal rather than a promise of automatic revival. A request already accepted before inactivity can expire or be closed without model invocation. See §12 for additional question constraints.

### 10.2 Presentation examples

Large runtime result, reusing BG:

```text
[tool_completion] Message m42 — runtime-delivered external result, not a new human request.
Origin: reviewer child; invocation j17; outcome failed.
Summary: Two blocking cancellation findings are recorded in the retained handoff.
Additional evidence is available through messages.read or the existing artifact reader.
```

Parent task steering:

```text
[parent_steering] Message m43 — task guidance from this child's authenticated parent.
Activity: <exact launch/resume identity>.
Subject: Keep this change API-only
Body: "Do not modify the database schema. Keep the existing persistence interface."
This does not change tool permissions, approvals, execution budgets or higher-priority instructions.
```

Child notice:

```text
[agent_notice] Message m44 — information from your authorized child, not a human instruction.
Activity: <exact launch/resume identity>.
Subject: Found a related cancellation edge case
Body: "The test fixture also exercises late completion after cancellation."
```

Question/reply presentation additionally includes `question_id`, direction, expiry/current question state, and whether it has already been answered. IDs in these examples are illustrative, not a required numeric format. Bodies must be escaped/delimited so quoted text cannot replace provenance fields.

The inherited converter maps custom roles to prefixed provider user messages. The security contract is honest provenance plus actual runtime authorization, not a claim that a prefix prevents all prompt injection. Do not elevate bodies into system/developer messages or label child text as a new human request. [BG §9.2](hatfield-tools-backgrounding-spec-v2.md#92-preserve-provenance-using-existing-custom-messages)

### 10.3 Presentation versus message immutability

The retained envelope is immutable. The runtime computes a bounded application presentation using available reader capabilities, current safe question/delivery state, and the shared cap. It commits that exact text once. Applying a later reply does not retroactively alter the text of the original question in old provider history.

MSG-01's introduction changes future available tool guidance, not historical prefixes. No repeated regeneration of earlier summaries, timestamps, counters, or unread lists in the system prompt is required. After compaction, the inbox still serves the original retained bodies; compaction is not a new notification-delivery event.

### 10.4 Delivery and attention labels

Use precise states:

- **Accepted by sender owner:** persisted submission; receiver status may be unknown.
- **Received:** destination committed a receipt.
- **Applied to context:** bounded presentation is in canonical model-bound history.
- **Parked/rejected:** automatic application is disallowed; display a reason.

A body retrieval can be observed in the ordinary retrieval tool-result events if useful for diagnostics. It is not an operational “read acknowledgment” and should not become a mandatory extra write/transaction on every read. Do not maintain a state called “acted on” unless an explicit separate workflow operation actually proves it.

## 11. Parent steering and notices

### 11.1 Parent-to-child steering

The parent selects one existing child artifact handle. The host resolves it to a direct child with a current authorized activity. It freezes that identity in the accepted message so a delayed command cannot target a different resume.

The child sees the instruction at its next safe model boundary. Its current tool call may finish first. No in-flight work is restarted, its original arguments remain unchanged, and current human input stays pending. The steering is visible as parent guidance in the child's transcript, not as an operator-entered user prompt.

A parent that requires immediate termination must use an actual supported cancel operation, not send “stop” and assume native cancellation has happened. The receiving model may act on instructions within its permissions, but delivery state is not proof that it has done so.

### 11.2 Child-to-parent notices

An active child may notify its parent of a blocker, discovery, progress milestone or relevant fact. The message need not wait for final handoff, but the runtime's native task/result state does not change.

While the parent still waits for the child's foreground task, the notice can be shown in TUI and retained for the parent's next valid model boundary. Do not claim the parent model can immediately respond before that wait resolves. For truly interactive model exchanges, the relationship should already be backgrounded.

When the child finishes, BG's normal handoff producer remains authoritative. The child need not send an extra completion message. A prior notice is still a distinct useful communication; do not automatically suppress a later actual handoff by comparing similar text.

### 11.3 Parent-to-child notices and replies

A notice from the parent is informational rather than task steering. The runtime retains the distinct intent even though both come from the same authenticated parent. A parent reply to a child's question may contain a task decision, but cannot bypass higher-priority human constraints or runtime gates.

There is no generic agent broadcast. To communicate with two children the parent must submit two explicit authorized messages. Each has its own ID and delivery state; this is not a multicast API with an unbounded recipient set.

## 12. Questions, replies and deadlock avoidance

### 12.1 MSG-03 is nonblocking by design

`messages.ask` resolves its own tool call immediately after source acceptance. It does not wait on the reply. The requester can continue independent work. The recipient's response arrives as a separate `reply` message; the runtime relates it to the retained question.

Do not introduce a new waiting run status, leave the ask's ordinary tool-call ID unresolved, or synthesize a later second result for it. This preserves provider tool-call sequencing and avoids making native child completion depend on a parent model turn that cannot yet run.

If work cannot safely proceed without the answer, report that dependency. In the initial release the agent can use the existing human escalation/final handoff mechanisms as appropriate; it must not busy-poll, repeatedly ask, or pretend a missing reply is approval. A future blocking yield needs its own state machine and review.

### 12.2 Foreground parent/child cycle

Refuse child→parent `ask` when the existing native launch correlation says the parent is still waiting for the child through a foreground tool invocation:

```text
parent awaits foreground child result
child attempts to ask parent for a decision
    → reject parent_waiting_on_child
    → child may send a nonblocking notice or use its normal handoff/escalation
```

The check is based on actual caller disposition in BG, not on whether the parent has read a background acknowledgement or how the TUI labels the child. If the parent acknowledgement is already committed but siblings in its current batch are pending, the ask can be retained; the parent's eventual model boundary may still be delayed, and expiry continues.

For initial MSG-03, require a background-acknowledged supervised activity for child→parent questions. Parent→child questions also require an active authorized child and must not implicitly detach it. This is a conservative release rule, not a universal claim that all nonblocking questions inherently deadlock.

### 12.3 Requester-owned question state

The immutable question body stays in its retained message. A small canonical/question projection records:

```text
question_id / requester_run_id / designated_responder_run_id
parent_child_activity_id
accepted_at / expires_at
state: open | answered | expired | cancelled | closed_activity
accepted_answer_message_id, when answered
closure_event_reference / reason
```

Only the requester owner transitions this state. A reply is not an answer until that owner validates it and commits `open → answered`. The transition uses the existing verified metadata path. There is at most one accepted final answer per question; a follow-up question gets a new ID.

### 12.4 First accepted answer and races

| Race | Required outcome |
|---|---|
| Same reply delivered twice | One received/application record, one answered transition |
| Two distinct replies compete | First valid requester-owner acceptance wins; later reply is retained as `question_already_answered` evidence, not a replacement answer |
| Reply accepted before expiry closure | `answered`; late expiry is a no-op |
| Expiry condition already true when reply is accepted | `expired`; late reply is parked/marked late and cannot reopen it |
| Cancellation/history/activity change before reply acceptance | Close/park according to fence; no automatic restart or application to a new task |
| Responder/source accepts reply, requester rejects it later | Submission receipt stays truthful; delivery/question acceptance reports the rejection separately |
| Requester child completes before reply | Do not resume it; close its open question with `closed_activity` and retain any later reply as evidence |

Expiry is checked against the requester's owner clock on every relevant acceptance/application. An earlier sender timestamp is not proof of on-time arrival. Persist `expires_at` for restart/repair; use the existing clock abstraction/fakes for deterministic tests. A normal completed/idle root can still own an open question within a live authorized detached-child activity under §10; a terminal child cannot.

### 12.5 Expiry scheduling without another scheduler

Use existing Messenger delayed delivery to send a typed `ExpireParentChildQuestion(question_id)` to the requester owner. Before changing state, recheck the canonical question and current clock; duplicate/early/late commands are harmless or rescheduled with the same identity as appropriate.

This is a normal delayed owner command, not a new timer daemon. Acceptance and listing/reading compute effective expiry from the stored deadline even if a delayed transport delivery is late. A read does not become an unauthorized writer; owner reconciliation commits terminal question state at its next applicable entry or repair. Transport loss follows the existing explicit repair limitation.

Expiry closes a question only. It does not cancel the underlying child/job, answer a human request, extend execution budgets, trigger a fake successful response, or automatically resume a stopped model. UI can show expiration; model notification of it is subject to the same safe-boundary/wake rules and should not spawn a new conversation loop.

### 12.6 Blocking wait is a separately reviewed extension

No blocking API ships in MSG-01–03. A later proposal would have to define an explicit suspended continuation, waiter ownership, a capacity-free wait, wake/cancel/timeout transitions, native child-deadline interaction, and rejection of parent↔child wait cycles before admitting them. A boolean flag on `ask` is not sufficient.

This exclusion is intentional so the first hierarchical messaging release does not become a full conversational agent orchestration rewrite.

## 13. Bounds, retention and result fidelity

### 13.1 Internal budgets

BG SC-06 governs stored envelopes and automatic presentation. This spec adds only reader/submission bounds:

| Boundary | Proposed initial bound |
|---|---|
| Message subject | 256 UTF-8 bytes |
| Authored body | 8 KiB UTF-8 bytes; oversize input rejected before acceptance |
| Referenced attachments on authored submission | At most 8 explicitly authorized owned references; larger evidence stays in a single manifest/artifact |
| List page | At most 50 entries and 16 KiB rendered output including wrapper/cursor |
| Read batch | At most 8 IDs and 16 KiB rendered output including wrappers/cursors; default 8 KiB |
| Automatic message presentation | Shared BG 16 KiB per safe boundary, not 16 KiB per producer |
| Open questions | At most 8 per requester run/activity; reject excess without delaying normal notices/completions |
| Question lifetime | Default 300 seconds; 1–3600 requested seconds, capped by an earlier applicable native task deadline |

Honor smaller effective downstream caps and preserve space for error/continuation metadata. Invalid UTF-8, unsafe depth, opaque objects and malformed references follow existing result representation policy; do not silently substitute or stringify them into apparently valid evidence. Read adapters must be bounded before materialization, not only after a huge string was allocated.

Use existing run/model/tool budgets as the main limit on authored traffic. Pending messages consume no live model execution stack. Coalesce UI attention and model-bound delivery; do not automatically answer notices. If an actual resource limit is reached, surface a typed refusal, not silently drop accepted data.

### 13.2 Evidence retention and cleanup

The owning root session's retained evidence lifetime is the default for local message bodies. Mailbox consumption, a successful `read`, child context release, and prompt compaction do not shorten it. Existing session deletion removes its retained message scope and associated owned artifacts together.

An individual child artifact cleanup must honor retained result references or replace them with a sealed root-owned snapshot before deletion. Do not orphan a parent's message by deleting the only old child handoff when `agent_resume` overwrites a native file. Keep one authoritative immutable body, not one body copy per recipient/read.

Body integrity metadata is immutable. Availability changes are separate diagnostic/delivery evidence, not a mutation from “job succeeded” to “job never completed.” A missing body never authorizes rerunning the tool or automatically resuming a child to regenerate text.

### 13.3 Projection versus prompt memory

A model can forget a message after compaction; the runtime should not pretend otherwise. The retained inbox is an evidence-recovery surface. It does not mean all messages remain loaded in every prompt or that the model is required to retrieve every body.

Use static tool guidance describing recovery/list/read when relevant, not a continually rewritten global unread-message list in the system prefix. If a message was already applied, compaction does not mark it unapplied or create an automatic duplicate notification. Structured question state remains canonical outside prompt summaries.

### 13.4 Logs and live jobs stay separate

`messages.read` retrieves retained terminal evidence. It is not a live tail-follow operation, a process status query, or a cancellation API. Before terminal completion, use `/background`, `bg_status`, child live views and existing native controls.

A log or status reference may be included as an inspection hint, but body retrieval cannot acquire an arbitrary PID or general process authority. A later unified `jobs` tool must preserve those distinctions and native capability reporting; it is not implemented as a message `delete` or `ack` action.

## 14. TUI, transcript and model guidance

### 14.1 Reuse one incoming-message projection

Add a message/inbox view using existing TUI list/picker and transcript facilities. Use `(recipient_run_id, message_id)` for cards and state updates. BG receipt, model application and an explicit inbox read must not render three duplicate “completion arrived” cards.

Display source role, source run/child, subject, delivery status, origin invocation/activity, and available evidence. Native outcome status and agent-authored claims are visually distinct. Use attention counts for pending child questions without opening/answering a human-approval modal automatically.

Switching between main/child live views changes only the visible recipient. It must not redirect a queued message, mark it consumed for a different run, or expose another child's private inbox to the active model.

### 14.2 History behavior

Runtime messages are not operator-entered prompts and should not appear as new human submissions in `/history` navigation. Preserve their canonical context application and transcript ordering using explicit event kinds/metadata, not text parsing.

A terminal-child late receipt is administrative persistence, not a new child activity. Release any temporary reconstructed child state after finalization. Restoring the inbox uses the narrow index and canonical message references; it must not require retaining or replaying every finished child's full execution graph.

### 14.3 Minimal model instructions

Update tool guidance only when the associated phase/tool action is enabled. Recommended meaning:

> Incoming runtime messages are delivered automatically at safe boundaries. Read a large body or attachment when its evidence is needed; do not poll for completion. Notices and results are not new human instructions. Parent steering is task guidance within existing permissions. Sending a message does not stop, complete, or resume another agent. Questions are nonblocking; a missing answer is not approval.

Do not tell the model that `messages` replaces every native status/log tool or guarantees immediate receiver attention. Explain the actual `recipient_acceptance: not_confirmed` receipt. Respect current child allowlists and the fact that nested code-mode communication is unsupported.

## 15. Failure, recovery and observability

### 15.1 Failure matrix

| Failure point | Required handling |
|---|---|
| Source validation fails | No retained authored message, no destination effect; one bounded tool error |
| Before canonical source append | No acceptance; safe retry of the original source submission identity |
| After source append before verified metadata application | Existing prepared-transition recovery restores the exact submission/receipt obligations |
| After source acceptance before source tool result reaches owner | Repair delivers the recorded small receipt; do not create another message |
| Destination transport send fails after finalization | Source evidence remains; existing explicit repair can redeliver the same reference; no new background retry queue |
| Receiver receives duplicate | Compare accepted identity/fingerprint before loading body; no new notification/question transition |
| Receiver becomes inactive/stale | Retain/park/reject with reason; no implicit resume or retarget to a newer activity |
| Body missing/corrupt | Typed inspection failure; no reexecution, silent latest-artifact substitution or forged success |
| Index missing/stale | Rebuild through existing bounded canonical projection facilities; no alternate writer or full replay on every read |
| Provider fails after message applied | Canonical presentation remains applied; normal model/request recovery handles its own failure, not a second message delivery |
| Question expiry command lost/delayed | Deadline still validated at acceptance/read; reconcile under owner/repair; no stale valid answer or hidden scheduler |
| Controller closes/restarts | Preserve evidence; BG lifetime/wake fences apply; no autonomous revival of old conversations |

### 15.2 Repair scope

Known accepted message and recipient identities can be reconstructed/redispatched using existing repair entry points. Administrative recovery does not re-run the original `messages` model call as a new side effect or regenerate native results. A source submission whose acceptance cannot be established must not be presented as delivered.

Do not promise guaranteed automatic retransmission after the inherited post-finalization effect-loss boundary. In a healthy controller, delivery is automatic and requires no model polling. When infrastructure strands it, show the actionable repair condition rather than instructing the model to repeatedly list its inbox.

### 15.3 Observability

Log structured identifiers and state transitions, not full message bodies, credentials, command arguments or task prompts by default. Useful fields: message ID/kind, source/destination run, activity/invocation ID, accepted/received/applied state, failure reason, latency from committed receipt to application, and bounded body bytes returned.

Distinguish tool submission latency, receiver acceptance latency, and model-bound application latency. A receipt can be delayed by a legitimate human wait or occupied tool batch; do not mislabel that as a transport outage. Include counts for parked stale messages, duplicate deliveries, expired questions, body failures and index rebuilds without retaining full histories in memory.

## 16. File-level implementation map

Paths inherited from BG are existing or identified seams, not a claim of a fresh inspection. **New** names are proposed. Before editing, inspect current definitions, more-specific `AGENTS.md`, service wiring and serializer/action registries.

| Area | Existing source seam / proposed addition | Required work |
|---|---|---|
| Shared envelope and receipt | BG §7.5 internal DTOs, canonical completion record and recipient policy | Reuse unchanged for MSG-01; add closed authored variants only in MSG-02/03 |
| Inbox read model | **New** `RunMessageIndex` / narrow reader adapter in CodingAgent session/repository facilities | Bounded recipient/sent queries, canonical locators, unique recipient ID, incremental rebuild; no body blobs |
| Body readers | Existing Bash/child/session artifact readers; **new** small result-reference reader interface/adapters where needed | Verify ownership/generation/digest; preserve structured parts and native error behavior |
| Tool registration | **New** `src/CodingAgent/Tool/MessagesTool.php`, typed arguments/action validation; existing `ToolDefinitionDTO`/registry | Phase-specific `list/read/send/ask/reply`; real run context; no unscoped/cross-session/raw command API |
| Tool access | Existing per-run toolset resolver, child definition allowlists and exclusions | Expose only where allowed; document child configuration requirements; never override a denylist |
| Source submission | Existing `ExecuteToolCallWorker`, `DeferredToolCompletionOutcome`, deferred repository; **new** messaging registration listener and `AcceptParentChildMessage` owner handler | Short source-acceptance deferral, exact identity replay, normal submission tool result; no worker wait on another model |
| Deferred result routing | Existing `CompleteDeferredToolCallHandler`, result factory and owner admission | Preserve BG accepted-final semantics; do not route message submission as a child launch or background job |
| Canonical commit | Existing `RunMessageProcessor`, `RunCommit`, `TransitionFinalizer`, `LocalMetadataCoordinator` and action codecs | Message/recipient/question event and projection deltas through one verified path |
| Delivery | BG's mailbox policy and `AdvanceRunHandler`; **new** authored-message receiver adapter | One safe-boundary budget, retained references, generation/activity fences and coalesced successor |
| Relation checks | Existing run-relationship/native child lifecycle and artifact metadata seams identified in BG | Direct parent–child authorization and exact launch/resume activity; no team registry |
| Steering | Existing authorized steering/continuation seams and BG custom-message application | Preserve parent provenance and waits; no unconditional user follow-up/restart or in-flight mutation |
| Questions | **New** requester-owned question projection/close action and expiry owner command | Single answer, deadline checks, activity closure and no blocking wait status |
| Code mode | `src/CodingAgent/Tool/CodeMode/CodeModeUnsupportedTools.php`, bridge pre-invocation checks | Refuse nested `messages` before submission; do not discover unsupported deferral after send |
| Provenance / provider mapping | `AgentMessage`, `ModelNotificationDTO` / codec, `AgentMessageConverter`, current reasoning/compaction adapters | Explicit custom-role text and correct original-tool-call sequencing; no elevated human/system source |
| Child memory | Existing terminal-child release in `RunCommit` / `RunMessageProcessor` | Administrative delivery/replay must not leave child graphs resident |
| TUI | Existing runtime event translator, transcript projection, message lists/pickers, child live-view context | One card identity, precise delivery state, inbox navigation, attention without human-question mutation |
| Wiring / storage | `config/services.yaml`, `config/packages/messenger.yaml`, current DB migration and event/action serialization registries | Route owner messages to `run_control`; no new bus/transport/database requirement |
| Documentation | Existing tools/agents/backgrounding/human-input/session-history documentation and model guidelines | Explain retained versus pending, direction, nonblocking questions, capacity limits, unchanged native controls |

The spec does not mandate one PHP class per table row or a broad generic framework. Keep adapters small and colocated with their existing responsibility; use interfaces only at actual layer boundaries. No new public extension messaging API is part of this work.

## 17. Migration and compatibility

### 17.1 MSG-01 on a shipped BG installation

Index retained BG version-1 completion envelopes and recipient events. The same `message_id = completion_id`, original outcome and applied status must remain. Do not send another completion merely because the model can now read it by ID.

Do not backfill ordinary foreground tool results or invent authoritative identities for old synthetic uncorrelated Bash notifications. Legacy native inspection remains available; label records unsupported for inbox migration when identity cannot be proven. Inbox omission of uncorrelated legacy data is preferable to fabricated provenance.

Artifact retention from BG is a prerequisite. An inbox rollout must not promise bodies that the prior BG implementation already deleted after delivery. Run the upgrade/cleanup gate before enabling `messages`.

### 17.2 Adding authored actions

Require a clean controller/worker upgrade to a version understanding the new variants and deferred-submission listener. Expand the tool's schema and guidance only for deployed actions. Unknown persisted variants remain unsupported/parked evidence; they are not interpreted as arbitrary commands.

Existing child task allowlists/exclusions still govern exposure. Existing launch/resume identities must be sufficient to authenticate an active edge. Where a historical activity lacks the necessary generation evidence, refuse authored automatic-wake/steering privileges until a new explicit authorized activity establishes them; do not guess from the current run status.

### 17.3 What is not removed

Keep `bg_status`, `agent_retrieve`, `/background`, child live views, native stop controls, and existing human-input tools. `messages` is not a replacement for human escalation. Do not change command timestamps, role provenance or already applied prompt history just to present an inbox.

A later `jobs` facade or blocking question protocol requires explicit scope and compatibility decisions. Neither should delay the releases in §18.

## 18. Implementation sequence and acceptance gates

### Dependency graph

```text
BG-01 → BG-02 → BG-03 → BG-04 = complete Bash backgrounding release
                                  ├──→ MSG-01 = result inbox (Bash ready)
                                  ├──→ BG-05  = child backgrounding/handoffs
                                  ├──→ BG-06  = code mode, independent
                                  └──→ BG-07  = MCP, later and independent

MSG-01 + BG-05 → MSG-02 = parent–child notices and parent steering
MSG-02         → MSG-03 = nonblocking questions/replies
```

The work can be divided into multiple reviewable PRs within a phase. Do not expose an action until its end-to-end permissions, persistence, delivery and validation gates pass. This is one architecture with independent release milestones, not one mandatory combined rewrite.

### MSG-01 — Model-visible retained result inbox

**Implement:** narrow canonical-derived inbox index; scoped `messages.list/read`; bounded native body/attachment adapters; provenance and TUI integration; non-destructive retention; phase-specific tool registration and top-level restriction.

**Do not implement:** sending, steering, questions, sibling identity discovery, a universal jobs tool, or mandatory new runtime completion polling.

**Gate:** messages produced by BG without the inbox become readable with identical IDs and no re-notification. Small and large results are useful, bodies survive mailbox cleanup/compaction, foreign IDs are denied, missing bodies are explicit, and BG works with `messages` excluded from the toolset. A real Bash result is sufficient to ship; add child handoff coverage with BG-05.

### MSG-02 — Directed notices and parent steering

**Implement:** `messages.send`; direct-edge/activity authorization; short deferred owner submission; receiver receipts; existing safe-boundary delivery; narrow idle-root permission for live detached-child activity; parent-task-owner provenance; stale/terminal refusal; sent view.

**Do not implement:** reply waiting, broadcast/sibling communication, implicit resume, arbitrary control commands, or new execution capacity claims.

**Gate:** an active detached child receives parent steering and can notify its parent; both submissions resolve without waiting for the other model. A foreground-child notice is retained without breaking the parent's pending tool call. Cancel/resume/history races are fenced. Forged sender/target/control-shaped payloads have no authority. No unfinished submission or duplicate accepted message remains after deterministic registration/send/receipt fault tests.

### MSG-03 — Correlated nonblocking questions and replies

**Implement:** `messages.ask/reply`; immutable question/reply records; requester-owned open/answered/expired/cancelled/activity-closed state; deadline checks and existing delayed owner commands; deterministic first-answer and late-reply handling; foreground-cycle refusal.

**Do not implement:** `wait`, `await_message`, `WaitingParent`, auto-resume, or an unresolved original ask tool call.

**Gate:** requester continues after accepted ask; one valid reply resolves correlation separately. Two replies/expiry/cancel/activity races yield one coherent state. Foreground child asks are refused without side effects; human approvals remain untouched. A terminated child never restarts on its answer. All behavior is visible in retained inbox records without forcing routine polling.

## 19. Deterministic verification specification

Use fake clocks, explicit source/receiver barriers, controlled Messenger delivery, real owned native fixtures where relevant, and fault injection. Do not use sleep timing as proof of a race. Read the implementation worktree's testing skill and `tests/AGENTS.md`; apply the focused and full gates recorded in BG S30. No tests listed below were executed to prepare this document.

### 19.1 Inbox, retention and source fidelity — MSG-01

| ID | Required proof |
|---|---|
| IN01 | A BG-only terminal message can be read after installing MSG-01; ID, outcome and body generation remain unchanged |
| IN02 | Consuming the mailbox entry leaves repeatable header/body reads intact |
| IN03 | Compaction removes provider text but not retained body/index recovery; does not redeliver an applied message |
| IN04 | An 8-ID read stays within the total effective byte budget and explicitly identifies unreturned entries |
| IN05 | List pagination uses a fixed high-water snapshot; concurrent arrivals cause no duplicates or omissions within that snapshot |
| IN06 | A guessed sibling/unrelated/root-private message or attachment ID yields no data and no existence oracle |
| IN07 | A changed or missing local body produces a typed inspection error; no native invocation is dispatched |
| IN08 | A child resume that changes native artifacts cannot change a previously retained result body |
| IN09 | Structured code-mode value and diagnostics, when BG-06 exists, are separately readable; truncation is explicit and valid fragments are labelled |
| IN10 | Index deletion/rebuild uses bounded canonical traversal; repeated reads do not replay all sessions or retain child `RunState` graphs |
| IN11 | A small automatic completion includes full useful content when it fits; a large one includes status/diagnostic and an available reader reference |
| IN12 | Unknown version/kind is unsupported/parked rather than auto-executed as a control operation |
| IN13 | Inbox tool disabled/excluded leaves BG completion automatic and existing native inspection usable |
| IN14 | Native/temp cleanup preserves the last promised local evidence until the root message scope is explicitly deleted |
| IN15 | Cursor tampering, attachment path injection, invalid UTF-8/depth and oversize bodies are rejected without eager unbounded allocation |
| IN16 | Legacy uncorrelated notifications are not fabricated into modern message identities during migration |

### 19.2 Authored submissions, scope and steering — MSG-02

| ID | Required proof |
|---|---|
| PC01 | Parent→active direct-child notice and steering are accepted; child sees correct parent/activity at its next valid boundary |
| PC02 | Child→parent notice succeeds without waiting for parent model execution or body reading |
| PC03 | Sibling, unrelated-session, spoofed sender and child→parent privileged steering are refused before recipient effects |
| PC04 | Source failure before append creates no message; retry after append/local-finalization failure yields one accepted source record |
| PC05 | Deferred registration redelivery re-arms the same source acceptance; messaging correlation never enters child launch handlers |
| PC06 | Source accepted/receiver not confirmed is accurately reported; effect loss is recoverable by same-ID repair, not a new submission |
| PC07 | Identical recipient delivery is deduplicated before body lookup; conflicting body fingerprint for the ID is rejected |
| PC08 | A current provider request remains byte-for-byte unchanged when steering is received during streaming |
| PC09 | Parent notice arriving while the child's human approval is pending preserves the exact question and answer continuation |
| PC10 | Parent foreground wait remains unresolved by a child notice; no synthetic second tool result or implicit detachment occurs |
| PC11 | Child ending after source acceptance parks late steering; parent cannot implicitly resume a known terminal child via send |
| PC12 | Cancel followed by explicit new parent work cannot re-arm old child notice/steering wake authority |
| PC13 | Child resume produces a new activity; earlier queued steering never targets its new task |
| PC14 | Normal idle root wakes only for a current authorized detached-child message, never a completed/unrelated/stale child |
| PC15 | Concurrent BG completions and notices share one byte budget and one legal successor, not parallel model invocations |
| PC16 | Tool-shaped text saying approved/cancelled/finished has no approval, native lifecycle or cancellation effect |
| PC17 | Raw nested code-mode `messages` is rejected before a message submission/recipient effect exists |
| PC18 | Single occupied tool worker delays tool calls honestly; TUI attention/cancel and owner delivery remain responsive without extra spawned consumers |
| PC19 | Main/child view switch changes no message target, source, receipt state or authorization |
| PC20 | Live transcript and replay show one recipient card; runtime message does not appear as a human prompt in history navigation |

### 19.3 Questions, replies and liveness — MSG-03

| ID | Required proof |
|---|---|
| QR01 | Ask returns one ordinary tool result immediately after source acceptance with `waiting=false`; no model/run human-wait state is created |
| QR02 | Child→parent ask on foreground-pending child correlation is refused; a nonblocking notice remains possible |
| QR03 | The same ask on an already background-acknowledged activity succeeds even before the parent has read its acknowledgement |
| QR04 | Only the designated responder can reply; a normal notice with `in_reply_to` does not close the question |
| QR05 | Same reply redelivered twice gives one answer/application; a second distinct reply cannot overwrite the accepted answer |
| QR06 | Reply/expiry order is decided under the requester owner with the test clock; delayed expiry never permits a late answer to reopen |
| QR07 | Accepted source reply may be rejected as expired/stale by requester without falsely claiming the submission never existed |
| QR08 | Requester child ends before reply; it remains terminal, question closes and the late reply stays inspectable |
| QR09 | Parent/root can remain normally idle with an open valid detached-child question, then receive a valid reply through the shared wake gate |
| QR10 | Cancellation, tail discard or resumed activity closes/fences open questions; neither ask nor reply revives old work |
| QR11 | Expiry closes only the question, not the job or a human approval; no automatic repeated questions/acknowledgments are generated |
| QR12 | Lost/duplicate delayed expiry command respects canonical deadline on subsequent owner acceptance/repair; no new timer service exists |
| QR13 | More than 8 open questions is refused with no accepted excess record; normal runtime completion is not blocked by that limit |
| QR14 | A question and reply retain their original bodies after mailbox consumption and compaction; body reading is not a question state transition |

### 19.4 Provider and full-system gates

Run conversion/replay tests for every currently supported provider path affected by custom-role messages, including reasoning transitions and tool-call ordering. The initial acknowledgement and later result inbox/read must never create a second original tool result or unmatched assistant call.

Full integrated proof should cover parent backgrounding a real child invocation, sending task steering, child notification, correlated nonblocking question/reply, normal child terminal handoff and result retrieval after compaction. Use existing fixtures/test adapters for determinism and the repository's required real integration gate where needed. Mock-only success is not evidence that the TUI/runtime/provider sequence works live.

## 20. Definition of done and implementation safeguards

**MSG-01 is complete** when retained BG evidence is discoverable/readable by authorized recipients after mailbox cleanup and compaction, without changing native completion or forcing a retrieval call for every small result. Migration proves stable identities and no duplicate notifications.

**MSG-02 is complete** when directed parent–child notice/steering has owner-accepted source receipts, scoped recipient delivery, task/wake/history fencing and explicit terminal handling, with no worker waiting for the other model and no general team channel.

**MSG-03 is complete** when questions/replies have one requester-owned correlation outcome, expiry/cancel/activity handling, foreground-cycle refusal and immediate ask-tool completion. A blocking wait is not a hidden missing requirement; it is explicitly outside this release.

Main worktree validation risks are the precise deferred-registration listener routing, existing mailbox/history-selection behavior, immutable child artifact retention across resume, per-run tool policy exposure, and custom-role provider conversion. Inspect these before editing. If the current repository has moved since the pinned baseline, reconcile the implementation map in a small documented update; do not silently replace these contracts with guessed API behavior.

## 21. Requirement coverage and rejected alternatives

| User direction / requirement | Coverage |
|---|---|
| Model receives an ID and can retrieve a result | MSG-01 `list/read`, stable retained envelope, §§6–7 |
| Useful automatic pings, not routine status polling | §§10, 13–14; shared BG mailbox/presentation |
| Keep global awareness/control of running work | Native/TUI inventory retained; inbox deliberately not a job supervisor (§13.4) |
| Parent steering and child notifications/questions, not teams | §§2, 4, 8, 11–12; direct-edge matrix |
| Implement in pieces, not one large rework | §§2 and 18; BG remains independently shippable |
| Reuse artifacts and handoffs | §§6.5, 13.2, 17; BG result owner remains authoritative |
| No approval/cancel/status spoofing | MI04–05, §8.3 and §10 provenance |
| Preserve native deadlines, cancellation and capacity | BG remains owner; §§9.3, 12–13 add no exemptions |

| Alternative rejected | Reason |
|---|---|
| Expose the internal command mailbox to models | Leaks execution-control concepts and makes reading destructive/scheduling-dependent |
| Send only “new message ID” for everything | Adds unnecessary retrieval round trips and hides actionable failures |
| Mirror all handoffs/logs into a second message database | Duplicates payload and authority; native artifacts already own evidence |
| Use every incoming message as an ordinary user `follow_up` | Risks provenance loss, cleared human waits, stale-task restart and competing continuation semantics |
| Let `messages.send` wait synchronously for recipient model | Occupies workers and creates parent–child cycles |
| Let child ask suspend its foreground result automatically | Parent may be waiting on that exact result; needs an explicit separate continuation design |
| Make reading the message an execution/answer acknowledgment | Model-dependent reads cannot serve as deterministic native lifecycle proof |
| Allow sibling IDs because the parent owns both | Exceeds agreed scope and grants implicit cross-child access |
| Auto-resume a terminal recipient on message arrival | Changes task budgets, identity and lifetime without explicit authorization |
| Delete native inspection tools at inbox rollout | A retained message does not provide live process state, logs or cancellation |

## 22. Sources and inspection limits

**[R0]** Supplied `Pasted text(1).txt`, “Backgroundable tool execution: requirements and decision brief,” SHA-256 `211e73baa329cc4fd7b74953e855a4181357b038928ffa372b355682fe0c4f71`. Especially shared completion coordination, specialized result surfaces, no competing scheduler/journal, and automatic delivery without routine polling.

**[R1]** Subsequent user conversation: make results/model communication visible through a mailbox/inbox; retain useful status/log/cancel controls; restrict authored communication to parent↔child; split backgrounding from messaging and ship in pieces. The user did not require blocking reply waits, sibling teams, a specific tool name, or a universal jobs-tool replacement.

**[BG]** [Hatfield tool-backgrounding specification, revision 2](hatfield-tools-backgrounding-spec-v2.md). Its §7.5 SC-01–SC-10 is the normative shared contract; §21 preserves the pinned source bibliography from the original static review. This companion extends the retained-message producer/query surface rather than re-auditing current repository HEAD.

Existing source symbols and implementation observations are grounded in the attached baseline and recorded source map. New messages, views, tool actions, limits and question policies are clearly proposed design choices. No repository changes or repository tests were performed in preparing these revised specifications.

[R0]: #22-sources-and-inspection-limits
[R1]: #22-sources-and-inspection-limits
[BG]: hatfield-tools-backgrounding-spec-v2.md
