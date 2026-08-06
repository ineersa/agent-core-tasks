# Fork TUI: start visible transcript at delegated task

## Goal
Follow-up UX task from FORK-MVP-03 testing. Fork children should retain inherited/compacted parent context for the model, but the fork child TUI should not render the inherited prior conversation. The visible transcript should begin with the actual delegated fork task. This is a presentation/projection concern only and must not change snapshot sanitization, compaction, model-visible context, handoff generation, or ordinary subagent behavior.

## Acceptance criteria
- Fork child model still receives the same sanitized/compacted inherited context
- Fork child TUI transcript hides inherited prior history and begins with the delegated task
- Behavior is keyed by typed fork child/session metadata, not agent-name strings
- Ordinary parent and subagent transcript rendering remains unchanged
- Add automated proof at the lowest correct TUI layer per tests/AGENTS.md

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
- Created: 2026-07-18T16:43:30.231Z

## Task workflow update - 2026-08-04T20:22:53.655Z
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Cancelled at user request; no implementation was started.
