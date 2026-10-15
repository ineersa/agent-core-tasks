# Correct OpenAI Responses cache-write cost accounting

## Goal
OpenAI documents usage.input_tokens_details.cache_write_tokens for GPT-5.6 and later, priced at 1.25 times the applicable uncached input rate. Cache reads and writes are included in input_tokens, not added on top.

Investigation on 2026-10-09 found no matching open issue or PR in symfony/ai using cache_write_tokens, "cache write", "cache creation", GPT-5.6, and OpenResponses cache searches. Upstream main revision 55278a85fc6496bcd34da8cfd89bb787594d025f drops cache_write_tokens in src/platform/src/Bridge/OpenResponses/TokenUsageExtractor.php. Non-streaming extraction and streamed response usage, including response.completed, delegate to fromDataArray(). An upstream issue draft was provided to the user, but no issue was filed.

Hatfield main d0f13d86db1ee60d40b26a174635be61f6198cfd is affected:
- vendor/ineersa/symfony-ai-openai-codex-platform/src/ResultConverter.php also uses the OpenResponses extractor for streamed usage.
- src/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapter.php::extractUsage() forwards getCacheCreationTokens() as cache_creation_tokens, but the extractor leaves it null.
- src/CodingAgent/Config/Ai/AiCostCalculator.php already partitions total input into ordinary, cache-read, and cache-write tokens and supports a separate write rate. With the write count missing, those tokens are charged at the ordinary input rate instead.
- config/ai-catalog.yaml already contains cache_write rates for GPT-5.6 and newer Codex models. ProvidersUpdateCommand also imports cache_write from models.dev. Audit applicable model rates and context tiers against OpenAI pricing before changing them; do not assume a new pricing field or calculator rewrite is necessary.

Scope: restore cache-write telemetry through the existing Symfony AI facilities, adopt the upstream fix/dependency update when available, audit bundled and refreshed catalog prices, and change calculation only where evidence identifies a gap. Preserve the distinction between subscription cost estimates and actual API billing. No vendor edits or custom compatibility fallback without approval. Investigate inconsistent provider counters before deciding whether the current clamping behavior needs changing. Do not claim historical sessions can recover counts that were never persisted.

References:
- https://developers.openai.com/api/docs/guides/prompt-caching
- https://developers.openai.com/api/docs/pricing
- https://github.com/symfony/ai/blob/55278a85fc6496bcd34da8cfd89bb787594d025f/src/platform/src/Bridge/OpenResponses/TokenUsageExtractor.php
- https://github.com/symfony/ai/blob/55278a85fc6496bcd34da8cfd89bb787594d025f/src/platform/src/Bridge/OpenResponses/ResultConverter.php

Proposed ownership: one implementation fork for the dependency/usage/catalog slice, with independent review. Run focused Castor validation during implementation; the CODE-REVIEW transition owns the full castor check gate.

## Acceptance criteria
- OpenResponses cache_write_tokens reaches Hatfield usage events as cache_creation_tokens through non-streaming and streamed usage paths, including the Codex transport paths that use the same extractor.
- Verify applicable cache-write rates and context pricing tiers against provider documentation; correct catalog entries and refresh behavior only where needed.
- Cost accounting bills each documented input token class once: ordinary input = input_tokens - cached_tokens - cache_write_tokens; cache writes use their model-specific rate.
- Regression coverage verifies positive, zero, and absent write counts, mixed reads and writes, and the resulting Hatfield cost without double billing or regressions to other providers.
- Document any unresolved provider-counter inconsistencies and distinguish API-priced estimates from subscription charges; focused validation passes through Castor.

## Workflow metadata
Status: TODO
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-10-09T18:22:25+00:00
