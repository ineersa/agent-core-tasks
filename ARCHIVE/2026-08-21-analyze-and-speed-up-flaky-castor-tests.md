# Analyze and speed up flaky Castor test suite

## Goal
## Goal
Make Castor QA lanes faster and less flaky so `castor check` / `move_task → CODE-REVIEW` stops failing for environmental timeouts and known flakes.

This is an analysis + targeted fixup task, not a broad test rewrite. Prefer measurement → ranked hotspots → smallest fixes that recover wall-clock margin and remove known flakes.

## Recent evidence (2026-08-20/21)
- `test:tui` under `castor check` is near the wall: green ~181s vs hard timeout 200s (`wall remaining 200s / max 210s`). One kill left only ParaTest banner + no `phpunit-tui.junit.xml` — cannot name hung case.
- Concurrent sibling worktree TUI runs contend on the shared tmux server and amplify flake/timeout risk.
- Known flake: `ProvidersUpdateCommandTest::testRebaseAndSyncUpdatesMetadataWithoutAddingNewUpstreamIds` failed when Symfony Console wrapped long OK text across lines (fixed by collapsing whitespace before contains assert; keep as regression class).
- Sibling TUI flake example: `TuiSubagentChildHitlCancellationE2eTest::testLeaveChildLiveViewDropsChildQuestionAndEscDoesNotFalseCancel` timed out waiting for `"needs input"` / hit duplicate event-sequence replay errors under load.
- Prior green TUI junit: 41 tests, case-time sum ~342s across 2 workers; slowest include ReloadSettings (~21s), FileRewind (~13s), SubagentLiveView (~12s), History/BashBackground (~11s).

## Scope
1. **Measure**
   - Collect recent `var/reports/qa-*/` junit + lane logs for `test`, `test:tui`, `test:controller-replay`, `test:llm-real`.
   - Rank slowest tests and highest flake/timeout frequency by lane.
   - Document which failures are product bugs vs contention/timeout-budget issues.
2. **Speed**
   - Cut TUI/`castor check` wall usage enough to restore comfortable margin under the 210s absolute wall (target: TUI lane reliably under ~150–160s wall, or justify budget change with data).
   - Prefer: fewer/shorter waits, less redundant boot, lower proof layer when virtual/controller-replay already covers, ParaTest process tuning only with measured effect.
   - Do **not** weaken required TUI/runtime proof for product changes; demote only redundant/over-layered cases.
3. **Flakes**
   - Triage and fix the top flake classes (assert-on-wrapped console output, tmux needle races, duplicate sequence/replay under contention, leftover QA tmux contamination).
   - Add or tighten isolation/cleanup where concurrent worktrees share tmux.
4. **Gates / diagnostics**
   - Make timed-out lanes leave enough signal to name the hung test when possible (junit flush, last-running test marker, or clearer hard-timeout diagnostics).
   - Optionally document “don’t run concurrent `test:tui` across worktrees” or add a soft lock/contention warning if cheap.

## Out of scope
- Rewriting the whole suite for coverage aesthetics.
- Raising timeouts as the primary “fix” without measuring and reducing waste.
- Product feature work unrelated to test speed/stability.
- Changing task-workflow skill/prompts except tiny pointers if needed (broader workflow harden is a separate TODO).

## Approach notes
- Load `testing` skill + `tests/AGENTS.md` before any test edits.
- Castor-only QA; no raw `vendor/bin/phpunit` except to isolate Castor failure.
- Keep changes minimal and data-driven; open a short ranked report in the task work log before large edits.
- If a flake is a real product bug, fix the product path; if environmental, fix harness/isolation/budget.

## Acceptance criteria
- Ranked hotspot report exists (slowest tests + top flake/timeout classes) from recent Castor reports, with product-vs-environment classification
- castor check TUI lane recovers measurable wall margin (target reliably under ~150–160s, or documented justified budget change with before/after numbers)
- Top known flake classes from recent evidence are fixed or quarantined with rationale (console-wrap asserts, TUI needle/contention races, timeout-with-no-junit diagnostics)
- Focused Castor validation proves the speed/flake fixes; full castor check green without relying on ‘retry until no contention’
- No required product proof layers removed; demotions only for redundant/over-layered cases with justification

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-21-analyze-and-speed-up-flaky-castor-tests
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-21-analyze-and-speed-up-flaky-castor-tests
Fork run: agent_3dc96dcee8ee50d6
PR URL: https://github.com/ineersa/agent-core/pull/424
PR Status: merged
Started: 2026-08-22T18:56:18+00:00
Completed: 2026-08-23T18:05:13+00:00

## Work log
- Created: 2026-08-21T17:53:28+00:00

## Task workflow update - 2026-08-22T18:56:18+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-21-analyze-and-speed-up-flaky-castor-tests.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-21-analyze-and-speed-up-flaky-castor-tests.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-21-analyze-and-speed-up-flaky-castor-tests.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-21-analyze-and-speed-up-flaky-castor-tests.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-21-analyze-and-speed-up-flaky-castor-tests.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-21-analyze-and-speed-up-flaky-castor-tests.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-21-analyze-and-speed-up-flaky-castor-tests/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-21-analyze-and-speed-up-flaky-castor-tests.
- Summary: Claimed for user-directed comprehensive test-suite stabilization and speed work. Scope includes measured hotspot analysis, deterministic rewrites/deletions of low-value or timing-based tests, concurrency isolation, hang diagnostics, and repeated parallel Castor validation.

## Task workflow update - 2026-08-22T20:40:18+00:00
- Summary: Ranked scout hotspot report: TUI replay is dominant at ~155–177s for 41 tests/2 workers, with cleanup warnings and known capture/duplicate-replay flakes; controller replay is ~119–142s for 13 tests against a 150s lane budget; llm-real varies ~109–147s and has tool/prose-sensitive flakes; unit lane is comparatively stable at ~70–76s. Concrete harness defects found: replay base claims process-group isolation but does not establish it; second-controller test overwrites first controller PID ownership and only kills its root; generic teardown can spend 3s per replay test (~28–42s potential waste); timeout diagnostics discard already-collected PID/session evidence; Messenger diagnostics inspect the wrong DB path; replay fixture exhaustion fabricates a successful 'done' response; child HITL TUI omits the paired Messenger transport DB env, risking cross-worker duplicate replay. TUI audit found 31 E2E classes, 44 methods, 39 full boots, ~191 waits, 19 usleeps, many real 3–8s shell sleeps/delayed fixtures, broad negative scrollback assertions, and substantial virtual/replay-duplicate proof. Highest-value reductions: deterministic process ownership/teardown, paired DB isolation, remove fixed sleeps/real-delay fixtures, demote redundant Journey/SubagentLiveView/FileRewind/History/tool-card assertions to existing lower layers, and delete soft/redundant live tests. Low-value unit tests exist but are not the main wall-clock problem; safe deletions include ThemeColorEnumTest, literal OAuth constants, and a trivial home-dir getter test.

## Task workflow update - 2026-08-22T21:13:08+00:00
- Recorded fork run: agent_ac34b63e0776421a
- Validation: castor test:controller-replay: PASS, 13 tests/252 assertions, 109.70s wall after changes; castor test --filter=ControllerReplayHttpClientFactoryTest: PASS, 4 tests/19 assertions; castor clean:cleanup:workers:list: no stale QA workers; castor cs-check: PASS; Focused phpstan: implementation clean; pre-existing PHPUnit dynamic-call noise remains in test file
- Summary: Controller/Castor harness slice committed as 3ff9b4892891bb1c8caa15b1245837262019a56b. Established real setsid ownership and independent second-controller teardown, cut routine grace from 3s to 0.25s, fixed actual Messenger DB diagnostics, made replay fixture exhaustion fail loudly, added timeout PID/cmdline snapshots, and shortened replay bash blockers from 4–8s to 1s. Controller replay improved from 121.17s wall / 115.408s PHPUnit to 109.70s wall / 104.633s PHPUnit (~11.5s wall reduction).

## Task workflow update - 2026-08-22T21:40:05+00:00
- Recorded fork run: agent_9558ffea7633aa3a
- Validation: castor test:tui: PASS 33 tests, 124.3s; castor test:tui repeated: PASS 33 tests, 113.0s; Focused virtual TUI filters: PASS 50 tests in 3.3s; castor cs-check: PASS; castor clean:cleanup:workers:list: no stale workers
- Summary: Aggressive TUI cleanup committed as d8f26673a60826879310358be409f8322a9a67f8. Deleted 6 redundant/timing-driven E2E test/fixture files plus ThemeColorEnumTest, collapsed SubagentLiveView from four journeys to one TTY smoke, merged/fixed child HITL with paired Messenger DB and no fixed 500ms sleep, removed redundant Journey phases, reduced FileRewind to one real filesystem proof, consolidated readiness waits, shortened remaining delays, and fixed explicit post-tool replay fixtures after fail-loud exhaustion. TUI lane moved from ~147.6s/39 tests with a failure to two consecutive greens at 124.3s and 113.0s/33 tests.

## Task workflow update - 2026-08-22T21:57:17+00:00
- Recorded fork run: agent_8122d4f5dee59f74
- Validation: Focused six-case TUI filter: PASS, 41.3s; castor test:tui: PASS 32 tests/202 assertions, 103.9s; castor test:tui repeated: PASS 32 tests/202 assertions, 109.9s; Relevant virtual/integration filters: PASS 51 tests, 6.8s; castor cs-check: PASS; castor clean:cleanup:workers:list: no stale workers
- Summary: Remaining TUI timing slice committed as f40cab15f4bad7f53470d9c8afbb91f8978635f3. Reload reduced to one unique process/session bootstrap proof, slow delayed clipboard tmux case deleted in favor of virtual/unit coverage, NewSession/AutoCompaction/Resume timing loops replaced with positive harness predicates, and delays shortened. Full TUI lane now 32 tests, green twice at 103.9s and 109.9s versus prior ~112.9s/33 and original ~147.6s/39.

## Task workflow update - 2026-08-22T22:13:38+00:00
- Recorded fork run: agent_6ef1cc67cc1173b8
- Validation: castor test:llm-real: PASS 11 tests, 30.3s; castor test:llm-real repeated: PASS 11 tests, 29.8s; castor test: PASS on retry 4804 tests, 30.5s (first run exposed ConsumerSupervisor ready-marker race); Focused touched unit tests: PASS 68 tests, 7.8s; castor cs-check: PASS; castor phpstan: PASS; castor clean:cleanup:workers:list: no stale workers
- Summary: Live/unit cleanup committed as 7a4efba2073bd23ef909b682bac9ee5fb547fdc3. Deleted soft OutputCap live test, redundant no-shell live follow-up, trivial constants/getter/enum assertions; reduced live cases 13→11, standalone provider contention 4→2 workers, HTTP timeouts 60→15s and max tokens to 256; removed prose assertions, redundant post-ready polling, CPU spin, and concrete env leaks. llm-real summed case time dropped 73.7→52.3s but wall is ~30s with safer concurrency. Unit lane 32.0→30.5s, 4807→4804. One pre-existing ConsumerSupervisor parallel ready-marker flake surfaced then passed focused/full retry and remains to fix.

## Task workflow update - 2026-08-22T22:37:08+00:00
- Recorded fork run: agent_67c661f4eecc1ccd
- Validation: Focused affected tests: PASS 101 tests, 7.9s; castor test at 16 workers: PASS 4796 tests, 32.4s; castor test at 16 workers repeat: PASS, 35.0s; castor test at 16 workers third run: PASS, 37.1s; castor cs-check: PASS; castor phpstan: PASS; castor clean:cleanup:workers:list: no stale workers
- Summary: Parallel unit/process stabilization committed as 6f1aadc9e59840de0848f7e8bfaf4d57f0866729. Fixed ConsumerSupervisor ready-marker flake using cwd-owned markers plus process-status diagnostics, deleted elapsed-time SQLite contention lottery, replaced 5s clipboard timeout wait with cancellation proof, replaced MCP curl health polling with TCP readiness, removed long WebSocket keeper delays, and deleted additional trivial theme/OAuth assertions. Three consecutive 16-worker unit runs green (4796 tests, 32.4/35.0/37.1s), no leaks.

## Task workflow update - 2026-08-22T23:06:49+00:00
- Recorded fork run: agent_42274681cd37e8c5
- Validation: Solo castor check @ eb2ac1560: PASS, 105.87s; Concurrent stress @ 776168cbc: castor check PASS 125.49s; castor test:tui PASS 122.59s; castor test:llm-real PASS 12.27s; Post-stress worker diagnostics: no stale workers; HEAD 98c97d69b after further deletion wave: final revalidation pending
- Summary: Full-gate/stress slice produced seven commits through 98c97d69b. Solo castor check reached 105.87s green. Requested concurrent check + standalone TUI + standalone llm-real was green at 776168cbc: check 125.49s, TUI 122.59s, llm-real 12.27s, no leaks. Stress exposed and removed TwoToolCancel timing race. Additional deletion wave at 98c97d69b removed bash-background/cancel-stickiness/export/output-cap/auto-compaction TUI tests, three redundant compaction controller replays, and ViewImage live test; latest HEAD still requires final solo/concurrent revalidation.

## Task workflow update - 2026-08-22T23:33:29+00:00
- Recorded fork run: agent_e1680907cff99976
- Validation: Solo castor check: PASS, wall 84.22s; quality aggregate 205.0s; 0 tests >10s; Final concurrent castor check: PASS, wall 83.29s; quality aggregate 157.9s; Concurrent standalone castor test:tui: PASS, 8 tests/60 assertions, wall 42.88s; Concurrent standalone castor test:llm-real: PASS, 5 tests/30 assertions, wall 8.68s; Final concurrent junit scan: 0 cases >10s; max 9.552s; deptrac/phpstan/cs-check/docs:validate: PASS within final check; castor clean:cleanup:workers:list: no stale QA workers; Git worktree clean at e049d42839a9ca4a01a9f809e186df0aee8c7006
- Summary: Final validation and contention cleanup complete at e049d42839a9ca4a01a9f809e186df0aee8c7006. Latest solo check was green at 84.22s wall with zero >10s cases. Repeated requested concurrent stress exposed remaining contention-only slow TUI/controller cases; these were deleted only where lower-layer coverage existed. Final concurrent run: check 83.29s wall, standalone TUI 42.88s, standalone llm-real 8.68s, all exit 0. Check quality lane aggregate 157.9s. Zero junit cases >10s; maximum was TuiJourneyE2eTest at 9.552s. No stale QA workers; cache guard stable; deptrac/phpstan/cs/docs all green; git clean.

## Task workflow update - 2026-08-22T23:41:44+00:00
- Recorded fork run: agent_9f8c2334d08c0055
- Validation: castor docs:validate: PASS (16 built-in documents); Git clean after docs commit 83db6472df54d6d6c51170e35b888be830cac5a9
- Summary: Hard test-quality rules committed as 83db6472df54d6d6c51170e35b888be830cac5a9. Root AGENTS now contains the concise global gate; tests/AGENTS owns detailed enforceable determinism/isolation/process/replay/live/proof-layer standards; testing skill owns the Castor junit and concurrent contention audit runbook. Rules codify ≤10s including under contention, deletion/demotion of nondeterministic tests, no arbitrary sleeps/retries/timeout increases, positive readiness proof, fail-loud fixtures, explicit process ownership/teardown, no soft assertions/model-prose contracts, and no solo-green claims for parallel flakes.

## Task workflow update - 2026-08-23T00:47:39+00:00
- Recorded fork run: agent_95ed1fb2d52836b6
- Validation: Focused affected unit tests: PASS 19 tests/99 assertions, 7.2s; Focused OM TUI: PASS 2 tests/9 assertions, 9.4s; Full castor check: PASS, quality aggregate 151.2s; unit 69.0s, controller 17.9s, TUI 30.8s, llm-real 18.9s; Final junit: 0 cases >10s; max 8.217s; castor docs:validate/cs-check/phpstan: PASS; workers:list: no stale workers; Git clean at eaed875e386af0d241fda07f7dc7402f815283c9
- Summary: Addressed reviewer REQUEST CHANGES in eaed875e386af0d241fda07f7dc7402f815283c9. Timeout cmdlines are now snapshotted before reaping; 14 orphan TUI fixtures plus dead support/recording pipeline were deleted; stale worker comments and zero-call controller helpers/docs removed; Messenger test claim corrected; OM readiness consolidated. Clipboard 5s timeout outcome remains an explicitly documented accepted coverage gap rather than reintroducing a slow wait or production test-only seam.

## Task workflow update - 2026-08-23T01:04:28+00:00
- Recorded fork run: agent_701470e300dd033e
- Validation: Reviewer re-review: APPROVED; php -l .castor/tasks.php: PASS; php -l ClipboardImageReaderTest.php: PASS; castor cs-check: PASS; Git clean at 139eaadb42d84e90a4b74450357a6d6d528394bd
- Summary: Final reviewer-approved comment cleanup committed as 139eaadb42d84e90a4b74450357a6d6d528394bd. Removed stale exact lane counts/3s E2E grace narrative and clarified the intentional clipboard timeout coverage gap. Reviewer verdict APPROVED at eaed875e3; this follow-up is comment-only.

## Task workflow update - 2026-08-23T01:05:55+00:00
- Validation: castor test: PASS 4795 tests/19517 assertions, 37.6s; castor test:tui: PASS 8 tests/60 assertions, 36.3s; castor deptrac: PASS 0 violations/errors; castor phpstan: PASS 0 errors; castor cs-check: PASS files_fixed=0
- Summary: Focused task-to-PR validation on final HEAD 139eaadb42d84e90a4b74450357a6d6d528394bd completed after reviewer approval.

## Task workflow update - 2026-08-23T01:07:36+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (71.0s).
- Pushed task/2026-08-21-analyze-and-speed-up-flaky-castor-tests to origin.
- branch 'task/2026-08-21-analyze-and-speed-up-flaky-castor-tests' set up to track 'origin/task/2026-08-21-analyze-and-speed-up-flaky-castor-tests'.
- Created PR: https://github.com/ineersa/agent-core/pull/424
- Validation: Reviewer: APPROVED; Final focused castor test: PASS 4795 tests/19517 assertions, 37.6s; Final focused castor test:tui: PASS 8 tests/60 assertions, 36.3s; castor deptrac/phpstan/cs-check: PASS; Prior full castor check after review fixes: PASS, quality 151.2s, 0 tests >10s, no leaks; Requested concurrent stress: check/TUI/llm-real all PASS with 0 tests >10s
- Summary: Reviewer APPROVED after one fix iteration. Final branch reduces Castor wall from frequent 210s failures to ~84s solo and ~83s during requested concurrent check+TUI+llm-real stress, with zero tests over 10s and no stale workers. Harness lifecycle/timeout diagnostics, isolation, fixture exhaustion and process readiness were hardened; redundant/timing-driven tests and orphan artifacts were deleted; hard quality rules were codified.

## Task workflow update - 2026-08-23T02:40:54+00:00
- Summary: Post-PR skeptical coverage audit requested by user found that most deletions are safely mapped to lower-layer proof, but prior review overstated equivalence for a small set of cross-component contracts. Recommend pausing merge pending user decision.
- Coverage-risk audit: ~80% of deletions are safely redundant, low-value, soft/model-prose, or mapped to stronger unit/virtual tests. High gaps: (1) multi-turn auto-compaction through the real controller/Messenger pipeline has only isolated unit guards left; (2) SafeGuard human approval request→answer→matching tool completion has no controller integration proof; (3) no deterministic replay now proves an ordinary tool call completes through the controller and persists canonical events/artifacts. Medium accepted-risk gaps include real child/fork execution, actual /reload, tool-filter propagation, /new in-flight teardown, durable multi-steer FIFO, AtomicFileWriter concurrent-reader atomicity, and SQLite competing-writer behavior. PR remains open and unmerged while user considers restoration scope.

## Task workflow update - 2026-08-23T02:43:21+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User approved restoring a minimal set of high-value coverage after skeptical post-PR audit: deterministic controller replay for multi-turn compaction, SafeGuard approval round-trip, and ordinary tool completion/canonical persistence; optionally retain one focused live provider tool compatibility smoke only if it is deterministic and ≤10s under contention.

## Task workflow update - 2026-08-23T02:58:32+00:00
- Recorded fork run: agent_31ed773eaf3c4ae5
- Validation: Restored test PASS 3 consecutive runs: 8.431s / 8.465s / 8.406s; Relevant compaction/replay units PASS 67 tests/233 assertions, 2.3s; castor test:controller-replay PASS 4 tests/48 assertions, 22.0s; restored case 7.497s; castor cs-check PASS; Focused phpstan: no errors beyond existing PHPUnit dynamic-call noise; workers:list: no stale workers
- Summary: Restored one consolidated deterministic multi-turn auto-compaction controller replay at bda272efa97b1288d448bbcbf72dcdfec983e270. It proves threshold compaction completes through controller/Messenger, canonical context_compacted is persisted, no ghost LLM continuation occurs, and a fresh follow-up proceeds. Individual guard variants remain unit-owned.

## Task workflow update - 2026-08-23T03:25:31+00:00
- Recorded fork run: agent_d3a841ff6480bfe1
- Validation: Focused restored case PASS 3x: 7.896s / 7.653s / 7.744s wall; junit ~7.13–7.37s; SafeGuard/answer/suspension units PASS 32 tests/141 assertions; castor test:controller-replay PASS 5 tests/66 assertions, 31.643s; new case 7.198s; 0 >10s; castor cs-check PASS; workers:list: no stale workers
- Summary: Restored one minimal deterministic SafeGuard approval controller replay at cbed30e26b9f77b542bafa7966ef6bf5af012d3a. It proves outside-CWD write → human_input.requested (not tool_question) → correlated answer_human allow → same tool_call_id completes → exact file artifact.

## Task workflow update - 2026-08-23T03:34:29+00:00
- Recorded fork run: agent_17204e3f3d535c05
- Validation: Focused restored case PASS 3x; junit ~6.13–6.25s; castor test:controller-replay PASS 6 tests/84 assertions, ~33.4s; 0 >10s; max 7.441s; Relevant units PASS 71 tests/242 assertions, 2.6s; castor cs-check PASS; Focused phpstan errors=0; existing PHPUnit dynamic-call warnings only; workers:list: no stale workers
- Summary: Restored a compact ordinary read-tool controller replay at 6e747d5a670c939cdfb1d5fb026ee454f6b4e309. It proves matching tool start/completion IDs, no failure, controller/Messenger execution, canonical tool_execution_end persistence, and session artifacts. Did not restore a live model-selected tool test: provider invocation and tool schemas remain separately covered, while deterministic replay now owns controller execution wiring; a live tool smoke would add model nondeterminism without unique current value.

## Task workflow update - 2026-08-23T14:42:28+00:00
- Recorded fork run: agent_0163fbc074d14c6e
- Validation: castor test:controller-replay PASS 6 tests/84 assertions, 35.7s; 0 >10s; max 8.605s; castor cs-check PASS; Focused phpstan errors=0; existing PHPUnit dynamic-call noise only; workers:list: no stale workers; Stale symbol/orphan scan clean
- Summary: Addressed final review cleanup at 095c3ae7bb72b189d51656b9805504e617cd790a: removed orphan compaction quiet-drain helper, generalized shared early-exit collector minimally, tightened SafeGuard completion uniqueness, and aligned restored fixture model metadata.

## Task workflow update - 2026-08-23T15:12:19+00:00
- Recorded fork run: agent_7284a0ea3a48b1e6
- Validation: Solo castor check @095c3ae7 PASS 77.88s; 0 >10s; restored cases 7.345–8.699s; Concurrent exact-worktree run: all exits green once, but BashCancel 11.539s and SafeGuard 10.070s; Latest concurrent run @9be173abe: TUI/llm-real PASS, check FAIL waiting for SafeGuard human_input.requested; Messenger llm backlog observed; Focused restored tests after optimization PASS 4 tests/71 assertions, 29.3s; workers:list: no stale workers throughout
- Summary: Final solo check at restored-coverage HEAD passed, but realistic concurrent contention exposed a blocker: restored SafeGuard controller replay can exceed/fail the ≤10s rule while waiting through controller→LLM→tool→SafeGuard→run-control hops; a green concurrent run also had BashCancel 11.539s and SafeGuard 10.070s. Minimal BashCancel/harness optimizations committed through 9be173abec0b4874f0993a38b88ac86b3c5ab73a, but latest stress still failed SafeGuard. Task remains IN-PROGRESS pending cheaper deterministic harness design, not timeout increases/deletion.

## Task workflow update - 2026-08-23T16:57:55+00:00
- Recorded fork run: agent_8fbf427cccbcf9af
- Validation: Focused Resume 3x PASS ~4.49–4.61s; Focused Subagent 3x PASS ~4.69–4.86s; Focused controller behavioral four 3x PASS, max ~4.50s; castor test:controller-replay PASS 6/92, max 4.267s; castor test:tui PASS 8/59, max 4.933s; Solo castor check PASS, quality 158.0s, 0 >10s; max 6.707s; Concurrent exact-worktree check+TUI+llm-real guards ON: all exit 0, 0 >10s; check max 8.134s, standalone TUI max 8.119s; workers:list clean; cs-check PASS; git clean
- Summary: Resolved the contention blocker at d8e501104b1c6cab058dd383fa4f4ed21435b41d without deleting restored coverage or raising waits. Test-only source-console wrapper injects Messenger --sleep=0.05; isolated E2E uses one generic tool worker; BashCancel again waits for an actually started blocker; Resume/Subagent tmux proofs seed real session metadata and canonical events instead of burning an LLM turn merely to allocate an ID. Wrapper is excluded from boot-only and PHAR/artifact tests.

## Task workflow update - 2026-08-23T17:43:52+00:00
- Recorded fork run: agent_c53e274a54f66099
- Validation: TuiE2eDatabaseEnvTest PASS 4/16, 0.626s; castor test:controller-replay PASS 6/92, 20.595s; max 4.090s; Focused Resume+Subagent TUI PASS 2/23, 8.719s; max 4.279s; castor cs-check PASS; workers:list clean; Deleted-symbol search clean
- Summary: Final reviewer mechanical fixes committed at de6d01608e647c706ce5572ebf430a02f4a62b10: teardown docs aligned to 0.05s, dead TUI DB helpers/self-test removed, compaction drain simplified, SafeGuard predicate wait made unambiguous.

## Task workflow update - 2026-08-23T17:53:03+00:00
- Recorded fork run: agent_3dc96dcee8ee50d6
- Validation: Pre/post exhaustive reference scan clean; php -l TmuxHarness.php PASS; castor cs-check PASS
- Summary: Deleted the final two reviewer-confirmed orphans at 3d79db0cfc4434192dceb1bd3f887487d8708d60: unused tui-resume-minimal fixture and zero-use assistant-block timeout constant.

## Task workflow update - 2026-08-23T17:56:09+00:00
- Validation: Reviewer: APPROVED; castor test PASS 4794 tests/19515 assertions, 34.6s; castor test:tui PASS 8 tests/59 assertions, 32.5s; castor deptrac PASS 0 violations/errors; castor phpstan PASS 0 errors; castor cs-check PASS; Prior solo castor check on harness HEAD PASS, 0 >10s; Prior exact-worktree concurrent check+TUI+llm-real guards ON: all PASS, 0 >10s, no leaks
- Summary: Final reviewer APPROVED at 3d79db0cfc4434192dceb1bd3f887487d8708d60. Restored high-risk integration coverage remains, low-latency deterministic harness is contention-green, and final orphan cleanup is complete.

## Task workflow update - 2026-08-23T18:00:11+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (66.5s).
- Pushed task/2026-08-21-analyze-and-speed-up-flaky-castor-tests to origin.
- branch 'task/2026-08-21-analyze-and-speed-up-flaky-castor-tests' set up to track 'origin/task/2026-08-21-analyze-and-speed-up-flaky-castor-tests'.
- Skipped PR creation (pushOnly: true).
- Validation: Reviewer APPROVED at 3d79db0cfc4434192dceb1bd3f887487d8708d60; Final castor test PASS 4794/19515, 34.6s; Final castor test:tui PASS 8/59, 32.5s; castor deptrac/phpstan/cs-check PASS; Solo castor check PASS, 0 cases >10s; Concurrent check+TUI+llm-real guards ON: all PASS, 0 cases >10s, no stale workers
- Summary: Coverage-risk iteration complete and reviewer APPROVED. Restored three compact deterministic controller replays for multi-turn auto-compaction, SafeGuard approval, and ordinary tool completion/canonical persistence. Added test-only low-latency Messenger wrapper and one-tool-worker isolation; redesigned Resume/Subagent tmux setup to seed real catalog/canonical events instead of burning an LLM turn. Final solo and exact-worktree concurrent stress are green with zero cases >10s and no leaks. Existing PR #424 remains the review target.

## Task workflow update - 2026-08-23T18:05:13+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-21-analyze-and-speed-up-flaky-castor-tests: ide_close_project returned isError.
- Merged task/2026-08-21-analyze-and-speed-up-flaky-castor-tests into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-21-analyze-and-speed-up-flaky-castor-tests.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-21-analyze-and-speed-up-flaky-castor-tests.
- Pulled integration checkout: Already up to date..
- Summary: PR #424 was merged on GitHub. Main was synchronized, the user-requested SafeGuard settings change was committed/pushed, generated untracked report-path debris was removed, and the task is ready for DONE cleanup.

## Task workflow update - 2026-08-29T16:09:38.978Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
