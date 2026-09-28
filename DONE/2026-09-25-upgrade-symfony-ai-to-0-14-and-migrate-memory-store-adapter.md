# Upgrade Symfony AI to 0.14 and migrate observational-memory integration

## Goal
## Goal
Replace Hatfield's Symfony AI development pins and 0.13 packages with reproducible 0.14 releases. Preserve flat DTO tool schemas, constructor-derived required fields, agent execution, and observational-memory search. Do not include the MCP SDK upgrade or an unrelated Composer-wide update.

## Context
Symfony AI [v0.14.0](https://github.com/symfony/ai/releases/tag/v0.14.0) includes `#[MapToolArguments]` (#2510), the promoted-property schema fix (#2563), and the non-promoted-property and optional-only-object fixes (#2593). These are the reasons `composer.json` currently uses `symfony/ai-agent: dev-main` and `symfony/ai-platform: dev-main as 0.13.1`. Root `composer.json` also requires the generic and Open Responses bridges and the Store, SQLite Store, and Vektor Store packages at 0.13. `.hatfield/extensions/observational-memory/composer.json` independently requires five AI packages at 0.13. Both manifests must agree on 0.14.

This is not a constraints-only update. Symfony AI Store 0.14 makes `StoreInterface` extend `Countable` and widens `add()` to `VectorDocumentInterface|array`. `.hatfield/extensions/observational-memory/src/Semantic/MemoryStoreAdapter.php` implements the old interface and must be migrated. Review its filtering and `query()` path and `SemanticIndexService.php` for interface-typed documents, while preserving search results, truncation, and index behavior. Review the 0.14 Agent, Platform, Generic Platform, and Store changelogs for other API changes that affect Hatfield. Existing test token-usage fakes implement `getModel()`, but confirm they remain compatible.

## Scope and boundaries
- Update root and observational-memory extension Composer manifests and lockfiles as applicable. Install released 0.14 packages; do not use `dev-main`, inline version aliases, local path links, or modified vendor files to supply the three upstream fixes.
- Adapt only the callers and tests affected by the 0.14 API. Retain Hatfield's per-run tool registry, provider-aware schema describer, raw MCP and extension array handling, and model-facing contracts unless a verified upstream incompatibility requires a change.
- Keep `mcp/sdk` at its current version in this task. Evaluate MCP SDK and `symfony/ai-mcp-tool` in the existing TODO task `2026-09-10-update-composer-dependencies-and-migrate-to-current-symfony-ai-and-mc.md`; do not duplicate that migration.
- Do not introduce new settings or compatibility shims. Do not open a new PR without the user's explicit permission.

## References
- Symfony AI [0.14 release notes](https://github.com/symfony/ai/releases/tag/v0.14.0).
- Released [StoreInterface contract](https://github.com/symfony/ai-store/blob/v0.14.0/src/StoreInterface.php), [PropertySubject requiredness](https://github.com/symfony/ai-platform/blob/v0.14.0/src/Contract/JsonSchema/Subject/PropertySubject.php), and [Describer optional-only object handling](https://github.com/symfony/ai-platform/blob/v0.14.0/src/Contract/JsonSchema/Describer/Describer.php).
- Completed Hatfield migration: PR #523.

## Acceptance criteria
- Composer resolves all required Symfony AI packages to tagged 0.14 releases in root and observational-memory installations. Committed manifests and lockfiles reproduce the result without vendor overlays, aliases, or development branches. `mcp/sdk` remains unchanged.
- `MemoryStoreAdapter` satisfies 0.14 `StoreInterface`, including `Countable` and widened document types, without changing indexing, filtering, truncation, or search results. Any other affected 0.14 APIs are migrated at their callers rather than hidden behind compatibility code.
- Focused Castor tests prove flat mapped tool schemas and argument resolution, promoted and non-promoted constructor requiredness, observational-memory indexing and search, and provider conversion. Extend tests only for a behavior not already covered at the right layer; no duplicate framework tests or slow timing-based cases.
- Run focused `castor test`, `castor phpstan`, package and PHAR boot checks during implementation. For this provider and LLM-visible change, warm the live smoke with focused `castor test:llm-real`; the tracked CODE-REVIEW transition runs the required full `castor check` after explicit PR permission. Record any environmental blocker instead of treating focused checks as the full gate.
- Independent review checks specification fidelity, Symfony AI changelog compatibility, resolved lockfile versions, extension manifest alignment, and no unrelated MCP or broad Composer upgrade. Ask for permission before opening any PR.

## Workflow metadata
Status: DONE
Branch: task/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter
Fork run: agent_b5ca3051920f1651
PR URL: https://github.com/ineersa/agent-core/pull/534
PR Status: merged
Started: 2026-09-27T00:32:48+00:00
Completed: 2026-09-27T19:10:10+00:00

## Work log
- Created: 2026-09-25T13:31:00+00:00

## Task workflow update - 2026-09-25T13:42:19+00:00
- Summary: Expanded the 0.14 upgrade per user request to include a focused reuse and deletion audit of the other relevant Agent/Platform changes. Only remove custom code when released 0.14 behavior preserves Hatfield's contracts; keep MCP adoption and MCP SDK upgrades in their separate tasks.
- Added 0.14 tool-error audit: upstream symfony/ai#2365 reports missing or invalid arguments through InvalidToolCallArgumentsException. Check whether RegistryBackedToolbox's ToolException/NotNormalizableValueException translation is now dead for native resolver errors. Remove only redundant branches after verifying ToolExecutor's sanitized, structured error results, retryability, and extension failure hooks still receive the exact rewritten ToolCall. Preserve raw-array extension and MCP handling.
- Added cancellation audit: upstream Execution::cancel() uses its internal Cancellation object to cancel an active RawHttpResult and check between Agent steps; it does not consume Hatfield CancellationTokenInterface. Compare with RunCancellationToken, LlmCancelAwareHttpClient, and ConfiguredModelAgentRunner, which currently passes NullCancellationToken and blocks on getResult(). If an existing owned in-process cancellation signal can reach the live extension-agent Execution, connect it without a new public API and prove timely abort, cleanup, and no cross-run cancellation. Otherwise document why polling after getResult() or introducing a parallel cancellation path is not useful. Do not replace durable run/worker cancellation or the existing HTTP token checks.
- Audit 0.14 AbstractToolbox, ChainToolbox, FiberToolExecutor, per-run tool restrictions, TokenUsageInterface::getModel(), replay body redaction, and traceable timing against actual Hatfield callers. Reuse only when it deletes duplicated code without changing session-scoped tool visibility, Messenger execution/deduplication, privacy, or provider behavior; record the reason for retaining each non-applicable adapter. No speculative concurrency or observability feature is required.
- Boundary update: dedicated TODO 2026-09-25-evaluate-symfony-ai-mcptoolbox-adoption owns McpToolbox integration assessment, and add-proper-mcp-tool-call-cancellation owns MCP request-scoped cancellation. Keep mcp/sdk unchanged in this Symfony AI 0.14 upgrade.

## Task workflow update - 2026-09-26T03:13:06+00:00
- Summary: User expanded the 0.14 upgrade to include the standalone ineersa/symfony-ai-openai-codex-platform package. Update its Symfony AI Platform and OpenResponses constraints for released 0.14, validate the package against 0.14, then consume the reviewed package revision when updating Hatfield. Keep Codex-specific protocol and transport behavior; remove a workaround only after proving the released upstream behavior replaces it.
- Scope amendment (user, 2026-09-25): Include /home/ineersa/projects/symfony-ai-openai-codex-platform in the Symfony AI 0.14 upgrade. Its composer.json currently allows only ^0.12 || ^0.13 for symfony/ai-platform and symfony/ai-open-responses-platform. Resolve both against tagged 0.14 releases, update code/tests only for verified API changes, and run package Castor test, phpstan, cs-check, and composer-validate. Review upstream OpenResponses overlap without dropping Codex-only replay, tool IDs, request shape, error handling, or WebSocket transport. Acceptance: the package installs and passes its checks on released 0.14; Hatfield locks a tested 0.14-capable Codex package revision along with the 0.14 dependency set; provider conversion and replay remain covered. Coordinate with extraction PR #530, which currently uses the earlier package revision. Do not push the public standalone package without explicit user approval.

## Task workflow update - 2026-09-27T00:32:48+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter/.idea.

## Task workflow update - 2026-09-27T00:38:43+00:00
- Ownership: owner=fork; fork_run=pending-launch; revision=package checkout /home/ineersa/projects/symfony-ai-openai-codex-platform @ f721528 (== origin/main); scope=Slice A: enable + validate Symfony AI 0.14 in the standalone Codex package (constraints to include ^0.14, migrate only verified API fallout, keep Codex-only replay/tool IDs/request shape/error handling/WebSocket behavior, run package Castor test/phpstan/cs-check/composer validate against resolved 0.14, commit on a branch, NO push without explicit user approval); outcome=assigned; commit=none
- Ownership: owner=main; fork_run=none; revision=task/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter @ c66e2ce6f; scope=Slice B: root composer.json/lock to tagged symfony/ai-* 0.14 (no dev-main, no aliases, no path repos; mcp/sdk unchanged; no broad update), observational-memory manifest to ^0.14 + MemoryStoreAdapter Countable/VectorDocumentInterface migration preserving filtering/truncation/search, SemanticIndexService typing coherence, affected-caller migration for verified 0.14 changes, audits (tool-error dead branches, Execution::cancel vs Hatfield cancellation, toolbox/FiberToolExecutor/TokenUsage::getModel/replay redaction reuse) per task decision rules; blocked-by=Slice A tested revision; outcome=assigned; commit=none

## Task workflow update - 2026-09-27T00:42:40+00:00
- Recorded fork run: agent_b5ca3051920f1651
- Slice A complete: fork agent_b5ca3051920f1651; package branch symfony-ai-014 @ 8fb96d3 (from f721528); constraints symfony/ai-platform + symfony/ai-open-responses-platform widened to ^0.12 || ^0.13 || ^0.14; lock resolves both to tagged v0.14.0, no dev branches/aliases; NO src/tests migration needed (all 0.14 audit items dispositioned keep with rationale: #2533 Codex composite tool IDs preserved by own normalizers, #2504 wire encoding kept (request shape), #2475/#1566 streamed usage already via OpenResponses extractor, #2447/#2477 upstream-only, #2408/#2557/#2501 not used by package); package QA green on 0.14: castor test 206/1012, phpstan clean, cs-check clean (82 files), composer validate ok; main reviewed diff (2 files, 4 constraint lines); NOT pushed per approval gate. Ownership: owner=fork; fork_run=agent_b5ca3051920f1651; revision=package symfony-ai-014 @ 8fb96d3; scope=Slice A Codex package 0.14 enablement; outcome=completed; commit=8fb96d3fceaf21d2b91d56be38ca908a94b31619

## Task workflow update - 2026-09-27T02:36:58+00:00
- Validation: castor test: 4844 tests, 21910 assertions OK (includes OM extension suite via phpunit.xml.dist); castor test --filter=RegistryBackedToolboxTest: 49 tests OK after exception-translation fix; castor test --filter=DurableResultConverterTest: 30 tests OK after finish_reason flush fix; castor test:controller-replay: 13 tests, 218 assertions OK (44.4s); castor phpstan: 0 errors; castor cs-check: 0 fixes; castor deptrac: 0 violations; castor phar:build + PHAR smoke: ok (help/version/cache isolation); castor test:llm-real (warm): 5 tests, 30 assertions OK; No worktree process leaks; observed agent --controller/messenger:consume belong to the live harness session in the main checkout (left untouched per root AGENTS.md)
- Summary: Slice B complete (root + OM migration), commit 66d78f426 on task branch. All seven symfony/ai-* constraints → ^0.14; lock resolves tagged v0.14.0 + Codex package dev-main@8fb96d3 (slice A, merged/published earlier). Code migrations: MemoryStoreAdapter (Countable + VectorDocumentInterface widening, behavior parity incl. FTS re-tokenization/truncation), SemanticIndexService closure widening, RegistryBackedToolbox resolver-origin InvalidToolCallArgumentsException→non-retryable ToolCallException translation with ConstraintViolationList distinguisher (getToolCallResult() falls back to message string; listener-origin passes through raw to FaultTolerantToolbox), DurableResultConverter flush-on-any-finish_reason with phantom-block suppression. Audits concluded: vendor Execution\Cancellation byte-identical old↔new and unconsumed (Hatfield RunCancellationToken retained, DB-status-driven); no withToolbox/Traceable* callers (#2602/#2487/#2524 N/A); no Hatfield TokenUsageInterface implementers, convertStreamUsage() called without new optional $model param = 0.13 parity decision.

## Task workflow update - 2026-09-27T03:08:04+00:00
- Validation: Post-fix focused: castor test --filter=DurableResultConverterTest: 31 tests, 144 assertions OK (incl. new flushesFinalToolCallsOnStopFinishReason); castor cs-check: 0 fixes; castor phpstan: 0 errors; Reviewer read-only focused runs in worktree: RegistryBackedToolboxTest 49/124 OK, DurableResultConverterTest 30/138 OK, MemoryStoreAdapterTest|SemanticIndexServiceTest 18/392 OK; Warm castor test:llm-real 5/30 OK recorded earlier this revision cycle (provider-visible change)
- Summary: task-to-pr phase: reviewer subagent agent_bae5b70a15ffd197 (role: reviewer, specification-fidelity + senior review) reviewed revision 66d78f426 vs merge-base 5685998fe across all 8 diff files plus vendored 0.14.0 sources. VERDICT: APPROVE WITH SUGGESTIONS — no CRITICAL/BUG; SEC note acceptable (violation list stringifies via normalizeResultText, no new disclosure); EDGE note documents the ConstraintViolationList distinguisher coupling; OBS note: ToolCallFailedEvent.exception now carries InvalidToolCallArgumentsException instead of the 0.13 unwrapped cause — verified ExtensionToolHookEventSubscriber only consumes getMessage()/::class, no regression. Applied both NTH fixes in commit 8b1437842: positive flush-on-stop regression test (DurableResultConverterTest, 31 tests OK) and stale v0.11 docblock → v0.14. Reviewer independently verified lock scope (only symfony/ai-* + codex 8fb96d3 moved; mcp/sdk v0.8.1, vektor, tui fork untouched), StoreInterface parity, exception-translation end-to-end paths, converter gate equivalence-plus-phantom-safety, and no stray ToolException references.

## Task workflow update - 2026-09-27T03:09:58+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (100.0s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter/var/reports/qa-20260927-030818-31076-7a47e2dc.
- Session/run: 70.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-27T03:10:00+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter/var/reports/qa-20260927-030818-31076-7a47e2dc.
- Session/run: 70.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-27T03:10:03+00:00
- castor check passed (100.0s).
- Pushed task/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter to origin.
- Created PR: <url>
- Session/run: 70.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-27T03:10:03+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (100.0s).
- Pushed task/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/534

## Task workflow update - 2026-09-27T18:43:56+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Accepted user feedback: remove argument-exception origin sniffing and adopt native argument errors at ToolExecutor boundary. IsolatedAgentToolbox inheritance refactor remains outside this revision.

## Task workflow update - 2026-09-27T18:44:03+00:00
- Ownership: owner=main; fork_run=none; revision=8b1437842; scope=remove RegistryBackedToolbox argument-origin translation; classify native argument errors in ToolExecutor and update behavioral tests; outcome=assigned; commit=none

## Task workflow update - 2026-09-27T18:54:27+00:00
- Validation: Focused Castor tests: RegistryBackedToolboxTest|ToolExecutorTest 73 tests, 228 assertions OK; castor phpstan: zero errors; castor deptrac: zero violations; Removed toolbox retryability assertions are now protected by ToolExecutorTest::testArgumentResolutionFailureBecomesActionableNonRetryableErrorResult (wrong-type and missing-field rows) and testConstraintViolationBecomesActionableNonRetryableErrorResult. Native chain remains covered by RegistryBackedToolboxTest::testDenormalizerFailurePreservesNativeExceptionChain.
- Summary: User-approved native adaptation committed as 02ff00f88. Removed ConstraintViolationList origin sniffing from RegistryBackedToolbox; native InvalidToolCallArgumentsException now propagates to ToolExecutor and is classified non-retryable for both resolver and validator failures. error_type intentionally uses the native exception class. Existing native-message sanitization and cancelled-run stale-result handling now apply uniformly. Reviewer agent_dff2344e2cee478f reviewed 02ff00f88 vs 8b1437842, role=code review/specification fidelity, verdict=APPROVE; no blockers. Earlier adoption audit corrected: AbstractToolbox and Execution::cancel already existed at old dev-main pin; checking Cancellation.php alone was insufficient.
- Ownership: owner=main; fork_run=none; revision=8b1437842; scope=native argument failures at ToolExecutor boundary and behavioral tests; outcome=completed; commit=02ff00f88

## Task workflow update - 2026-09-27T18:55:55+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter.md.
- Session/run: 70.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter/var/reports/qa-20260927-185459-2017-1c469acc.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-27T18:59:32+00:00
- Validation: Focused two corrected tests: 2 tests / 3 assertions OK; castor test:llm-real at 02ff00f88: 5 tests / 30 assertions OK; subsequent changes tests only
- Summary: Gate found two stale ToolCallException expectations in BashToolTest/BgStatusToolTest. Fixed in 06480ab8c to assert native FaultTolerantToolbox messages, retaining Bash no-process side effects. Reviewer agent_dff2344e2cee478f re-reviewed combined 8b1437842..06480ab8c and APPROVED. No remaining blockers. Removed harness retryability assertion remains covered by ToolExecutorTest argument-resolution/validation cases.
- Ownership: owner=main; fork_run=none; revision=02ff00f88; scope=fix two stale argument-wrapper test expectations found by full gate; outcome=completed; commit=06480ab8c

## Task workflow update - 2026-09-27T19:00:48+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (56.5s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter/var/reports/qa-20260927-185952-5630-2604660f.
- Session/run: 70.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-27T19:00:50+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter/var/reports/qa-20260927-185952-5630-2604660f.
- Session/run: 70.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-27T19:00:51+00:00
- castor check passed (56.5s).
- Pushed task/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter to origin.
- PR already exists: <url>
- Session/run: 70.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-27T19:00:51+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (56.5s).
- Pushed task/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/534
- Summary: Native argument-error handling reviewed and approved at 06480ab8c; stale tool-harness expectations fixed.

## Task workflow update - 2026-09-27T19:10:10+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-25-upgrade-symfony-ai-to-0-14-and-migrate-memory-store-adapter.
- Pulled integration checkout: Already up to date..
- Summary: GitHub confirms PR #534 merged at 9c7079eaa. Integration checkout clean; proceeding with post-merge validation.

## Task workflow update - 2026-09-27T19:13:58+00:00
- Updated PR Status: merged
- Validation: castor check: all 11 lanes passed; report var/reports/qa-20260927-191247-12655-a9b7f2a1; Unit/integration: 4856 tests, 21968 assertions; controller replay: 14/253; TUI: 6/40; live LLM: 5/30; QA leak assertion and llama-proxy cache guard passed
- Summary: Post-merge validation complete on integration 63ca629fb. Initial gate loaded old dependency contracts and failed test/phpstan/dead-code; composer install regenerated autoload files with no package changes. Focused argument tests and PHPStan then passed, followed by the full post-merge gate. Git status clean and task worktree removed.
