# Refresh generated session context on startup and resume

## Goal
Separate from jbcontext PR #471. User-approved behavior: recover session state first, rebuild system prompt/project context/skills and agent definitions using current runtime assets, replace only generated context before next model request, append a durable event to events.jsonl so recovery reproduces refreshed context. Preserve historical conversation, compacted summaries, and existing child sessions. Keep separate message roles; do not consolidate into system prompt. Rebuild each startup/resume; no comparison mechanism required. Document provider prompt-cache implications without implying loss of conversation history. User requests main implementation only, no forks, minimal change and minimal focused tests subject to required project gates.

## Acceptance criteria
- Startup/resume refreshes generated context after recovery and before next model request.
- Refresh is recorded as an append-only event and replay restores updated context.
- Conversation history and compaction summaries remain unchanged; existing child runs are not rewritten.
- Documentation explains provider cache impact.
- Main implements without forks; use smallest deterministic regression proof and mandatory QA.

## Workflow metadata
Status: DONE
Branch: task/2026-09-06-refresh-generated-session-context-on-startup-and-resume
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-06-refresh-generated-session-context-on-startup-and-resume
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/474
PR Status: merged
Started: 2026-09-06T23:13:41+00:00
Completed: 2026-09-07T01:40:02+00:00

## Work log
- Created: 2026-09-06T23:12:48+00:00

## Task workflow update - 2026-09-06T23:13:41+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-06-refresh-generated-session-context-on-startup-and-resume.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-06-refresh-generated-session-context-on-startup-and-resume.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-06-refresh-generated-session-context-on-startup-and-resume.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-06-refresh-generated-session-context-on-startup-and-resume.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-06-refresh-generated-session-context-on-startup-and-resume.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-06-refresh-generated-session-context-on-startup-and-resume.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-06-refresh-generated-session-context-on-startup-and-resume/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-06-refresh-generated-session-context-on-startup-and-resume.
- Summary: Main owns, no forks. Demo assets temporarily preserved in stash for clean integration precondition; restore after worktree creation.

## Task workflow update - 2026-09-06T23:20:19+00:00
- Summary: Separate worktree created. Main-only implementation, no forks. Existing run_control owns locked durable changes; refresh will use that boundary instead of direct controller event writes. Routing inspection complete; draft message untracked, no behavior changed yet. Restored unrelated demo assets after temporary stash for task creation.
- Ownership: owner=main; fork_run=none; revision=20e041cea; scope=generated context refresh on attach with durable event and replay; outcome=assigned; commit=none

## Task workflow update - 2026-09-06T23:31:28+00:00
- Validation: castor test --filter='RefreshRunContextHandlerTest|InProcessAgentSessionClient|InProcessSelectHistory': 6 tests33assertions PASS; castor deptrac: 0violations; castor docs:validate: PASS; scoped PHPStan for changed client/reducer/handler/orchestrator/model PASS; castor test:controller-replay: tool execution error, no green evidence; full gate deferred
- Summary: Main-only implementation committed9ec03b8c5, no forks. Shared generated context builder start/attach; attach dispatches RefreshRunContext through run_control locked recovery/commit, event context_refreshed carries generated messages only; common replacement used live/replay preserving history. Cache documentation added. Focused6tests33assertions passed, scoped PHPStan, deptrac and docs validate pass. Full PHPStan/controller-replay attempts returned generic tool execution error without usable output, NOT claimed green. Full gate and independent review remain task-to-pr; no push/PR.
- Ownership: owner=main; fork_run=none; revision=20e041cea; scope=generated context refresh on attach; outcome=completed; commit=9ec03b8c5

## Task workflow update - 2026-09-06T23:50:31+00:00
- Validation: castor test:controller-replay: PASS 6 tests, 88 assertions, 22.5s.
- Review: role=reviewer; artifact=agent_7d6de29670f07678; revision=9ec03b8c5; scope=origin/main...HEAD specification fidelity and correctness; verdict=REQUEST CHANGES; missing attach dispatch proof and context_refreshed runtime debug-payload leak.
- Ownership: owner=main; fork_run=none; revision=9ec03b8c5; scope=review corrections and focused validation; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T00:03:05+00:00
- Validation: Regression first failed against original runtime translator, then passed after internal event drop.; castor test --filter='RefreshRunContextHandlerTest|InProcessAgentSessionClient|InProcessSelectHistory|RuntimeEventMapperTest': PASS 54 tests, 212 assertions, 6.0s.; castor phpstan: 0 errors at be17b3869; scoped AgentCommand PHPStan: 0 errors at f00cf9b81.; castor test:llm-real --filter=LlamaCppSmokeTest: PASS 1 test, 8 assertions.; castor cs-fix: no outstanding changes.
- Summary: Review corrections complete at f00cf9b81: generated context no longer enters runtime debug events; attach dispatch verified each resume; missing sessions rejected without creating state or stopping headless stdin; replacement policy moved to Domain/Run. Independent reviewer APPROVE. Proceeding to mandatory CODE-REVIEW gate.
- Ownership: owner=main; fork_run=none; revision=9ec03b8c5; scope=review corrections and focused validation; outcome=completed; commit=f00cf9b81
- Review: role=reviewer; artifact=agent_7d6de29670f07678; revision=f00cf9b81; scope=origin/main...HEAD with iterative corrections; verdict=APPROVE; unresolved code blockers=none

## Task workflow update - 2026-09-07T00:07:33+00:00
- Validation: castor test --filter=AgentResumeExecutionServiceTest: PASS 14 tests, 51 assertions, 3.7s.; Failed gate report: var/reports/qa-20260907-000321-2642-d15b944d; unit fake-session fixture failed; controller/TUI/live/static lanes passed. No stale QA worker candidates.
- Summary: First full gate failed on pre-existing fake parent fixture in AgentResumeExecutionServiceTest when attach now requires an existing session. Replaced only that fixture with container-created session at 54c3db2de. Focused class green; reviewer APPROVE. Fresh full transition gate still required; prior partial lane successes are not a substitute.
- Ownership: owner=main; fork_run=none; revision=f00cf9b81; scope=gate-failing parent lifetime fixture; outcome=completed; commit=54c3db2de
- Review: role=reviewer; artifact=agent_7d6de29670f07678; revision=54c3db2de; scope=fixture correction plus previous approved implementation; verdict=APPROVE

## Task workflow update - 2026-09-07T00:11:12+00:00
- Validation: castor test --filter='InProcessAttachDoesNotContinueTest|InProcessAgentSessionClient|AgentResumeExecutionServiceTest': PASS 21 tests, 78 assertions.; Second failed gate report: var/reports/qa-20260907-000746-4803-c019716d; unit failed only on nonexistent session-attach-42 fixture; other test lanes green.
- Summary: Second gate found remaining fake attach fixture in InProcessAttachDoesNotContinueTest. Corrected with real container session, retained no-turn-advance assertion. Audited all attach test callers. Reviewer approves 15b68ea82; ready for fresh complete gate.
- Ownership: owner=main; fork_run=none; revision=54c3db2de; scope=remaining attach fixture and call-site audit; outcome=completed; commit=15b68ea82
- Review: role=reviewer; artifact=agent_7d6de29670f07678; revision=15b68ea82; scope=test fixture delta and prior implementation; verdict=APPROVE

## Task workflow update - 2026-09-07T00:27:14+00:00
- Validation: castor test --suite=tui: PASS 1176 tests, 5010 assertions, 9.5s with 16 ParaTest workers.; castor cs-check: PASS; scoped test PHPStan finds existing dynamic assertSame in untouched hash check only.; Root-cause artifacts: /tmp/worker_02_stdout_6a9e013bdeef0_progress (1364 completed), result_cache (1363), no junit/test_result. Other three workers completed.
- Summary: Third gate hit unrelated ParaTest event-loop hang. Worker artifacts isolate DeferredCursorCommitScreenWriterTest: runOneLoopTurn drained process-global loop with inherited watchers. Test now owns a fresh Revolt driver per case and restores previous driver, no timeout changes. TUI unit suite green; reviewer APPROVE at 952c60738. Proceeding to fresh full gate.
- Investigation: role=scout; artifact=agent_701214d531db6a50; revision=15b68ea82; scope=unit timeout artifacts; outcome=read-only findings; parent found decisive worker02 progress/result-cache gap.
- Ownership: owner=main; fork_run=none; revision=15b68ea82; scope=cursor test event-loop isolation; outcome=completed; commit=952c60738
- Review: role=reviewer; artifact=agent_7d6de29670f07678; revision=952c60738; scope=event-loop test isolation and prior implementation; verdict=APPROVE

## Task workflow update - 2026-09-07T00:28:53+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (79.6s).
- Pushed task/2026-09-06-refresh-generated-session-context-on-startup-and-resume to origin.
- branch 'task/2026-09-06-refresh-generated-session-context-on-startup-and-resume' set up to track 'origin/task/2026-09-06-refresh-generated-session-context-on-startup-and-resume'.
- Created PR: https://github.com/ineersa/agent-core/pull/474
- Summary: Independent review approves 952c60738. Fixed attach fixtures and isolated an unrelated cursor test's event loop after diagnosing full-gate timeout.

## Task workflow update - 2026-09-07T01:40:02+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-06-refresh-generated-session-context-on-startup-and-resume: ide_close_project returned isError.
- Merged task/2026-09-06-refresh-generated-session-context-on-startup-and-resume into integration checkout.
- Merge made by the 'ort' strategy.
 config/packages/messenger.yaml                                                                       |   1 +
 config/services.yaml                                                                                 |   1 +
 docs/session-storage.md                                                                              |  12 ++++++
 src/AgentCore/Application/AGENTS.md                                                                  |   1 +
 src/AgentCore/Application/Pipeline/RefreshRunContextHandler.php                                      |  33 +++++++++++++++
 src/AgentCore/Application/Pipeline/RunOrchestrator.php                                               |   8 ++++
 src/AgentCore/Application/Replay/RunStateReducer.php                                                 |  20 +++++++++
 src/AgentCore/Domain/Event/AGENTS.md                                                                 |   3 ++
 src/AgentCore/Domain/Event/RunEventTypeEnum.php                                                      |   1 +
 src/AgentCore/Domain/Message/AGENTS.md                                                               |   1 +
 src/AgentCore/Domain/Message/RefreshRunContext.php                                                   |  14 +++++++
 src/AgentCore/Domain/Run/GeneratedContext.php                                                        |  31 ++++++++++++++
 src/CodingAgent/CLI/AgentCommand.php                                                                 |   6 +--
 src/CodingAgent/Runtime/InProcess/InProcessAgentSessionClient.php                                    | 266 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++-----------------------------------------------------
 src/CodingAgent/Runtime/Protocol/RuntimeEventTranslator.php                                          |   1 +
 src/CodingAgent/Session/Export/EffectiveModelContextProjector.php                                    |   2 +-
 tests/AgentCore/Application/Pipeline/RefreshRunContextHandlerTest.php                                |  36 ++++++++++++++++
 tests/CodingAgent/Agent/Execution/AgentResumeExecutionServiceTest.php                                |   3 +-
 tests/CodingAgent/Runtime/InProcess/InProcessAgentSessionClientEventsTest.php                        |  38 ++++++++++++++++-
 tests/CodingAgent/Runtime/InProcess/InProcessAttachDoesNotContinueTest.php                           |   8 ++--
 tests/CodingAgent/Runtime/InProcess/InProcessSelectHistoryTurnEmitsRunHistoryPositionChangedTest.php |   1 +
 tests/CodingAgent/Runtime/RuntimeEventMapperTest.php                                                 |   9 ++++
 tests/Tui/Terminal/DeferredCursorCommitScreenWriterTest.php                                          |  16 +++++++
 23 files changed, 381 insertions(+), 131 deletions(-)
 create mode 100644 src/AgentCore/Application/Pipeline/RefreshRunContextHandler.php
 create mode 100644 src/AgentCore/Domain/Message/RefreshRunContext.php
 create mode 100644 src/AgentCore/Domain/Run/GeneratedContext.php
 create mode 100644 tests/AgentCore/Application/Pipeline/RefreshRunContextHandlerTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-06-refresh-generated-session-context-on-startup-and-resume.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-06-refresh-generated-session-context-on-startup-and-resume.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User merged PR #474. GitHub confirms MERGED at a8d6e6c8c6279ca06a1559ed4bb24fbf01e4f374. Integrating locally and running the post-merge Castor gate.

## Task workflow update - 2026-09-07T01:41:46+00:00
- Validation: Post-merge QA report: var/reports/qa-20260907-014012-17453-f7624700 at a290b96797be6973262b06e17a359eea22c29e12.; Controller replay PASS 6/88; TUI PASS 8/61; llm-real PASS 5/30; PHPStan, dead-code, deptrac, style, docs and catalog logs show success. Missing unit log/result; full check incomplete.; git status --short empty; refresh worktree no longer exists. Tool reported IDE project close degraded, filesystem worktree removal succeeded.
- Summary: PR #474 confirmed merged; local integration complete and task worktree removed. Integration checkout clean. Post-merge validation INCOMPLETE: LLM_MODE=true castor check terminated with unclean exit before unit-lane output/final summary. Do not interpret completed non-unit lanes as full-gate success.

## Task workflow update - 2026-09-07T02:06:42+00:00
- Validation: QA qa-20260907-014012-17453-f7624700 unit PASS4775/19519; controller6/88, TUI8/61, llm5/30 and static lanes PASS.; Subsequent jbcontext integrated PR gate including PR474 passed85.0s at ab9d0566b; this is branch validation, not a substitute for main's missing finalizer evidence.
- Summary: Correction to initial post-merge report: unit log appeared after tool reported unclean exit. All retained lane logs/JUnit are green, including unit4775tests19519assertions75.813s. Exact full Castor finalizer/exit remains unconfirmed because invoking tool returned unclean exit with no final summary. No stale QA workers. Task remains DONE; no need to rerun test lanes blindly.
- Investigation: role=scout; artifact=agent_701214d531db6a50; revision=a290b96797; scope=post-merge Castor tool unclean exit; outcome=all retained lanes complete green; no timeout/OOM evidence found; parent exit cause unconfirmed.
