# P0: Recover stale running sessions after LLM worker crash and rewind orphan tool messages

## Goal
Live RENDER-05B smoke exposed a session integrity/runtime recovery bug in session 1. Evidence preserved at `/home/ineersa/.hatfield/dumps/session-1-render05b-corrupt-20260701T183010Z`.

Observed sequence:
- Session 1 became stuck Working after resume: `state.json` status=running, turnNo=27, lastSeq=318, activeStepId=follow_up-13503477865733.
- Events ended after follow_up applied + turn_advanced + leaf_set; no llm_step_completed.
- Logs showed ExecuteLlmStep sent to Doctrine llm transport, then llm consumer crashed with `include(): zlib: data error`; messenger row was marked delivered, so work was lost in-flight.
- Resume/attach is passive and did not continue/re-dispatch, leaving infinite Working.
- User then tried go-back/rewind and hit provider history error: `Tool-call sequence violation at message 10: orphan tool message with tool_call_id="call_00_Zm7aROqgBCMbqsuWtGpr0544" (no open batch expecting it).`

This appears to be a P0 session recovery / rewind history integrity issue, separate from transcript styling. Need preserve user session history and prevent dead sessions after worker crashes.

## Acceptance criteria
- Detect stale running sessions on resume/attach where activeStepId has turn_advanced/leaf_set but no llm_step_completed/failure and no live in-flight work; recover safely or surface an actionable error instead of infinite Working.
- Handle delivered-but-lost ExecuteLlmStep after llm worker crash: either idempotently re-dispatch the active step when no canonical LLM/tool output exists for it, or mark run retryable with a clear continue/retry path.
- Ensure rewind/go-back never constructs LLM-visible history with orphan tool result messages; if a branch would contain a tool result without its assistant tool call, filter/repair/reject with recoverable session error.
- Add regression coverage using the preserved dump or distilled fixtures for: stale running after worker crash on resume, and rewind after stale/incomplete turn not producing orphan tool messages.
- Do not mutate or discard existing session events during recovery without backup-first repair flow; document any manual recovery command if needed.
- Validation includes Castor-only focused tests plus full relevant runtime/controller checks; live/replay mismatch must be considered per AGENTS guidance.

## Workflow metadata
Status: DONE
Branch: task/fix-stale-running-session-recovery-orphan-tool-messages
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-stale-running-session-recovery-orphan-tool-messages
Fork run: bhm9d7865su9
PR URL: https://github.com/ineersa/agent-core/pull/359
PR Status: merged
Started: 2026-08-04T20:24:09.964Z
Completed: 2026-08-04T22:56:25.845Z

## Work log
- Created: 2026-07-01T18:30:33.836Z

## Task workflow update - 2026-07-01T18:31:13.590Z
- Recorded fork run: kajmm9gjii3e
- Validation: Evidence preserved in dump: /home/ineersa/.hatfield/dumps/session-1-render05b-corrupt-20260701T183010Z; Evidence: current state after go-back status=running, turnNo=27, lastSeq=327, messages=78; index 10 is orphan tool result call_00_Zm7aROqgBCMbqsuWtGpr0544; Evidence: events seq 24 llm_step_completed contains assistant text plus top-level tool_calls including call_00_Zm7aROqgBCMbqsuWtGpr0544; seq 35 message_end contains matching tool result; Evidence: RunStateReplayService::replayAssistantMessage drops top-level tool_calls when AgentMessage::fromPayload succeeds on text content; Evidence: logs show rewind to turn 26 rebuilt completed state then next LLM failed with MalformedToolCallSequenceException; second rewind to turn 27 rebuilt running state with stale activeStepId
- Summary: Read-only orphan tool-message diagnosis completed. Session events are recoverable; hot state after rewind is invalid. Exact replay bug: RunStateReplayService::replayAssistantMessage() calls AgentMessage::fromPayload($payload) and returns early for assistant messages with text content, so top-level llm_step_completed.tool_calls are not copied into AgentMessage metadata. For tool-call-only assistant payloads, fromPayload fails and fallback preserves tool_calls, so only text+tool_calls assistant turns are affected. In session 1, seq 24 has assistant text + 3 tool_calls including call_00_Zm7aROqgBCMbqsuWtGpr0544; after rewind/rebuild, state message index 9 is assistant text with empty metadata.tool_calls, while message index 10 is the matching tool result, causing AgentMessageToolCallSequenceValidator to throw orphan tool message. Go-back triggered rebuild and exposed the replay bug; events.jsonl is not missing data. Compounding issue remains: stale running turn 27 after lost ExecuteLlmStep due to llm consumer crash.

## Task workflow update - 2026-07-01T18:35:05.466Z
- Recorded fork run: 6crrqt625e38
- Validation: Commit e16d34126 in /home/ineersa/projects/agent-core-worktrees/render-05b-transcript-lifecycle-hitl-overlay-polish; castor test --filter='testReplayAssistantTextWithTopLevelToolCallsPreservesMetadataForValidator|testReplayToolCallPath|testReplayToolCallOnlyAssistantMessage' OK (3 tests, 22 assertions); castor deptrac OK (0 violations); castor phpstan OK (no errors); castor cs-check OK (0 files fixed)
- Summary: Minimal replay fix implemented in current RENDER-05B worktree at commit e16d34126 (`fix(agent-core): preserve replayed assistant tool calls`). RunStateReplayService::replayAssistantMessage() now copies non-empty top-level llm_step_completed.tool_calls into AgentMessage metadata when AgentMessage::fromPayload() succeeds on text-bearing assistant messages, mirroring live ReplayService / AgentMessageNormalizer semantics. Added regression test proving assistant text + tool_calls followed by matching tool message_end rebuilds into validator-safe history for the session-1 orphan id pattern. Current state.json may remain corrupt because rebuildIfStale skips when state.lastSeq >= maxEventSeq; user must trigger rewind/rebuild again or future repair command. Stale running/lost ExecuteLlmStep recovery remains open.

## Task workflow update - 2026-07-01T18:37:23.461Z
- Summary: Live retest after replay fix e16d34126: orphan tool-call error appears resolved, but session 1 still hangs at `◐ Working...`. This confirms the remaining P0 is stale running / lost in-flight ExecuteLlmStep recovery. Current state likely still status=running with activeStepId from turn 27 and no active LLM work consumed.

## Task workflow update - 2026-07-01T18:42:48.830Z
- Recorded fork run: t8ceu0vdonnp
- Validation: Commit e7cd0c3e8 in /home/ineersa/projects/agent-core-worktrees/render-05b-transcript-lifecycle-hitl-overlay-polish; castor test --filter='testResumeInfersCancelledActivityFromLatestAgentEnd|testReplayAgentEndCancelled' OK (2 tests, 10 assertions); castor deptrac OK (0 violations); castor phpstan OK (no errors); castor cs-check OK (0 files fixed)
- Summary: Additional current-worktree fix committed at e7cd0c3e8 (`fix(tui): clear working state after terminal resume`). After user retest, worktree session 1 hot state was status=cancelled turnNo=29 lastSeq=339 activeStepId=null, but UI could still show Working because passive resume reset active mid-turn state to Idle without inferring latest full-stream agent_end terminal state, resume did not clear optimistic working text before ticks, and replay applyAgentEnd preserved activeStepId. Fix: SessionInitializer infers terminal activity from latest canonical agent_end in full event stream after passive resume reset; InteractiveMode clears working message on passive resume when activity is not active; RunStateReplayService::applyAgentEnd clears activeStepId. This addresses cancelled/terminal resume Working mismatch, not original stale running lost ExecuteLlmStep recovery.

## Task workflow update - 2026-07-01T18:44:36.665Z
- Summary: Live retest after e7cd0c3e8 still shows user message followed by `◐ Working...` and cannot resume. User prompt: `Want to test bash in parallel, cah you explore docs directory, launch few bash commands in parallel`. Need further diagnose whether follow_up on cancelled session sets optimistic Working and command rejection/terminal state is not propagated, or whether a new running turn was created and ExecuteLlmStep again is not consumed.

## Task workflow update - 2026-07-01T18:47:14.361Z
- Summary: Further live retest evidence: after e7cd0c3e8, session 1 again advanced to running turnNo=30 lastSeq=343 activeStepId=follow_up-16054675060987. Logs at 18:43:30 show ExecuteLlmStep sent to llm queue, then at 18:43:35 an older DD_VERSION=6aed5b378 llm#0 consumer crashed with `include(): zlib: data error`. Process snapshot shows two simultaneous user-owned controllers for the same worktree/session/queue: PID 97272 (DD_VERSION=6aed5b378, HATFIELD_SESSION_ID=1) and PID 192289 (DD_VERSION=e7cd0c3e8, HATFIELD_SESSION_ID=1), plus consumers for both sharing llm_1/run_control_1 queues. This explains repeated lost LLM steps: stale older controller/consumer can consume the new session's queued ExecuteLlmStep and crash. Immediate recovery requires stopping duplicate/stale session-1 controllers by normal TUI exit/user action; code fix should enforce single controller/session ownership and stale worker detection/recovery.

## Task workflow update - 2026-07-01T19:02:08.020Z
- Recorded fork run: bhm9d7865su9
- Summary: Remaining live issue refined: after stopping old SSH TUI process and successfully going back to safe turn/resume, user still sees stale pending queued line `⏳ Want to test bash in parallel...`. Also reports normal idle follow-up briefly renders as queued then user message. Fork bhm9d7865su9 launched in current RENDER-05B worktree to fix pending queue/projection cleanup and idle follow-up queued flicker.

## Task workflow update - 2026-08-04T20:24:09.964Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-stale-running-session-recovery-orphan-tool-messages.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-stale-running-session-recovery-orphan-tool-messages.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-stale-running-session-recovery-orphan-tool-messages.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-stale-running-session-recovery-orphan-tool-messages.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-stale-running-session-recovery-orphan-tool-messages.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-stale-running-session-recovery-orphan-tool-messages.
- Summary: Task-start approved. Finalized minimal scope: use Symfony Messenger Doctrine keepalive plus shorter abandoned-message redelivery timeout so delivered-but-unacked LLM work is reclaimed after worker/controller failure; enforce single live controller owner per project/session to prevent duplicate consumers sharing session queues; preserve explicit cancellation and existing canonical idempotency; no new user-facing setting, command, repair format, or custom heartbeat/outbox/reconciler. Previously merged replay/tool-history fixes satisfy the orphan-tool acceptance seam; this phase targets remaining stale-running transport recovery.

## Task workflow update - 2026-08-04T20:50:20.361Z
- Validation: Forks explicitly read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before runtime/Messenger/test work.; `castor test --filter='ConsumerSupervisorTest|JsonlProcessAgentSessionClientTransportDsnTest|HeadlessControllerLlmWorkerCountResolutionTest|MessengerDoctrineRedeliverTimeoutLeaseTest'` — OK (13 tests, 75 assertions).; `castor test:controller-replay` — OK (12 tests, 165 assertions); proves second controller for same project/session exits nonzero without runtime.ready while first remains healthy.; `castor deptrac` — 0 violations.; `castor phpstan` — 0 errors.; `castor cs-check` — clean.; No full `castor check` run during task-start, per workflow. No leaked controller/consumer processes observed.
- Summary: Task-start implementation complete at HEAD edf831d5135bc300a5a0577e6dee7d3ab6576fe8 (core fix c0e06fe4b, non-destructive origin/main topology merge 3bdbdbfa7, Ponytail cleanup edf831d51). Implemented Symfony Messenger `--keepalive=5` for all supervised consumers, fixed `redeliver_timeout=60` on all session-scoped Doctrine transport DSNs and E2E parity, pcntl fail-fast before consumer launch, and a process-lifetime Symfony flock keyed by canonical project CWD + session ID acquired before orphan reaping and released after consumer shutdown. Duplicate controllers now fail before runtime.ready. No recovery UI, setting, migration, outbox, custom heartbeat, reconciler, or orphan-tool changes. Existing orphan-tool replay regression remains present. Branch clean; origin/main is the single merge base; intended delta 11 files (+459/-20).

## Task workflow update - 2026-08-04T21:22:36.524Z
- Validation: Reviewer read root AGENTS.md, testing skill, tests/AGENTS.md, task spec, full diff, Symfony Messenger/Lock internals; final verdict APPROVE WITH SUGGESTIONS (no blockers).; `castor test` — OK (4436 tests, 16513 assertions).; `castor test:controller-replay` after final lock fix — OK (12 tests, 165 assertions).; `castor deptrac` — 0 violations.; `castor phpstan` — 0 errors.; `castor cs-check` — clean.; Final focused lock tests — OK (5 tests, 12 assertions).
- Summary: Task-to-PR review completed at HEAD def2fefc6de5bf747f01162c860488ef373e7fbd. Initial reviewer found one blocking cross-install lock gap: FrameworkBundle default FlockStore is partitioned by kernel.project_dir, allowing source/worktree/PHAR controllers for the same project/session to miss each other. Fixed with a dedicated HeadlessController-only Symfony FlockStore rooted at `%app.cwd%/.hatfield/tmp/controller-locks` (commit def2fefc6); global lock factory unchanged. Re-review APPROVED WITH SUGGESTIONS, no blockers; specification fidelity and Ponytail minimality pass. Nice-to-have test simplifications intentionally skipped.

## Task workflow update - 2026-08-04T21:25:38.140Z
- Validation: Failed gate artifact: `var/reports/qa-20260804-212244-543507-1f7dd9a3/check-test:tui.log` — 33 tests, 212 assertions, 1 unrelated failure.; `castor clean:cleanup:workers:list` — no stale QA worker candidates.; `castor test:tui --filter=testOmBackgroundStatusAppearsOnceNotInFooter` — OK (1 test, 3 assertions).
- Summary: First CODE-REVIEW gate attempt failed only in unrelated existing TUI test `TuiOmCommandsE2eTest::testOmBackgroundStatusAppearsOnceNotInFooter` (footer candidate line missing from viewport). Task delta does not touch the failing extension/TUI files. Leak diagnostic found no stale QA workers; focused Castor rerun passed (1 test, 3 assertions), confirming transient viewport flake. Retrying deterministic gate unchanged.

## Task workflow update - 2026-08-04T21:28:00.020Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (128.2s).
- Pushed task/fix-stale-running-session-recovery-orphan-tool-messages to origin.
- branch 'task/fix-stale-running-session-recovery-orphan-tool-messages' set up to track 'origin/task/fix-stale-running-session-recovery-orphan-tool-messages'.
- Created PR: https://github.com/ineersa/agent-core/pull/359

## Task workflow update - 2026-08-04T21:28:04.245Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/359
- Updated PR Status: open
- Validation: Deterministic `castor check` — passed in 128.2s.; PR: https://github.com/ineersa/agent-core/pull/359
- Summary: Task-to-PR complete. Final reviewer approved with no blockers after dedicated project-CWD lock-store fix. Deterministic Castor gate passed on retry; branch pushed and PR #359 opened.

## Task workflow update - 2026-08-04T22:56:25.846Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-stale-running-session-recovery-orphan-tool-messages into integration checkout.
- Auto-merging src/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClient.php
Auto-merging tests/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClientTransportDsnTest.php
Merge made by the 'ort' strategy.
 config/services.yaml                               |  20 ++
 config/services_test.yaml                          |   7 +
 .../Runtime/Controller/ConsumerSupervisor.php      |  37 +++-
 .../Runtime/Controller/HeadlessController.php      |  77 +++++++
 .../Process/JsonlProcessAgentSessionClient.php     |  14 +-
 .../MessengerDoctrineRedeliverTimeoutLeaseTest.php |  82 +++++++
 .../Runtime/Controller/ConsumerSupervisorTest.php  |   1 +
 .../Controller/E2E/ControllerE2eTestCase.php       |  12 +-
 .../Controller/E2E/ControllerReplayE2eTestCase.php |  12 +-
 ...dlessControllerLlmWorkerCountResolutionTest.php |   4 +
 ...adlessControllerSessionOwnerLockFactoryTest.php |  65 ++++++
 ...adlessControllerSessionOwnerLockProcessTest.php | 240 +++++++++++++++++++++
 ...nlProcessAgentSessionClientTransportDsnTest.php |   1 +
 13 files changed, 552 insertions(+), 20 deletions(-)
 create mode 100644 tests/CodingAgent/Messenger/MessengerDoctrineRedeliverTimeoutLeaseTest.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/HeadlessControllerSessionOwnerLockFactoryTest.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/HeadlessControllerSessionOwnerLockProcessTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-stale-running-session-recovery-orphan-tool-messages.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-stale-running-session-recovery-orphan-tool-messages.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User confirmed PR #359 merged and requested task completion.

## Task workflow update - 2026-08-04T22:58:25.844Z
- Validation: `LLM_MODE=true castor check` — quality OK.; Unit/integration: 4438 tests, 16547 assertions.; Controller replay: 12 tests, 165 assertions.; TUI replay: 34 tests, 222 assertions.; Live LLM: 13 tests, 144 assertions.; Deptrac 0 violations; PHPStan 0 errors; CS clean.; llama-proxy cache stable 223→223; QA artifact integrity and leak checks OK; 6 exact-run cache roots removed.; PHAR rebuilt and smoke checks passed.
- Summary: PR #359 merged; task moved to DONE; worktree and IDEA exclusions removed; integration checkout synchronized. Post-merge full runtime gate passed.
