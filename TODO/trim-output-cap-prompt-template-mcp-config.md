# Trim OutputCap, prompt-template, and MCP config plumbing

## Goal
Verified ponytail audit of three bounded CodingAgent areas. Estimated net reduction: ~350 lines, no dependency changes.

Scope:

### OutputCap
- Remove production-unused/test-only `OutputCap::process()` and `OutputCap::config()`.
- Remove `capForPath()` and the processor's duplicate pre-check; call `capIfNeeded()` once.
- Centralize duplicated read-specific notice construction and offset extraction currently present in both `OutputCapToolResultProcessor` and `OutputCapLlmTransformHook`.
- Consolidate `OutputCapPathResolver` into the existing OutputCap implementation if this produces the smallest behavior-preserving diff.
- Retain both `OutputCapToolResultProcessor` (primary enforcement) and `OutputCapLlmTransformHook` (LLM-bound defense in depth).
- Reduce implementation-mirroring tests while retaining behavioral proof for capping, persistence, cleanup, path classification, detail sanitization, notification delivery, and primary/late-stage agreement.

### MCP Config + Catalog
- Fold the single-caller `McpConfigValidator` and `McpEnvInterpolator` into `McpConfigLoader` and/or `McpServerDefinitionDTO`, establishing one validation owner.
- Preserve all JSON trust-boundary checks, transport/inheritance semantics, environment interpolation, secret-safe errors, and exact merged configuration behavior.
- Inline the single-production-consumer `McpToolNameMapper` mapping/sanitization logic into `McpInitializeSessionHandler`; remove test-only `reverseKey()` and implementation-mirroring mapper tests.
- Remove forwarding `McpConfigDTO::fromServers()` and construct the DTO directly.
- Keep `McpToolCatalogStoreInterface`, catalog DTO serialization boundaries, and `SessionFileMcpToolCatalogStore`: they are justified by cross-process persistence and test seams.

### Prompt templates
- Replace `PromptTemplateArgumentParser::splitChars()` with installed/native `mb_str_split()` support and inline its one-line whitespace predicate.
- Keep the remaining PromptTemplate class boundaries; their parser, substitution, loading, diagnostics, and runtime contracts are independently meaningful.

Explicit exclusions:
- Do not modify MCP Client, transport, cancellation, or deadline behavior; `add-proper-mcp-tool-call-cancellation` owns that work.
- Do not modify Subagent or Fork production/test files or alter their cap classification.
- Do not alter user-visible output-cap notices, limits, persistence paths, retention, MCP configuration semantics, catalog file format, or prompt-template grammar.
- Add no APIs, abstractions, settings, compatibility paths, or dependencies.

## Acceptance criteria
- Production behavior and model-facing OutputCap notices remain byte-for-byte equivalent for covered scenarios; primary enforcement and late defense-in-depth remain active.
- OutputCap test-only APIs and duplicate cap/path/notice logic listed in scope are removed without weakening persistence, sanitization, notification, or path-classification contracts.
- MCP configuration has one validation/hydration flow while preserving unknown-field rejection, transport rules, inherited disable semantics, typed fields, interpolation, and secret redaction.
- `McpToolNameMapper`, `McpConfigValidator`, and `McpEnvInterpolator` are removed when their behavior has been folded into existing owners; no replacement abstraction is introduced.
- Prompt-template argument parsing preserves its existing quoting, whitespace, Unicode, and malformed-quote behavior using `mb_str_split()`.
- No MCP Client, Subagent, or Fork file is modified.
- Focused Castor tests for OutputCap, prompt templates, MCP config/catalog, and MCP initialization pass; `castor deptrac`, `castor phpstan`, and `castor cs-check` pass.
- Because OutputCap and MCP tool exposure affect LLM-visible/runtime flow, deterministic `castor check` passes before CODE-REVIEW.
- Final diff is behavior-preserving and removes approximately 300 lines without dependency changes.

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
- Created: 2026-08-06T19:55:50.003Z
