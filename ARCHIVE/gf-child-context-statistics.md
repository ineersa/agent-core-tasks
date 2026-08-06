# GF-02: Show child-agent context statistics

## Goal
Extract child/subagent context usage display from the abandoned fork MVP branch into a fresh task based on current origin/main. This is generic existing-subagent UX and must not depend on the fork tool.

Desired behavior:
- Propagate the child's model and latest input-token/context-window usage through canonical subagent progress data.
- Show compact model/context usage in the child transcript card, `/agents-live` picker row, and child live-view footer.
- Reuse the same context thresholds/colors and token formatting as the main footer.
- Avoid duplicating formatter logic across footer, cards, and picker.

Historical references only—do not cherry-pick tests or commits wholesale:
- `a4f003b8e` — child context statistics.
- `0a6cba1c4` — historical test stabilization.

Likely production surface includes `SubagentChildProgressSummary`, `SubagentChildProgressSummaryBuilder`, `ContextUsageFormatter`, `SubagentLiveChildDTO`, `SubagentLiveCatalog`, `SubagentResultRenderer`, `SubagentTranscriptCardBuilder`, footer/listener/screen wiring, and picker formatting. Strip fork-only nested catalog ingestion if consulting historical diffs.

Keep this task independent from transcript virtualization/performance optimization; it only specifies accurate context data and rendering.

## Acceptance criteria
- First commit/push contains only reviewed RED specification tests; no production implementation. Accepted tests are immutable during implementation.
- Red contract test proves a real subagent progress payload carries model and latest input-token usage from the latest child LLM step.
- Red virtual TUI tests prove child card, picker row, and live-view footer display the same context percentage/model for one canonical child state.
- Formatting and warning thresholds match the main footer's existing context usage behavior.
- Missing model/context-window/token data degrades cleanly without fabricated percentages.
- No fork tool, nested child polling, or fork-specific catalog method is introduced.
- No whole-transcript performance optimization is bundled into this task.
- All QA uses Castor; no historical branch tests are copied.

## Workflow metadata
Status: ARCHIVE
Branch: task/gf-child-context-statistics
Worktree: /home/ineersa/projects/agent-core-worktrees/gf-child-context-statistics
Fork run: 1a0l57q1bppy
PR URL: https://github.com/ineersa/agent-core/pull/281
PR Status: merged
Started: 2026-07-11T17:56:45.820Z
Completed: 2026-07-11T20:41:38.027Z

## Work log
- Created: 2026-07-11T17:43:23.429Z

## Task workflow update - 2026-07-11T17:56:45.820Z
- Moved TODO → IN-PROGRESS.
- Created branch task/gf-child-context-statistics.
- Created worktree /home/ineersa/projects/agent-core-worktrees/gf-child-context-statistics.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/gf-child-context-statistics.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/gf-child-context-statistics.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/gf-child-context-statistics.
- Summary: Claimed for task-start implementation. Acceptance requires an immutable RED specification-test commit before production implementation.

## Task workflow update - 2026-07-11T17:59:44.745Z
- Summary: Scouting completed. Current flow is child LlmStepCompleted usage → SubagentChildProgressSummary/Builder → progress payload → SubagentLiveCatalog/DTO → transcript card, /agents-live picker, and child live-view footer. A shared TUI ContextUsageFormatter is absent and should centralize model context-window resolution, token formatting, and 50%/75% Success/Warning/Error thresholds. Historical commits are reference-only; exclude fork/nested-child catalog ingestion and transcript performance work. Required implementation sequence remains immutable tests-only RED commit first, production commit second.
- Both scouts confirmed they read .agents/skills/testing/SKILL.md and tests/AGENTS.md before test-related exploration.
- Test thesis: one canonical child state must carry latest (not cumulative) LLM-step input tokens and model, then render the same percentage/detail across card, picker, and live footer; missing data must omit percentages. Required layers: contract/unit + virtual TUI, plus user-requested minimal replay-backed TmuxHarness proof in existing subagent live-view E2E path.

## Task workflow update - 2026-07-11T18:00:48.444Z
- Recorded fork run: ap1x8ehpseil
- Summary: Implementation fork launched in task worktree with strict two-commit sequence: immutable RED tests/fixtures-only commit first, production-only commit second. Fork was instructed to add contract + virtual TUI proof and a real replay-backed TmuxHarness E2E feature-path proof, use Castor only, avoid full castor check/push/PR, and exclude fork-specific/nested/performance scope.

## Task workflow update - 2026-07-11T18:09:35.368Z
- Recorded fork run: ap1x8ehpseil
- Validation: PASS: castor deptrac (reported by fork); PASS: castor phpstan (reported by fork); PASS: castor cs-check after castor cs-fix (reported by fork); FAIL: castor test:tui --filter=TuiSubagentLiveViewE2eTest — missing parent transcript CTX line; picker/live footer visible context passed; FAIL: castor test:tui --filter=TuiSubagentProgressE2eTest — missing parent transcript CTX line; INVALID EVIDENCE: fork ran raw vendor/bin/phpunit despite Castor-only requirement; BLOCKED: picker test file has duplicate RuntimeEvent/RuntimeEventTypeEnum imports and cannot load
- Summary: Fork output rejected as incomplete. Commits exist (`e0e14778c` RED tests-only; `e488638c8` production-only) and worktree is clean, but the immutable RED commit contains duplicate imports causing a PHP fatal, required Tmux transcript-card CTX assertions fail, and some validation used forbidden raw vendor/bin PHPUnit commands. Production also introduced optional AppConfig explicitly to preserve old test constructors, conflicting with no compatibility/test-only production API rules. Task remains IN-PROGRESS.
- Parent verified worktree branch is clean with exactly two task commits and `git diff --name-only e0e14778c..HEAD -- tests` is empty.
- Correcting the malformed immutable RED commit while preserving required history order needs explicit approval for history reconstruction/rewrite; project rules prohibit destructive/history-rewriting git operations without user approval.

## Task workflow update - 2026-07-11T18:11:49.319Z
- Summary: User explicitly approved discarding unpublished commits `e0e14778c` and `e488638c8` and restarting from `origin/main`. Recovery will reset only the task branch/worktree, remove the GF-specific stash, create a fresh RED tests-only commit, validate that tests load and fail for intended missing behavior via Castor, and STOP before production for review/acceptance.

## Task workflow update - 2026-07-11T18:20:31.098Z
- Recorded fork run: h258ja830eix
- Validation: Verified: worktree clean; one commit over origin/main; 9 changed paths all under tests/**; git diff --check clean; Reported intended RED Castor failures load and execute without parse/fatal errors; Independent RED review: REJECT — one green characterization test, missing graceful-degradation specification, and incomplete E2E provider fixture
- Summary: Fresh RED-only commit `64b5d90d0` was created and parent verified clean tests-only scope, but independent scout review REJECTED it before acceptance/implementation. No production commit exists. Required corrections: replace the green main-footer characterization with RED child-surface threshold assertions, add missing-data degradation assertions within a RED behavior test, and add deepseek provider settings to `TuiSubagentProgressE2eTest` so the expected 272k context can resolve.

## Task workflow update - 2026-07-11T18:31:36.544Z
- Recorded fork run: co2y37s9bbrq
- Summary: User approved amending unaccepted unpublished RED commit `64b5d90d0`. Correction fork launched to make threshold proof RED on child surfaces, add graceful missing-data specification, repair progress E2E model settings, validate via Castor only, and leave one tests-only commit with no production implementation.

## Task workflow update - 2026-07-11T18:40:51.565Z
- Validation: Verified: `85c7e40c4` is a clean single tests-only commit; 9 tests/** paths; diff check clean; Independent general RED review initially approved assertions and prior fixes; Focused DI/data-flow feasibility audit rejected: no legitimate current seam supplies 272000 to AppConfig-free virtual surfaces
- Summary: Amended RED commit `85c7e40c4` is still not accepted. Parent verification found an architectural feasibility gap missed by the first review: unit/virtual tests expect 272k context resolution while supplying only model/latest tokens to AppConfig-free surfaces (`FooterStateSegmentProvider($state)`, static picker buildItems, transcript renderer). A second focused architecture audit REJECTED hardcoded/ambient/optional-dependency solutions and recommends making `context_window` canonical progress data, so TUI consumers receive model + latest_input_tokens + context_window and share a pure formatter. No production implementation has started.

## Task workflow update - 2026-07-11T18:56:06.569Z
- Recorded fork run: 3opwrrukif9t
- Summary: User approved a second RED amend and pointed to the final fork-mvp implementation as reference. Fork launched to inspect that generic context_window flow, amend tests so context_window is canonical payload data, strengthen contract/catalog/missing-data assertions, use Castor only, and stop before production.

## Task workflow update - 2026-07-11T18:58:14.092Z
- Recorded fork run: 3opwrrukif9t
- Summary: Second amended RED commit `63c1e7310` correctly makes `context_window` canonical for TUI payload/DTO consumers and confirms fork-mvp only resolved it via AppConfig. However parent review found one remaining impossible contract setup: `SubagentChildProgressSummaryBuilderTest` constructs the builder with only its event-store factory and supplies only model metadata, yet expects the builder to resolve 272k from a catalog it cannot access. Before acceptance, the child RunStarted test event must carry canonical `context_window` metadata and the assertion must specify propagation, not catalog resolution. No production exists.

## Task workflow update - 2026-07-11T19:01:35.212Z
- Recorded fork run: rjurtwz6quwr
- Summary: User approved final RED amend. Narrow fork launched to put context_window in child RunStarted metadata, specify propagation rather than impossible catalog resolution, validate via Castor, and stop before production.

## Task workflow update - 2026-07-11T19:05:14.336Z
- Summary: RED specification commit `b2f5d2a9c` accepted as immutable after final independent architecture review. Parent verified clean worktree, one tests-only commit over origin/main, 9 tests/** paths, no non-test changes, and diff check clean. Approved canonical flow: child RunStarted metadata model+context_window plus latest LLM-step tokens → summary progress fields → catalog DTO → pure shared formatter → card/picker/live footer. Production implementation may now begin without any test/fixture edits.

## Task workflow update - 2026-07-11T19:05:55.275Z
- Recorded fork run: 04l257xpjaob
- Summary: Production-only fork launched on accepted immutable RED `b2f5d2a9c`. Instructions require canonical RunStarted model/context_window population, latest-step token propagation, DTO/catalog preservation, one pure shared formatter reused by main/child surfaces, real Tmux proof, Castor-only focused validation, no test edits, no castor check/push/PR.

## Task workflow update - 2026-07-11T19:14:12.053Z
- Recorded fork run: 04l257xpjaob
- Summary: Production fork ended without handoff or commit. Parent inspected worktree: 12 production paths are uncommitted (10 modified, 2 new formatter/DTO files), no tests changed, HEAD remains accepted RED `b2f5d2a9c`. Last fork note mentioned a Tmux wait collision where live footer text `scout [completed]` prematurely matched parent-restore proof. Work is incomplete and requires continuation fork; no validation accepted yet.

## Task workflow update - 2026-07-11T19:14:35.220Z
- Recorded fork run: 5e4bsizlmfs8
- Summary: Continuation fork launched in dirty task worktree to inspect/complete interrupted production changes, resolve Tmux footer wait collision in production without changing immutable tests, run Castor-only focused/full validation, and commit one production-only commit.

## Task workflow update - 2026-07-11T19:15:00.100Z
- Recorded fork run: 5e4bsizlmfs8
- Summary: Continuation fork failed with no output and made no observable changes. Parent rechecked: worktree remains dirty with the same 12 production paths, HEAD still `b2f5d2a9c`, no tests changed, no production commit. Task remains IN-PROGRESS.

## Task workflow update - 2026-07-11T19:15:21.878Z
- Recorded fork run: oa85ugmvw30q
- Summary: Second rescue fork launched to finish existing production diff, resolve known Tmux collision, validate through Castor, preserve immutable tests, and commit. It was explicitly instructed to report last command/error instead of exiting silently if blocked.

## Task workflow update - 2026-07-11T19:15:31.732Z
- Recorded fork run: oa85ugmvw30q
- Summary: Second rescue fork also failed silently with no output. No result can be accepted. Parent will retry a foreground fork session to surface errors while preserving the same dirty production worktree and immutable tests.

## Task workflow update - 2026-07-11T19:15:58.136Z
- Summary: Implementation is blocked by fork infrastructure failure. After initial fork left a valid-looking uncommitted 12-path production diff, three continuation attempts (`5e4bsizlmfs8`, `oa85ugmvw30q`, plus a foreground fork) failed with `(no output)` before making changes/reporting. Parent cannot edit directly under orchestrator rules. Worktree remains IN-PROGRESS at RED HEAD `b2f5d2a9c`, dirty only in production paths, with immutable tests unchanged. No commit or validation claim.
- Inspected tmux panes after failures; no surviving fork pane was available to resume. No processes were signaled or killed.
- Exact foreground tool error: `Fork error: (no output)`.

## Task workflow update - 2026-07-11T19:21:14.676Z
- Recorded fork run: gfvkft0pj6s7
- Summary: User requested another retry. Fork `gfvkft0pj6s7` launched successfully to resume the existing dirty production implementation, validate via Castor, preserve immutable tests, and commit.

## Task workflow update - 2026-07-11T19:29:11.939Z
- Summary: User requirement recorded: do not launch any reviewer before proper manual testing. This task-start phase will not run a reviewer regardless; implementation must remain IN-PROGRESS until the user-visible flow is manually exercised and accepted, in addition to required automated Castor validation.

## Task workflow update - 2026-07-11T19:31:17.285Z
- Recorded fork run: gfvkft0pj6s7
- Validation: PASS: focused accepted Castor unit/virtual filters (fork report); PASS: both exact castor test:tui GF-02 E2E filters (fork report); PASS: castor test — 4224 tests (fork report); PASS: castor deptrac (fork report); PASS: castor phpstan (fork report); PASS: castor cs-check --path=src (fork report); FAIL: full castor cs-check — style/order in task RED tests; PENDING: proper manual user-visible test before any reviewer
- Summary: Production commit `fedff1662` created on top of RED `b2f5d2a9c`; worktree clean; accepted RED files unchanged. Focused unit/virtual tests, both required replay-backed Tmux E2Es, full castor test (4224), deptrac, phpstan, and src-only cs-check reportedly pass. Implementation is NOT accepted yet and no reviewer will run: proper user/manual testing is still required. Parent also identified cleanup blockers before manual test/review: `SubagentExecutionService` uses optional trailing `?AppConfig = null` solely to avoid updating constructor tests (conflicts with no test-only compatibility paths), and full `castor cs-check` fails on style in the RED test commit.

## Task workflow update - 2026-07-11T19:46:38.906Z
- Summary: User explicitly approved rewriting the two unpublished task commits to fix RED code style and remove the optional test-only AppConfig path, and additionally requested merging current origin/main into the task branch. No reviewer may run before proper manual testing.

## Task workflow update - 2026-07-11T19:47:08.093Z
- Recorded fork run: 1a0l57q1bppy
- Summary: Rewrite+merge fork launched. It will create a safety ref, rebuild CS-clean RED and production commits, require AppConfig with explicit existing-test wiring, merge fetched origin/main into the task branch, run full focused/Castor validation including full cs-check, and stop with manual testing pending and no reviewer.

## Task workflow update - 2026-07-11T20:24:09.558Z
- Recorded fork run: 1a0l57q1bppy
- Validation: PASS: all GF-02 focused Castor unit/virtual filters (8 commands); PASS: both exact castor test:tui E2E replay-backed filters; PASS: full castor test (4239 tests, +15 from merge); PASS: castor test --filter=SubagentExecutionServiceTest (22 tests); PASS: castor deptrac (0 violations); PASS: castor phpstan; PASS: full castor cs-check (after one merge-resolved cs-fix); Safety tag: safety/gf-02-pre-rewrite-20260711 at old fedff1662; RED immutability proof: git diff --name-only 28b6c13bf..792ad920b -- <9 GF-02 RED paths> → empty
- Summary: Rewrite + merge finished with full Castor validation green. Clean worktree at `c21d35282`, which holds rebuilt RED `28b6c13bf` → production `792ad920b` → merge origin/main `9d1a43bdc`. Three repairs applied: `SubagentExecutionService` now requires non-null `AppConfig`; existing constructor tests updated to supply it via container; and `castor cs-check` passes with full tree. Both replay-backed Tmux E2E GF-02 tests pass. No reviewer was launched. Manual user-visible testing is the next required step.

## Task workflow update - 2026-07-11T20:28:41.037Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (105.8s).
- Pushed task/gf-child-context-statistics to origin.
- branch 'task/gf-child-context-statistics' set up to track 'origin/task/gf-child-context-statistics'.
- Created PR: https://github.com/ineersa/agent-core/pull/281
- Validation: PASS: castor test (4239 tests, 13826 assertions); PASS: castor deptrac (0 violations); PASS: castor phpstan (0 errors); PASS: castor cs-check (0 files fixed, full tree)
- Summary: Implementation complete with two-commit sequence (RED 28b6c13bf + production 792ad920b) merged with origin/main (9d1a43bdc). SubagentExecutionService requires non-null AppConfig; context_window flows from catalog through RunStarted metadata into canonical progress payload; one shared pure ContextUsageFormatter drives card, picker, and live footer; full Castor validation green (4239 tests, deptrac, phpstan, full cs-check, both replay-backed Tmux E2Es). No reviewer was launched per user instruction.

## Task workflow update - 2026-07-11T20:41:38.028Z
- Moved CODE-REVIEW → DONE.
- Merged task/gf-child-context-statistics into integration checkout.
- Merge made by the 'ort' strategy.
 depfile.yaml                                       |   1 +
 src/AgentCore/Domain/Run/RunMetadata.php           |   2 +
 .../Execution/SubagentChildProgressSummary.php     |   6 ++
 .../SubagentChildProgressSummaryBuilder.php        |  14 ++-
 .../Agent/Execution/SubagentExecutionService.php   |  99 ++++++++++++++-----
 src/Tui/Footer/ContextUsageDTO.php                 |  19 ++++
 src/Tui/Footer/ContextUsageFormatter.php           |  49 ++++++++++
 src/Tui/Listener/FooterStateSegmentProvider.php    |  45 ++++++---
 src/Tui/Picker/SubagentLivePickerController.php    |  10 ++
 src/Tui/Runtime/SubagentLiveCatalog.php            |  40 ++++++++
 src/Tui/Runtime/SubagentLiveChildDTO.php           |   3 +
 src/Tui/Runtime/SubagentLiveViewState.php          |   2 +-
 src/Tui/Transcript/SubagentResultRenderer.php      |  15 +++
 .../Transcript/SubagentTranscriptCardBuilder.php   |  49 ++++++++++
 .../SubagentChildProgressSummaryBuilderTest.php    |  14 ++-
 .../Execution/SubagentExecutionServiceTest.php     |  10 ++
 tests/Tui/E2E/TuiSubagentLiveViewE2eTest.php       |  11 ++-
 tests/Tui/E2E/TuiSubagentProgressE2eTest.php       |   3 +
 .../Listener/FooterStateSegmentProviderTest.php    |  80 ++++++++++++++++
 .../Picker/SubagentLivePickerControllerTest.php    |  31 ++++++
 tests/Tui/Runtime/SubagentLiveCatalogTest.php      |  66 +++++++++++++
 .../Tui/Support/ChildContextStatisticsFixture.php  | 106 +++++++++++++++++++++
 .../Tui/Support/SubagentProgressEventsFixture.php  |   2 +-
 .../Tui/Transcript/SubagentResultRendererTest.php  |  59 ++++++++++++
 24 files changed, 693 insertions(+), 43 deletions(-)
 create mode 100644 src/Tui/Footer/ContextUsageDTO.php
 create mode 100644 src/Tui/Footer/ContextUsageFormatter.php
 create mode 100644 tests/Tui/Support/ChildContextStatisticsFixture.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/gf-child-context-statistics.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/gf-child-context-statistics.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #281 merged by user. Deepseek provider in E2E settings is a no-URL catalog entry for context window resolution only—no HTTP requests. All automated validation green; two-commit RED-then-production sequence preserved.

## Task workflow update - 2026-08-06T20:59:11.348Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
