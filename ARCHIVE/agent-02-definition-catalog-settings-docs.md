# AGENT-02 Agent definition catalog, discovery, settings, and docs

## Goal
Build the production agent definition catalog/registry on top of AGENT-01. This task is about locating, loading, validating, and listing definitions. It must not implement agent execution, hidden child runs, artifacts, or TUI controls.

Context:
- Depends on AGENT-01.
- Reference plan: `.pi/plans/agents-subagents-implementation-plan.md`.
- Discovery must support Hatfield-native and cross-tool agent locations.
- `.agents/` support is first-class, not a legacy fallback.
- Avoid compatibility/fallback layers; document and enforce one deterministic precedence model.

Required discovery precedence to confirm/implement/document:
1. builtin agents bundled with Hatfield
2. user agents under `~/.hatfield/agents/`
3. user agents under `~/.agents/`
4. project agents under `.hatfield/agents/`
5. project agents under `.agents/`

Suggested scope:
- `AgentDefinitionCatalog` / registry service.
- Discovery loaders for builtin/user/project locations.
- Settings keys for enabling/disabling agents and configuring definition directories if needed.
- Deterministic override behavior by agent name.
- Disabled definitions are reported/listed appropriately but not launchable later.
- Initial builtin definition files for `scout`, `reviewer`, `researcher`, and `worker` may be added if they can be validated by the catalog, but no execution behavior should be wired.

## Acceptance criteria
- Catalog can discover and validate definitions from builtin, user, and project locations using deterministic documented precedence.
- Project definitions override user definitions by name; user/project `.agents/` locations are first-class.
- Duplicate/invalid definitions produce actionable diagnostics without silently falling back to another format.
- Settings changes, if any, are reflected in `.hatfield/settings.yaml` and `docs/settings.md`.
- `docs/agents.md` documents definition format, discovery paths, precedence, and disabled definitions.
- No agent launch/runtime/TUI/artifact behavior is implemented.
- Validation uses Castor commands only, with focused catalog/parser tests and `castor phpstan`/`castor deptrac` as relevant.

## Workflow metadata
Status: DONE
Branch: task/agent-02-definition-catalog-settings-docs
Worktree: /home/ineersa/projects/agent-core-worktrees/agent-02-definition-catalog-settings-docs
Fork run: 2b521d646
PR URL: https://github.com/ineersa/agent-core/pull/191
PR Status: merged
Started: 2026-06-21T23:19:56.431Z
Completed: 2026-06-22T01:47:44.852Z

## Work log
- Created: 2026-06-15T22:52:17.143Z

## Task workflow update - 2026-06-21T23:19:56.431Z
- Moved TODO → IN-PROGRESS.
- Created branch task/agent-02-definition-catalog-settings-docs.
- Created worktree /home/ineersa/projects/agent-core-worktrees/agent-02-definition-catalog-settings-docs.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/agent-02-definition-catalog-settings-docs.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/agent-02-definition-catalog-settings-docs.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/agent-02-definition-catalog-settings-docs.
- Summary: Starting AGENT-02. Initial context read: task file and `.pi/plans/agents-subagents-implementation-plan.md`; loaded task-workflow and testing skills; read tests/AGENTS.md. Scope is catalog/discovery/settings/docs on top of AGENT-01 only; no launch/runtime/TUI/artifact behavior.

## Task workflow update - 2026-06-21T23:25:30.884Z
- Summary: Orchestrator prep/scout context gathered. AGENT-01 currently provides `AgentDefinitionDTO`, `AgentDefinitionParser`, frontmatter DTO/parser, MCP/system-prompt enums, and validation exception under `src/CodingAgent/Agent/Definition`; parser is single-file only and has no catalog/discovery consumer. Scouts found closest reusable patterns: `SkillDiscovery`/`SkillRegistry` for path scanning, caching, collision diagnostics, and body reading; `PromptsConfig`/`AppConfigLoader::PATH_CONFIG` for settings path resolution; `AppResourceLocator` for bundled resources; `PromptTemplateLoader` for non-recursive `*.md` discovery; `docs/settings.md` for settings docs. Critical design corrections for implementation: do NOT reintroduce `type`/`AgentTypeEnum` (removed in AGENT-01 review); implement catalog/discovery only, no launch/runtime/TUI/artifacts; deterministic precedence should be documented as builtin < user `~/.hatfield/agents` < user `~/.agents` < project `.hatfield/agents` < project `.agents` (and any configured `agents.paths` if added, explicitly documented); project definitions must override user definitions by name. `AppAgent` deptrac currently allows AppMarkdown/SymfonyYaml/SymfonySerializer/SymfonyValidator only, so using `AppConfig`/`AppResourceLocator`/`SettingsPathResolver` from agent discovery requires adding AppConfig to AppAgent allowed deps. Settings/docs gaps: no `agents` section exists in `config/hatfield.defaults.yaml`, `.hatfield/settings.yaml`, `docs/settings.md`, or `AppConfig` yet; no `docs/agents.md` or `config/agents/*.md` builtins exist.

## Task workflow update - 2026-06-21T23:34:37.815Z
- Recorded fork run: 2b521d646
- Summary: First fork completed commit `2b521d646` with catalog/discovery/settings/docs implementation and passing validations, but parent verification found one acceptance gap before accepting: `agents.enabled` exists and is documented, but `AgentDefinitionDiscovery::discover()` does not honor it. Need a narrow follow-up fork to return an empty catalog when global agent discovery is disabled and add a focused test. Other parent checks: worktree clean; diff stat shows expected new catalog/discovery/config/docs/builtin/tests files; grep confirms no `AgentTypeEnum` and no frontmatter `type:` in production/builtins/docs except diagnostic `type` strings.

## Task workflow update - 2026-06-21T23:37:01.316Z
- Validation: Parent verification: `git status --short --branch` clean on worktree branch `task/agent-02-definition-catalog-settings-docs`.; Parent verification: `git diff --stat main...HEAD` shows 21 expected files changed (catalog/discovery/config/docs/builtins/tests), 1708 insertions.; Parent grep verification: no `AgentTypeEnum` and no frontmatter `type:` in `src/CodingAgent/Agent`, `config/agents`, `docs/agents.md`, or agent tests; only diagnostic DTO `type` strings exist.; Parent read verification: `AgentDefinitionDiscovery::discover()` now returns/caches empty `AgentDefinitionCatalog` when `!$this->agentsConfig->enabled` before any scans.; `castor test --filter=AgentDefinitionDiscoveryTest` passed: OK (14 tests, 35 assertions).; `castor test --filter=AgentDefinitionCatalogTest` passed: OK (15 tests, 36 assertions).; `castor test --filter=AgentsConfigTest` passed: OK (9 tests, 19 assertions).; `castor test --filter=AgentDefinitionParserTest` passed: OK (74 tests, 175 assertions).; `castor phpstan` passed: errors=0, file_errors=0.; `castor deptrac` passed: violations=0, errors=0.; `castor cs-check` passed: files_fixed=0 / no fixable issues.
- Summary: Implementation complete and verified on branch `task/agent-02-definition-catalog-settings-docs`. Final HEAD `2cd88bf0a` includes initial catalog/discovery/settings/docs commit `2b521d646` plus follow-up `2cd88bf0a` honoring `agents.enabled=false`. Implemented `AgentDefinitionCatalog`, `AgentDefinitionDiscovery`, `AgentDefinitionDiagnosticDTO`, `AgentsConfig`, built-in agent definitions (`config/agents/{scout,reviewer,researcher,worker}.md`), settings integration (`agents.enabled`, `agents.paths`), DI wiring, deptrac rule update, `docs/agents.md`, `docs/settings.md`, and focused tests. Scope stayed within catalog/discovery/settings/docs only: no launch/runtime/TUI/artifact behavior and no `type`/`AgentTypeEnum` reintroduced. Parent verified git status clean and expected diff stat (21 files, 1708 insertions).

## Task workflow update - 2026-06-21T23:47:05.027Z
- Summary: Reviewer subagent returned APPROVE WITH SUGGESTIONS for HEAD `2cd88bf0a`. No critical/security/blocking issues. Actionable findings to address before PR: configured explicit non-`.md` file paths are silently skipped instead of diagnostic; `loadFile()` catches only `AgentDefinitionValidationException` so unexpected parser throwables can abort all discovery; stale test comment says unknown `type` while fixture uses `invalid`; docs use `websearch__*`/`context7__*` wildcard shorthand for researcher tools while built-in enumerates concrete tool IDs. Non-actionable/subjective findings left for later: cwd injection convention vs AppConfig, redundant defensive branch in `loadDirectory`, possible case-insensitive `.MD` support, PHAR packaging note, whitespace-leading paths edge case.

## Task workflow update - 2026-06-21T23:55:39.571Z
- Validation: Reviewer subagent initial decision for `2cd88bf0a`: APPROVE WITH SUGGESTIONS, no critical/security/blocking issues.; Fork commit `52001e650` (`Address agent discovery review feedback`) fixed actionable reviewer findings.; Reviewer subagent re-review decision for `52001e650`: APPROVED.; `castor test` passed: OK (3188 tests, 10136 assertions).; `castor deptrac` passed: violations=0, errors=0, uncovered=972, allowed=1307.; `castor phpstan` passed: errors=0, file_errors=0.; `castor cs-check` passed: files_fixed=0 / no fixable issues.; Pre-CODE-REVIEW git status clean on branch `task/agent-02-definition-catalog-settings-docs`.
- Summary: Code review complete for current HEAD `52001e650`. Initial reviewer returned APPROVE WITH SUGGESTIONS; fork commit `52001e650` addressed actionable findings (explicit non-`.md` configured file emits `invalid_path`, unexpected parser throwables become `invalid_definition` diagnostics + warning, stale test comment fixed, docs clarify wildcard shorthand). Re-review returned APPROVED for `52001e650`; only non-blocking nice-to-haves remained (cwd convention alignment, docs table value clarity, optional direct throwable-catch test, optional assertion that invalid configured non-md path does not prevent builtin discovery, `.MD` case-sensitivity note). Local validation passed. Worktree status clean.

## Task workflow update - 2026-06-21T23:57:50.588Z
- Validation: First move_task CODE-REVIEW: castor check failed only `test:controller-replay` with exit code 124.; `castor test:controller-replay` rerun passed: OK (3 tests, 41 assertions), errors=0, failures=0, skipped=0.
- Summary: First CODE-REVIEW transition failed because deterministic castor check timed out only in `test:controller-replay` lane (exit code 124). Logs showed other lanes passed (`test`, `test:tui`, `phpstan`, `deptrac`, `cs-check`) and controller-replay log had no assertion failure output. Stale process check found no lingering controller/messenger/phpunit/castor processes for this worktree. Focused rerun `castor test:controller-replay` passed, confirming timeout was transient/unrelated to AGENT-02 changes. Retrying CODE-REVIEW transition.

## Task workflow update - 2026-06-21T23:59:21.799Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (73.0s).
- Pushed task/agent-02-definition-catalog-settings-docs to origin.
- branch 'task/agent-02-definition-catalog-settings-docs' set up to track 'origin/task/agent-02-definition-catalog-settings-docs'.
- Created PR: https://github.com/ineersa/agent-core/pull/191
- Validation: Reviewer subagent re-review: APPROVED for `52001e650`.; `castor test` passed: OK (3188 tests, 10136 assertions).; `castor deptrac` passed: violations=0, errors=0.; `castor phpstan` passed: errors=0, file_errors=0.; `castor cs-check` passed: files_fixed=0 / no fixable issues.; First CODE-REVIEW castor check attempt failed only `test:controller-replay` timeout; focused `castor test:controller-replay` rerun passed: OK (3 tests, 41 assertions).
- Summary: Prepared for PR/code review at HEAD `52001e650`. Reviewer subagent re-review approved. Local focused Castor validation passed (`castor test`, `castor deptrac`, `castor phpstan`, `castor cs-check`); after a transient first castor-check controller-replay timeout, focused `castor test:controller-replay` rerun passed. Scope remained catalog/discovery/settings/docs only; no launch/runtime/TUI/artifact behavior; no `type`/`AgentTypeEnum` reintroduced.

## Task workflow update - 2026-06-22T00:25:55.569Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Moving back for PR #191 review iteration. User left two inline comments rejecting bundled built-in agent definitions and built-in `config/agents` discovery: comment on `config/agents/researcher.md` says existing `.agents` should be used instead, and comment on `config/hatfield.defaults.yaml` says 'No bundles, we respect .agents.' Actionable classification: remove bundled `config/agents/*.md`, remove built-in config/agents discovery layer/docs/defaults/tests, and keep discovery/settings focused on `.agents`/`.hatfield/agents` plus explicit `agents.paths`. No runtime/TUI scope.

## Task workflow update - 2026-06-22T00:46:42.725Z
- Validation: PR comments read via GitHub: `config/agents/researcher.md` comment rejected added bundled agents; `config/hatfield.defaults.yaml` comment rejected built-in bundles and requested respecting `.agents`.; Fork commit `74e1f3c40` (`Remove bundled agent definitions`) addressed both comments.; Fork commit `3442a3c05` (`Clarify custom agent role examples`) addressed reviewer NTH ambiguity around example role names implying built-ins.; Parent grep verification: no remaining agent-specific `config/agents`, `builtin agents`, `built-in agents`, `bundled agents`, `acme-bundled`, or `getBuiltinAgentsPath` references in src/config/docs/tests except unrelated generic built-in defaults/themes references.; `castor test --filter=AgentDefinitionDiscoveryTest` passed: OK (14 tests, 37 assertions).; `castor test --filter=AgentDefinitionCatalogTest` passed: OK (15 tests, 36 assertions).; `castor test --filter=AgentsConfigTest` passed: OK (9 tests, 19 assertions).; `castor phpstan` passed: errors=0, file_errors=0.; `castor deptrac` passed: violations=0, errors=0, uncovered=972, allowed=1306.; `castor cs-check` passed: files_fixed=0 / no fixable issues.; Reviewer subagent final re-review at `3442a3c05`: APPROVE WITH SUGGESTIONS, no actionable blockers.
- Summary: PR #191 review feedback addressed. User rejected bundled `config/agents/*.md` definitions and built-in bundle discovery; fork commit `74e1f3c40` removed all bundled agent definition files, removed the built-in `config/agents` discovery layer and `AppResourceLocator::getBuiltinAgentsPath()`, and updated docs/defaults/tests to use only `.hatfield/agents`, `.agents`, and configured `agents.paths`. Follow-up commit `3442a3c05` clarified remaining prose so `scout`/`reviewer`/`researcher`/`worker` are examples users can define, not shipped roles. Reviewer re-review at `3442a3c05` returned APPROVE WITH SUGGESTIONS with no actionable blockers; only NTH was a direct transitive precedence test and reviewer explicitly said it is not worth adding in this iteration. Worktree clean and ready to push/update PR.

## Task workflow update - 2026-06-22T00:48:42.758Z
- Validation: First move_task CODE-REVIEW after PR feedback: castor check failed `test:controller-replay` exit 124 and `test:tui` exit 2 in `TuiCompactCommandE2eTest::testCompactCommandVisibleAndFunctional`.; `castor test:controller-replay` focused rerun passed: OK (3 tests, 41 assertions).; `castor test:tui --filter=TuiCompactCommandE2eTest` focused rerun passed: OK (2 tests, 8 assertions).
- Summary: First CODE-REVIEW retry after PR feedback failed deterministic castor check only in replay E2E lanes: `test:controller-replay` timed out with no assertion output and `test:tui` failed `TuiCompactCommandE2eTest::testCompactCommandVisibleAndFunctional` waiting for a compaction failure block. These lanes are unrelated to AGENT-02 docs/catalog changes. Stale process check found no lingering worktree controller/messenger/phpunit/castor processes. Focused reruns passed for both failing lanes, so retrying CODE-REVIEW transition.

## Task workflow update - 2026-06-22T00:52:51.060Z
- Validation: Second move_task CODE-REVIEW retry: castor check failed only `test:controller-replay` exit 124; controller replay log contained only PHPUnit header and no assertion failure.; After unrelated concurrent validation processes finished, focused `castor test:controller-replay` passed: OK (3 tests, 41 assertions).
- Summary: Second CODE-REVIEW retry failed deterministic castor check only in `test:controller-replay` with exit code 124 and no assertion output. At the time, another task worktree was running TUI/replay validation processes, likely contributing resource contention; those unrelated processes finished. Focused `castor test:controller-replay` in AGENT-02 worktree passed again. Retrying CODE-REVIEW after confirming no competing worktree processes remain.

## Task workflow update - 2026-06-22T00:55:27.793Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (69.4s).
- Pushed task/agent-02-definition-catalog-settings-docs to origin.
- branch 'task/agent-02-definition-catalog-settings-docs' set up to track 'origin/task/agent-02-definition-catalog-settings-docs'.
- PR already exists: https://github.com/ineersa/agent-core/pull/191
- Validation: PR comments addressed: no bundled agent definitions, no built-in `config/agents` discovery; discovery now respects `.agents`/`.hatfield/agents` plus `agents.paths`.; Reviewer subagent final re-review: APPROVE WITH SUGGESTIONS, no actionable blockers.; `castor test --filter=AgentDefinitionDiscoveryTest` passed: OK (14 tests, 37 assertions).; `castor test --filter=AgentDefinitionCatalogTest` passed: OK (15 tests, 36 assertions).; `castor test --filter=AgentsConfigTest` passed: OK (9 tests, 19 assertions).; `castor phpstan` passed: errors=0, file_errors=0.; `castor deptrac` passed: violations=0, errors=0.; `castor cs-check` passed: files_fixed=0 / no fixable issues.; Focused rerun after prior gate failure: `castor test:controller-replay` passed OK (3 tests, 41 assertions).; Focused rerun after prior gate failure: `castor test:tui --filter=TuiCompactCommandE2eTest` passed OK (2 tests, 8 assertions).
- Summary: PR #191 review feedback addressed at HEAD `3442a3c05`. Removed bundled `config/agents/*.md` definitions and built-in `config/agents` discovery in favor of `.hatfield/agents`, `.agents`, and explicit `agents.paths`; removed `getBuiltinAgentsPath()`; updated docs/defaults/tests accordingly; clarified role names as examples only. Reviewer subagent final re-review found no actionable blockers. Focused validation passed. Prior CODE-REVIEW retries hit unrelated replay E2E timeouts/flakes during concurrent validation in another worktree; focused reruns passed, and retry is after competing processes finished.

## Task workflow update - 2026-06-22T01:47:44.852Z
- Moved CODE-REVIEW → DONE.
- Merged task/agent-02-definition-catalog-settings-docs into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/settings.yaml                            |  13 +
 AGENTS.md                                          |   1 +
 config/hatfield.defaults.yaml                      |  35 ++
 config/services.yaml                               |  14 +
 depfile.yaml                                       |   1 +
 docs/agents.md                                     | 156 ++++++++
 docs/settings.md                                   |  33 ++
 .../Agent/Definition/AgentDefinitionCatalog.php    | 130 +++++++
 .../Definition/AgentDefinitionDiagnosticDTO.php    |  36 ++
 .../Agent/Definition/AgentDefinitionDiscovery.php  | 280 ++++++++++++++
 src/CodingAgent/Config/AgentsConfig.php            |  65 ++++
 src/CodingAgent/Config/AppConfig.php               |   3 +
 src/CodingAgent/Config/AppConfigLoader.php         |   1 +
 .../Definition/AgentDefinitionCatalogTest.php      | 239 ++++++++++++
 .../Definition/AgentDefinitionDiscoveryTest.php    | 402 +++++++++++++++++++++
 tests/CodingAgent/Config/AgentsConfigTest.php      | 137 +++++++
 16 files changed, 1546 insertions(+)
 create mode 100644 docs/agents.md
 create mode 100644 src/CodingAgent/Agent/Definition/AgentDefinitionCatalog.php
 create mode 100644 src/CodingAgent/Agent/Definition/AgentDefinitionDiagnosticDTO.php
 create mode 100644 src/CodingAgent/Agent/Definition/AgentDefinitionDiscovery.php
 create mode 100644 src/CodingAgent/Config/AgentsConfig.php
 create mode 100644 tests/CodingAgent/Agent/Definition/AgentDefinitionCatalogTest.php
 create mode 100644 tests/CodingAgent/Agent/Definition/AgentDefinitionDiscoveryTest.php
 create mode 100644 tests/CodingAgent/Config/AgentsConfigTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/agent-02-definition-catalog-settings-docs.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/agent-02-definition-catalog-settings-docs.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Pre-DONE task board check: `agent-02-definition-catalog-settings-docs` was in CODE-REVIEW with PR https://github.com/ineersa/agent-core/pull/191.
- Summary: User confirmed PR #191 merged and requested task DONE. Moving CODE-REVIEW → DONE to merge/sync integration checkout and clean up task worktree.

## Task workflow update - 2026-06-22T01:49:27.371Z
- Validation: `LLM_MODE=true castor check` passed on integration checkout: quality ok (130.0s).; Castor check lanes: deptrac OK (0.9s); test OK (3190 tests, 10155 assertions, 19.1s); test:controller-replay OK (3 tests, 41 assertions, 26.9s); test:tui OK (14 tests, 124 assertions, 70.7s); phpstan OK (0 errors/file_errors, 10.4s); cs-check OK (2.0s).; Worktree removed: `/home/ineersa/projects/agent-core-worktrees/agent-02-definition-catalog-settings-docs` no longer exists.; Task board confirms `agent-02-definition-catalog-settings-docs` is in DONE.
- Summary: Post-DONE validation completed on integration checkout after merging PR #191. Worktree cleanup verified. Integration checkout is clean (`git status --short --branch` shows `main...origin/main [ahead 4]` with no modified/untracked files).
