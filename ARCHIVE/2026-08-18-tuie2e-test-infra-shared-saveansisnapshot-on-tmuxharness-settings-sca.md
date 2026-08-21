# TuiE2E test infra: shared saveAnsiSnapshot on TmuxHarness + settings scaffold builder (de-dupe 21 files)

## Goal
Consolidate TUI E2E (tests/Tui/E2E/) copy-paste test infrastructure into shared helpers. Pure test-infra task, no production code.

## Evidence (from tui-04 review + orchestrator analysis)

1. **saveAnsiSnapshot**: byte-identical private method duplicated in **21 test files** (verified via diff on TuiToolOutputE2eTest vs TuiToolExchangeCardE2eTest — IDENTICAL), plus `snapshotDir` property + mkdir boilerplate (set up in each setUp as `<dir>/.hatfield/tmp/tui/smoke`). TmuxHarness has no snapshot support. Snapshots land in `<testdir>/.hatfield/tmp/tui/smoke/<tag>-<timestamp>.ansi` (kept on pass under var/tmp/tui-e2e-*/).

2. **Settings scaffold**: ~60 lines hand-rolled per file (model/provider/extensions/safe-guard settings array → Yaml::dump → dual write to `<dir>/.hatfield/settings.yaml` AND `<dir>/home/.hatfield/settings.yaml`, wrapped by TuiE2eDatabaseEnv::withSingleLlmWorkerForReplay). Present in 21 E2E files (grep "settings.yaml|Yaml::dump" — most files: settings:3 iso:2). tests/AGENTS.md §"Config fixtures" says: "If the same shape recurs across files, use/share a builder rather than pasting large arrays." Controller E2E has base classes (ControllerReplayE2eTestCase); TUI E2E has none.

3. **Shared pieces already exist and are used**: TestDirectoryIsolation::createProjectTempDir(), TuiE2eDatabaseEnv (allocatePaths/shellPrefix/withSingleLlmWorkerForReplay), TmuxHarness (startDetached/capturePlain*/captureAnsi/sendKey/sendLiteral/waits/killAll).

## Design direction (minimal)

- Move `saveAnsiSnapshot(TmuxPane $pane, string $tag)` onto TmuxHarness (harness derives the snapshot dir from its own state or takes the test-project dir; keep the `<testdir>/.hatfield/tmp/tui/smoke/` location + `<tag>-<timestamp>.ansi` naming — existing artifact conventions and var/tmp retention must not change).
- Add one settings-scaffold helper, e.g. `TuiE2eDatabaseEnv::writeReplaySettings(string $testProjectDir, array $settings): void` (or a small TuiE2eSettings fixture class if the array-shape variance across tests demands a builder with overrides — fork decides the minimal shape; a builder taking overrides array is the documented preference since settings arrays DO vary per test: models, extensions, safe_guard blocks, reasoning flags).
- Mechanical sweep: replace the 21 private copies + per-file scaffolds with the shared helpers. No assertion, wait, or capture behavior changes. No fixture changes.

## Scope / boundaries

- tests/ only (tests/Tui/E2E/* + TmuxHarness.php + TuiE2eDatabaseEnv.php). Zero production changes. Zero behavior changes.
- Do NOT invent a base TestCase class unless genuinely minimal — prefer extending the existing two helper classes. (A base class is acceptable ONLY if it reduces net lines; the goal is shared helpers, not new hierarchy.)
- Per-file settings variance must be preserved exactly (byte-equivalent YAML output for an unchanged test where feasible; at minimum identical effective settings).

## Validation

castor test (unit), castor test:tui (full group — this is the real gate), castor cs-check, castor phpstan. Snapshot artifacts still saved (verify at least one .ansi file lands in the expected location after test:tui run).

## Acceptance criteria
- TmuxHarness owns saveAnsiSnapshot($pane, $tag); zero private saveAnsiSnapshot copies remain under tests/Tui/E2E
- One shared TUI E2E settings scaffold helper (isolated dirs + dual settings.yaml write + replay single-worker wrap); the ~60-line array scaffold is gone from all E2E test files
- All 21 affected TUI E2E test files swept; no per-file copies of either pattern remain
- Behavior identical: snapshots land in equivalent locations; settings files byte-equivalent for an unchanged test
- castor test green; castor test:tui green (full group); castor cs-check + castor phpstan clean; tests/AGENTS.md config-fixture guidance satisfied

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca
Fork run: lljkso0zb8uq
PR URL: https://github.com/ineersa/agent-core/pull/411
PR Status: merged
Started: 2026-08-18T19:21:15.285Z
Completed: 2026-08-18T20:29:37.071Z

## Work log
- Created: 2026-08-18T14:10:42.248Z

## Task workflow update - 2026-08-18T19:21:15.285Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca.

## Task workflow update - 2026-08-18T19:23:17.417Z
- Summary: Task started (task-start phase). Worktree created at /home/ineersa/projects/agent-core-worktrees/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca. Orchestrator scouted: 22 private saveAnsiSnapshot copies (7 md5 variants, all trivial deltas: timestamp format, label-only naming in TuiImagePasteE2eTest, double-write in TuiSubagentProgressE2eTest, 3-arg dir-param variant in TuiSubagentLiveViewE2eTest); 28 files with Yaml::dump dual-write settings scaffold (canonical base in TuiToolOutputE2eTest/TuiJourneyE2eTest; per-test deltas: default_model, tools.output_cap, extra providers, safe_guard tweaks). Implementation fork lljkso0zb8uq launched on worktree with byte-equivalence constraint + full test:tui gate requirement.
- 2026-08-18: task-start claimed, worktree created, fork lljkso0zb8uq launched (shared saveAnsiSnapshot on TmuxHarness + writeReplaySettings/base builder on TuiE2eDatabaseEnv, sweep ~28 files)

## Task workflow update - 2026-08-18T19:40:51.651Z
- Recorded fork run: lljkso0zb8uq
- Validation: castor test --filter=TuiE2eDatabaseEnvTest: PASS (5 tests); castor test: PASS (4628 tests, 18400 assertions); castor test:tui (full group): PASS (40 tests, 341 assertions, 147.6s, no flake); castor cs-check (after cs-fix): PASS; castor phpstan: PASS (errors=0); rg 'private function saveAnsiSnapshot' tests/Tui/E2E: empty; rg 'Yaml::dump' tests/Tui/E2E: only TuiE2eDatabaseEnv.php helper; .ansi artifact proof: var/tmp/tui-e2e-tool-output-4aa4f7c45381d273/.hatfield/tmp/tui/smoke/tool-output-20260818-193757.ansi (verified by orchestrator)
- Summary: Implementation complete via fork lljkso0zb8uq. Commit a7b00d8a46b24f25e664d8213b02fbbbe647f069 on task branch (unpushed, clean worktree). 30 files, +298/−1218, tests/Tui/E2E/ only, zero src/ changes.

Delivered:
- TmuxHarness: setSnapshotDir() + public saveAnsiSnapshot($pane, $tag); all 22 private copies swept (unified signature; ImagePaste label-only naming now timestamped; SubagentProgress dead double-write fixed; LiveView 3-arg variant converted; ProviderError savePlainSnapshot path re-wired).
- TuiE2eDatabaseEnv: replayBaseSettings() (canonical ToolOutput/Journey array) + writeReplaySettings() (withSingleLlmWorkerForReplay + Yaml::dump(6,4) + dual settings.yaml write). All Yaml::dump sites routed through helper (only helper itself greps).
- TuiE2eDatabaseEnvTest: 2 new unit tests for the helpers.
- Non-base settings shapes kept custom (ResumeModelRestore, Startup, RichTranscript, FileRewind, BashBackground, AskHuman/Export/ProcessTransport 127.0.0.1, Subagent compact) — dump/dual-write still via helper.

Known deviation: RichTranscript YAML dump depth 8→6 (array-identical, formatting-only inline-vs-block differences). Fork stated testing skill + tests/AGENTS.md read/followed.
- 2026-08-18: fork lljkso0zb8uq finished; orchestrator verified commit a7b00d8a4, diff stat 30 files +298/−1218 tests-only, worktree clean, artifact present

## Task workflow update - 2026-08-18T20:17:01.374Z
- Updated PR Status: pending
- Validation: Reviewer: APPROVED (0 blockers, NTH-only); castor test: OK (4628 tests, 18400 assertions, 47.6s); castor deptrac: violations=0, errors=0; castor phpstan: errors=0, file_errors=0; castor cs-check: clean (files_fixed=0); castor test:tui: OK (40 tests, 335 assertions, 140.6s)
- Summary: task-to-pr phase: reviewer subagent APPROVED (specification fidelity gate applied; settings arrays verified ===-identical pre/post for 5 sample files; zero src/ changes; no assertion/wait/capture/fixture changes; snapshot naming deviations are diagnostics-only alignment to mandated convention; RichTranscript depth 8→6 verified array-identical). Non-blocking NTHs deliberately not applied (project-dir mkdir absorption, depth param) — would expand mechanical sweep scope. Focused local validation on worktree all green: castor test (4628 pass), castor deptrac (0 violations), castor phpstan (0 errors), castor cs-check (clean), castor test:tui (40 pass, 140.6s). Commit a7b00d8a46b24f25e664d8213b02fbbbe647f069.
- 2026-08-18: task-to-pr — reviewer APPROVED, focused gates green, moving to CODE-REVIEW

## Task workflow update - 2026-08-18T20:19:52.885Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (159.3s).
- Pushed task/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca to origin.
- branch 'task/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca' set up to track 'origin/task/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca'.
- Created PR: https://github.com/ineersa/agent-core/pull/411

## Task workflow update - 2026-08-18T20:20:04.754Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/411
- Updated PR Status: open
- 2026-08-18: CODE-REVIEW transition — deterministic castor check passed in worktree (159.3s), branch pushed, PR #411 created

## Task workflow update - 2026-08-18T20:29:37.071Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca.
- Merged task/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca into integration checkout.
- Merge made by the 'ort' strategy.
 tests/Tui/E2E/BashBackgroundE2eTestSupport.php     |  6 +-
 tests/Tui/E2E/CancelStickinessE2eTest.php          | 63 ++-------------
 tests/Tui/E2E/TmuxHarness.php                      | 25 ++++++
 .../Tui/E2E/TuiAskHumanOverlayMarkdownE2eTest.php  | 24 ++----
 tests/Tui/E2E/TuiAutoCompactionCancelE2eTest.php   | 80 ++-----------------
 tests/Tui/E2E/TuiAutoCompactionE2eTest.php         | 80 ++-----------------
 tests/Tui/E2E/TuiE2eDatabaseEnv.php                | 85 ++++++++++++++++++++
 tests/Tui/E2E/TuiE2eDatabaseEnvTest.php            | 31 ++++++++
 tests/Tui/E2E/TuiExportCommandE2eTest.php          | 25 +-----
 tests/Tui/E2E/TuiFileRewindE2eTest.php             | 22 ++---
 tests/Tui/E2E/TuiHistoryCommandE2eTest.php         | 66 ++-------------
 tests/Tui/E2E/TuiImagePasteE2eTest.php             | 82 ++-----------------
 tests/Tui/E2E/TuiJourneyE2eTest.php                | 93 +++-------------------
 tests/Tui/E2E/TuiOutputCapNoticeE2eTest.php        | 80 ++-----------------
 .../E2E/TuiProcessTransportToolFilterE2eTest.php   | 25 +-----
 tests/Tui/E2E/TuiProviderErrorE2eTest.php          | 88 +++-----------------
 tests/Tui/E2E/TuiQueuedSteerE2eTest.php            | 79 ++----------------
 tests/Tui/E2E/TuiReloadSettingsE2eTest.php         | 69 ++--------------
 tests/Tui/E2E/TuiRepairCommandE2eTest.php          | 55 ++-----------
 tests/Tui/E2E/TuiResumeModelRestoreE2eTest.php     | 27 ++-----
 tests/Tui/E2E/TuiResumeSessionSwitchE2eTest.php    | 76 +++---------------
 .../TuiRichTranscriptProductValidationE2eTest.php  | 21 +----
 tests/Tui/E2E/TuiStartupSnapshotTest.php           |  4 +-
 .../Tui/E2E/TuiStatusRowReasoningNoticeE2eTest.php | 79 ++----------------
 .../TuiSubagentChildHitlCancellationE2eTest.php    |  4 +-
 .../Tui/E2E/TuiSubagentLiveChildExportE2eTest.php  |  4 +-
 tests/Tui/E2E/TuiSubagentLiveViewE2eTest.php       | 37 +++------
 tests/Tui/E2E/TuiSubagentProgressE2eTest.php       | 21 +----
 tests/Tui/E2E/TuiToolExchangeCardE2eTest.php       | 83 ++-----------------
 tests/Tui/E2E/TuiToolOutputE2eTest.php             | 82 ++-----------------
 30 files changed, 298 insertions(+), 1218 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-tuie2e-test-infra-shared-saveansisnapshot-on-tmuxharness-settings-sca.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-08-18T20:33:42.845Z
- Updated PR Status: merged
- Validation: PR #411: MERGED (4919c4afd, 2026-08-18T20:29:15Z); Integration checkout: clean, branch merged + pulled; Worktree: removed, IDEA exclusions cleaned; LLM_MODE=true castor check: all lanes green except test:tui (known pre-existing TODO/fix-tui-tool-exchange-card-e2e-width-fragility, PR #407 width-fragile assertion, deterministic on integration checkout, reproduced in isolation, unrelated to this merge)
- Summary: task-done: PR #411 merged (merge commit 4919c4afd on GitHub, pulled into integration checkout as 772c3dd73, then 71f032cd7). Task branch merged into integration checkout, worktree removed, IDEA exclusions cleaned, JetBrains project closed. Net result: 30 files, +298/−1218 (−920 LOC).

Post-merge validation: LLM_MODE=true castor check — ALL lanes green (test 4628, deptrac 0, phpstan 0, cs-check clean, controller-replay, llm-real, docs:validate) EXCEPT test:tui: 1 failure = TuiToolExchangeCardE2eTest width-fragile assertion. Verified PRE-EXISTING, tracked, deterministic issue from PR #407: TODO/fix-tui-tool-exchange-card-e2e-width-fragility documents this exact failure mode (integration checkout short var/tmp path → 120-col wrap splits "1 addition, 1 deletion"; worktree long path passed by luck — our worktree check passed twice + deterministic gate). Reproduced in isolation via castor test:tui --filter=TuiToolExchangeCardE2eTest. Not a regression from this merge; fix (whitespace-normalize before assertion) already scoped in the existing TODO task. Integration checkout git status clean.
- 2026-08-18: task-done — merged, worktree cleaned. Post-merge check: 1 known pre-existing flake (TuiToolExchangeCardE2eTest width fragility, tracked in TODO/fix-tui-tool-exchange-card-e2e-width-fragility); all other lanes green

## Task workflow update - 2026-08-19T18:16:37.480Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
