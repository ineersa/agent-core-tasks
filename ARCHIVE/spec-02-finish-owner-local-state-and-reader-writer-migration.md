# SPEC- Finish owner-local state and reader/writer migration

## Goal
Implement the remaining Change 2 requirements from reports/implementation-specification.md on the external task board. Continue after PR #545 and PR #546; verify merge status before starting. Do not reimplement file-backed immutable child inputs or owner-committed shell/progress work.

Replace implicit replay-on-miss with explicit createNew, loadRecovered, requireLoaded, replaceCurrent, and release responsibilities. Migrate remaining cross-process full-state readers, including resume, artifact retrieval, and deferred child outcomes, to narrow inputs or explicit historical products. Route remaining history, repair, and failure canonical writes through exclusive owner coordination, with maintenance sharing ownership while runtime is stopped. Maintain a current call-site inventory. Do not introduce full-state RPC or cache.app graph storage.

Source: specification sections 2–3, 11, and Change 2. Coordinate bootstrap integration with SPEC-04 and lifetime/admission enforcement with SPEC-05 and SPEC-06. This task does not claim the crash protocol in SPEC-03.

## Acceptance criteria
- Ordinary requireLoaded lookup performs no I/O, serialization, cache writes, or implicit recovery; missing state never creates a new run.
- Every cross-process full-state reader has a supported replacement, and canonical writer inventory shows owner-controlled runtime writes.
- Creation requires durable reservation; recovery and explicit historical replacement are separate from getters.
- Remove superseded adapters and unsupported fallback paths after callers migrate; preserve attach, human-input, history, and repair semantics.
- Focused tests prove ownership and ordinary-read behavior; independent specification-fidelity review and workflow full gate pass.

## Workflow metadata
Status: DONE
Branch: task/spec-02-finish-owner-local-state-and-reader-writer-migration
Worktree: /home/ineersa/projects/agent-core-worktrees/spec-02-finish-owner-local-state-and-reader-writer-migration
Fork run: agent_84c6b2489d1c1a79
PR URL: https://github.com/ineersa/agent-core/pull/547
PR Status: merged
Started: 2026-10-02T20:53:59+00:00
Completed: 2026-10-02T21:52:33+00:00

## Work log
- Created: 2026-10-02T20:33:12+00:00

## Task workflow update - 2026-10-02T20:53:59+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/spec-02-finish-owner-local-state-and-reader-writer-migration.
- Created worktree /home/ineersa/projects/agent-core-worktrees/spec-02-finish-owner-local-state-and-reader-writer-migration.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/spec-02-finish-owner-local-state-and-reader-writer-migration.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/spec-02-finish-owner-local-state-and-reader-writer-migration.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/spec-02-finish-owner-local-state-and-reader-writer-migration.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/spec-02-finish-owner-local-state-and-reader-writer-migration/.idea.
- Summary: Start the next specification task after merged PR #546 and passing integrated full gate. Main refreshes reader/writer inventory before assigning bounded implementation slices; no crash-journal or indexed-bootstrap completion is claimed.

## Task workflow update - 2026-10-02T20:55:47+00:00
- Summary: Routing pass on 0087f7b6645f588223fa7fd22006a345a53c6336 confirms ActiveRunContext still replays on miss and creates queued state when replay returns no state. Remaining side writers: HistorySelectionService, HistoryTailDiscardService, SessionRepairService, WorkerFailedEventSubscriber. Reader inventory: AgentResumeExecutionService reconstructs full child state only to inspect status; artifact retrieval uses status/counts and explicit history messages; deferred failed/cancelled handoffs pass a full child state for partial context. First bounded slice is resume eligibility from existing narrow operational status with no replay.
- Ownership: owner=fork; fork_run=pending; revision=0087f7b6645f588223fa7fd22006a345a53c6336; scope=AgentResumeExecutionService narrow status eligibility and focused regressions; outcome=assigned; commit=none
- Main inspected service entry points, operational projection writer/reader, current resume tests, configured status interface alias, and side-writer call sites. Fork is appropriate for the bounded cross-process freshness and real-container validation investigation. No public behavior ambiguity is delegated; preserve resume eligibility and fail closed on unavailable status.

## Task workflow update - 2026-10-02T21:02:07+00:00
- Recorded fork run: agent_84c6b2489d1c1a79
- Validation: Fork read and followed testing skill and tests/AGENTS.md.; Focused castor test filter AgentResumeExecutionServiceTest|RunOperationalProjectionRepositoryTest passed: 26 tests / 107 assertions.; PHPStan, Deptrac, cs-check, and git diff --check passed. No full suite/check or controller replay for this isolated reader slice.; Existing cancellation test now protects narrow operational status rejection; added configured-reader refresh after direct DB status change, missing row, and reader exception with no follow-up execution.
- Summary: First bounded reader migration source-reviewed and committed locally as f020a0799. AgentResumeExecutionService uses existing RunOperationalStatusReaderInterface for eligibility instead of reconstructing full child RunState; missing/unavailable status fails closed before follow-up. SPEC-02 remains IN-PROGRESS, not complete; no push or PR.
- Ownership: owner=fork; fork_run=agent_84c6b2489d1c1a79; revision=0087f7b6645f588223fa7fd22006a345a53c6336; scope=resume eligibility narrow status and focused regressions; outcome=completed; commit=f020a0799
- Parent reviewed both-file diff and committed. Worktree clean. Independent review and full gate await the appropriate checkpoint.
- Remaining SPEC-02: artifact metadata/history and failed/cancelled outcome full-state readers; history/repair/failure canonical side writers; explicit registry and explicit owner recovery boundaries. Do not change stateFor to requireLoaded before replacement paths exist.
- Known unresolved durability detail: operational status has no validity watermark after append succeeds but projection publication fails. Current slice fails closed on missing/read-error but does not solve surviving stale-row evidence. Evaluate during owner recovery/durability integration and independent review; do not claim crash-safe eligibility.

## Task workflow update - 2026-10-02T21:17:03+00:00
- Summary: Independent reviewer agent_9a6bd91412c05fa7 REQUEST CHANGES at f020a0799: resume can accept a stale terminal operational projection after canonical append succeeds but publication fails. Old canonical replay rejected active child. Holding CODE-REVIEW until bounded freshness solution preserves safety without full-state replay or broad polling regressions.
- Review: role=reviewer; artifact=agent_9a6bd91412c05fa7; revision=f020a0799; scope=resume eligibility migration and specification fidelity; decision=REQUEST CHANGES; blocker=stale terminal projection can admit active child.
- Ownership: owner=fork; fork_run=agent_84c6b2489d1c1a79; revision=f020a0799; scope=correct stale eligibility detection without full-state replay; outcome=assigned; commit=none

## Task workflow update - 2026-10-02T21:30:03+00:00
- Validation: Correction filter AgentResumeExecutionServiceTest|RunOperationalProjectionRepositoryTest|AstraReasoningTransitionHooksTest|PlatformIntegrationTest: 57 tests / 369 assertions passed.; Independent reviewer rerun of three touched test classes: 51 tests / 349 assertions passed.; PHPStan, Deptrac, dead-code, cs-check, and git diff --check passed.; Real child archive fault proof rejects stale terminal status before reservation/followUp; measured one reverse read, 8192 bytes, one decoded event, no full scan for fixture.; Allocation gaps before and after terminal append do not invalidate current status. Comparison uses actual appended event, not sequence.cursor.
- Summary: Corrected and independently approved resume-reader checkpoint at f68aff695. Terminal eligibility now checks operational lastEventSequence against actual canonical tail under the child run lock; no full replay and no archive reads added to ordinary status polls. This supersedes the earlier claim that stale-row risk could be deferred. Only this bounded checkpoint is being submitted, not full SPEC-02 completion.
- Ownership: owner=fork; fork_run=agent_84c6b2489d1c1a79; revision=f020a0799; scope=stale terminal eligibility correction; outcome=completed; commit=f68aff695
- Review: role=reviewer; artifact=agent_9a6bd91412c05fa7; revision=f020a0799 plus exact correction committed as f68aff695; scope=full checkpoint, staleness fix, locking, sequence semantics, configured stores and specification fidelity; decision=APPROVE; blockers=none.
- Remaining SPEC-02 acceptance is explicitly outstanding: other full-state readers, history/repair/failure side writers, explicit registry and recovery boundaries. Do not treat checkpoint merge as completion of those requirements.
- Resume-only tail read cost depends on last record size. Future record-budget and committed-cut guarantees are not claimed; existing read-then-act concurrency remains outside this slice.

## Task workflow update - 2026-10-02T21:31:43+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (63.6s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/spec-02-finish-owner-local-state-and-reader-writer-migration/var/reports/qa-20261002-213040-50248-d3e4f17a.
- Session/run: 75.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-02T21:31:45+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/spec-02-finish-owner-local-state-and-reader-writer-migration to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/spec-02-finish-owner-local-state-and-reader-writer-migration/var/reports/qa-20261002-213040-50248-d3e4f17a.
- Session/run: 75.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-02T21:31:48+00:00
- castor check passed (63.6s).
- Pushed task/spec-02-finish-owner-local-state-and-reader-writer-migration to origin.
- Created PR: <url>
- Session/run: 75.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-02T21:31:48+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (63.6s).
- Pushed task/spec-02-finish-owner-local-state-and-reader-writer-migration to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/547
- Summary: Submit the independently approved resume-eligibility checkpoint at f68aff695. This is a partial SPEC-02 checkpoint; remaining reader/writer/registry requirements remain outstanding.

## Task workflow update - 2026-10-02T21:52:33+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/spec-02-finish-owner-local-state-and-reader-writer-migration into integration checkout.
- Merge made by the 'ort' strategy.
 src/AgentCore/Contract/RunOperationalStatusDTO.php                      |   2 +-
 src/CodingAgent/Agent/Execution/AgentResumeExecutionService.php         |  58 +++++++++++++++----------------
 src/CodingAgent/Repository/RunOperationalProjectionRepository.php       |   2 +-
 tests/AgentCore/Infrastructure/SymfonyAi/PlatformIntegrationTest.php    |   2 +-
 tests/CodingAgent/Agent/Execution/AgentResumeExecutionServiceTest.php   | 185 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++------------
 tests/CodingAgent/Agent/Execution/AstraReasoningTransitionHooksTest.php |   2 +-
 6 files changed, 194 insertions(+), 57 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/spec-02-finish-owner-local-state-and-reader-writer-migration.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/spec-02-finish-owner-local-state-and-reader-writer-migration.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User merged and approved PR #547 at f68aff695; GitHub confirms merge 4a97c191a69a13906f6b8e42c66d342f96612b2b. DONE applies only to the resume-eligibility checkpoint. Remaining original scope is preserved in TODO/spec-02b-owner-local-registry-and-remaining-reader-writer-migration.md. Post-merge validation follows.

## Task workflow update - 2026-10-02T21:54:05+00:00
- Validation: Post-merge: 4954 tests / 22850 assertions; controller replay 14 / 253; TUI 6 / 46; live smoke 5 / 30. All static/docs/LSP/catalog lanes passed.; Leak assertion, artifact integrity, and proxy cache guard passed.; Reports: var/reports/qa-20261002-215242-55356-1be178ea. Log: var/tmp/spec-resume-post-merge-check.log.
- Summary: Post-merge full castor check passed at integration revision b1ba9ac95c787f04d88bf2b66dfce1f066158938. Checkout clean; task worktree removed. User source review approved with no blockers; optional archive-failure/read-cost and recovery-message suggestions recorded in SPEC-02b continuation. Wider SPEC-02 remains unfinished.
