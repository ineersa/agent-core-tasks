# MCP-01 Config loader and SDK boundary

## Goal
Implement phase 0 of `.pi/plans/mcp-client-implementation-plan.md`.

Goal: introduce the MCP client dependency boundary and configuration loading without wiring runtime execution yet.

Scope:
- Add/use `mcp/sdk` behind Hatfield-owned adapter interfaces/classes only.
- Add config loading for `~/.hatfield/mcp.json` and project `.hatfield/mcp.json`.
- Support `mcpServers` definitions for STDIO (`command`, `args`, `env`, `cwd`) and HTTP (`url`, `headers`).
- Support `enabled`, `timeoutMs`, `startupTimeoutMs`, `excludeTools`.
- Merge global < project definitions.
- Validate exactly one of `command` or `url` unless a project override only disables an inherited server.
- Interpolate env vars in `env` and `headers`, failing clearly if missing.
- Keep SDK types isolated under `src/CodingAgent/Mcp/Client` or equivalent; do not leak SDK types into AgentCore, TUI, or ExtensionApi.

Non-goals:
- No broker consumer yet.
- No dynamic tool registration yet.
- No MCP tool invocation yet.
- No OAuth.

## Acceptance criteria
- Config tests cover empty config, global config, project override, disabling inherited server, invalid command+url, invalid missing transport, env/header interpolation, and missing env var failure.
- A typed MCP server definition model exists for STDIO and HTTP servers.
- SDK usage is wrapped behind Hatfield-owned interfaces/adapters and does not leak into AgentCore, Tui, or ExtensionApi.
- Documentation or inline config examples reference `.pi/plans/mcp-client-implementation-plan.md` as the design source.
- Relevant validation is run through Castor, not raw vendor/bin commands.

## Workflow metadata
Status: DONE
Branch: task/mcp-01-config-sdk-boundary
Worktree: /home/ineersa/projects/agent-core-worktrees/mcp-01-config-sdk-boundary
Fork run: bukrcqnvnz4m
PR URL: https://github.com/ineersa/agent-core/pull/168
PR Status: merged
Started: 2026-06-18T19:46:40.267Z
Completed: 2026-06-18T20:27:21.696Z

## Work log
- Created: 2026-06-12T18:06:16.421Z

## Task workflow update - 2026-06-18T19:33:40.159Z
- Validation: Planning only; no QA commands run.; Suggested implementation validation: castor test --filter=Mcp; castor test; castor deptrac; castor phpstan; castor cs-check; castor check before CODE-REVIEW.
- Summary: Task-explain plan recorded: implement Phase 0 by adding mcp/sdk behind a Hatfield-owned client boundary; create src/CodingAgent/Mcp/Config typed DTOs, loader, validator, and env/header interpolator for ~/.hatfield/mcp.json plus project .hatfield/mcp.json; merge global < project by server name; validate STDIO vs HTTP transport definitions and disabled overrides; enforce SDK isolation via deptrac; add focused config tests using TestDirectoryIsolation; add docs/examples referencing .pi/plans/mcp-client-implementation-plan.md. No runtime broker, dynamic registration, or tool invocation in this phase.
- Task-explain open questions to resolve before implementation: (1) server override semantics: whole-server replace vs deep merge, (2) behavior of enabled:false for a non-inherited server, (3) whether empty env vars are treated as missing, (4) MCP-01 adapter scope: real SDK adapter vs dependency/boundary skeleton, (5) mcp/sdk version constraint: ^0.6 vs 0.6.*.

## Task workflow update - 2026-06-18T19:38:40.699Z
- Summary: Open-question decisions added to the plan: whole-server project overrides; inherited disable-only override special case; empty interpolated env/header variables fail; include a thin real SDK boundary only; use latest compatible mcp/sdk constraint while keeping SDK types isolated.
- Task-explain decisions resolved with user: (1) project mcpServers override uses whole-server replacement, with a special disable-only override for inherited servers; no deep merge within server definitions. (2) {enabled:false} without a transport is valid only when disabling an inherited server; non-inherited disable-only entries should fail clearly, while complete disabled local definitions remain valid. (3) Env/header interpolation treats missing or empty interpolated variables as configuration errors; literal empty strings remain allowed. (4) MCP-01 should include a thin real McpSdkClientAdapter/McpSdkClientFactory boundary that compiles against the installed SDK, but must not wire runtime connection, discovery, dynamic tools, or tool invocation. (5) Use the latest compatible mcp/sdk Composer constraint rather than an exact pin, with SDK imports isolated to the Hatfield MCP client boundary.

## Task workflow update - 2026-06-18T19:46:40.267Z
- Moved TODO → IN-PROGRESS.
- Created branch task/mcp-01-config-sdk-boundary.
- Created worktree /home/ineersa/projects/agent-core-worktrees/mcp-01-config-sdk-boundary.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/mcp-01-config-sdk-boundary.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/mcp-01-config-sdk-boundary.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/mcp-01-config-sdk-boundary.
- Summary: Starting implementation for MCP-01 after task-explain decisions were confirmed by the user. Scope remains Phase 0 only: config loader, DTOs, validation/interpolation, SDK boundary, tests, docs, and Castor validation; no runtime broker/tool wiring.

## Task workflow update - 2026-06-18T19:47:28.370Z
- Recorded fork run: 6uxrwxtxq15t
- Launched implementation fork 6uxrwxtxq15t in worktree /home/ineersa/projects/agent-core-worktrees/mcp-01-config-sdk-boundary with instructions covering task scope, resolved decisions, mandatory testing docs, SDK boundary, config loader/validation/interpolation, docs, tests, Castor validation, and commit handoff requirements.

## Task workflow update - 2026-06-18T19:58:00.891Z
- Recorded fork run: 6uxrwxtxq15t
- Validation: Fork confirmed it read the testing skill and tests/AGENTS.md and used TestDirectoryIsolation/Castor conventions.; castor test --filter=McpConfigLoader: 20 tests, 54 assertions — OK.; castor test: 2726 tests, 8123 assertions — OK.; castor deptrac: 0 violations, 0 errors.; castor phpstan: 0 errors.; castor cs-check: clean.; Not run by fork: castor check, castor test:tui, castor test:llm-real; MCP-01 does not touch runtime/TUI/controller/LLM-visible flow. Per task workflow, parent stopped after implementation verification; task-to-pr is separate.
- Summary: Implementation fork completed Phase 0 in worktree /home/ineersa/projects/agent-core-worktrees/mcp-01-config-sdk-boundary. Commit 62a23c2baa1562aa3879d30021dc56b778b4dd86 adds mcp/sdk (^0.6.0), typed MCP config DTOs/loader/validator/interpolator, a Hatfield-owned SDK client boundary, deptrac SDK isolation layers, docs/mcp.md, and focused config tests. Scope stayed within config loading and SDK boundary; no broker, runtime/TUI wiring, dynamic tool registration, OAuth, or tool invocation integration. Parent verified git status is clean and diff stat contains the expected MCP config/client/docs/tests/composer/deptrac changes.

## Task workflow update - 2026-06-18T20:05:20.065Z
- Summary: task-to-pr reviewer pass returned APPROVE WITH SUGGESTIONS, not final approval. Actionable findings to address before PR: remove dead/no-op requireStringList block; preserve callTool isError in adapter/interface return shape; avoid silent loss for audio/resource/unknown MCP content types; remove vendor exception @throws leak from McpClientInterface by using Hatfield-owned exception; avoid misleading default STDIO transport type on direct DTO construction; simplify disabled-server filtering; make JSON read missing-file handling clearer; clarify duplicate DTO enabled validation as standalone safety; replace hardcoded client info literals with constants/comment. Launching fix fork before re-review.

## Task workflow update - 2026-06-18T20:05:50.445Z
- Recorded fork run: 6myurtichiuz
- Launched review-fix fork 6myurtichiuz in worktree /home/ineersa/projects/agent-core-worktrees/mcp-01-config-sdk-boundary to address all actionable reviewer suggestions before re-review/code-review move. Required fixes cover adapter result mapping/isError, Hatfield-owned connection exception, DTO transport default cleanup, dead-code removal, loader simplification, factory client-info constants, focused tests, and Castor validation.

## Task workflow update - 2026-06-18T20:11:03.324Z
- Recorded fork run: 6myurtichiuz
- Validation: Review-fix fork confirmed it read AGENTS.md, testing skill, and tests/AGENTS.md.; castor test --filter=Mcp: 29 tests, 89 assertions — OK.; castor test: 2735 tests, 8158 assertions — OK.; castor deptrac: 0 violations, 0 errors.; castor phpstan: 0 errors, 0 file_errors.; castor cs-check: clean.; Not run: castor check, castor test:tui, castor test:llm-real — no runtime/TUI/controller changes.
- Summary: Review-fix fork completed. Commit 87f31f141 fixes all actionable reviewer suggestions while staying in MCP-01 Phase 0 scope: removed dead DTO list-validation block, made DTO transport type nullable by default, added Hatfield-owned McpClientConnectionException and wraps SDK connection failures, preserves callTool isError, maps text/image/audio/resource content to native arrays and throws on unknown content, simplified disabled-server filtering and JSON loading, replaced factory client-info literals with constants, and added focused McpSdkClientAdapter tests. Parent verified worktree is clean and HEAD includes commits 62a23c2ba and 87f31f141 with expected MCP diff stat.

## Task workflow update - 2026-06-18T20:14:58.347Z
- Summary: Second reviewer pass returned APPROVED for current HEAD. Reviewer verified the prior fix commit addressed all earlier findings and MCP-01 requirements still hold. It noted one sensible edge-case polish item and two Phase-3 mapping TODO gaps: present-but-wrong-type mcpServers currently loads empty instead of failing; adapter intentionally omits structuredContent/meta and resource annotations. Launching a final small polish fork before focused validation/PR to address the edge case and document deferred mapping gaps, even though the reviewer verdict is APPROVED.

## Task workflow update - 2026-06-18T20:18:18.521Z
- Recorded fork run: bukrcqnvnz4m
- Validation: Final polish fork confirmed it read AGENTS.md, testing skill, and tests/AGENTS.md.; castor test --filter=Mcp: 30 tests, 92 assertions — OK.; castor test: 2736 tests, 8161 assertions — OK.; castor deptrac: 0 violations, 0 errors.; castor phpstan: 0 errors, 0 file_errors.; castor cs-check: clean after castor cs-fix --path=src/CodingAgent/Mcp/Config/McpConfigLoader.php.; Not run: castor check, castor test:tui, castor test:llm-real, castor test:controller — no runtime/TUI/controller/LLM-live changes.
- Summary: Final polish fork completed. Commit bb5d00da5 adds explicit wrong-type mcpServers rejection via extractServers(), a focused config-loader test for scalar mcpServers, and Phase-3 TODO comments documenting deferred adapter mapping for structuredContent/meta and resource annotations. Scope stayed within MCP-01 Phase 0; no runtime/broker/TUI/controller wiring. Fork reported clean worktree and all requested Castor validation passing.

## Task workflow update - 2026-06-18T20:21:40.709Z
- Validation: Final reviewer verdict: APPROVED for current HEAD bb5d00da5.; Parent focused validation: castor test — OK (2736 tests, 8161 assertions).; Parent focused validation: castor deptrac — 0 violations, 0 errors.; Parent focused validation: castor phpstan — 0 errors, 0 file_errors.; Parent focused validation: castor cs-check — files_fixed=0.; Pre-CODE-REVIEW stale process check: no matching worktree controller/consumer/test/castor processes except the inspection command itself.
- Summary: Final reviewer pass on current HEAD bb5d00da5 returned APPROVED. Reviewer verified MCP-01 requirements and final scalar mcpServers polish. Non-blocking nits were duplicate assertion and typo in a test method name; not blocking CODE-REVIEW. Parent ran focused local Castor validation in worktree after approval and confirmed git status clean.

## Task workflow update - 2026-06-18T20:22:33.182Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (35.6s).
- Pushed task/mcp-01-config-sdk-boundary to origin.
- branch 'task/mcp-01-config-sdk-boundary' set up to track 'origin/task/mcp-01-config-sdk-boundary'.
- Created PR: https://github.com/ineersa/agent-core/pull/168
- Validation: Reviewer subagent final verdict: APPROVED for HEAD bb5d00da5.; Parent focused validation before CODE-REVIEW: castor test — OK (2736 tests, 8161 assertions).; Parent focused validation before CODE-REVIEW: castor deptrac — 0 violations, 0 errors.; Parent focused validation before CODE-REVIEW: castor phpstan — 0 errors, 0 file_errors.; Parent focused validation before CODE-REVIEW: castor cs-check — files_fixed=0.; move_task CODE-REVIEW will run deterministic castor check before pushing/PR creation.
- Summary: MCP-01 Phase 0 implementation is approved and ready for PR. Current HEAD bb5d00da5 includes mcp/sdk dependency, typed MCP config loader/validator/interpolator, Hatfield-owned SDK client boundary, deptrac SDK isolation, docs/mcp.md, config/adapter tests, reviewer fixes, and final scalar mcpServers validation polish. No runtime broker/TUI/controller/dynamic tool wiring was added.

## Task workflow update - 2026-06-18T20:27:21.696Z
- Moved CODE-REVIEW → DONE.
- Merged task/mcp-01-config-sdk-boundary into integration checkout.
- Merge made by the 'ort' strategy.
 composer.json                                      |   3 +-
 composer.lock                                      | 468 ++++++++++++++++-
 depfile.yaml                                       |  21 +
 docs/mcp.md                                        | 164 ++++++
 phpstan-baseline.neon                              |  36 +-
 .../Mcp/Client/McpClientConnectionException.php    |  23 +
 src/CodingAgent/Mcp/Client/McpClientInterface.php  |  59 +++
 src/CodingAgent/Mcp/Client/McpSdkClientAdapter.php | 180 +++++++
 src/CodingAgent/Mcp/Client/McpSdkClientFactory.php |  86 ++++
 src/CodingAgent/Mcp/Config/McpConfigDTO.php        |  32 ++
 src/CodingAgent/Mcp/Config/McpConfigLoader.php     | 175 +++++++
 src/CodingAgent/Mcp/Config/McpConfigValidator.php  | 128 +++++
 src/CodingAgent/Mcp/Config/McpEnvInterpolator.php  |  84 ++++
 .../Mcp/Config/McpServerDefinitionDTO.php          | 210 ++++++++
 .../Mcp/Config/McpTransportTypeEnum.php            |  17 +
 .../Mcp/Client/McpSdkClientAdapterTest.php         | 195 ++++++++
 .../CodingAgent/Mcp/Config/McpConfigLoaderTest.php | 553 +++++++++++++++++++++
 17 files changed, 2420 insertions(+), 14 deletions(-)
 create mode 100644 docs/mcp.md
 create mode 100644 src/CodingAgent/Mcp/Client/McpClientConnectionException.php
 create mode 100644 src/CodingAgent/Mcp/Client/McpClientInterface.php
 create mode 100644 src/CodingAgent/Mcp/Client/McpSdkClientAdapter.php
 create mode 100644 src/CodingAgent/Mcp/Client/McpSdkClientFactory.php
 create mode 100644 src/CodingAgent/Mcp/Config/McpConfigDTO.php
 create mode 100644 src/CodingAgent/Mcp/Config/McpConfigLoader.php
 create mode 100644 src/CodingAgent/Mcp/Config/McpConfigValidator.php
 create mode 100644 src/CodingAgent/Mcp/Config/McpEnvInterpolator.php
 create mode 100644 src/CodingAgent/Mcp/Config/McpServerDefinitionDTO.php
 create mode 100644 src/CodingAgent/Mcp/Config/McpTransportTypeEnum.php
 create mode 100644 tests/CodingAgent/Mcp/Client/McpSdkClientAdapterTest.php
 create mode 100644 tests/CodingAgent/Mcp/Config/McpConfigLoaderTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/mcp-01-config-sdk-boundary.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/mcp-01-config-sdk-boundary.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User reported PR #168 was merged. Moving MCP-01 to DONE and merging/syncing integration checkout via task workflow.

## Task workflow update - 2026-06-18T20:30:34.177Z
- Updated PR Status: merged
- Validation: move_task DONE: merged task/mcp-01-config-sdk-boundary into integration checkout and removed worktree /home/ineersa/projects/agent-core-worktrees/mcp-01-config-sdk-boundary.; Initial post-merge LLM_MODE=true castor check: FAILED in test/phpstan because Mcp\* classes were missing from integration vendor after composer.lock merge.; Environment sync: composer install --no-interaction installed php-http/discovery, psr/http-server-handler, psr/http-server-middleware, opis/string, opis/uri, opis/json-schema, and mcp/sdk v0.6.0 from lockfile.; Post-sync stale process scan: no stale worktree/integration controller, messenger, PHPUnit, ParaTest, or Castor processes found.; Post-merge validation: LLM_MODE=true castor check — OK (quality: ok, 112.7s). Steps: deptrac OK; test OK (2740 tests, 8173 assertions); test:controller-replay OK (3 tests, 41 assertions); test:tui OK (9 tests, 91 assertions); phpstan OK (0 errors); cs-check OK.; Final git status: clean working tree on main; main is ahead of origin/main by 4 commits after local task workflow merge.
- Summary: DONE transition completed after user reported PR #168 was merged. move_task merged task/mcp-01-config-sdk-boundary into the integration checkout, removed the task worktree and IDEA exclusions, and pulled/synced the integration checkout. Post-merge validation initially failed because the integration checkout vendor directory did not yet contain the new lockfile dependency mcp/sdk; ran composer install --no-interaction to sync vendor with composer.lock, then reran the full deterministic Castor gate successfully. Integration checkout is clean, with local main ahead of origin/main by the merge commit/branch commits after the task workflow merge.
