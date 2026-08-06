# SESSION-07A Move turn tree projector and branch filter to App session layer

## Goal
Parent: session-07-arch-relocate-turn-tree-and-replay-out-of-agentcore.

Purpose: first behavior-preserving slice of the AgentCore turn-tree relocation. Move the pure turn-tree projection/model and branch replay filter out of `src/AgentCore/` into the CodingAgent/App session layer. This should be mostly namespace/import/depfile/test relocation, with no behavior change.

Approved architecture decisions:
- Core keeps canonical run/event facts (`RunEvent`, `RunEventTypeEnum`, `RunState`, event store contracts).
- App owns branch/leaf/session interpretation.
- Preferred target namespace: `src/CodingAgent/Session/TurnTree/` and `src/CodingAgent/Session/Replay/` rather than adding more under `src/CodingAgent/Runtime/Session/`.
- No backward-compat shims or dual-location classes.

Scope:
- Move `TurnTreeProjector`, `TurnTreeDTO`, `TurnTreeNodeDTO` from `src/AgentCore/Domain/Run/` to `src/CodingAgent/Session/TurnTree/`.
- Move `TurnTreeReplayFilter` and `TurnBranchReplayDTO` from `src/AgentCore/Application/...` to `src/CodingAgent/Session/Replay/`.
- Update `SessionTurnTreeProvider` and all consumers/imports.
- Move corresponding tests out of `tests/AgentCore/...` into `tests/CodingAgent/Session/...`.
- Update `depfile.yaml` rules only as needed; do not weaken boundaries broadly.

Out of scope:
- Splitting `RunStateReplayService` / `ReplayService`.
- TUI transcript contract changes.
- RunLeafChanged factory.
- Any product behavior change.

## Acceptance criteria
- No `TurnTreeProjector`, `TurnTreeDTO`, `TurnTreeNodeDTO`, `TurnTreeReplayFilter`, or `TurnBranchReplayDTO` remain under `src/AgentCore/`.
- AgentCore has no dependency on CodingAgent; `castor deptrac` is green.
- Behavior-preserving tests moved and passing: `TurnTreeProjectorTest`, `TurnTreeReplayFilterTest`, `SessionTurnTreeProviderTest`.
- No backward-compat alias/shim classes are introduced for old namespaces.
- Focused validation run via Castor: `castor test --filter="TurnTreeProjector|TurnTreeReplayFilter|SessionTurnTreeProvider"`, `castor deptrac`, `castor phpstan`, `castor cs-check`.

## Workflow metadata
Status: DONE
Branch: task/session-07a-move-turn-tree-projector-and-filter-to-app-session
Worktree: /home/ineersa/projects/agent-core-worktrees/session-07a-move-turn-tree-projector-and-filter-to-app-session
Fork run: 3mxoosjyxhyh
PR URL: https://github.com/ineersa/agent-core/pull/262
PR Status: merged
Started: 2026-07-06T01:33:26.741Z
Completed: 2026-07-06T14:15:57.986Z

## Work log
- Created: 2026-07-06T01:29:47.402Z

## Task workflow update - 2026-07-06T01:33:26.741Z
- Moved TODO → IN-PROGRESS.
- Created branch task/session-07a-move-turn-tree-projector-and-filter-to-app-session.
- Created worktree /home/ineersa/projects/agent-core-worktrees/session-07a-move-turn-tree-projector-and-filter-to-app-session.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/session-07a-move-turn-tree-projector-and-filter-to-app-session.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/session-07a-move-turn-tree-projector-and-filter-to-app-session.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/session-07a-move-turn-tree-projector-and-filter-to-app-session.
- Summary: Starting implementation phase for 07A. User confirmed split: Core keeps pure event reducers; App owns branch-aware/session interpretation. This slice is behavior-preserving relocation of turn-tree projector/DTOs and branch filter into CodingAgent/App session layer.

## Task workflow update - 2026-07-06T01:36:41.692Z
- Validation: Parent read testing skill and tests/AGENTS.md before preparing test strategy.; Scout context saved in /home/ineersa/.pi/agent/tmp/2026-07--11196ba6.txt.
- Summary: Pre-implementation scouting completed. Parent loaded `task-workflow`, `testing`, `subagents`, and read `tests/AGENTS.md`. Key implementation caveat: a naive move of `TurnTreeProjector`/`TurnTreeReplayFilter` into `src/CodingAgent/Session/...` and then updating AgentCore handler imports would create an `AgentCore -> CodingAgent/AppSession` dependency, violating the task acceptance and root architecture rule. Implementation must NOT add `AppSession` to AgentCore's deptrac ruleset. It must preserve `AgentCore` independence, likely by introducing a narrow Core contract seam for the existing branch-filter collaborator as an interim bridge, or by finding an equally small behavior-preserving alternative that avoids Core importing CodingAgent. Scope remains 07A: relocate projector/DTO/filter classes and tests; do not perform the larger 07B replay-service split unless unavoidable and explicitly justified in the fork handoff.

## Task workflow update - 2026-07-06T01:43:02.781Z
- Recorded fork run: ozgvwewfb0ym
- Validation: Fork preflight read testing skill and tests/AGENTS.md.; Fork validation on commit 88333ecee: `castor test --filter="TurnTreeProjector|TurnTreeReplayFilter|SessionTurnTreeProvider|ReplayService|RunStateReplayService"` OK (83 tests, 435 assertions).; Fork validation: `castor deptrac` OK (0 violations).; Fork validation: `castor phpstan` OK.; Fork validation: `castor cs-check` initially found 15 fixable issues; fork ran `castor cs-fix`; final `castor cs-check` OK.; Parent cherry-picked implementation onto task branch as fd3c73241 and verified worktree clean.
- Summary: Implementation fork ozgvwewfb0ym completed. Fork accidentally committed on integration checkout `main` as 88333ecee instead of the task worktree. Parent preserved the implementation by cherry-picking that commit into the task worktree branch, producing task-branch commit `fd3c73241` (`Move turn tree projection to app session layer`). Worktree `/home/ineersa/projects/agent-core-worktrees/session-07a-move-turn-tree-projector-and-filter-to-app-session` is clean at `fd3c73241`.

Implemented shape: moved turn-tree projector/DTOs and branch replay filter to `src/CodingAgent/Session/{TurnTree,Replay}/`; removed old AgentCore locations; added Core-owned turn-tree contracts/snapshot DTOs under `src/AgentCore/Contract/TurnTree/`; App session provides contract adapters so existing Core replay/rewind services depend only on AgentCore contracts and do not import CodingAgent. `config/services.yaml` aliases the Core contracts to App session adapters. Tests moved/updated under `tests/CodingAgent/Session/{TurnTree,Replay}/`, and handler/provider tests updated.

Parent verification after cherry-pick: `git diff --stat HEAD~1..HEAD` shows expected 25 files changed (+278/-46); `git status` in task worktree is clean; `rg -F 'use Ineersa\\CodingAgent' src/AgentCore` reports no imports from CodingAgent in AgentCore; old named files no longer exist under `src/AgentCore/`.

Outstanding cleanup: accidental commit `88333ecee` still sits on local integration checkout `main` in `/home/ineersa/projects/agent-core`. It is preserved on the task branch as `fd3c73241`; integration main should be reset back to `af69d4711` only with explicit user approval because destructive reset is forbidden without approval.

## Task workflow update - 2026-07-06T02:00:40.696Z
- Summary: User approved cleanup of the accidental duplicate implementation commit on the local integration checkout. Parent ran `git reset --hard af69d4711` in `/home/ineersa/projects/agent-core` only after approval. Integration `main` is now back at `af69d4711` and remains ahead of origin/main by pre-existing merge commits; the implementation remains preserved on the task worktree branch at `fd3c73241`. Task worktree branch was not reset.

## Task workflow update - 2026-07-06T02:22:22.255Z
- Recorded fork run: si43r0jlvhk9
- Validation: Fork preflight read testing skill and tests/AGENTS.md.; Fork validation: `castor test --filter="TurnTreeProjector|TurnTreeReplayFilter|SessionTurnTreeProvider|ReplayService|RunStateReplayService|InProcessRewindEmitsRunLeafChanged"` OK (84 tests, 440 assertions).; Fork validation: `castor deptrac` OK (0 violations).; Fork validation: `castor phpstan` OK.; Fork validation: `castor cs-check` initially found 1 fixable issue; fork ran `castor cs-fix`; final `castor cs-check` OK, but parent found that cs-fix change was not included in commit da66790cc and is being committed by follow-up fork 3mxoosjyxhyh.
- Summary: Reviewer follow-up fork si43r0jlvhk9 implemented contract-narrowing suggestions with commit `da66790cc` (`Narrow turn tree core contracts`). Changes: narrowed `TurnTreeNodeSnapshotDTO` to `turnNo` + `parentTurnNo`; narrowed `TurnTreeSnapshotDTO` to `runId`, `nodesByTurnNo`, `currentLeafTurnNo`; removed unused `canonicalLastSeq` from `BranchReplayResultDTO`; updated App session adapters; added a concise docs/session-storage.md note about SESSION-07A relocation/contract boundary. Fork confirmed no `use Ineersa\CodingAgent` imports under `src/AgentCore` and deptrac stayed green.

Parent post-handoff inspection found one uncommitted cs-fix docblock alignment change left in `src/AgentCore/Contract/TurnTree/TurnTreeSnapshotDTO.php` after commit `da66790cc`; tiny cleanup fork `3mxoosjyxhyh` launched to commit that single formatting change and re-run `castor cs-check`.

## Task workflow update - 2026-07-06T02:22:39.802Z
- Recorded fork run: 3mxoosjyxhyh
- Validation: Cleanup fork preflight read testing skill and tests/AGENTS.md.; Cleanup fork validation: `castor cs-check` PASS (0 of 1288 files fixable).; Cleanup fork final status: clean worktree at `fb418f50e`.
- Summary: Tiny cleanup fork 3mxoosjyxhyh completed with commit `fb418f50e` (`Apply turn tree contract cs fix`), committing the single leftover cs-fix docblock alignment in `src/AgentCore/Contract/TurnTree/TurnTreeSnapshotDTO.php`. Worktree is clean at HEAD `fb418f50e`.

## Task workflow update - 2026-07-06T02:41:55.532Z
- Validation: Reviewer subagent at HEAD `fb418f50e`: APPROVED.; Focused local validation after extension autoload setup: `castor test` PASS (4151 tests, 13479 assertions).; Focused local validation: `castor deptrac` PASS (0 violations, 0 errors).; Focused local validation: `castor phpstan` PASS (errors=0, file_errors=0).; Focused local validation: `castor cs-check` PASS (files_fixed=0).; Environment setup note: first `castor test` failed before task code due missing/stale ignored Composer autoload artifacts for project-level file-rewind extension; fixed by `composer install` under `.hatfield/extensions` and `composer dump-autoload` at repo root; worktree remained clean.
- Summary: Reviewer re-review at HEAD `fb418f50e` returned APPROVED. Reviewer verified: no `AgentCore -> CodingAgent` production dependency (`use Ineersa\CodingAgent` absent under `src/AgentCore`), no backward-compat aliases, Core contracts are now narrow, adapters correctly map wide session DTOs to narrow Core contracts, service wiring is correct, behavior-preserving relocation remains scoped.

Local validation initially failed because the worktree lacked ignored Composer autoload artifacts for project-level extensions (`.hatfield/extensions/vendor` and stale root `vendor/autoload.php`). Parent installed `.hatfield/extensions` dependencies with Composer and refreshed root Composer autoload; these are ignored environment artifacts, and `git status` remained clean. After setup, focused validation passed.

PR-scope blocker discovered before CODE-REVIEW move: task branch is based on local integration `main` at `af69d4711`, but `origin/main` is still `4ef0880af`. `git diff main...HEAD` is scoped to SESSION-07A (26 files), while `git diff origin/main...HEAD` includes unrelated prior file-rewind changes (37 files total). Do not move to CODE-REVIEW until user chooses whether to sync/push local main or rebase/cherry-pick SESSION-07A onto `origin/main` to keep the GitHub PR scoped.

## Task workflow update - 2026-07-06T02:48:41.302Z
- Summary: User chose to sync/push local integration `main` before CODE-REVIEW so the GitHub PR base is scoped correctly. Parent pushed `/home/ineersa/projects/agent-core` `main` from `4ef0880af` to `af69d4711`; `origin/main` now matches local `main`. This removes the previous PR-scope blocker where `origin/main...HEAD` included unrelated prior file-rewind changes.

## Task workflow update - 2026-07-06T02:50:32.378Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (96.9s).
- Pushed task/session-07a-move-turn-tree-projector-and-filter-to-app-session to origin.
- branch 'task/session-07a-move-turn-tree-projector-and-filter-to-app-session' set up to track 'origin/task/session-07a-move-turn-tree-projector-and-filter-to-app-session'.
- Created PR: https://github.com/ineersa/agent-core/pull/262
- Validation: Reviewer subagent at HEAD `fb418f50e`: APPROVED.; Focused validation: `castor test` PASS (4151 tests, 13479 assertions).; Focused validation: `castor deptrac` PASS (0 violations, 0 errors).; Focused validation: `castor phpstan` PASS (errors=0, file_errors=0).; Focused validation: `castor cs-check` PASS (files_fixed=0).; Before CODE-REVIEW move: worktree clean; `git diff --stat origin/main...HEAD` scoped to 26 SESSION-07A files.
- Summary: Prepared for CODE-REVIEW at HEAD `fb418f50e`. Reviewer re-review returned APPROVED. PR base scope blocker was resolved by user-approved push of local integration `main` to `origin/main` (`af69d4711`), so `origin/main...HEAD` now shows only the SESSION-07A relocation/narrow-contract diff (26 files).

## Task workflow update - 2026-07-06T14:15:57.986Z
- Moved CODE-REVIEW → DONE.
- Merged task/session-07a-move-turn-tree-projector-and-filter-to-app-session into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                               |  6 ++++
 docs/session-storage.md                            |  5 +++
 src/AgentCore/Application/AGENTS.md                | 20 +++++------
 .../Application/Handler/ReplayService.php          |  4 +--
 .../Application/Handler/RunRewindService.php       |  6 ++--
 .../Application/Handler/RunStateReplayService.php  |  4 +--
 .../TurnTree/BranchReplayFilterInterface.php       | 26 +++++++++++++++
 .../Contract/TurnTree/BranchReplayResultDTO.php    | 31 +++++++++++++++++
 .../Contract/TurnTree/TurnTreeNodeSnapshotDTO.php  | 19 +++++++++++
 .../TurnTree/TurnTreeProjectorInterface.php        | 21 ++++++++++++
 .../Contract/TurnTree/TurnTreeSnapshotDTO.php      | 24 +++++++++++++
 .../Runtime/Protocol/TurnTreeNodeView.php          |  2 +-
 src/CodingAgent/Runtime/Protocol/TurnTreeView.php  |  2 +-
 .../Replay/BranchReplayFilterContractAdapter.php   | 39 ++++++++++++++++++++++
 .../Session/Replay}/TurnBranchReplayDTO.php        |  8 ++---
 .../Session}/Replay/TurnTreeReplayFilter.php       |  5 ++-
 .../Session/SessionTurnTreeProvider.php            |  4 +--
 .../Session/TurnTree}/TurnTreeDTO.php              |  2 +-
 .../Session/TurnTree}/TurnTreeNodeDTO.php          |  2 +-
 .../Session/TurnTree}/TurnTreeProjector.php        |  2 +-
 .../TurnTree/TurnTreeProjectorContractAdapter.php  | 39 ++++++++++++++++++++++
 .../Application/Handler/ReplayServiceTest.php      |  7 ++--
 .../Handler/RunStateReplayServiceTest.php          |  9 ++---
 .../Session}/Replay/TurnTreeReplayFilterTest.php   |  8 +++--
 .../Session/SessionTurnTreeProviderTest.php        |  4 +--
 .../Session/TurnTree}/TurnTreeProjectorTest.php    |  4 +--
 26 files changed, 256 insertions(+), 47 deletions(-)
 create mode 100644 src/AgentCore/Contract/TurnTree/BranchReplayFilterInterface.php
 create mode 100644 src/AgentCore/Contract/TurnTree/BranchReplayResultDTO.php
 create mode 100644 src/AgentCore/Contract/TurnTree/TurnTreeNodeSnapshotDTO.php
 create mode 100644 src/AgentCore/Contract/TurnTree/TurnTreeProjectorInterface.php
 create mode 100644 src/AgentCore/Contract/TurnTree/TurnTreeSnapshotDTO.php
 create mode 100644 src/CodingAgent/Session/Replay/BranchReplayFilterContractAdapter.php
 rename src/{AgentCore/Application/Dto => CodingAgent/Session/Replay}/TurnBranchReplayDTO.php (73%)
 rename src/{AgentCore/Application => CodingAgent/Session}/Replay/TurnTreeReplayFilter.php (97%)
 rename src/{AgentCore/Domain/Run => CodingAgent/Session/TurnTree}/TurnTreeDTO.php (94%)
 rename src/{AgentCore/Domain/Run => CodingAgent/Session/TurnTree}/TurnTreeNodeDTO.php (97%)
 rename src/{AgentCore/Domain/Run => CodingAgent/Session/TurnTree}/TurnTreeProjector.php (99%)
 create mode 100644 src/CodingAgent/Session/TurnTree/TurnTreeProjectorContractAdapter.php
 rename tests/{AgentCore/Application => CodingAgent/Session}/Replay/TurnTreeReplayFilterTest.php (96%)
 rename tests/{AgentCore/Domain/Run => CodingAgent/Session/TurnTree}/TurnTreeProjectorTest.php (99%)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/session-07a-move-turn-tree-projector-and-filter-to-app-session.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/session-07a-move-turn-tree-projector-and-filter-to-app-session.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User reported PR #262 was merged and asked to move the tracked task to DONE. Moving task through workflow completion.
