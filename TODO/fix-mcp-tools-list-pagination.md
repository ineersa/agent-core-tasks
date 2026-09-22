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
Status: TODO
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-09-12T01:41:06+00:00
