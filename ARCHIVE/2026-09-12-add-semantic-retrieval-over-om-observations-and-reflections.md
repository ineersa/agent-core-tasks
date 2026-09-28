# Add semantic retrieval over OM observations and reflections

## Goal
Follow-up to 2026-09-12-add-cross-session-om-text-search-and-recall. Add meaning-based discovery of prior work, for example finding session 32 from 'where we upstreamed flat DTO arguments' without knowing PR #2510 or MapToolArguments.

Reuse the base task's result provenance and cross-session recall. Investigate existing Symfony AI retrieval, embeddings, and vector-store facilities before proposing a design. The fact OM already invokes an agent does not by itself supply an embedding index. Avoid a second memory-summary pipeline.

Finalize provider/model selection, index storage, backfill and incremental updates, retention handling, and ranking integration after the base task is implemented. This task is deferred; no semantic implementation in the base task.

## Acceptance criteria
- Inspect Symfony AI facilities and propose the smallest compatible semantic retrieval design before implementation.
- Retrieve observations and reflections by meaning while preserving exact-text lookup for identifiers.
- Return source-session and memory references usable by cross-session recall.
- Define indexing ownership, backfill, updates, privacy implications, and failure behavior explicitly.
- Demonstrate retrieval quality and cost on representative prior-session queries, including the Symfony AI PR #2510 example.

## Workflow metadata
Status: DONE
Branch: task/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections
Fork run: agent_aad42ed4e483b03d
PR URL: https://github.com/ineersa/agent-core/pull/524
PR Status: merged
Started: 2026-09-22T13:06:09+00:00
Completed: 2026-09-23T19:57:37+00:00

## Work log
- Created: 2026-09-12T22:14:09+00:00

## Task workflow update - 2026-09-22T13:05:59+00:00
- Summary: User approved implementation of .hatfield/tmp/om-semantic-retrieval-plan.md and autonomous progression through task-to-pr; final deliverable is an open PR, not merge. Reuse this existing matching task rather than duplicate it.
- Final scope: one memory_search; current exact LIKE behavior without embeddings; with embeddings use Vektor HNSW + SQLite FTS5 BM25 + Symfony CombinedStore RRF + optional reranker. Fail configured stage failures, never silently degrade. Retain recall provenance and date/limit semantics. Async resumable indexing on existing extension_agent transport. No native extension or packaging changes. Tool descriptions differ for exact versus hybrid BM25+semantic capability; no RRF/reranker internals in model-facing description.
- Model configuration supplied by user: embedding http://localhost:8059/v1 model coderankembed-q8_0.gguf; prefix 'Represent this query for searching relevant code:'; Vera prepares prefix + space + query (verified source). Chunk bytes=1200, overlap bytes=192, max lines=80; embedding batch=4, concurrency=1; rerank http://localhost:8060/v1 model bge-reranker-base-q8_0.gguf, max batch=8, max doc chars=768. No import of Vera completion/code-indexing options. User's Vera config is endpoint/model constraint evidence; retain OM's existing result limits and finalized settings scope.
- Plan is an implementation proposal, not permission for unnecessary adapters or operations. Verify CombinedStore's text-query semantics, Vektor process-global state with multi-project jobs, crash consistency, cancellation and privacy before choosing exact edits. Full castor check is owned by CODE-REVIEW transition.

## Task workflow update - 2026-09-22T13:06:09+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections/.idea.

## Task workflow update - 2026-09-22T13:07:05+00:00
- Ownership: owner=fork; fork_run=none; revision=2715f242b62be7500850a56751a1923db4ee1da4; scope=extension-owned hybrid memory search, settings and descriptions, resumable indexing, focused proof and real-corpus benchmark; outcome=assigned; commit=none
- Routing complete: entry points ObservationalMemoryExtension registration, OmQueryService::search, Tool/SearchToolHandler, Runtime/OmSettings, OmDatabaseFactory/OmSchemaMigrator, Observer/ObserveBoundaryJobHandler and Compaction/ReflectGenerationJobHandler. Existing OM tests use IsolatedKernelTestCase and OmDatabaseFactoryTestService. No local OM AGENTS.md. Extension uses public ExtensionApi plus vendor facilities; minimal approved Symfony AI/HttpClient/Lock Deptrac edges may be needed. Main retains review, task transitions and PR ownership.

## Task workflow update - 2026-09-22T13:21:02+00:00
- Validation: Castor scratch probe on disposable 1000-row corpus: embeddings 2.747s, HNSW build116.266s, 200 self searches10.406s, peak77.13MiB including retained corpus; not final quality acceptance.; Castor deletion probe reproduced tombstone starvation: 80 vectors, delete nearest60, k10 returns0 despite20 live. Recovery must rebuild/compact before healthy.
- Ownership: owner=fork; fork_run=agent_6a751bef82bcf051; revision=2715f242b62be7500850a56751a1923db4ee1da4; scope=Vektor feasibility and failure-mode probes; outcome=completed; commit=none
- Ownership: owner=fork; fork_run=agent_6a751bef82bcf051; revision=2715f242b62be7500850a56751a1923db4ee1da4; scope=implement approved extension hybrid retrieval with durable index health and focused tests; outcome=assigned; commit=none

## Task workflow update - 2026-09-22T14:15:31+00:00
- Summary: Implementation committed at b4966eff9; focused Castor tests (82/841), phpstan, deptrac, dead-code, style and docs validation passed. Full-corpus relevance/cost report and independent review remain before PR.
- Ownership: owner=fork; fork_run=agent_6a751bef82bcf051; revision=2715f242b62be7500850a56751a1923db4ee1da4; scope=extension-owned hybrid retrieval and focused deterministic proof; outcome=completed; commit=b4966eff9c309bdb17433063fc5781876973ff37
- Ownership: owner=fork; fork_run=agent_6a751bef82bcf051; revision=b4966eff9c309bdb17433063fc5781876973ff37; scope=production-path full-corpus quality and cost benchmark with supplied endpoints and durable report; outcome=assigned; commit=none

## Task workflow update - 2026-09-22T14:47:03+00:00
- Summary: Independent reviewer agent_98f56b935ceb6d35 reviewed 7da27b77c and requested changes: date-filtered vector retrieval must have a hard candidate cap, transient read errors must not dirty/rebuild the entire index, expose planned index progress, remove unsupported reranker query-prefix setting. Full benchmark still running; no second build started.
- Review: role=reviewer; artifact=agent_98f56b935ceb6d35; revision=7da27b77c; scope=full diff, vendor contracts, specification fidelity, test quality; verdict=REQUEST CHANGES
- Ownership: owner=fork; fork_run=agent_6a751bef82bcf051; revision=7da27b77c; scope=review fixes for bounded filtered retrieval, transient read errors, progress counts, unsupported reranker prefix; outcome=assigned; commit=none

## Task workflow update - 2026-09-22T15:08:53+00:00
- Recorded fork run: 699f349b-219e-5189-bf2b-562ec050261b
- Validation: castor test --filter=ObservationalMemory:86 tests886 assertions PASS; castor phpstan/deptrac/cs-check/docs:validate PASS; Full-corpus benchmark and final query-only --reuse PASS; report .hatfield/extensions/observational-memory/docs/om-semantic-retrieval-benchmark.md; live corpus never modified
- Summary: Review fixes and full-corpus report committed at 2bb0bc879. Backfill 9352 memories/9364 chunks in32m25s, max batch2.262s; paraphrase finds original session32 rank1,171ms without reranker/440ms with. Report explicitly documents quadratic source traversal, bounded date recall and no absence threshold. Re-review pending.
- Ownership: owner=fork; fork_run=699f349b-219e-5189-bf2b-562ec050261b; revision=7da27b77c; scope=bounded candidate retrieval, transient failures, progress, raw reranker query, shortened lock scopes, full-corpus report; outcome=completed; commit=2bb0bc879fd3a0f99079506b96a6f93ee28b03d8
- Review: role=reviewer; artifact=agent_98f56b935ceb6d35; revision=2bb0bc879; scope=review fixes and durable full-corpus evidence; verdict=pending

## Task workflow update - 2026-09-22T15:16:05+00:00
- Validation: castor test:llm-real PASS (5 tests30 assertions,5.6s); proxy cache warmed before transition; Same-revision focused proof reused:86 OM tests886 assertions, phpstan/deptrac/style/docs; endpoint smoke and full-corpus query proof documented
- Summary: Independent reviewer agent_98f56b935ceb6d35 APPROVE at2bb0bc879. All prior blockers resolved and full-corpus report cross-checked against JSON artifacts. No unresolved PR blockers; initial indexing cost and bounded dated recall documented. Ready for CODE-REVIEW gate.
- Review: role=reviewer; artifact=agent_98f56b935ceb6d35; revision=2bb0bc879fd3a0f99079506b96a6f93ee28b03d8; scope=review fixes, spec fidelity, vendor failure contracts, benchmark evidence; verdict=APPROVE

## Task workflow update - 2026-09-22T15:17:27+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (56.6s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections/var/reports/qa-20260922-151631-6597-2250961a.
- Session/run: 63.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-22T15:17:30+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections/var/reports/qa-20260922-151631-6597-2250961a.
- Session/run: 63.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-22T15:17:33+00:00
- castor check passed (56.6s).
- Pushed task/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections to origin.
- Created PR: <url>
- Session/run: 63.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-22T15:17:33+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (56.6s).
- Pushed task/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/524

## Task workflow update - 2026-09-22T15:30:50+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User reports irrelevant search results and requires a minimum relevance cutoff. Reopen PR524 for calibration on existing disposable corpus, then bounded filtering with deterministic proof. Preserve user-local enabled semantic settings.

## Task workflow update - 2026-09-22T15:31:01+00:00
- Ownership: owner=fork; fork_run=699f349b-219e-5189-bf2b-562ec050261b; revision=2bb0bc879; scope=read-only relevance calibration using existing disposable benchmark index and supplied endpoints; outcome=assigned; commit=none

## Task workflow update - 2026-09-22T15:38:13+00:00
- Summary: User explicitly requests calibration using 20–30 selected observations and prepared questions, plus a configurable relevance threshold. Initial 11-query probe found raw reranker scores −10.197..5.526, not probabilities; a single cutoff has precision/recall tradeoffs. Expand to25 diverse source observations with held-out evaluation and negative queries before choosing local model cutoff.
- Ownership: owner=fork; fork_run=699f349b-219e-5189-bf2b-562ec050261b; revision=2bb0bc879; scope=read-only relevance calibration using existing disposable benchmark index and supplied endpoints; outcome=completed; commit=none
- Ownership: owner=fork; fork_run=699f349b-219e-5189-bf2b-562ec050261b; revision=2bb0bc879; scope=25-observation calibration dataset and evidence plus configurable reranker relevance floor and focused proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-23T15:22:56+00:00
- Validation: castor test --filter=ObservationalMemory PASS92 tests905 assertions; castor phpstan PASS; castor cs-check PASS; castor docs:validate PASS; git diff --cached --check PASS
- Summary: Calibration dataset tracked under OM docs:25 source observations with paraphrased questions, eight negative questions, session/ID labels and frozen splits. At -4 raw score, first-ten judged precision calibration 43.7→67.0%, heldout15.0→21.6%; heldout topic hit8→6/10 and unrelated negative hits4→2/4. Configurable finite reranker_api.min_score implemented at03ba2ceff, local worktree setting enabled at -4 remains deliberately uncommitted.
- Ownership: owner=fork; fork_run=699f349b-219e-5189-bf2b-562ec050261b; revision=2bb0bc879; scope=25-observation calibration dataset and evidence plus configurable reranker relevance floor and focused proof; outcome=blocked; commit=none
- Ownership: owner=main; fork_run=none; revision=2bb0bc879; scope=continue interrupted fork artifacts, persist frozen questions and calibration report under OM git, finish floor tests/docs/commit; outcome=completed; commit=03ba2ceff
- Review: role=reviewer; artifact=agent_98f56b935ceb6d35; revision=03ba2ceff; scope=threshold spec fidelity and calibration evidence; verdict=pending

## Task workflow update - 2026-09-23T15:40:54+00:00
- Validation: castor test --filter=ObservationalMemory PASS93 tests984 assertions; castor phpstan PASS; castor cs-check PASS; castor docs:validate PASS; Mutating candidate-cap return to post-filter count caused targeted test failure; restoring production passed; targeted case1.3s
- Summary: Reviewer agent_2166f9385542ab76 approved score cutoff and calibration; reviewer-found candidate-cap regression test weakness fixed and mutation-verified (fails on reverted production line, passes restored), final comment rationale restored at1590c40e7. Local active settings excluded from commits and will be preserved through transition gate.
- Review: role=reviewer; artifact=agent_2166f9385542ab76; revision=1590c40e7; scope=reranker floor, calibration dataset, truncation fix and discriminating test, specification fidelity; verdict=APPROVE

## Task workflow update - 2026-09-23T15:42:22+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (61.2s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections/var/reports/qa-20260923-154121-3186-06804794.
- Session/run: 63.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-23T15:42:23+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections/var/reports/qa-20260923-154121-3186-06804794.
- Session/run: 63.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-23T15:42:24+00:00
- castor check passed (61.2s).
- Pushed task/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections to origin.
- PR already exists: <url>
- Session/run: 63.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-23T15:42:24+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (61.2s).
- Pushed task/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/524
- Summary: Add opt-in calibrated reranker min_score for PR #524 based on 25 saved source-observation questions and eight negatives. Model-specific -4 floor improves judged first-ten precision but still admits irrelevant matches and loses some relevant results; both tradeoffs documented. Active worktree settings temporarily backed up to ignored .hatfield/tmp/om-calibration25/enabled-settings-backup.yaml to meet clean gate; restore immediately afterward.

## Task workflow update - 2026-09-23T16:41:55+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User requested expand saved OM calibration set from 33 to up to 50 questions and calibrate pre-rerank vector/BM25 candidate budgets, fused RRF ranking and relevance floor. PR #524 remains open. Preserve local enabled worktree settings as uncommitted; use disposable index, not canonical live OM.

## Task workflow update - 2026-09-23T16:42:09+00:00
- Summary: Routing: existing dataset docs/relevance-calibration-questions.json (25 positives+8 negatives); source MemoryStoreAdapter supplies up to100 per store; CombinedStore RRF k60; SemanticIndexService reranks fused top100, then min_score -4. Focus calibration of candidate recall@k before changing limits. Use existing private fully indexed corpus and local endpoints; no reindex. One bounded evaluation/doc slice delegated; main reviews data and decides minimal production change.
- Ownership: owner=fork; fork_run=none; revision=1590c40e7; scope=expand frozen OM questions from 33 to 50, measure pre-rerank vector/BM25 and RRF candidate recall/cost plus model score-floor outcomes using existing disposable corpus, update calibration evidence; outcome=assigned; commit=none

## Task workflow update - 2026-09-23T16:58:53+00:00
- Summary: 50-question frozen set and candidate-stage report committed at03c87eb1d; initial analysis found fused top200 recovers five held-out targets as candidates, but final top20 gain with floor -4 is only 8→10/16 and +~270ms rerank. Additional blinded N200 judgment needed before changing production. Prior fork cannot resume (context limit); assign narrow continuation.
- Ownership: owner=fork; fork_run=agent_65faca2dee4152ba; revision=1590c40e7; scope=expand frozen OM questions from33 to50 and measure candidate stages/score on disposable corpus; outcome=completed; commit=03c87eb1d
- Ownership: owner=fork; fork_run=none; revision=03c87eb1d; scope=validate N200 score capture and blind newly surfaced candidates, report actual tool-visible top10/top20 precision and recall before production choice; outcome=assigned; commit=none

## Task workflow update - 2026-09-23T17:09:31+00:00
- Recorded fork run: agent_aad42ed4e483b03d
- Validation: Disposable-corpus blinded N200 judgment, negative checks and duplicate-score-collision audit, no reindex; castor docs:validate PASS
- Summary: 50-question calibration shows held-out exact target@20 with floor -4 rises 8/16→10/16 for fused100→200; judged lower-bound held-out precision@10 18.0→24.0%, topic11→12/16; negative-with-hits remains2/6; rerank cost13→25 HTTP batches, about+270ms. RRF k20/60/100 unchanged recall; keep vendor k60, per-store100, floor optional -4. Implement minimal fused200 constant with regression proof.
- Ownership: owner=fork; fork_run=agent_aad42ed4e483b03d; revision=03c87eb1d; scope=validate top10/top20 user-visible N200 gains and score capture collisions; outcome=completed; commit=7ac71571b
- Ownership: owner=main; fork_run=none; revision=7ac71571b; scope=raise bounded fused candidate cutoff 100→200, update deterministic user-visible candidate/floor proof and docs, preserve local config; outcome=assigned; commit=none

## Task workflow update - 2026-09-23T17:14:15+00:00
- Validation: castor test --filter=ObservationalMemory PASS93 tests1059 assertions; castor phpstan PASS; castor cs-check PASS; castor docs:validate PASS; Targeted N200 regression1.8s and deliberate cap100 mutation FAIL(67 vs100), restored N200 PASS
- Summary: At e08a69d26 raised fused chunk cap100→200 while keeping100 per store/RRFk60 and score floor optional. Integration regression indexes two disjoint 100-doc streams; full fused200 returns100 lexical parents after rerank while temporary cap100 returned67 (mutation fails). 93 OM tests1059 assertions; phpstan/cs/docs pass. Reviewer pending.
- Ownership: owner=main; fork_run=none; revision=7ac71571b; scope=raise bounded fused candidate cutoff100→200, update deterministic user-visible candidate/floor proof and docs; outcome=completed; commit=e08a69d26
- Review: role=reviewer; artifact=pending; revision=e08a69d26; scope=expanded dataset, calibration methodology, fused cap/proof, spec fidelity; verdict=pending

## Task workflow update - 2026-09-23T17:25:19+00:00
- Validation: castor test --filter=ObservationalMemory PASS93 tests1059 assertions at e08a69d26; targeted cap regression PASS157 assertions at96fc28244; castor phpstan/cs-check/docs:validate PASS on production revision; updated doc and targeted test revalidated; Mutation cap100 causes test FAIL67 vs100; cap200 restored PASS
- Summary: Expanded 50-question calibration and production fused cap200 complete. Independent reviewer agent_7ec06e82ef16fe8a APPROVE revision96fc28244 after preserving zero-survivor truncation contract and historic benchmark wording. No unresolved blockers; local enabled settings will be backed up/restored around clean CODE-REVIEW gate.
- Ownership: owner=main; fork_run=none; revision=e08a69d26; scope=preserve empty-survivor truncation test and clarify historical benchmark request count; outcome=completed; commit=96fc28244
- Review: role=reviewer; artifact=agent_7ec06e82ef16fe8a; revision=96fc28244; scope=expanded50-case dataset, blinded judgments, fused candidate200, score floor/truncation, spec fidelity; verdict=APPROVE

## Task workflow update - 2026-09-23T17:27:09+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (59.0s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections/var/reports/qa-20260923-172611-11748-bb1790a7.
- Session/run: 63.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-23T17:27:11+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections/var/reports/qa-20260923-172611-11748-bb1790a7.
- Session/run: 63.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-23T17:27:12+00:00
- castor check passed (59.0s).
- Pushed task/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections to origin.
- PR already exists: <url>
- Session/run: 63.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-23T17:27:12+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (59.0s).
- Pushed task/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/524
- Validation: castor test --filter=ObservationalMemory PASS93/1059; castor phpstan, cs-check, docs:validate PASS; castor test:llm-real PASS5/30; Reviewer agent_7ec06e82ef16fe8a approved96fc28244; no blockers
- Summary: Expanded OM calibration to 50 saved questions and audited vector/BM25/RRF candidate ranks and raw reranker floor. Widen fused candidate cap100→200, retain per-store100/RRFk60/min_score optional. Local enabled worktree settings backed up at .hatfield/tmp/om-calibration50/enabled-settings-backup.yaml for clean full-QA gate, restore afterward.

## Task workflow update - 2026-09-23T18:24:11+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Address PR #524 inline comment discussion_r4085837755: extension-specific semantic settings documentation belongs in distributed extension docs, not core docs/settings.md. Keep implementation and local enabled settings intact.

## Task workflow update - 2026-09-23T18:25:13+00:00
- Summary: Accepted owner comment on docs/settings.md: remove extension-specific semantic settings reference from core docs; retain and complete extension README packaged with observational-memory. Scope docs only; no runtime, public API, or configuration changes.
- Ownership: owner=main; fork_run=none; revision=96fc28244; scope=move OM semantic settings reference from core docs/settings.md to distributed OM README; outcome=assigned; commit=none

## Task workflow update - 2026-09-23T18:27:11+00:00
- Validation: castor docs:validate PASS (21 built-in docs); git diff --cached --check PASS
- Summary: Moved all OM semantic setting reference keys from core docs/settings.md to distributed OM README; preserved package README's indexing/privacy section and added concise key/default table. Documentation validation passed. Local enabled settings untouched.
- Ownership: owner=main; fork_run=none; revision=96fc28244; scope=move OM semantic settings reference from core docs/settings.md to distributed OM README; outcome=completed; commit=439e2d1f4

## Task workflow update - 2026-09-23T18:29:45+00:00
- Validation: castor docs:validate PASS; Reviewer agent_7ec06e82ef16fe8a APPROVE revision439e2d1f4
- Summary: Review response revision439e2d1f4 approved independently, no blockers. Documentation lives in mirrored extension README rather than core settings guide; no runtime changes.
- Review: role=reviewer; artifact=agent_7ec06e82ef16fe8a; revision=439e2d1f4; scope=PR discussion_r4085837755 docs ownership, extension package distribution, key/default accuracy, specification fidelity; verdict=APPROVE

## Task workflow update - 2026-09-23T18:31:09+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (56.9s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections/var/reports/qa-20260923-183013-18512-11af60a4.
- Session/run: 63.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-23T18:31:11+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections/var/reports/qa-20260923-183013-18512-11af60a4.
- Session/run: 63.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-23T18:31:12+00:00
- castor check passed (56.9s).
- Pushed task/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections to origin.
- PR already exists: <url>
- Session/run: 63.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-23T18:31:12+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (56.9s).
- Pushed task/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/524
- Validation: castor docs:validate PASS; git diff --check PASS; Reviewer agent_7ec06e82ef16fe8a APPROVE revision439e2d1f4; castor test:llm-real PASS5 tests30 assertions (gate warmup)
- Summary: Resolved PR #524 comment discussion_r4085837755 by relocating extension-specific semantic settings documentation into distributed OM README. Core docs/settings.md has no OM section. Revision439e2d1f4 reviewed APPROVE; local worktree semantic config restored to clean baseline only for QA gate (backed up under .hatfield/tmp/om-calibration50/enabled-settings-backup.yaml), then will be restored.

## Task workflow update - 2026-09-23T19:57:37+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections into integration checkout.
- Auto-merging .hatfield/settings.yaml
Merge made by the 'ort' strategy.
 .castor/om-semantic.php                                                                          | 207 ++++++++++++++++++++++++++++++++++++++++++++++++++++++
 .hatfield/extensions/observational-memory/README.md                                              | 141 +++++++++++++++++++++++++++++++++++--
 .hatfield/extensions/observational-memory/composer.json                                          |  12 +++-
 .hatfield/extensions/observational-memory/docs/om-relevance-calibration.md                       | 119 +++++++++++++++++++++++++++++++
 .hatfield/extensions/observational-memory/docs/om-semantic-retrieval-benchmark.md                | 113 ++++++++++++++++++++++++++++++
 .hatfield/extensions/observational-memory/docs/relevance-calibration-questions.json              | 452 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 .hatfield/extensions/observational-memory/src/Compaction/ReflectGenerationJobHandler.php         |   6 ++
 .hatfield/extensions/observational-memory/src/ObservationalMemoryExtension.php                   |  21 ++++--
 .hatfield/extensions/observational-memory/src/Observer/ObserveBoundaryJobHandler.php             |   2 +
 .hatfield/extensions/observational-memory/src/Query/OmQueryService.php                           |  50 ++++++++++++-
 .hatfield/extensions/observational-memory/src/Runtime/OmSettings.php                             |   4 ++
 .hatfield/extensions/observational-memory/src/Semantic/MemoryChunker.php                         |  44 ++++++++++++
 .hatfield/extensions/observational-memory/src/Semantic/MemoryStoreAdapter.php                    |  91 ++++++++++++++++++++++++
 .hatfield/extensions/observational-memory/src/Semantic/SearchInterruptedException.php            |  14 ++++
 .hatfield/extensions/observational-memory/src/Semantic/SemanticApiClient.php                     | 136 +++++++++++++++++++++++++++++++++++
 .hatfield/extensions/observational-memory/src/Semantic/SemanticIndexJobHandler.php               |  68 ++++++++++++++++++
 .hatfield/extensions/observational-memory/src/Semantic/SemanticIndexService.php                  | 298 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 .hatfield/extensions/observational-memory/src/Semantic/SemanticIndexStartupHook.php              |  22 ++++++
 .hatfield/extensions/observational-memory/src/Semantic/SemanticSearchException.php               |  15 ++++
 .hatfield/extensions/observational-memory/src/Semantic/SemanticSettings.php                      | 103 +++++++++++++++++++++++++++
 .hatfield/extensions/observational-memory/src/Storage/OmSchemaMigrator.php                       |  11 +++
 .hatfield/extensions/observational-memory/src/Tool/SearchToolHandler.php                         |   3 +-
 .hatfield/extensions/observational-memory/tests/MemoryChunkerTest.php                            |  32 +++++++++
 .hatfield/extensions/observational-memory/tests/MemoryStoreAdapterTest.php                       |  44 ++++++++++++
 .hatfield/extensions/observational-memory/tests/ObservationalMemoryExtensionRegistrationTest.php |  36 ++++++++++
 .hatfield/extensions/observational-memory/tests/ObserveBoundaryJobHandlerTest.php                |  19 ++++-
 .hatfield/extensions/observational-memory/tests/ReflectGenerationJobHandlerTest.php              |  17 +++++
 .hatfield/extensions/observational-memory/tests/SemanticApiClientTest.php                        |  93 ++++++++++++++++++++++++
 .hatfield/extensions/observational-memory/tests/SemanticIndexJobHandlerTest.php                  |  61 ++++++++++++++++
 .hatfield/extensions/observational-memory/tests/SemanticIndexServiceTest.php                     | 459 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 .hatfield/extensions/observational-memory/tests/SemanticIndexStartupHookTest.php                 |  32 +++++++++
 .hatfield/extensions/observational-memory/tests/SemanticSettingsTest.php                         |  56 +++++++++++++++
 .hatfield/extensions/observational-memory/tests/Support/TransientFtsStatement.php                |  23 ++++++
 .hatfield/settings.yaml                                                                          |  16 +++++
 castor.php                                                                                       |   1 +
 composer.json                                                                                    |   6 +-
 composer.lock                                                                                    | 422 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++-
 depfile.yaml                                                                                     |   5 +-
 38 files changed, 3235 insertions(+), 19 deletions(-)
 create mode 100644 .castor/om-semantic.php
 create mode 100644 .hatfield/extensions/observational-memory/docs/om-relevance-calibration.md
 create mode 100644 .hatfield/extensions/observational-memory/docs/om-semantic-retrieval-benchmark.md
 create mode 100644 .hatfield/extensions/observational-memory/docs/relevance-calibration-questions.json
 create mode 100644 .hatfield/extensions/observational-memory/src/Semantic/MemoryChunker.php
 create mode 100644 .hatfield/extensions/observational-memory/src/Semantic/MemoryStoreAdapter.php
 create mode 100644 .hatfield/extensions/observational-memory/src/Semantic/SearchInterruptedException.php
 create mode 100644 .hatfield/extensions/observational-memory/src/Semantic/SemanticApiClient.php
 create mode 100644 .hatfield/extensions/observational-memory/src/Semantic/SemanticIndexJobHandler.php
 create mode 100644 .hatfield/extensions/observational-memory/src/Semantic/SemanticIndexService.php
 create mode 100644 .hatfield/extensions/observational-memory/src/Semantic/SemanticIndexStartupHook.php
 create mode 100644 .hatfield/extensions/observational-memory/src/Semantic/SemanticSearchException.php
 create mode 100644 .hatfield/extensions/observational-memory/src/Semantic/SemanticSettings.php
 create mode 100644 .hatfield/extensions/observational-memory/tests/MemoryChunkerTest.php
 create mode 100644 .hatfield/extensions/observational-memory/tests/MemoryStoreAdapterTest.php
 create mode 100644 .hatfield/extensions/observational-memory/tests/SemanticApiClientTest.php
 create mode 100644 .hatfield/extensions/observational-memory/tests/SemanticIndexJobHandlerTest.php
 create mode 100644 .hatfield/extensions/observational-memory/tests/SemanticIndexServiceTest.php
 create mode 100644 .hatfield/extensions/observational-memory/tests/SemanticIndexStartupHookTest.php
 create mode 100644 .hatfield/extensions/observational-memory/tests/SemanticSettingsTest.php
 create mode 100644 .hatfield/extensions/observational-memory/tests/Support/TransientFtsStatement.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-12-add-semantic-retrieval-over-om-observations-and-reflections.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #524 state MERGED, mergedAt 2026-09-23T19:52:00Z; Main git status clean before transition; generated SPC cache/log files ignored without deletion; Task worktree enabled settings backed up at main .hatfield/tmp/om-semantic-worktree-settings-20260923.yaml (SHA256 9cdc237d...)
- Summary: PR #524 merged on GitHub at 9887df867f8b451bd7cce5ad1f7c51ca10a2e834. Preserved user's enabled semantic settings before cleaning task worktree; user requested enable semantic settings in main. Ignored generated SPC artifacts via main commit3b708a2fb. Post-merge full validation and main settings migration remain.

## Task workflow update - 2026-09-23T20:00:01+00:00
- Validation: Post-merge castor check QA run qa-20260923-195830-25261-fb5ba76f: test/phpstan/dead-code FAIL missing Symfony AI StoreInterface; other 8 lanes PASS; castor test:llm-real PASS5/30; cache guard/process cleanup PASS
- Summary: DONE merge succeeded and worktree removed. Post-merge castor check failed because integration checkout vendor/ was stale after composer.lock merge: Symfony AI StoreInterface missing in tests/PHPStan/dead-code. Installing locked dependencies before revalidation; no product code failure diagnosed.
- Post-merge validation blocker: integration vendor stale after merge; composer.lock includes symfony/ai-store but vendor not installed; safe action composer install --no-interaction --prefer-dist then castor check. QA report var/reports/qa-20260923-195830-25261-fb5ba76f.

## Task workflow update - 2026-09-23T20:05:07+00:00
- Validation: castor check PASS on integration QA run qa-20260923-200314-34738-14fc5df3 (11 lanes, 5,037 unit/integration tests/22,428 assertions; controller replay13/218, TUI6/40, llm-real5/30, PHPStan0, dead-code0, docs, deptrac, LSP, CS, catalog; integrity/cache/leak guards passed); Initial check failed due stale dependencies after merge; composer install --no-interaction --prefer-dist synced locked packages. Second check failed due stale PHPStan result cache from the initial absent-class scan; phpstan clear-result-cache isolated/fixed; final check passed.; git status --porcelain=v1 clean; task worktree removed; generated SPC artifacts retained and ignored
- Summary: DONE and post-merge validation complete. Main project settings now enable hybrid OM with localhost embedding/reranker endpoints and min_score -4 (commit395af1eb4); generated SPC cache/log files ignored without deleting them (commit3b708a2fb). Integration checkout clean and task worktree removed. Existing main OM has 9,721 observations/109 reflections; semantic index not yet created, expected to backfill asynchronously at next session start.
