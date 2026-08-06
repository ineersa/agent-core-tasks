# FORK-06 Fork v1 E2E hardening, docs, and gate validation

## Goal
Harden the tmux-backed fork v1 end-to-end after FORK-02 through FORK-05 land.

Reference docs/sources:
- Plan: `/home/ineersa/projects/agent-core/.aiassistant/fork/plan.md`
- Existing settings docs: `docs/settings.md`
- Session storage docs: `docs/session-storage.md`
- Pi fork docs/sources: `/home/ineersa/claw/my-pi/packages/extensions/extensions/fork/`

Design decisions to preserve:
- Fork v1 is tmux-backed separate Hatfield instance, not integrated interactive tabs.
- Readonly subagent detail tabs remain separate future feature, not part of fork v1.
- Handoff validation/repair is mandatory to prevent corrupted handoffs after steering.
- `fork_retrieve` remains the model-facing retrieve API while sharing backend retrieval infrastructure.

## Acceptance criteria
- Adds/updates user-facing docs/settings for fork config, level model mapping, tmux requirement, artifact behavior, and `fork_retrieve`.
- Adds end-to-end proof for the fork lifecycle at the lowest correct layers: launch child, write valid handoff, retrieve result, deliver `[FORK_DONE]`, and invalid-handoff repair/failure path.
- Verifies cleanup/lifecycle: no leaked child processes/workers from tests; no root-owned workers touched.
- Runs focused Castor validation and full `castor check` gate when prerequisites are available, using Castor only.
- Records any remaining limitations explicitly (parallel fork max-3 future target, OM virtual compaction caveats, tmux availability).

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
- Created: 2026-06-29T16:12:40.197Z

## Task workflow update - 2026-06-29T23:33:44.455Z
- Summary: Reference implementation path: Pi fork extension at `/home/ineersa/claw/my-pi/packages/extensions/extensions/fork/` (primary `fork.ts`; related `runner.ts`, `runner-events.ts`, `session-result.ts`, `status-store.ts`, `tmux.ts`, `types.ts`). Use as source of truth for final E2E/docs/hardening expectations.

## Task workflow update - 2026-06-30T02:17:01.005Z
- Summary: ARCHITECTURE RESET: Existing FORK-06 scope must be rewritten around built-in fork extension architecture. E2E/docs should validate extension-owned fork lifecycle and minimal generic hooks, not intrusive core ForkRunFinalizer/ForkControllerStartService style.

## Cancellation note - 2026-07-07
- Cancelled by user direction after SUBAGENT-LIVE-04 landed. Old fork/tmux/built-in-extension planning is superseded by the new actual fork-tool plan: implement fork as a thin model-facing tool over the production child-run/subagent backend, with no tmux launcher and no fork-specific generic runtime branches.
