# Show LLM prompt-cache usage in footer/session stats

## Context
Hatfield already collects model token usage through Symfony AI and projects it into the TUI footer (`UsageProjection` / `FooterStateSegmentProvider`). `LlmPlatformAdapter::extractUsage()` already reads Symfony AI `TokenUsageInterface` and emits token usage fields, including `cached_tokens` today.

User wants cache-hit visibility similar to a `/cache` command idea, but surfaced near the existing footer input/output stats as a per-session cache percentage.

Target providers for this task are the providers the user actually uses: **OpenAI, DeepSeek, and z.ai**. Anthropic is explicitly out of scope unless support falls out naturally from Symfony AI with no provider-specific work.

## Goal
Collect provider-reported prompt-cache telemetry for OpenAI, DeepSeek, and z.ai where available, normalize it into Hatfield usage events/projections, and show a concise cache-read hit percentage in the footer near input/output token stats.

## Key implementation notes
- Do not estimate cache hits by local prompt hashing for v1. Use provider-reported usage telemetry only.
- Symfony AI `TokenUsageInterface` already exposes:
  - `getCachedTokens()`
  - `getCacheCreationTokens()`
  - `getCacheReadTokens()`
- Current `LlmPlatformAdapter::extractUsage()` emits `cached_tokens`, but should also emit normalized fields for cache read/write when available, e.g. `cache_read_tokens`, `cache_creation_tokens`.
- `UsageProjection` should accumulate cache-read and cache-creation tokens across the session and preserve latest-turn values if useful.
- Footer should display a compact session cache-read hit rate, e.g. `↻ 78%`, only when cache telemetry is present.
- Define metric clearly as cache-read/input-token percentage, not generic cache efficiency.

## Symfony AI extractor caveat
Provider support depends on Symfony AI bridge extractors. Existing Symfony AI code appears to map some cache fields:
- OpenAI Responses: `usage.input_tokens_details.cached_tokens` → `cachedTokens`
- Generic completions: `usage.num_cached_tokens` → `cachedTokens`

Provider-specific fields to investigate and support for the user's providers:
- OpenAI/OpenAI-compatible: `usage.input_tokens_details.cached_tokens` and any Chat Completions equivalent exposed by Symfony AI.
- DeepSeek: likely `prompt_cache_hit_tokens` and `prompt_cache_miss_tokens`.
- z.ai: verify actual OpenAI-compatible usage payload shape; patch only if it exposes cache telemetry that Symfony AI currently drops.

Anthropic-specific cache fields are not a target for this task.

If Symfony AI does not map OpenAI/DeepSeek/z.ai cache fields into `TokenUsageInterface`, this task may need to patch/extend the relevant Symfony AI usage extractor or Hatfield bridge layer so the metadata reaches `LlmPlatformAdapter::extractUsage()`.

## Suggested affected areas
- `src/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapter.php`
- `src/Tui/Runtime/UsageProjection.php`
- `src/Tui/Listener/FooterStateSegmentProvider.php`
- Runtime event/translator tests if usage payload expectations need updates
- Symfony AI bridge/extractor layer if provider cache fields are not currently surfaced
- Footer/TUI docs if token/footer stat semantics are documented

## Validation notes
Follow AGENTS.md testing rules:
- load the `testing` skill and read `tests/AGENTS.md` before writing/running tests;
- use Castor only;
- add focused projection/footer tests for usage payloads containing cache fields;
- if footer rendering changes are user-visible TUI behavior, include or update real `TmuxHarness` E2E proof as required by project rules;
- run deterministic `castor check` before CODE-REVIEW;
- run `castor test:llm-real` only if provider/LLM-visible bridge compatibility changes require live validation.

## Acceptance criteria
- LLM usage extraction emits normalized cache telemetry fields when Symfony AI exposes them (`cached_tokens`, `cache_read_tokens`, `cache_creation_tokens` or equivalent normalized names).
- `UsageProjection` accumulates session cache-read/cache-creation token totals and can compute a cache-read hit percentage safely.
- Footer displays a concise cache-hit/cache-read percentage near existing input/output token stats only when telemetry exists; it does not show misleading 0% when provider telemetry is absent.
- Provider-specific extractor gap is investigated for OpenAI, DeepSeek, and z.ai; if Symfony AI does not surface their cache fields, implementation either patches the relevant extractor/bridge or records the unsupported-provider limitation explicitly.
- Tests cover usage payloads with OpenAI-style cached tokens, DeepSeek-style cache hit/miss tokens if supported/implemented, z.ai payload shape if cache telemetry exists, and absent telemetry.
- Project QA uses Castor only and passes required validation, including TUI E2E proof if footer behavior is changed.

## Workflow metadata
Status: DONE
Branch: task/usage-cache-footer-stats
Worktree: /home/ineersa/projects/agent-core-worktrees/usage-cache-footer-stats
Fork run: 6w126qeehxbq
PR URL: https://github.com/ineersa/agent-core/pull/179
PR Status: merged
Started: 2026-06-19T19:08:24.117Z
Completed: 2026-06-19T22:35:19.814Z

## Work log
- Created: 2026-06-19T18:11:46.623Z
- Updated: narrowed provider scope to OpenAI, DeepSeek, and z.ai; Anthropic-specific support is out of scope.

## Task workflow update - 2026-06-19T19:08:24.117Z
- Moved TODO → IN-PROGRESS.
- Created branch task/usage-cache-footer-stats.
- Created worktree /home/ineersa/projects/agent-core-worktrees/usage-cache-footer-stats.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/usage-cache-footer-stats.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/usage-cache-footer-stats.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/usage-cache-footer-stats.
- Summary: Claimed for task-start. Loaded task-workflow and testing skills, read tests/AGENTS.md before preparing implementation/testing instructions.

## Task workflow update - 2026-06-19T19:15:54.833Z
- Context gathered for implementation in worktree /home/ineersa/projects/agent-core-worktrees/usage-cache-footer-stats.
- Mandatory docs loaded by orchestrator: .agents/skills/task-workflow/SKILL.md, .agents/skills/testing/SKILL.md, tests/AGENTS.md, subagents skill.
- Scout findings: current flow is LlmPlatformAdapter::extractUsage() -> RuntimeEventTranslator passthrough -> RuntimeEventPoller -> UsageProjection::accumulate() -> FooterStateSegmentProvider. Current payload has cached_tokens only; UsageProjection/footer do not track cache yet.
- Research findings: installed Symfony AI generic extractor maps only num_cached_tokens; OpenResponses maps input_tokens_details.cached_tokens. DeepSeek uses usage.prompt_cache_hit_tokens / prompt_cache_miss_tokens; z.ai uses OpenAI-compatible usage.prompt_tokens_details.cached_tokens; in-scope providers expose cache-hit/read tokens but not creation/read split except direct TokenUsageInterface fields when present.
- TUI proof target: extend replay-backed TmuxHarness coverage (likely TuiJourneyE2eTest phase after model interaction) with fixture/cache usage flowing through FixtureTokenUsage and asserting visible footer segment such as ↻ 78%.

## Task workflow update - 2026-06-19T19:17:14.705Z
- Recorded fork run: da5kw3y0evq5
- Implementation fork launched in background on worktree /home/ineersa/projects/agent-core-worktrees/usage-cache-footer-stats with required cache telemetry, footer, focused tests, and real TmuxHarness E2E proof instructions. Fork run: da5kw3y0evq5.

## Task workflow update - 2026-06-19T19:39:12.131Z
- Recorded fork run: da5kw3y0evq5
- Validation: Fork reported reading .agents/skills/testing/SKILL.md and tests/AGENTS.md before implementation.; Fork reported: castor test --filter=UsageProjectionTest OK (18 tests, 71 assertions).; Fork reported: castor test --filter=FooterStateSegmentProviderTest OK (7 tests, 38 assertions).; Fork reported: castor test --filter=DurableResultConverterTest OK (20 tests, 116 assertions).; Fork reported: castor test --filter=PlatformIntegrationTest OK (9 tests, 48 assertions).; Fork reported combined focused test filter OK (54 tests, 273 assertions).; Fork reported: castor test:tui --filter=TuiJourneyE2eTest OK (1 test, 29 assertions), satisfying real TmuxHarness E2E proof requirement.; Fork reported: castor test:llm-real --filter=LlamaCppSmokeTest OK (1 test, 8 assertions).; Fork reported: castor deptrac OK (0 violations).; Fork reported: castor phpstan OK (0 errors; noted one harmless nullable-interface-return warning).; Fork reported: castor cs-fix + castor cs-check clean.; Orchestrator verification: git status --porcelain clean; git log shows HEAD e7f0e64e7; git diff --stat main...HEAD shows expected 13 files changed; grep/read confirmed TUI E2E assertion '↻ 78%' in TuiJourneyE2eTest.
- Summary: Implementation fork completed and was verified by orchestrator. Commit e7f0e64e7 (feat: add prompt-cache hit percentage to TUI footer) exists on branch task/usage-cache-footer-stats; worktree status is clean. Diff stat from main shows expected 13 files changed: LlmPlatformAdapter usage extraction, generic PromptCacheTokenUsageExtractor/DurableResultConverter bridge, UsageProjection, FooterStateSegmentProvider, replay infrastructure, TUI fixture/E2E, and focused tests. Verified real TmuxHarness E2E proof in tests/Tui/E2E/TuiJourneyE2eTest.php asserts visible footer text '↻ 78%' after replay-backed model interaction, with fixture cache telemetry in tests/Tui/E2E/fixtures/tui-simple-text-response.json. Accepted fork handoff as complete for task-start phase. No PR/review/check gate was run by orchestrator.

## Task workflow update - 2026-06-19T20:13:34.298Z
- Validation: Manual real-provider smoke: z.ai/glm-5.1 displayed footer cache segment `↻ 60%` with normal token/context/throughput segments.; Manual smoke: llama.cpp worked.; Manual smoke note: vLLM showed `0%`; not a target provider for this task and likely indicates zero reported cache-read tokens rather than cache-footer failure.; Separate issue created for unrelated OpenAI Codex continuation null-content bug: https://github.com/ineersa/agent-core/issues/177.
- Summary: Additional manual provider smoke from user: z.ai real-provider footer telemetry works. User observed footer line `◆ glm-5.1 | 20.8k/800 $0.00 ↻ 60% 4% 8.2k/200.0k ⚡ 37.1 t/s =)`, confirming real z.ai/glm-5.1 cache-hit percentage display. User also noted llama.cpp works; vLLM shows 0% likely because it reports zero cached tokens or no provider-side cache hit. Separate OpenAI Codex continuation failure was investigated by fork k8v8ioilys1n and filed as GitHub issue #177; judged unrelated to this cache-footer task.

## Task workflow update - 2026-06-19T20:16:27.974Z
- Validation: Manual real-provider smoke: deepseek/deepseek-v4-flash displayed footer cache segment `↻ 64%` with normal token/context/throughput segments.
- Summary: Additional manual provider smoke from user: DeepSeek real-provider cache telemetry works. User observed footer line `◆ deepseek-v4-flash | 23.3k/2.3k $0.00 ↻ 64% 1% 8.7k/1000.0k ⚡ 83.8 t/s`, confirming DeepSeek `prompt_cache_hit_tokens` path and footer segment in real usage.

## Task workflow update - 2026-06-19T20:32:02.909Z
- Recorded fork run: cxcz7pizmqqq
- Reviewer subagent returned APPROVE WITH SUGGESTIONS. Launched fix fork cxcz7pizmqqq to address sensible low-risk suggestions: clarify DeepSeek prompt_cache_miss_tokens is not cache creation, document reasoning token extraction and cached_tokens fallback/session-cumulative semantics, make footer priority test less implementation-mirroring, and make TUI E2E cache assertion wait for visible segment.

## Task workflow update - 2026-06-19T20:37:13.248Z
- Recorded fork run: cxcz7pizmqqq
- Validation: Fork confirmed it read .agents/skills/testing/SKILL.md and tests/AGENTS.md before editing/running tests.; castor test --filter=FooterStateSegmentProviderTest OK (7 tests, 40 assertions).; castor test --filter=UsageProjectionTest OK (18 tests, 71 assertions).; castor test --filter=DurableResultConverterTest OK (20 tests, 116 assertions).; castor test:tui --filter=TuiJourneyE2eTest OK (1 test, 29 assertions).; castor phpstan --path=src/Platform/Bridge/Generic OK (0 errors; 1 ignorable/pre-existing noted).; castor phpstan --path=src/Tui/Runtime OK (0 errors).; castor phpstan --path=tests/Tui/Listener OK (0 errors; unrelated ignorable file_errors reported by Castor).; castor phpstan --path=tests/Tui/E2E OK (0 errors; unrelated ignorable warnings unchanged).; castor cs-check clean.
- Summary: Review-fix fork completed. Commit 04b0f681c applied reviewer suggestions without changing production behavior: documented DeepSeek prompt_cache_miss_tokens exclusion and reasoning token extraction, clarified UsageProjection cached_tokens fallback and session-cumulative hit %, replaced magic context priority test with ordering assertion, and made TUI E2E wait for visible `↻ 78%` segment. Worktree reported clean.

## Task workflow update - 2026-06-19T21:00:39.884Z
- Recorded fork run: ppciojg71jvz
- During task-to-pr focused validation, `castor test`, `castor deptrac`, and `castor cs-check` passed, but full `castor phpstan` exited nonzero due one return-type warning in DurableResultConverter::getTokenUsageExtractor(). Launched fork ppciojg71jvz for minimal signature fix and validation.

## Task workflow update - 2026-06-19T21:02:17.223Z
- Recorded fork run: vy8vwcfwpuwx
- Previous PHPStan-fix fork ppciojg71jvz reported commit 11bd7f4ad, but orchestrator verification found HEAD still at 04b0f681c and DurableResultConverter still declared nullable return type; `git show 11bd7f4ad` failed. Full validation still failed at `castor phpstan` with the same warning. Launched replacement fork vy8vwcfwpuwx with strict cwd/git verification instructions.

## Task workflow update - 2026-06-19T21:05:01.531Z
- Recorded fork run: vy8vwcfwpuwx
- Validation: Fork confirmed it read .agents/skills/testing/SKILL.md and tests/AGENTS.md.; castor phpstan --path=src/Platform/Bridge/Generic OK (0 errors, 0 file_errors).; castor phpstan OK (0 errors, 0 file_errors).; castor cs-check clean.; castor test --filter=DurableResultConverterTest OK (20 tests, 116 assertions).
- Summary: Replacement PHPStan-fix fork completed successfully. Actual current HEAD commit 242b4edab removes nullable `?` from DurableResultConverter::getTokenUsageExtractor() return type, matching its non-null behavior. Worktree reported clean.

## Task workflow update - 2026-06-19T21:06:52.955Z
- Validation: Reviewer verdict: APPROVE WITH SUGGESTIONS — clear to move to CODE-REVIEW.; Focused validation at current HEAD 242b4edab: castor test OK (2849 tests, 8613 assertions).; castor deptrac OK (0 violations, 0 errors).; castor phpstan OK (0 errors, 0 file_errors).; castor cs-check clean (0 files fixed).; castor test:tui OK (11 tests, 101 assertions), satisfying TUI E2E proof gate.; castor test:llm-real OK; llama.cpp generation preflight ok; PHPUnit OK (5 tests, 51 assertions).; Stale worker scan noted one root-owned `php bin/console messenger:consume --all --exclude-receivers=failed` process (PID 3361) could not be killed by this user; validation still passed with isolated test infrastructure.
- Summary: Reviewer re-run after follow-up fixes returned APPROVE WITH SUGGESTIONS and explicitly stated clear to move to CODE-REVIEW. Mandatory TUI gate passed: reviewer verified real replay-backed `TmuxHarness` E2E proof in `tests/Tui/E2E/TuiJourneyE2eTest.php` with `#[Group('tui-e2e-replay')]`, actual replay SSE usage path, UsageProjection/FooterStateSegmentProvider, and visible `↻ 78%` assertion. Prior suggestions were addressed by 04b0f681c; PHPStan follow-up fixed by 242b4edab. Remaining reviewer notes are non-blocking follow-ups/nice-to-haves (not CODE-REVIEW blockers), especially cost double-count risk when paid cache pricing is configured.

## Task workflow update - 2026-06-19T21:08:27.663Z
- Recorded fork run: 6w126qeehxbq
- First CODE-REVIEW move ran deterministic castor check and failed only in `test:tui`: TuiStartupSnapshotTest expected footer elapsed `⏱ 0s`, got `⏱ 1s`. Launched fork 6w126qeehxbq to fix test-side snapshot normalization for dynamic elapsed footer time and rerun TUI validation.

## Task workflow update - 2026-06-19T21:11:58.825Z
- Recorded fork run: 6w126qeehxbq
- Validation: Fork confirmed it read .agents/skills/testing/SKILL.md and tests/AGENTS.md.; castor test:tui --filter=TuiStartupSnapshotTest OK (2 tests, 7 assertions).; castor test:tui OK (11 tests, 101 assertions).; castor cs-check clean.; castor phpstan --path=tests/Tui/E2E: 0 new errors; only pre-existing ignorable file_errors reported by fork.
- Summary: Fork 6w126qeehxbq completed deterministic TUI snapshot timing-flake fix. Commit 6be1e31ae updates test-only snapshot normalization in tests/Tui/E2E/TmuxHarness.php so elapsed footer time `⏱ Ns` normalizes to stable `⏱ 0s`, and footer line detection includes `⏱`/`↻`. Worktree reported clean.

## Task workflow update - 2026-06-19T21:13:12.913Z
- Validation: castor check OK (quality: ok, 125.7s).; deptrac OK (1.9s).; test OK (2845 tests, 8601 assertions, 13.4s).; test:controller-replay OK (3 tests, 41 assertions, 28.1s).; test:tui OK (11 tests, 101 assertions, 54.7s).; phpstan OK (0 errors, 0 file_errors, 27.0s).; cs-check OK (0.7s).
- Summary: After fork 6w126qeehxbq, orchestrator verified worktree clean at HEAD 6be1e31ae and ran full deterministic `castor check`; all lanes passed. Current-user stale worker preflight found none; root-owned PID 3361 was observed and intentionally left untouched per AGENTS.md.

## Task workflow update - 2026-06-19T21:14:18.982Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (51.4s).
- Pushed task/usage-cache-footer-stats to origin.
- branch 'task/usage-cache-footer-stats' set up to track 'origin/task/usage-cache-footer-stats'.
- Created PR: https://github.com/ineersa/agent-core/pull/179

## Task workflow update - 2026-06-19T22:35:19.814Z
- Moved CODE-REVIEW → DONE.
- Merged task/usage-cache-footer-stats into integration checkout.
- Merge made by the 'ort' strategy.
 .../SymfonyAi/LlmPlatformAdapter.php               |  10 ++
 .../Bridge/Generic/DurableResultConverter.php      |  21 ++-
 .../Generic/PromptCacheTokenUsageExtractor.php     | 132 +++++++++++++++++
 src/Tui/Listener/FooterStateSegmentProvider.php    |  17 ++-
 src/Tui/Runtime/UsageProjection.php                |  83 ++++++++++-
 .../SymfonyAi/PlatformIntegrationTest.php          | 148 ++++++++++++++++++-
 .../SymfonyAi/Replay/FixtureReplayModelClient.php  |  20 ++-
 .../Replay/ControllerReplayHttpClientFactory.php   |  24 +++-
 .../Bridge/Generic/DurableResultConverterTest.php  | 147 +++++++++++++++++++
 tests/Tui/E2E/TmuxHarness.php                      |  11 +-
 tests/Tui/E2E/TuiJourneyE2eTest.php                |  21 +++
 .../Tui/E2E/fixtures/tui-simple-text-response.json |   4 +-
 .../Listener/FooterStateSegmentProviderTest.php    |  91 ++++++++++++
 tests/Tui/Runtime/UsageProjectionTest.php          | 159 +++++++++++++++++++++
 14 files changed, 868 insertions(+), 20 deletions(-)
 create mode 100644 src/Platform/Bridge/Generic/PromptCacheTokenUsageExtractor.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/usage-cache-footer-stats.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/usage-cache-footer-stats.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User confirmed PR #179 was merged. Moving task to DONE and cleaning up task worktree.
