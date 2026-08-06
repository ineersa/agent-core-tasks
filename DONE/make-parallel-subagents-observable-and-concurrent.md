# Make parallel subagent execution observable and genuinely concurrent

## Goal
In session 33, parallel scouts appeared to alternate control rather than execute concurrently. Both eventually completed, so the earlier claim that the parent run was permanently stuck is retracted and must not be used as the bug thesis. The remaining problem is that either execution is bottlenecked/serialized or TUI progress is too sparse to distinguish real overlap from serialization.

Instrument the actual child lifecycle and establish timestamped evidence before changing scheduling. Inspect concurrency limits, process spawning, shared locks, stdout/result polling, tool-call aggregation, and any single-threaded await loop. Keep child prompts tightly scoped during reproduction; prior broad scouts consumed excessive context and obscured the timing signal.

This task should fix a real scheduler bottleneck if found and make per-child progress visible enough that users can see which children are queued, running, producing progress, and terminal.

## Acceptance criteria
- For a deterministic two-child barrier fixture with configured concurrency >=2, prove both child processes enter the running region before either is released; wall time and lifecycle timestamps must demonstrate overlap rather than alternating serialized execution.
- Identify and remove any unnecessary global lock, sequential await, blocking stdout read, result-file poll, or single-child critical section that prevents independent children from progressing concurrently.
- Respect configured concurrency and backpressure: never spawn beyond the limit, and queued children transition to running as slots free.
- Emit bounded, privacy-safe per-child lifecycle/progress events with child ID, agent name, queued/running/terminal status, and timestamps; do not stream raw child prompts or full outputs as progress.
- The TUI presents simultaneous child states rather than only the last child update, so real concurrency is observable without reading logs.
- Collect terminal results deterministically for all launched children and distinguish success, failure, and cancellation. Do not claim or test a session-33 permanent parent hang; both children completed.
- Add the lowest-correct runtime scheduler proof plus virtual TUI proof for simultaneous child status. Avoid live-provider timing as the concurrency oracle.
- Run all QA through Castor, including focused runtime/TUI tests and full `castor check` before CODE-REVIEW.

## Workflow metadata
Status: DONE
Branch: task/make-parallel-subagents-observable-and-concurrent
Worktree: /home/ineersa/projects/agent-core-worktrees/make-parallel-subagents-observable-and-concurrent
Fork run: 86stjtz0tez4
PR URL: https://github.com/ineersa/agent-core/pull/321
PR Status: merged
Started: 2026-07-25T15:00:42.398Z
Completed: 2026-07-26T23:31:20.354Z

## Work log
- Created: 2026-07-23T18:52:18.411Z

## Task workflow update - 2026-07-23T19:15:46.859Z
- Summary: Exact root cause confirmed. Batch launch is non-blocking and starts all three children within ~14ms, but `HeadlessController` launches exactly one Messenger `llm` consumer while every child `ExecuteLlmStep` routes to the shared `llm` queue. Session-33 provider-call intervals have zero overlap and rotate child A→B→C with ~10-14ms queue handoffs. Tool work has multiple workers and can overlap. TUI therefore truthfully shows only the child holding the sole LLM slot as active, creating alternating-control behavior. Shared `completed_at` values are batch artifact-finalization time, not actual child completion times.
- First provider intervals: child 3d451a76 18:09:11.087-17.719, child 7f3b1cc5 18:09:17.732-28.308, child a9d95724 18:09:28.321-32.821, then the same serial rotation repeats.

## Task workflow update - 2026-07-23T19:57:05.229Z
- Summary: Recommended design decision: use the existing `ConsumerSupervisor::launchMultiple()` fixed-pool pattern and start four LLM consumers at controller startup by default. Multiple consumers on the Doctrine `llm` transport are safe; each message is claimed once and results remain serialized through `run_control`. Do not spawn consumers dynamically when a subagent batch starts: that adds launch latency, oversubscription races, shutdown complexity, and destroys process-local provider/WebSocket cache reuse. Prefer a typed LLM execution concurrency setting (default 4) rather than silently conflating provider concurrency with tool-only semantics; apply a conservative bound and document provider connection/rate-limit implications.
- Existing supervisor already tracks, restarts, and shuts down transport#instance workers independently. Four matches current common parallelism and bounds process-local provider connections.

## Task workflow update - 2026-07-23T20:08:18.323Z
- Summary: User approved the fixed startup pool approach: launch four LLM consumers by default using the existing configurable worker/settings mechanism, allowing operators to raise or lower the count. Do not dynamically spawn workers per batch.

## Task workflow update - 2026-07-23T20:58:31.122Z
- Summary: Resource caveat added: before making four LLM consumers the default, measure per-worker idle RSS, startup CPU/time, active-turn CPU, and duplicated process-local provider/cache state. Four remains the target concurrency, but implementation must avoid blindly multiplying an unexpectedly heavy worker and should document/tune the setting from measured budgets.

## Task workflow update - 2026-07-25T15:00:42.398Z
- Moved TODO → IN-PROGRESS.
- Created branch task/make-parallel-subagents-observable-and-concurrent.
- Created worktree /home/ineersa/projects/agent-core-worktrees/make-parallel-subagents-observable-and-concurrent.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/make-parallel-subagents-observable-and-concurrent.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/make-parallel-subagents-observable-and-concurrent.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/make-parallel-subagents-observable-and-concurrent.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/make-parallel-subagents-observable-and-concurrent.
- Summary: Starting implementation of proven single-LLM-consumer bottleneck. Use configurable fixed startup pool (default 4), measure resource footprint before finalizing default, prove overlap with deterministic barrier fixture, and surface bounded per-child lifecycle state.

## Task workflow update - 2026-07-25T15:06:23.667Z
- Summary: Two narrow scouts completed. Concurrency root is isolated to HeadlessController launching one `llm` consumer; ConsumerSupervisor::launchMultiple already provides independent supervision/restart/shutdown and stdout multiplexing. Implement configurable bounded fixed pool, default 4. Deterministic proof should use a subprocess barrier so two real ExecuteLlmStep handlers enter provider invocation before release. Existing parallel child snapshots/catalog/picker already preserve simultaneous states; smallest observability gap is reserved children: no initial progress delivery and unprojected reserved rows are mislabeled running instead of pending. No new runtime event type is needed.
- Scout 1: add typed bounded LLM worker count setting; replace launch('llm') with launchMultiple; transport/supervisor already safe. Use flock-based subprocess barrier precedent for overlap proof.
- Scout 2: simultaneous running/terminal child rows already exist end-to-end and have tests. Fix only initial reserved→pending snapshot emission and strengthen virtual picker assertion; keep payload bounded/privacy-safe.

## Task workflow update - 2026-07-25T15:09:50.613Z
- Summary: User decision: keep LLM execution concurrency as a separate setting with default 4, and also reduce `agents.max_agents` default from 8 to 4. Rationale: more than four simultaneous children has no compelling use case and creates excessive token/provider/resource spend. Existing `tools.execution.max_parallelism` remains tool-only.

## Task workflow update - 2026-07-25T15:09:59.001Z
- Decision: separate bounded LLM consumer concurrency setting, default 4.
- Decision: change built-in `agents.max_agents` default and documented/project example default from 8 to 4; explicit user overrides remain supported by existing validation.
- Decision: `tools.execution.max_parallelism` remains tool-only and must not size LLM consumers.

## Task workflow update - 2026-07-25T15:15:00.775Z
- Recorded fork run: 9ckbj65gyljm
- Summary: Initial implementation fork stopped dirty after correctly recognizing LLM pool sizing must not live under tools.execution. Relaunched continuation fork 9ckbj65gyljm to preserve valid pending-progress edits, move configuration to typed `runtime.llm_worker_count` default 4, reduce `agents.max_agents` default to 4, replace an implementation-mirroring reflection test with real deterministic overlap/controller proof, measure resources, validate, and commit cleanly.

## Task workflow update - 2026-07-25T23:07:49.076Z
- Recorded fork run: zu9nox0epkl0
- Validation: Intermediate full castor test PASS 4527/15792.; Intermediate castor deptrac PASS 0 violations; castor phpstan PASS 0 errors.; Resource evidence: controlled 1-LLM controller ~201MB total vs 4-LLM ~434MB; ~77.7MB per LLM worker, +233MB total; startup ~1.6s both.
- Summary: Continuation fork 9ckbj65gyljm completed the production redesign and resource measurement but exhausted context before final CS/controller replay/commit. Worktree remains dirty and uncommitted. Launched narrow completion fork zu9nox0epkl0 to inspect test quality/idempotency, diagnose the one controller-replay failure, finish all Castor validation, commit, and leave clean.

## Task workflow update - 2026-07-25T23:16:31.139Z
- Recorded fork run: zu9nox0epkl0
- Validation: Focused new/changed tests PASS 69 tests / 468 assertions.; castor test PASS 4527 tests / 15792 assertions.; castor test:controller-replay PASS 9 tests / 136 assertions; replay fixture isolation explicitly uses one LLM worker because FIFO cursor is process-local.; castor deptrac PASS 0 violations.; castor phpstan PASS 0 errors.; castor cs-check PASS; git diff --check clean.; Resource measurement: one LLM worker controller+LLM RSS ~201MB; four ~434MB; ~77.7MB per LLM worker and +233MB total; startup ~1.6s for both.
- Summary: Implementation complete and committed as 991240a3e7c1dacbcee8cb0636df34f13c88c6d8. Adds typed `runtime.llm_worker_count` default 4 (1..8), reduces `agents.max_agents` default to 4, launches fixed LLM consumer pool, and emits one initial bounded pending snapshot for reserved children. Deterministic barrier proves two real ExecuteLlmStep workers enter PlatformInterface::invoke concurrently before release; controlled controller process proof confirms configured worker topology. Worktree clean. Ready for user-triggered task-to-PR phase.

## Task workflow update - 2026-07-25T23:36:33.102Z
- Summary: First PR-readiness reviewer verdict REQUEST CHANGES. Architecture/config/concurrency barrier approved, but blockers: explicit user-required replay-backed TmuxHarness proof is absent; HeadlessController process test can orphan tagged worker children on forced parent kill. Additional actionable findings: progress delivery can append duplicate snapshots under cross-process race because CAS occurs after append; heavy process test belongs in controller-replay lane; misleading lifecycle comments; simplify/fail-closed worker-count resolution; quantify documented resource cost.
- Reviewer positively verified runtime.llm_worker_count wiring, separate tool pool, controller replay single-worker fixture isolation, deterministic real ExecuteLlmStep barrier, and happy-path pending revision semantics.
- All actionable blocker/edge/convention/simplify findings will be addressed in a fork before re-review.

## Task workflow update - 2026-07-26T00:11:21.792Z
- Summary: Second reviewer at HEAD 99126d250 verified every prior blocker resolved: real replay-backed TmuxHarness pending+running proof, exact owned process-tree teardown with survivor assertion, claim-before-append progress race handling and later-revision recovery, process lane isolation, fail-closed worker count, resource docs. Verdict APPROVE WITH SUGGESTIONS solely for two dead-code leftovers and minor diagnostics/test cleanup. Launched fork uiqbpbjo7bay to address all sensible items before final review.
- Review-fix commit 99126d250 added the required terminal E2E, process lifecycle safety, progress revision claiming/recovery, count validation, and resource note.
- Reviewer found no critical/bug/edge/security blockers on the current design; final small cleanup fork removes deprecated zero-caller API, duplicate cmdline predicate, aligns APP_DEBUG, and removes inert SQL test update.

## Task workflow update - 2026-07-26T00:19:27.771Z
- Validation: castor test PASS: 4533 tests, 15813 assertions (31.3s).; castor deptrac PASS: 0 violations, 0 errors.; castor phpstan PASS: 0 errors.; castor cs-check PASS: 0 files fixed.; castor test:tui PASS: 38 tests, 200 assertions (139.7s), including replay-backed TmuxHarness pending/running child proof.; Focused cleanup validation PASS: 20 tests, 297 assertions.; No focused test:llm-real was required before the gate: provider schemas/model routing/prompts/stream conversion are unchanged; deterministic CODE-REVIEW castor check will run its standard llm-real lane.
- Summary: Final independent reviewer APPROVED HEAD cda2c632be5298904680e6679f95020b7080bb88 with no actionable findings. All prior blockers and cleanup suggestions verified resolved. Branch clean and ready for deterministic CODE-REVIEW gate.
- Implementation commits: 991240a3e concurrent LLM pool + pending progress; 99126d250 reviewer blockers (Tmux proof, process-tree teardown, progress claim race/recovery, fail-closed count, docs); cda2c632 cleanup of deprecated API/test leftovers.
- Reviewer progression: REQUEST CHANGES → APPROVE WITH SUGGESTIONS → APPROVED at current HEAD.

## Task workflow update - 2026-07-26T00:25:25.690Z
- Validation: CODE-REVIEW castor check: all code lanes reached cache guard; gate failed because llama-proxy grew 146→148.; First castor test:llm-real warmup: FAIL 2 tests (OutputCapReadFileControllerTest, ShellFollowUpLiveE2eTest), cache 148→156.; Second warmup: FAIL OutputCapReadFileControllerTest after tool-call events but before tool execution start; cache 156→160.; castor clean:cleanup:workers:list: no stale QA workers.
- Summary: First deterministic CODE-REVIEW gate failed only on llama-proxy cache growth (146→148). Required warmup then exposed a real test-isolation issue under the new default: `castor test:llm-real` repeatedly failed under parallel live E2Es while each isolated controller spawned four LLM consumers (first run OutputCap + ShellFollowUp, second OutputCap), and proxy cache grew 148→156→160. No worker leaks. This mirrors the already-fixed replay FIFO/resource isolation issue; production concurrency proofs remain separate. Launching fork to force one LLM consumer in the shared live ControllerE2E isolated settings, preserving production default 4.

## Task workflow update - 2026-07-26T00:40:35.724Z
- Validation: castor test:llm-real PASS twice: 13 tests / 175 assertions; llama-proxy cache stable 166→166 both runs.; castor test:controller-replay PASS: 10 tests / 136 assertions.; castor test:tui PASS: 38 tests / 200 assertions.; castor test final retry PASS: 4533 tests / 15813 assertions. Initial attempt hit known unrelated SQLite contention threshold flake at 136.5ms vs 140ms; unchanged test passed on full retry.; castor deptrac PASS: 0 violations.; castor phpstan PASS: 0 errors.; castor cs-check PASS: clean.; castor clean:cleanup:workers:list: no stale workers after live suite.
- Summary: Final delta aab519f867f4521b9b16bc5a13196e703988c954 independently APPROVED. Shared live ControllerE2eTestCase now forces one LLM worker for isolated ParaTest scenarios; production default 4 and concurrency topology proofs are untouched. LLM proxy cache warmed and stable at 166 entries. Branch clean; reattempting CODE-REVIEW gate.

## Task workflow update - 2026-07-26T00:43:18.312Z
- Validation: Second gate cache guard only: 166→167.; Post-gate warmup PASS 13 tests / 175 assertions in 34.4s; proxy cache 167→167 stable.
- Summary: Second CODE-REVIEW transition gate passed code/tests but cache guard found one previously unwarmed check-only request (166→167). No code failure. Post-gate `castor test:llm-real` PASS 13/175 with proxy cache stable 167→167; retrying deterministic gate without code changes.

## Task workflow update - 2026-07-26T00:51:01.978Z
- Validation: castor test:llm-real PASS 13/175 in 28.3s, cache 168→168.; Exact generation-preflight payload returned HTTP 200 twice with cache remaining 168→168.
- Summary: Third gate attempt pending. Prior deterministic gate again passed test lanes but cache guard found one different previously uncached request (167→168). Exact preflight request was manually confirmed cache-hit twice at 168. Required full llm-real warmup now PASS 13/175 with cache stable 168→168. No code changes.

## Task workflow update - 2026-07-26T00:57:55.100Z
- Validation: Gate TUI-only failure: TuiTreeCommandE2eTest::testTreeEnterRewindsTranscriptToEarlierTurn capture still contained BANG_REWIND_07C.; castor clean:cleanup:workers:list: no stale workers.; Standalone castor test:tui retry PASS 38 tests / 200 assertions in 118.7s.; llama-proxy cache remains 168 entries.
- Summary: Third CODE-REVIEW gate cleared the warmed proxy cache guard but hit one TUI timing/render-settle failure in pre-existing TuiTreeCommandE2eTest: abandoned marker remained in capture immediately after rewind. No workers leaked and branch unchanged. Standalone full `castor test:tui` immediately PASS 38/200; proxy cache remains 168. Retrying deterministic gate.

## Task workflow update - 2026-07-26T01:01:01.237Z
- Validation: Gate test:controller-replay exit 124; empty JUnit and only PHPUnit header before kill.; Standalone controller-replay at current HEAD previously PASS 10/136 in 88.2s.; All other fourth-gate lanes passed; llama-proxy cache stayed 168.; No stale workers reported after prior attempts.
- Summary: Fourth CODE-REVIEW gate cleared proxy and all other lanes but controller-replay hit its 90s shell timeout before PHPUnit output. This task adds a real controller topology process test to the controller-replay lane; standalone suite now passes 10/136 in ~88s, leaving ~2s idle margin and no margin under concurrent gate load. Existing `.castor/tasks.php` comment is stale (8 tests, ~59s). Launching narrow fork to update the bounded lane budget/comment; not retrying a structurally insufficient timeout.

## Task workflow update - 2026-07-26T01:05:32.598Z
- Validation: castor test:controller-replay PASS 10/136 in 79.3s after timeout change.; castor phpstan PASS 0 errors; castor cs-check PASS; git diff --check clean.; castor clean:cleanup:workers:list: no stale workers.; Final reviewer verdict APPROVED.
- Summary: Final reviewer APPROVED HEAD 17b8c40ca9feda62f0e8ac7bd6210c9c233b1067. The only delta raises controller-replay gate shell budget 90→150s after this task grew the sequential lane to 10 tests/136 assertions and observed 79–88s standalone. Reviewer verified bounded 165s Castor hard stop, accurate group count, unchanged unrelated budgets, preserved per-test waits and leak guard. Retrying CODE-REVIEW gate.

## Task workflow update - 2026-07-26T01:08:56.414Z
- Validation: Gate llm-real-only failure: ShellFollowUpLiveE2eTest::testShellThenFollowUpOnCompletedRun; follow-up observed command.ack + run.completed only, controller still running.; Source inspection confirms shell phase waits only through bash tool completion; follow-up code suppresses second collection whenever any run.completed is present, allowing delayed shell terminal to contaminate the follow-up phase.; llama-proxy cache stable 168; no stale workers.
- Summary: Fifth CODE-REVIEW gate cleared expanded controller-replay budget and cache, but exposed a deterministic race in existing ShellFollowUpLiveE2eTest under gate load. Shell phase stops at tool_execution.completed, leaving its trailing run.completed unread. After sending follow_up, follow-up collection can consume command.ack + the shell's delayed run.completed, misclassify that stale terminal as follow-up terminal, skip further collection, and fail with no assistant response. This is test phase-boundary contamination, not a live provider/run failure. Launching narrow fork to await/drain shell terminal before follow-up and preserve issue #183 thesis.

## Task workflow update - 2026-07-26T01:29:44.537Z
- Validation: Focused ShellFollowUpLiveE2eTest PASS 3 consecutive runs: 2 tests / 21 assertions each.; Full castor test:llm-real PASS 13 tests / 175 assertions in 35.4s; proxy cache 168→168.; castor phpstan PASS 0 errors; castor cs-check PASS; git diff --check clean.; No stale workers. Final reviewer APPROVED.
- Summary: Reviewer APPROVED HEAD 83393f85b4733ae6370e0f9073d503f4babf4686. Test-only ShellFollowUp collector now rejects delayed pre-ack shell terminals and requires matching follow-up ack + assistant evidence + parent terminal after ack; true issue #183 dead follow-up still fails. Full live suite green and cache stable; retrying CODE-REVIEW gate.

## Task workflow update - 2026-07-26T01:32:09.567Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (131.4s).
- Pushed task/make-parallel-subagents-observable-and-concurrent to origin.
- branch 'task/make-parallel-subagents-observable-and-concurrent' set up to track 'origin/task/make-parallel-subagents-observable-and-concurrent'.
- Created PR: https://github.com/ineersa/agent-core/pull/321

## Task workflow update - 2026-07-26T01:32:18.965Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/321
- Updated PR Status: open
- Validation: Deterministic castor check PASS (131.4s), including unit/integration, controller replay, TUI replay, llm-real, Deptrac, PHPStan, CS, cache guard, report integrity, and leak guard.
- Summary: Task-to-PR complete. Deterministic castor check passed in 131.4s at HEAD 83393f85b4733ae6370e0f9073d503f4babf4686; branch pushed and PR #321 opened.

## Task workflow update - 2026-07-26T17:27:59.912Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #321 is CONFLICTING/DIRTY against current main. No review comments are pending. Reopening task to resolve merge conflicts via fork, validate, re-review, and update the existing PR.

## Task workflow update - 2026-07-26T17:28:26.022Z
- Recorded fork run: u5hx6vjpdddg
- Summary: Launched merge-resolution fork u5hx6vjpdddg. It will fetch and normally merge origin/main (no rebase/reset), resolve conflicts semantically while preserving both current-main behavior and all task invariants, run focused/full Castor validation, and leave a clean local merge commit for re-review.

## Task workflow update - 2026-07-26T17:35:59.737Z
- Recorded fork run: u5hx6vjpdddg
- Validation: castor test PASS on retry: 4533 tests / 15813 assertions; first run hit known SQLite timing flake, focused middleware PASS 4/19.; castor test:controller-replay PASS 10/136 in 75.7s.; castor test:tui PASS 38/200 in 115.0s.; castor test:llm-real PASS 13/175 in 29.0s; proxy cache stable 168→168.; castor deptrac PASS 0 violations; castor phpstan PASS 0 errors; castor cs-check clean; git diff --check clean.; castor clean:cleanup:workers:list: no stale workers.; Post-merge reviewer: APPROVED.
- Summary: Merge conflicts resolved in normal merge commit 32fe3aa1aee053cc6b9885ed13b09e4128112779 against origin/main d885b97332d3d6f359ad8607aab80d91b4be6001. Only conflict was `.castor/tasks.php`; resolution keeps 150s for this branch's 10-test controller-replay suite while incorporating main's 120s/9-test history. Final reviewer APPROVED: merge graph is 0 behind, conflict scope exact, no task/main behavior dropped, bounded hard-stop preserved. Branch clean; updating PR #321 through CODE-REVIEW gate.

## Task workflow update - 2026-07-26T17:38:16.139Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (125.5s).
- Pushed task/make-parallel-subagents-observable-and-concurrent to origin.
- branch 'task/make-parallel-subagents-observable-and-concurrent' set up to track 'origin/task/make-parallel-subagents-observable-and-concurrent'.
- PR already exists: https://github.com/ineersa/agent-core/pull/321

## Task workflow update - 2026-07-26T17:38:39.739Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/321
- Updated PR Status: open
- Validation: Post-merge deterministic castor check PASS (125.5s).; GitHub PR #321: CLEAN / MERGEABLE at head 32fe3aa1a, base d885b9733.
- Summary: PR #321 conflict resolution complete. Merge commit 32fe3aa1a merged current origin/main; deterministic castor check passed in 125.5s; branch pushed. GitHub now reports mergeable=MERGEABLE and mergeStateStatus=CLEAN.

## Task workflow update - 2026-07-26T18:01:13.522Z
- Summary: Read all six new inline comments on PR #321 and traced the affected paths. Candid review: pending-state semantics are valid; the nullable progress-delivery dependency is an unjustified manual-test accommodation and should be required; `deliverInitialProgressIfNeeded` is opaque naming and its recovery comment is factually stale after claim-before-append; the subprocess barrier is legitimate process-overlap proof but, combined with the separate controller topology test, still does not constitute one end-to-end Messenger work-distribution test. TUI/controller replay intentionally force one LLM worker because their replay fixture cursor is process-local. No implementation started pending user review of the explanation.
- PR comment classification: actionable — remove nullable progressDelivery/default-null, rename/clarify initial pending delivery, correct append-failure recovery comment; likely strengthen to one true multi-consumer message-distribution integration proof. Valid as-is — Pending before first child lifecycle projection; Pending→Running/terminal on observed child events/recovery; single-worker replay isolation; use of a child-process flock barrier as a lower-level concurrency proof.

## Task workflow update - 2026-07-26T18:11:50.162Z
- Summary: User rejected the pending-child observability scope: the requested fix was the proven single-LLM-consumer bottleneck, not a redesign of subagent progress delivery. Root process error acknowledged: the assistant-created task/acceptance criteria conflated concurrency with observability, Scout 2 expanded the implementation into initial pending snapshots, and reviewer instructions then compounded it with progress-delivery CAS/Tmux requirements. No correction implemented yet. Recommended correction is to strip all pending/progress-delivery production changes and their tests from PR #321, retain only configurable LLM worker pool/default-4, explicit max_agents default-4 decision, required deterministic replay/live isolation, and minimal genuine multi-worker proof.

## Task workflow update - 2026-07-26T18:13:12.505Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User confirmed PR #321 must be narrowed to the actual requested feature: configurable fixed LLM worker pool/default 4, max_agents default 4, config/docs, genuine concurrency proof, and deterministic replay isolation only. Reopening to remove all unrequested pending-child observability/progress-delivery changes and associated tests.

## Task workflow update - 2026-07-26T18:14:20.807Z
- Recorded fork run: q2cjpmbzoede
- Summary: Launched cleanup fork q2cjpmbzoede to remove the entire unrequested pending-child observability/progress-delivery redesign and unrelated ShellFollowUp test change from PR #321. It will retain only worker-pool config/runtime/docs, max_agents default 4, deterministic topology+overlap proofs, and required single-worker test-fixture isolation; validate via focused/full Castor commands and commit without pushing.

## Task workflow update - 2026-07-26T18:39:33.793Z
- Summary: Strict cleanup reviewer returned REQUEST CHANGES despite approving production scope cleanup. Actionable minimality findings: remove the 168-line direct-worker barrier because it proves only two OS processes can overlap and exercises unchanged code/no Messenger; shrink the 397-line topology test by reusing ControllerE2eTestCase process lifecycle infrastructure; deduplicate the 221-line RuntimeConfig test fixture; use injected Symfony ValidatorInterface instead of constructing a throwaway validator inside AppConfig. Reviewer accepts actual controller topology (N real messenger:consume llm children) plus trusted Symfony Messenger transport semantics as the valid proof. Thirty one-line TUI isolation wraps and 150s sequential lane budget are justified.

## Task workflow update - 2026-07-26T18:40:01.117Z
- Recorded fork run: 6dhr0cvjoq31
- Summary: Launched minimality follow-up fork 6dhr0cvjoq31: remove the meaningless direct-worker barrier, collapse topology test onto shared ControllerE2eTestCase lifecycle infrastructure, deduplicate RuntimeConfig tests, inject Symfony ValidatorInterface, retain honest controller N-consumer topology proof only, and revalidate without pushing.

## Task workflow update - 2026-07-26T19:23:45.708Z
- Summary: Strict re-review APPROVED HEAD 3296caad4: worker-pool-only scope exact; observability/ShellFollowUp gone; topology proof sufficient; shared teardown safe; no actionable correctness issues. Reviewer found one benign zero-caller protected wrapper `discoverControllerChildPids()` left after refactor. Because user explicitly requested strict minimality and project forbids compatibility leftovers, removing it before PR update.

## Task workflow update - 2026-07-26T19:23:56.223Z
- Recorded fork run: q40561u2mwa4
- Summary: Launched final tiny cleanup fork q40561u2mwa4 to delete the sole zero-caller wrapper left by the topology-test refactor, run focused Castor validation, and commit without pushing.

## Task workflow update - 2026-07-26T19:28:50.819Z
- Summary: Final delta reviewer APPROVED 669317f61, but the deletion exposed `ControllerE2eTestCase::isProcessOwnedForTeardown()` as another zero-caller method (verified sole definition, no callers). Performing one systematic pass over branch-added shared-base helpers to remove all remaining zero-caller leftovers rather than leave dead compatibility code.

## Task workflow update - 2026-07-26T19:29:02.165Z
- Recorded fork run: 86stjtz0tez4
- Summary: Launched systematic final dead-helper fork 86stjtz0tez4 to remove verified zero-caller isProcessOwnedForTeardown and audit every branch-added shared controller test helper/field for callers before the PR update.

## Task workflow update - 2026-07-26T19:33:25.397Z
- Recorded fork run: 86stjtz0tez4
- Validation: Focused config/controller/worker tests PASS 33/131 at 3296caad4.; castor test PASS 4525/15753; castor test:controller-replay PASS 10/135 (latest 80.0s); castor test:tui PASS 37/190 on retry; castor test:llm-real PASS 13/175 with cache 168→168.; castor deptrac PASS 0 violations; castor phpstan PASS 0 errors; castor cs-check clean; git diff --check clean; no stale workers.; Strict reviewer progression after scope cleanup: REQUEST CHANGES → APPROVED; final two dead-helper deltas individually APPROVED.
- Summary: Worker-pool-only cleanup finalized at HEAD 26921e8d51fa302006fb7103a0f9133b900ad81f. Final reviewer APPROVED with no actionable findings. All pending/progress observability and unrelated ShellFollowUp changes are gone; meaningless direct-worker barrier deleted; topology test reduced 397→140 lines on shared controller lifecycle; zero-caller ownership wrappers removed; RuntimeConfig tests deduplicated; AppConfig uses injected Symfony validator. Updating existing PR #321 with narrowed scope.

## Task workflow update - 2026-07-26T19:36:53.745Z
- Validation: Gate qa-20260726-193337-230272-814646c3: test lane only failure MessengerSqliteImmediateTransactionMiddlewareTest threshold 79.49ms < 140ms; all other lanes green.; Focused `castor test --filter=MessengerSqliteImmediateTransactionMiddlewareTest` PASS 4 tests / 19 assertions.; No stale QA workers.
- Summary: First narrowed-scope CODE-REVIEW gate failed only on pre-existing MessengerSqliteImmediateTransactionMiddlewareTest timing oracle: measured ~79.5ms vs required 140ms. Focused test immediately passed 4/19; full castor test at 16 ParaTest workers reproduced the same unrelated contention flake (~82ms). No task code path touches this middleware/test, all other gate lanes passed, and no workers leaked. Retrying deterministic 4-worker gate without adding unrelated test changes to PR #321.

## Task workflow update - 2026-07-26T19:39:47.668Z
- Validation: Gate qa-20260726-193705-241084-08fed399: sole failure MessengerSqliteImmediateTransactionMiddlewareTest 136.74ms < 140ms; all other lanes passed.; No stale QA workers.
- Summary: Second narrowed-scope gate again failed only the unrelated SQLite wall-clock threshold test, now by 3.26ms (136.74ms vs 140ms). All other lanes green; no leaks. This confirms the test itself is a nondeterministic timing oracle under parallel gate scheduling, not a PR #321 regression. Making one final unchanged gate attempt; if it repeats, PR #321 will remain IN-PROGRESS behind a separate baseline-test blocker rather than absorb unrelated changes.

## Task workflow update - 2026-07-26T19:42:10.764Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (131.6s).
- Pushed task/make-parallel-subagents-observable-and-concurrent to origin.
- branch 'task/make-parallel-subagents-observable-and-concurrent' set up to track 'origin/task/make-parallel-subagents-observable-and-concurrent'.
- PR already exists: https://github.com/ineersa/agent-core/pull/321

## Task workflow update - 2026-07-26T19:42:55.253Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/321
- Updated PR Status: open
- Validation: Final deterministic castor check PASS (131.6s).; PR #321 CLEAN/MERGEABLE; head 26921e8d5; net 50 files, +802/-95 vs main.; All six user inline comments answered with corresponding code removal/proof clarification.
- Summary: PR #321 updated at HEAD 26921e8d51fa302006fb7103a0f9133b900ad81f with strict worker-pool-only scope. Deterministic castor check passed in 131.6s; branch pushed; PR retitled and body rewritten to remove observability claims. Replied to all six inline comments with exact resolution: initial pending/progress paths removed, nullable dependency removed, subagent state changes restored to main, meaningless barrier deleted, real 3-consumer controller topology proof retained. GitHub reports CLEAN/MERGEABLE.

## Task workflow update - 2026-07-26T23:31:20.354Z
- Moved CODE-REVIEW → DONE.
- Merged task/make-parallel-subagents-observable-and-concurrent into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/tasks.php                                  |  22 ++-
 config/hatfield.defaults.yaml                      |  10 ++
 config/services.yaml                               |   7 +
 docs/agents.md                                     |   2 +-
 docs/settings.md                                   |  38 ++++-
 docs/tool-execution.md                             |   8 +-
 src/CodingAgent/Config/AgentsConfig.php            |   4 +-
 src/CodingAgent/Config/AppConfig.php               |  31 ++++
 src/CodingAgent/Config/RuntimeConfig.php           |  49 ++++++
 .../Runtime/Controller/HeadlessController.php      |  32 +++-
 tests/CodingAgent/Agent/Tool/SubagentToolTest.php  |   6 +-
 tests/CodingAgent/Config/AgentsConfigTest.php      |  11 +-
 tests/CodingAgent/Config/AppConfigTest.php         |  10 ++
 tests/CodingAgent/Config/CompactionConfigTest.php  |   6 +-
 .../Config/RuntimeConfigLlmWorkerCountTest.php     | 186 +++++++++++++++++++++
 .../Runtime/Controller/ConsumerSupervisorTest.php  |  25 +++
 .../Controller/E2E/ControllerE2eTestCase.php       | 150 ++++++++++++-----
 .../Controller/E2E/ControllerReplayE2eTestCase.php |   5 +
 ...dlessControllerLlmWorkerCountResolutionTest.php |  75 +++++++++
 .../HeadlessControllerLlmWorkerPoolProcessTest.php | 140 ++++++++++++++++
 tests/Tui/E2E/BashBackgroundE2eTestSupport.php     |   2 +-
 tests/Tui/E2E/CancelStickinessE2eTest.php          |   2 +-
 .../Tui/E2E/TuiAskHumanOverlayMarkdownE2eTest.php  |   2 +-
 tests/Tui/E2E/TuiAutoCompactionCancelE2eTest.php   |   2 +-
 tests/Tui/E2E/TuiAutoCompactionE2eTest.php         |   2 +-
 tests/Tui/E2E/TuiCompactCommandE2eTest.php         |   2 +-
 tests/Tui/E2E/TuiCompactHeaderE2eTest.php          |   2 +-
 tests/Tui/E2E/TuiE2eDatabaseEnv.php                |  22 +++
 tests/Tui/E2E/TuiFileRewindE2eTest.php             |   2 +-
 tests/Tui/E2E/TuiImagePasteE2eTest.php             |   2 +-
 tests/Tui/E2E/TuiJourneyE2eTest.php                |   2 +-
 tests/Tui/E2E/TuiMarkdownRenderE2eTest.php         |   2 +-
 tests/Tui/E2E/TuiOutputCapNoticeE2eTest.php        |   2 +-
 tests/Tui/E2E/TuiProviderErrorE2eTest.php          |   2 +-
 tests/Tui/E2E/TuiQueuedSteerE2eTest.php            |   2 +-
 tests/Tui/E2E/TuiRepairCommandE2eTest.php          |   2 +-
 tests/Tui/E2E/TuiResumeModelRestoreE2eTest.php     |   2 +-
 tests/Tui/E2E/TuiResumeSessionSwitchE2eTest.php    |   2 +-
 .../TuiRichTranscriptProductValidationE2eTest.php  |   2 +-
 tests/Tui/E2E/TuiStartupSnapshotTest.php           |   2 +-
 .../Tui/E2E/TuiStartupTranscriptConfigE2eTest.php  |   2 +-
 .../Tui/E2E/TuiStatusRowReasoningNoticeE2eTest.php |   2 +-
 .../TuiSubagentChildHitlCancellationE2eTest.php    |   2 +-
 .../Tui/E2E/TuiSubagentLiveChildExportE2eTest.php  |   2 +-
 tests/Tui/E2E/TuiSubagentLiveViewE2eTest.php       |   2 +-
 tests/Tui/E2E/TuiSubagentProgressE2eTest.php       |   2 +-
 tests/Tui/E2E/TuiToolOutputE2eTest.php             |   2 +-
 tests/Tui/E2E/TuiTranscriptRenderE2eTest.php       |   2 +-
 tests/Tui/E2E/TuiTreeCommandE2eTest.php            |   2 +-
 tests/Tui/E2E/TuiUsageCommandE2eTest.php           |   2 +-
 50 files changed, 802 insertions(+), 95 deletions(-)
 create mode 100644 src/CodingAgent/Config/RuntimeConfig.php
 create mode 100644 tests/CodingAgent/Config/RuntimeConfigLlmWorkerCountTest.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/HeadlessControllerLlmWorkerCountResolutionTest.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/HeadlessControllerLlmWorkerPoolProcessTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/make-parallel-subagents-observable-and-concurrent.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/make-parallel-subagents-observable-and-concurrent.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Pre-merge deterministic castor check PASS in 131.6s at task HEAD 26921e8d51fa302006fb7103a0f9133b900ad81f.
- Summary: PR #321 merged on GitHub at 2026-07-26T23:30:51Z as merge commit 9ac2d8fa1b8f754682bf7dee4c8b558a9cc0f66e. Final scope: configurable fixed LLM Messenger consumer pool only; no subagent observability/progress redesign.

## Task workflow update - 2026-07-26T23:36:41.772Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/321
- Updated PR Status: merged
- Validation: PR #321 verified MERGED at 2026-07-26T23:30:51Z, merge commit 9ac2d8fa1b8f754682bf7dee4c8b558a9cc0f66e.; Post-merge `LLM_MODE=true castor check` run qa-20260726-233136-379110-66ac751e: deptrac, controller-replay 10/135, TUI 37/190, llm-real 13/175, phpstan, cs-check, cache guard 168→168, artifact integrity, and leak check all PASS; unit lane sole failure was SubagentLivePickerControllerTest selection-order assumption.; Focused `castor test --filter=SubagentLivePickerControllerTest` PASS 17 tests / 56 assertions.; Post-merge retry qa-20260726-233357-390080-98c15a0b reproduced the same sole unit failure; all other lanes PASS, cache stable 168→168, no leaks.; Integration git status clean (`main...origin/main [ahead 2]`); task worktree removed.
- Summary: DONE workflow complete: task branch merged into integration checkout, remote sync run, worktree removed, IDEA exclusions cleaned. Integration checkout clean. Post-merge full gate was attempted twice; every runtime/architecture/static/style lane passed both times, but the unit lane deterministically exposed an unrelated existing picker-test selection-order flaw in SubagentLivePickerControllerTest::dismissFeedbackReplacesStaleExportFeedbackInPickerHeader. The test assumes list index 0 is agent_dismiss_done after inserting a second completed child, but under 4-worker ParaTest index 0 is child-keep, so export reports 'Session child-keep has no events to export.' Focused sequential class passes 17/56. No PR #321 production or subagent-progress code touches this behavior; no task code change was made post-merge.
