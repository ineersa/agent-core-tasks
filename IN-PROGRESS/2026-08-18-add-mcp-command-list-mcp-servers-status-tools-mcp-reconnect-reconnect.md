# Add /mcp command: list MCP servers/status/tools + /mcp reconnect (reconnect all)

## Goal
Add a built-in `/mcp` slash command exposing MCP server state to the user:

1. `/mcp` — lists available MCP servers (from settings/config) with their connection statuses and tools (names/counts — whatever the existing MCP infrastructure exposes cheaply).
2. `/mcp reconnect` — triggers reconnect to ALL configured MCP servers in one shot.

Design decision (user-resolved): an interactive picker with per-server reconnect was considered and REJECTED — reconnect-to-all is deliberately simpler. Do not add per-server selection, per-server reconnect flags, or subcommands beyond these two.

Context:
- MCP infrastructure already exists in `src/CodingAgent/Mcp/` (manager/client layer, `mcp` consumer queue, `McpWorkerShutdownSubscriber`); a scout at task-start should map it before designing — the command handler should be a thin adapter over existing MCP services, not new lifecycle machinery.
- `/mcp reconnect` is the cheap targeted fix for stale MCP connections, complementary to `/reload` (SETTINGS-06) which rebuilds everything.
- Follow the built-in slash-command pattern: catalog metadata + per-session bind via a registrar (see `SessionCommandRegistrar` / `ReloadCommandHandler` for the established shape); prompt templates cannot shadow built-ins.
- Related, NOT in scope: PHAR-mode `/reload` verification (known untested corner of SETTINGS-06, tracked informally).

## Acceptance criteria
- /mcp is a built-in local slash command (unshadowable by prompt templates) listing configured MCP servers with live connection status and their tools (per-server: name, status, tool names or counts as the infra exposes).
- /mcp reconnect reconnects ALL configured MCP servers in place (no /reload, no container rebuild) and reports per-server outcome; failures are surfaced with actionable messages, not swallowed.
- Guard behavior during nonterminal activity matches the constraints of the underlying MCP operations as the scout maps them (e.g. reject while a run is active if reconnect would break in-flight MCP tool calls); reuse existing guard patterns (RunActivityStateEnum/CompactCommandHandler style).
- Every caught exception rethrown/propagated or explicitly logged as intentional local degradation; no empty catches.
- Tests at the lowest correct layer per testing skill + tests/AGENTS.md (likely virtual for /mcp listing + focused unit/replay for reconnect against the existing MCP manager); focused castor validation; full castor check before review since MCP/runtime paths are touched.
- No speculative surface: no per-server picker, no subcommands beyond the two specified.

## Workflow metadata
Status: IN-PROGRESS
Branch: task/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect
Fork run:
PR URL:
PR Status:
Started: 2026-08-18T18:56:48.341Z
Completed:

## Work log
- Created: 2026-08-18T17:52:22.159Z

## Task workflow update - 2026-08-18T18:56:48.341Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect.
