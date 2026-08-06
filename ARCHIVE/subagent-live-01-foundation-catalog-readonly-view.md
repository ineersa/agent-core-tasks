# SUBAGENT-LIVE-01 Production live-view foundation

## Goal
Productionize the successful POC foundation for subagent live view. Reference POC remains intentionally available at worktree `/home/ineersa/projects/agent-core-worktrees/subagent-live-view-steering-poc`, branch `task/subagent-live-view-steering-poc`, latest known POC head `b5c6fca29`. Planning document: `.aiassistant/fork/subagent-live-view-production-plan.md`.

POC evidence/lessons to carry forward:
- Parent TUI can observe child runs through child event streams and project child transcript live.
- Parent must keep polling even while child view is active; otherwise parent transcript/status stalls.
- Process-mode event forwarding must register child run IDs; child events exist in artifact stores but are not visible over JSONL unless controller forwards them.
- JsonlProcessAgentSessionClient::events() must not drop non-matching run-id events when child and parent polling share one JSONL pipe; POC fixed this via re-buffering.
- Severe flicker happened when cached child transcript was repainted every tick; render only on new child blocks and dedupe status/working updates.
- Use POC code as reference, not as direct production shape; extract logic out of TickPollListener where possible.

Scope:
- Production child catalog for active/recent subagents with artifact ID, child run ID, agent name, task excerpt, status, last activity, terminal/attention flags.
- `/agents-live` picker/list that can select a subagent.
- Readonly child transcript/live view for selected child.
- `/agents-main` return to parent screen.
- Parent polling always active while child view is displayed.
- Process-mode child event forwarding and non-dropping event buffering hardened.
- No steering/HITL/cancel in this first task except preserving safe behavior/no crashes.

Architecture guidance:
- Keep execution lifecycle in CodingAgent subagent services; live view is TUI/runtime observation/control.
- Do not couple lifecycle state to rendered transcript text; use structured artifact/run metadata and runtime events.
- Keep TUI depending only on runtime contracts/protocols, not AgentCore internals.
- Prefer typed statuses/enums once stable.
- Preserve comments explaining event-buffering and no-flicker rationale.

## Acceptance criteria
- `/agents-live` shows known subagent child runs with agent name, artifact id, child run id, status, and task excerpt.
- Selecting a child opens a readonly live transcript view and `/agents-main` returns to parent.
- Parent runtime polling continues while child live view is active; parent completion/progress is not starved.
- Process-mode child events are forwarded to the TUI once child run id is known.
- Polling child events does not drop parent JSONL events or non-matching run-id events.
- No per-tick transcript repaint/flicker regression: screen updates child transcript only when new child blocks arrive.
- Automated proof at lowest appropriate layer for picker/open/return and event-buffering/parent-polling contracts; use Castor only and follow testing docs.
- Focused validation recorded; TUI/runtime scope requires appropriate Castor validation before CODE-REVIEW.

## Workflow metadata
Status: DONE
Branch: task/subagent-live-01-foundation-catalog-readonly-view
Worktree: /home/ineersa/projects/agent-core-worktrees/subagent-live-01-foundation-catalog-readonly-view
Fork run: qovv55cfc1ao
PR URL: https://github.com/ineersa/agent-core/pull/251
PR Status: merged
Started: 2026-07-02T19:46:52.484Z
Completed: 2026-07-02T22:02:51.848Z

## Work log
- Created: 2026-07-01T16:55:59.709Z

## Task workflow update - 2026-07-02T19:46:52.484Z
- Moved TODO → IN-PROGRESS.
- Created branch task/subagent-live-01-foundation-catalog-readonly-view.
- Created worktree /home/ineersa/projects/agent-core-worktrees/subagent-live-01-foundation-catalog-readonly-view.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/subagent-live-01-foundation-catalog-readonly-view.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/subagent-live-01-foundation-catalog-readonly-view.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/subagent-live-01-foundation-catalog-readonly-view.
- Summary: Claiming task-start phase. Read task file, referenced production plan, testing skill, and tests/AGENTS.md. Preparing scouts for code exploration and fork implementation instructions.

## Task workflow update - 2026-07-02T19:58:21.311Z
- Summary: Context gathered for implementation. Read testing skill and tests/AGENTS.md. Ran three read-only scout subagents: (1) POC comparison identified Phase 1 source files/services and rationale comments; (2) TUI command/picker/TmuxHarness patterns; (3) runtime/process/subagent event plumbing and contract test targets. Worktree: /home/ineersa/projects/agent-core-worktrees/subagent-live-01-foundation-catalog-readonly-view. Key implementation areas: SubagentProgressSnapshotBuilder + SubagentExecutionService agent_run_id payloads, JsonlProcessAgentSessionClient non-matching event re-buffering, RuntimeEventEmitter child-run registration from subagent_progress, TUI SubagentLive catalog/view/picker/commands, TickPollListener child-first + parent-always polling/no-flicker updates, isolated child TranscriptProjector service, and TmuxHarness proof for /agents-live → select → /agents-main.

## Task workflow update - 2026-07-02T19:59:38.833Z
- Recorded fork run: c9jwd8f60w45
- Summary: Implementation fork launched on worktree /home/ineersa/projects/agent-core-worktrees/subagent-live-01-foundation-catalog-readonly-view with Phase 1 readonly subagent live-view scope, explicit TmuxHarness E2E requirement, process event-buffer/forwarding tests, and focused Castor validation instructions. Fork run: c9jwd8f60w45.

## Task workflow update - 2026-07-02T20:00:03.368Z
- Validation: git -C /home/ineersa/projects/agent-core-worktrees/subagent-live-01-foundation-catalog-readonly-view status --short => clean; git -C /home/ineersa/projects/agent-core-worktrees/subagent-live-01-foundation-catalog-readonly-view log --oneline --decorate -5 => HEAD 3dbeffb9d (task branch/main), no task commit
- Summary: Implementation fork c9jwd8f60w45 failed before making changes. Reported failure: `TMUX_PANE=%8 (pane PID 188350) is not associated with the current Pi process (PID 2)`. Verified worktree remains clean and HEAD is still 3dbeffb9d; no implementation commit exists. This is an environment/tooling blocker for fork-based implementation, not a code failure.

## Task workflow update - 2026-07-02T21:03:17.005Z
- Summary: User clarified the previous fork failure note was a bubblewrap retry fork issue. Proceeding with another implementation attempt rather than treating it as a task/code blocker.

## Task workflow update - 2026-07-02T21:03:55.145Z
- Recorded fork run: fmxgtvj2j8nb
- Summary: Relaunched implementation fork after user clarified prior failure as bubblewrap retry issue. New implementation fork run: fmxgtvj2j8nb. Scope remains Phase 1 readonly subagent live-view foundation with required TmuxHarness E2E and focused Castor validation; no push/PR/task moves by fork.

## Task workflow update - 2026-07-02T21:13:49.811Z
- Validation: git -C /home/ineersa/projects/agent-core-worktrees/subagent-live-01-foundation-catalog-readonly-view status --short => clean; git -C /home/ineersa/projects/agent-core-worktrees/subagent-live-01-foundation-catalog-readonly-view log --oneline --decorate -3 => HEAD 161c7c1db feat(tui): add readonly subagent live view foundation; git -C /home/ineersa/projects/agent-core-worktrees/subagent-live-01-foundation-catalog-readonly-view diff --stat HEAD~1..HEAD => 23 files changed, 1356 insertions(+), 3 deletions(-); Fork reported: castor test --filter=SubagentLiveCatalogTest OK (4 tests); Fork reported: castor test --filter=JsonlProcessAgentSessionClientEventBufferTest OK (1 test); Fork reported: castor test --filter=testEmitRegistersChildRunFromSubagentProgress OK (1 test); Fork reported: castor test --filter=TickPollListenerSubagentLiveTest OK (1 test); Fork reported: combined targeted filter OK (7 tests); Fork reported: castor test:tui --filter=TuiSubagentLiveViewE2eTest OK (~4.5s, tmux available); Fork reported: castor deptrac OK; Fork reported: castor phpstan OK; Fork reported: castor cs-check OK after castor cs-fix
- Summary: Implementation fork fmxgtvj2j8nb completed successfully. Commit 161c7c1db9d59649ff6f4e9877acc0ef46544c42 (`feat(tui): add readonly subagent live view foundation`) is present on branch task/subagent-live-01-foundation-catalog-readonly-view. Verified worktree is clean and diff stat matches fork report: 23 files changed, 1356 insertions(+), 3 deletions(-). Phase 1 readonly live-view scope implemented: progress payload child run ids, JSONL non-matching event re-buffering, RuntimeEventEmitter child-run cursor registration, TUI live catalog/view/poller/commands, child-first plus parent-always tick polling, isolated child transcript projector, and tests including real TmuxHarness E2E. Fork explicitly read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` and used Castor-only validation.

## Task workflow update - 2026-07-02T21:15:28.400Z
- Validation: move_task IN-PROGRESS -> CODE-REVIEW failed: Castor check FAILED (exit code 1); quality failed: test exit code 2, test:tui exit code 2, test:llm-real exit code 1
- Summary: Attempted CODE-REVIEW transition after fork completion, but workflow castor gate failed before PR creation. Task remains IN-PROGRESS. move_task reported: `quality failed: test exit code 2; test:tui exit code 2; test:llm-real exit code 1`. User then instructed: `reviewer first`. Next step is read-only reviewer pass before further PR transition attempts.

## Task workflow update - 2026-07-02T21:34:17.471Z
- Validation: Reviewer ran `castor test` => FAIL: TickPollListenerTest uninitialized subagentLiveChildPoller in reflection-built tests; Reviewer ran `castor test --filter TickPollListenerSubagentLive` => OK; Reviewer ran `castor test --filter SubagentLiveCatalog` => OK; Reviewer ran `castor test --filter JsonlProcessAgentSessionClientEventBuffer` => OK; Reviewer ran `castor test --filter RuntimeEventEmitter` => OK; Reviewer ran `castor test:tui` => OK (27 tests, 140 assertions); gate-time tui failure identified as unrelated TuiQueuedSteerE2eTest flake; Reviewer ran `castor test:llm-real` => FAIL: SubagentRetrieveLiveE2eTest and SubagentParallelLiveE2eTest missing tool_execution.completed because child run.completed terminates harness collection early; Reviewer ran `castor phpstan` => OK; Reviewer ran `castor deptrac` => OK; Reviewer ran `castor cs-check` => OK
- Summary: Read-only reviewer pass completed after user requested reviewer first. Reviewer verdict: REQUEST CHANGES. The implementation structure is sound, targeted tests and static checks pass, but two blocking regressions must be fixed before PR: (1) existing reflection-based TickPollListenerTest methods construct TickPollListener without initializing new `subagentLiveChildPoller` typed property, causing `Typed property ... must not be accessed before initialization` in `castor test`; (2) controller E2E harness treats any multiplexed child `run.completed` as terminal for the parent stream, so `test:llm-real` stops before parent `tool_execution.completed` in SubagentRetrieveLiveE2eTest/SubagentParallelLiveE2eTest. Reviewer also noted gate-time `test:tui` failure was likely pre-existing flake; `castor test:tui` passed during review.

## Task workflow update - 2026-07-02T21:34:38.471Z
- Recorded fork run: qovv55cfc1ao
- Summary: Launched fix worker fork qovv55cfc1ao to address reviewer blockers: initialize new TickPollListener subagentLiveChildPoller dependency in existing reflection-based tests, update controller E2E collection helpers to demultiplex child run terminal events, remove spurious NullLogger arg/import in TickPollListenerSubagentLiveTest, run focused Castor validation, and commit fixes.

## Task workflow update - 2026-07-02T21:38:44.940Z
- Validation: git status --short in worktree => clean; git log --oneline --decorate -5 => HEAD 4c8a4d8fa test(tui): account for multiplexed subagent live events; previous 161c7c1db feat(tui): add readonly subagent live view foundation; git diff --stat HEAD~1..HEAD => 3 files changed, 72 insertions(+), 6 deletions(-); git diff --stat 3dbeffb9d..HEAD => 25 files changed, 1426 insertions(+), 7 deletions(-); Fork qovv55cfc1ao reported: castor test --filter testAwaitingFreeFormGuardPreventsReOpen OK; Fork qovv55cfc1ao reported: castor test --filter testOrphanedQuestionHealedWhenRunTerminal OK; Fork qovv55cfc1ao reported: castor test --filter TickPollListenerTest OK (23 tests); Fork qovv55cfc1ao reported: castor test --filter TickPollListener OK (24 tests); Fork qovv55cfc1ao reported: castor test:llm-real --filter SubagentRetrieveLiveE2eTest OK (~7.5s); Fork qovv55cfc1ao reported: castor test:llm-real --filter SubagentParallelLiveE2eTest OK (~6.3s); Fork qovv55cfc1ao reported: castor cs-check OK
- Summary: Fix worker fork qovv55cfc1ao completed successfully. New commit 4c8a4d8fa (`test(tui): account for multiplexed subagent live events`) fixes both reviewer blockers: existing reflection-based TickPollListener tests now initialize the new `subagentLiveChildPoller` dependency; controller E2E collectors now demultiplex terminal run lifecycle events so forwarded child `run.completed` does not end parent collection before parent `tool_execution.completed`. Also removed the spurious NullLogger constructor argument/import in TickPollListenerSubagentLiveTest. Verified clean worktree and branch now contains feature commit 161c7c1db plus fix commit 4c8a4d8fa.

## Task workflow update - 2026-07-02T21:40:30.914Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (87.0s).
- Pushed task/subagent-live-01-foundation-catalog-readonly-view to origin.
- branch 'task/subagent-live-01-foundation-catalog-readonly-view' set up to track 'origin/task/subagent-live-01-foundation-catalog-readonly-view'.
- Created PR: https://github.com/ineersa/agent-core/pull/251
- Validation: git status --short in worktree => clean; git diff --stat 3dbeffb9d..HEAD => 25 files changed, 1426 insertions(+), 7 deletions(-); Feature fork fmxgtvj2j8nb reported targeted Castor tests OK, TmuxHarness E2E OK, deptrac OK, phpstan OK, cs-check OK; Reviewer pass requested changes; fix fork qovv55cfc1ao addressed both blockers; Fix fork qovv55cfc1ao reported: castor test --filter TickPollListener OK (24 tests), castor test:llm-real --filter SubagentRetrieveLiveE2eTest OK, castor test:llm-real --filter SubagentParallelLiveE2eTest OK, castor cs-check OK
- Summary: Implementation and reviewer fixes complete. Branch contains feature commit 161c7c1db (`feat(tui): add readonly subagent live view foundation`) plus fix commit 4c8a4d8fa (`test(tui): account for multiplexed subagent live events`). Reviewer blockers were fixed: TickPollListener reflection tests initialize new live poller dependency; controller E2E collectors demultiplex forwarded child run terminal events. Worktree verified clean before transition.

## Task workflow update - 2026-07-02T21:56:56.938Z
- Summary: User manually tested PR #251 with a live subagent run. Result: live subagent flow appears to work; subagent scout completed and returned artifact `agent_95bd52211a275551`; `/agents-live`/readonly live-view behavior appeared functional. User-observed follow-up UX gaps: subagent block has no rich styling/rendering and Ctrl+O expand support is absent for that block.

## Task workflow update - 2026-07-02T22:02:51.848Z
- Moved CODE-REVIEW → DONE.
- Merged task/subagent-live-01-foundation-catalog-readonly-view into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                               |  15 ++
 .../Agent/Execution/SubagentExecutionService.php   |   2 +
 .../Execution/SubagentProgressSnapshotBuilder.php  |   5 +
 .../Runtime/Controller/RuntimeEventEmitter.php     |  60 +++++++
 .../Process/JsonlProcessAgentSessionClient.php     |  13 +-
 src/Tui/Listener/AgentsLiveCommandHandler.php      |  26 +++
 src/Tui/Listener/AgentsMainCommandHandler.php      |  37 ++++
 src/Tui/Listener/SubagentLiveCommandRegistrar.php  |  55 ++++++
 src/Tui/Listener/TickPollListener.php              |  72 +++++++-
 src/Tui/Picker/SubagentLivePickerController.php    | 194 +++++++++++++++++++++
 src/Tui/Runtime/SubagentLiveCatalog.php            | 109 ++++++++++++
 src/Tui/Runtime/SubagentLiveChildDTO.php           |  41 +++++
 src/Tui/Runtime/SubagentLiveChildViewPoller.php    |  96 ++++++++++
 src/Tui/Runtime/SubagentLiveStatusEnum.php         |  42 +++++
 src/Tui/Runtime/SubagentLiveViewState.php          | 114 ++++++++++++
 src/Tui/Runtime/TuiRuntimeEventApplier.php         |   1 +
 src/Tui/Runtime/TuiSessionState.php                |   6 +
 .../Controller/E2E/ControllerE2eTestCase.php       |  62 ++++++-
 .../Runtime/Controller/RuntimeEventEmitterTest.php |  51 ++++++
 ...onlProcessAgentSessionClientEventBufferTest.php |  68 ++++++++
 tests/Tui/E2E/TuiSubagentLiveViewE2eTest.php       | 128 ++++++++++++++
 .../Listener/TickPollListenerSubagentLiveTest.php  | 134 ++++++++++++++
 tests/Tui/Listener/TickPollListenerTest.php        |  14 ++
 tests/Tui/Runtime/SubagentLiveCatalogTest.php      |  86 +++++++++
 .../Tui/Support/SubagentProgressEventsFixture.php  |   2 +
 25 files changed, 1426 insertions(+), 7 deletions(-)
 create mode 100644 src/Tui/Listener/AgentsLiveCommandHandler.php
 create mode 100644 src/Tui/Listener/AgentsMainCommandHandler.php
 create mode 100644 src/Tui/Listener/SubagentLiveCommandRegistrar.php
 create mode 100644 src/Tui/Picker/SubagentLivePickerController.php
 create mode 100644 src/Tui/Runtime/SubagentLiveCatalog.php
 create mode 100644 src/Tui/Runtime/SubagentLiveChildDTO.php
 create mode 100644 src/Tui/Runtime/SubagentLiveChildViewPoller.php
 create mode 100644 src/Tui/Runtime/SubagentLiveStatusEnum.php
 create mode 100644 src/Tui/Runtime/SubagentLiveViewState.php
 create mode 100644 tests/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClientEventBufferTest.php
 create mode 100644 tests/Tui/E2E/TuiSubagentLiveViewE2eTest.php
 create mode 100644 tests/Tui/Listener/TickPollListenerSubagentLiveTest.php
 create mode 100644 tests/Tui/Runtime/SubagentLiveCatalogTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/subagent-live-01-foundation-catalog-readonly-view.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/subagent-live-01-foundation-catalog-readonly-view.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Manual live test by user: subagent scout completed; artifact agent_95bd52211a275551; live view flow appeared to work; CODE-REVIEW transition validation: deterministic castor check passed in 87.0s; PR #251 merged externally by user
- Summary: User manually tested the live TUI behavior and reported it looks decent; PR #251 was merged externally. Marking tracked task done. Implementation delivered Phase 1 readonly subagent live view with `/agents-live`, `/agents-main`, child event forwarding, JSONL demux/rebuffering, parent-always polling, TmuxHarness E2E, reviewer fixes, and deterministic castor check passed during CODE-REVIEW transition.
