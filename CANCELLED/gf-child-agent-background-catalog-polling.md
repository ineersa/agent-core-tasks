# GF-06: Extract nested child-agent catalog discovery and background polling

## Goal
Extract the generic nested child-agent discovery/background-catalog functionality from the fork-MVP branch into a fresh task based on current main. This task belongs to the existing `/agents-live` child-agent infrastructure and must not depend on the fork tool.

Problem:
The existing live-view path primarily projects the currently selected child. When multiple children are active—or when one child launches another child—the TUI needs to ingest catalog/progress updates for non-selected runs without stealing the selected child's event stream. The historical fork branch added `SubagentLiveBackgroundChildPoller`, but keeping that inside fork work silently modifies shared subagent behavior and previously introduced a critical competing-consumer bug: the background poller drained the selected run's queue before `SubagentLiveChildViewPoller` could render it.

Architectural contract:
- The child-agent registry/catalog is flat from the TUI user's perspective: every discovered child run is selectable by `agent_run_id`/artifact identity regardless of nesting depth or launcher kind.
- Canonical artifact registries may be physically nested under a child run's artifact scope. Discovery must traverse those registries safely.
- Exactly one transcript consumer owns the selected child's live event queue.
- A background catalog poller may observe non-selected child streams only for catalog/progress ingestion. It does not project transcripts, route HITL, consume canonical replay/backfill, or render UI.

Recommended naming:
Use a generic name such as `ChildAgentCatalogPoller` rather than `SubagentLiveBackgroundChildPoller`. The responsibility is child-agent catalog synchronization, not a second live-view renderer.

Likely production surface:
- `src/CodingAgent/Agent/Artifact/AgentChildRunDirectory.php`
- new `src/Tui/Runtime/ChildAgentCatalogPoller.php`
- `src/Tui/Runtime/SubagentLiveChildViewPoller.php` only if a minimal explicit selected-stream ownership seam is required
- `src/Tui/Runtime/TuiSessionState.php` or a dedicated typed poller-state DTO
- `src/Tui/Runtime/SubagentLiveCatalog.php` only through its existing canonical runtime-event ingestion API
- `src/Tui/Listener/TickPollListener.php` or composition wiring only for invoking the catalog poller
- `config/services.yaml`

Historical reference only—do not cherry-pick tests or commits wholesale:
- Fork-MVP branch `task/fork-mvp-01-fork-tool-over-child-run-backend` around `ca001122b`, `9e070c43d`, and cleaned branch `bd1b295fc`.
- `ca001122b`: recursive/BFS registry scan in `AgentChildRunDirectory`.
- `9e070c43d`: background poller skips selected stream; selected child poller owns catalog ingestion callbacks.
- The historical implementation is evidence, not the target architecture. Reassess names, state ownership, scan frequency and API boundaries against current main.

Detailed requirements:

1. Recursive canonical artifact discovery
- Seed discovery from canonical top-level sessions.
- Traverse each discovered child's registry scope breadth-first or equivalently cycle-safe.
- Track visited parent scopes/run IDs; malformed cycles must not loop.
- Resolve nested child event/state paths from registry metadata and existing path resolvers; never invent pseudo-session paths or trust unsanitized IDs.
- Do not read full `events.jsonl` files merely to discover children.
- Existing top-level child discovery behavior must remain unchanged.

2. Selected stream ownership
- Determine the selected child run ID before invoking any client event read.
- The background poller must never call `AgentSessionClient::events()` for the selected run—not even to inspect and re-buffer results.
- `SubagentLiveChildViewPoller` remains the sole live-event consumer for the selected run.
- Selection changes must transfer ownership without consuming events from the newly selected run during the transition.

3. Background catalog ingestion
- Poll only known/discovered non-selected child runs that need catalog synchronization.
- Ingest only canonical runtime events containing supported child progress/catalog information, currently `tool_execution_update.subagent_progress`.
- Route through the catalog's existing runtime-event ingestion API; do not add a second payload parser with divergent semantics.
- Ignore transcript deltas, question events and unrelated runtime events.
- Do not call `BackfillEventProviderInterface`, replay snapshot providers, `EventStoreInterface::allFor()`, or directly open child `events.jsonl` during ordinary background ticks.

4. Replay and terminal-parent correctness
- Background polling must not consume one-shot or repeatable stored replay intended for selected-child transcript projection.
- A nested child remains discoverable when its launcher becomes terminal before the user opens `/agents-live`.
- If canonical registry discovery is the source of that guarantee, prove it directly; do not keep polling every terminal parent forever.

5. State and lifecycle
- Poll cursors, throttle timestamps and visited-run bookkeeping must be typed and explicitly owned; avoid dynamic properties on `TuiSessionState`.
- Bound retained per-run state and remove entries when runs disappear or no longer require polling.
- Polling cadence must be bounded and must not keep the TUI in active/busy rendering mode when no events arrive.
- No full-session or full-registry scan on every high-frequency TUI tick. Discovery refresh must be event-triggered or throttled with an explicit rationale.

6. Existing behavior protection
- Existing direct subagent transcript replay, steering, HITL, cancellation and selected live streaming must remain unchanged.
- Do not change `RuntimeQuestionEventHandler` or global child HITL routing.
- Do not change prompt/context behavior, progress payload schema, artifact statuses, footer rendering, export, context statistics or picker navigation.

Explicitly excluded:
- Fork tool/config/model/thinking support.
- `child_kind=fork` handling.
- Allowing subagents/forks to launch nested agents (`AgentDepthGuard` changes).
- Fork transcript bootstrap filtering or compact-summary projection.
- Fork HITL policy.
- Virtual compaction.
- Automatic deletion or retention settings.
- Transcript virtualization/performance redesign.

Test policy:
- First implementation commit must contain only reviewed RED behavior tests; no production code.
- Tests copied from the historical fork branch are forbidden. Reproduce stable user/runtime contracts against current main.
- Accepted red tests are immutable during implementation; fix production code, not tests.
- Use the lowest correct proof layer: focused service/runtime tests for stream ownership and recursive discovery, plus virtual TUI proof only where selection transitions require real screen/controller composition. Do not default to tmux.
- If a new unrelated defect appears, stop and create a separate bug task before changing behavior.
- All QA through Castor only.

Suggested incremental sequence:
1. Red tests for selected/non-selected event-stream ownership and backfill non-consumption.
2. Implement typed background catalog poller against current flat children only.
3. Red tests for nested registry discovery, cycle safety and terminal-launcher discovery.
4. Implement recursive `AgentChildRunDirectory` discovery with bounded refresh.
5. Wire invocation/selection ownership through the existing runtime composition.
6. Run focused Castor validation, reviewer, then deterministic full gate.

After this task merges, the fork-MVP branch should merge main and delete its copies of `SubagentLiveBackgroundChildPoller`, BFS directory scanning and shared selected-stream ownership changes, retaining only genuinely fork-specific integration.

## Acceptance criteria
- First commit/push contains only reviewed RED specification tests; no production implementation. Accepted tests remain unchanged during implementation.
- Background polling never calls `events()` for the currently selected child run.
- The selected child poller remains the sole consumer of selected-run live events, including across selection changes.
- Background polling consumes no stored replay/backfill and performs no direct `events.jsonl` reads.
- Only non-selected child `subagent_progress` runtime events are ingested into the catalog; transcript/HITL/unrelated events are ignored.
- Nested registry discovery finds at least main → child → nested-child canonical artifacts in a fresh process with no pre-populated in-memory directory cache.
- Recursive discovery is cycle-safe, validates canonical identities/paths and does not create pseudo-session directories.
- A nested child remains discoverable after its launcher reaches a terminal state without permanently polling every terminal launcher.
- Existing top-level child discovery and selected-child replay/live streaming behavior remain unchanged.
- Poller state is represented by a dedicated typed state object or equally explicit typed ownership—not ad hoc dynamic fields scattered across TuiSessionState.
- Discovery/polling is bounded: no full event-log reads and no unthrottled all-session registry scan on high-frequency ticks.
- Idle background polling does not trigger transcript invalidation or force active TUI render cadence when no catalog change occurs.
- `RuntimeQuestionEventHandler`, child HITL routing, prompts, progress schema, export, context statistics and picker navigation are unchanged.
- No fork-specific classes, metadata, conditions or terminology are required by the implementation.
- Focused Castor service/runtime and appropriate virtual TUI tests pass; final `castor test`, `castor test:controller-replay` where relevant, `castor deptrac`, `castor phpstan`, `castor cs-check`, and deterministic `castor check` pass.
- After merge, the fork-MVP branch can remove its duplicated background poller/BFS/stream-ownership code and consume the main implementation without altering incoming main behavior.

## Workflow metadata
Status: CANCELLED
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed: 2026-07-18T23:06:45Z

## Work log
- Created: 2026-07-12T21:19:47.281Z

## Task workflow update - 2026-07-18T23:06:45Z
- Cancelled because nested child agents are structurally forbidden and the obsolete GF follow-up is no longer required.
