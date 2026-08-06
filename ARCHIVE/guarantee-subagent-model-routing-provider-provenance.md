# Guarantee subagent model routing and provider provenance

## Goal
In session 33 a subagent surfaced `Codex WebSocket send timeout` even though scouts are configured for `deepseek/deepseek-v4-flash`, and the visible model/usage line showed DeepSeek. The current evidence is insufficient to know whether the timeout came from the child, the parent run, an inherited default, or misleading TUI attribution. Treat this as a routing/provenance bug: correlate each child launch and transport error to the exact child run, requested model, resolved provider, and transport.

Do not paper over the issue by disabling cached Codex transport. The cached transport hardening remains valid; this task must ensure an explicitly configured DeepSeek scout cannot silently instantiate or inherit Codex unless a documented fallback was explicitly selected.

Relevant areas include the subagents extension launch configuration, per-agent model overrides, child process environment/arguments, model resolver/provider factory, tool result/error envelopes, usage aggregation, and parent TUI attribution.

## Acceptance criteria
- Trace the session-33 incident to a specific parent or child run using stable correlation IDs, and record whether the Codex timeout was emitted by the parent transport or a child transport.
- An explicit scout model override such as `deepseek/deepseek-v4-flash` is carried end-to-end into the child model resolver; it cannot silently fall back to the parent/default Codex provider.
- If model resolution or provider construction disagrees with the requested child model, fail before provider I/O with an actionable structured error rather than using another provider implicitly.
- Record privacy-safe per-child provenance (`child_run_id`, agent name, requested model, resolved model/provider/transport) in lifecycle diagnostics and result metadata without prompts, tool output, credentials, or environment values.
- The parent TUI must not present one child's model/usage as though it were the parent or another child; attribution must be explicit wherever subagent usage is surfaced.
- Add an integration regression with a Codex-default parent and DeepSeek-configured scout proving only the DeepSeek client path is selected for the child; add a separate attribution proof for a provider error.
- Keep `websocket-cached` enabled and preserve its existing continuation behavior.
- Run Castor-only focused routing/provider tests, `castor test:llm-real` when the live provider path is touched, and full `castor check` before CODE-REVIEW.

## Workflow metadata
Status: ARCHIVE
Branch: task/guarantee-subagent-model-routing-provider-provenance
Worktree: /home/ineersa/projects/agent-core-worktrees/guarantee-subagent-model-routing-provider-provenance
Fork run: aumxw2whb1ll
PR URL: https://github.com/ineersa/agent-core/pull/318
PR Status: merged
Started: 2026-07-23T20:35:53.997Z
Completed: 2026-07-24T19:50:21.123Z

## Work log
- Created: 2026-07-23T18:52:18.410Z

## Task workflow update - 2026-07-23T19:15:46.857Z
- Summary: Exact root cause confirmed. Failed child `agent_dd2f69d9cb4c075c` launched with `metadata.model=deepseek/deepseek-v4-flash`, but `ExecuteLlmStep` carries no model. On every turn `ExecuteLlmStepWorker` late-resolves by run ID through `ModelSelectionActiveModelResolver`, which calls `resolveInitialModel(explicitModel: null)`. At turn 620 it resolved the current/default `openai-codex/gpt-5.6-sol`; the cached Codex socket then hit the bounded 120s send timeout. The DeepSeek TUI line was accurate launch metadata but false as execution attribution.
- Authoritative app log lines 13021-13023 identify child run 3d451a76-e371-5ece-b9ca-8769167d85e4, turn 620, model openai-codex/gpt-5.6-sol, cache_reused=true, phase=send. Root design defect is missing model identity on the queued LLM command / run execution snapshot.

## Task workflow update - 2026-07-23T19:57:05.231Z
- Summary: P0 / CRITICAL cost-safety defect. Exact fallback is now proven: the child DeepSeek model exists only in AgentCore child `RunMetadata` and `deferred_subagent_child.definition_model`; no `hatfield_session` row exists for the child UUID. `ExecuteLlmStep` omits the model, so the worker calls `resolveInitialModel(null, childUuid)`. `SessionMetadataStore` ultimately casts the UUID `3d451...` to integer `3`, queries normal session ID 3, finds no usable model, then falls through to configured `ai.default_model=openai-codex/gpt-5.6-sol`. If numeric-prefix session 3 had a model, the child could have inherited that unrelated session's model instead. Fix must snapshot/propagate the resolved child model on execution and forbid UUID-to-numeric session fallback.
- Proven DB state: child UUID exists in deferred_subagent_child with definition_model=deepseek/deepseek-v4-flash; no hatfield_session row exists for it; normal session row 3 has model NULL; effective default is openai-codex/gpt-5.6-sol.

## Task workflow update - 2026-07-23T20:08:18.321Z
- Validation: Required future implementation sequence: RED failing regression on current code → GREEN minimal production fix → focused regression suite → full Castor validation. No production-first test backfill is acceptable.
- Summary: User mandates strict RED-GREEN development because this is a P0 cost-safety boundary. Before changing production, add a regression reproducing the exact failure: child definition/run metadata says DeepSeek, child UUID has a numeric prefix colliding with a normal session ID, default is Codex, and the queued/executed LLM step must still use DeepSeek. Record the pre-fix failure. Then make `ExecuteLlmStep` carry all immutable execution identity needed by the worker, including model and session/run identity, and remove the incompatible late fallback. Prove the test fails again if model propagation is removed.

## Task workflow update - 2026-07-23T20:35:53.997Z
- Moved TODO → IN-PROGRESS.
- Created branch task/guarantee-subagent-model-routing-provider-provenance.
- Created worktree /home/ineersa/projects/agent-core-worktrees/guarantee-subagent-model-routing-provider-provenance.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/guarantee-subagent-model-routing-provider-provenance.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/guarantee-subagent-model-routing-provider-provenance.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/guarantee-subagent-model-routing-provider-provenance.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/guarantee-subagent-model-routing-provider-provenance.
- Summary: Started as P0/CRITICAL cost-safety fix. Implementation must use strict RED-GREEN development: first reproduce child DeepSeek metadata drifting through UUID-to-integer session lookup to Codex, then make ExecuteLlmStep self-contained with immutable execution identity and eliminate incompatible fallback. User export was preserved under ~/.hatfield/dumps before worktree creation.

## Task workflow update - 2026-07-23T20:43:36.421Z
- Recorded fork run: l7zgc5tqas48
- Summary: Launched P0 implementation fork l7zgc5tqas48 with mandatory two-commit RED-GREEN workflow. RED must reproduce the exact UUID-prefix collision and prove the platform receives Codex instead of the child DeepSeek model before any production edit. GREEN must resolve at scheduling, carry required model on ExecuteLlmStep, remove worker-time fallback, fail closed for unknown UUIDs/missing child models, preserve normal session model changes, and pass mutation sensitivity.
- 2026-07-23: Design scout confirmed RunMetadata.model is retained in run_started but discarded from RunState; durable child definition model already exists in deferred_subagent_child. Minimal approved design resolves child definition at scheduling through CodingAgent resolver, serializes required model on ExecuteLlmStep, and removes mutable resolution from worker. Avoid broad RunState field/copy changes unless proven necessary.

## Task workflow update - 2026-07-23T20:57:22.393Z
- Summary: Additional cost/performance invariant from user: ExecuteLlmStepWorker must perform zero session/metadata/model-selection reads. The queued command is authoritative and self-contained. Child models are immutable for the child run and should be pinned from launch/run metadata rather than re-resolved through mutable defaults every turn. If scheduling still performs a child-repository read each turn, review whether retaining the already-canonical run-start model in replay state can remove it without introducing duplicate sources. Normal-session model changes may resolve/update at the explicit model-change or scheduling boundary, never inside provider execution.

## Task workflow update - 2026-07-23T21:00:56.159Z
- Recorded fork run: m3fcim8fya5z
- Validation: RED commit eadc59a5e09a165a33714bdd0229fe6640eb91a4: regression failed as expected (DeepSeek expected, unrelated session-3 Codex luna observed).; First GREEN draft: focused tests, controller replay, Deptrac, scoped PHPStan, and CS passed; llm-real and mutation proof remain pending.
- Summary: First fork delivered valid committed RED proof and working GREEN changes but left GREEN dirty, skipped mutation proof and llm-real, and relaxed unknown non-UUID IDs for test fixtures. Launched completion fork m3fcim8fya5z to finish safely: preserve numeric-session/UUID-child domains, fail closed for every other ID instead of retaining test-driven fallback, update replay test IDs to canonical values at the narrow seam, run mutation sensitivity and required Castor validation, commit GREEN, and leave a clean worktree.

## Task workflow update - 2026-07-23T21:25:30.124Z
- Recorded fork run: 4ta2yi6g8gjg
- Validation: `castor test:controller-replay`: PASS, 9 tests / 127 assertions (JUnit artifact).; `castor test:llm-real`: PASS, 13 tests / 175 assertions; report runtime 29.5s.; GREEN commit: bbfd1b1ff2e86d6ea8021d8b2e0a618b0beb975b; worktree clean before acceptance audit.
- Summary: GREEN is now committed as bbfd1b1ff after RED eadc59a5e; worktree is clean. Recovered validation artifacts show controller replay PASS (9/127) and live llm-real PASS (13/175). Because completion handoff text was truncated and GREEN spans canonical run-ID corrections across 32 files, launched final acceptance audit 4ta2yi6g8gjg to independently verify invariants, perform/record mutation sensitivity, flag accidental scope, and leave branch untouched/clean.

## Task workflow update - 2026-07-23T21:27:40.995Z
- Recorded fork run: 4ta2yi6g8gjg
- Validation: Mutation sensitivity: temporarily hard-coded worker Codex model; `castor test --filter=ChildRunModelRoutingProvenanceRegressionTest` failed with DeepSeek expected / Codex actual; restored exact file; rerun PASS (1 test, 8 assertions); final worktree clean.; `castor test:controller-replay`: PASS, 9 tests / 127 assertions.; `castor test:llm-real`: PASS, 13 tests / 175 assertions.; Focused GREEN regression and related ExecuteLlmStep/AdvanceRun/model resolver/message contracts: PASS.; `castor deptrac`: PASS, 0 violations.; Scoped `castor phpstan`: PASS, 0 errors.; `castor cs-check`: PASS.; Full `castor check` intentionally deferred to CODE-REVIEW gate.
- Summary: Task-start implementation accepted. Strict RED→GREEN history is complete and clean: RED eadc59a5e reproduces DeepSeek child drift to Codex through UUID-prefix/session-3 collision; GREEN bbfd1b1ff pins a required model on ExecuteLlmStep, resolves only at scheduling, makes the worker perform zero model/session/default lookups, rejects unknown ID domains, and prevents non-digit session PK casts. Independent audit verdict ACCEPT with no critical issue. Residual rich TUI provenance is intentionally outside this P0 fix.

## Task workflow update - 2026-07-23T22:34:56.317Z
- Summary: CODE-REVIEW reviewer verdict APPROVE WITH SUGGESTIONS; no critical/blocking issues. Reviewer confirmed RED is genuine, worker zero-read invariant, strict run-ID domains, queued-step model immutability, safe dual-source child fallback, necessary 32-file scope, DB/kernel test conventions, and no UUID-to-session cast path. One directly relevant performance suggestion remains: agent_child metadata fallback currently scans run_started metadata twice per scheduled step; address before PR because user explicitly raised unnecessary metadata/state reads.
- Reviewer nonblocking notes deferred: unreachable defensive empty-model guard, pre-existing stale HatfieldSessionStore docblock, repetitive test resolver fakes. These do not affect correctness or PR readiness.

## Task workflow update - 2026-07-23T22:39:29.038Z
- Recorded fork run: l148ub0b2rmc
- Validation: `castor test` at 1069b854e: FAILED, 877 tests / 3766 assertions / 2 errors. Exact failures: Gf05BareAgentsEffectiveContextIntegrationTest nonnumeric root runId; DeferredSubagentBatchLaunchTest missing sixth SubagentChildLaunchInputFactory constructor argument.
- Summary: Task-to-PR full `castor test` exposed two missed test integration errors despite focused/replay/live suites: one effective-context integration fixture still used a UUID as a root session runId, and one deferred launch test manually constructed SubagentChildLaunchInputFactory with the old arity. Launched narrow fork l148ub0b2rmc to update tests to canonical numeric session creation and the required model-selection dependency without weakening production or adding compatibility defaults.

## Task workflow update - 2026-07-23T22:45:52.881Z
- Validation: Reviewer: APPROVED production branch; no critical/blocking issues. Follow-up perf commit re-review: APPROVED. Test-only final commit re-review: APPROVED.; `castor test`: PASS, 4479 tests / 15543 assertions.; `castor deptrac`: PASS, 0 violations / 0 errors.; `castor phpstan`: PASS, 0 errors.; `castor cs-check`: PASS, 0 files fixed.; `castor test:llm-real`: PASS, 13 tests / 175 assertions (31.6s).; llama-proxy warmed; cache stats before CODE-REVIEW gate: 146 entries / 11,500,875 bytes.; Worktree clean at HEAD 2b40b7945.
- Summary: Final task-to-PR review is APPROVED with no blockers after performance follow-up 1069b854e and test-contract alignment 2b40b7945. Full unit/integration suite initially exposed three stale test fixtures; fixed test-only without weakening canonical numeric session enforcement. Worktree clean at 2b40b7945 and ready for deterministic CODE-REVIEW gate.

## Task workflow update - 2026-07-23T22:48:09.135Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (117.1s).
- Pushed task/guarantee-subagent-model-routing-provider-provenance to origin.
- branch 'task/guarantee-subagent-model-routing-provider-provenance' set up to track 'origin/task/guarantee-subagent-model-routing-provider-provenance'.
- Created PR: https://github.com/ineersa/agent-core/pull/318

## Task workflow update - 2026-07-23T22:48:14.205Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/318
- Updated PR Status: open
- Validation: Deterministic `castor check`: PASS (117.1s), including full unit/integration, controller replay, TUI replay, live llm-real, Deptrac, PHPStan, CS, cache guard, artifact integrity, and leak checks.
- Summary: Task-to-PR complete. Deterministic CODE-REVIEW `castor check` passed in 117.1s; branch pushed and PR #318 opened.

## Task workflow update - 2026-07-23T23:03:20.742Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #318 review feedback is valid: current GREEN fixes the incident but couples generic run identity to numeric Hatfield session IDs, creates sessions inside InProcess start when runId is empty, and re-resolves child inheritance from parentRunId/session/default. This is not the durable architecture requested. Reopened for redesign rather than patching the two comments locally. Target direction: preserve authoritative model/session identity in canonical run metadata/state, schedule ExecuteLlmStep directly from that identity, and remove UUID/numeric ID-domain inference from model routing.

## Task workflow update - 2026-07-23T23:15:44.670Z
- Summary: Durable redesign evidence: `run_started` already persists authoritative RunMetadata.model, but RunStateReducer discards it. PR #318 became hacky by reconstructing execution identity from runId/session shape instead of retaining canonical model state. Redesign must make model first-class RunState data, schedule ExecuteLlmStep from state only, snapshot parent model into tool/child launch, and apply `/model` at a safe command boundary as a durable state transition. Revert ID-domain routing, child repository scheduling lookup, parentRunId-as-session lookup, and session creation inside InProcess start. Keep required ExecuteLlmStep.model, worker zero lookups, and HatfieldSessionStore's local numeric guard.
- Architecture scouts traced current `/model`: it only persists session settings/footer and scheduling rereads DB each turn. Proper behavior requires updating canonical run state at the next safe boundary while already-queued ExecuteLlmStep retains its immutable model.
- Parent model must be captured into tool execution and child launch so nested/deferred children inherit the exact model that produced the tool call, not a later session/default lookup. Generic runId remains opaque; parent session relationship must be explicit, never inferred from UUID/digit shape.

## Task workflow update - 2026-07-24T02:12:25.617Z
- Recorded fork run: jlnulmfbdk76
- Summary: User approved durable redesign and requested re-review then CODE-REVIEW. Launched implementation fork jlnulmfbdk76 to replace PR #318's runId-shape/session inference with canonical run metadata/state: model retained from run_started, safe model-change transition, immutable ExecuteLlmStep/ExecuteToolCall model snapshots, durable nested/deferred child inheritance, no session creation inside InProcess start, and no scheduling/worker model lookups. Existing RED proof remains; final diff must remove reviewed hacks and implementation-mirroring resolver tests.

## Task workflow update - 2026-07-24T02:30:32.322Z
- Recorded fork run: iriwuocbk9m0
- Summary: Durable redesign fork jlnulmfbdk76 made substantial uncommitted progress (63 files; canonical RunState model, model-change path, tool/child model snapshots, deferred persistence migration, removal of child scheduling lookup) and reported focused regressions green, but terminated before StartRun fixture alignment, broad validation, and commit. Launched continuation fork iriwuocbk9m0 to preserve/finish the dirty implementation, validate the architecture, run all focused/full Castor suites, commit cleanly, then hand back for reviewer and CODE-REVIEW.

## Task workflow update - 2026-07-24T03:13:31.935Z
- Recorded fork run: cuzqka69goiy
- Summary: Durable redesign committed as 3eff6da61 and independently reviewed: architecture APPROVED WITH SUGGESTIONS; no silent model drift or ID-inference hack remains. Before CODE-REVIEW, fixing one real crash-recovery bug (ToolBatchStateDTO dropped parentModel on persistence), adding focused model-change handler proof, removing a hidden fork-compaction fallback to mutable session/default model, and cleaning three small lifecycle/comment issues. Fork cuzqka69goiy is implementing and validating these review corrections.
- Reviewer verified canonical flow run_started.metadata.model → RunState.model → ExecuteLlmStep.model, worker/scheduler zero lookups, durable /model transition, direct/deferred child snapshot inheritance, local-only numeric session guard, and removal of reviewed hacks.
- Reviewer found ToolBatchStateDTO reconstruction reads parentModel but serialization omitted it; fail-closed prevented wrong-provider execution but deferred resumed child launch could fail instead of inherit. Treating as blocking.

## Task workflow update - 2026-07-24T13:31:22.358Z
- Recorded fork run: aumxw2whb1ll
- Summary: Review-fix fork cuzqka69goiy orphaned without tracked changes and left only an empty accidental file. Relaunched as aumxw2whb1ll with the same narrow blocker list plus artifact cleanup. A fresh reviewer will run immediately after the fix commit; reviewing the unchanged HEAD before fixes would only repeat the existing findings.

## Task workflow update - 2026-07-24T14:51:56.995Z
- Validation: castor test: PASS 4487 tests / 15598 assertions (final HEAD); castor phpstan: PASS, 0 errors (final HEAD); castor cs-check: PASS (final HEAD); castor deptrac: PASS, 0 violations (final HEAD); castor test:tui: PASS 36 tests / 186 assertions (final HEAD); castor test:controller-replay: PASS 9 tests / 127 assertions (HEAD 51b59c139; final commit only logging/state-copy refactor); castor test:llm-real: PASS 13 tests / 175 assertions (HEAD 51b59c139; final commit only logging/state-copy refactor)
- Summary: Final HEAD 08976704679b60946cd5487e54452f23361db551. Independent final reviewer verdict APPROVED with no blocking issues. Canonical model execution identity, zero hot-path mutable lookup, durable /model transition, recovered child model snapshots, fork fail-closed semantics, local session numeric guard, and session-33 regression proof all verified. Final review fixes add structured Ctrl+P degradation logging and use RunState::with() for model transition.

## Task workflow update - 2026-07-24T14:54:19.999Z
- Validation: Failed gate artifact: qa-20260724-145212-81576-4bcc2677; test:tui one startup timeout at TmuxHarness waiting for initial cursor with empty pane capture; controller-replay exit 124 with empty test log.; castor clean:cleanup:workers:list after failure: no stale QA worker candidates.; No surviving process matched failed HATFIELD_QA_RUN_ID.
- Summary: First deterministic CODE-REVIEW gate attempt failed only in runtime lanes under parallel gate load: BashBackgroundAcceptE2eTest tmux pane produced an empty capture and timed out waiting for initial cursor; controller-replay process timed out before producing test output. Same final HEAD had just passed standalone full test:tui 36/186 and prior controller-replay 9/127. Gate leak check/dry-run found no stale QA workers and no processes remained for the QA run ID. Treating as transient startup/contention and retrying deterministic gate without code changes.

## Task workflow update - 2026-07-24T14:56:38.236Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (124.9s).
- Pushed task/guarantee-subagent-model-routing-provider-provenance to origin.
- branch 'task/guarantee-subagent-model-routing-provider-provenance' set up to track 'origin/task/guarantee-subagent-model-routing-provider-provenance'.
- PR already exists: https://github.com/ineersa/agent-core/pull/318
- Validation: Final focused validation: castor test 4487/15598; deptrac 0 violations; phpstan 0 errors; cs-check pass; test:tui 36/186; controller-replay 9/127; llm-real 13/175.; First deterministic gate transient documented; no stale workers or surviving QA-run processes after failure.
- Summary: Durable canonical model identity redesign complete at 08976704679b60946cd5487e54452f23361db551; final independent reviewer APPROVED. Retrying deterministic gate after a documented transient tmux/controller startup timeout with no leaked workers.

## Task workflow update - 2026-07-24T19:50:21.123Z
- Moved CODE-REVIEW → DONE.
- Merged task/guarantee-subagent-model-routing-provider-provenance into integration checkout.
- Auto-merging config/services.yaml
Merge made by the 'ort' strategy.
 config/services.yaml                               |  13 +-
 migrations/Version20260720120000.php               |  20 ++
 migrations/Version20260723230000.php               |  40 ++++
 .../Application/Handler/ExecuteLlmStepWorker.php   |  35 ++--
 .../Application/Handler/ExecuteToolCallWorker.php  |   1 +
 src/AgentCore/Application/Handler/ToolExecutor.php |   1 +
 .../Application/Pipeline/AdvanceRunHandler.php     |  18 ++
 src/AgentCore/Application/Pipeline/AgentRunner.php |   7 +
 .../Application/Pipeline/ApplyCommandHandler.php   |  70 +++++++
 .../Pipeline/ApplyShellCommandHandler.php          |   1 +
 .../Application/Pipeline/CommandMailboxPolicy.php  |   1 +
 .../Application/Pipeline/LlmStepResultHandler.php  |   6 +
 src/AgentCore/Application/Pipeline/RunCommit.php   |   2 +
 .../Application/Pipeline/StartRunHandler.php       |  18 ++
 .../Application/Pipeline/ToolCallResultHandler.php |   3 +
 .../Application/Replay/RunStateReducer.php         |  63 ++++++
 src/AgentCore/Application/Tool/ToolContext.php     |  10 +
 src/AgentCore/Contract/AgentRunnerInterface.php    |   2 +
 .../Compaction/CompactionServiceInterface.php      |   1 +
 .../Compaction/PreLlmCompactionGuardInterface.php  |   1 +
 src/AgentCore/Domain/Command/CoreCommandKind.php   |   2 +
 src/AgentCore/Domain/Event/EventFactory.php        |   1 +
 src/AgentCore/Domain/Event/RunEventTypeEnum.php    |   1 +
 src/AgentCore/Domain/Message/ExecuteLlmStep.php    |  12 ++
 src/AgentCore/Domain/Message/ExecuteToolCall.php   |   2 +
 src/AgentCore/Domain/Run/RunState.php              |  49 +++++
 src/AgentCore/Domain/Tool/ToolBatchStateDTO.php    |   4 +
 .../Launch/DeferredSubagentBatchLaunchPlanDTO.php  |   1 +
 .../Launch/DeferredSubagentBatchLaunchService.php  |   5 +-
 .../DeferredSubagentBatchPreparationService.php    |   6 +
 .../DeferredSubagentBatchProjectionDTO.php         |   1 +
 .../SubagentChildLaunchInputFactory.php            |  22 +-
 .../Execution/SubagentLaunchPreparationService.php |   7 +-
 .../Agent/Fork/ForkChildLaunchInputBuilder.php     |  29 ++-
 .../Agent/Fork/ForkExecutionService.php            |  15 +-
 .../Application/Pipeline/CompactRunHandler.php     |  16 +-
 .../Pipeline/CompactionStepResultHandler.php       |   2 +
 src/CodingAgent/CLI/AgentCommand.php               |   4 +
 .../Compaction/AutoCompactionHookSubscriber.php    |   6 +-
 .../CodingAgentPreLlmCompactionGuard.php           |  13 +-
 src/CodingAgent/Compaction/CompactionService.php   |  12 +-
 .../ModelSelectionActiveModelResolver.php          |   7 +-
 src/CodingAgent/Config/SessionMetadataStore.php    |  11 +
 src/CodingAgent/Entity/DeferredSubagentBatch.php   |   7 +
 .../Entity/DeferredSubagentBatchRepository.php     |   3 +
 .../Migrations/ApplicationMigrationExecutor.php    |   2 +
 src/CodingAgent/Runtime/Contract/UserCommand.php   |   2 +-
 .../CommandHandler/ChangeModelHandler.php          |  69 +++++++
 .../Controller/CommandHandler/StartRunHandler.php  |  33 ++-
 .../InProcess/InProcessAgentSessionClient.php      |  90 ++++----
 .../Messenger/WorkerFailedEventSubscriber.php      |   1 +
 .../Process/JsonlProcessAgentSessionClient.php     |   1 +
 .../Runtime/Protocol/RuntimeEventTranslator.php    |  16 ++
 .../Session/CommittedRunEventAppender.php          |   1 +
 src/CodingAgent/Session/HatfieldSessionStore.php   |  16 +-
 .../Session/Repair/SessionRepairService.php        |   1 +
 .../Replay/SessionRunStateReplayService.php        |   2 +
 src/Tui/Listener/ModelCommandHandler.php           |  30 +++
 src/Tui/Listener/ModelControlListener.php          |  31 ++-
 .../Handler/ExecuteLlmStepWorkerTest.php           |  36 ++--
 .../Handler/ExecutionFailureDrillTest.php          |   4 +-
 .../Application/Handler/ExecutionWorkerTest.php    |  18 +-
 .../Application/Pipeline/AdvanceRunHandlerTest.php |  33 +++
 .../Pipeline/AgentRunnerStartIdempotencyTest.php   |   4 +-
 .../Pipeline/ApplyCommandHandlerTest.php           | 116 ++++++++---
 .../Pipeline/ApplyShellCommandHandlerTest.php      |   4 +-
 .../Pipeline/CommandMailboxPolicyTest.php          |  20 +-
 .../Pipeline/LlmStepResultHandlerTest.php          |  20 +-
 .../RunCommitAfterTurnCommitPersistedSeqTest.php   |   2 +-
 .../Application/Pipeline/RunCommitLoggingTest.php  |  12 +-
 .../Pipeline/RunStateModelIdentityTest.php         | 151 ++++++++++++++
 .../Domain/Command/CommandBoundaryTest.php         |   3 +-
 .../Domain/Message/AgentBusMessageContractTest.php |   7 +-
 .../ToolBatchStateDTOParentModelRoundTripTest.php  |  62 ++++++
 .../Storage/InMemoryRunStoreCasTest.php            |  10 +-
 .../Infrastructure/SymfonyAi/LlamaCppSmokeTest.php |   2 +-
 .../SymfonyAi/PlatformIntegrationTest.php          |  10 +-
 .../SymfonyAi/Replay/ReplayRecordingTest.php       |   2 +-
 .../Infrastructure/SymfonyAi/Replay/ReplayTest.php |   2 +-
 .../Infrastructure/SymfonyAi/TraceReplayTest.php   |   4 +-
 .../AgentCore/Support/Builder/RunStateBuilder.php  |  10 +
 .../Support/Builder/StartRunMessageBuilder.php     |   2 +
 .../Artifact/AgentArtifactRetrievalServiceTest.php |   4 +-
 .../Agent/Artifact/AgentChildRunStoreTest.php      |  28 +--
 .../Agent/Artifact/ChildAwareRunStoreTest.php      |   8 +-
 ...05BareAgentsEffectiveContextIntegrationTest.php |   8 +-
 .../Launch/DeferredSubagentBatchLaunchTest.php     |   3 +-
 .../DeferredSubagentBatchLifecycleTest.php         |   8 +
 ...hildRunModelRoutingProvenanceRegressionTest.php | 227 +++++++++++++++++++++
 .../SubagentChildLaunchModelInheritanceTest.php    |  88 ++++++++
 .../SubagentChildProgressSummaryBuilderTest.php    |   2 +-
 .../SubagentPromptUserContextContractTest.php      |  10 +-
 .../Support/PipelineCapturingAgentRunner.php       |   4 +
 .../Fork/ForkChildStartRunInputCompositionTest.php |  12 +-
 .../Agent/Fork/ForkExecutionServiceTest.php        |  93 ++++++++-
 .../ForkSnapshotCompactionBeforeLaunchTest.php     |   9 +-
 .../Application/Pipeline/CompactRunHandlerTest.php |  31 +--
 .../Pipeline/CompactionStepResultHandlerTest.php   |  15 +-
 .../AutoCompactionHookSubscriberTest.php           |   5 +-
 .../ModelSelectionActiveModelResolverTest.php      |  79 ++-----
 ...ControllerReplayAutoCompactionMultiTurnTest.php |   2 +-
 ...ReplayAutoCompactionRepeatedReplicationTest.php |   2 +-
 ...ControllerReplayAutoCompactionToolCycleTest.php |   2 +-
 .../Controller/E2E/ControllerReplayE2eTestCase.php |  14 ++
 .../E2E/ControllerReplaySummaryOnlyGuardTest.php   |   2 +-
 .../Controller/E2E/RewindBranchLiveE2eTest.php     |  13 +-
 .../InProcessAttachDoesNotContinueTest.php         |   4 +
 .../ParentPromptUserContextRegressionTest.php      |   9 +-
 .../PromptTemplateExpansionInProcessTest.php       |  19 +-
 .../InProcess/StartRunPersistsSessionModelTest.php |  20 +-
 .../Messenger/WorkerFailedEventSubscriberTest.php  |  13 +-
 tests/CodingAgent/Session/AggregateResumeTest.php  |   4 +-
 ...RunEventAppenderLiveProgressIntegrationTest.php |   2 +-
 .../Session/CommittedRunEventAppenderTest.php      |   2 +-
 .../Session/HatfieldSessionStoreTest.php           |  11 +
 .../Session/Repair/SessionRepairServiceTest.php    |  14 +-
 .../Replay/SessionRunStateReplayServiceTest.php    |   8 +
 .../SessionRewindServiceDuplicateSequenceTest.php  |   2 +-
 tests/CodingAgent/Session/SessionRunStoreTest.php  |  16 +-
 .../ToolBatchSnapshotCleanupHookSubscriberTest.php |   6 +-
 .../CodingAgent/Support/TestDirectoryIsolation.php |  32 ++-
 tests/Tui/E2E/TuiRepairCommandE2eTest.php          |   2 +-
 122 files changed, 1789 insertions(+), 424 deletions(-)
 create mode 100644 migrations/Version20260723230000.php
 create mode 100644 src/CodingAgent/Runtime/Controller/CommandHandler/ChangeModelHandler.php
 create mode 100644 tests/AgentCore/Application/Pipeline/RunStateModelIdentityTest.php
 create mode 100644 tests/AgentCore/Domain/Tool/ToolBatchStateDTOParentModelRoundTripTest.php
 create mode 100644 tests/CodingAgent/Agent/Execution/Subagent/ChildRun/ChildRunModelRoutingProvenanceRegressionTest.php
 create mode 100644 tests/CodingAgent/Agent/Execution/Subagent/ChildRun/Preparation/SubagentChildLaunchModelInheritanceTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/guarantee-subagent-model-routing-provider-provenance.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/guarantee-subagent-model-routing-provider-provenance.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Final PR gate: deterministic castor check PASS (124.9s).; Final independent reviewer: APPROVED.
- Summary: PR #318 confirmed merged on GitHub at 2026-07-24T19:49:54Z (merge commit 3aff4302c4bd846c2cec0b822f4b9ea66d8918f1). Completing task workflow and cleaning worktree.

## Task workflow update - 2026-07-24T19:58:07.176Z
- Validation: Post-merge check qa-20260724-195515-250183-b95046d6: deptrac PASS; controller-replay PASS 9/127; TUI PASS 37/190; llm-real PASS 13/175; phpstan PASS; cs-check PASS; cache guard stable 146→146; leak check PASS.; Unit lane sole failure: OmPackageConsoleSmokeTest::testListAndMigrateAgainstTempDatabase — observational-memory bin/console cannot autoload Ineersa\HatfieldExt\ObservationalMemory\Kernel.; After refreshing Composer autoload, ObservationRepositoryIdempotencyTest PASS 2/5; controller replay standalone PASS 9/127; focused TUI PASS 1/10.; First post-merge gate qa-20260724-195026-243065-99ffdc80 additionally had transient controller/TUI startup failures; no worker leaks. Those lanes passed on retry.
- Summary: Post-merge integration validation was run. Model-routing/runtime lanes are green, but the overall integration check remains blocked by a pre-existing/concurrently merged observational-memory package smoke failure unrelated to PR #318: its standalone bin/console searches only up to `.hatfield/vendor/autoload.php` and cannot reach repository-root `vendor/autoload.php`, so `Ineersa\HatfieldExt\ObservationalMemory\Kernel` is not autoloaded. This OM change was not on the task branch but is an ancestor of GitHub merge commit 3aff4302c after merging current main.

## Task workflow update - 2026-08-06T20:59:11.562Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
