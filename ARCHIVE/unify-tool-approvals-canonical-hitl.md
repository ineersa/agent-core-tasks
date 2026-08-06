# Unify exact tool approvals with canonical HITL continuations

## Goal
Replace SafeGuard's separate blocking ToolQuestion approval lifecycle with canonical AgentCore HITL while preserving exact automatic execution of the originally intercepted tool call.

Approved architecture:
- Canonical human-input state supports two typed continuation semantics: model-turn continuation (existing ask_human) and exact tool-call continuation (approval).
- AgentCore owns only generic pending-human-request state, correlation, WaitingHuman lifecycle, answer validation, replay, and continuation dispatch; it must not know SafeGuard or extension implementations.
- For approval, preserve the unresolved original call in existing durable tool-batch state; do not append the approval answer to model context and do not run an extra LLM turn before executing/denying the tool.
- SafeGuard's extension layer resolves Allow once / Always allow / Block / cancel. Approval must bind server-side to the persisted run/request/tool-call identity and original argument signature.
- Allow once automatically executes exactly the original call with a durable one-shot permit; Always allow persists policy before executing; failure, malformed answer, deny, cancel, unattended mode, and persistence failure fail closed.
- Waiting approval must not block a tool worker. Reuse existing ToolBatchStateDTO/pending tool semantics rather than inventing a separate deferred/prelaunch/recovery lifecycle unless a focused design audit proves a missing primitive.
- Existing ask_human model-continuation behavior remains unchanged.
- SafeGuard uses canonical waiting_human → human_input.requested → answer_human surface and existing child WaitingHuman parent-progress projection. Do not add cross-run AgentSessionClient exceptions, TUI status latches, transcript reconciliation walkers, or duplicate status owners.
- Keep RuntimeBashBackgroundPromptAdapter/ToolQuestion infrastructure out of scope unless required; it has different tool-local boolean/background semantics.
- Fix only the minimal generic question view-ownership issue needed so a child question closes visually on return to main, cannot capture parent submit/ESC, and reappears when the child view is re-entered.

Before implementation, produce an approved state/sequence diagram covering allow, deny, answer redelivery, worker crash/redelivery, multiple approval-requiring calls in one batch, parent/child cancellation, and replay/resume. Keep one active human request per run and queue additional approvals durably. Avoid compatibility paths during active development.

Supersedes rejected PR #301 / fix-child-safeguard-needs-input-status.

## Acceptance criteria
- SafeGuard RequireApproval transitions the exact child or parent run to canonical WaitingHuman and emits human_input.requested through the existing runtime protocol.
- Allow once automatically executes exactly the originally intercepted tool call and arguments, with no approval-induced LLM turn and no model-visible approval answer.
- Always allow persists the approved policy before executing that exact call; persistence failure fails closed.
- Deny, cancel, malformed/stale/mismatched answers, unattended mode, and hook resolution failure never execute the tool.
- Answers are correlated and routed server-side from durable request/run/tool-call identity; duplicate answers and Messenger redelivery cannot execute the tool twice or authorize another call.
- No tool worker remains blocked while waiting for approval; pending tool/batch state is replayable and recoverable using existing canonical state machinery.
- Existing ask_human behavior remains a model-turn continuation and passes its current contract tests.
- Child WaitingHuman status appears correctly in parent transcript card and agents-live picker through canonical subagent_progress projection, without TUI latch or transcript rewriting.
- Returning from child live view closes only the overlay, preserves the pending request, and prevents hidden child questions from consuming parent submit/ESC; re-entering restores the question.
- One approval at a time is active per run; additional approval-requiring calls are durably queued and each requires its own exact authorization.
- SafeGuard's old ToolQuestion event/answer/polling path is removed after replacement proof; unrelated background-process ToolQuestion behavior remains intact.
- Automated proof includes focused domain/runtime contracts plus the lowest-correct-layer TUI proof and a live controller/LLM reproduction of the previously observed child SafeGuard scenario; all QA runs through Castor and final castor check passes.
- docs/hitl-and-approvals.md documents the unified request surface and the distinct model-turn versus exact-tool-call continuation semantics.

## Workflow metadata
Status: ARCHIVE
Branch: task/unify-tool-approvals-canonical-hitl
Worktree: /home/ineersa/projects/agent-core-worktrees/unify-tool-approvals-canonical-hitl
Fork run: c2zer0s9nzcz
PR URL: https://github.com/ineersa/agent-core/pull/305
PR Status: merged
Started: 2026-07-20T15:47:53.967Z
Completed: 2026-07-20T23:50:16.949Z

## Work log
- Created: 2026-07-20T00:35:56.435Z

## Task workflow update - 2026-07-20T00:37:54.090Z
- Summary: Read-only architecture audit completed before task start. Recommended core model: typed PendingHumanRequestDTO persisted in RunState/events with continuation kind ModelTurn or ToolCall; approval leaves pendingToolCalls[toolCallId]=false and references the existing durable ToolBatchStateDTO call envelope. AnswerHuman validates the persisted request server-side; ModelTurn keeps current append-user-message/AdvanceRun behavior, while ToolCall invokes the app-owned approval resolver and redispatches the exact stored ExecuteToolCall with a durable one-shot permit, or completes it blocked on deny/cancel. No blocked worker and no approval-induced LLM turn.
- Existing DeferredToolCompletionCorrelation is only partially reusable: it preserves a call envelope but currently means 'tool already returned deferred; later deliver result', not 'tool has not executed; approve then execute'. Prefer existing ToolBatchStateDTO as the original-call authority; do not create a parallel prelaunch/recovery lifecycle.
- Canonical WaitingHuman currently discards request payload in RunState replay and AnswerHuman always creates a model continuation; these are the central generic seams to extend. AgentCore must retain typed pending request identity/continuation while remaining unaware of SafeGuard/extensions.
- SafeGuard's current ApprovalSessionTracker leaks allow-once authorization because direct blocked-worker continuation never consumes the stored grant; policy writer read-modify-write is not concurrency-safe; these security contracts must be corrected as part of migration, not copied forward.
- Estimated clean implementation is nontrivial (~14–16 production files plus possible schema/event changes), so implementation remains gated on an approved state diagram covering multi-approval batches, redelivery, cancellation, and replay.

## Task workflow update - 2026-07-20T15:47:53.967Z
- Moved TODO → IN-PROGRESS.
- Created branch task/unify-tool-approvals-canonical-hitl.
- Created worktree /home/ineersa/projects/agent-core-worktrees/unify-tool-approvals-canonical-hitl.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/unify-tool-approvals-canonical-hitl.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/unify-tool-approvals-canonical-hitl.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/unify-tool-approvals-canonical-hitl.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/unify-tool-approvals-canonical-hitl.
- Summary: Starting clean replacement work after closing PR #301. First implementation gate is an exact state/sequence design for canonical human-input continuations, including redelivery, multi-approval batches, cancellation, and replay; no production edits until that design is reviewed.

## Task workflow update - 2026-07-20T16:09:41.892Z
- Summary: Architecture gate refined: canonical RunState must own an ordered typed pending-human-request queue; ToolBatchStateDTO remains exact-call authority and gains only approval suspension/redispatch coordination. Tool worker returns a typed generic suspension and exits. Run-control admission removes the call from executable in-flight state, enqueues the request, and sets WaitingHuman. AnswerHuman validates only the active persisted request and delegates ToolCall resolution through an AgentCore contract implemented by CodingAgent. Allow marks the exact stored call approved with an internal permit and returns it to the existing batch dispatcher; deny/cancel produces the original call's terminal blocked result. Additional parallel approvals queue in RunState and batch state; ordinary parallel results may still collect. No event-log scanning, client-kind routing, blocked worker, duplicated call envelope, or separate approval entity.
- Final audit found no existing durable post-commit redrive for an approved call: ApplyCommandHandler currently marks human response applied before commit, and postCommit dispatch can be lost. The implementation must make approved/denied continuation intent durable in RunState/ToolBatchState and let existing/new minimal batch recovery redrive it; never dispatch synchronously before CAS.
- General arbitrary tool execution remains at-least-once across worker crash after side effect, as today. This task guarantees duplicate answers/redelivery do not authorize a different call or create an additional logical dispatch beyond the existing tool execution guarantee; it must not claim impossible global exactly-once side effects.
- Implementation will proceed in reviewed micro-slices, starting with generic persisted pending-human-request state/replay while preserving ask_human behavior.

## Task workflow update - 2026-07-20T17:06:46.118Z
- Summary: Slice A accepted after reviewer suggestions at HEAD 884d04053 (typed pending request, strict replay/answer validation, serializer proof). First Slice B implementation at 0c7732028 is rejected before continuation: it violated hard scope with 16 files/+850, a 170-line parallel handler, 376-line test, public test seam, and incomplete event-only ToolCall replay. It will be non-destructively reverted and replaced by the smaller existing ToolCallResult run-control envelope/handler path rather than adding a second suspension message/router/handler.
- Slice A commits: c398b7d02 RED, c29504ce9 GREEN, 15b582517 minimality hardening, 884d04053 reviewer corrections. Focused tests/deptrac/phpstan/cs-check green.
- Rejected Slice B commits: dd2c66998 RED and 0c7732028 GREEN. Do not build on their new ToolExecutionSuspension message/handler/router/config or batchSnapshot test seam. Replacement target: typed non-terminal suspension variant carried through existing ToolCallResult envelope; ToolCallResultHandler branches before terminal collection, preserving one run-control path.

## Task workflow update - 2026-07-20T17:47:06.784Z
- Summary: Micro-slices A and B are implemented and reviewer-approved at HEAD 083f44221. A: durable typed pending human-input queue in RunState/replay with strict ask_human answer correlation and serializer proof. B: typed non-terminal tool suspension reuses existing ToolCallResult run_control path; ToolBatchState tracks awaiting calls; canonical WaitingHuman payload is replay-complete; parallel sibling results preserve WaitingHuman; no separate message/handler/router/lifecycle. Focused Castor tests, phpstan, deptrac, cs-check green.
- Rejected oversized first Slice B (second suspension message/handler/router, +850) was non-destructively reverted. Minimal replacement was implemented and reviewer BLOCKED once for parallel WaitingHuman status; corrected at 083f44221 and re-review APPROVED.
- Current cumulative diff vs task base: 31 files, +972/-28 (production/config +558/-12; tests +414/-16). The broad file count is largely RunState copy propagation plus durable batch serialization. SafeGuard migration, answer→exact-call resume/deny, old ToolQuestion-path deletion, and minimal view ownership remain. Pause before Slice C to keep scope/cost visible.

## Task workflow update - 2026-07-20T18:01:04.212Z
- Summary: User correctly challenged current +972/-28 as additive rather than unified. Read-only audit confirms Slice A/B only built canonical admission foundation; legacy SafeGuard ToolQuestion blocking-poll path could not yet be deleted because answer/resume was not implemented. Current additions classify as ~+30 immutable RunState propagation, ~+528 meaningful runtime foundation, ~+414 tests/support. Before Slice C, semantics-preserving compaction can target +740–790 total. Slice C must be a replacement slice: implement exact-call answer/resume and simultaneously delete the legacy SafeGuard polling/tracker/string-answer path, not leave both.
- Legacy deletion inventory: ExtensionToolHookEventSubscriber blocking approval path ~290–335 LOC; ApprovalSessionTracker 138 entire file; SafeGuard tracker/answer coupling ~180–235; ToolQuestion SafeGuard string-answer path ~75–85; AnswerToolQuestionHandler approval/string routing ~80–105; optional local enum branch ~70–95. Generic ToolQuestion subsystem remains because bash background prompts use it.
- Legacy tests deletable/replaced: ApprovalSessionTrackerTest 97; SafeGuard tracker/callback test sections ~430–500; subscriber polling tests ~400–480; old controller replay 323; old TUI E2E 333. Realistic finished cumulative diff estimate is +1600–1900/-2600–3000 (net code reduction), if Slice C deletes old path in the same change. Hard guard proposed: no extra entity/table/router/lifecycle, total final net negative, production near neutral or net negative; otherwise stop/revert rather than ship another additive architecture.

## Task workflow update - 2026-07-20T19:29:29.520Z
- Validation: Read testing skill and tests/AGENTS before implementation/QA.; Focused modified-class Castor tests passed (107 tests, then 63 tests after final fixes).; castor test:controller-replay: 8 tests / 100 assertions OK.; castor deptrac: 0 violations.; castor phpstan: 0 errors.; castor cs-check: clean after castor cs-fix.; Full castor check intentionally deferred to task-to-pr gate.
- Summary: Implementation complete at HEAD 714aecef3. Slice C replaces the old SafeGuard ToolQuestion blocking-poll path in the same slice: RequireApproval now emits canonical WaitingHuman/human_input.requested, answer_human requeues the exact stored ExecuteToolCall through existing batch/maxParallelism scheduling, resumed answer is validated against run/tool/question/originating hook, Allow executes once without extra LLM turn, Block becomes normal terminal denied result. ApprovalSessionTracker, string ToolQuestion answer path, SafeGuard local-choice TUI branch, and obsolete broad SafeGuard TUI E2E are deleted; bash boolean ToolQuestion remains. No new resolver/entity/table/router/lifecycle/recovery worker (one migration only drops obsolete answer_text).
- Slice C commit 714aecef3: +1023/-3038. Cumulative vs base 867df7925: +1992/-3063, net -1071. Production/docs/config/migrations net +494; total code is materially net-negative, but production did not reach near-neutral because durable RunState/batch foundation remains. No further line-count deletion was forced at expense of semantics/rationale.
- Accepted existing atomicity class: batch mutation can precede RunState CAS and post-commit effect dispatch; same-answer retry is idempotent and returns same effect. No task-specific phase/recovery lifecycle was added.

## Task workflow update - 2026-07-20T19:52:52.376Z
- Validation: Focused Castor suspension/SafeGuard/subscriber tests passed.; Broader focused AgentCore filters: 97 tests / 507 assertions OK.; castor test:controller-replay: 8 tests / 100 assertions OK.; castor deptrac: 0 violations.; castor phpstan: 0 errors.; castor cs-check: clean after castor cs-fix.; Full castor check deferred to task-to-pr gate.
- Summary: Task-start implementation finalized at HEAD b1353acc1 after correcting Slice C crash/redelivery, strict-correlation, privacy, cancellation, and stale legacy-code issues. Exact approval resume now survives CAS retry and post-commit effect-dispatch failure without a new lifecycle: durable batch answer redrive returns the same ExecuteToolCall effect; command is marked applied only after effects; RunMessageProcessor executes effects/callbacks even for no-state redrive results. Malformed/cross-correlated refs fail closed; suspension requires inFlight; logs/results no longer expose raw exception text; SafeGuard canonical cancellation vocabulary supported. Obsolete operation-key identity, hook index, stale polling docs/comments removed.
- Correction commit b1353acc1. Cumulative vs 867df7925: 61 files, +2491/-3284, total net -793; src net +615. Slice C itself is approximately production-near-neutral relative to the +546 A/B foundation, but cumulative production remains positive because the durable canonical RunState/batch continuation foundation is new. No broad RunState-copy refactor or semantic deletion was performed solely for LOC.
- Remaining accepted atomicity: once Messenger accepts the exact ExecuteToolCall, ordinary transport durability/tool execution idempotency applies. No task-specific recovery worker/phase/entity added.

## Task workflow update - 2026-07-20T20:12:11.450Z
- Summary: User live smoke at b1353acc1 disproved replay-green completion: SafeGuard canonical prompt answered Allow once, state transitioned WaitingHuman→Running and exact write ExecuteToolCall was redispatched/executed, but terminal ToolCallResult was silently ignored; run stayed Running with pendingToolCalls[call]=false and no events after agent_command_applied seq 111. Exact live artifacts: session 1, tool call call_bb5e65e3e0ce4d7d9907801e, batch still inFlight with humanInputAnswer. Root cause: suspension ToolCallResult and resumed terminal ToolCallResult reuse ExecuteToolCall idempotencyKey, so RunMessageProcessor marks the non-terminal suspension handled under the same key and drops the later terminal result before ToolCallResultHandler. Existing controller replay did not catch this real topology/runtime idempotency behavior.
- Live evidence: first suspension dispatched ToolCallResult and committed waiting_human seq110; answer committed seq111 and dispatched exact ExecuteToolCall; resumed worker sent terminal ToolCallResult message 55; RunOrchestrator handled it in 0.21ms with no persistence commit/event because idempotency short-circuited. Must give non-terminal suspension and terminal result distinct deterministic idempotency identities.
- Per testing rules, correction requires a live llm-real controller regression reproducing start→SafeGuard human_input→answer_human→same tool_call_id completion, because replay was green while live remained stuck. Do not touch root-owned messenger consumer PID 3343.

## Task workflow update - 2026-07-20T20:19:29.643Z
- Validation: Focused unit/processor tests: 11 tests / 80 assertions OK.; castor test:controller-replay: 8 tests / 103 assertions OK; replay now requires completed tool_call_id match the human-input request.; castor test:llm-real --filter=SafeGuardAllowOnceLiveE2eTest: 1 test / 14 assertions OK (~19s), real controller write outside cwd → Allow once → matching same tool_call_id completed and file exists.; castor deptrac: 0 violations.; castor phpstan scoped factory: 0 errors.; castor cs-check: clean.; Full castor check pending task-to-pr.
- Summary: Live Allow-once stuck bug fixed at HEAD 05704b1ec (RED 43d4bec9e, GREEN 05704b1ec). Root cause was shared run_control idempotency identity: non-terminal suspension ToolCallResult marked the ExecuteToolCall key handled, causing resumed terminal ToolCallResult to be silently dropped. ToolCallResultFactory now generates distinct deterministic keys: suspension keyed by run/step/toolCall/questionId; terminal/throwable/deferred keyed by run/step/toolCall. Same suspension dedups, different later suspension remains valid, terminal executes once.
- User's currently stuck session was launched at b1353acc1 and cannot pick up the fix; restart from worktree HEAD 05704b1ec and use a new session for retest. Historical stuck session remains unchanged; no root-owned or active workers were touched.

## Task workflow update - 2026-07-20T20:40:21.474Z
- Summary: User live-tested corrected Allow once path successfully at 05704b1ec. New simplification request for same task: remove free-form `Type your answer` from SafeGuard approval UI because arbitrary text is denied; reduce SafeGuard approval choices to exactly Allow and Deny; remove Always allow runtime behavior/persistence; preserve static settings-based allowlists for allowed commands/paths/tools. Implementation pending focused impact audit.
- Scope boundary: do not remove the generic canonical HITL free-form capability or generic boolean ToolQuestion flow. Only SafeGuard approval request/schema/answer handling should become fixed two-choice. Preserve reading configured SafeGuard allow rules; remove only interactive Always allow choice and its policy mutation functionality.

## Task workflow update - 2026-07-20T20:51:23.515Z
- Recorded fork run: c2zer0s9nzcz
- Validation: Focused Castor unit/integration: 96 tests / 355 assertions OK.; castor test:controller-replay: 8 tests / 103 assertions OK.; castor test:llm-real --filter=SafeGuardAllowLiveE2eTest: 1 test / 14 assertions OK (~15s).; castor deptrac: 0 violations.; castor phpstan: 0 errors.; castor cs-check: clean.; Full castor check remains pending task-to-pr gate.
- Summary: SafeGuard simplification completed at HEAD 50c688c80: exact two fixed choices `✅ Allow` / `❌ Deny`; tool-call continuations set allowOther=false so `Type your answer` is absent; model-turn ask_human free-form remains; interactive Always allow and SafeGuardPolicyWriter production/test removed; static YAML settings allowlists and classification policies preserved. Net slice +159/-582 (-423) across 19 files. Worktree clean.
- Implementation fork c2zer0s9nzcz read testing skill, tests/AGENTS.md, external SafeGuard ai-index + target settings/maintenance docs. No new entity/table/migration/router/service/API/lifecycle. Generic boolean ToolQuestion remains for bash background questions; generic model-turn HITL retains free-form answers.
- Renamed live proof SafeGuardAllowOnceLiveE2eTest → SafeGuardAllowLiveE2eTest and updated real flow to answer `✅ Allow`, requiring matching same tool_call_id completion.

## Task workflow update - 2026-07-20T20:59:33.524Z
- Summary: Final user-requested presentation correction before review: denied/cancelled resumed approval currently renders raw JSON (`{"denied":true,...}`). Change only this structured approval-block tool result to existing TOON encoding at ExtensionToolHookEventSubscriber boundary; no broader result-renderer refactor. After focused proof, launch reviewer and transition task to CODE-REVIEW as explicitly requested.

## Task workflow update - 2026-07-20T21:02:18.905Z
- Validation: RED proof failed before fix because raw result was array; GREEN ExtensionToolHookEventSubscriberTest 3 tests / 13 assertions OK.; castor deptrac: 0 violations.; castor phpstan scoped subscriber: 0 errors.; castor cs-check: clean.
- Summary: Final presentation slice committed at e80f34376: resumed approval Block/cancel ToolResult now uses existing HelgeSverre TOON encoder, preserving denied/reason/message fields. Minimal two-file change; ordinary tool formatting untouched.

## Task workflow update - 2026-07-20T21:21:02.176Z
- Summary: Final reviewer verdict APPROVE WITH SUGGESTIONS. Two actionable correctness/security findings will be addressed before CODE-REVIEW: (1) multi-request FIFO post-commit dispatch failure retry can reject already-applied q1 because q2 remains active WaitingHuman, leaving q1 inFlight and stuck; (2) originating approval hook unloaded before resume can leave typed answer unconsumed and allow real handler fall-through. Also correct stale non-blocking-worker doc wording. Quality extraction/dead historical projection NTH findings intentionally out of scope to avoid unrelated refactor.
- Reviewer comprehensive final audit found no critical issues and confirmed user contracts, legacy deletion, exact correlation, idempotency, replay/live proofs, and net-negative scope. Re-review required after focused fixes because project reviewer policy requires addressing APPROVE WITH SUGGESTIONS actionable findings.

## Task workflow update - 2026-07-20T21:33:29.727Z
- Validation: castor test at 2f5c56038: FAILED 1/778 tests after 13.7s; only failure DeferredToolCompletionRuntimeTest::testImmediateToolStillDispatchesCanonicalToolCallResult line 142 expected old shared key, actual distinct terminal SHA-256 key.
- Summary: Re-review APPROVED at HEAD 2f5c56038. Pre-PR full `castor test` then exposed one stale assertion not covered by focused suites: DeferredToolCompletionRuntimeTest expected terminal ToolCallResult to reuse ExecuteToolCall key `tool-idemp-immediate`, but production intentionally now hashes a distinct terminal key to prevent suspension/terminal collision. Need update this existing test to the new stable contract, then rerun local gates before CODE-REVIEW.

## Task workflow update - 2026-07-20T21:39:03.998Z
- Validation: Reviewer verdict: APPROVED; no issues. Verified q1 redrive while q2 remains WaitingHuman and unconsumed origin-hook answer fail-closed TOON denial.; castor test: OK 4443 tests / 15242 assertions (second full run; first unrelated SQLite timing flake passed focused and on rerun).; castor test:tui: OK 36 tests / 185 assertions.; castor test:llm-real: OK 12 tests / 163 assertions; proxy cache warmed.; castor test:controller-replay: OK 8 tests / 103 assertions.; castor deptrac: 0 violations.; castor phpstan: 0 errors.; castor cs-check: clean.
- Summary: Final re-review APPROVED at 2f5c56038 after fixing multi-request postCommit redrive and origin-hook unload fail-closed behavior. Test-only stale idempotency assertions then corrected at c746d4c84; no production change. Branch ready for CODE-REVIEW gate at HEAD c746d4c84.
- Reviewer NTH-only observations intentionally not implemented: helper extractions, dead historical projection cleanup, negative duplicate test, variable rename; none affect correctness and would broaden scope.

## Task workflow update - 2026-07-20T21:43:09.963Z
- Validation: move_task deterministic castor check: exit 1 solely cache guard growth 82→84.; castor test:llm-real after failure: 12/163 OK, but cache again grew 84→86; not stable/warm.
- Summary: CODE-REVIEW transition gate failed only llama-proxy cache-growth guard: 82→84 entries. A subsequent full warm run also grew 84→86, proving new SafeGuardAllowLiveE2eTest prompt is non-cacheable because it embeds random sessionId in `../sg-live-<session>.txt`; each run generates two unique LLM request keys. Tests themselves passed, but deterministic gate cannot pass until the live test uses a stable prompt/path while retaining isolated filesystem side effects.

## Task workflow update - 2026-07-20T21:46:00.610Z
- Validation: Focused SafeGuard live #1: 1/14 OK, proxy 86→88 (expected new cassette warm).; Focused SafeGuard live #2: 1/14 OK, proxy 88→88.; Full castor test:llm-real: 12/163 OK, proxy 88→88.; castor cs-check: clean.
- Summary: Deterministic llama-proxy blocker fixed test-only at 114acff7f: SafeGuard live test now runs controller from nested fixed cwd under unique isolation root, uses constant `../sg-live-allow.txt` prompt path, and cleans the isolation root. Prompt cache stable without shared filesystem target or production/test-base API changes.

## Task workflow update - 2026-07-20T21:48:13.465Z
- Validation: Gate test:tui error: empty pane startup timeout in BashCancelFollowUpE2eTest line47; only 4 tests reached before stop.; castor clean:cleanup:workers:list: no stale QA worker candidates.; castor test:tui --filter=BashCancelFollowUpE2eTest: OK 1/1.; Prior castor test:tui full: OK 36/185.
- Summary: Second CODE-REVIEW gate passed all lanes except unrelated TUI startup flake: BashCancelFollowUpE2eTest timed out waiting 10s for initial cursor with empty tmux capture under parallel full-gate load. No stale QA workers remained. Exact focused test immediately passed 1/1 in 12.5s; prior full castor test:tui passed 36/185. No code change warranted; retry deterministic gate.

## Task workflow update - 2026-07-20T21:50:43.404Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (131.9s).
- Pushed task/unify-tool-approvals-canonical-hitl to origin.
- branch 'task/unify-tool-approvals-canonical-hitl' set up to track 'origin/task/unify-tool-approvals-canonical-hitl'.
- Created PR: https://github.com/ineersa/agent-core/pull/305
- Validation: Reviewer APPROVED.; castor test: 4443/15242 OK.; castor test:tui full: 36/185 OK; focused BashCancelFollowUp 1/1 OK after gate startup flake.; castor test:controller-replay 8/103 OK.; castor test:llm-real 12/163 OK; cache stable at 88.; Deptrac/PHPStan/CS green.
- Summary: Retrying CODE-REVIEW deterministic gate at clean HEAD 114acff7f after unrelated empty-pane TUI startup timeout. Focused failing TUI test passed immediately, no stale workers, full TUI suite previously green, llama-proxy stable/warm. Reviewer APPROVED.

## Task workflow update - 2026-07-20T23:50:16.949Z
- Moved CODE-REVIEW → DONE.
- Merged task/unify-tool-approvals-canonical-hitl into integration checkout.
- Merge made by the 'ort' strategy.
 docs/hitl-and-approvals.md                         | 331 +++----
 docs/settings.md                                   |  11 +-
 migrations/Version20260720120000.php               |  30 +
 .../Application/Handler/ExecuteToolCallWorker.php  |   3 +
 .../Application/Handler/ToolBatchCollector.php     | 306 ++++++-
 .../Application/Handler/ToolCallResultFactory.php  |  76 +-
 src/AgentCore/Application/Handler/ToolExecutor.php |  37 +
 .../Application/Pipeline/AdvanceRunHandler.php     |   7 +
 .../Application/Pipeline/ApplyCommandHandler.php   | 293 +++++-
 .../Application/Pipeline/CommandMailboxPolicy.php  |   1 +
 .../Application/Pipeline/LlmStepResultHandler.php  |   5 +
 src/AgentCore/Application/Pipeline/RunCommit.php   |   2 +
 .../Application/Pipeline/RunMessageProcessor.php   |  11 +-
 .../Application/Pipeline/ToolCallResultHandler.php | 107 ++-
 .../Application/Replay/RunStateReducer.php         |  60 +-
 src/AgentCore/Application/Tool/ToolContext.php     |  16 +
 src/AgentCore/Domain/Event/EventFactory.php        |   1 +
 .../Domain/Message/ExecuteShellToolCall.php        |   8 +-
 src/AgentCore/Domain/Message/ExecuteToolCall.php   |  26 +
 src/AgentCore/Domain/Message/ToolCallResult.php    |  18 +-
 .../Domain/Run/HumanInputContinuationKindEnum.php  |  17 +
 .../Domain/Run/PendingHumanInputRequestDTO.php     | 132 +++
 src/AgentCore/Domain/Run/RunState.php              |   8 +-
 src/AgentCore/Domain/Tool/ToolBatchStateDTO.php    |  43 +
 .../Domain/Tool/ToolCallHumanInputAnswerDTO.php    |  74 ++
 .../Tool/ToolExecutionHumanInputSuspension.php     |  23 +
 .../Application/Pipeline/CompactRunHandler.php     |   2 +
 .../Pipeline/CompactionStepResultHandler.php       |   2 +
 src/CodingAgent/Entity/ToolQuestion.php            |  31 +-
 .../Builtin/SafeGuard/ApprovalSessionTracker.php   | 138 ---
 .../Builtin/SafeGuard/SafeGuardExtension.php       |  14 +-
 .../Builtin/SafeGuard/SafeGuardPolicyWriter.php    | 148 ----
 .../Builtin/SafeGuard/SafeGuardToolCallHook.php    | 217 +----
 .../Extension/ExtensionHookRegistry.php            |  33 +-
 .../Extension/ExtensionToolHookEventSubscriber.php | 433 +++++----
 .../Approval/ApprovalAnswerHookInterface.php       |  10 +-
 .../CommandHandler/AnswerHumanHandler.php          |  12 +-
 .../CommandHandler/AnswerToolQuestionHandler.php   |  81 +-
 .../CommandHandler/ExecuteShellToolCallWorker.php  |   5 +-
 .../Messenger/WorkerFailedEventSubscriber.php      |   1 +
 .../Runtime/Protocol/RuntimeEventTranslator.php    |  12 +-
 .../Session/CommittedRunEventAppender.php          |   1 +
 .../Session/Repair/SessionRepairService.php        |   1 +
 .../Replay/SessionRunStateReplayService.php        |   2 +
 .../Tool/ToolQuestion/ToolQuestionStore.php        |  40 +-
 .../ToolQuestion/ToolQuestionStoreInterface.php    |  18 -
 src/Tui/Listener/RuntimeQuestionEventHandler.php   | 116 +--
 .../Handler/DeferredToolCompletionRuntimeTest.php  |  10 +-
 .../Handler/ToolCallHumanInputSuspensionTest.php   | 691 +++++++++++++++
 .../Handler/ToolCallResultFactoryDeferredTest.php  |  76 ++
 .../PendingHumanInputAnswerValidationTest.php      | 141 +++
 .../AgentCore/Support/Builder/RunStateBuilder.php  |  19 +-
 ...05BareAgentsEffectiveContextIntegrationTest.php |   1 +
 .../SubagentPromptUserContextContractTest.php      |   1 +
 .../SafeGuard/ApprovalSessionTrackerTest.php       |  97 --
 .../Builtin/SafeGuard/SafeGuardExtensionTest.php   |  73 +-
 .../SafeGuard/SafeGuardPolicyWriterTest.php        | 147 ----
 .../SafeGuard/SafeGuardToolCallHookTest.php        | 978 ++-------------------
 .../ExtensionToolHookEventSubscriberTest.php       | 911 ++++++-------------
 .../CommandHandler/AnswerHumanHandlerTest.php      |   4 +-
 .../AnswerToolQuestionHandlerTest.php              |  16 -
 .../Controller/E2E/SafeGuardAllowLiveE2eTest.php   | 154 ++++
 .../E2E/SafeGuardApprovalControllerReplayTest.php  | 262 ++----
 .../Replay/SessionRunStateReplayServiceTest.php    |   1 +
 tests/CodingAgent/Session/SessionRunStoreTest.php  |  79 +-
 .../Session/SessionToolBatchStoreTest.php          |   3 +
 tests/Tui/E2E/SafeGuardApprovalTuiE2eTest.php      | 333 -------
 tests/Tui/Listener/TickPollListenerTest.php        |  97 +-
 tests/Tui/Question/QuestionControllerTest.php      |  35 +-
 69 files changed, 3307 insertions(+), 3785 deletions(-)
 create mode 100644 migrations/Version20260720120000.php
 create mode 100644 src/AgentCore/Domain/Run/HumanInputContinuationKindEnum.php
 create mode 100644 src/AgentCore/Domain/Run/PendingHumanInputRequestDTO.php
 create mode 100644 src/AgentCore/Domain/Tool/ToolCallHumanInputAnswerDTO.php
 create mode 100644 src/AgentCore/Domain/Tool/ToolExecutionHumanInputSuspension.php
 delete mode 100644 src/CodingAgent/Extension/Builtin/SafeGuard/ApprovalSessionTracker.php
 delete mode 100644 src/CodingAgent/Extension/Builtin/SafeGuard/SafeGuardPolicyWriter.php
 create mode 100644 tests/AgentCore/Application/Handler/ToolCallHumanInputSuspensionTest.php
 create mode 100644 tests/AgentCore/Application/Pipeline/PendingHumanInputAnswerValidationTest.php
 delete mode 100644 tests/CodingAgent/Extension/Builtin/SafeGuard/ApprovalSessionTrackerTest.php
 delete mode 100644 tests/CodingAgent/Extension/Builtin/SafeGuard/SafeGuardPolicyWriterTest.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/E2E/SafeGuardAllowLiveE2eTest.php
 delete mode 100644 tests/Tui/E2E/SafeGuardApprovalTuiE2eTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/unify-tool-approvals-canonical-hitl.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/unify-tool-approvals-canonical-hitl.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Pre-merge reviewer: APPROVED.; Pre-merge deterministic castor check: passed in 131.9s.; Focused validation recorded on task: unit/integration 4443/15242, TUI 36/185, controller replay 8/103, live LLM 12/163, Deptrac/PHPStan/CS green.
- Summary: User confirmed PR #305 merged. Moving task to DONE, syncing integration checkout, and cleaning task worktree.

## Task workflow update - 2026-07-20T23:52:36.470Z
- Validation: Post-merge `LLM_MODE=true castor check`: quality OK in 317.7s.; Unit/integration: 4438 tests / 15185 assertions.; Controller replay: 8 / 103.; TUI replay: 36 / 185.; Live LLM: 12 / 163.; Deptrac, PHPStan, CS: green.; llama-proxy cache guard stable 88→88.; QA artifact integrity: 7 lane logs OK.; QA run leak check: no owned processes or tmux sessions.
- Summary: Post-merge validation completed on integration checkout; task worktree removed and IDEA exclusions cleaned.

## Task workflow update - 2026-08-06T20:59:35.151Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
