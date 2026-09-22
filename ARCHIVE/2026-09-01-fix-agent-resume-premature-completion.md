# Fix agent_resume completing before resumed child starts

## Goal
Live regression captured in session 1 under the Codex-timeout task worktree. `agent_resume` returned a completed handoff at 18:01:39 immediately after dispatching the child follow-up, while the resumed child actually ran from 18:01:39 through 18:02:26. Canonical evidence: parent seq 239 agent_resume start, seq 240 completed progress, seq 241 tool end; child artifact cursor advanced from old terminal seq 109 to seq 110 `agent_command_queued`, then deferred projection incorrectly became Completed because the after-turn committed RunState was still the pre-resume Completed state. Batch terminal completion was enqueued at 18:01:39 and later resumed child events seq 111-137 were ignored.

Root cause: resume rebind correctly preserves the old child event cursor and resets its lifecycle projection to Running, but `DeferredChildRunEventProjector::apply()` unconditionally overwrites event-derived Running with `committedStatus`. The first post-resume `agent_command_queued` commit retains the child's old Completed RunStatus until the follow-up is drained, so the deferred observer treats the resumed child as terminal before it starts.

Implement the smallest architecture-consistent fix with no polling/sleeps: a resumed child whose projection is Running must not be terminalized solely by the stale terminal committed status accompanying the wake command queue/apply boundary. It must remain pending until post-resume canonical evidence actually establishes terminal status. Preserve normal terminal projection and recovery behavior. No new setting, schema field, compatibility path, or test-only production API.

## Acceptance criteria
- `agent_resume` does not emit completed progress or tool result before the resumed child begins and reaches a new terminal event
- Old pre-resume terminal events remain excluded by the preserved child event cursor
- A post-resume wake command commit with stale Completed/Failed/Cancelled committed status keeps the rebound child lifecycle Running
- Subsequent resumed Running events advance normally; a new canonical terminal event completes/fails/cancels the batch exactly once
- Deterministic lowest-layer test reproduces the observed seq-109 terminal → seq-110 command-queued race without sleeps or timing windows
- Existing subagent launch, recovery, cancellation, and deferred lifecycle tests remain green; no schema or settings changes

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-09-01-fix-agent-resume-premature-completion
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-01-fix-agent-resume-premature-completion
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/450
PR Status: merged
Started: 2026-09-01T18:11:26.859Z
Completed: 2026-09-01T18:54:13.206Z

## Work log
- Created: 2026-09-01T18:11:05.059Z

## Task workflow update - 2026-09-01T18:11:26.859Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-01-fix-agent-resume-premature-completion.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-01-fix-agent-resume-premature-completion.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-01-fix-agent-resume-premature-completion.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-01-fix-agent-resume-premature-completion.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-01-fix-agent-resume-premature-completion.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-01-fix-agent-resume-premature-completion.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-01-fix-agent-resume-premature-completion/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-01-fix-agent-resume-premature-completion.
- Validation: Live parent session evidence: seq239 start, seq240 completed progress, seq241 tool end at 18:01:39; Live child evidence: old agent_end seq109 at 17:58:07; resume agent_command_queued seq110 at 18:01:38; resumed work seq111-137 through 18:02:26; SQLite evidence: resumed batch terminal_completion_enqueued_at=18:01:39, child_event_cursor=110, child projection status=completed
- Summary: Task started after direct source/session evidence established the race: resume rebind cursor 109 + Running projection; child seq110 agent_command_queued committed while canonical state remained Completed; unconditional committedStatus overlay changed projection back to Completed and terminalized batch before child seq111-137 resumed execution. Scope is a deterministic no-polling fix at the deferred child projection/observation seam with existing cursor and canonical event contracts.

## Task workflow update - 2026-09-01T18:12:24.146Z
- Summary: Routing pass complete in the dedicated worktree. Cohesive root-cause slice stays with main: `DeferredChildRunEventProjector` must recognize a canonical `agent_command_queued(kind=follow_up)` wake boundary and prevent the old terminal committed RunStatus from overriding the rebound Running projection for that commit. Subsequent `agent_command_applied` + `turn_advanced` commits carry Running normally; a new canonical terminal event still terminalizes. Deterministic projector test will reproduce live seq109 terminal cursor → seq110 follow-up queue with committedStatus Completed, then resumed Running and new terminal completion. Existing lifecycle tests retain parent pending/delivery proof. No schema, settings, polling, waits, or API change.
- Ownership: owner=main; fork_run=none; revision=30505f06a; scope=deferred child projector wake-boundary correction and deterministic regression test; outcome=assigned; commit=none

## Task workflow update - 2026-09-01T18:20:37.906Z
- Validation: Live evidence mapped exactly in regression test: old terminal cursor seq109, follow_up queued seq110 with stale terminal committed status, resumed applied/turn seq111-112, new terminal seq137; Focused Castor PASS: DeferredChildRunEventProjectorTest + DeferredSubagentBatchLifecycleTest + AgentResumeExecutionServiceTest, 38 tests/408 assertions; castor deptrac PASS: 0 violations; castor phpstan PASS: 0 errors; castor dead-code PASS: 0 errors; castor cs-check PASS; git diff --check PASS; IDE diagnostics clean for production and test files
- Summary: Implemented at commit 9354b49c5. DeferredChildRunEventProjector now recognizes `agent_command_queued(kind=follow_up)` as the resume wake boundary. When that first post-resume commit still reports the child's previous terminal RunStatus, the rebound projection remains Running instead of terminalizing the batch. The wake guard clears on the next recognized lifecycle event, so resumed Running progress and the new terminal event project normally. No polling, waits, schema, settings, or API changes.
- Ownership: owner=main; fork_run=none; revision=30505f06a; scope=deferred child projector wake-boundary correction and deterministic regression test; outcome=completed; commit=9354b49c5

## Task workflow update - 2026-09-01T18:46:54.401Z
- Validation: Reviewer: APPROVE WITH SUGGESTIONS at 9354b49c5; role=reviewer; artifact=inline pi-subagents read-only review; scope=origin/main...HEAD specification fidelity, correctness, edge cases, architecture, privacy/security, proof sufficiency; Focused Castor PASS at 9354b49c5: 38 tests/408 assertions; deptrac/phpstan/dead-code/cs-check/git diff --check PASS at 9354b49c5
- Summary: Independent read-only reviewer completed specification-fidelity and correctness review at exact HEAD 9354b49c5 with verdict APPROVE WITH SUGGESTIONS and no CRITICAL/BUG/SEC/SPEC blockers. Reviewer confirmed the wake boundary is production-correct, scoped only to follow_up, preserves cursor advancement, clears before genuine terminal evidence, adds no API/settings/schema/fallback, and has deterministic lowest-layer proof. Optional NTH suggestions (fixture turn initially 0, DRY flag clearing, comment naming agent_resume) are non-blocking and intentionally not expanded into this minimal bug fix.

## Task workflow update - 2026-09-01T18:48:34.862Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (78.5s).
- Pushed task/2026-09-01-fix-agent-resume-premature-completion to origin.
- branch 'task/2026-09-01-fix-agent-resume-premature-completion' set up to track 'origin/task/2026-09-01-fix-agent-resume-premature-completion'.
- Created PR: https://github.com/ineersa/agent-core/pull/450
- Validation: Independent reviewer APPROVE WITH SUGGESTIONS at 9354b49c5; no CRITICAL/BUG/SEC/SPEC blockers; Focused tests PASS: 38 tests/408 assertions; deptrac/phpstan/dead-code/cs-check/git diff --check PASS
- Summary: Ready for review at 9354b49c5. Fix keeps a resumed deferred child Running across the first follow_up queue commit when canonical RunState still carries the child's pre-resume terminal status; later resumed lifecycle evidence and the new terminal event project normally. Independent reviewer approved with no blockers.

## Task workflow update - 2026-09-01T18:54:13.206Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-01-fix-agent-resume-premature-completion: ide_close_project returned isError.
- Merged task/2026-09-01-fix-agent-resume-premature-completion into integration checkout.
- Merge made by the 'ort' strategy.
 .../Deferred/DeferredChildRunEventProjector.php    | 25 ++++++++-
 .../DeferredChildRunEventProjectorTest.php         | 63 ++++++++++++++++++++++
 2 files changed, 87 insertions(+), 1 deletion(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-01-fix-agent-resume-premature-completion.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-01-fix-agent-resume-premature-completion.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #450 state=MERGED at 2026-09-01T18:53:23Z; CODE-REVIEW transition castor check passed in 78.5s at 9354b49c5
- Summary: PR #450 merged on GitHub as 2b48f8bb37baf59d87fa7ce82ae16ddb6cc5d80d. Completing task and integrating the approved fix into the primary checkout.

## Task workflow update - 2026-09-01T18:56:29.505Z
- Validation: Post-merge castor check PASS: qa-20260901-185420-248638-02242684, 164.6s; unit 4663 tests/19069 assertions; controller-replay 6/88; TUI 8/60; llm-real 5/30; deptrac/phpstan/dead-code/cs-check/docs/catalog all PASS; JUnit max 6.508s; zero cases >10s; QA leak check PASS; llama-proxy entries 393→393
- Summary: Post-merge validation completed on integration checkout at e0bddc99a. LLM_MODE=true castor check passed all lanes in 164.6s; artifact integrity, leak guard, cache cleanup, and llama-proxy guard passed. JUnit audit found zero cases over 10s (max 6.508s). Integration checkout is clean and task worktree was removed.

## Task workflow update - 2026-09-06T15:40:56+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
