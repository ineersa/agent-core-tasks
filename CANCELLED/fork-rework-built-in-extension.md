# FORK-REWORK Built-in fork extension architecture

## Goal
Supersedes the current FORK-02/FORK-03/FORK-04/FORK-05/FORK-06 implementation plan.

Decision: current PR #240 / FORK-03 implementation is a spike and is not mergeable as architecture. Current approach is too intrusive inside Hatfield core/runtime/TUI. Fork must be rebuilt as a built-in extension, modeled after Pi's fork extension.

Reference implementation:
- /home/ineersa/claw/my-pi/packages/extensions/extensions/fork/
- Primary files: fork.ts, runner.ts, runner-events.ts, session-result.ts, status-store.ts, tmux.ts, types.ts

New direction:
- Built-in fork extension owns fork tool, launcher, status/result storage, child-mode behavior, finalization, retrieval, and parent follow-up delivery.
- Hatfield core should expose only generic extension hooks/capabilities where needed.
- No fork-specific branches in AgentCommand, InteractiveMode, SessionInitializer, StartRunHandler, RuntimeEventEmitter, InProcessAgentSessionClient, or generic runtime/TUI code.
- Fork child should be a normal Hatfield process/session with extension-provided bootstrap/finalization behavior, similar to Pi extension's PI_FORK mode.
- Parent should observe extension-owned status/result artifacts and append completed result into parent conversation via existing follow-up/append-message capability.

Salvage candidates from prior FORK-02/FORK-03 work:
- snapshot sanitizer/compaction and prompt/handoff builders
- tmux launcher/status/result artifact learnings
- TmuxHarness E2E proof shape
- useful DTOs/enums only if they belong inside extension boundary

Prior tasks:
- FORK-02 is DONE but architecturally superseded; salvage only.
- FORK-03 PR #240 is spike/not mergeable; revert/remove intrusive runtime implementation.
- FORK-04/05/06 TODO scopes must be rewritten under this extension-first architecture.

## Acceptance criteria
- Architecture plan documents extension-first fork design and explicitly maps Pi fork extension concepts to Hatfield built-in extension equivalents.
- Inventory of missing generic extension hooks/capabilities is produced, with minimal API proposals and layer-boundary analysis.
- Current intrusive FORK-03 runtime/TUI/controller code is reverted or removed from the final implementation path.
- Built-in fork extension owns fork tool registration, snapshot storage, tmux launch, status/result artifacts, child finalization, retrieve, and parent follow-up delivery.
- Storage layout is extension-owned under .hatfield extension runtime dirs, with any parent-session linkage kept as a pointer/index rather than core fork lifecycle ownership.
- Validation plan includes focused unit/contract tests plus the lowest appropriate TUI/process proof, using Castor only.

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
- Created: 2026-06-30T02:24:31.847Z

## Task workflow update - 2026-06-30T02:28:17.062Z
- Summary: Scout planning completed with explicit model override deepseek/deepseek-v4-flash. Scout 1 analyzed Pi fork extension reference at /home/ineersa/claw/my-pi/packages/extensions/extensions/fork/: parent tool builds sanitized/compacted session snapshot, writes run status/result paths, launches tmux pane/script/log, polls exit marker/result artifact; child mode PI_FORK early-returns from tool registration, before_agent_start appends fork child prompt, agent_end writes result.json atomically and exits; background mode uses followUp user-message delivery. Scout 2 analyzed Hatfield extension API gaps: existing reusable capabilities are registerTool, prompt contributors, tool hooks, TUI commands/slots, exec, settings; missing/minimal generic capabilities likely needed are runtime/agent lifecycle hook(s), extension follow-up/send-message API, programmatic exit-after-run or child auto-exit hook, and extension-owned storage convention. Scout 3 produced salvage/revert inventory: remove current intrusive FORK-03 code from AgentCommand/InteractiveMode/SessionInitializer/StartRunHandler/RuntimeEventEmitter fork-specific finalization/ForkControllerStartService/ForkRunFinalizer/ForkAutoExitRegistrar; salvage FORK-02 prompt/context/sanitizer/compaction/handoff pieces into extension boundary. Important caveat: scouts proposed some APIs like forkRun() on ExtensionApi, but per user architecture direction this should be treated skeptically; prefer generic hooks/capabilities only, with fork behavior owned by built-in extension.

## Cancellation note - 2026-07-07
- Cancelled by user direction after SUBAGENT-LIVE-04 landed. Old fork/tmux/built-in-extension planning is superseded by the new actual fork-tool plan: implement fork as a thin model-facing tool over the production child-run/subagent backend, with no tmux launcher and no fork-specific generic runtime branches.
