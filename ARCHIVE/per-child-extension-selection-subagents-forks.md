# Add per-subagent and fork extension allowlists

## Goal
## Problem

Hatfield extensions are currently loaded process-wide from global `extensions.enabled`. Extension registrations do not retain their owning extension, so every globally enabled tool, prompt contributor, tool-call hook, rewrite hook, and after-turn hook can affect every parent and child run. There is no subagent-frontmatter or fork-settings extension scope.

In this project, all four configured native extensions are globally enabled, so subagents and forks currently inherit SafeGuard, TaskWorkflow, FileRewind, and CastorLlmMode whether they need them or not.

## Desired behavior

Add explicit per-child extension selection:

### Subagents

Agent definition frontmatter accepts an extension allowlist:

```yaml
---
name: scout
extensions:
  - Ineersa\CodingAgent\Extension\Builtin\SafeGuard\SafeGuardExtension
  # optional agent-specific extensions
---
```

SafeGuard is mandatory for every subagent. It is included in the effective allowlist even if omitted and cannot be disabled by an agent definition.

### Forks

Hatfield settings accept a fork extension allowlist. This project configures forks with SafeGuard and the project-local Castor LLM mode extension:

```yaml
forks:
  extensions:
    - Ineersa\CodingAgent\Extension\Builtin\SafeGuard\SafeGuardExtension
    - Ineersa\HatfieldExt\CastorLlmMode\CastorLlmModeExtension
```

SafeGuard is mandatory for forks. Castor is a project choice in `.hatfield/settings.yaml`, not a shipped global default, because its extension class is project-local.

## Current architecture

Relevant areas:

- `AgentFrontmatterDTO`, `AgentDefinitionParser`, `AgentDefinitionDTO`
- `SubagentLaunchPreparationService` and `SubagentChildLaunchInputFactory`
- `ForksConfigDTO`, `ForkExecutionService`, `ForkChildLaunchInputBuilder`
- `RunMetadata` and child metadata readers
- `ExtensionManager`, `ExtensionToolRegistryBridge`, `ExtensionHookRegistry`, and extension event subscribers

The main implementation challenge is not YAML parsing. Extension registrations currently lack owner identity and workers use one process-global registry. The effective child allowlist must survive asynchronous Messenger execution and filter every run-relevant extension surface. Keep ownership/filtering internal to CodingAgent; do not add Hatfield-specific runtime dependencies to the public `ExtensionApi` compatibility surface.

Canonical classes in this checkout:

- SafeGuard: `Ineersa\CodingAgent\Extension\Builtin\SafeGuard\SafeGuardExtension`
- Castor LLM mode: `Ineersa\HatfieldExt\CastorLlmMode\CastorLlmModeExtension`

The Castor extension is distinct from the Castor task runner and the `castor` agent skill. It rewrites Castor-related `bash` commands before SafeGuard evaluates them.

## Acceptance criteria
- Add one documented `extensions` list field to subagent definition frontmatter with strict parsing and validation; do not add aliases or compatibility formats.
- Resolve each subagent to an explicit effective extension allowlist. SafeGuard is always present and cannot be removed or disabled by frontmatter.
- Add a documented `forks.extensions` allowlist to Hatfield settings and typed configuration.
- Configure this project’s forks with exactly SafeGuard and `CastorLlmModeExtension` unless another extension is explicitly added later; do not implicitly leak every globally enabled optional extension into forks.
- SafeGuard is always present in a fork’s effective allowlist even if project/user fork settings omit it.
- Persist the effective extension allowlist in child run metadata so controller and Messenger worker processes enforce the same selection across retries and resume/replay paths.
- Associate each internal extension registration with its owning extension and enforce the run allowlist across extension-provided tools, prompt contributors, tool-call policy hooks, rewrite hooks, result hooks, and after-turn hooks. Unselected extensions must have no child-run side effects.
- Keep per-run extension selection inside CodingAgent internals. Do not add agent/runtime dependencies or required breaking methods to `ExtensionApiInterface`, `HatfieldExtensionInterface`, or public registration DTOs.
- Fail closed with a clear diagnostic if mandatory SafeGuard is unavailable or fails to register for a child run. Optional unavailable extensions produce an explicit configuration/launch diagnostic rather than being silently treated as active.
- Preserve hook ordering so the selected Castor rewrite executes before selected SafeGuard policy evaluation in forks.
- Define and document omitted-frontmatter semantics as SafeGuard-only for subagents; optional extensions must be explicitly selected. Parent/global extension behavior remains unchanged.
- Update `docs/agents.md`, `docs/settings.md`, project `.hatfield/settings.yaml`, and relevant example/default comments to describe child allowlists and mandatory SafeGuard behavior.
- Add focused proof for frontmatter/settings validation, effective allowlist resolution, SafeGuard enforcement, optional-extension isolation, metadata propagation, and selected Castor→SafeGuard execution. Add controller-replay coverage for the asynchronous child worker path; no TUI proof is needed unless visible TUI behavior changes.
- Because this changes runtime/Messenger child execution and a safety boundary, run focused Castor validation and mandatory `castor check` before CODE-REVIEW.

## Workflow metadata
Status: ARCHIVE
Branch: task/per-child-extension-selection-subagents-forks
Worktree: /home/ineersa/projects/agent-core-worktrees/per-child-extension-selection-subagents-forks
Fork run: 459gu3af80l6
PR URL: https://github.com/ineersa/agent-core/pull/341
PR Status: merged
Started: 2026-07-30T21:35:51.358Z
Completed: 2026-07-31T19:49:27.077Z

## Work log
- Created: 2026-07-21T20:26:02.971Z

## Task workflow update - 2026-07-21T20:45:42.737Z
- Summary: User revised mandatory SafeGuard semantics: SafeGuard remains enabled by default and is automatically added independently of the `extensions` allowlist, but an explicit separate `noSafeguard`/`no_safeguard` option may opt a child out. This clarification supersedes acceptance text saying SafeGuard can never be disabled.
- Authoritative option shape: subagent frontmatter supports `noSafeguard: true`; Hatfield fork settings support `forks.no_safeguard: true` following settings snake_case conventions. Both default to false.
- `extensions` remains the optional-extension allowlist; SafeGuard is added separately unless the explicit opt-out is true. Merely omitting SafeGuard from `extensions` must not disable it.
- Reject contradictory configuration where `noSafeguard: true`/`no_safeguard: true` is combined with an explicit SafeGuard entry, rather than silently choosing precedence.
- Persist the effective SafeGuard-enabled/disabled decision in child run metadata alongside the extension allowlist so asynchronous workers enforce the same policy. The opt-out is configuration-only, not a model-selectable fork/subagent tool argument.

## Task workflow update - 2026-07-21T20:48:22.689Z
- Summary: User replaced the `noSafeguard` design with declarative always-on extension lists. This clarification supersedes both the earlier non-disableable SafeGuard rule and the later `noSafeguard`/`no_safeguard` option. Children must not inherit globally enabled optional extensions when no child selection is supplied.
- Authoritative effective-selection model: subagent extensions = `agents.extensions.always_on` union the agent definition frontmatter `extensions` list. Fork extensions = `forks.extensions.always_on` union `forks.extensions.enabled`. Deduplicate while preserving deterministic order.
- Use snake_case in Hatfield YAML settings: `agents.extensions.always_on`, `forks.extensions.always_on`, and `forks.extensions.enabled`. Agent frontmatter keeps the direct `extensions` list.
- Configure SafeGuard in both always-on lists. Configure `CastorLlmModeExtension` in this project’s `forks.extensions.enabled` list.
- If subagent frontmatter omits `extensions`, it enables no optional extensions; only `agents.extensions.always_on` applies. If fork optional `enabled` is omitted/empty, only `forks.extensions.always_on` applies. Neither path inherits optional entries from global `extensions.enabled`.
- Remove `noSafeguard` and `forks.no_safeguard` from the design. To change always-on policy, users edit the corresponding `always_on` list explicitly; there is no special SafeGuard-shaped configuration key.
- Keep global `extensions.enabled` as the set available to the parent/process, but child run filtering uses only the effective child selection. Validate that child-selected/always-on classes are available/registered and report clear configuration errors otherwise.

## Task workflow update - 2026-07-30T21:26:32.355Z
- Summary: Reconfirmed during OM-05 manual fork testing: subagents and forks currently inherit all globally enabled extensions because only tool exclusion exists. This existing task is the correct follow-up; do not create a duplicate. OM provides the concrete isolation proof: when enabled for the parent, ObservationalMemoryExtension must remain absent from child effective allowlists, register no child hooks/tools/jobs, and write no child OM state. Parent-side OM projection used to prepare inherited fork context is separate and must not imply loading OM in the child.
- 2026-07-30 — Docs audit confirmed no excluded-extensions setting exists today. Existing authoritative design remains `agents.extensions.always_on + agent frontmatter extensions` and `forks.extensions.always_on + forks.extensions.enabled`, with project forks configured SafeGuard + CastorLlmMode only. Add OM non-registration/non-write proof to the implementation validation.

## Task workflow update - 2026-07-30T21:35:51.358Z
- Moved TODO → IN-PROGRESS.
- Created branch task/per-child-extension-selection-subagents-forks.
- Created worktree /home/ineersa/projects/agent-core-worktrees/per-child-extension-selection-subagents-forks.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/per-child-extension-selection-subagents-forks.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/per-child-extension-selection-subagents-forks.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/per-child-extension-selection-subagents-forks.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/per-child-extension-selection-subagents-forks.
- Summary: Claimed for implementation. Authoritative scope uses agents.extensions.always_on + agent frontmatter extensions, and forks.extensions.always_on + forks.extensions.enabled; no noSafeguard option.

## Task workflow update - 2026-07-30T21:43:57.038Z
- Summary: Three read-only scouts traced subagent config/launch metadata, fork config/launch, and all extension registration/dispatch surfaces. Durable seam is RunMetadata persisted in run_started; owner capture is centralized at ExtensionManager→ExtensionToolRegistryBridge. Existing RegistryBackedToolbox ordering already runs Castor rewrite before SafeGuard policy. No TUI behavior changes are required; controller-replay is the correct async proof layer.
- Specification fidelity: latest task update is authoritative. Implement only agents.extensions.always_on + frontmatter extensions and forks.extensions.always_on + forks.extensions.enabled, stable first-seen dedup; do not add noSafeguard aliases or inherit global optional extensions.
- Scouts read the task, testing skill, and tests/AGENTS.md. Identified OM isolation contract as child-invisible hooks/tools/jobs and no child OM writes even when OM is parent-enabled.

## Task workflow update - 2026-07-30T21:44:31.043Z
- Recorded fork run: s9f5r0mx1lsz
- Summary: Implementation fork launched in task worktree with exact authoritative selection model, internal ownership/filtering boundaries, OM isolation proof, controller-replay async proof, focused Castor validation, and commit required.

## Task workflow update - 2026-07-30T21:58:33.719Z
- Recorded fork run: s9f5r0mx1lsz
- Validation: git status --short: dirty; 36 tracked modifications and 8 untracked files; git log: no task commit; HEAD still 74f9c79fb; git diff --check: passed
- Summary: First implementation fork ended with a corrupted handoff and no commit. Verification found the task worktree dirty with 36 tracked files modified plus 8 untracked files; HEAD remained at the task base. Diff check was clean, but output is unacceptable until a fork completes implementation, runs focused Castor validation, and commits.

## Task workflow update - 2026-07-30T21:59:03.217Z
- Recorded fork run: vwk13kl1skvq
- Summary: Recovery fork launched to audit and finish the prior dirty implementation, resolve incomplete ExtensionManager/selection behavior, run focused Castor validation, and commit a clean task branch. No reset/discard authorized.

## Task workflow update - 2026-07-30T22:09:01.015Z
- Recorded fork run: vwk13kl1skvq
- Validation: Focused sequential report: OK (35 tests, 187 assertions); castor test ParaTest report: ERROR, 4 errors / 1266 tests; generated container requires PlatformInterface proxy from a different isolated var/tmp/hatfield-test-* root; No task commit; HEAD remains 74f9c79fb
- Summary: Second recovery fork also ended without a commit and returned a truncated handoff. It added constructor/test-fixture fixes and got 35 focused tests green, but full `castor test` failed 4/1266 tests due ParaTest compiled-container proxy paths crossing isolated test roots. Worktree remains dirty at base HEAD. A final narrow fork is required to fix the cache/isolation root cause, run required focused validation, and commit.

## Task workflow update - 2026-07-30T22:09:38.110Z
- Recorded fork run: xpt7uxjj3czp
- Summary: Final narrow recovery fork launched with exact ParaTest container-proxy crossover evidence. Scope is root-cause isolation fix, required focused Castor validation, staging all intended files, and committing; no feature expansion or gate/PR work.

## Task workflow update - 2026-07-30T22:33:32.546Z
- Recorded fork run: xpt7uxjj3czp
- Validation: castor test focused: OK 27 tests / 135 assertions (fork report); castor test: OK 4381 tests / 15996 assertions (fork report); castor test:controller-replay: OK 10 tests / 139 assertions (fork report); castor deptrac: violations=0; castor phpstan: errors=0; castor cs-check: clean (fork report); Verified HEAD 1c5322dd8 and clean status
- Summary: Recovery implementation committed cleanly at 1c5322dd8c6019f8b74ebe7b4fd36e081f387d64. Verified clean worktree, expected child-selection/OM/controller-replay files, and 55-file diff. Focused/full/controller-replay/Deptrac/PHPStan/CS validations reported green. One boundary issue remains before accepting task-start: the commit changed two public ExtensionApi provider interface method signatures to add owner-filter parameters; ownership/filtering must remain CodingAgent-internal and existing external implementors would be signature-incompatible. Launching a narrow follow-up fork to restore the public signatures and use an internal provider seam.

## Task workflow update - 2026-07-30T22:33:51.567Z
- Recorded fork run: mgmotmvycfk4
- Summary: Narrow compatibility fork launched to restore unchanged public ExtensionApi provider signatures and move owner-filtering behind a CodingAgent-internal seam, preserving behavior and prior green implementation.

## Task workflow update - 2026-07-30T22:40:45.010Z
- Recorded fork run: mgmotmvycfk4
- Validation: castor test focused feature pass: OK 27 tests / 135 assertions; castor test full feature pass: OK 4381 tests / 15996 assertions; castor test:controller-replay after compatibility fix: OK 10 tests / 139 assertions; castor test focused compatibility pass: OK 58 tests / 159 assertions; castor deptrac: violations=0, errors=0; castor phpstan: errors=0, file_errors=0; castor cs-check: clean; Verified HEAD 3b3211021 and clean git status; public prompt/rewrite provider signatures restored
- Summary: Implementation complete in clean task worktree at HEAD 3b3211021c7130ed8ebfcdf43323f4b4ff1b865d (feature commit 1c5322dd8 + compatibility commit 3b3211021). Verified both commits exist, worktree is clean, required OM/controller-replay proofs are present, and public ExtensionApi provider signatures are restored unchanged. Effective child extension allowlists are typed, persisted in run metadata, and enforced across owned tools/prompts/hooks/jobs without global optional leakage. No TUI change/proof was needed.

## Task workflow update - 2026-07-30T23:11:49.422Z
- Validation: Reviewer: APPROVED; castor test: OK 4381 tests / 15996 assertions; castor test:controller-replay: OK 10 tests / 139 assertions; castor deptrac: violations=0, errors=0; castor phpstan: errors=0, file_errors=0; castor cs-check: files_fixed=0; git status --short: clean
- Summary: CODE-REVIEW preparation complete. Reviewer verdict: APPROVED after reviewing all 53 changed files against latest task clarifications and specification fidelity gate. Reviewer verified strict config/frontmatter semantics, stable effective allowlists, durable metadata, owner filtering across required surfaces, OM isolation, Castor→SafeGuard ordering, async controller-replay proof, unchanged public ExtensionApi signatures, and no unmapped external surface. HEAD 3b3211021c7130ed8ebfcdf43323f4b4ff1b865d; worktree clean.

## Task workflow update - 2026-07-30T23:13:48.787Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (103.5s).
- Pushed task/per-child-extension-selection-subagents-forks to origin.
- branch 'task/per-child-extension-selection-subagents-forks' set up to track 'origin/task/per-child-extension-selection-subagents-forks'.
- Created PR: https://github.com/ineersa/agent-core/pull/341

## Task workflow update - 2026-07-30T23:13:57.136Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/341
- Updated PR Status: open
- Validation: move_task deterministic castor check: passed in 103.5s; Pushed task/per-child-extension-selection-subagents-forks; PR: https://github.com/ineersa/agent-core/pull/341
- Summary: Deterministic gate passed; branch pushed and PR #341 created.

## Task workflow update - 2026-07-31T18:54:26.637Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Reopened for a minimal follow-up discovered during manual session-1 validation: CastorLlmMode rewrite executes, but its environment-assignment regex falsely treats quoted diagnostic text such as `LLM_MODE=%s` as an existing assignment and skips the export. Fix the parser narrowly and add regression coverage before updating PR #341.

## Task workflow update - 2026-07-31T18:54:43.712Z
- Recorded fork run: 459gu3af80l6
- Summary: Launched narrow implementation fork to fix CastorLlmMode assignment false positives, add regression coverage, run focused Castor validation, and commit.

## Task workflow update - 2026-07-31T19:00:32.498Z
- Recorded fork run: 459gu3af80l6
- Validation: castor test --filter=CastorCommandRewriterTest: OK 17 tests / 17 assertions; castor cs-check: clean; Reviewer for commit 6eeb2620c: APPROVED; Changed only CastorCommandRewriter.php and CastorCommandRewriterTest.php
- Summary: Manual session-1 follow-up fixed and committed at 6eeb2620c242b40a35352a565c4633b893a85709. CastorLlmMode assignment detection no longer mistakes the exact quoted `printf 'LLM_MODE=%s'` diagnostic labels for shell assignments; genuine assignment cases remain covered. Narrow reviewer verdict: APPROVED. Worktree clean.

## Task workflow update - 2026-07-31T19:02:26.064Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (103.5s).
- Pushed task/per-child-extension-selection-subagents-forks to origin.
- branch 'task/per-child-extension-selection-subagents-forks' set up to track 'origin/task/per-child-extension-selection-subagents-forks'.
- PR already exists: https://github.com/ineersa/agent-core/pull/341

## Task workflow update - 2026-07-31T19:02:30.806Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/341
- Updated PR Status: open
- Validation: Follow-up commit: 6eeb2620c242b40a35352a565c4633b893a85709; move_task deterministic castor check: passed in 103.5s; Existing PR updated: https://github.com/ineersa/agent-core/pull/341
- Summary: CastorLlmMode follow-up pushed to existing PR #341; deterministic gate passed. Task is good to review/merge.

## Task workflow update - 2026-07-31T19:49:27.077Z
- Moved CODE-REVIEW → DONE.
- Merged task/per-child-extension-selection-subagents-forks into integration checkout.
- Auto-merging .hatfield/settings.yaml
Auto-merging config/services.yaml
Auto-merging docs/settings.md
Auto-merging src/CodingAgent/Extension/ExtensionCompactionHookDispatcher.php
Auto-merging src/CodingAgent/Extension/ExtensionToolRegistryBridge.php
Auto-merging tests/CodingAgent/Extension/ExtensionToolRegistryBridgeTest.php
Merge made by the 'ort' strategy.
 .../castor-llm-mode/src/CastorCommandRewriter.php  |  10 +-
 .../tests/CastorCommandRewriterTest.php            |  19 ++
 .hatfield/settings.yaml                            |   4 +
 config/hatfield.defaults.yaml                      |  15 ++
 config/services.yaml                               |  29 ++-
 depfile.yaml                                       |   3 +
 docs/agents.md                                     |   4 +
 docs/settings.md                                   |  26 ++
 src/AgentCore/Domain/Run/RunMetadata.php           |   2 +
 .../Agent/ChildExtensionSelectionService.php       | 121 +++++++++
 .../Agent/Definition/AgentDefinitionDTO.php        |   4 +-
 .../Agent/Definition/AgentDefinitionParser.php     |   1 +
 .../Agent/Definition/AgentFrontmatterDTO.php       |  24 +-
 .../Agent/Execution/AgentPromptBuilder.php         |  11 +-
 .../SubagentChildLaunchInputFactory.php            |  40 +++
 .../Agent/Execution/SubagentRunMetadataReader.php  |  43 +++-
 .../Agent/Execution/SubagentToolSetResolver.php    |  28 +-
 .../Agent/Fork/ForkChildLaunchInputBuilder.php     |  42 ++-
 .../Agent/Fork/ForkChildMessageComposer.php        |  11 +-
 src/CodingAgent/Config/AgentsConfig.php            |   4 +
 src/CodingAgent/Config/AppConfig.php               |   7 +-
 .../Config/ChildExtensionsConfigDTO.php            |  72 ++++++
 src/CodingAgent/Config/ForksConfigDTO.php          |  27 +-
 .../Extension/Agent/ExtensionAgentJobRegistry.php  |  15 +-
 .../Extension/Agent/ExtensionAgentJobWorker.php    |  58 +++++
 .../ChildRunExtensionAllowlistReaderInterface.php  |  22 ++
 .../ExtensionAfterTurnCommitHookSubscriber.php     |   5 +-
 .../ExtensionCompactionHookDispatcher.php          |   5 +-
 .../Extension/ExtensionHookRegistry.php            | 131 ++++++----
 src/CodingAgent/Extension/ExtensionManager.php     |  31 ++-
 .../Extension/ExtensionRegistrationContext.php     |  43 ++++
 .../Extension/ExtensionToolHookEventSubscriber.php |  22 +-
 .../Extension/ExtensionToolRegistryBridge.php      |   1 +
 .../SystemPrompt/SystemPromptBuilder.php           |  24 +-
 src/CodingAgent/Tool/RegistryBackedToolbox.php     |  30 ++-
 src/CodingAgent/Tool/ToolDefinitionDTO.php         |   2 +
 src/CodingAgent/Tool/ToolRegistry.php              |   2 +
 src/CodingAgent/Tool/ToolRegistryInterface.php     |   2 +
 .../Agent/Definition/AgentDefinitionParserTest.php |  54 +++-
 ...05BareAgentsEffectiveContextIntegrationTest.php |   2 +
 ...AppendStructuralPermanentSubsetContractTest.php |  58 ++---
 .../Launch/DeferredSubagentBatchLaunchTest.php     |   5 +-
 .../SubagentChildExtensionMetadataTest.php         | 101 ++++++++
 .../Execution/SubagentExecutionServiceTest.php     |   2 +
 .../SubagentPromptUserContextContractTest.php      |   3 +
 .../Execution/SubagentToolSetResolverTest.php      |  19 +-
 .../Support/SubagentExecutionServiceFactory.php    |  27 +-
 tests/CodingAgent/Config/AgentsConfigTest.php      |  54 ++++
 .../Extension/ChildExtensionOmIsolationTest.php    | 151 +++++++++++
 .../ChildExtensionSelectionServiceTest.php         | 240 +++++++++++++++++
 .../Extension/ExtensionOwnerFilteringTest.php      | 249 ++++++++++++++++++
 .../Extension/ExtensionToolRegistryBridgeTest.php  |   2 +
 ...ChildExtensionSelectionControllerReplayTest.php | 284 +++++++++++++++++++++
 .../SystemPrompt/SystemPromptBuilderTest.php       |  36 +--
 .../CodingAgent/Tool/RegistryBackedToolboxTest.php |  34 +--
 55 files changed, 2070 insertions(+), 191 deletions(-)
 create mode 100644 src/CodingAgent/Agent/ChildExtensionSelectionService.php
 create mode 100644 src/CodingAgent/Config/ChildExtensionsConfigDTO.php
 create mode 100644 src/CodingAgent/Extension/ChildRunExtensionAllowlistReaderInterface.php
 create mode 100644 src/CodingAgent/Extension/ExtensionRegistrationContext.php
 create mode 100644 tests/CodingAgent/Agent/Execution/Subagent/ChildRun/Preparation/SubagentChildExtensionMetadataTest.php
 create mode 100644 tests/CodingAgent/Extension/ChildExtensionOmIsolationTest.php
 create mode 100644 tests/CodingAgent/Extension/ChildExtensionSelectionServiceTest.php
 create mode 100644 tests/CodingAgent/Extension/ExtensionOwnerFilteringTest.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/E2E/ChildExtensionSelectionControllerReplayTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/per-child-extension-selection-subagents-forks.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/per-child-extension-selection-subagents-forks.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #341 merged by user; Integration checkout clean on main before DONE transition
- Summary: User confirmed PR #341 merged. Completing task and syncing integration checkout.

## Task workflow update - 2026-07-31T19:49:43.689Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/341
- Updated PR Status: merged
- Validation: DONE merge completed successfully; Integration checkout git status: clean; origin/main contains PR #341 merge commit 41d9a014f
- Summary: Task completed. Integration checkout synced, task worktree removed, and IDEA exclusions cleaned up.

## Task workflow update - 2026-08-06T20:59:22.895Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
