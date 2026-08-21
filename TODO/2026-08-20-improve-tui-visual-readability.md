# Improve TUI visual rhythm and readability

## Goal
The TUI currently feels cramped and is difficult to scan during active agent work. Use the captured ANSI snapshots in `/home/ineersa/projects/agent-core/.hatfield/tmp/tui/snapshots/` as the starting reference, especially the dense handoff, tool-call, subagent, working-status, and chrome regions.

Improve visual hierarchy using Symfony TUI's existing/native styling capabilities and the current theme system. Favor modest whitespace, margins/padding, typography, and alignment over introducing card-style containers.

Specific observations from the snapshots:
- Adjacent content blocks need clearer separation so handoffs, reasoning/status text, tool calls, subagents, and working state are easier to track.
- Collapsed-content text such as `… 113 more lines` can use subdued italic styling where supported.
- Leading glyphs/spinners and tool-call markers appear poorly aligned with their text; improve perceived vertical alignment/rhythm using supported terminal layout/styling (not CSS-style line-height assumptions).
- Preserve information density and the existing single-column architecture.

Out of scope:
- Full card treatment, pervasive backgrounds, or heavy borders around every block.
- A new styling framework, dependency, settings surface, or broad TUI redesign.
- Changing runtime behavior or the information shown.

## Acceptance criteria
- Distinct transcript/runtime blocks have consistent, modest spacing that makes the active flow easier to scan without materially wasting terminal space.
- Collapsed-content indicators (for example, `… 113 more lines`) are visually secondary and italicized when Symfony TUI/terminal capabilities support it.
- Glyphs, spinners, and tool-call markers have consistent spacing/alignment with their associated text and no longer look vertically detached.
- The result uses existing Symfony TUI and Hatfield theme/layout APIs; no card-style backgrounds, pervasive boxed panels, or new dependency/settings surface is introduced.
- The single-column layout and all existing content/actions remain intact across narrow and wide terminal sizes.
- Relevant automated TUI rendering proof is updated at the lowest correct layer, and focused Castor validation passes.
- Before/after ANSI snapshots demonstrate improved readability for the handoff/tool-call/subagent flow represented by the referenced captures.

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
- Created: 2026-08-20T15:45:09+00:00
