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
Status: CODE-REVIEW
Branch: task/spec-06-byte-budgets-admission-and-backpressure
Worktree: /home/ineersa/projects/agent-core-worktrees/spec-06-byte-budgets-admission-and-backpressure
Fork run: 8bd94fcb-a6df-5cb6-aa84-680092b8a433
PR URL: https://github.com/ineersa/agent-core/pull/568
PR Status: open
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

## Task workflow update - 2026-10-15T21:20:19+00:00
- Summary: Submission phase started at user request. Assigning independent read-only reviewer for revision63db29ea33acdd7fb9cda16e3bb1f500402a92d4, base885bfec6d. Scope is only streaming requested file range and lossless split consumer JSONL preservation; require specification-fidelity review against latest narrowed user-approved scope, not superseded broad task Goal.
- Review assignment: role=independent reviewer; target=63db29ea33acdd7fb9cda16e3bb1f500402a92d4; scope=all seven changed files, correctness/test value/specification fidelity; artifact/run ID to be recorded on return.

## Task workflow update - 2026-10-15T21:27:14+00:00
- Validation: Independent reviewer: Castor three new regressions PASS3tests/21assertions; no QA leaks.; Main same-revision focused Castor suite PASS51tests/167assertions; strict128M selected-range probe peak6MiB on88MiB fixture.; Fork static/style checks and docs validation already passed for current unchanged files.; Reviewer reported an additional non-Castor parity experiment; not counted as required project validation or a substitute for the submission gate.
- Summary: Independent reviewer agent_083f847846886e13 APPROVE WITH SUGGESTIONS at63db29ea33acdd7fb9cda16e3bb1f500402a92d4: no correctness/security/specification-fidelity blockers. Latest narrowed two-defect scope is satisfied; no extra quotas/settings/API or child-view changes. Reviewer confirmed testing prerequisites and reran the three regressions via Castor:3 tests/21 assertions PASS. Same-revision main51/167 focused result reused. Optional suggestions do not require edits. Accepted limitations: selected huge lines can allocate; trusted local-worker partial JSONL remains retained until newline. Proceed to transition-owned full Castor gate; no source changes since review.
- Review: role=independent reviewer; artifact=agent_083f847846886e13; target=63db29ea33acdd7fb9cda16e3bb1f500402a92d4; scope=all7changed files and latest two-defect requirements; verdict=APPROVE WITH SUGGESTIONS; blockers=none.

## Task workflow update - 2026-10-15T21:29:15+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (96.8s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/spec-06-byte-budgets-admission-and-backpressure/var/reports/qa-20261015-212738-42051-99a53d93.
- Session/run: 75.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-15T21:29:17+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/spec-06-byte-budgets-admission-and-backpressure to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/spec-06-byte-budgets-admission-and-backpressure/var/reports/qa-20261015-212738-42051-99a53d93.
- Session/run: 75.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-15T21:29:21+00:00
- castor check passed (96.8s).
- Pushed task/spec-06-byte-budgets-admission-and-backpressure to origin.
- Created PR: <url>
- Session/run: 75.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-15T21:29:21+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (96.8s).
- Pushed task/spec-06-byte-budgets-admission-and-backpressure to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/568
- Validation: Focused Castor suite:51tests/167assertions PASS.; Independent reviewer Castor regressions:3tests/21assertions PASS.; Isolated memory_limit128M probe:92,250,021-byte (~88MiB) file; exact3-line range and continuation;6MiB peak.; PHPStan,Deptrac,dead-code,formatting,docs validation and diff checks PASS.; Full castor check is owned by this transition.
- Summary: Submit the two user-approved fixes at63db29ea33acdd7fb9cda16e3bb1f500402a92d4: stream requested file ranges without whole-file retention and preserve valid large consumer JSONL across pipe splits. Independent specification-fidelity review approved with suggestions and no blockers. Broad SPEC06 admission/quota work is superseded and excluded.

## Task workflow update - 2026-10-15T21:53:16+00:00
- Summary: User reviewed63db29ea with no new blocking findings and approved the narrowed scope. User-reported isolated-copy probes:79,560 file() comparisons passed; selected reads from88MiB source and160MiB unselected-prefix-line source stayed at2MiB peak; original cap loses event12 and revised framing preserves it;50randomized two-consumer cases passed. These are user-reported algorithm probes, not a Castor/PHPUnit rerun or whole validation-pipeline memory proof. Existing transition-owned full Castor check remains the repository gate evidence. No code changes or status transition.
- Non-blocking follow-up: unrestricted fgets() in ReadFileTargetValidator::rejectsOffsetPastEof() can exhaust128M on a huge unselected line before the bounded reader executes. Main confirmed the source call. Tracked separately as TODO/2026-10-15-bound-read-file-offset-validation-for-giant-lines.md, including a real validation-pipeline regression. The current direct-reader memory proof does not cover validation. PR568 remains scoped to the approved two fixes.

## Task workflow update - 2026-10-15T23:19:30+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User approved fixing the read-offset validator issue in PR568 rather than a separate follow-up. Add bounded logical-line counting and extend the existing strict128M read fixture to go through the native validation pipeline with a giant unselected first line. No broader quotas or behavior changes.

## Task workflow update - 2026-10-15T23:19:48+00:00
- Summary: Main owns this two-file one-off: bound ReadFileTargetValidator offset counting and strengthen existing memory fixture to exercise native validation before reading. Existing test already runs that fixture under128M, so no new test file/framework is needed. Absorb separate follow-up task into this PR after validation.
- Ownership: owner=main; fork_run=none; revision=63db29ea33acdd7fb9cda16e3bb1f500402a92d4; scope=bounded read-offset validator counting plus existing validation-pipeline memory fixture; outcome=assigned; commit=none

## Task workflow update - 2026-10-15T23:22:48+00:00
- Validation: RED: castor test --filter=testSelectedRangeFromLargeFileAvoidsArchiveSizedResidentAllocation; original validator exhausts128M at unrestricted fgets via native ValidateToolCallArgumentsListener.; GREEN: castor test --filter='ReadFileToolTest|ConsumerStdoutPollerTest|ConsumerSupervisorTest':51tests/167assertions PASS; no QA leaks.; Native validation+reader probe:167773207-byte source, before6291456 bytes, peak10485760 bytes under128M, exact3-line result/continuation, no unselected data.; castor phpstan PASS0errors; changed-path castor cs-fix clean; git diff --check PASS.
- Summary: Implemented the user-approved validator correction in current PR at861bf713eda3eeebdfbbb265dd58f5074a45ae0a (two files). ReadFileTargetValidator counts logical lines with8KiB bounded fgets chunks, preserves early exit/exact EOF counts/read-error delegation and handle cleanup. Existing memory fixture now uses native ToolValidationHarness and a160MiB unselected first line with offset2/limit3. Castor regression RED on original validator due OOM at128M, GREEN on fix with10MiB peak and exact output. Assign existing independent reviewer agent_083f847846886e13 to correction range63db29ea3..861bf713e before resubmission.
- Ownership: owner=main; fork_run=none; revision=63db29ea33acdd7fb9cda16e3bb1f500402a92d4; scope=bounded read-offset validator counting plus existing validation-pipeline memory fixture; outcome=completed; commit=861bf713eda3eeebdfbbb265dd58f5074a45ae0a
- Review assignment: role=independent reviewer; artifact=agent_083f847846886e13; target=861bf713eda3eeebdfbbb265dd58f5074a45ae0a; scope=two-file correction63db29ea3..861bf713e and user-approved same-PR validator extension.

## Task workflow update - 2026-10-15T23:26:51+00:00
- Validation: Independent reviewer: castor test filtered giant-prefix memory plus valid/past-EOF offset cases PASS4tests/11assertions; no QA leaks.; Same-revision main suite PASS51tests/167assertions; phpstan0errors; cs-fix/diffcheckclean.; Native validation+reader:160MiB unselected first line; exactoffset2/limit3 result/continuation; peak10MiB under128M.; Full castor check remains transition-owned for newrevision.
- Summary: Independent reviewer agent_083f847846886e13 APPROVE WITH SUGGESTIONS for63db29ea3..861bf713e, no blockers. Validator counts logical lines across bounded chunks with exact final/unterminated/chunk-boundary handling; native validation-pipeline memory proof replaces direct-reader-only fixture. Reviewer reran4focused Castor cases,11assertions PASS, reproducing10MiB peak under128M on160MiB first line. Previous two-defect approval remains applicable. Ready to resubmit PR568 with user-approved validator correction; separate follow-up is absorbed.
- Review: role=independent reviewer; artifact=agent_083f847846886e13; target=861bf713eda3eeebdfbbb265dd58f5074a45ae0a; scope=63db29ea3..861bf713e two-file validator/pipeline-proof correction; verdict=APPROVE WITH SUGGESTIONS; blockers=none.

## Task workflow update - 2026-10-15T23:28:38+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (83.7s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/spec-06-byte-budgets-admission-and-backpressure/var/reports/qa-20261015-232714-48948-5662fbf3.
- Session/run: 75.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-15T23:28:40+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/spec-06-byte-budgets-admission-and-backpressure to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/spec-06-byte-budgets-admission-and-backpressure/var/reports/qa-20261015-232714-48948-5662fbf3.
- Session/run: 75.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-15T23:28:40+00:00
- castor check passed (83.7s).
- Pushed task/spec-06-byte-budgets-admission-and-backpressure to origin.
- PR already exists: <url>
- Session/run: 75.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-15T23:28:40+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (83.7s).
- Pushed task/spec-06-byte-budgets-admission-and-backpressure to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/568
- Validation: Main focused Castor suite51tests/167assertions PASS.; Independent reviewer focused Castor validation4tests/11assertions PASS.; RED original validator OOM under128M through ValidateToolCallArgumentsListener; GREEN corrected native validation+read peaks10MiB on160MiB prefix, exactoffset2/limit3.; PHPStan0errors; scopedformatting and diffcheckclean.; Full castor check is owned by this transition.
- Summary: Resubmit PR568 at861bf713eda3eeebdfbbb265dd58f5074a45ae0a. Added user-approved bounded offset validation to existing streaming-read fix and strengthened memory regression through native validation+handler with160MiB unselected first line. Prior split-JSONL fix unchanged. Independent reviewer approved correction with no blockers.
