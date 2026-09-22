# Show messages queued during compaction with the hourglass icon

## Goal
## Bug

The user reports that submitting the next message while compaction is running does not show it as queued in the UI. Queued steering messages already display an hourglass icon.

## Reproduction

1. Start compaction in a session.
2. Submit another message before compaction finishes.
3. Observe that the UI does not show the message as queued with the hourglass icon.

## Expected behavior

Queue the message and show it immediately using the existing queued steering-message presentation, including the hourglass icon. Process it after compaction finishes.

Keep this a small bug fix. Whether queueing itself fails or only its UI presentation is not yet verified.

## Acceptance criteria
- A message submitted during compaction appears immediately in the UI as queued with the existing hourglass icon.
- After compaction finishes, the queued message is processed once and its pending indicator clears through the existing message lifecycle.
- Existing queued steering-message behavior remains unchanged.

## Workflow metadata
Status: DONE
Branch: task/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/522
PR Status: merged
Started: 2026-09-21T21:00:36+00:00
Completed: 2026-09-21T22:13:06+00:00

## Work log
- Created: 2026-09-09T17:06:12+00:00

## Task workflow update - 2026-09-21T21:00:36+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon/.idea.

## Task workflow update - 2026-09-21T21:07:52+00:00
- Summary: Routing pass found that submission and one-time post-compaction dispatch already work through TuiSessionState::queuedFollowUp and RuntimeEventPoller. The missing behavior is presentation: TickPollListener only synchronizes canonical queuedUserMessages, so it omits the locally held compaction follow-up. Scope is one cohesive TUI slice owned by main, with virtual render proof plus focused submit and poller lifecycle tests.
- Ownership: owner=main; fork_run=none; revision=9b54bf007a9e975c1929afef07ed72c7b3af69b9; scope=show locally queued compaction follow-up in the existing pending-message widget and prove submit, render, dispatch-once, and clear lifecycle; outcome=assigned; commit=none

## Task workflow update - 2026-09-21T21:15:24+00:00
- Validation: castor test --filter='(TickPollListenerTest|SubmitListenerDispatchRuntimeTest|RuntimeEventPollerTest)' passed: 95 tests, 489 assertions.; castor test --filter=testTickShowsAndClearsFollowUpQueuedDuringCompaction passed: 1 test, 5 assertions after adding unchanged-steering coverage.; castor phpstan passed with 0 errors.; castor cs-check passed with 0 files requiring fixes.; Full castor check is deferred to the task-to-pr CODE-REVIEW transition, as required by the workflow.
- Summary: Implemented the compaction queue presentation. SubmitListener now adds a locally held compaction follow-up to the existing pending-message widget immediately. TickPollListener keeps it visible beside canonical queued steering messages until RuntimeEventPoller dispatches the follow-up after compaction. Added virtual and focused lifecycle regression coverage.
- Ownership: owner=main; fork_run=none; revision=9b54bf007a9e975c1929afef07ed72c7b3af69b9; scope=show locally queued compaction follow-up in the existing pending-message widget and prove submit, render, dispatch-once, and clear lifecycle; outcome=completed; commit=c85577cbe

## Task workflow update - 2026-09-21T21:30:39+00:00
- Summary: Independent review approved c85577cbe with suggestions, but identified one low-severity immediate-paint issue: SubmitListener updates the pending widget after its earlier forced frame and otherwise waits for the idle tick cadence. Main will make the compaction queue frame render synchronously, then request re-review.
- Review: role=reviewer; artifact=agent_d452ea16b3aa49e3; revision=c85577cbe; scope=specification fidelity, immediate pending presentation, queue lifecycle, coexistence, cancellation, subagent-live behavior, and proof layer; verdict=APPROVE WITH SUGGESTIONS
- Ownership: owner=main; fork_run=none; revision=c85577cbe; scope=force the newly queued compaction message frame to render immediately after SubmitListener updates the pending widget; outcome=assigned; commit=none

## Task workflow update - 2026-09-21T21:31:41+00:00
- Validation: After the review fix, castor test --filter='(TickPollListenerTest|SubmitListenerDispatchRuntimeTest|RuntimeEventPollerTest)' passed: 95 tests, 491 assertions.; After the review fix, castor phpstan passed with 0 errors.; After the review fix, castor cs-check passed with 0 files requiring fixes.
- Summary: Addressed the review's immediate-paint concern at 4d3643d30 by requesting and processing a TUI render after updating the pending widget and queued status message.
- Ownership: owner=main; fork_run=none; revision=c85577cbe; scope=force the newly queued compaction message frame to render immediately after SubmitListener updates the pending widget; outcome=completed; commit=4d3643d30

## Task workflow update - 2026-09-21T21:36:52+00:00
- Validation: castor test --filter='(TickPollListenerTest|SubmitListenerDispatchRuntimeTest|RuntimeEventPollerTest)' passed: 96 tests, 497 assertions.; castor phpstan passed with 0 errors.; castor cs-check passed with 0 files requiring fixes.
- Summary: Addressed the second review at 56619c565. The immediate queued-message repaint now degrades locally on a transient terminal render failure instead of entering the runtime dispatch failure path. Added a focused fault-injection regression test.
- Ownership: owner=main; fork_run=none; revision=4d3643d30; scope=guard immediate queued-message repaint as non-fatal and prove render failure preserves the queued follow-up and compaction activity; outcome=completed; commit=56619c565

## Task workflow update - 2026-09-21T21:39:58+00:00
- Ownership: owner=main; fork_run=none; revision=4d3643d30; scope=guard immediate queued-message repaint as non-fatal and prove render failure preserves the queued follow-up and compaction activity; outcome=assigned; commit=none
- Ownership: owner=main; fork_run=none; revision=56619c565; scope=add required event-style correlation fields to the new queued-message render degradation log; outcome=assigned; commit=none

## Task workflow update - 2026-09-21T21:40:56+00:00
- Validation: castor test --filter=queuedCompactionMessageSurvivesImmediateRenderFailure passed: 1 test, 6 assertions.; castor phpstan passed with 0 errors.; castor cs-check passed with 0 files requiring fixes.
- Summary: Added run, session, component, and event-type correlation fields to the queued-message render degradation log at 1c8e3dd52.
- Ownership: owner=main; fork_run=none; revision=56619c565; scope=add required event-style correlation fields to the new queued-message render degradation log; outcome=completed; commit=1c8e3dd52

## Task workflow update - 2026-09-21T21:42:09+00:00
- Summary: Reviewer approved revision 1c8e3dd52 with optional suggestions only. No correctness, security, specification-fidelity, or proof blockers remain.
- Review: role=reviewer; artifact=agent_d452ea16b3aa49e3; revision=1c8e3dd52; scope=final specification fidelity, immediate repaint degradation, structured logging, queue lifecycle, and deterministic proof; verdict=APPROVE WITH SUGGESTIONS

## Task workflow update - 2026-09-21T21:44:30+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (125.4s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon/var/reports/qa-20260921-214224-4034-24e93bee.
- Session/run: 63.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-21T21:44:31+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon/var/reports/qa-20260921-214224-4034-24e93bee.
- Session/run: 63.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-21T21:44:35+00:00
- castor check passed (125.4s).
- Pushed task/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon to origin.
- Created PR: <url>
- Session/run: 63.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-21T21:44:35+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (125.4s).
- Pushed task/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/522

## Task workflow update - 2026-09-21T21:57:24+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #522 review comment requests that the queued-message repaint exception not be hidden at debug level. Reopening implementation to replace the hidden degradation with proper exception reporting.

## Task workflow update - 2026-09-21T21:59:33+00:00
- Summary: Accepted PR #522 inline comment as a blocker. The caught repaint exception is currently logged at debug while the default logging threshold is warning, so it is hidden. Scope is to report this degradation at error level with the exception context and prove the log contract while preserving the queued follow-up.
- PR feedback: comment=https://github.com/ineersa/agent-core/pull/522#discussion_r4066746913; classification=blocker; resolution=raise queued-message repaint failure from hidden debug output to structured error reporting with the exception context
- Ownership: owner=main; fork_run=none; revision=1c8e3dd52; scope=report queued-message repaint exceptions at error level and add focused proof without changing queue or dispatch behavior; outcome=assigned; commit=none

## Task workflow update - 2026-09-21T22:00:46+00:00
- Validation: castor test --filter=queuedCompactionMessageSurvivesImmediateRenderFailure passed: 1 test, 9 assertions.; castor phpstan passed with 0 errors.; castor cs-check passed with 0 files requiring fixes.
- Summary: Addressed PR #522 feedback at 1915b605d. A queued-message repaint exception now emits a structured error record with the exception and correlation fields. The focused failure-path test asserts the error level, event message, and exception while confirming the follow-up remains queued.
- Ownership: owner=main; fork_run=none; revision=1c8e3dd52; scope=report queued-message repaint exceptions at error level and add focused proof without changing queue or dispatch behavior; outcome=completed; commit=1915b605d

## Task workflow update - 2026-09-21T22:05:42+00:00
- Summary: Independent reviewer approved PR-feedback revision 1915b605d with optional test-maintenance suggestions only. The hidden-debug concern is resolved and no blocker remains.
- Review: role=reviewer; artifact=agent_6eec75c2fe6504c2; revision=1915b605d; scope=PR #522 exception-reporting feedback, structured logging, queue preservation, deterministic proof, and specification fidelity; verdict=APPROVE WITH SUGGESTIONS

## Task workflow update - 2026-09-21T22:06:55+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (55.9s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon/var/reports/qa-20260921-220559-1263-8e6d4973.
- Session/run: 63.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-21T22:06:57+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon/var/reports/qa-20260921-220559-1263-8e6d4973.
- Session/run: 63.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-21T22:06:57+00:00
- castor check passed (55.9s).
- Pushed task/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon to origin.
- PR already exists: <url>
- Session/run: 63.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-21T22:06:57+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (55.9s).
- Pushed task/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/522

## Task workflow update - 2026-09-21T22:13:06+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Listener/SubmitListener.php                      | 16 ++++++++++++++++
 src/Tui/Listener/TickPollListener.php                    | 12 ++++++++----
 src/Tui/Runtime/TuiSessionState.php                      |  9 ++++-----
 tests/Tui/Listener/SubmitListenerDispatchRuntimeTest.php | 62 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++-----
 tests/Tui/Listener/TickPollListenerTest.php              | 28 ++++++++++++++++++++++++++++
 tests/Tui/Runtime/RuntimeEventPollerTest.php             | 34 ++++++++++++++++++++++++++++++++++
 6 files changed, 147 insertions(+), 14 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-09-show-messages-queued-during-compaction-with-the-hourglass-icon.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #522 merged at 386b23af9636a6bf3eb9812c8d47252a83b79c05. Moving the task to DONE before the required integrated post-merge gate.

## Task workflow update - 2026-09-21T22:14:36+00:00
- Validation: Post-merge castor check passed in 148.0s on integration commit e24edc1bb60ff849fc1457b8cce521f280b32ed7.; QA run qa-20260921-221315-6340-119b040c: all 11 lanes passed, including 4,997 unit/integration tests, controller replay, TUI replay, live LLM smoke, PHPStan, LSP diagnostics, dead-code analysis, CS check, docs validation, Deptrac, and catalog version check.; QA artifact integrity, process/tmux leak checks, exact-run cache cleanup, and llama-proxy cache guard passed.
- Summary: Post-merge validation passed on the integrated revision. The task is complete.
