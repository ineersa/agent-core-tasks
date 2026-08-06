# QH-05 AgentCore interrupt compatibility for ask_human

## Goal
Plan: .pi/plans/tui-question-hitl-plan.md

**V1 decision (2026-06-28):** Slim scope — ToolCallExtractor payload preservation only. ToolExecutor interrupt-compatibility is folded into QH-04 (QH-04 adds the defensive `ask_human` fallback in ToolExecutor alongside existing `ask_user`).

Scope:
- Ensure ToolCallExtractor::interruptPayloadFromToolResult() preserves header, ui_kind/kind, choices, default, allow_other, and secret where available.

Exclusions:
- Do not add a new blocking tool execution path.
- Do not implement TUI question widgets or answer routing.
- Do not replace existing WaitingHuman/HumanResponse flow.
- ToolExecutor interrupt-compatibility is already covered by QH-04.

Dependencies: QH-04.
Parallelizable with: QH-02, QH-03.

## Acceptance criteria
- Interrupt payload preserves UI metadata (header, ui_kind, choices, default, allow_other, secret) needed by runtime/TUI projection.
- No new blocking/oneshot tool execution path is introduced.
- castor deptrac passes.

## Workflow metadata
Status: DONE
Branch: task/qh-05-ask-human-agentcore-compatibility
Worktree: /home/ineersa/projects/agent-core-worktrees/qh-05-ask-human-agentcore-compatibility
Fork run: 932rx6l654rj
PR URL: https://github.com/ineersa/agent-core/pull/232
PR Status: merged
Started: 2026-06-29T00:22:24.110Z
Completed: 2026-06-29T00:49:33.937Z

## Work log
- Created: 2026-05-18T00:04:34.543Z

## Task workflow update - 2026-06-29T00:22:24.110Z
- Moved TODO → IN-PROGRESS.
- Created branch task/qh-05-ask-human-agentcore-compatibility.
- Created worktree /home/ineersa/projects/agent-core-worktrees/qh-05-ask-human-agentcore-compatibility.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/qh-05-ask-human-agentcore-compatibility.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/qh-05-ask-human-agentcore-compatibility.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/qh-05-ask-human-agentcore-compatibility.

## Task workflow update - 2026-06-29T00:31:59.349Z
- Recorded fork run: ny9o32mmnum7
- Validation: castor test --filter ToolCallExtractor: 10 tests, 36 assertions OK (377ms); castor deptrac: 0 violations, 0 errors; castor phpstan: 0 errors, 0 file_errors; castor cs-check: 0 files fixed; git diff --stat origin/main...HEAD: 2 files changed (+271/-10): ToolCallExtractor.php, ToolCallExtractorTest.php; All validations independently re-run by orchestrator — confirmed fork's reported results.
- Summary: QH-05 implemented via fork ny9o32mmnum7 (commit 0cf1c8cf3, 2 files changed: +271/-10). ToolCallExtractor::interruptPayloadFromToolResult() rewritten with generic passthrough: payload starts from full interrupt array ($payload = $interrupt), then 5 typed core-field fallbacks applied on top (tool_call_id, tool_name, question_id, prompt, schema). Removed blanket array_filter so default=>null survives. AgentCore enumerates zero tool-specific field names — consistent with QH-04 architecture principle. New ToolCallExtractorTest.php (10 tests, 36 assertions) is the first focused test for this path (previously zero interrupt coverage). Main checkout verified clean post-fork.

## Task workflow update - 2026-06-29T00:45:20.122Z
- Recorded fork run: 932rx6l654rj
- Validation: castor test --filter ToolCallExtractor: 11 tests, 40 assertions OK (380ms); castor test --suite=agent-core: 516 tests, 2211 assertions OK (2.5s); castor deptrac: 0 violations, 0 errors; castor phpstan: 0 errors, 0 file_errors; castor cs-check: 0 files fixed; Reviewer verdict: APPROVED (re-review of c2b1ecc3d); Production file rg for tool-specific refs (AskHuman/ask_human/ask_user/ui_kind/allow_other/secret/header/choices): none found — fully tool-agnostic; Pure AgentCore change — no test:tui / test:llm-real required
- Summary: QH-05 reviewer APPROVE WITH SUGGESTIONS addressed via fork 932rx6l654rj (commit c2b1ecc3d, 2 files, +33/-8). All 6 findings fixed: (1) dropped AskHumanPayloadFactory class name from AgentCore comment, (2-3) removed tool-specific field enumerations from docblock + inline comment, (4) added kind='interrupt' survival assertion, (5) new testToolNamePassthroughWhenOuterResultLacksIt proves tool_name from interrupt survives when outer result lacks it, (6) added tool_name absence assertion in fallback test. Production diff verified comment-only (byte-identical when stripped). Reviewer re-review returned APPROVED. Residual test docblock reference to AskHumanPayloadFactory judged acceptable (no code dependency, descriptive context for simulated shape) — left as-is.

## Task workflow update - 2026-06-29T00:46:38.717Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (58.9s).
- Pushed task/qh-05-ask-human-agentcore-compatibility to origin.
- branch 'task/qh-05-ask-human-agentcore-compatibility' set up to track 'origin/task/qh-05-ask-human-agentcore-compatibility'.
- Created PR: https://github.com/ineersa/agent-core/pull/232

## Task workflow update - 2026-06-29T00:46:48.556Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/232
- Updated PR Status: open
- Summary: Moved to CODE-REVIEW. Deterministic castor check passed in worktree (58.9s). Branch task/qh-05-ask-human-agentcore-compatibility pushed to origin. PR #232 created: https://github.com/ineersa/agent-core/pull/232

## Task workflow update - 2026-06-29T00:49:33.937Z
- Moved CODE-REVIEW → DONE.
- Merged task/qh-05-ask-human-agentcore-compatibility into integration checkout.
- Merge made by the 'ort' strategy.
 .../Application/Pipeline/ToolCallExtractor.php     |  51 ++++-
 .../Application/Pipeline/ToolCallExtractorTest.php | 255 +++++++++++++++++++++
 2 files changed, 296 insertions(+), 10 deletions(-)
 create mode 100644 tests/AgentCore/Application/Pipeline/ToolCallExtractorTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/qh-05-ask-human-agentcore-compatibility.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/qh-05-ask-human-agentcore-compatibility.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-06-29T00:51:04.819Z
- Updated PR Status: merged
- Validation: Post-merge LLM_MODE=true castor check: quality: ok (184.5s); deptrac OK (1.8s) — 0 violations; test OK (29.1s) — 3779 tests, 12010 assertions; test:controller-replay OK (58.0s) — 8 tests, 112 assertions; test:tui OK (59.7s) — 18 tests, 91 assertions; test:llm-real OK (32.3s) — 9 tests, 110 assertions; phpstan OK (2.9s) — 0 errors; cs-check OK (0.7s); llama-proxy cache guard ok (174→174); QA artifact integrity ok (7 lane logs); QA leak check ok; Integration checkout: main ahead of origin by 2 commits (local merge + pull)
- Summary: MOVED TO DONE. PR #232 merged (GitHub merge at 2026-06-29T00:49:01Z). move_task merged task/qh-05-ask-human-agentcore-compatibility into integration checkout via ort strategy (2 files, +296/-10), worktree removed, IDEA exclusions cleaned up, integration checkout pulled. Post-merge LLM_MODE=true castor check passed clean (184.5s): deptrac OK, test OK (3779 tests, 12010 assertions), controller-replay OK (8 tests, 112 assertions), tui OK (18 tests, 91 assertions), llm-real OK (9 tests, 110 assertions), phpstan OK (0 errors), cs-check OK, llama-proxy cache guard ok (174→174), QA artifact integrity ok, QA leak check ok.
