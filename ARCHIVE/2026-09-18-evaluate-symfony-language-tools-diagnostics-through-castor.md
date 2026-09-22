# Evaluate Symfony Language Tools diagnostics through Castor

## Goal
Evaluate the headless Symfony Language Tools checker and expose useful Symfony-specific diagnostics through a Castor task. Use the existing symfony-lsp check CLI rather than implementing an LSP client. Compare findings with existing PHPStan and other Castor checks, focusing on this HTTP-less application's services, container configuration, and Messenger usage. Keep this separate from the JetBrains index-server removal and any optional JetBrains CLI proposal. Start as an opt-in check; inclusion in castor check is not approved by this task.

## Acceptance criteria
- Evaluate compatibility with the project's Symfony/PHP versions and HTTP-less kernel; record additional actionable findings, overlap with existing checks, and execution time.
- If the evaluation establishes value, add a thin Castor task using the existing headless checker, supporting file/directory selection and machine-readable output.
- Document installation/version requirements, whether analysis executes application code, and command usage.
- Preserve existing PHPStan and QA checks; do not add an LSP daemon, custom LSP client, or mandatory full-gate dependency.

## Workflow metadata
Status: DONE
Branch: task/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor
Fork run: ba95586a-55d6-50c5-8757-2366c9082db9
PR URL: https://github.com/ineersa/agent-core/pull/511
PR Status: merged
Started: 2026-09-18T16:03:19+00:00
Completed: 2026-09-19T18:35:51+00:00

## Work log
- Created: 2026-09-18T14:27:22+00:00

## Task workflow update - 2026-09-18T16:03:19+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor/.idea.
- Summary: User requested trying Symfony Language Tools lsp:check. Existing Symfony CLI is 5.16.1; inspect headless compatibility and evaluate via Castor, leaving mandatory QA unchanged.

## Task workflow update - 2026-09-18T16:04:28+00:00
- Summary: Existing Symfony CLI 5.16.1 predates headless integration. Downloaded official standalone Symfony Language Tools v0.20.3 Linux x64 to worktree-local ignored var/tmp and verified release SHA256SUMS; no global installation/upgrade. Inspection: check supports saved file selection, JSON, explicit kernel and source-only mode; runtime mode executes application code.
- Ownership: owner=main; fork_run=none; revision=2958a742d169e21033853fd485f325f7051b556b; scope=headless Symfony diagnostics evaluation and minimal opt-in Castor integration if useful; outcome=assigned; commit=none

## Task workflow update - 2026-09-18T16:07:48+00:00
- Validation: Cold source-only: complete=true, 1959 files, 0 diagnostics, 7.8578s, exit 0.; Runtime all files: complete=false, 1959 files, 1 warning, 2.9267s, exit 12; services/container/Messenger metadata loaded, routes metadata failed (debug:router absent).; Runtime single-file config/packages/doctrine.yaml: 1 selected file, same actionable warning, 1.8097s, exit 12.; Warm source-only: complete=true, 1959 files, 0 diagnostics, 0.9995s, exit 0.; Finding config/packages/doctrine.yaml:48 doctrine.orm.enable_native_lazy_objects is genuinely deprecated; confirmed vendor/doctrine/doctrine-bundle/src/DependencyInjection/Configuration.php:413-416 (deprecated since 3.1, always enabled, removed in 4.0). Existing PHPStan/full QA passed before this evaluation without blocking on it.; Reports in task worktree var/reports/lsp-evaluation/{source,runtime,single-file,source-warm}.{json,log}. Git status clean.
- Summary: Evaluated official checksum-verified Symfony Language Tools v0.20.3 through a throwaway Castor task, var/tmp/lsp-probe.php. No tracked files changed and no global tool upgraded. Source-only mode works; runtime mode finds a real deprecated Doctrine configuration key but returns incomplete/exit 12 because upstream unconditionally invokes debug:router in our intentionally router-disabled CLI application. The documented v0.20.3 configuration has no per-runtime-section exclusion; current upstream main retains the unconditional route command. Do not add routing to Hatfield, swallow exit 12, or label partial analysis clean. Recommend upstream HTTP-less compatibility fix before retaining a permanent Castor task. Task remains IN-PROGRESS with runtime integration blocker.
- Ownership: owner=main; fork_run=none; revision=2958a742d169e21033853fd485f325f7051b556b; scope=headless Symfony diagnostics evaluation and minimal opt-in Castor integration if useful; outcome=blocked; commit=none
- Sources: https://github.com/symfony/language-tools/blob/v0.20.3/docs/project-configuration.rst ; https://github.com/symfony/language-tools/blob/main/resources/bridge/sections/routes.php . No upstream issue or PR opened.

## Task workflow update - 2026-09-18T16:19:43+00:00
- Validation: Symfony CLI-managed runtime check via Castor: exit 12, same incomplete routes metadata (debug:router missing).; Symfony CLI-managed source-only check via Castor: exit 0.; Reports: task worktree var/reports/lsp-evaluation/symfony-cli-{runtime,source}.{json,log}.
- Summary: User upgraded Symfony CLI to 5.20.0 and allowed its tools cache in bwrap. symfony lsp:check --help now successfully downloads/runs Language Tools 0.20.3. Updated ignored Castor probe to invoke Symfony CLI rather than standalone binary and reran both modes. No tracked code changes.

## Task workflow update - 2026-09-19T15:22:12+00:00
- Validation: v0.21.0 first full runtime: 1959 files, 9.6363s, exit 0, complete=true, one warning.; v0.21.0 warm full runtime: 1959 files, 2.1534s, exit 0, complete=true, one warning.; v0.21.0 single config/packages/doctrine.yaml: 1 file, 1.8088s, exit 0, complete=true, one warning.; Warning remains config.deprecated_key at config/packages/doctrine.yaml:48 for doctrine.orm.enable_native_lazy_objects; no operational errors.; Reports in task worktree var/reports/lsp-evaluation/v021-{runtime,runtime-warm,single-file}.{json,log}. Git status clean.
- Summary: Upgraded Symfony CLI-managed Language Tools to released v0.21.0 after user requested update/retest. Forced manager release refresh by setting only checkedAt=0 in existing user tools state.json, then CLI downloaded official release. Existing Castor probe's isolated HOME also fetched v0.21.0. Router-disabled compatibility blocker is resolved: full runtime and single-file checks now return complete=true, runtime ready, errors=[], exit 0. Mandatory QA and tracked files unchanged; permanent opt-in Castor integration remains pending.

## Task workflow update - 2026-09-19T16:26:25+00:00
- Summary: User explicitly expanded finalized scope: implement standalone castor lsp:check with whole-project/selected-file diagnostics AND include it in mandatory castor check alongside existing lanes. Preserve checker failure on errors/incomplete analysis; warnings non-blocking. Require Symfony CLI with Language Tools >=0.21.0. This supersedes original opt-in-only acceptance restriction. Do not remove existing PHPStan or QA lanes.
- Implementation routing: bounded Castor integration with process/environment isolation and real external-tool failure verification. Delegate implementation and focused validation to isolated fork in existing task worktree; main owns scope and final diff review. No task-to-pr transition authorized yet.

## Task workflow update - 2026-09-19T17:40:11+00:00
- Recorded fork run: ba95586a-55d6-50c5-8757-2366c9082db9
- Validation: Fork confirmed testing skill/tests AGENTS read and followed.; Full runtime lsp:check: 1959 files, complete=true, exit0, one warning; selected doctrine.yaml also passed.; Temporary missing-parameter Autowire probe produced blocking parameter.not_found and exit1, then restored with no source diff.; Missing Symfony CLI probe failed with actionable prerequisite message.; castor test --filter=QaSessionEnvSanitizationTest: 4 tests, 44 assertions passed.; castor phpstan, cs-check, docs:validate (21 docs), git diff --check passed.; Full castor check intentionally deferred to CODE-REVIEW transition.
- Summary: Implemented standalone castor lsp:check (--path optional) and mandatory parallel QA lane in commit 39e4b781908d4e57b1434d2e4ad8454dd65e1628. JSON diagnostics/profile preserved; errors, incomplete/invalid reports, old versions and missing CLI fail; warnings nonblocking. Project-local Symfony CLI cache persists across isolated QA homes. Docs updated. Parent reviewed final diff; full gate and independent reviewer remain for task-to-pr. User fork model setting edit remains uncommitted and separate.
- Ownership: owner=fork; fork_run=ba95586a-55d6-50c5-8757-2366c9082db9; revision=2958a742d; scope=Castor LSP command and mandatory parallel QA lane; outcome=completed; commit=39e4b781908d4e57b1434d2e4ad8454dd65e1628
- Previous failed fork ed649525-4b9d-518a-94ce-a464b4befc30 made no code changes; resume after reload rejected as previous parent lifetime. Replacement fork used reloaded model configuration.

## Task workflow update - 2026-09-19T18:03:54+00:00
- Validation: Followup commit889347e89: stable CLI cache/six transport DSN env regression 5 tests 56 assertions passed.; Actual subdirectory --path selected-file runtime check passed; synthetic timeout exit124 produces explicit failure.; PHPStan and style passed; independent final review approved; full gate delegated to CODE-REVIEW transition.
- Summary: Independent reviewer APPROVE final revision 889347e8988ed42f4154b890c89d02a14deb1591, artifact agent_cf4344b96b2d1de9; scope finalized mandatory LSP gate and standalone command specification fidelity, env/cache isolation, fail semantics, path handling. No blockers. Separate user fork-model settings saved in stash a2b206cb2bf4658f6b0688f0f9dbb48a137fba58 for clean transition; home and integration settings remain unchanged at requested Codex/low.

## Task workflow update - 2026-09-19T18:06:08+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor.md.
- Session/run: 58.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor/var/reports/qa-20260919-180409-3926-6a26949d.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-19T18:16:44+00:00
- Validation: Focused TUI/harness 6 tests61 assertions passed.; Concurrent full TUI9 tests69 assertions and full LSP1959 files passed; slowest TUI case5.051s all <=10s.; PHPStan/style/diff checks passed; no stale workers. No timeout changes, arbitrary sleeps, retries, or process killing added.
- Summary: First gate failed pre-existing TUI cleanup lifecycle race while LSP lane passed. Root evidence: isolated home/.hatfield/ai-catalog.yaml recreated after directory cleanup; no stale workers. Imported exact origin/main9a9439f6 fix as4fe2195cd (tmux pane death verification, nonzero window targeting), then df368d0c adds explicit existing pane-exit synchronization after C-d in two failing tests. Independent reviewer artifact agent_cf4344b96b2d1de9 APPROVE final df368d0c; verified cherry-pick patch-id equivalence, scope and safety.
- Ownership: owner=fork; fork_run=ba95586a-55d6-50c5-8757-2366c9082db9; revision=889347e8988ed42f4154b890c89d02a14deb1591; scope=diagnose and repair gate-blocking preexisting TUI teardown race; outcome=completed; commit=df368d0c028e0323d0b0d8921f5e475cd090a20a

## Task workflow update - 2026-09-19T18:17:53+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (49.6s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor/var/reports/qa-20260919-181704-12948-f1d38f1f.
- Session/run: 58.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-19T18:17:55+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor/var/reports/qa-20260919-181704-12948-f1d38f1f.
- Session/run: 58.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-19T18:17:58+00:00
- castor check passed (49.6s).
- Pushed task/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor to origin.
- Created PR: <url>
- Session/run: 58.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-19T18:17:58+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (49.6s).
- Pushed task/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/511

## Task workflow update - 2026-09-19T18:35:51+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor into integration checkout.
- Auto-merging tests/Tui/E2E/TuiJourneyE2eTest.php
Merge made by the 'ort' strategy.
 .agents/skills/testing/SKILL.md                           |  3 ++-
 .castor/env.php                                           | 14 ++++++++++++++
 .castor/tasks.php                                         | 12 +++++++++---
 .castor/tools.php                                         | 57 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 AGENTS.md                                                 |  4 +++-
 tests/CodingAgent/Castor/QaSessionEnvSanitizationTest.php | 20 ++++++++++++++++++++
 tests/Tui/E2E/TuiJourneyE2eTest.php                       |  1 +
 tests/Tui/E2E/TuiSubagentProgressE2eTest.php              |  1 +
 8 files changed, 107 insertions(+), 5 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-18-evaluate-symfony-language-tools-diagnostics-through-castor.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: Verified PR511 merged on GitHub as0aa53a9b8076fcacf596e563d592473d99bba1f2. Integration has intentional user fork-model settings edit and untracked .hatfield/hatfield.db; preserve both, so requireCleanMain disabled only for these inspected known paths. Duplicate task-local settings preserved in named git stash before worktree cleanup.

## Task workflow update - 2026-09-19T18:37:10+00:00
- Validation: Full gate: 11 lanes green, quality ok150.3s; QA report var/reports/qa-20260919-183558-18509-4aa909a8.; LSP lane passed14.5s; unit5114 tests22078 assertions, controller replay11/175, TUI9/69, llm-real5/30; PHPStan/dead-code0 errors, deptrac/style/docs/catalog passed.; Artifact integrity and owned-process/tmux leak checks passed; exact-run caches cleaned, llama-proxy entries unchanged402→402.
- Summary: Post-merge castor check passed at integrated revision8baf86886a4298bf0abf6536483cff7a22ea3580. Task worktree removed. Integration retains only pre-existing user .hatfield/settings.yaml edit and untracked .hatfield/hatfield.db; neither discarded nor committed.
