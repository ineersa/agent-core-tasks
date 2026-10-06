# Restore per-child context usage in parallel subagent cards

## Goal
User reports parallel subagent running/completed cards show cache reuse, tool counts, cumulative tokens and elapsed time, but omit the per-child context indicator such as `CTX 98% 196.9k/200.0k`.

Initial source finding: SubagentProgressChildRowDTO already carries latestInputTokens and contextWindow, and SubagentProgressSnapshotBuilder supplies both. SubagentProgressCardWidget::buildSingleLines returns early for parallel children before the existing context-usage rendering used by single cards. The omission is in presentation, not a missing wire field.

Restore per-child CTX in parent-transcript parallel cards, both while running and in completed results. Reuse the existing context formatter, latest-request usage and child-specific window. Do not confuse context utilization with cumulative token totals or cache reuse. Keep the two-row-per-child layout height-stable and preserve the existing cache indicator and details. Omit unavailable telemetry as single cards already do. No new protocol fields, storage, settings or unrelated card redesign.

Main owns this cohesive TUI correction. Use deterministic virtual transcript rendering as the lowest correct proof. Implementation has been requested; review/full QA and PR submission remain separate phases.

## Acceptance criteria
- Parallel running cards display each child's own CTX percentage and used/window token counts when available.
- Completed parallel result cards retain the same per-child context information.
- Context uses latest input tokens and child context window, not cumulative tokens or cache reuse; absent telemetry is not fabricated.
- Existing cache percentages, activity/task text and fixed two-row child footprint remain intact.
- Focused virtual-render regression proves distinct child values, running/completed output and unavailable telemetry. Targeted Castor validation passes; independent review and transition-owned full gate follow before PR submission.

## Workflow metadata
Status: DONE
Branch: task/2026-10-06-restore-context-usage-in-parallel-subagent-cards
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-10-06-restore-context-usage-in-parallel-subagent-cards
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/554
PR Status: merged
Started: 2026-10-06T17:07:32+00:00
Completed: 2026-10-06T17:34:09+00:00

## Work log
- Created: 2026-10-06T17:07:19+00:00

## Task workflow update - 2026-10-06T17:07:32+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-10-06-restore-context-usage-in-parallel-subagent-cards.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-10-06-restore-context-usage-in-parallel-subagent-cards.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-10-06-restore-context-usage-in-parallel-subagent-cards.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-10-06-restore-context-usage-in-parallel-subagent-cards.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-10-06-restore-context-usage-in-parallel-subagent-cards.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-10-06-restore-context-usage-in-parallel-subagent-cards/.idea.
- Summary: User requests task creation and restoration. Main owns one cohesive presentation change using existing child context telemetry and formatter. Preserve parallel card height and cache statistics; no new wire fields/settings. Focused virtual proof during implementation, review and full QA in later PR phase.

## Task workflow update - 2026-10-06T17:10:09+00:00
- Summary: Routing pass confirmed child DTO and builder already carry latest input/window for parallel rows. Restore context in the existing detail row before activity/task so it remains visible on narrow screens; retain existing context colors and activity styling. Main owns the cohesive widget/tests change; no independent fork slice. Testing skill and tests/AGENTS.md read/followed.
- Ownership: owner=main; fork_run=none; revision=0f00a13909e864a22c740b26e5a40b9978bbf67b; scope=parallel card context presentation using existing telemetry/formatter, virtual lifecycle regression and existing height invariant; outcome=assigned; commit=none

## Task workflow update - 2026-10-06T17:15:27+00:00
- Validation: RED: testParallelContextUsageSurvivesCompletion@narrow card failed on baseline 0f00a1390 because CTX was absent.; GREEN focused Castor: 91 tests /557 assertions, max individual case 0.0492s, none over 10s.; Virtual transcript regression covers 80/120 columns, running-to-terminal tool events, two distinct context values, cumulative input distinct from latest input, missing latest usage, preserved cache/token/activity/task output.; Existing renderer height regression now includes nonzero context telemetry and still asserts fixed heights across running, waiting-human and terminal updates.; Targeted PHPStan and LSP, deptrac, cs-check, git diff --check PASS. Worktree clean after commit; testing skill and tests/AGENTS.md read/followed.; Full castor check deferred to CODE-REVIEW transition as required.
- Summary: Implemented and committed at fa9746e748de2c887c39369babbac1f0ee8a4baa. Each parallel child detail row now starts with existing CTX usage before activity/task, in both live and completed cards. Context colors are limited to context text; existing activity styling and cache/header metrics retained. Child-specific latest input/window are reused, with missing usage omitted. Two-row footprint unchanged. No wire, settings, storage or provider changes. Ready for task-to-pr; not pushed, reviewed, merged or deployed.
- Ownership: owner=main; fork_run=none; revision=0f00a13909e864a22c740b26e5a40b9978bbf67b; scope=parallel card context presentation using existing telemetry/formatter, virtual lifecycle regression and existing height invariant; outcome=completed; commit=fa9746e748de2c887c39369babbac1f0ee8a4baa
- Next phase: task-to-pr with independent specification-fidelity review and transition-owned full QA before PR submission.

## Task workflow update - 2026-10-06T17:24:38+00:00
- Summary: Independent specification-fidelity reviewer approved fa9746e74 with suggestions only, no blockers. Confirmed per-child CTX for live/completed cards, existing styling/cache metrics and stable height, genuine virtual proof, no cumulative fallback, and minimal scope. Keep approved code unchanged; optional comment/constant cleanup is not needed for submission.
- Review: role=reviewer; artifact=agent_76562506b774cc1d; revision=fa9746e748de2c887c39369babbac1f0ee8a4baa; scope=full three-file diff 0f00a1390..fa9746e74, specification fidelity and TUI proof; decision=APPROVE WITH SUGGESTIONS; blockers=none. Reviewer explicitly confirmed testing skill and tests/AGENTS.md read/followed.
- Reuse unchanged 91-case focused validation. Full QA remains CODE-REVIEW transition-owned.

## Task workflow update - 2026-10-06T17:26:56+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (103.8s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-06-restore-context-usage-in-parallel-subagent-cards/var/reports/qa-20261006-172513-15474-af568d65.
- Session/run: 79.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-06T17:26:58+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-10-06-restore-context-usage-in-parallel-subagent-cards to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-06-restore-context-usage-in-parallel-subagent-cards/var/reports/qa-20261006-172513-15474-af568d65.
- Session/run: 79.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-06T17:27:01+00:00
- castor check passed (103.8s).
- Pushed task/2026-10-06-restore-context-usage-in-parallel-subagent-cards to origin.
- Created PR: <url>
- Session/run: 79.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-06T17:27:01+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (103.8s).
- Pushed task/2026-10-06-restore-context-usage-in-parallel-subagent-cards to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/554
- Validation: 91 focused tests /557 assertions passed, maximum individual case 0.0492s; baseline RED proof recorded.; Targeted PHPStan/LSP, deptrac, style and diff checks passed.; Existing live-smoke cache warmup passed: 5 tests /30 assertions, 5.609s.
- Summary: Submit reviewed revision fa9746e748de2c887c39369babbac1f0ee8a4baa. Reviewer agent_76562506b774cc1d approved with suggestions only, no blockers. Restore CTX in parallel live/completed cards using existing telemetry, formatter and detail row; preserve cache metrics and height.

## Task workflow update - 2026-10-06T17:28:11+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/554
- Updated PR Status: open
- Validation: Full castor check PASS in 103.8s.; JUnit audit: 5,076 total cases; 5,051 unit/integration, 14 controller replay, 6 TUI, 5 llm-real. Maximum individual case 5.6847s; none over 10s.; Reports: var/reports/qa-20261006-172513-15474-af568d65/ in task worktree.; Final git status and diff checks clean.
- Summary: PR #554 created and description updated with full gate results at fa9746e748de2c887c39369babbac1f0ee8a4baa. Independent reviewer approved with no blockers; CODE-REVIEW transition-owned castor check passed. Verified PR OPEN at exact reviewed head and clean worktree. Not merged or deployed.
- Review: agent_76562506b774cc1d; scope=full three-file diff 0f00a1390..fa9746e74; target=fa9746e748de2c887c39369babbac1f0ee8a4baa; decision=APPROVE WITH SUGGESTIONS; blockers=none. No code changes after review.

## Task workflow update - 2026-10-06T17:34:09+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-10-06-restore-context-usage-in-parallel-subagent-cards into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Transcript/SubagentProgressCardWidget.php         | 14 ++++++++++----
 tests/Tui/Screen/TuiTranscriptBlocksVirtualRenderTest.php | 59 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Transcript/SubagentResultRendererTest.php       |  2 ++
 3 files changed, 71 insertions(+), 4 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-10-06-restore-context-usage-in-parallel-subagent-cards.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-10-06-restore-context-usage-in-parallel-subagent-cards.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: Verified PR #554 MERGED on GitHub at 2026-10-06T17:32:48Z as a287a88306a5a17b765bb6ff5414cdeb8d32640f, reviewed head fa9746e748de2c887c39369babbac1f0ee8a4baa. Integration checkout clean. Close task, integrate and remove worktree, then validate merged result.

## Task workflow update - 2026-10-06T17:35:48+00:00
- Updated PR Status: merged
- Validation: Integrated castor check PASS in 148.1s, all 11 lanes. Reports: var/reports/qa-20261006-173422-20741-c717f9de/.; JUnit: 5,076 cases, max 5.289411s, none over 10s. Unit/integration 5,051/23,765 assertions; controller replay 14/261; TUI 6/46; llm-real 5/30.; QA artifacts and owned-resource leak check clean; proxy cache stable 406 to 406.; git status and diff checks clean; task worktree absence verified.
- Summary: PR #554 verified merged and task closed. Post-merge integration castor check passed at HEAD fe3b43f76e979ff032cb5fc53baca2193315c69e. Worktree removed and integration checkout clean. Integration PHAR rebuilt and smoke tests passed; installed launcher and live sessions untouched.
