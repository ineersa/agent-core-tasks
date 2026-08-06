# SUBAGENT-LIVE-02 Interaction mode and child steering

## Goal
Build on SUBAGENT-LIVE-01 to make live view an explicit child-control mode with safe input routing. Reference POC: `/home/ineersa/projects/agent-core-worktrees/subagent-live-view-steering-poc` at latest known POC head `b5c6fca29`; planning doc `.aiassistant/fork/subagent-live-view-production-plan.md`.

POC evidence/lessons:
- Plain text from live view can be routed to selected child run id as steer/follow_up.
- Steer lands, but it is not interruptive: it applies at command/turn boundaries and cannot stop an in-flight shell/tool call or already emitted tool batch.
- `/new` while live view was active switched sessions underneath live state and stranded the user.
- User confirmed `/agents-main` is useful; Ctrl+\\ works but is awkward on notebooks.
- Product decision: block slash commands only while subagent live view is active, not in main view.

Scope:
- Introduce/centralize an explicit interaction-mode/input-policy concept for subagent live mode.
- In main mode, existing command/input behavior remains normal.
- In subagent live mode, plain text targets selected child run id:
  - active child -> steer;
  - idle/completed child -> follow_up where valid.
- In subagent live mode, block all slash commands except live-view-aware commands (`/agents-main`, `/agents-live`, and optionally `/agents-cancel` if introduced here).
- Blocked commands should produce a clear system/status message: leave live view first with `/agents-main`.
- Provide notebook-friendly return UX: `/agents-main` prominently visible in status/footer; shortcut may remain but must not be the only path.
- Do not implement HITL/cancellation here unless a minimal command stub is needed.

Architecture guidance:
- Avoid scattering string-prefix checks across listeners; prefer a small policy/service if feasible.
- Question overlay/picker ownership must remain higher priority than live steering.
- Keep comments explaining precedence and slash-command blocking rationale.

## Acceptance criteria
- While in main view, slash commands behave normally.
- While in subagent live view, `/agents-main` returns to parent and `/agents-live` can reopen/switch picker.
- While in subagent live view, unrelated slash commands such as `/new`, `/resume`, `/tasks`, `/rename` are blocked and do not mutate parent/session state.
- While in subagent live view, plain text is sent to selected child run id, not parent run id.
- Routing chooses steer vs follow_up based on child activity/terminal state and documents non-interruptive steer semantics.
- Footer/status clearly exposes `/agents-main` as a return path.
- Automated proof covers command blocking and child run-id command targeting.
- Focused Castor validation recorded; no raw vendor/bin commands.

## Workflow metadata
Status: DONE
Branch: task/subagent-live-02-interaction-mode-steering
Worktree: /home/ineersa/projects/agent-core-worktrees/subagent-live-02-interaction-mode-steering
Fork run: edbde3ulhsed
PR URL: https://github.com/ineersa/agent-core/pull/252
PR Status: merged
Started: 2026-07-02T22:04:02.461Z
Completed: 2026-07-03T17:00:09.003Z

## Work log
- Created: 2026-07-01T16:55:59.710Z

## Task workflow update - 2026-07-02T22:04:02.461Z
- Moved TODO → IN-PROGRESS.
- Created branch task/subagent-live-02-interaction-mode-steering.
- Created worktree /home/ineersa/projects/agent-core-worktrees/subagent-live-02-interaction-mode-steering.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/subagent-live-02-interaction-mode-steering.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/subagent-live-02-interaction-mode-steering.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/subagent-live-02-interaction-mode-steering.
- Validation: Read TODO/subagent-live-02-interaction-mode-steering.md; Read .aiassistant/fork/subagent-live-view-production-plan.md; Loaded .agents/skills/task-workflow/SKILL.md; Loaded .agents/skills/testing/SKILL.md; Read tests/AGENTS.md
- Summary: Starting task-start for SUBAGENT-LIVE-02. Loaded task-workflow skill, testing skill, tests/AGENTS.md, and referenced production plan `.aiassistant/fork/subagent-live-view-production-plan.md`. Scope: Phase 2 steering/follow-up routing, explicit input policy, slash-command blocking in subagent live mode, child run-id targeting, and real TmuxHarness replay E2E proof per user instruction.

## Task workflow update - 2026-07-02T22:17:10.678Z
- Recorded fork run: ljdxt7raq6ru
- Summary: Launched implementation fork ljdxt7raq6ru on worktree `/home/ineersa/projects/agent-core-worktrees/subagent-live-02-interaction-mode-steering` with explicit SUBAGENT-LIVE-02 scope: centralized live input policy, child run-id steer/follow_up routing, slash/shell command blocking while live view is active, `/agents-main` return UX, and mandatory TmuxHarness replay E2E proof. Fork instructed to commit changes only, not push/PR/task move/full gate.
- Loaded task-workflow, testing, tests/AGENTS.md, subagents skill, task file, and `.aiassistant/fork/subagent-live-view-production-plan.md`.
- Moved task to IN-PROGRESS and created worktree `/home/ineersa/projects/agent-core-worktrees/subagent-live-02-interaction-mode-steering`.
- Ran three read-only scout subagents. Scout 1 mapped TUI input/command routing: `SubmitListener` is the main insertion point; current flow has no live-view gate before `SubmissionRouter`; `TickPollListener` owns live status; `SubagentLiveViewState` and `SubagentLiveCommandRegistrar` already exist.
- Scout 2 mapped runtime plumbing: `AgentSessionClient::send(runId, UserCommand)` and both Jsonl/InProcess implementations already propagate arbitrary run ids; the only missing decision is SubmitListener choosing child run id and child activity instead of parent run id/activity.
- Scout 3 compared POC b5c6fca29: POC `SubmitListener::dispatchToChildLiveView()` is useful reference, but production must broaden blocking beyond `/new`/`/resume`/`/r` and exclude Phase 3/4 HITL/cancel code from this task. Existing `TuiSubagentLiveViewE2eTest` is the TmuxHarness target to update.

## Task workflow update - 2026-07-02T22:32:37.290Z
- Summary: Reviewer subagent completed read-only review of commit da672e08d with verdict APPROVED (with suggestions). No critical blockers. Focused Castor validation from reviewer: `castor test --filter=SubagentLiveInput` OK (8 tests, 48 assertions), `castor test --filter=SubmitListenerDispatchRuntimeTest` OK (15 tests, 67 assertions), `castor test:tui --filter=TuiSubagentLiveViewE2eTest` OK (1 test, 4 assertions), `castor deptrac` OK, `castor phpstan --path=changed src files` OK, `castor cs-check` OK. Reviewer flagged two user-visible polish items worth fixing before PR despite approved verdict: stale `readonly` copy in `SubagentLivePickerController`, and `SubmitListener::dispatchToChildLiveView()` transient `Sending to subagent...` working message can stick because `lastLiveWorkingMessage` cache is not invalidated. Reviewer also noted PR hygiene: local main is ahead of origin/main by 3 commits, so `origin/main...HEAD` includes an unrelated docs file unless origin/main is synced before PR creation.
- Reviewer approved Phase 2 scope: central `SubagentLiveInputPolicy`, live-view question overlay precedence preserved, normal text routes to selected child run id with steer/follow_up, unrelated slash/shell commands blocked except `/agents-main`/`/main`/`/agents-live`, no Phase 3/4 HITL/cancel/SafeGuard changes.
- Reviewer confirmed architecture/deptrac boundaries are clean: policy lives in `TuiCommand` and avoids `TuiCommand -> TuiRuntime` dependency by accepting bool/primitives; runtime access remains through `AgentSessionClient`.
- Reviewer confirmed tests are appropriate: virtual/in-process tests prove exact child run-id targeting and command blocking; existing TmuxHarness replay test proves real terminal integration for picker/live/block/return flow.
- Reviewer non-blocking recommendations: update remaining stale `readonly` wording in `src/Tui/Picker/SubagentLivePickerController.php`; invalidate `subagentLiveView->lastLiveWorkingMessage` after child dispatch to prevent transient `Sending to subagent...` indicator from sticking; optionally remove dead `SubagentLiveInputPolicy::parse()` and assert no parent transcript echo in steer test.

## Task workflow update - 2026-07-02T22:33:07.899Z
- Recorded fork run: t9u69qaab2ak
- Summary: Launched small review-polish fork t9u69qaab2ak to address reviewer-suggested fixes before PR: update stale `readonly` live-view copy in `SubagentLivePickerController`, invalidate `lastLiveWorkingMessage` after successful child dispatch so `Sending to subagent...` cannot stick, and optionally remove dead policy parse/add no-parent-transcript assertion if tiny. Fork instructed to commit only, not push/PR/task move, and run focused Castor validation.

## Task workflow update - 2026-07-02T22:35:24.146Z
- Summary: Review-polish fork t9u69qaab2ak completed and committed `ff06a2a2d74690171781b1e2858787ba4f1bd733` (`fix(tui): polish subagent live interaction copy`) on top of `da672e08d`. Verified worktree is clean and HEAD is `ff06a2a2d`. Commit changes: 5 files (+8/-13) — updated interactive live-view copy in `SubagentLivePickerController` and `SubagentLiveViewState`, invalidated `lastLiveWorkingMessage` after successful child dispatch in `SubmitListener`, removed dead policy parse method, and added no-parent-transcript assertion. Fork validation: `castor test --filter=SubagentLiveInput` OK (8 tests, 49 assertions), `castor test --filter=SubmitListenerSubagentLiveInputTest` OK (5 tests, 36 assertions), `castor test:tui --filter=TuiSubagentLiveViewE2eTest` OK (1 test, 4 assertions), `castor cs-check` OK, `castor phpstan` OK. Final branch diff vs local main is 9 files (+469/-9). PR hygiene issue remains: `origin/main...HEAD` includes unrelated local-only docs file `...tui-widget-lifecycle-for-transcript-rendering.md`; branch should be rebased onto `origin/main` (requires explicit user approval for history rewrite) or origin/main synced before CODE-REVIEW PR creation.

## Task workflow update - 2026-07-02T22:40:17.091Z
- Summary: User approved push/rebase cleanup for PR hygiene. Rebased task branch `task/subagent-live-02-interaction-mode-steering` in worktree onto `origin/main` with `git rebase --onto origin/main main`, dropping unrelated local-only main/docs commits from the PR diff. Old feature commits `da672e08d` and `ff06a2a2d` were rewritten to `195a68dba` (`feat(tui): route subagent live input to child runs`) and `7aae6b30d` (`fix(tui): polish subagent live interaction copy`). Worktree is clean. `origin/main...HEAD` now contains only the expected 9 task files (+469/-9), no unrelated docs file. No QA rerun after rebase because task-file content is unchanged except removal of unrelated local-only docs from branch diff; previous focused Castor validation remains recorded.

## Task workflow update - 2026-07-02T22:42:25.787Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (81.3s).
- Pushed task/subagent-live-02-interaction-mode-steering to origin.
- branch 'task/subagent-live-02-interaction-mode-steering' set up to track 'origin/task/subagent-live-02-interaction-mode-steering'.
- Created PR: https://github.com/ineersa/agent-core/pull/252
- Validation: Reviewer subagent APPROVED commit da672e08d pre-polish; no blockers. Polish follow-up addressed stale readonly copy and working-message cache issue.; Fork validation before rebase: castor test --filter=SubagentLiveInput OK; castor test --filter=SubmitListenerDispatchRuntimeTest OK; castor test --filter=SubmitListenerSubagentLiveInputTest OK; castor test:tui --filter=TuiSubagentLiveViewE2eTest OK; castor deptrac OK; castor phpstan OK; castor cs-check OK.; After user-approved rebase onto origin/main, task branch HEAD is 7aae6b30d with expected origin/main...HEAD diff of 9 task files (+469/-9).

## Task workflow update - 2026-07-02T23:02:56.563Z
- Summary: Manual test feedback from user on PR #252: (1) blocked-command message `· Leave subagent live view first with /agents-main before running other commands.` works functionally for `/new` and `!pwd` but should be more prominent/colored with theme error or warning; (2) live view still shows `◐ Child agent working...` even when status says selected child is `[completed]` and report is visible; (3) live view TUI feels different from main: no visible steering queue, and child bash tool with sleep did not suggest backgrounding in live view whereas backgrounding works on main. User explicitly says `/new` and `!pwd` worked correctly; need make error more prominent. Treat first two as PR #252 fixes; investigate/record third as likely Phase 3/4 HITL/tool-terminal/queue UX scope unless a small Phase 2 display fix is obvious.

## Task workflow update - 2026-07-02T23:03:00.384Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Returning PR #252 to IN-PROGRESS for manual-test feedback iteration: make blocked live-view command warning more visually prominent, fix stale `Child agent working...` after child completion, and investigate/triage live-view differences around steering queue/backgrounding.

## Task workflow update - 2026-07-02T23:05:31.743Z
- Summary: Additional manual-test feedback: user later observed child status/working line can show `Child agent idle` with `[completed]` after follow-up state changes, so stale `Child agent working...` may be transition-specific but should still be fixed defensively by mapping terminal child statuses to terminal/idle activity. User reiterated `/new` and `!pwd` work functionally and main immediate need is making blocked-command message more prominent. Steering queue absence and child bash sleep/background prompt discrepancy noted; likely triage separately unless small Phase 2 queue display fix is straightforward, while backgrounding prompt is child HITL/tool-terminal integration scope.

## Task workflow update - 2026-07-02T23:06:15.149Z
- Summary: User identified a stronger Phase 2 design issue during manual testing: sending follow_up to a completed child produces child output in live view, but the result is not propagated to the actual parent/main agent as a user-visible/user-message handoff, making completed-child follow_up functionally useless for the parent run. This changes the iteration priority: do not just polish follow_up status; investigate/fix the semantics so terminal-child live-view input either propagates the child response back to the main agent in a useful way, or is intentionally blocked/reframed if safe propagation is out of scope. Active-child steer remains useful.

## Task workflow update - 2026-07-02T23:06:29.074Z
- Summary: Additional UX/lifecycle feedback from user: agents-live needs a way to remove completed/stale agents if there is no automatic deletion. Suggested user-facing affordance: hotkey in `/agents-live` picker/live view to delete/remove an agent entry. This is likely lifecycle/polish scope (Phase 4/5) rather than Phase 2 routing, unless completed-child follow_up semantics require list cleanup/blocking now.

## Task workflow update - 2026-07-02T23:11:23.190Z
- Recorded fork run: 1in8ivh2pijj
- Summary: Launched review-iteration fork 1in8ivh2pijj for PR #252 manual feedback. Scope: make blocked live-view warning prominent with theme warning/error styling; prevent terminal/completed child normal text from sending child follow_up (instead warn and ask user to `/agents-main`); defensively map terminal child statuses to terminal/idle activity so `Child agent working...` cannot stick when status is completed. Explicitly out of scope: parent propagation of completed-child follow_up, child HITL/background prompt bridge, delete-agent hotkey/auto-delete, cancellation/ESC/SafeGuard. Fork will commit only and run focused Castor validation.

## Task workflow update - 2026-07-03T15:50:37.300Z
- Summary: User questioned current decision to block terminal-child followups: `So we won't have followups? well it's not very usable honestly for subagents, more like for workers/forks and with backgrounding`. Interpret as product-direction feedback: active steering remains useful, but completed-child live view without a parent-handoff followup is not very useful for subagents. Need decide whether PR #252 should remain safe active-steering-only, or add a parent-targeted followup mode for terminal children (send user's text to main/parent run with child/artifact context) instead of simply blocking. Also note user sees this feature as more aligned with workers/forks/backgrounding than one-shot subagents.

## Task workflow update - 2026-07-03T16:48:53.769Z
- Validation: reviewer subagent: REQUEST CHANGES; reviewer ran Castor focused checks: SubagentLiveInputPolicyTest OK; SubmitListenerSubagentLiveInputTest OK; TickPollListenerSubagentLiveTest OK; SubmitListenerDispatchRuntimeTest OK; deptrac OK; phpstan --path src/Tui OK; cs-check OK; test:tui --filter=TuiSubagentLiveViewE2eTest flaky FAIL then PASS x5
- Summary: Reviewer re-pass after latest manual-test commit 00f92532b returned REQUEST CHANGES. Production code/semantics/boundaries approved, but reviewer reproduced a flaky TUI E2E: `TuiSubagentLiveViewE2eTest` waits for `/agents-main`, which already appears in persistent live status, so the wait can resolve before warning block renders. Suggested test-only fix: wait for distinctive rendered warning strings (`has finished`, `Leave subagent live view`) instead. Do not move to CODE-REVIEW until fixed and re-reviewed.

## Task workflow update - 2026-07-03T16:49:06.730Z
- Recorded fork run: edbde3ulhsed
- Summary: Launched focused test-only fix fork edbde3ulhsed for reviewer blocker: update `TuiSubagentLiveViewE2eTest` waits to use distinctive warning text instead of `/agents-main`, rerun TmuxHarness E2E repeatedly, commit only, no push/task moves.

## Task workflow update - 2026-07-03T16:51:03.416Z
- Recorded fork run: edbde3ulhsed
- Validation: fork read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before QA; castor test:tui --filter=TuiSubagentLiveViewE2eTest x6: OK (1 test, 5 assertions each); castor cs-check: OK (0 files to fix)
- Summary: Focused test-only fork edbde3ulhsed completed and committed 119f3b736c8b6ee597235aacf5eff84a82cbe47f (`test(tui): wait for subagent live warnings`) on task branch. It changed only `tests/Tui/E2E/TuiSubagentLiveViewE2eTest.php` (+3/-3), replacing flaky waits for `/agents-main` with distinctive warning substrings `has finished` and `Leave subagent live view`. Production code unchanged.

## Task workflow update - 2026-07-03T16:55:03.164Z
- Validation: reviewer re-pass: APPROVED; reviewer validation: castor test:tui --filter=TuiSubagentLiveViewE2eTest OK (1 test, 5 assertions, 5.1s); reviewer validation: castor cs-check OK (0 files fixed); reviewer inspected git show 119f3b736 and confirmed test-only 1-file / 3-line change; reviewer confirmed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` read per AGENTS.md
- Summary: Reviewer re-pass after test-only fix commit 119f3b736 returned APPROVED. Reviewer verified the flake root cause was fixed by waiting for warning-specific text (`has finished`, `Leave subagent live view`) instead of persistent `/agents-main` status text. Production code unchanged from prior approved semantics review. No blockers remain; parent should move task to CODE-REVIEW.

## Task workflow update - 2026-07-03T16:56:44.287Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (86.0s).
- Pushed task/subagent-live-02-interaction-mode-steering to origin.
- branch 'task/subagent-live-02-interaction-mode-steering' set up to track 'origin/task/subagent-live-02-interaction-mode-steering'.
- PR already exists: https://github.com/ineersa/agent-core/pull/252
- Validation: manual user retest: latest behavior seems to be working; reviewer re-pass: APPROVED; fork validation: castor test:tui --filter=TuiSubagentLiveViewE2eTest x6 OK; castor cs-check OK; reviewer validation: castor test:tui --filter=TuiSubagentLiveViewE2eTest OK; castor cs-check OK; previous focused validation on branch: SubagentLiveInputPolicyTest OK; SubmitListenerSubagentLiveInputTest OK; TickPollListenerSubagentLiveTest OK; SubmitListenerDispatchRuntimeTest OK; deptrac OK; phpstan OK; cs-check OK
- Summary: Moved after reviewer APPROVED re-pass. Branch includes active-child steering, live-view command blocking, completed-child orphan followup blocking with prominent warning, terminal status/activity mapping fix, and test-only Tmux wait flake fix. Manual user retest passed. Reviewer confirmed production semantics and boundaries, with no blockers remaining.

## Task workflow update - 2026-07-03T17:00:09.003Z
- Moved CODE-REVIEW → DONE.
- Merged task/subagent-live-02-interaction-mode-steering into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Command/SubagentLiveInputPolicy.php        |  91 ++++++++
 src/Tui/Listener/SubmitListener.php                | 117 ++++++++++-
 src/Tui/Listener/TickPollListener.php              |  14 +-
 src/Tui/Picker/SubagentLivePickerController.php    |   7 +-
 src/Tui/Runtime/SubagentLiveViewState.php          |  11 +-
 tests/Tui/Command/SubagentLiveInputPolicyTest.php  |  54 +++++
 tests/Tui/E2E/TuiSubagentLiveViewE2eTest.php       |  22 +-
 .../Listener/SubmitListenerDispatchRuntimeTest.php |   4 +
 .../SubmitListenerSubagentLiveInputTest.php        | 233 +++++++++++++++++++++
 .../Listener/TickPollListenerSubagentLiveTest.php  |  71 +++++++
 10 files changed, 610 insertions(+), 14 deletions(-)
 create mode 100644 src/Tui/Command/SubagentLiveInputPolicy.php
 create mode 100644 tests/Tui/Command/SubagentLiveInputPolicyTest.php
 create mode 100644 tests/Tui/Listener/SubmitListenerSubagentLiveInputTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/subagent-live-02-interaction-mode-steering.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/subagent-live-02-interaction-mode-steering.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: CODE-REVIEW transition castor check: passed (86.0s); Reviewer re-pass: APPROVED; Manual user retest: latest behavior working; PR #252: user reports merged externally
- Summary: User confirmed PR #252 was merged externally. Moving task to DONE after approved review, deterministic castor check pass during CODE-REVIEW transition, branch push, and external merge.
