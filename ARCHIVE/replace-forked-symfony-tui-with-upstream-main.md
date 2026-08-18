# Replace the forked Symfony TUI dependency with upstream main

## Goal
Symfony TUI issue https://github.com/ineersa/agent-core/issues/332 is now fixed upstream and merged into Symfony TUI main. Remove the temporary fork dependency and consume the upstream Symfony TUI repository/version instead.

Scope should first verify the exact upstream commit/release containing the required fix and inventory all Composer repository/version overrides, patches, lockfile entries, documentation, and CI/build assumptions tied to the fork. Use the smallest dependency-only migration; do not absorb unrelated Symfony TUI upgrades or compatibility work.

## Acceptance criteria
- Composer configuration no longer references the temporary Symfony TUI fork.
- The selected upstream Symfony TUI revision contains the fix tracked by agent-core issue #332.
- composer.lock resolves Symfony TUI from the canonical upstream source with no stale fork metadata.
- Existing application code requires no fork-specific compatibility path; any obsolete workaround directly tied to #332 is removed only if proven unnecessary.
- Focused TUI tests and the full deterministic `castor check` pass.
- Relevant dependency/setup documentation is updated if it currently names the fork.

## Workflow metadata
Status: ARCHIVE
Branch: task/replace-forked-symfony-tui-with-upstream-main
Worktree: /home/ineersa/projects/agent-core-worktrees/replace-forked-symfony-tui-with-upstream-main
Fork run: 2f3if1e22hwc
PR URL: https://github.com/ineersa/agent-core/pull/374
PR Status: merged
Started: 2026-08-12T22:17:23.058Z
Completed: 2026-08-13T00:32:14.053Z

## Work log
- Created: 2026-08-12T22:15:17.655Z

## Task workflow update - 2026-08-12T22:17:23.058Z
- Moved TODO → IN-PROGRESS.
- Created branch task/replace-forked-symfony-tui-with-upstream-main.
- Created worktree /home/ineersa/projects/agent-core-worktrees/replace-forked-symfony-tui-with-upstream-main.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/replace-forked-symfony-tui-with-upstream-main.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/replace-forked-symfony-tui-with-upstream-main.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/replace-forked-symfony-tui-with-upstream-main.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/replace-forked-symfony-tui-with-upstream-main.
- Summary: Starting dependency migration after user confirmed Symfony TUI issue #332 is fixed upstream on main. Scope remains dependency-only: verify exact upstream fix revision, remove temporary fork source/aliases, update lock metadata, remove only proven obsolete #332 workarounds, and validate TUI behavior.

## Task workflow update - 2026-08-12T22:34:16.963Z
- Recorded fork run: 2f3if1e22hwc
- Validation: composer validate --no-check-publish — valid (pre-existing warnings only).; composer install --dry-run --no-interaction — no install/update/remove operations.; castor test:tui — OK (35 tests, 237 assertions).; castor phpstan — OK, 0 errors (before lock-diff-only tightening; no source changes afterward).; castor cs-check — clean (before lock-diff-only tightening; no source changes afterward).; No ineersa/tui or fix-tui-markdown-inline-html remains in composer.json/composer.lock.
- Summary: Implemented minimal fork-to-upstream dependency swap and committed as f6c756d82bb523b2ae3dde508124a2ac2743fae6. Removed the ineersa/tui VCS repository and fork branch alias; composer.lock now pins canonical symfony/tui 8.1.x-dev at b490918497b19a388e5cac435667b66ea68ace4d. Tightened lock diff to preserve every unrelated package/version/metadata entry. Worktree clean.

## Task workflow update - 2026-08-13T00:23:10.923Z
- Validation: Reviewer verdict: APPROVED.; composer install --dry-run --no-interaction — no operations.; castor test — OK (4407 tests, 16549 assertions).; castor test:tui — OK (35 tests, 237 assertions).; castor deptrac — 0 violations/errors.; castor phpstan — 0 errors.; castor cs-check — clean.; Worktree clean at 0e8b250fd082b4f58996f62c5509bbc6965b177a.
- Summary: Task-to-PR review APPROVED at 0e8b250fd082b4f58996f62c5509bbc6965b177a. Reviewer confirmed all fork repository, branch alias, and root lock alias metadata is removed; only symfony/tui changes among locked packages; canonical upstream ref is internally consistent; scope is minimal. Full focused validation passed and worktree is clean.

## Task workflow update - 2026-08-13T00:25:24.048Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (122.9s).
- Pushed task/replace-forked-symfony-tui-with-upstream-main to origin.
- branch 'task/replace-forked-symfony-tui-with-upstream-main' set up to track 'origin/task/replace-forked-symfony-tui-with-upstream-main'.
- Created PR: https://github.com/ineersa/agent-core/pull/374

## Task workflow update - 2026-08-13T00:32:14.053Z
- Moved CODE-REVIEW → DONE.
- Merged task/replace-forked-symfony-tui-with-upstream-main into integration checkout.
- Merge made by the 'ort' strategy.
 composer.json |  6 +-----
 composer.lock | 44 ++++++++++++++++++++++++++++----------------
 2 files changed, 29 insertions(+), 21 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/replace-forked-symfony-tui-with-upstream-main.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/replace-forked-symfony-tui-with-upstream-main.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #374 merged on GitHub at 2026-08-13T00:31:55Z as 2d2c4b86166d6cc1f2a69912ad39bc2fa75d1d9c. Moving task to DONE, syncing integration checkout, and cleaning its worktree.

## Task workflow update - 2026-08-13T00:35:57.609Z
- Validation: PR #374 merged as 2d2c4b86166d6cc1f2a69912ad39bc2fa75d1d9c.; LLM_MODE=true castor check — quality OK: unit 4410/16559, controller replay 12/165, TUI 35/233, llm-real 13/144, deptrac/phpstan/cs all OK, artifact integrity/leak/cache guard OK.; git status — clean (main ahead of origin/main by local integration merges).; Task worktree removed.
- Summary: Post-merge integration validation passed. Initial castor check waited behind a sibling-worktree gate and timed out acquiring the shared lock; retry acquired it after 16.8s and completed successfully. Integration checkout is clean and task worktree is removed.

## Task workflow update - 2026-08-14T19:53:41+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
