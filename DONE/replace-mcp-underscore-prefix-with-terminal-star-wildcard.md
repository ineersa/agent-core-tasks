# Replace MCP underscore-prefix selector with terminal-star wildcard

## Goal
The current child-agent MCP selector grammar treats any selector value ending in `_` as a prefix match (for example `mcp:websearch_`). This is surprising and conflicts with normal wildcard expectations.

Agreed target grammar:
- `mcp:websearch_search` selects that exact runtime tool name.
- `mcp:websearch_` is also an exact selector; underscore has no special meaning.
- `mcp:websearch_*` selects all MCP runtime tool names beginning with `websearch_`.
- `mcp:*` continues to select all MCP tools.
- `mcp:-` continues to select no MCP tools.

Implement only a terminal `*` prefix wildcard, not general globbing. Active-development rule applies: replace the old underscore convention outright; do not add a compatibility shim.

Known affected areas include `AgentMcpToolsResolver::expandSelectors()`, resolver/policy tests, `docs/agents.md`, `internal-docs/agents.md`, and `.hatfield/skills/subagents/FRONTMATTER.md`. Check all selector examples and documentation references. The user-level Hatfield researcher agent now uses three explicit exact selectors and does not depend on prefix syntax.

## Acceptance criteria
- Selectors without `*` are exact matches, including names ending in `_`.
- A selector with one terminal `*`, such as `mcp:websearch_*`, performs prefix matching against catalog runtime tool names.
- `mcp:*` still selects every MCP runtime tool and `mcp:-` still selects none.
- The old implicit trailing-underscore prefix behavior is removed with no backward-compatibility path.
- Focused regression tests prove exact, terminal-wildcard, all, and none behavior through Castor.
- Public/internal agent docs and subagent frontmatter guidance use the new terminal-star syntax and contain no old `mcp:<prefix_>` examples.
- Castor Deptrac, PHPStan, and coding-style validation pass.

## Workflow metadata
Status: DONE
Branch: task/replace-mcp-underscore-prefix-with-terminal-star-wildcard
Worktree: /home/ineersa/projects/agent-core-worktrees/replace-mcp-underscore-prefix-with-terminal-star-wildcard
Fork run: 5ut8i3qds4se
PR URL: https://github.com/ineersa/agent-core/pull/306
PR Status: merged
Started: 2026-07-20T23:59:28.144Z
Completed: 2026-07-21T02:30:28.268Z

## Work log
- Created: 2026-07-20T21:56:25.281Z

## Task workflow update - 2026-07-20T22:00:35.491Z
- Summary: Scope clarification approved by user: implementation must also audit and update Hatfield agent definitions, not only resolver code/tests/docs. Check project `.hatfield/agents/*.md`, user `~/.hatfield/agents/*.md`, and any maintained/bundled Hatfield agent examples for old implicit trailing-underscore prefix selectors; convert intended prefixes to terminal-star syntax or explicit exact selectors. Do not modify Pi's separate `~/.agents/*.md` definitions unless they are independently wrong for Pi.
- User explicitly required Hatfield agent definitions to be updated with the selector grammar change. Current audit: `~/.hatfield/agents/researcher.md` already uses explicit exact selectors (`mcp:websearch_search`, `mcp:websearch_open`, `mcp:websearch_find`) and needs no wildcard migration; `.hatfield/skills/subagents/FRONTMATTER.md` still contains old `mcp:websearch_` prefix examples and remains in implementation scope.

## Task workflow update - 2026-07-20T23:48:17.586Z
- Summary: Coordination note: this task should land before TODO/make-global-mcp-inheritance-and-align-hatfield-agents because both modify AgentMcpToolsResolver, focused tests, and agent docs. The later task will build explicit-global inheritance on the terminal-star grammar.

## Task workflow update - 2026-07-20T23:59:28.144Z
- Moved TODO → IN-PROGRESS.
- Created branch task/replace-mcp-underscore-prefix-with-terminal-star-wildcard.
- Created worktree /home/ineersa/projects/agent-core-worktrees/replace-mcp-underscore-prefix-with-terminal-star-wildcard.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/replace-mcp-underscore-prefix-with-terminal-star-wildcard.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/replace-mcp-underscore-prefix-with-terminal-star-wildcard.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/replace-mcp-underscore-prefix-with-terminal-star-wildcard.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/replace-mcp-underscore-prefix-with-terminal-star-wildcard.
- Summary: Starting implementation after PR #304 merged and passed post-merge castor check. User confirmed terminal-star-only prefix wildcard semantics and required Hatfield agent/config guidance audit.

## Task workflow update - 2026-07-21T00:02:48.954Z
- Recorded fork run: 5ut8i3qds4se
- Summary: Implementation fork launched with terminal-star-only selector grammar, focused regression tests, docs/frontmatter updates, Hatfield agent audit, Castor validation, and commit/clean-worktree requirements.
- Scout confirmed the sole production matcher is AgentMcpToolsResolver::expandSelectors(); current old-semantics references are limited to AgentMcpToolsResolverTest, docs/agents.md, internal-docs/agents.md, and .hatfield/skills/subagents/FRONTMATTER.md. Current user Hatfield researcher/scout definitions already use exact selectors and require no wildcard migration.

## Task workflow update - 2026-07-21T00:07:15.768Z
- Recorded fork run: 5ut8i3qds4se
- Validation: Fork confirmed root AGENTS.md, testing skill, and tests/AGENTS.md were read before QA; castor test --filter=AgentMcpToolsResolverTest OK (7 tests, 18 assertions); castor deptrac OK (0 violations); castor phpstan OK (0 errors); castor cs-check OK (0 files fixed); git diff --check main...HEAD clean; Verified commit 8fe3ce733 and clean worktree
- Summary: Implementation completed and committed at 8fe3ce733d669ec61e0140da02294d961975379d. AgentMcpToolsResolver now treats exactly one terminal `*` as the only prefix wildcard; trailing underscore and all non-wildcard selectors are exact; embedded/multiple stars are non-glob exact fallthrough. Updated focused tests, public/internal agent docs, and subagent frontmatter guidance. Hatfield agent audit found no old implicit underscore-prefix selectors requiring migration. Worktree is clean.

## Task workflow update - 2026-07-21T00:30:54.448Z
- Validation: Final reviewer APPROVED after reading AGENTS.md, testing skill, and tests/AGENTS.md; castor test OK (4445 tests, 15249 assertions); castor deptrac OK (0 violations, 0 errors); castor phpstan OK (0 errors); castor cs-check OK (0 files fixed); castor test:llm-real OK (12 tests, 163 assertions); git diff --check origin/main...HEAD clean; Worktree clean at 7d03a3e1ec2e322f94e213f0d3ed6eee3f82ec6e
- Summary: Task-to-PR review complete at HEAD 7d03a3e1ec2e322f94e213f0d3ed6eee3f82ec6e. Initial reviewer APPROVED with one concrete FRONTMATTER ambiguity; docs commits bc2155f35 and 8e6cc8eb corrected precedence/combinability wording. Re-review identified an unreachable sentinel guard and inaccurate helper docblock; cleanup commit 7d03a3e1e addressed both. Final reviewer verdict: APPROVED with zero actionable task-scope findings. Worktree clean.
- Reviewed and intentionally deferred a pre-existing metadata-only inconsistency when both `mcp:*` and `mcp:-` are present: tool resolution correctly lets `mcp:-` win, while mcp_policy.mode can report `all`. It does not affect allowed tools and belongs with the next global-MCP-inheritance task, which explicitly owns deny precedence semantics.

## Task workflow update - 2026-07-21T00:34:10.034Z
- Validation: First move_task castor check blocked by llama-proxy cache growth guard (88 → 89); Warmup castor test:llm-real OK (12 tests, 163 assertions); llama-proxy cache stable during warmup (89 → 89 entries)
- Summary: First CODE-REVIEW transition was blocked only by deterministic gate cache guard: llama-proxy grew 88→89 entries during castor check. No code/test lane failure was reported. Followed runbook: reran castor test:llm-real and verified proxy cache remained stable at 89 entries before/after warmup; ready to retry the gate.

## Task workflow update - 2026-07-21T00:36:27.741Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 600s)...
- castor check passed (125.2s).
- Pushed task/replace-mcp-underscore-prefix-with-terminal-star-wildcard to origin.
- branch 'task/replace-mcp-underscore-prefix-with-terminal-star-wildcard' set up to track 'origin/task/replace-mcp-underscore-prefix-with-terminal-star-wildcard'.
- Created PR: https://github.com/ineersa/agent-core/pull/306

## Task workflow update - 2026-07-21T00:36:32.684Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/306
- Updated PR Status: open
- Validation: Deterministic castor check passed in 125.2s during CODE-REVIEW transition; Branch task/replace-mcp-underscore-prefix-with-terminal-star-wildcard pushed to origin; PR #306 created: https://github.com/ineersa/agent-core/pull/306
- Summary: Task moved to CODE-REVIEW. Deterministic castor check passed after proxy warmup, branch pushed, and PR #306 created.

## Task workflow update - 2026-07-21T02:30:28.268Z
- Moved CODE-REVIEW → DONE.
- Merged task/replace-mcp-underscore-prefix-with-terminal-star-wildcard into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/skills/subagents/FRONTMATTER.md          | 13 +--
 docs/agents.md                                     |  7 +-
 .../Agent/Execution/AgentMcpToolsResolver.php      | 21 +++--
 .../Agent/Execution/AgentMcpToolsResolverTest.php  | 95 ++++++++++++++--------
 4 files changed, 91 insertions(+), 45 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/replace-mcp-underscore-prefix-with-terminal-star-wildcard.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/replace-mcp-underscore-prefix-with-terminal-star-wildcard.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #306 state MERGED; Final reviewer APPROVED; Pre-PR castor test OK (4445 tests, 15249 assertions); Pre-PR castor deptrac OK (0 violations); Pre-PR castor phpstan OK (0 errors); Pre-PR castor cs-check OK; Pre-PR castor test:llm-real OK (12 tests, 163 assertions); CODE-REVIEW deterministic castor check passed in 125.2s
- Summary: PR #306 was merged on GitHub at 2026-07-21T02:30:02Z (merge commit e4d334243d2c1480196ee0a04a8303dfe271bfdd). Moving task to DONE, syncing integration checkout, and cleaning its worktree.

## Task workflow update - 2026-07-21T02:32:34.583Z
- Updated PR Status: merged
- Validation: LLM_MODE=true castor check OK (QA run qa-20260721-023031-666019-915b56fc); deptrac OK; test OK (4440 tests, 15192 assertions); test:controller-replay OK (9 tests, 127 assertions); test:tui OK (36 tests, 185 assertions); test:llm-real OK (12 tests, 163 assertions); phpstan OK (0 errors); cs-check OK; llama-proxy cache guard stable (90 → 90); QA artifact integrity OK; QA run leak check OK
- Summary: Post-merge integration validation completed successfully after syncing merged PR #306. Task worktree and IDEA exclusions were removed.
