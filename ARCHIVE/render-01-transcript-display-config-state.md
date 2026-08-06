# RENDER-01: Transcript display config, mapper, and live state foundation

## Goal
Part of `.pi/plans/tui-rich-transcript-blocks-plan.md`, revised for the current TUI architecture.

This is the root task for rich transcript rendering. The current TUI already has `ChatScreen`, `LiveTextWidget`, `TranscriptBlockWidget`, `TranscriptBlockRenderer`, `TranscriptProjector`, footer/status widgets, overlays, and slot APIs. Do not rebuild those foundations. Add only the display configuration and session state needed by the existing transcript pipeline.

Order: first/root task. Blocks RENDER-02, RENDER-03, RENDER-04, RENDER-05, and RENDER-06.

Scope:
- Add Hatfield config DTO `Ineersa\CodingAgent\Config\TuiTranscriptConfig` for `tui.transcript.*` settings.
- Add defaults under `tui.transcript`.
- Add TUI-local `Ineersa\Tui\Transcript\TranscriptDisplayConfig`.
- Add session-only `Ineersa\Tui\Transcript\TranscriptDisplayState` with `previewableBlocksExpanded` initialized from config.
- Add mapper/adapter at the TUI application boundary; keep `Tui\Transcript` independent from `CodingAgent\Config`.
- Wire `TranscriptDisplayConfig` and `TranscriptDisplayState` into `TuiSessionState` during interactive TUI startup.
- Stop treating projection `TranscriptBlock::collapsed` as thinking display policy; display policy belongs to `TranscriptDisplayConfig` and renderer code only.
- Update `depfile.yaml` so the `TuiTranscript` layer may use Symfony TUI directly for the renderer work that starts in RENDER-02.

Non-goals:
- Projection owns canonical block facts; rendering config owns local display policy.
- The old `collapsed` thinking behavior is not carried forward.
- No new transcript projection model.
- No replacement for `ChatScreen`, `LiveTextWidget`, `TranscriptProjector`, or the existing runtime event pipeline.

## Acceptance criteria
- `tui.transcript.thinking.visible`, `tui.transcript.thinking.style`, `tui.transcript.previews.expanded_by_default`, `tui.transcript.previews.tool_result_lines`, and `tui.transcript.previews.diff_lines` are parsed with documented defaults.
- `TranscriptDisplayConfig` and `TranscriptDisplayState` exist under `src/Tui/Transcript/` with explicit semantic suffixes.
- `TuiSessionState` exposes the display config and session-only preview expansion state initialized from `previews.expanded_by_default` during interactive startup.
- The mapper from Hatfield config to TUI config lives at the TUI application boundary; `src/Tui/Transcript/` does not depend on `CodingAgent\Config`.
- Thinking visibility is no longer derived from `TranscriptBlock::collapsed`; projection does not set thinking display defaults.
- `depfile.yaml` explicitly allows the dependency needed for `TuiTranscript` to build/render Symfony TUI widgets.
- `.hatfield/settings.yaml` and `docs/settings.md` document the new keys.
- Focused Castor validation is reported, including `castor phpstan`, `castor deptrac`, and focused tests for touched config/state paths.

## Workflow metadata
Status: DONE
Branch: task/render-01-transcript-display-config-state
Worktree: /home/ineersa/projects/agent-core-worktrees/render-01-transcript-display-config-state
Fork run: h3i2wluizyof
PR URL: https://github.com/ineersa/agent-core/pull/238
PR Status: merged
Started: 2026-06-29T20:43:57.597Z
Completed: 2026-06-29T21:43:30.896Z

## Work log
- Created: 2026-05-22T19:08:35.427Z
- Revised: 2026-06-29 — Updated for current TUI architecture and made depfile/display-state responsibilities explicit.

## Task workflow update - 2026-06-29T20:36:23.106Z
- Summary: Task-start attempted but move to IN-PROGRESS was blocked because the integration checkout is not clean: modified .pi/plans/tui-rich-transcript-blocks-plan.md. Waiting for user direction to commit, stash, or otherwise resolve that existing change before claiming the task.

## Task workflow update - 2026-06-29T20:43:57.597Z
- Moved TODO → IN-PROGRESS.
- Created branch task/render-01-transcript-display-config-state.
- Created worktree /home/ineersa/projects/agent-core-worktrees/render-01-transcript-display-config-state.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/render-01-transcript-display-config-state.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/render-01-transcript-display-config-state.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/render-01-transcript-display-config-state.
- Summary: Started by orchestrator after committing the rich transcript plan update requested by user. Loaded task-workflow and testing skills plus tests/AGENTS.md before preparing implementation/test instructions.

## Task workflow update - 2026-06-29T20:47:02.385Z
- Summary: Scouted implementation context with two scout subagents after claiming the task. Key findings: config hydration goes through TuiConfig denormalization in AppConfig::fromContainer; TuiSessionState should gain TranscriptDisplayState; InteractiveMode session loop is the correct initialization point; AssistantStreamProjectionSubscriber hardcodes collapsed:true for thinking in live and replay reconstruction paths; depfile TuiTranscript ruleset needs SymfonyTui allowance; focused tests should cover AppConfig hydration, mapper/state defaults, projection collapsed=false, and a minimal replay-backed TmuxHarness startup proof with new tui.transcript settings per user prompt.

## Task workflow update - 2026-06-29T20:48:22.973Z
- Recorded fork run: gwewazxjzr1h
- Summary: Launched implementation fork gwewazxjzr1h in worktree /home/ineersa/projects/agent-core-worktrees/render-01-transcript-display-config-state. Fork instructions require config/state DTOs, InteractiveMode/TuiSessionState wiring, projection collapsed=false updates, docs/defaults/deptrac updates, focused Castor validation, commit, and a minimal replay-backed TmuxHarness proof per the user prompt.

## Task workflow update - 2026-06-29T20:57:52.665Z
- Recorded fork run: gwewazxjzr1h
- Validation: Fork reported: castor test --filter=AppConfigTest => 7 tests, 23 assertions OK; Fork reported: castor test --filter=TranscriptProjectorTest => 95 tests, 373 assertions OK; Fork reported: castor test --filter=TranscriptDisplayConfigMapperTest => 4 tests, 14 assertions OK; Fork reported: castor test:tui --filter=TuiStartupTranscriptConfigE2eTest => 1 test, 3 assertions PASS; Fork reported: castor test:tui => 19 tests, 94 assertions OK; Fork reported: castor phpstan --path='src/Tui src/CodingAgent/Config src/CodingAgent/Runtime/ProjectionPipeline' => 0 errors; Fork reported: castor deptrac => 0 violations; Fork reported: castor cs-check after cs-fix => clean
- Summary: Implementation fork completed and parent verified branch state. Commit e64095c5d on task/render-01-transcript-display-config-state is present and worktree is clean. Diff against local main contains 16 files changed: config DTOs, TUI display config/state, application mapper, TuiSessionState/InteractiveMode wiring, thinking projection collapsed=false update, docs/defaults/deptrac updates, and focused tests including minimal TmuxHarness startup proof. Parent did not move to CODE-REVIEW yet per task-start workflow; task-to-pr is the next explicit phase.

## Task workflow update - 2026-06-29T21:10:26.535Z
- Summary: Reviewer subagent returned REQUEST CHANGES. Blocking issue: src/CodingAgent/Config/TuiTranscriptConfig.php defines three config classes in one PSR-4 file; TuiTranscriptThinkingConfig and TuiTranscriptPreviewsConfig cannot be autoloaded independently, causing method-level filtered tests/direct construction to fatal. Non-blocking actionable items to address in same iteration: move TranscriptDisplayConfigMapperTest to tests/Tui/Application, qualify user-facing Ctrl+O docs as planned/future until RENDER-06, consider mapping immutable display config once outside InteractiveMode loop, and add themePaths assertion to AppConfigTest to lock in SerializedName behavior. TUI proof passed reviewer assessment.

## Task workflow update - 2026-06-29T21:15:19.395Z
- Recorded fork run: h3i2wluizyof
- Validation: Fork reported: castor test --filter=TranscriptDisplayConfigMapperTest => 4 tests, 14 assertions OK; Fork reported: castor test --filter=testMapWithOverrides => 1 test, 5 assertions OK; Fork reported: castor test --filter=AppConfigTest => 7 tests, 24 assertions OK; Fork reported: castor test --filter=TranscriptProjectorTest => 95 tests, 373 assertions OK; Fork reported: castor test:tui --filter=TuiStartupTranscriptConfigE2eTest => 1 test, 3 assertions OK; Fork reported: castor phpstan --path='src/Tui src/CodingAgent/Config src/CodingAgent/Runtime/ProjectionPipeline' => 0 errors; Fork reported: castor deptrac => 0 violations; Fork reported: castor cs-check => clean
- Summary: Review-fix fork h3i2wluizyof completed. Parent verified clean worktree and commit f8c50761f on task branch. Fixes: split TuiTranscriptThinkingConfig/TuiTranscriptPreviewsConfig/TuiTranscriptConfig into PSR-4 one-class-per-file layout, moved TranscriptDisplayConfigMapperTest to tests/Tui/Application, qualified Ctrl+O docs/docblock as planned future behavior, moved immutable display config mapping outside InteractiveMode session loop, added theme_paths hydration assertion.

## Task workflow update - 2026-06-29T21:21:08.640Z
- Summary: Reviewer re-review returned APPROVED for current HEAD f8c50761f. Reviewer verified PSR-4 autoload fix, mapper test move, Ctrl+O future wording, InteractiveMode config mapping hoist, theme_paths assertion, architecture boundary, and real TmuxHarness E2E proof. Remaining notes were NTH only and non-blocking.

## Task workflow update - 2026-06-29T21:23:53.247Z
- Validation: Reviewer subagent decision: APPROVED for current HEAD f8c50761f; castor test => OK (3880 tests, 12395 assertions) in 21.1s; castor test:tui => OK (19 tests, 94 assertions) in 63.1s; castor deptrac => violations=0, errors=0; castor phpstan => errors=0, file_errors=0; castor cs-check => files_fixed=0 / clean
- Summary: Focused local validation completed by orchestrator after reviewer approval. Current HEAD f8c50761f; worktree clean. Note: after git fetch, origin/main advanced beyond the local-main base and PR diff against origin/main includes the earlier local main plan commit bd757ba9a (.pi/plans/tui-rich-transcript-blocks-plan.md) in addition to RENDER-01 commits.

## Task workflow update - 2026-06-29T21:25:13.526Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (68.6s).
- Pushed task/render-01-transcript-display-config-state to origin.
- branch 'task/render-01-transcript-display-config-state' set up to track 'origin/task/render-01-transcript-display-config-state'.
- Created PR: https://github.com/ineersa/agent-core/pull/238

## Task workflow update - 2026-06-29T21:25:18.017Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/238
- Updated PR Status: open
- Validation: move_task castor check => passed (68.6s); PR created: https://github.com/ineersa/agent-core/pull/238
- Summary: Moved to CODE-REVIEW. move_task deterministic castor check passed in worktree (68.6s), branch pushed to origin/task/render-01-transcript-display-config-state, and PR #238 created.

## Task workflow update - 2026-06-29T21:43:30.896Z
- Moved CODE-REVIEW → DONE.
- Merged task/render-01-transcript-display-config-state into integration checkout.
- Auto-merging depfile.yaml
Merge made by the 'ort' strategy.
 .hatfield/settings.yaml                            |  12 ++
 config/hatfield.defaults.yaml                      |  16 ++
 depfile.yaml                                       |   1 +
 docs/settings.md                                   |  42 +++++
 src/CodingAgent/Config/TuiConfig.php               |  12 +-
 src/CodingAgent/Config/TuiTranscriptConfig.php     |  37 ++++
 .../Config/TuiTranscriptPreviewsConfig.php         |  36 ++++
 .../Config/TuiTranscriptThinkingConfig.php         |  30 +++
 .../AssistantStreamProjectionSubscriber.php        |   6 +-
 src/Tui/Application/InteractiveMode.php            |  19 ++
 .../Application/TranscriptDisplayConfigMapper.php  |  29 +++
 src/Tui/Runtime/TuiSessionState.php                |  25 +++
 src/Tui/Transcript/TranscriptDisplayConfig.php     |  34 ++++
 src/Tui/Transcript/TranscriptDisplayState.php      |  28 +++
 tests/CodingAgent/Config/AppConfigTest.php         |  88 ++++++++-
 .../Runtime/Projection/TranscriptProjectorTest.php |  11 +-
 .../TranscriptDisplayConfigMapperTest.php          |  97 ++++++++++
 .../Tui/E2E/TuiStartupTranscriptConfigE2eTest.php  | 206 +++++++++++++++++++++
 18 files changed, 709 insertions(+), 20 deletions(-)
 create mode 100644 src/CodingAgent/Config/TuiTranscriptConfig.php
 create mode 100644 src/CodingAgent/Config/TuiTranscriptPreviewsConfig.php
 create mode 100644 src/CodingAgent/Config/TuiTranscriptThinkingConfig.php
 create mode 100644 src/Tui/Application/TranscriptDisplayConfigMapper.php
 create mode 100644 src/Tui/Transcript/TranscriptDisplayConfig.php
 create mode 100644 src/Tui/Transcript/TranscriptDisplayState.php
 create mode 100644 tests/Tui/Application/TranscriptDisplayConfigMapperTest.php
 create mode 100644 tests/Tui/E2E/TuiStartupTranscriptConfigE2eTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/render-01-transcript-display-config-state.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/render-01-transcript-display-config-state.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #238 reported merged by user
- Summary: User reported PR #238 was merged; moving tracked task to DONE and merging/syncing integration checkout.
