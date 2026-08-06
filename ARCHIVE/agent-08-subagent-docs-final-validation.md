# AGENT-08 Subagent docs, examples, and final product validation

## Goal
Final production polish task after AGENT-04 through AGENT-07.

Goal: document and validate the v1 subagent product end-to-end: user-defined agent definitions, foreground `subagent` tool behavior, inline progress, retrieval, parallel execution, limitations, settings, and safety model.

Scope:
- Update `docs/agents.md`, `docs/settings.md`, and relevant prompt/tool docs.
- Ensure `.hatfield/settings.yaml` stays in sync if new settings were added.
- Provide examples for project/user-defined agents under `.agents/` / `.hatfield/agents/` without bundling built-in definitions.
- Document v1 exclusions: no steering, no child HITL questions, no nested approvals, no interactive child conversations, no async/background subagents, no dock/view/overlay.
- Document troubleshooting and retrieval flows for failed/needs-clarification runs.
- Add final product-level validation coverage only where it protects user-visible behavior or stable contracts.

Out of scope:
- New runtime features beyond documentation/test fixes needed to complete v1.

## Acceptance criteria
- Docs explain agent definition discovery/precedence, settings, `subagent`, `agent_retrieve`, inline progress, parallel mode, artifacts, and retrieval.
- Docs explicitly state that agent names such as scout/reviewer/researcher/worker are user/project examples, not bundled built-ins.
- Docs clearly describe v1 limitations and the needs-clarification/failed artifact pattern.
- Settings docs and `.hatfield/settings.yaml` are synchronized for any new keys.
- At least one product-level validation path proves foreground subagent run + retrieval behavior; TUI-visible inline rendering has real TmuxHarness E2E proof if not already covered by earlier tasks.
- No new compatibility/fallback layers, built-in agent definitions, `/agents` dock/view, or async/background launch are introduced.
- Full deterministic validation passes via `castor check`; include focused `castor test:tui` if TUI snapshots/rendering changed.

## Workflow metadata
Status: DONE
Branch: task/agent-08-subagent-docs-final-validation
Worktree: /home/ineersa/projects/agent-core-worktrees/agent-08-subagent-docs-final-validation
Fork run: da6z4pphxvp4
PR URL: https://github.com/ineersa/agent-core/pull/219
PR Status: merged
Started: 2026-06-25T02:17:06.896Z
Completed: 2026-06-25T21:18:20.642Z

## Work log
- Created: 2026-06-22T19:04:51.803Z

## Task workflow update - 2026-06-25T02:17:06.896Z
- Moved TODO → IN-PROGRESS.
- Created branch task/agent-08-subagent-docs-final-validation.
- Created worktree /home/ineersa/projects/agent-core-worktrees/agent-08-subagent-docs-final-validation.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/agent-08-subagent-docs-final-validation.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/agent-08-subagent-docs-final-validation.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/agent-08-subagent-docs-final-validation.
- Summary: User requested starting AGENT-08 so they can test it in the worktree. Moving task to IN-PROGRESS and creating task branch/worktree for manual testing.

## Task workflow update - 2026-06-25T02:31:19.405Z
- Recorded fork run: 2kgsobhpmtyj
- Summary: Launched implementation fork for user-reported flaw: add <available_agents> context channel analogous to <available_skills>, sourced from enabled agent definitions frontmatter name/description, plus system prompt/context-channel instructions. Explicitly instructed fork not to touch project .pi files, to avoid exposing full agent instructions, to add focused tests, and to commit locally only.

## Task workflow update - 2026-06-25T02:35:27.834Z
- Recorded fork run: 2kgsobhpmtyj
- Validation: Parent loaded testing skill and read tests/AGENTS.md before rerunning validation.; castor test --filter='AgentContextRendererTest|AgentsContextBuilderTest|SystemPromptBuilderTest' OK (28 tests, 85 assertions); castor phpstan OK (0 errors); castor deptrac OK (0 violations); castor cs-check OK (0 files to fix); Inspected rendered snippets: config/SYSTEM.md now says available agent definitions may appear in user-context messages with <available_agents>; renderer emits names/descriptions only and no full instructions; runtime injects metadata source=agents_definitions_context.; Worktree status: branch task/agent-08-subagent-docs-final-validation at 14c3c868f with untracked hatfield-session-1.html only.
- Summary: Implementation fork completed at commit 14c3c868f: adds an <available_agents> user-context channel for parent runs. New AgentContextRenderer renders <agents_instructions>/<available_agents> with enabled, foreground-launchable agent names and descriptions only; AgentsContextBuilder gates on agents.enabled and filters catalog.enabled() by foregroundAllowed; InProcessAgentSessionClient injects the rendered block after skills_context with metadata source=agents_definitions_context. config/SYSTEM.md documents the <available_agents> context channel; docs/agents.md updated; depfile.yaml allows AppRuntimeInternals -> AppAgent for the new runtime injection. Note: fork used raw vendor/bin validation, which was not accepted; parent reran focused validation through Castor successfully. Worktree has untracked user artifact hatfield-session-1.html left untouched.

## Task workflow update - 2026-06-25T02:50:12.769Z
- Recorded fork run: 9l3ocltblmgn
- Summary: Launched fix fork for live <available_agents> missing bug. Scout determined runtime injection path is wired, but AgentDefinitionCatalog is empty because existing ~/.agents/*.md frontmatter is rejected by AgentDefinitionParser: comma-separated string tools, singular/string skill(s), and missing tools in some agents. Fork instructed to implement parser normalization/defaults, add focused parser/catalog/context/session-start tests, use Castor only, avoid project .pi and avoid editing user ~/.agents files, commit locally only.

## Task workflow update - 2026-06-25T02:53:41.013Z
- Recorded fork run: 9l3ocltblmgn
- Validation: Fork attempted mandatory reads; testing skill/tests AGENTS reads were workspace-blocked in fork, but parent previously loaded testing skill/tests AGENTS and fork used Castor validation as authoritative.; castor test --filter='AgentDefinitionParserTest\|AgentDefinitionDiscoveryTest\|AgentsContextBuilderTest\|AgentContextRendererTest' OK (101 tests, 240 assertions); castor phpstan OK (0 errors); castor deptrac OK (0 violations); castor cs-check OK after cs-fix on parser; Manual discovery + AgentsContextBuilder against real HOME: enabled count=5 and rendered <available_agents> includes architect, browser, researcher, reviewer, scout with names/descriptions only.
- Summary: Fix fork completed at commit b7956bcc5. Root cause confirmed: AgentsContextBuilder was wired, but AgentDefinitionCatalog was empty because real ~/.agents/*.md frontmatter did not parse under strict AgentDefinitionParser rules (comma-separated tools strings, singular skill, string skills, missing tools). Fix normalizes observed frontmatter shapes before denormalization: string tools -> list, string skills -> list, singular string skill merged into skills, missing tools defaults to ['read']. This allows real agents architect/browser/researcher/reviewer/scout to be discovered and rendered in <available_agents>. InProcessAgentSessionClient injection remains unchanged from prior commit. Worktree left with untracked hatfield-session-1.html and hatfield-session-3.html user/debug exports.

## Task workflow update - 2026-06-25T03:12:23.336Z
- Recorded fork run: a8k7kylblxcp
- Summary: Launched fix fork for two AGENT-08 issues found during smoke testing: (1) SafeGuard disabled because project .hatfield/settings.yaml extensions.enabled replaced defaults and listed only TaskWorkflowExtension; fork to enable both SafeGuardExtension and TaskWorkflowExtension and add minimal docs/test if useful. (2) parallel subagent default incorrect: existing agents without parallelAllowed (e.g. scout) are denied in parallel mode; product expectation is omitted parallelAllowed defaults to true, only explicit false disallows. Fork instructed to implement minimal code/config fixes with focused Castor validation, commit locally only, no push/task move.

## Task workflow update - 2026-06-25T03:16:30.047Z
- User added AGENT-08 requirement: add a project-level Hatfield skill for subagents under tracked `.hatfield` (not `.pi`). The skill should serve as user/model-facing docs for subagent usage and agent definition frontmatter, including fields like name, description, tools, skills/skill alias, model, thinking, inheritProjectContext, systemPromptMode, maxDepth, foregroundAllowed, backgroundAllowed, parallelAllowed default/explicit opt-out, disabled, handoffFormat, MCP policy, and examples for single/parallel subagent usage plus agent_retrieve. This should be included in AGENT-08 docs/final validation scope before task-to-PR.

## Task workflow update - 2026-06-25T16:09:35.052Z
- Recorded fork run: ar282jq3e0qg
- Summary: Launched implementation fork to add tracked project-level Hatfield subagents skill under `.hatfield` (not `.pi`), documenting subagent usage and agent definition frontmatter/defaults/examples. Fork also instructed to absorb existing parser style-only dirty change from cs-fix, leave debug HTML exports untracked, validate with Castor, commit locally only, no push/task move.

## Task workflow update - 2026-06-25T16:11:06.205Z
- Recorded fork run: ar282jq3e0qg
- Validation: Mandatory reads performed by fork: root AGENTS.md, testing skill, tests/AGENTS.md, write-a-skill skill, castor skill, SkillDiscovery.; castor test --filter=SkillDiscoveryTest OK (14 tests, 38 assertions); castor cs-check OK; castor phpstan OK; Full castor check not run per task scope.
- Summary: Fork completed at commit cb4db50be docs(agents): add project subagents skill. Added tracked project-level Hatfield skill under `.hatfield/skills/subagents/` (SKILL.md + FRONTMATTER.md), updated docs/agents.md with project skill pointer and corrected limitations table, and included parser whitespace CS cleanup. Skill documents subagent usage, agent definition frontmatter/defaults, SafeGuard/extensions.enabled list-replacement warning, single/parallel subagent examples, MCP policy, agent_retrieve, and current inline progress limitation pending structured widget work. SkillDiscovery confirms `.hatfield/skills/**/SKILL.md` is auto-discovered from project CWD. Worktree clean except untracked hatfield-session-*.html debug exports.

## Task workflow update - 2026-06-25T16:14:41.534Z
- Recorded fork run: 4yn0nx19twz7
- Summary: Launched tiny docs-correction fork after user noted the new subagents skill must be truthful. Parent spotted `.hatfield/skills/subagents/FRONTMATTER.md` says `tools` is Required, but implementation defaults missing tools to `['read']`. Fork instructed to fix that wording, audit the two new skill docs for other clear default/behavior drift, run castor cs-check, commit locally, leave debug HTML exports alone.

## Task workflow update - 2026-06-25T16:15:33.902Z
- Recorded fork run: 4yn0nx19twz7
- Validation: castor cs-check OK (docs-only, no PHP/tests touched)
- Summary: Docs correction fork completed at commit 80c95be76 docs(agents): correct subagent skill frontmatter defaults. Fixed `.hatfield/skills/subagents/FRONTMATTER.md` and `SKILL.md` so `tools` is documented as optional in YAML, defaulting to `['read']` when omitted, with explicit non-empty allowlist recommended and explicit `[]`/blank entries invalid. Audited other defaults against parser/DTO/tests; no other skill-doc drift found. Worktree clean except untracked hatfield-session-*.html debug exports.

## Task workflow update - 2026-06-25T16:15:52.725Z
- Recorded fork run: tyxk4vnwvsi1
- Summary: Launched tiny docs truth-sync fork after verifying canonical `docs/agents.md` still said `tools` was required with no default, contradicting parser behavior and the new `.hatfield/skills/subagents/FRONTMATTER.md`. Fork instructed to update only the `tools` row/nearby wording to optional/default `['read']`, run castor cs-check, commit locally, leave debug HTML exports alone.

## Task workflow update - 2026-06-25T16:16:21.214Z
- Recorded fork run: tyxk4vnwvsi1
- Validation: castor cs-check OK (0 files to fix)
- Summary: Canonical docs truth-sync fork completed at commit ed93861b3 docs(agents): sync tools frontmatter default. Updated `docs/agents.md` frontmatter table so `tools` is Required=no, Default=`['read']`, and description says omitted tools default to read-only, explicit non-empty allowlist is recommended, YAML list preferred, comma-separated strings normalized, and empty/blank explicit values invalid. No PHP/skill file changes. Worktree clean except untracked hatfield-session-*.html debug exports.

## Task workflow update - 2026-06-25T16:25:48.196Z
- Summary: User smoke-tested AGENT-08 worktree session 5: parallel subagent launch appears to work and cancellation was exercised. Read-only scout investigation confirmed cancelled scout child was actually cancelled: parent session `.hatfield/sessions/5/events.jsonl` shows subagent tool cancellation result at 16:18:19; child artifact `agent_80d4aa7c5bcd1fcd` has `events.jsonl` with `llm_step_aborted` stop_reason `aborted` and `agent_end` reason `cancelled`, registry status `cancelled`, state status `cancelled`, errorMessage `Parent run cancelled parallel subagent tool.`, pendingToolCalls 0, and no stale child subagent processes remain. Artifacts were not literally lost: `events.jsonl`, `state.json`, `metadata.json`, and `handoff.md` persisted; perceived loss is UX/model behavior because cancelled handoff only contains status/summary (`Cancelled by parent run.`) and parent LLM did not call `agent_retrieve` for the cancelled artifact when asked. Potential follow-up: improve cancelled artifact handoff/UX to expose partial context or make cancelled artifacts easier to retrieve.

## Task workflow update - 2026-06-25T16:33:27.000Z
- Summary: Read-only scout investigated user concern that omitted agent `tools` defaulting to `['read']` may not match plan/pi reference. Finding: current Hatfield behavior matches neither source. Current code: `AgentDefinitionParser::normalizeFrontmatterFields()` injects `['read']` when `tools` is absent; `AgentToolPolicyResolver` then allows only read (minus subagent filtering). Original `.pi/plans/agents-subagents-implementation-plan.md` section 6.3 listed `tools` as required/explicit allowlist, implying omission should fail validation. Pi reference subagents extension under `/home/ineersa/claw/my-pi/packages/subagents` treats omitted tools as undefined and does not pass `--tools`, so child inherits all available tools; reference worker agent omits `tools` intentionally for full capabilities. Current `['read']` behavior was introduced during AGENT-08 to avoid dropping existing user agent definitions without tools, but it is a new safety-first product decision rather than plan/reference parity. Recommendation from parent: if Hatfield should match pi reference/product expectations, fix AGENT-08 before PR by changing omitted `tools` to inherit all parent-available tools (still excluding `subagent` and preserving SafeGuard/tool hooks), and update docs/skill/tests accordingly; alternatively explicitly document the deliberate divergence as a security-first behavior.

## Task workflow update - 2026-06-25T16:39:01.023Z
- Recorded fork run: 7nnxb8axrvq6
- Summary: Launched implementation fork to align omitted agent-definition `tools` behavior with pi reference subagents extension. Desired behavior: omitted tools inherits all parent-available/model-visible tools except `subagent`; explicit `tools` remains a restrictive allowlist; explicit empty/blank tools invalid; SafeGuard/tool hooks still apply. Fork instructed to update parser/DTO/policy/resolver/tests/docs/skill, run focused Castor validation, commit locally, no push/task move, leave debug HTML exports alone.

## Task workflow update - 2026-06-25T16:41:31.444Z
- Summary: User found two additional AGENT-08/subagent issues during smoke in the same session: (1) agent frontmatter `skills`/`skill` are apparently not preloaded for child agents even when declared; original plan referenced a specific preload format, so current implementation may not be honoring frontmatter skills. (2) Runtime events/state do not expose which provider/model actually ran an agent/turn; where token usage stats are persisted, provider/model should also be fixed/recorded so users can inspect what model executed each turn/subagent.

## Task workflow update - 2026-06-25T16:49:02.620Z
- Summary: User found broader child-context injection failures during AGENT-08 smoke: in subagent child context, frontmatter `skill: improve-codebase-architecture` did not preload skill content (only a text instruction to load it), `inheritProjectContext: true` did not include project context, and `inheritAgentsMd: true` did not include AGENTS.md. Treat as related bugs in child prompt/context assembly: skills/skill content loading, project context inheritance, and AGENTS.md inheritance are not actually injecting expected user-context/system content for child runs.

## Task workflow update - 2026-06-25T16:53:16.131Z
- Summary: AGENT-08 blocker/escalation: user observed that multiple subagent frontmatter/options from the original plan/reference are either ignored or behave differently than intended. Confirmed/likely issues so far: omitted `tools` currently defaults to `['read']` instead of pi-reference inherit-all behavior; `skill`/`skills` are parsed but not preloaded into child context; `inheritProjectContext` and `inheritAgentsMd` defaults/options are not honored in child prompt/context; child/turn events do not record resolved provider/model alongside usage; cancellation does not surface artifact handles. Treat AGENT-08 as blocked until a systematic plan/reference parity audit and fixes are complete, not ready for PR based on docs alone.

## Task workflow update - 2026-06-25T16:53:39.000Z
- Recorded fork run: 7nnxb8axrvq6
- Validation: Fork read testing skill + tests/AGENTS.md before test work.; Focused raw PHPUnit after Castor focused test blocked by Xdebug infinite loop: 115 tests, 331 assertions OK (diagnostic fallback only; not sufficient final Castor evidence).; castor phpstan OK; castor deptrac OK (0 violations); castor cs-check OK after cs-fix; Need parent/follow-up to obtain Castor focused test validation before PR because raw vendor/bin PHPUnit is not acceptable final QA evidence except as diagnostic fallback.
- Summary: Fork completed omitted-tools pi parity at commit 4a91f0d35 fix(agents): inherit tools when omitted. Behavior: omitted `tools` now remains null through AgentFrontmatterDTO/AgentDefinitionDTO and AgentToolPolicyResolver resolves it at child launch using ToolRegistryInterface::activeToolNames(), then filters out `subagent`; explicit `tools` remains a restrictive allowlist; explicit empty/blank tools invalid; SafeGuard/tool hooks unchanged. Docs/skill updated to inherit-all wording. This fixes smoke issue where scout without tools was silently restricted to read-only.

## Task workflow update - 2026-06-25T16:54:13.158Z
- Recorded fork run: 0lnhkkiwjq8t
- Summary: Launched implementation fork for child-context parity after smoke showed `skills`/`skill`, `inheritProjectContext`, and `inheritAgentsMd` were parsed but not actually honored. Fork scope: inject actual skill content for definition `skills`/`skill`, honor AGENTS/project context inheritance flags in child messages, add focused tests proving child RunState/messages contain expected context and suppression works, update docs/skill if semantics clarified, Castor validation, commit locally only.

## Task workflow update - 2026-06-25T17:00:56.904Z
- Recorded fork run: 0lnhkkiwjq8t
- Validation: Fork read testing skill and tests/AGENTS.md before test work.; castor test --filter='AgentPromptBuilderTest|SkillsContextBuilderTest|SubagentExecutionServiceTest' failed due Xdebug stack depth during test DB migration (environment/tooling blocker).; Diagnostic fallback: php -d xdebug.mode=off vendor/bin/phpunit --filter='AgentPromptBuilderTest|SkillsContextBuilderTest' OK (10 tests, 27 assertions).; Diagnostic fallback: php -d xdebug.mode=off vendor/bin/phpunit --filter='SubagentExecutionServiceTest' OK (16 tests, 99 assertions).; castor phpstan on changed src paths OK.; castor deptrac OK after depfile update.; castor cs-check OK.; Need acceptable Castor-focused validation before PR if Castor/Xdebug blocker is resolved or worked around via Castor-sanctioned env.
- Summary: Fork completed child-context parity fix at commit bd52f4405 fix(agents): preload child context from agent definition. Implemented actual definition `skills`/`skill` preload via `SkillsContextBuilder::buildFor()`, injecting rendered skill bodies as `user-context` source `skills_context`; `SubagentExecutionService` now resolves child launch context per definition and honors `inheritProjectContext`/`inheritAgentsMd` by forwarding parent `agents_context` when enabled; `AgentPromptBuilder` inserts skills_context before the non-interactive child contract and user task. Updated docs/agents.md and `.hatfield/skills/subagents/FRONTMATTER.md`; added AgentPromptBuilderTest and SkillsContextBuilder buildFor tests; updated SubagentExecutionServiceTest; depfile.yaml allows AppAgent→AppSkills. Open caveats: parent CLI `--skills` context is not forwarded unless listed in child definition; inheritProjectContext and inheritAgentsMd currently gate same `agents_context` source because there is no separate project-context source.

## Task workflow update - 2026-06-25T17:06:19.567Z
- Summary: User specified desired MCP semantics for subagent/main parity: MCP tools should support (1) globally available everywhere, likely via `mcp: all`; (2) per-agent MCP allowlists declared directly in agent `tools` using `mcp:<tool>` names; (3) when an MCP server/tool is configured as specific, it should not be loaded for the main agent by default, but can be loaded for an agent if its definition includes the corresponding `mcp:<tool>` entries. Proposed sentinel semantics: `mcp:*` means all MCP tools loaded for that agent; `mcp:-` means no MCP tools for that agent. Desired smoke setup: in `mcp.json`, remove database server, leave context7 globally available, move websearch to specific; configure researcher with MCP websearch tools only. Smoke expectations: main agent lists tools and can use context7; main websearch should fail; researcher subagent using context7 should fail when researcher has a specific MCP list excluding it; researcher websearch should succeed. Need assess/implement current code parity because existing `mcp.mode: all/specific` docs/code appear inconsistent and current system may not support `mcp:*`/`mcp:-` tools-list semantics yet.

## Task workflow update - 2026-06-25T17:08:07.021Z
- Summary: Correction to MCP semantics: do NOT invent custom selector syntax like `mcp:<server>:<tool>`. Use the standard MCP tool naming syntax already exposed to the model/tool registry for concrete tools. The only proposed special tokens are `mcp:*` for all MCP tools and `mcp:-` for no MCP tools. Agent definitions should list standard MCP tool names in `tools` to opt into specific MCP tools; implementation/docs must avoid non-standard colon selector syntax.

## Task workflow update - 2026-06-25T17:08:33.535Z
- Summary: Second correction to MCP tool naming: do NOT assume double-underscore names like `mcp__server__tool`. Current Hatfield `McpToolNameMapper` maps MCP tools as `{server}_{tool}` using sanitized components joined by a single underscore (e.g. `websearch_search`, `context7_resolve-library-id` depending on upstream tool names after sanitization). If user refers to a different 'standard MCP syntax', implementation must inspect/currently expose the actual model-visible names and use those; no invented `__` syntax in docs or tests.

## Task workflow update - 2026-06-25T17:09:06.252Z
- Summary: Clarification: current Hatfield concrete MCP tool names do NOT have an `mcp` prefix. `McpToolNameMapper` maps as `{server}_{tool}` with sanitized components joined by `_` (for example `websearch_search` if server=websearch and upstream tool=search). The only `mcp:*` / `mcp:-` discussion is for NEW sentinel tokens if implemented; concrete MCP allowlist entries should be the actual model-visible tool names with no invented `mcp` prefix.

## Task workflow update - 2026-06-25T17:10:25.180Z
- Summary: Finalized desired MCP frontmatter syntax per user: concrete MCP allowlist entries in agent `tools` must be prefixed with `mcp:` even though runtime/model-visible tool names have no `mcp` prefix. Example: `mcp:websearch_search` permits the concrete exposed MCP tool `websearch_search`. To include a whole server/prefix, use `mcp:websearch_` as a prefix selector. Special sentinels: `mcp:*` means all MCP tools; `mcp:-` means no MCP tools. Implementation should strip/interpret `mcp:` frontmatter selectors when resolving child MCP policy, while preserving actual tool registry names like `websearch_search` for model/tool execution. Remove prior incorrect selector assumptions (`mcp:<server>:<tool>`, `mcp__...`, or no frontmatter prefix for MCP entries).

## Task workflow update - 2026-06-25T17:11:03.121Z
- Summary: MCP default rule finalized: if an agent definition's `tools` contains no `mcp:` entries at all, treat it the same as `mcp:-` for MCP access — no MCP tools are exposed to that child, even if normal non-MCP tools are inherited/allowed. This is independent of omitted `tools` default for non-MCP tools: omitted tools may inherit parent non-MCP tools, but MCP remains denied unless explicitly opted in via `mcp:*`, `mcp:<exposed_tool_name>`, or `mcp:<prefix_>`.

## Task workflow update - 2026-06-25T17:12:20.738Z
- Summary: Revised MCP default semantics: distinguish omitted `tools` from explicit tools list. If `tools` is omitted entirely, inherit parent/default non-MCP tools AND include MCP tools from servers/tools marked globally available (`all`), but do not include MCP tools marked `specific`. If `tools` is present and contains no `mcp:` selectors, treat it as an explicit non-MCP allowlist and expose no MCP tools. `mcp:-` remains explicit no-MCP even for globally available MCP. `mcp:*` means all MCP including specific. `mcp:<actual_exposed_tool_name>` permits one MCP tool (e.g. `mcp:websearch_search`). `mcp:<prefix_>` permits all exposed MCP tools whose names start with that prefix (e.g. `mcp:websearch_`).

## Task workflow update - 2026-06-25T17:13:46.047Z
- Recorded fork run: anr4pkm5lnyi
- Summary: Launched implementation fork anr4pkm5lnyi in AGENT-08 worktree to implement finalized MCP config/frontmatter semantics: config availability all vs specific; frontmatter `mcp:` selectors (`mcp:*`, `mcp:-`, `mcp:<actual_tool>`, `mcp:<prefix_>`); omitted tools inherit non-MCP + global MCP, explicit tools without mcp selectors expose no MCP; update .hatfield/mcp.json smoke config (remove database, context7 global, websearch specific), docs/skill, tests, and focused Castor validation. Fork must commit locally only, leave untracked session HTML alone, and report exact smoke instructions.

## Task workflow update - 2026-06-25T17:37:00.504Z
- Recorded fork run: anr4pkm5lnyi
- Validation: Fork read AGENTS.md, testing skill, and tests/AGENTS.md.; castor test --filter='AgentMcpToolsResolverTest|...' failed due test DB migration/container recursion.; APP_ENV=test php bin/console lint:container failed with max stack/container recursion.; castor phpstan scoped to changed Agent/MCP paths OK.; castor deptrac OK after depfile update.; castor cs-check OK.; Not run: castor test:llm-real, castor check.
- Summary: Fork anr4pkm5lnyi partially completed MCP semantics at commit 2eada35c0 (`fix(agents): enforce mcp selectors for child agents`). Implemented child-agent MCP policy from `tools` + `mcp:` selectors, `availability` in MCP config (all/specific, default all), worktree smoke `.hatfield/mcp.json` (database removed, context7 all, websearch specific), `.hatfield/agents/researcher-websearch.md`, docs/skill updates, focused unit tests, depfile allowances. Child selector semantics implemented: `mcp:*`, `mcp:-`, `mcp:<runtime_tool>`, `mcp:<prefix_>` expand to actual runtime `{server}_{tool}` names in child allowed_tools; omitted tools => global MCP only; explicit tools without mcp selectors => no MCP. Important gap: parent/main-run hiding of `availability: specific` MCP tools is NOT implemented; main may still see websearch because MCP catalog registration still exposes all catalog tools. Legacy frontmatter `mcp.mode`/`mcp.tools` is now ignored for child policy in favor of `tools` entries with `mcp:` selectors. Fork reports APP_ENV=test container recursion blocking Castor tests in this worktree; deptrac/phpstan/cs-check clean.

## Task workflow update - 2026-06-25T17:37:41.425Z
- Recorded fork run: d6pdmgqi8omi
- Summary: Launched follow-up fork d6pdmgqi8omi at AGENT-08 worktree HEAD 2eada35c0 to complete MCP semantics: implement parent/main hiding of `availability: specific` MCP tools without breaking child selector access, avoid DI recursion, ensure long-lived registry does not leak specific tools between policies/runs, diagnose/fix Castor/container recursion if branch-caused, add focused parent-vs-child MCP filtering tests, update docs/skill, run focused Castor validation, and commit locally only.

## Task workflow update - 2026-06-25T17:51:49.683Z
- Recorded fork run: d6pdmgqi8omi
- Validation: Fork read AGENTS.md, testing skill, and tests/AGENTS.md.; castor test --filter='AgentMcpToolsResolverTest|...' failed due test kernel migration/Xdebug stack overflow recursion.; Diagnostic fallback: php -d xdebug.mode=off vendor/bin/phpunit on 5 focused test files OK (36 tests, 90 assertions).; bin/console lint:container with xdebug off in worktree stack overflow/unreliable; fork reports same behavior at pre-MCP bd52 baseline and integration checkout lint OK, suggesting worktree/env issue not new services.; castor phpstan scoped to touched MCP/metadata paths OK after fix commit b4742ca0e.; castor deptrac OK (0 violations).; castor cs-check initially failed on touched paths, castor cs-fix applied; final reported clean for touched paths.; Not run: castor test:llm-real, full castor check.
- Summary: Follow-up fork d6pdmgqi8omi completed at HEAD e6b0d4575 on top of partial 2eada35c0. Implemented parent/main hiding of MCP tools from `availability: specific` servers via `McpParentAvailabilityToolSetResolver`, added `McpServerToolAvailability`, child MCP registration fallback to parent catalog via `McpCatalogRegisteringToolSetResolver::registerUsingCatalogFrom`, `SubagentRunMetadataReader::readParentRunId()`, cycle-conscious service chain, docs/skill updates, and focused tests. Parent/main flow now registers catalog tools then filters active toolset to remove specific-server runtime tool names; child flow uses parent catalog if child catalog missing, skips parent availability filter, then `SubagentToolSetResolver` intersects with child `allowed_tools` built from `mcp:` selectors. Manual smoke expectation: main context7 works and websearch unavailable; `researcher-websearch` (`tools: [mcp:websearch_]`) gets websearch and not context7. Known remaining gaps: no child-run first-class MCP init (parent catalog fallback only), no llm-real smoke, worktree Castor test/lint container recursion remains unresolved and appears worktree/env-related, untracked `config/services.yaml.bak` plus session HTML exports left untouched.

## Task workflow update - 2026-06-25T17:55:12.249Z
- Summary: User proposed replacing current `systemPromptMode: append` semantics. Current append appends whole parent system prompt, which can include misleading parent-only/subagent/agent-registry guidance. Desired direction: child system prompt should always include a safe base prompt assembled from the system prompt template pieces needed for good tool use: `<available_tools>` rendered for the child resolved toolset, `<guidelines>` from registered tool guidelines, current date, and cwd, plus child agent instructions, inherited AGENTS/project context, explicit preloaded skills, and child non-interactive contract. Do NOT include `<available_agents>` / agents registry / subagent tool guidance in child prompt because subagent tool is disabled for children. Repurpose `systemPromptMode: append` to mean include APPEND_SYSTEM.md / prompt appends into the child prompt (safe appends channel), not append the entire parent system prompt. This should make future IDE MCP via APPEND_SYSTEM work for subagents while avoiding parent-only agent-launch guidance.

## Task workflow update - 2026-06-25T17:56:10.614Z
- Recorded fork run: cx4u1ftq0h65
- Summary: Launched fork cx4u1ftq0h65 to replace current child `systemPromptMode: append` semantics. Target: build child-safe subagent system prompts that always include child agent instructions, child-resolved `<available_tools>`, relevant `<guidelines>`, date/cwd, inherited AGENTS/project context as flags allow, explicit child skills, contract/task; never include `<available_agents>`, subagent tool guidance, unloaded tools, parent-only tool/MCP guidance, or whole parent system prompt. Repurpose `systemPromptMode: append` to include APPEND_SYSTEM.md/prompt appends with child-safe placeholders rather than parent system prompt. Update tests/docs and run focused Castor validation; commit locally only.

## Task workflow update - 2026-06-25T17:58:52.689Z
- Recorded fork run: cx4u1ftq0h65
- Validation: Fork read AGENTS.md, testing skill, and tests/AGENTS.md.; castor test --filter='AgentPromptBuilderTest|...' failed due worktree test kernel/Xdebug 512-frame recursion.; Diagnostic fallback: php -d xdebug.mode=off vendor/bin/phpunit on focused tests OK (7 tests, 29 assertions).; castor deptrac OK (0 violations).; castor phpstan scoped to changed paths OK.; castor cs-fix applied to touched paths.; SubagentExecutionServiceTest not verified via Castor due kernel block; suggested integration checkout focused Castor filter: castor test --filter='AgentPromptBuilderTest|SubagentExecutionServiceTest|SystemPromptBuilderTest'.
- Summary: Fork cx4u1ftq0h65 completed child-safe subagent prompt semantics at commit aa94f2ceb (`fix(agents): build child-safe subagent system prompts`). Added `config/SUBAGENT_SYSTEM.md`; extended `SystemPromptBuilder` with `buildChildHarnessFragment(array $allowedToolNames)` and `buildChildAppendsFragment(array $allowedToolNames)`; `AgentPromptBuilder` now uses child instructions + child-safe harness + inherited agents/project context + APPEND_SYSTEM/contributors only in append mode; removed whole-parent-system-prompt append and `extractSystemPrompt` path. Child message order remains system → optional `skills_context` user-context → `agent_child_contract` user-context → user task. Default `systemPromptMode: replace` excludes APPEND_SYSTEM; `append` includes APPEND_SYSTEM + prompt contributors rendered with child-safe placeholders. Child prompt excludes `<available_agents>`/agent registry/subagent tool guidance and filters available tools/guidelines to child resolved allowedTools. Updated docs and tests; depfile allows AppAgent→AppSystemPrompt. Known risks: dynamic/MCP tools without promptLine get generic prompt line; prompt contributors are not filtered for child-safety beyond child placeholder values; Castor kernel tests still blocked in worktree by Xdebug/test-kernel recursion.

## Task workflow update - 2026-06-25T18:01:45.662Z
- Summary: Cleaned AGENT-08 worktree untracked files per user approval: removed `config/services.yaml.bak` and `hatfield-session-1.html`, `hatfield-session-3.html`, `hatfield-session-4.html`. Worktree now clean at HEAD aa94f2ceb.

## Task workflow update - 2026-06-25T18:02:53.234Z
- Recorded fork run: 8hoqan4xd9z4
- Summary: Launched urgent fork 8hoqan4xd9z4 to diagnose and fix the worktree Castor recursion/test-kernel failure. User stated `castor check` must pass and recursion did not exist before. Fork is required to reproduce via Castor first, identify exact root cause/service chain, fix code/config/tests while preserving MCP/prompt semantics, run focused Castor validation plus full deterministic `castor check` (warming llm-real cache first if needed), commit locally, and leave worktree clean.

## Task workflow update - 2026-06-25T18:15:13.076Z
- Recorded fork run: 8hoqan4xd9z4
- Summary: Urgent gate-fix fork 8hoqan4xd9z4 completed blocked with no final committed fix. It proved the Castor/test-kernel recursion is a real branch regression, not environment-only: with fresh var/cache and xdebug off, APP_ENV=test bin/console lint:container fails from AGENT-08 commit 2eada35c0 onward while integration main passes. Root cause identified as compile-time DI cycle: eager ToolRegistry constructs all `hatfield.tool_provider` services, including `SubagentTool`; `SubagentTool` depends on `SubagentExecutionService`; service depends on AgentPromptBuilder/SystemPromptBuilder and AgentToolPolicyResolver; those now depend on ToolRegistryInterface, closing ToolRegistry -> SubagentTool -> SubagentExecutionService -> prompt/policy -> ToolRegistry. Lazy-service experiments made lint pass only as a crutch and were intentionally not committed; post-boot registration attempt also invalid (`KernelEvents::BOOT` absent). Secondary test issue: tests stubbing final `McpConfigLoader` must use real loader/test factory. Worktree may be dirty from exploratory edits; needs inspection/revert before next implementation. Recommended real fix direction: split subagent tool definition/provider from execution handler and/or remove ToolRegistry dependency from prompt/policy paths; do not use `#[Lazy]` as final architecture.

## Task workflow update - 2026-06-25T18:18:02.369Z
- Recorded fork run: 9ddgwm60qdwi
- Summary: Launched fork 9ddgwm60qdwi for approved DI-cycle architecture fix. Scope: clean/revert only known exploratory crutch edits from prior diagnostic fork; restore branch from detached dirty state; split `SubagentTool` into lightweight definition provider plus separate runtime handler boundary so eager ToolRegistry registration does not construct `SubagentExecutionService`; use narrow runtime service locator/provider for `SubagentExecutionService` inside handler invocation instead of `#[Lazy]`; keep only definition provider tagged as `hatfield.tool_provider`; remove invalid boot subscriber; fix tests that stub final `McpConfigLoader` using a real test factory; preserve MCP and child prompt semantics; run focused Castor validation, warm llm-real cache if needed, and full deterministic `castor check`; commit locally and leave worktree clean.

## Task workflow update - 2026-06-25T18:53:51.802Z
- Recorded fork run: 9ddgwm60qdwi
- Validation: Fork read AGENTS.md, `.agents/skills/testing/SKILL.md`, and tests/AGENTS.md.; castor test --filter='SubagentToolTest' OK (7 tests).; castor test --filter='SubagentExecutionServiceTest|AgentMcpToolsResolverTest|AgentToolPolicyResolverTest' OK (24 tests after notice fix).; castor phpstan OK.; castor deptrac OK (0 violations).; castor cs-check OK (721/721 clean).; castor test:llm-real OK (9 tests, ~17s) to warm live cache.; Full deterministic castor check OK: QA qa-20260625-183146-5228-ab484e98, quality ok in 147.2s; lanes OK including test 3597/11427, controller-replay 7/97, tui 14/71, llm-real 9/110, phpstan, deptrac, cs-check; llama-proxy cache guard stable 125 -> 125; leak check OK.
- Summary: Fork 9ddgwm60qdwi completed AGENT-08 DI-cycle gate fix at commit c42709938 (`fix(agents): split subagent tool definition from execution`). It restored the branch from detached/dirty diagnostic state, removed exploratory crutches (`SubagentToolRegistrySubscriber`, boot-subscriber/autoconfigure edits), and fixed the ToolRegistry compile-time cycle architecturally without `#[Lazy]` or post-boot registration. New shape: `SubagentToolDefinitionProvider` is the lightweight `hatfield.tool_provider`; `SubagentToolHandler` owns runtime invocation and uses a narrow Symfony `service_locator` to fetch `SubagentExecutionService` only when the `subagent` tool is invoked; `SubagentToolDefinitionBuilder` centralizes schema/prompt metadata. This breaks the former cycle `ToolRegistry -> SubagentTool -> SubagentExecutionService -> prompt/policy -> ToolRegistry` while keeping `subagent` a permanent tool. MCP and child prompt semantics are unchanged. Secondary test issue fixed by using real `TestMcpConfigLoaderFactory` instead of stubbing final `McpConfigLoader`; PHPUnit 13 mock-without-expectations notice fixed. Worktree reported clean on branch `task/agent-08-subagent-docs-final-validation`. Risk noted: `AgentRetrieveTool` still has provider+handler-heavy shape but is not in the current cycle.

## Task workflow update - 2026-06-25T19:02:04.341Z
- Recorded fork run: izw5xwe15o0e
- Summary: Fork izw5xwe15o0e created local manual smoke fixtures in AGENT-08 worktree, uncommitted by design. New untracked files: `.hatfield/APPEND_SYSTEM.md`, `.hatfield/agents/read-only-smoke.md`, `.hatfield/agents/testing-smoke.md`, `.hatfield/agents/append-smoke.md`, `.hatfield/agents/replace-smoke.md`, `.hatfield/agents/omit-tools-smoke.md`. Purpose: allow user to manually smoke append/replace systemPromptMode, explicit read-only tool allowlist, skill preload, omitted-tools inheritance, and local project agent discovery. Worktree intentionally dirty with only these six untracked files at branch HEAD c42709938. Prompts provided in fork handoff for agents read-only-smoke, testing-smoke, append-smoke, replace-smoke, omit-tools-smoke, scout SafeGuard, researcher-websearch MCP, and parent MCP check.

## Task workflow update - 2026-06-25T19:08:05.552Z
- Summary: User manually smoke-tested AGENT-08 local smoke fixtures created by fork izw5xwe15o0e and reported: "Good this passed". This covers the local smoke set for append/replace prompt semantics, explicit read-only allowlist, skill preload, omitted-tools inheritance, and local smoke agent discovery.

## Task workflow update - 2026-06-25T19:18:24.862Z
- Summary: User manual smoke follow-up found remaining issues/decisions after local smoke set passed: (1) child SafeGuard test (`scout` reading ~/.bashrc) ran too long on turn 1 instead of cleanly failing/reporting — likely SafeGuard/HITL auto-reject path not working for subagent children; must solve without crossing architecture boundaries. (2) Parent sees agent report and tends to call `agent_retrieve`; need verify what subagent report enters main-agent context and adjust guidance so parent retrieves only when full handoff/history/detail is needed, not redundantly. (3) Parallel subagent call timed out after 90s; user states subagent tool should not have a short fixed timeout because child agents may run 5–10 minutes. (4) Need to split/move user agents out of `~/.agents` to `~/.pi` and `~/.hatfield` because pi and Hatfield agent frontmatters/setup differ; Hatfield agents should be configured according to Hatfield docs.

## Task workflow update - 2026-06-25T19:22:49.941Z
- Summary: Read-only scouts completed after user smoke findings. Root causes: (1) SafeGuard/HITL hang: child subagent inherits `HATFIELD_APPROVAL_CHANNEL=controller`; SafeGuard sees an approval channel and returns RequireApproval for protected reads instead of auto-denying noninteractive child. `ExtensionToolHookEventSubscriber::handleRequireApproval()` then blocks polling a ToolQuestion forever (no timeout/cancellation check), and SubagentExecutionService WaitingHuman detection is unreachable because the tool never completes. Need architecture-safe fix so noninteractive children auto-deny protected operations without orphaned HITL polls. (2) 90s subagent timeout: `LlmStepResultHandler::resolveToolPolicy()` stamps every ExecuteToolCall with hardcoded `timeout_seconds => 90`, overriding ToolExecutionConfig default 300 and flowing into ToolContext; SubagentExecutionService then uses 90 instead of its 120 fallback, and ToolExecutor may overwrite structured timeout output with generic `Tool "subagent" timed out after 90 second(s)`. Need per-tool timeout policy or subagent-specific longer/configurable timeout. (3) agent_retrieve redundancy: single completed subagent tool result already includes the child final assistant text, same as handoff.md result, so immediate agent_retrieve handoff is redundant; current subagent promptLine/guidelines over-instruct retrieval. Parallel results remain truncated to 240 chars and should still recommend retrieve. (4) agent discovery split: current discovery includes `~/.agents` and `.agents`, causing pi/Hatfield frontmatter overlap; user wants Hatfield agents moved/split to `~/.hatfield`/project `.hatfield` and pi agents to `.pi`/pi-specific location. Treat as separate discovery/migration decision unless included in AGENT-08 after explicit approval.

## Task workflow update - 2026-06-25T19:23:28.442Z
- Recorded fork run: q9uo9m4bc8z9
- Summary: Launched fork q9uo9m4bc8z9 to fix AGENT-08 smoke regressions: SafeGuard/HITL noninteractive child auto-deny (avoid inherited `HATFIELD_APPROVAL_CHANNEL=controller` causing orphaned approval polls), subagent timeout policy (replace hardcoded 90s with per-tool timeout and long subagent timeout suitable for 5–10 min tasks), and reduce redundant `agent_retrieve` guidance by making single-mode full handoff inline explicit while keeping retrieve guidance for parallel/truncated/errors/metadata/history/debug. Fork must not commit untracked smoke fixtures; full Castor validation requested if possible.

## Task workflow update - 2026-06-25T19:27:33.990Z
- Summary: User clarified AGENT-08 smoke fixes: (1) HITL/approval auto-deny must NOT be SafeGuard-specific — all interactive/approval-requiring tool flows should be denied/fail cleanly for noninteractive child subagents. Children should not enter any human approval/wait path. Architecture should represent this generically as noninteractive-run policy, not a SafeGuard special case. (2) Tool timeout fix should not merely set subagent to a larger hard timeout. The hardcoded `timeout_seconds => 90` is wrong; some tools (`subagent`, `bash`) should not have a generic ToolExecutor timeout at all, and bash already has its own timeout. Even write can exceed arbitrary 90s for large/chunky operations. Desired direction: remove generic hard per-call timeout or make timeout opt-in/per-tool nullable, relying on tool-specific timeout/cancellation where appropriate. (3) User confirmed it is good that subagent report is already included in parent LLM context/tool result; retrieve guidance should reflect that.

## Task workflow update - 2026-06-25T19:32:57.007Z
- Recorded fork run: q9uo9m4bc8z9
- Validation: Fork read AGENTS.md, `.agents/skills/testing/SKILL.md`, and tests/AGENTS.md.; Focused Castor tests passed (exact filters not fully listed in handoff).; castor phpstan OK.; castor deptrac OK.; castor cs-check OK after cs-fix.; castor test:llm-real warmed/validated.; Full deterministic castor check passed: QA qa-20260625-193135-5571-6ba011b0; cache guard stable 137 -> 137.
- Summary: Fork q9uo9m4bc8z9 completed commit b16ffc776 (`fix(agents): harden subagent approvals and timeout policy`) addressing the smoke findings as originally scoped before later user clarifications. Changes: added `NoninteractiveChildRunProbe`, `ExtensionToolHookEventSubscriber` now annotates tool call metadata with `noninteractive_child_run`, SafeGuard uses that metadata to auto-deny relaxable protected operations for noninteractive child runs even when `HATFIELD_APPROVAL_CHANNEL=controller` is inherited; added nullable per-tool timeout through `ToolDefinitionDTO`, `ActiveToolSet`, tool set resolvers, and `LlmStepResultHandler` using ToolExecutionSettingsInterface default 300 plus per-tool overrides; added `AgentsConfig::subagentToolTimeoutSeconds` default 900 and `SubagentToolDefinitionBuilder` uses max(60, configured); updated subagent/agent_retrieve guidance so single completed subagent says full handoff is inline and retrieve is optional, while parallel/truncated/errors/metadata/history/debug still recommend retrieve. Worktree clean except six untracked smoke fixtures. IMPORTANT caveat: after fork launch, user clarified desired semantics broader than this commit: all HITL/interactive approval flows should be denied generically for noninteractive child runs, not SafeGuard-specific; and generic hard tool timeouts should be removed/nullable rather than merely default 300 + subagent 900. This commit improves behavior but may need follow-up iteration to match clarified architecture.

## Task workflow update - 2026-06-25T19:33:46.116Z
- Recorded fork run: mkbcexepdjmw
- Summary: Launched follow-up fork mkbcexepdjmw after user clarified broader semantics beyond b16ffc776. Scope: move noninteractive child approval denial to generic HITL/approval handling path (all RequireApproval decisions deny/fail cleanly for noninteractive child runs; no ToolQuestion/poll, not SafeGuard-specific); remove generic hard ToolExecutor timeout model so timeoutSeconds is nullable/opt-in and LlmStepResultHandler does not stamp default hard timeout for every tool; ensure ToolExecutor only post-fails on explicit timeout; preserve retrieve guidance that successful single subagent handoff is inline and agent_retrieve is optional only for metadata/history/debug or parallel/truncated/error cases. Must leave six untracked smoke fixtures uncommitted and run Castor validation/full check if feasible.

## Task workflow update - 2026-06-25T19:38:42.498Z
- Recorded fork run: mkbcexepdjmw
- Validation: Fork read AGENTS.md, `.agents/skills/testing/SKILL.md`, and tests/AGENTS.md.; castor test --filter='ExtensionToolHookEventSubscriberTest|ToolExecutionPolicyResolverTest|ToolExecutorTest|LlmStepResultHandlerTest|SubagentToolDefinitionBuilderTest' OK (40 tests, 208 assertions).; Expanded focused tests including SubagentExecutionServiceTest etc. ran during iteration; final gate broader.; castor phpstan OK.; castor deptrac OK (0 violations).; castor cs-check OK after cs-fix.; castor test:llm-real OK (9 tests, 110 assertions, ~17s).; Full deterministic castor check OK: QA qa-20260625-193721-1655-d4b8053a; 3606 tests; llama-proxy cache stable 137 -> 137.
- Summary: Fork mkbcexepdjmw completed corrected AGENT-08 follow-up at commit e461f9b86 (`fix(agents): deny child approvals and remove generic tool timeouts`). It generalized noninteractive child HITL handling beyond SafeGuard: `ExtensionToolHookEventSubscriber::handleRequireApproval()` now immediately denies any RequireApproval decision when tool-call metadata has `noninteractive_child_run=true`, logs `tool.approval_denied_noninteractive_child`, and does not create/poll ToolQuestion. SafeGuard metadata auto-deny from b16ffc776 remains defense-in-depth, but generic layer is now the backstop for all approval hooks. Timeout semantics changed to nullable/opt-in: `LlmStepResultHandler` no longer stamps default 90/300 timeout; ExecuteToolCall/ToolContext/ToolExecutionPolicy can carry null; ToolExecutor post-hoc timeout check runs only when timeoutSeconds is explicit; `config/hatfield.defaults.yaml` no longer sets global `tools.execution.timeout_seconds`; `SubagentToolDefinitionBuilder` sets `timeoutSeconds=null`; `SubagentExecutionService` keeps internal poll timeout from `AgentsConfig::subagentToolTimeoutSeconds` default 900; bash internal timeout remains separate and null context timeout does not force background poll deadline. Retrieve guidance from b16ffc776 preserved: successful single subagent includes full handoff inline, agent_retrieve only for parallel/truncated/errors/metadata/history/debug needs. Six smoke fixtures remain untracked and uncommitted. Open decisions: whether to remove/document internal subagent 900s cap if product wants no internal cap, and separate agent path split/migration.

## Task workflow update - 2026-06-25T20:11:14.696Z
- Summary: User clarified internal subagent timeout decision: keep the 900s internal `agents.subagent_tool_timeout_seconds` cap for now, but make sure it is configurable in Hatfield settings and documented/synced in settings docs/example. This is separate from removing generic ToolExecutor hard timeouts, which is already done at e461f9b86.

## Task workflow update - 2026-06-25T20:15:07.892Z
- Summary: Follow-up scout investigated actual artifact `agent_36e89905696e2f39` in worktree session `.hatfield/sessions/6`: SafeGuard/subagent/retrieve segment completed cleanly with terminal `agent_end(reason=completed)` at seq 922; both `tool_execution_end` results are non-empty (subagent result includes inline full handoff and Artifact, agent_retrieve result includes full handoff). Later parallel subagent segment was cancelled and final state is cancelled with no pendingToolCalls/activeStepId/working_message. Therefore `Working...` stuck and resume transcript gaps appear to be separate TUI/runtime projection/activity bugs, not AGENT-08 core logic. Created GitHub issues: #215 TUI can remain stuck Working after cancelling parallel subagent execution; #216 Transcript replay can hide tool handoff/retrieve result after tool progress deltas; #217 Resume transcript can contain empty assistant block for tool-call-only LLM step; #218 Footer tokens/sec can disappear when usage payload is missing or turn timing resets.

## Task workflow update - 2026-06-25T20:15:22.733Z
- Recorded fork run: 20x89xfhbnka
- Summary: Launched small docs/settings fork 20x89xfhbnka to ensure `agents.subagent_tool_timeout_seconds` is discoverable/configurable in Hatfield settings and synced in `.hatfield/settings.yaml`, `docs/settings.md`, and project subagents skill if relevant. Scope: no production change unless config parsing missing; commit locally; leave smoke fixtures uncommitted.

## Task workflow update - 2026-06-25T20:16:39.002Z
- Recorded fork run: 20x89xfhbnka
- Validation: Read AGENTS.md before work.; castor cs-check OK (0 files).; castor test --filter=AgentsConfigTest OK (11 tests, 23 assertions).; Full castor check not run because docs/config discovery only, no production code change.
- Summary: Fork 20x89xfhbnka completed docs/settings sync at commit 79cc31ba5 (`docs(agents): document subagent timeout setting`). Confirmed production wiring already existed: `AgentsConfig::$subagentToolTimeoutSeconds` serialized as `subagent_tool_timeout_seconds` default 900, AppConfig loads agents config from Hatfield settings, `SubagentExecutionService::timeoutSeconds()` uses max(60, configured) for internal single/parallel polling. Changes are docs/config examples only: `config/hatfield.defaults.yaml` commented example, `.hatfield/settings.yaml` commented example, `docs/settings.md` dedicated section, `docs/agents.md` stale 120s corrected to 900 and clarified internal poll vs ToolExecutor, `.hatfield/skills/subagents/SKILL.md` defaults table row. Six smoke fixtures remain untracked/uncommitted.

## Task workflow update - 2026-06-25T20:42:34.837Z
- Summary: User smoke observed current reviewer/subagent can run ~15min, so internal `agents.subagent_tool_timeout_seconds` default 900s is too low. Desired new default: 30 minutes / 1800 seconds. Need update production default, tests, docs/settings/examples/skill from 900 to 1800 while preserving configurability and no generic ToolExecutor timeout.

## Task workflow update - 2026-06-25T20:43:10.639Z
- Validation: Reviewer subagent verdict APPROVE WITH SUGGESTIONS; zero blocking.; Reviewer ran focused Castor tests: 48 tests/142 assertions OK and 181 tests/617 assertions OK.; Reviewer ran castor test:controller-replay OK (7 tests/97 assertions).; Reviewer ran castor phpstan OK, castor deptrac OK, castor cs-check OK, php -l on 14 production files OK.
- Summary: Reviewer subagent review at HEAD 79cc31ba5 returned APPROVE WITH SUGGESTIONS (zero blocking). Reviewer verified smoke issues addressed: child SafeGuard/approval denial fixed via SafeGuard + generic ExtensionToolHookEventSubscriber denial, no generic 90s subagent timeout (`SubagentToolDefinitionBuilder` timeout null; ToolExecutor skips null), omitted tools inheritance fixed, skills/project context loading fixed, MCP selectors fixed (`mcp:*`, `mcp:-`, `mcp:<tool>`, `mcp:<prefix_>`), parent-specific MCP hiding fixed, single-mode handoff inline fixed. Validation by reviewer: focused Castor tests 48/142 OK + 181/617 OK, controller-replay 7/97 OK, phpstan 0, deptrac 0, cs-check clean, php -l new/changed production files clean. Reviewer non-blocking findings: (1) `.hatfield/skills/subagents/FRONTMATTER.md` still says legacy `mcp.mode: specific` merges tool names into child allowlist, but runtime uses only `tools` entries with `mcp:` selectors; fix docs or wire legacy block; pragmatic doc fix recommended. (2) `LlmStepResultHandler::resolveToolPolicy()` now honors configured `tools.execution.max_parallelism` default 4 instead of previous hardcoded 1 for all tools; likely correct but broader behavior change; production settings=4 path not directly unit tested. (3) `SubagentExecutionService::timeoutSeconds()` clamps below-60 config silently; docs mention floor but logging warning could help. (4) Minor code smell: `InProcessAgentSessionClient` variable `$agentsContext` reused for AGENTS.md array and available-agents string; rename second to `$availableAgents`. (5) Coverage gap: `McpCatalogRegisteringToolSetResolver::resolveCatalogRunId()` child-own-catalog before parent fallback not directly unit tested. (6) Docs table has duplicate `tools` rows. (7) Nice-to-have: smoke fixtures under `.hatfield` risk accidental commit; clean before PR. (8) Explicit null timeout comment in `hatfield.defaults.yaml` could be clearer.

## Task workflow update - 2026-06-25T20:43:18.428Z
- Summary: Architect subagent review at HEAD 79cc31ba5 returned architecturally sound / ready for merge, zero blockers. Positive findings: child prompt construction deepened correctly via `SystemPromptBuilder::buildChildHarnessFragment()`/`buildChildAppendsFragment()` using child-safe filtered placeholders and not copying parent system prompt; timeout separation is clean with three layers (ToolExecutor opt-in nullable default none, ActiveToolSet per-run map, subagent internal poll via settings); dual noninteractive approval gates are sound defense-in-depth; subagent tool definition/handler split resolves DI cycle pragmatically with service locator; frontmatter normalization is well-structured and documented; docs/settings/skill coherence good; App/Core/TUI boundaries and ExtensionApi boundary preserved; TUI resume issues are separate from AGENT-08 scope. Architect suggestions (non-blocking): (1) conceptual layering concern: `McpParentAvailabilityToolSetResolver` in AppMcpTool depends on `SubagentRunMetadataReader` from AppAgent to detect child runs; deptrac allows it but a cleaner design would pass an isAgentChild boolean/context instead of MCP resolver reading agent event metadata. (2) Extract MCP selector expansion from 209-line `AgentMcpToolsResolver` for testability. (3) Service locator in `SubagentToolHandler` acceptable v1 but fragile string key; consider const/public key or compiler/proxy later. (4) Rename confusing `AgentContextRenderer` vs `AgentsContextRenderer`. (5) SkillsContextBuilder does full skill discovery per child launch; performance concern for parallel skills, not correctness.

## Task workflow update - 2026-06-25T20:47:26.356Z
- Summary: CRITICAL smoke investigation of session 6 parallel hang found an AGENT-08 blocker: not timeout, but a durable tool batch lost-update race. In `.hatfield/sessions/6`, parallel subagent turn 34 launched scout `agent_ca4c863473b2bf80` and reviewer `agent_6df9eda9610d51c6`; scout completed, reviewer stalled at turn 2 after issuing 3 parallel `read` tools. All 3 read tool executions completed and wrote events, but `DbalToolBatchStore::save()` performs non-atomic read-modify-write with Doctrine flush and no lock/version. Concurrent tool result workers each loaded batch state, added one result, and saved, overwriting other results. `ToolBatchCollector` then saw only 1/3 results forever, never emitted `tool_batch_committed`, reviewer stayed Running with pendingToolCalls, parent subagent polling loop reported reviewer turn 2 until user cancelled after ~9m. No old session workers remain; current worker PIDs are active session. This is triggered by parallel tool execution generally, but AGENT-08 now honors configured `tools.execution.max_parallelism` (default 4) instead of hardcoded 1, exposing the race. Correct fix is atomic batch updates in `DbalToolBatchStore` (pessimistic/optimistic locking + retry or atomic JSON update). Mitigations like forcing child tools sequential are inferior. Treat as AGENT-08 blocker before PR.

## Task workflow update - 2026-06-25T20:47:57.284Z
- Recorded fork run: r1g0myum3yr7
- Summary: Launched blocker-fix fork r1g0myum3yr7. Scope: fix durable tool batch lost-update race in `DbalToolBatchStore`/`ToolBatchCollector` with atomic read-modify-write (transaction + pessimistic lock or equivalent) and add DB integration regression for concurrent same-batch results; bump `agents.subagent_tool_timeout_seconds` default/docs/tests from 900 to 1800; correct FRONTMATTER legacy `mcp.mode` doc nit if small; run focused Castor tests plus phpstan/deptrac/cs-check/controller-replay and full castor check if feasible; commit locally; leave six smoke fixtures uncommitted.

## Task workflow update - 2026-06-25T20:52:44.035Z
- Recorded fork run: r1g0myum3yr7
- Validation: Fork read AGENTS.md, `.agents/skills/testing/SKILL.md`, and tests/AGENTS.md.; castor test --filter='DbalToolBatchStoreConcurrencyTest|DbalToolBatchStoreTest|ToolBatchCollectorDurableTest|AgentsConfigTest' OK (27 tests, 91 assertions).; castor phpstan OK (0 errors).; castor deptrac OK (0 violations).; castor cs-check OK after cs-fix.; castor test:controller-replay OK (7 tests, 97 assertions).; castor test:llm-real OK (9 tests, 110 assertions; warm).; Full deterministic castor check OK: QA qa-20260625-205131-1289-a291fb50; 3608 tests; llama-proxy cache stable 137 -> 137.
- Summary: Fork r1g0myum3yr7 completed AGENT-08 blocker fix at commit 6b08a0154 (`fix(runtime): serialize tool batch updates and extend subagent timeout default`). Implemented `ToolBatchStoreMutation` and `ToolBatchStoreInterface::mutate()`. `DbalToolBatchStore::mutate()` wraps mutation in transaction and applies pessimistic write lock on `(run_id, turn_no, step_id)` row before callback/persist, preventing concurrent tool workers from lost-updating the JSON batch. `ToolBatchCollector::collect()` now uses durable `mutate()` when a store exists, applying shared `applyCollectToBatch()` inside the locked mutation; in-memory behavior preserved. Added DB regression `DbalToolBatchStoreConcurrencyTest` proving 3 parallel collects finalize and sequential mutates accumulate, plus durable collector test updates. This addresses session-6 hang where reviewer child turn 2 had 3 parallel read results overwrite each other, leaving batch 1/3 forever. Also changed default `agents.subagent_tool_timeout_seconds` from 900 to 1800 in `AgentsConfig`, tests, docs/settings/examples/skill; subagent still has `ToolDefinitionDTO.timeoutSeconds=null` (no ToolExecutor cap). Corrected `.hatfield/skills/subagents/FRONTMATTER.md` MCP wording to use `tools` + `mcp:` selectors, not legacy `mcp.mode`. Six smoke fixtures remain untracked/uncommitted. Open risk: `save()` remains non-atomic for expected-batch registration, but registration is single-writer; collect race fixed. If a non-SQLite DB weakens pessimistic locking, revisit optimistic version+retry.

## Task workflow update - 2026-06-25T20:57:13.978Z
- Recorded fork run: fc9iuti55hml
- Summary: Launched small hardening fork fc9iuti55hml after user agreed silent clamp is concerning. Scope: remove silent `max(60, ...)` behavior for `agents.subagent_tool_timeout_seconds`; validate settings/config so explicit values below 60 are rejected with clear error/diagnostic instead of silently overridden; keep default 1800; update docs/tests; preserve no generic ToolExecutor timeout; commit locally; leave smoke fixtures uncommitted.

## Task workflow update - 2026-06-25T20:58:48.609Z
- Recorded fork run: fc9iuti55hml
- Validation: Fork read AGENTS.md, `.agents/skills/testing/SKILL.md`, and tests/AGENTS.md.; castor test --filter=AgentsConfigTest OK (14 tests, 28 assertions).; castor phpstan OK (0 errors).; castor deptrac OK (0 violations).; castor cs-check OK after cs-fix on AgentsConfig.php.; Full castor check not run for tiny config/docs change; previous full check at 6b08a0154 passed.
- Summary: Fork fc9iuti55hml completed hardening at commit 176b6ced2 (`fix(agents): reject too-low subagent timeout config`). `AgentsConfig::fromRaw()` now validates `agents.subagent_tool_timeout_seconds`: missing -> default 1800; non-int -> InvalidArgumentException; <60 -> InvalidArgumentException; >=60 accepted. Added constants `DEFAULT_SUBAGENT_TOOL_TIMEOUT_SECONDS=1800` and `SUBAGENT_TOOL_TIMEOUT_SECONDS_MIN=60` (names per implementation). `SubagentExecutionService::timeoutSeconds()` now returns config value directly with no `max(60, ...)` silent clamp. Docs/settings/agents/skill/defaults comment updated to say min 60 and invalid values fail config load, not silently adjusted. Added AgentsConfigTest coverage for reject 30, reject string, accept 60. Six smoke fixtures remain untracked/uncommitted.

## Task workflow update - 2026-06-25T20:59:20.636Z
- Recorded fork run: da6z4pphxvp4
- Summary: Launched final low-risk cleanup fork da6z4pphxvp4. Scope: review/doc nits only — ensure FRONTMATTER legacy `mcp.mode` wording is corrected, clean duplicate/confusing `tools` rows in docs/agents.md, clarify hatfield.defaults timeout comment (generic ToolExecutor null/opt-in vs subagent internal timeout), rename confusing `$agentsContext` reuse in InProcessAgentSessionClient, add focused McpCatalogRegisteringToolSetResolver child-catalog fallback test if straightforward, optionally tiny service-locator key cleanup. Explicitly out of scope: MCP layering refactor, selector extraction, renderer renames, TUI resume issues. Leave untracked smoke fixtures uncommitted/unremoved.

## Task workflow update - 2026-06-25T21:01:29.126Z
- Recorded fork run: da6z4pphxvp4
- Validation: Fork read AGENTS.md, `.agents/skills/testing/SKILL.md`, and tests/AGENTS.md.; castor test --filter=McpCatalogRegisteringToolSetResolverTest OK (6 tests, 14 assertions).; castor phpstan OK (0 errors).; castor deptrac OK (0 violations).; castor cs-check OK.; Full castor check not run for docs/rename/focused-test cleanup; previous full check at 6b08a0154 passed, with focused validation after commits 176b6ced2 and b11218ee1.
- Summary: Fork da6z4pphxvp4 completed final low-risk review cleanup at commit b11218ee1 (`chore(agents): address final review nits`). Addressed docs clarity: `.hatfield/skills/subagents/FRONTMATTER.md` now clearly distinguishes legacy top-level `mcp:` block from selector syntax and examples use `tools` list only; `docs/agents.md` has a single merged `tools` row and removed confusing legacy `mcp.tools` row/examples; `config/hatfield.defaults.yaml` clarifies there is no generic ToolExecutor timeout default and subagent polling uses `agents.subagent_tool_timeout_seconds`. Code clarity: `InProcessAgentSessionClient` now uses `$availableAgentsContext` for the `<available_agents>` block instead of reusing `$agentsContext`. Added focused `McpCatalogRegisteringToolSetResolverTest` coverage for child own catalog vs parent fallback using observed registered tools because `McpToolRegistrar` is final. Skipped broad nits intentionally: MCP layering refactor, selector extraction, renderer rename, SubagentToolHandler locator key change (already documented as `execution`). Six smoke fixtures remain untracked/uncommitted.

## Task workflow update - 2026-06-25T21:15:40.345Z
- Validation: Final pre-CODE-REVIEW full gate at HEAD b11218ee1: `castor check` OK, QA qa-20260625-211447-365474-28e1cf09.; Castor check lane results: deptrac OK; test OK (3613 tests, 11477 assertions); test:controller-replay OK (7 tests, 97 assertions); test:tui OK (14 tests, 71 assertions); test:llm-real OK (9 tests, 110 assertions); phpstan OK; cs-check OK.; llama-proxy cache guard stable 137 -> 137; QA artifact integrity OK; QA run leak check OK.; Removed six untracked smoke fixtures before gate so worktree tracked/untracked state is clean.

## Task workflow update - 2026-06-25T21:16:48.438Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (44.0s).
- Pushed task/agent-08-subagent-docs-final-validation to origin.
- branch 'task/agent-08-subagent-docs-final-validation' set up to track 'origin/task/agent-08-subagent-docs-final-validation'.
- Created PR: https://github.com/ineersa/agent-core/pull/219
- Validation: Pre-transition full gate: castor check OK at HEAD b11218ee1, QA qa-20260625-211447-365474-28e1cf09.; Pre-transition lanes: deptrac OK; test OK (3613 tests, 11477 assertions); test:controller-replay OK (7 tests, 97 assertions); test:tui OK (14 tests, 71 assertions); test:llm-real OK (9 tests, 110 assertions); phpstan OK; cs-check OK.; Pre-transition llama-proxy cache guard stable 137 -> 137; QA artifact integrity OK; leak check OK.; Focused validations from final forks: AgentsConfigTest OK (14 tests, 28 assertions); McpCatalogRegisteringToolSetResolverTest OK (6 tests, 14 assertions); DbalToolBatchStore/ToolBatchCollector/AgentsConfig focused tests OK (27 tests, 91 assertions).
- Summary: AGENT-08 ready for code review at HEAD b11218ee1. Implemented final subagent product validation/docs/parity scope: project-level `.hatfield` subagents skill, available-agents context, parser normalization (comma-separated tools/skills, singular skill alias), omitted tools inherit parent-available tools excluding subagent, explicit tool allowlists, child skills/context injection, child-safe system prompts with tool/guideline harness and append semantics, MCP availability/specific selectors (`mcp:*`, `mcp:-`, `mcp:<tool>`, `mcp:<server_>`), parent hiding for specific MCP tools, split subagent tool definition/handler to resolve ToolRegistry DI cycle, noninteractive child approval denial, nullable/opt-in generic tool timeout with subagent internal poll timeout default 1800, clearer agent_retrieve guidance, atomic durable tool batch mutation to fix parallel result lost-update hang, and final docs/test cleanup. Reviewer and architect subagents returned approve-with-suggestions/no blockers; subsequent blocker and nits were fixed. Manual user smokes passed including parallel 5-scout docs read after atomic batch fix. Six smoke fixtures were removed before final gate so worktree is clean.

## Task workflow update - 2026-06-25T21:18:20.642Z
- Moved CODE-REVIEW → DONE.
- Merged task/agent-08-subagent-docs-final-validation into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/agents/researcher-websearch.md           |  11 +
 .hatfield/mcp.json                                 |  26 +-
 .hatfield/settings.yaml                            |   5 +
 .hatfield/skills/subagents/FRONTMATTER.md          |  94 ++++++
 .hatfield/skills/subagents/SKILL.md                |  69 ++++
 config/SUBAGENT_SYSTEM.md                          |  18 ++
 config/SYSTEM.md                                   |   1 +
 config/hatfield.defaults.yaml                      |  11 +-
 config/services.yaml                               |  35 +-
 depfile.yaml                                       |   6 +
 docs/agents.md                                     |  90 +++---
 docs/settings.md                                   |  30 +-
 .../Application/Handler/InMemoryToolBatchStore.php |  17 +
 .../Application/Handler/ToolBatchCollector.php     |  75 ++++-
 .../Handler/ToolExecutionPolicyResolver.php        |   8 +-
 src/AgentCore/Application/Handler/ToolExecutor.php |  16 +-
 .../Application/Pipeline/LlmStepResultHandler.php  |  25 +-
 src/AgentCore/Application/Tool/ToolContext.php     |   4 +-
 src/AgentCore/Contract/Tool/ActiveToolSet.php      |   3 +
 .../Contract/Tool/ToolBatchStoreInterface.php      |  11 +
 .../Contract/Tool/ToolBatchStoreMutation.php       |  21 ++
 .../Tool/ToolExecutionSettingsInterface.php        |   2 +-
 src/AgentCore/Domain/Tool/ToolExecutionPolicy.php  |   2 +-
 .../Agent/Context/AgentContextRenderer.php         |  55 ++++
 .../Agent/Context/AgentsContextBuilder.php         |  40 +++
 .../Agent/Definition/AgentDefinitionDTO.php        |   8 +-
 .../Agent/Definition/AgentDefinitionParser.php     |  60 +++-
 .../Agent/Definition/AgentFrontmatterDTO.php       |  33 +-
 .../Agent/Definition/SystemPromptModeEnum.php      |  11 +-
 .../Agent/Execution/AgentMcpToolsResolver.php      | 209 ++++++++++++
 .../Agent/Execution/AgentPromptBuilder.php         |  89 ++---
 .../Agent/Execution/AgentToolPolicyResolver.php    |  56 ++--
 .../Agent/Execution/SubagentExecutionService.php   |  85 +++--
 .../Agent/Execution/SubagentRunMetadataReader.php  |  23 ++
 .../Agent/Execution/SubagentToolSetResolver.php    |   8 +
 src/CodingAgent/Agent/Tool/AgentRetrieveTool.php   |   4 +-
 src/CodingAgent/Agent/Tool/SubagentTool.php        | 124 -------
 .../Agent/Tool/SubagentToolDefinitionBuilder.php   |  74 +++++
 .../Agent/Tool/SubagentToolDefinitionProvider.php  |  29 ++
 src/CodingAgent/Agent/Tool/SubagentToolHandler.php |  86 +++++
 src/CodingAgent/Config/AgentsConfig.php            |  32 +-
 src/CodingAgent/Config/ToolExecutionConfig.php     |  10 +-
 src/CodingAgent/Config/ToolSettings.php            |   6 +-
 .../Builtin/SafeGuard/SafeGuardToolCallHook.php    |  18 +-
 .../Extension/ExtensionToolHookEventSubscriber.php |  31 ++
 .../Extension/NoninteractiveChildRunProbe.php      |  54 ++++
 src/CodingAgent/Mcp/Config/McpConfigValidator.php  |   8 +-
 .../Mcp/Config/McpServerAvailabilityEnum.php       |  17 +
 .../Mcp/Config/McpServerDefinitionDTO.php          |  13 +
 .../Tool/McpCatalogRegisteringToolSetResolver.php  |  23 +-
 .../Tool/McpParentAvailabilityToolSetResolver.php  |  93 ++++++
 .../Mcp/Tool/McpServerToolAvailability.php         |  63 ++++
 src/CodingAgent/Mcp/Tool/McpToolRegistrar.php      |  13 +-
 .../InProcess/InProcessAgentSessionClient.php      |  14 +
 src/CodingAgent/Skills/SkillsContextBuilder.php    |  55 ++++
 .../SystemPrompt/SystemPromptBuilder.php           | 170 ++++++++++
 .../Tool/CodingAgentToolSetResolver.php            |   5 +
 src/CodingAgent/Tool/Store/DbalToolBatchStore.php  |  51 +++
 src/CodingAgent/Tool/ToolDefinitionDTO.php         |   2 +
 .../RuntimeBashBackgroundPromptAdapter.php         |  13 +-
 .../Handler/ToolBatchCollectorDurableTest.php      |  33 +-
 .../Handler/ToolExecutionPolicyResolverTest.php    |  23 +-
 .../Application/Handler/ToolExecutorTest.php       |  39 +++
 .../Pipeline/LlmStepResultHandlerTest.php          |  49 +++
 .../Agent/Context/AgentContextRendererTest.php     |  78 +++++
 .../Agent/Context/AgentsContextBuilderTest.php     | 117 +++++++
 .../Definition/AgentDefinitionDiscoveryTest.php    |  34 ++
 .../Agent/Definition/AgentDefinitionParserTest.php | 108 +++++--
 .../Agent/Execution/AgentMcpToolsResolverTest.php  | 134 ++++++++
 .../Agent/Execution/AgentPromptBuilderTest.php     | 239 ++++++++++++++
 .../Execution/AgentToolPolicyResolverTest.php      | 147 ++++-----
 .../Execution/SubagentExecutionServiceTest.php     | 357 +++++++++++++++++++--
 .../Tool/SubagentToolDefinitionBuilderTest.php     |  21 ++
 tests/CodingAgent/Agent/Tool/SubagentToolTest.php  |  38 +--
 tests/CodingAgent/Config/AgentsConfigTest.php      |  31 ++
 tests/CodingAgent/Config/AppConfigLoaderTest.php   |  24 ++
 .../SafeGuard/SafeGuardToolCallHookTest.php        |  43 +++
 .../ExtensionToolHookEventSubscriberTest.php       |  87 +++++
 .../Extension/NoninteractiveChildRunProbeTest.php  |  49 +++
 .../CodingAgent/Mcp/Config/McpConfigLoaderTest.php |  26 ++
 .../McpCatalogRegisteringToolSetResolverTest.php   | 150 ++++++++-
 .../McpParentAvailabilityToolSetResolverTest.php   | 123 +++++++
 .../Skills/SkillsContextBuilderTest.php            |  25 ++
 .../Support/Mcp/TestMcpConfigLoaderFactory.php     |  49 +++
 .../SystemPrompt/SystemPromptBuilderTest.php       |  16 +
 .../Store/DbalToolBatchStoreConcurrencyTest.php    | 132 ++++++++
 86 files changed, 3885 insertions(+), 552 deletions(-)
 create mode 100644 .hatfield/agents/researcher-websearch.md
 create mode 100644 .hatfield/skills/subagents/FRONTMATTER.md
 create mode 100644 .hatfield/skills/subagents/SKILL.md
 create mode 100644 config/SUBAGENT_SYSTEM.md
 create mode 100644 src/AgentCore/Contract/Tool/ToolBatchStoreMutation.php
 create mode 100644 src/CodingAgent/Agent/Context/AgentContextRenderer.php
 create mode 100644 src/CodingAgent/Agent/Context/AgentsContextBuilder.php
 create mode 100644 src/CodingAgent/Agent/Execution/AgentMcpToolsResolver.php
 delete mode 100644 src/CodingAgent/Agent/Tool/SubagentTool.php
 create mode 100644 src/CodingAgent/Agent/Tool/SubagentToolDefinitionBuilder.php
 create mode 100644 src/CodingAgent/Agent/Tool/SubagentToolDefinitionProvider.php
 create mode 100644 src/CodingAgent/Agent/Tool/SubagentToolHandler.php
 create mode 100644 src/CodingAgent/Extension/NoninteractiveChildRunProbe.php
 create mode 100644 src/CodingAgent/Mcp/Config/McpServerAvailabilityEnum.php
 create mode 100644 src/CodingAgent/Mcp/Tool/McpParentAvailabilityToolSetResolver.php
 create mode 100644 src/CodingAgent/Mcp/Tool/McpServerToolAvailability.php
 create mode 100644 tests/CodingAgent/Agent/Context/AgentContextRendererTest.php
 create mode 100644 tests/CodingAgent/Agent/Context/AgentsContextBuilderTest.php
 create mode 100644 tests/CodingAgent/Agent/Execution/AgentMcpToolsResolverTest.php
 create mode 100644 tests/CodingAgent/Agent/Execution/AgentPromptBuilderTest.php
 create mode 100644 tests/CodingAgent/Agent/Tool/SubagentToolDefinitionBuilderTest.php
 create mode 100644 tests/CodingAgent/Extension/NoninteractiveChildRunProbeTest.php
 create mode 100644 tests/CodingAgent/Mcp/Tool/McpParentAvailabilityToolSetResolverTest.php
 create mode 100644 tests/CodingAgent/Support/Mcp/TestMcpConfigLoaderFactory.php
 create mode 100644 tests/CodingAgent/Tool/Store/DbalToolBatchStoreConcurrencyTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/agent-08-subagent-docs-final-validation.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/agent-08-subagent-docs-final-validation.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #219 merged by user.; Pre-CODE-REVIEW castor check OK at b11218ee1: QA qa-20260625-211447-365474-28e1cf09; deptrac OK; test OK (3613 tests, 11477 assertions); controller-replay OK (7 tests, 97 assertions); test:tui OK (14 tests, 71 assertions); test:llm-real OK (9 tests, 110 assertions); phpstan OK; cs-check OK; llama-proxy cache stable 137 -> 137; leak check OK.; CODE-REVIEW transition deterministic castor check passed and PR #219 was created successfully.
- Summary: AGENT-08 PR #219 was merged by user. Final implementation completed full subagent product validation/docs/parity pass: project-level subagents skill, available-agents context, parser normalization and omitted-tools inheritance, child skills/context injection and child-safe prompts, MCP selectors and parent/specific availability handling, subagent definition/handler split resolving DI cycle, noninteractive child approval denial, nullable tool timeout semantics with 1800s subagent poll default and validation, clearer retrieve guidance, atomic durable batch mutation fixing parallel result lost-update hang, docs/test cleanup. Pre-PR and transition gates passed; user confirmed merge.
