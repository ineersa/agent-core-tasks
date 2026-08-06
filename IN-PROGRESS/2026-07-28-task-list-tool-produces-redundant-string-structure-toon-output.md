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
Status: IN-PROGRESS
Branch: task/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-28-task-list-tool-produces-redundant-string-structure-toon-output
Fork run:
PR URL:
PR Status:
Started: 2026-07-31T19:57:32+00:00
Completed:

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
