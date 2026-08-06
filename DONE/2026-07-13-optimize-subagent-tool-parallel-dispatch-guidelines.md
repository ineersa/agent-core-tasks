# Optimize subagent tool description and parallel-dispatch guidelines

## Goal

Make it unambiguous that independent scout/reviewer work should be dispatched through one parallel-mode `subagent` call, rather than multiple sequential single-mode calls.

## Context

In session 5, the model needed three independent scouts but emitted three separate calls using the single-agent shape:

```json
{"agent":"scout","task":"..."}
```

The `subagent` tool is registered with sequential execution for individual calls, so the scouts ran one at a time. The tool also supports the parallel shape:

```json
{"tasks":[
  {"agent":"scout","task":"..."},
  {"agent":"scout","task":"..."},
  {"agent":"scout","task":"..."}
]}
```

This was an orchestration/guidance failure, not a technical limitation. The existing guidance mentions parallel mode but does not make the decision rule prominent enough at the point where agents plan scout work.

## Acceptance criteria

- The subagent tool description and/or agent guidance explicitly says: use one `tasks` array for independent scouts/reviewers; use `agent` + `task` only for a single child or work that must be serialized.
- Include concise valid and invalid examples showing the difference between parallel and sequential dispatch.
- State that independent reconnaissance should be batched before execution whenever the parallel-agent limit permits it.
- State when sequential dispatch is correct, such as dependent investigations, follow-up work requiring an earlier result, or deliberately serialized implementation steps.
- Preserve the artifact retrieval guidance for parallel results, including retrieving complete handoffs by artifact ID when summaries are truncated.
- Review all duplicated subagent/task-workflow instructions for consistency so they do not imply that separate single-mode calls are parallel.
- Do not change subagent execution semantics or introduce production behavior solely to mask an instruction problem.

## Likely files

- `.hatfield/skills/subagents/SKILL.md`
- `.agents/skills/task-workflow/SKILL.md`
- Agent/tool documentation or generated tool-description source, if the repository contains one.
- `docs/agents.md`, if the canonical agent documentation needs the same clarification.

## Suggested validation

- Search the repository for subagent parallel-mode instructions and confirm the examples are consistent.
- Perform a documentation review using both a multi-scout and a dependent-scout scenario to verify the decision rule is obvious.
- Run only applicable documentation/format checks; no runtime or test changes are expected unless an existing documentation test covers these files.

## Workflow metadata

Status: DONE
Branch: task/2026-07-13-optimize-subagent-tool-parallel-dispatch-guidelines
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-13-optimize-subagent-tool-parallel-dispatch-guidelines
Fork run: k3c621ldrvm3
PR URL: https://github.com/ineersa/agent-core/pull/310
PR Status: merged
Started: 2026-07-21T17:23:45.206Z
Completed: 2026-07-21T18:42:31.882Z

## Work log

- Created: 2026-07-13

## Task workflow update - 2026-07-21T17:11:19.246Z
- Summary: Reconfirmed by user on 2026-07-21 as active, worthwhile work. Keep in TODO. The task should improve orchestration guidance—not runtime semantics—so independent scout/reviewer work is batched in one parallel `tasks` call while dependent work remains sequential.
- 2026-07-21: User explicitly confirmed this task should be done; retain as an active TODO.

## Task workflow update - 2026-07-21T17:23:45.206Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-07-13-optimize-subagent-tool-parallel-dispatch-guidelines.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-07-13-optimize-subagent-tool-parallel-dispatch-guidelines.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-07-13-optimize-subagent-tool-parallel-dispatch-guidelines.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-07-13-optimize-subagent-tool-parallel-dispatch-guidelines.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-07-13-optimize-subagent-tool-parallel-dispatch-guidelines.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-07-13-optimize-subagent-tool-parallel-dispatch-guidelines.
- Summary: Starting implementation with user-approved scope restricted to agent-core's native Hatfield guidance. External Pi subagents extension, user-level Pi configuration, and repository .pi/prompts copies are excluded. Runtime execution semantics remain unchanged.

## Task workflow update - 2026-07-21T17:24:18.188Z
- Recorded fork run: k3c621ldrvm3
- Summary: Implementation fork launched in the task worktree. Scope is restricted to agent-core native Hatfield tool/guidance and focused regression coverage; external/user Pi sources and repository .pi/prompts are excluded. Runtime semantics must remain unchanged.

## Task workflow update - 2026-07-21T17:30:58.477Z
- Recorded fork run: k3c621ldrvm3
- Validation: castor test --filter=SubagentToolDefinitionBuilderTest — OK (2 tests, 16 assertions); castor deptrac — OK (0 violations); castor phpstan — OK (0 errors); castor cs-check — OK (0 files fixed); git diff --check — clean; castor test:llm-real — FAILED in 4 broader live E2Es (rewind expected pineapple, controller missing assistant completion, fork deferred result size, output-cap read case); none directly asserted the changed subagent guidance. Re-baseline/retry during task-to-pr before CODE-REVIEW. Full castor check intentionally not run during task-start.
- Summary: Implementation complete at commit b116f344356d6a25da62252c085f89c38c8c0506. Updated the authoritative model-visible SubagentToolDefinitionBuilder plus native Hatfield skill/docs/task prompts and task-workflow guidance so independent scouts/reviewers are batched in one tasks call, while single mode is reserved for one child or genuinely dependent/serialized work. Added correct-batch, anti-pattern, and dependent-sequential guidance; preserved Artifact/agent_retrieve instructions. Added one focused concept-level builder regression test. Runtime execution/schema/artifact behavior is unchanged. Exactly 9 repository files changed; external Pi sources, user Pi files, and repository .pi/prompts remained untouched. Fork confirmed it read the testing skill and tests/AGENTS.md; worktree is clean.

## Task workflow update - 2026-07-21T17:49:14.607Z
- Validation: castor test — OK (4450 tests, 15284 assertions); castor deptrac — OK (0 violations, 0 errors); castor phpstan — OK (0 errors); castor cs-check — OK (0 files fixed); git diff --check origin/main...HEAD — clean; castor test:llm-real on integration main baseline — OK (12 tests, 163 assertions); castor test:llm-real in task worktree — broader live suite unstable: first run 3 unrelated failures; retry 2 unrelated failures (ControllerSmokeTest assistant completion and OutputCapReadFileControllerTest read-tool start). Filtered RewindBranchLiveE2eTest and ControllerSmokeTest then passed; filtered OutputCapReadFileControllerTest remained red in worktree while the same filtered test passed on integration main. These tests do not exercise subagent dispatch guidance. Deterministic castor check gate remains required by move_task.
- Summary: Task-to-PR review complete on HEAD fc5e83a9684ddd823b72d13322a75bc9dc73e1d1. Initial review found three guidance clarity issues; fixed in 36c313d2b. Re-review found one remaining invalid pseudo-JSON example in authoritative model-visible guidance; fixed with a focused regression assertion in fc5e83a96. Final reviewer verdict: APPROVED with zero actionable task-scope findings. Full diff remains 9 files, guidance/test only; runtime/schema behavior and excluded Pi paths unchanged; worktree clean.

## Task workflow update - 2026-07-21T17:54:34.708Z
- Validation: First move_task CODE-REVIEW gate — FAILED only cache guard: llama-proxy entries 105→106; Warmup `castor test:llm-real` — populated remaining cassettes; cache reached 111 entries; Repeat warmup — cache stable 111→111; direct 4-worker run had unrelated ShellFollowUp/ViewImage model-path failures
- Summary: First CODE-REVIEW gate ran all validation but failed the final llama-proxy cache-growth guard (105→106), not a code/test lane. Followed warmup runbook with repeated `castor test:llm-real`; proxy cache reached 111 entries and remained stable at 111 on the next identical warmup. Direct 4-worker live runs remain model-path unstable in unrelated tests, while the prior deterministic gate's test lane passed. Ready to retry deterministic gate with stable cache.

## Task workflow update - 2026-07-21T18:06:38.049Z
- Validation: Branch warmup run: cache 112→113; Branch warmup run: cache 113→114; Branch warmup stability run: cache 114→114
- Summary: Continued branch-specific llama-proxy warmup after user confirmation that changed tool descriptions require new cache keys. Successive runs populated 112→113→114 entries; final identical warmup was stable at 114→114. Direct 4-worker lane still reports one cached ShellFollowUp model-path failure, but cache is now fully warm for the branch request set. Retrying deterministic gate to establish authoritative result.

## Task workflow update - 2026-07-21T18:21:48.931Z
- Validation: castor test — OK (4450 tests, 15284 assertions) on final HEAD; castor deptrac — OK (0 violations, 0 errors) on final HEAD; castor phpstan — OK (0 errors) on final HEAD; castor cs-check — OK (0 files fixed) on final HEAD; castor test:llm-real — OK (12 tests, 163 assertions) on final HEAD; repeat castor test:llm-real — OK (12 tests, 163 assertions); llama-proxy cache stable 139→139; git diff --check origin/main...HEAD — clean; Two earlier CODE-REVIEW gates failed cache-growth guard while provider schema was changed (105→106 and 111→112); final provider-schema restoration removes that branch-specific cache-key churn.
- Summary: Final task-to-PR HEAD is 9a267dcf4090d2828c114ac23542bc020775489d. Provider-facing subagent description/schema were restored byte-for-byte to origin/main so llama-proxy can reuse proven tool-schema cassettes; the new dispatch decision rule lives in compact promptGuidelines plus maintained Hatfield skills/docs/prompts. Prompt guidance is 3 concise lines covering independent batching, dependent serialization, inner concurrency vs outer serialization/cap overflow, and Artifact/agent_retrieve retrieval. Detailed valid/anti-pattern/sequential examples remain in documentation surfaces. Final reviewer verdict APPROVED with no actionable findings; runtime/schema constraints unchanged; worktree clean.

## Task workflow update - 2026-07-21T18:24:06.279Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (125.5s).
- Pushed task/2026-07-13-optimize-subagent-tool-parallel-dispatch-guidelines to origin.
- branch 'task/2026-07-13-optimize-subagent-tool-parallel-dispatch-guidelines' set up to track 'origin/task/2026-07-13-optimize-subagent-tool-parallel-dispatch-guidelines'.
- Created PR: https://github.com/ineersa/agent-core/pull/310

## Task workflow update - 2026-07-21T18:24:13.590Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/310
- Updated PR Status: open
- Validation: Deterministic castor check — PASSED (125.5s)
- Summary: Task moved to CODE-REVIEW. Deterministic castor check passed in 125.5s, branch pushed, and PR #310 created: https://github.com/ineersa/agent-core/pull/310.

## Task workflow update - 2026-07-21T18:42:31.883Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-07-13-optimize-subagent-tool-parallel-dispatch-guidelines into integration checkout.
- Merge made by the 'ort' strategy.
 .agents/skills/task-workflow/SKILL.md              | 13 ++++++---
 .hatfield/prompts/task-explain.md                  |  3 ++-
 .hatfield/prompts/task-review-iterate.md           |  2 +-
 .hatfield/prompts/task-start.md                    |  3 ++-
 .hatfield/prompts/task-to-pr.md                    |  4 +--
 .hatfield/skills/subagents/SKILL.md                | 24 ++++++++++++++---
 docs/agents.md                                     | 20 +++++++++++---
 .../Agent/Tool/SubagentToolDefinitionBuilder.php   | 21 ++++++++-------
 .../Tool/SubagentToolDefinitionBuilderTest.php     | 31 ++++++++++++++++++++++
 9 files changed, 96 insertions(+), 25 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-07-13-optimize-subagent-tool-parallel-dispatch-guidelines.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-07-13-optimize-subagent-tool-parallel-dispatch-guidelines.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #310 verified merged on GitHub at 5561f66543070e367011c7288173df469d727e22 (merged 2026-07-21T18:42:04Z). Moving task to DONE and cleaning its worktree.

## Task workflow update - 2026-07-21T18:44:43.765Z
- Validation: LLM_MODE=true castor check — OK (QA run qa-20260721-184242-434880-09ae9723, 304.7s); deptrac — OK; test — OK (4445 tests, 15227 assertions); test:controller-replay — OK (9 tests, 127 assertions); test:tui — OK (36 tests, 185 assertions); test:llm-real — OK (12 tests, 163 assertions); phpstan — OK (0 errors); cs-check — OK; llama-proxy cache guard stable (139→139); QA artifact integrity — OK; QA run leak check — OK
- Summary: Post-merge integration validation completed successfully. Task worktree and IDEA exclusions were removed; integration checkout contains merged PR #310.
