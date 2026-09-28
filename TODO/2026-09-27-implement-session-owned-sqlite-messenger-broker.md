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
Status: TODO
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
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
