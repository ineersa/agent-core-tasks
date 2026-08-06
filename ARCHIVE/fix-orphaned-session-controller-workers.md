# P0: Prevent orphaned Hatfield controller and worker processes per session

## Goal
Live RENDER-05B smoke revealed orphaned user-owned Hatfield controller/worker processes for session 1. After user believed no processes should be running, `ps` showed an old controller tree from DD_VERSION=6aed5b378 still alive in the RENDER-05B worktree with `HATFIELD_SESSION_ID=1`, sharing `run_control_1`, `llm_1`, `tool_1`, `agent_1`, and `mcp_1` queues. A newer controller from e7cd0c3e8 had also been launched earlier. The stale old `llm` consumer could steal ExecuteLlmStep messages from the active session queue and crash with `include(): zlib: data error`, leaving runs stuck in Working.

Observed stale process tree included PID 97272 `agent --controller` and child `messenger:consume` processes for tool, agent, scheduler_default, run_control, mcp, and llm, all user-owned, `HATFIELD_SESSION_ID=1`, `HATFIELD_CWD=/home/ineersa/projects/agent-core-worktrees/render-05b-transcript-lifecycle-hitl-overlay-polish`, `DD_VERSION=6aed5b378`. Root-owned PID 3231 is unrelated and must not be touched.

This is distinct from but compounds the stale-running recovery bug: orphaned controllers make lost in-flight LLM steps repeat by allowing old/stale workers to consume new messages from the same session queues.

## Acceptance criteria
- Controller lifecycle guarantees no orphaned `agent --controller` / `messenger:consume` child process tree remains after normal TUI/session exit.
- Starting/resuming a session detects an existing controller for the same session/worktree/queue ownership and either reuses it safely, refuses with a clear error, or terminates only its own stale user-owned prior tree through a documented safe shutdown protocol — never root-owned processes.
- Workers are tied to a controller ownership token/version so stale workers from an older controller cannot consume messages for a newly started controller on the same session queues.
- If a consumer exits unexpectedly, supervisor reports a visible runtime/protocol error and performs bounded cleanup; it must not silently leave the run in infinite Working.
- Add focused lifecycle tests proving controller shutdown cleans child consumers and duplicate controller/session ownership is rejected or recovered.
- Diagnostics expose active session workers and owner/version information without requiring manual `ps` spelunking.

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
- Created: 2026-07-01T18:51:29.515Z

## Task workflow update - 2026-07-01T18:56:37.847Z
- Summary: Reclassified after user clarification: the apparent orphaned 6aed5b378 controller tree was an old Hatfield instance still running over SSH/tmux, not an unexpected OS-orphan/product leak. After user stopped the SSH instance, only root-owned global worker remained and no user-owned session-1 controller/consumers were left. This task may be downgraded/closed or narrowed later to duplicate-controller ownership protection rather than orphan cleanup P0.

## Task workflow update - 2026-07-01T18:58:50.565Z
- Summary: Cancelled by user: not a real orphaned-process product bug in this incident. Root cause was an old Hatfield instance still running over SSH/tmux. After user stopped it, no user-owned session-1 controller/consumer processes remained. Do not implement as P0 from this incident; remaining issue is stale pending queued command/projection cleanup after cancel/rewind/resume, tracked under fix-stale-running-session-recovery-orphan-tool-messages. Attempted TODO→DONE move, but move_task failed with git status --porcelain error; leaving cancellation recorded in task metadata for now.
