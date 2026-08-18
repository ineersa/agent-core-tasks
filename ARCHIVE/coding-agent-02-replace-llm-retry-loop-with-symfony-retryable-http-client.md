# coding-agent-02: Replace the LLM retry loop with Symfony RetryableHttpClient

## Goal
## Goal
Delete the custom blocking HTTP retry loop and express Hatfield's provider-specific retry policy through Symfony HttpClient's native retry strategy seam.

## Architecture report evidence (candidate 2)
- `src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmRetryingHttpClient.php` manually implements attempts, body inspection, cancellation, Retry-After handling, and `usleep()` inside `request()`.
- Blocking `usleep()` is not cancellable and reduces concurrency.
- Installed `Symfony\Component\HttpClient\RetryableHttpClient` already owns retry orchestration.
- Its `RetryStrategyInterface::shouldRetry()` can return `null` to request body-fetch classification, and `getDelay()` receives `AsyncContext`, allowing custom headers such as `retry-after-ms`.
- Symfony uses `$context->pause()` rather than blocking `usleep()`.
- `LlmHttpRetryPolicy` contains real product knowledge—terminal billing/quota body patterns and retry classification—and should remain as policy, adapted to Symfony's strategy contract rather than rewritten.

## Smallest viable direction
- Replace loop mechanics with `RetryableHttpClient` plus a minimal `RetryStrategyInterface` implementation/adaptation.
- Collapse the decorator to direct factory/DI wiring where possible; do not wrap Symfony with another generic retry framework.
- Preserve configured attempt limits, retryable statuses/body classifications, Retry-After and `retry-after-ms` behavior, cancellation, logging privacy, and terminal billing/quota behavior.

## Scope boundaries
- No new Composer dependency.
- No provider/model routing or public settings changes.
- No raw prompts, response bodies, credentials, or headers containing secrets in logs.
- Do not alter retry policy merely to fit Symfony defaults; map explicit existing semantics or surface a decision before implementation.

## Test impact noted by the report
Replace tests that mirror custom loop internals with focused strategy/policy behavior. Keep policy regression coverage, including body-dependent terminal errors and delay headers. Because this is provider/LLM-visible infrastructure, validation must include focused live LLM proof and the full required gate.

## Acceptance criteria
- `LlmRetryingHttpClient`'s manual attempt loop and blocking `usleep()` are removed.
- Symfony `RetryableHttpClient` owns orchestration and cancellable pauses; Hatfield-specific decisions live in a minimal `RetryStrategyInterface` policy adapter.
- Existing attempt limits, status/body classification, terminal billing/quota handling, Retry-After, `retry-after-ms`, cancellation, and failure propagation remain behaviorally equivalent.
- No new generic retry abstraction, setting, dependency, or public API is introduced.
- Behavior-focused tests cover retry/no-retry/body-fetch and delay decisions without re-testing Symfony internals.
- The fork and handoff state that `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` were read and followed.
- Focused Castor tests, `castor test:llm-real`, `castor test:controller-replay`, `castor deptrac`, `castor phpstan`, `castor cs-check`, and final `castor check` pass.

## Workflow metadata
Status: ARCHIVE
Branch: task/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client
Worktree: /home/ineersa/projects/agent-core-worktrees/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/398
PR Status: merged
Started: 2026-08-16T22:07:40.207Z
Completed: 2026-08-17T01:03:56.396Z

## Work log
- Created: 2026-08-15T23:06:31.076Z

## Task workflow update - 2026-08-15T23:19:03.244Z
- Full architect report: [`/home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md`](file:///home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md) — CodingAgent report, candidate 2.

## Task workflow update - 2026-08-16T22:07:40.207Z
- Moved TODO → IN-PROGRESS.
- Created branch task/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client.
- Created worktree /home/ineersa/projects/agent-core-worktrees/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client.
- Summary: Claimed for implementation. Scope: replace the custom LLM retry loop with Symfony RetryableHttpClient while preserving Hatfield retry policy, delays, cancellation, privacy, and provider-visible behavior.

## Task workflow update - 2026-08-16T22:15:33.869Z
- Summary: Claimed and scouted. The exact installed Symfony 8.1.1 RetryableHttpClient cannot preserve all current documented behavior through RetryStrategyInterface alone, so implementation is blocked on a specification decision rather than silently changing provider behavior. Native RetryableHttpClient does own async orchestration/body buffering/cancel-before-retry/pause and uses the same maxRetries count, but four incompatibilities remain: timeout chunks bypass RetryStrategyInterface; only TransportExceptionInterface reaches the strategy (not arbitrary Throwable); native private Retry-After parsing overrides strategy delay for standard retry-after, so retry-after-ms cannot retain priority when both exist and standard delay cannot be capped; max_duration becomes one total retry budget instead of resetting per attempt.
- Three parallel agents completed: production flow/policy scout, tests/config impact scout, and exact Symfony 8.1.1 vendor/API researcher.
- Controller replay, TUI replay, live controller smoke, and LlamaCppSmokeTest currently bypass the production retry client through test HttpClient injection/direct construction; focused strategy+RetryableHttpClient integration tests are required. Replay lanes remain downstream validation, while task acceptance still requires focused llm-real and controller-replay.
- No files edited and no tests run. Fork intentionally not launched until the user chooses native Symfony semantics versus exact legacy compatibility.

## Task workflow update - 2026-08-17T00:03:36.421Z
- Summary: Specification decision resolved by user: use Symfony-native retry behavior and do not reinvent compatibility orchestration. This explicitly accepts installed RetryableHttpClient semantics for timeout chunks, TransportExceptionInterface typing, standard Retry-After precedence/delay, and total max_duration budget where they differ from the legacy loop. Preserve Hatfield status/body/terminal-billing/backoff/retry-after-ms decisions only through Symfony's public RetryStrategyInterface seam; do not recreate the old loop around it.

## Task workflow update - 2026-08-17T00:15:40.369Z
- Validation: castor test --filter=LlmHttpRetryStrategyTest: PASS (11 tests, 18 assertions; twice); castor test --filter='LlmHttpRetryPolicyTest|SymfonyAiProviderFactoryTest': PASS (31 tests, 106 assertions); castor test: PASS (4495 tests, 17662 assertions); castor test:controller-replay: PASS (12 tests, 165 assertions); castor test:llm-real: PASS (13 tests, 144 assertions); castor deptrac: PASS (0 violations, 0 errors); castor phpstan: PASS (0 errors; stale baseline entries for deleted class removed); castor cs-check: PASS (0 files fixed); git diff --check: clean; IDE diagnostics: 0 problems in strategy, factory, and strategy test
- Summary: Implementation complete and committed as a42ca483a on task/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client. Deleted the 192-line manual LlmRetryingHttpClient and its 261-line loop-mirroring test; added one 103-line LlmHttpRetryStrategy implementing Symfony RetryStrategyInterface and 172 lines of behavior-focused tests; SymfonyAiProviderFactory now wires native RetryableHttpClient directly. 7 files changed, +290/-472 (net -182). Hatfield status/body/terminal-billing/backoff/retry-after-ms policy remains in LlmHttpRetryPolicy. Native Symfony owns retries, response cancellation, replacement, pause, exhaustion, standard Retry-After, transport typing, and total max_duration. Privacy-safe retry logging stays in the strategy; Symfony's logger is intentionally not passed because its message can include exception text. Worktree clean; no push, PR, or castor check.
- Implementation fork explicitly read and followed .agents/skills/testing/SKILL.md and tests/AGENTS.md.
- Verified commit a42ca483a exists in the exact task worktree; cumulative diff is 7 files +290/-472; IDE indexed search finds zero remaining LlmRetryingHttpClient references.
- Accepted native deltas recorded for PR review: timeout chunks bypass strategy; arbitrary non-TransportExceptionInterface throwables propagate; standard Retry-After has Symfony precedence and is uncapped by Hatfield; max_duration is cumulative across attempts.

## Task workflow update - 2026-08-17T00:36:24.265Z
- Validation: Reviewer: APPROVED; no blocking findings; castor test: PASS (4495 tests, 17662 assertions) after one unrelated pre-existing LoaderWidget timing flake; focused flaky test then full rerun both passed; castor test --filter=ChatScreenStatusRowVirtualRenderTest: PASS (1 test, 23 assertions); castor test:llm-real: PASS (13 tests, 144 assertions); castor test:controller-replay: PASS (12 tests, 165 assertions); castor deptrac: PASS (0 violations, 0 errors); castor phpstan: PASS (0 errors); castor cs-check: PASS (0 files fixed); git diff --check: clean; worktree clean
- Summary: Task-to-PR review APPROVED on commit a42ca483a. Reviewer verified installed Symfony 8.1.1 control flow, body-fetch/null contract, policy mapping, retry_count/backoff, retry-after-ms, exhaustion, typed transport path, streaming passthrough, cancellation/pause ownership, direct factory wiring, privacy-safe logging, stale-reference cleanup, test minimality, and specification fidelity. No blockers or unmapped public surface; no fixes required.
- Initial task-to-PR castor test hit unrelated ChatScreenStatusRowVirtualRenderTest LoaderWidget frame timing assertion; task diff does not touch that test/TUI path. Focused rerun passed, then complete castor test passed. No code changes made for the flake.
- Reviewer noted non-blocking accepted observability delta: Symfony consumes standard Retry-After before strategy getDelay(), so Hatfield's privacy-safe retry log is skipped on that path rather than duplicating native delay logic or exposing exception messages through Symfony's logger.

## Task workflow update - 2026-08-17T00:38:41.919Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (123.7s).
- Pushed task/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client to origin.
- branch 'task/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client' set up to track 'origin/task/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client'.
- Created PR: https://github.com/ineersa/agent-core/pull/398

## Task workflow update - 2026-08-17T01:03:56.396Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client.
- Merged task/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client into integration checkout.
- Merge made by the 'ort' strategy.
 phpstan-baseline.neon                              |  10 -
 .../SymfonyAi/Http/LlmHttpRetryStrategy.php        | 103 ++++++++
 .../SymfonyAi/Http/LlmRetryingHttpClient.php       | 192 ---------------
 .../SymfonyAi/SymfonyAiProviderFactory.php         |  22 +-
 .../Bridge/OpenAICodex/CodexModelClient.php        |   2 +-
 .../SymfonyAi/Http/LlmHttpRetryStrategyTest.php    | 172 ++++++++++++++
 .../SymfonyAi/Http/LlmRetryingHttpClientTest.php   | 261 ---------------------
 7 files changed, 290 insertions(+), 472 deletions(-)
 create mode 100644 src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpRetryStrategy.php
 delete mode 100644 src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmRetryingHttpClient.php
 create mode 100644 tests/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpRetryStrategyTest.php
 delete mode 100644 tests/CodingAgent/Infrastructure/SymfonyAi/Http/LlmRetryingHttpClientTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/coding-agent-02-replace-llm-retry-loop-with-symfony-retryable-http-client.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #398 merged on GitHub as 1be24268caa2c72c66813ad2835ff8182a91813a. Moving task to DONE and syncing integration checkout.

## Task workflow update - 2026-08-17T01:06:30.991Z
- Validation: Post-merge LLM_MODE=true castor check PASS — qa-20260817-010402-818382-2afe3916; deptrac PASS; test PASS (4491 tests, 17673 assertions); test:controller-replay PASS (12 tests, 165 assertions); test:tui PASS (38 tests, 297 assertions); test:llm-real PASS (13 tests, 144 assertions); phpstan PASS (0 errors); cs-check PASS; docs:validate PASS; QA artifact integrity PASS; leak check PASS; llama-proxy cache stable 326→326; Integration git status clean; task worktree removed
- Summary: DONE: PR #398 merged as 1be24268caa2c72c66813ad2835ff8182a91813a; integration checkout synced at 1e8747c3c; task worktree and IDEA exclusions removed; integration checkout clean.

## Task workflow update - 2026-08-18T00:06:36.245Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
