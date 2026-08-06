# Auto-install extensions vendor on worktree creation

## Goal
When `move_task` transitions a task to IN-PROGRESS and creates a new worktree, the task-workflow extension should also run `composer install -d .hatfield/extensions` inside the new worktree so the Hatfield extensions (task-workflow, castor-llm-mode) are autoloadable from the start.

Currently, after cloning/pulling a worktree, the user must manually run:
```bash
composer install -d .hatfield/extensions
```

This should be automatic as part of the worktree setup in `WorktreeManager` or `MoveTaskHandler`.

## Acceptance criteria
- composer install -d .hatfield/extensions runs automatically inside new worktrees created by move_task → IN-PROGRESS
- Existing worktrees are not affected (idempotent)
- Errors from the composer command are handled gracefully (logged, not fatal)
- Tests cover the new behavior

## Workflow metadata
Status: DONE
Branch: task/2026-06-30-auto-install-extensions-vendor-on-worktree-creation
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-06-30-auto-install-extensions-vendor-on-worktree-creation
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/260
PR Status: merged
Started: 2026-07-04T21:45:59+00:00
Completed: 2026-07-04T22:17:53+00:00

## Work log
- Created: 2026-06-30T21:27:49+00:00

## Task workflow update - 2026-07-04T21:45:59+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-06-30-auto-install-extensions-vendor-on-worktree-creation.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-06-30-auto-install-extensions-vendor-on-worktree-creation.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-06-30-auto-install-extensions-vendor-on-worktree-creation.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-06-30-auto-install-extensions-vendor-on-worktree-creation.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-06-30-auto-install-extensions-vendor-on-worktree-creation.

## Task workflow update - 2026-07-04T21:56:44+00:00
- Validation: castor test: 4067 tests OK (25.5s); castor cs-check: 0 fixes needed; castor phpstan: no errors; castor deptrac: 0 violations
- Summary: Implemented auto-install of extensions vendor on worktree creation:
- Added `ExecInterface` dependency to `WorktreeManager`
- Added `installExtensionsVendor()` method that runs `composer install -d .hatfield/extensions` in the new worktree with graceful error handling (non-fatal)
- Added `extensionsVendorInstalled` field to `WorktreeCreateResult`
- Wired `ExecInterface` through `TaskWorkflowExtension`
- Added user-visible note in `MoveTaskHandler` when extensions vendor is installed
- Added 2 new tests: happy path (verifies composer install is called) and failure tolerance (verifies task still moves when composer fails)

## Task workflow update - 2026-07-04T22:02:37+00:00
- Amended: moved .hatfield/extensions/ setup to test setUp() instead of per-test git commits, removed duplicate mkdir calls in test methods, 4067 tests OK (zero warnings), cs-check/phpstan/deptrac clean

## Task workflow update - 2026-07-04T22:16:23+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (83.7s).
- Pushed task/2026-06-30-auto-install-extensions-vendor-on-worktree-creation to origin.
- branch 'task/2026-06-30-auto-install-extensions-vendor-on-worktree-creation' set up to track 'origin/task/2026-06-30-auto-install-extensions-vendor-on-worktree-creation'.
- Created PR: https://github.com/ineersa/agent-core/pull/260

## Task workflow update - 2026-07-04T22:17:53+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-06-30-auto-install-extensions-vendor-on-worktree-creation into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/extensions/task-workflow/src/TaskWorkflowExtension.php         |   2 +-
 .hatfield/extensions/task-workflow/src/Tool/MoveTaskHandler.php          |   3 +++
 .hatfield/extensions/task-workflow/src/Worktree/WorktreeCreateResult.php |   1 +
 .hatfield/extensions/task-workflow/src/Worktree/WorktreeManager.php      |  29 +++++++++++++++++++++++++++++
 .hatfield/extensions/task-workflow/tests/MoveTaskHandlerTest.php         | 117 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++---
 5 files changed, 148 insertions(+), 4 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-06-30-auto-install-extensions-vendor-on-worktree-creation.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-06-30-auto-install-extensions-vendor-on-worktree-creation.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-07-04T22:19:38+00:00
- Validation: post-merge castor test: 4068 tests OK (23.4s); post-merge castor cs-check: 0 fixes needed; post-merge castor phpstan: no errors; post-merge castor deptrac: 0 violations
