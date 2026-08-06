# MCP-02 Broker transport and single MCP consumer

## Goal
Implement phase 1 broker transport foundation from `.pi/plans/mcp-client-implementation-plan.md`.

Goal: add a dedicated `mcp` Messenger transport/consumer supervised by the controller, but without full tool invocation yet.

Scope:
- Add a Messenger transport/queue for MCP broker messages.
- Update controller/consumer supervision so exactly one `mcp` consumer is launched per controller/session in v1.
- Add broker lifecycle/message skeletons such as initialize, refresh catalog, call tool, and disconnect commands as needed.
- Add structured logging fields for MCP broker lifecycle (`component=mcp`, `run_id`, `session_id`, `server_name`, `transport`, `correlation_id` where applicable).
- Ensure this remains CodingAgent app-layer infrastructure, not AgentCore/TUI/ExtensionApi.

Depends on: MCP-01.

## Acceptance criteria
- Controller/supervisor starts one MCP consumer alongside existing run_control/llm/tool/scheduler consumers.
- The MCP consumer can receive and handle a no-op or initialization message without affecting normal sessions.
- V1 configuration prevents accidental multiple generic MCP consumers.
- Structured MCP broker lifecycle logs are present and redact sensitive fields.
- Castor validation for touched runtime/Messenger code is run as required by project instructions.

## Workflow metadata
Status: DONE
Branch: task/mcp-02-broker-transport-consumer
Worktree: /home/ineersa/projects/agent-core-worktrees/mcp-02-broker-transport-consumer
Fork run: lmkyfpzn3ugl
PR URL: https://github.com/ineersa/agent-core/pull/170
PR Status: merged
Started: 2026-06-18T20:50:25.449Z
Completed: 2026-06-18T21:26:04.174Z

## Work log
- Created: 2026-06-12T18:06:24.722Z

## Task workflow update - 2026-06-18T20:50:20.249Z
- Summary: Task-start decisions resolved by user: (1) dispatch MCP initialize during session start/resume now, with no-op/non-fatal behavior for this phase; (2) defer McpCallToolCommand/tool invocation request-reply to MCP-05 instead of adding a placeholder in MCP-02; (3) invalid MCP config during initialize should be warning-only and must not break normal sessions.

## Task workflow update - 2026-06-18T20:50:25.449Z
- Moved TODO → IN-PROGRESS.
- Created branch task/mcp-02-broker-transport-consumer.
- Created worktree /home/ineersa/projects/agent-core-worktrees/mcp-02-broker-transport-consumer.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/mcp-02-broker-transport-consumer.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/mcp-02-broker-transport-consumer.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/mcp-02-broker-transport-consumer.
- Summary: Starting implementation for MCP-02 after user confirmed scope decisions. The implementation fork should add the dedicated mcp Messenger transport/consumer, dispatch initialize on start/resume, defer tool call command/request-reply to MCP-05, and make initialize config failures warning-only/non-fatal.

## Task workflow update - 2026-06-18T20:51:46.351Z
- Recorded fork run: 9wvs6pm082xd
- Summary: Launched implementation fork 9wvs6pm082xd in worktree /home/ineersa/projects/agent-core-worktrees/mcp-02-broker-transport-consumer. Fork instructions require reading testing skill and tests/AGENTS.md, implementing the mcp Messenger transport/consumer, dispatching initialize on start/resume, deferring call-tool request/reply to MCP-05, warning-only initialize config failures, focused tests, Castor validation, and a clean committed handoff.

## Task workflow update - 2026-06-18T21:02:31.002Z
- Recorded fork run: 9wvs6pm082xd
- Validation: Fork reported: castor deptrac OK (0 violations); Fork reported: castor test --filter=Mcp OK (34 tests, 96 assertions); Fork reported: castor test:controller-replay OK (3 tests, 41 assertions); Fork reported: castor test:tui OK (9 tests, 91 assertions); Fork reported: castor phpstan OK (0 errors); Fork reported: castor cs-check clean; Fork reported: LLM_MODE=true castor check OK (all 6 steps, 87.4s); Fork reported: full test suite OK (2744 tests, 8177 assertions)
- Summary: Implementation fork 9wvs6pm082xd completed successfully at commit 823227cc0 on branch task/mcp-02-broker-transport-consumer. Parent verified worktree is clean, branch is correct, and diff stat vs origin/main is 16 files changed with 438 insertions and 4 deletions. Implementation adds mcp Messenger transport/routing, session-scoped MCP DSNs, one supervised mcp consumer, app-layer MCP lifecycle commands/handler/dispatcher, initialize dispatch on start/resume, warning-only config failures, Deptrac MCP broker/message layers, docs update, and focused MCP handler tests. Scope deferrals preserved: no McpCallToolCommand/request-reply (MCP-05), no SDK connect/listTools/catalog persistence (MCP-03/04), no dynamic tool registration (MCP-04).

## Task workflow update - 2026-06-18T21:08:27.779Z
- Summary: Reviewer subagent completed task-to-pr review of commit 823227cc0 with verdict APPROVE WITH SUGGESTIONS. No critical/correctness/security blockers found. Suggestions: fix inaccurate HeadlessController comment that says McpCallToolCommand is Phase 3+ (should be MCP-05/deferred), improve MCP handler tests that currently assert only no-throw/callability by asserting structured logs via TestLogger or dispatching real messages, consider clearer handler naming and catch/log simplification as non-blocking quality items.

## Task workflow update - 2026-06-18T21:08:45.983Z
- Recorded fork run: lmkyfpzn3ugl
- Summary: Launched review-polish fork lmkyfpzn3ugl to address non-blocking reviewer suggestions before PR: correct inaccurate McpCallToolCommand phase comment, replace weak no-throw/callability MCP handler tests with behavior/log assertions using shared TestLogger where practical, and run focused Castor validation (McpInitializeSessionHandlerTest, Mcp filter, deptrac, phpstan, cs-check).

## Task workflow update - 2026-06-18T21:11:48.470Z
- Recorded fork run: lmkyfpzn3ugl
- Validation: Fork reported: castor test --filter=McpInitializeSessionHandlerTest OK (5 tests, 30 assertions); Fork reported: castor test --filter=Mcp OK (35 tests, 122 assertions); Fork reported: castor deptrac OK (0 violations); Fork reported: castor phpstan OK (0 errors); Fork reported: castor cs-check clean (files_fixed=0)
- Summary: Review-polish fork lmkyfpzn3ugl completed successfully at commit f41398f57. Changes: corrected HeadlessController comment to place McpCallToolCommand in MCP-05, replaced weak MCP handler no-throw/callability tests with 5 structured-log behavior tests using shared TestLogger and TestDirectoryIsolation, and merged duplicate dispatcher catch blocks into one non-fatal structured warning path. Worktree reported clean.

## Task workflow update - 2026-06-18T21:16:04.215Z
- Validation: Parent ran focused validation on worktree HEAD f41398f57: castor test --filter=Mcp OK (35 tests, 122 assertions); Parent ran focused validation: castor deptrac OK (0 violations); Parent ran focused validation: castor phpstan OK (0 errors); Parent ran focused validation: castor cs-check clean (files_fixed=0)
- Summary: Final re-review on HEAD f41398f57 returned APPROVE WITH SUGGESTIONS with no issues found in polish changes and no blockers. Reviewer confirmed core MCP-02 acceptance still holds: one mcp consumer, initialize dispatch on start/resume, mcp routing, warning-only config failures without secret leakage, SDK types isolated under Mcp/Client, and no McpCallToolCommand production implementation. Remaining notes are non-blocking naming/log-filtering nice-to-haves.

## Task workflow update - 2026-06-18T21:19:03.939Z
- Validation: Parent reran castor test:controller-replay after failed transition: OK (3 tests, 41 assertions, 24.0s); Parent reran LLM_MODE=true castor check on final HEAD f41398f57: deptrac OK (1.9s), test OK (2745 tests, 8203 assertions), test:controller-replay OK (3 tests, 41 assertions), test:tui OK (9 tests, 91 assertions), phpstan OK, cs-check OK; quality ok (83.6s)
- Summary: First CODE-REVIEW transition attempt hit a transient deterministic gate failure: castor check failed only in test:controller-replay because ControllerReplaySmokeTest::testControllerReplayToolExecution timed out waiting for runtime.ready after 5s with no stderr/events. Parent immediately re-ran focused controller replay in the worktree and it passed (3 tests, 41 assertions), then re-ran full LLM_MODE=true castor check and it passed all lanes in 83.6s. Worktree remained clean.

## Task workflow update - 2026-06-18T21:21:04.936Z
- Validation: Parent reran castor test:tui after failed transition: OK (9 tests, 91 assertions, 38.4s)
- Summary: Second CODE-REVIEW transition attempt hit a different transient deterministic gate failure: test:tui failed only in TuiStartupSnapshotTest because the footer elapsed timer rendered ⏱ 1s instead of the golden ⏱ 0s. Parent immediately re-ran castor test:tui standalone in the worktree and it passed (9 tests, 91 assertions, 38.4s). This appears unrelated to MCP-02 behavior and consistent with timing-sensitive snapshot drift under gate load; worktree remains clean.

## Task workflow update - 2026-06-18T21:22:16.365Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (42.5s).
- Pushed task/mcp-02-broker-transport-consumer to origin.
- branch 'task/mcp-02-broker-transport-consumer' set up to track 'origin/task/mcp-02-broker-transport-consumer'.
- Created PR: https://github.com/ineersa/agent-core/pull/170
- Validation: Fork validation: LLM_MODE=true castor check OK on base implementation (all 6 steps, 87.4s); Fork polish validation: castor test --filter=McpInitializeSessionHandlerTest OK (5 tests, 30 assertions); Fork polish validation: castor test --filter=Mcp OK (35 tests, 122 assertions); Fork polish validation: castor deptrac OK (0 violations), phpstan OK (0 errors), cs-check clean; Parent focused validation on final HEAD f41398f57: castor test --filter=Mcp OK (35 tests, 122 assertions), deptrac OK, phpstan OK, cs-check clean; Parent rerun after first transient transition failure: castor test:controller-replay OK (3 tests, 41 assertions); LLM_MODE=true castor check OK (quality ok, 83.6s); Parent rerun after second transient transition failure: castor test:tui OK (9 tests, 91 assertions); move_task CODE-REVIEW deterministic castor check will run before push/PR creation
- Summary: MCP-02 is ready for PR at commit f41398f57. Implementation adds dedicated mcp Messenger transport/routing, session-scoped MCP DSNs for controller/replay, exactly one supervised mcp consumer, app-layer lifecycle commands/handler/dispatcher, initialize dispatch on start/resume, warning-only non-fatal config failures, structured redacted MCP lifecycle logs, Deptrac AppMcpMessage/AppMcpBroker layers, docs update, and focused structured-log tests. User-confirmed deferrals preserved: no McpCallToolCommand/request-reply until MCP-05, no real SDK connect/listTools/catalog until MCP-03/04, no dynamic tool registration until MCP-04. Reviewer re-review approved with only non-blocking nice-to-haves. Previous transition attempts hit unrelated timing flakes (controller runtime.ready timeout, then TUI elapsed timer snapshot 1s vs 0s); immediate standalone reruns and a full manual LLM_MODE=true castor check passed on the final HEAD.

## Task workflow update - 2026-06-18T21:26:04.174Z
- Moved CODE-REVIEW → DONE.
- Merged task/mcp-02-broker-transport-consumer into integration checkout.
- Merge made by the 'ort' strategy.
 .env                                               |   1 +
 config/packages/messenger.yaml                     |   6 +
 config/packages/test/messenger.yaml                |   3 +
 config/services.yaml                               |  24 +++
 depfile.yaml                                       |  18 ++
 docs/mcp.md                                        |  25 ++-
 .../Mcp/Handler/McpInitializeSessionHandler.php    | 119 +++++++++++++
 .../Mcp/McpSessionLifecycleDispatcher.php          |  62 +++++++
 .../Mcp/Message/McpDisconnectSessionCommand.php    |  24 +++
 .../Mcp/Message/McpInitializeSessionCommand.php    |  34 ++++
 .../Mcp/Message/McpRefreshCatalogCommand.php       |  23 +++
 .../Runtime/Controller/HeadlessController.php      |   5 +
 .../InProcess/InProcessAgentSessionClient.php      |  10 ++
 .../Process/JsonlProcessAgentSessionClient.php     |   1 +
 .../Handler/McpInitializeSessionHandlerTest.php    | 198 +++++++++++++++++++++
 .../Controller/E2E/ControllerReplayE2eTestCase.php |   1 +
 16 files changed, 550 insertions(+), 4 deletions(-)
 create mode 100644 src/CodingAgent/Mcp/Handler/McpInitializeSessionHandler.php
 create mode 100644 src/CodingAgent/Mcp/McpSessionLifecycleDispatcher.php
 create mode 100644 src/CodingAgent/Mcp/Message/McpDisconnectSessionCommand.php
 create mode 100644 src/CodingAgent/Mcp/Message/McpInitializeSessionCommand.php
 create mode 100644 src/CodingAgent/Mcp/Message/McpRefreshCatalogCommand.php
 create mode 100644 tests/CodingAgent/Mcp/Handler/McpInitializeSessionHandlerTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/mcp-02-broker-transport-consumer.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/mcp-02-broker-transport-consumer.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #170 was reported merged by user before DONE transition.
- Summary: User confirmed PR #170 was merged; moving MCP-02 task to DONE and merging/syncing integration checkout.

## Task workflow update - 2026-06-18T21:27:24.398Z
- Validation: LLM_MODE=true castor check on /home/ineersa/projects/agent-core after merge: quality ok (109.8s); deptrac OK (0.9s); test OK (2745 tests, 8203 assertions, 23.4s); test:controller-replay OK (3 tests, 41 assertions, 27.7s); test:tui OK (9 tests, 91 assertions, 46.6s); phpstan OK (9.1s, 0 errors); cs-check OK (2.0s)
- Summary: Post-merge validation completed on the integration checkout after moving MCP-02 to DONE. The integration checkout merged PR #170/task branch successfully, pulled remote changes, removed the task worktree, and cleaned IDEA exclusions. Before validation, testing skill and tests/AGENTS.md were read per project instructions; no identifiable stale controller/consumer processes from this checkout were present. Final integration checkout validation passed.
