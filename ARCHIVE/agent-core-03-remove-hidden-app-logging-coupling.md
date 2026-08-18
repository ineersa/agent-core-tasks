# agent-core-03: Remove hidden App logging coupling from AgentCore

## Goal
## Goal
Remove AgentCore's string-literal dependency on CodingAgent handler class names while preserving structured logging component attribution.

## Full report
[`/home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md`](file:///home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md) — AgentCore candidate 6.

## Architecture report evidence
`RunMessageProcessor::componentForHandler()` matches literal App FQCN strings for `Ineersa\CodingAgent\Application\Pipeline\CompactRunHandler` and `CompactionStepResultHandler`. Deptrac cannot see string coupling. Renaming either App class silently falls back to the `runtime` component, degrading logs without an error.

The report proposed `RunMessageHandler::logComponent()`, but that is a candidate design, not a finalized requirement. Adding a method to every handler or a one-implementation provider interface may cost more than the two literals it removes.

## Required design gate
Trace all `RunMessageHandler` implementations, registration, extension/public compatibility, and structured-log consumers. Choose the smallest ownership model that:
- lets App-owned compaction handlers declare or register their component without Core naming App classes,
- keeps the default `runtime` attribution cheap,
- fails visibly or remains type-safe on rename,
- does not force unrelated handlers to carry boilerplate.
Possible mechanics include handler-owned metadata, typed registration metadata, or message/processor configuration; do not assume a new interface method is best.

## Scope boundaries
- Preserve exact `component`, `run_id`, `session_id`, and event-style logging behavior.
- Do not move logging into domain entities or introduce reflection/string routers, attributes solely for two classes, a service locator, or a generic metadata framework unless it is demonstrably smaller.
- No new public ExtensionApi surface, runtime event, setting, command, or log content containing prompts/session data.
- Candidate 7 is excluded: the active `reuse-symfony-components-outside-tui` task already removes the inert HookDispatcher Serializer/EventDispatcher bridge. Rebase around that work; do not duplicate it.

## Test thesis
Rename-safe component attribution should be tested through real handler registration/processing: compaction remains `compaction`, ordinary handlers remain `runtime`, and no Core source references CodingAgent FQCNs.

## Acceptance criteria
- AgentCore contains no CodingAgent handler FQCN literals or equivalent hidden App-name matching for log attribution.
- Compaction handlers retain the `compaction` component and ordinary handlers retain the `runtime` default.
- The chosen ownership mechanism is rename-safe, type-safe where practical, and adds no boilerplate to unrelated handlers unless unavoidable.
- Structured logs preserve correlation fields and privacy constraints.
- No reflection router, string metadata switch, generic provider hierarchy, setting, public API, or compatibility shim is introduced.
- HookDispatcher work from the active Symfony-reuse task is not duplicated or reverted.
- A focused regression proves App compaction and Core/default component attribution through the production registration path.
- The fork and handoff state that `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` were read and followed.
- Focused Castor tests, `castor test`, `castor deptrac`, `castor phpstan`, `castor cs-check`, and final `castor check` pass.

## Workflow metadata
Status: ARCHIVE
Branch: task/agent-core-03-remove-hidden-app-logging-coupling
Worktree: /home/ineersa/projects/agent-core-worktrees/agent-core-03-remove-hidden-app-logging-coupling
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/393
PR Status: merged
Started: 2026-08-16T15:49:37.190Z
Completed: 2026-08-16T18:03:18.135Z

## Work log
- Created: 2026-08-15T23:28:18.389Z

## Task workflow update - 2026-08-16T15:49:37.190Z
- Moved TODO → IN-PROGRESS.
- Created branch task/agent-core-03-remove-hidden-app-logging-coupling.
- Created worktree /home/ineersa/projects/agent-core-worktrees/agent-core-03-remove-hidden-app-logging-coupling.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/agent-core-03-remove-hidden-app-logging-coupling.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/agent-core-03-remove-hidden-app-logging-coupling.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/agent-core-03-remove-hidden-app-logging-coupling.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/agent-core-03-remove-hidden-app-logging-coupling.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/agent-core-03-remove-hidden-app-logging-coupling/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/agent-core-03-remove-hidden-app-logging-coupling.
- Summary: Starting next sequential architecture task after agent-core-02. Scope is limited to removing AgentCore's hidden CodingAgent class-name matching while preserving compaction/runtime structured-log attribution.

## Task workflow update - 2026-08-16T15:57:46.920Z
- Validation: Scout read and followed root/nested AGENTS.md, `.agents/skills/testing/SKILL.md`, and `tests/AGENTS.md`.; IDE implementation search confirmed exactly 8 RunMessageHandler implementations and no external/ExtensionApi implementations.; Specification gate: no user-facing surface or unresolved product decision; preserve existing component and correlation fields exactly.
- Summary: Scout completed the design gate. Eight implementations exist: four default runtime, LLM, tool, and two App compaction handlers. Registration is a Symfony tagged iterator; no ExtensionApi implementation exists. Hidden mapping is used only by RunMessageProcessor. Smallest mechanism is handler-owned typed metadata with zero boilerplate for defaults. Implementation should prefer a default `RunMessageHandler` interface constant overridden only by LLM/tool/compaction handlers if PHP/project conventions support it; this is smaller than the scout's optional four-implementation capability interface. Production registration-path regression required.

## Task workflow update - 2026-08-16T16:10:46.593Z
- Validation: Fork explicitly read and followed root/nested AGENTS.md, `.agents/skills/testing/SKILL.md`, and `tests/AGENTS.md`.; castor test --filter=RunMessageProcessorLogComponentTest — PASS (3 tests, 28 assertions).; castor test — PASS (4487 tests, 17608 assertions).; castor test:controller-replay — PASS (12 tests, 165 assertions).; castor deptrac — PASS (0 violations, 0 errors).; castor phpstan — PASS (0 errors).; castor cs-check — PASS (0 files fixed after one import-order cs-fix pass).; git diff --check — clean.; castor check intentionally deferred to task-to-pr.
- Summary: Implementation committed as 05bba15c7 on task/agent-core-03-remove-hidden-app-logging-coupling. Used the smallest typed ownership model: `RunMessageHandler::LOG_COMPONENT` defaults to `runtime`; only LLM, tool, and two App compaction handlers override it. RunMessageProcessor now reads `$handler::LOG_COMPONENT`; deleted the 8-class FQCN router. Added one production-registration/processing regression covering both compaction handlers, ordinary runtime attribution, LLM/tool declarations, correlation context, and Monolog context processing. 7 files changed, +289/-16; production change is +20/-16 and one 269-line integration test. Worktree clean, commit not pushed, no PR. IDE diagnostics show 0 errors in RunMessageHandler, RunMessageProcessor, and CompactRunHandler.

## Task workflow update - 2026-08-16T16:35:40.266Z
- Validation: Initial reviewer read/followed testing skill and tests/AGENTS.md; REQUEST CHANGES had no production blocker.; Fix commit 001849d50 focused test — PASS (3 tests, 21 assertions); phpstan/cs-check/diff-check passed.; Re-review read/followed testing skill and tests/AGENTS.md; verdict APPROVED.; Final castor test — PASS (4487 tests, 17601 assertions).; Final castor deptrac — PASS (0 violations, 0 errors).; Final castor phpstan — PASS (0 errors).; Final castor cs-check — PASS (0 files fixed).; Earlier castor test:controller-replay — PASS (12 tests, 165 assertions).
- Summary: task-to-pr review completed. Initial reviewer REQUEST CHANGES targeted only test bloat/brittleness. Fix commit 001849d50 removed the handler-count canary and duplicate registration matrix, replaced an unused real replay fixture graph with a minimal valid fake, and corrected the test name (-18 net lines). Re-review verdict: APPROVED. Cumulative implementation commits: 05bba15c7 + 001849d50; production mechanism unchanged and approved.

## Task workflow update - 2026-08-16T16:38:06.113Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (134.7s).
- Pushed task/agent-core-03-remove-hidden-app-logging-coupling to origin.
- branch 'task/agent-core-03-remove-hidden-app-logging-coupling' set up to track 'origin/task/agent-core-03-remove-hidden-app-logging-coupling'.
- Created PR: https://github.com/ineersa/agent-core/pull/393
- Validation: Focused and full validation passed: RunMessageProcessorLogComponentTest, castor test, controller replay, deptrac, phpstan, cs-check, git diff --check.; Worktree clean.
- Summary: Reviewer APPROVED cumulative commits 05bba15c7 and 001849d50. AgentCore no longer names CodingAgent handler classes for component attribution; handler-owned typed constants preserve runtime/llm/tool/compaction values.

## Task workflow update - 2026-08-16T17:52:08.674Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User requested PR iteration: replace the public `RunMessageHandler::LOG_COMPONENT` constant/dynamic class-constant dispatch with a clearer optional interface exposing `logComponent(): string`. Preserve runtime default and all existing attribution values.

## Task workflow update - 2026-08-16T17:55:30.818Z
- Validation: Fork read/followed testing skill and tests/AGENTS.md.; castor test --filter=RunMessageProcessorLogComponentTest — PASS (3 tests, 23 assertions).; castor deptrac — PASS (0 violations, 0 errors).; castor phpstan — PASS (0 errors).; castor cs-check — PASS after one cs-fix formatting pass.; git diff --check — clean.
- Summary: User-requested PR iteration implemented as commit 2af23121f. Replaced the rejected public typed constant mechanism with optional `RunMessageHandlerLogComponentInterface::getLogComponent(): string`; only LLM, tool, and two compaction handlers implement it, while ordinary handlers retain a direct `runtime` fallback. `LOG_COMPONENT` removed completely. PR #393 remains open and branch update is local pending re-review.

## Task workflow update - 2026-08-16T17:59:44.229Z
- Validation: Iteration reviewer read/followed testing skill and tests/AGENTS.md; verdict APPROVED.; Final castor test — PASS (4487 tests, 17603 assertions).; Final castor deptrac — PASS (0 violations, 0 errors).; Final castor phpstan — PASS (0 errors).; Final castor cs-check — PASS (0 files fixed).; Focused log-component test — PASS (3 tests, 23 assertions).; IDE diagnostics — 0 errors in new interface and RunMessageProcessor.; Worktree clean; git diff --check clean.
- Summary: User-requested interface revision commit 2af23121f re-reviewed and APPROVED. Public/dynamic `LOG_COMPONENT` constants are gone; optional `RunMessageHandlerLogComponentInterface::getLogComponent()` is implemented only by LLM/tool/compaction handlers, with ordinary handlers using the runtime fallback. Ready to update PR #393.

## Task workflow update - 2026-08-16T18:02:09.179Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (135.3s).
- Pushed task/agent-core-03-remove-hidden-app-logging-coupling to origin.
- branch 'task/agent-core-03-remove-hidden-app-logging-coupling' set up to track 'origin/task/agent-core-03-remove-hidden-app-logging-coupling'.
- PR already exists: https://github.com/ineersa/agent-core/pull/393
- Validation: Focused/full tests, deptrac, phpstan, cs-check, IDE diagnostics, and diff-check passed.; Worktree clean.
- Summary: Updated per user feedback: commit 2af23121f replaces typed constants with optional `RunMessageHandlerLogComponentInterface::getLogComponent()`. Re-review APPROVED; exact attribution preserved.

## Task workflow update - 2026-08-16T18:03:18.135Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/agent-core-03-remove-hidden-app-logging-coupling.
- Merged task/agent-core-03-remove-hidden-app-logging-coupling into integration checkout.
- Merge made by the 'ort' strategy.
 .../Application/Pipeline/LlmStepResultHandler.php  |   7 +-
 .../RunMessageHandlerLogComponentInterface.php     |  17 ++
 .../Application/Pipeline/RunMessageProcessor.php   |  22 +-
 .../Application/Pipeline/ToolCallResultHandler.php |   7 +-
 .../Application/Pipeline/CompactRunHandler.php     |   8 +-
 .../Pipeline/CompactionStepResultHandler.php       |   8 +-
 .../RunMessageProcessorLogComponentTest.php        | 268 +++++++++++++++++++++
 7 files changed, 317 insertions(+), 20 deletions(-)
 create mode 100644 src/AgentCore/Application/Pipeline/RunMessageHandlerLogComponentInterface.php
 create mode 100644 tests/AgentCore/Application/Pipeline/RunMessageProcessorLogComponentTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/agent-core-03-remove-hidden-app-logging-coupling.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/agent-core-03-remove-hidden-app-logging-coupling.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR https://github.com/ineersa/agent-core/pull/393 state: MERGED.; Pre-merge integration checkout clean.
- Summary: PR #393 confirmed merged on GitHub at merge commit 9df042be3d2a5469fc9bc8b94060b2cd51be505c. Moving task to DONE and syncing integration checkout.

## Task workflow update - 2026-08-16T18:06:05.276Z
- Validation: LLM_MODE=true castor check — PASS (QA run qa-20260816-180336-332490-f1d0e056).; All eight lanes passed: deptrac, test (4487 tests/17603 assertions), controller replay, TUI replay, llm-real, phpstan, cs-check, docs validation.; QA leak check passed; llama-proxy cache remained stable at 306 entries.
- Summary: Post-merge integration validation complete. The user-requested optional `getLogComponent()` interface design is merged. Worktree cleanup completed.

## Task workflow update - 2026-08-18T00:06:36.249Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
