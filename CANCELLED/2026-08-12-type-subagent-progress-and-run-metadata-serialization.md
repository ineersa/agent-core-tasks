# Replace subagent progress array serialization and defensive metadata parsing with typed Symfony serialization

## Goal
Follow-up from PR #373 review. Audit the array-based `SubagentChildProgressSummary::toProgressFields()` payload construction and repeated defensive `RunStarted` metadata parsing in `SubagentChildProgressSummaryBuilder` / deferred projection. Replace hand-written array serialization/parsing with existing typed DTOs and Symfony Serializer where the runtime boundary permits, rather than adding more local walkers. Keep runtime event compatibility and architecture boundaries explicit; determine whether `RunMetadata` can remain typed through the event/projection path instead of degrading to arrays.

## Acceptance criteria
- Map the current RunMetadata → RunStarted event → subagent progress projection serialization/deserialization flow and identify the actual boundary requiring normalization.
- Use Symfony Serializer/Normalizer and typed DTOs at the boundary; remove redundant manual payload walkers and defensive scalar checks where typed deserialization guarantees them.
- Do not introduce a second parallel representation or compatibility shim during active development.
- Update focused runtime/projection tests and run Castor test, deptrac, phpstan, and cs-check.
- Document any boundary that must remain array-shaped and why.

## Workflow metadata
Status: CANCELLED
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-08-12T22:05:21.419Z

## Task workflow update - 2026-08-12T22:09:08.424Z
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Superseded before implementation: owner clarified this is not a subagent-progress-only issue. The replacement task must audit the entire codebase for internal array-shaped data flows, defensive `is_array`/`isset`/scalar parsing, and missed Symfony Serializer/value-object/DTO usage.
