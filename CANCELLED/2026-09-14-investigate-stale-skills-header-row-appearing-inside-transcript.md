# Investigate stale skills header row appearing inside transcript

## Goal
User screenshot: /home/ineersa/projects/agent-core/.hatfield/sessions/32/attachments/pasted-image-1.png. Parent visually confirmed a stray cyan skills list with clipped magenta 'kills' label between two completed Bash cards, while the normal skills block also appears above the editor. Read-only fork had no vision and used parent's image description.

Source investigation identifies the skills catalog as CompactHeaderWidget, not FooterBarWidget. Likely stale terminal painting/scrollback after transcript or chrome height changes, not yet reproduced. Entry points: src/Tui/CompactHeader/CompactHeaderWidget.php, src/Tui/Screen/ChatScreen.php, src/Tui/Terminal/SynchronizedCursorScreenWriter.php (writeInternal, redrawViewport, overheight shrink guard). redrawViewport preserves scrollback whereas fullRender(clear:true) clears it; this is a candidate mechanism, NOT a confirmed cause. Do not assume clipping originates in truncateToWidth without evidence.

Investigation found running session PHAR hash prefix 7dfc5d5, different from current/global artifacts; verify exact runtime revision when reproducing. Related prior changes PR #477 picker/shrink repaint behavior and PR #491 upstream renderer migration. No code changed or tests run during investigation. Never signal running/root/session-tagged processes.

## Acceptance criteria
- Reproduce the duplicate skills row and identify the triggering frame transition using actual render/terminal evidence; distinguish transcript data duplication from stale terminal output.
- Fix the proven cause without blindly clearing scrollback or reintroducing excessive full repaints.
- Add deterministic regression proof at the lowest layer that models the relevant terminal semantics; use a minimal terminal smoke only where lower layers cannot prove the contract.
- Preserve picker/overlay shrink behavior and correct CompactHeader/editor/footer placement through overheight growth, shrink, and relevant resize transitions.
- Follow testing skill and tests/AGENTS; run targeted Castor checks and full workflow gate before submission.

## Workflow metadata
Status: CANCELLED
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-09-14T00:40:21+00:00

## Task workflow update - 2026-09-14T01:05:15+00:00
- Validation: Real tmux reproduction confirmed by user run and parent read of var/tmp/ghost-skills-investigation/1-history.txt.; Disposable probe uses unique tmux socket and finally cleanup; no production edits.; Earlier 2268 emulator frame checks did not reproduce duplicate visible skills; emulator does not retain scrollback.
- Summary: Deterministic real-tmux probe reproduced stale chrome in scrollback, not exact screenshot's single clipped row. User ran castor --castor-file=var/tmp/ghost-skills-investigation/castor.php probe. 80x24 terminal, frame with 40 transcript rows + 5 chrome rows, shrink to 10 transcript rows + same chrome, then grow to11: screen skills count stays1; history count goes1→2→2. Capture 1-history.txt contains old full skills/agents/editor/footer followed by newly rendered transcript/header. Source shrink path firstChanged10 < viewportTop21 selects redrawViewport using ED2/home while preserving scrollback. Need distinguish this confirmed stale-scrollback mechanism from screenshot trigger; no production fix yet.

## Task workflow update - 2026-09-14T01:12:43+00:00
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Cancelled at user request. Cause unknown; exact screenshot artifact not reproduced. User reports no recurrence across three running sessions. Synthetic shrink probe demonstrated stale scrollback only and does not explain the visible stray row. No production fix made; preserve screenshot/investigation evidence if it recurs.
