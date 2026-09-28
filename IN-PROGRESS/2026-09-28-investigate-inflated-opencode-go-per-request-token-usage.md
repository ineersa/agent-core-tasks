# Investigate inflated OpenCode Go per-request token usage

## Goal
Issue: https://github.com/ineersa/agent-core/issues/536

Investigate implausible per-request usage for opencode-go/deepseek-v4.1-flash in Hatfield v0.0.28. Individual llm_step_completed records report up to 112,456,170 input tokens and 3,192,057 output tokens. The corruption precedes TUI projection.

Compare Symfony AI 0.13 and 0.14 stream usage conversion and aggregation. Determine whether OpenCode Go sends cumulative snapshots, duplicated final usage, genuine deltas, or malformed values. Capture only sanitized usage and chunk ordering if provider evidence is available. Do not retain credentials or conversation content.

Initial scope is investigation and a proposed fix, not implementation. Do not hide the defect with a UI clamp. Keep this separate from the subscription /usage task.

## Acceptance criteria
- Identify the earliest confirmed source of inflated usage and distinguish evidence from hypotheses.
- Compare affected-build usage conversion with Symfony AI 0.13 and 0.14.
- Determine provider streaming semantics from sanitized evidence, or record the exact evidence still missing.
- Propose the smallest correction and regression coverage for per-request usage, cumulative totals, latest input, and context-limit decisions.

## Workflow metadata
Status: IN-PROGRESS
Branch: task/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage
Fork run:
PR URL:
PR Status:
Started: 2026-09-28T13:15:22+00:00
Completed:

## Work log
- Created: 2026-09-28T13:02:01+00:00

## Task workflow update - 2026-09-28T13:15:22+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-28-investigate-inflated-opencode-go-per-request-token-usage/.idea.
- Summary: Starting investigation worktree for user-led reproduction with OpenCode Go DeepSeek in parent and fork runs. No fix selected yet; cumulative stream-usage aggregation remains a hypothesis.
