# Refactor built-in tool arguments and remove duplicate validation

## Goal
User requests a systematic review and refactor of tool implementations. Reported examples: SettingsTool does not use a DTO, while ViewImage has DTO validation plus many handler checks that may duplicate it. Treat these as investigation entry points, not proof that every handler check is redundant.

Inventory built-in tool declarations, argument DTOs, schemas, validators, adapters, and supported execution paths. Establish one owner for each input constraint using existing Symfony Serializer/Validator and Symfony AI facilities. Migrate statically defined built-in arguments to typed DTOs where appropriate, starting with SettingsTool; account for its operation-dependent fields and dynamically typed setting values without flattening or weakening their semantics. Remove handler branches that genuinely repeat validation already guaranteed on every supported call path. Keep execution failures, mutable-resource rechecks, authorization, and safety enforcement at their correct boundaries. Do not remove checks merely because they are if statements.

Preserve flat model-facing schemas, accepted inputs, defaults, errors, approvals, cancellation, and side effects unless a behavior change is explicitly approved. Raw MCP/extension tools with runtime-defined schemas are distinct from built-ins: do not force arbitrary dynamic schemas into fixed DTOs. Document concrete reasons for any built-in exceptions, not speculative compatibility.

Coordinate with the MapToolArguments migration and Composer dependency-upgrade tasks. Reuse upstream native mapping once available; do not introduce another local mapping framework. Delete superseded validation code and tests in the same change. Keep ownership clear between input validation and execution, using existing project facilities rather than new generic tool abstractions.

Task creation only; implementation not started.

## Acceptance criteria
- Inventory built-in tools and identify DTO gaps, duplicate validation, and necessary execution-time/safety checks; explicitly assess SettingsTool and ViewImageTool.
- Use typed DTOs and Symfony validation for statically defined built-in inputs; retain raw dynamic tools only where their schema contract requires it.
- Remove genuinely redundant handler validation and superseded helpers without weakening safety or supported execution paths.
- Preserve provider-visible schemas and behavior, including operation-specific SettingsTool validation and ViewImage resource/content failures.
- Add or update focused deterministic behavior tests at the lowest correct layer; follow testing prerequisites and Castor workflow gates.
- Review the final diff for unnecessary abstractions, duplicate checks, and accidental behavior changes; document justified remaining exceptions.

## Workflow metadata
Status: DONE
Branch: task/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/493
PR Status: merged
Started: 2026-09-11T12:32:10+00:00
Completed: 2026-09-11T17:15:34+00:00

## Work log
- Created: 2026-09-11T02:33:09+00:00

## Task workflow update - 2026-09-11T12:32:10+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation.

## Task workflow update - 2026-09-11T12:33:20+00:00
- Summary: Started isolated worktree at 6d59a21f1. Initial entry-point inspection found SettingsTool raw-array handling preserves omitted versus explicit-null values and operation-specific errors; ViewImageTool explicitly retains operational I/O failures and mutable-file MIME rechecks. Native MapToolArguments is absent from this checkout's dependency. Related migration remains staged/uncommitted in its own worktree; dependency upgrade remains TODO. No source changes made. Need integration sequencing decision before coupling refactor to unreleased mapping.
- Ownership: owner=main; fork_run=none; revision=6d59a21f1; scope=Initial routing and dependency coordination; outcome=blocked; commit=none

## Task workflow update - 2026-09-11T12:34:13+00:00
- Summary: User clarified: retain current Hatfield mapping implementation and dependencies; refactor tools without waiting for or importing MapToolArguments migration. Dependency blocker cleared.
- Ownership: owner=fork; fork_run=pending; revision=6d59a21f1; scope=Built-in argument inventory and tool-only DTO/validation refactor using existing Hatfield mapping; outcome=assigned; commit=none

## Task workflow update - 2026-09-11T13:02:22+00:00
- Validation: Fork read/followed testing skill and tests/AGENTS.md.; Fork focused suite: 123 tests / 415 assertions passed. This does not establish acceptance of schema/error differences.
- Summary: Draft implementation and focused checks completed, but parent has NOT accepted fidelity. Settings DTO conversion currently changes optional scope schema and structured input error hints, and fork added a shared RegistryBackedToolbox normalization change outside tool-only scope. Changes remain uncommitted. Need remove shared change and use existing explicit schema facilities or report blocker before finalizing.
- Ownership: owner=fork; fork_run=agent_d64292d1934a4c02; revision=6d59a21f1; scope=Built-in argument inventory and tool-only DTO/validation refactor using existing Hatfield mapping; outcome=blocked; commit=none

## Task workflow update - 2026-09-11T13:15:06+00:00
- Summary: User approved narrow existing-Hatfield mapper change to support typed handlers with explicit schemas, plus Settings input errors standardized on Symfony validation. Dependencies and MapToolArguments remain untouched; preserve accepted JSON values and schema.
- Ownership: owner=fork; fork_run=agent_d64292d1934a4c02; revision=6d59a21f1; scope=Finish Settings DTO using explicit-schema typed mapping and Symfony validation errors; outcome=assigned; commit=none

## Task workflow update - 2026-09-11T13:19:13+00:00
- Summary: Fork context limit reached. Main takes final polish; approved explicit-schema typed mapping inspected. No dependencies changed.
- Ownership: owner=main; fork_run=none; revision=6d59a21f1; scope=Final diff inspection and assertion strengthening; outcome=assigned; commit=none

## Task workflow update - 2026-09-11T13:20:26+00:00
- Validation: Final parent castor test --filter='SettingsToolTest|RegistryBackedToolboxTest|ToolCallArgumentResolverContainerTest': 67 tests, 205 assertions passed.; Final parent castor test:llm-real --filter=LlamaCppSmokeTest::testRealLlamaCppInvocation: 1 test, 8 assertions passed.; Fork ViewImage/HatfieldDocs/BgStatus suite: 56 tests, 205 assertions passed; scoped PHPStan/style/docs validation passed.; git diff --check passed; castor check intentionally deferred to CODE-REVIEW transition.
- Summary: Implemented built-in inventory, Settings DTO validation with missing-versus-null preservation, and approved narrow explicit-schema typed mapping. Removed Settings handler input guards; retained ViewImage operational/safety checks. Exact Settings schema covered by regression. Dependencies untouched. Main inspected diff, strengthened invalid-input assertions and no-mutation proof, removed duplicate presence-only test. Changes uncommitted; ready for task-to-pr phase, not full-gate validated.
- Ownership: owner=fork; fork_run=agent_d64292d1934a4c02; revision=6d59a21f1; scope=Finish Settings DTO using explicit-schema typed mapping and Symfony validation errors; outcome=completed; commit=none
- Ownership: owner=main; fork_run=none; revision=6d59a21f1; scope=Final diff inspection and assertion strengthening; outcome=completed; commit=none

## Task workflow update - 2026-09-11T13:42:55+00:00
- Review: role=reviewer; artifact=agent_183b8f34c698404d; revision=6d59a21f1+working diff; scope=full specification fidelity; verdict=REQUEST CHANGES, missing iterable PHPDoc and unused test setup.
- Ownership: owner=main; fork_run=none; revision=6d59a21f1; scope=Review PHPDoc and dead-code fixes; outcome=assigned; commit=none

## Task workflow update - 2026-09-11T13:45:20+00:00
- Validation: Reviewer final 67 tests / 205 assertions pass; scoped Tool PHPStan zero errors.; Parent Settings tests 9 / 41 pass; prior focused live smoke 1 / 8 pass.
- Summary: Committed d342b8198. Independent reviewer APPROVE after PHPDoc and unused-test-setup fixes; no blockers. Focused validation reused with final revision verification.
- Review: role=reviewer; artifact=agent_183b8f34c698404d; revision=d342b8198; scope=full diff plus blocker re-review; verdict=APPROVE.
- Ownership: owner=main; fork_run=none; revision=d342b8198; scope=Review PHPDoc and dead-code fixes; outcome=completed; commit=d342b8198

## Task workflow update - 2026-09-11T13:47:32+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation.md.
- Session/run: 41.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation/var/reports/qa-20260911-134531-7200-71caa936.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-11T13:50:00+00:00
- Validation: Final reviewer single-path cs-check clean; Tool PHPStan zero errors; 67 tests / 205 assertions pass.
- Summary: First transition failed cs-check only: value property must precede constructor. Formatter moved declaration verbatim in 187e405fe. Reviewer agent_183b8f34c698404d APPROVE on final revision. No push or PR completed in failed attempt.
- Ownership: owner=main; fork_run=none; revision=187e405fe; scope=Formatter member-order gate fix; outcome=completed; commit=187e405fe

## Task workflow update - 2026-09-11T13:51:04+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (53.6s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation/var/reports/qa-20260911-135011-12128-2820a782.
- Session/run: 41.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-11T13:51:06+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation/var/reports/qa-20260911-135011-12128-2820a782.
- Session/run: 41.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-11T13:51:09+00:00
- castor check passed (53.6s).
- Pushed task/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation to origin.
- Created PR: <url>
- Session/run: 41.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-11T13:51:09+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (53.6s).
- Pushed task/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/493

## Task workflow update - 2026-09-11T15:29:43+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User requests removing isTypedDtoHandler, prefer DTO plus separate value or accepts raw Settings boundary with local Symfony serialization/validation if simpler. Also requires concrete duplicate-validation cleanup across other tools.

## Task workflow update - 2026-09-11T15:29:51+00:00
- Summary: Prior implementation fork cannot resume due context limit. New bounded owner needed for sequential revision; preserve dependencies, flat schemas, safety and operational failures.
- Ownership: owner=fork; fork_run=pending; revision=187e405fe; scope=Replace Settings mapping design and audit/remove other built-in redundant validation; outcome=assigned; commit=none

## Task workflow update - 2026-09-11T15:45:22+00:00
- Validation: Reviewer agent_183b8f34c698404d APPROVE 0925e58c8; 66 tests/208 assertions plus 201/667 other-tool suite pass; Tool PHPStan and scoped style clean.; Fork read testing skill/tests AGENTS; live smoke 1/8 pass.
- Summary: Revision 0925e58c8 removes isTypedDtoHandler and shared mapping change. Uses user-authorized plain Settings arguments with local Symfony serialization/validation. Other built-in audit independently corroborated: remaining checks are operational/resource/context checks, no further redundant input guards identified. No other handlers changed; not claiming otherwise.
- Ownership: owner=fork; fork_run=agent_4c154818a834a12a; revision=0925e58c8; scope=Replace Settings mapping design and audit other built-in validation; outcome=completed; commit=0925e58c8
- Review: role=reviewer; artifact=agent_183b8f34c698404d; revision=0925e58c8; scope=feedback fidelity and skeptical other-tool audit; verdict=APPROVE

## Task workflow update - 2026-09-11T15:46:37+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (58.4s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation/var/reports/qa-20260911-154539-20377-f90c67b7.
- Session/run: 41.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-11T15:46:39+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation/var/reports/qa-20260911-154539-20377-f90c67b7.
- Session/run: 41.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-11T15:46:39+00:00
- castor check passed (58.4s).
- Pushed task/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation to origin.
- PR already exists: <url>
- Session/run: 41.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-11T15:46:39+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (58.4s).
- Pushed task/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/493

## Task workflow update - 2026-09-11T16:50:17+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Fix required serializer DI and consolidate duplicate ViewImage resource inspection rather than defend repeated reads as race checks. Audit other tools for duplicated work, not only identical guards.

## Task workflow update - 2026-09-11T16:50:31+00:00
- Ownership: owner=fork; fork_run=agent_4c154818a834a12a; revision=0925e58c8; scope=Required serializer DI and single-owner tool resource validation; outcome=assigned; commit=none

## Task workflow update - 2026-09-11T17:00:23+00:00
- Validation: Independent 165 tests/483 assertions passed; Tool PHPStan zero errors; full cs-check clean. Fork broader suite 202/713 passed.
- Summary: 613b3599f requires @serializer injection without fallback. ViewImage now owns one resource inspection; deleted ViewImageTarget constraint/validator and moved all limits/vision checks into handler. Independent reviewer APPROVE verified each old policy preserved and container wiring. Other read/edit handlers do not repeat validator inspection.
- Ownership: owner=fork; fork_run=agent_4c154818a834a12a; revision=613b3599f; scope=Required serializer DI and single-owner tool resource validation; outcome=completed; commit=613b3599f
- Review: role=reviewer; artifact=agent_183b8f34c698404d; revision=613b3599f; scope=single ownership and production DI; verdict=APPROVE

## Task workflow update - 2026-09-11T17:01:40+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (60.6s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation/var/reports/qa-20260911-170039-27768-bc8e2922.
- Session/run: 41.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-11T17:01:42+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation/var/reports/qa-20260911-170039-27768-bc8e2922.
- Session/run: 41.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-11T17:01:42+00:00
- castor check passed (60.6s).
- Pushed task/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation to origin.
- PR already exists: <url>
- Session/run: 41.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-11T17:01:42+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (60.6s).
- Pushed task/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/493

## Task workflow update - 2026-09-11T17:15:34+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation: ide_close_project returned isError.
- Merged task/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation into integration checkout.
- Merge made by the 'ort' strategy.
 architecture/tools-and-mcp.md                                          |  24 +++++++++++++++++++
 config/services.yaml                                                   |   4 ++++
 src/CodingAgent/Tool/Arguments/SettingsArgumentsDTO.php                |  72 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 src/CodingAgent/Tool/Arguments/ViewImageArgumentsDTO.php               |   7 ++----
 src/CodingAgent/Tool/RegistryBackedToolbox.php                         |   2 +-
 src/CodingAgent/Tool/SettingsTool.php                                  | 148 +++++++++++++++++++++++++++++++++++++++++++++++++++++-----------------------------------------------------------------
 src/CodingAgent/Tool/ToolDefinitionDTO.php                             |   7 +++---
 src/CodingAgent/Tool/Validation/Settings/SettingsPath.php              |  25 ++++++++++++++++++++
 src/CodingAgent/Tool/Validation/Settings/SettingsPathValidator.php     |  34 +++++++++++++++++++++++++++
 src/CodingAgent/Tool/Validation/ViewImage/ViewImageTarget.php          |  25 --------------------
 src/CodingAgent/Tool/Validation/ViewImage/ViewImageTargetValidator.php | 135 -----------------------------------------------------------------------------------------------------------
 src/CodingAgent/Tool/ViewImageTool.php                                 |  71 ++++++++++++++++++++++++---------------------------------
 tests/CodingAgent/Tool/RegistryBackedToolboxTest.php                   |  25 +-------------------
 tests/CodingAgent/Tool/SettingsToolTest.php                            | 176 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++----------------------------
 tests/CodingAgent/Tool/Support/ToolValidationHarness.php               |   5 ++--
 tests/CodingAgent/Tool/ToolCallArgumentResolverContainerTest.php       |  19 +++++++++-------
 tests/CodingAgent/Tool/ViewImageToolTest.php                           | 182 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++-----------------------------------------------------------------------------
 17 files changed, 502 insertions(+), 459 deletions(-)
 create mode 100644 src/CodingAgent/Tool/Arguments/SettingsArgumentsDTO.php
 create mode 100644 src/CodingAgent/Tool/Validation/Settings/SettingsPath.php
 create mode 100644 src/CodingAgent/Tool/Validation/Settings/SettingsPathValidator.php
 delete mode 100644 src/CodingAgent/Tool/Validation/ViewImage/ViewImageTarget.php
 delete mode 100644 src/CodingAgent/Tool/Validation/ViewImage/ViewImageTargetValidator.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-11-refactor-built-in-tool-arguments-and-remove-duplicate-validation.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR #493 merged at c205ba01e5e3acde8c56f9f148cf35beedd46271.

## Task workflow update - 2026-09-11T17:16:55+00:00
- Updated PR Status: merged
- Validation: Integration castor check passed all 10 lanes. Reports: var/reports/qa-20260911-171541-32571-d513fd8d. Unit 4941/20750; controller replay 11/175; TUI 9/64; live 5/30. Leak/cache guards passed.
- Summary: Post-merge validation complete at integration 1ca7fec401f6d8a07300ad280d566ecaae1fd930. Git status clean; task worktree removed. IDE close reported degradation, filesystem cleanup succeeded.
