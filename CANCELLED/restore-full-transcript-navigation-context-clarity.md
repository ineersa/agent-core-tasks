# Restore full transcript navigation and make context usage explainable

## Goal
In session 33 the footer showed roughly `CTX 16% 158.7k/1000.0k`, while live transcript view exposed only about 5–7 recent turns. Canonical inspection found a much larger history (445 events, 22 turn advances, 56 state messages) and no compaction event at inspection time. The transcript widget iterates supplied blocks, so the symptom may be viewport/history navigation rather than projection truncation. Usage projection also tracks cumulative session input separately from latest per-request input; the UI must not conflate those values.

Treat canonical events plus the active leaf as transcript truth. Determine whether older blocks are omitted during replay, discarded by widgets, inaccessible because scrolling/virtualization is broken, or merely off-screen without discoverable navigation. Reconcile the context indicator with the actual latest provider measurement and clearly separate it from cumulative billed/input totals. Do not assume visible prose alone equals provider context: system/user-context/tool/schema content may contribute without being rendered as normal turns.

## Acceptance criteria
- On resume of an isolated 20+ turn branched session, every user-visible block on the active leaf is present in transcript projection in canonical order; no arbitrary recent-turn cap is applied unless explicitly configured and disclosed.
- Users can navigate from the latest block to the oldest active-branch block in the live TUI using discoverable existing or new transcript navigation; viewport size must not make history effectively unavailable.
- If projection is already complete, fix the render/scroll/history-loading path rather than duplicating transcript storage.
- Label and compute current context-window usage from the latest provider request measurement, while cumulative session input/output and cost remain separately identified; never display cumulative input as `CTX`.
- Provide an explainable breakdown or help text stating that hidden system/user-context/tool/schema content can contribute to context even when not rendered as ordinary conversational turns.
- Add virtual TUI proof with enough turns to exceed the viewport and assert navigation reaches old content, plus usage replay proof distinguishing latest context from cumulative totals. Add controller replay only if the defect is in resume protocol/projection rather than local rendering.
- Preserve active-leaf branch semantics and canonical `events.jsonl` as the source of truth; do not introduce compatibility readers or a parallel transcript format.
- Run all QA through Castor, including focused virtual/runtime validation and full `castor check` before CODE-REVIEW.

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
- Created: 2026-07-23T18:52:18.411Z

## Task workflow update - 2026-07-23T19:15:46.854Z
- Summary: Exact root causes confirmed for the subagent live view. The displayed `620 turns` is an internal AgentCore turn/event cursor, not conversational turns: the child had 25 completed LLM steps, one failed step, 1 user message, 25 assistant messages, and 103 tool messages. Live replay/projector retains and forwards the complete transcript with no recent-N cap; only ~5-7 blocks are visible because the terminal viewport shows the bottom and older blocks are not effectively navigable. `CTX 158.7k` came from the child DTO's stale/estimated `latestInputTokens`, not canonical provider usage: latest successful input was 118,182 and cumulative input was 1,821,397. Hidden system, user-context, and 103 tool messages explain why provider context exceeds visible prose, but not the stale 158.7k value.
- Bounded scout verified `SubagentLiveChildViewPoller::replaySnapshot()` replays all events, `TranscriptProjectionState::blocks()` returns all ordered blocks, and picker forwards them to ChatScreen. The defect is viewport/navigation plus stale usage attribution, not event polling truncation.

## Task workflow update - 2026-07-23T19:57:05.226Z
- Summary: HTML comparison resolved. The only export is `hatfield-child-agent_87b2096b0ea059ff.html`, for a different completed child (`agent_87`), not failed child `agent_dd`. It contains exactly that child's 629 canonical events and is an event-level debug export: every event card, raw JSON, tool-call details, and tool result are rendered, producing 106 tool interactions / 212 tool DOM blocks. It has 23 assistant messages versus failed child `agent_dd`'s 25, so it does not contain 5–6× more conversational turns. The live view receives all 132 failed-child transcript blocks but ChatScreen has no transcript scrolling/virtualization; only the bottom viewport is usable. The user-visible defect remains real: full projected history is effectively inaccessible, and the export/live representations make that impossible to understand.
- SessionEventsExportService renders the supplied child eventsPath directly. SubagentLiveChildViewPoller and TranscriptProjectionState retain all child blocks; ChatScreen passes all blocks to one LiveTextWidget without transcript-specific scroll navigation.

## Task workflow update - 2026-07-23T20:08:18.324Z
- Summary: Deferred by user pending additional manual reproduction. Do not start or broaden this task yet. Preserve current evidence about different child export identity, raw-event HTML density, full in-memory projection, absent transcript navigation, stale CTX DTO, and misleading internal turn counter for later comparison.

## Task workflow update - 2026-08-04T20:24:20.403Z
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Cancelled at user request; no implementation was started.
