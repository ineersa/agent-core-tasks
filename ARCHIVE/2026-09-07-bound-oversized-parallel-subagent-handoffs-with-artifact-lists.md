# Bound oversized parallel subagent handoffs with artifact lists

## Goal
Session 15 on main exposed prompt-cache prefix churn after a 50,500-character parallel agent_resume result. The historical oversized result was recapped at every LLM invocation, producing new random saved paths. User-selected minimal fix: when the combined parallel subagent or agent_resume response exceeds 50,000 characters, return a short oversized-response notice and list of artifacts instead of inline handoffs. Keep existing responses at or below the limit unchanged. Do not change deferred completion plumbing, output-cap persistence, settings, or historical session data in this task.

## Acceptance criteria
- Parallel subagent and agent_resume responses exceeding 50,000 characters omit inline handoffs and return an oversized notice plus artifact references.
- Responses at or below 50,000 characters retain existing presentation; single-agent responses remain unchanged.
- Deterministic boundary tests cover Unicode character counting and artifact access guidance.
- Focused Castor validation passes; full gate and independent review belong to task-to-pr.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists
Fork run: b7a11835-4deb-548a-99bc-95ba630c2b72
PR URL: https://github.com/ineersa/agent-core/pull/478
PR Status: merged
Started: 2026-09-07T15:51:20+00:00
Completed: 2026-09-07T17:11:36+00:00

## Work log
- Created: 2026-09-07T15:46:51+00:00

## Task workflow update - 2026-09-07T15:47:36+00:00
- Summary: User clarification supersedes the initial single-agent exclusion: apply the same 50,000-character response limit to single subagent and agent_resume results. Oversized single responses return a notice and artifact reference; oversized parallel responses return a notice and artifact list. Keep the broader late-hook/deferred processor changes out of scope.

## Task workflow update - 2026-09-07T15:50:58+00:00
- Summary: Additional finalized requirement: every output-cap occurrence must be visible in TUI, including repeated late-hook caps of the same historical tool call. Investigate notification dedup/projection/rendering and fix visibility at the lowest owning layer. Retain the requested 50k artifact-only response behavior for both single and parallel subagent/resume. Do not rewrite session 15.

## Task workflow update - 2026-09-07T15:51:20+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists.

## Task workflow update - 2026-09-07T15:53:08+00:00
- Ownership: owner=fork; fork_run=none; revision=1235d5c7d; scope=make every durable output-cap occurrence visible at the current TUI transcript position with virtual regression proof; outcome=assigned; commit=none
- Ownership: owner=main; fork_run=none; revision=1235d5c7d; scope=bound single and parallel subagent/resume inline responses at 50000 characters using existing artifact references; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T16:09:22+00:00
- Ownership: owner=fork; fork_run=none; revision=1235d5c7d; scope=make every durable output-cap occurrence visible at the current TUI transcript position with virtual regression proof; outcome=completed; commit=d54d5eba8213acae50433ebcd77fa5d9c63eb153
- Fork artifact agent_b156a257296f61bb handed ownership back to main. Confirmed stable notification ID replaced the old transcript block in place; projection now identifies each occurrence by runtime event sequence. Focused projection and virtual TUI regression cases pass. Fork read and followed testing skill and tests/AGENTS.

## Task workflow update - 2026-09-07T16:14:47+00:00
- Recorded fork run: b7a11835-4deb-548a-99bc-95ba630c2b72
- Validation: castor test --filter='SubagentChildRunHandoffRendererTest|SubagentParallelAggregateResultFormatterTest|DeferredSubagentBatchLifecycleTest|TranscriptProjectorTest|TuiTranscriptBlocksVirtualRenderTest': 170 tests, 926 assertions, 4.248s PHPUnit.; castor test:controller-replay: 8 tests, 125 assertions; slowest case 4.123s, none above 10s.; castor test:llm-real --filter=LlamaCppSmokeTest: 1 test, 8 assertions, 4.309s.; Scoped production PHPStan passed for both changed formatters and notification projection subscriber.; castor deptrac: zero violations/errors; castor cs-check and castor docs:validate passed.; Full castor check deferred to task-to-pr transition as required.
- Summary: Implemented requested formatter limit for single and parallel subagent/resume results and fixed repeated cap visibility. Full artifact content is preserved; oversized inline results return artifact references. TUI notification blocks now distinguish separate runtime event occurrences instead of overwriting the original block by content-stable notification ID. Task-start complete, no full gate, independent review, push, or PR yet. Unrelated TUI workaround committed on main at user request as 1235d5c7d; task commits d54d5eba8 and d01bc1fdd remain isolated.
- Ownership: owner=fork; fork_run=b7a11835-4deb-548a-99bc-95ba630c2b72; revision=1235d5c7d; scope=make every durable output-cap occurrence visible at the current TUI transcript position with virtual regression proof; outcome=completed; commit=d54d5eba8213acae50433ebcd77fa5d9c63eb153
- Ownership: owner=main; fork_run=none; revision=1235d5c7d; scope=bound single and parallel subagent/resume inline responses at 50000 characters using existing artifact references; outcome=completed; commit=d01bc1fdd

## Task workflow update - 2026-09-07T16:36:18+00:00
- Reviewer: role=reviewer; artifact=agent_a4f6333c959d9bc7; revision=d01bc1fdd; scope=full task diff and specification fidelity; outcome=REQUEST CHANGES. Parallel failure/interruption headers are prepended after the cap, allowing oversized final responses. Update stale notification-ID comments. Reviewer read and followed testing skill and tests/AGENTS. Full gate not run.
- Ownership: owner=main; fork_run=none; revision=d01bc1fdd; scope=include parallel failure/interruption headers in response limit and update invalidated notification comments with focused regression proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T16:42:55+00:00
- Validation: Review fix: castor test --filter='SubagentChildRunHandoffRendererTest|SubagentParallelAggregateResultFormatterTest|DeferredSubagentBatchLifecycleTest': 25 tests, 325 assertions, 3.098s.; castor phpstan --path=src/CodingAgent/Agent/Execution/Subagent and castor cs-check passed on 8f1781b27.; Reuse prior projection/virtual/controller replay and focused live smoke evidence for unchanged behavior; no unresolved blocker before transition gate.
- Ownership: owner=main; fork_run=none; revision=d01bc1fdd; scope=include parallel failure/interruption headers in response limit and update invalidated notification comments with focused regression proof; outcome=completed; commit=8f1781b27
- Reviewer: role=reviewer; artifact=agent_a4f6333c959d9bc7; revision=8f1781b27; scope=full task specification fidelity plus header-boundary fix; outcome=APPROVE WITH SUGGESTIONS. No blocking findings. Optional naming/constant suggestions deferred for minimality. Testing prerequisites confirmed. Remaining mandatory proof is transition-owned castor check.

## Task workflow update - 2026-09-07T16:47:37+00:00
- Summary: First transition failed; task remains IN-PROGRESS. Tool returned only generic execution error, so inspected QA reports directly. Unit lane has stale expected subagent guidance. Live lane ControllerSmokeTest::testControllerSpawnAndCompleteRun failed with HTTP 200 SSE idle timeout from port 9052, case 12.217s. Other lanes passed. No stale QA workers. Reports: var/reports/qa-20260907-164313-4897-0e274570. Updating expectation and investigating/warming live proxy after tool-description changes; no timeout increases or blind gate retry.
- Ownership: owner=main; fork_run=none; revision=8f1781b27; scope=update stale tool guidance test and diagnose live gate failure; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T16:56:04+00:00
- Validation: castor test --filter=SubagentToolDefinitionBuilderTest: 2 tests, 8 assertions, 2.117s.; castor test:llm-real warmup: 5 tests, 30 assertions, 8.344s; max case 3.351s.; castor clean:cleanup:workers:list: no stale QA worker candidates.
- Summary: Updated stale guidance assertion in 29384e239. Full live proxy warmup passed 5 tests/30 assertions in 8.344s, slowest 3.351s, cache entries stable at 395. No worker leaks or timeout changes. Initial SSE timeout cause is not conclusively established; warm concurrent transition gate is required and recurrence must be investigated rather than retried blindly. Reviewer agent_a4f6333c959d9bc7 APPROVE on 29384e239.
- Ownership: owner=main; fork_run=none; revision=8f1781b27; scope=update stale tool guidance test and diagnose live gate failure; outcome=completed; commit=29384e239
- Reviewer: role=reviewer; artifact=agent_a4f6333c959d9bc7; revision=29384e239; scope=stale assertion fix and gate-failure evidence; outcome=APPROVE; remaining proof=full transition gate.

## Task workflow update - 2026-09-07T16:57:49+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (84.5s).
- Pushed task/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists to origin.
- branch 'task/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists' set up to track 'origin/task/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists'.
- Created PR: https://github.com/ineersa/agent-core/pull/478
- Summary: Reviewer approved 29384e239. Fixed all blocking review findings and stale guidance assertion; live proxy warmup passed. Run full gate and publish PR.

## Task workflow update - 2026-09-07T16:58:13+00:00
- Validation: Transition-owned castor check passed, 84.5s, on 29384e239.; JUnit duration audit: 4889 cases, max 8.204379s, zero above 10s.
- Summary: CODE-REVIEW ready at 29384e239. PR #478 published after independent approval and full deterministic gate passed in 84.5s. Final QA reports var/reports/qa-20260907-165620-7571-37ca738a: 4,889 cases across junit reports, maximum 8.204s, none above 10s. Worktree clean. Not merged.

## Task workflow update - 2026-09-07T17:11:36+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists: ide_close_project returned isError.
- Merged task/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists into integration checkout.
- Merge made by the 'ort' strategy.
 docs/agents.md                                                                                                              |  5 +++--
 src/AgentCore/Domain/Notification/ModelNotificationDTO.php                                                                  |  3 ++-
 src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Completion/DeferredSubagentBatchTerminalCompletionService.php       |  3 +--
 src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Interruption/DeferredSubagentBatchInterruptionCompletionService.php |  6 ++----
 src/CodingAgent/Agent/Execution/Subagent/ChildRun/Result/SubagentChildRunHandoffRenderer.php                                | 27 ++++++++++++++++++++++++---
 src/CodingAgent/Agent/Execution/Subagent/SubagentParallelAggregateResultFormatter.php                                       | 29 ++++++++++++++++++++++++-----
 src/CodingAgent/Agent/Tool/AgentResumeToolDefinitionBuilder.php                                                             |  2 +-
 src/CodingAgent/Agent/Tool/AgentResumeToolHandler.php                                                                       |  2 +-
 src/CodingAgent/Agent/Tool/SubagentToolDefinitionBuilder.php                                                                |  4 ++--
 src/CodingAgent/Agent/Tool/SubagentToolHandler.php                                                                          |  2 +-
 src/CodingAgent/Resources/skills/subagents/SKILL.md                                                                         |  2 +-
 src/CodingAgent/Runtime/ProjectionPipeline/ModelNotificationProjectionSubscriber.php                                        | 27 ++++++++++++++++++---------
 src/CodingAgent/Tool/OutputCapLlmTransformHook.php                                                                          |  4 ++--
 tests/CodingAgent/Agent/Execution/Subagent/ChildRun/Result/SubagentChildRunHandoffRendererTest.php                          | 33 ++++++++++++++++++++++++++++++++
 tests/CodingAgent/Agent/Execution/Subagent/SubagentParallelAggregateResultFormatterTest.php                                 | 95 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Agent/Tool/SubagentToolDefinitionBuilderTest.php                                                          |  2 +-
 tests/CodingAgent/Runtime/Projection/TranscriptProjectorTest.php                                                            | 79 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Screen/TuiTranscriptBlocksVirtualRenderTest.php                                                                   | 83 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 18 files changed, 373 insertions(+), 35 deletions(-)
 create mode 100644 tests/CodingAgent/Agent/Execution/Subagent/SubagentParallelAggregateResultFormatterTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-bound-oversized-parallel-subagent-handoffs-with-artifact-lists.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR #478 merged at 2026-09-07T17:10:53Z, merge commit 9ab5980066a0e323c0a773fb4f601c1746a25212. Sync integration checkout and remove task worktree; post-merge full validation follows.

## Task workflow update - 2026-09-07T17:12:53+00:00
- Updated PR Status: merged
- Validation: LLM_MODE=true castor check on integration checkout: passed in 125.3s, all 10 lanes; reports var/reports/qa-20260907-171144-12163-10b950de.; 4895 JUnit cases; slowest 7.571s, none above 10s. QA leak assertion and proxy cache guard passed.; git status --short empty; task worktree directory absent.
- Summary: DONE. PR #478 confirmed merged. Integration revision 4c3a7be5c7f3946c7f5982d68aa3f011e21eb55e passes post-merge full gate. Git status clean and task worktree removed. Transition reported IDE project-close degradation, but filesystem/worktree cleanup succeeded.

## Task workflow update - 2026-09-10T22:49:40+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
