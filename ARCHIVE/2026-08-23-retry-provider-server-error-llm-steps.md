# Retry transient provider server_error LLM step failures

## Goal
Bug observed in Hatfield session 45 for architect child `agent_146454fb5116b318`, run `b497e907-883d-5033-83bc-e608251779f9`, model `openai-codex/gpt-5.6-sol`. The child completed 23 LLM steps, then failed terminally at 2026-08-23T15:22:04Z with `LLM provider error: [server_error/server_error]`. It showed no retry attempt and surfaced as `architect [failed]`.

Evidence from `.hatfield/sessions/45/artifacts/agents/agent_146454fb5116b318/`:
- `state.json`: `status=failed`, `retryable_failure=false`, `retry_attempts=0`, `error_message=LLM provider error: [server_error/server_error]`.
- `events.jsonl` seq 586: `llm_step_failed`, provider category, `retryable=false`, `retry_attempt=0`, `max_retries=2`, step `advance-after-tools-4525563952646`; no subsequent retry/completion event.
- Metadata/registry maps artifact to child run `b497e907-883d-5033-83bc-e608251779f9`, created 15:13:11Z and terminal at 15:22:04Z.

Root cause found by scout: `LlmProviderErrorClassifier` recognizes transient HTTP 500/502/503/504 and exact structured signals `server_is_overloaded` and `service_unavailable_error`, but not the exact provider code `server_error/server_error`. It falls through to provider/non-retryable. `LlmPlatformAdapter` converts the exception into a normal failed invocation result, so Messenger never receives a thrown exception and Messenger retry middleware is not involved. `LlmStepResultHandler` only schedules the configured AgentCore retry when the classifier sets `retryable=true`; therefore this child was immediately projected terminal failed.

Datadog warning-only evidence matches current implementation: provider stream failures and `llm.request.failed` are logged at warning level; this path does not emit an error log. The task should intentionally define telemetry for retry scheduling and terminal exhaustion rather than assuming Messenger or SDK retries occurred.

## Acceptance criteria
- Add an exact, structured classification for provider `server_error/server_error` as transient/retryable. Do not use broad `server_error` substring matching that could classify unrelated permanent failures as retryable.
- A matching failure persists `llm_step_failed.retryable=true` and schedules the existing bounded AgentCore LLM-step retry path using configured retry limits/delay; do not add a second retry framework.
- Regression proof shows the first matching provider failure advances the retry attempt and causes another LLM invocation instead of immediately making the child terminal.
- While a retry is scheduled/pending, deferred subagent projection and batch completion must not publish a terminal failed child/tool result.
- If all configured attempts fail, the child becomes terminal failed exactly once with observable attempt/max-attempt information and no infinite retry loop.
- Successful retry resumes the same child run/step lifecycle correctly and produces the normal completion projection without requiring manual resume or parent intervention.
- Keep retry layers distinct in implementation and tests: this incident is handled by AgentCore LLM-step retry classification, not Messenger redelivery, SDK retry assumptions, or automatic child relaunch.
- Add structured, privacy-safe telemetry for classification and retry decision/scheduling with run_id/session_id, component, event_type, error category/code, retry attempt, and max attempts. Do not log prompts, transcript content, tool output, credentials, or raw provider payloads.
- Define and test log severity intentionally: transient attempt failures may remain warnings, but terminal exhaustion must be clearly distinguishable in telemetry. Do not create duplicate warning/error records for one transition.
- Add focused classifier tests for the exact structured code plus nearby permanent/unknown provider errors proving they remain non-retryable.
- Add pipeline/deferred-child regression coverage proving retry scheduling, successful recovery, and terminal exhaustion behavior. Prefer the lowest correct deterministic/replay layer; no real provider call is required unless deterministic coverage cannot prove the contract.
- Load and follow `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before test work. Run focused Castor tests, `castor deptrac`, `castor phpstan`, `castor cs-check`, and `castor check` before CODE-REVIEW because this changes LLM-visible/runtime child flow.
- Preserve existing context-overflow and explicitly permanent-provider handling; do not retry authentication, validation, policy, malformed request, or other known non-transient failures.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-23-retry-provider-server-error-llm-steps
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-23-retry-provider-server-error-llm-steps
Fork run: wszm2co7kcu6
PR URL: https://github.com/ineersa/agent-core/pull/441
PR Status: merged
Started: 2026-08-28T21:29:56.413Z
Completed: 2026-08-29T01:50:05.326Z

## Work log
- Created: 2026-08-23T15:41:07+00:00

## Task workflow update - 2026-08-23T15:43:48+00:00
- Validation: Add classifier matrix tests proving known permanent provider failures are non-retryable while unknown provider failures default retryable.; Regression case `[server_error/server_error]` must pass through the default retry behavior without a dedicated transient allowlist entry.; Prove bounded retry/exhaustion prevents infinite retries even when unknown provider errors remain retryable.; Audit current classifier call sites so the inverted default applies only to provider invocation failures and cannot retry coding/configuration/serialization/cancellation failures accidentally.
- Summary: Architecture direction updated per user: invert provider retry classification. Provider-operation failures should be retryable by default under the existing bounded AgentCore retry policy; the classifier should identify the finite set of known non-retryable/permanent failures rather than maintain an ever-growing allowlist of transient strings. The original `server_error/server_error` incident becomes a regression case proving unknown provider failures retry automatically, not a one-off string addition.
- Replace transient-signal allowlisting as the primary decision model with permanent-error denylisting/default-retry semantics at the provider-operation boundary.
- Keep the scope precise: only failures already established as provider-operation failures default retryable. Local programming errors, cancellation, invalid internal state, and non-provider exceptions must not become retryable merely because classification is unknown.
- Known permanent/non-useful retry cases should remain non-retryable, including invalid authentication/authorization, malformed or invalid request, unsupported model/feature, safety/policy rejection, context overflow handled by its dedicated path, and exhausted/insufficient quota. Distinguish temporary rate limiting/capacity from permanently exhausted quota; temporary throttling remains retryable with existing bounded delay/Retry-After behavior where available.
- Unknown or newly introduced provider server failures should retry without requiring new string literals. Exact provider codes/statuses may still be parsed to identify permanent cases, categorize telemetry, or honor retry delay, but not to grant retry eligibility one transient code at a time.

## Task workflow update - 2026-08-28T21:29:56.413Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-23-retry-provider-server-error-llm-steps.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-23-retry-provider-server-error-llm-steps.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-23-retry-provider-server-error-llm-steps.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-23-retry-provider-server-error-llm-steps.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-23-retry-provider-server-error-llm-steps.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-23-retry-provider-server-error-llm-steps.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-23-retry-provider-server-error-llm-steps/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-23-retry-provider-server-error-llm-steps.
- Summary: Implementation started. Finalized direction: unknown failures already established at the provider-operation boundary default retryable under the existing bounded AgentCore LLM-step policy; known permanent/local/internal/cancellation failures remain non-retryable. AgentCore scheduling will honor provider Retry-After within the configured delay cap. Existing completion events/logs remain the recovery signal; no duplicate retry framework or dedicated recovery log will be added.

## Task workflow update - 2026-08-28T21:30:07.483Z
- Ownership: owner=main; fork_run=none; revision=84b08473a09f304ea20f3274a4b2b50a380668ea; scope=provider-boundary permanent/default-retry classification, Retry-After-aware existing AgentCore scheduling, privacy-safe transition telemetry, deterministic classifier/pipeline/deferred-child regression proof, focused validation; outcome=assigned; commit=none

## Task workflow update - 2026-08-28T21:48:33.186Z
- Validation: Focused classifier/adapter/handler/continue/deferred-child/batch lifecycle tests — PASS (130 tests, 830 assertions).; Focused worker and session replay tests — PASS (53 tests, 257 assertions).; castor deptrac — PASS (0 violations/errors).; castor phpstan — PASS (0 errors).; castor cs-check — PASS (clean).; git diff --check and git show --check — PASS; worktree clean.; IDE diagnostics for LlmProviderErrorClassifier.php and LlmStepResultHandler.php — 0 problems.
- Summary: Implementation completed at 80af0a087. Unknown errors already normalized by LlmPlatformAdapter at the provider-operation boundary now default retryable under the existing bounded AgentCore policy; exact known permanent auth/request/model/policy/quota/context, cancellation, and local programming failures remain non-retryable. AgentCore scheduling honors Retry-After using the greater configured/provider delay capped by agent_retry.max_delay_ms. Scheduling emits one privacy-safe info transition and terminal exhaustion one error transition after commit; existing worker warnings retain classification telemetry without a duplicate retry warning. Exhaustion attempt counters now remain bounded at max_retries. Successful retry clears stale deferred-child error state, and pending retry remains nonterminal through deferred child and batch completion projection.
- Ownership: owner=main; fork_run=none; revision=84b08473a09f304ea20f3274a4b2b50a380668ea; scope=provider-boundary permanent/default-retry classification, Retry-After-aware existing AgentCore scheduling, privacy-safe transition telemetry, deterministic classifier/pipeline/deferred-child regression proof, focused validation; outcome=completed; commit=80af0a087

## Task workflow update - 2026-08-28T22:26:53.520Z
- Summary: Pi reviewer a34a6033 reviewed revision 80af0a087 against origin/main with specification-fidelity, testing-skill, and tests/AGENTS.md requirements. Verdict: REQUEST CHANGES. Required code blocker: permanent no-status provider codes usage_limit_reached, invalid_argument, and validation_error currently reach bounded default retry and must be denylisted with focused matrix proof. The reviewer also noted castor check is required before CODE-REVIEW; that gate is intentionally performed by the subsequent move_task transition after blockers and re-review. Non-blocking suggestions: remove contradictory '(retryable)' wording on terminal exhaustion, document the per-apply deferred retry-pending assumption, and optional transport-routing proof.
- Reviewer assignment/result: role=Pi reviewer; run_id=a34a6033; artifact=/home/ineersa/.pi/agent/sessions/--home-ineersa-projects-agent-core--/subagent-artifacts/a34a6033_reviewer_output.md; target=80af0a087; base=origin/main; scope=specification fidelity, correctness/security/privacy, retry and delayed-dispatch semantics, replay/projection/batch terminality, telemetry, proof quality, dead code/fallbacks; verdict=REQUEST CHANGES.

## Task workflow update - 2026-08-28T22:28:38.742Z
- Validation: Post-review focused tests (LlmProviderErrorClassifierTest|LlmStepResultHandlerTest|DeferredChildRunEventProjectorTest) — PASS (85 tests, 345 assertions).; Post-review castor deptrac — PASS (0 violations/errors).; Post-review castor phpstan — PASS (0 errors).; Post-review castor cs-check — PASS (clean).; Post-review git diff --check — PASS.
- Summary: Resolved Pi reviewer a34a6033 code blocker in b2876d02a: no-status usage_limit_reached is terminal quota/billing; invalid_argument and validation_error are terminal malformed-request signals; classifier matrix covers all three. Also removed stale '(retryable)' wording from exhausted terminal messages and documented the deferred projector's per-apply retry-pending invariant. Focused affected tests and static/architecture/style gates pass. Awaiting reviewer re-review before CODE-REVIEW transition.
- Review iteration ownership: owner=main; source_reviewer=a34a6033; blocker_scope=permanent structured quota/request denylist proof; outcome=resolved; commit=b2876d02a.

## Task workflow update - 2026-08-28T22:39:29.350Z
- Summary: Pi reviewer re-review e6591acc approved revision b2876d02a with non-blocking suggestions. All prior blockers are resolved; complete diff has no critical/bug/security/specification-fidelity/dead-code findings. Remaining notes are optional documentation/comment/routing-proof refinements and do not block CODE-REVIEW. Deterministic castor check remains the pending transition gate.
- Reviewer re-review: role=Pi reviewer; run_id=e6591acc; artifact=/home/ineersa/.pi/agent/sessions/--home-ineersa-projects-agent-core--/subagent-artifacts/e6591acc_reviewer_output.md; target=b2876d02a; base=origin/main; scope=prior blocker resolution plus full specification-fidelity/correctness/security/retry-state/proof re-scan; verdict=APPROVE WITH SUGGESTIONS.

## Task workflow update - 2026-08-28T22:40:18.345Z
- Validation: Final focused Castor tests covering classifier, adapter, handler, Continue application, worker, state replay, deferred child projection, and deferred batch lifecycle — PASS (186 tests, 1095 assertions).; Final castor deptrac — PASS (0 violations/errors).; Final castor phpstan — PASS (0 errors).; Final castor cs-check — PASS (clean).; Final git diff --check — PASS; worktree clean.; Reviewer e6591acc — APPROVE WITH SUGGESTIONS at b2876d02a; no blocking findings.
- Summary: task-to-pr preflight complete at b2876d02a. Independent Pi reviewer e6591acc verdict APPROVE WITH SUGGESTIONS; no unresolved blockers. Branch is clean and based on current origin/main (ahead 2, behind 0). Focused deterministic proof and required static/architecture/style gates all pass. Focused live LLM was not added because no real provider can deterministically induce the structured server failure and the task explicitly maps this contract to classifier/pipeline/replay proof; the CODE-REVIEW transition's full castor check supplies the required controller/TUI/llm-real runtime gate.
- CODE-REVIEW preflight: target=b2876d02a; base=origin/main@84b08473a; ahead=2; behind=0; unresolved_blockers=none; pending_gate=move_task deterministic castor check.

## Task workflow update - 2026-08-28T22:41:42.782Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (70.4s).
- Pushed task/2026-08-23-retry-provider-server-error-llm-steps to origin.
- branch 'task/2026-08-23-retry-provider-server-error-llm-steps' set up to track 'origin/task/2026-08-23-retry-provider-server-error-llm-steps'.
- Created PR: https://github.com/ineersa/agent-core/pull/441
- Validation: Pi reviewer e6591acc — APPROVE WITH SUGGESTIONS; no blockers.; Focused Castor suite — PASS (186 tests, 1095 assertions).; castor deptrac — PASS.; castor phpstan — PASS.; castor cs-check — PASS.; git diff --check — PASS; clean worktree.
- Summary: Prepared for code review at b2876d02a after Pi reviewer e6591acc APPROVE WITH SUGGESTIONS. The change implements bounded default retry for unknown provider-operation failures while denylisting known permanent/local/cancellation failures, honors Retry-After within configured caps, emits privacy-safe scheduled/exhausted telemetry, preserves bounded attempt state, and keeps deferred child/batch lifecycle nonterminal until recovery or exhaustion.

## Task workflow update - 2026-08-28T22:50:21.385Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User requested a focused simplification pass on PR #441 after questioning its +610/-103 size. No PR inline comments exist. Review iteration will preserve finalized retry behavior, privacy, attempt accounting, deferred lifecycle behavior, and stable contracts while deleting unnecessary additive implementation/test complexity. Main owns the cohesive simplification slice; no scout is needed because the implementation and prior review context are already fully loaded.

## Task workflow update - 2026-08-28T22:50:25.332Z
- Ownership: owner=main; fork_run=none; revision=b2876d02a; scope=complete PR #441 simplification against origin/main, preserving finalized provider-boundary bounded-default retry, permanent failures, Retry-After cap, privacy-safe telemetry, retry attempt/state replay, and deferred child/batch lifecycle behavior while minimizing production/test diff; outcome=assigned; commit=none

## Task workflow update - 2026-08-28T23:27:29.791Z
- Validation: Three independent simplify scouts da72494c completed read-only full-PR audits and followed testing instructions.; Focused Castor tests after final fixes — PASS (186 tests, 1095 assertions).; castor deptrac — PASS (0 violations).; castor phpstan — PASS (0 errors).; castor cs-check — PASS.; Full castor check qa-20260828-230443-515688-06a84ca8 on near-final simplification tree — PASS (all lanes).; Exact-final-tree castor check qa-20260828-231744-533357-b82d6312 — all lanes PASS except unrelated ConsumerSupervisorTest::testShutdownSignalsAllTrackedConsumersBeforeSharedGraceWait ready-marker contention in unit lane; leak/cache/artifact finalizers PASS. Not retried.; Final reviewer a8b54495 — APPROVE WITH SUGGESTIONS; no code blockers; pending green exact-revision gate only.
- Summary: Completed the user-requested simplify pass across all of PR #441. Three independent scouts (group da72494c: reuse/architecture, simplicity/tests, runtime/efficiency) audited the complete diff. Accepted simplifications consolidated exception/signal classification, eliminated duplicate structured parsing and dead helpers/guards/state, replaced duplicate envelope history with lastEnvelope, and preserved permanent-before-transient precedence. The PR shrank from +610/-103 (net +507) to +561/-118 (net +443), a net reduction of 64 lines, while adding a correctness guard for permanent HTTP status over transient wrapper types. Final reviewer a8b54495: APPROVE WITH SUGGESTIONS, no code blockers. Task remains IN-PROGRESS because the exact-final-tree castor check had one unrelated ConsumerSupervisorTest ready-marker contention failure; all other lanes and focused validation passed. Per test policy this was not blindly retried.
- Ownership: owner=main; fork_run=none; revision=8d3e21f39; scope=complete PR #441 simplification preserving finalized behavior; outcome=completed; commit=be17fd273,ef122664e
- Simplify audits: role=three independent scouts; run_group=da72494c; target=b2876d02a; lenses=reuse/architecture,simplicity/test-quality,runtime/efficiency; outcome=completed.
- Final review: role=Pi reviewer; run_id=a8b54495; artifact=/home/ineersa/.pi/agent/sessions/--home-ineersa-projects-agent-core--/subagent-artifacts/a8b54495_reviewer_output.md; target=8d3e21f39; base=origin/main; verdict=APPROVE WITH SUGGESTIONS; unresolved_code_blockers=none; transition_blocker=exact-final-revision castor check unit-lane contention failure.

## Task workflow update - 2026-08-28T23:52:16.284Z
- Summary: User rejected custom HTTP retry/timing logic and parsing provider error messages. Latest clarification overrides prior implementation: use Symfony HttpClient retry facilities/strategies at the provider transport boundary; classify/retry from stable typed exceptions, HTTP status codes, headers, and structured public response fields only; remove all actual error-message parsing and superseded retry infrastructure rather than maintaining provider prose patterns.
- Ownership: owner=main; fork_run=none; revision=8d3e21f39; scope=replace custom provider message classification/retry timing with Symfony HttpClient retry strategy driven only by stable transport contracts (status/headers/typed exceptions), delete message parsing and superseded AgentCore retry machinery/tests while preserving required terminal/recovery/deferred lifecycle semantics; outcome=assigned; commit=none

## Task workflow update - 2026-08-29T00:26:36.273Z
- Validation: Focused Castor tests after Symfony retry refactor — PASS (156 tests, 1020 assertions).; castor deptrac — PASS (0 violations).; castor phpstan — PASS (0 errors).; castor cs-check — PASS.; castor docs:validate — PASS.; git diff --check — PASS.; IDE diagnostics — no problems in classifier, HTTP policy, and handler; one pre-existing empty-body warning in Codex ResultConverter.
- Summary: Reworked PR #441 per user review: Symfony RetryableHttpClient + GenericRetryStrategy now own transport errors, retryable HTTP statuses, exponential backoff, and standard Retry-After handling. Deleted the custom RetryStrategy implementation and every response-body/exception-message-based retry decision. Provider classification now uses only typed Symfony AI/HttpClient exceptions and HTTP status codes; unknown RuntimeException prose is non-retryable. Codex structured stream server_error/internal_error events are converted at the event boundary into Symfony AI ServerException, so the existing bounded AgentCore retry remains only for typed mid-stream failures that HttpClient cannot replay after streaming has begun. AgentCore custom delay/Retry-After parsing and agent_retry delay settings were removed. The PR is now net -1144 lines against main.
- Ownership: owner=main; fork_run=none; revision=8d3e21f39; scope=replace custom provider message classification/retry timing with Symfony HttpClient retry strategy driven only by stable transport contracts (status/headers/typed exceptions), delete message parsing and superseded AgentCore retry machinery/tests while preserving required terminal/recovery/deferred lifecycle semantics; outcome=completed; commit=b02d0499f

## Task workflow update - 2026-08-29T00:45:18.479Z
- Summary: Independent reviewer on model deepseek/deepseek-v4-flash returned REQUEST CHANGES for b02d0499f. Core Symfony HttpClient redesign was verified correct. Blocking cleanup: remove unused retry_after_seconds diagnostic write, orphaned retry_after_ms runtime passthrough, unreferenced CATEGORY_QUOTA_BILLING constant, and remaining message-regex context-overflow decision in Generic DurableResultConverter. Exact-final-revision castor check remains required after fixes.
- Reviewer: model=deepseek/deepseek-v4-flash; revision=b02d0499f; verdict=REQUEST CHANGES; no edits performed.

## Task workflow update - 2026-08-29T00:54:42.471Z
- Recorded fork run: wszm2co7kcu6
- Ownership: owner=fork; fork_run=wszm2co7kcu6; revision=b02d0499f; scope=remove four reviewer blockers only: dead retry_after_seconds write, orphaned retry_after_ms passthrough, unused quota category constant, and remaining message-regex context-overflow classification with focused proof; outcome=assigned; commit=none

## Task workflow update - 2026-08-29T00:57:58.851Z
- Recorded fork run: wszm2co7kcu6
- Validation: castor test --filter='(DurableResultConverterTest|LlmPlatformAdapterTest|LlmProviderErrorClassifierTest|RuntimeEvent)' — PASS (235 tests, 984 assertions).; castor deptrac — PASS (0 violations).; castor phpstan — PASS (0 errors).; castor cs-fix / castor cs-check — PASS.; git diff --check — PASS.; IDE diagnostics — no errors; one unrelated pre-existing empty-body warning in DurableResultConverter.
- Summary: Fork wszm2co7kcu6 completed all four review blockers at commit 0485da7e2: removed dead retry_after_seconds diagnostic, orphaned retry_after_ms passthrough, unused quota category constant, and message-prose context-overflow classification; added lowest-layer structured-code-versus-identical-prose converter proof. Worktree clean.
- Ownership: owner=fork; fork_run=wszm2co7kcu6; revision=b02d0499f; scope=remove four reviewer blockers only: dead retry_after_seconds write, orphaned retry_after_ms passthrough, unused quota category constant, and remaining message-regex context-overflow classification with focused proof; outcome=completed; commit=0485da7e2195c3f92576e4c00b60475094423778

## Task workflow update - 2026-08-29T01:07:16.685Z
- Validation: Reviewer reran focused suites at 0485da7e — PASS (157 tests, 1021 assertions total).; Reviewer reran castor deptrac/phpstan/cs-check — PASS.
- Summary: Independent re-review at 0485da7e with deepseek/deepseek-v4-flash returned APPROVE WITH SUGGESTIONS. All four prior blockers were verified resolved; complete PR rescan found no code/security/spec/dead-code blockers. Remaining notes are non-blocking operational observations. Exact-final-revision castor check is the only pending CODE-REVIEW transition gate.
- Reviewer: role=reviewer; model=deepseek/deepseek-v4-flash; revision=0485da7e2195c3f92576e4c00b60475094423778; scope=verify four prior blockers and full PR #441 redesign; decision=APPROVE WITH SUGGESTIONS; commit=0485da7e2195c3f92576e4c00b60475094423778; unresolved_blockers=exact-final castor check only.

## Task workflow update - 2026-08-29T01:09:09.380Z
- Validation: castor check qa-20260829-010723-618691-15b6f5b7 — FAIL only test:tui (1 stale expectation); deptrac, test (4845/19850), controller-replay, llm-real, phpstan, cs-check, docs, catalog all PASS; leak/cache/integrity checks PASS.
- Summary: Exact-final castor check at 0485da7e failed only the TUI replay lane: TuiProviderErrorE2eTest still expected the old word 'retryable' for an HTTP 429 after Symfony retries are exhausted, while the new correct terminal message is 'rate limit remained active after HTTP retries were exhausted.' All other 8 lanes passed, including 4845 unit tests and live llm-real.
- Ownership: owner=main; fork_run=none; revision=0485da7e2195c3f92576e4c00b60475094423778; scope=update stale TUI replay expectation/documentation to assert terminal HTTP-retry exhaustion and absence of retryable promise, then focused TUI validation; outcome=assigned; commit=none

## Task workflow update - 2026-08-29T01:09:43.979Z
- Validation: castor test:tui --filter=TuiProviderErrorE2eTest — PASS (1 test, 7 assertions).; castor cs-check — PASS.; git diff --check — PASS.
- Summary: Updated the stale TUI replay gate at fd98196fe: HTTP 429 after Symfony retry exhaustion now asserts the sanitized terminal exhaustion message and explicitly rejects a further 'retryable' promise. Focused TUI proof passes.
- Ownership: owner=main; fork_run=none; revision=0485da7e2195c3f92576e4c00b60475094423778; scope=update stale TUI replay expectation/documentation to assert terminal HTTP-retry exhaustion and absence of retryable promise, then focused TUI validation; outcome=completed; commit=fd98196fe

## Task workflow update - 2026-08-29T01:18:15.388Z
- Summary: Final independent re-review at fd98196fe with deepseek/deepseek-v4-flash returned APPROVE WITH SUGGESTIONS. The TUI gate fix accurately asserts the observed sanitized terminal HTTP retry-exhaustion contract, rejects a retry promise, preserves sentinel privacy proof, and introduces no blocker. Exact-final castor check remains the sole transition gate.
- Reviewer: role=reviewer; model=deepseek/deepseek-v4-flash; revision=fd98196fe; scope=final TUI gate fix plus complete PR regression scan; decision=APPROVE WITH SUGGESTIONS; commit=fd98196fe; unresolved_blockers=exact-final castor check only.

## Task workflow update - 2026-08-29T01:21:06.971Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (71.6s).
- Pushed task/2026-08-23-retry-provider-server-error-llm-steps to origin.
- branch 'task/2026-08-23-retry-provider-server-error-llm-steps' set up to track 'origin/task/2026-08-23-retry-provider-server-error-llm-steps'.
- PR already exists: https://github.com/ineersa/agent-core/pull/441
- Validation: Exact-final castor check qa-20260829-011818-631953-700fe24a — PASS, quality ok in 155.6s.; Unit/integration lane — PASS (4845 tests, 19848 assertions).; Controller replay — PASS (6 tests, 88 assertions).; TUI replay — PASS (8 tests, 60 assertions).; Live llm-real — PASS (5 tests, 30 assertions).; Deptrac, phpstan, cs-check, docs:validate, catalog:version-check — PASS.; QA artifact integrity, process leak check, and llama-proxy cache guard — PASS.; Independent reviewer deepseek/deepseek-v4-flash — APPROVE WITH SUGGESTIONS at fd98196fe; no blockers.
- Summary: PR #441 redesign completed and independently approved at fd98196fe. Symfony RetryableHttpClient/GenericRetryStrategy now owns transport/status retries, backoff, and Retry-After without inspecting provider prose. AgentCore retry remains immediate/bounded only for typed mid-stream failures; custom strategy, message parsing, custom delay machinery, dead diagnostics, and stale tests were removed. Final PR is substantially smaller than main baseline and all review blockers are resolved.

## Task workflow update - 2026-08-29T01:50:05.326Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-23-retry-provider-server-error-llm-steps: ide_close_project returned isError.
- Merged task/2026-08-23-retry-provider-server-error-llm-steps into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                               |   2 -
 docs/settings-models.md                            |   2 +-
 .../Application/Pipeline/LlmStepResultHandler.php  | 106 +++--
 .../SymfonyAi/LlmPlatformAdapter.php               |  72 +--
 .../SymfonyAi/LlmProviderErrorClassifier.php       | 349 +++-----------
 .../Deferred/DeferredChildRunEventProjector.php    |   6 +-
 src/CodingAgent/Config/Ai/AiAgentRetryConfig.php   |  78 +---
 .../SymfonyAi/Http/LlmHttpRetryPolicy.php          | 211 +--------
 .../SymfonyAi/Http/LlmHttpRetryStrategy.php        | 103 -----
 .../SymfonyAi/SymfonyAiProviderFactory.php         |  22 +-
 .../Runtime/Protocol/RuntimeEventTranslator.php    |   2 +-
 .../Bridge/Generic/DurableResultConverter.php      |   2 +-
 .../Bridge/OpenAICodex/ResultConverter.php         |  26 +-
 .../Pipeline/ApplyCommandHandlerTest.php           |   6 +-
 .../Pipeline/LlmStepResultHandlerTest.php          | 125 ++++-
 .../SymfonyAi/LlmPlatformAdapterTest.php           | 102 ++--
 .../SymfonyAi/LlmProviderErrorClassifierTest.php   | 513 +++------------------
 .../DeferredSubagentBatchLifecycleTest.php         |  28 +-
 .../DeferredChildRunEventProjectorTest.php         |  36 +-
 .../Config/Ai/AiAgentRetryConfigTest.php           |  35 +-
 .../SymfonyAi/Http/LlmHttpRetryPolicyTest.php      | 292 ++----------
 .../SymfonyAi/Http/LlmHttpRetryStrategyTest.php    | 172 -------
 .../Bridge/Generic/DurableResultConverterTest.php  |  15 +-
 .../Bridge/OpenAICodex/ResultConverterTest.php     |  27 +-
 tests/Tui/E2E/TuiProviderErrorE2eTest.php          |  18 +-
 25 files changed, 608 insertions(+), 1742 deletions(-)
 delete mode 100644 src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpRetryStrategy.php
 delete mode 100644 tests/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpRetryStrategyTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-23-retry-provider-server-error-llm-steps.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-23-retry-provider-server-error-llm-steps.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Pre-merge exact-final castor check at fd98196fe — PASS (all 9 lanes, quality ok).; Independent reviewer deepseek/deepseek-v4-flash — APPROVE WITH SUGGESTIONS, no blockers.
- Summary: PR #441 merged on GitHub at f6cd66f3a3019b1f7103e4b580c1a03feadac46c. Final design delegates transport/status retries, exponential backoff, and Retry-After to Symfony HttpClient; AgentCore retains only immediate bounded retries for typed mid-stream failures. Provider prose parsing and superseded custom retry machinery were removed.

## Task workflow update - 2026-08-29T01:51:52.917Z
- Validation: Post-merge `LLM_MODE=true castor check` qa-20260829-015009-662385-178a2723 — PASS, quality ok in 170.5s.; Unit/integration 4845 tests / 20496 assertions; controller replay 6/88; TUI 8/60; llm-real 5/30; all static/docs/catalog/integrity/leak/cache lanes PASS.; Task worktree removed; integration working tree has no uncommitted changes.; Local main divergence: ahead of origin/main by redundant merge commits d18211480 and 98b801615; requires explicit user approval before realignment.
- Summary: Post-merge integration validation passed. Worktree removed. Integration working tree is clean, but move_task created a redundant local task-branch merge and then merged already-merged origin/main; local main is ahead of origin/main by two merge commits (d18211480, 98b801615). No history rewrite/reset performed without user approval.

## Task workflow update - 2026-08-29T16:09:39.043Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
