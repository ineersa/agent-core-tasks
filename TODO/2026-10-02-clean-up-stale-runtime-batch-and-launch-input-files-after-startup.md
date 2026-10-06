# Clean up stale runtime batch and launch-input files after startup

## Goal
Follow-up to PR #545. User requests reusing the existing output-cap cleanup approach and scheduler facilities where applicable, rather than adding another cleanup framework. Inspect those facilities before selecting the implementation.

Normal cleanup already runs after canonical ToolBatchCommitted, terminal AgentEnd, and registered child handoff. This follow-up should reclaim leftovers from failed deletion or a crash between commit and cleanup without delaying startup.

Prefer a bounded, idempotent background cleanup pass triggered or scheduled at startup using existing facilities. Establish eligibility from durable lifecycle evidence and coordinate with active owners. File age alone is not permission to delete. Preserve inputs and batch coordination needed by pending execution, approval waits, supported redelivery, or recovery. Leave uncertain cases alone. Reuse existing storage/path resolution and cleanup operations. Do not add TTL-based invalidation, a new recovery protocol, or broad directory deletion based only on apparent inactivity.

Keep this separate from PR #545. Investigation should determine how the output-cap cleanup and scheduler can be reused and identify the smallest safe scope, initially leftovers from terminal runs if supported by existing ownership checks.

## Acceptance criteria
- Document the existing output-cap cleanup and scheduler entry points and reuse the applicable facilities.
- Cleanup runs outside the startup critical path and is bounded and idempotent.
- Durable eligibility checks and ownership coordination prevent deletion during resume or while execution, approval, redelivery, or recovery still requires the files.
- Reclaim eligible leftover tool-batch snapshots, launch-input files, and relevant staging files without scanning full canonical archives during ordinary startup.
- Deletion failures emit sanitized structured logs without blocking startup or completion delivery.
- Deterministic tests prove eligible cleanup, retention of active or uncertain work, repeated cleanup, and coordination with concurrent ownership. Validate through Castor.

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
- Created: 2026-10-02T18:23:02+00:00
