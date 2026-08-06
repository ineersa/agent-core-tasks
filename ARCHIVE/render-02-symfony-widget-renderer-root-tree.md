# RENDER-02: Symfony TUI transcript widget renderer

## Goal
Part of `.pi/plans/tui-rich-transcript-blocks-plan.md`, revised for the current TUI architecture.

The current root TUI layout is already `ChatScreen -> LiveTextWidget -> TranscriptBlockWidget`. This task must not introduce a second root layout or alternate transcript stack. Replace the flat transcript rendering internals with a Symfony TUI widget-tree renderer while keeping the existing `ChatScreen` bridge intact.

Order: depends on RENDER-01.

Scope:
- Add `SymfonyTuiWidgetRenderer` as the adapter isolating direct Symfony TUI `Renderer` usage inside `src/Tui/Transcript/`.
- Change `TranscriptBlockWidget::render()` to build one root Symfony TUI widget tree for the whole transcript and render it once through the adapter.
- Add initial `TranscriptBlockCardFactory` or equivalent factory for the currently supported transcript block kinds.
- Port existing readable rendering for all current block kinds to the new widget-tree path: user, assistant, thinking, tool call, tool result, progress, question, approval, cancelled, error, system, and structured subagent result blocks.
- Keep `ChatScreen -> LiveTextWidget -> TuiWidget::render(): string[]` structurally unchanged.

Non-goals:
- The active flat transcript renderer is replaced in this task rather than kept as an alternate renderer.
- `TranscriptEntry` / `TranscriptWidget` are not extended.
- No new root transcript layout outside `TranscriptBlockWidget`.
- No markdown, YAML tool cards, previews, or diff classification yet; those land in RENDER-03 through RENDER-05.

Glyph/testing stability:
- Preserve current transcript block glyphs and prefix spacing for existing block kinds unless the user explicitly approves a visual language change.
- Centralize transcript glyphs/prefixes in the new renderer/factory layer so tests and rendering assertions can reference named semantics instead of scattered raw Unicode.
- RENDER-02 should not force broad snapshot churn; existing snapshots should change only where the new renderer intentionally changes structure.

## Acceptance criteria
- `TranscriptBlockWidget::render()` renders the whole transcript with one root Symfony widget tree per render.
- `SymfonyTuiWidgetRenderer` is the only normal path that invokes Symfony TUI rendering for transcript blocks.
- Direct Symfony TUI API exposure is localized to adapter/factory code under `src/Tui/Transcript/`.
- `ChatScreen` and the `LiveTextWidget` bridge remain structurally unchanged.
- All block kinds currently rendered by `TranscriptBlockRenderer` are ported to the new renderer path in the same task; the old renderer is not retained as an alternate path.
- Existing block glyphs/prefixes remain stable for `UserMessage`, `AssistantMessage`, `AssistantThinking`, `ToolCall`, `ToolResult`, `Progress`, `Question`, `Approval`, `Cancelled`, `Error`, and system severities.
- Focused tests assert glyph behavior through named helper/constant semantics where practical, while preserving at least one explicit default-glyph contract test.
- Structured subagent result rendering remains supported in the new path.
- Focused Castor validation is reported, including `castor test` for transcript rendering tests, `castor phpstan`, and `castor deptrac`.

## Workflow metadata
Status: DONE
Branch: task/render-02-symfony-widget-renderer-root-tree
Worktree: /home/ineersa/projects/agent-core-worktrees/render-02-symfony-widget-renderer-root-tree
Fork run: woxcqh5heupx
PR URL: https://github.com/ineersa/agent-core/pull/239
PR Status: merged
Started: 2026-06-29T21:54:01.432Z
Completed: 2026-06-29T22:56:22.213Z

## Work log
- Created: 2026-05-22T19:08:42.951Z
- Revised: 2026-06-29 — Reframed around the existing `ChatScreen`/`TranscriptBlockWidget` pipeline.
- Revised: 2026-06-29 — Added glyph-stability and test-churn mitigation requirements.

## Task workflow update - 2026-06-29T21:54:01.432Z
- Moved TODO → IN-PROGRESS.
- Created branch task/render-02-symfony-widget-renderer-root-tree.
- Created worktree /home/ineersa/projects/agent-core-worktrees/render-02-symfony-widget-renderer-root-tree.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/render-02-symfony-widget-renderer-root-tree.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/render-02-symfony-widget-renderer-root-tree.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/render-02-symfony-widget-renderer-root-tree.
- Summary: Starting RENDER-02 per user request. Loaded task-workflow/testing context earlier in session and re-read task plus rich transcript plan. Goal is to replace active flat transcript rendering internals with a Symfony TUI root widget tree while preserving ChatScreen/LiveTextWidget bridge, glyph language, and current block behavior.

## Task workflow update - 2026-06-29T22:00:52.860Z
- Recorded fork run: b5bxgrw1mqbk
- Implementation fork b5bxgrw1mqbk dispatched for RENDER-02 with instructions to replace TranscriptBlockWidget/TranscriptBlockRenderer internals with Symfony TUI widget-tree rendering, preserve glyph/prefix contract, add unit + virtual TUI proof, run focused Castor validation, commit changes, and stop before PR/reviewer phase.

## Task workflow update - 2026-06-29T22:11:37.692Z
- Recorded fork run: b5bxgrw1mqbk
- Summary: Implementation fork b5bxgrw1mqbk completed commit 85e60e673 on branch task/render-02-symfony-widget-renderer-root-tree. Verified worktree is clean and diff stat shows 6 files changed: created SymfonyTuiWidgetRenderer and TranscriptBlockWidgetFactory; modified TranscriptBlockRenderer, TranscriptBlockWidget, SubagentResultRenderer, TranscriptBlockRendererTest. Fork validation reported: castor test --filter=TranscriptBlockRendererTest\|SubagentResultRendererTest OK (35 tests, 80 assertions), castor test --filter=Tui OK (886 tests, 2428 assertions), castor deptrac 0 violations, castor phpstan 0 errors, castor cs-check clean. Follow-up required before accepting task-start handoff: implementation instructions required a VirtualTuiHarness/ChatScreen-level proof, but diff stat shows only TranscriptBlockRendererTest changed; dispatching a narrow follow-up fork to add/update virtual proof and audit no-fallback/no-compat wording/path for SubagentResultRenderer.

## Task workflow update - 2026-06-29T22:11:54.245Z
- Recorded fork run: jf34c5d3sae5
- Follow-up fork jf34c5d3sae5 dispatched after initial implementation handoff to add/update the missing VirtualTuiHarness/ChatScreen proof and audit SubagentResultRenderer for no-fallback/no-compatibility compliance.

## Task workflow update - 2026-06-29T22:15:42.939Z
- Recorded fork run: jf34c5d3sae5
- Validation: fork jf34c5d3sae5: castor test --filter=TuiTranscriptBlocksVirtualRenderTest OK (9 tests, 27 assertions); fork jf34c5d3sae5: castor test --filter=TranscriptBlockRendererTest\|SubagentResultRendererTest OK (35 tests, 80 assertions); fork jf34c5d3sae5: castor test --filter=Tui OK (895 tests, 2455 assertions); fork jf34c5d3sae5: castor deptrac OK (0 violations); fork jf34c5d3sae5: castor phpstan OK (0 errors); fork jf34c5d3sae5: castor cs-check clean
- Summary: Follow-up fork jf34c5d3sae5 completed commit e08845a91. Verified worktree clean; branch HEAD has commits 85e60e673 and e08845a91. Combined task diff vs base changes 8 files: created SymfonyTuiWidgetRenderer, TranscriptBlockWidgetFactory, TuiTranscriptBlocksVirtualRenderTest; modified TranscriptBlockRenderer, TranscriptBlockWidget, SubagentResultRenderer, SubagentResultRendererTest, TranscriptBlockRendererTest. Follow-up added 9-test VirtualTuiHarness/ChatScreen proof, removed SubagentResultRenderer::render() flat fallback path, and introduced named glyph constants in TranscriptBlockWidgetFactory. Task-start implementation is complete and ready for user-requested task-to-pr phase.

## Task workflow update - 2026-06-29T22:26:35.900Z
- Summary: Reviewer subagent returned APPROVED for HEAD e08845a91 but listed actionable cleanup findings before PR preparation: remove dead TranscriptBlockRenderer::getSubagentRenderer(), remove unused TranscriptBlockFactory import in TuiTranscriptBlocksVirtualRenderTest, consider color application ANSI byte risk and ensure castor test:tui runs, promote TranscriptBlockWidgetFactory to a constructor field in TranscriptBlockRenderer, tighten or document public helper methods on TranscriptBlockWidgetFactory, remove redundant first render in TranscriptBlockRendererTest::testWidgetUsesDefaultRenderer(), optionally add virtual assertions for streaming suffix and Approval/Question/Cancelled glyphs. Reviewer verified docs/tests and found no architecture/no-fallback blockers. Per user task-to-pr instruction for TUI tasks, follow-up fork will also add a minimal real TmuxHarness replay-backed proof for the user-visible transcript rendering feature before CODE-REVIEW.
- Reviewer subagent completed review of RENDER-02 at e08845a91 with APPROVED verdict plus actionable cleanup items; dispatching fix fork to address sensible findings and satisfy the explicit TmuxHarness E2E proof requirement from the task-to-pr command.

## Task workflow update - 2026-06-29T22:26:55.004Z
- Recorded fork run: tevw6c1qn9ds
- Fix fork tevw6c1qn9ds dispatched for reviewer cleanup findings and explicit minimal TmuxHarness E2E proof requirement before moving RENDER-02 to CODE-REVIEW.

## Task workflow update - 2026-06-29T22:31:59.455Z
- Recorded fork run: tevw6c1qn9ds
- Validation: fork tevw6c1qn9ds: castor cs-check clean; fork tevw6c1qn9ds: castor test --filter=TuiTranscriptBlocksVirtualRenderTest OK (13 tests, 38 assertions); fork tevw6c1qn9ds: castor test --filter=TranscriptBlockRendererTest\|SubagentResultRendererTest OK (35 tests, 80 assertions); fork tevw6c1qn9ds: castor deptrac OK (0 violations); fork tevw6c1qn9ds: castor phpstan OK (0 errors); fork tevw6c1qn9ds: castor test:tui --filter=TuiTranscriptRenderE2eTest OK (1 test, 1 assertion); fork tevw6c1qn9ds: castor test:tui OK (21 tests, 99 assertions)
- Summary: Fix fork tevw6c1qn9ds completed commit 1255eeee6. Verified worktree clean at HEAD. Combined diff vs origin/main now changes 9 files: adds SymfonyTuiWidgetRenderer, TranscriptBlockWidgetFactory, TuiTranscriptBlocksVirtualRenderTest, TuiTranscriptRenderE2eTest; modifies TranscriptBlockRenderer, TranscriptBlockWidget, SubagentResultRenderer, SubagentResultRendererTest, TranscriptBlockRendererTest. Fork addressed reviewer cleanup findings: removed TranscriptBlockRenderer::getSubagentRenderer(), removed unused import, promoted factory to constructor field, made factory helper methods private while keeping glyph constants public, removed redundant test render, added virtual streaming/approval/question/cancelled assertions, and added minimal replay-backed TmuxHarness E2E proof. Fork validation reported all focused commands passed including full castor test:tui.

## Task workflow update - 2026-06-29T22:38:19.775Z
- Summary: Re-reviewer approved HEAD 1255eeee6 and verified the explicit TmuxHarness E2E proof is real replay-backed tmux, not a mock/substitute. Re-review listed only non-blocking design cleanups: clarify TranscriptBlockRenderer role/docblock (it is no longer display-loop-composed), consider resolving/justifying the existing per-block facade, reduce coupling of glyph constants from SubagentResultRenderer to TranscriptBlockWidgetFactory, simplify SymfonyTuiWidgetRenderer constructor/property, and minor NTH items. Per task-to-pr instruction to address sensible actionable findings before CODE-REVIEW, dispatching one final narrow cleanup fork.
- Reviewer subagent re-review at 1255eeee6 returned APPROVED with no blocking issues; final cleanup fork will address/document reasonable non-blocking design findings before local validation and CODE-REVIEW transition.

## Task workflow update - 2026-06-29T22:38:38.310Z
- Recorded fork run: woxcqh5heupx
- Final cleanup fork woxcqh5heupx dispatched to address/document non-blocking reviewer design findings (TranscriptBlockRenderer role/docblock, glyph constant coupling, SymfonyTuiWidgetRenderer simplification) before local validation and CODE-REVIEW transition.

## Task workflow update - 2026-06-29T22:48:52.975Z
- Summary: Final reviewer subagent approved HEAD 12dc67b45. Reviewer verified final cleanup commit is behavior-preserving, TranscriptGlyphs values/autoload are correct, no fallback/compat paths remain, no scope creep into later render tasks, unit+virtual+real TmuxHarness replay-backed proof are appropriate, and architecture/naming/deptrac expectations are satisfied. Only non-blocking NTH: TranscriptBlockRenderer facade is now referenced only by tests but pre-existed and delegates to real production components; acceptable for this PR, possible future cleanup.
- Final reviewer subagent approved RENDER-02 at HEAD 12dc67b45; proceeding to focused local Castor validation before CODE-REVIEW transition.

## Task workflow update - 2026-06-29T22:50:23.678Z
- Validation: local: castor test OK (3912 tests, 12525 assertions, 20.1s); local: castor test:tui OK (21 tests, 99 assertions, 54.0s); local: castor deptrac OK (0 violations, 0 errors); local: castor phpstan OK (0 errors, 0 file errors); local: castor cs-check clean (0 files fixed)
- Summary: Focused local validation completed successfully at HEAD 12dc67b45 after final reviewer approval. Worktree clean before CODE-REVIEW transition.

## Task workflow update - 2026-06-29T22:51:35.604Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (62.8s).
- Pushed task/render-02-symfony-widget-renderer-root-tree to origin.
- branch 'task/render-02-symfony-widget-renderer-root-tree' set up to track 'origin/task/render-02-symfony-widget-renderer-root-tree'.
- Created PR: https://github.com/ineersa/agent-core/pull/239

## Task workflow update - 2026-06-29T22:51:42.786Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/239
- Updated PR Status: open
- Validation: move_task CODE-REVIEW: deterministic castor check passed (62.8s)
- Summary: RENDER-02 moved to CODE-REVIEW. Deterministic move_task castor check passed in 62.8s, branch pushed to origin/task/render-02-symfony-widget-renderer-root-tree, PR #239 created.

## Task workflow update - 2026-06-29T22:56:22.213Z
- Moved CODE-REVIEW → DONE.
- Merged task/render-02-symfony-widget-renderer-root-tree into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Transcript/SubagentResultRenderer.php      |  33 +-
 src/Tui/Transcript/SymfonyTuiWidgetRenderer.php    |  52 +++
 src/Tui/Transcript/TranscriptBlockRenderer.php     | 146 ++------
 src/Tui/Transcript/TranscriptBlockWidget.php       |  25 +-
 .../Transcript/TranscriptBlockWidgetFactory.php    | 171 ++++++++++
 src/Tui/Transcript/TranscriptGlyphs.php            |  55 +++
 tests/Tui/E2E/TuiTranscriptRenderE2eTest.php       | 193 +++++++++++
 .../TuiTranscriptBlocksVirtualRenderTest.php       | 379 +++++++++++++++++++++
 .../Tui/Transcript/SubagentResultRendererTest.php  |   3 +-
 .../Tui/Transcript/TranscriptBlockRendererTest.php |  14 +-
 10 files changed, 909 insertions(+), 162 deletions(-)
 create mode 100644 src/Tui/Transcript/SymfonyTuiWidgetRenderer.php
 create mode 100644 src/Tui/Transcript/TranscriptBlockWidgetFactory.php
 create mode 100644 src/Tui/Transcript/TranscriptGlyphs.php
 create mode 100644 tests/Tui/E2E/TuiTranscriptRenderE2eTest.php
 create mode 100644 tests/Tui/Screen/TuiTranscriptBlocksVirtualRenderTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/render-02-symfony-widget-renderer-root-tree.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/render-02-symfony-widget-renderer-root-tree.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User reported PR #239 was merged. Moving RENDER-02 to DONE; task branch merged/synced into integration checkout and worktree cleanup requested.

## Task workflow update - 2026-06-29T23:00:36.671Z
- Validation: post-merge: castor check OK (190.3s), QA run qa-20260629-225925-509792-66cc4b55; post-merge: deptrac OK (1.0s); post-merge: test OK (3908 tests, 12513 assertions, 30.1s); post-merge: test:controller-replay OK (8 tests, 112 assertions, 53.5s); post-merge: test:tui OK (21 tests, 99 assertions, 62.3s); post-merge: test:llm-real OK (9 tests, 110 assertions, 31.2s); post-merge: phpstan OK (10.3s); post-merge: cs-check OK (2.1s); post-merge: llama-proxy cache guard OK (entries 198 → 198); post-merge: QA artifact integrity OK; leak check OK
- Summary: Post-merge validation completed successfully on integration checkout after RENDER-02 DONE merge.
