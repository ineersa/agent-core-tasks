# Preserve complete bash cards while watching live fork and subagent output

## Goal
## Goal
Fix the intermittent live-only child transcript defect: while watching a fork/subagent live, normal LLM-generated bash tool calls can appear as flat output-only `● bash` entries with no command arguments or exchange-card styling. After restarting/resuming the same session, canonical replay renders those same calls correctly.

## Root-cause evidence
Parent and selected-child pollers consume a shared JSONL stream. When the parent poll receives child transient tool-call events first, `RuntimeEventPerRunCompactBuffer` treats `tool_call.arguments_completed` as a seq=0 checkpoint: it prunes the corresponding `tool_call.started`/argument deltas and discards the completion event itself, even when that child is explicitly observed. The child live projector can therefore receive tool execution results without the ToolCall block. Which poller drains the batch first makes the symptom intermittent. Resume bypasses this compact cross-run tail and rebuilds complete calls from ordered canonical `assistant.message_completed.tool_calls` data.

Keep PR #368/direct `!command` handling separate and unchanged. Fix the shared live child event buffering/projection path; do not add a fork-specific renderer.

## Acceptance criteria
- While actively watching a fork or subagent, normal bash tool calls render as the existing complete colored exchange card with command arguments, matching resumed replay.
- An explicitly observed child retains the sufficient seq=0 tool-call completion checkpoint when its events are consumed first by the parent-side shared JSONL poll; transient superseded chunks may still be compacted.
- Unobserved runs keep existing compact-tail/memory behavior.
- Normal parent transcript polling, child replay/resume, and existing direct `!command` rendering remain unchanged.
- No duplicate ToolCall blocks are created when streaming and canonical completion events both arrive.
- Add the smallest regression at the live cross-run demux/poller layer that reproduces parent-first consumption and would fail without the production fix; use the lowest correct TUI/runtime test layer.
- Run focused Castor validation and full required runtime/TUI gate before PR.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-07-preserve-complete-bash-cards-in-live-child-event-view
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-07-preserve-complete-bash-cards-in-live-child-event-view
Fork run: 0putftx9umqt
PR URL: https://github.com/ineersa/agent-core/pull/371
PR Status: merged
Started: 2026-08-07T02:54:43.116Z
Completed: 2026-08-07T16:27:49.027Z

## Work log
- Created: 2026-08-07T02:54:16.089Z

## Task workflow update - 2026-08-07T02:54:43.117Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-07-preserve-complete-bash-cards-in-live-child-event-view.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-07-preserve-complete-bash-cards-in-live-child-event-view.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-07-preserve-complete-bash-cards-in-live-child-event-view.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-07-preserve-complete-bash-cards-in-live-child-event-view.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-07-preserve-complete-bash-cards-in-live-child-event-view.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-07-preserve-complete-bash-cards-in-live-child-event-view.
- Summary: Task started from current main after PR #368 merge. Loaded task-workflow/testing guidance and tests/AGENTS.md. Read-only direct investigation isolated the intermittent live-only race to explicitly observed child seq=0 checkpoint compaction on the shared JSONL parent/child stream; canonical resume bypasses this path.

## Task workflow update - 2026-08-07T02:59:16.858Z
- Recorded fork run: 9npor83fiosd
- Validation: PASS failing-before proof: with production change stashed, new parent-first observed-child regression failed because drain contained only `tool_execution.completed`; restored fix passes.; PASS: `castor test --filter='JsonlProcessAgentSessionClientEventBufferTest\|RuntimeEventPerRunCompactBufferTest'` — 26 tests, 87 assertions.; PASS: focused `castor phpstan --path=src/CodingAgent/Runtime/Process/RuntimeEventPerRunCompactBuffer.php` — 0 errors.; PASS: `castor cs-check` after scoped `castor cs-fix` — clean.; Deptrac not run at task-start because no architecture boundary changed; full required gate deferred to task-to-pr.
- Summary: Implementation completed and committed as `f1175a2e4fbf86ba3cc93e2627ab22c2dbfc3c96`. The shared compact buffer now retains a seq=0 stream checkpoint after pruning only when the run is explicitly observed; unobserved child compaction remains prune-only. Parent-first JSONL demux now delivers `tool_call.arguments_completed` to child live projection, producing one complete bash ToolCall/ToolResult pair. PR #368 direct-shell behavior is untouched. Verified clean worktree and exact 2-file diff.

## Task workflow update - 2026-08-07T03:17:39.815Z
- Recorded fork run: 0putftx9umqt
- Validation: PASS: focused compact-buffer/JSONL event-buffer tests — 26 tests, 87 assertions.; PASS: focused PHPStan — 0 errors.; PASS: cs-check clean.
- Summary: Addressed reviewer blocker in `2779aba946bfa9a01fb87fb2deb5f15ed32ad168`: hoisted duplicated stream-checkpoint prune/observed-retain handling into one shared branch without behavior changes.

## Task workflow update - 2026-08-07T03:28:15.930Z
- Validation: APPROVED reviewer re-review after exact blocker fix.; PASS: `castor test` — 4399 tests, 16489 assertions.; PASS: `castor test:tui` — 34 tests, 226 assertions.; PASS: `castor deptrac` — 0 violations/errors.; PASS: `castor phpstan` — 0 errors.; PASS: `castor cs-check` — clean.
- Summary: Reviewer iteration complete. Initial REQUEST CHANGES blocker (duplicated checkpoint branch) was fixed in `2779aba946bfa9a01fb87fb2deb5f15ed32ad168`; re-review verdict APPROVED. Specification fidelity passed: no new API/settings/storage/protocol surface, and direct `!command` path remains untouched.

## Task workflow update - 2026-08-07T03:30:18.156Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 240s)...
- castor check passed (109.1s).
- Pushed task/2026-08-07-preserve-complete-bash-cards-in-live-child-event-view to origin.
- branch 'task/2026-08-07-preserve-complete-bash-cards-in-live-child-event-view' set up to track 'origin/task/2026-08-07-preserve-complete-bash-cards-in-live-child-event-view'.
- Created PR: https://github.com/ineersa/agent-core/pull/371
- Validation: Reviewer verdict APPROVED after one resolved DRY blocker.; Focused gates passed: castor test, test:tui, deptrac, phpstan, cs-check.
- Summary: Approved fix retains observed-child seq=0 completion checkpoints across parent-first shared JSONL demux, preserving complete live bash cards while keeping unobserved compaction unchanged.

## Task workflow update - 2026-08-07T16:27:49.027Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-08-07-preserve-complete-bash-cards-in-live-child-event-view into integration checkout.
- Merge made by the 'ort' strategy.
 .../Process/RuntimeEventPerRunCompactBuffer.php    |  27 ++--
 ...onlProcessAgentSessionClientEventBufferTest.php | 143 +++++++++++++++++++++
 2 files changed, 156 insertions(+), 14 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-07-preserve-complete-bash-cards-in-live-child-event-view.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-07-preserve-complete-bash-cards-in-live-child-event-view.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #371 confirmed MERGED.; Pre-merge reviewer APPROVED and deterministic `castor check` passed.
- Summary: PR #371 merged on GitHub as `d01d7392238f613ead352528b03b172f2c20d488`. Completing task and cleaning its worktree.

## Task workflow update - 2026-08-07T16:31:08.925Z
- Validation: PASS: `LLM_MODE=true castor check` on integration checkout — all 7 lanes passed; 4399 unit/integration tests, 12 controller replay tests, 34 TUI tests, 13 llm-real tests; Deptrac/PHPStan/CS clean; cache guard stable 244→244; no leaked QA processes.; Initial post-merge attempt was blocked only by the repository-wide Castor lock held by another task; holder exited and retry passed without killing any process.
- Summary: Post-merge integration validation passed after waiting for a concurrent sibling-worktree Castor lock holder to exit. Task worktree removed and IDEA exclusions cleaned.

## Task workflow update - 2026-08-14T19:53:37+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
