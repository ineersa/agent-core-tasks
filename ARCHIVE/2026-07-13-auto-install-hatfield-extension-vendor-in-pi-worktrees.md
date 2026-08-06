# Auto-install Hatfield extension vendor in Pi-created worktrees

## Goal

Port the worktree extension-dependency installation behavior to the Pi task-workflow extension. When Pi's `move_task` transitions a task from TODO to IN-PROGRESS and creates a new worktree, it should run:

```bash
composer install -d .hatfield/extensions
```

inside that worktree, so Hatfield project extensions are immediately autoloadable when the worktree is opened in Hatfield.

## Context

The existing DONE ticket `2026-06-30-auto-install-extensions-vendor-on-worktree-creation.md` implemented this only in the native Hatfield extension:

- `.hatfield/extensions/task-workflow/src/Worktree/WorktreeManager.php`
- `.hatfield/extensions/task-workflow/src/Tool/MoveTaskHandler.php`

The Pi counterpart was intentionally retained by the original extension-port task, but `.pi/extensions/task-workflow/worktrees.ts` still only copies the root `vendor/` and `.vera/` directories. It does not install `.hatfield/extensions/vendor/`.

This is a Pi-specific follow-up; do not remove or replace the existing Hatfield implementation.

## Acceptance criteria

- Pi `move_task` TODO → IN-PROGRESS runs `composer install -d .hatfield/extensions` in the newly created worktree when `.hatfield/extensions/composer.json` exists.
- The command is invoked with the new worktree as its working directory and non-interactive/no-progress options as appropriate for the Pi exec wrapper.
- Missing `.hatfield/extensions` or `composer.json` is handled as a no-op.
- Composer failures are non-fatal: the worktree is still returned and the task transition completes, with a useful user-visible note or diagnostic.
- Installation is idempotent and only applies to newly created worktrees; existing worktrees are not modified by this path.
- The worktree result/notes expose whether extension dependencies were installed, matching the existing Hatfield behavior where useful.
- Add focused Pi extension tests covering the successful install, missing package directory, and non-fatal failure paths. Use test-local fixtures/mocks for command execution; do not run Composer against the real task board or repository.
- Verify the existing Hatfield auto-install behavior remains unchanged.

## Likely implementation area

- `.pi/extensions/task-workflow/worktrees.ts`
- `.pi/extensions/task-workflow/index.ts`
- `.pi/extensions/task-workflow/types.ts`
- Relevant Pi task-workflow tests/fixtures, if present.

Inspect the Pi `exec.ts` API before choosing command arguments and failure handling. Preserve the existing vendor/.vera copy and IDEA exclusion behavior.

## Test thesis

Without the production change, a worktree claimed through Pi lacks `.hatfield/extensions/vendor/autoload.php`; the successful-install regression test should fail. Failure and missing-directory tests protect the requirement that worktree creation remains usable when Composer is unavailable or the project has no Hatfield extension package.

## Suggested validation

- Run the focused Pi task-workflow tests.
- Run the repository's applicable Castor test, static-analysis, and coding-style checks according to the testing skill.
- Manually inspect a temporary isolated worktree to confirm `.hatfield/extensions/vendor/autoload.php` is created.

## Workflow metadata

Status: DONE
Branch: task/2026-07-13-auto-install-hatfield-extension-vendor-in-pi-worktrees
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-13-auto-install-hatfield-extension-vendor-in-pi-worktrees
Fork run: gs0qq0b3h8hl
PR URL: https://github.com/ineersa/agent-core/pull/288
PR Status: merged
Started: 2026-07-14T15:51:25.777Z
Completed: 2026-07-14T17:11:36.005Z

## Work log

- Created: 2026-07-13

## Task workflow update - 2026-07-14T15:51:25.777Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-07-13-auto-install-hatfield-extension-vendor-in-pi-worktrees.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-07-13-auto-install-hatfield-extension-vendor-in-pi-worktrees.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-07-13-auto-install-hatfield-extension-vendor-in-pi-worktrees.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-07-13-auto-install-hatfield-extension-vendor-in-pi-worktrees.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-07-13-auto-install-hatfield-extension-vendor-in-pi-worktrees.
- Summary: Claimed for task-start implementation; preparing Pi task-workflow extension context and fork instructions.

## Task workflow update - 2026-07-14T15:59:24.462Z
- Summary: Context gathering complete. Pi TODO→IN-PROGRESS calls createWorktreeForTask() in .pi/extensions/task-workflow/worktrees.ts, then index.ts renders WorktreeCreateResult notes. Native Hatfield parity uses a non-fatal composer install, a result boolean, and a conditional success note. Pi exec wrapper signature is run(pi, command, args, cwd, signal?, timeoutMs?) with pi.exec options {cwd, signal, timeout}. No existing agent-core TypeScript test harness was found; implementation fork must add focused, isolated Pi extension tests and a Castor-runnable test entry without modifying external pi-mono.
- Loaded task-workflow, testing, and subagents skills; read tests/AGENTS.md and Pi extensions documentation.
- Two scouts mapped the Pi call flow and native Hatfield parity. Required production scope: .pi/extensions/task-workflow/worktrees.ts, types.ts, index.ts; preserve root vendor/.vera copies and IDEA exclusion logic.
- Test thesis: success must create .hatfield/extensions/vendor/autoload.php through mocked command execution; missing composer.json must make no composer call; composer non-zero/throw must not prevent worktree creation and must expose a useful diagnostic.

## Task workflow update - 2026-07-14T15:59:59.789Z
- Recorded fork run: b33j5zzx36xb
- Summary: Launched implementation fork b33j5zzx36xb in the task worktree. Scope includes Pi worktree install/result/notes, isolated mocked command-execution tests, a Castor-runnable focused Pi test entry, focused native parity validation, and a committed clean branch. Fork explicitly instructed not to run castor check, push, create a PR, or modify external pi-mono.

## Task workflow update - 2026-07-14T16:14:48.463Z
- Recorded fork run: wrx557407ro3
- Summary: Initial implementation fork b33j5zzx36xb committed production behavior as 80aafc4aa and reported focused tests green, but its committed Castor test harness depended on external pi-mono/tsx with a hardcoded home-path fallback and had rough duplicated test setup. Launched follow-up fork wrx557407ro3 to make the focused test self-contained (standard Node runtime, no npm/network/external checkout), clean test source, rerun focused Castor validations, and commit a second fixup. Production behavior is to remain unchanged.

## Task workflow update - 2026-07-14T16:42:40.480Z
- Recorded fork run: qrajc0sfpjxr
- Summary: Follow-up commit 144b8033b removed external pi-mono/tsx dependencies and made the branch clean, but its fork did not read mandatory tests/AGENTS.md and the review found two concrete gaps: the Node check accepted versions before TypeScript stripping was available/default, and tests did not assert signal propagation or the missing-composer.json guard. Launched final compliance fork qrajc0sfpjxr to read both mandatory testing documents, make Node >=22.6 stripping explicit, tighten tests to three focused contracts with variants, rerun focused Castor validation, and commit if needed.

## Task workflow update - 2026-07-14T16:43:20.720Z
- Summary: Compliance fork qrajc0sfpjxr ended before handoff/commit. Preserved its partial uncommitted change in `.castor/pi-task-workflow.php`: accurate Node >=22.6 parsing/diagnostic and explicit `--experimental-strip-types`. Test consolidation and focused validation were not completed; branch remains based at 144b8033b with that one modified file. Retrying narrowly rather than discarding partial work.

## Task workflow update - 2026-07-14T16:43:36.517Z
- Recorded fork run: cj0zmrakqez0
- Summary: Launched retry fork cj0zmrakqez0 to preserve interrupted Castor changes, complete three focused test contracts, read both mandatory testing documents, run focused Castor validations, and commit a clean final compliance change.

## Task workflow update - 2026-07-14T16:44:48.773Z
- Recorded fork run: cj0zmrakqez0
- Validation: castor test:pi-task-workflow — passed: 3 top-level tests, 0 failures; covers success/autoload + exact argv/cwd/signal/timeout, missing-dir and missing-composer guards, non-zero and thrown execution failures; castor test --filter=MoveTaskHandlerTest — passed: 7 tests, 36 assertions; native Hatfield behavior unchanged; castor cs-check — passed: 0 files to fix after focused castor cs-fix on .castor/pi-task-workflow.php; castor check intentionally not run during task-start, per user instruction
- Summary: Task-start implementation complete and verified on branch. Commits: 80aafc4aa (Pi worktree auto-install production behavior), 144b8033b (self-contained Node test harness), ca2b3a0bc (Node >=22.6 compliance and three focused contracts). Verified HEAD is ca2b3a0bc84d9644ee62f2feada926abb82c49d4, branch worktree is clean, and cumulative diff is 9 expected files (+424/-1). Behavior: new Pi-created worktrees run non-interactive Composer install for .hatfield/extensions only when composer.json exists; result and move_task notes expose success/failure; execution failures remain non-fatal with concise diagnostics. Existing native Hatfield production files are unchanged. Testing forks confirmed mandatory testing skill and tests/AGENTS.md conventions.

## Task workflow update - 2026-07-14T16:47:29.701Z
- Recorded fork run: gs0qq0b3h8hl
- Summary: User explicitly rejected the bespoke Pi test/Castor harness due to its 355-line overhead and requested its removal. Launched cleanup fork gs0qq0b3h8hl to delete all newly added test-runner files and Castor import, keep only the three production Pi extension files, avoid replacement tests/QA, and commit a clean non-destructive cleanup.

## Task workflow update - 2026-07-14T16:52:30.429Z
- Recorded fork run: gs0qq0b3h8hl
- Validation: Inspection only per explicit user direction: git status clean; cumulative diff 3 files +69/-1; harness reference search returned no matches; No tests or QA commands run after harness removal; Pi-side automated tests were explicitly removed at user request
- Summary: User-directed cleanup complete. Commit f4580f214195ffb3d07c0a95915491ea4b7879dc removes the entire bespoke Node/Castor Pi test harness and its castor.php import without rewriting history. Verified clean branch and cumulative diff against b133461e7: exactly 3 production files (`.pi/extensions/task-workflow/index.ts`, `types.ts`, `worktrees.ts`), 69 insertions/1 deletion. No `test:pi-task-workflow`, loader, or harness references remain. Production Composer auto-install behavior is unchanged.

## Task workflow update - 2026-07-14T17:06:35.847Z
- Summary: task-to-pr review started at clean branch tip f4580f214195ffb3d07c0a95915491ea4b7879dc. Reviewer verdict: APPROVED with no blockers. Reviewer confirmed exact Composer argv/cwd/signal/timeout, guarded no-op, non-fatal diagnostics, user-visible notes, no command injection risk, and preservation of existing worktree setup. Optional note only: duplicated 240-character diagnostic limit could be consolidated later; no change requested. Focused Castor validation is running before CODE-REVIEW transition.

## Task workflow update - 2026-07-14T17:07:27.888Z
- Validation: castor test — OK: 4390 tests, 14553 assertions (23.1s); castor deptrac — 0 violations, 0 errors; castor phpstan — 0 errors, 0 file errors; castor cs-check — 0 files to fix; Reviewer — APPROVED, no blocking issues
- Summary: Pre-PR review and focused validation complete at f4580f214195ffb3d07c0a95915491ea4b7879dc. Reviewer APPROVED with no blockers. Final cumulative diff is 3 production Pi extension files (+69/-1); bespoke tests remain removed per explicit user direction.

## Task workflow update - 2026-07-14T17:09:32.786Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (112.9s).
- Pushed task/2026-07-13-auto-install-hatfield-extension-vendor-in-pi-worktrees to origin.
- branch 'task/2026-07-13-auto-install-hatfield-extension-vendor-in-pi-worktrees' set up to track 'origin/task/2026-07-13-auto-install-hatfield-extension-vendor-in-pi-worktrees'.
- Created PR: https://github.com/ineersa/agent-core/pull/288
- Validation: castor test — 4390 tests, 14553 assertions passed; castor deptrac — 0 violations/errors; castor phpstan — 0 errors; castor cs-check — 0 files to fix; Independent reviewer — APPROVED
- Summary: Reviewer approved production-only diff at f4580f214. Pi worktree creation now installs `.hatfield/extensions` Composer dependencies when a manifest exists, reports success, and degrades non-fatally with concise diagnostics on command failure. Missing extension manifests are a no-op. Final diff is 3 files (+69/-1); bespoke Pi test harness was removed per user direction.

## Task workflow update - 2026-07-14T17:11:36.005Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-07-13-auto-install-hatfield-extension-vendor-in-pi-worktrees into integration checkout.
- Merge made by the 'ort' strategy.
 .pi/extensions/task-workflow/index.ts     |  4 ++
 .pi/extensions/task-workflow/types.ts     |  3 ++
 .pi/extensions/task-workflow/worktrees.ts | 63 ++++++++++++++++++++++++++++++-
 3 files changed, 69 insertions(+), 1 deletion(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-07-13-auto-install-hatfield-extension-vendor-in-pi-worktrees.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-07-13-auto-install-hatfield-extension-vendor-in-pi-worktrees.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #288 state verified as MERGED via gh pr view; Pre-merge integration checkout had no working-tree changes
- Summary: Confirmed GitHub PR #288 is MERGED at 2026-07-14T17:11:11Z with merge commit 9cd5da6a3d33ab8f992641088f116ae231df0928. Completing task workflow and synchronizing the integration checkout.

## Task workflow update - 2026-07-14T17:13:41.437Z
- Validation: LLM_MODE=true castor check — quality OK (266.8s); deptrac OK; unit/integration OK: 4386 tests, 14541 assertions; controller replay OK: 8 tests, 112 assertions; TUI replay OK: 36 tests, 186 assertions; llm-real OK: 10 tests, 122 assertions; phpstan OK: 0 errors; cs-check OK; llama-proxy cache guard stable: 327 → 327; QA artifact integrity OK: 7 lane logs; QA leak check OK: no processes for run qa-20260714-171139-161542-f772801a
- Summary: Post-merge validation complete. PR #288 is merged, task worktree and IDEA exclusions were removed, integration checkout is clean, and local main is synchronized through the remote PR merge (local merge/pull commits remain ahead of origin by 2).
