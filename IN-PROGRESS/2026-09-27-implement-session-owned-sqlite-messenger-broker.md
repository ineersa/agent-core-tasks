# Implement a session-owned PHP Messenger broker backed by SQLite

## Goal
## Implementation plan

[Session-owned SQLite Messenger broker plan](../reports/2026-09-27-session-owned-sqlite-messenger-broker-plan.md)

Plan baseline: agent-core `8490bee7200887f58c099befd1d13081e7d01634`. This task authorizes planning and tracks future implementation; no implementation has started.

## Agreed direction

One controller-owned PHP broker, started and stopped through the existing supervision lifecycle, owns one durable Messenger SQLite database inside the controller-root session's configured storage directory. All six durable queues and their publishers/consumers use that broker. Existing child-agent work continues to share the parent controller's queues; it does not create a broker per child run.

Start with existing Symfony Doctrine queue operations/schema and Doctrine DBAL, not an in-memory queue engine or custom journal. Reuse Symfony Messenger serializers, stamps and retries; Symfony Process/Lock and runtime executable resolution; existing Amp socket/byte-stream facilities and Revolt. Keep `run_control` single-owner, existing worker counts, application state.sqlite, canonical events, MCP ownership, and Scheduler behavior.

The goal is to remove transport writer-lock competition and unnecessary idle pickup delay. Storage commits and serial run-control work still cost time. No speedup is presumed.

## Sequence

1. Confirm queue/failure contracts, session bootstrap and ownership, and repeatable current-transport baseline.
2. Build the smallest socket broker and Messenger adapter over existing SQLite operations; validate with ordinary polling first.
3. Integrate session storage, broker-only queue migration access, positive startup readiness, shutdown, and restart boundaries.
4. Add bounded readiness waiting coordinated with Messenger idle sleep and delayed availability.
5. Prove durability/failure handling, source/PHAR/native lifecycle, and performance with matching durability; independent review and normal workflow QA gate.

## Decisions required before implementation/default adoption

- Confirm fail-visible handling of ambiguous socket outcomes without automatic RPC replay. Safe transparent retries would require separately approved durable deduplication, including after ACK.
- Confirm coordinated runtime handling of broker death while handlers may still execute. Do not silently apply independent consumer restart policy to the broker or automatically redrive claimed work.
- Decide isolated evaluation-only scope versus an approved offline migration of existing project-wide queue rows. No silent loss, reset of claimed rows, dual writers, or compatibility fallback.

## References evaluated

- TuxWeb/amp-sql-queue: reference only. Async SQL worker library using PostgreSQL/MySQL with leases and retry policy, not the requested broker.
- fabpot/amp-sqlite3: possible later measured alternative if database calls stall the broker loop. Starts a dedicated process per connection, needs ext-sqlite3/amphp-parallel and a DBAL/packaging compatibility evaluation. Not required for the first version.
- minnzen's SQLite agent-queue article/sqliteq: reference for conditional delivery acknowledgments. Its visibility-timeout retries are not Hatfield policy, and its single-process microbenchmarks do not establish Hatfield IPC/tail latency. Atomic UPDATE still acquires SQLite's writer lock.

Pinned source links and detailed findings are in the plan.

## Related work

- DONE: `2026-09-25-fix-current-tool-latency-and-parallel-batch-feedback`, PR #533, supplies the merged baseline.
- TODO: `2026-09-25-integrate-measured-messenger-concurrency-and-correct-tool-timing` remains a separate future worker-pool evaluation.
- TODO: `2026-09-05-revisit-safe-automatic-recovery-for-abandoned-messenger-deliveries` remains separate; this task does not authorize short leases, keepalive, or automatic recovery.

## Ownership

Main owns contract decisions and the coupled minimal broker/storage/adapter work. After those contracts are fixed, lifecycle and packaged-process proof may be one bounded, sequential fork slice if its subprocess investigation justifies delegation. No implementation owner/fork is assigned yet.

## Acceptance criteria
- Record the approved ambiguity, broker-failure, and existing-data cutover policies before implementing behavior that depends on them. No unresolved product decisions are delegated to a fork.
- Exactly one broker owns each controller-root session's durable queue database. Session path configuration, parent/child routing, and independent-session isolation are preserved; unknown session identities never share a fallback database/socket.
- Reuse existing Symfony Doctrine storage operations and schema through DBAL first. All queue writes, migrations and maintenance go through broker ownership; no worker-side auto-setup or direct queue access remains in the supported broker path.
- Preserve configured serializers, binary body/headers, DelayStamp precision, fetch size, queue ordering, terminal reject semantics, and Messenger-owned retries. Keep the current no-keepalive/long-redelivery-horizon policy and explicit repair baseline.
- Publish/reserve/ACK/reject confirmation follows commit. Ambiguous failures are handled according to the approved contract, with no hidden resubmission. Stale or foreign delivery receipts cannot acknowledge another reservation.
- Use bounded persistent Unix-socket requests, private namespace ownership, payload/output-buffer limits and backpressure. No broker message traffic leaks into runtime stdout streaming or default content logs.
- Controller acquires ownership and receives positive broker readiness before consumers/runtime.ready. Partial startup, EOF, new/resume/reload, broker/controller death and shutdown have deterministic owned-tree cleanup; persisted queue state survives ordinary shutdown.
- Bounded WAIT cannot miss ready work, reserves nothing itself, respects delayed messages and shutdown, and does not add a residual 50ms sleep or idle busy loop. Scheduler and TUI polling behavior remain unchanged.
- Deterministic lowest-layer tests cover real SQLite commit/claim behavior, lost confirmation, uncertain external effects, broker restart, stale ACK, delayed restart, storage errors, slow readers, ownership and teardown. DB tests boot the test container; no arbitrary sleeps, timing-window assertions or production test hooks.
- Same-workload evidence compares the merged current transport, test-only per-session direct Doctrine, polling broker, and WAIT-enabled broker with matching durability. Include payload/backlog/concurrent-session cases and latency tails, CPU, memory and process counts; do not infer a win from RAM-only or weaker-durability results.
- Verify source, PHAR and native executable startup and process cleanup using existing Castor facilities. Independent specification-fidelity review and transition-owned castor check pass before submission; post-merge check remains separate.
- No Symfony 8.2 migration, amphp/parallel pool, custom journal, full in-memory queue mirror, automatic lease recovery, general queue-admin API, speculative backend abstraction, or permanent benchmark fallback is introduced. Candidate async SQLite adoption requires a measured need and separate compatibility decision.

## Workflow metadata
Status: IN-PROGRESS
Branch: task/2026-09-27-implement-session-owned-sqlite-messenger-broker
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-27-implement-session-owned-sqlite-messenger-broker
Fork run: agent_62f58e21da12adf3
PR URL:
PR Status:
Started: 2026-10-15T20:42:26+00:00
Completed:

## Work log
- Created: 2026-09-27T18:40:50+00:00

## Task workflow update - 2026-09-27T18:41:34+00:00
- Summary: Created the implementation plan and TODO only. Main inspected current controller/supervisor, process DSNs, migration ownership, Doctrine storage and existing tests. Evaluated all three user-supplied references with pinned source revisions; none is selected as a first-version dependency. Product source, dependencies, task status and worker topology are unchanged. No tests or benchmarks were run for this planning deliverable.
- Procedure scope: .hatfield/skills/task-workflow/references/task-explain.md says, "This phase is read-only. Do not change task status or metadata, edit files, or launch an implementation fork." The user's explicit request to create a plan and task overrides that phase default only for these external-board planning artifacts. No implementation fork or production changes were authorized or started.
- Plan: reports/2026-09-27-session-owned-sqlite-messenger-broker-plan.md. First implementation uses existing Symfony/Doctrine SQLite storage behind a controller-owned broker. Ambiguous RPC handling, broker-failure behavior and existing-data cutover are explicit decisions to settle before implementation/default adoption.

## Task workflow update - 2026-09-27T19:01:55+00:00
- Summary: Whiteboard decision: build the reusable PHP SQLite broker as a separate project, with Hatfield limited to supervision, session-path configuration and Messenger integration. Added the required Messenger SQLite versus real-broker acceptance benchmark as linked TODO 2026-09-27-benchmark-messenger-sqlite-against-php-queue-broker. No benchmark code or candidate measurements exist yet.
- Authoritative scope revision: the user approved a separate reusable PHP broker inspired by goqite/sqliteq, with Revolt and an async SQLite client. This supersedes the original task/plan requirements to implement the broker inside CodingAgent, reuse Doctrine queue storage/schema for the candidate, and defer async SQLite until a later experiment. Doctrine remains the current benchmark baseline, not a mandatory candidate storage engine. Hatfield-specific supervision and session lifecycle remain integration concerns. Do not implement from the old step-by-step plan without reconciling this revision.
- Required acceptance deliverable: TODO/2026-09-27-benchmark-messenger-sqlite-against-php-queue-broker.md. Compare both through the real Messenger path at verified equivalent durability, with paired repeated runs, publish/pickup/ACK/four-result timing distributions, raw samples, failures and owned-tree resource costs. Count the async SQLite persistence process. Include a per-session direct-Doctrine control for multi-session measurements.
- Acceptance refinement: a benchmark report alone is not evidence of speed. A performance-win claim requires repeatable primary fan-in/batch-delivery tail-latency gains beyond baseline variation. Neutral, regressing or inconclusive evidence pauses default adoption for review. Keep long performance runs outside timing-sensitive unit assertions. A polling-only candidate is optional diagnosis, not a mandatory product mode.
- Planning-only scope: .hatfield/skills/task-workflow/references/task-explain.md states 'This phase is read-only. Do not change task status or metadata, edit files, or launch an implementation fork.' The user's benchmark/acceptance request is applied here to external-board planning records only. Status stays TODO, and no implementation or QA commands were run.

## Task workflow update - 2026-09-27T19:11:23+00:00
- Summary: Rewrote reports/2026-09-27-session-owned-sqlite-messenger-broker-plan.md to match the agreed independent-package MVP. Delayed delivery is mandatory, not deferred. The plan now separates generic broker/adapter/benchmark work from Hatfield supervision and integration, and removes the obsolete Doctrine-first implementation sequence. Planning only; no source or dependency changes.
- Current scope and acceptance below replace conflicting original body/acceptance text and earlier planning entries. The rewritten linked plan is the current specification, not the original Hatfield-internal proposal.
- MVP: one table for named queues; durable send and atomic receive; delivery-specific ACK and terminal reject; opaque bodies/headers; mandatory delayed delivery; explicit visibility/redelivery policy; bounded persistent socket server/client; consumer notifications; commit-before-confirmation and defined disconnect/restart behavior; thin Symfony Messenger adapter; standalone package benchmark.
- Delayed-delivery acceptance: preserve Messenger DelayStamp millisecond delays without early delivery or positive subsecond truncation; persist the original availability deadline; recover scheduling across broker restart without restarting the delay interval; make overdue messages eligible; wake idle consumers when due; handle a newly earlier deadline and queue isolation. Verify adapter mapping, before/after-deadline restart and wakeups with deterministic lowest-layer proof plus bounded real-process coverage. A broker supporting only immediate delivery cannot satisfy MVP.
- Deferred: priorities, batch send, batch receive, dead-letter administration, detailed queue stats, multiple SQLite drivers, and a separate typed-message framework. No full JS feature-suite port is required. Failure reporting and retained-state behavior remain mandatory despite deferred administration.
- The independent package owns Revolt, async SQLite, the protocol/client and generic Messenger adapter. Hatfield owns session paths, supervision/readiness, runtime failure integration, and cleanup. Complete storage operations must remain serialized across async waits. Preserve the existing conservative Hatfield recovery behavior pending explicit policy decisions.
- The benchmark belongs in the broker package bench/ directory and runs without Hatfield installed. Use standard Symfony Messenger Doctrine SQLite as the portable baseline and the real broker adapter as candidate, at equivalent durability. Symfony components may be benchmark-only dependencies. Hatfield-specific transport controls, four-tool journeys, Castor bootstrap, and session fixtures are not package benchmark requirements.
- Open decisions remain visibility/redelivery details, ambiguous RPC outcomes, broker-failure handling and existing Hatfield queue-data cutover. The selected async client must support a safe atomic claim despite the inspected RETURNING limitation; do not silently relax safety or change the agreed async direction.
- Procedure scope: .hatfield/skills/task-workflow/references/task-explain.md says "This phase is read-only. Do not change task status or metadata, edit files, or launch an implementation fork." The explicit user request to reflect the agreement in plan/task overrides that default for these external planning records only. Both tasks remain TODO; no implementation or QA runs.

## Task workflow update - 2026-09-27T20:10:51+00:00
- Summary: Confirmed per-session ownership for MVP after considering a shared project broker. Keep one broker and one queue database per controller-root session, supervised by that controller. This matches the current canonical plan; the project-wide shared-broker proposal is not selected.
- Final ownership decision: one broker and one durable queue database inside each controller-root session's configured storage directory. Child-agent work sharing that controller shares its broker; separate controller-root sessions have separate brokers, databases, and socket namespaces.
- The controller owns broker readiness, failure reporting, and shutdown. The broker owns its async SQLite persistence child, and teardown must account for the full owned tree. Preserve queued state across ordinary shutdown/resume and prevent overlapping owners of the same session database.
- Do not implement a detached project-wide broker, shared-project attach/detach lifecycle, or last-session shutdown coordination for MVP. Per-session ownership reduces lifecycle complexity and separates database contention and broker failures between sessions; it is not an OS security sandbox.
- The standalone reusable package, Revolt/async SQLite direction, mandatory delayed delivery, portable Messenger adapter, and package-owned benchmark remain unchanged. No implementation start or status transition is implied by 'start with per session for now'.

## Task workflow update - 2026-10-15T20:42:10+00:00
- Summary: Revived at the user's request for integration of the implemented /home/ineersa/projects/sqlite-queue package into Hatfield Messenger. Read current README and broker/transport contracts and inspected Hatfield controller, consumer supervision, DSN creation and Doctrine middleware.
- Current requested scope: consume the external broker package, preserve controller-root session ownership, replace direct Doctrine queue access and polling workarounds, integrate broker readiness/failure/shutdown, and evaluate native keepalive with realistic redelivery timeouts together with the abandoned-delivery recovery task. Package implementation is not part of this Hatfield task.
- Scope constraints: keep app-state SQLite concurrency protection unless evidence shows it is obsolete. The existing BEGIN IMMEDIATE middleware also protects state.sqlite; moving queue writes does not remove those app-state writers.
- Current evidence: package supports signal-safe native keepalive, separate notification/operations connections and native Messenger notification waits. Its docs explicitly say heartbeat failure does not stop the running handler, expired leases permit redelivery, and unconfirmed mutations must not be replayed automatically. Short leases require an approved loss-of-ownership policy, not only enabling --keepalive.
- Current benchmark evidence does not establish improved backlog throughput or lower total resource use. Broker integration must retain matching durability and measure real Hatfield workloads; no performance win is claimed. Resolve legacy queue cutover and broker-failure policy before implementing dependent behavior.

## Task workflow update - 2026-10-15T20:42:26+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-27-implement-session-owned-sqlite-messenger-broker.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-27-implement-session-owned-sqlite-messenger-broker.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-27-implement-session-owned-sqlite-messenger-broker.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-27-implement-session-owned-sqlite-messenger-broker.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-27-implement-session-owned-sqlite-messenger-broker.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-27-implement-session-owned-sqlite-messenger-broker/.idea.
- Summary: Begin external broker integration. Initial routing pass complete; finalize cutover and lost-ownership policies before implementation delegation.

## Task workflow update - 2026-10-15T20:46:56+00:00
- Summary: User finalized recovery and cutover policies: trial 60-second reservations with 5-second native keepalive, test actual redelivery causes; no legacy queue migration. Implement package integration in the task worktree.
- Authoritative clarification: user approved 60 sec + 5 sec keepalive for testing and will decide whether to keep it based on stability. Diagnose causes of redelivery; do not promise exactly-once external execution. No data migration is needed. Leave existing project-wide queue data untouched; no compatibility transport or hidden replay.
- Implement the previously proposed fail-visible session behavior on detected broker/ownership loss, not transparent broker reconnect/replay. Persist the broker queue across normal session shutdown/resume. Retain manual /repair and state.sqlite concurrency protection.
- Ownership: owner=fork; fork_run=none; revision=885bfec6d; scope=external sqlite-queue dependency, controller-owned broker lifecycle and Messenger cutover with 60s reservations/5s keepalive, removal of obsolete direct queue infrastructure, focused regression proof and affected docs/packaging; outcome=assigned; commit=none

## Task workflow update - 2026-10-15T21:12:41+00:00
- Recorded fork run: agent_5214862b6fbc9487
- Ownership: owner=fork; fork_run=agent_5214862b6fbc9487; revision=885bfec6d; scope=external broker integration and affected regression harnesses; outcome=blocked; commit=none
- First implementation handoff is partial, uncommitted and unvalidated. Production wiring and dependency cutover exist, but process/E2E harness repairs and known syntax failures remain. Raw php -l used by the fork is not accepted as project validation; all subsequent QA must use Castor. No completion or stability claim is made.

## Task workflow update - 2026-10-15T21:13:41+00:00
- Ownership: owner=fork; fork_run=agent_5214862b6fbc9487; revision=885bfec6d plus existing uncommitted integration; scope=finish existing broker integration in /home/ineersa/projects/agent-core-worktrees/2026-09-27-implement-session-owned-sqlite-messenger-broker, repair harnesses and establish focused Castor proof; outcome=assigned; commit=none
- Explicit continuation handoff: same fork retains sole writer ownership. Inspect current status/diff before resuming. Main inspected new lifecycle/path classes and flagged private socket directory, intentional exception logging, graceful broker-before-driver shutdown, full partial-start cleanup, native packaging compatibility and ownership-loss handling for correction/verification.

## Task workflow update - 2026-10-15T21:14:26+00:00
- Ownership: owner=fork; fork_run=agent_5214862b6fbc9487; revision=885bfec6d plus uncommitted integration; scope=initial external broker wiring; outcome=blocked; commit=none
- agent_resume refused the existing child because its context exceeded the resume threshold (187934 input tokens). Transfer sole-writer ownership to a fresh fork rather than duplicating an active child.
- Ownership: owner=fork; fork_run=none; revision=885bfec6d plus uncommitted integration; scope=finish core broker lifecycle/path safety and focused runnable transport/controller proof in existing task worktree; outcome=assigned; commit=none

## Task workflow update - 2026-10-15T21:38:36+00:00
- Ownership: owner=fork; fork_run=agent_bf3fc59f00d08cc8; revision=885bfec6d plus uncommitted integration; scope=core lifecycle/path corrections and focused proof; outcome=completed; commit=none
- Fork reports Castor-focused core suite passes (16 tests/124 assertions) and event-buffer suite (19/78); scoped PHPStan passes. Replay proof used raw vendor/bin/phpunit outside a Castor failure isolation and is not accepted as project validation. Repeat through Castor. Main found remaining owned-process-group/endpoint ownership safety issues and timing-based lease test proof to resolve before accepting integration.
- Ownership: owner=fork; fork_run=agent_bf3fc59f00d08cc8; revision=885bfec6d plus existing dirty integration; scope=resolve core owned-tree teardown and endpoint cleanup safety, deterministic renewal proof, reconcile remaining queue harnesses and Castor static/replay validation; outcome=assigned; commit=none

## Task workflow update - 2026-10-15T21:46:29+00:00
- Ownership: owner=fork; fork_run=946cc934-6f04-5d12-a8ca-ccbe10296b39; revision=885bfec6d plus dirty integration; scope=owned-process-group and endpoint safety corrections; outcome=completed; commit=none
- Linux process-group probe confirmed proc_open create_process_group did not isolate the broker; setsid via Symfony Process does. Teardown/path fixes now need rerun validation. Remaining harness reconciliation and ownership-loss fail-stop are not completed.
- Ownership: owner=fork; fork_run=946cc934-6f04-5d12-a8ca-ccbe10296b39; revision=885bfec6d plus dirty integration; scope=only DSN expectations, obsolete test harness cleanup and focused Castor test/format proof in existing task worktree; outcome=assigned; commit=none

## Task workflow update - 2026-10-15T21:47:29+00:00
- Ownership: owner=fork; fork_run=946cc934-6f04-5d12-a8ca-ccbe10296b39; revision=885bfec6d plus dirty integration; scope=remaining harness reconciliation; outcome=blocked; commit=none
- Resume refused due child context threshold (183304 latest input tokens). Transfer sole writer to a fresh bounded harness/validation fork. No prior fork is running.
- Ownership: owner=fork; fork_run=none; revision=885bfec6d plus dirty integration; scope=finish obsolete queue harness/DSN assumptions and run focused Castor proof, no new features; outcome=assigned; commit=none

## Task workflow update - 2026-10-15T22:02:23+00:00
- Recorded fork run: agent_62f58e21da12adf3
- Ownership: owner=fork; fork_run=agent_62f58e21da12adf3; revision=885bfec6d plus dirty integration; scope=test harness cutover and focused regression proof; outcome=completed; commit=none
- Current focused Castor results: 31 tests/174 assertions pass for broker paths/lifecycle/client, consumer argv, DSNs, application transaction middleware, startup migrations, isolation helpers. castor test:controller-replay passes 15 tests/871 assertions. Timing-window lease test removed; real Client send/renew/no-second-claim/ACK remains in supervisor test. Package owns engine expiry semantics.
- Main ran castor phpstan on current tree; two findings remain in SessionSqliteQueueBrokerSupervisor: unused nullable return at line 445 and forbidden short ternary at line 496. Deptrac did not run because the chained PHPStan command failed.
- Ownership: owner=fork; fork_run=agent_62f58e21da12adf3; revision=885bfec6d plus dirty integration; scope=final source-integration consistency, remove unused Doctrine Messenger dependency, extension guards/docs and Castor static checks; outcome=assigned; commit=none

## Task workflow update - 2026-10-15T22:13:08+00:00
- Ownership: owner=fork; fork_run=agent_62f58e21da12adf3; revision=c4e01af11b61c37e604b903e13eefdf7de1d936b; scope=final source integration consistency/dependencies/guards/docs; outcome=completed; commit=c4e01af11b61c37e604b903e13eefdf7de1d936b
- Source integration committed. Current Castor focused suite passes 36 tests/183 assertions; controller-replay passes 15 tests/871 assertions; phpstan has 0 errors, deptrac 0 violations/errors, docs validation and formatting pass. Main additionally ran castor test --filter=PharSmokeTest at c4e01af11: build/boot smoke and 8 tests/9003 assertions pass. This validates PHAR boot, not yet a PHAR broker/driver journey.
- Remaining acceptance: native/fused broker driver execution unresolved (Amp uses PHP_BINARY to run arbitrary worker PHP); native renewal-failure fail-stop not implemented because Symfony catches and logs renewal failures, handlers can continue. No stability/performance/exactly-once claim, no full castor check or independent review in task-start.
- Skill defect noted without changing skill: .agents/skills/testing/references/e2e.md instructs 'Pair app and Messenger transport databases through TuiE2eDatabaseEnv or the existing controller base.' The Messenger Doctrine database is removed by this requested integration; tests instead isolate broker storage per session. Follow the current product requirement and retain unaffected isolation rules.
- Ownership: owner=main; fork_run=none; revision=c4e01af11; scope=one-file stale invariant-comment correction in JsonlProcessAgentSessionClient and focused validation; outcome=assigned; commit=none

## Task workflow update - 2026-10-15T22:14:26+00:00
- Validation: castor test --filter='SessionSqliteQueue|ConsumerSupervisorTest|JsonlProcessAgentSessionClientTransportDsnTest|SqliteImmediateTransactionMiddlewareTest|StartupDatabaseMigratorTest|TuiE2eDatabaseEnvTest|ParaTestWorkerIsolationTest|ExtensionAgentJobRegistryTest': 36 tests, 183 assertions pass; castor test:controller-replay: 15 tests, 871 assertions pass; castor phpstan: errors=0,file_errors=0; castor deptrac: violations=0,errors=0; castor docs:validate and affected-path formatting pass; castor test --filter=PharSmokeTest: 8 tests, 9003 assertions pass; packaged command boot only, not a broker/driver journey; Final comment-only revision: castor cs-check --path=src/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClient.php and castor test --filter=JsonlProcessAgentSessionClientTransportDsnTest pass (3 tests/35 assertions); git diff --check clean; worktree clean
- Summary: Source broker integration committed on task branch; focused correctness, controller replay, static analysis, architecture and PHAR boot checks pass. Task stays IN-PROGRESS for packaged broker/native child-process compatibility and remaining recovery acceptance; no PR/full gate yet.
- Ownership: owner=main; fork_run=none; revision=1193f4cb0; scope=one-file invariant-comment correction and focused validation; outcome=completed; commit=1193f4cb0
- Implementation commits: c4e01af11b61c37e604b903e13eefdf7de1d936b and 1193f4cb0. No merge/push/PR/full castor check or independent review performed in task-start.
- Outstanding compatibility evidence: source Linux broker lifecycle is tested. PHAR build/boot is tested, but packaged broker/driver startup still needs a dedicated journey. Fused-native Amp driver process spawning is unresolved: ProcessContext::locateBinary returns PHP_BINARY under CLI; Hatfield native locators explicitly handle fused runtimes where PHP_BINARY is empty or self-executable. Do not assert that the native broker works or introduce an external-PHP fallback without approval.
- Outstanding recovery boundary: broker exit stops session; native renewal failure only warns in installed Symfony Worker and does not abort a running handler. User-approved 60s/5s policy is a trial of at-least-once delivery, not an exactly-once guarantee. Stability, native failure policy, live long-handler heartbeat behavior and Hatfield performance/resource measurements remain unproved.

## Task workflow update - 2026-10-15T23:33:53+00:00
- Summary: User confirmed proceeding with the 60s reservation/5s keepalive trial despite the explained at-least-once risk. Evaluate this broker in practice rather than treating duplicate-proof execution as a prerequisite.
- User clarification: current transport effectively has no automatic redelivery, only /repair; prior short-lease behavior failed because of SQLite contention and the PHP/Symfony heartbeat issue. User wants to see how the new broker behaves.
- Keep the approved 60s/5s configuration for testing. Do not claim duplicate execution is impossible, but do not add speculative fencing or treat exactly-once guarantees as a requirement for this trial. Preserve /repair.
- Agreed evaluation direction from preceding discussion: build a fresh candidate artifact from the integration branch, not the older installed executable; reuse a small Harbor SWE-bench case from the prior selected ten and existing tool-timing instrumentation. Normal-workflow evidence and controlled heartbeat-failure evidence answer different questions. No new Harbor run has been executed yet.

## Task workflow update - 2026-10-15T23:36:10+00:00
- Summary: User authorized a bounded old-versus-new Harbor trial using one small case from the prior selected ten, fresh branch-built artifacts, existing tool-timing measurements, and checks for errors/duplicates/owned-process cleanup.
- Evaluation budget: one baseline attempt and one candidate attempt on the same pinned SWE-bench case and matching settings. No retries until success. Validate fresh candidate executable/broker startup before spending model budget. Do not use/replace the user's installed executable.
- User explicitly requested Harbor SWE-bench execution. This overrides the testing live-llm reference's production-provider prohibition for this bounded benchmark only; it does not change ordinary E2E provider rules. Follow Harbor access-only private credential-copy procedure, never upload live auth/refresh tokens.
- Ownership: owner=fork; fork_run=agent_62f58e21da12adf3; revision=1193f4cb0; scope=bounded fresh native artifact compatibility and old/new Harbor timing evaluation, ignored diagnostic artifacts only, no production changes; outcome=assigned; commit=none
