# Reusable PHP SQLite broker MVP and Hatfield integration

Status: agreed scope, with failure-policy decisions still open. Planning only. No broker, adapter, or benchmark has been implemented.

This revision replaces the earlier Hatfield-internal, Doctrine-first broker proposal. The user chose a separate reusable package, an async SQLite client with Revolt, and a benchmark owned by that package. Delayed delivery is mandatory for MVP.

Tracked work:
- [Broker and integration task](../TODO/2026-09-27-implement-session-owned-sqlite-messenger-broker.md)
- [Standalone benchmark task](../TODO/2026-09-27-benchmark-messenger-sqlite-against-php-queue-broker.md)

## Purpose and boundaries

The package provides a durable PHP queue broker backed by SQLite. One database owner serves publishers and consumers over persistent sockets. The goal is to replace competing SQLite writers and repeated consumer polling with serialized queue operations and notifications. SQLite commits, request scheduling, and IPC still cost time. Performance must be measured, not inferred from using an event loop.

The package must operate and benchmark without Hatfield. Hatfield becomes one consumer through a thin Symfony Messenger transport adapter. Messenger retains routing, handlers, serialization, middleware, and explicit handler-failure retries. This replaces the Doctrine transport, not Messenger itself.

```text
Standalone broker package
  SQLite queue engine inspired by goqite and sqliteq
  Revolt event loop and persistent socket server
  PHP client
  One broker-owned async SQLite connection
    Persistence child when required by the selected client
  Symfony Messenger transport adapter
  Package-owned bench/ runner

Hatfield integration
  Controller supervision and readiness
  Session database and socket paths
  Messenger transport configuration
  Runtime failure handling and owned-tree cleanup
```

Use goqite as the primary design reference. Evaluate sqliteq's delivery-generation checks and small benchmark runner as specific references. Do not port their entire feature suites or assume maturity alone establishes correctness.

The broker receives database and socket configuration, not Hatfield session objects, tool messages, or transcript types. Its storage schema is not required to retain Doctrine's schema. Doctrine Messenger is a comparison backend, not a required production storage layer for the new broker.

## MVP scope

### Required queue behavior

- One message table holds all named queues. Creating a queue name does not require a schema migration.
- Durable send and atomic receive. A reservation belongs to only one consumer at a time.
- ACK and terminal reject using delivery-specific receipts. Stale or foreign receipts cannot delete a newer claim.
- Opaque body bytes and headers, without a requirement to decode application payloads as JSON.
- Delayed delivery, including Messenger `DelayStamp` compatibility, persisted availability, and due-message wakeups after restart.
- An explicit visibility and redelivery policy that supports Hatfield's conservative recovery behavior. Automatic redelivery is not silently inherited from either reference library.

### Required broker behavior

- Persistent socket server and client using existing Amp stream/socket and Revolt facilities.
- One database writer. All publishers and consumers use the broker, rather than opening their own async SQLite connections.
- Complete storage operations remain serialized across asynchronous waits. Another request must not interleave statements inside a reservation transaction.
- Bounded frames, pending requests, and client output buffers. Slow clients cannot cause unbounded memory use or retain a database transaction while awaiting a socket write.
- Readiness notifications with bounded, cancellable waiting. Registering a waiter must not miss a publication that races an empty receive.
- Confirmation after commit for send, claim, ACK, and reject. Document known failure versus unknown outcome after disconnect.
- Defined startup, shutdown, and restart behavior, including ownership of the SQLite persistence process and exclusion of a second broker writer.
- A thin Messenger transport adapter and a standalone package benchmark.

### Deferred features

Priorities, batch send, batch receive, dead-letter administration, detailed queue stats, and multiple SQLite drivers are outside MVP. Typed PHP annotations may describe an existing API, but do not require a separate typed-message framework. No custom journal, full in-memory queue mirror, alternate backend framework, or autonomous job runner is required.

Delays are not deferred. A broker that only supports immediate messages does not satisfy the replacement requirement.

## Delayed-delivery contract

The adapter must preserve the requested delay at Messenger's millisecond API boundary. Do not truncate a positive subsecond delay to immediate delivery. Store the resulting availability time durably, rather than keeping only a process-local timer or reapplying the original delay after restart.

A message is not eligible before its availability time. The broker must wake waiting consumers when it becomes eligible, without requiring another publication or a broker restart. Inserting an earlier delayed deadline must update the wake-up schedule. A notification is a hint to receive, not ownership of a message.

After restart, rebuild wake-up scheduling from persisted availability. A message already due can be received immediately. A future message retains its original deadline. Delayed messages must not block otherwise-ready messages or another queue.

Proof must cover:
- Immediate and positive subsecond `DelayStamp` mapping through the real adapter.
- No early receive, followed by eligibility at the deadline.
- Consumer notification when a delayed message becomes due on an otherwise idle queue.
- An earlier deadline arriving while another delayed wake-up is scheduled.
- Restart both before and after the persisted deadline, without restarting the delay interval.
- Queue isolation and interaction with claimed messages.

Use controlled clocks and deterministic events at the lowest suitable layer, plus a bounded real restart proof. Do not add arbitrary sleeps or production test hooks. Keep timing measurements separate from correctness assertions.

## Redelivery and failure decisions still required

These decisions must be settled before implementing the relevant behavior:

1. **Visibility expiry.** A healthy but slow consumer can outlive a lease. The generic policy must distinguish that risk from explicit handler failure and support Hatfield's conservative treatment of uncertain external effects. The current Hatfield no-keepalive and long-redelivery-horizon behavior is the integration baseline. Do not introduce short leases or automatic recovery as an incidental transport change.
2. **Uncertain publication or reservation.** A lost reply does not mean the operation failed. The smallest proposed policy fails visibly without transparently replaying an uncertain request. If automatic replay is required, define durable deduplication and retention after ACK. An in-memory request-ID map is insufficient.
3. **Broker failure during handler execution.** Specify the generic connection/receipt contract and the separate Hatfield supervisor response. Restarting the broker must not make stale receipts valid or silently repeat effects through the controller's resume path.
4. **Existing Hatfield queues.** Decide evaluation-only sessions versus an approved migration before replacing the live backend. Do not ignore old rows, reset claims, weaken delays, or let old and new controllers compete during cutover.

Delivery receipts protect queue mutations, not exactly-once external effects. Messenger owns explicit failure retries and backoff. Broker reject must not independently enqueue a second retry.

Dead-letter administration is deferred, but failures cannot silently discard messages. The MVP still needs documented error reporting and retained-state behavior under its chosen delivery policy.

## Async SQLite and source references

The selected direction uses an async SQLite client from the first broker version. `fabpot/amphp-sqlite3` is the evaluated candidate. At inspected revision `1ee168273e29af037a5c4576349ff89e3a53b5fd`, it starts a dedicated process per connection and requires `ext-sqlite3` and `amphp/parallel`. Count this child in lifecycle and benchmark costs. Do not use a connection pool that recreates multiple competing writers.

Its Amp SQL API is not Doctrine DBAL, but the separate queue engine no longer requires a DBAL adapter. Native and PHAR compatibility still need proof for Hatfield integration. Its documented writable-file defaults use WAL and `synchronous=NORMAL`. Select and verify an explicit durability policy for equivalent benchmark comparisons.

The inspected client does not support data-changing statements with `RETURNING`. Therefore sqliteq's exact `UPDATE ... RETURNING` cannot be copied unchanged into that client. Confirm the chosen versions and an atomic supported claim operation before implementing. Do not silently change clients or relax claim safety to bypass the limitation.

References:
- [goqite](https://github.com/maragudk/goqite): queue operations, SQLite design, and visibility semantics to inspect before adaptation.
- [sqliteq implementation at 202d9c7](https://github.com/minnzen/sqliteq/blob/202d9c7e864950bdb0966a5b3d25f22f0c7e229b/src/queue.ts): conditional delete/extend using the receive generation.
- [sqliteq benchmark at 202d9c7](https://github.com/minnzen/sqliteq/blob/202d9c7e864950bdb0966a5b3d25f22f0c7e229b/bench/run.ts): small standalone runner. Its direct single-process timings are not broker latency predictions.
- [SQLite agent-queue article](https://dev.to/minnzen/building-a-durable-message-queue-on-sqlite-for-ai-agent-orchestration-335m): shared distribution motivation. Atomic SQL still acquires SQLite's writer lock.
- [Async SQLite README at 1ee1682](https://github.com/fabpot/amp-sqlite3/blob/1ee168273e29af037a5c4576349ff89e3a53b5fd/README.md), [requirements](https://github.com/fabpot/amp-sqlite3/blob/1ee168273e29af037a5c4576349ff89e3a53b5fd/composer.json), and [connector](https://github.com/fabpot/amp-sqlite3/blob/1ee168273e29af037a5c4576349ff89e3a53b5fd/src/SqliteConnector.php).
- [TuxWeb/amp-sql-queue at c0c64c4](https://github.com/TuxWeb/amp-sql-queue/tree/c0c64c4ef363c9e594de6944327d9495d6676eaf): API/waiting reference, not a selected dependency. Its SQL worker, lease, and retry policies are not a substitute for the required broker contracts.

Review upstream licenses and retain required attribution for adapted code. Pin source revisions when adaptation begins. Repository/package naming and the public API are not finalized by this plan.

## Package-owned benchmark

The benchmark lives in the new package's `bench/` directory and runs without Hatfield installed. Keep it a small standalone runner, not a general benchmark framework. Symfony Messenger and Doctrine transport may be benchmark-only dependencies. No Hatfield classes, custom SQLite factory, session storage, controller fixtures, tool messages, or Castor runtime are prerequisites for the package runner.

Primary comparison:
- Symfony Messenger with its standard Doctrine SQLite transport.
- Symfony Messenger with the new broker's real transport adapter.

Use the same portable workload and envelope serialization for both. A native broker-client microbenchmark may be reported separately, but must not be presented as an equivalent comparison against the full Messenger path. Record the baseline's polling behavior and the candidate's notification behavior rather than disabling either design's intended mechanism.

The initial suite covers send/receive/ACK, concurrent publishers and consumers, backlog across named queues, and idle consumer wake-up. Include delayed wake-up lateness measured from eligibility, not from initial send. These are generic queue workloads, not Hatfield tool journeys. Benchmark no batch or priority API while those features are deferred.

Report publish confirmation, handler-entry delivery, and ACK latency with p50, p95, p99, maximum, sample counts, and throughput. Include failures, unfinished work, and owned-tree CPU, memory, and process counts. Record raw samples and exact versions/configuration. Count socket and async-persistence IPC costs.

Fairness requirements:
- File-backed SQLite on the same storage, with verified equivalent journal/synchronous durability and commit-before-confirmation semantics.
- Fixed payloads, worker counts, warmup, and workload schedule, with repeated paired runs and alternating or seeded-random order.
- Separate cold startup from warm behavior. Retain outliers and failures. Report per-run variation and scheduling lag so overload cannot hide behind reduced offered load.
- Check IDs, payloads, and ACK outcomes. Use enough samples to support reported tail percentiles, or label the evidence inconclusive.
- Define the comparison method from baseline variation before candidate tuning. No weaker durability, asymmetric batching, or favorable-run selection.

Acceptance requires the benchmark and reproducible evidence. A performance-win claim requires repeatable concurrent-delivery tail-latency improvement beyond baseline variation. Neutral, regressing, or inconclusive results pause default adoption for review. Do not turn machine-dependent timing limits into flaky unit tests.

The benchmark justifies the package independently: what does a single-writer broker improve over clients accessing SQLite directly, and what does it cost? Hatfield end-to-end measurements can later provide separate integration evidence. They are not package benchmark prerequisites.

## Hatfield integration

Hatfield owns one broker per controller-root session and one queue database inside that session's configured storage directory. Child-agent work sharing the controller pools shares that broker. A different controller-root session has a different broker/database. Keep application `state.sqlite`, canonical `events.jsonl`, worker counts, MCP ownership, Scheduler behavior, and single-owner `run_control` unchanged.

Resolve session paths through existing facilities. Verify that the canonical session identity exists before startup; do not let unidentified controllers share a fallback namespace. Use a private short runtime socket path and check platform path-length limits. Broker ownership must exclude a surviving earlier writer before removing stale sockets or opening the database.

Start the broker and wait for positive readiness before launching consumers and advertising `runtime.ready`. Keep it available while consumers and shutdown publishers finish. Supervise its persistence child as part of the owned tree. Preserve the database across normal shutdown/resume and delete it only through the approved inactive-session lifecycle.

The adapter preserves configured serializers, headers, stamps, terminal reject semantics, and delayed delivery. Coordinate its readiness wait with Messenger's idle sleep so notification is not followed by an unnecessary sleep remainder. Preserve cancellation and worker limits without creating a busy loop.

Use existing Symfony Process/Lock, runtime executable resolution, and project cleanup facilities. Prove source, PHAR, and native startup/teardown. Broker schema initialization belongs to the package's sole database owner, not to Hatfield consumers or the old Doctrine migration executor.

## Implementation sequence and proof

1. Finalize the portable delivery/protocol contract, durability policy, async-client compatibility, and package location. Record unresolved product decisions before assigning implementation.
2. Establish the package-local benchmark baseline against standard Messenger SQLite. Do not invent a mock broker API to complete the comparison early.
3. Build the queue engine, socket broker/client, delayed scheduling, and Messenger adapter in the independent package. All MVP requirements above are required before declaring it a replacement candidate.
4. Prove atomic claims, commit ordering, stale receipts, binary payloads, bounded I/O, disconnect/restart handling, and the delayed-delivery contract. Use real SQLite for persistence claims and deterministic clocks/barriers where appropriate. Benchmark results do not replace failure tests.
5. Run the real candidate comparison. Report the result without promising a speedup in advance.
6. Integrate the package into Hatfield only under the approved failure and existing-data cutover policy. Verify lifecycle, routing, package artifacts, and configured Messenger behavior.

The package's tests and benchmark cannot depend on Hatfield's test kernel or helpers. Hatfield integration tests must use the existing test container, directory/process isolation, and Castor facilities. Read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before that work. Keep ordinary correctness cases deterministic and within the project's duration budget. Long measurements belong in an explicit bounded benchmark run, not a timing-window PHPUnit assertion.

Independent review must check the agreed scope, transport mapping, transaction ownership across async waits, failure policy, delay persistence/wakeups, and benchmark fairness. For tracked Hatfield work, the CODE-REVIEW transition owns `castor check`; post-merge validation is separate. No production LLM or Harbor run is required to benchmark the standalone package.

Main owns contract clarification and initial repository routing. Any later implementation fork needs a bounded finalized scope and explicit ownership handoff. No implementation owner or fork is assigned by this planning update.

## Hatfield source map and related work

Initial source investigation used agent-core `8490bee7200887f58c099befd1d13081e7d01634`. Recheck current source before implementation.

- `src/CodingAgent/Runtime/Controller/HeadlessController.php`: session ownership, startup, readiness, and shutdown.
- `src/CodingAgent/Runtime/Controller/ConsumerSupervisor.php`: consumer lifecycle, restart policy, and idle settings.
- `src/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClient.php`: session DSNs, controller restart, and resume.
- `src/CodingAgent/Runtime/Process/RuntimeProcessConfig.php`: source/PHAR/native executable resolution.
- `src/CodingAgent/Session/HatfieldSessionStore.php`: configured session paths and lifecycle.
- `config/packages/messenger.yaml`, `config/packages/doctrine.yaml`, and `config/services.yaml`: routing, serializers, old transport DB, and migrations.
- `src/CodingAgent/Infrastructure/Doctrine/SqlitePollingConnection.php` and `SqliteImmediateTransactionMiddleware.php`: Hatfield's optimized baseline for later integration evidence, not standalone package dependencies.
- `vendor/symfony/messenger/Worker.php`: idle-event/sleep and adapter contract to verify against the selected version.
- `depfile.yaml`: authoritative Hatfield integration boundaries. No queue engine in AgentCore, TUI, or ExtensionApi.
- `docs/session-storage.md`, `docs/async-runtime-architecture.md`, and packaging docs: update with approved integration behavior.

PR #533 and DONE task `2026-09-25-fix-current-tool-latency-and-parallel-batch-feedback` provide historical timing evidence. The prior 107ms send included 103.6ms in SQLite transaction acquisition, but that sample preceded later optimizations and is not the standalone benchmark baseline.

Future Symfony concurrency work and `2026-09-05-revisit-safe-automatic-recovery-for-abandoned-messenger-deliveries` remain separate. This plan does not authorize a Symfony 8.2 migration, short-lease recovery, or lease renewal in Hatfield.
