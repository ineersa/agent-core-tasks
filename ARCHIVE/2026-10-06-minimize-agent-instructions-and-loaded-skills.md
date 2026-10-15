# Minimize AGENTS.md and loaded skills; remove redundant semantic-search skill

## Goal
## Goal

Reduce instruction context, not just line counts. Keep direct, actionable rules. Delete duplicated tool guidance, architecture catalogs, document maps, examples, background, and obsolete instructions. Move only necessary detail into task-specific skill references. Do not archive discarded text wholesale.

User limits: root AGENTS.md <=80 lines; every in-scope SKILL.md <=50 lines; retained nested AGENTS.md <20 lines. Count frontmatter and blank lines. Do not meet limits with long packed lines. Shorten skill descriptions too, because discovery puts them in the available-skills catalog.

## Audit findings

Baseline at main d7f246bc0, clean checkout:

| Instruction | Lines | Words |
|---|---:|---:|
| AGENTS.md | 193 | 2569 |
| .agents/skills/testing/SKILL.md | 446 | 4017 |
| tests/AGENTS.md | 202 | 2111 |
| src/AgentCore/Application/AGENTS.md | 142 | 2219 |
| src/CodingAgent/Runtime/Protocol/AGENTS.md | 46 | 710 |
| src/AgentCore/Domain/Message/AGENTS.md | 46 | 252 |
| src/CodingAgent/Runtime/Process/AGENTS.md | 29 | 145 |
| src/AgentCore/Domain/Event/AGENTS.md | 26 | 225 |
| src/CodingAgent/Runtime/AGENTS.md | 18 | 221 |
| src/Tui/AGENTS.md | 13 | 173 |
| .hatfield/extensions/extension-api/AGENTS.md | 8 | 165 |
| src/AgentCore/Domain/AGENTS.md | 8 | 47 |
| src/AgentCore/Infrastructure/Doctrine/AGENTS.md | 10 | 88 |
| .hatfield/extensions/task-workflow/skills/task-workflow/SKILL.md | 52 | 329 |
| .pi/skills/task-workflow/SKILL.md | 40 | not measured |
| src/CodingAgent/Resources/skills/subagents/SKILL.md | 97 | 896 |
| .agents/skills/castor/SKILL.md | 36 | 271 |
| .hatfield/skills/datadog/SKILL.md | 33 | 390 |
| ~/.agents/skills/unslop/SKILL.md | 82 | 1010 |
| ~/.agents/skills/technical-writing/SKILL.md | 129 | 1908 |
| ~/.agents/skills/playwright-cli/SKILL.md | 280 | 781 |
| ~/.hatfield/skills/subagents/SKILL.md | 97 | 896 |

These are file inventories, not a claim that every body is injected at startup. SkillsContextBuilder::build() exposes a catalog and configured preloads; buildFor() loads agent-frontmatter skills. AgentsContextDiscovery walks globals and cwd ancestors, not all descendants. Root-started sessions load root AGENTS.md; nested files are read when entering an area. The mandatory root + testing skill + tests instructions currently total 841 lines and 8697 words before test work.

The Domain root file is a map. Doctrine instructions are mostly location/config facts. Application, Event, Message, Process, and Protocol instructions contain large source-derived catalogs and implementation histories. Root repeats testing/workflow details and has layout, framework setup, and docs maps. tests/AGENTS.md duplicates much of the testing skill. Subagent skill repeats model-visible tool contracts. Browser skill is mostly command examples.

## Scope and implementation

Main owns all writes and edits, as requested. Scouts may investigate bounded unknowns read-only. No implementation fork planned. Independent review remains required.

1. Rewrite root as project-specific commands and constraints only. Delete layout and docs maps, framework facts, duplicated tool guidance, and repeated module/testing/workflow explanations. Keep necessary project rule authority and short conditional skill-loading instructions. Do not replace the removed docs map with another catalog.
2. Delete map-only nested AGENTS.md, including Domain and Doctrine unless fresh inspection finds an instruction that needs retention. Delete other nested files with no unique rule. For retained files, keep only non-obvious local instructions in <=19 lines. Update references to deleted files. Do not re-create architecture catalogs as skill references when code, configuration, or existing docs already supply the facts.
3. Make testing/SKILL.md a <=50-line entry point with essential standards and narrow reference-loading triggers. Put retained test conventions, helper usage, command details, process/isolation guidance, and diagnostics in focused references. Keep tests/AGENTS.md below 20 lines or delete it and update prerequisite instructions. Avoid requiring every reference for ordinary test work. Remove duplicate ownership across root, tests, testing, and workflow references.
4. Minimize all listed project and bundled skill entry points, including already-short files. Keep the active Hatfield workflow source, installed copy, and Pi workflow consistent where rules are shared. Preserve runtime-specific differences. Minimize bundled and installed subagents together. References must load only for the relevant operation, not unconditionally.
5. Apply the same <=50-line limit to the listed user-level unslop, technical-writing, playwright-cli, and subagents entry points. User-level files are outside the code repository; record their changes and validation separately rather than pretending a Git PR contains them. Inspect their current state before editing. Keep any necessary examples or detailed conventions in conditional references. Do not touch unrelated global skills or third-party/vendor/tmp assets.
6. Remove jbcontext-semantic-search entirely while retaining code_search. Keep search instructions in the existing tool description and promptGuidelines, not in AGENTS.md or another skill. Retain only any genuinely missing instruction there; examples and guessed restart/index remedies are not needed.

## Semantic-search removal details

This skill is not unused: .hatfield/agents/scout.md preloads it. It is redundant with the code_search description and guidelines in .hatfield/extensions/jbcontext/src/JbcontextExtension.php. Its tracked 35-line source and ignored installed copy are separate files.

Inspect these entry points and callers:
- .hatfield/extensions/jbcontext/resources/skills/jbcontext-semantic-search/SKILL.md
- .hatfield/extensions/jbcontext/src/Assets/JbcontextAssetInstaller.php
- .hatfield/extensions/jbcontext/src/Assets/JbcontextMarkdownFrontmatter.php
- .hatfield/extensions/jbcontext/src/State/JbcontextPaths.php
- .hatfield/extensions/jbcontext/src/Job/JbcontextEligibilityJobHandler.php
- .hatfield/agents/scout.md
- .hatfield/extensions/jbcontext/README.md and .gitignore

Remove the bundled skill, skill installation/version-only helpers and fields, scout skills injection, and committed scout preload reference. Preserve unrelated user scout skills, code_search availability, eligibility/indexing, and existing scout copy behavior. Remove redundant appended scout guidance where tool guidance already covers it. Update installer/eligibility tests and docs. Remove the known ignored installed .hatfield/skills/jbcontext-semantic-search copy in this checkout so discovery stops advertising it. Do not add a permanent compatibility migration or change generic skill discovery.

Relevant existing tests:
- .hatfield/extensions/jbcontext/tests/JbcontextAssetInstallerTest.php
- .hatfield/extensions/jbcontext/tests/JbcontextEligibilityJobHandlerTest.php
- .hatfield/extensions/jbcontext/tests/Controller/ControllerReplayJbcontextStartupEligibilityTest.php

## Source and installation risks

.hatfield/skills/task-workflow is an ignored installed copy of .hatfield/extensions/task-workflow/skills/task-workflow. TaskWorkflowSkillInstaller refreshes the whole directory only when the version changes; bump the bundled version as needed and refresh the installed copy through the existing facility. Editing only the installed copy does not ship the change.

~/.hatfield/skills/subagents/SKILL.md currently matches src/CodingAgent/Resources/skills/subagents/SKILL.md. Inspect the existing built-in installer before updating both. Do not add a new installer or context-loading mechanism.

Preserve enforceable safety and product constraints, not every explanatory sentence. Essential rules must remain discoverable with a clear trigger: Castor-only QA; task gate/transition ownership; deterministic tests and <=10s case ceiling; resource teardown/isolation; no root-owned or HATFIELD_SESSION_ID worker signaling; authoritative Deptrac boundaries; published ExtensionApi independence; no HTTP-serving stack; minimal specification-faithful changes; no unsupported fallback paths; privacy-safe runtime logs. Detailed procedures belong to their existing skill references. Removing duplicate rules must not remove their only authoritative occurrence.

Do not expand this task into changing discovery behavior, runtime protocols, test implementations unrelated to skill installation, agent models, or general product documentation.

## Validation

Load the existing testing skill and tests/AGENTS.md before test work, and the exact workflow phase before transitions. Use targeted Castor checks during implementation. Run installer/eligibility tests using existing Castor facilities, including the disabled-startup regression where relevant. Run castor docs:validate. Verify frontmatter, all changed reference links, source/installed equality where applicable, and no dangling semantic-search preload. Verify code_search remains registered with its description/guidelines and installer no longer creates the removed skill. Do not run a live LLM just to judge prose.

Report before/after lines, words, and bytes for scoped entry points and representative loaded sets: root startup plus catalog; ordinary testing; workflow phase; scout preload; writing. State the exact loaded files for each set. References count toward savings whenever their triggers require loading them. No new product setting, test-only production API, or permanent budget tool is needed. The CODE-REVIEW transition owns the full castor check gate.

## Acceptance criteria
- Root AGENTS.md is <=80 total lines and contains only direct project instructions, with no layout or docs map.
- Every retained in-scope SKILL.md is <=50 total lines, including frontmatter; every retained nested AGENTS.md is <=19 total lines.
- Duplicate, obsolete, source-derived, and explanatory text is deleted. Only necessary details remain in conditionally loaded skill references, without unconditional load-all routing or packed-line limit evasion.
- All scoped entry points and representative loaded sets show measured before/after context reduction in words and bytes; file inventories are not mislabeled as startup context.
- jbcontext-semantic-search is absent from bundled/installed skills, installer logic, scout preloads, and stale documentation. code_search remains available with sufficient tool-owned guidance and unchanged search behavior.
- Bundled sources and installed copies are updated consistently. Home-directory skill edits are recorded separately from repository changes.
- Retained safety, architecture, testing, and workflow requirements have one clear authority and loading trigger. No broken references or new speculative behavior.
- Main performs implementation; independent review and the workflow-owned Castor gate pass.

## Workflow metadata
Status: DONE
Branch: task/2026-10-06-minimize-agent-instructions-and-loaded-skills
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-10-06-minimize-agent-instructions-and-loaded-skills
Fork run: 373b9da0-baa3-5c10-88fe-614fefed3491
PR URL: https://github.com/ineersa/agent-core/pull/556
PR Status: merged
Started: 2026-10-07T00:03:01+00:00
Completed: 2026-10-07T12:52:44+00:00

## Work log
- Created: 2026-10-06T23:40:11+00:00

## Task workflow update - 2026-10-07T00:03:01+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-10-06-minimize-agent-instructions-and-loaded-skills.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-10-06-minimize-agent-instructions-and-loaded-skills.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-10-06-minimize-agent-instructions-and-loaded-skills.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-10-06-minimize-agent-instructions-and-loaded-skills.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-10-06-minimize-agent-instructions-and-loaded-skills.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-10-06-minimize-agent-instructions-and-loaded-skills/.idea.

## Task workflow update - 2026-10-07T00:03:18+00:00
- Ownership: owner=main; fork_run=none; revision=d7f246bc0; scope=project and user instruction minimization plus semantic-search skill removal; outcome=assigned; commit=none

## Task workflow update - 2026-10-07T00:31:21+00:00
- Validation: Castor instruction audit passed: all scoped line budgets, frontmatter, local reference links, and bundled/installed equality. Reproduce from the task worktree with castor instructions:audit --castor-file=var/reports/instruction-context/audit.php. The audit is temporary, ignored, and not product code.; Measured word reduction: root plus rendered skill catalog 78.0%; focused testing 89.7%; test authoring including required helpers 87.8%; scout context 78.7%; document writing including mode reference 81.6%; task-start including required phase references 64.3%. Full words/bytes/read-set evidence is in /home/ineersa/projects/agent-core/var/reports/instruction-context-2026-10-06/measurement.json.; castor test --filter='Jbcontext|SkillDiscoveryTest|SkillRegistryTest|SkillContextRendererTest|SkillsContextBuilderTest|TaskWorkflowSkillInstallerTest': 106 tests, 428 assertions passed. Max case 0.061272s.; castor test:controller-replay: 14 tests, 260 assertions passed. Max case 5.674433s. Temporary supervision loss was resolved from exit-status file 0 and completed JUnit/log evidence; the command was not rerun.; castor test --filter=PharSmokeTest: 7 tests, 8775 assertions passed. Max case 1.417294s. Rebuilt PHAR contains the relocated subagents reference.; Final castor test --filter=TaskWorkflowSkillInstallerTest: 5 tests, 26 assertions passed after replacing remaining heading assertions with copied-file identity checks.; castor phpstan, scoped jbcontext phpstan, castor dead-code, castor deptrac, castor cs-check, castor docs:validate, and git diff --check passed. A scoped phpstan invocation on workflow/tests found no analyzable files; full castor phpstan passed instead.; castor clean:cleanup:workers:list reported no stale candidates. No root-owned or active-session workers were signaled.; Task worktree is clean after commit. No castor check, reviewer, push, PR, or merge was performed in task-start.
- Summary: Implemented and committed as d691db994 in the task worktree. Root AGENTS.md is 45 lines; retained nested files are 5–11 lines, including tests at 7; scoped skill entry points are 12–34 lines. Removed map-only Domain/Doctrine instructions, duplicate prose, and semantic-search skill installation/preloads. Preserved code_search registration and behavior. Necessary detail is in conditional references. Updated Hatfield/Pi workflow sources, bumped bundled workflow version to 1.1.2, and refreshed the task-worktree installed copy through the existing installer. Updated four home-directory skill entries separately and moved/shortened the subagents reference. Removed the ignored semantic-search skill copy from the primary checkout. Repository changes are on the task branch, not merged; main's old installers can recreate old built-in assets until integration. Full gate and independent review remain for task-to-pr.
- Ownership: owner=main; fork_run=none; revision=d7f246bc0; scope=project and user instruction minimization plus semantic-search skill removal; outcome=completed; commit=d691db994
- Home-directory changes are outside Git. Persistent before/after snapshots, home-changes.json, measurements, and case durations are under /home/ineersa/projects/agent-core/var/reports/instruction-context-2026-10-06/. The built-in subagents file/reference permissions were restored to read-only.
- Coverage deletion evidence: removed installsBundledSkillWhenAbsent, reinstallsSkillWhenInstalledVersionMissingOrDifferent, and leavesSameVersionSkillUntouched because the user explicitly retired skill installation/versioning. No remaining installation behavior is claimed. copiesUserScoutWithCodeSearchWithoutChangingSkillsOrBody protects retained scout/tool/model/skills/body preservation and proves no skills directory is created. eligibleRunsSilentIndexAndInstallsAssets protects eligibility plus scout installation without skill assets. testControllerStartupDisablesEligibilityWithoutIdeaBeforeAnyTurn and testControllerRestartRecoversStaleDisabledToEligibleAndCodeSearch retain real controller eligibility/search proofs.
- Next phase is task-to-pr. It must load its procedure, obtain independent review, and use the CODE-REVIEW transition for the full castor check gate.

## Task workflow update - 2026-10-07T02:03:44+00:00
- Summary: Preparing PR for d691db994. Independent reviewer explicitly waived by the user: "no need for reviewer part, go straight to PR". Worktree is clean. Reusing focused validation for this revision; required transition gate remains enabled.
- Review exception: user waived reviewer for revision d691db994. No reviewer subagent launched; no approval verdict claimed.

## Task workflow update - 2026-10-07T02:05:18+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (67.5s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-06-minimize-agent-instructions-and-loaded-skills/var/reports/qa-20261007-020411-4514-bddb0190.
- Session/run: 80.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-07T02:05:20+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-10-06-minimize-agent-instructions-and-loaded-skills to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-06-minimize-agent-instructions-and-loaded-skills/var/reports/qa-20261007-020411-4514-bddb0190.
- Session/run: 80.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-07T02:05:23+00:00
- castor check passed (67.5s).
- Pushed task/2026-10-06-minimize-agent-instructions-and-loaded-skills to origin.
- Created PR: <url>
- Session/run: 80.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-07T02:05:23+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (67.5s).
- Pushed task/2026-10-06-minimize-agent-instructions-and-loaded-skills to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/556
- Validation: castor test:llm-real: 5 tests, 30 assertions passed before the transition.
- Summary: Submit d691db994. Reviewer skipped at explicit user request. Reused targeted validation and warmed the live-test proxy cache before the required transition gate.

## Task workflow update - 2026-10-07T02:50:27+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User approved implementing all discussed PR feedback plus the exact compact generic fork operating contract and conditional handoff format. Scope: restore scout reminder and original development rules; simplify implementation ownership and clarify specialist roles; preserve testing standards in skill/references; replace fork prompt strings, expose parent task convention, update affected tests. Existing reviewer waiver remains in effect for this PR.

## Task workflow update - 2026-10-07T02:50:52+00:00
- Summary: One implementation fork owns the entire follow-up to avoid shared-index and generated-skill collisions. Main owns coordination, diff inspection, task transitions, and PR submission. No reviewer requested under the user's existing waiver.
- Ownership: owner=fork; fork_run=none; revision=d691db994; scope=all approved instruction/ownership/testing/scout feedback and compact fork prompt plus parent guidelines and affected tests in existing task worktree; outcome=assigned; commit=none

## Task workflow update - 2026-10-07T02:57:44+00:00
- Recorded fork run: 373b9da0-baa3-5c10-88fe-614fefed3491
- Summary: Parent diff inspection confirmed exact approved fork strings and updated tests. Found unrequested root-rule omissions while restoring Development rules (dead-code/fallback prohibition and explicit approved architecture boundaries), plus duplicate new reference text and an unnecessary Pi resume addition. Returning these bounded corrections to the same implementation fork before submission.
- Ownership: owner=fork; fork_run=373b9da0-baa3-5c10-88fe-614fefed3491; revision=d691db994; scope=approved PR feedback and compact fork contract; outcome=completed; commit=bcbaae2c194f45a95359376f7b17b5f553b26e98
- Ownership: owner=fork; fork_run=373b9da0-baa3-5c10-88fe-614fefed3491; revision=bcbaae2c194f45a95359376f7b17b5f553b26e98; scope=preserve displaced root safety/architecture rules and remove redundant testing additions and unrelated Pi resume instructions; outcome=assigned; commit=none

## Task workflow update - 2026-10-07T03:00:19+00:00
- Summary: Verified bounded corrections and approved prompt strings. Final comparison with the original hard testing standards found two small wording omissions: busy-spin prohibition must reach all test authors, and duration exceptions must be documented in the test. Main owns this one-file correction under the approved 1–2-file policy.
- Ownership: owner=fork; fork_run=373b9da0-baa3-5c10-88fe-614fefed3491; revision=bcbaae2c194f45a95359376f7b17b5f553b26e98; scope=restore retained root rules and remove redundant/unrelated additions; outcome=completed; commit=0ca855fde1967ad466c721877513eecf8a9442fc
- Ownership: owner=main; fork_run=none; revision=0ca855fde1967ad466c721877513eecf8a9442fc; scope=one-file testing-standard clarification: preserve general no-busy-spin rule and locate duration exception evidence in the test; outcome=assigned; commit=none

## Task workflow update - 2026-10-07T03:02:18+00:00
- Validation: Fork's focused prompt/composition/tool/workflow/jbcontext/skill checks passed: 49 tests/216 assertions and broader 76 tests/306 assertions; no cases exceeded 10s. Subsequent commits only alter instruction text, so unchanged runtime evidence reused.; Fork's scoped phpstan and style checks passed; both handoffs confirm testing skill and tests/AGENTS.md read and followed.; Final Castor instruction audit passed all line budgets, frontmatter, links, and source/installed equality. Root 50 lines, testing skill 36, nested files below 20. Updated measured reductions: startup read set 72.3%, focused testing 87.3%.; Final castor docs:validate passed (21 docs); git diff --check passed; worktree clean after commit.; castor test:llm-real passed before transition: 5 tests, 30 assertions, 4.6s. Existing live group warms cache, not a prose-quality evaluation.
- Summary: All approved feedback implemented; clean revision 236a20ad8. Main inspected prompt strings, tests, roles, retained development/architecture/testing rules, and corrected omissions without broadening scope. Reviewer remains waived by the user's PR instruction. Root now 50 lines; testing skill 36. Fixed fork text reduced from 1,841 words/12,556 bytes to 336 words/2,400 bytes (81.7% fewer words), excluding task and inherited context. No delivery protocol changes.
- Ownership: owner=main; fork_run=none; revision=0ca855fde1967ad466c721877513eecf8a9442fc; scope=one-file readiness/duration exception standard clarification; outcome=completed; commit=236a20ad8
- Review exception: independent reviewer remains waived by user for PR #556; no independent approval verdict claimed for revision 236a20ad8.

## Task workflow update - 2026-10-07T03:03:58+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (63.3s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-06-minimize-agent-instructions-and-loaded-skills/var/reports/qa-20261007-030255-11883-9259dffb.
- Session/run: 80.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-07T03:04:00+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-10-06-minimize-agent-instructions-and-loaded-skills to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-06-minimize-agent-instructions-and-loaded-skills/var/reports/qa-20261007-030255-11883-9259dffb.
- Session/run: 80.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-07T03:04:01+00:00
- castor check passed (63.3s).
- Pushed task/2026-10-06-minimize-agent-instructions-and-loaded-skills to origin.
- PR already exists: <url>
- Session/run: 80.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-07T03:04:01+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (63.3s).
- Pushed task/2026-10-06-minimize-agent-instructions-and-loaded-skills to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/556
- Summary: Submit approved feedback and compact fork contract at 236a20ad8. Implementation delegated to one fork with a bounded correction pass; main completed a one-file standards clarification. User waived independent reviewer. Required full gate remains enabled.

## Task workflow update - 2026-10-07T12:52:44+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-10-06-minimize-agent-instructions-and-loaded-skills into integration checkout.
- Merge made by the 'ort' strategy.
 .agents/skills/castor/SKILL.md                                                                      |  34 +----
 .agents/skills/testing/SKILL.md                                                                     | 474 +++++--------------------------------------------------------
 .agents/skills/testing/references/commands.md                                                       |  36 +++++
 .agents/skills/testing/references/diagnostics.md                                                    |  21 +++
 .agents/skills/testing/references/e2e.md                                                            |  32 +++++
 .agents/skills/testing/references/helpers.md                                                        |  14 ++
 .agents/skills/testing/references/live-llm.md                                                       |  24 ++++
 .agents/skills/testing/references/value.md                                                          |  14 ++
 .gitignore                                                                                          |   1 -
 .hatfield/agents/scout.md                                                                           |   2 -
 .hatfield/extensions/extension-api/AGENTS.md                                                        |  11 +-
 .hatfield/extensions/jbcontext/README.md                                                            |   7 +-
 .hatfield/extensions/jbcontext/resources/skills/jbcontext-semantic-search/SKILL.md                  |  35 -----
 .hatfield/extensions/jbcontext/src/Assets/JbcontextAssetInstaller.php                               |  78 +---------
 .hatfield/extensions/jbcontext/src/Assets/JbcontextMarkdownFrontmatter.php                          |  24 ----
 .hatfield/extensions/jbcontext/src/JbcontextExtension.php                                           |   3 +-
 .hatfield/extensions/jbcontext/src/Job/JbcontextEligibilityJobHandler.php                           |   3 +-
 .hatfield/extensions/jbcontext/src/State/JbcontextPaths.php                                         |   2 -
 .hatfield/extensions/jbcontext/tests/Controller/ControllerReplayJbcontextStartupEligibilityTest.php |   1 -
 .hatfield/extensions/jbcontext/tests/JbcontextAssetInstallerTest.php                                |  72 ++--------
 .hatfield/extensions/jbcontext/tests/JbcontextEligibilityJobHandlerTest.php                         |  24 ++--
 .hatfield/extensions/task-workflow/skills/task-workflow/SKILL.md                                    |  58 ++------
 .hatfield/extensions/task-workflow/skills/task-workflow/references/board.md                         |   6 +
 .hatfield/extensions/task-workflow/skills/task-workflow/references/implementation-ownership.md      |  85 +++--------
 .hatfield/extensions/task-workflow/skills/task-workflow/references/task-explain.md                  |   2 +-
 .hatfield/extensions/task-workflow/skills/task-workflow/references/task-start.md                    |   2 +-
 .hatfield/extensions/task-workflow/tests/TaskWorkflowSkillInstallerTest.php                         |   6 +-
 .hatfield/skills/datadog/SKILL.md                                                                   |  41 ++----
 .pi/skills/task-workflow/SKILL.md                                                                   |  44 ++----
 .pi/skills/task-workflow/references/board.md                                                        |   6 +
 .pi/skills/task-workflow/references/implementation-ownership.md                                     |  74 +++-------
 .pi/skills/task-workflow/references/task-explain.md                                                 |   2 +-
 .pi/skills/task-workflow/references/task-start.md                                                   |   2 +-
 AGENTS.md                                                                                           | 203 ++++----------------------
 architecture/documentation-audit.md                                                                 |   4 +-
 src/AgentCore/Application/AGENTS.md                                                                 | 153 ++------------------
 src/AgentCore/Domain/AGENTS.md                                                                      |   8 --
 src/AgentCore/Domain/Event/AGENTS.md                                                                |  31 +---
 src/AgentCore/Domain/Message/AGENTS.md                                                              |  51 +------
 src/AgentCore/Infrastructure/Doctrine/AGENTS.md                                                     |  10 --
 src/CodingAgent/Agent/Fork/ForkChildMessageComposer.php                                             |   2 +-
 src/CodingAgent/Agent/Fork/ForkTaskPromptBuilder.php                                                | 284 ++----------------------------------
 src/CodingAgent/Agent/Tool/ForkToolDefinitionBuilder.php                                            |   1 +
 src/CodingAgent/Resources/skills/subagents/FRONTMATTER.md                                           | 102 -------------
 src/CodingAgent/Resources/skills/subagents/SKILL.md                                                 |  99 +------------
 src/CodingAgent/Resources/skills/subagents/references/frontmatter.md                                |  33 +++++
 src/CodingAgent/Runtime/AGENTS.md                                                                   |  27 ++--
 src/CodingAgent/Runtime/Process/AGENTS.md                                                           |  32 +----
 src/CodingAgent/Runtime/Protocol/AGENTS.md                                                          |  56 ++------
 src/Tui/AGENTS.md                                                                                   |  17 +--
 tests/AGENTS.md                                                                                     | 207 +--------------------------
 tests/CodingAgent/Agent/Fork/ForkChildStartRunInputCompositionTest.php                              |  16 +--
 tests/CodingAgent/Agent/Fork/ForkTaskPromptBuilderTest.php                                          |  62 ++++----
 tests/CodingAgent/Agent/Tool/ForkToolContractTest.php                                               |   1 +
 tests/CodingAgent/Phar/PharSmokeTest.php                                                            |   4 +-
 55 files changed, 495 insertions(+), 2148 deletions(-)
 create mode 100644 .agents/skills/testing/references/commands.md
 create mode 100644 .agents/skills/testing/references/diagnostics.md
 create mode 100644 .agents/skills/testing/references/e2e.md
 create mode 100644 .agents/skills/testing/references/helpers.md
 create mode 100644 .agents/skills/testing/references/live-llm.md
 create mode 100644 .agents/skills/testing/references/value.md
 delete mode 100644 .hatfield/extensions/jbcontext/resources/skills/jbcontext-semantic-search/SKILL.md
 create mode 100644 .hatfield/extensions/task-workflow/skills/task-workflow/references/board.md
 create mode 100644 .pi/skills/task-workflow/references/board.md
 delete mode 100644 src/AgentCore/Domain/AGENTS.md
 delete mode 100644 src/AgentCore/Infrastructure/Doctrine/AGENTS.md
 delete mode 100644 src/CodingAgent/Resources/skills/subagents/FRONTMATTER.md
 create mode 100644 src/CodingAgent/Resources/skills/subagents/references/frontmatter.md
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-10-06-minimize-agent-instructions-and-loaded-skills.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-10-06-minimize-agent-instructions-and-loaded-skills.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR #556 was merged by the user at 2026-10-07T12:50:29Z, merge commit 3540487f37d69e909a614476197392e7617abbaa. Integration and task worktrees inspected clean. Preserved final instruction measurements and transition QA reports under integration var/reports/instruction-context-2026-10-06/pr-feedback before cleanup. Post-merge validation pending.

## Task workflow update - 2026-10-07T12:57:59+00:00
- Updated PR Status: merged
- Validation: Post-merge castor test:llm-real: PASS, 5 tests/30 assertions, 5.6s.; Post-merge castor check: PASS, 140.7s. All 11 lanes passed; 5059 unit/integration tests, 14 controller replay, 6 TUI, 5 live LLM tests. QA reports: var/reports/qa-20261007-125453-340-5b8cb8f8.; QA owned-process/tmux leak check passed; proxy cache guard unchanged at 406 entries. PHAR rebuilt and smoke checks passed.; Git working tree clean; task worktree no longer present; no file differences between integrated HEAD and origin/main.; Final pre-merge measurements and gate reports preserved under var/reports/instruction-context-2026-10-06/pr-feedback.
- Summary: Post-merge validation completed on integrated main beb63b8acea93c3caa0095fcf0bb8c64f1d176cd. Task worktree removed and Git working tree clean. Integrated files match origin/main (3540487f37d69e909a614476197392e7617abbaa); the DONE tool created two additional local merge commits, leaving main ahead of origin/main by 2. History left untouched; no reset, rewrite, or main push performed. Installed workflow refreshed to 1.1.3.
