# FORK-05 fork/fork_retrieve tools, completion watcher, and follow-up

## Goal
Wire the user/model-facing fork API over the launcher, registry, and artifact backend.

Reference docs/sources:
- Plan: `/home/ineersa/projects/agent-core/.aiassistant/fork/plan.md`
- Pi fork tool/retrieve/follow-up: `/home/ineersa/claw/my-pi/packages/extensions/extensions/fork/fork.ts` (`fork`, `fork_retrieve`, background follow-up)
- Existing Hatfield retrieve pattern: `src/CodingAgent/Agent/Tool/AgentRetrieveTool.php`
- Existing background completion pattern: `src/CodingAgent/Runtime/Controller/BackgroundProcessCompletionPoller.php`

Design decisions to preserve:
- `fork` params: `task` required, `cwd?`, `level?` (`junior|middle|senior`, default `middle`). No `background`, `model`, or `thinking` params in v1.
- `fork_retrieve(run_id)` remains fork-specific model-facing API for Pi compatibility/clarity, but should reuse existing agent/artifact retrieval backend internally.
- Completion follow-up uses append-message/new parent turn with `[FORK_DONE]` so parent actively incorporates the result.
- Parent should be told to continue working after launching a fork.

## Acceptance criteria
- Registers permanent `fork` and `fork_retrieve` tools with clear LLM-facing descriptions, including level-selection guidance.
- `fork` validates params, builds sanitized context via FORK-02, creates artifact/registry entry, launches child via FORK-04, and returns run ID immediately.
- `fork_retrieve(run_id)` resolves only valid fork runs for the current parent context and returns status, metadata, accepted handoff, or diagnostics for failed/cancelled/invalid-handoff runs.
- Adds completion watcher/notifier that detects child completion/exit/artifact readiness, finalizes registry status, and sends `[FORK_DONE]` append-message/follow-up to the parent run.
- Completion notification is idempotent and does not deliver duplicate `[FORK_DONE]` messages.
- Focused Castor validation via Castor only, including retrieval and notification contract tests.

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
- Created: 2026-06-29T16:12:30.427Z

## Task workflow update - 2026-06-29T23:33:44.442Z
- Summary: Reference implementation path: Pi fork extension at `/home/ineersa/claw/my-pi/packages/extensions/extensions/fork/` (primary `fork.ts`; related `runner-events.ts`, `session-result.ts`, `status-store.ts`, `types.ts`). Use as source of truth for fork tool, fork_retrieve, parent polling/completion, and result append/follow-up behavior.

## Task workflow update - 2026-06-30T02:17:00.988Z
- Summary: ARCHITECTURE RESET: Existing FORK-05 scope must be rewritten. fork/fork_retrieve tools and parent completion watcher should live in built-in fork extension. Parent should poll/watch extension-owned result artifact and append result via existing follow-up/append-message capability. Do not add fork-specific runtime branches to generic clients/handlers.

## Cancellation note - 2026-07-07
- Cancelled by user direction after SUBAGENT-LIVE-04 landed. Old fork/tmux/built-in-extension planning is superseded by the new actual fork-tool plan: implement fork as a thin model-facing tool over the production child-run/subagent backend, with no tmux launcher and no fork-specific generic runtime branches.
