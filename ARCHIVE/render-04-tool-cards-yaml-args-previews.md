# RENDER-04: Tool cards with fenced YAML args and normal output previews

## Goal
Part of `.pi/plans/tui-rich-transcript-blocks-plan.md`, revised for the current TUI architecture.

Build on the RENDER-02 Symfony TUI transcript renderer and render existing `ToolCall` and non-diff `ToolResult` blocks as compact cards. This task owns normal tool output previews; diff-classified edit/write results are handled by RENDER-05.

Order: depends on RENDER-01 and RENDER-02. Can run in parallel with RENDER-03. RENDER-05 builds on this card pipeline.

Scope:
- Render tool calls and normal tool results as compact/subtle cards with semantic headers.
- Tool call arguments must be visible and rendered as fenced YAML in the tool-call card.
- Define and document the exact metadata source for tool arguments, preferring existing `TranscriptBlock::$meta` keys emitted by the projection pipeline.
- Implement normal `ToolResult` line previews using `tui.transcript.previews.tool_result_lines`.
- Respect `TranscriptDisplayState::previewableBlocksExpanded` for normal tool results.
- Do not preview/collapse tool calls themselves.
- Keep errors generous/full by default; do not hide useful failures behind tiny previews.

Non-goals:
- The active flat tool renderer is replaced in this task rather than kept as an alternate renderer.
- No use of `TranscriptBlock::collapsed` as the preview/collapse source.
- No diff classification or diff rendering; RENDER-05 owns edit/write diff results.
- No new transcript block kind.

Glyph/testing stability:
- Tool cards keep the existing `●` tool visual language in headers/prefixes.
- Tool card tests should assert semantic tool-card behavior and use named glyph helpers/constants where practical.
- Existing non-tool transcript snapshots should not be rewritten by this task.

## Acceptance criteria
- Tool call blocks render as cards with tool name/status header.
- Tool call arguments render as fenced YAML using a documented `TranscriptBlock::$meta` source.
- Normal tool result blocks render as cards with readable output.
- Long normal tool results are previewed to `tool_result_lines` when `previewableBlocksExpanded=false`.
- Normal tool results render fully when `previewableBlocksExpanded=true`.
- Tool calls, user messages, assistant messages, thinking, system, progress, question, approval, cancelled, and error blocks are not affected by normal result preview truncation.
- `TranscriptBlock::collapsed` is not used as the trigger for normal tool result previews.
- Focused Castor validation is reported for tool card rendering and preview behavior, including `castor test`, `castor phpstan`, and `castor deptrac`.

## Workflow metadata
Status: DONE
Branch: task/render-04-tool-cards-yaml-args-previews
Worktree: /home/ineersa/projects/agent-core-worktrees/render-04-tool-cards-yaml-args-previews
Fork run: kz1f41zhjkys
PR URL: https://github.com/ineersa/agent-core/pull/244
PR Status: merged
Started: 2026-06-30T16:12:30.339Z
Completed: 2026-06-30T20:07:09.456Z

## Work log
- Created: 2026-05-22T19:08:57.316Z
- Revised: 2026-06-29 — Clarified normal tool preview ownership and the `collapsed` boundary.
- Revised: 2026-06-29 — Added tool glyph-stability and snapshot-churn mitigation requirements.

## Task workflow update - 2026-06-30T16:12:30.339Z
- Moved TODO → IN-PROGRESS.
- Created branch task/render-04-tool-cards-yaml-args-previews.
- Created worktree /home/ineersa/projects/agent-core-worktrees/render-04-tool-cards-yaml-args-previews.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/render-04-tool-cards-yaml-args-previews.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/render-04-tool-cards-yaml-args-previews.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/render-04-tool-cards-yaml-args-previews.
- Summary: User approved RENDER-04 planning decisions and requested implementation start. Decisions: use minimal/no-border compact tool cards to preserve dense transcript charisma; use Symfony YAML dependency for arguments; error/cancelled/timed-out tool results should render full even if large. Moving to IN-PROGRESS and creating worktree for implementation fork.

## Task workflow update - 2026-06-30T16:13:56.882Z
- Recorded fork run: 5qs5rlw4duov
- Summary: Implementation fork launched for RENDER-04 on worktree /home/ineersa/projects/agent-core-worktrees/render-04-tool-cards-yaml-args-previews. Fork instructions include user decisions: minimal/no-border compact cards, Symfony YAML for arguments, full rendering for error/cancelled/timed-out tool results, TranscriptDisplayState wiring into transcript renderer path, no diff/Ctrl+O scope, automated virtual + replay-backed Tmux proof, and Castor-only validation.

## Task workflow update - 2026-06-30T16:18:20.947Z
- Recorded fork run: 0rnpc5fyiwxm
- Summary: Relaunched implementation fork with exact RENDER-04 plan after user reported previous fork did not launch. Plan covers minimal/no-border tool cards, Symfony YAML arguments, full error/cancelled/timed-out results, TranscriptDisplayState wiring from InteractiveMode→ChatScreen→TranscriptBlockWidget→TranscriptBlockWidgetFactory, dedicated ToolCall/ToolResult branches preserving SubagentResultRenderer, focused renderer/virtual/tmux tests, and Castor-only validation.

## Task workflow update - 2026-06-30T16:22:36.998Z
- Recorded fork run: 0rnpc5fyiwxm
- Validation: castor test --filter=TranscriptBlockRendererTest — OK (40 tests, 87 assertions); castor test --filter=TuiTranscriptBlocksVirtualRenderTest — OK (23 tests, 70 assertions); castor test --filter='TranscriptBlockRendererTest\|TuiTranscriptBlocksVirtualRenderTest' — OK (63 tests, 157 assertions); castor test:tui --filter=TuiToolOutputE2eTest — OK (1 test, 7 assertions); castor deptrac — OK (0 violations); castor phpstan — OK; castor cs-check — OK
- Summary: Implementation complete at commit 81c67793f (feat(tui): render tool cards with YAML previews). Verified worktree clean on branch task/render-04-tool-cards-yaml-args-previews and diff vs origin/main is 9 files changed, 529 insertions, 80 deletions. Core changes: compact no-border ToolCall cards with fenced Symfony YAML arguments; normal ToolResult cards with successful-output preview truncation via TranscriptDisplayConfig::toolResultPreviewLines and TranscriptDisplayState::previewableBlocksExpanded; error/cancelled/timed_out tool results render full; TranscriptDisplayState wired InteractiveMode→ChatScreen→TranscriptBlockWidget→TranscriptBlockWidgetFactory; SubagentResultRenderer remains first/unchanged; depfile permits TuiTranscript→SymfonyYaml.

## Task workflow update - 2026-06-30T18:19:17.940Z
- Recorded fork run: ooci982ok2uh
- Summary: Launched refinement fork after user visual review. Requested changes: remove visible ```yaml fences from tool arg cards; keep YAML-like args but format multiline patch/content with literal block style instead of escaped \n strings; preview-limit large ToolCall arguments using existing preview state/config so write/edit params don't flood transcript; preserve ToolResult preview/full-error behavior; investigate `◇ [assistant]` source and fix fixture/test artifact if local rather than adding renderer stripping.

## Task workflow update - 2026-06-30T18:21:22.082Z
- Recorded fork run: ooci982ok2uh
- Validation: castor test --filter=TranscriptBlockRendererTest — OK (43 tests, 96 assertions); castor test --filter=TuiTranscriptBlocksVirtualRenderTest — OK (24 tests, 73 assertions); castor test:tui --filter=TuiToolOutputE2eTest — OK (1 test, 7 assertions); castor deptrac — OK (0 violations); castor phpstan — OK; castor cs-fix + castor cs-check — OK
- Summary: Visual refinement complete at commit ab9271e12 (refactor(tui): simplify tool card argument previews), on top of 81c67793f. Verified worktree clean on branch task/render-04-tool-cards-yaml-args-previews. Combined diff vs origin/main is 10 files changed, 709 insertions, 80 deletions. Refinement removes visible fenced YAML markers from tool arg cards, adds ToolArgumentsFormatter using Symfony YAML DUMP_MULTI_LINE_LITERAL_BLOCK for readable multiline patch/content args, preview-limits ToolCall argument display using existing toolResultPreviewLines + previewableBlocksExpanded, preserves ToolResult preview/full-error behavior, and leaves `◇ [assistant]` unchanged after finding it is the existing empty AssistantMessage placeholder rather than a fixture artifact.

## Task workflow update - 2026-06-30T19:11:30.480Z
- Summary: Reviewer returned APPROVE WITH SUGGESTIONS for RENDER-04 at HEAD ab9271e12. No critical/security/correctness/integration blockers. Verified compact no-border cards, Symfony YAML/deptrac, TranscriptDisplayState wiring, no collapsed-driven preview, no visible YAML fences, multiline literal-block args, ToolCall arg preview, ToolResult preview/full-error behavior, SubagentResultRenderer first/unchanged, and no RENDER-05/06/provider scope creep. Non-blocking suggestions: clean a vacuous escaped-newline assertion, remove duplicate patch assertions, consider DRYing YAML dump for array/object results, remove invalid inline @var comment, restore a small invariant comment around collapsed/display state, optionally document <=0 preview behavior and singular ellipsis path.

## Task workflow update - 2026-06-30T19:13:01.538Z
- Validation: reviewer subagent — APPROVE WITH SUGGESTIONS, no blockers; castor test — OK (3936 tests, 12596 assertions); castor test:tui — OK (22 tests, 108 assertions); castor deptrac — OK (0 violations); castor phpstan — OK (errors=0, file_errors=0); castor cs-check — OK (files_fixed=0)
- Summary: Task-to-pr local validation completed at HEAD ab9271e12 after reviewer approval. Full unit/integration, full replay-backed TUI E2E, deptrac, phpstan, and cs-check all pass in the RENDER-04 worktree.

## Task workflow update - 2026-06-30T19:14:18.969Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (65.8s).
- Pushed task/render-04-tool-cards-yaml-args-previews to origin.
- branch 'task/render-04-tool-cards-yaml-args-previews' set up to track 'origin/task/render-04-tool-cards-yaml-args-previews'.
- Created PR: https://github.com/ineersa/agent-core/pull/244

## Task workflow update - 2026-06-30T19:21:16.960Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Review-iterate after user PR comments: several rationale comments/docblocks were pruned in TranscriptBlockWidget and TranscriptBlockWidgetFactory and should be restored or rewritten rather than removed. Also address small reviewer suggestions while touching files if low-risk.

## Task workflow update - 2026-06-30T19:21:56.857Z
- Recorded fork run: kz1f41zhjkys
- Summary: Launched review-iterate fork to address PR comments about pruned comments/docblocks. Instructions: restore/rewrite rationale comments in TranscriptBlockWidget and TranscriptBlockWidgetFactory, including widget-tree architecture, no flat fallback path, display config/state ownership, hidden-thinking-not-collapsed invariant, branch-boundary comments, markdown left-padding rationale, helper section comments, preview-state behavior comments, plus small reviewer cleanup nits (vacuous assertion, duplicate patch assertions, invalid inline @var).

## Task workflow update - 2026-06-30T19:23:15.849Z
- Recorded fork run: kz1f41zhjkys
- Validation: castor test --filter=TranscriptBlockRendererTest — OK (43 tests, 95 assertions); castor test --filter=TuiTranscriptBlocksVirtualRenderTest — OK (24 tests, 72 assertions); castor test:tui --filter=TuiToolOutputE2eTest — OK (1 test, 7 assertions); castor deptrac — OK (0 violations); castor phpstan — OK; castor cs-check — OK
- Summary: Review-iteration fork completed at commit bf9523ea2 (docs(tui): restore transcript renderer rationale comments). Changes restore/rewrite rationale docblocks and branch-boundary comments in TranscriptBlockWidget and TranscriptBlockWidgetFactory per PR feedback; remove invalid ToolArgumentsFormatter inline @var comment; replace vacuous escaped-newline test assertion with meaningful `+++ b/tmp/test.md` assertion; remove duplicate `patch: |` assertions. Production behavior unchanged except comments.

## Task workflow update - 2026-06-30T19:28:17.062Z
- Summary: Re-review after comment-restoration commit bf9523ea2 returned APPROVED. Reviewer verified restored TranscriptBlockWidget and TranscriptBlockWidgetFactory docblocks/comments are accurate, branch comments match code, hidden-thinking-not-collapsed invariant is documented, preview helper docs match behavior, projection is_error/cancelled/timed_out note is correct, markdown left-padding rationale is accurate, and test cleanups are valid. No critical issues, no behavior regression, no scope creep.

## Task workflow update - 2026-06-30T19:29:50.810Z
- Validation: re-reviewer subagent — APPROVED at bf9523ea2; castor test — OK (3936 tests, 12594 assertions); castor test:tui — OK (22 tests, 108 assertions); castor deptrac — OK (0 violations); castor phpstan — OK (errors=0, file_errors=0); castor cs-check — OK (files_fixed=0)
- Summary: Final local validation after review-iteration commit bf9523ea2 completed. Full unit/integration, full replay-backed TUI E2E, deptrac, phpstan, and cs-check all pass in the RENDER-04 worktree. Worktree remains clean and one commit ahead of origin/task branch.

## Task workflow update - 2026-06-30T19:31:04.990Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (65.2s).
- Pushed task/render-04-tool-cards-yaml-args-previews to origin.
- branch 'task/render-04-tool-cards-yaml-args-previews' set up to track 'origin/task/render-04-tool-cards-yaml-args-previews'.
- PR already exists: https://github.com/ineersa/agent-core/pull/244

## Task workflow update - 2026-06-30T20:07:09.456Z
- Moved CODE-REVIEW → DONE.
- Merged task/render-04-tool-cards-yaml-args-previews into integration checkout.
- Merge made by the 'ort' strategy.
 depfile.yaml                                       |   1 +
 src/Tui/Application/InteractiveMode.php            |   2 +-
 src/Tui/Screen/ChatScreen.php                      |   4 +-
 src/Tui/Transcript/ToolArgumentsFormatter.php      |  38 +++
 src/Tui/Transcript/TranscriptBlockWidget.php       |  25 +-
 .../Transcript/TranscriptBlockWidgetFactory.php    | 257 +++++++++++++----
 tests/Tui/E2E/TuiToolOutputE2eTest.php             |  18 ++
 .../TuiTranscriptBlocksVirtualRenderTest.php       | 155 ++++++++++
 tests/Tui/Support/VirtualTuiHarness.php            |   3 +
 .../Tui/Transcript/TranscriptBlockRendererTest.php | 313 ++++++++++++++++++++-
 10 files changed, 750 insertions(+), 66 deletions(-)
 create mode 100644 src/Tui/Transcript/ToolArgumentsFormatter.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/render-04-tool-cards-yaml-args-previews.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/render-04-tool-cards-yaml-args-previews.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User reported PR #244 was merged. Moving RENDER-04 to DONE, merging task branch into integration checkout, cleaning worktree/exclusions, and syncing integration checkout.

## Task workflow update - 2026-06-30T20:08:30.576Z
- Updated PR Status: merged
- Validation: castor check — OK (quality ok, 180.2s); deptrac — OK (0.8s); test — OK (3932 tests, 12582 assertions, 30.2s); test:controller-replay — OK (8 tests, 112 assertions, 44.9s); test:tui — OK (22 tests, 108 assertions, 64.4s); test:llm-real — OK (9 tests, 110 assertions, 32.5s); phpstan — OK (errors=0, file_errors=0, 5.5s); cs-check — OK (1.9s); llama-proxy cache guard — OK (entries 248 → 248); QA artifact integrity — OK; QA run leak check — OK
- Summary: Post-merge validation on integration checkout completed after DONE merge/worktree cleanup. castor check passed with QA run qa-20260630-200715-235955-c35e949d; llama-proxy cache guard stable and leak check clean.
