# FORK-02 Pi-style fork context, prompt, artifact, and handoff contracts

## Goal
Implement the core fork contracts before wiring any process launch.

Reference docs/sources:
- Plan: `/home/ineersa/projects/agent-core/.aiassistant/fork/plan.md`
- Pi fork sanitization: `/home/ineersa/claw/my-pi/packages/extensions/extensions/fork/fork.ts` (`sanitizeForkSnapshotBranch`, `injectVirtualCompaction`, `buildForkSessionSnapshotJsonl`)
- Pi fork prompt: `/home/ineersa/claw/my-pi/packages/extensions/extensions/fork/runner.ts` (`buildForkTaskPrompt`)

Design decisions to preserve:
- Child starts from a sanitized copied/snapshotted parent session, not an empty prompt.
- Match Pi trim behavior: locate the last assistant `fork` tool call, then trim back through the preceding user message. The launch user message is removed too.
- Prefer Pi-style virtual snapshot compaction: sanitize first, then compact only the fork snapshot; do not mutate the parent session. OM folding can start minimal but the API should leave room for OM details.
- Child receives fork prompt as a system-prompt append plus one large generated user message containing fork instructions, handoff schema, and raw task under `Task:`.
- `fork_retrieve(run_id)` remains fork-specific model-facing API, backed by shared artifact/retrieval infrastructure later.
- Handoff schema/validator is mandatory foundation: final message is not accepted unless it matches the schema.

## Acceptance criteria
- Defines fork level/config DTOs/enums and result artifact contract (`handoff.md`, `metadata.json`, `events.jsonl`, `state.json`) without launching processes.
- Adds `ForkContextBuilder` (or equivalent) that performs Pi-style sanitization including removal of the launch user message and fork tool call/result.
- Adds virtual-compaction boundary/API for fork snapshots; parent session is never mutated.
- Ports/adapts Pi-style `buildForkTaskPrompt(task)` structure into Hatfield, with source comments pointing to Pi runner.ts.
- Adds `ForkHandoffValidator` or equivalent validating required handoff sections/schema used by the prompt.
- Focused Castor validation via Castor only, with tests for sanitization, prompt shape, artifact metadata shape, and handoff validator behavior.

## Workflow metadata
Status: DONE
Branch: task/fork-02-context-prompt-artifact-contract
Worktree: /home/ineersa/projects/agent-core-worktrees/fork-02-context-prompt-artifact-contract
Fork run: sug121jfnwad (impl) + fix forks (review fixes x2)
PR URL: https://github.com/ineersa/agent-core/pull/235
PR Status: merged
Started: 2026-06-29T17:30:30.146Z
Completed: 2026-06-29T19:48:42.879Z

## Work log
- Created: 2026-06-29T16:12:01.077Z

## Task workflow update - 2026-06-29T17:30:30.146Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fork-02-context-prompt-artifact-contract.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fork-02-context-prompt-artifact-contract.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fork-02-context-prompt-artifact-contract.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fork-02-context-prompt-artifact-contract.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fork-02-context-prompt-artifact-contract.

## Task workflow update - 2026-06-29T17:30:48.322Z
- Summary: IN-PROGRESS. Worktree: /home/ineersa/projects/agent-core-worktrees/fork-02-context-prompt-artifact-contract. Branch: task/fork-02-context-prompt-artifact-contract.

Scope confirmed (contracts only, NO process launch — that is FORK-03/04):
- Fork level/config DTOs+enums + result artifact contract (kind discriminator + ForkRunMetadataDTO).
- ForkContextBuilder: Pi-style sanitization (remove launch user msg + fork tool call/result) + virtual-compaction boundary API; parent never mutated.
- ForkTaskPromptBuilder: port Pi buildForkTaskPrompt(task) + FORK_CHILD_SYSTEM_PROMPT append.
- ForkHandoffValidator: required handoff sections + repair instruction.
- Castor unit tests (NOT a TUI task; NOT provider-integration -> no tmux/llm-real proof required).

Key evidence gathered:
- Assistant tool calls live in AgentMessage->metadata['tool_calls'] = [['id'=>..,'name'=>..,'arguments'=>..,'order_index'=>..],...]; user msg = role 'user'. See AgentMessageNormalizer + AgentMessageToolCallSequenceValidator::extractToolCallIds().
- Compaction boundary: CompactionBoundarySelector::findBoundary()/findSafeBoundary() operate on list<AgentMessage>; CompactionServiceInterface::prepare() in src/AgentCore/Contract/Compaction. Prior compaction summary is a user msg with metadata['compact_summary']=true.
- Artifact: AgentArtifactEntryDTO/PathsDTO/StatusEnum in src/CodingAgent/Agent/Artifact; AgentArtifactRegistry::create(parentRunId,artifactId,agentRunId,agentName) called at SubagentExecutionService.php:111 and :398. Paths via AgentArtifactPathsDTO::forArtifactId() already yields handoff.md/metadata.json/events.jsonl/state.json.
- Config DTO pattern: src/CodingAgent/Config/CompactionConfig.php (denormalizable, fromAppConfig, DEFAULT_* consts). AppConfig.php adds sections in fromContainer() via denormalizer->denormalize((array)($data['section']??[]), XConfig::class). Add `forks` section there.
- DTO/enum conventions: final readonly class + promoted public props + #[SerializedName('snake_case')] + #[Assert.*]; enums are `*Enum` suffixed string cases.
- Deptrac: AppAgent (src/CodingAgent/Agent/.*) may depend on AgentCore, AppTool, AppConfig, AppSession, Symfony Serializer/Validator/Lock. New code under src/CodingAgent/Agent/Fork/ is fine.
- Pi sources: /home/ineersa/claw/my-pi/packages/extensions/extensions/fork/{fork.ts,runner.ts} (sanitizeForkSnapshotBranch, injectVirtualCompaction, buildForkSessionSnapshotJsonl, buildForkTaskPrompt).
- Tests mirror src namespace under tests/CodingAgent/Agent/Fork/; pure unit tests extend PHPUnit TestCase; use TestDirectoryIsolation for tmp dirs.

## Task workflow update - 2026-06-29T17:53:59.290Z
- Recorded fork run: sug121jfnwad
- Updated PR Status: implemented (in worktree, not yet pushed)
- Validation: castor test --filter 'Fork|AgentArtifactKind' -> OK (63 tests, 208 assertions); castor test --filter 'AgentArtifactRegistry' -> OK (50 tests, 132 assertions) [existing tests, kind breaking change correctly threaded]; castor deptrac -> 0 violations, 0 errors (Config/ vs Agent/Fork/ layout decision sound); fork reported: castor phpstan 0 errors, castor cs-check clean; NOT run (out of scope): castor check, castor test:llm-real, TmuxHarness — this is a contracts-only task (no process launch, no TUI, no provider integration)
- Summary: FORK COMPLETE. Fork run sug121jfnwad implemented FORK-02 contracts and committed abce25f9f on branch task/fork-02-context-prompt-artifact-contract. Independently verified by orchestrator: 63 new fork tests + 50 existing artifact tests pass via Castor; deptrac clean.

Files (31 changed, +3016/-38): 14 new production + 5 modified + 8 new test + 2 modified test.

Delivered:
- Fork level/config: ForkLevelEnum, ForkLevelConfigDTO, ForksConfigDTO (under src/CodingAgent/Config/ — deptrac forbids AppConfig->AppAgent), ForkConfigResolver + ForkResolvedConfigDTO (Agent/Fork/). AppConfig.php wires `forks:` section via denormalizeForksConfig (handles levels sub-map).
- Result artifact contract: AgentArtifactKindEnum (subagent/fork); `kind` added as REQUIRED param to AgentArtifactEntryDTO (no backward-compat shim); AgentArtifactRegistry::create() updated; SubagentExecutionService call sites pass kind:Subagent. ForkRunMetadataDTO with serializer round-trip proven.
- ForkContextBuilder pipeline: ForkSnapshotSanitizer (ports Pi sanitizeForkSnapshotBranch — scans for last assistant fork tool_call, trims back to preceding user msg, never mutates input) -> ForkSnapshotCompactor (virtual compaction via CompactionBoundarySelector, carries forward prior compact_summary, no LLM, parent never mutated) -> ForkTaskPromptBuilder (ports Pi buildForkTaskPrompt 11-section handoff + FORK_CHILD system append) -> ForkSessionSnapshotDTO.
- ForkHandoffValidator: mandatory sections 1/5/11 + section-1 filesystem-changes statement; produces repairInstruction. ForkHandoffValidationResultDTO.
- Pluggable ForkSnapshotSummaryProviderInterface + DefaultForkSnapshotSummaryProvider (NOOP, carries forward prior summary only; LLM-backed provider deferred to FORK-05).

Surfaced decisions/notes:
- Virtual compaction v1 only carries forward a PRIOR compact_summary; no LLM summary (so uncompacted parent sessions won't be reduced — by design, LLM provider is later task).
- Validator marks sections 1/5/11 mandatory; 9/10 NOT mandatory (open decision if stricter validation desired — adjust ForkHandoffValidator::MANDATORY_SECTIONS).
- Out of scope (correctly excluded): process launch, tmux, fork/fork_retrieve tools, Messenger, parent mutation.
- Fork sug121jfnwad launched on worktree to implement contracts.
- Fork committed abce25f9f (31 files, +3016/-38).
- Orchestrator independently verified via Castor: 63 fork tests + 50 artifact tests pass; deptrac 0 violations.
- Implementation done; NOT pushed, NOT moved to CODE-REVIEW — awaiting task-to-pr.

## Task workflow update - 2026-06-29T18:32:02.734Z
- Recorded fork run: sug121jfnwad (impl) + fix forks (review fixes x2)
- Updated PR Status: ready for PR (moving to CODE-REVIEW)
- Validation: castor test (FULL UNFILTERED, on 3ed166735): 3845 tests, 12242 assertions OK — CRITICAL regression resolved; castor test --filter 'Fork|AgentArtifact' (on final HEAD 91d611046): 127 tests, 399 assertions OK; castor phpstan: 0 errors; castor cs-check: clean (after cs-fix); castor deptrac: 0 violations, 0 errors (Config/ vs Agent/Fork/ layout sound; ForkSessionSnapshotDTO->level import valid); independent orchestrator grep: ZERO remaining 4-arg create() calls in src/ or tests/; NOT run (out of scope for contracts-only task): castor check, castor test:tui, castor test:llm-real — no TUI/runtime/Messenger code and no live provider/LLM integration touched
- Summary: REVIEW CYCLE COMPLETE — ready for CODE-REVIEW.

Reviewer verdict progression: REQUEST CHANGES (on abce25f9f) -> APPROVE WITH SUGGESTIONS (on 3ed166735) -> all suggestions resolved (on 91d611046). Equivalent to APPROVED.

Branch: task/fork-02-context-prompt-artifact-contract. Two commits:
- abce25f9f — FORK-02 contracts (31 files, +3016/-38)
- 91d611046 — fix(FORK-02): address review findings (14 files, +118/-119, amended with docblock fix)

Review findings addressed (13 + 1 NIT):
- [CRITICAL] 18 missed AgentArtifactRegistry::create() 4-arg call sites in 3 test files (AgentArtifactRetrievalServiceTest x12, AgentArtifactChildRunDirectoryTest x5, AgentArtifactSessionListingTest x1) that broke the FULL unfiltered suite — masked by filtered runs + phpstan excluding tests/. All now pass AgentArtifactKindEnum::Subagent.
- [BUG] single-quoted '\n' literal in ForkSnapshotCompactor summary wrapper -> double-quoted <summary> tags matching SessionCompactor.
- [EDGE] ForkHandoffValidator section1 filesystem-statement regex loosened to accept singular + verb variants.
- [EDGE] documented no-summary v1 bail (LLM provider deferred to FORK-05).
- [DEAD CODE] collapsed pointless under-budget branch; removed dead -1 sentinel.
- [CONVENTION] AppConfig denormalizeForksConfig relocated so validateDefaultModel keeps its docblock.
- [TEST] ForkHandoffValidatorTest rewritten with real missing-section failure paths (1/5/11).
- [TEST] ForkSnapshotCompactorTest safe-boundary test no longer vacuous (leads with assertTrue compacted).
- [TEST] AgentArtifactKindEnumTest serializer now honors #[SerializedName] (ClassMetadataFactory + MetadataAwareNameConverter), asserts snake_case keys.
- [NTH] ForkRunMetadataDTO docblock notes kind lives on entry DTO not duplicated.
- [NTH] ForkSessionSnapshotDTO exposes resolved ForkLevelEnum level (avoids FORK-04 double-resolution).
- [NTH] renamed misleading resolver test.
- [NIT] stale findPriorCompactSummary docblock (removed -1 clause); SessionCompactor wrapper constants are private so left a pointing comment rather than expanding scope.
- Reviewer subagent on abce25f9f -> REQUEST CHANGES: 1 CRITICAL (18 missed create() call sites breaking full suite, masked by filtered runs) + BUG (single-quote \n) + edge/dead-code/convention/test findings.
- Fork applied all 13 fixes -> commit 3ed166735. Orchestrator independently confirmed: zero 4-arg create() remaining; full castor test 3845 OK.
- Reviewer re-run on 3ed166735 -> APPROVE WITH SUGGESTIONS; all 13 verified, only stale docblock NIT.
- Fork amended docblock fix -> commit 91d611046 (SessionCompactor constants private -> pointing comment, no scope creep).
- Final orchestrator validation on 91d611046: focused tests 127 OK, deptrac 0 violations, tree clean.
- Ready to move to CODE-REVIEW.

## Task workflow update - 2026-06-29T18:33:32.484Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (59.1s).
- Pushed task/fork-02-context-prompt-artifact-contract to origin.
- branch 'task/fork-02-context-prompt-artifact-contract' set up to track 'origin/task/fork-02-context-prompt-artifact-contract'.
- Created PR: https://github.com/ineersa/agent-core/pull/235

## Task workflow update - 2026-06-29T18:33:41.418Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/235
- Updated PR Status: open
- Validation: castor check (deterministic gate in worktree, run by move_task->CODE-REVIEW): PASSED (59.1s); castor test (full unfiltered): 3845 tests, 12242 assertions OK; castor test --filter 'Fork|AgentArtifact': 127 tests, 399 assertions OK; castor phpstan: 0 errors; castor cs-check: clean; castor deptrac: 0 violations; Branch pushed to origin; PR #235 created
- Moved to CODE-REVIEW: deterministic castor check PASSED (59.1s), branch pushed, PR #235 created.

## Task workflow update - 2026-06-29T19:28:54.880Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Review-iterate: addressing 5 PR review comments. Direction approved by user: (1) reuse compaction.keep_recent_tokens via injected CompactionConfig; (2) keep concrete ForkSnapshotCompactor adapter (no interface); (3) depend directly on SessionCompactor. Refactor deletes 3 speculative abstraction files, delegates virtual compaction to SessionCompactor::prepare()+buildCompactedMessages(), fixes latent prologue-ordering bug.

## Task workflow update - 2026-06-29T19:43:00.748Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 600s)...
- castor check passed (71.0s).
- Pushed task/fork-02-context-prompt-artifact-contract to origin.
- branch 'task/fork-02-context-prompt-artifact-contract' set up to track 'origin/task/fork-02-context-prompt-artifact-contract'.
- PR already exists: https://github.com/ineersa/agent-core/pull/235
- Validation: castor test (full unfiltered, independently re-run): 3845 tests, 12242 assertions OK (17.5s); castor phpstan: 0 errors; castor cs-check: clean (after castor cs-fix); castor deptrac: 0 violations, 0 errors; Reviewer verdict on 71b316b70: APPROVE — all 5 PR comments resolved, no regressions, delegation correctness verified, prologue-ordering fix confirmed; Independent grep: 3 deleted abstraction files absent, zero dangling references to deleted symbols in src/ or tests/; testPrologueOrdering: assertive regression test (forces compaction, asserts summaryIndex > lastPrologueIndex)
- Summary: Review-iterate cycle #2 complete. Refactor commit 71b316b70 addresses all 5 PR review comments: (1) budget from CompactionConfig not duplicate constant, (2) delegated compaction orchestration to SessionCompactor eliminating duplication, (3-5) deleted 3 speculative abstractions (ForkSnapshotCompactorInterface, ForkSnapshotSummaryProviderInterface, DefaultForkSnapshotSummaryProvider). Bonus: fixed latent prologue-ordering bug (summary now after system/user-context prologue) with assertive regression test testPrologueOrdering. Independent full unfiltered castor test: 3845 tests / 12242 assertions OK. Reviewer verdict: APPROVE. Branch now has 3 commits: abce25f9f + 91d611046 + 71b316b70.

## Task workflow update - 2026-06-29T19:43:06.742Z
- Updated PR Status: open-review-iterate-2-done
- Validation: castor check (deterministic gate in worktree, run by move_task->CODE-REVIEW): PASSED (71.0s); castor test (full unfiltered, independently re-run): 3845 tests, 12242 assertions OK (17.5s); castor phpstan: 0 errors; castor cs-check: clean; castor deptrac: 0 violations, 0 errors; Reviewer verdict on refactor commit 71b316b70: APPROVE — all 5 PR comments resolved, no regressions, delegation correctness verified, prologue-ordering fix confirmed; Branch pushed to origin (3 commits: abce25f9f + 91d611046 + 71b316b70); PR #235 updated
- Review-iterate #2 → CODE-REVIEW: refactor commit 71b316b70 addresses all 5 PR review comments. Deterministic castor check PASSED (71.0s), branch pushed, PR #235 updated.

## Task workflow update - 2026-06-29T19:48:42.879Z
- Moved CODE-REVIEW → DONE.
- Merged task/fork-02-context-prompt-artifact-contract into integration checkout.
- Merge made by the 'ort' strategy.
 .../Agent/Artifact/AgentArtifactEntryDTO.php       |   6 +-
 .../Agent/Artifact/AgentArtifactKindEnum.php       |  17 +
 .../Agent/Artifact/AgentArtifactRegistry.php       |  16 +-
 .../Agent/Execution/SubagentExecutionService.php   |   3 +
 .../Agent/Fork/ForkCompactionResult.php            |  30 ++
 src/CodingAgent/Agent/Fork/ForkConfigResolver.php  |  41 +++
 src/CodingAgent/Agent/Fork/ForkContextBuilder.php  |  76 +++++
 .../Agent/Fork/ForkHandoffValidationResultDTO.php  |  29 ++
 .../Agent/Fork/ForkHandoffValidator.php            | 189 +++++++++++
 .../Agent/Fork/ForkResolvedConfigDTO.php           |  25 ++
 src/CodingAgent/Agent/Fork/ForkRunMetadataDTO.php  |  83 +++++
 .../Agent/Fork/ForkSessionSnapshotDTO.php          |  42 +++
 .../Agent/Fork/ForkSnapshotCompactor.php           | 134 ++++++++
 .../Agent/Fork/ForkSnapshotSanitizer.php           |  91 ++++++
 .../Agent/Fork/ForkTaskPromptBuilder.php           | 290 +++++++++++++++++
 src/CodingAgent/Config/AppConfig.php               |  47 +++
 src/CodingAgent/Config/ForkLevelConfigDTO.php      |  65 ++++
 src/CodingAgent/Config/ForkLevelEnum.php           |  40 +++
 src/CodingAgent/Config/ForksConfigDTO.php          |  96 ++++++
 .../Agent/Artifact/AgentArtifactKindEnumTest.php   | 117 +++++++
 .../Agent/Artifact/AgentArtifactRegistryTest.php   |  64 ++--
 .../Artifact/AgentArtifactRetrievalServiceTest.php |  25 +-
 .../Artifact/AgentArtifactSessionListingTest.php   |   3 +-
 .../Agent/Artifact/AgentChildRunDirectoryTest.php  |  11 +-
 .../Agent/Fork/ForkContextBuilderTest.php          | 287 +++++++++++++++++
 .../Agent/Fork/ForkHandoffValidatorTest.php        | 285 +++++++++++++++++
 .../CodingAgent/Agent/Fork/ForkLevelConfigTest.php | 234 ++++++++++++++
 .../Agent/Fork/ForkRunMetadataDTOTest.php          | 149 +++++++++
 .../Agent/Fork/ForkSnapshotCompactorTest.php       | 349 +++++++++++++++++++++
 .../Agent/Fork/ForkSnapshotSanitizerTest.php       | 209 ++++++++++++
 .../Agent/Fork/ForkTaskPromptBuilderTest.php       |  98 ++++++
 31 files changed, 3095 insertions(+), 56 deletions(-)
 create mode 100644 src/CodingAgent/Agent/Artifact/AgentArtifactKindEnum.php
 create mode 100644 src/CodingAgent/Agent/Fork/ForkCompactionResult.php
 create mode 100644 src/CodingAgent/Agent/Fork/ForkConfigResolver.php
 create mode 100644 src/CodingAgent/Agent/Fork/ForkContextBuilder.php
 create mode 100644 src/CodingAgent/Agent/Fork/ForkHandoffValidationResultDTO.php
 create mode 100644 src/CodingAgent/Agent/Fork/ForkHandoffValidator.php
 create mode 100644 src/CodingAgent/Agent/Fork/ForkResolvedConfigDTO.php
 create mode 100644 src/CodingAgent/Agent/Fork/ForkRunMetadataDTO.php
 create mode 100644 src/CodingAgent/Agent/Fork/ForkSessionSnapshotDTO.php
 create mode 100644 src/CodingAgent/Agent/Fork/ForkSnapshotCompactor.php
 create mode 100644 src/CodingAgent/Agent/Fork/ForkSnapshotSanitizer.php
 create mode 100644 src/CodingAgent/Agent/Fork/ForkTaskPromptBuilder.php
 create mode 100644 src/CodingAgent/Config/ForkLevelConfigDTO.php
 create mode 100644 src/CodingAgent/Config/ForkLevelEnum.php
 create mode 100644 src/CodingAgent/Config/ForksConfigDTO.php
 create mode 100644 tests/CodingAgent/Agent/Artifact/AgentArtifactKindEnumTest.php
 create mode 100644 tests/CodingAgent/Agent/Fork/ForkContextBuilderTest.php
 create mode 100644 tests/CodingAgent/Agent/Fork/ForkHandoffValidatorTest.php
 create mode 100644 tests/CodingAgent/Agent/Fork/ForkLevelConfigTest.php
 create mode 100644 tests/CodingAgent/Agent/Fork/ForkRunMetadataDTOTest.php
 create mode 100644 tests/CodingAgent/Agent/Fork/ForkSnapshotCompactorTest.php
 create mode 100644 tests/CodingAgent/Agent/Fork/ForkSnapshotSanitizerTest.php
 create mode 100644 tests/CodingAgent/Agent/Fork/ForkTaskPromptBuilderTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fork-02-context-prompt-artifact-contract.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fork-02-context-prompt-artifact-contract.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-06-29T19:49:28.692Z
- Updated PR Status: merged
- Validation: Merge: clean (ort strategy, no conflicts), integration checkout pulled; Post-merge focused validation on integration checkout:;   castor test --filter 'Fork|AgentArtifact': 127 tests, 399 assertions OK;   castor deptrac: 0 violations, 0 errors;   castor phpstan: 0 errors;   castor cs-check: clean (0 files fixed); Worktree + IDEA exclusions cleaned up
- DONE: PR #235 merged, task branch merged into integration checkout, post-merge validation green. FORK-02 complete.

## Task workflow update - 2026-06-30T02:17:00.996Z
- Summary: ARCHITECTURE RESET: User decided FORK-02/FORK-03 architecture should be scrapped/reworked into a built-in extension modeled after Pi fork extension. FORK-02 contains useful salvageable pieces (compaction/snapshot sanitizer, prompt/handoff builders, DTO concepts), but those should not justify core/runtime intrusion. New direction: extension-owned fork snapshot/result lifecycle; core exposes generic extension hooks only.
