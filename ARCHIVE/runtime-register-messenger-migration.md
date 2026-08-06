# Register messenger_messages startup migration

## Goal
User observed startup PDOException in issue-221 worktree: `SQLSTATE[HY000]: General error: 1 no such table: messenger_messages` from Doctrine DBAL PDO connection. Investigation found `migrations/Version20260617141000.php` already creates `messenger_messages` with `CREATE TABLE IF NOT EXISTS`, but `ApplicationMigrationExecutor::KNOWN_MIGRATIONS` omits `DoctrineMigrations\Version20260617141000::class` while including later cache migration `Version20260617141001`. Runtime startup uses ApplicationMigrationExecutor, not filesystem discovery, so the migration is never applied at app startup. Fix should register the messenger migration in the known list before cache migration and add regression coverage that the startup executor applies it / keeps known migrations in sync.

## Acceptance criteria
- Fresh runtime startup migration creates `messenger_messages` before consumers can access Doctrine transport.
- `ApplicationMigrationExecutor::KNOWN_MIGRATIONS` includes `DoctrineMigrations\Version20260617141000::class` in chronological order before `Version20260617141001`.
- Regression test proves the startup executor creates `messenger_messages` from an empty SQLite DB (or otherwise detects known migration omission).
- Use Castor validation only: at minimum focused migration/runtime test, `castor phpstan`, `castor cs-check`; run `castor deptrac` if layer boundaries are touched.
- Do not use destructive git operations or raw vendor/bin QA commands.

## Workflow metadata
Status: DONE
Branch: task/runtime-register-messenger-migration
Worktree: /home/ineersa/projects/agent-core-worktrees/runtime-register-messenger-migration
Fork run: gfo0ap1y89mg
PR URL: https://github.com/ineersa/agent-core/pull/223
PR Status: merged
Started: 2026-06-26T18:58:49.262Z
Completed: 2026-06-26T19:59:09.484Z

## Work log
- Created: 2026-06-26T18:58:37.419Z

## Task workflow update - 2026-06-26T18:58:49.262Z
- Moved TODO → IN-PROGRESS.
- Created branch task/runtime-register-messenger-migration.
- Created worktree /home/ineersa/projects/agent-core-worktrees/runtime-register-messenger-migration.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/runtime-register-messenger-migration.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/runtime-register-messenger-migration.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/runtime-register-messenger-migration.
- Validation: Pre-start investigation: migrations/Version20260617141000.php exists and uses CREATE TABLE IF NOT EXISTS messenger_messages, but src/CodingAgent/Migrations/ApplicationMigrationExecutor.php KNOWN_MIGRATIONS omits Version20260617141000 while including Version20260617141001 cache migration.
- Summary: Starting implementation for user-reported startup PDOException: existing messenger_messages migration is omitted from runtime ApplicationMigrationExecutor known migrations.

## Task workflow update - 2026-06-26T18:59:30.257Z
- Recorded fork run: gfo0ap1y89mg
- Launched implementation fork gfo0ap1y89mg in worktree /home/ineersa/projects/agent-core-worktrees/runtime-register-messenger-migration. Scope: register Version20260617141000 in ApplicationMigrationExecutor known migrations, add focused startup migration regression test for messenger_messages table creation, run Castor validation, commit.

## Task workflow update - 2026-06-26T19:01:56.141Z
- Recorded fork run: gfo0ap1y89mg
- Validation: Fork read testing skill, tests/AGENTS.md, root AGENTS instructions, and task markdown before edits/QA.; castor test --filter=ApplicationMigrationExecutorTest PASS (2 tests, 3 assertions).; castor phpstan PASS.; castor cs-check PASS (0 files to fix).; castor deptrac PASS (0 violations).
- Summary: Implementation fork completed commit e3c40c90152ba11d9dde6e02243e5ff4a66dcce5. Registered DoctrineMigrations\\Version20260617141000 in ApplicationMigrationExecutor::KNOWN_MIGRATIONS before Version20260617141001, so PHAR-safe/runtime startup applies the existing messenger_messages migration. Added tests/CodingAgent/Migrations/ApplicationMigrationExecutorTest.php proving an empty isolated SQLite DB gets messenger_messages and records Version20260617141000 after executor run, and that repeated invoke is idempotent per process. Fork explained the original missing-table path: StartupDatabaseMigrator uses explicit known migrations, so omitting 17141000 let fresh DB startup skip messenger table creation before messenger consumers touched Doctrine transport.

## Task workflow update - 2026-06-26T19:05:26.155Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (47.9s).
- Pushed task/runtime-register-messenger-migration to origin.
- branch 'task/runtime-register-messenger-migration' set up to track 'origin/task/runtime-register-messenger-migration'.
- Created PR: https://github.com/ineersa/agent-core/pull/223
- Validation: Fork validation: castor test --filter=ApplicationMigrationExecutorTest PASS (2 tests, 3 assertions).; Fork validation: castor phpstan PASS.; Fork validation: castor cs-check PASS (0 files to fix).; Fork validation: castor deptrac PASS (0 violations).; Warmup after cache-guard failure: castor test:llm-real PASS (9 tests, 110 assertions); llama-proxy stats entries=138.; move_task CODE-REVIEW rerun will run deterministic castor check before pushing/opening PR.
- Summary: Per user instruction, skipping reviewer subagent; user will review PR. Implementation complete at e3c40c901: registered DoctrineMigrations\Version20260617141000 in ApplicationMigrationExecutor::KNOWN_MIGRATIONS before Version20260617141001 and added focused regression tests proving runtime startup executor creates messenger_messages on an empty SQLite DB and records the migration version. This fixes fresh-start PDOException `no such table: messenger_messages` caused by the PHAR-safe runtime migration list omitting the existing messenger migration. Initial CODE-REVIEW transition hit llama-proxy cache guard (cache grew 137→138); warmed proxy with castor test:llm-real, stats now entries=138, then retried deterministic gate.

## Task workflow update - 2026-06-26T19:59:09.484Z
- Moved CODE-REVIEW → DONE.
- Merged task/runtime-register-messenger-migration into integration checkout.
- Merge made by the 'ort' strategy.
 .../Migrations/ApplicationMigrationExecutor.php    |  1 +
 .../ApplicationMigrationExecutorTest.php           | 72 ++++++++++++++++++++++
 2 files changed, 73 insertions(+)
 create mode 100644 tests/CodingAgent/Migrations/ApplicationMigrationExecutorTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/runtime-register-messenger-migration.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/runtime-register-messenger-migration.
- Deleted branch task/runtime-register-messenger-migration.
- Pulled integration checkout: Already up to date..
- Validation: PR #223 was reviewed/merged externally per user confirmation.; PR branch passed deterministic castor check during CODE-REVIEW transition.
- Summary: User confirmed PR #223 merged. Moving task to DONE and syncing integration checkout after issue #221 merge.
