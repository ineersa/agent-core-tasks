# AGENT-07 Parallel subagent execution and concurrency caps

## Goal
Production follow-up after AGENT-05.

Goal: extend the foreground `subagent` tool with parallel execution, following Pi subagents' single-tool shape. The parent LLM still waits for the foreground tool call to finish, while multiple child runs execute concurrently with inline aggregate progress and per-child artifacts.

Scope:
- Add mutually exclusive parallel mode (`tasks`) to `subagent` tool.
- Enforce per-parent and global concurrency caps from settings/config.
- Create one parent-scoped child run/artifact per task.
- Stream aggregate and per-child compact progress inline in the normal tool widget.
- Return aggregate final result with per-child success/failure summaries and artifact ids.
- Handle partial failures deterministically.

Out of scope:
- Background/async launch.
- `agent_status` or separate polling/status tool.
- Dock/view/overlay/control plane.
- Inter-agent conversation or child-to-child communication.

## Acceptance criteria
- `subagent` accepts either single mode or parallel `tasks` mode, never both; invalid combinations are rejected with actionable errors.
- Parallel execution creates distinct parent-scoped child artifacts and event/state paths for each task.
- Concurrency caps are enforced and documented; excess tasks queue or fail according to the selected design.
- Inline tool rendering shows aggregate status plus per-child progress/results without requiring a separate view.
- Aggregate final result clearly reports completed, failed, cancelled, and needs-clarification children with artifact ids.
- Partial failures do not lose successful child artifacts and do not corrupt registry state.
- Focused tests cover parameter validation, concurrency cap behavior, partial failures, cancellation where supported, and registry consistency.
- If user-visible TUI rendering is changed, add/update real TmuxHarness TUI E2E proof and run `castor test:tui`.
- Validation via Castor: focused tests, `castor phpstan`, `castor deptrac`, `castor cs-check`; `castor check` before PR.

## Workflow metadata
Status: DONE
Branch: task/agent-07-parallel-subagent-execution
Worktree: /home/ineersa/projects/agent-core-worktrees/agent-07-parallel-subagent-execution
Fork run: 4sj7l1ihteir
PR URL: https://github.com/ineersa/agent-core/pull/209
PR Status: merged
Started: 2026-06-24T17:43:29.382Z
Completed: 2026-06-25T02:10:56.032Z

## Work log
- Created: 2026-06-22T19:04:42.270Z

## Task workflow update - 2026-06-24T17:43:29.382Z
- Moved TODO → IN-PROGRESS.
- Created branch task/agent-07-parallel-subagent-execution.
- Created worktree /home/ineersa/projects/agent-core-worktrees/agent-07-parallel-subagent-execution.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/agent-07-parallel-subagent-execution.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/agent-07-parallel-subagent-execution.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/agent-07-parallel-subagent-execution.
- Validation: Pre-start git status checked: integration main clean and aligned with origin/main.
- Summary: Started AGENT-07 after task-explain discussion. User-approved design decisions: one settings-backed cap only (`agents.max_agents`, default 8) applied per `subagent` tool call; no user-supplied concurrency parameter; parallel mode runs all tasks up to cap; if LLM requests more than max, fail fast with clear message and split-work hint; propagate configured max into tool description/schema; cap is per parent/per subagent tool call only (no global cap). Partial failure should make overall tool call fail, but include per-agent artifact/status report and preserve successful artifacts. Tests must be fast, deterministic, and cache-stable: focused unit/service tests for scheduler/cap/failure/cancellation, plus minimal real llm-real E2E with llama-proxy cache stability verification (second run adds zero cache entries).

## Task workflow update - 2026-06-24T17:43:42.182Z
- Validation: Scout output artifact saved by subagent tool at /home/ineersa/.pi/agent/tmp/2026-06--edce1d45.txt (output-capped).; Task-explain phase followed user instruction to rely on scouts and avoid unbounded code reads.
- Summary: Planning/scout phase completed before implementation. Three scout subagents inspected existing subagent execution/tool/storage/testing architecture. Key findings: SubagentTool currently supports only single `{agent, task}` and explicitly rejects `tasks`/`concurrency`/`background`; SubagentExecutionService owns single-child lifecycle and should be refactored/extracted rather than expanded blindly; child artifact registry/stores already support distinct parent-scoped child artifacts with per-parent registry locks and per-child run locks; existing ToolExecutionUpdate → tool_execution.output_delta → transcript path can carry inline aggregate/per-child progress without new runtime event types; tests to update include SubagentToolTest and SubagentExecutionServiceTest, with live llm-real proof for LLM-visible `tasks` schema/cache stability.

## Task workflow update - 2026-06-24T17:44:30.185Z
- Recorded fork run: n6gm94jkhd9q
- Validation: Worktree created by move_task: /home/ineersa/projects/agent-core-worktrees/agent-07-parallel-subagent-execution.; Implementation fork n6gm94jkhd9q launched in background.
- Summary: Launched implementation fork n6gm94jkhd9q in worktree `/home/ineersa/projects/agent-core-worktrees/agent-07-parallel-subagent-execution` on branch `task/agent-07-parallel-subagent-execution`. Fork instructions include user-approved design decisions: single `agents.max_agents` cap default 8 per subagent tool call, no concurrency arg, fail-fast over cap, propagate max into tool description/schema, all tasks run concurrently up to cap, partial failure fails overall tool call while reporting all child artifacts/statuses. Fork instructed to keep tests fast/deterministic/cache-stable, add minimal llm-real E2E for parallel tasks, verify proxy second run adds zero cache entries, use Castor-only QA, commit locally, and not push/move task/run full castor check.

## Task workflow update - 2026-06-24T17:59:06.602Z
- Recorded fork run: n6gm94jkhd9q
- Validation: Parent spot-check: `git log` shows HEAD `be1f07e1e feat(agents): parallel subagent tasks with max_agents cap`.; Parent spot-check: `git diff --stat HEAD~1..HEAD` shows 12 files changed (+1150/-212), matching fork report.; Parent spot-check found untracked file `95` containing `run2 exit:0 stats:95-`; worktree not clean yet.
- Summary: Implementation fork n6gm94jkhd9q reported completion at commit `be1f07e1e` on branch `task/agent-07-parallel-subagent-execution`: parallel `subagent` tasks mode, single `agents.max_agents` cap default 8, no concurrency arg, fail-fast over cap, per-child artifacts/status report, partial-failure ToolCallException with artifact listing, focused unit/service tests, docs, and minimal live llm-real cache-stable proof. Parent spot-check confirmed commit and expected 12-file diff stat, but found an unexpected untracked file named `95` in the worktree despite fork handoff claiming clean. Cleanup follow-up needed before accepting implementation as ready.

## Task workflow update - 2026-06-24T18:02:43.246Z
- Recorded fork run: 19u4putvmjnu
- Validation: Cleanup fork verified `95` was untracked temp artifact and deleted it.; Parent verification: `git status -sb` in `/home/ineersa/projects/agent-core-worktrees/agent-07-parallel-subagent-execution` shows clean branch `task/agent-07-parallel-subagent-execution`; `git rev-parse --short HEAD` = `be1f07e1e`.
- Summary: Cleanup fork 19u4putvmjnu removed stray untracked validation artifact `95` from AGENT-07 worktree root. File contained `run2 exit:0 stats:95-` from proxy stats validation and was not part of implementation. No tracked code edits, no commit, no push, no QA. Parent verified worktree clean at HEAD `be1f07e1e` after cleanup.

## Task workflow update - 2026-06-24T18:21:21.014Z
- Validation: Reviewer verdict: APPROVE WITH SUGGESTIONS, no blockers.; Reviewer stated TUI E2E proof not required because no TUI rendering changed and existing ToolExecutionUpdate output path is reused.
- Summary: Reviewer subagent completed read-only review of AGENT-07 at HEAD `be1f07e1e` with verdict APPROVE WITH SUGGESTIONS (0 critical/blocking). Reviewer verified focused Castor checks: `castor test --filter='SubagentExecutionServiceTest|SubagentToolTest|AgentsConfigTest'` OK (28 tests/98 assertions), `castor cs-check` OK, `castor phpstan` OK, `castor deptrac` OK. Reviewer found sensible actionable suggestions: handle mid-parallel-launch `AgentRunner::start()` exceptions to avoid orphaned already-started children; make Cancelled/Cancelling status handling explicit in match; simplify `SubagentArgumentsFactory`/DTO double-validation/dead trim helpers; comment intentional cap-check duplication; document bounded parallel summaries in prompt guidance; avoid redundant per-poll child state reads in progress emission; consider reducing unchanged progress-event spam. Non-actionable/deferred: pre-existing non-atomic progress seq heuristic and config invalid-value logging consistency.

## Task workflow update - 2026-06-24T18:24:49.250Z
- Recorded fork run: 5tvh2tyhpn4e
- Validation: Fork validation: read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`; Castor-only QA.; Fork validation: `castor test --filter='SubagentExecutionServiceTest\|SubagentToolTest\|AgentsConfigTest'` OK (29 tests, 111 assertions).; Fork validation: `castor phpstan` OK (0 errors).; Fork validation: `castor deptrac` OK (0 violations).; Fork validation: `castor cs-check` OK after `castor cs-fix --path=src/CodingAgent/Agent/Execution/SubagentExecutionService.php`.; Parent verification: `git status --short --branch` clean on `task/agent-07-parallel-subagent-execution`; `git log -5` shows HEAD `6067c43c3`.
- Summary: Fix fork 5tvh2tyhpn4e addressed AGENT-07 reviewer suggestions and committed `6067c43c3` (`fix(agents): address parallel subagent reviewer suggestions`). Changes: mid-parallel-launch failure cleanup/finalization with aggregate ToolCallException report; explicit Cancelled/Cancelling match arms; simplified `SubagentArgumentsFactory` nested task normalization to trimmed arrays for Symfony denormalizer/validator; defense-in-depth cap-check comments; prompt guidance documenting bounded parallel summaries and `agent_retrieve`; progress update emission now uses already-fetched state and emits only on aggregate signature change. Parent verified worktree clean at HEAD `6067c43c3` with expected 12-file diff vs origin/main.

## Task workflow update - 2026-06-24T18:33:37.633Z
- Validation: Reviewer verdict at `6067c43c3`: APPROVE WITH SUGGESTIONS, no critical/blocking issues.; Reviewer verified prior fix categories resolved: launch failure cleanup, explicit Cancelled/Cancelling, DTO/factory simplification, cap comments, progress dedup, docs/tests.
- Summary: Re-review of AGENT-07 at HEAD `6067c43c3` returned APPROVE WITH SUGGESTIONS. Reviewer verified all prior findings resolved, focused Castor validation clean, and no TUI E2E required. Remaining actionable cleanup items: avoid phantom artifact directories in abortParallelLaunch for never-created registry entries, add defensive fallback for unexpected terminal/non-running registry statuses during abort, remove duplicate docblock before abortParallelLaunch, add structured log event for parallel launch abort cleanup, consider clearer never-started child failure summary and a 3-task test covering children after the failing launch. Launching another focused fork to address these before PR.

## Task workflow update - 2026-06-24T18:36:28.876Z
- Recorded fork run: gz6bolhp1ng9
- Validation: Fork validation: read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`; Castor-only QA.; Fork validation: `castor test --filter='SubagentExecutionServiceTest\|SubagentToolTest\|AgentsConfigTest'` OK (29 tests, 116 assertions).; Fork validation: `castor phpstan` OK (0 errors).; Fork validation: `castor deptrac` OK (0 violations).; Fork validation: `castor cs-check` OK after `castor cs-fix` on service file.; Parent verification: `git status --short --branch` clean; HEAD `ff4e6caa2`; diff vs origin/main remains 12 files.
- Summary: Second cleanup fork gz6bolhp1ng9 addressed remaining re-review suggestions and committed `ff4e6caa2` (`fix(agents): harden parallel launch abort cleanup`). Changes: `abortParallelLaunch()` no longer finalizes artifacts absent from registry (avoids phantom artifact dirs), syncs defensive terminal statuses into reports, removes duplicate docblock, logs structured `subagent_execution.parallel_launch_aborted` warning without raw prompts/tasks/exception message, gives never-launched children distinct report text, preserves launch exception as ToolCallException previous, and extends launch-failure test to 3 tasks covering a never-launched child. Parent verified worktree clean at HEAD `ff4e6caa2`.

## Task workflow update - 2026-06-24T18:44:12.306Z
- Validation: Final reviewer verdict: APPROVED at HEAD `ff4e6caa2`.; Reviewer reported validation: `castor phpstan` 0 errors, `castor cs-check` clean, `castor deptrac` 0 violations, `castor test --filter 'SubagentExecutionServiceTest|SubagentToolTest|AgentsConfigTest'` OK (29 tests, 116 assertions).; Reviewer confirmed TmuxHarness E2E is not required: no `src/Tui/` or TUI rendering/input/widget changes; existing ToolExecutionUpdate path reused.
- Summary: Final reviewer subagent reviewed AGENT-07 at HEAD `ff4e6caa2` and returned APPROVED. Reviewer confirmed abort cleanup hardening, phantom-artifact prevention, terminal-status preservation, privacy-safe structured log, 3-task launch-failure test, no schema/config/progress/cancellation regressions, and no TUI E2E requirement for this non-TUI change. Only remaining notes are non-actionable pre-existing robustness observations (cancel fire-and-forget; cancel throwing could interrupt abort cleanup) and a cosmetic `.hatfield/settings.yaml` example parity note.

## Task workflow update - 2026-06-24T18:44:58.504Z
- Validation: Parent validation: `castor test` OK — 3519 tests, 11123 assertions (19.1s).; Parent validation: `castor deptrac` OK — 0 violations, 0 errors.; Parent validation: `castor phpstan` OK — 0 errors, 0 file errors.; Parent validation: `castor cs-check` OK — files_fixed=0.; Parent validation: `castor test:llm-real --filter=SubagentParallelLiveE2eTest` OK — 1 test, 15 assertions (11.6s).; Parent verification: `git status --short --branch` clean on `task/agent-07-parallel-subagent-execution`; HEAD `ff4e6caa2`.
- Summary: Focused local validation completed on AGENT-07 worktree at HEAD `ff4e6caa2`; worktree clean and ready for CODE-REVIEW transition.

## Task workflow update - 2026-06-24T18:45:09.912Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (0.2s).
- Pushed task/agent-07-parallel-subagent-execution to origin.
- branch 'task/agent-07-parallel-subagent-execution' set up to track 'origin/task/agent-07-parallel-subagent-execution'.
- Created PR: https://github.com/ineersa/agent-core/pull/209

## Task workflow update - 2026-06-24T19:07:10.251Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Pulling AGENT-07 back from CODE-REVIEW to implement dedicated subagent/agent transport so blocking foreground `subagent` orchestration does not occupy generic tool workers and starve child tool calls.

## Task workflow update - 2026-06-24T19:50:03.885Z
- Recorded fork run: jgtl7d5ifs39
- Validation: Fork jgtl7d5ifs39: castor test --filter='SubagentExecuteToolCallRoutingMiddlewareTest\|SubagentExecutionServiceTest\|SubagentToolTest\|AgentsConfigTest' OK (34 tests, 124 assertions); Fork jgtl7d5ifs39: castor phpstan OK (0 errors); Fork jgtl7d5ifs39: castor deptrac OK (0 violations); Fork jgtl7d5ifs39: castor cs-check OK; Fork jgtl7d5ifs39: castor test:controller-replay OK (7 tests, 97 assertions)
- Summary: Dedicated agent transport implementation fork completed at commit eecd19a12 on branch task/agent-07-parallel-subagent-execution. Scope: new SubagentExecuteToolCallRoutingMiddleware routes ExecuteToolCall(toolName='subagent') to TransportNamesStamp(['agent']); messenger config/test config/.env add HATFIELD_AGENT_TRANSPORT_DSN and agent transport; HeadlessController launches one agent consumer; JsonlProcessAgentSessionClient and controller E2E bases set per-session agent queue DSN; services.yaml registers middleware; docs/agents.md documents transport starvation avoidance; tests/CodingAgent/Agent/Messenger/SubagentExecuteToolCallRoutingMiddlewareTest added. Worktree reported clean, not pushed.

## Task workflow update - 2026-06-24T20:04:19.004Z
- Recorded fork run: czxrxnvo17tz
- Validation: Parent targeted validation after eecd19a12: castor test --filter='SubagentExecuteToolCallRoutingMiddlewareTest|SubagentExecutionServiceTest|SubagentToolTest|AgentsConfigTest' OK (34 tests, 124 assertions); Parent targeted validation after eecd19a12: castor phpstan OK (0 errors); Parent targeted validation after eecd19a12: castor deptrac OK (0 violations); Parent targeted validation after eecd19a12: castor cs-check OK; Reviewer fork czxrxnvo17tz: castor test --filter='SubagentExecuteToolCallRoutingMiddlewareTest\|SubagentExecutionServiceTest\|SubagentToolTest\|AgentsConfigTest' OK (34 tests, 124 assertions); Reviewer fork czxrxnvo17tz: castor phpstan OK (0 errors), deptrac OK (0 violations), cs-check OK; Reviewer fork czxrxnvo17tz: castor test:controller-replay failed once in SafeGuardApprovalControllerReplayTest missing tool_execution.started, then passed on immediate retry (7 tests, 97 assertions); treated as flake, not blocker; Per user instruction, full castor check skipped because check is currently unreliable/fucked; using targeted validation only for this review iteration.
- Summary: Read-only review of dedicated agent transport commit eecd19a12 returned APPROVE WITH SUGGESTIONS with no blocking correctness or architecture issues. Reviewer verified middleware routing, messenger config, HeadlessController topology, per-session agent queue DSNs, starvation fix for tools.execution.max_parallelism=1, tests, and deptrac boundaries. Non-blocking suggestions: docs/settings.md lacks HATFIELD_AGENT_TRANSPORT_DSN but other transport DSNs also absent; docs/async-runtime-architecture.md should mention agent_<runId> queue; no automated assertion that messenger:consume agent is launched, consistent with existing MCP gap; one controller-replay SafeGuard flake failed once and passed on immediate retry.

## Task workflow update - 2026-06-24T20:05:45.570Z
- Summary: Attempted optional docs-only cleanup for reviewer DOC-2 (docs/async-runtime-architecture.md agent queue mention). Fork xx834t1g1hsm returned incomplete with no commit; fork 9qkz8u2odzrc failed with no output; fork w0gtg3pinzg9 failed with no output. Verified worktree still clean at eecd19a12 and branch ahead origin by 1. Since DOC-2 was non-blocking and user requested skipping full castor check, leaving optional docs drift unresolved for now rather than continuing to burn fork attempts.

## Task workflow update - 2026-06-24T20:17:15.705Z
- Recorded fork run: qtkqhdq5fgpl
- Validation: Fork qtkqhdq5fgpl: castor cs-check OK (files_fixed=0); No tests/check run for docs-only commit per user instruction to skip full castor check and fork scope.
- Summary: Docs-only cleanup fork completed reviewer DOC-2 at commit 59cdbf0d2 (docs(agents): document dedicated agent transport queue). Updated docs/async-runtime-architecture.md only: top messenger consumer diagram includes agent consumer/agent_{sessionId}; Key boundaries explain built-in subagent ExecuteToolCall routes to agent so parent orchestration does not occupy generic tool workers while child tools stay on tool and MCP stays on mcp; Doctrine SQLite Queue Layout includes agent_{sessionId} row; per-session scoping and .hatfield env block include HATFIELD_AGENT_TRANSPORT_DSN=doctrine://default?queue_name=agent_<runId>. docs/settings.md intentionally unchanged because transport env vars are not documented there. Worktree reported clean; branch ahead origin by 2 commits (eecd19a12 + 59cdbf0d2).

## Task workflow update - 2026-06-24T20:23:31.511Z
- Summary: Pushed branch task/agent-07-parallel-subagent-execution to origin at HEAD 59cdbf0d2, updating existing PR branch with dedicated agent transport commit eecd19a12 and docs cleanup commit 59cdbf0d2. Full castor check remains intentionally skipped per user instruction; task remains IN-PROGRESS pending decision on CODE-REVIEW transition/check gate.

## Task workflow update - 2026-06-25T01:26:42.650Z
- Recorded fork run: opamaz6jgq9r
- Summary: Launched merge-refresh fork opamaz6jgq9r to merge latest origin/main into task/agent-07-parallel-subagent-execution after testing infrastructure rework. Fork instructed to reread root AGENTS.md, .agents/skills/testing/SKILL.md, and tests/AGENTS.md before changes; merge origin/main normally; resolve conflicts/test updates minimally; run targeted Castor validation only (focused subagent tests, phpstan, deptrac, cs-check, controller-replay if feasible); do not run full castor check, push, or move task.

## Task workflow update - 2026-06-25T01:29:00.195Z
- Recorded fork run: opamaz6jgq9r
- Validation: Fork opamaz6jgq9r read root AGENTS.md, .agents/skills/testing/SKILL.md, and tests/AGENTS.md before changes.; Merged origin/main de8e9b03b into task branch; resulting HEAD 3efff98f1.; castor test --filter='SubagentExecuteToolCallRoutingMiddlewareTest\|SubagentExecutionServiceTest\|SubagentToolTest\|AgentsConfigTest' OK (34 tests, 124 assertions); castor phpstan OK (0 errors); castor deptrac OK (0 violations); castor cs-check OK (0 files to fix); castor test:controller-replay OK on first run (7 tests, 98 assertions); Full castor check not run per task/user instruction.
- Summary: Merge-refresh fork completed: merged latest origin/main de8e9b03b into task/agent-07-parallel-subagent-execution with normal merge commit 3efff98f1 (Merge origin/main into task/agent-07-parallel-subagent-execution). No manual conflicts or post-merge code/test edits were required; git auto-merged docs/settings.md and tests/CodingAgent/Runtime/Controller/E2E/ControllerE2eTestCase.php. AGENT-07 behavior preserved: parallel subagents + max_agents, dedicated agent transport for subagent ExecuteToolCall, async-runtime docs for agent_{runId}. Worktree clean; branch ahead of origin/task branch by 45 commits; not pushed.

## Task workflow update - 2026-06-25T01:34:59.697Z
- Recorded fork run: 6mp3q0ekr6ra
- Summary: Launched test compliance/performance fork 6mp3q0ekr6ra for SubagentParallelLiveE2eTest. Scope: reread AGENTS.md, testing skill, tests/AGENTS.md; time focused llm-real test and proxy cache stats if easy; enforce new <15s test rule if needed; fix isolation compliance by replacing ad-hoc .hatfield mkdir with shared TestDirectoryIsolation/helper pattern; run focused validation only; commit locally; no push/task move/full castor check.

## Task workflow update - 2026-06-25T01:36:41.284Z
- Recorded fork run: 6mp3q0ekr6ra
- Validation: Fork 6mp3q0ekr6ra read root AGENTS.md, .agents/skills/testing/SKILL.md, and tests/AGENTS.md before changes.; Proxy cache stats stayed at entries=113 before/after timed runs.; castor test:llm-real --filter=SubagentParallelLiveE2eTest OK x3 warm; 15 assertions; PHPUnit ~5.2–6.3s, Castor wall ~7.0–8.1s; castor test --filter='SubagentExecutionServiceTest\|SubagentToolTest\|AgentsConfigTest\|SubagentExecuteToolCallRoutingMiddlewareTest' OK (34 tests, 124 assertions); castor phpstan OK (0 errors); castor deptrac OK (0 violations); castor cs-check OK (0 files to fix); Full castor check not run per instruction.
- Summary: Test compliance/performance fork completed at commit d105f4327. SubagentParallelLiveE2eTest meets new <15s warm-test rule: three focused warm runs were PHPUnit ~5.2–6.3s and Castor wall ~7.0–8.1s; proxy cache entries stayed stable at 113 with no growth. Test remains in #[Group('llm-real')] because it is fast and still provides LLM-visible tasks schema proof. Isolation compliance fixed by replacing ad-hoc mkdir for .hatfield/agents with TestDirectoryIsolation::ensureDirectory() under the ControllerE2eTestCase-created isolated .hatfield tree; timeouts tightened from liveLlmToolWaitTimeout 90s→25s and HATFIELD_TEST_LLM_HTTP_TIMEOUT 120→60. Worktree clean per fork; not pushed.

## Task workflow update - 2026-06-25T01:38:06.466Z
- Summary: Pushed task/agent-07-parallel-subagent-execution to origin at HEAD d105f4327. Push range 59cdbf0d2..d105f4327 includes merge commit 3efff98f1 (origin/main de8e9b03b merged) and test compliance/performance commit d105f4327 for SubagentParallelLiveE2eTest. Branch was clean before push; task remains IN-PROGRESS pending CODE-REVIEW/check gate decision.

## Task workflow update - 2026-06-25T01:44:46.443Z
- Recorded fork run: 97w7waa66qax
- Summary: Launched final read-only reviewer fork 97w7waa66qax for AGENT-07 at current HEAD d105f4327. Scope: reread AGENTS.md/testing skill/tests/AGENTS.md; review parallel subagent execution, dedicated agent transport, merge freshness, docs, and SubagentParallelLiveE2eTest compliance/performance; run targeted validation only (focused unit tests, focused llm-real, phpstan, deptrac, cs-check, optional controller-replay); no edits/push/task move/full castor check.

## Task workflow update - 2026-06-25T01:46:21.160Z
- Recorded fork run: 97w7waa66qax
- Validation: Reviewer 97w7waa66qax read root AGENTS.md, .agents/skills/testing/SKILL.md, and tests/AGENTS.md before review.; castor test --filter='SubagentExecutionServiceTest\|SubagentToolTest\|AgentsConfigTest\|SubagentExecuteToolCallRoutingMiddlewareTest' OK (34 tests, 124 assertions; wall ~4.58s); castor test:llm-real --filter=SubagentParallelLiveE2eTest OK (1 test, 15 assertions; PHPUnit 6.193s; wall ~8.03s); Proxy cache entries stayed at 113 before/after focused llm-real review run (cache-stable, no growth).; castor phpstan OK (0 errors); castor deptrac OK (0 violations); castor cs-check OK (0 files to fix); castor test:controller-replay OK (7 tests, 97 assertions; wall ~39.08s); Full castor check not run per instruction. Branch clean and pushed at d105f4327 per reviewer.
- Summary: Final read-only reviewer pass at HEAD d105f4327 returned APPROVE WITH SUGGESTIONS with no blocking issues. Reviewer confirmed parallel semantics (max_agents fail-fast, concurrent launch, overall failure on child failure with artifact report, launch-abort cleanup without phantom artifacts), dedicated agent transport wiring (middleware, order before MCP, one agent consumer, per-session HATFIELD_AGENT_TRANSPORT_DSN in production and E2E), merge freshness after origin/main, docs coherence, and test compliance. Non-blocking suggestions: duplicate private findCompletedPayloadForCallId helper, pre-existing SubagentRetrieveLiveE2eTest ad-hoc mkdir, pre-existing progress seq CAS limitation, single agent consumer serializes concurrent parent subagent calls by design, invalid max_agents silently falls back, bounded parallel success summaries differ from single mode, cold proxy cache may exceed 15s, cosmetic .hatfield/settings.yaml drift, no automated assertion for messenger:consume agent process.

## Task workflow update - 2026-06-25T01:51:54.683Z
- Summary: Attempted final reviewer subagent after user clarified 'subagent, not fork'. Subagent artifact at /home/ineersa/.pi/agent/sessions/--home-ineersa-projects-agent-core--/subagent-artifacts/29803030_reviewer_output.md only contains startup line ('I've read all three mandatory docs...') and no review/validation output, indicating the reviewer subagent died or failed silently before producing a usable verdict. Existing final read-only fork review 97w7waa66qax remains the latest complete review evidence at HEAD d105f4327.

## Task workflow update - 2026-06-25T01:53:26.055Z
- Recorded fork run: ue75dlrsoazf
- Summary: Launched final cleanup fork ue75dlrsoazf before CODE-REVIEW. Scope: fix SubagentRetrieveLiveE2eTest isolation to use TestDirectoryIsolation::ensureDirectory() instead of ad-hoc .hatfield/agents mkdir; optionally add agents.max_agents example to .hatfield/settings.yaml if naturally missing; run targeted Castor validation only; commit locally; no push/task move/full castor check.

## Task workflow update - 2026-06-25T01:54:30.738Z
- Recorded fork run: ue75dlrsoazf
- Validation: Fork ue75dlrsoazf read root AGENTS.md, .agents/skills/testing/SKILL.md, and tests/AGENTS.md before edits.; castor test:llm-real --filter=SubagentRetrieveLiveE2eTest OK (1 test, 23 assertions; PHPUnit 12.698s; wall 14.561s); castor phpstan OK (0 errors); castor deptrac OK (0 violations); castor cs-check OK (0 files to fix); Proxy cache entries 113 -> 116 during retrieve run; fork notes this likely means scenario had cache misses and can be warmed before strict cache-guard gate if needed.; Full castor check not run by fork.
- Summary: Final cleanup fork completed at commit b01855934 (chore(tests): align subagent live E2E isolation and settings example). Changed SubagentRetrieveLiveE2eTest to use TestDirectoryIsolation::ensureDirectory() for .hatfield/agents instead of ad-hoc mkdir, matching SubagentParallelLiveE2eTest and tests/AGENTS.md isolation conventions. Added commented max_agents: 8 to .hatfield/settings.yaml agents example to keep project-local example aligned with docs/settings.md. Test behavior unchanged. Worktree clean per fork; not pushed yet.

## Task workflow update - 2026-06-25T01:55:47.129Z
- Summary: First CODE-REVIEW transition attempt at b01855934 failed during move_task Castor gate because llama-proxy cache guard detected cache growth from 116 to 124 entries during castor check. Gate output instructed to warm proxy cache with castor test:llm-real and rerun. Task remains IN-PROGRESS; branch is pushed.

## Task workflow update - 2026-06-25T02:00:45.256Z
- Summary: Direct castor check attempt after warming could not acquire the new Castor check lock. Diagnostics showed stale holder metadata pid 790601 missing, but lsof revealed current-user orphaned messenger:consume workers from the failed gate still holding the Symfony lock file open (PIDs 791090 llm, 791093-791100 tool, 791102 agent, 791106 mcp) in the AGENT-07 worktree. This appears to be a cleanup/lifecycle leak after the cache-guard failure, not an active check. Proceeding with explicit current-user worker cleanup for this checkout before rerunning castor check.

## Task workflow update - 2026-06-25T02:06:39.755Z
- Recorded fork run: 4sj7l1ihteir
- Validation: Fork 4sj7l1ihteir read root AGENTS.md, .agents/skills/testing/SKILL.md, and tests/AGENTS.md before edits.; composer dump-autoload -q OK in worktree; castor test --filter=SubagentExecuteToolCallRoutingMiddlewareTest OK (5 tests, 8 assertions); castor test --filter='SubagentExecuteToolCallRoutingMiddlewareTest\|McpExecuteToolCallRoutingMiddlewareTest' OK (13 tests, 20 assertions); castor cs-check OK (0 files to fix); castor phpstan OK (0 errors); castor deptrac OK (0 violations); castor check qa-20260625-020520-801522-6d57bb4a: all functional lanes OK including main test lane (3560 tests), but command failed cache guard because llama-proxy entries grew 124 -> 125.
- Summary: Fix fork completed at local commit 41187e704. Root cause of direct castor check test-lane failure: SubagentExecuteToolCallRoutingMiddlewareTest imported Ineersa\CodingAgent\Tests\Mcp\Messenger\TestStack, but TestStack was declared at the bottom of McpExecuteToolCallRoutingMiddlewareTest.php and therefore was not PSR-4 autoloadable in isolated ParaTest workers. Fork extracted it to tests/CodingAgent/Support/Messenger/TestStack.php and updated both MCP and Subagent middleware tests to import Ineersa\CodingAgent\Tests\Support\Messenger\TestStack. Full castor check functional lanes all passed at 41187e704, but cache guard failed 124 -> 125 entries, so cache warm + rerun is still needed before CODE-REVIEW gate.

## Task workflow update - 2026-06-25T02:09:09.766Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (50.0s).
- Pushed task/agent-07-parallel-subagent-execution to origin.
- branch 'task/agent-07-parallel-subagent-execution' set up to track 'origin/task/agent-07-parallel-subagent-execution'.
- PR already exists: https://github.com/ineersa/agent-core/pull/209
- Validation: Final reviewer 97w7waa66qax: APPROVE WITH SUGGESTIONS, no blockers at d105f4327.; castor test --filter='SubagentExecutionServiceTest\|SubagentToolTest\|AgentsConfigTest\|SubagentExecuteToolCallRoutingMiddlewareTest' OK (34 tests, 124 assertions); castor test:llm-real --filter=SubagentParallelLiveE2eTest OK (1 test, 15 assertions; warm wall ~8s; proxy cache stable); Final cleanup ue75dlrsoazf: castor test:llm-real --filter=SubagentRetrieveLiveE2eTest OK (1 test, 23 assertions; wall 14.561s), phpstan/deptrac/cs-check OK; Fix fork 4sj7l1ihteir: extracted Messenger TestStack to tests/CodingAgent/Support/Messenger/TestStack.php; routing tests OK, phpstan/deptrac/cs-check OK; After warming full llm-real, direct castor check qa-20260625-020710-810306-9455c390 passed: deptrac OK, test OK (3560 tests, 11316 assertions), controller-replay OK (7 tests, 97 assertions), test:tui OK (13 tests, 66 assertions), test:llm-real OK (9 tests, 110 assertions), phpstan OK, cs-check OK; llama-proxy cache guard stable 125 -> 125; QA artifact integrity OK; leak check OK; quality OK in 151.2s.; Branch pushed to origin at 41187e704 before CODE-REVIEW transition.
- Summary: AGENT-07 ready for CODE-REVIEW at HEAD 41187e704. Implements parallel subagent execution with agents.max_agents cap, LLM-visible tasks argument shape, concurrent child launches up to cap, fail-fast validation over cap, overall failure on any failed child with per-child artifact report, and launch-abort cleanup that avoids phantom artifacts. Adds dedicated agent Messenger transport for built-in subagent ExecuteToolCall so foreground parent subagent orchestration does not occupy generic tool workers needed by child tools. Branch refreshed with origin/main de8e9b03b, docs updated for agent_{runId}, live E2E tests aligned with new isolation/performance conventions, and final TestStack helper extracted to autoloadable test support for ParaTest isolation. Final reviewer fork 97w7waa66qax approved with suggestions at d105f4327; subsequent commits were limited test isolation/settings cleanup and autoload helper fix validated by full castor check.

## Task workflow update - 2026-06-25T02:10:56.032Z
- Moved CODE-REVIEW → DONE.
- Merged task/agent-07-parallel-subagent-execution into integration checkout.
- Merge made by the 'ort' strategy.
 .env                                               |   1 +
 .hatfield/settings.yaml                            |   1 +
 config/packages/messenger.yaml                     |   9 +
 config/packages/test/messenger.yaml                |   3 +
 config/services.yaml                               |   6 +
 docs/agents.md                                     |  51 ++-
 docs/async-runtime-architecture.md                 |  58 ++-
 docs/settings.md                                   |   7 +
 .../Agent/Execution/SubagentArgumentsDTO.php       | 107 +++++
 .../Agent/Execution/SubagentArgumentsFactory.php   |  98 +++++
 .../Agent/Execution/SubagentExecutionService.php   | 487 ++++++++++++++++++++-
 .../Agent/Execution/SubagentTaskDTO.php            |  31 ++
 .../SubagentExecuteToolCallRoutingMiddleware.php   |  51 +++
 src/CodingAgent/Agent/Tool/SubagentTool.php        | 126 +++---
 src/CodingAgent/Config/AgentsConfig.php            |  14 +-
 .../Runtime/Controller/HeadlessController.php      |   6 +-
 .../Process/JsonlProcessAgentSessionClient.php     |   1 +
 .../Execution/SubagentExecutionServiceTest.php     | 303 +++++++++++++
 ...ubagentExecuteToolCallRoutingMiddlewareTest.php |  94 ++++
 tests/CodingAgent/Agent/Tool/SubagentToolTest.php  | 172 +++-----
 tests/CodingAgent/Config/AgentsConfigTest.php      |   8 +
 .../McpExecuteToolCallRoutingMiddlewareTest.php    |  19 +-
 .../Controller/E2E/ControllerE2eTestCase.php       |   1 +
 .../Controller/E2E/ControllerReplayE2eTestCase.php |   1 +
 .../Controller/E2E/SubagentParallelLiveE2eTest.php | 168 +++++++
 .../Controller/E2E/SubagentRetrieveLiveE2eTest.php |   5 +-
 tests/CodingAgent/Support/Messenger/TestStack.php  |  27 ++
 27 files changed, 1602 insertions(+), 253 deletions(-)
 create mode 100644 src/CodingAgent/Agent/Execution/SubagentArgumentsDTO.php
 create mode 100644 src/CodingAgent/Agent/Execution/SubagentArgumentsFactory.php
 create mode 100644 src/CodingAgent/Agent/Execution/SubagentTaskDTO.php
 create mode 100644 src/CodingAgent/Agent/Messenger/SubagentExecuteToolCallRoutingMiddleware.php
 create mode 100644 tests/CodingAgent/Agent/Messenger/SubagentExecuteToolCallRoutingMiddlewareTest.php
 create mode 100644 tests/CodingAgent/Runtime/Controller/E2E/SubagentParallelLiveE2eTest.php
 create mode 100644 tests/CodingAgent/Support/Messenger/TestStack.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/agent-07-parallel-subagent-execution.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/agent-07-parallel-subagent-execution.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Pre-DONE CODE-REVIEW transition gate passed: castor check passed during move_task (50.0s).; Direct pre-transition castor check qa-20260625-020710-810306-9455c390 passed with quality OK, llama-proxy cache stable 125 -> 125, QA artifact integrity OK, and leak check OK.
- Summary: User confirmed AGENT-07 PR was merged. Moving task to DONE and cleaning up task worktree. Final branch HEAD before merge/cleanup was 41187e704 with direct castor check passing and CODE-REVIEW gate passing.

## Task workflow update - 2026-06-25T02:12:35.704Z
- Validation: Post-merge castor check qa-20260625-021113-824293-9254ab7f: deptrac OK; controller-replay OK (7 tests, 97 assertions); test:tui OK (13 tests, 66 assertions); test:llm-real OK (9 tests, 110 assertions); phpstan OK; cs-check OK; cache guard OK 125 -> 125; leak check OK; main test lane FAIL due TuiExportCommandVirtualTest filename wrapping assertion.; Focused castor test --filter=TuiExportCommandVirtualTest FAIL reproduced deterministic wrapping assertion: rendered path contains hatfield-session-virtual-export-session.ht\nml instead of contiguous filename.
- Summary: Post-merge integration validation after moving AGENT-07 to DONE did not pass. Full castor check on integration checkout ran qa-20260625-021113-824293-9254ab7f: deptrac OK, controller-replay OK, test:tui OK, test:llm-real OK, phpstan OK, cs-check OK, llama-proxy cache guard stable 125 -> 125, QA artifact integrity OK, leak check OK, but main test lane failed. Failure is deterministic in TuiExportCommandVirtualTest::testExportSlashCommandRoutesLocallyRendersConfirmationAndWritesHtml: rendered export filename wraps at terminal width as 'hatfield-session-virtual-export-session.ht\nml', so assertion looking for contiguous 'hatfield-session-virtual-export-session.html' fails. Focused castor test --filter=TuiExportCommandVirtualTest reproduced the same failure. This appears to be a TUI virtual test assertion/terminal wrapping issue rather than AGENT-07 parallel subagent behavior.
