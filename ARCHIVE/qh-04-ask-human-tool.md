# QH-04 ask_human tool and interrupt payload normalization

## Goal
Plan: .pi/plans/tui-question-hitl-plan.md

**V1 decision (2026-06-28):** Thin non-blocking tool — returns interrupt payload immediately; no oneshot/blocking path; no resume responsibility. This is the main remaining implementation task.

Scope:
- Add `src/CodingAgent/Tool/AskHumanTool.php` plus a Hatfield tool definition/provider for `ask_human` instead of relying on `#[AsTool]` metadata.
- Return kind=interrupt payload immediately; do not block waiting for input.
- Support prompt, header, kind, schema, choices, default, allow_other, secret, and optional question_id.
- Normalize bare string choices to label/description objects.
- Generate stable fallback question_id when absent.
- Include defensive `ask_human` interrupt fallback in ToolExecutor alongside existing `ask_user`.

Exclusions:
- Do not implement a blocking/oneshot tool path like Codex.
- Do not implement TUI widgets or input routing.
- Do not implement resume-pending-HITL behavior.

Dependencies: TOOLS-R02 for Hatfield tool definition conventions, TOOLS-R03 for registry-backed Toolbox.
Parallelizable with: QH-01, QH-02, QH-03 after the TOOLS-R02/TOOLS-R03 conventions are stable.

## Acceptance criteria
- `ask_human` is discoverable through registry-backed Symfony Toolbox metadata and present in ToolRegistry permanent metadata.
- Tool result contains kind=interrupt, question_id, prompt, schema, normalized choices, and UI metadata.
- Unit tests cover text, confirm, choice, approval, and fallback id behavior.
- Tool returns immediately and does not wait for human input.
- castor deptrac passes.

## Workflow metadata
Status: DONE
Branch: task/qh-04-ask-human-tool
Worktree: /home/ineersa/projects/agent-core-worktrees/qh-04-ask-human-tool
Fork run: emgk06z4x9om
PR URL: https://github.com/ineersa/agent-core/pull/231
PR Status: merged
Started: 2026-06-28T16:27:51.555Z
Completed: 2026-06-28T21:33:26.772Z

## Work log
- Created: 2026-05-18T00:04:28.622Z

## Task workflow update - 2026-06-28T16:27:51.555Z
- Moved TODO → IN-PROGRESS.
- Created branch task/qh-04-ask-human-tool.
- Created worktree /home/ineersa/projects/agent-core-worktrees/qh-04-ask-human-tool.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/qh-04-ask-human-tool.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/qh-04-ask-human-tool.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/qh-04-ask-human-tool.
- Summary: Starting implementation for thin v1 ask_human tool. Scope: new Hatfield tool provider/handler for ask_human, interrupt payload normalization, defensive ToolExecutor ask_human fallback, focused tests, no blocking path, no TUI routing, no resume support.

## Task workflow update - 2026-06-28T16:32:49.032Z
- Loaded task-workflow and testing skills, read tests/AGENTS.md, task file, QH plan sections, and focused implementation context in the task worktree.
- Scout completed read-only exploration. Parent corrected one scout recommendation: ToolExecutor must still add explicit ask_human fallback because the existing ask_user name check does not cover ask_human; QH-04 also owns interrupt payload normalization, while QH-05 owns downstream ToolCallExtractor/runtime preservation.

## Task workflow update - 2026-06-28T17:11:51.738Z
- Validation: Fork reported: castor test --filter=AskHumanToolTest — OK (33 tests, 65 assertions).; Fork reported: castor test --filter=ToolExecutorTest — OK (18 tests, 104 assertions after follow-up).; Fork reported: castor test --filter="/ToolExecutorTest|AskHumanToolTest/" — OK (51 tests, 169 assertions).; Fork reported: castor deptrac — 0 violations, 0 errors.; Fork reported: castor phpstan --path=src/CodingAgent/Tool/AskHumanTool.php — 0 errors.; Fork reported: castor phpstan --path=src/AgentCore/Application/Handler/ToolExecutor.php — 0 errors.; Fork reported: castor cs-check — clean.; Parent verification: git status --short in worktree clean; git diff --stat main...HEAD shows only 4 expected files changed.
- Summary: Implementation completed in worktree /home/ineersa/projects/agent-core-worktrees/qh-04-ask-human-tool on branch task/qh-04-ask-human-tool. Commits: 1e3b8b7fc feat(QH-04): add ask_human tool and interrupt payload normalization; aa4f09942 fix(QH-04): normalize choices and derive schema/ui_kind in ToolExecutor interruptResult. Changed files: src/CodingAgent/Tool/AskHumanTool.php, src/AgentCore/Application/Handler/ToolExecutor.php, tests/CodingAgent/Tool/AskHumanToolTest.php, tests/AgentCore/Application/Handler/ToolExecutorTest.php. Parent verified clean worktree, HEAD commit exists, and diff against main is limited to the expected 4 files. Note: no TUI/tmux proof added because QH-04 excludes TUI routing/widgets; QH-05/QH-06 will handle downstream payload propagation. castor test:llm-real is suitable opt-in validation during task-to-pr because this adds an LLM-visible tool schema.

## Task workflow update - 2026-06-28T17:50:17.507Z
- Validation: Reviewer subagent final decision: APPROVED; no blockers remain.; castor test — OK (3762 tests, 12032 assertions) after refreshing copied vendor autoload with composer dump-autoload.; castor deptrac — violations=0, errors=0.; castor phpstan — errors=0, file_errors=0.; castor cs-check — files_fixed=0, clean.; castor test:llm-real — OK (9 tests, 110 assertions).; Focused fork validation after final polish: castor test --filter="/ToolExecutorTest|AskHumanToolTest/" — OK (53 tests, 205 assertions); focused phpstan on modified source files — 0 errors; cs-check clean.
- Summary: Task-to-PR review completed. Reviewer initially returned REQUEST CHANGES for production ask_human question_id stability/parity, empty prompt handling, choice shape consistency, and parity tests. Fork addressed in commit a4249972b. Reviewer then returned APPROVE WITH SUGGESTIONS for two polish items (docblock HTML entity and json_encode flags); fork addressed in commit e32e866e3. Final reviewer decision: APPROVED — no blockers remain. Current HEAD: e32e866e3 on branch task/qh-04-ask-human-tool. Worktree clean. Diff remains limited to 4 expected files: src/CodingAgent/Tool/AskHumanTool.php, src/AgentCore/Application/Handler/ToolExecutor.php, tests/CodingAgent/Tool/AskHumanToolTest.php, tests/AgentCore/Application/Handler/ToolExecutorTest.php. Note: initial full castor test failed because copied worktree vendor autoload lacked .hatfield/extensions/castor-llm-mode PSR-4 classes; ran composer dump-autoload to refresh vendor autoload, then castor test passed. No tracked files changed.

## Task workflow update - 2026-06-28T17:51:29.687Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (62.5s).
- Pushed task/qh-04-ask-human-tool to origin.
- branch 'task/qh-04-ask-human-tool' set up to track 'origin/task/qh-04-ask-human-tool'.
- Created PR: https://github.com/ineersa/agent-core/pull/231
- Validation: Reviewer final decision: APPROVED; no blockers remain.; castor test — OK (3762 tests, 12032 assertions).; castor deptrac — violations=0, errors=0.; castor phpstan — errors=0, file_errors=0.; castor cs-check — clean, files_fixed=0.; castor test:llm-real — OK (9 tests, 110 assertions).
- Summary: Ready for code review. Reviewer subagent final decision: APPROVED. Branch HEAD e32e866e3. Focused local validation passed: castor test, castor deptrac, castor phpstan, castor cs-check, and LLM-visible opt-in castor test:llm-real.

## Task workflow update - 2026-06-28T18:33:41.821Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Validation: Read PR inline comments via gh api: ToolExecutor.php line 223 architecture objection; AskHumanTool.php line 223 serializer/DTO objection.
- Summary: Review-iterate: PR #231 feedback requires architectural redesign. Decisions: remove ask_user/ask_human name special-cases from AgentCore; AgentCore should understand only generic interrupt results/mode, not tool-specific schemas or names; use Symfony Serializer/DTO/validation rather than duplicated manual normalization where practical.

## Task workflow update - 2026-06-28T18:35:46.649Z
- Recorded fork run: t4rq02mnu3t5
- Summary: Launched review-iterate implementation fork for PR #231 redesign: remove ask_user/ask_human ToolExecutor name special-cases, execute Interrupt-mode tools through normal toolbox path, keep only generic interrupt-result recognition in AgentCore, and move ask_human normalization to CodingAgent DTO/Serializer/Validator-based implementation.

## Task workflow update - 2026-06-28T18:53:28.770Z
- Recorded fork run: 7y4n8wha4c5g
- Summary: Reviewer subagent returned REQUEST CHANGES after redesign: prompt documented as alias but rejected without question; resolvePrompt fallback and AskHumanChoiceDTO dead code; optional kind/uiKind validation. Launched targeted fork to fix prompt alias validation, remove dead code, add focused tests, and rerun Castor validation.

## Task workflow update - 2026-06-28T19:14:24.317Z
- Validation: Reviewer subagent: APPROVED after bdd435943; confirmed all four prior REQUEST CHANGES fixed and AgentCore remains tool-name-agnostic generic interrupt handling only.; castor test: OK (3761 tests, 11959 assertions); castor deptrac: 0 violations, 0 errors; castor phpstan: 0 errors, 0 file_errors; castor cs-check: files_fixed=0 (clean); castor test:llm-real: OK (9 tests, 110 assertions)
- Summary: Review-iterate complete. Fork 7y4n8wha4c5g committed bdd435943 fixing all reviewer findings: prompt-only alias accepted via Callback validation, unreachable prompt fallback removed, unused AskHumanChoiceDTO deleted, kind/ui_kind validated with Assert\Choice. Reviewer subagent re-reviewed and APPROVED. Worktree clean at bdd435943.

## Task workflow update - 2026-06-28T19:15:40.108Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (63.3s).
- Pushed task/qh-04-ask-human-tool to origin.
- branch 'task/qh-04-ask-human-tool' set up to track 'origin/task/qh-04-ask-human-tool'.
- PR already exists: https://github.com/ineersa/agent-core/pull/231
- Validation: Reviewer subagent: APPROVED after bdd435943; castor test: OK (3761 tests, 11959 assertions); castor deptrac: 0 violations, 0 errors; castor phpstan: 0 errors, 0 file_errors; castor cs-check: files_fixed=0 (clean); castor test:llm-real: OK (9 tests, 110 assertions)
- Summary: Review-iterate complete for PR #231. Redesign now satisfies PR feedback: AgentCore ToolExecutor has no ask_user/ask_human or tool-specific interrupt fabrication; CodingAgent ask_human executes through normal toolbox path and uses Symfony Serializer/Validator DTO factory. Follow-up commit bdd435943 fixed prompt alias validation, removed dead code, and added kind/ui_kind Choice validation. Reviewer subagent APPROVED; focused Castor validation passed.

## Task workflow update - 2026-06-28T19:24:07.881Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR review feedback: simplify ask_human model-facing parameters. Remove model-facing arbitrary JSON Schema parameter and nested/object choice shapes because LLMs are unreliable at authoring JSON inside tool args. v1 should expose semantic kind plus simple string choices only; derive answer schema internally.

## Task workflow update - 2026-06-28T19:24:49.763Z
- Recorded fork run: ik4i5u5a7gz0
- Summary: Launched review-iterate fork for PR feedback: remove model-facing schema JSON object from ask_human, make choices simple string arrays only, derive schema internally, update validation/tests, and keep AgentCore untouched.

## Task workflow update - 2026-06-28T19:38:14.557Z
- Validation: Reviewer subagent: APPROVED for commit 301d16818; verified no model-facing schema, string-only choices, derived payload schema, DTO validation, and AgentCore remains generic.; castor test: OK (3762 tests, 11957 assertions); castor deptrac: 0 violations, 0 errors; castor phpstan: 0 errors, 0 file_errors; castor cs-check: files_fixed=0 (clean); castor test:llm-real: first full lane attempt had local provider/controller timeouts; diagnostic cleanup list found no stale workers; retry passed OK (9 tests, 110 assertions).
- Summary: Review-iterate complete for model-facing ask_human simplification. Fork ik4i5u5a7gz0 committed 301d16818: removed `schema` from LLM-visible tool definition/DTO, made `choices` string[] only, derives interrupt payload schema internally from kind/choices, rejects nested/object or empty choices, and requires choices for kind=choice. Reviewer subagent APPROVED. Worktree clean.

## Task workflow update - 2026-06-28T19:39:24.999Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (58.9s).
- Pushed task/qh-04-ask-human-tool to origin.
- branch 'task/qh-04-ask-human-tool' set up to track 'origin/task/qh-04-ask-human-tool'.
- PR already exists: https://github.com/ineersa/agent-core/pull/231
- Validation: Reviewer subagent: APPROVED for commit 301d16818; castor test: OK (3762 tests, 11957 assertions); castor deptrac: 0 violations, 0 errors; castor phpstan: 0 errors, 0 file_errors; castor cs-check: files_fixed=0 (clean); castor test:llm-real: retry passed OK (9 tests, 110 assertions) after an initial local provider timeout; cleanup diagnostics found no stale workers.
- Summary: Review-iterate complete. PR feedback addressed: ask_human no longer exposes raw JSON Schema to the model, choices are simple string arrays only, and AskHumanPayloadFactory derives the interrupt payload schema internally from kind/choices. Runtime validation rejects nested/object/empty choices and requires choices for kind=choice. AgentCore remains generic and untouched. Reviewer subagent APPROVED; focused Castor validation passed.

## Task workflow update - 2026-06-28T21:06:26.004Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR review feedback: update ask_human model-facing prompt guidelines to avoid exposing internal interrupt/pause/return mechanics. From the model perspective it should be a normal tool call used when user input is needed.

## Task workflow update - 2026-06-28T21:06:41.412Z
- Recorded fork run: emgk06z4x9om
- Summary: Launched review-iterate fork to update ask_human model-facing guidance: remove internal runtime/interrupt/pause/return mechanics and present ask_human as a normal tool call for cases where user input is needed.

## Task workflow update - 2026-06-28T21:16:35.232Z
- Validation: Reviewer subagent: APPROVED for commit 631ae631c; verified model-facing ToolDefinitionDTO fields are free of interrupt/pause/resume/block/internal runtime mechanics and simplified ask_human contract remains intact.; castor test: OK (3762 tests, 11957 assertions); castor deptrac: 0 violations, 0 errors; castor phpstan: 0 errors, 0 file_errors; castor cs-check: files_fixed=0 (clean); castor test:llm-real: initial parallel attempts had local provider timeout/flakiness after prompt text change; cleanup diagnostics found no stale workers. Warmed/retried with HATFIELD_CHECK_LLM_REAL_PARATEST_PROCESSES=1: OK (9 tests, 110 assertions), then normal parallel castor test:llm-real passed: OK (9 tests, 110 assertions).
- Summary: Review-iterate complete for model-facing guidance cleanup. Fork emgk06z4x9om committed 631ae631c, changing only AskHumanTool ToolDefinitionDTO description/promptLine/final guideline to remove internal pause/block/resume/runtime mechanics. Reviewer subagent APPROVED; worktree clean.

## Task workflow update - 2026-06-28T21:17:45.055Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (58.7s).
- Pushed task/qh-04-ask-human-tool to origin.
- branch 'task/qh-04-ask-human-tool' set up to track 'origin/task/qh-04-ask-human-tool'.
- PR already exists: https://github.com/ineersa/agent-core/pull/231
- Validation: Reviewer subagent: APPROVED for commit 631ae631c; castor test: OK (3762 tests, 11957 assertions); castor deptrac: 0 violations, 0 errors; castor phpstan: 0 errors, 0 file_errors; castor cs-check: files_fixed=0 (clean); castor test:llm-real: normal parallel retry passed OK (9 tests, 110 assertions) after cache warm / local provider flakiness; cleanup diagnostics found no stale workers.
- Summary: Review-iterate complete. Latest PR feedback addressed: ask_human model-facing ToolDefinitionDTO text no longer exposes internal runtime mechanics (no pause/block/resume/returns-immediately wording); from the model perspective it is a normal tool call for when user input is needed. Behavior unchanged. Reviewer subagent APPROVED; focused Castor validation passed.

## Task workflow update - 2026-06-28T21:33:26.772Z
- Moved CODE-REVIEW → DONE.
- Merged task/qh-04-ask-human-tool into integration checkout.
- Merge made by the 'ort' strategy.
 src/AgentCore/Application/Handler/ToolExecutor.php |  45 --
 .../Tool/AskHuman/AskHumanArgumentsDTO.php         |  91 ++++
 .../Tool/AskHuman/AskHumanPayloadFactory.php       | 230 +++++++++++
 src/CodingAgent/Tool/AskHumanTool.php              | 130 ++++++
 .../Application/Handler/ToolExecutorTest.php       |  42 +-
 tests/CodingAgent/Tool/AskHumanToolTest.php        | 456 +++++++++++++++++++++
 6 files changed, 937 insertions(+), 57 deletions(-)
 create mode 100644 src/CodingAgent/Tool/AskHuman/AskHumanArgumentsDTO.php
 create mode 100644 src/CodingAgent/Tool/AskHuman/AskHumanPayloadFactory.php
 create mode 100644 src/CodingAgent/Tool/AskHumanTool.php
 create mode 100644 tests/CodingAgent/Tool/AskHumanToolTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/qh-04-ask-human-tool.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/qh-04-ask-human-tool.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Pre-DONE CODE-REVIEW gate had passed: deterministic castor check in worktree passed (58.7s).; Focused validation before CODE-REVIEW: castor test OK (3762 tests, 11957 assertions); castor deptrac 0 violations; castor phpstan 0 errors; castor cs-check clean; castor test:llm-real OK (9 tests, 110 assertions).
- Summary: PR #231 was merged by user. Marking QH-04 complete and syncing integration checkout; final implementation adds ask_human as model-facing HITL tool with simplified arguments, Symfony DTO/Serializer/Validator-backed payload normalization, internally derived answer schema, string-only choices, and generic AgentCore interrupt handling without tool-name special cases.

## Task workflow update - 2026-06-28T21:34:55.558Z
- Validation: Post-merge integration checkout validation: LLM_MODE=true castor check passed (quality ok, 173.9s).; deptrac OK; test OK (3768 tests, 11970 assertions); test:controller-replay OK (8 tests, 112 assertions); test:tui OK (18 tests, 91 assertions); test:llm-real OK (9 tests, 110 assertions); phpstan OK (0 errors); cs-check OK.; llama-proxy cache guard OK (entries 174 → 174); QA artifact integrity OK; QA run leak check OK.
- Summary: Post-merge DONE validation completed successfully on integration checkout.
