# Cancel pending human questions across cancellation and session restart

## Goal
User requirement: cancel, relaunch, resume, and reload must cancel outstanding human questions durably. Do not retain an unanswered question behind a newly displayed question. Handle late answers safely and recover existing affected sessions without deleting history.

Observed in multiline trial session4: waiting question A atseq65; follow_up at68 without resolvingA; waitingB at75; human_response cancellingB at76. Replay expects FIFO headA and throws, blocking context refresh/cancellation. All90events retained so not compaction filtering. Inspect RunStateReducer, ApplyCommandHandler, SessionRunStateReplayService and runtime question lifecycle. Keep user data intact; no live session mutation without explicit recovery plan.

## Acceptance criteria
- Pending questions cancelled durably on cancel/relaunch/resume/reload; live and replay agree.
- Follow-up cannot leave older orphan question blocking a later response.
- Late responses to cancelled questions handled explicitly without corrupting replay.
- Historical session4-shaped event stream rebuilds without deleting events.
- Deterministic lowest-layer regression coverage plus required Castor gate.

## Workflow metadata
Status: DONE
Branch: task/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/507
PR Status: merged
Started: 2026-09-18T02:06:19+00:00
Completed: 2026-09-18T15:39:59+00:00

## Work log
- Created: 2026-09-18T02:05:48+00:00

## Task workflow update - 2026-09-18T02:06:19+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar.

## Task workflow update - 2026-09-18T02:06:49+00:00
- Summary: Routing confirms reducer follow_up sets Running without clearing pending; responses require FIFO. Delegate to diagnosing fork for runtime/replay lifecycle and deterministic regression iteration. No session4 mutation.
- Ownership: owner=fork; fork_run=agent_05f80d334ceee68e; revision=task-start baseline; scope=durable HITL lifecycle cancellation and replay recovery; outcome=assigned; commit=none

## Task workflow update - 2026-09-18T02:39:29+00:00
- Validation: Historical session4 reproduction failed before fix at reducer mismatch.; 33 focused lifecycle/replay/real ResumeHandler seam tests185 assertions passed.; After required dependency correction24 focused tests102assertions passed; scoped PHPStan/CS passed.
- Summary: Implemented pending-human lifecycle cancellation and historical session4-shaped replay recovery. Cancel/follow-up/terminal boundaries clear pending; attach/resume/relaunch/reload durably cancels outstanding waits through required ActiveRunContext. Late answers do not inject messages; same-turn FIFO preserved. No live session4 changes. Uncommitted, full gate pending.
- Ownership: owner=fork; fork_run=agent_f9fc1023e6234833; revision=task worktree uncommitted; scope=HITL lifecycle and recovery; outcome=completed; commit=none

## Task workflow update - 2026-09-18T13:23:05+00:00
- Summary: Independent reviewer agent_3758480c8ecb8845 REQUEST CHANGES on uncommitted baseline: tool-call continuation waits may wedge Cancelling on attach because suspended workers cannot deliver results. Verify/fix and cover actual deferred-tool lifecycle; update docs. ModelTurn session4 recovery reviewed sound.
- Review: role=reviewer; artifact=agent_3758480c8ecb8845; revision=uncommitted atop414b77005; scope=HITL cancellation specification fidelity; verdict=REQUEST CHANGES
- Ownership: owner=fork; fork_run=agent_f9fc1023e6234833; revision=current task tree; scope=review fixes deferred tool lifecycle and docs; outcome=assigned; commit=none

## Task workflow update - 2026-09-18T13:52:14+00:00
- Validation: 22 focused tests96 assertions including real attach/runner/deferred batch proof.; CS, scoped PHPStan, docs pass.; Focused ControllerSmokeTest live1test7assertions pass.
- Summary: Review blocker resolved deferred-tool cancel now emits cancelled tool results/batch/end and terminalizes. Reviewer agent_3758480c8ecb8845 APPROVE WITH SUGGESTIONS on committed tree ebe734116. No unresolved blockers; optional helper duplication cleanup deferred. Existing already-Cancelling tool sessions still require repair; session4-shaped wait/followup stream recovery covered.
- Review: role=reviewer; artifact=agent_3758480c8ecb8845; revision=ebe734116; scope=HITL lifecycle specification and deferred-tool cancellation; verdict=APPROVE WITH SUGGESTIONS
- Ownership: owner=fork; fork_run=agent_099102165ec3c9cc; revision=ebe734116; scope=review fixes; outcome=completed; commit=ebe734116

## Task workflow update - 2026-09-18T13:54:41+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (133.4s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar/var/reports/qa-20260918-135228-4114-86c9ca6e.
- Session/run: 56.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-18T13:54:43+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar/var/reports/qa-20260918-135228-4114-86c9ca6e.
- Session/run: 56.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-18T13:54:49+00:00
- castor check passed (133.4s).
- Pushed task/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar to origin.
- Created PR: <url>
- Session/run: 56.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-18T13:54:49+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (133.4s).
- Pushed task/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/507

## Task workflow update - 2026-09-18T14:16:47+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Accepted user P1: completed sibling output replaced by synthetic cancellation in live messages, unlike replay. Fix stored-result projection and add mixed-batch parity regression.

## Task workflow update - 2026-09-18T14:16:57+00:00
- Ownership: owner=fork; fork_run=agent_099102165ec3c9cc; revision=ebe734116; scope=mixed completed/deferred cancellation correction with live/replay proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-18T14:37:47+00:00
- Validation: Mixed completed/deferred regression red before fix at completed A isError; green afterward.; 21 focused tests118 assertions pass including live/replay output/details/notifications parity and missing-store failure.; CS and scoped PHPStan pass.
- Summary: Fixed user P1 in 951877371: completed siblings project collector-stored results with notifications; only unresolved calls synthesized. Missing stored completed result fails explicitly instead of fabricated cancellation. Reviewer agent_3758480c8ecb8845 APPROVE; retracted prior incorrect parity claim.
- Ownership: owner=fork; fork_run=agent_1a9ae30273e2fe8a; revision=951877371; scope=P1 completed sibling result preservation; outcome=completed; commit=951877371
- Review: role=reviewer; artifact=agent_3758480c8ecb8845; revision=951877371; scope=P1 specification fidelity and live/replay parity; verdict=APPROVE; blockers=none

## Task workflow update - 2026-09-18T14:38:47+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (51.9s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar/var/reports/qa-20260918-143755-11712-fc809e0d.
- Session/run: 56.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-18T14:38:49+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar/var/reports/qa-20260918-143755-11712-fc809e0d.
- Session/run: 56.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-18T14:38:50+00:00
- castor check passed (51.9s).
- Pushed task/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar to origin.
- PR already exists: <url>
- Session/run: 56.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-18T14:38:50+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (51.9s).
- Pushed task/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/507
- Summary: P1 completed-sibling preservation fixed and independently approved at951877371; rerun full gate and update existing PR507.

## Task workflow update - 2026-09-18T15:39:59+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar: ide_close_project returned isError.
- Merged task/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                                                                                 |   2 +
 docs/human-input.md                                                                                  |  16 +++++
 docs/session-storage.md                                                                              |   7 ++
 src/AgentCore/Application/Pipeline/AdvanceRunHandler.php                                             |   3 +
 src/AgentCore/Application/Pipeline/ApplyCommandHandler.php                                           | 317 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++-----
 src/AgentCore/Application/Pipeline/ToolCallResultHandler.php                                         |   1 +
 src/AgentCore/Application/Replay/RunStateReducer.php                                                 |  44 ++++++++----
 src/CodingAgent/Runtime/InProcess/InProcessAgentSessionClient.php                                    |  18 +++++
 tests/AgentCore/Application/Pipeline/AdvanceRunHandlerTest.php                                       |   8 +++
 tests/AgentCore/Application/Pipeline/ApplyCommandHandlerTest.php                                     | 453 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++--
 tests/AgentCore/Application/Pipeline/PendingHumanInputAnswerValidationTest.php                       | 232 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++--
 tests/AgentCore/Application/Replay/RunStateReducerPendingHumanLifecycleTest.php                      | 207 +++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Runtime/Controller/CommandHandler/ResumeHandlerCancelsPendingHumanTest.php         | 156 ++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Runtime/InProcess/InProcessAgentSessionClientEventsTest.php                        |  21 ++++++
 tests/CodingAgent/Runtime/InProcess/InProcessAttachCancelsDeferredToolHumanTest.php                  | 176 +++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Runtime/InProcess/InProcessAttachCancelsPendingHumanTest.php                       | 217 +++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Runtime/InProcess/InProcessSelectHistoryTurnEmitsRunHistoryPositionChangedTest.php |   4 +-
 17 files changed, 1842 insertions(+), 40 deletions(-)
 create mode 100644 tests/AgentCore/Application/Replay/RunStateReducerPendingHumanLifecycleTest.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/CommandHandler/ResumeHandlerCancelsPendingHumanTest.php
 create mode 100644 tests/CodingAgent/Runtime/InProcess/InProcessAttachCancelsDeferredToolHumanTest.php
 create mode 100644 tests/CodingAgent/Runtime/InProcess/InProcessAttachCancelsPendingHumanTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-18-cancel-pending-human-questions-across-cancellation-and-session-restar.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR507 merged at2026-09-18T15:39:04Z merge e399c359f4a1df01f0c256a524d0e3315a64e900. Integrate and clean task worktree; post-merge gate next.

## Task workflow update - 2026-09-18T15:41:20+00:00
- Updated PR Status: merged
- Validation: castor check PASS147.9s integration271f482a1; QA qa-20260918-154007-16957-c6326fc2.; 5108 unit tests,11 controller replay,9 TUI,5 live tests passed; all static/docs lanes and leak/cache guards passed.
- Summary: Post-merge integration validation complete. Git status clean; task worktree removed. IDE project-close reported degradation during cleanup, filesystem cleanup succeeded.
