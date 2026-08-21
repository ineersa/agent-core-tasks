# Add specific Datadog MCP and a datadog-logs specialist agent

## Goal
Add Datadog as a Hatfield MCP server with `availability: specific`, then add a dedicated `datadog-logs` subagent for Datadog observability work.

Ownership / prerequisite:

- User will connect/provision the Datadog MCP and provide the supported transport/auth configuration ("on me"). Do not invent an endpoint or commit credentials.
- Implementation begins once that connection information is available. Secrets must stay in environment variables or the MCP server's supported local authentication mechanism.

Requested behavior:

- Configure the Datadog MCP under a stable server name such as `datadog` with `availability: specific`, so its tools are hidden from the parent and unrelated agents.
- Add a discovered Hatfield agent named `datadog-logs` with a clear catalog description.
- Restrict the agent to the Datadog MCP via exact/prefix `mcp:` selectors and only the minimum non-MCP tools needed. Do not grant all MCP servers or broad implementation tools speculatively.
- The specialist should investigate Datadog logs, traces, metrics, monitors, dashboards, incidents, and related observability data; correlate evidence; and report concise findings with query/time-range context.
- When the connected Datadog MCP exposes write operations, the agent may perform explicitly requested Datadog edits. Its prompt must distinguish read-only investigation from mutating actions and require clear user authorization before creating, editing, muting, or deleting Datadog resources.
- Keep this configuration/agent-definition work minimal; use Hatfield's existing `availability: specific` and agent frontmatter mechanisms rather than adding product code or a custom Datadog integration.

Likely files after the user supplies connection details:

- `.hatfield/mcp.json`
- `.hatfield/agents/datadog-logs.md`
- Existing agent/MCP documentation only if validation shows a project-specific note is necessary.

## Acceptance criteria
- Datadog is configured as an MCP server with `availability: specific`; it is not exposed to the parent or unrelated subagents.
- No Datadog token, API key, application key, session cookie, or other credential is committed.
- A discovered `datadog-logs` agent opts into only the Datadog MCP using valid `mcp:` tool selectors and has the minimum necessary non-MCP tools.
- The agent prompt covers log/trace/metric/monitor/dashboard/incident investigation, evidence correlation, explicit time ranges and queries, and concise source-backed handoffs.
- Read-only investigation is the default; Datadog mutations require an explicit request/authorization and the agent does not silently edit, mute, or delete resources.
- Configuration and agent discovery/allowlisting are validated using existing Hatfield tests or the smallest appropriate Castor validation; no new production abstraction is introduced.

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
- Created: 2026-08-20T17:22:06+00:00
