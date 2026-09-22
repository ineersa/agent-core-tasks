# Add cross-session OM text search and recall

## Goal
Enable agents to find prior work and recover its evidence using the existing observational-memory database. Motivating example: Symfony AI PR #2510 is explicitly recorded in session 32 observation 0ed52caa7dc430ffbe7472ff58610c3f96c09210c76f7a5238052c22ea30e28e, pointing to event 499. The project OM database currently contains 4,942 observations across 38 sessions; session 32 has 85 observations.

Scope: text search over retained observations and reflections in the configured OM database, plus explicit cross-session recall of selected results. Reuse existing source references and canonical session events. No separate summary-generation system, embeddings, semantic search, or cross-project database aggregation in this task.

Current recall is SQL-scoped to the current run in MemoryGenerationRepository; do not assume cross-session retrieval already works. Inspect the recall service and ExtensionApi source-event access before defining the change.

Design decisions to finalize before implementation: search default time scope, meaning of time filters, bounded result contract, and explicit originating-session parameter for recall. User suggested sessions touched in the last seven days with an explicit way to search older work, but did not finalize that default. Choose SQL text search versus SQLite full-text indexing from measured representative queries and growth probes, not an assumed performance problem. Keep identifiers such as PR numbers, URLs, branches, and symbols searchable.

Entry points: .hatfield/extensions/observational-memory/README.md, src/ObservationalMemoryExtension.php, src/Storage/MemoryGenerationRepository.php within that extension. Follow extension boundaries and existing storage facilities.

## Acceptance criteria
- An agent can locate prior-session observations and reflections by text, including PR identifiers and code symbols.
- Search results are bounded and identify the source session, memory ID, timestamp, and matching content.
- Explicit cross-session recall recovers the selected memory and its supporting evidence from the originating session; omitted session selection retains current-session behavior.
- Retained historical memory is searchable, not only the active compaction pool.
- Time-filter semantics and default scope are explicitly agreed before implementation; older memories remain discoverable.
- Document query-plan and timing evidence on representative data and a larger disposable corpus to justify the smallest search implementation.
- Use deterministic focused validation for search isolation, result bounds, ambiguous IDs, and source-session recall; follow project Castor requirements.
- Document the tools and limits, including incomplete OM coverage and stale historical context.

## Workflow metadata
Status: DONE
Branch: task/2026-09-12-add-cross-session-om-text-search-and-recall
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/496
PR Status: merged
Started: 2026-09-12T22:19:00+00:00
Completed: 2026-09-13T23:27:06+00:00

## Work log
- Created: 2026-09-12T22:13:57+00:00

## Task workflow update - 2026-09-12T22:18:36+00:00
- Summary: User finalized discovery policy: search all retained OM history by default, with an optional filter on memory date. Do not restrict the default to sessions touched in the last seven days. Search output remains bounded. Semantic retrieval is tracked separately in 2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections.

## Task workflow update - 2026-09-12T22:19:00+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-12-add-cross-session-om-text-search-and-recall.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall.

## Task workflow update - 2026-09-12T22:19:38+00:00
- Summary: Routing: bounded extension-owned slice delegated because query/index growth investigation and existing contextual tool integration require substantial isolated iteration. Main inspected RecallToolHandler, OmQueryService, storage and existing kernel-based RecallToolHandlerTest. Existing query service already reads referenced events through public session API. Fork owns detailed implementation; main reviews output. No semantic retrieval in this slice.
- Ownership: owner=fork; fork_run=none; revision=1ca7fec401f6d8a07300ad280d566ecaae1fd930; scope=OM cross-session text search and recall including focused validation; outcome=assigned; commit=none

## Task workflow update - 2026-09-12T22:37:50+00:00
- Validation: Focused Castor tests PASS: 10 tests, 195 assertions.; Scoped Castor phpstan PASS; scoped cs-fix PASS.; docs:validate passed initial implementation; final benchmark documentation added afterward.; Full castor check not run: task-to-pr transition owns gate.; Disposable search probe: approximately 1ms at 4.9k rows; 12–16ms at 50k rows, with leading-wildcard table scan. Benchmark report committed in extension docs.
- Summary: Implemented om_search(query, after?, before?, limit?) over all retained OM observations/reflections and recall(id, session_id?) with explicit originating-session provenance. Default result limit 20, maximum 50. Escaped SQLite LIKE retained based on measured current and 50k-row performance; semantic follow-up remains TODO. Parent inspection caught truncation, mixed timestamp ordering, date-bound validation, and inaccurate case-sensitivity claims; fork corrected these with regression coverage in 054e4f08. Full gate and independent review remain next-phase work.
- Ownership: owner=fork; fork_run=agent_0028d55d3cc7c34f; revision=6c8d2f66c1a8e581a49707e8b2f39c2564ad6f87; scope=parent-requested search correctness fixes in task worktree; outcome=assigned; commit=6c8d2f66c1a8e581a49707e8b2f39c2564ad6f87
- Ownership: owner=fork; fork_run=agent_0028d55d3cc7c34f; revision=054e4f08c0df1169482da41f0680a5998bd63125; scope=OM cross-session text search and recall including focused validation; outcome=completed; commit=054e4f08c0df1169482da41f0680a5998bd63125

## Task workflow update - 2026-09-13T02:48:25+00:00
- Validation: Reused focused 10 tests/195 assertions and scoped phpstan for unchanged revision.; castor test:llm-real --filter=LlamaCppSmokeTest PASS, 1 test/8 assertions, 1.1s.
- Summary: Independent reviewer agent_d02e712bca6283e8 reviewed revision 054e4f08c0df1169482da41f0680a5998bd63125 against specification, security, correctness, and minimality: APPROVE WITH SUGGESTIONS. Follow-up clarified no actual SEC findings; remaining suggestions cosmetic/optional, deferred without scope expansion. Clean worktree. No unresolved blockers; full gate pending transition.
- Review: role=reviewer; artifact=agent_d02e712bca6283e8; revision=054e4f08c0df1169482da41f0680a5998bd63125; scope=origin/main...HEAD OM search/recall specification-fidelity and correctness; verdict=APPROVE WITH SUGGESTIONS

## Task workflow update - 2026-09-13T02:50:55+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (139.1s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall/var/reports/qa-20260913-024836-5470-b6eb1d93.
- Session/run: 44.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-13T02:50:57+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-12-add-cross-session-om-text-search-and-recall to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall/var/reports/qa-20260913-024836-5470-b6eb1d93.
- Session/run: 44.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-13T02:50:59+00:00
- castor check passed (139.1s).
- Pushed task/2026-09-12-add-cross-session-om-text-search-and-recall to origin.
- Created PR: <url>
- Session/run: 44.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-13T02:50:59+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (139.1s).
- Pushed task/2026-09-12-add-cross-session-om-text-search-and-recall to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/496

## Task workflow update - 2026-09-13T02:57:12+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User approved renaming om_search to memory_search and refining descriptions/guidelines: prior-work discovery trigger, literal keyword strategy, selected-evidence recall, incomplete coverage and stale-context warnings. No behavioral expansion.

## Task workflow update - 2026-09-13T02:57:18+00:00
- Summary: Resume existing owner for bounded revision using its registration/docs/test context; no independent redesign.
- Ownership: owner=fork; fork_run=agent_0028d55d3cc7c34f; revision=054e4f08c; scope=approved tool rename and guidance refinement with matching docs/tests; outcome=assigned; commit=none

## Task workflow update - 2026-09-13T03:01:16+00:00
- Validation: Focused Castor tests PASS 10 tests/195 assertions.; Scoped formatting/static analysis and docs:validate PASS.
- Summary: Renamed tool to memory_search without alias and refined discovery, literal-query, provenance, incomplete-memory, and stale-context guidance. Reviewer agent_d02e712bca6283e8 APPROVE at 0a2cf2ba5c5330b5cdccb8633e60fd9f512941d2. No blockers.
- Ownership: owner=fork; fork_run=agent_0028d55d3cc7c34f; revision=0a2cf2ba5c5330b5cdccb8633e60fd9f512941d2; scope=approved tool rename and guidance refinement; outcome=completed; commit=0a2cf2ba5c5330b5cdccb8633e60fd9f512941d2
- Review: role=reviewer; artifact=agent_d02e712bca6283e8; revision=0a2cf2ba5c5330b5cdccb8633e60fd9f512941d2; scope=rename and guideline specification fidelity; verdict=APPROVE

## Task workflow update - 2026-09-13T03:02:22+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (55.2s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall/var/reports/qa-20260913-030127-11376-6e326d82.
- Session/run: 44.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-13T03:02:23+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-12-add-cross-session-om-text-search-and-recall to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall/var/reports/qa-20260913-030127-11376-6e326d82.
- Session/run: 44.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-13T03:02:24+00:00
- castor check passed (55.2s).
- Pushed task/2026-09-12-add-cross-session-om-text-search-and-recall to origin.
- PR already exists: <url>
- Session/run: 44.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-13T03:02:24+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (55.2s).
- Pushed task/2026-09-12-add-cross-session-om-text-search-and-recall to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/496

## Task workflow update - 2026-09-13T17:53:07+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Address user test feedback: actionable selected-session recall error; importance instead of relevance in tool output; explicit contiguous phrase, newest-first/truncation, memory-content-only search guidance. No new pagination/filter/compact-mode APIs.

## Task workflow update - 2026-09-13T17:53:19+00:00
- Ownership: owner=fork; fork_run=agent_0028d55d3cc7c34f; revision=0a2cf2ba5; scope=user-tested recall error and search guidance corrections; outcome=assigned; commit=none

## Task workflow update - 2026-09-13T18:04:40+00:00
- Validation: Focused Castor tests PASS 10 tests/213 assertions; scoped static, style, docs PASS.
- Summary: Implemented user test feedback at 8229e904c: importance output, actionable selected-session not_found, phrase matching/newest-first/truncation/content-only guidance. New owner/reviewer used because previous artifacts rejected resume after parent lifetime change. Reviewer agent_d5ce25f2ecb1c0d4 APPROVE; no blockers.
- Ownership: owner=fork; fork_run=agent_d11c88e82ebf0ea3; revision=0a2cf2ba5; scope=replacement owner for user feedback after resume rejection; outcome=assigned; commit=none
- Ownership: owner=fork; fork_run=agent_d11c88e82ebf0ea3; revision=8229e904cbaa9ad7d2c8e3d74473d085eeffb085; scope=user test feedback corrections; outcome=completed; commit=8229e904cbaa9ad7d2c8e3d74473d085eeffb085
- Review: role=reviewer; artifact=agent_d5ce25f2ecb1c0d4; revision=8229e904c; scope=feedback delta specification fidelity; verdict=APPROVE

## Task workflow update - 2026-09-13T18:05:52+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-12-add-cross-session-om-text-search-and-recall.md.
- Session/run: 44.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall/var/reports/qa-20260913-180450-1150-5474447d.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-13T18:10:47+00:00
- Validation: castor clean:cleanup:workers:list: no stale QA workers.; Gate failure log var/reports/qa-20260913-180450-1150-5474447d/check-test:controller-replay.log; Feedback revision 8229e904c remains locally committed, independently approved; not pushed by failed transition.
- Summary: CODE-REVIEW transition blocked before push by controller-replay CancelDuringActiveBashThenFollowUp failure: bash never started, only ack/run.started. Read-only diagnosis artifact agent_d11c88e82ebf0ea3 found base harness excludes bash and test does not override, suggesting a pre-existing test configuration gap; earlier gates passed so causal explanation needs further verification before modifying unrelated tests. Copied OM data excluded by test isolation. No blind retry or unrelated code change made.

## Task workflow update - 2026-09-13T18:59:58+00:00
- Summary: User authorized gate fix. Prior bash-exclusion diagnosis disproved by successful fixture execution with same exclusion. Failed artifacts show undelivered ExecuteLlmStep in isolated llm queue; exact cause still under investigation. No code changed.
- Ownership: owner=fork; fork_run=agent_d11c88e82ebf0ea3; revision=8229e904c; scope=diagnose and fix full-gate controller-replay non-delivery with contention proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-13T19:24:11+00:00
- Validation: Controller replay PASS 11 tests/175 assertions.; ConsumerSupervisor restored baseline tests PASS 6/51.; Scoped test-base phpstan exposes existing errors outside delta; full configured gate pending.
- Summary: Gate correction now limited to two E2E harness files: isolated controller cache via shared env hook and accurate refreshed diagnostics. Removed speculative production env changes after reviewer verified Symfony already inherits env. Original flake not conclusively reproduced; actual shared-cache isolation defect addressed. Reviewer agent_d5ce25f2ecb1c0d4 APPROVE at dcbb265c5.
- Ownership: owner=fork; fork_run=agent_fce471272d0d0d73; revision=9609a4cd2; scope=reduce gate fix after previous owner context limit; outcome=assigned; commit=none
- Ownership: owner=fork; fork_run=agent_fce471272d0d0d73; revision=dcbb265c5bda215ee78031a7288ca18f1f4d9bc8; scope=gate harness isolation and diagnostics; outcome=completed; commit=dcbb265c5bda215ee78031a7288ca18f1f4d9bc8
- Review: role=reviewer; artifact=agent_d5ce25f2ecb1c0d4; revision=dcbb265c5; scope=net gate fix; verdict=APPROVE

## Task workflow update - 2026-09-13T19:25:33+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (73.4s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall/var/reports/qa-20260913-192420-19699-3923bc85.
- Session/run: 44.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-13T19:25:35+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-12-add-cross-session-om-text-search-and-recall to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall/var/reports/qa-20260913-192420-19699-3923bc85.
- Session/run: 44.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-13T19:25:36+00:00
- castor check passed (73.4s).
- Pushed task/2026-09-12-add-cross-session-om-text-search-and-recall to origin.
- PR already exists: <url>
- Session/run: 44.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-13T19:25:36+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (73.4s).
- Pushed task/2026-09-12-add-cross-session-om-text-search-and-recall to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/496
- Summary: Independent review approved OM feedback revision and minimal controller E2E cache isolation fix. Re-run transition gate on dcbb265c5.

## Task workflow update - 2026-09-13T23:27:06+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall: ide_close_project returned isError.
- Merged task/2026-09-12-add-cross-session-om-text-search-and-recall into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/extensions/observational-memory/README.md                                              |  42 ++++++++--
 .hatfield/extensions/observational-memory/docs/om-search-like-benchmark.md                       | 129 ++++++++++++++++++++++++++++++
 .hatfield/extensions/observational-memory/src/ObservationalMemoryExtension.php                   |  74 ++++++++++++++---
 .hatfield/extensions/observational-memory/src/Query/OmQueryService.php                           | 324 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++------
 .hatfield/extensions/observational-memory/src/Storage/MemoryGenerationRepository.php             |  65 +++++++++++++++
 .hatfield/extensions/observational-memory/src/Storage/ObservationRepository.php                  |  68 ++++++++++++++++
 .hatfield/extensions/observational-memory/src/Tool/RecallToolHandler.php                         |  15 +++-
 .hatfield/extensions/observational-memory/src/Tool/SearchToolHandler.php                         |  98 +++++++++++++++++++++++
 .hatfield/extensions/observational-memory/tests/ObservationalMemoryExtensionRegistrationTest.php |  42 +++++++---
 .hatfield/extensions/observational-memory/tests/OmQueryServiceTest.php                           | 515 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 .hatfield/extensions/observational-memory/tests/RecallToolHandlerTest.php                        | 104 ++++++++++++++++++++++++
 docs/tools.md                                                                                    |   2 +-
 tests/CodingAgent/Runtime/Controller/E2E/ControllerE2eTestCase.php                               |   9 ++-
 tests/CodingAgent/Runtime/Controller/E2E/ControllerReplayE2eTestCase.php                         |   2 +
 14 files changed, 1434 insertions(+), 55 deletions(-)
 create mode 100644 .hatfield/extensions/observational-memory/docs/om-search-like-benchmark.md
 create mode 100644 .hatfield/extensions/observational-memory/src/Tool/SearchToolHandler.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-cross-session-om-text-search-and-recall.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR #496 merged as 14bbf6ccbb87dcda89c096f81cd75e51df892fd1. Integrate and perform post-merge validation.

## Task workflow update - 2026-09-13T23:28:18+00:00
- Updated PR Status: merged
- Validation: Post-merge castor check PASS 136.2s, all 10 lanes, artifact integrity/leak/cache guards pass.; QA reports: var/reports/qa-20260913-232714-26038-2e196d70; git status --short empty; worktree directory absent.
- Summary: DONE. Integration revision 56dd66852acb5bfb0e016b152a025f52c1d53097 validated; clean Git status and task worktree removed. IDE close reported degraded but filesystem cleanup succeeded. Initial command supervision lost status; waited on existing process, recovered final quality: ok from log without rerunning.
