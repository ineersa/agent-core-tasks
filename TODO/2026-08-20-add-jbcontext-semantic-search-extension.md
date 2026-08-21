# Add jbcontext semantic-search extension

## Goal
Build a project-scoped Hatfield extension around the `jbcontext` CLI. The integration must be opt-in by prior manual indexing: Hatfield never creates a first index for an arbitrary directory.

## Finalized behavior

### Eligibility and startup

- The extension is enabled through the existing `extensions.enabled` mechanism; add no new enable setting.
- At interactive Hatfield startup, never block the TUI. Dispatch one background eligibility/refresh job through the existing extension-agent transport.
- Eligibility requires both:
  1. `<project>/.idea` is a directory.
  2. `jbcontext status --project-path <project> --json-output` succeeds and reports at least one existing index snapshot for that exact project.
- If no existing index is reported, do not run `jbcontext index`; mark the integration disabled and tell the user in the TUI status area that manual `jbcontext index` is required. This is the security boundary preventing accidental first-time indexing of home or another directory.
- Transient status failures (missing/unavailable binary, auth/network/daemon failure, malformed response) retry in the background with exponential backoff bounded to approximately 30 seconds total. Use a small fixed schedule; do not add settings for it.
- After startup retries are exhausted, disable search and refresh for the rest of that Hatfield session. Do not retry on later turns. Surface one concise sanitized disabled status in the TUI; log structured diagnostics without source, prompts, tool output, credentials, MCP URLs, or environment values.
- Once eligibility succeeds, run incremental `jbcontext index --project-path <project> --silent` in the background. Official jbcontext documentation states repeated indexing processes only changed content.

### Refresh cadence

- After successful startup eligibility, enqueue an incremental background reindex after each successfully completed assistant turn (`agent_end` with completed reason), using `AfterTurnCommitHookInterface` and the extension-agent job API.
- Do not index after every edit, before every query, on cancelled/failed turns, or with aggressive reminder/enforcement hooks.
- Keep hot hooks cheap and idempotent. Expensive CLI work runs only in the background worker. Coalesce safely using the existing single extension-agent path and deterministic job identity; do not introduce a new scheduler or detached-process subsystem.

### Hatfield tool

- Register a permanent model-visible Hatfield tool named `code_search`; do not configure or wrap a jbcontext MCP client.
- Implement it through the public extension exec capability and `jbcontext search --project-path <project> --json-output`.
- Tool schema matches jbcontext MCP semantics:
  - required string `text`
  - optional string `path_filter`, project-relative
  - no model-visible limit; use a small fixed internal result limit
  - reject absolute/traversing path filters before execution
- Parse the CLI JSON internally, normalize only the useful ranked result fields (relative path, line/range when available, and code content/snippet), and return a top-level TOON string via `helgesverre/toon`, not JSON or a duplicated envelope.
- The tool must use cooperative cancellation and a bounded timeout through `ExecOptionsDTO`/`ToolInvocationContextDTO`.
- When startup eligibility is pending or disabled, return a concise TOON unavailable result; never attempt first-time indexing from a tool call.
- Supply Hatfield-native description, prompt summary, and prompt guidelines: use one focused natural-language semantic query for unfamiliar behavior/location; optionally narrow once with `path_filter`; then read promising files; prefer direct reads/IDE definition/references for known files or symbols; do not use semantic search for builds, tests, Git, or diff review.

### Project-level skill and scout

- Bundle a non-aggressive semantic-search skill derived from JetBrains Context guidance and install it at project scope under `.hatfield/skills/` only after eligibility succeeds.
- Install an updated project-level `.hatfield/agents/scout.md` after eligibility succeeds so it overrides, but never modifies, the user-level scout. The project scout must include the semantic-search skill, allow `code_search`, use it for meaning-based unfamiliar-code discovery, and retain IDE semantic navigation/direct-read guidance for exact symbols and impact analysis.
- No new specialist agent: the existing scout owns multi-step exploration.
- Installed project assets must contain an extension-managed marker. Create/update only absent or previously extension-managed destinations; never overwrite user-owned project scout/skill files. Log a concise collision warning and continue.
- Because eligibility runs asynchronously after startup discovery, newly installed project assets may take effect on the next Hatfield session; document this explicitly.

### Packaging and scope

- Package under `.hatfield/extensions/jbcontext/` using existing `hatfield-extension` conventions and register it in `.hatfield/extensions/composer.json`.
- Use only Extension API contracts from extension product code; do not depend on CodingAgent/AgentCore/TUI internals.
- Reuse the observational-memory extension patterns for after-turn dispatch, deterministic jobs, status-file polling, structured failure handling, and extension packaging.
- Do not add aggressive nudges/hooks, MCP configuration distribution, a new agent registration API, a watch daemon, settings, or a duplicate background-process abstraction.
- Document prerequisites, manual first index, enablement, privacy/security implications, refresh timing, unavailable states, managed project assets, and current-session/next-session behavior.

## Reference material

- JetBrains Context public integration exposes MCP `code_search(text, pathFilter)` and recommends broad semantic search followed by local reads and at most one narrowed retry.
- Official setup installs a background `jbcontext index --silent` session-start hook; indexing is documented as incremental.
- Public repo: https://github.com/JetBrains/context
- Existing patterns: `.hatfield/extensions/observational-memory/`, `.hatfield/extensions/task-workflow/src/Tool/ToolResult.php`, `.hatfield/extensions/extension-api/src/ExtensionApiInterface.php`.

## Acceptance criteria
- Hatfield startup remains responsive while one background job checks `.idea`, verifies a prior jbcontext index, and refreshes only eligible projects.
- The extension never invokes `jbcontext index` for a project without both `.idea` and a pre-existing index reported by `jbcontext status`; ineligible projects show a concise disabled TUI status.
- Transient startup status failures use bounded exponential retries totaling about 30 seconds; exhaustion disables search/reindex for that session with structured sanitized diagnostics and no later-turn retries.
- Eligible sessions run an incremental startup index and enqueue incremental indexes only after successfully completed assistant turns, never per edit/query/cancelled/failed turn.
- Permanent `code_search` exposes only required `text` and optional safe project-relative `path_filter`, executes the jbcontext CLI with cancellation/timeout, and never triggers first indexing.
- Successful and unavailable `code_search` responses are top-level TOON strings; successful results preserve useful ranked path, line/range, and snippet content without a JSON envelope.
- Tool description, prompt summary, and guidelines clearly teach focused semantic discovery, one optional path-narrowed retry, follow-up reads, and when direct/IDE search is preferable without aggressive enforcement.
- After eligibility, marker-managed project-level semantic-search skill and scout are installed/updated; user-owned project files and all user-level files are never overwritten, and no new specialist agent is added.
- Extension package, registration, documentation, structured logging, and async lifecycle follow existing extension conventions without dependencies on Hatfield internals.
- Focused Castor tests cover eligibility guards, no-first-index safety, retries/exhaustion, completed-turn dispatch, TOON output, path validation, managed-file collision behavior, and TUI disabled-status polling; full validation follows the project testing skill and Castor-only rules.

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
- Created: 2026-08-20T23:48:59+00:00
