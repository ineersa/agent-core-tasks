# Bound read-file offset validation for giant unselected lines

## Goal
Follow-up identified in the user's non-blocking review of PR #568 at63db29ea33acdd7fb9cda16e3bb1f500402a92d4.

ReadFileTargetValidator::rejectsOffsetPastEof() calls unrestricted fgets() while counting lines. A request such as offset=2,limit=3 can exhaust PHP memory while validating a huge unselected first line, before ReadFileTool's new bounded reader runs. Source inspection confirms the unrestricted call. The user reproduced validator-loop failure under memory_limit=128M using an isolated algorithm probe; this has not yet been reproduced through the repository validation pipeline.

The PR #568 memory regression invokes ReadFileTool directly and proves only that reader path. It does not cover pre-execution target validation. Keep this follow-up separate from the approved PR.

Entry points: src/CodingAgent/Tool/Validation/ReadFile/ReadFileTargetValidator.php; tests/CodingAgent/Tool/ReadFileToolTest.php; existing ToolValidationHarness and selected-range memory fixture.

Keep the change and regression coverage minimal. Reuse existing IO, validation, process, and test-isolation facilities. No new file-size limits, quotas, settings, public APIs, malformed-byte reproductions, or memory-limit increases.

## Acceptance criteria
- Count logical lines using bounded reads so a giant unselected line cannot create line-sized resident allocation during offset validation.
- Preserve early return once the offset is proven valid, exact past-EOF line counts and messages, existing newline semantics, operational-error policy, and handle cleanup.
- Add a focused regression through the real tool validation pipeline, using a giant valid-text first line and a small selected range under memory_limit=128M; build the isolated fixture with bounded writes and clean up owned resources.
- Run focused Castor validation in the task checkout. Attribute the user's isolated probe results separately from repository validation.

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
- Created: 2026-10-15T21:51:11+00:00

## Task workflow update - 2026-10-15T23:29:51+00:00
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Absorbed into PR568 at user's request, not left as separate follow-up. Implemented at861bf713eda3eeebdfbbb265dd58f5074a45ae0a: bounded offset-validation reads and existing memory regression strengthened through native validation pipeline. RED original128M OOM, GREEN10MiB peak on160MiB first line. Independent review approved; full Castor gate passed83.7s. Source task spec-06-byte-budgets-admission-and-backpressure is in CODE-REVIEW with PR568 open.
