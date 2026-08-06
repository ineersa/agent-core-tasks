# Show real LLM step count in subagent progress cards

## Goal
Replace the misleading subagent card `turns` value, which currently renders the internal event-derived `turn_no`, with a durable count of LLM attempts. This is a narrow follow-up discovered while auditing session 36; leave `restore-full-transcript-navigation-context-clarity` deferred and unchanged.

Count both `llm_step_completed` and `llm_step_failed`. Update the existing deferred child lifecycle projection and delivered parent `subagent_progress` snapshot so normal `/resume` reconstructs the card from the parent event log without reading child `events.jsonl`. Do not add parallel storage or compatibility fallbacks.

## Acceptance criteria
- The deferred child lifecycle projection durably increments one LLM-step counter for every `llm_step_completed` and `llm_step_failed` event.
- Single and parallel subagent progress payloads carry the counter through the existing projection/snapshot path.
- Subagent transcript cards display `N LLM steps` instead of rendering internal `turn_no` as `N turns`.
- Normal `/resume` reconstructs the displayed count from parent `subagent_progress`; no child event-log reread is added.
- Focused tests cover completed and failed LLM-step counting, payload propagation, rendering, and parent replay/resume behavior at the lowest appropriate layer.
- All QA uses Castor; full `castor check` is required before CODE-REVIEW because the change touches runtime/TUI flow.

## Workflow metadata
Status: ARCHIVE
Branch: task/show-subagent-llm-step-count
Worktree: /home/ineersa/projects/agent-core-worktrees/show-subagent-llm-step-count
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/337
PR Status: merged
Started: 2026-07-30T01:07:12.081Z
Completed: 2026-07-30T01:44:12.328Z

## Work log
- Created: 2026-07-30T01:07:04.785Z

## Task workflow update - 2026-07-30T01:07:12.081Z
- Moved TODO → IN-PROGRESS.
- Created branch task/show-subagent-llm-step-count.
- Created worktree /home/ineersa/projects/agent-core-worktrees/show-subagent-llm-step-count.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/show-subagent-llm-step-count.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/show-subagent-llm-step-count.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/show-subagent-llm-step-count.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/show-subagent-llm-step-count.
- Summary: Started as a narrow follow-up to session 36 audit. Original transcript/context task remains deferred.

## Task workflow update - 2026-07-30T01:13:17.484Z
- Validation: `castor test --filter='DeferredChildRunEventProjectorTest|SubagentChildProgressSummaryBuilderTest|SubagentProgressProjectionTest|SubagentResultRendererTest'` — OK (21 tests, 162 assertions); `castor deptrac` — 0 violations; `castor phpstan` — 0 errors; `castor cs-check` — clean; Full `castor check` intentionally deferred to task-to-pr/CODE-REVIEW gate
- Summary: Implemented and committed e0c22475ffb82b1fcd2bcbaa7fb07ef56fbdec0a. Added durable llm_step_count for completed+failed LLM attempts in the existing child lifecycle projection, propagated it through parent subagent_progress, and replaced misleading N turns rendering with singular/plural N LLM step(s). Normal resume remains parent-event-only; no child JSONL read or compatibility fallback added. Worktree is clean. Fork confirmed it read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`.

## Task workflow update - 2026-07-30T01:23:13.902Z
- Validation: Reviewer: APPROVED; read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`; no correctness, architecture, security, scope, or complexity blockers; `castor test` — OK (4470 tests, 16284 assertions); `castor test:tui` — OK (38 tests, 194 assertions; replay-backed, no live LLM); `castor deptrac` — 0 violations; `castor phpstan` — 0 errors; `castor cs-check` — clean
- Summary: CODE-REVIEW preparation complete. Reviewer verdict APPROVED with no blocking findings; specification-fidelity gate passed. External behavior maps directly to task acceptance: durable completed+failed LLM attempt count, parent progress propagation, singular/plural card rendering, and parent-only resume replay. Commit e0c22475ffb82b1fcd2bcbaa7fb07ef56fbdec0a; worktree clean.

## Task workflow update - 2026-07-30T01:25:31.519Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (123.4s).
- Pushed task/show-subagent-llm-step-count to origin.
- branch 'task/show-subagent-llm-step-count' set up to track 'origin/task/show-subagent-llm-step-count'.
- Created PR: https://github.com/ineersa/agent-core/pull/337

## Task workflow update - 2026-07-30T01:44:12.328Z
- Moved CODE-REVIEW → DONE.
- Merged task/show-subagent-llm-step-count into integration checkout.
- Merge made by the 'ort' strategy.
 .../Deferred/DeferredChildRunEventProjector.php    |   4 +
 .../DeferredChildRunLifecycleProjectionDTO.php     |   3 +
 .../Execution/SubagentChildProgressSummary.php     |   2 +
 .../SubagentChildProgressSummaryBuilder.php        |   9 ++
 .../SubagentProgressDisplayFormatter.php           |  21 +++--
 src/Tui/Transcript/SubagentResultRenderer.php      |   2 +-
 .../Transcript/SubagentTranscriptCardBuilder.php   |  10 ++-
 .../DeferredChildRunEventProjectorTest.php         | 100 ++++++++++++++++++++-
 .../SubagentChildProgressSummaryBuilderTest.php    |   2 +
 .../Projection/SubagentProgressProjectionTest.php  |   9 +-
 tests/Tui/E2E/TuiSubagentProgressE2eTest.php       |   3 +-
 .../Tui/Support/SubagentProgressEventsFixture.php  |   2 +
 .../Tui/Transcript/SubagentResultRendererTest.php  |  29 +++++-
 13 files changed, 173 insertions(+), 23 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/show-subagent-llm-step-count.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/show-subagent-llm-step-count.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #337 confirmed merged on GitHub at 2026-07-30T01:43:47Z (merge commit ed28ffc23339e7ab64f651e1bcfc37e089b90ed6).

## Task workflow update - 2026-07-30T01:46:48.490Z
- Validation: `LLM_MODE=true castor check` on integration checkout — quality OK (366.0s); deptrac OK; test OK (4470 tests, 16284 assertions); controller replay OK (10 tests, 135 assertions); TUI replay OK (38 tests, 194 assertions); llm-real OK (15 tests, 186 assertions); phpstan OK; cs-check OK; llama-proxy cache guard stable (200 → 200); QA artifact integrity OK; QA leak check OK; Integration `git status --short` clean; task worktree absent
- Summary: Post-merge completion verified. PR #337 merged, integration checkout synchronized and clean at b3d68ace3, task worktree removed, and IDEA exclusions cleaned up.

## Task workflow update - 2026-08-06T20:59:35.259Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
