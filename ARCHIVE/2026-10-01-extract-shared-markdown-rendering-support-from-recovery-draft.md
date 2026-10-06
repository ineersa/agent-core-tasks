# Extract shared Markdown rendering support from recovery draft

## Goal
User approved splitting standalone improvements out of draft PR541 before replacing its shared-state architecture. Extract only shared Symfony Markdown parser/highlighter support and its focused deterministic regression from source commit932ce8500. Base new work on main, not shared cache architecture. Preserve rendering behavior and existing public APIs; no runtime/state/cache/claim-recovery/logging changes. Parent recovery task retains redesign scope. This is first independently reviewable slice, not a complete memory fix.

## Acceptance criteria
- Per-screen renderers reuse Markdown parser/highlighter through existing Symfony facilities.
- Focused regression proves sharing reduces allocation without requiring shared RunState/history cache architecture.
- No unrelated changes to session recovery, canonical ownership, metadata policies, worker claims or memory limits.
- Focused Castor tests, static/architecture/style checks pass; full gate remains task-to-pr.

## Workflow metadata
Status: DONE
Branch: task/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft
Fork run: agent_381bd2793ef79f25
PR URL: https://github.com/ineersa/agent-core/pull/542
PR Status: merged
Started: 2026-10-01T21:11:00+00:00
Completed: 2026-10-02T01:52:39+00:00

## Work log
- Created: 2026-10-01T21:09:13+00:00

## Task workflow update - 2026-10-01T21:11:00+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft/.idea.
- Summary: User rolled back the eight integration-main test edits; verified both main and recovery worktree clean. Proceed with independent Markdown extraction from932ce8500 onto main4f107b034.

## Task workflow update - 2026-10-01T21:11:33+00:00
- Summary: Main owns bounded nine-file extraction. Read source diff: six Transcript production paths plus standalone subprocess memory fixture/test and SubagentResultRenderer test adaptation. No shared projection/runtime dependency needed; source reference932ce8500, targetmain4f107b034. QuestionOverlay comment-only change omitted. Existing renderers/factory, Symfony MarkdownWidget constructor and focused regression are the proof boundary.
- Ownership: owner=main; fork_run=none; revision=4f107b034; scope=extract independent Markdown parser/highlighter sharing and focused allocation regression; outcome=assigned; commit=none

## Task workflow update - 2026-10-01T21:14:58+00:00
- Validation: Focused Castor70tests393assertions PASS,0.650s PHPUnit/1.1s runner.; Counterfactual Castor memory case fails75497472bytes vs50331648budget; restored source passes29360128bytes at128M hardlimit.; Castor phpstan PASS0errors; deptrac PASS0violations; cs-check PASS; gitdiffcheck PASS.; No actual-session128M fix or long-term plateau claim. No original live session or integration-main edits.
- Summary: Independent Markdown extraction implemented on clean main baseline and committed32c8c51df: nine files only, no shared cache/state/runtime ownership changes. Corrected misleading historical fixture claims: synthetic render load is not actual production session2 window. Repeated counterfactual removing shared parser/highlighter fails at72MiB against48MiB budget; restored sharing passes at28MiB. Not pushed or moved toCODE-REVIEW; independent review/full transition gate still required.
- Ownership: owner=main; fork_run=none; revision=32c8c51df; scope=independent shared Markdown renderer extraction and allocation regression; outcome=completed; commit=32c8c51df

## Task workflow update - 2026-10-01T23:52:32+00:00
- Summary: Independent reviewer=agent_d4e9f04be34471e0; target=32c8c51df; scope=entire independent extraction; specification-fidelity verdict=APPROVE WITH SUGGESTIONS.

## Task workflow update - 2026-10-01T23:57:58+00:00
- Summary: Independent review approved by agent_d4e9f04be34471e0; reviewed tree committed at 32c8c51df. Specification fidelity confirmed for independent Change1 scope; no unresolved blocking review findings. Ready for transition-owned full QA gate.

## Task workflow update - 2026-10-02T00:00:01+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (100.0s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft/var/reports/qa-20261001-235821-10968-e15b9fea.
- Session/run: 75.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-02T00:00:03+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft/var/reports/qa-20261001-235821-10968-e15b9fea.
- Session/run: 75.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-02T00:00:07+00:00
- castor check passed (100.0s).
- Pushed task/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft to origin.
- Created PR: <url>
- Session/run: 75.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-02T00:00:07+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (100.0s).
- Pushed task/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/542

## Task workflow update - 2026-10-02T00:20:22+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Address user review before merge: widget-owned parser lifetime (#542), failed handle-stat diagnostics (#543), applied batch boundary detection and best-effort checkpoint failures (#544). No merge authorized or performed.

## Task workflow update - 2026-10-02T00:20:54+00:00
- Ownership: owner=fork; fork_run=pending; revision=32c8c51df; scope=widget-owned parser lifetime and WeakReference release proof; outcome=assigned; commit=none

## Task workflow update - 2026-10-02T01:10:05+00:00
- Recorded fork run: agent_381bd2793ef79f25
- Summary: User review fixes completed: widget-owned parser, WeakReference eviction proof,32MiB shared/72MiB unshared. Independent reviewer agent_d4e9f04be34471e0 APPROVED current committed tree 1791e3e07; no blocking findings. Full transition gate pending.
- Ownership: owner=fork; fork_run=agent_381bd2793ef79f25; revision=1791e3e07; scope=user PR review corrections; outcome=completed; commit=1791e3e07

## Task workflow update - 2026-10-02T01:11:25+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (66.0s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft/var/reports/qa-20261002-011019-34901-258a09e9.
- Session/run: 75.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-02T01:11:27+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft/var/reports/qa-20261002-011019-34901-258a09e9.
- Session/run: 75.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-02T01:11:27+00:00
- castor check passed (66.0s).
- Pushed task/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft to origin.
- PR already exists: <url>
- Session/run: 75.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-02T01:11:27+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (66.0s).
- Pushed task/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/542

## Task workflow update - 2026-10-02T01:52:39+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Transcript/MarkdownRenderSupport.php                          |  47 +++++++++++++++++++++++++++++
 src/Tui/Transcript/SubagentResultRenderer.php                         |   3 +-
 src/Tui/Transcript/TranscriptBlockRenderer.php                        |   7 +++--
 src/Tui/Transcript/TranscriptBlockWidgetFactory.php                   |  21 +++++++++++--
 src/Tui/Transcript/TranscriptVisualProjector.php                      |   4 ---
 src/Tui/Transcript/WriteToolCallContentRenderer.php                   |   3 +-
 tests/Tui/Transcript/Fixtures/large-resume-render-memory.php          | 164 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Transcript/Fixtures/markdown-render-support-ast-release.php | 102 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Transcript/LargeResumeRenderMemoryTest.php                  |  47 +++++++++++++++++++++++++++++
 tests/Tui/Transcript/MarkdownRenderSupportAstReleaseTest.php          |  79 ++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Transcript/SubagentResultRendererTest.php                   |   4 +--
 11 files changed, 467 insertions(+), 14 deletions(-)
 create mode 100644 src/Tui/Transcript/MarkdownRenderSupport.php
 create mode 100644 tests/Tui/Transcript/Fixtures/large-resume-render-memory.php
 create mode 100644 tests/Tui/Transcript/Fixtures/markdown-render-support-ast-release.php
 create mode 100644 tests/Tui/Transcript/LargeResumeRenderMemoryTest.php
 create mode 100644 tests/Tui/Transcript/MarkdownRenderSupportAstReleaseTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft.
- Pulled integration checkout: Merge made by the 'ort' strategy.
 docs/datadog.md                                                                     |  48 ++++++
 docs/session-storage.md                                                             |   4 +
 src/CodingAgent/Agent/Artifact/AgentChildRunEventStore.php                          | 104 ++++++++----
 src/CodingAgent/Logging/LogContextProcessor.php                                     |  11 ++
 src/CodingAgent/Logging/ProcessMemorySnapshotLogger.php                             | 135 ++++++++++++++++
 src/CodingAgent/Runtime/ProjectionPipeline/CompactionProjectionSubscriber.php       |  26 +++
 src/CodingAgent/Session/JsonlPhysicalReadObservation.php                            |  87 ++++++++++
 src/CodingAgent/Session/JsonlRunEventLog.php                                        | 103 +++++++++++-
 src/CodingAgent/Session/SessionRunEventStore.php                                    | 125 +++++++++------
 src/Tui/Application/InteractiveMode.php                                             |  53 ++++++-
 src/Tui/Listener/TickPollListener.php                                               | 109 ++++++++++++-
 src/Tui/Runtime/AppliedMemoryBoundaryObservation.php                                |  27 ++++
 src/Tui/Runtime/RuntimeEventPoller.php                                              |  88 +++++++++++
 tests/CodingAgent/Agent/Artifact/AgentChildRunEventStoreTest.php                    |  49 +++++-
 tests/CodingAgent/Logging/LogContextProcessorTest.php                               |  49 +++++-
 tests/CodingAgent/Logging/ProcessMemorySnapshotLoggerTest.php                       | 100 ++++++++++++
 tests/CodingAgent/Runtime/ProjectionPipeline/CompactionProjectionSubscriberTest.php |  67 ++++++++
 tests/CodingAgent/Session/Fixtures/session-run-event-store-physical-read-memory.php | 249 +++++++++++++++++++++++++++++
 tests/CodingAgent/Session/JsonlRunEventLogReverseLinesFailureTest.php               | 349 ++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Session/SessionRunEventStoreTest.php                              | 145 +++++++++++++++++
 tests/Tui/Application/InteractiveModeMemoryCheckpointWiringTest.php                 | 359 +++++++++++++++++++++++++++++++++++++++++
 tests/Tui/E2E/TuiArtifactBootE2eTest.php                                            |  71 ++++++++-
 tests/Tui/Listener/TickPollListenerTest.php                                         | 747 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 23 files changed, 3006 insertions(+), 99 deletions(-)
 create mode 100644 src/CodingAgent/Logging/ProcessMemorySnapshotLogger.php
 create mode 100644 src/CodingAgent/Session/JsonlPhysicalReadObservation.php
 create mode 100644 src/Tui/Runtime/AppliedMemoryBoundaryObservation.php
 create mode 100644 tests/CodingAgent/Logging/ProcessMemorySnapshotLoggerTest.php
 create mode 100644 tests/CodingAgent/Session/Fixtures/session-run-event-store-physical-read-memory.php
 create mode 100644 tests/CodingAgent/Session/JsonlRunEventLogReverseLinesFailureTest.php
 create mode 100644 tests/Tui/Application/InteractiveModeMemoryCheckpointWiringTest.php.
- Summary: User merged PR542 on GitHub; verified MERGED at77644a69f12efb1c42033fce9b137046734fb9cd. Synchronize integration checkout and clean task worktree; combined post-merge gate follows.

## Task workflow update - 2026-10-02T01:56:56+00:00
- Updated PR Status: merged
- Validation: castor check PASS: var/reports/qa-20261002-015453-65964-786ea6b2; 4912 unit/integration tests22388assertions; TUI6/46; live5/30; controller replay/static/dead-code/style/docs/lsp pass; git status --short: empty
- Summary: Post-merge integrated castor check PASS at b58640142. Initial gate failed only stale PHPStan DCD result cache referencing missing hash; preserved cache in var/tmp/phpstan-dead-code-stale-20261002-015304 and rebuilt via Castor. Final all lanes green, leak check clean. Integration working tree clean; task worktree removed.
