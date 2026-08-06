# COMP-04 Compaction hooks, observability, and TUI event projection

## Goal
Plan reference: `.pi/plans/context-compaction-implementation-plan.md` sections 7, 14.2, 16, 18, 19.3, 21 Phase 1.

Scope:
- Add before-compaction hook contracts for cancel/replacement summary/additional instructions/metadata.
- Ensure after-compaction observation works through committed events and existing after-turn/event hooks.
- Project `context_compaction_started`, `context_compacted`, and `context_compaction_failed` into runtime/TUI-visible events/status as needed.
- Add structured logging for compaction lifecycle without raw prompts or full session content.

Core compaction services already landed (COMP-01): production classes under `Ineersa\CodingAgent\Compaction` (`SessionCompactor`, `CompactionPreparationDTO`, `CompactionPreparationResultDTO`, `CompactionSkipReasonEnum`, `CompactResultDTO`, etc.). Hook DTOs should be in the same compaction namespace.

Execution order: depends on COMP-02. Can run in parallel with COMP-03 after COMP-02 lands, but coordinate on runtime/TUI event names.

## Acceptance criteria
- Before-compaction hooks can cancel compaction with a reason, provide replacement summary, append instructions, and attach metadata.
- Replacement summary path skips the LLM call but still emits the mandatory lifecycle events and compacted checkpoint.
- Existing after-turn/event hooks can observe committed `context_compacted` events with compacted state.
- Runtime/TUI projection surfaces start/success/failure lifecycle without leaking raw prompts or full summaries by default.
- Structured logs include correlation fields such as `run_id`, `session_id`, `component`, and `event_type` and do not include raw prompts/tool output/full session content.
- Relevant Castor tests pass; final PR must pass `castor check` if runtime/TUI projection is touched.

## Workflow metadata
Status: DONE
Branch: task/comp-04-compaction-hooks-and-observability
Worktree: /home/ineersa/projects/agent-core-worktrees/comp-04-compaction-hooks-and-observability
Fork run: iaaeplddeqw8
PR URL: https://github.com/ineersa/agent-core/pull/188
PR Status: merged
Started: 2026-06-21T20:14:03.161Z
Completed: 2026-06-21T22:03:10.443Z

## Work log
- Created: 2026-06-08T15:40:15.290Z

## Task workflow update - 2026-06-20T23:15:30.750Z
- Summary: Updated after COMP-02 merge (PR #184). Observability/projection should integrate with the landed canonical compaction events: `context_compaction_started`, `context_compacted`, and `context_compaction_failed`. Failure payloads use `messages_replaced=false` as the canonical field (not `preserved_messages`); result-handler failures include `step_id` for replay fidelity, structural pre-start failures do not. AgentCore replay already handles activeStepId clearing/preservation for started/compacted/failed events. Hooks should remain in CodingAgent/Compaction product layer; do not add Hatfield-specific concepts such as `thinking_level` taxonomy to AgentCore. If hooks alter model invocation options, pass them through CodingAgent into generic `modelOptions` rather than adding named AgentCore fields.

## Task workflow update - 2026-06-21T20:14:03.161Z
- Moved TODO → IN-PROGRESS.
- Created branch task/comp-04-compaction-hooks-and-observability.
- Created worktree /home/ineersa/projects/agent-core-worktrees/comp-04-compaction-hooks-and-observability.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/comp-04-compaction-hooks-and-observability.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/comp-04-compaction-hooks-and-observability.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/comp-04-compaction-hooks-and-observability.
- Summary: Starting task-start phase for COMP-04. Main agent is orchestrator; implementation will be delegated to a fork after reading task/plans and gathering scout context.

## Task workflow update - 2026-06-21T20:18:00.393Z
- Validation: Loaded task-workflow skill before task-start orchestration.; Loaded testing skill and tests/AGENTS.md before preparing test-related fork instructions.; Scout performed read-only exploration and returned concrete file/signature guidance.
- Summary: Scout exploration completed (saved at /home/ineersa/.pi/agent/tmp/2026-06--546573a1.txt). Key findings: primary insertion point is src/CodingAgent/Application/Pipeline/CompactRunHandler.php between compaction preparation and context_compaction_started; CompactionStepResultHandler emits context_compacted/context_compaction_failed; ExecuteCompactionStepWorker already has structured RunLogContext logging; CompactRunHandler and CompactionStepResultHandler lack structured logging; existing after-turn HookDispatcher already observes committed events including context_compacted, but summaries only include seq/type; COMP-03 already added runtime/TUI projection via RuntimeEventTranslator, CompactionProjectionSubscriber, TuiSessionState, RuntimeEventPoller, and TuiCompactCommandE2eTest. Recommended hook design: keep before-compaction hook contracts internal under CodingAgent/Compaction to avoid deptrac issues and because plan says do not expose public ExtensionApi methods yet; use tagged iterable dispatcher; before-hook failure should be logged and non-fatal; first cancel wins; replacement summary skips LLM worker but emits started+compacted lifecycle and checkpoint; additional instructions merge in order; metadata attaches to lifecycle payloads without raw prompt/session content. Existing TUI E2E likely sufficient unless fork changes user-visible TUI behavior.

## Task workflow update - 2026-06-21T20:29:21.927Z
- Recorded fork run: dfayr3tcxsw9
- Validation: git status after fork: clean on task/comp-04-compaction-hooks-and-observability; git cat-file -t 72302bd17: commit; git diff --stat HEAD~1..HEAD: 9 files changed, 1230 insertions, 69 deletions; castor test --filter='CompactionHook|CompactRunHandler|CompactionStepResultHandler' PASS (23 tests, 151 assertions); castor deptrac PASS (violations=0, errors=0); castor cs-check PASS (files_fixed=0); castor test:tui --filter=TuiCompactCommandE2eTest PASS (2 tests, 8 assertions); castor phpstan FAIL: CompactRunHandler::handleReplacementSummary() parameter $hookMetadata missing array value type; CompactionHookResultDTO::__construct() parameter $metadata missing array value type
- Summary: Implementation fork dfayr3tcxsw9 completed commit 72302bd17 and worktree was clean. Main verified commit exists and diff stat matches report: 9 files changed, 1230 insertions, 69 deletions. Implemented internal CodingAgent before-compaction hook interface/DTOs/dispatcher, service tag wiring, CompactRunHandler integration for cancel/replacement/additional-instructions/metadata paths, and structured logging in CompactRunHandler + CompactionStepResultHandler. However fork validation used raw vendor/bin commands, so main reran focused Castor validation. Focused tests, deptrac, cs-check, and focused TUI E2E passed, but castor phpstan failed on missing array value types in new/changed methods. Launching narrow follow-up fork to fix PHPStan annotations only.

## Task workflow update - 2026-06-21T20:31:12.669Z
- Recorded fork run: 3kkdfzuewki6
- Validation: commit 72302bd17: COMP-04 implementation fork dfayr3tcxsw9, 9 files changed, 1230 insertions, 69 deletions; commit f6ce61694: tiny PHPStan annotation fix in CompactRunHandler.php and CompactionHookResultDTO.php; post-fix git status --short --branch: clean on task/comp-04-compaction-hooks-and-observability; post-fix diff stat over HEAD~2..HEAD: 9 files changed, 1235 insertions, 69 deletions; castor test --filter='CompactionHook|CompactRunHandler|CompactionStepResultHandler' PASS before annotation fix (23 tests, 151 assertions); castor deptrac PASS before annotation fix (violations=0, errors=0); castor cs-check PASS before annotation fix (files_fixed=0); castor test:tui --filter=TuiCompactCommandE2eTest PASS before annotation fix (2 tests, 8 assertions); castor phpstan initially failed on missing array value types; fixed by f6ce61694; post-fix castor phpstan PASS (errors=0, file_errors=0); post-fix castor cs-check PASS (files_fixed=0); post-fix castor test --filter='CompactionHook|CompactRunHandler' PASS (17 tests, 100 assertions)
- Summary: Follow-up fork 3kkdfzuewki6 completed commit f6ce61694 on branch task/comp-04-compaction-hooks-and-observability, fixing the two PHPStan missing iterable value type issues from commit 72302bd17. Final implementation state: internal before-compaction hook contracts/DTOs/dispatcher in CodingAgent/Compaction; service tag wiring `coding_agent.before_compaction_hook`; CompactRunHandler hook integration for cancel, replacement summary, additional instructions, and hook metadata; structured safe logging in CompactRunHandler and CompactionStepResultHandler; focused tests for dispatcher aggregation and handler hook paths. Worktree is clean. Per task-start workflow, stopping here and not moving to CODE-REVIEW/reviewer/PR.

## Task workflow update - 2026-06-21T21:31:04.294Z
- Validation: reviewer verdict: REQUEST CHANGES on HEAD f6ce61694; reviewer confirmed existing real TmuxHarness TuiCompactCommandE2eTest remains valid; no new TUI E2E required because COMP-04 made no user-visible TUI changes
- Summary: Reviewer subagent reviewed HEAD f6ce61694 and returned REQUEST CHANGES. Blockers: (1) CompactionHookContextDTO documents safe context/no raw prompts but exposes CompactionPreparationDTO, whose message lists contain raw conversation/tool content; fix by making hook context a safe scalar view or explicitly sensitive accessor, preferred safe scalar view. (2) CompactRunHandler::handleCompaction() retyped runtime settings as bare object and accesses model/thinkingLevel/keepRecentTokens; restore concrete CompactionRuntimeSettingsDTO and confirm phpstan. Additional actionable items: replacement-summary context_compaction_started schema should include same base fields as async path (keep_recent_tokens, prior_summary_present); remove dead $resolvedModel assignment in handle(); sanitize/validate hook metadata before event persistence; update dispatcher doc to mention metadata continues after replacement; add session_id/log context in CompactionStepResultHandler logs; clarify mutable DTO class doc; remove dead test variable/ctor arg; consider cancelled flag and hook_metadata parity into async context_compacted if feasible. Reviewer confirmed no new TUI behavior, and existing TuiCompactCommandE2eTest remains valid lifecycle proof.

## Task workflow update - 2026-06-21T21:36:35.810Z
- Recorded fork run: s2v9rz8xe4dw
- Validation: fork validation: castor test --filter='CompactionHook|CompactRunHandler|CompactionStepResultHandler' PASS (25 tests, 166 assertions); fork validation: castor phpstan PASS (errors=0, file_errors=0); fork validation: castor cs-check PASS (files_fixed=0); fork validation: castor deptrac PASS (violations=0, errors=0); fork validation: castor test --filter=Compaction PASS (77 tests, 531 assertions); fork validation: castor test:tui --filter=TuiCompactCommandE2eTest PASS (2 tests, 8 assertions)
- Summary: Fix fork s2v9rz8xe4dw completed commit 9395da3d8 addressing COMP-04 review findings: safe scalar-only CompactionHookContextDTO (no raw message lists), concrete CompactionRuntimeSettingsDTO type in CompactRunHandler, replacement started-event schema parity, hook metadata sanitization before event/transport persistence, explicit session_id in CompactionStepResultHandler logs, dispatcher/result DTO docs, dead test cleanup, cancel payload cancelled=true, and async hook_metadata propagation through ExecuteCompactionStep → worker → CompactionStepResult → context_compacted.

## Task workflow update - 2026-06-21T21:43:44.441Z
- Validation: reviewer verdict: REQUEST CHANGES on HEAD 9395da3d8; reviewer ran/confirmed castor test --filter=CompactionHookDispatcherTest PASS; reviewer ran/confirmed castor test --filter='CompactRunHandlerTest|CompactionStepResultHandlerTest|ExecuteCompactionStepWorkerTest|ExecuteCompactionStepSerializerTest' PASS; reviewer ran/confirmed castor phpstan PASS, cs-check PASS, deptrac PASS
- Summary: Re-review on HEAD 9395da3d8 returned REQUEST CHANGES with one narrow remaining BUG: CompactRunHandler hook-cancel path still attaches raw $hookResult->metadata to context_compaction_failed, bypassing CompactionHookDispatcher::sanitiseMetadata(), while async and replacement paths sanitize. This can break event persistence if a cancelling hook includes object/resource/closure metadata. All prior blockers except this partial metadata sanitization gap were verified fixed; reviewer confirmed no new TUI E2E needed because no user-visible TUI changes.

## Task workflow update - 2026-06-21T21:45:38.486Z
- Recorded fork run: iaaeplddeqw8
- Validation: fork validation: castor test --filter='testHookCancelMetadataPresentInFailedPayload' PASS (1 test, 7 assertions); fork validation: castor test --filter='CompactRunHandlerTest|CompactionHookDispatcherTest|CompactionStepResultHandlerTest' PASS (25 tests, 168 assertions); fork validation: castor phpstan PASS (errors=0, file_errors=0); fork validation: castor cs-check PASS (files_fixed=0)
- Summary: Tiny fix fork iaaeplddeqw8 completed commit bd624f5f2 fixing the one remaining reviewer finding: CompactRunHandler hook-cancel path now sanitizes hook metadata before emitting context_compaction_failed, matching async and replacement paths. Extended CompactRunHandlerTest cancel-metadata test to include unsafe object/Closure values and assert they are stripped while safe scalars remain.

## Task workflow update - 2026-06-21T21:47:14.172Z
- Validation: reviewer verdict: APPROVED on HEAD bd624f5f2; reviewer confirmed no TUI files/user-visible TUI behavior changed; existing TUI E2E lifecycle proof remains sufficient
- Summary: Final reviewer subagent re-reviewed HEAD bd624f5f2 and returned APPROVED. Reviewer verified cancel-path hook metadata is now sanitized, regression test covers unsafe object/Closure stripping and safe scalar preservation, all prior blockers remain fixed (safe scalar hook context, concrete runtime settings type, replacement started schema parity, logging correlation/privacy, docs/test cleanup), and no new TUI-visible behavior was introduced.

## Task workflow update - 2026-06-21T21:49:19.962Z
- Validation: HEAD bd624f5f2; castor test PASS (3078 tests, 9873 assertions); castor deptrac PASS (violations=0, errors=0); castor phpstan PASS (errors=0, file_errors=0); castor cs-check PASS (files_fixed=0); castor test:tui PASS (13 tests, 115 assertions); castor test:llm-real --filter=CompactionLiveSmokeTest PASS (1 test, 20 assertions); worktree clean before CODE-REVIEW move
- Summary: COMP-04 ready for CODE-REVIEW: final reviewer approved HEAD bd624f5f2; local focused validation passed including full unit/integration, architecture, static analysis, formatting, deterministic TUI E2E, and focused live compaction smoke because compaction LLM invocation context changed.

## Task workflow update - 2026-06-21T21:50:41.521Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (65.7s).
- Pushed task/comp-04-compaction-hooks-and-observability to origin.
- branch 'task/comp-04-compaction-hooks-and-observability' set up to track 'origin/task/comp-04-compaction-hooks-and-observability'.
- Created PR: https://github.com/ineersa/agent-core/pull/188
- Validation: final reviewer verdict: APPROVED on HEAD bd624f5f2; castor test PASS (3078 tests, 9873 assertions); castor deptrac PASS (violations=0, errors=0); castor phpstan PASS (errors=0, file_errors=0); castor cs-check PASS (files_fixed=0); castor test:tui PASS (13 tests, 115 assertions); castor test:llm-real --filter=CompactionLiveSmokeTest PASS (1 test, 20 assertions)
- Summary: COMP-04 prepared for PR/code review at HEAD bd624f5f2. Reviewer approved after two review/fix rounds. Implemented internal before-compaction hooks, safe scalar hook context, cancel/replacement/additional-instructions/metadata hook semantics, replacement-summary LLM-skip path with lifecycle events, structured safe logging, hook metadata sanitization, and async hook_metadata propagation. No new TUI-visible behavior; existing TmuxHarness TUI proof remains valid and test:tui passed.

## Task workflow update - 2026-06-21T22:03:10.443Z
- Moved CODE-REVIEW → DONE.
- Merged task/comp-04-compaction-hooks-and-observability into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                               |   7 +
 .../Handler/ExecuteCompactionStepWorker.php        |   2 +
 .../Domain/Message/CompactionStepResult.php        |   2 +
 .../Domain/Message/ExecuteCompactionStep.php       |   2 +
 .../Application/Pipeline/CompactRunHandler.php     | 284 +++++++++--
 .../Pipeline/CompactionStepResultHandler.php       |  45 ++
 .../Compaction/BeforeCompactionHookInterface.php   |  34 ++
 .../Compaction/CompactionHookContextDTO.php        |  56 +++
 .../Compaction/CompactionHookDispatcher.php        | 165 +++++++
 .../Compaction/CompactionHookResultDTO.php         |  91 ++++
 .../Application/Pipeline/CompactRunHandlerTest.php | 527 +++++++++++++++++++--
 .../Compaction/CompactionHookDispatcherTest.php    | 300 ++++++++++++
 12 files changed, 1445 insertions(+), 70 deletions(-)
 create mode 100644 src/CodingAgent/Compaction/BeforeCompactionHookInterface.php
 create mode 100644 src/CodingAgent/Compaction/CompactionHookContextDTO.php
 create mode 100644 src/CodingAgent/Compaction/CompactionHookDispatcher.php
 create mode 100644 src/CodingAgent/Compaction/CompactionHookResultDTO.php
 create mode 100644 tests/CodingAgent/Compaction/CompactionHookDispatcherTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/comp-04-compaction-hooks-and-observability.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/comp-04-compaction-hooks-and-observability.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Before DONE move: integration checkout clean on main...origin/main; PR #188 merged by user
- Summary: PR #188 for COMP-04 was merged by the user. Moving task to DONE, merging task branch into integration checkout, syncing main, and cleaning up the worktree.
