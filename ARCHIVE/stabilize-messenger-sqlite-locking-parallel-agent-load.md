# Stabilize SQLite Messenger locking under parallel agent load

## Goal
Manual live smoke on PR #284/session 1 produced multiple `Unhandled console exception (last-resort)` records associated with database locking while parallel subagents and HITL were active. Investigate as a standalone runtime reliability issue rather than masking it in the SubagentExecutionService refactor. First establish which SQLite database/connection is locked (`.hatfield/messenger-transport.sqlite`, `.hatfield/state.sqlite`, or test/runtime alternative), exact exception class/stack, operation/queue, transaction owner, and correlation fields. Preserve privacy: no raw prompts/tool output/environment values in diagnostics. Do not assume the fix is merely increasing timeouts; identify transaction/consumer contention and correct ownership/retry/configuration layer.

## Acceptance criteria
- Capture a deterministic or bounded live reproduction of parallel subagent + HITL load that triggers the lock, with structured run/session/component/event_type evidence and no sensitive content
- Identify the exact SQLite file, Doctrine/Messenger connection, SQL operation, lock holder/transaction lifetime, and consumer topology responsible
- Expected transient SQLite contention is handled at the correct Messenger/DB boundary with bounded behavior; no last-resort unhandled console exception escapes
- Evaluate WAL, busy_timeout, transaction scope, receiver concurrency, and bounded retry/backoff based on evidence; document why the selected mechanism is correct
- Preserve durable deferred-tool completion, batch projection CAS semantics, message idempotency, and cancellation/HITL behavior under retries
- Add focused regression coverage at the lowest layer that reproduces the real contention contract; use live controller validation if replay cannot reproduce process/locking topology
- Structured diagnostics are rate-limited/actionable and include correlation fields without raw prompts, tool output, DB contents, or environment values
- Run focused Castor tests plus deptrac, phpstan, cs-check, and full castor check before CODE-REVIEW

## Workflow metadata
Status: ARCHIVE
Branch: task/stabilize-messenger-sqlite-locking-parallel-agent-load
Worktree: /home/ineersa/projects/agent-core-worktrees/stabilize-messenger-sqlite-locking-parallel-agent-load
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/297
PR Status: merged
Started: 2026-07-16T22:09:15.530Z
Completed: 2026-07-17T00:24:26.206Z

## Work log
- Created: 2026-07-14T21:41:24.882Z

## Task workflow update - 2026-07-14T21:42:18.713Z
- Summary: This task is the task-board implementation tracker for existing open GitHub issue #268, not a new independent bug: https://github.com/ineersa/agent-core/issues/268 — `Messenger Doctrine receiver fails with "database is locked" under concurrent worker load (SQLite)`. Issue #268 already proves messenger-transport.sqlite, DoctrineReceiver.php:59, SQLITE_BUSY after the configured 5s busy_timeout, WAL active, shared transport file across queues, consumer death/restart, and correlated JSONL buffer pressure. The 2026-07-14 PR #284 manual parallel-subagent/HITL smoke is a fresh reproduction under increased Messenger load. Prior DONE task/PR #257 split Messenger transport from state.sqlite, but intentionally did not solve contention among all Messenger queues sharing messenger-transport.sqlite.
- Linked existing issue #268 as canonical bug report and retained this task only as its workflow/implementation tracker.
- No pre-existing TODO/IN-PROGRESS/CODE-REVIEW task for issue #268 was found; prior related task `2026-07-04-split-messenger-transport-sqlite-db` is DONE and addressed state-vs-transport isolation, not same-transport concurrent writer contention.

## Task workflow update - 2026-07-16T22:09:15.530Z
- Moved TODO → IN-PROGRESS.
- Created branch task/stabilize-messenger-sqlite-locking-parallel-agent-load.
- Created worktree /home/ineersa/projects/agent-core-worktrees/stabilize-messenger-sqlite-locking-parallel-agent-load.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/stabilize-messenger-sqlite-locking-parallel-agent-load.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/stabilize-messenger-sqlite-locking-parallel-agent-load.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/stabilize-messenger-sqlite-locking-parallel-agent-load.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/stabilize-messenger-sqlite-locking-parallel-agent-load.
- Summary: Implementation approved. Scope narrowed to a messenger_transport-only DBAL middleware that starts outer SQLite transactions with BEGIN IMMEDIATE, focused contention/transaction-scoping tests, and concise configuration rationale. No new observability subsystem or broad controller/docs expansion unless focused evidence shows it is needed.

## Task workflow update - 2026-07-16T22:27:58.941Z
- Validation: castor test --filter=MessengerSqliteImmediateTransactionMiddlewareTest — OK (5 tests, 19 assertions); castor test --filter=MessengerTransportSqliteIsolationTest — OK (3 tests, 9 assertions); castor deptrac — OK, 0 violations; castor phpstan — OK, no errors; castor cs-check — OK, 0 fixable; castor check intentionally not run during task-start phase
- Summary: Implemented narrowed SQLite locking fix in task worktree. Commits: c18b4f13a adds messenger_transport-only Doctrine DBAL middleware/driver/connection using BEGIN IMMEDIATE while leaving default state.sqlite unchanged; 27411bf54 replaces initial standalone DB test setup with convention-compliant Symfony test-kernel subprocess workers and deterministic writer/claim barriers. Added focused wiring, contention, nested savepoint, commit/rollback, and default-connection proofs; updated concise Doctrine config rationale. Final conformance pass read root AGENTS.md, testing skill, and tests/AGENTS.md in full and found no further changes needed. No observability subsystem, controller/TUI E2E, generic retries, timeout changes, vendor edits, or broad docs added. Integration main was restored after an initial fork cwd mistake; task worktree is clean on task/stabilize-messenger-sqlite-locking-parallel-agent-load.

## Task workflow update - 2026-07-16T23:48:08.290Z
- Validation: castor test — first run FAILED: unrelated SubagentLivePickerControllerTest::dismissFeedbackReplacesStaleExportFeedbackInPickerHeader assertion under ParaTest; castor test --filter=SubagentLivePickerControllerTest — OK (17 tests, 56 assertions); castor test — rerun OK (4463 tests, 15071 assertions); castor deptrac — OK, 0 violations/errors; castor phpstan — OK, 0 errors; castor cs-check — OK, 0 fixable files; castor check not run because CODE-REVIEW transition was deferred while GitHub is down
- Summary: Local task-to-PR preparation completed through reviewer approval, but CODE-REVIEW transition/push/PR was intentionally not attempted because the user reported GitHub is down. Review fix commits: 2f2417ed7 strengthened begin-specific timing and subprocess lifecycle; 18bde677e hardened cleanup/stdout draining and removed dead worker mode; 6992b109b performed final simplification/comment/test cleanup. Final reviewer decision on HEAD 6992b109b6a99ca016fcd5b801370d94f7431916: APPROVED with no actionable findings. Worktree is clean. Resume by rerunning/confirming focused validation if desired, then move_task to CODE-REVIEW once GitHub is available; that transition will run deterministic castor check, push, and create the PR.
- Final local reviewer gate: APPROVED on commit 6992b109b6a99ca016fcd5b801370d94f7431916; no actionable findings remain.
- GitHub outage blocker recorded: did not call move_task(to=CODE-REVIEW), did not push, and did not create a PR.

## Task workflow update - 2026-07-17T00:14:15.889Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (125.5s).
- Pushed task/stabilize-messenger-sqlite-locking-parallel-agent-load to origin.
- branch 'task/stabilize-messenger-sqlite-locking-parallel-agent-load' set up to track 'origin/task/stabilize-messenger-sqlite-locking-parallel-agent-load'.
- Created PR: https://github.com/ineersa/agent-core/pull/297
- Validation: castor test — rerun OK (4463 tests, 15071 assertions); initial unrelated ParaTest flake was isolated and passed (SubagentLivePickerControllerTest: 17 tests, 56 assertions); castor deptrac — 0 violations/errors; castor phpstan — 0 errors; castor cs-check — 0 fixable files; Final reviewer decision — APPROVED
- Summary: Final reviewer APPROVED commit 6992b109b6a99ca016fcd5b801370d94f7431916 with no actionable findings. Local focused validation is green; proceeding now that GitHub is available.

## Task workflow update - 2026-07-17T00:24:26.207Z
- Moved CODE-REVIEW → DONE.
- Merged task/stabilize-messenger-sqlite-locking-parallel-agent-load into integration checkout.
- Merge made by the 'ort' strategy.
 config/packages/doctrine.yaml                      |   3 +
 config/services.yaml                               |   7 +
 ...ssengerSqliteImmediateTransactionConnection.php |  23 ++
 .../MessengerSqliteImmediateTransactionDriver.php  |  20 ++
 ...ssengerSqliteImmediateTransactionMiddleware.php |  22 ++
 ...gerSqliteImmediateTransactionMiddlewareTest.php | 267 +++++++++++++++++++++
 ...rSqliteImmediateTransactionKernelTestKernel.php |  34 +++
 ...engerSqliteImmediateTransactionKernelWorker.php | 215 +++++++++++++++++
 8 files changed, 591 insertions(+)
 create mode 100644 src/CodingAgent/Infrastructure/Doctrine/MessengerSqliteImmediateTransactionConnection.php
 create mode 100644 src/CodingAgent/Infrastructure/Doctrine/MessengerSqliteImmediateTransactionDriver.php
 create mode 100644 src/CodingAgent/Infrastructure/Doctrine/MessengerSqliteImmediateTransactionMiddleware.php
 create mode 100644 tests/CodingAgent/Doctrine/MessengerSqliteImmediateTransactionMiddlewareTest.php
 create mode 100644 tests/CodingAgent/Doctrine/Support/MessengerSqliteImmediateTransactionKernelTestKernel.php
 create mode 100644 tests/CodingAgent/Doctrine/Support/MessengerSqliteImmediateTransactionKernelWorker.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/stabilize-messenger-sqlite-locking-parallel-agent-load.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/stabilize-messenger-sqlite-locking-parallel-agent-load.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: User live test: no SQLite lock errors observed under tested load; PR #297 confirmed MERGED via gh
- Summary: User confirmed production testing completed without SQLite lock errors and PR #297 was merged. GitHub confirms PR state MERGED at 2026-07-17T00:23:43Z (merge commit dce42f18d911f00dd74b5d56eaa7de7907db9a04).

## Task workflow update - 2026-07-17T00:26:40.118Z
- Validation: LLM_MODE=true castor check — OK (QA run qa-20260717-002429-640353-baecc7d3): deptrac, 4459 unit/integration tests, 8 controller-replay tests, 37 TUI tests, 10 llm-real tests, phpstan, cs-check all passed; proxy cache stable 386→386; artifact integrity and leak checks passed
- Summary: Post-merge validation completed successfully in the integration checkout. Task closure confirmed after user live testing found no lock errors.

## Task workflow update - 2026-08-06T20:59:35.254Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
