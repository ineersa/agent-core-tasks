# GF-04: Repair stale cancellation and incomplete tool-call sessions

## Goal
Extract and redesign session repair independently from the abandoned fork MVP branch. Start from current origin/main and treat historical branch code only as forensic reference.

Historical work includes:
- `addca6eda` — simplified `/repair` command.
- `a5950f868` — stale cancellation repair.
- `e4bad96d9` — repair cancellation with unresolved tool call.

Important historical constraint: a prior guard rejected repair whenever `pendingToolCalls` contained `false`, but that is exactly the state produced by a tool that started and never completed. Conversely, clearing a pending tool call merely because a later result is considered stale/untracked would hide data loss and allow the run to advance incorrectly. Repair semantics must be derived from canonical events, not guessed from mutable snapshots or exception text.

Before implementation, perform forensic classification of recoverable versus non-recoverable states. The command UX should remain a single explicit `/repair` action unless the reviewed specification decides otherwise. Do not bring sequence allocation, fork execution, or broad replay refactors into this task.

## Acceptance criteria
- First commit/push contains only reviewed RED specification tests for each approved repair case; no production implementation. Accepted tests are immutable during implementation.
- Tests build canonical event sequences representing the real stale states; they do not mutate production session files manually after setup to force expected output.
- A replay-clean session stuck in Cancelling with no terminal agent_end is repaired by appending canonical abort/terminal events and replays to Cancelled.
- A tool call that started and never produced a durable result is never silently marked successful or cleared merely to advance the run.
- A durably committed tracked tool result is accepted exactly once; redelivery does not corrupt or duplicate terminal state.
- Duplicate/missing sequence corruption, active streaming, or ambiguous pending work produces a typed refusal rather than destructive repair.
- Repair is idempotent: running it again after successful repair makes no additional canonical changes.
- Repair never edits/reorders existing events; it appends through the canonical sequenced writer available on main at implementation time.
- Runtime diagnostics are structured and do not sniff exception messages or log raw prompts/tool output.
- All DB-touching tests use the Symfony test kernel/container; all QA uses Castor; no historical branch tests are copied.

## Workflow metadata
Status: DONE
Branch: task/gf-session-repair-stale-cancellation
Worktree: /home/ineersa/projects/agent-core-worktrees/gf-session-repair-stale-cancellation
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/283
PR Status: merged
Started: 2026-07-11T20:31:35.681Z
Completed: 2026-07-12T02:16:05.799Z

## Work log
- Created: 2026-07-11T17:43:23.429Z

## Task workflow update - 2026-07-11T20:31:35.681Z
- Moved TODO → IN-PROGRESS.
- Created branch task/gf-session-repair-stale-cancellation.
- Created worktree /home/ineersa/projects/agent-core-worktrees/gf-session-repair-stale-cancellation.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/gf-session-repair-stale-cancellation.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/gf-session-repair-stale-cancellation.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/gf-session-repair-stale-cancellation.

## Task workflow update - 2026-07-11T20:41:18.936Z
- Validation: vendor/bin/phpunit --configuration phpunit.xml.dist --filter=SessionRepairServiceTest: 8 tests, 8 errors — all LogicException: Not implemented (RED)
- Summary: ## RED specification tests committed

**Commit:** `5985439a4` on `task/gf-session-repair-stale-cancellation`

**Files created (3):**
- `src/CodingAgent/Session/Repair/RepairResult.php` — DTO for repair results
- `src/CodingAgent/Session/Repair/SessionRepairService.php` — shell with constructor deps; `repair()` throws `LogicException('Not implemented')`
- `tests/CodingAgent/Session/Repair/SessionRepairServiceTest.php` — 8 RED specification tests

**8 test scenarios:**
1. Clean terminal session → no repair needed (dry-run + apply, unchanged JSONL)
2. Dry-run reports stale non-terminal cancellation
3. Stale cancellation → append terminal events (AgentEnd cancelled), contiguous seq, replay Cancelled, first 7 lines unchanged
4. Unresolved tool never completed → append synthetic ToolExecutionEnd (not marked successful) + AgentEnd
5. Idempotent repair (second run: needsRepair=false, zero new events)
6. Repair never edits/reorders existing events (byte-identical prefix)
7. Duplicate sequences → typed refusal (duplicateSeqs non-empty, JSONL unchanged)
8. Active streaming → typed refusal (message contains "active streaming", JSONL unchanged)

**Validation:** `vendor/bin/phpunit --filter=SessionRepairServiceTest` → **8/8 LogicException: Not implemented** — RED as intended. All tests fail because the production implementation doesn't exist, proving they test real behavior.

**Test infrastructure:**
- Uses `TestDirectoryIsolation::createProjectTempDir()` for temp dirs
- Uses real `SessionRunEventStore` pointed at temp dir (via `HatfieldSessionStore` with stubbed `EntityManager`)
- Uses real `EventPayloadNormalizer`, `ReplayEventPreparer`, `RunStateReducer`, `EventFactory`
- Uses `InMemoryRunStore` for RunState
- Uses `RunLockManager` with temp `FlockStore`
- No DB (no kernel needed); no historical branch tests copied

## Task workflow update - 2026-07-11T23:52:45.000Z
- Validation: castor test --filter=SessionRepairServiceTest — OK (11 tests, 89 assertions); castor test --filter=RepairCommandHandlerTest — OK (3 tests, 8 assertions); castor test:tui --filter=TuiRepairCommandE2eTest — OK (1 test, 3 assertions); castor test --filter=TuiE2eDatabaseEnvTest — OK (3 tests, 10 assertions); castor test — OK (4247 tests); castor deptrac — OK (0 violations); castor phpstan --path=src/CodingAgent/Session/Repair — OK; castor cs-check focused repair/TUI paths — OK; castor check — intentionally not run during task-start phase
- Summary: ## Implementation complete — GF-04 stale cancellation repair

**Worktree:** `/home/ineersa/projects/agent-core-worktrees/gf-session-repair-stale-cancellation`

**Commits:**
- `5985439a4` — initial RED session-repair specifications
- `36ef2bf92` — extended RED typed-refusal and `/repair` TUI specifications
- `e3e772f5f` — GREEN session repair implementation and `/repair` command
- `62c389484` — restore accepted RED service/handler specs and align production
- `68ff0a3a6` — correct defective TUI RED database isolation after user review

**Production implementation:**
- Added `SessionRepairService` classification and append-only repair under `RunLockManager`.
- Added `SessionRepairRefusalReasonEnum` and structured `RepairResult` refusal data.
- Refuses duplicate/missing sequence corruption, active streaming, unavailable state, ambiguous pending work, replay validation failure, and CAS conflict without modifying events.
- Repairs canonical stale cancellation by appending reducer-compatible LLM abort, explicit cancelled/error tool end for canonically started unresolved tools, tool batch commit, and terminal `agent_end(cancelled)` events.
- Durable tool results are not duplicated; repeated repair is idempotent.
- Uses `EventStoreInterface` canonical append path; does not rewrite/reorder JSONL or add sequence allocation.
- Added safe single-action `/repair` TUI handler and registrar with structured user messages.
- Structured logs include run/component/event/refusal metadata and no raw prompts/tool output.

**Automated proof:**
- 11 service specifications for repair/refusal/idempotency/canonical preservation.
- 3 `/repair` handler tests.
- Real replay-backed `TmuxHarness` `/repair` E2E.
- E2E SQLite corrected to true per-scenario isolation: app and Messenger DBs physically live under `var/tmp/tui-e2e-repair-*/.hatfield/tmp/test-db/`; migration, PDO seed, and tmux agent resolve the same files.
- No production/global Doctrine configuration changed.

**Final worktree:** clean; expected 11 files changed from origin/main.

## Task workflow update - 2026-07-12T01:00:13.236Z
- Validation: castor test — OK (4262 tests, 14010 assertions); castor deptrac — OK (0 violations, 0 errors); castor phpstan — OK (0 errors); castor cs-check — OK (0 files requiring fixes); castor test:tui — OK (34 tests, 177 assertions; replay-backed, no live LLM); Reviewer final verdict — APPROVED at 29468c3fb1d7698f09b2e59fdf175055c1371ccb
- Summary: ## task-to-pr review complete

**Final reviewer verdict:** APPROVED on HEAD `29468c3fb1d7698f09b2e59fdf175055c1371ccb`.

Review iterations addressed all actionable findings:
- merged latest origin/main (`926ea2620`) and preserved PR #281 ancestry;
- canonical complete synthetic tool result group with replayed tool-message pairing;
- pre-append hypothetical replay validation and durable CAS-degraded semantics;
- latest-phase/multi-turn LLM abort classification;
- canonical event fixtures and exactly-once/orphan regression proofs;
- removed test-only/dead APIs; final service behind narrow interface;
- typed/refined result semantics and structured privacy-safe diagnostics;
- production migration FQCN compatibility, removing the test-only normalization workaround;
- truly isolated app/Messenger SQLite files for real TmuxHarness `/repair` E2E;
- handler catch logging without exception messages/session content;
- all final quality suggestions resolved.

**Final scope:** 13 files, 2320 insertions, 2 deletions versus current origin/main. Worktree clean.

**Reviewer:** APPROVED; mandatory real TmuxHarness proof verified.

## Task workflow update - 2026-07-12T01:02:23.648Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (119.2s).
- Pushed task/gf-session-repair-stale-cancellation to origin.
- branch 'task/gf-session-repair-stale-cancellation' set up to track 'origin/task/gf-session-repair-stale-cancellation'.
- Created PR: https://github.com/ineersa/agent-core/pull/283

## Task workflow update - 2026-07-12T01:02:30.425Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/283
- Updated PR Status: open
- Validation: Deterministic castor check — PASSED (119.2s); PR created: https://github.com/ineersa/agent-core/pull/283
- Summary: Moved to CODE-REVIEW after final APPROVED review. Deterministic `castor check` passed in 119.2s; branch pushed and PR #283 created.

## Task workflow update - 2026-07-12T02:16:05.799Z
- Moved CODE-REVIEW → DONE.
- Merged task/gf-session-repair-stale-cancellation into integration checkout.
- Merge made by the 'ort' strategy.
 .../Migrations/ApplicationMigrationExecutor.php    |   4 +-
 src/CodingAgent/Session/Repair/RepairResult.php    |  25 +
 .../Repair/SessionRepairRefusalReasonEnum.php      |  16 +
 .../Session/Repair/SessionRepairService.php        | 591 +++++++++++++
 .../Repair/SessionRepairServiceInterface.php       |  10 +
 src/Tui/Listener/RepairCommandHandler.php          | 112 +++
 src/Tui/Listener/RepairCommandRegistrar.php        |  48 +
 .../ApplicationMigrationExecutorTest.php           |  21 +
 .../Session/Repair/SessionRepairServiceTest.php    | 963 +++++++++++++++++++++
 tests/Tui/E2E/TuiE2eDatabaseEnv.php                | 116 +++
 tests/Tui/E2E/TuiE2eDatabaseEnvTest.php            |  19 +
 tests/Tui/E2E/TuiRepairCommandE2eTest.php          | 281 ++++++
 tests/Tui/Listener/RepairCommandHandlerTest.php    | 116 +++
 13 files changed, 2320 insertions(+), 2 deletions(-)
 create mode 100644 src/CodingAgent/Session/Repair/RepairResult.php
 create mode 100644 src/CodingAgent/Session/Repair/SessionRepairRefusalReasonEnum.php
 create mode 100644 src/CodingAgent/Session/Repair/SessionRepairService.php
 create mode 100644 src/CodingAgent/Session/Repair/SessionRepairServiceInterface.php
 create mode 100644 src/Tui/Listener/RepairCommandHandler.php
 create mode 100644 src/Tui/Listener/RepairCommandRegistrar.php
 create mode 100644 tests/CodingAgent/Session/Repair/SessionRepairServiceTest.php
 create mode 100644 tests/Tui/E2E/TuiRepairCommandE2eTest.php
 create mode 100644 tests/Tui/Listener/RepairCommandHandlerTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/gf-session-repair-stale-cancellation.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/gf-session-repair-stale-cancellation.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #283 confirmed merged on GitHub at 2026-07-12T02:15:37Z (merge commit d1ca0cf23bf0d75821e9b40359e7423379005c6b). Proceeding with DONE integration, pull, and worktree cleanup.

## Task workflow update - 2026-07-12T02:23:30.991Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/283
- Updated PR Status: merged
- Validation: PR #283 — MERGED at 2026-07-12T02:15:37Z; merge commit d1ca0cf23bf0d75821e9b40359e7423379005c6b; Initial `LLM_MODE=true castor check` — all lanes passed except test:tui fixed 120s timeout; cache guard/artifact integrity/leak check passed; `castor test:tui` diagnostic — OK (36 tests, 186 assertions, 124.8s), no assertion failures; `LLM_MODE=true HATFIELD_CHECK_TUI_PARATEST_PROCESSES=4 castor check` — PASSED (236.7s): test 4279/14079; controller replay 8/112; TUI 36/186; llm-real 10/121; deptrac/phpstan/cs-check OK; cache 292→292; artifact integrity OK; leak check OK; Integration checkout clean (no uncommitted changes); task worktree removed
- Summary: DONE completion verified. PR #283 merged (`d1ca0cf23bf0d75821e9b40359e7423379005c6b`), task branch integrated locally, remote changes pulled, task worktree removed, and integration checkout has no uncommitted changes. Post-merge deterministic gate passed with the documented supported TUI worker budget of 4. The first post-merge gate attempt timed out only in the TUI lane at its fixed 120s cap; isolated `castor test:tui` then passed 36 tests in 124.8s with no leaks, and the full gate passed after setting `HATFIELD_CHECK_TUI_PARATEST_PROCESSES=4` (all guards remained enabled).
