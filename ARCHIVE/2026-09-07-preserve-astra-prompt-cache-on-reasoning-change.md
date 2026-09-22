# Preserve Astra prompt cache when reasoning level changes

## Goal
Implement only GPT-6 Astra `configuration_update` reasoning changes. Do not include async tool calling or mid-turn steering.

Hatfield currently resolves the selected reasoning level as request-level `reasoning.effort`, so changing the level invalidates WebSocket continuation compatibility and the cached prompt prefix. For one active Astra provider conversation, keep the initial request-level effort as its baseline. On every subsequent request that contains real model-visible input, prepend exactly one harness-authored `configuration_update` containing the currently selected effort. Repeating an unchanged update on later requests is intentional: it avoids durable last-submitted-effort tracking while retaining a stable request-level baseline.

Example: initial baseline `medium`; later requests use baseline `medium` plus update `high`; after selection changes to `low`, requests use baseline `medium` plus update `low`. Start a new baseline when a new provider conversation epoch begins, including a new session or a model change incompatible with the previous continuation.

Constraints:
- Never send a request containing only `configuration_update`.
- Never generate adjacent configuration updates inside an input sequence.
- Apply a changed selection to the next provider request, not an already-running response.
- Keep non-Astra models and providers unchanged.
- Gate support explicitly to the Astra model/capability after verifying Hatfield's configured Codex OAuth endpoint accepts the documented item.
- Do not add async tool calling, `response.steer`, new user settings, or speculative compatibility paths.
- Do not infer effective reasoning from response metadata; Astra reports the request-level baseline there.
- Preserve existing application-side compaction behavior and do not enable Responses automatic compaction or truncation.

Reference plan: `.pi/plans/gpt-6-astra-three-features-implementation-plan.md`, specifically the reasoning section. OpenAI Codex at `9f70e348e0` contains a more stateful deduplicating implementation; this task intentionally chooses the smaller repeated-update design.

## Acceptance criteria
- The first request of an Astra provider conversation uses the selected reasoning effort as the request-level baseline.
- Every later Astra request with model-visible input keeps the baseline effort and includes exactly one `configuration_update` with the current selected effort.
- Changing the selection from medium to high and later to low affects the next request without changing the request-level baseline or invalidating continuation solely because of reasoning.
- Repeated unchanged selections do not create adjacent updates, and Hatfield never sends an update-only request.
- Starting a new provider conversation establishes a fresh baseline from the current selection.
- Non-Astra request shaping and reasoning behavior remain unchanged.
- Focused deterministic provider-bridge tests cover initial baseline, repeated update, changed update, no-input handling, continuation compatibility, and non-Astra behavior.
- Required Castor validation passes according to the testing skill and task workflow.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/482
PR Status: merged
Started: 2026-09-08T01:22:18+00:00
Completed: 2026-09-08T17:34:19+00:00

## Work log
- Created: 2026-09-07T17:11:59.080Z

## Task workflow update - 2026-09-08T01:22:18+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change.

## Task workflow update - 2026-09-08T01:24:17+00:00
- Summary: Routing pass found CodexRequestBodyFactory, CodexWebSocketModelClient and committed CodexWebSocketContinuationState as the request/continuation seam, with existing cached-client and continuation tests. Main owns this cohesive provider change. Configured Codex endpoint is https://chatgpt.com/backend-api/codex/responses, transport websocket, model gpt-6-astra. Endpoint acceptance is a prerequisite. Task requirements supersede the older plan's durable deduplication and runtime event additions.
- Ownership: owner=main; fork_run=none; revision=0fd103447; scope=Astra endpoint capability proof and bounded Codex provider-bridge reasoning updates with deterministic tests; outcome=assigned; commit=none

## Task workflow update - 2026-09-08T01:26:18+00:00
- Routing: task already IN-PROGRESS with an existing clean worktree at baseline 0fd103447; reusing it without repeating the TODO transition. Read task-workflow router, task-start, implementation-ownership, and specification-fidelity.
- Routing: Codex request shaping is in CodexRequestBodyFactory, SSE CodexModelClient, and CodexWebSocketModelClient. Cached conversation ownership is in CodexWebSocketConnectionCache/ContinuationState; ReasoningOptionsResolver currently supplies selected request-level effort. Existing bridge tests cover wire frames and continuation. No nested Platform instructions found; root/test instructions apply. The task supersedes the reference plan's durable last-submitted-effort/event design.
- Ownership: owner=fork; fork_run=none; revision=0fd103447; scope=Verify configured Codex OAuth configuration_update support, implement Astra-only repeated reasoning updates at the provider conversation boundary, and run focused provider validation; outcome=assigned; commit=none
- Delegation rationale: one bounded provider slice with unfamiliar connection/continuation lifecycle and required external endpoint investigation. A fork can own detailed exploration and focused iteration while main checks the resulting diff and evidence. No parallel writers. No TUI behavior changes, async tools, steering, settings, automatic compaction, or speculative compatibility paths.

## Task workflow update - 2026-09-08T01:35:20+00:00
- Summary: User clarified scope: support plain WebSocket through session metadata baseline and an Astra-only compatibility feature. On restart/resume, establish baseline from the currently selected reasoning effort. Keep changes minimal; main implements without forks. Endpoint proof passed three requests over configured Codex OAuth WebSocket endpoint using baseline medium then configuration_update high and low, each completing successfully. No credentials or prompt contents recorded.

## Task workflow update - 2026-09-08T13:53:38+00:00
- Validation: Configured Codex OAuth endpoint probe: wss://chatgpt.com/backend-api/codex/responses, gpt-6-astra, baseline medium with high then low updates; all three response.completed.; castor test --filter='SessionAwareModelResolverTest|Codex.*Test|InProcessAttachDoesNotContinueTest|HatfieldSessionStoreTest': 267 tests, 912 assertions; maximum individual case 0.408s.; castor test:llm-real --filter=LlamaCppSmokeTest: 1 test, 8 assertions.; castor phpstan, castor deptrac, castor dead-code, castor docs:validate, castor cs-check, castor catalog:version-check and git diff --check passed.
- Summary: Implemented database-backed Astra baseline, model-level compatibility flag, repeated wire-only configuration updates, cached continuation comparison, and resume/model-change reset. Plain WebSocket and SSE use the shared body factory. Explicit compaction overrides remain unchanged. Resume also suppresses stale previous_response_id even when the new baseline equals the old one. Removed the stopped writer's in-memory epoch implementation after preserving its diff under ignored var/tmp/astra-overlap-backup; retained and extended its cached-client regression test. Main implemented without forks. Commit 89412f311. Worktree clean. task-to-pr is next; full gate and independent review have not run.
- Ownership: owner=main; fork_run=none; revision=0fd103447; scope=database-backed Astra reasoning baseline, compatibility flag, provider input and continuation shaping, resume reset and focused regression proof; outcome=completed; commit=89412f311
- Pinned settings model maps replace catalog models. Existing pinned Astra definitions need compatibility.supports_reasoning_configuration_updates=true; user runtime settings were not modified.

## Task workflow update - 2026-09-08T15:27:07+00:00
- Summary: Independent specification-fidelity review APPROVE WITH SUGGESTIONS at 89412f311; no blocking correctness/security/specification findings. Reusing same-revision focused validation. Full deterministic gate reserved for CODE-REVIEW transition.
- Review: role=reviewer; artifact=agent_12b8277f644fb26f; revision=89412f311; scope=complete task diff, specification fidelity, baseline epoch lifecycle, all Codex transports, compaction and deterministic tests; outcome=APPROVE WITH SUGGESTIONS. Reviewer confirmed testing skill and tests/AGENTS.md read and followed.
- Nonblocking suggestions retained without expanding scope: optional live tool-loop endpoint probe, scalar-input hardening, doc/comment cleanup. No unresolved transition blockers.

## Task workflow update - 2026-09-08T15:30:19+00:00
- Summary: CODE-REVIEW gate failed; task remains IN-PROGRESS. QA reports var/reports/qa-20260908-152724-3038-2a82244f. Controller/live lanes fail at startup with no such column h0_.reasoning_baseline: new migration missing from ApplicationMigrationExecutor registration. Unit lane fails RunOrchestratorInvalidateRunContextTest with generated PlatformInterface proxy pointing into deleted prior test cache root. Worker diagnostics report no stale QA candidates. Fixing causes, no blind retry.
- Ownership: owner=main; fork_run=none; revision=89412f311; scope=register Astra migration and diagnose gate test-container cache isolation failure; outcome=assigned; commit=none

## Task workflow update - 2026-09-08T15:40:04+00:00
- Validation: Focused KernelCacheIsolationTest|RunOrchestratorInvalidateRunContextTest|ApplicationMigrationExecutorTest: 13 tests, 91 assertions.; castor test:controller-replay: 9 tests, 131 assertions.; castor phpstan passed; targeted cs-fix applied; git diff --check passed.; castor test:llm-real --filter=LlamaCppSmokeTest: 1 test, 8 assertions.
- Summary: Fixed startup migration registration and unrelated deterministic kernel container-class collision exposed by the gate. Container names now include cache-root identity; added two-project kernel regression. Reviewer agent_12b8277f644fb26f re-reviewed delta at 57aea46dd and APPROVED. No unresolved blockers.
- Ownership: owner=main; fork_run=none; revision=89412f311; scope=register Astra migration and fix gate test-container cache isolation failure; outcome=completed; commit=57aea46dd
- Review: role=reviewer; artifact=agent_12b8277f644fb26f; revision=57aea46dd; scope=migration registration and kernel cache-root container-class identity delta; outcome=APPROVE

## Task workflow update - 2026-09-08T15:41:26+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (45.9s).
- Pushed task/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change to origin.
- branch 'task/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change' set up to track 'origin/task/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change'.
- Created PR: https://github.com/ineersa/agent-core/pull/482
- Summary: Reviewer approved 57aea46dd after fixes to both deterministic gate failures. Focused migration/kernel tests, controller replay and live smoke passed.

## Task workflow update - 2026-09-08T17:34:19+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change: ide_close_project returned isError.
- Merged task/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change into integration checkout.
- Merge made by the 'ort' strategy.
 config/ai-catalog.yaml                                                     |  4 +++-
 docs/session-storage.md                                                    |  6 ++++++
 docs/settings-models.md                                                    |  9 +++++++++
 migrations/application/Version20260908010000.php                           | 26 +++++++++++++++++++++++++
 src/CodingAgent/Agent/Execution/SessionAwareModelResolver.php              | 21 ++++++++++++++++++--
 src/CodingAgent/Config/Ai/AiCompatibility.php                              |  2 ++
 src/CodingAgent/Entity/HatfieldSession.php                                 |  5 +++++
 src/CodingAgent/Kernel.php                                                 |  7 +++++++
 src/CodingAgent/Migrations/ApplicationMigrationExecutor.php                |  1 +
 src/CodingAgent/Runtime/InProcess/InProcessAgentSessionClient.php          |  3 +++
 src/CodingAgent/Session/HatfieldSessionStore.php                           | 33 ++++++++++++++++++++++++++++++++
 src/Platform/Bridge/OpenAICodex/CodexRequestBodyFactory.php                | 22 +++++++++++++++++++++
 src/Platform/Bridge/OpenAICodex/CodexWebSocketContinuationState.php        | 28 +++++++++++++++++++++++++++
 src/Platform/Bridge/OpenAICodex/CodexWebSocketModelClient.php              |  7 +++++++
 tests/CodingAgent/Agent/Execution/SessionAwareModelResolverTest.php        | 41 +++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Phar/KernelCacheIsolationTest.php                        | 27 ++++++++++++++++++++++++++
 tests/CodingAgent/Runtime/InProcess/InProcessAttachDoesNotContinueTest.php |  7 ++++++-
 tests/Platform/Bridge/OpenAICodex/CodexRequestBodyFactoryTest.php          | 53 +++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Platform/Bridge/OpenAICodex/CodexWebSocketCachedModelClientTest.php  | 93 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 19 files changed, 391 insertions(+), 4 deletions(-)
 create mode 100644 migrations/application/Version20260908010000.php
 create mode 100644 tests/Platform/Bridge/OpenAICodex/CodexRequestBodyFactoryTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-preserve-astra-prompt-cache-on-reasoning-change.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR #482 merged at 2026-09-08T17:33:31Z, merge commit 0a9834c1ae4a5d1be7a0b0504173a7672a0c2805. Integrating locally; post-merge castor check follows.

## Task workflow update - 2026-09-08T17:35:37+00:00
- Updated PR Status: merged
- Validation: castor check passed all 10 lanes; QA reports var/reports/qa-20260908-173426-548-3722040f.; Unit 4901 tests/20408 assertions; controller replay 9/135; TUI 9/64; live 5/30.; No cases exceeded 10s; maximum 6.922s. QA leak check and cache guard passed.
- Summary: Post-merge validation passed on integration revision 3fd7ff4cf. Main working tree clean; task worktree confirmed removed. JetBrains close reported degradation during removal, but filesystem cleanup succeeded.

## Task workflow update - 2026-09-10T22:49:40+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
