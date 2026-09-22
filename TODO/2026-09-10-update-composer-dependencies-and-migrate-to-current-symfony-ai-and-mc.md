# Update Composer dependencies and migrate to current Symfony AI and MCP SDK APIs

## Goal
User requests updating dependencies for main with composer update, including Symfony AI, MCP SDK, and other eligible packages. Implement in an isolated task worktree, not directly in the integration checkout. Inspect current constraints and latest compatible releases; identify deliberate constraint changes if needed. Related flat DTO migration task: 2026-09-10-replace-hatfield-flat-dto-tool-adapters-with-symfony-ai-maptoolargume. Its linked current Symfony AI already exposed removal of Symfony\AI\Agent\Toolbox\AgentProcessor and changes to lazy Execution result handling; ConfiguredModelAgentRunner adaptation is staged there and should be assessed/reused without mixing in unreleased MapToolArguments requirements. MCP SDK update need/version must be verified rather than assumed. Use reproducible Composer dependencies, not local symlinks or uncommitted upstream code. Include other Composer packages eligible under project constraints; inspect framework/platform compatibility and breaking changes. No implementation requested in this turn.

## Acceptance criteria
- Review dependency constraints and available Symfony AI, MCP SDK, and other package releases; record chosen versions and required compatibility changes.
- Run composer update in the task worktree and commit reproducible composer.json/composer.lock changes when authorized; no local path dependencies.
- Migrate affected callers to updated supported APIs without compatibility shims; coordinate ConfiguredModelAgentRunner changes with the flat DTO task.
- Validate application startup and packaged PHAR behavior, tool execution, and relevant MCP integration using project Castor requirements; full gate through task workflow.
- Document remaining unreleased features separately, especially MapToolArguments, so the main dependency upgrade does not imply that feature is shipped.

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
- Created: 2026-09-10T23:24:22+00:00

## Task workflow update - 2026-09-22T02:10:11+00:00
- Summary: PR #523 follow-up identified Symfony AI's newly released symfony/ai-mcp-tool package. Its McpToolbox extends AbstractToolbox and forwards flat arguments directly, but requires mcp/sdk ^0.8.1 while Hatfield currently locks 0.7.1. Evaluate replacing Hatfield's per-tool MCP handlers/raw argument wrapping with McpToolbox plus ChainToolbox during this dependency task. Preserve Hatfield's broker-owned per-run clients, parent-session catalog reuse, registry collision/visibility rules, rewrite/policy hooks, ambient execution context, and result/error mapping; do not adopt it if those contracts would be bypassed.
- Follow-up from PR #523: assess symfony/ai-mcp-tool McpToolbox/AbstractToolbox as part of the MCP SDK upgrade, rather than broadening the flat DTO migration PR. RawAwareToolCallArgumentResolver remains necessary for public extension array handlers unless that API is migrated too.
