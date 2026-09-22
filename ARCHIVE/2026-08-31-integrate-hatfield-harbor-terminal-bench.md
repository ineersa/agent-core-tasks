# Integrate Hatfield with Harbor Terminal-Bench 2.1

## Goal
Set up a separate, reproducible Terminal-Bench 2.1 evaluation harness for Hatfield. First implement a Harbor custom installed-agent adapter that runs an exact Hatfield build inside task containers and preserves Hatfield event data for privacy-safe analysis. Validate Harbor and the dataset with oracle runs, then prove the adapter on one task. Pilot a broader CPU-only candidate set with one fixed approved model, select 10 stable edit-heavy tasks for routine regression checks, and retain a separate full-suite configuration for occasional runs. Keep benchmark tooling outside production runtime code; do not add product settings or provider-specific behavior. This task is separate from the edit-patch guidance task, which will consume its baseline and before/after reports.

## Acceptance criteria
- Harbor and Terminal-Bench 2.1 versions/commits are pinned and the installation and oracle preflight procedure is documented.
- A Hatfield-specific Harbor adapter runs an exact selected Hatfield build in an isolated task container, forwards model credentials without logging secrets, preserves owned logs/events, and tears down deterministically.
- One Terminal-Bench task passes through the complete Hatfield adapter path before task selection begins.
- A candidate pool is piloted with one fixed approved model and reasoning level; selection evidence includes runtime, reliability, edit-call count, resource needs, and task outcome.
- Exactly 10 fixed CPU-only tasks are selected for routine regression runs, with the pinned task names and selection rationale documented.
- Routine and occasional full-run commands/configurations are documented; routine stochastic failures are confirmed with repeated runs rather than treated as deterministic test failures.
- Privacy-safe reports include task pass/fail, first edit-attempt validity, E_PATCH_FORMAT/stale/ambiguous counts, edit attempts per successful edit, final verifier result, duration, token/cost data when available, and parent-versus-child edit counts without raw prompts or file contents.
- A baseline report is produced for the selected 10-task suite using the approved model; no causal or statistical improvement claim is made without comparable repeated before/after runs.
- Benchmark code and results do not become production runtime dependencies, settings, commands, or required Castor test lanes.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-31-integrate-hatfield-harbor-terminal-bench
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-31-integrate-hatfield-harbor-terminal-bench
Fork run: f6j4ohtbia0e
PR URL:
PR Status:
Started: 2026-08-31T17:44:13.925Z
Completed: 2026-09-01T23:22:26.197Z

## Work log
- Created: 2026-08-31T17:39:39.966Z

## Task workflow update - 2026-08-31T17:44:13.925Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-31-integrate-hatfield-harbor-terminal-bench.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-31-integrate-hatfield-harbor-terminal-bench.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-31-integrate-hatfield-harbor-terminal-bench.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-31-integrate-hatfield-harbor-terminal-bench.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-31-integrate-hatfield-harbor-terminal-bench.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-31-integrate-hatfield-harbor-terminal-bench.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-31-integrate-hatfield-harbor-terminal-bench/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-31-integrate-hatfield-harbor-terminal-bench.
- Summary: Starting as a separate repository under ~/projects per user direction. Initial sequence: pin Harbor/TBench, oracle preflight, Hatfield adapter, one end-to-end task, candidate pilot, fixed 10-task suite, baseline.

## Task workflow update - 2026-08-31T17:46:19.378Z
- Ownership: owner=fork; fork_run=pending; revision=agent-core@22e7a830e + empty /home/ineersa/projects/hatfield-harbor; scope=bootstrap separate pinned Harbor/TBench repository, implement and test Hatfield Harbor adapter and controller wrapper, document oracle/one-task preflight, no paid-model pilot or 10-task selection; outcome=assigned; commit=none

## Task workflow update - 2026-08-31T17:49:34.819Z
- Recorded fork run: f6j4ohtbia0e
- Ownership: owner=fork; fork_run=f6j4ohtbia0e; revision=agent-core@22e7a830e + empty hatfield-harbor main; scope=bootstrap pinned Harbor/TBench repo, Hatfield adapter/controller wrapper, deterministic tests, oracle preflight/docs; outcome=assigned; commit=none

## Task workflow update - 2026-08-31T17:54:43.370Z
- Finalized pilot model: openai-codex/gpt-5.6-luna with medium reasoning, chosen because the privacy-safe historical aggregate attributes 72/78 malformed-prefix failures to Luna and most Luna runs used medium reasoning.
- Scope order clarified: do not invest further in old-session analysis because the event shape changed. First complete adapter and one successful Terminal-Bench run; only then add a current-shape session inspection/report tool to the adapter repository.

## Task workflow update - 2026-08-31T18:31:13.848Z
- Recorded fork run: f6j4ohtbia0e
- Validation: uv pip install -e '.[dev]' — PASS; harbor==0.22.0; pytest -q — PASS; 12 tests; ./scripts/fetch-terminal-bench.sh — PASS; pinned dataset commit 7131e4375048a0e408a8fb404b5f499d726b695b; Fork oracle preflight — BLOCKED: /var/run/docker.sock unavailable in fork environment
- Summary: Fork completed the adapter bootstrap at hatfield-harbor commit 2a62016a6c6c738bf2af62ac88e2252bbc926374. Deterministic tests pass; Docker-backed oracle and live Hatfield runs remain for main because the fork environment lacked the Docker socket.
- Ownership: owner=fork; fork_run=f6j4ohtbia0e; revision=agent-core@22e7a830e + empty hatfield-harbor main; scope=bootstrap pinned Harbor/TBench repo, Hatfield adapter/controller wrapper, deterministic tests, oracle preflight/docs; outcome=blocked; commit=2a62016a6c6c738bf2af62ac88e2252bbc926374
- Ownership: owner=main; fork_run=none; revision=hatfield-harbor@2a62016a6c6c738bf2af62ac88e2252bbc926374; scope=review adapter, run Docker oracle preflight, build exact Hatfield PHAR, run one Luna/medium Terminal-Bench task, then add current-shape session analyzer; outcome=assigned; commit=none

## Task workflow update - 2026-08-31T18:55:28.112Z
- Ownership: owner=main; fork_run=none; revision=hatfield-harbor@2a62016 + agent-core main CI v0.0.9; scope=replace PHP+PHAR transport with CI native executable, correct project-local session artifact capture, and configure isolated Hatfield settings for agents with safe-guard disabled; outcome=assigned; commit=none

## Task workflow update - 2026-08-31T21:03:18.914Z
- Summary: User rejected Terminal-Bench task quality and redirected the active harness to SWE-bench Verified for the first real calibration smoke. A future Hatfield/PHP-specific 30–50 task benchmark is a promising follow-up design direction but is not yet finalized. Current scope is to finish the generic Harbor adapter, establish the Python test environment, and run one SWE-bench Verified task.
- Ownership: owner=main; fork_run=none; revision=hatfield-harbor@2a62016; scope=pivot repository from Terminal-Bench-specific bootstrap to generic Harbor adapter, deterministic Python test environment, and one SWE-bench Verified smoke; outcome=assigned; commit=none

## Task workflow update - 2026-08-31T21:39:40.481Z
- Validation: uv sync --extra dev — PASS; 94 packages resolved from uv.lock; pytest -q — PASS; 16 tests; python -m compileall -q src tests — PASS; python -m json.tool reports/swe-bench-smoke-2026-08-31.json — PASS; git diff --check / git diff --cached --check — PASS; Harbor SWE-bench smoke jobs/2026-08-31__17-33-55 — COMPLETE; 1 trial, 0 exceptions, verifier reward 0.0, total 95.32s
- Summary: Generic Harbor adapter now runs the CI native Hatfield executable against SWE-bench Verified. Python environment is locked with uv and deterministic tests cover controller framing, open-pipe reads, canonical terminal detection, project-local session capture, and worker tool-filter propagation. One full django__django-14376 run completed adapter+verifier in 95.32s with no harness exception; reward 0.0 was a genuine model failure (3 FAIL_TO_PASS failures, 6 PASS_TO_PASS successes). Privacy-safe report records 3 edit calls, 2 E_PATCH_FORMAT errors, 1 successful edit, 101,228 tokens, and $0.0399748 cost. Commit remains blocked because GPG pinentry was cancelled twice.
- Ownership: owner=main; fork_run=none; revision=hatfield-harbor@2a62016; scope=pivot repository from Terminal-Bench-specific bootstrap to generic Harbor adapter, deterministic Python test environment, and one SWE-bench Verified smoke; outcome=completed; commit=none
- Commit blocker: GPG key CDBF03D56FBA5376ED97A8CBBE170B5F4883B870 pinentry timed out/cancelled; all intended changes are staged in /home/ineersa/projects/hatfield-harbor. No unsigned commit was created.

## Task workflow update - 2026-08-31T22:10:20.170Z
- Validation: git commit -m 'Pivot harness to SWE-bench smoke' — PASS; commit 4007b56
- Summary: User approved committing the completed SWE-bench adapter pivot and requested a 10-task subset run to measure edit-tool failures. Commit 4007b56 records the adapter, locked Python environment, smoke proof, and privacy-safe single-task report. Next slice: add the current-session aggregate reporter and fixed 10-task SWE-bench subset, then run the subset with Luna/medium and report task/edit metrics.
- Ownership: owner=main; fork_run=none; revision=hatfield-harbor@4007b56; scope=add privacy-safe current-session aggregate reporter, pin 10 SWE-bench tasks, run one attempt per task with openai-codex/gpt-5.6-luna medium, and summarize edit failures; outcome=assigned; commit=none

## Task workflow update - 2026-08-31T23:05:44.984Z
- Validation: uv/pytest -q — PASS; 17 tests; python -m compileall -q src tests — PASS; bash -n scripts/run-swe-bench-candidate-10.sh — PASS; 10-task Harbor run — PASS as harness execution; 10 completed, 0 adapter exceptions, 3 verifier passes, 7 verifier failures; report assertions — PASS; 10 tasks, 122 E_PATCH_FORMAT, all 122 trailing_end_patch, 0 other format shapes; python -m json.tool reports/swe-bench-candidate-10-2026-08-31.json — PASS; git diff --check — PASS; process/container teardown check — PASS; no surviving Harbor, Hatfield controller, or benchmark containers
- Summary: Completed the first 10-task SWE-bench candidate subset at hatfield-harbor commit 7e95b67. All 10 trials completed with no adapter exceptions; 3 passed. The run produced 158 edit calls: 32 successful, 122 E_PATCH_FORMAT, 3 ambiguous, 1 stale, with 0/10 first edit attempts succeeding and 4.9375 attempts per successful edit. Every E_PATCH_FORMAT failure was the same model output shape: a valid @@ hunk ending with unsupported `*** End Patch`. Total trial duration was 2773.35s, usage 4,439,441 tokens, cost $1.3946944. Privacy-safe committed report: reports/swe-bench-candidate-10-2026-08-31.json.
- Ownership: owner=main; fork_run=none; revision=hatfield-harbor@4007b56; scope=add privacy-safe current-session aggregate reporter, pin 10 SWE-bench tasks, run one attempt per task with openai-codex/gpt-5.6-luna medium, and summarize edit failures; outcome=completed; commit=7e95b67

## Task workflow update - 2026-09-01T23:22:26.197Z
- Moved IN-PROGRESS → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-31-integrate-hatfield-harbor-terminal-bench: ide_close_project returned isError.
- Merged task/2026-08-31-integrate-hatfield-harbor-terminal-bench into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-31-integrate-hatfield-harbor-terminal-bench.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-31-integrate-hatfield-harbor-terminal-bench.
- Pulled integration checkout: Already up to date..
- Validation: hatfield-harbor clean at commit 7e95b67; pytest 17 tests PASS; compileall, JSON validation, bash syntax, and git diff checks PASS; 10 SWE-bench trials completed with 0 adapter exceptions; privacy-safe report committed; Baseline: 158 edit calls, 122 E_PATCH_FORMAT, all trailing_end_patch; Candidate common-task comparison: 19 edit calls, 18 successful, 0 E_PATCH_FORMAT; Current agent-core post-merge castor check qa-20260901-231654-439872-2b7fb8ff PASS; task branch has no agent-core changes
- Summary: Benchmark task closed per user direction. Scope evolved from rejected Terminal-Bench tasks to a generic Harbor adapter plus SWE-bench calibration. Separate repository `/home/ineersa/projects/hatfield-harbor` is clean at 7e95b67 with adapter bootstrap, locked environment, deterministic tests, privacy-safe reporter, pinned 10-task suite, baseline report, and candidate run results. The benchmark successfully isolated the repeated trailing `*** End Patch` failure, enabled PR #447, and the v0.0.12 comparison confirmed format errors fell from 60/71 common-task edit calls to 0/19. The later Codex WebSocket hang found during benchmark execution was fixed by merged PRs #449 and #454. Agent-core task branch contains no benchmark code and is clean.

## Task workflow update - 2026-09-06T15:40:55+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
