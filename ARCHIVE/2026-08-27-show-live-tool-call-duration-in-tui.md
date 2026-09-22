# Show live and final tool-call duration in the TUI

## Goal
Show how long each tool call takes directly in the Hatfield TUI.

While a tool call is running, its visible tool-call presentation should show a live elapsed duration. When execution finishes, fails, or is cancelled, replace/freeze that live value with the final elapsed duration for that tool call.

The exact presentation—such as a colored timer, a started-at label plus elapsed value, or placement within the existing tool-call card—must be decided during task-explain using the current transcript/tool presentation architecture. Prefer the smallest extension of existing timestamps and presentation models. Do not add a setting, storage field, or protocol surface unless existing data cannot satisfy the requirement and the user explicitly approves that design change.

## Acceptance criteria
- Each running tool call visibly shows an elapsed duration that updates while execution is active.
- Completed, failed, and cancelled tool calls show a stable final duration.
- Concurrent tool calls retain independent timers and final durations.
- Timing presentation integrates with existing tool-call styling and does not create duplicate transcript entries, uncontrolled scroll growth, flicker, or a high-frequency busy loop.
- Resume/replay behavior is defined from existing canonical timestamps where possible; no new persistence or protocol field is introduced without explicit approval.
- Add deterministic proof at the lowest correct layer for live updates and final frozen duration, using controller replay only if runtime events are part of the contract and minimal tmux only if PTY/process boot is uniquely required.
- Run focused Castor validation for the selected proof layer and `castor check` before CODE-REVIEW because this changes TUI/runtime-visible behavior.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-27-show-live-tool-call-duration-in-tui
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-27-show-live-tool-call-duration-in-tui
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/480
PR Status: merged
Started: 2026-09-08T01:24:48+00:00
Completed: 2026-09-08T15:30:41+00:00

## Work log
- Created: 2026-08-27T17:12:15.597Z

## Task workflow update - 2026-08-31T15:40:53.105Z
- Summary: Expanded scope: completed subagent cards should retain a compact execution summary such as `38 tools · 49k tok · 2m19s` for both single and parallel scout/fork rows, while keeping the existing detailed LLM/model/reasoning/context lines unchanged.
- Absorbed the only remaining presentation follow-up from `2026-08-24-fix-missing-scout-handoff-in-tui`: show final elapsed subagent duration together with tool count and compact aggregate tokens.
- Current code note: `SubagentProgressCardWidget::formatHeaderLine()` renders tool count, compact tokens, and elapsed time only for active/running status, so completed cards omit that compact summary even though final LLM and context details render correctly.
- Data note for task-explain: `SubagentProgressSingleSnapshotDTO` already has `elapsedMs`; `SubagentProgressChildRowDTO` for parallel children currently does not. Prefer deriving final duration from existing canonical child start/terminal timestamps or current projection data. If a parallel-row protocol field is truly required, explain the smallest change and obtain explicit approval before adding it.
- Acceptance addition: completed, failed, and cancelled single/parallel subagent rows freeze and display their final compact execution summary without changing the live presentation, duplicating transcript rows, adding a busy loop, or losing replay parity.

## Task workflow update - 2026-09-08T01:24:48+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-27-show-live-tool-call-duration-in-tui.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-27-show-live-tool-call-duration-in-tui.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-27-show-live-tool-call-duration-in-tui.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-27-show-live-tool-call-duration-in-tui.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-27-show-live-tool-call-duration-in-tui.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-27-show-live-tool-call-duration-in-tui.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-27-show-live-tool-call-duration-in-tui/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-27-show-live-tool-call-duration-in-tui.

## Task workflow update - 2026-09-08T01:29:03+00:00
- Summary: User approved passing canonical timestamps on tool start/end runtime events and derived elapsed_ms on parallel subagent rows. No new settings or database fields. Presentation: elapsed suffix in existing tool headers, final compact subagent summary retained.
- Ownership: owner=fork; fork_run=none; revision=0fd103447; scope=subagent final compact summaries and canonical single/parallel elapsed timing; outcome=assigned; commit=none
- Ownership: owner=main; fork_run=none; revision=0fd103447; scope=ordinary live/final tool timing through runtime projection and existing mounted TUI tick/render path; outcome=assigned; commit=none

## Task workflow update - 2026-09-08T01:53:16+00:00
- Summary: Fork completed and handed ownership back to main. Main uses native Symfony ScheduledTickTrait in tool header widgets, avoiding changes to poller or transcript tick plumbing. User requested main continue directly if any fork fails.
- Ownership: owner=fork; fork_run=agent_3895f1599a93b7ff; revision=0fd103447; scope=subagent final compact summaries and canonical single/parallel elapsed timing; outcome=completed; commit=c4ddc3ab47a6376674f09ecfb972c992d88b5725
- Ownership: owner=main; fork_run=none; revision=c4ddc3ab47a6376674f09ecfb972c992d88b5725; scope=integrated duration implementation, including review and corrections to subagent slice; outcome=assigned; commit=none

## Task workflow update - 2026-09-08T02:06:12+00:00
- Validation: castor test --filter='TuiMountedTranscriptVirtualTest|RuntimeEventMapperTest|TranscriptProjectorTest|TuiTranscriptBlocksVirtualRenderTest|SubagentResultRendererTest|DeferredSubagentBatchProgressSnapshotFactoryTest|SubagentProgressSnapshotSerializerTest|SubagentProgressProjectionTest' PASS: 236 tests, 1172 assertions; max case 0.020331s.; castor test:controller-replay PASS: 9 tests, 135 assertions; max case 4.055127s. Existing tool completion proof now checks transported start/end timestamps.; Mounted virtual proof drives real Symfony scheduler with explicit time and MockClock; verifies independent timers, frozen success/failure/cancellation, replay parity, stable node identity, unchanged-second invalidation avoidance, and detach/terminal timer cleanup. No sleeps or new process journeys.; castor deptrac, castor phpstan, castor dead-code, castor cs-check, castor docs:validate PASS. git diff --check PASS.; castor check not run, as required by task-start. CODE-REVIEW transition owns full gate.; A scoped PHPStan run on the test file reported existing test-only static assertion/style diagnostics; normal project castor phpstan passed. No broad test cleanup included.
- Summary: Implemented live once-per-second tool header durations with native Symfony ScheduledTickTrait. Completed, failed, and cancelled durations freeze using explicit duration_ms or canonical start/end timestamps, including replay. Terminal single/parallel subagent cards retain compact tools/tokens/elapsed summaries; existing active parallel presentation and detailed model/context lines stay unchanged. No settings or database fields added. Commits c4ddc3ab4 and 8a0abfc30; worktree clean. task-start complete; full castor check and independent review remain for task-to-pr.
- Ownership: owner=main; fork_run=none; revision=c4ddc3ab47a6376674f09ecfb972c992d88b5725; scope=ordinary live/final tool timing and integrated subagent slice corrections; outcome=completed; commit=8a0abfc30

## Task workflow update - 2026-09-08T13:59:46+00:00
- Review: role=reviewer; artifact=agent_4f364f2fb1e9c2f9; revision=8a0abfc30; scope=full origin/main...HEAD diff and specification fidelity; outcome=APPROVE WITH SUGGESTIONS, pending clarification of optional-hardening item labeled SEC. Reviewer attested testing skill and tests/AGENTS.md read and followed.

## Task workflow update - 2026-09-08T14:02:05+00:00
- Validation: Reused focused validation on unchanged revision 8a0abfc30 in the same worktree: 236 tests/1172 assertions, controller replay 9 tests/135 assertions, deptrac/phpstan/dead-code/cs-check/docs:validate and diff check passed.; Virtual mounted scheduler proof covers live independent durations, frozen terminal states, replay, stable nodes and timer cleanup. Controller replay covers timestamp transport. No additional tmux or focused live-provider test needed for this change; full transition gate remains mandatory.
- Summary: Independent specification-fidelity review approved with suggestions at 8a0abfc30. No blockers or changes required. Reviewer clarified malformed internal timestamp parsing is optional diagnostic polish, not a security finding: timestamps have a trusted canonical producer and upstream validation. Deferred optional naming, formatting consistency, and legacy replay suggestions to avoid unsupported scope.
- Review: role=reviewer; artifact=agent_4f364f2fb1e9c2f9; revision=8a0abfc30; scope=full 19-file diff, specification fidelity, scheduler lifecycle, cancellation, replay and test quality; outcome=APPROVE WITH SUGGESTIONS; blockers=none.

## Task workflow update - 2026-09-08T14:05:20+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/480
- Updated PR Status: open
- Validation: QA reports: var/reports/qa-20260908-140221-7178-5166fb50. All lanes pass: 4877 unit/integration, 9 controller replay, 9 TUI, 5 llm-real tests; no cases over 10s. Static/docs/catalog lanes pass.; castor clean:cleanup:workers:list found no stale QA worker candidates. Worktree unchanged and clean at 8a0abfc30.
- Summary: Transition passed QA and pushed branch, then failed because task-tool gh credential is invalid. Diagnosed actual exception in runtime log. Shell gh auth is valid; created PR #480 using that existing authenticated environment. Will complete status through move_task with pushOnly to avoid the broken task-tool PR credential, preserving recorded PR metadata.

## Task workflow update - 2026-09-08T14:06:13+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (44.8s).
- Pushed task/2026-08-27-show-live-tool-call-duration-in-tui to origin.
- branch 'task/2026-08-27-show-live-tool-call-duration-in-tui' set up to track 'origin/task/2026-08-27-show-live-tool-call-duration-in-tui'.
- Skipped PR creation (pushOnly: true).
- Summary: PR #480 created using authenticated shell after task-tool gh auth failure. Independent reviewer approved with suggestions at 8a0abfc30, no blockers. Completing transition without task-tool PR creation; PR metadata already recorded.

## Task workflow update - 2026-09-08T15:23:07+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Accepted user review nit: display durations below one second in milliseconds instead of 0s. Keep existing tick cadence and durations of one second or more unchanged.

## Task workflow update - 2026-09-08T15:23:48+00:00
- Ownership: owner=main; fork_run=none; revision=8a0abfc30; scope=subsecond duration formatting in tool headers and subagent summaries with boundary rendering proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-08T15:28:32+00:00
- Validation: castor test --filter='ToolDurationHeaderWidgetTest|SubagentResultRendererTest|TuiMountedTranscriptVirtualTest|SubagentProgressProjectionTest' passed 44 tests, 344 assertions. Boundary proof covers 0, 123, 999, 1000ms and existing minute formatting. git diff --check passed.
- Summary: Durations below 1000ms now display milliseconds in tool headers and subagent card/plain-text summaries; seconds/minutes and tick cadence unchanged.
- Ownership: owner=main; fork_run=none; revision=8a0abfc30; scope=subsecond duration formatting and boundary rendering proof; outcome=completed; commit=997d10124
- Review: role=reviewer; artifact=agent_4f364f2fb1e9c2f9; revision=997d10124; scope=accepted millisecond display nit, five-file incremental diff and specification fidelity; outcome=APPROVE WITH SUGGESTIONS; blockers=none.

## Task workflow update - 2026-09-08T15:29:29+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (45.3s).
- Pushed task/2026-08-27-show-live-tool-call-duration-in-tui to origin.
- branch 'task/2026-08-27-show-live-tool-call-duration-in-tui' set up to track 'origin/task/2026-08-27-show-live-tool-call-duration-in-tui'.
- Skipped PR creation (pushOnly: true).
- Summary: Push accepted millisecond display nit at 997d10124 to existing PR #480. Independent review approved with suggestions; no blockers. pushOnly retains workaround for previously diagnosed task-tool GitHub authentication failure.

## Task workflow update - 2026-09-08T15:30:41+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-27-show-live-tool-call-duration-in-tui: ide_close_project returned isError.
- Merged task/2026-08-27-show-live-tool-call-duration-in-tui into integration checkout.
- Auto-merging src/CodingAgent/Runtime/Projection/SubagentProgressDisplayFormatter.php
Auto-merging src/Tui/Transcript/SubagentProgressCardWidget.php
Auto-merging tests/CodingAgent/Runtime/Projection/SubagentProgressProjectionTest.php
Auto-merging tests/Tui/Transcript/SubagentResultRendererTest.php
Merge made by the 'ort' strategy.
 src/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Progress/DeferredSubagentBatchProgressSnapshotFactory.php       |  99 ++++++++++++++++++++++++++++++++--------------
 src/CodingAgent/Agent/Execution/SubagentProgressParallelChildReportDTO.php                                              |   1 +
 src/CodingAgent/Agent/Execution/SubagentProgressSnapshotBuilder.php                                                     |   3 ++
 src/CodingAgent/Runtime/Contract/SubagentProgress/SubagentProgressChildRowDTO.php                                       |   2 +
 src/CodingAgent/Runtime/Projection/SubagentProgressDisplayFormatter.php                                                 |   9 +++--
 src/CodingAgent/Runtime/ProjectionPipeline/ToolProjectionSubscriber.php                                                 |  36 +++++++++++++++--
 src/CodingAgent/Runtime/Protocol/AGENTS.md                                                                              |   2 +
 src/CodingAgent/Runtime/Protocol/RuntimeEventTranslator.php                                                             |   2 +
 src/Tui/Transcript/SubagentProgressCardWidget.php                                                                       |   7 +++-
 src/Tui/Transcript/SubagentResultRenderer.php                                                                           |  13 +++---
 src/Tui/Transcript/ToolDurationHeaderWidget.php                                                                         |  80 +++++++++++++++++++++++++++++++++++++
 src/Tui/Transcript/TranscriptToolRenderer.php                                                                           |  26 +++++++-----
 tests/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Progress/DeferredSubagentBatchProgressSnapshotFactoryTest.php | 209 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Runtime/Contract/SubagentProgressSnapshotSerializerTest.php                                           |   4 ++
 tests/CodingAgent/Runtime/Controller/E2E/ControllerReplayToolCompletionTest.php                                         |   6 +++
 tests/CodingAgent/Runtime/Projection/SubagentProgressProjectionTest.php                                                 |   3 +-
 tests/CodingAgent/Runtime/RuntimeEventMapperTest.php                                                                    |  18 +++++++++
 tests/Tui/Screen/TuiMountedTranscriptVirtualTest.php                                                                    |  98 +++++++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Transcript/SubagentResultRendererTest.php                                                                     |  70 ++++++++++++++++++++++++++++++--
 tests/Tui/Transcript/ToolDurationHeaderWidgetTest.php                                                                   |  39 ++++++++++++++++++
 20 files changed, 670 insertions(+), 57 deletions(-)
 create mode 100644 src/Tui/Transcript/ToolDurationHeaderWidget.php
 create mode 100644 tests/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Progress/DeferredSubagentBatchProgressSnapshotFactoryTest.php
 create mode 100644 tests/Tui/Transcript/ToolDurationHeaderWidgetTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-27-show-live-tool-call-duration-in-tui.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-27-show-live-tool-call-duration-in-tui.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR #480 merged at 2026-09-08T15:29:55Z, merge commit 9eb6e8ac711fc07b3fb2c734e968d39a48841e87. Integration checkout clean before transition.

## Task workflow update - 2026-09-08T15:32:17+00:00
- Updated PR Status: merged
- Validation: Post-merge castor check PASS; reports var/reports/qa-20260908-153049-21632-8a808de8. 4895 unit/integration tests, 9 controller replay, 9 TUI, 5 llm-real; all static/docs/catalog lanes pass.; All test cases below 10 seconds; maximum 6.577727s. Artifact integrity, leak check and cache guard passed. Final output: quality: ok (121.2s).; castor clean:cleanup:workers:list after QA completion found no stale candidates. Git status clean; task worktree absent.
- Summary: DONE. Integration checkout clean at 2be36dbf543d63e0e5d0148e001217949f4971e2; task worktree removed. IDE close reported degradation, but filesystem cleanup succeeded. Post-merge full QA passed. Bash initially returned an unclean result while the QA process continued; recovered final successful output from its command log without rerunning QA.

## Task workflow update - 2026-09-10T22:49:35+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
