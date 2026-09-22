# Make tool execution status and partial workflow failures reliable

## Goal
## Goal
Make tool results accurately report whether execution is still running, completed, failed, or interrupted. Report completed workflow steps and remaining work when an operation partially succeeds, so agents do not repeat side effects or claim failure prematurely.

## Observed failures
During task `2026-08-27-show-live-tool-call-duration-in-tui`, PR #480:

1. `bash` returned `Command failed with unclean exit` for integration `castor check`, but the QA process continued. Inspection found PID 21632 still running. Its final command log reported `quality: ok (121.2s)`, with all lanes, artifact integrity, leak check, and cache guard passing. The agent had to inspect processes and files to recover the actual outcome.
   - QA reports: `/home/ineersa/projects/agent-core/var/reports/qa-20260908-153049-21632-8a808de8`
   - Command log: `/home/ineersa/projects/agent-core/.hatfield/tmp/bg/1d1b9513a08ce0c1.log`
   - A user-requested repeat completed with normal tool output: `quality: ok (122.1s)`, QA run `qa-20260908-153336-26081-e05e74dc`.
2. `move_task(to="CODE-REVIEW")` returned only `An error occurred while executing tool "move_task".` QA had passed and the branch had been pushed, but PR creation failed authentication. The task remained IN-PROGRESS. The underlying exception in `agent-2026-09-08.log` named the GitHub authentication error and completed push, but the tool response omitted both.
   - QA reports: `/home/ineersa/projects/agent-core-worktrees/2026-08-27-show-live-tool-call-duration-in-tui/var/reports/qa-20260908-140221-7178-5166fb50`, now potentially unavailable because task worktree cleanup removed it.
   - Exception originated in `.hatfield/extensions/task-workflow/src/Tool/MoveTaskHandler.php:308` at the observed revision.
   - Recovery used authenticated shell `gh pr create`, task metadata update, then `move_task(pushOnly=true)`, which repeated QA.

The user clarified that GitHub authentication failed because the desktop keyring had not been unlocked after startup. Do not treat credential storage or authentication unification as the bug. The bug is failure reporting that hides the cause and partial success.

## Scope and design questions
Investigate command supervision/result delivery and extension-tool exception reporting. Determine whether these have a shared cause or need separate fixes under this task.

Provide a reliable operation reference with discoverable status and logs. Reuse existing tool-call IDs, process records, artifacts, and task workflow facilities where possible. Example of useful workflow output: `QA passed. Push succeeded. PR creation failed authentication. Task remains IN-PROGRESS.` Include what can safely happen next.

At task-explain, decide the smallest supported recovery design. Do not assume a new generic operation database, public resume API, automatic retries, or permission to skip QA. If resuming from a failed step requires a new API or changed validation policy, get explicit approval first.

Keep existing feature workflow and independent-review requirements. Simplifying repository procedure and fixing keyring behavior are out of scope.

## Acceptance criteria
- Reproduce and identify why bash can report an unclean exit while the owned command continues; distinguish command failure from supervision or result-delivery failure.
- Tool output does not report a command as terminally failed when it is known to remain active. If final status is unknown, state that explicitly and provide an existing or approved operation reference for status and logs.
- Partial move_task failure reports completed steps, failed step and bounded cause, current task status, and available QA/log references. A generic error alone is insufficient.
- Operation identity and final outcome remain discoverable after interrupted result delivery, using existing facilities where possible; agents can determine what happened without scanning arbitrary runtime logs or process tables.
- Recovery guidance accounts for completed side effects and does not recommend blindly repeating the whole operation. Any step-resume implementation preserves clean-revision and mandatory QA safeguards and requires finalized scope.
- Errors and operation records redact credentials and sensitive command content; preserve process ownership and never signal active-session or root-owned workers.
- Add deterministic regression proof at the lowest correct layer for continued execution after delivery/supervision failure, genuine failure, successful completion, and workflow failure after successful earlier steps. Use controlled process barriers, not timing windows or sleeps.
- Run focused Castor validation and the required full transition gate. Document the resulting status/error/recovery contract.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/483
PR Status: merged
Started: 2026-09-08T17:19:25+00:00
Completed: 2026-09-08T21:14:54+00:00

## Work log
- Created: 2026-09-08T15:57:53+00:00

## Task workflow update - 2026-09-08T16:01:13+00:00
- Summary: Additional user-reported failure: subagents could not start because a required extension was not installed, but the visible result was only a generic tool-call failure. The user had to inspect logs to discover the cause. Include subagent startup failures in error-visibility scope; exact message and reproduction remain to be verified.
- Acceptance addition: when subagent startup fails because a required extension is missing, report the missing extension and actionable cause in the parent-visible tool result, not only runtime logs. Preserve child/run correlation and available diagnostic references without exposing sensitive data.
- Validation addition: add deterministic regression proof for missing-extension subagent startup failure and propagation to the parent-visible result. Inspect other startup failures along the same propagation path; do not broaden this into extension installation or automatic recovery behavior.

## Task workflow update - 2026-09-08T16:01:49+00:00
- Summary: User clarified that actionable error visibility applies to tool calls generally, not only subagent startup or move_task.
- Acceptance addition: ordinary tool-call failures must propagate a bounded, actionable cause to the model-visible result and user-visible TUI. Generic failure labels must not replace available error details. Include diagnostic references when details cannot safely fit inline, and redact secrets.
- Scope clarification: inspect the shared tool-error propagation path, including built-in and extension-provided tools. Use representative deterministic regression cases at shared boundaries rather than duplicating tests for every tool. Preserve existing failure semantics; do not add automatic retries or recovery behavior.

## Task workflow update - 2026-09-08T17:19:25+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable.

## Task workflow update - 2026-09-08T17:21:06+00:00
- Ownership: owner=fork; fork_run=none; revision=2be36dbf5; scope=bash/background-process lifecycle false terminal detection, existing status/log discoverability, deterministic focused proof; outcome=assigned; commit=none
- Routing: main inspected BashTool, BackgroundProcessManager.resolveEntityStatus, ProcessLifecycle.launchProcess/isAlive, RegistryBackedToolbox exception translation, ToolExecutor catch paths, MoveTaskHandler CODE-REVIEW phases and existing tests. Independent process slice delegated sequentially; generic error propagation and workflow partial reporting remain pending main ownership after handoff. No automatic retry, operation DB or resume API planned.

## Task workflow update - 2026-09-08T17:40:30+00:00
- Summary: Process fork reproduced transient setsid launcher PID mismatch and committed launch fix. Main review found unrequested legacy PID-rebinding fallback and foreground bg_status guidance needing correction after shared-exception slice handoff.
- Ownership: owner=fork; fork_run=agent_4b9e87256c3298d8; revision=2be36dbf5; scope=bash/background-process lifecycle; outcome=completed; commit=feb7f3ce570ed28420f37d1c7f5b9a4df54469df
- Ownership: owner=fork; fork_run=none; revision=feb7f3ce570; scope=shared safe actionable tool-exception propagation including missing-extension child startup, model-visible and TUI proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-08T17:54:28+00:00
- Summary: Shared exception slice landed. Main will correct over-broad 500-character truncation of established ToolCallException output and verify missing-extension path after workflow slice. No new public recovery API required: use existing task records, call/result history and bash sidecars.
- Ownership: owner=fork; fork_run=agent_1658e2b04c2db56f; revision=feb7f3ce570; scope=shared safe actionable tool exceptions; outcome=completed; commit=fb394fe7911de4efe916fa1996b924e92af791ed
- Ownership: owner=fork; fork_run=none; revision=fb394fe791; scope=task-workflow partial transition failure reporting and durable existing-board evidence; outcome=assigned; commit=none

## Task workflow update - 2026-09-08T18:04:28+00:00
- Ownership: owner=fork; fork_run=agent_ecb78448f1539926; revision=fb394fe791; scope=task-workflow partial reporting; outcome=completed; commit=c082a148b6fc63130a41bed2670c2be6e31cd88d
- Ownership: owner=main; fork_run=none; revision=c082a148b6; scope=integration corrections, minimality, safe output retention, process ownership and focused proof across slices; outcome=assigned; commit=none

## Task workflow update - 2026-09-08T18:21:49+00:00
- Validation: castor test --filter='ProcessLifecycleTest|BackgroundProcessManagerTest|BashToolTest|ToolExecutorTest|RegistryBackedToolboxTest|IsolatedAgentToolboxTest|MoveTaskHandlerTest|DiagnosticMessageSanitizerTest|DeferredSubagentBatchLaunchTest|TuiTranscriptBlocksVirtualRenderTest|SubagentChildExtensionMetadataTest|McpConnectionManagerTest|McpResultMapperTest' PASS: 242 tests, 1099 assertions; maximum case 1.066781s; none over 10s.; castor test:controller-replay PASS: 9 tests, 135 assertions; maximum case 3.981958s. Final log recovered after old running harness prematurely returned unclean exit.; castor deptrac, phpstan, dead-code, cs-check, docs:validate and git diff --check PASS.; Lowest-layer proof: socket-controlled owned supervisor identity, heredoc/exit status recording, supervisor-loss unknown outcome, parent model-visible failed results, actual missing-extension preparation, virtual TUI live/replay failure rendering, fake-exec QA/push/PR partial failure with durable board evidence.; Testing skill and tests/AGENTS.md read and followed by main and all three implementation forks. Full castor check and independent reviewer intentionally deferred to task-to-pr.
- Summary: task-start implementation complete at a3058bf6d, worktree clean. Fixed setsid launcher/wrapper identity by publishing actual supervisor PID over launch pipe; isolated user bash syntax from status recorder. Removed speculative legacy PID rebinding. Genuine missing status now reports unknown workload outcome with existing log/status/PID/record references. Shared exception translation exposes bounded redacted causes and preserves explicit tool diagnostics. Missing-extension launch uses actual container preparation proof. CODE-REVIEW failures retain completed-step evidence in existing board logs and report cause/status/QA references; recovery instructions explicitly retain mandatory QA on retries. No operation database, public resume API, new setting, or automatic retry added.
- Ownership: owner=main; fork_run=none; revision=c082a148b6; scope=integration corrections, process ownership, output retention and cross-slice proof; outcome=completed; commit=a3058bf6d
- Fork corrections: removed old-row PID fallback and sidecar readiness polling; native launch pipe now supplies wrapper identity. Corrected unconditional 500-character truncation of explicit tool failure output, expanded common credential redaction, and replaced misleading bg_status foreground guidance. Fixed workflow retry guidance that implied QA could be skipped.
- Limitations: redaction covers common credential patterns, not arbitrary secrets in prose. Partial-operation evidence is task/run/path based, not a new resumable operation engine. Unexpected output-delivery loss remains recoverable from existing records/artifacts; no automatic continuation is introduced. Current running harness is the earlier installed revision and continued to show its old unclean-exit bug during commands; final command artifacts were inspected instead of blind retries.

## Task workflow update - 2026-09-08T19:01:29+00:00
- Review: role=reviewer; artifact=agent_7f3a6a823650d483; revision=a3058bf6d; scope=origin/main...HEAD, specification fidelity, process ownership, shared diagnostics, partial workflow evidence and lowest-layer proof; outcome=APPROVE WITH SUGGESTIONS; reported blockers=none. Main is checking two recovery-contract observations before accepting the verdict.
- Reviewer confirmed testing skill and tests/AGENTS.md were read. Reviewer also reported raw php -l, contrary to Castor-only QA instructions; that lint is not accepted as validation evidence. Existing Castor validation remains authoritative.

## Task workflow update - 2026-09-08T19:03:55+00:00
- Validation: Reused focused Castor evidence at unchanged a3058bf6d: 242 tests/1099 assertions, controller replay 9/135, no cases over 10 seconds; deptrac, phpstan, dead-code, cs-check, docs:validate and diff whitespace checks passed.; Full castor check reserved for the CODE-REVIEW transition.
- Summary: Independent review accepted at a3058bf6d: APPROVE WITH SUGGESTIONS, no blockers. Corrected review verified durable PR URL metadata and Castor report-directory behavior; both initial concerns were retracted. Optional catch/docblock cleanup deferred. Unchanged background completion-notification wording remains outside this foreground-result fix; no claim that all background notification wording was changed.
- Review: role=reviewer; artifact=agent_7f3a6a823650d483; revision=a3058bf6d; scope=full task diff plus recovery-contract clarification; outcome=APPROVE WITH SUGGESTIONS; blockers=none. Specification fidelity and lowest-layer proof accepted. No source changes during task-to-pr.

## Task workflow update - 2026-09-08T19:05:07+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (49.5s).
- Pushed task/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable to origin.
- branch 'task/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable' set up to track 'origin/task/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable'.
- Created PR: https://github.com/ineersa/agent-core/pull/483
- Summary: Independent reviewer agent_7f3a6a823650d483 approved a3058bf6d with non-blocking suggestions. Focused validation reused at unchanged revision; transition owns full QA gate.

## Task workflow update - 2026-09-08T21:14:54+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable: ide_close_project returned isError.
- Merged task/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/extensions/task-workflow/skills/task-workflow/references/task-to-pr.md                       |   2 +
 .hatfield/extensions/task-workflow/src/Tool/MoveTaskHandler.php                                        | 197 ++++++++++++++++++++++++++++++++++++++++++++++++++++++-----
 .hatfield/extensions/task-workflow/tests/MoveTaskHandlerTest.php                                       | 131 +++++++++++++++++++++++++++++++++++++++
 docs/background-processes.md                                                                           |   4 ++
 docs/tools.md                                                                                          |  13 ++++
 src/AgentCore/Application/Handler/ToolExecutor.php                                                     |  21 ++++---
 src/AgentCore/Contract/Tool/DiagnosticMessageSanitizer.php                                             |  51 ++++++++++++++++
 src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Launch/DeferredSubagentBatchLaunchService.php  |  15 ++++-
 src/CodingAgent/Extension/Agent/IsolatedAgentToolbox.php                                               |  20 +++++-
 src/CodingAgent/Mcp/Client/McpConnectionManager.php                                                    |  38 +-----------
 src/CodingAgent/Tool/BackgroundProcess/ProcessLifecycle.php                                            |  19 +++---
 src/CodingAgent/Tool/BashTool.php                                                                      |  11 +++-
 src/CodingAgent/Tool/RegistryBackedToolbox.php                                                         |   7 ++-
 tests/AgentCore/Application/Handler/ToolExecutorTest.php                                               |  42 ++++++++++++-
 tests/AgentCore/Contract/Tool/DiagnosticMessageSanitizerTest.php                                       |  49 +++++++++++++++
 tests/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Launch/DeferredSubagentBatchLaunchTest.php   |   2 +-
 tests/CodingAgent/Agent/Execution/Subagent/ChildRun/Preparation/SubagentChildExtensionMetadataTest.php |  31 ++++++++++
 tests/CodingAgent/Extension/Agent/IsolatedAgentToolboxTest.php                                         |   2 +-
 tests/CodingAgent/Tool/BackgroundProcess/ProcessLifecycleTest.php                                      |  65 ++++++++++++++++++++
 tests/CodingAgent/Tool/BashToolTest.php                                                                |  14 ++++-
 tests/CodingAgent/Tool/RegistryBackedToolboxTest.php                                                   |  12 ++--
 tests/Tui/Screen/TuiTranscriptBlocksVirtualRenderTest.php                                              |  21 ++++---
 22 files changed, 679 insertions(+), 88 deletions(-)
 create mode 100644 src/AgentCore/Contract/Tool/DiagnosticMessageSanitizer.php
 create mode 100644 tests/AgentCore/Contract/Tool/DiagnosticMessageSanitizerTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User merged PR #483. GitHub confirms MERGED at 2026-09-08T21:14:00Z, merge commit 0ba98fb2ee263211e429457a8fd9537bfa5d1547. Sanitizer concerns remain in the separate follow-up task; current implementation retained as approved.

## Task workflow update - 2026-09-08T21:16:26+00:00
- Updated PR Status: merged
- Validation: Integration castor check PASS: quality ok in 123.9s. QA reports: var/reports/qa-20260908-211510-429-e7477dfd.; All 10 lanes passed: 4912 unit/integration tests, 20488 assertions; controller replay 9/135; TUI 9/64; llm-real 5/30; deptrac, phpstan, dead-code, cs-check, docs:validate and catalog:version-check passed.; 4935 JUnit cases inspected, maximum 6.908213 seconds, zero cases over 10 seconds.; QA artifact integrity and owned-process/tmux leak checks passed. Llama-proxy cache unchanged at 396 entries.
- Summary: Post-merge validation complete at integration revision f2086af3ab45b8d87d8da92b4ab3d36cc0358d29. Git status clean and task worktree removed. JetBrains close reported degradation during cleanup, but worktree removal succeeded.

## Task workflow update - 2026-09-10T22:49:41+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
