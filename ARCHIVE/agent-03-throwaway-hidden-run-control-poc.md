# AGENT-03 Throwaway hidden-run and agent-control POC

## Goal
Run a deliberately disposable spike after AGENT-01/AGENT-02 to validate the hard architecture before production implementation. This is not an MVP and must not become a fallback/compatibility path.

Context:
- Depends on AGENT-01 and AGENT-02, unless the user explicitly chooses to spike with one hardcoded definition first.
- Reference plan: `.pi/plans/agents-subagents-implementation-plan.md`, especially Stage -1.
- Goal is to learn whether hidden child runs + parent registry + selected child event replay can work cleanly through the runtime/TUI boundary.
- Production implementation should be rewritten based on findings; do not preserve messy spike seams.

POC questions to answer:
- Can a parent run start/supervise a hidden child run without exposing it in normal session listing?
- Can a parent-scoped file registry track `agent_run_id`, status, artifact id, and attention/completion state?
- Can `/agents` or a temporary agent-control view select a child and rebuild from that child session's own `events.jsonl`?
- Can live selected-child updates be streamed/polled as normal runtime events with `runId = child_run_id` plus `parent_run_id` metadata, without mirroring every child event into the parent session?
- What TUI projection/state problems appear before production design is locked in?

Spike constraints:
- Hardcode one agent if needed, likely `scout`.
- Skip MCP policy, broad builtins, parallel execution, foreground `WaitingAgent`, and polished docs.
- Skip compatibility/fallback layers.
- Keep all code clearly disposable.
- Preferred final deliverable is a findings document/handoff; if spike code is not deleted before review, the task is not production-ready.

## Acceptance criteria
- A concise findings document or task handoff records what worked, what failed, and the recommended production structure changes.
- The spike demonstrates or falsifies hidden child run creation, parent registry tracking, selected child replay, and live selected-child update routing.
- The parent session does not duplicate full child events; child session events remain the detailed source of truth.
- Normal session listing exclusion for hidden child runs is evaluated and documented, even if implemented crudely in the spike.
- No compatibility/fallback layers are introduced.
- Do not move this task to CODE-REVIEW/DONE with retained TUI/runtime code unless a real TmuxHarness TUI E2E proof exists and passes `castor test:tui`; preferred completion is deleting spike code and committing only findings.
- Validation and observations use Castor commands only where tests are run; record blockers if tmux or llama.cpp:9052 prerequisites are unavailable.

## Workflow metadata
Status: DONE
Branch: task/agent-03-throwaway-hidden-run-control-poc
Worktree: /home/ineersa/projects/agent-core-worktrees/agent-03-throwaway-hidden-run-control-poc
Fork run: yazdsyieiz75
PR URL: https://github.com/ineersa/agent-core/pull/195
PR Status: merged
Started: 2026-06-22T02:21:22.961Z
Completed: 2026-06-22T20:27:25.336Z

## Work log
- Created: 2026-06-15T22:52:30.239Z

## Task workflow update - 2026-06-22T02:21:22.962Z
- Moved TODO → IN-PROGRESS.
- Created branch task/agent-03-throwaway-hidden-run-control-poc.
- Created worktree /home/ineersa/projects/agent-core-worktrees/agent-03-throwaway-hidden-run-control-poc.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/agent-03-throwaway-hidden-run-control-poc.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/agent-03-throwaway-hidden-run-control-poc.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/agent-03-throwaway-hidden-run-control-poc.
- Summary: Starting task per user request. Key design correction from user: child run storage should be evaluated as nested under the parent session rather than assuming top-level `.hatfield/sessions/<child_run_id>/events.jsonl`; fork instructions will make this the primary architecture question/proof for the POC.

## Task workflow update - 2026-06-22T02:22:10.867Z
- Recorded fork run: yzy33du01yci
- Summary: Launched implementation fork `yzy33du01yci` in worktree `/home/ineersa/projects/agent-core-worktrees/agent-03-throwaway-hidden-run-control-poc`. Fork instructions make the user's storage correction the primary POC question: evaluate nested child-run event storage under the parent session (for example `.hatfield/sessions/<parent_run_id>/agents/<child_run_id>/events.jsonl` or artifact-scoped equivalent) instead of assuming top-level `.hatfield/sessions/<child_run_id>/events.jsonl`. Preferred final deliverable is findings-doc-only after deleting spike code; retained TUI/runtime spike code requires real TmuxHarness E2E proof and `castor test:tui`.

## Task workflow update - 2026-06-22T02:25:57.050Z
- Recorded fork run: yzy33du01yci
- Validation: Fork validation: `castor cs-check` passed with 0 fixable issues / 0 files fixed.; Parent verification: `git status --short --branch` in worktree shows clean branch `task/agent-03-throwaway-hidden-run-control-poc`.; Parent verification: `git log --oneline -5` shows HEAD `b62d6e198 Add AGENT-03 hidden run control POC findings document`.; Parent verification: `git diff --stat main...HEAD` shows only `docs/agents-hidden-run-control-poc.md | 378 ++++++++++++++++++++++++++++++++++`.; Parent read-back verified the findings doc includes nested storage recommendation, existing path assumptions, parent registry design, session listing/hidden child recommendation, selected-child replay/live update recommendation, production task decomposition, and validation performed.
- Summary: Fork `yzy33du01yci` completed findings-only AGENT-03 POC deliverable. Commit `b62d6e198` created `docs/agents-hidden-run-control-poc.md` (378 insertions) and changed no production/test/config files. Parent verification confirmed branch status clean, commit present, diff stat exactly one new docs file. Core conclusion: user correction is validated — child runs should be stored nested under the parent session, recommended as `.hatfield/sessions/<parent_run_id>/artifacts/agents/<child_artifact_id>/{events.jsonl,state.json}` with parent registry at `.hatfield/sessions/<parent_run_id>/artifacts/agents/registry.json`. The doc rejects top-level `.hatfield/sessions/<child_run_id>/` because it pollutes session listing, complicates stale scan/cleanup, and would require DB hidden rows. Findings identify five flat-path assumptions localized to `SessionRunEventStore`, `SessionRunStore`, and `HatfieldSessionStore`, while `RuntimeEventEmitter`, `AgentSessionClient::events()`, `EventStoreInterface`, `TranscriptProjectorInterface`, and ChatScreen overlay APIs are already compatible with multi-run/child replay shape.

## Task workflow update - 2026-06-22T02:29:54.604Z
- Summary: User clarified that AGENT-03 should include an actual prototype in this project, not just findings documentation. Adjusting task scope: implement a disposable/test-local executable POC on the existing branch/worktree to prove nested child run event/state storage under parent session, parent-scoped registry tracking, selected-child replay, and live child update polling shape. Keep production surface minimal/clearly throwaway; if any retained TUI/runtime UI code is added, it must include TmuxHarness E2E proof. Prefer test-local POC and docs update so no product API is introduced solely for the spike.

## Task workflow update - 2026-06-22T02:31:18.805Z
- Summary: User asked how the prototype will be smoke tested. Acceptance clarified: executable POC should include an explicit manual smoke path, preferably a Castor task or clearly documented command, not only hidden test assertions. Smoke should create an isolated parent session, nested child events/state under `artifacts/agents/<child>`, registry.json, parent summary events only, run selected-child replay/polling, print the temp artifact path for inspection, and fail non-zero on assertion failure. If current fork returns test-only without a user-runnable smoke command/docs, relaunch for this gap.

## Task workflow update - 2026-06-22T02:31:45.946Z
- Summary: Correction after user feedback: previous follow-up fork was killed and the intended prototype is NOT just test-local/filesystem assertions. User expected a smoke-testable TUI overlay/control prototype to validate whether the agent-control view will work. Next implementation fork must build an explicitly disposable TUI-visible POC with a manual smoke path: a slash command/overlay or equivalent real TUI interaction that can be run in the project, creates or loads nested child-run fixture data under the active parent session, displays a selectable child list + selected child transcript/live update proof, and documents exact manual smoke steps. Because this touches TUI behavior, it must include a real TmuxHarness E2E proof and pass `castor test:tui` before handoff.

## Task workflow update - 2026-06-22T02:32:14.662Z
- Summary: User clarified no automated tests are needed for this POC; they will manually smoke test it. Implementation target is a disposable TUI-visible prototype, not test-local assertions. Because this intentionally skips mandatory TmuxHarness proof, keep task IN-PROGRESS/manual-smoke until user validates; do not move to CODE-REVIEW/DONE as production-complete without either user override or later automated TUI proof.

## Task workflow update - 2026-06-22T02:40:35.991Z
- Recorded fork run: rjqrkyarqxc2
- Validation: Fork validation reported: `php -l` on both new PHP files passed.; Fork validation reported: `castor phpstan --path=...` passed with 0 errors / 0 file_errors.; Fork validation reported: `castor deptrac` passed with 0 violations / 0 errors.; Fork validation reported: `castor cs-check` passed with 0 fixable issues.; Parent verification: `git status --short --branch` shows clean branch `task/agent-03-throwaway-hidden-run-control-poc`.; Parent verification: `git log --oneline -5` shows HEAD `5f4d06e29 Add AGENT-03 TUI smoke-testable POC prototype`.; Parent verification: `git diff --stat b62d6e198..HEAD` shows 4 files changed: `depfile.yaml`, `docs/agents-hidden-run-control-poc.md`, `src/Tui/Listener/AgentPocCommandHandler.php`, `src/Tui/Listener/AgentPocRegistrar.php`.; Parent read-back verified manual smoke docs include `castor run:agent-test`, `/agent-poc`, `/agent-poc tick`, `/agent-poc close`, expected visible `AGENT CONTROL POC`, and artifact inspection commands.
- Summary: Fork `rjqrkyarqxc2` completed the user-requested smoke-testable TUI POC. Commit `5f4d06e29` adds disposable `/agent-poc` slash command + TUI overlay and updates docs with manual smoke steps. Parent verification confirmed clean branch with HEAD `5f4d06e29`; diff from prior findings commit: 4 files changed (523 insertions): new `src/Tui/Listener/AgentPocCommandHandler.php`, new `src/Tui/Listener/AgentPocRegistrar.php`, modified `depfile.yaml`, modified `docs/agents-hidden-run-control-poc.md`. Prototype behavior: `/agent-poc` creates nested POC child data under the active parent session at `.hatfield/sessions/<parent>/artifacts/agents/scout-poc/{events.jsonl,state.json}` plus `registry.json`, then opens an `AGENT CONTROL POC` overlay showing parent session, paths, child status, event count, and transcript rebuilt from child events. `/agent-poc tick` appends a synthetic live child event and refreshes overlay; `/agent-poc close` removes it. No automated tests were added per explicit user request; this remains IN-PROGRESS/manual-smoke until the user validates.

## Task workflow update - 2026-06-22T02:57:37.027Z
- Summary: User manual-smoked screenshot and rejected current POC: overlay looks like a large text dump, steals focus so commands cannot be typed/closed, and misses the plan's split between compact agent status in the main TUI and a control overlay. Plan section 15 requires compact status (`Agents: scout#2 running...`) plus dedicated agent view/control. Next fix must: keep editor focus after `/agent-poc`, add/update compact `agents` status entry via `ChatScreen::setStatus()`, make overlay a small control panel/list+selected detail rather than full transcript dump, ensure `/agent-poc tick` and `/agent-poc close` are typable immediately after opening, and update manual smoke docs. No automated tests per user; user will manually smoke.

## Task workflow update - 2026-06-22T03:01:59.433Z
- Recorded fork run: qw75hm7bfvrm
- Validation: Fork validation reported: `php -l src/Tui/Listener/AgentPocCommandHandler.php` passed.; Fork validation reported: `php -l src/Tui/Listener/AgentPocRegistrar.php` passed.; Fork validation reported: `castor phpstan --path=src/Tui/Listener/AgentPocCommandHandler.php --path=src/Tui/Listener/AgentPocRegistrar.php` passed with 0 errors / 0 file_errors.; Fork validation reported: `castor deptrac` passed with 0 violations / 0 errors.; Fork validation reported: `castor cs-check` passed with 0 fixable issues after one auto-fix before final check.; Parent verification: `git status --short --branch` shows clean branch `task/agent-03-throwaway-hidden-run-control-poc`.; Parent verification: `git log --oneline -5` shows HEAD `73e88dd3c Fix AGENT-03 POC: editor focus, compact status, and smaller overlay`.; Parent verification: read-back confirms `insertOverlayAfterEditor()` is now used, there is no `setFocus()` call in the POC overlay, `ChatScreen::setStatus()` sets/clears `Agents`, and docs now require focus-preserving smoke steps.
- Summary: Fork `qw75hm7bfvrm` completed fix after user screenshot rejection. Commit `73e88dd3c` fixes the POC to match plan section 15: adds compact `Agents` status entry via `ChatScreen::setStatus('Agents', 'scout-poc running · N events · /agent-poc')`, changes overlay insertion to `insertOverlayAfterEditor()` and removes `setFocus()` so editor keeps focus, redesigns overlay from large transcript dump into compact agent-control panel/list + selected-child latest event + controls, and updates manual smoke docs. Parent verification confirmed clean branch at HEAD `73e88dd3c`; diff from prior POC commit: 3 files changed (98 insertions, 74 deletions): `AgentPocCommandHandler.php`, `AgentPocRegistrar.php`, and `docs/agents-hidden-run-control-poc.md`. No automated tests added per user request; manual smoke still required.

## Task workflow update - 2026-06-22T03:09:34.819Z
- Summary: User rejected second fix: `insertOverlayAfterEditor()` is not an overlay/control plane; it is a below-editor inline widget. User clarified plan intent: compact agent status belongs in main TUI (status/updatable/sticky area), while overlay should be a modal-like Symfony TUI control plane for full agent details, steering, cancel/retrieve/close controls. I re-read plan section 15: 15.1 compact status, 15.2 dedicated agent view with controls, 15.3 selected child detail from child event stream, 15.4 attention flow. Next fix must stop treating the agent details panel as a passive after-editor widget. Implement `/agent-poc` as a focused modal/control plane (picker/question-style overlay, likely `insertOverlayBeforeEditor()` with focused SelectListWidget/control list), closeable with Esc/Close control rather than needing slash command typing while open. Keep compact status separately in main TUI.

## Task workflow update - 2026-06-22T03:14:14.225Z
- Recorded fork run: yazdsyieiz75
- Validation: Verified git worktree clean on branch task/agent-03-throwaway-hidden-run-control-poc at HEAD b3198be2c.; Fork reported validation: php -l changed PHP files OK; castor phpstan scoped 0 errors; castor deptrac 0 violations; castor cs-check 0 fixable issues. No castor test:tui/check by explicit user request for POC manual smoke.
- Summary: Fork yazdsyieiz75 completed commit b3198be2c: corrected AGENT-03 POC to test the actual plan goal — compact agent status plus a modal/focused agent control overlay. `/agent-poc` now sets compact `Agents` status separately and opens a modal-like control plane via `insertOverlayBeforeEditor()` + focused `SelectListWidget` (QuestionController pattern). Controls: tick/update, steer, cancel child, retrieve artifact, close. Esc closes overlay via onCancel while status persists. Nested storage under `.hatfield/sessions/<parent>/artifacts/agents/<child>/` remains intact. Docs updated with manual smoke flow. No automated tests per user instruction; manual smoke pending user validation.

## Task workflow update - 2026-06-22T03:15:24.555Z
- Summary: User manual smoke screenshot after commit b3198be2c shows `/agent-poc` does NOT look like a real overlay: it renders as an inline block in the normal chat layout above the editor/status area. Conclusion/finding: despite focus and SelectListWidget controls working, `ChatScreen::insertOverlayBeforeEditor()` is an insertion-slot API, not a floating/modal overlay. The POC has now answered the plan question: current TUI overlay mechanism is insufficient for the planned agent control overlay visual/UX. Next architectural decision: implement real overlay stack/layering in TUI infrastructure, or build the agent control plane as a dedicated/full-screen view instead of relying on current insertOverlay* APIs.

## Task workflow update - 2026-06-22T03:24:30.101Z
- Summary: Design discussion after failed overlay POC: user concluded subagents likely need simplification too. Steering/input/HITL-style child questions are too hard to make understandable in a dock/detail-view UI because the parent user cannot easily know what the subagent is doing or why it asks. Emerging direction: simplify subagents toward non-interactive fire-and-report workers with visible status/progress and artifacts/results, not interactive child conversations requiring steering or user input.

## Task workflow update - 2026-06-22T03:37:14.783Z
- Summary: Updated `.pi/plans/agents-subagents-implementation-plan.md` directly per user request (no fork). Commit 6ab966781 rewrites the plan based on AGENT-03 POC and design discussion: current insertOverlay* APIs are insertion slots, not real overlays; production should use compact status plus agent dock/dedicated view; v1 subagents are simplified to non-interactive fire-and-report workers; remove steering, live child user input, child HITL questions, nested approval flows, and interactive child conversations from v1; QH/ask_human no longer a v1 prerequisite; parent-scoped nested child storage under `.hatfield/sessions/<parent>/artifacts/agents/<artifact>/` is the v1 direction; no bundled built-in agent definitions; async launch returns handles/artifacts immediately; foreground/WaitingAgent deferred.

## Task workflow update - 2026-06-22T18:14:27.843Z
- Summary: Launched/read Pi subagents reconnaissance for `/home/ineersa/claw/my-pi/packages/subagents`: Pi subagents render entirely inline in the chat transcript tool widget via normal tool update callbacks; no overlay, dock, dedicated view, status tool, or async retrieve flow for primary UX. Updated plan in commit c028d6d7b to reflect user decisions: drop dedicated agent dock/view for v1; keep only foreground `subagent` tool execution with live inline progress; keep file-backed registry and `agent_retrieve` for handoff/history/events retrieval after failures/completion; drop completion notifications, selected-child replay, status tool, separate cancel/control plane, and background/async launch from v1; use normal in-flight cancellation (Esc) for running subagents.

## Task workflow update - 2026-06-22T19:59:32.878Z
- Validation: fork validation: `castor phpstan` -> 0 errors; fork validation: `castor deptrac` -> 0 violations; fork validation: `castor cs-check` -> 0 fixable
- Summary: Cleanup fork completed: removed disposable AGENT-03 POC production code and POC-only deptrac allowance, leaving only plan updates and findings doc for PR. Commit 62b866df9 (`Remove AGENT-03 throwaway POC code`) deletes `src/Tui/Listener/AgentPocCommandHandler.php`, deletes `src/Tui/Listener/AgentPocRegistrar.php`, and removes the unnecessary `TuiWidget` dependency allowance from `depfile.yaml`. Remaining diff vs main is docs/plan only: `.pi/plans/agents-subagents-implementation-plan.md` and `docs/agents-hidden-run-control-poc.md`. User explicitly requested no reviewer for this cleanup/PR.

## Task workflow update - 2026-06-22T20:00:57.391Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (71.6s).
- Pushed task/agent-03-throwaway-hidden-run-control-poc to origin.
- branch 'task/agent-03-throwaway-hidden-run-control-poc' set up to track 'origin/task/agent-03-throwaway-hidden-run-control-poc'.
- Created PR: https://github.com/ineersa/agent-core/pull/195
- Validation: cleanup fork validation: `castor phpstan` -> 0 errors; cleanup fork validation: `castor deptrac` -> 0 violations; cleanup fork validation: `castor cs-check` -> 0 fixable
- Summary: AGENT-03 finalized as docs/plan-only POC deliverable. Disposable `/agent-poc` TUI code and POC-only deptrac change were removed in commit 62b866df9; remaining branch content is `.pi/plans/agents-subagents-implementation-plan.md` and `docs/agents-hidden-run-control-poc.md`. User requested skipping reviewer for this PR.

## Task workflow update - 2026-06-22T20:27:25.336Z
- Moved CODE-REVIEW → DONE.
- Merged task/agent-03-throwaway-hidden-run-control-poc into integration checkout.
- Merge made by the 'ort' strategy.
 .pi/plans/agents-subagents-implementation-plan.md | 841 +++++++++-------------
 docs/agents-hidden-run-control-poc.md             | 526 ++++++++++++++
 2 files changed, 875 insertions(+), 492 deletions(-)
 create mode 100644 docs/agents-hidden-run-control-poc.md
- Removed worktree /home/ineersa/projects/agent-core-worktrees/agent-03-throwaway-hidden-run-control-poc.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/agent-03-throwaway-hidden-run-control-poc.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #195 merged. AGENT-03 completed as docs/plan-only POC deliverable: captured parent-scoped hidden child run storage findings, proved current insertOverlay APIs are not real modal overlays, updated production plan to simplified foreground inline `subagent` tool + `agent_retrieve` direction, and removed all throwaway `/agent-poc` production code before merge.
