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
Status: TODO
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-07-31T21:08:48.115Z
