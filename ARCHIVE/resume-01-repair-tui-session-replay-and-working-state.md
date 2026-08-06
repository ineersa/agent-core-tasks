# RESUME-01 Repair TUI session replay, tool results, and working state

## Goal
User smoke testing during AGENT-08 showed resume is broadly broken: after resuming, user messages can disappear, subagent progress/handoff/agent_retrieve results may not be visible, tool-call-only assistant steps can render as blank assistant messages, and the footer tokens/sec can appear broken. A concrete inspected session `.hatfield/sessions/6` showed the SafeGuard/subagent/retrieve segment completed correctly (terminal agent_end and non-empty tool_execution_end results), but a later cancelled parallel subagent segment left the TUI looking stuck/ambiguous. This task should repair resume/transcript replay and terminal working-state cleanup holistically, not patch AGENT-08 subagent logic.

Related GitHub issues created from scout findings:
- #215 TUI can remain stuck Working after cancelling parallel subagent execution
- #216 Transcript replay can hide tool handoff/retrieve result after tool progress deltas
- #217 Resume transcript can contain empty assistant block for tool-call-only LLM step
- #218 Footer tokens/sec can disappear when usage payload is missing or turn timing resets

Likely files/areas:
- src/Tui/Application/SessionInitializer.php
- src/Tui/Runtime/RuntimeEventPoller.php
- src/Tui/Runtime/ActivityStateMachine.php
- src/Tui/Listener/TickPollListener.php
- src/Tui/Status/WorkingStatusWidget.php
- src/CodingAgent/Runtime/ProjectionPipeline/ToolProjectionSubscriber.php
- src/CodingAgent/Runtime/ProjectionPipeline/AssistantStreamProjectionSubscriber.php
- src/CodingAgent/Runtime/ProjectionPipeline/UserMessageProjectionSubscriber.php
- src/CodingAgent/Runtime/Projection/TranscriptProjectionState.php
- src/Tui/Runtime/UsageProjection.php

Important constraints:
- Must load testing skill and tests/AGENTS.md before touching tests/TUI/runtime.
- Must prove user-visible resume behavior at the lowest correct layer: virtual/in-process projection tests for transcript replay and working-state cleanup; controller replay if runtime protocol sequencing matters; tmux only if terminal integration is specifically affected.
- Do not fold this into AGENT-08 unless explicitly requested; AGENT-08 SafeGuard/subagent/retrieve segment itself completed correctly in inspected session.

## Acceptance criteria
- Resuming a session with user messages, tool-call-only assistant steps, subagent progress deltas, subagent final result, and agent_retrieve result reconstructs a visible transcript with no missing user messages, no blank assistant-only artifacts, and visible final tool result/handoff text.
- Cancelling an in-flight parallel subagent/tool execution transitions TUI activity/working state to terminal/idle and clears stale `Working...` indicators after run.cancelled/run.completed/run.failed.
- Projection/replay handles ToolExecutionStart -> ToolExecutionUpdate -> ToolExecutionEnd with non-empty result by showing the final result, not only progress text; legacy/empty-result completions preserve useful accumulated output and finalize streaming state.
- Footer tokens/sec behavior is deterministic after resume and during tool-heavy turns: either displays valid usage when available or degrades explicitly/cleanly when provider usage is absent, without appearing stuck/broken.
- Automated tests cover the reproduced resume/projection bugs using virtual/runtime replay layers; Castor validation passes for focused tests plus required TUI/runtime lanes (`castor check` before CODE-REVIEW if runtime/TUI flow touched).

## Workflow metadata
Status: DONE
Branch: task/resume-01-repair-tui-session-replay-and-working-state
Worktree: /home/ineersa/projects/agent-core-worktrees/resume-01-repair-tui-session-replay-and-working-state
Fork run: 56uo0vzwqfm7
PR URL: https://github.com/ineersa/agent-core/pull/225
PR Status: merged
Started: 2026-06-26T22:05:12.142Z
Completed: 2026-06-27T00:43:14.224Z

## Work log
- Created: 2026-06-25T20:23:11.173Z

## Task workflow update - 2026-06-25T23:20:29.430Z
- Summary: Added AGENT-09 smoke/resume requirement: resumed sessions currently show generic old transcript blocks (`● subagent(tasks: ...)`, `agent_retrieve(...)`, assistant thinking/messages) and no structured AGENT-09 subagent widget/live progress. RESUME-01 should include a replay fixture with AGENT-09-style `tool_execution.output_delta` / `subagent_progress` metadata and assert transcript replay renders the structured subagent widget after resume. Also cover old sessions/events without `subagent_progress` gracefully (generic fallback is acceptable for old events), but new AGENT-09 events must survive canonical replay/projection. This is intentionally not bundled into AGENT-09 because the observed symptom is broader resume reconstruction/tool-result replay behavior tracked by RESUME-01/issues #215-#218.

## Task workflow update - 2026-06-26T17:43:49.318Z
- Summary: Carry-over from ISSUE-205 live testing: bash background/cancel works without resume on HEAD e5c4ab8f9, but after `/resume` something in resume/replay/working-state still breaks. Observed session 4 after resume: previously cancelled/completed session was reanimated by follow_up, later got stuck/corrupted around resumed working state. ISSUE-205 is intentionally skipping resume-specific repair per user instruction; RESUME-01 should investigate resume path separately. Evidence to inspect if needed: worktree `.hatfield/sessions/4/events.jsonl` and `state.json` from ISSUE-205 branch. Prior scouts found resume-related suspects: ActivityStateMachine replay after terminal agent_end, RunStateReplayService stale Cancelling reconstruction, and TUI/process delivery/replay mismatch after resuming multi-turn cancelled sessions. Product symptom: same cancel/background flow works in fresh/non-resumed run but fails after resume.

## Task workflow update - 2026-06-26T22:05:08.198Z
- Summary: Architecture direction agreed during task-explain: stop treating resume as a special reconstruction path. Canonical events.jsonl must be sufficient to rebuild the same visible TUI state that live processing produced. Introduce/extract one shared TUI runtime event applier/reducer used by both RuntimeEventPoller (live) and SessionInitializer replay. Widgets should render from TuiSessionState and should not need resume-specific hacks. If a visible live behavior cannot be reconstructed after resume, fix the root boundary: canonical event payload writing, RuntimeEventTranslator mapping, or the shared applier/projection state — not SessionInitializer/widget symptom patches. Implementation should add an equivalence test proving applying a canonical event sequence through the live-style applier and replaying the same events through SessionInitializer produce matching visible state for user messages, tool-call-only assistant step, tool progress deltas, final subagent/agent_retrieve results, cancellation/terminal working state, and usage/footer-relevant state. Footer throughput may need an explicit contract if exact t/s depends on non-persisted wall-clock timing.
- User explicitly approved root-cause architecture approach and asked to move task to IN-PROGRESS and launch an implementation fork.

## Task workflow update - 2026-06-26T22:05:12.142Z
- Moved TODO → IN-PROGRESS.
- Created branch task/resume-01-repair-tui-session-replay-and-working-state.
- Created worktree /home/ineersa/projects/agent-core-worktrees/resume-01-repair-tui-session-replay-and-working-state.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/resume-01-repair-tui-session-replay-and-working-state.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/resume-01-repair-tui-session-replay-and-working-state.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/resume-01-repair-tui-session-replay-and-working-state.
- Summary: Starting implementation with root-cause architecture approach: shared live/replay TUI runtime event applier, canonical events.jsonl as source of truth, fix payload/translator/applier gaps instead of widget/resume-specific symptom patches.

## Task workflow update - 2026-06-26T22:05:51.008Z
- Recorded fork run: uuho2nlp7hzl
- Launched implementation fork uuho2nlp7hzl in worktree /home/ineersa/projects/agent-core-worktrees/resume-01-repair-tui-session-replay-and-working-state with instructions to implement shared live/replay TUI runtime event applier architecture, root equivalence tests, focused projection/replay/footer/working-state tests, and a replay-backed TmuxHarness proof.

## Task workflow update - 2026-06-26T23:10:27.288Z
- Recorded fork run: uuho2nlp7hzl
- Validation: Parent inspected worktree: git status --short clean.; Parent inspected commit: f4f6aabd5 RESUME-01: shared TuiRuntimeEventApplier for live+replay state.; Parent inspected diff against main: 15 files changed, 388 insertions, 79 deletions.; Fork reported successful focused validation: castor test --filter=RuntimeEventPollerTest; castor test --filter=SessionInitializerReplayTest; castor test --filter=TranscriptProjectorTest; castor test --filter=ActivityStateMachineTest; castor test --filter=UsageProjectionTest; castor test --filter=FooterStateSegmentProviderTest; castor test --filter=TuiResumeSessionVirtualTest; castor test --filter=TuiRuntimeEventApplierTest; castor test:tui --filter=TuiResumeSessionSwitchE2eTest; castor deptrac; castor phpstan; castor cs-check.; Not yet run by parent/fork: castor check and castor test:controller-replay.
- Summary: Implementation fork completed and committed f4f6aabd5b8b20f98d63b17d12184562bb60dcea on task/resume-01-repair-tui-session-replay-and-working-state. Parent verification: worktree is clean; commit files match RESUME-01 scope (15 files). Implementation introduced src/Tui/Runtime/TuiRuntimeEventApplier.php as the shared live/replay reducer used by RuntimeEventPoller and SessionInitializer, fixed tool completion projection for progress-to-final and empty legacy results, added replay footer timing contract via UsageProjection::resetTurnForReplay(), and suppresses stale Working when resumed state has no live handle. Tests added/updated for applier equivalence, transcript projection, session replay, usage/footer contract, and resume virtual/TUI coverage. No canonical event writer changes were needed; RuntimeEventTranslator already carried subagent_progress/result when present.

## Task workflow update - 2026-06-26T23:20:14.151Z
- Summary: User smoke test reports all previous resume issues still reproduce after commit f4f6aabd5. Parent diagnosis: this likely proves resume is not merely re-rendering/attaching. Current TUI resume path calls InteractiveMode::startOrResumeRun() -> AgentSessionClient::resume($runId). JsonlProcessAgentSessionClient::resume sends a controller 'resume' command; ResumeHandler calls InProcessAgentSessionClient::resume(); InProcessAgentSessionClient::resume currently calls AgentRunnerInterface::continue($runId), which dispatches CoreCommandKind::Continue. Therefore opening/resuming a session mutates AgentCore runtime state before the user submits a new message. That can change activity/routing for subsequent user messages (steer vs follow_up), reanimate terminal/cancelled state, and explain why future messages/tool results behave differently after resume. Next iteration should treat this as a deeper root cause than TUI replay projection: split 'attach/open existing session/controller event stream' from 'advance/continue a run'. TUI /resume should attach to the existing session and replay/drain events without dispatching Continue. Only an explicit runtime recovery path for truly unfinished active runs should dispatch Continue, and only when replayed state says it is safe/needed.

## Task workflow update - 2026-06-26T23:22:58.402Z
- Recorded fork run: wu0imkqf9kxl
- Launched background fork wu0imkqf9kxl to merge origin/main into task branch, preserving main's background-tool cancellation fixes and RESUME-01 replay/applier changes; fork instructed to avoid destructive git operations, read testing skill + tests/AGENTS before validation, resolve conflicts conservatively, run focused Castor validation, and commit if successful.

## Task workflow update - 2026-06-26T23:24:10.250Z
- Recorded fork run: wu0imkqf9kxl
- Validation: castor test --filter='RuntimeEventPollerTest|TuiRuntimeEventApplierTest|SessionInitializerReplayTest|ActivityStateMachineTest|TickPollListenerTest|BackgroundProcessCompletionPollerTest' => OK (116 tests, 291 assertions); castor deptrac => OK (0 violations); castor phpstan --path=src/Tui/Runtime => OK (no errors); castor cs-check => OK (0 files to fix)
- Summary: Merge fork completed successfully. origin/main (including ISSUE-205 / PR #213 background-tool cancellation fixes) merged into task branch without dropping RESUME-01 commit f4f6aabd5. Final merge commit: d6dcd3de77c07643f802e7e172d5b9a27aa5068d. One manual conflict resolved in src/Tui/Runtime/RuntimeEventPoller.php: kept RESUME-01 TuiRuntimeEventApplier projection path and kept main's isolated invokeEventCallback handling; removed obsolete direct projector->accept path. Post-merge resume() still calls runner->continue($runId), so passive attach remains the next architecture change.

## Task workflow update - 2026-06-26T23:28:33.075Z
- Summary: Product/architecture decision confirmed by user: resume is purely old TUI history + state/session load. It must have no runtime side effects. After resume, the session should behave exactly like a new/live session from that point onward; opening/resuming must not dispatch AgentCore Continue, mutate canonical run state, reanimate cancelled/completed work, or alter next-message routing. Implementation should make this hard to regress by splitting/naming the runtime boundary so attach/open is passive and Continue remains an explicit AgentCore operation only when intentionally requested.

## Task workflow update - 2026-06-26T23:29:00.994Z
- Recorded fork run: ed8ql6jowikh
- Launched implementation fork ed8ql6jowikh to make session resume passive attach/history+state load only. Fork instructed to split/rename runtime boundary if appropriate, remove AgentRunner::continue side effect from resume/attach, update controller/headless/TUI call sites, add regression tests proving attach does not dispatch Continue, run focused Castor validation, and commit changes.

## Task workflow update - 2026-06-26T23:31:46.803Z
- Recorded fork run: ed8ql6jowikh
- Validation: Verified worktree status clean and log contains dc1378bbb atop d6dcd3de/f4f6aabd5.; Fork validation: castor test --filter='InProcessAttachDoesNotContinueTest|ResumeHandlerTest|RuntimeEventPollerTest|TuiRuntimeEventApplierTest|SessionInitializerReplayTest|ActivityStateMachineTest|TickPollListenerTest|BackgroundProcessCompletionPollerTest|CompactHandlerTest' => OK (123 tests, 312 assertions); Fork validation: castor test:controller-replay => OK (8 tests, 112 assertions); Fork validation: castor deptrac => OK (0 violations); Fork validation: castor phpstan --path=src/CodingAgent/Runtime => OK; Fork validation: castor phpstan --path=src/Tui/Application => OK; Fork validation: castor cs-check => OK
- Summary: Passive attach implementation fork completed and was verified present in worktree. Commit dc1378bbb (RESUME-01: passive attach replaces resume-on-open AgentCore Continue) is on branch after merge d6dcd3de. Worktree status clean. Diff from d6dcd3de..dc1378bbb changes 19 files: replaces AgentSessionClient::resume() with attach(), updates InProcess/Process clients, controller ResumeHandler, headless AgentCommand, TUI InteractiveMode, MCP attach reason, docs, fakes, and adds passive attach regression tests. Semantics: TUI/session resume is history+state load and runtime transport attach only; no AgentRunner::continue(), no CoreCommandKind::Continue, no canonical run mutation on open. JSONL wire command remains 'resume' but handler calls attach().

## Task workflow update - 2026-06-26T23:39:32.296Z
- Summary: User smoke after passive attach commit dc1378bbb reports 0 observable improvement: TUI still acts completely differently after resume. Conclusion: Continue-on-resume was a real bug but not the only/root smoke path. Next investigation must compare the actual live fresh-session path vs resumed-session path end-to-end, including command routing after resume, TUI state reconstruction from SessionInitializer, projected activity/queued state/handle, controller queue/session scoping, and whether the running binary/session under smoke is actually using the task branch changes.

## Task workflow update - 2026-06-26T23:45:57.630Z
- Summary: Read-only scouts after failed smoke found passive attach fixed one bug but left resume-specific state/transport divergence. Most critical: controller RuntimeEventEmitter only registers drain cursors on run.started; after passive attach/resume a fresh controller emits run.resumed but no cursor is registered, so drainRegisteredRunsOnce() skips the run entirely and no canonical events (tool completions, terminal events, usage) are forwarded to TUI after resume. This explains why resumed sessions behave differently even without Continue. Additional replay divergences: TuiSessionState::isShellRun is ephemeral and not reconstructed, so shell-only sessions after resume route next normal prompt as follow_up instead of starting a fresh LLM run; replay can leave activity Running/Cancelling from mid-cutoff events, and after attach handle is non-null so SubmitListener routes next input as steer/queued and TickPollListener shows Working even though attach is passive and no live work may exist.
- Scout output saved at /home/ineersa/.pi/agent/tmp/2026-06--0b17ad06.txt. Key files: RuntimeEventEmitter.php cursor registration, ResumeHandler.php run.resumed emission, SessionInitializer.php replayFromEvents, SubmitListener.php dispatchToRuntime routing, TuiSessionState.php isShellRun/activity fields.

## Task workflow update - 2026-06-26T23:46:22.260Z
- Recorded fork run: 5eq8a3gyh4bg
- Launched implementation fork 5eq8a3gyh4bg to fix post-passive-attach smoke failure: register controller event drain on run.resumed/passive attach, reconstruct shell-only isShellRun on replay, normalize stale active replay state for passive resume, add focused regressions, run Castor validation, and commit.

## Task workflow update - 2026-06-26T23:50:23.594Z
- Recorded fork run: 5eq8a3gyh4bg
- Validation: Fork read testing/SKILL.md and tests/AGENTS.md before test work.; castor test --filter='RuntimeEventEmitterTest|SessionInitializerReplayTest' => OK (18 tests, 67 assertions); castor test:controller-replay => OK (8 tests, 112 assertions); castor deptrac => OK (0 violations); castor phpstan --path=src/CodingAgent/Runtime/Controller/RuntimeEventEmitter.php => OK; castor phpstan --path=src/Tui/Application/SessionInitializer.php => OK; castor cs-check => OK after cs-fix
- Summary: Fork 5eq8a3gyh4bg completed and committed post-passive-attach smoke fixes. Commit 8188f0dc3 (RESUME-01: drain on attach, shell replay, idle resume activity) fixes: RuntimeEventEmitter registers canonical drain cursor on RunResumed/passive attach; SessionInitializer infers shell-only sessions from canonical events and restores isShellRun; replay normalizes active resume activity to Idle/isCompacting=false so passive attach does not preserve stale Working/steer state. No AgentRunner::continue reintroduced. Open risks: HITL WaitingHuman now resumes Idle until live events restore state; shell-only inference is heuristic for bash tool start + completed agent_end + no run_started/llm_step_completed; full castor check/tmux/live smoke not run.

## Task workflow update - 2026-06-27T00:05:06.367Z
- Validation: Reviewer read testing/SKILL.md and tests/AGENTS.md before reviewing validation/test adequacy.; Reviewer decision: APPROVE WITH SUGGESTIONS; treat Compacting stale Working as pre-PR fix.
- Summary: Reviewer subagent completed on branch head 8188f0dc3 with verdict APPROVE WITH SUGGESTIONS. Reviewer confirmed passive attach avoids Continue, RunResumed cursor registration restores canonical draining, tests are strong, and boundaries are clean. Reviewer flagged one real pre-PR edge bug: replay normalization only handles activity->isActive(), but Compacting is not isActive(); a resume after context_compaction_started can keep activity=Compacting/isCompacting=true and show stale Working forever. Recommended fix: normalize Compacting to Idle/isCompacting=false during passive resume and add regression. Non-blocking notes: RunResumed cursor starts at 0 and re-drains old events (TUI lastSeq dedup avoids correctness issue); run.resumed seq=1 could be seq=0 future improvement; UsageProjection docblock drift; Jsonl autoResumed status running vs attached cosmetic. Mandatory before CODE-REVIEW: focused validation including castor test:tui where applicable and full castor check via move_task gate.

## Task workflow update - 2026-06-27T00:06:24.997Z
- Recorded fork run: g0sjehpcamq5
- Validation: Fork read testing/SKILL.md and tests/AGENTS.md before tests.; castor test --filter='SessionInitializerReplayTest|UsageProjectionTest' => OK (32 tests, 124 assertions); castor phpstan --path=src/Tui/Application/SessionInitializer.php => OK; castor phpstan --path=src/Tui/Runtime/UsageProjection.php => OK; castor cs-check => OK
- Summary: Fork g0sjehpcamq5 completed reviewer pre-PR edge fix. Commit d367a71d7 normalizes Compacting to Idle/isCompacting=false during passive resume replay, preserving terminal states and avoiding stale Working when replay ends after context_compaction_started. Also corrected UsageProjection resetTurn/resetTurnForReplay docblocks. No AgentRunner::continue reintroduced.

## Task workflow update - 2026-06-27T00:09:11.567Z
- Validation: Focused reviewer decision after d367a71d7: APPROVED.
- Summary: Focused reviewer rerun after commit d367a71d7 returned APPROVED. Reviewer verified Compacting stale-Working fix is correct/localized, regression test is meaningful at the replay/session-state layer, UsageProjection docblock cleanup is harmless, and no AgentRunner::continue was reintroduced. No blockers remain from reviewer. Remaining pre-PR requirement: run focused validation (including TUI layer as appropriate) and full castor check gate during move_task to CODE-REVIEW.

## Task workflow update - 2026-06-27T00:11:37.972Z
- Recorded fork run: 1rh98rpy44b9
- Launched merge fork 1rh98rpy44b9 to fetch/merge origin/main after user merged agent workflow cancellation changes. Instructions: preserve passive attach/no Continue, RunResumed canonical drain, replay shell-only/active/Compacting normalization; integrate main cancellation changes carefully; run focused Castor validation and commit merge/resolution.

## Task workflow update - 2026-06-27T00:13:33.930Z
- Recorded fork run: 1rh98rpy44b9
- Validation: Fork read testing/SKILL.md and tests/AGENTS.md before validation.; castor test --filter='RuntimeEventPollerTest|TuiRuntimeEventApplierTest|SessionInitializerReplayTest|ActivityStateMachineTest|TickPollListenerTest|RuntimeEventEmitterTest|ResumeHandlerTest|InProcessAttachDoesNotContinueTest' => OK (122 tests, 302 assertions); castor test:controller-replay => OK (8 tests, 112 assertions); castor deptrac => OK (0 violations); castor phpstan --path=src/Tui => OK; castor phpstan --path=src/CodingAgent/Runtime => OK; castor cs-check => OK
- Summary: Merge fork 1rh98rpy44b9 completed. origin/main (including PR #224 agent workflow cancellation/subagent reporting changes) merged into RESUME-01 branch without conflicts or manual edits. Merge commit 1ee122a4c atop d367a71d7. RESUME-01 invariants verified by fork: attach remains passive/no runner->continue, RuntimeEventEmitter registers canonical drain on RunResumed, SessionInitializer replay uses TuiRuntimeEventApplier, restores shell-only isShellRun, and normalizes stale active/Compacting replay to Idle. Incoming main changes touched AgentCore tool cancellation/subagent reporting and RuntimeEventTranslator; focused validation green.

## Task workflow update - 2026-06-27T00:22:39.073Z
- Validation: move_task to CODE-REVIEW ran castor check and failed: quality failed - test exit code 2; Failing log: /home/ineersa/projects/agent-core-worktrees/resume-01-repair-tui-session-replay-and-working-state/var/reports/qa-20260627-002114-486839-4f1af0af/check-test.log
- Summary: CODE-REVIEW transition failed because deterministic castor check failed in unit/integration lane. Report var/reports/qa-20260627-002114-486839-4f1af0af/check-test.log: SessionInitializerTest::testBuildInitialTranscriptFreshSessionReturnsWelcome errors with typed property $projector accessed before initialization at tests/Tui/Application/SessionInitializerTest.php:65. Cause visible in setUp(): SessionRunEventStore is constructed with new TuiRuntimeEventApplier($this->projector) before $this->projector is assigned. Need fix and rerun focused test/check before retrying CODE-REVIEW.

## Task workflow update - 2026-06-27T00:24:00.751Z
- Recorded fork run: 56uo0vzwqfm7
- Validation: Fork read testing/SKILL.md and tests/AGENTS.md before tests.; castor test --filter='SessionInitializerTest|SessionInitializerReplayTest' => OK (22 tests, 93 assertions); castor test => OK (3702 tests, 11848 assertions); castor cs-check => OK
- Summary: Fork 56uo0vzwqfm7 fixed CODE-REVIEW gate unit-lane failure. Commit 26fa90a8b updates tests/Tui/Application/SessionInitializerTest.php setUp(): initializes projector before use and removes invalid eventApplier named arg from SessionRunEventStore construction. Production untouched. Full castor test lane now green; ready to retry CODE-REVIEW transition.

## Task workflow update - 2026-06-27T00:25:54.975Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (94.5s).
- Pushed task/resume-01-repair-tui-session-replay-and-working-state to origin.
- branch 'task/resume-01-repair-tui-session-replay-and-working-state' set up to track 'origin/task/resume-01-repair-tui-session-replay-and-working-state'.
- Created PR: https://github.com/ineersa/agent-core/pull/225
- Validation: User smoke after RunResumed drain fix: resume seems to work now.; Reviewer: APPROVED after d367a71d7 Compacting edge fix.; Focused post-main-merge validation: castor test --filter='RuntimeEventPollerTest|TuiRuntimeEventApplierTest|SessionInitializerReplayTest|ActivityStateMachineTest|TickPollListenerTest|RuntimeEventEmitterTest|ResumeHandlerTest|InProcessAttachDoesNotContinueTest' => OK (122 tests, 302 assertions); Focused post-main-merge validation: castor test:controller-replay => OK (8 tests, 112 assertions); Focused post-main-merge validation: castor deptrac => OK (0 violations); Focused post-main-merge validation: castor phpstan --path=src/Tui => OK; Focused post-main-merge validation: castor phpstan --path=src/CodingAgent/Runtime => OK; Focused post-main-merge validation: castor cs-check => OK; Post-gate-failure fix validation: castor test --filter='SessionInitializerTest|SessionInitializerReplayTest' => OK (22 tests, 93 assertions); Post-gate-failure fix validation: castor test => OK (3702 tests, 11848 assertions); Post-gate-failure fix validation: castor cs-check => OK
- Summary: Retrying CODE-REVIEW after fixing prior castor check unit-lane failure. Branch head: 26fa90a8b (test fix for SessionInitializerTest projector init order) atop merge 1ee122a4c. Key RESUME-01 fixes remain: shared live/replay TuiRuntimeEventApplier; passive attach replaces resume-on-open Continue; RunResumed registers canonical event drain; shell-only resume restores isShellRun; stale active/Compacting replay normalizes to Idle; tool projection finalization fixes. User smoke passed, reviewer approved, full castor test unit/integration lane passed after the fix.

## Task workflow update - 2026-06-27T00:43:14.225Z
- Moved CODE-REVIEW → DONE.
- Merged task/resume-01-repair-tui-session-replay-and-working-state into integration checkout.
- Merge made by the 'ort' strategy.
 docs/session-storage.md                            |   2 +-
 src/CodingAgent/CLI/AgentCommand.php               |   4 +-
 .../Mcp/McpSessionLifecycleDispatcher.php          |   4 +-
 .../Mcp/Message/McpInitializeSessionCommand.php    |   4 +-
 .../Runtime/Contract/AgentSessionClient.php        |   8 +-
 src/CodingAgent/Runtime/Contract/RunHandle.php     |   2 +-
 .../Controller/CommandHandler/ResumeHandler.php    |  16 +-
 .../Runtime/Controller/RuntimeEventEmitter.php     |   7 +-
 .../InProcess/InProcessAgentSessionClient.php      |  11 +-
 .../Process/JsonlProcessAgentSessionClient.php     |   6 +-
 .../ToolProjectionSubscriber.php                   |  34 ++--
 src/Tui/Application/InteractiveMode.php            |   2 +-
 src/Tui/Application/SessionInitializer.php         |  71 +++++---
 src/Tui/Listener/TickPollListener.php              |   3 +
 src/Tui/Runtime/RuntimeEventPoller.php             |  36 +---
 src/Tui/Runtime/TuiRuntimeEventApplier.php         |  59 +++++++
 src/Tui/Runtime/UsageProjection.php                |  13 ++
 .../Handler/McpInitializeSessionHandlerTest.php    |  10 +-
 .../BackgroundProcessCompletionPollerTest.php      |   2 +-
 .../CommandHandler/AnswerHumanHandlerTest.php      |   4 +-
 .../CommandHandler/CompactHandlerTest.php          |   4 +-
 .../CommandHandler/ResumeHandlerTest.php           | 118 +++++++++++++
 .../CommandHandler/ShellCommandHandlerTest.php     |   4 +-
 .../Runtime/Controller/RuntimeEventEmitterTest.php |  41 ++++-
 .../InProcessAttachDoesNotContinueTest.php         |  92 ++++++++++
 .../Runtime/Projection/TranscriptProjectorTest.php |  46 +++++
 .../Application/SessionInitializerReplayTest.php   | 105 +++++++++--
 tests/Tui/Application/SessionInitializerTest.php   |  10 +-
 tests/Tui/Listener/CompactCommandHandlerTest.php   |   4 +-
 tests/Tui/Runtime/RuntimeEventPollerTest.php       |   3 +-
 tests/Tui/Runtime/TuiRuntimeEventApplierTest.php   | 194 +++++++++++++++++++++
 tests/Tui/Runtime/UsageProjectionTest.php          |  20 +++
 tests/Tui/Screen/TuiResumeSessionVirtualTest.php   |   2 +-
 tests/Tui/Support/ResumeCanonicalEventsFixture.php |  11 +-
 .../ResumeSessionInitializerTestFactory.php        |   2 +
 35 files changed, 821 insertions(+), 133 deletions(-)
 create mode 100644 src/Tui/Runtime/TuiRuntimeEventApplier.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/CommandHandler/ResumeHandlerTest.php
 create mode 100644 tests/CodingAgent/Runtime/InProcess/InProcessAttachDoesNotContinueTest.php
 create mode 100644 tests/Tui/Runtime/TuiRuntimeEventApplierTest.php
- Worktree cleanup failed: error: failed to delete '/home/ineersa/projects/agent-core-worktrees/resume-01-repair-tui-session-replay-and-working-state': Directory not empty

- IDEA exclusions preserved for /home/ineersa/projects/agent-core-worktrees/resume-01-repair-tui-session-replay-and-working-state because worktree removal failed.
- Pulled integration checkout: Merge made by the 'ort' strategy..
