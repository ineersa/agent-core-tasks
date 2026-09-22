# Fix LLM cancellation completion and post-cancel TUI ghost streams

## Goal
Session 1 reproduced a cancellation defect: cancel was canonically terminalized while an LLM operation was still in flight; the LLM worker continued because its operational-status read was stale; trailing transient assistant/tool events reopened TUI activity and created noncanonical ghost transcript blocks. The later tool call was not executed or persisted.

Finalized behavior: do not hide unresolved in-flight LLM work behind immediate logical completion. Cancellation of an active LLM operation must remain Cancelling until the worker acknowledges abort/completion through the normal result path. Make cancellation-status reads fresh across processes, check cancellation before/during/after stream consumption, and fence terminal TUI state/transcript projection against stale seq=0 assistant/tool events until a canonical sequenced event proves a new turn began. Preserve legitimate multi-turn continuation after terminal events.

## Acceptance criteria
- Cancelling an in-flight LLM operation does not immediately append agent_end(cancelled); the run remains Cancelling until its worker result/abort is handled.
- LLM cancellation status reads observe cross-process status changes rather than Doctrine identity-map state.
- LLM stream consumption checks cancellation before, during, and after consumption and returns an aborted outcome when cancellation is observed.
- After RunCancelled, stale seq=0 assistant/tool stream events cannot reopen activity or add ghost transcript blocks.
- A canonical sequenced new-turn event can still reopen a terminal run for legitimate multi-turn continuation.
- Deterministic lowest-layer regression tests cover the reported sequence without sleeps or timing races.
- Focused Castor validation passes during task-start; castor check is deferred to task-to-pr per workflow.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams
Fork run: tqqixi21qlnc
PR URL: https://github.com/ineersa/agent-core/pull/458
PR Status: merged
Started: 2026-09-03T18:35:35.310Z
Completed: 2026-09-03T21:20:04.993Z

## Work log
- Created: 2026-09-03T18:35:19.357Z

## Task workflow update - 2026-09-03T18:35:35.310Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams.
- Summary: User finalized behavior: active LLM cancellation must remain visible as Cancelling until worker acknowledgement; do not immediately hide unresolved work behind terminal completion. Also require fresh cross-process status reads, before/during/after stream cancellation checks, and a terminal fence against stale seq=0 assistant/tool events while preserving canonical multi-turn continuation.

## Task workflow update - 2026-09-03T18:36:17.021Z
- Ownership: owner=main; fork_run=none; revision=16ddf4ba0; scope=fresh cross-process cancellation status, in-flight LLM Cancelling lifecycle, stream cancellation checks, and terminal TUI transient-event fence with deterministic focused tests; outcome=assigned; commit=none

## Task workflow update - 2026-09-03T18:38:50.537Z
- Recorded fork run: w66sp63r97uf
- Ownership: owner=main; fork_run=none; revision=16ddf4ba0; scope=fresh cross-process cancellation status, in-flight LLM Cancelling lifecycle, stream cancellation checks, and terminal TUI transient-event fence with deterministic focused tests; outcome=blocked; commit=none
- Ownership: owner=fork; fork_run=w66sp63r97uf; revision=16ddf4ba0; scope=minimal complete implementation of finalized cancellation and post-terminal transient-event fixes, reusing existing tests with focused Castor validation; outcome=assigned; commit=none

## Task workflow update - 2026-09-03T18:52:03.918Z
- Summary: Parent review accepted the cancellation lifecycle, fresh Doctrine status, stream checks, and focused regression coverage. A narrow fork revision is in progress to remove an unnecessary public TUI helper/enum list and move the terminal fence before all stale-event side effects, reducing code while preserving behavior.
- Ownership: owner=fork; fork_run=w66sp63r97uf; revision=16ddf4ba0; scope=minimal complete implementation of finalized cancellation and post-terminal transient-event fixes, reusing existing tests with focused Castor validation; outcome=blocked; commit=97cb12640491d239b36325c7cda1113b21163f8e
- Ownership: owner=fork; fork_run=73ykqb4c16ki; revision=97cb12640491d239b36325c7cda1113b21163f8e; scope=remove unnecessary public TUI helper/list and fully ignore stale terminal seq=0 assistant/tool events using a minimal inline family predicate; outcome=assigned; commit=none

## Task workflow update - 2026-09-03T18:53:45.070Z
- Validation: castor test --filter='ApplyCommandHandlerTest|PlatformIntegrationTest|RunOperationalProjectionRepositoryTest|ActivityStateMachineTest|TuiRuntimeEventApplierTest' — PASS (126 tests, 332 assertions); castor cs-fix on touched files — PASS; castor phpstan on touched production files — PASS (0 errors); castor test --filter='ActivityStateMachineTest|TuiRuntimeEventApplierTest' after simplification — PASS (83 tests, 100 assertions); Parent reviewed commits 97cb12640 and f335167c1; worktree clean. castor check intentionally deferred to task-to-pr.
- Summary: Implemented and reviewed at task branch HEAD f335167c1. Active currentOperation now keeps cancellation in Cancelling until worker acknowledgement; operational status reads refresh Doctrine-managed state; LLM streaming checks cancellation before/during/after consumption; terminal TUI activity rejects seq=0 assistant/tool event families before any state/projection side effects while sequenced continuation remains supported. No new settings, storage, or public APIs.
- Ownership: owner=fork; fork_run=w66sp63r97uf; revision=16ddf4ba0; scope=minimal complete implementation of finalized cancellation and post-terminal transient-event fixes, reusing existing tests with focused Castor validation; outcome=completed; commit=97cb12640491d239b36325c7cda1113b21163f8e
- Ownership: owner=fork; fork_run=73ykqb4c16ki; revision=97cb12640491d239b36325c7cda1113b21163f8e; scope=remove unnecessary public TUI helper/list and fully ignore stale terminal seq=0 assistant/tool events using a minimal inline family predicate; outcome=completed; commit=f335167c103753db642715bb75223a753587b607

## Task workflow update - 2026-09-03T20:44:01.318Z
- Summary: Specification-fidelity reviewer requested changes at f335167c1: LlmStepResultHandler cancellation terminalization omitted the existing post-cancel AdvanceRun wake, which can strand queued follow-ups; using any currentOperation as active cancellation work also unintentionally captures standalone shell operations whose result path does not honor Cancelling. Revision required before CODE-REVIEW.
- Review: role=reviewer; artifact=foreground-subagent-result; revision=f335167c103753db642715bb75223a753587b607; scope=full origin/main...HEAD specification-fidelity, cancellation lifecycle/races, Doctrine freshness, stream checks, TUI fence, and lowest-layer tests; decision=REQUEST CHANGES; blockers=queued post-cancel follow-up drain missing from LlmStepResultHandler abort path; currentOperation check unintentionally changes standalone-shell cancellation

## Task workflow update - 2026-09-03T20:44:19.425Z
- Recorded fork run: s316okyn5fec
- Ownership: owner=fork; fork_run=s316okyn5fec; revision=f335167c103753db642715bb75223a753587b607; scope=limit wait-for-ack cancellation to genuine in-flight LLM work while preserving standalone-shell behavior, and restore post-cancel AdvanceRun wake from LLM terminalization with minimal focused tests; outcome=assigned; commit=none

## Task workflow update - 2026-09-03T20:49:11.849Z
- Validation: castor test --filter='ApplyCommandHandlerTest|LlmStepResultHandlerTest' — PASS (34 tests, 251 assertions); castor phpstan on ApplyCommandHandler.php and LlmStepResultHandler.php — PASS (0 errors); castor cs-check on four revision files — PASS; Parent inspected commit d6775f120; worktree clean.
- Ownership: owner=fork; fork_run=s316okyn5fec; revision=f335167c103753db642715bb75223a753587b607; scope=limit wait-for-ack cancellation to genuine in-flight LLM work while preserving standalone-shell behavior, and restore post-cancel AdvanceRun wake from LLM terminalization with minimal focused tests; outcome=completed; commit=d6775f120a72b09b51477a277fc83d7003509d78

## Task workflow update - 2026-09-03T20:59:22.042Z
- Summary: Re-review at d6775f120 approved with non-blocking suggestions. Both blockers are resolved: queued follow-ups receive the established post-cancel AdvanceRun wake, and standalone-shell operations preserve immediate cancellation while genuine LLM operations remain Cancelling until acknowledgement.
- Review: role=reviewer; artifact=/tmp/agent-core-review-d6775f120.md; revision=d6775f120a72b09b51477a277fc83d7003509d78; scope=full origin/main...HEAD specification-fidelity re-review including prior blockers, cancellation races, Doctrine freshness, stream checks, TUI fence, and proof placement; decision=APPROVE WITH SUGGESTIONS; blockers=none

## Task workflow update - 2026-09-03T21:00:50.829Z
- Validation: castor test --filter='ApplyCommandHandlerTest|LlmStepResultHandlerTest|PlatformIntegrationTest|RunOperationalProjectionRepositoryTest|ActivityStateMachineTest|TuiRuntimeEventApplierTest' — PASS (137 tests, 443 assertions); castor deptrac — PASS (0 violations/errors); castor phpstan — FAIL: repeated CancellationTokenInterface::isCancellationRequested() checks treated as pure/constant at LlmPlatformAdapter.php:388,418; cs-check and llm-real not reached
- Summary: Task-to-PR focused tests and Deptrac passed, but full PHPStan correctly flagged repeated cancellation-token reads as constant because the polling contract is not marked impure. A minimal contract annotation is required before remaining QA/re-review.

## Task workflow update - 2026-09-03T21:01:05.929Z
- Recorded fork run: 1ov59607irpy
- Ownership: owner=fork; fork_run=1ov59607irpy; revision=d6775f120a72b09b51477a277fc83d7003509d78; scope=mark the mutable cancellation poll contract PHPStan-impure so before/during/after checks analyze correctly; no behavior or test changes; outcome=assigned; commit=none

## Task workflow update - 2026-09-03T21:02:14.463Z
- Validation: castor phpstan --path=LlmPlatformAdapter.php — PASS (0 errors) after @phpstan-impure contract; castor phpstan --path=CancellationTokenInterface.php — PASS (0 errors); castor cs-check --path=CancellationTokenInterface.php — PASS; parent inspected dc659e260; worktree clean
- Ownership: owner=fork; fork_run=1ov59607irpy; revision=d6775f120a72b09b51477a277fc83d7003509d78; scope=mark the mutable cancellation poll contract PHPStan-impure so before/during/after checks analyze correctly; no behavior or test changes; outcome=completed; commit=dc659e260f581199145605cc8c7663c76d39612a

## Task workflow update - 2026-09-03T21:07:15.148Z
- Validation: castor phpstan at dc659e260 — FAIL only NullCancellationToken.php:9 (`impureMethod.pure`); mutable adapter polling errors are resolved; castor cs-check and castor test:llm-real — not reached due chained PHPStan failure
- Summary: Final full PHPStan now accepts mutable polling at adapter call sites but flags the constant NullCancellationToken implementation as declared impure through inheritance. The null implementation needs an explicit pure override; remaining cs-check/llm-real were not reached.

## Task workflow update - 2026-09-03T21:07:27.631Z
- Recorded fork run: tqqixi21qlnc
- Ownership: owner=fork; fork_run=tqqixi21qlnc; revision=dc659e260f581199145605cc8c7663c76d39612a; scope=declare the constant NullCancellationToken implementation pure under the impure polling interface and prove with full PHPStan; outcome=assigned; commit=none

## Task workflow update - 2026-09-03T21:10:49.075Z
- Validation: castor phpstan at 43395399f — PASS full project (0 errors); castor cs-check --path=NullCancellationToken.php — PASS; parent inspected 43395399f; worktree clean
- Ownership: owner=fork; fork_run=tqqixi21qlnc; revision=dc659e260f581199145605cc8c7663c76d39612a; scope=declare the constant NullCancellationToken implementation pure under the impure polling interface and prove with full PHPStan; outcome=completed; commit=43395399fafe2aaec6cedc42d773409db8ae25d9

## Task workflow update - 2026-09-03T21:13:49.413Z
- Validation: Focused task tests — PASS (137 tests, 443 assertions); castor deptrac — PASS (0 violations/errors); castor phpstan — PASS full project (0 errors); castor cs-check — PASS full project (0 files fixed); castor test:llm-real — PASS (5 tests, 30 assertions); Worktree clean at 43395399f; unresolved blockers: none
- Summary: Final reviewer approved HEAD 43395399f with no blockers. Task behavior is complete: genuine in-flight LLM cancellation remains Cancelling until acknowledgement, fresh status polling works before/during/after stream consumption, queued post-cancel follow-ups wake correctly, standalone-shell behavior is preserved, and stale seq=0 assistant/tool transients cannot reopen terminal TUI state while sequenced continuation remains allowed.
- Review: role=reviewer; artifact=/tmp/agent-core-review-43395399f.md; revision=43395399fafe2aaec6cedc42d773409db8ae25d9; scope=final full-task specification-fidelity readiness plus incremental pure/impure effect-contract verification; decision=APPROVE; blockers=none

## Task workflow update - 2026-09-03T21:16:31.328Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (145.3s).
- Pushed task/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams to origin.
- branch 'task/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams' set up to track 'origin/task/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams'.
- Created PR: https://github.com/ineersa/agent-core/pull/458
- Validation: Focused task tests: PASS (137 tests, 443 assertions); castor deptrac: PASS; castor phpstan: PASS (0 errors); castor cs-check: PASS; castor test:llm-real: PASS (5 tests, 30 assertions); CODE-REVIEW transition: deterministic castor check required
- Summary: Implemented and independently reviewed LLM cancellation lifecycle fixes: fresh cancellation status reads, cancellation checks before/during/after stream consumption, Cancelling retained until LLM acknowledgement with post-cancel wake, standalone-shell behavior preserved, and terminal TUI runs fenced from stale seq=0 assistant/tool transients. Final reviewer verdict: APPROVE at 43395399f; no blockers.

## Task workflow update - 2026-09-03T21:20:04.993Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams: ide_close_project returned isError.
- Merged task/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams into integration checkout.
- Merge made by the 'ort' strategy.
 .../Application/Pipeline/ApplyCommandHandler.php   |  15 ++-
 .../Application/Pipeline/LlmStepResultHandler.php  |   9 ++
 .../Contract/Hook/CancellationTokenInterface.php   |   5 +
 .../Contract/Hook/NullCancellationToken.php        |   5 +
 .../SymfonyAi/LlmPlatformAdapter.php               |  58 ++++++----
 .../RunOperationalProjectionRepository.php         |   5 +
 src/Tui/Runtime/ActivityStateMachine.php           |  21 ++--
 src/Tui/Runtime/TuiRuntimeEventApplier.php         |  13 +++
 .../Pipeline/ApplyCommandHandlerTest.php           | 122 +++++++++++++--------
 .../Pipeline/LlmStepResultHandlerTest.php          |  10 ++
 .../SymfonyAi/PlatformIntegrationTest.php          |  49 ++++++++-
 .../RunOperationalProjectionRepositoryTest.php     |  18 +++
 tests/Tui/Runtime/ActivityStateMachineTest.php     |  26 +++++
 tests/Tui/Runtime/TuiRuntimeEventApplierTest.php   |  30 +++++
 14 files changed, 303 insertions(+), 83 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-03-fix-llm-cancellation-and-post-cancel-ghost-streams.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #458 state: MERGED at 2026-09-03T21:19:40Z; Pre-merge integration checkout: clean
- Summary: PR #458 merged on GitHub as 0785b7ae57d77d275c4198d47515ce64ec83da26. Moving the approved task into DONE and integrating the merged branch into the main checkout.

## Task workflow update - 2026-09-03T21:22:31.443Z
- Validation: Post-merge `LLM_MODE=true castor check` — FAIL only test:tui; QA report: var/reports/qa-20260903-212010-467984-726175df; Passing post-merge lanes: deptrac, test (4699 tests, 19209 assertions), controller-replay (6 tests, 88 assertions), llm-real (5 tests, 30 assertions), phpstan, dead-code, cs-check, docs:validate, catalog:version-check; test:tui: tests themselves passed (8 tests, 59 assertions) but lane exited 1 due 3 teardown warnings from TestDirectoryIsolation while removing TuiResumeSessionSwitchE2eTest temp home; residual file is var/tmp/tui-e2e-6da86303205ae42f/home/.hatfield/ai-catalog.yaml; QA artifact integrity PASS; QA-owned process/tmux leak check PASS; llama-proxy cache guard PASS; Integration checkout clean; task worktree removed; local main ahead of origin/main by two workflow merge commits
- Summary: Task is DONE and PR #458 is merged. Post-merge integration QA completed with every lane green except test:tui, which failed solely on three teardown warnings from TuiResumeSessionSwitchE2eTest leaving generated `home/.hatfield/ai-catalog.yaml`; QA leak check found no owned processes or tmux sessions. This is recorded rather than retried or hidden.

## Task workflow update - 2026-09-06T15:40:58+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
