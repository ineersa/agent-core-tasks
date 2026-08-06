# Fix z.ai GLM reasoning shaping + add glm-5.2 (clear_thinking, model-level reasoning_effort)

## Goal

Make hatfield drive z.ai GLM thinking the same way pi-mono does, so that **glm-5.2** can use reasoning-effort levels and so that all z.ai models send `clear_thinking:false` to let replayed `reasoning_content` participate in provider caching. This is **resolver code first, catalog second** — adding glm-5.2 to the settings catalog alone does NOT work because `ReasoningOptionsResolver` can't currently express the required request shape.

Confirmed real gaps after auditing pi-mono `origin/main` (merge commit `054f5065`).

## Why this is not just a catalog edit — three gaps

Reference: pi-mono `packages/ai/src/api/openai-completions.ts` z.ai branch (~lines 600-614):
```ts
zaiParams.thinking = options.reasoningEffort ? { type: "enabled", clear_thinking: false } : { type: "disabled" };
if (options.reasoningEffort && compat.supportsReasoningEffort) {
    zaiParams.reasoning_effort = model.thinkingLevelMap?.[level] ?? level;
}
```
- glm-5.1 (supportsReasoningEffort=false) → `thinking:{type:enabled,clear_thinking:false}` only
- glm-5.2 (supportsReasoningEffort=true, map `{minimal:null,low:"high",medium:"high",high:"high",xhigh:"max"}`) → thinking **plus** `reasoning_effort` high/max

Hatfield `src/CodingAgent/Config/ReasoningOptionsResolver.php` z.ai branch (line ~62) currently does:
```php
if ('zai' === $thinkingFormat) return ['enable_thinking' => true];   // that's ALL
```

**GAP 1 — no `clear_thinking:false`**: nowhere in hatfield `src/`. Flows verbatim to the wire (`ReasoningOptionsFeatureShaper` just merges). Replay of prior `reasoning_content` can't join z.ai provider caching (pi-mono #6083).

**GAP 2 — no `reasoning_effort` for glm-5.2**: the zai branch returns early *before* checking effort support, so no z.ai model ever gets reasoning_effort. The whole point of "use glm-5.2 with effort levels" won't work without fixing this.

**GAP 3 — model-level `supportsReasoningEffort` ignored**: `ReasoningOptionsResolver::supportsReasoningEffort()` reads only **provider-level** compat (`$this->catalog->getProvider(...)->compatibility->supportsReasoningEffort`), never the model-level `?AiCompatibility` override — even though `AiModelDefinition` supports one (glm-5.1 already uses it for `zai_tool_stream`). A per-model `supports_reasoning_effort:true` on glm-5.2 would be silently ignored. **This is the foundational fix the other two depend on.**

## Work to do

### Primary — `src/CodingAgent/Config/ReasoningOptionsResolver.php`
1. `supportsReasoningEffort()` helper: fall back to **model-level** compatibility when the model has its own `?AiCompatibility`, else provider-level, else default true. (Mirrors how `thinkingFormat()` already prefers model-level.)
2. Rework the zai branch to mirror pi-mono:
   - When reasoning active: emit `thinking => ['type' => 'enabled', 'clear_thinking' => false]` (drop bare `enable_thinking` — confirm z.ai accepts only `thinking.type`; pi-mono uses `enable_thinking` exclusively for Qwen, NOT z.ai).
   - When reasoning off: emit `thinking => ['type' => 'disabled']`.
   - When **model-level** `supportsReasoningEffort` is true: ALSO emit `reasoning_effort => $mappedValue` from the model `thinkingLevelMap` (glm-5.2 → high/max).
3. Update the stale `AiCompatibility.php` doc comment (`'zai for enable_thinking boolean'`) → reflect `thinking.type` + `clear_thinking`.

### Catalog — `.hatfield/settings.yaml` (+ `docs/settings.md`)
4. Add `glm-5.2` to the `zai` provider: context_window 1000000, max_tokens 131072, input [text], reasoning true, tool_calling true, cost all 0 (z.ai coding plan = free).
5. Per-model override `compatibility: { supports_reasoning_effort: true, zai_tool_stream: true }` on glm-5.2 to override the provider-level `supports_reasoning_effort: false`.
6. `thinking_level_map` reflecting `{minimal: null, low: high, medium: high, high: high, xhigh: max}` — confirm hatfield's `null` semantics vs pi-mono (pi-mono `null` = level maps to "off"/no-effort; the resolver returns `[]` when mappedValue is null, which is correct).
7. Optionally add a `zai-coding-cn` provider (bigmodel.cn endpoint) mirroring zai models — currently absent in hatfield. (Out of scope if low value; note decision.)
8. `docs/settings.md`: update `favorite_models` examples (`zai/glm-5.1` → mention `zai/glm-5.2`); keep settings doc in sync per AGENTS.md.

### Compat flag parity (minor)
9. pi-mono catalogs gained `supportsStore:false` across providers. Hatfield `AiCompatibility` does not model `supportsStore`. Add only if it maps to a real hatfield behavior; otherwise note as out-of-scope (it is a no-op for hatfield's request shaping today).

## deepseek — NO CHANGE (confirmed)
pi-mono catalog unchanged (v4-pro/flash, same prices/context). Hatfield deepseek matches. Compat gained `supportsStore:false`/`supportsDeveloperRole:false` (hatfield already sets `requires_reasoning_content_on_assistant_messages` + `thinking_format: deepseek`). Verify prices/context still match after edits.

## Reference data — pi-mono glm-5.2 (zai) on origin/main
```
glm-5.2: api openai-completions, baseUrl https://api.z.ai/api/coding/paas/v4
compat: supportsStore:false, supportsDeveloperRole:false, supportsReasoningEffort:true,
        thinkingFormat:"zai", zaiToolStream:true
reasoning: true
thinkingLevelMap: {minimal:null, low:"high", medium:"high", high:"high", xhigh:"max"}
input: [text]
contextWindow: 1000000   maxTokens: 131072
cost: input 0, output 0, cacheRead 0, cacheWrite 0   (z.ai coding plan = free)
```
(zai-coding-cn mirror uses baseUrl `https://open.bigmodel.cn/api/coding/paas/v4`.)

Upstream commits for reference: `75b0d723 fix(ai): support Z.AI GLM-5.2 effort levels` (#5770), `b91bdd5a fix(ai): preserve Z.AI thinking content` (#6083).

## Acceptance criteria
- [ ] `ReasoningOptionsResolver::supportsReasoningEffort()` honors model-level compatibility (fallback chain: model → provider → default true) — proven by a focused resolver test where a model-level override flips the outcome vs provider-level
- [ ] z.ai branch emits `thinking:{type:enabled,clear_thinking:false}` when reasoning on and `thinking:{type:disabled}` when off — NOT bare `enable_thinking`
- [ ] glm-5.2 (model-level supports_reasoning_effort:true) additionally emits `reasoning_effort` high/max from its thinkingLevelMap; glm-5.1/glm-5v-turbo (supports_reasoning_effort:false) emit thinking only — both behaviors asserted in a resolver/mapper test
- [ ] `glm-5.2` added to `zai` provider in `.hatfield/settings.yaml` with context_window 1000000, max_tokens 131072, reasoning true, per-model `supports_reasoning_effort: true` + `zai_tool_stream: true`, cost 0
- [ ] `docs/settings.md` updated and in sync (favorite_models examples etc.); stale `AiCompatibility` doc comment corrected
- [ ] deepseek (v4-pro/flash) prices/context confirmed unchanged vs pi-mono
- [ ] Focused Castor validation passes (`castor test` for the resolver/mapper path; `castor phpstan` + `castor cs-check` for the resolver change). Virtual-layer proof is sufficient; no TUI/tmux test required for request shaping.

## Workflow metadata
Status: DONE
Branch: task/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/271
PR Status: merged
Started: 2026-07-09T15:06:31.515Z
Completed: 2026-07-09T16:29:39.120Z

## Work log
- Created: 2026-07-07T22:18:12.830Z
- 2026-07-07: Split scope. Codex reliability work moved to its own task (`2026-07-07-codex-provider-reliability-after-pi-mono-merge`) since it is a separate exploratory workstream (test + port what's meaningful without websockets). This task now owns z.ai/GLM only and is resolver-code-first.
- 2026-07-07 CORRECTED ANALYSIS (supersedes the original optimistic 'just add glm-5.2 to catalog' framing):
  - GAP 1 — z.ai param shape: pi-mono emits `thinking:{type:"enabled",clear_thinking:false}` (commit b91bdd5a, #6083). Hatfield emits ONLY `['enable_thinking' => true]`. No clear_thinking anywhere in src/. Flows verbatim to the wire (ReasoningOptionsFeatureShaper just merges; no downstream reshape).
  - GAP 2 — glm-5.2 reasoning_effort: pi-mono sends reasoning_effort (high/max via thinkingLevelMap) when compat.supportsReasoningEffort is true (commit 75b0d723, #5770). Hatfield's zai branch returns early BEFORE checking effort support → NEVER emits reasoning_effort for any z.ai model.
  - GAP 3 — model-level supportsReasoningEffort ignored: ReasoningOptionsResolver::supportsReasoningEffort() reads only PROVIDER-level compat, not the per-model `?AiCompatibility` override that AiModelDefinition already supports (glm-5.1 uses it for zai_tool_stream). A model-level supports_reasoning_effort:true on glm-5.2 would be ignored.
  - Resolver work: (a) zai branch → thinking.type+clear_thinking, + reasoning_effort when model-level supportsReasoningEffort; (b) supportsReasoningEffort() fallback to model-level; (c) fix stale AiCompatibility doc comment. pi-mono ref: api/openai-completions.ts ~600-614.

## Task workflow update - 2026-07-09T15:06:31.515Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge.
- Summary: Claiming task-start for implementation orchestration. Main agent will gather context and dispatch implementation to a fork; no direct edits in integration checkout.

## Task workflow update - 2026-07-09T15:11:29.366Z
- Summary: Task-start context gathered. Worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge. Loaded task-workflow and testing instructions plus tests/AGENTS.md. Scout context identified primary implementation files: src/CodingAgent/Config/ReasoningOptionsResolver.php, src/CodingAgent/Config/Ai/AiCompatibility.php, src/CodingAgent/Config/SessionAwareModelResolver.php, .hatfield/settings.yaml, config/hatfield.defaults.yaml, docs/settings.md, tests/CodingAgent/Config/ReasoningOptionsResolverTest.php, tests/CodingAgent/Config/SessionAwareModelResolverTest.php, tests/AgentCore/Infrastructure/SymfonyAi/ProviderCompatibilityRequestShaperTest.php, and catalog parsing tests as needed. Scope decisions for fork: no TUI/tmux proof needed; virtual/unit resolver/request-shaping proof is sufficient. Do not add supportsStore or zai-coding-cn unless a real Hatfield behavior is found; current scout found neither concept in this codebase.

## Task workflow update - 2026-07-09T15:21:07.607Z
- Validation: castor test --filter=ReasoningOptionsResolverTest — PASS (fork report: 23 tests); castor test --filter=SessionAwareModelResolverTest — PASS (fork report: 11 tests); castor test --filter=ProviderCompatibilityRequestShaperTest — PASS (fork report: 7 tests); castor test --filter=AiConfigTest — PASS (fork report: 16 tests); castor phpstan --path=src/CodingAgent/Config — PASS; castor cs-check — PASS after final comment cleanup (0 fixable files); Verification: git status --short in task worktree is clean; git diff --stat main...HEAD shows 14 expected changed files; rg enable_thinking in task worktree only reports docs/settings.md Codex negative mention.
- Summary: Implementation fork completed on worktree /home/ineersa/projects/agent-core-worktrees/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge. Final task branch HEAD: 9d928922c (with feature commit 63174632f and follow-up style/comment commits 704644815, 9d928922c). Worktree verified clean. Changed expected files: ReasoningOptionsResolver, AiCompatibility explicit supports_reasoning_effort handling, SessionAwareModelResolver off-path, z.ai catalog/docs/defaults, resolver/session/shaper/config tests, and source comment examples. Key decisions: z.ai active uses thinking.type enabled + clear_thinking:false; z.ai off emits thinking.type disabled; explicit model supports_reasoning_effort true enables glm-5.2 effort without letting zai_tool_stream-only model compat imply effort support. supportsStore and zai-coding-cn remained out of scope; deepseek prices/context were not edited. Note: cleanup fork accidentally created comment-only commit 9efbb32f3 on the integration checkout main; equivalent comment cleanup was subsequently committed correctly on the task branch as 9d928922c. Integration checkout cleanup/reset requires user approval because destructive git reset is forbidden without approval.

## Task workflow update - 2026-07-09T15:40:33.871Z
- Validation: Reviewer subagent decision: APPROVED on HEAD d11f19cea after re-review; reviewer read .agents/skills/testing/SKILL.md and tests/AGENTS.md.; castor test — PASS (4215 tests, 13723 assertions, 22.2s); castor deptrac — PASS (violations=0, errors=0, uncovered=1612, allowed=2063); castor phpstan — PASS (errors=0, file_errors=0); castor cs-check — PASS (files_fixed=0); castor test:llm-real — PASS (llama.cpp generation ok; 10 tests, 121 assertions, 19.7s); git status --short --branch in task worktree — clean on task/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge; HEAD d11f19cea.
- Summary: Task-to-PR review completed. Reviewer first returned APPROVE WITH SUGGESTIONS due to malformed docs/settings.md z.ai YAML example, null-compatibility fallback warning/fatal risk in ReasoningOptionsResolver, and misleading comments. Fork addressed all actionable findings in commit d11f19cea (`fix(ai): address z.ai reviewer findings for docs and reasoning resolver`): fixed docs YAML model blocks, made provider compatibility fallback explicit/null-safe, added z.ai off rationale, corrected SessionAwareModelResolver reasoning-options comment, and added a focused provider-without-compatibility resolver test. Reviewer re-reviewed current HEAD d11f19cea and returned APPROVED. Non-blocking NTH only: model ordering differs between .hatfield/settings.yaml and docs/settings.md; optional YAML-to-resolver glm-5.2 integration test could be added later but existing generic parsing + resolver tests cover behavior.

## Task workflow update - 2026-07-09T15:42:25.310Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (96.3s).
- Pushed task/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge to origin.
- branch 'task/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge' set up to track 'origin/task/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge'.
- Created PR: https://github.com/ineersa/agent-core/pull/271
- Validation: Reviewer: APPROVED on d11f19cea.; castor test — PASS (4215 tests, 13723 assertions); castor deptrac — PASS (violations=0, errors=0); castor phpstan — PASS (errors=0, file_errors=0); castor cs-check — PASS (files_fixed=0); castor test:llm-real — PASS (10 tests, 121 assertions)
- Summary: Prepared for code review. Reviewer subagent approved HEAD d11f19cea after one forked fix pass. Focused validation passed locally: castor test, castor deptrac, castor phpstan, castor cs-check, and castor test:llm-real. Moving to CODE-REVIEW will run deterministic castor check, push the task branch, and create/update the PR.

## Task workflow update - 2026-07-09T16:03:41.117Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Reopening CODE-REVIEW task to resolve PR merge conflicts against updated origin/main. PR #271 currently reports mergeable=CONFLICTING / mergeStateStatus=DIRTY.

## Task workflow update - 2026-07-09T16:14:35.353Z
- Validation: Reviewer after merge commit 36fc3a50f: APPROVE WITH SUGGESTIONS; no blockers, only stale docs/settings.md `enable_thinking` comparison.; Fork validation for merge conflict resolution: castor cs-check — PASS; castor test --filter=ReasoningOptionsResolverTest — PASS (24 tests, 39 assertions).; Reviewer final on HEAD fc52512b1: APPROVED.; castor test — PASS (4224 tests, 13899 assertions, 21.8s); castor deptrac — PASS (violations=0, errors=0, uncovered=1612, allowed=2061); castor phpstan — PASS (errors=0, file_errors=0); castor cs-check — PASS (files_fixed=0); castor test:llm-real — PASS (llama.cpp generation ok; 10 tests, 121 assertions, 20.1s); git status --short --branch: clean, branch ahead of origin/task by 10 before push; HEAD fc52512b1.
- Summary: Conflict-resolution iteration completed for PR #271. Fork merged origin/main into the task branch using merge commit 36fc3a50f and resolved four comment-only conflicts in ProviderRequestOptionKeys.php, ResolvedModel.php, ModelSelectionService.php, and SessionAwareModelResolver.php. A reviewer returned APPROVE WITH SUGGESTIONS for the merge because docs/settings.md still had a stale Codex `enable_thinking` comparison; fork addressed it in fc52512b1 (`docs(settings): drop enable_thinking from Codex reasoning note`). Final reviewer re-review of HEAD fc52512b1 returned APPROVED. `rg enable_thinking` now has no matches in src/tests/config/docs/.hatfield/settings.yaml. Branch is ready to push/update PR again.

## Task workflow update - 2026-07-09T16:16:23.186Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (94.5s).
- Pushed task/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge to origin.
- branch 'task/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge' set up to track 'origin/task/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge'.
- PR already exists: https://github.com/ineersa/agent-core/pull/271
- Validation: Reviewer final decision: APPROVED on fc52512b1.; castor test — PASS (4224 tests, 13899 assertions); castor deptrac — PASS (violations=0, errors=0); castor phpstan — PASS (errors=0, file_errors=0); castor cs-check — PASS (files_fixed=0); castor test:llm-real — PASS (10 tests, 121 assertions); rg enable_thinking src tests config docs .hatfield/settings.yaml — PASS (no matches)
- Summary: Resolved PR #271 conflicts against updated origin/main. Merge commit 36fc3a50f merged origin/main and resolved comment-only conflicts. Follow-up docs commit fc52512b1 removed the last stale `enable_thinking` mention. Reviewer approved current HEAD fc52512b1; focused validation passed. Moving back to CODE-REVIEW will run deterministic castor check and push the updated branch/PR.

## Task workflow update - 2026-07-09T16:29:39.120Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/settings.yaml                            |  13 ++-
 config/hatfield.defaults.yaml                      |  10 ++
 docs/settings.md                                   |  39 +++++--
 .../Domain/Model/ProviderRequestOptionKeys.php     |   3 +-
 src/CodingAgent/Config/Ai/AiCompatibility.php      |  18 ++-
 src/CodingAgent/Config/ModelSelectionService.php   |   2 +-
 .../Config/ReasoningOptionsResolver.php            |  68 ++++++++---
 .../Config/SessionAwareModelResolver.php           |   6 +-
 .../ProviderCompatibilityRequestShaperTest.php     |   8 +-
 tests/CodingAgent/Config/Ai/AiConfigTest.php       |   5 +
 .../Config/ReasoningOptionsResolverTest.php        | 126 +++++++++++++++++++--
 .../Config/SessionAwareModelResolverTest.php       |  43 +++++++
 12 files changed, 295 insertions(+), 46 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-07-07-sync-zai-glm52-and-provider-catalog-after-pi-mono-merge.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Before DONE: GitHub PR #271 state=MERGED, mergedAt=2026-07-09T16:29:12Z, mergeCommit=870c274911f2137416f0ae27c6fe97e8380a4189.; CODE-REVIEW validation already recorded: reviewer APPROVED; move_task CODE-REVIEW deterministic castor check PASS (94.5s); focused castor test/deptrac/phpstan/cs-check/test:llm-real all PASS.
- Summary: PR #271 was merged on GitHub (merge commit 870c274911f2137416f0ae27c6fe97e8380a4189 at 2026-07-09T16:29:12Z). Moving tracked task to DONE and syncing integration checkout.
