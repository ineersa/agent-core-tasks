# Make background-process shutdown proof deterministic

## Goal
Investigate post-merge failure of BackgroundProcessControllerSessionLifecycleListenerTest::shutdownEventCleansAcceptedState (assertProcessStopped sees PID alive after cleanup). User authorizes fixing the flake, moving proof to a lower layer if needed, or deleting it if deterministic proof is not possible. Do not raise timeouts, retry until green, or keep arbitrary sleep fixtures. Preserve public process lifecycle/safety behavior. Prior failing log: var/reports/qa-20260907-225633-16891-5482b9af/check-test.log. isAlive treats /proc presence as alive, including possible zombies, but root cause not yet proven.

## Acceptance criteria
- Determine failure mechanism with process-state evidence or deterministic reproduction.
- Fix product/test lifecycle at source, or replace/delete flaky process proof with documented lower-layer mapping.
- Keep process ownership and teardown deterministic; never signal active Hatfield or root-owned workers.
- Focused checks and relevant concurrent gate proof pass with cases under 10 seconds.

## Workflow metadata
Status: CANCELLED
Branch: task/2026-09-07-make-background-process-shutdown-proof-deterministic
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-07-make-background-process-shutdown-proof-deterministic
Fork run:
PR URL:
PR Status:
Started: 2026-09-07T23:02:43+00:00
Completed:

## Work log
- Created: 2026-09-07T23:02:13+00:00

## Task workflow update - 2026-09-07T23:02:43+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-07-make-background-process-shutdown-proof-deterministic.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-make-background-process-shutdown-proof-deterministic.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-07-make-background-process-shutdown-proof-deterministic.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-07-make-background-process-shutdown-proof-deterministic.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-07-make-background-process-shutdown-proof-deterministic.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-07-make-background-process-shutdown-proof-deterministic.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-07-make-background-process-shutdown-proof-deterministic/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-make-background-process-shutdown-proof-deterministic.

## Task workflow update - 2026-09-07T23:02:56+00:00
- Summary: Routing: listener cleanup calls manager stop then removes rows/sidecars; test fixture exec sleep 30 is detached, zero stop grace, final helper polls isAlive for 1s. isAlive counts /proc presence without zombie check; launch captures setsid PID and resolves PGID asynchronously. Unknown whether observed failure is zombie/reaping, cached stat, or launch/kill race. Delegate bounded process investigation and test ownership to one fork because meaningful OS/test iteration needed. Main reviews resulting diff/evidence.
- Ownership: owner=fork; fork_run=none; revision=12def7027; scope=background-process lifecycle flake investigation, deterministic replacement/fix and focused concurrent validation; outcome=assigned; commit=none

## Task workflow update - 2026-09-07T23:27:26+00:00
- Moved IN-PROGRESS → CANCELLED.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-07-make-background-process-shutdown-proof-deterministic: ide_close_project returned isError.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-make-background-process-shutdown-proof-deterministic.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-07-make-background-process-shutdown-proof-deterministic.
- Validation: Focused 31 tests/103 assertions passed. Full main gate qa-20260907-232623-36672-a52a1672 passed with 4855 unit/integration tests plus controller/TUI/live lanes.; Root cause repro from fork: old warm stat-cache check stayed alive 97/100 after kill, OS check after cache clear dead; fixed 0/100. Zombie false-positive also proven.
- Summary: User requested direct-main fix instead of tracked workflow. Implemented on main in fdf473bb3 and 5ec2dd28f; cancel only the redundant task/worktree, not the fix. Main replaced fork sleep/kill tests with owned pipe-driven child regressions. Independent reviewer agent_d044196ed1ff47cf approved. Full castor check on main passed all 10 lanes, leak and cache guards passed.
