# Make test suite deterministic and keep every test under 15 seconds

## Goal
## Problem

`castor check` is increasingly flaky and timing-sensitive. The suite relies on broad waits, polling, fixed sleeps, long-lived subprocess/tmux scenarios, and some low-value duplicate tests. The gate must remain deterministic and reliable under its normal parallel load, and no individual test may take longer than 15 seconds.

## Baseline (2026-07-29)

A deterministic `/usr/bin/time -p castor check` passed in **114.50s** (`quality: 315.5s` is summed lane time). Artifacts: `var/reports/qa-20260729-202411-268989-fede1f65/`.

| Lane | Time |
| --- | ---: |
| unit/integration (4,468 tests) | 61.0s |
| controller replay (10 tests) | 83.1s |
| TUI replay (38 tests) | 111.7s |
| live LLM (15 tests) | 53.9s |
| deptrac/phpstan/cs | 0.9s / 2.0s / 3.0s |

One test exceeded the requested ceiling: `BuildDistributionScriptTest::testLockContentionFailsClosed` at **30.012s**. Tests close to the ceiling include `BashBackgroundCancelE2eTest` (14.806s), `RewindBranchLiveE2eTest` (13.945s), controller bash-cancel follow-up (12.414s), `ForkDeferredLiveE2eTest` (12.217s), TUI bash-cancel follow-up (11.613s), `TuiFileRewindE2eTest` (11.392s), `SubagentParallelLiveE2eTest` (11.136s), SafeGuard approval replay (10.959s), repeated auto-compaction replay (10.672s), and `SubagentRetrieveLiveE2eTest` (10.185s).

Latest 30 complete QA runs show recurring non-green lanes: unit 8/30 (`testRoundTripPreservesEventTypes`), live LLM 5/30 (`testRealLlamaCppInvocation`), controller replay 2/30 (auto-compaction), and TUI 2/30 (bash-background acceptance/bg-status).

Static audit found 128 lexical `sleep`/`usleep` occurrences across 56 test files and 35 across five Castor files. Many are polling intervals, but fixed synchronization delays and excessive caps exist. High-risk timeout budgets include repeated 20–60s waits in Rewind, SafeGuard approval, subagent retrieval/parallelism, deferred forks, queued steering, image paste, bash cancellation, installer/distribution, and shared tmux helpers. `FileRunSequenceAllocatorTest` has child workers that can wait indefinitely for a parent-created file.

Ponytail test audit identified about **33 tests / 782 lines** that may be removed or consolidated, led by duplicate tmux transcript/markdown tests already covered by the rich transcript E2E plus virtual rendering, then helper self-tests, method-existence/metadata checks, direct DTO/property checks, and intrinsic enum-value checks.

## Scope

1. Establish per-test timing reports from Castor/JUnit and use them to drive the work; do not merely lower timeout constants.
2. Root-cause and fix the recurring flaky tests named above. Trust real/live reproductions over replay when they disagree.
3. Ensure every test completes within 15 seconds under normal parallel `castor check` load. Split broad journeys, narrow assertions to the first exact event/visible proof, shorten deliberate delay fixtures, or move behavior to the lowest correct test-pyramid layer.
4. Replace fixed sleeps used for synchronization with bounded readiness/event/file/process/terminal polling. Keep real delays only when elapsed time is itself the contract, minimize them, and document why deterministic clocks/events cannot prove it.
5. Remove unbounded child waits and ensure subprocess/tmux/controller teardown cannot leak workers. Cleanup failures must remain diagnosable; never hide them in empty catches.
6. Apply the ponytail audit: delete or consolidate tests that protect no user-visible behavior, stable runtime/protocol contract, safety/security boundary, or observed regression. Preserve the smallest proof at the lowest correct layer.
7. Keep the suite isolated and ParaTest-safe using existing shared test infrastructure; do not add production APIs for tests or introduce new dependencies/abstractions.
8. Record before/after lane and slowest-test timings and the deleted/consolidated tests in the task handoff.

## Initial candidates

- Remove redundant `TuiTranscriptRenderE2eTest` and `TuiMarkdownRenderE2eTest` if `TuiRichTranscriptProductValidationE2eTest` plus virtual transcript tests fully preserve the contracts.
- Reassess `TestDirectoryIsolationTest`, weak tool `method_exists`/non-empty metadata assertions, class-specific payload constructor mirrors in `AgentBusMessageContractTest`, intrinsic enum-value tests, and `TuiRenderContextTest`.
- Replace the 2.5s image-paste synchronization sleep and 600ms subagent anti-flash sleep with exact readiness/visible proof.
- Rework `BuildDistributionScriptTest::testLockContentionFailsClosed` so lock contention is proven without a 30s child sleep.
- Narrow/split Rewind, SafeGuard approval, subagent, deferred-fork, auto-compaction, queued-steer, and bash-cancellation journeys whose aggregate timeout budgets exceed 15s.
- Fix unbounded file-gate child loops in `FileRunSequenceAllocatorTest`.
- Prefer the already-installed `MockClock` for wall-clock-sensitive assertions where the clock is injectable.

## Guardrails

- Read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before test work.
- All QA goes through Castor. No raw `vendor/bin/*` except explicit Castor-failure diagnosis.
- Do not mask flakes with retries or larger timeouts.
- Do not touch root-owned or `HATFIELD_SESSION_ID` worker processes; leaked workers are lifecycle bugs.
- Required runtime/TUI validation remains `castor check`; replay green alone is not proof when live behavior disagrees.
- Keep changes minimal and delete before adding infrastructure.

## Acceptance criteria
- A deterministic `castor check` passes at least three consecutive times with the normal gate configuration and no stress/guard/concurrency overrides; cache guard, artifact integrity, and post-run leak assertion remain enabled.
- Every individual test reported by Castor JUnit artifacts completes in 15.0 seconds or less on each acceptance run, including lock-contention, TUI, controller replay, and live-LLM tests.
- The four historically recurring failures (`testRoundTripPreservesEventTypes`, `testRealLlamaCppInvocation`, controller auto-compaction replay, and TUI bash-background/bg-status) are either root-cause fixed with the smallest valid regression proof or removed only with documented evidence that they duplicate stronger coverage.
- No fixed `sleep`/`usleep` remains as a synchronization mechanism in tests. Any remaining real delay directly proves elapsed-time behavior, is below the 15-second test ceiling, and has an explanatory comment.
- All child-process, controller, Messenger, and tmux waits are bounded; teardown leaves no current-run workers, and cleanup failures are surfaced diagnostically rather than swallowed.
- Low-value and duplicate tests identified by the audit are deleted or consolidated where their contract is already covered, with a handoff inventory of removals and retained replacement proof.
- Before/after evidence records full check wall time, each lane time, slowest individual tests, removed test count/lines, and the three acceptance-run artifact directories.
- Focused Castor validation passes during implementation, followed by the required deterministic `castor check` acceptance runs; no raw PHPUnit/tooling command is used.

## Workflow metadata
Status: ARCHIVE
Branch: task/refactor-flaky-slow-timing-dependent-tests
Worktree: /home/ineersa/projects/agent-core-worktrees/refactor-flaky-slow-timing-dependent-tests
Fork run: r7pt10imy7x9
PR URL: https://github.com/ineersa/agent-core/pull/339
PR Status: merged
Started: 2026-07-29T22:03:57.232Z
Completed: 2026-07-30T18:15:29.207Z

## Work log
- Created: 2026-07-29T20:55:49.106Z

## Task workflow update - 2026-07-29T22:02:15.887Z
- Summary: Deep exhaustive ponytail audit completed across all 489 test files / 3,551 methods (~139.6K lines). Additional evidence identifies roughly 3.1K–3.3K lines and ~100 test methods as removable/consolidatable, including ~12 expensive tmux/live-controller launches. High-confidence cuts: readonly DTO/constructor/getter/enum mechanics in AgentCore boundaries; provider-driven consolidation of repetitive malformed/result extraction cases; test-helper self-tests; runtime config/property tests; duplicate registry/invokable/metadata checks; duplicate generated-PHP/client URL/model-resolver smokes; nine redundant or mergeable tmux launches; one duplicate live controller happy path; exhaustive glyph assertions. Retain distinct process/provider/safety/concurrency/regression proofs. Apply method-by-method contract verification before deletion.
- Deep audit totals: AgentCore 60 files/408 methods -> ~61 methods and ~1,150 lines removable/consolidatable; CodingAgent+Platform 269 files/2,612 methods -> ~451 high-confidence lines or ~619 with conditional consolidations; TUI+runtime E2E 142 TUI test files plus controller tests -> ~1,550 lines and ~12 expensive launches possible.
- Largest runtime cuts: delete/merge redundant transcript, Markdown, edit-diff, `/compact`, `/usage`, startup config/header, manual compaction lifecycle, tree open/close, resume duplicate, and no-shell live controller tests while preserving canonical virtual/replay/tmux/live proofs.
- No files were edited and no tests were run during the deep static audit. All three scouts confirmed reading `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`; ponytail-audit rules were applied.

## Task workflow update - 2026-07-29T22:03:57.232Z
- Moved TODO → IN-PROGRESS.
- Created branch task/refactor-flaky-slow-timing-dependent-tests.
- Created worktree /home/ineersa/projects/agent-core-worktrees/refactor-flaky-slow-timing-dependent-tests.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/refactor-flaky-slow-timing-dependent-tests.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/refactor-flaky-slow-timing-dependent-tests.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/refactor-flaky-slow-timing-dependent-tests.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/refactor-flaky-slow-timing-dependent-tests.
- Summary: Starting implementation after user approval. Scope is test-only/Castor test infrastructure: remove or consolidate low-value duplicate coverage, replace timing synchronization with deterministic bounded proof, enforce <=15s individual tests, and preserve distinct behavioral/runtime/safety contracts.

## Task workflow update - 2026-07-29T22:04:48.123Z
- Recorded fork run: qur8ilvz3iek
- Implementation fork launched in task worktree with exhaustive audit candidates, <=15s/test requirement, deterministic wait/teardown constraints, historical flake targets, focused Castor validation, and explicit prohibition on full `castor check` during task-start.

## Task workflow update - 2026-07-29T23:44:32.531Z
- Summary: Implementation fork was manually stopped during its first test run. It left an uncommitted 50-file diff (96 additions, 2,948 deletions) with no completed validation or commit. Current diagnostics show no stale QA worker candidates and no user-owned PHPUnit/ParaTest/controller/Messenger workers; root-owned PID 3361 remains untouched.
- The displayed shell command appears to place `castor test` and `--filter=...` on separate lines, so the intended filter may not have been applied. Treat the entire partial diff as unvalidated until the hang is isolated with bounded, one-filter-at-a-time Castor invocations.
- Unexpected non-test change `docs/compaction.md` is present and must be reviewed/reverted non-destructively if unrelated; no direct cleanup performed by the orchestrator.

## Task workflow update - 2026-07-29T23:46:00.304Z
- Summary: User clarified a mandatory Castor invariant: every process launched by any Castor task must have an internally enforced hard timeout; no Castor-spawned process may run indefinitely. Maximum permitted timeout is 180 seconds. This must be implemented before any further QA execution; external agent/tool timeouts are only a secondary guard, not the solution.
- Pause test execution. Audit all Castor process-launch paths (`Process`, `proc_open`, `passthru`, shell helpers, parallel lane runners) and route them through bounded execution or add equivalent internal deadlines. Validate the timeout mechanism with a short deterministic hanging fixture, not a 180-second test.

## Task workflow update - 2026-07-29T23:46:38.828Z
- Recorded fork run: afld8gfljrqn
- Emergency narrow fork launched to enforce Castor-internal <=180s timeout on every subprocess path before any further QA. It must preserve the interrupted fork's dirty diff, run only a short timeout regression after implementation, and commit only Castor timeout files.

## Task workflow update - 2026-07-29T23:47:43.847Z
- Summary: Scope correction: do NOT add a universal Castor subprocess timeout. The <=180s hard ceiling applies to test runner processes launched by Castor (`test`, filtered/suite variants, controller replay/live, TUI, llm-real, and test lanes inside `check`). Build, packaging, Docker, CI orchestration, and other legitimately long-running Castor tasks must remain unaffected. Full `castor check`/CI wall time may exceed 180s; each spawned test lane/runner must be bounded.
- Emergency fork afld8gfljrqn failed without output before delivering a change. Supersede its universal-timeout instruction with test-only scope.

## Task workflow update - 2026-07-29T23:48:45.603Z
- Summary: Final timeout scope: (1) top-level `castor check` has a hard 180-second wall-clock ceiling, in addition to existing lane timeouts; (2) every test runner/process started by any Castor test task or check lane has a hard ceiling of at most 180 seconds; (3) non-test Castor tasks such as builds, packaging, Docker, and long CI orchestration are unaffected.
- User stopped failed fork afld8gfljrqn. Inspection shows no `.castor`/`castor.php` changes from it; only the prior interrupted test-refactor diff remains.

## Task workflow update - 2026-07-29T23:49:06.876Z
- Recorded fork run: 3lzbep2r3u35
- Corrected narrow fork launched: top-level check absolute 180s deadline plus <=180s for every Castor-started test runner, with non-test/build/CI tasks explicitly excluded.

## Task workflow update - 2026-07-29T23:57:50.803Z
- Recorded fork run: 3lzbep2r3u35
- Validation: `php -l .castor/{process,phpunit,e2e,tasks,llm-replay}.php` — passed.; `LLM_MODE=0 timeout --kill-after=5s 45s castor test:timeout-hardstop` — passed; short hanging child exited 124 with no descendants.
- Summary: Committed narrow Castor test-timeout implementation at `0e7e8ab19`: standalone Castor test runners are internally bounded to <=180s with session/process-tree reaping; check lanes are clamped to remaining wall; non-test/build tasks unchanged. Prior 50-file test-refactor diff remains uncommitted.
- Follow-up required before broad QA: make the top-level `castor check` 180s deadline begin at command entry so lock acquisition counts toward the user's absolute wall-clock requirement; current commit starts the wall inside the check body after lock acquisition.

## Task workflow update - 2026-07-30T00:31:33.765Z
- Recorded fork run: 0jgzfgr3h4ed
- Validation: `php -l .castor/{tasks,helpers,process}.php` — passed.; `LLM_MODE=0 timeout --kill-after=10s 45s castor test:timeout-hardstop` — A–K passed, including entry-deadline accounting.
- Summary: Committed follow-up `dff0096ea`: `castor check` 180s wall now begins at task entry and includes lock wait, setup/preflight, lanes, and finalizers. Preflight/lock budgets clamp to remaining wall. Non-test Castor tasks remain unaffected.
- Ready to resume the original dirty test refactor using the newly bounded Castor test runners.

## Task workflow update - 2026-07-30T00:32:23.205Z
- Recorded fork run: hmn2ps1917dz
- Continuation fork launched to audit every line of the interrupted 50-file diff, reject blind timeout reductions, isolate hang-risk tests first, then run bounded focused Castor suites and commit only validated test refactoring.

## Task workflow update - 2026-07-30T01:04:35.284Z
- Recorded fork run: hmn2ps1917dz
- Validation: `castor test --filter=FileRunSequenceAllocatorTest` — 6 tests passed.; `castor test --filter=BuildDistributionScriptTest::testLockContentionFailsClosed` — passed in ~0.64s test time.; `castor test` — 4,359 tests passed, 33.3s suite wall.; `castor test:controller-replay` — 10 tests passed, 78.2s.; `castor test:tui` — 29 tests passed, 99.3s (down from 38 tests / 111.7s baseline).; `castor test:llm-real` — 15 tests passed, 30.5s.; `castor cs-check` — passed.; `castor clean:cleanup:workers:list` — no stale QA workers.; `castor phpstan` — failed with 9 errors in the preceding Castor timeout commits; follow-up required.
- Summary: Committed validated test cleanup/refactor `62e3160ba` atop the two Castor timeout commits. Removed 2,751 net lines and 9 expensive tmux launches; replaced lock/file/sleep synchronization with bounded file/event proof; rejected blind timeout reductions. Worktree reported clean.
- Remaining implementation blockers: fix PHPStan errors in `.castor` timeout code; produce per-test JUnit timing evidence for <=15s; root-cause the four historical recurring flakes rather than relying on one green run.

## Task workflow update - 2026-07-30T01:23:46.475Z
- Summary: Historical artifact diagnosis refined remaining work: `testRoundTripPreservesEventTypes` has 99 retained JUnit passes and zero evidenced flakes; old serializer bug was already fixed by per-event denormalization. The lone Llama failure was a ParaTest worker crash during system-wide process contention, not HTTP/provider failure. Controller auto-compaction failure occurred before any LLM/compaction event because `runtime.ready` was emitted before consumers launched. TUI bash-background failures stalled on a redundant `Running` assertion before reaching the stronger background prompt/bg_status proof; an older pane-loss failure is already addressed by exact-run tmux ownership.
- Final implementation follow-up should: fix 9 PHPStan errors in Castor timeout commits; remove redundant TUI `Running`/fixed-delay synchronization; make controller readiness ordering reflect launched consumers without adding a new protocol surface; reduce check live-LLM worker contention minimally; generate JUnit timing artifacts and enforce <=15s per test.

## Task workflow update - 2026-07-30T01:24:14.152Z
- Recorded fork run: 52s77p3gc2su
- Final task-start fork launched for PHPStan cleanup, evidenced controller/TUI startup synchronization fixes, check-only live-LLM concurrency reduction, and JUnit per-test <=15s proof.

## Task workflow update - 2026-07-30T01:38:51.219Z
- Recorded fork run: 52s77p3gc2su
- Validation: `castor phpstan` — passed.; `castor cs-check` — passed after `castor cs-fix`.; Filtered bash-background TUI — 2 tests passed.; `castor test:tui` — 29 tests passed twice.; `castor test:controller-replay` — 10 tests passed on three final runs; one intermediate SafeGuard failure remains to monitor during gate.; `castor test:llm-real` — 15 tests passed twice, standalone 4 workers.; JUnit timing: unit max 5.010s; controller-replay max 10.175s; TUI max 13.911s; llm-real max 14.599s; zero tests >15s.; `castor clean:cleanup:workers:list` — no stale workers.
- Summary: Committed final implementation follow-up `8b45e5aae`: PHPStan fixed, controller emits ready after consumer launch, TUI bash-background tests wait directly for specific overlay/proof, check llm-real default reduced 2→1 while standalone remains 4, and unsupported 10s live HTTP timeout removed.
- Documentation sync fork qmcihcxoq2kd launched because testing skill/tests docs still claimed check llm-real uses 2/4 workers and did not state the new Castor test/check hard ceilings.

## Task workflow update - 2026-07-30T01:41:27.578Z
- Recorded fork run: qmcihcxoq2kd
- Validation: Focused implementation validation is green: unit 4,359; controller replay 10; TUI 29; live LLM 15; PHPStan; CS check; no stale workers.; JUnit per-test maxima: unit 5.010s, controller replay 10.175s, TUI 13.911s, live LLM 14.599s; zero >15s.; Full `castor check` intentionally not run during task-start; required during task-to-pr.
- Summary: Implementation phase complete on clean branch tip `287785089`. Four commits enforce bounded Castor tests/check, remove low-value coverage, replace timing synchronization with deterministic proof, fix controller/TUI readiness races, reduce check-only live-LLM contention, and synchronize canonical QA docs. Task remains IN-PROGRESS pending task-to-pr review and deterministic gate.
- Docs sync commit `287785089` updated AGENTS.md, testing skill, tests/AGENTS.md, and docs/qa-metrics.md for <=180s test runners, absolute 180s check wall, check llm-real=1 vs standalone=4.
- Implementation commits: `0e7e8ab19`, `dff0096ea`, `62e3160ba`, `8b45e5aae`, `287785089`.

## Task workflow update - 2026-07-30T02:01:06.431Z
- Summary: Reviewer verdict: APPROVED. Specification-fidelity inventory found no unmapped external surface or unnecessary complexity; deleted tests have verified replacement proof. Before final gate, applying a tiny mandatory comment/timeout cleanup identified by review: update stale 4-worker rationale comments, clarify check llm-real default call site, and independently bound direct Llama smoke HTTP below the 15s per-test ceiling.
- Reviewer confirmed reading root AGENTS, testing skill, and tests/AGENTS. Validation gap remains the task-to-pr gate requirement: three consecutive deterministic `castor check` runs.

## Task workflow update - 2026-07-30T02:02:36.990Z
- Recorded fork run: vuctbvnh72aw
- Validation: Filtered `LlamaCppSmokeTest::testRealLlamaCppInvocation` — passed, 1 test/9 assertions, 2.2s.; `castor phpstan` — passed.; `castor cs-check` — passed.; Fork confirmed testing skill and tests/AGENTS were read first.
- Summary: Review cleanup committed as `f9b6fbd2c`: updated stale check/standalone worker rationale comments, aligned llm-real helper fallback/default call site to 1, and restored a direct 10s HTTP timeout so the smoke cannot stall past the 15s individual-test ceiling.

## Task workflow update - 2026-07-30T02:05:36.416Z
- Summary: Re-review of `287785089..f9b6fbd2c`: APPROVED. Reviewer confirmed the prior full-branch approval remains valid; no unmapped behavior or complexity, worker docs/fallback accurate, and direct 10s smoke timeout correctly protects the <=15s test ceiling.
- Proceeding to focused Castor validation, then two explicit deterministic check acceptance runs; CODE-REVIEW transition will run the third deterministic check before push/PR.

## Task workflow update - 2026-07-30T02:17:50.098Z
- Validation: `castor test`, deptrac, phpstan, cs-check, controller replay, and TUI all passed before live-lane failure.; `castor test:llm-real` failed: RewindBranchLiveE2eTest Pineapple turn missing run.completed; JUnit duration 18.315s (>15s).; `castor clean:cleanup:workers:list` — no stale QA worker candidates.
- Summary: Focused validation found a real acceptance failure: `RewindBranchLiveE2eTest` failed at 18.315s while waiting for its second of five dependent live generations. No leak remained. Artifact diagnosis found no controller/provider/index defect; the test is a broad probabilistic context journey duplicating deterministic rewind state and TUI proofs. It must be removed rather than retried or given more time.
- Scout diagnosis: five sequential live generations/two rewinds cannot meet <=15s under normal 4-worker standalone contention. Deterministic `SessionRunStateReplayServiceTest` proves abandoned context exclusion; replay TUI tree test proves visible rewind; in-process proof covers run.leaf_changed; controller smoke covers live provider lifecycle. No distinct stable contract justifies the probabilistic five-generation live journey.

## Task workflow update - 2026-07-30T03:06:32.447Z
- Recorded fork run: 26qazb969j4n
- Validation: Deterministic abandoned-context replay proof — passed.; In-process run.leaf_changed proof — passed.; Filtered TUI rewind proof — passed in 9.455s.; Filtered live ControllerSmokeTest — passed in 4.419s.; Full `castor test:llm-real` — 14 tests passed; JUnit max 12.779s; zero >15s.
- Summary: Deleted over-budget probabilistic `RewindBranchLiveE2eTest` in commit `e5f92d1b9` after verifying stronger deterministic/in-process/TUI/live-lifecycle replacement proofs. Remaining full live lane: 14 tests, max 12.779s, zero >15s.

## Task workflow update - 2026-07-30T03:18:41.511Z
- Validation: Check run `qa-20260730-030909-1433876-7c0f7db3` — passed; 4,412 JUnit cases, max 13.898s, zero >15s, cache/artifacts/leaks green.; Check run `qa-20260730-031113-1445745-87394819` — failed controller replay; SafeGuard block test 47.214s with two warnings; no leaks.
- Summary: Acceptance run 1 passed, but run 2 exposed the remaining SafeGuard replay false positive: `testWriteOutsideCwdBlockHasNoFilesystemSideEffect` took 47.214s, never observed `human_input.requested`, emitted undefined-key warnings, then passed solely because no file existed. This invalidates the consecutive-run sequence.
- Coverage audit found the flawed first-call deny method has no distinct contract: direct hook tests prove write requires approval and Deny→Block/safeguard_denied; subscriber/suspension tests prove blocked resume cannot fall through; neighboring sequential controller replay proves a denied second edit has no side effect after prior Allow. Delete only the false-positive method; do not retry or lengthen waits.

## Task workflow update - 2026-07-30T03:28:51.894Z
- Recorded fork run: qfr0l4uvudk6
- Validation: Full controller replay run 1 — 9 tests/132 assertions passed; max 12.028s, zero >15s.; Full controller replay run 2 — 9 tests/132 assertions passed; max 11.900s, zero >15s.; `castor cs-check` — passed.
- Summary: Deleted only the false-positive SafeGuard block-only replay method in `022016d59`; Allow and stronger sequential Allow→Deny proof remain. Controller replay is now 9 tests.

## Task workflow update - 2026-07-30T03:35:11.315Z
- Validation: `qa-20260730-033058-1481084-9d099f18` — gate green, cache/artifacts/leaks green, but JUnit max 20.882s SubagentRetrieveLiveE2eTest (1 >15).; `qa-20260730-033306-1491944-47142672` — gate green, cache/artifacts/leaks green; 4,411 JUnit tests, max 14.257s, zero >15s.
- Summary: After SafeGuard cleanup, two deterministic gates were green at process level, but acceptance run 1 still violated the per-test ceiling: `SubagentRetrieveLiveE2eTest` took 20.882s. Run 2 had zero >15s (max 14.257s). The acceptance sequence is not valid; do not proceed to PR until the remaining broad live retrieve journey is root-caused.

## Task workflow update - 2026-07-30T03:41:43.724Z
- Summary: Root cause of 20.882s `SubagentRetrieveLiveE2eTest`: one test serializes three live model generations (parent tool selection, child completion, parent retrieve selection) plus controller/Messenger/artifact work. This structurally varies past 15s under normal parallel gate load; the 60s collectors only hide it. The model choosing `agent_retrieve` is not a stable code contract.
- All stable component contracts already have deterministic proof: artifact lookup by ID/child run and parent scoping, relative-path safety, registry/handoff persistence, retrieve tool schema, child state/events, successful deferred-subagent presentation/projection, and generic controller tool transport. Existing live subagent/deferred tests retain live provider/process coverage. Use deletion, not a new replay journey: a three-fixture controller test would duplicate those lower-layer proofs and preserve only model/tool-selection choreography.

## Task workflow update - 2026-07-30T03:51:57.684Z
- Recorded fork run: v5fwy5o32npq
- Validation: Focused deterministic retrieval/artifact/lifecycle/projection coverage — 89 tests passed.; Full live lane run 1 — 13 tests passed; max 12.101s, zero >15s.; Full live lane run 2 — 13 tests passed; max 11.460s, zero >15s.; `castor cs-check` — passed.
- Summary: Deleted over-budget three-generation `SubagentRetrieveLiveE2eTest` in `ede9f29d6`; stable retrieval/registry/path-safety/lifecycle/projection contracts remain deterministic, while other live subagent/fork/controller tests retain provider/process coverage. Live lane now 13 tests.

## Task workflow update - 2026-07-30T03:56:38.097Z
- Validation: `qa-20260730-035435-1518860-e10ec35e` — all lanes/cache/artifacts/leaks green; 4,410 JUnit tests; max 15.351s; one >15s (BashBackgroundCancelE2eTest).
- Summary: First acceptance gate on `ede9f29d6` was process-green but still not acceptance-green: `BashBackgroundCancelE2eTest::testBashBackgroundPromptPathCanBeCancelledWithoutLeaks` measured 15.351s. Sequence resets; do not proceed to PR.

## Task workflow update - 2026-07-30T04:02:40.173Z
- Summary: Root cause of the 15.351s TUI cancel test is production lifecycle code, not the fixture: `BackgroundProcessManager::stopProcessEntity()` always sleeps the full configured 5s grace after SIGTERM even when the child exits immediately. Event timing shows ~6s from cancel command to cancelled result. Fix the shared stop path to return as soon as the process is dead while retaining deadline and SIGKILL fallback.
- Keep the TUI test: it uniquely proves visible background prompt → Escape/Deny → cancellation with no leaked worker. Do not shorten the `sleep 4` fixture; it creates the prompt window. Add/update one smallest manager proof for cooperative early exit and retain TERM/KILL behavior tests.

## Task workflow update - 2026-07-30T16:32:42.085Z
- Recorded fork run: 88kknfuknphv
- Validation: `castor clean:cleanup:workers:list` — no stale QA worker candidates.
- Summary: BackgroundProcessManager fix fork orphaned before validation/commit, leaving a two-file dirty diff. The diff replaces unconditional grace sleep with bounded 50ms exit/status polling and updates the existing TERM test. No stale workers. A continuation fork will audit, validate, and commit rather than discard the work.

## Task workflow update - 2026-07-30T16:37:17.853Z
- Recorded fork run: 63omy4hkwuz6
- Validation: BackgroundProcessManagerTest — passed.; BashToolTest — passed.; Filtered BashBackgroundCancel TUI — passed.; Full TUI — 29 tests passed; max 10.429s, zero >15s.; PHPStan and CS check — passed.; No stale workers.
- Summary: Committed root-cause cooperative-stop fix `4f6ea7499`: TERM now polls boundedly for status/PID exit and returns immediately; SIGKILL and forensic marker fallback remain. BashBackgroundCancel fell 15.351s→8.216s; full TUI max 10.429s.

## Task workflow update - 2026-07-30T16:38:41.656Z
- Recorded fork run: 2bp7km1660is
- Validation: BackgroundProcessManagerTest — 21 tests passed.; PHPStan and CS check — passed.
- Summary: Committed test-only readiness cleanup `0c0e49db7`: cooperative TERM test now waits on a bounded ready sentinel after trap installation instead of fixed 100ms synchronization. Production stop fix remains `4f6ea7499`.

## Task workflow update - 2026-07-30T16:43:28.453Z
- Summary: Final reviewer: APPROVED, including cooperative-stop correctness and all prior deletions/timeout infrastructure. Reviewer also surfaced a strict acceptance gap outside the latest delta: remaining fixed usleep synchronization in BackgroundProcessManagerTest and potentially elsewhere. Before restarting gates, performing a final lexical/classification audit so CODE-REVIEW does not claim the 'no fixed sleep synchronization' criterion without evidence.
- Known remaining examples in BackgroundProcessManagerTest at prior lines 86,108,188,213,250,277,306,321,352,371 need classification/replacement unless they directly prove elapsed-time behavior.

## Task workflow update - 2026-07-30T16:52:16.075Z
- Summary: Strict final audit found acceptance work still required despite green focused suites: 18 non-TUI fixed synchronization waits, 53 TUI fixed synchronization waits, and Castor test setup/smoke subprocess paths without internal bounds. Bounded polling intervals and behavior/elapsed delays were separately classified and are not blanket deletions. PR gate remains paused.
- Plan: three minimal sequential cleanup commits on the same worktree: non-TUI synchronization, TUI synchronization, then Castor test-process bounds. Do not lower arbitrary timeout caps; final three deterministic checks provide actual <=15s evidence.

## Task workflow update - 2026-07-30T16:56:02.335Z
- Recorded fork run: xezh6u23lxzo
- Validation: Combined touched-class filter — 51 tests passed.; BashToolTest — 26 tests passed.; PHPStan/CS check — passed.; No stale workers.
- Summary: Committed non-TUI synchronization cleanup `7e60e6d73`: 18 fixed waits removed or replaced with bounded finished/log/HTTP/process readiness; six test files only, no production/TUI changes.

## Task workflow update - 2026-07-30T17:11:07.344Z
- Recorded fork run: r7e126m8nj9o
- Validation: Full TUI run 1 — 29 tests passed; max 8.960s, zero >15s.; Full TUI run 2 — 29 tests passed; max 10.923s, zero >15s.; PHPStan/CS check — passed.; No stale workers.
- Summary: Committed TUI synchronization cleanup `e30913349`: 53 fixed waits removed/replaced across 23 test/harness files using existing readiness/event/file proofs; no production changes.

## Task workflow update - 2026-07-30T17:16:47.272Z
- Recorded fork run: cek8uis54wff
- Validation: test:timeout-hardstop A–K — passed.; castor test — 4359 tests passed in 25.1s.; Focused BashInstaller/background/TUI tests — passed.; PHPStan/CS check — passed.; No stale workers.
- Summary: Committed Castor test-process bounds `c272fc9d9`: standalone test DB migrations and ParaTest worker migration/lock capped at 60s; all test runners remain capped at 180s; hardstop smoke child calls explicitly bounded; non-test tasks unaffected.

## Task workflow update - 2026-07-30T17:27:54.353Z
- Summary: Final full-branch reviewer verdict: APPROVED WITH SUGGESTIONS; no correctness/specification/complexity blockers across 82 files. One real origin/main semantic merge conflict identified in TuiSubagentProgressE2eTest: retain readiness-proof cleanup while adopting main's renamed `3 LLM steps` assertion. Final 3× checks still pending on final tip.
- Reviewer verified timeout walls/process reaping, production root-cause fixes, test deletions/replacement contracts, fixed-wait cleanup, docs, and specification fidelity. Optional edge: shared.php timeout kill-after can theoretically add 15s for SIGTERM-ignoring check preflight, but current migrate/curl commands honor TERM and lane reaping is stricter.

## Task workflow update - 2026-07-30T17:29:59.633Z
- Recorded fork run: tt9v82hlhnpy
- Validation: TuiSubagentProgressE2eTest — 1 test/14 assertions passed in 4.9s.; PHPStan/CS check — passed.; No stale workers.
- Summary: Merged current origin/main into task branch as `7e682f8f6`; verified semantic auto-merge keeps event/readiness waits and adopts current `3 LLM steps` assertion. Worktree clean.

## Task workflow update - 2026-07-30T17:35:26.121Z
- Recorded fork run: 2zojmy4q18us
- Validation: Hardstop A–K passed.; Unit 4363 tests/25.2s, max 5.015s.; Controller replay 9 tests/66.7s, max 10.688s.; TUI 29 tests/85.1s, max 11.028s.; Live LLM 13 tests/21.9s, max 11.127s.; Deptrac/PHPStan/CS check passed; no stale workers.; Combined JUnit: 4414 tests, zero >15s.
- Summary: Final focused validation on merged tip `7e682f8f6` passed all required lanes and quality tools; JUnit proves every individual test below 15s.

## Task workflow update - 2026-07-30T17:41:49.842Z
- Recorded fork run: p89aufqfpmvm
- Validation: Check 1 qa-20260730-173554-72-0770d02d — wall 93.198s; 4414 tests; max 12.983s; zero >15s; cache 202→202; no leaks; 7/7 logs.; Check 2 qa-20260730-173740-7591-524a40a7 — wall 90.216s; 4414 tests; max 11.788s; zero >15s; cache 202→202; no leaks; 7/7 logs.; Check 3 qa-20260730-173918-15006-1968feb8 — wall 98.542s; 4414 tests; max 13.687s; zero >15s; cache 202→202; no leaks; 7/7 logs.; Post-gate workers diagnostic clean; git clean.
- Summary: Final acceptance on merged tip `7e682f8f6`: three consecutive deterministic castor checks passed. Reviewer APPROVED full branch and post-merge delta; all task acceptance criteria are met.

## Task workflow update - 2026-07-30T17:43:39.283Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 240s)...
- castor check passed (95.0s).
- Pushed task/refactor-flaky-slow-timing-dependent-tests to origin.
- branch 'task/refactor-flaky-slow-timing-dependent-tests' set up to track 'origin/task/refactor-flaky-slow-timing-dependent-tests'.
- Created PR: https://github.com/ineersa/agent-core/pull/339
- Validation: Focused: hardstop A–K, unit, deptrac, phpstan, cs-check, controller replay, TUI, live LLM all passed; 4414 tests max 11.127s.; Three consecutive deterministic castor checks passed: 93.198s/max12.983s; 90.216s/max11.788s; 98.542s/max13.687s. Zero >15s, zero proxy-cache growth, zero leaks, 7/7 logs each.
- Summary: Reviewer approved complete 82-file branch and post-origin/main merge. Refactor removes low-value/duplicated tests, replaces fixed synchronization sleeps with event/condition proofs, fixes runtime readiness and cooperative process-stop root causes, and hard-bounds Castor check/test subprocesses. Final tip `7e682f8f6` passed three consecutive deterministic acceptance checks with every test below 15s and wall below 180s.

## Task workflow update - 2026-07-30T18:15:29.208Z
- Moved CODE-REVIEW → DONE.
- Merged task/refactor-flaky-slow-timing-dependent-tests into integration checkout.
- Merge made by the 'ort' strategy.
 .agents/skills/testing/SKILL.md                    |  18 +-
 .castor/distribution.php                           |   4 +-
 .castor/e2e.php                                    | 131 ++++----
 .castor/helpers.php                                | 104 ++++--
 .castor/llm-replay.php                             |  13 +-
 .castor/phpunit.php                                |  56 +++-
 .castor/process.php                                | 178 +++++++++-
 .castor/tasks.php                                  | 150 +++++++--
 AGENTS.md                                          |   6 +-
 docs/compaction.md                                 |  15 +-
 docs/qa-metrics.md                                 |  13 +-
 .../Runtime/Controller/HeadlessController.php      |  19 +-
 src/CodingAgent/Tool/BackgroundProcessManager.php  |  26 +-
 tests/AGENTS.md                                    |   6 +-
 .../Domain/Command/CommandBoundaryTest.php         | 134 --------
 .../Domain/Message/AgentBusMessageContractTest.php | 254 --------------
 .../Domain/Model/ModelInvocationContractTest.php   | 137 --------
 tests/AgentCore/Domain/Run/RunStateTest.php        |  48 ---
 tests/AgentCore/Domain/Tool/ToolBoundaryTest.php   | 130 --------
 .../Infrastructure/SymfonyAi/LlamaCppSmokeTest.php |   3 +-
 .../Agent/Artifact/AgentArtifactKindEnumTest.php   |   6 -
 .../Build/ApplicationBuildIdentityTest.php         |  16 -
 .../ModelSelectionActiveModelResolverTest.php      |  27 --
 .../CodingAgent/Distribution/BashInstallerTest.php |  21 +-
 .../Distribution/BuildDistributionScriptTest.php   |  48 ++-
 .../Controller/E2E/ControllerE2eTestCase.php       |  14 +-
 .../Controller/E2E/RewindBranchLiveE2eTest.php     | 364 ---------------------
 .../E2E/SafeGuardApprovalControllerReplayTest.php  |  41 ---
 .../Controller/E2E/SubagentRetrieveLiveE2eTest.php | 299 -----------------
 .../ErrorCapture/RuntimeErrorCaptureConfigTest.php |  42 ---
 .../Runtime/Projection/TranscriptBlockTest.php     | 107 ------
 tests/CodingAgent/Runtime/RuntimeEventTypeTest.php |  44 ---
 .../Session/FileRunSequenceAllocatorTest.php       |  37 ++-
 .../Support/TestDirectoryIsolationTest.php         | 107 ------
 tests/CodingAgent/Tool/AskHumanToolTest.php        |  34 --
 .../Tool/BackgroundProcessManagerTest.php          |  98 ++++--
 tests/CodingAgent/Tool/BashToolTest.php            |  19 +-
 tests/CodingAgent/Tool/BgStatusToolTest.php        |  39 ++-
 tests/CodingAgent/Tool/ReadFileToolTest.php        |  49 ---
 .../Tool/ToolQuestion/ToolQuestionStoreTest.php    |   2 +-
 tests/CodingAgent/Tool/ViewImageToolTest.php       |  47 ---
 tests/CodingAgent/Tool/WriteFileToolTest.php       |  49 ---
 .../Generic/SanitizedGenericModelClientTest.php    |  11 +-
 .../OpenAICodex/CodexWebSocketUrlResolverTest.php  |  21 --
 tests/Tui/E2E/BashBackgroundAcceptE2eTest.php      |  10 +-
 tests/Tui/E2E/BashBackgroundCancelE2eTest.php      |   5 +-
 tests/Tui/E2E/BashBackgroundE2eTestSupport.php     |   5 +-
 tests/Tui/E2E/BashCancelFollowUpE2eTest.php        |  13 +-
 tests/Tui/E2E/CancelStickinessE2eTest.php          |  10 +-
 tests/Tui/E2E/TmuxHarness.php                      |  11 +-
 .../Tui/E2E/TuiAskHumanOverlayMarkdownE2eTest.php  |   1 -
 tests/Tui/E2E/TuiAutoCompactionCancelE2eTest.php   |  31 +-
 tests/Tui/E2E/TuiAutoCompactionE2eTest.php         | 119 +------
 tests/Tui/E2E/TuiCompactCommandE2eTest.php         | 190 -----------
 tests/Tui/E2E/TuiCompactHeaderE2eTest.php          | 150 ---------
 tests/Tui/E2E/TuiFileRewindE2eTest.php             |   3 -
 tests/Tui/E2E/TuiImagePasteE2eTest.php             |  28 +-
 tests/Tui/E2E/TuiJourneyE2eTest.php                |   1 -
 tests/Tui/E2E/TuiMarkdownRenderE2eTest.php         | 243 --------------
 tests/Tui/E2E/TuiOutputCapNoticeE2eTest.php        |   1 -
 tests/Tui/E2E/TuiProviderErrorE2eTest.php          |   1 -
 tests/Tui/E2E/TuiQueuedSteerE2eTest.php            |   1 -
 tests/Tui/E2E/TuiRepairCommandE2eTest.php          |   1 -
 tests/Tui/E2E/TuiResumeModelRestoreE2eTest.php     |   9 +-
 tests/Tui/E2E/TuiResumeSessionSwitchE2eTest.php    |  13 +-
 .../TuiRichTranscriptProductValidationE2eTest.php  |  12 +-
 .../Tui/E2E/TuiStartupTranscriptConfigE2eTest.php  | 210 ------------
 .../Tui/E2E/TuiStatusRowReasoningNoticeE2eTest.php |   1 -
 .../TuiSubagentChildHitlCancellationE2eTest.php    |   7 +-
 .../Tui/E2E/TuiSubagentLiveChildExportE2eTest.php  |  39 ++-
 tests/Tui/E2E/TuiSubagentLiveViewE2eTest.php       |  15 +-
 tests/Tui/E2E/TuiSubagentProgressE2eTest.php       |   7 +-
 tests/Tui/E2E/TuiToolOutputE2eTest.php             |  63 ----
 tests/Tui/E2E/TuiTranscriptRenderE2eTest.php       | 197 -----------
 tests/Tui/E2E/TuiTreeCommandE2eTest.php            | 119 -------
 tests/Tui/E2E/TuiUsageCommandE2eTest.php           | 183 -----------
 .../Tui/E2E/fixtures/tui-tool-call-bash-sleep.json |   8 +-
 .../E2E/fixtures/tui-tool-call-bash-sleep8.json    |   8 +-
 .../TuiTranscriptBlocksVirtualRenderTest.php       |  58 ----
 tests/Tui/Theme/ThemeColorEnumTest.php             |   4 -
 tests/Tui/Widget/TuiRenderContextTest.php          |  36 --
 tests/paratest-bootstrap.php                       |  31 +-
 82 files changed, 947 insertions(+), 3884 deletions(-)
 delete mode 100644 tests/AgentCore/Domain/Command/CommandBoundaryTest.php
 delete mode 100644 tests/AgentCore/Domain/Message/AgentBusMessageContractTest.php
 delete mode 100644 tests/AgentCore/Domain/Tool/ToolBoundaryTest.php
 delete mode 100644 tests/CodingAgent/Compaction/ModelSelectionActiveModelResolverTest.php
 delete mode 100644 tests/CodingAgent/Runtime/Controller/E2E/RewindBranchLiveE2eTest.php
 delete mode 100644 tests/CodingAgent/Runtime/Controller/E2E/SubagentRetrieveLiveE2eTest.php
 delete mode 100644 tests/CodingAgent/Runtime/ErrorCapture/RuntimeErrorCaptureConfigTest.php
 delete mode 100644 tests/CodingAgent/Support/TestDirectoryIsolationTest.php
 delete mode 100644 tests/Platform/Bridge/OpenAICodex/CodexWebSocketUrlResolverTest.php
 delete mode 100644 tests/Tui/E2E/TuiCompactCommandE2eTest.php
 delete mode 100644 tests/Tui/E2E/TuiCompactHeaderE2eTest.php
 delete mode 100644 tests/Tui/E2E/TuiMarkdownRenderE2eTest.php
 delete mode 100644 tests/Tui/E2E/TuiStartupTranscriptConfigE2eTest.php
 delete mode 100644 tests/Tui/E2E/TuiTranscriptRenderE2eTest.php
 delete mode 100644 tests/Tui/E2E/TuiUsageCommandE2eTest.php
 delete mode 100644 tests/Tui/Widget/TuiRenderContextTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/refactor-flaky-slow-timing-dependent-tests.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/refactor-flaky-slow-timing-dependent-tests.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #339 state MERGED at 2026-07-30T18:14:57Z.; Pre-merge integration checkout clean.
- Summary: PR #339 merged on GitHub as e211e9723295d7f6f8c690942dd48c70c7014222. Task acceptance completed: deterministic suite, every measured test under 15s, Castor test/check subprocesses bounded, and three consecutive acceptance checks green.

## Task workflow update - 2026-07-30T18:18:48.131Z
- Recorded fork run: r7pt10imy7x9
- Validation: LLM_MODE=true castor check passed in 100.409s wall.; All 7 lanes green; 4414 tests; max individual 12.649s; zero >15s.; Proxy cache 202→202; 7/7 logs; no leaked or stale workers.; Integration git status clean.
- Summary: Post-merge DONE validation passed on integration checkout HEAD `74f9c79fb`; PR #339 and task tip are ancestors, checkout remains clean.

## Task workflow update - 2026-08-06T20:59:22.885Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
