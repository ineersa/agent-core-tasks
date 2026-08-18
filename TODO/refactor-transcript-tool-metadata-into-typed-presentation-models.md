# Refactor transcript tool metadata into typed presentation models

## Goal
Follow-up from PR #387 review. Investigate replacing manual array shape checks in TUI transcript formatting with typed presentation/projection DTOs. Do not reuse live execution argument DTOs: transcript inputs are persisted/replayed metadata, may be historical or malformed, and include dynamic MCP/extension tools. Respect depfile.yaml TuiTranscript boundaries and preserve graceful fallback rendering.

## Acceptance criteria
- Typed presentation boundaries replace avoidable manual array walking for supported built-in transcript cards.
- Historical, malformed, MCP, and extension tool metadata still degrade safely without breaking transcript rendering.
- No TUI dependency on CodingAgent AppTool execution DTOs unless depfile.yaml architecture is deliberately redesigned and approved.
- Focused virtual transcript tests and castor deptrac pass.

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
- Created: 2026-08-17T02:01:49.727Z
