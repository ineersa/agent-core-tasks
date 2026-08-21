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
Status: ARCHIVE
Branch: task/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/410
PR Status: merged
Started: 2026-08-18T18:56:48.341Z
Completed: 2026-08-18T20:28:51.098Z

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

## Task workflow update - 2026-08-18T19:20:58.073Z
- Validation: castor test --filter=Mcp — 176 tests, 660 assertions OK; castor test — 4635 tests, 18443 assertions OK; castor deptrac — 0 violations; castor phpstan — 0 errors; castor cs-fix + cs-check — clean; castor test:controller-replay — 12 tests, 165 assertions OK; castor docs:validate — ok; castor clean:cleanup:workers:list — no stale workers; No tmux E2E (justified: local slash + file-poll + runtime dispatch; lowest layers are virtual + unit + controller-replay)
- Summary: Implementation complete via fork. Commit f64830d19 on task/2026-08-18-add-mcp-command-* (25 files, +1082/−11). /mcp lists servers from mcp.json + session catalog file via new McpStatusSnapshotProvider (lives in Tui/CompactHeader because Deptrac forbids TuiListener→AppMcp — CompactHeader is the approved seam). /mcp reconnect: guard (active run / draft) → client->refreshMcpCatalog → new mcp_refresh protocol verb → controller McpRefreshHandler → dispatches previously-dormant McpRefreshCatalogCommand → onRefreshCatalog now honestly reconnects (disconnectAll first — discover() only dropped per-server stale clients, removed servers would linger; generation was hardcoded 1, now bumped on refresh so TUI 250ms/10s poll can observe completion). Fork authority decisions (all mechanics, mapped to task): generation param on McpToolCatalogBuilder, dispatchRefresh() on McpSessionLifecycleDispatcher, snapshot view DTOs in CompactHeader. Known gap: no live-STDIO/HTTP MCP server E2E (unit/controller-replay lanes only).

## Task workflow update - 2026-08-18T19:46:45.319Z
- Updated PR Status: approved-with-preapproved-nit
- Validation: Fix commit af4791d9a: castor test --filter=Mcp 181 OK/690, castor test 4640 OK/18473, deptrac 0, phpstan 0, cs clean, test:controller-replay 12 OK/165; Cleanup commit 05ecb0b5a: McpCommandHandlerTest|UsageCommandHandlerTest 7 OK/42, phpstan 0, cs-check clean, deptrac 0
- Summary: Review loop complete. Reviewer pass 1 (f64830d19): REQUEST CHANGES — BLOCKER: reconnect poll exited on first partial catalog write (onRefreshCatalog writes per-server), showing mid-discovery state as final; BUG: failed refresh clobbered good catalog → silent "not initialized"; NITs: generation regression on initialize-resume (hardcoded 1 vs refresh prev+1), in-process docblock missing, sanitizeError duplicate; spec-fidelity PASS. Fix commit af4791d9a (7 files, +289/−22): poll caveat when any not_initialized server, invalidated-catalog warning (gen>0 + all not_initialized), monotonic generation on initialize/refresh incl. failure paths, in-process docblock, shared ThrowableMessage::sanitize in src/Tui/Utility (Deptrac-clean TuiListener→TuiUtility edge), 5 contract-pinning tests. Reviewer pass 2: all findings RESOLVED; residual edge (all-pending mid-discovery briefly shows invalidated warning) judged acceptable/truthful — do not tighten; only remaining item = 4-line pre-approved wrapper deletion. Cleanup commit 05ecb0b5a (2 files, +3/−13): inlined ThrowableMessage::sanitize, deleted forwarders. Re-review skipped per pre-approval (SETTINGS-06 precedent). Reviewer also flagged PRE-EXISTING out-of-scope: McpToolCatalogBuilder::computeConfigHash silent catch (~L125, present in base) violates no-silent-degradation — candidate for separate follow-up, not this task.

## Task workflow update - 2026-08-18T19:49:29.548Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (156.5s).
- Pushed task/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect to origin.
- branch 'task/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect' set up to track 'origin/task/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect'.
- Created PR: https://github.com/ineersa/agent-core/pull/410

## Task workflow update - 2026-08-18T19:54:05.639Z
- Moved CODE-REVIEW → IN-PROGRESS.

## Task workflow update - 2026-08-18T20:10:27.031Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (178.4s).
- Pushed task/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect to origin.
- branch 'task/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect' set up to track 'origin/task/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect'.
- PR already exists: https://github.com/ineersa/agent-core/pull/410

## Task workflow update - 2026-08-18T20:10:35.057Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/410
- Updated PR Status: open
- Validation: e8b2288ee: castor test --filter=Mcp 182 OK/691, castor test 4641 OK/18474, phpstan 0, deptrac 0, cs-check clean; Re-gate: castor check passed 178.4s
- Summary: Post-review scope addition (user-directed): pre-existing silent catch McpToolCatalogBuilder::computeConfigHash removed via commit e8b2288ee (2 files) — catch deleted (handler boundary already logs one structured warning + writes empty catalog, so no double-log), return type tightened ?string→string, one JsonException propagation test. castor check 178.4s green on re-gate; branch pushed; PR #410 updated to 4 commits total (f64830d19, af4791d9a, 05ecb0b5a, e8b2288ee). Follow-up task created separately: TODO/cleanup-silent-catch-degradation (10 audit findings).

## Task workflow update - 2026-08-18T20:28:51.098Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect.
- Merged task/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect into integration checkout.
- Merge made by the 'ort' strategy.
 docs/mcp.md                                        |   7 +-
 .../Mcp/Catalog/McpToolCatalogBuilder.php          |  88 ++---
 .../Mcp/Handler/McpInitializeSessionHandler.php    |  66 +++-
 .../Mcp/McpSessionLifecycleDispatcher.php          |  17 +
 .../Runtime/Contract/AgentSessionClient.php        |   8 +
 .../CommandHandler/McpRefreshHandler.php           |  58 ++++
 .../InProcess/InProcessAgentSessionClient.php      |  18 +
 .../Process/JsonlProcessAgentSessionClient.php     |  18 +
 src/Tui/CompactHeader/McpServerStatusView.php      |  26 ++
 src/Tui/CompactHeader/McpStatusSnapshot.php        |  22 ++
 .../CompactHeader/McpStatusSnapshotProvider.php    |  91 ++++++
 src/Tui/Listener/McpCommandHandler.php             | 214 ++++++++++++
 src/Tui/Listener/McpCommandRegistrar.php           |  52 +++
 src/Tui/Listener/UsageCommandHandler.php           |  15 +-
 src/Tui/Utility/ThrowableMessage.php               |  23 ++
 .../Mcp/Catalog/McpToolCatalogBuilderTest.php      |  14 +
 .../Handler/McpInitializeSessionHandlerTest.php    | 117 +++++++
 .../BackgroundProcessCompletionPollerTest.php      |   4 +
 .../CommandHandler/AnswerHumanHandlerTest.php      |   4 +
 .../CommandHandler/CompactHandlerTest.php          |   4 +
 .../CommandHandler/McpRefreshHandlerTest.php       |  88 +++++
 .../CommandHandler/ResumeHandlerTest.php           |   4 +
 .../Runtime/Controller/RuntimeEventEmitterTest.php |   4 +
 tests/Tui/Listener/CompactCommandHandlerTest.php   |   4 +
 tests/Tui/Listener/McpCommandHandlerTest.php       | 361 +++++++++++++++++++++
 .../Listener/TickPollListenerSubagentLiveTest.php  |   4 +
 .../SubagentLivePickerObservationLifecycleTest.php |   4 +
 tests/Tui/Screen/TuiMcpCommandVirtualTest.php      | 137 ++++++++
 tests/Tui/Support/RecordingAgentSessionClient.php  |   4 +
 29 files changed, 1408 insertions(+), 68 deletions(-)
 create mode 100644 src/CodingAgent/Runtime/Controller/CommandHandler/McpRefreshHandler.php
 create mode 100644 src/Tui/CompactHeader/McpServerStatusView.php
 create mode 100644 src/Tui/CompactHeader/McpStatusSnapshot.php
 create mode 100644 src/Tui/CompactHeader/McpStatusSnapshotProvider.php
 create mode 100644 src/Tui/Listener/McpCommandHandler.php
 create mode 100644 src/Tui/Listener/McpCommandRegistrar.php
 create mode 100644 src/Tui/Utility/ThrowableMessage.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/CommandHandler/McpRefreshHandlerTest.php
 create mode 100644 tests/Tui/Listener/McpCommandHandlerTest.php
 create mode 100644 tests/Tui/Screen/TuiMcpCommandVirtualTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-add-mcp-command-list-mcp-servers-status-tools-mcp-reconnect-reconnect.
- Pulled integration checkout: Already up to date..

## Task workflow update - 2026-08-18T20:29:08.884Z
- Summary: Merged. User live smoke test passed (real MCP server: list + reconnect). PR #410 merged to main; merged into integration checkout (ort, 29 files +1408/−68); worktree cleaned. Final surface: /mcp list, /mcp reconnect (mcp_refresh verb → McpRefreshCatalogCommand, previously dormant), McpStatusSnapshotProvider (CompactHeader seam), monotonic catalog generation, computeConfigHash silent-catch fix. Follow-up audit task TODO/cleanup-silent-catch-degradation still pending.

## Task workflow update - 2026-08-19T18:16:16.896Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
