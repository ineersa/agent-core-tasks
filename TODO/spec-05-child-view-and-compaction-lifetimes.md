# SPEC- Bound child-state, view, and compaction lifetimes

## Goal
Implement Change 5 lifetime requirements from reports/implementation-specification.md, sections 7 and 10. Depends on SPEC-02 registry ownership, SPEC-03 durable terminal handoffs, and SPEC-04 replay products. Coordinate admission and byte limits with SPEC-06.

Persist bounded terminal handoff and pending parent-delivery evidence before unloading completed children. Parent acknowledgement must not be required to retain the child's execution graph. Enumerate nonterminal children and outstanding handoffs in pages on recovery.

Child views build transcript-only products from stable cuts, without execution recovery or shared transcript caches. Retain only parent plus one visible child view. Release readers, blocks, widgets, ASTs, listeners, callbacks, and worker-owned accumulators on exit or terminal handoff. Audit compaction ownership through the legitimate old-plus-new transition lifetime.

Keep the user's existing terminal-handoff cleanup decision for immutable launch inputs; do not introduce a new startup acknowledgement protocol.

## Acceptance criteria
- Completed children leave the registry before parent result consumption; restart delivery uses durable handoff without rehydrating completed execution state.
- Child view enter/exit/re-entry executes no tools and performs no execution recovery.
- Weak-reference probes prove obsolete context and view graphs become unreachable after their legitimate lifetime.
- Repeated compactions and increasing completed-child counts show bounded quiescent memory for equal active workloads.
- Existing worker reset facilities clear non-owned service/ORM accumulators; recovery/catalog reads are paged.
- Failed terminal finalization retains necessary durable evidence and backpressures new admission rather than accumulating completed graphs.

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
- Created: 2026-10-02T20:34:15+00:00
