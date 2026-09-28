# Fix cached Codex continuation for zero-argument function calls

## Goal
User-reported session 64 on 2026-09-25 resumed under GPT-6 Sol and failed with prefix_mismatch after a completed five-call response. Structured log: first_mismatch_index=73, function_call vs function_call, arguments left=list and right=other, IDs equal, pending outputs all in delta. Canonical llm_step_completed seq 537 has first call task_list with empty arguments list. CodexToolCallNormalizer JSON-encodes empty ToolCall arguments as [] although the provider's object-valued arguments normalize to a stdClass; the comparator must fail because the replay wire shape changed. Investigate and fix at the owning conversion boundary. No full-history fallback, no bypass, no raw session content in logs or task records. Do not change the transport default. Explain how Harbor coverage missed this shape without asserting tasks never exercised zero-arg calls.

## Acceptance criteria
- Deterministic contract regression proves a provider function_call with empty object arguments survives stream conversion, canonical history serialization, same-model replay, and sends a nonempty function_call_output delta with previous_response_id; changed arguments still fail loud.
- Zero-argument tool calls serialize as a JSON object in the Codex request, not a JSON list; nonempty args and real arrays retain correct semantics.
- Focused Castor tests/static checks pass, independent review, and task-to-pr gate when requested. Do not run a paid provider benchmark without user approval.

## Workflow metadata
Status: DONE
Branch: task/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/528
PR Status: merged
Started: 2026-09-25T02:41:05+00:00
Completed: 2026-09-25T03:00:11+00:00

## Work log
- Created: 2026-09-25T02:40:46+00:00

## Task workflow update - 2026-09-25T02:41:05+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls/.idea.
- Validation: Inspected session 64 canonical event seq 537 (five calls, first task_list arguments empty list), structured mismatch event at 2026-09-25T02:32:21Z (index 73, response item offset 1, field arguments, list vs other, IDs equal), ResultConverter, CodexToolCallNormalizer, comparator, and existing tests.
- Summary: Session 64 read-only evidence isolates zero-argument task_list function_call mismatch after cached response: function_call IDs match and only arguments shape differs, list vs object. Main owns a bounded Codex normalizer/continuation regression fix; no live provider call. Keep fail-loud guard and unchanged default.

## Task workflow update - 2026-09-25T02:41:14+00:00
- Ownership: owner=main; fork_run=none; revision=agent-core main@e54867e5e; scope=Recover zero-argument Codex function-call object shape at same-model replay and prove cached delta without weakening mismatch guard; outcome=assigned; commit=none

## Task workflow update - 2026-09-25T02:48:24+00:00
- Validation: New end-to-end canonical history + Codex contract + continuation regression failed RED at '{}'(expected) vs '[]'(actual), then PASS after fix. 135 focused tests/738 assertions pass. Scoped castor phpstan Platform 0 errors; castor deptrac 0 violations; castor dead-code 0 errors; castor cs-check 0 changes; git diff --check clean. Stream conversion test covers output_item.done for zero-argument tool; no paid Codex trial.; Worktree clean at task@35c0b069e.
- Summary: Confirmed session 64 failure was first resumed response function_call task_list with empty arguments. Provider response's empty object was collapsed by ResultConverter's array-based ToolCall representation; CodexToolCallNormalizer re-encoded [] instead of {}. This changed replayed function-call arguments and triggered fail-loud prefix_mismatch at response offset 1. Commit 35c0b069e serializes empty ToolCall arguments as {} for Codex while retaining nonempty argument behavior; no comparator bypass, fallback, transport default change, or paid request. No PR yet; next phase task-to-pr.
- Ownership: owner=main; fork_run=none; revision=agent-core task@35c0b069e; scope=Recover zero-argument Codex function-call object shape at same-model replay and prove cached delta without weakening mismatch guard; outcome=completed; commit=35c0b069e

## Task workflow update - 2026-09-25T02:50:07+00:00
- Summary: Task-to-pr review requested on committed revision 35c0b069e; worktree clean and main aligned with branch base 1a1290e1a.
- Review: role=reviewer; artifact=agent_7a487490efab233e; revision=agent-core task@35c0b069e; scope=Specification-fidelity and correctness of zero-argument Codex call replay fix, comparator safety, and regression proof; outcome=assigned

## Task workflow update - 2026-09-25T02:50:34+00:00
- Review: role=reviewer; artifact=agent_7a487490efab233e; revision=agent-core task@35c0b069e; scope=Resume previous Cached Codex reviewer; outcome=blocked; reason=artifact belongs to previous parent lifetime after /resume. Assigning fresh reviewer.

## Task workflow update - 2026-09-25T02:56:23+00:00
- Validation: Reviewer's focused Castor 2 tests/16 assertions; implementation focused Castor 135 tests/738 assertions; phpstan Platform 0, deptrac 0, dead-code 0, cs-check clean; castor test:llm-real --filter=LlamaCppSmokeTest 1 test/8 assertions pass; worktree clean. Full castor check owned by CODE-REVIEW transition.
- Summary: Independent reviewer agent_9beb29b0d5edbba1 approved committed revision 35c0b069e with no blocking findings; no source changes since focused validation. Ready for CODE-REVIEW transition gate and PR creation; no paid Codex calls.
- Review: role=reviewer; artifact=agent_9beb29b0d5edbba1; revision=agent-core task@35c0b069e; scope=Codex zero-argument replay shape, canonical history and cached continuation safety; decision=APPROVE; blockers=none.

## Task workflow update - 2026-09-25T02:57:40+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (63.2s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls/var/reports/qa-20260925-025637-3072-d12fdaa0.
- Session/run: 65.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-25T02:57:42+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls/var/reports/qa-20260925-025637-3072-d12fdaa0.
- Session/run: 65.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-25T02:57:44+00:00
- castor check passed (63.2s).
- Pushed task/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls to origin.
- Created PR: <url>
- Session/run: 65.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-25T02:57:44+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (63.2s).
- Pushed task/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/528
- Validation: Focused Castor 135 tests/738 assertions pass; scoped phpstan, deptrac, dead-code, cs-check pass; local LlamaCppSmokeTest 1 test/8 assertions pass; full castor check via task transition.
- Summary: Commit 35c0b069e fixes zero-argument Codex function_call replay by encoding empty typed ToolCall arguments as JSON {} instead of []. Focused stream ingestion and full canonical-history continuation regressions pass; genuine changes still reject. Independent reviewer approved with no blockers; no paid Codex request.

## Task workflow update - 2026-09-25T02:58:15+00:00
- Validation: castor check passed 63.2s on 35c0b069e; worktree clean and pushed; GitHub PR #528 open, no GitHub review decision.
- Summary: PR #528 created at 35c0b069e after full Castor check passed (63.2s). Independent code reviewer approved. GitHub currently shows state=OPEN, reviewDecision empty, mergeStateStatus=CLEAN, so task-done procedure requires GitHub approval or merged state before DONE transition; awaiting GitHub review/merge, no manual branch/status moves.

## Task workflow update - 2026-09-25T03:00:11+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls into integration checkout.
- Merge made by the 'ort' strategy.
 src/Platform/Bridge/OpenAICodex/Contract/CodexToolCallNormalizer.php |  6 +++++-
 tests/AgentCore/Infrastructure/SymfonyAi/PlatformIntegrationTest.php | 47 +++++++++++++++++++++++++++++++++++++++++++++++
 tests/Platform/Bridge/OpenAICodex/ResultConverterTest.php            | 26 ++++++++++++++++++++++++++
 3 files changed, 78 insertions(+), 1 deletion(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-25-fix-cached-codex-continuation-for-zero-argument-function-calls.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub state MERGED at 2026-09-25T02:59:41Z; pre-merge castor check passed on 35c0b069e.
- Summary: PR #528 merged on GitHub at 599af7f42aee136d825cbc837874a7149bb662c0. Integrate branch and remove task worktree; run post-merge castor check on integration checkout next.

## Task workflow update - 2026-09-25T03:01:54+00:00
- Validation: castor check on integrated main dae0583cd: 5075 unit/integration tests, 22801 assertions; 13 controller-replay, 6 TUI replay, 5 local llm-real; deptrac/phpstan/lsp:check/dead-code/cs-check/docs:validate/catalog:version-check all pass; QA run qa-20260925-030035-7836-80626203 leak check clean; llama-proxy cache entries unchanged 406→406; git worktree clean and task worktree removed.
- Summary: PR #528 merged on GitHub (merge commit 599af7f42). DONE transition integrated changes locally and removed task worktree. Post-merge full castor check passed in integration main at dae0583cd; working tree clean. Local main has two merge commits ahead of origin/main produced by DONE integration (GitHub already contains PR content); no push performed.
