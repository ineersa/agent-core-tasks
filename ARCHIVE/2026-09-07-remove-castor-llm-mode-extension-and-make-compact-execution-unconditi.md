# Remove Castor LLM-mode extension and make compact execution unconditional

## Goal
User requested removing the Castor extension completely and making the current LLM_MODE behavior the default and only execution mode for project Castor commands.

Remove the castor-llm-mode extension, its registration/configuration references, package wiring, and extension-specific tests/docs. Inspect the legacy Pi equivalent and remove it if still present. Move the required behavior into the existing Castor facilities rather than another extension or wrapper. Remove LLM_MODE branching and obsolete environment configuration; preserve the current compact output, reports, noninteractive behavior, and Castor list formatting where applicable.

Keep QA safety gates, timeouts, isolation, Xdebug defaults and explicit Xdebug overrides unchanged. Do not alter unrelated extensions or add an output-mode setting. Update relevant docs and skills so agents and users no longer need LLM_MODE exports or extension setup.

## Acceptance criteria
- Project Castor commands use the current LLM-mode behavior unconditionally, without needing LLM_MODE or a tool-call rewrite hook.
- The castor-llm-mode extension and any legacy equivalent are removed, including package/configuration wiring, dead code, and extension-specific tests.
- Preserve compact output and report generation across filtered tests, full suites, static checks and the full QA gate; preserve the intended compact Castor list output using existing Castor facilities.
- Documentation and skills describe the single execution mode and contain no obsolete instructions to enable the extension or export LLM_MODE.
- Focused regression validation and the mandatory CODE-REVIEW Castor gate pass without weakening existing QA safety or lifecycle contracts.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/479
PR Status: merged
Started: 2026-09-07T17:18:38+00:00
Completed: 2026-09-07T22:56:17+00:00

## Work log
- Created: 2026-09-07T17:16:09+00:00

## Task workflow update - 2026-09-07T17:18:38+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi.

## Task workflow update - 2026-09-07T17:18:52+00:00
- Summary: Initial routing found mode branches across .castor/phpunit, e2e, tasks, tools, logs, shared and helpers; extension wiring in Composer, QA configs, release splitter and workflow gate exports. Delegating bounded mechanical implementation plus Castor-native startup/list behavior research to one owner. Parent owns runtime settings cleanup after child handoff; no concurrent writers.
- Ownership: owner=fork; fork_run=none; revision=HEAD at task creation; scope=Castor single-mode implementation and removal of extension code/wiring/docs/tests except Hatfield runtime settings; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T17:48:50+00:00
- Validation: Fork confirmed reading/following testing skill and tests/AGENTS.md. Focused extension/workflow/generic-hook tests passed; PHPStan .castor/compact.php 0 errors; deptrac 0 violations; cs-check/docs passed.; Parent: env -u LLM_MODE -u NO_COLOR -u CLICOLOR -u CASTOR_DISABLE_VERSION_CHECK castor list produced compact markdown without extension exports.; Parent same env-unset prefix castor test --filter='QaSessionEnvSanitizationTest|MoveTaskHandlerTest|RegistryBackedToolboxTest': 67 tests/263 assertions passed, no progress/colors; JUnit report emitted. cs-check and docs:validate passed.; No full castor check or independent reviewer in task-start. CODE-REVIEW transition owns full gate; task remains IN-PROGRESS.
- Summary: Implemented in task worktree at 8e0dd4b1a plus settings cleanup 39b7ee5ea. Removed Hatfield and Pi Castor rewrite extensions, Composer/autoload/QA/release wiring, mode helper/branches and obsolete workflow exports. Native Castor listeners supply quiet environment and compact markdown list defaults; project commands always emit existing compact output/reports. Explicit normal Castor options and Xdebug overrides remain available. Parent removed project enabled/settings/fork references via settings tool in integration checkout (320481117) and cherry-picked that settings-only commit into task worktree; no filesystem settings edits. Main otherwise retains old implementation until landing. User session restart needed for extension selection change.
- Ownership: owner=fork; fork_run=none; revision=task creation baseline; scope=single-mode Castor and extension removal except settings; outcome=completed; commit=8e0dd4b1ae3dc7d26a27c282c59ce719c6211e84
- Implementation handoff: artifact=agent_de522a78b33149af; testing prerequisites confirmed; parent reviewed command assembly/lifecycle changes and performed env-unset probes.
- Ownership: owner=main; fork_run=none; revision=8e0dd4b1a; scope=settings cleanup via settings tool and integration validation; outcome=completed; commit=39b7ee5ea
- Proof mapping: deleted extension rewrite tests cover obsolete string rewriting only; default compact behavior now checked using actual Castor list/test/static commands with mode/color/version-check env unset or LLM_MODE=false. Generic rewrite-hook tests retained with NO_COLOR examples; no generic hook removed.

## Task workflow update - 2026-09-07T22:01:24+00:00
- Reviewer: role=reviewer; artifact=agent_d044196ed1ff47cf; revision=39b7ee5ea; scope=full specification-fidelity and correctness; outcome=REQUEST CHANGES. Native list listener uses wrong event name and input options are overwritten by rebind; ContextCreated listener unreachable without named contexts. Prior parent list proof contaminated by still-loaded rewrite hook, withdrawn.
- Ownership: owner=main; fork_run=none; revision=39b7ee5ea; scope=native Castor list lifecycle correction and uncontaminated validation; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T22:07:57+00:00
- Validation: CompactListTest: 1 test, 5 assertions, 0.333s, passed. Child argv execution unsets LLM_MODE/color/version environment and cannot be rewritten by tool hooks.; Uncontaminated variable-executable list probe passed default markdown/no ANSI and explicit JSON override.; Scoped PHPStan .castor/compact.php 0 errors; cs-check 0 fixes; git diff --check passed.; Reviewer independently verified default output byte-identical to explicit --format=md --short, --format=json/txt and --ansi overrides, and bare castor default routing.
- Summary: Review blockers fixed at ff6bea58d. Native AfterBoot installs minimal Symfony ListCommand subclass, preserving framework rendering/options while applying md default and short after input bind. Dead listeners removed. Reviewer agent_d044196ed1ff47cf re-reviewed and APPROVE WITH SUGGESTIONS, no blockers. Earlier contaminated parent list evidence remains withdrawn; new actual subprocess regression plus extension-free reviewer probes provide valid proof. Settings-tool-only violation in first reviewer inspection acknowledged/excluded, not repeated.
- Ownership: owner=main; fork_run=none; revision=39b7ee5ea; scope=native Castor list lifecycle correction and uncontaminated validation; outcome=completed; commit=ff6bea58d
- Reviewer: role=reviewer; artifact=agent_d044196ed1ff47cf; revision=ff6bea58d; scope=blocker fixes, specification fidelity, native list regression; outcome=APPROVE WITH SUGGESTIONS; unresolved blockers=none. Testing prerequisites read and followed.

## Task workflow update - 2026-09-07T22:11:04+00:00
- Summary: Transition gate failed in dead-code lane, qa-20260907-220812-7452-79e238c2/check-dead-code.log. Removed extension was last in-repo caller of published ExtensionApiInterface::registerToolCallRewriteHook. Existing usage provider already retains published registerToolResultHook implementations by exact method/interface; extending same published-contract rule, not removing public API or blanket suppressing analysis. All test lanes and other checks passed this attempt.
- Ownership: owner=main; fork_run=none; revision=ff6bea58d; scope=published rewrite-hook dead-code recognition after caller removal; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T22:14:27+00:00
- Validation: castor dead-code: 0 errors after published-contract rule update; castor cs-check passed; git diff --check passed.
- Ownership: owner=main; fork_run=none; revision=ff6bea58d; scope=published rewrite-hook dead-code recognition after caller removal; outcome=completed; commit=09d44cae9
- Reviewer: role=reviewer; artifact=agent_d044196ed1ff47cf; revision=09d44cae9; scope=published ExtensionApi usage recognition plus whole-task approval; outcome=APPROVE WITH SUGGESTIONS; no blockers.

## Task workflow update - 2026-09-07T22:15:37+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (45.1s).
- Pushed task/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi to origin.
- branch 'task/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi' set up to track 'origin/task/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi'.
- Created PR: https://github.com/ineersa/agent-core/pull/479

## Task workflow update - 2026-09-07T22:16:28+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/479
- Updated PR Status: open
- Validation: Transition-owned deterministic castor check passed in 45.1s. Reports var/reports/qa-20260907-221448-12112-3469ee7c.; JUnit: 4874 cases; zero over 10s; max TuiJourneyE2eTest 6.395121s.
- Summary: Moved to CODE-REVIEW, PR #479 created at 09d44cae9. Worktree clean. Independent review approved; all blockers resolved.

## Task workflow update - 2026-09-07T22:56:17+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi: ide_close_project returned isError.
- Merged task/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi into integration checkout.
- Merge made by the 'ort' strategy.
 .agents/skills/castor/SKILL.md                                                                         |   1 +
 .agents/skills/testing/SKILL.md                                                                        |   5 ++---
 .castor/compact.php                                                                                    |  59 ++++++++++++++++++++++++++++++++++++++++++++++++++++++
 .castor/e2e.php                                                                                        |  45 ++++++++++++++++-------------------------
 .castor/helpers.php                                                                                    |  16 ++-------------
 .castor/logs.php                                                                                       |   8 +++-----
 .castor/phpunit.php                                                                                    |  15 +++++++-------
 .castor/shared.php                                                                                     |  11 +++++-----
 .castor/tasks.php                                                                                      |  11 +++++-----
 .castor/tools.php                                                                                      |  43 ++++++++++++++++-----------------------
 .github/workflows/release.yml                                                                          |   2 --
 .hatfield/extensions/castor-llm-mode/README.md                                                         |  22 --------------------
 .hatfield/extensions/castor-llm-mode/composer.json                                                     |  20 -------------------
 .hatfield/extensions/castor-llm-mode/src/CastorCommandRewriter.php                                     |  53 ------------------------------------------------
 .hatfield/extensions/castor-llm-mode/src/CastorLlmModeExtension.php                                    |  22 --------------------
 .hatfield/extensions/castor-llm-mode/src/CastorLlmModeToolCallHook.php                                 |  37 ----------------------------------
 .hatfield/extensions/castor-llm-mode/tests/CastorCommandRewriterTest.php                               | 124 -----------------------------------------------------------------------------------------------------------------
 .hatfield/extensions/castor-llm-mode/tests/CastorLlmModeToolCallHookTest.php                           |  93 -------------------------------------------------------------------------------------
 .hatfield/extensions/composer.json                                                                     |   5 -----
 .hatfield/extensions/task-workflow/skills/task-workflow/references/task-done.md                        |   2 +-
 .hatfield/extensions/task-workflow/src/Tool/MoveTaskHandler.php                                        |   3 +--
 .pi/extensions/castor-llm-mode.ts                                                                      |  48 --------------------------------------------
 .pi/extensions/task-workflow/index.ts                                                                  |   2 +-
 .pi/skills/task-workflow/references/task-done.md                                                       |   2 +-
 architecture/documentation-audit.md                                                                    |   1 -
 castor.php                                                                                             |   1 +
 composer.json                                                                                          |   2 --
 depfile.yaml                                                                                           |   8 --------
 docs/distribution.md                                                                                   |   2 +-
 docs/settings-agents.md                                                                                |   2 +-
 phpstan.dead-code.neon                                                                                 |   1 -
 phpstan.dist.neon                                                                                      |   1 -
 phpunit.xml.dist                                                                                       |   1 -
 tests/CodingAgent/Agent/Definition/AgentDefinitionParserTest.php                                       |   4 ++--
 tests/CodingAgent/Agent/Execution/Subagent/ChildRun/Preparation/SubagentChildExtensionMetadataTest.php |   3 ---
 tests/CodingAgent/Castor/CompactListTest.php                                                           |  49 +++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Extension/ChildExtensionSelectionServiceTest.php                                     |   8 ++++----
 tests/CodingAgent/Tool/RegistryBackedToolboxTest.php                                                   |   8 ++++----
 tools/phpstan/DeadCode/HatfieldDeadCodeUsageProvider.php                                               |   8 ++++----
 39 files changed, 188 insertions(+), 560 deletions(-)
 create mode 100644 .castor/compact.php
 delete mode 100644 .hatfield/extensions/castor-llm-mode/README.md
 delete mode 100644 .hatfield/extensions/castor-llm-mode/composer.json
 delete mode 100644 .hatfield/extensions/castor-llm-mode/src/CastorCommandRewriter.php
 delete mode 100644 .hatfield/extensions/castor-llm-mode/src/CastorLlmModeExtension.php
 delete mode 100644 .hatfield/extensions/castor-llm-mode/src/CastorLlmModeToolCallHook.php
 delete mode 100644 .hatfield/extensions/castor-llm-mode/tests/CastorCommandRewriterTest.php
 delete mode 100644 .hatfield/extensions/castor-llm-mode/tests/CastorLlmModeToolCallHookTest.php
 delete mode 100644 .pi/extensions/castor-llm-mode.ts
 create mode 100644 tests/CodingAgent/Castor/CompactListTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-remove-castor-llm-mode-extension-and-make-compact-execution-unconditi.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User merged PR #479. GitHub confirmed MERGED at 6fbd2a54a3a65c40e6b83143713cd166f3516803. Integrating and removing task worktree; post-merge gate follows.

## Task workflow update - 2026-09-07T22:59:02+00:00
- Updated PR Status: merged
- Validation: Post-merge castor check failed: var/reports/qa-20260907-225633-16891-5482b9af/check-test.log; 2235 tests/8977 assertions before stop, 1 failure.; Other nine lanes passed, including dead-code, PHPStan, controller replay, TUI and live smoke. Artifact integrity, process/tmux leak assertion and llama-proxy cache guard passed.; git status clean at 12def7027; task worktree absent.
- Summary: DONE transition complete; PR #479 merged, integration revision 12def7027 clean and task worktree removed. Post-merge validation INCOMPLETE: full castor check failed unit lane at BackgroundProcessControllerSessionLifecycleListenerTest::shutdownEventCleansAcceptedState, failed asserting true is false at lines 119/107. Did not blindly retry or modify unrelated lifecycle behavior. JetBrains close degraded during cleanup, but filesystem worktree removal succeeded.

## Task workflow update - 2026-09-07T23:27:51+00:00
- Validation: 4877 JUnit cases; none over 10s; maximum 8.03013s. Process/tmux leak and cache guards passed. Independent review approved follow-up fix.
- Summary: Post-merge validation now complete. Follow-up liveness defect fixed directly on main per user instruction, commits fdf473bb3 and 5ec2dd28f. Full gate qa-20260907-232623-36672-a52a1672 passed all 10 lanes; clean main.

## Task workflow update - 2026-09-10T22:49:40+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
