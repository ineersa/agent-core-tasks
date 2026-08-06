# Fix process-transport tool filter propagation to controller

## Goal
## Problem

The default `agent` TUI transport is `process`. `AgentCommand` accepts `--tools` and `--tools-excluded` and applies them to the TUI process's local `ToolRegistry`, but `JsonlProcessAgentSessionClient::spawnProcess()` starts the actual controller as only `agent --controller --cwd=...` plus prompt-template arguments. It does not forward either tool-filter option.

The controller boots a fresh container/registry and therefore exposes the unfiltered tool set to the model. For example, `agent --tools-excluded=bash` can still include `bash` in the controller's provider-visible tools. This also affects `--tools` allowlists and combined allowlist/denylist behavior.

This was discovered while revising PR #352 (`2026-07-27-show-available-tools-in-session-export`): a real process-transport Tmux test launched with `--tools-excluded=bash`, but the durable post-processor `available_tools` snapshot still contained `bash`. Removing the misleading test flag was correct for that export task; propagation belongs here.

## Relevant code

- `src/CodingAgent/CLI/AgentCommand.php`: default `transport = 'process'`; `applyToolFilters()` mutates only the current process registry.
- `src/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClient.php`: `spawnProcess()` builds controller argv without `--tools` / `--tools-excluded`.

## Scope

Forward the effective CLI allowlist and denylist into every spawned/restarted process-transport controller so the controller's final provider-visible tool set matches the user's command. Preserve in-process behavior and existing unknown-tool diagnostics. Keep the fix limited to tool-filter propagation; do not introduce a generic arbitrary-CLI forwarding mechanism.

## Test thesis

A regression proof must fail on current main by exercising the real process-controller boundary—not just an in-process registry or argv unit test—and demonstrate that the controller/model-visible final tool set honors exclusions and allowlists. A replay-backed process/controller or minimal Tmux proof is appropriate; use the lowest layer that actually crosses `JsonlProcessAgentSessionClient` subprocess startup.

## Acceptance criteria
- With default `--transport=process`, `--tools-excluded=bash` removes `bash` from the controller's final provider-visible tool set.
- With default process transport, `--tools=<names>` restricts the controller's final provider-visible tool set to the requested allowlist.
- Combined allowlist and denylist semantics remain `(allowlist or all) minus exclusions`, matching in-process behavior.
- Tool filters survive controller restarts because every spawn receives the same effective filter arguments.
- Unknown tool names still fail with a clear diagnostic before useful model work begins; validation is not silently weakened.
- No unrelated CLI options are generically forwarded and no new user-facing setting is added.
- Automated regression crosses the real process-controller boundary and proves provider-visible membership/absence; in-process-only or argv-string-only tests are insufficient.
- Focused Castor validation includes the appropriate process/controller replay proof, static analysis, dependency boundaries, and style checks.

## Workflow metadata
Status: DONE
Branch: task/fix-process-transport-tool-filter-propagation
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-process-transport-tool-filter-propagation
Fork run: 1yes78eow5mm
PR URL: https://github.com/ineersa/agent-core/pull/360
PR Status: merged
Started: 2026-08-04T20:25:56.141Z
Completed: 2026-08-04T22:37:35.389Z

## Work log
- Created: 2026-08-02T18:15:12.016Z

## Task workflow update - 2026-08-04T20:25:56.141Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-process-transport-tool-filter-propagation.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-process-transport-tool-filter-propagation.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-process-transport-tool-filter-propagation.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-process-transport-tool-filter-propagation.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-process-transport-tool-filter-propagation.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-process-transport-tool-filter-propagation.
- Summary: Starting implementation. Scope is limited to forwarding effective `--tools` and `--tools-excluded` filters across every process-controller spawn/restart, preserving in-process behavior and existing diagnostics, with a real process/controller-boundary regression proof.

## Task workflow update - 2026-08-04T20:40:57.297Z
- Summary: Scouting found the filter must reach not only `agent --controller` but also fresh Messenger consumer containers where provider-visible tools are resolved. All controller spawns share `JsonlProcessAgentSessionClient::spawnProcess()`; all worker launches/recycles inherit controller environment through `ConsumerSupervisor::launch()`. Minimal implementation should use dedicated tool-filter runtime state/argv+internal env propagation, then reapply after extension loading. No generic CLI forwarding or settings.
- Production scout: naive controller-argv-only fix is incomplete because LLM workers boot independent ToolRegistry instances. Provider-visible snapshot comes from LlmPlatformAdapter in the LLM consumer.
- Test scout: smallest real boundary proof is a focused replay-backed TmuxHarness test based on TuiExportCommandE2eTest, reading canonical llm_step_completed.available_tools. One combined allowlist+denylist case can prove allow, exclude, and combined semantics without bloating TuiJourneyE2eTest.
- Restart proof should be supplemental at the process-client spawn seam; do not kill real controller children in Tmux just to prove a single shared spawn method preserves arguments.
- Existing MCP dynamic-name validation behavior is out of scope and must remain unchanged.

## Task workflow update - 2026-08-04T20:41:32.210Z
- Recorded fork run: m4yo3bi9ijqn
- Summary: Implementation delegated to fork m4yo3bi9ijqn in the task worktree. Required deliverables: minimal dedicated filter propagation through controller and Messenger workers, a real replay-backed TmuxHarness provider-visible regression, supplemental spawn/restart proof, focused Castor validation, and commit. No `castor check`/review/PR in task-start phase.

## Task workflow update - 2026-08-04T20:52:37.972Z
- Recorded fork run: m4yo3bi9ijqn
- Validation: `castor test --filter='JsonlProcessToolFilterOptionsTest|JsonlProcessPromptTemplateOptionsTest|ToolRegistryTest'` — OK, 59 tests / 183 assertions; `castor test:tui --filter=TuiProcessTransportToolFilterE2eTest` — OK, 1 test / 9 assertions; `castor test:controller-replay` — OK, 11 tests / 160 assertions; `castor test` — OK, 4438 tests / 16526 assertions; `castor deptrac` — 0 violations; `castor phpstan` — 0 errors; `castor cs-check` — clean after `castor cs-fix`; `castor check` — intentionally deferred to task-to-PR phase per workflow
- Summary: Implementation completed and committed as 2e3984f1e15064f137d456feffe686f3866be0b4. Dedicated ToolFilterRuntimeConfig carries canonical filters into every controller spawn via argv and internal env; ExtensionLoaderSubscriber reapplies them after extension registration in controller/Messenger worker containers. Real replay-backed Tmux test proves provider-visible available_tools is exactly ['read'] for combined `--tools=read,bash --tools-excluded=bash`. Worktree is clean. Task remains IN-PROGRESS pending task-to-PR review and required `castor check`.

## Task workflow update - 2026-08-04T21:10:10.438Z
- Summary: Task-to-PR review on 2e3984f1e returned APPROVE WITH SUGGESTIONS. Specification fidelity PASS, architecture PASS, correct real Tmux proof PASS, no blocking correctness issue. A narrow cleanup is being applied before final re-review: remove one implementation-mirroring getter test, simplify list parsing with stdlib while preserving dedup/order semantics, and correct restart/environment-inheritance comments that overstate the test/mechanism.
- Reviewer confirmed the Tmux test crosses parent TUI → Jsonl controller → LLM Messenger worker and asserts durable provider-visible available_tools exactly ['read'].
- Reviewer confirmed no AgentCore changes, no generic forwarding, no new user-facing setting, and consumer recycle receives inherited filter env.
- Deferred as out-of-scope: only manual controller invocations with intentionally disagreeing env and argv can produce config/registry state mismatch; normal production spawn always supplies identical values.

## Task workflow update - 2026-08-04T21:20:31.762Z
- Validation: `castor test` on HEAD 96169daaf — OK, 4437 tests / 16524 assertions; `castor test:tui` on HEAD 96169daaf — FAILED: TuiJourneyE2eTest expected inline `!ls`, but its own command excluded bash; this is the now-effective filter exposing a stale contradictory test flag; `castor clean:cleanup:workers:list` — no stale QA worker candidates
- Summary: Final local `castor test:tui` exposed one task-related test conflict: `TuiJourneyE2eTest` still launched with `--tools-excluded=bash` but later explicitly exercises `!ls`, which routes through the bash tool. With propagation now correctly fixed, the journey fails because bash is genuinely absent. No stale QA workers were found. A narrow test-fixture correction is required, then re-review and full validation will repeat.

## Task workflow update - 2026-08-04T21:28:43.916Z
- Validation: Reviewer final decision — APPROVED; no actionable findings; `castor test:tui --filter=TuiJourneyE2eTest` — OK, 1 test / 7 assertions; `castor test:tui` — OK, 33 tests / 217 assertions; `castor test --filter=SubagentLivePickerControllerTest` — OK, 17 tests / 99 assertions after unrelated transient parallel failure; `castor test --filter=ConsumerSupervisorTest` — OK, 5 tests / 38 assertions after unrelated transient parallel failure; Final `castor test` retry — OK, 4437 tests / 16524 assertions; `castor deptrac` — 0 violations / 0 errors; `castor phpstan` — 0 errors; `castor cs-check` — clean; `castor clean:cleanup:workers:list` after TUI failure — no stale QA worker candidates
- Summary: Final review on HEAD bb21a3e63790ecc3e82095c9e6cfea1dabe03ba5: APPROVED. Specification fidelity PASS, ponytail PASS, architecture/security PASS, and test proof PASS. Full TUI validation exposed and fixed one stale contradictory journey flag; final focused and full TUI lanes pass. Ready for deterministic CODE-REVIEW gate, push, and PR.

## Task workflow update - 2026-08-04T21:30:40.590Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (106.3s).
- Pushed task/fix-process-transport-tool-filter-propagation to origin.
- branch 'task/fix-process-transport-tool-filter-propagation' set up to track 'origin/task/fix-process-transport-tool-filter-propagation'.
- Created PR: https://github.com/ineersa/agent-core/pull/360

## Task workflow update - 2026-08-04T21:30:45.270Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/360
- Updated PR Status: open
- Validation: Deterministic `castor check` — PASSED in 106.3s
- Summary: Task-to-PR complete. Deterministic `castor check` passed in 106.3s, branch pushed, and PR #360 opened. Task is now CODE-REVIEW.

## Task workflow update - 2026-08-04T21:46:20.248Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Addressing PR #360 feedback. Actionable: remove redundant explicit DI registration; extract comma-separated trim/filter parsing into a shared static helper and reuse the existing duplicate in AgentDefinitionParser. Environment lookup has no current shared helper or matching duplicate; implementation fork will evaluate the smallest semantic static utility rather than introducing a broad generic bag of helpers. No behavior/public-surface changes.

## Task workflow update - 2026-08-04T21:46:51.662Z
- Recorded fork run: 1yes78eow5mm
- Summary: PR #360 feedback classified and delegated to fork 1yes78eow5mm. Changes are internal-only: remove redundant DI registration; extract exact environment lookup into EnvironmentVariableReader; extract trim/drop-empty comma parsing into CommaSeparatedListParser and reuse AgentDefinitionParser's existing duplicate while keeping tool-filter dedup caller-specific. No behavior/settings/AgentCore changes.
- PR comment r3716367250 — actionable: explicit ToolFilterRuntimeConfig service registration is redundant under the broad CodingAgent resource.
- PR comment r3716372851 — actionable per author preference: move exact three-source env lookup to a narrow static EnvironmentVariableReader; do not refactor unrelated env semantics.
- PR comment r3716380874 — actionable with real reuse: move trim/drop-empty comma parsing to CommaSeparatedListParser and use it from ToolFilterRuntimeConfig plus AgentDefinitionParser; dedup remains tool-filter-specific.

## Task workflow update - 2026-08-04T21:54:48.589Z
- Validation: Feedback-focused tests — OK, 84 tests / 222 assertions; Feedback commit `castor deptrac` — 0 violations; Feedback commit `castor phpstan` — 0 errors; Feedback commit `castor cs-check` — clean after cs-fix; Re-review — APPROVE WITH SUGGESTIONS; no actionable findings
- Summary: PR feedback implementation committed as df17957bf59bfc1a49521e44c47e6ba54c9729f1. Re-review returned APPROVE WITH SUGGESTIONS with no actionable issue: all three comments resolved, semantic equivalence verified, specification fidelity PASS, ponytail PASS, and existing process/Tmux proof remains adequate. Proceeding with final focused/full Castor validation before updating PR #360.

## Task workflow update - 2026-08-04T21:57:12.945Z
- Validation: Final `castor test` — OK, 4437 tests / 16524 assertions; Final `castor test:tui` — OK, 33 tests / 215 assertions; Final `castor deptrac` — 0 violations / 0 errors; Final `castor phpstan` — 0 errors; Final `castor cs-check` — clean
- Summary: PR feedback iteration ready for CODE-REVIEW. HEAD df17957bf is clean and one commit ahead of the existing PR branch. Full local validation passed; moving through deterministic gate and push next.

## Task workflow update - 2026-08-04T21:59:06.927Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (105.1s).
- Pushed task/fix-process-transport-tool-filter-propagation to origin.
- branch 'task/fix-process-transport-tool-filter-propagation' set up to track 'origin/task/fix-process-transport-tool-filter-propagation'.
- PR already exists: https://github.com/ineersa/agent-core/pull/360

## Task workflow update - 2026-08-04T21:59:26.935Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/360
- Updated PR Status: open
- Validation: Deterministic `castor check` after PR feedback — PASSED in 105.1s
- Summary: PR #360 feedback iteration complete. Commit df17957bf pushed after deterministic `castor check` passed in 105.1s. Replied to all three inline comments with the implemented resolution. Task returned to CODE-REVIEW.
- Replied to r3716367250: removed redundant explicit DI registration.
- Replied to r3716372851: extracted internal static EnvironmentVariableReader.
- Replied to r3716380874: extracted/reused internal CommaSeparatedListParser; tool-specific dedup remains local.

## Task workflow update - 2026-08-04T22:37:35.389Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-process-transport-tool-filter-propagation into integration checkout.
- Merge made by the 'ort' strategy.
 .../Agent/Definition/AgentDefinitionParser.php     |  28 +--
 src/CodingAgent/CLI/AgentCommand.php               |  51 +---
 .../Extension/ExtensionLoaderSubscriber.php        |  14 ++
 .../Process/JsonlProcessAgentSessionClient.php     |   7 +
 src/CodingAgent/Tool/ToolFilterRuntimeConfig.php   | 134 ++++++++++
 .../Utility/CommaSeparatedListParser.php           |  26 ++
 .../Utility/EnvironmentVariableReader.php          |  30 +++
 ...onlProcessAgentSessionClientEventBufferTest.php |   2 +
 ...nlProcessAgentSessionClientTransportDsnTest.php |   3 +
 .../JsonlProcessPromptTemplateOptionsTest.php      |   2 +
 .../JsonlProcessShellStandalonePayloadTest.php     |   2 +
 .../Process/JsonlProcessToolFilterOptionsTest.php  | 194 +++++++++++++++
 tests/Tui/E2E/TuiJourneyE2eTest.php                |   3 +-
 .../E2E/TuiProcessTransportToolFilterE2eTest.php   | 270 +++++++++++++++++++++
 14 files changed, 695 insertions(+), 71 deletions(-)
 create mode 100644 src/CodingAgent/Tool/ToolFilterRuntimeConfig.php
 create mode 100644 src/CodingAgent/Utility/CommaSeparatedListParser.php
 create mode 100644 src/CodingAgent/Utility/EnvironmentVariableReader.php
 create mode 100644 tests/CodingAgent/Runtime/Process/JsonlProcessToolFilterOptionsTest.php
 create mode 100644 tests/Tui/E2E/TuiProcessTransportToolFilterE2eTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-process-transport-tool-filter-propagation.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-process-transport-tool-filter-propagation.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #360 confirmed merged on GitHub at 2026-08-04T22:34:21Z with merge commit 12acb43f11918e83143d2fe3737cdf530e32327b. Completing task, synchronizing integration checkout, and cleaning the task worktree.

## Task workflow update - 2026-08-04T22:39:46.028Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/360
- Updated PR Status: merged
- Validation: Post-merge `LLM_MODE=true castor check` — quality OK in 332.0s; Unit/integration lane — 4436 tests / 16526 assertions; Controller replay lane — 11 tests / 160 assertions; TUI replay lane — 34 tests / 218 assertions; Live LLM lane — 13 tests / 144 assertions; Deptrac, PHPStan, CS check — all OK; QA artifact integrity — OK; QA process/tmux leak check — OK; llama-proxy cache guard — OK, entries 223 → 223
- Summary: Task complete. PR #360 merged (12acb43f11918e83143d2fe3737cdf530e32327b), integration checkout synchronized, task worktree and IDEA exclusions removed, and post-merge full quality gate passed. Integration checkout is clean.
