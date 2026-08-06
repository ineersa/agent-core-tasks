# AGENT-01 Agent definition parser and validation

## Goal
Create the production-ready, standalone agent definition parsing layer. This is intentionally limited to reading/parsing/validating one agent definition file and typed DTOs; do not implement launching, hidden runs, artifacts, TUI, MCP execution, or compatibility/fallback paths.

Context:
- Reference plan: `.pi/plans/agents-subagents-implementation-plan.md`.
- Reuse patterns from existing settings/skills/frontmatter parsing where appropriate.
- Supported definition file shape is Markdown with YAML frontmatter and body instructions.
- `.agents/` support is first-class; discovery/catalog precedence is handled by AGENT-02, not this task.
- No backward-compatibility shims or dual-format readers unless explicitly requested.

Suggested scope:
- `AgentDefinitionDTO` and related typed value objects/enums.
- `AgentDefinitionParser` / `AgentFrontmatterParser` for a single file.
- Validation errors with actionable file/path/field messages.
- Fields should cover at least: `name`, `description`, `type`, `tools`, `mcpTools`/MCP access shape, `model`, `thinking`, `skills`, inheritance flags, `maxDepth`, `backgroundAllowed`, `foregroundAllowed`, `parallelAllowed`, `disabled`, and body prompt/instructions.
- Keep implementation in the app layer; do not add AgentCore or TUI dependencies.

## Acceptance criteria
- A single agent definition Markdown file can be parsed into typed DTOs with body prompt preserved.
- Invalid/missing frontmatter fields produce actionable validation errors including file path and field name.
- Unit/integration coverage exercises valid definitions and representative invalid definitions without testing trivial getters only.
- No launch/runtime/TUI/MCP execution behavior is added.
- No fallback/compatibility/legacy dual-format code is added.
- Docs or task notes identify the exact frontmatter schema AGENT-02 should consume.
- Validation uses Castor commands only, with at least focused `castor test --filter=...`, plus `castor phpstan`/`castor deptrac` if relevant.

## Workflow metadata
Status: DONE
Branch: task/agent-01-definition-parser-validation
Worktree: /home/ineersa/projects/agent-core-worktrees/agent-01-definition-parser-validation
Fork run: bsp2h9b286zw
PR URL: https://github.com/ineersa/agent-core/pull/187
PR Status: merged
Started: 2026-06-21T20:12:05.194Z
Completed: 2026-06-21T23:16:51.066Z

## Work log
- Created: 2026-06-15T22:52:05.370Z

## Task workflow update - 2026-06-21T20:12:05.195Z
- Moved TODO → IN-PROGRESS.
- Created branch task/agent-01-definition-parser-validation.
- Created worktree /home/ineersa/projects/agent-core-worktrees/agent-01-definition-parser-validation.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/agent-01-definition-parser-validation.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/agent-01-definition-parser-validation.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/agent-01-definition-parser-validation.
- Summary: Starting task-start orchestration. Loaded task-workflow and testing skills, read tests/AGENTS.md, and reviewed referenced agents/subagents implementation plan before implementation dispatch.

## Task workflow update - 2026-06-21T20:17:14.657Z
- Orchestrator context: worktree `/home/ineersa/projects/agent-core-worktrees/agent-01-definition-parser-validation`. Scouts found relevant patterns in `PromptTemplateFrontmatterParser`, `McpConfigValidator`, `McpServerDefinitionDTO`, `SkillDefinition`, and tests using `TestDirectoryIsolation`. AGENT-01 scope is intentionally single-file parsing/validation only; do not implement catalog/discovery/launch/TUI/MCP execution.
- Schema for AGENT-02 to consume: Markdown file with required YAML frontmatter fields `name`, `description`, `type`, `tools` and body prompt/instructions. Optional fields: `mcp` object (`mode: none|specific|all`, `tools` required non-empty only when `specific`), `model`, `thinking`, `skills`, `inheritProjectContext`, `inheritAgentsMd`, `systemPromptMode`, `maxDepth`, `backgroundAllowed`, `foregroundAllowed`, `parallelAllowed`, `disabled`, `handoffFormat`. Strict validation should reject unknown keys and include file path + field in errors. Defaults: `mcp.mode=none`, `skills=[]`, `inheritProjectContext=true`, `inheritAgentsMd=true`, `systemPromptMode=replace`, `maxDepth=1`, `backgroundAllowed=true`, `foregroundAllowed=true`, `parallelAllowed=false`, `disabled=false`, nullable optionals default null.

## Task workflow update - 2026-06-21T20:23:51.494Z
- Recorded fork run: bsp2h9b286zw
- Validation: Fork reported `castor test --filter=AgentDefinitionParserTest` passed: 48 tests / 127 assertions.; Fork reported `castor phpstan --path=src/CodingAgent/Agent/Definition` passed: 0 errors.; Fork reported `castor phpstan --path=tests/CodingAgent/Agent/Definition` passed: 0 errors.; Fork reported `castor deptrac` passed: 0 violations / 0 errors.; Fork reported `castor cs-check` passed: 0 fixable issues.; Parent verification: `git -C /home/ineersa/projects/agent-core-worktrees/agent-01-definition-parser-validation log --oneline -1` => `42f236c67 Implement agent definition parser and validation`.; Parent verification: `git diff --stat origin/main...HEAD` => 10 files changed, 1671 insertions.
- Summary: Implementation fork completed and committed `42f236c67` (`Implement agent definition parser and validation`) on branch `task/agent-01-definition-parser-validation`. Verified worktree status is clean, commit exists, and diff stat contains expected AGENT-01 files: new `src/CodingAgent/Agent/Definition/` DTO/parser/validator/enums/MCP policy classes, new focused parser tests under `tests/CodingAgent/Agent/Definition/`, and `depfile.yaml` AppAgent layer. Scope remained single-file parsing/validation only; no catalog/discovery/launch/TUI/runtime/MCP execution behavior added. Fork confirmed it read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before test work.

## Task workflow update - 2026-06-21T21:27:02.703Z
- Summary: Reviewer subagent returned REQUEST CHANGES for current HEAD `42f236c67`. Blocking findings: `validateMcp()` silently drops `mcp.tools` when `mcp.mode` is omitted; `mcp.tools` and `skills` accept empty-string entries. Additional sensible findings to address before re-review: stricter frontmatter closing delimiter check, BOM stripping, explicit `mcp: null` consistency, test namespace convention, DTO docblock order, ineffective inline phpstan comments, and a parseFile real-file test.
- Reviewer decision for `42f236c67`: REQUEST CHANGES. Preparing fork fix instructions for all sensible actionable findings; main orchestrator will not edit files directly.

## Task workflow update - 2026-06-21T21:37:14.091Z
- Validation: Reviewer subagent decision on `cb1a2b211`: APPROVED (non-blocking notes only).; Focused validation: `castor test` passed: OK (3099 tests, 9697 assertions).; Focused validation: `castor deptrac` passed: violations=0, errors=0.; Focused validation: `castor phpstan` passed: errors=0, file_errors=0.; Focused validation: `castor cs-check` passed: files_fixed=0 / no fixable issues.; Parent verification: `git status --short --branch` shows clean branch `task/agent-01-definition-parser-validation`.
- Summary: Re-reviewer approved current HEAD `cb1a2b211` after fix commit. Fix fork addressed all prior actionable findings: MCP tools without mode now errors, empty/whitespace list entries are rejected consistently, BOM and closing delimiter edge cases handled, explicit `mcp: null` normalized, test namespace fixed, DTO docblock/comments cleaned, and `parseFile()` coverage added. Worktree is clean.

## Task workflow update - 2026-06-21T21:39:55.940Z
- Validation: First CODE-REVIEW transition attempt ran deterministic `castor check` and failed only in `test:tui` with exit code 143 / log `Terminated`. Diagnosed stale user-owned TUI/controller/messenger processes from this worktree and terminated only those current-user processes.; After cleanup, focused `castor test:tui` passed: OK (11 tests, 107 assertions). Worktree remained clean.

## Task workflow update - 2026-06-21T21:41:02.999Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (56.5s).
- Pushed task/agent-01-definition-parser-validation to origin.
- branch 'task/agent-01-definition-parser-validation' set up to track 'origin/task/agent-01-definition-parser-validation'.
- Created PR: https://github.com/ineersa/agent-core/pull/187

## Task workflow update - 2026-06-21T21:41:09.069Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/187
- Updated PR Status: open
- Validation: CODE-REVIEW gate: `castor check` passed in worktree (56.5s).; Pushed branch `task/agent-01-definition-parser-validation` to origin.; Created PR https://github.com/ineersa/agent-core/pull/187.
- Summary: Moved to CODE-REVIEW. Deterministic `castor check` passed during transition (56.5s), branch `task/agent-01-definition-parser-validation` pushed to origin, and PR created: https://github.com/ineersa/agent-core/pull/187.

## Task workflow update - 2026-06-21T21:51:51.112Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Starting task-review-iterate for PR #187 feedback. Read task metadata and all PR comments/inline review comments via gh. Actionable feedback: remove `type`/`AgentTypeEnum`; DRY/reuse existing frontmatter parsing instead of custom parser where possible; replace manual schema validation/DTO assembly with Symfony Serializer denormalization and Symfony Validator attribute constraints on DTOs. Moving back to IN-PROGRESS before implementation fork per workflow.

## Task workflow update - 2026-06-21T21:56:42.175Z
- Summary: Classified PR #187 feedback from three inline comments. All are actionable: (1) remove `type` / `AgentTypeEnum` because planned agent roles are not definition types; (2) DRY duplicated markdown/YAML frontmatter extraction with existing prompt template/skill parsing paths; (3) replace the large manual validation/DTO construction in `AgentDefinitionParser` with Symfony Serializer denormalization and Symfony Validator attribute constraints/callbacks on DTOs. Scout subagents confirmed relevant existing patterns: `SkillDiscovery::parseFrontmatter`, `PromptTemplateFrontmatterParser`, `AppConfig::fromContainer` serializer usage, framework serializer+validator enabled, existing `SymfonySerializer` deptrac layer but no `SymfonyValidator` layer yet.
- PR comments read via `gh pr view 187 --comments` and `gh api repos/ineersa/agent-core/pulls/187/comments --paginate`; no external docs were referenced by comments. Branch slug confirmed from PR headRefName: `task/agent-01-definition-parser-validation`.

## Task workflow update - 2026-06-21T22:15:24.691Z
- Summary: First review-iteration fork committed `3d2f2dc15` and addressed broad PR comments, but parent verification found follow-up blockers before re-review/CODE-REVIEW: serializer type enforcement was explicitly disabled and handoff admitted scalar coercion (`'yes'`→bool, `'3'`→int), which violates strict schema/no-coercion intent; tools/skills/mcp.tools whitespace-wrapped entries are now trimmed instead of rejected; cross-field invariants remain parser-side instead of Validator callbacks; shared extractor still accepts closing delimiter lines with trailing junk after a space; unknown fields are still manually checked instead of using Serializer extra-attribute handling. Launching a second fork to tighten these before reviewer re-review.

## Task workflow update - 2026-06-21T22:23:49.345Z
- Summary: Parent inspection of second fork commit `67876f5c7` found two additional small correctness gaps before accepting the iteration: `AgentDefinitionParser` TypeError field extraction regex has no capture group but reads `$matches[1]`, and strict `list<string>` semantics are not enforced for `tools`, `skills`, or `mcp.tools` (YAML mappings with string values would be accepted and `array_values()`-normalized). Launching a small follow-up fork to fix via Validator callbacks/tests.

## Task workflow update - 2026-06-21T22:41:30.190Z
- Validation: Reviewer subagent decision on `7162065a9`: REQUEST CHANGES (PHPStan only).; Parent reproduced: `castor phpstan` failed with file_errors=6: ternary.shortNotAllowed, nullsafe.neverNull x2, missingType.iterableValue x3.
- Summary: Reviewer subagent on HEAD `7162065a9` returned REQUEST CHANGES due only to full `castor phpstan` failures (6 file errors). Parent reproduced with full `castor phpstan`: short ternary in `AgentDefinitionParser::validateFrontmatterDto`, nullsafe+coalesce in `mapToDefinition`, and missing `list<string>` phpdoc for array constructor params in `AgentFrontmatterDTO` and `McpFrontmatterDTO`. Logic/review feedback otherwise approved. Launching small fork to fix PHPStan issues.

## Task workflow update - 2026-06-21T22:50:03.633Z
- Validation: Focused: `castor test --filter=AgentDefinitionParserTest` passed: OK (74 tests, 175 assertions).; Focused: `castor test --filter=PromptTemplateFrontmatterParserTest` passed: OK (11 tests, 33 assertions).; Focused: `castor test --filter=SkillDiscoveryTest` passed: OK (17 tests, 44 assertions).; Full unit/integration: `castor test` passed: OK (3112 tests, 9717 assertions).; Full static analysis: `castor phpstan` passed: errors=0, file_errors=0.; Architecture: `castor deptrac` passed: violations=0, errors=0.; Coding style: `castor cs-check` passed: files_fixed=0 / no fixable issues.; Reviewer subagent decision on `fe43600a0`: APPROVED (non-blocking notes only).; Parent verification: worktree clean, branch ahead of origin by 4 commits.
- Summary: Review iteration completed at HEAD `fe43600a0` after four follow-up commits. PR feedback addressed: `type` / `AgentTypeEnum` removed and `type` is now rejected as unknown; shared `MarkdownFrontmatterExtractor` DRYs agent/prompt-template/skill frontmatter extraction; agent frontmatter denormalizes through Symfony Serializer with strict type/extra-attribute handling and validates through Symfony Validator attributes/callbacks; strict list shape/whitespace/no-coercion/cross-field/delimiter behavior covered by tests. Reviewer subagent re-review approved `fe43600a0` with only non-blocking notes.

## Task workflow update - 2026-06-21T22:51:21.974Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (59.4s).
- Pushed task/agent-01-definition-parser-validation to origin.
- branch 'task/agent-01-definition-parser-validation' set up to track 'origin/task/agent-01-definition-parser-validation'.
- PR already exists: https://github.com/ineersa/agent-core/pull/187

## Task workflow update - 2026-06-21T22:51:27.078Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/187
- Updated PR Status: open
- Validation: CODE-REVIEW gate: `castor check` passed in worktree (59.4s).; Pushed branch `task/agent-01-definition-parser-validation` to origin.; Existing PR updated: https://github.com/ineersa/agent-core/pull/187.
- Summary: Moved review iteration back to CODE-REVIEW. Deterministic `castor check` passed during transition (59.4s), branch `task/agent-01-definition-parser-validation` pushed to origin with commits through `fe43600a0`, and existing PR remains https://github.com/ineersa/agent-core/pull/187.

## Task workflow update - 2026-06-21T23:03:00.475Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #187 has one remaining inline review comment at `src/CodingAgent/Skills/SkillDiscovery.php`: "Why not autowired?" referring to the lazy `createExtractor()` / manual `new MarkdownFrontmatterExtractor()` helper. User requested a small fix and explicitly asked not to relaunch reviewer afterward. Moving back to IN-PROGRESS for a narrow review-iteration fix.

## Task workflow update - 2026-06-21T23:06:50.605Z
- Summary: User rejected prior small fix because `SkillDiscovery::stripFrontmatter()` still manually instantiates `MarkdownFrontmatterExtractor`; review comment intent is no production manual `new` for this shared extractor. Launching follow-up fork to remove the static helper/manual instantiation entirely by injecting the extractor through constructors (e.g. `SkillsContextBuilder` → `SkillRegistry`) and updating tests. No reviewer relaunch per user instruction.

## Task workflow update - 2026-06-21T23:10:56.817Z
- Validation: Parent verified commit: `170cd3233 Inject skill frontmatter extractor everywhere`.; Parent grep proof: `rg "new MarkdownFrontmatterExtractor" src/CodingAgent/Skills` returned no matches.; `castor test --filter=SkillDiscoveryTest` passed: OK (14 tests, 38 assertions).; `castor test --filter=SkillRegistryTest` passed: OK (9 tests, 16 assertions).; `castor test --filter=SkillsContextBuilderTest` passed: OK (6 tests, 17 assertions).; `castor phpstan` passed: errors=0, file_errors=0.; `castor deptrac` passed: violations=0, errors=0.; `castor cs-check` passed: files_fixed=0 / no fixable issues.
- Summary: Follow-up fix completed for remaining PR #187 comment. Commit `170cd3233` removes the remaining production `new MarkdownFrontmatterExtractor()` in `src/CodingAgent/Skills`: deleted static `SkillDiscovery::stripFrontmatter()`, injected `MarkdownFrontmatterExtractor` into `SkillRegistry`, injected it into `SkillsContextBuilder`, and passed it through when constructing the registry. Parent verified `rg "new MarkdownFrontmatterExtractor" src/CodingAgent/Skills` returns no matches. Per user instruction, no reviewer subagent was relaunched.

## Task workflow update - 2026-06-21T23:12:01.438Z
- Validation: Failed CODE-REVIEW gate: `castor check` failed with `test:tui` exit code 1; log showed `CancelStickinessE2eTest::testCancellingDoesNotRevertToWorking` failure.; Stale-process check before rerun: no current-user `messenger:consume`, `agent --controller`, phpunit/paratest/castor, or `bin/console agent` processes found.; Focused rerun: `castor test:tui --filter=CancelStickinessE2eTest` passed: OK (1 test, 2 assertions).
- Summary: Attempted CODE-REVIEW transition after final autowiring fix, but deterministic `castor check` failed only in TUI lane on `CancelStickinessE2eTest::testCancellingDoesNotRevertToWorking` (cancel text did not render). This area is unrelated to the skills/frontmatter DI change. Checked for stale current-user worker/castor/phpunit/controller processes before rerun; none found besides the grep process. Focused TUI rerun passed.

## Task workflow update - 2026-06-21T23:13:11.702Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (58.8s).
- Pushed task/agent-01-definition-parser-validation to origin.
- branch 'task/agent-01-definition-parser-validation' set up to track 'origin/task/agent-01-definition-parser-validation'.
- PR already exists: https://github.com/ineersa/agent-core/pull/187

## Task workflow update - 2026-06-21T23:13:17.318Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/187
- Updated PR Status: open
- Validation: Final gate: `castor check` passed in worktree (58.8s).; Branch `task/agent-01-definition-parser-validation` pushed to origin with commit `170cd3233`.; Existing PR updated: https://github.com/ineersa/agent-core/pull/187.
- Summary: Final review-comment fix pushed to PR #187. Commit `170cd3233` removes all production manual `new MarkdownFrontmatterExtractor()` calls under `src/CodingAgent/Skills` by deleting `SkillDiscovery::stripFrontmatter()`, injecting the extractor into `SkillRegistry`, and injecting/passing it through `SkillsContextBuilder`. No reviewer subagent was relaunched per user instruction. CODE-REVIEW transition retry passed deterministic `castor check` and pushed the branch.

## Task workflow update - 2026-06-21T23:16:51.066Z
- Moved CODE-REVIEW → DONE.
- Merged task/agent-01-definition-parser-validation into integration checkout.
- Merge made by the 'ort' strategy.
 depfile.yaml                                       |   24 +
 .../Agent/Definition/AgentDefinitionDTO.php        |   45 +
 .../Agent/Definition/AgentDefinitionParser.php     |  214 ++++
 .../AgentDefinitionValidationException.php         |   18 +
 .../Agent/Definition/AgentFrontmatterDTO.php       |  147 +++
 .../Agent/Definition/AgentFrontmatterParser.php    |   68 ++
 .../Agent/Definition/McpAgentModeEnum.php          |   20 +
 .../Agent/Definition/McpFrontmatterDTO.php         |   50 +
 src/CodingAgent/Agent/Definition/McpPolicyDTO.php  |   24 +
 .../Agent/Definition/SystemPromptModeEnum.php      |   19 +
 .../Markdown/MarkdownFrontmatterExtractor.php      |  150 +++
 .../PromptTemplateFrontmatterParser.php            |   56 +-
 src/CodingAgent/Skills/SkillDiscovery.php          |   21 +-
 src/CodingAgent/Skills/SkillRegistry.php           |    6 +-
 src/CodingAgent/Skills/SkillsContextBuilder.php    |    4 +-
 .../Agent/Definition/AgentDefinitionParserTest.php | 1256 ++++++++++++++++++++
 .../PromptTemplateFrontmatterParserTest.php        |    3 +-
 .../PromptTemplate/PromptTemplateLoaderTest.php    |    3 +-
 .../PromptTemplate/PromptTemplateServiceTest.php   |   19 +-
 tests/CodingAgent/Skills/SkillDiscoveryTest.php    |   29 +-
 tests/CodingAgent/Skills/SkillRegistryTest.php     |   19 +-
 .../Skills/SkillsContextBuilderTest.php            |    3 +
 22 files changed, 2102 insertions(+), 96 deletions(-)
 create mode 100644 src/CodingAgent/Agent/Definition/AgentDefinitionDTO.php
 create mode 100644 src/CodingAgent/Agent/Definition/AgentDefinitionParser.php
 create mode 100644 src/CodingAgent/Agent/Definition/AgentDefinitionValidationException.php
 create mode 100644 src/CodingAgent/Agent/Definition/AgentFrontmatterDTO.php
 create mode 100644 src/CodingAgent/Agent/Definition/AgentFrontmatterParser.php
 create mode 100644 src/CodingAgent/Agent/Definition/McpAgentModeEnum.php
 create mode 100644 src/CodingAgent/Agent/Definition/McpFrontmatterDTO.php
 create mode 100644 src/CodingAgent/Agent/Definition/McpPolicyDTO.php
 create mode 100644 src/CodingAgent/Agent/Definition/SystemPromptModeEnum.php
 create mode 100644 src/CodingAgent/Markdown/MarkdownFrontmatterExtractor.php
 create mode 100644 tests/CodingAgent/Agent/Definition/AgentDefinitionParserTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/agent-01-definition-parser-validation.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/agent-01-definition-parser-validation.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User confirmed PR #187 was merged. Moving task to DONE and cleaning up the task worktree.

## Task workflow update - 2026-06-21T23:19:13.222Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/187
- Updated PR Status: merged
- Validation: DONE transition merged task branch into integration checkout and removed worktree.; Initial post-merge `LLM_MODE=true castor check` failed only in `test:tui` on `CancelStickinessE2eTest::testCancellingDoesNotRevertToWorking` (same transient cancel-render flake seen earlier); all other lanes passed: deptrac, test, controller replay, phpstan, cs-check.; Stale-process check before focused rerun: no current-user `messenger:consume`, `agent --controller`, phpunit/paratest/castor, or `bin/console agent` processes found.; Focused rerun `castor test:tui --filter=CancelStickinessE2eTest` passed: OK (1 test, 2 assertions).; Retry `LLM_MODE=true castor check` passed: quality ok (113.7s); deptrac OK, test OK (3145 tests, 10030 assertions), controller replay OK (3 tests, 41 assertions), TUI OK (13 tests, 115 assertions), phpstan OK (0 errors/file_errors), cs-check OK.; Final integration checkout status: clean; `main...origin/main [ahead 2]`.
- Summary: Task moved to DONE after user confirmed PR #187 was merged. Worktree `/home/ineersa/projects/agent-core-worktrees/agent-01-definition-parser-validation` removed and IDEA exclusions cleaned. Integration checkout post-merge validation completed. Note: integration checkout is clean but `main` is currently ahead of `origin/main` by 2 local merge commits produced by the DONE workflow after the GitHub PR merge (`58ad10089`, `a517ca61f`); no destructive cleanup was performed.
