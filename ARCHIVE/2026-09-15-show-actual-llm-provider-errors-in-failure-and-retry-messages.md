# Show actual LLM provider errors in failure and retry messages

## Goal
Session 26's Grok forks failed with HTTP 402 and 'Grok Build usage balance exhausted', but the transcript displayed only 'LLM provider request failed after retries were exhausted.' User requests actual errors in the UI. Preserve provider diagnostics through classification and retry exhaustion into existing failure presentation. No retry-policy changes or unrelated duplicate-card redesign.

## Acceptance criteria
- Retry and final failure messages retain the actual provider error and available HTTP status rather than replacing it with category-only prose.
- Session 26 Grok 402 balance-exhausted regression is covered through existing error propagation and presentation.
- Use existing diagnostic sanitization facilities where applicable; do not expose credentials, request payloads or terminal control sequences. No new setting or API.

## Workflow metadata
Status: DONE
Branch: task/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/500
PR Status: merged
Started: 2026-09-15T01:24:59+00:00
Completed: 2026-09-15T17:35:10+00:00

## Work log
- Created: 2026-09-15T01:24:39+00:00

## Task workflow update - 2026-09-15T01:24:59+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages.

## Task workflow update - 2026-09-15T01:26:19+00:00
- Ownership: owner=main; fork_run=none; revision=f731e9a1e; scope=preserve provider diagnostics in classifier and retry exhaustion with propagation/rendering regression proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-15T01:34:03+00:00
- Validation: castor test --filter='LlmProviderErrorClassifierTest|LlmRequestRetryExecutorTest|TuiMountedTranscriptVirtualTest|ExecuteLlmStepWorkerTest|DeferredChildRunEventProjectorTest|RuntimeEventTranslator|LlmStepResultHandlerTest|TranscriptProjectorTest': 157 tests, 820 assertions passed; castor phpstan: 0 errors; castor deptrac: 0 violations, 0 errors; git diff --check: passed; castor cs-fix --path=<changed production classifier and virtual transcript test>: no changes; Skill defect: testing/SKILL.md advertises positional cs-fix [path], but Castor accepts --path. Used supported --path; no skill modification.
- Summary: Preserved actual bounded, redacted exception text and available HTTP status in user_message. Retry exhaustion now keeps that cause. Retry policy unchanged. Virtual mounted transcript regression proves Grok balance exhaustion remains visible after retries. Committed d9025c9a9, not pushed. Full gate and independent review remain for task-to-pr.
- Ownership: owner=main; fork_run=none; revision=f731e9a1e; scope=preserve provider diagnostics in classifier and retry exhaustion with propagation/rendering regression proof; outcome=completed; commit=d9025c9a9

## Task workflow update - 2026-09-15T13:57:11+00:00
- Review: role=reviewer; artifact=agent_651eba08d77c743c; revision=d9025c9a9; scope=specification fidelity and error visibility; verdict=REQUEST CHANGES for two stale E2E assertions.
- Ownership: owner=main; fork_run=none; revision=d9025c9a9; scope=update superseded controller and terminal error visibility assertions; outcome=assigned; commit=none

## Task workflow update - 2026-09-15T14:01:58+00:00
- Ownership: owner=main; fork_run=none; revision=d9025c9a9; scope=update superseded controller and terminal error visibility assertions; outcome=completed; commit=4348f16c3
- Review: role=reviewer; artifact=agent_651eba08d77c743c; revision=4348f16c3; scope=full specification fidelity plus E2E expectation corrections; verdict=APPROVE WITH SUGGESTIONS. No blockers. Existing focused validation reused for unchanged production code. No forks used.

## Task workflow update - 2026-09-15T14:04:43+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (132.9s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages/var/reports/qa-20260915-140230-1153-6e03b86f.
- Session/run: 52.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-15T14:05:25+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages/var/reports/qa-20260915-140230-1153-6e03b86f.
- Session/run: 52.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-15T14:05:59+00:00
- castor check passed (132.9s).
- Pushed task/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages to origin.
- Created PR: <url>
- Session/run: 52.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-15T14:05:59+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (132.9s).
- Pushed task/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/500
- Summary: Review approved revision 4348f16c3 after updating superseded E2E assertions. Actual provider causes survive retries and render in the transcript. Retry policy unchanged.

## Task workflow update - 2026-09-15T17:35:10+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages: ide_close_project returned isError.
- Merged task/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages into integration checkout.
- Merge made by the 'ort' strategy.
 src/AgentCore/Infrastructure/SymfonyAi/LlmProviderErrorClassifier.php                      | 11 +++++++++++
 src/AgentCore/Infrastructure/SymfonyAi/Retry/LlmRequestRetryExecutor.php                   |  9 +--------
 tests/AgentCore/Infrastructure/SymfonyAi/LlmProviderErrorClassifierTest.php                | 20 +++++++++++++++++---
 tests/AgentCore/Infrastructure/SymfonyAi/Retry/LlmRequestRetryExecutorTest.php             |  2 +-
 tests/CodingAgent/Runtime/Controller/E2E/ControllerReplayLlmRequestRetryVisibilityTest.php |  4 ++--
 tests/Tui/E2E/TuiProviderErrorE2eTest.php                                                  | 24 +++++++-----------------
 tests/Tui/Screen/TuiMountedTranscriptVirtualTest.php                                       | 39 +++++++++++++++++++++++++++++++++++++++
 7 files changed, 78 insertions(+), 31 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-15-show-actual-llm-provider-errors-in-failure-and-retry-messages.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: Confirmed PR #500 merged on GitHub at ab4c40628378b90bb11d5f7bc6fdb9560c88c774. Integrating and running post-merge validation.

## Task workflow update - 2026-09-15T17:38:07+00:00
- Validation: castor check: all 10 lanes passed; report var/reports/qa-20260915-173521-349-fdd0bcea; Unit 5001 tests; controller replay 11; TUI 9; live smoke 5. Leak and cache guards passed.; git status --short: clean; task worktree absent
- Summary: Post-merge validation passed on integration revision 6f0dc233ab801457b4f3734190ea360e3ae39eaa. Git status clean; task worktree removed. IDE close reported degradation during cleanup, but filesystem removal succeeded.
