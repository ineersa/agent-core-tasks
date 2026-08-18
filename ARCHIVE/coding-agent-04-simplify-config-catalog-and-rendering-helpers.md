# coding-agent-04: Simplify CodingAgent config, MCP catalog, and rendering helpers

## Goal
## Goal
Combine the three lower-priority CodingAgent cleanup candidates into one bounded task: standardize config hydration, extract MCP catalog construction from its lifecycle handler, and remove trivial XML escaping duplication where that reduces code.

## Architecture report evidence

### Config hydration drift (candidate 5)
`src/CodingAgent/Config/AppConfig.php` already denormalizes and validates `tui`, `logging`, `sessions`, `extensions`, `tools`, `compaction`, and `runtime` via Symfony Serializer + Validator. Other sections use hand-written parsing/validation:
- `PromptsConfig::fromRaw()` manually filters strings and currently documents silent dropping of invalid entries,
- `ForksConfigDTO::fromRaw()` and `ChildExtensionsConfigDTO::fromRaw()` manually parse nested values,
- `AgentsConfig` manually implements type/range checks and custom error strings.
`AgentDefinitionParser` and `RuntimeConfig` are existing reference implementations for strict denormalization plus validation. The current split produces inconsistent error behavior and messages.

### MCP lifecycle handler owns catalog construction (candidate 6)
`src/CodingAgent/Mcp/Handler/McpInitializeSessionHandler.php` is ~501 lines, handles three message types, and also owns `buildCatalog`, tool mapping, tool-name sanitization, and config hashing. Catalog construction is independently testable policy and can move to one focused collaborator, leaving lifecycle handling thin.

### XML escaping duplication (candidate 7)
The identical `htmlspecialchars($value, ENT_XML1 | ENT_QUOTES, 'UTF-8')` appears in:
- `src/CodingAgent/Skills/SkillContextRenderer.php`,
- `src/CodingAgent/Agent/Context/AgentContextRenderer.php`,
- `src/CodingAgent/SystemPrompt/AgentsContextRenderer.php`.
This is deliberately the smallest item. Reuse an existing renderer/helper if one naturally owns XML-ish LLM context output; otherwise leave the three stdlib one-liners rather than adding a new abstraction that increases coupling.

## Smallest viable direction
- Use the existing Serializer/Validator pattern for complete config-owning boundaries; do not add codecs, normalizers, mappers, compatibility shims, or another Serializer stack.
- Preserve finalized semantic rules, including nested agent-extension constraints.
- Extract only MCP catalog construction/policy; do not split every private method into a service.
- Treat XML escaping as an evidence-based cleanup, not a mandatory helper class.

## Required decision before implementation
Config invalid-input behavior is externally observable. The report proposes unified Validator failures, but `PromptsConfig` currently silently drops invalid entries. During task-explain, inventory existing documented behavior/tests and get explicit user approval before changing silent-drop vs fail-fast semantics or public error text. Do not decide this inside the fork.

## Scope boundaries
- No new config keys, settings, catalog format, MCP protocol behavior, tool naming behavior, hash semantics, or public API.
- Preserve catalog ordering, serialization keys, strict malformed-data handling, and lifecycle timing.
- Do not combine unrelated MCP transport/client changes.

## Test impact noted by the report
Config behavior tests should remain contract-focused; update error assertions only after the explicit behavior decision. Extract catalog-builder tests from handler tests while retaining lifecycle integration proof. XML escaping already has coverage and needs no standalone trivial test.

## Acceptance criteria
- Config sections use one existing Serializer/Validator idiom where it reduces manual parsing without adding glue or changing unapproved behavior.
- Silent-drop versus fail-fast behavior and any user-visible validation-message change are explicitly decided and recorded before implementation.
- `McpInitializeSessionHandler` delegates catalog building, tool mapping/sanitization, and config hashing to one focused collaborator; message lifecycle behavior stays unchanged.
- MCP catalog ordering, names, hashes, serialized keys, malformed-data behavior, and protocol output remain unchanged.
- The three XML escaping copies are reused only if an existing or minimal natural owner reduces total code; no standalone abstraction is added merely to remove one-line stdlib calls.
- No new setting, API, compatibility shim, custom normalizer, or Serializer stack is introduced.
- The fork and handoff state that `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` were read and followed.
- Focused Castor tests, `castor deptrac`, `castor phpstan`, `castor cs-check`, and final `castor check` pass.

## Workflow metadata
Status: ARCHIVE
Branch: task/coding-agent-04-simplify-config-catalog-and-rendering-helpers
Worktree: /home/ineersa/projects/agent-core-worktrees/coding-agent-04-simplify-config-catalog-and-rendering-helpers
Fork run: phfwwrmg2ij7
PR URL: https://github.com/ineersa/agent-core/pull/402
PR Status: merged
Started: 2026-08-17T15:46:31.379Z
Completed: 2026-08-17T22:13:39.584Z

## Work log
- Created: 2026-08-15T23:06:31.078Z

## Task workflow update - 2026-08-15T23:19:03.253Z
- Full architect report: [`/home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md`](file:///home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md) — CodingAgent report, candidates 5–7.

## Task workflow update - 2026-08-17T15:46:31.379Z
- Moved TODO → IN-PROGRESS.
- Created branch task/coding-agent-04-simplify-config-catalog-and-rendering-helpers.
- Created worktree /home/ineersa/projects/agent-core-worktrees/coding-agent-04-simplify-config-catalog-and-rendering-helpers.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/coding-agent-04-simplify-config-catalog-and-rendering-helpers.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/coding-agent-04-simplify-config-catalog-and-rendering-helpers.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/coding-agent-04-simplify-config-catalog-and-rendering-helpers.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/coding-agent-04-simplify-config-catalog-and-rendering-helpers.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/coding-agent-04-simplify-config-catalog-and-rendering-helpers/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/coding-agent-04-simplify-config-catalog-and-rendering-helpers.
- Summary: Starting implementation phase. Required config invalid-input behavior decision remains open; will inventory current documented/tested behavior before asking the user and launching implementation.

## Task workflow update - 2026-08-17T15:53:02.748Z
- Summary: Three read-only scouts completed. MCP extraction is the only unblocked code change: add one concrete McpToolCatalogBuilder and move build/mapping/sanitization/hash policy while retaining callbacks, writes, lifecycle timing, failure invalidation, count/log/error helpers in the handler. XML helper should be skipped: no natural owner exists and three stdlib one-liners are less coupling/code. Config standardization is blocked on the task's required user decision; strict Serializer/Validator cannot preserve current mixed fallback/filter/trim/custom-message semantics without new glue.
- Config inventory: prompts silently drops malformed entries; forks/agents mix silent fallback with strict custom InvalidArgumentException contracts; child extensions have role-dependent policy. Runtime is already strict Serializer+Validator. Migrating manual sections either changes external invalid-input behavior/messages or adds normalization glue.
- Recommended config decision: preserve current silent-drop/fallback semantics and leave manual policy owners unchanged; this satisfies minimality and avoids an unapproved error surface. Alternative is explicit fail-fast prompts via Serializer+Validator with new validation exception/message contract.
- MCP extraction target: src/CodingAgent/Mcp/Catalog/McpToolCatalogBuilder.php (concrete, no interface/factory) plus handler injection; preserve order, first-seen collisions, exact sanitization/hash fields/flags, malformed propagation, serialized shape and partial/final write timing.
- XML candidate skipped: no shared XML helper exists; moving three htmlspecialchars one-liners creates a cross-layer utility/Deptrac coupling and does not reduce code.
- All scouts stated .agents/skills/testing/SKILL.md and tests/AGENTS.md were read and followed; no tests or edits run.

## Task workflow update - 2026-08-17T15:56:25.687Z
- Summary: Config decision finalized by user: explicitly malformed configured values must fail fast; silent dropping/default substitution makes users believe invalid configuration works. Missing/omitted values may keep documented defaults, but a present invalid value must surface a validation error. Apply this consistently to the target prompts/forks/agents sections while preserving valid-value semantics and nested extension rules.
- User rationale: silently turning invalid input into a valid reduced/default config hides mistakes and creates false confidence. No compatibility shim or warning-only path; fail fast at config load.

## Task workflow update - 2026-08-17T16:00:09.442Z
- Recorded fork run: infejoxbykwx
- Summary: Implementation fork launched with finalized fail-fast semantics for every explicit malformed prompts/forks/agents value; omitted values alone retain defaults. Fork also performs the bounded MCP catalog builder extraction and skips the counterproductive XML helper.

## Task workflow update - 2026-08-17T16:15:27.847Z
- Recorded fork run: phfwwrmg2ij7
- Summary: Initial implementation commit d4435f1ce is functionally green but not accepted yet. A narrow cleanup fork is removing the unapproved tui.theme_paths behavior extension and collapsing redundant tests/comments while preserving finalized config fail-fast and MCP contracts.
- Verification: d4435f1ce exists, worktree clean, 13 files +836/-389; production+depfile net +66, tests net +381; IDE diagnostics 0 on builder; full Castor gates reported green.
- Rejected for handoff as-is: loader strictness unintentionally affected tui.theme_paths outside target prompts/forks/agents scope; 29 added test methods and +491 test lines duplicate DTO/builder behavior beyond the smallest contract proof.
- Cleanup target: restore exact theme_paths behavior; reduce builder tests 9→≤3 and AppConfig integration tests 12→minimal representative/data-driven proof; trim only new prose while retaining moved MCP invariant comments.

## Task workflow update - 2026-08-17T16:27:48.075Z
- Validation: castor test PASS (4509 tests, 17744 assertions) after cleanup; Final AppConfigLoaderTest PASS (35 tests, 96 assertions); Focused config tests PASS (AppConfigLoader/AppConfig/PromptsConfig/AgentsConfig); Focused MCP tests PASS (McpToolCatalogBuilder + McpInitializeSessionHandler: 11 tests, 69 assertions); castor deptrac PASS (0 violations); castor phpstan PASS (0 errors); castor cs-check PASS (0 files fixed); git diff --check clean; IDE diagnostics: changed production files 0 problems; one pre-existing AppConfigTest serializer-interface warning unchanged from origin/main
- Summary: Implementation complete at HEAD 0b955a1f0 (commits d4435f1ce + fc5cc0ddb + 0b955a1f0), worktree clean and unpushed. Final cumulative diff: 14 files +687/-418; production+depfile +344/-279 (net +65), tests +343/-139 (net +204). Explicit malformed prompts/forks/agents values now fail during AppConfig load; omitted values retain defaults and documented null/blank fork scalar values retain their valid 'unset' meaning. McpInitializeSessionHandler delegates catalog construction/mapping/sanitization/hash policy to one concrete McpToolCatalogBuilder while lifecycle/timing/writes/errors remain unchanged. XML helper deliberately skipped.
- Strict config uses the existing DTO fromRaw boundaries rather than Symfony ObjectNormalizer because empirical probing showed ObjectNormalizer coerces wrong scalar types before Validator can reject them; adding a custom normalizer was explicitly forbidden and would add more glue.
- AppConfigLoader validates malformed prompts/agents path-list entries before path resolution so invalid entries cannot be masked; tui.theme_paths retains exact legacy non-string skip behavior and now has a focused regression test.
- MCP builder tests collapsed from 9 to 3 contract tests; AppConfig integration tests collapsed from 12 to 3/data-driven proof. Follow-up cleanup removed 199 net lines and restored scope.
- No TUI/controller/live-LLM validation required: this is config hydration and internal MCP catalog policy extraction with no TUI or provider-visible flow. `castor check` intentionally deferred to task-to-pr per workflow.

## Task workflow update - 2026-08-17T16:56:50.890Z
- Summary: Task-to-PR reviewer verdict: REQUEST CHANGES. MCP extraction and strict-fromRaw design approved, but config fail-fast is incomplete for associative-map/list shape mismatches and missing acceptance proof for valid fork null/blank unset semantics. Root-cause review also found explicit null sections and unknown keys can still be silently treated as omission/ignored, contrary to the user's finalized 'incorrect settings must fail; only omitted values default' rule.
- Blocking: AppConfigLoader rewrites associative prompts/agents.paths maps into lists before DTO checks; strict list shapes need array_is_list validation before resolution and direct DTO parity.
- Blocking: non-empty list supplied for agents/forks mapping silently returns defaults; reject list-shaped mappings while allowing empty array because YAML empty map/list is indistinguishable.
- Required proof: forks.model/thinking_level explicit null/blank remain documented valid unset values.
- Root-cause extension mapped to finalized requirement: reject explicit null target sections and unknown keys in agents/forks/nested child-extension mappings; otherwise typos and explicit invalid values remain silently ignored. Omitted sections/keys alone retain defaults.
- Minimality follow-up: encode strict-list in PATH_CONFIG instead of hardcoding path comparisons; hoist path name; no new helper class.

## Task workflow update - 2026-08-17T17:20:25.679Z
- Validation: Reviewer final verdict: APPROVED; castor test PASS (4532 tests, 17792 assertions); castor deptrac PASS (0 violations); castor phpstan PASS (0 errors); castor cs-check PASS (0 files fixed); git diff --check clean; Focused final unknown-key tests PASS (60 tests, 144 assertions); IDE diagnostics on changed production config/MCP files: 0 problems
- Summary: Task-to-PR review complete at HEAD 105aafa53. Reviewer initially requested config-shape/null/unknown-key fixes; commits 6dece1619 + 105aafa53 closed all blockers. Final reviewer verdict APPROVED. Explicit malformed target settings now fail actionably (including wrong shapes, null sections, unknown keys and nested extension typos), omission alone defaults, valid fork null/blank unset remains. MCP extraction approved as fidelity-preserving and minimal; XML helper skip approved.
- Final unknown-key fix uses reset(array_diff(...)) value rather than array_key_first numeric index; tests pin actual bogus key and special agents.extensions.enabled guidance.
- Cumulative diff vs origin/main at review: 15 files +849/-437; production+depfile +422/-294, tests +427/-143.
- Final review confirmed no remaining silent malformed paths in target sections, exact theme_paths legacy scope, no MCP lifecycle/hash/order/log regressions, and no XML renderer changes.

## Task workflow update - 2026-08-17T17:27:45.611Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (131.1s).
- Pushed task/coding-agent-04-simplify-config-catalog-and-rendering-helpers to origin.
- branch 'task/coding-agent-04-simplify-config-catalog-and-rendering-helpers' set up to track 'origin/task/coding-agent-04-simplify-config-catalog-and-rendering-helpers'.
- Created PR: https://github.com/ineersa/agent-core/pull/402
- Validation: castor test: PASS (4532 tests, 17792 assertions); castor deptrac: PASS (0 violations); castor phpstan: PASS (0 errors); castor cs-check: PASS (0 files fixed); git diff --check: clean; Reviewer: APPROVED
- Summary: Retrying after sibling tui-04 castor check released the repository lock. Previous retry failed only on the 60-second lock acquisition timeout; no task code or validation changed.

## Task workflow update - 2026-08-17T17:27:52.429Z
- Validation: CODE-REVIEW castor check PASS (131.1s)
- Summary: PR #402 created at https://github.com/ineersa/agent-core/pull/402. Deterministic castor check passed in 131.1s; branch pushed at 105aafa53.
- First PR creation attempt hit transient GitHub GraphQL HTTP 503 after branch push; task remained IN-PROGRESS.
- First retry hit only repository castor-check lock contention from sibling tui-04 worktree; waited for holder to exit without signaling it, then retried successfully.

## Task workflow update - 2026-08-17T17:40:10.231Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User requested merging current origin/main into the PR branch. Returning to IN-PROGRESS before changing branch history.

## Task workflow update - 2026-08-17T17:54:01.200Z
- Validation: Final merge reviewer: APPROVED; composer install --no-interaction --no-progress PASS; lock unchanged; Symfony AI installed v0.12.0 matches lock; castor test PASS (4578 tests, 17936 assertions); castor deptrac PASS (0 violations); castor phpstan PASS (0 errors); castor cs-check PASS (0 files fixed); git diff --check clean
- Summary: Merged current origin/main (bd1e74902) into PR branch as normal conflict-free merge c8557b17e. Synced worktree vendor to merged composer.lock without tracked changes. Final merge re-review APPROVED; task-only diff remains 15 files +849/-437 and remote task SHA 105aafa53 is an ancestor, so normal push is fast-forward.
- User-requested origin/main merge completed as c8557b17e with parents 105aafa53 and bd1e74902; no conflicts or manual resolutions.
- Reviewer initially blocked only on stale copied vendor after main's dependency bump; vendor sync closed it with no tracked changes.
- Final reviewer confirmed current-main tool/schema changes have zero task-file overlap and no semantic conflict with config strictness or MCP catalog extraction.

## Task workflow update - 2026-08-17T18:01:34.405Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (142.6s).
- Pushed task/coding-agent-04-simplify-config-catalog-and-rendering-helpers to origin.
- branch 'task/coding-agent-04-simplify-config-catalog-and-rendering-helpers' set up to track 'origin/task/coding-agent-04-simplify-config-catalog-and-rendering-helpers'.
- Skipped PR creation (pushOnly: true).
- Validation: castor test PASS (4578 tests, 17936 assertions); castor deptrac PASS (0 violations); castor phpstan PASS (0 errors); castor cs-check PASS (0 files fixed); git diff --check clean; Merge reviewer APPROVED
- Summary: Retry after integration checkout castor check released the shared repository lock. Existing PR #402 already tracks pushed merge c8557b17e; skip redundant PR creation.

## Task workflow update - 2026-08-17T18:01:47.159Z
- Validation: Post-origin/main-merge castor check PASS (142.6s)
- Summary: Current origin/main merged into PR #402 branch as c8557b17e and pushed normally. Deterministic post-merge castor check passed in 142.6s; task returned to CODE-REVIEW.
- Initial CODE-REVIEW retry hit transient GitHub GraphQL HTTP 503 after the branch had already pushed; existing PR #402 remains the tracked PR.
- Next retry hit only shared repository castor-check lock contention from integration checkout; waited for the holder to exit without signaling it, then pushOnly transition succeeded.

## Task workflow update - 2026-08-17T22:13:39.584Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/coding-agent-04-simplify-config-catalog-and-rendering-helpers.
- Merged task/coding-agent-04-simplify-config-catalog-and-rendering-helpers into integration checkout.
- Merge made by the 'ort' strategy.
 depfile.yaml                                       |   3 +-
 src/CodingAgent/Config/AgentsConfig.php            |  65 +++++-
 src/CodingAgent/Config/AppConfig.php               |  32 +--
 src/CodingAgent/Config/AppConfigLoader.php         |  29 ++-
 .../Config/ChildExtensionsConfigDTO.php            |  23 +-
 src/CodingAgent/Config/ForksConfigDTO.php          |  60 +++--
 src/CodingAgent/Config/PromptsConfig.php           |  26 ++-
 .../Mcp/Catalog/McpToolCatalogBuilder.php          | 242 +++++++++++++++++++++
 .../Mcp/Handler/McpInitializeSessionHandler.php    | 236 +-------------------
 tests/CodingAgent/Config/AgentsConfigTest.php      |  72 ++++--
 tests/CodingAgent/Config/AppConfigLoaderTest.php   |  21 ++
 tests/CodingAgent/Config/AppConfigTest.php         | 189 +++++++++++++---
 tests/CodingAgent/Config/PromptsConfigTest.php     |  49 ++++-
 .../Mcp/Catalog/McpToolCatalogBuilderTest.php      | 143 ++++++++++++
 .../Handler/McpInitializeSessionHandlerTest.php    |  96 +-------
 15 files changed, 849 insertions(+), 437 deletions(-)
 create mode 100644 src/CodingAgent/Mcp/Catalog/McpToolCatalogBuilder.php
 create mode 100644 tests/CodingAgent/Mcp/Catalog/McpToolCatalogBuilderTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/coding-agent-04-simplify-config-catalog-and-rendering-helpers.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/coding-agent-04-simplify-config-catalog-and-rendering-helpers.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-08-17T22:16:26.719Z
- Validation: Post-merge LLM_MODE=true castor check PASS (qa-20260817-221344-606892-d19d0d73, 417.9s): deptrac OK, test OK (4589 tests/17964 assertions), controller-replay OK (12/165), tui OK (38/295), llm-real OK (13/144), phpstan OK (0 errors), cs-check OK, docs:validate OK; QA artifact integrity ok, leak check ok, llama-proxy cache stable 348→348; Integration checkout clean at efc46deb4; worktree removed
- Summary: PR #402 merged (dcc207c0b); task branch merged into integration checkout (23178a912), synced with origin/main via pull (efc46deb4). Worktree removed, IDEA exclusions cleaned, JetBrains project closed. Post-merge QA all 8 lanes green.

## Task workflow update - 2026-08-18T00:06:36.284Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
