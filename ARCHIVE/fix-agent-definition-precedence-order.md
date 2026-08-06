# Fix agent definition precedence order to prefer .hatfield over .agents

## Goal
Currently AgentDefinitionDiscovery::discover() loads directories in this order:
1. ~/.hatfield/agents/ (loaded first, lowest precedence)
2. ~/.agents/ (loaded second, overrides .hatfield)
3. .hatfield/agents/ (project)
4. .agents/ (project, overrides project .hatfield)
5. agents.paths from settings (highest)

But it should be (reversed at each scope):
1. ~/.agents/ (lowest)
2. ~/.hatfield/agents/
3. .agents/ (project)
4. .hatfield/agents/ (project)
5. agents.paths (highest)

## Acceptance criteria
- Swap load order in `src/CodingAgent/Agent/Definition/AgentDefinitionDiscovery.php` so that at each scope level (user then project), `.hatfield/agents/` is loaded AFTER `.agents/` (last-write wins)
- Update the docblock precedence comment to match the correct order
- Update `testUserAgentsOverridesUserHatfield()` -> `testUserHatfieldOverridesUserAgents()` with swapped assertions
- Update `testProjectAgentsOverridesProjectHatfield()` -> `testProjectHatfieldOverridesProjectAgents()` with swapped assertions
- Update `docs/agents.md` if it documents the wrong order
- Update the test `testOverrideProducesCollisionDiagnostic()` if it references specific winner/loser ordering

## Workflow metadata
Status: ARCHIVE
Branch: task/fix-agent-definition-precedence-order
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-agent-definition-precedence-order
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/302
PR Status: merged
Started: 2026-07-20T18:58:10.285Z
Completed: 2026-07-20T20:21:15.794Z

## Work log
- Created: 2026-07-18T01:16:53+00:00

## Task workflow update - 2026-07-20T18:58:10.285Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-agent-definition-precedence-order.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-agent-definition-precedence-order.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-agent-definition-precedence-order.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-agent-definition-precedence-order.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-agent-definition-precedence-order.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-agent-definition-precedence-order.
- Summary: Implementation approved. Scope includes correcting agent definition precedence, synchronizing documentation (including the linked implementation plan), and removing the project .hatfield webresearch agent definition if present.

## Task workflow update - 2026-07-20T19:29:41.993Z
- Validation: castor test --filter=AgentDefinitionDiscoveryTest — OK (15 tests, 40 assertions); castor phpstan --path src/CodingAgent/Agent/Definition/AgentDefinitionDiscovery.php — OK (0 errors); castor cs-check — OK (clean)
- Summary: Implementation committed as abd729e9b1f949ed3d072094be026a080dfd4ab5. Corrected same-scope discovery so ~/.hatfield/agents overrides ~/.agents and project .hatfield/agents overrides .agents; synchronized docs/default config/subagent skill/linked plan; renamed and reversed focused precedence tests; deleted .hatfield/agents/researcher-websearch.md while leaving .hatfield/mcp.json unchanged. Worktree is clean. Fork confirmed it read .agents/skills/testing/SKILL.md and tests/AGENTS.md.

## Task workflow update - 2026-07-20T19:40:25.064Z
- Validation: castor test — OK (4478 tests, 15289 assertions; 28.7s); castor deptrac — OK (0 violations, 0 errors); castor phpstan — OK (0 errors); castor cs-check — OK (0 files fixed / clean); Reviewer re-review of HEAD e9c38b5b7 — APPROVED
- Summary: Code review complete at e9c38b5b74a651f63806f4d7fe70ae9a521619b8. Reviewer verdict: APPROVED with no actionable findings. Full diff satisfies precedence acceptance criteria, synchronized documentation, and approved removal of researcher-websearch while preserving .hatfield/mcp.json. Worktree is clean.
- Implementation commit abd729e9b: corrected same-scope discovery precedence, synchronized docs/config/skill/plan, and removed .hatfield/agents/researcher-websearch.md.
- Review follow-up commit e9c38b5b7: strengthened collision diagnostic test with exact winnerPath/loserPath assertions.
- Reviewer initially returned APPROVE WITH SUGGESTIONS; reasonable test-hardening suggestion was implemented by fork, then current HEAD was re-reviewed and APPROVED.

## Task workflow update - 2026-07-20T19:42:35.416Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (117.0s).
- Pushed task/fix-agent-definition-precedence-order to origin.
- branch 'task/fix-agent-definition-precedence-order' set up to track 'origin/task/fix-agent-definition-precedence-order'.
- Created PR: https://github.com/ineersa/agent-core/pull/302
- Validation: castor test — OK (4478 tests, 15289 assertions); castor deptrac — OK (0 violations, 0 errors); castor phpstan — OK (0 errors); castor cs-check — OK (clean)
- Summary: Reviewer approved HEAD e9c38b5b74a651f63806f4d7fe70ae9a521619b8 after one review-fix iteration. Focused Castor validation passed: test, deptrac, phpstan, cs-check.

## Task workflow update - 2026-07-20T19:42:40.363Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/302
- Updated PR Status: open
- Validation: Deterministic castor check during CODE-REVIEW transition — OK (117.0s)
- Pushed task/fix-agent-definition-precedence-order and opened PR #302: https://github.com/ineersa/agent-core/pull/302

## Task workflow update - 2026-07-20T20:21:15.794Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-agent-definition-precedence-order into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/agents/researcher-websearch.md           | 11 ---------
 .hatfield/skills/subagents/SKILL.md                |  6 ++---
 .pi/plans/agents-subagents-implementation-plan.md  |  6 ++---
 config/hatfield.defaults.yaml                      | 12 +++++-----
 docs/agents.md                                     | 12 +++++-----
 .../Agent/Definition/AgentDefinitionDiscovery.php  | 26 +++++++++++-----------
 .../Definition/AgentDefinitionDiscoveryTest.php    | 14 ++++++------
 7 files changed, 38 insertions(+), 49 deletions(-)
 delete mode 100644 .hatfield/agents/researcher-websearch.md
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-agent-definition-precedence-order.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-agent-definition-precedence-order.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #302 confirmed merged on GitHub at 2026-07-20T20:20:44Z (merge commit 1949d42f5240ce8d5379c2dcf1e301a880cebcbf). Reviewer approved and pre-merge deterministic Castor gate passed.

## Task workflow update - 2026-07-20T20:23:36.985Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/302
- Updated PR Status: merged
- Validation: LLM_MODE=true castor check — OK (297.8s); deptrac — OK (1.0s); test — OK (4473 tests, 15232 assertions; 45.6s); test:controller-replay — OK (8 tests, 112 assertions; 77.6s); test:tui — OK (37 tests, 193 assertions; 113.4s); test:llm-real — OK (11 tests, 149 assertions; 54.9s); phpstan — OK (0 errors; 2.1s); cs-check — OK (3.2s); llama-proxy cache guard stable (78 → 78); artifact integrity and QA leak checks passed
- Summary: Task completed after PR #302 merge. Integration checkout merged/pulled successfully, task worktree and IDEA exclusions were removed, and post-merge deterministic validation passed. Integration working tree is clean; local main is ahead of origin/main by two workflow-created merge commits.
- PR #302 merged with GitHub merge commit 1949d42f5240ce8d5379c2dcf1e301a880cebcbf.
- DONE transition removed /home/ineersa/projects/agent-core-worktrees/fix-agent-definition-precedence-order and its IDEA exclusions.
- Post-merge integration HEAD: 96ba49fb2; git working tree clean; main ahead of origin/main by two local workflow merge commits.

## Task workflow update - 2026-08-06T20:58:59.986Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
