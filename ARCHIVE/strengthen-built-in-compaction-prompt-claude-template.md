# Strengthen built-in compaction prompt from Claude Code template

## Goal
Adapt the built-in Hatfield compaction prompt using the stronger structure from Claude Code, without adding product- or extension-specific workflow content.

Reference sources:
- Claude Code prompt: `/home/ineersa/claw/claude-code/src/services/compact/prompt.ts`
- Current Hatfield built-in prompt: `/home/ineersa/projects/agent-core/config/COMPACTION.md`
- Prompt loader/precedence: `/home/ineersa/projects/agent-core/src/CodingAgent/Compaction/CompactionPromptBuilder.php`
- Current docs: `/home/ineersa/projects/agent-core/docs/compaction.md`
- Prior Hatfield compaction design task: `/home/ineersa/projects/agent-core-tasks/ARCHIVE/comp-01-compactor-service-settings-prompt.md`

Intent:
- Bring over the useful Claude Code compaction prompt shape: strong no-tools/text-only instruction, detailed handoff sections, preservation of user intent, current work, errors/fixes, files/code references, pending tasks, and next step.
- Keep the built-in prompt generic to Hatfield core. Do NOT mention Pi, task board status, Castor validation, AGENTS instructions, worktrees, PRs, or extension-specific workflow concepts. Those can be custom prompt overrides/hooks/extensions, not built-in defaults.
- Preserve existing template mechanics and placeholders unless there is a focused reason to change them.

## Acceptance criteria
- `config/COMPACTION.md` is updated to a stronger generic Hatfield handoff prompt inspired by Claude Code's structure.
- The built-in prompt explicitly instructs the summarizer to output text only and not call tools.
- The prompt includes generic sections for user intent, key context/decisions, files/code/errors, current work, pending tasks, and next step, while avoiding Pi/task-board/Castor/AGENTS/worktree/PR/extension-specific content.
- Existing template placeholder behavior remains compatible with `CompactionPromptBuilder` (`{custom_instructions_part}`, `{date}`, `{cwd}` unless deliberately changed with tests/docs).
- Docs and focused tests are updated only if needed to reflect the changed built-in template behavior.
- Validation is run through Castor only, with at least the focused relevant test/docs check and broader validation appropriate for a prompt-template-only change.

## Workflow metadata
Status: DONE
Branch: task/strengthen-built-in-compaction-prompt-claude-template
Worktree: /home/ineersa/projects/agent-core-worktrees/strengthen-built-in-compaction-prompt-claude-template
Fork run: q63u4mihipvi
PR URL: https://github.com/ineersa/agent-core/pull/270
PR Status: merged
Started: 2026-07-09T02:28:25.938Z
Completed: 2026-07-09T02:43:12.599Z

## Work log
- Created: 2026-07-09T02:23:52.964Z

## Task workflow update - 2026-07-09T02:28:25.938Z
- Moved TODO → IN-PROGRESS.
- Created branch task/strengthen-built-in-compaction-prompt-claude-template.
- Created worktree /home/ineersa/projects/agent-core-worktrees/strengthen-built-in-compaction-prompt-claude-template.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/strengthen-built-in-compaction-prompt-claude-template.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/strengthen-built-in-compaction-prompt-claude-template.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/strengthen-built-in-compaction-prompt-claude-template.
- Summary: Starting implementation per user request. Goal: adapt Claude Code's stronger compaction prompt structure into Hatfield's built-in COMPACTION.md while keeping it generic and avoiding Pi/task-board/Castor/AGENTS/worktree/PR/extension-specific workflow content.

## Task workflow update - 2026-07-09T02:30:07.497Z
- Recorded fork run: hncfzlkkhywh
- Validation: Fork read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before QA.; `castor cs-check` in worktree: pass, 0 fixable issues.; `castor test --filter=SessionCompactorTest` in worktree: pass, 28 tests, 273 assertions.
- Summary: Implementation fork completed. Updated the built-in Hatfield compaction prompt from Claude Code's stronger structure while keeping it generic and excluding Pi/task-board/Castor/AGENTS/worktree/PR/extension workflow content. Docs were aligned by removing the stale `{summary_prefix}` placeholder row and clarifying the runtime wrapper lives in `SessionCompactor`. Commit: `5f6e470c9`. Files changed: `config/COMPACTION.md`, `docs/compaction.md`.

## Task workflow update - 2026-07-09T02:36:04.878Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (109.1s).
- Pushed task/strengthen-built-in-compaction-prompt-claude-template to origin.
- branch 'task/strengthen-built-in-compaction-prompt-claude-template' set up to track 'origin/task/strengthen-built-in-compaction-prompt-claude-template'.
- Created PR: https://github.com/ineersa/agent-core/pull/270
- Validation: Parent read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before QA.; Parent `git diff --stat main...HEAD`: only `config/COMPACTION.md` and `docs/compaction.md`, 23 insertions/13 deletions.; Parent `castor cs-check`: pass, files_fixed=0.; Parent `castor test --filter=SessionCompactorTest`: pass, 28 tests, 273 assertions.
- Summary: Preparing PR. Parent inspected the task worktree: branch clean; `main...HEAD` contains only `config/COMPACTION.md` and `docs/compaction.md`. The prompt remains generic Hatfield core and excludes Pi/task-board/Castor/AGENTS/worktree/PR/extension workflow content. Existing placeholders remain `{custom_instructions_part}`, `{date}`, and `{cwd}`. A reviewer subagent invocation returned no final artifact/output in this session, so parent performed focused review against the acceptance criteria before PR creation.

## Task workflow update - 2026-07-09T02:36:15.062Z
- Recorded fork run: q63u4mihipvi
- Updated PR URL: https://github.com/ineersa/agent-core/pull/270
- Updated PR Status: open
- Validation: Reviewer fork `q63u4mihipvi`: APPROVED, no blocking findings.; `move_task` CODE-REVIEW deterministic `castor check`: passed in 109.1s.
- Summary: Readonly review completed after PR creation transition. Reviewer verdict: APPROVED. Reviewer confirmed the branch only changes `config/COMPACTION.md` and `docs/compaction.md`, excludes forbidden workflow/product-extension concepts, preserves `{custom_instructions_part}`, `{date}`, `{cwd}`, removes phantom `{summary_prefix}` docs, and correctly distinguishes the prompt template from `SessionCompactor` runtime wrapping.

## Task workflow update - 2026-07-09T02:43:12.600Z
- Moved CODE-REVIEW → DONE.
- Merged task/strengthen-built-in-compaction-prompt-claude-template into integration checkout.
- Merge made by the 'ort' strategy.
 config/COMPACTION.md | 31 +++++++++++++++++++++----------
 docs/compaction.md   |  5 ++---
 2 files changed, 23 insertions(+), 13 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/strengthen-built-in-compaction-prompt-claude-template.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/strengthen-built-in-compaction-prompt-claude-template.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User reported PR #270 was merged. Moving task to DONE and merging/syncing integration checkout.

## Task workflow update - 2026-07-09T02:45:06.683Z
- Updated PR Status: merged
- Validation: `LLM_MODE=true castor check`: pass in 226.9s. Lanes OK: deptrac, test (4207 tests, 13697 assertions), test:controller-replay (8 tests, 112 assertions), test:tui (32 tests, 163 assertions), test:llm-real (10 tests, 121 assertions), phpstan, cs-check. llama-proxy cache guard OK (157 → 157). QA artifact integrity OK. QA run leak check OK.
- Summary: Post-merge validation completed on integration checkout.
