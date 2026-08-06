# Migrate transcript to mounted Symfony TUI widgets using the proven spike subset

## Goal
## Context

Salvage the successful core migration from `tui-native-symfony-widget-architecture-spike` into a clean production task.

The spike proved that a mounted Symfony widget subtree:

- gives Markdown widgets a real `WidgetContext`;
- enables Hatfield Markdown sub-element theming;
- supports stable keyed transcript reconciliation;
- feels faster and less clunky than the detached string-rendering path;
- renders user/assistant content and tool exchanges correctly.

The spike also explored many terminal-output workarounds. Those are explicitly not part of this task. Residual bouncing with Symfony's stock writer is an accepted known trade-off for this migration and can be revisited separately through Symfony upstream issue work.

References:

- Source spike task/worktree: `tui-native-symfony-widget-architecture-spike`
- Hatfield follow-up issue: https://github.com/ineersa/agent-core/issues/303
- Symfony writer extension issue: https://github.com/symfony/symfony/issues/64941

## Implementation direction

Use the spike worktree as source material, but port the selected architecture cleanly onto a fresh task branch rather than merging or cherry-picking the entire experimental history.

Include:

- A mounted Symfony transcript container under `ChatScreen`.
- Stable visual-node wrappers keyed by transcript presentation identity.
- Granular add/update/remove/tail-append reconciliation without ordinary whole-transcript `clear()+add()` churn.
- Mounted Symfony `MarkdownWidget` instances for Markdown transcript content.
- Theme stylesheet rules for Markdown headings, links, code, quotes, rules, and list bullets through the live `WidgetContext`.
- Existing transcript presentation behavior: welcome content, turn separators, user/assistant/thinking blocks, tool call/result exchanges, duplicate/suppressed blocks, questions, and generic extension-provided transcript widgets where supported by the spike.
- Stable widget identity during streaming updates where the underlying visual item remains the same.

Explicitly exclude:

- `PiStyleScreenWriter` and every custom ScreenWriter experiment.
- `PiStyleScreenWriterAliasInstaller` or any `class_alias`, reflection, or vendor patch.
- ScreenWriter/terminal redraw telemetry and tmux capability probing.
- Scroll-ahead, tail rewrite, bottom-to-top repaint, or ANSI row-policy code.
- Fixed/clipped viewport compositors, custom transcript scrolling, DECSTBM, scroll regions, and virtualization.
- Tick cadence or frame-rate changes.
- Fixed two-row footer geometry introduced only for bounce experiments.
- `StreamingMonotonicMarkdownWidget` height reservation; treat streaming-height policy as a separate follow-up if still wanted.
- POC-only transcript telemetry unless a small permanent metric has a clear production purpose.
- Dual transcript modes, feature flags, compatibility renderers, or fallback to the old transcript path.

Symfony's normal `Tui` and stock `ScreenWriter` must remain in use.

## Known trade-off

Mounted widgets may produce more visible row movement than the old flattened transcript on terminals or multiplexers without synchronized-output support. This task accepts that limitation in exchange for proper widget lifecycle, styling, maintainability, and improved perceived transcript performance. Do not reopen viewport or terminal-writer work inside this task.

## Acceptance criteria
- `ChatScreen` mounts a first-class Symfony transcript widget subtree and no longer renders the transcript through the detached all-in-one `LiveTextWidget` path.
- Mounted Markdown transcript widgets receive a non-null live `WidgetContext`, and Hatfield Markdown sub-element stylesheet selectors are visibly applied.
- Transcript reconciliation preserves stable visual wrapper/widget identity for ordinary streaming updates and performs granular tail additions/removals without whole-subtree rebuilds.
- User, assistant, thinking, welcome, separator, tool exchange, suppression, question, and supported generic transcript behavior retain production parity.
- The old transcript rendering path is removed rather than retained behind a feature flag, fallback, or compatibility mode; legacy adapters may remain only for TUI regions outside this task's transcript scope.
- Hatfield uses Symfony's stock `Tui` and `ScreenWriter`; no writer alias, replacement, vendor patch, reflection, terminal scroll algorithm, or ScreenWriter telemetry is introduced.
- No viewport, clipping, custom transcript scrolling, DECSTBM, scroll-region, virtualization, ticker, frame-rate, fixed-footer, or streaming-height-reservation experiment is included.
- Virtual/in-process tests prove mounted widget/context behavior, stylesheet-backed Markdown, stable reconciliation, streaming updates, tool exchanges, and key transcript presentation rules.
- Existing replay-backed TUI/tmux product validation is updated only where output legitimately changes and continues to exercise the production path.
- `castor test`, `castor deptrac`, `castor phpstan`, `castor cs-check`, `castor test:tui`, and the required final `castor check` pass before CODE-REVIEW.
- Documentation records the mounted transcript architecture and the accepted stock-writer bouncing trade-off, linking Hatfield issue #303 and Symfony issue #64941.

## Workflow metadata
Status: ARCHIVE
Branch: task/migrate-transcript-to-mounted-symfony-tui-widgets
Worktree: /home/ineersa/projects/agent-core-worktrees/migrate-transcript-to-mounted-symfony-tui-widgets
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/323
PR Status: merged
Started: 2026-07-26T19:11:06.255Z
Completed: 2026-07-27T15:39:06.492Z

## Work log
- Created: 2026-07-23T22:25:38.459Z

## Task workflow update - 2026-07-26T19:11:06.255Z
- Moved TODO → IN-PROGRESS.
- Created branch task/migrate-transcript-to-mounted-symfony-tui-widgets.
- Created worktree /home/ineersa/projects/agent-core-worktrees/migrate-transcript-to-mounted-symfony-tui-widgets.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/migrate-transcript-to-mounted-symfony-tui-widgets.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/migrate-transcript-to-mounted-symfony-tui-widgets.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/migrate-transcript-to-mounted-symfony-tui-widgets.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/migrate-transcript-to-mounted-symfony-tui-widgets.
- Summary: Started implementation. The task will cleanly salvage only the mounted Symfony transcript widget architecture from the preserved spike, while retaining Symfony's stock Tui/ScreenWriter and excluding all terminal writer, viewport, ticker, footer, and streaming-height experiments.

## Task workflow update - 2026-07-26T19:27:44.034Z
- Validation: castor test --filter=TuiMountedTranscriptVirtualTest => OK (3 tests, 40 assertions); castor test --filter=TuiTranscriptBlocksVirtualRenderTest => OK (30 tests, 96 assertions); castor test --filter='TuiStartupVirtualRenderTest|TuiLoadedResourcesVirtualRenderTest|TuiSettingsShowVirtualTest|TuiUsageCommandVirtualTest|ChatScreenTest|PreviewExpansionInputListenerTest' => OK (21 tests, 107 assertions); castor test --filter=TranscriptBlockRendererTest => OK (79 tests, 208 assertions); castor test:tui --filter=TuiRichTranscriptProductValidationE2eTest => OK (1 test, 11 assertions); castor test:tui --filter=TuiTranscriptRenderE2eTest => OK (1 test, 1 assertion); castor deptrac => 0 violations after TuiScreen->AppRuntimeProjection edge; castor phpstan --path=src/Tui/Screen --path=src/Tui/Transcript --path=src/Tui/Listener/PreviewExpansionInputListener.php --path=src/Tui/Application/SessionInitializer.php => 0 errors; castor cs-check => clean
- Summary: Implemented clean production migration of transcript to mounted Symfony widgets without custom ScreenWriter. Commit 3c5c38475 on task/migrate-transcript-to-mounted-symfony-tui-widgets.
- Read task-workflow, testing skill, tests/AGENTS.md, and active task file before edits.
- Ported clean mounted transcript subset from spike without writer/telemetry/streaming-height/footer/viewport experiments.
- Added TuiMountedTranscriptVirtualTest (3 cases) for live context/theming, streaming+tool identity, and granular tail append/removal.
- Local commit only: 3c5c38475. No push/PR. No full castor check.

## Task workflow update - 2026-07-26T19:28:40.355Z
- Validation: castor test --filter=TuiMountedTranscriptVirtualTest: passed (3 tests, 40 assertions); castor test --filter=TuiTranscriptBlocksVirtualRenderTest: passed (30 tests, 96 assertions); related virtual ChatScreen/preview tests: passed (21 tests, 107 assertions); castor test --filter=TranscriptBlockRendererTest: passed (79 tests, 208 assertions); castor test:tui --filter=TuiRichTranscriptProductValidationE2eTest: passed (1 test, 11 assertions); castor test:tui --filter=TuiTranscriptRenderE2eTest: passed (1 test, 1 assertion); castor deptrac: passed (0 violations); scoped castor phpstan for affected TUI production paths: passed (0 errors); castor cs-check: passed; Full castor check intentionally deferred to task-to-pr
- Summary: Implemented clean mounted Symfony transcript migration in commit 3c5c38475. ChatScreen now mounts TranscriptMountedWidget with live Markdown stylesheet context and stable granular reconciliation, while retaining Symfony's stock Tui/ScreenWriter and excluding all spike writer, viewport, footer, telemetry, ticker, and streaming-height experiments.
- Implementation commit: 3c5c38475b96b19afd1ee11f05e0ddece78a7158
- Added TranscriptMountedWidget, TranscriptVisualNodeWidget, MarkdownThemeStyleSheetFactory, TurnSeparatorWidget, WelcomeTranscriptWidget, and a 3-case mounted structural virtual proof.
- Stable reconciliation preserves Markdown and tool-exchange wrapper identity, supports granular tail append/removal, and only outer-resyncs genuine reorder/non-tail insertion.
- Stock Symfony Tui/ScreenWriter retained. Explicitly omitted PiStyle writer/alias, terminal and transcript POC telemetry, streaming height reservation, footer changes, viewport/scroll-region work, and tick/frame changes.
- Read testing skill and tests/AGENTS.md; virtual proof was chosen as lowest correct structural layer, with focused replay-backed tmux product smokes retained.

## Task workflow update - 2026-07-27T00:10:23.812Z
- Validation: Final reviewer: APPROVED at HEAD 396d924f5; castor test: passed (4528 tests, 15794 assertions); an unrelated pre-existing SQLite contention timing test flaked on earlier attempts but passed and reviewer confirmed no task-owned DB/Messenger files; castor test:tui: passed (37 tests, 190 assertions); castor deptrac: passed (0 violations); castor phpstan: passed (0 errors); castor cs-check: passed; Mounted focused virtual proof: passed (3 tests, 40 assertions); Rich transcript TmuxHarness E2E: passed (1 test, 11 assertions)
- Summary: Task-to-PR review completed. Initial reviewer returned APPROVE WITH SUGGESTIONS; cleanup commit 5524eba76 addressed all sensible findings after non-destructive merge of origin/main in 9587b7481. Final reviewer found one stale reflected ChatScreen property; fix commit 396d924f5 updated the subagent-live test and removed dead streaming node state. Final reviewer decision: APPROVED.
- Merged current origin/main non-destructively: 9587b7481
- Post-review cleanup: 5524eba76
- Stale test reflection and dead streaming state fix: 396d924f52343ff1b765dc74a652141813628d0b
- Reviewer explicitly verified real replay-backed TmuxHarness production-path proof and all excluded writer/telemetry/viewport/footer/streaming-height experiments remain absent.
- Branch is clean and ready for deterministic CODE-REVIEW gate.

## Task workflow update - 2026-07-27T00:19:28.346Z
- Validation: Failed deterministic gate: test:tui CancelStickinessE2eTest timed out before assertions waiting 10s for first cursor; empty pane and cache completion at timeout; castor test:tui --filter=CancelStickinessE2eTest: passed (1 test, 2 assertions) after fix; castor cs-check: passed; Reviewer at HEAD 652a89c52: APPROVED
- Summary: First CODE-REVIEW gate failed only because CancelStickinessE2eTest retained a known hard-coded 10s empty-pane startup timeout under parallel contention. Commit 652a89c52 now uses the existing shared 20s contention-aware TUI startup constant; cancellation assertions are unchanged. Reviewer re-approved current HEAD.
- CODE-REVIEW gate startup-timeout fix: 652a89c52aa837bb4d0a12ded984bbb464e9be70
- No production or transcript behavior changed; only the existing E2E startup wait now uses TmuxHarness::TUI_STARTUP_LOGO_TIMEOUT_PARALLEL.

## Task workflow update - 2026-07-27T00:20:56.559Z
- Validation: CODE-REVIEW retry: blocked before gate by active repository Castor lock; no task lanes executed; Lock holder alive: PID 477358, castor:check, sibling worktree memoize-session-event-reads-during-resume
- Summary: Second CODE-REVIEW transition did not run this task's gate because the repository-wide Castor lock was legitimately held by an active check in sibling worktree memoize-session-event-reads-during-resume (PID 477358). No task validation failed; holder was inspected read-only and left untouched. Retry after it finishes.
- No workers were signaled or killed; waiting for normal lock release before retry.

## Task workflow update - 2026-07-27T00:32:51.617Z
- Validation: castor test:tui --filter=TuiTreeCommandE2eTest: OK (2 tests, 10 assertions); castor test:tui: OK (37 tests, 190 assertions); castor cs-check: clean
- Summary: Narrow TuiTreeCommandE2eTest fix: wait for ● idle after BANG_REWIND_07C before /tree rewind so late abandoned tool events cannot re-add the bang block. Commit 250c6060c.
- Gate flake root cause: marker-only wait allowed rewind while shell tool events still in flight (leaf_set then stale tool_execution_start/end).
- Fix: after BANG_REWIND_07C, waitForCallback until ● idle with TUI_GATE_CALLBACK_TIMEOUT_PARALLEL; preserved marker + post-rewind absence assertions.
- No production RuntimeEventPoller/writer/mounted-transcript/tree changes.

## Task workflow update - 2026-07-27T00:36:16.192Z
- Validation: Failed deterministic gate: TuiTreeCommandE2eTest observed late abandoned tool events after rewind because test waited for output but not terminal idle; castor test:tui --filter=TuiTreeCommandE2eTest: passed (2 tests, 10 assertions); castor test:tui: passed (37 tests, 190 assertions); castor cs-check: passed; Reviewer at HEAD 250c6060c: APPROVED
- Summary: Third CODE-REVIEW gate reached full TUI lane and exposed a race in an existing tree-rewind E2E: it rewound immediately after shell output appeared, before tool terminal events settled. Commit 250c6060c now waits for marker plus idle before rewind, matching the test's completed-command thesis. Production transcript code is unchanged; final reviewer re-approved.
- Tree rewind E2E sequencing fix: 250c6060cba015e852e1bf0cfdf03ea24121d94d
- No production behavior changed. The separate runtime race for rewinding while a tool is actively finishing remains out of scope; this test now accurately proves completed canonical bang content disappears after rewind.

## Task workflow update - 2026-07-27T00:38:43.065Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (133.7s).
- Pushed task/migrate-transcript-to-mounted-symfony-tui-widgets to origin.
- branch 'task/migrate-transcript-to-mounted-symfony-tui-widgets' set up to track 'origin/task/migrate-transcript-to-mounted-symfony-tui-widgets'.
- Created PR: https://github.com/ineersa/agent-core/pull/323

## Task workflow update - 2026-07-27T00:38:50.174Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/323
- Updated PR Status: open
- Validation: Deterministic castor check during CODE-REVIEW transition: passed in 133.7s; Branch pushed: task/migrate-transcript-to-mounted-symfony-tui-widgets; PR created: https://github.com/ineersa/agent-core/pull/323
- Summary: Prepared for code review and opened PR #323 after reviewer approval and successful deterministic gate.
- Final CODE-REVIEW HEAD: 250c6060cba015e852e1bf0cfdf03ea24121d94d
- PR #323 is open and task is now CODE-REVIEW.

## Task workflow update - 2026-07-27T14:13:23.901Z
- Validation: Manual real-TTY validation: bare terminal smooth and stable; Manual tmux validation after synchronized-output capability/update: smooth and stable; residual bounce/flicker eliminated
- Summary: User completed manual interactive validation after enabling tmux synchronized-output support/upgrading tmux and reported the mounted transcript TUI is smooth, stable, and no longer bouncing.
- Confirmed the earlier instability was largely caused by the tmux presentation/capability path rather than the mounted Symfony widget architecture itself.

## Task workflow update - 2026-07-27T15:39:06.492Z
- Moved CODE-REVIEW → DONE.
- Merged task/migrate-transcript-to-mounted-symfony-tui-widgets into integration checkout.
- Merge made by the 'ort' strategy.
 depfile.yaml                                       |   1 +
 docs/tui-architecture.md                           |  42 +-
 src/Tui/Application/SessionInitializer.php         |   2 +-
 src/Tui/Listener/PreviewExpansionInputListener.php |   4 +-
 src/Tui/Screen/ChatScreen.php                      |  51 +-
 .../Transcript/MarkdownThemeStyleSheetFactory.php  |  70 +++
 src/Tui/Transcript/TranscriptBlockWidget.php       |   5 +-
 .../Transcript/TranscriptBlockWidgetFactory.php    |   3 +-
 src/Tui/Transcript/TranscriptMountedWidget.php     | 596 +++++++++++++++++++++
 src/Tui/Transcript/TranscriptVisualNodeWidget.php  |  41 ++
 src/Tui/Transcript/TurnSeparatorWidget.php         |  28 +
 src/Tui/Transcript/WelcomeTranscriptWidget.php     |  50 ++
 tests/Tui/E2E/CancelStickinessE2eTest.php          |   4 +-
 tests/Tui/E2E/TuiTranscriptRenderE2eTest.php       |   6 +-
 tests/Tui/E2E/TuiTreeCommandE2eTest.php            |  14 +-
 .../Listener/TickPollListenerSubagentLiveTest.php  |  15 +-
 .../Tui/Screen/TuiMountedTranscriptVirtualTest.php | 245 +++++++++
 .../TuiTranscriptBlocksVirtualRenderTest.php       |   8 +-
 18 files changed, 1124 insertions(+), 61 deletions(-)
 create mode 100644 src/Tui/Transcript/MarkdownThemeStyleSheetFactory.php
 create mode 100644 src/Tui/Transcript/TranscriptMountedWidget.php
 create mode 100644 src/Tui/Transcript/TranscriptVisualNodeWidget.php
 create mode 100644 src/Tui/Transcript/TurnSeparatorWidget.php
 create mode 100644 src/Tui/Transcript/WelcomeTranscriptWidget.php
 create mode 100644 tests/Tui/Screen/TuiMountedTranscriptVirtualTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/migrate-transcript-to-mounted-symfony-tui-widgets.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/migrate-transcript-to-mounted-symfony-tui-widgets.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #323 state: MERGED; Merge commit: 35271a9b7e275df288968c3e4568ea064db6e846; Pre-merge integration checkout clean
- Summary: PR #323 was merged on GitHub as 35271a9b7. Moving the approved task to DONE and synchronizing the integration checkout.

## Task workflow update - 2026-07-27T15:42:20.374Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/323
- Updated PR Status: merged
- Validation: PR #323 merged as 35271a9b7e275df288968c3e4568ea064db6e846; DONE transition merged branch into integration checkout and pulled origin/main; Post-merge `LLM_MODE=true castor check`: deptrac passed; test passed (4526 tests, 15751 assertions); controller replay passed (10 tests, 135 assertions); TUI replay passed (37 tests, 190 assertions); phpstan passed; cs-check passed; cache guard/artifact integrity/leak check passed; Post-merge live lane failure: ShellFollowUpLiveE2eTest issue #183 returned only command.ack + run.completed and no assistant response; Focused `castor test:llm-real --filter=ShellFollowUpLiveE2eTest`: reproduced same unrelated live-runtime failure; `castor clean:cleanup:workers:list`: no stale QA worker candidates; Integration working tree clean; task worktree removed
- Summary: Task completed and merged. Integration checkout synchronized and task worktree removed. Post-merge full gate passed every task-relevant deterministic/TUI/static lane; the unrelated existing live ShellFollowUp issue #183 smoke remains failing consistently and was recorded rather than hidden.
- Final integration HEAD after workflow merge/pull: 2b9d54dd1
- The mounted transcript task does not modify the failing live controller follow-up path; no unrelated production fix was folded into this completed task.

## Task workflow update - 2026-08-06T20:59:22.869Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
