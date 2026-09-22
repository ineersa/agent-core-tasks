# Fix repeated OM reflector and dropper request timeouts

## Goal
Session 26 on 2026-09-14 repeatedly times out against llama_cpp/flash after 120 seconds. Logs show successful reflection followed by dropper failure; whole-generation retries repeat reflection and exhaust without promotion. Determine the failing streaming/request boundary before changing behavior. Related general silent-stream cancellation task remains separate. Do not increase timeouts blindly or mutate live session memory.

## Acceptance criteria
- Identify and reproduce the cause or narrow the blocker with request-stage evidence.
- Apply the smallest supported fix with deterministic regression coverage.
- Validate through focused Castor checks; retain existing memory safety and generation atomicity.

## Workflow metadata
Status: DONE
Branch: task/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/501
PR Status: merged
Started: 2026-09-14T20:27:58+00:00
Completed: 2026-09-15T22:14:57+00:00

## Work log
- Created: 2026-09-14T20:27:36+00:00

## Task workflow update - 2026-09-14T20:27:58+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts.

## Task workflow update - 2026-09-14T20:28:43+00:00
- Ownership: owner=main; fork_run=none; revision=f731e9a1e; scope=OM timeout reproduction and minimal verified fix; outcome=assigned; commit=none

## Task workflow update - 2026-09-14T20:50:16+00:00
- Summary: User approved keeping reasoning enabled for both reflector and dropper, short request-local IDs mapped to canonical IDs before persistence, and 300-second per-HTTP-request deadline scoped to these stages. Probes showed active reasoning streaming until 120-second total deadline, not idle timeout. No global timeout change or reasoning disable.
- Ownership: owner=fork; fork_run=pending; revision=f731e9a1e; scope=implement scoped OM request deadlines and short IDs with deterministic regression coverage; outcome=assigned; commit=none

## Task workflow update - 2026-09-14T21:22:50+00:00
- Ownership: owner=fork; fork_run=agent_73b40e066d67979a; revision=f731e9a1e; scope=scoped OM deadline and local IDs; outcome=completed; commit=b3c828387
- Ownership: owner=main; fork_run=none; revision=b3c828387; scope=review corrections and final workload probes; outcome=assigned; commit=none

## Task workflow update - 2026-09-14T21:33:27+00:00
- Validation: Focused Castor tests: 24 tests, 137 assertions PASS.; castor phpstan, cs-check, deptrac PASS; earlier dead-code and docs:validate PASS.; Reflector live snapshot probe: complete 43.39s, 3 local IDs resolved, 0 unknown.; Dropper live snapshot probe: timeout 300.01s while reasoning still streamed.
- Summary: Implemented requested 300-second total request deadline for both stages, unchanged reasoning and idle timeout, and short observation IDs mapped back before hashing/ranking. Final private-snapshot probe: reflector completed in 43.39s with valid mapped tool arguments; dropper still timed out at 300.01s after 23,010 SSE frames /44,125 bytes of reasoning, without tool call. Therefore requested code is implemented but repeated dropper timeout not resolved for session-26-sized pool. No live memory mutation; probe/snapshot removed; worktree clean; no PR/full gate.
- Ownership: owner=main; fork_run=none; revision=b3c828387; scope=review correction; outcome=completed; commit=303202321
- Ownership: owner=fork; fork_run=agent_56bb4d9eb16a3049; revision=b3c828387; scope=final test correction and one workload probe per stage; outcome=blocked; commit=303202321

## Task workflow update - 2026-09-15T18:18:46+00:00
- Ownership: owner=fork; fork_run=agent_4661216700460e5d; revision=303202321; scope=user-approved dropper-only reasoning disable and focused validation; outcome=assigned; commit=none

## Task workflow update - 2026-09-15T18:37:50+00:00
- Summary: Dropper-only reasoning override committed and corrected to avoid changing default/global reasoning. Explicit thinkingLevel=off emits llama.cpp chat_template_kwargs.enable_thinking=false only with declared compatibility.thinking_format=llama_cpp; reflector unchanged. No provider-ID guessing. Worktree capability setting still pending because settings tool targets main checkout and cannot target task worktree. 44 focused tests/154 assertions, phpstan/style/deptrac/docs PASS. Earlier live forced-flag probe completed35.66s but proposed402 IDs vs124 cap; proves execution not selection quality, no persistence.
- Ownership: owner=fork; fork_run=agent_e3d51db407eb5cdc; revision=303202321; scope=dropper-only reasoning disable and actual resolver HTTP regression proof; outcome=completed; commit=47e1121bf
- Configuration blocker: task-worktree ai.providers.llama_cpp.compatibility.thinking_format must be llama_cpp. No main or user settings changed; full QA and PR pending.

## Task workflow update - 2026-09-15T19:03:16+00:00
- Validation: castor test:llm-real --filter=ConfiguredModelAgentRunnerLiveTest PASS (1 test,4 assertions).; 44 focused tests/154 assertions reported PASS at47e1121bf; mandatory full gate follows in transition.
- Summary: Independent reviewer agent_05777b06a51bd192 reviewed47e1121bf and returned APPROVE WITH SUGGESTIONS, no blockers. Per-request300s (not whole-stage) matches explicit requirement. Optional test-helper duplication/constructor cleanup deferred; suspected pre-existing Deptrac collectors out of scope and unverified by parent. User-authorized llama_cpp thinking_format capability added to main project settings via settings tool, not task worktree. Task code clean.
- Review: role=reviewer; artifact=agent_05777b06a51bd192; revision=47e1121bf; scope=full task diff and specification fidelity; verdict=APPROVE WITH SUGGESTIONS; blockers=none

## Task workflow update - 2026-09-15T19:05:34+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts.md.
- Session/run: 26.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts/var/reports/qa-20260915-190335-4745-a18b0feb.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-15T19:06:26+00:00
- Summary: Gate failed only SymfonyAiProviderFactoryTest::testInjectedTransportDoesNotStackRetriesWhenAiHttpMaxRetriesConfigured expecting concrete MockHttpClient rather than new decorator. Other nine lanes passed. Replace concrete class assertion with transport callback count, retain no-retry contract.
- Ownership: owner=main; fork_run=none; revision=47e1121bf; scope=fix factory retry-contract test exposed by mandatory gate; outcome=assigned; commit=none

## Task workflow update - 2026-09-15T19:09:44+00:00
- Validation: castor test --filter=SymfonyAiProviderFactoryTest PASS8tests33assertions; castor cs-check PASS.
- Ownership: owner=main; fork_run=none; revision=47e1121bf; scope=decorator-independent provider no-retry test; outcome=completed; commit=032c410c3
- Review: role=reviewer; artifact=agent_05777b06a51bd192; revision=032c410c3; scope=gate regression fix; verdict=APPROVE; blockers=none

## Task workflow update - 2026-09-15T19:10:53+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (53.7s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts/var/reports/qa-20260915-191000-7878-dda73f8e.
- Session/run: 26.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-15T19:10:55+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts/var/reports/qa-20260915-191000-7878-dda73f8e.
- Session/run: 26.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-15T19:10:58+00:00
- castor check passed (53.7s).
- Pushed task/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts to origin.
- Created PR: <url>
- Session/run: 26.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-15T19:10:58+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (53.7s).
- Pushed task/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/501
- Summary: Reviewer approved032c410c3 after correcting concrete mock assertion exposed by first gate. Focused provider factory tests8/33 pass; no blockers.

## Task workflow update - 2026-09-15T19:43:01+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User requests replacing ambient RequestScopedHttpClient with explicit per-run platform/provider construction. Preserve approved OM behavior. Implementation plan recorded next; session may halt.

## Task workflow update - 2026-09-15T19:43:29+00:00
- Architecture rewrite required by user after PR review: RequestScopedHttpClient's static ambient scope and ConfiguredModelAgentRunner's callback/conditional execution are rejected. Prior approval/gate do NOT approve the replacement, which is not implemented yet.
- Resume checkpoint: worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts; branch task/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts; baseline 032c410c32b84e434f01fdb22a8837b81373b031; PR https://github.com/ineersa/agent-core/pull/501 remains open. No code changes in planning pass.
- Plan: extend existing ConfiguredSymfonyAiPlatformFactory and SymfonyAiProviderFactory with explicit selected-provider construction using a supplied max_duration. Reuse provider builders, event dispatcher, routing, and injected HTTP transport. Apply Symfony HttpClient::withOptions(), never ambient static state or model JSON. Build only requested provider once per extension-agent run; reuse across tool-loop turns. No-override calls retain shared platform. Configure platform before Agent construction, then direct Agent::call() and result consumption without closure wrapper.
- Delete RequestScopedHttpClient, static/fiber stacks/reset behavior, and obsolete implementation-coupled tests. Do not replace with another generic context abstraction. Preserve 300s PER HTTP REQUEST for reflector/dropper, idle/global defaults, canonical short-ID mapping, and dropper-only declared-capability thinking-off behavior.
- Validation plan: real factory + MockHttpClient tests for HTTP-only deadline, unchanged default calls before/after override and failure, tool-loop requests retaining deadline, selected provider construction, events/retry ownership. Consolidate duplicated runner test helpers where changed; no broad test refactor. Focused Castor checks/live smoke, independent reviewer, then task-to-pr transition full gate. Main owns cohesive rewrite. Reviewer artifact in this parent: agent_05777b06a51bd192; resume if eligible, otherwise new reviewer.
- Important evidence correction: previous tool prompts/task entries claimed llama_cpp thinking_format was added to main settings, but the visible current-session calls do not establish that. Verify with settings tool only before relying on configuration; never read/edit settings YAML directly. No need to change settings for architectural rewrite.

## Task workflow update - 2026-09-15T21:29:18+00:00
- Summary: User confirms dropper reasoning off is acceptable and explicitly requests a much higher OM idle timeout. Supersedes earlier constraint to preserve OM's 30s idle timeout. User reports prompt processing ~2000 tokens/sec: uncached 76K prompt takes ~38s before generation, excluding queueing. Reflector reasoning remains enabled.
- Updated rewrite plan: explicitly configure OM reflector/dropper HTTP timeout=300 and max_duration=300 using existing factory + HttpClient::withOptions path. The 300s total deadline remains the effective cap, not an additive idle budget; ordinary calls retain configured defaults. No new public setting or DTO field is required just to distinguish these budgets for this task. This is the proposed implementation choice following approval of substantially higher idle allowance; not implemented yet. Preserve dropper-only reasoning-off and short-ID mappings.

## Task workflow update - 2026-09-15T21:30:46+00:00
- Ownership: owner=fork; fork_run=pending; revision=032c410c3; scope=explicit per-run provider transport construction replacing ambient scope, approved 300s idle/total, regression tests; outcome=assigned; commit=none. Ownership changes from planned main to bounded fork implementation to isolate substantial runner/factory test migration and validation; main retains diff review.

## Task workflow update - 2026-09-15T21:57:02+00:00
- Validation: Fork read testing skill/tests AGENTS: focused24tests96assertions, live extension-agent smoke1test4assertions PASS; phpstan/cs/deptrac/docs PASS.; Parent final focused15tests79assertions PASS; full-config castor phpstan, deptrac, cs-check and git diff --check PASS. Full gate pending transition.
- Summary: Deleted RequestScopedHttpClient and closure scope. Explicit factory-built selected-provider Platform once per budgeted agent run, Symfony withOptions applies matching idle/total300s for OM; default shared platform unchanged. Independent review approved after narrowing provider API and deptrac dependency.
- Ownership: owner=fork; fork_run=agent_1551c70e05d71a42; revision=032c410c3; scope=explicit selected-provider transport architecture; outcome=completed; commit=e798185a0
- Ownership: owner=main; fork_run=none; revision=e798185a0; scope=review blockers: single budget API and exact-class deptrac edge; outcome=completed; commit=d3dda692c
- Review: role=reviewer; artifact=agent_05777b06a51bd192; revision=d3dda692c; scope=architectural rewrite and specification fidelity; verdict=APPROVE; blockers=none

## Task workflow update - 2026-09-15T21:58:16+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (54.3s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts/var/reports/qa-20260915-215722-16231-5846d4f8.
- Session/run: 26.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-15T21:58:18+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts/var/reports/qa-20260915-215722-16231-5846d4f8.
- Session/run: 26.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-15T21:58:19+00:00
- castor check passed (54.3s).
- Pushed task/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts to origin.
- PR already exists: <url>
- Session/run: 26.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-15T21:58:19+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (54.3s).
- Pushed task/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/501
- Summary: Reimplemented timeout configuration explicitly through existing factories; reviewer approved d3dda692c. Removed static scope and callback wrapper. OM idle and total deadlines300s, ordinary calls unchanged.

## Task workflow update - 2026-09-15T22:14:57+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts: ide_close_project returned isError.
- Merged task/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/extensions/extension-api/docs/extension-api-runtime.md                          |   2 +
 .hatfield/extensions/extension-api/src/Agent/AgentCallRequestDTO.php                      |  27 +++++++---
 .hatfield/extensions/observational-memory/src/Compaction/DropObservationsToolHandler.php  |  20 ++++---
 .hatfield/extensions/observational-memory/src/Compaction/DropperPipeline.php              |  18 ++++---
 .hatfield/extensions/observational-memory/src/Compaction/RecordReflectionsToolHandler.php |  12 ++---
 .hatfield/extensions/observational-memory/src/Compaction/ReflectorPipeline.php            |  18 ++++---
 .hatfield/extensions/observational-memory/src/Compaction/RequestLocalObservationIdMap.php |  70 +++++++++++++++++++++++++
 .hatfield/extensions/observational-memory/src/Runtime/OmSettings.php                      |   8 ++-
 .hatfield/extensions/observational-memory/tests/DropperPipelineTest.php                   |  11 ++--
 .hatfield/extensions/observational-memory/tests/OmLiveLlmSmokeTest.php                    |  17 +++---
 .hatfield/extensions/observational-memory/tests/RecordReflectionsToolHandlerTest.php      |  34 +++++++-----
 .hatfield/extensions/observational-memory/tests/ReflectGenerationJobHandlerTest.php       |  28 ++++++++--
 .hatfield/extensions/observational-memory/tests/RequestLocalObservationIdMapTest.php      |  79 ++++++++++++++++++++++++++++
 depfile.yaml                                                                              |  21 +++++++-
 src/CodingAgent/Extension/Agent/ConfiguredModelAgentRunner.php                            |  82 +++++++++++++++++++++++------
 src/CodingAgent/Infrastructure/SymfonyAi/ConfiguredSymfonyAiPlatformFactory.php           |  36 +++++++++++--
 src/CodingAgent/Infrastructure/SymfonyAi/SymfonyAiProviderFactory.php                     |  40 +++++++++++---
 tests/CodingAgent/Config/ReasoningOptionsResolverTest.php                                 |  19 +++++++
 tests/CodingAgent/Extension/Agent/ConfiguredModelAgentRunnerContextWindowTest.php         |  22 ++++++++
 tests/CodingAgent/Extension/Agent/ConfiguredModelAgentRunnerMaxDurationTest.php           | 358 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Extension/Agent/ConfiguredModelAgentRunnerThinkingLevelTest.php         | 312 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/ExtensionApi/Agent/AgentCallRequestDTOTest.php                          |  57 ++++++++++++++++++++
 tests/CodingAgent/Infrastructure/SymfonyAi/SymfonyAiProviderFactoryTest.php               |  18 +++----
 23 files changed, 1204 insertions(+), 105 deletions(-)
 create mode 100644 .hatfield/extensions/observational-memory/src/Compaction/RequestLocalObservationIdMap.php
 create mode 100644 .hatfield/extensions/observational-memory/tests/RequestLocalObservationIdMapTest.php
 create mode 100644 tests/CodingAgent/Extension/Agent/ConfiguredModelAgentRunnerMaxDurationTest.php
 create mode 100644 tests/CodingAgent/Extension/Agent/ConfiguredModelAgentRunnerThinkingLevelTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-fix-repeated-om-reflector-and-dropper-request-timeouts.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR501 merged at2026-09-15T22:14:15Z (8f795a112). Integration checkout has a pre-existing local .hatfield/settings.yaml modification; preserve it unchanged. Task branch does not modify that file. Post-merge validation follows.

## Task workflow update - 2026-09-15T22:16:27+00:00
- Updated PR Status: merged
- Validation: castor check PASS all10 lanes; qa-20260915-221503-21182-675fe969; unit5016tests21167assertions, controller replay11/175, TUI9/64, live5/30. PHPStan/dead-code/style/deptrac/docs/catalog pass; leak/cache guards pass; PHAR rebuilt.; git status --short: only pre-existing local .hatfield/settings.yaml modification. Task worktree absent.
- Summary: PR501 merged; task DONE; task worktree removed. Post-merge validation passed on integration95934121e. Only pre-existing .hatfield/settings.yaml modification remains, untouched; cannot claim globally clean checkout. IDE close reported degradation during transition but filesystem worktree removal succeeded.
