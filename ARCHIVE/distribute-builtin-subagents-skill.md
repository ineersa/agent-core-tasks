# Bundle and safely materialize Hatfield's built-in subagents skill

## Goal
## Goal

Ship Hatfield's internal `subagents` skill with source, PHAR, and static distributions, then make it discoverable for installed Hatfield users without requiring a project checkout or manual copying.

The current canonical skill lives at `.hatfield/skills/subagents/` and is only available inside this repository. Move/refine it into a bundled application resource under `src/` (exact existing resource convention to be identified), include its supporting files, and materialize it under `~/.hatfield/skills/subagents/` before skill discovery.

## Investigation and decisions

1. Trace the earliest shared startup point for source, PHAR, static relaunch, CLI, controller, and worker processes, and identify where bundled resources can be copied before `SkillDiscovery` first runs.
2. Reuse `AppResourceLocator` and any existing atomic resource-install/copy pattern before adding code.
3. Finalize safe ownership semantics for `~/.hatfield/skills/subagents/`:
   - absent destination: create it;
   - Hatfield-managed destination: refresh it when bundled content changes;
   - pre-existing/unmanaged destination: do not silently destroy user content; choose and document a deterministic warning/failure/migration policy before implementation.
4. Define atomic update and interrupted-write behavior so startup never leaves a partial `SKILL.md` or supporting files.
5. Confirm precedence/collision behavior. The generated home `.hatfield` skill should retain existing discovery precedence and remain overridable by project-local skills.
6. Decide whether startup synchronization runs every process or through one idempotent/locked initializer; avoid repeated unsafe writes from concurrent controller/workers.
7. Avoid a general skill marketplace, remote updater, package manager, manifest schema, or speculative built-in-skill registry. This task covers the single Hatfield-owned `subagents` skill unless one tiny shared resource primitive already exists.

## Skill refinement

Audit and tighten the current subagents skill against the canonical runtime behavior and docs:

- `subagent` single vs parallel invocation and `agents.max_agents`;
- `agent_retrieve` artifact lifecycle;
- agent definition paths/frontmatter;
- child tool/MCP inheritance and SafeGuard behavior;
- timeout/settings names;
- unsupported nested subagents;
- source/PHAR/static portability.

Copy all required local references (including `FRONTMATTER.md` if retained) into the bundled skill resource. Remove or replace repository-relative links that would break after installation under `~/.hatfield/skills/subagents/`. Keep the skill concise and use canonical product docs rather than duplicating them where runtime-accessible documentation already exists.

## Packaging and lifecycle

- The canonical bundled copy must be inside the application resources packaged by existing PHAR/static builds.
- Source execution, PHAR execution, and native/static relaunch must resolve the same bundled resource.
- Startup materialization must not depend on Composer, network access, release mirrors, or project `.hatfield/` contents.
- Updates must replace only Hatfield-owned managed files and remove obsolete managed files without touching unrelated user skills.
- Uninstall cleanup is not required unless Hatfield already has an uninstall lifecycle; document that the last materialized managed copy may remain.

## Non-goals

- Distributing extension-owned skills (handled separately by `registerSkill()`).
- Migrating task-workflow or other extension skills.
- A generic marketplace, registry, downloader, or remote update service.
- User-editable synchronization or bidirectional merges.
- New settings unless a concrete unresolved safety requirement requires user approval.
- TUI behavior changes.

## Expected implementation areas

- bundled resource under `src/` using the existing application resource layout;
- startup/resource initialization service before skill discovery;
- `AppResourceLocator` or equivalent existing locator seam;
- PHAR/static packaging only if `src/` inclusion does not already cover the resource;
- current `.hatfield/skills/subagents/` project copy migration/removal;
- focused source and packaged-resource tests;
- concise skill/distribution/settings documentation.

## Validation strategy

Before tests, load the testing skill and read `tests/AGENTS.md`. Use Castor only.

Test thesis: a fresh Hatfield home receives the complete bundled subagents skill before discovery; a managed older copy refreshes atomically; an unmanaged pre-existing directory is never silently destroyed; project precedence still overrides the generated home skill.

Use 1–3 focused tests at the lowest non-TUI layer, plus:

- `castor test` focused/full as appropriate;
- `castor deptrac`;
- `castor phpstan`;
- `castor cs-check`;
- `castor phar:build` and a bounded PHAR smoke proving the bundled resource can be materialized/discovered from an isolated home;
- normal `castor check` at CODE-REVIEW.

No TUI/tmux test is required unless implementation unexpectedly changes TUI behavior.

## Acceptance criteria
- The canonical subagents skill and all required local references are bundled in an existing `src/` application-resource path and included in source, PHAR, and static distributions.
- Hatfield materializes the bundled skill under `~/.hatfield/skills/subagents/` before discovery without Composer, network access, or a project checkout.
- Install/update behavior is idempotent and atomic, with explicit managed ownership; pre-existing unmanaged user content is not silently overwritten.
- Concurrent startup processes cannot leave partial or conflicting skill files.
- The installed skill has no broken repository-relative links and accurately reflects current subagent tools, agent frontmatter, MCP/tool inheritance, artifacts, settings, and limitations.
- Existing skill precedence remains documented and project-local skills can override the generated home skill.
- Focused automated tests prove fresh install, managed refresh, unmanaged-content safety, and discovery; PHAR smoke proves packaged resource availability.
- No marketplace, remote updater, broad plugin framework, speculative setting, or unrelated skill migration is introduced.

## Workflow metadata
Status: ARCHIVE
Branch: task/distribute-builtin-subagents-skill
Worktree: /home/ineersa/projects/agent-core-worktrees/distribute-builtin-subagents-skill
Fork run: hgapjriibjhn
PR URL: https://github.com/ineersa/agent-core/pull/367
PR Status: merged
Started: 2026-08-06T01:29:23.844Z
Completed: 2026-08-06T20:39:45.097Z

## Work log
- Created: 2026-08-05T21:11:44.733Z

## Task workflow update - 2026-08-05T21:17:40.757Z
- Summary: Scope correction from user: `~/.hatfield/skills/subagents/` is an authoritative Hatfield-owned built-in skill location. Startup should rewrite/replace it from the bundled copy; protecting or preserving pre-existing unmanaged content at that exact path is explicitly not required. Keep writes atomic/concurrency-safe so the built-in copy is never partial.
- User clarification supersedes the task body and acceptance language about unmanaged-content protection: the `subagents` destination is built-in and Hatfield owns/replaces the whole directory. No ownership marker, migration warning, merge, or preservation behavior is needed.

## Task workflow update - 2026-08-06T01:29:16.721Z
- Summary: Finalized scope for implementation. Built-in skills use a general directory convention under `src/CodingAgent/Resources/skills/<name>/`; all direct skill directories are materialized into `~/.hatfield/skills/<name>/` immediately before normal skill discovery. Keep implementation deliberately simple: Symfony Filesystem mirroring/overwrite, no lock, staging tree, backup/recovery protocol, ownership marker, startup subscriber, setting, registry, manifest, or marketplace. Hatfield owns and rewrites each bundled destination. The subagents skill must be precise, succinct, below 20,000 characters across its complete bundled skill content, and have excellent trigger-focused frontmatter. Every behavioral/settings/tool-schema claim must be verified against current production code and canonical docs.
- User approved task start and explicitly required no overengineering. Latest clarification supersedes earlier task text requiring atomic/concurrency-safe replacement and unmanaged-content protection.
- Generalizability means directory convention plus enumeration: adding another built-in skill directory requires no materializer code change. It does not authorize a broader plugin/update framework.
- Skill quality gate: combined bundled subagents skill text must be <20,000 characters; trigger description must clearly state capability and when to load; information must be current and exact.

## Task workflow update - 2026-08-06T01:29:23.844Z
- Moved TODO → IN-PROGRESS.
- Created branch task/distribute-builtin-subagents-skill.
- Created worktree /home/ineersa/projects/agent-core-worktrees/distribute-builtin-subagents-skill.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/distribute-builtin-subagents-skill.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/distribute-builtin-subagents-skill.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/distribute-builtin-subagents-skill.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/distribute-builtin-subagents-skill.
- Summary: Implementation authorized after task-explain. Final scope is the simple convention-based materializer plus a verified, concise subagents skill; earlier lock/atomic/preservation requirements are superseded.

## Task workflow update - 2026-08-06T02:16:06.631Z
- Recorded fork run: 6wzbhcsfr6rl
- Validation: Mandatory testing skill and `tests/AGENTS.md` read by implementation/follow-up forks; `castor test --filter=SkillDiscoveryTest`: PASS (21 tests / 74 assertions); Full `castor test`: PASS (4450 tests / 16657 assertions); `castor deptrac`: PASS (0 violations); `castor phpstan`: PASS (0 errors); `castor cs-check`: PASS; `castor phar:build`: PASS; PHAR smoke: PASS (7 tests / 89 assertions); Bundled subagents skill: 10,495 characters / 10,531 bytes, below 20,000-character requirement; No repository-relative links in bundled skill; Worktree clean at `957cc52f0bf8bded471ec0a87eda86fe1fbda5c0`
- Summary: Implementation complete and committed on `task/distribute-builtin-subagents-skill`: `bc96484d3` adds convention-based built-in skill materialization and the bundled subagents skill; `48bb557fd` corrects five fact-audit issues in the skill; `957cc52f0` aligns directly related canonical agent docs with runtime. Final worktree is clean. No push, PR, task transition, or full castor check performed.
- Precision follow-up fork `h191z7uscilx` committed `48bb557fd`: corrected child user-context channel, durable timeout semantics, catalog-only handoffFormat, retrieve range, and model fallback wording.
- Canonical-doc follow-up fork `vr683bws3hwr` committed `957cc52f0`: corrected directly contradictory `docs/agents.md` field, HITL/live-view, timeout, retrieval, tool-policy, and prompt-channel statements.
- Read-only three-scout settings/frontmatter/policy audit found separate cleanup candidates; no cleanup was folded into this finalized task.

## Task workflow update - 2026-08-06T02:21:06.764Z
- Summary: User explicitly expanded the current task to include the completed settings/frontmatter cleanup audit rather than create a follow-up. Approved cleanup: remove dead agent frontmatter (`maxDepth`, `backgroundAllowed`, `handoffFormat`, legacy top-level `mcp`); keep only `inheritAgentsMd` and remove duplicate `inheritProjectContext`; remove `foregroundAllowed` in favor of `disabled`; remove singular `skill` alias in favor of `skills`; stop accepting unused `agents.extensions.enabled` while preserving `forks.extensions.enabled`; hardcode `agent_retrieve` limits at 20/100/240 and remove `agents.retrieve` settings/config plumbing. Preserve active tool MCP selectors, extension security/filtering, `agents.extensions.always_on`, `agents.subagent_excluded_tools`, and other active agent settings/frontmatter.
- Scope expansion approved after the read-only policy/settings audit. This supersedes the previous work-log note that cleanup would remain separate. No backward-compatibility aliases or migration shims are wanted during active development.

## Task workflow update - 2026-08-06T02:40:28.916Z
- Recorded fork run: umaulgae9alq
- Validation: Fork read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before implementation and used Castor-only QA; Focused parser/context/config/retrieval/launch tests: PASS; `castor test`: PASS (4435 tests / 16615 assertions); `castor deptrac`: PASS (0 violations); `castor phpstan`: PASS (0 errors); `castor cs-check`: PASS after `castor cs-fix`; `castor phar:build`: PASS; `castor test --filter=PharSmokeTest`: PASS (7 tests / 89 assertions); Bundled skill: 10,274 characters, below 20,000-character limit; no repository-relative links; `castor test:llm-real`: intentionally skipped because agent_retrieve schema remains identical (1–100, default 20); only its constant source changed; Worktree clean at `4ad801267`
- Summary: User-approved in-task cleanup completed at `4ad801267`: removed dead agent frontmatter/settings and four unused types, collapsed inheritance to `inheritAgentsMd`, made `disabled` the sole launch switch, rejected singular `skill` and unused `agents.extensions.enabled`, fixed `agent_retrieve` bounds at shared 20/100/240 constants, and updated bundled skill/docs/tests. Net change: 112 insertions, 719 deletions across 40 files (-607 lines). Active MCP selectors, extension security/filtering, materializer, and retained settings remain unchanged. Worktree clean; no push, PR, task transition, live LLM run, or full gate.
- Verified branch tip, clean worktree, commit history, and diff stat after fork `umaulgae9alq`: commit `4ad801267 refactor(agents): drop dead frontmatter and fixed retrieve bounds`; 40 files, 112 insertions, 719 deletions.

## Task workflow update - 2026-08-06T02:44:30.362Z
- Recorded fork run: j7n9f5ooi3tb
- Validation: Diff inspected: one file, two comment lines replaced; No tests or CS run warranted for YAML-comment-only change; Verified clean worktree at `fc36f4fb6`
- Summary: Final comment-only accuracy fix committed as `fc36f4fb6`: `config/hatfield.defaults.yaml` now describes `agents.subagent_tool_timeout_seconds` as the durable deferred-batch deadline with scheduled interruption rather than a SubagentExecutionService poll timeout. No runtime/settings/schema change. Worktree clean.
- Follow-up fork `j7n9f5ooi3tb` corrected the only stale removed-semantics wording found by the post-cleanup exact identifier/timeout search.

## Task workflow update - 2026-08-06T02:45:56.041Z
- Summary: User explicitly removed the per-definition `disabled` option from the finalized surface. Agent definitions are now enabled by existence and valid parsing; deleting/moving the definition file is the way to remove it. Remove `disabled` from frontmatter, DTO/parser/catalog/launch paths, bundled skill, docs/default comments, and tests; reject `disabled` as unknown. Preserve global `agents.enabled`. No compatibility alias or migration shim.
- Latest user clarification supersedes the prior decision to keep `disabled` as the sole per-definition launchability switch.

## Task workflow update - 2026-08-06T02:51:25.307Z
- Recorded fork run: txuh7ear7htn
- Validation: Fork read testing skill and `tests/AGENTS.md`; Castor-only QA; Focused catalog/parser/discovery/context/subagent/compact-header tests: PASS (100 tests / 228 assertions); `castor test`: PASS (4427 tests / 16592 assertions); `castor deptrac`: PASS (0 violations); `castor phpstan`: PASS (0 errors); `castor cs-check`: PASS after `castor cs-fix`; `castor phar:build` and `PharSmokeTest`: PASS (7 tests / 89 assertions); Bundled skill: 10,344 characters; no repository-relative links; Live LLM skipped because no provider/tool-schema change; Verified clean worktree at `e87f2e965`
- Summary: Per-definition `disabled` removed at `e87f2e965`: valid discovered definitions are always listed/launchable; delete or move the file to remove one. Redundant catalog `enabled()`/`disabled()`/`requireEnabled()` APIs collapsed to `all()`/`require()`. Global `agents.enabled` remains. 21 files, 51 insertions, 257 deletions; worktree clean.
- Verified fork `txuh7ear7htn` commit/history/diff stat. `disabled` now fails frontmatter parsing as an unknown field; no compatibility path.

## Task workflow update - 2026-08-06T02:51:36.580Z
- Summary: User corrected the duplicate context-flag cleanup: retain the broader `inheritProjectContext` name and remove `inheritAgentsMd`, not vice versa. The sole flag controls inheritance of the parent `agents_context` (resolved AGENTS.md hierarchy) as child user-context. Skills remain explicit and parent agent-catalog context remains excluded.
- Context-flag rename/semantic correction is approved but not yet implemented; hold for the next narrow fork, potentially combined with the pending `systemPromptMode` simplification decision.

## Task workflow update - 2026-08-06T03:00:53.906Z
- Summary: User finalized two immediate corrections: (1) retain `systemPromptMode`; `append` is useful selectively because APPEND_SYSTEM is static and can be irrelevant when JetBrains MCP is unavailable, (2) clean only Hatfield definitions under `~/.hatfield/agents`—never touch Pi/shared `~/.agents`. Hatfield modes: scout/reviewer/architect `append`; researcher/browser `replace`; remove deleted `foregroundAllowed` from all; architect singular `skill` becomes `skills`. Also implement the previously approved code correction: sole context flag is `inheritProjectContext`, not `inheritAgentsMd`.
- User additionally proposed expanding distribution to bundled agent definitions plus a Symfony CLI initializer that copies bundled agents from `src` to `~/.hatfield/agents`. Public command name, overwrite/update safety, and portable contents/dependencies remain unresolved and require task-explain before implementation.

## Task workflow update - 2026-08-06T03:07:46.460Z
- Summary: User finalized bundled agent distribution in this task: ship all five opinionated Hatfield definitions (`scout`, `reviewer`, `researcher`, `architect`, `browser`) under `src/CodingAgent/Resources/agents/`, preserving their explicit model/MCP/skill dependencies. Add public Symfony command `agents:init` to copy them into `~/.hatfield/agents/`. Default behavior must fail before copying anything if any bundled target filename already exists. `--force` overwrites those five bundled filenames. Never delete or overwrite unrelated agent files. Source/PHAR/static resource resolution must use existing application resource/path services; no automatic startup materialization.
- Overwrite policy approved: collision is an error by default, not skip; `--force` explicitly overwrites bundled names. Preflight all collisions before writes to avoid partial default runs. All five opinionated definitions are intentionally accepted despite external model/MCP/skill dependencies.

## Task workflow update - 2026-08-06T03:09:40.973Z
- Recorded fork run: impz9uri0edv
- Validation: Focused context/parser tests: PASS (84 tests / 217 assertions); `castor test`: PASS (4427 tests / 16592 assertions); `castor deptrac`: PASS (0 violations); `castor phpstan`: PASS (0 errors); `castor cs-check`: PASS; `castor phar:build` and `PharSmokeTest`: PASS (7 tests / 89 assertions); Bundled skill: 10,505 characters; no repository-relative links; Verified clean worktree at `d2f60fd64` and inspected final five Hatfield frontmatters; Verified `/home/ineersa/.agents` untouched
- Summary: Context flag correction committed at `d2f60fd64`: `inheritProjectContext` is the sole live/default-true flag and gates parent `agents_context`; `inheritAgentsMd` is rejected unknown. Five external Hatfield definitions under `~/.hatfield/agents` were cleaned to non-default keys only: append for scout/reviewer/architect, replace-by-default for researcher/browser, plural architect `skills`; `~/.agents` remained untouched. Worktree clean.
- Fork `impz9uri0edv` completed the dirty predecessor worktree, stripped default-only local Hatfield frontmatter, and committed only repository changes.

## Task workflow update - 2026-08-06T03:18:09.233Z
- Recorded fork run: hgapjriibjhn
- Validation: Fork read testing skill and `tests/AGENTS.md`; Castor-only QA; `castor test --filter=AgentsInitCommandTest`: PASS (3 tests / 99 assertions); `castor test --filter=ConsoleEntrypointUxTest`: PASS (2 tests / 29 assertions); Full `castor test`: first run hit existing ConsumerSupervisorTest ParaTest flake; focused rerun PASS (5/39), full rerun PASS (4430/16692); `castor deptrac`: PASS (0 violations); `castor phpstan`: PASS (0 errors); `castor cs-check`: PASS; `castor phar:build`: PASS; `castor test --filter=PharSmokeTest`: PASS (7 tests / 131 assertions), including isolated-HOME `agents:init` from PHAR; All five bundled files byte-identical to cleaned `~/.hatfield/agents` sources; Bundled subagents skill: 10,725 characters; no repository-relative links; Live LLM skipped because no provider/tool schema changed; Verified commit/diff, clean worktree at `89b36a504`, and bundled-agent equality
- Summary: Bundled agent distribution completed at `89b36a504`: all five cleaned opinionated Hatfield definitions live under `src/CodingAgent/Resources/agents/`; public `agents:init` installs them into `~/.hatfield/agents`, fails before writes on any collision, and `--force` overwrites only bundled names while preserving unrelated agents. Source/PHAR packaging and isolated-HOME initialization are covered. 13 files, 671 insertions, 2 deletions; worktree clean.
- PHAR resource enumeration learning: `glob()` returned no `phar://` matches; implementation uses `scandir()` with deterministic filtering/sort.
- Opinionated dependency caveat documented: initialized agents retain explicit models, external skills/CLI, and MCP selectors by user decision.

## Task workflow update - 2026-08-06T14:52:49.256Z
- Validation: Reviewer verdict: APPROVED; specification fidelity APPROVED (17/17 finalized requirements mapped); Reviewer confirmed root `AGENTS.md`, testing skill, and `tests/AGENTS.md` read; Task-to-PR `castor test`: PASS (4430 tests / 16692 assertions, 23.7s); Task-to-PR `castor deptrac`: PASS (0 violations); Task-to-PR `castor phpstan`: PASS (0 errors); Task-to-PR `castor cs-check`: PASS (0 files); Worktree clean at `89b36a5043f1004efd8f2a5ae128af894813bf2b`
- Summary: Task-to-PR review APPROVED at `89b36a504`. Reviewer verified all finalized external surfaces and specification-fidelity requirements, security boundaries, source/PHAR behavior, collision semantics, parser cleanup, and Ponytail minimality. One non-blocking Markdown ordered-list numbering nit and one harmless SymfonyStyle input nice-to-have were intentionally not churned. Branch is clean, 8 commits ahead / 4 behind origin/main; no history rewrite/rebase performed because reviewer found no blocker and overlapping main changes should merge cleanly.
- Reviewer non-blocking note: bundled SKILL.md raw ordered list repeats `3.`; Markdown renders correctly. Skipped as cosmetic-only.
- Reviewer merge note: branch base is four commits behind origin/main; likely overlapping import removals auto-resolve. No rebase because history rewrite requires explicit approval and is unnecessary before PR.

## Task workflow update - 2026-08-06T14:54:50.637Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 300s)...
- castor check passed (106.2s).
- Pushed task/distribute-builtin-subagents-skill to origin.
- branch 'task/distribute-builtin-subagents-skill' set up to track 'origin/task/distribute-builtin-subagents-skill'.
- Created PR: https://github.com/ineersa/agent-core/pull/367
- Validation: Reviewer APPROVED; specification fidelity 17/17; castor test PASS 4430/16692; castor deptrac PASS; castor phpstan PASS; castor cs-check PASS
- Summary: Reviewer APPROVED the complete branch and all focused Castor validation passed. Moving to CODE-REVIEW for deterministic full gate, push, and PR creation.

## Task workflow update - 2026-08-06T20:12:11.655Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Starting review iteration for PR #367. Seven inline comments classified: one behavior change approved by the user (`parallelAllowed: false` for bundled browser/researcher); six are explanation/minimality questions. Production MCP resolver confirms `mcp:*` would be redundant for scout/architect/reviewer because explicit tool lists already inherit all `availability: all` MCP tools. Existing builder/policy methods retain real guards/error translation and should not be inlined into multiple callers.

## Task workflow update - 2026-08-06T20:17:22.934Z
- Validation: Implementation fork confirmed root AGENTS.md, testing skill, and tests/AGENTS.md read; `castor test --filter=AgentsInitCommandTest`: PASS (3 tests / 104 assertions); `castor phpstan`: PASS (0 errors); `castor cs-check`: PASS; Re-review verdict: APPROVED; specification fidelity APPROVED; Worktree clean at `cb986ae21bdde36163553286b78c7e31ff36e9b9`
- Summary: PR #367 review iteration implemented at `cb986ae21`: bundled researcher and browser now explicitly disallow parallel batch launch while retaining single launch. Existing bundled-definition parser test pins false for those two and default true for scout/reviewer/architect. Re-review APPROVED; all other inline comments were verified as explanation-only rather than code churn.
- PR comment classification: retrieval constants are fixed output bounds (20 default rows, 100 maximum rows, 240 history-summary characters); no code change.
- PR comment classification: AgentsContextBuilder retains the global agents.enabled guard and central catalog rendering; inlining would duplicate behavior across parent/child consumers; no code change.
- PR comment classification: SubagentLaunchDefinitionPolicyService::requireForegroundDefinition translates catalog RuntimeException into non-retryable ToolCallException; it is not a one-line delegate; no code change.
- PR comment classification: explicit tool lists already inherit MCP tools from servers with availability: all. Adding mcp:* to scout/reviewer/architect would not add tools and would only change metadata mode from inherited_global to all; intentionally skipped.

## Task workflow update - 2026-08-06T20:21:11.841Z
- Validation: `castor cs-fix --path tests/CodingAgent/CLI/AgentsInitCommandTest.php`: qualified `sprintf` only; `castor cs-check`: PASS; `castor test --filter=AgentsInitCommandTest`: PASS (3 tests / 104 assertions); Final formatter-only re-review: APPROVED; Worktree clean at `8cb600b531bda5067e1ecddedd3e7610346fd906`
- Summary: First CODE-REVIEW retry exposed one CS-only issue missed by the implementation fork's earlier claim: unqualified `sprintf` in the updated test. Fixed at `8cb600b53` via Castor formatter; final re-review APPROVED with zero semantic/spec impact.
- CODE-REVIEW transition attempt failed closed before push: full deterministic gate reported cs-check exit 8 on AgentsInitCommandTest.
- Formatter-only fix changes `sprintf` to `\sprintf`; approved behavior map remains unchanged.

## Task workflow update - 2026-08-06T20:23:10.781Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 300s)...
- castor check passed (107.3s).
- Pushed task/distribute-builtin-subagents-skill to origin.
- branch 'task/distribute-builtin-subagents-skill' set up to track 'origin/task/distribute-builtin-subagents-skill'.
- PR already exists: https://github.com/ineersa/agent-core/pull/367
- Validation: AgentsInitCommandTest PASS 3/104; PHPStan PASS; CS check PASS; Re-review APPROVED
- Summary: PR feedback fix and CS follow-up are complete at `8cb600b53`; semantic and formatter-only re-reviews both APPROVED. Re-running deterministic gate, pushing, and updating PR #367.

## Task workflow update - 2026-08-06T20:39:45.097Z
- Moved CODE-REVIEW → DONE.
- Merged task/distribute-builtin-subagents-skill into integration checkout.
- Auto-merging config/services.yaml
Auto-merging tests/CodingAgent/Agent/Artifact/AgentArtifactRetrievalServiceTest.php
Auto-merging tests/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Launch/DeferredSubagentBatchLaunchTest.php
Auto-merging tests/Tui/Screen/TuiSkillReadCardVirtualRenderTest.php
Merge made by the 'ort' strategy.
 .hatfield/skills/subagents/FRONTMATTER.md          |  98 ------
 .hatfield/skills/subagents/SKILL.md                | 102 ------
 bin/console                                        |   1 +
 config/hatfield.defaults.yaml                      |  14 +-
 config/services.yaml                               |   5 -
 docs/agents.md                                     | 104 +++---
 docs/settings.md                                   |   6 +
 .../Artifact/AgentArtifactRetrievalService.php     |  12 +-
 .../Agent/Context/AgentsContextBuilder.php         |  12 +-
 .../Agent/Definition/AgentDefinitionCatalog.php    |  54 +---
 .../Agent/Definition/AgentDefinitionDTO.php        |   7 -
 .../Agent/Definition/AgentDefinitionDiscovery.php  |   4 +-
 .../Agent/Definition/AgentDefinitionParser.php     |  47 +--
 .../Agent/Definition/AgentFrontmatterDTO.php       |  46 ---
 .../Agent/Definition/McpAgentModeEnum.php          |  20 --
 .../Agent/Definition/McpFrontmatterDTO.php         |  50 ---
 src/CodingAgent/Agent/Definition/McpPolicyDTO.php  |  24 --
 .../Agent/Execution/AgentPromptBuilder.php         |   2 +-
 .../SubagentChildLaunchInputFactory.php            |   4 +-
 .../SubagentLaunchDefinitionPolicyService.php      |   8 +-
 .../Agent/Fork/ForkInternalAgentDefinition.php     |   6 -
 .../Agent/Fork/ForkToolPolicyResolver.php          |   4 -
 src/CodingAgent/Agent/Tool/AgentRetrieveTool.php   |   4 +-
 src/CodingAgent/CLI/AgentsInitCommand.php          | 122 ++++++++
 .../Config/AgentArtifactRetrievalLimitsConfig.php  |  63 ----
 src/CodingAgent/Config/AgentsConfig.php            |   7 +-
 src/CodingAgent/Config/AppResourceLocator.php      |  23 ++
 .../Config/ChildExtensionsConfigDTO.php            |  10 +-
 src/CodingAgent/Resources/agents/architect.md      |  46 +++
 src/CodingAgent/Resources/agents/browser.md        |  30 ++
 src/CodingAgent/Resources/agents/researcher.md     |  77 +++++
 src/CodingAgent/Resources/agents/reviewer.md       | 135 ++++++++
 src/CodingAgent/Resources/agents/scout.md          |  42 +++
 .../Resources/skills/subagents/FRONTMATTER.md      |  97 ++++++
 .../Resources/skills/subagents/SKILL.md            |  86 +++++
 .../LoadedResourcesSummaryBuilder.php              |   1 -
 src/CodingAgent/Skills/SkillDiscovery.php          |  58 +++-
 .../CompactHeaderSnapshotProvider.php              |   3 +-
 .../Artifact/AgentArtifactRetrievalServiceTest.php |   2 -
 .../Agent/Context/AgentContextRendererTest.php     |   8 +-
 .../Agent/Context/AgentsContextBuilderTest.php     |  28 +-
 .../Definition/AgentDefinitionCatalogTest.php      | 106 +------
 .../Definition/AgentDefinitionDiscoveryTest.php    |  26 +-
 .../Agent/Definition/AgentDefinitionParserTest.php | 348 +++------------------
 .../Agent/Execution/AgentPromptBuilderTest.php     |   3 -
 .../Execution/AgentToolPolicyResolverTest.php      |   3 -
 ...05BareAgentsEffectiveContextIntegrationTest.php |   3 -
 ...AppendStructuralPermanentSubsetContractTest.php |   3 -
 .../Launch/DeferredSubagentBatchLaunchTest.php     |   6 -
 .../SubagentChildExtensionMetadataTest.php         |   3 -
 .../SubagentChildLaunchModelInheritanceTest.php    |   3 -
 .../Execution/SubagentExecutionServiceTest.php     |  52 +--
 .../SubagentPromptUserContextContractTest.php      |   5 -
 tests/CodingAgent/CLI/AgentsInitCommandTest.php    | 171 ++++++++++
 tests/CodingAgent/CLI/ConsoleEntrypointUxTest.php  |   1 +
 tests/CodingAgent/Config/AgentsConfigTest.php      |  13 +
 .../ChildExtensionSelectionServiceTest.php         |   3 -
 .../Extension/ExtensionOwnerFilteringTest.php      |   4 +
 .../Extension/ExtensionToolRegistryBridgeTest.php  |   4 +
 .../FileRewindExtensionIntegrationTest.php         |   4 +
 tests/CodingAgent/Phar/PharSmokeTest.php           |  47 +++
 .../Controller/E2E/SubagentParallelLiveE2eTest.php |   6 -
 .../ParentPromptUserContextRegressionTest.php      |   2 +-
 .../LoadedResourcesSummaryBuilderTest.php          |  10 +-
 .../Projection/SkillReadProjectionReplayTest.php   |   4 +
 tests/CodingAgent/Skills/SkillDiscoveryTest.php    | 101 ++++++
 .../Skills/SkillsContextBuilderTest.php            |   4 +
 .../CompactHeaderSnapshotProviderTest.php          |  16 +-
 tests/Tui/Listener/CompactHeaderRegistrarTest.php  |   4 +
 .../LoadedResourcesStartupRegistrarTest.php        |   3 +
 .../Screen/TuiSkillReadCardVirtualRenderTest.php   |   4 +
 71 files changed, 1271 insertions(+), 1163 deletions(-)
 delete mode 100644 .hatfield/skills/subagents/FRONTMATTER.md
 delete mode 100644 .hatfield/skills/subagents/SKILL.md
 delete mode 100644 src/CodingAgent/Agent/Definition/McpAgentModeEnum.php
 delete mode 100644 src/CodingAgent/Agent/Definition/McpFrontmatterDTO.php
 delete mode 100644 src/CodingAgent/Agent/Definition/McpPolicyDTO.php
 create mode 100644 src/CodingAgent/CLI/AgentsInitCommand.php
 delete mode 100644 src/CodingAgent/Config/AgentArtifactRetrievalLimitsConfig.php
 create mode 100644 src/CodingAgent/Resources/agents/architect.md
 create mode 100644 src/CodingAgent/Resources/agents/browser.md
 create mode 100644 src/CodingAgent/Resources/agents/researcher.md
 create mode 100644 src/CodingAgent/Resources/agents/reviewer.md
 create mode 100644 src/CodingAgent/Resources/agents/scout.md
 create mode 100644 src/CodingAgent/Resources/skills/subagents/FRONTMATTER.md
 create mode 100644 src/CodingAgent/Resources/skills/subagents/SKILL.md
 create mode 100644 tests/CodingAgent/CLI/AgentsInitCommandTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/distribute-builtin-subagents-skill.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/distribute-builtin-subagents-skill.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #367 state: MERGED at 2026-08-06T20:39:05Z; PR head: `8cb600b531bda5067e1ecddedd3e7610346fd906`; CODE-REVIEW `castor check`: PASS (107.3s)
- Summary: PR #367 merged on GitHub as `98d444f9ba3936c2a6a0f35af8a68c497b8dae07`. Reviewer and deterministic CODE-REVIEW gate approved; completing local integration and worktree cleanup.

## Task workflow update - 2026-08-06T20:42:04.118Z
- Validation: Post-merge `LLM_MODE=true castor check`: PASS (357.7s); deptrac PASS (1.0s); test PASS (4426 tests / 16689 assertions, 58.1s); controller replay PASS (12 tests / 165 assertions, 100.9s); TUI PASS (34 tests / 216 assertions, 106.9s); llm-real PASS (13 tests / 144 assertions, 81.0s); PHPStan PASS (0 errors, 7.5s); CS check PASS (2.1s); QA artifact integrity PASS; leak check PASS; llama-proxy cache stable 225→225; Integration checkout clean; worktree removed
- Summary: Post-merge integration validation completed successfully. PR #367 is merged; task is DONE; worktree and IDEA exclusions are removed. Integration checkout is clean at `d56d29a82` and intentionally not pushed because local main contains four local merge commits beyond origin/main.

## Task workflow update - 2026-08-06T20:58:47.839Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
