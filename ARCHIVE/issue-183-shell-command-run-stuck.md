# Fix TUI shell commands leaving agent run stuck after approval/startup timeout (#183)

## Goal
## Problem

After using a `!` shell command in the interactive chat/TUI, the agent run can become effectively broken. Follow-up/new commands fail with:

```text
Runtime error: Agent process did not emit run_started event within 15s
Controller diagnostics: stderr=<empty> stdout_buffer=<empty>
```

The session state can also remain permanently stuck in `cancelling`, and failed shell command startup leaves live controller/consumer processes behind.

Ref: https://github.com/ineersa/agent-core/issues/183

## Evidence (from issue)

### Session 2 — stuck `cancelling`

`.hatfield/sessions/2/state.json` remained stuck:

```json
{
  "status": "cancelling",
  "turnNo": 3,
  "lastSeq": 32,
  "isStreaming": false,
  "errorMessage": "Command \"cancel\" rejected because cancellation is in progress."
}
```

Event sequence: initial run → `bash ls` → `write tests.md` → assistant completed normally → turn-less shell tool event (`tool_execution_start/end`, `turn: null`) → follow-up command queued → cancel applied → follow-up rejected: `Rejected because cancel command was accepted.` → repeated cancels rejected → state never returned to idle/cancelled.

### Session 3 — startup timeout after `!rm tests.md`

`.hatfield/sessions/3/events.jsonl` contained only a single event:

```json
{"schema_version":"1.0","run_id":"3","seq":1,"turn_no":0,"type":"tool_execution_start","payload":{"tool_call_id":"sh_6a360175ec91c4.57473068","tool_name":"bash","order_index":0},"ts":"2026-06-20T02:56:53+00:00"}
```

`.hatfield/sessions/3/state.json` was empty (0 bytes).

Logs around session 3:

```text
Draft session promoted for shell command
Controller stdin EOF — parent process disconnected, shutting down  # previous session 2
Launched messenger consumer ...
Successfully acquired/released lock hatfield-run-3
tool_question.created
tool.approval_question_created
tool.approval_polling_start
SubmitListener: message dispatch failed
RuntimeException: Agent process did not emit run_started event within 15s
Controller diagnostics: stderr=<empty> stdout_buffer=<empty>
```

Error source:
- `src/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClient.php:173`
- `src/Tui/Listener/SubmitListener.php:290`
- `src/Tui/Listener/SubmitListener.php:125`

A current-user process tree stayed alive after the failure: `php bin/console agent`, `agent --controller`, and several `messenger:consume` workers.

## Suspected cause

The `!` shell-command path appears to promote a draft shell session and create a tool approval question **before** the process client observes `run_started`. `SubmitListener` then waits 15s for `run_started`, times out, and leaves a partial session (`tool_execution_start` only, empty `state.json`) plus live controller/consumer processes.

There may also be a race between queued follow-up/cancel and shell command completion that leaves session state stuck in `cancelling`.

## Areas to inspect

- `src/Tui/Listener/SubmitListener.php` (esp. around lines 125 and 290 — run_started wait + message dispatch failure)
- `src/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClient.php` (esp. around line 173 — the 15s `run_started` timeout)
- shell-command / draft-session promotion path (`Draft session promoted for shell command`)
- tool approval polling path for shell commands (`tool.approval_polling_start`)
- cancellation state transition after queued follow-up rejection (sticky `cancelling`)
- controller shutdown/cleanup when stdin EOF or startup timeout occurs

## Out of scope

- The edit-tool failure path (no `edit` tool call occurred in failing sessions). This issue is separate from PR #181.

## Acceptance criteria
- `!` shell commands do not poison the interactive agent run/session — follow-up/new commands work after a `!` shell command.
- If tool approval is required for a shell command, the runtime either emits/observes the expected startup/run state OR avoids waiting for `run_started` on a shell-only flow (no spurious 15s `run_started` timeout).
- Cancellation reaches a terminal state (`idle`/`cancelled`/completed) instead of remaining permanently `cancelling`.
- Failed shell-command startup cleans up controller/consumer processes or leaves a recoverable session state (no leaked `agent --controller` / `messenger:consume` workers).
- Real TmuxHarness E2E proof exercising the shell-command / approval / cancel flow (replay-backed, no live LLM required) is added or updated and passes `castor test:tui`.
- Focused contract/regression tests added for the bug(s) fixed (smallest failing repro first, then fix).
- `castor check` passes in the worktree (deterministic, replay-backed).

## Workflow metadata
Status: DONE
Branch: task/issue-183-shell-command-run-stuck
Worktree: /home/ineersa/projects/agent-core-worktrees/issue-183-shell-command-run-stuck
Fork run: imwgnj2qbt1f
PR URL: https://github.com/ineersa/agent-core/pull/193
PR Status: merged
Started: 2026-06-22T00:27:44.319Z
Completed: 2026-06-22T20:18:43.112Z

## Work log
- Created: 2026-06-22T00:27:25.722Z

## Task workflow update - 2026-06-22T00:27:44.319Z
- Moved TODO → IN-PROGRESS.
- Created branch task/issue-183-shell-command-run-stuck.
- Created worktree /home/ineersa/projects/agent-core-worktrees/issue-183-shell-command-run-stuck.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/issue-183-shell-command-run-stuck.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/issue-183-shell-command-run-stuck.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/issue-183-shell-command-run-stuck.

## Task workflow update - 2026-06-22T00:35:10.627Z
- Scout run completed (fork). Identified 3 root causes: (1) PRIMARY: ShellCommandHandler executes bash tool synchronously in the controller process via InProcessAgentSessionClient.send()/executeShellCommand()/toolExecutor.execute(); SafeGuard RequireApproval triggers a blocking usleep poll in ExtensionToolHookEventSubscriber that freezes the controller Revolt event loop so approval questions never reach TUI. The blocking poll is by-design for tool CONSUMER processes; shell path bypasses async Messenger tool path. (2) SECONDARY: follow-up normal prompt after shell cmd calls client.start() which waits 15s for run_started from frozen controller -> RuntimeException at JsonlProcessAgentSessionClient.php:~173 with NO stopProcess() cleanup -> orphaned controller+consumers. (3) TERTIARY: ApplyCommandHandler.applyCancelCommand() (~line 215-260) transitions to Cancelling without guarding terminal states (Completed/Failed); RunCancelled never emitted -> sticky Cancelling (Session 2 evidence). Distinct from issue-152 (which fixed cross-process CacheCommandStore + pending steer/follow_up rejection on cancel).
- Key files: src/CodingAgent/Runtime/Controller/CommandHandler/ShellCommandHandler.php (in-process send bypass), src/CodingAgent/Runtime/InProcess/InProcessAgentSessionClient.php (executeShellCommand), src/CodingAgent/Extension/ExtensionToolHookEventSubscriber.php:~291 (blocking poll), src/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClient.php (start() 15s timeout ~line 128-173, shellExecute() ~309-332), src/AgentCore/Application/Pipeline/ApplyCommandHandler.php (cancel guard ~line 215-260), src/Tui/Listener/SubmitListener.php (isShellRun&&isTerminal branch calls start()). Existing tests: CancelStickinessE2eTest (replay), ShellCommandHandlerTest (unit), ActivityStateMachineTest.

## Task workflow update - 2026-06-22T00:36:25.288Z
- Recorded fork run: kz3pt5zayr3b
- Summary: Implementation fork launched (run kz3pt5zayr3b) in worktree to fix all 3 root causes: Fix A (route shell tool exec through async Messenger tool path instead of synchronous in-controller execution), Fix B (defensive stopProcess() cleanup on start()/shellExecute() timeout), Fix C (terminal-state guard in applyCancelCommand). Fork instructed to add TUI E2E replay proof + cancel-guard regression test + controller E2E for out-of-process shell exec, and pass castor check. Fork will STOP before CODE-REVIEW (no PR/push).

## Task workflow update - 2026-06-22T00:57:00.480Z
- Validation: castor test --filter=testCancelOn(Completed|Failed)RunIsRejected: OK (2 tests, 11 assertions) — independently re-run by parent; castor test:tui --filter=TuiJourneyE2eTest: OK (1 test, 35 assertions) — real TmuxHarness !ls -1 proof, independently re-run by parent; Fork-reported full gate (to be re-run in task-to-pr): castor check all 6 lanes green; castor test 3154 tests/10078 assertions; deptrac 0 violations; phpstan 0 errors; cs-check clean
- Summary: Fork run kz3pt5zayr3b COMPLETE. Commit 72178ea59 on task/issue-183-shell-command-run-stuck (9 files, +500/-50). All 3 root causes fixed: Fix A (ShellCommandHandler now dispatches ExecuteShellToolCall on agent.execution.bus → new ExecuteShellToolCallWorker runs bash in tool consumer process; synthetic RunStarted seq 0 preserved for cursor registration; diagnostic ProtocolError on transport failure), Fix B (start() has 3 cleanup paths — waitForRuntimeReady timeout, assertProcessStillRunning premature-exit, run_started timeout — all call stopProcess() and rethrow with structured logging), Fix C (ApplyCommandHandler guards Completed/Failed [non-retryable] before applyCancelCommand, preserving retryableFailure retry path and issue-152 pending-steer/follow_up rejection). Fork's full castor check reported green.

Parent independent verification (this session):
- git diff --stat confirmed (9 files, +500/-50).
- Fix A/B/C source reviewed — correct, comments preserved/added, no exception swallowing, no BC shims.
- Cancel-guard regression (RC3) independently re-run: 2 tests, 11 assertions OK.
- TUI E2E proof (RC1+RC2, hard gate) independently re-run: TuiJourneyE2eTest Phase 4 real !ls -1 through TmuxHarness, 1 test, 35 assertions OK.
- No stale current-user workers; root-owned messenger:consume (PID 3343) left untouched.
Status: implementation accepted. STOPPING before CODE-REVIEW per orchestrator model — awaiting user task-to-pr.

## Task workflow update - 2026-06-22T03:18:31.947Z
- Recorded fork run: yc1y9oyeyk54
- Summary: task-to-pr review phase. Reviewer investigation done directly by orchestrator (reviewer subagent ended mid-trace 3× without verdict). VERDICT: REQUEST CHANGES — 1 blocker found.

BUG (blocker): Fix A introduced an ordering regression. ShellCommandHandler dispatches ExecuteShellToolCall (async, separate consumer process) then immediately calls client->completeRun() (sync, in controller), writing AgentEnd to the shared EventStore BEFORE the worker writes tool_execution_start/end. Net order [AgentEnd seq1, tool_exec_start seq2, tool_exec_end seq3]: violates LifecycleOrderValidator (agent_end must be final, line 60/151) and causes TUI to mark run Completed before shell output arrives (drain forwards in seq order). TUI E2E test missed it (asserts marker+status, not ordering). Original sync code guaranteed order; async broke it.

NTH (promoted): no ExecuteShellToolCallWorker test — needed to guard the fix.

Other areas verified clean: Fix B logger non-nullable (safe); Fix C exhaustive (Cancelled already guarded at line 80, retryableFailure preserved); issue-152 REJECT_ON_CANCEL_KINDS intact; layering legal; no empty catches; comments preserved; TUI E2E proof is genuine TmuxHarness. InProcess sync path noted as acceptable edge (prod uses Process+async).

Fix fork launched (run yc1y9oyeyk54): move standalone AgentEnd write into ExecuteShellToolCallWorker (single-writer ordering); add standalone flag to message; remove sync completeRun from handler standalone branch; add focused worker ordering test (thesis: tool_exec + AgentEnd strict seq order). Awaiting fork completion before CODE-REVIEW.
- task-to-pr: reviewer investigation (reviewer subagent ended mid-trace; orchestrator completed the review directly). VERDICT: REQUEST CHANGES. Blocker: Fix A async dispatch races completeRun() — AgentEnd lands in EventStore before tool_execution events (violates LifecycleOrderValidator agent_end-must-be-final; TUI sees Completed before shell output). TUI E2E test didn't catch it (asserts marker+status, not ordering).
- Fix fork yc1y9oyeyk54 launched: move standalone AgentEnd write into ExecuteShellToolCallWorker (single consumer-process writer → guaranteed order); add standalone flag to ExecuteShellToolCall message; remove sync completeRun from ShellCommandHandler standalone branch; add ExecuteShellToolCallWorkerTest guarding seq ordering. Awaiting completion.

## Task workflow update - 2026-06-22T03:32:56.538Z
- Summary: Follow-up fix fork yc1y9oyeyk54 complete and VERIFIED by orchestrator. Commit 65d2ffce2 on task branch (5 files, +246/-16). Ordering race from Fix A is fixed correctly.

VERIFICATION (orchestrator ran directly):
- Commit/diff inspected: handler passes `standalone` flag, no sync `completeRun` for standalone; `complete_run` branch untouched; dispatch-failure catch emits ProtocolError (defensive, not a BC shim).
- Worker writes [tool_exec_start seq N, tool_exec_end seq N+1, agent_end seq N+2] from a single process; payload `{reason:'completed'}` matches InProcessAgentSessionClient::completeRun() exactly.
- New ExecuteShellToolCallWorkerTest (2 tests/19 assertions): real in-memory EventStore (anonymous class, not mocks), asserts strict ascending seq order, AgentEnd is final event, and non-standalone emits only tool_exec events. Thesis targets the regression directly.
- ShellCommandHandlerTest updated: line 151 asserts completeRunCalls===0 for standalone ('Handler must NOT call completeRun for standalone — worker owns AgentEnd'); complete_run branch still asserts 1 call.
- castor test --filter=ExecuteShellToolCallWorkerTest: 2/19 OK
- castor test --filter=ShellCommandHandlerTest: 8/29 OK
- castor test:tui: 14/124 OK (standalone !ls -1 flow works end-to-end)
- castor deptrac: 0 violations; castor phpstan: 0 errors
- No double-AgentEnd for standalone (JsonlProcessAgentSessionClient::shellExecute sends only shell_command).

KNOWN RESIDUAL (out of scope for #183 primary case, documented in commit msg): subsequent-shell race — SubmitListener sends shell_command then complete_run for non-standalone; complete_run writes AgentEnd sync while worker processes shell_command async. Fix needs cross-process sync or TUI protocol change. Deferred.

Ready for CODE-REVIEW.
- Follow-up fix verified directly by orchestrator (no blind acceptance). Commit 65d2ffce2: handler passes standalone flag + removes sync completeRun for standalone; worker writes [tool_exec_start, tool_exec_end, agent_end] with seqs N/N+1/N+2 in single process, payload matches completeRun() exactly.
- castor test --filter=ExecuteShellToolCallWorkerTest: 2/19 OK. castor test --filter=ShellCommandHandlerTest: 8/29 OK (line 151 asserts completeRunCalls===0 for standalone). castor test:tui: 14/124 OK. castor deptrac: 0 violations. castor phpstan: 0 errors.
- Subsequent-shell race documented as known residual (out of scope for #183 primary case). Ready for CODE-REVIEW.

## Task workflow update - 2026-06-22T03:34:39.809Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (68.7s).
- Pushed task/issue-183-shell-command-run-stuck to origin.
- branch 'task/issue-183-shell-command-run-stuck' set up to track 'origin/task/issue-183-shell-command-run-stuck'.
- Created PR: https://github.com/ineersa/agent-core/pull/193

## Task workflow update - 2026-06-22T03:34:54.234Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/193
- Updated PR Status: open
- Validation: move_task CODE-REVIEW gate: castor check passed (68.7s, all 6 lanes green) in worktree; PR created: https://github.com/ineersa/agent-core/pull/193; Branch task/issue-183-shell-command-run-stuck pushed to origin

## Task workflow update - 2026-06-22T15:07:30.783Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Review-iterate: rework issue-183 Fix A properly. The async shell-execution mechanism (parallel ExecuteShellToolCall pipeline) creates a dual-writer race on the canonical event stream and only covers the standalone path. Rework to reuse the normal tool pipeline (single ordering authority) and fix the documented subsequent-shell residual. Keep Fix B (cleanup) and Fix C (cancel guard).

## Task workflow update - 2026-06-22T15:09:23.028Z
- Recorded fork run: bqt29zmzj5fw
- task-review-iterate: moved CODE-REVIEW → IN-PROGRESS to rework Fix A
- Launched fork bqt29zmzj5fw (review-iterate) to rework shell-command async execution with a SINGLE event-stream ordering authority; evaluate Approach A (reuse normal ExecuteToolCall→ToolCallResult→orchestrator pipeline, delete parallel ExecuteShellToolCall) vs Approach B (keep dedicated path but sole-writer for both standalone+inline); fail-first repro of documented subsequent-shell ordering residual; keep Fix B + Fix C; TUI E2E ordering proof required

## Task workflow update - 2026-06-22T15:32:39.602Z
- Validation: castor test --filter=ExecuteShellToolCallWorkerTest: 3 tests, 28 assertions OK; castor test --filter=ShellCommandHandlerTest: 7 tests, 31 assertions OK; castor test:tui --filter=TuiJourneyE2eTest: 1 test, 39 assertions OK (includes Phase 9 inline-shell proof); castor deptrac: 0 violations; grep confirmed ShellCommandHandler has no completeRun calls (comments only); fork reported castor check all 6 lanes green: deptrac 0, test 3156/10111, controller-replay 3/41, tui 14/128, phpstan 0, cs-check clean
- Summary: Review-iterate complete: fork bqt29zmzj5fw reworked Fix A using Approach B (worker as sole ordering authority). Commit 07afed12e (9 files, +360/-94). Controller no longer calls completeRun() for any shell path; ExecuteShellToolCallWorker writes AgentEnd in-order for both standalone and completeAfter flags. New Phase 9 TUI E2E proof exercises inline-shell-on-completed-run + followup, with assertShellEventsOrder ordering assertion (fail-first verified). Fix B (cleanup) and Fix C (cancel guard) preserved. Orchestrator independently verified: commit/diff confirmed, controller completeRun removed (grep clean), all focused lanes green (worker 3/28, handler 7/31, TUI E2E 1/39 incl Phase 9, deptrac 0). Note: double-AgentEnd on subsequent shells is pre-existing behavior, out of scope.
- Fork bqt29zmzj5fw chose Approach B (worker sole writer) over A (reuse ExecuteToolCall pipeline) — Approach A rejected because ToolCallResultHandler requires LLM step/turn context; shell-only runs have no LLM step
- Fork added completeAfter flag to ExecuteShellToolCall; SubmitListener passes complete_after=true for shells on terminal runs instead of calling client->completeRun()
- Controller ShellCommandHandler no longer handles complete_run or depends on AgentSessionClient — solely dispatches ExecuteShellToolCall via execution.bus
- New TUI E2E Phase 9: inline !ls on completed run + followup message, with assertShellEventsOrder helper checking AgentEnd is last event
- Orchestrator verified: commit 07afed12e exists, diff stat matches, all focused tests green, deptrac clean

## Task workflow update - 2026-06-22T15:34:04.893Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (73.2s).
- Pushed task/issue-183-shell-command-run-stuck to origin.
- branch 'task/issue-183-shell-command-run-stuck' set up to track 'origin/task/issue-183-shell-command-run-stuck'.
- PR already exists: https://github.com/ineersa/agent-core/pull/193
- Summary: Review-iterate: reworked Fix A with worker as sole ordering authority (Approach B). Commit 07afed12e appends to PR #193. Controller no longer writes terminal events for shells; worker writes AgentEnd in-order for both standalone and completeAfter. New Phase 9 TUI E2E proof for inline-shell-on-completed-run + followup with ordering assertion. Fixes B/C preserved. All focused lanes verified green by orchestrator.

## Task workflow update - 2026-06-22T17:22:37.401Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Reopen: replay-based proofs (Phase 9 TUI E2E + ExecuteShellToolCallWorkerTest) do NOT exercise the real controller subprocess via JsonlProcessAgentSessionClient::start() — the exact path that hangs in live sessions (user confirmed session 4 still hangs). User directive: write a LIVE LLM controller E2E test (start_run -> shell_command -> follow_up) using ControllerSmokeTest pattern, reproduce the real hang, then fix. Castor test:controller / test:llm-real infrastructure already exists but was never used for this task.

## Task workflow update - 2026-06-22T17:23:42.669Z
- Recorded fork run: imwgnj2qbt1f
- Summary: Reopened for live LLM proof. Replay proofs passed but session 4 still hangs — they don't exercise the real controller subprocess. Launched fork imwgnj2qbt1f to write ShellFollowUpLiveE2eTest (#[Group('llm-real')]) using ControllerSmokeTest pattern: start_run -> shell_command -> follow_up against live llama.cpp:9052. Fail-first reproduction of the real hang with full diagnostics (events, messenger queue pending counts, state.json, stderr), then fix the actual root cause. Prime suspect: follow_up on a run that already has terminal AgentEnd(s). No more replay-based speculation.
- User directive: stop the replay-based circle, use live LLM test (castor test:controller / llm-real infrastructure exists but was never used)
- Fork imwgnj2qbt1f: write live E2E ShellFollowUpLiveE2eTest reproducing turn1->!ls->followup hang, capture diagnostics (esp. messenger queue pending counts), then fix real cause

## Task workflow update - 2026-06-22T18:41:00.880Z
- Summary: ROOT CAUSE ISOLATED via live log forensics (no more speculation):

LIVE EVIDENCE (test run f115b0721c90, agent-2026-06-22.log):
- Turn 1 completes normally: seq 1-5, status=completed (L65)
- Shell executes: seq 6-7, acked fine (L74-82)
- Follow_up ApplyCommand: enqueued to CacheCommandStore, agent_command_queued seq 8 committed (L89-107)
- AdvanceRun #2 dispatched via postCommit (L104), received+acked in consumer (L108-112) — BUT the turn.orchestrator.advance span (L109-110) has NO inner turn_start_boundary mailbox span, NO persistence.commit, NO events.
- Compare AdvanceRun #1 (L39-48): full chain with boundary span + commit + 2 events.
- Messenger queues ALL EMPTY post-failure. Controller still RUNNING.

CONCLUSION: AdvanceRunHandler.handle() returned an EMPTY HandlerResult for AdvanceRun #2. The only empty-result bail paths are:
  (1) AdvanceRunHandler.php line 56-57: hasUnresolvedToolCalls guard — RULED OUT (state shows pending_tool_calls=0)
  (2) AdvanceRunHandler.php line 154: boundaryEventSpecs empty → mailbox drain found nothing
Since resume branch (line 112) requires non-empty boundaryEventSpecs, and no run_resumed/turn_advanced was written, the mailbox drain in AdvanceRun #2's process yielded NOTHING despite the follow_up being enqueued.

LAST UNKNOWN (single hypothesis for the fork to confirm with a probe): CacheCommandStore::pending($runId) returns empty when called from the AdvanceRun #2 consumer process, despite the enqueue in the ApplyCommand consumer process. cache.app = cache.adapter.doctrine_dbal (shared SQLite). Possible: DBAL cache adapter cross-process commit timing — enqueue writes inside the ApplyCommand Doctrine transaction; if AdvanceRun #2 runs in a process whose cache-pool connection reads before the ApplyCommand transaction commits, pending() sees empty. OR a status/order mismatch.

SECONDARY (boyscout): MCP consumer crashes repeatedly on missing HATFIELD_MCP_TRANSPORT_DSN in PHAR test env — red herring for #183 but noisy and should be fixed (provide default/stub DSN or make MCP transport optional in test env).
- Orchestrator captured the diagnostic the fork omitted: all messenger queues EMPTY, controller alive, events.jsonl ends at seq=8 agent_command_queued
- Orchestrator read the structured log (agent-2026-06-22.log) and reconstructed the full causal timeline: AdvanceRun #2 (L108-112) returned EMPTY HandlerResult — no boundary span, no commit, no events, unlike AdvanceRun #1 (L39-48)
- Orchestrator localized bug to AdvanceRunHandler line 154 bail (boundaryEventSpecs empty) because mailbox drain found no pending follow_up despite enqueue
- Orchestrator ruled out: unresolved-tool-calls guard (pending_tool_calls=0), in-memory store (cache.app is DBAL/shared), PendingCommand serialization (clean final readonly DTO, PHP 8.2-safe), consumer crash (MCP-only, unrelated)
- Boyscout scope added: fix MCP consumer HATFIELD_MCP_TRANSPORT_DSN crash in PHAR test env

## Task workflow update - 2026-06-22T19:12:42.487Z
- Validation: Orchestrator independent verification of commit 8efbd9b7c:;   git show 8efbd9b7c: 3 files +29/-1 (RunStateReplayService.php, messenger.yaml, ControllerE2eTestCase.php);   applyToolExecutionEnd is symmetric with applyToolExecutionStart (false→true);   Worker writes tool_call_id in both start/end payloads — resolver matches;   castor test:llm-real --filter=ShellFollowUpLiveE2eTest: OK (2 tests, 13 assertions, 18.6s) — THE GATE;   castor test --filter=RunStateReplay: OK (32 tests, 136 assertions);   castor test --filter=ApplyCommandHandlerTest: OK (14 tests, 98 assertions);   castor deptrac: 0 violations;   castor test:tui --filter=TuiJourneyE2eTest: OK (1 test, 39 assertions); Fork reported castor check: all 6 lanes green (117.8s); Branch: task/issue-183-shell-command-run-stuck, clean working tree, PR #193 OPEN
- Summary: ROOT CAUSE FOUND AND FIXED. Live LLM proof passes.

ACTUAL ROOT CAUSE (different from all prior theories): RunStateReplayService::applyEvent() mapped ToolExecutionEnd to applyNoMutation. Shell commands don't go through an LLM step (which normally clears pendingToolCalls), so tool_execution_end was the ONLY resolver for shell tool calls — and it was a no-op. When AdvanceRunHandler rebuilt RunState from events for the follow_up's AdvanceRun, the shell's pending tool call (sh_*=>false from tool_execution_start) stayed unresolved, tripping the unresolved-tool-call guard (AdvanceRunHandler line 57) → empty HandlerResult → no events → run appeared dead.

This was proven by runtime probes across consumer processes: enqueue succeeded, pending() returned the follow_up, but AdvanceRunHandler bailed at line 57 with sh_* still false.

FIX (commit 8efbd9b7c, 3 files +29/-1): Added applyToolExecutionEnd handler in RunStateReplayService that sets pendingToolCalls[tool_call_id]=true — symmetric pair with applyToolExecutionStart (which sets false). This resolves ALL standalone/non-LLM-step tool calls in state replay.

BOYSCOUT: MCP consumer crashed on missing HATFIELD_MCP_TRANSPORT_DSN in PHAR test env. Fixed by adding env default to messenger.yaml + explicit env in ControllerE2eTestCase::spawnController().

PROOF: ShellFollowUpLiveE2eTest (#[Group('llm-real')]) — start_run → shell_command → follow_up against real llama.cpp:9052 via real controller subprocess. Was RED (hang), now GREEN (2 tests, 13 assertions). Asserts assistant responds + run.completed after follow_up.

OPEN DECISIONS FOR REVIEWER (not blockers — tests pass, all gates green):
1. RunResumed emitted from BOTH UserMessageHandler AND AdvanceRunHandler — assess if duplicative
2. completeAfter flag in ExecuteShellToolCall DTO — worker ignores it (only checks standalone). Dead code or needed for future?
3. Branch has 4 stacked commits from debugging journey (72178ea59 async pipeline, 07afed12e ordering, ef6140875 cursor lifecycle, 8efbd9b7c real fix). Each is individually defensible but the total surface is large (1352 lines).
- Orchestrator independently verified the real root cause fix: ToolExecutionEnd was mapped to applyNoMutation in RunStateReplayService, leaving shell tool calls unresolved in state replay. Fix adds applyToolExecutionEnd handler (symmetric with applyToolExecutionStart).
- Orchestrator ran the live test personally: ShellFollowUpLiveE2eTest passes (2 tests, 13 assertions) — first time the exact user scenario (start_run → !ls → follow_up) passes against real llama.cpp via real controller subprocess
- Orchestrator confirmed all focused lanes green: RunStateReplay 32/136, ApplyCommandHandler 14/98, deptrac 0 violations, TUI E2E 1/39
- Orchestrator confirmed MCP boyscout fix: live test runs clean, no HATFIELD_MCP_TRANSPORT_DSN crashes
- Open decisions documented for reviewer: RunResumed duplication, completeAfter dead code, 4-commit debugging journey surface area

## Task workflow update - 2026-06-22T19:14:15.508Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (73.2s).
- Pushed task/issue-183-shell-command-run-stuck to origin.
- branch 'task/issue-183-shell-command-run-stuck' set up to track 'origin/task/issue-183-shell-command-run-stuck'.
- PR already exists: https://github.com/ineersa/agent-core/pull/193

## Task workflow update - 2026-06-22T19:50:06.065Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: task-review-iterate: reviewer APPROVED with non-blocking cleanups. Launching cleanup fork to remove speculative/dead code before final merge: (1) RunResumed event type + all emitters/mappers, (2) completeAfter dead flag from DTO + handlers + SubmitListener + tests, (3) testFollowUpWithoutShell placeholder, (4) add missing unit test for shell-only ToolExecutionEnd resolution.

## Task workflow update - 2026-06-22T20:10:02.803Z
- Validation: Orchestrator verification of cleanup commit 9eff57bbe:;   git show --stat: 15 files +161/-176 (net -15 lines);   Fork deviation verified correct: RuntimeEventTypeEnum::RunResumed (TUI protocol) is pre-existing in origin/main (lines 109,133), ResumeHandler:51 consumes it;   grep RunEventTypeEnum::RunResumed in src/tests: clean (zero);   grep completeAfter/complete_after in src/tests/config: clean (zero);   grep run.resumed in src/AgentCore + tests/AgentCore: clean (domain fully removed);   Placeholder assertNotEmpty('not yet implemented') gone;   New unit test testToolExecutionEndResolvesPendingShellToolCallWithoutLlmStep: real assertions (assertNotContains false in pendingToolCalls), OK (1 test, 4 assertions);   New isolation test testFollowUpWithoutShell: real start_run → follow_up flow (no shell), passes live;   castor test:llm-real --filter=ShellFollowUpLiveE2eTest: OK (2 tests, 21 assertions, 23.8s);   castor deptrac: 0 violations;   castor phpstan: 0 errors; Fork reported castor check: all 6 lanes green (121.6s)
- Summary: CLEANUP COMPLETE + VERIFIED. Commit 9eff57bbe, 15 files +161/-176 (net -15 lines).

4 reviewer-requested changes all done:
1. Removed domain RunEventTypeEnum::RunResumed + emitters (UserMessageHandler, AdvanceRunHandler) + translator mapping. Preserved pre-existing TUI-protocol RuntimeEventTypeEnum::RunResumed (origin/main) consumed by ResumeHandler:51 for normal session resume — reviewer had conflated the two enums; fork correctly distinguished them.
2. Removed dead completeAfter flag from DTO + 3 propagation sites + 2 redundant tests. Zero residual references.
3. Implemented testFollowUpWithoutShell as a real isolation test (start_run → follow_up, no shell) — proves follow_up path is healthy independently. Placeholder gone.
4. Added testToolExecutionEndResolvesPendingShellToolCallWithoutLlmStep unit test — the regression guard for the #183 bug: asserts pendingToolCalls has no false values after shell-only ToolExecutionStart→End without an LLM step.

ORCHESTRATOR VERIFICATION (independent):
- Commit 9eff57bbe exists, diff stat matches (15 files, +161/-176)
- Fork deviation verified correct: RuntimeEventTypeEnum::RunResumed (TUI protocol) IS in origin/main (lines 109, 133), ResumeHandler:51 DOES consume it — preserving it was right, would have broken session resume otherwise
- grep confirms ZERO residual: no RunEventTypeEnum::RunResumed in domain, no completeAfter/complete_after anywhere
- castor test --filter=testToolExecutionEndResolves...: OK (1 test, 4 assertions)
- castor deptrac: 0 violations
- castor phpstan: 0 errors
- castor test:llm-real --filter=ShellFollowUpLiveE2eTest: OK (2 tests, 21 assertions, 23.8s) — both isolation + full scenario pass
- Fork reported castor check: all 6 lanes green (121.6s)

BRANCH STATE: 5 stacked commits on task/issue-183-shell-command-run-stuck (72178ea59 async pipeline, 07afed12e ordering, ef6140875 cursor lifecycle, 8efbd9b7c real fix, 9eff57bbe cleanup). Net surface reduced from 1352 to ~1337 lines vs origin/main. Mergeable.
- Orchestrator verified cleanup commit 9eff57bbe: 15 files +161/-176, all 4 reviewer-requested changes real and complete
- Orchestrator confirmed fork's deviation was correct: preserved pre-existing RuntimeEventTypeEnum::RunResumed (TUI protocol) that ResumeHandler:51 consumes for normal session resume — reviewer had conflated domain RunEventTypeEnum with TUI-protocol RuntimeEventTypeEnum; blindly removing all RunResumed would have broken resume
- Orchestrator verified zero residual references: no RunEventTypeEnum::RunResumed in domain, no completeAfter/complete_after anywhere, placeholder test gone
- Orchestrator ran new unit test (1/4 OK), deptrac (0 violations), phpstan (0 errors), live test (2 tests/21 assertions, 23.8s)
- Branch now has 5 stacked commits, mergeable, net surface reduced ~15 lines

## Task workflow update - 2026-06-22T20:11:54.951Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (71.5s).
- Pushed task/issue-183-shell-command-run-stuck to origin.
- branch 'task/issue-183-shell-command-run-stuck' set up to track 'origin/task/issue-183-shell-command-run-stuck'.
- PR already exists: https://github.com/ineersa/agent-core/pull/193

## Task workflow update - 2026-06-22T20:18:43.112Z
- Moved CODE-REVIEW → DONE.
- Merged task/issue-183-shell-command-run-stuck into integration checkout.
- Auto-merging config/services.yaml
Merge made by the 'ort' strategy.
 config/packages/messenger.yaml                     |   4 +
 config/services.yaml                               |   7 +
 .../Application/Handler/RunStateReplayService.php  |  26 +-
 .../Application/Pipeline/ApplyCommandHandler.php   |  18 ++
 .../Domain/Message/ExecuteShellToolCall.php        |  50 ++++
 .../CommandHandler/ExecuteShellToolCallWorker.php  | 167 +++++++++++++
 .../CommandHandler/ShellCommandHandler.php         |  92 ++++---
 .../CommandHandler/UserMessageHandler.php          |   1 +
 .../Runtime/Controller/RuntimeEventEmitter.php     |  30 +--
 .../Process/JsonlProcessAgentSessionClient.php     |  76 ++++--
 src/Tui/Listener/SubmitListener.php                |  39 +--
 .../Handler/RunStateReplayServiceTest.php          |  54 +++++
 .../Pipeline/ApplyCommandHandlerTest.php           | 133 ++++++++++
 .../ExecuteShellToolCallWorkerTest.php             | 184 ++++++++++++++
 .../CommandHandler/ShellCommandHandlerTest.php     | 130 ++++++----
 .../Controller/E2E/ControllerE2eTestCase.php       |   1 +
 .../Controller/E2E/ShellFollowUpLiveE2eTest.php    | 269 +++++++++++++++++++++
 tests/Tui/E2E/TuiJourneyE2eTest.php                | 180 +++++++++++++-
 tests/Tui/E2E/fixtures/tui-followup-response.json  |  33 +++
 19 files changed, 1352 insertions(+), 142 deletions(-)
 create mode 100644 src/AgentCore/Domain/Message/ExecuteShellToolCall.php
 create mode 100644 src/CodingAgent/Runtime/Controller/CommandHandler/ExecuteShellToolCallWorker.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/CommandHandler/ExecuteShellToolCallWorkerTest.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/E2E/ShellFollowUpLiveE2eTest.php
 create mode 100644 tests/Tui/E2E/fixtures/tui-followup-response.json
- Removed worktree /home/ineersa/projects/agent-core-worktrees/issue-183-shell-command-run-stuck.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/issue-183-shell-command-run-stuck.
- Pulled integration checkout: Merge made by the 'ort' strategy..
