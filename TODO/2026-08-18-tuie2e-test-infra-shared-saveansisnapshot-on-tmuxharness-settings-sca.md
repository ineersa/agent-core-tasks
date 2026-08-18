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
Status: TODO
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-08-18T14:10:42.248Z
