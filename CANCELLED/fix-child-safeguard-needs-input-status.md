# Fix child SafeGuard needs-input status in TUI

## Goal
SafeGuard approval questions work and can be answered for child runs, but the live child row/footer can remain `Running` while the question overlay is visible. SafeGuard intentionally uses transient `tool_question.requested` and does not transition AgentCore `RunStatus` to `WaitingHuman`. The TUI already marks the selected child optimistically through `SubagentLiveAttention`, but stale `subagent_progress: running` can overwrite it in the same/later tick. Fix the generic subagent live-view attention state for both fork and ordinary subagents. Do not redesign SafeGuard or convert tool questions into canonical ask_human lifecycle.

## Acceptance criteria
- A SafeGuard/tool question for a selected fork or ordinary subagent changes the child live-view status/footer to needs input.
- Stale Running progress cannot erase the needs-input state while the question remains pending.
- Answer, cancel, tool terminal, or child terminal clears/transitions the attention state correctly without leaving a stale latch.
- The fix is generic live-view/TUI behavior with no SafeGuard-specific or fork-specific status router and no AgentCore RunStatus change.
- Automated proof exercises the real TUI tick/question/catalog path at the lowest correct layer and would fail on the reported Running-while-question-visible bug.
- Testing skill and `tests/AGENTS.md` conventions are followed; focused Castor validation and full `castor check` pass before CODE-REVIEW.

## Workflow metadata
Status: CANCELLED (superseded by unify-tool-approvals-canonical-hitl)
Branch: task/fix-child-safeguard-needs-input-status
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-child-safeguard-needs-input-status
Fork run: nhhphdthixhl
PR URL: https://github.com/ineersa/agent-core/pull/301
PR Status: closed
Started: 2026-07-18T23:19:33.802Z
Completed:

## Work log
- Created: 2026-07-18T17:10:57.228Z

## Task workflow update - 2026-07-18T23:19:33.802Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-child-safeguard-needs-input-status.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-child-safeguard-needs-input-status.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-child-safeguard-needs-input-status.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-child-safeguard-needs-input-status.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-child-safeguard-needs-input-status.
- Parent IDEA module has no <content> element; cannot insert exclusions.
- Summary: Starting generic child live-view SafeGuard attention-state fix. Scope: prevent stale running progress from overwriting pending needs-input, clear the latch on answer/cancel/tool terminal/child terminal, no AgentCore RunStatus redesign. TUI deliverable must include a replay-backed real TmuxHarness proof per task-start request.

## Task workflow update - 2026-07-18T23:26:27.023Z
- Summary: Scout audit confirmed the generic race: selected-child tool_question.requested marks WaitingHuman through SubagentLiveAttention, then stale parent subagent_progress:running is ingested by SubagentLiveCatalog and overwrites it before TickPollListener refreshes the selected child/footer. Smallest fix is a run-id keyed needs-input latch owned by SubagentLiveCatalog: nonterminal progress cannot clear it; terminal progress always clears and wins; answer/cancel/matching tool terminal clear idempotently. No SafeGuard/fork-specific router and no AgentCore RunStatus change. Test thesis: a real replay-backed TmuxHarness child SafeGuard approval path must show needs input despite later stale running progress, then visibly clear after resolution; focused lower-layer tests cover terminal/cancel/tool-terminal latch clearing.
- Required testing context loaded before test planning: `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`.
- Worktree creation succeeded at `/home/ineersa/projects/agent-core-worktrees/fix-child-safeguard-needs-input-status`, but IDEA exclusion insertion reported that the parent module used a self-closing `<content />`; implementation fork will repair that external generated module block without committing it to the task branch.

## Task workflow update - 2026-07-18T23:27:12.056Z
- Recorded fork run: gnwvqw2pq02g
- Summary: Implementation fork launched in the task worktree with exact generic catalog-latch architecture, answer/cancel/tool-terminal/child-terminal lifecycle requirements, mandatory replay-backed real TmuxHarness proof for the stale-running-after-tool-question path, focused Castor-only validation, and no PR/review/push authority.

## Task workflow update - 2026-07-18T23:40:45.157Z
- Recorded fork run: gnwvqw2pq02g
- Summary: Initial implementation fork produced the generic catalog latch and focused tests but stopped before commit because the mandatory tmux test asserted immediately after the footer had already returned to `[running]`; the captured working line had not yet repainted and still contained the previous needs-input text. Worktree has 8 modified + 2 untracked task files and no commit. A narrow continuation is required to make the E2E wait for the complete post-answer render condition, reassess minimality, finish focused validation, and commit.

## Task workflow update - 2026-07-18T23:41:24.713Z
- Recorded fork run: gsqrpyy7yozz
- Summary: Continuation fork launched to fix the tmux post-answer synchronization predicate, reassess latch implementation minimality and duplicate clearing, run all required focused Castor validations, and commit. No full gate/review/push/PR authority.

## Task workflow update - 2026-07-18T23:44:58.825Z
- Recorded fork run: gsqrpyy7yozz
- Validation: `castor test --filter='SubagentLiveCatalogTest|SubagentLiveAttentionTest|TickPollListenerChildHitlTest'` — OK, 29 tests / 113 assertions.; `castor test:tui --filter=TuiSubagentChildSafeguardNeedsInputE2eTest` — OK, 1 test / 8 assertions; real replay-backed TmuxHarness path proves selected child approval remains needs-input across stale running progress and clears after answer.; `castor deptrac` — 0 violations.; `castor phpstan` — 0 errors.; `castor cs-check` — clean.
- Summary: Implementation complete at commit 00bba38007f3834a262f497c38c0abde66ae6dd5 (`fix(tui): latch child needs-input against stale running progress`). Verified clean worktree and expected 8-file diff: generic run-id needs-input latch in SubagentLiveCatalog, SubagentLiveAttention delegation, symmetric choice-question cancellation clear, focused catalog/attention/question tests, and new replay-backed TmuxHarness SafeGuard child proof with DB-seeded transient tool_question plus later stale parent running progress. No AgentCore/SafeGuard policy/fork-specific production changes. Testing skill and tests/AGENTS.md were read. Ready for separate task-to-pr phase; no review, push, PR, full castor check, or status move performed here.

## Task workflow update - 2026-07-19T00:25:06.445Z
- Summary: Reviewer verdict at HEAD 00bba3800: APPROVE WITH SUGGESTIONS. Mandatory real replay-backed TmuxHarness proof passed review, but actionable cleanup remains: make E2E race ordering deterministic (observe question/latch before appending stale running and prove that row was ingested), remove redundant parent-cancel clear, trim tautological duplicate attention test, avoid public latch API used only by tests, standardize new DTO construction, clarify terminal-cancel comment/fallback rationale, improve E2E failure artifact handling if supported, and document raw ToolQuestion DDL source. Launching a focused correction fork; re-review required until APPROVED.

## Task workflow update - 2026-07-19T00:25:30.398Z
- Recorded fork run: m3ccoi36chpu
- Summary: Focused review-correction fork launched for all actionable reviewer findings, especially deterministic post-latch stale-progress ingestion proof, production API/test simplification, lifecycle comment/clear cleanup, and failure diagnostics. No push/PR/full gate authority.

## Task workflow update - 2026-07-19T00:44:04.066Z
- Recorded fork run: m3ccoi36chpu
- Validation: `castor test` — first run exposed unrelated existing MessengerSqliteImmediateTransactionMiddleware timing failure (133ms vs 140ms threshold); focused `castor test --filter=MessengerSqliteImmediateTransactionMiddlewareTest` passed 4 tests / 19 assertions; full rerun passed 4485 tests / 15326 assertions.; `castor test:tui` — passed 38 tests / 201 assertions in 118.6s, including mandatory replay-backed TmuxHarness proof.; `castor deptrac` — 0 violations / 0 errors.; `castor phpstan` — 0 errors.; `castor cs-check` — clean.
- Summary: Task-to-PR review complete at HEAD a6e51ceaa17b5587bb28979b5377e16a63b3db43. Initial reviewer returned APPROVE WITH SUGGESTIONS; correction commit a6e51ceaa addressed all actionable findings (deterministic/accurate proof split, private latch query, redundant-clear/fallback cleanup, test simplification, comments/named args/failure snapshots/DDL source note). Re-review verdict: APPROVED with no remaining actionable findings. Real replay-backed TmuxHarness proof explicitly accepted: production ToolQuestionPoller DB seam drives selected-child approval overlay, waiting footer/working line, and clear-after-answer; deterministic stale-running latch invariant is covered at catalog layer because post-resume events.jsonl is not the live event bus. Worktree is clean. No live LLM validation required because no provider/model/tool-schema/prompt changes.
- Implementation commits: 00bba3800 (generic child needs-input latch and proof) + a6e51ceaa (all reviewer corrections). Reviewer re-review verdict: APPROVED. Focused task-to-PR Castor validation passed; branch remains unpushed pending CODE-REVIEW transition.

## Task workflow update - 2026-07-19T00:46:25.215Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (127.0s).
- Pushed task/fix-child-safeguard-needs-input-status to origin.
- branch 'task/fix-child-safeguard-needs-input-status' set up to track 'origin/task/fix-child-safeguard-needs-input-status'.
- Created PR: https://github.com/ineersa/agent-core/pull/301
- Validation: castor test: OK 4485 tests / 15326 assertions on rerun (after isolated existing timing test passed 4/19).; castor test:tui: OK 38 tests / 201 assertions.; castor deptrac: 0 violations.; castor phpstan: 0 errors.; castor cs-check: clean.; Reviewer re-review: APPROVED.
- Summary: Reviewer APPROVED current HEAD a6e51ceaa after all actionable suggestions were addressed. Focused Castor validation and full replay-backed TUI lane passed; mandatory real TmuxHarness proof is present. Moving to CODE-REVIEW for deterministic castor check, branch push, and PR creation.

## Task workflow update - 2026-07-19T02:48:18.159Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User live test rejected PR #301 behavior: session 1 still does not visibly show child needs-input, and navigating from child live view back to main migrates the active child approval question into the main view. Replay-backed proof is insufficient for the real path; trust live reproduction and return to implementation. Investigating exact session-1 event/question/view-routing state before any correction.

## Task workflow update - 2026-07-19T03:08:49.460Z
- Summary: Live session-1 forensics confirmed PR #301 tested the wrong ordering. Parent run 1 launched child 72cbc5dd-5a7d-54ae-b860-84835efd1c52 (artifact agent_a2f95c7c6a6b5a6a). SafeGuard created/emitted tool_question at 02:46:34 and it was only answered at 02:46:54 after user navigation. JsonlProcessAgentSessionClient::events(parent) partitions the child-scoped transient by run ID, so while main is active the TUI never invokes RuntimeQuestionEventHandler and never latches NeedsInput. Entering child drains the buffered event and opens the single global QuestionController overlay; SubagentLiveMainReturn changes transcript/view but leaves that overlay mounted, so it appears to migrate to main. Existing Tmux test enters child before seeding the question, exactly missing the real ordering. User explicitly requires the correction to be the simplest solution with minimal code and minimal tests.

## Task workflow update - 2026-07-19T03:09:54.811Z
- Recorded fork run: evg2jrvsb9ub
- Summary: Launched minimal live-correction fork. Hard constraints: no new types/interfaces/services/queues; consume/yield child ToolQuestionRequested once through existing run drain (process + in-process parity), pass state/screen to existing handler, scope existing overlay/submit/cancel by request run ID, close overlay immediately on main return without cancelling request, rewrite existing Tmux test to exact main-first live ordering, at most 1–2 focused lower-layer tests.

## Task workflow update - 2026-07-19T03:35:15.274Z
- Recorded fork run: evg2jrvsb9ub
- Validation: Focused unit tests: 62 tests/214 assertions OK; final narrow 3/14 OK; castor test:tui --filter=TuiSubagentChildSafeguardNeedsInputE2eTest: 1 test/6 assertions OK (~8s); castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean
- Summary: Implementation correction committed at 55791c645: real main-first child SafeGuard flow now routes ToolQuestionRequested consume-once through process/in-process drains; existing shared QuestionCoordinator/Controller remains the HITL UI; overlays/input are scoped by request run; return-to-main closes visual only; hidden terminal requests self-reject. Rewritten existing Tmux test exercises production ToolQuestionPoller DB seam in exact live ordering. Reviewer verdict BLOCKED on one narrow issue only: AgentsMainCommandHandler introduced unnecessary nullable `?QuestionController = null` despite sole production caller always supplying it. Core behavior/proof approved; no other blocking findings.

## Task workflow update - 2026-07-19T03:41:17.120Z
- Summary: Re-review of c657f839e: APPROVED, nullable DI blocker fully fixed with no remaining review findings. Orchestrator focused `castor test` then exposed one existing CancelListener test setup mismatch: testEscDuringFreeFormDoesNotCancelRun manually sets QuestionController.awaitingFreeForm but gives the listener a different empty QuestionCoordinator, so the newly-correct visible-question guard treats it as no visible request and does not restore. Production behavior is correct; test must use one shared coordinator containing a visible parent request. Suite stopped at 3495 tests/11853 assertions with that single failure; remaining validation not yet run.

## Task workflow update - 2026-07-19T03:45:27.598Z
- Recorded fork run: 1xq7f6pzx9dh
- Validation: castor test: OK (4488 tests, 15340 assertions, 22.8s); castor test:tui: OK (38 tests, 199 assertions, 113.5s); castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean; Reviewer: APPROVED at HEAD c657f839e; subsequent 5b2a877e6 is test-only setup correction discovered by full suite
- Summary: Review iteration complete at HEAD 5b2a877e6. Re-review verdict APPROVED with no remaining actionable findings. c657f839e made QuestionController required in AgentsMainCommandHandler; 5b2a877e6 corrected one stale CancelListener free-form test setup to share a visible parent QuestionRequest/coordinator. No production behavior changed after approved main fix 55791c645. Full focused validation is green.

## Task workflow update - 2026-07-19T03:47:44.424Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (125.6s).
- Pushed task/fix-child-safeguard-needs-input-status to origin.
- branch 'task/fix-child-safeguard-needs-input-status' set up to track 'origin/task/fix-child-safeguard-needs-input-status'.
- PR already exists: https://github.com/ineersa/agent-core/pull/301
- Validation: castor test: 4488 tests / 15340 assertions OK; castor test:tui: 38 tests / 199 assertions OK; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean
- Summary: Live-rejected PR #301 corrected minimally. Child SafeGuard tool_question is consumed once across run demux and reaches shared HITL UI while main is active; needs-input latches before entering child; overlay/input are scoped to owning run; Ctrl+\\ and /agents-main close visual only; re-enter restores same pending question; answer clears; hidden terminal child rejects orphan. Exact-order production ToolQuestionPoller/Tmux proof passes. Reviewer APPROVED and focused validation green.

## Task workflow update - 2026-07-19T03:52:11.154Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Live retest rejected PR #301 again. Cross-view overlay migration is fixed and /agents-live picker correctly shows `⚠ needs input`, proving tool_question delivery + catalog latch work. Remaining exact bug: main transcript's live child progress card still renders `● scout [running]` and footer `Working...` for run 7be4ffc0-4b57-5b77-b1c2-765518093a8c while picker simultaneously shows needs input. Trust live reproduction; return to implementation. Scope now only synchronize the existing main child progress presentation/footer from the already-correct catalog needs-input latch; no new lifecycle or event system.

## Task workflow update - 2026-07-19T03:53:58.642Z
- Summary: Second live retest narrows remaining defect precisely: run 7be4ffc0-4b57-5b77-b1c2-765518093a8c is correctly latched in SubagentLiveCatalog (picker shows `scout [⚠ needs input]`) and overlay migration is fixed, but main transcript card still renders its immutable projected `subagent_progress.status=running`, and Tick derives `Working...` only from parent activity. Existing SubagentLiveAttention comment says attention is shown on inline transcript cards, but it currently only clears status-panel keys. Minimal correction: reconcile existing parent transcript card progress status from the already-authoritative catalog before rendering, and make main working row prefer the first child needing attention. Rewrite no tests; extend the existing exact-order Tmux assertion before opening picker to require main card `⚠ scout [needs input]` and a child-needs-input working message.

## Task workflow update - 2026-07-19T04:05:50.106Z
- Recorded fork run: 7uxdtpg81f4i
- Validation: castor test:tui --filter=TuiSubagentChildSafeguardNeedsInputE2eTest: 1 test / 6 assertions OK; castor test --filter='SubagentLiveAttention|TickPollListener': 53 tests / 198 assertions OK; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean; Reviewer: APPROVE WITH SUGGESTIONS; behavior correct, simplify +113-line reconcile for explicit user minimality
- Summary: Main-presentation correction at 66d8c17fc is behaviorally correct and exact-order E2E passes, but strict review returned APPROVE WITH SUGGESTIONS because SubagentLiveAttention status overlay is +113 lines across 3 helpers and can be expressed in ~55–65 lines without semantic loss. Given user's repeated minimality requirement, will simplify before PR update. Also guard child attention working-line override so parent Cancelling remains authoritative. No behavior/test expansion.

## Task workflow update - 2026-07-19T22:59:55.942Z
- Recorded fork run: nhhphdthixhl
- Validation: Final reviewer: APPROVED at e7674ee5c; exact live-symptom Tmux proof, overlay ownership, terminal precedence, unchanged-screen fast path, and minimality accepted; Correction focused validation: TUI E2E 1/6 OK; focused unit 53/198 OK; phpstan 0; cs-check clean
- Summary: Minimal main-presentation correction finalized at e7674ee5c: catalog-backed needs-input overlay reconciles existing single/parallel transcript cards only on actual status changes; main working line reports child waiting while preserving parent Cancelling; exact main-before-picker Tmux proof retained. Final reviewer APPROVED with no actionable findings.

## Task workflow update - 2026-07-19T23:03:24.573Z
- Validation: castor test:tui: 38 tests / 199 assertions OK; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean; castor test: FAILED only MessengerSqliteImmediateTransactionMiddlewareTest::testBeginImmediateWaitsForCompetingWriterThenCompletesClaimTransaction under 16-worker ParaTest (timing threshold 140ms; observed 43ms then 133.9ms); castor test --filter=MessengerSqliteImmediateTransactionMiddlewareTest: 4 tests / 19 assertions OK; Final reviewer: APPROVED
- Summary: Task-to-PR final review complete at e7674ee5c. Focused TUI proof and full TUI replay lane are green. Default `castor test` at 16 ParaTest workers hit the same unrelated Messenger SQLite timing assertion twice (observed waits 43ms and 133.9ms vs 140ms threshold); the exact test passes isolated 4/19. This is recorded transparently; deterministic CODE-REVIEW gate uses a conservative 4-worker unit budget and will be authoritative.

## Task workflow update - 2026-07-19T23:05:40.425Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 600s)...
- castor check passed (123.5s).
- Pushed task/fix-child-safeguard-needs-input-status to origin.
- branch 'task/fix-child-safeguard-needs-input-status' set up to track 'origin/task/fix-child-safeguard-needs-input-status'.
- PR already exists: https://github.com/ineersa/agent-core/pull/301
- Validation: Final reviewer APPROVED; castor test:tui: 38/199 OK; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean; Focused Messenger SQLite middleware test: 4/19 OK; default 16-worker castor test had unrelated timing threshold flake recorded in task
- Summary: Updated PR #301 with the live-rejected-path correction: cross-run child question delivery while main is active, existing HITL overlay scoped to owning view, catalog latch protected from stale running, main transcript card and working line now show child needs-input, and answer clears it. Final reviewer APPROVED at e7674ee5c.

## Task workflow update - 2026-07-19T23:36:48.641Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User review rejected PR #301 on trust/maintainability despite working behavior: correctness is distributed across many conditionals and 26 files, with duplicated state requiring a 96-line transcript/catalog reconciliation. Return to IN-PROGRESS for read-only reduction audit before any further edits. Audit goal: classify every changed file as essential, accidental, or replaceable, identify one owner each for cross-run attention delivery, question view ownership, and displayed child status, then agree on a smaller design with the user.

## Task workflow update - 2026-07-19T23:49:33.081Z
- Summary: Read-only reduction audit complete (3 parallel scouts + focused renderer/controller feasibility pass). PR diff is 26 files, +1418/-60: 14 production files (+410/-52) and 12 test/support files (+1008/-8). Root complexity is architectural: run-scoped AgentSessionClient::events($runId) is special-cased into a session attention channel for ToolQuestionRequested; question/view ownership is split among coordinator, controller, handler, tick, submit, cancel, and navigation; displayed child status has two owners (catalog and projected transcript), requiring a 96-line reconciliation walker. Audit also found correctness contradictions: pre-row latch is documented/supported but orphan cleanup immediately rejects when catalog row is absent; markNeedsInputForRun can revive a terminal catalog row; queued/non-active terminal question cleanup is incomplete. No edits made.
- Reduction classifications: revert comment-only SubagentLivePickerController/SubagentLiveMainReturn changes and unrelated free-form ESC test expansion; replace 96-line transcript mutation with renderer-time catalog status resolution; deepen existing QuestionController as sole visibility/open-close/input owner; retain QuestionCoordinator as request lifecycle owner and SubagentLiveCatalog as sole effective status/latch owner; choose explicitly between current implicit cross-run ToolQuestion exception (smallest but low-integrity) and an AgentSessionClient session-attention channel (recommended, clearer contract).
- Realistic trustworthy target: 10–14 production files, +260–320/-30–50 LOC; 6–8 test/support files, +300–450/-50–100 LOC. Keep one process and one in-process consume-once contract test, two catalog contracts, one compact virtual ownership test, and one narrow main-first Tmux journey. Current added tests are ~1008 LOC with duplicated catalog/attention/listener assertions and a 387-line journey + 191-line custom fixture.

## Task workflow update - 2026-07-19T23:59:11.758Z
- Summary: Explored user's alternative: merge SafeGuard/ToolQuestion approvals into canonical ask_human-style HITL. Key finding: UI is already shared; runtime semantics are not. Literal existing `kind=interrupt` + `answer_human` would immediately give canonical child RunStatus::WaitingHuman, parent progress/card/picker status, durable human_input.requested, and eliminate PR #301's cross-run ToolQuestion delivery, catalog latch, and 96-line transcript reconciliation. But existing AgentCore interrupt finalizes the original tool call, appends the human answer to model context, and always starts a new LLM turn; it cannot resume the exact intercepted tool call. Preserving exact-call approval/no extra LLM turn requires a new generic suspended-tool approval continuation with durable original execution envelope, answer routing, hook callback/context persistence, one-shot bypass, batch/idempotency handling. Remaining TUI issue even with canonical HITL: session-global overlay/coordinator still needs centralized per-view ownership so a child question closes visually on main return and cannot capture parent submit/ESC.
- Literal ask_human migration risks: model may not retry or may change tool args; approval answer becomes model-visible/transcript-visible; SafeGuard Allow once/Always allow/Block callbacks and policy writes need durable routing; cancel vocabulary differs; deterministic identity/context and retry/crash semantics must be redesigned; parallel tool batches and result dedup prevent naïve same-call replay. Current ToolQuestion preserves exact original call/no extra LLM turn but blocks a tool worker indefinitely and has cancellation/restart weaknesses.
- If literal HITL semantics are accepted, most PR #301 code can be deleted: 3 transport special-cases, needs-input latch/reconcile, main status override, SafeGuard-specific fixture/E2E. Existing child WaitingHuman already maps to parent subagent_progress.waiting_human and existing card/picker. Keep only a much smaller question view-ownership fix around QuestionController/Tick/Submit/Cancel/navigation and reuse existing child HITL proof.

## Task workflow update - 2026-07-20T00:36:15.138Z
- Updated PR Status: closed
- Summary: CANCELLED/SUPERSEDED by TODO/unify-tool-approvals-canonical-hitl.md. PR #301 closed without merge at user direction. Do not merge or continue the latch/cross-run/transcript-reconciliation approach.
- 2026-07-19: User approved closing PR #301 and replacing the task with canonical HITL exact-tool-call continuation architecture. Current task workflow tool has no CANCELLED transition, so cancellation is recorded explicitly in metadata while cleanup/status reconciliation remains pending.

## Task workflow update - 2026-07-20T00:36:21.680Z
- Moved IN-PROGRESS → TODO.
- Summary: Cancelled and superseded by unify-tool-approvals-canonical-hitl; PR #301 is closed without merge. Returned out of active work because the current task workflow API has no CANCELLED status.
