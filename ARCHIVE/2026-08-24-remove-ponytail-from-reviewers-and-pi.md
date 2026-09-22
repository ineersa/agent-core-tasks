# Remove Ponytail from reviewers and Pi

## Goal
Uninstall Ponytail from the active Hatfield and Pi environments. Remove Ponytail skills and package loading, remove Ponytail-specific reviewer dependencies/instructions from user-level reviewer definitions, and update the bundled agent-core reviewer so agents:init cannot restore it. Preserve the reviewer’s normal correctness, security, design, and test review behavior. Existing `ponytail:` source comments are not loaders and remain untouched.

## Acceptance criteria
- `~/.hatfield/agents/reviewer.md` and `~/.agents/reviewer.md` contain no Ponytail skill or Ponytail-specific review section.
- `~/.pi/agent/settings.json` no longer installs the Ponytail package or exposes its skill path.
- Global Hatfield Ponytail skill directories and the Pi Ponytail package checkout are removed.
- Bundled `src/CodingAgent/Resources/agents/reviewer.md` contains no Ponytail dependency or output section, preventing reinstallation through agents:init.
- Normal reviewer correctness, security, design-quality, integration, and test-coverage review instructions remain intact.
- No active Ponytail loader references remain; existing non-loader `ponytail:` source comments remain untouched.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-24-remove-ponytail-from-reviewers-and-pi
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-24-fix-critical-runtime-event-performance-and-delivery-worktrees/2026-08-24-remove-ponytail-from-reviewers-and-pi
Fork run: kjmj8yknwcs1
PR URL: https://github.com/ineersa/agent-core/pull/429
PR Status: merged
Started: 2026-08-24T18:46:37+00:00
Completed: 2026-08-25T14:32:56.720Z

## Work log
- Created: 2026-08-24T18:46:19+00:00

## Task workflow update - 2026-08-24T18:46:37+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-24-remove-ponytail-from-reviewers-and-pi.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-24-fix-critical-runtime-event-performance-and-delivery-worktrees/2026-08-24-remove-ponytail-from-reviewers-and-pi.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-24-fix-critical-runtime-event-performance-and-delivery-worktrees/2026-08-24-remove-ponytail-from-reviewers-and-pi.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-24-fix-critical-runtime-event-performance-and-delivery-worktrees/2026-08-24-remove-ponytail-from-reviewers-and-pi.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-24-fix-critical-runtime-event-performance-and-delivery-worktrees/2026-08-24-remove-ponytail-from-reviewers-and-pi.
- Parent IDEA module not found or ambiguous; skipping exclusion update.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-24-fix-critical-runtime-event-performance-and-delivery-worktrees/2026-08-24-remove-ponytail-from-reviewers-and-pi/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-24-fix-critical-runtime-event-performance-and-delivery-worktrees/2026-08-24-remove-ponytail-from-reviewers-and-pi.
- Summary: User approved full Ponytail removal from reviewer agents and Pi settings. Scope includes active user-level loaders plus bundled reviewer cleanup to prevent agents:init reintroducing Ponytail.

## Task workflow update - 2026-08-24T19:40:41+00:00
- Recorded fork run: agent_b3d4905d5e5f9bb1
- Validation: Zero Ponytail references in ~/.hatfield/agents/reviewer.md, ~/.agents/reviewer.md, ~/.pi/agent/settings.json, and bundled worktree reviewer.; No ~/.hatfield/skills/ponytail* directories remain; Pi Ponytail checkout is absent.; Pi settings JSON parses successfully.; Focused Castor validation from fork: AgentsInitCommandTest|AgentDefinition PASS 93 tests/306 assertions; castor docs:validate PASS.; Tracked bundled reviewer commit: 5039a1a92; task worktree clean.
- Summary: Ponytail removal is complete across active Hatfield and Pi loaders. After the user adjusted bubblewrap and restarted, the staged cleaned global reviewer was applied successfully. User also confirmed Pi Atlas removal was separately authorized; its usage-data cache remains preserved.

## Task workflow update - 2026-08-25T14:16:32.768Z
- Validation: Existing focused validation: AgentsInitCommandTest|AgentDefinition PASS, 93 tests/306 assertions.; Existing castor docs:validate PASS.; Worktree clean; branch changes only bundled architect.md and reviewer.md.
- Summary: User confirmed unrelated-looking commit 0e8c2e0c7 is intentional output from the skills update and should be included for review alongside bundled reviewer Ponytail removal. Moving branch to CODE-REVIEW so user can inspect both bundled agent definition changes.

## Task workflow update - 2026-08-25T14:18:37.366Z
- Validation: castor check FAILED: unit lane 1 failure, 707 tests/2907 assertions.; Failure: AgentResumeExecutionServiceTest::testRejectsForkChildKindFromMetadata expected ToolCallException; likely stale branch/test wiring already corrected on newer main.; No tracked dirty changes after failed gate.
- Summary: First CODE-REVIEW transition was blocked by deterministic castor check unit failure on stale branch: AgentResumeExecutionServiceTest::testRejectsForkChildKindFromMetadata did not throw expected ToolCallException. All other check lanes produced artifacts; worktree remained clean. Fix fork is merging latest origin/main and will preserve only architect/reviewer PR diff, then rerun focused test and full gate.

## Task workflow update - 2026-08-25T14:22:51.155Z
- Recorded fork run: jvmwvhy7u2t6
- Validation: castor test --filter=AgentResumeExecutionServiceTest PASS: 13 tests/43 assertions.; castor check PASS: 4,838 tests/19,708 assertions; controller replay 6/92; TUI 8/60; llm-real 5/30; phpstan/deptrac/cs/docs/catalog/leak/cache guards PASS.; JUnit audit: no individual testcase >10s.; git diff --check PASS; worktree clean; final diff only architect.md and reviewer.md.
- Summary: Merged latest origin/main at aebb0f2b6, which supplied the authoritative firstFor() stale-test correction. Focused and full gates pass. Final branch diff contains only the user-approved bundled architect/reviewer updates; ready to retry CODE-REVIEW transition.

## Task workflow update - 2026-08-25T14:24:17.934Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (73.4s).
- Pushed task/2026-08-24-remove-ponytail-from-reviewers-and-pi to origin.
- branch 'task/2026-08-24-remove-ponytail-from-reviewers-and-pi' set up to track 'origin/task/2026-08-24-remove-ponytail-from-reviewers-and-pi'.
- Created PR: https://github.com/ineersa/agent-core/pull/429
- Validation: castor check PASS on aebb0f2b6.; Focused AgentResumeExecutionServiceTest PASS: 13 tests/43 assertions.; Final diff only src/CodingAgent/Resources/agents/architect.md and reviewer.md.; No individual JUnit testcase >10s; worktree clean.
- Summary: Bundled agent definitions ready for user review: remove Ponytail from reviewer and include intentional architect HTML candidate-report skill refresh. Branch refreshed from origin/main; stale test issue resolved without weakening assertions.

## Task workflow update - 2026-08-25T14:32:56.720Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-24-fix-critical-runtime-event-performance-and-delivery-worktrees/2026-08-24-remove-ponytail-from-reviewers-and-pi: ide_close_project returned isError.
- Merged task/2026-08-24-remove-ponytail-from-reviewers-and-pi into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-24-fix-critical-runtime-event-performance-and-delivery-worktrees/2026-08-24-remove-ponytail-from-reviewers-and-pi.
- Parent IDEA module not found; skipping exclusion cleanup.
- Pulled integration checkout: Already up to date..
- Validation: PR #429 state MERGED; mergedAt 2026-08-25T14:29:52Z.; Pre-merge branch castor check PASS; focused AgentResumeExecutionServiceTest PASS.; Integration main clean and synchronized with origin/main before transition.
- Summary: PR #429 merged on GitHub at d38ac1e51. Moving task to DONE and cleaning its nested worktree.

## Task workflow update - 2026-08-25T14:36:13.305Z
- Recorded fork run: kjmj8yknwcs1
- Validation: LLM_MODE=true castor check PASS in 146.2s: all 9 lanes green.; 4,857 JUnit testcases inspected; none >10s, max 6.640s.; QA artifact integrity, exact-run leak check, and llama-proxy cache guard 391→391 PASS.; Git clean; origin/main...main divergence 0/0.
- Summary: Post-merge validation completed successfully on current integration main. PR #429 task is fully closed; worktree removed and main synchronized with origin.

## Task workflow update - 2026-08-29T16:09:39.017Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
