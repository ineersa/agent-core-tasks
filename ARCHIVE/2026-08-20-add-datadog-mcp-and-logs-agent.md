# Add specific Datadog MCP and a datadog-logs specialist agent

## Goal
Add Datadog as a Hatfield MCP server with `availability: specific`, then add a dedicated `datadog-logs` subagent for Datadog observability work.

Ownership / prerequisite:

- User will connect/provision the Datadog MCP and provide the supported transport/auth configuration ("on me"). Do not invent an endpoint or commit credentials.
- Implementation begins once that connection information is available. Secrets must stay in environment variables or the MCP server's supported local authentication mechanism.

Requested behavior:

- Configure the Datadog MCP under a stable server name such as `datadog` with `availability: specific`, so its tools are hidden from the parent and unrelated agents.
- Add a discovered Hatfield agent named `datadog-logs` with a clear catalog description.
- Restrict the agent to the Datadog MCP via exact/prefix `mcp:` selectors and only the minimum non-MCP tools needed. Do not grant all MCP servers or broad implementation tools speculatively.
- The specialist should investigate Datadog logs, traces, metrics, monitors, dashboards, incidents, and related observability data; correlate evidence; and report concise findings with query/time-range context.
- When the connected Datadog MCP exposes write operations, the agent may perform explicitly requested Datadog edits. Its prompt must distinguish read-only investigation from mutating actions and require clear user authorization before creating, editing, muting, or deleting Datadog resources.
- Keep this configuration/agent-definition work minimal; use Hatfield's existing `availability: specific` and agent frontmatter mechanisms rather than adding product code or a custom Datadog integration.

Likely files after the user supplies connection details:

- `.hatfield/mcp.json`
- `.hatfield/agents/datadog-logs.md`
- Existing agent/MCP documentation only if validation shows a project-specific note is necessary.

## Acceptance criteria
- Datadog is configured as an MCP server with `availability: specific`; it is not exposed to the parent or unrelated subagents.
- No Datadog token, API key, application key, session cookie, or other credential is committed.
- A discovered `datadog-logs` agent opts into only the Datadog MCP using valid `mcp:` tool selectors and has the minimum necessary non-MCP tools.
- The agent prompt covers log/trace/metric/monitor/dashboard/incident investigation, evidence correlation, explicit time ranges and queries, and concise source-backed handoffs.
- Read-only investigation is the default; Datadog mutations require an explicit request/authorization and the agent does not silently edit, mute, or delete resources.
- Configuration and agent discovery/allowlisting are validated using existing Hatfield tests or the smallest appropriate Castor validation; no new production abstraction is introduced.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-20-add-datadog-mcp-and-logs-agent
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-datadog-mcp-and-logs-agent
Fork run: lwd4v633q16r
PR URL: https://github.com/ineersa/agent-core/pull/437
PR Status: merged
Started: 2026-08-27T23:31:57.207Z
Completed: 2026-08-28T02:15:00.036Z

## Work log
- Created: 2026-08-20T17:22:06+00:00

## Task workflow update - 2026-08-27T23:31:57.207Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-20-add-datadog-mcp-and-logs-agent.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-datadog-mcp-and-logs-agent.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-datadog-mcp-and-logs-agent.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-datadog-mcp-and-logs-agent.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-datadog-mcp-and-logs-agent.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-datadog-mcp-and-logs-agent.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-datadog-mcp-and-logs-agent/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-datadog-mcp-and-logs-agent.
- Summary: User supplied Datadog MCP transport and auth contract: HTTP endpoint `https://mcp.datadoghq.com/v1/mcp`; Authorization bearer value must come from environment variable `DATADOG_MCP_KEY`, with no credential committed. Scope expanded explicitly to configure both Hatfield and Pi, plus the Hatfield `datadog-logs` specialist agent.

## Task workflow update - 2026-08-27T23:37:04.070Z
- Validation: Read-only config/schema inspection: Hatfield supports `Authorization: Bearer ${DATADOG_MCP_KEY}` and `availability: specific`; Read-only Pi adapter inspection: Pi supports `auth: bearer` + `bearerTokenEnv: DATADOG_MCP_KEY`; no Hatfield-style availability field; Environment preflight: DATADOG_MCP_KEY — NOT SET in current orchestrator process
- Summary: Shallow routing confirmed safe config syntax but found two blockers before implementation: `DATADOG_MCP_KEY` is not present in the current orchestrator environment, so the required live connection/tool-catalog test cannot run; Hatfield explicit selectors currently start from globally available MCP tools, so `mcp:datadog_*` would also grant the specialist `jetbrains-index_*`, conflicting with the task's Datadog-only acceptance criterion. User also expanded scope to include a specialized Datadog usage skill in addition to the agent.
- Scout: role=scout; artifact_run_id=not-exposed-by-subagent-tool; revision=1d30c13521f0ad241f213ee8bbabda0b474e9523; scope=Hatfield/Pi Datadog MCP config syntax, env auth, selector isolation, agent discovery, and validation routing; outcome=completed; blockers=DATADOG_MCP_KEY absent from current process and Hatfield specific selectors inherit global jetbrains-index tools

## Task workflow update - 2026-08-27T23:44:35.074Z
- Validation: Environment preflight: DATADOG_MCP_KEY — SET (value not printed)
- Summary: User resolved both blockers: `DATADOG_MCP_KEY` is now visible to the orchestrator environment; the specialist may inherit globally available JetBrains MCP tools. Agent should have the same non-Datadog tools as scout, including bash, plus Datadog MCP tools. User explicitly requires live MCP connection/tool inspection before finalizing the specialist agent and Datadog usage skill.
- Decision: Hatfield datadog-logs may inherit globally available jetbrains-index MCP tools; strict Datadog-only isolation is no longer required. Non-MCP tool baseline must match scout and include bash. Live read-only Datadog MCP discovery is authorized; Datadog mutations remain forbidden unless explicitly requested.

## Task workflow update - 2026-08-27T23:45:35.339Z
- Recorded fork run: bobrna9iv8c1
- Ownership: owner=fork; fork_run=bobrna9iv8c1; revision=235c7a4f9ccb9a94682d90ee85011683184acdd4; scope=live read-only Datadog MCP discovery, Hatfield/Pi project config, datadog-logs agent, shared Datadog usage skill, focused validation, and commit; outcome=assigned; commit=none

## Task workflow update - 2026-08-27T23:52:12.486Z
- Validation: Read-only live tools/list size probe — 28 tools, 66,894 definition bytes, rough 16.7k–19.1k tokens; no tenant payload printed
- Summary: User explicitly removed Pi from scope after measuring the Datadog MCP catalog at 28 tools / roughly 17–19k tokens for all definitions. Final implementation must be Hatfield-only: no Datadog entry in `.pi/mcp.json`, no Pi-discoverable Datadog agent/skill, and no Pi runtime integration. The active fork was already running when this superseding decision arrived; its handoff must be checked and any Pi/shared-discovery changes removed before implementation is accepted.
- Decision: superseded prior both-runtime scope. Datadog MCP, datadog-logs agent, and Datadog skill are Hatfield-only. Remove/omit `.pi/mcp.json` Datadog configuration and avoid shared discovery paths that expose the specialist or skill to Pi.

## Task workflow update - 2026-08-27T23:54:58.119Z
- Recorded fork run: bobrna9iv8c1
- Validation: Live Datadog MCP initialize/tools/list/resources/prompts — PASS (28 tools, 134 resources, 0 prompts); Safe read-only list_datadog_skills call — PASS; Initial focused Castor test — PASS (98 tests, 293 assertions); Initial deptrac/phpstan/cs-check/docs:validate/git diff check — PASS
- Summary: Initial implementation fork completed as e957621f71b2de33790e244c7a0fb0a2aed6ff60 with successful live Datadog discovery and validation, but it includes Pi config/agent and a Pi-discoverable shared skill that were superseded while the fork was running. A corrective Hatfield-only slice is required before accepting implementation.
- Ownership: owner=fork; fork_run=bobrna9iv8c1; revision=235c7a4f9ccb9a94682d90ee85011683184acdd4; scope=live read-only Datadog MCP discovery, Hatfield/Pi project config, datadog-logs agent, shared Datadog usage skill, focused validation, and commit; outcome=completed; commit=e957621f71b2de33790e244c7a0fb0a2aed6ff60
- Handoff: e957621f includes superseded Pi-facing changes; do not treat as final. Preserve its Hatfield config, Hatfield agent, live discovery learnings, and concise skill content while removing Pi exposure.

## Task workflow update - 2026-08-27T23:55:21.190Z
- Recorded fork run: g1eywerra1co
- Ownership: owner=fork; fork_run=g1eywerra1co; revision=e957621f71b2de33790e244c7a0fb0a2aed6ff60; scope=remove all Pi/shared Datadog exposure, relocate skill to Hatfield-only discovery, retain validated Hatfield MCP/agent, and revalidate; outcome=assigned; commit=none

## Task workflow update - 2026-08-28T00:15:23.899Z
- Recorded fork run: g1eywerra1co
- Validation: castor test --filter='McpConfigLoaderTest|AgentMcpToolsResolverTest|AgentDefinitionDiscoveryTest|SkillDiscoveryTest' — PASS (98 tests, 293 assertions); Hatfield production discovery probe — PASS (agent + skill discovered; scout built-ins, global JetBrains, and datadog_* resolved; specific websearch excluded); castor deptrac — PASS (0 violations); castor phpstan — PASS (0 errors); castor cs-check — PASS; castor docs:validate — PASS (16 documents); JSON semantic validation — PASS (Pi only jetbrains-index; Hatfield specific Datadog); Secret/placeholder/Pi exposure scan — PASS; git diff --check — PASS
- Summary: Final Hatfield-only correction completed as f426397662b4fa7fb4495d59cfce5b36f77e2b20. Pi Datadog config/agent and shared skill exposure were removed; Hatfield retains the specific Datadog MCP, datadog-logs specialist, and Hatfield-local Datadog skill. Worktree reported clean.
- Ownership: owner=fork; fork_run=g1eywerra1co; revision=e957621f71b2de33790e244c7a0fb0a2aed6ff60; scope=remove all Pi/shared Datadog exposure, relocate skill to Hatfield-only discovery, retain validated Hatfield MCP/agent, and revalidate; outcome=completed; commit=f426397662b4fa7fb4495d59cfce5b36f77e2b20

## Task workflow update - 2026-08-28T00:23:35.129Z
- Summary: Independent review at f426397662b4fa7fb4495d59cfce5b36f77e2b20 returned APPROVE WITH SUGGESTIONS with no blockers. Reviewer verified secret-safe env interpolation, Hatfield discovery and tool resolution, absence of Pi/shared exposure, mutation safeguards, concise skill scope, existing lowest-layer proof, and clean cumulative diff. Optional suggestion to qualify runtime-prefixed tool examples is non-blocking and not required for this transition.
- Review: role=reviewer; artifact_run_id=not-exposed-by-subagent-tool; revision=f426397662b4fa7fb4495d59cfce5b36f77e2b20; scope=full Hatfield-only Datadog MCP/agent/skill review against finalized requirements; decision=APPROVE WITH SUGGESTIONS; blockers=none; suggestion=optionally qualify skill examples with runtime datadog_ prefix

## Task workflow update - 2026-08-28T00:27:03.902Z
- Validation: move_task CODE-REVIEW castor check — FAIL; Failing lane: test:controller-replay; Log: var/reports/qa-20260828-002346-778542-e0157c4d/check-test:controller-replay.log; castor clean:cleanup:workers:list — no stale QA worker candidates
- Summary: CODE-REVIEW transition blocked by deterministic gate failure in unrelated `ControllerReplayBashCancelFollowUpTest`: controller-replay lane collected only command.ack/run.started/turn.started and did not reach bash `tool_execution.started`; no stale QA workers remained. Per no-flake policy, task stays IN-PROGRESS for root-cause correction rather than retry/allowlist/timeout increase.
- Gate failure: revision=f426397662b4fa7fb4495d59cfce5b36f77e2b20; lane=test:controller-replay; test=ControllerReplayBashCancelFollowUpTest::testBashCancelThenFollowUpProducesAssistantResponse; evidence=tool start not observed under check contention; outcome=blocked pending root-cause fix

## Task workflow update - 2026-08-28T00:27:40.411Z
- Recorded fork run: lwd4v633q16r
- Ownership: owner=fork; fork_run=lwd4v633q16r; revision=f426397662b4fa7fb4495d59cfce5b36f77e2b20; scope=root-cause and eliminate ControllerReplayBashCancelFollowUpTest check-contention flake without timeout/retry/allowlist, preserve or deterministically remap regression proof, validate under normal castor check, and commit; outcome=assigned; commit=none

## Task workflow update - 2026-08-28T00:34:19.397Z
- Validation: castor test:controller-replay — PASS (6 tests, 88 assertions); Focused deterministic cancel/follow-up lower-layer tests — PASS (5 tests, 56 assertions); castor deptrac — PASS (0 violations); castor phpstan — PASS (0 errors); castor cs-check — PASS; git diff --check — PASS; LLM_MODE=1 castor check — PASS (170.1s, all lanes OK); Controller-replay JUnit max case time — 7.357s; corrected mapped tool-completion case — 6.429s; castor clean:cleanup:workers:list before/after — no stale QA worker candidates
- Summary: Fork lwd4v633q16r eliminated the unrelated check-contention flake by deleting the inherently timing-window controller journey and documenting deterministic proof mapping; no timeout/retry/allowlist and no product or Datadog changes. Correction committed as 09e33ed0988db57c20a9981b8365b3662fb316c2. Normal unweakened LLM_MODE=1 castor check passed with all controller-replay cases <=10s.
- Ownership: owner=fork; fork_run=lwd4v633q16r; revision=f426397662b4fa7fb4495d59cfce5b36f77e2b20; scope=root-cause and eliminate ControllerReplayBashCancelFollowUpTest check-contention flake without timeout/retry/allowlist, preserve or deterministically remap regression proof, validate under normal castor check, and commit; outcome=completed; commit=09e33ed0988db57c20a9981b8365b3662fb316c2

## Task workflow update - 2026-08-28T00:45:16.994Z
- Summary: Final cumulative re-review at 09e33ed0988db57c20a9981b8365b3662fb316c2 returned APPROVE WITH SUGGESTIONS with no blockers. Reviewer independently verified Hatfield-only Datadog semantics/security/discovery, complete Pi removal, no secret or dead paths, accurate deterministic proof mapping after deletion of the flaky controller journey, and <=10s controller-replay evidence. Suggestions and residual risks are non-blocking and add no finalized task requirement.
- Review: role=reviewer; artifact_run_id=not-exposed-by-subagent-tool; revision=09e33ed0988db57c20a9981b8365b3662fb316c2; scope=full cumulative Datadog task plus no-flake gate correction; decision=APPROVE WITH SUGGESTIONS; blockers=none; suggestions=runtime-prefixed tool examples, optional internal-tool exclusions/timeout, possible CancelHandler unit test, possible follow-up for graceful missing-env config-load handling

## Task workflow update - 2026-08-28T00:47:02.753Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (83.8s).
- Pushed task/2026-08-20-add-datadog-mcp-and-logs-agent to origin.
- branch 'task/2026-08-20-add-datadog-mcp-and-logs-agent' set up to track 'origin/task/2026-08-20-add-datadog-mcp-and-logs-agent'.
- Created PR: https://github.com/ineersa/agent-core/pull/437
- Validation: Final independent reviewer — APPROVE WITH SUGGESTIONS, no blockers; Live Datadog MCP initialize/tools/list/resources/prompts — PASS (28 tools, 134 resources, 0 prompts); Safe read-only list_datadog_skills call — PASS; Focused Datadog discovery/config/tool-resolution tests — PASS (98 tests, 293 assertions); castor test:controller-replay — PASS (6 tests, 88 assertions); Focused deterministic cancel/follow-up seam tests — PASS (5 tests, 56 assertions); Normal unweakened LLM_MODE=1 castor check — PASS (170.1s, all lanes OK); All controller-replay cases <=10s; max 7.357s; castor deptrac — PASS (0 violations); castor phpstan — PASS (0 errors); castor cs-check — PASS; castor docs:validate — PASS (16 documents); Secret/placeholder/Pi exposure scan — PASS; castor clean:cleanup:workers:list — no stale QA worker candidates; git diff --check — PASS
- Summary: Hatfield-only Datadog integration and deterministic no-flake gate correction are ready for CODE-REVIEW at 09e33ed0988db57c20a9981b8365b3662fb316c2. Final independent review found no blockers.

## Task workflow update - 2026-08-28T02:15:00.036Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-datadog-mcp-and-logs-agent: ide_close_project returned isError.
- Merged task/2026-08-20-add-datadog-mcp-and-logs-agent into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/agents/datadog-logs.md                   |  15 ++
 .hatfield/mcp.json                                 |   7 +
 .hatfield/skills/datadog/SKILL.md                  |  26 +++
 .../E2E/ControllerReplayBashCancelFollowUpTest.php | 195 ---------------------
 .../E2E/ControllerReplayToolCompletionTest.php     |  16 +-
 5 files changed, 59 insertions(+), 200 deletions(-)
 create mode 100644 .hatfield/agents/datadog-logs.md
 create mode 100644 .hatfield/skills/datadog/SKILL.md
 delete mode 100644 tests/CodingAgent/Runtime/Controller/E2E/ControllerReplayBashCancelFollowUpTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-datadog-mcp-and-logs-agent.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-datadog-mcp-and-logs-agent.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #437 state — MERGED; Merge commit — 8a4fa73df21453e2e4923d111cc584ef87e0bf0b
- Summary: PR #437 was merged on GitHub as 8a4fa73df21453e2e4923d111cc584ef87e0bf0b. Completing task lifecycle and synchronizing the integration checkout.

## Task workflow update - 2026-08-28T02:16:51.233Z
- Validation: Post-merge `LLM_MODE=true castor check` — PASS (159.9s); Unit lane — PASS (4914 tests, 20133 assertions); Controller replay — PASS (6 tests, 88 assertions); TUI replay — PASS (8 tests, 59 assertions); LLM real — PASS (5 tests, 30 assertions); Deptrac/phpstan/cs-check/docs/catalog/PHAR smoke — PASS; QA artifact integrity and run leak check — PASS; llama-proxy cache guard — PASS (391 → 391); Post-validation worker diagnostics — no stale QA worker candidates; Integration checkout — clean; main contains origin/main and is ahead by expected local task/integration merge commits; Task worktree — removed
- Summary: Post-merge integration validation completed successfully. PR #437 is merged; integration checkout is synchronized and clean; task worktree removed. JetBrains project close degraded non-fatally before filesystem cleanup, which completed.

## Task workflow update - 2026-08-29T16:09:38.962Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
