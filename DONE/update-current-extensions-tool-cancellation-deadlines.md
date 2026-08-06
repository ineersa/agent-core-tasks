# Update current extensions to honor tool cancellation and deadlines

## Goal
Follow-up to `fix-generic-tool-timeout-preemption-semantics` after the public ExtensionApi exposes cancellation/deadline data through `ToolInvocationContextDTO`.

Audit the current in-repo extensions under `.hatfield/extensions/` and update every extension tool that can block, poll, acquire locks, invoke subprocesses, perform HTTP/provider work, or otherwise run long enough to require interruption. Use the documented ExtensionApi cancellation/deadline contract and structured cancelled/timed-out outcomes. Do not add process isolation to short finite handlers.

Current extensions to inventory: castor-llm-mode, file-rewind, observational-memory, and task-workflow. At creation time, tool registrations are present in castor-llm-mode, observational-memory, and task-workflow; verify rather than assuming this remains complete.

## Acceptance criteria
- Every current in-repo extension is inventoried, with each tool handler classified as short finite or cancellation/deadline-aware.
- Long-running extension subprocesses, polling loops, lock waits, and provider/HTTP operations observe the public ToolInvocationContextDTO cancellation/deadline contract and clean up owned resources.
- Short finite handlers are not wrapped in unnecessary subprocesses or speculative abstractions.
- Cancelled and timed-out handlers return structured outcomes consistent with the ExtensionApi documentation.
- Focused regression tests prove at least one extension operation stops before normal completion on cancellation and at least one deadline path cleans up owned work.
- Testing skill and tests/AGENTS.md conventions are followed; all QA runs through Castor.

## Workflow metadata
Status: DONE
Branch: task/update-current-extensions-tool-cancellation-deadlines
Worktree: /home/ineersa/projects/agent-core-worktrees/update-current-extensions-tool-cancellation-deadlines
Fork run: s3f5vuqgz5s6
PR URL: https://github.com/ineersa/agent-core/pull/355
PR Status: merged
Started: 2026-08-04T17:25:51.132Z
Completed: 2026-08-04T20:07:40.691Z

## Work log
- Created: 2026-07-31T20:40:37.893Z

## Task workflow update - 2026-08-04T17:25:51.132Z
- Moved TODO → IN-PROGRESS.
- Created branch task/update-current-extensions-tool-cancellation-deadlines.
- Created worktree /home/ineersa/projects/agent-core-worktrees/update-current-extensions-tool-cancellation-deadlines.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/update-current-extensions-tool-cancellation-deadlines.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/update-current-extensions-tool-cancellation-deadlines.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/update-current-extensions-tool-cancellation-deadlines.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/update-current-extensions-tool-cancellation-deadlines.
- Summary: Scope finalized in task-explain: add cooperative cancellation to existing public ExecOptionsDTO/ExecResultDTO bridge; no new ToolRegistrationDTO timeout budgets/settings; registered extension tools only; keep file-rewind hooks and observational-memory provider jobs intact; active task-list output work remains independent.

## Task workflow update - 2026-08-04T17:27:47.361Z
- Recorded fork run: abik1w59tumj
- Summary: Implementation fork launched in isolated task worktree with finalized scope: additive cancellable ExtensionApi exec, registered long-running extension tools only, no new timeout budgets/settings, structured outcomes/resource cleanup, focused Castor tests, and no task-list output changes.

## Task workflow update - 2026-08-04T17:40:35.943Z
- Recorded fork run: abik1w59tumj
- Validation: Fork confirmed it read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before test work.; Focused Castor filter suite — OK, 55 tests.; `castor test` — OK, 4433 tests / 16471 assertions.; `castor deptrac` — 0 violations.; `castor phpstan` — 0 errors.; `castor cs-check` — clean after `castor cs-fix`.; Parent verified branch HEAD 0141b9c0d, clean worktree, and no unrelated/generated files in the commit list.; `castor check` intentionally deferred to task-to-pr.
- Summary: Implementation complete at commit 0141b9c0db0edc1f9725407edb773edc043b2279. Added cancellable public ExtensionApi exec polling/results, preserved structured cancel/timeout outcomes in ToolExecutor, converted blocking task-workflow handlers and lock/subprocess phases to cooperative control, and added recall checkpoints. Verified extension inventory; finite task_list, castor rewrite, file-rewind hooks, and OM background/isolated tools remain unchanged. Parent confirmed clean worktree and expected 23-file commit diff (+1490/-262).

## Task workflow update - 2026-08-04T18:20:09.022Z
- Recorded fork run: 3e3uv2638a2x
- Summary: Task-to-PR reviewer returned REQUEST CHANGES on 0141b9c0d. Fix fork launched for: preserving timed_out during concurrent post-invoke cancellation; retaining IDEA exclusions when partial worktree removal fails; preserving ExecBridge interrupt flags if stop throws; removing unused repoRoot control plumbing; and documenting Castor shutdown slack.
- Reviewer on 0141b9c0d: REQUEST CHANGES. Specification fidelity PASS; test thesis partially satisfied. One blocker and sensible edge-case/simplification findings delegated to fix fork 3e3uv2638a2x.

## Task workflow update - 2026-08-04T18:31:22.052Z
- Recorded fork run: n545b6uas1g7
- Summary: Re-review on 974e5215a returned APPROVE WITH SUGGESTIONS: all prior correctness findings resolved, specification fidelity PASS, test thesis PASS. Narrow fork launched to remove two stale duplicate WorktreeManager PHPDoc blocks; pre-existing dead repoRoot and synthetic stop-throw test explicitly left out of scope.
- Re-review on 974e5215a: APPROVE WITH SUGGESTIONS. Five prior findings resolved. One sensible convention cleanup delegated to fork n545b6uas1g7; remaining notes were pre-existing/out-of-scope or explicitly low-value non-actionable.

## Task workflow update - 2026-08-04T18:36:50.941Z
- Validation: Final reviewer on HEAD 689e8dfe7 — APPROVED; no actionable findings; specification fidelity PASS; ponytail PASS; test thesis PASS.; Worktree `castor test` — OK, 4435 tests / 16481 assertions.; Worktree `castor deptrac` — 0 violations / 0 errors.; Worktree `castor phpstan` — 0 errors / 0 file errors.; Worktree `castor cs-check` — clean, 0 files fixed.; Worktree clean at commit 689e8dfe7ffd194a08659a6798452ed2d5b10f48.; An earlier accidental `castor test` ran in the integration checkout and also passed (4424 tests / 16426 assertions); authoritative task validation is the subsequent worktree run above.
- Summary: Final HEAD 689e8dfe7ffd194a08659a6798452ed2d5b10f48. Reviewer APPROVED the complete 23-file diff after two fix iterations; specification fidelity PASS, ponytail PASS, and test thesis PASS with no actionable findings remaining. Worktree is clean and focused task-to-PR validation is green.
- Task-to-PR review complete: initial REQUEST CHANGES fixed by 974e5215a; final convention cleanup committed as 689e8dfe7; final reviewer APPROVED current HEAD. Focused Castor validation passed in the task worktree.

## Task workflow update - 2026-08-04T18:39:54.718Z
- Validation: First automatic `castor check`: failed only `test:llm-real` ShellFollowUpLiveE2eTest; artifact `var/reports/qa-20260804-183658-264016-25bb677b/check-test:llm-real.log`.; Focused `castor test:llm-real --filter=ShellFollowUpLiveE2eTest` — OK, 2 tests / 21 assertions in 16.7s.; No code changes after the transient live-lane failure; HEAD remains 689e8dfe7 and worktree clean.
- Summary: First CODE-REVIEW gate attempt failed only in live `ShellFollowUpLiveE2eTest::testShellThenFollowUpOnCompletedRun` (expected tool_execution.started; no task-related production failure). Focused worktree rerun `castor test:llm-real --filter=ShellFollowUpLiveE2eTest` passed both tests (2 tests / 21 assertions) with no code changes. Worktree remains clean; retrying deterministic gate.

## Task workflow update - 2026-08-04T18:41:46.267Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (99.4s).
- Pushed task/update-current-extensions-tool-cancellation-deadlines to origin.
- branch 'task/update-current-extensions-tool-cancellation-deadlines' set up to track 'origin/task/update-current-extensions-tool-cancellation-deadlines'.
- Created PR: https://github.com/ineersa/agent-core/pull/355
- Summary: Final reviewer APPROVED HEAD 689e8dfe7. Focused worktree validation passed. First gate attempt had a transient ShellFollowUp live test miss; focused live rerun passed 2 tests / 21 assertions with no code changes. Retrying deterministic gate and PR creation.

## Task workflow update - 2026-08-04T18:41:54.477Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/355
- Updated PR Status: open
- Validation: Automatic deterministic `castor check` — PASSED in 99.4s.; Branch `task/update-current-extensions-tool-cancellation-deadlines` pushed to origin.; PR created: https://github.com/ineersa/agent-core/pull/355
- Summary: Task moved to CODE-REVIEW. Deterministic `castor check` passed on retry in 99.4s, branch pushed, and PR #355 created. Final reviewed commit: 689e8dfe7ffd194a08659a6798452ed2d5b10f48.

## Task workflow update - 2026-08-04T19:43:47.319Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Addressing user review feedback: extension-specific cancellation/timed_out result normalization must not modify AgentCore. Rework the result bridge under CodingAgent extension namespace and remove the ToolExecutor/AgentCore test changes.

## Task workflow update - 2026-08-04T19:51:19.019Z
- Summary: Architecture probe confirmed `kind=interrupt` adapter mapping is unsafe because core interprets it as human-input suspension (`WaitingHuman`). Chosen minimal boundary-safe fix: revert all AgentCore production/test changes and use the existing ToolResultProcessor extension point from a `src/CodingAgent/Extension` processor, restricted to extension-owned registrations. No new public API/settings.
- User review decision: extension-specific cancellation/timed_out normalization must not touch AgentCore. Scout rejected the tempting `kind=interrupt` shortcut because it would misroute cancellation/timeouts into HITL. Implementation will stay in CodingAgent Extension namespace and reuse the existing generic ToolResultProcessor contract.

## Task workflow update - 2026-08-04T19:52:13.380Z
- Recorded fork run: s3f5vuqgz5s6
- Summary: Review-fix implementation delegated to fork s3f5vuqgz5s6: restore AgentCore production/test files to origin/main and implement extension-owned outcome arbitration through a CodingAgent Extension ToolResultProcessor, explicitly avoiding the unsafe `kind=interrupt` shortcut.
- Fork s3f5vuqgz5s6 launched for PR feedback iteration. Required production normalization location: `src/CodingAgent/Extension/**`; AgentCore diff must become empty.

## Task workflow update - 2026-08-04T19:56:21.542Z
- Recorded fork run: s3f5vuqgz5s6
- Validation: Fork read testing skill and tests/AGENTS.md before test work.; `castor test --filter='ExtensionToolControlFlagToolResultProcessorTest\|ExtensionToolRegistryBridgeTest\|ToolExecutorTest'` — OK, 48 tests / 152 assertions.; `castor test` — OK, 4434 tests / 16488 assertions.; `castor deptrac` — 0 violations.; `castor phpstan` — 0 errors.; `castor cs-check` — clean.; Verified `git diff origin/main...HEAD -- src/AgentCore tests/AgentCore` is empty.
- Summary: Fork completed at 0dd2e6bfa2e4154db09d2f870263d6e8a067c94b. AgentCore ToolExecutor and its test now match origin/main exactly; extension-owned control-flag arbitration moved to `src/CodingAgent/Extension/ExtensionToolControlFlagToolResultProcessor.php` using the existing processor extension point. Worktree clean.
- Review fix commit 0dd2e6bfa moves extension control-map normalization out of AgentCore. New processor is restricted by existing `extensionOwnerClass`; no `kind=interrupt`, public API, settings, or behavior additions.

## Task workflow update - 2026-08-04T20:02:35.656Z
- Validation: Review-fix reviewer on HEAD 0dd2e6bfa — APPROVED with no actionable findings.; Specification fidelity PASS; architecture boundary PASS; ponytail PASS; test thesis PASS.; Worktree clean and one commit ahead of existing PR branch.; Verified no `origin/main...HEAD` diff under `src/AgentCore` or `tests/AgentCore`.; Focused/full Castor validation from implementation fork remains green: 4434 tests / 16488 assertions, deptrac 0, phpstan 0, cs-check clean.
- Summary: PR-feedback iteration complete at 0dd2e6bfa2e4154db09d2f870263d6e8a067c94b. Reviewer APPROVED: specification fidelity PASS, architecture boundary PASS, ponytail PASS, test thesis PASS. AgentCore diff is empty; extension outcome arbitration now lives under CodingAgent Extension only.
- Final review confirmed `kind=interrupt` is absent and extension-only processor ordering is safe for current short control maps. Ready to update PR #355.

## Task workflow update - 2026-08-04T20:04:32.974Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (108.9s).
- Pushed task/update-current-extensions-tool-cancellation-deadlines to origin.
- branch 'task/update-current-extensions-tool-cancellation-deadlines' set up to track 'origin/task/update-current-extensions-tool-cancellation-deadlines'.
- PR already exists: https://github.com/ineersa/agent-core/pull/355
- Summary: Addressed user review feedback in 0dd2e6bfa: AgentCore production/tests now match origin/main; extension-specific cancelled/timed_out arbitration moved to `src/CodingAgent/Extension/ExtensionToolControlFlagToolResultProcessor.php`. Reviewer APPROVED; focused/full tests, deptrac, phpstan, and cs-check passed. Updating PR #355.

## Task workflow update - 2026-08-04T20:04:38.642Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/355
- Updated PR Status: open
- Validation: Automatic deterministic `castor check` — PASSED in 108.9s after review fix.; Updated branch pushed to existing PR #355.; Final reviewed HEAD: 0dd2e6bfa2e4154db09d2f870263d6e8a067c94b.
- Summary: PR #355 updated with boundary-clean review fix 0dd2e6bfa2e4154db09d2f870263d6e8a067c94b. Automatic deterministic `castor check` passed in 108.9s and branch was pushed. AgentCore remains untouched by the final diff.

## Task workflow update - 2026-08-04T20:07:40.691Z
- Moved CODE-REVIEW → DONE.
- Merged task/update-current-extensions-tool-cancellation-deadlines into integration checkout.
- Merge made by the 'ort' strategy.
 .../src/Query/OmQueryService.php                   |  99 ++++++-
 .../src/Tool/RecallToolHandler.php                 |  36 ++-
 .../tests/RecallToolHandlerTest.php                |  91 +++++++
 .../task-workflow/src/Exec/GitExecutor.php         |  44 ++-
 .../extensions/task-workflow/src/Pr/PrManager.php  |  71 ++++-
 .../task-workflow/src/Store/TaskBoardLock.php      |  64 ++++-
 .../task-workflow/src/Tool/CreateTaskHandler.php   |  53 ++--
 .../task-workflow/src/Tool/InvocationControl.php   | 105 +++++++
 .../task-workflow/src/Tool/MoveTaskHandler.php     | 302 ++++++++++++++++-----
 .../task-workflow/src/Tool/UpdateTaskHandler.php   | 121 +++++----
 .../task-workflow/src/Worktree/WorktreeManager.php | 193 +++++++++++--
 .../tests/CreateTaskHandlerCancellationTest.php    | 123 +++++++++
 .../task-workflow/tests/MoveTaskHandlerTest.php    | 170 ++++++++++--
 .../task-workflow/tests/TaskBoardLockTest.php      |  34 +++
 .../tests/TaskWorkflowHandlerToonOutputTest.php    |   5 +-
 docs/tool-execution.md                             |   6 +
 src/CodingAgent/Extension/ExtensionExecBridge.php  | 108 ++++++--
 ...ExtensionToolControlFlagToolResultProcessor.php |  86 ++++++
 .../ExtensionApi/Exec/ExecInterface.php            |   8 +-
 .../ExtensionApi/Exec/ExecOptionsDTO.php           |  10 +-
 .../ExtensionApi/Exec/ExecResultDTO.php            |   1 +
 .../Extension/ExtensionExecBridgeTest.php          |  65 ++++-
 ...nsionToolControlFlagToolResultProcessorTest.php | 158 +++++++++++
 23 files changed, 1693 insertions(+), 260 deletions(-)
 create mode 100644 .hatfield/extensions/task-workflow/src/Tool/InvocationControl.php
 create mode 100644 .hatfield/extensions/task-workflow/tests/CreateTaskHandlerCancellationTest.php
 create mode 100644 src/CodingAgent/Extension/ExtensionToolControlFlagToolResultProcessor.php
 create mode 100644 tests/CodingAgent/Extension/ExtensionToolControlFlagToolResultProcessorTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/update-current-extensions-tool-cancellation-deadlines.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/update-current-extensions-tool-cancellation-deadlines.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #355 confirmed merged on GitHub at 2026-08-04T20:06:57Z (merge commit ec1d495ec5a9c85878a700e3d6e9c1cb6a13b2e0). Moving task to DONE and syncing integration checkout.

## Task workflow update - 2026-08-04T20:09:53.312Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/355
- Updated PR Status: merged
- Validation: GitHub PR #355 state MERGED at 2026-08-04T20:06:57Z; merge commit ec1d495ec5a9c85878a700e3d6e9c1cb6a13b2e0.; Post-merge `LLM_MODE=true castor check` — PASSED (`quality: ok`).; Post-merge lanes: unit 4435 tests / 16490 assertions; controller replay 11 / 160; TUI 32 / 204; llm-real 13 / 144; deptrac/phpstan/cs-check all OK.; QA artifact integrity, leak check, and llama-proxy cache guard all passed.; Integration checkout clean at 5bd5f0cda; task worktree removed and IDEA exclusions cleaned.
- Summary: PR #355 merged and task moved to DONE. Integration checkout synchronized, task worktree and IDEA exclusions removed. Post-merge `LLM_MODE=true castor check` passed; integration checkout is clean.
