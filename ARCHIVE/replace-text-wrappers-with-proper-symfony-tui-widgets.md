# Replace text wrappers with proper Symfony TUI widgets

## Goal
## Goal
Audit Hatfield's TUI presentation paths and replace semantic UI components that pre-render strings or hide inside `TextWidget` with proper Symfony TUI `AbstractWidget` implementations. Preserve the product's themed visual character. Ordinary static text stays `TextWidget`; do not create one-class-per-line widget noise.

## Hotkeys finalized requirement
- Preserve `/hotkeys` grouping, themed borders/headers/keys/descriptions, empty-state copy, footer notes, and charm.
- Replace `HotkeyTableRenderer` plus pre-rendered ANSI transcript text with a proper `HotkeyTableWidget extends AbstractWidget` that owns structured rows and returns lines directly from `render(RenderContext)`.
- Do not route the table through `TextWidget` or `MarkdownWidget`.
- Use Symfony TUI style elements and `AnsiUtils` (`visibleWidth()`, `truncateToWidth(..., pad: true)`) rather than copied ANSI width/truncation/padding helpers. Recompute from `RenderContext` width so resize works.
- Keep it hotkey-specific unless the audit proves a second real table consumer; no speculative generic table framework.

## Full widget audit
- Inventory every production class named `*Widget`, every `build*Widget()` factory branch, extension slot adapter, and renderer that ultimately returns `TextWidget` or pre-rendered ANSI.
- Convert only semantically rich/stateful/responsive surfaces whose behavior currently leaks across factory/listener/renderer seams. Each converted widget must own its typed input, rendering, style elements, and width-dependent behavior behind its `AbstractWidget` boundary.
- Keep plain transcript text, simple labels, and already-native Symfony widgets as-is.
- Reassess the duplicate `TuiWidget` / `TuiRenderContext` adapter layer only after reference and ExtensionApi compatibility proof; do not break the public extension API.

## Known Symfony TUI reuse
- Replace copied UTF-8/control-byte logic in `EditorState::fromText()` with `Widget\Util\StringUtils::sanitizeUtf8()` and `stripControlBytes()`, preserving CRLF/CR normalization order.
- Let `SelectListWidget` own completion Up/Down selection, wrapping, visible-window scrolling, and selection events while Hatfield retains provider lifecycle, replacement ranges, acceptance/cancellation, and editor focus.
- Replace `WelcomeTranscriptWidget` only if a native widget or a proper semantic widget preserves normal and pathological narrow-width UX with less code.
- Do not import `Widget\Util\Line` directly: `EditorWidget` already owns line editing and Hatfield has no duplicate line engine to remove.
- Do not replace read-only `/settings-show` with `SettingsListWidget`; an interactive `/settings` editor is a separate product decision.

## Discipline
Use the lowest correct virtual/controller-replay/minimal TUI proof. Do not add production seams for tests. Every new widget must delete a larger renderer/factory/listener seam or remain unimplemented.

## Acceptance criteria
- Non-TUI infrastructure is not changed by this task.
- Every TextWidget/pre-rendered-string path is inventoried and classified as CONVERT, DIRECT NATIVE REPLACEMENT, or KEEP with concrete evidence.
- `/hotkeys` uses a real `AbstractWidget` with structured input and direct `RenderContext` rendering; no MarkdownWidget/TextWidget table output or pre-rendered ANSI transcript payload.
- Hotkey visual character is preserved; narrow-width and resize proof covers ANSI-aware column sizing, truncation, and style elements.
- Converted semantic widgets own their rendering and responsive behavior instead of spreading it through listeners, factories, and pure string renderers.
- Plain text remains TextWidget; no speculative generic widget framework or shallow one-use wrappers are introduced.
- EditorState delegates exact sanitation to Symfony TUI StringUtils, and completion navigation delegates to SelectListWidget without focus/replacement regressions.
- ExtensionApi compatibility and slot behavior remain intact.
- Virtual behavior proof is used at the lowest correct layer, followed by required `castor test`, `castor test:controller-replay`, `castor test:tui`, `castor deptrac`, `castor phpstan`, `castor cs-check`, and final `castor check`.
- Final reviewer verifies each new widget is deeper than the seam it replaces and Ponytail verdict is lean.

## Workflow metadata
Status: ARCHIVE
Branch: task/replace-text-wrappers-with-proper-symfony-tui-widgets
Worktree: /home/ineersa/projects/agent-core-worktrees/replace-text-wrappers-with-proper-symfony-tui-widgets
Fork run: dazse6bikky3
PR URL: https://github.com/ineersa/agent-core/pull/386
PR Status: merged
Started: 2026-08-15T00:33:25.122Z
Completed: 2026-08-15T03:37:34.889Z

## Work log
- Created: 2026-08-14T23:40:43.263Z

## Task workflow update - 2026-08-14T23:43:54.758Z
- Recorded fork run: pwxatzqqlpkz
- Summary: Three read-only TUI audits completed (forks hn6643vx3583, i28kvf24dtwi, pwxatzqqlpkz). Highest-confidence targets: HotkeyTableWidget, SubagentProgressCardWidget, completion navigation delegation to SelectListWidget, and convergence of internal TuiWidget/LiveTextWidget chrome onto native AbstractWidget after ExtensionApi reference proof. Plain labels/status lines and existing Markdown/Loader/Editor/SelectList usage should remain native as-is.
- Audit evidence: public ExtensionApi TUI already accepts Symfony AbstractWidget; the internal TuiWidget/TuiRenderContext slot API is the lagging duplicate system. ChatScreen creates 11 LiveTextWidget adapters around six TuiWidget implementors.
- Convert candidates: HotkeyTableRenderer string path (HIGH), SubagentResultRenderer/SubagentTranscriptCardBuilder ANSI card string (HIGH), completion moveNext/movePrevious duplication (HIGH), Header/Footer/Status/Pending/CompactHeader/LoadedResources native promotion plus adapter deletion (MEDIUM), rich multi-line tool cards only if one deep widget deletes substantial factory code (MEDIUM).
- KEEP: WelcomeTranscriptWidget, TurnSeparatorWidget, TranscriptMountedWidget, StreamingMarkdownTranscriptWidget, mounted identity shells, PromptEditor, LoaderWidget, picker/question overlay composition, plain TextWidget labels/system/error/progress/hidden-thinking lines.
- Dead-surface candidate: internal setEditorComponent/getEditorComponent is stored/tested but ChatScreen never reads it; verify product intent before deletion. No Symfony TableWidget exists in vendored TUI; approved solution is a hotkey-specific AbstractWidget using RenderContext and AnsiUtils, not Markdown/TextWidget or a speculative generic framework.

## Task workflow update - 2026-08-15T00:32:30.642Z
- Summary: Task-start blocked before claim: integration checkout has uncommitted changes in .hatfield/settings.yaml and config/hatfield.defaults.yaml (GLM 5.3 settings additions). Left them untouched; no worktree or implementation fork was created.
- 2026-08-14: move_task TODO→IN-PROGRESS failed because the integration checkout is dirty. User action required: commit or stash the existing settings changes, then rerun task-start.

## Task workflow update - 2026-08-15T00:33:25.122Z
- Moved TODO → IN-PROGRESS.
- Created branch task/replace-text-wrappers-with-proper-symfony-tui-widgets.
- Created worktree /home/ineersa/projects/agent-core-worktrees/replace-text-wrappers-with-proper-symfony-tui-widgets.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/replace-text-wrappers-with-proper-symfony-tui-widgets.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/replace-text-wrappers-with-proper-symfony-tui-widgets.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/replace-text-wrappers-with-proper-symfony-tui-widgets.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/replace-text-wrappers-with-proper-symfony-tui-widgets.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/replace-text-wrappers-with-proper-symfony-tui-widgets/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/replace-text-wrappers-with-proper-symfony-tui-widgets.
- Summary: Claimed after integration checkout was cleaned. Starting focused TUI implementation from the finalized acceptance criteria and prior audit.

## Task workflow update - 2026-08-15T00:43:04.295Z
- Summary: Exact-worktree recon completed. Scope gate: implement required HotkeyTableWidget, Symfony StringUtils sanitation delegation, and SelectListWidget-owned completion navigation. Keep broad internal TuiWidget/LiveTextWidget slot migration out of this task because the active unmerged architecture spike is not authoritative; ExtensionApi references prove no public break is needed here. Keep WelcomeTranscriptWidget, mounted transcript shells, Markdown/Loader/Editor/SelectList native paths, generic system/progress/error labels, question/tool/edit/write/static preview TextWidgets, and image metadata paths. Subagent card conversion is optional only if it deletes SubagentTranscriptCardBuilder cleanly without expanding scope.
- Read-only scouts read and followed AGENTS.md, .agents/skills/testing/SKILL.md, and tests/AGENTS.md. Hotkeys currently flow HotkeyTableData → SubmitListener array adapter → HotkeyTableRenderer ANSI string → System TranscriptBlock → generic TextWidget; exact semantic replacement points are SubmitListener, TranscriptBlockWidgetFactory/TranscriptMountedWidget, ChatScreen styles, virtual tests, and replay-backed TuiJourneyE2eTest.
- Audit classifications: CONVERT hotkey table; DIRECT NATIVE REPLACEMENT EditorState sanitation and completion Up/Down selection/window/events; KEEP ordinary TextWidget labels, existing Markdown/Loader/Editor/SelectList widgets, Welcome/TurnSeparator/TranscriptMounted/StreamingMarkdown/ToolExchange/Question shells, static tool previews, and image metadata. Internal Header/Footer/Status/Pending/LoadedResources/CompactHeader adapter migration is deferred pending the separate architecture spike and explicit compatibility decision.
- Required proof plan: focused HotkeyTableWidget row/style/ANSI-width tests plus mounted virtual narrow-width/resize proof, completion virtual navigation/window/acceptance proof, and a real replay-backed TmuxHarness /hotkeys phase in TuiJourneyE2eTest. No live LLM validation is needed because provider/LLM-visible code is untouched.

## Task workflow update - 2026-08-15T00:44:32.492Z
- Recorded fork run: ib66v7qikxof
- Summary: Implementation fork launched in the exact task worktree with required hotkey/subagent semantic widgets, Symfony-native sanitation/completion delegation, focused virtual proofs, and non-negotiable replay-backed TmuxHarness /hotkeys journey proof. Fork must commit but not push or run castor check.

## Task workflow update - 2026-08-15T00:55:41.581Z
- Recorded fork run: ib66v7qikxof
- Validation: Fork ib66v7qikxof: no tests or Castor commands run; not acceptable for completion.
- Summary: First implementation fork exhausted its budget with an uncommitted partial diff (164 tracked insertions / 1057 tracked deletions plus three new files). Production skeleton exists, but stale renderer tests/references remain, real TmuxHarness /hotkeys proof is missing, no validation ran, and no commit exists. Re-forking narrowly to finish and validate; task remains IN-PROGRESS.

## Task workflow update - 2026-08-15T00:56:14.912Z
- Recorded fork run: u87449toyk6b
- Summary: Continuation fork launched on the existing partial diff to finish stale test/reference migration, add mandatory replay-backed TmuxHarness /hotkeys journey proof, run focused/full Castor lanes except castor check, and commit.

## Task workflow update - 2026-08-15T01:11:19.565Z
- Recorded fork run: u87449toyk6b
- Validation: Focused Hotkey/Virtual/Preview/Completion/Editor/Subagent Castor tests: PASS (99 tests).; castor test: PASS (4471 tests, 17432 assertions).; castor test:controller-replay: PASS (12 tests, 165 assertions).; castor test:tui: PASS (36 tests, 284 assertions), including TuiJourneyE2eTest::journeyPhaseHotkeysCatalog and ANSI snapshot.; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-fix followed by castor cs-check: PASS.; Parent verification: commit 2ab543e1e exists on task branch; worktree clean; stale HotkeyTableRenderer/SubagentTranscriptCardBuilder/completion navigation symbol search clean; IDE diagnostics for HotkeyTableWidget show 0 errors.; castor check intentionally not run in task-start; reserved for task-to-pr. Live LLM tests not applicable.
- Summary: Implementation completed and committed as 2ab543e1e2d34b921a7b424c38ecab8cd1f7ba01 (20 files, +1134/-999). Verified clean worktree, expected hotkey/subagent widget additions and renderer/builder deletions, no stale symbols, and a real replay-backed TmuxHarness `/hotkeys` phase in TuiJourneyE2eTest exercising registrars → SubmitListener → mounted HotkeyTableWidget. EditorState now delegates sanitation to Symfony StringUtils; completion navigation/selection/windowing delegates to SelectListWidget. Internal TuiWidget/LiveTextWidget and ExtensionApi remain unchanged per scope gate.

## Task workflow update - 2026-08-15T01:21:09.742Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (14.2s).
- Pushed task/replace-text-wrappers-with-proper-symfony-tui-widgets to origin.
- Pushed task/replace-text-wrappers-with-proper-symfony-tui-widgets to origin.
- Skipped PR creation (pushOnly: true).
- Validation: Focused tests PASS (99 tests).; castor test PASS (4471 tests, 17432 assertions).; castor test:controller-replay PASS (12 tests, 165 assertions).; castor test:tui PASS (36 tests, 284 assertions), including real /hotkeys TmuxHarness journey.; castor deptrac PASS (0 violations).; castor phpstan PASS (0 errors).; castor cs-check PASS.
- Summary: User requested a draft PR for one-eye review; skipping automated reviewer until the draft is ready for formal review. Implementation commit 2ab543e1e; worktree clean.

## Task workflow update - 2026-08-15T01:22:02.703Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Corrected premature workflow transition: user requested only a draft PR for inspection, not CODE-REVIEW. Draft PR created at https://github.com/ineersa/agent-core/pull/386; task remains IN-PROGRESS until user requests formal review/readiness.

## Task workflow update - 2026-08-15T01:22:09.938Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/386
- Updated PR Status: draft
- Summary: Draft PR #386 created for user code inspection. Task intentionally remains IN-PROGRESS.

## Task workflow update - 2026-08-15T01:35:50.869Z
- Recorded fork run: 8y01qydf41g1
- Summary: Addressing draft PR #386 inline comment r3788116457. Narrow fix will generalize the existing Markdown stylesheet factory to own Markdown/hotkey/subagent palette rule creation, deleting copied addRule/styleSheetFromPalette plumbing without adding a new hierarchy/framework or changing widget behavior.

## Task workflow update - 2026-08-15T01:38:27.781Z
- Recorded fork run: kn4ijpb01py1
- Validation: Fork 8y01qydf41g1 failed with no output; verified no worktree changes.
- Summary: Previous comment-fix fork 8y01qydf41g1 failed with no output and left the worktree clean. Relaunched exact narrow fix as kn4ijpb01py1 using deepseek/deepseek-v4-flash at xhigh.

## Task workflow update - 2026-08-15T01:46:47.474Z
- Recorded fork run: kn4ijpb01py1
- Validation: Focused affected Castor tests: PASS (20 tests, 332 assertions) on final state.; castor test: PASS (4471 tests, 17432 assertions).; castor test:tui: PASS (36 tests, 290 assertions).; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS after cs-fix.; Parent verification: commit 0ee7a5c16 exists; worktree clean; stale styleSheetFromPalette/MarkdownThemeStyleSheetFactory search clean; exactly one TUI addRule definition; IDE diagnostics 0 errors.
- Summary: Addressed draft PR #386 comment r3788116457 in commit 0ee7a5c16 and pushed it. Generalized the existing Markdown stylesheet factory into ThemeStyleSheetFactory with one shared palette→Style addRule implementation and explicit Markdown/hotkey/subagent builders. Removed duplicated widget-local styleSheetFromPalette/addRule methods. Draft remains IN-PROGRESS.

## Task workflow update - 2026-08-15T02:00:12.626Z
- Validation: Reviewer inspected full origin/main...HEAD diff, relevant callers, Symfony TUI vendor APIs, Deptrac edges, and tests.; Reviewer decision: REQUEST CHANGES.
- Summary: Mandatory reviewer decision: REQUEST CHANGES. No correctness/security blockers; reviewer found dead CompletionMenu update()/isOpen() methods and an unnecessary restylingSelection guard introduced around setItems/setSelectedIndex (those methods invalidate but do not dispatch SelectionChangeEvent). Narrow deletion fix required before re-review.

## Task workflow update - 2026-08-15T02:00:28.030Z
- Recorded fork run: dazse6bikky3
- Summary: Fix fork launched for reviewer-requested CompletionMenu dead-code deletion only; will commit and run focused/full TUI Castor validation before re-review.

## Task workflow update - 2026-08-15T02:05:03.952Z
- Recorded fork run: dazse6bikky3
- Validation: Fork read/followed testing skill and tests/AGENTS.md.; Focused completion/TUI virtual tests: PASS (48 tests, 86 assertions).; castor test: PASS (4471 tests, 17432 assertions).; castor test:tui: PASS (36 tests, 284 assertions).; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS.; Fork reports clean worktree and no stale CompletionMenu symbols.
- Summary: Reviewer cleanup committed as 293503aa2: deleted dead CompletionMenu update()/isOpen() and impossible SelectListWidget restyle re-entry guard. One file, +6/-37; no behavior/test changes.

## Task workflow update - 2026-08-15T02:10:02.204Z
- Validation: Reviewer decision: APPROVED.; Reviewer independently inspected full origin/main...HEAD diff, changed callers, vendor SelectListWidget/StringUtils contracts, Deptrac edges, and TUI tests.; Final worktree clean; IDE diagnostics for CompletionMenu: 0 errors; stale CompletionMenu symbol search clean.; Latest-HEAD required focused gates already PASS: castor test, castor test:tui, castor deptrac, castor phpstan, castor cs-check.
- Summary: Mandatory re-review after commit 293503aa2: APPROVED. Reviewer confirmed prior blockers resolved, no critical/issues, no unmapped surface or unnecessary complexity, architecture and TUI proof appropriate. Final reviewed HEAD: 293503aa2.

## Task workflow update - 2026-08-15T02:12:35.221Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (133.6s).
- Pushed task/replace-text-wrappers-with-proper-symfony-tui-widgets to origin.
- branch 'task/replace-text-wrappers-with-proper-symfony-tui-widgets' set up to track 'origin/task/replace-text-wrappers-with-proper-symfony-tui-widgets'.
- PR already exists: https://github.com/ineersa/agent-core/pull/386
- Validation: Latest-HEAD castor test: PASS (4471 tests, 17432 assertions).; Latest-HEAD castor test:tui: PASS (36 tests, 284 assertions).; Latest-HEAD focused completion tests: PASS (48 tests, 86 assertions).; Latest-HEAD castor deptrac: PASS (0 violations).; Latest-HEAD castor phpstan: PASS (0 errors).; Latest-HEAD castor cs-check: PASS.; Reviewer: APPROVED.
- Summary: Task-to-pr complete at 293503aa2. Mandatory reviewer APPROVED after one cleanup iteration; prior draft PR comment addressed via shared ThemeStyleSheetFactory; CompletionMenu dead code removed. Existing PR #386 should be used.

## Task workflow update - 2026-08-15T02:12:50.821Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/386
- Updated PR Status: open
- Validation: move_task deterministic castor check: PASS (133.6s).; Branch pushed; PR head verified at 293503aa2.; PR is open, ready for review, and cleanly mergeable.
- Summary: PR #386 updated to final HEAD 293503aa2, body refreshed, and marked ready for review (no longer draft). GitHub reports mergeStateStatus=CLEAN. Task remains CODE-REVIEW.

## Task workflow update - 2026-08-15T03:37:34.889Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/replace-text-wrappers-with-proper-symfony-tui-widgets.
- Merged task/replace-text-wrappers-with-proper-symfony-tui-widgets into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Command/Hotkey/HotkeyTableData.php         |  10 +-
 src/Tui/Completion/CompletionState.php             |  63 ++---
 src/Tui/Editor/EditorState.php                     |  17 +-
 src/Tui/Listener/CompletionListener.php            |  25 +-
 src/Tui/Listener/CompletionMenu.php                |  92 +++----
 src/Tui/Listener/SubmitListener.php                |  25 +-
 src/Tui/Screen/ChatScreen.php                      |  10 +-
 src/Tui/Transcript/HotkeyTableRenderer.php         | 297 ---------------------
 src/Tui/Transcript/HotkeyTableWidget.php           | 288 ++++++++++++++++++++
 .../Transcript/MarkdownThemeStyleSheetFactory.php  |  70 -----
 ...dBuilder.php => SubagentProgressCardWidget.php} | 227 ++++++++++++++--
 src/Tui/Transcript/SubagentResultRenderer.php      | 188 +------------
 src/Tui/Transcript/ThemeStyleSheetFactory.php      | 105 ++++++++
 .../Transcript/TranscriptBlockWidgetFactory.php    |  44 +++
 tests/Tui/Completion/CompletionStateTest.php       | 132 +--------
 tests/Tui/E2E/TuiJourneyE2eTest.php                |  60 ++++-
 tests/Tui/E2E/TuiSubagentProgressE2eTest.php       |   2 +-
 tests/Tui/Listener/CompletionListenerTest.php      |  42 +++
 .../Listener/PreviewExpansionInputListenerTest.php |  41 +--
 .../Tui/Screen/TuiMountedTranscriptVirtualTest.php |   4 +-
 tests/Tui/Screen/TuiVirtualInputTest.php           | 106 ++++++--
 tests/Tui/Transcript/HotkeyTableRendererTest.php   | 181 -------------
 tests/Tui/Transcript/HotkeyTableWidgetTest.php     | 209 +++++++++++++++
 23 files changed, 1152 insertions(+), 1086 deletions(-)
 delete mode 100644 src/Tui/Transcript/HotkeyTableRenderer.php
 create mode 100644 src/Tui/Transcript/HotkeyTableWidget.php
 delete mode 100644 src/Tui/Transcript/MarkdownThemeStyleSheetFactory.php
 rename src/Tui/Transcript/{SubagentTranscriptCardBuilder.php => SubagentProgressCardWidget.php} (56%)
 create mode 100644 src/Tui/Transcript/ThemeStyleSheetFactory.php
 delete mode 100644 tests/Tui/Transcript/HotkeyTableRendererTest.php
 create mode 100644 tests/Tui/Transcript/HotkeyTableWidgetTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/replace-text-wrappers-with-proper-symfony-tui-widgets.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/replace-text-wrappers-with-proper-symfony-tui-widgets.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #386 state: MERGED.; Final task-to-PR deterministic castor check: PASS (133.6s).; User manual smoke test: PASS.
- Summary: User manually tested `/hotkeys`, completion, subagents/pickers smoke and reported it works. GitHub PR #386 merged at 2026-08-15T03:36:47Z with merge commit 0e949d680.

## Task workflow update - 2026-08-15T03:40:49.694Z
- Updated PR Status: merged
- Validation: Post-merge LLM_MODE=true castor check: 7/8 lanes PASS; test:tui FAIL only in TuiTaskListToolE2eTest::testTaskListRendersStructuredPayloadWithoutRedundantMessage.; Focused castor test:tui --filter=TuiTaskListToolE2eTest: reproducible FAIL (1 test, 4 assertions) due terminal line wrap in fixture title.; Worker diagnostics: no stale QA worker candidates.; Failing test was not touched by PR #386; task branch castor check passed before merge.; Integration checkout clean; task worktree removed.
- Summary: DONE cleanup complete: PR #386 merged, task branch merged into integration checkout, exact task worktree closed/removed, IDEA exclusions removed. Post-merge integration `LLM_MODE=true castor check` had one unrelated deterministic failure in TuiTaskListToolE2eTest from separately merged task-list PR #388: expected contiguous `Demo task from E2E`, but terminal output wraps it as `Demo\ntask from E2E`. Focused rerun reproduced. All other check lanes passed; no stale QA workers.

## Task workflow update - 2026-08-15T17:16:43.325Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
