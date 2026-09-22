# Prune low-value tests by at least 30% and replace timing-dependent proofs

## Goal
## Goal

Make the test suite smaller and more deterministic while preserving useful regression, safety, runtime, and user-visible behavior coverage. User requests at least 30% fewer low-value tests. Remove redundant proof, not just assertions or reporting entries. Rewrite valuable flaky or expensive cases at the lowest correct layer.

## Baseline

Repository revision: a3a5b620ebde1384f5772f9e14fcac6f7b2ed709.
Reports: var/reports/qa-20260920-001121-364-379463d7.
Count leaf <testcase> entries, including dataset expansions, across the four executed test lanes. Group by testcase file path, not namespace. Do not add parent testsuite totals, which double-count nested suites.

- Unit/integration: 5,123 executed cases. File-path groups: CodingAgent 3,061; TUI 1,235; AgentCore 460; Platform 177; configured extension suites 190.
- Controller replay: 11 cases.
- TUI: 9 cases.
- LLM-real: 5 cases.
- Total gate baseline: 5,148 executed cases. A net 30% reduction requires at least 1,545 fewer cases, leaving at most 3,603, including replacement tests.
- Static discovery found 619 *Test.php files under tests and extension directories. This is NOT the execution denominator.
- Full gate wall time: 137.1s. Parallel lane durations: unit 39.5s, controller replay 40.6s, TUI 22.8s, live 10.1s. Do not sum concurrent lane wall times to claim gate duration.
- Slowest recorded case: TuiJourneyE2eTest, 6.667s; provider-error terminal test 5.978s. No recorded case exceeded 10s in this successful baseline. One passing report does not establish flake freedom.
- Prior failure evidence: provider-error and OM tests entered fail-closed teardown before supported shutdown finished. PR #514 fixed these with existing positive pane-exit synchronization. Do not undo that ownership proof.

## Scout audit and confidence

Four read-only scouts audited separate areas, then performed a source-verified second pass. All confirmed testing skill and tests/AGENTS prerequisites. Artifacts from parent session 60:
- agent_e549eee4b99024fb: AgentCore and Platform.
- agent_f3628b29cec2f91a: CodingAgent except Runtime/Tool.
- agent_f91dab84ed210b43: Runtime and Tool.
- agent_cc45521c524ca267: TUI and configured extensions.

The audit found real duplication and timing-dependent tests, but has NOT justified 1,545 safe removals. Treat 30% as an outstanding acceptance target, not an achieved or proven-safe estimate. Further contract-level inventory is required. Scouts corrected initial over-broad recommendations: matching expected outputs do not make distinct input types, protocol states, safety boundaries, or enum mapping branches redundant. Parameterizing two scenarios still executes two cases and is not a count reduction.

## First deletion candidates, verify retained proof before editing

- Platform/Bridge/Generic/DurableResultConverterTest::streamOptionRoutesToStreamPath asserts a locally-created stream flag and return type. convertsSingleValidToolCall already exercises meaningful streamed conversion.
- Tui/Picker/PickerOverlayTest: private screen-property reflection checks and initial getter tests duplicate mount/close behavior. testMountSetsIsOpen already asserts initial closed/null state and mounted state.
- CodingAgent/Config/AgentsConfigTest::testFromRawWithMaxAgents duplicates explicit hydration already covered by testDefaultMaxAgentsIsFour.
- CodingAgent/Agent/Artifact/AgentArtifactRegistryTest::testCreateWithDifferentArtifactIds duplicates two-ID creation exercised by testListReturnsAllEntriesForParent. Corrupt-registry create/get/list cases share one reader; audit consolidation while retaining each unique public-entry safety boundary.
- CodingAgent/Tool/RegistryBackedToolboxTest::testImplementsToolboxInterface tests a declaration rather than behavior. Inspect overlapping no-op cache-identity tests separately.
- CodingAgent/Runtime/Projection/TranscriptProjectorTest: narrower monotonic-sequence case overlaps the full-suite sequence proof. Preserve update-in-place, canonical reconstruction, ordering, and cancellation cases.
- RuntimeExceptionBoundaryTest::captureEnabledReturnsNormally and JsonlProcessAgentSessionClientShutdownTest::testShutdownIsSafeAndIdempotentWithoutProcess use weak assertion-count/success-only proof. Compare stronger event-dispatch and observable shutdown-state cases.
- Continue config/parser/auth/template/registry audit. Earlier speculative estimates are not approved deletion lists. Do not reduce credential/path/command security matrices just because multiple rows fail identically.

## Determinism and expensive-proof work

1. Rewrite BashToolTest::testProcessFinishesWhilePromptBlocksReturnsCompletedOutput. It combines shell sleep 0.1, adapter usleep(200000), and 50ms polling to manufacture a completion race. Preserve the real regression: a process that completes while the background prompt is pending must return completed output, not background it. Use an owned completion barrier and positive terminal process-record state through existing test facilities/injected prompt adapter. A marker before process exit alone is insufficient.
2. Audit ControllerReplayHttpClientFactoryTest::sseChunkDelayMsSpacesSseStreamObservationsAcrossWallClock and StreamPacingHttpClient. The test inserts 120ms delays and checks >=100ms gaps. Determine all fixture callers before removing test-only pacing infrastructure. Keep necessary streaming/cancellation contracts with deterministic synchronization; remove unsupported/dead pacing paths. No new abstraction solely to preserve a useless helper test.
3. Rewrite RawWebSocketResultTest::testFragmentedMessageBufferTimeoutIsBoundedAndInvalidatesCache where feasible. Preserve timeout exception/cause, cache invalidation, and diagnostic contract; remove scheduler-sensitive elapsed-time proof. MockClock::sleep in connection-cache expiry tests is deterministic and must not be treated as a real delay.
4. TuiSubagentProgressE2eTest is a candidate for virtual mounted-transcript proof; SubagentResultRendererTest already covers card/handoff content. Verify viewport and resumed-transcript obligations before removing terminal coverage.
5. Demote OM background-status placement/clearing to a virtual mounted/status-row proof if it preserves the actual contract. Existing command-service tests alone do not prove placement. Retain one real extension command/boot smoke if unique.
6. Audit startup/resource-expansion duplication against virtual startup/LoadedResourcesWidget coverage and existing artifact/terminal boot smoke.
7. Keep bounded readiness polling with child liveness, explicit ownership, and deterministic teardown. Real process timeout fixtures are not automatically arbitrary sleeps. Do not replace synchronization with longer waits.

## Protected behavior and quality review

Preserve independent proof for approvals and SafeGuard, credential/redaction/path boundaries, process cancellation/reaping/PID reuse, session locks, canonical event replay/idempotency, HITL, tool ordering, storage/migrations, public extension protocols, provider wire/stream contracts, release integrity, and real terminal/artifact boundaries unavailable below.

High-count does not mean low-value: five large TUI unit clusters total 238 cases but only about 0.29 aggregate test seconds. Mapper/projector tests often protect different boundaries. Custom enum mappings are product logic, unlike PHP enum intrinsic tests. Do not delete these matrices by quota.

## Implementation plan and ownership

1. Preserve a reproducible baseline inventory in the task evidence before report files rotate. Record method, dataset, file, lane, and runtime separately.
2. Complete a deletion/rewrite manifest. For each cluster record the contract, defect class it can catch, duplication/low-value evidence, retained test names, expected count change, and risk. Distinguish confirmed removals from candidates awaiting verification.
3. Main owns inventory reconciliation and integration. Bounded implementation slices can cover core/provider, app/configuration, runtime/tools, and TUI/extensions. Use sequential owners in one task worktree or separate branches/worktrees with explicit integration order; no concurrent writers in one worktree.
4. Remove verified dead/tautological/duplicate tests first. Delete helpers and fixtures only when no supported callers remain. Rewrite unique-value timing tests and demote expensive presentation journeys with retained-proof mapping.
5. Run focused Castor checks per slice and compare failure detection for affected regressions. Do not claim preserved coverage quality from green tests or a line-coverage percentage alone.
6. Independent reviewer checks every deleted contract and rejects count gaming. Record achieved net count, retained/gained proof, any intentionally lost proof, runtime differences, and unresolved risks.
7. If contract mapping still cannot justify 30%, report the exact shortfall and proposed coverage trade-offs to the user. Do not silently lower the target or delete safety proof to meet it.

## Non-goals

No production feature changes, published API changes, skips/quarantines, moving tests outside discovery, lane exclusions, relaxed assertions solely for green runs, retry-until-green, longer timeout fixes, arbitrary sleeps, or giant loops/tests that conceal unchanged scenario count. Use existing project/framework test facilities.

## Validation

Use Castor only. Targeted checks during implementation; move_task to CODE-REVIEW owns full castor check. For known contention failures use the relevant concurrent lanes, not solo-green proof. Every retained case must satisfy the existing <=10s rule or document a legitimate unique external/process exception. Keep all required gate lanes and process/cache leak guards. Report before/after per-lane counts, per-case durations, slowest cases, wall time, and retained-contract manifest. Post-merge full gate remains required.

## Acceptance criteria
- At least 30% net reduction from the frozen 5,148-case gate baseline: at most 3,603 executed cases, counting replacements and dataset expansions. Any baseline change must be explained, not used to weaken the target.
- Every removed or demoted meaningful contract has a named retained proof or an explicit user-approved loss. Reviewer verifies the mapping; no bulk quota-driven deletion.
- Remove low-value getter, declaration, intrinsic, tautological, private-implementation, and duplicate proofs where source/caller evidence supports removal. Preserve distinct boundary and state-transition behavior.
- Rewrite or remove arbitrary sleep/time-window tests. Retain required real-process/protocol coverage using positive deterministic synchronization and explicit resource ownership.
- Do not game counts with exclusions, skips, quarantines, giant loop tests, or provider-to-loop conversion. Shared fixtures/data providers count as maintainability work, not fewer executed scenarios.
- Validate relevant concurrent lanes for known contention failures; preserve <=10s case rule, fail-closed teardown, and process/cache leak guards without timeout increases.
- Record before/after inventory, runtimes, retained-proof mapping, and independent approval. Full transition-owned castor check passes. If 30% conflicts with evidence-backed coverage preservation, report the shortfall and obtain an explicit scope decision rather than mark complete.

## Workflow metadata
Status: DONE
Branch: task/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/518
PR Status: merged
Started: 2026-09-20T00:49:00+00:00
Completed: 2026-09-21T01:54:46+00:00

## Work log
- Created: 2026-09-20T00:30:38+00:00

## Task workflow update - 2026-09-20T00:40:57+00:00
- Summary: User approved revised acceptance: 30% is a stretch goal, not a mandatory deletion quota. The acceptance amendment below supersedes all conflicting percentage requirements in the original title, goal, implementation plan, and acceptance criteria.
- Acceptance amendment, approved by user: (1) Remove every confirmed low-value or redundant test found in the agreed audit scope. (2) Rewrite valuable timing-dependent tests with deterministic synchronization. (3) Demote expensive tests where cheaper proof covers the same contract. (4) Report net case reduction, runtime changes, and any lost behavioral coverage. (5) Keep 30% as an ambition, without deleting useful tests to reach it.
- The agreed audit scope remains the core, platform, CodingAgent, TUI, and configured extension suites identified in the task. Complete the candidate review and record retained-proof mappings and reasons for keeping rejected candidates. Do not stop at an arbitrary smaller percentage.
- The 5,148-case baseline remains the comparison reference. Removing 1,545 cases or reaching at most 3,603 cases is NOT a completion requirement. Falling short of 30% alone does not block completion or require further scope approval. Intentional loss of meaningful coverage still requires explicit review and user approval.
- All nonconflicting quality gates remain: no count gaming, skips, discovery exclusions, arbitrary sleeps, timeout increases, or weakened teardown. Validate relevant concurrent lanes for known contention failures, preserve required process/protocol proofs, and obtain independent review plus the transition-owned full Castor gate.

## Task workflow update - 2026-09-20T00:49:00+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs/.idea.
- Summary: Starting implementation under amended acceptance: remove verified redundant tests, stabilize valuable timing proofs, demote unnecessarily expensive proofs; 30% remains a stretch goal.

## Task workflow update - 2026-09-20T00:49:47+00:00
- Ownership: owner=fork; fork_run=none; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=Runtime/Tool and Platform timing-proof rewrites and verified redundant tests; outcome=assigned; commit=none
- Sequential implementation slices: Runtime/Tool/Platform first, TUI/extensions second, remaining app/core pruning third. Main owns baseline reconciliation and final focused validation. Forks are justified by independent module boundaries and substantial process/virtual-test investigation. No parallel writers.

## Task workflow update - 2026-09-20T01:03:20+00:00
- Ownership: owner=fork; fork_run=agent_85940f8862682354; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=Runtime/Tool/Platform correction: replace child busy-spin barrier and inspect async keeper teardown; outcome=assigned; commit=none

## Task workflow update - 2026-09-20T01:09:46+00:00
- Ownership: owner=fork; fork_run=agent_85940f8862682354; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=Runtime/Tool/Platform cleanup; outcome=completed; commit=none
- Runtime slice removed 7 cases and unused replay pacing helper. Parent required busy-spin correction and explicit async ownership; corrected handoff reports focused 260 cases passing plus concurrent barrier/timeout proof. Full gate not run.
- Ownership: owner=fork; fork_run=none; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=TUI/extensions verified duplicate removal and expensive presentation proof demotion; outcome=assigned; commit=none

## Task workflow update - 2026-09-20T01:19:28+00:00
- Ownership: owner=fork; fork_run=agent_b450d7da7b3c395f; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=TUI cleanup and journey duration diagnosis; outcome=assigned; commit=none
- TUI slice reports net -15 cases and 9→7 terminal cases. One normal TUI lane observed TuiJourneyE2eTest 10.213s; later solo green is insufficient. Continue diagnosis rather than accept rerun as resolution.

## Task workflow update - 2026-09-20T01:28:05+00:00
- Ownership: owner=fork; fork_run=agent_b450d7da7b3c395f; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=TUI cleanup and journey duration diagnosis; outcome=completed; commit=none
- TUI journey hardens canonical completion and removes swallowed timeout before shutdown. Concurrent normal TUI and controller-replay lanes passed; journey 5.566s, no consumer escalation in latest log. TUI terminal count 9→7.
- Ownership: owner=fork; fork_run=none; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=remaining CodingAgent configuration/artifact/auth/template and AgentCore verified low-value test cleanup; outcome=assigned; commit=none

## Task workflow update - 2026-09-20T01:36:43+00:00
- Ownership: owner=fork; fork_run=agent_0e9fe285dc5580ef; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=remaining app/core candidate cleanup; outcome=completed; commit=none
- App/core slice removed 5 verified duplicates/declaration checks; core and app suite passes. Report explicitly limits audit completeness.
- Ownership: owner=fork; fork_run=agent_85940f8862682354; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=bound FIFO parent release under child failure; outcome=assigned; commit=none

## Task workflow update - 2026-09-20T01:39:43+00:00
- Ownership: owner=fork; fork_run=agent_85940f8862682354; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=bound FIFO parent release under child failure; outcome=completed; commit=none
- Ownership: owner=fork; fork_run=agent_b450d7da7b3c395f; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=retain unique overheight terminal writer proof while keeping deterministic journey completion; outcome=assigned; commit=none

## Task workflow update - 2026-09-20T01:45:12+00:00
- Ownership: owner=fork; fork_run=agent_b450d7da7b3c395f; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=retain actual shell-to-follow-up runtime regression and investigate residual shutdown duration; outcome=assigned; commit=none
- Parent rejected service-only pending-call replay as sufficient replacement for actual follow-up after inline shell. Latest concurrent attempt with follow-up restored overheight observed 10.513s and idle tool consumer shutdown escalation. Prior completion-only fix did not establish root cause. Do not claim timing issue solved by dropping the unique integration obligation.

## Task workflow update - 2026-09-20T01:48:19+00:00
- Ownership: owner=fork; fork_run=agent_b450d7da7b3c395f; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=TUI candidate cleanup; outcome=blocked; commit=none
- Ownership: owner=fork; fork_run=agent_85940f8862682354; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=diagnose idle Messenger consumer shutdown escalation blocking deterministic TUI validation, minimal lifecycle fix only if source evidence supports; outcome=assigned; commit=none

## Task workflow update - 2026-09-20T01:57:14+00:00
- Ownership: owner=fork; fork_run=agent_85940f8862682354; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=idle Messenger shutdown contention investigation; outcome=blocked; commit=none
- Owned held-lock probe reproduced deferred PHP SIGTERM during SQLite busy wait; configured busy timeout and shutdown grace both 5s. This is a plausible mechanism matching historical escalation timings, not a captured stack/proc sample of the actual slow QA consumer. Subsequent green concurrent runs did not reproduce escalation and do not resolve blocker. No runtime production changes retained; broader Messenger/Doctrine contention fix exceeds this test-only slice.
- Ownership: owner=main; fork_run=none; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=aggregate diff/count review, full unit lane and focused static validation, record unresolved TUI duration blocker; outcome=assigned; commit=none

## Task workflow update - 2026-09-20T02:01:46+00:00
- Validation: Final parent castor test PASS: 5098 tests, 22114 assertions, 17.2s; slowest case 2.057479s. var/reports/prune-parent-final/phpunit-parallel.junit.xml.; castor phpstan PASS: 0 errors. Final castor dead-code PASS after removing two helpers made unused by demotions. castor cs-check and git diff --check PASS.; Concurrent TUI/controller replay runs functionally passed, but observed journey10.213s and10.513s exceed hard case limit. Later green concurrency runs do not resolve this defect.; No full castor check or live lane run in task-start. Baseline and standalone worker counts differ, so no claimed overall speedup.
- Summary: Implementation partially complete, still IN-PROGRESS. Net 27 executed cases removed (5148→5121 including unchanged live lane); terminal cases 9→7; 993 net lines removed across 28 files. Deterministic Bash barriers, owned WebSocket timeout watcher, stronger positive TUI completion proofs. Unresolved intermittent worker shutdown causes required TUI journey >10s. No commit/review/PR/full gate.
- Ownership: owner=main; fork_run=none; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=aggregate validation and dead helper removal; outcome=blocked; commit=none
- Evidence report: task worktree var/reports/test-pruning-summary.md. Detailed runtime/TUI/app-core/shutdown handoffs under var/reports/test-pruning-*-handoff.md. Frozen baseline JUnit copies under var/reports/prune-baseline.
- Retained proof, Runtime/Platform: removed stream-flag self-assertion, toolbox declaration check, exception boundary nonthrow-only case, no-process shutdown assertion-count case, unused pacing test/helper/option, exhaustive obsolete event-name list, and narrower seq test. Retained meaningful converter streaming, toolbox behavior, dispatched-error proof, owned shutdown/state clearing, replay streaming/cancellation, mapper/naming behavior, and full-sequence ordering proof.
- Retained proof, app: removed duplicate max_agents hydration and artifact two-ID creation plus three interface declaration cases. Retained explicit config override/validation, artifact multi-entry listing/storage, compaction lifecycle and SafeGuard registration/settings behavior.
- Retained proof, TUI: removed four private/default PickerOverlay checks, six weak/duplicate Theme aliases, two duplicate ThemeRegistry successes, two expanded outcome-color rows, startup tmux case, and subagent tmux case; added one mounted virtual subagent proof. Mount/close/layout, actual style/fallback, theme errors/collisions, outcome color and preview expansion, source/package boot, real Ctrl+R dispatch, and detailed subagent rendering remain. Shell-to-normal-follow-up and overheight real-writer obligations were restored after parent rejected insufficient lower-layer mappings.
- Rewrites preserve useful tests: owned nonblocking-parent FIFO endpoints replace Bash timing windows/busy spins; owned cancellable safety watcher replaces orphaned infinite Amp keeper; TUI uses exact shell event tails and newly appended final agent_end, no swallowed completion timeout or missing-file soft-pass.
- Blocker: idle worker occasionally survives SIGTERM until5s grace escalation. Held-lock PDO probe demonstrates a plausible deferred-signal mechanism at matching SQLite busy_timeout, but no actual failing-worker stack was captured. Broader Messenger/Doctrine contention work remains. No timeout changes or runtime fixes made; no stale QA workers reported.
- Audit completeness: candidate-led source checks across all routed areas, not exhaustive review of all619 files. Distinct safety, protocol, parser, provider, and state-machine cases retained. 30% remains stretch goal; current reduction0.52%.

## Task workflow update - 2026-09-20T02:47:40+00:00
- Summary: User explicitly prefers deletion when a test is flaky or cannot be made deterministic. This authorizes removing the demonstrated flaky TuiJourneyE2eTest rather than broadening into Messenger/Doctrine runtime changes. Record lost unique integration proof honestly; retain lower-layer contracts and other terminal proofs.
- Acceptance clarification: deletion of evidenced flaky/non-deterministic tests is preferred over preserving them or undertaking speculative runtime changes. Do not label partial lower-layer coverage equivalent to removed integration coverage.
- Ownership: owner=fork; fork_run=agent_b450d7da7b3c395f; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=delete evidenced flaky TUI journey, remove newly dead helpers/fixtures, validate remaining concurrent lanes and document coverage loss; outcome=assigned; commit=none

## Task workflow update - 2026-09-20T02:53:55+00:00
- Validation: Concurrent normal castor test:tui + castor test:controller-replay PASS with separate QA lanes/report directories: TUI6 tests/31 assertions,14.842s; each case2.278–4.178s. Controller replay11 tests/175 assertions,35.110s.; Focused surviving helper callers PASS:69 tests/430 assertions,0.306s.; Final castor dead-code and castor cs-check PASS. Parent castor docs:validate and git diff --check PASS.; Prior full unit lane PASS5098 tests/22114 assertions; latest continuation deletes one excluded TUI journey and unused fixture/helper constant only. Full castor check not run; next phase owns gate.
- Summary: Implementation phase complete under user-approved flaky-test deletion policy. Deleted evidenced flaky TuiJourneyE2eTest and its exclusive fixture/dead helper constant. Net28 cases removed (5148→5120), terminal cases9→6, net1390 lines removed across30 files. Remaining concurrent TUI/controller lanes passed. No runtime shutdown fix claimed. Ready for task-to-pr independent review and transition-owned full gate.
- Ownership: owner=fork; fork_run=agent_b450d7da7b3c395f; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=delete evidenced flaky journey and validate remaining concurrent lanes; outcome=completed; commit=none
- Ownership: owner=main; fork_run=none; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=review final deletion/helper diff, documentation check and evidence reconciliation; outcome=completed; commit=none
- Approved coverage loss: no remaining single-TUI-process test proves two shell commands followed by a normal prompt advance through the actual controller; no remaining test proves source-runtime physical writer bottom chrome under overheight /hotkeys. Lower-layer replay/rendering/hotkey and packaged boot proofs remain but are not equivalent. User explicitly preferred deletion of this demonstrated flaky integration test.
- The unresolved runtime shutdown contention mechanism still exists as a product investigation possibility; deleting the unreliable test does not fix or disprove it. It no longer blocks this test-cleanup scope under latest user instruction. No retries/timeouts/exclusions were used to hide the journey; the case was deleted.
- Skill-reference defect found by parent: .agents/skills/testing/SKILL.md lines368,372 still name deleted TuiJourneyE2eTest and TuiStartupSnapshotTest. Reported, not edited because skill repair requires explicit user scope authorization. tests/AGENTS.md stale journey pointer was corrected. Earlier cs-fix/phpstan positional-path examples in the testing skill also remain inaccurate.
- Final aggregate evidence: var/reports/test-pruning-summary.md and test-pruning-tui-handoff.md in task worktree. Net28 cases/0.54%; net1390 lines. No exhaustive-method-audit claim, independent review pending, no commit or PR.

## Task workflow update - 2026-09-20T03:23:25+00:00
- Summary: User explicitly drops overheight /hotkeys rendering coverage. Do not restore that journey phase or add a replacement physical-writer test.
- Scope clarification approved by user: overheight /hotkeys rendering coverage is not required for this cleanup. Its deletion is intentional; no replacement, follow-up task, or coverage blocker is needed for that contract. Existing unrelated hotkey routing/rendering tests remain unchanged.
- This clarification does not waive the separate shell-command-to-normal-follow-up contract. A focused controller-replay replacement was proposed but has not yet been implemented.

## Task workflow update - 2026-09-20T20:22:52+00:00
- Ownership: owner=fork; fork_run=agent_b450d7da7b3c395f; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=one focused controller-replay replacement for normally completed shell commands then normal follow-up, no hotkeys replacement; outcome=assigned; commit=none
- User approved continuing after explicitly dropping overheight hotkeys coverage. Preserve actual controller contract via lower-cost focused replay if deterministic under concurrent lanes; do not revive tmux journey or expand into runtime rewrite.

## Task workflow update - 2026-09-20T20:34:01+00:00
- Validation: castor test:controller-replay PASS:12 tests/202 assertions,39.6s; new case4.030s.; Concurrent normal castor test:controller-replay + castor test:tui PASS with isolated QA lanes/reports: replay12 tests/203 assertions,40.0s; TUI6 tests/31 assertions,15.3s. New case4.134s; all cases below10s.; castor cs-check and git diff --check PASS. castor clean:cleanup:workers:list reported no stale QA workers. Full castor check not run in task-start.
- Summary: Added focused controller-replay replacement for two normally completed direct shell commands followed by a normal prompt on the same controller/session. Overheight hotkeys coverage remains intentionally deleted. Concurrent replay/TUI validation passed; implementation ready for independent review and transition-owned full gate.
- Prior child resume failed because artifact agent_b450d7da7b3c395f belonged to a previous parent lifetime. New fork agent_c21e283e2337e127 took over the bounded replay replacement; no simultaneous writers.
- Ownership: owner=fork; fork_run=agent_c21e283e2337e127; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=focused controller-replay shell-shell-follow-up regression; outcome=completed; commit=none
- Parent read both new files: ControllerReplayShellThenFollowUpTest.php and fixtures/controller-shell-followup.json. Proof covers real shell output, matching command acknowledgements and tool execution pairing, persisted canonical shell lifecycle order, normal follow_up acknowledgement/completion and exactly one fixture-backed LLM turn. Reuses controller E2E facilities; no production changes or sleeps.
- Updated aggregate:5148→5121 net27 cases removed. Unit5098, controller replay12, terminal6, live5 unchanged/not rerun. This restores actual controller contract, not original TUI input/render integration. Overheight physical-writer coverage explicitly waived by user.
- Evidence: task worktree var/reports/test-pruning-shell-followup-handoff.md and updated test-pruning-summary.md. Implementation complete; independent review and full transition gate pending. No commit/PR yet.

## Task workflow update - 2026-09-20T21:03:46+00:00
- Summary: User rejected shallow first pass. Three new read-only scouts audited enum/DTO/getter/framework-self-test/mock-setup categories. Continue with concrete source-verified removals; distinct branches and public API labels alone are not value justification. Some scout references used pre-cleanup state; verify exact task worktree before edits/counts.
- Ownership: owner=fork; fork_run=none; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=second-pass concrete low-value enum/DTO/getter/framework-self-test pruning across audited manifest, preserve meaningful protocol/safety proofs; outcome=assigned; commit=none
- Scout artifacts: agent_feeacb96c2d2700d app; agent_4a135c19350a8983 TUI/extensions; agent_4e7e97acede24b0f core/runtime/tools. Current-file verification mandatory because some manifests refer to already deleted methods.

## Task workflow update - 2026-09-20T21:10:25+00:00
- Validation: castor test PASS:5046 tests,21907 assertions,17.488s; slowest2.045s, none>10s.; Focused changed classes PASS158 tests/517 assertions. Final SessionPickerControllerTest after dead helper removal PASS1 test/1 assertion.; castor dead-code PASS0 errors; castor cs-check PASS; git diff --check PASS. Parent removed two leftover section headings after review, comment-only.; Previously verified concurrent controller replay12 and terminal6 cases unchanged by this pass; live5 unchanged. Full transition gate and independent review still pending.
- Summary: Second scout-driven value pass implemented52 additional case deletions, bringing aggregate net reduction to79 cases (5148→5069,1.53%). Removed concrete enum/framework-self-tests, trivial DTO/property copies, getter/mock-return passthrough tests, duplicate defaults/aliases/lifecycle no-ops. No production changes.
- Ownership: owner=fork; fork_run=agent_b2338286d5bef330; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=second scout-driven low-value test manifest; outcome=completed; commit=none
- Whole files deleted this pass: AgentArtifactKindEnumTest handbuilt Symfony Serializer enum round-trips, RuntimeEventTypeTest enum-format enumeration, CurrentToolCallDTOTest scalar constructor-copy assertions, QuestionRequestTest promoted-property/default assertions, FileRewindConfigTest constructor defaults.
- Per-method deletions: bridge getter/fake exec echoes and PHP null-typed-argument TypeError; config/default scalar catalogs; SafeGuard DTO/default duplicates while retaining actual protected-path policy; mutable display-state property assertion; redundant hotkey/slash aliases and empty registries; picker getter/noop/empty-map states; theme getter/default checks. Removed lock elapsed-time assertion while retaining real held-lock/error proof.
- CodeModeArgumentsDTO invalid bounds retained after source verification: generic resolver tests do not cover actual code_mode duration/memory limits. Removed default-field self-test only. Retaining product bounds is not treating native Symfony functionality itself as the tested contract.
- Evidence: worktree var/reports/test-pruning-second-pass-handoff.md plus updated aggregate summary. Current total: unit5046, replay12, terminal6, live5=5069. Parent reviewed key deletion diffs and corrected stale empty section headings. No claim that all remaining5000+ cases are valuable or exhaustively audited.

## Task workflow update - 2026-09-20T21:47:01+00:00
- Summary: User requested systematic successive rounds of 3–4 scouts followed by sequential implementation forks, aiming at meaningful20–30% reduction. Added advisor's fault-detection criteria and required exact deletion proof mapping to tests/AGENTS.md, including exceptions for real contracts and narrow mock claims. New audit covers high-count matrices and overlapping layers, not only simple getters.
- Ownership: owner=main; fork_run=none; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=codify user test-value criteria in tests/AGENTS.md; outcome=completed; commit=none
- Round A audit scopes: AgentCore+Platform; CodingAgent Agent/Auth/Config/Extension/ExtensionApi; TUI excluding Transcript+Screen; CodingAgent Tool. Round B: remaining CodingAgent non-Runtime/non-Tool; Runtime; TUI Transcript+Screen; all extension suites. Scouts read-only, exclusive fork writer after each round.

## Task workflow update - 2026-09-20T22:04:58+00:00
- Ownership: owner=fork; fork_run=none; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=Round A source-verified scout cuts across core/platform/tools/app/TUI nonscreen plus ActivityStateMachine weak-assertion repair; outcome=assigned; commit=none
- Round A artifacts: agent_d151d9786d3ce6f1 core/platform; agent_d64d27be01030a47 app incl continuation; agent_5b4785e3d36e8f52 TUI incl corrected continuation; agent_5f3acb1af66c61e7 tools. Parent challenged Running→Running mapping proof because default$current hides removed mappings. Require correct rewrite not blind removal.
- Scouts reported partial deep coverage, not an exhaustive per-body audit. App31deep/84mechanicallyinspected; TUI18deep/103triaged. Do not equate census with source proof. Fork must reject unsupported equivalence in candidate manifest.

## Task workflow update - 2026-09-20T22:24:29+00:00
- Validation: Round A castor test PASS5003 tests21840 assertions16.684s; slowest2.049s; castor dead-code/cs-check/docs:validate pass. Fullcheck notrun.
- Ownership: owner=fork; fork_run=agent_a830ce0a2eb8bc41; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=Round A verified deletion and weak-proof rewrites; outcome=completed; commit=none
- Ownership: owner=fork; fork_run=agent_a830ce0a2eb8bc41; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=Round B source-verified remaining-app/runtime/TUI-Screen-Transcript/extension findings, anchor proof rewrites; outcome=assigned; commit=none
- Round A:43 further cases removed,5003unit/5026aggregate. Repaired23state-mapping rows and exact model selection assertion. Rejected unsupported missingdefault/fork-noop duplicate claims and YAML factory deadcode claim. All targeted/unit/deadcode/style/docs pass. Round B scouts found few safe duplicates and important file-rewind anchor assertion gaps; prefer rewrite for actual event precedence over deletion.

## Task workflow update - 2026-09-20T22:33:01+00:00
- Validation: Final unit lane castor test PASS4996 tests21837 assertions16.929s; slowest2.048s,none>10s.; Round B focused PASS58 tests290 assertions5.145s.; castor dead-code0 errors; cs-check/docs:validate/gitdiff--check pass. Parent finaledit comments-only.; No fullcastorcheck, commit, review, push, PR, or statustransition. Task remainsIN-PROGRESS.
- Summary: User-requested fault-detection criteria now in tests/AGENTS.md. Completed two rounds of four scoped scout audits and sequential fork implementation. This request removed50 additional cases and repaired23 activity-mapping rows, exact model selection, and6 persisted file-rewind anchors. Aggregate5148→5019(-129,2.51%); desired20–30% not reached. Census/structural scans are not an exhaustive source audit. Do not mark broader audit complete or claim retainedsuite fully justified.
- Ownership: owner=fork; fork_run=agent_a830ce0a2eb8bc41; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=Round B verified deletion and file-rewind anchor proof repair; outcome=completed; commit=none
- Parent reviewed updated policy and key diffs; removed two orphaned docblocks for deleted state-machine methods. Documentation-only cleanup after validated revision.
- Durable evidence test-pruning-round-a.md:\n## Proof map

The table lists every removed method or data row and the exact remaining proof. "Not a requirement" means the removed assertion did not protect observable behavior.

| Removed method or row | Remaining proof or reason |
| --- | --- |
| `CodexContractTest::testItDoesNotContainMessagesKey` | `CodexContractTest::testCreateRequestPayload` row `user message only` compares the full payload, including `input` and the absence of `messages`. |
| `CodexContractTest::testUserContentIsTypedInputText` | The same `user message only` row compares the nested `role`, `type`, and text. |
| `CodexOAuthServiceTest::testConstructWithStorage` | `testRefreshCredentialsWithStoredExpiredDataAttemptsRefresh` constructs the service with the refresher and exercises it. |
| `CodexOAuthServiceTest::testConstructAcceptsNullRefresher` | `testRefreshCredentialsThrowsWhenRefresherNotConfigured` exercises the supported null-refresher path. |
| `GrokOAuthServiceTest::testConstructWithStorage` | `testRefreshCredentialsPersistsFreshRecord` constructs the service with a refresher and exercises storage. |
| `GrokOAuthServiceTest::testConstructAcceptsNullRefresher` | `testRefreshCredentialsThrowsWhenRefresherNotConfigured` exercises the null-refresher path. |
| `SubagentToolTest::testProviderIsAutoRegistered` | `testDefinitionHasCorrectNameAndParallelSchema` fetches `SubagentToolDefinitionProvider` from the real container and checks its product definition. The removed `instanceof` only repeated its declared interface. |
| `SafeGuardToolCallHookTest::testBashDestructiveStillRequiresApprovalWhenChannelSet` | `testBashDestructiveRequiresApprovalWithAllowDenyOnly` checks the same approval decision plus the question ID and exact choices. |
| `SafeGuardToolCallHookTest::testOnApprovalAnsweredIsNoOp` | Not a requirement. The method only called a no-op and incremented PHPUnit's assertion count. |
| `SafeGuardClassifierTest::testAllowDecisionIsAllowed` | `SafeGuardToolCallHookTest::testBashSafeCommandIsAllowed` consumes an allow decision through the policy hook. |
| `SafeGuardClassifierTest::testBlockDecisionIsNotAllowed` | `SafeGuardToolCallHookTest::testAutoDenyBlocksWhenNoApprovalChannel` consumes a block decision through the policy hook. |
| `PromptsConfigTest::testDefaultIsEmpty` | `testFromRawWithEmptyListIsValid` checks the supported empty configuration input. `testFromAppConfig` remains because `config/services.yaml` uses that factory. |
| `AppConfigLoaderTest::testOverlayConfigScalarOverride` | `testOverlayConfigScalarWinsNotArray` checks the same scalar replacement and its resulting type. |
| `AppConfigLoaderTest::testOverlayConfigBoolOverride` and `testOverlayConfigIntOverride` | `testOverlayConfigMixedTypeOverride` checks the type-agnostic replacement behavior. No type-specific loader rule exists. |
| `AgentDefinitionParserTest::testToolsCommaSeparatedStringWithoutSpaces` | `testToolsCommaSeparatedStringIsNormalized` checks the same comma-splitting branch. Whitespace rejection has separate entry tests. |
| `AgentDefinitionParserTest::testSkillsStringScalarIsNormalized` | `testSkillsStringIsNormalized` uses the same scalar input and expected one-element list. |
| `AgentDefinitionParserTest::testSerializerRejectsUnknownTopLevelField` | `testUnknownFieldThrowsWithFieldNameAndFilePath` checks rejection and the source-path diagnostic. |
| `ExtensionManagerTest::testLoadExtensionsEmptyListLoadsNothing` | `testLoadExtensionsWithEmptyListDoesNothing` uses the same empty configuration and checks zero registrations. |
| `CodexOAuthConfigTest::invalidProfileProvider` row `contains whitespace` | Row `contains space` exercises the same invalid-profile pattern. |
| `SessionSwitchServiceTest::testHasPendingSwitchIsFalseInitially` | `testConsumePendingSwitchReturnsNullWhenNothingPending` checks the same empty pending-switch state. |
| `CommandParserTest::testTextWithSurroundingWhitespaceIsTrimmed` | Not a trimming proof. It only checked the command class, which `testNormalTextReturnsNormalPrompt` checks. |
| `CommandParserTest::testSlashSpaceReturnsNormalPrompt` | `testSlashAloneReturnsNormalPrompt` checks the same lone-slash result after input trimming. |
| `CommandParserTest::testOriginalTextPreservesTrimmedInput` | Not an `originalText` proof. It only checked the command class. |
| `QuestionCoordinatorTest::testActionRequiredFalseInitially` and `testActiveStatusIsNullWhenEmpty` | `testFreshCoordinatorStartsWithNoState` checks both initial values. |
| `QuestionCoordinatorTest::testActiveStatusAfterAnswerLastRequest` | `testFifoOrderPreservedWithThreeRequests` checks that the active request becomes null after the queue drains. |
| `FooterStateSegmentProviderTest::testModelNameColoredWithThinkingColorNotAccent` and `testDiamondAndModelNameShareSameColor` | `testThinkingColorReturnsDedicatedTokenForEachLevel` checks the diamond and model color for every reasoning level, including `high`. |
| `ActivityStateMachineTest::testStartingToRunning` | `testRunningTransitions` row `TurnStarted` now starts at `Starting` and expects `Running`. |
| `ActivityStateMachineTest::testCancellingRepeatedCancelEventsStayCancelling` | `testCancellingSticksOnMidTurnDeltas` is the guard proof. Removing the guard changes those rows to `Running`; repeated cancellation would remain `Cancelling` through the ordinary mapping. |
| `ActivityStateMachineTest::testCancellingUnknownEventStaysCancelling` | Not a separate contract. The ordinary default returns the current state for unknown events. |
| `ActivityStateMachineTest::testCancelledAllowsNewTerminalOutcomeOnRunCancelled` | `testCancelledStaysCancelledOnStaleToolExecutionCancelled`, `testCancelledStaysCancelledOnStaleCancellationRequested`, and `testCancelledStaysCancelledOnTransientSeqZeroToolCallStarted` detect removal of the terminal guard. `RunCancelled` does not. |
| `ActivityStateMachineTest::provideDefaultTransitions` row `streaming event (seq=0 passthrough)` | `testRunningTransitions` row `AssistantTextDelta` now starts at `Starting`; removing the explicit mapping fails the row. The unknown row now checks `Starting` to `Starting`. |
| `WriteFileToolTest::testWriteEmptyContentRemainsEmpty` | `testWriteEmptyContent` checks file creation, zero bytes, and exact empty contents. |
| `CodeModeHostBridgeTest::testScriptWallBudgetHonorsParentTimeoutCeiling` | `testRequestedTimeoutIsCappedBySmallerParentBudget` checks the same one-second parent ceiling. Native Code Mode argument bounds remain tested. |
| `ViewImageToolTest::testRejectsHtmlFile` | `testRejectsTextFile` and `testRejectsPdfFile` check unsupported magic-byte content at the tool boundary. HTML has no separate product rule. |
| `ViewImageToolTest::testDtoRejectsBlankPathWithWhitespace` | `testDtoRejectsBlankPath` checks the same `NotBlank` constraint, whose contract includes whitespace-only strings. |
| `BashToolTest::testSuccessfulCommand` | `testSuccessfulCommandWithNewlines` checks successful foreground execution and exact returned output. |
| `ToolQuestionAnswerResolverTest` rows `string YES`, `string TRUE`, `string NO`, and `string random` | Rows `string Yes` and `string No` retain case normalization for both outcomes. Lowercase `yes`, `true`, `no`, and `false` retain accepted tokens. `string unknown` retains unrecognized-string behavior. |

## Rewrites and rejected cuts

`ActivityStateMachineTest::provideRunningTransitions` now passes `Starting` as the current state for all 23 mapped events. Before this change, deleting a production mapping still returned `Running` through `default => $current`, so every row passed. The revised matrix fails when any explicit mapping disappears.

`ModelResolverTest::testFirstAvailableWhenNoDefault` remains. Missing default configuration and an unavailable configured default are different paths. The test now asserts `deepseek/deepseek-v4-pro` instead of only asserting a non-null result. `testUnavailableDefaultFallsBackToFirstAvailableAndWarns` retains the warning contract for the other path.

`ForkSnapshotCompactionBeforeLaunchTest::testStructuralNoOpStillLaunchesViaOrdinaryDeferredPath` remains. It expects both `CompactionServiceInterface::compactMessages()` and `AgentRunnerInterface::start()` once, so it proves that a structural no-op compaction does not abort launch. The ordinary lifecycle test does not enter this compaction result branch.

`PromptsConfig::fromAppConfig()` and `PromptsConfigTest::testFromAppConfig` remain. `config/services.yaml` uses the method as the service factory. An initial deletion caused container compilation to fail and was reverted before successful validation.

`ToolQuestionAnswerResolverTest` retains mixed-case truthy and falsy rows. Code Mode duration and memory bounds also remain because no other test checks those product limits.

The four scouts did not read every test body. Their coverage was deep for 83 AgentCore and Platform files, 31 application files, 18 TUI files, and 36 Tool files. Mechanical triage covered the rest of each Round A scope. This report does not claim that every retained test has passed a full source-value audit.


- Durable evidence test-pruning-round-b.md:\n## Proof map

| Removed method or assertion | Remaining proof or reason |
| --- | --- |
| `PromptTemplateSubstitutorTest::testAtCaseSensitivity` | `testArgumentsAndAtAreEquivalent` checks exact `$@` replacement. `testArgumentCaseSensitivity` checks the only case-sensitive named placeholder. The removed method did not exercise its `$@s` claim. |
| `ProviderContextUsageResolverTest::testIneligibleWhenStartedAndFailedBothExistAfterProviderMeasurement` | `testFailedAutoCompactionDoesNotReopenEligibility` supplied the same completed, started, and failed events with the same sequence ordering and asserted the same null result. |
| `HeadlessControllerLlmWorkerCountResolutionTest::testValidConstructionWithSettingsDefault` | `HeadlessControllerLlmWorkerPoolProcessTest::testControllerProcessStartsConfiguredLlmConsumerPool` proves that a configured count of three starts three real LLM consumers. The removed test only asserted that construction returned its declared class. |
| `HeadlessControllerLlmWorkerCountResolutionTest::testZeroOverrideDoesNotFail` | Same process test owns the settings-backed default path. This was a second constructor-only zero-override case. |
| `HeadlessControllerLlmWorkerCountResolutionTest::testValidConstructionWithInRangeOverride` | No supported production caller supplies a positive constructor override. Repository search found the override only in this test. The assertion did not observe the resolved count, so it did not prove override selection. `testInvalidOverrideFailsClosedAtConstruction` remains as the class's fail-closed boundary proof. |
| `FileRewindAfterTurnCommitHookTest::testRecordsCheckpointWhenToolBatchSharesCommitWithAgentCommandApplied` | `testRecordsCheckpointOnToolBatchCommitWithToolResultEvents` includes `tool_batch_committed` and `agent_command_applied` in the same commit. It now reads the public ledger and asserts `anchor_seq=11`. |
| `ObserverChunkAndToolTest::testNoToolCallLeavesEmptyCollectionValid` | No tool or model operation ran. The test only checked a newly constructed handler's empty collection. `testMultiCallAccumulateAndInvalidCitationDoesNotMutate` checks accepted accumulation and rejected-citation non-mutation through actual handler calls. |
| Third `setWorkingMessage(null)` call and revision assertion in `TuiStartupVirtualRenderTest::testNoopWorkingMessageNullDoesNotInvalidateWidget` | The immediately preceding identical setter call and assertion prove the no-op revision contract. The test case remains discovered. |

## File rewind assertion repair

Six positive `FileRewindAfterTurnCommitHookTest` methods previously asserted only that a checkpoint existed. They now read checkpoints through `FileRewindLedgerStore::readCheckpoints()` and assert the persisted anchor sequence:

- `testRecordsCheckpointWhenToolBatchAndAgentEndShareCommit`: `anchor_seq=2`, proving `agent_end` wins over `tool_batch_committed`.
- `testRecordsCheckpointOnAgentEnd`: `anchor_seq=5`.
- `testRecordsCheckpointOnPostToolToolBatchCommitted`: `anchor_seq=11`.
- `testRecordsCheckpointOnPostToolFinalAssistantLlmStepCompleted`: `anchor_seq=9`.
- `testRecordsCheckpointWhenToolBatchAndFinalAssistantShareCommit`: `anchor_seq=2`, proving `llm_step_completed` wins over `tool_batch_committed`.
- `testRecordsCheckpointOnToolBatchCommitWithToolResultEvents`: `anchor_seq=11`, proving unrelated tool-result and command events do not replace the tool-batch anchor.

The helper also asserts one matching ledger row. Negative pending-effects and disabled cases still assert that no checkpoint exists.

## Audit limits

The four Round B scouts returned a small, source-specific manifest. They did not deeply read every test body in the remaining CodingAgent, Runtime, TUI Transcript and Screen, or extension scopes. This pass implements only the candidates that survived current-source verification. It does not claim that all retained tests have completed a method-by-method value audit.

The observational-memory threshold test's estimator precondition remains unchanged because that case has meaningful dispatch assertions. No production behavior or API changed.

## Task workflow update - 2026-09-20T22:40:29+00:00
- Validation: Warmup castor test:llm-real PASS5tests30assertions5.6s.; castor check qa-20260920-223807-14365-fe805477 returned quality:ok147.6s, all11lanes passed and leak/cache guards passed. test4996cases39.7s; controller-replay12cases48.0s; tui6cases19.5s; llm-real5cases9.2s. Lane durations are elapsed command times; JUnit aggregate ParaTest suite time is not wall time.; JUnit: TuiProviderErrorE2eTest::testProviderRateLimitErrorShowsSanitizedRedBlock11.037331s >10s. Next slowest overall ControllerReplayLlmRequestRetryVisibilityTest7.614481s. Unit max2.036375s.; Fresh fixture var/tmp/tui-e2e-provider-error-fc1674856fc27df8/.hatfield/logs/agent-2026-09-20.log: stdin_eof22:38:26.508113; consumer.shutdown_escalated mcp#0 pid163617 at22:38:31.525475, shared grace5.017362s. Shutdown contention still reproducible in ordinary parallel gate.; castor clean:cleanup:workers:list found no stale QA worker candidates. No processes signalled or cleaned.; No same-revision clean checkout available for immediate matched before/after: integration HEAD4a3e62443f10f0791adc13603f10b2fbd8004c97 differs from task baselinea3a5b620ebde1384f5772f9e14fcac6f7b2ed709. Historical137.1s gate is not a controlled comparison. No speedup established.
- Summary: Explicit user-requested full parallel measurement performed while IN-PROGRESS, overriding normal deferral of castor check to transition. Gate returned green but JUnit duration inspection fails project's hard quality standard: TuiProviderErrorE2eTest11.037331s. Stability goal NOT achieved; existing Messenger shutdown escalation reproduced. No retry-until-green or deletion performed.
- Full parallel gate green is insufficient: per-case duration standard failed. Keep task IN-PROGRESS; do not claim stable, flake-free, final-ready, or allremainingtestsvaluable. Audit remains partial. Current129case reduction is2.51%, well short of user's20–30% ambition.

## Task workflow update - 2026-09-21T00:34:24+00:00
- Validation: Baseline qa-20260921-002623-20172-63f32004 failed unit lane:4562executed then TuiSkillReadCardVirtualRenderTest line152 substringdocs/unrelated wrapped. Otherlanespass: replay54.8s,tui36.4s,llm17.9s; coldphpstan68.4s/deadcode124s. Not fairwallcomparison against warmcandidate147.6s. Leak/cacheguards passed.
- Summary: Proper measurement started with detached baseline checkout at exact task base a3a5b620, same dependency tree. First baseline full gate exposed another real test defect: TuiSkillReadCardVirtualRenderTest assumes removing newlines un-wraps indented absolute path; checkout path length splits docs/unrelated with indentation. Preserve failed baseline evidence; repair assertion in both measured revisions before matched comparison, report calibration explicitly.
- Benchmark checkout only, no task status move: /home/ineersa/projects/agent-core-worktrees/test-pruning-benchmark-before, detached a3a5b620ebde1384f5772f9e14fcac6f7b2ed709. Dependencies copied from candidate. Primary checkout unchanged.
- Ownership: owner=fork; fork_run=agent_a830ce0a2eb8bc41; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=deterministic provider-error proof at lowest correct layer plus worktree-path-dependent skill-card assertion repair; outcome=assigned; commit=none
- Runtime scout confirms worker get() obtains BEGIN IMMEDIATE even empty queues; nativeSQLite5s signaldeferral remainsstrongcandidate rootcause, not forensicproof. No changingruntimegrace, timeout, customtransport, or testonlyproduction API. Preserve providererror fullruntimehandoff plusvirtualpresentation rather than deletingcontract.

## Task workflow update - 2026-09-21T01:05:11+00:00
- Validation: Calibratedbaseline qa-20260921-005708-31837-8caafd5e PASSall11lanes,wall50.96s,5123unit/11replay/9terminal/5live, allcases<10s,guardsPASS. Originalbaselinefailurepreservedqa-20260921-002623-20172-63f32004.; Candidate qa-20260921-005758-36946-758a2c90 PASSall11lanes,wall55.84s,4986unit/13replay/5terminal/5live, allcases<10s,guardsPASS.; Contention qa-20260921-010014-41613-007d6914 PASSall11lanes,wall57.92s plusconcurrentstandaloneTUI5tests25assertionsandlive5tests30assertions, allcases<10s,max5.454235s. Supervisorstatusinitiallyunknownrecoveredfinalsavedstatus0/log; no rerun.; Final castor clean:cleanup:workers:list foundnostaleQAworkers;gitdiff--checkclean.; No speedupclaim or flake-freeguarantee. Normal+contentionruns supportthisrevision; nativeSQLitebusywait remains inferred unresolvedproductiondefect. No raisingtimeouts/weakeningteardown/hiddenexclusions.
- Summary: Finalcandidate passed normal fullgate and fullgate concurrentwithstandaloneTUI/live lanes, allindividualcases<10s. Realprovider429events now renderthroughmountedvirtualUI, original402proofrestored. Fixedcheckout-path-dependentassertion. Threeadditionaldeepboundedscoutaudits led10furthercuts. Aggregate5148→5009(-139,2.70%);20–30%ambitionnotmet, remainingauditnotexhaustive. No demonstratedoverall speedup; baseline50.96s/candidate55.84s wall. ProductionSQLite/shutdown issueunfixed. TaskIN-PROGRESS, no commit/PR.
- Ownership: owner=fork; fork_run=agent_a830ce0a2eb8bc41; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=provider-error demotion using actual runtime events, path-dependent assertion repair, and ten source-verified duplicate cuts; outcome=completed; commit=none
- Ownership: owner=main; fork_run=none; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=matched baseline/candidate gates and concurrent contention validation; outcome=completed; commit=none
- IMPORTANT TIMING CORRECTION: all earlier147.6/137.1s quality:ok figures are SUMMED LANE TIMES, not elapsedwall time. .castor/tasks.php:373 printsarray_sum($timings). Currentexternal /usr/bin/time measurement verifiedwalltime. No productionchanges were made to timer or runtime.
- Baseline detachedbenchmark checkout intentionally retained for repeatcomparisons at /home/ineersa/projects/agent-core-worktrees/test-pruning-benchmark-before. Onlytrackedchange is identicalskillcardassertioncalibration; status/diff inspected. Not a taskstatusworktree transition. No ownedQAworkers remain.
- Evidence test-pruning-measured-gates.md:
# Measured full-gate comparison

## Method

The baseline is a detached checkout of task base `a3a5b620ebde1384f5772f9e14fcac6f7b2ed709` at `/home/ineersa/projects/agent-core-worktrees/test-pruning-benchmark-before`. Dependencies were copied from the candidate. The candidate is the uncommitted cleanup in the task worktree at the same base revision.

The first baseline gate failed. `TuiSkillReadCardVirtualRenderTest` removed newlines but not continuation indentation from a wrapped absolute path. The different checkout path split `docs/unrelated` and exposed that invalid oracle. Both versions now use the same path-unwrapping assertion correction. The failed baseline was retained as `qa-20260921-002623-20172-63f32004`; its cold analysis caches and early-stopped unit lane make it unsuitable for a speed comparison.

After correcting that assertion, baseline and candidate full gates ran sequentially with no other QA jobs launched by this task. Both used the default gate worker counts, the cache guard, and the leak guard. Both rebuilt the PHAR. Neither run changed production SQLite or shutdown settings.

`/usr/bin/time -p` measured elapsed command wall time. The `quality: ok (...s)` text is NOT wall time: `.castor/tasks.php:373` prints `array_sum($timings)`. Earlier reports treating 137–148 seconds as elapsed time were wrong. ParaTest's JUnit suite time is also not lane wall time.

## Matched pair

| Measurement | Calibrated baseline | Candidate |
|---|---:|---:|
| Elapsed gate wall time | 50.96s | 55.84s |
| Unit lane wall time | 39.8s | 38.6s |
| Controller replay lane wall time | 41.4s | 46.2s |
| TUI lane wall time | 23.6s | 14.7s |
| Live lane wall time | 8.9s | 9.1s |
| Unit cases | 5,123 | 4,986 |
| Controller replay cases | 11 | 13 |
| Terminal cases | 9 | 5 |
| Live cases | 5 | 5 |
| Total cases | 5,148 | 5,009 |

Baseline report: `qa-20260921-005708-31837-8caafd5e` in the baseline checkout. Candidate report: `qa-20260921-005758-36946-758a2c90`.

Both gates passed all eleven lanes, cache guards, and leak guards. No case exceeded ten seconds. This one matched pair shows no overall speedup: the candidate took 4.88 seconds longer. The terminal lane became shorter, but the two replacement controller proofs run in the sequential replay lane, which remains the critical path. Do not present the lower sum of lane times as reduced elapsed time.

## Contention validation

Ran these three commands concurrently, with separate reports and normal safeguards:

- `castor check`
- `castor test:tui`
- `castor test:llm-real`

Gate report: `qa-20260921-010014-41613-007d6914`. Gate elapsed wall time was 57.92 seconds. All eleven lanes and both guards passed. The shell supervisor initially lost the foreground status, so completion was verified from its saved final log and status file, which contained exit code zero. The workload was not rerun.

Standalone TUI: five cases, 25 assertions, runner time 15.7 seconds, command wall time 24.58 seconds. Standalone live: five cases, 30 assertions, runner time 4.7 seconds, command wall time 5.88 seconds. Both exited zero.

JUnit inspection across the gate and both standalone lanes found no case over ten seconds. The slowest was `ControllerReplayCancelDuringBashThenFollowUpTest::testCancelDuringActiveBashThenFollowUpWithoutRestart`, at 5.454235 seconds. The slowest standalone TUI case took 4.955254 seconds. A final `castor clean:cleanup:workers:list` found no stale QA worker candidates.

These runs establish passing normal and contention validation for this revision, not a guarantee that no future flake exists.

## Changes and limits

The over-budget provider-error tmux case was replaced by `ControllerReplayProviderErrorTest::testRateLimitExhaustionRendersActualRuntimeFailureEvent`. It runs real controller and Messenger processes, captures actual HTTP 429 failure events, projects those events, and verifies the mounted virtual TUI's error color, retry-exhaustion text, HTTP status, and provider detail. The pre-existing HTTP 402 mounted-transcript proof remains unchanged. The real-TTY input-to-provider-error journey is no longer covered; packaged terminal boot remains covered separately.

The production shutdown contention problem remains unresolved. Matching five-second escalation timings and the idle receiver's `BEGIN IMMEDIATE` support a native SQLite busy-wait explanation, but no blocked worker stack was captured. This cleanup did not fix or weaken production shutdown.

Three further scouts deeply read nineteen bounded test files and their owners, rather than treating structural triage as source review. Ten more duplicate cases were removed with named surviving detectors. One representative unknown-projector-event row was retained, rejecting the proposed deletion of all unknown-event coverage. Exact maps are in `test-pruning-review-corrections.md` and the external task log.

Aggregate reduction is 139 cases, or 2.70%. The requested 20–30% ambition remains unmet. The rest of the suite has not received an exhaustive method-by-method value audit, and this report does not justify every retained test.

- Evidence test-pruning-review-corrections.md:
# Review corrections and final bounded cuts

The existing HTTP 402 retry-executor-to-widget test was restored unchanged. The new provider 429 proof now captures runtime events from the real controller and Messenger pipeline, feeds those exact events to `TranscriptProjector`, and renders the resulting blocks with `VirtualTuiHarness`. It checks the red `✕ Run failed` header, HTTP 429, retry exhaustion, and the provider sentinel in one case.

The deleted tmux journey's unique loss remains real TTY input-to-provider-error presentation. Packaged terminal boot remains owned by `TuiArtifactBootE2eTest`.

The 11.037331-second tmux case and logs are strong evidence that `mcp#0` remained in the native SQLite busy-wait interval during shutdown. They do not provide a captured native stack, so the exact blocked call remains an inference. No production shutdown change was made.

## Deletion proof map

- `AppConfigLoaderTest::testOverlayConfigListReplacesEntirely`: `testOverlayConfigListDoesNotIndexMerge` uses the same base and overlay lists and asserts the exact replacement.
- `RuntimeEventMapperTest::testNormalizesToolExecutionEndStructuredCancellationMetadataToToolExecutionCancelled`: `testNormalizesToolExecutionEndCancelledToToolExecutionCancelled` supplies the same structured cancellation type and message and asserts the same runtime event.
- `RuntimeEventMapperTest::testRunIdAndSeqArePreserved`: `testNormalizesRunStartedToRunStarted` already asserts the mapped run ID and sequence. The dedicated case added no different mapping path.
- The error=true iteration in `testToolTimingUsesCanonicalEventTimestamps`: the error flag does not select timestamp mapping. The retained normal result asserts the canonical end timestamp. Error mapping remains covered elsewhere.
- `TranscriptProjectorTest::noOpEventProvider` row `model.changed`: row `progress.updated` remains as the representative unknown/no-subscriber event. Known assistant no-op rows remain for their distinct ownership guards.
- `ToolRegistryTest::testConstructorRegistersEmptyProviders`: `testEmptyRegistryReturnsEmptyLists` checks all empty registry projections.
- `ToolRegistryTest::testAddDynamicTool`: `testActiveToolNamesCombinesPermanentAndDynamicInOrder` checks dynamic insertion and ordering.
- `ToolRegistryTest::testRemoveDynamicTool`: `testActiveToolNamesDoesNotIncludeRemovedDynamicTools` checks removal while preserving permanent tools.
- `ToolRegistryTest::testIdenticalReRegistrationIsIdempotent`: `testIdenticalPermanentReRegistrationKeepsFirstDefinition` checks identity preservation. `testDedupesDuplicatePromptLinesAcrossTools` and `testDedupesDuplicateGuidelinesAcrossTools` retain metadata deduplication.
- `ExtensionToolRegistryBridgeTest::testGuidelineDeduplication`: `testRegisterToolForwardsToRegistry` checks DTO guideline forwarding, while `ToolRegistryTest::testDedupesDuplicateGuidelinesAcrossTools` owns deduplication.
- `ExtensionToolRegistryBridgeTest::testEmptyDescriptionThrowsInvalidArgumentException`: `testEmptyNameThrowsInvalidArgumentException` retains bridge error propagation for the registry's shared non-empty name/description guard. `ToolRegistryTest::testRegisterPermanentToolWithEmptyDescriptionThrows` retains the description boundary itself.

`TranscriptProjectorTest` keeps one representative unknown event. Both unknown rows were not deleted.

## Counts and validation

- Unit lane: 4,986 tests and 21,820 assertions passed in 17.156 seconds.
- Controller replay: 13 tests and 218 assertions passed in 42.641 seconds.
- Focused changed unit set: 263 tests and 1,000 assertions passed.
- Controller provider-error case remained below 10 seconds.
- Full `castor check` was not run. Parent owns the gate.

## Task workflow update - 2026-09-21T01:12:34+00:00
- Summary: User requested closing the current cleanup scope and moving to CODE-REVIEW. Freeze this submission at the measured 139-case reduction, 2.70%; do not expand pruning to meet the original title's 30% quota. Prior clarification made that target aspirational. Preserve the acknowledged coverage losses, lack of overall speedup, and unresolved production shutdown contention in the PR. Independent review and transition-owned gate remain required.

## Task workflow update - 2026-09-21T01:31:35+00:00
- Summary: Independent reviewer agent_9c75a589a66d65b1 requested removal of the unused response_delay_ms fixture path and correction of stale retained-proof mappings. No PR exists; task remains IN-PROGRESS. Main owns this small review correction. Other reviewed code approved in principle; coverage losses and performance limits will be explicit in PR.
- Review: role=reviewer; artifact=agent_9c75a589a66d65b1; target=a3a5b620 plus 80 tracked and 3 untracked changes; scope=whole cleanup diff and specification fidelity; verdict=REQUEST CHANGES.
- Ownership: owner=main; fork_run=none; revision=a3a5b620ebde1384f5772f9e14fcac6f7b2ed709; scope=remove dead replay response delay and correct evidence mappings; outcome=assigned; commit=none

## Task workflow update - 2026-09-21T01:35:56+00:00
- Validation: After review correction: factory tests 3 cases/13 assertions PASS; controller replay 13 cases/218 assertions PASS in 42.105s; dead-code zero errors; git diff --check clean; committed worktree clean.; Earlier final-scope normal and concurrent gates passed, max individual case 5.454235s. Only subsequent code change removes an unused replay-delay block; transition will validate submitted commit.
- Summary: Reviewer agent_9c75a589a66d65b1 re-reviewed and APPROVED the final diff, including specification fidelity. Committed unchanged reviewed code as fea35db70d4b835817cf89655f4614b7d1efb023. No unresolved blockers. Ready for transition-owned full gate.
- Ownership: owner=main; fork_run=none; revision=fea35db70d4b835817cf89655f4614b7d1efb023; scope=review fixes and submission; outcome=completed; commit=fea35db70d4b835817cf89655f4614b7d1efb023
- Review: role=reviewer; artifact=agent_9c75a589a66d65b1; target=fea35db70d4b835817cf89655f4614b7d1efb023; scope=whole cleanup plus blocker correction and specification fidelity; verdict=APPROVE; blockers=none.
- Evidence correction supersedes historical logs: response_delay_ms had no fixture callers, so its unused implementation was deleted. AgentsConfig explicit max_agents hydration is covered by AppConfigTest::testTargetSectionsHydrateValidValues; default four by RuntimeConfigLlmWorkerCountTest::testDefaultLlmWorkerCountIsFourAndDistinctFromToolAndAgentLimits. Earlier named testDefaultMaxAgentsIsFour was deleted later.
- Accepted limitations to include in PR: loss of dedicated config-default drift checks for compaction thresholds/enabling, agent timeout/excluded-tool defaults, transcript defaults, file-rewind configuration, and HTTP null defaults; overheight/hotkeys terminal-writer proof intentionally dropped; realTTY provider-error presentation demoted. Skills still reference deleted journey examples; reported only, no skill-repair authorization.

## Task workflow update - 2026-09-21T01:37:19+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (55.9s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs/var/reports/qa-20260921-013624-51173-85ccb84a.
- Session/run: 60.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-21T01:37:21+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs/var/reports/qa-20260921-013624-51173-85ccb84a.
- Session/run: 60.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-21T01:37:23+00:00
- castor check passed (55.9s).
- Pushed task/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs to origin.
- Created PR: <url>
- Session/run: 60.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-21T01:37:23+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (55.9s).
- Pushed task/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/518
- Validation: Factory 3 tests/13 assertions PASS; replay 13 tests/218 assertions PASS; dead-code zero errors; clean committed worktree.; Transition owns mandatory full castor check for fea35db70.
- Summary: Submit user-approved bounded cleanup after independent APPROVE; no unresolved review blockers. Preserve reduction, coverage-loss, and performance limitations in PR.

## Task workflow update - 2026-09-21T01:37:48+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/518
- Updated PR Status: open
- Validation: Transition-owned full gate passed for fea35db70 in 55.9s. JUnit qa-20260921-013624-51173-85ccb84a: 5009 cases; slowest 4.597217s; no case over 10s.

## Task workflow update - 2026-09-21T01:54:46+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/extensions/file-rewind/tests/FileRewindAfterTurnCommitHookTest.php              |  46 ++++----
 .hatfield/extensions/file-rewind/tests/FileRewindConfigTest.php                           |  18 ---
 .hatfield/extensions/observational-memory/tests/ObserverChunkAndToolTest.php              |  10 --
 tests/AGENTS.md                                                                           |  51 +++++++--
 tests/AgentCore/Application/Handler/RunLockManagerTest.php                                |   5 -
 tests/AgentCore/Domain/Run/CurrentToolCallDTOTest.php                                     |  33 ------
 tests/CodingAgent/Agent/Artifact/AgentArtifactKindEnumTest.php                            | 112 ------------------
 tests/CodingAgent/Agent/Artifact/AgentArtifactRegistryTest.php                            |  14 ---
 tests/CodingAgent/Agent/Definition/AgentDefinitionParserTest.php                          |  38 -------
 tests/CodingAgent/Agent/Tool/SubagentToolTest.php                                         |  10 --
 tests/CodingAgent/Auth/CodexOAuthConfigTest.php                                           |   1 -
 tests/CodingAgent/Auth/CodexOAuthServiceTest.php                                          |  12 --
 tests/CodingAgent/Auth/GrokOAuthServiceTest.php                                           |  12 --
 tests/CodingAgent/Compaction/AutoCompactionHookSubscriberTest.php                         |  10 --
 tests/CodingAgent/Compaction/CodingAgentPreLlmCompactionGuardTest.php                     |   6 -
 tests/CodingAgent/Compaction/ProviderContextUsageResolverTest.php                         |  27 -----
 tests/CodingAgent/Config/AgentsConfigTest.php                                             |  50 --------
 tests/CodingAgent/Config/Ai/AiHttpConfigTest.php                                          |  24 ----
 tests/CodingAgent/Config/AppConfigLoaderTest.php                                          |  42 -------
 tests/CodingAgent/Config/AppConfigTest.php                                                |  47 --------
 tests/CodingAgent/Config/CompactionConfigTest.php                                         |  16 ---
 tests/CodingAgent/Config/ModelResolverTest.php                                            |   1 +
 tests/CodingAgent/Config/PromptsConfigTest.php                                            |   6 -
 tests/CodingAgent/Extension/Builtin/SafeGuard/Classifier/SafeGuardClassifierTest.php      |  15 ---
 tests/CodingAgent/Extension/Builtin/SafeGuard/Policy/SafeGuardPolicyTest.php              |  25 ----
 tests/CodingAgent/Extension/Builtin/SafeGuard/SafeGuardConfigTest.php                     |  22 ----
 tests/CodingAgent/Extension/Builtin/SafeGuard/SafeGuardExtensionTest.php                  |   6 -
 tests/CodingAgent/Extension/Builtin/SafeGuard/SafeGuardToolCallHookTest.php               |  21 ----
 tests/CodingAgent/Extension/ExtensionManagerTest.php                                      |  15 ---
 tests/CodingAgent/Extension/ExtensionToolRegistryBridgeTest.php                           | 101 -----------------
 tests/CodingAgent/PromptTemplate/PromptTemplateSubstitutorTest.php                        |   8 --
 tests/CodingAgent/Runtime/Contract/RuntimeExceptionBoundaryTest.php                       |  10 --
 tests/CodingAgent/Runtime/Controller/E2E/ControllerReplayShellThenFollowUpTest.php        | 206 +++++++++++++++++++++++++++++++++
 tests/CodingAgent/Runtime/Controller/E2E/Replay/ControllerReplayHttpClientFactory.php     |  46 +-------
 tests/CodingAgent/Runtime/Controller/E2E/Replay/ControllerReplayHttpClientFactoryTest.php |  55 ---------
 tests/CodingAgent/Runtime/Controller/E2E/Replay/StreamPacingHttpClient.php                |  63 -----------
 tests/CodingAgent/Runtime/Controller/E2E/fixtures/controller-shell-followup.json          |  15 +++
 tests/CodingAgent/Runtime/Controller/HeadlessControllerLlmWorkerCountResolutionTest.php   |  22 +---
 tests/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClientShutdownTest.php          |  16 +--
 tests/CodingAgent/Runtime/Projection/TranscriptProjectorTest.php                          |  21 ----
 tests/CodingAgent/Runtime/RuntimeEventMapperTest.php                                      |  33 +-----
 tests/CodingAgent/Runtime/RuntimeEventTypeTest.php                                        | 145 ------------------------
 tests/CodingAgent/Tool/BashToolTest.php                                                   | 122 +++++++++++---------
 tests/CodingAgent/Tool/CodeMode/CodeModeArgumentsDTOTest.php                              |   9 --
 tests/CodingAgent/Tool/CodeMode/CodeModeHostBridgeTest.php                                |  33 ------
 tests/CodingAgent/Tool/OutputCapTest.php                                                  |   5 +-
 tests/CodingAgent/Tool/RegistryBackedToolboxTest.php                                      |  11 --
 tests/CodingAgent/Tool/ToolQuestion/ToolQuestionAnswerResolverTest.php                    |   4 -
 tests/CodingAgent/Tool/ToolRegistryTest.php                                               |  34 ------
 tests/CodingAgent/Tool/ViewImageToolTest.php                                              |  21 ----
 tests/CodingAgent/Tool/WriteFileToolTest.php                                              |   9 --
 tests/Platform/Bridge/Generic/DurableResultConverterTest.php                              |  15 ---
 tests/Platform/Bridge/OpenAICodex/CodexContractTest.php                                   |  27 -----
 tests/Platform/Bridge/OpenAICodex/RawWebSocketResultTest.php                              |  22 ++--
 tests/Tui/Application/SessionSwitchServiceTest.php                                        |   7 --
 tests/Tui/Application/TranscriptDisplayConfigMapperTest.php                               |  10 --
 tests/Tui/Command/CommandParserTest.php                                                   |  22 ----
 tests/Tui/Command/Hotkey/HotkeyRegistryTest.php                                           |  37 ------
 tests/Tui/Command/SlashCommandRegistryTest.php                                            |  80 -------------
 tests/Tui/Completion/CompletionProviderRegistryTest.php                                   |   9 --
 tests/Tui/E2E/ControllerReplayProviderErrorTest.php                                       | 121 ++++++++++++++++++++
 tests/Tui/E2E/TmuxHarness.php                                                             |  22 ----
 tests/Tui/E2E/TuiJourneyE2eTest.php                                                       | 453 -------------------------------------------------------------------------
 tests/Tui/E2E/TuiProviderErrorE2eTest.php                                                 | 180 -----------------------------
 tests/Tui/E2E/TuiStartupSnapshotTest.php                                                  | 194 -------------------------------
 tests/Tui/E2E/TuiSubagentProgressE2eTest.php                                              | 237 --------------------------------------
 tests/Tui/E2E/fixtures/tui-followup-response.json                                         |  33 ------
 tests/Tui/E2E/fixtures/tui-provider-rate-limit-error.json                                 |  10 --
 tests/Tui/Listener/FooterStateSegmentProviderTest.php                                     |  37 ------
 tests/Tui/Picker/PickerOverlayTest.php                                                    | 129 ---------------------
 tests/Tui/Picker/SessionPickerControllerTest.php                                          |  79 -------------
 tests/Tui/Question/QuestionCoordinatorTest.php                                            |  25 ----
 tests/Tui/Question/QuestionRequestTest.php                                                |  67 -----------
 tests/Tui/Runtime/ActivityStateMachineTest.php                                            |  51 +--------
 tests/Tui/Screen/TuiMountedTranscriptVirtualTest.php                                      |  46 ++++++++
 tests/Tui/Screen/TuiSkillReadCardVirtualRenderTest.php                                    |   5 +-
 tests/Tui/Screen/TuiStartupVirtualRenderTest.php                                          |   3 -
 tests/Tui/Screen/TuiTranscriptBlocksVirtualRenderTest.php                                 |   2 -
 tests/Tui/Support/ChildContextStatisticsFixture.php                                       |  24 ----
 tests/Tui/Theme/DefaultThemeTest.php                                                      |  59 ----------
 tests/Tui/Theme/ThemePaletteTest.php                                                      |  36 ------
 tests/Tui/Theme/ThemeRegistryTest.php                                                     |  18 ---
 tools/phpstan/DeadCode/HatfieldDeadCodeUsageProvider.php                                  |   3 +-
 83 files changed, 547 insertions(+), 3200 deletions(-)
 delete mode 100644 .hatfield/extensions/file-rewind/tests/FileRewindConfigTest.php
 delete mode 100644 tests/AgentCore/Domain/Run/CurrentToolCallDTOTest.php
 delete mode 100644 tests/CodingAgent/Agent/Artifact/AgentArtifactKindEnumTest.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/E2E/ControllerReplayShellThenFollowUpTest.php
 delete mode 100644 tests/CodingAgent/Runtime/Controller/E2E/Replay/StreamPacingHttpClient.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/E2E/fixtures/controller-shell-followup.json
 delete mode 100644 tests/CodingAgent/Runtime/RuntimeEventTypeTest.php
 create mode 100644 tests/Tui/E2E/ControllerReplayProviderErrorTest.php
 delete mode 100644 tests/Tui/E2E/TuiJourneyE2eTest.php
 delete mode 100644 tests/Tui/E2E/TuiProviderErrorE2eTest.php
 delete mode 100644 tests/Tui/E2E/TuiStartupSnapshotTest.php
 delete mode 100644 tests/Tui/E2E/TuiSubagentProgressE2eTest.php
 delete mode 100644 tests/Tui/E2E/fixtures/tui-followup-response.json
 delete mode 100644 tests/Tui/E2E/fixtures/tui-provider-rate-limit-error.json
 delete mode 100644 tests/Tui/Question/QuestionRequestTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-20-prune-low-value-tests-and-replace-timing-dependent-proofs.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR #518 merged at 2026-09-21T01:53:49Z, merge commit 038659dda6755ffe02ccc981f4d095850d078715. Integrating completed cleanup; post-merge validation follows.

## Task workflow update - 2026-09-21T01:57:45+00:00
- Updated PR Status: merged
- Validation: Post-merge castor check: all 11 lanes PASS, wall time 116.76s. Dead-code lane took 107.0s; printed quality duration 238.0s is summed lane time, not wall time.; JUnit qa-20260921-015505-56225-60cb8b3f: 5010 integrated cases, maximum 5.368609s, no case over 10s. Unit 4987/21833 assertions; replay 13/218; terminal 5/25; live 5/30.; QA process/tmux leak guard and cache guard passed. Lost initial shell supervision recovered through saved final log and status=0; no rerun.; Verified clean main checkout and task worktree absent from filesystem and git worktree list.
- Summary: PR #518 merged and task completed. Post-merge integration validation passed at 9c93ea20ffa002168828d47634d118707a00327d. Integration Git status clean; task worktree removed.
