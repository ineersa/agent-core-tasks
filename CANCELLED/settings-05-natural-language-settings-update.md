# SETTINGS-05: Add agent-assisted /settings-update command

## Goal
Add `/settings-update <request>` as a built-in slash command that dispatches a purpose-built agent turn. The prompt instructs Hatfield to use the internal documentation tool for reference and the singular settings tool for reading and mutations. If the request does not identify user or project scope, or is otherwise ambiguous, the agent must use `ask_human` before changing anything. It must not edit raw settings YAML through generic file tools.

This task also provides the user-facing one-time migration path for legacy full settings snapshots: Hatfield can inspect the chosen layer and remove inherited/default entries one at a time while preserving intentional differing/unknown overrides.

Dependencies: SETTINGS-02, SETTINGS-03, SETTINGS-04.

## Acceptance criteria
- `/settings-update <natural-language request>` dispatches an LLM-visible instruction through the normal runtime path.
- The instruction requires `documentation` for internal reference, `settings(read)` for inspection, and singular `settings(set|remove)` calls for changes; raw settings YAML editing is explicitly prohibited.
- Missing target scope and ambiguous requests cause `ask_human` clarification before mutation.
- Each mutation remains independently subject to SafeGuard confirmation from SETTINGS-03.
- The command reports that disk changes require restart and does not claim the boot-time configuration was hot-reloaded.
- The flow supports guided cleanup of legacy complete settings snapshots without silently deciding whether differing values are stale or intentional.
- The real TUI slash-command routing is proven at the virtual layer, and the LLM-visible prompt/tool flow receives focused live-LLM validation plus the required full Castor gate before review.

## Workflow metadata
Status: CANCELLED
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed: 2026-07-18T22:41:38.159Z

## Work log
- Created: 2026-07-16T17:35:03.663Z

## Task workflow update - 2026-07-17T18:06:32.603Z
- MANDATORY implementation constraints: REUSE SYMFONY COMPONENTS and existing slash-command/runtime/tool paths; do not invent a parallel prompt, routing, or settings-update framework. DO NOT OVERENGINEER; choose the simplest viable composition of existing documentation/settings/ask_human/SafeGuard capabilities. MINIMIZE CHANGES AND TESTS: no unrelated renames/refactors/churn; add only the smallest lowest-layer proof for command dispatch and mutation guidance.

## Task workflow update - 2026-07-18T22:41:38.159Z
- Status set to CANCELLED by user.
- Reason: the agent already uses the settings tool effectively; a dedicated `/settings-update` wrapper is unnecessary and would add tricky/duplicated command semantics. If a concrete settings discoverability or mutation UX gap appears later, create a focused task for that gap instead.
