# Fix fork Bash timeout loss and stale background prompts

## Goal
Dogfood bugs observed in Hatfield session #41 while viewing/running fork artifact `agent_6e7526c0865149c8` (`agent_run_id=c406bf86-658c-5ba6-b94a-a63517e90912`). The parent was launched through `castor run:agent`.

## Defect 1: Bash timeout is dropped in fork/child execution

A parent-session control call (`sleep 10`, `timeout=5`) correctly terminated after five seconds. The same Bash timeout contract was not enforced for Bash calls executed by the fork.

Canonical child evidence from `.hatfield/sessions/41/artifacts/agents/agent_6e7526c0865149c8/events.jsonl`:

- Event/result seq 1007 at `2026-08-21T14:25:36Z`: Bash arguments requested `timeout=180`; recorded duration was `223575 ms`. The command ended only because Castor's independent 210-second hard timeout fired.
- Event/result seq 1016 at `2026-08-21T14:29:19Z`: Bash arguments requested `timeout=120`; recorded duration was `214637 ms`. Again, Castor's independent 210-second hard timeout fired.
- Event/result seq 1037 at `2026-08-21T14:33:28Z`: Bash arguments requested `timeout=60`; recorded duration was `227709 ms`; result became `stale_due_to_cancel` only when the fork/run was cancelled.
- All three tool-result details recorded `timeout_seconds: null`, despite the requested Bash timeout being present in the original arguments.
- `mode: parallel` describes child tool-batch execution; these commands were not intentionally OS-backgrounded.

This made a repeatedly executed hanging test appear like an indefinitely hung fork and prevented the normal Bash timeout contract from bounding it.

## Defect 2: stale Bash background offer reappears on fork live-view re-entry

User-visible reproduction in session #41:

1. A fork has or previously had a long-running foreground Bash call that triggered the normal ~15-second `Move it to the background?` offer.
2. Leave the fork live view and later re-enter it.
3. The background offer reappears even though there is no currently backgroundable foreground Bash execution.
4. Accepting/activating it does nothing; it is a no-op.

At investigation time the child state had `pending_human_input_requests=[]`; the stale UI must not be reconstructed merely from historical child events or an expired tool question. Correlate the prompt to the currently active child Bash `tool_call_id` and its live question/continuation state.

Investigate whether both symptoms share child/deferred tool argument or lifecycle projection state, but do not force one abstraction if they are independent. Fix at the smallest owning boundaries.

## Acceptance criteria
- Fork/subagent Bash executions enforce the requested `timeout` exactly like parent Bash executions; a deterministic child `sleep 10` with `timeout=5` returns as timed out near five seconds, not at an outer Castor/tool/fork deadline.
- Child Bash result metadata retains the effective timeout instead of recording `timeout_seconds: null` when a timeout was supplied.
- Timeout terminates/reaps the child command process tree safely and resolves the fork's deferred tool result; no orphaned command, indefinite parent wait, or cancellation-only recovery.
- The `Move it to the background?` prompt is shown only for a currently running, eligible foreground Bash call and is correlated to its child run plus `tool_call_id`/question continuation.
- Leaving and re-entering a fork live view does not resurrect an expired, answered, cancelled, timed-out, or terminal Bash background offer.
- If the command finishes between prompt display and user response, the prompt is dismissed or returns a clear terminal outcome—not a silent no-op—and no background-process record is fabricated.
- Regression proof covers the fork/child transport path, not only direct parent `BashTool` invocation, plus a TUI live-view leave/re-enter scenario at the lowest correct layer.
- Load and follow `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`. Because this touches fork runtime, deferred tool execution, TUI child live view, and process lifecycle, run `castor check` before CODE-REVIEW. Never kill or signal root-owned or `HATFIELD_SESSION_ID` processes during diagnosis/tests.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt
Fork run: 9f73f7fa-bcdd-5c10-802a-e4eefc2fbf40
PR URL: https://github.com/ineersa/agent-core/pull/421
PR Status: merged
Started: 2026-08-21T16:13:06+00:00
Completed: 2026-08-22T03:02:44+00:00

## Work log
- Created: 2026-08-21T14:40:18+00:00

## Task workflow update - 2026-08-21T14:46:33+00:00
- Summary: Finalized scope decision: disable Bash backgrounding and the 15-second background offer entirely for `agent_child` sessions (forks and subagents). Child runs are already asynchronous from the parent perspective, so nested background-process lifecycle is redundant. Preserve parent/main-session backgrounding. The crucial child behavior is reliable foreground Bash timeout propagation, process-tree reaping, cancellation, and deferred-result resolution. This clarification supersedes acceptance language about supporting a valid child background prompt.

## Task workflow update - 2026-08-21T16:13:06+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt.

## Task workflow update - 2026-08-21T16:39:02+00:00
- Recorded fork run: 2d222fbe-17be-56d5-9249-c3b9d4a4948f
- Validation: castor test --filter='BashToolTest|BashToolAgentChildTimeoutIntegrationTest' — passed (31 tests, 125 assertions); castor test:tui --filter=testAgentsLiveChildBashPathDoesNotShowBackgroundOverlayOnReenter — passed (1 test, 5 assertions); castor phpstan --path=src/CodingAgent/Tool/BashTool.php — passed (0 errors); castor cs-check — passed; JetBrains diagnostics: no errors in BashTool.php or TuiSubagentLiveViewE2eTest.php; Commit verified; git diff --check clean; task worktree clean
- Summary: Implemented commit 76e6ba0afe7a06fb1f25089f58c4da9ee174c81a. Root cause: agent_child Bash entered the 15-second background prompt adapter and blocked awaiting HITL inside BashTool's supervision loop, preventing its per-call timeout checks. BashTool now detects canonical agent_child metadata, skips background prompting, continues foreground supervision, enforces requested timeout, and reaps the process group; parent-session backgrounding remains unchanged. Added unit, container integration, and replay-backed TmuxHarness live-view leave/re-enter proof. Generic ToolExecutor `timeout_seconds` remains correctly defined as ambient policy timeout rather than being repurposed for BashArgumentsDTO timeout.

## Task workflow update - 2026-08-21T18:13:00+00:00
- Recorded fork run: 2d222fbe-17be-56d5-9249-c3b9d4a4948f
- Validation: Reviewer final verdict: APPROVED at HEAD 089b3e35d; no actionable findings; castor test — passed (4,793 tests, 19,424 assertions); castor deptrac — passed (0 violations, 0 errors); castor phpstan — passed (0 errors); castor cs-check — passed (0 files fixed); castor test:tui — passed after fixture cursor root fix (42 tests, 356 assertions); Focused Tmux task proof — passed (testAgentsLiveChildBashPathDoesNotShowBackgroundOverlayOnReenter); Focused child HITL regression — passed (testLeaveChildLiveViewDropsChildQuestionAndEscDoesNotFalseCancel); Worktree clean; origin/main...HEAD diff check clean; JetBrains diagnostics/build errors clean
- Summary: Final HEAD 089b3e35da653580d1b7db6dfd62b553d4a351d9. Implementation commits: 76e6ba0af disables child Bash background HITL so per-call deadlines remain supervised; 47721fee4 adds fail-closed malformed-metadata handling and test hardening; e424ff8a5 routes detection through an @internal AppExtension seam to satisfy architecture and deduplicates child fixture scaffolding; 089b3e35d fixes stale sequence.cursor when progress fixtures overwrite events, eliminating duplicate sequence 5/6 in full parallel TUI runs. Final dependent reviewer verdict APPROVED with no actionable findings; real replay-backed TmuxHarness child enter→leave→re-enter negative background-overlay proof confirmed intact.
- Review cycle 1 REQUEST CHANGES: hardened metadata failures, corrected telemetry, deduplicated RunStarted setup, tightened Tmux marker (commit 47721fee4).
- Review cycle 2 REQUEST CHANGES: fixed AppTool→AppAgent Deptrac violation via AgentChildRunDetectorInterface and reduced fixture duplication (commit e424ff8a5).
- Full TUI gate exposed duplicate sequence 5/6 from stale fixture sequence.cursor; fixed at root and added fixture regression proof (commit 089b3e35d).
- Final dependent reviewer approved HEAD 089b3e35d with mandatory replay-backed TmuxHarness proof confirmed.

## Task workflow update - 2026-08-21T18:16:21+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (184.0s).
- Pushed task/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt to origin.
- branch 'task/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt' set up to track 'origin/task/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt'.
- Created PR: https://github.com/ineersa/agent-core/pull/421

## Task workflow update - 2026-08-21T18:16:31+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/421
- Updated PR Status: open
- Validation: move_task deterministic castor check — passed (184.0s)
- Summary: Moved to CODE-REVIEW. Deterministic castor check passed in 184.0s; branch pushed and PR #421 created.

## Task workflow update - 2026-08-21T21:20:07+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #421 feedback requires redesign: remove BashTool runtime child detection/EventStore reads and the misplaced AppExtension detector seam. Resolve child background eligibility once in the per-run toolset policy and propagate it through the existing ActiveToolSet → ExecuteToolCall → ToolContext path.

## Task workflow update - 2026-08-22T00:21:54+00:00
- Recorded fork run: c2f433f4-d93d-5d09-98b7-6d1ac6533032
- Validation: Final dependent reviewer: APPROVED at HEAD 0c2c95beb; no actionable findings; castor test — passed (4,794 tests, 19,429 assertions) after removing stale compiled ParaTest caches from the constructor change; castor deptrac — passed (0 violations, 0 errors); castor phpstan — passed (0 errors); castor cs-check — passed; castor test:tui — initial unrelated empty-pane startup timeout in TuiResumeSessionSwitchE2eTest; focused retry passed (1/5), then full retry passed (42 tests, 354 assertions); Mandatory replay-backed Tmux child live-view proof remains intact and previously focused-passed; Worktree clean; commit 496c0b615 redesign + 0c2c95beb cleanup
- Summary: PR feedback redesign completed at HEAD 0c2c95beb99429ace4d5e157ee8b35ebccf1e5d6. Removed the misplaced AppExtension child detector and all BashTool runtime EventStore reads. Child Bash background eligibility is now resolved in SubagentToolSetResolver and propagated as typed per-tool policy through ActiveToolSet → ExecuteToolCall snapshot/copy → ToolContext → BashTool. Final reviewer APPROVED with no actionable findings and confirmed both PR comments addressed. Follow-up session-storage I/O audit is tracked separately as 2026-08-21-audit-session-storage-file-io.
- PR #421 comments accepted: Extension detector seam removed; Bash no longer reads all session events at the background threshold.
- Policy redesign commit 496c0b615 propagates server-owned backgroundPromptAllowed from per-run toolset resolution to Bash invocation context.
- Cleanup commit 0c2c95beb deletes orphaned RunStarted test factory and merges duplicate assertions.
- Created separate assessment task 2026-08-21-audit-session-storage-file-io for SubagentRunMetadataReader and comprehensive session file I/O analysis.

## Task workflow update - 2026-08-22T00:28:06+00:00
- Validation: First transition report: var/reports/qa-20260822-002210-73181-91a32dc3/check-test:tui.log — 42 tests, 354 assertions, warning from TestDirectoryIsolation.php:157 rmdir Directory not empty; Other first-transition lanes passed: test 4,794/19,429; llm-real 13/144; controller-replay 13/252; deptrac/phpstan/cs/docs passed; Remote PR #421 still points to 089b3e35d; local branch is clean and ahead 2 at 0c2c95beb, confirming no push occurred; Second transition report: var/reports/qa-20260822-002535-85042-0b88e2d1 stopped during controller replay; worker PID 85186 has HATFIELD_SESSION_ID and was not touched
- Summary: Diagnosed failed CODE-REVIEW transitions without retrying. First transition's deterministic castor check ran all lanes but failed its TUI gate because TestDirectoryIsolation emitted a warning: rmdir(.../var/tmp/tui-e2e-da304571bc9314cd): Directory not empty in TuiResumeSessionSwitchE2eTest::testSelectSessionFromPickerTransitionsCleanly. PHPUnit reported 42 tests/354 assertions with 1 warning; fail-on-all-issues makes this a failed check, explaining the opaque move_task error before push (remote PR head remains 089b3e35d while local HEAD is 0c2c95beb). The second transition was interrupted during controller-replay and left its timeout/phpunit/controller process tree running. It is user-owned but tagged HATFIELD_SESSION_ID, so it was inspected and deliberately not signalled or cleaned per safety rules.

## Task workflow update - 2026-08-22T00:47:11+00:00
- Recorded fork run: e89e826d-c4b7-5fc6-bc94-ea59e6ea3e30
- Validation: Focused TuiResumeSessionSwitchE2eTest — passed on final run (3 tests, 14 assertions); Full castor test:tui — passed (42 tests, 358 assertions), no Directory-not-empty warning; castor cs-check — passed; Full castor phpstan — passed (0 errors); Worktree clean at 154995b4a
- Summary: Fixed the mandatory TUI gate blocker at commit 154995b4afdcd7b099d4b54e9a1ec1d79c27cbee. TuiResumeSessionSwitchE2eTest now waits for each successful C-d pane to exit before isolated directory teardown, preventing late ai-catalog.yaml writes from racing with rmdir. No production behavior changed.
- Confirmed lifecycle race fix uses existing TmuxHarness::waitUntilPaneExits($pane, 15.0) after all three successful C-d paths; no sleeps, warning suppression, deletion retries, or broad harness refactor.

## Task workflow update - 2026-08-22T00:53:25+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (197.4s).
- Pushed task/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt to origin.
- branch 'task/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt' set up to track 'origin/task/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt'.
- PR already exists: https://github.com/ineersa/agent-core/pull/421
- Summary: PR feedback redesign and blocking TUI teardown race fixed. Final reviewer APPROVED HEAD 154995b4afdcd7b099d4b54e9a1ec1d79c27cbee; focused/full TUI, tests, Deptrac, PHPStan, and CS validation passed. Update existing PR #421.

## Task workflow update - 2026-08-22T00:53:36+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/421
- Updated PR Status: open
- Validation: move_task deterministic castor check — passed (197.4s); Final blocker reviewer — APPROVED; no actionable findings; Branch pushed; PR #421 updated
- Summary: Updated existing PR #421 to HEAD 154995b4afdcd7b099d4b54e9a1ec1d79c27cbee after deterministic castor check passed in 197.4s.

## Task workflow update - 2026-08-22T02:01:43+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User rejected AgentCore background-prompt policy as a semantic boundary leak. Approved temporary direction: remove all backgroundPromptAllowed plumbing from AgentCore/toolset policy; CodingAgent Bash checks child identity via a bounded process-local cache of immutable RunStarted metadata, avoiding repeated full events.jsonl reads. Broader cheap run-identity architecture is tracked in session I/O audit.

## Task workflow update - 2026-08-22T02:47:22+00:00
- Validation: User architecture approval: AppTool → AppAgent layer allowance is intentional and acceptable within CodingAgent; Do not replace the allowed edge with Deptrac skip_violations
- Summary: Architecture decision: keep the broad AppTool → AppAgent Deptrac allowance. The user explicitly considers both layers internal parts of CodingAgent and accepts their dependency; the protected boundaries are AgentCore, TUI separation, and clean Extension/ExtensionApi layering. Reviewer request to replace the layer edge with a one-off skip_violations exception is rejected because it would encode a narrower architecture than intended and hide an allowed dependency as a violation exception.
- Cancelled fork agent_1e3370b6daba76d3 after reviewer feedback was rejected; no handoff or changes accepted.
- Reviewer confirmed the substantive implementation is correct: AgentCore leak removed, reader cache correct, Bash resolves child policy once before process start, tests and Tmux proof valid. Its only blocking finding was the now explicitly accepted broad AppTool → AppAgent rule.

## Task workflow update - 2026-08-22T02:51:12+00:00
- Recorded fork run: 9f73f7fa-bcdd-5c10-802a-e4eefc2fbf40
- Validation: castor test --filter=SubagentRunMetadataReaderCacheTest — passed (5 tests, 30 assertions); castor phpstan — passed (0 errors); castor cs-check — passed; JetBrains diagnostics clean; worktree clean
- Summary: Final accepted cleanup committed at fb286537b8d97e821eeb8251b8032782c61bc0a4: removed redundant dead cache-hit guard from SubagentRunMetadataReader::remember(). Broad AppTool → AppAgent allowance remains intentionally approved; optional fixture/NTH suggestions remain out of scope.

## Task workflow update - 2026-08-22T02:54:34+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (187.1s).
- Pushed task/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt to origin.
- branch 'task/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt' set up to track 'origin/task/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt'.
- PR already exists: https://github.com/ineersa/agent-core/pull/421
- Summary: Final architecture approved by user: AgentCore remains generic; CodingAgent Bash consults cached immutable child-run metadata; broad internal AppTool → AppAgent edge is intentional. Dead cache guard removed at fb286537b. Update existing PR #421 after deterministic check.

## Task workflow update - 2026-08-22T02:54:47+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/421
- Updated PR Status: open
- Validation: move_task deterministic castor check — passed (187.1s); Branch pushed and existing PR #421 updated; Final worktree clean
- Summary: PR #421 updated to final HEAD fb286537b8d97e821eeb8251b8032782c61bc0a4. AgentCore background policy leak removed; CodingAgent owns child Bash decision using cached RunStarted metadata. Deterministic castor check passed in 187.1s.

## Task workflow update - 2026-08-22T03:02:44+00:00
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt.
- Merged task/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt into integration checkout.
- Auto-merging tests/Tui/E2E/TuiResumeSessionSwitchE2eTest.php
Auto-merging tests/Tui/Support/SubagentProgressEventsFixture.php
Merge made by the 'ort' strategy.
 depfile.yaml                                                             |   2 ++
 src/CodingAgent/Agent/Execution/SubagentRunMetadataReader.php            |  31 +++++++++++++++--
 src/CodingAgent/Tool/BashTool.php                                        |  63 +++++++++++++++++++++++++++++------
 tests/CodingAgent/Agent/Execution/SubagentRunMetadataReaderCacheTest.php | 248 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Tool/BashToolAgentChildTimeoutIntegrationTest.php      | 111 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Tool/BashToolTest.php                                  | 261 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++--
 tests/Tui/E2E/TuiResumeSessionSwitchE2eTest.php                          |   3 ++
 tests/Tui/E2E/TuiSubagentLiveViewE2eTest.php                             |  94 ++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Support/SubagentChildBashBackgroundPromptFixture.php           |  62 ++++++++++++++++++++++++++++++++++
 tests/Tui/Support/SubagentChildHitlEventsFixture.php                     |  98 +++++++++++-------------------------------------------
 tests/Tui/Support/SubagentChildLiveViewFixtureSupport.php                | 107 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Support/SubagentProgressEventsFixture.php                      |   5 +++
 tests/Tui/Support/SubagentProgressEventsFixtureSequenceCursorTest.php    |  95 ++++++++++++++++++++++++++++++++++++++++++++++++++++
 13 files changed, 1085 insertions(+), 95 deletions(-)
 create mode 100644 tests/CodingAgent/Agent/Execution/SubagentRunMetadataReaderCacheTest.php
 create mode 100644 tests/CodingAgent/Tool/BashToolAgentChildTimeoutIntegrationTest.php
 create mode 100644 tests/Tui/Support/SubagentChildBashBackgroundPromptFixture.php
 create mode 100644 tests/Tui/Support/SubagentChildLiveViewFixtureSupport.php
 create mode 100644 tests/Tui/Support/SubagentProgressEventsFixtureSequenceCursorTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-21-fix-fork-bash-timeout-and-stale-background-prompt.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #421 merged on GitHub at 2026-08-22T03:01:58Z; merge commit 1e3253cf4a916e2250a3b84348452662815a6c39. Complete task workflow and clean worktree.

## Task workflow update - 2026-08-22T03:08:07+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/421
- Updated PR Status: merged
- Validation: PR #421 merged at 2026-08-22T03:01:58Z (GitHub merge commit 1e3253cf4a916e2250a3b84348452662815a6c39); move_task merged branch into integration checkout, pulled remote, removed worktree and IDEA exclusions; Post-merge LLM_MODE=true castor check: deptrac/test/controller-replay/llm-real/phpstan/cs/docs/catalog passed; test:tui failed 1 error; TUI error: TuiResumeSessionSwitchE2eTest::testResumeRepaintsSelectedSessionInVisiblePane timed out after 15.0s waiting for pane %1270 to exit after shutdown key; QA report: var/reports/qa-20260822-030257-138212-16d16527/check-test:tui.log; QA leak check passed; no owned processes/tmux sessions remained
- Summary: Task moved to DONE and worktree cleaned after PR #421 merge. Post-merge LLM_MODE=true castor check exposed a remaining TUI lifecycle flake in TuiResumeSessionSwitchE2eTest: testResumeRepaintsSelectedSessionInVisiblePane timed out after 15s waiting for pane exit. All other lanes passed and QA leak check was clean.

## Task workflow update - 2026-08-29T16:09:39.014Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
