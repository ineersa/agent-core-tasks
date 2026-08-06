# Fix release Node 24 action pins and native consumer shutdown leak

## Goal
Follow-up from merged PHAR/static distribution task and failed v0.0.1 release run 30372889350. Release evidence: PHAR and 3/4 static targets passed; linux-arm64 failed because extension_agent was the last of 13 consumers and survived after controller shutdown. Root cause: ConsumerSupervisor stops children sequentially with a per-child 5s grace while parent teardown kills controller sooner; HATFIELD_LLM_WORKER_COUNT smoke override is unused. Also release.yml pins Node20 actions and GitHub warns while forcing Node24. The v0.0.1 GitHub release exists with zero assets; publish job was skipped.

## Acceptance criteria
- Upgrade release.yml immutable pins to verified Node24 action releases: checkout v7.0.1, upload-artifact v7.0.1, download-artifact v8.0.1, cache v6.1.0; retain setup-php/action-gh-release pins already on Node24.
- ConsumerSupervisor shutdown signals all tracked consumers before one shared grace deadline, then escalates only survivors; no per-child cumulative grace and no owned orphan.
- Controller parent teardown timeout is aligned so it cannot kill the controller before ConsumerSupervisor's shared grace/escalation completes, including session-change teardown.
- Native topology release smoke retains hard leak assertion and cleans only pre-captured owned survivors after recording failure; never touches unrelated/root/session workers.
- Add one focused regression proof that would fail under sequential shutdown and pass under simultaneous TERM/shared deadline; no duplicated process-management implementation.
- Release workflow remains v* tag-only; all checksum, canonical PHAR handoff, four-target topology, and publish gates remain unchanged.
- Run focused Castor validation, reviewer, full deterministic Castor gate, and open a new PR from current main.

## Workflow metadata
Status: DONE
Branch: task/fix-release-node24-actions-consumer-shutdown-leak
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-release-node24-actions-consumer-shutdown-leak
Fork run: h6qazt1ncgtq
PR URL: https://github.com/ineersa/agent-core/pull/330
PR Status: merged
Started: 2026-07-28T16:04:07.320Z
Completed: 2026-07-28T18:33:27.023Z

## Work log
- Created: 2026-07-28T16:03:53.110Z

## Task workflow update - 2026-07-28T16:04:07.320Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-release-node24-actions-consumer-shutdown-leak.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-release-node24-actions-consumer-shutdown-leak.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-release-node24-actions-consumer-shutdown-leak.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-release-node24-actions-consumer-shutdown-leak.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-release-node24-actions-consumer-shutdown-leak.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-release-node24-actions-consumer-shutdown-leak.
- Validation: Release run 30372889350 inspected: linux-arm64 alone failed; exact extension_agent PID survived unchanged >10s; Node24 action pins verified: checkout 3d3c42e5 (v7.0.1), upload 043fb46d (v7.0.1), download 3e5f45b2 (v8.0.1), cache 55cc8345 (v6.1.0)
- Summary: Starting root-cause fix from failed v0.0.1 release. Research verified Node24 action SHAs; scout traced leak to sequential per-consumer shutdown and shorter parent timeout.

## Task workflow update - 2026-07-28T16:14:08.504Z
- Recorded fork run: uxplts57ynyc
- Validation: Fork confirmed testing skill + tests/AGENTS conventions read; castor test --filter=ConsumerSupervisorTest PASS: 5 tests, 40 assertions; castor test --filter='JsonlProcessAgentSessionClient|JsonlProcess' PASS: 24 tests, 109 assertions; castor test:controller-replay PASS: 10 tests, 135 assertions; castor deptrac PASS: 0 violations; castor phpstan PASS: 0 errors; castor cs-check PASS: files_fixed=0; No stale QA workers; Worktree clean at 8f69751e11efd2445636f9d4405667187bad5e3a
- Summary: Implemented root-cause fix at 8f69751e1 (5 files +279/-32): upgraded all release official action pins to verified Node24 versions; ConsumerSupervisor now TERM-signals all tracked consumers before one shared 5s grace and escalates only survivors; production/controller release parent waits aligned to 7s; topology gate cleans only unchanged pre-captured owned survivors then still fails; one real-subprocess regression proves simultaneous signaling.

## Task workflow update - 2026-07-28T17:13:50.734Z
- Recorded fork run: h6qazt1ncgtq
- Validation: castor test PASS: 4415 tests, 15516 assertions; castor test:controller-replay PASS: 10 tests, 135 assertions; castor deptrac PASS: 0 violations; castor phpstan PASS: 0 errors; castor cs-check PASS: files_fixed=0; No stale QA workers; Final reviewer APPROVE at 064318f3b
- Summary: Final review iteration complete at 064318f3b: removed redundant test teardown loop; final reviewer APPROVE, Ponytail 'Lean already. Ship.' Ready for PR.

## Task workflow update - 2026-07-28T17:16:04.180Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (120.8s).
- Pushed task/fix-release-node24-actions-consumer-shutdown-leak to origin.
- branch 'task/fix-release-node24-actions-consumer-shutdown-leak' set up to track 'origin/task/fix-release-node24-actions-consumer-shutdown-leak'.
- Created PR: https://github.com/ineersa/agent-core/pull/330
- Validation: castor test PASS: 4415 tests, 15516 assertions; castor test:controller-replay PASS: 10 tests, 135 assertions; castor deptrac PASS: 0 violations; castor phpstan PASS: 0 errors; castor cs-check PASS; Final reviewer APPROVE; No stale QA workers
- Summary: Open follow-up PR with Node24 action pins and root lifecycle fix for v0.0.1 linux-arm64 orphan. Final reviewer APPROVE.

## Task workflow update - 2026-07-28T18:33:27.023Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-release-node24-actions-consumer-shutdown-leak into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/distribution.php                           |  60 +++++++++-
 .github/workflows/release.yml                      |  16 +--
 .../Runtime/Controller/ConsumerSupervisor.php      |  72 ++++++++++--
 .../Process/JsonlProcessAgentSessionClient.php     |  32 +++---
 .../Runtime/Controller/ConsumerSupervisorTest.php  | 121 +++++++++++++++++++++
 5 files changed, 269 insertions(+), 32 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-release-node24-actions-consumer-shutdown-leak.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-release-node24-actions-consumer-shutdown-leak.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #330 merged 2026-07-28T18:06:58Z; Pre-merge deterministic castor check passed; v0.0.2 release in progress
- Summary: PR #330 merged by user as 2068485eb. v0.0.2 release matrix currently exercising the fix.
