# SESSION-07B Split Core reducers from App replay/rewind orchestration

## Goal
Parent: session-07-arch-relocate-turn-tree-and-replay-out-of-agentcore.

Purpose: second architecture slice after SESSION-07A. Keep Core responsible for pure event reducers, while moving branch-aware run-state/hot-prompt replay and rewind orchestration into the CodingAgent/App session layer.

Approved architecture decisions:
- Core keeps pure event→`RunState` and event→`PromptState` reducers/helpers.
- App owns branch-aware orchestration: active leaf selection, branch filtering, rewind-to-turn rebuild, and branch-aware hot prompt rebuild.
- Pure prompt-state reducer can stay in Core; branch-aware hot prompt rebuild moves.
- No compatibility shims / dual paths during active development.

Scope:
- Split `RunStateReplayService`: extract/keep pure event-list-to-`RunState` reducer in Core; move `rebuildIfStale()`, `rebuildForLeaf()`, branch filtering, and active-leaf orchestration into `src/CodingAgent/Session/Replay/` or equivalent.
- Split `ReplayService`: keep pure prompt-state/message replay reducer in Core; move branch-aware hot prompt rebuild orchestration into App session replay.
- Move `RunRewindService` into App session rewind orchestration, since rewind-to-turn is a session/runtime operation.
- Introduce narrow Core contracts as needed so Core pipeline (`RunMessageProcessor`, `RunCommit`) does not depend on App concrete classes. Do not create AgentCore→CodingAgent dependency.
- Extract shared replay preparation/integrity code (`prepareForReplay()` or equivalent) to remove duplicate logic between stale rebuild and leaf rebuild.
- Move/update affected tests from `tests/AgentCore/Application/Handler/...` to appropriate `tests/CodingAgent/Session/Replay/...` where orchestration moves, while retaining Core reducer tests for Core-owned logic.

Dependencies:
- Start after SESSION-07A lands or consciously rebase on it.

Out of scope:
- TUI transcript provider contract.
- Single `RunLeafChanged` factory unless needed as a local seam; final closure is SESSION-07C.

## Acceptance criteria
- Core contains only pure replay reducers/helpers and contracts; branch-aware replay/rewind orchestration lives under CodingAgent/App session layer.
- `RunMessageProcessor` and `RunCommit` depend on Core contracts/interfaces, not App concrete classes, and no `AgentCore -> CodingAgent` dependency is introduced.
- `RunRewindService` or equivalent rewind-to-turn orchestration no longer lives under `src/AgentCore/`.
- Branch-aware hot prompt rebuild no longer lives as Core-specific orchestration; pure prompt-state/message replay remains Core-owned if needed.
- Duplicate replay preparation/integrity logic between stale rebuild and leaf rebuild is extracted/shared.
- Moved/split tests prove behavior preservation for `RunStateReplayService`, `ReplayService`, and rewind replay scenarios.
- Focused validation run via Castor: `castor test --filter="RunStateReplayService|ReplayService|RunRewindService|RewindBranchLiveE2e"`, `castor deptrac`, `castor phpstan`, `castor cs-check`.

## Workflow metadata
Status: DONE
Branch: task/session-07b-split-core-reducers-from-app-replay-rewind-orchestration
Worktree: /home/ineersa/projects/agent-core-worktrees/session-07b-split-core-reducers-from-app-replay-rewind-orchestration
Fork run: ibh8x6efyzjz
PR URL: https://github.com/ineersa/agent-core/pull/263
PR Status: merged
Started: 2026-07-06T14:16:32.646Z
Completed: 2026-07-06T16:13:03.646Z

## Work log
- Created: 2026-07-06T01:30:04.236Z

## Task workflow update - 2026-07-06T14:16:32.646Z
- Moved TODO → IN-PROGRESS.
- Created branch task/session-07b-split-core-reducers-from-app-replay-rewind-orchestration.
- Created worktree /home/ineersa/projects/agent-core-worktrees/session-07b-split-core-reducers-from-app-replay-rewind-orchestration.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/session-07b-split-core-reducers-from-app-replay-rewind-orchestration.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/session-07b-split-core-reducers-from-app-replay-rewind-orchestration.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/session-07b-split-core-reducers-from-app-replay-rewind-orchestration.
- Summary: Starting implementation phase for SESSION-07B. Goal: split Core pure replay reducers from App session branch-aware replay/rewind orchestration after SESSION-07A landed. Parent loaded task-workflow and testing instructions plus tests/AGENTS.md before starting.

## Task workflow update - 2026-07-06T14:21:46.693Z
- Summary: Pre-implementation scouting completed. Parent loaded task-workflow, testing, and tests/AGENTS.md. Scout context saved at `/home/ineersa/.pi/agent/tmp/2026-07--b7b37f73.txt`.

Key context: current SESSION-07A baseline has `RunStateReplayService`, `ReplayService`, and `RunRewindService` still under `src/AgentCore/Application/Handler/`. `RunStateReplayService::replay()` is the pure RunState reducer; `rebuildIfStale()` and `rebuildForLeaf()` are branch-aware orchestration using event store + branch filter + logging/integrity. `ReplayService::replayMessages()`/token estimate are pure prompt replay helpers, while `rebuildHotPromptState()` is branch-aware store orchestration. `RunRewindService` is pure orchestration (lock/read/project/append LeafSet/rebuild/CAS). Current Core pipeline call sites: `RunMessageProcessor` uses `RunStateReplayService::rebuildIfStale()`, `RunCommit` uses `ReplayService::rebuildHotPromptState()`, and runtime rewind handlers use `RunRewindService`.

Implementation direction for fork: keep Core to pure reducers/helpers plus narrow contracts; move branch-aware run-state rebuild, hot prompt rebuild, and rewind-to-turn orchestration to `src/CodingAgent/Session/Replay` / `src/CodingAgent/Session/Rewind`; inject those back into Core pipeline through Core contracts/interfaces, not App concrete classes. Do not move turn-tree classes back into Core; SESSION-07A deliberately put them in App session layer. Extract shared canonical replay preparation/integrity helper to remove duplicated stale-vs-leaf checks. Update service aliases and move/split tests accordingly.

## Task workflow update - 2026-07-06T15:38:56.368Z
- Recorded fork run: 7cwta56axkqn
- Summary: Implementation fork 7cwta56axkqn returned with architecture split mostly implemented but not committed. Worktree has uncommitted substantial changes: Core contracts/reducers added, App session replay/rewind orchestration added, old Core handler services deleted, and wiring/tests partially updated. Fork reported `RunStateReplayServiceTest` focused 46/46 OK and no `use Ineersa\CodingAgent` imports under `src/AgentCore`, but full focused validation/deptrac/phpstan/cs-check were not run and changes need fixes + commit.

## Task workflow update - 2026-07-06T15:43:22.238Z
- Recorded fork run: azeib9s2lzk3
- Validation: castor test --filter="RunStateReplayService\|ReplayService\|RunRewindService\|RewindBranchLiveE2e\|InProcessRewindEmitsRunLeafChanged\|TurnTreeReplayFilter\|SessionTurnTreeProvider\|RunCommitLogging\|CommandMailboxPolicy" — OK (73 tests, 368 assertions); castor deptrac — OK (0 violations); castor phpstan — OK; castor cs-check — OK after castor cs-fix (10 files); castor test:llm-real --filter=RewindBranchLiveE2e — OK (1 test, 11 assertions; llama-proxy health 200, generation preflight OK); Parent verification: git status clean; rg -n -F 'use Ineersa\\CodingAgent' src/AgentCore returned no matches; old Core handler orchestration files deleted
- Summary: Implementation continuation fork azeib9s2lzk3 completed SESSION-07B and committed `98d32d18a` (`Split replay orchestration into app session layer`). Core now keeps pure replay components (`RunStateReducer`, `PromptStateReplayService`, `ReplayEventPreparer`) plus narrow contracts (`RunStateRebuilderInterface`, `HotPromptStateRebuilderInterface`, `HotPromptIntegrityVerifierInterface`, `RunRewindServiceInterface`); branch-aware run-state replay, hot prompt rebuild, and rewind orchestration now live in `src/CodingAgent/Session/Replay/` and `src/CodingAgent/Session/Rewind/`. Old Core handler orchestration files `ReplayService.php`, `RunStateReplayService.php`, and `RunRewindService.php` are deleted. Parent verified worktree is clean, no `use Ineersa\CodingAgent` imports exist under `src/AgentCore`, and old Core handler files are absent.

Known non-blocking gaps from fork: behavior tests for Session* services still live under `tests/AgentCore/Application/Handler/` paths; `docs/session-storage.md` may still mention old replay service names. SESSION-07C remains responsible for single `RunLeafChanged` producer and TUI transcript-block contract.

## Task workflow update - 2026-07-06T15:46:56.035Z
- Validation: castor check — OK (266.0s). QA run qa-20260706-154509-94130-c611799d: deptrac OK, test OK (4147 tests, 13467 assertions), test:controller-replay OK (8 tests, 112 assertions), test:tui OK (31 tests, 159 assertions), test:llm-real OK (10 tests, 121 assertions), phpstan OK, cs-check OK, llama-proxy cache guard OK (103→103), artifact integrity OK, leak check OK.

## Task workflow update - 2026-07-06T15:59:15.495Z
- Validation: Reviewer verdict: APPROVED WITH SUGGESTIONS; no blocking issues.
- Summary: Reviewer pass after full castor check returned APPROVED WITH SUGGESTIONS. No blockers. Reviewer confirmed architecture split is correct, behaviorally faithful, old Core orchestration files deleted, no AgentCore→CodingAgent dependency, and all consumers/DI wiring updated. Non-blocking suggestions: (1) move/rename `tests/AgentCore/Application/Handler/ReplayServiceTest.php` and `RunStateReplayServiceTest.php` to `tests/CodingAgent/Session/Replay/` later because they now target App session services; (2) in `RunStateReplayException.php`, change short `{@see SessionRunStateReplayService}` to Core contract `RunStateRebuilderInterface`; (3) import `RunRewindServiceInterface` in `SessionRewindService` instead of inline FQCN; (4) minor services.yaml blank line cleanup; (5) optional unit test for `ReplayEventPreparer`; (6) optional comment that `RunDebugService` intentionally depends on two narrow interfaces implemented by same service. Reviewer noted `ReplayEventPreparer::duplicateSequences()` now reports unique duplicate seqs with O(1) lookup, a strict improvement but small behavior change beyond pure move.

## Task workflow update - 2026-07-06T16:03:00.794Z
- Recorded fork run: ibh8x6efyzjz
- Validation: castor test --filter="SessionHotPromptReplayService\|SessionRunStateReplayService\|ReplayEventPreparer\|RunDebugService\|RunCommitLogging\|CommandMailboxPolicy" — OK (66 tests, 304 assertions); castor deptrac — OK (0 violations); castor phpstan — OK; castor cs-check — OK after castor cs-fix; rg -n -F 'use Ineersa\\CodingAgent' src/AgentCore — no matches
- Summary: Reviewer cleanup fork ibh8x6efyzjz completed and committed `b034b7ff0` (`Apply replay orchestration review cleanups`). Applied reviewer suggestions: moved/renamed App-session replay tests to `tests/CodingAgent/Session/Replay/SessionHotPromptReplayServiceTest.php` and `SessionRunStateReplayServiceTest.php`; updated `RunStateReplayException` docblock to reference Core `RunStateRebuilderInterface`; imported `RunRewindServiceInterface` in `SessionRewindService`; removed extra services.yaml blank line; added explanatory `RunDebugService` comment for the two narrow hot-prompt contracts; added focused `ReplayEventPreparerTest` for duplicate/missing sequence behavior. No production behavior change intended. Worktree clean per fork handoff.

## Task workflow update - 2026-07-06T16:04:50.631Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (92.9s).
- Pushed task/session-07b-split-core-reducers-from-app-replay-rewind-orchestration to origin.
- branch 'task/session-07b-split-core-reducers-from-app-replay-rewind-orchestration' set up to track 'origin/task/session-07b-split-core-reducers-from-app-replay-rewind-orchestration'.
- Created PR: https://github.com/ineersa/agent-core/pull/263
- Validation: Pre-cleanup full castor check — OK (266.0s), QA run qa-20260706-154509-94130-c611799d: test OK (4147 tests, 13467 assertions), controller replay OK, TUI OK, llm-real OK, deptrac/phpstan/cs-check OK, cache guard/leak check OK; Reviewer verdict — APPROVED WITH SUGGESTIONS; no blockers; Cleanup focused validation — castor test filter OK (66 tests, 304 assertions), deptrac OK, phpstan OK, cs-check OK after cs-fix; Architecture verification — rg -n -F 'use Ineersa\\CodingAgent' src/AgentCore returned no matches; old Core handler orchestration files deleted
- Summary: SESSION-07B ready for code review at HEAD `b034b7ff0`. Core now contains pure replay reducers/helpers/contracts only (`RunStateReducer`, `PromptStateReplayService`, `ReplayEventPreparer`, and narrow replay/rewind contracts). Branch-aware run-state replay, hot prompt rebuild, and rewind orchestration live in `src/CodingAgent/Session/Replay/` and `src/CodingAgent/Session/Rewind/`. Old Core orchestration handlers (`ReplayService.php`, `RunStateReplayService.php`, `RunRewindService.php`) are deleted. Reviewer returned APPROVED WITH SUGGESTIONS; cleanup commit applied the suggestions (test moves/renames, doc/import/style fixes, focused `ReplayEventPreparerTest`). Parent verified worktree clean and no AgentCore→CodingAgent imports.

## Task workflow update - 2026-07-06T16:13:03.646Z
- Moved CODE-REVIEW → DONE.
- Merged task/session-07b-split-core-reducers-from-app-replay-rewind-orchestration into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                               |  14 +-
 src/AgentCore/Application/AGENTS.md                |   8 +-
 .../Application/Handler/ReplayService.php          | 237 ----------
 .../Application/Handler/RunDebugService.php        |  13 +-
 .../Handler/RunStateReplayException.php            |   4 +-
 src/AgentCore/Application/Pipeline/RunCommit.php   |   6 +-
 .../Application/Pipeline/RunMessageProcessor.php   |   8 +-
 .../Replay/PromptStateReplayService.php            |  91 ++++
 .../Application/Replay/ReplayEventPreparer.php     |  84 ++++
 .../RunStateReducer.php}                           | 481 +--------------------
 .../Replay/HotPromptIntegrityVerifierInterface.php |  12 +
 .../Replay/HotPromptStateRebuilderInterface.php    |  12 +
 .../Contract/Replay/RunStateRebuilderInterface.php |  15 +
 .../Contract/Rewind/RunRewindServiceInterface.php  |  13 +
 .../CommandHandler/RewindToTurnHandler.php         |   6 +-
 .../InProcess/InProcessAgentSessionClient.php      |   4 +-
 .../Replay/SessionHotPromptReplayService.php       | 115 +++++
 .../Replay/SessionRunStateReplayService.php        | 332 ++++++++++++++
 .../Session/Rewind/SessionRewindService.php}       |  11 +-
 .../Pipeline/CommandMailboxPolicyTest.php          |   8 +-
 .../Application/Pipeline/RunCommitLoggingTest.php  |  10 +-
 .../Application/Replay/ReplayEventPreparerTest.php |  51 +++
 .../Replay/SessionHotPromptReplayServiceTest.php}  |  31 +-
 .../Replay/SessionRunStateReplayServiceTest.php}   |  16 +-
 24 files changed, 816 insertions(+), 766 deletions(-)
 delete mode 100644 src/AgentCore/Application/Handler/ReplayService.php
 create mode 100644 src/AgentCore/Application/Replay/PromptStateReplayService.php
 create mode 100644 src/AgentCore/Application/Replay/ReplayEventPreparer.php
 rename src/AgentCore/Application/{Handler/RunStateReplayService.php => Replay/RunStateReducer.php} (61%)
 create mode 100644 src/AgentCore/Contract/Replay/HotPromptIntegrityVerifierInterface.php
 create mode 100644 src/AgentCore/Contract/Replay/HotPromptStateRebuilderInterface.php
 create mode 100644 src/AgentCore/Contract/Replay/RunStateRebuilderInterface.php
 create mode 100644 src/AgentCore/Contract/Rewind/RunRewindServiceInterface.php
 create mode 100644 src/CodingAgent/Session/Replay/SessionHotPromptReplayService.php
 create mode 100644 src/CodingAgent/Session/Replay/SessionRunStateReplayService.php
 rename src/{AgentCore/Application/Handler/RunRewindService.php => CodingAgent/Session/Rewind/SessionRewindService.php} (91%)
 create mode 100644 tests/AgentCore/Application/Replay/ReplayEventPreparerTest.php
 rename tests/{AgentCore/Application/Handler/ReplayServiceTest.php => CodingAgent/Session/Replay/SessionHotPromptReplayServiceTest.php} (93%)
 rename tests/{AgentCore/Application/Handler/RunStateReplayServiceTest.php => CodingAgent/Session/Replay/SessionRunStateReplayServiceTest.php} (99%)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/session-07b-split-core-reducers-from-app-replay-rewind-orchestration.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/session-07b-split-core-reducers-from-app-replay-rewind-orchestration.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #263 merged by user before DONE transition.; CODE-REVIEW gate previously passed deterministic castor check (92.9s).
- Summary: User reported PR #263 was merged. Completing SESSION-07B task: Core replay orchestration split finished and merged; branch-aware replay/rewind orchestration now lives in CodingAgent session layer while AgentCore retains pure reducers/helpers/contracts.
