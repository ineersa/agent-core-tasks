# Add OpenCode Go provider and subscription usage to Hatfield

## Goal
User bought OpenCode Go and requests a selectable provider plus `/usage` integration. Research: OpenCode Go uses an API key, not OAuth. Official docs https://opencode.ai/docs/go list per-model endpoints at https://opencode.ai/zen/go/v1/chat/completions, /responses, or /messages. Pi https://github.com/earendil-works/pi has `opencode-go` with `OPENCODE_API_KEY`, mixed APIs, and x-opencode-session routing header. The usage extension https://github.com/Dakai/pi-opencode-go-usage queries GET https://opencode.ai/console/api/go/status with browser `__Host-console_session` cookie and x-org-id workspace; the model API key alone does not authenticate that console endpoint. Clarify whether storing a separate browser cookie and workspace ID for `/usage` is acceptable before implementing usage. Use curated model IDs on a supported transport and follow catalog upgrade/version rules. Respect the separate provider-extraction worktree; do not alter it.

## Acceptance criteria
- OpenCode Go appears as an opt-in API-key provider in the bundled AI catalog and providers setup, using the correct endpoint and supported model metadata.
- Configured OpenCode Go models can be selected and routed through existing Symfony AI infrastructure; provider behavior is verified at the lowest correct layer.
- If the user approves separate console authentication, `/usage` reports Go window limits without logging or printing credentials, with deterministic failure handling and TUI virtual proof.
- Document setup, credential separation, and limitations; run focused Castor validation before task-to-pr full gate.

## Workflow metadata
Status: DONE
Branch: task/2026-09-26-add-opencode-go-provider-and-usage
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-26-add-opencode-go-provider-and-usage
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/531
PR Status: merged
Started: 2026-09-26T14:51:09+00:00
Completed: 2026-09-27T00:07:25+00:00

## Work log
- Created: 2026-09-26T14:50:00+00:00

## Task workflow update - 2026-09-26T14:50:46+00:00
- Summary: User clarified scope: implement OpenCode Go provider now; defer /usage because its console endpoint requires a separately configured workspace ID and browser session cookie. Do not add cookie storage or usage probing in this task. Existing usage acceptance criterion is deferred, not part of implementation.
- Ownership: owner=main; fork_run=none; revision=84b0d8eb7; scope=OpenCode Go provider catalog, configuration docs, and focused tests only; outcome=assigned; commit=none

## Task workflow update - 2026-09-26T14:51:09+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-26-add-opencode-go-provider-and-usage.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-26-add-opencode-go-provider-and-usage.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-26-add-opencode-go-provider-and-usage.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-26-add-opencode-go-provider-and-usage.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-26-add-opencode-go-provider-and-usage.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-26-add-opencode-go-provider-and-usage/.idea.

## Task workflow update - 2026-09-26T15:00:47+00:00
- Validation: castor test --filter='(AiCatalogTest|ProvidersSetupFlowTest|ProvidersUpdateCommandTest|SymfonyAiProviderFactoryTest)' (44 tests, 300 assertions); castor test --filter='(testBundledOpenCodeGo|testBundledCatalogIncludesCurrentPiModels)' (3 tests, 57 assertions after last edit); castor test:llm-real --filter=LlamaCppSmokeTest (1 test, 8 assertions); castor phpstan (0 errors); castor deptrac (0 violations); castor docs:validate (pass); castor cs-check (0 files); castor lsp:check --path=tests/CodingAgent/Infrastructure/SymfonyAi/SymfonyAiProviderFactoryTest.php (0 diagnostics); git diff --check (pass)
- Summary: Added opt-in OpenCode Go API-key provider to catalog version 9 with four curated chat-completions models (GLM-5.3-Flash, Kimi K3, DeepSeek V4 Pro, MiMo-V2.6-Flash), setup instructions and regression coverage for catalog, Bearer auth, endpoint routing and response. `/usage` deferred by user; follow-up TODO 2026-09-26-opencode-go-subscription-usage tracks separate cookie/workspace decision. No live Go key was available; actual paid endpoint not exercised. Task branch includes pre-existing 29521f286 settings change made before this implementation; preserve it.
- Ownership: owner=main; fork_run=none; revision=84b0d8eb7; scope=OpenCode Go provider catalog, configuration docs, and focused tests only; outcome=completed; commit=892b33e1a

## Task workflow update - 2026-09-26T15:01:10+00:00
- Summary: Verified commit 29521f286 is already on integration main; PR diff against current main contains only the Go provider commit 892b33e1a.

## Task workflow update - 2026-09-26T15:05:28+00:00
- Summary: User asked whether OpenCode session header is sent. Found regression: generic Symfony AI client only sends Content-Type/Bearer, while Pi issue #9326 reports OpenCode now rejects requests without x-opencode-session. Reopening implementation within same IN-PROGRESS task to pass stable run ID per invocation and test header before PR.
- Ownership: owner=main; fork_run=none; revision=892b33e1a; scope=OpenCode Go x-opencode-session per-run request header and focused regression proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-26T15:15:01+00:00
- Summary: User selected four Go models instead of live discovery: DeepSeek V4.1 Flash, Muse Spark 1.3 Contributor, Space Bunny Free, LongCat 2.5 Preview Free. Pi generator maps three to chat completions and Muse to OpenAI Responses; implement both routes under Go and move provider-specific wiring out of shared generic factory. Go /models only returns IDs, not API/metadata.

## Task workflow update - 2026-09-26T15:36:39+00:00
- Validation: castor test --filter='(AiCatalogTest|SymfonyAiProviderFactoryTest|LlmPlatformAdapterTest|LlmCancelAwareHttpClientTest|OpenCodeSessionHttpClientTest|ProvidersSetupFlowTest|ProvidersUpdateCommandTest)' (62 tests, 462 assertions); castor phpstan (0 errors); castor deptrac (0 violations); castor cs-check (0 files); castor docs:validate (pass); castor lsp:check --path=src/CodingAgent/Infrastructure/SymfonyAi/OpenCodeGo (0 diagnostics); castor test:llm-real --filter=LlamaCppSmokeTest (1 test, 8 assertions), not live Go
- Summary: Revision 579ef7226 replaces preliminary four models with user-selected four, routes Muse to Responses and others to chat completions, and propagates stable run ID as x-opencode-session. User has a configured Go key and will test manually in the worktree after PR; no credentials or live Go request used by the implementation checks. /usage remains deferred.
- Ownership: owner=main; fork_run=none; revision=892b33e1a; scope=OpenCode Go selected-model mixed API routing, x-opencode-session, docs and focused tests; outcome=completed; commit=579ef7226
- Review: owner=reviewer; artifact=agent_0287ebceb28dbfb2; revision=579ef7226; scope=origin/main...HEAD, specification fidelity, auth/header/session routing, correctness, tests; verdict=APPROVE WITH SUGGESTIONS. No blockers. Optional deduplicate env: key resolution and rename test; deferred as nonrequired cleanup.

## Task workflow update - 2026-09-26T15:38:59+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (124.3s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-26-add-opencode-go-provider-and-usage/var/reports/qa-20260926-153655-963-f7ee94b5.
- Session/run: 69.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-26T15:39:01+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-26-add-opencode-go-provider-and-usage to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-26-add-opencode-go-provider-and-usage/var/reports/qa-20260926-153655-963-f7ee94b5.
- Session/run: 69.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-26T15:39:03+00:00
- castor check passed (124.3s).
- Pushed task/2026-09-26-add-opencode-go-provider-and-usage to origin.
- Created PR: <url>
- Session/run: 69.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-26T15:39:03+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (124.3s).
- Pushed task/2026-09-26-add-opencode-go-provider-and-usage to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/531
- Validation: Focused Castor tests 62 passed/462 assertions; phpstan/deptrac/cs-check/docs/lsp passed; local llama smoke passed.; Full castor check owned by CODE-REVIEW transition; no live paid OpenCode Go request made.
- Summary: OpenCode Go provider ready for manual worktree testing after automated full QA. Selected four user-requested models, Go session header, mixed API routing, opt-in docs; usage postponed separately. Reviewer APPROVE WITH SUGGESTIONS at 579ef7226.

## Task workflow update - 2026-09-26T15:54:23+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User-reported live 401 and missing reasoning-level controls. Read PR #531 and 3 inline comments. Current user settings explicitly reference env:OPENCODE_GO_API_KEY (absent); OPENCODE_API_KEY is present. Investigate default key selection without overriding explicit config, verify correct per-model reasoning support, answer session/header and scope-entry review comments. No code changed yet.

## Task workflow update - 2026-09-26T15:54:35+00:00
- Ownership: owner=main; fork_run=none; revision=579ef7226; scope=Go default API-key env resolution, model-specific reasoning-level routing, PR review comments, and focused proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-26T18:23:59+00:00
- Ownership: owner=main; fork_run=none; revision=1eb86afaa; scope=Correct existing user Go env reference, reject unresolved Go env on request, complete DeepSeek compatibility and focused proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-26T18:28:59+00:00
- Validation: castor test --filter='(AiCatalogTest|ReasoningOptionsResolverTest|SymfonyAiProviderFactoryTest|OpenCodeSessionHttpClientTest|SetupScreenVirtualRenderTest|ModelControlListenerTest|ProvidersUpdateCommandTest|ProvidersSetupFlowTest)' (111 tests, 585 assertions); castor phpstan (0 errors); castor cs-check (0 files); castor docs:validate (pass); castor deptrac (0 violations); castor lsp:check --path=src/CodingAgent/Infrastructure/SymfonyAi/OpenCodeGo (0 diagnostics)
- Summary: Fixed ~/.hatfield/settings.yaml OpenCode Go explicit env reference to env:OPENCODE_API_KEY and ran providers:update (catalog version 11). Verified key exists; no secret was printed. Changed Go HTTP client to reject unset configured env refs on request rather than send unauthenticated traffic; completed DeepSeek multi-turn compatibility metadata after reviewer REQUEST CHANGES.
- Ownership: owner=main; fork_run=none; revision=1eb86afaa; scope=Correct existing user Go env reference, reject unresolved Go env on request, complete DeepSeek compatibility and focused proof; outcome=completed; commit=2ea01fd3c
- Review: owner=reviewer; artifact=agent_0287ebceb28dbfb2; revision=2ea01fd3c; scope=incremental Go auth/settings and DeepSeek compatibility, specification fidelity and tests; verdict=pending

## Task workflow update - 2026-09-26T18:31:46+00:00
- Validation: castor test:llm-real --filter=LlamaCppSmokeTest (1 test, 8 assertions); Reviewer independently ran castor phpstan --path src/CodingAgent/Infrastructure/SymfonyAi (0 errors) and castor test --filter for affected classes (95 tests, 499 assertions)
- Summary: Settings reference and catalog copy updated; source revision 2ea01fd3c reviewed APPROVE without blockers. Live Go request with valid key returned workspace Global regions restriction on DeepSeek; no workspace privacy preference was modified.
- Review: owner=reviewer; artifact=agent_0287ebceb28dbfb2; revision=2ea01fd3c; scope=Go auth/settings, DeepSeek multi-turn compatibility, specification fidelity and test quality; verdict=APPROVE; blockers=none

## Task workflow update - 2026-09-26T18:32:58+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (63.7s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-26-add-opencode-go-provider-and-usage/var/reports/qa-20260926-183155-10725-0ce94c84.
- Session/run: 69.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-26T18:33:00+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-26-add-opencode-go-provider-and-usage to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-26-add-opencode-go-provider-and-usage/var/reports/qa-20260926-183155-10725-0ce94c84.
- Session/run: 69.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-26T18:33:01+00:00
- castor check passed (63.7s).
- Pushed task/2026-09-26-add-opencode-go-provider-and-usage to origin.
- PR already exists: <url>
- Session/run: 69.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-26T18:33:01+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (63.7s).
- Pushed task/2026-09-26-add-opencode-go-provider-and-usage to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/531
- Validation: Focused Castor tests 111 passed, 585 assertions; phpstan, cs-check, docs:validate, deptrac, lsp:check passed; local llama smoke passed; full castor check owned by transition.
- Summary: Updated PR #531 for the user-reported 401: setup and bundled catalog use OPENCODE_API_KEY; unresolved env references fail locally on request; added Go headers, model reasoning levels, and DeepSeek multi-turn compatibility. Updated user settings and local catalog v11. Reviewer APPROVE at 2ea01fd3c.

## Task workflow update - 2026-09-26T19:02:07+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Advisor identified an unscoped extension-agent Go request path and DeepSeek off-option mismatch. Verified runner bypasses LlmPlatformAdapter and sends Go requests without entering LlmInvocationCancelScope; resolver currently returns [] for DeepSeek off. Reopening for narrow fixes and regression tests.

## Task workflow update - 2026-09-26T19:02:14+00:00
- Summary: Go-only decorator does not affect unrelated providers; separate extension runner path lacks scope and is a confirmed blocker. DeepSeek off currently omits the disable request; existing test pins that incorrect behavior and must change.
- Ownership: owner=main; fork_run=none; revision=2ea01fd3c; scope=Bind extension-agent Go requests to invocation scope through tool-loop completion, correct explicit DeepSeek off request, add focused regression proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-26T19:15:33+00:00
- Validation: castor test --filter='(ConfiguredModelAgentRunner|ReasoningOptionsResolverTest|SessionAwareModelResolverTest|SymfonyAiProviderFactoryTest|OpenCodeSessionHttpClientTest)' (72 tests, 245 assertions); castor phpstan (0); castor deptrac (0); castor cs-check (0); castor docs:validate (pass); castor lsp:check runner (0 diagnostics); castor test:llm-real --filter=LlamaCppSmokeTest (1, 8 assertions)
- Summary: Verified advisor P1/P2: extension-agent runner had session ID but lacked per-invocation scope; DeepSeek off omitted disabling thinking. Fixed both. Regression invokes actual Go builder through extension-agent runner with streamed mocked response, asserts Go-only header, off payload and scope cleanup; converter/tool continuation fixture was dropped as it failed to observe target behavior, with existing lower-layer streamed tool contracts retained. Reviewer APPROVE at 3bd5d644f.
- Ownership: owner=main; fork_run=none; revision=2ea01fd3c; scope=Bind extension-agent Go requests to invocation scope through tool-loop completion, correct explicit DeepSeek off request, add focused regression proof; outcome=completed; commit=3bd5d644f
- Review: owner=reviewer; artifact=agent_0287ebceb28dbfb2; revision=3bd5d644f; scope=advisor P1 runner scope, P2 DeepSeek off, spec fidelity, streaming/mock test quality; verdict=APPROVE; blockers=none

## Task workflow update - 2026-09-26T19:17:06+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (67.9s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-26-add-opencode-go-provider-and-usage/var/reports/qa-20260926-191558-17348-1b273faa.
- Session/run: 69.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-26T19:17:07+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-26-add-opencode-go-provider-and-usage to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-26-add-opencode-go-provider-and-usage/var/reports/qa-20260926-191558-17348-1b273faa.
- Session/run: 69.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-26T19:17:08+00:00
- castor check passed (67.9s).
- Pushed task/2026-09-26-add-opencode-go-provider-and-usage to origin.
- PR already exists: <url>
- Session/run: 69.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-26T19:17:08+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (67.9s).
- Pushed task/2026-09-26-add-opencode-go-provider-and-usage to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/531
- Validation: Focused Castor 72 tests/245 assertions; phpstan 0, deptrac 0, cs-check clean, docs:validate pass, lsp:check 0, local llama smoke 1 test/8 assertions; full check via transition.
- Summary: Addressed advisor review at 3bd5d644f: extension-agent Go calls now carry session ID across Agent execution and clear scope afterward; DeepSeek explicit off emits thinking.type=disabled. Integration test runs actual Go builder through extension runner without manual scope. Reviewer APPROVE; user settings and catalog already corrected.

## Task workflow update - 2026-09-27T00:07:25+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-26-add-opencode-go-provider-and-usage into integration checkout.
- Auto-merging docs/settings-models.md
Auto-merging src/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapter.php
Auto-merging tests/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapterTest.php
Merge made by the 'ort' strategy.
 config/ai-catalog.yaml                                                                     | 57 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++-
 docs/ai-catalog.md                                                                         |  4 ++--
 docs/settings-models.md                                                                    | 11 ++++++++++-
 src/AgentCore/Infrastructure/SymfonyAi/LlmInvocationCancelScope.php                        | 23 +++++++++++++++--------
 src/AgentCore/Infrastructure/SymfonyAi/LlmInvocationScopeEntry.php                         | 17 +++++++++++++++++
 src/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapter.php                              |  2 +-
 src/CodingAgent/Config/ReasoningOptionsResolver.php                                        |  6 +++---
 src/CodingAgent/Extension/Agent/ConfiguredModelAgentRunner.php                             |  6 +++++-
 src/CodingAgent/Infrastructure/SymfonyAi/Http/OpenCodeSessionHttpClient.php                | 48 ++++++++++++++++++++++++++++++++++++++++++++++++
 src/CodingAgent/Infrastructure/SymfonyAi/OpenCodeGo/OpenCodeGoProvider.php                 | 56 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 src/CodingAgent/Infrastructure/SymfonyAi/OpenCodeGo/OpenCodeGoSymfonyAiProviderBuilder.php | 82 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 src/Tui/Setup/SetupScreen.php                                                              |  4 ++++
 tests/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapterTest.php                        | 10 +++++++---
 tests/CodingAgent/Config/Ai/AiCatalogTest.php                                              | 30 ++++++++++++++++++++++++++++++
 tests/CodingAgent/Config/ReasoningOptionsResolverTest.php                                  | 27 +++++++++++++++++++++++++--
 tests/CodingAgent/Extension/Agent/ConfiguredModelAgentRunnerThinkingLevelTest.php          | 54 +++++++++++++++++++++++++++++++++++++++++++-----------
 tests/CodingAgent/Infrastructure/SymfonyAi/Http/OpenCodeSessionHttpClientTest.php          | 62 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Infrastructure/SymfonyAi/SymfonyAiProviderFactoryTest.php                | 93 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Listener/ModelControlListenerTest.php                                            | 41 +++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Setup/SetupScreenVirtualRenderTest.php                                           | 13 +++++++++++++
 20 files changed, 613 insertions(+), 33 deletions(-)
 create mode 100644 src/AgentCore/Infrastructure/SymfonyAi/LlmInvocationScopeEntry.php
 create mode 100644 src/CodingAgent/Infrastructure/SymfonyAi/Http/OpenCodeSessionHttpClient.php
 create mode 100644 src/CodingAgent/Infrastructure/SymfonyAi/OpenCodeGo/OpenCodeGoProvider.php
 create mode 100644 src/CodingAgent/Infrastructure/SymfonyAi/OpenCodeGo/OpenCodeGoSymfonyAiProviderBuilder.php
 create mode 100644 tests/CodingAgent/Infrastructure/SymfonyAi/Http/OpenCodeSessionHttpClientTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-26-add-opencode-go-provider-and-usage.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-26-add-opencode-go-provider-and-usage.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #531 merged on GitHub at dc12a413239640a89a0d074701744b875d50b2a4; integrate locally and run post-merge castor check.

## Task workflow update - 2026-09-27T00:08:54+00:00
- Updated PR Status: merged
- Validation: Post-merge integration checkout /home/ineersa/projects/agent-core: castor check passed (QA qa-20260927-000733-24188-4bca6265; unit 4844 tests/21910 assertions; controller replay 13/218; TUI 6/40; local llm-real 5/30; all 11 lanes passed).; git status --short clean; Go task worktree removed; PR #531 merged at dc12a4132.
- Summary: Marked DONE after merged PR #531, integrated locally and validated the integrated main checkout with a clean full Castor gate. The deferred /usage feature remains a separate TODO.
