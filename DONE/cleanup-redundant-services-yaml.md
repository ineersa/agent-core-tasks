# Remove redundant services.yaml wiring

## Goal
Audit `config/services.yaml` and delete explicit service declarations already covered unchanged by the existing resource discovery, autowiring, and autoconfiguration defaults. Preserve declarations that provide real behavior such as custom arguments/bindings, factories, aliases, tags, visibility, exclusions, or environment-specific wiring.

Explicitly out of scope:
- `config/reference.php` (generated; never edit manually)
- Composer requirements, including `symfony/expression-language` and `symfony/var-exporter`
- unrelated service/container refactors or new abstractions

## Acceptance criteria
- Redundant explicit service declarations are removed from `config/services.yaml`.
- Every remaining explicit declaration has behavior not supplied by default autowiring/autoconfiguration/resource discovery.
- Symfony container compilation succeeds with existing behavior preserved.
- Focused Castor validation passes; all QA/tooling commands use Castor.
- No changes to `config/reference.php` or Composer dependency declarations.

## Workflow metadata
Status: DONE
Branch: task/cleanup-redundant-services-yaml
Worktree: /home/ineersa/projects/agent-core-worktrees/cleanup-redundant-services-yaml
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/365
PR Status: merged
Started: 2026-08-06T01:32:22.209Z
Completed: 2026-08-06T03:19:49.094Z

## Work log
- Created: 2026-08-06T01:32:16.441Z

## Task workflow update - 2026-08-06T01:32:22.210Z
- Moved TODO → IN-PROGRESS.
- Created branch task/cleanup-redundant-services-yaml.
- Created worktree /home/ineersa/projects/agent-core-worktrees/cleanup-redundant-services-yaml.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/cleanup-redundant-services-yaml.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/cleanup-redundant-services-yaml.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/cleanup-redundant-services-yaml.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/cleanup-redundant-services-yaml.
- Summary: Task created from user-approved ponytail audit follow-up. Scope is limited to redundant services.yaml wiring; generated config/reference.php and Composer requirements are explicitly excluded.

## Task workflow update - 2026-08-06T01:42:57.388Z
- Validation: Fork confirmed it read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`.; `castor test --filter='AgentCommandPromptTemplatesOptionsTest|ToolQuestionStoreTest|DeferredSubagentRunControlHandlersWiringTest|KernelCacheIsolationTest|CommittedRunEventAppenderLiveProgressIntegrationTest|RunResultMessagesRouteToRunControlTest'` — OK (23 tests, 94 assertions).; `castor cs-check` skipped because php-cs-fixer does not include config/services.yaml; no PHP changed.; `castor check` intentionally not run during task-start; reserved for task-to-pr gate.
- Summary: Implementation committed as cd34a35624ceaad0a82a6507fa3ff9b731263f14. Removed 34 redundant explicit service declarations from config/services.yaml; resource discovery, autowiring/autoconfiguration, `_instanceof` tags, and all behavior-bearing aliases/factories/arguments/custom tags remain. Only config/services.yaml changed (3 insertions, 99 deletions). Worktree is clean. Generated config/reference.php and Composer requirements were not touched.

## Task workflow update - 2026-08-06T01:57:02.409Z
- Summary: User clarified that this task covers the full ponytail-audit cleanup, not only services.yaml. Keep the committed services.yaml cleanup, then verify and implement all remaining defensible audit findings. Explicit exclusions remain: never hand-edit/delete generated config/reference.php; do not remove required symfony/expression-language or symfony/var-exporter. Candidates must be re-verified before deletion because the initial audit produced false positives.
- Scope expanded by explicit user clarification: full verified ponytail-audit cleanup plus services.yaml cleanup. Do not blindly apply scout findings; confirm references, runtime/build/package intent, and public API compatibility first.

## Task workflow update - 2026-08-06T02:07:26.092Z
- Summary: Re-audit completed with bounded read-only scouts. Additional verified cuts: remove AgentArtifactPathResolver forwarding facade in favor of SessionAgentArtifactPathResolver; remove unused LogContext forwarding alias and wrapper-only test; remove duplicate file-rewind null guard; remove unused latest_file_mtime Castor helper; remove unread HATFIELD_LLM_WORKER_COUNT smoke env entry; delete 11 unreferenced test fixtures. Rejected/kept: ForkExecutionServiceInterface and in-memory stores are useful test/runtime seams; CompletionState/Footer APIs and extension contracts are public surfaces; ext-tokenizer is required at runtime; stale POC doc is already absent.

## Task workflow update - 2026-08-06T02:23:45.965Z
- Validation: Focused artifact/TUI/logging tests: `castor test --filter='SessionAgentArtifactPathResolverTest|AgentArtifactRegistryTest|AgentArtifactRetrievalServiceTest|AgentArtifactSessionListingTest|AgentChildRunDirectoryTest|AgentChildRunEventStoreTest|AgentChildRunStoreTest|ChildRunArtifactLifecycleServiceTest|SubagentChildProgressSummaryBuilderTest|SessionCacheInspectCommandTest|SessionToolBatchStoreTest|TuiFileRewindPickerExtensionVirtualTest|LogContextProcessorTest|RunLogContextTest'` — OK (139 tests, 452 assertions).; Additional modified deferred-batch tests: `castor test --filter='DeferredSubagentBatchLaunchTest|DeferredSubagentBatchRecoveryTest'` — OK (9 tests, 107 assertions).; `castor phpstan` — 0 errors.; `castor cs-check` — passed after targeted Castor cs-fix.; Independent scout diff audit verified equivalent DI/resource behavior, canonical resolver rewrites, no remaining consumers of deleted artifacts, equivalent TUI guard deletion, and no public API removals.; `castor check` intentionally deferred to task-to-pr per workflow.
- Summary: Full verified ponytail-audit implementation is complete on commits cd34a3562 and ff460f038. Total branch diff: 39 files, 58 insertions, 560 deletions. Includes services.yaml cleanup, removal of the artifact-path facade and unused LogContext alias, duplicate TUI guard deletion, dead Castor helper/env deletion, and 11 unreferenced fixture deletions. Explicitly preserved generated config/reference.php, all Composer requirements, public extension/TUI APIs, useful interfaces, and in-memory test stores. Independent read-only scout review found no code blockers or remaining consumers; worktree is clean.
- Process note: the implementation fork used temporary Python rewrite helpers despite an explicit no-ad-hoc-scripts instruction. They were deleted before commit. The complete diff was independently re-audited read-only, all rewritten call sites were verified, omitted modified test classes received a separate exact Castor run, and the worktree is clean.

## Task workflow update - 2026-08-06T03:02:30.254Z
- Validation: Reviewer APPROVED full branch under specification-fidelity and ponytail review; no code, API, compatibility, or complexity blockers.; `castor test` — OK (4443 tests, 16631 assertions).; `castor deptrac` — 0 violations, 0 errors.; `castor phpstan` — 0 errors.; `castor cs-check` — 0 files requiring fixes.; `castor test:tui` — OK (34 tests, 222 assertions).; Focused blocker regression: `castor test --filter=TuiSkillReadCardVirtualRenderTest` — OK (1 test, 22 assertions).
- Summary: Task-to-PR review APPROVED after commit fdeb99fcd fixed a long-worktree-only VirtualTerminal path-wrap assertion exposed by full validation. Branch commits: cd34a3562, ff460f038, fdeb99fcd. Worktree clean.

## Task workflow update - 2026-08-06T03:04:36.063Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (114.4s).
- Pushed task/cleanup-redundant-services-yaml to origin.
- branch 'task/cleanup-redundant-services-yaml' set up to track 'origin/task/cleanup-redundant-services-yaml'.
- Created PR: https://github.com/ineersa/agent-core/pull/365

## Task workflow update - 2026-08-06T03:19:49.094Z
- Moved CODE-REVIEW → DONE.
- Merged task/cleanup-redundant-services-yaml into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/distribution.php                           |   2 -
 .castor/helpers.php                                |  38 --------
 .../file-rewind/src/FileRewindPickerController.php |   3 -
 config/services.yaml                               | 102 +--------------------
 .../Agent/Artifact/AgentArtifactPathResolver.php   |  65 -------------
 .../Agent/Artifact/AgentArtifactPathsDTO.php       |   2 +-
 .../Agent/Artifact/AgentArtifactRegistry.php       |  19 ++--
 .../Agent/Artifact/AgentChildRunEventStore.php     |   7 +-
 .../Artifact/AgentChildRunEventStoreFactory.php    |   3 +-
 .../Agent/Artifact/AgentChildRunStore.php          |   9 +-
 .../Agent/Artifact/AgentChildRunStoreFactory.php   |   3 +-
 .../ChildAwareToolBatchRunStoragePaths.php         |   3 +-
 src/CodingAgent/Logging/LogContext.php             |  69 --------------
 .../Session/SessionAgentArtifactPathResolver.php   |   4 +-
 .../apply-extension-cancel-safe-command.json       |  13 ---
 .../Schema/commands/apply-extension-command.json   |  13 ---
 .../Schema/commands/apply-steer-command.json       |  16 ----
 .../Fixtures/Schema/commands/start-run.json        |  26 ------
 .../Schema/events/ext-compaction-start.json        |  12 ---
 .../AgentCore/Fixtures/Schema/events/run-end.json  |  11 ---
 .../Fixtures/Schema/events/tool-execution-end.json |  14 ---
 .../Fixtures/Schema/events/turn-start.json         |   9 --
 .../Schema/execution/execute-llm-step.json         |  10 --
 .../Fixtures/Schema/execution/llm-step-result.json |  27 ------
 .../Agent/Artifact/AgentArtifactRegistryTest.php   |   5 +-
 .../Artifact/AgentArtifactRetrievalServiceTest.php |   3 +-
 .../Artifact/AgentArtifactSessionListingTest.php   |   3 +-
 .../Agent/Artifact/AgentChildRunDirectoryTest.php  |   5 +-
 .../Agent/Artifact/AgentChildRunEventStoreTest.php |   5 +-
 .../Agent/Artifact/AgentChildRunStoreTest.php      |   5 +-
 .../ChildRunArtifactLifecycleServiceTest.php       |   5 +-
 .../Launch/DeferredSubagentBatchLaunchTest.php     |   4 +-
 .../Recovery/DeferredSubagentBatchRecoveryTest.php |   3 +-
 .../SubagentChildProgressSummaryBuilderTest.php    |  11 +--
 .../CLI/Session/SessionCacheInspectCommandTest.php |   7 +-
 tests/CodingAgent/Logging/LogContextTest.php       |  57 ------------
 .../SessionAgentArtifactPathResolverTest.php       |   7 +-
 .../Session/SessionToolBatchStoreTest.php          |   5 +-
 .../fixtures/tui-compaction-summary-response.json  |  13 ---
 .../Screen/TuiSkillReadCardVirtualRenderTest.php   |   2 +-
 40 files changed, 59 insertions(+), 561 deletions(-)
 delete mode 100644 src/CodingAgent/Agent/Artifact/AgentArtifactPathResolver.php
 delete mode 100644 src/CodingAgent/Logging/LogContext.php
 delete mode 100644 tests/AgentCore/Fixtures/Schema/commands/apply-extension-cancel-safe-command.json
 delete mode 100644 tests/AgentCore/Fixtures/Schema/commands/apply-extension-command.json
 delete mode 100644 tests/AgentCore/Fixtures/Schema/commands/apply-steer-command.json
 delete mode 100644 tests/AgentCore/Fixtures/Schema/commands/start-run.json
 delete mode 100644 tests/AgentCore/Fixtures/Schema/events/ext-compaction-start.json
 delete mode 100644 tests/AgentCore/Fixtures/Schema/events/run-end.json
 delete mode 100644 tests/AgentCore/Fixtures/Schema/events/tool-execution-end.json
 delete mode 100644 tests/AgentCore/Fixtures/Schema/events/turn-start.json
 delete mode 100644 tests/AgentCore/Fixtures/Schema/execution/execute-llm-step.json
 delete mode 100644 tests/AgentCore/Fixtures/Schema/execution/llm-step-result.json
 delete mode 100644 tests/CodingAgent/Logging/LogContextTest.php
 delete mode 100644 tests/Tui/E2E/fixtures/tui-compaction-summary-response.json
- Removed worktree /home/ineersa/projects/agent-core-worktrees/cleanup-redundant-services-yaml.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/cleanup-redundant-services-yaml.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #365 confirmed merged on GitHub at 2026-08-06T03:18:34Z (merge commit 4b4f32906c217c56d94e63e7c36e4c03ea2a58a0). Proceeding with DONE integration and worktree cleanup.

## Task workflow update - 2026-08-06T03:23:10.259Z
- Validation: First post-merge `LLM_MODE=true castor check` attempt correctly failed closed after 60s waiting on another sibling-worktree check lock; holder process was not signaled and subsequently exited.; Post-merge retry `LLM_MODE=true castor check` — quality OK. deptrac OK; test OK (4443 tests, 16631 assertions); controller replay OK (12 tests, 165 assertions); TUI OK (34 tests, 218 assertions); llm-real OK (13 tests, 144 assertions); phpstan OK; cs-check OK; QA artifact integrity/leak/cache checks OK; llama-proxy entries stable 224→224.; Final integration `git status --short` clean; task worktree confirmed removed.
- Summary: DONE integration complete. PR #365 was merged, task branch integrated into main, remote changes pulled, task worktree removed, IDEA exclusions cleaned, and integration checkout is clean at 5f66532ec.
