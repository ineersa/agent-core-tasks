# Cap child-agent resume context at 200k tokens

## Goal
The current resume guard in src/CodingAgent/Agent/Execution/AgentResumeExecutionService.php::assertContextBudgetAllowsResume() calculates max(floor(0.75 * contextWindow), 200_000). This makes 200k a floor instead of the intended ceiling.

Observed in fork agent_93b40a46c0f74331, run 36e3e2c0-a680-5ea8-b2e4-8aae43f6f3fa: recorded context window 1,050,000 produced a 787,500-token cutoff, allowing resume at 256,638 latest input tokens. The resumed run finished at 316,132.

User explicitly requested replacing max with min. For a known positive context window, reject resume at or above min(floor(0.75 * contextWindow), 200_000). Retain the 200,000 threshold when the window is unknown. Update misleading floor naming, affected tests, and docs/agents.md, which currently documents max. Keep scope limited to resume eligibility; do not change compaction or active-run limits.

## Acceptance criteria
- Known positive context windows use min(floor(0.75 * contextWindow), 200_000); unknown windows retain the 200,000-token cutoff.
- Resume is rejected at or above the threshold. Regression coverage includes a large window capped at 200k, a small window limited to 75%, unknown-window handling, and below-threshold eligibility.
- Documentation and affected names describe a ceiling rather than a floor. Focused validation runs through Castor.

## Workflow metadata
Status: DONE
Branch: task/2026-09-22-cap-child-agent-resume-context-at-200k-tokens
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-22-cap-child-agent-resume-context-at-200k-tokens
Fork run: none
PR URL: https://github.com/ineersa/agent-core/pull/525
PR Status: merged
Started: 2026-09-22T16:00:37+00:00
Completed: 2026-09-22T16:28:05+00:00

## Work log
- Created: 2026-09-22T15:51:46+00:00

## Task workflow update - 2026-09-22T16:00:37+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-22-cap-child-agent-resume-context-at-200k-tokens.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-22-cap-child-agent-resume-context-at-200k-tokens.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-22-cap-child-agent-resume-context-at-200k-tokens.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-22-cap-child-agent-resume-context-at-200k-tokens.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-22-cap-child-agent-resume-context-at-200k-tokens.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-22-cap-child-agent-resume-context-at-200k-tokens/.idea.

## Task workflow update - 2026-09-22T16:02:52+00:00
- Recorded fork run: none
- Summary: Ownership: owner=main; fork_run=none; revision=task/2026-09-22-cap-child-agent-resume-context-at-200k-tokens@baseline; scope=Cap resume-context guard at min(floor(0.75*window),200k), rename floor constant, update AgentResumeExecutionServiceTest coverage, fix docs/agents.md line; outcome=assigned; commit=none

## Task workflow update - 2026-09-22T16:05:58+00:00
- Validation: castor test --filter=AgentResumeExecutionServiceTest (task worktree): OK — 17 tests, 76 assertions, 1.2s; castor phpstan --path=src/CodingAgent/Agent/Execution/AgentResumeExecutionService.php: errors=0, file_errors=0; castor phpstan --path=tests/.../AgentResumeExecutionServiceTest.php: 45 ignorable findings, all pre-existing class (baseline pristine file shows 44 of the same identifiers; +1 is the new assertSame line)
- Summary: Ownership: owner=main; fork_run=none; revision=task/2026-09-22-cap-child-agent-resume-context-at-200k-tokens@4763019b9; scope=Cap resume-context guard at min(floor(0.75*window),200k), rename floor constant, update AgentResumeExecutionServiceTest coverage, fix docs/agents.md line; outcome=completed; commit=4763019b9

## Task workflow update - 2026-09-22T16:10:10+00:00
- Validation: castor test --filter=AgentResumeExecutionServiceTest after NTH fix: OK — 17 tests, 76 assertions, 1.2s; castor phpstan --path=<service file> (implementation revision): errors=0; castor test:llm-real: not applicable — no provider/schema/prompt/streaming/routing change
- Summary: Review: role=reviewer; artifact=agent_f5ec63637b62edea; revision=4763019b9; scope=spec-fidelity+correctness+test-quality review of resume-cap commit (3 files). Verdict=APPROVE WITH SUGGESTIONS (no CRITICAL/BUG/SEC). NTH 1 (pin threshold 200000 in unknown-window test) applied in 66c7a8c87; NTH 2 (docs floor() prose) declined, matches existing doc style. Ownership: owner=main; fork_run=none; revision=task/2026-09-22-cap-child-agent-resume-context-at-200k-tokens@66c7a8c87; scope=apply review NTH assertion tightening; outcome=completed; commit=66c7a8c87. No unresolved blockers.

## Task workflow update - 2026-09-22T16:12:35+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (131.3s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-22-cap-child-agent-resume-context-at-200k-tokens/var/reports/qa-20260922-161024-1895-1ea79c1b.
- Session/run: 66.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-22T16:12:37+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-22-cap-child-agent-resume-context-at-200k-tokens to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-22-cap-child-agent-resume-context-at-200k-tokens/var/reports/qa-20260922-161024-1895-1ea79c1b.
- Session/run: 66.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-22T16:12:41+00:00
- castor check passed (131.3s).
- Pushed task/2026-09-22-cap-child-agent-resume-context-at-200k-tokens to origin.
- Created PR: <url>
- Session/run: 66.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-22T16:12:41+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (131.3s).
- Pushed task/2026-09-22-cap-child-agent-resume-context-at-200k-tokens to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/525

## Task workflow update - 2026-09-22T16:28:05+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-22-cap-child-agent-resume-context-at-200k-tokens into integration checkout.
- Merge made by the 'ort' strategy.
 docs/agents.md                                                        |  2 +-
 src/CodingAgent/Agent/Execution/AgentResumeExecutionService.php       |  6 +++---
 tests/CodingAgent/Agent/Execution/AgentResumeExecutionServiceTest.php | 53 ++++++++++++++++++++++++++++++++++++++++++++++-------
 3 files changed, 50 insertions(+), 11 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-22-cap-child-agent-resume-context-at-200k-tokens.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-22-cap-child-agent-resume-context-at-200k-tokens.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-09-22T16:29:34+00:00
- Updated PR Status: merged
- Validation: Post-merge integration castor check: quality ok (130.7s) — 11 lane logs OK, QA leak check ok, cache cleanup ok, llama-proxy cache guard ok (entries 405 → 405); git status clean in /home/ineersa/projects/agent-core; HEAD 64996b268 includes task merge 84eb312c5 and PR merge 00a7ee4d6; Task worktree removed
- Summary: Task completed. PR #525 merged (GitHub merge commit 00a7ee4d, 2026-09-22T16:26:18Z); DONE transition merged task branch into integration checkout (84eb312c5, pull merge 64996b268), worktree removed. Post-merge castor check in integration checkout: quality ok (130.7s), all lanes OK, leak check ok, llama-proxy cache guard ok (405→405). Integration checkout clean.
