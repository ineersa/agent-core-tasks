# Explore running castor run:agent inside bubblewrap via pi-bwrap

## Goal
Investigate whether `castor run:agent` can be launched inside the local Bubblewrap wrapper at `~/bin/pi-bwrap`.

Current wrapper characteristics observed:
- Uses `bwrap --die-with-parent --unshare-all --share-net`.
- Binds `/proc`, `/dev`, `/tmp`, `/run`.
- Binds system dirs read-only (`/usr`, `/bin`, `/lib`, `/etc`, etc.).
- Binds `$HOME` read-only, then makes `$HOME/projects`, `$HOME/claw`, `$HOME/.pi`, and `$HOME/.hatfield` writable.
- Preserves writable cwd when launched from a shared directory.
- Forwards `SSH_AUTH_SOCK` when present.

Initial feasibility guess: likely possible in principle for `castor run:agent` if all runtime writes stay under the project, `.hatfield`, `.pi`, `/tmp`, or `/run`; risks are Composer/cache writes outside shared dirs, tmux/PTY/runtime socket assumptions, process discovery across namespaces, and any expectation that child processes are visible in the host PID namespace.

## Acceptance criteria
- Document the exact command(s) tested, e.g. `~/bin/pi-bwrap castor run:agent ...`, without running destructive operations.
- Identify which directories/files `castor run:agent` needs writable access to, and whether the current `pi-bwrap` bind list is sufficient.
- Verify whether interactive TUI/TTY behavior works under Bubblewrap, including subprocess spawning and cleanup semantics.
- Verify whether task/session files, logs, caches, and temporary files land in intended writable locations and do not require broad writable `$HOME`.
- If it fails, record the exact blocker and propose the minimal `pi-bwrap` change or Hatfield/Castor setting needed.
- Add/update lightweight documentation or notes for how to run the agent under Bubblewrap if feasible.

## Workflow metadata
Status: DONE
Branch: task/explore-castor-run-agent-bubblewrap
Worktree: /home/ineersa/projects/agent-core-worktrees/explore-castor-run-agent-bubblewrap
Fork run: e3r37nvlbdr2
PR URL: https://github.com/ineersa/agent-core/pull/214
PR Status: merged
Started: 2026-06-25T17:30:42.840Z
Completed: 2026-06-25T19:21:58.488Z

## Work log
- Created: 2026-06-25T16:49:40.267Z

## Task workflow update - 2026-06-25T16:51:50.502Z
- Summary: User clarified the primary concern is tmux. The task should specifically investigate/remove tmux involvement from the `castor run:agent` path when running under Bubblewrap, and aim for a single command entrypoint rather than a multi-step tmux wrapper/workflow.
- Clarification: prioritize whether `castor run:agent` shells out through tmux or depends on tmux-managed panes/sessions; if so, design a no-tmux execution path suitable for `~/bin/pi-bwrap`.
- Clarification: desired UX is one command to launch the agent in Bubblewrap, not a sequence of manual setup commands. Candidate shape: `castor run:agent:bwrap` or an option/env-mode on `castor run:agent` that delegates through `~/bin/pi-bwrap` and avoids tmux.

## Task workflow update - 2026-06-25T16:52:31.290Z
- Summary: User further clarified the desired default behavior: any command path that launches `castor run:agent` should automatically use Bubblewrap when `~/bin/pi-bwrap` is available, rather than requiring users to remember a separate bwrap-specific command.
- Clarification: treat Bubblewrap as an auto-detected launcher wrapper for all `castor run:agent` invocations/aliases that start the agent, when `~/bin/pi-bwrap` exists and is executable.
- Design should avoid recursive wrapping if `castor run:agent` is already executing inside Bubblewrap; use an env sentinel or comparable guard.
- Acceptance should include preserving a bypass/diagnostic path if bwrap is unavailable or explicitly disabled, so local development and troubleshooting remain possible.

## Task workflow update - 2026-06-25T16:53:19.327Z
- Summary: Additional scope clarification: Bubblewrap auto-wrapping should cover Datadog/debug launch variants and test/smoke launch variants that ultimately start `castor run:agent`, not just the plain command. The task must explicitly determine whether tmux-based harness tests can run inside the current `~/bin/pi-bwrap` sandbox or require a host-side/no-bwrap path.
- Clarification: include Datadog launch commands/aliases in the audit. If a Datadog command launches the agent/runtime via `castor run:agent`, it should also auto-use `~/bin/pi-bwrap` when available unless explicitly bypassed.
- Clarification: include test/smoke commands that launch the agent/runtime via `castor run:agent`. Decide whether they should use the same bwrap wrapper by default or need explicit opt-out for tmux harness coverage.
- Question to resolve: can the current sandbox run the tmux harness? Validate empirically. It may not be impossible in principle, but the current wrapper uses `--unshare-all` and synthetic `/dev`, so tmux/PTY/devpts/server socket behavior and process cleanup must be proven rather than assumed.
- If tmux harness cannot run inside bwrap, document the minimal reason and provide a clear split: runtime agent launches use bwrap by default; tmux harness validation remains host-side or uses an explicit `HATFIELD_BWRAP=0`/similar bypass.

## Task workflow update - 2026-06-25T16:58:08.972Z
- Summary: Priority clarified: the important sandbox guarantee is read-only `$HOME` with only selected writable directories. Process namespace isolation is not a priority, so the design may relax/remove `--unshare-all` and/or use host `/dev` if needed for tmux, as long as the read-only-home boundary is preserved.
- Clarification: prefer a bwrap mode optimized for compatibility with tmux/TUI tests over process hiding. The non-negotiable requirement is that `$HOME` remains read-only except explicit writable binds such as `$HOME/projects`, `$HOME/claw`, `$HOME/.pi`, `$HOME/.hatfield`, plus tmp/runtime dirs.
- Implementation investigation should consider replacing `--unshare-all` with narrower/no namespace isolation and replacing synthetic `--dev /dev` with a host `/dev` bind if tmux/PTY requires it. Validate that those relaxations do not make the rest of `$HOME` writable.
- Acceptance update: include a proof command that attempts to write to an unshared `$HOME` path and confirms it fails, while writes under approved directories succeed.

## Task workflow update - 2026-06-25T17:30:42.840Z
- Moved TODO → IN-PROGRESS.
- Created branch task/explore-castor-run-agent-bubblewrap.
- Created worktree /home/ineersa/projects/agent-core-worktrees/explore-castor-run-agent-bubblewrap.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/explore-castor-run-agent-bubblewrap.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/explore-castor-run-agent-bubblewrap.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/explore-castor-run-agent-bubblewrap.
- Summary: Starting implementation/exploration per user request. Parent loaded task-workflow, castor, testing skill, and tests/AGENTS.md before dispatching implementation because this touches Castor/TUI/tmux validation.

## Task workflow update - 2026-06-25T17:31:22.976Z
- Recorded fork run: 8u6she97siun
- Launched implementation fork 8u6she97siun in worktree `/home/ineersa/projects/agent-core-worktrees/explore-castor-run-agent-bubblewrap`. Fork instructed to inspect Castor run-agent variants, implement shared bwrap auto-wrapper with recursion guard and bypass, empirically test readonly-home and tmux/bwrap behavior, update docs/notes, validate via Castor, commit changes, and report exact results.

## Task workflow update - 2026-06-25T18:08:39.945Z
- Validation: Fork reported: `castor cs-check` OK; `castor test --filter=PiBwrapCastorHelperTest` OK; `HATFIELD_BWRAP=0 castor test:tui --filter=TuiJourneyE2eTest` OK; manual readonly-home/project-write/detached-tmux bwrap probes OK.; Parent inspected worktree: `git status --short` clean; `git log --oneline --decorate -3` shows commit `673be91a2`; `git diff --stat main...HEAD` shows 6 files changed, 284 insertions, 82 deletions.; Parent reviewed `.castor/helpers.php`, `.castor/run.php`, `.castor/shared.php`, docs, and `PiBwrapCastorHelperTest.php`; identified the wrapper argv bug above.
- Summary: Fork 8u6she97siun completed and committed 673be91a2, but parent review found a likely blocker in the auto-wrap re-exec command shape before accepting the implementation: `maybe_reexec_castor_task_under_pi_bwrap()` passes the whole inner command as one argument to `~/bin/pi-bwrap`, while the observed wrapper ends with `bwrap ... -- "$@"` and does not run a shell. That means bwrap will likely try to execute a program literally named `env HATFIELD_INSIDE_PI_BWRAP=1 ...` instead of `env` with arguments. Auto-wrap was not end-to-end tested by the fork. A follow-up fork is needed to fix and prove the re-exec path.
- Follow-up required: change the re-exec invocation so `pi-bwrap` receives a real argv vector, e.g. `~/bin/pi-bwrap env HATFIELD_INSIDE_PI_BWRAP=1 <php> <castor.php> <task>` or `~/bin/pi-bwrap sh -lc '<command>'`, and add a test/proof that would have caught the one-arg command bug.
- Also note: fork handoff said testing skill and tests/AGENTS.md were read only partially. Before further QA/tests, the next fork must read both fully and mention that in handoff.

## Task workflow update - 2026-06-25T18:09:04.916Z
- Recorded fork run: z38lttfdsazg
- Launched follow-up fork z38lttfdsazg to fix the auto-wrap argv bug found in parent review. Fork instructed to make `pi-bwrap` receive a real argv vector, add coverage/proof that catches the previous one-arg command bug, run focused Castor validation, read testing docs fully, and commit on top of 673be91a2.

## Task workflow update - 2026-06-25T18:12:29.442Z
- Validation: Fork reported full prerequisite docs read before QA: task-workflow, castor, testing skill, and tests/AGENTS.md.; Fork reported `castor cs-check` OK and `castor test --filter=PiBwrapCastorHelperTest` OK (6 tests, 34 assertions).; Parent inspected worktree: `git status --short` clean; `git log --oneline --decorate -5` shows HEAD `480a18605`; `git diff --stat main...HEAD` shows the expected 6 changed files.; Parent reviewed `.castor/helpers.php` and `PiBwrapCastorHelperTest.php`: the old one-arg bwrap bug is fixed and covered.
- Summary: Follow-up fork z38lttfdsazg completed and committed 480a18605. Parent review confirms the argv-vector blocker is fixed: re-exec now builds separate tokens `[pi-bwrap, env, HATFIELD_INSIDE_PI_BWRAP=1, <castor CLI>, <task>]`, with tests covering the previous one-quoted-inner-command bug and an end-to-end stub wrapper proof. One minor documentation gap remains: new override `HATFIELD_CASTOR_EXECUTABLE` is used by the helper but not documented in the Bubblewrap docs/env table.
- Remaining polish before accepting implementation: document `HATFIELD_CASTOR_EXECUTABLE` in `docs/tui-testing.md` (and optionally Datadog docs if useful) because it is a real override used by the launcher. Also consider cleaning up the test temp directory from `PiBwrapCastorHelperTest` if easy, matching tests/AGENTS.md cleanup guidance.

## Task workflow update - 2026-06-25T18:12:43.310Z
- Recorded fork run: kzvkm55geq8m
- Launched polish fork kzvkm55geq8m to document `HATFIELD_CASTOR_EXECUTABLE`, optionally clean up `PiBwrapCastorHelperTest` temp directories, run focused Castor validation, and commit.

## Task workflow update - 2026-06-25T18:13:26.371Z
- Validation: Polish fork reported `castor cs-check` PASS.; Polish fork reported `castor test --filter=PiBwrapCastorHelperTest` PASS (6 tests, 34 assertions).; Parent inspected worktree after polish: `git status --short` clean; HEAD `26c856354`; diff stat main...HEAD shows 6 changed files, 481 insertions, 82 deletions.; Parent read `docs/tui-testing.md` Bubblewrap section and confirmed `HATFIELD_CASTOR_EXECUTABLE` is documented.; Parent read `PiBwrapCastorHelperTest.php` and confirmed stub temp dirs are tracked and removed in `tearDown()`.
- Summary: Implementation is complete for the task-start phase. Final commit stack is 673be91a2, 480a18605, and 26c856354. Parent review after the polish fork confirmed the docs include `HATFIELD_CASTOR_EXECUTABLE`, the test temp dirs are cleaned with `TestDirectoryIsolation::removeDirectory()`, and the worktree is clean. Per workflow, stopping here and not moving to CODE-REVIEW until the user requests task-to-pr.

## Task workflow update - 2026-06-25T18:19:49.307Z
- Validation: Parent ran actual `~/bin/pi-bwrap` smoke probes in worktree: readonly-home probe PASS (`READONLY_HOME_OK`), project write probe PASS (`PROJECT_WRITE_OK`), detached tmux probe PASS (`TMUX_DETACHED_OK`).; Parent ran actual auto-reexec probe with real `~/bin/pi-bwrap` and `HATFIELD_CASTOR_EXECUTABLE` pointing to a safe stub executable: `castor run:agent` exited with stub code 77 before tmux, stub observed `HATFIELD_INSIDE_PI_BWRAP=1`, cwd preserved as the task worktree, and argv `run:agent`. This proves the real bwrap wrapper executes the argv-vector reexec path. Interactive attach was not run.

## Task workflow update - 2026-06-25T18:20:36.923Z
- Summary: User performed a real interactive smoke and the agent successfully wrote `/home/ineersa/test.md` despite the intended read-only home boundary. Parent analysis: this strongly suggests the current implementation only sandboxes the Castor launcher/tmux client, not necessarily the TUI process. In particular, when `castor run:agent` is executed from inside an existing tmux session, the bwrapped Castor process calls `tmux new-window`, but the existing host tmux server creates the new pane and executes `php bin/console agent` on the host outside Bubblewrap. Therefore the home write succeeds. This is a task blocker: implementation does not satisfy the core read-only-home requirement for the actual agent process.
- New requirement from smoke: ensure the actual TUI/agent process is inside Bubblewrap, not just the Castor task process or tmux client. The implementation must account for tmux server semantics: commands passed to an existing host tmux server run outside the bwrap namespace.
- Likely fix direction: wrap the inner tmux command itself, e.g. the tmux pane should execute `~/bin/pi-bwrap env HATFIELD_INSIDE_PI_BWRAP=1 php bin/console agent ...`, or a dedicated helper script/argv-safe shell command, so the process created by the tmux server immediately enters bwrap before running the agent. Re-execing Castor alone is insufficient when using existing host tmux.
- Follow-up must add a manual/automated proof that attempts to write outside approved home dirs through the actual launched agent path fails, or at minimum a stubbed tmux/inner command proof showing the command executed by tmux starts with the bwrap wrapper.

## Task workflow update - 2026-06-25T18:22:04.844Z
- Summary: User clarified: do not add Bubblewrap-specific tests. Bubblewrap is optional and Hatfield should behave the same with or without it. User also wants to consider removing the `tmux new-window ... 'php bin/console agent'` launch shape rather than layering more bwrap/tmux complexity.
- Clarification: avoid adding more Bubblewrap-specific automated tests. Prefer manual smoke/proof notes for bwrap behavior; keep product behavior test coverage independent of whether bwrap is present.
- Clarification/question to implement: remove or replace `tmux new-window ... 'php bin/console agent'` for `run:agent*` so the actual agent process is not launched by a pre-existing host tmux server outside bwrap. Candidate direction: run/exec the agent directly in the current terminal after optional bwrap re-exec, or make tmux a separate explicit helper rather than the default path.
- Confirmed implication: TmuxHarness-based tests (`castor test:tui`) are not inside bwrap unless their inner command is explicitly wrapped. Current policy intentionally leaves `castor test:tui` on the host; bwrap remains an optional/manual launcher concern, not part of QA semantics.

## Task workflow update - 2026-06-25T18:25:19.488Z
- Summary: User clarified final direction: remove tmux from the default `run:agent` path instead of preserving it as a separate helper. Keep tmux only for the explicit agent test/manual TUI task (`run:agent-test` / "agent:test"), not for normal `run:agent`. Bubblewrap should remain optional; avoid Bubblewrap-specific automated tests and rely on manual smoke for the local wrapper.
- Implementation correction: `run:agent` should launch the TUI directly in the current terminal after optional bwrap re-exec, so the actual `php bin/console agent` process is inside bwrap when bwrap is enabled. This fixes the tmux-server escape observed by the user.
- Apply same direct-launch reasoning to `run:agent-capture` unless there is a strong reason not to; capture should not require tmux by default.
- Keep tmux only for the explicit manual test helper (`run:agent-test`; user called this `agent:test`). `castor test:tui` / TmuxHarness remain host-side and are not bwrap-specific.
- Remove Bubblewrap-specific automated tests added in this task if they make bwrap look like a product contract. Hatfield should behave the same with or without bwrap; local bwrap behavior is validated manually. Keep normal tests only for non-bwrap launch behavior if needed.

## Task workflow update - 2026-06-25T18:25:41.327Z
- Recorded fork run: c3es2frm1kof
- Launched correction fork c3es2frm1kof to remove tmux from default `run:agent`/`run:agent-capture`, keep tmux only for `run:agent-test`, remove bwrap-specific automated tests, update docs, run focused Castor validation, and commit.

## Task workflow update - 2026-06-25T18:53:04.972Z
- Validation: Correction fork reported `castor cs-check` PASS after `castor cs-fix --path=.castor/shared.php`.; Correction fork reported `castor test --filter=Castor` PASS (2 tests, 18 assertions).; Parent inspected worktree: `git status --short` clean; HEAD `cfdd8d419`; `git diff --stat main...HEAD` shows 5 files changed (test file removed), 321 insertions, 123 deletions.; Parent reviewed `.castor/run.php`: `run:agent` and `run:agent-capture` call `maybe_reexec_castor_task_under_pi_bwrap()` then `launch_agent_direct_terminal()`; `run:agent-test` no longer auto-wraps and remains tmux-only.; Parent reviewed `.castor/shared.php`: direct terminal helper uses `passthru('exec bash -lc ...')` and tmux helper remains for `run:agent-test`.; Parent reviewed `docs/tui-testing.md`: docs now state `run:agent` direct terminal, `run:agent-test` host tmux, `castor test:tui`/TmuxHarness host-side, and manual `~/test.md` write should fail under active bwrap.
- Summary: Correction fork c3es2frm1kof completed and committed cfdd8d419. Parent review confirms default `run:agent` and `run:agent-capture` now launch directly in the current terminal after optional bwrap re-exec, while `run:agent-test` remains the explicit host tmux helper with no auto-wrap. Bubblewrap-specific PHPUnit coverage was removed per user direction; docs now describe bwrap as optional local sandboxing/manual smoke only. Worktree is clean at HEAD cfdd8d419.
- Note: c3es2frm1kof handoff says testing docs were read partially rather than fully. Parent had already loaded the testing skill and tests/AGENTS.md fully earlier, reviewed the final changes, and confirmed no bwrap-specific automated tests remain and no test temp-dir conventions are relevant after deletion. Before CODE-REVIEW, reviewer/parent should still run the normal task-to-pr validation per workflow.

## Task workflow update - 2026-06-25T18:58:24.775Z
- Summary: User manually re-ran the interactive `castor run:agent` smoke after commit cfdd8d419 and reported: "Cool seems to work!" This confirms the previous `/home/ineersa/test.md` write bypass was fixed for normal `run:agent` after removing tmux from the default launch path.

## Task workflow update - 2026-06-25T19:07:08.318Z
- Recorded fork run: e3r37nvlbdr2
- Validation: Reviewer ran `castor phpstan` and it failed with exactly one error: `.castor/helpers.php:213` useless cast in `exit((int) $exitCode)`.; Reviewer ran `castor cs-check` PASS.; Reviewer ran `castor list` and confirmed `run:agent*` tasks register with updated descriptions.; Reviewer reported `php -l` on changed `.castor/*.php` had no syntax errors.
- Summary: Reviewer subagent requested changes before CODE-REVIEW. Blocker: `castor phpstan` fails on `.castor/helpers.php:213` with `cast.useless` because `exit((int) $exitCode)` casts an already-int `passthru` result. Launched fix fork e3r37nvlbdr2 to apply the one-line fix, run `castor phpstan` and `castor cs-check`, and commit.

## Task workflow update - 2026-06-25T19:07:44.054Z
- Validation: Fix fork reported `castor phpstan` PASS (`[OK] No errors`).; Fix fork reported `castor cs-check` PASS (0 fixable files).
- Summary: Fix fork e3r37nvlbdr2 completed and committed e766bddc3. The reviewer-blocking PHPStan `cast.useless` issue was fixed by changing `exit((int) $exitCode)` to `exit($exitCode)` in `.castor/helpers.php`. Runtime semantics unchanged.
- Next CODE-REVIEW step: re-run reviewer on HEAD e766bddc3, then run focused local validation before `move_task(to=CODE-REVIEW)`.

## Task workflow update - 2026-06-25T19:12:41.511Z
- Validation: Reviewer re-review: APPROVED; independently ran `castor phpstan` PASS and `castor cs-check` PASS, no remaining CODE-REVIEW blockers.; Parent ran `castor test` PASS: 3569 tests, 11350 assertions; PHAR smoke OK.; Parent ran `castor deptrac` PASS: violations=0, errors=0.; Parent ran `castor phpstan` PASS: errors=0, file_errors=0.; Parent ran `castor cs-check` PASS: files_fixed=0.; Parent ran `castor test:tui` PASS: 14 tests, 71 assertions.; User manually smoked `castor run:agent` after direct-launch fix and reported it seems to work.
- Summary: Reviewer re-review after fix commit e766bddc3 returned APPROVED. Parent ran focused pre-PR validation in the task worktree and all focused gates passed. Worktree status was clean before validation and no tracked changes appeared after validation.

## Task workflow update - 2026-06-25T19:13:46.392Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (45.8s).
- Pushed task/explore-castor-run-agent-bubblewrap to origin.
- branch 'task/explore-castor-run-agent-bubblewrap' set up to track 'origin/task/explore-castor-run-agent-bubblewrap'.
- Created PR: https://github.com/ineersa/agent-core/pull/214
- Validation: Reviewer re-review APPROVED.; User manual interactive smoke after direct-launch fix: `castor run:agent` seems to work, replacing the prior failed smoke where the agent could write `/home/ineersa/test.md`.; Focused validation in task worktree: `castor test` PASS (3569 tests, 11350 assertions; PHAR smoke OK).; Focused validation in task worktree: `castor deptrac` PASS (violations=0, errors=0).; Focused validation in task worktree: `castor phpstan` PASS (errors=0, file_errors=0).; Focused validation in task worktree: `castor cs-check` PASS (files_fixed=0).; Focused validation in task worktree: `castor test:tui` PASS (14 tests, 71 assertions).
- Summary: Moving to CODE-REVIEW after reviewer APPROVED and focused validation passed. Final branch HEAD is e766bddc3. Behavior: `castor run:agent` and `run:agent-capture` optionally re-exec Castor under `~/bin/pi-bwrap` when available and then launch the actual `php bin/console agent` directly in the current terminal, avoiding the host-tmux-server sandbox escape. `run:agent-test` remains the explicit host tmux helper and is not bwrap-wrapped. Bubblewrap-specific automated tests were removed per user direction; bwrap is documented as optional local sandboxing with manual smoke validation.

## Task workflow update - 2026-06-25T19:21:58.488Z
- Moved CODE-REVIEW → DONE.
- Merged task/explore-castor-run-agent-bubblewrap into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/helpers.php | 181 ++++++++++++++++++++++++++++++++++++++++++++++++++++
 .castor/run.php     | 123 +++++++++--------------------------
 .castor/shared.php  |  70 +++++++++++++++++++-
 docs/datadog.md     |   5 ++
 docs/tui-testing.md |  65 +++++++++++--------
 5 files changed, 321 insertions(+), 123 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/explore-castor-run-agent-bubblewrap.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/explore-castor-run-agent-bubblewrap.
- Pulled integration checkout: Already up to date..
- Validation: PR #214 was created from branch `task/explore-castor-run-agent-bubblewrap`.; CODE-REVIEW gate previously passed deterministic `castor check` during move to CODE-REVIEW.
- Summary: User reported PR #214 was merged. Moving task to DONE and merging/cleaning up task workflow state.
