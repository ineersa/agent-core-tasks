# Launch agent by default and hide framework console commands

## Goal
Make installed/source `hatfield` behave like Castor's default-task UX: plain `hatfield` launches the `agent` command, `hatfield --help` shows agent help, and `hatfield list` presents a focused Hatfield command surface instead of every FrameworkBundle/Doctrine/Messenger command.

Research basis (JoliCode Castor v1.6.1): Castor calls Symfony Console `Application::setDefaultCommand($taskName)` with the one-argument form, never single-command mode; it keeps exact command dispatch and uses `Command::setHidden(true)` for internal commands. Symfony's second `true` argument would enable single-command mode and must not be used here because Hatfield relaunches `messenger:consume` and invokes Doctrine commands from QA.

Current Hatfield bootstrap is `bin/console`: FrameworkBundle Application auto-registers all bundle commands and currently defaults to `list`. Internal callers requiring exact commands include ConsumerSupervisor (`messenger:consume`), QA/test migrations (`doctrine:migrations:migrate`), controller relaunch (`agent --controller`), and Hatfield log/auth/completion commands.

Minimal intended implementation: configure default command `agent`; after command registration, mark non-user-facing commands hidden rather than unregistering/removing them. Keep user-facing Hatfield commands visible (`agent`, `auth:codex`, `completion:file-index:refresh`, `log:*`) plus Symfony console usability commands (`help`, `list`, `completion`). Hidden framework commands remain callable by exact name. Prefer a one-file bootstrap change unless a stable existing seam makes less code.

## Acceptance criteria
- Running `hatfield` with no command dispatches to `agent` and launches the normal TUI; explicit `hatfield agent ...` behavior remains unchanged.
- `hatfield --help` displays help for `agent` without launching it; use `setDefaultCommand('agent')` without Symfony single-command mode.
- `hatfield list` omits FrameworkBundle/Doctrine/Messenger/internal commands such as `cache:*`, `debug:*`, `doctrine:*`, and `messenger:*`; it retains Hatfield user-facing commands plus `help`, `list`, and `completion`.
- Hidden commands are not removed: exact invocations such as `hatfield messenger:consume ...` and `hatfield doctrine:migrations:migrate ...` remain resolvable for runtime and Castor/test internals.
- No command-call rewrites, custom routing framework, bundle removal, or new dependency; use Symfony Console's native default-command and hidden-command APIs.
- Add the smallest CLI-level regression proof that would catch the original UX: default help targets `agent`, normal list excludes representative framework commands while retaining Hatfield commands, and a representative hidden command remains directly callable (e.g. `messenger:consume --help`). Avoid implementation-mirroring tests.
- Update only concise CLI/README documentation if an existing command-usage statement becomes false; do not broaden into command refactors.
- Run focused Castor tests plus `castor deptrac`, `castor phpstan`, `castor cs-check`, and full `castor check` before PR because the default executable path/TUI startup is user-visible.

## Workflow metadata
Status: ARCHIVE
Branch: task/make-agent-default-hide-framework-console-commands
Worktree: /home/ineersa/projects/agent-core-worktrees/make-agent-default-hide-framework-console-commands
Fork run: pcj478tq5szy
PR URL: https://github.com/ineersa/agent-core/pull/334
PR Status: merged
Started: 2026-07-28T20:54:48.133Z
Completed: 2026-07-29T17:37:01.791Z

## Work log
- Created: 2026-07-28T20:43:32.716Z

## Task workflow update - 2026-07-28T20:54:48.133Z
- Moved TODO → IN-PROGRESS.
- Created branch task/make-agent-default-hide-framework-console-commands.
- Created worktree /home/ineersa/projects/agent-core-worktrees/make-agent-default-hide-framework-console-commands.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/make-agent-default-hide-framework-console-commands.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/make-agent-default-hide-framework-console-commands.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/make-agent-default-hide-framework-console-commands.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/make-agent-default-hide-framework-console-commands.
- Summary: Starting implementation after Castor v1.6.1 research confirmed Symfony-native pattern: one-argument setDefaultCommand plus hidden internal commands; no single-command mode or command removal.

## Task workflow update - 2026-07-28T20:57:40.804Z
- Recorded fork run: 3sj67k0l6uj2
- Summary: Implementation fork launched with Castor-matched minimum: one-argument default command, explicit focused visible allowlist, all other commands hidden but callable; one behavioral CLI regression proof and only a narrow existing TUI startup adjustment if needed.

## Task workflow update - 2026-07-28T21:02:01.800Z
- Recorded fork run: 3sj67k0l6uj2
- Validation: Manual prod console smoke PASS: exact 10-command visible surface, --help targets agent, messenger:consume --help remains callable; castor test --filter=ConsoleEntrypointUxTest PASS (2 tests, 23 assertions); castor test PASS (4417 tests, 15546 assertions); castor deptrac PASS (0 violations); castor phpstan PASS (0 errors); castor cs-check PASS (files_fixed=0); castor clean:cleanup:workers:list PASS (no stale workers); Worktree clean at eb0a76006
- Summary: Implementation complete at eb0a76006: `agent` is the one-argument Symfony default; console registration hides non-allowlisted framework/internal commands without removing them; focused subprocess regression proves default help, filtered list and exact hidden messenger invocation. 2 files, +140, clean worktree, not pushed.

## Task workflow update - 2026-07-28T21:06:45.740Z
- Summary: User requested a final help UX polish before PR: agent help should explain the default interactive behavior, include practical Hatfield examples, point to `hatfield list` and `hatfield help <command>`, and avoid Symfony's now-false global help wording about defaulting to list.

## Task workflow update - 2026-07-28T21:09:49.117Z
- Recorded fork run: x7s87srqbw81
- Validation: Manual rendered `hatfield --help` PASS with new description/examples/navigation; castor test --filter=ConsoleEntrypointUxTest PASS (2 tests, 28 assertions); castor phpstan PASS (0 errors); castor cs-check PASS (files_fixed=0); Worktree clean at a346ecf3e
- Summary: Help UX follow-up complete at a346ecf3e: friendly agent description, concise public `hatfield` launch/resume/cwd examples, pointers to `hatfield list` and `hatfield help <command>`, and corrected global -h wording. Existing behavioral test updated; no behavior/docs expansion.

## Task workflow update - 2026-07-28T21:36:51.218Z
- Recorded fork run: pcj478tq5szy
- Validation: Final reviewer APPROVE at 90ce1a692; specification fidelity gate passed; Ponytail: Lean already. Ship.; castor test PASS (4417 tests, 15550 assertions, 41.4s); castor deptrac PASS (0 violations); castor phpstan PASS (0 errors); castor cs-check PASS (files_fixed=0)
- Summary: Final HEAD 90ce1a692. Initial reviewer requested two ponytail simplifications (remove alias object-id dedupe; replace one help option by direct keyed assignment) and removal of redundant `about` assertion. Fix fork committed +9/-18. Final reviewer APPROVE: no correctness/security/spec-fidelity issues; native CLI subprocess is the lowest correct proof and no new TmuxHarness test is warranted because TUI implementation path is unchanged and both default help/no-arg dispatch use Symfony's same defaultCommand field.
- Review round: REQUEST CHANGES for 11 lines of unnecessary complexity; fork pcj478tq5szy committed cleanup as 90ce1a692 (+9/-18).
- Re-review current HEAD: APPROVE, no actionable findings. TUI proof explicitly adjudicated: existing CLI subprocess proof is sufficient; no new tmux journey needed for unchanged TUI code.

## Task workflow update - 2026-07-28T21:39:04.266Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (120.4s).
- Pushed task/make-agent-default-hide-framework-console-commands to origin.
- branch 'task/make-agent-default-hide-framework-console-commands' set up to track 'origin/task/make-agent-default-hide-framework-console-commands'.
- Created PR: https://github.com/ineersa/agent-core/pull/334
- Validation: Reviewer APPROVE; spec fidelity and TUI proof layer adjudicated; castor test PASS (4417 tests, 15550 assertions); castor deptrac PASS (0 violations); castor phpstan PASS (0 errors); castor cs-check PASS (files_fixed=0)
- Summary: Prepared for code review at 90ce1a692 after final reviewer APPROVE and focused Castor validation. Implements native Symfony default-agent dispatch, focused command listing, hidden-but-callable internal commands, and friendly public help.

## Task workflow update - 2026-07-28T21:39:12.534Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/334
- Updated PR Status: open
- Validation: move_task deterministic castor check PASS (120.4s); PR: https://github.com/ineersa/agent-core/pull/334
- Summary: PR #334 created after deterministic `castor check` passed in 120.4s; branch pushed at 90ce1a692.

## Task workflow update - 2026-07-29T17:37:01.791Z
- Moved CODE-REVIEW → DONE.
- Merged task/make-agent-default-hide-framework-console-commands into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/make-agent-default-hide-framework-console-commands.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/make-agent-default-hide-framework-console-commands.
- Pulled integration checkout: Already up to date..
- Validation: PR #334 state MERGED; Worktree clean and branch matches origin; Feature shipped in v0.0.4
- Summary: Found as a second orphaned clean worktree/task after PR #334 had already merged on 2026-07-28 as 06151ff603bb897ee4d0ab8ecd7d29ecd824672d. Marking DONE and cleaning worktree.

## Task workflow update - 2026-08-06T20:59:22.682Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
