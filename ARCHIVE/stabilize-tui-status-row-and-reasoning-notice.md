# Stabilize TUI status row and clear transient reasoning-level notice

## Goal
Fix two related TUI status-area rendering issues:

1. Status changes such as Working → Idle currently appear to clear/remove the status row before repainting it, causing the layout to jump vertically by one row.
2. After changing the model thinking/reasoning level, the selected level is correctly shown above the status block, but the notice remains indefinitely. It should be transient and disappear when the next turn starts.

Use the existing TUI widget/layout lifecycle rather than adding compatibility paths. Add automated behavior proof at the lowest correct layer, expected to be the virtual/in-process TUI harness unless exploration shows terminal integration is essential.

## Acceptance criteria
- Status transitions repaint in a stable reserved row without a one-row vertical layout jump.
- The reasoning-level notice remains visible after a thinking-level change while idle.
- The reasoning-level notice is cleared when the next turn starts.
- Automated TUI behavior proof exercises the real screen/widget path at the lowest correct layer.
- Focused validation is run through Castor only, following the testing skill and tests/AGENTS.md conventions.

## Workflow metadata
Status: DONE
Branch: task/stabilize-tui-status-row-and-reasoning-notice
Worktree: /home/ineersa/projects/agent-core-worktrees/stabilize-tui-status-row-and-reasoning-notice
Fork run: 8041i6xw3omc
PR URL: https://github.com/ineersa/agent-core/pull/289
PR Status: merged
Started: 2026-07-14T18:50:57.609Z
Completed: 2026-07-15T20:30:32.577Z

## Work log
- Created: 2026-07-14T18:50:51.987Z

## Task workflow update - 2026-07-14T18:50:57.609Z
- Moved TODO → IN-PROGRESS.
- Created branch task/stabilize-tui-status-row-and-reasoning-notice.
- Created worktree /home/ineersa/projects/agent-core-worktrees/stabilize-tui-status-row-and-reasoning-notice.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/stabilize-tui-status-row-and-reasoning-notice.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/stabilize-tui-status-row-and-reasoning-notice.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/stabilize-tui-status-row-and-reasoning-notice.
- Summary: Task created from user-reported TUI rendering issues. Testing skill and tests/AGENTS.md were read before implementation planning.

## Task workflow update - 2026-07-14T18:59:31.779Z
- Recorded fork run: jwjnmfxqcae6
- Summary: Implementation delegated to fork jwjnmfxqcae6 in the task worktree. Scope includes stable status-row reservation, clearing the transient reasoning notice on next-turn submission, and virtual/in-process behavior proof. Full castor check is deferred to task-to-pr.

## Task workflow update - 2026-07-14T19:02:27.899Z
- Recorded fork run: jwjnmfxqcae6
- Validation: castor test --filter='ChatScreenStatusRowVirtualRenderTest|SubmitListenerReasoningNoticeClearTest|TuiStartupVirtualRenderTest' — OK (5 tests, 29 assertions); castor cs-check — OK (0 fixable issues); Verified task worktree is clean and HEAD is 8f1737408
- Summary: Implementation completed and committed as 8f1737408. ChatScreen now reserves exactly one blank row when the working indicator is hidden, preventing downstream layout movement. SubmitListener clears the transient panel-only reasoning notice when dispatching the next turn while preserving the selected footer/editor reasoning state. Added virtual ChatScreen row-position proof and SubmitListener reasoning-notice regression proof. Fork confirmed it read AGENTS.md, the testing skill, and tests/AGENTS.md.

## Task workflow update - 2026-07-14T19:52:50.595Z
- Recorded fork run: 6c464m2es7jg
- Summary: PR-phase reviewer returned REQUEST CHANGES on commit 8f1737408. Blocker: latest user instruction requires a real replay-backed TmuxHarness E2E proof; virtual tests alone are insufficient for this review. Additional actionable fixes: remove unnecessary ReflectionProperty use, name the hidden-row spacer constant, and strengthen anchor-position proof where non-brittle. Fixes delegated to fork 6c464m2es7jg.
- Reviewer pass on 8f1737408: REQUEST CHANGES. Production behavior judged sound; missing required tmux E2E proof is blocking. Follow-up implementation fork launched.

## Task workflow update - 2026-07-14T19:57:15.862Z
- Recorded fork run: cbe641sojihi
- Summary: Reviewer-fix fork 6c464m2es7jg ended with an incomplete handoff and left uncommitted edits. Recovery/completion delegated to fork cbe641sojihi with instructions to inspect, validate, correct, and commit the existing draft tmux proof and reviewer fixes.
- Fork 6c464m2es7jg produced draft reviewer fixes but no commit or complete validation handoff; recovery fork cbe641sojihi launched.

## Task workflow update - 2026-07-14T20:10:00.449Z
- Recorded fork run: 01urygyjgs5i
- Summary: Second reviewer pass on 6d401fbc0 returned APPROVE WITH SUGGESTIONS. Previous blocker is nominally addressed, but reviewer found the tmux layout-gap assertion vacuous because it measures the footer separator rather than an editor/status boundary. A follow-up fork was launched to make the real-terminal row-stability proof meaningful and address dead code/convention/cleanup findings.
- Reviewer pass on 6d401fbc0: APPROVE WITH SUGGESTIONS. Required iteration: correct vacuous tmux row-stability assertion, remove dead parameter, import Yaml, clean isolated directory, document intentional cleanup catch. Fork 01urygyjgs5i launched.

## Task workflow update - 2026-07-14T20:21:54.495Z
- Recorded fork run: bd2g0qh5e6aw
- Summary: Third reviewer pass on 58881062f returned APPROVE WITH SUGGESTIONS. Tmux proof was judged material and valid. Final actionable iteration delegated: rename the misleading virtual footer-separator helper and move reasoning-notice clearing after pasted-image validation so an aborted submission does not clear the notice; add focused regression proof.
- Reviewer pass on 58881062f: APPROVE WITH SUGGESTIONS; no blockers. Final sensible fixes delegated to fork bd2g0qh5e6aw before requesting plain APPROVED.

## Task workflow update - 2026-07-14T20:35:35.130Z
- Recorded fork run: 8041i6xw3omc
- Summary: Fourth reviewer pass on fd7cd5849 found no issues but returned APPROVE WITH SUGGESTIONS for two documentation clarifications. Final comment-only fork 8041i6xw3omc launched; explicitly non-actionable NTH items are excluded.
- Reviewer pass on fd7cd5849: APPROVE WITH SUGGESTIONS, no critical/issues/security findings. Two precise comment clarifications delegated to fork 8041i6xw3omc.

## Task workflow update - 2026-07-14T20:44:01.635Z
- Validation: castor test — OK (4393 tests, 14573 assertions); castor deptrac — OK (0 violations, 0 errors); castor phpstan — OK (0 errors); castor cs-check — OK (0 files fixable); castor test:tui — OK (37 tests, 193 assertions); Reviewer — APPROVED at 60968e061
- Summary: Final reviewer verdict: APPROVED at HEAD 60968e061. Production fix reserves the hidden working row and clears the transient reasoning notice only after pre-dispatch validation. Virtual, listener, and replay-backed TmuxHarness coverage are present. All reviewer iterations completed; no actionable findings remain.
- Implementation commits: 8f1737408, 6d401fbc0, 58881062f, fd7cd5849, 60968e061. Final reviewer pass: APPROVED. Focused Castor validation fully green.

## Task workflow update - 2026-07-14T20:46:06.953Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (113.6s).
- Pushed task/stabilize-tui-status-row-and-reasoning-notice to origin.
- branch 'task/stabilize-tui-status-row-and-reasoning-notice' set up to track 'origin/task/stabilize-tui-status-row-and-reasoning-notice'.
- Created PR: https://github.com/ineersa/agent-core/pull/289
- Validation: castor test — OK (4393 tests, 14573 assertions); castor deptrac — OK; castor phpstan — OK; castor cs-check — OK; castor test:tui — OK (37 tests, 193 assertions); Reviewer — APPROVED
- Summary: Reviewer APPROVED at 60968e061. Focused Castor validation passed: test, deptrac, phpstan, cs-check, and full replay-backed test:tui. Ready for deterministic castor check gate, push, and PR creation.

## Task workflow update - 2026-07-14T20:46:11.211Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/289
- Updated PR Status: open
- Validation: move_task deterministic castor check — PASSED (113.6s)
- Summary: Moved to CODE-REVIEW after deterministic castor check passed in 113.6s. Branch pushed and PR #289 created.
- PR created: https://github.com/ineersa/agent-core/pull/289

## Task workflow update - 2026-07-15T20:30:32.577Z
- Moved CODE-REVIEW → DONE.
- Merged task/stabilize-tui-status-row-and-reasoning-notice into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Listener/SubmitListener.php                |  44 +--
 src/Tui/Screen/ChatScreen.php                      |  34 ++-
 .../Tui/E2E/TuiStatusRowReasoningNoticeE2eTest.php | 309 +++++++++++++++++++++
 .../SubmitListenerReasoningNoticeClearTest.php     | 160 +++++++++++
 .../ChatScreenStatusRowVirtualRenderTest.php       |  98 +++++++
 5 files changed, 621 insertions(+), 24 deletions(-)
 create mode 100644 tests/Tui/E2E/TuiStatusRowReasoningNoticeE2eTest.php
 create mode 100644 tests/Tui/Listener/SubmitListenerReasoningNoticeClearTest.php
 create mode 100644 tests/Tui/Screen/ChatScreenStatusRowVirtualRenderTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/stabilize-tui-status-row-and-reasoning-notice.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/stabilize-tui-status-row-and-reasoning-notice.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #289 confirmed merged on GitHub at 2026-07-15T20:30:08Z with merge commit aec2b341d82860b4bbec8100bac29e3db3bc38a1. Moving task to DONE, syncing integration checkout, and cleaning its worktree.

## Task workflow update - 2026-07-15T20:32:49.819Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/289
- Updated PR Status: merged
- Validation: PR #289 — MERGED at 2026-07-15T20:30:08Z; Post-merge LLM_MODE=true castor check — PASS (270.8s); castor test lane — PASS (4417 tests, 14634 assertions); castor test:controller-replay lane — PASS (8 tests, 112 assertions); castor test:tui lane — PASS (37 tests, 193 assertions); castor test:llm-real lane — PASS (10 tests, 122 assertions); deptrac/phpstan/cs-check — PASS; llama-proxy cache guard and QA leak check — PASS; Task worktree directory — removed; Integration checkout — clean
- Summary: Task completed. PR #289 merged as aec2b341d82860b4bbec8100bac29e3db3bc38a1, integration checkout synced, task worktree fully removed, and post-merge deterministic validation passed.
