# Add context-budget system reminders that make agents wrap up before exhaustion

## Goal
## Problem

Forks, subagents, and long-running parent agents can consume nearly their entire model context while exploring, then terminate with a length/context failure instead of returning a useful final answer or child handoff. The urgent requirement is a built-in model-visible reminder when fewer than 25,000 usable context tokens remain. Large-context GPT-family models also need an earlier provider-neutral checkpoint so they do not spend hundreds of thousands of tokens on open-ended exploration before the final warning.

## Requested behavior

At low remaining context, tell the model to stop broad exploration, avoid starting new delegation, and immediately produce the best concise final response or child handoff with concrete findings, partial results, and next steps.

This must cover normal parent, fork, and subagent runs. It must not depend on model names or provider-specific branches.

## Claude Code reference

Local exploration of `/home/ineersa/claw/claude-code` found:

- `src/services/compact/autoCompact.ts` separates effective context, warning, auto-compaction, and hard limits. It reserves output headroom and uses 20k warning/error buffers plus a 13k auto-compaction buffer.
- `src/utils/attachments.ts` can emit model-visible token usage (`used/total; remaining`) as a meta user message wrapped in `<system-reminder>`.
- Large-window compaction guidance is feature-gated and tells Claude that auto-compaction exists; it does **not** tell the model to wrap up.
- No verified Claude Code implementation asks the model to finish near exhaustion. This task intentionally adds different behavior suitable for delegated agents that must return a useful artifact.
- Stale remaining-context warnings are suppressed after compaction until fresh provider usage is available.

## Agent Core findings

Agent Core already has the required provider-neutral inputs:

- `AiModelDefinition::contextWindow` and `RunMetadata::contextWindow`
- provider-reported latest `input_tokens` persisted by `LlmStepResultHandler`
- `ProviderContextUsageResolver`
- pre-LLM policy/deduplication patterns in `CodingAgentPreLlmCompactionGuard`
- shared normal-run scheduling in `AdvanceRunHandler`
- shared provider prompt assembly in `LlmPlatformAdapter`

Fork and subagent launchers already propagate context-window metadata. Parent runs may require catalog fallback when `RunMetadata` lacks it.

The preferred boundary is a dedicated pre-LLM context-budget policy invoked for normal agent turns after compaction has been considered. The reminder should be transient invocation context, not a canonical user message, and compaction/summarization invocations must be excluded.

## Design scope

Implement two separately defined checkpoints:

1. **Urgent wrap-up checkpoint:** default at fewer than 25,000 usable tokens remaining.
2. **Early exploration checkpoint:** a configurable, provider-neutral soft checkpoint for large or long-running contexts, intended to make the model reassess, synthesize, and stop low-yield exploration before the urgent threshold. Choose and document a defensible metric (for example effective context occupancy and/or bounded tool-turn exploration), rather than special-casing GPT model IDs. Do not blindly sum cumulative input tokens, which repeatedly include prior context.

The implementation should ask rather than abruptly kill an active child. Any hard enforcement such as blocking new exploratory delegation must be a separately justified policy decision and must preserve finalization, cancellation, compaction, and artifact retrieval.

## Acceptance criteria
- Define usable remaining context from catalog/run context-window metadata and fresh provider-reported prompt usage, including explicit output headroom semantics.
- Inject a one-shot urgent system reminder by default when usable remaining context drops below 25,000 tokens, covering parent, fork, and subagent normal turns.
- The urgent reminder explicitly tells the model to stop broad exploration, avoid new delegation, and return a concise final answer or child handoff with findings, partial results, and next steps.
- Add a configurable provider-neutral early exploration checkpoint that addresses runaway use of very large contexts without branching on provider IDs or model names; document the chosen metric and why it does not misinterpret cumulative prompt-token reuse.
- Keep reminders transient and out of canonical `RunState.messages`/session replay while ensuring they are delivered as authoritative model-visible system guidance.
- Exclude compaction/summarization invocations and prevent duplicate reminder injection across retries or repeated handling of the same provider usage measurement.
- After successful compaction, suppress stale low-context decisions until fresh usage is available and allow a later genuinely low-context episode to trigger again.
- Define behavior when context-window metadata or provider usage is unavailable; do not estimate or enforce silently without documented semantics.
- Preserve retry safety: a committed/scheduled reminder must not be lost if effect dispatch retries, and duplicate run-control handling must not send it repeatedly.
- Add focused policy and provider-boundary regression proof for threshold edges, one-shot behavior, parent/child coverage, transient prompt injection, compaction reset, and missing metadata. Add protocol/replay proof only if runtime protocol or canonical events change.
- Update settings documentation and project example settings for any new configuration keys.
- Because this changes runtime and LLM-visible guidance, run focused Castor validation including `castor test:llm-real`, then mandatory `castor check` before CODE-REVIEW.

## Workflow metadata
Status: DONE
Branch: task/context-budget-wrap-up-system-reminders
Worktree: /home/ineersa/projects/agent-core-worktrees/context-budget-wrap-up-system-reminders
Fork run: gi3n6k05lqhc
PR URL: https://github.com/ineersa/agent-core/pull/327
PR Status: merged
Started: 2026-07-23T18:08:38+00:00
Completed: 2026-07-29T17:08:44.809Z

## Work log
- Created: 2026-07-21T20:01:51.068Z

## Task workflow update - 2026-07-21T20:05:48.455Z
- Summary: Scope clarification from user: this task is reminder-only. It must not add, redesign, trigger, tune, or otherwise change compaction; must not add hard token-budget enforcement, block tools/delegation, cancel children, or kill runs. Existing compaction is relevant only as an exclusion/reset boundary so a reminder is not injected into a summarization call or repeated from stale pre-compaction usage.
- Authoritative narrowed reminder plan: (1) one urgent, transient system reminder on the first normal model turn where fresh usage shows <25,000 context tokens remaining; instruct the model to stop exploring and finish with a concise final answer/handoff; (2) an optional earlier advisory reminder for unusually long exploration remains a product decision and must not be implemented until its exact trigger is agreed. Reminder injection only—no compaction or enforcement feature.

## Task workflow update - 2026-07-21T20:13:51.472Z
- Summary: User finalized reminder triggers. Apply reminder policy everywhere normal agent execution occurs: parent agents, forks, subagents, and any other normal model run using the shared pipeline. Add two independent one-shot transient system reminders: an early wrap-up nudge once current prompt/context usage reaches 200,000 tokens, and an urgent wrap-up reminder once fewer than 25,000 usable context tokens remain. Reminder-only scope remains authoritative; no compaction changes or hard enforcement.
- Finalized early trigger: latest fresh provider-reported context/input usage >= 200,000 tokens. Suggested text: “Context usage is already very high. Stop further exploration and do not start new delegated work. Finish now with the best concise final answer or handoff, including concrete findings, incomplete work, and next steps.”
- Finalized urgent trigger: remaining usable context < 25,000 tokens. Text: “Context is nearly exhausted. Stop further exploration and do not start new delegated work. Finish now with the best concise final answer or handoff, including concrete findings, incomplete work, and next steps.”
- Each reminder is emitted at most once per run/threshold, remains transient (not canonical history), and does not forcibly terminate the model. If a run crosses directly past both thresholds, send only the stronger urgent reminder rather than duplicating near-identical reminders in one invocation.

## Task workflow update - 2026-07-23T18:08:38+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/context-budget-wrap-up-system-reminders.
- Created worktree /home/ineersa/projects/agent-core-worktrees/context-budget-wrap-up-system-reminders.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/context-budget-wrap-up-system-reminders.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/context-budget-wrap-up-system-reminders.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/context-budget-wrap-up-system-reminders.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/context-budget-wrap-up-system-reminders.
- Summary: Claimed for implementation of transient one-shot early (>=200k fresh prompt usage) and urgent (<25k usable remaining) model-visible wrap-up reminders across the shared normal-run pipeline; reminder-only scope, with no compaction or enforcement changes.

## Task workflow update - 2026-07-27T23:13:53.246Z
- Recorded fork run: 1dmclsiggp8j
- Implementation resumed 2026-07-27. Loaded task-workflow, testing, and ponytail skills; read tests/AGENTS.md. Two scouts traced the shared normal-run path and transient prompt boundary. Delegated all implementation to fork run 1dmclsiggp8j in the task worktree, with instructions to merge current main non-destructively, implement reminder-only behavior, add focused proof, run focused Castor validation, commit, and stop before PR/gate.

## Task workflow update - 2026-07-27T23:17:35.249Z
- Summary: Fork 1dmclsiggp8j was stopped after consuming substantial context in repetitive design narration without producing file changes. The branch was successfully fast-forwarded to current main and remains clean; contrary to the fork's narration, there was no merge conflict. Relaunching implementation with narrower, concrete instructions.

## Task workflow update - 2026-07-27T23:18:00.480Z
- Recorded fork run: gc9whb8tfqat
- Relaunched implementation as fork gc9whb8tfqat with a concrete minimal design: internal non-content completion markers for durable one-shot episodes, immutable reminder transport on ExecuteLlmStep retries, transient provider system guidance, successful-compaction freshness barrier, catalog fallback, and focused tests/docs.

## Task workflow update - 2026-07-27T23:27:49.922Z
- Recorded fork run: gc9whb8tfqat
- Validation: PASS: filtered Castor reminder tests — 14 tests; PASS: related filtered Castor tests — 89 tests; PASS: castor deptrac — 0 violations; PASS: castor phpstan — 0 errors; PASS: castor cs-check; PARTIAL: castor test — 4537 tests with 1 unrelated/pre-existing observational-memory extension autoload error (`OmTestDatabase` not found); focused and related suites pass; NOT RUN by design in task-start phase: castor test:llm-real and castor check; required before CODE-REVIEW
- Summary: Implementation committed at bc0c96ff0097db034c210d3ba8a4b1e3148834d1. Added provider-neutral early (>=200k current prompt tokens) and urgent (<25k usable remaining) one-shot transient system reminders on the shared normal LLM path; immutable retry transport; non-content completion/abortion markers for durable episode dedup; successful-compaction freshness reset; run metadata/catalog context-window resolution; explicit zero output-headroom default; settings and docs. Reminder prose never enters RunState.messages/events/transcript. Worktree is clean. Required testing skill and tests/AGENTS.md were read by the fork.

## Task workflow update - 2026-07-27T23:49:21.188Z
- Validation: PASS: reviewer — APPROVE WITH SUGGESTIONS, no blockers; PASS: castor test — 4549 tests, 15928 assertions (after refreshing stale copied vendor autoload); PASS: castor test:llm-real — 13 tests, 175 assertions; PASS: castor deptrac — 0 violations/errors; PASS: castor phpstan — 0 errors; PASS: castor cs-check — clean; PASS: focused transient-reminder tests from implementation — 14 tests; related filter — 89 tests
- Summary: Task-to-PR review completed. Reviewer found no critical/correctness/security blockers and returned APPROVE WITH SUGGESTIONS (treated as approved): optional comments/tests for failed-vs-aborted marker intent and an AdvanceRun integration assertion. Current implementation already satisfies the required policy and provider-boundary proof; suggestions are non-blocking and no code changes were made. Refreshed the worktree's copied vendor/ from current main after diagnosing stale Composer autoload metadata; the previously failing observational-memory support class then loaded and the full suite passed. Commit remains bc0c96ff0097db034c210d3ba8a4b1e3148834d1 and worktree is clean.

## Task workflow update - 2026-07-27T23:52:40.714Z
- Validation: FAIL (first CODE-REVIEW gate): castor check test:llm-real lane — ShellFollowUpLiveE2eTest missed command.ack; PASS: castor clean:cleanup:workers:list — no stale QA worker candidates; PASS: castor test:llm-real --filter=ShellFollowUpLiveE2eTest — 2 tests, 21 assertions
- Summary: First deterministic CODE-REVIEW gate attempt failed only in live ShellFollowUpLiveE2eTest: follow-up collection saw run.completed but missed command.ack. No stale QA workers were present. Focused `castor test:llm-real --filter=ShellFollowUpLiveE2eTest` immediately passed both tests (21 assertions), indicating a transient live E2E timing failure rather than reminder behavior; retrying the deterministic gate without code changes.

## Task workflow update - 2026-07-27T23:54:59.822Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (129.1s).
- Pushed task/context-budget-wrap-up-system-reminders to origin.
- branch 'task/context-budget-wrap-up-system-reminders' set up to track 'origin/task/context-budget-wrap-up-system-reminders'.
- Created PR: https://github.com/ineersa/agent-core/pull/327
- Validation: Focused ShellFollowUpLiveE2eTest rerun passed (2 tests, 21 assertions); Worktree clean at bc0c96ff0097db034c210d3ba8a4b1e3148834d1
- Summary: Retrying CODE-REVIEW transition after the sole failed live E2E test passed focused rerun; no code changes and no stale workers.

## Task workflow update - 2026-07-28T01:36:57.765Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User review found an invented public setting: output_headroom_tokens duplicates the existing 25k wrap-up reserve and was not specified as a separate configurable buffer. Reopening to remove the setting and subtraction, define remaining context as context window minus current prompt usage, and keep the 25k threshold as the sole reserve.

## Task workflow update - 2026-07-28T01:37:14.582Z
- Recorded fork run: rlsyoerpmmd8
- User-requested correction delegated to fork rlsyoerpmmd8: remove all output_headroom settings/calculation/docs/tests, retain 25k as the sole reserve, and make no other changes.

## Task workflow update - 2026-07-28T01:41:14.422Z
- Recorded fork run: ny7xpxs8a4c1
- Validation: PASS: focused reminder tests — 13 tests, 34 assertions; PASS: castor phpstan; PASS: castor cs-check; PASS: castor deptrac; REVIEW: REQUEST CHANGES only for one stale test arithmetic comment
- Summary: Re-review requested one comment-only correction: a threshold-edge test comment retained the removed `- 0` arithmetic term. Delegated exact one-line cleanup to fork ny7xpxs8a4c1; production behavior and configuration removal are otherwise approved.

## Task workflow update - 2026-07-28T01:42:55.894Z
- Recorded fork run: ny7xpxs8a4c1
- Validation: PASS: focused reminder tests — 13 tests, 34 assertions; PASS: focused policy test after comment cleanup — 11 tests, 26 assertions; PASS: castor phpstan; PASS: castor cs-check; PASS: castor deptrac; PASS: final reviewer — APPROVED
- Summary: User-requested correction complete at 1394f05a6 + comment cleanup 115d4c54b. Removed the unrequested output-headroom setting and subtraction from all config, docs, policy, and tests. Remaining context is now exactly context window minus latest current prompt usage; the 25k urgent threshold is the sole reserve. Final re-review APPROVED with no blockers.

## Task workflow update - 2026-07-28T01:44:24.015Z
- Validation: BLOCKED: deterministic castor check could not acquire shared repo lock within 60s; holder is an active current-user castor:check in /home/ineersa/projects/agent-core
- Summary: CODE-REVIEW gate retry was blocked only by the shared repository Castor lock: an unrelated `castor check` is running in the integration checkout (holder pid 932989, qa run qa-20260728-014247-932989-d577e351). No validation or code failure; waiting through the normal lock mechanism and retrying without touching the holder process.

## Task workflow update - 2026-07-28T01:47:04.927Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (149.1s).
- Pushed task/context-budget-wrap-up-system-reminders to origin.
- branch 'task/context-budget-wrap-up-system-reminders' set up to track 'origin/task/context-budget-wrap-up-system-reminders'.
- PR already exists: https://github.com/ineersa/agent-core/pull/327
- Validation: Focused reminder/policy tests passed; Deptrac, PHPStan, and CS checks passed; Final reviewer APPROVED; Worktree clean at 115d4c54bb5e2dc4dbb15a4cc8343bfedbec0b16
- Summary: Retrying CODE-REVIEW transition after shared Castor lock contention; user-requested headroom removal and final review remain complete.

## Task workflow update - 2026-07-28T16:15:38.691Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Owner review rejects the current architecture as incorrect. Authoritative correction: after an LLM response, use the existing hook/event boundary to check reported context usage; when the finalized early/urgent trigger is crossed, queue an existing user steering/append message wrapped in `<system-reminder>`. AgentCore must not know about reminders. Remove the custom AgentCore reminder transport, DTOs, policy wiring, result markers, and prompt-injection path rather than adapting them.

## Task workflow update - 2026-07-28T16:22:06.378Z
- Recorded fork run: enito6nc0hmw
- Summary: After reading the documented runtime/session/agent/compaction architecture, delegated full correction to fork enito6nc0hmw. Required final shape: one CodingAgent after-turn hook subscriber using existing AgentRunner::appendMessage with user `<system-reminder>` content; existing two thresholds only; canonical generic queued/applied events for one-shot/reset; all reminder-specific AgentCore/provider/pipeline machinery removed.

## Task workflow update - 2026-07-28T17:17:13.581Z
- Recorded fork run: enito6nc0hmw
- Validation: Fork: castor test --filter=ContextBudgetReminder — OK (7 tests, 15 assertions); Parent: castor test — OK (4421 tests, 15525 assertions); Parent: castor deptrac — 0 violations; Parent: castor phpstan — 0 errors; Parent: castor cs-check — clean (files_fixed=0); git diff --check — clean; git diff origin/main...HEAD -- src/AgentCore — empty; Reviewer: APPROVE WITH SUGGESTIONS; specification fidelity and minimality gates passed
- Summary: Architecture correction committed at fec882b6d. Final diff vs origin/main is 8 CodingAgent/config/docs/test files (+759) with zero changes under src/AgentCore. Replaced rejected pre-LLM transport/policy/provider injection with one existing after-turn HookSubscriber that inspects committed llm_step_completed usage and queues a user append_message wrapped in <system-reminder>. Exactly two thresholds remain; canonical generic queued/applied events provide one-shot behavior reset by successful compaction. Reviewer verdict: APPROVED WITH SUGGESTIONS, no blockers or unmapped surface.

## Task workflow update - 2026-07-28T17:24:37.139Z
- Validation: castor test:llm-real focused RewindBranchLiveE2eTest — OK (1 test, 19 assertions), cache entries stable 194→194; Full llm-real previously passed 13 tests/175 assertions during initial gate/fork validation; later parallel warm attempt showed unrelated RewindBranchLiveE2eTest completion timing failure
- Summary: First CODE-REVIEW gate failed only because llama-proxy cache grew 181→185; implementation lanes were not reported failing. Warmed live cache. Full llm-real warm runs exposed one unrelated parallel-only RewindBranchLiveE2eTest timing failure; focused rerun passed (1 test, 19 assertions) with cache stable. Retrying deterministic gate after warmup.

## Task workflow update - 2026-07-28T17:26:45.567Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (115.9s).
- Pushed task/context-budget-wrap-up-system-reminders to origin.
- branch 'task/context-budget-wrap-up-system-reminders' set up to track 'origin/task/context-budget-wrap-up-system-reminders'.
- PR already exists: https://github.com/ineersa/agent-core/pull/327
- Validation: Reviewer approved with no blockers; castor test: 4421 tests, 15525 assertions; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean; AgentCore diff: empty; Live cache warmed; focused RewindBranch live test passed
- Summary: Owner-requested architecture rewrite complete and re-reviewed. PR #327 now uses only existing CodingAgent after-turn hooks and AgentRunner append_message; all reminder-specific AgentCore/provider transport code removed.

## Task workflow update - 2026-07-28T18:32:09.485Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Owner requested temporary project test override: change `.hatfield/settings.yaml` early_input_tokens from 200000 to 30000 only. Defaults/docs remain 200000.

## Task workflow update - 2026-07-28T18:33:07.396Z
- Recorded fork run: 706mj6wtr9tx
- Validation: One-file one-line diff verified; Working tree clean; Broad QA skipped for YAML scalar-only temporary override
- Summary: Temporary manual-test override committed locally at d61b27b38: `.hatfield/settings.yaml` early_input_tokens changed 200000→30000 only. Defaults/docs/code/tests and urgent threshold unchanged. Branch is one commit ahead of PR and intentionally remains IN-PROGRESS/unpushed pending owner testing; revert or keep before returning to review.

## Task workflow update - 2026-07-28T19:26:36.931Z
- Recorded fork run: 0cc70y6f78a8
- Validation: Focused projection + virtual TUI tests — OK (2 tests, 14 assertions); castor phpstan — 0 errors; castor cs-check — clean after scoped cs-fix; git diff --check — clean
- Summary: Reminder transcript styling committed locally at c72104538. Complete `<system-reminder>` user messages now project as the existing warning System block (`⚠` + theme Warning color + inner prose) for live/replay presentation. Canonical/model append_message retains the full wrapper. No AgentCore/protocol/settings/pending-queue changes. Branch remains unpushed for owner manual retest and still includes temporary 30k override commit d61b27b38.

## Task workflow update - 2026-07-28T20:20:12.467Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/327
- Summary: Signed Symfony monorepo patch pushed at d0ddc674f5c9d30dad5feda8390815e21c9136dd on ineersa/symfony:fix-tui-markdown-inline-html. Composer integration using that monorepo VCS URL failed because it does not publish component package symfony/tui; failed composer.json edits are being restored. Created Hatfield follow-up issue #332 documenting reproduction, root cause, intentional existing strip-HTML contract, signed patch, Composer limitation, and upstream issue/PR checklist.
- Hatfield follow-up issue: https://github.com/ineersa/agent-core/issues/332
- Symfony signed patch: https://github.com/ineersa/symfony/commit/d0ddc674f5c9d30dad5feda8390815e21c9136dd
- Composer monorepo VCS resolution failed: branch is not exposed as package symfony/tui; component-split fork/source is required for temporary dependency override.

## Task workflow update - 2026-07-28T20:48:31.292Z
- Recorded fork run: 6esohqpqru1w
- Validation: Scoped `composer update symfony/tui --no-interaction` succeeded with exactly one package update.; Verified composer.lock source `https://github.com/ineersa/tui.git` at `6de5aaa989d561e61682b1fc48d984bca22eb581`.; No QA run per owner's instruction for this Composer-only switch.
- Summary: Temporary Symfony TUI component fork wired successfully. composer.json now declares VCS repository https://github.com/ineersa/tui and requires dev-fix-tui-markdown-inline-html; Composer-generated lock pins symfony/tui to 6de5aaa989d561e61682b1fc48d984bca22eb581. Commit 7324642215f15642e9a25409cc0e105b5003b237; branch ahead 3 and clean; not pushed.

## Task workflow update - 2026-07-28T22:28:56.669Z
- Recorded fork run: l9gi95dni6nx
- Validation: castor test --filter=ContextBudgetReminderHookSubscriberTest: OK (7 tests, 15 assertions); castor test --filter=RuntimeEventMapperTest: OK (50 tests, 171 assertions); castor test --filter=TranscriptProjectorTest: OK (97 tests, 387 assertions); castor test --filter=TuiTranscriptBlocksVirtualRenderTest: OK (31 tests, 103 assertions); castor phpstan: 0 errors; castor cs-check: files_fixed=0; castor deptrac: 0 violations; Final diff vs origin/main under src/AgentCore is empty; Composer uses upstream symfony/tui.
- Summary: Corrected reminder presentation provenance in commit 7635d5fa273cdac3e3ed86476d15fedcba41c4dc. Context-budget append now sets existing AgentMessage metadata.system_reminder=true; CodingAgent runtime translator preserves non-empty message metadata; user projection requires marker + exact wrapper before rendering System/warning. Identical manually typed wrapper remains UserMessage. Restored upstream symfony/tui ^8.1, locked v8.1.1 at 51ff44b98a27c6a5522d9ac6829b28f2b40d0014; removed fork override. Closed obsolete Hatfield issue #332 after confirming assistant source used Markdown backticks and Symfony behavior was not the reminder root cause. Branch clean, ahead 4, not pushed.

## Task workflow update - 2026-07-29T00:04:46.641Z
- Recorded fork run: nx1pdj90o24q
- Validation: Reviewer: APPROVE, no blockers.; castor test: 2601 tests / 9579 assertions with one unrelated known flaky SQLite contention timing failure (82ms vs >=140ms).; Focused rerun `castor test --filter=MessengerSqliteImmediateTransactionMiddlewareTest`: OK (4 tests, 19 assertions).; castor test:tui: OK (38 tests, 193 assertions).; castor deptrac: 0 violations.; castor phpstan: 0 errors.; castor cs-check: files_fixed=0.
- Summary: Final reviewer APPROVED after restoring tracked project early threshold to 200000 in commit 3387b8cb8ffc69ec488fbe3360f20e1d5d37c30f. Specification fidelity verified: 200k/25k consistent, metadata provenance correct, manual wrapper remains UserMessage, upstream Symfony TUI only, zero reminder diff under src/AgentCore.

## Task workflow update - 2026-07-29T00:07:34.571Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (150.7s).
- Pushed task/context-budget-wrap-up-system-reminders to origin.
- branch 'task/context-budget-wrap-up-system-reminders' set up to track 'origin/task/context-budget-wrap-up-system-reminders'.
- PR already exists: https://github.com/ineersa/agent-core/pull/327
- Validation: Reviewer APPROVE; Focused reminder/runtime/projection/virtual TUI tests pass; castor test:tui: 38 tests, 193 assertions; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean; SQLite contention flaky test passed focused rerun
- Summary: Final architecture corrected and approved: after-turn CodingAgent append_message hook with metadata provenance, warning projection only for marked exact wrappers, manual text preserved, temporary 30k setting restored to 200k, upstream Symfony TUI restored. Reviewer APPROVED; focused runtime/projection/TUI tests, TUI E2E, deptrac, phpstan, cs-check passed. One unrelated SQLite timing test flaked in full unit run and passed focused rerun.

## Task workflow update - 2026-07-29T00:18:07.597Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Owner live Session 4 still shows ordinary typed `<system-reminder>` disappearing. Canonical events confirm user source is raw `Check where we are using <system-reminder>?`, while assistant thinking source uses Markdown backticks around the tag. Metadata provenance fix correctly handles internal reminders but ordinary UserMessage still routes through MarkdownWidget, where upstream CommonMark intentionally strips raw inline HTML. Correct Hatfield fix is literal user transcript rendering (TextWidget) while assistant/thinking remain Markdown.

## Task workflow update - 2026-07-29T00:33:21.491Z
- Summary: Owner approved restoring the Symfony TUI patch with corrected semantics: CommonMark HtmlInline nodes must render through MarkdownWidget's existing inline-code style, equivalent to explicit backticks. Reopened issue #332 with corrected rationale. Rejected UserMessage→TextWidget commit 050b642cc was non-destructively reverted by a1fbae97f. Updating both Symfony monorepo fork and component fork before restoring Agent Core Composer pin.
- Fork ukamupu463d7 updating ineersa/symfony monorepo patch.
- Fork cav7ettkqs5q updating ineersa/tui component fork.

## Task workflow update - 2026-07-29T00:49:43.924Z
- Recorded fork run: shu20bb2ra5u
- Summary: Component fork patch is pushed unsigned (owner-authorized) at ineersa/tui commit 130bc651f2e22919c6a4036f6fe8658cfd2df78c. HtmlInline now renders through MarkdownWidget's existing code style; component MarkdownTest proves raw tag output equals explicit backticks (22 tests, 80 assertions). Delegated Agent Core Composer pin restoration plus one exact Session 4 virtual user-message regression; no production app changes.

## Task workflow update - 2026-07-29T00:49:53.840Z
- Summary: Both Symfony patches are now committed and pushed unsigned per owner authorization: monorepo ineersa/symfony 91b632da04e5f3d4c66165a69e94cf4c05612592; component ineersa/tui 130bc651f2e22919c6a4036f6fe8658cfd2df78c. Both branches clean and synchronized with origin. Agent Core Composer integration fork shu20bb2ra5u remains in progress.

## Task workflow update - 2026-07-29T00:53:09.515Z
- Recorded fork run: shu20bb2ra5u
- Validation: castor test --filter=TuiTranscriptBlocksVirtualRenderTest: OK (31 tests, 104 assertions); castor phpstan: 0 errors; castor cs-check: files_fixed=0; castor deptrac: 0 violations; git diff --check: clean; Final local delta vs pushed task branch: composer.json, composer.lock, one existing virtual test only.
- Summary: Agent Core integration complete at commit 54568a9584468415bd648d3af33d8efcf326e1f8. Composer pins ineersa/tui dev-fix-tui-markdown-inline-html at exact SHA 130bc651f2e22919c6a4036f6fe8658cfd2df78c; only symfony/tui changed in lock. Existing virtual user Markdown test now uses exact Session 4 raw `<system-reminder>` input, proves it remains visible with ❯, and retains inline Markdown behavior. No Hatfield production source changed. Issue #332 updated with monorepo/component/Agent Core commit evidence.

## Task workflow update - 2026-07-29T01:15:36.602Z
- Recorded fork run: 2uc707k2v8qo
- Validation: Final reviewer: APPROVED, no blockers.; composer validate: valid; composer install --dry-run: no changes/stale-lock warning.; castor test: 2771 tests / 10072 assertions with one unrelated known SQLite contention timing flake (132ms vs >=140ms).; Focused rerun `castor test --filter=MessengerSqliteImmediateTransactionMiddlewareTest`: OK (4 tests, 19 assertions).; castor test:tui: OK (38 tests, 193 assertions).; castor deptrac: 0 violations.; castor phpstan: 0 errors.; castor cs-check: files_fixed=0.
- Summary: Final reviewer APPROVED after refreshing Composer lock content hash in commit 2478831a905418ae20a000a77f4872d8f9e8e335. Exact fork SHA/source preserved; package graph unchanged; composer validate/install dry-run clean. Full local validation completed for final patched dependency and exact user-message regression.

## Task workflow update - 2026-07-29T01:17:59.259Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (125.9s).
- Pushed task/context-budget-wrap-up-system-reminders to origin.
- branch 'task/context-budget-wrap-up-system-reminders' set up to track 'origin/task/context-budget-wrap-up-system-reminders'.
- PR already exists: https://github.com/ineersa/agent-core/pull/327
- Validation: Reviewer APPROVED; Component/monorepo MarkdownTest: 22 tests, 80 assertions; Agent Core exact virtual regression: 31 tests, 104 assertions; castor test:tui: 38 tests, 193 assertions; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean; Known unrelated SQLite contention timing flake passed focused rerun (4 tests, 19 assertions)
- Summary: Final review iteration complete. Reminder architecture/provenance remains approved; ordinary UserMessage remains Markdown. Patched Symfony TUI fork maps HtmlInline to existing inline-code style, Agent Core pins exact component SHA 130bc651f2e22919c6a4036f6fe8658cfd2df78c, and exact Session 4 user input has virtual regression proof. Composer lock hash fixed and validated. Reviewer APPROVED.

## Task workflow update - 2026-07-29T16:52:47.431Z
- Summary: Updated open Hatfield issue #332 title/body to final semantics and current evidence: HtmlInline renders via existing inline-code style (not plain literal), assistant/thinking source distinction via backticks, separate reminder metadata provenance, current Symfony monorepo/component SHAs, Agent Core PR integration, 22-test/80-assertion proof, exact Session 4 regression, and upstream/removal follow-ups.

## Task workflow update - 2026-07-29T17:08:44.809Z
- Moved CODE-REVIEW → DONE.
- Merged task/context-budget-wrap-up-system-reminders into integration checkout.
- Auto-merging .hatfield/settings.yaml
Auto-merging config/services.yaml
Auto-merging docs/settings.md
Merge made by the 'ort' strategy.
 .hatfield/settings.yaml                            |   3 +
 composer.json                                      |   8 +-
 composer.lock                                      |  39 +--
 config/hatfield.defaults.yaml                      |  20 ++
 config/services.yaml                               |   7 +
 docs/settings.md                                   |  56 ++++
 src/CodingAgent/Config/AppConfig.php               |   6 +
 .../Config/ContextBudgetReminderConfig.php         |  39 +++
 .../ContextBudgetReminderHookSubscriber.php        | 269 +++++++++++++++
 .../UserMessageProjectionSubscriber.php            |  64 +++-
 .../Runtime/Protocol/RuntimeEventTranslator.php    |  18 +-
 .../ContextBudgetReminderHookSubscriberTest.php    | 365 +++++++++++++++++++++
 .../Runtime/Projection/TranscriptProjectorTest.php |  38 +++
 .../CodingAgent/Runtime/RuntimeEventMapperTest.php |   2 +
 .../TuiTranscriptBlocksVirtualRenderTest.php       |  57 +++-
 15 files changed, 950 insertions(+), 41 deletions(-)
 create mode 100644 src/CodingAgent/Config/ContextBudgetReminderConfig.php
 create mode 100644 src/CodingAgent/ContextBudget/ContextBudgetReminderHookSubscriber.php
 create mode 100644 tests/CodingAgent/ContextBudget/ContextBudgetReminderHookSubscriberTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/context-budget-wrap-up-system-reminders.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/context-budget-wrap-up-system-reminders.
- Pulled integration checkout: Merge made by the 'ort' strategy.
 bin/console                                       |  38 ++++++++
 src/CodingAgent/CLI/AgentCommand.php              |  31 +++---
 tests/CodingAgent/CLI/ConsoleEntrypointUxTest.php | 114 ++++++++++++++++++++++
 3 files changed, 170 insertions(+), 13 deletions(-)
 create mode 100644 tests/CodingAgent/CLI/ConsoleEntrypointUxTest.php.
- Validation: PR #327 MERGED at 2026-07-29T17:04:39Z; Final deterministic castor check passed; Reviewer APPROVED; Worktree clean before DONE transition
- Summary: PR #327 merged on GitHub at 293c6e81b91b2162e3433a9129e95d8d698c5510. Cleaned the sole authorized local worktree edit by restoring `.hatfield/settings.yaml` to merged HEAD (200000/25000). Final implementation and Symfony fork integration validated and approved.

## Task workflow update - 2026-07-29T17:17:18.900Z
- Validation: composer install synchronized symfony/tui to 130bc651f2e22919c6a4036f6fe8658cfd2df78c; castor test --filter=TuiTranscriptBlocksVirtualRenderTest: OK (31 tests, 104 assertions); castor test:llm-real --filter=ShellFollowUpLiveE2eTest: OK (2 tests, 21 assertions); Post-merge castor check: unit OK (4469 tests, 16219 assertions); controller replay OK (10 tests, 135 assertions); TUI OK (38 tests, 193 assertions); deptrac/phpstan/cs/cache/artifact/leak guards OK; one unrelated RewindBranchLiveE2eTest run.completed timing failure; castor test:llm-real --filter=RewindBranchLiveE2eTest: OK (1 test, 19 assertions)
- Summary: Post-merge validation on integration checkout: initial check used stale vendor and failed exact HtmlInline regression; ran Composer install to synchronize vendor to pinned ineersa/tui SHA, after which exact virtual regression passed. Subsequent full check passed test, controller replay, TUI, deptrac, phpstan, cs-check, cache/artifact/leak guards; only unrelated known RewindBranch live-LLM timing flake failed, and its focused rerun passed.

## Task workflow update - 2026-07-30T13:31:38.862Z
- Summary: Squashed the two Symfony monorepo feature commits into one byte-identical unsigned commit cb1ef7d08182648871bdbca4f6c2024cba75f932 and force-pushed with lease. Component fork and Agent Core pin remain unchanged at 130bc651f2e22919c6a4036f6fe8658cfd2df78c. Updated Hatfield issue #332 body and comment to replace retired monorepo SHA references.
