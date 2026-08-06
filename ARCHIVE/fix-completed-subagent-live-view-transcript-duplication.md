# Fix completed subagent live-view transcript duplication and transient layout corruption

## Goal
User-reproduced TUI bug, split from `atomic-per-run-file-sequence-allocation` after its live-progress streaming fix.

## Reproduction

1. Launch multiple subagents.
2. Stay in the parent/main view until a child completes.
3. Open `/agents-live` and enter the completed child.
4. The completed child's transcript visibly repeats assistant/reasoning content; horizontal separators/footer/cursor layout may be corrupted or clipped.
5. Swapping main ↔ live repeatedly can transiently restore the footer/layout, but duplicate transcript content remains.

Important contrast: entering the child's live view while it is still running and watching through completion renders correctly. The defect is specifically completed-child entry/re-entry, not continuous live streaming.

## Evidence gathered

- Real session 5 child event logs contain unique canonical sequence numbers/events; no duplicate persisted events were found.
- A diagnostic full-projector test covered fresh completed replay, cached re-entry replay, empty post-replay poll, provider→poller replay, streaming assistant replay, and trailing completion noise. It found no duplicate `TranscriptBlock` IDs. Those exploratory passing tests were intentionally not retained in the atomic-sequence task.
- Therefore do not assume `SubagentLiveChildViewPoller::replaySnapshot()` literally duplicates block IDs. Investigate visible duplicate content under different IDs, cached entry state, `SubagentLivePickerController::enterLiveView()`, `ChatScreen::setTranscriptBlocks()`, widget invalidation/render reconciliation, and the first tick after completed-child entry.
- Historical picker arrow rerender defect was separately fixed by commit `c67241263`; arrow navigation is now smooth. Do not conflate that fixed picker issue with this transcript issue.
- Broad perf/background-poller/transcript-merge attempt `5d1dcfe96` broke live rendering and was reverted by `0e370e931`. Do not revive that design wholesale.

## Scope

Focus only on completed-child live-view entry/re-entry correctness. General TUI performance with several running subagents is explicitly out of scope unless the same proven root cause directly explains both.

Because replay/unit projections disagree with the real TUI reproduction, trust the live path. Capture exact block IDs/text and ANSI/screen state at the completed-child transition before selecting a fix. Prefer the smallest failing proof at the correct layer; if virtual/replay cannot reproduce the terminal corruption, add a minimal live-controller/TmuxHarness journey that follows the exact completed-before-entry path.

## Acceptance criteria
- Entering `/agents-live` for a child that completed before first entry renders each logical transcript item exactly once.
- Main ↔ completed-child live-view toggles preserve correct footer, separators, cursor placement, and final line without transient corruption.
- Entering while the child is running and watching through completion remains correct.
- A failing automated reproduction proves the exact user-visible path before the production fix; a projector-only duplicate-ID test is insufficient because that path already passed.
- Validation uses Castor only and includes the required TUI/runtime lanes plus `castor check` before CODE-REVIEW.
- No unrelated subagent performance refactor, background poller port, atomic sequence change, or picker-arrow change is included.

## Workflow metadata
Status: DONE
Branch: task/fix-completed-subagent-live-view-transcript-duplication
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-completed-subagent-live-view-transcript-duplication
Fork run: 63p8vbrhvxjz
PR URL: https://github.com/ineersa/agent-core/pull/290
PR Status: merged
Started: 2026-07-15T16:51:34.561Z
Completed: 2026-07-15T18:37:35.032Z

## Work log
- Created: 2026-07-11T21:07:40.888Z

## Task workflow update - 2026-07-15T16:51:28.074Z
- Summary: User approved implementation after PR #284 smoke/root-cause analysis. Current architecture confirmed: main view consumes parent `subagent_progress` only; JsonlProcessAgentSessionClient buffers raw RuntimeEvents for every unselected child; first live-view entry replays committed child events.jsonl, then drains the child's old raw buffer. Durable seq>0 overlap is ignored by cursor, but old transient seq=0 events cannot be deduplicated and can replay stale/duplicate content. Implement durable-checkpoint + compact-tail semantics: unselected child memory should retain only compact current non-durable stream/turn state; committed completion checkpoints prune superseded raw events; terminal cleanup removes remaining tail; HITL/tool-question/control events remain protected. Do not raise the 10k cap or merely suppress warnings. Existing parent stats, canonical child replay, selected-child live polling, steering, and continuous-live behavior must remain unchanged.

## Task workflow update - 2026-07-15T16:51:34.561Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-completed-subagent-live-view-transcript-duplication.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-completed-subagent-live-view-transcript-duplication.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-completed-subagent-live-view-transcript-duplication.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-completed-subagent-live-view-transcript-duplication.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-completed-subagent-live-view-transcript-duplication.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-completed-subagent-live-view-transcript-duplication.
- Summary: Starting approved durable-checkpoint + compact-tail implementation. Reusing the existing completed-child duplication task rather than creating a duplicate. Architecture scout already completed; implementation will be delegated in the new worktree. Test thesis: staying in main view while a child streams/completes must not retain replay-covered raw events or stale seq=0 deltas that duplicate/corrupt first completed-child live-view entry; selected continuous live streaming and protected HITL/control delivery remain correct.

## Task workflow update - 2026-07-15T16:52:47.633Z
- Recorded fork run: zrn9qae4pl1m
- Summary: Implementation fork launched in isolated worktree with durable-checkpoint + compact-tail scope. Fork instructed to read testing skill/tests AGENTS first, inspect actual RuntimeEvent completion semantics, preserve selected-child race correctness and protected controls, add 1–3 focused process-client/live-view proofs, use Castor only, commit once, and stop without push/PR/full gate.

## Task workflow update - 2026-07-15T16:56:59.229Z
- Summary: Parent verification found a blocking semantic defect in f130a4cef: `compactAppendTransient()` replaces repeated delta events with the latest chunk. Production `AssistantTextStreamSubscriber` emits incremental `payload['text']` chunks and `AssistantStreamProjectionSubscriber::onTextDelta()` appends each chunk, so latest-wins loses the middle of an in-flight child stream (e.g. started `A` + deltas `B`,`C` drains as `A`,`C`). Existing 5k-coalescing test asserts count only, not reconstructed content. Additional concern: actual transient completion events such as thinking/tool arguments may be seq=0, while f130a4cef only executes checkpoint pruning inside `seq > 0`; canonical assistant.message_completed correlation must be verified against block_id/step_id/message_id. Commit not accepted yet; launching narrow correction/audit fork.

## Task workflow update - 2026-07-15T16:57:19.479Z
- Recorded fork run: jxn1qwgzm3qf
- Summary: Launched correction/audit fork for incremental-delta data loss and real checkpoint/correlation semantics. Parent will wait for its result and will not duplicate investigation.

## Task workflow update - 2026-07-15T17:00:01.416Z
- Validation: castor test --filter='RuntimeEventPerRunCompactBufferTest|JsonlProcessAgentSessionClientEventBufferTest' — 16 tests, 55 assertions OK; castor phpstan --path=src/CodingAgent/Runtime/Process/RuntimeEventPerRunCompactBuffer.php — OK; castor deptrac — 0 violations; castor cs-check on touched files after cs-fix — clean; Forks confirmed .agents/skills/testing/SKILL.md and tests/AGENTS.md read before test work
- Summary: Implementation complete at HEAD f9dd76f88 (base compact-buffer commit f130a4cef plus correction). Added RuntimeEventPerRunCompactBuffer and wired JsonlProcessAgentSessionClient cross-run buffering through durable-checkpoint + compact-tail semantics. Correction fixes incremental-delta data loss by append-coalescing production payload keys (`text`, `thinking`, `partial_json`, `delta`), handles latest-wins `subagent_progress`, recognizes seq=0 stream checkpoints, and lets durable assistant.message_completed prune replay-covered text/thinking/tool-call tail across message_id/step_id/block_id correlation. Process-client + TranscriptProjector tests prove ABC content preservation and completed-child drain without stale seq=0 replay tail. Worktree clean; expected 4 files changed across two commits; git diff --check clean. Per task-start workflow, no reviewer/full gate/push/PR/status move performed.

## Task workflow update - 2026-07-15T17:09:02.540Z
- Summary: Manual live validation FAILED: user still sees event-buffer warning spam. Current implementation has an explicit unthrottled warning path in JsonlProcessAgentSessionClient: once compact tail count is >=10,000, every subsequent ingest logs `jsonl_event_buffer.tail_at_capacity`. This violates the task acceptance criterion even though old drop-oldest logging was removed. No local worktree logs were available to identify the exact Datadog event; investigating the remaining unbounded tail categories and a correctness-preserving bound before further edits. Live reproduction overrides green focused tests.

## Task workflow update - 2026-07-15T17:11:17.718Z
- Summary: Read-only failure investigation confirmed f9dd76f88 is not globally bounded: all seq>0 non-checkpoint child events are append-retained until events(childRunId) drains, but unselected parallel children are never drained from main view. This is the dominant remaining growth path; tail_at_capacity then logs on every increasing ingest >=10k. A safe correction must distinguish explicitly observed/selected child runs: unobserved child durable events are replayable and should not be retained, while selected children must retain events arriving during the parent-poll half of the child-first/parent-second race. Therefore activeRunId alone is insufficient; observation must begin before snapshot replay and end on live-view exit. Next correction will add a narrow observation lifecycle at AgentSessionClient boundary, preserve compact seq0 tails/protected controls for unobserved runs, clear replay-covered tail on unobserved terminal, and add parent-only multi-child pressure proof plus selected-child race proof. If Datadog event_type is drop_oldest rather than tail_at_capacity, user was running old/main binary and that must be reported separately.

## Task workflow update - 2026-07-15T17:17:22.460Z
- Validation: castor test --filter='JsonlProcessAgentSessionClientEventBufferTest|RuntimeEventPerRunCompactBufferTest|SubagentLivePickerObservationLifecycleTest' — 22 tests, 70 assertions OK (1 non-failing PHPUnit mock notice); castor phpstan — OK; castor deptrac — 0 violations; castor cs-check — 0 fixable; Fork confirmed .agents/skills/testing/SKILL.md and tests/AGENTS.md read
- Summary: Second live-failure correction complete at HEAD 706797523. Added explicit begin/end child observation lifecycle before snapshot replay and on switch/exit; JsonlProcessAgentSessionClient now retains durable half-race events only for explicitly observed children, while unobserved children retain compact seq0/protected control tail and drop replayable seq>0 backlog. Ending observation discards durable backlog. Capacity diagnostic is rate-limited instead of per-ingest. Added parent-only 4-child/12k realistic event pressure proof (<200 retained, zero capacity spam), selected child parent-poll half-race proof, picker observation lifecycle proof, and retention unit coverage. Worktree clean; 21 expected contract/process/TUI/test files changed; git diff --check clean. Awaiting user manual live re-test before task-to-pr.

## Task workflow update - 2026-07-15T17:50:09.350Z
- Validation: Reviewer read AGENTS/testing docs and returned APPROVE WITH SUGGESTIONS; no critical/security issues; castor test — 4404 tests, 14589 assertions, but exit 1 due 1 PHPUnit notice (mock without expectations)
- Summary: Task-to-PR reviewer verdict APPROVE WITH SUGGESTIONS, but orchestrator is treating one finding as blocking: during child-first polling, events(childRunId) can consume parent durable events from the same stdout chunk while activeRunId=child; parent is not in observedChildRunIds, so those parent events are dropped and the immediate parent poll cannot recover them until resume. Also full `castor test` failed (4404 tests/14589 assertions) solely on a PHPUnit notice: SubagentLivePickerObservationLifecycleTest uses createMock without expectations. Before CODE-REVIEW, fixing persistent primary/parent run retention across child polls, the notice, misleading durable-retention comment, redundant terminal protected branch, test-only tailSnapshotForTests API, and extra blank line; then re-review and rerun validation.

## Task workflow update - 2026-07-15T17:52:28.819Z
- Recorded fork run: e7vrvdjx9i0f
- Validation: castor test focused — 23 tests, 73 assertions OK; castor test — 4405 tests, 14592 assertions, exit 0, zero notices; castor phpstan — OK; castor deptrac — 0 violations; castor cs-check — 0 fixable
- Summary: Reviewer correction complete at HEAD 287f87fb3: persistent primary/session run retention prevents parent durable events consumed during child-first poll from being dropped; regression proves next parent poll receives event exactly once. PHPUnit notice removed with test-local fixed snapshot provider; misleading comment/dead branch/test-only production API/blank line cleaned. Worktree clean. Proceeding to reviewer re-check and task-to-PR validation.

## Task workflow update - 2026-07-15T17:59:09.176Z
- Summary: Latest reviewer again APPROVE WITH SUGGESTIONS/no critical or security issues, but found one latent contract bug worth closing before PR: StartRunRequest allows omitted runId, yet JsonlProcessAgentSessionClient sets primaryRunId only when request runId is explicit; on run.started it updates activeRunId but not primaryRunId, reintroducing parent-event loss during later child polls. Current TUI callers always pass explicit sessionId, so not seen in manual smoke. Narrow correction will set primaryRunId from run.started, test no-runId path, clear stale observedChildRunIds on new start/attach, and rename advisory MAX constant to threshold semantics.

## Task workflow update - 2026-07-15T18:01:11.785Z
- Recorded fork run: 63p8vbrhvxjz
- Validation: Focused castor test — 25 tests, 80 assertions OK; Scoped castor phpstan — OK; castor deptrac — 0 violations; castor cs-check — 0 fixable
- Summary: Final narrow reviewer correction complete at HEAD b197c3161: generated run.started now establishes primaryRunId+sessionId, stale observed child IDs reset on start/attach, advisory capacity constant renamed accurately. Proceeding to final reviewer and task-to-PR validation.

## Task workflow update - 2026-07-15T18:18:53.282Z
- Validation: castor test — 4407 tests, 14599 assertions OK; castor test:controller-replay — 8 tests, 112 assertions OK; castor test:tui — 36 tests, 186 assertions OK; castor phpstan — 0 errors; castor deptrac — 0 violations; castor cs-check — 0 fixable; git diff --check origin/main...HEAD — clean; Reviewer — APPROVE WITH SUGGESTIONS, no blockers; Manual live smoke — no event-buffer warning spam; behavior looks fine
- Summary: Final reviewer verdict APPROVE WITH SUGGESTIONS at HEAD b197c3161: no critical, security, correctness, or merge blockers. Reviewer confirmed completed-child stale seq0 pruning, incremental stream preservation, explicit selected-child observation lifecycle, primary parent retention (explicit/generated start and attach), unobserved bounded tail, protected controls, structured logging, and no test-only production APIs. User manual smoke confirmed event-buffer warnings no longer appear and behavior looks correct. Ready for deterministic CODE-REVIEW gate.

## Task workflow update - 2026-07-15T18:21:00.628Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 600s)...
- castor check passed (112.9s).
- Pushed task/fix-completed-subagent-live-view-transcript-duplication to origin.
- branch 'task/fix-completed-subagent-live-view-transcript-duplication' set up to track 'origin/task/fix-completed-subagent-live-view-transcript-duplication'.
- Created PR: https://github.com/ineersa/agent-core/pull/290
- Validation: castor test — 4407 tests, 14599 assertions OK; castor test:controller-replay — 8 tests, 112 assertions OK; castor test:tui — 36 tests, 186 assertions OK; castor phpstan — 0 errors; castor deptrac — 0 violations; castor cs-check — 0 fixable; Reviewer — APPROVE WITH SUGGESTIONS, no blockers; Manual live smoke — no event-buffer warning spam
- Summary: Implementation and iterative live-failure fixes complete at b197c3161. Durable-checkpoint + compact-tail buffering prevents replay-covered seq0 child events from duplicating completed live-view transcripts; unobserved child durable backlog is dropped, selected children use explicit observation lifecycle, primary parent events survive child-first polling, generated run IDs establish primary identity, and capacity diagnostics are bounded. Reviewer approved with no blockers; user manual smoke confirmed warnings are gone.

## Task workflow update - 2026-07-15T18:29:11.567Z
- Validation: Manual live smoke at b197c3161 — NOT YET PERFORMED (prior smoke was wrong worktree); Deterministic castor check — still passed in 112.9s; Reviewer — still approved with no blockers
- Summary: Correction: the previously recorded manual live smoke was performed in a different worktree, not this task worktree. Withdraw that manual evidence. Automated reviewer/Castor results remain valid, PR #290 remains open, but the exact live reproduction at HEAD b197c3161 is still unverified. Correct worktree: /home/ineersa/projects/agent-core-worktrees/fix-completed-subagent-live-view-transcript-duplication. Do not merge until the user retests there; if the bug or warning spam persists, move back to IN-PROGRESS for iteration.

## Task workflow update - 2026-07-15T18:37:35.032Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-completed-subagent-live-view-transcript-duplication into integration checkout.
- Merge made by the 'ort' strategy.
 .../Runtime/Contract/AgentSessionClient.php        |  13 +
 .../InProcess/InProcessAgentSessionClient.php      |   8 +
 .../Process/JsonlProcessAgentSessionClient.php     | 149 ++++---
 .../Process/RuntimeEventPerRunCompactBuffer.php    | 442 +++++++++++++++++++++
 src/Tui/Listener/AgentsMainCommandHandler.php      |   4 +-
 src/Tui/Listener/SubagentLiveCommandRegistrar.php  |   3 +-
 .../Listener/SubagentLiveToggleInputListener.php   |   4 +-
 src/Tui/Picker/SubagentLivePickerController.php    |  17 +-
 src/Tui/Runtime/SubagentLiveMainReturn.php         |   8 +-
 .../BackgroundProcessCompletionPollerTest.php      |   8 +
 .../CommandHandler/AnswerHumanHandlerTest.php      |   8 +
 .../CommandHandler/CompactHandlerTest.php          |   8 +
 .../CommandHandler/ResumeHandlerTest.php           |   8 +
 .../CommandHandler/ShellCommandHandlerTest.php     |   8 +
 .../Runtime/Controller/RuntimeEventEmitterTest.php |   8 +
 ...onlProcessAgentSessionClientEventBufferTest.php | 348 ++++++++++++++--
 .../RuntimeEventPerRunCompactBufferTest.php        | 226 +++++++++++
 tests/Tui/Listener/CompactCommandHandlerTest.php   |   8 +
 .../Listener/TickPollListenerSubagentLiveTest.php  |   8 +
 .../SubagentLivePickerObservationLifecycleTest.php | 141 +++++++
 tests/Tui/Support/RecordingAgentSessionClient.php  |   8 +
 21 files changed, 1323 insertions(+), 112 deletions(-)
 create mode 100644 src/CodingAgent/Runtime/Process/RuntimeEventPerRunCompactBuffer.php
 create mode 100644 tests/CodingAgent/Runtime/Process/RuntimeEventPerRunCompactBufferTest.php
 create mode 100644 tests/Tui/Picker/SubagentLivePickerObservationLifecycleTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-completed-subagent-live-view-transcript-duplication.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-completed-subagent-live-view-transcript-duplication.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #290 state MERGED at 2026-07-15T18:36:44Z; Deterministic castor check passed before PR; Reviewer approved with no blockers; User requested DONE transition
- Summary: PR #290 merged on GitHub at 36c6e562d8fff91225a80607317a20536aab45e8. Final implementation adds durable-checkpoint + compact-tail cross-run buffering, explicit selected-child observation, bounded unobserved child retention, primary parent retention across child-first polling, generated-run primary binding, and rate-limited capacity diagnostics.
