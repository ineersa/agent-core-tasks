# Messenger concurrency and Hatfield runtime performance

Date: 2026-09-25. Status: source investigation followed by a bounded Harbor run and current-runtime display fix. Symfony pool integration remains deferred.

Hatfield baseline: `fb4096b6537883246b93c0f46b71b7575690d391` in `/home/ineersa/projects/agent-core`.

## Updated priority after user follow-up

The user chose to address current worker pickup, tool latency, and apparent serialization before replacing consumers. The [current-runtime task](../IN-PROGRESS/2026-09-25-fix-current-tool-latency-and-parallel-batch-feedback.md) owns that work on Symfony 8.1. The upstream integration task is deferred until these findings are available and Symfony 8.2 is stable. This supersedes the immediate development-branch evaluation sequence below. A later migration depends on measured remaining bottlenecks, not the release date alone.

The existing 10 ms idle polling interval is not a guarantee of 10 ms pickup. Measure admission, transport wait, actual execution, result commit, and TUI delivery separately. A live label changing from milliseconds to seconds is formatting behavior, not proof of serialization.

## Recommendation

Evaluate Symfony 8.2 Messenger concurrency in an isolated integration branch, but do not present it as a way to replace eight PHP workers with two PHP processes. Its default backend still boots a PHP application in each child process. It reduces the number of queue consumers, not necessarily the number of application processes or their aggregate memory.

Correction after tracing the protocol and measuring a real run: ordinary tool timers start before batch admission and queue consumption, but their completed labels already use the same lifecycle interval. Worker execution duration is nested under result details and does not reach the top-level protocol duration field. The original claim that ordinary completion replaces lifecycle elapsed time with executor-only time was incorrect.

The investigation also found real serialization. MCP calls share one consumer, and nested `code_mode` calls execute synchronously. Adding concurrency to the generic tool queue will not accelerate either path. Native parallel tool calls already use multiple Messenger workers.

The integration task should have a measured adoption gate. Compare the existing four-consumer pool against one Symfony parent with four execution children, not against one serial consumer. Adopt the new topology only if it preserves behavior and improves the measured Hatfield workload. Keep the existing topology if the candidate only adds overhead.

## Evidence and limits

The original investigation was source-only. The measured follow-up below adds one completed Harbor workload and deterministic local tests. It is not a reproduction of a slow TUI session or a pool resource benchmark. No dependency or worker-topology changes were made.

Main inspected the worker launcher, queues, CLI bootstrap, timing path, scheduler, and installed dependency sources. A researcher inspected upstream changes. A read-only scout traced tool execution and MCP ownership. Both main and the scout read the testing skill and `tests/AGENTS.md` before test-related analysis.

Findings below distinguish source-confirmed behavior from compatibility risks and unmeasured performance hypotheses. The user's perceived lack of parallelism is not evidence that all generic tool calls are serial.

Local source citations use paths and line ranges at the baseline above. Upstream branch links are moving references inspected on the report date. GitHub's PR API reports that [Symfony PR 63650][concurrency-pr] merged into `8.2` on 2026-08-14, at `02b2550d2eefb2c72c7a17d8e65d342d3913d421`. The branch-head API returned `172e4ad1a5fb88cce1494b3dd4f73b22637f0ac0`; the opened branch pages were not independently verified byte-for-byte against that SHA. Pin and record the resolved Composer source references in the integration branch.

## Measured current-runtime follow-up

The user authorized at most two Harbor attempts. The first failed before model execution because the private OAuth copy omitted the required `refresh` field. The second used a validated access-only copy with `refresh: ""`, completed normally, and made 16 tool calls. Source credentials were unchanged and private copies were deleted. No third attempt was run.

- Task: `swe-bench/mwaskom__seaborn-3069`, trial `mwaskom__seaborn-3069__H3TMmRh`.
- Application artifact: commit `1f6f753a9f7b553c497db7f7b1f9342126565879`, SHA-256 `e5405700da56b6d500c9362abd9f6f3b29b425d339d31e48c33f02916a65e135`.
- Settings: four LLM workers and four generic tool workers, plain WebSocket, GPT-6 Luna/medium, no child agents or extensions, no model retries. These differ from the historical serial benchmark configuration.
- Result: `run.completed`, reward `0.0`, 58.04 seconds agent execution, 115.8 seconds wall time. A completed controller run does not imply the issue was solved.
- Calls: ten bash, four read, two edit. Admission-to-worker-start ranged from 13.7 to 74.1 ms, median 20.2 ms. Read execution ranged from 2.4 to 9.7 ms, bash from 11.6 to 281.9 ms, edit from 6.5 to 11.1 ms.
- Native bash and three-read batches overlapped. Some very short parallel-admitted batches did not overlap in their measured worker intervals. That is not proof of a serial dispatch path.
- All 16 terminal tool protocol events lacked top-level `duration_ms`; canonical result details retained executor durations. Projection derives the final header duration from `started_at` and `ended_at`.
- Cleanup found no surviving owned processes or matching containers. A retained-artifact scan found no copied access token.

Evidence is under `/home/ineersa/projects/hatfield-harbor/jobs/diag-tool-latency-20260925/run-seaborn-3069-attempt2/`, including sanitized `TIMING-ANALYSIS.json`. The original prepared task digest was `sha256:866e82c22b999b2758decdd49bd0fe903aaed14efeee77ffcdd1fdfbedf6c7c4`. The run is headless, so it does not measure TUI rendering delay. Worker spans and admission timestamps also do not separate every transport and result-delivery phase.

Deterministic controller replay now uses two FIFO-barrier-controlled native bash calls. Both must enter before release. The earlier unblocked read-plus-bash timing assertion was removed because it was a scheduling race. Real read overlap remains observational evidence from Harbor, not the deterministic regression contract.

The user selected whole seconds for live headers and milliseconds for terminal calls below one second. The implementation changes only that formatting condition. A clock-controlled mounted transcript proves the initial live header is `0s`, a 375 ms completion freezes as `375ms`, independent long-running timers still tick, and success, failure, cancellation, and replay stay frozen correctly. Targeted unit and virtual validation passed with 120 tests and 549 assertions. The new live-format test first failed against the old implementation.

This is a presentation correction, not a measured execution speedup. No before-and-after worker latency improvement is claimed. Full QA and independent review remain pending in the task-to-pr phase. The evidence does not justify replacing the worker pool.

## What Symfony adds

### Concurrent execution uses a child pool

The [Symfony announcement][blog] introduces `messenger:consume async --concurrency=4` and requires `amphp/parallel`. The parent fetches messages and delegates execution to child contexts. Amp uses processes by default. Threads require PHP built with ZTS and the `parallel` extension. Hatfield's inspected CLI is NTS and has no `parallel` extension.

The upstream [execution strategy][strategy] coordinates the pool. [DispatchTask][dispatch] boots the application in each child, dispatches through the message bus, and returns results or sanitized failures over a channel. Handler completion and transport acknowledgement remain separate responsibilities. This is not cooperative execution of several LLM requests inside one PHP application instance.

The reported 7.5x result compares concurrency eight with a single worker handling 20 ms messages. Hatfield already runs multiple workers. That benchmark does not predict a 7.5x improvement here. Fetching and unserializing also remain parent-side work.

The blog's batch-handler exception means Symfony `BatchHandlerInterface` handlers, whose batch state must stay with one child. Hatfield's `ToolBatchCollector` is an application-level admission and result coordinator, not such a handler. A literal search found no `BatchHandlerInterface` or `BatchHandlerTrait` use under `src/`. The shared word "batch" does not make the blog's exception apply to every Hatfield tool batch.

### AMQP changes do not apply to the current transport

[PR 65924][amqp-prefetch] adds broker-driven AMQP prefetch. [PR 65168][amqp-delay] rounds AMQP delays upward to reduce delay-queue proliferation. Hatfield uses Doctrine transports backed by SQLite, not RabbitMQ. Neither change directly improves today's queues.

Switching to RabbitMQ solely to obtain the blog's transport benchmark would add a service and change deployment, ordering, and redelivery behavior. That is outside this integration.

### Logging can help, but does not measure end-to-end latency

The optional [logging middleware][logging] records downstream bus processing time and memory change. It distinguishes a message sent to transport from a handled message. Middleware position determines which work it includes.

Its `memory_usage` is the difference between two `memory_get_usage()` readings. It is not peak memory, process RSS, aggregate pool memory, or PSS. Its `duration_ms` is not queue residence time or time until the TUI renders a result. Logs alone cannot establish TUI delivery latency.

Hatfield already has tool and LLM tracing and structured logs. Reuse those facilities, add missing phase correlation only where needed, and avoid duplicate high-volume logs. If enabling upstream logging, audit the exception context against Hatfield's privacy rules. The middleware includes an exception object on failure and does not automatically add Hatfield's run and session correlation fields.

## Availability now

The [Symfony release page][releases] lists 8.1 as stable and 8.2 as under development. Stable minor releases normally arrive in May and November. There is no released 8.2 adoption claim in this report.

The local `composer.lock` contains:

| Package | Locked version |
|---|---|
| `symfony/messenger` | 8.1.5 |
| `symfony/framework-bundle` | 8.1.5 |
| `symfony/doctrine-messenger` | 8.1.4 |
| `symfony/http-client` | 8.1.5 |
| `amphp/process` | 2.1.0 |
| `revolt/event-loop` | 1.0.9 |
| `amphp/parallel` | Absent |

`composer.json` requires PHP >=8.5, Messenger and FrameworkBundle `^8.1`, and Doctrine Messenger `^8.0`. It already permits development stability and prefers stable releases. Existing Amp WebSocket and process dependencies do not provide the parallel worker package.

A trial is possible now with explicitly selected 8.2 development packages and `amphp/parallel:^2.0` in a task worktree. A normal lock-preserving install cannot add the feature. Resolve the smallest coherent Symfony package update, record exact versions and source references, and inspect the dependency diff. No Composer solve was run during this investigation, so compatibility of the full package set remains unproven. Do not copy the new worker into Hatfield or patch vendor files as a substitute.

The installed 8.1 command already supports `--fetch-size`, and the installed Doctrine connection implements `get(int $fetchSize = 1)` with `setMaxResults($fetchSize)`. That is available without 8.2. It batches claims, not handler execution. Raising it for long LLM or tool jobs can let one serial consumer claim work that other consumers could have handled. Keep the current value of one as the baseline. Evaluate larger values only for a demonstrated queue-fetch bottleneck, initially on short messages, with fairness and abandoned-claim checks.

## Hatfield's current execution path

### Fixed pools and single-owner queues

`HeadlessController.php:165-204` launches these consumers at startup:

| Role | Default processes | Ownership |
|---|---:|---|
| `run_control` | 1 | Run state, canonical transitions, result collection |
| `llm` | 4 | Ordinary LLM calls and compaction |
| `tool` | 4 | Generic tools and direct shell work |
| `agent` | 1 | Subagent admission and deferred orchestration |
| `mcp` | 1 | MCP connections and synchronous calls |
| `scheduler_default` | 1 | Recurring maintenance |
| `extension_agent` | 1 | Extension-owned agent jobs |

That is 13 consumers, plus the controller and TUI, before shell children, MCP servers, or provider helpers. These are source defaults, not a measurement of the user's effective merged settings. Parent and child runs share the pools.

`runtime.llm_worker_count` controls the fixed LLM process count, default four, range one to eight. `tools.execution.max_parallelism` defaults to four and determines the normal generic tool worker count and batch admission limit. `agents.max_agents` is another limit, not a replacement for either worker count.

Sources: `src/CodingAgent/Config/RuntimeConfig.php:16-37`, `config/hatfield.defaults.yaml:140,350-357`, and `config/packages/messenger.yaml:54-133`.

Replacing each four-worker pool with one parent and four process children gives five processes per pool when fully populated. Replacing both pools gives 15 consumers or execution children, plus controller and TUI, rather than the current total of 15 application processes. Pool allocation may affect idle counts, which must be measured. This arithmetic is not a memory forecast.

### Polling has a cost, but its size is unmeasured

`ConsumerSupervisor.php:61-68,110-135` passes `--sleep=0.01` and `--memory-limit=256M` to every consumer. There is no `--concurrency`, explicit `--fetch-size`, or `--keepalive`. `ConsumerStdoutPoller.php:41-50` also polls child output every 10 ms.

An idle nonblocking queue consumer can therefore attempt roughly 100 polls per second before query and loop overhead. Do not multiply that number by every process and report it as measured SQL throughput. Scheduler behavior and actual transport work differ.

One queue-fetching parent per pool could reduce idle database polling and competing claims. It might also add IPC, another booted application, and a central dispatch bottleneck. These are competing hypotheses. Measure CPU, database operations, queue delay, process count, and aggregate PSS together.

The 256 MiB setting is a graceful Messenger recycle threshold, not a hard aggregate pool memory limit. Preserve and verify recycling when execution moves into children.

### Parallel admission does not guarantee parallel execution on every route

For ordinary model calls, the path is:

1. `ToolCallExtractor` produces individual calls in model order.
2. `LlmStepResultHandler` resolves each call's registered execution mode.
3. `ToolBatchCollector` admits calls under the batch cap and sequential barriers.
4. `StepDispatcher` dispatches messages through `agent.execution.bus`.
5. Routing middleware selects `tool`, `agent`, or `mcp`.
6. An available consumer executes the call and sends its result to `run_control`.

`ToolBatchCollector.php:527-572` admits parallel calls up to the cap. A sequential or interrupt call waits until in-flight work is empty, then runs alone. The scheduler does not skip that call to admit later parallel calls. That preserves ordering rather than maximizing throughput at any cost.

Examples of registered policy are `ReadFileTool.php:99` and `BashTool.php:293`, both parallel, and `WriteFileTool.php:86`, `EditFileTool.php:76`, and `CodeModeTool.php:53`, all sequential. Do not make writes or edits parallel merely to improve a benchmark.

`ToolCallResultHandler.php:222-267` emits an execution-end event for each accepted result. It waits for batch completion before adding ordered results to model history. An ordered transcript or a next LLM turn that waits for the slowest sibling is not proof that execution was serial.

The Codex request factory defaults `parallel_tool_calls=true`. That permits the model to request multiple tools; it does not bypass Hatfield's execution modes or queue ownership. No literal `multi_tool_use.parallel` handler was found in the application. Native multi-tool responses become separate calls. A wrapper shown by a provider or harness must be traced at its own boundary rather than assumed to be a Hatfield scheduler.

### Two confirmed serial paths

MCP-backed `ExecuteToolCall` messages are routed to the single `mcp` consumer. `McpConnectionManager.php:12-21` documents one client per run and server in that broker process. `callTool()` at lines 220-256 blocks on that client. Independent MCP calls therefore serialize at this boundary even if the outer batch admits them together.

Adding MCP workers would duplicate connection ownership, including session-scoped STDIO server processes. The existing boundary exposes no asynchronous request handle. Any MCP concurrency work needs a separate design that preserves one owner while allowing supported request multiplexing. Coordinate that work with the existing MCP cancellation and McpToolbox evaluation tasks.

`code_mode` is also sequential. `CodeModeHostBridge.php:224-285` builds a sequential nested context and invokes the toolbox directly for each IPC request. Several `tool()` calls in one script do not become several Messenger messages. Batching them saves model round trips, not tool wall time through parallel execution. This integration must not silently change that contract.

## Why the timer feels wrong

There are two separate issues.

### The unit change is intentional formatting

`ToolDurationHeaderWidget.php:36-39,58-73` renders immediately, then updates once per second while streaming. Values below 1000 ms use milliseconds. Later values use whole seconds, then minutes and seconds. A first frame such as `37ms` followed by `1s` follows directly from this code. It does not prove execution slowed down.

The user subsequently approved whole seconds for the live timer and milliseconds only for terminal subsecond calls. The current-runtime fix implements that choice. No separate queued label or worker-entry event was introduced.

### Lifecycle timing differs from worker execution timing

For ordinary model-issued tool calls:

- `LlmStepResultHandler.php:367-405` creates `ToolExecutionStart` for every effect before post-commit batch registration and dispatch. This includes siblings not yet admitted.
- `RuntimeEventTranslator.php:262-283` exposes the canonical event timestamp as `started_at`.
- `ToolProjectionSubscriber.php:184-199` creates a streaming `Running…` result block.
- `ToolDurationHeaderWidget.php:58-64` computes live elapsed time from that timestamp.
- `ToolExecutor.php:139-197,369-390` measures execution inside the worker and attaches nested `result.details.duration_ms`.
- `RuntimeEventTranslator.php:338-340` forwards only a top-level duration, not that nested field.
- `ToolProjectionSubscriber.php:410-427` derives ordinary final elapsed time from the canonical start and end timestamps when the protocol carries no explicit duration. Its explicit-duration path remains unchanged.

The executor uses its injected clock. `nowMicros()` at lines 615-621 converts a clock instant into epoch microseconds; this is not evidence of a monotonic duration clock.

The live value includes admission and queue wait, execution, and time until the visible terminal update. The ordinary final value includes admission and queue wait through the canonical terminal timestamp. TUI delivery delay after that timestamp is outside the frozen final interval. The original illustration of a queued two-second timer becoming an executor-only `40ms` was not observed and was based on an incorrect reading of the translator.

The protocol instructions previously said that live timers start at execution. They now state that ordinary starts precede admission and include queue time. The timer is total lifecycle elapsed time, not evidence that a worker has already entered the tool handler.

Deferred subagents have a different intended interval. Their progress factory derives child and aggregate elapsed time from launch and terminal lifecycle timestamps. See `DeferredSubagentBatchProgressSnapshotFactory.php:343-383`. That time includes orchestration. Preserve it rather than replacing it with the short executor time needed to register a deferred batch. Audit direct shell, cancellation, failure, human-input suspension, and background handoff independently before unifying labels.

## Compatibility gates for Symfony concurrency

### The console bootstrap is a concrete incompatibility

Upstream `DispatchTask::bootstrap()` requires the configured console file and expects a returned `ContainerProviderInterface`, or a Runtime closure that resolves to one. It then obtains `messenger.routable_message_bus` from that application.

Hatfield's `bin/console:177-216` constructs its application, calls `run()`, handles the `/reload` loop, and exits. It does not return an application to its includer. Appending `--concurrency=4` after a dependency upgrade is therefore not a complete integration.

Use the framework's supported application-return/bootstrap contract. Preserve early `--cwd` resolution, source and PHAR autoloading, environment initialization, default command behavior, and `/reload`. Verify how the upgraded FrameworkBundle supplies the console entry-point path. Do not add an independent custom kernel bootstrap or depend on upstream internal execution classes as Hatfield APIs.

### Child stdout is part of Hatfield's protocol

Current consumers each have an output pipe tracked by `ConsumerSupervisor`. Both transient LLM output and committed runtime events use those pipes. `StdoutRuntimeEventSink.php:62-87` writes JSONL to `php://stdout`; `ConsumerStdoutPoller` parses and forwards it.

With a new parent and grandchildren, verify where Amp routes output. The reviewed upstream bootstrap buffer is not proof of complete stdout forwarding. If output is merged, verify JSONL frame integrity for concurrent large frames, backpressure, per-run identity, and transient sequence zero. Do not assume pipe writes of arbitrary size are atomic. Lost deltas or interleaved lines would make apparent latency worse even if handlers finish sooner.

### Process ownership must extend through the pool

`ConsumerSupervisor.php:264-337` signals its directly tracked consumers, waits against a shared grace deadline, and escalates survivors. A pool adds grandchildren and a second lifecycle manager.

Verify normal shutdown, parent death, child death, cancellation during I/O, memory recycling, restart exhaustion, `/reload`, and child subprocess cleanup. A Messenger retry after a child crash must not be described as exactly-once external execution. Audit `LlmWorkerFailedEventSubscriber`, `WorkerFailedEventSubscriber`, and related failure subscribers in their new process context.

Hatfield deliberately runs without SIGALRM keepalive and uses `redeliver_timeout=315360000` for session Doctrine queues. Sources are `ConsumerSupervisor.php:33-35` and `src/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClient.php:595-603`. The existing follow-up task defers automatic abandoned-delivery recovery. Preserve this policy. Do not reintroduce short leases or automatic replay of side effects as part of a performance change.

### Serialization and service reset need real application proof

Execution children receive serialized envelopes and return results or failures. Verify Hatfield's actual message classes, stamps, run context, cancellation lookup, locks, and result messages. Test service reset between messages and Doctrine recovery after failure. Upstream service reset support does not prove Hatfield's application state is safe.

A single fetching parent reduces receiver connections for that pool. Execution children can still open application database connections and send results to the transport. Do not promise one database connection for the entire pool.

### Distributed artifacts are a separate gate

Amp process workers need a usable PHP executable and their worker/bootstrap files. Hatfield distributes source execution, PHAR, and static binaries. The source investigation does not establish PHAR inclusion paths, Amp child executable discovery, or PHP-micro behavior. The integration must prove supported artifacts, not only `bin/console` in a developer checkout. ZTS threads are not the initial target.

## Integration sequence and decision rules

The [companion TODO task](../TODO/2026-09-25-integrate-measured-messenger-concurrency-and-correct-tool-timing.md) is the actionable implementation plan. The intended sequence keeps unrelated failures separate:

1. Establish phase-level timing and baseline resource measurements on the existing topology. Resolve the live timer presentation decision before implementing user-visible changes.
2. Correct the timer's queue-versus-execution semantics with the smallest existing runtime and projection mechanisms. Preserve single-owner canonical event writes and deferred lifecycle timing.
3. Evaluate a locked Symfony 8.2 development dependency set in a task worktree, including the supported console bootstrap and Amp process backend.
4. Trial the generic tool pool first at unchanged concurrency. Keep run control, MCP, agent admission, scheduler, and extension jobs unchanged. Expand to LLM workers only after output streaming and process lifecycle pass.
5. Compare resource use and user-visible latency at equal concurrency. Adopt only the proven improvement. Otherwise retain the existing pools and record the result.

No new public setting is required by this report. `runtime.llm_worker_count` currently means processes. Do not silently redefine it as concurrent messages. Finalize a replacement or revised documented contract before a production topology change. Do not leave two speculative execution backends or compatibility aliases in production.

The older [hybrid async LLM plan](../../agent-core/.pi/plans/hybrid-async-llm-runtime-plan.md) remains a different option: overlap provider requests inside one execution process to reduce repeated application state. Symfony's new process pool does not implement that design. Reuse its provider and acknowledgement analysis if the pool fails the memory objective. Do not combine both migrations in one revision.

## Measurement and validation contract

Performance measurement belongs in the integration task, using Castor and isolated test-owned processes. It must not target active Hatfield workers.

Compare the following at equal effective concurrency:

| Candidate | Comparison purpose |
|---|---|
| Existing fixed pools | Current baseline |
| Same code with timing correction | Separate display correction from actual performance |
| One 8.2 tool consumer with four children | Queue polling and generic tool throughput |
| LLM pool candidate after tool gates | Provider streaming and resource cost |
| Smaller existing fixed pools | Resource trade-off control, not an equal-throughput claim |

Record warm and cold startup separately. Exercise independent generic tools, mixed sequential and parallel calls, a slow sibling with fast siblings, simultaneous child-agent LLM work, MCP batches, and ordinary `code_mode`. The latter two are controls expected to remain serial. Use barriers or explicit readiness events, not artificial sleeps, to prove overlap.

Report batch wall time, per-call execution time, admission wait, transport wait, result-to-display delay, LLM time to first visible token, completed operations per second, CPU, total PSS, RSS, process counts, and idle polling or query counts. Distinguish application heap from process memory. Report sample count, medians and tail values, environment, effective settings, workload, and exact revision. Do not report p95 from an inadequate sample or turn noisy wall-time assertions into regression tests.

A performance win is lower resource use or lower latency on the equal-concurrency workload without a material regression in the other objective. Define acceptable measurement variance from repeated baseline observations before evaluating the candidate. Do not invent a percentage target from the upstream blog.

Correctness proof belongs at the lowest useful layer:

- Clock-controlled virtual TUI and projection tests for live versus final timing, queued state, terminal freezing, and deferred progress.
- Existing collector and durable-state integration tests for cap, sequential barriers, ordering, and duplicate-result handling.
- Deterministic process/controller proof that two eligible tool handlers enter before either is released, followed by one completion per call and preserved model order.
- Real process lifecycle and streaming proof for the parent-child topology, including artifact boot and teardown.
- Focused live LLM tests only when provider-visible flow changes. A replay pass alone does not settle a live streaming or cancellation hang.

Targeted commands include `castor test --filter=ToolDurationHeaderWidgetTest`, `castor test --filter=ToolBatchCollector`, `castor test --filter=ToolExecutorTest`, `castor test:controller-replay`, and the focused TUI or live lanes required by the changed behavior. Extend existing test files where possible. Individual cases must stay within ten seconds under normal relevant load, without retry-until-green or timeout inflation.

For tracked implementation, `move_task(to="CODE-REVIEW")` owns the full `castor check` gate. Run targeted checks during implementation rather than running that gate twice. The integration requires independent review and post-merge validation. Concurrency defects need the relevant concurrent Castor lanes, not just a solo green run.

## Open decisions and remaining evidence

- Choose live timer presentation and whether to expose queue wait separately. The requirement that queued work must not be reported as executing is the recommended correctness contract.
- Decide whether a pinned prerelease Symfony stack is acceptable for shipping, rather than for an isolated evaluation only.
- Resolve worker-count setting semantics if process topology changes.
- Measure whether queue polling, execution, provider latency, database contention, or display delivery dominates real slow sessions. This investigation identifies mechanisms, not their measured contribution.
- Prove Amp stdout forwarding, grandchildren teardown, PHAR/static support, and the full Composer solve.
- Keep MCP multiplexing as separately scoped work with its connection-ownership and cancellation decisions. Do not promise the generic worker migration solves MCP latency.

## Source index

The main local evidence is in:

- `composer.json`, `composer.lock`, and installed `vendor/symfony/messenger/Command/ConsumeMessagesCommand.php:81,313-317`.
- `vendor/symfony/doctrine-messenger/Transport/Connection.php:161-168` and `DoctrineReceiver.php:44-51`.
- `src/CodingAgent/Runtime/Controller/HeadlessController.php:165-204` and `ConsumerSupervisor.php:61-68,110-162,264-337`.
- `config/packages/messenger.yaml` and `config/hatfield.defaults.yaml`.
- `src/AgentCore/Application/Pipeline/LlmStepResultHandler.php:248-275,367-446` and `ToolCallResultHandler.php:222-267`.
- `src/AgentCore/Application/Handler/ToolBatchCollector.php:527-572` and `ToolExecutor.php:139-197,369-390,615-621`.
- `src/CodingAgent/Runtime/Protocol/RuntimeEventTranslator.php:262-350`, `Runtime/ProjectionPipeline/ToolProjectionSubscriber.php:152-199,410-427`, and `src/Tui/Transcript/ToolDurationHeaderWidget.php`.
- `src/CodingAgent/Tool/CodeModeTool.php:47-63`, `Tool/CodeMode/CodeModeHostBridge.php:224-285`, and `Mcp/Client/McpConnectionManager.php:12-21,220-256`.
- `src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Progress/DeferredSubagentBatchProgressSnapshotFactory.php:343-383`.
- `tests/Tui/Transcript/ToolDurationHeaderWidgetTest.php`, `tests/AgentCore/Application/Handler/ToolBatchCollectorTest.php`, `ToolBatchCollectorDurableTest.php`, and `ToolExecutorTest.php`.

[blog]: https://symfony.com/blog/new-in-symfony-8-2-faster-messenger-workers
[releases]: https://symfony.com/releases
[concurrency-pr]: https://github.com/symfony/symfony/pull/63650
[strategy]: https://github.com/symfony/symfony/blob/8.2/src/Symfony/Component/Messenger/Execution/ParallelExecutionStrategy.php
[dispatch]: https://github.com/symfony/symfony/blob/8.2/src/Symfony/Component/Messenger/Execution/DispatchTask.php
[logging]: https://github.com/symfony/symfony/blob/8.2/src/Symfony/Component/Messenger/Middleware/LoggingMiddleware.php
[amqp-prefetch]: https://github.com/symfony/symfony/pull/65924
[amqp-delay]: https://github.com/symfony/symfony/pull/65168
