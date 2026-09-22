# Fix extension startup failure isolation

## Goal
## Problem

ExtensionManager promises to log per-extension lifecycle failures and continue loading remaining extensions. Its loadSingle() method constructs extensions and injects their logger outside the try block that guards register(). Subscriber attachment is also unguarded, after a successful load outcome has already been recorded. A throwing constructor can therefore abort console startup, including Messenger workers.

## Evidence

Architecture review: `.pi/reports/architecture-review-20260905-190304.html#extension-loading`, committed in `9f744008c`, based on source revision `24cd515bfd5d96992587b7b2337a0a9ff7b0cb0a`.

Entry points:
- `src/CodingAgent/Extension/ExtensionManager.php`, especially loadSingle() and its documented lifecycle contract.
- `src/CodingAgent/Extension/ExtensionLoaderSubscriber.php`, which loads extensions for console commands.
- `tests/CodingAgent/Extension/ExtensionManagerTest.php`.

## Scope

Make the existing manager own per-extension failure reporting across construction, logger injection, registration, and subscriber attachment. Preserve successful loading order, distinct missing-class and invalid-implementation diagnostics, and per-process idempotency. Record success only after the extension's loading steps succeed.

Keep the correction local. Do not add public interfaces, settings, a new loader framework, or transactional rollback of partial registrations. The existing registration path already permits partial registrations before failure; this task does not promise to undo them. Shared Composer autoloader failure recovery is outside scope.

## Validation

Follow the testing skill and tests/AGENTS.md. Use deterministic tests through the existing ExtensionManager test surface, starting with a constructor-throwing extension followed by a valid extension. Cover the other corrected lifecycle failure paths at the lowest correct layer. Run focused Castor validation during implementation; the CODE-REVIEW transition owns the full gate.

## Acceptance criteria
- A throwing extension constructor produces a logged failed load outcome without aborting loading of later enabled extensions.
- Per-extension logger injection, registration, and subscriber attachment failures follow the same documented continuation policy; failed attachment does not leave a successful load outcome.
- Existing missing-class and invalid-implementation diagnostics, successful registration order, TUI extension selection, and per-process idempotency remain correct.
- Deterministic regression tests prove failure reporting and successful loading of a later extension through the existing loader test surface.
- No new public interface or setting, no rollback framework, and no unrelated architecture changes are introduced.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-09-05-fix-extension-startup-failure-isolation
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-extension-startup-failure-isolation
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/487
PR Status: merged
Started: 2026-09-09T18:31:27+00:00
Completed: 2026-09-09T20:38:45+00:00

## Work log
- Created: 2026-09-06T01:40:40+00:00

## Task workflow update - 2026-09-09T18:31:27+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-05-fix-extension-startup-failure-isolation.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-extension-startup-failure-isolation.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-extension-startup-failure-isolation.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-extension-startup-failure-isolation.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-extension-startup-failure-isolation.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-extension-startup-failure-isolation/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-extension-startup-failure-isolation.

## Task workflow update - 2026-09-09T18:32:22+00:00
- Summary: Routing confirms one cohesive manager change; console subscriber needs no changes. Preserve existing diagnostics and partial registrations, defer success and TUI selection until subscriber attachment succeeds.
- Ownership: owner=main; fork_run=none; revision=c97f4f2bcf133b6402909a2b5dc791419c126de9; scope=local ExtensionManager lifecycle isolation and deterministic regression coverage; outcome=assigned; commit=none

## Task workflow update - 2026-09-09T18:35:34+00:00
- Validation: castor test --filter='ExtensionManagerTest|ExtensionLoaderSubscriberTest|ChildExtensionSelectionServiceTest': 21 tests, 103 assertions, passed in 0.092s PHPUnit time.; castor phpstan --path=src/CodingAgent/Extension/ExtensionManager.php: passed.; castor deptrac: zero violations.; castor cs-check and git diff --check: passed.
- Summary: Implemented local lifecycle isolation across construction, logger injection, registration, and subscriber attachment. Success and TUI selection happen only after attachment succeeds. Added deterministic coverage of all four failures, later successful loading, failed/successful TUI selection, and idempotent diagnostics. No public interfaces, settings, or rollback added. Committed 1338b9734. Ready for task-to-pr; full gate and independent review deferred to that phase.
- Ownership: owner=main; fork_run=none; revision=c97f4f2bcf133b6402909a2b5dc791419c126de9; scope=local ExtensionManager lifecycle isolation and deterministic regression coverage; outcome=completed; commit=1338b9734

## Task workflow update - 2026-09-09T20:29:03+00:00
- Summary: Review approved. Correcting validation description: 21 passing cases belong to ExtensionManagerTest and ChildExtensionSelectionServiceTest; ExtensionLoaderSubscriberTest in the filter does not exist.
- Review: role=reviewer; artifact=agent_c00dab443504a1d6; revision=1338b9734; scope=specification fidelity, lifecycle failure isolation, deterministic regression tests; verdict=APPROVE WITH SUGGESTIONS; blockers=none
- Ownership: owner=main; fork_run=none; revision=1338b9734; scope=restore host-dispatcher rationale comment from review; outcome=assigned; commit=none

## Task workflow update - 2026-09-09T20:30:26+00:00
- Summary: Final reviewer approval at 6325e5493. Reusing focused validation from 1338b9734 because follow-up changes only a comment. Proceeding to transition-owned full QA gate.
- Ownership: owner=main; fork_run=none; revision=1338b9734; scope=restore host-dispatcher rationale comment from review; outcome=completed; commit=6325e5493
- Review: role=reviewer; artifact=agent_c00dab443504a1d6; revision=6325e5493; scope=final specification-fidelity review plus comment-only follow-up; verdict=APPROVE; blockers=none

## Task workflow update - 2026-09-09T20:32:36+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (119.6s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-extension-startup-failure-isolation/var/reports/qa-20260909-203037-12995-d464cce3.
- Session/run: 29.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-09T20:32:38+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-05-fix-extension-startup-failure-isolation to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-extension-startup-failure-isolation/var/reports/qa-20260909-203037-12995-d464cce3.
- Session/run: 29.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-09T20:32:41+00:00
- castor check passed (119.6s).
- Pushed task/2026-09-05-fix-extension-startup-failure-isolation to origin.
- Created PR: <url>
- Session/run: 29.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-09T20:32:41+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (119.6s).
- Pushed task/2026-09-05-fix-extension-startup-failure-isolation to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/487

## Task workflow update - 2026-09-09T20:38:45+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-extension-startup-failure-isolation: ide_close_project returned isError.
- Merged task/2026-09-05-fix-extension-startup-failure-isolation into integration checkout.
- Merge made by the 'ort' strategy.
 src/CodingAgent/Extension/ExtensionManager.php       | 55 ++++++++++++++++++++++++++++---------------------------
 tests/CodingAgent/Extension/ExtensionManagerTest.php | 61 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 2 files changed, 89 insertions(+), 27 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-extension-startup-failure-isolation.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-fix-extension-startup-failure-isolation.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-09-09T20:39:57+00:00
- Updated PR Status: merged
- Validation: Post-merge castor check passed at integration revision 31a8c0aa89ccb718f3e39b27c91c0be2becf6845, 122.7s. Reports: var/reports/qa-20260909-203852-17879-6a110b00. All lanes passed; no QA worker leaks; proxy cache unchanged.; git status --short: clean. Task worktree directory absent.
- Summary: Confirmed GitHub PR #487 merged. DONE transition completed and task worktree removed. Integration checkout clean. IDE project close reported degradation, but filesystem cleanup succeeded.

## Task workflow update - 2026-09-10T22:49:39+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
