# SPEC- Implement indexed replay and coordinated parent bootstrap

## Goal
Implement Change 4 from reports/implementation-specification.md, sections 4 and 8. Integrate with SPEC-02 owner registry and SPEC-03 committed-cut/recovery protocol.

Build a disposable run-local scalar history index with actual event offsets/sequences, immutable turn anchors, selection/discard metadata, and checkpoint locations, not event bodies or full states. Missing or invalid indexes use bounded two-pass reconstruction with unresolved references on disk. Preserve sequence holes, reused displayed turn numbers, queued-command semantics, and operational metadata preceding compaction.

One owner-led replay coordinator constructs execution state and bounded bootstrap products. Transfer a sealed private spool through bounded frames; distinguish runtime.ready from session.ready, mount at a validated cut, require matching bootstrap acknowledgement, and catch up from bounded archive suffixes. Controller/TUI must not independently reconstruct execution state. Coordinate budgets with SPEC-06.

## Acceptance criteria
- Valid index enables coordinated selected-record replay; missing/corrupt index triggers explicit bounded two-pass rebuild without all-turn snapshots.
- History selection/discard and reused turn-number behavior match established fixtures and an independent reference implementation.
- Readers honor actual committed offsets and sequences, not allocation counters or file size.
- Bootstrap validation, stale ID/epoch rejection, acknowledgement cleanup, cancellation, and suffix catch-up are tested.
- Slow/disconnected TUI cannot hold owner transition locks or cause unbounded suffix buffering.
- Existing attach and human-input policy remains intact; attach alone does not start a model turn.
- Physical archive bytes and read reasons are measured honestly; duplicate reconstruction entry points are removed.

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
- Created: 2026-10-02T20:33:54+00:00
