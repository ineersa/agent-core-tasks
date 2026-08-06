# Verify MCP tools are not always loaded into model context

## Goal
Right now ALL connected MCP server tool schemas are injected into every LLM request regardless of whether the agent actually needs them. This burns ~12K tokens (45% of first-turn context) that the model can never use.

From analysis of session 8:
- jetbrains-index: 18 tools, ~10K tokens - always loaded
- context7: 2 tools, ~1.2K tokens - always loaded
- websearch: 3 tools, ~800 tokens - always loaded

These are injected as `available_tools` in the system prompt + as MCP tool schemas on every turn.

**Goal:** MCP tools should only be loaded when relevant to the session, not always dumped into every LLM request.

Possible approaches:
- Per-session settings to enable/disable MCP servers
- Per-project default + override
- Lazy loading (only inject when tool name is mentioned)

The model should only see tool schemas for servers that are relevant to the current session context.

## Acceptance criteria
- MCP tool schemas are NOT injected into every LLM request by default
- There is a way to opt-in to specific MCP servers per session/project
- Context budget is reduced when MCP servers are disabled

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
- Created: 2026-07-27T13:55:21+00:00

## Task workflow update - 2026-08-04T17:21:11.887Z
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Validation: Behavior manually checked: `availability: specific` MCP tools were absent from parent context.; Planning review confirmed dynamic MCP tools are excluded from the system prompt's permanent `<available_tools>` list. No Castor QA was run because no code changed.
- Summary: Verified that MCP servers configured with `availability: specific` are not loaded into parent LLM tool schemas/context. Existing `ActiveToolSet` filtering and Symfony AI's per-request `tools` allowlist already enforce this behavior. Servers configured as `availability: all` are included by design. No configuration change or per-session selector is wanted, so no implementation remains.
