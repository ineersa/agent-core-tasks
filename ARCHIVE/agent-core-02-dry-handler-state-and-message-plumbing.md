# agent-core-02: DRY handler state and message plumbing

## Goal
## Goal
Remove repeated handler callback, notification, message-text, and state-copy plumbing after agent-core-01 establishes safe `RunState` mutation.

## Full report
[`/home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md`](file:///home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md) — AgentCore candidates 2–5.

## Architecture report evidence

### Post-commit callbacks
The same `hrtime` step ID, SHA-256 idempotency key, `AdvanceRun` dispatch, and Messenger exception wrapping appears in:
- `LlmStepResultHandler::followUpAdvanceCallback()`,
- `ApplyCommandHandler`,
- `ToolCallResultHandler`,
- `StartRunHandler::initialAdvanceCallback()`,
- `AdvanceRunHandler` post-cancel closure.
`turnCompletedCallbacks` is also duplicated in `LlmStepResultHandler` and `ToolCallResultHandler`. Key formats intentionally vary by flow and must remain explicit inputs rather than be normalized accidentally.

### Model notifications
- Notification denormalization is duplicated in `ToolCallResultHandler` and `LlmPlatformAdapter`.
- Notification DTO → `ModelNotification` event-spec collection is byte-identical in `ToolCallResultHandler` and `LlmStepResultHandler`.

### Message text
Four sites extract text from content parts:
- `AgentMessageNormalizer`,
- `ToolCallResultHandler`,
- `AgentMessageConverter`,
- `CommandMailboxPolicy`.
Three join parts with `"\n"`; `CommandMailboxPolicy` joins with `''`. A canonical `AgentMessage::textContent()` or equally direct domain owner can remove drift, but the join difference is externally observable.

### State copying
`CommandMailboxPolicy::copyState()` privately reimplements `RunState::with()` and is vulnerable to the same future-field drift.

## Required decision before implementation
Inventory the four text-extraction callers and tests. During task-explain, obtain explicit user approval for the canonical multi-part join semantics if changing `CommandMailboxPolicy` from `''` to `"\n"` affects transcript, mailbox, prompt, or model-visible text. Do not silently choose. If semantics intentionally differ, retain a named explicit option or separate operation rather than forcing false DRY.

## Smallest viable direction
- One narrow callback creator owns shared dispatch/error mechanics while each caller supplies its existing key/step context.
- One notification normalizer/event-spec owner handles the two duplicated boundaries without adding another Serializer stack.
- Put canonical content-part text extraction on the existing domain object only if all callers truly share semantics.
- Delete `CommandMailboxPolicy::copyState()` and use `RunState::with()`.
- Prefer static/private existing owners over factories/interfaces with one implementation.

## Scope boundaries
- Preserve idempotency keys, step IDs, callback order, metrics, dispatch timing, Messenger failure propagation, notification payloads/order, and state transitions.
- No new event, command, setting, public API, Serializer stack, normalizer hierarchy, or compatibility path.
- Do not mix orchestration/storage scaffolding from agent-core-04.

## Test thesis
Test callback key/dispatch/failure behavior once at the shared boundary, retain handler integration assertions, and cover one representative multi-part text case after the semantic decision. Do not mirror each helper implementation in every handler test.

## Acceptance criteria
- Repeated follow-up/initial `AdvanceRun` callback mechanics and turn-completed callbacks have one narrow owner while preserving every existing key format and callback order.
- Model-notification denormalization and event-spec construction each have one owner with unchanged DTO/event payloads and ordering.
- `CommandMailboxPolicy::copyState()` is deleted in favor of `RunState::with()`.
- Content-part text extraction is unified only after the `''` versus `"\n"` behavior decision; intentional differing semantics remain explicit rather than hidden.
- No generic callback framework, builder, new Serializer/normalizer stack, public API, or compatibility shim is introduced.
- Focused behavior tests cover the shared mechanics and approved text semantics without broad test refactoring.
- The fork and handoff state that `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` were read and followed.
- Focused Castor tests, `castor test`, `castor test:controller-replay`, `castor deptrac`, `castor phpstan`, `castor cs-check`, and final `castor check` pass.

## Workflow metadata
Status: ARCHIVE
Branch: task/agent-core-02-dry-handler-state-and-message-plumbing
Worktree: /home/ineersa/projects/agent-core-worktrees/agent-core-02-dry-handler-state-and-message-plumbing
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/392
PR Status: merged
Started: 2026-08-16T00:25:13.887Z
Completed: 2026-08-16T02:57:17.977Z

## Work log
- Created: 2026-08-15T23:28:18.389Z

## Task workflow update - 2026-08-15T23:59:10.711Z
- Dependency update: `agent-core-01` commit f9ef99882 already deletes `CommandMailboxPolicy::copyState()` and replaces it with `RunState::with()` as part of closing the complete state-copy bug class. When this task starts, exclude candidate 5 and retain only callbacks, notifications, and explicitly decided text extraction work.

## Task workflow update - 2026-08-16T00:25:13.887Z
- Moved TODO → IN-PROGRESS.
- Created branch task/agent-core-02-dry-handler-state-and-message-plumbing.
- Created worktree /home/ineersa/projects/agent-core-worktrees/agent-core-02-dry-handler-state-and-message-plumbing.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/agent-core-02-dry-handler-state-and-message-plumbing.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/agent-core-02-dry-handler-state-and-message-plumbing.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/agent-core-02-dry-handler-state-and-message-plumbing.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/agent-core-02-dry-handler-state-and-message-plumbing.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/agent-core-02-dry-handler-state-and-message-plumbing/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/agent-core-02-dry-handler-state-and-message-plumbing.
- Summary: Starting agent-core-02 after agent-core-01 merged. Candidate 5/state-copy work is already complete and excluded. Implementation is gated on scouting the externally observable content-part join difference and obtaining the user's explicit semantics decision.

## Task workflow update - 2026-08-16T00:30:48.254Z
- Summary: Scout completed. Callback and model-notification duplication still exists and has clear behavior-preserving consolidation paths. Candidate 5 is already complete. Canonical AgentMessage::textContent() is a KEEP because three paths consume raw arrays; forcing one abstraction would add machinery. Implementation is blocked only on the externally observable multipart mailbox separator decision.
- Scout read and followed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`; no edits or QA run.
- Multipart mailbox text currently joins text parts with an empty string (`a` + `b` → `ab`), while normalizer, tool result transcript, and provider conversion use newline (`a\nb`). Mailbox text flows to canonical agent_command_applied payloads, RuntimeEventTranslator, HistoryProjector, and transcript-facing user text; command routing and current model prompt are unaffected.
- Recommended implementation boundary: do not add AgentMessage::textContent() or a generic mixed-array text abstraction. Either preserve the mailbox-specific join explicitly or change only its separator after user approval.

## Task workflow update - 2026-08-16T02:20:46.373Z
- Summary: User explicitly approved changing multipart mailbox text joining from empty-string concatenation to newline joining for readable consistency. This permits the externally observable behavior change in canonical agent_command_applied payloads and runtime/history/transcript text; command routing and model prompt conversion remain unchanged.
- Finalized semantics: text parts `a` and `b` in mailbox-applied messages must render as `a\nb`, matching the other extraction paths.
- Minimality decision: align the separator and add one behavior proof, but KEEP the raw-array extractors separate because they consume different boundary types/filter rules; do not add a generic mixed-array extractor or AgentMessage API that would not replace those paths.

## Task workflow update - 2026-08-16T02:37:29.688Z
- Validation: Fork confirmed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` were read and followed.; Focused callback/notification tests — PASS (7 tests, 34 assertions).; CommandMailboxPolicyTest — PASS (9 tests, 52 assertions).; Focused handler/metrics tests — PASS (71 tests, 554 assertions).; Focused integration/replay-adjacent tests — PASS (49 tests, 318 assertions).; castor test — PASS (4479 tests, 17558 assertions).; castor test:controller-replay — PASS (12 tests, 165 assertions).; castor deptrac — PASS (0 violations, 0 errors).; castor phpstan — PASS (0 errors).; castor cs-check — PASS (0 files fixed).; git diff --check — PASS; worktree clean.; castor check intentionally not run during task-start.
- Summary: Implementation committed as e11c86680. Six AdvanceRun callback blocks now share one internal factory; duplicate turn-completed callback mechanics live on RunMetrics; notification denormalization and event-spec construction share ModelNotificationCodec; approved mailbox multipart text now joins with newline. Candidate 5 and generic AgentMessage text abstraction remain excluded. Worktree clean; 13 files, production net -44 lines, tests +246 lines. IDE diagnostics report no errors in both new owners and the two primary handlers. Stopping at task-start; not pushed and no PR.

## Task workflow update - 2026-08-16T02:50:25.893Z
- Validation: Reviewer read and followed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`.; Reviewer verdict: APPROVED.; castor test — PASS (4479 tests, 17558 assertions).; castor deptrac — PASS (0 violations, 0 errors).; castor phpstan — PASS (0 errors).; castor cs-check — PASS (0 files fixed).; Earlier implementation validation: castor test:controller-replay — PASS (12 tests, 165 assertions).
- Summary: task-to-pr review complete. Reviewer verdict: APPROVED with no blocking findings. Specification fidelity passes: the only externally observable change is the user-approved multipart mailbox newline join; callback and notification changes preserve existing behavior. Final commit e11c86680; 13 files, production net -44 lines.

## Task workflow update - 2026-08-16T02:52:37.668Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (120.8s).
- Pushed task/agent-core-02-dry-handler-state-and-message-plumbing to origin.
- branch 'task/agent-core-02-dry-handler-state-and-message-plumbing' set up to track 'origin/task/agent-core-02-dry-handler-state-and-message-plumbing'.
- Created PR: https://github.com/ineersa/agent-core/pull/392
- Validation: Final focused task-to-pr validation passed: castor test, castor deptrac, castor phpstan, castor cs-check.; Implementation validation passed: castor test:controller-replay.; Worktree clean and git diff --check passed.
- Summary: Reviewer APPROVED commit e11c86680. Consolidated AdvanceRun callbacks, turn-completed callbacks, and model-notification plumbing; approved mailbox multipart text now uses newline joining.

## Task workflow update - 2026-08-16T02:57:17.977Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/agent-core-02-dry-handler-state-and-message-plumbing.
- Merged task/agent-core-02-dry-handler-state-and-message-plumbing into integration checkout.
- Merge made by the 'ort' strategy.
 .../Handler/AdvanceRunCallbackFactory.php          |  42 ++++++++
 src/AgentCore/Application/Handler/RunMetrics.php   |  15 +++
 .../Application/Pipeline/AdvanceRunHandler.php     |  18 +---
 .../Application/Pipeline/ApplyCommandHandler.php   |  18 +---
 .../Application/Pipeline/CommandMailboxPolicy.php  |   7 +-
 .../Application/Pipeline/LlmStepResultHandler.php  |  66 ++----------
 .../Application/Pipeline/StartRunHandler.php       |  19 +---
 .../Application/Pipeline/ToolCallResultHandler.php |  98 ++----------------
 .../Domain/Notification/ModelNotificationCodec.php |  70 +++++++++++++
 .../SymfonyAi/LlmPlatformAdapter.php               |  23 +----
 .../Handler/AdvanceRunCallbackFactoryTest.php      |  85 +++++++++++++++
 .../Pipeline/CommandMailboxPolicyTest.php          |  46 +++++++++
 .../Notification/ModelNotificationCodecTest.php    | 115 +++++++++++++++++++++
 13 files changed, 403 insertions(+), 219 deletions(-)
 create mode 100644 src/AgentCore/Application/Handler/AdvanceRunCallbackFactory.php
 create mode 100644 src/AgentCore/Domain/Notification/ModelNotificationCodec.php
 create mode 100644 tests/AgentCore/Application/Handler/AdvanceRunCallbackFactoryTest.php
 create mode 100644 tests/AgentCore/Domain/Notification/ModelNotificationCodecTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/agent-core-02-dry-handler-state-and-message-plumbing.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/agent-core-02-dry-handler-state-and-message-plumbing.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR https://github.com/ineersa/agent-core/pull/392 state: MERGED.; Pre-merge integration checkout clean.
- Summary: PR #392 confirmed merged on GitHub at merge commit b26a1ed8453cae67c7f17b652a52e024e3c79917. Moving task to DONE and syncing integration checkout.

## Task workflow update - 2026-08-16T02:59:37.304Z
- Validation: LLM_MODE=true castor check — PASS (QA run qa-20260816-025727-364657-6d8d53af).; All eight lanes passed: deptrac, test, controller-replay, TUI replay, llm-real, phpstan, cs-check, docs validation.; QA leak check passed; llama-proxy cache remained stable at 282 entries.
- Summary: Post-merge integration validation complete. Worktree cleanup was performed by the DONE transition. No next task started per user request.

## Task workflow update - 2026-08-18T00:06:36.218Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
