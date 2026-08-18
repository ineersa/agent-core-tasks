# task_list tool produces redundant string+structure TOON output

## Goal
When calling `task_list` in the TUI, the tool result shows both a raw string (`message`) containing a bullet-list of tasks AND structured `tasks[]` data with the same information. This is redundant and confusing.

**Root cause:** `ListTasksHandler` calls `ToolResult::text($text, ['tasks' => [...], 'include_archive' => ...])`. The `message` field in the resulting TOON payload is the full formatted bullet list from `TaskListFormatter::format()`, while `tasks[]` carries the same data in structured form. The TUI renders both, creating the double output.

Compare with `settings` tool which produces clean single-level TOON output with no redundant fields.

Likely fix: either drop `message` and make a short summary, or drop the structured `tasks` and keep only the formatted string. The tool should produce one canonical representation, not both.

## Acceptance criteria
- task_list output in TUI shows clean single representation (no duplicate string + structure)
- Other task-workflow tools (move_task, create_task, update_task) are checked for the same pattern
- All task-workflow tools still produce valid TOON output the model can consume

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/388
PR Status: merged
Started: 2026-07-31T19:57:32+00:00
Completed: 2026-08-15T03:30:27.411Z

## Work log
- Created: 2026-07-28T19:27:51+00:00

## Task workflow update - 2026-07-28T19:28:43+00:00
- Decision: keep structured `tasks[]` TOON, drop redundant `message` string — same pattern as `settings` tool

## Task workflow update - 2026-07-31T19:57:32+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output.
- Summary: Claimed for implementation after committing the requested .hatfield/extensions-data ignore rule. Finalized decision: retain structured tasks[] output and remove the redundant formatted message; inspect sibling task-workflow tools for the same duplication pattern.

## Task workflow update - 2026-07-31T20:07:04+00:00
- Recon completed in parallel by scouts agent_d38d509ae9dda9a6 and agent_5288fcbefca68d36. Root bug is ListTasksHandler formatting the full list into ToolResult::text while also emitting tasks[]. Sibling audit: create_task uses only a short action summary; move_task/update_task repeat notes in a summary but do not duplicate an entire canonical entity list. Scope remains the finalized decision: task_list keeps structured tasks[] + include_archive and removes message; no new public setting/API/storage surface.
- Implementation worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output. Required proof per task-start request: focused extension contract test plus replay-backed TmuxHarness E2E through the user-visible task_list tool path.

## Task workflow update - 2026-08-06T21:46:47+00:00
- Implementation paused for Hatfield reliability/configuration work. First implementation fork agent_a7f0997ff6034869 failed with Codex WebSocket send error after leaving an uncommitted ToolResult.php change; retry agent_2b5cd10a070cbbaa was cancelled. Resuming now: implementation fork must inspect the partial diff, fetch and merge current origin/main because extension code changed substantially, then complete the finalized structured-only task_list TOON fix and required tests.

## Task workflow update - 2026-08-15T01:50:21.245Z
- Summary: Revived the existing work in place: committed the 7-file partial implementation as ddc99633d (`WIP: fix redundant task_list output`) and merged current main cleanly as c3c5d6da2. No conflicts; worktree is clean. Validation has not yet been run.

## Task workflow update - 2026-08-15T01:57:28.222Z
- Validation: Read and followed .agents/skills/testing/SKILL.md and tests/AGENTS.md.; castor test --filter='TaskWorkflowHandlerToonOutputTest|ToolResultTest' — passed (6 tests, 54 assertions).; castor test:tui --filter=TuiTaskListToolE2eTest — passed (1 test, 7 assertions; tmux 3.7b).; castor phpstan --path=.hatfield/extensions/task-workflow — passed (0 errors).; castor cs-check — passed.; castor deptrac — passed (0 violations).; castor clean:cleanup:workers:list — no stale QA workers.
- Summary: Implementation completed on current main. Verified the structured-only task_list TOON change and constructor references; retained the replay-backed tmux proof because no shared lower-level harness covers this full tool-to-transcript path. Fixed the E2E agent command and code style, committed as 7e9e7eef4. Worktree is clean and remains IN-PROGRESS pending task-to-pr review/full gate.

## Task workflow update - 2026-08-15T02:04:26.166Z
- Summary: task-to-pr reviewer returned REQUEST CHANGES. Core production behavior is correct. Actionable cleanup: remove unused/redundant test lines. Reviewer also requested an explicit decision on unrelated local-main commit 0e230004b (`.pi/settings.json` fork-model change) and proposed extracting duplicated tmux E2E scaffolding; the latter conflicts with the repository rule against broad test refactors in implementation tasks and will be re-reviewed after minimal cleanup.
- Reviewer decision: REQUEST CHANGES. No correctness or security defects in the production task_list change. Blockers raised: unmapped `.pi/settings.json` rider from local main; copied TUI E2E scaffolding; minor dead test lines.

## Task workflow update - 2026-08-15T02:06:48.215Z
- Summary: User explicitly approved keeping local-main commit 0e230004b (`.pi/settings.json`: DeepSeek Flash implementation-fork model) in this PR. This rider is intentional and no longer a specification-fidelity blocker. Narrow reviewer cleanup committed as 62e28dc61.

## Task workflow update - 2026-08-15T02:08:33.657Z
- Summary: Second reviewer pass returned REQUEST CHANGES for one minimality issue: ListTasksHandler redundantly maps TaskInfo fields already normalized by ToolResult::structured(). User also authorized pushing once review and gates pass.

## Task workflow update - 2026-08-15T02:18:14.159Z
- Validation: Final reviewer: APPROVED; specification fidelity, correctness, security, minimality, and test layer accepted.; castor test — passed in task worktree (4481 tests, 17199 assertions).; castor deptrac — passed (0 violations, 0 errors).; castor phpstan — passed (0 errors).; castor cs-check — passed (0 files fixed).; castor test:tui — passed in task worktree (37 tests, 292 assertions).; castor clean:cleanup:workers:list — no stale QA workers.; git status clean; git diff --check origin/main...HEAD passed.
- Summary: Final reviewer decision: APPROVED. All prior findings resolved; replay-backed tmux test confirmed as the correct task-mandated user-flow layer. Final implementation commit is 2f634b46d044958ad4b8e44bdacec239b9e2f3d5. User authorized pushing.

## Task workflow update - 2026-08-15T02:20:24.492Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (120.2s).
- Pushed task/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output to origin.
- branch 'task/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output' set up to track 'origin/task/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output'.
- Created PR: https://github.com/ineersa/agent-core/pull/388

## Task workflow update - 2026-08-15T03:30:27.411Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output: ide_close_project returned isError.
- Merged task/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output into integration checkout.
- Merge made by the 'ort' strategy.
 .../task-workflow/src/TaskWorkflowExtension.php    |   2 +-
 .../task-workflow/src/Tool/ListTasksHandler.php    |  20 +-
 .../task-workflow/src/Tool/ToolResult.php          |  24 +-
 .../tests/TaskWorkflowHandlerToonOutputTest.php    |  16 +-
 .../task-workflow/tests/ToolResultTest.php         |  24 ++
 tests/Tui/E2E/TuiTaskListToolE2eTest.php           | 272 +++++++++++++++++++++
 .../Tui/E2E/fixtures/tui-task-list-tool-call.json  |  31 +++
 7 files changed, 360 insertions(+), 29 deletions(-)
 create mode 100644 tests/Tui/E2E/TuiTaskListToolE2eTest.php
 create mode 100644 tests/Tui/E2E/fixtures/tui-task-list-tool-call.json
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #388 confirmed merged on GitHub at 2026-08-15T03:29:52Z (merge commit 13aa02d09585ba64e710410afd4a1f4abf3dcb41). Moving to DONE and syncing integration main.

## Task workflow update - 2026-08-15T03:33:21.167Z
- Validation: PR #388 merged as 13aa02d09585ba64e710410afd4a1f4abf3dcb41.; Post-merge LLM_MODE=true castor check: deptrac, test (4481/17199), controller-replay (12/165), llm-real (13/144), phpstan, cs-check, docs:validate passed; test:tui failed 1 assertion in TuiTaskListToolE2eTest due terminal line wrapping inside the expected title.; QA leak check passed; llama-proxy cache guard remained 282→282.
- Summary: Post-merge `LLM_MODE=true castor check` failed only in the new TuiTaskListToolE2eTest: the expected title wrapped as `Demo\ntask from E2E` under the parallel TUI lane, making the contiguous substring assertion brittle. All other lanes passed; leak check and llama-proxy cache guard passed. QA report: var/reports/qa-20260815-033032-1017758-03816bbc. Task is merged and worktree cleanup completed, but this test flake needs a follow-up fix.

## Task workflow update - 2026-08-15T03:46:56.185Z
- Validation: castor test:tui --filter=TuiTaskListToolE2eTest — passed (1 test, 6 assertions).; castor test:tui — passed in parallel mode (37 tests, 296 assertions).; castor cs-check — passed.; LLM_MODE=true castor check — passed all lanes; TUI 37/296, unit 4476/17508, controller replay 12/165, llm-real 13/144; leak and cache guards passed. QA report: var/reports/qa-20260815-034439-1063032-8a67be51.; git push origin main — d39249c7e pushed.
- Summary: Direct-main follow-up completed at user request: moved the extension-specific task_list TUI E2E and fixture from core tests into `.hatfield/extensions/task-workflow/tests/Tui/`, following the observational-memory extension precedent, and made the title assertion tolerate terminal whitespace wrapping. Commit d39249c7e pushed to origin/main.

## Task workflow update - 2026-08-15T17:16:43.323Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
