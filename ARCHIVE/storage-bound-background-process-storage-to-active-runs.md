# STORAGE- Bound background-process storage to active runs

## Goal
Fix `.hatfield/tmp/bg` and background-process persistence/lifecycle. The measured snapshot contains 19,945 files / 281.4 MiB under `.hatfield/tmp/bg` and 6,696 `background_process` database rows, which is incompatible with temporary active-run ownership.

Finalized product behavior:
- Track only commands that actually transition to background execution.
- Foreground-only Bash/tool executions must not create durable background-process files or rows.
- Background jobs, `bg_status` listing, output retrieval, and stop operations are scoped to the currently active owning run.
- While the run is active, retain only the minimal artifacts required to control the owned process and retrieve its background output.
- When the run ends cleanly and has no queued/in-flight continuation, stop all of its owned background process trees and clear that run's `.hatfield/tmp/bg` artifacts.
- Cancellation, failure, session deletion, and controller shutdown must also trigger owned background-process termination and cleanup at the correct lifecycle boundary.
- Background-process storage is temporary operational state, not historical session data.

This supersedes cancelled task `2026-08-21-audit-bg-status-process-list`; the previously open retention decision is now finalized.

## Acceptance criteria
- Trace every creation/read/update/delete path for `.hatfield/tmp/bg`, `background_process` rows, Bash foreground-to-background transition, `bg_status`, deferred completion, cancellation, controller shutdown, run terminalization, session deletion, and startup recovery before implementation.
- Define one owning lifecycle keyed by run ID and background job ID. Files and database rows must identify the same current active-run job; cross-run listing, retrieval, stop, or cleanup is forbidden.
- Do not persist a background-process row or durable `.hatfield/tmp/bg` artifact merely because a foreground command was launched or monitored. Publish background ownership only when the command actually transitions to background. If provisional process control is unavoidable, keep it private and remove it deterministically before returning a foreground result.
- Restructure `.hatfield/tmp/bg` ownership per active run. Keep only artifacts demonstrably needed for process ownership/control and output retrieval. Do not retain duplicate PID/status metadata in files when the database is authoritative, and do not retain copied command payloads or unrelated foreground output.
- Scope `bg_status list`, inspect/tail, and stop to the caller's current run. Exclude foreground-only, cross-run, orphaned, and historical terminal entries.
- Keep completed background-job output available only while its owning run remains active and the result may still be inspected/consumed. Once the run reaches a clean quiescent end, delete its background rows and files; do not preserve a permanent background-job history table.
- At clean quiescent run shutdown—no queued/in-flight continuation and no pending deferred/background completion—terminate all still-running owned background process groups, wait through the existing bounded shutdown policy, record/emit the required terminal result while the run is still owned, then delete that run's rows and `.hatfield/tmp/bg` tree.
- Apply the same owned cleanup on cancellation, unrecoverable failure, explicit session deletion, and controller shutdown. Define whether queued follow-up keeps the run active so cleanup cannot race a legitimate continuation.
- Handle controller/process crashes safely: on startup/resume, reconcile only verifiably owned rows/artifacts; detect PID reuse using existing process identity metadata or add the minimum required identity token; never signal a process based only on a stale PID file.
- Never signal or modify root-owned processes, processes tagged with `HATFIELD_SESSION_ID`, unrelated process groups, or another run's jobs. Process-tree ownership and teardown must follow root AGENTS.md safety constraints.
- Make filesystem publication/removal atomic where files remain. Partial `.pid`, `.status`, or output files must not be interpreted as valid ownership. Cleanup must be idempotent and safe after partial failure.
- Do not move retained temporary output into another unbounded cache, log directory, database blob, or event payload. If output must be bounded, reuse the existing output-cap policy rather than introducing a second arbitrary retention mechanism.
- Add privacy-safe observability for active background-job counts, bytes, cleanup counts/failures, and orphan reconciliation without logging commands, output, paths, environment values, or raw run/job IDs.
- Provide a one-time explicit cleanup command or documented operator procedure for existing legacy `.hatfield/tmp/bg` artifacts and stale background rows, but do not run bulk deletion against user data without separate user authorization.
- Add deterministic lowest-layer tests for foreground-only execution, explicit/automatic background transition, active-run listing/output/stop, cross-run isolation, completion, queued follow-up, terminal/cancel/failure/controller shutdown cleanup, partial publication, crash reconciliation, PID reuse, and owned process-tree teardown. Use deterministic process barriers and project helpers; no arbitrary sleeps, timing races, retry-until-green, test-only production APIs, or cases over 10 seconds.
- Update `.pi/reports/session-storage-file-io-audit.md`, background-process documentation, and relevant lifecycle comments with the temporary ownership boundary and cleanup semantics.
- Because this touches Bash/background execution, runtime terminalization, process ownership, and controller shutdown, run focused Castor tests, `castor deptrac`, `castor phpstan`, `castor cs-check`, controller replay, and full `castor check` before CODE-REVIEW. Inspect JUnit and remediate any case over 10 seconds.

## Workflow metadata
Status: ARCHIVE
Branch: task/storage-bound-background-process-storage-to-active-runs
Worktree: /home/ineersa/projects/agent-core-worktrees/storage-bound-background-process-storage-to-active-runs
Fork run: pib7msr7r23y
PR URL: https://github.com/ineersa/agent-core/pull/433
PR Status: merged
Started: 2026-08-25T22:55:00.050Z
Completed: 2026-08-27T19:53:52.799Z

## Work log
- Created: 2026-08-25T18:44:24.026Z

## Task workflow update - 2026-08-25T22:55:00.050Z
- Moved TODO → IN-PROGRESS.
- Created branch task/storage-bound-background-process-storage-to-active-runs.
- Created worktree /home/ineersa/projects/agent-core-worktrees/storage-bound-background-process-storage-to-active-runs.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/storage-bound-background-process-storage-to-active-runs.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/storage-bound-background-process-storage-to-active-runs.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/storage-bound-background-process-storage-to-active-runs.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/storage-bound-background-process-storage-to-active-runs.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/storage-bound-background-process-storage-to-active-runs/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/storage-bound-background-process-storage-to-active-runs.
- Summary: User started implementation. Scope is finalized active-run ownership for background-process rows/files, foreground-only non-persistence, cross-run isolation, lifecycle cleanup, safe crash reconciliation, legacy cleanup procedure, deterministic process-safety tests, and required docs/observability.

## Task workflow update - 2026-08-25T23:02:24.247Z
- Summary: Three read-only scouts completed full storage, lifecycle/safety, and test/docs/observability tracing. Current defect is confirmed: every Bash command calls BackgroundProcessManager::start(), which immediately launches sidecars and inserts a row; only later does acceptance set backgrounded_at. Storage is flat, rows retain command/path metadata, PID/PGID are trusted without boot/start identity, cleanup is global retention-based and apparently unused, controller shutdown only stops without deletion, session deletion ignores background jobs, and no startup reconciliation exists. Existing after-turn hook and terminal cleanup subscriber pattern are the narrow normal terminalization seam; permanent WorkerFailed bypass and controller/session deletion need explicit calls. BgStatus passes run ID but PID lookup is global-first. Existing manager/Bash/BgStatus tests and controller replay provide deterministic lowest-layer proof; no TUI source change is required, so this is not a TUI feature task and no new TmuxHarness journey is warranted unless implementation unexpectedly changes TUI behavior.

## Task workflow update - 2026-08-25T23:03:32.139Z
- Recorded fork run: 09ke4y9wtb98
- Summary: Single implementation fork launched with full scout-derived plan. It owns provisional-vs-published Bash execution, run/job DB ownership and process identity, safe scoped bg_status, atomic minimal artifacts, terminal/controller/session cleanup, startup reconciliation, aggregate observability, safe legacy cleanup surface, deterministic unit/integration/controller-replay proof, docs/settings/migration updates. Explicit safety: no real-data cleanup, no broad shell deletion, no root/HATFIELD signals. No TUI source change authorized; controller replay is the required integration layer.

## Task workflow update - 2026-08-25T23:06:59.996Z
- Recorded fork run: 09ke4y9wtb98
- Summary: Implementation fork refused due breadth and made no changes. Parent verified this is not an unresolved product decision: user already authorized the complete replacement and required one implementation fork. Relaunching with a narrower execution plan divided into sequential commits inside one actual implementation run; no partial unsafe end state will be accepted.

## Task workflow update - 2026-08-25T23:07:41.145Z
- Recorded fork run: cqqppvbpqlqa
- Summary: Replacement implementation fork launched after the first fork made no changes. Explicit staged decision supplied inside one implementation run: (1) new operational schema + safe process/artifact primitives, (2) provisional foreground + publish on acceptance, (3) one run cleanup coordinator across terminal/failure/controller/session seams, (4) legacy cleanup/docs/controller replay/validation. Intermediate commits allowed; final handoff must be coherent and complete.

## Task workflow update - 2026-08-25T23:09:29.834Z
- Recorded fork run: cqqppvbpqlqa
- Summary: Second implementation fork also refused due total scope and made no changes; worktree remains at b49cd0769. The explicit four-commit plan did not resolve execution-capacity refusal. Continuing now requires overriding the original single-implementation-fork constraint and using sequential narrowly scoped forks on the same worktree (schema/process primitives; provisional publication/status; lifecycle cleanup/reconciliation; legacy/docs/replay/validation), or splitting the task.

## Task workflow update - 2026-08-25T23:25:31.662Z
- Recorded fork run: 580sk530oif8
- Summary: Focused current-flow diagram saved untracked at /home/ineersa/projects/agent-core/.hatfield/tmp/background-process-current-flow.md (20,233 bytes). Diagnosis: measured bloat is caused by every Bash invocation persisting row + .pid/.log/.status before prompt acceptance, foreground paths never deleting them, and cleanupStale() having no production caller. The task over-scopes that leak into a full process-identity/schema/crash-recovery/runtime-terminalization redesign. Key product ambiguity: requiring literally zero temporary DB row before acceptance forces a provisional-handle rewrite because Bash foreground supervision currently uses the row id; if the intended invariant is zero retained state when foreground returns, existing backgrounded_at-null rows can serve as private provisional state and be synchronously deleted, yielding a much smaller fix.

## Task workflow update - 2026-08-27T14:28:38.311Z
- Summary: LATEST USER CLARIFICATION supersedes the broad redesign: solve accumulation with existing model. Treat `backgrounded_at IS NULL` rows as private provisional foreground supervision; `bg_status` and background completion operate only on `backgrounded_at IS NOT NULL`. Add a recurring scheduler job every 5 minutes to refresh and delete safely finished provisional rows plus their exact row-owned `.pid/.status/.log` artifacts; never delete running provisional rows. Keep existing accepted-background retention/lifecycle for now. Do not add job-id schema, boot/start identity, new publication handle, startup reconciliation, universal terminal coordinator, or process-safety redesign in this task. Remove/avoid broad PID orphan scanning in the recurrent cleanup; legacy finished provisional rows are cleaned by the same bounded sweep.

## Task workflow update - 2026-08-27T14:29:25.291Z
- Recorded fork run: uvrdq3obt912
- Summary: Focused implementation fork launched for latest simplified scope: existing null-backgrounded rows remain private foreground supervision; bg_status becomes accepted-only and run+PID query-scoped; existing Symfony Scheduler sweeps only finished null-backgrounded rows every 5 minutes after one-interval grace, deleting exact row-owned sidecars without global scans. No schema/process-identity/lifecycle rewrite.

## Task workflow update - 2026-08-27T14:41:38.531Z
- Recorded fork run: uvrdq3obt912
- Validation: castor test --filter='(BackgroundProcessProvisionalCleanupTaskTest|BackgroundProcessManagerTest|BgStatusToolTest)' — PASS, 42 tests/119 assertions; castor test --filter=BackgroundProcessProvisionalCleanupTaskTest — PASS, 5 tests/20 assertions; castor deptrac — PASS, 0 violations/errors; castor phpstan — PASS, 0 errors; castor cs-check — PASS, 0 files fixed; castor docs:validate — PASS, 16 docs; git diff --check — PASS
- Summary: Implementation complete at commit 3dd35cd7c (`fix(storage): sweep finished foreground process state`). Existing null-backgrounded rows remain private foreground supervision; bg_status list/log/stop now use accepted-only exact run+PID repository queries; a 300-second Symfony periodic task refreshes state then removes only finished null-backgrounded rows older than one interval and exact row-owned log/status/pid sidecars. No schema/process identity/controller lifecycle rewrite. Docs and storage audit updated.

## Task workflow update - 2026-08-27T15:48:50.076Z
- Recorded fork run: 1m52fptc4722
- Validation: castor test --filter=BackgroundProcessProvisionalCleanupTaskTest — PASS, 7 tests/27 assertions; castor test focused background group — PASS, 44 tests/126 assertions; castor deptrac — PASS; castor phpstan — PASS; castor cs-check — PASS; castor docs:validate — PASS; git diff --check — PASS; IDE diagnostics changed production/test files — 0 errors
- Summary: Reviewer blocker fixed in commit 16192ba39 (`fix(storage): handle provisional cleanup paths safely`): canonical containment now compares both directories canonically for symlinked storage, safely permits stale-row deletion when the configured directory and exact sidecars are all absent, remains fail-closed otherwise; scheduler/grace share one 300-second constant; audit table structure repaired.

## Task workflow update - 2026-08-27T16:02:46.931Z
- Validation: castor test — PASS, 4,863 tests/19,828 assertions, 38.1s; castor test:controller-replay — PASS, 6 tests/92 assertions, 21.3s; castor deptrac — PASS, 0 violations/errors; castor phpstan — PASS, 0 errors; castor cs-check — PASS, 0 files fixed; castor docs:validate — PASS, 16 docs; git diff --check — PASS; JUnit unit: 4,863 cases, max 4.017174s, 0 over 10s; JUnit controller replay: 6 cases, max 4.043509s, 0 over 10s
- Summary: Re-review after 16192ba39: APPROVED with no code blockers. Prior canonical-vs-lexical containment and missing-storage-directory blocker is closed; exact-sidecar containment remains fail-closed; specification fidelity confirmed against latest simplified scope. Focused local validation completed on final HEAD 16192ba39. Controller replay emitted one tracked-PID teardown warning for PID 158141, but immediate /proc inspection showed the PID had already exited (delayed teardown observation, no surviving process).

## Task workflow update - 2026-08-27T16:04:22.475Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (70.6s).
- Pushed task/storage-bound-background-process-storage-to-active-runs to origin.
- branch 'task/storage-bound-background-process-storage-to-active-runs' set up to track 'origin/task/storage-bound-background-process-storage-to-active-runs'.
- Created PR: https://github.com/ineersa/agent-core/pull/433
- Validation: Focused full test lane PASS: 4,863 tests/19,828 assertions; Controller replay PASS: 6 tests/92 assertions; Deptrac/PHPStan/CS/docs/diff checks PASS; JUnit 4,869 total cases across local lanes, none over 10s
- Summary: Reviewer APPROVED after one blocker-fix iteration. Final branch commits: 3dd35cd7c and 16192ba39. Simplified implementation keeps current foreground supervision, hides provisional rows from bg_status, query-scopes accepted operations, and adds exact periodic cleanup for old finished provisional rows.

## Task workflow update - 2026-08-27T16:04:43.726Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/433
- Updated PR Status: open
- Validation: move_task deterministic castor check — PASS in 70.6s; Full-gate JUnit: 4,882 cases across unit/controller-replay/TUI/llm-real, 0 over 10s; Full-gate max case: 6.618952s ControllerReplaySafeGuardApprovalTest; Worktree git status clean; origin task branch divergence 0/0
- Summary: Task-to-PR complete. PR #433 created after reviewer APPROVED and deterministic CODE-REVIEW gate passed. Worktree clean; pushed branch synchronized 0/0 with origin task branch.

## Task workflow update - 2026-08-27T16:22:07.697Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #433 feedback is actionable: collapse duplicate accepted-only manager methods into the existing list/read/stop API where safe, move provisional sweep orchestration from BackgroundProcessManager into the actual periodic task, and replace impossible empty run-id fallback with StackToolExecutionContextAccessor::requireCurrent().

## Task workflow update - 2026-08-27T16:22:43.614Z
- Recorded fork run: n9616dplhf3d
- Summary: PR feedback fix fork launched: remove duplicate listBackgrounded/readBackgroundedLogTail/stopBackgrounded APIs by tightening existing list/readLogTail/stop semantics; move the sweep implementation and 300s constant into BackgroundProcessProvisionalCleanupTask; use requireCurrent() in BgStatus instead of impossible empty run-id fallback. Record-ID/find/internal unscoped stop paths remain because foreground supervision, prompt status, and shutdown/reap still use them.

## Task workflow update - 2026-08-27T16:31:59.366Z
- Recorded fork run: n9616dplhf3d
- Validation: Focused background/Bash tests — PASS, 76 tests/250 assertions; castor deptrac — PASS; castor phpstan — PASS; castor cs-check — PASS; castor docs:validate — PASS; git diff --check — PASS; IDE diagnostics all changed files — 0 errors
- Summary: Implemented all three PR comments in commit 5140199ae (`refactor(storage): tighten background process boundaries`): removed duplicate accepted-only manager APIs and dead fetchAll; tightened existing list/readLogTail/session-scoped stop; moved sweep orchestration + interval constant into BackgroundProcessProvisionalCleanupTask; BgStatus now uses requireCurrent(). Record-ID/find/internal unscoped stop paths retained for foreground prompt/shutdown/reap.

## Task workflow update - 2026-08-27T16:46:13.657Z
- Recorded fork run: a05pvqbt5wb9
- Validation: castor docs:validate — PASS; castor cs-check — PASS; git diff --check — PASS; worktree clean
- Summary: Reviewer convention polish completed in 179330b93 (`docs(storage): clarify background process boundaries`): BgStatus comments now accepted-only, manager null-session stop comment precisely describes internal shutdown/reap boundary, and the audit footprint table is actually contiguous. No behavior changes.

## Task workflow update - 2026-08-27T16:53:32.297Z
- Validation: Focused background/Bash tests at 5140199ae — PASS, 76 tests/250 assertions; Deptrac/PHPStan/CS/docs/diff checks — PASS; 179330b93 docs/CS/diff checks — PASS; Final reviewer — APPROVED
- Summary: Final re-review after PR feedback commits 5140199ae + 179330b93: APPROVED with no blockers. All three inline comments fully resolved; duplicate APIs removed, retained global/record-ID paths have live internal foreground/prompt/shutdown/reap callers, periodic task owns sweep, requireCurrent fail-fast is correct, cleanup/containment/log privacy invariants preserved, and audit table/docs are accurate. Non-blocking suggestions deferred: stale pre-existing global-PID doc wording and minor readability comments.

## Task workflow update - 2026-08-27T16:55:00.111Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (75.7s).
- Pushed task/storage-bound-background-process-storage-to-active-runs to origin.
- branch 'task/storage-bound-background-process-storage-to-active-runs' set up to track 'origin/task/storage-bound-background-process-storage-to-active-runs'.
- PR already exists: https://github.com/ineersa/agent-core/pull/433
- Validation: Focused background/Bash tests PASS (76 tests/250 assertions); Deptrac, PHPStan, CS, docs, diff checks PASS; Final reviewer APPROVED
- Summary: PR feedback iteration complete. Commits 5140199ae and 179330b93 resolve all three owner comments; final reviewer APPROVED. Existing PR #433 will be updated.

## Task workflow update - 2026-08-27T17:07:53.701Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/433
- Updated PR Status: open
- Validation: Deterministic castor check on final HEAD 179330b93 — PASS in 75.7s; Full-gate JUnit: 4,882 cases, 0 over 10s; Max case: 6.662533s ControllerReplaySafeGuardApprovalTest; Worktree clean; origin task branch divergence 0/0
- Summary: PR #433 updated with feedback commits 5140199ae and 179330b93. Final reviewer APPROVED; branch pushed/synchronized; PR body refreshed and replies posted to all three owner comments explaining exact resolution.

## Task workflow update - 2026-08-27T17:15:40.609Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Owner requested removal of the dead retention path. cleanupStale() has no production caller; remove it and its exclusively-dead stale/orphan query/file helpers, retentionSeconds config/default/docs, and the test that only proved unreachable behavior. Existing periodic provisional sweep remains the sole scheduled cleanup.

## Task workflow update - 2026-08-27T17:16:01.807Z
- Recorded fork run: oqj5ck77c5o8
- Summary: Dead-code cleanup fork launched: remove cleanupStale and exclusively-owned orphan/stale helpers, background-process retention config/default/docs, and unreachable test/helper parameters. Preserve output-cap retention and live periodic provisional cleanup.

## Task workflow update - 2026-08-27T17:27:41.705Z
- Recorded fork run: okdnnmp76qav
- Summary: Reviewer REQUEST CHANGES on d63fd45f4: one stale BackgroundProcess PID docblock still claimed removed 24h retention and referenced nonexistent BackgroundProcessCleanupTask. Focused fix fork launched; also remove/reconcile orphaned ProcessLifecycle section banner.

## Task workflow update - 2026-08-27T17:37:43.143Z
- Summary: Owner extended finalized lifecycle: accepted background-process rows and exact sidecars must live only for the controller session. Add typed controller session-starting and session-shutdown events; a proper event listener cleans accepted process data on both. Shutdown is the normal lifecycle boundary; startup/resume repairs crash leftovers. Cleanup must include controller-owned child run IDs, stop unfinished processes before deletion, preserve fail-closed exact-sidecar validation, and leave the 5-minute provisional scheduler unchanged. No retention setting.

## Task workflow update - 2026-08-27T17:42:00.009Z
- Recorded fork run: zb78p6oycc5e
- Summary: Implementation fork launched for typed ControllerSessionStartingEvent + ControllerSessionShutdownEvent and dedicated accepted-background cleanup listener. Events carry minimal session ID; listener cleans parent/child accepted rows at startup/shutdown using record-ID stop and exact fail-closed sidecar deletion; provisional scheduler remains unchanged.

## Task workflow update - 2026-08-27T18:05:39.873Z
- Recorded fork run: r0pu2lb0ra6n
- Summary: Final reviewer REQUEST CHANGES: implementation is spec-correct, but controller event dispatch seam lacks asserted process-layer regression proof. Fix fork launched to add deterministic startup+shutdown controller cleanup proof and remove newly unreachable poller empty-scope branch.

## Task workflow update - 2026-08-27T18:28:38.119Z
- Recorded fork run: pib7msr7r23y
- Summary: Re-review APPROVED and prior blocker closed. Small polish fork launched for reviewer-noted test hermeticity (HOME isolation), ProcessLifecycle storage-dir reuse, removal of no-op test overrides, and correction of stale flat sidecar layout wording before final gate.

## Task workflow update - 2026-08-27T18:38:36.822Z
- Validation: castor test PASS: 4,864 tests, 19,840 assertions, 33.3s; Unit JUnit: 4,864 cases, 0 over 10s; max 6.459568s; castor test:controller-replay PASS: 7 tests, 105 assertions; lifecycle case 3.827823s; castor deptrac PASS: 0 violations/errors; castor phpstan PASS: 0 errors; castor cs-check PASS: 0 files fixed; castor docs:validate PASS: 16 documents; git diff --check PASS; worktree clean
- Summary: Final re-review APPROVED at HEAD 010078d42. All prior blockers closed: dead retention removed; typed controller start/shutdown events and dedicated accepted-background cleanup listener; parent+child scope; record-ID stop; exact fail-closed sidecar deletion; real controller startup/shutdown process proof; hermetic seeding. Worktree clean and ready for deterministic gate/PR update.

## Task workflow update - 2026-08-27T18:39:59.805Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (70.5s).
- Pushed task/storage-bound-background-process-storage-to-active-runs to origin.
- branch 'task/storage-bound-background-process-storage-to-active-runs' set up to track 'origin/task/storage-bound-background-process-storage-to-active-runs'.
- PR already exists: https://github.com/ineersa/agent-core/pull/433
- Validation: Final reviewer APPROVED at 010078d42; Focused castor test: 4,864 tests / 19,840 assertions, 0 cases >10s; Controller replay: 7 tests / 105 assertions, lifecycle case 3.827823s; Deptrac/PHPStan/CS/docs/diff all PASS
- Summary: PR feedback iteration complete. Removed unreachable 24h retention surface; added controller lifecycle start/shutdown events and accepted-background cleanup listener; cleanup includes parent/child runs, stops by record ID, deletes exact sidecars then rows, and startup repairs crash leftovers. Added deterministic real-controller lifecycle proof and hermetic test seeding. Final reviewer APPROVED.

## Task workflow update - 2026-08-27T19:53:52.799Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/storage-bound-background-process-storage-to-active-runs: ide_close_project returned isError.
- Merged task/storage-bound-background-process-storage-to-active-runs into integration checkout.
- Auto-merging .pi/reports/session-storage-file-io-audit.md
Merge made by the 'ort' strategy.
 .pi/reports/session-storage-file-io-audit.md       |   6 +-
 config/hatfield.defaults.yaml                      |   3 -
 docs/background-processes.md                       |  18 +-
 docs/settings.md                                   |   1 -
 src/CodingAgent/Config/BackgroundProcessConfig.php |   7 +-
 src/CodingAgent/Entity/BackgroundProcess.php       |  14 +-
 .../Entity/BackgroundProcessRepository.php         |  59 +++--
 .../BackgroundProcessCompletionPoller.php          |  66 +++--
 ...ndProcessControllerSessionLifecycleListener.php | 123 ++++++++++
 .../Event/ControllerSessionShutdownEvent.php       |  21 ++
 .../Event/ControllerSessionStartingEvent.php       |  21 ++
 .../Runtime/Controller/HeadlessController.php      |  35 ++-
 .../Tool/BackgroundProcess/ProcessLifecycle.php    |  85 +++----
 .../Tool/BackgroundProcess/ProcessStore.php        |  52 ++--
 src/CodingAgent/Tool/BackgroundProcessManager.php  | 183 +++-----------
 .../BackgroundProcessProvisionalCleanupTask.php    |  61 +++++
 src/CodingAgent/Tool/BgStatusTool.php              |  18 +-
 ...ocessControllerSessionLifecycleListenerTest.php | 153 ++++++++++++
 .../ControllerReplayBackgroundProcessSeeder.php    | 219 +++++++++++++++++
 ...rollerBackgroundProcessLifecycleProcessTest.php | 120 ++++++++++
 .../Tool/BackgroundProcessManagerTest.php          |  96 ++++----
 ...BackgroundProcessProvisionalCleanupTaskTest.php | 265 +++++++++++++++++++++
 tests/CodingAgent/Tool/BashToolTest.php            |  16 +-
 tests/CodingAgent/Tool/BgStatusToolTest.php        |  27 ++-
 24 files changed, 1297 insertions(+), 372 deletions(-)
 create mode 100644 src/CodingAgent/Runtime/Controller/BackgroundProcessControllerSessionLifecycleListener.php
 create mode 100644 src/CodingAgent/Runtime/Controller/Event/ControllerSessionShutdownEvent.php
 create mode 100644 src/CodingAgent/Runtime/Controller/Event/ControllerSessionStartingEvent.php
 create mode 100644 src/CodingAgent/Tool/BackgroundProcessProvisionalCleanupTask.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/BackgroundProcessControllerSessionLifecycleListenerTest.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/E2E/Support/ControllerReplayBackgroundProcessSeeder.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/HeadlessControllerBackgroundProcessLifecycleProcessTest.php
 create mode 100644 tests/CodingAgent/Tool/BackgroundProcessProvisionalCleanupTaskTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/storage-bound-background-process-storage-to-active-runs.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/storage-bound-background-process-storage-to-active-runs.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User confirmed PR #433 merged. Merge task branch into integration checkout, sync remote, and clean task worktree.

## Task workflow update - 2026-08-27T19:55:48.725Z
- Validation: LLM_MODE=true castor check PASS in 148.6s; test: 4,905 tests / 20,056 assertions; controller replay: 7 tests / 105 assertions; TUI: 8 tests / 59 assertions; llm-real: 5 tests / 30 assertions; deptrac/phpstan/cs/docs/catalog all PASS; JUnit: 4,925 cases total, 0 over 10s; max 5.852583s; QA artifact integrity PASS; exact-run leak check PASS; llama-proxy cache stable 391→391; git status clean (main ahead of origin by local integration merge history)
- Summary: Post-merge validation completed on integration main after PR #433 merge. Task worktree removed; JetBrains project close reported a degraded non-fatal error, but worktree and IDEA exclusions were cleaned successfully. Integration checkout is clean.

## Task workflow update - 2026-08-29T16:09:39.020Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
