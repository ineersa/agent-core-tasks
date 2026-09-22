# Fix duplicate tool execution after Messenger lease expiry

## Goal
Investigate and fix duplicate execution of long-running tool calls when the Doctrine Messenger delivery lease expires. Separate from the completed stale background-process shutdown cleanup task.

Confirmed evidence from .hatfield/logs/agent-2026-09-05.log, session/run 4:
- 03:41:27 UTC: worker 29 launched process 31865, record 1723, turn 560. At 03:42:28 worker 30 executed the same run/turn/step/tool-call identity and launched process 34734, record 1736.
- 03:45:00 UTC: worker 32 launched process 36128, record 1760, turn 602. At 03:46:00 worker 15688 executed the same identity and launched process 38999, record 1764.
- Both pairs share trace ID 141705511402775637729089819902942518514. The original castor check held the repository lock; its duplicate waited and reported lock acquisition failure.

The ~60-second duplicate starts match the tool transport redeliver_timeout=60. Consumers use --keepalive=5. No corresponding keepalive records were found for these deliveries. The precise cause of failed lease refresh remains unknown. A single controller tool-start event does not exclude multiple Messenger deliveries. No evidence establishes that tests killed the invoking worker.

Likely entry points: ExecuteToolCallWorker, ToolExecutor, BashTool, BackgroundProcessManager, ConsumerSupervisor, JsonlProcessAgentSessionClient, Symfony Messenger Worker and Doctrine transport keepalive. Current result deduplication checks completed results, leaving concurrent in-flight execution unprotected.

Plan:
1. Correlate delivery attempts, worker identity, transport message identity, and keepalive updates. Inspect the actual running PHAR behavior, not source alone.
2. Reproduce lease loss/redelivery with isolated workers and deterministic barriers at the lowest correct layer. Establish why keepalive failed before selecting the fix.
3. Fix the lease-refresh cause and prevent a duplicate in-flight delivery from launching the same tool side effect again. Define crash recovery explicitly; do not claim arbitrary external side effects can be made exactly-once.
4. Prove legitimate long-running execution, duplicate delivery, and worker-failure recovery under relevant concurrent lanes.

Constraints: use existing Symfony Messenger/Lock and project storage facilities; no blind timeout increases, disabled locks, or blanket error suppression. Never signal root-owned or HATFIELD_SESSION_ID processes. Keep new diagnostics structured and free of raw commands/prompts/secrets.

## Acceptance criteria
- Reproduce and explain why the affected long-running tool delivery lost its lease despite --keepalive=5, with delivery/keepalive evidence.
- Concurrent delivery of the same tool-call identity does not launch a second live command while the first execution remains active.
- Legitimate long-running tools retain lease refresh and complete normally; worker-failure recovery remains supported and its side-effect limitations are documented.
- Deterministic lowest-layer regression tests cover duplicate delivery and recovery without arbitrary sleeps or timeout increases; owned test resources have deterministic teardown.
- Relevant concurrent Castor validation and full castor check pass, with no leaked QA workers.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/470
PR Status: merged
Started: 2026-09-05T15:46:11+00:00
Completed: 2026-09-05T19:04:17+00:00

## Work log
- Created: 2026-09-05T04:11:03+00:00

## Task workflow update - 2026-09-05T15:46:11+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry.

## Task workflow update - 2026-09-05T15:47:09+00:00
- Summary: Main routing refreshed: ExecuteToolCallWorker invokes ToolExecutor and only has durable pending-deferred guard; ConsumerSupervisor launches keepalive=5 workers. Domain includes delivery identity run/tool-call and lifecycle running/completed/recoverable, distinct from OS process ownership. Existing tool result storage/locking and Symfony Messenger are required facilities. Delegating cohesive runtime investigation and implementation due to substantial process/concurrency validation. Prior scout historical same-call duplicate evidence retained; exact lease failure still unknown. No new product settings or unsupported exactly-once claims.
- Ownership: owner=fork; fork_run=none; revision=44cb6fcd72754ec2fe55068a6eda5fc65afab7d4; scope=Reproduce lease loss, fix tool delivery ownership and focused regression proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-05T15:59:31+00:00
- Summary: Paused implementation while reopening original task 2026-08-24-fix-out-of-order-deepseek-stream-deltas per user request. Original history reveals unresolved idle-worker one-shot alarm loss even after successful rearm; v8.1.4 fixed a separate proven Doctrine keepalive transaction race and passed one 146s live run. Prior explicit user instruction prohibits transient fencing or timeout masking. No product changes made by investigation fork; do not treat incomplete fork as validation-ready.
- Ownership: owner=fork; fork_run=agent_96f3197bee63cb7b; revision=44cb6fcd72754ec2fe55068a6eda5fc65afab7d4; scope=Reproduce lease loss, fix tool delivery ownership and focused regression proof; outcome=blocked; commit=none

## Task workflow update - 2026-09-05T16:14:25+00:00
- Summary: User finalized replacement scope: remove Messenger keepalive and elapsed-time reclaim across runtime transports; accept explicit manual recovery for abandoned messages. Do not debug SIGALRM further, add automatic ownership recovery, raise timeout as a substitute, or mask transient output. Use this existing worktree for implementation; reopened historical task records investigation. Verify actual /repair, cancellation, and restart behavior and document limitations rather than assume. Follow-up created: 2026-09-05-revisit-safe-automatic-recovery-for-abandoned-messenger-deliveries.

## Task workflow update - 2026-09-05T16:15:03+00:00
- Summary: Main routing: six session transport DSNs originate JsonlProcessAgentSessionClient lines567-574; consumer flag/alarm guard in ConsumerSupervisor; manual recovery SessionRepairService and RepairCommandHandler. Existing tests MessengerDoctrineRedeliverTimeoutLeaseTest and ConsumerSupervisorTest need updated behavior proof. Delegate cohesive transport/runtime implementation because adapter discovery and process validation are substantial. Domain shape: queued/unclaimed delivery versus claimed delivery; age must not make a claimed delivery eligible again. Use existing Symfony facilities, not huge timeout or zero without verifying upstream semantics.
- Ownership: owner=fork; fork_run=none; revision=44cb6fcd72754ec2fe55068a6eda5fc65afab7d4; scope=Remove keepalive and elapsed-time reclaim, verify manual recovery, focused runtime proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-05T16:46:55+00:00
- Validation: Both forks read and followed testing skill and tests/AGENTS.; Initial focused tests15/138 and controller replay6/88 passed; final focused8/90 passed.; Production PHPStan, Deptrac, style, docs passed. Path-scoped test PHPStan reports PHPUnit dynamic-call findings; full configured gate pending.; Parent verified clean worktree and git diff --check; castor check deferred per task-start.
- Summary: Implementation complete on task branch at c5b48e2db after74924d0aa. Removed consumer keepalive/alarm requirement and automatic age reclaim via SQLite claim-only Symfony Doctrine connection/factory. Parent inspection removed unsupported Oracle/locking fallbacks and meaningless explicit timeout DSNs. Manual recovery verified: restart alone does not release claimed rows; /repair redrives current operation as fresh envelope, leaving abandoned claimed row retained. No automatic recovery added. Active main/session unchanged. Next phase task-to-pr for independent review and full check.
- Ownership: owner=fork; fork_run=agent_6f7f347a2aee80ce; revision=44cb6fcd72754ec2fe55068a6eda5fc65afab7d4; scope=Remove keepalive and elapsed-time reclaim, verify manual recovery, focused runtime proof; outcome=completed; commit=74924d0aa673a52ad887778c870c1fb4fd6b26d9
- Ownership: owner=fork; fork_run=agent_0aad3316bb802162; revision=74924d0aa673a52ad887778c870c1fb4fd6b26d9; scope=Simplify SQLite transport and prove manual repair fresh delivery; outcome=completed; commit=c5b48e2db4982f13ab24d60646303062d6e1644e

## Task workflow update - 2026-09-05T17:11:21+00:00
- Summary: Reviewer agent_e32b04b50b31bb65 reviewed c5b48e2db read-only: APPROVE WITH SUGGESTIONS, no supported-path blocker reported. Independently verified atomic SQLite claim, DI decoration, no age reclaim or keepalive, manual repair limits; focused tests/static/deptrac/docs passed. Suggestions: broader no-keepalive argv assertion, auto-setup flag parity, SQLite atomicity rationale comment, test naming and failure-safe cleanup. Full castor check pending. User requested review only; no PR transition performed.
- Review: role=reviewer; artifact=agent_e32b04b50b31bb65; revision=c5b48e2db4982f13ab24d60646303062d6e1644e; scope=specification fidelity, claim-only transport and manual repair proof; outcome=APPROVE WITH SUGGESTIONS

## Task workflow update - 2026-09-05T17:18:52+00:00
- Summary: Merged origin/main without conflicts. Inspected failed gate qa-20260905-171514-6282-fe5340b4: dead-code rejects two unmatched missingType.iterableValue ignores in ClaimOnlyDoctrineTransportFactory. Fix deterministic annotation issue before retry.
- Ownership: owner=main; fork_run=none; revision=e9d77d654; scope=Replace unmatched PHPStan suppressions with options map annotations; outcome=assigned; commit=none

## Task workflow update - 2026-09-05T17:18:52+00:00
- Summary: Merged origin/main without conflicts. Inspected failed gate qa-20260905-171514-6282-fe5340b4: dead-code rejects two unmatched missingType.iterableValue ignores in ClaimOnlyDoctrineTransportFactory. Fix deterministic annotation issue before retry.
- Ownership: owner=main; fork_run=none; revision=e9d77d654; scope=Replace unmatched PHPStan suppressions with options map annotations; outcome=assigned; commit=none

## Task workflow update - 2026-09-05T17:26:56+00:00
- Validation: castor dead-code PASS; castor phpstan scoped production PASS; castor cs-check PASS; Reviewer independently reran all three PASS; worktree clean
- Summary: Fixed dead-code gate failure at6f50f2ac4 using explicit array<array-key,mixed> options types instead of unmatched ignores. Main merged. Reviewer agent_e32b04b50b31bb65 reapproved6f50f2ac4 with suggestions; no new blockers or behavior changes.
- Ownership: owner=main; fork_run=none; revision=e9d77d654; scope=Replace unmatched PHPStan suppressions with options map annotations; outcome=completed; commit=6f50f2ac4
- Review: role=reviewer; artifact=agent_e32b04b50b31bb65; revision=6f50f2ac4; scope=merge and gate fix; outcome=APPROVE WITH SUGGESTIONS

## Task workflow update - 2026-09-05T18:08:16+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (68.7s).
- Pushed task/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry to origin.
- branch 'task/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry' set up to track 'origin/task/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry'.
- Created PR: https://github.com/ineersa/agent-core/pull/470
- Summary: User authorized retry after interrupting prior gate to address Datadog issues. Reviewed revision6f50f2ac4 unchanged.

## Task workflow update - 2026-09-05T18:08:16+00:00
- Summary: Retry returned generic move_task error again. No PR exists; worktree clean. QA child remains active (timeout PID4314 parent29; report qa-20260905-180703-4315-7ca51bd3) after tool error; not retrying concurrently or signaling session processes. Available completed static and llm-real logs pass, but full gate outcome not established. Earlier report qa-20260905-180452-55-aca47f90 has incomplete unit log. Transition remains blocked pending actual gate result.

## Task workflow update - 2026-09-05T18:08:39+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/470
- Updated PR Status: open
- Summary: Correction to raced status observation: transition completed asynchronously despite generic tool response. Board confirms CODE-REVIEW and castor check PASS68.7s; GitHub confirms PR470 OPEN at6f50f2ac4. Prior pending/no-PR observation superseded.

## Task workflow update - 2026-09-05T18:32:41+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User replaces never-reclaim implementation with stock Symfony Doctrine ten-year redeliver_timeout, keeping keepalive disabled. Delete custom connection/factory and verify finite lease behavior. User accepts eventual reclaim after ten years.

## Task workflow update - 2026-09-05T18:32:53+00:00
- Summary: Main routing unchanged: six DSNs in JsonlProcessAgentSessionClient, custom two-class adapter, transport fixtures and regression tests, docs. New organizing shape is stock finite delivery lease in seconds: ten365-day years315360000; keepalive remains off. Delete adapter-specific tests and unsupported never-reclaim statements. Delegating bounded replacement due to real transport/manual-repair test rewrite and validation.
- Ownership: owner=fork; fork_run=none; revision=6f50f2ac4; scope=Replace custom claim-only transport with stock ten-year redelivery and focused proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-05T18:45:08+00:00
- Validation: Focused13tests141assertions PASS; Controller replay6tests88assertions PASS; Runtime PHPStan, dead-code, style, Deptrac, docs PASS; Reviewer independently confirmed focused/replay/phpstan/docs and no stale workers; full transition gate pending
- Summary: User-requested ten-year stock Doctrine revision complete8a28263cd. Custom connection/factory deleted; keepalive still off. Finite315360000-second lease on all6 queues, real inside/beyond horizon proof and manual repair tests pass. Prior reviewer resume unavailable after parent lifetime; replacement agent_7412d4033b536afe APPROVE with independent focused validation.
- Ownership: owner=fork; fork_run=agent_ab0065e8a66f40b8; revision=6f50f2ac4; scope=Replace custom claim-only transport with stock ten-year redelivery and focused proof; outcome=completed; commit=8a28263cdec8742bb391b9a53421f10e982464e4
- Review: role=reviewer; artifact=agent_7412d4033b536afe; revision=8a28263cd; scope=finalized finite lease specification fidelity; outcome=APPROVE

## Task workflow update - 2026-09-05T18:46:53+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (81.7s).
- Pushed task/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry to origin.
- branch 'task/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry' set up to track 'origin/task/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry'.
- PR already exists: https://github.com/ineersa/agent-core/pull/470
- Summary: Submit approved8a28263cd: stock Symfony ten-year lease, keepalive disabled, custom transport removed.

## Task workflow update - 2026-09-05T19:04:17+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry: ide_close_project returned isError.
- Merged task/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry into integration checkout.
- Merge made by the 'ort' strategy.
 docs/async-runtime-architecture.md                                                     |   4 +-
 docs/session-storage.md                                                                |  14 +++----
 src/CodingAgent/Runtime/Controller/ConsumerSupervisor.php                              |  38 +++---------------
 src/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClient.php                     |  17 ++++----
 tests/CodingAgent/Castor/QaSessionEnvSanitizationTest.php                              |   2 +-
 tests/CodingAgent/Messenger/MessengerDoctrineRedeliverTimeoutLeaseTest.php             | 279 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++--------------
 tests/CodingAgent/Runtime/Controller/ConsumerSupervisorTest.php                        |   3 +-
 tests/CodingAgent/Runtime/Controller/E2E/ControllerE2eTestCase.php                     |  12 +++---
 tests/CodingAgent/Runtime/Controller/E2E/ControllerReplayE2eTestCase.php               |  12 +++---
 tests/CodingAgent/Runtime/Controller/HeadlessControllerSessionOwnerLockProcessTest.php |  12 +++---
 tests/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClientTransportDsnTest.php   |   2 +-
 11 files changed, 294 insertions(+), 101 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-duplicate-tool-execution-after-messenger-lease-expiry.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR470 merged2026-09-05T19:03:47Z. Approved stock Symfony ten-year timeout with keepalive removed; premerge full gate passed81.7s. Postmerge integration gate next.

## Task workflow update - 2026-09-05T19:05:29+00:00
- Summary: DONE transition complete; clean integration checkout and removed task worktree verified. Postmerge LLM_MODE=true castor check running as background PID15208, log .hatfield/tmp/bg/bb8c5cd3f56eacbb.log; outcome pending.

## Task workflow update - 2026-09-05T19:06:37+00:00
- Validation: QA report: var/reports/qa-20260905-190425-15211-7d9b8c71; 4759 tests19432assertions; controller replay6/88; TUI8/61; llm-real5/30 PASS; PHPStan, dead-code, Deptrac, style, docs, catalog PASS; no QA-owned leaks; cache394→394
- Summary: Postmerge integration castor check completed exit0 on8853dc2ff53d05e7b34df8248430652d752750cd. All10 lanes passed; artifact integrity, worker/tmux leak check, and cache guard passed. DONE validation complete.

## Task workflow update - 2026-09-06T15:41:20+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
