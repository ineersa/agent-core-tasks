# Extract Codex Symfony AI platform into a standalone Composer package

## Goal
## Motivation

The OpenAI Codex bridge has grown into a substantial Symfony AI platform implementation inside agent-core: roughly 30 production files / 2,944 lines plus about 17 focused bridge tests. It now owns SSE, WebSocket, cached WebSocket continuation, UUIDv7 correlation, request shaping, result conversion, contracts/normalizers, cache lifecycle, and bounded token refresh integration.

The bridge already follows Symfony AI platform package conventions:

- namespace `Symfony\\AI\\Platform\\Bridge\\OpenAICodex`
- static `Factory::createProvider()` / `Factory::createPlatform()` boundary
- `ModelClientInterface`, `ResultConverterInterface`, model/catalog and contract normalizers
- dedicated PSR-4 production and test mappings

Only two agent-core production integration points directly import the bridge: `CodexSymfonyAiProviderBuilder` and `CodexWebSocketWorkerShutdownSubscriber`.

## Proposed boundary

Create a separately versioned Composer package, tentatively `ineersa/symfony-ai-openai-codex-platform`, shaped like the official Symfony AI platform packages. The package owns the provider/transport implementation only. Hatfield remains the composition root and retains OAuth credential storage/login, provider config/model projection, session provider-cache identity, retry HTTP decoration, and Messenger worker lifecycle integration.

### Move to package

- all `src/Platform/Bridge/OpenAICodex/` production classes
- matching bridge tests from `tests/Platform/Bridge/OpenAICodex/`
- Factory, model/model catalog, request body factory, contract normalizers
- SSE and WebSocket clients, connectors, result conversion
- WebSocket connection cache, leases, continuation comparison/state, TTL/max-age handling
- UUIDv7 correlation and prompt-cache-key mapping
- package-owned cancellation/abort seam required by reusable WebSocket results

### Remain in agent-core

- `CodingAgent/Auth/*` Codex OAuth/storage/refresh classes and `CodexAuthCommand`
- `CodexSymfonyAiProviderBuilder` as Hatfield adapter/composition code
- `CodexWebSocketWorkerShutdownSubscriber` as Messenger lifecycle adapter
- `AiProviderConfig`, model projection, reasoning resolution, and settings YAML/docs
- `HatfieldSession::$providerCacheKey` and `SessionAwareModelResolver`
- generic retry HTTP client and generic-provider wire sanitization
- AgentCore `LlmPlatformAdapter`

## Known extraction seams

1. Replace the bridge's dependency on `Ineersa\\Platform\\Result\\CancellableRawResultInterface` with a package-owned/public abort contract or another explicit lifecycle seam that Hatfield can consume without reverse dependency.
2. Remove Hatfield-specific defaults (`originator`, User-Agent) from package internals; require/configure them through Factory arguments.
3. Keep Hatfield internal option names out of package policy where possible. Define explicit typed/request options or configurable internal-key consumption for `provider_cache_key`, `run_id`, `_agent_core_invocation`, `_hatfield_reasoning`, `tools_ref`, and `turn_no`.
4. Keep the connection cache DI-owned and process-scoped; the package exposes lifecycle methods while Hatfield's Messenger subscriber calls `closeAll()`.
5. Account for Symfony AI `^0.10` pre-1.0 churn. Package releases and agent-core dependency bumps must be synchronized and tested; extraction must not imply a stable API promise that upstream does not provide.
6. Decide repository strategy before implementation: standalone repository immediately versus a temporary Composer path repository used only to bootstrap the split. Final state must be separately versioned and installable, not merely another source directory in agent-core.

## Suggested phases

1. Record package namespace/name, repository, public API, dependency constraints, and cancellation seam.
2. Create package and move bridge production/tests without behavior changes.
3. Integrate package into agent-core through Composer and Hatfield adapters.
4. Remove agent-core's OpenAICodex PSR-4 mappings and in-repo bridge files/tests.
5. Validate package independently and run full provider/runtime validation in agent-core.

## Risks

- Symfony AI bridge internals are pre-1.0 and may change between releases.
- Cached WebSocket lifecycle must retain one shared DI-owned cache per long-lived worker.
- 401 refresh closure, account identity, UUIDv7 correlation, continuation deltas, privacy-safe error conversion, and idle reconnect behavior are easy to regress across package boundaries.
- A package that also owns Hatfield OAuth/config/session concepts would recreate the coupling rather than remove it.

No implementation should start until the package destination/name and cancellation seam are explicitly agreed.

## Acceptance criteria
- A package-boundary decision is documented: package/repository name, namespace, supported Symfony AI versions, release strategy, and whether bootstrap uses a temporary Composer path repository.
- The standalone package contains the complete OpenAICodex Symfony AI provider/transport implementation and its focused tests, with no dependency on `Ineersa\\AgentCore`, `Ineersa\\CodingAgent`, Hatfield settings/session entities, Symfony Messenger, or Hatfield auth storage.
- Hatfield-specific originator and User-Agent values are supplied by agent-core integration rather than hardcoded as package defaults.
- The cancellation/abort lifecycle uses a package-owned public contract or equally explicit dependency-neutral seam; AgentCore can still abort an active WebSocket stream deterministically.
- SSE, one-shot WebSocket, cached WebSocket continuation, busy one-shot fallback, UUIDv7 correlation, bounded 401 refresh, prompt-cache identity, privacy-safe errors, idle TTL/max-age reconnect, and worker-shutdown cleanup preserve existing behavior.
- Codex OAuth login/storage/refresh policy, provider configuration, session provider-cache-key generation, retry HTTP decoration, and Messenger shutdown subscription remain in agent-core adapters.
- Agent-core consumes the package through Composer and removes the in-repo OpenAICodex production/test PSR-4 mappings and duplicated bridge files.
- Package-level tests pass using package-local helpers; agent-core focused provider tests, `castor test`, `castor deptrac`, `castor phpstan`, `castor cs-check`, `castor test:llm-real`, and deterministic `castor check` pass.
- A live GPT-5.6 smoke proves first-turn connection creation, same-session cached continuation reuse, and post-idle fresh reconnect with full context after extraction.
- Package and agent-core documentation describe ownership, configuration, lifecycle, and synchronized upgrade expectations for Symfony AI pre-1.0 changes.

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
- Created: 2026-07-13T23:18:48.978Z

## Task workflow update - 2026-07-13T23:34:31.986Z
- Summary: User clarified package scope: OAuth login, credential storage, and refresh policy are conceptually part of the Codex integration, but they will remain outside the standalone package for v1. V1 extracts the Symfony AI provider/transport implementation and preserves a dependency-neutral token-refresh callback/credential seam; a later version may add an optional OAuth/auth companion module without coupling the transport core to Hatfield paths, Console UI, browser launching, or secret-storage policy.
- Scope decision: v1 excludes OAuth implementation while treating it as a legitimate future package concern, not permanently Hatfield-owned architecture. Design v1 APIs so OAuth can move later without changing the provider transport boundary.
