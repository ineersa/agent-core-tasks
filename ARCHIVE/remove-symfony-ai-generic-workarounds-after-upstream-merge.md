# Remove Generic Symfony AI workarounds after upstream PRs merge

## Goal
Follow-up cleanup after upstream Symfony AI Generic streaming PRs land.

Upstream PR status at creation:
- https://github.com/symfony/ai/pull/2273 merged — keys streamed tool calls by provider `tool_calls[].index` (issue #2193).
- https://github.com/symfony/ai/pull/2276 merged — exposes streamed `finish_reason` as metadata (issue #2194).
- https://github.com/symfony/ai/pull/2275 open — preserves cache/reasoning token usage in Generic completions (issue #2274).

Agent-core currently keeps local workarounds:
- `src/Platform/Bridge/Generic/DurableResultConverter.php`
- `src/Platform/Bridge/Generic/PromptCacheTokenUsageExtractor.php`
- Custom `DurableResultConverter` wiring in `src/CodingAgent/Infrastructure/SymfonyAi/SymfonyAiProviderFactory.php`
- Raw stream capture seam currently coupled to `DurableResultConverter`

Once all required upstream PRs are merged and Composer is updated to the merged Symfony AI commit/release, reduce local code safely. Do not blindly delete the durable converter: verify which edge cases remain uncovered upstream, especially empty-id suppression, arguments-before-id buffering, phantom block filtering, id/index reassociation, and Hatfield raw stream capture.

## Acceptance criteria
- Composer/Symfony AI dependency is updated to a commit/release containing upstream PRs #2273, #2275, and #2276 or equivalent merged changes.
- Compare upstream Generic `ResultConverter` behavior against Hatfield `DurableResultConverterTest` edge cases and document which local durability features remain necessary.
- Remove `PromptCacheTokenUsageExtractor` if upstream #2275 behavior fully covers Hatfield cache/reasoning token needs; update tests accordingly.
- Decide whether to remove, shrink, or keep `DurableResultConverter`; if raw stream capture is still needed, split it from provider parsing where practical.
- Wire finish_reason metadata to agent-core stop reason if upstream metadata is available and this is still desired; otherwise explicitly defer with rationale.
- Run focused provider/platform tests and required Castor validation (`castor test`, `castor deptrac`, `castor phpstan`, `castor cs-check`; `castor test:llm-real` if provider/LLM path changed; `castor check` before CODE-REVIEW per workflow).

## Workflow metadata
Status: DONE
Branch: task/remove-symfony-ai-generic-workarounds-after-upstream-merge
Worktree: /home/ineersa/projects/agent-core-worktrees/remove-symfony-ai-generic-workarounds-after-upstream-merge
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/291
PR Status: merged
Started: 2026-07-15T17:57:54.028Z
Completed: 2026-07-15T19:49:38.321Z

## Work log
- Created: 2026-07-12T17:04:40.330Z

## Task workflow update - 2026-07-15T17:50:59.737Z
- Summary: Task-explain decision: upgrade Symfony AI packages to stable ^0.11; there is no configured Symfony AI fork to update. Remove PromptCacheTokenUsageExtractor entirely and rely on Symfony AI v0.11 Generic TokenUsageExtractor because the extra local provider field variants were found during upstream PR work to be impossible/nonexistent. Keep/rebase DurableResultConverter onto v0.11 behavior while retaining Hatfield-only arguments-before-ID buffering, ID/index reassociation, phantom filtering, and raw stream capture; adopt upstream finish_reason metadata and error handling. Address relevant v0.11 API breaks and validate provider/runtime paths.
- 2026-07-15 task-explain: User confirmed PromptCacheTokenUsageExtractor should be removed, not shrunk. Symfony AI's upstream prompt-cache extraction is the desired source of truth; local extra field variants are impossible/nonexistent. User agreed with the remaining v0.11 upgrade and DurableResultConverter rebase plan.

## Task workflow update - 2026-07-15T17:57:54.028Z
- Moved TODO → IN-PROGRESS.
- Created branch task/remove-symfony-ai-generic-workarounds-after-upstream-merge.
- Created worktree /home/ineersa/projects/agent-core-worktrees/remove-symfony-ai-generic-workarounds-after-upstream-merge.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/remove-symfony-ai-generic-workarounds-after-upstream-merge.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/remove-symfony-ai-generic-workarounds-after-upstream-merge.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/remove-symfony-ai-generic-workarounds-after-upstream-merge.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/remove-symfony-ai-generic-workarounds-after-upstream-merge.
- Summary: Starting implementation after task-explain agreement: upgrade Symfony AI to stable 0.11, remove PromptCacheTokenUsageExtractor, and rebase DurableResultConverter onto upstream 0.11 while preserving Hatfield-only durability and raw capture behavior.

## Task workflow update - 2026-07-15T18:07:10.506Z
- Validation: castor test --filter=DurableResultConverterTest — PASS (20 tests); castor test --filter='RegistryBackedToolboxTest|ReasoningContentFeatureShaperTest' — PASS; castor test — PASS (4390 tests); castor test:controller-replay — PASS (8 tests), but teardown reported PID 1362 still alive; no process was signaled; castor deptrac — PASS (0 violations); castor phpstan — PASS; castor cs-check — PASS; castor test:llm-real --filter=LlamaCppSmokeTest — PASS; Full castor test:llm-real — not run during task-start; Full castor check — intentionally deferred to task-to-pr workflow
- Summary: Implementation committed as 03bdaf165. Updated all four Symfony AI packages to stable ^0.11/v0.11.0; deleted PromptCacheTokenUsageExtractor; rebased DurableResultConverter onto v0.11 error, token usage, incomplete-stream, and finish_reason behavior while retaining arguments-before-ID buffering, ID/index reassociation, phantom filtering, and raw capture. LlmPlatformAdapter now maps normalized finish_reason metadata to stop reasons, and affected Symfony AI 0.11 ToolCallMessage/tool-event APIs and tests were updated. Worktree is clean. Full castor check and unfiltered castor test:llm-real remain for task-to-pr.

## Task workflow update - 2026-07-15T18:48:23.749Z
- Validation: Reviewer final verdict at 354485f7d — APPROVED; castor test — PASS (4401 tests, 14580 assertions); castor deptrac — PASS (0 violations); castor phpstan — PASS (0 errors); castor cs-check — PASS (0 files); castor test:llm-real — initial parallel warmup runs exposed transient/cold-cache live failures; each failing scenario passed filtered on rerun; castor test:llm-real final full run — PASS (10 tests, 122 assertions); Worktree git status — clean
- Summary: Task-to-PR review completed at HEAD 354485f7d. Reviewer first returned APPROVE WITH SUGGESTIONS; all actionable findings were addressed in e986ebe7f (HTTP-status parity, stream error classification tests, end-to-end finish_reason metadata proof, finish-only empty-response guard, comments/docs cleanup) and 354485f7d (final stale invariant comment). Final reviewer verdict: APPROVED with no remaining actionable findings. Worktree is clean.

## Task workflow update - 2026-07-15T18:50:28.749Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (112.7s).
- Pushed task/remove-symfony-ai-generic-workarounds-after-upstream-merge to origin.
- branch 'task/remove-symfony-ai-generic-workarounds-after-upstream-merge' set up to track 'origin/task/remove-symfony-ai-generic-workarounds-after-upstream-merge'.
- Created PR: https://github.com/ineersa/agent-core/pull/291
- Validation: castor test — PASS (4401 tests, 14580 assertions); castor deptrac — PASS; castor phpstan — PASS; castor cs-check — PASS; castor test:llm-real final full run — PASS (10 tests, 122 assertions)
- Summary: Final reviewer verdict APPROVED at 354485f7d. Focused Castor validation passed, including full live provider smoke after cache warmup. Moving to CODE-REVIEW for deterministic castor check, push, and PR creation.

## Task workflow update - 2026-07-15T18:50:34.492Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/291
- Updated PR Status: open
- Validation: move_task deterministic castor check — PASS (112.7s); Branch pushed to origin/task/remove-symfony-ai-generic-workarounds-after-upstream-merge; PR created: https://github.com/ineersa/agent-core/pull/291
- Summary: PR #291 created after deterministic CODE-REVIEW gate passed. Final HEAD 354485f7d; reviewer APPROVED.

## Task workflow update - 2026-07-15T19:49:38.321Z
- Moved CODE-REVIEW → DONE.
- Merged task/remove-symfony-ai-generic-workarounds-after-upstream-merge into integration checkout.
- Merge made by the 'ort' strategy.
 composer.json                                      |   8 +-
 composer.lock                                      | 174 +++++++-------
 .../Application/Handler/ExecuteLlmStepWorker.php   |   9 +-
 .../SymfonyAi/AgentMessageConverter.php            |  17 +-
 .../SymfonyAi/LlmPlatformAdapter.php               |  20 +-
 .../Extension/ExtensionToolHookEventSubscriber.php |   2 +-
 .../Bridge/Generic/DurableResultConverter.php      | 168 +++++++-------
 .../Generic/PromptCacheTokenUsageExtractor.php     | 132 -----------
 .../Application/Handler/ExecutionWorkerTest.php    |  43 ++++
 .../DurableFinishReasonPlatformIntegrationTest.php | 197 ++++++++++++++++
 .../ReasoningContentFeatureShaperTest.php          |   2 +-
 .../Tool/OutputCapLlmTransformHookTest.php         |  12 +-
 .../CodingAgent/Tool/RegistryBackedToolboxTest.php |   4 +-
 tests/CodingAgent/Tool/ViewImageToolTest.php       |   8 +-
 .../Bridge/Generic/DurableResultConverterTest.php  | 249 +++++++++++++++++----
 15 files changed, 647 insertions(+), 398 deletions(-)
 delete mode 100644 src/Platform/Bridge/Generic/PromptCacheTokenUsageExtractor.php
 create mode 100644 tests/AgentCore/Infrastructure/SymfonyAi/DurableFinishReasonPlatformIntegrationTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/remove-symfony-ai-generic-workarounds-after-upstream-merge.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/remove-symfony-ai-generic-workarounds-after-upstream-merge.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #291 confirmed merged on GitHub at 2026-07-15T19:49:08Z with merge commit 7a77d9b583b94a4679ca5f8189169d0e8176e196. Moving task to DONE and syncing integration checkout.

## Task workflow update - 2026-07-15T19:59:56.809Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/291
- Updated PR Status: merged
- Validation: PR #291 — MERGED at 2026-07-15T19:49:08Z; move_task DONE — task branch merged into integration checkout and git pull sync completed; composer install --no-interaction --no-progress — PASS; integration vendor upgraded to Symfony AI v0.11.0; Post-merge castor check qa-20260715-195044-245128-d494b67b — code lanes passed; TUI lane hit 120s timeout; proxy cache warmed 354→355; castor test:tui standalone — PASS (36 tests, 186 assertions, 107.6s); castor test:llm-real standalone — PASS (10 tests, 122 assertions); proxy cache stable at 355; Post-merge castor check qa-20260715-195531-262293-dc3481fb — all lanes except TUI passed; TUI BashCancelFollowUpE2eTest timed out waiting for cancellation; castor test:tui --filter=BashCancelFollowUpE2eTest — PASS (1 test); Post-merge castor check qa-20260715-195706-266728-185d2f1f — deptrac, test (4414), controller replay, llm-real, phpstan, cs-check all PASS; only TUI lane timed out at 120s; All failed-gate worker leak checks — PASS; no stale QA worker candidates; Integration git status — clean (main ahead of origin from local workflow merge commits)
- Summary: Task merged and moved to DONE. PR #291 merged as 7a77d9b583b94a4679ca5f8189169d0e8176e196. Integration checkout was synced and Composer-installed to v0.11 dependencies. Post-merge relevant validation is green; repeated full castor check attempts were blocked only by the unrelated TUI lane (one existing BashCancelFollowUpE2eTest timing failure and repeated 120s lane timeouts), while every other deterministic lane passed, the full standalone TUI suite passed, and leak checks were clean. Git worktree registration was removed; a residual unregistered directory containing only `.hatfield/logs/agent-2026-07-15.log` remains for manual cleanup.
