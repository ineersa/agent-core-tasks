# Exclude immutable prompt context from observational-memory observations

## Goal
The observational-memory Observer reads canonical `run_started` events. Those events contain the full initial message list, including the system prompt and synthetic project context. `OmSourceBlockBuilder` currently renders every message without filtering by role, so the Observer can turn instructions into observations or reflections that later appear in OM replacement summaries.

Core compaction already treats leading `system` and `user-context` messages as an immutable prologue. OM must honor the same boundary when it builds Observer source blocks. This change concerns Observer input only. It must not alter the raw prologue or core compaction behavior.

Likely entry point: `.hatfield/extensions/observational-memory/src/Observer/OmSourceBlockBuilder.php`. Add the regression proof at the lowest correct extension test layer.

## Acceptance criteria
- When processing `run_started`, OM excludes messages with roles `system`, `developer`, and `user-context` from Observer source blocks.
- OM retains the real initial `user` message from the same `run_started` payload.
- Excluded instruction text cannot enter Observer chunk input or become an observation through the `run_started` source block path.
- Leading system and user-context messages remain raw and unchanged in the main conversation; core compaction behavior does not change.
- Deterministic extension tests cover a mixed initial message list and a list containing no observable conversation messages.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-09-01-exclude-project-context-from-om-observations
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-01-exclude-project-context-from-om-observations
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/451
PR Status: merged
Started: 2026-09-01T19:00:53.482Z
Completed: 2026-09-01T19:48:53.770Z

## Work log
- Created: 2026-09-01T17:51:56+00:00

## Task workflow update - 2026-09-01T19:00:53.482Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-01-exclude-project-context-from-om-observations.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-01-exclude-project-context-from-om-observations.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-01-exclude-project-context-from-om-observations.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-01-exclude-project-context-from-om-observations.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-01-exclude-project-context-from-om-observations.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-01-exclude-project-context-from-om-observations.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-01-exclude-project-context-from-om-observations/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-01-exclude-project-context-from-om-observations.
- Summary: Started task-start phase. Main will refresh routing in the task worktree, apply specification-fidelity checks, select implementation ownership, implement, and run focused validation. Full castor check is deferred to task-to-pr.

## Task workflow update - 2026-09-01T19:02:12.969Z
- Ownership: owner=main; fork_run=none; revision=e0bddc99a6684da9ad0ed2abd034b45e3f01f279; scope=filter immutable instruction roles from observational-memory run_started source blocks and add deterministic extension regressions; outcome=assigned; commit=none

## Task workflow update - 2026-09-01T19:05:05.479Z
- Validation: castor test --filter=ObserverChunkAndToolTest: OK (8 tests, 154 assertions, 0.279s PHPUnit time); castor test --suite=extensions: OK (140 tests, 871 assertions); castor cs-check: OK (0 files need fixes); castor phpstan --path=.hatfield/extensions/observational-memory: OK (0 errors); IDE diagnostics for changed source and test files: 0 problems; git diff --check: OK; Worktree clean after commit b0b4da340
- Summary: Implemented Observer-source filtering for run_started messages. OmSourceBlockBuilder now skips system, developer, and user-context roles while preserving real user messages. Added deterministic regressions for mixed initial messages, packed Observer input, and instruction-only run_started payloads. Core conversation and compaction code were not changed.
- Ownership: owner=main; fork_run=none; revision=e0bddc99a6684da9ad0ed2abd034b45e3f01f279; scope=filter immutable instruction roles from observational-memory run_started source blocks and add deterministic extension regressions; outcome=completed; commit=b0b4da340

## Task workflow update - 2026-09-01T19:13:47.026Z
- Summary: Independent reviewer approved revision b0b4da340 with suggestions only. No CRITICAL, BUG, or SEC blockers. Reviewer verified exact acceptance-criteria coverage, safe instruction-only handling through the existing empty-range fallback, no core compaction changes, and deterministic lowest-layer tests. Optional notes were to consider an explanatory invariant comment and remain aware that historical observations are not purged, both outside required changes.
- Review: role=reviewer; artifact=inline pi-subagents result; target_revision=b0b4da340; scope=full origin/main...HEAD diff, relevant OM pipeline/callers, test quality, security, and specification fidelity; decision=APPROVE WITH SUGGESTIONS; blockers=none

## Task workflow update - 2026-09-01T19:15:42.606Z
- Validation: castor test --filter=ObserverChunkAndToolTest: OK (8 tests, 154 assertions, 0.277s PHPUnit time); castor deptrac: OK (0 violations, 0 errors); castor phpstan: OK (0 errors); castor cs-check: OK (0 files need fixes); castor test:llm-real --filter=liveObserverCanRecordObservationsViaTwoToolCalls: OK (1 test, 8 assertions, 2.045s PHPUnit time); git status --short: clean; Reviewer target b0b4da340: APPROVE WITH SUGGESTIONS; no CRITICAL, BUG, or SEC blockers
- Summary: PR readiness validation completed at b0b4da340896b05ca0764ceb6b71fb73476ae65f. Independent review verdict: APPROVE WITH SUGGESTIONS, with no blockers. Optional notes do not require code changes. Worktree is clean and unresolved blockers: none.

## Task workflow update - 2026-09-01T19:18:00.283Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (124.0s).
- Pushed task/2026-09-01-exclude-project-context-from-om-observations to origin.
- branch 'task/2026-09-01-exclude-project-context-from-om-observations' set up to track 'origin/task/2026-09-01-exclude-project-context-from-om-observations'.
- Created PR: https://github.com/ineersa/agent-core/pull/451
- Validation: Independent reviewer: APPROVE WITH SUGGESTIONS; no CRITICAL, BUG, or SEC blockers; castor test --filter=ObserverChunkAndToolTest: OK; castor deptrac: OK; castor phpstan: OK; castor cs-check: OK; castor test:llm-real --filter=liveObserverCanRecordObservationsViaTwoToolCalls: OK; Worktree clean at b0b4da340896b05ca0764ceb6b71fb73476ae65f
- Summary: Prepared revision b0b4da340896b05ca0764ceb6b71fb73476ae65f for review. OmSourceBlockBuilder excludes system, developer, and user-context messages from run_started Observer source blocks while retaining the real user message. Deterministic tests cover mixed and instruction-only initial message lists. Independent reviewer approved with suggestions and no blockers.

## Task workflow update - 2026-09-01T19:48:53.770Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-01-exclude-project-context-from-om-observations: ide_close_project returned isError.
- Merged task/2026-09-01-exclude-project-context-from-om-observations into integration checkout.
- Merge made by the 'ort' strategy.
 .../src/Observer/OmSourceBlockBuilder.php          |  5 ++
 .../tests/ObserverChunkAndToolTest.php             | 76 ++++++++++++++++++++++
 2 files changed, 81 insertions(+)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-01-exclude-project-context-from-om-observations.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-01-exclude-project-context-from-om-observations.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #451 state: MERGED; Merge commit: bcea05b6e87bfcc5d3628b1bf97035aff72753fd
- Summary: PR #451 was merged on GitHub at 2026-09-01T19:25:17Z with merge commit bcea05b6e87bfcc5d3628b1bf97035aff72753fd. Moving the task to DONE and integrating the merged branch into the primary checkout.

## Task workflow update - 2026-09-01T19:50:46.317Z
- Updated PR Status: merged
- Validation: LLM_MODE=true castor check: OK (quality: ok, 161.5s); QA run qa-20260901-194910-296007-70688c5a: all 10 lanes passed; Unit/integration: OK (4665 tests, 19081 assertions); Controller replay: OK (6 tests, 88 assertions); TUI replay: OK (8 tests, 60 assertions); LLM real: OK (5 tests, 30 assertions); De ptrac, PHPStan, dead-code, CS, docs, and catalog version lanes: OK; QA artifact integrity, leak check, cache cleanup, and llama-proxy cache guard: OK; Integration git status: clean; Task worktree removed and absent from git worktree list
- Summary: DONE validation completed in the integration checkout. PR #451 is merged, the task branch is integrated, the primary checkout is clean, and the task worktree was removed. JetBrains project close reported a degraded IDE close response during cleanup, but filesystem and Git worktree cleanup completed successfully.

## Task workflow update - 2026-09-06T15:40:55+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
