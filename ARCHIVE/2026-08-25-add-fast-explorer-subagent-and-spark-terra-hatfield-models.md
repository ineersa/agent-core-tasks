# Add fast Explorer subagent and Spark/Terra Hatfield models

## Goal
Add an `explorer` agent for both Pi user-agent discovery (`~/.agents`) and Hatfield (`~/.hatfield`), using GPT-5.3 Codex Spark with xhigh thinking and deliberately simple, evidence-first codebase reconnaissance instructions. Clarify in agent documentation/skills that Explorer is the fast mechanical recon agent while Scout is the smarter, more contextual/reasoning-capable investigator. Update relevant Pi and Hatfield skills and agent instructions where those surfaces catalog or explain agents. Add GPT-5.6 Terra and GPT-5.3 Codex Spark to Hatfield's model catalog/configuration using their canonical entries from Pi's catalog. Fast-review is explicitly deferred pending experience with Explorer; it is not part of this task.

## Acceptance criteria
- Pi discovers an `explorer` user agent from `~/.agents` configured for GPT-5.3 Codex Spark with `thinking: xhigh`.
- Hatfield discovers an equivalent `explorer` agent under its canonical `~/.hatfield` agent location.
- Explorer instructions are short and constrain it to fast, evidence-first codebase reconnaissance with exact paths/snippets and no architectural or safety judgments.
- Relevant Pi/Hatfield skills and agent-facing documentation distinguish Explorer (fast mechanical recon) from Scout (smarter contextual investigation and reasoning).
- Hatfield's model catalog includes canonical GPT-5.6 Terra and GPT-5.3 Codex Spark entries copied/mapped from Pi's catalog.
- Focused Castor validation passes for model catalog and agent discovery/documentation changes; no fast-review agent is added.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/434
PR Status: merged
Started: 2026-08-25T23:15:22.324Z
Completed: 2026-08-27T16:25:48.421Z

## Work log
- Created: 2026-08-25T23:15:14.452Z

## Task workflow update - 2026-08-25T23:15:22.324Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models.
- Summary: Task started with finalized scope: add Explorer for Pi and Hatfield, distinguish it from Scout in relevant skills/docs, add Spark and Terra to Hatfield from Pi's catalog, and defer fast-review.

## Task workflow update - 2026-08-25T23:21:13.313Z
- Summary: Scope clarification: update Pi and Hatfield task-workflow prompts/instructions so the orchestrator chooses Explorer for simple, bounded mechanical reconnaissance and Scout for harder investigations requiring broader context or reasoning.
- User clarified that task-workflow prompts in both Pi and Hatfield must explicitly route simple exploration to Explorer and harder/contextual exploration to Scout. Fast-review remains out of scope.

## Task workflow update - 2026-08-26T00:37:48.958Z
- Validation: Fork read and followed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`.; `castor test --filter=AgentsInitCommandTest` — passed, 3 tests / 127 assertions.; `castor docs:validate` — passed, 16 built-in documents.; `castor test --filter=AiCatalogTest` — passed, 6 tests / 25 assertions.; `castor test --filter=AppConfigTest` — passed, 29 tests / 85 assertions.; `castor cs-check` — passed, files_fixed=0.; `git diff --check HEAD^ HEAD` — passed.; Live Pi Explorer smoke using openai-codex/gpt-5.3-codex-spark xhigh — passed; returned exact bundled definition path, line range, model, and thinking values without scope expansion.
- Summary: Implementation committed as 0240ab7e8. Added bundled Hatfield Explorer (Codex Spark/xhigh), clarified Explorer-vs-Scout routing in Pi and Hatfield task workflows/prompts/skills/docs/root instructions, added canonical Spark and corrected Terra metadata in Hatfield catalog/project settings, and updated agents:init proof. Deployed active user Explorer definitions and role guidance under ~/.agents, ~/.hatfield/agents, ~/.pi/agent/skills, and ~/.hatfield/skills. Fast-review was not added.
- Active home deployment created `~/.agents/explorer.md` and `~/.hatfield/agents/explorer.md`, updated both active Scout descriptions, and updated Pi/Hatfield subagents skills with role-selection guidance. The active `~/.hatfield/ai-catalog.yaml` was intentionally left unchanged; authoritative catalog/model metadata is committed in agent-core and the project override exposes Spark after merge.
- Explorer smoke demonstrated the intended fast bounded behavior.

## Task workflow update - 2026-08-27T14:13:53.380Z
- Summary: Scope changed: remove the Explorer subagent and all Explorer-vs-Scout workflow/docs/skill changes from Pi and Hatfield. Retain the GPT-5.3 Codex Spark provider/model catalog setup (and existing Terra catalog correction) so Spark can be tried later for observational memory. Do not configure observational memory in this task.
- User reconsidered Explorer after implementation. Explorer definitions and routing guidance must be removed; provider model setup remains. Observational-memory model selection is future experimentation, not current implementation scope.

## Task workflow update - 2026-08-27T15:40:47.519Z
- Validation: Rollback fork read and followed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`.; `castor test --filter=AiCatalogTest` — passed, 6 tests / 25 assertions.; `castor test --filter=AppConfigTest` — passed, 29 tests / 85 assertions.; `castor cs-check` — passed, files_fixed=0.; Final `git diff --name-status 0240ab7e8^..HEAD` contains only `.hatfield/settings.yaml` and `config/ai-catalog.yaml`.; Final `git diff --check 0240ab7e8^..HEAD` — passed.; No `explorer.md` remains under `~/.agents` or `~/.hatfield/agents`; no Explorer references remain in active Pi/Hatfield subagent skills.
- Summary: Latest scope implemented in follow-up commit 97476e2b4: Explorer and all Explorer routing/docs/tests were removed from the final branch result and from active Pi/Hatfield home configuration. Final task-base diff contains only Hatfield provider model setup in `config/ai-catalog.yaml` and `.hatfield/settings.yaml`, retaining GPT-5.3 Codex Spark and corrected Terra metadata. Observational memory was not configured.
- Follow-up commit `97476e2b4 chore(models): retain Spark and Terra catalog metadata` reverses Explorer without rewriting history.
- Active home Explorer definitions were deleted; Scout descriptions and subagent skills were restored to their pre-Explorer state. Provider catalogs were not changed during home cleanup.

## Task workflow update - 2026-08-27T15:47:49.020Z
- Summary: Additional finalized configuration: set Hatfield forks to `openai-codex/gpt-5.6-terra` while preserving existing fork thinking; set both Pi and Hatfield Reviewer to `zai/glm-5.3` with medium thinking. Keep observational-memory configuration unchanged for later manual experimentation.
- User requested active and durable Hatfield fork/Reviewer model updates plus active Pi Reviewer update. Observational memory remains untouched.

## Task workflow update - 2026-08-27T15:50:39.432Z
- Summary: Test-scope correction: remove Reviewer model/thinking literals from `AgentsInitCommandTest`; those values are mutable bundled configuration, not a stable `agents:init` behavior contract. Keep parser validity/copy behavior tests only.
- User correctly challenged model-specific assertions in `AgentsInitCommandTest`. They were overfitted configuration checks and will be removed.

## Task workflow update - 2026-08-27T15:59:12.745Z
- Validation: `castor test --filter=AgentsInitCommandTest` after assertion removal — passed, 3 tests / 104 assertions.; `castor cs-check` after assertion removal — passed, files_fixed=0.; Final task-base diff contains only `.hatfield/settings.yaml`, `config/ai-catalog.yaml`, and `src/CodingAgent/Resources/agents/reviewer.md`; no test-file diff remains.; Active home readback: Hatfield forks Terra/high; Pi Reviewer GLM 5.3/medium; Hatfield Reviewer GLM 5.3/medium.
- Summary: Final implementation now retains only model/config changes: Spark/Terra provider metadata, Hatfield project forks on Terra/high, bundled Hatfield Reviewer on GLM 5.3/medium, and matching active home pins for Hatfield forks plus Pi/Hatfield Reviewers. Explorer remains removed and observational memory unchanged. Overfitted model-name assertions were removed in corrective commit ff7c14809.
- Corrective commit `ff7c14809 test(agents): keep reviewer pins configurable` removed mutable model/thinking literals from `AgentsInitCommandTest`.
- Active home model pins deployed without changing observational-memory settings or provider catalogs.

## Task workflow update - 2026-08-27T16:14:53.150Z
- Validation: Pre-transition worktree inspection: clean; final diff vs origin/main is exactly `.hatfield/settings.yaml`, `config/ai-catalog.yaml`, and `src/CodingAgent/Resources/agents/reviewer.md`; `git diff --check origin/main...HEAD` passed.; Reviewer subagent skipped by explicit user instruction.; Focused task-to-pr validation skipped by explicit user instruction; CODE-REVIEW transition Castor check remains mandatory.
- Summary: User explicitly directed task-to-pr without reviewer subagent and without duplicate focused local validation; proceeding directly to CODE-REVIEW transition, whose mandatory Castor check gate will validate before push/PR.
- Final commit at transition: `ff7c14809`. User requested direct CODE-REVIEW transition because move_task runs the test gate.

## Task workflow update - 2026-08-27T16:17:03.680Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (119.8s).
- Pushed task/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models to origin.
- branch 'task/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models' set up to track 'origin/task/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models'.
- Created PR: https://github.com/ineersa/agent-core/pull/434
- Summary: Moved directly to code review per user instruction; reviewer subagent and duplicate focused local validation were skipped. Final scope is model configuration only: Spark/Terra catalog setup, Hatfield Terra fork default, and GLM 5.3/medium Reviewer.

## Task workflow update - 2026-08-27T16:25:48.422Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models: ide_close_project returned isError.
- Merged task/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/settings.yaml                      |  4 ++--
 config/ai-catalog.yaml                       | 13 +++++++++++--
 src/CodingAgent/Resources/agents/reviewer.md |  4 ++--
 3 files changed, 15 insertions(+), 6 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-25-add-fast-explorer-subagent-and-spark-terra-hatfield-models.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #434 confirmed merged at 2026-08-27T16:25:22Z with merge commit f8c3242ecfcab76393da69f20a1b7ca986d78eba. Completing task and cleaning worktree.

## Task workflow update - 2026-08-27T16:27:29.930Z
- Validation: PR #434 state MERGED; GitHub merge commit `f8c3242ecfcab76393da69f20a1b7ca986d78eba`.; Post-merge `LLM_MODE=true castor check` — passed in 146.1s: deptrac, 4,853 unit/integration tests (19,790 assertions), controller replay, TUI, llm-real, phpstan, cs-check, docs validation, catalog version check, artifact integrity, leak check, and llama-proxy cache guard all OK.; Integration checkout `git status --short` — clean.; Task worktree removed; IDEA close reported degradation during transition but cleanup and exclusion removal succeeded.
- Summary: Task completed after PR #434 merge. Integration checkout merged/synced, task worktree removed, and post-merge full Castor gate passed.

## Task workflow update - 2026-08-29T16:09:39.023Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
