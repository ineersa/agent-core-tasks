# Fix cancelled tool-batch terminal session corruption

## Goal
A live manual run in the tui-02 worktree exposed a pre-existing AgentCore cancellation race. During a two-tool batch, one tool durably emitted `tool_call_result_received` + `tool_execution_end`, but cancellation arrived before its `message_start`/`message_end`. The sibling tool emitted a cancelled tool message, then `tool_batch_committed(count=1)` and `agent_end(reason=cancelled)` were persisted. Every later follow-up fails `MalformedToolCallSequenceException` because the assistant tool call remains unmatched.

Observed canonical shape in session 2: assistant tool calls at seq 117; first tool start/result/end at 118/120/121 with no tool message; cancellation at 122; sibling cancelled result/end/message at 123-126; batch commit at 127; terminal agent end at 128. Missing tool result ID: `chatcmpl-tool-86ac71396b23bc7c`.

This is separate from tui-02: `TuiSessionSwitchService::cancelCurrentRun()` semantics are unchanged versus its parent, although `/new` and `/resume` use the same cancellation path and can expose the race while tools are active.

Current recovery is also insufficient: `SessionRepairService` returns “No repairable corruption detected” immediately when any terminal `agent_end` exists, so `/repair` cannot recover this exact terminal-but-malformed history.

Fix once at the shared cancellation/tool-batch commit boundary. Keep canonical repair append-only; do not edit or reorder existing events. Do not add a new command or setting.

## Acceptance criteria
- Cancelling during a multi-tool batch cannot leave any assistant tool call without exactly one matching durable tool message before the batch is committed/terminalized.
- The exact observed race—one tool completed before cancellation but its message projection was interrupted while a sibling is cancelled—has the smallest deterministic regression proof at the correct runtime layer.
- `SessionRepairService` recognizes and safely repairs the existing terminal-cancelled malformed sequence when prevention was not present; repair remains append-only, idempotent, and refuses ambiguous pending work.
- After repair, transcript reconstruction passes `AgentMessageToolCallSequenceValidator` and a subsequent user follow-up can proceed.
- Cancellation ordering and batch counts remain correct for successful, failed, and cancelled sibling tools; no duplicate tool results are introduced.
- No ExtensionApi, setting, command, storage schema, or unrelated TUI behavior changes.
- Testing skill and `tests/AGENTS.md` are followed; required focused lanes and final `castor check` pass with no leaked workers.

## Workflow metadata
Status: ARCHIVE
Branch: task/fix-cancelled-tool-batch-terminal-session-corruption
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-cancelled-tool-batch-terminal-session-corruption
Fork run: 7g8k49a3s8h3
PR URL: https://github.com/ineersa/agent-core/pull/415
PR Status: merged
Started: 2026-08-18T20:57:40.655Z
Completed: 2026-08-18T22:38:49.397Z

## Work log
- Created: 2026-08-17T00:21:02.249Z

## Task workflow update - 2026-08-18T20:57:40.655Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-cancelled-tool-batch-terminal-session-corruption.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-cancelled-tool-batch-terminal-session-corruption.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-cancelled-tool-batch-terminal-session-corruption.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-cancelled-tool-batch-terminal-session-corruption.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-cancelled-tool-batch-terminal-session-corruption.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-cancelled-tool-batch-terminal-session-corruption.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/fix-cancelled-tool-batch-terminal-session-corruption/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/fix-cancelled-tool-batch-terminal-session-corruption.

## Task workflow update - 2026-08-18T21:06:00.818Z
- Recorded fork run: 1nb3nktwtvsj
- Summary: task-start claimed; worktree /home/ineersa/projects/agent-core-worktrees/fix-cancelled-tool-batch-terminal-session-corruption. Three parallel scouts mapped: (1) root cause = ToolCallResultHandler Cancelling branch treats pendingToolCalls[id]===true as 'already projected' while the incomplete path defers projection to batch completion — collector-stored results bypassed; tool_batch_committed has no count invariant; pinning test ToolCallResultHandlerTest::testCancellingWithPartialCompleteClosesAll (~line 830) asserts the corrupt state. (2) SessionRepairService hasTerminalAgentEnd early return (~line 111) fires before message-sequence check; existing synthetic-event machinery (appendSyntheticCancelledToolResultEvents, canonicalCompletedToolGroupSpecs, idempotency/durable-end checks) reusable. (3) No ≥2-tool fixture exists; ControllerReplayBashCancelFollowUpTest is the deterministic cancel-injection template (gate on event → stdin cancel command). Fork 1nb3nktwtvsj launched with prevention guard + terminal-batch repair case + three-layer proofs (handler unit, repair unit, controller-replay E2E with new two-tool fixture).
- 2026-08-18: task-start — claimed, 3 scouts dispatched (boundary map, repair map, test-infra map), fork 1nb3nktwtvsj launched

## Task workflow update - 2026-08-18T21:31:47.209Z
- Recorded fork run: 1nb3nktwtvsj
- Validation: castor test --filter='ToolCallResultHandlerTest|SessionRepairServiceTest': PASS (36 tests, 368 assertions); castor test: PASS (4691 tests, 18690 assertions); castor test:controller-replay: PASS (13 tests, 252 assertions) incl. new race test; castor test:tui: PASS (40 tests, 331 assertions); castor phpstan: errors=0; castor cs-check (after cs-fix): clean; castor clean:cleanup:workers:list: no leaked workers; Repair grep-proofs: exactly one agent_end, one tool_batch_committed, one result_received/execution_end per id after apply
- Summary: Implementation complete via fork 1nb3nktwtvsj. Commit 0f0f53fd8621c08260923a4e0b04391ad39f930c on task/fix-cancelled-tool-batch-terminal-session-corruption (unpushed, clean worktree). 7 files, +1196/−74.

Prevention (ToolCallResultHandler + ToolBatchCollector):
- Cancelling terminalization now walks ALL pending ids in order_index order: true-but-unprojected ids get message-only projection from collector-stored results (new ToolBatchCollector::getStoredResult read accessor); false ids get the synthetic cancelled full group. tool_batch_committed.count covers all closing tool messages. Invariant comment added.

Repair (SessionRepairService):
- New terminal-cancelled malformed-batch case (gate: cancellation context + cancelled terminal + unmatched tool_call ids): missing id with durable end → message_start/message_end only, content rebuilt from durable end payload via AgentMessageNormalizer; without durable end → existing full synthetic group. No second agent_end/tool_batch_committed. Hypothetical replay validates status + AgentMessageToolCallSequenceValidator. Idempotent, append-only, ambiguous-refusal semantics unchanged.

Proofs: handler unit (exact race repro: A Running-committed/unprojected → cancel → B cancelled; both tool messages, count=2, validator-clean incl. follow-up); repair unit (exact observed shape, message-only appends, byte-identical originals, second repair no-op); controller-replay E2E ControllerReplayTwoToolCancelFollowUpTest + new two-tool fixture (printf fast + sleep 8 slow, cancel after fast tool_execution.completed, both tool messages in events.jsonl, count=2, single agent_end, follow_up completes with sentinel).
- 2026-08-18: fork 1nb3nktwtvsj finished; orchestrator verified commit 0f0f53fd8, 7 files +1196/−74, E2E proof present

## Task workflow update - 2026-08-18T21:49:44.391Z
- Recorded fork run: 7g8k49a3s8h3
- Summary: task-to-pr iteration 1: focused validation all green (castor test 4691, deptrac 0, phpstan 0, cs-check clean, controller-replay 13, test:tui 40). Reviewer REQUEST CHANGES — prevention fix approved correct/complete; blockers: (1) SessionRepairService::unmatchedAssistantToolCallIds is a drifted 44-line validator reimplementation (resets on user message → false 'no corruption' for failed-follow-up variant); (2) missing ceiling comment at synthetic-cancel fallback in ToolCallResultHandler. Cleanups: dedupe 15-line message-projection tail (appendCommittedToolResultEvents → delegate to appendToolMessageEvents), direct unit case for preserveIncoming entry (exact observed race entry). Fix fork 7g8k49a3s8h3 launched.
- 2026-08-18: task-to-pr — validation green, reviewer REQUEST CHANGES (2 blockers, 2 cleanups), fix fork 7g8k49a3s8h3 launched

## Task workflow update - 2026-08-18T22:00:57.003Z
- Validation: Reviewer iteration 1: REQUEST CHANGES (2 blockers: drifted detection walker, missing ceiling comment; 2 cleanups); Fix commit dc67b573b: all 4 addressed; rg unmatchedAssistantToolCallIds → empty; Reviewer iteration 2: APPROVED (0 blockers); castor test: PASS 4693/18745; test:controller-replay: PASS 13/252; test:tui: PASS 40/339; phpstan errors=0; cs-check clean; deptrac violations=0; no worker leaks
- Summary: task-to-pr iteration 2: fix fork 7g8k49a3s8h3 addressed all blockers on commit dc67b573b (5 files, +330/−68): validator-driven detection (walker deleted, injected validator, honest 'cannot reorder' no-op for non-missing_tool_results pinned by test), ceiling comment at synthetic fallback, projection tail dedupe (byte-identical), preserveIncoming unit case. Re-review: APPROVED (0 blockers, NTH wording nit only). Focused validation green across both iterations: castor test 4693, controller-replay 13, phpstan 0, cs-check clean, test:tui 40, deptrac 0, no worker leaks.
- 2026-08-18: task-to-pr — iteration 2 fix verified (dc67b573b), re-review APPROVED, moving to CODE-REVIEW

## Task workflow update - 2026-08-18T22:03:42.812Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (151.0s).
- Pushed task/fix-cancelled-tool-batch-terminal-session-corruption to origin.
- branch 'task/fix-cancelled-tool-batch-terminal-session-corruption' set up to track 'origin/task/fix-cancelled-tool-batch-terminal-session-corruption'.
- Created PR: https://github.com/ineersa/agent-core/pull/415

## Task workflow update - 2026-08-18T22:03:48.011Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/415
- Updated PR Status: open
- 2026-08-18: CODE-REVIEW transition — deterministic castor check passed (151.0s), branch pushed (0f0f53fd8 + dc67b573b), PR #415 created

## Task workflow update - 2026-08-18T22:38:49.397Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/fix-cancelled-tool-batch-terminal-session-corruption.
- Merged task/fix-cancelled-tool-batch-terminal-session-corruption into integration checkout.
- Merge made by the 'ort' strategy.
 .../Application/Handler/ToolBatchCollector.php     |  19 ++
 .../Application/Pipeline/ToolCallResultHandler.php | 205 ++++++++----
 .../Session/Repair/SessionRepairService.php        | 299 ++++++++++++++++-
 .../Pipeline/ToolCallResultHandlerTest.php         | 262 ++++++++++++++-
 .../ControllerReplayTwoToolCancelFollowUpTest.php  | 354 +++++++++++++++++++++
 .../fixtures/controller-two-bash-cancel-race.json  |  66 ++++
 .../Session/Repair/SessionRepairServiceTest.php    | 323 ++++++++++++++++++-
 tests/Tui/Listener/RepairCommandHandlerTest.php    |   2 +
 8 files changed, 1457 insertions(+), 73 deletions(-)
 create mode 100644 tests/CodingAgent/Runtime/Controller/E2E/ControllerReplayTwoToolCancelFollowUpTest.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/E2E/fixtures/controller-two-bash-cancel-race.json
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-cancelled-tool-batch-terminal-session-corruption.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-cancelled-tool-batch-terminal-session-corruption.
- Deleted branch task/fix-cancelled-tool-batch-terminal-session-corruption.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-08-18T22:52:51.045Z
- Validation: Merge: task branch merged into integration (8 files, +1457/−73), branch + worktree removed, JetBrains project closed; PR #415 confirmed merged on GitHub; Post-merge LLM_MODE=true castor check on integration: 7/8 lanes OK — test 4696/18762, controller-replay 13/252, llm-real 13/144, deptrac 0, phpstan 0, cs-check clean, docs:validate OK; no worker leaks; artifact integrity ok; test:tui lane: 1 failure — TuiToolExchangeCardE2eTest::testEditToolExchangeCardShowsCompactedBodyWithoutFileContextLeak; root-caused as PRE-EXISTING path-length-sensitive wrap flake from 45a6a97c1 (tui-04): asserted phrase '1 addition, 1 deletion' split by 120-col soft-wrap whose position depends on checkout absolute path (temp dir under <checkout>/var/tmp); reproduces serially on integration checkout, passes in task worktrees (different path length); unrelated to PR #415 (zero TUI files, no cancellation in test flow, same content green in worktree incl. deterministic castor check)
- 2026-08-18: task-done — merged, PR #415 confirmed; post-merge check 7/8 green; test:tui failure root-caused as pre-existing path-length wrap flake (tui-04), not this PR; follow-up fix task offered

## Task workflow update - 2026-08-19T18:16:51.197Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
