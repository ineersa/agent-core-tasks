# ARCH: relocate turn tree + replay/rewind logic out of AgentCore

## Goal
## Thesis (primary architectural concern)
Turn-tree and replay/rewind logic is misplaced in `src/AgentCore/`. AgentCore should own the **core loop, domain primitives, and contracts/storage** — NOT conversation/session-management concepts like turn branching, leaf sets, or rewind-to-turn. These are session/runtime operations triggered by the TUI and handled by the controller. Their current placement in Core is a category error that the architecture review (and the user) flagged as the dominant issue to address.

Stated as a rule for the target design: **AgentCore must stay free of session/conversation-management logic. Turn tree + replay/rewind belongs in the App (CodingAgent) layer, near the controller/runtime that drives it.**

## Why it matters
- **Conceptual boundary:** `TurnTreeProjector`, `TurnTreeReplayFilter`, `RunRewindService`, and the rewind bits of `RunStateReplayService` are session/conversation concepts, not core-loop/domain. They grow with rewind/branching features (see sibling `session-08-exact-file-rewind-checkpoints`) and will keep pulling Core toward runtime concerns.
- **Drift surface:** placing rewind orchestration in Core forced the App layer to mirror its emission across two transport paths (arch HIGH-1 below) and pulled transcript rebuild into the TUI (arch HIGH-2). Relocating the logic lets these be solved structurally.
- **Future features** (branch delete, re-root, multi-leaf, exact-file rewind) will all touch this cluster — better to have it in the right layer before they land.

## Current footprint to relocate (12 files, all in AgentCore)
**Domain (3):**
- `src/AgentCore/Domain/Run/TurnTreeProjector.php` — pure projector (events → tree), `activePathTo()`
- `src/AgentCore/Domain/Run/TurnTreeDTO.php`
- `src/AgentCore/Domain/Run/TurnTreeNodeDTO.php`

**Application / Replay + Handlers (5):**
- `src/AgentCore/Application/Replay/TurnTreeReplayFilter.php` — `filterForLeaf()`, `buildCommandSeqToCreatedTurnMap()` (command→created-turn attribution)
- `src/AgentCore/Application/Handler/RunRewindService.php` — rewind orchestration (NEW in session-07): lock → append LeafSet → rebuildForLeaf → CAS-persist
- `src/AgentCore/Application/Handler/RunStateReplayService.php` — `rebuildIfStale()`, `rebuildForLeaf()`, `filterAbandonedChildLaunchCommandsOnTargetLeaf()`, `resolveCurrentLeafTurnNo()`
- `src/AgentCore/Application/Handler/ReplayService.php` — older replay service (assess whether it shares the same category)
- `src/AgentCore/Application/Handler/RunStateReplayException.php`

**Application / DTOs (4):**
- `src/AgentCore/Application/Dto/ReplayIntegrity.php`
- `src/AgentCore/Application/Dto/ResolvedReplayEvents.php`
- `src/AgentCore/Application/Dto/RunStateReplayResult.php`
- `src/AgentCore/Application/Dto/TurnBranchReplayDTO.php`

**Related (assess, likely stays in Core):**
- `src/AgentCore/Application/Pipeline/AdvanceRunHandler.php` records `parent_turn_no` on `turn_advanced`. The recording of parent_turn_no on the canonical event is arguably domain; only the rewind/branch *interpretation* of it moves. Decide the split during design.

## What these depend on (boundary constraint — must remain reachable)
The relocated logic currently uses ONLY Core contracts/domain types:
`EventStoreInterface`, `RunStoreInterface` (Contract); `RunEvent`, `RunEventTypeEnum`, `RunState` (Domain). The App (CodingAgent) layer is allowed to depend on Core, so relocation is feasible without boundary violations — but `castor deptrac` rules (`depfile.yaml`) must be updated for any new namespace, and no new `AgentCore → CodingAgent` dependency may appear.

## Target layer (open design question — do NOT pre-decide in this task; resolve when picked up)
Candidates to evaluate:
1. **`src/CodingAgent/Session/`** — already exists; `SessionTurnTreeProvider` already lives nearby (`src/CodingAgent/Runtime/Session/SessionTurnTreeProvider.php`). Natural home for conversation-management logic.
2. **A new dedicated sub-namespace** under CodingAgent (e.g. `src/CodingAgent/Runtime/Replay/` + `src/CodingAgent/Runtime/Tree/`) if "Session" is too broad.
3. Keep a thin **pure projector** in Core Domain IF the team decides events→tree projection is genuinely domain-agnostic — but the user's preference is that even the tree model is session-management, so default toward moving it out.

⚠️ **Naming tension to resolve:** both `src/CodingAgent/Session` and `src/CodingAgent/Runtime/Session` exist today. Consolidate or clearly separate before adding more code under either.

## Fold in the architecture-review findings (solve during relocation)
These were flagged on the session-07 branch and are naturally resolved by the relocation:
- **arch HIGH-1 (drift):** `RunLeafChanged` is constructed independently in `RewindToTurnHandler` (Process) and `InProcessAgentSessionClient::handleInProcessRewind` (InProcess). Once rewind orchestration lives in the App layer, introduce a **single producer** (factory) that both transport paths call. (Note: `RunRewindService` cannot build `RuntimeEvent` itself while it's in Core — relocation removes that constraint, making the clean factory trivial.)
- **arch HIGH-2 (boundary leak):** TUI `RuntimeEventPoller` calls `activePathRuntimeEvents()` and projects transcript locally. Replace with a higher-level App-layer contract (e.g. `transcriptBlocksForLeaf(runId, leafTurnNo): list<TranscriptBlock>`) returning pre-projected blocks so the TUI only assigns, never projects.
- **arch MEDIUM-1:** command→created-turn attribution (`buildCommandSeqToCreatedTurnMap`) is tree data; co-locate with the projector in whatever layer the tree lands.
- **arch MEDIUM-2:** `rebuildForLeaf` duplicates ~60 lines of integrity-check logic from `rebuildIfStale`; extract a shared `prepareForReplay()` during the move.
- **arch LOW-1/2/3:** dedupe `maxSeq()`; consider `EventStoreInterface::maxTurnNo()`; decide `TurnTreeNodeView` `lastSeq`/`reason` forwarding.

## Approach guidance
- **Behavior-preserving refactor.** The immutable live proof `tests/CodingAgent/Runtime/Controller/E2E/RewindBranchLiveE2eTest.php` (11 assertions, full pineapple/apple branch scenario) must stay green and byte-identical — it is the guard that the relocation changed no behavior.
- **Stage the work:** produce a short relocation plan (which cluster moves first, what depfile rules change, how tests move) before large moves, so it can be reviewed in increments rather than one giant PR. Consider splitting into sub-tasks (e.g. (a) move projector+DTOs+filter, (b) move rewind/replay orchestration, (c) unify emission factory + transcript contract).
- **No backward-compat shims / dual-format paths** (AGENTS.md): replace old locations, update all references, move tests alongside.
- This is follow-up work off `main`, **after SESSION-07 (PR #242) merges**. Do not branch off the session-07 task branch.
- Load the `task-workflow` and `testing` skills when starting; run `castor check` (full gate) as the completion bar.

## Out of scope / deferred
- `session-08-exact-file-rewind-checkpoints` is a separate feature task; this architecture task should make room for it, not implement it.
- Per-test cache-nondeterminism is tracked separately (`llm-real-first-warm-cache-growth`).

## Acceptance criteria
- Target layer + namespace chosen with rationale (relocation out of AgentCore), naming tension with src/CodingAgent/Runtime/Session resolved
- Turn tree (Projector + DTOs) relocated or explicitly justified to stay; Core stays free of session/conversation-management logic
- Replay/rewind orchestration (Filter + RewindService + replay/rewind bits) relocated to chosen layer; Core contracts (EventStoreInterface/RunStoreInterface) still depended-on correctly (App→Core allowed)
- castor deptrac green with updated depfile rules; no AgentCore→CodingAgent dependencies introduced
- Single producer for RunLeafChanged RuntimeEvent (arch HIGH-1 closed): exactly one construction site, both transport paths call it
- TUI rebuild no longer projects events locally (arch HIGH-2 closed): higher-level transcript contract (e.g. transcriptBlocksForLeaf) returns pre-projected blocks
- RewindBranchLiveE2eTest 11/11 green unchanged (behavior-preserving) + full castor check green
- Relocation plan written/recorded (sub-tasks or a plan doc) before large moves so the work is reviewable in stages
- No new @phpstan-ignore annotations; no backward-compat shims; no history rewrite

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
- Created: 2026-07-01T00:29:51.044Z

## Task workflow update - 2026-07-06T00:49:41.145Z
- Summary: Architecture reflection after session-08b file-rewind loops: the turn tree is currently a mixed-purpose model, not merely a finicky picker implementation. It serves as branch graph, rewind/resume target list, transcript replay filter, display-label source, and file-rewind checkpoint label source. This overloaded role explains why file rewind pulled /tree into scope and caused repeated loops. Target cleanup should keep AgentCore limited to canonical run/event facts (turn_advanced, parent_turn_no, leaf_set) and move interpretation into CodingAgent/App session runtime. TUI should render projected views and request high-level operations, not replay/filter raw events or infer display roles/labels. File-rewind extension should consume generic session labels/turn-boundary views only, not own or reshape turn-tree semantics. Recommendation: do not broaden PR #255 further; keep current tree fixes minimal and make this session-07 task the dedicated design/refactor for tree/replay/rewind boundaries.
- Reflection added: turn tree debt is architectural, not just UI rendering. Current model conflates branch graph, safe resume/rewind boundaries, transcript rebuild, display labels, and file checkpoint targets.
- Suggested future split: AgentCore stores/emits canonical facts only; CodingAgent/App owns turn-tree projection, replay filtering, rewind/resume boundary semantics, and transcriptBlocksForLeaf-style APIs; TUI only renders/apply operations; file-rewind extension consumes generic session display/boundary data.
- PR #255 guidance recorded: avoid more tree architecture changes in file-rewind PR; use session-07 task for the larger relocation and clearer App-layer session model.

## Task workflow update - 2026-07-06T01:30:43.067Z
- Summary: User approved the key design split: Core keeps pure event reducers; CodingAgent/App owns branch-aware/session orchestration. Pure prompt-state reducer may stay in Core, but branch-aware hot prompt rebuild moves. `TurnTreeProviderInterface::activePathRuntimeEvents()` should be replaced, and TUI should consume projected transcript blocks for a leaf. Runtime rewind should use a single `RunLeafChanged` producer.

Implementation plan recorded as staged subtasks:

1. `session-07a-move-turn-tree-projector-and-filter-to-app-session` — behavior-preserving relocation of `TurnTreeProjector`, tree DTOs, `TurnTreeReplayFilter`, and `TurnBranchReplayDTO` from AgentCore into CodingAgent/App session namespaces. Preferred namespaces: `src/CodingAgent/Session/TurnTree/` and `src/CodingAgent/Session/Replay/`. Move tests alongside and keep deptrac green. This slice intentionally avoids replay-service splitting and TUI contract changes.

2. `session-07b-split-core-reducers-from-app-replay-rewind-orchestration` — split `RunStateReplayService` and `ReplayService` so Core retains pure event→`RunState` / event→`PromptState` reducers/helpers and contracts, while App session owns branch-aware rebuild, active-leaf replay, rewind-to-turn orchestration, and branch-aware hot prompt rebuild. Move `RunRewindService` out of AgentCore. Introduce narrow Core contracts as needed so `RunMessageProcessor`/`RunCommit` do not depend on App concrete classes. Extract shared replay preparation/integrity logic instead of duplicating stale-vs-leaf rebuild checks.

3. `session-07c-app-runtime-rewind-boundaries-and-tui-transcript-contract` — close runtime/TUI boundary findings after relocation: add projected transcript-block contract for leaf changes, remove TUI local active-path event replay, replace/remove `TurnTreeProviderInterface::activePathRuntimeEvents()`, and introduce a single `RunLeafChanged` factory used by both process and in-process rewind paths.

Parent task should now act as the architecture umbrella. Implement subtasks sequentially (07A → 07B → 07C) unless a future design pass finds a safer narrower slice. Completion bar for the umbrella remains: deptrac/phpstan/cs-check green, focused tests for each slice, `castor test:tui --filter=TuiTree` for TUI leaf-change behavior when touched, `castor test:llm-real --filter=RewindBranchLiveE2e`, and final deterministic `castor check` before CODE-REVIEW.

## Task workflow update - 2026-07-07T18:05:00Z
- Summary: Closed architecture umbrella after all staged subtasks completed and merged: SESSION-07A relocated turn tree projector/filter to App session layer (PR #262), SESSION-07B split Core pure reducers from App replay/rewind orchestration (PR #263), and SESSION-07C added App runtime rewind boundaries + TUI transcript contract (PR #264).
- Validation: Post-merge `LLM_MODE=true castor check` passed in 297.5s (deptrac, test, controller replay, TUI, llm-real, phpstan, cs-check, cache guard, artifact integrity, and leak check all OK).
- Status: DONE.
