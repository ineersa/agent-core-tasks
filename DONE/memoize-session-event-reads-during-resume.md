# Memoize canonical event reads during session resume

## Goal
Session 33 resume synchronously calls `SessionRunEventStore::allFor($runId)` three times through `SessionInitializer`, `SessionTurnTreeProvider`, and `SessionTranscriptProvider`. Each call rereads, decodes, denormalizes, and sorts the same append-only parent `events.jsonl`, causing 1,335 decodes and three sorts for 445 events before the session can render.

Use a minimal process-local cache inside `SessionRunEventStore`, keyed by run ID/path and a cheap append-safe file signature such as size plus mtime (with PHP stat-cache handling). Repeated `allFor()` calls against an unchanged file must reuse the already parsed immutable event array. Explicitly invalidate/update the entry on writes performed through the store. Do not redesign provider APIs, introduce a second storage format, add asynchronous loading, or broaden this task into controller-startup profiling.

The canonical JSONL file remains the source of truth. The cache must observe external appends from another worker/process on the next read; do not rely only on process-local write invalidation.

## Acceptance criteria
- The normal branch-aware resume path may call `allFor()` three times, but unchanged `events.jsonl` content is physically read/decoded/sorted only once per process snapshot.
- Cache identity uses an append-safe file signature and clears PHP stat cache before checking; an external append that changes the file is visible on the next `allFor()` call.
- Store-owned append/write operations invalidate or refresh the affected cache entry immediately.
- Returned event ordering, denormalized event types, active-leaf filtering, `lastSeq`, and transcript content remain unchanged.
- Keep the production change localized to the event store unless a directly required test adjustment proves otherwise; no async loading framework or new transcript/cache service.
- Add a RED-GREEN regression that fails before memoization by counting the stable decode/read seam without adding a production API solely for tests, plus an external-append invalidation test.
- Run all QA through Castor, including focused event-store/session-resume tests and full `castor check` before CODE-REVIEW.

## Workflow metadata
Status: DONE
Branch: task/memoize-session-event-reads-during-resume
Worktree: /home/ineersa/projects/agent-core-worktrees/memoize-session-event-reads-during-resume
Fork run: 1jtd4g2e36b2
PR URL: https://github.com/ineersa/agent-core/pull/322
PR Status: merged
Started: 2026-07-26T23:41:52.575Z
Completed: 2026-07-27T02:39:46.415Z

## Work log
- Created: 2026-07-23T20:08:08.864Z

## Task workflow update - 2026-07-26T23:41:52.575Z
- Moved TODO → IN-PROGRESS.
- Created branch task/memoize-session-event-reads-during-resume.
- Created worktree /home/ineersa/projects/agent-core-worktrees/memoize-session-event-reads-during-resume.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/memoize-session-event-reads-during-resume.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/memoize-session-event-reads-during-resume.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/memoize-session-event-reads-during-resume.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/memoize-session-event-reads-during-resume.
- Summary: Started with explicit user constraint: minimal localized process-memory memoization only. No async loading, provider API redesign, new cache service, second storage format, or unrelated startup profiling.

## Task workflow update - 2026-07-26T23:46:28.847Z
- Recorded fork run: 1jtd4g2e36b2
- Summary: Narrow scout confirmed the entire fix belongs in SessionRunEventStore plus its existing test file. Launched RED→GREEN implementation fork 1jtd4g2e36b2 with a strict two-file ceiling: private path→{size,mtime,events} cache, clearstatcache signature checks, pre/post-read race guard, centralized successful-write invalidation, and three public-behavior regressions. No new service/API/async/profiling.

## Task workflow update - 2026-07-26T23:53:09.876Z
- Recorded fork run: 1jtd4g2e36b2
- Validation: RED focused test failed expected once-decode assertion before production change.; Focused SessionRunEventStore + sequencing PASS 15 tests / 56 assertions.; castor test PASS on retry 4528 tests / 15767 assertions; first run hit known unrelated SQLite timing threshold flake, focused middleware PASS 4/19.; castor deptrac PASS 0 violations; castor phpstan PASS 0 errors; castor cs-check clean; git diff --check clean.
- Summary: Implementation complete at GREEN c59f441f4e7265010e6847088271b06bb39a55cd after RED 5a0319dd4834a469bddd2bb0ea3a8d662275748e. Exact two-file scope: SessionRunEventStore now has one private path-keyed size+mtime decoded-event cache, clearstatcache signature reads, pre/post-read stability guard, and successful-write invalidation; existing SessionRunEventStoreTest adds three public-behavior regressions. No new service/API/config/caller/async/profiling work. Worktree clean; not pushed.

## Task workflow update - 2026-07-27T00:12:39.701Z
- Validation: Reviewer: APPROVED, no critical/issues/actionable findings.; Focused `castor test --filter='SessionRunEventStoreTest|SessionRunEventStoreSequencingTest'` PASS 15 tests / 56 assertions.; Implementation full `castor test` PASS 4528 tests / 15767 assertions on retry.; Orchestrator full `castor test` attempts repeatedly hit the known unrelated MessengerSqliteImmediateTransactionMiddlewareTest wall-clock threshold flake (4.39ms/86.37ms < 140ms); focused middleware PASS 4/19. No task code touches it.; castor deptrac PASS 0 violations; castor phpstan PASS 0 errors; castor cs-check clean; git diff --check clean; no stale workers.
- Summary: Code review complete at HEAD c59f441f4e7265010e6847088271b06bb39a55cd. Reviewer verdict APPROVED with no actionable findings: implementation is minimal/correct, cache and race semantics sound, test budget appropriate, no new service/API/async/profiling. Optional unbounded-cache note explicitly marked non-actionable and deferred as YAGNI. RED commit 5a0319dd4 proves unchanged allFor re-decode; GREEN c59f441f4 implements the localized cache.
- task-to-pr review: strict minimality reviewer approved c59f441f4; full diff remains exactly two files. Focused session-store proof green. Known baseline SQLite timing oracle flaked during orchestrator full-suite reruns; implementation fork already recorded a full green retry.

## Task workflow update - 2026-07-27T00:15:46.427Z
- Validation: Gate qa-20260727-001248-452614-1d7935de: sole failure ShellFollowUpLiveE2eTest::testShellThenFollowUpOnCompletedRun with events command.ack, run.completed; all other lanes passed.; Focused `castor test:llm-real --filter=ShellFollowUpLiveE2eTest` PASS 2 tests / 21 assertions.; No stale QA workers.
- Summary: First CODE-REVIEW gate failed only in unrelated live ShellFollowUpLiveE2eTest phase-boundary race: follow-up collector saw stale shell run.completed after command.ack and reported no assistant response. Session event-store memoization code is not on this test synchronization path. Focused live test immediately passed 2/21; no leaks. Retrying unchanged gate.

## Task workflow update - 2026-07-27T00:18:37.998Z
- Validation: Gate qa-20260727-001555-464620-b9d83eee: sole failure TuiTreeCommandE2eTest::testTreeEnterRewindsTranscriptToEarlierTurn; all other lanes passed.; Focused `castor test:tui --filter=TuiTreeCommandE2eTest` PASS 2 tests / 10 assertions.; No stale QA workers.
- Summary: Second CODE-REVIEW gate failed only known unrelated TuiTreeCommandE2eTest rewind-render flake ('abandoned bang command must disappear'). Focused replay-backed TUI test immediately passed 2/10; no leaks. Retrying unchanged gate; no unrelated test code will be added to this minimal two-file PR.

## Task workflow update - 2026-07-27T00:20:56.208Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (127.8s).
- Pushed task/memoize-session-event-reads-during-resume to origin.
- branch 'task/memoize-session-event-reads-during-resume' set up to track 'origin/task/memoize-session-event-reads-during-resume'.
- Created PR: https://github.com/ineersa/agent-core/pull/322

## Task workflow update - 2026-07-27T00:21:01.933Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/322
- Updated PR Status: open
- Validation: Final deterministic castor check PASS in 127.8s.; PR #322 open: https://github.com/ineersa/agent-core/pull/322.; Earlier gate attempts exposed only known unrelated ShellFollowUp and TuiTree flakes; both focused reruns passed and final unchanged gate passed.
- Summary: Task-to-PR complete at HEAD c59f441f4e7265010e6847088271b06bb39a55cd. Strict reviewer APPROVED; deterministic castor check passed in 127.8s; branch pushed; PR #322 created. Scope remains exactly two files and one private process-local event snapshot cache.

## Task workflow update - 2026-07-27T02:39:46.415Z
- Moved CODE-REVIEW → DONE.
- Merged task/memoize-session-event-reads-during-resume into integration checkout.
- Merge made by the 'ort' strategy.
 src/CodingAgent/Session/SessionRunEventStore.php   |  68 +++++++++-
 .../Session/SessionRunEventStoreTest.php           | 140 ++++++++++++++++-----
 2 files changed, 174 insertions(+), 34 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/memoize-session-event-reads-during-resume.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/memoize-session-event-reads-during-resume.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Pre-merge deterministic castor check PASS in 127.8s at c59f441f4e7265010e6847088271b06bb39a55cd.
- Summary: PR #322 merged on GitHub at 2026-07-27T02:39:24Z as merge commit 5becfee929e54984a36e5e7d891e506ba6b069b9. Final implementation remains the approved two-file localized process-memory cache.

## Task workflow update - 2026-07-27T02:42:15.822Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/322
- Updated PR Status: merged
- Validation: Post-merge `LLM_MODE=true castor check` PASS (qa-20260727-023950-637906-f3c0b2ef, 369.8s): unit 4523/15710, controller-replay 10/135, TUI 37/190, llm-real 13/175, deptrac/phpstan/cs all green.; Llama-proxy cache stable 180→180; artifact integrity PASS; QA leak check PASS.; Integration checkout clean (`main...origin/main [ahead 2]`); task worktree removed.
- Summary: DONE workflow complete: merged into integration checkout, remote sync run, task worktree removed, IDEA exclusions cleaned. Post-merge deterministic full validation is green.
