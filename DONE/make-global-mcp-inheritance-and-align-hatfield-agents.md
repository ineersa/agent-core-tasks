# Make global MCP inheritance explicit and align Hatfield agents with JetBrains MCP

## Goal
User-approved follow-up to align Hatfield MCP availability semantics, IDE tooling, system guidance, and user agent definitions.

Desired policy:
- MCP servers configured with `availability: all` are inherited by every child agent, including agents with an explicit `tools:` list.
- `mcp:-` explicitly suppresses all MCP inheritance for that child.
- `availability: specific` tools remain opt-in through `mcp:` selectors.
- Explicit specific selectors add selected tools to the globally inherited MCP set.
- Raw MCP runtime names without the `mcp:` selector prefix must not bypass specific availability.

JetBrains IDE MCP:
- Copy the supported Hatfield subset of `.pi/mcp.json` server `jetbrains-index` into `.hatfield/mcp.json` with `availability: all`.
- URL: `http://127.0.0.1:29175/index-mcp/streamable-http`.
- Hatfield's normal `{server}_{tool}` namespacing is accepted. Runtime names will be `jetbrains-index_ide_*`.
- Do not copy unsupported Pi adapter fields (`toolPrefix`, `directTools`, `exposeResources`).

System guidance:
- Copy/adapt project `.pi/APPEND_SYSTEM.md` into project `.hatfield/APPEND_SYSTEM.md`.
- Rewrite every bare Pi `ide_*` tool reference to the corresponding Hatfield runtime name `jetbrains-index_ide_*`.
- Audit against the actual JetBrains MCP catalog; remove or correct stale tool references such as `ide_open_file` if the server does not expose them.

Hatfield user agents:
- Audit and update `~/.hatfield/agents/scout.md`, `researcher.md`, `reviewer.md`, and `architect.md` so frontmatter and body instructions reference only real Hatfield tools.
- Scout keeps explicit non-MCP tools `read`, `bash`, `view_image`; it receives Context7 and JetBrains through global MCP inheritance and must not receive availability-specific websearch.
- Researcher keeps `read`, `bash` plus explicit exact selectors for `websearch_search`, `websearch_open`, and `websearch_find`; it also receives global Context7 and JetBrains.
- Reviewer and architect must have explicit least-privilege non-MCP allowlists appropriate to their read-only roles, remove nonexistent direct `grep`/`find`/`ls` tool entries where those are shell commands rather than Hatfield tools, and use namespaced JetBrains tool names in instructions.
- Do not modify Pi's separate `~/.agents/*.md` definitions.

Dependencies/order:
- Depends on PR #304 (`fix-child-mcp-specific-availability-leak`) because that establishes catalog MCP classification during child policy merge.
- Coordinate with and preferably land `replace-mcp-underscore-prefix-with-terminal-star-wildcard` first because both tasks modify `AgentMcpToolsResolver`, its tests, and agent documentation.

This changes LLM-visible tool schemas/policy and requires full Castor validation plus focused live LLM validation.

## Acceptance criteria
- Explicit child `tools:` lists inherit all `availability: all` MCP runtime tools by default.
- `mcp:-` suppresses global and specific MCP tools, while `mcp:*` still permits the complete catalog.
- Explicit specific selectors merge selected `availability: specific` tools with the inherited global MCP set.
- A raw MCP runtime name without `mcp:` cannot bypass MCP availability/selector policy.
- `.hatfield/mcp.json` configures `jetbrains-index` at the existing local HTTP endpoint with `availability: all` using only Hatfield-supported fields.
- Hatfield discovers JetBrains tools under namespaced runtime names `jetbrains-index_ide_*`.
- Project `.hatfield/APPEND_SYSTEM.md` contains the adapted Pi IDE guidance using only actual namespaced Hatfield tool names.
- Hatfield scout, researcher, reviewer, and architect definitions have explicit appropriate non-MCP allowlists and instructions consistent with their effective tools; Pi `~/.agents` files remain unchanged.
- Focused resolver/policy tests cover explicit global inheritance, global-plus-specific selection, `mcp:-`, `mcp:*`, and raw-name bypass prevention.
- Public/internal MCP and agent documentation describes the updated availability behavior and matches the terminal-star selector grammar task.
- Castor test, Deptrac, PHPStan, coding-style, full `castor check`, and focused `castor test:llm-real` pass.

## Workflow metadata
Status: DONE
Branch: task/make-global-mcp-inheritance-and-align-hatfield-agents
Worktree: /home/ineersa/projects/agent-core-worktrees/make-global-mcp-inheritance-and-align-hatfield-agents
Fork run: 7xb63d7ei24f
PR URL: https://github.com/ineersa/agent-core/pull/309
PR Status: merged
Started: 2026-07-21T14:10:29.969Z
Completed: 2026-07-21T16:22:56.555Z

## Work log
- Created: 2026-07-20T23:48:13.251Z

## Task workflow update - 2026-07-21T14:10:29.969Z
- Moved TODO → IN-PROGRESS.
- Created branch task/make-global-mcp-inheritance-and-align-hatfield-agents.
- Created worktree /home/ineersa/projects/agent-core-worktrees/make-global-mcp-inheritance-and-align-hatfield-agents.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/make-global-mcp-inheritance-and-align-hatfield-agents.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/make-global-mcp-inheritance-and-align-hatfield-agents.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/make-global-mcp-inheritance-and-align-hatfield-agents.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/make-global-mcp-inheritance-and-align-hatfield-agents.
- Summary: Starting implementation after dependencies PR #304 and PR #306 merged. Scope includes explicit global MCP inheritance semantics, raw-name bypass prevention, JetBrains MCP Hatfield config/guidance, user Hatfield agent alignment, tests, docs, and focused live-LLM validation.

## Task workflow update - 2026-07-21T14:18:04.720Z
- Recorded fork run: 015z9c9zwjwe
- Summary: Implementation fork launched for catalog-based global MCP inheritance, deny/all precedence, raw-name bypass prevention, JetBrains MCP config and namespaced APPEND_SYSTEM guidance, docs/skill updates, and approved user Hatfield agent alignment. Full castor check intentionally deferred to task-to-pr; focused live-LLM validation required in fork.
- Two scouts confirmed the minimal production boundary is AgentMcpToolsResolver: explicit lists currently return no globals and retain raw catalog runtime names as non-MCP. Recommended exact catalog filtering, global union, deny-first precedence, and stable deduplication. Live jetbrains-index server v4.10.4 exposes 18 tools and no ide_open_file. User scout/researcher/reviewer/architect definitions were audited; Pi ~/.agents hashes recorded for no-change verification.

## Task workflow update - 2026-07-21T14:38:45.843Z
- Recorded fork run: 015z9c9zwjwe
- Validation: Read root AGENTS.md, .agents/skills/testing/SKILL.md, tests/AGENTS.md, and task-workflow skill before implementation/QA; castor test --filter=AgentMcpToolsResolverTest OK (8 tests, 26 assertions); castor test --filter=AgentToolPolicyResolverTest OK (7 tests, 24 assertions); castor test --filter=AgentPromptBuilderTest OK (6 tests, 26 assertions); castor test OK (4447 tests, 15260 assertions); castor deptrac OK (0 violations, 0 errors); castor phpstan OK (0 errors); castor cs-check OK (0 files fixed); castor test:llm-real OK (12 tests, 163 assertions); Repository git diff --check clean; worktree clean at 57e40809721c9c0a1b462ba8ce4cca07d729ec4e; JetBrains APPEND audit: 18 expected names, 18 unique, no missing/unexpected names, no bare ide_* refs, no ide_open_file; Hatfield user agent audit: no bare ide_* or ide_open_file refs; frontmatter verified; Pi ~/.agents SHA-256 hashes unchanged: scout be3c9891fd70d16f6a752edfd542ab24ec1dd06ec0b2551bf290b87d229cf868; researcher 39b99c4968adbadf953b09453a2001deba2ae1f05d0ab6564d260a14c481ddf9; reviewer 248bd793488a052c863e48f3b483f4aa1a92069eca34c3eadd8991ee06e7f7a3; architect 6290606e75a70120586629096261a127b9e00700a598a2c050fea69221b3a3f6
- Summary: Implementation complete and committed on task/make-global-mcp-inheritance-and-align-hatfield-agents at 57e40809721c9c0a1b462ba8ce4cca07d729ec4e. Explicit child tools now inherit availability=all MCPs, mcp:- deny-first, mcp:* full catalog, specific selectors merge globals, and raw catalog runtime names are stripped from non-MCP entries. Added JetBrains MCP config and namespaced .hatfield/APPEND_SYSTEM.md; updated agent/MCP docs and subagent guidance. Approved user Hatfield scout/reviewer/architect files aligned; researcher was already compliant and unchanged. Pi ~/.agents hashes verified unchanged. Worktree clean. Full castor check intentionally deferred to task-to-pr per instructions.
- First focused AgentMcpToolsResolver test run exposed expected semantic update in prior terminal-star tests: explicit selector cases now correctly include inherited global context7_resolve; adjusted assertions to preserve exact/prefix regression thesis while proving globals remain inherited.
- No full castor check run, as explicitly prohibited during task-start; task-to-pr owns the deterministic full gate.

## Task workflow update - 2026-07-21T14:39:25.799Z
- Validation: Additional focused config/system-prompt checks: castor test --filter=McpConfigLoaderTest OK (22 tests, 59 assertions); castor test --filter=SystemPromptBuilderTest OK (22 tests, 74 assertions); Post-validation git status clean; HEAD remains 57e40809721c9c0a1b462ba8ce4cca07d729ec4e

## Task workflow update - 2026-07-21T14:41:57.102Z
- Recorded fork run: 015z9c9zwjwe
- Validation: Fork read root AGENTS.md, testing skill, tests/AGENTS.md, and task-workflow skill before implementation/QA; castor test --filter=AgentMcpToolsResolverTest OK (8 tests, 26 assertions); castor test --filter=AgentToolPolicyResolverTest OK (7 tests, 24 assertions); castor test --filter=AgentPromptBuilderTest OK (6 tests, 26 assertions); castor test --filter=McpConfigLoaderTest OK (22 tests, 59 assertions); castor test --filter=SystemPromptBuilderTest OK (22 tests, 74 assertions); castor test OK (4447 tests, 15260 assertions); castor deptrac OK (0 violations); castor phpstan OK (0 errors); castor cs-check OK; castor test:llm-real OK (12 tests, 163 assertions); JetBrains APPEND audit: exactly 18 expected names, no missing/unexpected names, no bare ide_* or ide_open_file; Pi ~/.agents scout/researcher/reviewer/architect SHA-256 hashes unchanged; git diff --check main...HEAD clean; Verified commit 57e408097 and clean worktree; 10 expected repository files changed
- Summary: Implementation completed and committed at 57e40809721c9c0a1b462ba8ce4cca07d729ec4e. Explicit child tool lists now inherit availability:all MCP tools; mcp:- suppresses all and wins metadata precedence; mcp:* exposes the complete catalog; specific selectors add to globals; raw catalog runtime names are stripped from non-MCP entries. Added jetbrains-index as availability:all, created namespaced .hatfield/APPEND_SYSTEM.md using the exact 18 live tools, and updated agent/MCP docs and subagent guidance. External user Hatfield scout, reviewer, and architect definitions were aligned; researcher was already compliant. Pi ~/.agents files remained unchanged. Repository worktree is clean.
- External user configuration changed outside the repo: /home/ineersa/.hatfield/agents/scout.md tools now read/bash/view_image with namespaced IDE instructions; reviewer.md tools now read/bash with namespaced IDE instructions; architect.md now has read/bash with namespaced IDE instructions. researcher.md already had read/bash plus exact websearch selectors and was unchanged.
- Full deterministic castor check intentionally not run in task-start phase; task-to-pr owns review and gate.

## Task workflow update - 2026-07-21T15:12:16.354Z
- Recorded fork run: itp6yro03gnj
- Summary: PR-readiness reviewer verdict APPROVED with no blockers. Accepted follow-up improvements before final re-review: derive effective MCP tools from one computed policy mode to remove duplicate precedence/unreachable branches; clarify raw catalog-name docs; explicitly prohibit mutating JetBrains refactor/move tools in external read-only reviewer/architect instructions. Fix fork itp6yro03gnj launched.
- Reviewer read AGENTS.md, testing skill, tests/AGENTS.md, task file, all changed production/tests/docs/config, relevant MCP availability/config paths, and final external Hatfield agent definitions. Verified truth table and old-code regression value. No TUI proof required.
- Reviewer non-accepted notes: raw selector warning is separate UX/parser scope; localhost MCP default 30s timeout is intentional; redundant star/deny assertions conflict with focused test-value budget; APPEND_SYSTEM condensation is acceptable.

## Task workflow update - 2026-07-21T15:13:12.707Z
- Summary: PR-base audit found task branch was created from local main 77353e9a0 while remote main remains e4d334243. The task diff against local base is clean and scoped, but GitHub's current remote-main comparison adds unrelated `.pi/settings.json` from local-only commit 79839d875. Must resolve before PR creation; no history rewrite will be performed without explicit user approval.
- `git ls-remote origin refs/heads/main` = e4d334243d2c1480196ee0a04a8303dfe271bfdd. `git diff --stat origin/main..77353e9a0` shows only `.pi/settings.json` (1 insertion, 1 deletion), from local-only `Update pi fork default model` commit. Reviewer comparison used exact task base 77353e9a0, so task code review remains scoped; actual PR baseline needs cleanup.

## Task workflow update - 2026-07-21T15:20:29.703Z
- Recorded fork run: itp6yro03gnj
- Validation: castor test --filter=AgentMcpToolsResolverTest OK (8 tests, 26 assertions); castor test --filter=AgentToolPolicyResolverTest OK (7 tests, 24 assertions); castor phpstan OK (0 errors); castor cs-check OK; git diff --check clean; Fix fork read AGENTS.md, testing skill, tests/AGENTS.md, and task-workflow skill before changes/QA
- Summary: Accepted review findings implemented in commits 60e343337 and 83f4691f2. AgentMcpToolsResolver now computes policy mode once and derives effective tools/metadata from it; unreachable sentinel handling removed from expandSelectors with explicit precondition. Raw-name docs clarified. External read-only reviewer/architect instructions now prohibit mutating JetBrains rename/move tools. Worktree clean.
- Verified commits 60e343337 and 83f4691f2 exist, expected 4 repository files changed since initial implementation, external reviewer/architect constraints present, and worktree is clean.

## Task workflow update - 2026-07-21T15:29:01.530Z
- Validation: Final reviewer APPROVED; read AGENTS.md, testing skill, tests/AGENTS.md, task, all changed files/supporting paths, and external agent definitions; castor test OK (4447 tests, 15260 assertions); castor deptrac OK (0 violations, 0 errors); castor phpstan OK (0 errors); castor cs-check OK (0 files fixed); castor test:llm-real OK (12 tests, 163 assertions); llama-proxy cache currently warm at 90 entries; git diff --check clean; worktree clean at 83f4691f2
- Summary: Final re-review of HEAD 83f4691f2: APPROVED with zero actionable findings. All accepted fixes verified; original MCP truth table and provider-facing policy remain correct; exact JetBrains config/guidance and external agent constraints verified. Latest focused PR validation is green. PR transition remains blocked only by unrelated local-main ancestry `.pi/settings.json` pending explicit user approval to rebase unpublished task commits onto origin/main.
- Final reviewer had only non-actionable NTH observations about tighter docblock shape/comment wording; no correctness, security, edge-case, dead-code, docs, config, or test findings remain.
- No TUI behavior touched; no TmuxHarness proof or standalone castor test:tui required beyond the automatic deterministic gate.

## Task workflow update - 2026-07-21T15:33:32.817Z
- Summary: User explicitly approved pushing inherited local commit 79839d875 (`.pi/settings.json`: pi-fork defaultModel grok-cli/grok-composer-2.5-fast → openai-codex/gpt-5.6-luna) with this branch. No rebase needed; proceed to CODE-REVIEW with the extra settings change disclosed in PR.
- PR scope exception approved by user: include existing local-only `.pi/settings.json` default-model update inherited from local main.

## Task workflow update - 2026-07-21T15:35:43.207Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 600s)...
- castor check passed (113.0s).
- Pushed task/make-global-mcp-inheritance-and-align-hatfield-agents to origin.
- branch 'task/make-global-mcp-inheritance-and-align-hatfield-agents' set up to track 'origin/task/make-global-mcp-inheritance-and-align-hatfield-agents'.
- Created PR: https://github.com/ineersa/agent-core/pull/309
- Validation: castor test OK (4447 tests, 15260 assertions); castor deptrac OK (0 violations, 0 errors); castor phpstan OK (0 errors); castor cs-check OK (0 files fixed); castor test:llm-real OK (12 tests, 163 assertions); Final reviewer APPROVED; git diff --check clean; worktree clean at 83f4691f2
- Summary: Final reviewer APPROVED with zero actionable findings. Implemented global MCP inheritance for explicit child tool lists, deny-first mcp:- semantics, global-plus-specific selectors, raw catalog-name bypass prevention, JetBrains MCP availability/config and namespaced guidance, maintained docs, and approved external Hatfield agent alignment. User explicitly approved including inherited `.pi/settings.json` pi-fork default-model update in this branch.

## Task workflow update - 2026-07-21T15:35:56.495Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/309
- Updated PR Status: open
- Validation: Deterministic castor check gate passed (113.0s); PR #309 open against main; PR file list audited: 11 expected files only
- Summary: PR #309 created after deterministic castor check passed in 113.0s. GitHub file audit shows exactly the 10 task repository files plus the user-approved `.pi/settings.json` update; no unexpected files.

## Task workflow update - 2026-07-21T15:42:58.207Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User review feedback changes selector semantics: `mcp:*` must expose only `availability: all` tools, never every catalog tool. `availability: specific` tools may appear only when explicitly selected by exact or terminal-star server/tool selectors in the agent definition. PR #309 requires implementation, tests, docs, re-review, and a new gate.

## Task workflow update - 2026-07-21T15:43:45.032Z
- Recorded fork run: 7xb63d7ei24f
- Summary: User correctly rejected PR behavior where `mcp:*` exposed the full catalog. Revised invariant: broad `mcp:*` exposes availability:all tools only; availability:specific tools require an explicit exact or terminal-star prefix selector. Mixed `mcp:*` + specific selector adds only specifically matched tools; mcp:- still wins. Fix fork 7xb63d7ei24f launched for code, focused regression tests, and all maintained docs/guidance.
- PR #309 moved CODE-REVIEW → IN-PROGRESS for user feedback. No GitHub review comments were present; feedback was given directly in session.
- Revised metadata semantics: mcp:* alone mode=all with global-only effective tools; any non-sentinel exact/prefix selector mode=specific; deny mode=none; omitted/no selector mode=inherited_global.

## Task workflow update - 2026-07-21T15:49:34.605Z
- Recorded fork run: 7xb63d7ei24f
- Validation: castor test --filter=AgentMcpToolsResolverTest OK (9 tests, 31 assertions); castor test --filter=AgentToolPolicyResolverTest OK (8 tests, 28 assertions); castor phpstan OK (0 errors); castor cs-check OK; castor test:llm-real OK (12 tests, 163 assertions); git diff --check clean; Fix fork read AGENTS.md, testing skill, tests/AGENTS.md, and task-workflow skill before implementation/QA
- Summary: User-feedback fix completed at 7c5cf53ad. `mcp:*` now resolves to global availability only; exact/terminal-star selectors add specifically matched tools; mixed broad+specific selectors cannot leak unselected specific tools; deny remains first. Provider-facing policy regression and maintained docs updated. Worktree clean, one commit ahead of PR branch.
- Verified commit 7c5cf53ad exists, exactly 7 expected repository files changed, worktree clean/ahead origin by one, and IDE diagnostics report zero errors for AgentMcpToolsResolver.php.

## Task workflow update - 2026-07-21T15:56:23.811Z
- Summary: Re-review of user-feedback commit 7c5cf53ad: APPROVED with no blocking/actionable correctness or security findings. Reviewer verified mcp:* cannot reach empty-prefix expansion, unselected specific tools remain absent, mixed selectors and metadata are correct, provider-facing policy is covered, regression tests fail on prior HEAD, and maintained docs have no stale full-catalog claim.
- Reviewer SIMPLIFY note (identical global seed for all/specific) not accepted: explicit match arms intentionally document distinct policy modes. Defensive filtering of mcp:- in the specific-selector filter retained as harmless safety even though deny mode prevents current reachability. NTH comments/tests skipped under focused test-value budget.
- Reviewer read AGENTS.md, testing skill, tests/AGENTS.md, task, all 7 changed files, and relevant full policy/tool-set consumers. No TUI proof required.

## Task workflow update - 2026-07-21T15:58:08.977Z
- Validation: castor test OK (4449 tests, 15269 assertions); castor deptrac OK (0 violations, 0 errors); castor phpstan OK (0 errors); castor cs-check OK; castor test:llm-real OK (12 tests, 163 assertions); llama-proxy warmup rerun OK; cache stable 91 → 91; git diff --check clean; worktree clean/ahead origin by one; Reviewer APPROVED
- Summary: User-feedback fix at 7c5cf53ad is reviewer APPROVED and fully validated. Ready to update PR #309: mcp:* global-only; specific tools require exact/prefix selectors.

## Task workflow update - 2026-07-21T16:00:14.637Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 600s)...
- castor check passed (111.1s).
- Pushed task/make-global-mcp-inheritance-and-align-hatfield-agents to origin.
- branch 'task/make-global-mcp-inheritance-and-align-hatfield-agents' set up to track 'origin/task/make-global-mcp-inheritance-and-align-hatfield-agents'.
- PR already exists: https://github.com/ineersa/agent-core/pull/309
- Validation: castor test OK (4449 tests, 15269 assertions); castor deptrac OK (0 violations, 0 errors); castor phpstan OK (0 errors); castor cs-check OK; castor test:llm-real OK (12 tests, 163 assertions); llama-proxy cache stable 91 → 91 after warmup; Reviewer APPROVED; git diff --check clean; worktree clean at 7c5cf53ad
- Summary: Updated PR semantics per user feedback: mcp:* exposes availability:all tools only; availability:specific tools require exact/terminal-star selectors; mixed selectors add only explicit matches; deny/raw-name safeguards retained. Reviewer APPROVED and focused validation green.

## Task workflow update - 2026-07-21T16:00:40.180Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/309
- Updated PR Status: open
- Validation: Deterministic castor check passed (111.1s); Branch pushed; PR head 7c5cf53ad; GitHub PR body verified to state mcp:* global-only and current validation counts; PR #309 open
- Summary: PR #309 updated at 7c5cf53ad with corrected user-authoritative semantics: mcp:* is global-only; specific tools require exact/prefix selectors. Deterministic castor check passed in 111.1s. Existing PR body did not auto-refresh during transition, so it was explicitly corrected and verified on GitHub.

## Task workflow update - 2026-07-21T16:22:56.555Z
- Moved CODE-REVIEW → DONE.
- Merged task/make-global-mcp-inheritance-and-align-hatfield-agents into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/APPEND_SYSTEM.md                         | 37 +++++++++++++
 .hatfield/mcp.json                                 |  4 ++
 .hatfield/skills/subagents/FRONTMATTER.md          | 10 ++--
 .hatfield/skills/subagents/SKILL.md                | 17 +++++-
 docs/agents.md                                     | 31 +++++++----
 docs/mcp.md                                        | 24 ++++++++
 .../Agent/Execution/AgentMcpToolsResolver.php      | 64 +++++++++++++++-------
 .../Agent/Execution/AgentToolPolicyResolver.php    |  2 +-
 .../Agent/Execution/AgentMcpToolsResolverTest.php  | 55 ++++++++++++++-----
 .../Execution/AgentToolPolicyResolverTest.php      | 39 +++++++++++--
 10 files changed, 228 insertions(+), 55 deletions(-)
 create mode 100644 .hatfield/APPEND_SYSTEM.md
- Removed worktree /home/ineersa/projects/agent-core-worktrees/make-global-mcp-inheritance-and-align-hatfield-agents.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/make-global-mcp-inheritance-and-align-hatfield-agents.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #309 state MERGED; Merged at 2026-07-21T16:22:27Z; Final PR head 7c5cf53adb9f3dbc603ce09d201a4392533fa39a; Pre-merge deterministic castor check passed (111.1s)
- Summary: PR #309 confirmed merged on GitHub at 990c35256a1355441e132f8e2a09c23c6ff8d8cf. Merge task branch into integration checkout, sync remote, remove worktree, and clean IDEA exclusions.

## Task workflow update - 2026-07-21T16:25:14.382Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/309
- Updated PR Status: merged
- Validation: Post-merge `LLM_MODE=true castor check` OK (QA run qa-20260721-162300-196662-bbe7be4f, 296.0s); deptrac OK; test OK (4444 tests, 15212 assertions); test:controller-replay OK (9 tests, 127 assertions); test:tui OK (36 tests, 185 assertions); test:llm-real OK (12 tests, 163 assertions); phpstan OK (0 errors); cs-check OK; llama-proxy cache guard stable 91 → 91; QA artifact integrity OK (7 lane logs); QA run leak check OK; Integration checkout clean; worktree removed; Pi ~/.agents scout/researcher/reviewer/architect SHA-256 hashes unchanged
- Summary: Task complete. PR #309 merged at 990c35256a1355441e132f8e2a09c23c6ff8d8cf, task branch merged into integration checkout, remote synchronized, worktree removed, and IDEA exclusions cleaned. Post-merge deterministic validation passed. External Hatfield agent tool allowlists remain aligned; Pi ~/.agents hashes remain unchanged.
