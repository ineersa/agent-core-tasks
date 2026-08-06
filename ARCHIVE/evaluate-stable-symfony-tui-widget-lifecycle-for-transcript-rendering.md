# Evaluate stable Symfony TUI widget lifecycle for transcript rendering

## Goal
Future architecture investigation. Current transcript rendering uses Hatfield's project-native TuiWidget/string[] lifecycle and builds transient Symfony widgets as rendering helpers per transcript render. Because Symfony TUI render caching is per widget instance, rebuilding fresh TextWidget/MarkdownWidget/ContainerWidget instances defeats Symfony's internal cache, so RENDER-05 added a project-level rendered-line cache in TranscriptBlockWidget.

This task should evaluate whether transcript rendering should instead own stable per-block Symfony widget instances keyed by TranscriptBlock id and mutate/invalidate only changed blocks, letting Symfony TUI's native widget cache/layout lifecycle do more work.

Questions to answer:
- Can TranscriptBlockWidget keep a stable Symfony ContainerWidget/per-block widget tree without fighting ChatScreen/LiveTextWidget/string[] architecture?
- Would stable Symfony widgets replace or simplify the RENDER-05 rendered-line cache?
- How should streaming blocks, preview expansion, theme changes, terminal resize, hidden/suppressed blocks, and neighbor-aware empty-assistant suppression invalidate widgets?
- Does Symfony MarkdownWidget/TextWidget support clean content mutation, or would stable widgets still need recreation per changed block?
- Can this be done while preserving current compact terminal-native style and glyph contract?
- What tests/benchmarks would prove long-conversation responsiveness improves without broad snapshot churn?

Non-goal for now: do not block current RENDER series. This is an optional future architecture reconsideration if transcript rendering remains a performance/complexity hotspot.

## Acceptance criteria
- Architecture report compares current rendered-line cache approach vs stable Symfony widget lifecycle, with concrete pros/cons and migration risk.
- Prototype or spike demonstrates whether stable per-block Symfony widgets can preserve output equivalence for user/assistant/thinking/tool/question blocks.
- Decision documented: keep rendered-line cache, migrate to stable Symfony widgets, or defer with rationale.
- If migration is recommended, follow-up implementation tasks are proposed with test strategy and deptrac/architecture considerations.
- No current glyph/prefix visual contract changes are required by the proposal.

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
- Created: 2026-07-01T00:01:57.551Z

## Task workflow update - 2026-07-02T21:54:21.614Z
- Validation: No QA run; planning/report-only task.
- Summary: Architecture investigation complete. Report saved at `.pi/reports/evaluate-stable-symfony-tui-widget-lifecycle-for-transcript-rendering.md`. Decision: keep current RENDER-05 rendered-line cache for now; stable Symfony per-block widgets are technically feasible but expected benefits do not outweigh lifecycle complexity, memory/upgrade risk, and current cache effectiveness. Attempted to move task directly from TODO to DONE, but the workflow tool failed (`git status --porcelain failed`) for this no-worktree investigation task after the report was written.

## Task workflow update - 2026-07-02T22:00:33.617Z
- Validation: No QA run; planning/report-only task.
- Summary: Report committed in agent-core as `7eb00f847` (`docs: record Symfony TUI transcript lifecycle decision`). `move_task` was retried after committing the report and still failed with `git status --porcelain failed`; user authorized moving the task file directly for this planning-only closure.

## Task workflow update - 2026-07-05T22:33:09.446Z
- Summary: Follow-up created: TODO/tui-native-symfony-widget-architecture-spike.md. The original evaluation was scoped narrowly to transcript per-block render caching/performance and deferred migration. The new task revisits the broader architecture question: whether Hatfield should lean into first-class Symfony TUI widgets instead of the current hybrid `TuiWidget`/`LiveTextWidget`/offscreen Renderer model, especially after discovering MarkdownWidget sub-element theming only works naturally with Symfony `WidgetContext`.
