# Trim OutputCap, prompt-template, and MCP config plumbing

## Goal
Verified ponytail audit of three bounded CodingAgent areas. Estimated net reduction: ~350 lines, no dependency changes.

Scope:

### OutputCap
- Remove production-unused/test-only `OutputCap::process()` and `OutputCap::config()`.
- Remove `capForPath()` and the processor's duplicate pre-check; call `capIfNeeded()` once.
- Centralize duplicated read-specific notice construction and offset extraction currently present in both `OutputCapToolResultProcessor` and `OutputCapLlmTransformHook`.
- Consolidate `OutputCapPathResolver` into the existing OutputCap implementation if this produces the smallest behavior-preserving diff.
- Retain both `OutputCapToolResultProcessor` (primary enforcement) and `OutputCapLlmTransformHook` (LLM-bound defense in depth).
- Reduce implementation-mirroring tests while retaining behavioral proof for capping, persistence, cleanup, path classification, detail sanitization, notification delivery, and primary/late-stage agreement.

### MCP Config + Catalog
- Fold the single-caller `McpConfigValidator` and `McpEnvInterpolator` into `McpConfigLoader` and/or `McpServerDefinitionDTO`, establishing one validation owner.
- Preserve all JSON trust-boundary checks, transport/inheritance semantics, environment interpolation, secret-safe errors, and exact merged configuration behavior.
- Inline the single-production-consumer `McpToolNameMapper` mapping/sanitization logic into `McpInitializeSessionHandler`; remove test-only `reverseKey()` and implementation-mirroring mapper tests.
- Remove forwarding `McpConfigDTO::fromServers()` and construct the DTO directly.
- Keep `McpToolCatalogStoreInterface`, catalog DTO serialization boundaries, and `SessionFileMcpToolCatalogStore`: they are justified by cross-process persistence and test seams.

### Prompt templates
- Replace `PromptTemplateArgumentParser::splitChars()` with installed/native `mb_str_split()` support and inline its one-line whitespace predicate.
- Keep the remaining PromptTemplate class boundaries; their parser, substitution, loading, diagnostics, and runtime contracts are independently meaningful.

Explicit exclusions:
- Do not modify MCP Client, transport, cancellation, or deadline behavior; `add-proper-mcp-tool-call-cancellation` owns that work.
- Do not modify Subagent or Fork production/test files or alter their cap classification.
- Do not alter user-visible output-cap notices, limits, persistence paths, retention, MCP configuration semantics, catalog file format, or prompt-template grammar.
- Add no APIs, abstractions, settings, compatibility paths, or dependencies.

## Acceptance criteria
- Production behavior and model-facing OutputCap notices remain byte-for-byte equivalent for covered scenarios; primary enforcement and late defense-in-depth remain active.
- OutputCap test-only APIs and duplicate cap/path/notice logic listed in scope are removed without weakening persistence, sanitization, notification, or path-classification contracts.
- MCP configuration has one validation/hydration flow while preserving unknown-field rejection, transport rules, inherited disable semantics, typed fields, interpolation, and secret redaction.
- `McpToolNameMapper`, `McpConfigValidator`, and `McpEnvInterpolator` are removed when their behavior has been folded into existing owners; no replacement abstraction is introduced.
- Prompt-template argument parsing preserves its existing quoting, whitespace, Unicode, and malformed-quote behavior using `mb_str_split()`.
- No MCP Client, Subagent, or Fork file is modified.
- Focused Castor tests for OutputCap, prompt templates, MCP config/catalog, and MCP initialization pass; `castor deptrac`, `castor phpstan`, and `castor cs-check` pass.
- Because OutputCap and MCP tool exposure affect LLM-visible/runtime flow, deterministic `castor check` passes before CODE-REVIEW.
- Final diff is behavior-preserving and removes approximately 300 lines without dependency changes.

## Workflow metadata
Status: ARCHIVE
Branch: task/trim-output-cap-prompt-template-mcp-config
Worktree: /home/ineersa/projects/agent-core-worktrees/trim-output-cap-prompt-template-mcp-config
Fork run: 057j8zzr9icn
PR URL: https://github.com/ineersa/agent-core/pull/376
PR Status: merged
Started: 2026-08-12T22:30:25.063Z
Completed: 2026-08-13T03:18:40.381Z

## Work log
- Created: 2026-08-06T19:55:50.003Z

## Task workflow update - 2026-08-12T22:30:25.064Z
- Moved TODO → IN-PROGRESS.
- Created branch task/trim-output-cap-prompt-template-mcp-config.
- Created worktree /home/ineersa/projects/agent-core-worktrees/trim-output-cap-prompt-template-mcp-config.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/trim-output-cap-prompt-template-mcp-config.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/trim-output-cap-prompt-template-mcp-config.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/trim-output-cap-prompt-template-mcp-config.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/trim-output-cap-prompt-template-mcp-config.
- Summary: Task-start claimed by orchestrator; preparing scoped implementation fork.

## Task workflow update - 2026-08-12T22:36:42.079Z
- Recorded fork run: tbhhvtiyg370
- Summary: Implementation fork launched in task worktree with exact OutputCap, MCP config/catalog, prompt-template scope; focused Castor validation required, full castor check/PR explicitly deferred.

## Task workflow update - 2026-08-13T00:19:15.210Z
- Recorded fork run: tbhhvtiyg370
- Validation: castor test --filter='OutputCap' — OK (41 tests, 196 assertions); castor test --filter='PromptTemplateArgumentParserTest|PromptTemplateServiceTest' — OK (26 tests, 33 assertions); castor test --filter='McpConfigLoaderTest|McpInitializeSessionHandlerTest|SessionFileMcpToolCatalogStoreTest' — OK (40 tests, 159 assertions); castor test --filter='AgentMcpToolsResolverTest|AgentToolPolicyResolverTest|CompactHeaderRegistrarTest|CompactHeaderSnapshotProviderTest|McpConnectionManagerTest' — OK (34 tests, 155 assertions); castor deptrac — OK, 0 violations; castor phpstan — OK; castor cs-check — OK after scoped castor cs-fix; castor test:llm-real --filter='OutputCapReadFileControllerTest' — OK (1 test, 11 assertions); Full castor check intentionally deferred to task-to-pr phase per workflow.
- Summary: Implementation completed and committed as 594dc8e4bf70e0484785ab6bc7105c6162891b79. Verified clean worktree, 24 expected files changed, +628/-1326 (net -698), and no excluded MCP Client production, Subagent, Fork, dependency, settings, or docs paths. Removed folded OutputCap resolver/APIs, MCP validator/interpolator/mapper plumbing, and prompt parser helpers while preserving behavior-focused proofs.

## Task workflow update - 2026-08-13T00:27:46.028Z
- Summary: Reviewer approved commit 594dc8e4bf70e0484785ab6bc7105c6162891b79 with no blocking findings. Specification fidelity confirmed; optional notes only: invalid UTF-8 behavior of mb_str_split and public visibility of extractPathFromArguments. No changes requested.

## Task workflow update - 2026-08-13T00:29:14.910Z
- Validation: Reviewer: APPROVED; specification fidelity confirmed, no blockers; castor test — OK (4378 tests, 16534 assertions); castor deptrac — OK (0 violations, 0 errors); castor phpstan — OK (0 errors); castor cs-check — OK (0 files needing fixes); castor test:llm-real — OK (13 tests, 144 assertions)
- Summary: task-to-pr review and focused validation complete. Reviewer approved with no blocking findings; worktree clean at 594dc8e4bf70e0484785ab6bc7105c6162891b79. Ready for deterministic CODE-REVIEW gate.

## Task workflow update - 2026-08-13T00:31:39.553Z
- Validation: castor check via move_task — FAILED: unrelated SubagentLivePickerControllerTest expected export feedback but observed no-events feedback; castor test --filter='SubagentLivePickerControllerTest::dismissFeedbackReplacesStaleExportFeedbackInPickerHeader' — OK (1 test, 5 assertions)
- Summary: First deterministic CODE-REVIEW gate failed only in unrelated SubagentLivePickerControllerTest::dismissFeedbackReplacesStaleExportFeedbackInPickerHeader. Scoped rerun passed unchanged; task explicitly excludes Subagent files, so no code change made. Retrying gate.

## Task workflow update - 2026-08-13T00:33:49.599Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (112.4s).
- Pushed task/trim-output-cap-prompt-template-mcp-config to origin.
- branch 'task/trim-output-cap-prompt-template-mcp-config' set up to track 'origin/task/trim-output-cap-prompt-template-mcp-config'.
- Created PR: https://github.com/ineersa/agent-core/pull/376
- Validation: castor test — OK (4378 tests, 16534 assertions); castor deptrac — OK (0 violations); castor phpstan — OK; castor cs-check — OK; castor test:llm-real — OK (13 tests, 144 assertions); Scoped rerun of prior gate failure — OK (1 test, 5 assertions)
- Summary: Reviewer approved; focused validation passed. First deterministic gate had one unrelated transient Subagent picker test failure; exact scoped rerun passed unchanged. Retrying deterministic gate at 594dc8e4bf70e0484785ab6bc7105c6162891b79.

## Task workflow update - 2026-08-13T00:48:12.776Z
- Summary: Checked PR #376 inline review comment about replacing manual MCP config checks with Symfony Serializer/Validator. Classified as a valid architectural follow-up/question, not a blocker for this bounded deletion task: DTOs already exist; loader must retain source-aware global/project merge, inherited disable-only, interpolation, cwd resolution, and secret-safe trust-boundary logic. Symfony DTO constraints could remove duplicate scalar checks in a separately scoped refactor without changing semantics.

## Task workflow update - 2026-08-13T00:56:27.517Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Addressing PR #376 review feedback: use existing MCP DTO with Symfony Serializer/Validator for per-server schema and constraints while retaining loader-owned contextual merge, inherited-disable, interpolation, cwd, and secret-safe error logic.

## Task workflow update - 2026-08-13T00:57:33.237Z
- Recorded fork run: 41iudzwpn8jw
- Summary: Review-iteration fork launched to replace per-server MCP manual checks with existing DTO + Symfony Serializer/Validator, retaining only contextual merge/inheritance/interpolation/cwd logic in loader. No new DTOs or dependencies.

## Task workflow update - 2026-08-13T01:18:34.561Z
- Recorded fork run: 2iyj9g5uvq9x
- Summary: Re-review requested changes: preserve inherited disable-only semantics and validation diagnostic order, add dense trust-boundary type/list/map proof, and ensure invalid enum/inapplicable fields fail or ignore safely. Narrow fix fork launched.

## Task workflow update - 2026-08-13T01:35:54.869Z
- Recorded fork run: 2iyj9g5uvq9x
- Validation: Final reviewer — APPROVED, no blockers; castor test --filter='McpConfigLoaderTest|McpInitializeSessionHandlerTest|McpConnectionManagerTest|CompactHeaderRegistrarTest|CompactHeaderSnapshotProviderTest|AgentMcpToolsResolverTest|AgentToolPolicyResolverTest' — OK (94 tests, 350 assertions); castor deptrac — OK (0 violations); castor phpstan — OK (0 errors); castor cs-check — OK (0 files needing fixes)
- Summary: Review iteration complete at d2bb7a0709d94fad35cd2904195cda77d1ac7ff5. Final reviewer APPROVED: prior semantic and trust-boundary proof blockers resolved. Symfony Serializer/Validator now owns normal server schema validation; loader retains source-aware inheritance/merge/interpolation rules. Ready for deterministic gate and PR update.

## Task workflow update - 2026-08-13T01:38:16.972Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (132.4s).
- Pushed task/trim-output-cap-prompt-template-mcp-config to origin.
- branch 'task/trim-output-cap-prompt-template-mcp-config' set up to track 'origin/task/trim-output-cap-prompt-template-mcp-config'.
- PR already exists: https://github.com/ineersa/agent-core/pull/376
- Validation: Final reviewer — APPROVED; Focused MCP/consumer tests — OK (94 tests, 350 assertions); castor deptrac — OK (0 violations); castor phpstan — OK; castor cs-check — OK
- Summary: Addressed PR #376 feedback with existing McpServerDefinitionDTO + Symfony Serializer/Validator, preserving contextual loader semantics. Final re-review approved and focused QA passed at d2bb7a0709d94fad35cd2904195cda77d1ac7ff5.

## Task workflow update - 2026-08-13T01:38:30.411Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/376
- Updated PR Status: open
- Validation: Final reviewer — APPROVED; Focused MCP/consumer tests — OK (94 tests, 350 assertions); castor deptrac/phpstan/cs-check — OK; Deterministic castor check — OK (132.4s)
- Summary: Review feedback iteration pushed to PR #376 at d2bb7a0709d94fad35cd2904195cda77d1ac7ff5. Deterministic castor check passed (132.4s). Replied to inline discussion with implementation summary and validation evidence.
- Replied to https://github.com/ineersa/agent-core/pull/376#discussion_r3771431419 with resolved architecture and QA summary.

## Task workflow update - 2026-08-13T01:51:13.830Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User correctly identified that the review iteration still retained a manual allowed-field walker and enabled type check in McpConfigLoader, contradicting the requested Symfony Serializer/Validator ownership. Reopening to remove those checks rather than merely relocating validation.

## Task workflow update - 2026-08-13T01:56:06.393Z
- Recorded fork run: 057j8zzr9icn
- Summary: User is right: previous review iteration still duplicated schema/type validation in McpConfigLoader. Narrow fix launched to hydrate every definition through Symfony first, delete manual allowed-field/enabled checks and schema list, then retain only source-aware no-transport decisions.

## Task workflow update - 2026-08-13T02:06:58.533Z
- Recorded fork run: 057j8zzr9icn
- Validation: Final reviewer — APPROVED; grep proof — no $allowedFields, is_bool(), or 'Allowed fields:' in McpConfigLoader; Focused MCP/consumer tests — OK (95 tests, 351 assertions); castor deptrac — OK (0 violations); castor phpstan — OK (0 errors); castor cs-check — OK
- Summary: Final correction complete at 6752cf93277ec8a571bd4c5349c4e26def5725bb: deleted loader-owned allowed-field walking, enabled type checking, and duplicated schema lists. Every definition now hydrates through Symfony Serializer/Validator before loader-only inheritance/no-transport decisions. Final reviewer APPROVED.

## Task workflow update - 2026-08-13T02:09:12.428Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (125.8s).
- Pushed task/trim-output-cap-prompt-template-mcp-config to origin.
- branch 'task/trim-output-cap-prompt-template-mcp-config' set up to track 'origin/task/trim-output-cap-prompt-template-mcp-config'.
- PR already exists: https://github.com/ineersa/agent-core/pull/376
- Validation: Final reviewer — APPROVED; Focused MCP/consumer tests — OK (95 tests, 351 assertions); castor deptrac — OK; castor phpstan — OK; castor cs-check — OK
- Summary: Removed the remaining manual MCP schema/type checks identified by the user. Symfony Serializer/Validator now validates every definition; loader retains only contextual no-transport/inheritance and orchestration rules. Approved at 6752cf93277ec8a571bd4c5349c4e26def5725bb.

## Task workflow update - 2026-08-13T02:09:26.026Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/376
- Updated PR Status: open
- Validation: Final reviewer — APPROVED; Focused MCP/consumer tests — OK (95 tests, 351 assertions); castor deptrac/phpstan/cs-check — OK; Deterministic castor check — OK (125.8s)
- Summary: Final user correction pushed at 6752cf93277ec8a571bd4c5349c4e26def5725bb. Manual allowed-field/enabled checks are gone; every server validates through Symfony before contextual loader decisions. Deterministic castor check passed (125.8s), and PR discussion was updated acknowledging/correcting the prior incomplete response.
- Replied with correction at https://github.com/ineersa/agent-core/pull/376#discussion_r3771867322.

## Task workflow update - 2026-08-13T03:18:40.381Z
- Moved CODE-REVIEW → DONE.
- Merged task/trim-output-cap-prompt-template-mcp-config into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                               |   4 +-
 src/CodingAgent/Mcp/Catalog/McpToolNameMapper.php  |  76 ---
 src/CodingAgent/Mcp/Config/McpConfigDTO.php        |  10 -
 src/CodingAgent/Mcp/Config/McpConfigLoader.php     | 266 ++++++++--
 src/CodingAgent/Mcp/Config/McpConfigValidator.php  | 134 -----
 src/CodingAgent/Mcp/Config/McpEnvInterpolator.php  |  84 ----
 .../Mcp/Config/McpServerDefinitionDTO.php          | 261 +++-------
 .../Mcp/Handler/McpInitializeSessionHandler.php    |  47 +-
 .../PromptTemplateArgumentParser.php               |  25 +-
 src/CodingAgent/Tool/OutputCap.php                 | 184 +++++--
 src/CodingAgent/Tool/OutputCapLlmTransformHook.php |  73 +--
 src/CodingAgent/Tool/OutputCapPathResolver.php     | 101 ----
 .../Tool/OutputCapToolResultProcessor.php          |  78 +--
 .../Agent/Execution/AgentMcpToolsResolverTest.php  |   2 +-
 .../Execution/AgentToolPolicyResolverTest.php      |   2 +-
 .../Mcp/Catalog/McpToolNameMapperTest.php          |  98 ----
 .../Mcp/Client/McpConnectionManagerTest.php        |  11 +-
 .../CodingAgent/Mcp/Config/McpConfigLoaderTest.php | 231 ++++++++-
 .../Handler/McpInitializeSessionHandlerTest.php    |  33 +-
 .../Support/Mcp/TestMcpConfigLoaderFactory.php     |  55 +-
 .../CodingAgent/Tool/OutputCapPathResolverTest.php | 236 ---------
 tests/CodingAgent/Tool/OutputCapTest.php           | 554 ++++++++++-----------
 .../CompactHeaderSnapshotProviderTest.php          |   7 +-
 tests/Tui/Listener/CompactHeaderRegistrarTest.php  |   7 +-
 24 files changed, 1028 insertions(+), 1551 deletions(-)
 delete mode 100644 src/CodingAgent/Mcp/Catalog/McpToolNameMapper.php
 delete mode 100644 src/CodingAgent/Mcp/Config/McpConfigValidator.php
 delete mode 100644 src/CodingAgent/Mcp/Config/McpEnvInterpolator.php
 delete mode 100644 src/CodingAgent/Tool/OutputCapPathResolver.php
 delete mode 100644 tests/CodingAgent/Mcp/Catalog/McpToolNameMapperTest.php
 delete mode 100644 tests/CodingAgent/Tool/OutputCapPathResolverTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/trim-output-cap-prompt-template-mcp-config.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/trim-output-cap-prompt-template-mcp-config.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #376 state — MERGED at 2026-08-13T03:18:00Z
- Summary: PR #376 merged on GitHub at d9651d92a83590216af7fdbe087edac3bfc298ac. Moving task to DONE and syncing integration checkout.

## Task workflow update - 2026-08-13T03:21:02.916Z
- Validation: Post-merge LLM_MODE=true castor check — OK; Unit/integration — OK (4411 tests, 16643 assertions); Controller replay — OK (12 tests, 165 assertions); TUI replay — OK (35 tests, 237 assertions); LLM real — OK (13 tests, 144 assertions); deptrac/phpstan/cs-check — OK; QA artifact integrity, leak check, and llama-proxy cache guard — OK
- Summary: PR #376 merged and task completed. Integration checkout synced at 4042b421b6e38f185d23d0ab680768aae7de0043; task worktree and IDEA exclusions removed; integration checkout clean.

## Task workflow update - 2026-08-14T19:53:44+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
