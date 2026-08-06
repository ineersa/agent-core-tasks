# QH-08 Resume pending HITL question from session replay (DEFERRED — not v1)

## Goal
Plan: .pi/plans/tui-question-hitl-plan.md
Related plan: .pi/plans/runtime-transcript-vertical-slice-plan.md

**Decision (2026-06-28): Deferred. Not required for v1.**

If a session is resumed while a run is still `WaitingHuman`, the TUI will not restore
the question widget. The run remains paused. The user must cancel or rerun.

Scope (deferred):
- On resume, rebuild transcript blocks and active HITL question state from runtime events/session state.
- If latest run is still waiting for human input, show the pending question widget again.
- Ensure answer after resume still sends answer_human and continues the run.
- Do not restore local TUI questions.

Exclusions:
- Do not implement local question persistence.
- Do not implement fork/branch replay.

Dependencies: (none for v1; if picked up later, depends on QH-07-equivalent and RTVS-08)
Parallelizable with: none (deferred).

## Acceptance criteria (if picked up later)
- Resume while waiting shows the pending HITL question again.
- Answer after resume sends answer_human and continues the run.
- Local questions are not restored.
- Replay tests cover pending HITL reconstruction.
- castor deptrac passes.

## Workflow metadata
Status: DONE
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed: 2026-06-28T16:26:21+00:00

## Work log
- Created: 2026-05-18T00:04:55.029Z
- Closed: 2026-06-28T16:26:21+00:00 — Closed/deferred out of v1 scope. Thin ask_human v1 will not restore pending HITL overlays on resume; future support should be tracked as a new task if needed.
