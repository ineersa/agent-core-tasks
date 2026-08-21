# Show full ask_human option text with multiline wrapping

## Goal
The TUI presentation for `ask_human` choice questions appears to impose a limited option width and truncates option labels. Users must be able to read every choice in full before selecting it.

Remove width-based text truncation from choice labels. When an option does not fit on one terminal line, wrap it across as many lines as needed within the available TUI width. Preserve the visual association between continuation lines and their option marker/selection state, and keep keyboard navigation/select behavior unchanged.

Use the existing Symfony TUI layout/text-wrapping capabilities and current theme system. Do not add settings, dependencies, horizontal scrolling, or a new choice widget unless the existing implementation cannot support correct wrapping.

## Acceptance criteria
- `ask_human` choice labels are never ellipsized or width-truncated by the TUI; their complete text is visible.
- Options fitting within the available width remain on one line; longer options wrap naturally onto multiple lines.
- Wrapped continuation lines are clearly associated with the correct option and do not collide with option markers, neighboring choices, borders, or surrounding question text.
- The focused/selected style applies coherently to a wrapped option, and existing keyboard navigation, confirmation, cancellation, and returned option value remain unchanged.
- Rendering responds correctly when the terminal is resized and works at representative narrow and wide widths, including Unicode and ANSI-styled labels.
- Automated virtual TUI rendering proof covers full one-line and multiline option labels at constrained widths, and focused Castor validation passes.

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
- Created: 2026-08-20T16:36:38+00:00
