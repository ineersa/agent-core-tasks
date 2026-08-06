# Enforce specification fidelity and minimal implementation in agent workflow

## Goal
Add concise repository instructions preventing agents from inventing product surface or overengineering. Apply the principle in root AGENTS.md and enforce it procedurally in the task-workflow skill for fork preparation and review.

## Acceptance criteria
- AGENTS.md states that finalized task requirements are authoritative and forbids unrequested settings, APIs, storage fields, commands, user-visible behavior, speculative configuration, compatibility paths, abstractions, helpers, extensibility, and future-proofing.
- AGENTS.md requires the smallest solution using existing code/platform capabilities, while allowing only minimal indirection required by existing architecture boundaries.
- Ambiguity affecting behavior or public surface must be raised to the user rather than resolved by implementation.
- task-workflow requires fork instructions to map externally visible additions to exact finalized requirements and forbids introducing uncited product decisions.
- task-workflow requires reviewers to return REQUEST CHANGES for unmapped functionality or unnecessary complexity.
- Keep the wording concise and avoid duplicating broader Ponytail guidance.

## Workflow metadata
Status: DONE
Branch: task/enforce-specification-fidelity-minimality
Worktree: /home/ineersa/projects/agent-core-worktrees/enforce-specification-fidelity-minimality
Fork run: 4alf6hpxkyq8
PR URL: https://github.com/ineersa/agent-core/pull/329
PR Status: merged
Started: 2026-07-28T01:58:15.897Z
Completed: 2026-07-28T15:11:28.263Z

## Work log
- Created: 2026-07-28T01:58:09.765Z

## Task workflow update - 2026-07-28T01:58:15.897Z
- Moved TODO → IN-PROGRESS.
- Created branch task/enforce-specification-fidelity-minimality.
- Created worktree /home/ineersa/projects/agent-core-worktrees/enforce-specification-fidelity-minimality.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/enforce-specification-fidelity-minimality.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/enforce-specification-fidelity-minimality.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/enforce-specification-fidelity-minimality.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/enforce-specification-fidelity-minimality.
- Summary: User approved adding specification-fidelity and anti-overengineering rules to AGENTS.md plus fork/reviewer enforcement in task-workflow.

## Task workflow update - 2026-07-28T01:58:34.824Z
- Recorded fork run: 26y9idw04k4k
- Delegated exact two-file instruction update to fork 26y9idw04k4k: concise AGENTS.md principle plus task-workflow fork/reviewer enforcement, with no extra process or code changes.

## Task workflow update - 2026-07-28T02:00:04.480Z
- Recorded fork run: 26y9idw04k4k
- Validation: PASS: git diff --check; PASS: manual diff verification — only AGENTS.md and .agents/skills/task-workflow/SKILL.md changed; No Castor QA needed for Markdown-only workflow instructions
- Summary: Implemented and committed concise specification-fidelity/minimality rules at cd2fe357ecd80c3f4b677d5652e4c49a59afacfa. AGENTS.md now forbids unrequested product surface and speculative overengineering; task-workflow now requires exact requirement mapping before forks and blocks unmapped/unnecessary changes in review. Exactly two Markdown files changed; worktree clean.

## Task workflow update - 2026-07-28T02:06:09.476Z
- Validation: FAIL first castor check: unrelated unit timing, TUI reasoning notice, and live follow-up tests; PASS focused MessengerSqliteImmediateTransactionMiddlewareTest — 4 tests, 19 assertions; PASS focused ShellFollowUpLiveE2eTest — 2 tests, 21 assertions; PASS integration-main focused TuiStatusRowReasoningNoticeE2eTest — 1 test, 7 assertions; FAIL worktree focused TuiStatusRowReasoningNoticeE2eTest — reasoning notice timing; PASS castor clean:cleanup:workers:list — no stale candidates
- Summary: Direct PR transition (review intentionally skipped by user) was blocked by three unrelated gate tests: SQLite timing contention, TUI Shift+Tab reasoning notice timing, and live shell follow-up completion. No stale workers found. Focused SQLite and live follow-up reruns passed; the TUI test passes on integration main but remains flaky/failing in this worktree. No task code changes made.

## Task workflow update - 2026-07-28T02:35:03.763Z
- Recorded fork run: yoz9ys8b4u9j
- Validation: PASS focused castor test:tui --filter=TuiStatusRowReasoningNoticeE2eTest — 1 test, 7 assertions; PASS full castor test:tui — 37 tests, 190 assertions; PASS git diff --check; PASS castor clean:cleanup:workers:list — no stale candidates; Testing skill and tests/AGENTS.md read and followed; Reviewer skipped by explicit user request
- Summary: Fixed the gate blocker at 733132f5667e058de50aa17937457eb94c727d15. Root cause was a false-positive TUI wait: free-text `reasoning` matched the test temp path and `minimal` matched this branch's `minimality`, so the wait returned before the actual status row appeared. The test now waits for the exact status-row regex already asserted. No production behavior changed. User requested direct PR without review.

## Task workflow update - 2026-07-28T13:49:50.859Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (114.0s).
- Pushed task/enforce-specification-fidelity-minimality to origin.
- branch 'task/enforce-specification-fidelity-minimality' set up to track 'origin/task/enforce-specification-fidelity-minimality'.
- Created PR: https://github.com/ineersa/agent-core/pull/329
- Validation: Focused TUI reasoning test: PASS (1 test, 7 assertions); Full castor test:tui: PASS (37 tests, 190 assertions); git diff --check: PASS; No stale QA workers; Reviewer skipped by explicit user request
- Summary: Direct PR transition per user request, without reviewer. Includes concise specification-fidelity/minimality workflow rules and the minimal root-cause fix for the branch-name-sensitive false-positive TUI wait that blocked the gate.

## Task workflow update - 2026-07-28T15:05:47.578Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Addressing owner inline comment 3666684394: remove branch-specific explanatory wording and use an exact line-anchored status-row check rather than a branch/CWD-oriented comment.

## Task workflow update - 2026-07-28T15:07:17.231Z
- Recorded fork run: 4alf6hpxkyq8
- Validation: PASS castor clean:cleanup:workers:list — no stale candidates; PASS focused castor test:tui --filter=TuiStatusRowReasoningNoticeE2eTest — 1 test, 7 assertions; PASS git diff --check; Testing skill and tests/AGENTS.md read and followed; Reviewer skipped per user's earlier direct-PR instruction
- Summary: Addressed owner inline comment 3666684394 at da7f2fd3159f37eed8b6f62798336bf75226e915. Removed branch/CWD-specific commentary and tightened both wait and assertion to the exact line-anchored `reasoning minimal` status-row contract. No branch/path logic, production change, helper, timeout, or unrelated edit.

## Task workflow update - 2026-07-28T15:09:22.854Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (115.8s).
- Pushed task/enforce-specification-fidelity-minimality to origin.
- branch 'task/enforce-specification-fidelity-minimality' set up to track 'origin/task/enforce-specification-fidelity-minimality'.
- PR already exists: https://github.com/ineersa/agent-core/pull/329
- Validation: Focused TUI reasoning test: PASS (1 test, 7 assertions); git diff --check: PASS; No stale QA workers; Reviewer skipped per explicit user instruction
- Summary: Owner inline comment addressed: exact line-anchored status-row matching now replaces branch/CWD-oriented wording and loose capture matching. Returning directly to CODE-REVIEW without reviewer per user's prior instruction.

## Task workflow update - 2026-07-28T15:11:28.263Z
- Moved CODE-REVIEW → DONE.
- Merged task/enforce-specification-fidelity-minimality into integration checkout.
- Merge made by the 'ort' strategy.
 .agents/skills/task-workflow/SKILL.md              | 26 ++++++++++++++++------
 AGENTS.md                                          |  9 ++++++++
 .../Tui/E2E/TuiStatusRowReasoningNoticeE2eTest.php |  4 ++--
 3 files changed, 30 insertions(+), 9 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/enforce-specification-fidelity-minimality.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/enforce-specification-fidelity-minimality.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #329 state: MERGED; Deterministic castor check passed before final push; Focused TUI reasoning test passed (1 test, 7 assertions); Full TUI suite passed (37 tests, 190 assertions)
- Summary: PR #329 merged on GitHub at 846b36a39a09a3681933437a57169d52eb0ceeab. Specification-fidelity/minimality rules and exact TUI status-row test fix completed.

## Task workflow update - 2026-07-28T15:13:53.112Z
- Validation: Post-merge LLM_MODE=true castor check: TUI 37/190 PASS, llm-real 13/175 PASS, controller replay 10/135 PASS, Deptrac/PHPStan/CS PASS, cache/artifact/leak guards PASS; unit lane had one unrelated transient picker export failure; Focused castor test --filter=SubagentLivePickerControllerTest: PASS (17 tests, 56 assertions)
- Summary: Post-merge validation completed. Full LLM_MODE castor check passed all lanes except one unrelated transient SubagentLivePickerControllerTest export-fixture failure; focused class rerun passed 17 tests / 56 assertions. TUI lane, live LLM lane, controller replay, Deptrac, PHPStan, CS, cache guard, artifact integrity, and leak check all passed.
