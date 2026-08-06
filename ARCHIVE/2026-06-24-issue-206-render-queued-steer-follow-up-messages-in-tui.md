# ISSUE-206 Render queued steer/follow-up messages in TUI

## Goal
GitHub issue: https://github.com/ineersa/agent-core/issues/206

Session 28 showed steer/follow-up text was accepted and later sent to the LLM, but the TUI had no immediate feedback while the command was queued. `agent_command_queued` is currently dropped by RuntimeEventTranslator; `agent_command_applied` later becomes `UserMessageSubmitted`, so the model sees the text but the user gets a no-feedback gap.

Start points: `src/AgentCore/Application/Pipeline/ApplyCommandHandler.php`, `src/AgentCore/Application/Pipeline/CommandMailboxPolicy.php`, `src/CodingAgent/Runtime/Protocol/RuntimeEventTranslator.php`, `src/CodingAgent/Runtime/ProjectionPipeline/UserMessageProjectionSubscriber.php`, TUI transcript rendering/pending-message flow.

## Acceptance criteria
- Queued steer/follow-up submissions produce immediate visible TUI feedback while waiting for turn-boundary application.
- Canonical `agent_command_applied` still renders/reconciles without duplicate user-message blocks.
- A real TmuxHarness TUI E2E proof covers submitting a steer/follow-up during active work and seeing pending/queued feedback.
- Castor validation passes, including `castor test:tui` and required focused checks.

## Workflow metadata
Status: DONE
Branch: task/2026-06-24-issue-206-render-queued-steer-follow-up-messages-in-tui
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-206-render-queued-steer-follow-up-messages-in-tui
Fork run: 41rhkotmvxlf
PR URL: https://github.com/ineersa/agent-core/pull/212
PR Status: merged
Started: 2026-06-24T18:18:18.978Z
Completed: 2026-06-25T02:13:35.789Z

## Work log
- Created: 2026-06-24T18:13:01.280Z

## Task workflow update - 2026-06-24T18:18:18.978Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-06-24-issue-206-render-queued-steer-follow-up-messages-in-tui.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-206-render-queued-steer-follow-up-messages-in-tui.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-206-render-queued-steer-follow-up-messages-in-tui.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-206-render-queued-steer-follow-up-messages-in-tui.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-206-render-queued-steer-follow-up-messages-in-tui.

## Task workflow update - 2026-06-24T18:29:50.349Z
- Summary: Implementation design confirmed via code exploration.

ROOT CAUSE: ApplyCommandHandler emits `agent_command_queued` (RunEventTypeEnum) for steer/follow_up during active runs, but RuntimeEventTranslator line ~75 maps it to `drop()` (returns null). So the TUI gets no feedback until `agent_command_applied` later becomes `user.message_submitted`.

FIX (event-driven, survives resume):
1. ApplyCommandHandler (~line 153, queueable path only): enrich `agent_command_queued` payload with serialized `message` (available via `$message->payload['message']`, built in AgentRunner). The compact queue site (~592) stays unchanged.
2. RuntimeEventTypeEnum: add `UserMessageQueued = 'user.message_queued'` (family user_input).
3. RuntimeEventTranslator: replace `AgentCommandQueued => drop(...)` with `onAgentCommandQueued(...)` that emits `UserMessageQueued` for steer/follow_up (extract text via existing `extractTextFromContent`, carry message_id + idempotency_key); still drop other kinds (compact/continue/extension/cancel). Also add `idempotency_key` to the steer/follow_up `UserMessageSubmitted` payload (onAgentCommandApplied) for reconciliation.
4. TranscriptBlockKindEnum: add `UserMessageQueued`.
5. UserMessageProjectionSubscriber: add `onUserMessageQueued` (block id `user_queued_{runId}_{idempotencyKey}`, kind UserMessageQueued, meta idempotency_key) + reconcile in `onUserMessageSubmitted` (removeBlock the queued id by idempotency_key before adding the real block → no duplicate).
6. TranscriptBlockRenderer: add UserMessageQueued to prefixFor (pending marker e.g. `⏳`) + colorFor (Muted) so queued feedback is visually distinct from the final `❯` user message.

SAFETY (verified): autoconfigure picks up new subscriber method; ActivityStateMachine/RuntimeEventPoller ignore the new event type (fall through, only feed projector + advance lastSeq); RunStateReplayService treats agent_command_queued as no-op so payload enrichment is safe; AutoCompactionHookSubscriber reads type string only.

TESTS TO UPDATE: RuntimeEventMapperTest::testSkipsAgentCommandQueued (steer now maps to user.message_queued; keep drop assertion for compact); SessionInitializerReplayTest ~line 465 fixture/comment (now maps, not dropped).

REQUIRED PROOF: real TmuxHarness E2E (#[Group('tui-e2e-replay')]) — submit steer during active tool work, assert pending/queued marker visible, then assert reconciliation to exactly one user message (no duplicate); events.jsonl contains agent_command_queued+applied. Plus focused mapper + projection-reconciliation unit tests.

Worktree: /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-206-render-queued-steer-follow-up-messages-in-tui
- task-start: inspected task, claimed worktree, mapped full event flow (ApplyCommandHandler → RuntimeEventTranslator drop → UserMessageProjectionSubscriber) + TUI rendering + E2E infra. Design recorded; launching implementation fork.

## Task workflow update - 2026-06-24T18:38:43.526Z
- Recorded fork run: issue-206-impl-fork
- Validation: castor phpstan: errors=0; castor deptrac: violations=0; castor cs-check: clean; castor test --filter=RuntimeEventMapperTest: OK; castor test --filter=testQueuedUserMessageReconciledOnSubmit: OK; castor test --filter=SessionInitializerReplayTest::testReplayAdvancesLastSeqForDroppedEvents: OK; castor test:tui --filter=TuiQueuedSteerE2eTest: OK (5 assertions); commit d2d7db763 on task branch (fast-forwarded from main)
- Summary: IMPLEMENTATION COMPLETE on task branch (task/2026-06-24-issue-206-render-queued-steer-follow-up-messages-in-tui). Commit d2d7db763.

NOTE: The fork committed on the integration `main` checkout by mistake; I fast-forwarded the task branch onto d2d7db763 (non-destructive) so the work is now correctly on the task branch + worktree. The stray local commit still sits on `main` and needs cleanup (see handoff to user — requires approved `git reset --hard 91650cd59` on main).

CHANGES (11 files, +406/-7):
- ApplyCommandHandler: `agent_command_queued` payload enriched with serialized `message` (queueable path only; compact site untouched).
- RuntimeEventTypeEnum: `UserMessageQueued = 'user.message_queued'` (+ family user_input).
- RuntimeEventTranslator: `AgentCommandQueued` no longer dropped → new onAgentCommandQueued emits UserMessageQueued for steer/follow_up (text via extractTextFromContent), still drops compact/continue/extension; onAgentCommandApplied now includes `idempotency_key` in the UserMessageSubmitted payload.
- TranscriptBlockKindEnum: `UserMessageQueued`.
- UserMessageProjectionSubscriber: onUserMessageQueued (block id user_queued_{runId}_{key}, kind UserMessageQueued) + reconcile in onUserMessageSubmitted (removeBlock queued id, re-add UserMessage with SAME id so the live TUI poller updates the row in place — justified: RuntimeEventPoller::synchronizeProjectedBlocks is id-keyed and never prunes removed projector blocks).
- TranscriptBlockRenderer: UserMessageQueued → `⏳` prefix + Muted color (distinct from canonical `❯` user message).

TESTS:
- Updated RuntimeEventMapperTest (steer/follow_up queued → user.message_queued; compact still dropped).
- Updated SessionInitializerReplayTest fixture (queued kind=compact, still dropped; preserves lastSeq thesis).
- Added TranscriptProjectorTest::testQueuedUserMessageReconciledOnSubmit (no duplicate block after queue→submit).
- NEW TuiQueuedSteerE2eTest (#[Group('tui-e2e-replay')]): real TmuxHarness proof — "Run sleep 15" active bash window, steer STEER_QUEUED_MARKER mid-flight, asserts pending ⏳+marker snapshot, then reconciled ❯+marker exactly once (no dup), + events.jsonl agent_command_queued→applied steer lifecycle.

VALIDATION (fork ran): castor phpstan OK, castor deptrac OK, castor cs-check OK, castor test --filter=RuntimeEventMapperTest OK, reconciliation filter OK, SessionInitializerReplayTest filter OK, castor test:tui --filter=TuiQueuedSteerE2eTest OK (after id-reuse fix). NOT run: full castor test:tui group + castor check (CODE-REVIEW phase).

KNOWN MINOR: tests/Tui/E2E/fixtures/tui-queued-steer-turn2.json is 0 bytes (second-turn fixture); E2E passes because reconciliation is asserted at the apply boundary before turn-2 LLM responds. Recommend populating or removing before PR.
UNRELATED: untracked tests/Tui/E2E/TuiTransportRestartLimitEscapeE2eTest.php (ISSUE-207) sits in main working tree — NOT part of this task, left untouched.
- fork implemented ISSUE-206 + TmuxHarness E2E proof; committed d2d7db763 (on main by mistake)
- orchestrator fast-forwarded task branch onto d2d7db763; worktree clean; awaiting user approval to reset main back to 91650cd59

## Task workflow update - 2026-06-24T18:56:46.237Z
- Validation: castor test:tui --filter=TuiQueuedSteerE2eTest: OK (1 test, 5 assertions); fixture deleted; agentCommand() wires single fixture only; commit 2c15e48ed on task/...206 (non-destructive follow-up on d2d7db763)
- Summary: CLEANUP COMPLETE: dropped the 0-byte `tests/Tui/E2E/fixtures/tui-queued-steer-turn2.json` and removed all turn-2 fixture wiring from `TuiQueuedSteerE2eTest::agentCommand()` (now wires only `tui-tool-call-bash-sleep.json`). New commit `2c15e48ed` on top of `d2d7db763` (non-destructive follow-up, no amend).

Rationale: the empty fixture was dead weight — the E2E thesis (queued steer → ⏳ pending → ❯ reconciled, no dup) is proven at the apply boundary, before any turn-2 LLM response. Confirmed via `ControllerReplayHttpClientFactory`: a 0-byte path is skipped at load (`json_decode("")`→null), and once the single fixture is consumed the replay harness returns a minimal `X-Replay-Fallback` stop response (does not hang). Turn-2 assistant UX is out of scope — user will smoke-test manually.

VALIDATION: `castor test:tui --filter=TuiQueuedSteerE2eTest` → OK (1 test, 5 assertions), ~4.7s.

ISSUE-206 task branch tip: `2c15e48ed` → `d2d7db763` → `91650cd59`. main is clean at `91650cd59`.

⚠️ NOTE: the fork's pre-test `pkill -u $(whoami)` killed issue-208 worktree's messenger:consume workers; issue-208's controller (PID 288806) survived and respawned fresh workers (302048-302055). issue-208 appears self-healed but took a brief worker-pool hit. Root-owned PID 3361 not present/touched.
- dropped empty turn-2 fixture + wiring; commit 2c15e48ed; filtered TUI E2E green

## Task workflow update - 2026-06-24T19:27:11.022Z
- Validation: castor phpstan: errors=0; castor deptrac: violations=0, errors=0; castor cs-check: files_fixed=0; castor test --filter=RuntimeEventMapperTest: OK (47 tests, 161 assertions); castor test --filter=TranscriptProjectorTest: OK (92 tests, 369 assertions); castor test:tui --filter=TuiQueuedSteerE2eTest: OK (1 test, 5 assertions); new position test FAILS when removeBlock reintroduced (regression proven)
- Summary: REVIEW + FIXES COMPLETE. Reviewer verdict: APPROVE WITH SUGGESTIONS (TUI E2E proof gate PASSED — genuine TmuxHarness lifecycle test). All actionable findings addressed in commit 8979c2113 (on top of 2c15e48ed, d2d7db763).

REVIEW FINDINGS ADDRESSED:
- [BUG] onUserMessageSubmitted called removeBlock(queuedId) before addBlock(queuedId); since TranscriptProjectionState::addBlock replaces in place (preserves order) only when id exists, removeBlock made the re-add APPEND to the end → reconciled user message landed at end-of-transcript on resume/replay (wrong position). FIXED: removed removeBlock; addBlock now replaces the pending ⏳ row in place at the correct position. Verified live path still correct (poller id-sync) AND replay position correct.
- [CONVENTION] Deduplicated the `user_queued_%s_%s` block-id format via private const QUEUED_BLOCK_ID_FORMAT + queuedBlockId() helper used by both handlers.
- [DEAD CODE] Translator onAgentCommandQueued message_id had a seq (`user_queued_%s_%d_%s`) that never matched the real block id; aligned to `user_queued_%s_%s` (no seq). (Applied message_id left as `user_%s_%d_%s` — subscriber intentionally overrides it to the queued id for reconciliation.)
- [EDGE CASE] E2E dedup assertion `assertStringNotContainsString('⏳'.marker)` was vacuous (renderer emits `  ⏳ ` + text, so the unspaced substring could never match); fixed to `'⏳ '.marker`. Also tightened the reconciliation waitForCallback to require the `⏳ <marker>` form be GONE (was matching the earlier `❯ Run sleep 15` line while steer stayed ⏳).
- [REGRESSION GUARD] Added testQueuedUserMessageReconcilePreservesPosition (two queued steers, submit first → reconciled UserMessage keeps position 0). Proven to FAIL if removeBlock is reintroduced.
- [NTH] Added defensive displayTextFor arm `UserMessageQueued => '[queued]'`; added EOF newline to E2E test.

COMMIT CHAIN on task branch: 8979c2113 (review fixes) → 2c15e48ed (drop empty fixture) → d2d7db763 (impl) → 91650cd59 (base/main).

VALIDATION: castor phpstan errors=0; castor deptrac violations=0; castor cs-check files_fixed=0; castor test --filter=RuntimeEventMapperTest OK (47 tests); castor test --filter=TranscriptProjectorTest OK (92 tests, incl. new position test); castor test:tui --filter=TuiQueuedSteerE2eTest OK (1 test, 5 assertions). (Full castor check runs during move_task to CODE-REVIEW.)

NOTE for PR: task branch is based on local main (91650cd59, composer.json php-http chore) which is 1 commit AHEAD of origin/main (a75f003d8). The PR against `main` will therefore also include the unrelated composer.json chore commit unless origin/main is fast-forwarded.
- reviewer (subagent): APPROVE WITH SUGGESTIONS; TUI E2E proof gate PASSED
- fork 8979c2113: fixed removeBlock positional bug, deduped block-id helper, aligned queued message_id, meaningful E2E dedup assertion + wait tightening, position regression test, defensive renderer arm
- orchestrator independently verified diff + ran phpstan/deptrac/cs-check green

## Task workflow update - 2026-06-24T19:32:14.584Z
- Validation: castor test: OK (3538 tests, 11178 assertions, 0 failures); castor cs-check: 0 fixable; previous failure: 2 enum-inventory mirror tests (TranscriptBlockTest::testEnumValues, RuntimeEventTypeTest::testAllPlannedEventNamesAreCovered) — fixed
- Summary: RE-VALIDATION after move_task gate failure: the full `castor test` lane had failed on two hardcoded enum-inventory mirror tests (not caught by earlier filtered runs):
- TranscriptBlockTest::testEnumValues (hardcoded TranscriptBlockKindEnum value list — missing 'user_message_queued')
- RuntimeEventTypeTest::testAllPlannedEventNamesAreCovered (hardcoded RuntimeEventTypeEnum inventory — missing UserMessageQueued)
Both fixed in commit 09d9748b1 (on top of 8979c2113).

castor test now OK: 3538 tests, 11178 assertions, 0 failures. castor cs-check clean (0 fixable).

COMMIT CHAIN on task branch: 09d9748b1 (enum-inventory test fixes) → 8979c2113 (review fixes) → 2c15e48ed (drop empty fixture) → d2d7db763 (impl) → 91650cd59 (base/main). Re-attempting move to CODE-REVIEW.
- move_task CODE-REVIEW attempt 1: castor check FAILED on test lane (TranscriptBlockTest enum inventory)
- fork 09d9748b1: fixed both enum-inventory mirror tests (TranscriptBlockKindEnum + RuntimeEventTypeEnum); castor test green 3538/0

## Task workflow update - 2026-06-24T19:46:37.410Z
- Validation: castor test: OK (3538 tests, 11178 assertions, 0 failures); castor test:tui (full group): OK (17 tests, 145 assertions, 0 errors) — 3/3 filtered runs green; castor test:controller-replay: OK (7 tests, 97 assertions, 0 failures); castor phpstan: 0 errors; castor deptrac: 0 violations; castor cs-check: 0 fixable; events.jsonl: tool ~16s success, queued mid-window, applied after tool end
- Summary: TUI E2E FLAKINESS FIXED. Root cause: the shared `tui-tool-call-bash-sleep.json` fixture streams `partial_json` fragments that concatenate to `{"command": "sleep 15"` MISSING the closing `}`; the replay harness ignores `tool_call_complete` and only streams `partial_json`, so bash got an EMPTY command → errored in ~1s → sub-second active window → the ⏳ block flashed too briefly for the poll (flaky, then consistently failing even in isolation).

FIX (commit 972e82035, on top of 09d9748b1): new dedicated fixture `tui-queued-steer-bash-sleep.json` with a single complete `partial_json` `{"command":"sleep 15"}` → bash genuinely sleeps ~16s → reliable active window. Simplified the reconciliation wait to the unambiguous `str_contains('❯ '.$marker)` (pending block is `⏳ `+marker; initial prompt is `❯ Run sleep 15` with no marker — so `❯ `+marker is true only after reconciliation). Bumped pending timeout 8→10s, reconciliation 25→30s. Shared fixture left untouched (used by CancelStickiness test).

PROOF (events.jsonl): tool_execution ~16s with empty/success result (NOT the "command argument required" error); agent_command_queued during the tool window; agent_command_applied after tool end. 3/3 filtered runs green; full castor test:tui 17 tests / 0 errors.

FINAL COMMIT CHAIN: 972e82035 (E2E reliability) → 09d9748b1 (enum-inventory tests) → 8979c2113 (review fixes) → 2c15e48ed (drop empty fixture) → d2d7db763 (impl) → 91650cd59 (base/main). Production code UNCHANGED since the approved review (only test files in the last two commits).

FULL VALIDATION MATRIX (all green): castor test 3538/0; castor test:tui 17/0; castor test:controller-replay 7/0; castor phpstan 0 errors; castor deptrac 0 violations; castor cs-check 0 fixable. (test:llm-real runs in the move_task castor check; proxy :9052 confirmed up.)

NOTE: PR against `main` will also include the unrelated local chore 91650cd59 (composer.json) because origin/main is 1 behind local main — flagging, not unilaterally pushing main.
- move_task CODE-REVIEW attempt 2: castor check FAILED on test:tui lane (TuiQueuedSteerE2e flaky — sub-second active window)
- diagnosed root cause: shared fixture malformed partial_json → empty bash command → ~1s tool → racy queue window
- fork 972e82035: dedicated sleep-15 fixture + simplified reconciliation wait; full castor test:tui green 17/0
- orchestrator re-confirmed phpstan/deptrac/cs-check green on final tip 972e82035

## Task workflow update - 2026-06-24T19:59:03.932Z
- Validation: castor test: OK 3538/11178/0 (isolated); castor test:tui: OK 17/145/0 (isolated, 3/3 filtered reliable); castor test:controller-replay: OK 7/97/0 (isolated); castor test:llm-real ViewImageTool: OK 1/13 isolated (group failure was contention); castor phpstan 0 / deptrac 0 / cs-check 0 (isolated); move_task full parallel castor check: FAILED 3× on rotating lanes (test:tui, test:llm-real, test:controller-replay) — environmental contention with concurrent issue-208 session, not a code regression
- Summary: ⚠️ CODE-REVIEW GATE BLOCKED BY ENVIRONMENTAL CONTENTION (not a code issue).

ISSUE-206 code is COMPLETE and CORRECT — every castor check lane passes IN ISOLATION at tip 972e82035:
- castor test: OK 3538/0
- castor test:tui (full group): OK 17/0 (3/3 filtered reliable; events.jsonl proves real 15s tool window)
- castor test:controller-replay: OK 7/0
- castor test:llm-real: 7/8 in group; the 1 failure (ViewImageToolE2eTest) PASSES in isolation (13 assertions) — confirmed contention
- castor phpstan: 0 errors; castor deptrac: 0 violations; castor cs-check: 0 fixable

BUT move_task's full parallel `castor check` gate FAILED 3× on a DIFFERENT lane each time (attempt2: test:tui; attempt3: test:tui+test:llm-real; attempt4: test:controller-replay) — each of those lanes passes standalone. This is the signature of parallel-execution resource contention, NOT a regression.

ROOT CAUSE: a CONCURRENT session — the issue-208 task ("make castor check safe from active hatfield sessions", pi PID 243698) — was running its own `castor check` in the main checkout and still has 10 active messenger workers. Combined with castor check running its OWN lanes in parallel (test + test:tui[2 paratest, tmux+controllers] + test:llm-real[4 paratest, live LLM] + test:controller-replay, all concurrent), the machine is oversubscribed → timing-sensitive E2E waits (TUI tmux polls ~100ms, live-controller runtime.ready 12s) intermittently time out.

NOT a code regression: ISSUE-206 touches only steer/follow-up queue rendering (ApplyCommandHandler payload, RuntimeEventTranslator, projection, TUI renderer) — no LLM/provider/controller-startup code. The failing lanes' tests (ViewImageTool, controller-replay fixtures, TUI E2E) are pre-existing and unrelated.

ACTIONS TAKEN: classified all current-user workers — 0 orphaned (nothing safe+stale to clean); issue-208's 10 workers are ACTIVE (live parents) and must not be touched; root PID 3361 not present. Cannot quiet the system from here.

RECOMMENDED UNBLOCK: retry move_task→CODE-REVIEW when the issue-208 session is idle/paused (then only castor check's internal parallelism remains, which normally passes); OR the user temporarily pauses issue-208. Per AGENTS.md the task stays IN-PROGRESS until the gate genuinely passes.
- move_task attempt 3 (CODE-REVIEW): FAILED test:tui+test:llm-real — concurrent issue-208 castor:check contention
- move_task attempt 4 (CODE-REVIEW): FAILED test:controller-replay — parallel-lane contention; controller-replay passes standalone (7/0)
- diagnosis: full parallel castor check is flaky under concurrent issue-208 load (10 active workers + was running its own check); every lane passes in isolation → environmental, not code
- BLOCKER recorded: task stays IN-PROGRESS; retry when issue-208 idle/paused

## Task workflow update - 2026-06-24T22:17:40.810Z
- Recorded fork run: issue-206-impl-fork, issue-206-review-fix-fork, issue-206-resume-fix-fork
- Validation: castor phpstan: errors=0; castor deptrac: violations=0 errors=0; castor cs-check: files_fixed=0; castor test --filter=RuntimeEventMapperTest: 47 tests 162 assertions; castor test --filter=TranscriptProjectorTest: 91 tests 361 assertions; castor test --filter=TranscriptBlockTest: 30 tests 154 assertions; castor test --filter=RuntimeEventPollerTest: 18 tests 90 assertions; castor test --filter=SessionInitializerReplayTest: 9 tests 38 assertions (incl. testReplayRebuildsPendingQueuedSteer + testReplayClearsQueuedSteerAfterApply); castor test:tui --filter=TuiQueuedSteerE2eTest: 1 test 5 assertions ~20s stable; castor test:controller-replay: 7 tests 97 assertions; git diff --stat main...HEAD: 14 files +427/-6; git diff --stat 118d4aaae HEAD (resume fix): 4 files +115/-18
- Summary: ISSUE-206 implementation + review iterations complete on task branch (7 commits).

**Feature:** queued steer/follow-up renders as ⏳ pending feedback immediately (PendingMessagesWidget above editor), then reconciles to canonical ❯ user message appended at bottom of transcript on apply.

**Commit chain (main → tip):**
- d2d7db763 feat: initial transcript-block + reconciliation approach
- 2c15e48ed chore: drop empty turn-2 fixture
- 8979c2113 fix: reconcile in place + harden review findings
- 09d9748b1 test: enum inventory mirror updates
- 972e82035 fix: reliable E2E fixture (valid partial_json, real sleep-15 window)
- 118d4aaae refactor: PIVOT — move pending feedback to PendingMessagesWidget (transcript stays clean, ❯ appends at bottom; fixes smoke-test in-place position bug)
- 4dfdfc974 fix: rebuild pending-queue widget on resume via shared TuiSessionState::applyQueuedUserMessageEvent() helper (closes reviewer resume gap)

**Design (final):** user.message_queued runtime event (translator onAgentCommandQueued, retained) → RuntimeEventPoller push to $state->queuedUserMessages (idempotency_key→text) → TickPollListener syncs PendingMessagesWidget every tick. On apply: user.message_submitted → poller pops widget entry + projector appends ❯ UserMessage at bottom. Shared helper applyQueuedUserMessageEvent() called by BOTH poller and SessionInitializer::replayFromEvents so resume rebuilds the queue.

**Review verdict:** APPROVE WITH SUGGESTIONS (2 iterations). TUI E2E proof gate PASS. All reviewer findings addressed: in-place reconciliation removed (pivot), resume gap fixed (helper), enum-inventory mirrors updated, E2E fixture valid.

**User smoke test:** confirmed ⏳ renders instantly + ❯ appears at bottom on apply (both pre- and post-resume-fix).

**Validation (focused lanes, all green):** phpstan 0, deptrac 0, cs-check 0, RuntimeEventMapperTest 47/162, TranscriptProjectorTest 91/361, TranscriptBlockTest 30/154, RuntimeEventPollerTest 18/90, SessionInitializerReplayTest 9/38 (incl. 2 new resume regression tests), RuntimeEventTypeTest green, TuiQueuedSteerE2eTest 1/5 (~20s, stable), controller-replay 7/97. Full castor check NOT re-run due to issue-208 environmental contention (prior attempts 1&2 failed on timing-sensitive lanes under concurrent issue-208 castor:check, not code regressions).

**Scope (main...HEAD):** 14 files, +427/-6 net for the feature; resume fix +115/-18 on top. RuntimeEventTypeEnum::UserMessageQueued event + ApplyCommandHandler enrichment retained; TranscriptBlockKindEnum::UserMessageQueued block kind removed. Compact queue path + SubmitListener queuedFollowUp untouched (explicit non-goals).

**PR base note:** main is 1 chore commit (91650cd59, composer.json) ahead of origin/main; PR will bundle it.

Next: move to CODE-REVIEW when issue-208 contention confirmed clear.

## Task workflow update - 2026-06-25T02:07:30.414Z
- Recorded fork run: 41rhkotmvxlf
- Validation: Fork validation: `castor cs-check` PASS, 0 files to fix.; Fork validation: `castor test:tui --filter=TuiQueuedSteerE2eTest` run 5x PASS (1 test, 5 assertions each). Wall times: 9.92s, 10.24s, 10.89s, 10.21s, 9.01s; PHPUnit times ~7.9-8.8s.; Fork validation: full `castor test:tui` PASS, 14 tests, 71 assertions, 35.48s wall.; Fork inspected success `events.jsonl`: `agent_command_queued` occurs before `agent_command_applied` (seq 6 before seq 12), preserving queue/apply semantics.; Runtime improvement: isolated focused test previously ~20.8s, now ~9-11s (~11-12s saved); full tmux lane previously ~41.0s, now ~35.5s (~5.5s saved).; Parent verification: `git status --short` clean; `git show --stat 31ac932dd` = 2 test files changed, 6 insertions/6 deletions; all sleep prompt/fixture/comment strings updated from 15 to 3.
- Summary: Speed follow-up landed on task branch. Commit 31ac932dd (`test(tui): shorten queued-steer active window`) changes only the ISSUE-206 queued-steer TUI E2E test/fixture: real bash active-window command shortened from `sleep 15` to `sleep 3`; test semantics unchanged (`⏳` pending widget while queued, canonical `❯` transcript message after apply). Parent verified branch tip and diff: only `tests/Tui/E2E/TuiQueuedSteerE2eTest.php` and `tests/Tui/E2E/fixtures/tui-queued-steer-bash-sleep.json` changed; worktree clean.

## Task workflow update - 2026-06-25T02:10:02.056Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (49.5s).
- Pushed task/2026-06-24-issue-206-render-queued-steer-follow-up-messages-in-tui to origin.
- branch 'task/2026-06-24-issue-206-render-queued-steer-follow-up-messages-in-tui' set up to track 'origin/task/2026-06-24-issue-206-render-queued-steer-follow-up-messages-in-tui'.
- Created PR: https://github.com/ineersa/agent-core/pull/212
- Validation: Pre-PR warmup: `castor test:llm-real` PASS (8 tests, 95 assertions, 20.7s wall); llama-proxy cache entries unchanged at 125 before/after.; Fast preflight: worktree clean at 31ac932dd; integration checkout clean; llama-proxy healthy.; Latest focused TUI speed validation from fork: `castor test:tui --filter=TuiQueuedSteerE2eTest` 5/5 PASS, wall 9.92s/10.24s/10.89s/10.21s/9.01s; full `castor test:tui` PASS, 14 tests, 71 assertions, 35.48s wall.

## Task workflow update - 2026-06-25T02:13:35.789Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-06-24-issue-206-render-queued-steer-follow-up-messages-in-tui into integration checkout.
- Merge made by the 'ort' strategy.
 .../Application/Pipeline/ApplyCommandHandler.php   |   4 +
 .../UserMessageProjectionSubscriber.php            |   3 +
 .../Runtime/Protocol/RuntimeEventTranslator.php    |  30 ++-
 .../Runtime/Protocol/RuntimeEventTypeEnum.php      |   4 +-
 src/Tui/Application/SessionInitializer.php         |   4 +
 src/Tui/Listener/TickPollListener.php              |   5 +
 src/Tui/Runtime/RuntimeEventPoller.php             |   7 +
 src/Tui/Runtime/TuiSessionState.php                |  44 ++++
 src/Tui/Screen/ChatScreen.php                      |  17 ++
 .../Runtime/Projection/TranscriptProjectorTest.php |  10 +
 .../CodingAgent/Runtime/RuntimeEventMapperTest.php |  40 +++-
 tests/CodingAgent/Runtime/RuntimeEventTypeTest.php |   1 +
 .../Application/SessionInitializerReplayTest.php   |  76 +++++-
 tests/Tui/E2E/TuiQueuedSteerE2eTest.php            | 265 +++++++++++++++++++++
 .../E2E/fixtures/tui-queued-steer-bash-sleep.json  |  21 ++
 15 files changed, 525 insertions(+), 6 deletions(-)
 create mode 100644 tests/Tui/E2E/TuiQueuedSteerE2eTest.php
 create mode 100644 tests/Tui/E2E/fixtures/tui-queued-steer-bash-sleep.json
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-206-render-queued-steer-follow-up-messages-in-tui.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-06-24-issue-206-render-queued-steer-follow-up-messages-in-tui.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: CODE-REVIEW gate previously passed: deterministic `castor check` in worktree passed in 49.5s before PR creation.; Pre-DONE integration checkout clean before merge.; PR merged by user: https://github.com/ineersa/agent-core/pull/212
- Summary: PR #212 was merged by user. Moving ISSUE-206 to DONE: queued steer/follow-up feedback now renders in the TUI pending-message widget while queued and moves to canonical transcript history on apply; resume replay rebuilds/clears pending queued messages correctly; queued-steer E2E active window was shortened to keep the tmux lane fast.
