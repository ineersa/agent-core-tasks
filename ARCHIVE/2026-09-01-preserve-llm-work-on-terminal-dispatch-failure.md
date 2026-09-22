# Preserve LLM work when terminal failure dispatch fails

## Goal
Follow-up to merged PR #449. User approved the invariant that `LlmWorkerFailedEventSubscriber` must never swallow failure to dispatch the terminal `LlmStepResult` to run_control. Current behavior logs and returns, after which Symfony rejects/removes the exhausted `ExecuteLlmStep`; the run can remain Working forever. Change it to log and rethrow so Symfony aborts rejection and the original Doctrine message remains recoverable for redelivery after transport recovery, even if the provider is invoked again.

Also address owner inline review comment on PR #449: `RetryableLlmStepFailureException` in AgentCore must not depend on Symfony Messenger. `forceRetry:false` is equivalent to normal Messenger retry-strategy behavior, so keep the structured AgentCore failure exception framework-neutral and let CodingAgent/Messenger apply its ordinary bounded retry policy.

Keep this minimal: no finalizer queue, new consumer, setting, schema, outbox, fallback, or compatibility path.

## Acceptance criteria
- A failed terminal-result dispatch is logged and the exact throwable escapes `LlmWorkerFailedEventSubscriber`; it is never swallowed
- Deterministic proof covers Symfony Worker ordering: when the failed-event subscriber throws, the original `ExecuteLlmStep` envelope is not rejected or acknowledged and remains recoverable for later redelivery
- Successful terminal-result dispatch behavior remains unchanged and occurs exactly once
- `RetryableLlmStepFailureException` has no Symfony Messenger dependency while retryable provider failures still use the configured bounded `llm` transport retry strategy
- No new queue, consumer, setting, schema field, storage path, or provider retry policy
- Focused subscriber/worker/Messenger tests, deptrac, phpstan, dead-code, and cs-check pass

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/454
PR Status: merged
Started: 2026-09-01T21:33:51.831Z
Completed: 2026-09-01T23:16:46.867Z

## Work log
- Created: 2026-09-01T21:33:34.296Z

## Task workflow update - 2026-09-01T21:33:51.831Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure.
- Validation: PR #449 post-merge castor check qa-20260901-213053-373841-9c37428c PASS; Source proof: Symfony Worker dispatches WorkerMessageFailedEvent before receiver->reject(); subscriber throw aborts rejection; User explicitly approved provider re-execution on recovery rather than swallowing work
- Summary: Task started from merged PR #449 follow-up. Finalized behavior: log and rethrow terminal run_control dispatch failures so Symfony does not reject the original ExecuteLlmStep; provider re-execution on later recovery is explicitly acceptable. Also remove the new direct Symfony Messenger dependency from AgentCore's structured retryable failure exception, relying on normal bounded Messenger retry semantics. No finalizer queue or new product surface.

## Task workflow update - 2026-09-01T21:34:49.758Z
- Summary: Routing pass complete in the dedicated worktree. Main owns one cohesive slice across the existing failure exception, terminal failure subscriber, and their tests. Exact changes: make AgentCore `RetryableLlmStepFailureException` a framework-neutral RuntimeException because normal Messenger strategy already provides bounded retries; change subscriber catch to log and rethrow the same dispatch throwable; replace the swallow test and add deterministic Symfony Worker proof that a subscriber throw occurs before receiver reject/ack. Update Application topology comment to state failed terminal delivery leaves original work recoverable. No unresolved product/API decision.
- Ownership: owner=main; fork_run=none; revision=a2ca38bc1; scope=framework-neutral retryable LLM exception plus no-swallow terminal dispatch and deterministic Worker rejection-order proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-01T21:43:31.383Z
- Validation: Focused Castor tests PASS: 15 tests, 85 assertions; castor deptrac PASS: 0 violations; castor phpstan PASS: 0 errors; castor dead-code PASS: 0 errors; castor cs-check PASS after castor cs-fix; castor docs:validate PASS: 17 documents; git diff --check PASS; IDE diagnostics clean
- Summary: Implemented the approved no-swallow behavior. `LlmWorkerFailedEventSubscriber` logs and rethrows the exact run_control dispatch throwable, leaving the original ExecuteLlmStep unacknowledged/unrejected. AgentCore's structured retryable failure is now a framework-neutral RuntimeException; normal Messenger retry strategy remains authoritative. Added deterministic subscriber and Symfony Worker ordering proof.
- Ownership: owner=main; fork_run=none; revision=a2ca38bc1; scope=framework-neutral retryable LLM exception plus no-swallow terminal dispatch and deterministic Worker rejection-order proof; outcome=completed; commit=5f02b380f

## Task workflow update - 2026-09-01T21:51:54.955Z
- Summary: Independent review found no code, correctness, security, architecture, privacy, dead-code, or specification issue at 5f02b380f. Reviewer verified all four required Messenger semantics against Symfony 8.1 vendor internals. Its only REQUEST CHANGES item was absence of full `castor check`; task-to-pr procedure deliberately assigns that gate to the CODE-REVIEW transition, so a follow-up reviewer is being asked to distinguish code approval from the pending transition gate rather than duplicate the full gate.
- Review: role=reviewer; run_id=subagent-inline; revision=5f02b380f; scope=origin/main...HEAD specification-fidelity and Symfony Messenger semantics; verdict=REQUEST CHANGES (process-only: castor check pending transition); code_findings=none

## Task workflow update - 2026-09-01T21:59:08.447Z
- Validation: Reviewer verdict: APPROVE WITH SUGGESTIONS at 5f02b380f; Focused Castor tests PASS: 15 tests/85 assertions; deptrac/phpstan/dead-code/cs-check/docs:validate/git diff --check PASS
- Summary: Follow-up independent review at unchanged 5f02b380f returned APPROVE WITH SUGGESTIONS. No code, correctness, security, architecture, privacy, specification-fidelity, or focused-proof blockers. Reviewer verified exact Symfony Worker/Doctrine envelope preservation and bounded retry equivalence against installed Symfony 8.1 sources. NTH wording/fixture suggestions are optional and do not affect behavior.
- Review: role=reviewer; run_id=subagent-inline-followup; revision=5f02b380f; scope=unchanged code plus task-to-pr gate ownership clarification; verdict=APPROVE WITH SUGGESTIONS; blockers=none

## Task workflow update - 2026-09-01T22:00:51.859Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (78.1s).
- Pushed task/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure to origin.
- branch 'task/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure' set up to track 'origin/task/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure'.
- Created PR: https://github.com/ineersa/agent-core/pull/454
- Validation: Focused Castor tests: 15 tests/85 assertions PASS; deptrac/phpstan/dead-code/cs-check/docs:validate/git diff --check PASS; Independent reviewer APPROVE WITH SUGGESTIONS at 5f02b380f; no blockers
- Summary: Implementation complete at 5f02b380f. Terminal run_control dispatch errors are logged and rethrown exactly, so Symfony does not ack/reject the original ExecuteLlmStep and Doctrine can redeliver it after recovery. AgentCore retry failure is framework-neutral; normal bounded llm transport retries remain unchanged. Independent review approved with no blockers.

## Task workflow update - 2026-09-01T22:01:17.573Z
- Validation: CODE-REVIEW castor check PASS: qa-20260901-215929-395528-74ff5d96 in 78.1s; JUnit audit: zero test cases over 10s; max 6.414s; PR #454: https://github.com/ineersa/agent-core/pull/454; Worktree clean and branch tracks origin
- Summary: PR #454 created and branch pushed. Deterministic transition gate passed; worktree clean. Implementation is ready to merge.

## Task workflow update - 2026-09-01T23:16:46.867Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure: ide_close_project returned isError.
- Merged task/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure into integration checkout.
- Merge made by the 'ort' strategy.
 src/AgentCore/Application/AGENTS.md                |   2 +-
 .../Handler/RetryableLlmStepFailureException.php   |  15 +--
 .../Messenger/LlmWorkerFailedEventSubscriber.php   |   6 +-
 .../Handler/ExecuteLlmStepWorkerTest.php           |   4 +-
 .../LlmWorkerFailedEventSubscriberTest.php         | 106 +++++++++++++++++++--
 5 files changed, 107 insertions(+), 26 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-01-preserve-llm-work-on-terminal-dispatch-failure.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #454 state=MERGED at 2026-09-01T23:16:18Z; CODE-REVIEW castor check qa-20260901-215929-395528-74ff5d96 PASS in 78.1s; Independent review APPROVE WITH SUGGESTIONS; no blockers
- Summary: PR #454 merged at e48d7dd81a0d0c8f1b1bbfa8ddadbeac30f3a2d8. Completing task and integrating into primary checkout.

## Task workflow update - 2026-09-01T23:18:50.793Z
- Validation: Post-merge castor check PASS: qa-20260901-231654-439872-2b7fb8ff in 206.3s; unit 4664 tests/19058 assertions; controller-replay 6/88; TUI 8/60; llm-real 5/30; deptrac/phpstan/dead-code/cs-check/docs/catalog PASS; artifact integrity, leak check, cache cleanup, and llama-proxy guard PASS
- Summary: Post-merge integration validation passed after PR #454. Primary checkout is integrated; task worktree removed.

## Task workflow update - 2026-09-06T15:40:57+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
