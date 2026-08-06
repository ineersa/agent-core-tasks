# FORK-NEW-01 Discovery: fork as subagent mode after live-view productionization

## Goal
Discovery/design task to run only after SUBAGENT-LIVE-01/02/03/04 productionize child live view, steering, HITL, cancellation, and validation. This supersedes the tmux-first fork direction for future architecture planning, but does not itself implement fork.

Context/decision:
- User does not like tmux-backed fork: it breaks headless/no-tmux usage and duplicates runtime lifecycle.
- POC proved normal subagent child runs can provide live view, steering, HITL, cancellation, and parent attention.
- New likely direction: implement `fork` as a specialized subagent/child-run mode, not as separate tmux process.
- Keep a thin user-facing `fork` tool if useful, but backend should be generic child/subagent execution.
- If levels/model selection are not needed initially, fork can use current model and behave as a subagent mode with fork prompt/tool policy.

Reference material:
- Production plan: `.aiassistant/fork/subagent-live-view-production-plan.md`.
- POC reference kept alive by user: `/home/ineersa/projects/agent-core-worktrees/subagent-live-view-steering-poc`, branch `task/subagent-live-view-steering-poc`, latest known head `b5c6fca29`.
- Older fork planning docs under `.aiassistant/fork/` contain salvageable prompt/handoff/context ideas but tmux/process-launcher direction is no longer preferred.
- Older fork tasks FORK-04/05/06 and fork-rework-built-in-extension should be treated as stale/superseded inputs, not direct implementation instructions.

Discovery questions:
1. What generic child-run/subagent backend APIs are needed so both regular subagents and fork can use the same machinery?
2. Should `fork` be a separate thin tool over child-run mode, or a `subagent(type=fork)`/agent preset? Current preference: separate thin `fork` tool for model-facing clarity, shared backend internally.
3. Is background/non-blocking child execution needed before fork MVP, or can fork initially share current subagent blocking behavior plus live view? Desired end-state is background child run with completion follow-up.
4. How should fork result validation/repair integrate with subagent child runs?
5. How should `fork_retrieve(run_id)` map onto agent artifact registry/retrieval?
6. How should model selection work: current model only, optional level config, or both?
7. What tool policy disables nested fork/subagent access for fork children?
8. What storage/metadata distinguishes `AgentArtifactKindEnum::Fork` vs `Subagent`?

Likely fork-as-subagent shape:
- `fork(task, level?)` -> child-run execution service with kind `Fork`.
- Prompt: fork handoff prompt / 11-section schema (salvage from FORK-02/Pi fork planning as needed).
- Tools: no `fork`, no `subagent` by default to prevent cheating/delegation loops.
- Model: current model by default; optional level-based resolver can be added as sugar.
- Runtime: same child run/event/artifact infrastructure as production subagent live view.
- UX: same `/agents-live` view can inspect/steer/cancel fork children.
- Completion: eventual `[FORK_DONE]` follow-up into parent; retrieve via `fork_retrieve` over artifact registry.

Out of scope:
- No tmux process launcher.
- No separate child TUI process requirement.
- No implementation until subagent live view/control is productionized.
- No broad extension API rework unless discovery proves it is still needed.

Important caveats inherited from subagent backend:
- Steer is not interruptive; cancellation is separate.
- Long-running tools need reliable cancel semantics.
- Background execution may require worker/control-loop changes if current SubagentExecutionService blocks too much.
- HITL ownership must already be productionized before fork relies on it.
- Event buffering and child run registration must be robust before fork uses live view.

## Acceptance criteria
- Discovery starts only after SUBAGENT-LIVE production tasks have established stable child live view, steering, HITL, and cancellation primitives.
- Produce an architecture decision record comparing: thin `fork` tool over child-run mode vs `subagent(type=fork)` vs old tmux/process launcher.
- Inventory salvageable fork pieces from FORK-02/FORK-03 planning/code (prompt builder, context builder, handoff validator, DTOs, artifact kind) and list what should be discarded.
- Define minimal fork MVP API and behavior with no tmux dependency and headless compatibility.
- Define model selection plan: current model by default, optional level resolver if still desired.
- Define fork child tool policy preventing nested fork/subagent access.
- Define completion/retrieve plan over existing artifact registry and future background child-run completion follow-up.
- End with concrete implementation task breakdown; do not implement fork in this discovery task.

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
- Created: 2026-07-01T16:56:23.546Z

## Cancellation note - 2026-07-07
- Cancelled by user direction after SUBAGENT-LIVE-04 landed. Old fork/tmux/built-in-extension planning is superseded by the new actual fork-tool plan: implement fork as a thin model-facing tool over the production child-run/subagent backend, with no tmux launcher and no fork-specific generic runtime branches.
