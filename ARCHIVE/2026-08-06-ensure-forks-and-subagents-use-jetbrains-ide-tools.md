# Ensure forks and subagents use JetBrains IDE tools

## Goal
Forks and subagents appear not to use JetBrains MCP tools during implementation or code review. Verify whether unavailable/unopened project context is the cause, then fix the delegated-agent instructions and setup path.

Use the current tool name `ide_open_project` (rather than the older `jetbrains-index_ide_open_project` name). Instructions should require checking/opening the relevant checkout before task work when IDE code intelligence is unavailable, and emphasize semantic IDE navigation, impact analysis, diagnostics, and refactors over raw filesystem search where applicable.

## Acceptance criteria
- Determine and document why delegated forks/subagents currently skip IDE tools.
- Task implementation and code-review instructions explicitly prioritize appropriate IDE tools.
- When the relevant checkout is not open or IDE code intelligence is unavailable, delegated agents use `ide_open_project` before code work.
- Instructions account for sibling task worktrees and avoid opening the wrong checkout.
- Do not require IDE tools for operations they do not support or when unavailable; document the fallback.
- Add the smallest appropriate proof that delegated instructions include the required IDE setup/use guidance.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools
Fork run: 9zxwswic67gk
PR URL: https://github.com/ineersa/agent-core/pull/379
PR Status: merged
Started: 2026-08-13T14:48:59.769Z
Completed: 2026-08-14T20:48:33.929Z

## Work log
- Created: 2026-08-06T22:03:30.429Z

## Task workflow update - 2026-08-13T14:48:59.769Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools.
- Summary: Scope finalized: task-workflow will programmatically manage JetBrains project lifecycle through a small one-shot MCP client that resolves the runtime-specific project mcp.json server definition. Preserve Hatfield/Pi tool naming differences; open exact task worktree after creation and close before cleanup; tighten delegated implementation/review checkout scoping. No automated tests per user direction.

## Task workflow update - 2026-08-13T15:11:39.276Z
- Recorded fork run: f47im325n856
- Opened exact task worktree in JetBrains and waited for readiness before implementation.
- Implementation fork f47im325n856 launched to complete and commit the Pi/Hatfield programmatic MCP lifecycle work.

## Task workflow update - 2026-08-13T15:37:25.581Z
- Recorded fork run: f47im325n856
- Validation: Read .agents/skills/testing/SKILL.md and tests/AGENTS.md before validation.; php -l on touched PHP files: clean.; Pi TypeScript module import smoke via jiti: passed.; Manual Pi/PHP IDEA setup and MCP open smoke: passed.; Manual ide_file_structure scoped to exact worktree: passed.; castor deptrac: 0 violations.; castor phpstan --path=.hatfield/extensions/task-workflow/src: 0 errors.; castor cs-check --path=.hatfield/extensions/task-workflow/src: clean.; Parent verification: worktree clean, HEAD=e2ae19449; IDE diagnostics on WorktreeManager.php report 0 errors.
- Summary: Implemented and committed aligned Pi/Hatfield programmatic JetBrains lifecycle: minimal worktree .idea setup, runtime-specific one-shot MCP clients, exact worktree open on claim, close before safe DONE/CANCELLED removal with best-effort reopen, explicit degradation notes, and exact-project delegated guidance. Commit e2ae19449bed7aa936031080666466931f5cec17.

## Task workflow update - 2026-08-13T15:47:56.792Z
- Summary: Reviewer assessed commit e2ae19449 as approved with two concrete pre-PR suggestions: repair JetBrains heading placement that split orchestrator-role prose in six prompt files, and align Hatfield close-call timeout with Pi rather than using the open timeout. External surface fully mapped; no critical/security blockers.
- Reviewer read AGENTS.md, testing skill, and tests/AGENTS.md; inspected both ports, cleanup callers, SDK contracts, and live MCP schemas.
- Review external-surface inventory found no unmapped setting, dependency, runtime API, or command.

## Task workflow update - 2026-08-13T15:53:54.049Z
- Recorded fork run: 0s297xxp6sba
- Validation: Fix fork: php -l JetBrainsMcpClient.php passed.; Fix fork: castor phpstan --path=.hatfield/extensions/task-workflow/src/Ide/JetBrainsMcpClient.php passed with 0 errors.; Fix fork: castor cs-check --path=.hatfield/extensions/task-workflow/src/Ide/JetBrainsMcpClient.php clean.; Fresh reviewer: APPROVED; prior findings fixed, no unmapped external surface or cleanup-order blocker.; Parent verification: worktree clean at 70e87a64b; IDE diagnostics on JetBrainsMcpClient.php report 0 errors.
- Summary: Committed pre-PR review fixes at 70e87a64b0d9261e87ada5f56337363973d47009: corrected six prompt heading structures and made Hatfield close timeout 60s/open timeout 630s. Fresh re-review returned APPROVED; specification fidelity and Pi/Hatfield lifecycle parity passed with only non-blocking notes.

## Task workflow update - 2026-08-13T15:56:36.075Z
- Recorded fork run: d7geuazqmmjj
- Validation: Read .agents/skills/testing/SKILL.md and tests/AGENTS.md before QA.; castor test: PASS (4413 tests, 16668 assertions).; castor deptrac: PASS (0 violations, 0 errors).; castor phpstan: PASS (0 errors).; castor cs-check: PASS (files_fixed=0).; Final git status: clean.
- Summary: Focused task-to-pr validation passed on clean worktree at 70e87a64b0d9261e87ada5f56337363973d47009; ready for CODE-REVIEW transition.

## Task workflow update - 2026-08-13T15:58:41.885Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (111.3s).
- Pushed task/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools to origin.
- branch 'task/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools' set up to track 'origin/task/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools'.
- Created PR: https://github.com/ineersa/agent-core/pull/379
- Validation: Reviewer: APPROVED after pre-PR fixes.; castor test: PASS (4413 tests, 16668 assertions).; castor deptrac: PASS (0 violations, 0 errors).; castor phpstan: PASS (0 errors).; castor cs-check: PASS.; Manual Pi/PHP IDEA setup, MCP open, and exact-worktree semantic query: PASS.
- Summary: Implemented programmatic exact-worktree JetBrains lifecycle in Pi and Hatfield task-workflow ports, including minimal worktree IDEA metadata, one-shot MCP open/close, cleanup-safe reopen behavior, sanitized degradation, and exact-project delegated guidance. Fresh reviewer APPROVED; focused Castor gates passed.

## Task workflow update - 2026-08-13T16:53:32.422Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Address owner feedback: make root AGENTS.md JetBrains guidance coding-agent-neutral rather than naming Pi-specific ide_* tools or linking only .pi/APPEND_SYSTEM.md.

## Task workflow update - 2026-08-13T16:54:39.941Z
- Recorded fork run: f37tmbwnfi68
- Validation: Docs-only one-file change; manual heading/policy wording check passed.; Worktree clean after commit a483bf958.
- Summary: Owner wording feedback addressed in commit a483bf958: root AGENTS.md now states runtime-neutral JetBrains semantic-tool policy and defers exact names/capabilities to the active coding agent's system instructions.

## Task workflow update - 2026-08-13T16:55:28.241Z
- Validation: Reviewer: APPROVED for a483bf958.; Docs-only manual wording/structure validation passed.; Branch worktree clean and one commit ahead of PR branch.
- Summary: Fresh review of owner-feedback commit a483bf958 returned APPROVED. Runtime-neutral wording preserves semantic-tool preference, exact-checkout targeting, filesystem fallback, and runtime-owned names without adding surface.

## Task workflow update - 2026-08-13T16:57:32.841Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (113.2s).
- Pushed task/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools to origin.
- branch 'task/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools' set up to track 'origin/task/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools'.
- PR already exists: https://github.com/ineersa/agent-core/pull/379
- Validation: Owner-feedback reviewer: APPROVED.; Manual docs structure and policy fidelity check: PASS.; Previous castor test/deptrac/phpstan/cs-check: PASS before docs-only follow-up.
- Summary: Addressed PR owner feedback by making root AGENTS.md JetBrains tooling guidance runtime-neutral. Reviewer APPROVED; previous full focused gates remain valid because follow-up is docs-only.

## Task workflow update - 2026-08-14T20:48:33.929Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools: ide_close_project returned isError.
- Merged task/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-06-ensure-forks-and-subagents-use-jetbrains-ide-tools.
- Pulled integration checkout: Already up to date..
- Validation: GitHub PR #379 state: MERGED (merge commit 8b7cfbd2782a722b08eb74cd6398aa3d29568c6a).
- Summary: PR #379 verified merged at 2026-08-13T16:59:34Z. Finalize task metadata and remove the clean task worktree/IDE project.

## Task workflow update - 2026-08-14T20:52:27.532Z
- Recorded fork run: 9zxwswic67gk
- Validation: `LLM_MODE=true castor check`: FAIL on current integration main; deptrac/controller-replay/TUI/llm-real/cs-check/docs passed, unit test and phpstan failed at HatfieldDocsTool.php:129 missing AppResourceLocator::getAppRoot().; Failure attributed to later PR #382/docs catalog work, not PR #379; merge commit 8b7cfbd2782a722b08eb74cd6398aa3d29568c6a is an ancestor of HEAD.; Task worktree removed; JetBrains project no longer appears open; IDEA exclusions removed.; Integration had pre-existing unrelated modifications in ForkToolDefinitionBuilder.php and ForkToolContractTest.php; untouched.
- Summary: Post-merge validation confirmed PR #379 merge commit is present and JetBrains lifecycle files are on main. Full integration gate is red due to a later unrelated docs/catalog regression (`HatfieldDocsTool` calls missing `AppResourceLocator::getAppRoot()`), so the JetBrains task remains DONE.

## Task workflow update - 2026-08-15T17:16:43.326Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
