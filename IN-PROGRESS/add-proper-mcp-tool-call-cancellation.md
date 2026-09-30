# Add proper in-flight MCP tool-call cancellation and deadlines

## Goal
Follow-up to `fix-generic-tool-timeout-preemption-semantics`.

MCP tool calls currently block synchronously through installed `mcp/sdk v0.6.0`. The MCP protocol supports `notifications/cancelled`, and the SDK contains `CancelledNotification`, but the public `Client::callTool()` API does not expose the request ID or accept a cancellation token/per-call deadline. Hatfield Escape cancellation therefore cannot interrupt an in-flight MCP request; the call continues until response or the fixed SDK timeout, which currently tears down the client connection.

Implement proper request-scoped cancellation in the MCP SDK via an upstream contribution or maintained fork, then propagate Hatfield `ToolContext` cancellation/deadline through `McpToolInvoker`, connection manager, and client adapter. Do not duplicate the MCP protocol stack inside agent-core unless the dependency route is proven impossible.

## Acceptance criteria
- Decide and document the MCP SDK integration strategy (upstream release or maintained fork) without app-local protocol duplication by default.
- The MCP client call API accepts request-scoped cancellation and a per-call deadline/remaining timeout.
- Cancelling an in-flight request sends protocol-correct `notifications/cancelled` with the active request ID where required by the negotiated protocol/transport.
- Cancellation removes pending request state, stops waiting before normal tool completion, and ignores any late response.
- Successful cooperative cancellation preserves the shared MCP connection; transport failure still disconnects and reconnects safely.
- STDIO and Streamable HTTP semantics are handled according to the negotiated MCP protocol version; HTTP cancellation uses an actually abortable/asynchronous path rather than a post-hoc check.
- Hatfield propagates `ToolContext` cancellation and deadline through `McpToolInvoker`, connection manager, and SDK adapter, returning a structured cancelled/timed-out tool result.
- Regression tests prove Escape cancellation before a slow MCP tool completes, deadline cancellation, no pending-request/process leak, and a successful subsequent MCP call/reconnect.
- Document MCP server/extension expectations and known advisory-cancellation limits thoroughly.
- Testing skill and tests/AGENTS.md conventions are followed; all QA runs through Castor, including controller replay/runtime proof and the required full `castor check`.

## Workflow metadata
Status: IN-PROGRESS
Branch: task/add-proper-mcp-tool-call-cancellation
Worktree: /home/ineersa/projects/agent-core-worktrees/add-proper-mcp-tool-call-cancellation
Fork run: 88e66c3f-3403-5ec3-9d48-a21dee563a49
PR URL:
PR Status:
Started: 2026-09-27T22:41:19+00:00
Completed:

## Work log
- Created: 2026-07-31T21:08:48.115Z

## Task workflow update - 2026-09-27T22:41:19+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/add-proper-mcp-tool-call-cancellation.
- Created worktree /home/ineersa/projects/agent-core-worktrees/add-proper-mcp-tool-call-cancellation.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/add-proper-mcp-tool-call-cancellation.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/add-proper-mcp-tool-call-cancellation.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/add-proper-mcp-tool-call-cancellation.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/add-proper-mcp-tool-call-cancellation/.idea.

## Task workflow update - 2026-09-27T22:42:51+00:00
- Summary: SDK source checkout /home/ineersa/projects/php-sdk is on new branch task/add-proper-mcp-tool-call-cancellation from upstream/main 16836d4. Hatfield worktree baseline 811587184. User wants local Hatfield proof before inspecting SDK code; no PR.
- Ownership: owner=fork; fork_run=pending; revision=php-sdk:16836d4; scope=request-scoped SDK callTool cancellation and deadlines in /home/ineersa/projects/php-sdk, including STDIO and HTTP tests; outcome=assigned; commit=none
- Ownership: owner=main; fork_run=none; revision=agent-core:811587184; scope=Hatfield MCP cancellation wiring, structured results, docs, and runtime proof after SDK handoff; outcome=assigned; commit=none

## Task workflow update - 2026-09-27T22:51:55+00:00
- Recorded fork run: 18eca48d-e13b-5bd1-abd6-2bb9b1355ecd
- Summary: SDK fork investigation found PSR-18 HTTP sendRequest blocks before cancellation can be observed on no-response servers. Resuming the same owner for SDK implementation; HTTP must be genuinely abortable before acceptance.
- Ownership: owner=fork; fork_run=18eca48d-e13b-5bd1-abd6-2bb9b1355ecd; revision=php-sdk:16836d4; scope=request-scoped SDK callTool cancellation and deadlines in /home/ineersa/projects/php-sdk; outcome=assigned; commit=none

## Task workflow update - 2026-09-27T22:52:28+00:00
- Summary: Prior SDK fork cannot resume because its context reached the child limit; handing SDK ownership to a fresh fork with the completed investigation. No SDK tracked code changes were made by prior fork.
- Ownership: owner=fork; fork_run=18eca48d-e13b-5bd1-abd6-2bb9b1355ecd; revision=php-sdk:16836d4; scope=SDK discovery on cancellation and transport constraints; outcome=blocked; commit=none
- Ownership: owner=fork; fork_run=pending; revision=php-sdk:16836d4; scope=SDK cancellation and deadline implementation after previous fork context limit; outcome=assigned; commit=none

## Task workflow update - 2026-09-27T22:57:17+00:00
- Recorded fork run: 88e66c3f-3403-5ec3-9d48-a21dee563a49
- Validation: Throwaway probe in php-sdk checkout: CurlHttpClient::request() returned before headers; stream($response, 0.1) yielded a timeout chunk in 0.101s; cancel() caused server-side TCP EOF. Initial probe fixture closed too soon and was corrected; no repo files changed.
- Summary: Selected optional Symfony HttpClient async transport for cancellable HTTP calls after a local no-header probe: 0.1s stream timeout yielded, response cancel closed the TCP connection. SDK fork may now implement that path while retaining PSR-18 for existing callers.
- Ownership: owner=fork; fork_run=88e66c3f-3403-5ec3-9d48-a21dee563a49; revision=php-sdk:16836d4; scope=SDK cancellation implementation with optional Symfony async HTTP path, STDIO and HTTP tests; outcome=assigned; commit=none

## Task workflow update - 2026-09-27T23:50:21+00:00
- Summary: Previous SDK implementation fork could not resume because its context exceeded the child limit. Handing off the bounded STDIO/core implementation to a fresh fork; parent will integrate and validate in Hatfield.
- Ownership: owner=fork; fork_run=88e66c3f-3403-5ec3-9d48-a21dee563a49; revision=php-sdk:16836d4; scope=SDK investigation and HTTP abort approach; outcome=blocked; commit=none
- Ownership: owner=fork; fork_run=pending; revision=php-sdk:16836d4; scope=SDK Client::callTool cancellation/deadline plumbing, STDIO notification and late-response cleanup with deterministic tests; outcome=assigned; commit=none

## Task workflow update - 2026-09-27T23:56:29+00:00
- Summary: SDK fork ran out of runtime without editing; main takes SDK implementation ownership to avoid further stalled delegation.
- Ownership: owner=fork; fork_run=none; revision=php-sdk:16836d4; scope=SDK STDIO cancellation investigation; outcome=blocked; commit=none
- Ownership: owner=main; fork_run=none; revision=php-sdk:16836d4; scope=SDK STDIO cancellation and deadline implementation, then HTTP and Hatfield integration; outcome=assigned; commit=none

## Task workflow update - 2026-09-28T00:28:40+00:00
- Validation: SDK: make unit-tests passed (1583 tests, 4108 assertions); integration PHPUnit suite passed (72 tests, 169 assertions); make phpstan passed; make cs passed. make tests inspector lane blocked by read-only ~/.npm cache (EROFS fetching inspector@2.2.0).; Hatfield task worktree: castor test --filter=Mcp passed (189 tests, 727 assertions); castor phpstan passed; castor cs-check passed; castor deptrac passed; castor docs:validate passed; PHAR built and smoke passed with mirrored SDK path; real STDIO cancellation/reuse and HTTP async invocation integration cases passed.
- Summary: SDK branch task/add-proper-mcp-tool-call-cancellation now has commit 281d371 implementing request-scoped cancellation and deadlines, STDIO notifications, HTTP socket abort, late-response cleanup, and tests. Hatfield task worktree uses local Composer path mirror pinned to SDK commit for trial; no PR or push. Integration wiring and tests are uncommitted pending SDK review and portable dependency selection.
- Ownership: owner=main; fork_run=none; revision=php-sdk:281d371; scope=SDK request cancellation and deadlines with STDIO/HTTP tests; outcome=completed; commit=281d371
- Ownership: owner=main; fork_run=none; revision=agent-core:811587184+uncommitted; scope=Hatfield MCP token/deadline wiring and real STDIO/HTTP focused tests; outcome=blocked; commit=none

## Task workflow update - 2026-09-28T13:04:23+00:00
- Summary: User approved narrower framework-agnostic revision. No-header HTTP preemption is out of scope; remove its async-specific test and fixture. Keep existing STDIO cancellation proof and add deterministic cooperative HTTP coverage. No push or PR.
- Ownership: owner=main; fork_run=none; revision=php-sdk:281d371; scope=Remove Symfony HTTP async path; implement cooperative HTTP cancellation through existing SSE loop; document blocking I/O limits and align Hatfield wiring; outcome=assigned; commit=none

## Task workflow update - 2026-09-28T13:23:24+00:00
- Validation: SDK make unit-tests: 1586 tests, 4116 assertions; make integration-tests: 71 tests, 163 assertions; make phpstan clean; make cs applied formatting; git diff --check clean.; Hatfield refreshed local SDK mirror: castor test --filter=Mcp passed 189 tests, 727 assertions; castor phpstan, cs-check, docs:validate passed; PHAR build/smoke passed during first refresh.; Independent reviewer agent_d4e5347c662cdd61: APPROVE WITH SUGGESTIONS for local SDK redo; no implementation blockers. Machine-local Composer dependency remains a submission blocker, not committed.
- Summary: Reworked SDK HTTP cancellation to stay PSR-18/framework-agnostic. Removed asyncHttpClient API, Symfony-specific response handling, duplicate SSE parser path, and no-header-abort fixture/test. Existing tick loop checks interruption before/after SSE processing and closes active response body. Added deterministic pre-read/between-read cancellation and expired-deadline tests. docs/client/transports.md documents cooperative limits, timeout override, and server-side tradeoffs. Hatfield factory and docs aligned. Changes remain local/uncommitted; no push or PR.
- Ownership: owner=main; fork_run=none; revision=php-sdk:281d371+working-tree; scope=framework-agnostic cooperative HTTP cancellation redo and Hatfield alignment; outcome=completed; commit=none
- Review: role=reviewer; artifact=agent_d4e5347c662cdd61; revision=php-sdk:281d371+working-tree; scope=SDK cooperative HTTP simplification and Hatfield wiring/docs; decision=APPROVE WITH SUGGESTIONS; submission_blocker=portable dependency reference required before PR.

## Task workflow update - 2026-09-28T13:47:35+00:00
- Summary: User requested squash. Amended local SDK commit 281d371 with reviewed cooperative HTTP redo into fa67a3f (Support in-flight MCP tool call cancellation and deadlines). SDK branch is exactly one commit ahead of upstream/main 16836d4; tracked working tree clean. Untracked .idea/ and AGENTS.md preserved. No push. Hatfield worktree and its local Composer reference left unchanged.

## Task workflow update - 2026-09-28T14:41:13+00:00
- Validation: SDK unit suite passed 1594 tests / 4153 assertions; phpstan and formatting passed. Fork reports integration suite passed 71 tests / 163 assertions, with a separate concurrent port-contention failure recorded in its handoff.; Hatfield castor test --filter=Mcp passed 189 tests / 727 assertions; PHAR smoke and docs:validate passed.; Independent reviewer agent_d4e5347c662cdd61 APPROVE for working diff against fa67a3f; focused review tests 35 / 100 assertions passed.
- Summary: Addressed user's P1 synchronous JSON interruption bypass and P2 protocol-version signaling review. User selected best-effort cancellation POST for legacy HTTP. SDK rechecks interruption after send and discards buffered result; legacy HTTP notifies, modern HTTP closes stream; notification responses cannot overwrite active SSE request. Hatfield docs and local SDK mirror updated. Changes remain uncommitted, no push.
- Ownership: owner=fork; fork_run=agent_c07604ae354e64b2; revision=php-sdk:fa67a3f+working-tree; scope=post-send interruption cleanup and version-aware cancellation signaling/tests; outcome=completed; commit=none
- Ownership: owner=main; fork_run=none; revision=php-sdk:fa67a3f+working-tree; scope=review fixes, nonempty exception assertions, Hatfield docs and mirror validation; outcome=completed; commit=none
- Review: role=reviewer; artifact=agent_d4e5347c662cdd61; revision=php-sdk:fa67a3f+working-tree; scope=P1/P2 fixes; decision=APPROVE; submission_blocker=Hatfield portable dependency reference still required.

## Task workflow update - 2026-09-28T14:50:01+00:00
- Summary: User authorized squash and push. Squashed reviewed SDK P1/P2 fixes into single commit f0835c4 and pushed task/add-proper-mcp-tool-call-cancellation to origin git@github.com:ineersa/php-sdk.git using explicit force-with-lease against fa67a3f84f3893fc9c915091c9f877f7aea6cf4c. Branch remains one commit above 16836d4. Tracked SDK working tree clean; unrelated untracked .idea/ and AGENTS.md preserved. No PR opened. Hatfield worktree unchanged.

## Task workflow update - 2026-09-28T15:22:48+00:00
- Ownership: owner=main; fork_run=none; revision=php-sdk:f0835c4; scope=central post-suspension interruption check and buffered-response ordering regression; outcome=assigned; commit=none

## Task workflow update - 2026-09-28T15:26:24+00:00
- Validation: SDK make unit-tests passed 1596 tests / 4171 assertions; make integration-tests passed 71 / 163; make phpstan and make cs passed; diff check clean.; Refreshed Hatfield local SDK mirror; castor test --filter=Mcp passed 189 / 727.; Reviewer agent_d4e5347c662cdd61 APPROVE for Protocol.php and ProtocolTest.php diff versus f0835c4; focused 18 tests / 68 assertions passed.
- Summary: Fixed user's remaining STDIO response-first ordering P2 centrally: Protocol::exchange rechecks interruption after Fiber::suspend returns, and common catch discards buffered response. Added suspended cancellation/deadline rows asserting notification, cleanup and subsequent request. SDK changes uncommitted and not pushed.
- Ownership: owner=main; fork_run=none; revision=php-sdk:f0835c4+working-tree; scope=post-suspension interruption ordering fix and regression; outcome=completed; commit=none

## Task workflow update - 2026-09-28T15:31:00+00:00
- Summary: User authorized squash and push of STDIO ordering fix. SDK now single commit 3e3e2d8 above 16836d4, pushed to ineersa/php-sdk branch task/add-proper-mcp-tool-call-cancellation with exact force-with-lease against f0835c4969ec99e1b296430482f264889ff5ab6a. Tracked SDK working tree clean; unrelated untracked files preserved. No PR opened; Hatfield worktree unchanged.

## Task workflow update - 2026-09-28T16:11:38+00:00
- Validation: After user updated bubblewrap permissions, make inspector-tests completed successfully (exit status 0 confirmed in supervisor status file): 103 tests, 288 assertions, 7 skipped, 122.420 seconds. npm EROFS blocker no longer reproduces. This does not constitute make ci/conformance/docs-guides validation.

## Task workflow update - 2026-09-28T19:55:12+00:00
- Validation: make ci passed exit 0 on 3e3e2d8: 1775 tests, 4646 assertions, 7 skipped; PHPStan and formatting passed. make ci updated ignored local Composer dependencies firebase/php-jwt and phpdocumentor/type-resolver.; make docs-guides passed strict Zensical build.; 7 Inspector skips are explicit in HttpClientCommunicationTest::setUp: logging/setLevel and built-in PHP server sampling limitations.; make conformance-tests executed: client baseline check passed (12/54 checks; remaining expected). Server reported unexpected failures despite Makefile exit success (runner uses || true); server returned HTTP500.; Diagnosed server failure via fixture container running server.php as www-data: Session directory /app/tests/Conformance/sessions is not writable. Host sessions and logs dirs owned UID/GID65534, mode0755. CI workflow explicitly chmods these dirs to777; same host chmod failed Operation not permitted. Fixture containers/network cleaned up. Conformance not green; generated untracked score JSON files retained as evidence.
- Summary: Full CI and strict docs validation pass. Server conformance blocked by fixture directory permissions, not yet PR-ready on all required validation.

## Task workflow update - 2026-09-28T20:03:32+00:00
- Validation: After user fixed fixture directory permissions, make conformance-server passed: 80/80 checks, zero failures, baseline check passed. Docker fixture containers and network removed by Makefile. Combined with earlier client conformance baseline pass, make ci pass, and strict docs-guides pass, requested validation commands are now complete. Seven Inspector skips are existing explicit logging/sampling limitations.
- Summary: Server conformance permission blocker resolved. No code changes, commits, pushes or PR creation in this validation pass.

## Task workflow update - 2026-09-28T20:19:57+00:00
- Summary: Created upstream SDK PR https://github.com/modelcontextprotocol/php-sdk/pull/518 from ineersa:task/add-proper-mcp-tool-call-cancellation at 3e3e2d8 into main, closing issue #517. PR documents cooperative HTTP limitations, version-aware notifications, full CI/docs/conformance results and that new coverage is unit/STDIO integration rather than new Inspector scenarios. Hatfield integration task stays IN-PROGRESS with local dependency wiring; no Hatfield PR/status transition.
