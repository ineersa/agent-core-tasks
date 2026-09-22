# Surface actionable extension-agent failure reasons in the TUI

## Goal
Extension-agent jobs currently emit only the generic final-failure message `Extension background job failed after retrying.` In session 1, observational-memory observer jobs failed because OpenAI returned `usage_limit_reached`, but the TUI hid that actionable reason. Preserve the generic extension-agent boundary rather than adding OM-specific handling. Investigate the provider/exception classification path and propagate a safe, structured failure reason to the final runtime event and transcript error block without exposing prompts, tool output, credentials, arbitrary exception text, or other sensitive provider payloads.

## Acceptance criteria
- A permanently failed extension-agent job surfaces an actionable, privacy-safe reason in the TUI when Hatfield can classify it, including `usage_limit_reached`.
- Unknown or unsafe exceptions retain a sanitized generic message; raw arbitrary exception messages are never exposed.
- The behavior remains generic to extension-agent jobs and does not add observational-memory-specific branching.
- Failure reporting does not fail the parent run and preserves existing retry/final-failure semantics unless a separately justified classification change is required.
- Add deterministic automated regression coverage for safe reason propagation, sanitization fallback, and transcript projection at the lowest correct layer.
- Follow the testing skill and run the required Castor validation for the affected runtime/TUI path.

## Workflow metadata
Status: ARCHIVE
Branch: task/surface-actionable-extension-agent-failure-reasons
Worktree: /home/ineersa/projects/agent-core-worktrees/surface-actionable-extension-agent-failure-reasons
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/464
PR Status: merged
Started: 2026-09-04T19:33:12.550Z
Completed: 2026-09-04T21:29:42.632Z

## Work log
- Created: 2026-09-02T00:40:40.716Z

## Task workflow update - 2026-09-04T19:33:12.550Z
- Moved TODO → IN-PROGRESS.
- Created branch task/surface-actionable-extension-agent-failure-reasons.
- Created worktree /home/ineersa/projects/agent-core-worktrees/surface-actionable-extension-agent-failure-reasons.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/surface-actionable-extension-agent-failure-reasons.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/surface-actionable-extension-agent-failure-reasons.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/surface-actionable-extension-agent-failure-reasons.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/surface-actionable-extension-agent-failure-reasons.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/surface-actionable-extension-agent-failure-reasons/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/surface-actionable-extension-agent-failure-reasons.
- Summary: Started task-start phase; routing pass and implementation will run in the task worktree.

## Task workflow update - 2026-09-04T19:37:16.117Z
- Ownership: owner=main; fork_run=none; revision=9e0d2c2bc; scope=classify privacy-safe extension-agent failures, propagate the reason through the runtime event and transcript projection, and add lowest-layer regression proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-04T19:45:34.867Z
- Validation: PASS: `castor test --filter='ExtensionAgentJobFailed|testExtensionAgentUsageLimitFailureRendersActionableError'` (9 tests, 58 assertions); PASS: `castor phpstan --path=src/CodingAgent/Extension/Agent/ExtensionAgentJobFailedEventSubscriber.php`; PASS: `castor deptrac` (0 violations, 0 errors); PASS: `castor cs-check`; PASS: `git diff --check`
- Summary: Implemented generic privacy-safe extension-agent failure classification. Allowlisted structured provider code `usage_limit_reached` and existing typed LLM error categories now produce fixed actionable runtime messages/reasons; unknown or unsafe text keeps the generic retry-exhausted fallback. Added runtime-event, projection, and virtual TUI regression coverage without changing parent-run failure or retry behavior. Read and followed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`.
- Ownership: owner=main; fork_run=none; revision=9e0d2c2bc; scope=classify privacy-safe extension-agent failures, propagate the reason through the runtime event and transcript projection, and add lowest-layer regression proof; outcome=completed; commit=ca750484d

## Task workflow update - 2026-09-04T20:19:21.568Z
- Review: role=reviewer; artifact=inline-subagent-result; revision=ca750484d; scope=specification fidelity, privacy, retry/parent-run semantics, architecture, provider exception path, and lowest-layer TUI proof; verdict=REQUEST CHANGES; blockers=restrict class-based LLM attribution to unambiguous Symfony AI exceptions, align protocol payload documentation, then complete the transition-owned castor check
- Ownership: owner=main; fork_run=none; revision=ca750484d; scope=apply reviewer fixes for provider-class attribution and protocol documentation; outcome=assigned; commit=none

## Task workflow update - 2026-09-04T20:21:56.956Z
- Validation: PASS after reviewer fixes: `castor test --filter='ExtensionAgentJobFailed|testExtensionAgentUsageLimitFailureRendersActionableError'` (9 tests, 58 assertions); PASS after reviewer fixes: `castor phpstan`; PASS after reviewer fixes: `castor deptrac` (0 violations, 0 errors); PASS after reviewer fixes: `castor cs-check`; PASS focused provider smoke: `castor test:llm-real` (5 tests, 30 assertions); PASS after reviewer fixes: IDE diagnostics and `git diff --check`
- Summary: Applied reviewer fixes: class-based LLM messages now require a Symfony AI exception, so unrelated extension HTTP transport failures retain the generic fallback. Documented the bracketed ProviderErrorFormatter coupling and aligned the runtime payload table.
- Ownership: owner=main; fork_run=none; revision=ca750484d; scope=apply reviewer fixes for provider-class attribution and protocol documentation; outcome=completed; commit=46fee5385

## Task workflow update - 2026-09-04T20:29:15.473Z
- Validation: REVIEW APPROVED WITH SUGGESTIONS at 46fee5385: privacy-safe closed-set messages, generic fallback, retry semantics, transcript projection, and virtual TUI proof verified; Pending transition gate: deterministic `castor check` via `move_task(to=CODE-REVIEW)`
- Summary: Independent re-review approved the complete branch at 46fee5385 with suggestions only. No correctness, security, specification-fidelity, architecture, retry, parent-run, or test-layer blockers remain. Full `castor check` remains the transition-owned gate.
- Review: role=reviewer; artifact=inline-subagent-result; revision=46fee5385; scope=complete branch specification fidelity, privacy, retry/parent-run semantics, architecture, provider exception path, and lowest-layer TUI proof; verdict=APPROVE WITH SUGGESTIONS; blockers=none

## Task workflow update - 2026-09-04T20:31:39.570Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (127.7s).
- Pushed task/surface-actionable-extension-agent-failure-reasons to origin.
- branch 'task/surface-actionable-extension-agent-failure-reasons' set up to track 'origin/task/surface-actionable-extension-agent-failure-reasons'.
- Created PR: https://github.com/ineersa/agent-core/pull/464
- Validation: PASS: focused regression tests (9 tests, 58 assertions); PASS: `castor phpstan`; PASS: `castor deptrac`; PASS: `castor cs-check`; PASS: `castor test:llm-real` (5 tests, 30 assertions); Independent reviewer: APPROVE WITH SUGGESTIONS at 46fee5385
- Summary: Prepared for code review at 46fee5385 after independent approval. The implementation surfaces fixed privacy-safe extension-agent failure reasons, including usage_limit_reached, while preserving the generic fallback and retry/parent-run semantics.

## Task workflow update - 2026-09-04T20:51:53.800Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User review superseded the privacy-safe classification design: unwrap the final Messenger failure and show the extension agent's actual underlying error instead of maintaining allowlists, category mapping, or provider-message parsing.

## Task workflow update - 2026-09-04T20:52:29.432Z
- Summary: Accepted PR feedback: remove safe-provider classification and display the actual unwrapped extension-agent exception message. Keep final-failure gating, runtime/transcript projection, and parent-run semantics unchanged. Use the generic message only when the unwrapped exception message is empty.
- Specification clarification: latest user direction supersedes the original raw-exception sanitization criterion; final extension-agent failures should show the unwrapped underlying exception message without provider allowlists, category mapping, or message parsing.
- Ownership: owner=main; fork_run=none; revision=46fee5385; scope=replace provider failure classification with direct Messenger exception unwrapping and update runtime, projection, protocol, and virtual TUI proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-04T20:56:06.510Z
- Validation: PASS: `castor test --filter='ExtensionAgentJobFailed|testExtensionAgentFailureRendersUnderlyingError'` (8 tests, 50 assertions); PASS: `castor phpstan`; PASS: `castor deptrac` (0 violations, 0 errors); PASS: `castor cs-check`; PASS: IDE diagnostics and `git diff --check`
- Summary: Replaced provider allowlists, category mapping, and regex parsing with direct unwrapping of Symfony Messenger wrapped exceptions. Final extension-agent events now carry the underlying handler exception message. The transcript displays that message and keeps its existing generic fallback for empty or missing text.
- Ownership: owner=main; fork_run=none; revision=46fee5385; scope=replace provider failure classification with direct Messenger exception unwrapping and update runtime, projection, protocol, and virtual TUI proof; outcome=completed; commit=6e04056be

## Task workflow update - 2026-09-04T21:05:30.081Z
- Review: role=reviewer; artifact=inline-subagent-result; revision=6e04056be; scope=specification fidelity, Messenger unwrapping semantics, runtime/transcript behavior, protocol docs, and virtual TUI proof; verdict=REQUEST CHANGES; blockers=unwrap string-keyed Messenger handler exceptions, reproduce production keying in test, and align stale sanitization comment
- Ownership: owner=main; fork_run=none; revision=6e04056be; scope=fix production-shaped Messenger unwrap, add empty-message fallback proof, and align naming/documentation; outcome=assigned; commit=none

## Task workflow update - 2026-09-04T21:07:18.374Z
- Validation: PASS after reviewer fixes: `castor test --filter='ExtensionAgentJobFailed|testExtensionAgentFailureRendersUnderlyingError'` (8 tests, 50 assertions); PASS after reviewer fixes: `castor phpstan`; PASS after reviewer fixes: `castor deptrac` (0 violations, 0 errors); PASS after reviewer fixes: `castor cs-check`; PASS after reviewer fixes: IDE diagnostics and `git diff --check`
- Summary: Fixed unwrapping for production-shaped Messenger failures whose wrapped exceptions retain string handler keys. Updated the regression fixture to use a string handler key, proved the empty-message transcript fallback, and removed stale sanitized/safe naming.
- Ownership: owner=main; fork_run=none; revision=6e04056be; scope=fix production-shaped Messenger unwrap, add empty-message fallback proof, and align naming/documentation; outcome=completed; commit=bcd94070e

## Task workflow update - 2026-09-04T21:15:14.753Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (90.1s).
- Pushed task/surface-actionable-extension-agent-failure-reasons to origin.
- branch 'task/surface-actionable-extension-agent-failure-reasons' set up to track 'origin/task/surface-actionable-extension-agent-failure-reasons'.
- PR already exists: https://github.com/ineersa/agent-core/pull/464
- Validation: PASS: focused regression tests (8 tests, 50 assertions); PASS: `castor phpstan`; PASS: `castor deptrac` (0 violations, 0 errors); PASS: `castor cs-check`; Independent reviewer: APPROVE at bcd94070e
- Summary: Updated PR #464 after user review. Final extension-agent failures now unwrap Symfony Messenger handler failures and show the underlying exception message directly. Removed provider allowlists, classification, and regex parsing. Production string-keyed wrapper behavior and empty-message transcript fallback are covered.

## Task workflow update - 2026-09-04T21:15:23.464Z
- Validation: PASS: deterministic `castor check` at bcd94070e (90.1s); Independent reviewer: APPROVE at bcd94070e
- Summary: PR #464 updated at bcd94070e. Independent reviewer approved the direct unwrapping implementation. No unresolved blockers.
- Review: role=reviewer; artifact=inline-subagent-result; revision=bcd94070e; scope=complete branch specification fidelity, production Messenger unwrapping, runtime/retry/parent-run semantics, protocol docs, and lowest-layer TUI proof; verdict=APPROVE; blockers=none
- Review iteration: decision=APPROVE; revision=bcd94070e; commit=bcd94070e; validation=focused tests, phpstan, deptrac, cs-check, and transition castor check passed; unresolved_blockers=none

## Task workflow update - 2026-09-04T21:29:42.632Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/surface-actionable-extension-agent-failure-reasons: ide_close_project returned isError.
- Merged task/surface-actionable-extension-agent-failure-reasons into integration checkout.
- Merge made by the 'ort' strategy.
 config/packages/messenger.yaml                     |  2 +-
 .../ExtensionAgentJobFailedEventSubscriber.php     | 27 ++++++++----
 ...ExtensionAgentJobFailedProjectionSubscriber.php |  8 ++--
 src/CodingAgent/Runtime/Protocol/AGENTS.md         |  4 +-
 .../ExtensionAgentJobFailedEventSubscriberTest.php | 49 ++++++++++++++++++----
 ...nsionAgentJobFailedProjectionSubscriberTest.php | 10 +++--
 .../TuiTranscriptBlocksVirtualRenderTest.php       | 31 ++++++++++++++
 7 files changed, 106 insertions(+), 25 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/surface-actionable-extension-agent-failure-reasons.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/surface-actionable-extension-agent-failure-reasons.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #464 merged as 22f7fadf09c4f1b5ccf67257f53bca6c1506e9e8. Final implementation unwraps Symfony Messenger extension-agent failures and displays the underlying exception message directly in the TUI.

## Task workflow update - 2026-09-04T21:31:22.744Z
- Validation: PASS in integration checkout: `LLM_MODE=true castor check` (201.1s, 10 lanes, 4,738 unit tests, 19,277 assertions, no QA process/tmux leaks, llama-proxy cache unchanged)
- Summary: Post-merge validation passed in the integration checkout after PR #464 merged.

## Task workflow update - 2026-09-06T15:41:23+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
