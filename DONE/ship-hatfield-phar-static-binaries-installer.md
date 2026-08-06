# Ship Hatfield as PHAR and static binaries with a bash installer

## Goal
## Goal

Turn Hatfield into a versioned, downloadable CLI distribution similar to JoliCode Castor:

- a portable PHAR for users with PHP;
- self-contained native/static binaries for supported Linux and macOS targets;
- reproducible local build commands and a convenience bash build script;
- GitHub Actions artifact/release automation;
- a bash installer supporting latest and pinned versions.

## Existing Hatfield foundation

Hatfield already has a substantial PHAR implementation:

- `castor phar:build`, `phar:ensure`, `phar:clean`, and `phar:info`;
- isolated Box tooling under `tools/phar/`;
- production-only staging and bundled source/config/themes/migrations/internal docs;
- PHAR-aware runtime CWD, cache isolation, project-local settings/extensions, and subprocess resolution;
- PHAR smoke coverage in `tests/CodingAgent/Phar/`.

The missing distribution layer includes native/static binaries, version/build identity, CI artifact publication, release assets, checksums, installer, and local/CI parity.

Before distribution work, harden known PHAR build gaps in `.castor/helpers.php`/`.castor/phar.php`: subprocess exit codes are not consistently enforced, smoke failures only print, stale-input detection is incomplete, an old artifact may survive a failed build, and cleanup/docs are partially inconsistent.

## Reference implementations

### JoliCode Castor

Castor currently:

- builds target PHARs with Box;
- uses `static-php-cli` and PHP micro (`micro:combine`) for static binaries;
- produces Linux amd64/arm64 and macOS amd64/arm64 native binaries, plus platform PHARs including Windows;
- uses local composite GitHub actions for install, PHAR, static build, and static-build cache;
- validates artifacts before upload;
- publishes versioned GitHub Release assets;
- provides a committed bash installer with `--install-dir`, `--version`, and `--static`.

Relevant upstream sources:

- `.github/workflows/artifacts.yml`
- `.github/actions/phar/action.yaml`
- `.github/actions/static/action.yaml`
- `tools/phar/castor.php`
- `tools/static/castor.php`
- `src/Console/Command/CompileCommand.php`
- `tools/release/castor.php`
- `installer/bash-installer`

Castor does not currently publish a standalone checksum file or signatures; Hatfield should at least publish and verify SHA-256 checksums.

### Browser MCP experiment

`/home/ineersa/mcp-servers/browser-mcp/prepare_binary.sh` stages the project, builds a Box PHAR, runs a Docker/static-php-cli build, combines the PHAR into a native binary, and copies both into `dist/`. Reuse the concepts, not the script: its fixed `/tmp` path/container names, root-CWD assumption, lack of traps, single Linux amd64 target, and manual release flow are not suitable as Hatfield’s final pipeline.

## Critical runtime question

Hatfield’s controller and Messenger consumers spawn the current executable through `PharExecutableLocator`, currently represented as `[PHP_BINARY, <phar path>]`. A fused PHP-micro binary must prove that controller/worker subprocesses can relaunch the same native artifact correctly. Do not declare native packaging complete based only on `--version` or `list`; exercise the actual child-process topology.

The canonical PHAR should remain usable independently even if a target-native binary is unavailable.

## Acceptance criteria
- Add canonical version/build metadata sourced from release version and commit, exposed through `hatfield --version`/Symfony application metadata in both PHAR and native artifacts.
- Harden the existing PHAR build so failed Composer/Box/smoke commands fail the Castor task, stale output cannot be mistaken for success, all packaged source/resource/toolchain inputs invalidate the artifact, and clean removes documented staging/output files.
- Define and document release artifact names, including at minimum `hatfield.phar`, `hatfield.linux-amd64`, `hatfield.linux-arm64`, `hatfield.darwin-amd64`, and `hatfield.darwin-arm64`, with versioned GitHub Release assets. Add Windows PHAR support or clearly document/test why the canonical PHAR is platform-neutral.
- Build native binaries with a pinned static-php-cli/PHP-micro toolchain and an explicit minimal extension/library set matching Hatfield runtime requirements.
- Make native Hatfield correctly relaunch itself for controller and Messenger subprocesses; do not rely on a system PHP executable or a sibling source checkout when running the static artifact.
- Provide Castor tasks for local PHAR build, target-specific static build, complete distribution build, artifact verification, and cleanup. All QA/build-tool invocation remains behind Castor.
- Add a trap-safe, concurrency-safe local bash convenience script modeled conceptually on `prepare_binary.sh`. It must resolve the repository root, use unique temporary resources, clean up on failure, accept target/output options, and invoke the same Castor build tasks used by CI rather than duplicating build logic.
- Add GitHub Actions that use the same checked-in Castor tasks to build and verify the target matrix on native runners, cache the pinned static toolchain safely, and upload artifacts with `if-no-files-found: error`.
- Add a release workflow or documented release Castor task that validates the exact commit/tag, expected artifact set, executable smoke results, and artifact sizes before publishing a GitHub Release.
- Generate and publish `SHA256SUMS` for every release artifact. The installer must verify the selected download against the published checksum before installation.
- Commit a bash installer supporting at least `--version`, `--install-dir`, and PHAR-versus-static selection; detect Linux/macOS and amd64/arm64, support `latest`, fail clearly on unsupported targets, use atomic temporary downloads, clean up with traps, and install an executable `hatfield` command.
- For PHAR installation, validate the actual required PHP version/extensions and keep installer checks, `bin/console` guards, Composer requirements, and documentation synchronized.
- Add artifact-level proof from clean temporary directories for version/help/diagnostic boot, bundled defaults/themes/migrations/internal docs, writable-state isolation, project settings/extensions, and failed-install/checksum behavior.
- Add native-artifact process proof that a minimal headless run starts and tears down controller and Messenger children through the same binary. Add the lowest necessary TUI smoke to prove the installed artifact boots in a real terminal without broad journey-test duplication.
- Document local builds, CI/release flow, supported target matrix, system-PHP PHAR requirements, static installation, pinned-version installation, upgrades/reinstallation, checksum verification, and troubleshooting in `docs/phar-packaging.md` and installation-facing documentation.
- Reconcile stale PHAR/runtime checklist documentation and existing tests with the actual packaging pipeline rather than preserving outdated claims.
- Follow the project testing conventions and run focused artifact validation plus mandatory `castor check` before CODE-REVIEW; leaked controller/Messenger/PHPUnit/Castor processes are failures that must be fixed at the lifecycle source.

## Workflow metadata
Status: DONE
Branch: task/ship-hatfield-phar-static-binaries-installer
Worktree: /home/ineersa/projects/agent-core-worktrees/ship-hatfield-phar-static-binaries-installer
Fork run: jtk5c9bnvksu
PR URL: https://github.com/ineersa/agent-core/pull/326
PR Status: merged
Started: 2026-07-27T16:13:47.274Z
Completed: 2026-07-28T16:02:07.915Z

## Work log
- Created: 2026-07-21T21:02:13.656Z

## Task workflow update - 2026-07-27T16:13:47.274Z
- Moved TODO → IN-PROGRESS.
- Created branch task/ship-hatfield-phar-static-binaries-installer.
- Created worktree /home/ineersa/projects/agent-core-worktrees/ship-hatfield-phar-static-binaries-installer.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/ship-hatfield-phar-static-binaries-installer.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/ship-hatfield-phar-static-binaries-installer.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/ship-hatfield-phar-static-binaries-installer.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/ship-hatfield-phar-static-binaries-installer.
- Summary: Claimed for implementation orchestration.

## Task workflow update - 2026-07-27T16:22:05.780Z
- Recorded fork run: 5erwjye4imhw
- Summary: Implementation delegated to fork 5erwjye4imhw in /home/ineersa/projects/agent-core-worktrees/ship-hatfield-phar-static-binaries-installer. Fork brief covers PHAR hardening, embedded version/commit metadata, pinned static-php-cli/PHP-micro targets, native self-relaunch, Castor distribution tasks, build script, checksum-verifying installer, artifact/release workflows, artifact/process/TmuxHarness proofs, synchronized requirements, and packaging/install docs. Full castor check and PR/review explicitly deferred to task-to-pr.
- Scouts mapped existing PHAR pipeline and native controller/Messenger relaunch topology; both confirmed reading testing skill and tests/AGENTS.md.
- Researcher checked JoliCode Castor v1.6.1 and static-php-cli v3 current packaging patterns; immutable SPC commit 59584de4aa9d8067e4ce30d2ff990e7b9e14db43 and four native runner targets supplied to implementer.

## Task workflow update - 2026-07-27T17:12:29.695Z
- Recorded fork run: sa7aehbudkkm
- Validation: Commit a5763b0f8 exists; worktree clean; 27 files changed (+2709/-544).; Recovered focused reports: PHAR 6 passed; build identity 3 passed; installer 2 passed; fused locator 2 passed; TUI artifact test recorded pass but was permissive and is being hardened.; Recovered `castor test` report had 2,769 tests with one MessengerSqliteImmediateTransactionMiddlewareTest failure; no full castor check was run, as required for task-start.; No native artifact was produced by first fork; follow-up must attempt host static build and report exact blocker if environment prevents it.
- Summary: First implementation commit a5763b0f8 was clean and broad but a read-only completion audit found acceptance blockers. Follow-up fork sa7aehbudkkm is fixing complete five-asset release aggregation/checksums, hard native controller/Messenger topology proof, fingerprint-based PHAR freshness including internal-doc targets, complete artifact resource/settings/extension proof, atomic installer rollback, hard TmuxHarness artifact boot evidence, and host static build/verification.

## Task workflow update - 2026-07-27T17:57:21.904Z
- Recorded fork run: owqcu9wx0vhd
- Validation: 6b1f84a81 exists; worktree clean; incremental 14 files (+1130/-188), total task diff 29 files (+3657/-550).; castor phar:build OK; BashInstallerTest 3 OK; PharSmokeTest 7 OK; artifact TmuxHarness test 1 OK; castor test 4534 OK; deptrac/phpstan/cs-check OK.; castor distribution:build-static --target=linux-amd64 blocked locally: missing re2c, flex, gperf. No native artifact/topology execution available locally.; No castor check, push, PR, or reviewer run.
- Summary: Second fork committed 6b1f84a81 and passed focused PHAR/installer/Tmux/unit/deptrac/phpstan/style validation, but final inspection found CI static runners did not install the same re2c/flex/gperf prerequisites that blocked local build, and post-shutdown topology checks could miss reparented children. Final narrow fork owqcu9wx0vhd is adding reusable Linux/macOS CI prerequisites + PR matrix trigger and tracking exact observed child PIDs across shutdown.

## Task workflow update - 2026-07-27T18:03:11.565Z
- Recorded fork run: owqcu9wx0vhd
- Validation: castor phar:build: PASS (PHAR + input fingerprint marker).; castor test --filter=BashInstallerTest: PASS (3 tests, including checksum/smoke rollback).; HATFIELD_BINARY_PATH=<built-phar> castor test --filter=PharSmokeTest: PASS (7 tests).; castor test --filter=ApplicationBuildIdentityTest and FusedNativeExecutableLocatorTest: PASS.; castor test:tui --filter=TuiArtifactBootE2eTest: PASS using real TmuxHarness and built artifact.; castor test: PASS (4534 tests).; castor deptrac: PASS (0 violations); castor phpstan: PASS (0 errors); castor cs-check: PASS.; Final native PID reuse/no-leak helper filter: PASS (1 test, 6 assertions).; castor distribution:verify --skip-topology --allow-missing-native: PASS for PHAR-only local verification.; castor distribution:build-static --target=linux-amd64: BLOCKED locally because re2c, flex, and gperf are not installed. Verification remains hard-fail; PR CI now installs prerequisites on four native runners and runs full static/topology proof.; Full castor check intentionally not run in task-start phase. No push, PR, reviewer, or status transition performed.
- Summary: Implementation complete on task branch in three commits: a5763b0f8 (distribution pipeline), 6b1f84a81 (hard release/topology/freshness/installer/Tmux proofs), ceb0f6020 (CI static prerequisites and pre-captured PID leak assertion). Worktree is clean. Required real TmuxHarness artifact boot proof is present and passed. Complete five-artifact release/checksum pipeline, PR four-target static matrix, pinned SPC toolchain, native self-relaunch, hard controller/Messenger topology verifier, atomic checksum installer, PHAR fingerprinting, synchronized requirements, and docs are implemented.

## Task workflow update - 2026-07-27T19:48:25.213Z
- Recorded fork run: 1gay9zccqif2
- Validation: Reviewer read AGENTS.md, testing skill, tests/AGENTS.md, task/docs/nested runtime instructions and reviewed the distribution diff.; Research verified immutable pins: setup-php f3e473d... = 2.37.2 and fixes GHSA-pqwm-q9pv-ph8r; download-artifact d3f86a... = v4.3.0; action-gh-release 3d0d988... = v3.0.2.; Scout confirmed existing TuiArtifactBootE2eTest uses the actual packaged artifact from isolated CWD through real TmuxHarness, group tui-e2e-replay, no live LLM, visible logo proof, and owned-session teardown; follow-up adds explicit clean-exit proof.
- Summary: Task-to-PR reviewer (deepseek/deepseek-v4-pro per user request) returned REQUEST CHANGES. Independent adjudication rejected its claimed PID-helper extraction and unrelated-file findings as non-blocking/incorrect, but confirmed stale/miscommented GitHub Action pins and missing explicit packaged-TUI clean-exit assertion. Fix fork 1gay9zccqif2 is updating setup-php to security-fixed 2.37.2, download-artifact to v4.3.0, action-gh-release to supported v3.0.2, and hardening the real TmuxHarness artifact exit proof before re-review.

## Task workflow update - 2026-07-27T19:59:37.134Z
- Recorded fork run: j8xw0wmvvt0n
- Validation: Final reviewer: APPROVE; confirmed actual packaged artifact, isolated CWD, tui-e2e-replay, no live LLM, visible boot, clean exit and owned teardown.; castor test: PASS — 4,534 tests, 15,784 assertions.; castor deptrac: PASS — 0 violations/errors.; castor phpstan: PASS — 0 errors.; castor cs-check: PASS — files_fixed=0.; castor test:tui: PASS — 38 tests, 193 assertions, replay-backed with HATFIELD_BINARY_PATH set to built PHAR.; castor clean:cleanup:workers:list: PASS — no stale QA worker candidates.; Local native static build remains environment-blocked by missing re2c/flex/gperf; PR CI installs prerequisites on all four native runners and runs hard native topology verification.
- Summary: Task-to-PR review completed after two fix rounds. Final reviewer on deepseek/deepseek-v4-pro returned APPROVE at eecbb499f with no blockers. Credible review findings fixed in c113f8786 (secure immutable GitHub Action pins; hard real-Tmux packaged-artifact Ctrl+D exit proof) and eecbb499f (--version= support; dead no-op loop deletion). Worktree clean.

## Task workflow update - 2026-07-27T20:03:21.566Z
- Validation: First automatic castor check: FAIL — test:llm-real only, ShellFollowUpLiveE2eTest follow-up assistant response missing.; castor clean:cleanup:workers:list after failure: PASS — no stale candidates.; castor test:llm-real --filter=ShellFollowUpLiveE2eTest: PASS — 2 tests, 21 assertions in 17.7s.
- Summary: First CODE-REVIEW transition gate failed only in unrelated live-LLM ShellFollowUpLiveE2eTest: follow-up emitted command.ack + run.completed without assistant response. No stale QA workers remained. Focused replay against the same live path passed immediately; retrying deterministic full gate.

## Task workflow update - 2026-07-27T20:05:40.107Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (121.2s).
- Pushed task/ship-hatfield-phar-static-binaries-installer to origin.
- branch 'task/ship-hatfield-phar-static-binaries-installer' set up to track 'origin/task/ship-hatfield-phar-static-binaries-installer'.
- Created PR: https://github.com/ineersa/agent-core/pull/326
- Validation: Reviewer APPROVE at eecbb499f.; castor test: PASS (4,534 tests, 15,784 assertions).; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS.; castor test:tui: PASS (38 tests, 193 assertions, replay-backed packaged artifact).; Focused live retry after first gate-only failure: ShellFollowUpLiveE2eTest PASS (2 tests, 21 assertions).; No stale QA worker candidates after validation.; Local native build unavailable because host lacks re2c/flex/gperf; four-target PR CI installs prerequisites and runs hard native topology verification.
- Summary: Approved after deepseek/deepseek-v4-pro review and two focused fix rounds. Ships versioned PHAR + four static target pipeline, checksums/installer, hardened PHAR freshness/fail-fast behavior, native self-relaunch/process topology proof, release automation, and real packaged-artifact Tmux E2E boot/exit proof.

## Task workflow update - 2026-07-27T20:42:28.008Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #326 has no review comments but GitHub reports mergeable=CONFLICTING against current main. Reopening implementation phase to resolve mainline conflicts, revalidate, re-review, and update the PR.

## Task workflow update - 2026-07-27T20:47:52.643Z
- Recorded fork run: amqv0xmvqely
- Validation: Scout found exactly two conflicts at origin/main 716d03465: composer.lock and src/CodingAgent/Runtime/Process/AGENTS.md.; No PR comments/reviews; GitHub mergeable state was CONFLICTING.
- Summary: Conflict-resolution fork launched for PR #326. It will merge current origin/main without rewriting published history, regenerate composer.lock through Castor for combined ext requirements + test-pack removal, preserve implemented runtime distribution docs, run focused unit/architecture/static/style/controller-replay/Tmux validation, and commit the merge.

## Task workflow update - 2026-07-27T21:00:37.600Z
- Recorded fork run: amqv0xmvqely
- Validation: Reviewer APPROVE at merge commit 00e3da4a3.; castor test: PASS — 4,538 tests, 15,865 assertions.; castor deptrac: PASS — 0 violations/errors.; castor phpstan: PASS — 0 errors.; castor cs-check: PASS.; castor test:tui: PASS — 38 tests, 193 assertions using built PHAR.; castor test:controller-replay: PASS — 10 tests, 135 assertions.; castor clean:cleanup:workers:list: PASS — no stale candidates.
- Summary: Resolved PR #326 merge conflict by normal merge commit 00e3da4a3 (parents eecbb499f + origin/main 716d03465), preserving published history. composer.lock combines task ext requirements with main's test-pack/transitive removals; Runtime/Process docs retain implemented locator/distribution state. Final deepseek/deepseek-v4-pro reviewer APPROVED with no issues.

## Task workflow update - 2026-07-27T21:05:18.809Z
- Validation: Automatic castor check after merge: FAIL — controller replay timing assertion + live follow-up timing assertion.; castor test:controller-replay retry: PASS — 10 tests, 135 assertions.; castor test:llm-real --filter=ShellFollowUpLiveE2eTest retry: PASS — 2 tests, 21 assertions.; castor clean:cleanup:workers:list: PASS — no stale candidates.
- Summary: Post-merge CODE-REVIEW gate failed only under parallel full-gate load in two existing runtime tests: ControllerReplaySummaryOnlyGuardTest missed run.completed and ShellFollowUpLiveE2eTest missed follow-up assistant response. Both full controller-replay and focused live paths passed immediately in isolation, with no leaked workers; retrying gate.

## Task workflow update - 2026-07-27T21:07:50.379Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (131.7s).
- Pushed task/ship-hatfield-phar-static-binaries-installer to origin.
- branch 'task/ship-hatfield-phar-static-binaries-installer' set up to track 'origin/task/ship-hatfield-phar-static-binaries-installer'.
- PR already exists: https://github.com/ineersa/agent-core/pull/326
- Validation: Reviewer APPROVE at 00e3da4a3.; castor test: PASS (4,538 tests, 15,865 assertions).; castor deptrac/phpstan/cs-check: PASS.; castor test:tui: PASS (38 tests, 193 assertions).; castor test:controller-replay: PASS (10 tests, 135 assertions), including isolated retry after gate-load timeout.; ShellFollowUpLiveE2eTest isolated retry: PASS (2 tests, 21 assertions).; No stale QA worker candidates.; Local static build remains host-toolchain blocked; PR native matrix is authoritative.
- Summary: Merged current main into PR #326 without history rewrite, resolved composer/runtime-doc conflicts, passed focused integration validation, and received reviewer APPROVE.

## Task workflow update - 2026-07-27T21:08:03.740Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/326
- Updated PR Status: open
- Validation: Automatic castor check: PASS (131.7s).; GitHub mergeability: MERGEABLE.; GitGuardian Security Checks: PASS.; Distribution / Build canonical PHAR: QUEUED.
- Summary: PR #326 updated to merge commit 00e3da4a3 and is now MERGEABLE. Automatic deterministic castor check passed in 131.7s; branch pushed. Distribution CI has queued the canonical PHAR job; GitGuardian passed.

## Task workflow update - 2026-07-27T21:08:42.115Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #326 Distribution CI failed before build because workflow overwrote PATH with `${{ env.PATH }}` (empty in expression context), leaving `/usr/bin/env php` unavailable. Static jobs were skipped. Reopening to fix workflow environment handling and PR build identity, then revalidate/re-review.

## Task workflow update - 2026-07-27T21:09:10.784Z
- Recorded fork run: ljvxaiot2lry
- Validation: GitHub failure evidence: `/usr/bin/env: ‘php’: No such file or directory` with PATH equal only composer-bin suffix after `${{ env.PATH }}` expanded empty.
- Summary: CI fix fork launched for Distribution run 30305547545: replace destructive PATH env overrides with Composer global bin appended through GITHUB_PATH on Linux/macOS, and use stable PR head version/commit identity rather than synthetic 326/merge + merge SHA.

## Task workflow update - 2026-07-27T21:13:04.554Z
- Recorded fork run: ljvxaiot2lry
- Validation: Reviewer APPROVE at 64ccfeb8a.; castor cs-check: PASS.; castor distribution:info: PASS.; castor test --filter=ApplicationBuildIdentityTest: PASS — 3 tests, 10 assertions.; No residual `${{ env.PATH }}` or step PATH overrides in distribution/release workflows.
- Summary: Fixed Distribution CI PATH failure in commit 64ccfeb8a. All five global Castor installs now append Composer's platform-specific global bin directory through GITHUB_PATH; destructive env.PATH overrides are gone. PR builds now embed stable pr-N + head SHA identity. Reviewer APPROVED.

## Task workflow update - 2026-07-27T21:15:14.604Z
- Validation: Automatic castor check: FAIL — TuiImagePasteE2eTest startup logo timeout only.; castor test:tui --filter=TuiImagePasteE2eTest: PASS — 2 tests, 9 assertions.; castor clean:cleanup:workers:list: PASS — no stale candidates.
- Summary: Workflow-only fix gate failed once in unrelated TuiImagePasteE2eTest startup: empty pane, logo missed within 10s under parallel gate load. Focused real-Tmux replay test passed immediately; no leaked workers. Retrying deterministic gate.

## Task workflow update - 2026-07-27T21:17:36.134Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (129.4s).
- Pushed task/ship-hatfield-phar-static-binaries-installer to origin.
- branch 'task/ship-hatfield-phar-static-binaries-installer' set up to track 'origin/task/ship-hatfield-phar-static-binaries-installer'.
- PR already exists: https://github.com/ineersa/agent-core/pull/326
- Validation: Reviewer APPROVE at 64ccfeb8a.; castor cs-check: PASS.; castor distribution:info: PASS.; ApplicationBuildIdentityTest: PASS (3 tests, 10 assertions).; TuiImagePasteE2eTest isolated retry after gate startup timeout: PASS (2 tests, 9 assertions).; No stale QA worker candidates.
- Summary: Fixed PR #326 workflow PATH clobber and unstable PR build identity; reviewer approved the minimal two-workflow fix.

## Task workflow update - 2026-07-27T21:18:30.740Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #326 CI now preserves PATH and correct PR identity, but global Composer installation of Castor is unusable: Castor loads `/home/runner/.composer/vendor/jolicode/castor/vendor/symfony/string/Resources/functions.php`, which does not exist in Composer's flattened global install. Static jobs skipped. Reopening to use Castor's supported CI installation mechanism.

## Task workflow update - 2026-07-27T21:27:52.115Z
- Recorded fork run: r1iyncmvxzqx
- Validation: Run 30306243463 preserved PATH and correct pr-326/head SHA identity, then failed inside Composer-global Castor due missing nested Symfony String file.; Research verified exact v1.6.1 asset SHA-256 values and that setup-php 2.37.2 Castor tooling does not select arm64 assets.
- Summary: Second CI fix fork launched. It will replace unsupported Composer-global Castor installs with one reusable checksum-verified v1.6.1 static Castor composite action supporting all four Linux/macOS amd64/arm64 runners. setup-php's Castor tool is intentionally not used because 2.37.2 hardcodes amd64 assets.

## Task workflow update - 2026-07-27T21:31:28.015Z
- Recorded fork run: r1iyncmvxzqx
- Validation: Reviewer APPROVE at 6cc6722a9; all four Castor v1.6.1 checksums confirmed.; castor cs-check: PASS.; castor distribution:info: PASS.; ApplicationBuildIdentityTest: PASS — 3 tests, 10 assertions.; Five workflow jobs use ./.github/actions/setup-castor; no Composer-global Castor install remains.
- Summary: Replaced unsupported Composer-global Castor with a reusable checksum-verified Castor v1.6.1 static installer composite action in commit 6cc6722a9. All five distribution/release jobs now support Linux/macOS amd64/arm64 without global mutation. Reviewer APPROVED and independently confirmed all four release checksums.

## Task workflow update - 2026-07-27T21:33:55.257Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (132.6s).
- Pushed task/ship-hatfield-phar-static-binaries-installer to origin.
- branch 'task/ship-hatfield-phar-static-binaries-installer' set up to track 'origin/task/ship-hatfield-phar-static-binaries-installer'.
- PR already exists: https://github.com/ineersa/agent-core/pull/326
- Validation: Reviewer APPROVE at 6cc6722a9.; All four Castor v1.6.1 release checksums independently confirmed.; castor cs-check: PASS.; castor distribution:info: PASS.; ApplicationBuildIdentityTest: PASS (3 tests, 10 assertions).
- Summary: Fixed PR #326 Castor installation with a pinned checksum-verified multi-architecture static binary composite action; reviewer approved.

## Task workflow update - 2026-07-27T21:39:27.246Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #326 CI now installs/runs pinned Castor static successfully and builds/smokes Hatfield PHAR, but distribution verification fails because static Castor has empty PHP_BINARY, producing command `'' hatfield.phar --version`. Reopening to install Castor's checksum-verified platform PHAR instead, so Castor runs under setup-php and PHP_BINARY points to host PHP.

## Task workflow update - 2026-07-27T21:39:50.657Z
- Recorded fork run: y0aclt98lnaq
- Validation: Run 30307393015: Install Castor PASS; Hatfield PHAR build/smoke PASS with pr-326/head SHA; distribution verify FAIL because static Castor produced empty PHP_BINARY.; Research verified exact Castor v1.6.1 PHAR asset names/checksums and executable shebang behavior for all four runners.
- Summary: Third CI fix fork launched: keep checksum-verified four-architecture Castor composite action but switch v1.6.1 assets from static binaries to platform PHARs. Direct PHAR execution uses setup-php's host PHP, restoring PHP_BINARY for Hatfield distribution tasks.

## Task workflow update - 2026-07-27T21:42:04.609Z
- Recorded fork run: y0aclt98lnaq
- Validation: Reviewer APPROVE at 7d2f0e9d0.; All four Castor v1.6.1 PHAR checksums independently confirmed.; castor cs-check: PASS.; castor distribution:info: PASS.; ApplicationBuildIdentityTest: PASS — 3 tests, 10 assertions.
- Summary: Switched checksum-verified Castor CI installer from static binaries to v1.6.1 platform PHAR assets in commit 7d2f0e9d0. PHAR shebang uses setup-php host PHP, preserving PHP_BINARY required by Hatfield distribution tasks. Reviewer APPROVED and independently confirmed all four PHAR checksums.

## Task workflow update - 2026-07-27T21:44:30.225Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (129.7s).
- Pushed task/ship-hatfield-phar-static-binaries-installer to origin.
- branch 'task/ship-hatfield-phar-static-binaries-installer' set up to track 'origin/task/ship-hatfield-phar-static-binaries-installer'.
- PR already exists: https://github.com/ineersa/agent-core/pull/326
- Validation: Reviewer APPROVE at 7d2f0e9d0.; All four Castor v1.6.1 PHAR checksums confirmed.; castor cs-check/distribution:info: PASS.; ApplicationBuildIdentityTest: PASS (3 tests, 10 assertions).
- Summary: Fixed PR #326 Castor PHP_BINARY compatibility by installing pinned checksum-verified v1.6.1 platform PHARs on all four runners; reviewer approved.

## Task workflow update - 2026-07-27T22:03:52.379Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #326 PHAR job now passes, but all static matrix jobs exposed platform issues. Linux amd64/arm64: SPC PHP 8.5 micro-ext smoke segfaults after successful CLI/micro builds. Darwin amd64: SPC GitHub API downloads fail without token. Darwin arm64: native artifact boots/version-smokes but topology process inspection sees zero descendants. Reopening for root-cause fixes and another matrix run.

## Task workflow update - 2026-07-27T22:14:29.528Z
- Recorded fork run: v2x9pf2yw3kq
- Validation: PHAR job PASS on run 30308092239.; Linux amd64/arm64 builds reached completed PHP CLI/micro then failed SPC micro_ext_test code 139.; Darwin amd64 failed unauthenticated GitHub API dependency download; SPC supports GITHUB_TOKEN.; Darwin arm64 built and version-smoked native artifact, then topology collector returned zero because macOS ps uses sess/command rather than sid/args.
- Summary: Static matrix repair fork launched for run 30308092239: selectively skip only SPC's crashing PHP 8.5 bare micro extension smoke while retaining all final artifact proofs, pass github.token to SPC downloads, fix macOS sess/command process inspection with hard exit diagnostics, and drain controller stderr during topology polling.

## Task workflow update - 2026-07-27T22:26:27.728Z
- Recorded fork run: 7k8das6vaxnn
- Validation: Final reviewer APPROVE at dced45e84.; NativeProcessTopologyTest focused validation: PASS.; ApplicationBuildIdentityTest: PASS.; castor deptrac/phpstan/cs-check: PASS.; No full local native build due host prerequisites; PR four-runner matrix will execute the repaired paths.
- Summary: Static matrix repair complete across 92181d0e1, f9d5b07f3, and dced45e84. Selectively skips only SPC's crashing PHP 8.5 micro extension smoke, passes GitHub token for dependency downloads, adds Linux/macOS hard-fail process inspection and post-ready pipe diagnostics, and applies reviewer cleanups. Final reviewer verdict: APPROVE.

## Task workflow update - 2026-07-27T22:26:41.858Z
- Validation: Portable process helper filter: PASS — 2 tests, 17 assertions.; Full NativeProcessTopologyTest local filter: expected nonzero due one real native-artifact skip with no local binary; not a regression.; castor phpstan/cs-check: PASS.
- Summary: Validation clarification: full NativeProcessTopologyTest class filter executes 3 tests/17 assertions but exits nonzero because the real native-artifact test intentionally skips without HATFIELD_NATIVE_BINARY_PATH and Castor fails on skips. The two portable PID/process helper tests pass (17 assertions); real topology remains hard-gated by PR native matrix.

## Task workflow update - 2026-07-27T22:28:59.269Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (122.2s).
- Pushed task/ship-hatfield-phar-static-binaries-installer to origin.
- branch 'task/ship-hatfield-phar-static-binaries-installer' set up to track 'origin/task/ship-hatfield-phar-static-binaries-installer'.
- PR already exists: https://github.com/ineersa/agent-core/pull/326
- Validation: Final reviewer APPROVE at dced45e84.; Portable process helper tests: PASS (2 tests, 17 assertions).; ApplicationBuildIdentityTest: PASS.; castor deptrac/phpstan/cs-check: PASS.; Real native topology proof remains PR-matrix gated because local host lacks static prerequisites.
- Summary: Repaired all static-matrix root causes from run 30308092239; final reviewer approved at dced45e84.

## Task workflow update - 2026-07-27T22:44:36.178Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #326 Distribution run 30310878534 still failed Linux arm64: `--no-smoke-test=micro-exts` replaced extension probes with a marker payload but SPC still executes bare micro_ext_test; the bare PHP 8.5 micro executable segfaults (139). Need skip the full upstream bare micro smoke while retaining Hatfield fused artifact hard smokes/topology.

## Task workflow update - 2026-07-27T23:04:39.924Z
- Recorded fork run: lms90b5x1dg1
- Validation: Focused locator/build/topology tests: PASS (6 tests, 19 assertions).; AgentTestExecutable/fused locator filter: PASS (2 tests, 3 assertions).; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS.; Final reviewer: APPROVE.
- Summary: Completed follow-up static fixes at 9e59e60ce + bbdfbd9c9: skip SPC's entire crashing bare PHP 8.5 micro smoke while retaining all fused artifact hard gates; treat empty PHP_BINARY as native self across both production locators and AgentTestExecutable. Final reviewer APPROVE.

## Task workflow update - 2026-07-27T23:06:58.682Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (131.6s).
- Pushed task/ship-hatfield-phar-static-binaries-installer to origin.
- branch 'task/ship-hatfield-phar-static-binaries-installer' set up to track 'origin/task/ship-hatfield-phar-static-binaries-installer'.
- PR already exists: https://github.com/ineersa/agent-core/pull/326
- Validation: Final reviewer APPROVE at bbdfbd9c9.; Focused locator/build/topology tests pass.; castor deptrac/phpstan/cs-check pass.; Four-runner static CI remains required proof.
- Summary: Resolved run 30310878534 Linux bare-micro segfault and Darwin empty-PHP_BINARY native relaunch failures; final reviewer approved bbdfbd9c9.

## Task workflow update - 2026-07-27T23:18:17.922Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User requested documentation review iteration: split oversized docs/phar-packaging.md by concern while keeping PHAR packaging documentation PHAR-specific.

## Task workflow update - 2026-07-27T23:49:38.273Z
- Validation: Independent reviewer/scout audit: REQUEST CHANGES.; Release-only workflow deletion committed at ee0869ba8, not pushed.; PHP research: exact PHP 8.5.8 SHA256 58910198...; phpmicro commit fb6d497b...; SPC supports exact PHP version and custom-local source override.
- Summary: Compliance re-audit after workflow-scope correction found additional concrete gaps: static release jobs overwrite downloaded canonical PHAR; core PHP/phpmicro inputs float; local wrapper's temp dir is unused and sequence unlocked; PHAR-only wrapper verifies as if native exists; --release-version space form missing; post-replacement installer smoke cannot restore previous install. Fix round required before PR update.

## Task workflow update - 2026-07-28T00:08:56.371Z
- Validation: Reviewer: APPROVE WITH SUGGESTIONS (no critical findings).; Scout traced pinned SPC custom-local + exact PHP archive hash successfully.; Scout blocker: release.yml unconditional sha256sum fails standard macOS runners.; No push; task remains IN-PROGRESS.
- Summary: Review of ce729508c: deepseek reviewer APPROVE WITH SUGGESTIONS, but adversarial scout found one release blocker and three correctness issues requiring another fix: macOS release uses unavailable sha256sum; wrapper INT/TERM trap unlocks then may continue; installer still logs after atomic replacement; PHAR docs retain stale phar_ensure claim. Also lock test can overwrite a real holder and canonical handoff test is source-regex brittle.

## Task workflow update - 2026-07-28T00:25:52.739Z
- Validation: Final reviewer on 96afb0651: APPROVE.; castor test: PASS (4545 tests, 15917 assertions).; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS (files_fixed=0).; castor test:tui: PASS (38 tests, 193 assertions, replay-backed PHAR artifact).; castor clean:cleanup:workers:list: PASS (no stale QA workers).; Worktree clean at 96afb06516089144adea32aa2889714a177d1b30.
- Summary: Final compliance corrections complete through 96afb0651: GitHub packaging is v* tag-release only; docs split by PHAR/static/distribution concern; canonical PHAR handoff preserved byte-for-byte into static jobs; exact PHP 8.5.8 source + phpmicro commit pinned; wrapper lock/signal/PHAR-only/parser fixed; installer atomic failure semantics and safe version validation fixed. Final deepseek reviewer APPROVE with no remaining blockers.

## Task workflow update - 2026-07-28T00:28:15.923Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (130.7s).
- Pushed task/ship-hatfield-phar-static-binaries-installer to origin.
- branch 'task/ship-hatfield-phar-static-binaries-installer' set up to track 'origin/task/ship-hatfield-phar-static-binaries-installer'.
- PR already exists: https://github.com/ineersa/agent-core/pull/326
- Validation: castor test PASS: 4545 tests, 15917 assertions; castor deptrac PASS: 0 violations; castor phpstan PASS: 0 errors; castor cs-check PASS: files_fixed=0; castor test:tui PASS: 38 tests, 193 assertions; Final reviewer APPROVE; No stale QA workers
- Summary: Final review iteration approved. Push five local commits 95dbe92d8..96afb0651 to update PR #326; packaging workflow now exists only for v* tags, so PR update must not generate artifacts.

## Task workflow update - 2026-07-28T01:50:16.433Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User requested merge-up from origin/main followed by a ponytail over-engineering review of the full task diff.

## Task workflow update - 2026-07-28T02:13:02.892Z
- Validation: castor test PASS: 4414 tests, 15510 assertions; castor deptrac PASS: 0 violations; castor phpstan PASS: 0 errors; castor cs-check PASS: files_fixed=0; castor test:tui PASS: 38 tests, 193 assertions; No stale QA workers; Reviewer on 485304c7f: APPROVE; Ponytail: Lean already. Ship.; Worktree clean at 485304c7f6448bd14c24f227d7a295d2e08f80b4
- Summary: Merged origin/main 15f6336de cleanly as da0aedcf1, then applied user-approved ponytail simplification at 485304c7f: 9 files +73/-532 (net -459), primarily collapsing duplicated native topology test into production Castor proof. Final reviewer APPROVE; no packaging/runtime gates weakened.

## Task workflow update - 2026-07-28T02:15:18.715Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (126.9s).
- Pushed task/ship-hatfield-phar-static-binaries-installer to origin.
- branch 'task/ship-hatfield-phar-static-binaries-installer' set up to track 'origin/task/ship-hatfield-phar-static-binaries-installer'.
- PR already exists: https://github.com/ineersa/agent-core/pull/326
- Validation: castor test PASS: 4414 tests, 15510 assertions; castor deptrac PASS: 0 violations; castor phpstan PASS: 0 errors; castor cs-check PASS; castor test:tui PASS: 38 tests, 193 assertions; Final reviewer APPROVE; No stale QA workers
- Summary: Merged latest main and applied approved ponytail simplification (net -459 lines). Push da0aedcf1 + 485304c7f to update PR #326; release-only packaging invariant remains.

## Task workflow update - 2026-07-28T16:02:07.915Z
- Moved CODE-REVIEW → DONE.
- Merged task/ship-hatfield-phar-static-binaries-installer into integration checkout.
- Auto-merging AGENTS.md
Merge made by the 'ort' strategy.
 .castor/cleanup.php                                |   14 +
 .castor/distribution.php                           | 1331 ++++++++++++++++++++
 .castor/e2e.php                                    |   11 +
 .castor/helpers.php                                |  704 +++++++----
 .castor/phar.php                                   |   37 +-
 .castor/phpunit.php                                |   14 +-
 .github/actions/setup-castor/action.yml            |  111 ++
 .github/actions/static-prerequisites/action.yml    |   58 +
 .github/workflows/release.yml                      |  256 ++++
 AGENTS.md                                          |    2 +
 README.md                                          |   20 +
 bin/console                                        |    9 +-
 castor.php                                         |    1 +
 composer.json                                      |   13 +
 composer.lock                                      |   17 +-
 docs/distribution.md                               |  115 ++
 docs/phar-packaging.md                             |  290 ++---
 docs/static-packaging.md                           |  110 ++
 installer/bash-installer                           |  320 +++++
 scripts/build-distribution.sh                      |  218 ++++
 src/CodingAgent/Build/ApplicationBuildIdentity.php |  158 +++
 src/CodingAgent/Runtime/Process/AGENTS.md          |   82 +-
 .../Runtime/Process/ConfigExecutableLocator.php    |   41 +-
 .../Runtime/Process/PharExecutableLocator.php      |   39 +-
 .../Build/ApplicationBuildIdentityTest.php         |   65 +
 .../CodingAgent/Distribution/BashInstallerTest.php |  222 ++++
 .../Distribution/BuildDistributionScriptTest.php   |  199 +++
 .../Distribution/CanonicalPharHandoffTest.php      |  116 ++
 .../Distribution/NativeProcessTopologyTest.php     |   56 +
 tests/CodingAgent/Phar/PharSmokeTest.php           |  182 ++-
 .../Process/FusedNativeExecutableLocatorTest.php   |   55 +
 tests/CodingAgent/Support/AgentTestExecutable.php  |   45 +-
 tests/Tui/E2E/TmuxHarness.php                      |   21 +
 tests/Tui/E2E/TuiArtifactBootE2eTest.php           |  169 +++
 tools/static/README.md                             |   45 +
 tools/static/pin.json                              |   34 +
 36 files changed, 4622 insertions(+), 558 deletions(-)
 create mode 100644 .castor/distribution.php
 create mode 100644 .github/actions/setup-castor/action.yml
 create mode 100644 .github/actions/static-prerequisites/action.yml
 create mode 100644 .github/workflows/release.yml
 create mode 100644 docs/distribution.md
 create mode 100644 docs/static-packaging.md
 create mode 100755 installer/bash-installer
 create mode 100755 scripts/build-distribution.sh
 create mode 100644 src/CodingAgent/Build/ApplicationBuildIdentity.php
 create mode 100644 tests/CodingAgent/Build/ApplicationBuildIdentityTest.php
 create mode 100644 tests/CodingAgent/Distribution/BashInstallerTest.php
 create mode 100644 tests/CodingAgent/Distribution/BuildDistributionScriptTest.php
 create mode 100644 tests/CodingAgent/Distribution/CanonicalPharHandoffTest.php
 create mode 100644 tests/CodingAgent/Distribution/NativeProcessTopologyTest.php
 create mode 100644 tests/CodingAgent/Runtime/Process/FusedNativeExecutableLocatorTest.php
 create mode 100644 tests/Tui/E2E/TuiArtifactBootE2eTest.php
 create mode 100644 tools/static/README.md
 create mode 100644 tools/static/pin.json
- Removed worktree /home/ineersa/projects/agent-core-worktrees/ship-hatfield-phar-static-binaries-installer.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/ship-hatfield-phar-static-binaries-installer.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #326 merged at 2026-07-28T15:19:40Z; Release run 30372889350: PHAR PASS, 3/4 static targets PASS, linux-arm64 FAIL, publish SKIPPED; Failure evidence: 13 owned descendants captured; extension_agent alone survived controller shutdown >10s; No complete release assets published
- Summary: PR #326 was merged by user as a9b8cd74c. First v0.0.1 release matrix proved PHAR, linux-amd64, darwin-amd64, darwin-arm64 but exposed a linux-arm64 controller shutdown race: the last extension_agent consumer survived after the controller was killed. Release publish job was skipped; GitHub release exists with zero assets. Follow-up task will fix lifecycle and Node24 action pins.
