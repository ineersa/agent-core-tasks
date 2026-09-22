# Audit bg_status process list for non-background entries

## Goal
Dogfood follow-up: `bg_status` list currently shows many entries. Investigate whether every listed record represents a command that was actually moved to or launched in the background, versus foreground Bash calls, stale records, unrelated processes, or terminal historical entries.

Start with read-only evidence from current session/background-process storage and trace the list path from persistence through the `bg_status` tool. Do not assume whether completed background jobs should remain visible; document current semantics and ask for a product decision if changing history/retention behavior would alter the user-facing contract. If clearly incorrect non-background records are included, implement the smallest owning-boundary fix with regression coverage.

## Acceptance criteria
- Capture a reproducible `bg_status list` sample and correlate each representative entry to its originating Bash execution, background transition, persisted status, and owning session/run.
- Trace and document which persistence query/filter and status mapping populate `bg_status list`.
- Verify whether foreground-only Bash executions are being persisted or returned as background jobs, and whether stale, cancelled, timed-out, failed, completed, or cross-session records are included.
- Distinguish an actual filtering bug from an intentional history/retention policy; do not invent new retention or visibility semantics without user confirmation.
- If a defect is confirmed, ensure `bg_status list` excludes records that were never actually backgrounded while preserving valid background-job inspection and stop behavior.
- Add focused regression coverage for the confirmed contract and run Castor validation appropriate to any changed runtime/process-lifecycle code.
- Never signal or modify root-owned processes or processes tagged with `HATFIELD_SESSION_ID` during investigation or tests.

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
- Created: 2026-08-21T20:49:45+00:00

## Task workflow update - 2026-08-25T18:44:01.709Z
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Superseded by finalized storage/lifecycle task STORAGE-bound-background-process-storage-to-active-runs. User has now made the product decision that was previously open: only processes actually transitioned to background may appear; storage/listing is scoped to the currently active run; run end must stop owned background processes and clear that run's tmp/bg artifacts. The old read-only audit task is no longer the correct scope.
