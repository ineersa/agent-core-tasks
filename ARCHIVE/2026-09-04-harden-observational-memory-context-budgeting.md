# Harden observational-memory context budgeting

## Goal
Reduce observational-memory context-overflow risk using the finalized conservative budgeting changes. The observed llama.cpp server exposes n_ctx=160000 while project model metadata declares 200000; the current 3.25 chars/token estimator materially undercounts code-heavy prompts; and Observer permits 16 tool-call rounds. Do not add or change any output-token cap or Extension API field: reasoning models need their current output allowance.

Finalized values:
- OmTokenEstimator: 2.35 chars/token.
- Project llama_cpp/flash model context_window: 160000.
- Project observational_memory observer context_window_ratio: 0.35.
- Observational-memory agent max tool calls: 6.

## Acceptance criteria
- OmTokenEstimator uses 2.35 chars/token and affected deterministic tests are updated.
- The tracked project llama_cpp/flash catalog entry declares context_window 160000.
- The tracked project observational-memory observer override uses context_window_ratio 0.35.
- Observer/OM agent calls use at most 6 tool-call rounds instead of 16, with existing settings/tests/docs kept consistent where applicable.
- No max-output-token cap, output-token API, or truncation behavior is introduced.
- Focused Castor validation passes.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-09-04-harden-observational-memory-context-budgeting
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-04-harden-observational-memory-context-budgeting
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/462
PR Status: merged
Started: 2026-09-04T17:04:03.763Z
Completed: 2026-09-04T18:10:31.368Z

## Work log
- Created: 2026-09-04T17:03:50.675Z

## Task workflow update - 2026-09-04T17:04:03.764Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-04-harden-observational-memory-context-budgeting.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-04-harden-observational-memory-context-budgeting.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-04-harden-observational-memory-context-budgeting.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-04-harden-observational-memory-context-budgeting.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-04-harden-observational-memory-context-budgeting.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-04-harden-observational-memory-context-budgeting.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-04-harden-observational-memory-context-budgeting/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-04-harden-observational-memory-context-budgeting.
- Summary: Started implementation with finalized values: estimator 2.35 chars/token, llama_cpp/flash context 160000, observer ratio 0.35, and OM max tool calls 6. Explicitly excludes output-token caps and related API changes.

## Task workflow update - 2026-09-04T17:04:58.202Z
- Ownership: owner=main; fork_run=none; revision=9ad15da54; scope=OM estimator constant, shared OM agent tool-call cap, deterministic extension tests/docs, and tracked project llama/observer settings; outcome=assigned; commit=none

## Task workflow update - 2026-09-04T17:12:38.636Z
- Specification clarification: maxToolCalls=6 applies to Observer only, matching the user's explicit wording; Reflector and Dropper retain their existing maxToolCalls=16. No output-token behavior or API changes.

## Task workflow update - 2026-09-04T17:13:46.902Z
- Validation: castor test --filter=ObserverChunkAndToolTest — OK (8 tests, 130 assertions); castor test --suite=extensions — OK (140 tests, 847 assertions); castor test:llm-real --filter=OmLiveLlmSmokeTest — OK (2 tests, 11 assertions); castor phpstan --path=.hatfield/extensions/observational-memory — 0 errors; castor cs-check — 0 files fixed; castor deptrac — 0 violations, 0 errors; castor docs:validate — OK; git diff --check — OK
- Summary: Implemented conservative OM observer budgeting at db3376f65: estimator now uses 2.35 chars/token; tracked llama_cpp/flash context matches n_ctx=160000; project observer ratio is 0.35; Observer alone is capped at 6 tool-call rounds while Reflector/Dropper remain at 16. Updated deterministic sizing fixtures and OM docs. No output-token cap or Extension API change was introduced.
- Ownership: owner=main; fork_run=none; revision=9ad15da54; scope=OM estimator constant, Observer-only tool-call cap, deterministic extension tests/docs, and tracked project llama/observer settings; outcome=completed; commit=db3376f65

## Task workflow update - 2026-09-04T17:29:39.583Z
- Review: role=reviewer; run_id=88d2bdbb; target=db3376f65; scope=correctness, context-budget arithmetic, tests, privacy/security, architecture, dead code, and specification fidelity; verdict=APPROVE WITH SUGGESTIONS; blockers=none; artifact=not emitted

## Task workflow update - 2026-09-04T17:37:16.431Z
- Validation: Final reviewer 6719fe1b — APPROVE WITH SUGGESTIONS; no CRITICAL/BUG/SEC blockers; castor test --suite=extensions — OK (140 tests, 847 assertions); castor test:llm-real --filter=OmLiveLlmSmokeTest — OK (2 tests, 11 assertions); castor deptrac — 0 violations, 0 errors; castor phpstan --path=.hatfield/extensions/observational-memory — 0 errors; castor cs-check — 0 files fixed; Worktree clean at db3376f6507cbf575b7453dcf01e5fe3a78cc39b
- Summary: PR preparation complete at db3376f65. Final independent review found no blockers and confirmed exact specification fidelity: 2.35 chars/token, llama context 160000, project observer ratio 0.35, Observer-only six-round cap, no output-token changes.
- Review: role=reviewer; run_id=6719fe1b; target=db3376f6507cbf575b7453dcf01e5fe3a78cc39b; scope=final correctness, tests, privacy/security, architecture, dead code, and strict specification fidelity without assuming a provider output cap; verdict=APPROVE WITH SUGGESTIONS; blockers=none; artifact=/home/ineersa/.pi/agent/sessions/--home-ineersa-projects-agent-core--/subagent-artifacts/6719fe1b_reviewer_output.md

## Task workflow update - 2026-09-04T17:39:43.003Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (131.2s).
- Pushed task/2026-09-04-harden-observational-memory-context-budgeting to origin.
- branch 'task/2026-09-04-harden-observational-memory-context-budgeting' set up to track 'origin/task/2026-09-04-harden-observational-memory-context-budgeting'.
- Created PR: https://github.com/ineersa/agent-core/pull/462
- Validation: Final independent reviewer 6719fe1b — APPROVE WITH SUGGESTIONS, no blockers; castor test --suite=extensions — OK (140 tests, 847 assertions); castor test:llm-real --filter=OmLiveLlmSmokeTest — OK (2 tests, 11 assertions); castor deptrac — 0 violations, 0 errors; castor phpstan --path=.hatfield/extensions/observational-memory — 0 errors; castor cs-check — 0 files fixed
- Summary: Prepared db3376f65 for review. OM now estimates 2.35 chars/token, uses the actual 160000 llama context and 0.35 project Observer ratio, and limits only Observer to six tool-call rounds. Reflector/Dropper remain at 16 and output-token behavior is unchanged.

## Task workflow update - 2026-09-04T18:10:31.368Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-04-harden-observational-memory-context-budgeting: ide_close_project returned isError.
- Merged task/2026-09-04-harden-observational-memory-context-budgeting into integration checkout.
- Merge made by the 'ort' strategy.
 .../extensions/observational-memory/README.md      |  9 ++++----
 .../src/Observer/ObserverPipeline.php              |  6 +++--
 .../src/Observer/OmTokenEstimator.php              |  2 +-
 .../tests/ObserveBoundaryJobHandlerTest.php        |  2 +-
 .../tests/ObserverChunkAndToolTest.php             | 27 +++++++++++-----------
 .../tests/OmLiveLlmSmokeTest.php                   |  2 +-
 .../tests/OmQueryServiceTest.php                   |  4 ++--
 .../tests/ReflectGenerationJobHandlerTest.php      |  4 ++--
 .hatfield/settings.yaml                            |  4 ++--
 9 files changed, 32 insertions(+), 28 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-04-harden-observational-memory-context-budgeting.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-04-harden-observational-memory-context-budgeting.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #462 state: MERGED at 2026-09-04T18:09:50Z
- Summary: PR #462 is merged on GitHub (merge commit 10d866a771744254b28858a34bd276cd15811b78). Completing task integration and cleanup.

## Task workflow update - 2026-09-06T15:41:19+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
