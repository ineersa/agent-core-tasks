# SPEC- Complete integrated recovery and memory acceptance

## Goal
Close the acceptance matrix and done criteria in reports/implementation-specification.md, sections 13–14, after SPEC-02 through SPEC-06. This task integrates and audits evidence; prerequisite tasks must still carry their own focused regression tests.

Use real archives, configured serialization/storage, separate owned processes for crash/ownership proofs, and packaged application runs for memory. Cover every specification scenario, including long waits, completed-child accumulation, handoff restart, child-view disposal, successive compactions, oversized input, slow/disconnected TUI, every crash boundary, duplicate deliveries, unknown execution, lost result notifications/ACKs, delayed results after rewind, reused turn numbers, sequence holes, cache/index deletion, storage failure, and pending-human-input attach.

Measure physical archive reads, decoded-record peak, retained contexts/views, IPC pending bytes, durable spool use, and operation states. Compare equal active-window workloads as archived events, distinct turns, completed children, and progress revisions increase. Keep current process limits. Record intentional explicit-history/rebuild/suffix-read exceptions.

## Acceptance criteria
- Every acceptance-matrix row maps to concrete passing proof or a clearly reported blocker; no blanket completion based only on unit tests.
- Ordinary getters perform zero archive reads, full-graph serialization, or cache writes.
- Packaged repeated-compaction and completed-child tests show memory plateaus and actual obsolete-object release.
- Crash tests prove no automatic repeat of uncertain tools and no loss of already-durable results.
- Independent reference history checks cover adversarial selection/discard/compaction and late results.
- Tests follow deterministic synchronization, process ownership, teardown, and duration rules; no sleeps/retry-until-green or raised process limits.
- Independent specification-fidelity review and full workflow gate pass; final report distinguishes verified guarantees from remaining limitations.

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
- Created: 2026-10-02T20:34:59+00:00
