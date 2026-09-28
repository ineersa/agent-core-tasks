# Add GPT-6 Sol and Luna: pinned 272k context, configuration_update support

## Goal
OpenAI released GPT-6 Sol (gpt-6-sol) and GPT-6 Luna (gpt-6-luna) on 2026-09-22 (https://openai.com/index/introducing-gpt-6-sol-and-luna/). models.dev metadata: both limit.context 1050000 / limit.input 922000 / limit.output 128000, efforts none,low,medium,high,xhigh,max, input text/image/pdf, tool_call + structured_output true. Base-tier cost (USD/1M): sol 2/10 (cache 0.2/2.5), luna 0.1/0.5 (cache 0.01/0.125); pricing doubles above the 272k context tier.

User decisions:
- Pin ALL GPT-6 models (gpt-6-astra, gpt-6-sol, gpt-6-luna) to context_window 272000 because pricing doubles after the 272k tier boundary. Bundled catalog already carries 272000 for astra; models.dev limit.context (1050000) must NOT override the pin via providers:update sync.
- Implement the pin as a compat-style flag (user-approved): pin_context_window under the model compatibility block; providers:update sync must respect it. Document the flag.
- New models support configuration_update (reasoning effort changes mid-conversation preserving prompt cache) — same family feature as astra; wire sol/luna into the existing supportsReasoningConfigurationUpdates path.

Known entry points: config/ai-catalog.yaml (version 6), src/CodingAgent/CLI/Providers/ProvidersUpdateCommand.php (METADATA_KEYS + extractModelMetadata + sync apply loop ~L220-260), src/CodingAgent/Config/Ai/AiCompatibility.php (fromArray), src/CodingAgent/Config/Ai/AiModelDefinition.php, src/CodingAgent/Agent/Execution/SessionAwareModelResolver.php:100-119 (hardcoded 'gpt-6-astra' + flag gate), src/Platform/Bridge/OpenAICodex/CodexWebSocketModelClient.php:219 (hardcoded 'gpt-6-astra' + REASONING_RESET; option is only ever set under the flag gate in SessionAwareModelResolver, so option presence implies eligibility). Tests: tests/CodingAgent/Config/Ai/AiCatalogTest.php (testBundledCatalogIncludesCurrentPiModels pattern), tests/CodingAgent/Agent/Execution/SessionAwareModelResolverTest.php, AstraReasoningTransitionHooksTest.php, providers update command tests.

Out of scope: gpt-5.6-sol/luna/terra also report 1050000 on models.dev and get overwritten by sync today — leave for a follow-up decision; do not pin them here. No default-model change (ai.default_model stays null). No renaming of AstraReasoningTransition* internals.

## Acceptance criteria
- config/ai-catalog.yaml: version bumped 6→7 (castor catalog:version-check passes)
- gpt-6-sol + gpt-6-luna under openai-codex: context_window 272000, max_tokens 128000, input [text, image], tool_calling true, reasoning true, thinking_level_map off→none + minimal→low + low/medium/high/xhigh/max direct, base-tier cost from models.dev (sol 2/10/0.2/2.5, luna 0.1/0.5/0.01/0.125)
- pin_context_window compat flag on gpt-6-astra, gpt-6-sol, gpt-6-luna; providers:update sync skips context_window overwrite for pinned models (upstream 1050000 never replaces 272000)
- supports_reasoning_configuration_updates: true on gpt-6-sol and gpt-6-luna; hardcoded 'gpt-6-astra' id checks in SessionAwareModelResolver and CodexWebSocketModelClient replaced by compat-flag/option-presence gating (behavior for astra unchanged)
- pin_context_window documented in docs/ai-catalog.md (providers:update section + field-source table) and AiCompatibility docblock
- tests: AiCatalogTest bundled sol/luna assertions incl. pin flag; providers:update sync test proving the pin blocks context_window overwrite; SessionAwareModelResolver coverage for a non-astra configuration_update model
- focused validation green in task worktree (catalog + resolver + provider-update tests); full castor check via CODE-REVIEW transition

## Workflow metadata
Status: DONE
Branch: task/2026-09-23-gpt6-sol-luna-catalog
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-23-gpt6-sol-luna-catalog
Fork run: none
PR URL: https://github.com/ineersa/agent-core/pull/526
PR Status: merged
Started: 2026-09-23T13:14:44+00:00
Completed: 2026-09-23T14:11:31+00:00

## Work log
- Created: 2026-09-23T13:14:12+00:00

## Task workflow update - 2026-09-23T13:14:44+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-23-gpt6-sol-luna-catalog.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-23-gpt6-sol-luna-catalog.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-23-gpt6-sol-luna-catalog.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-23-gpt6-sol-luna-catalog.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-23-gpt6-sol-luna-catalog.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-23-gpt6-sol-luna-catalog/.idea.

## Task workflow update - 2026-09-23T13:15:38+00:00
- Recorded fork run: none
- Summary: Ownership: owner=main; fork_run=none; revision=task/2026-09-23-gpt6-sol-luna-catalog@baseline; scope=Catalog v7 with gpt-6-sol/luna (272k pinned), pin_context_window compat flag honored by providers:update sync, configuration_update support for sol/luna (drop hardcoded astra id checks), docs, tests; outcome=assigned; commit=none

## Task workflow update - 2026-09-23T13:22:33+00:00
- Validation: castor test --filter=AiCatalogTest: OK — 7 tests, 55 assertions; castor test --filter=ProvidersUpdateCommandTest: OK — 7 tests, 57 assertions (pin holds 272k, other keys sync, unpinned control syncs 1.05M); castor test --filter=SessionAwareModelResolverTest: OK — 21 tests, 66 assertions (non-astra sol baseline + configuration_update); castor catalog:version-check: ok (v6→v7 bump); castor phpstan --path on 4 changed src files: errors=0, file_errors=0
- Summary: Ownership: owner=main; fork_run=none; revision=task/2026-09-23-gpt6-sol-luna-catalog@c0c7150d6; scope=Catalog v7 with gpt-6-sol/luna (272k pinned), pin_context_window compat flag honored by providers:update sync, configuration_update support for sol/luna (drop hardcoded astra id checks), docs, tests; outcome=completed; commit=c0c7150d6

## Task workflow update - 2026-09-23T13:43:10+00:00
- Validation: castor test --filter=AstraReasoningTransitionHooksTest after fixes: OK — 10 tests, 120 assertions (new deterministic sol end-to-end test); castor test --filter=SessionAwareModelResolverTest: OK — 21 tests, 66 assertions; castor test --filter=AiCatalogTest: OK — 7 tests, 63 assertions (cache tiers pinned); castor test --filter=ProvidersUpdateCommandTest: OK — 7 tests, 57 assertions; castor cs-check: clean (files_fixed=0); castor phpstan scoped on changed src files: clean; castor test:llm-real: not applicable — catalog/config-layer change; no provider schema/prompt/streaming change
- Summary: Review: role=reviewer; artifact=agent_7ef5f7ecbc87039a; target revision=c0c7150d6 then d0453d744; scope=spec-fidelity+correctness+test-quality of GPT-6 Sol/Luna catalog + pin + configuration_update. Round 1 verdict=REQUEST CHANGES (CRITICAL: third hardcoded gpt-6-astra gate in AstraReasoningTransitionTransformHook::resolveChatModel stripped Sol/Luna transitions; BUG: missing cache-tier cost assertions; EDGE: stray blank line; NTHs). Fixes applied in d0453d744 (gate removed, deterministic end-to-end sol hook test via parameterized createTransformHook, cache/tool_calling assertions, comment+docs polish, cs alignment). Round 2 verdict=APPROVE, no new issues. Docblock staleness NTH applied in 18284e995. Ownership: owner=main; fork_run=none; revision=task/2026-09-23-gpt6-sol-luna-catalog@18284e995; scope=apply reviewer fixes; outcome=completed; commit=18284e995. No unresolved blockers.

## Task workflow update - 2026-09-23T13:45:30+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (126.6s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-23-gpt6-sol-luna-catalog/var/reports/qa-20260923-134324-3622-521dcb88.
- Session/run: 66.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-23T13:45:32+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-23-gpt6-sol-luna-catalog to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-23-gpt6-sol-luna-catalog/var/reports/qa-20260923-134324-3622-521dcb88.
- Session/run: 66.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-23T13:45:39+00:00
- castor check passed (126.6s).
- Pushed task/2026-09-23-gpt6-sol-luna-catalog to origin.
- Created PR: <url>
- Session/run: 66.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-23T13:45:39+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (126.6s).
- Pushed task/2026-09-23-gpt6-sol-luna-catalog to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/526

## Task workflow update - 2026-09-23T14:11:31+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-23-gpt6-sol-luna-catalog into integration checkout.
- Merge made by the 'ort' strategy.
 config/ai-catalog.yaml                                                    | 31 ++++++++++++++++++++++++++++++-
 docs/ai-catalog.md                                                        | 13 +++++++++----
 docs/settings-models.md                                                   |  9 +++++----
 src/CodingAgent/Agent/Execution/AstraReasoningTransitionTransformHook.php |  1 -
 src/CodingAgent/Agent/Execution/SessionAwareModelResolver.php             |  1 -
 src/CodingAgent/CLI/Providers/ProvidersUpdateCommand.php                  |  9 +++++++++
 src/CodingAgent/Config/Ai/AiCompatibility.php                             |  8 ++++++++
 src/Platform/Bridge/OpenAICodex/CodexReasoningTransitionMetadata.php      |  2 +-
 src/Platform/Bridge/OpenAICodex/CodexWebSocketModelClient.php             |  5 +++--
 tests/CodingAgent/Agent/Execution/AstraReasoningTransitionHooksTest.php   | 71 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++---
 tests/CodingAgent/Agent/Execution/SessionAwareModelResolverTest.php       | 25 +++++++++++++++++++++++++
 tests/CodingAgent/CLI/Providers/ProvidersUpdateCommandTest.php            | 99 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Config/Ai/AiCatalogTest.php                             | 33 +++++++++++++++++++++++++++++++++
 13 files changed, 290 insertions(+), 17 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-23-gpt6-sol-luna-catalog.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-23-gpt6-sol-luna-catalog.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-09-23T14:14:46+00:00
- Updated PR Status: merged
- Validation: Post-merge integration castor check: quality ok (146.2s), 11 lane logs OK, QA leak check ok, llama-proxy cache guard ok (405→405); bin/console providers:update from integration checkout: OK — catalog v7, 8 metadata refreshes, 73 upstream ids listed not added; Pin live-verified: gpt-6-astra/sol/luna context_window 272000 with pin flag after real models.dev sync (upstream reports 1050000); gpt-5.6-* unpinned models show 1050000 as expected; git status clean in /home/ineersa/projects/agent-core; task worktree removed
- Summary: Task completed. PR #526 merged (GitHub merge commit 1b008913c, 2026-09-23T14:06:59Z); DONE transition merged task branch into integration checkout and pulled. Post-merge castor check: quality ok (146.2s), all lanes OK, leak check ok, llama-proxy cache guard ok (405→405). Local machine catalog updated via bin/console providers:update after clearing a stale .hatfield/cache/prod container (referenced SettingsTool removed in 4948c7909): user catalog now version 7 with gpt-6-sol/gpt-6-luna; live models.dev sync (8 metadata refreshes, 73 upstream hints not added) kept all three pinned GPT-6 models at 272000 while unpinned gpt-5.6-sol/luna/terra were overwritten to 1050000 (documented follow-up exposure).
