# Scope fork live-view questions, statuses, and cancellation to the selected child

## Goal
Session 37 exposed cross-run state leakage in fork live view.

Evidence:
- Fork `687c45f3-c6f1-52ab-87d3-2f921199f092` (`agent_cbfe23d4b0927cdb`) failed because it reached the configured 1800-second child timeout, not because the background question was answered “No”. That question was already auto-resolved when the command finished; the delayed answer was skipped as resolved.
- Returning from the child live view leaves its child-owned question in the session-global `QuestionCoordinator`, allowing it to appear in main view.
- Late HITL callbacks use `SubagentLiveAttention` to optimistically set a catalog row to `Running` without guarding terminal states, so a failed/cancelled child can appear running again.
- Esc handling prioritizes global question state and can fall through or be dispatched again to parent cancellation. In session 37 the second fork (`c5a3bc75-952d-5403-96de-13eaa8ac3d38`) ended as “Cancelled by parent run” after Esc was used while viewing the first failed fork.

Make child question ownership, picker status transitions, and cancellation routing explicitly run-scoped. Fix the shared lifecycle/input routing rather than adding view-specific visual workarounds.

## Acceptance criteria
- A child-owned question is removed, rejected, or otherwise prevented from appearing in main view after leaving or switching away from that child live view.
- Question answers and cancellation are routed using the owning child `run_id`; stale or already-resolved answers do not affect another child or the parent.
- Late question/HITL callbacks cannot change a failed, cancelled, or completed picker entry back to `running`.
- Esc while viewing a terminal child cannot cancel another active child or the parent run.
- Cancelling a selected active child and cancelling the parent remain distinct actions; one Esc dispatch cannot cascade between scopes.
- The live picker consistently reflects terminal artifact/runtime status when concurrent forks start or finish.
- Add focused regression proof for question migration, terminal-status monotonicity, and cross-child/parent Esc cancellation at the lowest appropriate runtime/TUI layers.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec
Fork run: 8yk7p3r3h3c4
PR URL: https://github.com/ineersa/agent-core/pull/370
PR Status: merged
Started: 2026-08-07T01:05:17.050Z
Completed: 2026-08-12T22:05:55.523Z

## Work log
- Created: 2026-08-06T22:40:11.271Z

## Task workflow update - 2026-08-06T22:44:44.298Z
- Summary: Expanded finalized scope: change the default `agents.subagent_tool_timeout_seconds` from 1800 seconds (30 minutes) to 86400 seconds (24 hours). Keep the timeout configurable and preserve the existing minimum validation; do not add no-timeout support.
- User selected a 24-hour default after session 37 showed a legitimate fork being terminated at the current 30-minute deadline.

## Task workflow update - 2026-08-06T22:45:06.548Z
- Summary: Additional session 37 UX evidence: after pressing Esc while viewing the first failed child, the TUI displayed a “child cancelled” message even though the second fork's artifact records `Cancelled by parent run`. Cancellation confirmation must identify the actual target and scope, and only appear after the corresponding child/parent cancellation is confirmed.
- Recorded mismatch between the displayed “child cancelled” confirmation and persisted cancellation outcome/target.

## Task workflow update - 2026-08-07T01:05:17.051Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec.
- Summary: Claimed for implementation. Finalized scope includes run-scoped child question/cancellation/status routing, accurate cancellation confirmation, focused regression proof, and a configurable 86400-second default subagent timeout.

## Task workflow update - 2026-08-07T01:12:57.376Z
- Recorded fork run: 2io5tghsjdtx
- Summary: Dispatched implementation fork in task worktree. Scope covers run-owned question cleanup/routing, non-cascading Esc cancellation, confirmed target-specific cancellation UX, monotonic picker/runtime status reconciliation, 24-hour configurable default timeout, focused virtual/runtime tests, and mandatory replay-backed TmuxHarness regression proof.
- Scouts traced the root cause to session-global QuestionCoordinator ownership, live-view exit/switch paths not clearing child questions, CancelListener terminal-child fallthrough, optimistic SubagentLiveAttention transitions, and stale catalog/runtime reconciliation. Both scouts confirmed reading root AGENTS.md, testing skill, and tests/AGENTS.md.

## Task workflow update - 2026-08-07T01:16:46.375Z
- Recorded fork run: yc4mnpwsbmol
- Summary: Re-launched implementation as fork yc4mnpwsbmol after fork 2io5tghsjdtx stalled in repeated exploration/handoff generation and produced no edits. Narrowed instructions preserve the same finalized scope and mandatory TmuxHarness proof.
- Fork 2io5tghsjdtx was terminated after looping for several minutes while repeatedly claiming unperformed edits/tests; worktree remained clean. No output from that fork was accepted.

## Task workflow update - 2026-08-07T01:33:47.982Z
- Recorded fork run: yc4mnpwsbmol
- Validation: Fork confirmed it read root AGENTS.md, .agents/skills/testing/SKILL.md, and tests/AGENTS.md before test work.; castor test --filter='QuestionCoordinatorTest|SubagentLiveCatalogTest|SubagentLiveAttentionTest|CancelListenerTest|SubagentLiveHitlScenarioTest|AgentsConfigTest|SubagentToolDefinitionBuilderTest|SubagentLiveToggleInputListenerTest|AgentsMainCommandHandlerTest' — OK (114 tests).; castor test:tui --filter=TuiSubagentChildHitlCancellationE2eTest — OK (2 tests, 8 assertions); real replay-backed TmuxHarness proof added.; castor deptrac — OK, 0 violations.; castor phpstan --path=src/Tui — OK, 0 errors.; castor cs-check — OK after targeted cs-fix.; Full castor check intentionally not run during task-start.
- Summary: Implementation completed and committed as 1d458b9ddadc3a2cf6672c1249c41210609ed86b. Added run-scoped question removal/visible-owner routing, non-cascading child-vs-parent Esc handling, confirmation-based child cancellation status, terminal-monotonic catalog/attention reconciliation, 86400-second default timeout with docs, and focused regressions including a replay-backed TmuxHarness user-path proof. Verified the worktree is clean, commit exists, expected 29 files changed, and the E2E test exercises Ctrl+\\ leave plus false-cancellation assertions.

## Task workflow update - 2026-08-07T01:45:35.094Z
- Summary: Reviewer verdict REQUEST CHANGES. Blocker: leaving a WaitingHuman child removes its question, but cached re-entry replay omits HITL callbacks and childLastSeq skips the original event, so the child can be stranded without an answer overlay. Required cleanup: remove dead SubagentLiveAttention::markCancelledForRun and test; make AgentsMainCommandHandler coordinator/controller dependencies required rather than nullable test-only compatibility; replace test reflection with ChatScreen::registry(). Reviewer confirmed the TmuxHarness proof uses the production Ctrl+\\ path and found no unmapped external surface.
- Reviewer read root AGENTS.md, testing skill, and tests/AGENTS.md; traced callers and applied specification fidelity gate. Verdict: REQUEST CHANGES before PR.

## Task workflow update - 2026-08-07T01:45:57.372Z
- Recorded fork run: 9rd69t7bbh3x
- Summary: Dispatched review-fix fork 9rd69t7bbh3x to restore WaitingHuman question re-enqueue on cached child re-entry, add focused regression proof, remove dead cancellation code, require real AgentsMainCommandHandler dependencies, and remove reflection from the scenario test.

## Task workflow update - 2026-08-07T01:50:36.672Z
- Recorded fork run: 9rd69t7bbh3x
- Validation: castor test --filter='SubagentLivePickerObservationLifecycleTest|SubagentLiveAttentionTest|AgentsMainCommandHandlerTest|SubagentLiveHitlScenarioTest' — OK (22 tests, 117 assertions).; castor test:tui --filter=TuiSubagentChildHitlCancellationE2eTest — OK (2 tests, 8 assertions).; castor deptrac — OK, 0 violations.; castor phpstan --path=src/Tui — OK, 0 errors.; castor cs-check — OK.
- Summary: Review fixes committed as 97fe2c9c7a105e219fccf6a29480ff3fbd3d98a4. Cached WaitingHuman child re-entry now replays HITL callbacks and re-enqueues the run-owned question; focused regression added. Removed dead cancellation API/test, made AgentsMain dependencies required, and removed test reflection. Verified clean worktree and expected 9-file fix commit.

## Task workflow update - 2026-08-07T02:04:09.603Z
- Validation: Reviewer verdict: APPROVED; no blocking correctness, security, architecture, or specification-fidelity findings.; castor test — OK (4402 tests, 16495 assertions).; castor test:tui — OK (35 tests, 223 assertions).; castor deptrac — OK, 0 violations/errors.; castor phpstan — OK, 0 errors.; castor cs-check — OK, 0 files requiring fixes.; Worktree clean at 97fe2c9c7a105e219fccf6a29480ff3fbd3d98a4.
- Summary: Re-review APPROVED at commit 97fe2c9c7a105e219fccf6a29480ff3fbd3d98a4. Reviewer confirmed all prior blockers resolved, specification fidelity satisfied, architecture boundaries preserved, and the virtual plus replay-backed TmuxHarness proof is at the correct layers. Full focused task-to-pr validation passed; worktree remains clean.

## Task workflow update - 2026-08-07T02:06:23.308Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (118.9s).
- Pushed task/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec to origin.
- branch 'task/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec' set up to track 'origin/task/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec'.
- Created PR: https://github.com/ineersa/agent-core/pull/370

## Task workflow update - 2026-08-07T16:18:19.694Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User validation found local `tool_question.requested` events (notably Bash background prompts) are incorrectly mapped to child `WaitingHuman` attention. Fix in PR #370: local tool questions remain answerable in the selected live view but must not mutate child catalog/live activity to canonical WaitingHuman. Add one focused virtual regression; no new setting/API or broader HITL redesign.

## Task workflow update - 2026-08-07T16:29:24.015Z
- Recorded fork run: 8yk7p3r3h3c4
- Validation: Focused Castor tests — OK (84 tests, 400 assertions).; castor deptrac — 0 violations.; castor phpstan --path=src/Tui — 0 errors.; castor cs-check — clean.; Reviewer: REQUEST CHANGES for dead method/tests and unused parameter only; behavioral fix approved.
- Summary: Latest review iteration committed behavioral fix as 80ea98f1049f269da8dc41009abd3a847a272710: local child tool questions remain answerable but no longer own canonical WaitingHuman attention; Tui-source cleanup cannot clear independent AgentCore HITL. Reviewer REQUEST CHANGES only for dead surface left by the fix: delete zero-production-caller markChildNeedsInputForRun and tests, and remove now-unused handleToolQuestionRequested screen parameter/caller argument.

## Task workflow update - 2026-08-07T16:38:52.138Z
- Validation: Reviewer verdict: APPROVED after dead-code cleanup.; castor test — OK (4400 tests, 16498 assertions).; castor test:tui — OK (35 tests, 225 assertions).; castor deptrac — 0 violations/errors.; castor phpstan — 0 errors.; castor cs-check — clean.; Worktree clean at 550e95a3b350845edc29fdabeb04aef6889e5001.
- Summary: User-reported local background-question attention bug fixed in commits 80ea98f1049f269da8dc41009abd3a847a272710 and 550e95a3b350845edc29fdabeb04aef6889e5001. Tui-source tool questions remain answerable and route to the selected child but never own or clear canonical WaitingHuman attention. Dead attention helper/tests and unused screen parameter removed. Re-review APPROVED; worktree clean and two commits ahead of PR branch.

## Task workflow update - 2026-08-07T16:40:58.677Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (116.8s).
- Pushed task/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec to origin.
- branch 'task/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec' set up to track 'origin/task/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec'.
- PR already exists: https://github.com/ineersa/agent-core/pull/370

## Task workflow update - 2026-08-12T22:05:55.524Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec into integration checkout.
- Auto-merging config/hatfield.defaults.yaml
Auto-merging docs/agents.md
Auto-merging docs/settings.md
Auto-merging src/CodingAgent/Resources/skills/subagents/SKILL.md
Merge made by the 'ort' strategy.
 config/hatfield.defaults.yaml                      |   2 +-
 docs/agents.md                                     |   2 +-
 docs/settings.md                                   |   2 +-
 src/CodingAgent/Config/AgentsConfig.php            |   4 +-
 .../Resources/skills/subagents/FRONTMATTER.md      |   2 +-
 .../Resources/skills/subagents/SKILL.md            |   2 +-
 src/Tui/Listener/AgentsMainCommandHandler.php      |  10 ++
 src/Tui/Listener/CancelListener.php                |  59 +++++++---
 src/Tui/Listener/RuntimeQuestionEventHandler.php   |  27 ++---
 src/Tui/Listener/SubagentLiveCommandRegistrar.php  |  18 ++-
 .../Listener/SubagentLiveToggleInputListener.php   |  13 ++-
 src/Tui/Listener/SubmitListener.php                |  11 +-
 src/Tui/Listener/TickPollListener.php              |  48 ++++++--
 src/Tui/Picker/SubagentLivePickerController.php    |  33 +++++-
 src/Tui/Question/QuestionCoordinator.php           |  48 +++++++-
 src/Tui/Runtime/SubagentLiveAttention.php          |  53 ---------
 src/Tui/Runtime/SubagentLiveCatalog.php            |   5 +
 src/Tui/Runtime/SubagentLiveMainReturn.php         |  11 +-
 src/Tui/Runtime/TuiSessionState.php                |  13 +++
 .../Tool/SubagentToolDefinitionBuilderTest.php     |   4 +-
 tests/CodingAgent/Config/AgentsConfigTest.php      |   2 +-
 .../TuiSubagentChildHitlCancellationE2eTest.php    |  88 +++++++++++++++
 .../Tui/Listener/AgentsMainCommandHandlerTest.php  |  14 ++-
 tests/Tui/Listener/CancelListenerTest.php          |  65 ++++++++++-
 .../SubagentLiveToggleInputListenerTest.php        |   6 +-
 .../Tui/Listener/TickPollListenerChildHitlTest.php |  53 +++++++--
 .../SubagentLivePickerObservationLifecycleTest.php | 125 +++++++++++++++++++++
 tests/Tui/Question/QuestionCoordinatorTest.php     |  73 ++++++++++++
 tests/Tui/Runtime/SubagentLiveAttentionTest.php    |  59 +---------
 tests/Tui/Runtime/SubagentLiveCatalogTest.php      |  17 +++
 .../Tui/Scenario/SubagentLiveHitlScenarioTest.php  |  82 +++++++++++++-
 tests/Tui/Support/SubagentLiveScenarioHarness.php  |   7 +-
 32 files changed, 770 insertions(+), 188 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-06-scope-fork-live-view-questions-statuses-and-cancellation-to-the-selec.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #370 merged on GitHub at 2026-08-12T22:05:32Z as b9c06e6f9fcfc8c7d6062ee4911124acb02c6930. Moving task to DONE and syncing integration checkout.

## Task workflow update - 2026-08-12T22:09:32.844Z
- Validation: LLM_MODE=true castor check — deptrac OK; unit OK (4407 tests, 16549 assertions); controller replay OK (12 tests, 165 assertions); TUI OK (35 tests, 231 assertions); llm-real OK (13 tests, 144 assertions); cs-check OK; artifact integrity/leak/cache guard OK; overall nonzero only because phpstan hit the 90s lane timeout.; castor phpstan — OK, 0 errors.; git status — clean (main ahead of origin/main by local integration merges).
- Summary: Post-merge integration validation completed. Full gate's functional, architecture, replay/TUI/live-LLM, style, leak, artifact, and cache-guard lanes passed; phpstan exceeded its 90-second parallel lane budget but passed immediately as a standalone Castor check. Integration checkout is clean.

## Task workflow update - 2026-08-14T19:53:35+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
