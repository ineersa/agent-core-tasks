# MCP-04 Dynamic MCP tool registration from catalog

## Goal
Implement dynamic tool exposure from the MCP catalog according to `.pi/plans/mcp-client-implementation-plan.md`.

Goal: discovered MCP tools are visible to the LLM through the existing ToolRegistry/toolbox path before LLM schema resolution.

Scope:
- Read the session MCP catalog from every process that needs tool definitions; do not assume dynamic ToolRegistry state is shared across processes.
- Implement tool name mapping/sanitization using `{server}_{tool}` by default.
- Register MCP tools as dynamic tools with existing `ToolRegistry::addDynamicTool()` or equivalent.
- Preserve mapping from Hatfield tool name back to MCP server/tool name.
- Default MCP tools to sequential execution mode in v1.
- Detect collisions with permanent tools and other MCP tools; diagnose rather than overwrite.
- Ensure registration happens before LLM-visible tool schema resolution.

Depends on: MCP-03.

## Acceptance criteria
- MCP tools from the catalog appear in the active tool set before LLM calls.
- LLM-visible MCP tool names are namespaced/sanitized and schemas come from MCP input schemas.
- Name collisions are handled with clear diagnostics and no silent overwrite.
- Dynamic tool registration works correctly across the multi-process runtime model by using durable catalog data.
- MCP tools default to sequential execution mode.
- Castor validation covers tool registration/schema visibility.

## Workflow metadata
Status: DONE
Branch: task/mcp-04-dynamic-tool-registration
Worktree: /home/ineersa/projects/agent-core-worktrees/mcp-04-dynamic-tool-registration
Fork run: vdm3y7a2nvg7
PR URL: https://github.com/ineersa/agent-core/pull/173
PR Status: merged
Started: 2026-06-19T01:23:45.597Z
Completed: 2026-06-19T02:22:55.630Z

## Work log
- Created: 2026-06-12T18:06:42.104Z

## Task workflow update - 2026-06-19T01:23:40.348Z
- Summary: Plan decisions confirmed by user for MCP-04 dynamic tool registration: (1) first-turn MCP catalog race does not require blocking/waiting; register MCP dynamic tools when a session catalog exists, and missing catalog is acceptable. (2) MCP-04 should add only a handler placeholder/deferred handler for MCP tools; actual request/reply invocation remains MCP-05 scope. (3) collided MCP tool names should be skipped with structured diagnostics/logging, while non-colliding tools still register. (4) registrar must only track/remove MCP-owned dynamic tool names and must not clear unrelated dynamic tools. Implementation should bridge SessionFileMcpToolCatalogStore catalogs into ToolRegistry dynamic tools before LLM-visible tool schema/allowlist resolution, using sequential execution mode by default.
- Implementation plan to carry into task-start: add an MCP tool registration layer under src/CodingAgent/Mcp/Tool; wrap ToolSetResolverInterface so runId-based catalog registration happens before DynamicToolDescriptionProcessor/toolset resolution; wire services and Deptrac layer; add focused tests protecting catalog-to-registry-to-toolbox/schema behavior, collision skip behavior, missing catalog no-op, stale MCP-owned removal, and wrapper registration-before-delegate behavior.

## Task workflow update - 2026-06-19T01:23:45.597Z
- Moved TODO → IN-PROGRESS.
- Created branch task/mcp-04-dynamic-tool-registration.
- Created worktree /home/ineersa/projects/agent-core-worktrees/mcp-04-dynamic-tool-registration.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/mcp-04-dynamic-tool-registration.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/mcp-04-dynamic-tool-registration.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/mcp-04-dynamic-tool-registration.
- Summary: Starting MCP-04 implementation after user confirmed scope decisions: register only when catalog exists (no first-turn blocking), add handler placeholder only (MCP-05 owns request/reply invocation), skip collided tool names with diagnostics, and track only MCP-owned dynamic tools for stale cleanup.

## Task workflow update - 2026-06-19T01:24:39.602Z
- Recorded fork run: rte5abyyixd3
- Launched implementation fork rte5abyyixd3 in worktree /home/ineersa/projects/agent-core-worktrees/mcp-04-dynamic-tool-registration. Fork instructions include user-confirmed scope decisions, required testing docs preload, focused MCP tool registration implementation plan, Castor-only validation, and commit/handoff requirements.

## Task workflow update - 2026-06-19T01:31:30.660Z
- Recorded fork run: rte5abyyixd3
- Validation: Fork reported: castor test --filter=McpToolRegistrarTest OK (9 tests, 35 assertions); Fork reported: castor test --filter=McpCatalogRegisteringToolSetResolverTest OK (3 tests, 4 assertions); Fork reported: castor test --filter=McpTool OK (30 tests, 93 assertions); Fork reported: castor test --filter=Mcp OK (75 tests, 272 assertions); Fork reported: castor deptrac OK (violations=0, errors=0); Fork reported: castor phpstan OK (errors=0, file_errors=0); Fork reported: castor cs-check OK (files_fixed=0, clean); Parent verification: git status clean; HEAD=7c52c76d9; diff stat 7 files changed, 931 insertions, 1 deletion. Full deterministic castor check not run yet; per workflow it remains for task-to-pr/CODE-REVIEW phase.
- Summary: Implementation fork completed and committed. Commit 7c52c76d9 (MCP-04: Dynamic MCP tool registration from session catalog). Verified worktree is clean and diff stat matches expected scope: config/services.yaml, depfile.yaml, new src/CodingAgent/Mcp/Tool/{McpToolHandler,McpToolRegistrar,McpCatalogRegisteringToolSetResolver}.php, and new focused tests under tests/CodingAgent/Mcp/Tool/. The implementation bridges MCP session catalogs into ToolRegistry dynamic tools via a ToolSetResolver wrapper, skips collisions with structured warnings, preserves unrelated dynamic tools during stale cleanup, and uses a deferred non-retryable MCP tool handler until MCP-05.

## Task workflow update - 2026-06-19T01:52:21.489Z
- Recorded fork run: pw8q4lfa5xw4
- Summary: Reviewer returned APPROVE WITH SUGGESTIONS for commit 7c52c76d9. Actionable findings being addressed before CODE-REVIEW: guard McpCatalogRegisteringToolSetResolver so optional MCP catalog registration failures do not violate ToolSetResolverInterface resolve() contract; add run_id/session_id to MCP registrar warning logs; add/adjust focused tests for catch-and-continue/log correlation behavior. Launched fix fork pw8q4lfa5xw4 in the task worktree.

## Task workflow update - 2026-06-19T01:56:14.246Z
- Recorded fork run: pw8q4lfa5xw4
- Validation: Fork reported: castor test --filter=McpToolRegistrarTest OK (10 tests, 47 assertions); Fork reported: castor test --filter=McpCatalogRegisteringToolSetResolverTest OK (4 tests, 11 assertions); Fork reported: castor test --filter=McpTool OK (31 tests, 105 assertions); Fork reported: castor test --filter=Mcp OK (77 tests, 291 assertions); Fork reported: castor deptrac OK (violations=0, errors=0); Fork reported: castor phpstan OK (errors=0, file_errors=0); Fork reported: castor cs-check OK (files_fixed=0, clean); Fork reported: debug:container McpToolHandler shows Autowired: no
- Summary: Review-fix fork completed and committed. Commit a090799b4 addresses reviewer suggestions: resolver now catches/logs optional MCP registration failures before delegating, registrar warning logs include run_id/session_id, hidden permanent-tool collision path is documented and tested, and McpToolHandler is excluded from autowiring as a value object. Parent verification: worktree clean at a090799b4; diff stat now 7 files changed, 1134 insertions, 1 deletion.

## Task workflow update - 2026-06-19T02:03:51.033Z
- Recorded fork run: vdm3y7a2nvg7
- Summary: Second reviewer pass returned APPROVE WITH SUGGESTIONS. Remaining actionable minor items: add canonical event_type to McpToolRegistrar warning contexts and remove a duplicated test assertion. Launched minimal cleanup fork vdm3y7a2nvg7.

## Task workflow update - 2026-06-19T02:06:14.232Z
- Recorded fork run: vdm3y7a2nvg7
- Validation: Fork reported: castor test --filter=McpToolRegistrarTest OK (10 tests, 50 assertions); Fork reported: castor test --filter=McpCatalogRegisteringToolSetResolverTest OK (4 tests, 10 assertions); Fork reported: castor cs-check OK (files_fixed=0, clean)
- Summary: Final cleanup fork completed and committed. Commit fc080d71e adds canonical event_type to McpToolRegistrar warning log contexts, removes a duplicated resolver-test assertion, and extends existing registrar log assertions. Parent verification: worktree clean at fc080d71e; diff stat now 7 files changed, 1138 insertions, 1 deletion.

## Task workflow update - 2026-06-19T02:13:16.485Z
- Validation: Reviewer subagent final decision: APPROVED; Focused local validation in worktree: castor test OK (2831 tests, 8539 assertions); Focused local validation in worktree: castor deptrac OK (violations=0, errors=0); Focused local validation in worktree: castor phpstan OK (errors=0, file_errors=0); Focused local validation in worktree: castor cs-check OK (files_fixed=0); Focused live LLM validation in worktree: castor test:llm-real OK (5 tests, 51 assertions; llama.cpp generation preflight OK); Before CODE-REVIEW castor check, attempted stale worker cleanup. Found pid 3414 messenger:consume but kill returned Operation not permitted, likely not owned by this session; proceeding with task workflow gate.
- Summary: Final reviewer subagent decision on current HEAD fc080d71e: APPROVED. Reviewer verified final suggestions were addressed, scanned full origin/main...HEAD diff, and found no blocking issues. Current commit chain: 7c52c76d9 initial MCP-04 implementation, a090799b4 review fixes, fc080d71e final log/test cleanup. Worktree is clean.

## Task workflow update - 2026-06-19T02:14:23.562Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (53.7s).
- Pushed task/mcp-04-dynamic-tool-registration to origin.
- branch 'task/mcp-04-dynamic-tool-registration' set up to track 'origin/task/mcp-04-dynamic-tool-registration'.
- Created PR: https://github.com/ineersa/agent-core/pull/173
- Validation: Reviewer subagent final decision: APPROVED; castor test OK (2831 tests, 8539 assertions); castor deptrac OK (violations=0, errors=0); castor phpstan OK (errors=0, file_errors=0); castor cs-check OK (files_fixed=0); castor test:llm-real OK (5 tests, 51 assertions; generation preflight OK)
- Summary: Prepared MCP-04 for code review. Final reviewer decision: APPROVED. HEAD fc080d71e. Focused validations passed: castor test, castor deptrac, castor phpstan, castor cs-check, and castor test:llm-real. Moving to CODE-REVIEW to run deterministic castor check, push branch, and create PR.

## Task workflow update - 2026-06-19T02:22:55.630Z
- Moved CODE-REVIEW → DONE.
- Merged task/mcp-04-dynamic-tool-registration into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                               |  34 +-
 depfile.yaml                                       |   9 +
 .../Tool/McpCatalogRegisteringToolSetResolver.php  |  76 +++
 src/CodingAgent/Mcp/Tool/McpToolHandler.php        |  35 ++
 src/CodingAgent/Mcp/Tool/McpToolRegistrar.php      | 224 +++++++++
 .../McpCatalogRegisteringToolSetResolverTest.php   | 236 +++++++++
 .../CodingAgent/Mcp/Tool/McpToolRegistrarTest.php  | 525 +++++++++++++++++++++
 7 files changed, 1138 insertions(+), 1 deletion(-)
 create mode 100644 src/CodingAgent/Mcp/Tool/McpCatalogRegisteringToolSetResolver.php
 create mode 100644 src/CodingAgent/Mcp/Tool/McpToolHandler.php
 create mode 100644 src/CodingAgent/Mcp/Tool/McpToolRegistrar.php
 create mode 100644 tests/CodingAgent/Mcp/Tool/McpCatalogRegisteringToolSetResolverTest.php
 create mode 100644 tests/CodingAgent/Mcp/Tool/McpToolRegistrarTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/mcp-04-dynamic-tool-registration.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/mcp-04-dynamic-tool-registration.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: CODE-REVIEW gate previously passed: deterministic castor check passed in 53.7s during PR creation; Focused validations previously passed: castor test, castor deptrac, castor phpstan, castor cs-check, castor test:llm-real
- Summary: PR #173 was reported merged by user. Moving MCP-04 to DONE and cleaning up task worktree.
