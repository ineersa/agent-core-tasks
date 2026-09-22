# Replace unsafe Unicode text operations with UTF-8-safe string handling

## Goal
Sessions 18 and 20 were blocked when byte truncation split valid UTF-8 text in edit error previews. The malformed output prevented JSON serialization of a tool result and left an unmatched tool call. Audit all first-party code and shipped extensions for unsafe text operations and replace them, preferably with Symfony String and u(). Keep intentional binary, ASCII protocol, byte-offset, and byte-budget operations byte-based where required. Preserve terminal display-width contracts. Do not run a reproduction that emits malformed UTF-8 through agent tools. Existing dirty regression tests in the integration checkout belong to another session and must remain untouched. User requested scout discovery; two launch attempts failed without returning an artifact ID.

## Acceptance criteria
- Record the complete audit inventory, replacements, and justified byte-oriented exclusions.
- Replace unsafe operations on Unicode text using existing Symfony String facilities where appropriate, preserving valid UTF-8 and existing limits or width semantics.
- Cover the edit error preview regression and other distinct affected contracts at the lowest correct layer. Keep diagnostic output valid UTF-8, including failing test output.
- Do not alter binary/protocol behavior, add speculative APIs or settings, or modify active session logs as part of this task.
- Run focused Castor validation in the task worktree and record results.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin
Fork run: 559ee962-aa1a-57c7-aa96-3535e2ff0974
PR URL: https://github.com/ineersa/agent-core/pull/481
PR Status: merged
Started: 2026-09-08T13:41:56+00:00
Completed: 2026-09-08T15:11:28+00:00

## Work log
- Created: 2026-09-08T13:27:27+00:00

## Task workflow update - 2026-09-08T13:35:04+00:00
- Scout completed after user installed Composer dependencies. Artifact agent_fd6c6ede8b29b065; scope=read-only first-party Unicode safety audit at integration checkout. Found unsafe byte truncation in MCP errors, subagent handoffs/progress, artifact summaries, process/clipboard diagnostics, TUI labels, and task-workflow QA snippets. Existing mb_* uses are already safe and not automatically replacement targets. Preserve stream/parser byte offsets, binary lengths, ASCII/hash operations and terminal-cell contracts.

## Task workflow update - 2026-09-08T13:35:44+00:00
- Summary: Implementation blocked before worktree creation. move_task(IN-PROGRESS) refuses because the integration checkout has another session's six uncommitted files: ToolCallResultHandler, EditPatchApplicator, EditPatchParser and their three regression test files. Left those changes untouched. Scout audit completed successfully.
- Confirmed unsafe text caps: Tui/Transcript/SubagentProgressCardWidget.php:442-453; Runtime/Projection/SubagentProgressDisplayFormatter.php:253-260; Agent/Execution/Subagent/ChildRun/Result/SubagentChildRunHandoffRenderer.php:340-348; Mcp/Tool/McpResultMapper.php:99-117; Mcp/Client/McpConnectionManager.php:231-247; Runtime/Controller/BackgroundProcessCompletionPoller.php:226-230; Agent/Artifact/AgentArtifactRegistry.php:713-715; Tui/Picker/SubagentLivePickerController.php:138-144; task-workflow/src/Tool/MoveTaskHandler.php:345-368; Tui/ImagePaste/ClipboardImageReader.php:425-430; Tui/Utility/ThrowableMessage.php:18-21. Paths without src/CodingAgent prefix are under src/CodingAgent except Tui under src and task-workflow under .hatfield/extensions. Verify exact current source and contracts during routing; scout inventory also lists safe mb_* operations that do not need blanket conversion.
- Known edit-preview and recursive tool-result normalization fixes are actively being implemented by another session. Refresh baseline after those changes are committed; avoid duplicating or overwriting that work.

## Task workflow update - 2026-09-08T13:41:56+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin.

## Task workflow update - 2026-09-08T13:43:07+00:00
- Ownership: owner=fork; fork_run=none; revision=3a1cbabdd; scope=remaining arbitrary-text truncation in production source and shipped extensions plus focused regression proof and verified audit exclusions; outcome=assigned; commit=none
- Main routing reviewed MCP mapper and invoker/test references, subagent progress formatting/card truncation, picker labels, workflow QA snippets, stderr byte tails, local Runtime/TUI instructions, depfile boundaries and committed edit-preview regression. Fork warranted by mechanical cross-module replacements and independently testable regression cases. Do not convert already-safe mb_* operations merely for consistency, broaden slug semantics, add a new generic abstraction or introduce invalid-byte containment unrelated to truncation. Preserve byte budgets and existing terminal-width facilities.

## Task workflow update - 2026-09-08T13:58:13+00:00
- Recorded fork run: 559ee962-aa1a-57c7-aa96-3535e2ff0974
- Ownership: owner=fork; fork_run=559ee962-aa1a-57c7-aa96-3535e2ff0974; revision=3a1cbabdd; scope=remaining arbitrary-text truncation and focused regressions; outcome=completed; commit=5c2e1ce9d045a4a388859727e6c4735f3050bad5
- Ownership: owner=main; fork_run=none; revision=5c2e1ce9d045a4a388859727e6c4735f3050bad5; scope=parent verification and minimal corrections to ASCII limits and incremental byte-tail retention; outcome=assigned; commit=none

## Task workflow update - 2026-09-08T14:06:13+00:00
- Validation: Fork read and followed testing skill and tests/AGENTS.md; targeted suite 138 tests/691 assertions and post-format subset 80 tests/492 assertions passed.; Parent focused Castor suite across changed and adjacent contracts: 172 tests, 1046 assertions passed. Includes virtual widget proof, MCP, handoff, picker, CLI diagnostics and prior edit regressions.; Final parent Castor subset after correction: 98 tests, 514 assertions passed, including CompletionListener, ConsumerSupervisor, FileMentionIndexBuilder, EventLogMaxSeqBootstrapReader, ClipboardImageReader and SubagentLivePickerController.; castor phpstan --path=src/CodingAgent and --path=src/Tui: zero errors. castor deptrac: zero violations/errors. castor cs-fix --path=src: no changes. Fork also formatted tests and task-workflow source.; All test console logs captured to worktree-local files and displayed as ASCII-escaped text. No malformed UTF-8 emitted through tools. No castor check run in task-start.
- Summary: Implemented and committed in isolated task worktree: 5c2e1ce9d plus parent correction 8a68754ee. UTF-8-safe text caps now use Symfony String u() across MCP diagnostics, subagent progress/handoffs, TUI summaries/clipboard diagnostics, background command previews and task-workflow output. Diagnostic byte budgets remain byte budgets with safe boundaries. Existing edit-preview regressions retained. Removed CompletionListener's byte-slicing fallback. Clean worktree. Implementation phase complete; independent review and full CODE-REVIEW gate remain for task-to-pr.
- Ownership: owner=main; fork_run=none; revision=5c2e1ce9d045a4a388859727e6c4735f3050bad5; scope=parent verification and minimal corrections to ASCII limits and incremental byte-tail retention; outcome=completed; commit=8a68754ee
- Audit disposition: changed SubagentProgressCardWidget, SubagentProgressDisplayFormatter, SubagentChildRunHandoffRenderer, McpResultMapper, McpConnectionManager, BackgroundProcessCompletionPoller, SubagentLivePickerController task summary, ClipboardImageReader, task-workflow MoveTaskHandler/JetBrainsMcpClient; safe byte caps in FileMentionIndexBuilder, EventLogMaxSeqBootstrapReader, SessionCacheInspectCommand, ConsumerSupervisor. Parent ensured stderr trimming retains newest bytes and byte-budget appenders cut the combined buffer, not an isolated process chunk. Numeric/hash run ID preview remains unchanged.
- Justified exclusions: existing mb_* truncation including edit previews, artifact summaries, ThrowableMessage, provider/worker/log caps and OM chunking; JSONL framing/write offsets, parser/regex offsets, image/blob/base64 sizes, ASCII protocol/env/hash/ID/slug operations, completion byte ranges and terminal ANSI marker offsets. Existing terminal-cell layout facilities unchanged.
- Scout follow-up artifact agent_fd6c6ede8b29b065 audited remaining first-party src, shipped extension src, bin and scripts. Parent rejected blanket /u changes for ASCII whitespace/delimiter grammars: those do not split valid UTF-8 and broadening whitespace semantics is not required. Removed one remaining byte fallback in CompletionListener. No new sanitization fallback, API, setting, slug semantics or width redesign. Full audit artifact remains at worktree .hatfield/tmp/utf8-safety-audit.md; this work log is the durable updated disposition.

## Task workflow update - 2026-09-08T14:27:02+00:00
- Reviewer artifact agent_9f64ab313b02928b reviewed revision 8a68754ee, whole task diff plus exclusion scan and specification fidelity. REQUEST CHANGES: ProcessLifecycle::readLogTail uses tail -c and returns a leading partial UTF-8 code point to bg_status and completion notifications. Fix boundary and add regression. Existing diff otherwise approved. Optional suggestion to replace slice+append with truncate rejected: those preserve distinct legacy N+ellipsis limits, unlike truncate's ellipsis-inclusive cap.
- Ownership: owner=main; fork_run=none; revision=8a68754ee; scope=ProcessLifecycle truncated log-tail UTF-8 boundary and regression; outcome=assigned; commit=none

## Task workflow update - 2026-09-08T14:30:00+00:00
- Validation: castor test --filter='ProcessLifecycleTest|BackgroundProcessCompletionPollerTest|BgStatusToolTest': PASS 26 tests, 100 assertions.; castor phpstan --path=src/CodingAgent/Tool/BackgroundProcess/ProcessLifecycle.php: zero errors.; Earlier focused implementation validation reused for unchanged code. Full deterministic castor check reserved for transition.
- Summary: Independent reviewer APPROVE at 2df450628 after fixing the remaining background-log tail cut. Ready for CODE-REVIEW transition gate; no unresolved blockers.
- Reviewer agent_9f64ab313b02928b re-review: APPROVE; target=2df450628; scope=required ProcessLifecycle fix plus original whole-task specification review. Read/followed testing skill and tests/AGENTS; no remaining required fixes.
- Ownership: owner=main; fork_run=none; revision=8a68754ee; scope=ProcessLifecycle truncated log-tail UTF-8 boundary and regression; outcome=completed; commit=2df450628
- Audit addition: ProcessLifecycle::readLogTail tail -c output now skips leading UTF-8 continuation bytes while preserving newest output and byte ceiling. Covers bg_status and background completion notifications. Existing raw-stream invalid-byte sanitation remains outside task scope.

## Task workflow update - 2026-09-08T14:32:22+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (122.5s).
- Pushed task/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin to origin.
- branch 'task/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin' set up to track 'origin/task/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin'.
- Created PR: https://github.com/ineersa/agent-core/pull/481

## Task workflow update - 2026-09-08T15:11:28+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin: ide_close_project returned isError.
- Merged task/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/extensions/task-workflow/src/Ide/JetBrainsMcpClient.php                                  |  9 +++++----
 .hatfield/extensions/task-workflow/src/Tool/MoveTaskHandler.php                                    |  4 +++-
 src/CodingAgent/Agent/Execution/Subagent/ChildRun/Result/SubagentChildRunHandoffRenderer.php       |  5 +----
 src/CodingAgent/CLI/FileMentionIndexBuilder.php                                                    |  9 ++-------
 src/CodingAgent/CLI/Session/SessionCacheInspectCommand.php                                         |  2 +-
 src/CodingAgent/Mcp/Client/McpConnectionManager.php                                                |  6 +++---
 src/CodingAgent/Mcp/Tool/McpResultMapper.php                                                       |  6 +++---
 src/CodingAgent/Runtime/Controller/BackgroundProcessCompletionPoller.php                           |  6 +++---
 src/CodingAgent/Runtime/Controller/ConsumerSupervisor.php                                          |  8 +++++++-
 src/CodingAgent/Runtime/Projection/SubagentProgressDisplayFormatter.php                            |  8 +++-----
 src/CodingAgent/Session/EventLogMaxSeqBootstrapReader.php                                          | 13 ++++---------
 src/CodingAgent/Tool/BackgroundProcess/ProcessLifecycle.php                                        |  9 +++++++++
 src/Tui/ImagePaste/ClipboardImageReader.php                                                        |  8 ++++----
 src/Tui/Listener/CompletionListener.php                                                            | 11 +++--------
 src/Tui/Picker/SubagentLivePickerController.php                                                    |  6 +++---
 src/Tui/Transcript/SubagentProgressCardWidget.php                                                  |  7 +++----
 tests/CodingAgent/Agent/Execution/Subagent/ChildRun/Result/SubagentChildRunHandoffRendererTest.php | 33 +++++++++++++++++++++++++++++++++
 tests/CodingAgent/Mcp/Client/McpConnectionManagerTest.php                                          |  7 +++++++
 tests/CodingAgent/Mcp/Tool/McpResultMapperTest.php                                                 | 21 +++++++++++++++++++++
 tests/CodingAgent/Runtime/Controller/ConsumerSupervisorTest.php                                    | 31 +++++++++++++++++++++++++++++++
 tests/CodingAgent/Runtime/Projection/SubagentProgressProjectionTest.php                            | 26 ++++++++++++++++++++++++++
 tests/CodingAgent/Tool/BackgroundProcess/ProcessLifecycleTest.php                                  | 19 +++++++++++++++++++
 tests/Tui/Picker/SubagentLivePickerControllerTest.php                                              | 23 +++++++++++++++++++++++
 tests/Tui/Transcript/SubagentResultRendererTest.php                                                | 28 ++++++++++++++++++++++++++++
 24 files changed, 245 insertions(+), 60 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-08-replace-unsafe-unicode-text-operations-with-utf-8-safe-string-handlin.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR #481 merged at 2026-09-08T15:10:28Z, merge commit 5b6453dcdf9b843436efce5c95a162311d914b89. Updating integration checkout and completing task.

## Task workflow update - 2026-09-08T15:12:47+00:00
- Updated PR Status: merged
- Validation: Integration castor check: quality ok (126.0s), QA run qa-20260908-151137-10843-9b5a47cb. All 10 lanes passed, including 4875 unit/integration tests, controller replay, TUI and llm-real. Artifact integrity, exact-run leak check and cache guard passed.; Tool wrapper reported unclean exit, but the durable QA log .hatfield/tmp/utf8-postmerge-check.log confirms completed quality: ok; all reports under var/reports/qa-20260908-151137-10843-9b5a47cb.; git status --porcelain: empty. Task branch worktree absent from git worktree list. Residual former directory contains only .idea after IDE close degradation.
- Summary: Post-merge validation passed on integration revision 59b8565ca. Git status is clean; task no longer registered in git worktree list. Git worktree removed, but IDE close reported degradation and recreated/left an .idea-only directory at the former worktree path; no code worktree remains.

## Task workflow update - 2026-09-10T22:49:41+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
