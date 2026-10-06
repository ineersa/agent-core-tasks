# Reduce full-history memory during session replay

## Goal
Follow-up to PR #469. Compaction-window retention reduces retained transcript state, but resume/startup, history selection, and child-view entry still temporarily decode full canonical event history.

Inspect these replay paths and reduce temporary full-history materialization using existing event-store iteration facilities where possible. Keep this separate from the current retention change. Preserve canonical events and existing replay, history-selection, and pending-question behavior. Measure peak memory on a large multi-compaction session before and after; do not claim bounded memory unless demonstrated.

## Acceptance criteria
- Identify full-history allocations in resume, history selection, and child-view snapshot paths.
- Reduce avoidable temporary event collections without changing replay results or persisted history.
- Record before/after peak-memory evidence and remaining limits; add only focused regression coverage for changed contracts.

## Workflow metadata
Status: DONE
Branch: task/2026-09-05-reduce-full-history-memory-during-session-replay
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay
Fork run: agent_3ea1d4815976c7e8
PR URL: https://github.com/ineersa/agent-core/pull/543
PR Status: merged
Started: 2026-10-01T21:43:10+00:00
Completed: 2026-10-02T01:52:53+00:00

## Work log
- Created: 2026-09-05T22:32:59+00:00

## Task workflow update - 2026-10-01T21:42:59+00:00
- Summary: Start as implementation-specification.md Change1, independent storage-reader slice from clean main, coordinated with parent recovery task/PR541 replacement. Scope this first pass to eliminate avoidable whole-file text duplication in parent event reads, preserve existing caller/result/ordering/corruption semantics, and add honest physical-read scalar diagnostics where existing reader boundaries support them. Parent store already has no decoded archive cache; child store already streams. Do not import rejected shared projections, alter canonical append/dispatch, or claim callers holding allFor arrays are solved. Later owner/index stages remove whole-history caller materialization.
- Specification: /home/ineersa/projects/agent-core-tasks/reports/implementation-specification.md §§1,8,12 Change1,13. Main routing inspected SessionRunEventStore::allFor file_get_contents/explode; rangeFor existing generator; JsonlRunEventLog reverseLines; AgentChildRunEventStore streaming/physical suffix primitives. Fork owns bounded I/O correction and measured evidence, not full state ownership or index architecture.

## Task workflow update - 2026-10-01T21:43:10+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-05-reduce-full-history-memory-during-session-replay.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay/.idea.
- Summary: Change1 independent streaming reader slice. Fork implementation from main4f107b034, oldPR541 read-only reference.

## Task workflow update - 2026-10-01T21:44:07+00:00
- Ownership: owner=fork; fork_run=pending; revision=4f107b034; scope=independent streaming JSONL and physical-read scalar evidence without runtime semantics changes; outcome=assigned; commit=none

## Task workflow update - 2026-10-01T22:00:40+00:00
- Recorded fork run: agent_04b9aa3666481c06
- Summary: Fork implemented Change1 storage-reader draft, but parent review has NOT accepted its memory proof. Parent allFor now streams lines while retaining required decoded list, shared forward reader and physical-read observations added. No integration/commit/push. Parent identified mandatory corrections before accepting measured improvements: fixture compares cumulative memory_get_peak_usage deltas sequentially without resetting peaks or isolating processes, so a zero streaming delta is not an independent peak; baseline stores raw decoded arrays rather than same normalized RunEvent graph. Fixture uses raw sys_get_temp_dir/mkdir/custom recursive delete instead of TestDirectoryIsolation and cleanup is not in finally; parent test uses raw proc_open/no timeout instead of Symfony Process. Forward reader reports reachedEof=true after any fgets false rather than checking feof. Existing focused tests pass but do not establish the claimed before/after resource result.
- Ownership: owner=fork; fork_run=agent_04b9aa3666481c06; revision=4f107b034+reader-draft; scope=streaming read and physical-byte evidence; outcome=blocked; commit=none
- Next eligible continuation: same fork, exact worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay. Require independent process or properly reset peak measurements using equal normalized event products, TestDirectoryIsolation/finally teardown, bounded Symfony Process, accurate EOF/error evidence, and deterministic regression against parent allFor whole-text duplication. Do not claim memory_delta=0 proves no allocation.

## Task workflow update - 2026-10-01T22:41:30+00:00
- Ownership: owner=fork; fork_run=agent_04b9aa3666481c06; revision=4f107b034+existing-draft; scope=memory-proof and EOF corrections; outcome=assigned; commit=none

## Task workflow update - 2026-10-01T22:47:39+00:00
- Validation: Parent castor test --filter=(SessionRunEventStoreTest|AgentChildRunEventStoreTest|SessionRunEventStoreSequencingTest): PASS48tests200assertions; git diff --check: PASS
- Summary: Parent reviewed corrected independent-process normalized RunEvent memory proof and feof telemetry; focused parent validation PASS48tests200assertions. Independent peak increases: production allFor6291456bytes vs whole-text counterfactual9146368bytes; streaming consumer2097152bytes, archive2853546bytes. This eliminates text duplication, NOT retained allFor graphs or full-history caller scans. Implementation slice ready for task-to-pr independent review/full gate, not merged.
- Ownership: owner=fork; fork_run=agent_04b9aa3666481c06; revision=4f107b034+corrected-reader-draft; scope=streaming reader and independent physical-memory proof; outcome=completed; commit=none

## Task workflow update - 2026-10-01T23:52:32+00:00
- Summary: Independent reviewer=agent_b088c1759749839f; target=4f107b034+dirty; scope=entire independent extraction; specification-fidelity verdict=REQUEST CHANGES: formatting two test files.

## Task workflow update - 2026-10-01T23:57:58+00:00
- Summary: Independent review approved by agent_b088c1759749839f; reviewed tree committed at 19c1af136. Specification fidelity confirmed for independent Change1 scope; no unresolved blocking review findings. Ready for transition-owned full QA gate.

## Task workflow update - 2026-10-02T00:02:10+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (102.2s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay/var/reports/qa-20261002-000028-15848-31f9bc83.
- Session/run: 75.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-02T00:02:12+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-05-reduce-full-history-memory-during-session-replay to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay/var/reports/qa-20261002-000028-15848-31f9bc83.
- Session/run: 75.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-02T00:02:14+00:00
- castor check passed (102.2s).
- Pushed task/2026-09-05-reduce-full-history-memory-during-session-replay to origin.
- Created PR: <url>
- Session/run: 75.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-02T00:02:14+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (102.2s).
- Pushed task/2026-09-05-reduce-full-history-memory-during-session-replay to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/543

## Task workflow update - 2026-10-02T00:20:23+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Address user review before merge: widget-owned parser lifetime (#542), failed handle-stat diagnostics (#543), applied batch boundary detection and best-effort checkpoint failures (#544). No merge authorized or performed.

## Task workflow update - 2026-10-02T00:20:54+00:00
- Ownership: owner=fork; fork_run=agent_04b9aa3666481c06; revision=19c1af136; scope=handle-stat reverse-read failure telemetry; outcome=assigned; commit=none

## Task workflow update - 2026-10-02T01:10:05+00:00
- Recorded fork run: agent_3ea1d4815976c7e8
- Summary: User review fixes completed: handle fstat/seek/read failure labels;49tests227assertions. Independent reviewer agent_b088c1759749839f APPROVED current committed tree 95b35764d; no blocking findings. Full transition gate pending.
- Ownership: owner=fork; fork_run=agent_3ea1d4815976c7e8; revision=95b35764d; scope=user PR review corrections; outcome=completed; commit=95b35764d

## Task workflow update - 2026-10-02T01:12:44+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-05-reduce-full-history-memory-during-session-replay.md.
- Session/run: 75.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay/var/reports/qa-20261002-011141-39752-18768831.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-10-02T01:15:51+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (55.8s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay/var/reports/qa-20261002-011456-45151-3b4d4d70.
- Session/run: 75.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-02T01:15:53+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-05-reduce-full-history-memory-during-session-replay to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay/var/reports/qa-20261002-011456-45151-3b4d4d70.
- Session/run: 75.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-02T01:15:53+00:00
- castor check passed (55.8s).
- Pushed task/2026-09-05-reduce-full-history-memory-during-session-replay to origin.
- PR already exists: <url>
- Session/run: 75.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-02T01:15:53+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (55.8s).
- Pushed task/2026-09-05-reduce-full-history-memory-during-session-replay to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/543
- Summary: Fixed gate dead-code finding by deleting unused test-only urlStatCalls counter at663750b89. Focused4/38 and dead-code pass. Prior reviewer agent_b088c1759749839f re-approved revised commit; no contract assertions removed.

## Task workflow update - 2026-10-02T01:42:12+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User re-review confirms short/empty fread results can falsely certify reverse full_scan. Fix incomplete chunk detection, retain actual returned byte count and abort before incomplete tail emission. #542/#544 approved by user review.

## Task workflow update - 2026-10-02T01:49:06+00:00
- Summary: Short-read correction b240d1e2f independently APPROVED by reviewer agent_b088c1759749839f. Actual bytes counted before rejecting incomplete chunks; no incomplete-tail yield. Deterministic short/empty wrapper and real21000byte truncation proof. Focused52/256 and static/dead/style clean.
- Ownership: owner=fork; fork_run=agent_3ea1d4815976c7e8; revision=b240d1e2f; scope=short-read failure detection and actual-byte diagnostics; outcome=completed; commit=b240d1e2f

## Task workflow update - 2026-10-02T01:50:18+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (62.3s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay/var/reports/qa-20261002-014916-56016-b4875fbd.
- Session/run: 75.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-02T01:50:19+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-05-reduce-full-history-memory-during-session-replay to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay/var/reports/qa-20261002-014916-56016-b4875fbd.
- Session/run: 75.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-02T01:50:20+00:00
- castor check passed (62.3s).
- Pushed task/2026-09-05-reduce-full-history-memory-during-session-replay to origin.
- PR already exists: <url>
- Session/run: 75.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-02T01:50:20+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (62.3s).
- Pushed task/2026-09-05-reduce-full-history-memory-during-session-replay to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/543
- Summary: Reviewed short-read correction b240d1e2f; ready for full gate before merge.

## Task workflow update - 2026-10-02T01:52:53+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-05-reduce-full-history-memory-during-session-replay into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-reduce-full-history-memory-during-session-replay.
- Pulled integration checkout: Already up to date..
- Summary: Verified user merged PR on GitHub. Clean task worktree; combined integration post-merge gate follows.

## Task workflow update - 2026-10-02T01:56:56+00:00
- Updated PR Status: merged
- Validation: castor check PASS: var/reports/qa-20261002-015453-65964-786ea6b2; 4912 unit/integration tests22388assertions; TUI6/46; live5/30; controller replay/static/dead-code/style/docs/lsp pass; git status --short: empty
- Summary: Post-merge integrated castor check PASS at b58640142. Initial gate failed only stale PHPStan DCD result cache referencing missing hash; preserved cache in var/tmp/phpstan-dead-code-stale-20261002-015304 and rebuilt via Castor. Final all lanes green, leak check clean. Integration working tree clean; task worktree removed.
