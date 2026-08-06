# Update Codex model context windows to 272K

## Goal
OpenAI Codex model context limits changed from 372K to 272K. Update every active Hatfield source/configuration/documentation/test occurrence so model metadata, context budgeting, and `/usage` display fixtures consistently use `272000`.

Known current occurrences:
- `.hatfield/settings.yaml`: `gpt-5.6-luna`, `gpt-5.6-sol`, and `gpt-5.6-terra` each use `context_window: 372000`.
- `tests/CodingAgent/Config/ModelResolverTest.php`: GPT-5.6 Luna fixture uses `372000`.
- PR #316 introduces a `/usage` virtual TUI fixture with `contextWindow = 372000`; update it if present when this task starts.

At implementation time, search all tracked active files for `372000`, `372K`, and equivalent formatting before editing. Do not rewrite git history or archived external task records. Keep `max_tokens: 128000` and pricing unchanged unless separately confirmed.

## Acceptance criteria
- All active Codex GPT-5.6 model definitions use `context_window: 272000` instead of `372000`.
- All active tests, fixtures, examples, and documentation that represent the Codex context limit use 272K consistently, including `/usage` session display coverage if PR #316 has merged.
- No active tracked source/config/docs/test occurrence of the obsolete 372K Codex limit remains; historical git commits and archived external task records are excluded.
- Unrelated providers, output-token limits, reasoning mappings, and model pricing are unchanged.
- Relevant configuration/model-resolution/TUI tests pass through Castor; `castor phpstan` and `castor cs-check` are clean.

## Workflow metadata
Status: DONE
Branch: task/update-codex-context-windows-272k
Worktree: /home/ineersa/projects/agent-core-worktrees/update-codex-context-windows-272k
Fork run: fydcwja19bne
PR URL: https://github.com/ineersa/agent-core/pull/342
PR Status: merged
Started: 2026-07-31T00:12:36.530Z
Completed: 2026-07-31T03:00:28.828Z

## Work log
- Created: 2026-07-23T20:44:37.116Z

## Task workflow update - 2026-07-30T21:21:05+00:00
- Validation: castor test  (covers ModelResolverTest, ChildRunModelRoutingProvenanceRegressionTest, and the VIRTUAL TuiUsageCommandVirtualTest — all run in the main suite; no tmux/TmuxHarness E2E needed); castor phpstan  (must be clean per acceptance criteria); castor cs-check  (must be clean per acceptance criteria); Acceptance self-check: `git grep -nE '372000|372K|372,000' -- . ':(exclude).pi/'` excluding `.hatfield/{sessions,tmp,cache,logs}/` returns ZERO active hits; castor deptrac unaffected (no architectural boundary change) — optional
- Summary: VERIFIED BLAST RADIUS — exactly 4 files / 6 occurrences of `372000` → `272000`. 272000 is already canonical (`config/hatfield.defaults.yaml` examples + `ContextBudgetReminderHookSubscriberTest` use it); this aligns the GPT-5.6 family with that precedent.

EDITS (372000 → 272000):
1. `.hatfield/settings.yaml` (~line 70, inline `openai-codex` models map): change `context_window: 372000` → `272000` for ALL THREE of `gpt-5.6-luna`, `gpt-5.6-sol`, `gpt-5.6-terra`. Leave `max_tokens: 128000`, `cost`, `reasoning`, `thinking_level_map`, `tool_calling`, `input` unchanged.
2. `tests/CodingAgent/Config/ModelResolverTest.php:476` — `'context_window' => 372000,` → `272000`.
3. `tests/CodingAgent/Agent/Execution/Subagent/ChildRun/ChildRunModelRoutingProvenanceRegressionTest.php:201` AND `:210` — both `'context_window' => 372000,` → `272000`.
4. `tests/Tui/Screen/TuiUsageCommandVirtualTest.php:56` — `$state->contextWindow = 372000;` → `272000` (this is the PR #316 `/usage` fixture — confirmed merged/present).

OUT OF SCOPE (verified — do NOT change):
- No production code hardcodes 372000 or any derived value; context budgeting reads `context_window` from the model catalog at runtime.
- `config/hatfield.defaults.yaml` has NO gpt-5.6 entries (only 272000 examples for older models).
- No docs/README reference 372K. `.pi/plans/openai-codex-bridge.md` already shows 272K — LEAVE IT (historical planning artifact; task says don't rewrite history).
- The many `gpt-5.6-*` refs in pipeline/reasoning tests are model-NAME refs, not windows.
- `~/.hatfield/settings.yaml` does NOT define the openai-codex provider (only name refs in `ai.favorite_models` + `forks.model`) — no change there.

TEST SAFETY (assertions are value-agnostic; all safe to swap):
- ModelResolverTest → asserts reasoning-level set, not the window.
- ChildRunModelRoutingProvenanceRegressionTest → asserts model routing/provenance, not the window.
- TuiUsageCommandVirtualTest → asserts the `**Context (latest turn):**` LABEL exists; no computed-percentage assertion.

JUDGMENT CALLS: (1) Do NOT edit `.pi/plans/openai-codex-bridge.md`. (2) `castor test:llm-real` NOT required — pure metadata value swap, no LLM-visible/prompt/tool/streaming/provider-integration change.
- Recon completed (2026-07-30, task-explain phase): verified blast radius via repo-wide `git grep` for 372000|372K|372,000 — exactly 4 files / 6 occurrences. No production hardcodes, no docs 372K, no other gpt-5.6 model definitions. Read each affected test's assertions → all value-agnostic (safe to swap). Confirmed PR #316 `/usage` fixture (TuiUsageCommandVirtualTest:56) is present/merged. Checked `~/.hatfield/settings.yaml` via the `settings` tool (user-authorized read) — no openai-codex provider definition, only name refs in ai.favorite_models + forks.model; nothing to update there. Plan recall-verified (recall ids 6303b63fa497/a9e5ed5bce34/673960bebaf6) against fresh recon — byte-identical, no drift. Task left in TODO; NO branch/worktree/files touched. Implementation spec written into task summary for the implementing fork.

## Task workflow update - 2026-07-31T00:12:36.530Z
- Moved TODO → IN-PROGRESS.
- Created branch task/update-codex-context-windows-272k.
- Created worktree /home/ineersa/projects/agent-core-worktrees/update-codex-context-windows-272k.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/update-codex-context-windows-272k.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/update-codex-context-windows-272k.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/update-codex-context-windows-272k.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/update-codex-context-windows-272k.
- Summary: Claimed for implementation. Existing task-explain recon identifies four files and six literal substitutions; no new external surface or unresolved product decisions.

## Task workflow update - 2026-07-31T00:15:01.881Z
- Summary: Task-start recon in the fresh worktree confirmed four files but seven scalar occurrences (three values share `.hatfield/settings.yaml:70`, plus four test fixture values). The earlier 'six occurrences' count was a bookkeeping error; file scope and implementation plan are unchanged. No additional active 372K formatting was found and no external surface/ambiguity is introduced.
- Scout read the testing skill and tests/AGENTS.md, confirmed the worktree is clean, and found exactly the expected four active files. This is a metadata/fixture consistency change, not a TUI behavior implementation; existing virtual `/usage` coverage is the correct layer and no new TmuxHarness journey is warranted.

## Task workflow update - 2026-07-31T00:15:27.604Z
- Recorded fork run: fydcwja19bne
- Implementation fork fydcwja19bne launched in `/home/ineersa/projects/agent-core-worktrees/update-codex-context-windows-272k` with exact four-file/seven-value scope, mandatory testing-doc reads, Castor test/phpstan/cs-check validation, zero-obsolete-limit grep, and commit requirement.

## Task workflow update - 2026-07-31T00:18:40.871Z
- Recorded fork run: fydcwja19bne
- Validation: Fork confirmed it read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before QA.; `castor test` — PASS: 4362 tests, 15950 assertions.; `castor phpstan` — PASS: errors=0, file_errors=0.; `castor cs-check` — PASS: files_fixed=0.; Post-change tracked-active `git grep -nE '372000|372K|372,000'` with documented exclusions — PASS: zero matches (exit 1).; Parent verification: commit exists; worktree clean; diff is exactly 4 expected files, 5 insertions/5 deletions (inline YAML holds three scalar changes).; Not run by design in task-start: `castor check`, `castor test:llm-real`, PR/push/review steps.
- Summary: Implementation complete in commit `5155b421548eefec964c2bef32c08969651e68d6` (`Update Codex context windows to 272K`). Exactly seven `372000`→`272000` substitutions across the expected four files; max tokens, pricing, reasoning maps, unrelated providers, docs, APIs, and other behavior are unchanged. Parent verification confirmed a clean worktree, expected file list/stat, and zero active obsolete-limit matches. Existing virtual `/usage` coverage remains the correct proof because this only updates fixture metadata, not TUI behavior.

## Task workflow update - 2026-07-31T00:21:45.144Z
- Validation: Reviewer verdict: APPROVED; no findings. Reviewer read the testing skill and `tests/AGENTS.md`.; Commit: `5155b421548eefec964c2bef32c08969651e68d6`.; Task-to-PR focused `castor test` — PASS: 4362 tests, 15950 assertions.; Task-to-PR focused `castor deptrac` — PASS: violations=0, errors=0.; Task-to-PR focused `castor phpstan` — PASS: errors=0, file_errors=0.; Task-to-PR focused `castor cs-check` — PASS: files_fixed=0.
- Summary: Task-to-PR review approved. Reviewer confirmed exactly four files/seven scalar replacements, zero obsolete-limit hits, unchanged max tokens/pricing/reasoning/unrelated providers, no new surface or unnecessary complexity, and correct virtual proof layer for the metadata-only `/usage` fixture change.

## Task workflow update - 2026-07-31T00:23:47.995Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (104.7s).
- Pushed task/update-codex-context-windows-272k to origin.
- branch 'task/update-codex-context-windows-272k' set up to track 'origin/task/update-codex-context-windows-272k'.
- Created PR: https://github.com/ineersa/agent-core/pull/342
- Validation: `castor test`: PASS (4362 tests, 15950 assertions); `castor deptrac`: PASS (0 violations, 0 errors); `castor phpstan`: PASS (0 errors); `castor cs-check`: PASS (0 files fixed)
- Summary: Reviewer APPROVED with no findings. Focused Castor test, deptrac, phpstan, and cs-check all passed on clean commit `5155b421548eefec964c2bef32c08969651e68d6`.

## Task workflow update - 2026-07-31T03:00:28.828Z
- Moved CODE-REVIEW → DONE.
- Merged task/update-codex-context-windows-272k into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/settings.yaml                                               | 2 +-
 .../ChildRun/ChildRunModelRoutingProvenanceRegressionTest.php         | 4 ++--
 tests/CodingAgent/Config/ModelResolverTest.php                        | 2 +-
 tests/Tui/Screen/TuiUsageCommandVirtualTest.php                       | 2 +-
 4 files changed, 5 insertions(+), 5 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/update-codex-context-windows-272k.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/update-codex-context-windows-272k.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #342 state: MERGED (2026-07-31T03:00:07Z).; Integration checkout clean before merge.
- Summary: Confirmed GitHub PR #342 is merged at `82176be86b044bda3b71856d9448fbcd51adc52a`; completing task workflow and cleaning the task worktree.
