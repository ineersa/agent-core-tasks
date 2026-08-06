# RENDER-05: Edit/write ToolCall diff and content rendering

## Goal
Part of `.pi/plans/tui-rich-transcript-blocks-plan.md`, revised for the current TUI architecture.

Render file-modification payloads where they already live after RENDER-04: inside edit/write `ToolCall` cards. Do not duplicate large patches or full file contents in both call and result cards. The call card owns intent and payload preview; the result card owns outcome/status/error output.

Order: depends on RENDER-01, RENDER-02, RENDER-03, and RENDER-04.

Scope:
- Classify supported file-modification `ToolCall` blocks using explicitly documented current tool names and argument metadata.
- Edit-style tools with patch/diff arguments render those arguments through a dedicated diff renderer inside the ToolCall card.
- Write-style tools do **not** invent a diff in v1: write calls render the submitted full content as a content preview inside the ToolCall card.
- For Markdown write targets (`.md`, `.markdown`, or otherwise clearly markdown content), render the write content preview with the existing markdown rendering path where practical; otherwise render a raw content preview. Markdown rendering must still respect preview limits and must not create a bulky panel.
- Keep edit/write `ToolResult` cards compact and outcome-focused. Successful edit/write results should not repeat the patch/content already shown in the ToolCall. Error/cancelled/timed-out results still render their diagnostic output generously per RENDER-04.
- Add dedicated rendering logic/service/widget for edit patch previews and write content previews; do not add a `Diff` transcript kind.
- Use existing theme diff tokens: `DiffAdded`, `DiffRemoved`, and `DiffContext`; use existing markdown tokens/path where Markdown write preview is rendered.
- Apply `tui.transcript.previews.diff_lines` to edit diff previews and write content previews when `previewableBlocksExpanded=false`.
- Respect `TranscriptDisplayState::previewableBlocksExpanded` for both edit diff and write content previews.

Non-goals:
- No new `TranscriptBlockKindEnum::Diff`.
- No duplication of edit/write patch/content between ToolCall and successful ToolResult cards.
- No fabricated write diff when only full content is available.
- No broad parser for every possible tool output.
- Only current documented metadata/tool names are supported.
- No Ctrl+O input listener; this task only consumes preview state.

Glyph/testing stability:
- Edit/write renderers keep the RENDER-04 compact tool-card visual language and add diff/content/markdown styling only inside the card body.
- Diff/content tests should focus on classification, preview line limits, added/removed/context coloring, markdown write preview behavior, and non-duplication of successful result payloads without rewriting unrelated transcript glyph expectations.

## Acceptance criteria
- No new `TranscriptBlockKindEnum::Diff` is added for v1.
- Supported edit/write ToolCall blocks are identified using explicitly documented current metadata/tool names.
- Edit ToolCall cards render patch/diff arguments with dedicated added/removed/context styling.
- Write ToolCall cards render full content as a preview, not as a fabricated diff; Markdown write targets use markdown-aware rendering where practical.
- Long edit diffs and write content previews are limited by `diff_lines` when `previewableBlocksExpanded=false`.
- Edit diff and write content previews render fully when `previewableBlocksExpanded=true`.
- Successful edit/write ToolResult cards remain compact/status-oriented and do not duplicate patch/content already shown in the ToolCall.
- Error/cancelled/timed-out edit/write ToolResult cards continue to render diagnostic output fully/generously per RENDER-04.
- Non edit/write tool calls/results continue through the normal tool card rendering path from RENDER-04.
- Classification behavior is covered by focused tests for each supported edit/write tool name or metadata shape.
- Focused Castor validation is reported for diff/content rendering behavior, including `castor test`, `castor phpstan`, and `castor deptrac`.

## Workflow metadata
Status: DONE
Branch: task/render-05-edit-write-diff-classification-rendering
Worktree: /home/ineersa/projects/agent-core-worktrees/render-05-edit-write-diff-classification-rendering
Fork run: vpdp2oa6h6vl
PR URL: https://github.com/ineersa/agent-core/pull/246
PR Status: merged
Started: 2026-06-30T20:19:55.570Z
Completed: 2026-07-01T00:02:38.492Z

## Work log
- Created: 2026-05-22T19:09:05.491Z
- Revised: 2026-06-29 — Made edit/write classification explicit and documented current-metadata scope.
- Revised: 2026-06-29 — Added diff-rendering test-churn mitigation requirements.
- Revised: 2026-06-30 — Re-scoped from ToolResult diff rendering to ToolCall-level edit diff/write content rendering after RENDER-04 made tool call cards carry full arguments. Added non-duplication rule: ToolCall owns payload preview, successful ToolResult owns outcome/status only. Clarified write has no v1 diff source and should render content preview, with markdown-aware rendering for Markdown write targets where practical.

## Task workflow update - 2026-06-30T20:19:55.570Z
- Moved TODO → IN-PROGRESS.
- Created branch task/render-05-edit-write-diff-classification-rendering.
- Created worktree /home/ineersa/projects/agent-core-worktrees/render-05-edit-write-diff-classification-rendering.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/render-05-edit-write-diff-classification-rendering.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/render-05-edit-write-diff-classification-rendering.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/render-05-edit-write-diff-classification-rendering.
- Summary: User approved RENDER-05 planning decisions and requested implementation start. Decisions: markdown write previews use extension-only detection for .md/.markdown initially; successful edit/write ToolResult cards should strip duplicated payload/context because ToolCall owns patch/content preview; small metadata such as path remains always visible while diff/content body is preview-limited; adding a compact widget/container type for edit/write ToolCall cards is acceptable.

## Task workflow update - 2026-06-30T20:20:43.747Z
- Recorded fork run: uxvy761kiwvn
- Summary: Started implementation fork for RENDER-05. User-approved decisions recorded in fork instructions: markdown write preview detection extension-only (.md/.markdown); successful edit/write ToolResults strip duplicated payload/context; path/header metadata stays visible outside preview budget; patch/content body uses diff_lines and preview state; compact widget/container path allowed if needed while preserving RENDER-04 no-border charisma. Fork instructed to implement edit ToolCall diff renderer, write ToolCall content/markdown preview renderer, successful edit/write ToolResult compaction, focused unit/virtual/tmux tests, and Castor validation.

## Task workflow update - 2026-06-30T20:25:34.501Z
- Recorded fork run: uxvy761kiwvn
- Validation: castor test --filter=TranscriptBlockRendererTest — OK (49 tests); castor test --filter=TuiTranscriptBlocksVirtualRenderTest — OK (26 tests); castor test:tui --filter=TuiToolOutputE2eTest — OK (2 tests); castor test — OK (3944 tests, 12626 assertions); castor test:tui — OK (23 tests, 114 assertions); castor deptrac — OK (0 violations); castor phpstan — OK; castor cs-check — OK after cs-fix
- Summary: Implementation fork completed at commit 24298ec95 (feat(tui): render edit and write payload previews). RENDER-05 implemented edit/write ToolCall payload rendering: edit cards classify tool_name=edit with patch argument, show path outside preview budget, render patch body with DiffAdded/DiffRemoved/DiffContext colors and diffPreviewLines; write cards classify tool_name=write with content argument, show path outside preview budget, render raw or MarkdownWidget content preview for .md/.markdown paths using diffPreviewLines; successful edit ToolResult bodies strip duplicated `Updated file context:` context while errors/cancelled/timed_out remain full. No projection/protocol/block-kind changes.

## Task workflow update - 2026-06-30T20:54:21.838Z
- Recorded fork run: rtnsgurqsqd9
- Summary: Launched review-iterate fork to fold ask_human/HITL transcript cleanup into RENDER-05 per user request. Scope: suppress successful ask_human ToolCall args and ToolResult raw interrupt JSON because Question block owns the prompt; preserve ask_human errors/cancel/timeout; render Question blocks as Markdown while preserving question glyph and answered history; add focused unit/virtual tests and run Castor validation.

## Task workflow update - 2026-06-30T21:00:12.337Z
- Recorded fork run: rtnsgurqsqd9
- Validation: castor test --filter=TranscriptBlockRendererTest — OK (55 tests, 134 assertions); castor test --filter=TuiTranscriptBlocksVirtualRenderTest — OK (27 tests, 88 assertions); castor test — OK (3951 tests, 12649 assertions); castor test:tui — OK (23 tests, 114 assertions); castor deptrac — OK (0 violations); castor phpstan — OK; castor cs-check — OK (0 files to fix)
- Summary: Review-iterate fork completed at commit 72928da10 (feat(tui): suppress ask-human payload noise) on top of 24298ec95. Successful ask_human ToolCall and ToolResult cards are suppressed at render time so the Question block is the single human-readable transcript record; ask_human error/cancel/timeout results remain visible; Question blocks now render via MarkdownWidget with question glyph preserved; answered history remains visible from projection text/meta. No runtime/protocol/config/block-kind changes.

## Task workflow update - 2026-06-30T21:11:40.948Z
- Recorded fork run: 16mgf8wacfwi
- Summary: Launched review-iterate performance fork after user approved three mitigations: flatten RENDER-05 edit/write preview bodies to fewer multiline widgets with ANSI-colored lines, add block-level transcript render caching in TranscriptBlockWidget keyed by block/render inputs, and memoize YAML argument formatting. Fork instructed to preserve glyphs/compact style, keep ask_human cleanup intact, add focused render/cache proof, and run Castor validation.

## Task workflow update - 2026-06-30T21:15:46.695Z
- Recorded fork run: 16mgf8wacfwi
- Validation: castor test --filter=TranscriptBlockRendererTest — OK (59 tests, 142 assertions); castor test --filter=TuiTranscriptBlocksVirtualRenderTest — OK (27 tests, 88 assertions); castor test — OK (3955 tests, 12657 assertions); castor test:tui — OK (23 tests, 114 assertions); castor deptrac — OK (0 violations); castor phpstan — OK; castor cs-check — OK (0 files to fix)
- Summary: Performance hardening fork completed at commit f9afd9021 (perf(tui): cache transcript block rendering) on top of ask_human cleanup. Implemented block-level rendered-line caching in TranscriptBlockWidget keyed by block/render inputs, skipped suppressed ask_human blocks without blank lines, flattened edit diff and write plaintext bodies to multiline TextWidget output, and memoized ToolArgumentsFormatter YAML formatting. Worktree reported clean.

## Task workflow update - 2026-06-30T21:38:05.810Z
- Recorded fork run: c4f4bf8evil6
- Summary: Launched review-iterate fork to continue ask_human/HITL styling per user feedback. Scope: improve Question block styling with markdown prompt and separate answered section, avoid truncated duplicate prompt in active pending widget, optionally suppress empty assistant placeholder before question if low-risk, keep ask_human tool payload suppression and perf cache intact, and run focused/full Castor validation.

## Task workflow update - 2026-06-30T21:43:00.422Z
- Recorded fork run: c4f4bf8evil6
- Validation: castor test --filter=TranscriptBlockRendererTest\|QuestionControllerTest — OK (87 tests, 237 assertions); castor test — OK (3960 tests, 12674 assertions); castor test:tui — OK (23 tests, 114 assertions; reported earlier in fork session); castor deptrac — OK (0 violations; reported earlier in fork session); castor phpstan — OK; castor cs-check — OK (0 files to fix)
- Summary: Ask_human/HITL styling fork completed at commit 5adcc17b2 (refactor(tui): polish human input transcript rendering) on top of f9afd9021. Implemented compact multi-section Question block layout with stable ? glyph, markdown prompt from meta['prompt'], separate answer/status section from meta['answer']/status, no prompt duplication in text-kind QuestionController overlay, and suppression of empty AssistantMessage placeholder immediately before a Question in list rendering. Worktree reported clean. Remaining risk: terminal/full-screen repaint/flicker likely remains a broader redraw/layout issue; this change reduces local prompt duplication and layout churn only.

## Task workflow update - 2026-06-30T21:58:52.655Z
- Recorded fork run: olg1lhoa7fut
- Summary: Launched review-iterate fork for live HITL feedback: restore active text ask_human prompt visibility in the pending/overlay area without old awkward truncation, and ensure answered confirm/choice questions show the submitted yes/no or selected answer in the transcript. Fork instructed to keep prior ask_human suppression, Question styling, empty-assistant suppression, and render cache intact.

## Task workflow update - 2026-06-30T22:02:21.911Z
- Recorded fork run: olg1lhoa7fut
- Validation: castor test --filter='TranscriptBlockRendererTest\|QuestionControllerTest' — OK (88 tests, 242 assertions); castor test --filter=TuiTranscriptBlocksVirtualRenderTest — OK (27 tests, 88 assertions); castor test — OK (3963 tests, 12685 assertions); castor test:tui — OK (23 tests, 114 assertions); castor deptrac — OK (0 violations); castor phpstan — OK; castor cs-fix — fixed 1 file (HitlProjectionSubscriber); castor cs-check — OK
- Summary: Live HITL feedback iteration completed at commit a0ce9f7dc (refactor(tui): keep active human prompt visible) on top of 5adcc17b2. Restored full wrapped text prompt visibility in QuestionController text overlay (no ellipsis truncation), fixed confirm/approval boolean answers through ApplyCommandHandler → RuntimeEventTranslator → HitlProjectionSubscriber so transcript renders yes/no answer sections, and kept prior ask_human suppression/question styling/perf cache intact. Worktree reported clean. Remaining risk: full terminal repaint/flicker unchanged and should be tracked separately if still problematic.

## Task workflow update - 2026-06-30T22:27:13.790Z
- Validation: Reviewer recommended before CODE-REVIEW: castor test, castor test:controller-replay, castor test:tui, castor deptrac, castor phpstan, castor cs-check; move_task CODE-REVIEW will also run deterministic castor check.
- Summary: Final reviewer returned APPROVE WITH SUGGESTIONS on full RENDER-05 stack at a0ce9f7dc. No critical/security/correctness blockers; scope breadth acknowledged as user-approved; confirm boolean data path verified sound. Reviewer requested one cleanup before CODE-REVIEW: remove now-dead TranscriptBlockWidgetFactory::buildRoot() because TranscriptBlockWidget now renders per-block with cache and grep found no callers, avoiding duplicate suppression logic/future divergence. Other suggestions: comment/gate ask_human suppression ordering assumption, simplify compactSuccessfulEditWriteResultBody write no-op branch, share line preview service / reduce factory accessors, docblock update for diffPreviewLines, test fixture should use ui_kind for confirm, tidy blank lines, note style-only self::$this churn.

## Task workflow update - 2026-06-30T22:29:22.988Z
- Recorded fork run: vpdp2oa6h6vl
- Validation: castor test --filter=TranscriptBlockRendererTest — OK (64 tests, 156 assertions); castor test --filter=TranscriptProjectorTest — OK (96 tests, 376 assertions); castor test — OK (3963 tests, 12685 assertions); castor deptrac — OK (0 violations); castor phpstan — OK; castor cs-check initial — one file fixable; castor cs-fix — fixed TranscriptBlockWidgetFactory.php docblock indent; castor cs-check final — OK (0 files to fix)
- Summary: Final reviewer cleanup fork completed at commit caedf1ed1 (refactor(tui): remove obsolete transcript root builder) on top of a0ce9f7dc. Removed dead TranscriptBlockWidgetFactory::buildRoot(), added ask_human suppression ordering comment, simplified compactSuccessfulEditWriteResultBody(), updated diffPreviewLines docblock, changed confirm projection test to realistic ui_kind payload, and tidied renderer test whitespace. Worktree reported clean.

## Task workflow update - 2026-06-30T22:35:51.367Z
- Validation: Final reviewer: APPROVE; Post-cleanup validation: castor test:controller-replay — OK (8 tests, 112 assertions, 50.9s); Post-cleanup validation: castor test:tui — OK (23 tests, 114 assertions, 64.3s); Post-cleanup validation: castor deptrac — OK (0 violations); Post-cleanup validation: castor phpstan — OK (errors=0, file_errors=0); Post-cleanup validation: castor cs-check — OK (files_fixed=0); Worktree status before CODE-REVIEW: clean on task/render-05-edit-write-diff-classification-rendering at caedf1ed1
- Summary: Final re-review after cleanup commit caedf1ed1 returned APPROVE. Reviewer verified buildRoot removal and all prior cleanup requests addressed; no new blockers in rendering/cache/HITL paths. Worktree clean at HEAD caedf1ed1.

## Task workflow update - 2026-06-30T22:37:20.350Z
- Validation: move_task to CODE-REVIEW: castor check FAILED (exit 1) due llama-proxy cache grew from 24 to 26 entries; no code/test failure reported in move_task output.
- Summary: CODE-REVIEW transition attempted but deterministic castor check failed only on llama-proxy cache guard: cache grew from 24 to 26 entries during castor check, indicating uncached live LLM requests. Worktree remains IN-PROGRESS. Next action: warm proxy cache with castor test:llm-real, verify cache stats stabilize, then retry move_task to CODE-REVIEW.

## Task workflow update - 2026-06-30T22:38:09.084Z
- Validation: Warmup: castor test:llm-real — OK (9 tests, 110 assertions); cache entries now 27; Stability check: cache stats before={entries:27}, reran castor test:llm-real — OK (9 tests, 110 assertions), cache stats after={entries:27}; proxy cache stable for retry
- Summary: Warmed llama-proxy cache after CODE-REVIEW gate cache-guard failure. Re-ran castor test:llm-real twice; second run confirmed cache entries stable at 27.

## Task workflow update - 2026-06-30T22:39:32.879Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (73.0s).
- Pushed task/render-05-edit-write-diff-classification-rendering to origin.
- branch 'task/render-05-edit-write-diff-classification-rendering' set up to track 'origin/task/render-05-edit-write-diff-classification-rendering'.
- Created PR: https://github.com/ineersa/agent-core/pull/246

## Task workflow update - 2026-07-01T00:02:38.492Z
- Moved CODE-REVIEW → DONE.
- Merged task/render-05-edit-write-diff-classification-rendering into integration checkout.
- Merge made by the 'ort' strategy.
 .../Application/Pipeline/ApplyCommandHandler.php   |   2 +-
 .../HitlProjectionSubscriber.php                   |  34 +-
 .../Runtime/Protocol/RuntimeEventTranslator.php    |   2 +-
 src/Tui/Question/QuestionController.php            |  12 +-
 src/Tui/Transcript/EditToolCallDiffRenderer.php    |  63 +++
 src/Tui/Transcript/ToolArgumentsFormatter.php      |  15 +-
 src/Tui/Transcript/TranscriptBlockWidget.php       | 124 ++++-
 .../Transcript/TranscriptBlockWidgetFactory.php    | 302 ++++++++++--
 src/Tui/Transcript/TranscriptDisplayConfig.php     |   2 +-
 .../Transcript/TranscriptLinePreviewService.php    |  46 ++
 .../Transcript/WriteToolCallContentRenderer.php    |  96 ++++
 .../Runtime/Projection/TranscriptProjectorTest.php |  17 +
 .../CodingAgent/Runtime/RuntimeEventMapperTest.php |  15 +
 tests/Tui/E2E/TuiToolOutputE2eTest.php             |  88 ++++
 tests/Tui/E2E/fixtures/tui-tool-call-edit.json     |  62 +++
 tests/Tui/Question/QuestionControllerTest.php      | 211 ++++----
 .../TuiTranscriptBlocksVirtualRenderTest.php       | 113 ++++-
 .../Tui/Transcript/TranscriptBlockRendererTest.php | 543 ++++++++++++++++++++-
 18 files changed, 1595 insertions(+), 152 deletions(-)
 create mode 100644 src/Tui/Transcript/EditToolCallDiffRenderer.php
 create mode 100644 src/Tui/Transcript/TranscriptLinePreviewService.php
 create mode 100644 src/Tui/Transcript/WriteToolCallContentRenderer.php
 create mode 100644 tests/Tui/E2E/fixtures/tui-tool-call-edit.json
- Removed worktree /home/ineersa/projects/agent-core-worktrees/render-05-edit-write-diff-classification-rendering.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/render-05-edit-write-diff-classification-rendering.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Pre-DONE: gh pr view 246 — state=MERGED, url=https://github.com/ineersa/agent-core/pull/246; Pre-DONE: integration checkout clean, main...origin/main ahead 2
- Summary: PR #246 was already merged on GitHub (mergedAt 2026-07-01T00:01:59Z). Proceeding to DONE merge/sync and post-merge validation.

## Task workflow update - 2026-07-01T00:08:19.308Z
- Validation: move_task DONE — moved CODE-REVIEW → DONE, merged task/render-05-edit-write-diff-classification-rendering, removed worktree and IDEA exclusions, pulled integration checkout; Initial post-merge LLM_MODE=true castor check qa-20260701-000245-448276-6c2d3891 — FAILED only test:llm-real; deptrac OK, test OK (3959 tests, 12673 assertions), test:controller-replay OK (8 tests, 112 assertions), test:tui OK (23 tests, 114 assertions), phpstan OK, cs-check OK, llama-proxy cache guard OK (23→23), artifact integrity OK, leak check OK; Focused rerun castor test:llm-real — OK (9 tests, 110 assertions); Final post-merge LLM_MODE=true castor check qa-20260701-000638-456423-5e586c54 — OK quality in 186.1s: deptrac OK, test OK (3959 tests, 12673 assertions), test:controller-replay OK (8 tests, 112 assertions), test:tui OK (23 tests, 114 assertions), test:llm-real OK (9 tests, 110 assertions), phpstan OK, cs-check OK, llama-proxy cache guard OK (23→23), QA artifact integrity OK, leak check OK; Cleanup: /home/ineersa/projects/agent-core-worktrees/render-05-edit-write-diff-classification-rendering removed; Integration checkout: git status clean, main...origin/main ahead 2
- Summary: Post-merge validation complete. PR #246 was already merged on GitHub; move_task DONE merged task branch into integration checkout, removed worktree and IDEA exclusions, and pulled/synced. Initial post-merge castor check failed only in test:llm-real (SubagentRetrieveLiveE2eTest did not start agent_retrieve) while all other lanes and cache/leak checks passed; immediate focused rerun of castor test:llm-real passed, then full LLM_MODE=true castor check passed. Worktree directory removed. Integration checkout has no working-tree changes but remains ahead of origin/main by 2 local merge commits produced by move_task/pull after the GitHub PR merge; not force-resetting/destructively rewriting without explicit approval.
