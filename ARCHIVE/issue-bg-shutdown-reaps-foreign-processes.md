# Fix BackgroundProcessManager shutdown reaping foreign processes from shared DB (kills castor test)

## Goal
## Problem (root-caused and PROVEN via strace)

`BackgroundProcessManager::shutdownCleanup()` is called by the PHP shutdown handler (registered in `config/services.yaml` via `registerShutdownHandler: []`) with **no session argument**. With `null`, `fetchAllUnfinished(null)` returns **every unfinished background-process row across ALL sessions** from the CWD-resolved DB (`%app.cwd%/.hatfield/messenger.sqlite`). The handler then `stop()`s each → `ProcessLifecycle::sendTerm()` → `exec("kill -TERM -<pgid>")` → process-group SIGTERM.

### The trigger: PharSmokeTest shares the agent's DB

`PharSmokeTest::shellExecIsolated()` isolates **HOME** but NOT **CWD**. `castor test` always sets `HATFIELD_BINARY_PATH` (via `phar_ensure()`), so `PharSmokeTest` always runs. The PHAR (`php hatfield.phar list`) boots in the project CWD → opens the **same `.hatfield/messenger.sqlite` the live agent writes to** → its `register_shutdown_function` fires `shutdownCleanup(null)` → reaps the agent's running `castor test` bash tree.

### strace proof (session 6, `/tmp/strace.log`)

```
1494 (php hatfield.phar list)  clone3 → 1858
1858  execve("/bin/sh", ["sh","-c","--","kill -TERM -1138 2>/dev/null"])   # exact sendTerm() string
1494  <... clone3 resumed>) = 1858                                          # PHAR spawned the killer
1858  kill(-1138, SIGTERM)                                                  # whole castor tree dies → exit 143
1494  --- SIGTERM {si_pid=1858} ---                                         # PHAR suicides (it's in group 1138)
```

The `TableNotFoundException` guard in `shutdownCleanup()` only short-circuits when no table exists — useless when the real agent's table is present.

### Blast radius

This is also a **production footgun**: any `hatfield list` / `hatfield about` / PHAR command run in a project CWD where an agent has running bash processes murders them on exit. 0 actual test defects in the killed runs.

### Note: this is NOT the castor-llm-mode extension (EXT-03)

EXT-03 is exonerated — the bug reproduces on `main` with no extension. This is a separate `BackgroundProcessManager` defect.

## Fix design (instance-scoped shutdown)

The shutdown handler must reap **only processes this PHP process instance started** — never foreign rows from the shared DB.

1. Add `private array $ownedPids = [];` to `BackgroundProcessManager`.
2. In `start()`, after `$pid = $launchResult['pid'];`, append `$this->ownedPids[] = $pid;`.
3. `shutdownCleanup(?string $sessionId = null)`:
   - **`$sessionId === null`** (the shutdown-handler path): iterate `$this->ownedPids` and `stop()` each (with the existing RuntimeException try/catch + structured warning log). This **never queries the DB for foreign rows** — a fresh PHAR `list` has empty owned pids → reaps zero. Extract a `private function reapOwnedProcesses(): int` helper.
   - **`$sessionId !== null`** (controller controlled-shutdown path): keep the existing `fetchAllUnfinished($sessionId)` DB behavior + `TableNotFoundException` guard. Session-scoped reap is legitimate (one session = one controller).
4. Update docblocks on `registerShutdownHandler()` and `shutdownCleanup()` to document the instance-scoped invariant (the existing comment claiming "terminates all running background processes" was the misreading that caused this).

### Why existing tests still pass

All current `shutdownCleanup()` callers in `BackgroundProcessManagerTest` (and `cleanupProcesses()` tearDown) start their processes via the SAME manager instance they reap from, and `IsolatedKernelTestCase` gives each test its own DB. So no-arg = owned-pids reaps exactly what each test started. No existing test relies on cross-session DB reap.

## Regression test (thesis)

> A manager instance must NOT terminate background processes it did not start, even when they exist as unfinished rows in the shared DB.

Add to `tests/CodingAgent/Tool/BackgroundProcessManagerTest.php` a test that: starts a long `sleep` via manager A (same container ProcessStore), constructs a **second** manager B sharing the same ProcessStore/DB (simulating the PHAR sharing the agent's DB) that started nothing, asserts `B->shutdownCleanup() === 0` and A's process is still running, then asserts `A->shutdownCleanup() === 1`. This test is RED on the old code (B would reap A's process) and GREEN on the fix. Use real subprocesses matching the existing test style.

## Acceptance criteria

- [ ] `BackgroundProcessManager` tracks owned pids; no-arg `shutdownCleanup()` reaps only owned pids.
- [ ] Session-scoped `shutdownCleanup($sessionId)` DB behavior preserved for the controller path.
- [ ] Regression test added (red before / green after) using real subprocesses + shared store.
- [ ] Docblocks updated to document instance-scoped invariant; no explanatory comments deleted.
- [ ] `castor test --filter=BackgroundProcessManagerTest` green (existing + new).
- [ ] `castor phpstan` 0 errors; `castor cs-check` clean.
- [ ] `castor test` (default suite) green — no regression from shutdown-semantic change.
- [ ] Grep confirms no production no-arg `shutdownCleanup()` caller relies on cross-session DB reap.

## Out of scope (noted follow-ups, do NOT implement here)

- `PharSmokeTest` CWD isolation (defense-in-depth; the production fix alone fully resolves the bug).
- Auditing `refreshAllUnfinished(null)` callers (completion poller) — note any all-sessions caller but do not change.

## Acceptance criteria
- BackgroundProcessManager no-arg shutdownCleanup() reaps only pids this instance started (owned-pids tracking added in start())
- Session-scoped shutdownCleanup($sessionId) DB behavior preserved for controller controlled-shutdown
- Regression test proves a foreign manager instance sharing the DB reaps zero and does not kill another instance's process (red before fix, green after)
- castor test --filter=BackgroundProcessManagerTest green; castor phpstan 0 errors; castor cs-check clean; castor test default suite green
- Docblocks document the instance-scoped invariant; no explanatory comments removed

## Workflow metadata
Status: DONE
Branch: task/issue-bg-shutdown-reaps-foreign-processes
Worktree: /home/ineersa/projects/agent-core-worktrees/issue-bg-shutdown-reaps-foreign-processes
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/227
PR Status: merged
Started: 2026-06-27T01:32:37.575Z
Completed: 2026-06-27T02:14:06.162Z

## Work log
- Created: 2026-06-27T01:32:30.127Z

## Task workflow update - 2026-06-27T01:32:37.575Z
- Moved TODO → IN-PROGRESS.
- Created branch task/issue-bg-shutdown-reaps-foreign-processes.
- Created worktree /home/ineersa/projects/agent-core-worktrees/issue-bg-shutdown-reaps-foreign-processes.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/issue-bg-shutdown-reaps-foreign-processes.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/issue-bg-shutdown-reaps-foreign-processes.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/issue-bg-shutdown-reaps-foreign-processes.

## Task workflow update - 2026-06-27T01:37:21.715Z
- Validation: castor test --filter=BackgroundProcessManagerTest => OK 14 tests, 45 assertions (incl. new regression test); castor test --filter=PharSmokeTest => OK 4 tests, 12 assertions (the trigger test); castor phpstan => 0 errors (fork-reported); castor cs-check => clean after one cs-fix pass on production file (fork-reported); castor test (default suite) => OK 3704 tests, 11858 assertions — no regression from shutdown-semantic change (fork-reported); grep shutdownCleanup src/: only no-arg prod caller = registerShutdownHandler closure; HeadlessController uses explicit session — two-mode split safe
- Summary: Fork implemented the instance-scoped shutdown fix on branch task/issue-bg-shutdown-reaps-foreign-processes @ 6e2697da1.

Production (src/CodingAgent/Tool/BackgroundProcessManager.php): added private $ownedPids[] tracked in start(); shutdownCleanup() now branches — null session reaps ONLY owned pids via new reapOwnedProcesses() (never queries the DB for foreign rows), explicit session keeps the existing fetchAllUnfinished($sessionId) DB path + TableNotFoundException guard for the HeadlessController controlled-shutdown. Docblocks updated (not deleted) to document the instance-scoped invariant; RuntimeException catch + structured warning log preserved (no empty catch).

Regression test (tests/.../BackgroundProcessManagerTest.php): testShutdownCleanupDoesNotReapProcessesStartedByAnotherInstance — manager A starts sleep 30; a second manager sharing the SAME container ProcessStore (faithful PHAR-shares-agent-DB repro) with empty owned pids reaps 0 and leaves A's process running; A then reaps its own (1). Red on old code, green on fix. Added createOtherManager() helper.

Verified by parent: commit 6e2697da1 exists; diff --stat = 2 files (66+/8- prod, 40+ test) — no scope creep; grep confirms the only no-arg production shutdownCleanup caller is the registerShutdownHandler closure; HeadlessController passes explicit session.

Next: task-to-pr (CODE-REVIEW) runs castor check gate in worktree, pushes branch, opens PR.
- Diagnosed root cause via strace (session 6, /tmp/strace.log): PHAR 'list' (PID 1494) spawned sh -c 'kill -TERM -1138' on PHP shutdown, group-killing the whole castor test tree → exit 143. shutdownCleanup(null) reaped ALL unfinished rows from the shared CWD DB.
- Created task + moved IN-PROGRESS (worktree created).
- Launched implementation fork in worktree; fork committed 6e2697da1 with instance-scoped shutdown fix + regression test.
- Parent verified commit, diff scope (2 files), production logic correctness, regression test red/green reasoning, and re-ran focused BackgroundProcessManagerTest (14/45) + PharSmokeTest (4/12) green.

## Task workflow update - 2026-06-27T02:12:17.522Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (59.9s).
- Pushed task/issue-bg-shutdown-reaps-foreign-processes to origin.
- branch 'task/issue-bg-shutdown-reaps-foreign-processes' set up to track 'origin/task/issue-bg-shutdown-reaps-foreign-processes'.
- Created PR: https://github.com/ineersa/agent-core/pull/227

## Task workflow update - 2026-06-27T02:14:06.162Z
- Moved CODE-REVIEW → DONE.
- Merged task/issue-bg-shutdown-reaps-foreign-processes into integration checkout.
- Merge made by the 'ort' strategy.
 src/CodingAgent/Tool/BackgroundProcessManager.php  | 66 +++++++++++++++++++---
 .../Tool/BackgroundProcessManagerTest.php          | 40 +++++++++++++
 2 files changed, 98 insertions(+), 8 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/issue-bg-shutdown-reaps-foreign-processes.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/issue-bg-shutdown-reaps-foreign-processes.
- Pulled integration checkout: Merge made by the 'ort' strategy..
