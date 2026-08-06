# Publish Extension API and four extensions as release-split Composer repositories

## Goal
## Confirmed direction

Keep `agent-core` as the single source of truth. On each Hatfield `v*` release, the existing release workflow publishes five one-way repository mirrors:

1. `ineersa/hatfield-extension-api`
2. `ineersa/hatfield-ext-task-workflow`
3. `ineersa/hatfield-ext-castor-llm-mode`
4. `ineersa/hatfield-ext-file-rewind`
5. `ineersa/hatfield-ext-observational-memory`

The mirrors are distribution repositories, not independent development repositories. Normal changes and pull requests remain in `agent-core`.

## Problem

The four extensions already have package-like Composer manifests under `.hatfield/extensions/`, but they are not independently installable:

- the public `Ineersa\Hatfield\ExtensionApi` contracts live under `src/CodingAgent/ExtensionApi/` rather than in a Composer package;
- extension manifests rely on host-provided classes and omit direct dependencies;
- root Composer, PHPUnit, PHPStan, CS and Deptrac wiring directly reference monorepo paths;
- the existing release workflow publishes Hatfield artifacts only and has no subtree split step.

## Implementation scope

### 1. Extract the public API inside the monorepo

Create a canonical package directory at `.hatfield/extensions/extension-api/`, preserving the existing `Ineersa\Hatfield\ExtensionApi` namespace and public contracts. Move the current `src/CodingAgent/ExtensionApi/` implementation into that package using semantic file moves where available.

Update the host Composer/runtime wiring so Hatfield consumes the package from the local path repository during monorepo development. Do not introduce compatibility copies, fallback autoloaders, duplicate API classes, or namespace changes.

### 2. Make all five directories valid Composer packages

Add/update package manifests so every package declares its direct production dependencies and the four extensions require `ineersa/hatfield-extension-api`.

Known dependency corrections from reconnaissance:

- `file-rewind`: declare PSR logging and Symfony TUI dependencies it imports;
- `observational-memory`: declare Toon, Symfony Clock and SQLite/runtime requirements in addition to its existing DBAL/PSR requirements;
- preserve existing package names and extension namespaces/classes;
- keep local development through `.hatfield/extensions/composer.json` path repositories, now including `extension-api`.

Use Composer constraints compatible with the versions already installed by `agent-core`; do not add speculative dependencies.

### 3. Preserve monorepo validation ownership

Keep extension integration, kernel, runtime and TUI tests in `agent-core`. Update root PHPUnit, PHPStan, CS and Deptrac configuration only where paths/boundaries changed. Do not duplicate Hatfield’s full QA suite into mirror repositories in this task.

The Extension API boundary remains public and may depend only on its documented public dependencies (currently Symfony TUI types where explicitly approved); it must not gain dependencies on Hatfield internals, AgentCore, runtime wiring, settings, DI or packaging code.

### 4. Release-time repository splitting

Extend the existing `.github/workflows/release.yml` rather than creating a separate manually triggered release process.

On a successful Hatfield `v*` release, split and push these prefixes to the matching repositories:

- `.hatfield/extensions/extension-api` → `ineersa/hatfield-extension-api`
- `.hatfield/extensions/task-workflow` → `ineersa/hatfield-ext-task-workflow`
- `.hatfield/extensions/castor-llm-mode` → `ineersa/hatfield-ext-castor-llm-mode`
- `.hatfield/extensions/file-rewind` → `ineersa/hatfield-ext-file-rewind`
- `.hatfield/extensions/observational-memory` → `ineersa/hatfield-ext-observational-memory`

Use one matrix-driven split job and one maintained splitter rather than five copied jobs or a custom split framework. Publish the source release tag as the same `vX.Y.Z` tag in each destination repository and update its release branch/default branch to the split commit. The job must run only for release tags and only after the release’s required validation/build prerequisites succeed.

Cross-repository writes require a dedicated least-privilege Actions secret (document the chosen secret name and required repository permissions). Destination repositories are provisioned outside this code change; the workflow must fail clearly rather than silently skipping a missing repository or credential.

### 5. Documentation

Document:

- monorepo is authoritative and split repositories are read-only mirrors;
- the five source-prefix/destination mappings;
- release-triggered synchronization and shared Hatfield release version;
- required destination repository provisioning and Actions credential;
- how another Hatfield project installs released packages, while this repository continues to use path repositories locally.

Update existing distribution/extension documentation rather than adding an overlapping architecture document.

## Non-goals

- Independent package development, maintainers, permissions or release cadence.
- Accepting pull requests directly in split repositories.
- Private Packagist or custom package registry infrastructure.
- A general-purpose monorepo release framework.
- Backward-compatibility autoloading or duplicate API locations.
- Creating the five GitHub repositories or storing credentials from repository code.
- Moving root-dependent integration/TUI tests into package mirrors.

## Test thesis

The regression to prevent is that extracting the API or correcting package ownership breaks Hatfield container/runtime extension loading, architectural boundaries, or existing extension behavior. Existing extension, runtime and TUI proofs should continue to run through the canonical monorepo paths; add only the smallest focused test if an existing stable contract is otherwise unprotected.

## Required validation

All QA must use Castor. Before test work, forks must read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` and state this in handoff.

Focused implementation validation:

- `castor test`
- `castor deptrac`
- `castor phpstan`
- `castor cs-check`
- Composer/package manifest validation through an existing Castor task if available

Because this changes public extension contracts consumed by runtime/TUI extensions, final CODE-REVIEW gating requires `castor check` via the task workflow. Do not run raw `vendor/bin/*` QA commands.

## Acceptance criteria
- A fifth local Composer package, `ineersa/hatfield-extension-api`, owns the existing public ExtensionApi classes under `.hatfield/extensions/extension-api/` with namespace and API behavior preserved.
- Hatfield and all four local extensions resolve the API through Composer package dependencies; no duplicate classes, compatibility autoloaders or old API source copy remain.
- All five package manifests validate and declare their direct production dependencies using constraints compatible with the host application.
- Existing root extension tests and QA configuration use the new canonical paths and continue to enforce the Extension API architecture boundary.
- The existing release workflow has one matrix-driven, release-tag-only split job covering exactly five source-prefix/destination mappings.
- A `vX.Y.Z` Hatfield release updates each destination repository and publishes the same `vX.Y.Z` tag after required release prerequisites succeed.
- Cross-repository authentication, least-privilege permissions, repository pre-provisioning and failure behavior are documented; no credentials are committed.
- Documentation states that `agent-core` is authoritative and split repositories are read-only distribution mirrors, with installation guidance for another Hatfield project.
- Focused Castor validation passes, and the later CODE-REVIEW transition passes deterministic `castor check`.

## Workflow metadata
Status: DONE
Branch: task/release-split-extension-packages
Worktree: /home/ineersa/projects/agent-core-worktrees/release-split-extension-packages
Fork run: 0mkejg8tsya2
PR URL: https://github.com/ineersa/agent-core/pull/362
PR Status: merged
Started: 2026-08-04T20:47:13.549Z
Completed: 2026-08-05T01:43:27.147Z

## Work log
- Created: 2026-08-04T20:47:07.656Z

## Task workflow update - 2026-08-04T20:47:13.549Z
- Moved TODO → IN-PROGRESS.
- Created branch task/release-split-extension-packages.
- Created worktree /home/ineersa/projects/agent-core-worktrees/release-split-extension-packages.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/release-split-extension-packages.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/release-split-extension-packages.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/release-split-extension-packages.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/release-split-extension-packages.
- Summary: Task created from the agreed architecture: agent-core remains authoritative; the Extension API plus four extensions are published as five one-way repository mirrors from the existing v* release workflow.

## Task workflow update - 2026-08-04T20:48:23.465Z
- Recorded fork run: 0mkejg8tsya2
- Implementation fork launched in `/home/ineersa/projects/agent-core-worktrees/release-split-extension-packages` with finalized five-package extraction, release-split workflow, documentation, focused Castor validation, and commit instructions.

## Task workflow update - 2026-08-04T21:39:34.222Z
- Recorded fork run: 0mkejg8tsya2
- Validation: Fork confirmed it read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before test work and followed Castor-only conventions.; Composer validate on root and all five package manifests: valid.; `castor test`: PASS — 4,434 tests, 16,492 assertions.; `castor deptrac`: PASS — 0 violations.; `castor phpstan`: PASS — 0 errors.; `castor cs-check`: PASS.; `castor phar:build`: PASS including PHAR smoke and packaged ExtensionApi presence.; `castor check`: intentionally not run during task-start; reserved for task-to-pr/CODE-REVIEW gate.
- Summary: Implementation complete and committed as `70521c29e2db5b323fd3fb31be73852ec032867c` (`feat(extensions): extract ExtensionApi package and release-split mirrors`). Verified the worktree is clean. The commit moves the unchanged public API classes into `.hatfield/extensions/extension-api`, wires five valid local Composer packages and direct dependencies, updates Deptrac/runtime/PHAR packaging, adds the five-entry release-only split matrix, and documents mirror provisioning and installation. No repositories, credentials, pushes, or release runs were performed.

## Task workflow update - 2026-08-04T22:46:15.673Z
- Summary: Provisioned and verified all five public GitHub mirror repositories under `ineersa`; each has default branch `main` and the read-only mirror description. No code, tokens, secrets, rulesets, or releases were changed.
- Created `https://github.com/ineersa/hatfield-extension-api`.
- Created `https://github.com/ineersa/hatfield-ext-task-workflow`.
- Created `https://github.com/ineersa/hatfield-ext-castor-llm-mode`.
- Created `https://github.com/ineersa/hatfield-ext-file-rewind`.
- Created `https://github.com/ineersa/hatfield-ext-observational-memory`.
- Verified all five repositories are PUBLIC with default branch `main`. `MONOREPO_SPLIT_TOKEN` remains to be created and stored before the first release split.

## Task workflow update - 2026-08-04T23:02:30.315Z
- Summary: task-to-pr review returned REQUEST CHANGES. Blocking fixes accepted: remove unrelated Composer lock upgrades and simplify the PHAR path-package materialization while retaining the proven workaround Composer requires. Also remove duplicate Box source inclusion if PHAR smoke proves vendor-only packaging. Reviewer’s initialized-main concern was investigated against the pinned splitter source: it clones `main`, commits deletion of the seeded README plus package contents, then performs a normal push, so no force push or repo reset is needed. Token presence was verified via `gh secret list`.
- Reviewer verdict: REQUEST CHANGES.
- Verified `MONOREPO_SPLIT_TOKEN` exists on `ineersa/agent-core` (value not read or exposed).
- Pinned splitter v2.4.5 handles initialized mirror `main` by retaining `.git`, staging package contents in an otherwise empty worktree, committing README deletion/package additions, and pushing normally.
- Non-blocking reviewer suggestions rejected as out of scope or inaccurate: retain API PHP >=8.3 consistent with extension packages; retain Deptrac coverage explicitly requested by task instructions; external Hatfield projects can resolve stable `symfony/tui` without the root development fork.

## Task workflow update - 2026-08-04T23:30:49.106Z
- Validation: Reviewer: APPROVED after reading testing conventions and reviewing the complete final diff.; `timeout 200s castor test` in task worktree: PASS — 4,434 tests, 16,492 assertions.; `timeout 200s castor deptrac` in task worktree: PASS — 0 violations, 0 errors.; `timeout 200s castor phpstan` in task worktree: PASS — 0 errors.; `timeout 200s castor cs-check` in task worktree: PASS — 0 files changed.; Review-fix fork: bounded `castor phar:build` PASS with existing PHAR smoke.; Worktree status after validation: clean.
- Summary: task-to-pr re-review APPROVED after review fix commit `b3884256b`. Final worktree is clean. Review confirmed lock restoration, package-specific PHAR materialization, Box deduplication, release workflow security/semantics, and specification fidelity. The five public mirrors and `MONOREPO_SPLIT_TOKEN` are provisioned.

## Task workflow update - 2026-08-04T23:32:48.458Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 240s)...
- castor check passed (106.2s).
- Pushed task/release-split-extension-packages to origin.
- branch 'task/release-split-extension-packages' set up to track 'origin/task/release-split-extension-packages'.
- Created PR: https://github.com/ineersa/agent-core/pull/362

## Task workflow update - 2026-08-05T01:43:27.147Z
- Moved CODE-REVIEW → DONE.
- Merged task/release-split-extension-packages into integration checkout.
- Auto-merging config/services.yaml
Auto-merging depfile.yaml
Merge made by the 'ort' strategy.
 .castor/helpers.php                                | 48 ++++++++++++
 .github/workflows/release.yml                      | 60 +++++++++++++++
 .hatfield/extensions/castor-llm-mode/README.md     |  2 +-
 .hatfield/extensions/castor-llm-mode/composer.json |  5 +-
 .hatfield/extensions/composer.json                 |  5 ++
 .hatfield/extensions/extension-api/composer.json   | 17 +++++
 .../src}/Agent/AgentCallRequestDTO.php             |  0
 .../src}/Agent/AgentRunnerInterface.php            |  0
 .../extension-api/src}/Agent/AgentToolDTO.php      |  0
 .../Agent/ExtensionAgentJobHandlerInterface.php    |  0
 .../src}/Agent/ExtensionAgentJobRequestDTO.php     |  0
 .../src}/Approval/ApprovalAnswerContextDTO.php     |  0
 .../src}/Approval/ApprovalAnswerHookInterface.php  |  0
 .../src}/Command/CommandContextInterface.php       |  0
 .../src}/Command/CommandDefinitionDTO.php          |  0
 .../src}/Command/CommandRegistryInterface.php      |  0
 .../Command/ExtensionCommandHandlerInterface.php   |  0
 .../Compaction/BeforeCompactionHookContextDTO.php  |  0
 .../Compaction/BeforeCompactionHookInterface.php   |  0
 .../Compaction/BeforeCompactionHookResultDTO.php   |  0
 .../extension-api/src}/Exec/ExecInterface.php      |  0
 .../extension-api/src}/Exec/ExecOptionsDTO.php     |  0
 .../extension-api/src}/Exec/ExecResultDTO.php      |  0
 .../extension-api/src}/ExtensionApiInterface.php   |  0
 .../src}/HatfieldExtensionInterface.php            |  0
 .../Lifecycle/AfterTurnCommitEventSummaryDTO.php   |  0
 .../Lifecycle/AfterTurnCommitHookContextDTO.php    |  0
 .../Lifecycle/AfterTurnCommitHookInterface.php     |  0
 .../src}/Prompt/PromptContributorInterface.php     |  0
 .../Prompt/PromptContributorProviderInterface.php  |  0
 .../extension-api/src}/Session/SessionEventDTO.php |  0
 .../src}/Session/SessionEventReaderException.php   |  0
 .../src}/Session/SessionEventReaderInterface.php   |  0
 .../ContextualExtensionToolHandlerInterface.php    |  0
 .../src}/Tool/ExtensionToolHandlerInterface.php    |  0
 .../extension-api/src}/Tool/ToolCallContextDTO.php |  0
 .../src}/Tool/ToolCallDecisionDTO.php              |  0
 .../src}/Tool/ToolCallDecisionKindEnum.php         |  0
 .../src}/Tool/ToolCallHookInterface.php            |  0
 .../src}/Tool/ToolCallRewriteHookInterface.php     |  0
 .../Tool/ToolCallRewriteHookProviderInterface.php  |  0
 .../src}/Tool/ToolCancellationTokenInterface.php   |  0
 .../src}/Tool/ToolInvocationContextDTO.php         |  0
 .../src}/Tool/ToolRegistrationDTO.php              |  0
 .../src}/Tool/ToolResultContextDTO.php             |  0
 .../src}/Tool/ToolResultDecisionDTO.php            |  0
 .../src}/Tool/ToolResultDecisionKindEnum.php       |  0
 .../src}/Tool/ToolResultHookInterface.php          |  0
 .../src}/Tui/TuiExtensionContextInterface.php      |  0
 .../src}/Tui/TuiExtensionInterface.php             |  0
 .../src}/Tui/TuiProjectExtensionInterface.php      |  0
 .hatfield/extensions/file-rewind/composer.json     | 15 +++-
 .../extensions/observational-memory/composer.json  |  6 +-
 .hatfield/extensions/task-workflow/README.md       |  2 +-
 .hatfield/extensions/task-workflow/composer.json   |  5 +-
 AGENTS.md                                          |  4 +-
 composer.json                                      | 11 ++-
 composer.lock                                      | 89 +++++++++++++++-------
 config/services.yaml                               |  1 -
 depfile.yaml                                       | 15 +++-
 docs/distribution.md                               | 71 ++++++++++++++++-
 docs/hitl-and-approvals.md                         |  4 +-
 62 files changed, 312 insertions(+), 48 deletions(-)
 create mode 100644 .hatfield/extensions/extension-api/composer.json
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Agent/AgentCallRequestDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Agent/AgentRunnerInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Agent/AgentToolDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Agent/ExtensionAgentJobHandlerInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Agent/ExtensionAgentJobRequestDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Approval/ApprovalAnswerContextDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Approval/ApprovalAnswerHookInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Command/CommandContextInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Command/CommandDefinitionDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Command/CommandRegistryInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Command/ExtensionCommandHandlerInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Compaction/BeforeCompactionHookContextDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Compaction/BeforeCompactionHookInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Compaction/BeforeCompactionHookResultDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Exec/ExecInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Exec/ExecOptionsDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Exec/ExecResultDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/ExtensionApiInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/HatfieldExtensionInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Lifecycle/AfterTurnCommitEventSummaryDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Lifecycle/AfterTurnCommitHookContextDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Lifecycle/AfterTurnCommitHookInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Prompt/PromptContributorInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Prompt/PromptContributorProviderInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Session/SessionEventDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Session/SessionEventReaderException.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Session/SessionEventReaderInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tool/ContextualExtensionToolHandlerInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tool/ExtensionToolHandlerInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tool/ToolCallContextDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tool/ToolCallDecisionDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tool/ToolCallDecisionKindEnum.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tool/ToolCallHookInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tool/ToolCallRewriteHookInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tool/ToolCallRewriteHookProviderInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tool/ToolCancellationTokenInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tool/ToolInvocationContextDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tool/ToolRegistrationDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tool/ToolResultContextDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tool/ToolResultDecisionDTO.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tool/ToolResultDecisionKindEnum.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tool/ToolResultHookInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tui/TuiExtensionContextInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tui/TuiExtensionInterface.php (100%)
 rename {src/CodingAgent/ExtensionApi => .hatfield/extensions/extension-api/src}/Tui/TuiProjectExtensionInterface.php (100%)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/release-split-extension-packages.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/release-split-extension-packages.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #362 was merged on GitHub as `2f7cc5968fefd39de76d90ea1f3f2fea018e3c17`; release `v0.0.7` completed successfully and all five package mirrors received `main` plus tag `v0.0.7`.

## Task workflow update - 2026-08-05T01:46:55.498Z
- Validation: Post-merge `LLM_MODE=true timeout 200s castor check`: PASS — all seven lanes, artifact integrity, leak check, and llama-proxy cache guard green.; Initial post-merge check exposed stale root vendor autoload after ExtensionApi moved to a path package. `castor composer -- install` did not support the project manifest, so bounded raw `composer install --no-interaction --no-progress` was used as the documented Castor-failure diagnostic exception; it installed the locked path package, after which the full gate passed.
