# Fix assistant text rendering before thinking (RunPod provider) causing mid-transcript flicker

## Goal
Reported during manual smoke testing (session 1, worktree use-symfony-ai-native-tool-argument-resolution-and-validation, RunPod provider).

## Symptom
With the RunPod provider, the assistant turn in the TUI transcript shows the **assistant text content before the thinking/reasoning block** of the same turn. The thinking then streams in *above* the already-rendered text, so the assistant response stream is not appended at the tail of the transcript — it sits mid-transcript and the UI flickers as tokens arrive.

## Expected
Within one assistant turn the content order is stable (thinking/reasoning first, then text), and streamed assistant output is always appended at the tail of the transcript (pending area), not injected in the middle.

## Repro
Run a session with the RunPod provider and any assistant turn that produces both thinking and text. Observe the transcript order while the response streams.

## Investigation notes (unverified, from smoke session)
- Likely a provider-specific content-part ordering issue: the RunPod stream/adapter may emit (or the Platform bridge may map) reasoning content so that text blocks land before the thinking block in the assistant message, or thinking deltas arrive after text has started.
- Suspected layers: `src/Platform` provider bridge/stream adapter (RunPod/OpenAI-compatible), LLM stream event mapping in AgentCore, and/or TUI transcript/pending projection (TranscriptProjector / RuntimeEventPoller / pending streaming).
- Compare against another provider (e.g. direct OpenAI-compatible) to confirm the ordering delta is provider-specific.
- TUI/runtime changes require the testing-skill workflow; TUI proof at the lowest correct layer; `castor check` if TUI/runtime/LLM-visible flow is touched.

## Acceptance criteria
- With the RunPod provider, assistant turns render thinking before text and streamed output appends at the transcript tail with no mid-transcript flicker
- Behavior verified at the lowest correct test layer (virtual/castor test → controller-replay, minimal castor test:tui only if required)
- No regression in transcript ordering for other providers (replay coverage)
- castor check passes if TUI/runtime/LLM-visible flow is touched

## Workflow metadata
Status: DONE
Branch: task/fix-runpod-assistant-text-before-thinking-ordering
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-runpod-assistant-text-before-thinking-ordering
Fork run: f62t4uhb62tz
PR URL: https://github.com/ineersa/agent-core/pull/405
PR Status: merged
Started: 2026-08-18T00:31:36.033Z
Completed: 2026-08-18T01:44:16.003Z

## Work log
- Created: 2026-08-17T16:25:27+00:00

## Task workflow update - 2026-08-18T00:31:36.033Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-runpod-assistant-text-before-thinking-ordering.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-runpod-assistant-text-before-thinking-ordering.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-runpod-assistant-text-before-thinking-ordering.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-runpod-assistant-text-before-thinking-ordering.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-runpod-assistant-text-before-thinking-ordering.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-runpod-assistant-text-before-thinking-ordering.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/fix-runpod-assistant-text-before-thinking-ordering/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/fix-runpod-assistant-text-before-thinking-ordering.

## Task workflow update - 2026-08-18T00:34:42.994Z
- Recorded fork run: f62t4uhb62tz
- Validation: castor test --filter=DurableResultConverterTest — OK 29 tests, 136 assertions (includes new test); Mutation check: stash prod fix → new test FAILS as expected → stash pop restored; castor cs-check --path on both files — clean; php -l both files — OK; NOT yet run: full castor check / test:controller-replay (CODE-REVIEW transition gate runs these); optional live re-capture with RunPod
- Summary: Implementation complete. Root cause verified live via HATFIELD_LLM_RAW_STREAM_CAPTURE: vLLM's role chunk carries explicit empty `content`, DurableResultConverter's isset-only guard yielded TextDelta('') before thinking, AssistantTextStreamSubscriber fired text_started early → text block before thinking block (live vs replay order desync = the flicker). Fix: skip empty content deltas in DurableResultConverter (mirrors reasoning branch guard). Test: roleChunkWithEmptyContentDoesNotEmitTextDeltaBeforeThinking (mutation-verified to fail without fix). Commit f83b9e1c5 on task/fix-runpod-assistant-text-before-thinking-ordering.

## Task workflow update - 2026-08-18T00:43:19.088Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (167.7s).
- Pushed task/fix-runpod-assistant-text-before-thinking-ordering to origin.
- branch 'task/fix-runpod-assistant-text-before-thinking-ordering' set up to track 'origin/task/fix-runpod-assistant-text-before-thinking-ordering'.
- Created PR: https://github.com/ineersa/agent-core/pull/405

## Task workflow update - 2026-08-18T00:43:29.598Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/405
- Updated PR Status: open
- Validation: castor check in worktree — PASSED (167.7s) on retry; first attempt failed only on phpstan lane timeout (cold cache in fresh worktree), phpstan standalone: 0 errors in 62s

## Task workflow update - 2026-08-18T01:44:16.003Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/fix-runpod-assistant-text-before-thinking-ordering.
- Merged task/fix-runpod-assistant-text-before-thinking-ordering into integration checkout.
- Merge made by the 'ort' strategy.
 .../Bridge/Generic/DurableResultConverter.php      |  6 +++-
 .../Bridge/Generic/DurableResultConverterTest.php  | 42 ++++++++++++++++++++++
 2 files changed, 47 insertions(+), 1 deletion(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-runpod-assistant-text-before-thinking-ordering.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-runpod-assistant-text-before-thinking-ordering.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-08-18T01:44:53.026Z
- Updated PR Status: merged
- Validation: PR #405 approved and merged by user; task branch merged into integration checkout, worktree cleaned up
