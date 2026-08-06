# Upstream Symfony AI streamed finish_reason metadata

## Goal
Context:

- Symfony AI issue: https://github.com/symfony/ai/issues/2194 (`[Platform][Generic] Streamed finish_reason (stop/length/content_filter) is never surfaced to consumers`).
- User created local Symfony AI branch `/home/ineersa/projects/ai` `finish-reason-capture`. Observed at task creation: branch `finish-reason-capture` at `56eeda68`, same as `main`, with only untracked local IDE/demo directories.
- Scout recon on 2026-07-05 found the issue is still present in Symfony AI Generic completions streaming: `src/platform/src/Bridge/Generic/Completions/CompletionsConversionTrait.php` reduces `finish_reason` to a boolean for `IncompleteStreamException` guard and never yields/stores the value.
- Existing Symfony AI API already has `src/platform/src/Result/Stream/Delta/MetadataDelta.php` and `src/platform/src/Metadata/StreamListener.php`; Perplexity uses `MetadataDelta` for stream metadata. Smallest upstream fix appears to be yielding `MetadataDelta('finish_reason', $finishReason)` when terminal chunks provide `finish_reason`.
- Agent-core scout found `src/Platform/Bridge/Generic/DurableResultConverter.php` also captures `$finishReason` but only sends it to raw capture (`capture_end`), not stream metadata. `src/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapter.php` currently resolves stop reason as `tool_call` or `null`, so downstream cannot distinguish `length`/`content_filter` from clean text completion.
- Agent-core replay fixture machinery already maps fixture `stop_reason: length` to SSE `finish_reason: length` in `tests/CodingAgent/Runtime/Controller/E2E/Replay/ControllerReplayHttpClientFactory.php`.
- User specifically suggested a real LLM reproduction with very small max tokens against llama.cpp and an agent-core real-LLM test if practical.

Constraints:
- Load `task-workflow` before task phases/fork instructions.
- Load `testing` and read `tests/AGENTS.md` before any tests/QA or runtime/TUI/Messenger changes.
- Main agent remains orchestrator; implementation edits go through forks.
- In agent-core, all QA must go through Castor.
- In Symfony AI, follow `/home/ineersa/projects/ai/CLAUDE.md`: root `vendor/bin/php-cs-fixer fix src/platform/`, platform `vendor/bin/phpstan analyse`, and focused PHPUnit.
- Do not create a GitHub PR. User will inspect/open PR later. Local Symfony AI commit expected; push only if explicitly requested.
- Be careful with agent-core validation: production currently uses `DurableResultConverter`, so upstream vendor `MetadataDelta` behavior may need a temporary factory bypass for proof, or a local durable converter update if the user chooses to propagate the behavior in agent-core before upstream merge.

## Acceptance criteria
- Scout finding recorded: issue #2194 is still reproducible/present on Symfony AI `finish-reason-capture` branch before changes.
- Symfony AI branch `finish-reason-capture` has a local commit surfacing streamed Generic completions `finish_reason` via existing stream metadata mechanism, with tests covering at least `length` and preserving existing incomplete-stream guard/tool-call behavior.
- Symfony AI validation is recorded: php-cs-fixer on `src/platform/`, platform PHPStan (including whether any failures are pre-existing on `main`), and focused PHPUnit for changed tests.
- Agent-core validation links local Symfony AI branch into an agent-core task worktree and proves the upstream behavior in the agent-core dependency context; if `DurableResultConverter` bypass or local temporary changes are used, they are documented and reverted unless explicitly approved.
- A focused real-LLM or replay-backed agent-core test strategy is evaluated. If practical within scope, run a focused Castor validation (for live: `castor test:llm-real --filter=...`) with small max tokens to demonstrate `finish_reason: length`; otherwise document the blocker and recommended follow-up.
- No GitHub PR is created by agents; task records local commit hash, push status, validation results, and final worktree status.

## Workflow metadata
Status: DONE
Branch: task/upstream-symfony-ai-streamed-finish-reason-metadata
Worktree: /home/ineersa/projects/agent-core-worktrees/upstream-symfony-ai-streamed-finish-reason-metadata
Fork run: dmy1xwp5s251
PR URL: https://github.com/symfony/ai/pull/2276
PR Status: merged
Started: 2026-07-05T21:01:38.574Z
Completed: 2026-07-05T22:38:26.807Z

## Work log
- Created: 2026-07-05T21:01:29.523Z

## Task workflow update - 2026-07-05T21:01:38.574Z
- Moved TODO → IN-PROGRESS.
- Created branch task/upstream-symfony-ai-streamed-finish-reason-metadata.
- Created worktree /home/ineersa/projects/agent-core-worktrees/upstream-symfony-ai-streamed-finish-reason-metadata.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/upstream-symfony-ai-streamed-finish-reason-metadata.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/upstream-symfony-ai-streamed-finish-reason-metadata.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/upstream-symfony-ai-streamed-finish-reason-metadata.
- Summary: Starting per user request after scout recon confirmed Symfony AI issue #2194 is still present on branch `finish-reason-capture`. Goal: implement upstream finish_reason metadata capture, link into agent-core for validation, and explore focused real-LLM/replay proof with small max tokens where practical. No PR creation.

## Task workflow update - 2026-07-05T21:02:56.998Z
- Recorded fork run: afexk53yrhjy
- Summary: Scout recon completed before implementation: issue #2194 is still present on Symfony AI branch `finish-reason-capture` at `56eeda68`. Upstream `CompletionsConversionTrait::convertStream()` records finish_reason only as a boolean and drops the value. Existing Symfony AI `MetadataDelta` + `Metadata\StreamListener` appears to be the intended mechanism for surfacing stream metadata; Perplexity already uses it. Agent-core scout found `DurableResultConverter` similarly captures finish_reason only for debug capture, and `LlmPlatformAdapter` currently maps stop reason to `tool_call` or null, so upstream fix alone will not change agent-core production while DurableResultConverter remains wired unless agent-core is also updated later.
- Started implementation/validation fork afexk53yrhjy in `/home/ineersa/projects/ai` on local branch `finish-reason-capture`. Fork instructions: implement upstream finish_reason MetadataDelta behavior, add focused upstream tests, run Symfony AI php-cs-fixer/PHPStan/PHPUnit, commit locally without PR/push, link into agent-core streamed-finish-reason worktree, try focused agent-core validation including a real `llm-real` small-max-token proof if practical, then revert temporary agent-core edits and report final status.

## Task workflow update - 2026-07-05T21:09:21.837Z
- Recorded fork run: afexk53yrhjy
- Validation: Symfony AI: `cd /home/ineersa/projects/ai/src/platform && vendor/bin/phpunit src/Bridge/Generic/Tests/Completions/ResultConverterTest.php` → OK (30 tests, 101 assertions).; Symfony AI: `cd /home/ineersa/projects/ai && vendor/bin/php-cs-fixer fix src/platform/` → OK; fixed `ResultConverterTest.php` before commit.; Symfony AI: `cd /home/ineersa/projects/ai/src/platform && vendor/bin/phpstan analyse` → FAILED with one pre-existing `TypeInfoDescriber.php:92 generics.notGeneric` error; not introduced by #2194 changes.; Agent-core linked validation: `castor test --filter=GenericFinishReasonMetadataLinkedTest` with temporary test file → OK (3 tests, 8 assertions). Proved linked vendor `ResultConverter` exposes finish_reason metadata and production `DurableResultConverter` does not.; Agent-core linked live attempt: `castor test:llm-real --filter=GenericFinishReasonLlmRealLinkedTest` with temporary test file → FAILED before assertion with `BadRequestException: 'messages' is required` from provider response. Not treated as #2194 regression; live small-max-token proof remains unproven.; Fork reported it read Symfony AI `CLAUDE.md`, agent-core testing skill, and `tests/AGENTS.md` before QA.; Final agent-core state: temporary tests removed; streamed-finish-reason worktree clean.
- Summary: Fork afexk53yrhjy completed. Symfony AI issue #2194 is implemented locally in `/home/ineersa/projects/ai` on branch `finish-reason-capture` with commit `34923550` (`fix(platform): expose streamed finish_reason as metadata`). Changed upstream files: `src/platform/src/Bridge/Generic/Completions/CompletionsConversionTrait.php` and `src/platform/src/Bridge/Generic/Tests/Completions/ResultConverterTest.php`. Behavior: Generic completions streaming now yields `MetadataDelta('finish_reason', <value>)` for any terminal chunk with non-null `choices[0].finish_reason`; existing `Metadata\StreamListener`/`DeferredResult` promotion makes `finish_reason` available in result metadata after stream consumption. Scope preserved the existing incomplete-stream guard and tool-call completion behavior. No PR and no push. Agent-core linked deterministic proof passed for the vendor Generic `ResultConverter` + `DeferredResult`, and also confirmed production `DurableResultConverter` still does not emit finish_reason metadata. Temporary agent-core tests were removed and streamed-finish-reason worktree is clean. Parent verification: `/home/ineersa/projects/ai` is on `finish-reason-capture` at `34923550` with no tracked dirty changes; untracked local dirs plus `diff_files.txt` remain in that repo. Agent-core streamed-finish-reason worktree `git status -sb` is clean.

## Task workflow update - 2026-07-05T21:25:17.203Z
- Recorded fork run: dmy1xwp5s251
- Validation: Direct provider curl to `http://192.168.2.38:9052/v1/chat/completions` with `max_tokens:1` returned SSE terminal chunk with `finish_reason:"length"`.; Agent-core temporary live smoke command: `castor test:llm-real --filter=FinishReasonLengthLiveSmokeTest` final run had 5 assertions OK and capture proof of `finish_reason=length`, but Castor exited 1 because PHPUnit marked the temporary test risky for not removing its own exception handlers. No assertion failure.; Earlier temp iterations hit namespace and empty-message issues; final working pattern required explicit `messages` on `ModelInvocationInput` when using the DI `PlatformInterface`.; Final state verified by parent: `/home/ineersa/projects/agent-core-worktrees/upstream-symfony-ai-streamed-finish-reason-metadata` `git status -sb` is clean.
- Summary: Validation-only fork dmy1xwp5s251 completed an actual-provider smoke through the agent-core production DI path. Temporary `#[Group('llm-real')]` test used container `PlatformInterface` (`LlmPlatformAdapter` → `SymfonyAiProviderFactory` → `DurableResultConverter`) with `HATFIELD_LLM_RAW_STREAM_CAPTURE=1`, explicit `ModelInvocationInput::messages`, and `extraOptions['max_tokens']=1` against live `llama_cpp_test/test` on port 9052. Result: capture JSONL contained a raw chunk with `choices[0].finish_reason === "length"` and `capture_end.stop_reason === "length"`. As expected, `PlatformInvocationResult->stopReason` remained `null` because agent-core does not yet map finish_reason metadata/capture into stopReason. Temporary test was deleted; streamed-finish-reason worktree remains clean. Symfony AI remains at commit `34923550`; no push/PR by agents.

## Task workflow update - 2026-07-05T21:29:03.967Z
- Summary: User decided not to wire agent-core `DurableResultConverter`/`LlmPlatformAdapter` in this task after reviewing the upstream Symfony AI PR/change; the PR change is focused and considered sufficient for now. Agent-core production propagation of `finish_reason`/`stop_reason=length` remains a later follow-up after upstream merge.

## Task workflow update - 2026-07-05T22:04:11.852Z
- Updated PR URL: https://github.com/symfony/ai/pull/2276
- Validation: Symfony AI: `cd /home/ineersa/projects/ai/src/platform && vendor/bin/phpunit src/Bridge/DeepSeek/Tests/ResultConverterTest.php` → OK (11 tests, 39 assertions); Symfony AI: `cd /home/ineersa/projects/ai/src/platform && vendor/bin/phpunit src/Bridge/Generic/Tests/Completions/ResultConverterTest.php` → OK (30 tests, 101 assertions); Symfony AI: `cd /home/ineersa/projects/ai && vendor/bin/php-cs-fixer fix src/platform/src/Bridge/DeepSeek/Tests/ResultConverterTest.php --diff` → OK
- Summary: Symfony AI PR #2276 CI DeepSeek failure was addressed with follow-up commit 752f2ae7 (`test(platform): expect finish_reason metadata in DeepSeek stream`) on branch `finish-reason-capture`, pushed to origin. The stale DeepSeek test now expects the third streamed chunk to be `MetadataDelta('finish_reason', 'stop')`. No force-push/history rewrite; untracked local files preserved.

## Task workflow update - 2026-07-05T22:38:26.808Z
- Moved IN-PROGRESS → DONE.
- Merged task/upstream-symfony-ai-streamed-finish-reason-metadata into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/upstream-symfony-ai-streamed-finish-reason-metadata.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/upstream-symfony-ai-streamed-finish-reason-metadata.
- Deleted branch task/upstream-symfony-ai-streamed-finish-reason-metadata.
- Pulled integration checkout: Already up to date..
- Validation: Symfony AI Generic ResultConverter PHPUnit: OK (30 tests, 101 assertions).; Symfony AI DeepSeek ResultConverter PHPUnit after CI fix: OK (11 tests, 39 assertions).; Symfony AI php-cs-fixer: OK.; Symfony AI platform PHPStan: one pre-existing `TypeInfoDescriber.php:92 generics.notGeneric` error unrelated to #2194.; Agent-core linked deterministic proof: temporary `GenericFinishReasonMetadataLinkedTest` OK (3 tests, 8 assertions), proving vendor ResultConverter + DeferredResult exposes finish_reason metadata and production DurableResultConverter does not yet emit it.; Agent-core live actual-provider smoke through production DI with raw stream capture: assertions OK for provider `finish_reason=length` and capture_end stop_reason=length; PlatformInvocationResult->stopReason remained null as expected/deferred. Castor exited 1 because the temporary test was risky for exception-handler cleanup, not assertion failure.; Cleanup validation: `git status -sb` clean; no remaining linked `vendor/symfony` symlinks.
- Summary: Closed as upstream-only work. Symfony AI branch `finish-reason-capture` implemented issue #2194 with commit `34923550` (`fix(platform): expose streamed finish_reason as metadata`) plus follow-up commit `752f2ae7` (`test(platform): expect finish_reason metadata in DeepSeek stream`) for PR CI. Upstream PR is https://github.com/symfony/ai/pull/2276. Agent-core source remains intentionally unchanged; production propagation from DurableResultConverter/LlmPlatformAdapter was explicitly deferred. Cleanup: rolled back Symfony AI vendor symlinks in the task worktree with `/home/ineersa/projects/ai/link --rollback`, regenerated Composer autoload, confirmed no remaining `vendor/symfony` symlinks and clean worktree before closing.
