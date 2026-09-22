# Unify code_mode output capping and strengthen usage guidance

## Goal
Use standard output capping for combined code_mode return value and stdout/stderr, with a 50,000-character cap and normal saved-output recovery. Remove separate lossy 4 KB diagnostics truncation while retaining useful stream labels, error normalization, and process safety limits. Strengthen guidance to prefer code_mode for multi-step calls where intermediate results need no model interpretation, filter/compare before returning evidence, and avoid pointless single-call wrappers or wholesale intermediate dumps. Keep direct tools available. No exposure mode, new settings, or discovery API.

## Acceptance criteria
- Combined code_mode output uses the standard 50,000-character cap and recoverable saved output rather than a separate diagnostics truncation marker.
- Successful output and error-path diagnostics retain useful evidence through ordinary capping.
- Guidelines encourage suitable multi-step code_mode use without hiding other tools.
- Focused regression validation covers stdout/stderr beyond 4 KB and oversized output recovery.

## Workflow metadata
Status: DONE
Branch: task/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance
Fork run: 5f8dc8c0-a160-512d-bfa7-671556aec595
PR URL: https://github.com/ineersa/agent-core/pull/505
PR Status: merged
Started: 2026-09-17T19:58:39+00:00
Completed: 2026-09-17T20:45:39+00:00

## Work log
- Created: 2026-09-17T19:58:19+00:00

## Task workflow update - 2026-09-17T19:58:39+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance.

## Task workflow update - 2026-09-17T19:59:25+00:00
- Summary: Main routing found separate 4KB diagnostics cap, normal OutputCap default 20KB/document 50KB selection, and existing model-facing processor integration regressions asserting old truncation. Delegate cohesive pipeline slice because success/error normalization, notifications and subprocess validation require deeper investigation. No tool exposure changes.
- Ownership: owner=fork; fork_run=pending; revision=6151d2d83; scope=code_mode output pipeline, guidance, docs and focused validation; outcome=assigned; commit=none

## Task workflow update - 2026-09-17T20:14:31+00:00
- Recorded fork run: 554ba6a7-e4b3-0df1-0000-000000000000
- Summary: Parent rejects first pass as incomplete: 50KB host tails still discard full output before saved recovery. code_mode failures must also use requested 50k cap. Continuing same owner.
- Ownership: owner=fork; fork_run=agent_554ba6a7e4b30df1; revision=3c4e6ae96; scope=correct premature stdout/stderr loss and code_mode error cap selection in same worktree; outcome=assigned; commit=none

## Task workflow update - 2026-09-17T20:14:49+00:00
- Recorded fork run: 5f8dc8c0-a160-512d-bfa7-671556aec595
- Summary: Corrected previous malformed fork run metadata with verified child run ID.
- Ownership: owner=fork; fork_run=5f8dc8c0-a160-512d-bfa7-671556aec595; revision=3c4e6ae96; scope=correct premature stdout/stderr loss and code_mode error cap selection in same worktree; outcome=assigned; commit=none

## Task workflow update - 2026-09-17T20:23:49+00:00
- Recorded fork run: 5f8dc8c0-a160-512d-bfa7-671556aec595
- Validation: Focused Castor regressions passed: 80 tests, 369 assertions, including real-host >50k head/tail preservation on success and early exit.; Scoped Castor PHPStan passed; Castor style fix and docs:validate passed.; Full castor check not run in task-start; no live model adherence evaluation performed.
- Summary: Implementation complete after parent corrections. Removed both diagnostics truncation and host tail loss using native Symfony Process buffers. code_mode success and error results select standard document cap (default 50k) with saved recovery. Stronger batching/filtering guidance; direct tools unchanged. Clean worktree at 28dafa4b8. Full gate and independent review remain for task-to-pr.
- Ownership: owner=fork; fork_run=5f8dc8c0-a160-512d-bfa7-671556aec595; revision=6151d2d83; scope=code_mode output pipeline, guidance, docs and focused validation; outcome=completed; commit=28dafa4b8aeb7e2946fc012457e303ca9bef5087

## Task workflow update - 2026-09-17T20:36:15+00:00
- Summary: Independent reviewer agent_69ffc815ee977696 reviewed 28dafa4b8 against 6151d2d83, specification fidelity included. APPROVE WITH SUGGESTIONS, no blockers. Revalidated 80 tests/369 assertions and scoped PHPStan. Correct stale host-tail comment before submission.
- Review: role=reviewer; artifact=agent_69ffc815ee977696; revision=28dafa4b8; scope=complete task diff, specification fidelity, lifecycle/output recovery/tests; verdict=APPROVE WITH SUGGESTIONS
- Ownership: owner=main; fork_run=none; revision=28dafa4b8; scope=stale diagnostics comment correction; outcome=assigned; commit=none

## Task workflow update - 2026-09-17T20:37:29+00:00
- Summary: Final independent reviewer agent_69ffc815ee977696 APPROVE at e373aa8b7. Comment-only correction reviewed; focused validation reused. No blockers. No existing code_mode-specific live smoke found; full transition includes live smoke lane.
- Ownership: owner=main; fork_run=none; revision=28dafa4b8; scope=stale diagnostics comment correction; outcome=completed; commit=e373aa8b7
- Review: role=reviewer; artifact=agent_69ffc815ee977696; revision=e373aa8b7; scope=final delta plus previously reviewed full diff; verdict=APPROVE

## Task workflow update - 2026-09-17T20:40:14+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (149.0s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance/var/reports/qa-20260917-203745-6118-992cab50.
- Session/run: 57.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-17T20:40:16+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance/var/reports/qa-20260917-203745-6118-992cab50.
- Session/run: 57.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-17T20:40:18+00:00
- castor check passed (149.0s).
- Pushed task/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance to origin.
- Created PR: <url>
- Session/run: 57.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-17T20:40:18+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (149.0s).
- Pushed task/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/505
- Summary: Independent review approved e373aa8b7; focused 80 tests/369 assertions, scoped PHPStan and docs validation passed.

## Task workflow update - 2026-09-17T20:45:39+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance: ide_close_project returned isError.
- Merged task/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance into integration checkout.
- Merge made by the 'ort' strategy.
 docs/tool-execution.md                                                         |   1 +
 docs/tools.md                                                                  |  14 ++++++++++++--
 src/CodingAgent/Tool/CodeMode/CodeModeDiagnostics.php                          |  38 ++++++--------------------------------
 src/CodingAgent/Tool/CodeMode/CodeModeDiagnosticsToolResultProcessor.php       |  13 ++++++-------
 src/CodingAgent/Tool/CodeMode/CodeModeExecutionResult.php                      |   2 +-
 src/CodingAgent/Tool/CodeMode/CodeModeHostBridge.php                           | 130 ++++++++++++++++++++++++++++++++++------------------------------------------------------------------------------------------------
 src/CodingAgent/Tool/CodeModeTool.php                                          |   4 ++--
 src/CodingAgent/Tool/OutputCap.php                                             |  11 ++++++-----
 tests/CodingAgent/Tool/CodeMode/CodeModeDiagnosticsTest.php                    |  29 ++++++++---------------------
 tests/CodingAgent/Tool/CodeMode/CodeModeDiagnosticsToolResultProcessorTest.php |  32 ++++++++++++++++++++++++++++++++
 tests/CodingAgent/Tool/CodeMode/CodeModeHostBridgeTest.php                     |  41 ++++++++++++++++++++++++++++++++++-------
 tests/CodingAgent/Tool/CodeMode/CodeModeModelFacingDiagnosticsTest.php         |  37 ++++++++++++++++++++++---------------
 tests/CodingAgent/Tool/OutputCapTest.php                                       |  12 ++++++++++++
 tests/CodingAgent/Tool/OutputCapToolResultProcessorContractTest.php            |  33 +++++++++++++++++++++++++++++++++
 14 files changed, 209 insertions(+), 188 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-17-unify-code-mode-output-capping-and-strengthen-usage-guidance.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: Verified GitHub PR #505 merged at 2026-09-17T20:44:58Z, merge commit 3d1bb20208fb15355080e0f03479233c7d29541e. Running required post-merge integration validation next.

## Task workflow update - 2026-09-17T20:47:04+00:00
- Updated PR Status: merged
- Validation: castor check passed in integration checkout (155.6s), all 10 lanes; QA report var/reports/qa-20260917-204546-11361-414441ae.; 5086 unit/integration tests, controller replay 11, TUI 9, live LLM 5 passed; static/style/docs/deptrac/dead-code/catalog checks passed.; QA leak check clean; proxy cache unchanged; git status clean; task worktree absence confirmed.
- Summary: Post-merge integration validation passed at 3a9d76f83. Git status clean; task worktree removed. IDE project-close reported degraded during transition, but filesystem cleanup succeeded.
