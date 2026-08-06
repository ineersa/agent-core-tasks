# Fix flaky llm-real SubagentRetrieveLiveE2E retrieve-chain failure

## Goal
Post-merge validation for RENDER-05 initially failed in `LLM_MODE=true castor check` only on the live LLM lane:

`SubagentRetrieveLiveE2ETest::testSubagentThenAgentRetrieveChain` failed with `agent_retrieve must start`.

Context:
- QA run `qa-20260701-000245-448276-6c2d3891` failed `test:llm-real` while all other lanes passed: unit/integration, controller replay, TUI replay, deptrac, phpstan, cs-check, cache guard, artifact integrity, leak check.
- Immediate focused rerun `castor test:llm-real` passed.
- Final full `LLM_MODE=true castor check` passed (`qa-20260701-000638-456423-5e586c54`).

We should not leave this as an unexplained transient. Investigate and fix the root cause or harden the test so the live smoke reliably proves the intended behavior.

Initial hypotheses to test:
- llama-proxy cassette/cache key collision or stale cached response for `[llm-real:subagent-retrieve-chain-v1]` under normalized prologue.
- Model prompt/compliance weakness: test asks for subagent then agent_retrieve, but the small model sometimes stops after the first step.
- Timing/race in parent-child artifact availability: parent may not see artifact/retrieve metadata before deciding final answer.
- ParaTest/full `castor check` load affects live test timing differently than focused sequential/parallel rerun.
- Test assertion waits for `agent_retrieve` start in a way that can miss valid late events or stops too early on `run.completed`.

Required constraints:
- Load testing skill and read tests/AGENTS.md before investigation/fix.
- Use Castor only for tests/QA.
- Use live reproduction evidence over replay if they disagree.
- Do not paper over by increasing broad timeouts without identifying root cause.
- If code changes are needed, implement in a fork/worktree, not directly in main.

## Acceptance criteria
- Root cause is documented with evidence from failing artifacts, test code, and at least one targeted reproduction attempt.
- Fix or hardening is implemented so `SubagentRetrieveLiveE2ETest::testSubagentThenAgentRetrieveChain` reliably starts `agent_retrieve` under the intended scenario.
- Focused validation passes: `castor test:llm-real --filter=SubagentRetrieveLiveE2ETest` and full `castor test:llm-real`.
- Full `LLM_MODE=true castor check` passes, including cache guard stability.
- If the issue is external llama-proxy cassette corruption/staleness, document exact cache key/reset/warm procedure and update the test/runbook if needed.

## Workflow metadata
Status: DONE
Branch: task/fix-llm-real-subagent-retrieve-chain-flake
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-llm-real-subagent-retrieve-chain-flake
Fork run: 2ab6tjz52l4j
PR URL: https://github.com/ineersa/agent-core/pull/247
PR Status: merged
Started: 2026-07-01T00:12:39.708Z
Completed: 2026-07-01T00:35:08.745Z

## Work log
- Created: 2026-07-01T00:10:57.202Z

## Task workflow update - 2026-07-01T00:12:39.708Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-llm-real-subagent-retrieve-chain-flake.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-llm-real-subagent-retrieve-chain-flake.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-llm-real-subagent-retrieve-chain-flake.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-llm-real-subagent-retrieve-chain-flake.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-llm-real-subagent-retrieve-chain-flake.
- Validation: Pre-start context: post-merge RENDER-05 initial castor check failed only in SubagentRetrieveLiveE2ETest::testSubagentThenAgentRetrieveChain; immediate focused rerun and full rerun passed.
- Summary: User approved fixing the post-merge llm-real SubagentRetrieveLiveE2E retrieve-chain flake. Starting investigation/implementation task.

## Task workflow update - 2026-07-01T00:27:56.676Z
- Recorded fork run: 2ab6tjz52l4j
- Validation: Fork focused validation: castor test:llm-real --filter=SubagentRetrieveLiveE2eTest::testSubagentThenAgentRetrieveChain — OK multiple times including 3 consecutive post-commit runs; Fork full castor test:llm-real — FAILED in unrelated ControllerSmokeTest::testControllerSpawnAndCompleteRun with assistant.message_failed; fork reports not introduced by this change; Fork did not run full LLM_MODE=true castor check; parent to validate
- Summary: Implementation fork completed at commit 8b2abaea1 (test(llm-real): harden subagent retrieve chain prompt). Root cause identified as live model compliance conflict: production guidance says agent_retrieve is optional/redundant after successful single-mode subagent, while old test step-2 prompt weakly requested retrieval; failing artifact showed assistant text + run.completed and no retrieve-phase tool_execution events. Fix is test-only: stronger step-2 prompt explicitly marks retrieval verification incomplete until agent_retrieve is called, acknowledges optional guidance but requires retrieve for this test, forbids assistant text/other tools, and adds clearer failure diagnostic for text-only completion.

## Task workflow update - 2026-07-01T00:32:05.359Z
- Validation: Reviewer: APPROVE WITH SUGGESTIONS; no blocking findings; Parent validation: castor test:llm-real --filter=SubagentRetrieveLiveE2eTest::testSubagentThenAgentRetrieveChain — OK (1 test, 23 assertions); Parent validation: castor test:llm-real — OK (9 tests, 110 assertions); Parent validation: castor phpstan --path=tests/CodingAgent/Runtime/Controller/E2E/SubagentRetrieveLiveE2eTest.php — OK (errors=0, file_errors=0); Parent validation: castor cs-check — OK (files_fixed=0); Llama-proxy cache stats before CODE-REVIEW gate: entries=18, bytes=673102
- Summary: Parent validation/review: worktree clean at commit 8b2abaea1. Reviewer returned APPROVE WITH SUGGESTIONS, verifying root-cause evidence and test-only prompt hardening; no production semantics weakened and no blocking issues. Parent focused/full validation passed after fork.

## Task workflow update - 2026-07-01T00:33:31.643Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (73.2s).
- Pushed task/fix-llm-real-subagent-retrieve-chain-flake to origin.
- branch 'task/fix-llm-real-subagent-retrieve-chain-flake' set up to track 'origin/task/fix-llm-real-subagent-retrieve-chain-flake'.
- Created PR: https://github.com/ineersa/agent-core/pull/247

## Task workflow update - 2026-07-01T00:35:08.745Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-llm-real-subagent-retrieve-chain-flake into integration checkout.
- Merge made by the 'ort' strategy.
 .../Controller/E2E/SubagentRetrieveLiveE2eTest.php | 30 +++++++++++++++++++---
 1 file changed, 27 insertions(+), 3 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-llm-real-subagent-retrieve-chain-flake.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-llm-real-subagent-retrieve-chain-flake.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Pre-DONE: gh pr view 247 — state=MERGED, mergeCommit=712e3f5cc252de2ef2655d49baa212d0a71dc3a3, url=https://github.com/ineersa/agent-core/pull/247; Pre-DONE: integration checkout clean and aligned with origin/main
- Summary: PR #247 was merged on GitHub (mergedAt 2026-07-01T00:34:52Z). Proceeding to DONE merge/sync and post-merge validation.

## Task workflow update - 2026-07-01T00:36:36.889Z
- Validation: move_task DONE — moved CODE-REVIEW → DONE, merged task/fix-llm-real-subagent-retrieve-chain-flake, removed worktree and IDEA exclusions, pulled integration checkout; Post-merge LLM_MODE=true castor check qa-20260701-003511-489509-420cafab — OK quality in 211.7s: deptrac OK (1.0s), test OK (3985 tests, 12842 assertions), test:controller-replay OK (8 tests, 112 assertions), test:tui OK (23 tests, 114 assertions), test:llm-real OK (10 tests, 121 assertions), phpstan OK, cs-check OK, llama-proxy cache guard OK (23→23), QA artifact integrity OK, leak check OK; Cleanup: /home/ineersa/projects/agent-core-worktrees/fix-llm-real-subagent-retrieve-chain-flake removed; Integration checkout: git status clean, main...origin/main ahead 2
- Summary: Post-merge validation complete. PR #247 was already merged on GitHub; move_task DONE merged task branch into integration checkout, removed worktree and IDEA exclusions, and pulled/synced. Full post-merge LLM_MODE=true castor check passed. Worktree directory removed. Integration checkout working tree is clean but local main is ahead of origin/main by 2 local merge commits produced by move_task/pull after the GitHub PR merge; leaving as-is for user-directed push/pull policy.
