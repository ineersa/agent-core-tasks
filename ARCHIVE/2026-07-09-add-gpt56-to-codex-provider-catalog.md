# Replace Hatfield codex catalog with GPT-5.6 only (remove gpt-5.5/5.4/5.4-mini; delete user-global codex providers)

## Goal
Consolidate Hatfield's Codex providers to a single catalog containing **only** GPT-5.6 models (luna/sol/terra). Remove all legacy Codex models (gpt-5.5, gpt-5.4, gpt-5.4-mini) and delete the redundant user-global Codex account providers entirely.

**Authoritative specs — pi-mono `packages/ai/src/providers/openai-codex.models.ts` (Codex backend, `baseUrl https://chatgpt.com/backend-api`):**

| variant | input | output | cacheRead | cacheWrite | contextWindow | maxTokens | thinkingLevelMap (full Hatfield-style) |
|---|---|---|---|---|---|---|---|
| gpt-5.6-luna | 1.00 | 6.00 | 0.10 | 1.25 | **372000** | 128000 | `{minimal:low, low:low, medium:medium, high:high, xhigh:xhigh, max:max}` |
| gpt-5.6-sol | 5.00 | 30.00 | 0.50 | 6.25 | **372000** | 128000 | same |
| gpt-5.6-terra | 2.50 | 15.00 | 0.25 | 3.125 | **372000** | 128000 | same |

(pi-mono's sparse map is `{xhigh:xhigh, max:max, minimal:low}`; missing levels default to their own name. The full map above is equivalent + adds the new `max`.)

**Scope (REVISED — user decision 2026-07-10):**
1. Project `.hatfield/settings.yaml` → `openai-codex` provider: **REPLACE** the entire `models:` block (currently gpt-5.5/gpt-5.4/gpt-5.4-mini, lines ~451-478) with **only** gpt-5.6-luna / gpt-5.6-sol / gpt-5.6-terra. COMMITTED.
2. `~/.hatfield/settings.yaml`: **DELETE** the `openai-codex-personal` provider block entirely (starts ~line 367). NOT committed (runtime config).
3. `~/.hatfield/settings.yaml`: **DELETE** the `openai-codex-canada` provider block entirely (starts ~line 411). NOT committed (runtime config).

**Key decisions:**
- `context_window: 372000` for all three (Codex backend exposes 372K; long-context tier kicks in above 272K). Hatfield has no standalone OpenAI provider, so use the Codex 372K values.
- `thinking_level_map` includes the new `max: max` — coupled to task `2026-07-09-max-thinking-level-support`. Match the existing gpt-5.5 shape and ADD `max:max`.
- Hatfield's cost schema (`src/CodingAgent/Config/Ai/AiCost.php`) **supports** `cache_read`/`cache_write` (consumed by `AiCostCalculator`). Use full `cost: {input, output, cache_read, cache_write}` for the 5.6 models. It does **NOT** support long-context `tiers` (`inputTokensAbove`) — omit and document the tier omission. Do NOT invent unsupported fields.
- Each model entry keeps the existing gpt-5.5 schema shape (name, reasoning, thinking_level_map, tool_calling, input, context_window, max_tokens, cost) — only the cost gains the extra cache keys.
- Bare `gpt-5.6` does NOT exist — only luna/sol/terra.

**Coupling:** GPT-5.6 is the only model that supports `max`, so this unblocks end-to-end `max` testing with the max-thinking task.

**Validation:** `castor test` minimum — settings parse cleanly + the three 5.6 models resolve via the provider. No live Codex test required (the user-global Codex account providers are being deleted).

## Acceptance criteria
- Project .hatfield/settings.yaml openai-codex `models:` contains ONLY gpt-5.6-luna, gpt-5.6-sol, gpt-5.6-terra — gpt-5.5/gpt-5.4/gpt-5.4-mini all removed (committed)
- Each 5.6 entry matches pi-mono specs: context_window 372000, max_tokens 128000, reasoning true, input [text, image], thinking_level_map includes max:max, cost {input, output, cache_read, cache_write}
- ~/.hatfield/settings.yaml: openai-codex-personal and openai-codex-canada provider blocks fully deleted (YAML still valid, no orphan references)
- Settings load without YAML/parse errors; the three 5.6 models resolve/select via the provider
- Focused Castor validation passes (castor test at minimum)

## Workflow metadata
Status: DONE
Branch: task/2026-07-09-add-gpt56-to-codex-provider-catalog
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-09-add-gpt56-to-codex-provider-catalog
Fork run: xl15pqg6s5ry
PR URL: https://github.com/ineersa/agent-core/pull/275
PR Status: merged
Started: 2026-07-10T03:06:37.606Z
Completed: 2026-07-10T17:53:27.117Z

## Work log
- Created: 2026-07-10T03:05:55.614Z

## Task workflow update - 2026-07-10T03:06:37.606Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-07-09-add-gpt56-to-codex-provider-catalog.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-07-09-add-gpt56-to-codex-provider-catalog.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-07-09-add-gpt56-to-codex-provider-catalog.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-07-09-add-gpt56-to-codex-provider-catalog.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-07-09-add-gpt56-to-codex-provider-catalog.

## Task workflow update - 2026-07-10T03:10:15.052Z
- Moved IN-PROGRESS → TODO.

## Scope revision - 2026-07-10
- REVISED scope (user decision): REPLACE project openai-codex models with 5.6-only (remove gpt-5.5/gpt-5.4/gpt-5.4-mini); DELETE user-global openai-codex-personal + openai-codex-canada provider blocks entirely from ~/.hatfield/settings.yaml. Cost schema confirmed to support cache_read/cache_write (AiCost.php → AiCostCalculator) but NOT tiers. Reviving existing worktree/branch.

## Task workflow update - 2026-07-10T17:29:14.166Z
- Recorded fork run: xl15pqg6s5ry
- Started implementation (task-start). move_task IN-PROGRESS could not reuse the pre-existing worktree (known limitation: 'Worktree path already exists'); per user instruction to keep the worktree, reused it directly instead of recreating. Fast-forwarded the task branch to current main (9fa118d65, includes max-thinking merge) so the fork works against the latest resolver/themes. Status field remains TODO but functionally IN-PROGRESS; will jump to CODE-REVIEW on push.
- Dispatched fork xl15pqg6s5ry on worktree cwd with precise config-only instructions: (1) COMMIT worktree .hatfield/settings.yaml openai-codex models block — replace gpt-5.5/5.4/5.4-mini with gpt-5.6-luna/sol/terra (context_window 372000, max_tokens 128000, thinking_level_map +max:max, cost +cache_read/cache_write from pi-mono openai-codex.models.ts); (2) EDIT (no commit) /home/ineersa/.hatfield/settings.yaml — delete openai-codex-personal (L367-410) + openai-codex-canada (L411-454=EOF) blocks. Validation: YAML parse both files + model/provider key dumps + focused castor phpstan/cs-check.

## Task workflow update - 2026-07-10T17:32:07.785Z
- Validation: Fork xl15pqg6s5ry: YAML parse both files OK; model keys luna/sol/terra; phpstan OK; cs-check 0 issues; SymfonyAiProviderFactoryTest 8 tests/33 assertions OK; Orchestrator independent verify: git show 587dc7074 = 1 file .hatfield/settings.yaml 15+/15-; worktree git status clean; PHP parse project ai.providers.openai-codex.models = luna/sol/terra all ctx=372000 maxtok=128000 map_has_max=yes; costs luna 1/6/0.1/1.25 sol 5/30/0.5/6.25 terra 2.5/15/0.25/3.125; user-global grep openai-codex=0, wc -l=366, parses OK providers=deepseek,llama_cpp,runpod,zai
- Summary: Implementation complete + independently verified. Commit 587dc7074 on task branch (1 file: .hatfield/settings.yaml, 15+/15-). Project openai-codex now has exactly gpt-5.6-luna/sol/terra: all ctx=372000, max_tokens=128000, thinking_level_map includes max:max, cost has cache_read/cache_write — values match pi-mono openai-codex.models.ts. Worktree clean, no other files touched. User-global ~/.hatfield/settings.yaml: openai-codex-personal + openai-codex-canada blocks deleted (0 codex refs, 366 lines, parses OK; remaining providers deepseek/llama_cpp/runpod/zai). NOT committed (outside repo). Schema note confirmed: providers nest under ai.providers, not root providers.

## Task workflow update - 2026-07-10T17:44:58.091Z
- Validation: Focused castor green: castor test OK (4213 tests, 13709 assertions, 21.7s); castor phpstan OK (fork); castor cs-check OK 0 issues (fork); Reviewer subagent APPROVE: every value matches pi-mono spec; AiCost::fromArray reads cache_read/cache_write; ReasoningOptionsResolver ACTIVE_LEVELS includes max; no default_model orphan; both YAML files parse
- Summary: task-to-pr: Reviewer APPROVE (no issues). All 3 model entries verified vs pi-mono openai-codex.models.ts (prices, ctx 372000, maxtok 128000, max in thinking_level_map, cache_read/cache_write present). Scope clean: git diff main...HEAD = .hatfield/settings.yaml + docs/settings.md only. Old gpt-5.5/5.4/5.4-mini fully gone from committed files. User-global ~/.hatfield/settings.yaml: 2 codex blocks deleted (366 lines, 0 codex refs, valid YAML, providers deepseek/llama_cpp/runpod/zai). No test warranted (config data swap; existing SymfonyAiProviderFactoryTest/ProjectedSymfonyModelCatalogTest cover loading via fixtures, don't assert project catalog). Reviewer NTH (out of scope): config/hatfield.defaults.yaml:492-545 commented-out codex example still lists gpt-5.5/5.4/5.4-mini — dead template, optional follow-up.

## Task workflow update - 2026-07-10T17:45:22.491Z
- Moved TODO → CODE-REVIEW.

## Task workflow update - 2026-07-10T17:46:33.498Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/275
- Updated PR Status: open
- Summary: CODE-REVIEW: PR #275 created https://github.com/ineersa/agent-core/pull/275 (Replace Codex catalog with GPT-5.6 luna/sol/terra). Branch pushed to origin. Note: move_task TODO→CODE-REVIEW only flipped status (did not auto-run gate/push/PR, likely due to non-standard TODO→CODE-REVIEW path skipping IN-PROGRESS); completed push + gh pr create manually. Full castor check not required (config/docs-only, no runtime/TUI/Messenger impact per AGENTS.md); focused validation green (castor test 4213 OK, phpstan OK, cs-check 0 issues, reviewer APPROVE). Awaiting user review/merge → DONE.

## Task workflow update - 2026-07-10T17:53:27.117Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-07-09-add-gpt56-to-codex-provider-catalog into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/settings.yaml | 30 +++++++++++++++---------------
 docs/settings.md        |  2 +-
 2 files changed, 16 insertions(+), 16 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-07-09-add-gpt56-to-codex-provider-catalog.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-07-09-add-gpt56-to-codex-provider-catalog.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #275 merged by user. Codex catalog consolidated to GPT-5.6 (luna/sol/terra) + user-global codex account providers deleted. Config + docs only. Coupled to PR #274 (max thinking level) — GPT-5.6 is the first model with max in its thinking_level_map, now end-to-end testable.
