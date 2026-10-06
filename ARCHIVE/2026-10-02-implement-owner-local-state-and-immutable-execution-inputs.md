# Implement owner-local state and immutable execution inputs

## Goal
Change 2 of accepted /home/ineersa/projects/agent-core-tasks/reports/implementation-specification.md, following merged PRs542–544. New branch from integrated main, not rejected shared-cache PR541. Inventory every cross-process full-state reader and canonical writer, then implement explicit owner-local lifecycle and immutable execution inputs using existing Messenger/domain facilities. Preserve current behavior until consumers have correct replacements; do not add fallback getters, graph caches, TTL refreshes, new broker, or test-only APIs. Parent recovery task tracks the overall specification. Changes3–5 durability protocol, bootstrap/index, and lifetime/budget completion remain separate and must not be claimed by this slice.

## Acceptance criteria
- Maintain process/caller inventory including owner startup, handlers, execution workers, fork/subagent preparation, progress/failure writers, history/compaction, extension paths, and explicit inspection consumers.
- ActiveRunContext exposes explicit createNew/loadRecovered/requireLoaded/replaceCurrent/release responsibilities; ordinary requireLoaded performs zero I/O/serialization/recovery/cache writes and missing state never implies new run.
- Execution workers receive immutable invocation-specific inputs instead of retrieving/reconstructing entire runs; use existing execution messages and bounded narrow metadata facilities.
- Canonical side writers submit commands/results to run_control, with one authority and no shared full-state caches.
- Prove process boundaries with real serialization/storage where applicable; regression behavior and focused Castor checks pass; document deferred durability/admission acceptance explicitly.

## Workflow metadata
Status: DONE
Branch: task/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs
Fork run: agent_a77c1181787f7ca7
PR URL: https://github.com/ineersa/agent-core/pull/545
PR Status: merged
Started: 2026-10-02T02:12:15+00:00
Completed: 2026-10-02T18:24:49+00:00

## Work log
- Created: 2026-10-02T02:12:04+00:00

## Task workflow update - 2026-10-02T02:12:15+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs/.idea.
- Summary: Start specification Change2 from merged mainb58640142. Main owns routing inventory before sequential fork implementation. RejectedPR541 kept unchanged.

## Task workflow update - 2026-10-02T02:18:43+00:00
- Summary: Main routing + read-only scout inventory completed against b58640142. Main currently has process-local ActiveRunContext with lazy replay and narrow operational projection persistence, NOT PR541 shared caches. Registry cannot become requireLoaded safely until startup/missing-state and side-writer invalidation paths migrated. First implementation slice: owner-prepared immutable parent invocation inputs for fork/subagent execution, preserving current compaction and model/extension policies.
- Inventory scout artifact=agent_f3fc5e9bca1d113f. Main inspected owner processor/commit, execution DTOs, fork handler/service, child input factory, worker ToolCall context chain and appender.
- Cross-process full-state readers: ForkExecutionService; SubagentChildLaunchInputFactory::parentAgentsContext; AgentResumeExecutionService; DeferredSubagentBatchChildOutcomeFactory failed/cancelled outcome. Explicit read products: AgentArtifactRetrievalService, ChildRunTranscriptSnapshotProvider, SessionInitializer/history/transcript providers, diagnostics/catalog/repair.
- Canonical writers: RunCommit; HistoryTailDiscardService in owner; HistorySelectionService controller/in-process; WorkerFailedEventSubscriber owner exceptional path; ExecuteShellToolCallWorker execution-side start; CommittedRunEventAppender+InvalidateRunContext via subagent progress/lifecycle; maintenance repair. StreamingCommittedRuntimeEventStore publishes per append. Owner readiness/bootstrap remains future Change4, append/effect durability future Change3.
- Existing immutable seams: ExecuteLlmStep messages; ExecuteCompactionStep message subsets; ExecuteToolCall parentModel → worker ToolCall context → ToolContext → ForkToolHandler. LlmStepResultHandler creates ExecuteToolCall at lines252–273. Missing immutable parent context causes replay in fork and generic launch factory.
- Ownership: owner=fork; fork_run=pending; revision=b58640142; scope=immutable parent invocation inputs replacing fork/subagent preparation replay; outcome=assigned; commit=none

## Task workflow update - 2026-10-02T02:30:01+00:00
- Recorded fork run: agent_d6922626f97a220b
- Summary: First immutable-input implementation draft PARTIAL/UNVALIDATED. Fork added ToolLaunchContextDTO (fork run/turn/model/agents_context/messages; subagent narrow text only) through ExecuteToolCall/worker/ToolContext and deferred child preparation, removed parent replay deps from ForkExecutionService and SubagentChildLaunchInputFactory. No tests/static checks run yet; fixture migration and serialization proof incomplete. Resume refused184120tokens, no follow-up edits. No commit/push/PR; no ActiveRunContext switch.
- Ownership: owner=fork; fork_run=agent_d6922626f97a220b; revision=b58640142+dirty-immutable-input-draft; scope=owner-prepared fork/subagent immutable input; outcome=blocked; commit=none
- Parent source review corrections required before acceptance: LlmStepResultHandler unconditionally requires nonempty model for ALL tools (unsupported unrelated behavior; scope to child contexts); DTO forkMessages PHPDoc claims sanitized though sanitation remains worker; verify explicit fork context reaches launchSingleChildProfile rather than depending solely on ambient ToolContext; subagent factory must validate producingRunId against child parent identity, enclosing invocation turn/model coherence; remove nullable compatibility params where context truly required.
- Next owner: fresh sequential fork only because existing implementation fork cannot resume. Exact checkout /home/ineersa/projects/agent-core-worktrees/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs. Finish narrow corrections + real configured serializer/tool-batch roundtrip/HITL answer preservation/mixed ordinary-tool null context/parent successor immutability proofs. Target Castor LlmStepResultHandlerTest,ToolBatchStateDTOParentModelRoundTripTest,ForkSnapshotCompactionBeforeLaunchTest,SubagentChildLaunchModelInheritanceTest,DeferredSubagentBatchLaunchTest,SubagentPromptUserContextContractTest,Gf05BareAgentsEffectiveContextIntegrationTest,ForkChildStartRunInputCompositionTest; then canonical phpstan,deptrac,dead-code,cs-check and controller-replay as needed. No full castorcheck before task-to-pr.
- Remaining Change2 after input slice: resume/outcome readers, sidewriter commands/history selection/shell start, explicit owner startup/recovery/creation lifecycle and pure requireLoaded registry. Changes3–5 durability/index/bootstrap/budgets/lifetimes still separate. Keep PR541 untouched and never copy its full-state cache architecture.

## Task workflow update - 2026-10-02T02:40:11+00:00
- Summary: Continue existing draft in isolated task worktree. Prior fork agent_d6922626f97a220b cannot resume at184120tokens. Main verified main clean at26b636e37; leave user changes untouched. Fresh sequential owner receives bounded correction/validation only.
- Ownership: owner=fork; fork_run=pending; revision=b58640142+dirty-immutable-input-draft; scope=finish immutable parent input corrections and focused serialization/execution proof; outcome=assigned; commit=none

## Task workflow update - 2026-10-02T02:54:30+00:00
- Recorded fork run: agent_f524158933c235d4
- Validation: Parent HATFIELD_QA_RUN_ID=change2-parent-... castor test: PASS4916tests22432assertions18.7s; Fork focused52tests426assertions; phpstan/deptrac/dead-code/cs-check allPASS
- Summary: Input draft corrected; parent full castor test PASS4916/22432. Source confirms fork/subagent no longer depend on parent rebuilder and ordinary tools carry null context. Serialization proof currently uses AttributeSerializerValidatorTestFactory only, not configured Messenger/storage services: bounded final validation correction needed before accepting slice.
- Ownership: owner=fork; fork_run=agent_f524158933c235d4; revision=b58640142+28dirtypaths; scope=immutable parent input implementation; outcome=completed; commit=none
- Ownership: owner=fork; fork_run=agent_f524158933c235d4; revision=b58640142+28dirtypaths; scope=configured Messenger serialization and real tool-batch persistence proof plus boundary validation review; outcome=assigned; commit=none

## Task workflow update - 2026-10-02T03:07:35+00:00
- Recorded fork run: agent_749b6969a5af984c
- Summary: Configured kernel serializer + real SessionToolBatchStore proof added; focused73/516, controller-replay14/253 and static checks green. Parent found same identity checks repeated at three sequential levels; retain ToolExecutor owning boundary checks and required missing/kind guard, remove redundant downstream turn/model copies before input-slice checkpoint.
- Ownership: owner=fork; fork_run=agent_749b6969a5af984c; revision=b58640142+immutable-input-draft; scope=configured serialization/storage proof; outcome=completed; commit=none
- Ownership: owner=main; fork_run=none; revision=b58640142+immutable-input-draft; scope=remove duplicate post-boundary identity validation and verify integrated input slice; outcome=assigned; commit=none

## Task workflow update - 2026-10-02T03:09:43+00:00
- Validation: Parent full castor test4917tests22441assertions18.4sPASS; Parent castor cs-check/diff-checkPASS; Fork controller-replay14/253,phpstan,deptrac,dead-codePASS before parent duplicate-check deletion
- Summary: Immutable fork/subagent input slice checkpoint committed3934d035e. Parent removed redundant downstream identity checks, kept owning ToolExecutor boundary validation. Fullsuite4917/22441 and packaging smoke PASS. Next bounded slice: eliminate ExecuteShellToolCallWorker canonical start append through existing owner transition path. Main unchanged; no push.
- Ownership: owner=main; fork_run=none; revision=3934d035e; scope=immutable-input integration cleanup/checkpoint; outcome=completed; commit=3934d035e
- Ownership: owner=fork; fork_run=pending; revision=3934d035e; scope=owner-routed direct shell lifecycle start replacing execution-worker append; outcome=assigned; commit=none

## Task workflow update - 2026-10-02T03:16:03+00:00
- Recorded fork run: agent_a1cc20f1b86c4220
- Validation: Shell focused10tests155assertionsPASS; Controller replay14tests253assertionsPASS; Shell phpstan/deptrac/cs-checkPASS; Parent integrated full castor test4917tests22454assertions18.6sPASS; packaging smokePASS; gitdiffcheckPASS
- Summary: Two Change2 slices implemented and locally committed:3934d035e immutable fork/subagent inputs,61a674c32 owner-committed direct shell lifecycle start. Parent source-reviewed shell diff and fulltest PASS4917/22454. Worktree clean, no push/PR/status transition, main untouched. Overall task still IN-PROGRESS; explicit requireLoaded registry not implemented yet.
- Ownership: owner=fork; fork_run=agent_a1cc20f1b86c4220; revision=3934d035e; scope=owner-routed shell lifecycle start; outcome=completed; commit=61a674c32
- Shell ToolExecutionStart now emitted with command/history events in ApplyShellCommandHandler before dispatch, matching ordinary LlmStepResultHandler lifecycle acceptance. Running operational status matches emitted start. Worker EventStore dependency removed; worker executes and submits ToolCallResult only. Actual duration still result metadata; no new execution authorization/crash guarantee.
- Removed worker-start assertions migrated to ApplyShellCommandHandlerTest::testShellCommandOwnership data-provider scenarios with exact payload/turn/start order. ControllerReplayShellThenFollowUpTest retains canonical applied/start/end/agent_end order. Existing receipt/inflight dedup assertions retained.
- Next Change2 inventory: AgentResumeExecutionService and DeferredSubagentBatchChildOutcomeFactory replay consumers, canonical progress/history-selection/exception writers and invalidation, explicit run creation/owner recovery boundaries before requireLoaded pure registry. Artifact retrieval/child views product-specific replay Change4/5. Current ApplyShellCommandHandler reverseFor receipt scan unchanged, requires later disk index. Fullcastorcheck and independent review reserved task-to-pr; no completion/128M/bounded-memory claim.

## Task workflow update - 2026-10-02T03:28:06+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/545
- Updated PR Status: open
- Summary: User-requested early-review draftPR545 opened at61a674c32, containing immutable input3934d035e and shell owner start61a674c32. Branch pushed, task remainsIN-PROGRESS. PR body lists remainingChange2 work and explicitly no fullcastorcheck/independent task-to-PRreview yet. No merge/mainchanges.

## Task workflow update - 2026-10-02T13:30:28+00:00
- Summary: User rejects embedded launchContext graphs in run_control request state and Messenger payloads due to memory and tight broker message limits. Revised finalized direction: frozen launch input in a separate file referenced by small batch/message descriptor. Do not ship current3934d035e inline snapshot design. Prior suggestion of worker reconstruction from indexed archive is not the chosen direction.
- User requirement: launchContext belongs in a referenced file, not inline in the execution message or a retained owner-side launch snapshot. Existing owner current conversation remains its execution state; do not add retained per-fork conversation graphs. Prepare/write payload with bounded working buffers, not a whole serialized-string copy.
- Use separate immutable request payload storage rather than embedding body in mutable tool-batch JSON, so sibling mutations do not rewrite context. Message/batch carry only bounded reference/identity metadata. Respect specification§3: existing ephemeral output-cap directory is unsafe for unconsumed requests; no accessTTL or startup/shutdown cleanup may remove pending payloads.
- Pending work: define and prove publication-before-dispatch, private reference resolution/ownership checks, immutable content/cut, survival across queued/HITL/retry lifetimes, release after existing durable terminal/consumption evidence and bounded abandoned cleanup. Do not claim full Change3 crash safety. Keep collector-release and shell pending-map parity review fixes plus realistic serialized content regression pending. User raised broker message size limits; no new arbitrary broker limit assumed.

## Task workflow update - 2026-10-02T13:33:25+00:00
- Summary: User authorizes implementing review fixes plus file-referenced immutable launch payload correction, then pushing draftPR545 for user re-review. Keep scope bounded to corrected checkpoint, do not continue registry or broader specification before re-review.
- Ownership: owner=main; fork_run=none; revision=61a674c32; scope=review-fix routing and referenced-payload lifetime/transport boundary inspection; outcome=assigned; commit=none

## Task workflow update - 2026-10-02T13:34:07+00:00
- Ownership: owner=fork; fork_run=agent_a1cc20f1b86c4220; revision=61a674c32; scope=shell replay pending-map parity review fix; outcome=assigned; commit=none

## Task workflow update - 2026-10-02T13:51:43+00:00
- Recorded fork run: agent_b73a36be93ac2ca0
- Summary: Both source-review correctness fixes implemented and parent source-reviewed: durable collector no caching, in-memory release only after canonical/state commit, shell start/end pending-map parity. Focused121/949/static clean; no commit yet. Transfer to immutable file-reference owner; previous749 cannot resume across parentlifetime.
- Ownership: owner=fork; fork_run=agent_b73a36be93ac2ca0; revision=61a674c32+review-fixes; scope=collector lifetime and shell replay parity; outcome=completed; commit=none
- Ownership: owner=fork; fork_run=pending; revision=61a674c32+review-fixes; scope=replace inline launch graph with sealed file reference and strengthen transport/lifetime tests; outcome=assigned; commit=none

## Task workflow update - 2026-10-02T14:03:00+00:00
- Summary: User selects cleanup at existing fork/subagent terminal handoff formation, without adding a new result marker or acknowledgement protocol. Main verified DeferredSubagentBatchTerminalCompletionService already gates on durable terminal child projection and persists artifact outcome before building presentation; handoff reconstructs from child projection/archive, not pre-compaction input. Input lifetime is distinct from result-delivery lifetime.
- File-reference implementation must use existing terminal handoff path for idempotent input cleanup. No new result-ready marker, parent ACK requirement, or wait for unrelated sibling completion merely to delete launch input. Existing completion guards remain; add regression proving handoff rebuild/redelivery after input deletion does not reopen the input or repeat launch/compaction. Pending approval/not-yet-launched inputs still must not be deleted prematurely. Launch-failure/cancellation cleanup uses existing applicable lifecycle boundary. Previous payload fork a77c produced no changes; collector/parity fixes remain dirty.

## Task workflow update - 2026-10-02T14:30:37+00:00
- Recorded fork run: agent_a77c1181787f7ca7
- Updated PR URL: https://github.com/ineersa/agent-core/pull/545
- Updated PR Status: open
- Validation: Focused Castor158tests1481assertionsPASS; Controller replay14tests253assertionsPASS; PHPStan,Deptrac,dead-code,cs-check,docs:validate,diff-checkPASS; Message/batch sizes 591/1223B for one message and602/1244B for600messages with1339918B input file; sibling mutation doesn't read/rewrite input; Strict128M publication wrote33565288B; before/after/peak allocation54525952B; source weakref released; Fullsuite/fullcastorcheck and independent agent re-review NOT run fordfc961773; user re-review requested
- Summary: User-requested corrections implemented and pushed as dfc961773 to draft PR545; body updated for re-review. File-backed immutable launch input replaces inline conversation in owner effects/Messenger/batch. Existing shared single/parallel/interrupted handoff dispatcher deletes input; no new markers or ACK protocol. Collector lifetime and shell pending-map parity fixes included. Task remains IN-PROGRESS, worktree clean, main untouched.
- Ownership: owner=fork; fork_run=agent_a77c1181787f7ca7; revision=61a674c32+review-fixes; scope=file-reference launch inputs and handoff cleanup; outcome=completed; commit=dfc961773
- Parent inspected store publication/read/delete, worker resolution, producer, shared completion dispatcher and fallback canonical cleanup diff. Existing owner retains only original execution state, payload writing per message. New store154lines uses Symfony Filesystem resource dumpFile and existing LockFactory/child-aware paths. No graph payload in execution envelope or mutable snapshot.
- Known limitation: publication before failed transition leaves input for retry; terminal cleanup removes it, but abandoned never-terminal sessions can retain files. No new abandoned-session sweep/recovery framework. Future review should inspect lifetime/retry boundaries, especially existing deferred completion registration and missing-input behavior. Next action is user re-review, not further registry work.

## Task workflow update - 2026-10-02T16:33:46+00:00
- Summary: Resume prior payload implementation owner for bounded corrections with real original-envelope regression proofs. Its lifecycle/test context saves repeated investigation. Main owns review and transition; no further owner-registry implementation in this PR.
- Ownership: owner=fork; fork_run=agent_a77c1181787f7ca7; revision=dfc961773; scope=execution redelivery, owner-visible input failures, and isolated handoff cleanup; outcome=assigned; commit=none

## Task workflow update - 2026-10-02T16:47:07+00:00
- Summary: Parent inspected correction diff. Existing repository lookup now includes completed correlations; pending re-notifies and completed no-ops before input resolution. Active resolution sits inside error-result conversion. Cleanup waits for registration and logs deletion failures without blocking handoff. Focused 94 tests / 1,070 assertions and static checks pass.
- Ownership: owner=fork; fork_run=agent_a77c1181787f7ca7; revision=dfc961773 plus review corrections; scope=execution redelivery, owner-visible input failures, and isolated handoff cleanup; outcome=completed; commit=none
- Review: role=reviewer; artifact=pending; revision=dfc961773 plus current diff; scope=PR 545 specification fidelity and correctness, especially the three latest findings; outcome=assigned

## Task workflow update - 2026-10-02T17:08:21+00:00
- Ownership: owner=main; fork_run=none; revision=dfc961773 plus review corrections; scope=remove obsolete AgentMessage batch-serialization attributes left by superseded inline payload draft; outcome=assigned; commit=none
- Reviewer agent_31972556febaf6b4 confirms all three user findings fixed. Withdraws nested-child cleanup concern because nesting is unsupported. Requests one correction: remove nine dead AgentMessage snapshot-group attributes and two imports. No storage redesign or speculative atomic-status API needed.

## Task workflow update - 2026-10-02T17:11:17+00:00
- Validation: Implementation focused checks: 94 tests / 1,070 assertions passed; PHPStan, Deptrac, dead-code, style, docs passed.; Independent review focused checks: 159 tests across two batches passed; static checks passed.; After removing obsolete attributes: 22 tests / 155 assertions passed, rich configured serialization roundtrip and strict-128M publication included.; Original execution-envelope redelivery covered before registration, pending after deletion, and after completed delivery. Active input errors emit owner-visible error results with no external execution. Cleanup exceptions still dispatch completion.; git diff --check passed; committed worktree clean.
- Summary: Committed db5e4d8a7 with all three user findings fixed and obsolete inline-payload serialization attributes removed. Independent reviewer agent_31972556febaf6b4 APPROVES the complete checkpoint including corrections. No blocking findings remain. Ready for workflow-owned full gate, not merged.
- Ownership: owner=main; fork_run=none; revision=db5e4d8a7; scope=remove obsolete AgentMessage batch-serialization attributes; outcome=completed; commit=db5e4d8a7
- Review: role=reviewer; artifact=agent_31972556febaf6b4; revision=db5e4d8a7 content; scope=complete PR checkpoint, specification fidelity, all three user findings and obsolete annotations; decision=APPROVE; unresolved blockers=none
- CODE-REVIEW scope is the user-approved immutable launch-input and owner-committed shell lifecycle checkpoint. Remaining owner-local registry and wider specification phases are not claimed complete.

## Task workflow update - 2026-10-02T17:12:56+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (64.6s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs/var/reports/qa-20261002-171152-12226-03fbf795.
- Session/run: 75.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-02T17:12:58+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs/var/reports/qa-20261002-171152-12226-03fbf795.
- Session/run: 75.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-02T17:12:59+00:00
- castor check passed (64.6s).
- Pushed task/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs to origin.
- PR already exists: <url>
- Session/run: 75.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-02T17:12:59+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (64.6s).
- Pushed task/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/545
- Summary: Submit db5e4d8a7 after independent APPROVE. All three user findings fixed; scope is immutable file-backed launch inputs and owner-committed shell lifecycle, not the wider specification.

## Task workflow update - 2026-10-02T17:14:11+00:00
- Updated PR Status: open
- Validation: Full castor check passed for db5e4d8a7 during transition.; GitHub head db5e4d8a70a4294e120939e0cce481479c80f07a; isDraft=false.; Task worktree clean after commit and push.
- Summary: CODE-REVIEW transition succeeded: full castor check passed in 64.6 seconds and db5e4d8a7 pushed. PR545 marked ready for review, title/body updated with corrected lifetime/error behavior and current full-gate evidence. Independent approval recorded. No merge performed; main unchanged.

## Task workflow update - 2026-10-02T17:56:18+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Hold merge for user P2 at db5e4d8a7: synchronous pre-registration failure input cleanup makes execution redelivery emit a conflicting missing-input result while siblings remain pending. Correct by reusing existing durable matching batch result, without weakening duplicate checks.

## Task workflow update - 2026-10-02T17:56:29+00:00
- Summary: Resume payload implementation owner with its existing worker/lifecycle test context for one narrow correction. Main owns review and gate. No redesign or new receipt mechanism.
- Ownership: owner=fork; fork_run=agent_a77c1181787f7ca7; revision=db5e4d8a7; scope=reuse durable synchronous terminal batch result before launch-input resolution on execution redelivery; outcome=assigned; commit=none

## Task workflow update - 2026-10-02T18:09:36+00:00
- Validation: Focused implementation: 67 tests / 747 assertions passed.; Independent reviewer: 17 tests / 100 assertions passed and verified configured production worker receives durable ToolBatchCollector.; PHPStan, Deptrac, dead-code, cs-check, git diff --check passed.; Regression proves original failure and hint preserved, no input read or external execution, duplicate accepted without conflict, sibling still pending.
- Summary: Committed bef3fb3cc: referenced launch execution redelivery reuses the original durable terminal batch result before input reads. Added real persisted mixed-batch / canonical cleanup / fresh-worker regression. Corrected cleanup to read canonical tool_result.tool_call_id, exposed by this regression. Independent reviewer APPROVE, no blockers.
- Ownership: owner=fork; fork_run=agent_a77c1181787f7ca7; revision=bef3fb3cc; scope=reuse durable synchronous terminal batch result before launch-input resolution on execution redelivery; outcome=completed; commit=bef3fb3cc
- Review: role=reviewer; artifact=agent_31972556febaf6b4; revision=bef3fb3cc content; scope=latest P2 correction, canonical cleanup shape, regression strength, configured injection, specification fidelity; decision=APPROVE; unresolved blockers=none

## Task workflow update - 2026-10-02T18:10:53+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (64.9s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs/var/reports/qa-20261002-180949-19265-a0164e4c.
- Session/run: 75.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-02T18:10:55+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs/var/reports/qa-20261002-180949-19265-a0164e4c.
- Session/run: 75.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-02T18:10:56+00:00
- castor check passed (64.9s).
- Pushed task/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs to origin.
- PR already exists: <url>
- Session/run: 75.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-02T18:10:56+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (64.9s).
- Pushed task/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/545
- Summary: Submit bef3fb3cc after independent approval. Reuse original durable synchronous launch failure on execution redelivery before input resolution; regression exercises actual mixed-batch commit and cleanup with sibling still pending. Prior deferred and active-input error handling preserved.

## Task workflow update - 2026-10-02T18:24:49+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                                                                                             |  11 +-
 src/AgentCore/Application/AGENTS.md                                                                              |  23 ++++
 src/AgentCore/Application/Handler/ExecuteToolCallWorker.php                                                      | 107 ++++++++++++------
 src/AgentCore/Application/Handler/ToolBatchCollector.php                                                         |  51 ++++++---
 src/AgentCore/Application/Handler/ToolExecutor.php                                                               |  51 ++++++++-
 src/AgentCore/Application/Pipeline/ApplyShellCommandHandler.php                                                  |  22 +++-
 src/AgentCore/Application/Pipeline/LlmStepResultHandler.php                                                      |  54 ++++++++-
 src/AgentCore/Application/Pipeline/RunCommit.php                                                                 |   4 +
 src/AgentCore/Application/Replay/RunStateReducer.php                                                             |   6 +-
 src/AgentCore/Application/Tool/ToolContext.php                                                                   |  11 ++
 src/AgentCore/Contract/Tool/DeferredToolCompletionRepositoryInterface.php                                        |   3 +-
 src/AgentCore/Contract/Tool/ToolLaunchInputStoreInterface.php                                                    |  21 ++++
 src/AgentCore/Domain/Message/AGENTS.md                                                                           |   9 ++
 src/AgentCore/Domain/Message/ExecuteShellToolCall.php                                                            |   5 +-
 src/AgentCore/Domain/Message/ExecuteToolCall.php                                                                 |   5 +
 src/AgentCore/Domain/Tool/ToolLaunchContextDTO.php                                                               |  45 ++++++++
 src/AgentCore/Domain/Tool/ToolLaunchInputReferenceDTO.php                                                        |  31 ++++++
 src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Completion/DeferredSubagentBatchCompletionDispatcher.php |  18 +++
 src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Launch/DeferredSubagentBatchLaunchPlanDTO.php            |   2 +
 src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Launch/DeferredSubagentBatchLaunchService.php            |  10 +-
 src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Launch/DeferredSubagentBatchPreparationService.php       |   5 +
 src/CodingAgent/Agent/Execution/Subagent/ChildRun/Preparation/SubagentChildLaunchInputFactory.php                |  57 ++++------
 src/CodingAgent/Agent/Execution/SubagentLaunchPreparationService.php                                             |   3 +
 src/CodingAgent/Agent/Fork/ForkExecutionService.php                                                              |  41 ++++---
 src/CodingAgent/Agent/Fork/ForkExecutionServiceInterface.php                                                     |   2 +
 src/CodingAgent/Agent/Tool/ForkToolHandler.php                                                                   |   6 +
 src/CodingAgent/Entity/DeferredToolCompletionRepository.php                                                      |   6 +-
 src/CodingAgent/Runtime/Controller/CommandHandler/ExecuteShellToolCallWorker.php                                 |  24 +---
 src/CodingAgent/Session/SessionToolLaunchInputStore.php                                                          | 154 +++++++++++++++++++++++++
 src/CodingAgent/Session/ToolBatchSnapshotCleanupHookSubscriber.php                                               |  28 +++++
 src/Tui/Listener/SubmitListener.php                                                                              |   9 +-
 tests/AgentCore/Application/Handler/DeferredToolCompletionRuntimeTest.php                                        |  11 +-
 tests/AgentCore/Application/Handler/ToolCallHumanInputSuspensionTest.php                                         |   3 +
 tests/AgentCore/Application/Handler/ToolLaunchContextConfiguredBoundaryTest.php                                  | 211 +++++++++++++++++++++++++++++++++++
 tests/AgentCore/Application/Pipeline/ApplyShellCommandHandlerTest.php                                            |  90 ++++++++++++++-
 tests/AgentCore/Application/Pipeline/CommandMailboxPolicyTest.php                                                |   1 +
 tests/AgentCore/Application/Pipeline/LlmStepResultHandlerTest.php                                                | 182 ++++++++++++++++++++++++++++++
 tests/AgentCore/Application/Pipeline/RunCommitAfterTurnCommitPersistedSeqTest.php                                |   1 +
 tests/AgentCore/Application/Pipeline/RunCommitLoggingTest.php                                                    | 102 +++++++++++++++++
 tests/AgentCore/Application/Pipeline/StartRunProjectionFailureRedeliveryTest.php                                 |   1 +
 tests/AgentCore/Application/Pipeline/ToolCallResultHandlerTest.php                                               |  93 ++++++++++++++++
 tests/AgentCore/Domain/Tool/ToolBatchStateDTOParentModelRoundTripTest.php                                        |  10 +-
 tests/AgentCore/Support/InMemoryDeferredToolCompletionRepository.php                                             |   4 +-
 tests/CodingAgent/Agent/Execution/Gf05BareAgentsEffectiveContextIntegrationTest.php                              |  18 +++
 tests/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Launch/DeferredSubagentBatchLaunchTest.php             |  39 +++----
 tests/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Lifecycle/DeferredSubagentBatchLifecycleTest.php       | 118 ++++++++++++++++----
 tests/CodingAgent/Agent/Execution/Subagent/ChildRun/Preparation/SubagentChildExtensionMetadataTest.php           |   3 +-
 tests/CodingAgent/Agent/Execution/Subagent/ChildRun/Preparation/SubagentChildLaunchModelInheritanceTest.php      |  46 ++++++--
 tests/CodingAgent/Agent/Execution/SubagentExecutionServiceTest.php                                               |   9 ++
 tests/CodingAgent/Agent/Execution/SubagentPromptUserContextContractTest.php                                      |  22 ++++
 tests/CodingAgent/Agent/Execution/Support/PipelineCapturingAgentRunner.php                                       |   1 +
 tests/CodingAgent/Agent/Execution/Support/SubagentExecutionServiceFactory.php                                    |   1 -
 tests/CodingAgent/Agent/Fork/ForkExecutionServiceTest.php                                                        |  25 ++++-
 tests/CodingAgent/Agent/Fork/ForkSnapshotCompactionBeforeLaunchTest.php                                          |  89 ++++++++-------
 tests/CodingAgent/Agent/Tool/ForkToolContractTest.php                                                            |  20 ++++
 tests/CodingAgent/Entity/DeferredToolCompletionRepositoryFreshnessTest.php                                       |   6 +-
 tests/CodingAgent/Runtime/Controller/CommandHandler/ExecuteShellToolCallWorkerTest.php                           | 119 +-------------------
 tests/CodingAgent/Runtime/InProcess/InProcessAttachCancelsDeferredToolHumanTest.php                              |   1 +
 tests/CodingAgent/Runtime/InProcess/ParentPromptUserContextRegressionTest.php                                    |   1 +
 tests/CodingAgent/Session/Fixtures/tool-launch-input-memory.php                                                  |  38 +++++++
 tests/CodingAgent/Session/SessionToolLaunchInputStoreTest.php                                                    | 346 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Session/ToolBatchSnapshotCleanupHookSubscriberTest.php                                         |  40 ++++++-
 62 files changed, 2100 insertions(+), 375 deletions(-)
 create mode 100644 src/AgentCore/Contract/Tool/ToolLaunchInputStoreInterface.php
 create mode 100644 src/AgentCore/Domain/Tool/ToolLaunchContextDTO.php
 create mode 100644 src/AgentCore/Domain/Tool/ToolLaunchInputReferenceDTO.php
 create mode 100644 src/CodingAgent/Session/SessionToolLaunchInputStore.php
 create mode 100644 tests/AgentCore/Application/Handler/ToolLaunchContextConfiguredBoundaryTest.php
 create mode 100644 tests/CodingAgent/Session/Fixtures/tool-launch-input-memory.php
 create mode 100644 tests/CodingAgent/Session/SessionToolLaunchInputStoreTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-10-02-implement-owner-local-state-and-immutable-execution-inputs.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User merged PR545 on GitHub, merge commit 35c11f4f4856ae509aadcd88e3e715d4e0250385. Synchronize integration and close the immutable-input/shell-lifecycle checkpoint; broader specification remains unfinished.

## Task workflow update - 2026-10-02T18:26:52+00:00
- Updated PR Status: merged
- Validation: Integration castor check passed: 4,939 tests / 22,689 assertions; controller replay 14 / 253; TUI 6 / 46; live smoke 5 / 30; all static/docs/LSP/catalog lanes passed.; QA reports: var/reports/qa-20261002-182459-24327-712c1444. Leak check and cache guard passed.; Post-merge log: var/tmp/pr545-post-merge-check.log.
- Summary: Integrated PR545 and completed post-merge validation on 51abc5c255907ff54bfdb9f05ec7c4e2dfe59a55. Worktree removed, integration status clean. Proceeding to a separately tracked next specification checkpoint.
