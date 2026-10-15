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
Status: IN-PROGRESS
Branch: task/spec-06-byte-budgets-admission-and-backpressure
Worktree: /home/ineersa/projects/agent-core-worktrees/spec-06-byte-budgets-admission-and-backpressure
Fork run: 8bd94fcb-a6df-5cb6-aa84-680092b8a433
PR URL:
PR Status:
Started: 2026-10-15T20:52:22+00:00
Completed:

## Work log
- Created: 2026-10-02T20:34:36+00:00

## Task workflow update - 2026-10-15T20:52:07+00:00
- Summary: User-approved replacement scope: fix only (1) ReadFileTool loading the entire source file before applying offset/limit and (2) ConsumerStdoutPoller corrupting valid large JSONL events when split across pipe reads. This replaces the original broad Goal and acceptance criteria. Preserve read output/offset/limit/continuation semantics and valid event payload/order across split reads. Keep changes and tests minimal. No general byte accounting, aggregate admission system, disk reservations, new settings, process-memory increases, child transcript caps, transport redesign, or unrelated cleanup. Finish implementation with focused Castor validation; submission remains a separate phase.

## Task workflow update - 2026-10-15T20:52:22+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/spec-06-byte-budgets-admission-and-backpressure.
- Created worktree /home/ineersa/projects/agent-core-worktrees/spec-06-byte-budgets-admission-and-backpressure.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/spec-06-byte-budgets-admission-and-backpressure.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/spec-06-byte-budgets-admission-and-backpressure.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/spec-06-byte-budgets-admission-and-backpressure.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/spec-06-byte-budgets-admission-and-backpressure/.idea.
- Summary: Start the two user-approved fixes only; broad resource-budget specification superseded by the narrowed scope in the work log.

## Task workflow update - 2026-10-15T20:52:46+00:00
- Summary: Single implementation fork owns the task worktree and both focused corrections. Main completed routing against the entry points, caller/validation boundaries, and existing tests. No other writer; fork run ID will be recorded on return.
- Ownership: owner=fork; fork_run=none; revision=885bfec6d; scope=two approved fixes: stream requested file range without whole-file retention and preserve split large consumer JSONL events; outcome=assigned; commit=none

## Task workflow update - 2026-10-15T21:08:43+00:00
- Validation: Castor focused ReadFileToolTest|ConsumerStdoutPollerTest|ConsumerSupervisorTest: 51 tests,165 assertions PASS (fork).; Castor phpstan/deptrac/dead-code/changed-path cs-check/docs:validate PASS (fork).; Independent PHP process under128M: ~92MiB valid-text fixture, exact3-line selection, peak_bytes=before_bytes=6291456 (fork).
- Summary: Implementation fork completed both approved fixes at 63c69ed5539a24352ca8e372b4dfd439d1db076e. Main inspecting changes before accepting implementation. Fork confirmed testing skill and tests/AGENTS prerequisite reads. No submission, full gate, or independent review yet.
- Ownership: owner=fork; fork_run=agent_dc9004f765ab3070; revision=885bfec6d; scope=two approved fixes: streaming file range and split large consumer JSONL preservation; outcome=completed; commit=63c69ed5539a24352ca8e372b4dfd439d1db076e

## Task workflow update - 2026-10-15T21:10:45+00:00
- Recorded fork run: 8bd94fcb-a6df-5cb6-aa84-680092b8a433
- Summary: Main review found draft corrections needed: JSONL regression accumulated only60,000 bytes before final newline, so did not exercise former65,536-byte truncation; selected-line rtrim strips more CR bytes than original file(FILE_IGNORE_NEW_LINES); IO false/empty handling should not silently treat non-EOF read failure as success; accidental read tool description change is outside scope. Request simpler bounded native line skipping rather than per-line overread/rewind.
- Ownership: owner=fork; fork_run=8bd94fcb-a6df-5cb6-aa84-680092b8a433; revision=63c69ed5539a24352ca8e372b4dfd439d1db076e; scope=bounded corrections to regression coverage and file read semantics/error handling in same task worktree; outcome=assigned; commit=none

## Task workflow update - 2026-10-15T21:16:31+00:00
- Recorded fork run: 8bd94fcb-a6df-5cb6-aa84-680092b8a433
- Validation: Main: castor test --filter='ReadFileToolTest|ConsumerStdoutPollerTest|ConsumerSupervisorTest': PASS 51 tests/167 assertions; no QA leaks.; Main: independent read probe archive_bytes=92250021 (~88MiB), peak_bytes=before_bytes=6291456 (6MiB) under memory_limit=128M; exact three-line slice and continuation notice.; Fork: baseline poller + supervisor regression RED, corrected poller GREEN with70,000-byte incomplete prefix crossing65,536-byte old cutoff.; Fork: phpstan,deptrac,dead-code,cs-check PASS at63db29ea3; docs:validate PASS on unchanged docs at63c69ed55.; Main: final diff inspection and git diff --check PASS; full Castor gate/independent review not yet run.; Castor automatically rebuilt local PHAR for63db29ea3; all packaged smoke checks PASS.
- Summary: Both narrowed fixes implemented at 63db29ea33acdd7fb9cda16e3bb1f500402a92d4. Main inspected final diff and reran focused Castor suite successfully. ReadFileTool retains only requested lines and skips unselected text in bounded native fgets chunks, preserving file() newline semantics and exact continuation counts. ConsumerStdoutPoller preserves incomplete JSONL without destructive truncation; old constant and obsolete docs removed. Corrected regression demonstrably fails baseline and passes revised poller. Implementation phase complete; task remains IN-PROGRESS pending separate independent review and CODE-REVIEW gate. No new quotas/settings, child-view changes, pushes, or PR.
- Ownership: owner=fork; fork_run=8bd94fcb-a6df-5cb6-aa84-680092b8a433; revision=63c69ed5539a24352ca8e372b4dfd439d1db076e; scope=bounded corrections to regression coverage and file read semantics/error handling in same task worktree; outcome=completed; commit=63db29ea33acdd7fb9cda16e3bb1f500402a92d4
