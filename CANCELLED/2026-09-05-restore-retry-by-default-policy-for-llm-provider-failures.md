# Restore retry-by-default policy for LLM provider failures

## Goal
User reports fork termination on Grok proxy HTTP 502 and confirms intended policy: all errors are retryable except explicitly selected known non-retryable errors. HTTP transport retry exhaustion must not implicitly make transient provider errors terminal at the LLM-step layer.

Observed failures: child runs 900968cd-d601-560e-9e45-879f2b868bab and 5b5d565c-3672-5572-9905-fbdaf1e70737 on 2026-09-05. Both received HTTP 502 and were classified retryable:false. Evidence: .hatfield/logs/agent-2026-09-05.log around lines 186400–186430; first run events 235–241. Provider logs do not establish the internal HTTP retry count.

Investigation entry points:
- src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpRetryPolicy.php: HTTP retry strategy includes 502.
- SymfonyAiProviderFactory: RetryableHttpClient wiring.
- LlmProviderErrorClassifier: HTTP status classification maps server errors to terminal failure with misleading retry-exhaustion wording.
- LlmPlatformAdapter: status extraction and diagnostic mapping.
- ExecuteLlmStepWorker: classified retryable flag controls whole-step retry.

Restore retry-by-default classification. Inspect existing policy and retain only explicitly justified known terminal exceptions. Keep retries bounded by existing budgets; do not introduce infinite retries or new settings. Distinguish HTTP-attempt retries from whole-step retryability in diagnostics.

## Acceptance criteria
- HTTP 502 and other transient provider failures remain eligible for bounded LLM-step retries after HTTP transport retries are exhausted.
- Unknown/unclassified provider errors default to retryable; non-retryable outcomes use an explicit, justified allowlist consistent with project policy.
- Existing retry budgets and cancellation semantics remain enforced; no infinite retries or speculative settings.
- Diagnostics distinguish HTTP transport attempts from whole-step retry decisions and do not claim observed exhaustion without evidence.
- Deterministic Castor tests cover transient HTTP errors, unknown failures, known terminal exceptions, and bounded worker retry/exhaustion behavior at the lowest correct layer.

## Workflow metadata
Status: CANCELLED
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-09-05T22:36:26+00:00

## Task workflow update - 2026-09-05T22:38:09+00:00
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Cancelled at user's request; no implementation performed.
