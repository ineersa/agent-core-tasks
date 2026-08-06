# Make live controller E2E session IDs unambiguously ephemeral

## Goal
Deterministic castor check for PR #284 exposed an existing main test flake after SessionAwareModelResolver began treating digits-only IDs as persisted Hatfield sessions. `ControllerE2eTestCase::setUp()` uses `substr(bin2hex(random_bytes(16)), 0, 12)`; occasionally this is digits-only. In gate qa-20260715-202812, IDs `965972235315` and `581620849752` caused ViewImageToolE2eTest and WriteFileToolE2eTest to fail with `Session "..." has no metadata for model resolution.` Focused llm-real passed when random IDs contained hex letters. Fix test infrastructure on a separate main branch; do not weaken SessionAwareModelResolver or retry until lucky.

## Acceptance criteria
- Live ControllerE2eTestCase session IDs can never be ctype_digit; use an explicit ephemeral/test prefix while preserving uniqueness
- HATFIELD_SESSION_ID, isolated DB filenames, Messenger queue names, and controller process lifecycle continue to work
- Focused affected llm-real tests and full castor test:llm-real pass through Castor
- No production model-resolution behavior is weakened
- After merge, refactor PR #284 can sync main and deterministic castor check without this random failure

## Workflow metadata
Status: DONE
Branch: task/fix-controller-e2e-numeric-session-id-flake
Worktree:
Fork run: r395cg914vaq
PR URL:
PR Status:
Started: 2026-07-15T20:33:23.730Z
Completed: 2026-07-18T23:06:45Z

## Work log
- Created: 2026-07-15T20:33:10.188Z

## Task workflow update - 2026-07-15T20:33:23.730Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-controller-e2e-numeric-session-id-flake.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-controller-e2e-numeric-session-id-flake.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-controller-e2e-numeric-session-id-flake.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-controller-e2e-numeric-session-id-flake.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-controller-e2e-numeric-session-id-flake.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-controller-e2e-numeric-session-id-flake.
- Summary: Starting narrow test-infrastructure fix required to unblock deterministic gate. Branch will first merge latest origin/main, then make live controller E2E session IDs explicitly non-numeric without changing production resolver behavior.

## Task workflow update - 2026-07-15T20:36:40.273Z
- Recorded fork run: r395cg914vaq
- Summary: Implementation itself is correct and validated (ControllerE2eTestCase live IDs now `e2e-` + random hex; filtered 2/25 and full llm-real 10/122 pass; cs-check/deptrac clean), but fork changed into the integration checkout while reading root docs and committed 178532db1 on local main instead of the task worktree. Task worktree remains at 5cfcff330 with no fix commit. No separate PR will be opened until branch placement is corrected; refactor PR #284 is being given the exact fix to unblock its deterministic gate.

## Task workflow update - 2026-07-18T23:06:45Z
- Marked complete because the equivalent non-numeric ephemeral session-ID fix is present on main at commit 884e9f004. Stale worktree removed.
