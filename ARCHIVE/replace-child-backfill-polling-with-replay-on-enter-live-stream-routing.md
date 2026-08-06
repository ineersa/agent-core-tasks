# Replace child backfill polling with replay-on-enter live stream routing

## Goal
Architecture follow-up from fork/subagent live-view debugging. Current `/agents-live` selected child view uses BackfillEventProvider/EventStore reads on ticks to reconstruct child transcript/HITL state. This should be replaced with the same conceptual model as `/resume`: replay canonical child run events once when entering a child live view, then consume normal RuntimeEvents from the controller stream filtered by child run_id. Background/catalog polling must never poll/read events.jsonl.

Plan document: `.pi/plans/subagent-live-child-replay-stream-plan.md`

Context: this should be implemented as a separate task, not folded into FORK-MVP-01, to avoid destabilizing the fork MVP branch.

## Acceptance criteria
- A detailed implementation plan exists at `.pi/plans/subagent-live-child-replay-stream-plan.md` and is linked from this task.
- Selected child live view replays canonical child events once on entry, not on every tick.
- Post-entry selected child updates come from normal controller RuntimeEvents filtered by child run_id.
- Background/catalog polling never reads child `events.jsonl` on ticks.
- Existing backfill-specific tests are replaced/rewritten to assert replay-on-enter + live stream routing semantics.
- Direct scout HITL, fork HITL, and fork→scout HITL live smoke contracts are covered with appropriate Castor validation before PR.

## Workflow metadata
Status: DONE
Branch: task/replace-child-backfill-polling-with-replay-on-enter-live-stream-routing
Worktree: /home/ineersa/projects/agent-core-worktrees/replace-child-backfill-polling-with-replay-on-enter-live-stream-routing
Fork run: s42iwglnwizq
PR URL: https://github.com/ineersa/agent-core/pull/273
PR Status: merged
Started: 2026-07-09T18:23:28.625Z
Completed: 2026-07-09T22:02:35.131Z

## Work log
- Created: 2026-07-09T18:06:54.350Z

## Task workflow update - 2026-07-09T18:10:02.236Z
- Summary: Created detailed implementation plan on main at `.pi/plans/subagent-live-child-replay-stream-plan.md`. Plan assumes implementor has zero prior context and covers current architecture, failure modes, target replay-on-enter + runtime stream design, phased implementation, files to inspect/change, tests, Castor validation, manual smoke, reviewer checklist, and suggested task splitting.

## Task workflow update - 2026-07-09T18:23:28.625Z
- Moved TODO → IN-PROGRESS.
- Created branch task/replace-child-backfill-polling-with-replay-on-enter-live-stream-routing.
- Created worktree /home/ineersa/projects/agent-core-worktrees/replace-child-backfill-polling-with-replay-on-enter-live-stream-routing.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/replace-child-backfill-polling-with-replay-on-enter-live-stream-routing.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/replace-child-backfill-polling-with-replay-on-enter-live-stream-routing.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/replace-child-backfill-polling-with-replay-on-enter-live-stream-routing.
- Summary: Started task after committing linked plan on main (`502e2c7fb docs: add child live replay stream plan`) and reading `.pi/plans/subagent-live-child-replay-stream-plan.md` in full. Implementation will be split into fork-sized phases from the plan.

## Task workflow update - 2026-07-09T18:24:15.739Z
- Recorded fork run: ep8skljgsvhz
- Summary: Launched fork `ep8skljgsvhz` for Part A only: runtime stream contract proof. Fork instructions require reading the full plan first, loading testing skill and `tests/AGENTS.md`, strengthening `JsonlProcessAgentSessionClientEventBufferTest` plus streaming/emitter coverage, Castor focused validation, and committing changes. Later parts intentionally deferred: B) replay/snapshot service + replay-on-enter, C) remove per-tick backfill and update TUI/runtime tests.

## Task workflow update - 2026-07-09T18:28:10.016Z
- Recorded fork run: cp7v7viusiw9
- Summary: Part A inspection: commit `887a4b545` is test-only and focused tests pass locally. `castor cs-check` reported formatting issues in the two changed test files, so launched follow-up fork `cp7v7viusiw9` to run Castor CS fixer, verify style/tests, and commit CS-only changes before Part B.

## Task workflow update - 2026-07-09T18:30:14.076Z
- Validation: castor test --filter=JsonlProcessAgentSessionClientEventBufferTest — PASS (6 tests, 27 assertions); castor test --filter=StreamingCommittedRuntimeEventStoreTest — PASS (4 tests, 11 assertions); castor cs-check — PASS after commit 70be14350
- Summary: Part A complete and checked. Fork `ep8skljgsvhz` added test-only runtime stream routing proof in commit `887a4b545`; follow-up fork `cp7v7viusiw9` fixed CS in commit `70be14350`. Orchestrator verified worktree log/status, re-ran `castor cs-check` successfully, and accepted Part A. No production changes yet.

## Task workflow update - 2026-07-09T18:30:47.179Z
- Recorded fork run: nonzbtff86yc
- Summary: Launched Part B1 fork `nonzbtff86yc`: introduce child-run transcript snapshot provider/DTO/interface plus focused tests and service wiring only. Explicitly out of scope for this fork: picker entry wiring, child poller changes, backfill removal, HITL overlay replay.

## Task workflow update - 2026-07-09T18:33:07.098Z
- Validation: castor test --filter=ChildRunTranscriptSnapshotProviderTest — PASS (2 tests, 8 assertions); castor cs-check — PASS; castor phpstan --path=src/CodingAgent/Session/ChildRunTranscriptSnapshotProvider.php — PASS
- Summary: Part B1 complete and checked. Fork `nonzbtff86yc` added `ChildRunTranscriptSnapshotProviderInterface`, `ChildRunTranscriptSnapshotDTO`, `ChildRunTranscriptSnapshotProvider`, isolated `child_run.transcript.*` service wiring, and focused tests in commit `249924576`. Orchestrator inspected diff/status and accepted. No TUI entry wiring or backfill removal yet.

## Task workflow update - 2026-07-09T18:33:49.999Z
- Recorded fork run: z6wr46nzh3hg
- Summary: Launched Part B2 fork `z6wr46nzh3hg`: wire child snapshot replay-on-enter into `/agents-live`, seed child poller projector/state, and route stored HITL/tool-question replay callbacks while keeping existing per-tick backfill for Part C.

## Task workflow update - 2026-07-09T18:40:14.232Z
- Validation: castor test --filter=SubagentLiveChildViewPoller — PASS (6 tests, 43 assertions); castor test --filter=SubagentLivePickerControllerTest — PASS (5 tests, 17 assertions); castor test --filter=SubagentLiveCommandRegistrarTest — PASS (1 test, 3 assertions); castor test --filter=SubagentLiveViewStateTest — PASS (3 tests, 6 assertions); castor cs-check — PASS; castor phpstan --path=src/Tui/Runtime/SubagentLiveChildViewPoller.php --path=src/Tui/Picker/SubagentLivePickerController.php --path=src/Tui/Runtime/SubagentLiveViewState.php --path=src/Tui/Listener/SubagentLiveCommandRegistrar.php — PASS
- Summary: Part B2 complete and checked. Fork `z6wr46nzh3hg` added replay-on-enter wiring in commit `7826541da`: picker calls child snapshot provider on first child entry, `SubagentLiveChildViewPoller::replaySnapshot()` seeds child state/projector and dispatches stored HITL/tool callbacks, view state caches replay events, and registrar passes the runtime client. Existing per-tick backfill remains for Part C. Orchestrator inspected changed production/tests, verified clean git status, and accepted B2 with one noted follow-up risk: cached re-entry currently replays cached events without question callbacks; final review/live test should confirm desired remount policy.

## Task workflow update - 2026-07-09T18:41:02.648Z
- Recorded fork run: 8h3874alhpp3
- Summary: Launched Part C fork `8h3874alhpp3`: remove per-tick child `BackfillEventProvider` path from `SubagentLiveChildViewPoller`, delete obsolete backfill contract/implementation/wiring/tests if unreferenced, and keep replay-on-enter + live-stream-only tick polling.

## Task workflow update - 2026-07-09T19:19:30.792Z
- Validation: castor test --filter=SubagentLiveChildViewPoller — PASS (3 tests, 17 assertions); castor test --filter=SubagentLivePickerControllerTest — PASS (5 tests, 17 assertions); castor test --filter=SubagentLiveCommandRegistrarTest — PASS (1 test, 3 assertions); castor test --filter=ChildRunTranscriptSnapshotProviderTest — PASS (2 tests, 8 assertions); castor phpstan --path=src/Tui/Runtime/SubagentLiveChildViewPoller.php --path=src/Tui/Picker/SubagentLivePickerController.php --path=src/Tui/Runtime/SubagentLiveViewState.php --path=src/CodingAgent/Session/ChildRunTranscriptSnapshotProvider.php — PASS; castor cs-check — PASS; rg BackfillEventProvider|ChildRunBackfillEventProvider|backfillProvider|getStoredEvents over src/tests/config PHP/YAML — no matches; castor deptrac — FAIL: 2 violations in src/Tui/Picker/SubagentLivePickerController.php depending on Ineersa\Tui\Question\QuestionCoordinator and QuestionController (TuiPicker on TuiQuestion)
- Summary: Part C implementation accepted for its scoped removal: fork `8h3874alhpp3` committed `01ef8bf0e` removing per-tick child backfill polling, deleting `BackfillEventProviderInterface`, `ChildRunBackfillEventProvider`, the DI alias/injection, and backfill-only test file. Orchestrator inspected diff, confirmed no PHP/YAML references remain, and re-ran focused validations. Overall task is not ready yet because `castor deptrac` now fails on B2-introduced `SubagentLivePickerController` dependencies on TUI question services; launching a follow-up architecture-boundary fork before any review/PR.

## Task workflow update - 2026-07-09T19:20:03.875Z
- Recorded fork run: zyb36qy9zw9v
- Summary: Launched Part D fork `zyb36qy9zw9v`: fix Deptrac boundary by moving child replay HITL/tool-question callback construction out of `SubagentLivePickerController` (TuiPicker) into listener-layer wiring, preserving replay behavior and requiring `castor deptrac` pass.

## Task workflow update - 2026-07-09T19:22:26.202Z
- Validation: castor deptrac — PASS (0 violations); castor test --filter=SubagentLivePickerControllerTest — PASS (5 tests, 17 assertions); castor test --filter=SubagentLiveCommandRegistrarTest — PASS (1 test, 3 assertions); castor test --filter=SubagentLiveToggleInputListenerTest — PASS (1 test, 1 assertion); castor test --filter=SubagentLiveChildViewPoller — PASS (3 tests, 17 assertions); castor phpstan --path=src/Tui/Picker/SubagentLivePickerController.php --path=src/Tui/Listener/SubagentLiveCommandRegistrar.php — PASS; castor cs-check — PASS
- Summary: Part D boundary fix complete and accepted. Fork `zyb36qy9zw9v` committed `57a957358` plus style follow-up `e349e703e`: moved replay-on-enter question callback construction from `SubagentLivePickerController` (TuiPicker) into `SubagentLiveCommandRegistrar` (TuiListener), preserving first-entry HITL/tool callback behavior while removing picker dependencies on `TuiQuestion`. Orchestrator inspected changed files/diff, confirmed clean git status, and re-ran focused validations. All planned implementation parts A-D are now complete; remaining before PR/review is broader gate/reviewer/live validation per user workflow. Known product gap remains unchanged: cached child re-entry replays cached events without remounting HITL callbacks.

## Task workflow update - 2026-07-09T19:50:36.359Z
- Validation: Reviewer ran castor deptrac — PASS (0 violations); Reviewer ran castor phpstan — PASS (0 errors); Reviewer ran castor cs-check — PASS; Reviewer ran focused tests: SubagentLiveChildViewPollerReplayTest, SubagentLivePickerControllerTest, ChildRunTranscriptSnapshotProviderTest, JsonlProcessAgentSessionClientEventBufferTest, StreamingCommittedRuntimeEventStoreTest, SubagentLiveCommandRegistrarTest, SubagentLiveToggleInputListenerTest — all PASS; Reviewer did not run full castor check
- Summary: Reviewer subagent completed task-to-pr review with verdict REQUEST CHANGES. Blocking finding: cached child re-entry can lose live transcript blocks because `SubagentLiveChildViewPoller::poll()` updates cached `childTranscript`/`childLastSeq` after live events but does not append those applied live `RuntimeEvent`s to `childReplayEvents`; cached re-entry calls `replaySnapshot()` with replay-only events, resets projector, and can overwrite replay+live cached transcript with replay-only projected blocks. Reviewer recommended appending each applied live event to `childReplayEvents` and adding a regression test for non-empty replay events + live poll + cached re-entry/reprojection. Non-blocking notes: cached HITL callback remount remains an acceptable follow-up, redundant reset before snapshot, possibly dead `shouldResetProjectionFor`, test reflection cleanup.

## Task workflow update - 2026-07-09T19:51:02.411Z
- Recorded fork run: s42iwglnwizq
- Summary: Launched reviewer-fix fork `s42iwglnwizq` to address blocking cached re-entry transcript corruption: append accepted live events to `childReplayEvents` in `SubagentLiveChildViewPoller::poll()` and add regression proof for non-empty replay events + live poll + cached re-entry/reprojection preserving live blocks.

## Task workflow update - 2026-07-09T19:54:36.040Z
- Validation: castor test --filter=SubagentLiveChildViewPollerReplayTest — PASS (4 tests, 24 assertions); castor test --filter=SubagentLiveChildViewPoller — PASS (4 tests, 24 assertions); castor deptrac — PASS (0 violations); castor phpstan --path=src/Tui/Runtime/SubagentLiveChildViewPoller.php — PASS; castor phpstan — PASS (0 errors); castor cs-check — PASS; git status --short --branch in task worktree — clean
- Summary: Reviewer-fix fork `s42iwglnwizq` completed and was accepted. Commit `28e2448d6` fixes cached child live-view re-entry transcript corruption by appending each accepted/applied live `RuntimeEvent` to `SubagentLiveViewState::$childReplayEvents` in `SubagentLiveChildViewPoller::poll()` (stale skipped events are not appended). Added regression `cachedReentryReprojectsReplayAndLiveEvents()` proving non-empty replay events + live poll + cached re-entry/reprojection preserves both seed and live-tail transcript blocks and persists both replay events in cache. Orchestrator inspected diff/status and re-ran focused validation. Note: fork's scoped phpstan on the test file itself still hits existing PHPUnit dynamic assert warnings if requested path includes tests, but full project `castor phpstan` and source-scoped phpstan pass.

## Task workflow update - 2026-07-09T20:00:55.648Z
- Validation: Reviewer re-ran castor test --filter=SubagentLiveChildViewPollerReplayTest — PASS (4 tests, 24 assertions); Reviewer re-ran castor test --filter=SubagentLiveChildViewPoller — PASS (4 tests, 24 assertions); Reviewer re-ran castor test --filter=ChildRunTranscriptSnapshotProviderTest — PASS (2 tests, 8 assertions); Reviewer re-ran castor test --filter=SubagentLivePickerControllerTest — PASS (5 tests, 17 assertions); Reviewer re-ran castor test --filter=JsonlProcessAgentSessionClientEventBufferTest — PASS (6 tests, 27 assertions); Reviewer re-ran castor test --filter=StreamingCommittedRuntimeEventStoreTest — PASS (4 tests, 11 assertions); Reviewer re-ran castor deptrac — PASS (0 violations); Reviewer re-ran castor phpstan --path=src/Tui/Runtime/SubagentLiveChildViewPoller.php — PASS (0 errors); Reviewer re-ran castor cs-check — PASS (0 files fixed); Reviewer confirmed git status clean
- Summary: Re-review after fix commit `28e2448d6` completed with verdict APPROVED. Reviewer confirmed previous cached re-entry transcript corruption blocker is fixed: accepted live events are appended to `childReplayEvents`, cached re-entry restores replay+live event history, and the new regression test would fail before the fix. No blocking findings remain. Non-blocking follow-ups/risks: cached child re-entry still does not remount HITL callbacks (question text visible but overlay not rebuilt; acceptable follow-up), minor test reflection cleanup, seq=0 duplication edge case acceptable. Reviewer says branch is ready for broader `castor check` and user live testing.

## Task workflow update - 2026-07-09T21:24:31.878Z
- Validation: Scout read-only inspected .hatfield/sessions/1/events.jsonl, state.json, idempotency.jsonl, artifacts/agents, and .hatfield/logs/agent-2026-07-09.log; Scout observed active processes in worktree: TUI hatfield.phar agent, controller hatfield.phar agent --controller, messenger consumers; no processes killed
- Summary: Live manual test investigation: user reported session `1` stuck in Cancelling, so scout inspected worktree `.hatfield/sessions/1` and logs read-only. Finding: on-disk `state.json` is terminal `failed` with error `Cancel rejected: run is already in terminal state (failed).`, while TUI appears stuck Cancelling due repeated cancel commands after terminal failure. Root evidence: `events.jsonl` contains duplicate sequence numbers involving cancel commands and subagent progress/end events (seq 18 and 36; also seq 37 collision in tail), and log includes `run_state_replay.duplicate_sequences` for run_id=1 with duplicate_sequences=[18,36]. Active TUI/controller/messenger processes remain alive; scout did not signal/kill anything. This is a new blocking live-test issue likely involving concurrent event-store sequence assignment between cancel command application and subagent `tool_execution_update` progress writes, not the cached re-entry bug previously reviewed.

## Task workflow update - 2026-07-09T21:49:59.379Z
- Validation: git log --all --grep='allocate event sequences atomically|stream sequenced subagent progress events' found commits 44da56263 and e97e43372 on fork-mvp branch; git merge-base --is-ancestor 44da56263 HEAD in current task worktree returned false; git merge-base --is-ancestor e97e43372 HEAD in current task worktree returned false; test -f src/AgentCore/Contract/SequencedEventStoreInterface.php in current task worktree returned false
- Summary: Follow-up analysis on duplicate sequence live-test failure: atomic event sequence assignment was already implemented in separate IN-PROGRESS `fork-mvp-01-fork-tool-over-child-run-backend` branch/PR, not in this replay-on-enter task branch. Git evidence: fork-mvp branch contains commits `44da56263 fix(session): allocate event sequences atomically` and `e97e43372 fix(runtime): stream sequenced subagent progress events`; current task branch HEAD does not contain either commit and lacks `src/AgentCore/Contract/SequencedEventStoreInterface.php` / `appendWithNextSeq` code. This explains why live testing on this worktree can still hit duplicate sequence numbers on concurrent cancel + subagent progress writes.

## Task workflow update - 2026-07-09T21:56:30.657Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (139.6s).
- Pushed task/replace-child-backfill-polling-with-replay-on-enter-live-stream-routing to origin.
- branch 'task/replace-child-backfill-polling-with-replay-on-enter-live-stream-routing' set up to track 'origin/task/replace-child-backfill-polling-with-replay-on-enter-live-stream-routing'.
- Created PR: https://github.com/ineersa/agent-core/pull/273
- Validation: Pre-move git status in worktree clean; Branch HEAD: 28e2448d6 fix: preserve child live events across cached reentry; Reviewer APPROVED after fix; focused tests/deptrac/phpstan/cs-check previously recorded green; Initial move_task CODE-REVIEW castor check failed only on llama-proxy cache growth guard (251 -> 256 entries); Warmup: castor test:llm-real PASS (10 tests, 121 assertions, 26.3s); Proxy cache stats after warmup: entries=256
- Summary: User approved moving to CODE-REVIEW with known live-test caveat: duplicate sequence failure observed during manual cancellation is due to atomic sequencing fixes living on separate `fork-mvp-01-fork-tool-over-child-run-backend` branch (`44da56263`, `e97e43372`), not this replay-on-enter branch. User plans to merge/test the combined branches together. First CODE-REVIEW transition attempt failed because `castor check` detected llama-proxy cache growth from 251 to 256 entries; ran intentional warmup `castor test:llm-real` successfully and verified proxy cache stats at 256 entries before retrying.

## Task workflow update - 2026-07-09T22:02:35.131Z
- Moved CODE-REVIEW → DONE.
- Merged task/replace-child-backfill-polling-with-replay-on-enter-live-stream-routing into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                               |  20 +-
 .../Contract/BackfillEventProviderInterface.php    |  35 ---
 .../Contract/ChildRunTranscriptSnapshotDTO.php     |  25 ++
 ...ChildRunTranscriptSnapshotProviderInterface.php |  15 ++
 .../Process/ChildRunBackfillEventProvider.php      |  70 -----
 .../Session/ChildRunTranscriptSnapshotProvider.php |  63 +++++
 src/Tui/Listener/SubagentLiveCommandRegistrar.php  |  32 ++-
 src/Tui/Picker/SubagentLivePickerController.php    |  59 ++++-
 src/Tui/Runtime/SubagentLiveChildViewPoller.php    | 115 +++++++--
 src/Tui/Runtime/SubagentLiveViewState.php          |   9 +-
 ...onlProcessAgentSessionClientEventBufferTest.php |  76 ++++++
 .../StreamingCommittedRuntimeEventStoreTest.php    |  16 ++
 .../ChildRunTranscriptSnapshotProviderTest.php     | 111 ++++++++
 .../Listener/SubagentLiveCommandRegistrarTest.php  |   8 +
 .../SubagentLiveToggleInputListenerTest.php        |  12 +-
 .../Picker/SubagentLivePickerControllerTest.php    |  58 ++++-
 .../SubagentLiveChildViewPollerBackfillTest.php    | 284 ---------------------
 .../SubagentLiveChildViewPollerReplayTest.php      | 246 ++++++++++++++++++
 18 files changed, 820 insertions(+), 434 deletions(-)
 delete mode 100644 src/CodingAgent/Runtime/Contract/BackfillEventProviderInterface.php
 create mode 100644 src/CodingAgent/Runtime/Contract/ChildRunTranscriptSnapshotDTO.php
 create mode 100644 src/CodingAgent/Runtime/Contract/ChildRunTranscriptSnapshotProviderInterface.php
 delete mode 100644 src/CodingAgent/Runtime/Process/ChildRunBackfillEventProvider.php
 create mode 100644 src/CodingAgent/Session/ChildRunTranscriptSnapshotProvider.php
 create mode 100644 tests/CodingAgent/Session/ChildRunTranscriptSnapshotProviderTest.php
 delete mode 100644 tests/Tui/Runtime/SubagentLiveChildViewPollerBackfillTest.php
 create mode 100644 tests/Tui/Runtime/SubagentLiveChildViewPollerReplayTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/replace-child-backfill-polling-with-replay-on-enter-live-stream-routing.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/replace-child-backfill-polling-with-replay-on-enter-live-stream-routing.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: CODE-REVIEW transition previously passed deterministic castor check (139.6s); PR created: https://github.com/ineersa/agent-core/pull/273
- Summary: User explicitly requested moving to DONE despite known caveat that manual combined validation will happen with fork-mvp branch containing atomic sequence fixes. CODE-REVIEW gate had passed and PR was created at https://github.com/ineersa/agent-core/pull/273.
