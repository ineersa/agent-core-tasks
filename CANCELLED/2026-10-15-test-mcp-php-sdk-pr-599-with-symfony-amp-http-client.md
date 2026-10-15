# Test MCP PHP SDK PR 599 with Symfony Amp HTTP client

## Goal
## Goal
Set up an isolated Hatfield worktree to test https://github.com/modelcontextprotocol/php-sdk/pull/599 against the HTTP transport problem originally reported in issue #504.

## Verified starting point
Hatfield's McpSdkClientFactory explicitly injects Symfony Psr18Client backed by CurlHttpClient to avoid the SDK resuming an Amp I/O suspension with an MCP timeout Error. Main currently locks mcp/sdk dev-main at bb74bbe0318046dc00153b9b9cfb66b65359bb33. PR head inspected: 9cb9a0c8f8b8ef6c19480c5d61597ccf4a041fd0.

## Scope
Experimental validation only. Compare the upstream PR base and pinned head with Symfony Psr18Client backed by AmpHttpClient against an isolated local MCP HTTP server. Keep explicit cURL as a control and inspect discovery separately. Exercise handshake, tool listing and a tool call, plus relevant focused Hatfield regressions. Record exact dependencies, commands, outcomes and limitations.

Do not change main, adopt a production HTTP-client default, push, or open a Hatfield PR. Keep dependencies and test artifacts isolated in the task worktree. No real project sessions or external credentials.

## Acceptance criteria
- An isolated worktree is available with reproducible test artifacts and pinned upstream revisions.
- A baseline-versus-PR comparison uses the real Symfony Amp client, not only a fake suspending client, with cURL as a control.
- Record observed PSR discovery behavior and focused Hatfield MCP test results.
- Report whether the original failure reproduces and whether PR 599 fixes it, without claiming untested cancellation, listening, or concurrency behavior.

## Workflow metadata
Status: CANCELLED
Branch: task/2026-10-15-test-mcp-php-sdk-pr-599-with-symfony-amp-http-client
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-10-15-test-mcp-php-sdk-pr-599-with-symfony-amp-http-client
Fork run: 2b607b40-5d1e-5441-9e77-6de2a5f82cfe
PR URL:
PR Status:
Started: 2026-10-15T16:26:47+00:00
Completed:

## Work log
- Created: 2026-10-15T16:26:29+00:00

## Task workflow update - 2026-10-15T16:26:47+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-10-15-test-mcp-php-sdk-pr-599-with-symfony-amp-http-client.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-10-15-test-mcp-php-sdk-pr-599-with-symfony-amp-http-client.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-10-15-test-mcp-php-sdk-pr-599-with-symfony-amp-http-client.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-10-15-test-mcp-php-sdk-pr-599-with-symfony-amp-http-client.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-10-15-test-mcp-php-sdk-pr-599-with-symfony-amp-http-client.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-10-15-test-mcp-php-sdk-pr-599-with-symfony-amp-http-client/.idea.
- Summary: User requested an isolated worktree to test upstream MCP SDK PR599 and verify the explicit cURL workaround. Scope is experimental validation, not adopting a production default or submitting a Hatfield PR.

## Task workflow update - 2026-10-15T16:28:03+00:00
- Summary: Main inspected factory, adapter tests, locked dependency, upstream diff and PHP runtime. Worktree is clean at 94073221f. No Amp packages currently installed. Assigning one implementation fork for the experiment; production default must remain unchanged.
- Ownership: owner=fork; fork_run=none; revision=94073221f687b295d9b7321f7fb6ae6f80ec3346; scope=isolated PR599 baseline/head Symfony Amp and cURL experiments plus focused Hatfield MCP regression proof; outcome=assigned; commit=none

## Task workflow update - 2026-10-15T16:39:04+00:00
- Recorded fork run: 019ec9a4-e000-7000-8000-000000000000
- Summary: Initial experiment returned baseline Amp failure and PR-head success. Main inspection found probe cleanup could overwrite the native baseline error with its alarm and empty catch blocks violate project rules. Resuming the same writer to tighten the probe before accepting results.
- Ownership: owner=fork; fork_run=agent_8f196e3583e6e8f1; revision=94073221f687b295d9b7321f7fb6ae6f80ec3346; scope=probe failure preservation, explicit cleanup logging, outcome assertions and phase timing in existing isolated experiment; outcome=assigned; commit=none

## Task workflow update - 2026-10-15T16:39:38+00:00
- Recorded fork run: 2b607b40-5d1e-5441-9e77-6de2a5f82cfe
- Correction: previous forkRun value was recorded in error. Actual fork run verified from artifact metadata is 2b607b40-5d1e-5441-9e77-6de2a5f82cfe, artifact agent_8f196e3583e6e8f1.
- Ownership: owner=fork; fork_run=2b607b40-5d1e-5441-9e77-6de2a5f82cfe; revision=94073221f687b295d9b7321f7fb6ae6f80ec3346; scope=probe failure preservation, explicit cleanup logging, outcome assertions and phase timing in existing isolated experiment; outcome=assigned; commit=none

## Task workflow update - 2026-10-15T16:45:58+00:00
- Recorded fork run: 2b607b40-5d1e-5441-9e77-6de2a5f82cfe
- Validation: castor --castor-file=var/tmp/mcp-php-sdk-pr599/harness/castor.php mcp:pr599-probe --timeout=3: nine-case matrix, expected baseline+locked Amp failures; head Amp and all Curl/discovery controls pass.; castor test --filter='McpConnectionManagerTest|McpSdkClientAdapterTest': 28 tests/197 assertions, zero failures/errors/skips (JUnit independently checked by main).; Report: var/tmp/mcp-php-sdk-pr599/reports/REPORT.md; structured outcomes summary.json and per-case JSON/logs. Dependencies identical across SDK trees; base/head source delta matches upstream PR.; Not tested: listen:true, HTTP cancellation, concurrency, ReactPHP; Amp performance remains unexplained. Main production Curl pin unchanged.
- Summary: Experimental comparison completed in isolated worktree, no tracked changes or production client changes. Main verified latest report/probe, exact PR-head checkout, clean main/task status, absence of leftover probe servers, and focused regression JUnit (28 tests/197 assertions, zero issues). Real Symfony Amp fails on upstream base d1e809153 and Hatfield-locked bb74bbe with native FutureIterator TypeError; PR head9cb9a0c succeeds initialize/listTools/calculate=5. Curl and discovery pass all revisions; discovery selects Curl. Amp head request phases are ~1s each, not disconnect; no broad performance claim. Worktree remains available with vendor/mcp/sdk symlinked to isolated PR head; composer manifests untouched, original locked vendor tree preserved. Full gate, independent review, adoption, push and PR intentionally not performed.
- Ownership: owner=fork; fork_run=2b607b40-5d1e-5441-9e77-6de2a5f82cfe; revision=94073221f687b295d9b7321f7fb6ae6f80ec3346; scope=isolated PR599 baseline/head Symfony Amp and cURL experiments plus focused Hatfield MCP regression proof; outcome=completed; commit=none

## Task workflow update - 2026-10-15T17:08:32+00:00
- Moved IN-PROGRESS → CANCELLED.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-10-15-test-mcp-php-sdk-pr-599-with-symfony-amp-http-client.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-10-15-test-mcp-php-sdk-pr-599-with-symfony-amp-http-client.
- Validation: PR599 head9cb9a0c real Symfony Amp initialize/listTools/calculate passes; upstream base and Hatfield locked SDK reproduce native TypeError. Curl/discovery pass all revisions.; Focused Hatfield MCP regressions:28 tests/197 assertions, zero issues. Evidence retained in external artifact archive.; No probe/server processes found; worktree git status clean; report archive SHA256 matches original.
- Summary: User requested wrapping up the experimental worktree after successful testing. Investigation completed successfully; CANCELLED is used only as the workflow's no-merge worktree teardown path, not because tests failed. No implementation, adoption or PR is planned. Preserved report, structured outcomes, logs, harness, dependency manifests and focused JUnit outside the worktree at /home/ineersa/projects/agent-core-tasks/artifacts/2026-10-15-mcp-sdk-pr599 (676 KiB). Verified report/log/harness copies match originals. Worktree has no tracked/untracked changes, no unique task commits, and no running probe servers. Ignored SDK clones/dependencies and generated QA caches may be discarded with the worktree. Main and production Curl pin unchanged.
