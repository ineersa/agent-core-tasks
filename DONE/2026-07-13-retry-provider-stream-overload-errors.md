# Retry provider-level server overload stream errors

## Goal
Hatfield session 5 exposed a pre-existing retry-classification gap. Third scout child `agent_be1d6629bc1a29ca` / run `019f5c81-1e39-7f9f-a362-b910819c8bbd` failed immediately on Codex with `[server_is_overloaded/service_unavailable_error]`.

Evidence:
- `.hatfield/sessions/5/artifacts/agents/agent_be1d6629bc1a29ca/events.jsonl` records `llm_step_failed` with `retryable=false`, `retry_attempt=0`, `max_retries=2`.
- `.hatfield/logs/agent-2026-07-13.log` around 17:23:15 records `llm.provider.stream_error` followed by `llm.request.failed` with category provider and retryable=false.
- `LlmProviderErrorClassifier::classifyByMessagePattern()` recognizes transport and terminal billing patterns but not structured server-overload/service-unavailable provider codes.
- `LlmHttpRetryPolicy` already treats overload/service-unavailable responses as retryable, but it does not apply when the transport succeeds and the provider emits a structured error event inside the stream.

This is not caused by PR #285; it exists on main and is exposed by Codex WebSocket/SSE structured provider errors. Keep the fix generic and privacy-safe. Prefer exact/bounded structured overload codes over broad arbitrary provider-message matching.

## Acceptance criteria
- Provider-level `server_is_overloaded` and `service_unavailable_error` signals without an HTTP status are classified as retryable server errors.
- `LlmStepResultHandler` consequently schedules bounded automatic retries up to configured `max_retries`; terminal provider/model/auth/billing errors remain non-retryable.
- Classification remains privacy-safe: do not log or propagate arbitrary raw provider response content; match allowlisted structured codes/types or tightly bounded normalized signals.
- Add a focused regression test reproducing `[server_is_overloaded/service_unavailable_error]` with no HTTP status and proving retryable classification.
- Audit overlap with `LlmHttpRetryPolicy` and centralize/share semantics where architecture permits without coupling generic classification to Codex.
- Run Castor-only focused tests plus `castor deptrac`, `castor phpstan`, and `castor cs-check`.

## Workflow metadata
Status: DONE
Branch: task/2026-07-13-retry-provider-stream-overload-errors
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-13-retry-provider-stream-overload-errors
Fork run: l24g7kojj41w
PR URL: https://github.com/ineersa/agent-core/pull/348
PR Status: merged
Started: 2026-08-01T00:12:08.805Z
Completed: 2026-08-01T01:25:07.068Z

## Work log
- Created: 2026-07-13T17:35:05.469Z

## Task workflow update - 2026-08-01T00:12:08.805Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-07-13-retry-provider-stream-overload-errors.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-07-13-retry-provider-stream-overload-errors.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-07-13-retry-provider-stream-overload-errors.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-07-13-retry-provider-stream-overload-errors.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-07-13-retry-provider-stream-overload-errors.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-07-13-retry-provider-stream-overload-errors.
- Summary: Implementation scope finalized from task-explain: add exact allowlisted no-status stream overload classification in LlmProviderErrorClassifier; preserve terminal auth/model/billing behavior; reuse existing bounded LlmStepResultHandler retry scheduling; keep HTTP and stream retry layers separate; do not broaden to all ServerException instances.

## Task workflow update - 2026-08-01T00:12:33.988Z
- Recorded fork run: l24g7kojj41w
- Implementation fork launched with finalized minimal scope: exact structured no-status overload allowlist in LlmProviderErrorClassifier, existing bounded retry pipeline proof, observed Codex converter fixture, no HTTP-policy coupling or broad ServerException behavior.

## Task workflow update - 2026-08-01T00:17:33.333Z
- Recorded fork run: l24g7kojj41w
- Validation: Fork confirmed it read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before test work and used Castor only.; `castor test --filter='LlmProviderErrorClassifierTest|LlmStepResultHandlerTest|ResultConverterTest'` — OK (124 tests, 470 assertions).; `castor deptrac` — 0 violations, 0 errors.; `castor phpstan` — 0 errors.; `castor cs-check` — clean (0 files fixed).
- Summary: Implementation complete in commit dc4765b6af47fa5e993a804f878e0efbd905a450. Added exact allowlisted structured server overload/service-unavailable classification for no-status provider stream errors, preserved terminal classification precedence and privacy-safe messaging, reused existing bounded LlmStepResultHandler retry scheduling, and aligned focused classifier/handler/Codex converter regression tests. Worktree is clean; 4 expected files changed (+112/-14).
- Parent verified commit dc4765b6a exists on task/2026-07-13-retry-provider-stream-overload-errors, worktree is clean, and only the classifier plus three planned existing test files changed. Task-start intentionally stopped before review, live LLM validation, castor check, push, or PR creation.

## Task workflow update - 2026-08-01T00:50:17.480Z
- Validation: Reviewer: APPROVED WITH SUGGESTIONS; no critical, blocking, security, privacy, architecture, or specification-fidelity issues.; `castor test` — OK (4405 tests, 16274 assertions).; `castor test:llm-real` — OK (13 tests, 144 assertions); llama.cpp generation preflight OK.; `castor deptrac` — 0 violations, 0 errors.; `castor phpstan` — 0 errors.; `castor cs-check` — clean (0 files fixed).; Worktree clean at dc4765b6af47fa5e993a804f878e0efbd905a450 before CODE-REVIEW transition.
- Summary: Task-to-PR review complete at dc4765b6af47fa5e993a804f878e0efbd905a450. Reviewer approved: exact allowlisted no-status stream signals classify as retryable server errors; privacy, precedence, bounded retry reuse, architecture separation, test value, and specification fidelity all pass. No blocking findings. Non-blocking suggestion only: optional direct response_error_code/type test; skipped because observed bracket path and three required focused regression proofs already cover finalized scope/test budget.

## Task workflow update - 2026-08-01T00:52:13.606Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (105.5s).
- Pushed task/2026-07-13-retry-provider-stream-overload-errors to origin.
- branch 'task/2026-07-13-retry-provider-stream-overload-errors' set up to track 'origin/task/2026-07-13-retry-provider-stream-overload-errors'.
- Created PR: https://github.com/ineersa/agent-core/pull/348

## Task workflow update - 2026-08-01T01:25:07.068Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-07-13-retry-provider-stream-overload-errors into integration checkout.
- Merge made by the 'ort' strategy.
 .../SymfonyAi/LlmProviderErrorClassifier.php       | 59 +++++++++++++++++++++-
 .../Pipeline/LlmStepResultHandlerTest.php          | 21 +++++---
 .../SymfonyAi/LlmProviderErrorClassifierTest.php   | 23 +++++++++
 .../Bridge/OpenAICodex/ResultConverterTest.php     | 23 ++++++---
 4 files changed, 112 insertions(+), 14 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-07-13-retry-provider-stream-overload-errors.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-07-13-retry-provider-stream-overload-errors.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #348 merged on GitHub at 2026-08-01T01:24:47Z with merge commit 7ac802699babc946efbea22212358be7be062141. Completing task workflow and syncing integration checkout.

## Task workflow update - 2026-08-01T01:27:11.600Z
- Validation: Post-merge `LLM_MODE=true castor check` — quality OK.; Unit/integration: 4405 tests, 16274 assertions.; Controller replay: 11 tests, 160 assertions.; TUI replay: 30 tests, 183 assertions.; Live LLM: 13 tests, 144 assertions.; Deptrac, PHPStan, and CS checks passed.; llama-proxy cache guard stable at 221 entries; QA artifact integrity and leak check passed.; Integration checkout clean; task worktree removed.
- Summary: Task completed after merged PR #348. Integration checkout synced and clean; task worktree and IDEA exclusions removed.
