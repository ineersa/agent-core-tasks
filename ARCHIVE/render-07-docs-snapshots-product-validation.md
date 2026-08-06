# RENDER-07: Docs, snapshots, and rich transcript validation

## Goal
Part of `.pi/plans/tui-rich-transcript-blocks-plan.md`, revised for the current TUI architecture.

Final integration and documentation task for the rich transcript work. This task validates the behavior added by RENDER-01 through RENDER-06 at the lowest correct TUI layers and updates user-facing documentation.

Order: final integration task. Depends on RENDER-01 through RENDER-06.

Scope:
- Update docs for final transcript rendering behavior and keybindings.
- Add or refresh deterministic TUI/widget snapshots for the rich transcript cases.
- Validate the existing `ChatScreen -> TranscriptBlockWidget` rendering path, not a parallel rendering path.
- Validate assistant Markdown, visible thinking, hidden thinking placeholder, tool call YAML args, normal tool result preview, diff preview, and `Ctrl+O` expansion.
- Validate that existing glyphs/prefixes remain stable for unchanged block kinds and status/footer chrome.
- Capture and report session/snapshot artifacts on failure.
- Polish density/visual issues found in snapshots without changing the core architecture or terminal-native visual language.

Non-goals:
- Do not document old renderer behavior as accepted behavior.
- Do not document old `collapsed` thinking behavior as accepted behavior.
- Live LLM validation is required only if implementation changes provider/LLM-visible flow; rich transcript rendering itself should be proven deterministically at the TUI/widget layer.
- No new root layout, transcript event model, or session storage behavior.

## Acceptance criteria
- `docs/settings.md` and relevant TUI docs accurately describe `tui.transcript.*`, visible/hidden thinking, tool YAML args, previews, and `Ctrl+O`.
- Deterministic TUI/widget snapshots or equivalent focused assertions are updated for assistant Markdown, visible thinking, hidden thinking placeholder, tool call YAML args, normal tool result preview, diff preview, and `Ctrl+O` expansion.
- Validation uses the lowest correct layer for each behavior: virtual/widget tests for rendering, TUI input test for `Ctrl+O`, and tmux replay only if terminal integration is actually touched.
- Existing snapshots/assertions are not rewritten just because the renderer internals changed; snapshot updates are limited to intentional new rich transcript behavior.
- Existing glyphs/prefixes remain stable for unchanged block kinds, `● idle` / `◐ Working`, and footer chrome (`◆`, `↻`, `⌂`, `⎇`, `⏱`).
- Tests use named glyph/assertion helpers where practical to prevent raw Unicode duplication from spreading further.
- Product-level Castor validation is run and reported according to the touched layers: `castor test` for virtual/widget proof, `castor test:tui` only for terminal integration, plus `castor phpstan`, `castor deptrac`, and `castor cs-check`.
- Failures include captured snapshot/session artifacts where available, such as ANSI snapshots, `events.jsonl`, runtime event logs, or transcript fixtures.
- Card density is reviewed from snapshots; final cards remain compact/subtle unless user feedback says otherwise.
- Old renderer behavior is not documented or tested as accepted behavior.

## Workflow metadata
Status: DONE
Branch: task/render-07-docs-snapshots-product-validation
Worktree: /home/ineersa/projects/agent-core-worktrees/render-07-docs-snapshots-product-validation
Fork run: otr90gyaab99
PR URL: https://github.com/ineersa/agent-core/pull/250
PR Status: merged
Started: 2026-07-01T21:21:44.242Z
Completed: 2026-07-02T00:59:28.185Z

## Work log
- Created: 2026-05-22T19:09:21.898Z
- Prior note: 2026-05-22T19:11:12.284Z — Earlier validation scope considered a live model path for rich transcript rendering.
- Revised: 2026-06-29 — Updated validation to current project testing policy: deterministic lowest-layer proof for local TUI rendering; live model validation only if provider/LLM-visible behavior changes.
- Revised: 2026-06-29 — Added glyph-stability and snapshot-churn mitigation requirements.

## Task workflow update - 2026-07-01T21:21:44.242Z
- Moved TODO → IN-PROGRESS.
- Created branch task/render-07-docs-snapshots-product-validation.
- Created worktree /home/ineersa/projects/agent-core-worktrees/render-07-docs-snapshots-product-validation.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/render-07-docs-snapshots-product-validation.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/render-07-docs-snapshots-product-validation.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/render-07-docs-snapshots-product-validation.
- Summary: Starting RENDER-07 per user request. Initial context read: task file and rich transcript plan. Note: task says it depends on RENDER-01 through RENDER-06, but user explicitly requested starting this tracked task now; implementation fork will first assess current state and scope to already-merged behavior plus any available Ctrl+O state.

## Task workflow update - 2026-07-01T21:25:13.324Z
- Validation: Scouts confirmed docs/settings.md has stale Ctrl+O wording and incomplete tool_result_lines scope wording; Scouts confirmed no Ctrl+O listener/hotkey exists in src/Tui/Listener or hotkey catalog; Scouts identified existing renderer/virtual coverage in TranscriptBlockRendererTest and TuiTranscriptBlocksVirtualRenderTest, plus tmux config/render tests already present
- Summary: Scouting complete. Key finding: RENDER-06 Ctrl+O preview toggle is still TODO/not implemented in current branch (no listener, no hotkey catalog entry, no runtime mutation of TranscriptDisplayState::previewableBlocksExpanded). RENDER-07 acceptance requires Ctrl+O docs/validation, and user task-start prompt requires a real TmuxHarness E2E proof for this TUI task. Implementation fork will therefore include the minimal Ctrl+O listener/hotkey behavior needed to satisfy RENDER-07 (overlapping the RENDER-06 task), plus final docs/tests/snapshots. Existing rich transcript coverage already exists for markdown/thinking/tool args/previews/diffs at renderer/virtual layers; gaps to cover include docs accuracy, real tmux proof spanning rich transcript path, Ctrl+O real input toggle, and focused additional virtual assertions for current RENDER-05B polish where useful.

## Task workflow update - 2026-07-01T21:26:00.377Z
- Recorded fork run: x2y6y124ztz0
- Summary: Launched implementation fork x2y6y124ztz0 in worktree /home/ineersa/projects/agent-core-worktrees/render-07-docs-snapshots-product-validation. Scope: implement missing minimal Ctrl+O preview expansion listener/hotkey because RENDER-07 acceptance requires it and RENDER-06 is still TODO; update docs/settings.md and relevant TUI docs; add focused virtual/input product validation; add mandatory replay-backed TmuxHarness E2E proof for rich transcript path; preserve glyph contract and compact terminal-native rendering; run focused Castor validation (test, test:tui, deptrac, phpstan, cs-check); commit only, no push/PR/reviewer/castor check.

## Task workflow update - 2026-07-01T21:31:44.344Z
- Recorded fork run: sq2ee7kxwvqb
- Summary: Finish fork sq2ee7kxwvqb failed with no output. Parent is inspecting worktree status and will relaunch a narrower finish/commit fork; no PR/review/gate step will be run.

## Task workflow update - 2026-07-01T21:32:16.005Z
- Recorded fork run: 9wk0re9jii46
- Summary: Second finish/rescue fork 9wk0re9jii46 failed with no output. Worktree still appears dirty with the same expected RENDER-07 implementation changes. Parent will inspect status and decide whether to retry with an even smaller fork or ask user for permission to finish directly because the fork mechanism is failing before handoff.

## Task workflow update - 2026-07-01T22:39:34.012Z
- Validation: Read AGENTS.md context in conversation; loaded testing skill and tests/AGENTS.md before direct validation; castor test --filter='PreviewExpansionInputListenerTest|TuiVirtualInputTest' OK (6 tests, 49 assertions); castor test:tui --filter='TuiRichTranscriptProductValidationE2eTest' OK (1 test, 11 assertions); castor deptrac OK (0 violations); castor phpstan OK (0 errors); castor cs-check OK (0 files fixed); Real TmuxHarness E2E proof added and passed: tests/Tui/E2E/TuiRichTranscriptProductValidationE2eTest.php (replay-backed, no live LLM, saves ANSI snapshot on success/failure)
- Summary: Implementation completed directly with explicit user override after three forks failed with no output in this worktree. Commit c09494f0b (`feat(tui): document and validate rich transcript previews`) adds session-local Ctrl+O preview expansion listener, hotkey catalog entry/docs, rich transcript product docs, virtual input tests, and mandatory replay-backed TmuxHarness E2E product validation. Worktree clean. Stop point reached for task-start phase; no reviewer, PR, push, or castor check run.

## Task workflow update - 2026-07-01T23:37:28.863Z
- Summary: User requested folding custom-tool output preview limiting into RENDER-07 and reminded to use forks. Parent accidentally added failing regression-test edits directly before stopping; now launching fork to take over, fix implementation/tests, validate, and amend/new commit as appropriate.

## Task workflow update - 2026-07-01T23:40:51.333Z
- Recorded fork run: t803ckum51po
- Validation: castor test --filter='TranscriptBlockRendererTest|PreviewExpansionInputListenerTest|TuiVirtualInputTest' OK (85 tests, 257 assertions); castor test:tui --filter='TuiRichTranscriptProductValidationE2eTest' FAILED in fork with tmux waitForCallback timeout waiting for edit diff preview; not caused by fd196ca2e per fork, but must be investigated before CODE-REVIEW; castor deptrac OK (0 violations); castor phpstan OK (0 errors); castor cs-check OK (0 files fixed)
- Summary: Fork t803ckum51po completed at commit fd196ca2e (`fix(tui): limit generic tool exchange previews`). Investigation found production generic tool exchange preview path was already correct: buildGenericToolExchangeWidget -> toolExchangeResultBodyLines -> applyToolResultPreview -> TranscriptLinePreviewService::apply respects toolResultPreviewLines, is_error/cancelled/timed_out full-render, and previewableBlocksExpanded. The observed failing regression was due to test helper renderWidgetLines() silently ignoring config/state arguments; fork fixed helper to accept optional TranscriptDisplayConfig/TranscriptDisplayState, kept the new task_list/custom-tool exchange tests, reverted unrelated .pi/settings.json dirty change, and committed. Worktree reported clean. Fork noted filtered TUI E2E currently failed with a waitForCallback timeout in TuiRichTranscriptProductValidationE2eTest waiting for edit diff preview, believed pre-existing/unrelated and needs follow-up before PR.

## Task workflow update - 2026-07-01T23:47:58.593Z
- Recorded fork run: bm0k6t4ml5co
- Summary: User rejected treating the TuiRichTranscriptProductValidationE2eTest failure as pre-existing and instructed to fix it. Launched fork bm0k6t4ml5co to reproduce the tmux timeout, inspect artifacts, identify real root cause, fix production/test/fixture as appropriate, preserve replay-backed TmuxHarness proof, run focused validation, and commit. No PR/push/move_task/castor check.

## Task workflow update - 2026-07-01T23:54:13.089Z
- Recorded fork run: bm0k6t4ml5co
- Validation: castor test:tui --filter='TuiRichTranscriptProductValidationE2eTest' OK three consecutive runs (~6.4s each, 11 assertions each); castor test:tui OK full suite (26 tests, 138 assertions, ~74s); castor test --filter='TranscriptBlockRendererTest|PreviewExpansionInputListenerTest|TuiVirtualInputTest' OK (85 tests, 257 assertions); castor deptrac OK (0 violations); castor phpstan OK (0 errors); castor cs-check OK (0 files fixed)
- Summary: Fork bm0k6t4ml5co investigated the RENDER-07 TuiRichTranscriptProductValidationE2eTest timeout after user rejected treating it as pre-existing. It reproduced the test repeatedly and found no code/test changes needed: filtered TUI E2E passed 3/3, full castor test:tui passed (26 tests, 138 assertions), and filtered test still passed after full suite. Fork inspected fixtures and rendering paths: replay fixtures are served sequentially regardless of model name, edit exchange renders through buildEditToolExchangeWidget/EditToolCallDiffRenderer/toolExchangeResultBodyLines, TranscriptLinePreviewService respects previewableBlocksExpanded, and block cache key includes previewableBlocksExpanded. No filesystem changes; worktree clean at HEAD fd196ca2e. The earlier single timeout is now considered an environmental/transient TUI harness/process issue based on repeated clean passes, not waived without evidence.

## Task workflow update - 2026-07-02T00:24:48.789Z
- Recorded fork run: jxmcomhv9cea
- Validation: Reviewer subagent result: APPROVE; no blockers. Non-blocking doc contradiction fixed in 4f135ecfb.; castor test:tui --filter='TuiRichTranscriptProductValidationE2eTest' OK (1 test, 11 assertions); castor cs-check OK (0 files fixed)
- Summary: Reviewer returned APPROVE with one non-blocking doc contradiction. Fork jxmcomhv9cea addressed it at commit 4f135ecfb (`docs(tui): clarify rich transcript e2e proof`) by updating TuiRichTranscriptProductValidationE2eTest class docblock to state Ctrl+O is proven primarily at virtual/input layer and also smoke-asserted in tmux. No behavior changes. Worktree clean and ready for CODE-REVIEW.

## Task workflow update - 2026-07-02T00:26:01.963Z
- Recorded fork run: otr90gyaab99
- Validation: castor test:tui --filter='TuiRichTranscriptProductValidationE2eTest' OK (1 test, 11 assertions, ~6.4s); castor cs-check OK (0 files fixed)
- Summary: Correction fork otr90gyaab99 actually applied the reviewer docblock fix after prior fork handoff mismatch. Commit 41e4718f5 (`docs(tui): clarify rich transcript e2e proof`) updates TuiRichTranscriptProductValidationE2eTest docblock to say Ctrl+O preview expansion is proven primarily at virtual/input layer and also smoke-asserted in tmux. No behavior changes. Worktree reported clean and ready for CODE-REVIEW.

## Task workflow update - 2026-07-02T00:27:38.161Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (84.5s).
- Pushed task/render-07-docs-snapshots-product-validation to origin.
- branch 'task/render-07-docs-snapshots-product-validation' set up to track 'origin/task/render-07-docs-snapshots-product-validation'.
- Created PR: https://github.com/ineersa/agent-core/pull/250

## Task workflow update - 2026-07-02T00:59:28.185Z
- Moved CODE-REVIEW → DONE.
- Merged task/render-07-docs-snapshots-product-validation into integration checkout.
- Merge made by the 'ort' strategy.
 docs/settings.md                                   |  18 +-
 docs/tui-architecture.md                           |   2 +
 docs/tui-testing.md                                |   1 +
 src/Tui/Listener/AppHotkeyRegistrar.php            |  11 +-
 src/Tui/Listener/PreviewExpansionInputListener.php |  49 +++++
 src/Tui/Runtime/TuiSessionState.php                |   6 +-
 .../TuiRichTranscriptProductValidationE2eTest.php  | 221 +++++++++++++++++++++
 .../Listener/PreviewExpansionInputListenerTest.php | 210 ++++++++++++++++++++
 tests/Tui/Screen/TuiVirtualInputTest.php           |   2 +-
 .../Tui/Transcript/TranscriptBlockRendererTest.php |  94 ++++++++-
 10 files changed, 601 insertions(+), 13 deletions(-)
 create mode 100644 src/Tui/Listener/PreviewExpansionInputListener.php
 create mode 100644 tests/Tui/E2E/TuiRichTranscriptProductValidationE2eTest.php
 create mode 100644 tests/Tui/Listener/PreviewExpansionInputListenerTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/render-07-docs-snapshots-product-validation.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/render-07-docs-snapshots-product-validation.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User reported PR #250 merged on GitHub and requested moving RENDER-07 to DONE. Integration checkout has an unrelated local .pi/settings.json model preference change; proceeding with requireCleanMain=false to preserve that local change while merging/syncing the completed task.

## Task workflow update - 2026-07-02T01:01:06.712Z
- Validation: LLM_MODE=true castor check OK — qa-20260702-005934-512666-5fec3dcd, quality ok in 221.9s; deptrac OK (1.0s); test OK (4017 tests, 12962 assertions); test:controller-replay OK (8 tests, 112 assertions); test:tui OK (26 tests, 138 assertions); test:llm-real OK (10 tests, 121 assertions); phpstan OK (0 errors); cs-check OK; llama-proxy cache guard OK (entries 23 → 23); QA artifact integrity OK (7 lane logs); QA run leak check OK
- Summary: Post-merge validation completed after DONE transition. RENDER-07 is merged into integration checkout, worktree removed, IDEA exclusions cleaned. Integration checkout still has unrelated local .pi/settings.json model preference change preserved as requested by requireCleanMain=false; no task work remains dirty.
