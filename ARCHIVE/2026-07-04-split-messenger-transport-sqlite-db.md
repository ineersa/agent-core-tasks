# Split Messenger transport into separate SQLite database

## Goal
Context: Session 6 forensic investigation proved the TUI hang was caused by a child read tool completing successfully, then ToolCallResult dispatch/apply failing with `SQLSTATE[HY000]: database is locked`. Messenger retry scheduling also failed because retry persistence uses the same SQLite DB as app/runtime state. The result left ExecuteToolCall message_id 284 delivered/in-flight, tool_batch_state finalized=false, and the child run stuck running. Symfony/Doctrine connection PRAGMA checks show busy_timeout=5000 and journal_mode=wal, so the next architectural hardening is to split queue transport storage from app/runtime state storage while staying local/self-contained.

## Acceptance criteria
- Messenger Doctrine transports (run_control, llm, tool, agent, mcp/failed as applicable) use a dedicated SQLite DB file separate from the app/runtime Doctrine DB that stores tool_batch_state, background_process, tool_question, session metadata, etc.
- App/runtime state remains on its own SQLite DB with existing migrations/schema behavior; Messenger setup creates/uses its own transport table in the queue DB.
- Both SQLite connections explicitly enforce WAL and busy_timeout through the real Symfony/Doctrine runtime path, with automated proof querying the actual connections.
- Regression coverage proves an app-state write lock/failure during ToolCallResult handling does not prevent Messenger from persisting retry/redelivery metadata, or otherwise proves the chosen architecture prevents the session-6 retry-wedge class.
- No assumption-based changes to tool cancellation or read timeout behavior are reintroduced.
- Focused Castor validation passes: relevant DB/Messenger tests, phpstan on changed production files, deptrac, cs-check.

## Workflow metadata
Status: DONE
Branch: task/2026-07-04-split-messenger-transport-sqlite-db
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-04-split-messenger-transport-sqlite-db
Fork run: mpip1s4pn6zj
PR URL: https://github.com/ineersa/agent-core/pull/257
PR Status: merged
Started: 2026-07-04T02:45:57.946Z
Completed: 2026-07-04T19:58:42.178Z

## Work log
- Created: 2026-07-04T02:45:52.505Z

## Task workflow update - 2026-07-04T02:45:57.946Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-07-04-split-messenger-transport-sqlite-db.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-07-04-split-messenger-transport-sqlite-db.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-07-04-split-messenger-transport-sqlite-db.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-07-04-split-messenger-transport-sqlite-db.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-07-04-split-messenger-transport-sqlite-db.

## Task workflow update - 2026-07-04T02:48:03.577Z
- Recorded fork run: 88nw7uryqsxd
- Summary: Started implementation. Created worktree `/home/ineersa/projects/agent-core-worktrees/2026-07-04-split-messenger-transport-sqlite-db` on branch `task/2026-07-04-split-messenger-transport-sqlite-db` and launched fork 88nw7uryqsxd. Fork scope: split Messenger Doctrine transports onto a dedicated SQLite DB/connection separate from app/runtime state, update `JsonlProcessAgentSessionClient` DSN injection from `doctrine://default` to the new connection, prove both connections have WAL+busy_timeout through Doctrine, and add regression proof that app-state DB write lock does not block Messenger transport persistence. Explicitly forbidden: reintroducing cancellation throttling/read timeout changes or implementing broad stale in-flight recovery/outbox in this task.

## Task workflow update - 2026-07-04T02:53:32.199Z
- Recorded fork run: 88nw7uryqsxd
- Validation: Fork validation: castor test --filter='MessengerTransportSqliteIsolationTest|MessengerTransportSchemaEnsurerTest|JsonlProcessAgentSessionClientTransportDsnTest|ApplicationMigrationExecutorTest|SqliteConnectionConfigTest' OK — 11 tests, 37 assertions; Fork validation: castor phpstan on changed src paths OK — 0 errors; Fork validation: castor deptrac OK — 0 violations; Fork validation: castor cs-check OK — clean; Parent verification: task worktree fast-forwarded to 971dc0524 and is clean on branch task/2026-07-04-split-messenger-transport-sqlite-db
- Summary: Fork 88nw7uryqsxd completed implementation with commit 971dc0524 `fix(messenger): isolate transport storage from runtime state` but accidentally committed on the integration checkout `main` instead of the task worktree branch. Parent inspected both checkouts and non-destructively fast-forwarded the task worktree branch `task/2026-07-04-split-messenger-transport-sqlite-db` to the same commit, so the task branch now contains the implementation. Root `main` still points at the same commit and requires explicit user approval for a cleanup reset back to its previous HEAD f8d767c4a to remove the misplaced commit from integration checkout. Implementation summary: default DBAL app/runtime state remains `.hatfield/messenger.sqlite`; new `messenger_transport` DBAL connection uses `.hatfield/messenger-transport.sqlite`; controller env DSNs now use `doctrine://messenger_transport?...`; startup migrator ensures Messenger transport schema/WAL/busy_timeout on the transport connection; tests added for connection isolation, transport schema ensuring, and process DSN injection. Known gaps from fork: docs still mention old single-DB layout in some files; no stale in-flight recovery/outbox/idempotency; no full castor check/controller-replay yet.

## Task workflow update - 2026-07-04T02:57:02.442Z
- Validation: /home/ineersa/projects/agent-core: git reset --hard f8d767c4a completed; HEAD main=f8d767c4a; /home/ineersa/projects/agent-core-worktrees/2026-07-04-split-messenger-transport-sqlite-db: branch task/2026-07-04-split-messenger-transport-sqlite-db remains clean at 971dc0524
- Summary: User approved cleanup of the accidental implementation commit from integration checkout. Parent ran approved `git reset --hard f8d767c4a` in `/home/ineersa/projects/agent-core`, removing misplaced 971dc0524 from `main`. Task worktree/branch remains at 971dc0524 with clean status. Integration `main` is now back to f8d767c4a (still ahead of origin/main by 4 from pre-existing local merge commits, but no longer contains the queue split commit).

## Task workflow update - 2026-07-04T03:11:46.158Z
- Validation: Reviewer ran castor test --filter=MessengerTransportSchemaEnsurerTest PASS; Reviewer ran castor test --filter=MessengerTransportSqliteIsolationTest PASS once, FAIL immediate second run confirming idempotency bug (2 rows vs expected 1); Reviewer ran castor test --filter=JsonlProcessAgentSessionClientTransportDsnTest PASS; Reviewer ran castor test --filter=ApplicationMigrationExecutorTest PASS; Reviewer verified git check-ignore does not ignore messenger-transport.sqlite runtime files
- Summary: Reviewer subagent returned REQUEST CHANGES for 971dc0524. Blockers: `.hatfield/.gitignore` does not ignore `messenger-transport.sqlite`/WAL/SHM/journal (runtime queue DB leak risk); `MessengerTransportSqliteIsolationTest::testTransportWriteSucceedsWhileDefaultConnectionHoldsWriteTransaction` is non-idempotent because it inserts persistent `queue_name='isolation_test'` via a fresh connection and fails on immediate second run; `tests/paratest-bootstrap.php` transport DB path omits the QA run segment, violating per-run isolation; `docs/async-runtime-architecture.md` stale single-DB transport docs; doctrine busy_timeout comment/rationale was removed from config. Reviewer explicitly recommends keeping the app DB filename `.hatfield/messenger.sqlite` as-is in this task and renaming to `state.sqlite`/`data.sqlite` only in a separate follow-up because it can orphan existing local runtime data. Do not move to CODE-REVIEW until blockers fixed and re-reviewed.

## Task workflow update - 2026-07-04T03:13:26.024Z
- Recorded fork run: fzzbt3lx3hhq
- Validation: castor test --filter=MessengerTransportSqliteIsolationTest OK twice in a row — 3 tests, 9 assertions each; castor test --filter='MessengerTransportSchemaEnsurerTest|JsonlProcessAgentSessionClientTransportDsnTest|ApplicationMigrationExecutorTest|SqliteConnectionConfigTest' OK — 8 tests, 28 assertions; castor cs-check OK — clean; castor deptrac OK — 0 violations; git check-ignore -v .hatfield/messenger-transport.sqlite .hatfield/messenger-transport.sqlite-wal .hatfield/messenger-transport.sqlite-shm .hatfield/messenger-transport.sqlite-journal confirmed ignored; phpstan skipped by fork because no src PHP files changed in hygiene commit
- Summary: Fork fzzbt3lx3hhq addressed reviewer REQUEST CHANGES with commit 4ca384286 `fix(messenger): harden transport sqlite split hygiene`. Fixes: added `.hatfield/.gitignore` entries for `messenger-transport.sqlite` and sidecars; made MessengerTransportSqliteIsolationTest idempotent via unique queue name + cleanup; included HATFIELD_QA_RUN_ID segment in transport test DB path; updated docs/async-runtime-architecture.md to describe split DB + `doctrine://messenger_transport` DSNs; restored doctrine.yaml busy_timeout/PDO::ATTR_TIMEOUT rationale comment. App DB filename rename remains deferred to separate follow-up; no src production PHP changed in this hygiene commit.

## Task workflow update - 2026-07-04T03:20:00.192Z
- Validation: Reviewer: castor test --filter=MessengerTransportSqliteIsolationTest run 1 OK — 3 tests, 9 assertions; Reviewer: castor test --filter=MessengerTransportSqliteIsolationTest run 2 OK — 3 tests, 9 assertions; Reviewer: castor test --filter=MessengerTransportSchemaEnsurerTest OK — 2 tests, 3 assertions; Reviewer: castor test --filter=JsonlProcessAgentSessionClientTransportDsnTest OK — 1 test, 10 assertions; Reviewer: castor test --filter=ApplicationMigrationExecutorTest OK — 3 tests, 12 assertions; Reviewer: castor test --filter=SqliteConnectionConfigTest OK — 2 tests, 3 assertions; Reviewer: castor test:controller-replay OK — 8 tests, 112 assertions; Reviewer: castor phpstan --path=src/CodingAgent/Migrations/ OK — 0 errors; Reviewer: castor cs-check OK — 0 files fixed; Reviewer: git check-ignore for 4 transport DB files OK; Reviewer: git status clean
- Summary: Reviewer re-pass on 4ca384286 returned APPROVED. All prior blockers fixed: transport DB files ignored; isolation test idempotent; paratest transport DB path includes QA run segment; async runtime docs updated for two-DB split and `doctrine://messenger_transport`; doctrine busy_timeout rationale restored. Reviewer agrees app DB filename rename should remain a separate follow-up (misleading but not correctness issue; renaming needs migration/data-loss decision). Non-blocking suggestions: optionally restore old #228 reference in busy_timeout comment, and consider session-specific transport DB path in E2E bases later.

## Task workflow update - 2026-07-04T03:22:07.773Z
- Validation: move_task to CODE-REVIEW ran deterministic castor check and FAILED: quality failed: test:tui exit code 2; check-test:tui.log: BashBackgroundAcceptE2eTest timed out after 20s waiting for `Running`; check-test:tui.log: BashBackgroundCancelE2eTest timed out after 20s waiting for `Running`; capture showed `sleep 8` done/idle
- Summary: Attempted move to CODE-REVIEW after reviewer approval. The automatic deterministic `castor check` gate failed in the worktree, so the task remains IN-PROGRESS. Failure lane: `test:tui` exit code 2. Log `var/reports/qa-20260704-032014-700953-ecdb54b2/check-test:tui.log` shows two TmuxHarness errors: `BashBackgroundAcceptE2eTest::testAcceptingBashBackgroundPromptTracksProcessForBgStatus` timed out waiting for `Running`; `BashBackgroundCancelE2eTest::testBashBackgroundPromptPathCanBeCancelledWithoutLeaks` timed out waiting for `Running`. Last capture in cancel test showed bash command `sleep 8` already `done`/idle instead of background prompt path. Need diagnose whether split transport DB broke TUI replay/controller/worker timing/setup or exposed existing flake before retrying CODE-REVIEW.

## Task workflow update - 2026-07-04T03:33:02.485Z
- Recorded fork run: uun6gwnpmfvq
- Validation: castor test:tui --filter=BashBackgroundAcceptE2eTest OK; castor test:tui --filter=BashBackgroundCancelE2eTest OK on several solo reruns, but flaky/fails under paired/full ParaTest; castor test:tui full ParaTest FAILED after partial bash-only fix; shared transport DB isolation gap remains; castor test --filter=JsonlProcessAgentSessionClientTransportDsnTest OK
- Summary: Fork uun6gwnpmfvq partially diagnosed/fixed the CODE-REVIEW gate failure but did not commit. Root cause: unfiltered/ParaTest `castor test:tui` runs tmux-spawned agents/controllers that did not get per-test `HATFIELD_TEST_MESSENGER_TRANSPORT_DATABASE_PATH`; after the split they shared default `messenger_transport_test.sqlite` while app DBs were already unique, causing parallel TUI async-tool/HITL tests to contend on the queue DB and time out (bash background prompt path missing, cancel stuck). Filtered solo tests often pass because they are sequential. Partial uncommitted changes add TUI E2E env helpers and propagate explicit transport paths in some TUI/controller tests, but full `castor test:tui` still failed. Recommended finish: add `JsonlProcessAgentSessionClient` test-env derivation so when `APP_ENV=test` and `HATFIELD_TEST_DATABASE_PATH` is set but `HATFIELD_TEST_MESSENGER_TRANSPORT_DATABASE_PATH` is missing, controller subprocess env derives a paired messenger transport DB path; add unit coverage; validate full `castor test:tui`; commit.

## Task workflow update - 2026-07-04T03:35:50.246Z
- Recorded fork run: hlo63cpbizry
- Summary: Launched follow-up fork hlo63cpbizry to finish dirty WIP from uun6gwnpmfvq. Scope: implement robust `JsonlProcessAgentSessionClient` test-env derivation so controller subprocesses in APP_ENV=test derive `HATFIELD_TEST_MESSENGER_TRANSPORT_DATABASE_PATH` from unique `HATFIELD_TEST_DATABASE_PATH` when missing; add unit coverage for derived/preserved transport DB env; decide whether to keep partial TUI E2E env helper/.castor/controller changes; validate focused bash TUI filters plus full unfiltered `castor test:tui`, controller replay if relevant, phpstan/cs/deptrac; commit final fix. This addresses CODE-REVIEW gate failure caused by shared transport SQLite under parallel TUI tests.

## Task workflow update - 2026-07-04T03:40:42.564Z
- Recorded fork run: hlo63cpbizry
- Validation: castor test --filter=JsonlProcessAgentSessionClientTransportDsnTest OK — 3 tests, 16 assertions; castor test:tui --filter=BashBackgroundAcceptE2eTest OK — 1 test, 3 assertions; castor test:tui --filter=BashBackgroundCancelE2eTest OK — 1 test, 1 assertion; castor test:tui OK full ParaTest — 28 tests, 145 assertions; castor test:controller-replay OK — 8 tests, 112 assertions; castor phpstan --path=src/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClient.php OK — 0 errors; castor cs-check OK after cs-fix; castor deptrac OK — 0 violations; Full castor check not run by fork; should run via move_task CODE-REVIEW gate
- Summary: Fork hlo63cpbizry completed the CODE-REVIEW gate fix with commit ed2f35351 `fix(tests): isolate messenger transport db in tui subprocesses`. Root cause: full/ParaTest `castor test:tui` had tmux-spawned agent/controller subprocesses sharing default `messenger_transport_test.sqlite` because many TUI commands set unique `HATFIELD_TEST_DATABASE_PATH` but not unique `HATFIELD_TEST_MESSENGER_TRANSPORT_DATABASE_PATH`; solo filters passed because sequential. Fix: layered test DB isolation — `JsonlProcessAgentSessionClient` derives paired test messenger transport DB path when APP_ENV=test and only app DB path is set; Castor check env propagates transport path; controller E2E bases set per-session transport path; TUI bash background support uses new `TuiE2eDatabaseEnv` to pass both paths to tmux parent. DB rename remains deferred.

## Task workflow update - 2026-07-04T03:50:37.117Z
- Validation: Reviewer: castor test --filter=JsonlProcessAgentSessionClientTransportDsnTest PASS — 3 tests, 16 assertions; Reviewer: castor test:tui --filter=BashBackgroundAcceptE2eTest PASS — 1 test, 3 assertions; Reviewer: castor test:tui --filter="BashBackgroundCancelE2eTest|BashCancelFollowUpE2eTest" PASS — 2 tests, 2 assertions; Reviewer: castor test:tui --filter="TuiQueuedSteerE2eTest|TuiCompactHeaderE2eTest" PASS — 2 tests, 7 assertions; Reviewer: castor phpstan --path=src/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClient.php PASS — 0 errors; Reviewer: castor cs-check PASS — 0 violations; Reviewer: castor deptrac PASS — 0 violations; Reviewer: branch clean; 8 expected files only
- Summary: Reviewer re-pass on HEAD ed2f35351 returned APPROVED and explicitly said CODE-REVIEW retry is safe. Reviewer confirmed layered isolation for Messenger transport DB covers ParaTest workers, Castor check env, controller E2E, bash-background TUI parent, and `JsonlProcessAgentSessionClient` derivation safety net. No blockers. Non-blocking notes: production test-only derivation is a pragmatic acceptable trade-off despite being test-aware production code; optional docs note in tests/AGENTS.md for future TUI authors; pre-existing diagnostic path in ControllerE2eTestCase checks production transport DB location under test mode. Per user instruction, stopped after review and did not move task to CODE-REVIEW.

## Task workflow update - 2026-07-04T16:59:05.133Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (81.6s).
- Pushed task/2026-07-04-split-messenger-transport-sqlite-db to origin.
- branch 'task/2026-07-04-split-messenger-transport-sqlite-db' set up to track 'origin/task/2026-07-04-split-messenger-transport-sqlite-db'.
- Created PR: https://github.com/ineersa/agent-core/pull/257

## Task workflow update - 2026-07-04T17:29:56.848Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User requested including the app/runtime DB filename rename in the current task/PR: code should use `.hatfield/state.sqlite` instead of `.hatfield/messenger.sqlite`; user will manually rename existing local DB files, so no migration shim/dual-read fallback should be added. Moving back to IN-PROGRESS for a small follow-up commit, then re-review and re-run CODE-REVIEW gate.

## Task workflow update - 2026-07-04T17:36:15.434Z
- Recorded fork run: s7sycxfwfj0x
- Validation: castor test --filter=SqliteConnectionConfigTest\|MessengerTransportSqliteIsolationTest\|JsonlProcessAgentSessionClientTransportDsnTest OK — 8 tests, 28 assertions; castor test:controller-replay OK — 8 tests, 112 assertions; castor phpstan --path=src/CodingAgent/Migrations --path=src/CodingAgent/Runtime/Process OK — 0 errors; castor cs-check OK — 0 fixable; castor deptrac OK — 0 violations; Post-change grep for `messenger.sqlite` found only `.hatfield/.gitignore` stale-ignore block
- Summary: Fork s7sycxfwfj0x completed the app/runtime DB filename rename with commit 7e1934d84 `chore(db): rename default sqlite file to state`. Default Doctrine ORM/app runtime DB now uses `.hatfield/state.sqlite`; Messenger transport remains `.hatfield/messenger-transport.sqlite`. No migration shim, fallback, dual-read, or auto-rename was added per user instruction; existing dev DB files must be renamed manually. Updated config/docs/comments/gitignore and one TUI test comment. `.hatfield/.gitignore` now primarily ignores `state.sqlite` sidecars and intentionally keeps old `messenger.sqlite` ignores only as stale local runtime artifacts after manual rename.

## Task workflow update - 2026-07-04T17:42:03.262Z
- Validation: Reviewer accepted fork validation: focused DB/transport tests OK, controller replay OK, scoped phpstan OK, cs-check OK, deptrac OK; Reviewer grep/classification: no live config or production code path uses `messenger.sqlite`; old references only stale gitignore/historical/pre-existing doc drift
- Summary: Reviewer re-pass on HEAD 7e1934d84 returned APPROVED and said safe to move back to CODE-REVIEW. Reviewer confirmed default app/runtime DB path is changed to `.hatfield/state.sqlite`, Messenger transport remains `.hatfield/messenger-transport.sqlite`, no compatibility/migration shim was added, and remaining `messenger.sqlite` references are only intentional stale ignores in `.hatfield/.gitignore`, pre-existing testing skill doc drift, or historical plan docs. Non-blocking cosmetics: async runtime ASCII diagram alignment, RuntimeProcessConfig comment wrap, doctrine.yaml comment alignment.

## Task workflow update - 2026-07-04T17:43:35.940Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (82.2s).
- Pushed task/2026-07-04-split-messenger-transport-sqlite-db to origin.
- branch 'task/2026-07-04-split-messenger-transport-sqlite-db' set up to track 'origin/task/2026-07-04-split-messenger-transport-sqlite-db'.
- PR already exists: https://github.com/ineersa/agent-core/pull/257

## Task workflow update - 2026-07-04T18:01:33.223Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User/PR feedback: production code contains test-environment conditional derivation in JsonlProcessAgentSessionClient (`if APP_ENV=test` / `HATFIELD_TEST_*` handling). This is not acceptable: no test-specific env conditionals inside production code. Need remove the production test-only path and make test isolation explicit in test harness/Castor/controller/TUI setup instead.

## Task workflow update - 2026-07-04T18:08:29.848Z
- Recorded fork run: m4dbs6cuk3yn
- Validation: castor test --filter=JsonlProcessAgentSessionClientTransportDsnTest OK — 2 tests, 13 assertions; castor test --filter=TuiE2eDatabaseEnvTest OK — 2 tests, 6 assertions; castor test:tui --filter=BashBackgroundAcceptE2eTest OK — 1 test, 3 assertions; castor test:tui --filter=BashBackgroundCancelE2eTest OK — 1 test, 1 assertion; castor test:tui OK full ParaTest — 28 tests, 145 assertions; castor test:controller-replay OK — 8 tests, 112 assertions; castor phpstan --path=src/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClient.php OK — 0 errors; castor cs-check OK — 0 fixable; castor deptrac OK — 0 violations; Audit: `rg "HATFIELD_TEST|deriveTest|applyTestMessenger" src` clean (no matches); Fork audit: no TUI E2E command string sets only HATFIELD_TEST_DATABASE_PATH without TuiE2eDatabaseEnv helper
- Summary: Fork m4dbs6cuk3yn fixed PR feedback with commit 4b379d355 `fix(tests): keep transport db isolation out of production runtime`. Removed all test-only `APP_ENV=test` / `HATFIELD_TEST_*` derivation from production `JsonlProcessAgentSessionClient`; production now only injects legitimate `doctrine://messenger_transport?...` DSNs. Transport DB isolation now lives entirely in test/Castor harnesses: Castor QA env and paratest bootstrap, controller E2E bases, and TUI tmux command builders using `TuiE2eDatabaseEnv::shellPrefix()` to pass both app and transport DB paths. Updated/rewrote JsonlProcessAgentSessionClientTransportDsnTest to assert explicit parent env forwarding rather than production derivation, added `TuiE2eDatabaseEnvTest`, and updated TUI E2E command builders. Default app DB remains `.hatfield/state.sqlite`; Messenger transport remains `.hatfield/messenger-transport.sqlite`.

## Task workflow update - 2026-07-04T18:28:11.235Z
- Validation: Reviewer: grep for `HATFIELD_TEST|deriveTest|applyTestMessenger` in src clean; Reviewer: grep for `APP_ENV.*test|'test'.*APP_ENV` in src clean; Reviewer: castor test --filter=JsonlProcessAgentSessionClientTransportDsnTest OK — 2 tests, 13 assertions; Reviewer: castor test --filter=TuiE2eDatabaseEnvTest OK — 2 tests, 6 assertions; Reviewer: castor test --filter=MessengerTransportSchemaEnsurerTest OK — 2 tests, 3 assertions; Reviewer: castor test --filter=MessengerTransportSqliteIsolationTest OK — 3 tests, 9 assertions; Reviewer: castor phpstan --path src OK — 0 errors; Reviewer: castor cs-check --path src OK — production clean; Reviewer: castor deptrac OK — 0 violations; Reviewer: php -l on modified TUI E2E files OK; Reviewer did not rerun full test:tui/controller-replay; accepted fork green results
- Summary: Reviewer re-pass on HEAD 4b379d355 returned APPROVE WITH SUGGESTIONS. Substantive/user-reported blocker is fixed: no `HATFIELD_TEST*`, `APP_ENV=test` conditional, `deriveTest*`, or test-only DB derivation remains in `src/`; transport isolation is explicit in Castor/paratest/controller/TUI test harnesses. Reviewer says safe to move back to CODE-REVIEW after auto-fixing style nits. Non-critical but gate-relevant issue: reviewer found test-file cs violations in new/modified files (`TuiE2eDatabaseEnv.php`, `TuiE2eDatabaseEnvTest.php`, `JsonlProcessAgentSessionClientTransportDsnTest.php`, and triple blank lines in `CancelStickinessE2eTest.php`, `TuiQueuedSteerE2eTest.php`, `TuiStartupSnapshotTest.php`), so `castor cs-fix`/`castor cs-check` should run before CODE-REVIEW gate.

## Task workflow update - 2026-07-04T18:29:17.117Z
- Recorded fork run: mpip1s4pn6zj
- Validation: castor cs-fix on six targeted files completed; 5 files fixed in per-file pass, snapshot file already clean on rerun; castor cs-check on six targeted files OK — 0 fixable; castor test --filter=JsonlProcessAgentSessionClientTransportDsnTest OK — 2 tests, 13 assertions; castor test --filter=TuiE2eDatabaseEnvTest OK — 2 tests, 6 assertions
- Summary: Fork mpip1s4pn6zj completed style-only cleanup with commit 113b00ad8 `style(tests): fix transport db isolation test formatting`. It ran PHP CS Fixer on the six reviewer-flagged test files only, fixing assertion call style/blank-line/qualification formatting; no production code or behavior changed. This addresses the only gate-relevant issue from reviewer’s APPROVE WITH SUGGESTIONS for 4b379d355.

## Task workflow update - 2026-07-04T18:30:52.212Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (82.3s).
- Pushed task/2026-07-04-split-messenger-transport-sqlite-db to origin.
- branch 'task/2026-07-04-split-messenger-transport-sqlite-db' set up to track 'origin/task/2026-07-04-split-messenger-transport-sqlite-db'.
- PR already exists: https://github.com/ineersa/agent-core/pull/257

## Task workflow update - 2026-07-04T19:58:42.178Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-07-04-split-messenger-transport-sqlite-db into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/env.php                                    |   1 +
 .castor/helpers.php                                |   2 +
 .hatfield/.gitignore                               |   9 +
 config/packages/doctrine.yaml                      |  27 ++-
 config/packages/doctrine_migrations.yaml           |   2 +-
 config/packages/messenger.yaml                     |   5 +-
 config/packages/test/doctrine.yaml                 |   9 +
 config/services.yaml                               |   9 +
 docs/async-runtime-architecture.md                 |  38 +--
 docs/hitl-and-approvals.md                         |   4 +-
 migrations/Version20260617141000.php               |   6 +-
 .../Migrations/ApplicationMigrationExecutor.php    |   3 +-
 .../Migrations/MessengerTransportSchemaEnsurer.php | 104 +++++++++
 .../Migrations/StartupDatabaseMigrator.php         |   2 +
 .../Process/JsonlProcessAgentSessionClient.php     |  10 +-
 .../Runtime/Process/RuntimeProcessConfig.php       |   3 +-
 .../MessengerTransportSqliteIsolationTest.php      | 108 +++++++++
 .../Doctrine/SqliteConnectionConfigTest.php        |   1 +
 .../ApplicationMigrationExecutorTest.php           |  23 +-
 .../MessengerTransportSchemaEnsurerTest.php        |  70 ++++++
 .../Controller/E2E/ControllerE2eTestCase.php       |  23 +-
 .../Controller/E2E/ControllerReplayE2eTestCase.php |  11 +-
 ...nlProcessAgentSessionClientTransportDsnTest.php | 257 +++++++++++++++++++++
 tests/Tui/E2E/BashBackgroundE2eTestSupport.php     |   9 +-
 tests/Tui/E2E/CancelStickinessE2eTest.php          |  10 +-
 tests/Tui/E2E/SafeGuardApprovalTuiE2eTest.php      |   7 +-
 .../Tui/E2E/TuiAskHumanOverlayMarkdownE2eTest.php  |   7 +-
 tests/Tui/E2E/TuiAutoCompactionCancelE2eTest.php   |   9 +-
 tests/Tui/E2E/TuiAutoCompactionE2eTest.php         |  20 +-
 tests/Tui/E2E/TuiCompactCommandE2eTest.php         |  10 +-
 tests/Tui/E2E/TuiCompactHeaderE2eTest.php          |  10 +-
 tests/Tui/E2E/TuiE2eDatabaseEnv.php                |  68 ++++++
 tests/Tui/E2E/TuiE2eDatabaseEnvTest.php            |  34 +++
 tests/Tui/E2E/TuiJourneyE2eTest.php                |   8 +-
 tests/Tui/E2E/TuiMarkdownRenderE2eTest.php         |  10 +-
 tests/Tui/E2E/TuiOutputCapNoticeE2eTest.php        |  12 +-
 tests/Tui/E2E/TuiProviderErrorE2eTest.php          |  12 +-
 tests/Tui/E2E/TuiQueuedSteerE2eTest.php            |  10 +-
 tests/Tui/E2E/TuiResumeModelRestoreE2eTest.php     |  13 +-
 tests/Tui/E2E/TuiResumeSessionSwitchE2eTest.php    |  10 +-
 .../TuiRichTranscriptProductValidationE2eTest.php  |  10 +-
 tests/Tui/E2E/TuiStartupSnapshotTest.php           |  12 +-
 .../Tui/E2E/TuiStartupTranscriptConfigE2eTest.php  |  12 +-
 tests/Tui/E2E/TuiSubagentLiveViewE2eTest.php       |  10 +-
 tests/Tui/E2E/TuiSubagentProgressE2eTest.php       |  10 +-
 tests/Tui/E2E/TuiToolOutputE2eTest.php             |  22 +-
 tests/Tui/E2E/TuiTranscriptRenderE2eTest.php       |  10 +-
 tests/Tui/E2E/TuiTreeCommandE2eTest.php            |  10 +-
 tests/paratest-bootstrap.php                       |   8 +
 49 files changed, 937 insertions(+), 153 deletions(-)
 create mode 100644 src/CodingAgent/Migrations/MessengerTransportSchemaEnsurer.php
 create mode 100644 tests/CodingAgent/Doctrine/MessengerTransportSqliteIsolationTest.php
 create mode 100644 tests/CodingAgent/Migrations/MessengerTransportSchemaEnsurerTest.php
 create mode 100644 tests/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClientTransportDsnTest.php
 create mode 100644 tests/Tui/E2E/TuiE2eDatabaseEnv.php
 create mode 100644 tests/Tui/E2E/TuiE2eDatabaseEnvTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-07-04-split-messenger-transport-sqlite-db.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-07-04-split-messenger-transport-sqlite-db.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User reported PR #257 was merged. Integration checkout has only untracked runtime DB `.hatfield/state.sqlite` from user's manual local rename, so proceeding with requireCleanMain=false to avoid deleting/stashing runtime data. Task completed after merge/sync.
