# Add agent_resume for completed subagent continuity

## Goal
Design locked after grill-me on token-efficiency / review-iterate waste, then explain lock-ins.

## Problem
Cold-start re-reviews and re-scouts redo Phase-1 context gathering. Parent often has the artifact id + a short delta task, but today’s `subagent` always launches a fresh child. Live-view already can `follow_up` a selected child; orchestrators need the same continuity via a tool.

## Product surface (v1)
- New parent-orchestrator tool: **`agent_resume`**
- Not an extension of `subagent` launch params
- Humans keep using `/agents-live` follow_up unchanged
- Fork is out of scope for v1 (separate later)

## Locked decisions (post-explain)

### Entry / API
- Dedicated tool: `agent_resume` (not an extension of `subagent` launch params)
- Single: `{artifact_id, task}` (optional `agent_run_id` like retrieve; artifact_id primary)
- Parallel: `{tasks:[{artifact_id, task}, ...]}` same shape as `subagent`, capped by `agents.max_agents`
- Model does not emit parallel tool-calls; parallelism is via `tasks[]` on the tool

### Lifetime / scope
- Current parent session only; must **not** survive parent `/resume`
- Parent compaction is irrelevant: resume uses child run_id/artifact from current parent registry

### Continuity / transport
- True continuity: `follow_up` into the **existing child session/run**
- Child continues on messenger transport; parent tool uses deferred completion (reuse/extend deferred subagent batch machinery)
- Do not rebuild/relaunch a new child with seeded handoff in v1

### Blocking / concurrency
- Block until finished (same foreground deferred semantics as today’s `subagent`)
- In-flight: reject if artifact status is `running` or `needs_clarification`; allow if terminal
- Reject same-artifact concurrent resume; allow different artifacts concurrently
- Parent steering of running subagents is a separate future task

### Eligibility
- All non-fork subagent roles (scout/reviewer/researcher/architect/browser/etc.)
- Fork out of scope for v1
- Eligible statuses: completed + failed + cancelled when child run/session is still usable
- Failed/cancelled: **try resume**; refuse with clear error only if unusable
- Context-size guardrail: refuse resume when `latestInputTokens >= max(0.75 * contextWindow, 200_000)`; if `contextWindow` missing, use absolute `200_000`
- No frontmatter `resumeAllowed` gate in v1

### Artifact identity / handoffs / parent result
- Keep the **same artifact id**
- Parent-visible result **mirrors `subagent`**: single = full **latest** handoff inline; parallel = bounded summaries
- Handoff storage: keep `handoff.md` as latest; on each finalize, archive prior content to `handoffs/<n>.md` and update `handoffs/index.json`
- `agent_retrieve`: `mode=handoff` remains latest only; add `mode=handoff_history` (list by default; pass index/n to fetch one older body)
- Compact-always parent result was rejected (parent won’t reliably retrieve)

### Non-goals (v1)
- No parent `/resume` survival
- No fork resume
- No changing human live-view UX
- No parent-steer-of-running-children API
- No relaxing “main agent never edits files” rule (revisit after resume ships)
- No requirement that resume rewrite reviewer prompts; workflow/prompt delta-verify rules stay in harden-task-workflow task

## Suggested acceptance focus
- Parent can `agent_resume` a just-finished reviewer with a short “verify these fixes / review delta only” task
- Child retains prior transcript/tool history
- Same artifact id updated; parent gets full latest handoff on single resume (mirrors subagent)
- Prior handoffs archived and retrievable via `handoff_history`
- Rejects in-flight same-artifact resume; allows different artifacts concurrently via `tasks[]`
- Works for completed/failed/cancelled terminal children in current parent session only (try-resume)
- Live-view follow_up behavior unchanged
- Docs/skills updated for launch-vs-continue usage

## Acceptance criteria
- New parent tool `agent_resume` exists (single + `tasks[]` parallel) and is orchestrator-facing
- Resume sends `follow_up` to the existing child run_id and waits via deferred/messenger batch completion until terminal again
- Resume is limited to the current parent session artifact registry and does not survive parent `/resume`
- All non-fork subagent roles can be resumed; fork remains out of scope
- Terminal children in completed, failed, or cancelled status are eligible when the child run/session is still usable; unusable sessions refuse clearly
- Same artifact id is preserved across resumes
- Parent-visible resume result mirrors `subagent` (single full latest handoff; parallel bounded summaries)
- On finalize, previous `handoff.md` is archived under `handoffs/<n>.md` with `handoffs/index.json`; `handoff.md` remains latest
- `agent_retrieve` keeps `mode=handoff` as latest; adds `mode=handoff_history` list + index/n body fetch
- In-flight same-artifact resume (`running` / `needs_clarification`) is rejected; different artifacts may resume concurrently
- Context-size guardrail refuses oversized children per locked formula
- Human `/agents-live` follow_up path remains unchanged in v1
- Docs/skills describe when to use `agent_resume` vs launching a fresh `subagent`

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-20-add-agent-resume-for-completed-subagent-continuity
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity
Fork run: 561c58xdlv7i
PR URL: https://github.com/ineersa/agent-core/pull/420
PR Status: merged
Started: 2026-08-20T20:13:01+00:00
Completed: 2026-08-24T18:24:30.119Z

## Work log
- Created: 2026-08-20T16:57:22+00:00

## Task workflow update - 2026-08-20T17:28:45+00:00
- Summary: Clarified after Claude Code scout + user feedback: dedicated agent_resume (not on subagent launch); parallel via tasks[] like subagent (not model multi-tool-calls); resumed children continue on messenger/deferred transport; no frontmatter gate in v1; add context-size resume guardrails; borrow Claude Code launch-vs-continue prompting.
- Clarification: parent meant messenger/deferred transport semantics (like subagent), not putting resume onto the subagent launch tool.
- API remains dedicated agent_resume (Option A). Parallelism: support tasks[] the same way as subagent because the model does not emit parallel tool calls; cap with agents.max_agents.
- Implementation: follow_up existing child run_id; child continues on its messenger transport; parent tool uses deferred completion and waits for terminal outcome (same foreground deferred pattern as subagent).
- Frontmatter resumeAllowed deferred; all non-fork subagents resumable in v1.
- New guardrail candidate: refuse resume when child context is near limit (e.g. ~75% of window or absolute ~200k+ tokens) — exact threshold TBD during implement/explain; purpose is to stop parent abusing one fat child forever.
- Prompting inspiration from Claude Code Agent/SendMessage: launch tool stays fresh-only; continue tool preserves full context; tell model when to continue vs spawn fresh; for Hatfield, encode that as subagent vs agent_resume docs/skill text.
- Result shape still compact delta to parent by default; full via agent_retrieve.

## Task workflow update - 2026-08-20T18:30:00+00:00
- Summary: Explain locked: dedicated agent_resume with tasks[] parallel; follow_up + deferred batch; parent results mirror subagent (single=full latest handoff); try failed/cancelled; in-flight reject on running/needs_clarification; context refuse at max(75% window, 200k); handoff.md latest + handoffs/<n>.md archive + index.json; retrieve mode=handoff latest, mode=handoff_history list/fetch by index.
- Explain complete with user lock-ins.
- Parent resume result mirrors subagent: single full latest handoff inline; parallel bounded summaries. Compact-always rejected.
- Handoff history: keep handoff.md as latest; archive prior to handoffs/<n>.md + handoffs/index.json.
- agent_retrieve: mode=handoff stays latest; new mode=handoff_history lists by default and fetches one body by index/n.
- In-flight: reject running/needs_clarification; allow terminal.
- Failed/cancelled: try resume; refuse only if session/run unusable.
- Context guardrail: refuse at max(75% contextWindow, 200k); missing window -> absolute 200k.
- Deferred wait: reuse/extend deferred subagent batch machinery.
- No frontmatter resumeAllowed; no parent-steer; no fork resume; live-view unchanged.

## Task workflow update - 2026-08-20T20:13:01+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-20-add-agent-resume-for-completed-subagent-continuity.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity.

## Task workflow update - 2026-08-21T13:57:04+00:00
- Merged fresh origin/main into worktree (fast-forward to fc8697682). vendor present; hatfield-extension-api is path symlink.

## Task workflow update - 2026-08-21T14:39:35+00:00
- Recorded fork run: agent_6e7526c0865149c8
- Summary: First fork cancelled after hanging validation. Left uncommitted partial agent_resume + handoff history implementation; no AgentResume* tests on disk. Launching corrective fork with Castor-only validation rules.
- First fork cancelled by user after repeated hanging tests; reportedly used raw phpunit and bash timeouts failed.
- Worktree remains dirty with partial implementation: AgentResume* tool/execution files, DeferredSubagentChildRepository rebind path, handoff archive/retrieve updates, docs/skills. No commit yet.
- No AgentResume* test files present; only small AgentArtifactRegistryTest + routing middleware test edits.
- Corrective fork requested: finish remaining work, Castor-only tests, commit.

## Task workflow update - 2026-08-21T16:07:10+00:00
- Recorded fork run: agent_631b72c95fc3ada7
- Validation: castor test --filter=testWriteHandoffArchivesPreviousNonEmptyHandoff OK 1/1; castor test --filter=testHandoffHistoryListsArchivesAndFetchesByIndex OK 1/1; castor test --filter=AgentResumeExecutionServiceTest OK 8/8; castor test --filter=testStampsAgentTransportForAgentResumeTool OK 1/1; castor deptrac 0 violations; castor phpstan scoped 0 errors; castor cs-check clean
- Summary: Corrective fork committed 6fac187b2: fixed AgentArtifactRegistry FlockStore reentrancy deadlock (create→writeHandoff hang), finished agent_resume + handoff history, focused Castor tests green. No raw phpunit. Remaining gap: no full deferred resume lifecycle proof; confirm deferred completion handles agent_resume tool name.
- Root cause of prior hang: AgentArtifactRegistry create()/ensureReserved held parent FlockStore lock then called writeHandoff() which reacquired same lock → infinite hang; PHPUnit banner + Castor 210s timeout with 0 tests.
- Fix: persistPendingEntry writes empty handoff via writeHandoffInternal under existing lock.
- Committed 6fac187b2 with agent_resume stack, child rebind, handoff archive/retrieve, docs/skills, focused tests.

## Task workflow update - 2026-08-21T16:21:37+00:00
- Recorded fork run: agent_31415dd8eb07471f
- Validation: castor test --filter=testInsertReservedChildrenRebindsExistingTerminalChildOntoNewBatch OK 1/1 (22 assertions); castor test --filter=testAgentResumeToolNameDeferredLifecyclePreservesToolNameThroughCompleteDeferredToolCall OK 1/1 (9 assertions); castor cs-check clean
- Summary: Test-only follow-up committed 4d749e072: rebind unique-conflict DB proof + agent_resume deferred completion tool-name identity. Both Castor-green. Ready for task-to-pr/review.
- Added DeferredSubagentBatchLaunchTest::testInsertReservedChildrenRebindsExistingTerminalChildOntoNewBatch — locks resume unique-conflict rebind (cursor/tokens preserved, terminal cleared).
- Added DeferredSubagentBatchLifecycleTest::testAgentResumeToolNameDeferredLifecyclePreservesToolNameThroughCompleteDeferredToolCall — locks CompleteDeferredToolCall preserves tool_name=agent_resume.
- Note: progress delivery still hardcodes toolName subagent (cosmetic only); completion path correct.

## Task workflow update - 2026-08-21T16:46:51+00:00
- Summary: task-to-pr reviewer REQUEST CHANGES at 4d749e072: (1) followUp partial-failure leaves artifact Running / batch Reserved — mirror launch failure handling; (2) archive index stamps new run summary onto old handoff due to finalizer order; (3) castor check required at CODE-REVIEW move. Launching fix fork.

## Task workflow update - 2026-08-21T17:16:25+00:00
- Recorded fork run: agent_1e540a411cb82e5d
- Validation: castor test focused AgentResume/handoff/rebind/lifecycle/routing OK 14/14 (73 assertions); castor deptrac 0 violations; castor phpstan scoped 0 errors; castor cs-check clean
- Summary: Re-review APPROVE WITH SUGGESTIONS at bca1ce7d5. Focused Castor validation green (14 tests). Moving to CODE-REVIEW for castor check + PR.
- Reviewer re-review APPROVE WITH SUGGESTIONS (agent_04f91ad493440621): prior BUG/EDGE fixes verified; remaining NTHs only (applyLaunchSuccessState unwrap, archivedMeta DTO typing, progress toolName).
- HEAD chain: 6fac187b2 feat → 4d749e072 tests → bca1ce7d5 failure/archive fixes.

## Task workflow update - 2026-08-21T17:22:19+00:00
- Validation: castor check qa-20260821-171627-9015-c68257f1 FAILED: phpstan 1 error DeferredSubagentChildRepository serialize; test 1 error missing AgentResumeTasksSchemaProvider; controller-replay/llm-real/tui same schema provider error
- Summary: move_task CODE-REVIEW failed: castor check blocked by missing AgentResumeTasksSchemaProvider registration in SchemaAttributeDescriber providers map (broke unit/controller-replay/llm-real/tui) + ChildRepository phpstan serialize() typing. Fixing via fork.
- CODE-REVIEW transition failed; task remains IN-PROGRESS.
- Root cause: AgentResumeTasksSchemaProvider not added to services.yaml SchemaAttributeDescriber $providers alongside SubagentTasksSchemaProvider.

## Task workflow update - 2026-08-21T17:22:50+00:00
- Recorded fork run: agent_5d902ff005ccd3c6
- Validation: castor test --filter=Gf05BareAgentsEffectiveContextIntegrationTest OK 1/1; castor phpstan --path=DeferredSubagentChildRepository.php 0 errors; castor cs-check clean
- Summary: Gate fixes committed fcb7d6e47: registered AgentResumeTasksSchemaProvider; ChildRepository serializer intersection type. Retrying CODE-REVIEW move.

## Task workflow update - 2026-08-21T17:28:27+00:00
- Validation: qa-20260821-172253-11040-281d1813: phpstan 0; controller-replay OK 13; llm-real OK 13; tui OK 41; unit FAIL AgentRetrieveToolTest promptLines
- Summary: Second CODE-REVIEW attempt failed only on AgentRetrieveToolTest promptLines not including handoff_history blurb. Schema/phpstan/controller/llm/tui lanes green. Fixing test expectation then retry.

## Task workflow update - 2026-08-21T17:28:40+00:00
- Recorded fork run: agent_b143d8f88017dc72
- Validation: castor test --filter=testDefinitionHasCorrectNameAndSchema OK 1/1
- Summary: AgentRetrieveToolTest promptLines aligned (e79aaa618). Retrying CODE-REVIEW move.

## Task workflow update - 2026-08-21T17:43:30+00:00
- Summary: Third CODE-REVIEW attempt: all lanes green except test:tui Castor hard timeout after 200s (no junit written). Prior attempt TUI OK in 181s. Likely wall-budget clamp + concurrent TUI load from sibling worktree 2026-08-21-fix-fork-bash-timeout. Investigating; not a product regression in agent_resume.
- qa-20260821-172842-21405-35d347cc: unit OK 4801, controller-replay OK 13, llm-real OK 13, phpstan 0, docs/cs/deptrac ok; test:tui timed out at 200s with empty progress after ParaTest banner.
- Sibling worktree 2026-08-21-fix-fork-bash-timeout had concurrent castor test:tui around 13:32–13:42; leftover QA tmux session name seen afterward.
- No stale QA workers in agent-core checkout; agent-resume TUI timeout looks environmental/contention, not AgentResume schema failure.

## Task workflow update - 2026-08-21T17:45:43+00:00
- Summary: TUI timeout investigation: cannot name a hung test — lane killed at 200s with only ParaTest banner and no junit. Gate gave test:tui hard timeout 200s (wall remaining 200/210). Prior green TUI was 181s (19s margin). Concurrent sibling TUI activity around same window. Not attributable to a specific agent_resume TUI test.
- Castor check stdout (bg 24103d4ccd8e8c58): test:tui hard timeout 200s; wall remaining 200s / max 210s.
- Timed-out lane log has no per-test progress and no phpunit-tui.junit.xml — hard kill before PHPUnit finished writing results.
- Prior green TUI junit sum of case times ~342s across 2 workers; wall 181s. Slowest cases ~21s ReloadSettings, ~13s FileRewind, ~12s SubagentLiveView.
- Conclusion: environmental/contention + thin wall margin; not a named agent_resume TUI regression.

## Task workflow update - 2026-08-21T17:50:20+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (197.2s).
- Pushed task/2026-08-20-add-agent-resume-for-completed-subagent-continuity to origin.
- branch 'task/2026-08-20-add-agent-resume-for-completed-subagent-continuity' set up to track 'origin/task/2026-08-20-add-agent-resume-for-completed-subagent-continuity'.
- Created PR: https://github.com/ineersa/agent-core/pull/420

## Task workflow update - 2026-08-21T17:50:28+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/420
- Updated PR Status: open
- Validation: castor check passed (197.2s) during CODE-REVIEW move; focused Castor tests green earlier; deptrac/phpstan/cs clean
- Summary: task-to-pr complete: reviewer APPROVE WITH SUGGESTIONS; castor check passed (197.2s); PR #420 open at HEAD e79aaa618.

## Task workflow update - 2026-08-22T00:17:36+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Review-iterate: address PR #420 comments + live UX bugs (picker stale status, Handoff rename, archive status mismatch, guideline trim, DTO validation move, SerializerInterface alignment).

## Task workflow update - 2026-08-22T00:30:00+00:00
- Recorded fork run: agent_f9701badbe4549fa
- Validation: castor test --filter='SubagentLiveCatalogTest|testWriteHandoffArchivesPreviousNonEmptyHandoff|testWriteHandoffArchiveStatusPrefersBodyOverRunningRegistryMeta|AgentResumeExecutionServiceTest|testSingleModeNaturalTerminalPresentation' OK 26/81; castor deptrac 0; castor phpstan 0 on touched paths; castor cs-check clean
- Summary: Review-iterate fork landed 2fa6142d2: picker reopen on changed taskSummary, Running-before-followUp, Handoff: heading, archive status from body, drop reject guideline, DTO duplicate validation, SerializerInterface-only ChildRepository.

## Task workflow update - 2026-08-22T00:54:04+00:00
- Recorded fork run: agent_5f4ba0b4a6fb90d1
- Validation: castor test --filter=AgentResumeExecutionServiceTest OK 10/27; castor test --filter=SubagentLiveCatalogTest OK 15/47; castor test --filter=testWriteHandoffArchiveStatusPrefersBodyOverRunningRegistryMeta OK 1/3; castor deptrac 0; castor phpstan 0 on touched paths; castor cs-check clean
- Summary: Corrective commit 520c600e3: restore resolved-identity resume dedupe; revert Running→prior terminal on followUp failure; drop weak DTO Callback; merge ChildRepository catch.

## Task workflow update - 2026-08-22T00:57:44+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (186.0s).
- Pushed task/2026-08-20-add-agent-resume-for-completed-subagent-continuity to origin.
- branch 'task/2026-08-20-add-agent-resume-for-completed-subagent-continuity' set up to track 'origin/task/2026-08-20-add-agent-resume-for-completed-subagent-continuity'.
- PR already exists: https://github.com/ineersa/agent-core/pull/420

## Task workflow update - 2026-08-22T00:57:58+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/420
- Updated PR Status: open
- Validation: castor check passed (186.0s) during CODE-REVIEW move
- Summary: Review-iterate complete: reviewer APPROVE WITH SUGGESTIONS; castor check passed (186.0s); PR #420 updated at HEAD 520c600e3.

## Task workflow update - 2026-08-22T02:18:28+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Review-iterate: immutable UUID handoffs (drop mutable handoff.md); retrieve via handoff_id; drop index; address remaining PR comments with locked defaults.

## Task workflow update - 2026-08-22T02:41:14+00:00
- Recorded fork run: agent_de1dc8d8992c9704
- Validation: castor test --filter='AgentArtifactRegistryTest|AgentArtifactRetrievalServiceTest|AgentRetrieveToolTest|AgentResume' OK 82/247; castor deptrac 0; castor phpstan 0; castor cs-check clean
- Summary: Immutable UUID handoffs landed (96462ec83 + c9179bc95): dropped mutable handoff.md/archivedMeta/parse/index=n; retrieve via handoff_id; AgentResume ctor trim.

## Task workflow update - 2026-08-22T03:04:21+00:00
- Recorded fork run: agent_2c3499c8b2e57e89
- Validation: castor test --filter='AgentArtifactRegistryTest|AgentArtifactRetrievalServiceTest|AgentRetrieveToolTest|AgentResume' OK 83/250; castor deptrac 0; castor phpstan 0; castor cs-check clean; castor docs:validate ok
- Summary: NTH follow-up 0c6efa974: id-derived handoff paths (ignore index path on read), migration note, dead Length/''=== removed, DESCRIPTION wording.

## Task workflow update - 2026-08-24T16:59:23.227Z
- Summary: Salvage requested: reconcile the existing stale/incomplete merge with latest origin/main, audit and rewrite task tests against current hardened test standards, then run one fresh reviewer cycle and return to CODE-REVIEW after required fixes and Castor gates.
- Main orchestrator read and will enforce current `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`. Worktree currently has an unfinished merge (MERGE_HEAD bfbbe11b2) while origin/main is newer (a6bb60e45), with conflicts in ForkDeferredLiveE2eTest.php and SubagentChildHitlEventsFixture.php.

## Task workflow update - 2026-08-24T17:00:17.838Z
- Recorded fork run: 62wx2dyq1i85
- Launched one salvage implementation fork in the exact task worktree. Scope: safely replace/complete stale merge with latest origin/main, audit task tests against hardened standards, fix merge/task regressions, run Castor-only focused/full validation, and commit cleanly without push.

## Task workflow update - 2026-08-24T17:23:56.228Z
- Recorded fork run: 62wx2dyq1i85
- Validation: Read/followed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`.; Focused agent_resume/artifact/deferred/live-catalog tests: 124 tests, 662 assertions, green.; castor test: 4812 tests, 19601 assertions, green.; castor test:controller-replay: 6 tests, 92 assertions, green.; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean.; castor check: passed in 143.5s; all lanes and worker/cache guards green; task-specific max case 0.127s, new case 0.045s.
- Summary: Salvage fork completed cleanly: aborted the obsolete interrupted merge after proving it contained no bespoke work, merged latest origin/main a6bb60e45 in d4d45e298, retained hardened-main conflict resolutions, audited agent_resume tests, and added deterministic failed/cancelled parallel resume coverage in 396c2bc2c. Worktree clean; ready for fresh reviewer.

## Task workflow update - 2026-08-24T17:36:23.902Z
- Summary: Fresh reviewer REQUEST CHANGES after salvage. Blocking fixes: failure-safe launch-success persistence; replace stale skill `index` argument with `handoff_id`; enforce locked current-parent-lifetime rule so agent_resume does not survive parent /resume; give agent_resume subagent-equivalent document output cap; add primary child tool-policy strip assertion.
- Reviewer read current testing standards and accepted the hardened lowest-layer proof strategy, including deletion of the obsolete live fork test and the new failed/cancelled parallel resume test. No critical/security issues found. Non-blocking same-task picker heuristic/projection-version/progress-label observations deferred.

## Task workflow update - 2026-08-24T17:37:34.246Z
- Recorded fork run: 561c58xdlv7i
- Launched one corrective fork for the reviewer blockers only: launch-success failure recovery, stale skill argument, current-parent `/resume` lifetime enforcement, output-cap parity, and child policy-strip proof. No NTH scope.

## Task workflow update - 2026-08-24T17:49:44.296Z
- Recorded fork run: 561c58xdlv7i
- Validation: Focused corrective tests: 45 tests, 150 assertions, green.; castor test: 4815 tests, 19609 assertions, green.; castor test:controller-replay: 6 tests, 92 assertions, green.; castor deptrac 0 violations; castor phpstan 0 errors; castor cs-check clean.; castor check: passed in 150.3s; all lanes/cache/leak guards green; new tests 0.025s and 0.038s, task-specific max 0.147s.
- Summary: Corrective fork committed 9e8b74bbc and addressed all reviewer blockers: recoverable launch-success persistence degradation, immutable handoff skill wording, internal parent-lifetime rotation on start/attach with persistent retrieval retained, document output-cap parity, and child policy-strip proof. Worktree clean.

## Task workflow update - 2026-08-24T17:56:16.018Z
- Validation: Reviewer read root/testing/test standards and task file; verified deterministic lowest-layer tests and recorded castor check evidence.; Ready for CODE-REVIEW at HEAD 9e8b74bbc.
- Summary: Targeted re-review APPROVE WITH SUGGESTIONS at 9e8b74bbc. All five blockers verified resolved; no critical, bug, security, architecture, or required-proof findings remain. Suggestions are documentation/extra fail-closed coverage only and do not block.

## Task workflow update - 2026-08-24T17:57:42.810Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (75.9s).
- Pushed task/2026-08-20-add-agent-resume-for-completed-subagent-continuity to origin.
- branch 'task/2026-08-20-add-agent-resume-for-completed-subagent-continuity' set up to track 'origin/task/2026-08-20-add-agent-resume-for-completed-subagent-continuity'.
- PR already exists: https://github.com/ineersa/agent-core/pull/420
- Validation: Focused agent_resume/artifact/deferred/live-catalog tests green.; castor test green: 4815 tests, 19609 assertions.; castor test:controller-replay green: 6 tests, 92 assertions.; castor deptrac 0 violations; castor phpstan 0 errors; castor cs-check clean.; Pre-transition castor check passed in 150.3s; task-specific tests all under 0.15s.
- Summary: Salvaged onto latest origin/main, modernized tests under hardened standards, fixed all reviewer blockers, and received final APPROVE WITH SUGGESTIONS at 9e8b74bbc.

## Task workflow update - 2026-08-24T17:57:48.468Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/420
- Updated PR Status: open
- Validation: Deterministic CODE-REVIEW transition castor check passed in 75.9s.
- Summary: Task-to-PR salvage complete. Branch pushed at 9e8b74bbc; PR #420 updated; task is CODE-REVIEW.

## Task workflow update - 2026-08-24T18:24:30.119Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity: ide_close_project returned isError.
- Merged task/2026-08-20-add-agent-resume-for-completed-subagent-continuity into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity.
- Pulled integration checkout: Already up to date..
- Validation: Confirmed PR #420 state MERGED at 2026-08-24T18:23:20Z.; Integration checkout clean before DONE transition.
- Summary: PR #420 merged on GitHub at 10fadd0144927152eac6c15999104c617b7f198c. Moving task to DONE and integrating/cleaning the task worktree.

## Task workflow update - 2026-08-24T18:26:07.222Z
- Updated PR Status: merged
- Validation: Post-merge `LLM_MODE=true castor check` passed in 140.7s (qa-20260824-182440-280577-5c5c5a62): 4815 unit/integration tests, controller replay 6, TUI 8, llm-real 5, deptrac/phpstan/cs/docs/catalog all green; leak and llama-proxy cache guards passed.; Integration checkout clean and aligned with origin/main.
- Summary: DONE complete: PR #420 merged, integration main synchronized at 879f9d3f9, task worktree removed, and post-merge full Castor gate passed.

## Task workflow update - 2026-08-29T16:09:38.945Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
