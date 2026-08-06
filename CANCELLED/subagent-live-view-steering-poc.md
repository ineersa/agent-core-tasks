# POC Subagent live view and steering from TUI

## Goal
## Context

Create a POC using the existing subagent execution backend only — no fork implementation, no new tmux/process fork architecture.

Goal: keep subagents running as they already do from the main TUI, but expose their existing child run IDs so the user can open a read-only live view for a running subagent and steer/follow-up that child directly.

This is based on scout findings:
- Existing subagent runs already have `agentRunId` and artifact IDs recorded by `SubagentExecutionService` / `AgentArtifactRegistry`.
- Lower layers already allow `AgentRunnerInterface::steer($runId, ...)` / `followUp($runId, ...)` / `cancel($runId, ...)` for any run ID.
- Fastest MVP is for TUI/controller to target the child `agentRunId` directly, rather than changing the subagent blocking poll loop.
- Existing subagent progress events and artifact registry can provide running-agent selection data.

## POC behavior

1. Subagents keep executing exactly as today from the main TUI.
2. Add `/agents-live` command.
3. `/agents-live` shows a selectable list of currently running subagents/child agents.
   - Each item should identify agent name, artifact ID, child run ID, status, and task excerpt where available.
4. Pressing Enter on a selected item switches the TUI into that child agent's readonly live view.
   - Reuse the existing readonly tab/live-view plan/discovery from the earlier fork POC where possible.
   - The live view should render the selected child run's current events/transcript/progress without mutating it.
5. Add command/keybind to return from child live view to the main session.
6. While in a child live view, focus the normal editor for steering.
   - User input should be routed to the selected child run ID, not the parent run ID.
   - Use active child run state to decide steer vs follow_up: active child gets steer; completed/idle child gets follow_up if supported/appropriate.
   - This is intentionally experimental: record whether steering a running subagent actually works in practice.
7. Do not implement fork tool behavior in this task.

## Important constraints

- Main agent/orchestrator must not edit files directly; implementation must happen through fork when task is started.
- Before writing/running tests, implementation fork must load `.agents/skills/testing/SKILL.md` and read `tests/AGENTS.md`.
- All QA must use Castor, never raw `vendor/bin/*`.
- TUI behavior proof must be at the lowest correct layer: likely virtual/in-process for command/list/view routing plus controller-replay for runtime command targeting; tmux only if raw terminal integration is actually required.
- Avoid intrusive runtime rewrites. Prefer existing subagent artifacts/progress, existing `AgentSessionClient` command path, and existing TUI command/slot mechanisms.

## Likely affected areas from scout reports

- `src/CodingAgent/Agent/Execution/SubagentExecutionService.php` — emits progress and creates artifact/child run IDs.
- `src/CodingAgent/Agent/Artifact/AgentArtifactRegistry.php` / retrieval services — map artifact IDs to child run IDs and metadata.
- `src/Tui/Command/SlashCommandRegistry.php` / slash command handlers — add `/agents-live`.
- `src/Tui/Runtime/TuiSessionState.php` or equivalent state holder — track active live child view and known child runs.
- `src/Tui/Listener/SubmitListener.php` — route editor input to selected child run while in live view.
- `src/Tui/Listener/TickPollListener.php` / runtime poller/projection path — keep live view updated.
- `src/CodingAgent/Runtime/Contract/UserCommand.php`, `AgentSessionClient`, controller handlers — verify child run steering path.

## Open questions to resolve during task-explain/scout phase

- Source of truth for "currently running agents": live progress snapshots, artifact registry, child run directory, RunStore scan, or a TUI-maintained index from progress events?
- Exact live view UI: readonly tab, panel, alternate transcript state, or screen mode?
- Return UX: slash command (`/main` or `/agents-main`), keybind, or both?
- Steering semantics for terminal child: should follow_up revive completed child, or should POC only allow steer while active?
- How to identify parallel subagents cleanly in the picker.
- What is the smallest reliable automated proof for the live-view switch and child-run message routing?

## Acceptance criteria
- `/agents-live` lists currently running subagents/child agents with enough identifying data to choose one.
- Selecting a running agent switches TUI into a readonly live view for that child run.
- A command and/or keybind returns from child live view to the main session.
- While in child live view, editor submissions target the selected child run ID instead of the parent run ID.
- POC documents whether steer/follow_up to a running subagent works in practice, including known limitations/races.
- No fork tool or fork tmux/process implementation is added in this task.
- Automated TUI/runtime proof is added at the lowest correct layer, with Castor validation recorded.
- Implementation handoff confirms testing skill and `tests/AGENTS.md` were read before test work.

## Workflow metadata
Status: CANCELLED
Branch: task/subagent-live-view-steering-poc
Worktree:
Fork run: 1ndxe19zikyh
PR URL:
PR Status:
Started: 2026-06-30T16:13:08.038Z
Completed: 2026-07-18T23:06:45Z

## Work log
- Created: 2026-06-30T14:21:58.374Z

## Task workflow update - 2026-06-30T16:13:08.038Z
- Moved TODO → IN-PROGRESS.
- Created branch task/subagent-live-view-steering-poc.
- Created worktree /home/ineersa/projects/agent-core-worktrees/subagent-live-view-steering-poc.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/subagent-live-view-steering-poc.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/subagent-live-view-steering-poc.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/subagent-live-view-steering-poc.
- Summary: Starting POC implementation per user request: subagent-only live view and steering in TUI; no fork/tmux fork architecture; user requested no tests/gate checks for this exploratory POC.

## Task workflow update - 2026-06-30T16:13:44.144Z
- Recorded fork run: 3ldcgwfykfoq
- Summary: Implementation fork launched in worktree /home/ineersa/projects/agent-core-worktrees/subagent-live-view-steering-poc. Scope: exploratory POC for existing subagents only; /agents-live picker/list, child readonly live view, return to main, editor routing to selected child run via steer/follow_up. Per user request, no tests/gate checks for this POC; fork instructed to read testing skill and tests/AGENTS.md but not run validation unless unavoidable.

## Task workflow update - 2026-06-30T16:22:22.673Z
- Recorded fork run: 3ldcgwfykfoq
- Validation: No Castor tests/checks run per explicit user directive for exploratory POC.; Fork ran php -l on selected changed PHP files only.
- Summary: POC implementation fork completed at commit 71e59e245 on branch task/subagent-live-view-steering-poc. Implemented /agents-live picker/list, readonly child live view, /agents-main and Ctrl+\\ return, editor routing to selected child agentRunId via steer/follow_up, and agent_run_id propagation in subagent_progress. No fork/tmux fork architecture added. No tests/gate checks run per user directive; fork reported reading testing skill and tests/AGENTS.md before touching TUI/runtime code.

## Task workflow update - 2026-06-30T20:23:09.058Z
- Recorded fork run: vsu077rp91s0
- Summary: User manual smoke hit runtime crash: SelectListWidget::addListener() does not exist in Symfony TUI. Launched narrow fix fork to update SubagentLivePickerController to existing project pattern using SelectListWidget::onSelectionChange(), onSelect(), onCancel() plus Keybindings. No tests/gates per POC directive.

## Task workflow update - 2026-06-30T20:23:44.922Z
- Recorded fork run: vsu077rp91s0
- Validation: php -l src/Tui/Picker/SubagentLivePickerController.php passed in worktree.; No Castor tests/gates run per explicit user POC directive.
- Summary: Crash fix fork completed at commit 5ff6681fb. Fixed /agents-live runtime fatal by replacing invalid SelectListWidget::addListener() usage in SubagentLivePickerController with project-standard SelectListWidget callbacks: Keybindings + onSelectionChange/onSelect/onCancel. Picker behavior preserved: list, accent selection, Enter enters live view, Esc/Ctrl+C cancels. No Castor tests/gates run per POC directive; php -l on touched file passed.

## Task workflow update - 2026-06-30T20:39:24.020Z
- Summary: Scout investigation of user's manual session 2 completed. Findings: session id maps directly to .hatfield/sessions/2; parent events contain subagent_progress payloads; registry entry maps artifact agent_cc22edc9cd7f7ba0 to child run 22cad550-1e4c-4b1e-b353-4b8464982027; child events exist at .hatfield/sessions/2/artifacts/agents/agent_cc22edc9cd7f7ba0/events.jsonl with 39 events. Steering appears to have worked: child events include agent_command_applied for user steer text 'Skip last sleep'. Primary live-view bug: JsonlProcessAgentSessionClient only sees controller stdout events for registered parent runs; RuntimeEventEmitter never registers child run cursors, so child events are not emitted to TUI in process/PHAR mode. In-process path would work via ChildAwareEventStore. Secondary issue: SubagentLiveChildViewPoller creates scratch TuiSessionState per event instead of per batch, causing cosmetic child activity/usage projection inconsistencies.
- Scout session-2 data paths: .hatfield/sessions/2/events.jsonl; .hatfield/sessions/2/artifacts/agents/registry.json; .hatfield/sessions/2/artifacts/agents/agent_cc22edc9cd7f7ba0/events.jsonl; child run id 22cad550-1e4c-4b1e-b353-4b8464982027.
- Recommended fix from scouts: register child run IDs in RuntimeEventEmitter when parent tool_execution.output_delta/subagent_progress payloads are emitted, so the controller drains/forwards child canonical events; also move SubagentLiveChildViewPoller scratch TuiSessionState outside per-event loop.

## Task workflow update - 2026-06-30T20:41:37.447Z
- Recorded fork run: 9z1lty3yrrnq
- Validation: php -l passed on RuntimeEventEmitter.php, SubagentLiveChildViewPoller.php, SubagentLivePickerController.php in worktree.; No Castor tests/gates run per explicit user POC directive.
- Summary: Live-view event delivery fix completed at commit 7473e0cc9. RuntimeEventEmitter now registers child agent_run_id values from parent subagent_progress payloads so controller/process/PHAR mode drains and forwards child canonical events over JSONL; SubagentLiveChildViewPoller now uses one scratch TuiSessionState per batch; entering live view shows a placeholder until child events arrive. Session-2 evidence confirmed child steer 'Skip last sleep' was queued/applied in child artifact events, so steering path worked at storage/pipeline level; prior issue was child event delivery to TUI. No Castor tests/gates run per user directive; php -l passed on touched files.

## Task workflow update - 2026-06-30T21:06:39.703Z
- Summary: Additional scout investigation for user concern that steer did not reach subagent: sessions 2 and 3 both show steer was delivered and applied, but only at command mailbox/tool-batch boundary. Session 2: steer 'Skip last sleep' queued seq 6 at 20:28:32 while first sleep 60 was running, applied seq 12 at 20:29:06 after sleep completed; next child LLM turn issued ls/find, no second sleep. Session 3: steer 'Don't run 2nd sleep 60!' queued seq 6 at 20:58:41 during first sleep, applied seq 12 at 20:59:11 after sleep completed; next assistant explicitly said 'Understood. Skipping the second sleep' and issued non-sleep tools. Current steer semantics cannot interrupt an in-flight sleep and cannot prevent already-emitted later tool calls in the same assistant tool batch; it only affects the next LLM boundary. For stronger UX, POC should label steering as queued/non-interrupting or implement a separate tool-batch interruption/cancel design later.
- Scout code finding: AgentRunner::steer enqueues ApplyCommand/CoreCommandKind::Steer; CommandMailboxPolicy drains at turn-start or stop boundary and appends user message; ToolExecutor cancellation token checks do not observe steer and do not interrupt running shell tools.
- Scout code finding: if an LLM emits multiple tool calls in one step, steer arriving after tool 1 starts will not prevent tool 2 in the same batch; ToolBatchCollector has no mid-batch command-mailbox check to cancel remaining pending tool calls.

## Task workflow update - 2026-06-30T21:06:56.174Z
- Recorded fork run: 2cjuxvebaq3y
- Validation: php -l src/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClient.php passed in worktree.; No Castor tests/gates run per explicit user POC directive.
- Summary: Tab-switch/live-view hang fix completed at commit b349f9471. Root cause: JsonlProcessAgentSessionClient::events($runId) consumed all stdout pipe events but dropped non-matching run IDs; live child polling could therefore discard parent terminal/progress events, making TUI look hung while resume/replay from disk was correct. Fix re-buffers non-matching RuntimeEvents so parent and child polling can share the same JSONL pipe. RuntimeEventEmitter child registration retained because controller eventClient is InProcessAgentSessionClient and can read child artifact events via ChildAwareEventStore. No Castor tests/gates run per user POC directive; php -l on touched file passed.

## Task workflow update - 2026-06-30T21:27:41.280Z
- Summary: User manual smoke after b349f9471: steering worked in session 4, but actual tabs/live rendering remains fragile — live view blank/halts, updates mostly after switching main/live, main TUI does not show agent response/progress/results while live view active. Scouts found likely UI-state/polling causes: parent transcript is accumulated by RuntimeEventPoller but never pushed to screen while live view is active; /agents-main and Ctrl+\ restore stale subagentLiveParentTranscriptBackup instead of current state->transcript; live view resets child projection/cursor on every re-entry so previously streamed child events cannot be replayed from JSONL pipe, causing blank placeholder on re-entry; child poller returns null on no new events and TickPollListener leaves screen unchanged; working message early return hides parent activity. Additional runtime finding: post-b349f9471 parent/child events are present in sessions 3/4; problem is projection/UI state, not event absence.
- Loaded task-workflow and testing skills and read tests/AGENTS.md before preparing further TUI/runtime fork instructions.
- Main bounded reads inspected TickPollListener live branch and early return, SubagentLiveChildViewPoller poll/null behavior, SubagentLivePickerController enter/reset/placeholder, AgentsMainCommandHandler and SubagentLiveHotkeyRegistrar stale backup restore, SubmitListener child-run dispatch path, RuntimeEventPoller parent ingestion, SubagentLiveViewState reset-on-exit behavior, SubagentLiveCatalog DTO refresh API.

## Task workflow update - 2026-06-30T21:29:30.958Z
- Recorded fork run: t0is3utx6253
- Validation: php -l passed on SubagentLiveViewState.php, TickPollListener.php, SubagentLivePickerController.php, AgentsMainCommandHandler.php, SubagentLiveHotkeyRegistrar.php in worktree.; No Castor tests/gates run per ongoing POC manual loop.
- Summary: Subagent live-view TUI robustness pass completed at commit bc0863601. Changes: returning from /agents-main or Ctrl+\\ now renders current state->transcript instead of stale backup; SubagentLiveViewState preserves same-child selected/cache/lastSeq across main/live toggles; SubagentLivePickerController only resets child projection for a different child/no cache and reuses cached childTranscript on same-child re-entry; TickPollListener polls child first while live view active, still polls parent every tick, refreshes selected child status from catalog, repaints cached child transcript when no new child events arrive, and uses combined parent/child working message. Parent transcript still not shown while live view active (child-only screen by design), but should restore immediately on main. No tests/Castor gates run; php -l passed on touched files.

## Task workflow update - 2026-06-30T21:41:37.957Z
- Summary: User manual smoke after bc0863601: severe TUI flicker/cursor flying while live view active ('almost got epilepsy'), though something is drawing underneath. Suspected regression from robustness pass forcing cached child transcript repaint/requestRender every tick while live view is active, causing full terminal re-renders at ~20Hz even without new content.

## Task workflow update - 2026-06-30T21:42:53.148Z
- Recorded fork run: 2v14myrtwbx2
- Validation: php -l src/Tui/Runtime/SubagentLiveViewState.php passed in worktree.; php -l src/Tui/Listener/TickPollListener.php passed in worktree.; No Castor tests/gates run per ongoing POC manual loop.
- Summary: Flicker regression fix completed at commit d86f651e5. Root cause was bc0863601 live branch repainting cached childTranscript and forcing requestRender(true) every tick when child poll returned null, causing full terminal invalidation at ~50ms. Fix removes per-tick cached transcript repaint/requestRender; live transcript updates only when child poll returns new blocks or on explicit enter/re-entry. Added live-view working-message dedup via SubagentLiveViewState::lastLiveWorkingMessage cleared on exit. Parent poll every tick, main return behavior, and same-child cache behavior preserved. No Castor tests/gates run; php -l passed on touched files.

## Task workflow update - 2026-06-30T21:56:16.152Z
- Summary: User wants next POC step: enable real HITL for subagents so SafeGuard approval and ask_human can be exercised from the live child view. Scout findings: current subagents are intentionally non-interactive — SubagentExecutionService sets RunMetadata session.interactive=false, AgentPromptBuilder tells child not to ask humans, SubagentExecutionService cancels/fails child when RunStatus::WaitingHuman, NoninteractiveChildRunProbe marks these runs noninteractive, ExtensionToolHookEventSubscriber and SafeGuardToolCallHook auto-deny RequireApproval/noninteractive child runs. ask_human exists as ToolExecutionMode::Interrupt and is included when child agent has unrestricted tools, but explicit tool allowlists may omit it. TUI findings: parent HITL flow uses RuntimeEventPoller callbacks into QuestionCoordinator/QuestionController; SubagentLiveChildViewPoller currently only projects transcript and ignores human_input.requested/tool_question.requested; SubmitListener will answer active QuestionCoordinator question before steer/follow_up, so if child question callbacks enqueue with child runId, answer routing can target child. Need guard TickPollListener reject logic so parent-idle does not reject active child questions while child is still active/waiting.

## Task workflow update - 2026-06-30T21:59:01.941Z
- Recorded fork run: lprq25bta82h
- Validation: php -l passed on SubagentExecutionService.php, AgentPromptBuilder.php, AgentToolPolicyResolver.php, SubagentToolDefinitionBuilder.php, SubagentLiveChildViewPoller.php, SubagentLiveChildDTO.php, TickPollListener.php in worktree.; No Castor tests/gates run per ongoing POC manual loop.
- Summary: Interactive child/subagent HITL POC completed at commit eb1ad8eca. Changes: SubagentExecutionService starts child runs with session.interactive=true, treats WaitingHuman as non-terminal polling state and emits waiting_human progress; AgentPromptBuilder/SubagentToolDefinitionBuilder wording updated from non-interactive to foreground child that may use ask_human/SafeGuard approvals; AgentToolPolicyResolver appends ask_human when active and missing from subagent allowlist for POC; SubagentLiveChildViewPoller now invokes HITL/tool-question callbacks for child runtime events; TickPollListener wires child callbacks into existing QuestionCoordinator/QuestionController and fixes orphan-question self-heal so parent idle does not reject active child questions. Expected SafeGuard path: interactive metadata makes NoninteractiveChildRunProbe false, so SafeGuard/extension hooks should not auto-deny when TUI approval channel exists. Limitation: child HITL overlay currently requires the child live view to be active / polling that child; HITL outside live view remains follow-up work if needed. No Castor tests/gates run; php -l passed on 7 touched files.

## Task workflow update - 2026-07-01T00:24:07.906Z
- Summary: User manual smoke after eb1ad8eca: child HITL almost works and TUI is usable. Remaining UX issues: trouble going between main/live tabs especially using slash commands while live view is active (shortcut may work better); need better visibility on main screen/list that a child agent requires attention; /new while in live view ended the session/live state and user could not return to main; user asks whether cancellation is available in live view. These are next POC polish/fix candidates before considering productionization.

## Task workflow update - 2026-07-01T00:36:00.302Z
- Recorded fork run: 4uamdq3lyzd4
- Validation: php -l passed on SubagentLiveChildDTO.php, SubagentLiveCatalog.php, SubagentLiveViewState.php, SubagentLivePickerController.php, TickPollListener.php, SubmitListener.php, CancelListener.php, AgentsMainCommandHandler.php in worktree.; No Castor tests/gates run per ongoing POC manual loop.
- Summary: POC polish completed at commit 0f70f718d. Adds main-screen child attention visibility for waiting_human subagents via subagent_live status; /agents-live picker labels waiting child as needs input; live status hints mention /agents-main; blocks /new, /resume, /r while live view is active with a clear message to leave live view first; adds child-aware ESC cancellation in live view (guarded so active HITL overlay keeps question behavior); maps catalog waiting_human to childActivity WaitingHuman. Known limitations: /rename not blocked; attention depends on catalog/subagent_progress status updates; child HITL still primarily surfaced when live view/cursor polling path has observed child; no automated tests/Castor gates run.

## Task workflow update - 2026-07-01T01:21:27.656Z
- Summary: User reported fatal event-loop error after 0f70f718d: undefined method Ineersa\Tui\Listener\TickPollListener::syncSubagentAttentionStatus() from Symfony TUI ticker. Main bounded read confirmed TickPollListener calls self::syncSubagentAttentionStatus($state, $screen) around line 213, but rg shows no method definition; only shouldRejectOrphanedQuestion and other handlers exist. Need immediate fix fork to add missing private static method or remove call while preserving main-screen subagent attention status behavior.

## Task workflow update - 2026-07-01T01:22:37.180Z
- Recorded fork run: 1ndxe19zikyh
- Validation: php -l src/Tui/Listener/TickPollListener.php passed in worktree.; php -l src/Tui/Listener/SubmitListener.php passed in worktree.; php -l src/Tui/Listener/CancelListener.php passed in worktree.; No Castor tests/gates run per ongoing POC manual loop.
- Summary: Hotfix completed at commit b5c6fca29 for runtime fatal after 0f70f718d. TickPollListener called missing syncSubagentAttentionStatus(); fork added the missing helper to set/clear subagent_live status from firstChildNeedingAttention(). It also found and fixed three other missing helper definitions introduced by 0f70f718d: SubmitListener::isBlockedSessionSwitchWhileLiveView(), SubmitListener::showLiveViewSessionSwitchBlocked(), and CancelListener::shouldCancelSelectedChild(). Expected result: no immediate ticker fatal, /new live-view block no undefined helper fatal, and ESC child-cancel path no undefined helper fatal. No Castor tests/gates run; php -l passed on TickPollListener.php, SubmitListener.php, CancelListener.php.

## Task workflow update - 2026-07-18T16:54:01.078Z
- Summary: Manual fork testing found SafeGuard approval status UX bug: approval question renders and answer works, but child remains shown as Running instead of needs input. ask_human works because it commits RunStatus::WaitingHuman; SafeGuard intentionally uses transient tool_question.requested + blocked ToolQuestion store and leaves core RunStatus Running. TUI already optimistically calls SubagentLiveAttention::markChildNeedsInputForRun for selected child, but same-tick/stale parent subagent_progress status=running overwrites it because SubagentLiveCatalog only prevents terminal downgrades. Smallest fix is TUI attention latch: ignore stale Running progress while current catalog status is WaitingHuman; existing answer/cancel callbacks explicitly clear to Running. Add virtual TickPollListenerChildHitlTest proof: child tool question → needs input; stale running progress does not erase; answer clears. Do not convert SafeGuard to ask_human/domain WaitingHuman.

## Task workflow update - 2026-07-18T17:21:17.058Z
- Summary: SafeGuard child needs-input status bug has been split into focused TODO/fix-child-safeguard-needs-input-status.md. Keep the new focused task as owner of the generic TUI attention-latch correction; no implementation should be duplicated in this POC task.

## Task workflow update - 2026-07-18T23:06:45Z
- Cancelled as superseded by the production SUBAGENT-LIVE task series. Remaining focused SafeGuard and fork presentation issues are tracked separately. Worktree removed; branch retained for forensic history.
