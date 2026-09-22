# Remove SanitizedGenericModelClient and redesign internal invocation context

## Goal
Internal Hatfield control metadata is currently transported through Symfony AI's provider `options` bag under keys such as `_agent_core_invocation`, then defensively stripped by `SanitizedGenericModelClient` before generic HTTP serialization. This is unsafe by construction: adding a new internal key can silently leak nested runtime objects and volatile identifiers into provider request bodies, as happened when `PlatformInvocationMetadata` began carrying the full `ResolvedModel` and its `provider_cache_key`.

Redesign invocation context so internal runtime state (ModelInvocationInput, cancellation token, resolved routing identity, hook/control metadata) travels through a typed internal control channel and is structurally unavailable to provider wire serializers. Remove `SanitizedGenericModelClient` rather than maintaining a denylist of internal option names. Provider options should contain only explicitly provider-facing values. Preserve provider-specific mapping: the immutable per-session `provider_cache_key` remains internal correlation input and Codex maps it intentionally to `prompt_cache_key`; generic/DeepSeek-compatible requests do not receive unsupported internal fields.

This is follow-up architecture work. The current critical bug receives the minimal containment fix by adding `_agent_core_invocation` to `GenericProviderInternalOptionKeys::ALL`.

## Acceptance criteria
- `SanitizedGenericModelClient` and its internal-option denylist are removed.
- Internal invocation context cannot be serialized into provider HTTP/WebSocket request bodies by construction, not convention.
- Resolved model/reasoning identity is resolved once per invocation and shared with routing, hooks, results, and events without using the provider options bag as an internal message bus.
- Provider request options are explicit/allowlisted per provider contract; no Hatfield runtime DTOs, cancellation tokens, session/run/step IDs, or internal metadata objects reach wire payloads unintentionally.
- Persisted session `provider_cache_key` remains stable and available to providers that intentionally support it; Codex maps it to `prompt_cache_key`, while generic/DeepSeek providers omit it.
- Deterministic tests prove internal metadata cannot leak and provider-specific cache/correlation mapping remains correct.
- All tests follow the ≤10s deterministic standards; Castor test, controller replay, TUI, llm-real warm-cache stability, deptrac, phpstan, cs-check, and full check pass.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context
Fork run: aoxowk8q991q
PR URL: https://github.com/ineersa/agent-core/pull/444
PR Status: merged
Started: 2026-08-29T23:47:56.597Z
Completed: 2026-08-30T22:39:14.795Z

## Work log
- Created: 2026-08-24T02:47:07+00:00

## Task workflow update - 2026-08-29T23:47:33.248Z
- Summary: Finalized implementation direction after task-explain discussion: do not add a WeakMap, ambient invocation registry, or private context transport through Symfony AI. Prepare the invocation completely inside Hatfield before the Symfony AI boundary. Resolve model/provider once, resolve tools, apply compatibility shaping and request hooks, and map provider correlation there. Call Symfony AI with only provider-facing model, messages, and options. Keep cancellation/input/resolved identity as typed local Hatfield state for results/events. Make projected provider catalogs reject provider-qualified models owned by another provider so normal Symfony routing selects the exact provider without metadata. Reuse the same preparation path for ConfiguredModelAgentRunner. Codex receives an intentional prompt_cache_key mapping; Generic/DeepSeek receive no Hatfield correlation fields. Preserve Grok correlation only through an explicit provider-specific mapping if current behavior requires it. Remove SanitizedGenericModelClient, GenericProviderInternalOptionKeys, PlatformInvocationMetadata, internal option transport, obsolete subscribers/stripping, and superseded tests where callers are eliminated. Preserve explicitly caller-supplied ModelInvocationOptions.extraOptions as provider-facing options rather than adding a brittle exhaustive vendor option allowlist.

## Task workflow update - 2026-08-29T23:47:56.597Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context.
- Summary: Started with the finalized simplified design: Hatfield fully prepares invocations before Symfony AI and passes only provider-facing options across that boundary. No context registry or internal metadata in provider options.

## Task workflow update - 2026-08-29T23:48:11.275Z
- Ownership: owner=main; fork_run=none; revision=40598db94; scope=simplify Hatfield invocation preparation, provider routing/options, remove internal option transport and sanitizer, update deterministic provider/integration tests; outcome=assigned; commit=none

## Task workflow update - 2026-08-30T00:27:52.849Z
- Summary: Main implementation owner handed off the partially implemented migration after the design and core production changes were established. The fork now owns the remaining cohesive implementation, test migration, focused validation, and commit in the existing task worktree.
- Ownership: owner=main; fork_run=none; revision=40598db949c0d46336a28f578894b6058bf498b1; scope=simplify Hatfield invocation preparation, provider routing/options, remove internal option transport and sanitizer, and migrate deterministic tests; outcome=blocked; commit=none

## Task workflow update - 2026-08-30T00:28:23.613Z
- Recorded fork run: aoxowk8q991q
- Ownership: owner=fork; fork_run=aoxowk8q991q; revision=40598db949c0d46336a28f578894b6058bf498b1; scope=finish simplified invocation preparation migration, remove stale internal-option transport code, migrate deterministic provider/integration tests, run focused Castor validation, and commit; outcome=assigned; commit=none

## Task workflow update - 2026-08-30T00:41:12.574Z
- Recorded fork run: aoxowk8q991q
- Validation: Focused provider/adapter suite: PASS (137 tests, 662 assertions); castor test: PASS (4808 tests, 19439 assertions); castor deptrac: PASS (0 violations); castor phpstan: PASS (0 errors); castor cs-fix then castor cs-check: PASS; castor docs:validate: PASS; castor test:controller-replay: PASS (6 tests, 88 assertions); castor test:llm-real --filter='LlamaCppSmokeTest|PlatformIntegrationTest|TraceReplayTest': PASS (17 tests, 103 assertions); castor check: not run because task-start phase forbids the full gate
- Summary: Implementation complete at 62924ba55accd2e3e29fba7879ef72da5273d34d. Hatfield now prepares typed invocation state before crossing into Symfony AI and passes only provider-facing options. Removed the options-bag metadata transport, routing/request subscribers, provider registry, generic sanitizer/denylist, and superseded tests. Codex and Grok correlation use explicit prompt_cache_key mapping; Generic/DeepSeek omit Hatfield correlation fields. The worktree is clean. The fork confirmed it read and followed the testing skill and tests/AGENTS.md.
- Ownership: owner=fork; fork_run=aoxowk8q991q; revision=40598db949c0d46336a28f578894b6058bf498b1; scope=finish simplified invocation preparation migration, remove stale internal-option transport code, migrate deterministic provider/integration tests, run focused Castor validation, and commit; outcome=completed; commit=62924ba55accd2e3e29fba7879ef72da5273d34d

## Task workflow update - 2026-08-30T01:37:52.820Z
- Summary: Independent reviewer round 1 examined commit 62924ba55 against origin/main. Verdict was APPROVE WITH SUGGESTIONS and no critical/security blockers. The review confirmed the structural no-leak design and requirement mapping, but identified one ineffective provider-routing integration assertion and an unmapped Codex ephemeral-run correlation regression. Under the repository specification-fidelity and bad-test gates, these two findings will be corrected before transition.
- Review: role=reviewer; artifact=review-62924ba55-round1; target_revision=62924ba55accd2e3e29fba7879ef72da5273d34d; scope=full origin/main...HEAD correctness, security, design, deterministic proof, and specification-fidelity review; decision=APPROVE WITH SUGGESTIONS; blockers=ineffective owning-provider integration assertion and unmapped loss of stable Codex correlation for UUIDv7 ephemeral child runs

## Task workflow update - 2026-08-30T01:37:58.972Z
- Ownership: owner=main; fork_run=none; revision=62924ba55accd2e3e29fba7879ef72da5273d34d; scope=address reviewer round-1 blockers by preserving explicit Codex UUIDv7 child-run prompt_cache_key mapping and strengthening provider-routing integration proof; outcome=assigned; commit=none

## Task workflow update - 2026-08-30T01:41:51.313Z
- Validation: castor test --filter='SessionAwareModelResolverTest|PlatformIntegrationTest': PASS (31 tests, 104 assertions); castor deptrac: PASS (0 violations); castor phpstan: PASS (0 errors); castor cs-fix and castor cs-check: PASS; castor docs:validate: PASS; castor test:controller-replay: PASS (6 tests, 88 assertions); castor test:llm-real --filter='LlamaCppSmokeTest|PlatformIntegrationTest|TraceReplayTest': PASS (17 tests, 104 assertions)
- Summary: Reviewer round-1 follow-up completed at 919b88139. Codex UUIDv7 child runs now map their typed run identity explicitly to provider-facing prompt_cache_key when no persisted session row exists, preserving stable correlation without putting run_id in provider options. The provider-routing integration test now consumes the stream and proves the owning provider was selected. Stale option/storage documentation was corrected.
- Ownership: owner=main; fork_run=none; revision=62924ba55accd2e3e29fba7879ef72da5273d34d; scope=address reviewer round-1 blockers by preserving explicit Codex UUIDv7 child-run prompt_cache_key mapping and strengthening provider-routing integration proof; outcome=completed; commit=919b881397c4780412ffef2892eff5f8c3074097

## Task workflow update - 2026-08-30T01:58:18.692Z
- Summary: Independent reviewer round 2 approved HEAD 919b88139eee47d31df2e538f4f294c9c4956a95 with suggestions and explicitly reported no remaining blockers. The reviewer verified the Codex UUIDv7 child-run mapping, discriminating owning-provider integration proof, structural no-leak boundary, provider mappings, deleted-code cleanup, and deterministic coverage. Full castor check remains for the CODE-REVIEW transition.
- Correction: the exact reviewer-follow-up commit is 919b88139eee47d31df2e538f4f294c9c4956a95; the earlier appended full SHA for commit 919b88139 was inaccurate.
- Review: role=reviewer; artifact=pi-subagent-QxKF1C-round2; target_revision=919b88139eee47d31df2e538f4f294c9c4956a95; scope=full cumulative origin/main...HEAD correctness, security, dead-code, deterministic proof, round-1 fixes, and specification-fidelity follow-up; decision=APPROVE WITH SUGGESTIONS; blockers=none

## Task workflow update - 2026-08-30T02:00:03.503Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (88.0s).
- Pushed task/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context to origin.
- branch 'task/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context' set up to track 'origin/task/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context'.
- Created PR: https://github.com/ineersa/agent-core/pull/444
- Validation: Reviewer round 2: APPROVE WITH SUGGESTIONS, no blockers; castor test --filter='SessionAwareModelResolverTest|PlatformIntegrationTest': PASS (31 tests, 104 assertions); castor deptrac: PASS (0 violations); castor phpstan: PASS (0 errors); castor cs-check: PASS; castor docs:validate: PASS; castor test:controller-replay: PASS (6 tests, 88 assertions); castor test:llm-real --filter='LlamaCppSmokeTest|PlatformIntegrationTest|TraceReplayTest': PASS (17 tests, 104 assertions)
- Summary: Implementation and two independent review rounds complete. Reviewer approved HEAD 919b88139eee47d31df2e538f4f294c9c4956a95 with suggestions and no blockers. The branch removes Symfony AI options-bag metadata transport and generic sanitization, prepares typed invocation state before the provider boundary, uses explicit provider-specific correlation mapping, and strengthens deterministic routing/leak tests.

## Task workflow update - 2026-08-30T22:39:14.795Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context: ide_close_project returned isError.
- Merged task/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/settings.yaml                            |   4 +-
 config/services.yaml                               |  16 +-
 docs/session-storage.md                            |   2 +-
 .../Contract/Model/ProviderRegistryInterface.php   |  23 ---
 ...ProviderCompatibilityFeatureShaperInterface.php |   8 +-
 .../Domain/Model/ModelInvocationOptions.php        |   7 +-
 .../Domain/Model/ProviderRequestOptionKeys.php     |  43 -----
 src/AgentCore/Domain/Model/ResolvedModel.php       |  14 +-
 .../SymfonyAi/BeforeProviderRequestSubscriber.php  |  93 ----------
 .../SymfonyAi/DynamicToolDescriptionProcessor.php  |  33 ++--
 .../SymfonyAi/LlmPlatformAdapter.php               |  62 +++----
 .../SymfonyAi/ModelResolverRoutingSubscriber.php   |  97 -----------
 .../SymfonyAi/PlatformInvocationMetadata.php       |  61 -------
 .../SymfonyAi/PreparedInvocationPlatform.php       |  51 ++++++
 .../ProviderCompatibilityRequestShaper.php         |  89 ++--------
 .../SymfonyAi/ProviderRequestPreparedEvent.php     |  29 ++++
 .../SymfonyAi/ProviderRequestPreparer.php          |  80 +++++++++
 .../SymfonyAi/ReasoningContentFeatureShaper.php    |   3 +-
 .../SymfonyAi/ReasoningOptionsFeatureShaper.php    |  18 +-
 .../SymfonyAi/ZaiToolStreamFeatureShaper.php       |   3 +-
 .../PromptCacheDiagnosticsInvocationSubscriber.php | 106 ++----------
 .../Agent/Execution/SessionAwareModelResolver.php  |  40 +++--
 .../Extension/Agent/ConfiguredModelAgentRunner.php |  41 +++--
 .../Codex/CodexSymfonyAiProviderBuilder.php        |   1 +
 .../Grok/GrokSymfonyAiProviderBuilder.php          |   1 +
 .../SymfonyAi/ProjectedSymfonyModelCatalog.php     |   6 +-
 .../SymfonyAi/SymfonyAiProviderFactory.php         |  12 +-
 .../SymfonyAi/SymfonyAiProviderRegistry.php        |  52 ------
 .../Generic/GenericProviderInternalOptionKeys.php  |  34 ----
 .../Bridge/Generic/SanitizedGenericModelClient.php |  54 ------
 src/Platform/Bridge/Grok/GrokModelClient.php       |  12 +-
 .../OpenAICodex/CodexCorrelationProvenance.php     |  10 +-
 .../OpenAICodex/CodexCorrelationRequestId.php      |  37 +---
 .../Bridge/OpenAICodex/CodexModelClient.php        |   5 +-
 .../Bridge/OpenAICodex/CodexRequestBodyFactory.php |  56 ++----
 .../OpenAICodex/CodexWebSocketModelClient.php      |  12 +-
 .../DynamicToolDescriptionProcessorTest.php        |  69 +++-----
 .../Infrastructure/SymfonyAi/LlamaCppSmokeTest.php |  32 +---
 .../SymfonyAi/LlmPlatformAdapterTest.php           |  43 -----
 .../SymfonyAi/PlatformIntegrationTest.php          | 146 ++++++----------
 .../ProviderCompatibilityRequestShaperTest.php     | 189 +++------------------
 .../ReasoningContentFeatureShaperTest.php          |  15 +-
 .../Infrastructure/SymfonyAi/TraceReplayTest.php   |   2 -
 .../Execution/SessionAwareModelResolverTest.php    |  86 +++++++++-
 .../CLI/Session/SessionCacheInspectCommandTest.php |  57 +++----
 ...ConfiguredModelAgentRunnerContextWindowTest.php |  20 ++-
 .../SymfonyAi/ProjectedSymfonyModelCatalogTest.php |  31 ++--
 .../SymfonyAi/SymfonyAiProviderFactoryTest.php     |   2 +-
 .../SymfonyAi/SymfonyAiProviderRegistryTest.php    | 136 ---------------
 .../GenericCompletionsWireSanitizationTest.php     |  55 ------
 .../Generic/SanitizedGenericModelClientTest.php    |  78 ---------
 tests/Platform/Bridge/Grok/GrokModelClientTest.php |  10 +-
 .../OpenAICodex/CodexCorrelationRequestIdTest.php  |  77 +++------
 .../Bridge/OpenAICodex/CodexModelClientTest.php    | 117 ++-----------
 .../CodexWebSocketCachedModelClientTest.php        |  10 +-
 .../OpenAICodex/CodexWebSocketModelClientTest.php  |   9 +-
 56 files changed, 644 insertions(+), 1755 deletions(-)
 delete mode 100644 src/AgentCore/Contract/Model/ProviderRegistryInterface.php
 delete mode 100644 src/AgentCore/Domain/Model/ProviderRequestOptionKeys.php
 delete mode 100644 src/AgentCore/Infrastructure/SymfonyAi/BeforeProviderRequestSubscriber.php
 delete mode 100644 src/AgentCore/Infrastructure/SymfonyAi/ModelResolverRoutingSubscriber.php
 delete mode 100644 src/AgentCore/Infrastructure/SymfonyAi/PlatformInvocationMetadata.php
 create mode 100644 src/AgentCore/Infrastructure/SymfonyAi/PreparedInvocationPlatform.php
 create mode 100644 src/AgentCore/Infrastructure/SymfonyAi/ProviderRequestPreparedEvent.php
 create mode 100644 src/AgentCore/Infrastructure/SymfonyAi/ProviderRequestPreparer.php
 delete mode 100644 src/CodingAgent/Infrastructure/SymfonyAi/SymfonyAiProviderRegistry.php
 delete mode 100644 src/Platform/Bridge/Generic/GenericProviderInternalOptionKeys.php
 delete mode 100644 src/Platform/Bridge/Generic/SanitizedGenericModelClient.php
 delete mode 100644 tests/CodingAgent/Infrastructure/SymfonyAi/SymfonyAiProviderRegistryTest.php
 delete mode 100644 tests/Platform/Bridge/Generic/GenericCompletionsWireSanitizationTest.php
 delete mode 100644 tests/Platform/Bridge/Generic/SanitizedGenericModelClientTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-23-remove-sanitized-generic-model-client-and-redesign-invocation-context.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #444 state: MERGED; Merge commit: ccc97df8352a430f37004adfbad6b5a657165e67
- Summary: PR #444 was merged on GitHub at 2026-08-30T22:38:35Z with merge commit ccc97df8352a430f37004adfbad6b5a657165e67. Moving the tracked task to DONE and cleaning the task worktree.

## Task workflow update - 2026-08-30T22:41:02.541Z
- Validation: GitHub PR #444: MERGED at 2026-08-30T22:38:35Z; LLM_MODE=true castor check: PASS (180.5s), QA run qa-20260830-223919-255547-28f0d6df; Unit/integration lane: PASS (4809 tests, 19441 assertions); Controller replay, TUI, llm-real, deptrac, phpstan, cs-check, docs validation, and catalog version lanes: PASS; QA artifact integrity, leak check, cache cleanup, and llama-proxy cache guard: PASS; Integration checkout git status: clean; Task worktree: removed
- Summary: Task completed after PR #444 merged. Integration checkout merged and pulled the task branch, full post-merge QA passed, Git status is clean, and the task worktree was removed. JetBrains project close reported a degraded IDE close response, but worktree and IDEA exclusion cleanup both completed.

## Task workflow update - 2026-09-06T15:40:33+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
