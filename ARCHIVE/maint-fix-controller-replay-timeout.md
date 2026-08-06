# Fix flaky controller-replay tool-wait timeout (5s under parallel load)

## Goal
## Bug
`castor check` intermittently fails the `test:controller-replay` lane under parallel load. Observed twice in the QH-06 post-merge gate cycles this session.

Failure signature (from failed lane log):
```
ControllerReplaySmokeTest::testControllerReplayToolExecution
read tool must start. Event types: command.ack, run.started
Events collected: 2
Messenger DB: missing
Controller running: yes
Tracked PIDs: 184146
```
Standalone re-run of `castor test:controller-replay` passes cleanly (8/113) on the same commit. → **parallel-load flake, not a code regression.**

## Root cause
`tests/CodingAgent/Runtime/Controller/E2E/ControllerReplaySmokeTest.php:87`:
```php
$events = $this->collectEventsUntilToolCompleted('read', 5.0);
```
Hardcoded 5.0s timeout. Under `castor check` parallel load (unit ParaTest 4 workers + tui 2 + llm-real 2 + controller-replay all competing), the controller subprocess + Messenger consumers are CPU-starved and the tool-execution path (run_control → LLM replay → tool consumer → tool_execution.started/completed) exceeds 5s wall-clock.

## The fix already exists next door — this wait was missed
The sibling waits in the SAME test were already bumped to 12s for the identical reason:
- `ControllerE2eTestCase::liveControllerReadyTimeout()` returns `12.0` — docblock: *"Under full castor check, ParaTest llm-real (4 workers) competes with other parallel lanes; 5s flakes while standalone llm-real passes."*
- `ControllerE2eTestCase::liveLlmToolWaitTimeout()` returns `12.0` — docblock: *"Wall-clock budget for live LLM tool smoke tests (first LLM turn + tool execution). Replay-backed controller tests may pass with shorter timeouts; live llama.cpp is slower."*

5.0s is the one leftover hardcoded value below the canonical budget. The failed-run evidence matches precisely: `runtime.ready` + `run.started` arrived (12s budget worked), then tool execution starved past 5s.

## Fix (one line)
`ControllerReplaySmokeTest.php:87`: replace hardcoded `5.0` with `$this->liveLlmToolWaitTimeout()` (12.0). Proven-by-analogy to the sibling timeouts already in place.

## Out of scope (do NOT touch)
- The other replay tests' timeouts are already generous (8–30s) and don't flake. `ControllerReplayBashCancelFollowUpTest.php:112` uses 8.0 for `tool_execution.started` — borderline but not the reported flake; leave it unless a fork finds evidence it flakes.
- No production code changes. Test infrastructure only.

## Validation
1. `castor test:controller-replay` standalone — passes (sanity).
2. `castor check` × 2–3 runs — demonstrate the parallel-load flake is gone (the whole point). The flake rate observed was ~1-in-2 parallel runs, so 2–3 clean consecutive `castor check` runs is strong evidence.
3. `castor deptrac`, `castor phpstan`, `castor cs-check` — clean (test-only change, should be trivially green).

## Acceptance criteria
- ControllerReplaySmokeTest::testControllerReplayToolExecution uses liveLlmToolWaitTimeout() (12.0) instead of hardcoded 5.0
- castor test:controller-replay passes standalone
- castor check passes at least 2× consecutively under parallel load with no controller-replay flake
- deptrac / phpstan / cs-check clean

## Workflow metadata
Status: DONE
Branch: task/maint-fix-controller-replay-timeout
Worktree: /home/ineersa/projects/agent-core-worktrees/maint-fix-controller-replay-timeout
Fork run: ebf3b67b6
PR URL: https://github.com/ineersa/agent-core/pull/234
PR Status: merged
Started: 2026-06-29T18:22:03.238Z
Completed: 2026-06-29T18:29:07.121Z

## Work log
- Created: 2026-06-29T18:21:53.059Z

## Task workflow update - 2026-06-29T18:22:03.238Z
- Moved TODO → IN-PROGRESS.
- Created branch task/maint-fix-controller-replay-timeout.
- Created worktree /home/ineersa/projects/agent-core-worktrees/maint-fix-controller-replay-timeout.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/maint-fix-controller-replay-timeout.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/maint-fix-controller-replay-timeout.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/maint-fix-controller-replay-timeout.

## Task workflow update - 2026-06-29T18:26:45.499Z
- Recorded fork run: ebf3b67b6
- Validation: castor test:controller-replay standalone: OK 8 tests / 112 assertions; castor check run 1: quality ok (226.2s) — all lanes green, controller-replay 8/112, cache 198→198; castor check run 2: quality ok (160.0s) — all lanes green, controller-replay 8/112, cache 198→198; deptrac: 0 violations; phpstan: 0 errors; cs-check: clean; leak check: ok (both runs)
- Summary: Fork ebf3b67b6 applied the one-line fix: ControllerReplaySmokeTest.php:87 `collectEventsUntilToolCompleted('read', 5.0)` → `$this->liveLlmToolWaitTimeout()` (12.0). The hardcoded 5.0s was the lone leftover below the canonical budget; sibling waits (liveControllerReadyTimeout, liveLlmToolWaitTimeout) already returned 12.0 for the identical parallel-load reason. Comment updated to explain the rationale.

Independently verified: diff is exactly the surgical change, liveLlmToolWaitTimeout() confirmed returns 12.0.

Validation by fork:
- castor test:controller-replay standalone: OK 8/112
- castor check run 1/2: quality ok (226.2s) — all 7 lanes green, controller-replay 8/112, cache 198→198
- castor check run 2/2: quality ok (160.0s) — all 7 lanes green, controller-replay 8/112, cache 198→198
- deptrac 0, phpstan 0, cs-check clean, leak check ok both runs

Two consecutive clean castor check passes under parallel load is the proof the flake is resolved (flake rate was ~1-in-2). Trivial test-infra change, no reviewer complexity.

## Task workflow update - 2026-06-29T18:28:03.496Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (59.9s).
- Pushed task/maint-fix-controller-replay-timeout to origin.
- branch 'task/maint-fix-controller-replay-timeout' set up to track 'origin/task/maint-fix-controller-replay-timeout'.
- Created PR: https://github.com/ineersa/agent-core/pull/234

## Task workflow update - 2026-06-29T18:29:07.121Z
- Moved CODE-REVIEW → DONE.
- Merged task/maint-fix-controller-replay-timeout into integration checkout.
- Merge made by the 'ort' strategy.
 .../Runtime/Controller/E2E/ControllerReplaySmokeTest.php           | 7 +++++--
 1 file changed, 5 insertions(+), 2 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/maint-fix-controller-replay-timeout.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/maint-fix-controller-replay-timeout.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: Merged via PR #234. One-line boyscout fix: ControllerReplaySmokeTest tool-completion wait changed from hardcoded 5.0s to $this->liveLlmToolWaitTimeout() (12.0s), matching the sibling waits already using the canonical budget for parallel-load resilience. Resolves the ~1-in-2 castor check controller-replay flake observed during QH-06 post-merge cycles. Test infrastructure only (1 file, +5/−2), no production code.
