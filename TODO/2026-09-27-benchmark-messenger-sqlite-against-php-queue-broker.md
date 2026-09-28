# Benchmark Messenger SQLite against the reusable PHP queue broker

## Goal
## Goal

Implement a repeatable acceptance benchmark comparing current Hatfield Messenger SQLite transport with the independent PHP SQLite broker through its real Messenger adapter. Measure the whole transport path, including serialization, socket traffic, broker scheduling, and the async SQLite persistence process. Do not substitute a mock broker or compare raw broker SQL with a full Messenger baseline.

This is a required deliverable for `2026-09-27-implement-session-owned-sqlite-messenger-broker`. The broker and its adapter do not exist yet. Baseline measurement can precede them, but the candidate comparison cannot be marked complete until it exercises the actual broker. No broker API or repository name is prescribed by this task.

## Agreed architecture

The latest user decision supersedes the original parent task's Hatfield-internal, Doctrine-first broker design. Build a reusable PHP broker as a separate project, taking design inspiration from goqite and sqliteq, with Revolt and an async SQLite client. Hatfield owns supervision, session paths, and Messenger wiring. Queue/failure contracts still require explicit decisions. This benchmark must count the broker's persistence child and all client processes in resource costs.

## Comparison

Required baseline: the current merged Hatfield Doctrine transport, including its empty-poll optimization, BEGIN IMMEDIATE, and current worker idle settings. Pin the exact revision and effective configuration when measurements start.

Required candidate: the independent broker accessed through the real Symfony Messenger transport adapter, using the same envelopes, serializers, middleware, handler work, and publisher/consumer concurrency. Use the candidate's intended notification behavior, not an artificial shared sleep interval that disables its advantage.

For independent-session workloads, also run a test-only per-session direct-Doctrine control. This distinguishes database partitioning gains from broker gains. Keep controls in the benchmark, not product fallback paths. A polling-only broker variant is optional diagnostic evidence, not a required second product mode.

## Workloads

- One publisher and one consumer for a low-load reference.
- Four concurrent publishers or result senders feeding one consumer, matching the serialized run_control bottleneck.
- One dispatcher, four tool consumers, and one result consumer with no application work in handlers. Measure a complete four-message dispatch/result round trip.
- Ready-but-idle consumers followed by a burst, including idle queues that share the transport database. Use actual configured Hatfield consumer topology for the representative lane.
- A bounded prefilled backlog and concurrent independent sessions.
- Small envelopes and representative large serialized tool results. Fix the sizes, counts, worker topology, warmup, and run order in the benchmark manifest before evaluating the candidate.

Use predetermined arrival schedules as well as completion-paced batches where appropriate. Report scheduling lag and unfinished requests so a slower backend does not reduce offered load and hide queueing. Rate generation is benchmark workload scheduling, not a test synchronization sleep.

## Measurements

Record correlated monotonic timestamps and report p50, p95, p99, maximum, sample count, and throughput per workload and repetition:

- Publish invocation to durable confirmation.
- Publish invocation to handler entry, including pickup and waiting behind other work.
- ACK invocation to confirmed completion.
- Four-message dispatch to receipt of all four results.

Publish confirmation can reach the sender after a consumer has already received a message. Do not calculate confirmation-to-handler as a nonnegative queue-wait duration or clip negative values. Only claim a commit-to-pickup breakdown when both timestamps are actually observed.

Use same-host clock comparability where verified. Otherwise compute durations within one process or use request round trips rather than subtracting unrelated clocks.

Record total CPU time, idle CPU rate, peak memory, and process count for the owned process tree, including the async SQLite worker. Distinguish transport round-trip time from TUI rendering and LLM generation, which this benchmark does not measure.

## Fairness and evidence

- Use file-backed databases on the same filesystem/device, matching journal and synchronous durability policies. Verify effective PRAGMAs rather than assuming defaults. Include commit-before-confirmation behavior in the contract checks.
- Record PHP, SQLite and library versions, source revisions, payload sizes, offered load, actual completions, concurrency, polling/wakeup settings, database layout, checkpoint policy, warmup, and machine/storage information.
- Keep application work, serialization, middleware, payload generation, and diagnostic logging equivalent. Do not weaken durability, batch only one backend, remove fsync, or use in-memory storage to produce a win.
- Separate cold startup from warmed steady-state results. Repeat paired runs with alternating or seeded-random backend order. Preserve every scheduled run, including failures; never retry until green or select only favorable runs.
- Export raw samples and a machine-readable summary plus a readable comparison. Report individual runs and variation, not only pooled percentiles. Label p99 estimates with insufficient samples as inconclusive.
- Verify message IDs, payload integrity, and expected delivery/ACK counts. Report failures, losses, unexpected duplicate deliveries, and unfinished operations alongside timing. Keep fault-injection recovery correctness tests separate from performance measurements.

## Acceptance interpretation

The benchmark is a deliverable and adoption gate, not an ordinary timing-sensitive unit test. Its correctness and report-generation tests belong in normal QA; long performance measurements are an explicit bounded Castor lane.

Before candidate tuning, use baseline variation to fix the run budget, sufficient sample count, and comparison method. Require repeatable improvement beyond baseline variation in the primary four-result fan-in and batch-delivery tail latency to claim a performance win. Report other workload regressions and resource costs. If evidence is neutral, regresses, or remains noisy, report that outcome and stop default adoption for review. Do not invent an absolute latency promise or change the success criterion after seeing candidate results.

## Implementation and validation constraints

Reuse existing Castor, Symfony Process, test-container, isolated-directory, and owned-process teardown facilities. Strip inherited live-session DSNs, isolate app and transport databases, wait on positive readiness, and never touch live session or root-owned processes. Every run has a hard bound, fails with diagnostics, and tears down only its owned process tree.

Read the testing skill and tests/AGENTS.md before implementation. Main should own the initial benchmark contract and source routing. No implementation fork or task transition is part of this planning update.

## Acceptance criteria
- A reproducible Castor entry point executes the same Messenger workload against the actual merged SQLite transport and the real broker adapter. A baseline-only run is explicitly labeled incomplete for comparison; a missing candidate must not count as success.
- Workloads include single-client, four-to-one result fan-in, four-tool dispatch/result round trip, idle pickup with competing idle consumers, representative payloads/backlog, and independent sessions. Multi-session results include a per-session direct-Doctrine control.
- Effective durability, runtime versions, payloads, concurrency, offered load, polling/wakeup settings, warmup, and backend revisions are recorded. Both backends include real commits, adapter/serializer cost, and all relevant subprocesses.
- Reports include raw correlated samples, per-run p50/p95/p99/max and sample counts, throughput, failures and unfinished work, plus owned-tree CPU/memory/process counts. No LLM or physical-render latency claim is inferred from transport timing.
- Predetermined repeated paired runs and sufficient samples support the tail-latency comparison. Scheduling lag, overload, failures and outliers are retained; unknown timestamp boundaries and inadequate p99 sample sizes are labeled rather than invented.
- All expected message IDs, bodies and ACK outcomes are checked. Isolation, readiness, bounds, teardown, metric calculations, and report generation have deterministic focused checks using project facilities.
- The parent broker task cannot claim a performance improvement without repeatable gains beyond baseline variation for the primary fan-in/batch-delivery tail latency at equivalent durability. Neutral/regressing/inconclusive results block default adoption pending review rather than failing a flaky per-commit timing assertion.

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
- Created: 2026-09-27T19:01:31+00:00

## Task workflow update - 2026-09-27T19:11:23+00:00
- Summary: Corrected benchmark ownership and acceptance to the user-approved standalone package runner. Removed Hatfield as a required dependency or baseline implementation. Added delayed wake-up measurement consistent with mandatory delayed delivery in the MVP. No benchmark code or measurements yet.
- Current benchmark scope supersedes conflicting original requirements for a Hatfield Castor entry, optimized Hatfield transport, run_control/tool topology, test-container bootstrap, or per-session Hatfield controls. Canonical plan: reports/2026-09-27-session-owned-sqlite-messenger-broker-plan.md, Package-owned benchmark section.
- The runner lives in the new broker package bench/ directory and runs without Hatfield installed. Keep it small like sqliteq bench/run.ts. Standard Symfony Messenger and Doctrine transport may be benchmark-only dependencies; compare standard Doctrine SQLite with the actual broker Messenger adapter using identical portable envelopes/serialization. Do not introduce Hatfield classes, session paths or controller fixtures.
- Initial suite: send/receive/ACK; concurrent publishers and consumers; backlog across named queues; idle wakeup; delayed-message wake-up lateness measured from the persisted eligibility deadline, not initial send. Do not require batch or priority APIs while those features are deferred. Hatfield end-to-end benchmarks may be separate later integration evidence.
- Keep equivalent file-backed durability, all socket/persistence-process costs, raw samples, repeated paired runs, p50/p95/p99/max with counts, throughput, failures/unfinished operations, and owned-tree CPU/memory/process counts. A native-client microbenchmark is separate evidence, not an equivalent comparison with the full Messenger baseline.
- Acceptance: reproducible package-local command and report; message integrity and lifecycle checks; adequate samples and declared baseline-variation comparison method. A speedup claim needs repeatable concurrent-delivery tail gains at equivalent durability. Neutral, regressing or inconclusive evidence pauses adoption for review. Machine-dependent performance assertions do not belong in ordinary unit tests.
- Benchmark implementation remains pending the actual package/client/adapter. A baseline-only result must be labeled incomplete for the A/B comparison, not represented as proof of the nonexistent candidate.
