# SPEC- Enforce byte budgets, admission, and transport backpressure

## Goal
Implement specification section 9 and related lifetime limits from reports/implementation-specification.md. Coordinate with SPEC-02 through SPEC-05; add limits at each owning boundary rather than waiting for a final global clamp.

Cover per-run and aggregate contexts, admitted waiting children, model-visible tool results, transcript/render products, IPC frames and incomplete lines, pending peer bytes, archive batches/records, staged transitions, runtime payload storage, and bootstrap spools. Count and structural limits supplement byte limits. Reference envelopes must account for resolved payload budgets.

Treat the specification's numerical values as initial engineering defaults requiring packaged measurement and explicit configuration review, not established safe limits. Do not increase PHP limits to pass acceptance. Reuse existing output limiting, framing, storage, and settings facilities; no unapproved new settings or arbitrary silent truncation.

## Acceptance criteria
- Admission reserves peak headroom before constructing or queuing additional full contexts, including old-plus-new compaction state.
- Context accounting avoids repeatedly serializing whole runs and includes metadata, reasoning, tool arguments/results, and attachment references.
- Oversized mandatory context fails explicitly before dispatch when safe compaction cannot fit it; assistant/tool groups are not silently severed.
- Tool output is bounded during capture, and frames/line buffers are bounded before decode.
- Slow consumers coalesce transient data or resynchronize from durable cursors; durable results are never dropped.
- Disk exhaustion applies backpressure rather than deleting unfinished requests/results; reserve result capacity before external execution.
- Packaged probes validate heap amplification and structural bounds at existing process limits.

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
- Created: 2026-10-02T20:34:36+00:00
