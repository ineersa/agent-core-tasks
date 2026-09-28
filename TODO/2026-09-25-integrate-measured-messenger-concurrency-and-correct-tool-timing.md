# Integrate measured Messenger concurrency and correct tool timing

## Goal
## Goal

Reduce Hatfield runtime overhead and improve tool responsiveness without weakening ordering, cancellation, streaming, recovery, or process ownership. Evaluate Symfony 8.2 Messenger's process-pool concurrency against the existing multi-consumer topology at equal concurrency. Correct the mismatch between live tool elapsed time and completed execution duration.

Investigation requested by the user on 2026-09-25. Implementation has not started. This task authorizes planning and a bounded integration evaluation, not unreviewed product decisions or automatic rollout of a prerelease dependency stack.

## Investigation and baseline

Read [the investigation report](../reports/2026-09-25-messenger-worker-performance-investigation.md) before implementation. Source baseline is `fb4096b6537883246b93c0f46b71b7575690d391`. Recheck current source and package versions when starting.

Upstream announcement: https://symfony.com/blog/new-in-symfony-8-2-faster-messenger-workers
Concurrency PR: https://github.com/symfony/symfony/pull/63650

Confirmed findings:

- Locked Messenger and FrameworkBundle are 8.1.5. Doctrine Messenger is 8.1.4. `amphp/parallel` is absent. Symfony 8.2 is under development as of the report date.
- `--concurrency` uses an Amp child pool. On the inspected NTS CLI without ext-parallel, that means processes, not threads. Four consumers becoming one parent plus four children is not a process-count reduction.
- Default topology is 13 consumers plus controller and TUI. LLM and generic tool pools each contain four consumers. Ordinary workers poll idle queues every 10 ms.
- Native generic tool batches already admit multiple eligible messages. Sequential barriers are intentional. Read and bash tools are parallel. Write, edit, and code_mode are sequential.
- MCP calls route to one owner process and execute synchronously. Calls inside code_mode also execute synchronously and bypass outer Messenger batch dispatch. Do not claim this task makes either parallel.
- Ordinary tool start events are created before batch admission and queue consumption. The live timer includes those waits, while final duration_ms usually measures only executor-side work. The display switches from milliseconds to seconds by design, with a one-second live tick.
- Upstream DispatchTask requires its console entry point to return a ContainerProviderInterface or supported Runtime closure. Hatfield bin/console runs its application and exits. This bootstrap mismatch must be addressed before adding the concurrency flag.
- Installed Symfony already supports --fetch-size and Doctrine multi-fetch. Fetch batching is not execution concurrency and can hurt fairness for long jobs.

## Decisions to finalize before affected implementation

These are product decisions, not questions to outsource to a benchmark:

1. Choose live timer presentation. Recommendation: queued work is not labelled as executing; show whole seconds for live execution and milliseconds only for completed subsecond calls. Decide whether queue wait should be visible separately. Preserve deferred-subagent lifecycle elapsed time.
2. Decide whether prerelease Symfony packages are acceptable for production. An isolated pinned dev-branch evaluation is distinct from shipping it.
3. Resolve runtime.llm_worker_count semantics if changing process topology. It currently means fixed consumer processes. Do not silently reinterpret it as concurrent messages or add speculative compatibility settings.

Until resolved, evaluation can gather evidence, but must not introduce the affected user-visible behavior or public settings.

## Scope and sequence

### 1. Establish current timing and resource evidence

Use existing tracing, structured logs, runtime events, and test helpers. Identify batch admission, queue wait, worker execution, result commit, and visible delivery as separate intervals. Reuse existing identifiers: run_id, session_id, tool_call_id, step_id, transport, and attempt where applicable. Add only the minimal production observability required for this goal. Do not log raw arguments, prompts, results, credentials, or unsanitized exceptions.

Record effective merged settings, source revision, package references, PHP build, cold and warm startup, CPU, PSS, RSS, process counts, idle queue polling or query counts, per-call phase times, batch wall time, and time to first visible LLM token. The report contains the workload matrix. Establish baseline variability before choosing an acceptance threshold.

### 2. Correct tool timing semantics

Trace and update the existing event, projection, and widget path. Do not mark unadmitted or queued calls as executing. Keep live and final intervals consistent with their documented labels. Preserve run_control as canonical state owner rather than appending new canonical events directly from execution workers.

Audit normal completion, error, cancellation, human-input suspension, direct shell, background handoff, and deferred subagent completion. Deferred subagent elapsed time measures the child lifecycle and must not become the short duration of deferred registration.

Keep the approved display change separate from runtime performance claims. A less distracting label is not a speedup.

### 3. Resolve the upstream dependency and bootstrap gates

In the task worktree only, resolve the smallest coherent Symfony 8.2 development package set and add amphp/parallel:^2.0. Record exact resolved versions and source references. Inspect dependency changes and coordinate with dependency-upgrade tasks below. Do not patch vendor or copy Symfony internal worker implementations.

Use the supported framework bootstrap contract. Preserve early --cwd handling, APP_ENV and APP_DEBUG, source and PHAR autoloading, default command dispatch, and /reload. Verify FrameworkBundle's configured console path and the application returned to execution children.

Prove Amp child executable discovery and bundled worker resources in source, PHAR, and supported static distribution. Source-only success does not establish distribution support.

### 4. Evaluate one pool at a time

Trial the generic tool pool first: one queue consumer with four execution children versus the existing four independent consumers. Keep the same tool admission cap and execution modes. Keep run_control, mcp, agent admission, scheduler, and extension_agent unchanged.

Verify child output routing before expanding to LLM workers. Preserve JSONL frames, per-run identity, committed event ordering, transient sequence zero, error visibility, and output backpressure behavior. Then evaluate LLM and compaction workers at the same effective concurrency if the tool trial passes.

Preserve container resets, Doctrine failure recovery, cancellation lookups, middleware routing, envelope serialization, result delivery, and existing failure subscribers. Extend process ownership through grandchildren and tool subprocesses. Prove shutdown, crash, memory recycling, restart exhaustion, and /reload behavior.

### 5. Decide from equal-concurrency evidence

Compare the existing pool, timing-only correction, tool pool candidate, and any eligible LLM pool candidate. Smaller fixed pools are a resource trade-off control, not proof of equal-throughput improvement.

Adopt only if correctness passes and measurements show lower resources or lower latency without a material regression in the other objective. Record sample counts, warmup, environment, baseline variation, and medians and meaningful tails. Do not copy the blog's 7.5x claim into Hatfield targets.

If the new topology fails a correctness, distribution, or performance gate, retain the existing workers, remove trial-only production branches and dependencies, and append a supported no-go result to the report. The timing correction may ship independently after its own review. Do not keep speculative dual backends.

## Explicit exclusions and safety constraints

- No RabbitMQ migration, ZTS requirement, thread backend, custom Messenger worker, global async rewrite, cross-session daemon, autoscaler, or speculative public API.
- No parallelization of writes, edits, code_mode, MCP connections, run_control, or extension jobs in this task.
- Do not reintroduce SIGALRM keepalive, short elapsed-time leases, automatic abandoned-delivery reclaim, or exactly-once external side-effect claims. Preserve redeliver_timeout=315360000 and explicit repair policy.
- Do not signal or restart root-owned workers or any process carrying HATFIELD_SESSION_ID. Measurements and tests own isolated processes and deterministic teardown.
- No broad test refactors or removal of meaningful safety coverage.
- The earlier single-process hybrid async LLM plan is a separate design, not what --concurrency implements.

## Entry points

- `src/CodingAgent/Runtime/Controller/HeadlessController.php`
- `src/CodingAgent/Runtime/Controller/ConsumerSupervisor.php`
- `src/CodingAgent/Runtime/Controller/ConsumerStdoutPoller.php`
- `src/CodingAgent/Runtime/Stream/StdoutRuntimeEventSink.php`
- `src/CodingAgent/Runtime/Messenger/*FailedEventSubscriber.php`
- `bin/console`, `composer.json`, `composer.lock`, `config/packages/messenger.yaml`
- `src/AgentCore/Application/Pipeline/LlmStepResultHandler.php`
- `src/AgentCore/Application/Pipeline/ToolCallResultHandler.php`
- `src/AgentCore/Application/Handler/ToolBatchCollector.php`
- `src/AgentCore/Application/Handler/ToolExecutor.php`
- `src/CodingAgent/Runtime/Protocol/RuntimeEventTranslator.php`
- `src/CodingAgent/Runtime/ProjectionPipeline/ToolProjectionSubscriber.php`
- `src/Tui/Transcript/ToolDurationHeaderWidget.php`
- `src/CodingAgent/Mcp/Client/McpConnectionManager.php` and `src/CodingAgent/Tool/CodeMode/CodeModeHostBridge.php` as unchanged serialization controls

## Validation

Read the testing skill, tests/AGENTS.md, and applicable module instructions before implementation or test review. All QA uses Castor.

Extend clock-controlled virtual/projection coverage for queue versus execution timing, live formatting, terminal freezing, and deferred lifecycle duration. Extend existing batch tests for caps, sequential barriers, result ordering, and duplicate-result handling.

Prove real eligible-tool overlap with deterministic barriers: both handlers enter before either is released. Assert exactly one completion per call and ordered model results. Do not use fixed sleeps or wall-time races as correctness proof.

Prove real process streaming, cancellation, crash behavior, reset, teardown, and packaged boot through existing controller and artifact test facilities. Test a slow sibling alongside fast siblings and preserve independent result visibility. Run focused live LLM validation only when provider-visible execution changes.

Focused starting commands:

- `castor test --filter=ToolDurationHeaderWidgetTest`
- `castor test --filter=ToolBatchCollector`
- `castor test --filter=ToolExecutorTest`
- `castor test:controller-replay`
- Appropriate focused `castor test:tui` and `castor test:llm-real` cases for changed contracts

Cases must complete within ten seconds under relevant load. Concurrency or contention defects require relevant concurrent Castor lanes. No retry-until-green, timeout inflation, leaked children, or active-session process cleanup.

The CODE-REVIEW transition owns the full castor check gate. Run targeted checks during implementation, independent review before completion, and integrated post-merge validation separately.

## Dependencies and related work

Coordinate with these board tasks without folding their full scope into this task:

- `2026-09-10-update-composer-dependencies-and-migrate-to-current-symfony-ai-and-mc.md`
- `2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter.md`
- `2026-09-25-stop-reusing-idle-codex-websockets.md`, active provider transport work that can affect benchmark results
- `2026-09-05-revisit-safe-automatic-recovery-for-abandoned-messenger-deliveries.md`, deferred policy that must remain unchanged
- `add-proper-mcp-tool-call-cancellation.md` and `2026-09-25-evaluate-symfony-ai-mcptoolbox-adoption.md`, separate MCP ownership and concurrency prerequisites
- `refactor-transcript-tool-metadata-into-typed-presentation-models.md`, coordinate timer metadata edits without requiring a broad refactor

Earlier design: `.pi/plans/hybrid-async-llm-runtime-plan.md` and its companion work plan. Its single-process provider concurrency is an alternative if the process pool misses the memory goal.

## Tentative ownership

Main owns baseline selection, timing semantics, product decisions, report updates, integration order, and task transitions. After the initial current-source routing pass, a bounded fork may own the upstream pool/bootstrap/process-lifecycle slice because it requires substantial isolated process and artifact validation. Define acceptance and validation before handoff. Use sequential write ownership in one task worktree, or separate worktrees with an explicit integration order. Independent read-only review is required. No implementation ownership has yet been assigned.

## Acceptance criteria
- Baseline and equal-concurrency comparison are recorded with exact revisions, dependency references, effective settings, phase timing, CPU, PSS, RSS, process counts, workload, and sample counts. Performance hypotheses are not reported as measured improvements.
- Queued tool work is not falsely reported as executing, and live/final timing follows a user-approved documented presentation. Deferred subagent lifecycle duration and all terminal paths remain correct.
- The Symfony 8.2 trial uses a pinned coherent dependency set and the supported application bootstrap, with source, PHAR, and supported static distribution evidence or an explicit no-go blocker.
- Eligible generic tools demonstrably overlap under deterministic process barriers while sequential policies, admission caps, per-call completion, and ordered model history remain intact. MCP and code_mode are not advertised as newly parallel.
- Any adopted pool preserves JSONL streaming, cancellation, Doctrine/reset/failure behavior, result delivery, memory recycling, repair policy, and deterministic ownership and teardown of all descendants.
- Adoption is supported by equal-concurrency resource and latency evidence. A failed candidate is removed and documented rather than retained as an unsupported fallback.
- Required targeted Castor proofs, independent review, CODE-REVIEW castor check gate, and post-merge validation pass for any submitted implementation. This TODO creation does not claim those checks have run.

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
- Created: 2026-09-25T14:21:53+00:00

## Task workflow update - 2026-09-25T14:29:13+00:00
- Summary: Completed source-based investigation and created the linked report. Independent read-only review verified the worker topology, timer mismatch, serial MCP/code_mode paths, dependency baseline, and upstream bootstrap incompatibility. Report citation corrections applied. No implementation, dependency changes, benchmarks, or QA runs performed.
- Review: agent_59f085aba22910f1 approved with suggestions. The generated task has a cosmetic duplicate Goal heading because create_task supplies one and the submitted body also included one. Task tools expose no body-replacement operation; left unchanged rather than editing the task file manually.
- Validation targets to extend: ToolDurationHeaderWidgetTest, ToolBatchCollectorTest, ToolBatchCollectorDurableTest, ToolExecutorTest, and existing controller/artifact facilities. Read the report for the full measurement and proof contract.

## Task workflow update - 2026-09-25T20:31:38+00:00
- Summary: User reprioritized on 2026-09-25: diagnose and fix the existing runtime before changing worker architecture. The current-runtime baseline, tool timing correction, and read+bash overlap investigation now belong exclusively to 2026-09-25-fix-current-tool-latency-and-parallel-batch-feedback. This supersedes implementation steps 1 and 2 here and the earlier immediate prerelease-evaluation sequence.
- Scope and dependency update: defer Symfony 8.2/Amp integration until the current-runtime task is complete and Symfony 8.2 reaches stable release. Reassess need from measured remaining bottlenecks; migration is not automatic. Do not implement duplicate timing or baseline work in this task.
- Retain this task's upstream bootstrap, streaming, lifecycle, distribution, and equal-concurrency evaluation details for a later go/no-go decision. No status transition or implementation has occurred.

## Task workflow update - 2026-09-25T22:29:01+00:00
- Summary: Current-runtime follow-up found native generic tool overlap and modest admission-to-worker waits in one Harbor workload. The visible timer fix is formatting-only: whole seconds live, milliseconds for fast terminal calls. Ordinary terminal timing already uses lifecycle timestamps; the earlier report claim of an executor-only switch was corrected. No worker speedup, resource savings, or need for a Symfony pool replacement has been demonstrated. Keep upstream integration deferred pending stable8.2 and measured remaining bottlenecks.
- Evidence and limitations: reports/2026-09-25-messenger-worker-performance-investigation.md, measured current-runtime follow-up. Timer change83f09b1c4 remains in task-start with full gate/review pending.
