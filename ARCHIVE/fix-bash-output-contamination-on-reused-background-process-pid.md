# Fix bash output contamination on reused background-process PID

## Goal
## Problem

Hatfield can return output from an earlier, unrelated bash command when the OS reuses a process PID. The tool call identity, Messenger routing, tool-batch correlation, and current process execution are correct; the contamination occurs when `BashTool` rereads the completed process log using a non-unique PID lookup.

This was discovered during manual validation of session `1` in the `single-writer-run-state-mutation-coordinator` worktree, but the defect is pre-existing and independent of the single-writer changes. Implement this from the normal integration base as a separate narrow task/PR.

## Exact forensic evidence

Session 1 turn 11 launched three parallel bash calls:

- `call_00_wCjFnHjxhPxvv3MRk9vk0226`: `sleep 10 && echo "Task 1 done at $(date +%H:%M:%S.%3N)"` — correct.
- `call_01_5rhh8HVymyhqoqvuJBbJ3161`: `sleep 10 && echo "Task 2 done at $(date +%H:%M:%S.%3N)"` — incorrectly returned Composer package information.
- `call_02_pKvIozcQjuut0fci7RkH8575`: `sleep 10 && echo "Task 3 done at $(date +%H:%M:%S.%3N)"` — correct.

For the wrong second call:

- Current process row: background-process DB record `id=27`, PID `97`.
- Current unique log: `.hatfield/tmp/bg/17f6205da2d967ef.log`.
- Current log contents were correct: `Task 2 done at 14:02:02.358`.
- Earlier same-session Composer command at canonical event seq 91 had background-process DB record `id=6`, also PID `97`.
- Earlier unique log: `.hatfield/tmp/bg/990dea12191063c9.log`.
- Earlier log contained the returned `composer info --direct` package listing.

The current tool call was therefore executed correctly. Only its final output lookup selected the stale log.

## Confirmed root cause

The safe supervision path already uses the immutable background-process database record ID:

1. `BackgroundProcessManager::start()` returns PID, immutable DB record ID, and unique log path.
2. `BashTool` polls the current command through `findByRecordId($dbId, $sessionId)`.
3. On completion, `BashTool::handleFinished()` discards that unambiguous identity and calls `readOutput($pid, $sessionId)`.
4. `readOutput()` calls `BackgroundProcessManager::readLogFull($pid, $sessionId)`.
5. `readLogFull()` calls `ProcessStore::fetchByPid($pid)`.
6. `fetchByPid()` uses `findOneBy(['pid' => $pid])`, even though its own documentation states that OS PIDs are reusable and this lookup is non-unique.
7. With two retained rows for PID 97, Doctrine returned old row id 6, so the old Composer log became the current `ToolCallResult`.

Relevant production files:

- `src/CodingAgent/Tool/BashTool.php`
- `src/CodingAgent/Tool/BackgroundProcessManager.php`
- `src/CodingAgent/Tool/BackgroundProcess/ProcessStore.php`

## Explicitly excluded causes

The forensic evidence excludes:

- Reviewer or Castor stdout entering the tool-result channel.
- Cross-session Messenger queue stealing.
- Tool-call ID or order-index mismatch.
- `ToolBatchCollector` result reassignment.
- `ToolExecutionResultStore` cache collision.
- Wrong shell command execution.

The stale Composer output came from an earlier command in the same Hatfield session that happened to have the same recycled OS PID.

## Required fix

1. The foreground bash completion path must keep and use an immutable process identity from start through final output retrieval.
   - Prefer the already resolved `BackgroundProcess` record, its immutable DB record ID, or its unique log path.
   - Do not re-resolve a completed foreground process by OS PID.
2. Preserve session ownership validation before reading the log.
3. Preserve full-output semantics so `OutputCapToolResultProcessor` remains the primary output-capping layer.
4. Do not "fix" the foreground path merely by adding `ORDER BY id DESC` to the PID lookup. Choosing the newest row is still weaker than using the exact record already tracked by the invocation.
5. Audit the remaining PID-only `BackgroundProcessManager` operations (`readLogTail`, `stop`, `markBackgrounded`, and related callers). Keep this PR narrow, but either make any directly affected internal call use immutable identity or explicitly document safe user-facing PID behavior and create a separate follow-up if changing the public `bg_status pid=` contract would broaden scope.
6. Do not add test-only production seams, compatibility fallback readers, or raw output in structured logs.

## Regression-test thesis

A completed foreground bash invocation must return the log belonging to its exact immutable background-process record even when an older retained record has the same PID.

The deterministic test must:

- Use the project test infrastructure and follow `.agents/skills/testing/SKILL.md` plus `tests/AGENTS.md`.
- Avoid waiting for real OS PID reuse and avoid sleeps.
- Create two distinguishable background-process records/logs with the same PID: an older `COMPOSER_MARKER` log and a current `DATETIME_MARKER` log.
- Exercise the real foreground completion/output-selection path at the lowest suitable layer.
- Prove the unfixed implementation returns/selects the stale log and the fixed implementation returns `DATETIME_MARKER` only.
- Assert session ownership remains enforced.
- Use Castor for all testing and QA; no raw `vendor/bin/*` commands.

## Scope boundaries

- No Messenger transport redesign.
- No tool-batch or run-state mutation changes.
- No TUI changes.
- No broad background-process persistence rewrite.
- No production path that exists solely for tests.

## Relationship to other work

This bug was found while manually validating `single-writer-run-state-mutation-coordinator`, but it did not originate there. Keep its implementation and PR separate so the single-writer change is not expanded by an unrelated bash/background-process fix.

## Acceptance criteria
- Foreground BashTool completion reads output through the exact immutable process record/record ID/log path and never re-resolves that invocation by PID.
- A deterministic regression test with two retained records sharing one PID proves the current invocation returns only its own log output.
- The test requires no sleeps or real PID-reuse timing and follows the kernel/test-container rules for DB-touching tests.
- Session ownership validation and full-output/output-cap behavior remain intact.
- Relevant PID-only background-process call sites are audited; any intentionally retained PID lookup is justified and cannot contaminate foreground tool results.
- No reviewer/Castor/Messenger/tool-batch workaround is introduced because those layers were forensically excluded.
- Focused Castor tests, castor phpstan, castor deptrac, and castor cs-check pass; full castor check passes before CODE-REVIEW.

## Workflow metadata
Status: DONE
Branch: task/fix-bash-output-contamination-on-reused-background-process-pid
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-bash-output-contamination-on-reused-background-process-pid
Fork run: g43pqw46zall
PR URL: https://github.com/ineersa/agent-core/pull/294
PR Status: merged
Started: 2026-07-16T16:23:40.814Z
Completed: 2026-07-16T20:20:27.387Z

## Work log
- Created: 2026-07-10T18:25:35.514Z

## Task workflow update - 2026-07-16T16:23:40.814Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-bash-output-contamination-on-reused-background-process-pid.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-bash-output-contamination-on-reused-background-process-pid.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-bash-output-contamination-on-reused-background-process-pid.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-bash-output-contamination-on-reused-background-process-pid.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-bash-output-contamination-on-reused-background-process-pid.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-bash-output-contamination-on-reused-background-process-pid.
- Summary: Claimed for implementation via task-start workflow.

## Task workflow update - 2026-07-16T16:28:14.554Z
- Summary: Prepared implementation context. Worktree: /home/ineersa/projects/agent-core-worktrees/fix-bash-output-contamination-on-reused-background-process-pid. Scouts confirmed BashTool already tracks StartResult::$id and polls by findByRecordId(), but final read, cancellation/timeout stop, and background marking drop back to PID-only manager APIs. BackgroundProcessCompletionPoller likewise converts an exact entity to PID for log/notified operations. Public BgStatusTool remains PID-addressed and needs an explicit, narrow semantic decision rather than replacing the foreground fix with newest-PID ordering. Regression should exercise real BashTool foreground completion without sleeps by seeding an older COMPOSER_MARKER record, updating its PID to the current PID inside the real background-prompt callback, releasing a file-blocked command that writes DATETIME_MARKER, then asserting exact-current output and session ownership.
- Two scouts inspected production call sites and test infrastructure in the task worktree. Both confirmed they read .agents/skills/testing/SKILL.md and tests/AGENTS.md and followed the project test conventions in their recommendations.

## Task workflow update - 2026-07-16T16:29:16.306Z
- Recorded fork run: hggm14o13iif
- Summary: Implementation fork launched in the task worktree with immutable-record production changes, deterministic real-BashTool regression, PID-only call-site audit, focused Castor validation, and commit required.

## Task workflow update - 2026-07-16T16:34:02.688Z
- Recorded fork run: hggm14o13iif
- Validation: castor test --filter='ForegroundCompletionReturnsCurrentRecordLogWhenPidIsReused|ReadLogFullForRecordEnforcesSessionOwnership' — passed (2 tests, 11 assertions); castor test --filter='BashToolTest|BackgroundProcessManager' — passed (47 tests); castor test --suite=coding-agent — passed (2598 tests); castor test --filter=BackgroundProcessCompletionPollerTest — passed (8 tests); castor phpstan — passed; castor deptrac — passed (0 violations); castor cs-check — passed after castor cs-fix; git show --check e3636c7d8 — passed; Full castor check intentionally not run during task-start; reserved for task-to-pr.
- Summary: Implementation completed and committed as e3636c7d8bf5cf5079df93a4c855d6eca62d5e35. Verified the commit exists, the task worktree is clean, and the expected 6 files changed. BashTool now preserves immutable background-process record identity for polling, cancellation/timeout stop, background marking, and full foreground output retrieval. Completion polling also keeps exact record identity. Public bg_status PID operations remain PID-addressed with deterministic newest-retained-row semantics; the foreground path never relies on that lookup. Added a deterministic real BashTool regression with same-PID COMPOSER_MARKER/DATETIME_MARKER logs and a session-ownership test.
- Fork hggm14o13iif confirmed it read root AGENTS.md, .agents/skills/testing/SKILL.md, tests/AGENTS.md, and the external task file before implementation.
- Parent verification: worktree `/home/ineersa/projects/agent-core-worktrees/fix-bash-output-contamination-on-reused-background-process-pid` is clean at commit e3636c7d8; diff is 6 files, 403 insertions, 91 deletions.
- PID audit note: exact record IDs are now used wherever BashTool or BackgroundProcessCompletionPoller already owns one. BgStatusTool remains a user-facing PID API and resolves the newest retained record; historical same-PID selection remains a potential future record-id UX enhancement, but cannot contaminate foreground BashTool results.

## Task workflow update - 2026-07-16T16:59:10.624Z
- Summary: Reviewer verdict at e3636c7d8: APPROVE WITH SUGGESTIONS. Actionable findings to address before approval: restore misplaced BackgroundProcessRepository docblock; remove newly added dead markStoppedByUserForRecord; remove PID-based manager/store methods orphaned by this refactor instead of retaining parallel compatibility APIs; remove unused sessionId parameter from latest-PID lookup; update stale @see documentation; document intentional newest-PID public bg_status semantics and record-ID stop re-fetch rationale. Reviewer confirmed the core immutable-record fix and deterministic regression are sound.
- Reviewer found no critical/security correctness defects. Session ownership, output-cap semantics, record-ID stop behavior, and regression determinism were confirmed correct. A cleanup fork is required because task-to-pr policy addresses APPROVE WITH SUGGESTIONS before re-review.

## Task workflow update - 2026-07-16T16:59:49.604Z
- Recorded fork run: 6rv9pbvvbrwh
- Summary: Review-cleanup fork launched to address all actionable APPROVE WITH SUGGESTIONS findings, remove dead compatibility surface, fix documentation, adopt shared test temp isolation, run focused Castor validation, and commit.

## Task workflow update - 2026-07-16T17:19:35.357Z
- Recorded fork run: 6rv9pbvvbrwh
- Validation: castor test --filter='ForegroundCompletionReturnsCurrentRecordLogWhenPidIsReused|ReadLogFullForRecordEnforcesSessionOwnership' — passed (2 tests, 11 assertions); castor test --filter='BashToolTest|BackgroundProcessManagerTest|BackgroundProcessCompletionPollerTest' — passed (55 tests, 184 assertions); castor phpstan — passed; castor deptrac — passed (0 violations); castor cs-check — passed after castor cs-fix; git show --check 9be6be2be — passed
- Summary: Review cleanup completed in commit 9be6be2be. Parent verified clean worktree, commit existence, expected seven-file cumulative diff, and no remaining dead PID APIs identified by review. Cleanup repaired repository docs, removed parallel/dead store and manager methods, documented retained newest-PID bg_status semantics, simplified exact-entity mutations, and migrated BashToolTest to shared TestDirectoryIsolation.
- Fork 6rv9pbvvbrwh addressed every delegated reviewer finding; no suggestion was intentionally skipped. Optional future bg_status record-id UX remains outside this task.
- Parent `rg` verification in the task worktree found only the intended entity-level `BackgroundProcess::markBackgrounded()` and manager call to it; removed PID compatibility methods have no remaining references.

## Task workflow update - 2026-07-16T17:26:20.094Z
- Summary: Second reviewer verdict at 9be6be2be: APPROVE WITH SUGGESTIONS. All prior findings are resolved and core correctness/security/tests remain approved. Remaining sensible cleanup: correct BashTool::readOutput PHPDoc to describe full exact-record output and document recordId; remove now-redundant Doctrine identity-map re-fetch/null-check in stopProcessEntity after refresh; opportunistically fix `Unscopped` typo. Pending-notification PID map collision was explicitly marked pre-existing/no-action for this PR.
- Task-to-pr policy requires another narrow fork because reviewer returned APPROVE WITH SUGGESTIONS rather than APPROVED.

## Task workflow update - 2026-07-16T17:26:37.555Z
- Recorded fork run: ircaazgoqglr
- Summary: Second narrow review-cleanup fork launched for readOutput PHPDoc, redundant stop re-fetch removal, typo correction, focused Castor validation, and commit.

## Task workflow update - 2026-07-16T17:28:08.905Z
- Recorded fork run: ircaazgoqglr
- Validation: castor test --filter='ForegroundCompletionReturnsCurrentRecordLogWhenPidIsReused|ReadLogFullForRecordEnforcesSessionOwnership|BashToolTest|BackgroundProcessManagerTest|BackgroundProcessCompletionPollerTest' — passed (55 tests, 184 assertions); castor phpstan — passed; castor cs-check — passed; castor deptrac — passed (0 violations); git show --check 20e5b3c24 — passed
- Summary: Second review cleanup completed in commit 20e5b3c24028c3d59e45eb21b9078a2faa1299bf. Corrected exact-record full-output PHPDoc, removed redundant post-refresh identity-map re-fetch, and fixed poller comment typo. Core behavior unchanged; PID-keyed pending-notification map intentionally remains out of scope per reviewer.
- Fork ircaazgoqglr confirmed root AGENTS.md, testing skill, tests/AGENTS.md, and task file were read; Castor-only QA used. Parent verified task worktree clean at 20e5b3c24.

## Task workflow update - 2026-07-16T17:33:42.357Z
- Validation: Final reviewer — APPROVED; Final-HEAD focused castor test filter for BashToolTest, BackgroundProcessManagerTest, BackgroundProcessCompletionPollerTest and two regressions — passed (55 tests, 184 assertions); Final-HEAD castor phpstan — passed; Final-HEAD castor deptrac — passed (0 violations); Final-HEAD castor cs-check — passed; Task branch clean at 20e5b3c24028c3d59e45eb21b9078a2faa1299bf
- Summary: Final reviewer verdict at HEAD 20e5b3c24028c3d59e45eb21b9078a2faa1299bf: APPROVED with no issues or actionable nice-to-haves. Reviewer confirmed immutable record identity end-to-end, session ownership/security, stop/signal lifecycle, output-cap semantics, documented newest-PID bg_status contract, exact completion-poller identity, no dead compatibility APIs, and deterministic regression value. All first- and second-review findings are resolved.
- Non-actionable/out-of-scope observation retained: completion poller pending notification map remains PID-keyed; it is pre-existing, cannot contaminate foreground BashTool output, and was explicitly excluded from this narrow PR.

## Task workflow update - 2026-07-16T17:36:07.346Z
- Validation: move_task CODE-REVIEW castor check — failed cache guard only (llama-proxy entries 371→372); warmup required before retry
- Summary: First CODE-REVIEW transition gate failed only on deterministic llama-proxy cache-growth guard: cache entries increased 371→372 during castor check. Castor instructed warming with `castor test:llm-real` and retrying. No code/test/static-analysis failure was reported; task remains IN-PROGRESS and no push/PR occurred.
- Following testing runbook: warm the complete live-LLM group through Castor, verify proxy entry count stabilizes on repeat, then retry deterministic gate without disabling the guard.

## Task workflow update - 2026-07-16T17:36:19.978Z
- Recorded fork run: gwropp0cjzox
- Summary: Launched llama-proxy warmup fork to run full castor test:llm-real twice, verify cache-entry stabilization, and preserve clean worktree before retrying CODE-REVIEW gate.

## Task workflow update - 2026-07-16T17:37:50.270Z
- Recorded fork run: gwropp0cjzox
- Validation: castor test:llm-real warm run 1 — passed (10 tests, 122 assertions), cache 372→372; castor test:llm-real warm run 2 — passed (10 tests, 122 assertions), cache 372→372; llama-proxy health — OK, normalization enabled; git status --short — clean
- Summary: Llama-proxy warmup completed without filesystem changes. Proxy healthy and cache stable at 372 entries / 27,712,585 bytes across two consecutive complete castor test:llm-real runs. Worktree remains clean; deterministic CODE-REVIEW gate can be retried without disabling safeguards.
- Warmup fork gwropp0cjzox read root AGENTS.md, testing skill, and tests/AGENTS.md; used Castor-only QA and did not clear cache, disable guard, or touch workers.

## Task workflow update - 2026-07-16T17:40:38.615Z
- Validation: second move_task CODE-REVIEW castor check — failed cache guard only (374→375)
- Summary: Second CODE-REVIEW gate retry again failed only on llama-proxy cache growth, now 374→375, despite two preceding full test:llm-real runs stable at 372. This indicates a gate-specific or concurrent normalized request rather than an ordinary unwarmed full llm-real cassette. No PR/push occurred; task remains IN-PROGRESS while gate/proxy evidence is diagnosed rather than repeatedly warming blindly.
- Launching read-only diagnostics of per-run Castor artifacts, newest proxy cassette metadata, and differences between standalone test:llm-real and check-lane/preflight request keys. No cache guard override or worker signals will be used.

## Task workflow update - 2026-07-16T17:48:41.868Z
- Validation: diagnostic scouts: both failed gate runs had all test/deptrac/phpstan/cs/controller-replay/tui/llm-real lanes pass; only cache guard failed; current process inspection — no active castor check or test:llm-real process
- Summary: Read-only diagnostics attributed repeated cache-guard failures to concurrent sibling-worktree live-LLM traffic, not this branch: fork-mvp-03 ran branch-exclusive ForkDeferredLiveE2eTest requests during the second gate window. Both failed gate report directories show every QA lane passed; only the global proxy-entry guard failed. Current process inspection shows no active castor check/test:llm-real/llm-real PHPUnit process; proxy is now at 384 entries after external warming. Retrying immediately with safeguards enabled.
- Root operational limitation identified: castor check is serialized across worktrees, but standalone castor test:llm-real is not; global llama-proxy cassette growth from a sibling focused run can invalidate an otherwise passing deterministic gate. No project code was changed and no guard was disabled.

## Task workflow update - 2026-07-16T17:50:56.393Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (121.9s).
- Pushed task/fix-bash-output-contamination-on-reused-background-process-pid to origin.
- branch 'task/fix-bash-output-contamination-on-reused-background-process-pid' set up to track 'origin/task/fix-bash-output-contamination-on-reused-background-process-pid'.
- Created PR: https://github.com/ineersa/agent-core/pull/294

## Task workflow update - 2026-07-16T17:51:03.169Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/294
- Updated PR Status: open
- Validation: deterministic castor check — passed (121.9s); branch pushed to origin/task/fix-bash-output-contamination-on-reused-background-process-pid; PR created: https://github.com/ineersa/agent-core/pull/294
- Summary: Task-to-pr completed successfully. Third deterministic gate attempt ran without concurrent live-LLM cache pollution: full castor check passed in 121.9s. Branch pushed and PR #294 created. Task intentionally remains CODE-REVIEW; no merge performed.

## Task workflow update - 2026-07-16T20:20:27.387Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-bash-output-contamination-on-reused-background-process-pid into integration checkout.
- Merge made by the 'ort' strategy.
 src/CodingAgent/Entity/BackgroundProcess.php       |   2 +-
 .../Entity/BackgroundProcessRepository.php         |  19 ++
 .../BackgroundProcessCompletionPoller.php          |  10 +-
 .../Tool/BackgroundProcess/ProcessStore.php        |  65 +-----
 src/CodingAgent/Tool/BackgroundProcessManager.php  | 250 +++++++++++++--------
 src/CodingAgent/Tool/BashTool.php                  |  30 +--
 tests/CodingAgent/Tool/BashToolTest.php            | 148 +++++++++---
 7 files changed, 323 insertions(+), 201 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-bash-output-contamination-on-reused-background-process-pid.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-bash-output-contamination-on-reused-background-process-pid.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: gh pr view 294 — state MERGED, GitGuardian check SUCCESS; integration checkout clean before DONE transition
- Summary: PR #294 confirmed merged on GitHub at 2026-07-16T20:19:47Z with merge commit 5cb94628bc9c6b24ac41131f47b377a20eba8424. Moving task to DONE and syncing integration checkout; no further implementation changes.

## Task workflow update - 2026-07-16T20:20:45.341Z
- Recorded fork run: g43pqw46zall
- Summary: DONE transition succeeded: task branch merged into integration checkout, remote changes pulled, task worktree removed, and IDEA exclusions cleaned. Required post-merge `LLM_MODE=true castor check` launched in integration checkout via validation fork.
