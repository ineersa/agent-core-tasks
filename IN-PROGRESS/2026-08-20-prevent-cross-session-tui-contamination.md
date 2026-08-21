# Prevent cross-session TUI event contamination

## Goal
A user opened Hatfield twice concurrently from the same project directory and observed content/events from one running session appearing in the other session's TUI.

Investigate and fix session isolation across the complete live event path, including controller/runtime processes, Messenger transport/consumers, event polling, transcript projection, and any project-scoped temporary or IPC resources. Each TUI instance must consume and render only events belonging to its own canonical session/run ID (`session_id === run_id`), even when multiple Hatfield instances run concurrently in the same checkout.

Treat this as a live concurrency/isolation bug. Identify the root cause rather than adding a display-only filter that leaves shared-session processing or persistence contaminated. Preserve resume/replay behavior and intentional per-session event history.

Reproduction:
1. Open the same project directory in two terminal sessions.
2. Start Hatfield independently in both.
3. Run distinct prompts/actions concurrently.
4. Observe whether output, tool activity, status, or transcript entries from one session appear in the other TUI.

Safety: do not kill, signal, or restart root-owned workers or processes tagged with `HATFIELD_SESSION_ID` while investigating.

## Acceptance criteria
- Two concurrently running Hatfield TUI instances in the same project directory display only events, status, tool activity, and transcript content for their respective session/run IDs.
- Events persisted under `.hatfield/sessions/<id>/events.jsonl` remain isolated to the matching canonical session directory, with no cross-session writes or projection.
- The root cause in the shared runtime/event-consumption path is fixed; a presentation-only suppression is not the sole safeguard.
- Resume and canonical replay continue to reconstruct only the selected session without dropping or duplicating legitimate events.
- A deterministic regression test exercises two concurrent sessions with distinct identifiable events and fails if either TUI receives or renders the other's content.
- Because this concerns a user-reported live concurrency issue, validate at the lowest layer that reproduces the real shared-process/transport behavior; fixture-only replay is not accepted as sole proof if it cannot reproduce the contamination.
- Required focused Castor tests and full `castor check` pass, with worker lifecycle/teardown verified and no leaked consumers or controller processes.

## Workflow metadata
Status: IN-PROGRESS
Branch: task/2026-08-20-prevent-cross-session-tui-contamination
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-20-prevent-cross-session-tui-contamination
Fork run: agent_55a5957114c18ebf
PR URL: https://github.com/ineersa/agent-core/pull/419
PR Status: open
Started: 2026-08-20T20:17:54+00:00
Completed:

## Work log
- Created: 2026-08-20T15:48:48+00:00

## Task workflow update - 2026-08-20T15:59:56+00:00
- Summary: Additional reproduction detail from the user: the leaked content was a fork result appearing in the wrong TUI session.
- Prioritize tracing fork/subagent completion and handoff routing. Verify artifact/result events retain the initiating parent session/run ID through child completion, runtime transport, polling, and TUI projection; check for project-scoped rather than session-scoped channels or consumers.

## Task workflow update - 2026-08-20T20:17:54+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-20-prevent-cross-session-tui-contamination.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-prevent-cross-session-tui-contamination.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-20-prevent-cross-session-tui-contamination.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-20-prevent-cross-session-tui-contamination.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-20-prevent-cross-session-tui-contamination.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-20-prevent-cross-session-tui-contamination.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-20-prevent-cross-session-tui-contamination/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-prevent-cross-session-tui-contamination.
- Summary: Claimed for implementation. Reproduction specifically involves a fork result leaking between two concurrent Hatfield TUI sessions in one checkout.

## Task workflow update - 2026-08-20T20:24:19+00:00
- Summary: Recon found normal parent event polling and session event stores are run-filtered, while fork/subagent completion has weaker ownership boundaries: child discovery scans all project sessions, deferred child lookup is global by child run ID, and completion payloads rely on queue context rather than explicit parent-session correlation. Implementation must reproduce before fixing and enforce parent ownership at the earliest proven faulty boundary, not merely hide foreign events in the TUI.
- Testing recon read and followed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`. Required proof: deterministic two-session regression with distinct fork-result markers plus a real replay-backed `TmuxHarness` E2E using two Hatfield instances in one isolated project; focused commands include `castor test`, `castor test:controller-replay` where applicable, and `castor test:tui --filter=<new test>`. Full `castor check` remains for task-to-pr per workflow.

## Task workflow update - 2026-08-20T21:38:57+00:00
- Summary: Focused lifecycle investigation found a concrete gap: `/new` switches to a draft TUI without shutting down the old `JsonlProcessAgentSessionClient` controller or its Messenger consumers. They remain alive until the first prompt starts the new session, at which point session-change detection stops/restarts them. `/resume` does restart the old controller/consumers before attaching the target session. The client's compact per-run event buffer is not cleared on either switch, although delivery remains keyed by run ID. This is a credible stale-runtime lifecycle defect but is not yet proven to explain the reported cross-session fork rendering.
- Relevant flow: `NewSessionCommandHandler` → `TuiSessionSwitchService::requestNewDraft()` only cancels best-effort and stops the TUI; draft `InteractiveMode::startOrResumeRun()` returns without client shutdown. `/resume` calls `JsonlProcessAgentSessionClient::attach()`, whose session-change path calls `stopProcess()` then spawns session-scoped consumers. Required next step is a failing `/new` + in-flight fork completion reproduction before implementation.

## Task workflow update - 2026-08-20T21:52:11+00:00
- Summary: User clarified desired `/new` lifecycle: stop the old session's controller and consumers immediately when switching to the sessionless draft. Do not launch a replacement controller until the first prompt creates the new session ID; then start fresh session-scoped consumers. Also ensure old runtime buffers/child observation state cannot survive the boundary.
- Finalized behavior: `/new` means detach/terminate old runtime now, not keep it underneath the draft. Validate with an in-flight fork completing around `/new`, proving no old progress/result appears in the draft or subsequently created session. `/resume` should retain its existing stop-then-attach semantics, with stale buffer isolation verified.

## Task workflow update - 2026-08-20T22:11:50+00:00
- Recorded fork run: agent_9259e485ed9d2b36
- Validation: PASS: castor test --filter='JsonlProcessAgentSessionClientShutdownTest|RuntimeEventPerRunCompactBufferTest::testClearDropsEveryRetainedTail'; PASS: castor test --filter='JsonlProcessAgentSessionClientShutdownTest|RuntimeEventPerRunCompactBufferTest|SessionSwitchServiceTest|NewSessionCommandHandlerTest|TuiResumeSessionSwitchE2eTest' (35 tests); PASS: castor test:tui --filter=TuiNewSessionRuntimeTeardownE2eTest (real replay-backed TmuxHarness, 15 assertions); PASS: castor deptrac (0 violations); PASS: focused castor phpstan on changed production paths; PASS: castor cs-check after focused cs-fix; PASS: castor clean:cleanup:workers:list reported no stale QA worker candidates
- Summary: Implemented `/new` runtime isolation at commit 43b38fdcf2f16c9ffe32bbc7188f74c2959684bb. Session switches now shut down the owned controller/consumers and clear process-client event/child/session state before composing a draft or attaching a resumed session. A draft launches no replacement controller; first submit starts a fresh session-scoped runtime.
- Fork read and followed root AGENTS.md, `.agents/skills/testing/SKILL.md`, `tests/AGENTS.md`, and ponytail. Worktree is clean and commit exists. Full `castor check` intentionally deferred to task-to-pr.

## Task workflow update - 2026-08-21T00:03:28+00:00
- Recorded fork run: agent_bf7f334d2eeed336
- Validation: Reviewer APPROVED after test hardening; PASS: castor test (4732 tests, 19032 assertions); PASS: castor deptrac (0 violations); PASS: castor phpstan (0 errors); PASS: castor cs-check (0 files); Initial full castor test:tui had one unrelated startup timeout in TuiSubagentChildHitlCancellationE2eTest; immediate focused rerun passed (2 tests, 8 assertions), no stale QA workers found; full rerun pending.
- Summary: Task-to-pr review iteration resolved Tmux race and replay fixture coupling in test-only commit 4907d5bf77a075a1207bb906d75f4d8ed833c999. Re-review APPROVED (agent_f8832a061125157e).

## Task workflow update - 2026-08-21T00:03:57+00:00
- Validation: PASS on rerun: castor test:tui (41 tests, 348 assertions); prior isolated startup timeout did not reproduce; PASS: focused TuiSubagentChildHitlCancellationE2eTest (2 tests, 8 assertions); No stale QA worker candidates after initial timeout
- Summary: Full replay-backed TUI lane passed on rerun. Deterministic controller replay lane is still running before CODE-REVIEW transition.

## Task workflow update - 2026-08-21T00:29:08+00:00
- Summary: Task-to-pr halted after repeated TUI gate failures. Investigation found no task-worktree QA processes still running; only the active primary Hatfield session/controller remains and must not be touched. The new Tmux test has a real teardown defect: it sends Ctrl-D, immediately scans only controller/consumer PIDs, then force-kills tmux without waiting for pane/app/background descendants to exit, and never removes its isolated project directory. This can leave teardown/file writers racing parallel TUI startup and cache cleanup. No further gate retry until fixed and stress-validated.

## Task workflow update - 2026-08-21T03:01:28+00:00
- Recorded fork run: agent_8adc6c526186f0e0
- Validation: PASS final reviewer: agent_5ff85c221770ae5c; PASS castor check qa-20260821-025831-95466-2b16a453; test: 4732 tests, 19030 assertions; test:controller-replay: 13 tests, 252 assertions; test:tui: 41 tests, 355 assertions; test:llm-real: 13 tests, 144 assertions; deptrac/phpstan/cs-check/docs/catalog all PASS; QA artifact integrity, exact-run process/tmux leak check, exact-run cache cleanup, llama-proxy cache guard all PASS
- Summary: Final HEAD 4a79d63917b12f27668b5d944d07e4d5aae7e705. Fixed natural-exit/leak-proof teardown, restored strict PHPUnit assertion counting, and applied user-approved temporary Castor cap increase 180→210. Final reviewer APPROVED.

## Task workflow update - 2026-08-21T03:11:21+00:00
- Recorded fork run: agent_4fc552598edf8677
- Validation: PASS castor check qa-20260821-030827-117258-1b5ba2c3; test:tui: 41 tests, 353 assertions in 152.1s; All 9 lanes PASS; artifact integrity, exact-run leak check, cache cleanup, llama-proxy cache guard PASS
- Summary: Final HEAD 4a438e83596cfe2e15a6061de71da1e628f0e11a. Shared TUI readiness waiter now uses the existing 20s parallel startup budget instead of an inconsistent 3s default, resolving the repeated blank startup failures under castor-check contention.

## Task workflow update - 2026-08-21T03:18:14+00:00
- Recorded fork run: agent_55a5957114c18ebf
- Validation: PASS focused testMainAttentionLiveViewChildHitlQuestionSurfaces; PASS cs-check; No stale QA workers
- Summary: Transition check exposed another hardcoded parallel-budget mismatch in the same HITL test after startup succeeded: 12s child projection wait versus existing 20s generic parallel gate budget. Aligned that method's marker waits to the shared budget at b80e6d78985ad0b8bc0d16cfff2caf41c917f31c.

## Task workflow update - 2026-08-21T03:21:10+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (155.3s).
- Pushed task/2026-08-20-prevent-cross-session-tui-contamination to origin.
- branch 'task/2026-08-20-prevent-cross-session-tui-contamination' set up to track 'origin/task/2026-08-20-prevent-cross-session-tui-contamination'.
- Created PR: https://github.com/ineersa/agent-core/pull/419
- Validation: Reviewer APPROVED: agent_5ff85c221770ae5c; PASS castor check qa-20260821-030827-117258-1b5ba2c3 before final test-only wait alignment; PASS focused final HITL method, cs-check, worker diagnostics
- Summary: Implementation complete at b80e6d78985ad0b8bc0d16cfff2caf41c917f31c. `/new` runtime isolation, natural-exit/leak regression proof, 210s Castor wall, and consistent existing parallel TUI wait budgets are in place.

## Task workflow update - 2026-08-21T13:30:06+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR 419 approved by user but conflicts with current origin/main. Returning to IN-PROGRESS to merge origin/main and resolve conflicts before updating the PR.
