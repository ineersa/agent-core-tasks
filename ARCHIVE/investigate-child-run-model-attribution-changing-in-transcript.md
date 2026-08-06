# Investigate child-run model attribution changing in transcript

## Goal
Investigate session 37 reports that the model shown for scout/fork child runs changed or was misattributed over time. The user observed scouts running as `openai-codex/gpt-5.6-sol`, while the later transcript displayed `deepseek/deepseek-v4-flash`; similarly, fork model attribution appeared inconsistent. Recover immutable evidence from session events, child artifacts, provider requests, launch configuration resolution, and transcript projection. Determine whether the wrong model actually ran, launch metadata was overwritten, replay/projection resolved against current settings instead of stored run metadata, parallel child labels were crossed, or the TUI rendered stale/incorrect model information. Include the relevant configuration history: duplicate user scout definitions had conflicting model values; project override was briefly added then removed; fork defaults changed during the session. Model attribution must reflect the model resolved at launch and must not change retroactively when settings or agent definitions change.

## Acceptance criteria
- Recover the exact resolved model and thinking level used by every scout and fork child launched in session 37 using immutable event/provider evidence
- Explain why the live display and later transcript could show different model names, including whether the execution model itself differed from displayed attribution
- Ensure child-run model/thinking metadata is persisted at launch and transcript projection never recomputes historical attribution from current settings
- Add regression coverage for settings/agent-definition changes after child launch, parallel children, session resume/replay, and failed child runs
- Expose sufficient artifact/session diagnostics for users to verify which provider/model actually handled each child run

## Workflow metadata
Status: ARCHIVE
Branch: task/investigate-child-run-model-attribution-changing-in-transcript
Worktree: /home/ineersa/projects/agent-core-worktrees/investigate-child-run-model-attribution-changing-in-transcript
Fork run: qi2hw8mwh07f
PR URL: https://github.com/ineersa/agent-core/pull/354
PR Status: merged
Started: 2026-08-04T15:21:45.039Z
Completed: 2026-08-04T16:50:01.985Z

## Work log
- Created: 2026-07-31T20:47:43+00:00

## Task workflow update - 2026-08-04T15:21:45.039Z
- Moved TODO → IN-PROGRESS.
- Created branch task/investigate-child-run-model-attribution-changing-in-transcript.
- Created worktree /home/ineersa/projects/agent-core-worktrees/investigate-child-run-model-attribution-changing-in-transcript.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/investigate-child-run-model-attribution-changing-in-transcript.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/investigate-child-run-model-attribution-changing-in-transcript.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/investigate-child-run-model-attribution-changing-in-transcript.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/investigate-child-run-model-attribution-changing-in-transcript.
- Summary: Starting prioritized task #5. Investigation will first reconstruct immutable session-37 launch/provider evidence and trace launch metadata through durable child state, replay, handoff artifacts, transcript projection, and TUI. No implementation assumptions until the actual attribution drift seam is proven.

## Task workflow update - 2026-08-04T15:38:13.682Z
- Summary: Three read-only scouts completed. Immutable session-37 evidence shows no child changed model after launch: forks agent_34068a80, agent_a7f0997f, agent_2b5cd10a ran openai-codex/gpt-5.6-sol; parallel scouts agent_d38d509a/agent_5288fcbe and later explicit forks agent_8438fd0c/agent_de746897 ran deepseek/deepseek-v4-flash. Parent progress labels stayed associated with artifact/run IDs. Settings/definitions changed during the session, explaining different later launches and likely user-visible confusion; no July-31 provider wire log survived. Durable child run_started metadata/state is the immutable execution evidence. A real latent attribution bug was found in both DeferredChildRunEventProjector and SubagentChildProgressSummaryBuilder: definition_model is initialized first and canonical child run_started.metadata.model only applies when model is null, so stale/current definition metadata can override resolved launch evidence during live/recovery/transcript progress. Thinking/reasoning is already persisted at launch in run_started metadata; scouts without a configured thinking level correctly have null. Minimal fix: canonical run_started model must override definition fallback in both projection paths; preserve definition model only when no run_started exists (e.g. prelaunch/launch failure). Lowest proof is focused projector plus lifecycle/summary coverage; existing TUI replay fixtures inject model directly and cannot catch this seam, so no broad tmux test.

## Task workflow update - 2026-08-04T15:41:51.356Z
- Recorded fork run: 7qkle6dv3jo1
- Validation: castor test --filter='DeferredChildRunEventProjectorTest|SubagentChildProgressSummaryBuilderTest' — OK (6 tests, 76 assertions); castor phpstan --path=src/CodingAgent/Agent/Execution/Subagent/ChildRun/Deferred/DeferredChildRunEventProjector.php — 0 errors; castor phpstan --path=src/CodingAgent/Agent/Execution/SubagentChildProgressSummaryBuilder.php — 0 errors; castor cs-check — clean; git diff --check origin/main...HEAD — clean; Full castor check intentionally deferred to task-to-pr.
- Summary: Implementation complete in commit db8b7cb39. Canonical non-empty child `run_started.metadata.model` now overrides stale definition/current fallback in both DeferredChildRunEventProjector and SubagentChildProgressSummaryBuilder. Definition model remains fallback only when no launch evidence exists. Focused tests prove failed-child + resume preservation and two parallel artifact identities with distinct launch models despite one stale definition. No storage/protocol/DTO/TUI fields, model_changed handling, provider logging, or unrelated behavior added. Parent verified clean worktree, intended 4-file diff, and clean diff check. Task remains IN-PROGRESS pending task-to-PR.

## Task workflow update - 2026-08-04T16:17:58.033Z
- Recorded fork run: qi2hw8mwh07f
- Validation: Reviewer: APPROVED — no blockers; exact 4-file +130/-2 surface matches finalized requirements; castor test — OK (4424 tests, 16426 assertions); castor deptrac — 0 violations, 0 errors; castor phpstan — 0 errors; castor cs-check — clean; git diff --check origin/main...HEAD — clean; Topology: origin/main sole merge-base and ancestor; two-dot/three-dot diffs identical
- Summary: Task-to-PR review complete at HEAD 6fbe774fc. Normal non-destructive merge of origin/main repaired prior criss-cross topology; origin/main is now sole merge-base/ancestor and two-dot/three-dot both contain only the intended 4 files. Reviewer APPROVED with no blocking correctness, security, specification-fidelity, test-value, or complexity findings. Reviewer confirmed canonical run_started model precedence, definition fallback behavior, failed-child/resume proof, and per-artifact parallel child proof. No external surface added. Worktree remains clean.

## Task workflow update - 2026-08-04T16:19:52.832Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 240s)...
- castor check passed (100.6s).
- Pushed task/investigate-child-run-model-attribution-changing-in-transcript to origin.
- branch 'task/investigate-child-run-model-attribution-changing-in-transcript' set up to track 'origin/task/investigate-child-run-model-attribution-changing-in-transcript'.
- Created PR: https://github.com/ineersa/agent-core/pull/354
- Validation: castor test — OK (4424 tests, 16426 assertions); castor deptrac — 0 violations/errors; castor phpstan — 0 errors; castor cs-check — clean; Reviewer APPROVED; spec fidelity satisfied; clean 4-file diff
- Summary: Reviewer APPROVED at HEAD 6fbe774fc after non-destructive origin/main topology merge. Canonical child run_started model now overrides stale definition/current projection fallback in both attribution paths; focused regressions cover failed child, resume, and parallel artifact identity. Full focused QA is green.

## Task workflow update - 2026-08-04T16:50:01.985Z
- Moved CODE-REVIEW → DONE.
- Merged task/investigate-child-run-model-attribution-changing-in-transcript into integration checkout.
- Merge made by the 'ort' strategy.
 .../Deferred/DeferredChildRunEventProjector.php    |  4 +-
 .../SubagentChildProgressSummaryBuilder.php        |  3 +-
 .../DeferredChildRunEventProjectorTest.php         | 64 ++++++++++++++++++++++
 .../SubagentChildProgressSummaryBuilderTest.php    | 61 +++++++++++++++++++++
 4 files changed, 130 insertions(+), 2 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/investigate-child-run-model-attribution-changing-in-transcript.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/investigate-child-run-model-attribution-changing-in-transcript.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #354 state MERGED confirmed via gh CLI; Task-to-PR deterministic castor check passed before merge (100.6s)
- Summary: PR #354 merged on GitHub at 2026-08-04T16:49:40Z (merge commit 81b2e04dbd9b8d7178cc96bcfe65d24d2a0f152f). Moving task to DONE and cleaning worktree.

## Task workflow update - 2026-08-04T16:52:02.539Z
- Validation: LLM_MODE=true castor check — quality OK; castor test lane — 4424 tests, 16426 assertions; controller-replay — 11 tests, 160 assertions; TUI replay — 32 tests, 206 assertions; llm-real — 13 tests, 144 assertions; deptrac — OK; phpstan — 0 errors; cs-check — clean; llama-proxy cache stable 223→223; QA leak check OK; 6 exact-run cache roots removed; PHAR rebuilt and smoke tests passed; Worktree removed; integration checkout clean
- Summary: Post-merge validation complete on integration main at e83dadae0. PR #354 is merged, task is DONE, worktree and IDEA exclusions removed, integration checkout clean.

## Task workflow update - 2026-08-06T20:59:11.566Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
