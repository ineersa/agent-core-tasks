# Investigate inflated OpenCode Go per-request token usage

## Goal
Issue: https://github.com/ineersa/agent-core/issues/536

Investigate implausible per-request usage for opencode-go/deepseek-v4.1-flash in Hatfield v0.0.28. Individual llm_step_completed records report up to 112,456,170 input tokens and 3,192,057 output tokens. The corruption precedes TUI projection.

Compare Symfony AI 0.13 and 0.14 stream usage conversion and aggregation. Determine whether OpenCode Go sends cumulative snapshots, duplicated final usage, genuine deltas, or malformed values. Capture only sanitized usage and chunk ordering if provider evidence is available. Do not retain credentials or conversation content.

Initial scope is investigation and a proposed fix, not implementation. Do not hide the defect with a UI clamp. Keep this separate from the subscription /usage task.

## Acceptance criteria
- Identify the earliest confirmed source of inflated usage and distinguish evidence from hypotheses.
- Compare affected-build usage conversion with Symfony AI 0.13 and 0.14.
- Determine provider streaming semantics from sanitized evidence, or record the exact evidence still missing.
- Propose the smallest correction and regression coverage for per-request usage, cumulative totals, latest input, and context-limit decisions.

## Workflow metadata
Status: DONE
Branch: task/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage
Fork run: agent_1f3596120af186f4
PR URL: https://github.com/ineersa/agent-core/pull/538
PR Status: merged
Started: 2026-09-28T13:15:22+00:00
Completed: 2026-09-28T20:20:19+00:00

## Work log
- Created: 2026-09-28T13:02:01+00:00

## Task workflow update - 2026-09-28T13:15:22+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage/.idea.
- Summary: Starting investigation worktree for user-led reproduction with OpenCode Go DeepSeek in parent and fork runs. No fix selected yet; cumulative stream-usage aggregation remains a hypothesis.

## Task workflow update - 2026-09-28T18:38:15+00:00
- Summary: User now authorizes implementing per-response cumulative usage normalization and creating a PR. Preserve genuine delta semantics and aggregation across distinct requests; no UI clamps or historical record rewriting. Latest reproduction: sqlite-queue session 2 child agent_f8f714ebfcc7f9c6, run ef0a5008-c074-59d7-abfb-fcabf5bc7f8a. First 21 input counts below 200k, then seq144=6,627,654, seq206=167,445,845. Final input/cache share factor 1435, yielding plausible 116,687/116,224. Need raw usage evidence before attributing root cause. Existing generic raw capture is not wired into OpenCode Go builder.
- Ownership: owner=fork; fork_run=pending; revision=811587184; scope=OpenCode Go stream-usage evidence, normalization, focused regression tests and validation; outcome=assigned; commit=none

## Task workflow update - 2026-09-28T18:44:06+00:00
- Recorded fork run: agent_1f3596120af186f4
- Summary: Implementation fork obtained usage-only live evidence: four OpenCode Go DeepSeek request variants emitted identical final usage on terminal choice and trailing empty-choices chunk. Confirms 2x aggregation defect for these requests. Does not rule out many snapshots in other responses or explain historical extreme counts; GCD is only a consistency observation, not proof of chunk multiplicity. Continuing with bounded larger-stream investigation and normalization for known cumulative semantics. No code changes yet.
- Ownership: owner=fork; fork_run=agent_1f3596120af186f4; revision=811587184; scope=OpenCode Go stream-usage evidence, normalization and regression tests; outcome=assigned; commit=none

## Task workflow update - 2026-09-28T18:44:47+00:00
- Summary: Fork continuation refused by child-context limit at 201750 tokens. Main takes over implementation using returned evidence, without launching a duplicate child. Four bounded probes confirm duplicate final snapshots but cannot rule out intermittent many-snapshot reporting.
- Ownership: owner=fork; fork_run=agent_1f3596120af186f4; revision=811587184; scope=stream evidence and implementation; outcome=blocked; commit=none
- Ownership: owner=main; fork_run=none; revision=811587184; scope=usage normalization, focused regression validation and PR handoff; outcome=assigned; commit=none

## Task workflow update - 2026-09-28T19:45:19+00:00
- Validation: Regression before fix failed with expected100000 vs actual200000 input tokens.; castor test --filter='SymfonyAiProviderFactoryTest|DurableResultConverterTest|LlmPlatformAdapterTest': 56 tests,334 assertions passed; slowest0.017s.; castor phpstan: zero errors.; castor test:llm-real --filter=LlamaCppSmokeTest: 1 test,8 assertions passed.; Command supervision lost foreground status; recovered final exit0 from .hatfield/tmp/bg/46dda015cf7a6c76.status and complete log; no rerun.; git diff --check passed.
- Summary: Implemented e39eaaffe: opt-in whole-request usage snapshots in DurableResultConverter, enabled only by OpenCode Go completions builder. Retains last non-null snapshot per completed response, leaves generic deltas/global aggregation/Responses API untouched. Failed incomplete streams emit no buffered usage. No UI clamp or historical repair. Longer live diagnostic through Symfony platform + converter: 2354 chunks, only two usage reports at chunks2353/2354, both prompt56 completion8000; pre-fix metadata prompt112. Confirms duplicate-snapshot defect, NOT historical hundreds-million spikes. PR will describe partial scope and keep issue536 open.
- Ownership: owner=main; fork_run=none; revision=811587184; scope=usage normalization, focused regression validation and PR handoff; outcome=completed; commit=e39eaaffe

## Task workflow update - 2026-09-28T19:53:02+00:00
- Summary: Independent reviewer agent_5db4caf0d8abfbbc APPROVE on e39eaaffe. Confirmed provider scope, exact generic behavior preservation, metadata normalization, request independence, incomplete-stream handling and specification fidelity. No blocking findings. Retain explicit limitation: fixed confirmed duplicate/cumulative reports, historical extreme raw responses not captured. Failed or abandoned OpenCode streams no longer publish buffered partial snapshots. Ready for transition-owned full gate.
- Review: role=reviewer; artifact=agent_5db4caf0d8abfbbc; revision=e39eaaffe; scope=4-file OpenCode Go usage normalization diff; verdict=APPROVE.

## Task workflow update - 2026-09-28T19:55:08+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (106.2s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage/var/reports/qa-20260928-195322-1986-99f4345e.
- Session/run: 74.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-28T19:55:09+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage/var/reports/qa-20260928-195322-1986-99f4345e.
- Session/run: 74.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-28T19:55:12+00:00
- castor check passed (106.2s).
- Pushed task/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage to origin.
- Created PR: <url>
- Session/run: 74.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-28T19:55:12+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (106.2s).
- Pushed task/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/538

## Task workflow update - 2026-09-28T20:20:19+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage into integration checkout.
- Merge made by the 'ort' strategy.
 src/CodingAgent/Infrastructure/SymfonyAi/OpenCodeGo/OpenCodeGoSymfonyAiProviderBuilder.php |  3 ++-
 src/Platform/Bridge/Generic/DurableResultConverter.php                                     | 32 +++++++++++++++++++++++++-------
 tests/CodingAgent/Infrastructure/SymfonyAi/SymfonyAiProviderFactoryTest.php                | 75 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Platform/Bridge/Generic/DurableResultConverterTest.php                               | 19 +++++++++++++++++++
 4 files changed, 121 insertions(+), 8 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR #538 merged as 858983a39f362677121d87f84058a825dd028c32. Completed targeted snapshot normalization; issue #536 remains unresolved for historical extreme counts without raw evidence. Running separate post-merge integration validation.

## Task workflow update - 2026-09-28T20:22:02+00:00
- Updated PR Status: merged
- Validation: castor check: all 11 lanes passed in 146.3s; reports var/reports/qa-20260928-202032-7191-cec93a08. Leak, artifact integrity and proxy cache checks passed.; Clean integration checkout and removed task worktree verified.
- Summary: Post-merge full castor check passed on integration revision 95ab949b904e301f97e667c6d33774289ed51c0b. Clean Git status; task worktree removed. Confirmed duplicate usage fix completed, historical extreme counts still unproven. Symfony AI generic completions converter has the equivalent normalization opportunity; no upstream issue/PR created in this task.
