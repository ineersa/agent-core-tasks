# Route subagent lifecycle progress commits through run_control

## Goal
Next bounded Change 2 checkpoint in /home/ineersa/projects/agent-core-tasks/reports/implementation-specification.md after merged PR545.

Current source: SubagentProgressEventAppender submits canonical events through CommittedRunEventAppender. That appender writes from the execution process and dispatches InvalidateRunContext; ActiveRunContext then reconstructs on its next lookup. CommittedRunEventAppender currently has only this production consumer. DeferredSubagentBatchProgressDeliveryService owns revision gating and progress delivery markers. In controller streaming mode nonterminal updates are transient; terminal snapshots remain canonical. In-process mode currently persists progress too.

Replace this progress side-writer with a narrow command to the run_control owner using existing command bus, per-run transition locking, and commit facilities. Preserve progress payloads and supported single, parallel, failure, cancellation, and timeout behavior. Keep transient progress transient where it is today. Owner validation must use current invocation/lifecycle identities and prevent stale or duplicate deliveries from corrupting current progress. Reuse existing lifecycle revision/identity facilities; do not invent a generic remote append API or full crash journal.

Remove CommittedRunEventAppender if it has no supported callers after migration. Do not remove InvalidateRunContext globally while history-selection callers still require it. No owner-registry switch until remaining creation/recovery and canonical-writer paths are migrated. Maintain an inventory of remaining full-state readers and side writers.

Out of scope: startup file cleanup, full pending-transition/effect authorization protocol, bootstrap/index implementation, budgets, broad rendering changes, or adopting rejected PR541 caches.

## Acceptance criteria
- Execution-side subagent progress handling performs no canonical EventStore append and does not invalidate the owner's state.
- Canonical progress commands route to run_control and commit through existing owner serialization; live committed event publication follows that commit.
- Transient nonterminal progress and durable terminal/in-process progress preserve existing supported behavior and payload contracts.
- Validate source invocation/lifecycle identity and exercise duplicate, stale, cancellation, and failure delivery boundaries using existing durable metadata.
- A configured bus integration regression proves no append before owner handling and canonical/live publication after owner handling, without parent replay caused by progress.
- Remove superseded appender APIs and update their tests and architecture docs; retain remaining supported history/recovery paths.
- Run focused Castor lifecycle/progress/owner checks and controller replay. Full castor check belongs to the later CODE-REVIEW transition.

## Workflow metadata
Status: DONE
Branch: task/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control
Fork run: agent_715292220a0c2fa7
PR URL: https://github.com/ineersa/agent-core/pull/546
PR Status: merged
Started: 2026-10-02T18:28:46+00:00
Completed: 2026-10-02T20:52:11+00:00

## Work log
- Created: 2026-10-02T18:28:32+00:00

## Task workflow update - 2026-10-02T18:28:46+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control/.idea.
- Summary: Begin the next specification Change 2 checkpoint from validated integration 51abc5c255907ff54bfdb9f05ec7c4e2dfe59a55. Route execution-side subagent progress canonical writes through run_control without broad registry/recovery changes.

## Task workflow update - 2026-10-02T18:29:31+00:00
- Summary: Main routing pass completed in the new worktree. Progress side-writer is isolated to SubagentProgressEventAppender -> CommittedRunEventAppender; current delivery revision markers and forced interruption ordering need careful integration proof. Resume prior lifecycle implementation owner to save repeat investigation. Main remains read-only until handoff.
- Ownership: owner=fork; fork_run=agent_a77c1181787f7ca7; revision=51abc5c255907ff54bfdb9f05ec7c4e2dfe59a55; scope=route canonical child progress through run_control and remove progress-driven context invalidation; outcome=assigned; commit=none
- Remaining inventory: ActiveRunContext still implicitly replays on stateFor miss; full-state readers remain AgentResumeExecutionService, AgentArtifactRetrievalService, DeferredSubagentBatchChildOutcomeFactory and explicit history/repair paths. Canonical writers remain RunCommit, HistorySelectionService, HistoryTailDiscardService, WorkerFailedEventSubscriber, SessionRepairService. This checkpoint removes only execution-side progress appends; no requireLoaded switch or complete single-writer claim yet.

## Task workflow update - 2026-10-02T18:33:58+00:00
- Recorded fork run: agent_715292220a0c2fa7
- Summary: Previous owner could not resume due context limit. Fresh fork completed routing inspection only; no implementation delta yet. It identified synchronous progress revision markers and completion ordering as the main migration seams. Continue the same fork for implementation; no technical blocker established.
- Ownership: owner=fork; fork_run=agent_a77c1181787f7ca7; revision=51abc5c255907ff54bfdb9f05ec7c4e2dfe59a55; scope=route canonical child progress through run_control; outcome=blocked; commit=none
- Ownership: owner=fork; fork_run=agent_715292220a0c2fa7; revision=51abc5c255907ff54bfdb9f05ec7c4e2dfe59a55; scope=owner progress command and queued-versus-committed lifecycle integration; outcome=assigned; commit=none

## Task workflow update - 2026-10-02T19:08:33+00:00
- Recorded fork run: agent_715292220a0c2fa7
- Validation: Focused owner/lifecycle/progress checks: 45 tests / 611 assertions passed, including delayed single/parallel completion, cancellation/new-turn retirement, stale progress with no UI/canonical mutation, duplicate commands, and eventual old-lifecycle completion/cleanup.; Parent controller replay after final correction: 14 tests / 253 assertions passed, var/tmp/owner-progress-parent-controller.log.; PHPStan, Deptrac, dead-code, cs-check, docs:validate, and git diff --check passed.; Counterfactual stale-turn/resolved-call branches fail the new liveness regressions; temporary edits restored.; Configured bus proof: no producer append, canonical/live progress after owner consumption, coherent lastSeq, no invalidation, zero replay calls.; Worktree clean at 99a72e423. No full castor check or independent review yet.
- Summary: Implementation checkpoint committed locally as 99a72e423, 19 paths. Canonical subagent progress submits CommitSubagentProgress to run_control, commits through RunMessageProcessor/RunCommit, and no longer invalidates owner state. Transient controller progress preserved. Delivery markers represent consumed/discarded commands, and lifecycle completion waits for required progress consumption. Parent caught and corrected stale-turn/resolved-call liveness gap. Task-start complete; next phase is independent review and workflow-owned full gate. No push or PR yet.
- Ownership: owner=fork; fork_run=agent_715292220a0c2fa7; revision=99a72e423; scope=owner progress command, delayed-consumption coordination, superseded-progress liveness; outcome=completed; commit=99a72e423
- Removed obsolete CommittedRunEventAppender and its direct-append/invalidation mechanics tests. CommitSubagentProgressIntegrationTest preserves live mapped-publication and append-failure contracts; SubagentProgressEventAppenderTest preserves transient/controller and canonical/in-process routing. No supported history invalidation path removed.
- Known remaining failure window: canonical progress append and lifecycle marker persistence are separate, as before; the later pending-transition crash protocol is not implemented or claimed. Remaining writers are RunCommit, HistorySelectionService, HistoryTailDiscardService, WorkerFailedEventSubscriber, SessionRepairService. Remaining execution-state readers include AgentResumeExecutionService, AgentArtifactRetrievalService, DeferredSubagentBatchChildOutcomeFactory, and explicit history/repair paths. ActiveRunContext still has implicit replay on a miss.

## Task workflow update - 2026-10-02T20:04:10+00:00
- Summary: User authorizes task-to-pr. Worktree clean at 99a72e423. Reuse prior lifecycle reviewer context for independent review of this new checkpoint before the full transition gate.
- Review: role=reviewer; artifact=agent_31972556febaf6b4; revision=99a72e423; scope=owner progress migration, lifecycle ordering and liveness, identity/revision guards, specification fidelity, test replacement contracts; outcome=assigned

## Task workflow update - 2026-10-02T20:09:47+00:00
- Summary: Task-to-pr paused before transition. Reviewer agent_31972556febaf6b4 found no blocking production findings at 99a72e423 but explicitly did not finish test replacement/acceptance review. Resume refused at 205,452 tokens against 200,000 threshold. Partial approval is not sufficient for CODE-REVIEW. No new delegation started; no push, PR, or full gate run.
- Review: role=reviewer; artifact=agent_31972556febaf6b4; revision=99a72e423; scope=production routing, owner commit, stale/duplicate/interruption handling; decision=partial approval only; unresolved blocker=unfinished acceptance/test replacement review
- Next action: finish independent review of tests/CodingAgent/Session/CommitSubagentProgressIntegrationTest.php, rewritten SubagentProgressEventAppenderTest.php, added DeferredSubagentBatchLifecycleTest scenarios, and DeferredSubagentBatchRepository marker CAS implementations. Verify no append/publication before owner consumption, live publication and lastSeq afterward, zero replay/invalidation, append failure marker behavior, superseded progress eventual completion/cleanup, and contracts replacing deleted CommittedRunEventAppender tests.
- Prior reviewer cannot resume due context limit. A fresh read-only reviewer is eligible next session for the bounded remaining review. Focused filter: CommitSubagentProgressIntegrationTest|SubagentProgressEventAppenderTest|DeferredSubagentBatchLifecycleTest. Then record definitive approval and run move_task(CODE-REVIEW), which owns full castor check and push/PR creation.
- Existing evidence remains 45 focused tests / 611 assertions, parent controller replay 14 / 253, static/style/docs clean. Known nonblocking inherited window: canonical append and consumption-marker CAS are separate; retry can duplicate a cosmetic progress event. No exactly-once guarantee claimed.

## Task workflow update - 2026-10-02T20:24:07+00:00
- Validation: Independent focused rerun: castor test --filter 'CommitSubagentProgressIntegrationTest|SubagentProgressEventAppenderTest|DeferredSubagentBatchLifecycleTest' passed, 40 tests / 576 assertions in 4.3s.; Reviewer read and followed testing skill and tests/AGENTS.md. Confirmed deleted contracts have replacements, deterministic delayed-consumption coverage, eventual old-lifecycle handoff/cleanup, and no new exactly-once reliance.
- Summary: Independent review completed at unchanged 99a72e423. Prior production review plus reviewer agent_9a6bd91412c05fa7 remaining test/CAS/specification review approve with suggestions and no blockers. Reviewer clarified optional cleanup concerns are pre-existing and not introduced or orphaned by this checkpoint. Proceeding to workflow-owned full gate.
- Review: role=reviewer; artifact=agent_9a6bd91412c05fa7; revision=99a72e423; scope=remaining acceptance proofs, deleted-test replacement, lifecycle marker CAS and specification fidelity; decision=APPROVE WITH SUGGESTIONS; blockers=none. Completes production review by agent_31972556febaf6b4.
- Known inherited append/marker atomicity gap remains documented and outside scope. No full owner-registry or crash-recovery completion claim.

## Task workflow update - 2026-10-02T20:25:22+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (52.9s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control/var/reports/qa-20261002-202429-37252-4a1d5682.
- Session/run: 75.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-02T20:25:24+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control/var/reports/qa-20261002-202429-37252-4a1d5682.
- Session/run: 75.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-02T20:25:27+00:00
- castor check passed (52.9s).
- Pushed task/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control to origin.
- Created PR: <url>
- Session/run: 75.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-02T20:25:27+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (52.9s).
- Pushed task/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/546
- Summary: Submit 99a72e423 after completed independent review with no blockers. Canonical subagent progress now commits through the run-control owner; delayed superseded progress retires obligations without stale publication and allows lifecycle completion.

## Task workflow update - 2026-10-02T20:50:08+00:00
- Summary: User source-reviewed 99a72e423 against 35c11f4f4 and the implementation specification and approved this owner-routing checkpoint with no new merge-blocking findings. Approval does not claim completion of the wider specification or crash-safe exactly-once progress persistence. GitHub currently shows PR #546 OPEN with no reviewDecision, so task-done's GitHub approved-or-merged prerequisite is not yet satisfied.
- User suggested two nonblocking regression additions: (1) canonical append and consumption-marker update succeed, lifecycle redispatch fails, then progress-command redelivery produces exactly one event/sequence advance and eventual lifecycle delivery; (2) queued revision 1 is consumed after aggregate advances to 2/3 and parent destination is superseded, causing no append, consumption through current aggregate, and reachable completion.
- These additions are optional and do not hold this checkpoint. User review was source-only; existing executed Castor evidence remains author/reviewer evidence, not user-run validation.
- Append-to-marker failure window, including marker-write exceptions and process death, remains explicitly inherited and assigned to SPEC-03 durability work.

## Task workflow update - 2026-10-02T20:52:11+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control into integration checkout.
- Merge made by the 'ort' strategy.
 config/packages/messenger.yaml                                                                                   |   1 +
 config/services.yaml                                                                                             |   8 +--
 src/AgentCore/Application/AGENTS.md                                                                              |  13 +++-
 src/AgentCore/Application/Pipeline/RunOrchestrator.php                                                           |   7 +++
 src/AgentCore/Domain/Message/AGENTS.md                                                                           |   1 +
 src/AgentCore/Domain/Message/CommitSubagentProgress.php                                                          |  23 +++++++
 .../Agent/Execution/Subagent/Batch/Deferred/Interruption/DeferredSubagentBatchInterruptionCompletionService.php  |  68 +++-----------------
 .../Agent/Execution/Subagent/Batch/Deferred/Lifecycle/DeferredSubagentBatchLifecycleDeliveryService.php          |   5 ++
 .../Agent/Execution/Subagent/Batch/Deferred/Progress/DeferredSubagentBatchProgressDeliveryService.php            |  38 +++++++-----
 src/CodingAgent/Agent/Execution/Subagent/ChildRun/Progress/SubagentProgressEventAppender.php                     |  28 ++++++---
 src/CodingAgent/Application/Pipeline/CommitSubagentProgressHandler.php                                           |  90 +++++++++++++++++++++++++++
 src/CodingAgent/Session/CommittedRunEventAppender.php                                                            |  50 ---------------
 tests/CodingAgent/Agent/Execution/Gf05BareAgentsEffectiveContextIntegrationTest.php                              |   2 -
 tests/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Lifecycle/DeferredSubagentBatchLifecycleTest.php       | 149 ++++++++++++++++++++++++++++++++++++--------
 tests/CodingAgent/Agent/Execution/Subagent/ChildRun/Progress/SubagentProgressEventAppenderTest.php               | 102 ++++++++++++------------------
 tests/CodingAgent/Agent/Execution/SubagentPromptUserContextContractTest.php                                      |   2 -
 tests/CodingAgent/Session/CommitSubagentProgressIntegrationTest.php                                              | 191 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Session/CommittedRunEventAppenderLiveProgressIntegrationTest.php                               | 125 -------------------------------------
 tests/CodingAgent/Session/CommittedRunEventAppenderTest.php                                                      |  93 ----------------------------
 19 files changed, 546 insertions(+), 450 deletions(-)
 create mode 100644 src/AgentCore/Domain/Message/CommitSubagentProgress.php
 create mode 100644 src/CodingAgent/Application/Pipeline/CommitSubagentProgressHandler.php
 delete mode 100644 src/CodingAgent/Session/CommittedRunEventAppender.php
 create mode 100644 tests/CodingAgent/Session/CommitSubagentProgressIntegrationTest.php
 delete mode 100644 tests/CodingAgent/Session/CommittedRunEventAppenderLiveProgressIntegrationTest.php
 delete mode 100644 tests/CodingAgent/Session/CommittedRunEventAppenderTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-10-02-route-subagent-lifecycle-progress-commits-through-run-control.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User merged PR #546; GitHub confirms merge commit 62540790ae1aeb9a3e6ddc820d1851f53587bb6f. Integrating and running post-merge validation before starting SPEC-02.

## Task workflow update - 2026-10-02T20:53:40+00:00
- Validation: Post-merge full gate: 4949 tests / 22820 assertions; controller replay 14 / 253; TUI 6 / 46; live smoke 5 / 30; all static/docs/LSP/catalog lanes passed.; QA leak assertion and cache guard passed. Reports: var/reports/qa-20261002-205223-42160-6109f293. Summary log: var/tmp/spec-progress-post-merge-check.log.
- Summary: Post-merge castor check passed on integration revision 0087f7b6645f588223fa7fd22006a345a53c6336. Integration checkout clean and task worktree removed.
