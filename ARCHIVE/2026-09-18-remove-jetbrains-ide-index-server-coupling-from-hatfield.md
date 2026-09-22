# Remove JetBrains IDE index server coupling from Hatfield

## Goal
Remove JetBrains IDE index MCP registration, appended system instructions, agent guidance, and task-workflow IDE lifecycle calls. Keep jbcontext/code_search and existing .idea setup/exclusions. Update bundled and installed Hatfield agent definitions and installed workflow skill. Legacy .pi integration is out of scope.

## Acceptance criteria
- Hatfield task worktree creation, merge, and cancellation no longer contact JetBrains MCP.
- Remove dead JetBrains client and result/note plumbing.
- Hatfield MCP config, system append, root instructions, workflow skills, and agent templates no longer require IDE tools.
- Preserve jbcontext/code_search and existing .idea support; run focused Castor validation.

## Workflow metadata
Status: DONE
Branch: task/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/508
PR Status: merged
Started: 2026-09-18T13:55:24+00:00
Completed: 2026-09-18T15:53:56+00:00

## Work log
- Created: 2026-09-18T13:54:49+00:00

## Task workflow update - 2026-09-18T13:55:24+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield.

## Task workflow update - 2026-09-18T13:56:07+00:00
- Summary: Two scouts mapped workflow and instruction/config consumers. Main owns this cohesive removal. Preserve jbcontext and .idea filesystem setup/exclusions; legacy .pi is out of scope.
- Ownership: owner=main; fork_run=none; revision=414b77005; scope=Hatfield JetBrains MCP lifecycle/config/instruction removal and installed Hatfield agent guidance; outcome=assigned; commit=none

## Task workflow update - 2026-09-18T14:05:02+00:00
- Validation: castor test --filter='MoveTaskHandlerTest|WorktreeManagerCopyTest|AgentsInitCommandTest': PASS, 26 tests, 285 assertions, max case 0.173124s.; castor test --filter='TaskWorkflowSkillInstallerTest|JbcontextAssetInstallerTest': PASS, 11 tests, 49 assertions.; castor phpstan --path=.hatfield/extensions/task-workflow/src: PASS, zero errors. Initial positional-path command rejected; corrected to --path.; castor cs-check: PASS, zero fixes.; castor docs:validate: PASS, 21 documents.; git diff --check: PASS. No tracked JetBrains index MCP/client/open-close identifiers outside legacy .pi. Worktree clean after commit.; Full castor check not run in task-start; reserved for CODE-REVIEW transition.
- Summary: Implemented and committed 5fd98ca97 in task worktree. Deleted JetBrains MCP client and appended system prompt; removed workflow lifecycle calls/result plumbing, MCP entry, and IDE guidance from root/project/bundled agents and docs. Preserved jbcontext, .idea setup/exclusions, generic MCP, Datadog, and legacy .pi. Bumped task-workflow skill to 1.1.1 for automatic installed-copy refresh. Locally updated only guidance in ~/.hatfield/agents/{scout,architect,reviewer}.md preserving model/settings, plus ignored main-checkout installed task-start reference. Main checkout tracked files unchanged. Independent review and full gate remain for task-to-pr.
- Ownership: owner=main; fork_run=none; revision=414b77005; scope=Hatfield JetBrains MCP lifecycle/config/instruction removal and installed Hatfield agent guidance; outcome=completed; commit=5fd98ca97

## Task workflow update - 2026-09-18T14:14:15+00:00
- Validation: Prior 37 focused tests / 334 assertions, scoped PHPStan, cs-check and docs:validate retained for unchanged revision.; Current worktree clean at 5fd98ca97; installed home guidance and task-start reference checked on disk.
- Summary: task-to-pr independent specification-fidelity review APPROVE, no blocking findings. Reusing focused validation for unchanged clean revision 5fd98ca97. User explicitly confirmed keeping .idea setup/exclusions. Verified home APPEND_SYSTEM, installed Hatfield agents, and installed task-start reference have no IDE guidance; reviewer final home-prompt reminder was based on inherited stale prompt, not current disk. Full QA remains owned by CODE-REVIEW transition.
- Review: role=reviewer; artifact=agent_48a641e12c315a4b; revision=5fd98ca97c9057620ec69764d85e9b0085d909c0; scope=all 15 changed files, specification fidelity, removal completeness, fail-closed lifecycle safety, jbcontext/.idea preservation, versioned skill refresh; verdict=APPROVE; blockers=none

## Task workflow update - 2026-09-18T14:16:42+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (134.5s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield/var/reports/qa-20260918-141428-4369-44968513.
- Session/run: 58.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-18T14:16:44+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield/var/reports/qa-20260918-141428-4369-44968513.
- Session/run: 58.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-18T14:16:47+00:00
- castor check passed (134.5s).
- Pushed task/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield to origin.
- Created PR: <url>
- Session/run: 58.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-18T14:16:47+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (134.5s).
- Pushed task/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/508
- Summary: Independent reviewer approved revision 5fd98ca97 with no blockers. Preserve jbcontext and .idea setup/exclusions as confirmed by user.

## Task workflow update - 2026-09-18T15:53:56+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield: ide_close_project returned isError.
- Merged task/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/APPEND_SYSTEM.md                                                       |   7 -------
 .hatfield/agents/scout.md                                                        |   6 ++----
 .hatfield/extensions/task-workflow/skills/task-workflow/SKILL.md                 |   2 +-
 .hatfield/extensions/task-workflow/skills/task-workflow/references/task-start.md |   2 +-
 .hatfield/extensions/task-workflow/src/Ide/JetBrainsMcpClient.php                | 125 -----------------------------------------------------------------------------------------------------------------------------
 .hatfield/extensions/task-workflow/src/Tool/MoveTaskHandler.php                  |   3 ---
 .hatfield/extensions/task-workflow/src/Worktree/WorktreeCreateResult.php         |   1 -
 .hatfield/extensions/task-workflow/src/Worktree/WorktreeManager.php              |  54 +++---------------------------------------------------
 .hatfield/mcp.json                                                               |   4 ----
 AGENTS.md                                                                        |   6 +++---
 architecture/tools-and-mcp.md                                                    |   2 +-
 docs/tools.md                                                                    |   1 -
 src/CodingAgent/Resources/agents/architect.md                                    |   4 ++--
 src/CodingAgent/Resources/agents/reviewer.md                                     |   4 ++--
 src/CodingAgent/Resources/agents/scout.md                                        |   8 +++-----
 15 files changed, 18 insertions(+), 211 deletions(-)
 delete mode 100644 .hatfield/APPEND_SYSTEM.md
 delete mode 100644 .hatfield/extensions/task-workflow/src/Ide/JetBrainsMcpClient.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-18-remove-jetbrains-ide-index-server-coupling-from-hatfield.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub PR #508 confirmed MERGED at 2026-09-18T15:52:44Z, merge commit 9a69b7514431a75f850b93e65d83cf0b86865325. Installed ~/.hatfield agents were already updated preserving personal model settings; rechecked no remaining IDE guidance. Post-merge castor check follows.

## Task workflow update - 2026-09-18T15:55:39+00:00
- Updated PR Status: merged
- Validation: castor check: PASS, 143.1s, all 10 lanes green; unit 5108 tests, controller replay 11, TUI 9, live LLM 5.; QA reports: /home/ineersa/projects/agent-core/var/reports/qa-20260918-155405-10006-a9ccfc4d; Leak check passed; proxy cache unchanged 402→402; git status clean; task worktree absent.
- Summary: Post-merge validation complete at integrated revision 2958a742d169e21033853fd485f325f7051b556b. Integration checkout clean and task worktree removed. Installed workflow skill refreshed to 1.1.1. Home Hatfield agents and APPEND_SYSTEM rechecked without JetBrains tool guidance; personal model settings preserved. DONE transition used already-loaded old extension code and emitted one non-fatal IDE-close error before merging the removal; new source no longer contains that client.
