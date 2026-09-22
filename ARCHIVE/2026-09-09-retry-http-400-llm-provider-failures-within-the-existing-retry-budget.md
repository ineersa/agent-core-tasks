# Retry HTTP 400 LLM provider failures within the existing retry budget

## Goal
## Goal
Treat HTTP 400 LLM provider failures as retryable within the existing bounded retry mechanism. The user explicitly requested this because providers can return intermittent 400 responses. Do not add unlimited retries or new settings.

## Incident
On 2026-09-09, the reviewer in session `3` under `/home/ineersa/mcp-servers/mysql-server` failed twice while using `zai/glm-5.3` with medium reasoning. Both responses were HTTP 400 with content type `text/html`, a 171-byte body, and no structured JSON error. Hatfield classified both as `bad_request` with `retryable: false`, so neither received an automatic retry.

The parent explicitly resumed the reviewer after the first failure. Four LLM calls succeeded before the second failure. This supports allowing bounded retries, but does not establish why the upstream returned 400.

Reviewer run: `e93e5b90-e307-5a48-9074-e3e7c292527f`.

Evidence relative to the MySQL project:
- `.hatfield/logs/agent-2026-09-09.log:3633-3635`: first rejection at 14:04:40 UTC.
- `.hatfield/logs/agent-2026-09-09.log:4136-4138`: second rejection at 14:06:13 UTC.
- `.hatfield/sessions/3/artifacts/agents/agent_e2dce4eed922e32e/events.jsonl`: failures and successful calls between them.

## Scope
Use the existing provider-failure classification and retry budget. Include structured and non-JSON HTTP 400 responses. Preserve privacy-safe diagnostics and report the final failure after budget exhaustion. A retry must not re-execute previously completed tools.

This does not fix cross-provider history conversion. Permanently invalid requests may still exhaust the bounded retry budget.

## Acceptance criteria
- HTTP 400 LLM provider failures use the existing bounded automatic retry mechanism, including HTML and structured JSON responses.
- A subsequent successful response continues the run without requiring an explicit reviewer resume.
- Persistent HTTP 400 responses stop after the existing retry budget and report the terminal failure.
- Retrying an LLM request does not repeat previously completed tool executions; unrelated status-code handling remains unchanged.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/485
PR Status: merged
Started: 2026-09-09T14:15:39+00:00
Completed: 2026-09-09T16:05:48+00:00

## Work log
- Created: 2026-09-09T14:10:56+00:00

## Task workflow update - 2026-09-09T14:15:39+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget.

## Task workflow update - 2026-09-09T14:17:21+00:00
- Ownership: owner=main; fork_run=none; revision=1edbbb988; scope=HTTP 400 classification and focused regression validation using existing Messenger retry path; outcome=assigned; commit=none

## Task workflow update - 2026-09-09T14:19:44+00:00
- Validation: Focused classifier, adapter, execution tracing, and result-handler tests: 51 tests, 279 assertions passed.; Focused ExecuteLlmStepWorkerTest, LlmWorkerFailedEventSubscriberTest, and LlmHttpRetryPolicyTest: 20 tests, 101 assertions passed.; castor cs-check passed; git diff --check passed.
- Summary: Implemented HTTP 400 retry classification ahead of permanent exception classification, including BadRequestException HTML responses. Reuses the existing bounded Messenger retry path without changing retry configuration or tool execution. Added HTML, JSON, and generic exception regressions; non-400 handling remains unchanged. Full gate and independent review belong to task-to-pr.
- Ownership: owner=main; fork_run=none; revision=1edbbb988; scope=HTTP 400 classification and focused regression validation using existing Messenger retry path; outcome=completed; commit=60164d3ce

## Task workflow update - 2026-09-09T15:19:11+00:00
- Summary: User clarified: HTTP 400 must retry at the existing Symfony HTTP client layer with backoff and jitter, not Messenger. Supersedes initial Messenger implementation. Task remains IN-PROGRESS; no PR exists yet.
- Ownership: owner=main; fork_run=none; revision=60164d3ce; scope=replace Messenger classification change with HTTP retry policy and recovery/exhaustion tests; outcome=assigned; commit=none

## Task workflow update - 2026-09-09T15:25:28+00:00
- Validation: castor test --filter='LlmHttpRetryPolicyTest|LlmProviderErrorClassifierTest|ExecuteLlmStepWorkerTest|LlmWorkerFailedEventSubscriberTest': 43 tests, 161 assertions passed. Deterministic MockHttpClient proves HTML/JSON recovery and exhaustion; live provider cannot deterministically produce these 400s.
- Summary: Replaced Messenger change with HTTP policy change at 3a9b1b56b. Reviewer agent_a2eab50d318dd8cb approved with optional suggestions, no blockers. Reviewed specification fidelity and exhaustion path; HTTP 400 remains terminal after HttpClient budget exhaustion. Testing prerequisites explicitly confirmed by reviewer.
- Ownership: owner=main; fork_run=none; revision=60164d3ce; scope=replace Messenger classification change with HTTP retry policy and recovery/exhaustion tests; outcome=completed; commit=3a9b1b56b
- Review: role=reviewer; artifact=agent_a2eab50d318dd8cb; revision=3a9b1b56b; scope=net diff, specification fidelity, HTTP exhaustion; verdict=APPROVE WITH SUGGESTIONS; blockers=none

## Task workflow update - 2026-09-09T15:28:06+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (133.5s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget/var/reports/qa-20260909-152553-2718-ea9bca94.
- Session/run: 28.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-09T15:28:08+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget/var/reports/qa-20260909-152553-2718-ea9bca94.
- Session/run: 28.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-09T15:28:11+00:00
- castor check passed (133.5s).
- Pushed task/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget to origin.
- Created PR: <url>
- Session/run: 28.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-09T15:28:11+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (133.5s).
- Pushed task/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/485

## Task workflow update - 2026-09-09T16:05:48+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget: ide_close_project returned isError.
- Merged task/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget into integration checkout.
- Merge made by the 'ort' strategy.
 src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpRetryPolicy.php       |  2 +-
 tests/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpRetryPolicyTest.php | 30 ++++++++++++++++++++++++++++--
 2 files changed, 29 insertions(+), 3 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-09-retry-http-400-llm-provider-failures-within-the-existing-retry-budget.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: Confirmed PR #485 merged on GitHub at 83ba39f559ad86f5919d35de2d106ecb0ddf6e63.

## Task workflow update - 2026-09-09T16:07:02+00:00
- Validation: Post-merge castor check passed in integration checkout at 24280d4444db7fee2f0a3d69d86f9977c820d1d0 (127s). All 10 lanes passed, leak and cache guards passed. Reports: var/reports/qa-20260909-160554-7895-cd3a4743.; Integration git status clean; task worktree confirmed removed.
- Summary: DONE with successful post-merge validation. Transition reported degraded JetBrains project close, but worktree removal succeeded.

## Task workflow update - 2026-09-10T22:49:42+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
