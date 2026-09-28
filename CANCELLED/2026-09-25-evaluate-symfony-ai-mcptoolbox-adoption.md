# Evaluate Symfony AI McpToolbox adoption for Hatfield MCP tools

## Goal
Investigate whether `symfony/ai-mcp-tool` v0.14 `McpToolbox` can replace Hatfield's custom MCP tool listing, argument forwarding, or per-tool handlers after the MCP SDK upgrade. This is a focused follow-up to `2026-09-10-update-composer-dependencies-and-migrate-to-current-symfony-ai-and-mc`; coordinate dependency changes with that task rather than duplicating its general Composer update. Start from `src/CodingAgent/Mcp/Tool/McpToolRegistrar.php`, `McpToolInvoker.php`, `src/CodingAgent/Tool/RegistryBackedToolbox.php`, and `RawAwareToolCallArgumentResolver.php`. Compare upstream `McpToolbox`, `ToolsetInterface`, and `ChainToolbox` with Hatfield's broker-owned client and per-run catalog. Keep extension raw-array handling in scope only to verify it is not broken. The existing `add-proper-mcp-tool-call-cancellation` task owns request-scoped MCP cancellation and deadlines. Do not open a PR without user permission.

## Acceptance criteria
- Record a code-backed adoption decision, including the minimum `mcp/sdk` version and compatibility with Hatfield's broker-owned connections, parent-session catalog reuse, dynamic registry names and collisions, visibility snapshots, rewrite and policy hooks, ambient run context, and error/result handling.
- If upstream components preserve these contracts with less code, replace only the redundant MCP-specific adapter paths and remove their dead code. Otherwise keep the current implementation and document the specific incompatibilities; do not add a parallel toolbox path.
- Keep `RawAwareToolCallArgumentResolver` support for public extension handlers unless a separate, approved change replaces their raw-array contract.
- Validate any implemented change through focused Castor checks, MCP integration and cancellation regression coverage where relevant, and the task-workflow full QA gate. Do not open a new PR without explicit permission.

## Workflow metadata
Status: CANCELLED
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-09-25T13:39:17+00:00

## Task workflow update - 2026-09-27T22:32:39+00:00
- Summary: Read-only adoption investigation completed with user permission after task-start was blocked by unrelated dirty integration files. Recommendation: retain current MCP path under this task's contract-preservation requirement; do not add symfony/ai-mcp-tool or a parallel toolbox. Upstream v0.14.0 McpToolbox is composable via ToolsetInterface, so broker ownership is not an intrinsic blocker: a custom toolset could list stored catalogs and call McpConnectionManager using ambient run context. However, preserving current behavior requires additional mapping, composition and policy adapters rather than deleting the existing responsibilities. No code/dependency changes or tests run.
- Evidence: upstream https://github.com/symfony/ai-mcp-tool/blob/v0.14.0/composer.json requires mcp/sdk ^0.8.1 and symfony/ai-agent ^0.14, both satisfied by installed dependencies.
- Upstream https://github.com/symfony/ai-mcp-tool/blob/v0.14.0/McpToolbox.php caches its first successful listing, prefixes raw tool names, forwards raw arguments, and privately renders CallToolResult including structuredContent and non-text objects. It is final; rendering/naming policy cannot be overridden in a subclass. ToolsetInterface uses SDK McpTool/CallToolResult types. Our McpClientInterface/McpConnectionManagerInterface intentionally return native arrays; McpSdkClientAdapter::callTool discards structuredContent/meta. Bridging the existing boundary would require reconstruction without recovering omitted data, or a broader boundary migration.
- Catalog/naming evidence: McpToolCatalogBuilder::mapHatfieldName sanitizes both server/tool names and skips sanitized collisions. McpToolRegistrar::registerOneTool skips collisions with permanent/unrelated dynamic tools while preserving remaining tools. Upstream ChainToolbox::collectTools throws on cross-toolbox duplicate names instead. A native composition would need equivalent pre-filtering, alias mapping and invalidation per catalog/run.
- Visibility evidence: CodingAgentToolSetResolver::resolve derives names/allowlists/execution modes from registry definitions. McpCatalogRegisteringToolSetResolver::resolveCatalogRunId prefers child catalog then parent catalog, while invocation obtains current run ID from StackToolExecutionContextAccessor. Removing MCP registration without replacing this integration would omit tools from active schemas/allowlists. Per-server prefix/cache alone cannot replace run-aware visibility or parent-specific availability rules.
- Hooks and errors: RegistryBackedToolbox::applyRewrites precedes native policy events and publishes ToolCallFailedEvent with the rewritten call. A sibling McpToolbox in ChainToolbox would bypass these outer behaviors unless they are relocated. McpToolInvoker classifies connection errors retryable and McpResultMapper classifies isError non-retryable, bounds/redacts error text, and emits safe textual placeholders for binary/resource blocks. Upstream renderer returns richer content objects and structured output; adopting this output unchanged alters the current model-visible contract. RawAwareToolCallArgumentResolver must remain for extension raw-array handlers.
- ClientToolset https://github.com/symfony/ai-mcp-tool/blob/v0.14.0/ClientToolset.php owns lazy connect/disconnect and lists all pages, but lacks our repeated-cursor guard. It should not replace broker-owned connection lifecycle merely to gain pagination. Request-scoped cancellation/deadlines remain owned by add-proper-mcp-tool-call-cancellation.
- Useful upstream improvement identified: structuredContent support and duplicate JSON-text removal. This cannot be claimed as a transparent refactor because our current client boundary omits structuredContent and our result contract is text/placeholders. Consider a separately approved result-contract migration if desired.
- Scout agent_5f3e9183d9cc0ce0 independently mapped local broker/catalog/registry/hook contracts read-only. Parent inspected upstream v0.14.0 McpToolbox, ToolsetInterface, ClientToolset, ToolErrorException, composer manifest, local ChainToolbox, client interfaces, registrar, invoker, mapper, catalog naming, active resolver and raw schema handling. No executable adoption proof was run; conclusions are from source inspection. Task remains TODO; no worktree created.

## Task workflow update - 2026-09-27T22:43:45+00:00
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Cancelled at user request after read-only investigation. Native MCP toolbox adoption does not currently simplify the implementation while preserving catalog, naming, visibility, hook, and result/error contracts. structuredContent is not consumed by the current system. No code, dependencies, worktree, or PR created; investigation evidence remains in the task log.
