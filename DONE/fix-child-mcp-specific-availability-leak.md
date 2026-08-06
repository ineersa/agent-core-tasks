# Fix child MCP availability-specific policy leak

## Goal
A shared child-tool-policy bug lets dynamically registered MCP tools from `availability: specific` servers leak into fork and ordinary subagent runs whose agent definition omits `tools`. `AgentToolPolicyResolver` starts from raw `ToolRegistry::activeToolNames()`, while `AgentMcpToolsResolver` only adds the selected MCP set and does not remove catalog MCP tools already present. Implement this as a focused generic child-policy fix on current main, not as fork/websearch-specific filtering. This task should merge before FORK-MVP-03 is merged; then update the fork branch from main and revalidate it. Keep the dormant/ignored `AgentDefinitionDTO::$mcp` design inconsistency out of scope unless separately approved.

## Acceptance criteria
- Fork and ordinary subagents with omitted `tools` receive globally available MCP tools but not `availability: specific` MCP tools.
- An agent that explicitly selects a specific MCP tool/server continues to receive it.
- The fix is generic and contains no fork, websearch, or server-name hardcoding.
- Provider-visible tool schemas and generated tool guidance use the same filtered effective set.
- Regression proof covers a catalog with one globally available MCP tool and one specific MCP tool, including explicit-selection behavior.
- Focused Castor tests, `castor deptrac`, `castor phpstan`, and `castor cs-check` pass; full `castor check` passes before CODE-REVIEW.

## Workflow metadata
Status: DONE
Branch: task/fix-child-mcp-specific-availability-leak
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-child-mcp-specific-availability-leak
Fork run: sgrzp4w95jx4
PR URL: https://github.com/ineersa/agent-core/pull/304
PR Status: merged
Started: 2026-07-20T21:04:04.698Z
Completed: 2026-07-20T23:57:06.060Z

## Work log
- Created: 2026-07-18T17:10:57.227Z

## Task workflow update - 2026-07-20T21:04:04.698Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-child-mcp-specific-availability-leak.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-child-mcp-specific-availability-leak.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-child-mcp-specific-availability-leak.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-child-mcp-specific-availability-leak.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-child-mcp-specific-availability-leak.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-child-mcp-specific-availability-leak.
- Summary: Claimed task for implementation via fork.

## Task workflow update - 2026-07-20T21:09:38.824Z
- Summary: Scout confirmed the leak is in AgentToolPolicyResolver's omitted-tools merge: activeToolNames() already contains all catalog-registered MCP tools, while AgentMcpToolsResolver selects only global MCP tools. Recommended generic boundary: have MCP resolution expose the complete catalog-backed runtime-name set, subtract that set from the inherited active registry snapshot, then add only the selected MCP runtime tools. Ordinary subagents and forks share this resolver; provider schemas and generated guidance consume the resulting allowed_tools policy.
- Implementation worktree: /home/ineersa/projects/agent-core-worktrees/fix-child-mcp-specific-availability-leak
- Primary files: src/CodingAgent/Agent/Execution/AgentToolPolicyResolver.php, src/CodingAgent/Agent/Execution/AgentMcpToolsResolver.php, tests/CodingAgent/Agent/Execution/AgentToolPolicyResolverTest.php; add the smallest launch/provider boundary regression only if needed to directly prove schema/guidance alignment.
- Scope exclusions: no fork/websearch/server-name hardcoding; do not change dormant AgentDefinitionDTO::$mcp design; do not narrow McpToolRegistrar registration or alter parent availability resolver.

## Task workflow update - 2026-07-20T21:10:24.823Z
- Recorded fork run: 78xfewtj1zqc
- Implementation fork launched in worktree with focused generic policy fix, regression tests, Castor validation, commit, and clean-worktree requirements.

## Task workflow update - 2026-07-20T21:15:44.740Z
- Recorded fork run: 78xfewtj1zqc
- Validation: castor test --filter=AgentToolPolicyResolverTest — OK (6 tests, 21 assertions); castor test --filter=AgentMcpToolsResolverTest — OK (5 tests, 13 assertions); castor deptrac — OK (0 violations); castor phpstan — OK (0 errors/file errors); castor cs-check — OK (clean); castor test:llm-real — not run; fork determined the live provider smoke does not exercise omitted-tools MCP availability policy; castor check — intentionally not run during task-start; required in task-to-pr/CODE-REVIEW gate; Parent verification: commit exists, worktree clean, expected 4-file diff (37 insertions, 9 deletions), git diff --check clean
- Summary: Implementation completed and committed at 08eb39e32f3b0c5a2474c2b6e52ce14454710078. AgentMcpToolsResolver now exposes the complete connected/configured catalog MCP runtime-name set; AgentToolPolicyResolver removes those names from omitted-tools registry inheritance and re-adds only the selected set (global by default), while explicit selectors still admit availability-specific tools. The generic resolver is shared by ordinary subagents and forks. Existing allowed_tools consumers keep generated guidance, runtime allowlists, and provider schemas aligned. Worktree verified clean; diff contains only the four expected resolver/test files.
- Fork confirmed it loaded .agents/skills/testing/SKILL.md and read tests/AGENTS.md before edits/QA, using Castor-only validation and project test conventions.
- Regression thesis: when the active registry/catalog contains an ordinary tool, one global MCP tool, and one availability-specific MCP tool, omitted-tools child policy resolves to ordinary + global only; explicit mcp: selection still resolves the specific tool.
- No TUI files or behavior changed; no TmuxHarness proof required. No PR, push, reviewer, or full gate was run in this phase.

## Task workflow update - 2026-07-20T21:25:54.221Z
- Summary: Reviewer verdict on 08eb39e32: APPROVE WITH SUGGESTIONS. No critical/bug/security blockers; implementation is correct, generic, and acceptance coverage is sufficient. Actionable cleanup accepted: clarify the catalog-collision trade-off comment, document flattenExposed deduplication invariant, and rename the misleading test helper parameter. Collision ownership tracking is deliberately deferred because it requires a broader registrar/ownership model outside this focused task; redundant extra branch/scenario assertions are skipped under the project's focused test budget.
- Reviewer confirmed provider-schema/guidance alignment is compositionally covered by the shared policy tools → launch metadata/guidance → SubagentToolSetResolver/provider schema path, with no extra integration test required.
- Reviewer confirmed no TUI/runtime/Messenger changes and no TmuxHarness/controller proof requirement.
- A narrow cleanup fork will address reasonable comment/naming suggestions, then the current HEAD will be re-reviewed.

## Task workflow update - 2026-07-20T21:26:22.377Z
- Recorded fork run: sgrzp4w95jx4
- Reviewer-suggestion cleanup fork launched: comments only in AgentMcpToolsResolver plus test-helper naming/API cleanup; no production behavior changes.

## Task workflow update - 2026-07-20T21:30:47.695Z
- Recorded fork run: sgrzp4w95jx4
- Validation: Reviewer initial verdict on 08eb39e32: APPROVE WITH SUGGESTIONS; no correctness/security blockers; Reviewer final verdict on c40047f0f: APPROVED; no actionable findings; castor test — OK (4478 tests, 15291 assertions); castor deptrac — OK (0 violations, 0 errors); castor phpstan — OK (0 errors, 0 file errors); castor cs-check — OK (0 files fixed); castor test:llm-real — OK (11 tests, 149 assertions); git diff --check origin/main...HEAD — clean; Worktree status — clean at c40047f0f13ed2f6779850931cee2a14178af9eb
- Summary: Review iteration complete. Cleanup commit c40047f0f13ed2f6779850931cee2a14178af9eb clarified the accepted catalog-collision/dedup invariants and simplified the policy test fixture helper without changing production behavior. Re-review of current HEAD returned APPROVED with no remaining actionable findings. Worktree is clean; full diff remains four expected files.
- Review cleanup commit: c40047f0f13ed2f6779850931cee2a14178af9eb (on top of functional commit 08eb39e32f3b0c5a2474c2b6e52ce14454710078).
- Deferred collision ownership tracking as a separate broader design concern: current focused policy intentionally treats catalog-advertised Hatfield names as MCP-owned for inheritance filtering, now documented in code.
- Focused validation and live LLM smoke are green; ready for deterministic CODE-REVIEW gate.

## Task workflow update - 2026-07-20T21:33:51.317Z
- Validation: First move_task castor check: FAILED only test lane (1 unrelated TUI picker unit failure); deptrac/phpstan/cs-check/controller-replay/TUI replay/llm-real lanes passed; castor test --filter=dismissFeedbackReplacesStaleExportFeedbackInPickerHeader — OK (1 test, 5 assertions); castor test retry — OK (4478 tests, 15291 assertions); Worktree remains clean at c40047f0f
- Summary: First CODE-REVIEW transition gate failed in the unit lane on unrelated `SubagentLivePickerControllerTest::dismissFeedbackReplacesStaleExportFeedbackInPickerHeader`: expected export success text but saw the fixture's no-events message. All other deterministic lanes passed. The failing test passed immediately in focused isolation, and the full `castor test` suite then passed again. No task files or production behavior were changed.

## Task workflow update - 2026-07-20T21:36:16.930Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 600s)...
- castor check passed (134.8s).
- Pushed task/fix-child-mcp-specific-availability-leak to origin.
- branch 'task/fix-child-mcp-specific-availability-leak' set up to track 'origin/task/fix-child-mcp-specific-availability-leak'.
- Created PR: https://github.com/ineersa/agent-core/pull/304
- Validation: Reviewer: APPROVED; castor test — OK (4478 tests, 15291 assertions); castor deptrac — OK; castor phpstan — OK; castor cs-check — OK; castor test:llm-real — OK; Previously failing unrelated picker test — OK focused
- Summary: Reviewer-approved HEAD c40047f0f. Retrying deterministic gate after a single unrelated transient unit-test failure passed in isolation and the full unit suite passed again.

## Task workflow update - 2026-07-20T21:36:22.522Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/304
- Updated PR Status: open
- Validation: Deterministic castor check — PASSED (134.8s); PR: https://github.com/ineersa/agent-core/pull/304
- Summary: Task moved to CODE-REVIEW after reviewer approval and successful deterministic gate. Branch pushed and PR #304 created.
- Current reviewed HEAD: c40047f0f13ed2f6779850931cee2a14178af9eb
- PR opened: https://github.com/ineersa/agent-core/pull/304

## Task workflow update - 2026-07-20T23:57:06.060Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-child-mcp-specific-availability-leak into integration checkout.
- Merge made by the 'ort' strategy.
 .../Agent/Execution/AgentMcpToolsResolver.php      | 11 +++++++
 .../Agent/Execution/AgentToolPolicyResolver.php    | 10 +++++-
 .../Agent/Execution/AgentMcpToolsResolverTest.php  |  1 +
 .../Execution/AgentToolPolicyResolverTest.php      | 36 ++++++++++++++--------
 4 files changed, 44 insertions(+), 14 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-child-mcp-specific-availability-leak.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-child-mcp-specific-availability-leak.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Pre-PR focused validation: castor test OK (4478 tests, 15291 assertions); castor deptrac OK (0 violations); castor phpstan OK (0 errors); castor cs-check OK; castor test:llm-real OK (11 tests, 149 assertions); Reviewer re-review APPROVED with zero actionable findings; GitHub PR #304 state MERGED
- Summary: PR #304 was merged on GitHub at 2026-07-20T23:56:19Z (merge commit 132824dabb10c0b3016dac1be82fbbdbfd35ea5f). Moving task to DONE and syncing integration checkout.

## Task workflow update - 2026-07-20T23:59:16.373Z
- Validation: LLM_MODE=true castor check OK (QA run qa-20260720-235710-493111-df2ec864); deptrac OK; test OK (4438 tests, 15187 assertions); test:controller-replay OK (8 tests, 103 assertions); test:tui OK (36 tests, 185 assertions); test:llm-real OK (12 tests, 163 assertions); phpstan OK (0 errors); cs-check OK; llama-proxy cache guard stable (88 → 88); QA artifact integrity OK; QA run leak check OK
- Summary: Post-merge integration validation completed successfully on main after syncing merged PR #304.
