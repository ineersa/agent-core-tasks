# Align agent and skill precedence by scope before Hatfield specificity

## Goal
Change automatic agent and skill discovery precedence to the following order, from lowest to highest:

1. User generic: `~/.agents`
2. Project generic: `<project>/.agents`
3. User Hatfield-specific: `~/.hatfield`
4. Project Hatfield-specific: `<project>/.hatfield`

Apply the equivalent subdirectories for each resource type: agent definitions use `agents/`, and skills use `skills/`. This changes the middle precedence so a user-level Hatfield-specific definition overrides a project-level generic `.agents` definition.

Preserve the existing precedence of explicit configuration: `agents.paths` remains above automatic agent discovery, CLI `--skills-path` remains above automatic skill discovery, and extension-registered skills remain below it.

## Acceptance criteria
- Agent name collisions resolve, lowest to highest, as `~/.agents/agents` equivalent current agent root (`~/.agents`) → project `.agents` → `~/.hatfield/agents` → project `.hatfield/agents`.
- Skill name collisions resolve, lowest to highest, as `~/.agents/skills` → project `.agents/skills` → `~/.hatfield/skills` → project `.hatfield/skills`.
- `agents.paths`, CLI `--skills-path`, and extension-registered skill precedence remain unchanged.
- Collision diagnostics still identify the correct winner and ignored/overridden source.
- Update bundled skill guidance and canonical agent/settings documentation so the stated order matches production.
- Add focused regression coverage for the changed middle-layer collision in both agent and skill discovery.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-06-align-agent-and-skill-precedence-by-scope-before-hatfield-specificity
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-06-align-agent-and-skill-precedence-by-scope-before-hatfield-specificity
Fork run: 9p9vivfl58q2
PR URL: https://github.com/ineersa/agent-core/pull/369
PR Status: merged
Started: 2026-08-06T23:08:09.706Z
Completed: 2026-08-07T02:21:58.692Z

## Work log
- Created: 2026-08-06T22:23:02.071Z

## Task workflow update - 2026-08-06T22:23:08.386Z
- Summary: Exact agent-definition roots, lowest → highest: `~/.agents` → `<project>/.agents` → `~/.hatfield/agents` → `<project>/.hatfield/agents`. Exact skill roots: `~/.agents/skills` → `<project>/.agents/skills` → `~/.hatfield/skills` → `<project>/.hatfield/skills`. The phrase `~/.agents/agents equivalent` in the first acceptance bullet is not a path requirement; `~/.agents` is the correct agent root.

## Task workflow update - 2026-08-06T23:08:09.707Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-06-align-agent-and-skill-precedence-by-scope-before-hatfield-specificity.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-06-align-agent-and-skill-precedence-by-scope-before-hatfield-specificity.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-06-align-agent-and-skill-precedence-by-scope-before-hatfield-specificity.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-06-align-agent-and-skill-precedence-by-scope-before-hatfield-specificity.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-06-align-agent-and-skill-precedence-by-scope-before-hatfield-specificity.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-06-align-agent-and-skill-precedence-by-scope-before-hatfield-specificity.
- Summary: Claimed for implementation. Exact roots follow task clarification: agent roots ~/.agents, <project>/.agents, ~/.hatfield/agents, <project>/.hatfield/agents; skill roots use skills subdirectories.

## Task workflow update - 2026-08-06T23:12:58.147Z
- Summary: Scouting located the minimal change: reorder AgentDefinitionDiscovery's four auto roots (last-write wins) and SkillDiscovery's four auto roots (first-discovered wins), preserving explicit and extension ordering. Focused regressions belong in existing AgentDefinitionDiscoveryTest and SkillDiscoveryTest; canonical guidance is in docs/agents.md, docs/settings.md, bundled subagents SKILL.md, and config defaults comments.
- Specification fidelity: no new setting/API/storage/command. Exact production contract is only the four automatic roots; agents.paths, --skills-path, extension skills, and existing collision DTO/diagnostics remain unchanged.
- Test thesis: a project-generic definition must lose to a user Hatfield-specific definition, and collision diagnostics must report that user Hatfield source as winner, for both agents and skills.

## Task workflow update - 2026-08-06T23:13:28.278Z
- Recorded fork run: oxgerrw7pfx2
- Implementation fork launched in task worktree with exact production, regression, documentation, Castor validation, and commit requirements.

## Task workflow update - 2026-08-06T23:17:47.322Z
- Recorded fork run: oxgerrw7pfx2
- Validation: Fork confirmed it read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before QA and followed Castor-only/shared-helper conventions.; `castor test --filter=AgentDefinitionDiscoveryTest` — OK, 15 tests / 38 assertions.; `castor test --filter=SkillDiscoveryTest` — OK, 22 tests / 81 assertions.; `castor phpstan --path=src/CodingAgent/Agent/Definition` — 0 errors.; `castor phpstan --path=src/CodingAgent/Skills` — 0 errors.; `castor cs-check` — clean, files_fixed=0.; Parent verification: HEAD `22efa9c56`, worktree clean, exactly 8 expected modified files. `castor check` intentionally not run during task-start.
- Summary: Implementation complete and committed as 22efa9c560b98ffb229082b440740a11d621da40. Reordered automatic agent and skill roots so user Hatfield-specific overrides project generic, retained explicit/extension precedence and collision shapes, updated focused regressions and canonical/bundled guidance. Parent verified the commit, clean worktree, expected 8-file diff (86 insertions, 42 deletions), and directional winner/loser assertions.

## Task workflow update - 2026-08-06T23:56:56.015Z
- Validation: Reviewer verdict: APPROVED; specification fidelity, discovery ordering, diagnostics, docs, and focused regressions accepted.; `castor test` — FAILED: 1/2010, `LoadedResourcesSummaryBuilderTest::testAgentCollisionWinnerAndLoserPathsPropagate`; expected old winner project `.agents`, actual intended winner user `~/.hatfield/agents`. Subsequent validation commands did not run due fail-fast chaining.
- Summary: task-to-pr reviewer approved commit 22efa9c56, but full `castor test` exposed one stale integration assertion in LoadedResourcesSummaryBuilderTest: it still expected project `.agents` to beat user `~/.hatfield/agents`. Launching a narrow fork to align that propagation proof with the finalized precedence.

## Task workflow update - 2026-08-07T00:03:52.243Z
- Recorded fork run: smum8w1onkjp
- Validation: Fix fork read testing skill and tests/AGENTS; `castor test --filter=LoadedResourcesSummaryBuilderTest` — OK, 3 tests / 11 assertions; `castor cs-check` — clean.; Re-review verdict at HEAD f87211c66: APPROVED; no blockers, no unmapped external surface, collision propagation and documentation verified.; `castor test` — OK, 4393 tests / 16443 assertions.; `castor deptrac` — 0 violations, 0 errors.; `castor phpstan` — 0 errors, 0 file errors.; `castor cs-check` — clean, files_fixed=0.; Worktree clean at f87211c663d4468357471e7e7ce5e869c236f07c.
- Summary: Task-to-PR iteration completed. Commit f87211c663d4468357471e7e7ce5e869c236f07c corrected the sole stale LoadedResourcesSummaryBuilder collision expectation found by the full suite. Re-review approved the complete branch; focused task-to-PR validation is green and worktree is clean.

## Task workflow update - 2026-08-07T00:39:58.421Z
- Recorded fork run: 9p9vivfl58q2
- Validation: Failed CODE-REVIEW deterministic gate: test:controller-replay and test:llm-real exited 1 with command.ack then command.rejected and no run.started.; Focused `castor test:controller-replay` reproduced before main update.; Local main and origin/main now both at b84e4d6ffaacd223dcee555525a7cbaf8d33ba5b (`fix skill materialization over read-only home copies and hermetic controller HOME`).
- Summary: CODE-REVIEW gate failed because controller replay/live lanes rejected start commands. User reports the test/environment fix is now on main at b84e4d6ff; launched a fork to merge current main into the task branch and rerun the previously failing deterministic controller replay plus adjacent discovery tests.

## Task workflow update - 2026-08-07T00:46:50.527Z
- Recorded fork run: 9p9vivfl58q2
- Validation: Merge fork read testing docs; clean merge with no conflicts; worktree clean at 36cfed98f.; `castor test:controller-replay` — OK, 12 tests / 165 assertions.; `castor test --filter=SkillDiscoveryTest` — OK, 22 tests / 83 assertions.; `castor test --filter=AgentDefinitionDiscoveryTest` — OK, 15 tests / 38 assertions.; Final reviewer verdict after main merge: APPROVED; merge integrity and 9-file task diff verified.; `castor test` — OK, 4393 tests / 16445 assertions.; `castor deptrac` — 0 violations/errors.; `castor phpstan` — 0 errors/file errors.; `castor cs-check` — clean, files_fixed=0.
- Summary: Merged current main b84e4d6ff into the task branch at 36cfed98fd4595cfc46dc17366c3ac1575bd42c5. Main's hermetic controller HOME/read-only materialization fix resolved the prior gate failure; task precedence changes remain intact. Final re-review approved and all focused validation is green.

## Task workflow update - 2026-08-07T00:48:59.317Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (115.8s).
- Pushed task/2026-08-06-align-agent-and-skill-precedence-by-scope-before-hatfield-specificity to origin.
- branch 'task/2026-08-06-align-agent-and-skill-precedence-by-scope-before-hatfield-specificity' set up to track 'origin/task/2026-08-06-align-agent-and-skill-precedence-by-scope-before-hatfield-specificity'.
- Created PR: https://github.com/ineersa/agent-core/pull/369
- Validation: Reviewer: APPROVED.; castor test: OK 4393 tests, 16445 assertions.; castor test:controller-replay: OK 12 tests, 165 assertions.; castor deptrac: 0 violations/errors.; castor phpstan: 0 errors.; castor cs-check: clean.
- Summary: Final reviewer APPROVED at 36cfed98f after merging current main. Controller replay and full focused Castor validation pass. Moving to deterministic gate and PR creation.

## Task workflow update - 2026-08-07T02:21:58.692Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-08-06-align-agent-and-skill-precedence-by-scope-before-hatfield-specificity into integration checkout.
- Merge made by the 'ort' strategy.
 config/hatfield.defaults.yaml                      |  4 +--
 docs/agents.md                                     | 14 ++++++----
 docs/settings.md                                   | 26 ++++++++++++------
 .../Agent/Definition/AgentDefinitionDiscovery.php  | 14 +++++-----
 .../Resources/skills/subagents/SKILL.md            | 10 +++++--
 src/CodingAgent/Skills/SkillDiscovery.php          | 12 ++++----
 .../Definition/AgentDefinitionDiscoveryTest.php    | 16 +++++------
 .../LoadedResourcesSummaryBuilderTest.php          | 13 +++++----
 tests/CodingAgent/Skills/SkillDiscoveryTest.php    | 32 ++++++++++++++++++++--
 9 files changed, 93 insertions(+), 48 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-06-align-agent-and-skill-precedence-by-scope-before-hatfield-specificity.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-06-align-agent-and-skill-precedence-by-scope-before-hatfield-specificity.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #369 state: MERGED.; Integration checkout clean before DONE transition.
- Summary: PR #369 confirmed merged on GitHub at 2026-08-07T02:21:35Z as 4f2d81a62521438c6bb3875440129411036d291c. Completing task workflow and syncing integration checkout.

## Task workflow update - 2026-08-07T02:24:20.968Z
- Validation: `LLM_MODE=true castor check` on integration checkout — quality OK.; Gate lanes: deptrac OK; test OK (4393 tests / 16445 assertions); controller replay OK (12 / 165); TUI OK (34 / 216); llm-real OK (13 / 144); phpstan OK; cs-check OK.; QA artifact integrity, exact-run leak check, cache cleanup, and llama-proxy cache guard all passed (entries 244 → 244).; Integration checkout clean at 89e8de111; task worktree confirmed removed.
- Summary: Task completed. PR #369 merged, integration checkout synchronized, task worktree and IDEA exclusions removed, and post-merge deterministic validation passed.

## Task workflow update - 2026-08-14T19:53:32+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
