# Use Symfony TUI progress bar for status animation

## Goal
Replace Hatfield's hand-built animated status/progress string with Symfony TUI's native progress bar widget now available in the upstream dependency. Reuse the existing status slot and lifecycle; do not add settings, alternate modes, or a new abstraction. Coordinate with the existing native-widget architecture spike only by reusing proven patterns—this task owns the concrete progress/status migration.

## Acceptance criteria
- The active working/status area renders progress and animation through Symfony TUI's native progress bar widget instead of manually assembling the animated status string.
- Existing status visibility, text, start/update/stop lifecycle, layout placement, and theme behavior remain intact unless the native widget necessarily defines rendering.
- Obsolete custom progress/animation rendering code and tests are removed rather than retained as fallback compatibility.
- A focused virtual/in-process TUI regression test proves the real status render/lifecycle path uses the native progress widget and stops/clears correctly.
- castor test, castor test:tui, castor deptrac, castor phpstan, and castor cs-check pass; full castor check remains reserved for CODE-REVIEW transition.

## Workflow metadata
Status: CANCELLED
Branch: task/use-symfony-tui-progress-bar-for-status-animation
Worktree: /home/ineersa/projects/agent-core-worktrees/use-symfony-tui-progress-bar-for-status-animation
Fork run:
PR URL:
PR Status:
Started: 2026-08-13T00:44:35.943Z
Completed:

## Work log
- Created: 2026-08-13T00:44:29.094Z

## Task workflow update - 2026-08-13T00:44:35.943Z
- Moved TODO → IN-PROGRESS.
- Created branch task/use-symfony-tui-progress-bar-for-status-animation.
- Created worktree /home/ineersa/projects/agent-core-worktrees/use-symfony-tui-progress-bar-for-status-animation.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/use-symfony-tui-progress-bar-for-status-animation.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/use-symfony-tui-progress-bar-for-status-animation.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/use-symfony-tui-progress-bar-for-status-animation.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/use-symfony-tui-progress-bar-for-status-animation.
- Summary: Starting the minimal concrete migration requested by the user: replace Hatfield's custom status/progress animation string with Symfony TUI's installed native progress bar widget, preserving the existing status slot and lifecycle with no new settings or fallback implementation.

## Task workflow update - 2026-08-13T00:50:16.180Z
- Moved IN-PROGRESS → CANCELLED.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/use-symfony-tui-progress-bar-for-status-animation.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/use-symfony-tui-progress-bar-for-status-animation.
- Summary: Superseded before implementation: user clarified the correct native component is LoaderWidget with the built-in circle spinner, not ProgressBarWidget/CancellableLoaderWidget. No code changes were made.
