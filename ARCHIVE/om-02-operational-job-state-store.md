# OM-02 Extension-owned Messenger runtime and durable storage

## Goal
Plan: /home/ineersa/projects/agent-core/.aiassistant/reports/observational-memory-core-implementation-plan.md

Create the OM extension runtime independently of Hatfield infrastructure. The extension owns its SQLite database, Symfony Messenger bus/Doctrine transport, retry and failure-transport configuration, persistent consumer command, consumer supervisor, schema, migrations, and repositories.

Hatfield must not know the OM database path, transport DSN, queue names, message classes, schema, or consumer command. Symfony Messenger supplies durable delivery, safe concurrent claims, acknowledgement/rejection, delayed redelivery, retries, and failure transport behavior; OM must not recreate those mechanisms.

## Acceptance criteria
- OM uses an extension-owned SQLite database, defaulting to a documented path such as `.hatfield/extensions-data/observational-memory/om.sqlite`; it does not use Hatfield's Doctrine application database or Messenger transport database.
- The extension configures its own Symfony Messenger bus, SQLite/Doctrine transport, retry strategy, and failure transport without receiving Hatfield buses, transport DSNs, database connections, or container internals.
- Durable extension schema covers observations, reflections, source coverage, compaction request/results, and schema versioning/migrations. Messenger transport tables remain operational queue state.
- OM ships a normal persistent Messenger consumer command and an extension-owned supervisor that starts, monitors/restarts, and stops it through the generic runtime lifecycle notifications from OM-01.
- Consumer bootstrap has an explicit recursion guard so loading the OM extension inside its own consumer does not start another supervisor.
- The supervisor and consumer are process-safe when multiple Hatfield runtimes are active; Messenger transport claims provide concurrent message safety without application-level singleton job leases.
- Message handlers can commit durable OM results idempotently when a process crashes after result persistence but before Messenger acknowledgement.
- Storage/repository APIs are extension-local and do not leak into Hatfield Extension API.
- No observation/reflection content or OM job state is written to Hatfield `events.jsonl`, Hatfield Doctrine entities, or Hatfield Messenger transports.
- Focused Castor validation is run through Castor only. Messenger/runtime/process lifecycle work follows the testing skill and `tests/AGENTS.md`, and required runtime validation includes `castor check` with no leaked consumer processes.

## Explicit non-goals
- No custom queue lease, claim, retry, or acknowledgement implementation.
- No Observer/Reflector model prompts in this task.
- No Hatfield-owned OM scheduler or consumer supervision.
- No generic Extension API background-service registry.

## Workflow metadata
Status: ARCHIVE
Branch: task/om-02-operational-job-state-store
Worktree: /home/ineersa/projects/agent-core-worktrees/om-02-operational-job-state-store
Fork run: n7qkr1x1lt0x
PR URL: https://github.com/ineersa/agent-core/pull/313
PR Status: merged
Started: 2026-07-22T19:46:54.523Z
Completed: 2026-07-23T20:47:51.018Z

## Work log
- Created: 2026-06-28T21:32:59.619Z
- Revised: 2026-07-21 — replaced Hatfield operational-job tables with extension-owned SQLite, Symfony Messenger transport, persistent consumer, and supervisor.

## Task workflow update - 2026-07-22T19:46:54.523Z
- Moved TODO → IN-PROGRESS.
- Created branch task/om-02-operational-job-state-store.
- Created worktree /home/ineersa/projects/agent-core-worktrees/om-02-operational-job-state-store.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/om-02-operational-job-state-store.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/om-02-operational-job-state-store.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/om-02-operational-job-state-store.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/om-02-operational-job-state-store.
- Summary: User approved implementation as an architecture-preview PR with no reviewer yet and tests optional/minimal. Direction: extension-owned OM Messenger runtime and SQLite DB; one persistent OM consumer per active owning controller for MVP, no internal worker parallelism, no shared all-extension consumer, no Hatfield Messenger bus/transport/database use. Add only the narrow Hatfield bootstrap/lifecycle facilities needed for the extension to start/stop its own supervisor and boot a child with process-local ExtensionApi access. Push and create PR for direct user inspection.

## Task workflow update - 2026-07-22T19:47:55.866Z
- Recorded fork run: ygs0jsicso48
- Summary: Implementation fork launched for the user-approved architecture preview: extension-owned supervisor and private Symfony Messenger/SQLite runtime, narrow Symfony EventDispatcher owning-controller lifecycle events, generic extension entrypoint bootstrap only, one OM consumer per controller, no Hatfield Messenger/Doctrine/event-log OM state. Fork will commit but not push; parent will inspect then push/open PR without reviewer.

## Task workflow update - 2026-07-22T20:03:45.791Z
- Recorded fork run: n7qkr1x1lt0x
- Summary: Initial architecture-preview commit e7e3ac7c882745ebb6d28bbd98b82ae24c0a71b3 inspected. Before push, a narrow polish fork is addressing concrete incompleteness: wire periodic supervisor health/restart, use PSR LoggerAwareInterface instead of production NullLogger, remove test-driven optional EventDispatcher/runtime config seams, remove unused executablePath, stop orphan OM worker when its exact owning controller dies, and correct activation docs. No reviewer or full gate; second commit only.

## Task workflow update - 2026-07-22T20:13:03.751Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/313
- Updated PR Status: open
- Validation: Focused Castor tests: PASS (34 tests / 99 assertions); focused retest PASS (23 / 57).; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS.; Not run by user direction: reviewer, controller-replay, full castor check.
- Summary: Architecture-preview implementation completed and pushed for direct user inspection without reviewer. Commits: e7e3ac7c882745ebb6d28bbd98b82ae24c0a71b3 (private OM SQLite/Messenger runtime + narrow host bootstrap) and 1c9ff3d06b84dde9cf0de5417b821a77e17a1749 (periodic supervision, PSR logger injection, required lifecycle dependencies, parent-death stop, DTO/docs polish). Draft PR #313 created. Task intentionally remains IN-PROGRESS pending user architecture feedback; no task-to-pr reviewer/full gate yet.

## Task workflow update - 2026-07-23T14:08:28.245Z
- Summary: User rejected PR #313 topology after review comments: Hatfield generic extension:run bootstrap plus an extension-owned supervisor/programmatic Messenger runtime duplicates machinery and gains nothing. Superseding direction: rebuild OM as a complete self-contained Symfony package with its own dependencies, config, bin/console, configured Messenger bus/transports/handlers, SQLite schema, and small supervisor launching the package's own console. Remove Hatfield launch/lifecycle DTO/command machinery; retain only the smallest generic native Symfony integration needed for the extension to observe existing ConsoleEvents and receive host logger injection. No reviewer.

## Task workflow update - 2026-07-23T16:03:39.957Z
- Validation: castor test --filter=OmSchemaMigratorTest|ObservationRepositoryIdempotencyTest|OmConsoleLifecycleSelectionTest|OmConsumerSupervisorTest|OmParentDeathListenerTest|OmPackageConsoleSmokeTest|ExtensionManagerTest => 24 tests / 74 assertions OK; package bin/console list + om:migrate + messenger:setup-transports smoke OK; castor deptrac 0 violations; castor phpstan 0 errors; castor cs-check clean after cs-fix
- Summary: Completed self-contained OM package rebuild at 77513a352. No Hatfield extension:run/lifecycle DTOs. Minimal ExtensionManager LoggerAware + EventSubscriber support only. OM owns Kernel/config/bin/console/Messenger/schema/repos/supervisor. Validation green. Not pushed.

## Task workflow update - 2026-07-23T16:57:27.394Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/313
- Updated PR Status: open
- Validation: Focused OM + ExtensionManager Castor tests: PASS.; Package bin/console list / om:migrate / messenger:setup-transports / handler wiring smoke: PASS.; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS.; Not run by direction: reviewer, controller replay, full castor check.
- Summary: Rebuilt architecture pushed to draft PR #313 at ffa0c43212038a8931a08303746f8b2b245ca38f. The rejected Hatfield extension:run/custom lifecycle/programmatic Messenger topology is reverted. OM is now a self-contained Symfony package with declared dependencies, Kernel, bin/console, standard Doctrine Messenger config, private SQLite schema/repositories, and a small extension supervisor that directly launches OM bin/console migrate/setup/consume. Hatfield intrusion is limited to native PSR LoggerAware injection and Symfony EventSubscriber registration in ExtensionManager. Startup occurs during register() via ArgvInput because subscribers added during the in-flight COMMAND event cannot receive that event; native TERMINATE/ERROR stop the supervisor. Draft PR title/body updated for direct user inspection; no reviewer/status transition/full gate.

## Task workflow update - 2026-07-23T17:40:40.777Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (126.2s).
- Pushed task/om-02-operational-job-state-store to origin.
- branch 'task/om-02-operational-job-state-store' set up to track 'origin/task/om-02-operational-job-state-store'.
- PR already exists: https://github.com/ineersa/agent-core/pull/313
- Validation: Focused OM + ExtensionManager tests passed.; Package console/migration/Messenger wiring smoke passed.; Deptrac, PHPStan, and cs-check passed.
- Summary: User accepted the self-contained Symfony package direction as sufficient to proceed. OM owns dependencies, Kernel, bin/console, configured Messenger transports, SQLite schema/repositories, and supervisor; Hatfield retains only native LoggerAware/EventSubscriber integration. No further test expansion requested while remaining OM tasks are shaped.

## Task workflow update - 2026-07-23T18:15:46.576Z
- Summary: Reviewer completed with APPROVE WITH SUGGESTIONS and no blocking findings. Architecture, isolation, Messenger configuration, idempotency, privacy posture, and minimal Hatfield integration were accepted. Suggestions: avoid unbounded captured consumer output, clarify early session-id fallback, explain PDO timeout magic number, consider moving test-only OmDatabase out of production, document Revolt assumption, and sync latest main before merge. Reviewer initially claimed Process child env would contain only explicit variables when $_ENV is empty; parent verification against Symfony Process::start() showed Process always merges getDefaultEnv(), so that claim is not valid. However, current unset() calls do not suppress inherited Hatfield markers; Symfony Process requires setting env entries to false to remove them.

## Task workflow update - 2026-07-23T18:35:39.929Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User approved reviewer suggestions for implementation: discard/drain child consumer output to avoid long-running Process memory growth while preserving the existing 3600s time limit and 256M memory limit; move test-only OmDatabase helper out of production; suppress inherited Hatfield process markers using Symfony Process false env values; clarify session-ID fallback and Revolt-loop assumptions; sync current main before returning to review.

## Task workflow update - 2026-07-23T18:41:42.817Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (115.5s).
- Pushed task/om-02-operational-job-state-store to origin.
- branch 'task/om-02-operational-job-state-store' set up to track 'origin/task/om-02-operational-job-state-store'.
- PR already exists: https://github.com/ineersa/agent-core/pull/313
- Validation: Focused OM + ExtensionManager tests: PASS (28 tests / 90 assertions).; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS.
- Summary: Implemented all user-approved review suggestions at f45f95a7e after merging current origin/main: disabled output capture for long-running consumer while preserving one 3600s time limit and 256M memory limit; moved OmDatabase test helper to tests/Support; removed inherited Hatfield role markers with Symfony Process false env values; clarified session fallback/Revolt assumptions; documented PDO timeout. Existing reviewer verdict remains APPROVE WITH SUGGESTIONS and its suggestions are resolved.

## Task workflow update - 2026-07-23T20:47:51.018Z
- Moved CODE-REVIEW → DONE.
- Merged task/om-02-operational-job-state-store into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/om-02-operational-job-state-store.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/om-02-operational-job-state-store.
- Pulled integration checkout: Already up to date..
- Validation: Reviewer verdict: APPROVE WITH SUGGESTIONS; approved suggestions implemented.; Final deterministic castor check passed in 115.5s.; PR #313 merged by user.
- Summary: User confirmed PR #313 was merged and requested task completion.

## Task workflow update - 2026-08-06T20:59:22.772Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
