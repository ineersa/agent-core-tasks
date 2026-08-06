# Make MCP tool loading opt-in per-session instead of always loaded

## Goal
Currently ALL connected MCP server tool schemas are injected into every LLM request for every session. This burns significant context budget on every turn regardless of whether the tools are actually needed.

In a recent session analysis (Qwen3.6-27B), the breakdown was:
- jetbrains-index: 18 tools, ~9.7K tokens
- context7: 2 tools, ~1.1K tokens
- websearch: 3 tools, ~700 tokens
- Total MCP: ~11K tokens on every single turn

The `websearch` tools are always available even though they're a specialized research tool that should only be loaded when the user needs web research capability. Same for `context7` (documentation lookup) and `jetbrains-index` (IDE semantic navigation).

**Goal**: MCP tools should be loaded selectively, not blanket-injected into every session. Options:
- Settings-based allowlist per session/project
- Opt-in via slash command or settings toggle
- Only load MCP tools when the session type warrants it (e.g., research sessions get websearch, coding sessions get jetbrains-index)
- Lazy loading on demand

The model should only see the tool schemas for servers that are relevant to the current context.

## Acceptance criteria
- MCP tool schemas are not injected into every LLM request by default
- There is a mechanism to selectively enable/disable MCP servers per session or project
- Context budget is reduced when unused MCP servers are disabled
- Existing sessions that need MCP tools can still access them when enabled

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-07-27-make-mcp-tool-loading-opt-in-per-session-instead-of-always-loaded
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-27-make-mcp-tool-loading-opt-in-per-session-instead-of-always-loaded
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/356
PR Status: merged
Started: 2026-08-04T17:49:57.033Z
Completed: 2026-08-04T19:47:40.255Z

## Work log
- Created: 2026-07-27T13:49:24+00:00

## Task workflow update - 2026-08-04T17:49:57.033Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-07-27-make-mcp-tool-loading-opt-in-per-session-instead-of-always-loaded.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-07-27-make-mcp-tool-loading-opt-in-per-session-instead-of-always-loaded.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-07-27-make-mcp-tool-loading-opt-in-per-session-instead-of-always-loaded.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-07-27-make-mcp-tool-loading-opt-in-per-session-instead-of-always-loaded.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-07-27-make-mcp-tool-loading-opt-in-per-session-instead-of-always-loaded.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-07-27-make-mcp-tool-loading-opt-in-per-session-instead-of-always-loaded.
- Summary: Claimed for manual reproduction. User will run an A/B parent-session probe in the task worktree, comparing discovered MCP tools in `mcp-tools.json` against provider-visible `llm_step_completed.payload.available_tools` for `availability: specific` versus `all`. No implementation is planned until the historical behavior is reproduced or ruled out.

## Task workflow update - 2026-08-04T19:30:49.239Z
- Validation: Read-only inspection of `.hatfield/sessions/1/events.jsonl`, `mcp-tools.json`, child artifacts, effective `.hatfield/mcp.json`, and runtime log; no QA commands run.; Parent: 37 provider-visible tools (17 native + 20 MCP), schema estimate 14,045 tokens on each of 15 turns; websearch filtered.; Scout children: 23 tools each (3 native + 20 MCP), schema estimate 11,220 tokens; websearch filtered.; Researcher child: 25 tools (2 native + all 23 MCP), schema estimate 11,870 tokens; websearch included by explicit selector.; Worktree branch HEAD e83dadae0f76fcc822889c7b806d294823c2d63f; tracked/staged state clean and MCP/settings config identical to HEAD.
- Summary: Manual worktree session 1 forensic: all 23 MCP tools were discovered/registered, but provider-visible filtering matched policy. Parent and both scout children received the 20 `availability: all` tools from context7 (2) and jetbrains-index (18); all three `availability: specific` websearch tools were absent. The researcher child explicitly selected `mcp:websearch_*` and received all 23 MCP tools. No MCP availability leak was found. The high repeated schema budget is explained by context7 and jetbrains-index being configured `all`, not by `specific` filtering failure.

## Task workflow update - 2026-08-04T19:39:44.347Z
- Summary: Scope clarified by user after manual verification: no MCP filtering/runtime implementation is needed. The only requested code change is to remove the `context7` server definition from the tracked project `.hatfield/mcp.json`; retain `websearch` and `jetbrains-index` unchanged. This explicit clarification supersedes the original per-session opt-in scope.

## Task workflow update - 2026-08-04T19:41:08.196Z
- Validation: Parent verified commit and exact diff: `.hatfield/mcp.json | 8 deletions`; only the `context7` block was removed.; Fork reported successful JSON parsing after the edit. No Castor QA suite was run because this is a trivial data-only configuration deletion.
- Summary: Implementation complete at commit `bd77bdbadb17a207b0d49bdceeacd48f00114786`: removed only the `context7` server block from project `.hatfield/mcp.json`. `websearch` remains `specific`; `jetbrains-index` remains `all`. No runtime, documentation, test, settings, or session-artifact changes. Worktree is clean.

## Task workflow update - 2026-08-04T19:46:54.221Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (99.5s).
- Pushed task/2026-07-27-make-mcp-tool-loading-opt-in-per-session-instead-of-always-loaded to origin.
- branch 'task/2026-07-27-make-mcp-tool-loading-opt-in-per-session-instead-of-always-loaded' set up to track 'origin/task/2026-07-27-make-mcp-tool-loading-opt-in-per-session-instead-of-always-loaded'.
- Created PR: https://github.com/ineersa/agent-core/pull/356
- Validation: `castor test`: OK — 4,424 tests, 16,426 assertions.; `castor deptrac`: OK — 0 violations, 0 errors.; `castor phpstan`: OK — 0 errors, 0 file errors.; `castor cs-check`: clean — 0 files fixed.; `castor test:llm-real`: OK — 13 tests, 144 assertions.; Worktree clean; exact committed diff is `.hatfield/mcp.json` with 8 deletions removing only `context7`.; Reviewer subagent skipped by explicit user instruction.
- Summary: Ready for CODE-REVIEW at `bd77bdbadb17a207b0d49bdceeacd48f00114786`. Final scope is the user-requested removal of project `context7` MCP configuration after manual verification showed availability filtering itself works. Reviewer subagent intentionally skipped at the user's explicit request.

## Task workflow update - 2026-08-04T19:47:40.255Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-07-27-make-mcp-tool-loading-opt-in-per-session-instead-of-always-loaded into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/mcp.json | 8 --------
 1 file changed, 8 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-07-27-make-mcp-tool-loading-opt-in-per-session-instead-of-always-loaded.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-07-27-make-mcp-tool-loading-opt-in-per-session-instead-of-always-loaded.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #356 confirmed merged on GitHub at 2026-08-04T19:47:14Z (merge commit `2c7e4e741a65aac588a44adec8b6f2f1e57b576f`). Completing task and integrating the removal of project `context7` MCP configuration.

## Task workflow update - 2026-08-04T19:49:49.159Z
- Validation: Post-merge `LLM_MODE=true castor check`: quality OK. Deptrac, 4,424-test unit/integration lane, 11 controller-replay tests, 32 TUI tests, 13 live-LLM tests, PHPStan, and CS all passed.; QA artifact integrity passed; no HATFIELD_QA_RUN_ID process/tmux leaks; llama-proxy cache stable at 223 entries.; Integration checkout clean at `be4389db2`; `origin/main` contains GitHub merge commit `2c7e4e741`.
- Summary: Post-merge integration complete. PR #356 is merged, task worktree and IDEA exclusions were removed, and the integration checkout contains the context7 removal. Integration checkout is clean.

## Task workflow update - 2026-08-06T20:58:47.443Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
