# Reduce full-history memory during session replay

## Goal
Follow-up to PR #469. Compaction-window retention reduces retained transcript state, but resume/startup, history selection, and child-view entry still temporarily decode full canonical event history.

Inspect these replay paths and reduce temporary full-history materialization using existing event-store iteration facilities where possible. Keep this separate from the current retention change. Preserve canonical events and existing replay, history-selection, and pending-question behavior. Measure peak memory on a large multi-compaction session before and after; do not claim bounded memory unless demonstrated.

## Acceptance criteria
- Identify full-history allocations in resume, history selection, and child-view snapshot paths.
- Reduce avoidable temporary event collections without changing replay results or persisted history.
- Record before/after peak-memory evidence and remaining limits; add only focused regression coverage for changed contracts.

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
- Created: 2026-09-05T22:32:59+00:00
