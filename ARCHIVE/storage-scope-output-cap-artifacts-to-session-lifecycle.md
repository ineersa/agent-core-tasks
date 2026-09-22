# STORAGE- Scope output-cap artifacts to session lifecycle

## Goal
Fix lifecycle ownership for `.hatfield/tmp/output-cap`. The measured snapshot contains 103 files / 44.8 MiB. Existing `OutputCap` already performs age-based cleanup once on first use, with default `tools.output_cap.retention: 86400`, but it does not implement session lifecycle cleanup: production filenames default to a date prefix, the optional `sessionPrefix` is not dynamically bound to the active run, and `cleanup()` only deletes files older than the global retention cutoff.

Finalized behavior:
- Output-cap files are ephemeral active-session artifacts, not canonical session history.
- On session resume/start, clear any prior output-cap artifacts owned by that session before accepting new work.
- On clean quiescent session exit, clear that session's output-cap artifacts.
- Cancellation, unrecoverable failure, explicit session deletion, and controller shutdown must use the same owned cleanup where lifecycle semantics require ending the session.
- Existing age-based cleanup remains an orphan/crash fallback, not the primary ownership mechanism.
- Project logs are fine and are out of scope.

## Acceptance criteria
- Trace every `OutputCap::persist()` / `capIfNeeded()` caller, DI lifetime, worker/controller process boundary, path construction, event/tool-result reference, and existing cleanup invocation before choosing the owning lifecycle hook.
- Introduce real runtime ownership by session/run ID for output-cap artifacts. A static configured filename prefix or date prefix is not sufficient. Prefer a run-scoped directory beneath the configured output-cap root so cleanup removes only one session's artifacts and cannot race another active run.
- At session resume/start, delete only pre-existing output-cap artifacts owned by that session before new tool execution begins. Perform cleanup once at the owning lifecycle boundary, not on every poll/message/tool call.
- At clean quiescent session exit—no queued/in-flight continuation and controller ownership is ending—delete that session's output-cap directory. Apply equivalent owned cleanup on cancellation, unrecoverable failure, explicit session deletion, and controlled controller shutdown without deleting another active run's files.
- If a controller or worker crashes before cleanup, retain the existing age-based first-use cleanup as a bounded orphan fallback. Verify its default 24-hour retention remains enabled and document the distinction between lifecycle cleanup and stale cleanup.
- Do not introduce a cleanup daemon, database table, generic artifact repository, new user setting, or global directory wipe. Reuse existing OutputCap configuration/storage and session lifecycle hooks.
- Treat saved output-cap paths referenced from events/tool notices as explicitly ephemeral. After resume/start cleanup, historical notices may point to removed temp artifacts; canonical event payload/history must not depend on those files for replay, repair, transcript reconstruction, or state correctness. Verify this rather than assuming it.
- Keep output-cap persistence behavior itself unchanged unless a defect is proven: this task owns file lifetime, not cap thresholds, model-facing notices, or which tool outputs are capped.
- Make directory creation and owned recursive cleanup safe against path traversal, symlinks, partial files, concurrent workers, and repeated/idempotent cleanup. Never remove anything outside the configured output-cap root.
- Add privacy-safe observability for session-owned artifact count/bytes removed and cleanup failures without logging prompts, tool output, paths, environment values, or raw session IDs.
- Provide a separately authorized one-time operator cleanup procedure for existing legacy date-prefixed files that cannot be attributed to a session. Do not automatically bulk-delete legacy artifacts as part of normal startup.
- Add deterministic lowest-layer tests for run-scoped persistence, start/resume cleanup, clean-exit cleanup, cancellation/failure/deletion/shutdown paths, cross-run isolation, repeated cleanup, crash-stale fallback, symlink/path safety, and canonical replay independence. No sleeps, timing windows, retries-until-green, test-only production APIs, or cases over 10 seconds.
- Update `.pi/reports/session-storage-file-io-audit.md`, `docs/settings.md`, and `docs/session-storage.md` with active-session ownership, ephemeral references, lifecycle cleanup, and 24-hour orphan fallback.
- Because lifecycle hooks may touch runtime/controller behavior, run focused Castor tests, `castor deptrac`, `castor phpstan`, `castor cs-check`, controller replay where applicable, and full `castor check` before CODE-REVIEW. Inspect JUnit and remediate any individual case over 10 seconds.

## Workflow metadata
Status: ARCHIVE
Branch: task/storage-scope-output-cap-artifacts-to-session-lifecycle
Worktree: /home/ineersa/projects/agent-core-worktrees/storage-scope-output-cap-artifacts-to-session-lifecycle
Fork run: 0zw7krgqu4se
PR URL: https://github.com/ineersa/agent-core/pull/436
PR Status: merged
Started: 2026-08-27T20:11:32.089Z
Completed: 2026-08-27T22:49:01.484Z

## Work log
- Created: 2026-08-25T18:47:22.612Z

## Task workflow update - 2026-08-27T20:11:32.090Z
- Moved TODO → IN-PROGRESS.
- Created branch task/storage-scope-output-cap-artifacts-to-session-lifecycle.
- Created worktree /home/ineersa/projects/agent-core-worktrees/storage-scope-output-cap-artifacts-to-session-lifecycle.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/storage-scope-output-cap-artifacts-to-session-lifecycle.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/storage-scope-output-cap-artifacts-to-session-lifecycle.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/storage-scope-output-cap-artifacts-to-session-lifecycle.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/storage-scope-output-cap-artifacts-to-session-lifecycle.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/storage-scope-output-cap-artifacts-to-session-lifecycle/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/storage-scope-output-cap-artifacts-to-session-lifecycle.
- Summary: Starting implementation. Reuse the newly introduced typed controller session lifecycle events to clean output-cap artifacts at both startup/resume and shutdown, analogous to accepted background-process cleanup, without adding retention settings or new lifecycle surface.

## Task workflow update - 2026-08-27T20:18:14.962Z
- Summary: Scout inventory complete. OutputCap writes date/static-prefix files from the central tool-result processor and LLM transform fallback; workers share one configured root but separate service instances. Saved paths are canonical notice metadata only—replay/repair/projection never dereference files. Existing controller starting/shutdown events run before consumers and after consumer shutdown respectively and already resolve parent+child run IDs. Final interpretation from 'same as background': cleanup occurs at controller ownership start/resume, controller shutdown, and explicit non-active session deletion; cancellation/failure/completion alone remain resumable and therefore are not cleanup boundaries. Hard crashes recover on next start or 24h first-use stale fallback. Legacy date-prefixed cleanup will be a documented, explicitly authorized manual procedure only; no automatic bulk deletion/new command.

## Task workflow update - 2026-08-27T20:19:22.440Z
- Recorded fork run: nojz94woz8pe
- Summary: Implementation fork launched with finalized minimal design: SHA-256 run-scoped directories, explicit run ID through existing tool/LLM hook seams, Symfony per-scope locking, safe lifecycle/stale cleanup, existing controller start/shutdown event listener, explicit session deletion dispatching the same shutdown event, privacy-safe metrics, legacy manual procedure, and deterministic lowest-layer tests. No new setting/event/repository/daemon/command.

## Task workflow update - 2026-08-27T20:31:06.244Z
- Recorded fork run: nojz94woz8pe
- Validation: OutputCapTest PASS: 24 tests / 78 assertions; OutputCapLlmTransformHookTest PASS: 11 / 65; OutputCapToolResultProcessorContractTest PASS: 7 / 53; HatfieldSessionStoreTest PASS: 31 / 133; controller-replay PASS: 7 / 105; PHPStan PASS: 0 errors; Deptrac BLOCKED as expected by event layer before semantic move
- Summary: First implementation fork committed partial core at cc8c5625d and stopped on Deptrac: AppSession cannot depend on events declared under AppRuntimeInternals. Architecture resolution selected: semantically move the two existing ControllerSessionStartingEvent/ControllerSessionShutdownEvent declarations into `CodingAgent/Session/Event` (preserving class names; no new event), because AppRuntimeInternals is already allowed to depend on AppSession and session deletion must dispatch a session-lifecycle event. Do not weaken Deptrac or add a bridge. Follow-up fork will finish safety/test coverage and remove production-only test conveniences.

## Task workflow update - 2026-08-27T20:31:42.354Z
- Recorded fork run: 0k0yue2jaw8y
- Summary: Follow-up implementation fork launched. It will move the existing controller session lifecycle event declarations into the AppSession layer (same classes, no new event), close Deptrac cleanly, remove WIP test-convenience production defaults, harden scoped cleanup/locking/symlink behavior, complete lifecycle/deletion/replay/observability/stale-fallback tests and docs, and run focused Castor validation.

## Task workflow update - 2026-08-27T20:44:43.788Z
- Recorded fork run: 0k0yue2jaw8y
- Validation: Focused OutputCap + HatfieldSessionStore tests PASS: 82 tests / 362 assertions / 5.125s; Controller replay PASS: 7 tests / 105 assertions; Deptrac PASS: 0 violations/errors; PHPStan PASS: 0 errors; CS check PASS: 0 files fixed; Docs validate PASS: 16 documents; git diff --check PASS; IDE diagnostics PASS on key changed files
- Summary: Implementation complete at commits cc8c5625d + 853464eb4. Existing controller lifecycle events were semantically moved into AppSession (same classes; no duplicate/bridge), closing Deptrac. New output-cap files use explicit run identity under SHA-256 run scopes with shared Symfony locks; lifecycle start/shutdown and explicit session deletion clean parent+registered-child scopes; legacy files remain lifecycle-untouched; 24h first-use stale fallback handles exact legacy files and stale run scopes. Cleanup is canonical-root/symlink-safe, idempotent, privacy-safe logged, and historical saved_path metadata remains explicitly ephemeral. Cancellation/completion/failure remain resumable non-boundaries. Worktree clean; ready for user-requested task-to-pr phase, but no reviewer/full check/PR has been run yet.

## Task workflow update - 2026-08-27T21:03:22.701Z
- Summary: Task-to-PR reviewer returned REQUEST CHANGES. Critical blocker: required EventDispatcherInterface added to HatfieldSessionStore breaks ~65 direct two-argument test constructions across 41 files, missed by focused tests/PHPStan test exclusion. Additional required fixes: canonicalize per-scope lock identity consistently for symlinked configured roots; document historical custom session_prefix files as operator-owned/non-automatic; correct shutdown event docblock for explicit deletion dispatch. Core design/spec fidelity and security were otherwise approved.

## Task workflow update - 2026-08-27T21:03:44.450Z
- Recorded fork run: 9y9a5jeozlq1
- Summary: Reviewer-fix fork launched for all blockers: update every direct HatfieldSessionStore test construction with a real dispatcher, unify OutputCap lock identity under canonical roots, correct lifecycle event docs and custom-prefix legacy procedure, then run full unit + controller replay + static/docs validation and JUnit timing audit.

## Task workflow update - 2026-08-27T21:22:50.115Z
- Validation: Final reviewer APPROVED at 2baf75207; castor test PASS: 4,913 tests / 20,098 assertions / 32.0s; Unit JUnit: 4,913 cases, 0 over 10s; max 4.376022s; castor test:controller-replay PASS: 7 tests / 105 assertions / 27.3s; Controller JUnit: 7 cases, 0 over 10s; max 4.873289s; castor deptrac PASS: 0 violations/errors; castor phpstan PASS: 0 errors; castor cs-check PASS: 0 files fixed; castor docs:validate PASS: 16 documents; git diff --check PASS; worktree clean
- Summary: Final reviewer APPROVED after commit 2baf75207 closed all blockers. All 67 direct HatfieldSessionStore test constructions pass a required real dispatcher; no production default. OutputCap lock identity is canonical-root consistent across persist/lifecycle/stale cleanup with deterministic symlink proof. Lifecycle event docs and historical custom-prefix procedure corrected. Existing process event-dispatch and nonexistent saved_path projection tests provide non-duplicated proof mapping. One pre-existing background-process immediate isAlive ParaTest timing race was observed once by fork, isolated and full rerun passed; reviewer adjudicated unrelated/non-blocking and recommends separate follow-up.

## Task workflow update - 2026-08-27T21:24:18.138Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (73.7s).
- Pushed task/storage-scope-output-cap-artifacts-to-session-lifecycle to origin.
- branch 'task/storage-scope-output-cap-artifacts-to-session-lifecycle' set up to track 'origin/task/storage-scope-output-cap-artifacts-to-session-lifecycle'.
- Created PR: https://github.com/ineersa/agent-core/pull/436
- Validation: Reviewer APPROVED at 2baf75207; Full unit: 4,913 tests / 20,098 assertions; 0 cases >10s; Controller replay: 7 tests / 105 assertions; 0 cases >10s; Deptrac/PHPStan/CS/docs/diff all PASS
- Summary: Implementation and review complete. Output-cap artifacts are explicit SHA-256 run scopes; existing Session-layer start/shutdown events clean parent+registered-child scopes at start/resume, controlled shutdown, and explicit deletion; 24h exact stale fallback remains; legacy/custom-prefix artifacts are handled only by documented authorized procedures. Final reviewer APPROVED.

## Task workflow update - 2026-08-27T22:28:49.178Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Addressing user PR feedback: restore/update useful non-obvious rationale and invariant comments in OutputCap.php while keeping signature/type/control-flow noise removed. No behavior or public surface change.

## Task workflow update - 2026-08-27T22:28:53.831Z
- Ownership: owner=fork; fork_run=pending; revision=2baf75207; scope=restore useful OutputCap rationale/invariant comments only, with no behavior changes; outcome=assigned; commit=none

## Task workflow update - 2026-08-27T22:29:13.477Z
- Recorded fork run: 0zw7krgqu4se
- Ownership: owner=fork; fork_run=0zw7krgqu4se; revision=2baf75207; scope=restore useful OutputCap rationale/invariant comments only, with no behavior changes; outcome=assigned; commit=none

## Task workflow update - 2026-08-27T22:31:18.420Z
- Validation: castor cs-check PASS: 0 files fixed; git diff --check PASS; IDE diagnostics OutputCap.php: 0 problems; Patch inspection confirms comment/PHPDoc-only hunks; worktree clean
- Summary: User-requested PR feedback implemented in comment-only commit c63c7120b. Restored concise current rationale for cap classifiers, resolveCapPath precedence/exclusions, read-specific notice behavior, run-scoped persistence, first-use stale cleanup timing, and generic notice fallback. No executable changes.
- Ownership: owner=fork; fork_run=0zw7krgqu4se; revision=2baf75207; scope=restore useful OutputCap rationale/invariant comments only, with no behavior changes; outcome=completed; commit=c63c7120b51c1271be3c5d30e49ee931c1c0088a

## Task workflow update - 2026-08-27T22:35:12.606Z
- Validation: Reviewer APPROVED c63c7120b; zero blockers; Comment-stripped PHP token streams identical across delta; Final castor cs-check PASS: 0 files fixed; Final git diff --check PASS; Worktree clean; branch ahead of origin task branch by one comment-only commit
- Summary: Independent reviewer APPROVED comment-restoration delta 2baf75207..c63c7120b. Reviewer tokenized both revisions with comments/whitespace stripped and confirmed all 1,874 executable PHP tokens are byte-identical. Restored claims were traced to current config, classification, persistence, lifecycle listener, and notice behavior; no stale session-prefix behavior or boilerplate restored.
- Review: role=reviewer; target_revision=c63c7120b51c1271be3c5d30e49ee931c1c0088a; scope=OutputCap comment/PHPDoc-only PR feedback delta; decision=APPROVED; blockers=none

## Task workflow update - 2026-08-27T22:36:46.660Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (86.5s).
- Pushed task/storage-scope-output-cap-artifacts-to-session-lifecycle to origin.
- branch 'task/storage-scope-output-cap-artifacts-to-session-lifecycle' set up to track 'origin/task/storage-scope-output-cap-artifacts-to-session-lifecycle'.
- PR already exists: https://github.com/ineersa/agent-core/pull/436
- Validation: Reviewer APPROVED c63c7120b; castor cs-check PASS: 0 files fixed; git diff --check PASS; Comment-stripped PHP tokens identical
- Summary: Addressed user PR feedback by restoring concise, current OutputCap rationale and invariant comments. Delta is provably comment/PHPDoc-only with identical executable token stream; independent reviewer APPROVED.

## Task workflow update - 2026-08-27T22:49:01.484Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/storage-scope-output-cap-artifacts-to-session-lifecycle: ide_close_project returned isError.
- Merged task/storage-scope-output-cap-artifacts-to-session-lifecycle into integration checkout.
- Auto-merging tests/CodingAgent/Tool/OutputCapToolResultProcessorContractTest.php
Merge made by the 'ort' strategy.
 .pi/plans/toolbox-design-plan.md                   |   1 -
 .pi/reports/session-storage-file-io-audit.md       |   1 +
 config/hatfield.defaults.yaml                      |   9 +-
 docs/session-storage.md                            |   8 +
 docs/settings.md                                   |   4 +-
 .../Hook/TransformContextHookInterface.php         |   2 +-
 .../SymfonyAi/LlmPlatformAdapter.php               |   6 +-
 src/CodingAgent/Config/OutputCapConfig.php         |   5 +-
 ...ndProcessControllerSessionLifecycleListener.php |   4 +-
 .../Runtime/Controller/HeadlessController.php      |   4 +-
 ...OutputCapControllerSessionLifecycleListener.php |  39 ++
 .../Event/ControllerSessionShutdownEvent.php       |   5 +-
 .../Event/ControllerSessionStartingEvent.php       |   2 +-
 src/CodingAgent/Session/HatfieldSessionStore.php   |   5 +
 src/CodingAgent/Tool/OutputCap.php                 | 416 ++++++++++++---------
 src/CodingAgent/Tool/OutputCapLlmTransformHook.php |   8 +-
 .../Tool/OutputCapToolResultProcessor.php          |   3 +-
 .../Handler/ToolBatchCollectorDurableTest.php      |   2 +-
 .../ToolBatchCollectorFinalizedRedeliveryTest.php  |   2 +-
 .../Pipeline/ToolCallResultHandlerTest.php         |   2 +-
 .../Infrastructure/SymfonyAi/LlamaCppSmokeTest.php |   1 +
 .../SymfonyAi/PlatformIntegrationTest.php          |   8 +-
 .../Infrastructure/SymfonyAi/TraceReplayTest.php   |   1 +
 .../Agent/Artifact/AgentArtifactRegistryTest.php   |   1 +
 .../Agent/Artifact/AgentChildRunEventStoreTest.php |   1 +
 .../Agent/Artifact/AgentChildRunStoreTest.php      |   1 +
 .../Execution/SessionAwareModelResolverTest.php    |   1 +
 .../Recovery/DeferredSubagentBatchRecoveryTest.php |   1 +
 .../CLI/Session/SessionCacheInspectCommandTest.php |   1 +
 .../Config/ModelSelectionServiceTest.php           |   1 +
 ...ocessControllerSessionLifecycleListenerTest.php |   4 +-
 ...utCapControllerSessionLifecycleListenerTest.php |  99 +++++
 tests/CodingAgent/Session/AggregateResumeTest.php  |   1 +
 .../Session/HatfieldSessionStoreTest.php           |  40 +-
 .../Session/Repair/SessionRepairServiceTest.php    |   3 +
 .../SessionAgentArtifactPathResolverTest.php       |   1 +
 .../Session/SessionRunEventStoreSequencingTest.php |   1 +
 .../Session/SessionRunEventStoreTest.php           |   1 +
 tests/CodingAgent/Session/SessionRunStoreTest.php  |   2 +
 .../Session/SessionToolBatchStoreTest.php          |   4 +-
 .../ToolBatchSnapshotCleanupHookSubscriberTest.php |   2 +-
 tests/CodingAgent/Tool/BashToolTest.php            |  11 +-
 tests/CodingAgent/Tool/BgStatusToolTest.php        |   9 +-
 .../Tool/OutputCapLlmTransformHookTest.php         |  49 +--
 tests/CodingAgent/Tool/OutputCapTest.php           | 181 +++++++--
 .../OutputCapToolResultProcessorContractTest.php   |  53 ++-
 tests/CodingAgent/Tool/ReadFileToolTest.php        |  10 +-
 .../Application/SessionInitializerReplayTest.php   |   3 +-
 tests/Tui/Application/SessionInitializerTest.php   |  12 +-
 .../Tui/Application/TuiSessionCompositionTest.php  |   1 +
 tests/Tui/Listener/CancelListenerTest.php          |   1 +
 tests/Tui/Listener/CompletionListenerTest.php      |   9 +-
 tests/Tui/Listener/ExportCommandHandlerTest.php    |   3 +-
 tests/Tui/Listener/ExportCommandRegistrarTest.php  |   1 +
 tests/Tui/Listener/ImagePasteInputVirtualTest.php  |   2 +
 tests/Tui/Listener/ModelCommandHandlerTest.php     |   1 +
 .../Listener/RenameSessionCommandHandlerTest.php   |  10 +-
 .../Listener/ResumeSessionCommandHandlerTest.php   |  10 +-
 .../Listener/SubmitListenerDispatchRuntimeTest.php |   2 +
 tests/Tui/Picker/ModelPickerControllerTest.php     |   1 +
 tests/Tui/Picker/SessionPickerControllerTest.php   |   4 +
 .../Picker/SubagentLivePickerControllerTest.php    |   1 +
 tests/Tui/Runtime/TuiRuntimeEventApplierTest.php   |   4 +-
 tests/Tui/Screen/TuiExportCommandVirtualTest.php   |   1 +
 .../ResumeSessionInitializerTestFactory.php        |   3 +-
 tests/Tui/Support/SubagentLiveScenarioHarness.php  |   1 +
 .../Tui/Support/TuiRuntimeContextBuilderTrait.php  |   1 +
 .../Tui/Support/TuiSessionServicesFactoryTrait.php |   1 +
 68 files changed, 779 insertions(+), 309 deletions(-)
 create mode 100644 src/CodingAgent/Runtime/Controller/OutputCapControllerSessionLifecycleListener.php
 rename src/CodingAgent/{Runtime/Controller => Session}/Event/ControllerSessionShutdownEvent.php (75%)
 rename src/CodingAgent/{Runtime/Controller => Session}/Event/ControllerSessionStartingEvent.php (89%)
 create mode 100644 tests/CodingAgent/Runtime/Controller/OutputCapControllerSessionLifecycleListenerTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/storage-scope-output-cap-artifacts-to-session-lifecycle.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/storage-scope-output-cap-artifacts-to-session-lifecycle.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #436 state MERGED at 2026-08-27T22:48:34Z; Pre-merge integration checkout clean and synchronized with origin/main
- Summary: PR #436 merged on GitHub as ef2fcdc2df604d5b745ef1813408afe8ea566f41. Moving task to DONE and synchronizing integration checkout.

## Task workflow update - 2026-08-27T22:51:02.589Z
- Validation: LLM_MODE=true castor check PASS in 187.1s; Unit: 4,914 tests / 20,117 assertions; Controller replay: 7 tests / 105 assertions; TUI: 8 tests / 59 assertions; LLM-real: 5 tests / 30 assertions; Deptrac/PHPStan/CS/docs/catalog all PASS; QA artifact integrity PASS; exact-run leak check PASS; llama-proxy cache stable 391→391; JUnit: 4,934 cases total, zero over 10s; max 8.009537s; Integration working tree clean; local main ahead of origin/main by 2 workflow merge commits
- Summary: Task completed after PR #436 merge. Post-merge deterministic full gate passed on integration checkout. Worktree removed; JetBrains project close reported a non-blocking degradation. Integration working tree is clean; local main contains the workflow's local task merge plus remote PR merge and is two commits ahead of origin/main.

## Task workflow update - 2026-08-29T16:09:39.023Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
