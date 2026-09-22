# Rename Datadog agent and hide its skill from model discovery

## Goal
Rename the project Datadog specialist agent from `datadog-logs` to `datadog`. Mark the `datadog` skill as on-demand-only while keeping it explicitly attached to the specialist agent.

## Acceptance criteria
- The specialist agent is discovered and launched as `datadog`, with no remaining project agent definition named `datadog-logs`.
- The `datadog` skill has `disable-model-invocation: true`, so it is omitted from general model discovery but remains explicitly loadable.
- The renamed `datadog` agent still preloads the `datadog` skill.
- Relevant documentation and focused Castor validation pass.

## Workflow metadata
Status: DONE
Branch: task/2026-09-01-rename-datadog-agent-and-hide-skill
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-24-preserve-codex-reasoning-summary-in-tui-worktrees/2026-09-01-rename-datadog-agent-and-hide-skill
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/472
PR Status: merged
Started: 2026-09-01T23:15:53+00:00
Completed: 2026-09-06T17:55:37+00:00

## Work log
- Created: 2026-09-01T23:15:36+00:00

## Task workflow update - 2026-09-01T23:15:53+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-01-rename-datadog-agent-and-hide-skill.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-24-preserve-codex-reasoning-summary-in-tui-worktrees/2026-09-01-rename-datadog-agent-and-hide-skill.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-24-preserve-codex-reasoning-summary-in-tui-worktrees/2026-09-01-rename-datadog-agent-and-hide-skill.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-24-preserve-codex-reasoning-summary-in-tui-worktrees/2026-09-01-rename-datadog-agent-and-hide-skill.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-24-preserve-codex-reasoning-summary-in-tui-worktrees/2026-09-01-rename-datadog-agent-and-hide-skill.
- Parent IDEA module not found or ambiguous; skipping exclusion update.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-24-preserve-codex-reasoning-summary-in-tui-worktrees/2026-09-01-rename-datadog-agent-and-hide-skill/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-24-preserve-codex-reasoning-summary-in-tui-worktrees/2026-09-01-rename-datadog-agent-and-hide-skill.
- Summary: Starting implementation. Routing pass found one cohesive configuration/docs change: rename `.hatfield/agents/datadog-logs.md`, update its frontmatter name, and add `disable-model-invocation: true` to `.hatfield/skills/datadog/SKILL.md`. Main owns the slice.

## Task workflow update - 2026-09-01T23:16:08+00:00
- Ownership: owner=main; fork_run=none; revision=task/2026-09-01-rename-datadog-agent-and-hide-skill baseline; scope=rename the project Datadog agent, mark its skill on-demand-only, and update directly affected docs; outcome=assigned; commit=none

## Task workflow update - 2026-09-01T23:18:40+00:00
- Validation: `castor test --filter='AgentDefinitionDiscoveryTest|SkillDiscoveryTest|SkillsContextBuilderTest|AgentsContextDiscoveryTest'` — 65 tests, 193 assertions, passed.; `castor docs:validate` — passed.; `git diff --check` — passed.
- Summary: Renamed `.hatfield/agents/datadog-logs.md` to `.hatfield/agents/datadog.md`, changed the launch key to `datadog`, and marked the attached `datadog` skill with `disable-model-invocation: true`. The skill remains explicitly attached through the agent's `skills` list.
- Ownership: owner=main; fork_run=none; revision=task/2026-09-01-rename-datadog-agent-and-hide-skill; scope=rename the project Datadog agent, mark its skill on-demand-only, and update directly affected docs; outcome=completed; commit=a11dbf325

## Task workflow update - 2026-09-01T23:30:25+00:00
- Research: role=researcher; artifact=agent_b7880b0774faad36; revision=a11dbf325; scope=official Datadog MCP tools for dashboard/widget, monitor, notebook, incident, and telemetry capabilities; outcome=completed

## Task workflow update - 2026-09-01T23:38:41+00:00
- Validation: `python3 -m json.tool .hatfield/mcp.json` — valid JSON.; `git diff --check` — passed.
- Summary: Expanded the Datadog MCP endpoint to request the `core`, `alerting`, `dashboards`, and `widgets` server-side toolsets. This exposes dashboard upsert, widget schema/validation, monitor creation/validation, notebook writes, and widget inspection tools to the existing `mcp:datadog_*` agent selector.
- Ownership: owner=main; fork_run=none; revision=a11dbf325; scope=enable Datadog MCP server toolsets for dashboards, widgets, alerting, and core investigation; outcome=completed; commit=b2ade7f93

## Task workflow update - 2026-09-01T23:43:44+00:00
- Datadog trial: role=datadog-logs; artifact=agent_b470607413f1d42b; revision=b2ade7f93; scope=append one Hatfield warning-and-higher log widget to dashboard xza-j7r-e4s and collect durable Hatfield observability facts; outcome=blocked because the current session retained the pre-change MCP catalog and exposed no dashboard upsert/widget validation tools; mutation=none
- Ownership: owner=main; fork_run=none; revision=b2ade7f93; scope=rewrite the Datadog skill around confirmed Hatfield dashboard, log facets/query, trace correlation, and safe dashboard mutation procedure; outcome=assigned; commit=none

## Task workflow update - 2026-09-01T23:45:06+00:00
- Validation: `castor docs:validate` — passed after the skill rewrite.; `git diff --check` — passed.
- Summary: Rewrote the on-demand Datadog skill for Hatfield. It now records dashboard `xza-j7r-e4s`, stable service/env/source identifiers, the verified warning-and-higher query, log-to-trace correlation steps, safe dashboard/widget upsert and verification steps, other supported resource mutations, MCP toolset requirements, and precise handoff evidence.
- Ownership: owner=main; fork_run=none; revision=b2ade7f93; scope=rewrite the Datadog skill around confirmed Hatfield dashboard, log facets/query, trace correlation, and safe dashboard mutation procedure; outcome=completed; commit=1618416e8

## Task workflow update - 2026-09-06T15:57:31+00:00
- Summary: Same-status move_task did not recover the missing worktree. Recreated the registered missing worktree with git worktree add --force, retaining the task branch and commit history. Current main already contains the MCP toolsets and newer Datadog reference documentation; preserve those while finishing rename/hiding.
- Ownership: owner=main; fork_run=none; revision=1618416e8; scope=recover missing worktree, reconcile existing Datadog work with current main, and complete review/delivery; outcome=assigned; commit=none

## Task workflow update - 2026-09-06T16:04:00+00:00
- Validation: Reviewer ran focused discovery checks at 75d5b0952: 65 tests, 193 assertions passed.; Reviewer ran castor docs:validate: passed.
- Summary: Reconciled task with current main at 75d5b0952. Remaining diff only renames agent and hides its preloaded skill. Independent reviewer APPROVE WITH SUGGESTIONS; optional illustrative documentation naming cleanup deferred.
- Review: role=reviewer; artifact=agent_a476ea14af25d839; revision=75d5b0952; scope=specification fidelity, discovery contracts, retained current-main documentation, focused validation; outcome=APPROVE WITH SUGGESTIONS
- Ownership: owner=main; fork_run=none; revision=75d5b0952; scope=recover missing worktree and reconcile Datadog work with current main; outcome=completed; commit=75d5b0952

## Task workflow update - 2026-09-06T16:06:42+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (141.9s).
- Pushed task/2026-09-01-rename-datadog-agent-and-hide-skill to origin.
- branch 'task/2026-09-01-rename-datadog-agent-and-hide-skill' set up to track 'origin/task/2026-09-01-rename-datadog-agent-and-hide-skill'.
- Created PR: https://github.com/ineersa/agent-core/pull/472

## Task workflow update - 2026-09-06T17:55:37+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-24-preserve-codex-reasoning-summary-in-tui-worktrees/2026-09-01-rename-datadog-agent-and-hide-skill: ide_close_project returned isError.
- Merged task/2026-09-01-rename-datadog-agent-and-hide-skill into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/agents/{datadog-logs.md => datadog.md} | 2 +-
 .hatfield/skills/datadog/SKILL.md                | 1 +
 2 files changed, 2 insertions(+), 1 deletion(-)
 rename .hatfield/agents/{datadog-logs.md => datadog.md} (97%)
- Worktree cleanup failed: fatal: validation failed, cannot remove working tree: '/home/ineersa/projects/agent-core-worktrees/2026-08-24-preserve-codex-reasoning-summary-in-tui-worktrees/2026-09-01-rename-datadog-agent-and-hide-skill' does not point back to '.git/worktrees/2026-09-01-rename-datadog-agent-and-hide-skill'
- IDEA exclusions preserved for /home/ineersa/projects/agent-core-worktrees/2026-08-24-preserve-codex-reasoning-summary-in-tui-worktrees/2026-09-01-rename-datadog-agent-and-hide-skill because worktree removal failed.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-24-preserve-codex-reasoning-summary-in-tui-worktrees/2026-09-01-rename-datadog-agent-and-hide-skill.
- Reopened JetBrains project after failed worktree cleanup for /home/ineersa/projects/agent-core-worktrees/2026-08-24-preserve-codex-reasoning-summary-in-tui-worktrees/2026-09-01-rename-datadog-agent-and-hide-skill.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: Confirmed PR #472 merged on GitHub at 918fce89ef5c8c96a881667c441b2cdf58376d86. Completing integration checkout update and worktree cleanup.

## Task workflow update - 2026-09-06T17:58:16+00:00
- Updated PR Status: merged
- Validation: LLM_MODE=true castor check failed: test lane timeout after 120s. Reports: var/reports/qa-20260906-175544-6004-2ed2cb4b. check-test.log contains ParaTest startup only, no failing case identified.; Git status --short: clean. Task worktree removal failed; duplicate registrations observed.
- Summary: Moved to DONE after confirmed GitHub merge. Integration checkout is clean at 94722f2e2. Post-merge validation INCOMPLETE: unit test lane timed out at 120s; all other lanes passed, no QA-owned leaks, cache guard passed. No blind retry. Worktree cleanup INCOMPLETE: Git back-pointer validation failed; git worktree list shows duplicate registrations for the recovered task path. Worktree retained; requires targeted registration repair.
