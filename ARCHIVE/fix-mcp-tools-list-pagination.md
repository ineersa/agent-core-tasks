# Fix MCP tools/list pagination: follow nextCursor in McpSdkClientAdapter

## Goal
## Bug

`McpSdkClientAdapter::listTools()` (src/CodingAgent/Mcp/Client/McpSdkClientAdapter.php:57-75) calls `$this->client->listTools()` once and discards `ListToolsResult::$nextCursor`. The mcp/sdk `Client::listTools(?string $cursor = null): ListToolsResult` is paginated — one page per call. A server advertising more tools than one page silently truncates our session catalog (`mcp-tools.json`), so those tools never become LLM-visible or callable.

User hit this before with the Symfony MCP bundle; rediscovered while reviewing symfony/ai#2497 (`ClientToolset::getTools()`), whose cursor loop is the correct reference pattern:

```php
$cursor = null;
do {
    $page = $this->client->listTools($cursor);
    // collect $page->tools
    $cursor = $page->nextCursor;
} while (null !== $cursor);
```

## Scope

- Fix only `McpSdkClientAdapter::listTools()`. `McpConnectionManager::discover()`, catalog builder, and registrar need no changes — they consume the adapter's aggregated array.
- Guard against a hostile/buggy server looping the same cursor: a repeated identical non-null cursor must not spin forever (break or fail with the existing exception path; pick the deterministic option and document it).
- Tests live in tests/CodingAgent/Mcp/Client/McpSdkClientAdapterTest.php (extend the existing double/mock pattern there; see tests/AGENTS.md + testing skill before touching tests).

## Validation

Targeted `castor test` filter for the MCP client tests is sufficient — this is a pure adapter change below the TUI/runtime/Messenger lane thresholds. Full `castor check` belongs to the move_task(to=CODE-REVIEW) gate, not this phase.

## References

- Upstream cursor-loop pattern: symfony/ai PR #2497, src/agent/src/Bridge/Mcp/ClientToolset.php
- SDK contract: vendor/mcp/sdk/src/Client.php listTools(), Schema/Result/ListToolsResult.php (readonly ?string $nextCursor)
- Pipeline context: McpConnectionManager::discover() → McpToolCatalogBuilder → session mcp-tools.json → McpToolRegistrar

## Acceptance criteria
- McpSdkClientAdapter::listTools() follows nextCursor until null and returns tools from every page
- Multi-page discovery proven at the lowest correct layer (mocked/doubled SDK client in McpSdkClientAdapterTest), no live server required
- No arbitrary sleeps/retries; error propagation unchanged (SDK exceptions surface as today)
- Targeted castor test lane green; no production behavior changes beyond complete tool listing

## Workflow metadata
Status: DONE
Branch: task/fix-mcp-tools-list-pagination
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-mcp-tools-list-pagination
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/535
PR Status: merged
Started: 2026-09-27T18:49:57+00:00
Completed: 2026-09-27T19:09:43+00:00

## Work log
- Created: 2026-09-12T01:41:06+00:00

## Task workflow update - 2026-09-27T18:49:57+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-mcp-tools-list-pagination.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-mcp-tools-list-pagination.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-mcp-tools-list-pagination.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-mcp-tools-list-pagination.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-mcp-tools-list-pagination.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/fix-mcp-tools-list-pagination/.idea.

## Task workflow update - 2026-09-27T18:50:46+00:00
- Ownership: owner=main; fork_run=none; revision=8490bee72; scope=McpSdkClientAdapter::listTools pagination and adapter regression tests; outcome=assigned; commit=none

## Task workflow update - 2026-09-27T18:54:21+00:00
- Validation: castor test --filter=McpSdkClientAdapterTest: 11 tests, 43 assertions passed; castor phpstan --path=src/CodingAgent/Mcp/Client/McpSdkClientAdapter.php: passed; castor cs-check: passed; git diff --check: passed
- Summary: Adapter now collects every tools/list page and fails discovery if a server repeats a cursor; added adapter tests for multi-page listings and cursor cycles.
- Ownership: owner=main; fork_run=none; revision=8490bee72; scope=McpSdkClientAdapter::listTools pagination and adapter regression tests; outcome=completed; commit=22c6b4713

## Task workflow update - 2026-09-27T19:03:15+00:00
- Validation: Reviewer confirmed the SDK cursor contract and adapter test coverage; focused implementation validation applies unchanged to this revision.
- Summary: Independent read-only review approved revision 22c6b471337a3159f9ec162c10121abc7d917d7d with optional suggestions only. No blockers or code changes.
- Review: role=reviewer; artifact=agent_7b0c48c0b1126645; revision=22c6b471337a3159f9ec162c10121abc7d917d7d; scope=specification fidelity, pagination, cursor-cycle handling, and regression tests; decision=APPROVE WITH SUGGESTIONS; blockers=none

## Task workflow update - 2026-09-27T19:05:02+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (96.5s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/fix-mcp-tools-list-pagination/var/reports/qa-20260927-190326-1831-74e7cc4d.
- Session/run: 71.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-27T19:05:04+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/fix-mcp-tools-list-pagination to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/fix-mcp-tools-list-pagination/var/reports/qa-20260927-190326-1831-74e7cc4d.
- Session/run: 71.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-27T19:05:06+00:00
- castor check passed (96.5s).
- Pushed task/fix-mcp-tools-list-pagination to origin.
- Created PR: <url>
- Session/run: 71.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-27T19:05:06+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (96.5s).
- Pushed task/fix-mcp-tools-list-pagination to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/535

## Task workflow update - 2026-09-27T19:09:43+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/fix-mcp-tools-list-pagination into integration checkout.
- Merge made by the 'ort' strategy.
 src/CodingAgent/Mcp/Client/McpSdkClientAdapter.php       | 33 ++++++++++++++++++++++++---------
 tests/CodingAgent/Mcp/Client/McpSdkClientAdapterTest.php | 49 +++++++++++++++++++++++++++++++++++++++++++++++++
 2 files changed, 73 insertions(+), 9 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-mcp-tools-list-pagination.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-mcp-tools-list-pagination.
- Pulled integration checkout: Merge made by the 'ort' strategy.
 .hatfield/extensions/observational-memory/composer.json                         |  10 +++++-----
 .hatfield/extensions/observational-memory/src/Semantic/MemoryStoreAdapter.php   |  18 ++++++++++++------
 .hatfield/extensions/observational-memory/src/Semantic/SemanticIndexService.php |   7 ++++---
 composer.json                                                                   |  14 +++++++-------
 composer.lock                                                                   | 130 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++------------------------------------------------------------------------
 src/AgentCore/Application/Handler/ToolExecutor.php                              |   8 +++++++-
 src/CodingAgent/Tool/RegistryBackedToolbox.php                                  |  12 ------------
 src/Platform/Bridge/Generic/DurableResultConverter.php                          |  20 +++++++++++++++-----
 tests/AgentCore/Application/Handler/ToolExecutorTest.php                        |  59 +++++++++++++++++++++++++++++++++++++++++++++++++++--------
 tests/CodingAgent/Tool/BashToolTest.php                                         |  11 +++--------
 tests/CodingAgent/Tool/BgStatusToolTest.php                                     |   9 +++------
 tests/CodingAgent/Tool/RegistryBackedToolboxTest.php                            |  33 ++++++++++++++-------------------
 tests/Platform/Bridge/Generic/DurableResultConverterTest.php                    |  40 ++++++++++++++++++++++++++++++++++++++++
 13 files changed, 219 insertions(+), 152 deletions(-).

## Task workflow update - 2026-09-27T19:11:33+00:00
- Validation: Post-merge castor check on 63ca629fb: test, phpstan, dead-code failed; other eight lanes passed. Reports: var/reports/qa-20260927-190951-6898-dacecf92. Test failure references ToolExecutorTest; static failures reference observational-memory VectorDocumentInterface absent from stale v0.13.0 vendor. Integration validation incomplete.
- Summary: Post-merge castor check failed on the integration checkout because root vendor still has symfony/ai-store v0.13.0 while merged composer.lock requires v0.14.0. Failure is outside this MCP change; aligning dependencies before rechecking.

## Task workflow update - 2026-09-27T19:13:28+00:00
- Validation: Post-merge castor check on integration revision 63ca629fb: passed all 11 lanes in 151.5s. Reports: var/reports/qa-20260927-191154-9209-9c7cbfb4. First check failed with stale root vendor (symfony/ai-store 0.13.0 versus locked 0.14.0); composer install --no-interaction --prefer-dist aligned vendor before the successful run.; git status --porcelain=v1: clean; task worktree absent. Integration main ahead of origin/main by two local merge commits.
- Summary: Post-merge validation passed after installing dependencies from the merged composer.lock. Integration checkout is clean; task worktree was removed.
