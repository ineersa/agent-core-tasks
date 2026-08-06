# EXT-02 Port task-workflow extension to Hatfield project extension

## Goal
## Context

Port the pi `task-workflow` extension (`.pi/extensions/task-workflow/`, TypeScript, ~7 files) to a native Hatfield project-level Composer extension. The pi extension is **NOT being decommissioned** — it stays. This adds a native Hatfield equivalent so the same workflow can run without pi.

**Depends on EXT-01** (`ext-01-extend-api-exec-prompt-commands-rewrite`), which adds the four API capabilities this port consumes: `exec()`, `registerPromptContributor()`, `registerCommand()`, and (indirectly, for consistency) the rewrite decision.

The pi extension drives this very repo's task board at `/home/ineersa/projects/agent-core-tasks`. Read the pi source for the exact behavior to port:
- `.pi/extensions/task-workflow/index.ts` — 4 tools (task_list, create_task, move_task, update_task) + slash command registration + prompt injection
- `.pi/extensions/task-workflow/task-store.ts` — task root resolution, list/find, markdown field get/set, render, lock path
- `.pi/extensions/task-workflow/exec.ts` — git/shell wrappers (repoRoot, gitOk, run)
- `.pi/extensions/task-workflow/worktrees.ts` — worktree create/merge, vendor+.vera copy, parent IDEA `.iml` exclusion blocks
- `.pi/extensions/task-workflow/pr.ts` — push, gh availability, find/create PR
- `.pi/extensions/task-workflow/prompt.ts` — workflow lifecycle prompt text
- `.pi/extensions/task-workflow/types.ts` — TaskStatus, TaskInfo, WorktreeCreateResult

## Packaging

A Composer package at `.hatfield/extensions/task-workflow/`, autoloaded via `.hatfield/extensions/vendor/autoload.php` (the loader `ExtensionManager::requireExtensionAutoload()` already requires). Namespace `Ineersa\HatfieldExt\TaskWorkflow`.

```
.hatfield/extensions/task-workflow/
  composer.json          # PSR-4 autoload Ineersa\\HatfieldExt\\TaskWorkflow\\ => src/
  src/
    TaskWorkflowExtension.php        # implements HatfieldExtensionInterface
    Settings/TaskWorkflowSettings.php
    Store/TaskStatusEnum.php
    Store/TaskInfo.php
    Store/TaskBoardStore.php          # port of task-store.ts
    Store/TaskMarkdown.php            # field get/set, render, appendLog
    Exec/GitExecutor.php              # port of exec.ts over ExtensionApi exec()
    Worktree/WorktreeCreateResult.php
    Worktree/WorktreeManager.php      # port of worktrees.ts
    Pr/PrManager.php                  # port of pr.ts
    Prompt/WorkflowPrompt.php         # implements PromptContributorInterface
    Tool/
      ListTasksHandler.php            # implements ToolHandlerInterface
      CreateTaskHandler.php
      MoveTaskHandler.php
      UpdateTaskHandler.php
    Command/TasksCommandHandler.php   # implements ExtensionCommandHandlerInterface (parametrized by status)
```

The package depends on `ineersa/hatfield-extension-api` conceptually — but until that's split out (see AGENTS.md "Extension API boundary"), the extension consumes the `Ineersa\Hatfield\ExtensionApi` namespace loaded by the host. Composer package must NOT duplicate ExtensionApi classes (no classmap of them); it only references them as platform deps.

## Registration (`TaskWorkflowExtension::register`)

1. Read settings via `$api->getSettings('task_workflow')`.
2. Resolve task root (precedence: `HATFIELD_TASK_WORKFLOW_ROOT` env → `extensions.settings.task_workflow.task_root` → auto-detect sibling `<repoBasename>-tasks` if it has status dirs — port `resolveTaskRoot` from task-store.ts).
3. Register a `WorkflowPrompt` contributor via `$api->registerPromptContributor()` (renders the lifecycle prompt with the resolved task root).
4. Register the 4 tools via `$api->registerTool(ToolRegistrationDTO)` with JSON Schema params (port the Typebox schemas from index.ts). Handlers implement `ToolHandlerInterface::__invoke(array): mixed`.
5. Register the 5 slash commands via `$api->registerCommand()` — `/tasks`, `/tasks-todo`, `/tasks-in-progress`, `/tasks-code-review`, `/tasks-done`.

## Behavior to port faithfully (invariants)

- **Task board root is external to the code repo.** Task mutations NEVER git-commit to the code repo. Port the `// NOTE: No git commit to code repo.` invariant.
- **move_task transitions** (exact semantics from index.ts):
  - TODO→IN-PROGRESS: require clean integration checkout; create `task/<slug>` branch + worktree at `../<repo>-worktrees/<slug>`; copy vendor/ and .vera/ when present; update parent IDEA module exclusions; record Branch/Worktree/Started metadata.
  - IN-PROGRESS→CODE-REVIEW: verify worktree clean; run deterministic `castor check` in worktree (timeout configurable, default 480s); push branch; create/update PR via gh (unless `pushOnly`); record PR URL/Status.
  - →DONE: merge `--no-ff` into integration checkout (require clean unless `requireCleanMain=false`/`cleanupStaleIndexEntries`); on merge failure report conflicts and stay in CODE-REVIEW; on success remove worktree + IDEA exclusions (when `cleanupWorktree`), optionally delete branch, `git pull`.
- **Concurrent mutation guard:** pi uses `withFileMutationQueue(lockPath)`. Port to PHP `flock()` on `<taskRoot>/.task-workflow.lock` inside each mutating handler.
- **Markdown task format** (renderTask + field get/set + appendLog): port verbatim — Status/Branch/Worktree/Fork run/PR URL/PR Status/Started/Completed metadata block + Work log section. Existing task files on this board must remain readable.
- **findTask** ambiguity/empty errors, **normalizeStatus** (accept `IN_PROGRESS`/`INPROGRESS`/`IN-PROGRESS`), **slugify**, **today** — port all helpers.

## Settings schema (add to `.hatfield/settings.yaml` + `docs/settings.md`)

```yaml
extensions:
  enabled:
    - Ineersa\HatfieldExt\TaskWorkflow\TaskWorkflowExtension
  settings:
    task_workflow:
      task_root: /home/ineersa/projects/agent-core-tasks
      castor_check_timeout_seconds: 480
```

## Castor commands used by move_task

The port runs `castor check` via the new `exec()` API. It must run in the worktree with `LLM_MODE=true` (see how castor-llm-mode sets env — but since this calls exec directly, set env via ExecOptions). Use `timeout` wrapper like the pi version: `timeout --kill-after=30s <N>s env LLM_MODE=true castor check`.

## QA gates (run before CODE-REVIEW)

- `castor deptrac` — extension package is outside src/ so not subject to deptrac, but confirm no src/ layer accidentally depends on it.
- `castor phpstan`, `castor cs-check` on the extension package (configure phpstan/cs to include the extension path, or run standalone).
- `castor test` — contract tests for the handlers against an isolated `var/tmp/test-{uuid}` task board + a temp git repo (use the testing skill's isolation conventions). Cover:
  - create_task → task_list → move_task TODO→IN-PROGRESS (creates branch+worktree) → update_task → move_task →DONE (merges, cleans up) happy path
  - findTask ambiguity + empty errors
  - markdown field get/set/appendLog round-trip
  - move_task refuses dirty integration checkout
- No live LLM needed — these are filesystem/git tools. Load the `testing` skill and read `tests/AGENTS.md` before writing tests.
- Do NOT run a full live move_task through the real board during tests — use isolated copies.

## Out of scope

- EXT-01 API additions (must already be merged).
- castor-llm-mode port (EXT-03).
- Decommissioning the pi extension (explicitly not doing this).

## Dependencies

- **Requires:** EXT-01 merged (consumes exec(), registerPromptContributor(), registerCommand()).

## Acceptance criteria
- Composer package at .hatfield/extensions/task-workflow/ with PSR-4 namespace Ineersa\HatfieldExt\TaskWorkflow, autoloadable via .hatfield/extensions/vendor/autoload.php
- TaskWorkflowExtension implements HatfieldExtensionInterface and registers 4 tools (task_list/create_task/move_task/update_task), 5 slash commands, and a workflow PromptContributor
- Task board root resolution matches pi precedence: HATFIELD_TASK_WORKFLOW_ROOT env -> extensions.settings.task_workflow.task_root -> sibling auto-detect
- move_task TODO->IN-PROGRESS->CODE-REVIEW->DONE transitions faithful to pi: worktree+branch creation, castor check gate, gh PR, no-ff merge, worktree/IDEA cleanup
- Task mutations never git-commit to the code repo; concurrent mutations guarded by flock on .task-workflow.lock
- Markdown task format identical to pi (existing task files remain readable)
- Isolated contract tests (var/tmp/test-{uuid} task board + temp git repo) cover the full move lifecycle, findTask errors, and markdown round-trip
- castor phpstan + cs-check + test pass; testing skill + tests/AGENTS.md conventions followed
- pi extension at .pi/extensions/task-workflow/ left in place (not decommissioned)

## Workflow metadata
Status: DONE
Branch: task/ext-02-task-workflow-extension-port
Worktree: /home/ineersa/projects/agent-core-worktrees/ext-02-task-workflow-extension-port
Fork run: 6o9gs9qopkmw
PR URL: https://github.com/ineersa/agent-core/pull/201
PR Status: merged
Started: 2026-06-23T14:36:22.449Z
Completed: 2026-06-24T17:54:05+00:00

## Work log
- Created: 2026-06-22T22:13:00.442Z

## Task workflow update - 2026-06-23T14:36:22.449Z
- Moved TODO → IN-PROGRESS.
- Created branch task/ext-02-task-workflow-extension-port.
- Created worktree /home/ineersa/projects/agent-core-worktrees/ext-02-task-workflow-extension-port.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/ext-02-task-workflow-extension-port.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/ext-02-task-workflow-extension-port.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/ext-02-task-workflow-extension-port.

## Task workflow update - 2026-06-23T14:40:40.686Z
- Summary: Context gathering complete. Three scouts returned comprehensive specs:
1. Full pi TS port spec (tool schemas, move_task transitions, shell commands, metadata fields, lock path, markdown format)
2. EXT-01 PHP API surfaces (all signatures for exec/prompt/command/tool registration)
3. Testing conventions (TestDirectoryIsolation, git init pattern, InMemoryExtensionApiBridge, QA config gaps)

Key design decisions locked:
- Env var: HATFIELD_TASK_WORKFLOW_ROOT (Hatfield-prefixed, per task body)
- Extension namespace: Ineersa\\HatfieldExt\\TaskWorkflow
- Package at .hatfield/extensions/task-workflow/ with own composer.json
- QA configs (phpstan.dist.neon, .php-cs-fixer.dist.php, phpunit.xml.dist) need .hatfield/extensions/ path additions
- Tests use TestDirectoryIsolation + temp git repo fixtures, no kernel boot needed
- move_task castor check: `timeout --kill-after=30s <N>s env LLM_MODE=true castor check` via exec() API
- task-start: moved to IN-PROGRESS, worktree at /home/ineersa/projects/agent-core-worktrees/ext-02-task-workflow-extension-port, EXT-01 confirmed live on main
- scouted full pi task-workflow TS spec (7 files, 1327 lines) via scout 1 — exact tool schemas, move_task transition logic, all shell/git/gh commands quoted
- scouted EXT-01 PHP API surfaces via scout 2 — ExtensionApiInterface (9 methods incl exec/registerPromptContributor/registerCommand), ExecInterface/ExecOptionsDTO(cwd,timeout,env)/ExecResultDTO, ToolRegistrationDTO, ToolHandlerInterface, CommandDefinitionDTO/ExtensionCommandHandlerInterface/CommandContextInterface, PromptContributorInterface, HatfieldExtensionInterface, SafeGuardExtension as reference structure
- scouted testing conventions via scout 3 — TestDirectoryIsolation helper, git init pattern from FileMentionIndexBuilderTest, InMemoryExtensionApiBridge (throws LogicException for v2 methods), phpstan.dist.neon/.php-cs-fixer.dist.php/phpunit.xml.dist all need .hatfield/extensions/ path additions

## Task workflow update - 2026-06-23T14:44:05.537Z
- Recorded fork run: lgd8dxkv5sww
- launched implementation fork lgd8dxkv5sww in worktree (background) — full EXT-02 port with all design decisions pre-resolved: namespace reorg, 16-file package structure, faithful port of all 4 tools + 5 commands + prompt + git/exec/pr/worktree managers, flock-based concurrency, QA config additions, isolated contract tests with stubbed ExecInterface

## Task workflow update - 2026-06-23T14:57:22.227Z
- Recorded fork run: q50dpl3uw8xh
- impl fork lgd8dxkv5sww completed the implementation (19 src files, 7 tests, 6 config changes, castor test passes 3324) BUT failed to commit and left corrupted phpstan.dist.neon scanFiles (.castor/helpers.php replaced with bogus .castor/ dir + nonexistent .hatfield/extensions/helpers.php) and lost composer.json trailing newline; root .gitignore only covers /vendor/ not .hatfield/extensions/vendor/
- launched salvage fork q50dpl3uw8xh to: fix phpstan scanFiles corruption, restore composer.json newline, gitignore .hatfield/extensions/vendor/+composer.lock, run FULL validation via castor (cs-fix, phpstan, deptrac, cs-check, test), commit on the branch — implementation is done and working, only fix/validate/commit

## Task workflow update - 2026-06-23T15:02:20.040Z
- Recorded fork run: q50dpl3uw8xh
- Validation: salvage fork q50dpl3uw8xh: castor cs-fix (4 files), castor phpstan 0 errors, castor deptrac 0 violations, castor cs-check 0 fixes, castor test 3324 OK, castor test --suite=extensions 14 OK; impl fork lgd8dxkv5sww: castor test 3324 OK (but did not commit; salvage fork handled commit + remaining gates); git verified: commit 971890c5d on task/ext-02-task-workflow-extension-port (correct branch, NOT main), clean tree, 36 files +2223/-4, vendor/composer.lock NOT tracked (gitignored), main untouched at 38ca412a2; merge-base check: base 3ab077914 < main 38ca412a2, NO file overlap with upstream → rebase will be clean
- Summary: EXT-02 implementation COMPLETE — committed as 971890c5d on task/ext-02-task-workflow-extension-port.

Two-fork path: impl fork (lgd8dxkv5sww) wrote the full port but failed to commit + corrupted phpstan scanFiles; salvage fork (q50dpl3uw8xh) fixed config, gitignored extension vendor/lock, ran full validation, committed.

DELIVERABLE: Composer package at .hatfield/extensions/task-workflow/ (namespace Ineersa\HatfieldExt\TaskWorkflow), 19 src files (1702 lines) + 7 test files + README + own composer.json. Faithful port of pi task-workflow: 4 tools (task_list/create_task/move_task/update_task), 5 slash commands (/tasks, /tasks-todo, /tasks-in-progress, /tasks-code-review, /tasks-done), WorkflowPrompt contributor. 3-tier task-root resolution (HATFIELD_TASK_WORKFLOW_ROOT env → settings → sibling autodetect), flock-based concurrency on .task-workflow.lock, byte-faithful move_task transitions (worktree+branch+vendor/.vera copy+IDEA exclusions; castor check gate via `timeout --kill-after=30s Ns env LLM_MODE=true castor check`; --no-ff merge + cleanup), pi markdown format preserved. No git commit to code repo invariant preserved.

CONFIG CHANGES: composer.json (autoload ext ns + autoload-dev test ns), phpstan.dist.neon (paths +.hatfield/extensions/, excludes vendor/tests), .php-cs-fixer.dist.php (+.hatfield/extensions), phpunit.xml.dist (+extensions suite), .hatfield/settings.yaml (enable TaskWorkflowExtension + task_workflow settings), docs/settings.md (documented task_workflow settings), .gitignore (ignore .hatfield/extensions/vendor/ + composer.lock). 36 files changed, +2223/-4.

SALVAGE NOTE: fork removed unused GitExecutor dep from TaskBoardStore (was only-written, never read — property.onlyWritten phpstan finding) and fixed a getSubPathName→getSubPathname typo in WorktreeManager; behavior unchanged. Extension tests excluded from phpstan analysis path (src IS analyzed); vendor excluded.

VALIDATION (all via castor, green): phpstan 0 errors, deptrac 0 violations, cs-check 0 fixes, test 3324 OK, test --suite=extensions 14 OK. castor check (full controller/TUI replay gate) NOT run — task-to-pr's responsibility.

MERGE-BASE: branch based on 3ab077914, main advanced to 38ca412a2 (COMP-06 auto-compaction PR, ~47 commits). NO file overlap with EXT-02 — rebase at task-to-pr will be clean (unlike EXT-01's stray-commit pathology). Local main is also one local merge commit ahead of origin/main (minor sync item).
- impl fork lgd8dxkv5sww: wrote full port (19 src + 7 tests) but FAILED to commit + corrupted phpstan scanFiles (.castor/helpers.php → bogus .castor/ dir + nonexistent helpers.php) + lost composer.json trailing newline
- salvage fork q50dpl3uw8xh: fixed phpstan scanFiles, restored composer.json newline, gitignored extension vendor/+composer.lock, added README, ran full castor validation, committed 971890c5d on correct branch
- fork removed dead GitExecutor dep from TaskBoardStore (phpstan property.onlyWritten) + fixed getSubPathName→getSubPathname typo — behavior unchanged
- merge-base divergence is benign: main advanced with COMP-06 compaction PR (no EXT-02 file overlap) — rebase at task-to-pr will be clean

## Task workflow update - 2026-06-23T15:41:00.663Z
- Validation: reviewer: APPROVE WITH SUGGESTIONS — all 7 constraint checks PASS (extension only depends on ExtensionApi; no shell injection [all exec via ExecInterface array→Process proc_open]; no git commit to code repo on status moves; flock released in finally; move_task faithful to all pi transition branches; config wiring complete; no BC shims); independent focused validation on commit 371c49141: castor deptrac 0 violations (HatfieldExtTaskWorkflow layer active+enforced, collector verified matching 19 classes), castor phpstan 0 errors, castor cs-check 0 fixed, castor test --suite=extensions 17 OK (+3 new), castor test 3327 OK; git merge-tree --write-tree origin/main HEAD = exit 0 (clean future merge); depfile.yaml overlap with upstream in distinct regions → clean merge; no rebase needed; commit 371c49141 verified on task branch (NOT main), clean tree, main untouched at f878be96
- Summary: EXT-02 task-to-pr complete. Reviewer verdict: APPROVE WITH SUGGESTIONS (all 7 constraint checks PASS, no critical/security issues). All actionable findings addressed in commit 371c49141 (on top of impl 971890c5d):
1. Added 2 CODE-REVIEW transition tests (moveTaskToCodeReviewRunsCastorCheckPushesAndCreatesPr + refusesWhenCastorCheckFails) — protects the merge-gate safety boundary, the highest-risk untested branch.
2. Added lock-released-on-exception test (withLockReleasesOnException) — protects the flock finally-invariant.
3. Restored rationale comment on the silent catch in WorktreeManager::copyTreeIfMissing (AGENTS.md hard rule).
4. Added deptrac HatfieldExtTaskWorkflow layer (path + collector + rule → AppExtensionApi only) — machine-enforces the extension boundary (was convention-only).

Deferred (reviewer's own optional/low rating + pre-PR risk): eager task-root resolution in register() (host handles gracefully); PrManager/Manager naming (subjective, reviewer minor); flock-no-timeout (faithful to TS); naive rel(); pi- sentinel interop marker.

Branch: task/ext-02-task-workflow-extension-port, 2 commits (971890c5d + 371c49141), 38 files +2454/-4 vs origin/main. Merge-base stale-but-clean (15 behind); git merge-tree --write-tree = exit 0 (clean future merge); depfile.yaml overlaps upstream in DIFFERENT regions (EXT-02 adds layer/path/rule ~L10/116/610; upstream adds -AppTool ~L551) — merges cleanly. No rebase needed (benign, unlike EXT-01).

BOUNDARY ENFORCEMENT VERIFIED: collector regex matches all 19 extension FQCNs (collected into layer), excludes non-extension classes; deptrac 0 violations; a future CodingAgent import regression WILL be caught.
- task-to-pr: reviewer APPROVE WITH SUGGESTIONS, all 7 constraint checks PASS, no critical/security issues
- fork commit 371c49141: added 2 CODE-REVIEW transition tests (merge-gate boundary) + 1 lock-exception test + restored copyTreeIfMissing catch rationale comment + deptrac HatfieldExtTaskWorkflow layer (machine-enforced boundary)
- independently verified all focused gates green on 371c49141: deptrac 0 (layer enforcement proven via collector regex test matching 19 extension classes), phpstan 0, cs-check 0, extensions 17, full test 3327
- merge situation benign: stale-but-clean base (15 behind origin/main), merge-tree exit 0, depfile overlap in distinct regions → no rebase needed

## Task workflow update - 2026-06-23T19:24:36.997Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 600s)...
- castor check passed (0.2s).
- Pushed task/ext-02-task-workflow-extension-port to origin.
- branch 'task/ext-02-task-workflow-extension-port' set up to track 'origin/task/ext-02-task-workflow-extension-port'.
- Created PR: https://github.com/ineersa/agent-core/pull/201

## Task workflow update - 2026-06-24T00:10:59.148Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Addressing critical bug found via runtime testing: extension fails to connect at runtime. Root cause: EXT-01 API gap — no extension-facing tool handler contract. ExtensionToolRegistryBridge::registerTool() requires internal CodingAgent\Tool\ToolHandlerInterface (line 63) which extensions cannot implement (deptrac boundary). The bridge throws on the first tool (task_list) → entire register() aborts → zero tools AND zero commands register (confirmed in .hatfield/logs/agent-2026-06-23.log). Fix: (1) new ExtensionToolHandlerInterface in ExtensionApi\Tool + named ExtensionToolHandlerAdapter in App layer, (2) bridge accepts/adapts extension handlers, (3) EXT-02's 4 handlers implement the new interface, (4) add integration test through REAL bridges asserting tools+commands available (the test that would have caught this). Existing ExtensionToolRegistryBridgeTest enforces the broken behavior (testHandlerMustImplementToolHandlerInterface) and must be updated.

## Task workflow update - 2026-06-24T00:15:50.229Z
- Found & fixed CRITICAL runtime bug: extension silently failed to connect. Root cause = EXT-01 API gap — ExtensionToolRegistryBridge::registerTool() required internal CodingAgent\Tool\ToolHandlerInterface which extensions cannot implement (deptrac boundary). Bridge threw on first tool (task_list) → entire register() aborted → 0 tools + 0 commands. Confirmed in .hatfield/logs/agent-2026-06-23.log.
- Fix commit 9785e67a6: (1) new Ineersa\Hatfield\ExtensionApi\Tool\ExtensionToolHandlerInterface (boundary-clean handler contract), (2) named ExtensionToolHandlerAdapter in App layer wrapping it to internal ToolHandlerInterface, (3) bridge wraps handler via adapter, (4) tightened ToolRegistrationDTO.handler type mixed→ExtensionToolHandlerInterface, (5) 4 EXT-02 handlers implement new interface, (6) updated ExtensionToolRegistryBridgeTest (it previously ENFORCED the broken behavior).
- Added TaskWorkflowExtensionIntegrationTest — the test requested by user. Uses REAL ExtensionToolRegistryBridge + real ToolRegistry + real TuiCommandRegistryAdapter + real SlashCommandRegistry + real ExtensionManager (NOT in-memory stub). Asserts: loadExtensions() returns [] diagnostics, 4 tools registered, 5 slash commands available. This is the test that would have caught the bug.
- Independently verified by parent: integration test passes (1 test, 10 assertions) via real bridge; deptrac 0 violations; phpstan 0 errors; extensions suite 17 OK; full castor test 3504 OK (per fork). Branch pushed to origin (9785e67a6), PR #201 updated.

## Task workflow update - 2026-06-24T02:40:14.656Z
- FOUND PRE-EXISTING CASTOR BUG (not an EXT-02 issue — affects castor gate for ALL tasks, likely pi too): `.castor/tasks.php` cleanup_stale_check_workers() self-kills its own `timeout` wrapper when invoked as `timeout ... env LLM_MODE=true castor check` with cwd=worktree. (1) The `timeout` process has cwd=worktree=$root so it matches `_stale_check_worker_belongs_to_checkout` via cwd-under-root; (2) the `str_contains($cmdline,'castor check')` rule fires; (3) the self-PID guard only protects castor's PHP pid (getmypid), NOT its `timeout` ancestor; (4) SIGKILL timeout wrapper → whole tree dies → exit 0 in ~0.2s with ZERO lanes run. Confirmed via DEBUG probe: castor killed its own `timeout --kill-after=30s 480s env LLM_MODE=true castor check` (pid 593114).
- IMPACT: Every CODE-REVIEW transition's castor gate has been a FALSE PASS. move_task reports 'castor check passed (0.8s)' and creates the PR without real validation. This is why TEST-01 (2026-06-24-test-01-hello-world-in-readme) moved to CODE-REVIEW in 0.8s and created PR #204 with no real check.
- EXT-02 IS INNOCENT: its invocation is byte-identical to pi's task-workflow (same argv, same cwd=worktree). pi's gate is almost certainly inert too.
- TEMP FIX (diagnostic, uncommitted, TEST-01 worktree only): commented out `cleanup_stale_check_workers($root)` at line 78 of the TEST-01 worktree `.castor/tasks.php` with TEMP DISABLED marker. With preflight disabled, castor check runs for real (67s, all 7 lanes execute). PROPER FIX (later): self-guard must exclude the whole ancestor chain (walk ppid up to the castor php pid), or drop the generic 'castor check' substring + cwd-under-root combo. Decision deferred — not blocking EXT-02.

## Task workflow update - 2026-06-24T03:38:10.916Z
- Recorded fork run: r7fuh9fvbc70
- Validation: Verified by parent: branch task/ext-02-task-workflow-extension-port is ahead of origin by commit dbe3be11f; worktree clean except ahead state; touched files: `.castor/tasks.php` only.; Fork validation: `castor cs-fix` fixed style in `.castor/tasks.php`; `castor cs-check` passed (0 files to fix); `castor phpstan` passed (0 errors).; Fork performed non-destructive PHP ancestry dry-run; no signals/process kills; full `castor check` intentionally not run due live same-checkout EXT-02 controller/workers.
- Summary: Fork r7fuh9fvbc70 implemented the Castor stale-worker launcher ancestry guard on EXT-02 branch. Commit dbe3be11f08606aee03e8bb2bb05216206ab90d6 (`fix(castor): protect check cleanup launcher ancestry`) changes only `.castor/tasks.php` (+67/-5). It adds `_stale_check_worker_parent_pid()` and `_stale_check_worker_protected_launcher_ancestry()` using /proc PPid walking, then skips any stale-worker candidate whose PID is in the current Castor process ancestry. This prevents `timeout ... castor check` / shell / Symfony Process / Hatfield launcher ancestors from being killed by cleanup while preserving cleanup of unrelated stale same-checkout workers. No EXT-02 extension code touched; no processes killed; no full castor check run because live EXT-02 controller/workers were present and could be intentionally cleaned by preflight.

## Task workflow update - 2026-06-24T17:28:17.887Z
- Recorded fork run: ia61edqscvjf
- Validation: Merge: `git merge origin/main` succeeded cleanly; merge commit b2f34375b; no manual file edits; branch now ahead of origin/task by merge + dbe3be11f/main delta.; EXT-02 worktree castor check attempt #1: deptrac OK, test OK (3530 tests), test:controller-replay OK (7 tests), phpstan OK, cs-check OK; failed test:tui (TuiOutputCapNoticeE2eTest timeout) and test:llm-real (CompactionLiveSmokeTest no compaction.completed).; EXT-02 worktree castor check attempt #2: test:controller-replay OK again; failed test:tui (TuiAutoCompactionCancelE2eTest timeout / empty pane) and test:llm-real (CompactionLiveSmokeTest retryable LLM provider network error).; Focused isolation: `castor test:tui --filter=TuiOutputCapNoticeE2eTest` passed on EXT-02; `castor test:llm-real --filter=CompactionLiveSmokeTest` passed on main; full `castor check` on main passed (~148s).; No stale current-user worker processes for the EXT-02 worktree were found before checks; no processes killed; branch not pushed due failed gate.
- Summary: Fork ia61edqscvjf merged latest origin/main into EXT-02 with merge commit b2f34375b. The castor ancestry guard commit dbe3be11f remains intact. Post-merge castor check no longer fails controller-replay: controller-replay passed both full-check attempts, so previous controller-replay gate failure was branch staleness. Full castor check in the EXT-02 worktree still failed twice under parallel load on test:tui + test:llm-real with different/flaky symptoms; isolated focused tests passed and full castor check on main passed, pointing to worktree parallel resource/proxy contention rather than EXT-02 extension code. Fork did not push because the gate was not green.

## Task workflow update - 2026-06-24T17:39:06.026Z
- Recorded fork run: 6o9gs9qopkmw
- Validation: Fork loaded/followed testing conventions and tests/AGENTS.md per handoff; Castor-only QA used.; Cleaned stale current-user processes for this worktree only: PIDs 203719, 203724, 204689, 204695.; castor test:tui --filter=TuiAutoCompactionCancelE2eTest: OK (~5.9s); castor test:tui --filter=TuiOutputCapNoticeE2eTest: OK (~6.2s); castor test:llm-real --filter=CompactionLiveSmokeTest: OK (~27.6s); castor cs-fix / castor cs-check: 0 files to fix; castor check: OK, quality ok in 143.0s (deptrac, test 3530, controller-replay 7, tui 16, llm-real 9, phpstan, cs-check)
- Summary: Fork diagnosed CODE-REVIEW gate failures as parallel castor check load + tight waits, not EXT-02 task-workflow runtime behavior. Commit e3df9ab17 hardens shared test waits only (5 tests files, +32/-8): TmuxHarness named 20s startup/assistant block parallel-check budgets; TuiAutoCompactionCancelE2eTest and TuiOutputCapNoticeE2eTest use those budgets; ControllerE2eTestCase adds liveCompactionEventWaitTimeout()=60s; CompactionLiveSmokeTest uses it. No EXT-02 extension or production changes. Branch pushed to origin/task/ext-02-task-workflow-extension-port.

## Task workflow update - 2026-06-24T17:40:12.782Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (59.3s).
- Pushed task/ext-02-task-workflow-extension-port to origin.
- branch 'task/ext-02-task-workflow-extension-port' set up to track 'origin/task/ext-02-task-workflow-extension-port'.
- PR already exists: https://github.com/ineersa/agent-core/pull/201
- Validation: Fork 6o9gs9qopkmw full castor check: OK, quality ok in 143.0s on EXT-02 worktree after e3df9ab17.; Focused validations passed: TuiAutoCompactionCancelE2eTest, TuiOutputCapNoticeE2eTest, CompactionLiveSmokeTest, cs-check.; Reviewer had previously APPROVE WITH SUGGESTIONS; actionable findings addressed before this gate retry.
- Summary: Ready for CODE-REVIEW after merge from origin/main, castor stale-worker ancestry guard, and parallel-check wait hardening. Latest branch tip e3df9ab17 was pushed by fork; full castor check passed locally in the worktree before this transition.

## Task workflow update - 2026-06-24T17:54:05+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/ext-02-task-workflow-extension-port into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/ext-02-task-workflow-extension-port.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/ext-02-task-workflow-extension-port.
- Pulled integration checkout: Already up to date..

## Task workflow update - 2026-06-24T17:55:08+00:00
- Validation: castor deptrac: 0 errors, 0 warnings; castor phpstan: OK (no errors); castor cs-check: 0 files to fix out of 709; castor test --suite=extensions: OK (17 tests, 51 assertions)
- Post-merge validation: deptrac, phpstan, cs-check, extensions suite all pass.
