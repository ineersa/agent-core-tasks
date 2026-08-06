# OM-01 Extension API foundations for observational memory

## Goal
Plan: /home/ineersa/projects/agent-core/.aiassistant/reports/observational-memory-core-implementation-plan.md

Add the generic public Extension API capabilities required by an independently owned Observational Memory extension:

- stable post-commit conversation-boundary notifications;
- read-only canonical session event access;
- one blocking, non-streaming configured-model call;
- runtime/controller lifecycle notifications.

This task provides reusable platform contracts, not OM implementation. Public DTOs must remain independent of AgentCore, CodingAgent internals, Symfony AI implementation objects, Messenger, Doctrine, and TUI internals.

The model capability is intentionally one small synchronous API: `callModel(model, messages, tools, structuredContent)`. The extension reads settings, chooses the model name, owns prompts/tool-loop policy, and receives one completed response; streaming and separate model-resolution/capability APIs are out of scope.

## Acceptance criteria
- Extensions can register a post-commit conversation-boundary hook for completed, failed, and cancelled outcomes. The public context includes run/session identity, an opaque stable boundary identity, source sequence range, latest committed sequence, outcome, and correlation timestamps/metadata.
- Boundary hooks run only after the canonical Hatfield commit and are documented as best-effort acceleration; they do not mutate run state or perform model work.
- Extensions can read immutable public event DTOs by canonical source range without receiving `RunEvent`, `EventStoreInterface`, turn-tree implementation objects, session paths, or state mutation access.
- Event access supports canonical `(run_id, seq)` source references and does not treat `turn_no` as an event identity. No branch-aware event projection is added for OM MVP.
- Extension API exposes one blocking `callModel(string $model, array $messages, array $tools = [], ?array $structuredContent = null)` operation. The configured model name is resolved through Hatfield's existing AI infrastructure and the method returns one completed response with assistant content, tool calls, and requested structured content.
- The call exposes only extension-supplied tools and never Hatfield ambient tools, provider objects, credentials, Symfony AI DTOs, streaming APIs, context-window/capability DTOs, or a separate model-resolution API. Model/provider failures use one stable public error contract.
- Extensions receive generic runtime/controller start and stop notifications sufficient to start and stop extension-owned resources. Hatfield does not register or supervise OM processes.
- Extension API compatibility boundaries and privacy/logging requirements are documented and enforced by focused tests.
- Focused Castor validation is run through Castor only. Runtime changes follow the testing skill and `tests/AGENTS.md` conventions, and required runtime validation includes `castor check`.

## Explicit non-goals
- No OM event types or OM records in Hatfield `events.jsonl`.
- No OM tables in Hatfield's application database.
- No access to Hatfield Messenger buses/transports or Symfony DI container.
- No OM consumer, persistence, prompts, or compaction behavior in this task.

## Workflow metadata
Status: DONE
Branch: task/om-01-interaction-event-boundaries-projection
Worktree: /home/ineersa/projects/agent-core-worktrees/om-01-interaction-event-boundaries-projection
Fork run: jrrp6b0onmss
PR URL: https://github.com/ineersa/agent-core/pull/311
PR Status: merged
Started: 2026-07-21T17:07:30.387Z
Completed: 2026-07-22T16:48:17.991Z

## Work log
- Created: 2026-06-28T21:32:59.619Z
- Revised: 2026-07-21 — replaced the core OM interaction-event task with generic Extension API boundary, event-reader, model, and lifecycle capabilities.
- Revised: 2026-07-21 — reduced AI infrastructure to one blocking, non-streaming `callModel()` method; model selection and prompting policy remain extension-owned.
- Revised: 2026-07-21 — made OM session-global for MVP and reduced event access to the canonical append-only stream; no OM branch projection is required.

## Task workflow update - 2026-07-21T17:07:30.387Z
- Moved TODO → IN-PROGRESS.
- Created branch task/om-01-interaction-event-boundaries-projection.
- Created worktree /home/ineersa/projects/agent-core-worktrees/om-01-interaction-event-boundaries-projection.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/om-01-interaction-event-boundaries-projection.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/om-01-interaction-event-boundaries-projection.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/om-01-interaction-event-boundaries-projection.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/om-01-interaction-event-boundaries-projection.
- Summary: Claimed OM-01 to implement generic Extension API foundations: post-commit conversation boundaries, canonical event reader, blocking non-streaming model call, and runtime lifecycle notifications. OM remains extension-owned and session-global/non-branch-aware.

## Task workflow update - 2026-07-21T17:17:05.981Z
- Summary: Scouting complete. Existing AfterTurnCommit hook runs after allocated canonical seq persistence; OM needs a separate terminal boundary projection and direct permanent-worker-failure coverage. Canonical reader should wrap EventStoreInterface::allFor() into public inclusive-range DTOs. Blocking model call needs a dedicated non-streaming CodingAgent adapter (not stream/ambient-tool LlmPlatformAdapter). Runtime lifecycle should dispatch only from owning controller process, not every extension-loaded Messenger worker. OM remains session-global and event reader returns canonical history.
- All three scouts read AGENTS.md, .agents/skills/testing/SKILL.md, and tests/AGENTS.md before test/runtime analysis.
- Key risk: current model config supports exact provider/model references, not the plan's illustrative @compaction alias; OM-01 should not invent silent alias fallback.
- Key risk: WorkerFailedEventSubscriber appends terminal failure outside RunCommit, so boundary notifications need a shared post-commit notifier seam or explicit integration there.
- Test thesis: protect post-commit terminal boundary delivery, canonical inclusive range reads, isolated non-streaming extension-only model tools/results/errors, and once-per-controller lifecycle callbacks.

## Task workflow update - 2026-07-21T17:18:47.282Z
- Recorded fork run: 5j6ourfgc33c
- Summary: Implementation fork launched in the task worktree with exact scope for public boundary hooks, canonical event reader, isolated blocking model call, controller lifecycle hooks, focused docs/tests, Castor-only validation, and commit. Full castor check/PR/push explicitly deferred to task-to-pr.

## Task workflow update - 2026-07-21T17:30:46.066Z
- Recorded fork run: 5j6ourfgc33c
- Validation: Mandatory reads confirmed by fork: AGENTS.md, testing skill, tests/AGENTS.md, task file, implementation plan.; Attempted worktree Castor focused test: blocked at migration with `OpenAI Codex provider "openai-codex" requires stored OAuth credentials`; no tests executed.; No deptrac/phpstan/cs-check or commit yet.
- Summary: First implementation fork completed partially: production/public contracts, adapters, wiring, focused tests, and docs were written but not committed. Worktree contains only expected OM-01 source/test/docs changes plus untracked new files. Validation stopped during Castor test DB migration because the new ExtensionModelCaller eagerly injects the configured Symfony Platform, causing Codex provider construction under isolated QA HOME without OAuth. This is treated as an implementation issue to fix lazily, not an environment/auth workaround.

## Task workflow update - 2026-07-21T17:31:29.324Z
- Recorded fork run: r46kmv133t4d
- Summary: Follow-up fork launched to review the full uncommitted implementation, fix eager provider construction via lazy platform factory, resolve compile/test/static-analysis issues, run focused Castor validation, add the smallest live blocking-model proof if feasible, and commit without pushing.

## Task workflow update - 2026-07-21T17:44:38.857Z
- Recorded fork run: r46kmv133t4d
- Validation: Fork confirmed mandatory reads: worktree AGENTS.md, .agents/skills/testing/SKILL.md, tests/AGENTS.md, IN-PROGRESS OM-01 task, and OM implementation plan.; Focused OM-01 tests: PASS (18 tests).; castor test: PASS (4467 tests, 15344 assertions).; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS after castor cs-fix.; castor test:llm-real --filter=ExtensionModelCallLiveTest: PASS (1 test, 3 assertions).; Full castor check intentionally deferred to task-to-pr deterministic gate. One unrelated ParaTest timing flake in MessengerSqliteImmediateTransactionMiddlewareTest passed sequentially and on full rerun.
- Summary: OM-01 implementation completed and committed as 5c2934b2b898a6fa57de3cca5b84240f32d14278. Added public Extension API contracts for stable conversation boundaries, canonical session event reads, blocking non-streaming extension model calls, and owning-controller runtime lifecycle notifications; added internal adapters/wiring, permanent worker-failure boundary notification, lazy Symfony AI platform construction, docs, and focused tests. Scope deliberately excludes OM product logic, OM storage/Messenger, compaction hook, TUI, branch projection, aliases, push, and PR. Worktree verified clean on task branch. Ready for separate task-to-pr review/gate phase.

## Task workflow update - 2026-07-21T18:15:04.633Z
- Validation: Reviewer read AGENTS.md, testing skill, tests/AGENTS.md, task file, and OM plan.; Reviewer inspected full origin/main...HEAD diff and returned REQUEST CHANGES.
- Summary: Task-to-PR reviewer verdict: REQUEST CHANGES. Blockers: overbroad stream-substring classification can mislabel upstream/provider failures as unsupported; logs can disagree with public error code; result mapping can leak raw JsonException/ValueError outside ModelCallException contract; ConversationBoundaryProjector silently degrades on event-store read failure without required diagnostics; structuredContent docs overstate provider neutrality. Non-blocking notes include broad Deptrac allowances, platform rebuild cost, boundaryAt timestamp semantics, and a few optional tests.

## Task workflow update - 2026-07-21T18:20:23.321Z
- Recorded fork run: p52eas1cxike
- Validation: Mandatory docs/testing conventions confirmed read by fix fork.; Focused affected tests: PASS (15 tests, 79 assertions).; castor test: PASS (4471 tests, 15377 assertions).; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS after castor cs-fix.; castor test:llm-real --filter=ExtensionModelCallLiveTest: PASS (1 test, 3 assertions).
- Summary: Reviewer-requested fixes committed as f8e9fe70bd024cd3d916405f416ee4c7d26b6247. Corrected overbroad stream error classification and log/public-code disagreement; sanitized all result-mapping failures behind ModelCallException; removed silent inaccurate boundary-range degradation in favor of notifier-isolated failure; corrected structured-output compatibility docs; tightened structuredContent to ?array; documented boundaryAt semantics. Added four focused regressions. Worktree verified clean.

## Task workflow update - 2026-07-21T18:28:09.216Z
- Validation: Re-reviewer read AGENTS.md, testing skill, tests/AGENTS.md, task file, and OM plan.; Re-reviewer inspected full origin/main...HEAD diff and fix commit; verdict APPROVED.; Focused pre-PR gates remain green: castor test, deptrac, phpstan, cs-check, and focused llm-real model-call smoke.
- Summary: Task-to-PR re-review verdict: APPROVED at f8e9fe70bd024cd3d916405f416ee4c7d26b6247. Reviewer confirmed all five prior blockers resolved with focused regression coverage. Remaining notes are non-blocking: public session-reader chained previous exception exposure, event-store ordering invariant documentation, DTO constructor failures potentially mapped as result_mapping_failed, boundaryAt wording nuance, broad Deptrac allowances, and optional additional tests/performance cleanup.

## Task workflow update - 2026-07-21T18:33:54.509Z
- Validation: move_task CODE-REVIEW castor check attempt 1: FAILED only test:controller-replay exit 124.; castor clean:cleanup:workers:list: no stale QA worker candidates.; Worktree castor test:controller-replay: PASS (9 tests, 127 assertions, 76.3s).; Integration main castor test:controller-replay baseline: PASS (9 tests, 127 assertions, 78.5s).
- Summary: First CODE-REVIEW transition gate failed only because test:controller-replay exceeded its internal 90s lane timeout under parallel gate contention (exit 124; log contained only PHPUnit startup). No stale QA workers remained. Isolated worktree replay rerun passed 9 tests/127 assertions in 76.3s; integration-main baseline passed the same 9/127 in 78.5s, indicating tight shared gate budget/environment contention rather than an OM-01 regression. Retrying deterministic gate with warm caches.

## Task workflow update - 2026-07-21T18:36:05.670Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (116.9s).
- Pushed task/om-01-interaction-event-boundaries-projection to origin.
- branch 'task/om-01-interaction-event-boundaries-projection' set up to track 'origin/task/om-01-interaction-event-boundaries-projection'.
- Created PR: https://github.com/ineersa/agent-core/pull/311

## Task workflow update - 2026-07-21T18:36:19.508Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/311
- Updated PR Status: open
- Validation: Deterministic castor check: PASS (116.9s).; Branch pushed: origin/task/om-01-interaction-event-boundaries-projection at f8e9fe70bd024cd3d916405f416ee4c7d26b6247.; PR open, non-draft: https://github.com/ineersa/agent-core/pull/311.; Post-push worktree status: clean.
- Summary: Task-to-PR complete. Deterministic castor check passed on retry in 116.9s, branch pushed, and PR #311 opened against main. Worktree remains clean and local HEAD tracks the pushed branch.

## Task workflow update - 2026-07-21T19:29:08.092Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #311 feedback iteration started. Six inline comments read in full. Main themes: remove custom model-call reconstruction in favor of exposing Hatfield's already-configured model/message/tool invocation path, and eliminate full events.jsonl scans from per-boundary/per-range operations.

## Task workflow update - 2026-07-21T19:44:22.616Z
- Summary: User approved the PR-feedback redesign: replace custom array/DTO model adapter with normal non-streaming Symfony AI Agent invocation using Hatfield's configured lazy Platform, native MessageBag, optional ToolboxInterface with AgentProcessor tool-loop execution, and native ResultInterface. Remove structured output for MVP. Use Hatfield's accepted typed model reference instead of a raw string. Also remove allFor() from the boundary hot path; expose already-committed canonical event batches from hot commit context so the extension queues them directly, and reserve full session reads for recovery/compaction only.

## Task workflow update - 2026-07-21T20:02:35.434Z
- Recorded fork run: xno4mcgxeaij
- Validation: castor test: focused affected tests — OK; castor test — OK (4462 tests, 15340 assertions); castor deptrac — OK (0 violations); castor phpstan — OK (0 errors); castor cs-check — clean after castor cs-fix; castor test:llm-real --filter=ExtensionModelCallLiveTest — OK; castor test:controller-replay — OK (9 tests, 127 assertions)
- Summary: Implemented and committed PR #311 simplification at b56c48f6bb0ac31f648e4635621bd3bd9a9d7723 (56 files, +568/-1213 versus prior PR head). callModel now uses typed AiModelReference, native MessageBag, optional ToolboxInterface with standard AgentProcessor tool loop, native ResultInterface, and Hatfield's lazy configured Platform. Deleted custom model/result/tool/error DTO/mapping stack and structured output. Boundary projection now uses only the just-committed hot batch, removes sourceStartSeq and EventStore reads, and public after-turn hooks receive complete SessionEventDTO batches. Session reader is documented recovery/compaction-only. Worktree clean, branch ahead of remote by one commit; not pushed.

## Task workflow update - 2026-07-21T20:18:36.395Z
- Summary: Independent re-review of complete PR diff at b56c48f6b returned APPROVED WITH SUGGESTIONS. Reviewer confirmed all six inline PR comments are genuinely resolved: configured lazy container Platform is used, typed model is consumed by native Agent, MessageBag/Toolbox are native, custom options/result/schema stack is gone, boundary projection is batch-only with no allFor(), and full reader is recovery-only. No blocking correctness, security, architecture, or test findings. Non-blocking cleanup suggestions: clarify Cancelled enum documentation, make max-seq invariant explicit, broaden missing-reason hydration, relocate AiModelReferenceTest, and rename one existence-checker property.

## Task workflow update - 2026-07-21T20:20:48.804Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (115.4s).
- Pushed task/om-01-interaction-event-boundaries-projection to origin.
- branch 'task/om-01-interaction-event-boundaries-projection' set up to track 'origin/task/om-01-interaction-event-boundaries-projection'.
- PR already exists: https://github.com/ineersa/agent-core/pull/311
- Validation: castor test — OK (4462 tests, 15340 assertions); castor deptrac — OK (0 violations); castor phpstan — OK (0 errors); castor cs-check — clean; castor test:llm-real --filter=ExtensionModelCallLiveTest — OK; castor test:controller-replay — OK (9 tests, 127 assertions); independent reviewer — APPROVED WITH SUGGESTIONS
- Summary: PR feedback iteration complete at b56c48f6bb0ac31f648e4635621bd3bd9a9d7723. Native Symfony AI Agent bridge and hot committed-event batches replace the original custom adapter/history-scan design. Independent reviewer approved with no blockers; all six PR inline comments addressed.

## Task workflow update - 2026-07-22T15:32:33.660Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #311 redesign requested by user. Rework from a clean/minimal base where practical, preserve only viable OM-01 pieces, add a Hatfield-owned configured background-agent capability that reuses the existing Symfony AI Platform/Agent/AgentProcessor and supports streaming transports including Codex WebSocket. User explicitly requested no reviewer before push; push implementation branch first for direct inspection.

## Task workflow update - 2026-07-22T15:36:02.164Z
- Recorded fork run: shviqk3h1uub
- Summary: Implementation fork launched for clean PR #311 redesign. It will non-destructively revert the three rejected OM commits, merge current origin/main, implement a minimal Hatfield-owned blocking agent runner that streams internally through the configured Symfony Platform/Agent/AgentProcessor (including Codex WebSocket), enrich the existing after-turn hot batch rather than add boundary notifier infrastructure, retain an explicit recovery-only canonical reader, omit custom lifecycle APIs, run focused Castor gates, commit, and push normally to the existing PR branch for user inspection. No reviewer and no CODE-REVIEW transition before the user's check.

## Task workflow update - 2026-07-22T15:40:32.235Z
- Summary: User clarified this push is an architecture-direction preview. Tests are optional until the user approves the implementation direction; do not block the preview push or require test expansion now. Existing tests may be adjusted only as needed for the branch to remain usable. Full proof and QA will follow after design approval.

## Task workflow update - 2026-07-22T15:47:23.778Z
- Recorded fork run: shviqk3h1uub
- Validation: Focused Extension tests: PASS (46 tests / 114 assertions).; castor test: PASS on rerun (4456 tests / 15375 assertions); first run had an unrelated ParaTest SubagentLivePickerControllerTest flake.; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS after castor cs-fix.; castor test:llm-real: not run for architecture preview per user instruction.; castor check: not run pending user approval of direction.
- Summary: Architecture-preview redesign pushed normally to existing PR #311 at 7a73737a767b526bd7d976d6176aa1875dfb48a5. Rejected commits were reverted non-destructively, origin/main merged, and the new diff is 21 files (+660/-10): public ExtensionApi agent request/tool/runner contracts, configured internal streaming agent runner with isolated native AgentProcessor toolbox, enriched existing AfterTurnCommit hot batches, and recovery-only canonical event reader. No custom boundary/lifecycle notifier stack, no AiModelReference move, no SessionExistenceChecker, no force push, no reviewer, task remains IN-PROGRESS. Worktree and integration checkout verified clean; remote branch matches HEAD.

## Task workflow update - 2026-07-22T15:59:30.057Z
- Recorded fork run: jrrp6b0onmss
- Summary: User approved the minimal architecture direction and requested removal of NullLogger constructor fallbacks. Follow-up fork will require injected loggers, make new OM logging privacy-safe, carry actual per-event turn/timestamp through the existing committed hot batch, and add one focused live proof that the blocking streaming runner drains to completion and executes an isolated extension tool. No reviewer, status transition, full castor check, or force-push.

## Task workflow update - 2026-07-22T16:04:44.622Z
- Recorded fork run: jrrp6b0onmss
- Validation: castor test:llm-real --filter=ConfiguredModelAgentRunnerLiveTest: PASS (1 test / 4 assertions).; Focused after-turn/serializer/bridge tests: PASS (35 tests / 87 assertions).; castor test: PASS (4456 tests / 15380 assertions).; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS.; Full castor check deferred pending user inspection/task-to-pr.
- Summary: User-approved OM-01 polish committed and pushed normally at 565239f555d94e3d42194e42070b55c00aa3871d. Removed NullLogger fallbacks from the configured agent runner and recovery event reader; removed raw exception messages from new OM logs and the touched after-turn bridge; propagated each committed RunEvent's actual turnNo and ISO-8601 createdAt through the existing internal/public after-turn summaries and deferred recovery path. Added one focused live proof showing agent()->run(): void drains the configured provider stream and returns only after native AgentProcessor executes the isolated extension tool with exact arguments. Existing PR #311 updated; no reviewer/status move/full check.

## Task workflow update - 2026-07-22T16:08:39.036Z
- Summary: Superseding OM-01 acceptance criteria approved by user: (1) existing AfterTurnCommit hook exposes only the already-committed hot batch with payload, per-event turnNo, and createdAt, with no historical read or custom boundary/lifecycle framework; (2) SessionEventReaderInterface provides immutable canonical inclusive-range reads solely for recovery/compaction; (3) ExtensionApi exposes a blocking AgentRunnerInterface using exact provider/model strings, Hatfield's configured lazy Symfony Platform, native Agent/AgentProcessor, internally streamed transport, and request-isolated extension tools; (4) no structured-output/result mapping API, custom provider construction, AiModelReference relocation, SessionExistenceChecker, custom lifecycle APIs, or OM product/runtime/storage code; (5) live proof shows run(): void returns after the isolated tool side effect. OM-02 owns Messenger/SQLite/supervision, OM-03 Observer, OM-04 compaction hook/Reflector, OM-05 UX/settings/final docs. The task file's original goal and acceptance text is historical and superseded by this entry.

## Task workflow update - 2026-07-22T16:23:34.536Z
- Validation: Reviewer mandatory reads confirmed: AGENTS.md, testing skill, tests/AGENTS.md, current task including superseding criteria, OM plan.; Reviewer verdict: APPROVE WITH SUGGESTIONS; zero blockers.; Reviewer verified ExtensionApi boundary remains clean and accepted streaming AgentProcessor/tool isolation flow is correct.
- Summary: Independent reviewer inspected current HEAD 565239f555d94e3d42194e42070b55c00aa3871d against origin/main and the superseding accepted OM-01 scope. Verdict: APPROVE WITH SUGGESTIONS; no blocking correctness, security, architecture, or contract issues. Reviewer confirmed all 17 historical inline concerns are resolved by the current redesign/reverts. Non-blocking suggestions only: optionally wrap isolated tool handler exceptions like Symfony Toolbox, clarify generator validation/empty-return semantics in recovery reader, and minor ordering/model-format comments. No out-of-scope changes requested.

## Task workflow update - 2026-07-22T16:25:45.001Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (113.4s).
- Pushed task/om-01-interaction-event-boundaries-projection to origin.
- branch 'task/om-01-interaction-event-boundaries-projection' set up to track 'origin/task/om-01-interaction-event-boundaries-projection'.
- PR already exists: https://github.com/ineersa/agent-core/pull/311
- Validation: castor test: PASS (4456 tests / 15380 assertions).; castor test:llm-real --filter=ConfiguredModelAgentRunnerLiveTest: PASS (1 test / 4 assertions).; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS.; Independent reviewer: APPROVE WITH SUGGESTIONS, no blockers.
- Summary: User approved the minimal OM-01 architecture and requested closeout. Current HEAD 565239f555d94e3d42194e42070b55c00aa3871d is reviewer-approved with no blockers. Accepted scope is the existing hot committed-event batch enrichment, recovery-only canonical reader, and configured internally-streamed blocking agent runner with isolated extension tools; old task criteria are superseded. Existing PR #311 will be updated, not recreated.

## Task workflow update - 2026-07-22T16:40:04.996Z
- Validation: Deterministic castor check: PASS (113.4s).; PR #311 open at https://github.com/ineersa/agent-core/pull/311 with updated title/body.; All 17 review threads resolved; 0 unresolved remain.; Worktree clean; local HEAD equals remote branch at 565239f555d94e3d42194e42070b55c00aa3871d.
- Summary: OM-01 closeout complete for review. Superseding acceptance criteria recorded, independent reviewer approved with no blockers, deterministic castor check passed, branch/remote match at 565239f555d94e3d42194e42070b55c00aa3871d, PR #311 title/body rewritten for the accepted minimal design, all 17 historical inline comments received concrete resolution replies, and all review threads are resolved. Task is now CODE-REVIEW pending user approval/merge decision.

## Task workflow update - 2026-07-22T16:48:17.991Z
- Moved CODE-REVIEW → DONE.
- Merged task/om-01-interaction-event-boundaries-projection into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                               |   9 ++
 depfile.yaml                                       |   8 ++
 docs/session-storage.md                            |  10 ++
 .../Extension/AfterTurnCommitEventSummary.php      |   9 ++
 .../Extension/AfterTurnCommitHookContext.php       |   2 +
 .../DeferredSubagentBatchRecoveryService.php       |   2 +
 .../Extension/Agent/ConfiguredModelAgentRunner.php | 133 +++++++++++++++++
 .../Extension/Agent/IsolatedAgentToolbox.php       |  73 ++++++++++
 .../ExtensionAfterTurnCommitHookSubscriber.php     |  16 ++-
 .../Extension/ExtensionToolRegistryBridge.php      |  15 +-
 .../Session/ExtensionSessionEventReader.php        |  73 ++++++++++
 .../ExtensionApi/Agent/AgentCallRequestDTO.php     |  54 +++++++
 .../ExtensionApi/Agent/AgentRunnerInterface.php    |  29 ++++
 .../ExtensionApi/Agent/AgentToolDTO.php            |  38 +++++
 .../ExtensionApi/ExtensionApiInterface.php         |  23 +++
 .../Lifecycle/AfterTurnCommitEventSummaryDTO.php   |  14 ++
 .../ExtensionApi/Session/SessionEventDTO.php       |  38 +++++
 .../Session/SessionEventReaderException.php        |  18 +++
 .../Session/SessionEventReaderInterface.php        |  26 ++++
 .../AfterTurnCommitSerializerRegressionTest.php    |  10 +-
 .../Agent/ConfiguredModelAgentRunnerLiveTest.php   | 159 +++++++++++++++++++++
 .../ExtensionAfterTurnCommitHookSubscriberTest.php |  33 ++++-
 .../Extension/ExtensionToolRegistryBridgeTest.php  |  24 ++++
 .../FileRewindExtensionIntegrationTest.php         |  13 +-
 .../Extension/InMemoryExtensionApiBridge.php       |  23 +++
 .../TaskWorkflowExtensionIntegrationTest.php       |  11 ++
 26 files changed, 847 insertions(+), 16 deletions(-)
 create mode 100644 src/CodingAgent/Extension/Agent/ConfiguredModelAgentRunner.php
 create mode 100644 src/CodingAgent/Extension/Agent/IsolatedAgentToolbox.php
 create mode 100644 src/CodingAgent/Extension/Session/ExtensionSessionEventReader.php
 create mode 100644 src/CodingAgent/ExtensionApi/Agent/AgentCallRequestDTO.php
 create mode 100644 src/CodingAgent/ExtensionApi/Agent/AgentRunnerInterface.php
 create mode 100644 src/CodingAgent/ExtensionApi/Agent/AgentToolDTO.php
 create mode 100644 src/CodingAgent/ExtensionApi/Session/SessionEventDTO.php
 create mode 100644 src/CodingAgent/ExtensionApi/Session/SessionEventReaderException.php
 create mode 100644 src/CodingAgent/ExtensionApi/Session/SessionEventReaderInterface.php
 create mode 100644 tests/CodingAgent/Extension/Agent/ConfiguredModelAgentRunnerLiveTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/om-01-interaction-event-boundaries-projection.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/om-01-interaction-event-boundaries-projection.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #311 state: MERGED.; Integration checkout clean before DONE transition.
- Summary: User confirmed PR #311 merged. GitHub reports MERGED at 2026-07-22T16:47:48Z with merge commit c90f28c176d0bb4de9f81b8a66674fe3125106e7. Completing task workflow and syncing integration checkout.

## Task workflow update - 2026-07-22T16:50:26.451Z
- Validation: LLM_MODE=true castor check: PASS (308.8s).; deptrac PASS; unit/integration PASS (4451 tests / 15323 assertions); controller replay PASS (9 / 127); TUI replay PASS (36 / 186); llm-real PASS (13 / 173); phpstan PASS; cs-check PASS.; llama-proxy cache guard stable (143 → 143); artifact integrity PASS; QA leak check PASS.; Integration checkout git status: clean.
- Summary: Post-merge integration validation complete. Worktree removed, IDEA exclusions cleaned, task is DONE, integration checkout is clean.
