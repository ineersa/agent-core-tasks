# Issue 137 restart: generalized model notification system

## Goal
Restart for issue #137 after PR #164 rejection and rollback PR #165. Do not reuse the rejected output-cap-specific plumbing branch except as a cautionary reference. The architectural goal is a generalized model-facing notification/nudge system usable by OutputCap, SafeGuard, extensions, and future internal guidance.

Problem statement:
- Generated model-facing messages (output cap notices, SafeGuard denials/approvals, extension nudges, internal guidance) must be represented as first-class structured data.
- The exact text sent to the model must be visible in the TUI/events log exactly as sent: no paraphrase, no summary, no hidden assistant/system messages.
- Production code must not infer notice type by parsing arbitrary text (no preg_match/str_contains/str_starts_with on notice text). Notice identity/severity/source must be structured metadata.
- OutputCap should become a producer/user of the generic notification system, not a special case threaded through RuntimeEventTranslator, ToolProjectionSubscriber, and TUI renderer.

Required planning before implementation:
1. Freeze the generic abstraction/API: name, DTO/envelope shape, source/severity/kind fields, exact text field, target audience/model visibility, relation to tool_call_id/run/turn/step.
2. Identify one canonical persistence/projection path from producer → model input → events.jsonl/runtime events → TranscriptProjector/TUI.
3. Decide where notifications are injected into model context (tool result transform hook, tool executor result envelope, or a dedicated model-context notification collector) without output-cap-specific branching.
4. Decide how producers attach notifications: OutputCap first; SafeGuard/extensions should be possible without implementation-specific hacks.
5. Define a minimal validation strategy: one generic contract/regression test plus one real TmuxHarness E2E proof for output cap. Avoid enum/DTO/mapper noise tests.

Implementation scope after plan approval:
- Add the generalized model notification primitive and flow.
- Convert OutputCap to emit/use it as the first producer.
- TUI renders generic notifications as proper notice/System blocks using structured severity/kind/source metadata.
- Preserve normal ToolResult readability; capped output should not leak raw/full output where the model saw only a notice.
- Do not implement SafeGuard-specific styling unless explicitly added later; only ensure the generic path can support it.

## Acceptance criteria
- A short architecture plan is written and approved before implementation starts; no fork should implement before the abstraction is agreed.
- Generic model-notification DTO/envelope exists with exact model-facing text and structured kind/source/severity metadata; no output-cap-specific protocol plumbing through unrelated layers.
- Production code does not parse notice text to detect output caps or notice types.
- OutputCap emits a generic model notification and uses the common flow; TUI shows exactly that notice once via a generic notice/System block.
- Normal uncapped ToolResult display remains human-readable; capped raw/full output is not shown when the model receives only a notification.
- One TmuxHarness replay-backed E2E proof demonstrates the user-visible output-cap notification behavior.
- Tests are minimal and value-based: generic contract/regression proof plus output-cap integration/E2E proof; no enum/DTO/getter/private-helper/coverage-only tests.
- SafeGuard-specific UX remains deferred unless explicitly requested, but the generic design can support SafeGuard/extension producers.

## Workflow metadata
Status: DONE
Branch: task/issue-137-model-notification-system
Worktree: /home/ineersa/projects/agent-core-worktrees/issue-137-model-notification-system
Fork run: yh3tm93zinh5
PR URL: https://github.com/ineersa/agent-core/pull/169
PR Status: merged
Started: 2026-06-18T19:35:16.597Z
Completed: 2026-06-19T01:02:58.481Z

## Work log
- Created: 2026-06-18T17:14:09.183Z

## Task workflow update - 2026-06-18T18:21:10.511Z
- Summary: Planning/scout phase completed (read-only; no implementation branch/worktree yet). Three scouts inspected: (1) model invocation/tool-message pipeline, (2) OutputCap/SafeGuard/extension producer paths, and (3) canonical runtime/TUI projection. Key current-main facts: OutputCap::process() still returns a formatted string; ReadFileTool/BashTool/BgStatusTool call it directly; ToolExecutor::toDomainResult() turns raw Symfony tool results into AgentCore ToolResult(content/details); AgentMessageNormalizer::toolMessage() JSON-encodes the ToolCallResult into the model-facing tool message; AgentMessageConverter::buildToolMessages() sends that text as the provider ToolCallMessage; ToolCallResultHandler emits ToolExecutionEnd for TUI and MessageEnd for canonical tool history; RuntimeEventTranslator has no generic model-notification event path; TUI already has System transcript blocks but no generic notification subscriber. SafeGuard block/approval currently returns ToolCallDecisionDTO::block()/requireApproval(), which becomes a synthetic tool/interruption result, with no first-class model-notification producer API.

Frozen architectural direction to review before implementation: introduce a generalized ModelNotification primitive, not output-cap-specific metadata. A ModelNotification carries exact model-facing text plus structured source/kind/severity/delivery/relation metadata. Suggested fields: id (stable/dedup), source (output_cap|safeguard|extension|system), kind (output_capped|tool_blocked|approval_requested|nudge...), severity (info|warning|error), delivery (tool_result_replace for output cap and SafeGuard-style denials; future context_message for free extension nudges), text (exact provider-facing text), relation {tool_call_id, tool_name, order_index, run_id/turn/step when available}, and metadata (cap, char_count, saved_path, policy category, extension id, etc.). No production code may infer notification type by parsing text.

Proposed data flow for the first implementation PR: producers emit ModelNotification objects; OutputCap becomes the first producer. Tool handlers should return a generic tool-result envelope (name TBD) that separates human/TUI display text from model-facing notifications. ToolExecutor converts that envelope into AgentCore ToolResult with modelNotifications. AgentMessageNormalizer renders the exact notification text into the provider-facing tool AgentMessage when delivery=tool_result_replace, instead of JSON-encoding raw/full output. ToolCallResultHandler emits a generic RunEvent model_notification for each notification (same exact text and metadata), adjacent to the tool result/message events. RuntimeEventTranslator maps it to runtime event model.notification. A new NotificationProjectionSubscriber renders it as a System transcript block using exact payload.text and structured metadata. TranscriptBlockRenderer may style System blocks generically by severity (e.g. warning => ⚠ + Warning color), never by parsing text or output_cap-specific checks. Normal uncapped ToolResult rendering remains unchanged/readable. For capped output, the ToolResult block is compact/non-raw while the System notification shows exactly what the model receives.

Central/defense-in-depth cap path decision point: OutputCapLlmTransformHook currently caps any oversized model-bound tool message. To keep exact TUI visibility for central caps without output-cap-specific plumbing, either (recommended) make the hook attach a typed ModelNotification to the transformed AgentMessage and thread generic notifications through PlatformInvocationResult/LlmStepResult so LlmStepResultHandler can emit model_notification events on success/failure/abort, or (smaller first PR) restrict this iteration to producer-originated notifications and leave central caps as emergency LLM-only behavior. This needs explicit approval because the recommended path is the only planned generic threading through LLM-step results.

Implementation phases proposed: Phase 0 plan approval. Phase 1 add ModelNotification primitive/event/projection contract and generic System rendering by severity. Phase 2 add generic tool-result envelope/carrier and bridge through ToolExecutor/AgentMessageNormalizer/ToolCallResultHandler. Phase 3 convert OutputCap/read/bash/bg_status to emit notifications instead of embedding cap text; preserve docCap/defaultCap policy and saved-output behavior. Phase 4 handle/decide central OutputCapLlmTransformHook with the same generic notification path. Phase 5 minimal validation only: one contract/regression around generic notification event→projection exact text; one OutputCap integration proof; one replay-backed TmuxHarness E2E showing exact notification appears once and raw/full capped output does not appear; focused phpstan/deptrac/cs-check; final castor check before CODE-REVIEW. Avoid enum/DTO/getter/private-helper/mapper-mirror tests.

Open questions for user approval before implementation: (1) For the first PR, should we implement only tool-scoped delivery (enough for OutputCap and SafeGuard block later) or also free-standing extension nudges/context messages now? (2) For capped ToolResult visible text, should the compact block be just '<tool> completed' or include saved-path/status text, given the System notification already shows the exact model text? (3) Should the central LLM-bound cap path be included in this first PR via generic LlmStep notification threading, or deferred? (4) Confirm source/kind/severity naming: source=output_cap, kind=output_capped, severity=warning. (5) Should notification exact text be stored both in canonical AgentMessage history (so the provider sees it on resume) and in model_notification RunEvents (so TUI/replay can project it), with the same notification id for dedup?

## Task workflow update - 2026-06-18T18:35:13.929Z
- Summary: User approved planning decisions for restart architecture: (1) first implementation PR should start with generic tool-scoped model notifications only, while naming/DTO shape must leave room for future free-standing extension nudges/context messages; (2) capped ToolResult display should remain boring/compact (e.g. '<tool> completed') and the exact model-facing cap text should appear only in the generic notification block; (3) source/kind/severity naming approved as source=output_cap, kind=output_capped, severity=warning, with TUI icons/theme colors driven from severity/structured metadata; (4) exact notification text should be stored both in canonical AgentMessage history and model_notification events, with same notification id for dedup/replay.

Central cap direction refined by user question: user wants central cap as the 'once for all tools' hub, possibly removing per-tool OutputCap calls from read/bash/bg_status. Proposed answer to bake into plan: yes, centralize OutputCap, but not solely as late OutputCapLlmTransformHook. Primary cap should move to a central tool-result/model-facing preparation layer immediately after tool execution and before canonical tool result/message projection, so raw huge output is not leaked to TUI/model history and tools do not each call OutputCap. The late LLM transform hook remains only defense-in-depth for any oversized model-bound message that bypasses the primary tool-result processor, and if it fires it must emit the same generic ModelNotification through generic LLM-step notification threading — no output-cap-specific fields or text parsing.

UI intent clarified: user likes the Pi-style visual of a tool call block showing the cap notice. Implementation should render notification visibly adjacent to / grouped with the related tool call via tool_call_id, using generic notification/System block semantics and severity styling rather than output-cap-special ToolResult styling. Normal ToolResult stays readable/compact; exact model-facing notification text is shown once.

## Task workflow update - 2026-06-18T19:35:16.597Z
- Moved TODO → IN-PROGRESS.
- Created branch task/issue-137-model-notification-system.
- Created worktree /home/ineersa/projects/agent-core-worktrees/issue-137-model-notification-system.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/issue-137-model-notification-system.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/issue-137-model-notification-system.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/issue-137-model-notification-system.
- Summary: Task-start approved by user. Frozen plan: implement a generic, tool-scoped ModelNotification system first; keep DTO/naming open for future free-standing extension nudges. OutputCap should be centralized in a shared post-tool result/model-facing preparation layer rather than per-tool calls. Late LLM transform cap remains defense-in-depth and must use the same generic notification flow if it fires. Capped ToolResult display should be compact/boring (e.g. '<tool> completed'), exact notification text appears once adjacent to the related tool call. Use source=output_cap, kind=output_capped, severity=warning, with TUI icon/theme driven by structured severity metadata. Store exact text in canonical AgentMessage history and model_notification events with shared notification id for dedup/replay.

## Task workflow update - 2026-06-18T19:39:36.098Z
- Recorded fork run: btd11b3xv4z3
- Summary: Implementation fork launched in worktree /home/ineersa/projects/agent-core-worktrees/issue-137-model-notification-system. Fork instructions freeze the approved architecture: generic tool-scoped ModelNotification system; centralized post-tool result processor for OutputCap; compact capped ToolResult; exact model-facing notification shown once via generic model.notification projection; severity-driven TUI styling; no output-cap-specific runtime/TUI plumbing; no text-parsing notice detection; minimal value-based tests plus mandatory TmuxHarness E2E proof and Castor validation.

## Task workflow update - 2026-06-18T19:59:26.004Z
- Summary: Recovered the real fork handoff from git history because fork_retrieve returned only the final glyph tweak. Actual commits on task branch: ed7b96a7a Implement generalized model-notification system with OutputCap as first producer; de7c97677 Fix PHPStan iterable hints / renderer severity helpers; 81f3d99a0 Use nicer severity glyphs. Commit body reports: ModelNotificationDTO, ToolResultProcessorInterface, OutputCapToolResultProcessor, generic model_notification RunEvent/runtime event/projection, AgentMessageNormalizer exact notification text for delivery=tool_result_replace, centralized removal of per-tool OutputCap calls from read/bash/bg_status, severity-driven System rendering, and validation claim: 2706 tests pass, deptrac 0 violations, cs-check clean. Parent inspection found this is useful progress but not acceptable as complete yet: no new/updated TmuxHarness TUI E2E proof in the diff, no focused OutputCapToolResultProcessor integration proof, capped ToolResult details still appear to retain raw_result/full output, late OutputCapLlmTransformHook attaches output_cap metadata but does not emit/thread generic ModelNotification events, and OutputCap notice wording still contains shell-centric head/grep guidance. Follow-up fork required before task can be considered implemented.

## Task workflow update - 2026-06-18T20:24:31.365Z
- Recorded fork run: ja8ywy7tfjl9
- Summary: Follow-up fork launched to complete blockers from recovered implementation: add mandatory TmuxHarness TUI E2E proof, add one focused OutputCap generic notification integration proof, sanitize capped ToolResult details so raw/full output cannot leak, convert late OutputCapLlmTransformHook to generic model_notifications and thread generic notifications to events/TUI if feasible, remove unnecessary output_cap cross-layer metadata, update cap notice guidance to tool-first wording, and run focused Castor validation.

## Task workflow update - 2026-06-18T20:34:45.663Z
- Validation: castor test: 2706 tests, 8101 assertions OK; castor test:tui: 9 tests (87 assertions) OK, includes new TuiOutputCapNoticeE2eTest; castor test:controller-replay: 1 test, 14 assertions OK; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean
- Summary: Follow-up fork completed blockers. Commit 1c1e614ff on branch task/issue-137-model-notification-system.

Fixed blockers:
1. TUI E2E proof added: tests/Tui/E2E/TuiOutputCapNoticeE2eTest.php + fixture, TmuxHarness replay-backed, proves ⚠ warning visible, "read completed" compact ToolResult, exact cap notice text, raw output absent. 4 assertions pass.
2. Integration contract proof added: tests/CodingAgent/Tool/OutputCapToolResultProcessorContractTest.php, 3 tests (30 assertions) covering capped→notification+compact+sanitized, under-cap→passthrough, empty→passthrough.
3. Raw/full output leakage fixed: OutputCapToolResultProcessor::safeDetailsFromOriginal() strips raw_result from details when capped, preserves only attachment_refs + safe metadata (mode, duration_ms, sources).
4. Late hook generic path done: OutputCapLlmTransformHook now uses model_notifications (not output_cap) for metadata+details+skip detection. hasDeliveryToolResultReplace() checks delivery type.
5. Notice wording updated: OutputCap::buildCappedNotice() now says tool-first guidance (targeted read offset/limit) instead of shell-centric head/grep.
6. Removed unnecessary cross-layer noise: deleted addCapMetadata(), removed isError skip (even error results with oversized text get capped), simplified non-capped path to pure passthrough.
7. Architecture: no text-parsing fallbacks in src/; model_notifications is the generic boundary; output_cap key remains only as producer-local audit metadata in details.

Added ModelNotificationProjectionSubscriber to TranscriptProjectorTest, new test proving model.notification→System block with exact text+severity metadata.

Known limitation: late OutputCapLlmTransformHook attaches model_notifications to AgentMessage metadata+details but LlmStepResult/LlmStepResultHandler do not thread notifications to events/TUI. Deferred as separate task — involved but not spaghetti.

Validation: castor check all green (2706 unit, 9 TUI E2E, 1 controller-replay, deptrac 0, phpstan 0, cs-check clean).

## Task workflow update - 2026-06-18T20:36:24.897Z
- Validation: fork ja8ywy7tfjl9 reported `castor test --filter="OutputCapToolResultProcessorContractTest"` OK (3 tests, 30 assertions); fork ja8ywy7tfjl9 reported `castor test --filter="OutputCapLlmTransformHookTest"` OK (8 tests, 43 assertions); fork ja8ywy7tfjl9 reported `castor test --filter="OutputCapTest"` OK (28 tests, 54 assertions); fork ja8ywy7tfjl9 reported `castor test --filter="testModelNotificationProjectsSystemBlockWithExactText"` OK (1 test, 10 assertions); fork ja8ywy7tfjl9 reported `castor test:tui --filter="TuiOutputCapNoticeE2eTest"` OK (1 test, 4 assertions); fork ja8ywy7tfjl9 reported `castor test:tui` OK (9 tests, 87 assertions); fork ja8ywy7tfjl9 reported `castor test` OK (2706 tests, 8101 assertions); fork ja8ywy7tfjl9 reported `castor deptrac` 0 violations; fork ja8ywy7tfjl9 reported `castor phpstan` 0 errors; fork ja8ywy7tfjl9 reported `castor cs-check` clean; fork ja8ywy7tfjl9 reported `castor check` all lanes OK (66.9s)
- Summary: Follow-up fork ja8ywy7tfjl9 completed at commit 1c1e614ff (branch HEAD). Parent verification: worktree clean; diff vs origin/main is 28 files changed, 1339 insertions, 103 deletions. Implemented/fixed: capped ToolResult details sanitization via OutputCapToolResultProcessor::safeDetailsFromOriginal(), generic model_notifications in OutputCapLlmTransformHook metadata/details for skip detection, tool-first OutputCap notice wording, ModelNotificationProjectionSubscriber covered in TranscriptProjectorTest, new OutputCapToolResultProcessorContractTest, new replay-backed TmuxHarness TuiOutputCapNoticeE2eTest + fixture. Full fork-reported validation: castor check all lanes green (2706 unit tests, 9 TUI E2E tests, 1 controller-replay test, deptrac 0, phpstan 0, cs-check clean). Parent review notes: primary tool-result path now looks materially aligned and has TUI proof; remaining architectural limitation is late OutputCapLlmTransformHook caps model text but does not yet thread generic model_notifications through LlmPlatformAdapter → PlatformInvocationResult → LlmStepResult → LlmStepResultHandler into model_notification events/TUI. Fork explicitly reports this as a non-trivial separate cross-layer change. Also minor review concerns before PR/reviewer: new contract test uses sys_get_temp_dir/manual cleanup instead of TestDirectoryIsolation, and TUI E2E currently asserts notification presence but not exact-once/tool-call-visible as strongly as acceptance wording. Decision needed: accept current PR scope with late-hook threading as deferred follow-up, or run another focused iteration before task-to-pr.

## Task workflow update - 2026-06-18T20:40:16.657Z
- Recorded fork run: 9tdid2tigmfk
- Summary: User requested one more iteration. Launched fork 9tdid2tigmfk to complete the remaining late-hook generic notification threading and minor cleanup before PR/reviewer: collect transform-generated model_notifications in LlmPlatformAdapter without re-emitting existing primary notifications, thread through PlatformInvocationResult → ExecuteLlmStepWorker → LlmStepResult → LlmStepResultHandler, emit generic ModelNotification RunEvents on success/error/abort, compact related ToolResult blocks generically for delivery=tool_result_replace, switch OutputCapToolResultProcessorContractTest to TestDirectoryIsolation, strengthen TuiOutputCapNoticeE2eTest for exact-once and related tool-call visibility, and run focused Castor plus full deterministic castor check if feasible.

## Task workflow update - 2026-06-18T20:57:02.786Z
- Recorded fork run: 9tdid2tigmfk
- Validation: fork 9tdid2tigmfk reported `castor test --filter="OutputCapToolResultProcessorContractTest"` OK (3 tests, 30 assertions); fork 9tdid2tigmfk reported `castor test` OK before final check (2715 tests, 8143 assertions); fork 9tdid2tigmfk reported `castor test:tui --filter="TuiOutputCapNoticeE2eTest"` OK (1 test, 7 assertions); fork 9tdid2tigmfk reported `castor deptrac` 0 violations; fork 9tdid2tigmfk reported `castor phpstan` 0 errors; fork 9tdid2tigmfk reported `castor cs-check` clean; fork 9tdid2tigmfk reported full `castor check` all 6 lanes green: deptrac OK, test OK (2711 tests, 8131 assertions), controller-replay OK (1 test, 14 assertions), test:tui OK (9 tests, 90 assertions), phpstan OK, cs-check clean
- Summary: Fork 9tdid2tigmfk completed at commit 8549f73b1. Parent verification: worktree clean at 8549f73b1; diff vs origin/main is 34 files changed, 1969 insertions, 126 deletions. Fork implemented generic late-hook model_notification threading through LlmPlatformAdapter → PlatformInvocationResult → ExecuteLlmStepWorker → LlmStepResult → LlmStepResultHandler, with ID-based dedup so pre-existing primary-path notifications are not re-emitted. LlmStepResultHandler now emits generic ModelNotification RunEvents on success, provider error, and abort/cancel paths. ModelNotificationProjectionSubscriber generically compacts related ToolResult blocks for delivery=tool_result_replace while preserving existing metadata. Minor cleanup: OutputCapToolResultProcessorContractTest now uses TestDirectoryIsolation, and TuiOutputCapNoticeE2eTest asserts related tool-call visibility plus bounded duplicate marker count. Parent grep check found only producer-local output_cap/source/kind strings and unrelated str_contains/str_starts_with usage; no output-cap notice text parsing in the relevant notification/cap implementation paths. Known limitation from fork: transform-hook notification event can arrive before a future ToolResult block exists for bypassed tools, so generic compacting may skip in that defense-in-depth path, but the exact notification System block is visible; primary OutputCapToolResultProcessor path compacts correctly because tool_execution.completed precedes model.notification.

## Task workflow update - 2026-06-18T21:00:31.015Z
- Validation: reviewer verdict: APPROVE WITH SUGGESTIONS; reviewer explicitly accepted TmuxHarness E2E adequacy; reviewer explicitly confirmed no text-parsing fallback / generic architecture compliance; reviewer judged known late-hook compacting limitation acceptable for CODE-REVIEW
- Summary: Reviewer subagent completed task-to-pr review at HEAD 8549f73b1 with verdict APPROVE WITH SUGGESTIONS and no critical/blocking issues. Reviewer confirmed: architecture boundaries are respected; generic ModelNotification flow has no output-cap-specific runtime/TUI/projection plumbing; no production text-parsing fallback is used for notification identity; TmuxHarness E2E is adequate and replay-backed; raw/full output does not leak on the primary path; late-hook compacting race is acceptable for CODE-REVIEW because exact notification System block is visible and primary path compacts correctly. Non-blocking suggestions: guard `OutputCapToolResultProcessor::safeDetailsFromOriginal()` attachment_refs access with `is_array($rawResult)` for robustness; document that safeDetailsFromOriginal drops all but whitelisted safe metadata; consider configurability/documentation for path argument keys.

## Task workflow update - 2026-06-18T21:01:30.797Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (42.7s).
- Pushed task/issue-137-model-notification-system to origin.
- branch 'task/issue-137-model-notification-system' set up to track 'origin/task/issue-137-model-notification-system'.
- Created PR: https://github.com/ineersa/agent-core/pull/169
- Validation: fork 9tdid2tigmfk reported full deterministic `castor check` all lanes green at 8549f73b1: deptrac 0, test 2711 tests OK, controller-replay 1 OK, TUI E2E 9 OK, phpstan 0, cs-check clean; reviewer subagent verdict APPROVE WITH SUGGESTIONS; no blocking findings; TmuxHarness E2E adequate; no text-parsing fallback; generic architecture compliant
- Summary: Implementation complete and reviewer-approved. HEAD 8549f73b1 implements generalized ModelNotification system with OutputCap as first producer, centralized output-cap processor, exact model-facing notification projection, generic late-hook notification threading, severity-based TUI rendering, and TmuxHarness E2E proof. Reviewer verdict: APPROVE WITH SUGGESTIONS, no blockers; non-blocking robustness/doc suggestions recorded in task metadata.

## Task workflow update - 2026-06-18T21:30:51.475Z
- Validation: architect verdict: ARCHITECTURE APPROVED; architect explicitly answered: not crossing boundaries, not reinventing wheels, abstraction generic enough for SafeGuard/extensions, late-hook compacting limitation acceptable; architect confirmed deptrac layer compliance and no text-parsing fallback in notification path
- Summary: Architect subagent completed read-only architecture review for PR #169 / HEAD 8549f73b1. Verdict: ARCHITECTURE APPROVED. No blocking architectural concerns. Architect confirmed: no boundary violations (AgentCore has zero CodingAgent/TUI deps; TUI has zero AgentCore/output_cap/ModelNotification deps), deptrac clean, ToolResultProcessorInterface is a justified new post-execution/pre-canonical-result extension point rather than reinventing existing hooks, ModelNotificationDTO is a clean generic domain primitive, severity-driven TUI rendering is generic, source/kind/delivery/metadata fields are sufficient for SafeGuard/extensions/system nudges later, primary path lifecycle and late-hook dedup/threading are sound, and the known late-hook compacting limitation is acceptable for CODE-REVIEW because primary path compacts correctly and exact System notification remains visible. Non-blocking architecture suggestions: document/adjust safeDetailsFromOriginal metadata whitelist, document PATH_ARGUMENT_KEYS contract for tool authors, make attachment_refs guard more robust, optionally consider future named constructors for ModelNotificationDTO readability.

## Task workflow update - 2026-06-18T21:36:25.207Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User added PR review comments and asked to address viable reviewer/architect suggestions. PR comments to address: BashTool lines 147 and 365 have leftover `$capped = ...` variables no longer needed after central OutputCap processor; OutputCap line 89 should prefer Symfony String for Unicode-aware length handling. Viable non-blocking suggestions to include: robust `attachment_refs` guard in OutputCapToolResultProcessor::safeDetailsFromOriginal(); document safe metadata whitelist; document PATH_ARGUMENT_KEYS contract for tool authors. Task moved back to IN-PROGRESS for review iteration.

## Task workflow update - 2026-06-18T21:37:11.685Z
- Recorded fork run: 5391uxrxdsu8
- Summary: Launched review-iteration fork 5391uxrxdsu8 for PR #169 comments and viable reviewer/architect suggestions. Scope: remove redundant `$capped` variables in BashTool timeout/handleFinished paths and update stale docblock; switch OutputCap character counting to Symfony String consistently, including OutputCapToolResultProcessor cap-length comparison; robustly guard attachment_refs preservation in safeDetailsFromOriginal(); document safe metadata whitelist and PATH_ARGUMENT_KEYS contract; optionally clean adjacent duplicate OutputCap PHPDocs. Validation requested: focused Castor tests for OutputCap/OutputCapToolResultProcessor/BashTool, phpstan, cs-check, and TUI E2E filter if behavior changes.

## Task workflow update - 2026-06-18T21:41:43.957Z
- Recorded fork run: 5391uxrxdsu8
- Validation: fork read testing skill and tests/AGENTS.md per handoff; test thesis: existing focused contract/unit tests should remain green because changes are implementation-internal cleanup/refactors with no new behavior to prove; fork reported `castor test --filter="OutputCapTest|OutputCapToolResultProcessorContractTest|BashToolTest"` OK (52 tests, 159 assertions); fork reported `castor phpstan` 0 errors; fork reported `castor cs-fix` then `castor cs-check` clean; fork reported `castor deptrac` 0 violations; parent ran `git diff --check HEAD~1..HEAD` OK
- Summary: Review-iteration fork 5391uxrxdsu8 completed at commit 1a3da97c5. Parent verification: worktree clean; 3 files changed (+50/-36): BashTool.php, OutputCap.php, OutputCapToolResultProcessor.php. Addressed PR comments by removing redundant `$capped` intermediates in BashTool timeout and handleFinished paths, using `$partialOutput`/`$output` directly, and updating stale handleFinished return docblock. Switched OutputCap character-counting to Symfony String `u()->length()` in process(), capIfNeeded(), and buildCappedNotice(), and aligned OutputCapToolResultProcessor cap comparison to the same approach. Addressed viable reviewer/architect suggestions by robustly guarding attachment_refs preservation with array checks, documenting safeDetailsFromOriginal() whitelist behavior and ToolExecutor::withExecutionMetadata() preservation, documenting PATH_ARGUMENT_KEYS contract, and cleaning duplicate/orphaned OutputCap PHPDoc. No behavior/API/runtime/TUI changes beyond Unicode-aware equivalent length helper and comments/refactors.

## Task workflow update - 2026-06-18T21:42:44.626Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (42.5s).
- Pushed task/issue-137-model-notification-system to origin.
- branch 'task/issue-137-model-notification-system' set up to track 'origin/task/issue-137-model-notification-system'.
- PR already exists: https://github.com/ineersa/agent-core/pull/169
- Validation: fork reported focused `castor test --filter="OutputCapTest|OutputCapToolResultProcessorContractTest|BashToolTest"` OK (52 tests, 159 assertions); fork reported `castor phpstan` 0 errors; fork reported `castor cs-check` clean after cs-fix; fork reported `castor deptrac` 0 violations; parent verified clean worktree and `git diff --check HEAD~1..HEAD` OK
- Summary: PR review iteration complete. Commit 1a3da97c5 addresses the user's PR comments and viable reviewer/architect suggestions: redundant BashTool `$capped` variables removed, OutputCap length calculations switched to Symfony String `u()->length()`, OutputCapToolResultProcessor length check aligned, attachment_refs guard made robust, safe metadata whitelist and PATH_ARGUMENT_KEYS contract documented, and stale OutputCap PHPDocs cleaned. No new behavior/API/runtime/TUI changes beyond implementation cleanup.

## Task workflow update - 2026-06-18T21:46:50.441Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User smoke-tested PR #169 in worktree session 1 and found blocker: bash/read output-cap path double-caps and emits repeated notices. Evidence from `.hatfield/sessions/1/events.jsonl`: bash `cat -n .pi/plans/toolbox-design-plan.md` produced `tool_execution_end` result ~19,962 chars (tail output, not compact), then AgentMessageNormalizer serialized content+raw_result JSON to 41,570 chars, so OutputCapLlmTransformHook late-capped it and emitted model_notification seq 39; because transformed messages are not persisted, later LLM steps re-capped the same original tool message and emitted new notifications seq 49/60 with new saved paths. Reading the saved output file with `read limit=100` produced visible text ~41,577 chars (under .txt doc cap) but model-facing JSON duplicated to 86,250 chars and late hook capped at default 20k because it lacks path context, producing second/double notice. This is a CODE-REVIEW blocker: built-in tools must cap in the primary OutputCapToolResultProcessor path, normal tool messages must not be JSON-bloated into late caps, path-specific caps must apply at late hook when needed, and TUI must show one exact notification for a cap, not repeated notices.

## Task workflow update - 2026-06-18T21:47:54.501Z
- Recorded fork run: xd0ycbvoeedi
- Summary: Launched urgent blocker fork xd0ycbvoeedi to fix user smoke failure from worktree session 1: bash/read output cap double-capping and repeated notices. Fork brief includes parent investigation from `.hatfield/sessions/1/events.jsonl`: foreground BashTool returned only a ~20k log tail so primary OutputCapToolResultProcessor did not cap; AgentMessageNormalizer JSON-encoded content + details.raw_result producing 41,570-char model-facing JSON; late hook capped it and re-emitted on later LLM steps because late transform is not persisted and notification id uses random saved path; reading the saved output cap file produced 41,577 visible chars but 86,250-char model JSON and got late-capped at default cap because late hook lacks path context. Requested fixes: make foreground BashTool expose full command output to primary processor; make normal model-facing tool messages plain tool result/error text rather than JSON raw_result wrapper; persist/extract path arguments for path-aware late hook; consider stable late-hook notification IDs; add focused tests catching the smoke regression; run Castor validation/full check if feasible.

## Task workflow update - 2026-06-18T21:54:41.225Z
- Recorded fork run: tet0sxm1ch63
- Summary: Parent review of fork xd0ycbvoeedi/commit 75d272581 found two remaining blockers, so launched follow-up fork tet0sxm1ch63. Blocker 1: OutputCapLlmTransformHook claims stable late-hook notification IDs but hashes $capResult->noticeText, which includes random saved_path, so repeated late transforms of the same oversized message can still create new notification IDs/TUI blocks. Blocker 2: AgentMessageNormalizer still has a canonical JSON envelope fallback for empty content, reintroducing raw_result duplication into model-facing text. Fork instructed to fix both source issues and add minimal tests that would fail on current code: repeated late hook transform stable id from original content hash, and empty-content ToolCallResult with large details.raw_result not exposing sentinel/JSON envelope as model-facing text.

## Task workflow update - 2026-06-18T22:00:05.935Z
- Recorded fork run: tet0sxm1ch63
- Validation: castor check — OK (67.7s): deptrac OK, test OK (2713 tests, 8141 assertions), test:controller-replay OK (1 test, 14 assertions), test:tui OK (9 tests, 90 assertions), phpstan OK (0 errors), cs-check OK
- Summary: Follow-up fork tet0sxm1ch63 completed as commit 09495e86a. Parent-identified blockers fixed: late-hook output-cap notification ID now hashes stable original combined text (not noticeText/savedPath), and AgentMessageNormalizer no longer falls back to JSON-encoding full ToolCallResult/details.raw_result for empty content; it emits compact '<tool> completed/failed' labels while preserving raw_result in AgentMessage details. Added two focused regression tests in OutputCapLlmTransformHookTest that fail on the old behavior: repeated late-hook transforms produce same notification ID despite different saved paths, and empty-content ToolCallResult with large details.raw_result does not leak sentinel into model-facing text. Parent inspected key lines and ran full deterministic Castor check successfully.

## Task workflow update - 2026-06-18T22:04:44.658Z
- Validation: Reviewer subagent — APPROVE, no blocking findings. Parent notes castor check already passed after commit 09495e86a.
- Summary: Reviewer subagent reviewed branch at HEAD 09495e86a and returned APPROVE with no blocking issues. Reviewer verified the architecture is generic ModelNotification (output cap as first producer), no output-cap-specific runtime/TUI/projection plumbing, no production text parsing fallback for output-cap identity, smoke-blocker fixes are real (foreground BashTool full-log primary cap, normalizer plain tool text/compact label, path-aware late hook, stable notification IDs), and TUI proof uses real TmuxHarness. Non-blocking suggestions: add ModelNotification to an existing RuntimeEventTypeTest family provider (not pursued now because tests are already green and this resembles enum/list maintenance), consider aligning primary processor notification IDs to content-derived IDs, consider removing legacy OutputCap::process() if no external compatibility need, remove unused metadata copy in late hook if desired, and monitor full-log memory tradeoff.

## Task workflow update - 2026-06-18T22:05:36.160Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (41.1s).
- Pushed task/issue-137-model-notification-system to origin.
- branch 'task/issue-137-model-notification-system' set up to track 'origin/task/issue-137-model-notification-system'.
- Skipped PR creation (pushOnly: true).
- Validation: Parent: castor check — OK (67.7s): deptrac OK, test OK (2713 tests, 8141 assertions), test:controller-replay OK (1 test, 14 assertions), test:tui OK (9 tests, 90 assertions), phpstan OK (0 errors), cs-check OK; Reviewer subagent — APPROVE, no blocking findings; move_task CODE-REVIEW deterministic castor check gate should run before push
- Summary: Review iteration complete at commit 09495e86a. Fixed smoke-blocker follow-ups: late-hook notification ID now uses stable original content hash instead of noticeText/savedPath, and AgentMessageNormalizer no longer JSON-encodes empty-content ToolCallResult/details.raw_result into model-facing text. Reviewer approved with no blocking findings. Branch pushed for existing PR #169; pushOnly used because gh auth is unreliable in this environment.

## Task workflow update - 2026-06-18T22:06:33.070Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/169
- Updated PR Status: open
- Summary: Verified GitHub CLI auth is currently healthy and PR #169 is open at HEAD 09495e86a on branch task/issue-137-model-notification-system. Earlier pushOnly was conservative based on previous gh auth failures, but gh auth now works.

## Task workflow update - 2026-06-18T22:20:39.982Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User smoke passed PR #169 and requested a final wording polish before merge: shorten output-cap model-facing notice, avoid phrase 'Use targeted follow-up only', use read offset+limit examples with limit=200, and update read tool guidance/continuation hint to emphasize offset+limit together. This is a small review iteration scoped to wording/guidance only; no architecture changes.

## Task workflow update - 2026-06-18T22:21:02.980Z
- Recorded fork run: ae5kwlad72qu
- Summary: Launched wording-only review-iteration fork ae5kwlad72qu. Scope: shorten OutputCap::buildCappedNotice() model-facing notice with concrete examples, avoid phrase 'Use targeted follow-up only', use read(path, offset=1, limit=200) and grep example, and update ReadFileTool prompt guidelines/continuation hint to pair offset+limit (limit=200) instead of offset alone. No architecture changes.

## Task workflow update - 2026-06-18T22:26:32.598Z
- Recorded fork run: ae5kwlad72qu
- Validation: Fork reported: castor check OK (68.3s), focused OutputCap/ReadFile/TranscriptProjector/OutputCapLlmTransformHook tests OK, TuiOutputCapNoticeE2eTest OK, deptrac/phpstan/cs-check OK.
- Summary: Wording-only fork ae5kwlad72qu completed as commit 5dfa418a8. Changes: OutputCap::buildCappedNotice() now emits a shorter actionable notice with `[Output capped: ...]`, `Saved full output: ...`, `Next: use a focused follow-up, e.g.`, read example using `offset: 1, limit: 200`, grep example, and final warning not to rerun or read saved file without offset+limit. ReadFileTool prompt guidelines and continuation hint now pair offset+limit together (`offset=N limit=200`) instead of offset alone. Parent inspected changed lines; no architecture changes.

## Task workflow update - 2026-06-18T22:27:26.664Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (45.6s).
- Pushed task/issue-137-model-notification-system to origin.
- branch 'task/issue-137-model-notification-system' set up to track 'origin/task/issue-137-model-notification-system'.
- Skipped PR creation (pushOnly: true).
- Validation: Fork: castor check OK (68.3s), TuiOutputCapNoticeE2eTest OK, focused tests OK, deptrac/phpstan/cs-check OK; move_task CODE-REVIEW deterministic castor check gate runs before push
- Summary: Wording-only review iteration complete at commit 5dfa418a8. User-requested output-cap notice is shorter/actionable, avoids 'Use targeted follow-up only', uses read offset+limit with limit=200 and grep example, and ReadFileTool guidance/continuation hint now pairs offset+limit. Existing PR #169 remains open; branch pushed to update it.

## Task workflow update - 2026-06-19T00:32:32.054Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User smoke found blocker: capped read output saves the rendered cat -n text, and the output-cap notice tells the model to `read(path: saved-output, offset: 1, limit: 200)`, causing double line numbering when inspecting the saved cap artifact. Need review iteration: keep generic ModelNotification architecture, but fix follow-up guidance/source semantics so read-tool caps point follow-up reads at the original file (offset+limit=200) or inspect saved artifacts with shell head/grep, not read(saved cap file).

## Task workflow update - 2026-06-19T00:33:39.322Z
- Recorded fork run: ebcxmm1u0qkn
- Summary: Launched review-iteration fork ebcxmm1u0qkn for user-smoke bug: capped read output saves rendered cat-n text, and notice currently suggests read(saved output), causing double line numbers. Fork scope: keep generic ModelNotification architecture; do not move cap back into per-tool calls; make output-cap notice source-aware so capped read output suggests read(original path, offset/original offset, limit=200) and grep original path, while generic caps inspect saved rendered artifact with bash head/grep instead of read(savedPath). Add focused regression and TUI fixture/assertion updates as needed.

## Task workflow update - 2026-06-19T00:42:40.078Z
- Recorded fork run: ebcxmm1u0qkn
- Validation: Fork ebcxmm1u0qkn reported full castor check green: deptrac, test (2715 tests / 8163 assertions), controller-replay, TUI, phpstan, cs-check all OK.
- Summary: Fork ebcxmm1u0qkn completed as commit 3d32e896a, fixing user-smoke double line-number bug. Primary and late output-cap paths now build context-aware notices: capped read output suggests follow-up `read(original path, offset, limit=200)` and grep original path; generic/non-read caps suggest shell `head -200`/`grep` on saved rendered artifact. Generic ModelNotification architecture unchanged. Parent inspected changes and found two small follow-up issues before CODE-REVIEW: generic savedPath shell examples need escapeshellarg, read-specific fallback must not ever degrade to read(saved output) when original path missing, and read-specific final line should avoid the slightly contradictory phrase `Do not rerun the original command` since it asks for a smaller read follow-up. Launched tiny follow-up fork jj5hme4x1t6f for those polish fixes.

## Task workflow update - 2026-06-19T00:45:40.963Z
- Recorded fork run: jj5hme4x1t6f
- Validation: Fork: castor test --filter="OutputCapTest|OutputCapToolResultProcessorContractTest|OutputCapLlmTransformHookTest" OK (43 tests, 159 assertions); castor test:tui --filter=TuiOutputCapNoticeE2eTest OK (1 test, 7 assertions); castor phpstan 0 errors; castor cs-check clean; castor deptrac 0 violations.
- Summary: Tiny follow-up fork jj5hme4x1t6f completed as commit 8663dfde5. Fixed all parent-identified polish issues after the double-line-number fix: generic OutputCap notice now shell-escapes savedPath with escapeshellarg() in head/grep examples; read-specific contextual notices in both primary OutputCapToolResultProcessor and late OutputCapLlmTransformHook no longer fall back to read(saved cap artifact) when original path is missing, instead returning generic head/grep notice; read-specific final line now says 'Do not repeat the original full read or read the saved output with read.' Parent inspected diff; behavior is correct. Note: one comment typo ('outcapac artifact') remains in OutputCapLlmTransformHook PHPDoc but is non-functional and not worth another implementation round unless user wants polish.

## Task workflow update - 2026-06-19T00:46:29.483Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (41.9s).
- Pushed task/issue-137-model-notification-system to origin.
- branch 'task/issue-137-model-notification-system' set up to track 'origin/task/issue-137-model-notification-system'.
- PR already exists: https://github.com/ineersa/agent-core/pull/169
- Validation: Fork validation: focused output-cap/read-notice tests OK, TUI E2E output-cap test OK, phpstan 0, cs-check clean, deptrac 0; move_task CODE-REVIEW deterministic castor check gate runs before push/PR update
- Summary: Double-line-number smoke bug review iteration complete at commit 8663dfde5. Capped read output now points follow-up `read(path, offset, limit=200)` at the original read path; generic/non-read caps inspect saved rendered output via shell head/grep; no notice path tells models to read the saved cap artifact with the read tool. Generic ModelNotification architecture unchanged. Updating PR #169.

## Task workflow update - 2026-06-19T00:51:59.297Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User requested one more wording iteration after smoke: for generic/non-read output-cap notices, suggest `read(path: saved-output, offset: 1, limit: 200)` plus grep instead of `head -200`; read-tool caps must continue to use original file path and must not suggest read(saved cap artifact). Scope is notice wording only, no architecture changes.

## Task workflow update - 2026-06-19T00:52:19.877Z
- Recorded fork run: 53swufe5szo6
- Summary: Launched tiny wording fork 53swufe5szo6. Goal: generic/non-read output-cap notice should suggest `read(path: saved-output, offset: 1, limit: 200)` plus grep instead of `head -200`, with final warning not to read saved output without offset+limit. Read-tool caps must keep current original-path guidance and never suggest read(saved cap artifact). No architecture changes.

## Task workflow update - 2026-06-19T00:57:57.314Z
- Recorded fork run: 53swufe5szo6
- Validation: Fork 53swufe5szo6: castor test --filter="OutputCapTest|OutputCapToolResultProcessorContractTest|OutputCapLlmTransformHookTest" OK (43 tests, 163 assertions); castor test:tui --filter=TuiOutputCapNoticeE2eTest OK; castor phpstan 0 errors; castor cs-check clean; castor deptrac 0 violations.
- Summary: Fork 53swufe5szo6 completed as commit 57cc98f44: generic/non-read OutputCap notice now suggests `read(path: saved output, offset: 1, limit: 200)` plus grep instead of head. Read-specific contextual notices unchanged. Parent inspection found one missed instruction before CODE-REVIEW: the generic read example embeds raw savedPath inside double quotes; it should be JSON-style escaped in the read tool argument. Launched tiny correction fork yh3tm93zinh5 for that escaping-only fix.

## Task workflow update - 2026-06-19T01:01:04.267Z
- Recorded fork run: yh3tm93zinh5
- Validation: Fork: castor test --filter="OutputCapTest|OutputCapToolResultProcessorContractTest|OutputCapLlmTransformHookTest" OK (43 tests, 164 assertions); castor test:tui --filter=TuiOutputCapNoticeE2eTest OK (1 test, 7 assertions); castor phpstan 0 errors; castor cs-check clean; castor deptrac 0 violations.
- Summary: Fork yh3tm93zinh5 completed as commit 65eb24863. Parent inspected diff: generic OutputCap notice now uses JSON-encoded savedPath in the read-tool example (`read(path: <json string>, offset: 1, limit: 200)`) while grep continues to use escapeshellarg(); this fixes the missed raw-quote formatting issue from 57cc98f44. Only OutputCap.php and OutputCapTest.php changed. No architecture/TUI/runtime/projection changes.

## Task workflow update - 2026-06-19T01:01:52.587Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (42.2s).
- Pushed task/issue-137-model-notification-system to origin.
- branch 'task/issue-137-model-notification-system' set up to track 'origin/task/issue-137-model-notification-system'.
- PR already exists: https://github.com/ineersa/agent-core/pull/169
- Validation: Focused validation from forks: output-cap tests OK, TUI output-cap E2E OK, phpstan 0, cs-check clean, deptrac 0; move_task CODE-REVIEW deterministic castor check gate runs before push/PR update
- Summary: Generic/non-read output-cap notice wording iteration complete at commit 65eb24863. Generic caps now suggest `read(path: <saved-output>, offset: 1, limit: 200)` plus grep; the read-tool-specific path remains original-file focused and still never suggests reading the saved cap artifact with read. SavedPath is JSON-encoded in read example and shell-escaped in grep example. Updating PR #169.

## Task workflow update - 2026-06-19T01:02:58.481Z
- Moved CODE-REVIEW → DONE.
- Merged task/issue-137-model-notification-system into integration checkout.
- Auto-merging config/services.yaml
Auto-merging src/CodingAgent/Runtime/Protocol/RuntimeEventTranslator.php
Merge made by the 'ort' strategy.
 config/services.yaml                               |   5 +-
 .../Application/Handler/ExecuteLlmStepWorker.php   |   2 +
 .../Application/Handler/ExecuteToolCallWorker.php  |   2 +
 src/AgentCore/Application/Handler/ToolExecutor.php |  27 +-
 .../Application/Pipeline/LlmStepResultHandler.php  |  78 +++++-
 .../Application/Pipeline/ToolCallResultHandler.php |  41 +++
 .../Contract/Tool/ToolResultProcessorInterface.php |  37 +++
 src/AgentCore/Domain/Event/RunEventTypeEnum.php    |   2 +-
 .../Domain/Message/AgentMessageNormalizer.php      |  95 ++++++-
 src/AgentCore/Domain/Message/LlmStepResult.php     |   8 +-
 .../Domain/Model/PlatformInvocationResult.php      |   9 +-
 .../Domain/Notification/ModelNotificationDTO.php   |  84 ++++++
 .../SymfonyAi/LlmPlatformAdapter.php               |  95 ++++++-
 .../ModelNotificationProjectionSubscriber.php      | 144 ++++++++++
 .../Runtime/Protocol/RuntimeEventTranslator.php    |  18 ++
 .../Runtime/Protocol/RuntimeEventTypeEnum.php      |   7 +-
 .../Tool/BackgroundProcess/ProcessLifecycle.php    |  38 +++
 src/CodingAgent/Tool/BackgroundProcessManager.php  |  30 +++
 src/CodingAgent/Tool/BashTool.php                  |  22 +-
 src/CodingAgent/Tool/BgStatusTool.php              |   7 +-
 src/CodingAgent/Tool/OutputCap.php                 |  95 +++++--
 src/CodingAgent/Tool/OutputCapLlmTransformHook.php | 230 ++++++++++++++--
 src/CodingAgent/Tool/OutputCapResult.php           |  29 ++
 .../Tool/OutputCapToolResultProcessor.php          | 284 +++++++++++++++++++
 src/CodingAgent/Tool/ReadFileTool.php              |  13 +-
 src/CodingAgent/Tool/ToolHandlerInterface.php      |   5 +-
 src/Tui/Transcript/TranscriptBlockRenderer.php     |  43 ++-
 .../SymfonyAi/PlatformIntegrationTest.php          | 300 +++++++++++++++++++++
 .../Runtime/Projection/TranscriptProjectorTest.php | 126 ++++++++-
 tests/CodingAgent/Runtime/RuntimeEventTypeTest.php |   3 +
 tests/CodingAgent/Tool/BashToolTest.php            |   8 +-
 tests/CodingAgent/Tool/BgStatusToolTest.php        |  29 +-
 .../Tool/OutputCapLlmTransformHookTest.php         | 174 ++++++++++++
 tests/CodingAgent/Tool/OutputCapTest.php           |  17 +-
 .../OutputCapToolResultProcessorContractTest.php   | 240 +++++++++++++++++
 tests/CodingAgent/Tool/ReadFileToolTest.php        |  22 +-
 tests/Tui/E2E/TuiOutputCapNoticeE2eTest.php        | 287 ++++++++++++++++++++
 tests/Tui/E2E/fixtures/tui-output-cap-read.json    |  37 +++
 38 files changed, 2541 insertions(+), 152 deletions(-)
 create mode 100644 src/AgentCore/Contract/Tool/ToolResultProcessorInterface.php
 create mode 100644 src/AgentCore/Domain/Notification/ModelNotificationDTO.php
 create mode 100644 src/CodingAgent/Runtime/ProjectionPipeline/ModelNotificationProjectionSubscriber.php
 create mode 100644 src/CodingAgent/Tool/OutputCapResult.php
 create mode 100644 src/CodingAgent/Tool/OutputCapToolResultProcessor.php
 create mode 100644 tests/CodingAgent/Tool/OutputCapToolResultProcessorContractTest.php
 create mode 100644 tests/Tui/E2E/TuiOutputCapNoticeE2eTest.php
 create mode 100644 tests/Tui/E2E/fixtures/tui-output-cap-read.json
- Removed worktree /home/ineersa/projects/agent-core-worktrees/issue-137-model-notification-system.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/issue-137-model-notification-system.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: User confirmed PR #169 was merged. Completing task and syncing integration checkout.

## Task workflow update - 2026-06-19T01:03:48.269Z
- Validation: DONE merge completed successfully; move_task merged task branch into integration checkout, removed worktree, removed IDEA exclusions, and pulled origin/main. No additional post-merge castor check run because a root-owned stale `php bin/console messenger:consume --all --exclude-receivers=failed` process (PID 3414) is present and cannot be killed by this user (`Operation not permitted`). CODE-REVIEW gate previously passed deterministic castor check before merge.
- Summary: Task completed after user confirmed PR #169 was merged.
