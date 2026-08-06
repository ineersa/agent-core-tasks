# Reduce noisy debug logging in hot/cycled paths

## Goal
Context:
- Datadog is receiving debug logs despite config reportedly being INFO-level, and/or the application is emitting too many debug records.
- Observed volume is around 18k logs/minute, which is too noisy and likely dominated by cycled/hot actions such as lock retrieval and similar repeated lifecycle polling/coordination paths.
- Goal is not to remove useful observability, but to make hot-path logging intentional and low-volume.

Implementation notes:
- First verify effective logging configuration and why debug records reach Datadog when INFO is expected.
- Audit hot/cycled runtime paths that emit debug logs repeatedly (examples: lock retrieval/acquisition polling, repeated state polling, event loops, worker/runtime lifecycle loops).
- Remove or suppress repetitive debug logs from tight loops and frequently repeated no-op paths.
- Promote logs to INFO only when they are user/operator meaningful state transitions or rare lifecycle milestones.
- Preserve/adjust diagnostic logging for failures, degraded behavior, and actionable lifecycle transitions using structured event-style messages with correlation fields (`run_id`, `session_id`, `component`, `event_type`) and without raw prompts/tool output/secrets.
- Keep `docs/datadog.md` guidance in sync if logging policy/configuration changes.

## Acceptance criteria
- Effective Datadog/Monolog logging level behavior is understood and fixed or documented so DEBUG records are not shipped unexpectedly when INFO is configured.
- Repetitive debug logs in hot/cycled paths are removed, downgraded behind non-shipping diagnostics, or otherwise suppressed.
- Useful lifecycle milestones that operators need are logged at INFO with structured event-style fields; failures/degraded behavior remain visible at warning/error as appropriate.
- A before/after log-volume spot check or focused reproduction shows the noisy path is materially reduced from the reported ~18k/minute spam.
- Relevant focused Castor validation passes; run `castor check` if runtime/Messenger/TUI lifecycle code is touched, otherwise run the narrowest applicable Castor QA commands.

## Workflow metadata
Status: DONE
Branch: task/reduce-noisy-debug-logging
Worktree: /home/ineersa/projects/agent-core-worktrees/reduce-noisy-debug-logging
Fork run: r3gsr5rgq6vt
PR URL: https://github.com/ineersa/agent-core/pull/226
PR Status: merged
Started: 2026-06-27T00:29:54.719Z
Completed: 2026-06-27T01:16:27.583Z

## Work log
- Created: 2026-06-27T00:29:30.029Z

## Task workflow update - 2026-06-27T00:29:54.719Z
- Moved TODO → IN-PROGRESS.
- Created branch task/reduce-noisy-debug-logging.
- Created worktree /home/ineersa/projects/agent-core-worktrees/reduce-noisy-debug-logging.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/reduce-noisy-debug-logging.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/reduce-noisy-debug-logging.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/reduce-noisy-debug-logging.

## Task workflow update - 2026-06-27T00:30:28.590Z
- Recorded fork run: tjvuzj7n9rpt
- Started implementation fork tjvuzj7n9rpt in worktree /home/ineersa/projects/agent-core-worktrees/reduce-noisy-debug-logging with instructions to inspect Monolog/Datadog config, remove noisy hot-path debug logs, preserve useful structured lifecycle logging, validate via Castor, and commit changes without pushing/PR.

## Task workflow update - 2026-06-27T00:33:50.229Z
- Recorded fork run: tjvuzj7n9rpt
- Validation: Fork loaded .agents/skills/testing/SKILL.md and read tests/AGENTS.md before QA/tests.; castor test --filter='RunTracerTest|RunCommitLoggingTest|ExecutionWorkerTest' — PASS (13 tests, 67 assertions).; castor cs-check — PASS after castor cs-fix.; castor phpstan --path=src/AgentCore/Application/Pipeline/RunCommit.php — PASS.; Orchestrator verification: git status --short in worktree produced no output (clean); git log confirms HEAD 869b43d6e; git diff --stat origin/main...HEAD shows expected 14 changed files.
- Summary: Implementation fork completed and committed 869b43d6e on branch task/reduce-noisy-debug-logging. Root cause found: tracked .hatfield/settings.yaml overrode logging.level to debug even though defaults/config expect info. Hot-path noise reduced by restoring project logging level to info, removing per-event RunCommit event_store.appended INFO logs, demoting RunTracer start/finish span logs from INFO to DEBUG, and removing low-value DEBUG logs in tight/cycled or expected-noise paths including LlmStdoutPoller, run-state replay, worker terminal skips, incompatible session replay records, AI cost calculation, subagent tool passthrough, file mention index lock/VCS paths, and expected HTTP aborts. Added RunCommitLoggingTest to assert commit logging remains summary-only and does not emit per-event appended logs. Worktree inspected by orchestrator: clean status; diff vs origin/main is 14 files changed, 109 insertions, 143 deletions.

## Task workflow update - 2026-06-27T00:40:59.215Z
- Recorded fork run: cy0omr1jb3js
- CODE-REVIEW gate (castor check) failed on two lanes: phpstan (3 property.onlyWritten errors — previous commit 869b43d6e left unused $logger properties/params/imports in SubagentToolSetResolver, FileMentionIndexBuilder, AiCostCalculator) and test:tui (TuiStartupSnapshotTest::testRealTerminalBootsAndRendersHatfieldLogo — environmental tmux soft-wrap of long worktree path splitting e2e-startup/SKILL.md across lines). QA run dir: var/reports/qa-20260627-003541-517018-e7bdb95d. Launched fix fork cy0omr1jb3js to remove dead logger plumbing (+ empty if block in AiCostCalculator + $logger DI arg in services.yaml), strip \r/\n before the wrapped substring assertion, and re-validate with castor phpstan/cs-check/test:tui filter + logging tests.

## Task workflow update - 2026-06-27T00:41:40.809Z
- Recorded fork run: cy0omr1jb3js
- Validation: castor phpstan (full) — OK, 0 errors (3 property.onlyWritten resolved).; castor cs-check — PASS (0 fixable files).; castor test:tui --filter=testRealTerminalBootsAndRendersHatfieldLogo — PASS (1 test, 3 assertions).; castor test --filter='RunTracerTest|RunCommitLoggingTest|ExecutionWorkerTest' — PASS (13 tests, 67 assertions).; Orchestrator: git status clean; c1f776561 over 869b43d6e; fixup diff = 5 expected files.
- Summary: Gate-fix fork completed and committed c1f776561 on top of 869b43d6e. Removed dead $logger constructor params/imports and the empty if block from SubagentToolSetResolver, FileMentionIndexBuilder, AiCostCalculator; removed $logger DI arg from services.yaml SubagentToolSetResolver block. Normalized tmux capture (strip \\r/\\n) before the e2e-startup/SKILL.md substring assertion in TuiStartupSnapshotTest (scoped to that assertion; harness untouched). Orchestrator verification: worktree status clean; log shows HEAD c1f776561 over 869b43d6e; fixup diff stat is the expected 5 files, +2/-13.
- Gate-fix fork cy0omr1jb3js applied c1f776561: removed dead logger plumbing + normalized tmux wrap assertion. Focused Castor QA green (phpstan full OK, cs-check pass, test:tui filter pass, logging tests pass). Ready to re-attempt CODE-REVIEW gate.

## Task workflow update - 2026-06-27T00:44:20.352Z
- Recorded fork run: sajtkvn3qoka
- Second CODE-REVIEW gate attempt failed on the test lane (qa-20260627-004155-525325-bb73816d): CompletionFileIndexRefreshCommandTest::successWritesNoOutputAndReturnsSuccess errored 'Unknown named parameter $logger' (1 error of 910 tests). Root cause: the c1f776561 fix-up removed the $logger constructor params from FileMentionIndexBuilder and SubagentToolSetResolver but did NOT update the test call sites that still pass those args. Orchestrator grep enumerated all breakage sites: FileMentionIndexBuilderTest.php (11 sites), CompletionFileIndexRefreshCommandTest.php (1 site; keep the CompletionFileIndexRefreshCommand logger arg), SubagentToolSetResolverTest.php (5 sites + NullLogger import). AiCostCalculator tests already correct. Launched fix fork sajtkvn3qoka to update all test call sites to the new signatures, remove now-unused createLogger()/imports where fully unused, and validate with FULL castor test (the failing lane) + phpstan + cs-check.

## Task workflow update - 2026-06-27T00:45:34.576Z
- Recorded fork run: sajtkvn3qoka
- Validation: castor test (full) — OK (3692 tests, 11810 assertions) — the lane that failed in the gate.; castor phpstan (full) — OK, 0 errors.; castor cs-check — PASS (739 files; cs-fix needed 0 fixes).; Orchestrator: git status clean; 3 commits stacked correctly; grep sanity clean for logger:/NullLogger in the two affected test files.
- Summary: Test-signature fix-up fork completed and committed 53daf7539 on top of c1f776561. Mechanical test-only edits (no production changes) to align test call sites with the logger-removed constructors: removed stale logger/NullLogger args from FileMentionIndexBuilderTest.php (10 inline + 1 multiline), CompletionFileIndexRefreshCommandTest.php (1 multiline builder arg; kept the CompletionFileIndexRefreshCommand logger arg), and SubagentToolSetResolverTest.php (5 NullLogger args + import). Dropped now-unused createLogger()/LoggerInterface import from FileMentionIndexBuilderTest.php only (CompletionFileIndexRefreshCommandTest still uses createLogger for the command). Orchestrator verification: worktree clean; log shows 53daf7539 over c1f776561 over 869b43d6e; grep sanity confirms no leftover logger:/NullLogger test args.
- Test-signature fix-up 53daf7539 green on full castor test + phpstan + cs-check. Ready to re-attempt CODE-REVIEW gate.

## Task workflow update - 2026-06-27T00:46:49.063Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 600s)...
- castor check passed (57.7s).
- Pushed task/reduce-noisy-debug-logging to origin.
- branch 'task/reduce-noisy-debug-logging' set up to track 'origin/task/reduce-noisy-debug-logging'.
- Created PR: https://github.com/ineersa/agent-core/pull/226

## Task workflow update - 2026-06-27T01:07:25.314Z
- Moved CODE-REVIEW → IN-PROGRESS.

## Task workflow update - 2026-06-27T01:08:22.125Z
- Recorded fork run: r3gsr5rgq6vt
- Updated PR URL: https://github.com/ineersa/agent-core/pull/226
- Updated PR Status: open
- PR #226 review: 9 inline comments from ineersa — all 'over-deleted / should be more visible'. Reassessment confirmed fork masked real signals. Framework spam (Symfony Lock 'Successfully released', Messenger) verified all DEBUG in vendor source, so the .hatfield settings debug→info fix alone kills it; nothing to do there. Moved CODE-REVIEW→IN-PROGRESS (task-review-iterate). Launched corrective fork r3gsr5rgq6vt to restore/promote: SessionRunEventStore incompatible-schema→ERROR (silent event drop), LlmStdoutPoller unparseable-JSONL→WARNING (keep noise-line removed, keep threshold ERROR), WorkerFailedEventSubscriber skip→INFO, LlmPlatformAdapter abortConnection→INFO (+fix stale comment), CompletionFileIndexRefreshCommand x2→INFO, FileMentionIndexStartupListener→INFO, FileMentionIndexBuilder x2 VCS→INFO (re-adds $logger plumbing + test args), RunTracer revert→INFO. Stays removed: RunCommit per-event, poller non-RuntimeEvent noise line, AiCostCalculator, SubagentToolSetResolver passthrough, RunStateReplayService no_events. Fork validates with full castor test + phpstan + cs-check.

## Task workflow update - 2026-06-27T01:10:55.376Z
- Recorded fork run: r3gsr5rgq6vt
- Validation: castor cs-fix: 0 files fixed; castor cs-check: PASS (739 files); castor phpstan: 0 errors; castor test --filter='RunTracerTest|RunCommitLoggingTest|ExecutionWorkerTest': 13 tests OK; castor test (full): 3692 tests, 11810 assertions PASS; HEAD 4531feb45 over 53daf7539 over c1f776561 over 869b43d6e, clean tree
- Summary: PR #226 review iteration complete. Commit 4531feb45 on task/reduce-noisy-debug-logging (over 53daf7539). Restored all 9 reviewer-directed logs at promoted levels: SessionRunEventStore incompatible-schema→ERROR, LlmStdoutPoller unparseable-JSONL→WARNING (+threshold ERROR preserved, non-RuntimeEvent noise line stays removed), WorkerFailedEventSubscriber terminal-skip→INFO, LlmPlatformAdapter abortConnection→INFO (+stale comment updated), CompletionFileIndexRefreshCommand refresh_completed+refresh_lock_held→INFO, FileMentionIndexStartupListener startup_lock_held→INFO, FileMentionIndexBuilder VCS catch+else→INFO (re-added $logger ctor param/import/docblock; logger autowires, no services.yaml change), RunTracer trace.start/finish reverted→INFO. Test ctors updated: FileMentionIndexBuilderTest (re-added LoggerInterface import + createLogger() helper + logger: args on all 11 sites), CompletionFileIndexRefreshCommandTest (logger: arg on multiline builder). 10 files changed (+89/-12). Verified clean tree, all keep-removed files untouched (RunCommit per-event, AiCostCalculator, SubagentToolSetResolver passthrough, RunStateReplayService no_events, settings.yaml, services.yaml). Validation: full castor test 3692 tests/11810 assertions PASS, phpstan 0 errors, cs-check PASS. TUI lane NOT run by fork (parent runs on CODE-REVIEW move; path-wrap fix c1f776561 in branch history).

## Task workflow update - 2026-06-27T01:12:06.281Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (62.0s).
- Pushed task/reduce-noisy-debug-logging to origin.
- branch 'task/reduce-noisy-debug-logging' set up to track 'origin/task/reduce-noisy-debug-logging'.
- PR already exists: https://github.com/ineersa/agent-core/pull/226

## Task workflow update - 2026-06-27T01:12:31.638Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/226
- Updated PR Status: open
- Validation: castor check (deterministic gate): PASSED in 62.0s; branch task/reduce-noisy-debug-logging pushed to origin; PR #226 updated with comment addressing all 9 review threads (issuecomment-4814491710)
- Second CODE-REVIEW move succeeded: deterministic castor check gate PASSED (62.0s), branch pushed, PR #226 already existed. Posted summary comment addressing all 9 inline review threads (commit 4531feb45). Task ready for reviewer approval → DONE.

## Task workflow update - 2026-06-27T01:16:27.583Z
- Moved CODE-REVIEW → DONE.
- Merged task/reduce-noisy-debug-logging into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/settings.yaml                            |   2 +-
 config/services.yaml                               |   1 -
 .../Application/Handler/RunStateReplayService.php  |   5 -
 src/AgentCore/Application/Pipeline/RunCommit.php   |  61 +-----------
 .../Messenger/WorkerFailedEventSubscriber.php      |   4 +-
 .../SymfonyAi/LlmPlatformAdapter.php               |   6 +-
 .../Agent/Execution/SubagentToolSetResolver.php    |   7 --
 .../CLI/CompletionFileIndexRefreshCommand.php      |   4 +-
 src/CodingAgent/CLI/FileMentionIndexBuilder.php    |   4 +-
 .../CLI/FileMentionIndexStartupListener.php        |   2 +-
 src/CodingAgent/Config/Ai/AiCostCalculator.php     |  14 ---
 .../Runtime/Controller/LlmStdoutPoller.php         |  12 +--
 src/CodingAgent/Session/SessionRunEventStore.php   |   6 +-
 .../Application/Pipeline/RunCommitLoggingTest.php  | 103 +++++++++++++++++++++
 .../Execution/SubagentToolSetResolverTest.php      |  11 +--
 tests/Tui/E2E/TuiStartupSnapshotTest.php           |   3 +-
 16 files changed, 134 insertions(+), 111 deletions(-)
 create mode 100644 tests/AgentCore/Application/Pipeline/RunCommitLoggingTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/reduce-noisy-debug-logging.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/reduce-noisy-debug-logging.
- Pulled integration checkout: Merge made by the 'ort' strategy..
