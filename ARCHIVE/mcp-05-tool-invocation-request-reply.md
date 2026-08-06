# MCP-05 Tool invocation through broker request/reply

## Goal
Implement MCP tool calls from normal tool workers through the single MCP broker, as designed in `.pi/plans/mcp-client-implementation-plan.md`.

Goal: when the LLM calls a dynamic MCP tool, the normal ToolExecutor invokes an MCP handler that sends a correlated request to the broker, waits for the result, maps it, and returns through the existing tool pipeline.

Scope:
- Implement broker request/reply messages and correlation IDs for MCP `callTool`.
- Implement a result store for correlated results, preferably DB-backed unless a locked session file store is deliberately chosen.
- Implement tool-worker-side `McpToolHandler` with synchronous wait/poll and timeout.
- Implement broker-side call handler using `McpConnectionManager::callTool()` / SDK `Client::callTool()`.
- Implement `McpResultMapper` from MCP content blocks/errors to normal Hatfield tool result payloads.
- Handle timeouts, broker errors, SDK errors, and stale results.
- Ensure existing ToolExecutor/FaultTolerantToolbox error handling remains the outer boundary.

Depends on: MCP-04.

## Acceptance criteria
- A normal tool worker can invoke a discovered MCP tool through the broker and receive a result.
- With multiple tool workers configured, STDIO MCP server/process ownership remains broker-only and is not duplicated per worker.
- MCP text results map to normal tool output; unknown structured content is represented safely.
- MCP errors and timeouts become normal failed tool results without crashing workers.
- Correlation result records are cleaned up or marked stale after timeout/completion.
- Castor tests cover successful call, error call, timeout, and multi-worker STDIO ownership as far as practical.

## Workflow metadata
Status: DONE
Branch: task/mcp-05-tool-invocation-request-reply
Worktree: /home/ineersa/projects/agent-core-worktrees/mcp-05-tool-invocation-request-reply
Fork run: kzyb36znl63x
PR URL: https://github.com/ineersa/agent-core/pull/174
PR Status: merged
Started: 2026-06-19T18:06:05.539Z
Completed: 2026-06-19T20:20:46.061Z

## Work log
- Created: 2026-06-12T18:06:52.879Z

## Task workflow update - 2026-06-19T17:52:23.581Z
- Summary: Design correction from planning discussion: MCP-05 should NOT implement a separate request/reply result store or McpCallToolCommand mailbox. MCP tool invocations are normal ExecuteToolCall messages and should use the existing ToolExecutor/ToolCallResult pipeline. The single-owner broker requirement should be satisfied by routing MCP-backed ExecuteToolCall messages to the existing mcp transport before Messenger transport selection. Normal tools continue to route to tool transport. MCP tools route to mcp transport, where the existing ExecuteToolCallWorker runs in the single mcp consumer; ToolExecutor invokes McpToolHandler; McpToolHandler calls broker-owned McpConnectionManager directly; ExecuteToolCallWorker emits the normal ToolCallResult. Implementation implication: add MCP-aware outbound routing (likely Messenger middleware on agent.execution.bus or equivalent dispatch hook) that checks runId+toolName against the session MCP catalog and adds TransportNamesStamp(['mcp']) for MCP-backed tools, skipping already-received envelopes. Remove DB result-store/request-reply mailbox from MCP-05 plan.

## Task workflow update - 2026-06-19T17:55:59.539Z
- Summary: Additional design decisions from planning discussion: MCP call timeout should come from mcp.json server timeoutMs and be capped by the normal ToolExecutor/ToolContext timeout, i.e. effective timeout = min(mcp.json timeoutMs, ToolContext timeout). If an MCP-backed tool exists in the session catalog but the mcp consumer has no live client for runId+serverName, MCP-05 should attempt reconnect/discovery once before failing cleanly as a normal failed tool result. Remaining open design point: exact routing hook for MCP-backed ExecuteToolCall messages (Messenger middleware vs StepDispatcher/batch dispatch change vs dedicated router service).

## Task workflow update - 2026-06-19T17:58:10.809Z
- Summary: Routing design decision resolved: use Messenger middleware on agent.execution.bus for MCP-aware ExecuteToolCall routing. The middleware should inspect outbound ExecuteToolCall envelopes, skip already received messages and envelopes already explicitly transport-stamped, read the session MCP catalog by runId, and if toolName is MCP-backed add TransportNamesStamp(['mcp']). Otherwise leave the default ExecuteToolCall route to the tool transport unchanged. This keeps AgentCore generic, avoids StepDispatcher/ToolBatchCollector changes, and routes MCP tools to the single mcp consumer where the existing ExecuteToolCallWorker/ToolExecutor/ToolCallResult pipeline runs.

## Task workflow update - 2026-06-19T18:02:24.273Z
- Summary: McpToolHandler construction decision resolved: use hybrid Option 3. McpToolRegistrar should create per-tool McpToolHandler instances through a McpToolHandlerFactory, carrying only static MCP identity (serverName, raw mcpName/hatfield tool mapping as needed) plus a shared McpToolInvoker service. McpToolHandler stays a tiny value-object-like callable and delegates to McpToolInvoker. McpToolInvoker is the autowired service that reads StackToolExecutionContextAccessor for runId/toolCallId/ToolContext timeout, resolves mcp.json server timeoutMs, applies effective timeout min(mcp.json timeoutMs, ToolContext timeout), calls McpConnectionManagerInterface::callTool(), attempts reconnect/discovery once if the live client is missing, maps results through McpResultMapper, and throws structured ToolCallException on MCP errors/timeouts. Keep McpToolHandler itself explicitly not autowireable/autowire:false because it has scalar per-tool identity and must be produced by the factory, not by Symfony DI as a generic service.

## Task workflow update - 2026-06-19T18:06:05.539Z
- Moved TODO → IN-PROGRESS.
- Created branch task/mcp-05-tool-invocation-request-reply.
- Created worktree /home/ineersa/projects/agent-core-worktrees/mcp-05-tool-invocation-request-reply.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/mcp-05-tool-invocation-request-reply.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/mcp-05-tool-invocation-request-reply.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/mcp-05-tool-invocation-request-reply.

## Task workflow update - 2026-06-19T18:06:52.029Z
- Recorded fork run: b4b7uuh47kwm
- Summary: Started implementation fork b4b7uuh47kwm in worktree /home/ineersa/projects/agent-core-worktrees/mcp-05-tool-invocation-request-reply. Fork instructed to implement corrected no-mailbox design: MCP-aware Messenger middleware routing ExecuteToolCall to mcp transport, hybrid McpToolHandlerFactory/McpToolHandler/McpToolInvoker, McpConnectionManager::callTool with reconnect/discovery once, McpResultMapper, service/deptrac wiring, focused tests, Castor-only validation, and commit before handoff.

## Task workflow update - 2026-06-19T18:19:20.062Z
- Summary: Implementation fork b4b7uuh47kwm completed but wrote all changes to the integration checkout (/home/ineersa/projects/agent-core) instead of the task worktree. Parent preserved the misplaced changes in git stash `stash@{0}` with message `mcp-05 misplaced fork changes from b4b7uuh47kwm` and cleaned the integration checkout. Correct worktree remains clean. Relaunching a new implementation fork in the correct worktree to apply/salvage the stash, finish the incomplete resolver test update, run validation, and commit on task branch.

## Task workflow update - 2026-06-19T18:19:48.633Z
- Recorded fork run: 1xkvtxeyzjaq
- Summary: Relaunched implementation fork 1xkvtxeyzjaq in the correct worktree to apply/salvage stash@{0}, finish MCP-05 tests, run focused Castor validation, and commit on branch task/mcp-05-tool-invocation-request-reply. Fork explicitly instructed to confirm cwd/branch, read testing skill and tests/AGENTS.md, avoid committing unrelated .pi/safe-guard.json, and not drop the stash.

## Task workflow update - 2026-06-19T18:26:48.035Z
- Validation: Fork confirmed it read testing skill and tests/AGENTS.md and used Castor-only validation.; castor test --filter=McpCatalogRegisteringToolSetResolverTest — OK (4 tests, 10 assertions); castor test --filter=McpToolRegistrarTest — OK (10 tests, 50 assertions); castor test --filter=McpExecuteToolCallRoutingMiddlewareTest — OK (8 tests, 12 assertions); castor test --filter=McpResultMapperTest — OK (10 tests, 20 assertions); castor test --filter=McpToolHandlerTest — OK (4 tests, 9 assertions); castor test --filter=Mcp — OK (99 tests, 334 assertions); castor deptrac — OK (violations=0, errors=0); castor phpstan — OK (errors=0, file_errors=0); castor cs-check — OK (files_fixed=0, clean)
- Summary: Implementation fork 1xkvtxeyzjaq completed successfully in the correct worktree. Verified branch task/mcp-05-tool-invocation-request-reply has commit a6a27fa2a `Implement MCP tool invocation routing`; worktree status is clean. Diff stat origin/main...HEAD: 16 files changed, 1334 insertions, 32 deletions. Integration checkout is clean; preserved mistaken-change stash remains as stash@{0} (`mcp-05 misplaced fork changes from b4b7uuh47kwm`) and has not been dropped yet.

## Task workflow update - 2026-06-19T18:42:46.474Z
- Summary: Reviewer subagent reviewed HEAD a6a27fa2a and returned REQUEST CHANGES. Main findings: effective timeout min(mcp.json timeoutMs, ToolContext timeout) is computed/threaded but not enforced because McpConnectionManager::callTool drops requestedTimeoutMs and SDK client timeout is fixed at construction; successful empty MCP content currently becomes ToolCallException; LLM-visible ToolCallException messages include raw unsanitized SDK/MCP error text while logs are sanitized; malformed PHPDoc stray backslash in McpToolRegistrarTest; double-prefixing of MCP tool context in handler/connection errors; AppMcpClient throws AgentCore ToolCallException directly as an uncovered boundary edge; docblocks overstate reconnect/discovery and timeout behavior; cancellation omission should be documented; consider retryability for timeout/reconnect and URI redaction in mapper placeholders. Launching fix fork to address all sensible findings before re-review.

## Task workflow update - 2026-06-19T18:43:16.734Z
- Recorded fork run: 2c1f0h6q8sn7
- Summary: Launched fix fork 2c1f0h6q8sn7 in the correct worktree to address reviewer REQUEST CHANGES: timeout non-enforcement/dead parameter, sanitization of LLM-visible errors, empty successful content mapping, PHPDoc typo, double-prefixing, AppMcpClient boundary cleanup, docblock accuracy, cancellation/timeout TODO, retryability consideration, URI redaction, test comment accuracy, and focused Castor validation/commit.

## Task workflow update - 2026-06-19T18:49:38.232Z
- Validation: Fix fork read testing skill and tests/AGENTS.md and used Castor-only validation.; castor test --filter=McpResultMapperTest — OK (15 tests, 31 assertions); castor test --filter=McpToolHandlerTest — OK (4 tests, 9 assertions); castor test --filter=McpToolRegistrarTest — OK (10 tests, 50 assertions); castor test --filter=McpExecuteToolCallRoutingMiddlewareTest — OK (8 tests, 12 assertions); castor test --filter=Mcp — OK (104 tests, 345 assertions); castor deptrac — OK (violations=0, errors=0, uncovered=907); castor phpstan — OK (errors=0, file_errors=0); castor cs-fix && castor cs-check — OK (files_fixed=0)
- Summary: Fix fork 2c1f0h6q8sn7 completed and committed 4b90439c4 `Address MCP invocation review findings` on branch task/mcp-05-tool-invocation-request-reply. Verified worktree clean. Diff origin/main...HEAD now: 17 files changed, 1476 insertions, 33 deletions. Fork reports all 11 reviewer findings addressed: removed dead per-call timeout plumbing and documented SDK limitation; sanitized model-visible errors; empty successful content now maps to empty string; fixed PHPDoc/comment issues; removed double-prefixing; added McpClientInvocationException to keep AppMcpClient decoupled from AgentCore ToolCallException; updated docblocks/TODOs; resource URI credential redaction; retryability for client-layer transient failures.

## Task workflow update - 2026-06-19T19:01:02.621Z
- Summary: Reviewer subagent re-reviewed HEAD 4b90439c4 and returned APPROVED. Reviewer confirmed corrected no-mailbox ExecuteToolCall routing design, all prior REQUEST CHANGES items resolved, no critical/blocking issues, and noted only non-blocking NTH suggestions. Parent dropped the preserved misplaced-change stash after verifying the committed worktree; integration checkout remains clean.

## Task workflow update - 2026-06-19T19:02:04.951Z
- Validation: Reviewer subagent re-review of HEAD 4b90439c4 — APPROVED; no blockers, only non-blocking NTH/security-defense-in-depth suggestions.; castor test — OK (2858 tests, 8591 assertions); castor deptrac — OK (violations=0, errors=0, uncovered=907, allowed=1237); castor phpstan — OK (errors=0, file_errors=0); castor cs-check — OK (files_fixed=0); castor test:llm-real — OK (5 tests, 51 assertions; llama.cpp generation preflight OK)
- Summary: Pre-PR review/validation complete for HEAD 4b90439c4. Reviewer verdict: APPROVED. Worktree status clean on branch task/mcp-05-tool-invocation-request-reply. Proceeding to CODE-REVIEW move, which will run deterministic castor check, push branch, and create PR.

## Task workflow update - 2026-06-19T19:03:15.116Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (54.8s).
- Pushed task/mcp-05-tool-invocation-request-reply to origin.
- branch 'task/mcp-05-tool-invocation-request-reply' set up to track 'origin/task/mcp-05-tool-invocation-request-reply'.
- Created PR: https://github.com/ineersa/agent-core/pull/174
- Validation: Reviewer subagent re-review — APPROVED; castor test — OK (2858 tests, 8591 assertions); castor deptrac — OK (violations=0, errors=0, uncovered=907); castor phpstan — OK (errors=0, file_errors=0); castor cs-check — OK (files_fixed=0); castor test:llm-real — OK (5 tests, 51 assertions)
- Summary: MCP-05 implementation ready for review. Final HEAD 4b90439c4 implements MCP-aware ExecuteToolCall routing to the mcp transport, hybrid McpToolHandlerFactory/McpToolHandler/McpToolInvoker invocation, McpConnectionManager::callTool with reconnect-once behavior, McpResultMapper with sanitized LLM-visible error mapping, and focused tests. Reviewer re-review approved current HEAD. Focused validation passed before transition.

## Task workflow update - 2026-06-19T19:11:53.816Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR review iteration requested by user: add a smoke-testable project MCP config to the existing MCP-05 worktree/branch, converted to Hatfield's documented `.hatfield/mcp.json` schema. User-provided config includes context7 and websearch HTTP MCP servers; unsupported root `settings` fields should be translated/omitted according to docs (e.g. idleTimeout 60s -> server timeoutMs 60000; directTools/toolPrefix are not root schema fields).

## Task workflow update - 2026-06-19T19:21:06.971Z
- Validation: Fork validation: JSON decode smoke OK; castor test --filter=McpConfigLoaderTest OK (21 tests, 57 assertions); castor cs-check OK (files_fixed=0).; Parent read testing skill and tests/AGENTS.md before validation.; Parent noted stale root-owned messenger:consume pid 3361 but could not kill (root-owned); proceeded.; castor check — OK (quality ok, 94.5s): deptrac OK, test OK (2854 tests, 8579 assertions), test:controller-replay OK (3 tests, 41 assertions), test:tui OK (11 tests, 100 assertions), phpstan OK, cs-check OK.; Reviewer subagent on HEAD 45d803a3f — APPROVED with suggestions; no blockers.
- Summary: Config iteration fork 80lcti771hgn completed and committed 45d803a3f `Add project MCP smoke config` on task branch. Verified worktree clean and branch ahead of origin by one commit. New tracked `.hatfield/mcp.json` uses documented Hatfield schema with root `mcpServers` only: context7 HTTP server with `${CONTEXT7_API_KEY}` header placeholder and websearch HTTP server at local IP, both with `timeoutMs: 60000`. Unsupported original fields omitted (`settings.toolPrefix`, `settings.directTools`); original `idleTimeout: 60` translated to per-server `timeoutMs: 60000`. Reviewer subagent approved current HEAD with suggestions only. Deterministic castor check passed after config addition.

## Task workflow update - 2026-06-19T19:22:16.042Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (56.7s).
- Pushed task/mcp-05-tool-invocation-request-reply to origin.
- branch 'task/mcp-05-tool-invocation-request-reply' set up to track 'origin/task/mcp-05-tool-invocation-request-reply'.
- PR already exists: https://github.com/ineersa/agent-core/pull/174
- Validation: JSON decode smoke for .hatfield/mcp.json — OK; castor test --filter=McpConfigLoaderTest — OK (21 tests, 57 assertions); castor cs-check — OK (files_fixed=0); castor check — OK (quality ok, 94.5s): deptrac OK, test OK (2854 tests, 8579 assertions), test:controller-replay OK (3 tests, 41 assertions), test:tui OK (11 tests, 100 assertions), phpstan OK, cs-check OK; Reviewer subagent — APPROVED with suggestions, no blockers
- Summary: PR iteration ready. Added committed project MCP smoke config `.hatfield/mcp.json` in Hatfield schema for context7 and websearch HTTP MCP servers. Existing MCP-05 implementation remains approved; reviewer re-review of config iteration returned APPROVED with suggestions only. Deterministic castor check passed locally before transition.

## Task workflow update - 2026-06-19T19:29:31.476Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR review iteration requested by user: add a STDIO `database-server` MCP entry to the committed project `.hatfield/mcp.json` smoke config so the user can test a Docker Compose database MCP server. Existing HTTP context7/websearch entries should remain unchanged.

## Task workflow update - 2026-06-19T19:30:55.155Z
- Validation: Fork validation: JSON decode smoke OK; castor test --filter=McpConfigLoaderTest OK (21 tests, 57 assertions); castor cs-check OK (files_fixed=0).; Parent verification: git status clean on branch task/mcp-05-tool-invocation-request-reply; HEAD 05c4a77d9; `.hatfield/mcp.json` content inspected and matches requested STDIO entry.
- Summary: Config iteration fork cpkce22h6sjd completed and committed 05c4a77d9 `Add database MCP smoke server` on task branch. Verified worktree clean and branch ahead of origin by one commit. `.hatfield/mcp.json` now has three servers: context7 HTTP, websearch HTTP, and `database-server` STDIO via `/usr/bin/env` + docker compose command. Parent verified the committed file uses the requested `COMPOSE_IGNORE_ORPHANS=true` spelling. Per user request, skipping reviewer for this config-only iteration and proceeding to PR update via task workflow.

## Task workflow update - 2026-06-19T19:31:59.940Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (54.3s).
- Pushed task/mcp-05-tool-invocation-request-reply to origin.
- branch 'task/mcp-05-tool-invocation-request-reply' set up to track 'origin/task/mcp-05-tool-invocation-request-reply'.
- PR already exists: https://github.com/ineersa/agent-core/pull/174
- Validation: JSON decode smoke for .hatfield/mcp.json — OK; castor test --filter=McpConfigLoaderTest — OK (21 tests, 57 assertions); castor cs-check — OK (files_fixed=0); Parent verified committed file content and clean worktree
- Summary: Config-only PR iteration ready. Added committed `database-server` STDIO MCP smoke server to `.hatfield/mcp.json` while preserving existing context7/websearch entries. Reviewer intentionally skipped per user request. Worktree clean; focused validation passed. Moving back to CODE-REVIEW to run deterministic castor check and push PR update.

## Task workflow update - 2026-06-19T19:37:01.528Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR feedback / smoke-test issue: adding a slow/failing STDIO MCP server revealed design flaw. Server failures do not permanently break other servers, but sequential discovery plus single final catalog write means one slow server blocks all discovered tools from becoming visible until its startup timeout expires. Also logs lack a pre-connect/progress event, so discovery appears silent while blocked. Need fix: publish partial catalog as each server completes discovery (so successful servers become visible before slow/failing servers finish) and add structured per-server discovery start/progress logs before blocking connect/listTools calls.

## Task workflow update - 2026-06-19T19:38:07.199Z
- Summary: User confirmed smoke symptom precisely: MCP tools become visible only after the slow/failing `database-server` discovery times out (~1 minute). This validates the design bug: catalog publication is delayed by the slowest server because discovery is sequential and catalog write is final-only. Fix in progress via fork ikfuur5zj08h: incremental catalog publication plus pre-connect/progress logs.

## Task workflow update - 2026-06-19T19:43:06.616Z
- Summary: User identified the manual cause of the `database-server` startup hang from the compiled docker compose command and plans to fix that MCP server outside this code task. App-side fix remains needed: a slow/hung server must not delay publication of tools from already-discovered servers, and discovery must emit pre-connect/progress logs.

## Task workflow update - 2026-06-19T19:43:41.196Z
- Recorded fork run: ikfuur5zj08h
- Validation: Fork reported: castor test --filter=McpInitializeSessionHandlerTest — OK (9 tests, 54 assertions); Fork reported: castor test --filter=McpConnectionManagerTest — OK (6 tests, 56 assertions); Fork reported: castor test --filter="McpInitializeSessionHandlerTest|McpConnectionManagerTest|McpConfigLoaderTest" — OK (36 tests, 167 assertions); Fork reported: castor deptrac — OK; Fork reported: castor phpstan — OK; Fork reported: castor cs-check — OK after cs-fix
- Summary: Fix fork ikfuur5zj08h completed and committed e383a8dc0 (`Publish MCP catalog incrementally during discovery`). Reported changes: optional discovery progress callback, partial catalog writes in initialize/refresh after each server result, pre-connect `discovery.server_starting` logs, focused tests for partial catalog publication and start logs. Parent is inspecting/reviewing before moving back to CODE-REVIEW.

## Task workflow update - 2026-06-19T19:50:18.202Z
- Summary: User smoke-tested the incremental MCP catalog fix and confirmed it works with both broken and working `database-server`: context7/websearch tools become visible despite broken server, and behavior works once DB server is fixed. Reviewer then found a blocker edge case: partial-catalog callback is currently invoked inside McpConnectionManager discovery try/catch, so a catalog write/callback failure can be misclassified as server discovery failure and can wipe partial catalogs via the handler catch. Need one small fix before returning to CODE-REVIEW.

## Task workflow update - 2026-06-19T19:54:29.311Z
- Recorded fork run: xf4m1i0cghz1
- Validation: Fork reported: castor test --filter=McpConnectionManagerTest — OK (9 tests, 77 assertions); Fork reported: castor test --filter=McpInitializeSessionHandlerTest — OK (9 tests, 54 assertions); Fork reported: castor test --filter=McpConfigLoaderTest — OK (21 tests, 57 assertions); Fork reported: castor test --filter=McpResultMapperTest — OK (15 tests, 31 assertions); Fork reported: castor test --filter=McpExecuteToolCallRoutingMiddlewareTest — OK (8 tests, 12 assertions); Fork reported: castor deptrac — OK; Fork reported: castor phpstan — OK; Fork reported: castor cs-check — OK after cs-fix
- Summary: Fix fork xf4m1i0cghz1 completed and committed 51431b2ad (`Isolate MCP discovery callback failures`). It moved MCP discovery progress callback invocation outside the connect/listTools try/catch, wraps callback errors in structured `discovery.callback_failed` warnings, preserves final discovery results on callback failure, and adds event_type/session_id to discovery connected/failed logs. Added 3 McpConnectionManager regression tests covering callback failure after success, callback failure after failed server, and callback invocation count/shape. Parent inspection found one uncommitted user smoke config change remains in `.hatfield/mcp.json`: database-server command changed from `/usr/local/bin/docker` to `docker`. Need decide whether to commit/revert before CODE-REVIEW because move_task requires clean worktree.

## Task workflow update - 2026-06-19T20:04:01.656Z
- Recorded fork run: nik1slnosy8b
- Summary: Final cleanup fork nik1slnosy8b handoff claimed commit c6b1e9ceb, but parent verification found that commit does not exist in either task worktree or integration checkout. Task worktree is still at 51431b2ad with uncommitted `.hatfield/mcp.json` change only; `discovery.config_failed` log still lacks event_type/session_id. Handoff treated as invalid/incomplete; relaunching a tighter fork to apply and commit the intended cleanup in the correct worktree.

## Task workflow update - 2026-06-19T20:06:39.939Z
- Recorded fork run: l1mxtxg03elr
- Validation: Fork reported: castor test --filter=McpConnectionManagerTest — OK (9 tests, 77 assertions); Fork reported: castor test --filter=McpInitializeSessionHandlerTest — OK (9 tests, 54 assertions); Fork reported: castor test --filter=McpConfigLoaderTest — OK (21 tests, 57 assertions); Fork reported: castor phpstan — OK (0 errors); Fork reported: castor cs-check — OK (files_fixed=0); Fork reported: castor deptrac — OK (0 violations, 0 errors); Parent verified worktree status clean, .hatfield/mcp.json contains `docker`, and config_failed log includes event_type/session_id
- Summary: Final cleanup fork l1mxtxg03elr completed correctly. Parent verified actual commits exist and worktree is clean: 90dea95d9 (`Fix database-server Docker path in project MCP smoke config`) committed the user-smoke-tested `docker` command, and 526723eb5 (`Finalize MCP smoke config and discovery logs`) added event_type/session_id to `discovery.config_failed`. Reviewer re-review was APPROVED WITH SUGGESTIONS before this cleanup; suggestions addressed. Ready to move back to CODE-REVIEW.

## Task workflow update - 2026-06-19T20:08:16.571Z
- Summary: Move to CODE-REVIEW failed at deterministic castor check: `castor test` crashed on `McpToolHandlerTest::testHandlerDelegatesToInvokerAndReturnsResult`. Parent reproduced with `castor test --filter=McpToolHandlerTest`. Root cause is not stale worker: anonymous test doubles in McpToolHandlerTest implement McpConnectionManagerInterface with stale `discover(string $runId): array` signature after interface changed to `discover(string $runId, ?callable $onServerDiscovered = null): array`, causing PHP fatal/premature process end. Need tiny test-fix commit, then retry CODE-REVIEW.

## Task workflow update - 2026-06-19T20:10:27.339Z
- Recorded fork run: kzyb36znl63x
- Validation: Fork reported: castor test --filter=McpToolHandlerTest — OK (4 tests, 9 assertions); Fork reported: castor phpstan — OK; Fork reported: castor cs-check — OK; Fork reported: castor deptrac — OK; Parent ran: castor test --filter=McpToolRegistrarTest — OK (10 tests, 50 assertions); Parent ran: castor test --filter=McpToolHandlerTest — OK (4 tests, 9 assertions); Parent verified rg found no remaining stale `function discover(string $runId):` implementations in src/tests
- Summary: Fix fork kzyb36znl63x completed and parent verified commits 1a40f168a + 14710ef7e. All stale anonymous test-double implementations of McpConnectionManagerInterface::discover were updated to include the new optional progress callback parameter. Parent confirmed no remaining stale `discover(string $runId): array` signatures in src/tests and worktree clean. Focused tests now pass, including the previously crashing McpToolHandlerTest and McpToolRegistrarTest.

## Task workflow update - 2026-06-19T20:11:35.148Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (50.7s).
- Pushed task/mcp-05-tool-invocation-request-reply to origin.
- branch 'task/mcp-05-tool-invocation-request-reply' set up to track 'origin/task/mcp-05-tool-invocation-request-reply'.
- PR already exists: https://github.com/ineersa/agent-core/pull/174
- Validation: User manual smoke test: passed with broken server and with working database-server; Reviewer re-review of callback-failure fix: APPROVE WITH SUGGESTIONS, no blockers; castor test --filter=McpConnectionManagerTest — OK (9 tests, 77 assertions); castor test --filter=McpInitializeSessionHandlerTest — OK (9 tests, 54 assertions); castor test --filter=McpConfigLoaderTest — OK (21 tests, 57 assertions); castor test --filter=McpResultMapperTest — OK (15 tests, 31 assertions); castor test --filter=McpExecuteToolCallRoutingMiddlewareTest — OK (8 tests, 12 assertions); castor test --filter=McpToolRegistrarTest — OK (10 tests, 50 assertions); castor test --filter=McpToolHandlerTest — OK (4 tests, 9 assertions); castor deptrac — OK (0 violations, 0 errors); castor phpstan — OK (0 errors); castor cs-check — OK (files_fixed=0); move_task will run deterministic castor check before push/PR update
- Summary: MCP-05 PR iteration complete after smoke-discovered issue and test-signature fix. User manually smoke-tested incremental catalog publication successfully with broken and working `database-server`. Implemented incremental partial catalog writes during discovery so fast/healthy servers become visible before slow/failing servers finish; added pre-connect discovery logs; isolated partial-catalog callback failures so they cannot corrupt discovery results or wipe catalogs; committed working database-server smoke config (`docker` path), log-field consistency cleanup, and fixed stale McpConnectionManagerInterface test-double signatures that caused deterministic castor check crashes. Reviewer re-review returned APPROVE WITH SUGGESTIONS; suggestions addressed.

## Task workflow update - 2026-06-19T20:20:46.061Z
- Moved CODE-REVIEW → DONE.
- Merged task/mcp-05-tool-invocation-request-reply into integration checkout.
- Merge made by the 'ort' strategy.
 config/packages/messenger.yaml                     |   6 +
 config/services.yaml                               |  24 +-
 depfile.yaml                                       |  10 +
 .../Mcp/Client/McpClientInvocationException.php    |  22 ++
 .../Mcp/Client/McpConnectionManager.php            | 181 +++++++++++-
 .../Mcp/Client/McpConnectionManagerInterface.php   |  29 +-
 .../Mcp/Handler/McpInitializeSessionHandler.php    |  32 +-
 .../McpExecuteToolCallRoutingMiddleware.php        | 130 ++++++++
 src/CodingAgent/Mcp/Tool/McpResultMapper.php       | 160 ++++++++++
 src/CodingAgent/Mcp/Tool/McpToolHandler.php        |  32 +-
 src/CodingAgent/Mcp/Tool/McpToolHandlerFactory.php |  35 +++
 src/CodingAgent/Mcp/Tool/McpToolInvoker.php        | 103 +++++++
 src/CodingAgent/Mcp/Tool/McpToolRegistrar.php      |   3 +-
 .../Mcp/Client/McpConnectionManagerTest.php        | 196 +++++++++++-
 .../Handler/McpInitializeSessionHandlerTest.php    | 200 ++++++++++++-
 .../McpExecuteToolCallRoutingMiddlewareTest.php    | 327 +++++++++++++++++++++
 .../McpCatalogRegisteringToolSetResolverTest.php   |  18 +-
 tests/CodingAgent/Mcp/Tool/McpResultMapperTest.php | 252 ++++++++++++++++
 tests/CodingAgent/Mcp/Tool/McpToolHandlerTest.php  | 162 ++++++++++
 .../CodingAgent/Mcp/Tool/McpToolRegistrarTest.php  |  83 +++++-
 20 files changed, 1949 insertions(+), 56 deletions(-)
 create mode 100644 src/CodingAgent/Mcp/Client/McpClientInvocationException.php
 create mode 100644 src/CodingAgent/Mcp/Messenger/McpExecuteToolCallRoutingMiddleware.php
 create mode 100644 src/CodingAgent/Mcp/Tool/McpResultMapper.php
 create mode 100644 src/CodingAgent/Mcp/Tool/McpToolHandlerFactory.php
 create mode 100644 src/CodingAgent/Mcp/Tool/McpToolInvoker.php
 create mode 100644 tests/CodingAgent/Mcp/Messenger/McpExecuteToolCallRoutingMiddlewareTest.php
 create mode 100644 tests/CodingAgent/Mcp/Tool/McpResultMapperTest.php
 create mode 100644 tests/CodingAgent/Mcp/Tool/McpToolHandlerTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/mcp-05-tool-invocation-request-reply.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/mcp-05-tool-invocation-request-reply.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #174 merged per user confirmation; CODE-REVIEW move gate previously ran deterministic castor check — passed (50.7s)
- Summary: User confirmed PR #174 was merged. Moving MCP-05 to DONE: task implemented MCP tool invocation through broker transport, project MCP smoke config, incremental catalog publication/logging fixes, callback-failure isolation, and test-double signature cleanup. Deterministic CODE-REVIEW gate previously passed and branch was pushed.
