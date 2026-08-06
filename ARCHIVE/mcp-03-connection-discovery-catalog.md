# MCP-03 Connection manager, discovery, and session catalog

## Goal
Implement broker-owned MCP connections and tool discovery from `.pi/plans/mcp-client-implementation-plan.md`.

Goal: the single MCP broker owns SDK clients, connects to configured STDIO/HTTP servers, lists tools, and writes a session-scoped MCP catalog.

Scope:
- Implement `McpConnectionManager` or equivalent in the broker process.
- Maintain one SDK client per `(runId, serverName)`.
- STDIO: connect through SDK `StdioTransport`, list tools, keep client alive for the session.
- HTTP: connect through SDK `HttpTransport`, list tools; broker HTTP too in v1 for uniformity.
- Implement session-scoped catalog storage, e.g. `.hatfield/sessions/<runId>/mcp-tools.json` or a DB-backed store if chosen.
- Catalog maps Hatfield tool names to `(serverName, mcpToolName)` and stores descriptions/input schemas/status.
- Failed server discovery should not kill the session; mark server failed and omit its tools.

Depends on: MCP-01, MCP-02.

## Acceptance criteria
- A STDIO fixture/test MCP server can be connected and listed by the broker.
- An HTTP fixture/test MCP server can be connected and listed by the broker.
- A session-scoped MCP catalog is written with namespaced tool names and schemas.
- Failed MCP server discovery is logged/recorded and does not fail the whole run/session.
- The broker owns STDIO clients; normal tool workers do not start STDIO MCP servers.
- Castor tests/validation cover connection and catalog behavior.

## Workflow metadata
Status: DONE
Branch: task/mcp-03-connection-discovery-catalog
Worktree: /home/ineersa/projects/agent-core-worktrees/mcp-03-connection-discovery-catalog
Fork run: baqu68wfzx1h
PR URL: https://github.com/ineersa/agent-core/pull/172
PR Status: merged
Started: 2026-06-18T21:58:08.101Z
Completed: 2026-06-19T01:01:16.226Z

## Work log
- Created: 2026-06-12T18:06:33.823Z

## Task workflow update - 2026-06-18T21:58:04.270Z
- Summary: Task-start decisions recorded before implementation:

- Durable MCP catalog: use session-file catalog as source of truth at `.hatfield/sessions/<runId>/mcp-tools.json` (where `session_id === run_id`). Do not use generic `app.cache` as the canonical catalog because this is session state, not a performance cache; cache clears/adapters/serialization would make active tools less durable and less inspectable. A dedicated cache pool may be considered later only as an optimization, not the source of truth.
- Rediscovery model: full snapshot replacement, not incremental mutation. On start, resume, and explicit refresh, load current MCP config, connect/list enabled servers, and atomically replace the catalog snapshot (temp file + rename) so readers see either the previous complete snapshot or the next complete snapshot.
- Invalidation model: new discovery generation replaces previous catalog. Include metadata such as generatedAt/generation and configHash if practical. If config changes, new configHash/generation makes older catalogs stale for later readers. Failed server discovery must write a failed server entry with no tools and diagnostic-safe error info; never silently retain stale tools from a previously-successful discovery. Disconnect closes broker-owned clients but may retain the catalog as historical/debug session artifact; full lifecycle cleanup/hardening remains MCP-06.
- Discovery remains asynchronous in MCP-03. Do not block first LLM schema resolution on catalog readiness in this task; defer LLM/catalog synchronization and schema registration barrier to MCP-04.
- Shutdown wiring stays minimal in MCP-03: implement/use disconnect command handling for connection manager cleanup, but deeper graceful shutdown/orphan cleanup remains MCP-06.
- Scope boundary: MCP-03 owns broker-side connection manager, server connect/listTools discovery, session catalog write/read shape, namespaced Hatfield tool names and reverse mapping. MCP-04 will register dynamic tools from catalog. MCP-05 will implement callTool request/reply and result mapping.

## Task workflow update - 2026-06-18T21:58:08.101Z
- Moved TODO → IN-PROGRESS.
- Created branch task/mcp-03-connection-discovery-catalog.
- Created worktree /home/ineersa/projects/agent-core-worktrees/mcp-03-connection-discovery-catalog.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/mcp-03-connection-discovery-catalog.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/mcp-03-connection-discovery-catalog.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/mcp-03-connection-discovery-catalog.

## Task workflow update - 2026-06-18T22:00:16.489Z
- Recorded fork run: hegfymh5boah
- Summary: Implementation fork launched for MCP-03.

Fork run: hegfymh5boah
Worktree: /home/ineersa/projects/agent-core-worktrees/mcp-03-connection-discovery-catalog
Branch: task/mcp-03-connection-discovery-catalog

Fork instructions cover: broker-owned McpConnectionManager, session-file catalog at `.hatfield/sessions/<runId>/mcp-tools.json`, full-snapshot rediscovery/invalidation, async discovery with no LLM schema barrier, failed-server warning/failed catalog entries, structured secret-safe logs, name mapping/sanitization/reverse mapping, minimal disconnect cleanup, Deptrac/service/docs wiring, STDIO and HTTP fixture discovery tests, and Castor-only validation. Fork was explicitly instructed to read AGENTS.md, testing skill, and tests/AGENTS.md before runtime/test work and to commit changes before handoff.

## Task workflow update - 2026-06-18T22:10:32.483Z
- Recorded fork run: hegfymh5boah
- Summary: Implementation fork hegfymh5boah completed at commit 0aa1a085d on branch task/mcp-03-connection-discovery-catalog. Parent verification: worktree clean; diff vs origin/main is 17 files changed, 1930 insertions, 115 deletions. Fork reported Castor validation passing: castor test --filter=Mcp (59 tests/196 assertions), castor deptrac (0 violations), castor phpstan (0 errors), castor cs-check clean, full castor test (2773 tests/8289 assertions) OK.

Parent did NOT accept this as CODE-REVIEW-ready yet. Follow-up required because the handoff itself identifies an unmet acceptance criterion: no HTTP fixture/test MCP server was implemented, despite task acceptance and fork instructions requiring HTTP discovery/listing coverage. Parent spot-check also found polish/blockers to address before review: refresh failure currently preserves the previous catalog, contradicting the recorded full-snapshot invalidation decision; a production catch block swallows catalog invalidation write failures without diagnostic logging; error_message logging uses raw exception messages in some paths instead of the sanitized/truncated message; duplicate Hatfield tool-name detection is only per-server and can miss cross-server sanitized-name collisions; configHash currently excludes many discovery-affecting fields; McpConnectionManager was made non-final solely for test stubbing, which should be reconsidered or replaced with a real production interface if abstraction is needed.

## Task workflow update - 2026-06-18T22:11:14.439Z
- Recorded fork run: fwk9piioi0yb
- Summary: Follow-up implementation fork launched because initial fork handoff was not accepted as CODE-REVIEW-ready. Fork run: fwk9piioi0yb.

Follow-up scope: add required real HTTP fixture/discovery test; change refresh failure to invalidate stale catalog with safe empty snapshot instead of preserving prior catalog; add diagnostic logging for failed empty-catalog writes; sanitize all logged/catalog error messages; detect duplicate Hatfield names across whole catalog; improve configHash to include discovery-affecting fields; reconsider McpConnectionManager non-final test-only shape (restore final or add real interface); remove double config load in buildCatalog if low risk; run Castor validation and commit follow-up changes.

## Task workflow update - 2026-06-18T22:25:02.595Z
- Recorded fork run: fwk9piioi0yb
- Summary: Follow-up fork fwk9piioi0yb completed at commit 6d4c9ef57 and resolved the previously identified functional blockers: real HTTP fixture discovery test added, refresh failures now invalidate catalog with empty snapshot, silent catch replaced with diagnostic logging, secret-like messages sanitized, cross-server duplicate Hatfield names detected, config hash expanded, McpConnectionManager restored final behind McpConnectionManagerInterface. Fork validation reported: castor test --filter=Mcp (63 tests/230 assertions), full castor test (2777 tests/8323 assertions), deptrac 0 violations, phpstan 0 errors, cs-check clean.

Parent verification found a small cleanup blocker before accepting as ready for task-to-pr: the follow-up added unused dev dependency nyholm/psr7-server (fixture no longer uses it) and regenerated phpstan-baseline.neon to suppress new missing iterable-type issues in new MCP code. New code should be annotated/fixed rather than adding baseline entries, and unused Composer dependency should be removed.

## Task workflow update - 2026-06-18T22:25:23.742Z
- Recorded fork run: eyxspbcp6ouk
- Summary: Final cleanup fork launched: eyxspbcp6ouk. Scope: remove unused nyholm/psr7-server dev dependency from composer.json/lock, remove MCP-03-added phpstan-baseline suppressions, fix new-code iterable PHPStan issues with precise PHPDoc instead of baseline entries, run focused Castor validation, and commit cleanup changes.

## Task workflow update - 2026-06-19T00:32:18.536Z
- Recorded fork run: eyxspbcp6ouk
- Summary: Cleanup fork eyxspbcp6ouk completed at commit c43b1946a and removed the unused nyholm/psr7-server root dev dependency plus MCP-03-added phpstan baseline suppressions. Focused Castor validation reported passing: phpstan 0 errors, castor test --filter=Mcp 63 tests/230 assertions, deptrac 0 violations, cs-check clean.

Parent verification found one remaining PR-noise cleanup issue: composer.json now matches origin/main, so composer.lock should also have no MCP-03 dependency diff, but it still contains unrelated package version bumps (doctrine-bundle 3.2.3→3.2.4, guzzlehttp/guzzle 7.11.0→7.12.1, guzzlehttp/psr7 2.11.0→2.12.1, symfony/polyfill-deepclone 1.37.0→1.40.0, php-cs-fixer 3.95.7→3.95.8). These are unrelated to MCP-03 and should be reverted/minimized before PR. Also phpstan-baseline only has harmless extra blank lines from remove cycle; can clean while touching cleanup if desired.

## Task workflow update - 2026-06-19T00:34:55.765Z
- Recorded fork run: mpxbjmoiyzff
- Summary: Final PR-noise cleanup fork mpxbjmoiyzff completed at commit 98e4ca0fe. It restored composer.lock and phpstan-baseline.neon to origin/main, leaving composer.json/composer.lock/phpstan-baseline.neon byte-identical to origin/main (no MCP-03 dependency/baseline noise). Reported validation: castor phpstan 0 errors, castor cs-check clean, castor test --filter=Mcp 63 tests/230 assertions OK, castor deptrac 0 violations. Parent spot-check confirmed no diff for composer.json/composer.lock/phpstan-baseline.neon and worktree is on branch task/mcp-03-connection-discovery-catalog at HEAD 98e4ca0fe.

## Task workflow update - 2026-06-19T00:38:51.768Z
- Validation: Reviewer verdict: APPROVE WITH SUGGESTIONS (no blockers).; castor test: OK (2777 tests, 8323 assertions).; castor deptrac: 0 violations.; castor phpstan: 0 errors, 0 file_errors.; castor cs-check: files_fixed=0.
- Summary: Reviewer subagent completed final MCP-03 review on HEAD 98e4ca0fe with verdict: APPROVE WITH SUGGESTIONS. No critical/blocking issues. Non-blocking notes: possible narrow config-hash/catalog inconsistency if handler config load succeeds but connection manager's redundant config load fails under concurrent config change; handler still uses concrete McpConnectionManager static sanitizer despite depending on McpConnectionManagerInterface; sanitizer patterns are best-effort. Reviewer confirmed all prior blockers are addressed and no composer/phpstan-baseline noise remains.

## Task workflow update - 2026-06-19T00:40:30.551Z
- Summary: First attempt to move MCP-03 to CODE-REVIEW failed at deterministic castor check gate. Focused validation had passed, but castor check's unit/integration test lane exited 1 because PHPUnit treated a PHP notice as an issue: vendor/mcp/sdk/src/Client/Transport/StdioTransport.php:99 fwrite(): Broken pipe, triggered by McpConnectionManagerTest::testDiscoverFailedServerReturnsFailedStatus. Controller replay and TUI lanes passed. Root cause appears to be the failed-server regression test using a nonexistent STDIO command, which can race into a vendor broken-pipe notice during connection failure. Need a focused test fix that preserves failed-discovery behavior without relying on a broken STDIO process (e.g. failed HTTP endpoint/unused port) and then rerun validation.

## Task workflow update - 2026-06-19T00:43:35.794Z
- Recorded fork run: baqu68wfzx1h
- Validation: Fork validation: castor test --filter=McpConnectionManagerTest OK (6 tests, 47 assertions).; Fork validation: castor test OK (2777 tests, 8326 assertions).; Fork validation: castor phpstan 0 errors, deptrac 0 violations, cs-check clean.; Fork validation: LLM_MODE=true castor check passed all 6 lanes in 84.7s.
- Summary: Gate-fix fork baqu68wfzx1h completed at commit 464321b87. It changed McpConnectionManagerTest::testDiscoverFailedServerReturnsFailedStatus from a nonexistent STDIO command to an unused localhost HTTP endpoint so failed-discovery behavior is still covered without vendor StdioTransport broken-pipe PHP notices. Parent spot-check confirmed worktree clean at HEAD 464321b87; only test file changed in the fix commit.

## Task workflow update - 2026-06-19T00:44:39.988Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (48.3s).
- Pushed task/mcp-03-connection-discovery-catalog to origin.
- branch 'task/mcp-03-connection-discovery-catalog' set up to track 'origin/task/mcp-03-connection-discovery-catalog'.
- Created PR: https://github.com/ineersa/agent-core/pull/172
- Validation: Reviewer subagent verdict: APPROVE WITH SUGGESTIONS (no blockers).; Parent focused validation before first CODE-REVIEW attempt: castor test OK (2777 tests, 8323 assertions), deptrac 0 violations, phpstan 0 errors, cs-check clean.; Gate-fix fork validation: castor test --filter=McpConnectionManagerTest OK (6 tests, 47 assertions); castor test OK (2777 tests, 8326 assertions); phpstan 0 errors; deptrac 0 violations; cs-check clean; LLM_MODE=true castor check passed all 6 lanes in 84.7s.
- Summary: MCP-03 ready for code review at commit 464321b87. Implementation adds broker-owned McpConnectionManagerInterface/McpConnectionManager, STDIO+HTTP discovery coverage, session-file catalog store, Hatfield tool name mapping with cross-server collision detection, failed-server catalog entries, refresh invalidation, disconnect cleanup, structured secret-safe logs, and docs/deptrac/service wiring. Follow-up cleanup removed dependency/baseline noise; composer.json/composer.lock/phpstan-baseline.neon have no diff vs origin/main. Reviewer verdict: APPROVE WITH SUGGESTIONS with no blockers. A final gate-fix commit replaced the failed-server test's nonexistent STDIO command with a failed HTTP endpoint to avoid vendor StdioTransport broken-pipe notices; fork-reported LLM_MODE=true castor check passed all lanes.

## Task workflow update - 2026-06-19T01:01:16.227Z
- Moved CODE-REVIEW → DONE.
- Merged task/mcp-03-connection-discovery-catalog into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                               |  25 +-
 depfile.yaml                                       |   9 +-
 docs/mcp.md                                        |  86 ++++-
 .../Mcp/Catalog/McpServerCatalogEntryDTO.php       |  76 ++++
 .../Mcp/Catalog/McpServerCatalogStatusEnum.php     |  17 +
 src/CodingAgent/Mcp/Catalog/McpToolCatalogDTO.php  |  99 +++++
 .../Mcp/Catalog/McpToolCatalogStoreInterface.php   |  30 ++
 .../Mcp/Catalog/McpToolDefinitionDTO.php           |  65 ++++
 src/CodingAgent/Mcp/Catalog/McpToolNameMapper.php  |  76 ++++
 .../Mcp/Catalog/SessionFileMcpToolCatalogStore.php | 126 +++++++
 .../Mcp/Client/McpConnectionManager.php            | 245 ++++++++++++
 .../Mcp/Client/McpConnectionManagerInterface.php   |  55 +++
 .../Mcp/Handler/McpInitializeSessionHandler.php    | 360 ++++++++++++++++--
 .../Mcp/Catalog/McpToolNameMapperTest.php          |  98 +++++
 .../Catalog/SessionFileMcpToolCatalogStoreTest.php | 303 +++++++++++++++
 .../Mcp/Client/McpConnectionManagerTest.php        | 411 +++++++++++++++++++++
 .../CodingAgent/Mcp/Fixtures/http-echo-server.php  | 169 +++++++++
 .../CodingAgent/Mcp/Fixtures/stdio-echo-server.php |  86 +++++
 .../Handler/McpInitializeSessionHandlerTest.php    | 376 +++++++++++++++----
 19 files changed, 2597 insertions(+), 115 deletions(-)
 create mode 100644 src/CodingAgent/Mcp/Catalog/McpServerCatalogEntryDTO.php
 create mode 100644 src/CodingAgent/Mcp/Catalog/McpServerCatalogStatusEnum.php
 create mode 100644 src/CodingAgent/Mcp/Catalog/McpToolCatalogDTO.php
 create mode 100644 src/CodingAgent/Mcp/Catalog/McpToolCatalogStoreInterface.php
 create mode 100644 src/CodingAgent/Mcp/Catalog/McpToolDefinitionDTO.php
 create mode 100644 src/CodingAgent/Mcp/Catalog/McpToolNameMapper.php
 create mode 100644 src/CodingAgent/Mcp/Catalog/SessionFileMcpToolCatalogStore.php
 create mode 100644 src/CodingAgent/Mcp/Client/McpConnectionManager.php
 create mode 100644 src/CodingAgent/Mcp/Client/McpConnectionManagerInterface.php
 create mode 100644 tests/CodingAgent/Mcp/Catalog/McpToolNameMapperTest.php
 create mode 100644 tests/CodingAgent/Mcp/Catalog/SessionFileMcpToolCatalogStoreTest.php
 create mode 100644 tests/CodingAgent/Mcp/Client/McpConnectionManagerTest.php
 create mode 100644 tests/CodingAgent/Mcp/Fixtures/http-echo-server.php
 create mode 100644 tests/CodingAgent/Mcp/Fixtures/stdio-echo-server.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/mcp-03-connection-discovery-catalog.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/mcp-03-connection-discovery-catalog.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #172 merged by user before DONE transition.
- Summary: User confirmed PR #172 was merged; moving MCP-03 to DONE and merging task branch into integration checkout.

## Task workflow update - 2026-06-19T01:02:54.258Z
- Validation: LLM_MODE=true castor check on integration checkout: quality OK (93.3s).; check lanes: deptrac OK (1.0s); test OK (2800 tests, 8356 assertions, 15.2s); test:controller-replay OK (3 tests, 41 assertions, 24.3s); test:tui OK (10 tests, 93 assertions, 44.4s); phpstan OK (0 errors, 6.2s); cs-check OK (2.1s).; Worktree /home/ineersa/projects/agent-core-worktrees/mcp-03-connection-discovery-catalog removed by DONE transition; IDEA exclusions cleaned.
- Summary: Post-merge validation completed on integration checkout after moving MCP-03 to DONE. The task branch was merged into the integration checkout, PR #172 changes are present, worktree cleanup completed, and parent waited for an unrelated castor check in a sibling worktree to finish before running validation to avoid test-resource contention.
