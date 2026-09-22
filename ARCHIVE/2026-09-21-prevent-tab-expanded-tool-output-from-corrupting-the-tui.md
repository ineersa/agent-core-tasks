# Prevent tab-expanded tool output from corrupting the TUI

## Goal
A captured `gh pr checks 66187 --repo symfony/symfony` result reproduced the footer and status rows leaking into the transcript.

The tool result contains literal tab characters. `TranscriptToolResultPreviewWidget` sends the text directly to `TextWrapper::wrapTextWithAnsi()`. The width calculation does not account for terminal tab stops, but tmux expands each tab before display. Some rendered lines therefore exceed the 224-column pane, autowrap, and move the hardware cursor. Later footer writes land in transcript rows.

The captured ANSI stream reproduces the corruption in a fresh 224x60 tmux pane. Symfony `ScreenBuffer` renders the same stream without corruption because its model does not expand tabs like tmux. Replacing each tab with one space before replay prevents the corruption. Symfony `TextWidget` and Hatfield `LiveTextWidget` already normalize tabs before wrapping.

Likely fix location: `src/Tui/Transcript/TranscriptToolResultPreviewWidget.php`. Normalize tabs with `AnsiUtils::TAB_WIDTH` before wrapping. Keep the change local unless investigation finds another custom text widget that can emit raw tabs.

Evidence from this session:

- Raw capture: `/tmp/hatfield-pane-1-ansi-20260920-224203.log`
- Reproduction slice: `/tmp/hatfield-blocks-403-518.bin`
- The first corrupt frame is synchronized-output block 518.
- Replaying through block 517 is clean. Adding block 518 reproduces the corruption.
- Removing synchronized-output markers does not change the result, so CSI `?2026` is not the cause.
- The existing Symfony viewport and native-scrollback changes are not the cause.

## Acceptance criteria
- Normalize literal tabs before wrapping transcript tool-result text, using the same `AnsiUtils::TAB_WIDTH` rule as Symfony `TextWidget`.
- Ensure `TranscriptToolResultPreviewWidget` emits no raw tab characters and every rendered row fits the available width.
- Add a deterministic regression based on long, tab-delimited `gh pr checks` output that fails before the fix and proves footer and status rows stay in their assigned rows.
- Confirm a 224x60 tmux replay no longer reproduces the leaked `Working...`, prompt, or agent fragments.
- Run the required focused TUI checks during implementation and pass the full `castor check` gate before review.
- Do not change Symfony `ScreenWriter` or the native scrollback boundary unless new evidence shows a separate renderer defect.

## Workflow metadata
Status: DONE
Branch: task/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/520
PR Status: merged
Started: 2026-09-21T14:47:34+00:00
Completed: 2026-09-21T17:13:35+00:00

## Work log
- Created: 2026-09-21T02:52:14+00:00

## Task workflow update - 2026-09-21T14:47:34+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui/.idea.

## Task workflow update - 2026-09-21T14:49:54+00:00
- Ownership: owner=main; fork_run=none; revision=1d307a3f5; scope=normalize transcript tool-result tabs and add deterministic virtual plus 224x60 tmux regression proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-21T14:59:27+00:00
- Validation: Regression proof without the normalization: castor test:tui --filter=TuiTabDelimitedToolOutputE2eTest failed because tmux inserted extra rows while ScreenBuffer did not.; castor test:tui --filter=TuiTabDelimitedToolOutputE2eTest passed: 1 test, 15 assertions.; castor phpstan passed with 0 errors.; castor cs-check passed with 0 files requiring fixes.; Working tree clean at commit 8166e79d6.
- Summary: Normalized literal tabs in TranscriptToolResultPreviewWidget with AnsiUtils::TAB_WIDTH before wrapping. Added a deterministic 224x60 tmux replay using long gh pr checks-style output. The replay compares Symfony's assigned frame with the real pane and checks tool rows, status, prompt, agent text, and footer anchors.
- Ownership: owner=main; fork_run=none; revision=1d307a3f5; scope=normalize transcript tool-result tabs and add deterministic virtual plus 224x60 tmux regression proof; outcome=completed; commit=8166e79d6

## Task workflow update - 2026-09-21T16:37:27+00:00
- Summary: Independent review of 8166e79d6 requested changes. Reviewer approved the tool-result normalization itself, but asked for correction of the tmux test explanation and fixture evidence, plus normalization in PendingMessagesWidget after finding a second custom wrapper that can emit pasted tabs.
- Review: role=reviewer; artifact=agent_7eae4a212ddb0f16; revision=8166e79d6; scope=specification fidelity, TUI correctness, terminal regression proof, and test determinism; verdict=REQUEST CHANGES

## Task workflow update - 2026-09-21T16:39:08+00:00
- Ownership: owner=main; fork_run=none; revision=8166e79d6; scope=address review by strengthening terminal-overflow fixture, correcting ScreenBuffer rationale, and normalizing pasted tabs in PendingMessagesWidget with focused proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-21T16:42:52+00:00
- Validation: With TranscriptToolResultPreviewWidget normalization removed, castor test:tui --filter=TuiTabDelimitedToolOutputE2eTest failed at the ScreenBuffer-to-tmux frame comparison and showed the long job URL wrapped onto an extra terminal row.; castor test --filter=PendingMessagesWidgetTest passed: 1 test, 11 assertions.; castor test:tui --filter=TuiTabDelimitedToolOutputE2eTest passed: 1 test, 15 assertions.; castor phpstan passed with 0 errors.; castor cs-check passed with 0 files requiring fixes.
- Summary: Addressed review at c8967c5d3. The tmux fixture now forces a pre-fix right-margin wrap, the test explains ScreenBuffer's missing autowrap behavior, and PendingMessagesWidget normalizes tabs from pasted queued messages.
- Ownership: owner=main; fork_run=none; revision=8166e79d6; scope=address review by strengthening terminal-overflow fixture, correcting ScreenBuffer rationale, and normalizing pasted tabs in PendingMessagesWidget with focused proof; outcome=completed; commit=c8967c5d3

## Task workflow update - 2026-09-21T16:46:22+00:00
- Summary: Reviewer approved c8967c5d3 with optional suggestions only. The review verified that the fixture now reaches terminal column 227 before normalization while the wrapping model reports 217 of 220, so the 224x60 tmux replay exercises real right-margin autowrap.
- Review: role=reviewer; artifact=agent_7eae4a212ddb0f16; revision=c8967c5d3; scope=review-requested fixes, specification fidelity, lowest proof layer, and deterministic terminal regression; verdict=APPROVE WITH SUGGESTIONS

## Task workflow update - 2026-09-21T16:48:42+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (124.0s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui/var/reports/qa-20260921-164639-5157-2853e120.
- Session/run: 63.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-21T16:48:45+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui/var/reports/qa-20260921-164639-5157-2853e120.
- Session/run: 63.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-21T16:48:48+00:00
- castor check passed (124.0s).
- Pushed task/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui to origin.
- Created PR: <url>
- Session/run: 63.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-21T16:48:48+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (124.0s).
- Pushed task/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/520

## Task workflow update - 2026-09-21T17:13:35+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Transcript/PendingMessagesWidget.php             |   4 +++-
 src/Tui/Transcript/TranscriptToolResultPreviewWidget.php |   4 +++-
 tests/Tui/E2E/TuiTabDelimitedToolOutputE2eTest.php       | 151 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Transcript/PendingMessagesWidgetTest.php       |  34 ++++++++++++++++++++++++++++++++++
 4 files changed, 191 insertions(+), 2 deletions(-)
 create mode 100644 tests/Tui/E2E/TuiTabDelimitedToolOutputE2eTest.php
 create mode 100644 tests/Tui/Transcript/PendingMessagesWidgetTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-21-prevent-tab-expanded-tool-output-from-corrupting-the-tui.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #520 is merged on GitHub at merge commit 7deb6e80018a724299a912be10de99ad3ec0342e. Moving the tracked task to DONE before the required integrated castor check.

## Task workflow update - 2026-09-21T17:15:03+00:00
- Validation: Post-merge castor check passed in 134.2s for QA run qa-20260921-171344-10342-d1ca31ee.; All 11 lanes passed, including 4,992 unit/integration tests, controller replay, TUI replay, live LLM smoke, PHPStan, LSP diagnostics, dead-code analysis, CS check, and docs validation.; QA artifact integrity, process and tmux leak checks, cache cleanup, and llama-proxy cache guard passed.
- Summary: Integrated validation passed after the DONE merge. The full Castor gate completed successfully in the integration checkout.
