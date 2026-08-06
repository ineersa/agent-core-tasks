# FORK-04 Tmux-backed fork process launcher and run registry

## Goal
Implement the parent-side process launcher and registry for Pi-style fork v1.

Reference docs/sources:
- Plan: `/home/ineersa/projects/agent-core/.aiassistant/fork/plan.md`
- Pi tmux/process launch flow: `/home/ineersa/claw/my-pi/packages/extensions/extensions/fork/runner.ts`, `/home/ineersa/claw/my-pi/packages/extensions/extensions/fork/tmux.ts`, `/home/ineersa/claw/my-pi/packages/extensions/extensions/fork/status-store.ts`

Design decisions to preserve:
- Fork v1 launches a separate Hatfield instance in tmux, not Messenger queue/worker and not integrated TUI tabs.
- Parent tool returns immediately after spawning.
- V1 may allow only one active fork per parent session/cwd, but design must not block later max-3 parallel forks.
- Parent owns the registry entry/artifact directory; child writes into the path passed by parent.

## Acceptance criteria
- Adds `ForkProcessLauncher` or equivalent that creates a tmux pane/window and starts the child Hatfield process with snapshot/result paths and fork env/args.
- Adds `ForkRunRegistry` or equivalent storing runId, parentRunId, status, pid/tmux metadata, level/model, cwd, task, timestamps, artifact path, and error state.
- Implements v1 active-fork guard (one active fork per parent session/cwd) with structure compatible with later max-3 concurrency.
- Handles tmux unavailable/spawn failure with clear structured errors and registry status updates.
- Does not use Messenger worker as the primary fork execution mechanism.
- Focused Castor validation via Castor only, with process launcher tests using mocked/trapped spawn/tmux where practical.

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
- Created: 2026-06-29T16:12:20.278Z

## Task workflow update - 2026-06-29T23:33:44.471Z
- Summary: Reference implementation path: Pi fork extension at `/home/ineersa/claw/my-pi/packages/extensions/extensions/fork/` (primary `fork.ts`; related `runner.ts`, `status-store.ts`, `tmux.ts`, `types.ts`). Use as source of truth for tmux launcher/registry/status behavior.

## Task workflow update - 2026-06-30T02:17:00.998Z
- Summary: ARCHITECTURE RESET: Existing FORK-04 scope must be rewritten. Fork launcher/registry should be owned by a built-in fork extension modeled after /home/ineersa/claw/my-pi/packages/extensions/extensions/fork/ (tmux launcher, run status, result artifact, background/wait behavior), not core Hatfield runtime. Salvage current launcher learnings only.

## Cancellation note - 2026-07-07
- Cancelled by user direction after SUBAGENT-LIVE-04 landed. Old fork/tmux/built-in-extension planning is superseded by the new actual fork-tool plan: implement fork as a thin model-facing tool over the production child-run/subagent backend, with no tmux launcher and no fork-specific generic runtime branches.
