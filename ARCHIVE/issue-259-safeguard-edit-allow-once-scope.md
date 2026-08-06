# Issue #259: Fix SafeGuard edit allow-once scope for outside-CWD files

## Goal
GitHub issue: https://github.com/ineersa/agent-core/issues/259

Open issue: edits to the same file outside CWD ask for approval the first time, but a second edit can pass without approval even when the user selected allow-once. This suggests SafeGuard approval caching/scoping is too broad for outside-CWD edit operations.

Before implementation, load the relevant extension docs per project guard because this touches the safe-guard extension.

## Acceptance criteria
- Outside-CWD edit approvals with allow-once authorize only the single intended operation and do not silently authorize subsequent edits to the same file.
- Persistent/allow-always behavior, if available, remains distinct and intentionally scoped.
- Regression tests cover first edit approval followed by a second same-file outside-CWD edit requiring approval again.
- Audit/logging or user-facing decision metadata remains clear enough to explain why an edit is allowed or blocked.
- GitHub issue #259 is updated or closed with validation evidence.

## Workflow metadata
Status: ARCHIVE
Branch: task/issue-259-safeguard-edit-allow-once-scope
Worktree: /home/ineersa/projects/agent-core-worktrees/issue-259-safeguard-edit-allow-once-scope
Fork run: x5zjk3oh4954
PR URL: https://github.com/ineersa/agent-core/pull/307
PR Status: merged
Started: 2026-07-20T23:56:14.340Z
Completed: 2026-07-21T02:28:28.553Z

## Work log
- Created: 2026-07-07T18:08:35.713Z

## Task workflow update - 2026-07-20T23:56:14.341Z
- Moved TODO → IN-PROGRESS.
- Created branch task/issue-259-safeguard-edit-allow-once-scope.
- Created worktree /home/ineersa/projects/agent-core-worktrees/issue-259-safeguard-edit-allow-once-scope.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/issue-259-safeguard-edit-allow-once-scope.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/issue-259-safeguard-edit-allow-once-scope.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/issue-259-safeguard-edit-allow-once-scope.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/issue-259-safeguard-edit-allow-once-scope.
- Summary: Starting Issue #259 against current main after canonical HITL approval unification. First verify whether exact-call continuation already fixes the reported broad allow-once leak; implement the smallest remaining regression/fix only. SafeGuard extension docs and testing conventions are mandatory before edits.

## Task workflow update - 2026-07-21T00:05:38.237Z
- Validation: Scout read testing skill and tests/AGENTS; existing focused SafeGuard/subscriber/HITL tests and controller replay are green.; Selected lowest correct proof: extend SafeGuardApprovalControllerReplayTest with two sequential outside-CWD edit calls to the same file, distinct call IDs; Allow first, require a second independent human_input, Deny second, verify only first patch applied.
- Summary: Read-only audit completed after PR #305 merge. Normal distinct tool-call flow is already exact-call scoped: SafeGuard has no approval cache/tracker or interactive Always-allow persistence; Allow is attached only to the stored ExecuteToolCall and a later edit is classified afresh. Issue #259 remains open with no comments/linked PR. Implementation should be regression-only unless the exact two-edit controller replay disproves current semantics.
- Scope decision: do not broaden Issue #259 into generic provider duplicate tool_call_id/result-store hardening or symlink path-boundary changes; those are separate risks and not the reported normal distinct-call reproduction.
- SafeGuard extension docs loaded per guard: ai-index.json plus safe-guard settings.md and maintenance.md. Upstream docs describe interactive allowlists, but current Hatfield PR #305 intentionally removed interactive Always-allow persistence and retains only static settings allowlists.
- GitHub researcher: issue #259 is open, body only, zero comments/links; PR #305 likely resolved root behavior but lacks exact two-edit regression. Close/update issue after regression PR validation, during task-to-PR/DONE workflow.

## Task workflow update - 2026-07-21T00:06:07.030Z
- Recorded fork run: x5zjk3oh4954
- Summary: Implementation fork launched in task worktree. Scope is exact controller-replay regression for two distinct sequential outside-CWD edit calls to one file; regression-only if current PR #305 semantics pass. No generic duplicate-ID/path/TUI expansion, no reviewer/gate/push/issue mutation.

## Task workflow update - 2026-07-21T00:17:07.385Z
- Recorded fork run: x5zjk3oh4954
- Validation: Fork read `.agents/skills/testing/SKILL.md`, `tests/AGENTS.md`, task file, and required SafeGuard ai-index/settings/maintenance docs.; castor test:controller-replay: OK 9 tests / 127 assertions.; castor deptrac: 0 violations.; castor phpstan: 0 errors.; castor cs-check: clean.
- Summary: Implementation complete at ba0c4e7fb16884c9422cf624b1817caefcdec2b1. Added one controller-replay regression in SafeGuardApprovalControllerReplayTest proving two distinct sequential outside-CWD edit calls to the same file receive independent canonical approvals: Allow first applies only first patch; second emits a distinct human_input.requested/question/tool-call and Deny leaves first-edit content unchanged. Current PR #305 production semantics passed immediately; no production change was needed. Worktree clean, one file +185 lines.
- Regression collector detail: after first Allow, one collector waits for the second human_input.requested because stopping at the intermediate tool completion can destructively drain the following HITL event from the same stdout buffer.
- Issue #259 remains open by design; update/close it with PR/validation evidence during task-to-PR or DONE. No reviewer, full castor check, push, or PR performed in task-start phase.

## Task workflow update - 2026-07-21T00:31:52.814Z
- Summary: Task-to-PR reviewer verdict at ba0c4e7fb: APPROVE WITH SUGGESTIONS. No critical, blocking, security, or actionable findings. Reviewer confirmed regression fidelity, valid edit fixtures, correct destructive-collector handling, normal distinct tool_call_id semantics, test isolation, and that no production change is required after PR #305. Two NTH-only suggestions: explicit run.failed absence assertion and reducing one 30s replay timeout; neither changes the proven contract or blocks PR.
- Reviewer read root AGENTS.md, testing skill, tests/AGENTS.md, and task file before review. +185 lines judged justified for the multi-phase security-boundary controller E2E; one file only.

## Task workflow update - 2026-07-21T00:35:01.049Z
- Validation: castor test: OK 4443 tests / 15242 assertions.; castor test:controller-replay: OK 9 tests / 127 assertions.; castor deptrac: 0 violations / 0 errors.; castor phpstan: 0 errors.; castor cs-check: clean.; git status: clean.
- Summary: Task-to-PR focused validation completed on clean worktree at ba0c4e7fb. Reviewer verdict APPROVE WITH SUGGESTIONS with no actionable/blocking findings; only two NTH suggestions. Ready for deterministic CODE-REVIEW gate and PR.

## Task workflow update - 2026-07-21T00:38:41.484Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (123.6s).
- Pushed task/issue-259-safeguard-edit-allow-once-scope to origin.
- branch 'task/issue-259-safeguard-edit-allow-once-scope' set up to track 'origin/task/issue-259-safeguard-edit-allow-once-scope'.
- Created PR: https://github.com/ineersa/agent-core/pull/307
- Validation: Reviewer: APPROVE WITH SUGGESTIONS (NTH-only; no blockers/actionable findings).; castor test: OK 4443 tests / 15242 assertions.; castor test:controller-replay: OK 9 tests / 127 assertions.; castor deptrac: 0 violations.; castor phpstan: 0 errors.; castor cs-check: clean.; Worktree clean at ba0c4e7fb16884c9422cf624b1817caefcdec2b1.
- Summary: Regression-only implementation ready for review at ba0c4e7fb. PR #305 already fixed the production behavior; this adds the exact Issue #259 two-edit proof. Reviewer found no blockers/actionable findings. First deterministic gate attempt was blocked only by the shared repository Castor lock held by sibling task replace-mcp-underscore-prefix-with-terminal-star-wildcard; no task code/QA failure.

## Task workflow update - 2026-07-21T00:38:48.254Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/307
- Updated PR Status: open
- Validation: Deterministic castor check: PASSED (123.6s).; PR #307: https://github.com/ineersa/agent-core/pull/307
- Summary: Task-to-PR complete. Deterministic castor check passed in 123.6s on retry; branch pushed and PR #307 created. PR body includes `Closes #259`, so the GitHub issue will close automatically when merged.

## Task workflow update - 2026-07-21T02:28:28.553Z
- Moved CODE-REVIEW → DONE.
- Merged task/issue-259-safeguard-edit-allow-once-scope into integration checkout.
- Merge made by the 'ort' strategy.
 .../E2E/SafeGuardApprovalControllerReplayTest.php  | 185 +++++++++++++++++++++
 1 file changed, 185 insertions(+)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/issue-259-safeguard-edit-allow-once-scope.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/issue-259-safeguard-edit-allow-once-scope.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #307 state: MERGED at 2026-07-21T02:28:08Z.; Pre-merge integration checkout clean (`main...origin/main [ahead 5]`).
- Summary: PR #307 merged on GitHub at d7a94034a9d48660db154d51221d7bfeb01a38be. Issue #259 is closed by the PR's `Closes #259` linkage. Moving task to DONE and synchronizing/cleaning the task worktree.

## Task workflow update - 2026-07-21T02:30:43.012Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/307
- Updated PR Status: merged
- Validation: Post-merge `LLM_MODE=true castor check`: PASSED in 308.3s.; Gate lanes: deptrac OK; unit/integration 4438 tests / 15187 assertions; controller replay 9 / 127; TUI 36 / 185; llm-real 12 / 163; phpstan 0 errors; cs-check clean.; llama-proxy cache guard stable 90 → 90.; QA artifact integrity passed; no QA-owned process/tmux leaks.; GitHub Issue #259 state: CLOSED at 2026-07-21T02:28:09Z.; Integration checkout clean; issue task worktree absent.
- Summary: DONE verification complete. PR #307 merged; Issue #259 closed automatically; task branch merged into integration checkout; task worktree and IDEA exclusions removed. Integration checkout remains clean.

## Task workflow update - 2026-08-06T20:59:11.569Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
