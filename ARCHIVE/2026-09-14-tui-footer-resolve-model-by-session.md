# Resolve TUI footer model by session; surface silent model-resolution fallbacks

## Goal
Evidence from session 48 (2026-09-14): footer showed `gpt-6-astra | 186.4k/1050.0k` while the session row, `run_started.metadata.model`, and all 90 LLM steps were `runpod/Qwen3.8-27B`. Runtime per-turn resolution (commit eb201a389, empty-model sentinel resolved at the provider boundary from session metadata) works — every active session (26/42/48/49/50) matched its DB row. The defects are in the TUI/selection layer:

1. Draft promotion never re-resolves footer state. `FooterStateListener::register()` seeds the footer once at InteractiveMode loop start; when `SubmitListener` promotes a lazy draft (sessionId '' -> real), the row is created and the run starts, but `FooterStateInitializer::initialize()` is not re-run. Footer keeps draft-seeded values (request model fallback -> in-process AppConfig default), so it can show a different model/context window than the session row and actual turns.

2. Silent tier-3 -> tier-4 fallback. `ModelResolver::resolveInitialModel()` drops to `firstAvailableModel()` when `ai.default_model` is unavailable, with no log or status. Session 48 ended up on runpod at creation with no trace of why.

3. `AgentCommand::runTui()` drops `--model`/`--reasoning` when `--resume` is used without `--prompt` (request only built when prompt is non-empty).

Key files: src/Tui/Listener/SubmitListener.php, src/Tui/Listener/FooterStateInitializer.php, src/Tui/Listener/FooterStateListener.php, src/CodingAgent/Config/ModelResolver.php, src/CodingAgent/CLI/AgentCommand.php.

## Acceptance criteria
- Footer model/reasoning/context window re-resolved from the hatfield_session row when a lazy draft is promoted (SubmitListener) and on session switch/resume, matching the runtime's resolution tiers; draft-seeded values no longer survive into a live session
- ModelResolver tier-4 fallback (default unavailable -> firstAvailableModel) emits a visible status/log entry naming the requested default and the model actually used
- agent --resume without --prompt no longer silently drops --model/--reasoning (forwarded or explicitly rejected)
- Regression tests at the lowest correct layer (FooterStateInitializer/ModelResolver unit + SubmitListener draft-promotion state test); castor check gate via move_task to CODE-REVIEW
- No new settings, APIs, or persistence fields

## Workflow metadata
Status: DONE
Branch: task/2026-09-14-tui-footer-resolve-model-by-session
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/498
PR Status: merged
Started: 2026-09-14T22:05:43+00:00
Completed: 2026-09-16T22:59:54+00:00

## Work log
- Created: 2026-09-14T21:34:34+00:00

## Task workflow update - 2026-09-14T22:05:43+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-14-tui-footer-resolve-model-by-session.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session.

## Task workflow update - 2026-09-14T22:06:26+00:00
- Ownership: owner=main; fork_run=none; revision=worktree task/2026-09-14-tui-footer-resolve-model-by-session (from main f731e9a1); scope=FooterStateInitializer re-resolve on draft promotion, ModelResolver tier-4 fallback warning, AgentCommand --model on --resume-without-prompt handling, with unit/virtual tests; outcome=assigned; commit=none

## Task workflow update - 2026-09-14T22:22:35+00:00
- Ownership: owner=main; fork_run=none; revision=task/2026-09-14-tui-footer-resolve-model-by-session; scope=Footer re-seed on draft promotion + shell-restart start (SubmitListener + FooterStateInitializer), ModelResolver tier-4 fallback warning, AgentCommand --resume+--model rejection and draft-carrier forwarding, InteractiveMode empty-prompt draft; outcome=completed; commit=3b4540991
- Validation: php -l clean; castor test --filter SubmitListener|SubagentLiveScenario|ModelResolverTest|AgentCommandModelOptionTest|FooterState*|ModelControlListenerTest|ModelCommandHandlerTest|ModelPickerControllerTest -> OK (128 tests, 442 assertions); castor test --filter StartRunPersistsSessionModelTest|TuiSessionCompositionTest|ModelSelectionServiceTest|SessionAwareModelResolverTest|CompactRunHandlerTest|SnapshotCompactionExtensionHookTest|ForkToolContractTest|TraceReplayTest -> OK (86 tests, 422 assertions); castor phpstan --path src -> 0 errors; castor deptrac -> violations=0; castor cs-check -> clean (after cs-fix on AgentCommand).

## Task workflow update - 2026-09-14T22:42:29+00:00
- Review: role=reviewer subagent; artifact=agent_bd9bc0e52dc2f0bd; target_revision=3b4540991; scope=full diff vs origin/main, specification-fidelity review per references/specification-fidelity.md; verdict=REQUEST CHANGES.
- Review finding (BUG/spec-fidelity): AgentCommand guard rejected --resume + --prompt + --model/--reasoning, a previously-working combination outside the finalized task scope; comment was factually wrong for the with-prompt path. All other areas verified sound: EM-mock draft test faithful to both real start() persist paths, no double-reseed/fingerprint conflict, switch/resume non-regression confirmed, tier-4 warning logging reviewed (context privacy OK, autowired logger reaches production), tests deterministic at lowest correct layer.
- Review fix: Ownership: owner=main; fork_run=none; revision=afd808f14 (on 3b4540991); scope=narrow AgentCommand guard to '' !== resume && '' === prompt && options set, factored into assertUsableModelOptions() with corrected comment and message, added resumeWithPromptAndModelStaysUsable test; outcome=completed; commit=afd808f14.
- Validation (review fix): castor test --filter AgentCommandModelOptionTest -> OK (6 tests, 15 assertions); castor phpstan --path src -> 0 errors; castor cs-check -> clean.
- Review round 2: role=reviewer subagent (resume agent_bd9bc0e52dc2f0bd); target_revision=afd808f14; scope=blocking finding delta; verdict=APPROVE WITH SUGGESTIONS (blocking finding resolved; delta regression check clean; remaining NTH was a stale test docblock). NTH addressed in 45463f5ee (comment-only; AgentCommandModelOptionTest 6 tests OK after change). Final submitted revision: 45463f5ee. No unresolved blockers.

## Task workflow update - 2026-09-14T22:44:52+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (130.8s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session/var/reports/qa-20260914-224242-5410-ebb7c951.
- Session/run: 50.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-14T22:44:54+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-14-tui-footer-resolve-model-by-session to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session/var/reports/qa-20260914-224242-5410-ebb7c951.
- Session/run: 50.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-14T22:44:57+00:00
- castor check passed (130.8s).
- Pushed task/2026-09-14-tui-footer-resolve-model-by-session to origin.
- Created PR: <url>
- Session/run: 50.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-14T22:44:57+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (130.8s).
- Pushed task/2026-09-14-tui-footer-resolve-model-by-session to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/498

## Task workflow update - 2026-09-15T23:55:09+00:00
- Moved CODE-REVIEW → IN-PROGRESS.

## Task workflow update - 2026-09-16T00:06:00+00:00
- PR review feedback round 2 (PR #498 inline comments): (1) ModelResolver logger nullable — remove nullable; (2) tier-4 log-only warning is still a silent swap for the TUI user; (3) InteractiveMode empty-prompt draft carrier — roll back. User direction: footer re-seed on submit masks the problem; understand why the divergence happened.
- Root cause found and proven (repro: two EntityManagers over one SQLite file): em->find() serves identity-map snapshots across processes. TUI creates the row (model NULL); worker persists the effective model in its own EM; the TUI's tier-2 read (getCurrentModel / Ctrl+P baseline) sees NULL and silently degrades to the global ai.default_model (polluted by any other session's changeModel); the worker's per-turn resolution keeps its cached model and never sees mid-run Ctrl+P writes. This defeats the eb201a389 per-turn session-row contract in both directions and produced footer=gpt-6-astra while session 48 served runpod/Qwen3.8-27B.
- Ownership: owner=main; fork_run=none; revision=82e28a32f (on 45463f5ee); scope=HatfieldSessionStore::fetchEntityOrNull refresh for cross-process freshness + two-EM regression tests; roll back InteractiveMode empty-prompt carrier and AgentCommand model-only forwarding (--model/--reasoning without --prompt now rejected up front); ModelResolver logger non-nullable with all ctor call sites updated; SubmitListener promotion re-seed retained — with fresh reads it re-seeds from the row the worker actually persisted; outcome=completed; commit=82e28a32f.
- Validation: castor test --filter HatfieldSessionStoreCrossProcessFreshnessTest|AgentCommandModelOptionTest|ModelResolverTest -> OK (58 tests); affected suites (SubmitListener|SubagentLiveScenario|FooterState|Model*|StartRunPersistsSessionModelTest|TuiSessionCompositionTest|SessionAwareModelResolverTest|CompactRunHandlerTest|SnapshotCompactionExtensionHookTest|ForkToolContractTest|TraceReplayTest|HatfieldSessionStore) -> OK (192 tests, 884 assertions); castor phpstan --path src -> 0 errors; castor cs-check -> clean; castor deptrac -> violations=0.

## Task workflow update - 2026-09-16T00:20:39+00:00
- Review round 3: role=reviewer subagent; artifact=agent_572adf41073407ae; target_revision=82e28a32f; scope=full diff vs origin/main, spec-fidelity + refresh-mechanics focus; verdict=REQUEST CHANGES. Verified sound: refresh() ordering safe in all mutating store methods, per-turn liveness acceptable (WAL + busy_timeout), two-EM test faithful, footer re-seed 'no longer a mask — re-seeds from truth', InteractiveMode byte-identical to origin/main.
- Round-3 findings fixed in 4d9a300d7: (CRITICAL) /reload crash — ReloadArgvBuilder preserved --model/--reasoning while dropping --prompt, producing the rejected prompt-less shape; now drops them (resumed session row owns selection) with all-forms test; (BUG) five tmux E2E prompt-less boots passed redundant --model — removed; (BUG) cross-process test mapping path ../../../../src -> ../../../src; (BUG) restored resumeWithPromptAndModelStaysUsable no-throw coverage; (EDGE/NTH) documented ghost-read caveat in fetchEntityOrNull docblock and kernel-base deviation in the two-EM test class docblock.
- Validation (4d9a300d7): castor test --filter HatfieldSessionStoreCrossProcessFreshnessTest|AgentCommandModelOptionTest|ReloadArgvBuilderTest -> OK (13 tests); SubmitListenerDispatchRuntimeTest|HatfieldSessionStoreTest -> OK (53 tests); castor phpstan --path src -> 0 errors; castor cs-check -> clean. Final revision: 4d9a300d7.

## Task workflow update - 2026-09-16T00:30:57+00:00
- Review round 4: role=reviewer subagent (resume agent_28609c6e3e9d1136); target_revision=890a7f8ef; scope=round-3 blocking finding delta; verdict=APPROVE. All six round-3 findings verified resolved; kernel-base deviation documented; naming nit applied; delta hygiene clean (+9/-3, two files). Reviewer note: full castor check gate (incl. test:tui for the five modified E2E boots) is owned by the CODE-REVIEW transition. Final submitted revision: 890a7f8ef. No unresolved blockers.

## Task workflow update - 2026-09-16T00:33:27+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-14-tui-footer-resolve-model-by-session.md.
- Session/run: 50.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session/var/reports/qa-20260916-003114-5190-61bb7770.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-16T00:35:16+00:00
- CODE-REVIEW transition attempt 1 failed on test:tui (2 errors), cause deterministic and mine: (1) while removing --model from TuiJourneyE2eTest I wrongly added --tools-excluded=bash — the journey never excluded bash, so the !ls shell-run lost the bash tool; (2) .hatfield/extensions/observational-memory/tests/Tui/TuiOmCommandsE2eTest.php still spawned a prompt-less --model=llama_cpp_test/test, which startup now rejects, killing the tmux pane before the logo. Repo-wide sweep confirms no prompt-less --model spawns remain in tests/ or extensions.
- Fix in 2e3113445: journey spawn restored to plain 'agent 2>&1' (no exclusions); OM extension boot drops --model. Focused re-validation: castor test:tui --filter TuiJourneyE2eTest|TuiOmCommandsE2eTest -> OK (3 tests, 27 assertions, 12.5s); cs-check clean. Final revision: 2e3113445.

## Task workflow update - 2026-09-16T00:36:23+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-14-tui-footer-resolve-model-by-session.md.
- Session/run: 50.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session/var/reports/qa-20260916-003526-10973-22cfb1dc.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-16T00:37:41+00:00
- CODE-REVIEW transition attempt 2 failed on test:llm-real: LlamaCppSmokeTest passed 2 args to the now 3-arg ModelResolver ctor (call site missed by the earlier pattern sweep). Fixed in 9cbc138cf; programmatic arity sweep confirms every ModelResolver construction in tests/ and extensions passes 3 args. Focused validation: castor test:llm-real --filter LlamaCppSmokeTest -> OK (1 test, 8 assertions). Final revision: 9cbc138cf.

## Task workflow update - 2026-09-16T00:38:50+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-14-tui-footer-resolve-model-by-session.md.
- Session/run: 50.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session/var/reports/qa-20260916-003753-15765-62a822c7.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-16T00:40:05+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (56.8s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session/var/reports/qa-20260916-003908-20661-e0c39bb0.
- Session/run: 50.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-16T00:40:06+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-14-tui-footer-resolve-model-by-session to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session/var/reports/qa-20260916-003908-20661-e0c39bb0.
- Session/run: 50.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-16T00:40:07+00:00
- castor check passed (56.8s).
- Pushed task/2026-09-14-tui-footer-resolve-model-by-session to origin.
- PR already exists: <url>
- Session/run: 50.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-16T00:40:07+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (56.8s).
- Pushed task/2026-09-14-tui-footer-resolve-model-by-session to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/498

## Task workflow update - 2026-09-16T20:10:04+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User approved implementing required refreshes from the query map and correcting PR #498. Preserve supported CLI behavior; unify draft/live selection; no universal EM clear.

## Task workflow update - 2026-09-16T20:10:35+00:00
- Accepted scope expansion: implement Refresh rows in architecture/orm-freshness-map.md; preserve existing concurrency controls; no blanket EM clearing. Fix draft model-selection convergence and remove unsupported prompt-less CLI policy changes. Merge main without rewriting history; rewritten map copied into task worktree.
- Ownership: owner=fork; fork_run=none; revision=a71b4d5fc; scope=session and background-process freshness plus container-based regressions; outcome=assigned; commit=none

## Task workflow update - 2026-09-16T20:19:10+00:00
- Validation: Slice 1 targeted session/process suites: 40 + 57 tests passed; scoped phpstan and style passed. Full gate pending.
- Ownership: owner=fork; fork_run=agent_fe74ec6931283e3c; revision=a71b4d5fc; scope=session and background-process freshness; outcome=completed; commit=0ad6c26fba4dc66bb611e2eb8d0dcd0f1f2252ad
- Ownership: owner=fork; fork_run=none; revision=0ad6c26fb; scope=deferred completion/batch/child freshness and container regressions; outcome=assigned; commit=none

## Task workflow update - 2026-09-16T20:27:01+00:00
- Validation: Slice 2 focused suites 42 tests/469 assertions passed, production phpstan/style passed.
- Ownership: owner=fork; fork_run=agent_71aac31423f51e4d; revision=0ad6c26fb; scope=deferred completion/batch/child freshness; outcome=completed; commit=1a4ae2c41c02dbd77165ae275774784c1d54ebb0
- Ownership: owner=fork; fork_run=none; revision=1a4ae2c41; scope=unified model selection, footer/runtime agreement, CLI restriction cleanup and regression proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-16T20:55:48+00:00
- Ownership: owner=fork; fork_run=agent_6cad61d990dc9a7b; revision=1a4ae2c41; scope=model selection and CLI cleanup; outcome=completed; commit=f29026b191
- Review: ORM reviewer agent_4d87bbb4f2d135d7 approved with suggestions at f29026b191. TUI reviewer agent_2ff267b33c16f036 requests removal of dead cycling APIs; also identified explicit unavailable-model display divergence and post-start reseed failure handling.
- Ownership: owner=fork; fork_run=agent_6cad61d990dc9a7b; revision=f29026b191; scope=review corrections in model selection and footer; outcome=assigned; commit=none

## Task workflow update - 2026-09-16T20:58:51+00:00
- Ownership: owner=fork; fork_run=agent_6cad61d990dc9a7b; revision=f29026b191; scope=review corrections; outcome=blocked; commit=none
- Prior fork exhausted context after uncommitted cycle API edits. Handing those files and unfinished review corrections to fresh fork.
- Ownership: owner=fork; fork_run=none; revision=f29026b191 plus dirty cycle edits; scope=finish review corrections and validation; outcome=assigned; commit=none

## Task workflow update - 2026-09-16T21:18:45+00:00
- Validation: Targeted slice suites passed; correction 110 tests/335 assertions and dead-code followup 113 tests/526 assertions passed.; Controller replay: 11 tests/175 assertions. Focused live LlamaCppSmokeTest: 1 test/8 assertions.; Production phpstan, deptrac, dead-code and cs-check passed.
- Ownership: owner=fork; fork_run=agent_506167a127006a8e; revision=f29026b191; scope=review corrections and integrated dead-code blockers; outcome=completed; commit=69248bfb38132a32f38c3652fce8ce58ee121f16
- Ownership: owner=main; fork_run=none; revision=69248bfb3; scope=query map recovery and implementation notes; outcome=completed; commit=6cb7dd4f9
- Review: role=reviewer; artifact=agent_4d87bbb4f2d135d7; revision=6cb7dd4f9; scope=ORM freshness and final delta specification fidelity; decision=APPROVE.
- Review: role=reviewer; artifact=agent_2ff267b33c16f036; revision=6cb7dd4f9; scope=TUI/model/CLI full diff and final delta specification fidelity; decision=APPROVE. Initial explicit-unavailable parity finding withdrawn after tracing actual provider resolution.
- No unresolved implementation blockers. Existing concurrent rebind versioning concern is outside refresh scope. Full gate pending.

## Task workflow update - 2026-09-16T21:20:18+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-14-tui-footer-resolve-model-by-session.md.
- Session/run: 55.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session/var/reports/qa-20260916-211908-11750-69cfaba9.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-16T21:20:27+00:00
- Full gate failed test lane: RenameSessionCommandHandlerTest generic repository stub incompatible with new exists COUNT path. No push occurred.
- Ownership: owner=fork; fork_run=agent_fe74ec6931283e3c; revision=6cb7dd4f9; scope=fix session existence mock callers exposed by gate; outcome=assigned; commit=none

## Task workflow update - 2026-09-16T21:24:58+00:00
- Validation: Focused 53 tests/215 assertions passed; style passed; no stale QA workers found.
- Ownership: owner=fork; fork_run=agent_fe74ec6931283e3c; revision=6cb7dd4f9; scope=session-existence caller test correction; outcome=completed; commit=f29fa76e1ca2cb285d0e0a8282540349a6babed5
- Review: role=reviewer; artifact=agent_2ff267b33c16f036; revision=f29fa76e1; scope=gate-failure correction delta; decision=APPROVE; no blockers.

## Task workflow update - 2026-09-16T21:26:13+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-14-tui-footer-resolve-model-by-session.md.
- Session/run: 55.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session/var/reports/qa-20260916-212516-16022-7222a874.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-16T21:26:23+00:00
- Gate2 phpstan found unused FooterStateInitializer::$sessionStore after resolver unification. Scoped phpstan did not detect this whole-program unused-property check.
- Ownership: owner=fork; fork_run=agent_506167a127006a8e; revision=f29fa76e1; scope=remove unused FooterStateInitializer constructor dependency and update call sites; outcome=assigned; commit=none

## Task workflow update - 2026-09-16T21:29:14+00:00
- Validation: Whole production castor phpstan passed; 70 focused tests/311 assertions passed; cs-check passed.
- Ownership: owner=fork; fork_run=agent_506167a127006a8e; revision=f29fa76e1; scope=unused footer dependency correction; outcome=completed; commit=53ec91b488dc96581af8231adef0e343a9248653
- Review: role=reviewer; artifact=agent_2ff267b33c16f036; revision=53ec91b48; scope=unused dependency correction; decision=APPROVE.

## Task workflow update - 2026-09-16T21:30:30+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (57.4s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session/var/reports/qa-20260916-212932-21436-d41e3e0a.
- Session/run: 55.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-16T21:30:32+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-14-tui-footer-resolve-model-by-session to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session/var/reports/qa-20260916-212932-21436-d41e3e0a.
- Session/run: 55.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-16T21:30:32+00:00
- castor check passed (57.4s).
- Pushed task/2026-09-14-tui-footer-resolve-model-by-session to origin.
- PR already exists: <url>
- Session/run: 55.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-16T21:30:32+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (57.4s).
- Pushed task/2026-09-14-tui-footer-resolve-model-by-session to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/498
- Summary: Removed unused footer injection revealed by full phpstan. Review approved 53ec91b48; whole-production static analysis and focused tests passed.

## Task workflow update - 2026-09-16T21:31:27+00:00
- Validation: castor check: PASS at 53ec91b488dc96581af8231adef0e343a9248653; independent review approved.
- Summary: Pushed final revision 53ec91b48, worktree clean. Full castor check passed in 57.4s. Updated existing PR #498 title/body via gh after verifying move_task left old description unchanged. PR remains open; not merged.

## Task workflow update - 2026-09-16T22:00:31+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User approved latest review corrections: remove resume model/reasoning rejection and post-start footer reseed workaround. Resume overrides must work without a prompt; reload must retain current selection. Latest clarification supersedes original reseed/rejection acceptance wording.

## Task workflow update - 2026-09-16T22:00:42+00:00
- Ownership: owner=fork; fork_run=agent_506167a127006a8e; revision=53ec91b48; scope=resume overrides and removal of post-start footer workaround with transition regressions; outcome=assigned; commit=none
- Routing: existing owner retains TUI/startup correction slice because it requires CLI-to-session and draft-promotion validation across prior regression harnesses; prior context saves reinvestigation. Entry points inspected: AgentCommand runTui/buildInitialRequest/guard, ReloadArgvBuilder, SubmitListener first-run/shell paths, FooterStateInitializer resolution precedence. Required proof: options applied before resume without synthetic prompt; draft selection/persisted model/runtime resolution/footer agree without reseed. No ORM policy changes.

## Task workflow update - 2026-09-16T22:13:06+00:00
- Previous fork resume rejected: artifact belongs to previous parent lifetime after user /resume.
- Ownership: owner=fork; fork_run=agent_54f35e13c97f2b37; revision=53ec91b48; scope=resume overrides and post-start footer removal; outcome=completed; commit=d2162f20993534c6883fad40345536a27a7c97d0
- Parent review: production removal correct; CLI regression currently invokes private helper via reflection rather than public startup path and hand-builds .hatfield dirs despite helper rule. Needs test correction before acceptance.

## Task workflow update - 2026-09-16T22:20:25+00:00
- Review: role=reviewer; artifact=agent_9fc8b74a6f47c05e; revision=d2162f209; scope=latest user-approved correction; decision=REQUEST CHANGES for test wiring proof and isolation helpers. Prior reviewer could not resume across parent lifetime.
- Ownership: owner=fork; fork_run=agent_54f35e13c97f2b37; revision=d2162f209; scope=review correction proof and avoid invalid-reasoning partial updates; outcome=assigned; commit=none

## Task workflow update - 2026-09-16T22:36:07+00:00
- Validation: 33 focused tests/139 assertions passed before final teardown-only correction; final AgentCommandModelOptionTest 6 tests/33 assertions passed.; Controller replay 11 tests/175 assertions passed; whole production phpstan, dead-code, cs-check passed.
- Ownership: owner=fork; fork_run=agent_54f35e13c97f2b37; revision=d2162f209; scope=public startup proof, validation-before-write, test isolation; outcome=completed; commit=8abb5527e26595eda137706212354b21f19daf01
- Review: role=reviewer; artifact=agent_9fc8b74a6f47c05e; revision=8abb5527e; scope=latest user-approved review corrections and specification fidelity; decision=APPROVE. No unresolved blockers.
- Removed reseed-only tests because that behavior is explicitly deleted. Retained draft selection-through-submit coverage; public AgentCommand startup test proves actual resume wiring and persisted overrides before InteractiveMode. Existing reload argv tests retain current-selection semantics.
- Reviewer directly read runtime settings file in an earlier review contrary to tool policy; parent identified violation, instructed no repeat, reviewer acknowledged. No settings modifications reported.

## Task workflow update - 2026-09-16T22:37:06+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-14-tui-footer-resolve-model-by-session.md.
- Session/run: 55.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session/var/reports/qa-20260916-223615-5799-772cce43.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-16T22:37:51+00:00
- Gate failed test SessionCacheInspectCommandTest::subscriberDisabledDoesNotCreateParentOrChildSidecars because parent launch now exports HATFIELD_WRITE_PROMPT_CACHE_DIAGNOSTICS=1. Verified inherited env=1. Unrelated deterministic test isolation gap exposed by new session diagnostics; preserve session flag.
- Ownership: owner=fork; fork_run=agent_54f35e13c97f2b37; revision=8abb5527e; scope=minimal disabled-diagnostics test isolation correction exposed by full gate; outcome=assigned; commit=none

## Task workflow update - 2026-09-16T22:42:31+00:00
- Validation: With HATFIELD_WRITE_PROMPT_CACHE_DIAGNOSTICS=1, disabled case passed1test/5assertions and full SessionCacheInspectCommandTest passed5tests/46assertions; cs-check passed.
- Ownership: owner=fork; fork_run=agent_54f35e13c97f2b37; revision=8abb5527e; scope=disabled-diagnostics test isolation; outcome=completed; commit=d1cf7b775576a9ba52701a607ee736d12d3bdc70
- Review: role=reviewer; artifact=agent_9fc8b74a6f47c05e; revision=d1cf7b775; scope=test-only diagnostics isolation delta plus prior approved corrections; decision=APPROVE. No unresolved blockers.

## Task workflow update - 2026-09-16T22:43:39+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (56.7s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session/var/reports/qa-20260916-224242-9128-23e37305.
- Session/run: 55.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-16T22:43:40+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-14-tui-footer-resolve-model-by-session to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session/var/reports/qa-20260916-224242-9128-23e37305.
- Session/run: 55.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-16T22:43:41+00:00
- castor check passed (56.7s).
- Pushed task/2026-09-14-tui-footer-resolve-model-by-session to origin.
- PR already exists: <url>
- Session/run: 55.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-16T22:43:41+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (56.7s).
- Pushed task/2026-09-14-tui-footer-resolve-model-by-session to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/498
- Summary: Corrected deterministic diagnostics test isolation exposed by user relaunch with diagnostics enabled. Final reviewer approved d1cf7b775. Resume override rejection and footer reseed removed; full gate retry after focused reproduction passes.

## Task workflow update - 2026-09-16T22:59:54+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session: ide_close_project returned isError.
- Merged task/2026-09-14-tui-footer-resolve-model-by-session into integration checkout.
- Merge made by the 'ort' strategy.
 architecture/README.md                                                                               |   2 +-
 architecture/orm-freshness-map.md                                                                    | 271 +++++++++++++++++++++++++++++++++++-----------------------------------------------
 config/services_test.yaml                                                                            |  18 ++++++
 src/CodingAgent/CLI/AgentCommand.php                                                                 |  62 +++++++++++++++++--
 src/CodingAgent/CLI/ReloadArgvBuilder.php                                                            |  23 ++++++-
 src/CodingAgent/Config/ModelResolver.php                                                             |  20 ++++++-
 src/CodingAgent/Config/ModelSelectionService.php                                                     |  30 ++++------
 src/CodingAgent/Entity/BackgroundProcessRepository.php                                               |  26 +++++++-
 src/CodingAgent/Entity/DeferredSubagentBatchRepository.php                                           |  50 ++++++++++++----
 src/CodingAgent/Entity/DeferredSubagentChildRepository.php                                           |  53 ++++++++++++----
 src/CodingAgent/Entity/DeferredToolCompletionRepository.php                                          |  35 ++++++++++-
 src/CodingAgent/Entity/HatfieldSessionRepository.php                                                 |  18 ++++++
 src/CodingAgent/Runtime/Contract/StartRunRequest.php                                                 |  18 ++++++
 src/CodingAgent/Session/HatfieldSessionStore.php                                                     |  50 +++++++++++++---
 src/CodingAgent/Tool/BackgroundProcess/ProcessStore.php                                              |  23 +++----
 src/Tui/Listener/FooterStateInitializer.php                                                          | 115 ++++++++++-------------------------
 src/Tui/Listener/ModelCommandHandler.php                                                             |   2 +-
 src/Tui/Listener/ModelControlListener.php                                                            |  37 ++++--------
 src/Tui/Listener/PendingModelSelection.php                                                           |  65 ++++++++++++++++++++
 src/Tui/Picker/ModelPickerController.php                                                             |   3 +-
 tests/AgentCore/Infrastructure/SymfonyAi/LlamaCppSmokeTest.php                                       |   2 +-
 tests/AgentCore/Infrastructure/SymfonyAi/TraceReplayTest.php                                         |   4 +-
 tests/CodingAgent/Agent/Execution/SessionAwareModelResolverTest.php                                  |   2 +-
 tests/CodingAgent/Agent/Execution/Subagent/Batch/Deferred/Launch/DeferredSubagentBatchLaunchTest.php |  10 ++--
 tests/CodingAgent/Agent/Tool/ForkToolContractTest.php                                                |   1 +
 tests/CodingAgent/Application/Pipeline/CompactRunHandlerTest.php                                     |   2 +-
 tests/CodingAgent/CLI/AgentCommandModelOptionTest.php                                                | 379 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/CLI/ReloadArgvBuilderTest.php                                                      |  32 ++++++++--
 tests/CodingAgent/CLI/Session/SessionCacheInspectCommandTest.php                                     |   2 +-
 tests/CodingAgent/Compaction/SnapshotCompactionExtensionHookTest.php                                 |   2 +-
 tests/CodingAgent/Config/ModelResolverTest.php                                                       | 101 ++++++++++++++++++++++++++++---
 tests/CodingAgent/Config/ModelSelectionServiceTest.php                                               |  64 ++++++++++++++------
 tests/CodingAgent/Entity/DeferredSubagentBatchRepositoryFreshnessTest.php                            | 202 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Entity/DeferredSubagentChildRepositoryFreshnessTest.php                            | 181 +++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Entity/DeferredToolCompletionRepositoryFreshnessTest.php                           | 155 +++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Extension/Agent/ConfiguredModelAgentRunnerMaxDurationTest.php                      |   3 +-
 tests/CodingAgent/Extension/Agent/ConfiguredModelAgentRunnerThinkingLevelTest.php                    |   2 +-
 tests/CodingAgent/Runtime/Controller/BackgroundProcessControllerSessionLifecycleListenerTest.php     |  10 ++--
 tests/CodingAgent/Runtime/Controller/E2E/Support/ControllerReplayBackgroundProcessSeeder.php         |   2 +-
 tests/CodingAgent/Runtime/InProcess/StartRunPersistsSessionModelTest.php                             |   2 +
 tests/CodingAgent/Session/HatfieldSessionStoreCrossProcessFreshnessTest.php                          | 184 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Tool/BackgroundProcess/ProcessStoreFreshnessTest.php                               | 165 ++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/CodingAgent/Tool/BackgroundProcessProvisionalCleanupTaskTest.php                               |  14 ++---
 tests/Tui/Application/TuiSessionCompositionTest.php                                                  |   2 +-
 tests/Tui/Listener/FooterStateListenerTest.php                                                       |  23 +++++--
 tests/Tui/Listener/ModelCommandHandlerTest.php                                                       |   4 +-
 tests/Tui/Listener/ModelControlListenerTest.php                                                      |  41 ++++++++++++-
 tests/Tui/Listener/RenameSessionCommandHandlerTest.php                                               | 168 +++++++++++++++++++++------------------------------
 tests/Tui/Listener/ResumeSessionCommandHandlerTest.php                                               | 101 +++++++++++--------------------
 tests/Tui/Listener/SubmitListenerDispatchRuntimeTest.php                                             | 134 +++++++++++++++++++++++++++++++++++++++--
 tests/Tui/Picker/ModelPickerControllerTest.php                                                       |   2 +-
 tests/Tui/Support/TuiSessionServicesFactoryTrait.php                                                 |   2 +-
 52 files changed, 2338 insertions(+), 581 deletions(-)
 create mode 100644 src/Tui/Listener/PendingModelSelection.php
 create mode 100644 tests/CodingAgent/CLI/AgentCommandModelOptionTest.php
 create mode 100644 tests/CodingAgent/Entity/DeferredSubagentBatchRepositoryFreshnessTest.php
 create mode 100644 tests/CodingAgent/Entity/DeferredSubagentChildRepositoryFreshnessTest.php
 create mode 100644 tests/CodingAgent/Entity/DeferredToolCompletionRepositoryFreshnessTest.php
 create mode 100644 tests/CodingAgent/Session/HatfieldSessionStoreCrossProcessFreshnessTest.php
 create mode 100644 tests/CodingAgent/Tool/BackgroundProcess/ProcessStoreFreshnessTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-14-tui-footer-resolve-model-by-session.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR498 merged at6b4b902c0fa08e451aae0f0ac9c69b42fc8a25e5. Main had prior assistant query-map edits already included in task; preserved in named stash before integration. Post-merge full validation follows.

## Task workflow update - 2026-09-16T23:01:24+00:00
- Updated PR Status: merged
- Validation: castor check PASS on integration9cc454fb6 (179.0s), report var/reports/qa-20260916-230002-14223-a77327d3.; 5072 unit/integration tests;11 controller-replay;9 TUI;5 live smoke. All static/docs/catalog lanes passed; no QA-owned process/tmux leaks; cache entries unchanged401.; git status --short empty; task worktree removal confirmed.
- Summary: DONE. Integrated main at9cc454fb6; post-merge full castor check passed. Git status clean; task worktree removed. IDE close reported degraded but filesystem cleanup succeeded. Earlier query-map draft safely retained in named stash; merged files include those edits and updated implementation notes.
