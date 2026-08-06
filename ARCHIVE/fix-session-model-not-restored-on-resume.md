# Restore session-selected model on resume (#176)

## Goal
GitHub issue: https://github.com/ineersa/agent-core/issues/176 (no label)

## Problem
Session model is not restored after resume. On resume, the agent uses the global selected model instead of the model that was selected in the original session.

## Expected behavior
Resuming a session restores the model that was selected/active in that session, not the current global model.

## Scope
- Determine where the session-selected model is (or should be) persisted (state.json, hatfield_session metadata, events.jsonl) and where it is read on resume.
- Determine why resume falls back to global model instead of the persisted session model.
- Fix resume path to restore the session-selected model.
- Regression test: session with non-default model resumes with that model.

## Context (per AGENTS.md)
- `session_id === run_id`; session metadata lives in `hatfield_session` DB table; session dir `.hatfield/sessions/<id>/` with canonical `events.jsonl` and `state.json`; transcript projection rebuilt from events.jsonl on resume; metadata queried from DB (no metadata.yaml).

This is a placeholder task; scout analysis will be appended with confirmed root cause and fix approach before implementation.

## Acceptance criteria
- Confirmed where session-selected model is persisted and where it is read on resume
- Root cause identified for why resume falls back to global model
- Resume restores the model that was selected in the resumed session
- Regression test: session created with non-global model resumes with that model
- Focused Castor validation passing; TUI-visible if resume model display changes (may require TmuxHarness proof)

## Workflow metadata
Status: DONE
Branch: task/fix-session-model-not-restored-on-resume
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-session-model-not-restored-on-resume
Fork run: axfjt2fn6odr
PR URL: https://github.com/ineersa/agent-core/pull/189
PR Status: merged
Started: 2026-06-21T21:25:23.288Z
Completed: 2026-06-21T23:39:33.369Z

## Work log
- Created: 2026-06-20T00:11:43.093Z

## Task workflow update - 2026-06-20T00:17:56.218Z
- Summary: Scout analysis complete. Root cause CONFIRMED: session model persisted only on /model mid-session, NOT on session creation (--model or /new --model); resume read path works but finds nothing.
- SCOUT ROOT CAUSE ANALYSIS (GitHub #176):
- 
- ROOT CAUSE TYPE: (b) Persisted for /model changes but NOT persisted during session creation. Read path works but has no data to read.
- 
- WHERE GLOBAL MODEL IS STORED/READ:
- - config/services.yaml:6 app.default_model: '' (empty)
- - src/AgentCore/Infrastructure/SymfonyAi/ModelResolverRoutingSubscriber.php:38-83 intercepts ModelRoutingEvent -> SessionAwareModelResolver
- - ExecuteLlmStepWorker receives app.default_model via DI (never user-set)
- 
- RESUME PATH TRACE:
- InteractiveMode::run() [141-146] -> SessionInitializer::initialize($sessionId, null) [72-93] -> state->request = null (NO model)
- -> InteractiveMode::startOrResumeRun() [250-283] -> sessionStore->loadMetadata() -> client->resume($runId)
- -> InProcessAgentSessionClient::resume() [123] -> runner->continue($runId)
- -> AgentRunner::continue() -> ApplyCommand(Continue) -> AdvanceRun -> ExecuteLlmStep (no model field)
- -> ExecuteLlmStepWorker uses defaultModel ('') -> ModelRoutingEvent -> SessionAwareModelResolver -> ModelResolver::resolveInitialModel()
- 
- READ PATH WORKS (src/CodingAgent/Config/ModelResolver.php:60-86):
-   Tier 1 explicit model (null on resume)
-   Tier 2 session metadata: readSessionMetadata($sessionId)['model'] <- finds nothing because never written at start
-   Tier 3 catalog defaultModelReference() <- FALLS HERE
-   Tier 4 firstAvailableModel()
- 
- WHERE MODEL IS PERSISTED:
- - /model mid-session: CORRECT - src/CodingAgent/Config/ModelSettingsPersister.php:26-32 persistModel() writes model/model_provider/model_name to sessionMetaStore (DB hatfield_session).
- - --model CLI flag at startup: NOT PERSISTED - src/CodingAgent/Runtime/InProcess/InProcessAgentSessionClient.php:54-57 puts model into RunMetadata -> StartRunPayload -> run_started event only. No DB write.
- - /new --model draft promotion: NOT PERSISTED - src/Tui/Listener/SubmitListener.php:268-273 updateMetadata writes only run_id + prompt, omits model/reasoning despite $mergedRequest carrying them (256-263).
- 
- THE GAP (exact locations):
- PRIMARY: InProcessAgentSessionClient::start() at src/CodingAgent/Runtime/InProcess/InProcessAgentSessionClient.php:50-120 - creates RunMetadata with model but never persists model to session DB.
- SECONDARY: SubmitListener::dispatchToRuntime() at src/Tui/Listener/SubmitListener.php:268-273 (also 292-297, 414-419) - draft promotion updateMetadata omits model/reasoning.
- 
- DB SCHEMA (migration migrations/Version20260601152619.php):
-   model VARCHAR(255) DEFAULT NULL, model_provider VARCHAR(255) DEFAULT NULL, model_name VARCHAR(255) DEFAULT NULL, reasoning VARCHAR(255) DEFAULT NULL
- Entity src/CodingAgent/Entity/HatfieldSession.php has matching nullable columns.
- state.json: AgentCore RunState, no model field.
- events.jsonl: run_started has payload.payload.metadata.model (RunMetadata) but NOT read by SessionAwareModelResolver.
- 
- TUI/CLI MODEL PASSING: src/CodingAgent/CLI/AgentCommand.php:174-183 - resume: sessionId set, request null (no model).
- 
- RECOMMENDED FIX (Option A preferred):
- Persist in InProcessAgentSessionClient::start() after RunMetadata creation, before runner->start($input):
-   if (null !== $request->model || null !== $request->reasoning) {
-     $sessionId = '' !== $request->runId ? $request->runId : null;
-     if (null !== $sessionId) {
-       if (null !== $request->model) {
-         $ref = AiModelReference::tryParse($request->model);
-         if (null !== $ref) { $this->persister->persistModel($request->model, $ref->providerId, $ref->modelName, $sessionId); }
-       }
-       if (null !== $request->reasoning) { $this->persister->persistReasoning($request->reasoning, $sessionId); }
-     }
-   }
- Requires injecting ModelSettingsPersister into InProcessAgentSessionClient constructor.
- Option A handles both CLI --model and draft promotion through single path. Consider also JsonlProcessAgentSessionClient path.
- Option B (belt-and-suspenders): add model fields to SubmitListener updateMetadata calls (268-273, 292-297, 414-419).
- 
- REGRESSION TEST THESIS:
- A session created with a non-global model (via --model or /new --model) must resume with that same model after full process restart, not fall back to global default.
- Test: start() with model='llama_cpp/flash'; assert loadMetadata()['model']==='llama_cpp/flash' + provider/name; resolveInitialModel(null,$sessionId) returns that model.
- 
- EXISTING TESTS:
- - tests/CodingAgent/Config/SessionAwareModelResolverTest.php - tests resolution from DB but uses manually-written metadata, NOT persistence from start().
- - tests/AgentCore/Infrastructure/SymfonyAi/TraceReplayTest.php::testModelChangePersistsAcrossResume - tests /model mid-session path (uses ModelSelectionService::changeModel() which DOES persist) - does NOT cover initial-start persistence gap.
- GAP: no test verifies InProcessAgentSessionClient::start() persists model; no E2E test verifies --model survives resume.
- 
- START HERE: src/CodingAgent/Runtime/InProcess/InProcessAgentSessionClient.php:50-120 (inject ModelSettingsPersister, persist on start).

## Task workflow update - 2026-06-21T21:25:23.288Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-session-model-not-restored-on-resume.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-session-model-not-restored-on-resume.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-session-model-not-restored-on-resume.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-session-model-not-restored-on-resume.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-session-model-not-restored-on-resume.

## Task workflow update - 2026-06-21T21:31:56.771Z
- Summary: Root cause + fix approach CONFIRMED by orchestrator (builds on prior scout analysis):

ROOT CAUSE: InProcessAgentSessionClient::start() builds RunMetadata(model, reasoning) from StartRunRequest and passes it to runner->start() (→ run_started event only), but NEVER persists model/reasoning to the hatfield_session DB table. On resume, ModelResolver::resolveInitialModel() Tier-2 (session metadata) finds nothing and falls through to Tier-3 (global default). The read path is correct; the write path is missing at session start.

FIX POINT (single, covers ALL transports): InProcessAgentSessionClient::start() — confirmed that StartRunHandler (controller subprocess) delegates to InProcessAgentSessionClient::start(), so this one fix covers InProcess direct + JsonlProcess (controller subprocess) + CLI --model + /new --model draft promotion.

KEY CONSTRAINT: Must persist ONLY to session metadata (SessionMetadataStore::writeSessionMetadata). Must NOT use ModelSettingsPersister::persistModel()/persistReasoning() directly — those ALSO write the global home default via HomeSettingsWriter, which would make every `agent --model X` silently change the global default. Session start is session-scoped, not a default change.

APPROACH: Inject SessionMetadataStore into InProcessAgentSessionClient. In start(), when '' !== $request->runId (session row exists), write model (guard with AiModelReference::tryParse) + reasoning (guard with ModelResolver::LEVELS) to session metadata. updateMetadata() is a partial merge so the subsequent run_id/prompt write in SubmitListener is safe.

TESTS: (1) Contract test proving start() persists model/reasoning → ModelResolver::resolveInitialModel(null,$sessionId) returns it (fills gap left by TraceReplayTest which only tests read path + /model mid-session). (2) TUI E2E resume proof: start with --model=X, exit, resume WITHOUT --model, assert footer shows X.
- Orchestrator verified scout analysis against source: InProcessAgentSessionClient::start() (50-120), ModelSettingsPersister (persists home+session — wrong for start), ModelResolver::resolveInitialModel (Tier-2 read works), SessionMetadataStore::writeSessionMetadata (partial-merge primitive), HatfieldSessionStore::updateMetadata (partial merge confirmed), StartRunHandler (delegates to InProcessAgentSessionClient::start — single fix point for all transports), SessionInitializer (session row created before start() for CLI/prompt path; SubmitListener createSession before start for draft path).
- Confirmed DI: Ineersa\CodingAgent\ namespace autowired in services.yaml; SessionMetadataStore + ModelSettingsPersister auto-resolvable — no services.yaml change needed for new constructor param.
- Test templates identified: PromptTemplateExpansionInProcessTest (PerMethodIsolatedKernelTestCase + spy runner override pattern for InProcessAgentSessionClient), TraceReplayTest::testResumeUsesSessionMetadataOverGlobalDefaults (read-path assertion pattern), TuiResumeSessionSwitchE2eTest (TUI resume E2E + fixtures/tui-resume-minimal.json, uses --model + --resume).

## Task workflow update - 2026-06-21T21:42:37.636Z
- Recorded fork run: axfjt2fn6odr
- Validation: Fork: castor test --filter=StartRunPersistsSessionModelTest → OK (6 tests, 17 assertions); Fork: castor test:tui --filter=TuiResumeModelRestoreE2eTest → OK (1 test, 8 assertions); Fork: castor test:tui (full) → OK (14 tests, 123 assertions); Fork: castor phpstan → errors=0, file_errors=0; Fork: castor cs-check → files_fixed=0; Fork: castor deptrac → violations=0, errors=0; Orchestrator re-run: castor test --filter=StartRunPersistsSessionModelTest → OK (6 tests, 17 assertions, 6.8s); Orchestrator re-run: castor test:tui --filter=TuiResumeModelRestoreE2eTest → OK (1 test, 8 assertions, 5.0s); Orchestrator diff inspection: commit 104b597be — 3 files, +818 lines, production fix matches spec
- Summary: Implementation COMPLETE and verified by orchestrator. Commit 104b597be on branch task/fix-session-model-not-restored-on-resume (worktree /home/ineersa/projects/agent-core-worktrees/fix-session-model-not-restored-on-resume).

CHANGES (3 files, +818 lines):
1. src/CodingAgent/Runtime/InProcess/InProcessAgentSessionClient.php (+29) — injected SessionMetadataStore as required constructor param (before optional params); added persistence block in start() guarded by '' !== $sessionId, model via AiModelReference::tryParse(), reasoning via ModelResolver::LEVELS check, non-empty metaFields gate. Used SessionMetadataStore::writeSessionMetadata (NOT ModelSettingsPersister) to avoid mutating global home default. Non-obvious rationale preserved in comments.
2. tests/CodingAgent/Runtime/InProcess/StartRunPersistsSessionModelTest.php (new, 373 lines) — 6 contract/regression tests: persistence of model+reasoning, model-only, no-runId guard, no-model/reasoning guard, thesis test (resolveInitialModel returns session model over global default), fallback test. PerMethodIsolatedKernelTestCase + spy runner pattern.
3. tests/Tui/E2E/TuiResumeModelRestoreE2eTest.php (new, 416 lines) — REAL TmuxHarness E2E, #[Group('tui-e2e-replay')]: start TUI with --model=llama_cpp_test/test --prompt=hi, wait for replay response, Ctrl+D exit, resume SAME session with --resume=<id> and NO --model, assert footer shows 'test' (session-metadata restore, not flag restore). Shared HATFIELD_TEST_DATABASE_PATH across two launches.

ORCHESTRATOR VERIFICATION (re-ran independently):
- castor test --filter=StartRunPersistsSessionModelTest → OK (6 tests, 17 assertions) ✓
- castor test:tui --filter=TuiResumeModelRestoreE2eTest → OK (1 test, 8 assertions) ✓
- Fork reported castor phpstan (0 errors), cs-check (0 fixed), deptrac (0 violations), full castor test:tui (14 tests) all green.
- Production fix diff inspected: exactly as specified, guards + rationale comments intact.
- TUI E2E confirmed real (TmuxHarness::startDetached x2, --resume without --model, footer assertions) — not a mock/smoke script.

READY for task-to-pr. Not run per orchestrator scope: castor check, move_task(CODE-REVIEW), gh pr create, git push.
- Fork axfjt2fn6odr delivered: commit 104b597be, 3 files (+818 lines), all focused Castor validation green.
- Orchestrator independently re-ran the two critical tests (contract + TUI E2E) — both pass. Production fix and TUI proof verified genuine.
- Implementation scope complete. Awaiting task-to-pr to move to CODE-REVIEW.

## Task workflow update - 2026-06-21T23:01:35.378Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (81.1s).
- Pushed task/fix-session-model-not-restored-on-resume to origin.
- branch 'task/fix-session-model-not-restored-on-resume' set up to track 'origin/task/fix-session-model-not-restored-on-resume'.
- Created PR: https://github.com/ineersa/agent-core/pull/189

## Task workflow update - 2026-06-21T23:31:05.177Z
- Reviewer (subagent gu17sixwa8rm) found BLOCKER: TUI E2E Phase-3 assertions were non-discriminating — 'test' collided with transcript text 'Hello from the test harness.' (false positive), and 'deepseek-v4-pro' wasn't in catalog (meaningless negative). Test passed identically with/without the fix, failing AGENTS.md hard TUI gate.
- Review-iterate fork 07ozzjq4qn4c delivered commit 91a1303c8: renamed session model llama_cpp_test/test → llama_cpp_test/alpha (short name 'alpha', no transcript collision); footer-segment extraction (◆ to newline) with assertStringContainsString('alpha') + assertStringNotContainsString('default'); test 3 now asserts loadMetadata('') is empty; class docblock corrected re: setUpBeforeClass lifecycle.
- Orchestrator independently verified discrimination proof: neutralized persistence write (false && guard) → TUI E2E FAILED at line 185 (footer showed ◆ default, not alpha); restored → TUI E2E PASSED (9 assertions). Production file clean (git diff --stat src/ empty).
- Re-validated: TUI E2E OK (1 test, 9 assertions); combined contract+sibling filter OK (18 tests, 58 assertions, 0 risky). Commit 91a1303c8 pushed to origin, PR #189 updated.

## Task workflow update - 2026-06-21T23:39:33.369Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-session-model-not-restored-on-resume into integration checkout.
- Merge made by the 'ort' strategy.
 .../InProcess/InProcessAgentSessionClient.php      |  45 ++
 .../InProcess/StartRunPersistsSessionModelTest.php | 541 +++++++++++++++++++++
 tests/Tui/E2E/TuiResumeModelRestoreE2eTest.php     | 421 ++++++++++++++++
 tests/Tui/E2E/fixtures/tui-resume-minimal.json     |   2 +-
 4 files changed, 1008 insertions(+), 1 deletion(-)
 create mode 100644 tests/CodingAgent/Runtime/InProcess/StartRunPersistsSessionModelTest.php
 create mode 100644 tests/Tui/E2E/TuiResumeModelRestoreE2eTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-session-model-not-restored-on-resume.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-session-model-not-restored-on-resume.
- Pulled integration checkout: Merge made by the 'ort' strategy..
