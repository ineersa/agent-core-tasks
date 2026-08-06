# DRY synchronous snapshot compaction orchestration

## Goal
Follow-up approved during FORK-MVP-03 review. `CompactionService::compactMessages()/doCompactMessages()` correctly reuses `SessionCompactor` for partitioning, boundaries, prompt construction, and message assembly, but duplicates substantial orchestration from the canonical async `/compact` path: runtime/model settings resolution, hook orchestration, no-tools platform invocation, exception/empty-summary handling, and ineffective-summary validation. Keep synchronous immutable snapshot compaction as a supported CompactionService capability, but DRY shared execution against existing compaction worker/services. This is separate from fork cleanup and separate from the broader AgentCore ownership investigation.

## Acceptance criteria
- Synchronous `compactMessages()` remains an in-memory, non-mutating API and preserves current fork semantics.
- SessionCompactor remains the single partition/boundary/message-assembly implementation.
- No-tools model invocation and pure summary validation are shared with the canonical async compaction path rather than copy-pasted.
- No temporary sessions, prelaunch lifecycle, Messenger polling, fork-specific compactor, lazy nullable DI, or custom summarization algorithm is introduced.
- Async `/compact` event/state behavior remains unchanged; snapshot ineffective-compaction remains a structural no-op unless separately approved.
- Focused Castor compaction tests, deptrac, phpstan, cs-check, and full castor check pass before CODE-REVIEW.

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
- Created: 2026-07-18T18:33:07.944Z
