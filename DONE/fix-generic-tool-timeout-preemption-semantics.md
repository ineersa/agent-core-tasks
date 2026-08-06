# Make generic tool timeout preempt or remove post-hoc timeout semantics

## Goal
User identified a broad runtime bug while diagnosing session 7: `ToolExecutor` generic `tools.execution.timeout_seconds` is post-hoc, not preemptive. The executor currently waits for a tool handler to return, measures elapsed time, and may rewrite an otherwise successful result as `Tool "..." timed out`; it does not interrupt/kill a still-running handler. This means any tool that relies only on generic ToolExecutor timeout can keep running indefinitely, and side effects can complete before being reported as timeout.

Separate from the immediate subagent-specific fix. Investigate desired semantics and implement one of:
- real cooperative/preemptive timeout infrastructure with cancellation/deadline propagation and tool cleanup contracts, or
- remove/rename the generic post-hoc timeout so it is not represented as a kill/cancellation timeout, leaving real timeouts to tool-specific implementations (`ToolRuntime`, MCP client timeouts, subagent poll deadline, etc.).

Do not silently keep a setting named timeout if it only performs late SLA validation. Preserve structured diagnostics and cancellation behavior.

## Acceptance criteria
- Document current behavior and decide the intended contract for `tools.execution.timeout_seconds`.
- If kept as timeout, in-progress tools are actively cancelled/stopped or cooperatively interrupted on deadline, with tests proving a hung/long-running tool is stopped before handler completion.
- If not made preemptive, remove or rename post-hoc timeout behavior so successful long-running tools are not rewritten as timeout after completion.
- Existing tool-specific timeouts (bash/ToolRuntime, MCP, subagent internal deadline) remain intact.
- Regression tests cover at least one generic long-running tool/hanging handler scenario and one successful long-running result scenario.
- Use Castor for all QA; load testing skill and read tests/AGENTS.md before test work.

## Workflow metadata
Status: DONE
Branch: task/fix-generic-tool-timeout-preemption-semantics
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-generic-tool-timeout-preemption-semantics
Fork run: yyrft8jpo3wl
PR URL: https://github.com/ineersa/agent-core/pull/344
PR Status: merged
Started: 2026-07-31T21:10:00.450Z
Completed: 2026-07-31T23:56:04.051Z

## Work log
- Created: 2026-07-04T21:32:49.679Z

## Task workflow update - 2026-07-31T21:10:00.450Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-generic-tool-timeout-preemption-semantics.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-generic-tool-timeout-preemption-semantics.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-generic-tool-timeout-preemption-semantics.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-generic-tool-timeout-preemption-semantics.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-generic-tool-timeout-preemption-semantics.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-generic-tool-timeout-preemption-semantics.
- Summary: Started after task-explain discussion. Finalized scope: remove ToolExecutor post-hoc timeout rewrite; retain ToolContext cancellation/timeout as cooperative contract; expose cancellation/deadline through public ExtensionApi with thorough author documentation; fix per-tool timeout metadata propagation; preserve existing Bash/subagent/fork/MCP-specific mechanisms. Proper in-flight MCP cancellation and current-extension migrations are tracked separately.

## Task workflow update - 2026-07-31T21:11:18.427Z
- Recorded fork run: h849w9ahmky9
- Implementation fork launched with finalized scope: remove post-hoc ToolExecutor timeout rewriting; preserve cooperative ToolContext budget; add public ExtensionApi cancellation/deadline context; fix timeout metadata propagation; document extension author contract. MCP in-flight cancellation and current-extension migrations remain separate follow-ups.

## Task workflow update - 2026-07-31T21:18:33.861Z
- Validation: castor test --filter=ToolExecutorTest → OK (18 tests); castor test --filter='ExtensionToolRegistryBridgeTest|ToolRegistryTest|CodingAgentToolSetResolverTest|ToolExecutionPolicyResolverTest|ToolRuntimeTest' → OK (104 tests); castor test --filter='ExtensionToolRegistryBridgeTest|ToolExecutorTest|ToolRegistryTest|CodingAgentToolSetResolverTest|ToolExecutionPolicyResolverTest|ToolRuntimeTest' → OK (122 tests); castor deptrac → violations=0; castor phpstan → errors=0,file_errors=0; castor cs-check → files_fixed=0 after cs-fix; Read testing skill + tests/AGENTS.md before tests
- Summary: Implemented cooperative timeout contract: removed ToolExecutor post-hoc timeout rewrite; timeoutSeconds remains ToolContext budget; fixed registry/extension timeout propagation; extended ExtensionApi with ToolCancellationTokenInterface + optional cancellation/timeout on ToolInvocationContextDTO and ToolRegistrationDTO; documented contract in tool-execution/settings. Commit c2ea2cb04.

## Task workflow update - 2026-07-31T21:19:17.819Z
- Recorded fork run: h849w9ahmky9
- Validation: Fork confirmed it read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before test work.; `castor test --filter=ToolExecutorTest` — OK, 18 tests.; Focused combined registry/extension/policy/ToolRuntime tests — OK, 104; combined re-run with ToolExecutor — OK, 122.; `castor deptrac` — 0 violations.; `castor phpstan` — 0 errors.; `castor cs-check` — clean after focused `castor cs-fix`.; Parent verified clean worktree and commit c2ea2cb04; diff is 22 files, +393/-49.; `castor check` intentionally not run during task-start; required in task-to-pr gate.
- Summary: Implementation complete and committed at c2ea2cb042bef3a827227d55dbcebd603c126c16. Removed ToolExecutor's post-hoc timeout failure rewrite; retained cooperative timeout/duration metadata; fixed per-tool timeout propagation; added public ExtensionApi cancellation token and optional timeout budget with internal boundary adapter; documented extension-author cancellation/deadline patterns. Worktree is clean. Proper MCP cancellation and migration of current extensions remain separate tracked tasks.

## Task workflow update - 2026-07-31T21:31:04.529Z
- Recorded fork run: ybr966pnqcjf
- Validation: Task-to-PR full `castor test` found 1 error: `ExtensionOwnerFilteringTest` omitted the new `StackToolExecutionContextAccessor` constructor dependency for `ExtensionToolRegistryBridge`; 1521 tests / 5822 assertions reached before failure.
- Reviewer approved commit c2ea2cb04 with no blockers and specification fidelity PASS. Full focused-suite validation then exposed one missed test construction site; launched narrow fix fork ybr966pnqcjf.

## Task workflow update - 2026-07-31T21:36:46.546Z
- Recorded fork run: ybr966pnqcjf
- Validation: Fix fork read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`.; `castor test --filter=ExtensionOwnerFilteringTest` — OK, 2 tests / 17 assertions.; Full `castor test` — OK, 4399 tests / 16228 assertions.; `castor deptrac` — 0 violations / 0 errors.; `castor phpstan` — 0 errors.; `castor cs-check` — clean.; Final reviewer on HEAD 53b55b958 — APPROVED; all direct ExtensionToolRegistryBridge construction sites verified current; public surface maps to finalized requirements.
- Summary: Final branch HEAD 53b55b958. Narrow follow-up fixed the only stale direct ExtensionToolRegistryBridge constructor site in ExtensionOwnerFilteringTest; no production semantics changed. Final reviewer APPROVED the complete diff with specification fidelity PASS and no blockers or nice-to-haves.

## Task workflow update - 2026-07-31T21:38:38.970Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (103.0s).
- Pushed task/fix-generic-tool-timeout-preemption-semantics to origin.
- branch 'task/fix-generic-tool-timeout-preemption-semantics' set up to track 'origin/task/fix-generic-tool-timeout-preemption-semantics'.
- Created PR: https://github.com/ineersa/agent-core/pull/344

## Task workflow update - 2026-07-31T23:08:24.621Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User review clarification supersedes prior cooperative-global-budget scope: remove `tools.execution.timeout_seconds` entirely because it cannot guarantee interruption. Preserve real tool-owned Bash/subagent/fork/MCP timeouts, cancellation tokens, per-tool extension timeout budgets, and document that potentially blocking tools must own and enforce timeout/cleanup. PR #344 has no external comments or reviews yet.

## Task workflow update - 2026-07-31T23:15:05.753Z
- Recorded fork run: 7tqr673b78oo
- Task-review iteration launched after user finalized cleaner semantics: delete global `tools.execution.timeout_seconds` and all fallback plumbing; retain explicit per-tool ExtensionApi/ToolContext budgets plus Bash/subagent/fork/MCP tool-owned timeouts and cancellation. PR #344 had no GitHub comments; user feedback is the blocker.

## Task workflow update - 2026-07-31T23:27:09.964Z
- Recorded fork run: yyrft8jpo3wl
- Validation: Iteration fork read testing skill and tests/AGENTS.md.; Focused affected suite — OK, 153 tests / 490 assertions.; Full `castor test` — OK, 4396 tests / 16228 assertions.; `castor deptrac` — 0 violations / 0 errors.; `castor phpstan` — 0 errors.; `castor cs-check` — clean.; Search proof: global setting remains only as explicit denial in docs; no global `defaultTimeoutSeconds` product symbol remains outside Bash-owned config.
- Summary: Global timeout removal committed at 0babd7dfb with full focused validation green. Reviewer found the specification fully satisfied but requested one ponytail simplification: inline the now-trivial single-use ToolExecutor::resolveTimeoutSeconds helper. Narrow fix fork yyrft8jpo3wl launched.

## Task workflow update - 2026-07-31T23:31:14.320Z
- Recorded fork run: yyrft8jpo3wl
- Validation: Global-removal iteration: full `castor test` OK — 4396 tests / 16228 assertions; deptrac 0; phpstan 0; cs-check clean.; Final micro-fix fork read testing skill and tests/AGENTS.md.; After micro-fix: `castor test --filter=ToolExecutorTest` — OK, 16 tests / 64 assertions.; After micro-fix: `castor phpstan` — 0 errors; `castor cs-check` — clean.; Final reviewer on HEAD f8f60dccf — APPROVED; prior blocker resolved; complete final diff satisfies latest user specification.
- Summary: Final iteration HEAD f8f60dccf. Removed global tools.execution.timeout_seconds and fallback plumbing, retained explicit per-tool timeout propagation/public extension cancellation context and all tool-owned Bash/subagent/fork/MCP/ToolRuntime controls. Reviewer-requested single-use timeout normalization helper was inlined without behavior change. Final reviewer APPROVED with specification fidelity PASS and no blockers.

## Task workflow update - 2026-07-31T23:33:04.085Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (101.4s).
- Pushed task/fix-generic-tool-timeout-preemption-semantics to origin.
- branch 'task/fix-generic-tool-timeout-preemption-semantics' set up to track 'origin/task/fix-generic-tool-timeout-preemption-semantics'.
- PR already exists: https://github.com/ineersa/agent-core/pull/344

## Task workflow update - 2026-07-31T23:56:04.051Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-generic-tool-timeout-preemption-semantics into integration checkout.
- Auto-merging src/AgentCore/Application/Pipeline/LlmStepResultHandler.php
Merge made by the 'ort' strategy.
 .pi/plans/toolbox-design-plan.md                   |   1 -
 config/hatfield.defaults.yaml                      |   7 +-
 docs/settings.md                                   |  18 +--
 docs/tool-execution.md                             | 144 ++++++++++++++++++++-
 .../Handler/ToolExecutionPolicyResolver.php        |  10 +-
 src/AgentCore/Application/Handler/ToolExecutor.php |  50 ++-----
 .../Application/Pipeline/LlmStepResultHandler.php  |   2 +-
 src/AgentCore/Contract/Tool/ActiveToolSet.php      |   5 +-
 .../Tool/ToolExecutionSettingsInterface.php        |   2 -
 src/CodingAgent/Config/ToolExecutionConfig.php     |   9 +-
 src/CodingAgent/Config/ToolSettings.php            |   9 --
 .../ExtensionToolCancellationTokenAdapter.php      |  27 ++++
 .../Extension/ExtensionToolHandlerAdapter.php      |  13 +-
 .../Extension/ExtensionToolRegistryBridge.php      |   1 +
 .../ContextualExtensionToolHandlerInterface.php    |   4 +-
 .../Tool/ToolCancellationTokenInterface.php        |  17 +++
 .../ExtensionApi/Tool/ToolInvocationContextDTO.php |  15 ++-
 .../ExtensionApi/Tool/ToolRegistrationDTO.php      |   5 +
 src/CodingAgent/Tool/ToolDefinitionDTO.php         |   2 +-
 src/CodingAgent/Tool/ToolRegistry.php              |   3 +
 src/CodingAgent/Tool/ToolRegistryInterface.php     |   2 +
 .../Handler/ToolCallHumanInputSuspensionTest.php   |   2 +-
 .../Handler/ToolExecutionPolicyResolverTest.php    |  23 ++--
 .../Application/Handler/ToolExecutorTest.php       |  79 ++---------
 .../Extension/ExtensionToolRegistryBridgeTest.php  |  86 +++++++++++-
 .../Tool/CodingAgentToolSetResolverTest.php        |  32 +++++
 tests/CodingAgent/Tool/ToolRegistryTest.php        |  17 +++
 tests/CodingAgent/Tool/ViewImageToolTest.php       |   1 -
 28 files changed, 407 insertions(+), 179 deletions(-)
 create mode 100644 src/CodingAgent/Extension/ExtensionToolCancellationTokenAdapter.php
 create mode 100644 src/CodingAgent/ExtensionApi/Tool/ToolCancellationTokenInterface.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-generic-tool-timeout-preemption-semantics.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-generic-tool-timeout-preemption-semantics.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #344 confirmed merged on GitHub at b6110e63d64afd28a278763d01bf30b9a4297912 on 2026-07-31T23:55:33Z. Final behavior removes the misleading global tool timeout, preserves tool-owned timeout/cancellation controls, and exposes explicit per-tool cancellation/deadline context to extensions.

## Task workflow update - 2026-07-31T23:58:10.549Z
- Validation: GitHub PR #344 state MERGED; merge commit b6110e63d64afd28a278763d01bf30b9a4297912.; Post-merge `LLM_MODE=true castor check` — quality OK.; Unit/integration: 4399 tests / 16247 assertions.; Controller replay: 11 tests / 160 assertions.; TUI replay: 30 tests / 185 assertions.; Live LLM: 13 tests / 144 assertions.; Deptrac, PHPStan, CS check, llama-proxy cache guard, artifact integrity, and QA leak check all passed.; Integration checkout clean; task worktree removed.
- Summary: DONE after PR #344 merge. move_task merged/synced integration checkout and removed the task worktree/IDE exclusions. Post-merge deterministic full QA passed on integration main.
