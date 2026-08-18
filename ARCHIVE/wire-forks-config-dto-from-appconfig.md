# Wire ForksConfigDTO from AppConfig so fork model/thinking settings apply

## Goal
## Bug

`forks.model` / `forks.thinking_level` in project `.hatfield/settings.yaml` are silently ignored: `fork` launches always resolve to the **parent session's** model/reasoning. Reproduced twice on 2026-08-14 (typed-serialization worktree, session run `2`, `.hatfield/sessions/2/events.jsonl`):

- Original fork (run `3e0f716d-…`, artifact `agent_c39c9d7a7228fa6d`) → `"model":"runpod/Qwen3.8-27B","reasoning":"high","context_window":262144`
- Probe fork (run `313a2101-…`, artifact `agent_21305ad13610f6c1`) → same values

Expected (project settings at the time): `deepseek/deepseek-v4-flash` / `xhigh`.

## Root cause (verified against code + compiled container)

- `src/CodingAgent/Config/ForksConfigDTO` is auto-registered as a plain service by the broad `Ineersa\CodingAgent\:` resource auto-registration (`config/services.yaml` ~75-76). Its constructor is all-defaults (`model = null`, `thinkingLevel = null`), so the container builds an **empty** instance.
- The compiler inlines that empty DTO into the fork launch graph — see compiled container `getDeferredSubagentBatchLaunchServiceService.php:86-91`: `new ForkRuntimeConfigResolver(new ForksConfigDTO(NULL, NULL, new ChildExtensionsConfigDTO()), …)`.
- `ForkRuntimeConfigResolver` (chain stable since `1104136b5`): `firstNonEmpty($explicitModel, $this->forksConfig->model, $parentModel)` — with an empty DTO it always falls to `$parentModel`. Thinking likewise falls to parent reasoning.
- The intended wiring exists but is **unused**: `ForksConfigDTO::fromAppConfig()` (returns `$appConfig->forks`); `AppConfig::denormalizeForksConfig()` correctly reads the settings via `ForksConfigDTO::fromRaw()`.
- **Pre-existing on `origin/main`** (not caused by the typed-serialization branch): all-defaults DTO since `abce25f9f` (Jun 29), resolver since `1104136b5` (Jul 16), main's `services.yaml` has **zero** `ForksConfigDTO` references, identical auto-registration block.
- **Not affected:** `forks.extensions` — `ChildExtensionSelectionService` reads the real `AppConfig->forks->extensions` directly (so the project's fork extension allowlist works).
- **Test gap that let it ship:** all existing tests construct `ForksConfigDTO` manually with values (`tests/CodingAgent/Agent/Tool/ForkToolContractTest.php:96-140`, `tests/CodingAgent/Extension/ChildExtensionSelectionServiceTest.php:119-135`); container-builder tests pass explicit overrides (`tests/CodingAgent/Agent/Fork/ForkChildStartRunInputCompositionTest.php:37-60, 92-107`). Nothing asserts the container wiring.

## Scope

1. **Fix:** add the missing service definition in `config/services.yaml`, next to `CompactionConfig`/`PromptsConfig` (same factory pattern, ~line 181-184):
   ```yaml
   Ineersa\CodingAgent\Config\ForksConfigDTO:
     factory: ['Ineersa\CodingAgent\Config\ForksConfigDTO', 'fromAppConfig']
     arguments:
       - '@Ineersa\CodingAgent\Config\AppConfig'
   ```
   Verify `config/services_test.yaml` (imports production config; likely no duplicate needed) and that no other consumer of the `ForksConfigDTO` service expects the empty instance.
2. **Regression test (container-level):** assert the `ForksConfigDTO` service resolves from the settings' `forks` section (e.g. seed `forks.model`/`forks.thinking_level` via the test settings/config fixture pattern and assert the service values), so the DI wiring is covered, not just the resolver logic. Must fail before the fix, pass after.
3. **Docs:** check `docs/settings-agents.md` `forks.model`/`forks.thinking_level` wording — it should describe the actual (now-working) semantics: tool param → `forks.*` → parent. Fix only if inaccurate.

## Out of scope

- Changing the resolution precedence (explicit → `forks.*` → parent stays).
- Subagent model resolution (separate machinery — `SubagentRunMetadataReader` path; touch only if the regression test exposes a shared defect).
- PR #381 / commit `4bdf6ae86` (typed-serialization + task_list CANCELLED default) — unrelated.

## Validation notes

- Focused proof: the new container test (red → green).
- Live proof (best effort): in a session whose checkout includes the fix, launch a fork **without** `model`/`thinking` and confirm the parent-side fork event record shows the configured model/reasoning (event payload fields `model`, `reasoning`, `context_window` in the session `events.jsonl`).
- QA via Castor only. **Session-env caveat (tracked separately in `make-castor-qa-safe-to-run-from-hatfield-sessions`):** when running `castor check`/`castor test` from inside a Hatfield session, unset the six `HATFIELD_*_TRANSPORT_DSN` vars (env leak into the unit-test lane) and be aware the check rebuilds `var/tmp/phar/hatfield.phar` under a running session.

## Acceptance criteria
- config/services.yaml defines ForksConfigDTO via the fromAppConfig factory; regenerated container no longer inlines an all-null ForksConfigDTO into ForkRuntimeConfigResolver.
- New container-level test proves the ForksConfigDTO service reflects settings forks.model/forks.thinking_level — fails without the wiring, passes with it.
- Existing fork/subagent test suites remain green; focused Castor validation (test, phpstan, cs-check) green per the testing skill.
- Live fork launched without model params resolves to the configured forks.model/reasoning (verified from the parent-side fork event record), or explicit deferral with reason if live validation is not feasible in the session.

## Workflow metadata
Status: ARCHIVE
Branch: task/wire-forks-config-dto-from-appconfig
Worktree: /home/ineersa/projects/agent-core-worktrees/wire-forks-config-dto-from-appconfig
Fork run: 2yfjfh70vxyx
PR URL: https://github.com/ineersa/agent-core/pull/383
PR Status: merged
Started: 2026-08-14T21:43:27.632Z
Completed: 2026-08-14T23:12:31.623Z

## Work log
- Created: 2026-08-14T21:12:59+00:00

## Task workflow update - 2026-08-14T21:43:27.632Z
- Moved TODO → IN-PROGRESS.
- Created branch task/wire-forks-config-dto-from-appconfig.
- Created worktree /home/ineersa/projects/agent-core-worktrees/wire-forks-config-dto-from-appconfig.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/wire-forks-config-dto-from-appconfig.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/wire-forks-config-dto-from-appconfig.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/wire-forks-config-dto-from-appconfig.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/wire-forks-config-dto-from-appconfig.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/wire-forks-config-dto-from-appconfig/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/wire-forks-config-dto-from-appconfig.
- Summary: Claimed for task-start implementation; root cause and scope are fully specified in the task.

## Task workflow update - 2026-08-14T21:49:29.281Z
- Recorded fork run: hxivoyqm8kxx
- Summary: Dispatched implementation fork in the exact task worktree. Scope: explicit ForksConfigDTO factory wiring, one isolated container regression test, and minimal forks precedence wording correction; no resolver/subagent/TUI/provider changes.
- Scouts confirmed the only production injection consumer is ForkRuntimeConfigResolver; config/services_test.yaml inherits production wiring and needs no duplicate.
- Scouts read and followed .agents/skills/testing/SKILL.md and tests/AGENTS.md. Recommended modern IsolatedKernelTestCase pre-boot settings hook and identity assertion against AppConfig->forks.
- Specification fidelity gate: no new external surface; only restores documented forks.model/forks.thinking_level precedence. Full castor check and live worktree fork proof are deferred from task-start per phase boundaries.

## Task workflow update - 2026-08-14T21:54:01.218Z
- Recorded fork run: hxivoyqm8kxx
- Validation: PASS: castor test --filter=ForksConfigServiceTest (1 test, 3 assertions); PASS: castor cs-check; PASS: castor docs:validate; FAIL (pre-existing, reproduced by fork with task changes stashed): castor test — HatfieldDocsToolTest calls missing AppResourceLocator::getAppRoot(); FAIL (same pre-existing blocker): castor phpstan — undefined AppResourceLocator::getAppRoot() at HatfieldDocsTool.php:129; NOT RUN by design in task-start: castor check; reserved for task-to-pr; DEFERRED: live fork without model/thinking, because current parent session/tool does not run from this task worktree
- Summary: Implementation committed as e00c5b21ea3d35daf1b98664f852138e0abf2ee9. Verified clean worktree and exactly 3 expected files changed: config/services.yaml, docs/settings-agents.md, and new tests/CodingAgent/Config/ForksConfigServiceTest.php. The service now uses ForksConfigDTO::fromAppConfig; the test proves configured values and identity with AppConfig->forks; docs state explicit -> forks.* -> parent fallback. Live fork proof deferred because the parent session/tool is not executing from the task worktree.
- Parent verified commit e00c5b21e exists, worktree is clean, and diff is 53 insertions/2 deletions across only the 3 scoped files.
- Fork confirmed .agents/skills/testing/SKILL.md and tests/AGENTS.md were read and followed. No TUI/provider compatibility proof applies; this is container wiring only.
- Pre-existing getAppRoot failure was left untouched as out of scope; it may block task-to-pr castor check until resolved elsewhere.

## Task workflow update - 2026-08-14T22:12:43.559Z
- Recorded fork run: mle6w6gc397t
- Summary: Dispatched a verification-only fork to temporarily remove the ForksConfigDTO service block, capture focused Castor red evidence, restore committed wiring, capture green evidence, and leave HEAD/worktree unchanged and clean.

## Task workflow update - 2026-08-14T22:14:04.388Z
- Recorded fork run: mle6w6gc397t
- Validation: RED without only the ForksConfigDTO factory block: castor test --filter=ForksConfigServiceTest exited 1 — model was null instead of deepseek/deepseek-v4-flash at ForksConfigServiceTest.php:27 (1 test, 1 assertion, 1 failure); GREEN after restoring the factory block: castor test --filter=ForksConfigServiceTest exited 0 — OK (1 test, 3 assertions); Final verification: HEAD e00c5b21ea3d35daf1b98664f852138e0abf2ee9; git status and git diff empty
- Summary: Explicit red→green proof completed with no persistent changes. Parent verified HEAD remains e00c5b21ea3d35daf1b98664f852138e0abf2ee9 and status/diff are clean.
- Verification fork temporarily removed only config/services.yaml ForksConfigDTO factory wiring, captured the expected failing container test, restored the committed file, and captured the passing test. No commit/amend/push occurred.

## Task workflow update - 2026-08-14T22:22:36.419Z
- Validation: REVIEWER APPROVED: no correctness, DI, test-isolation, security, scope, or complexity blockers; Reviewer independently ran castor test --filter=ForksConfigServiceTest: PASS (1 test, 3 assertions); Reviewer confirmed .agents/skills/testing/SKILL.md and tests/AGENTS.md were read and followed
- Summary: Reviewer APPROVED commit e00c5b21e with no blocking findings. Specification fidelity passed: no new setting/API/command/storage, only the required service semantics, container regression test, and docs wording. Reviewer independently passed the focused test (1 test, 3 assertions).
- Reviewer identified branch staleness as the source of the unrelated getAppRoot failure: current origin/main contains the missing method in 82443369d. Branch should merge origin/main before deterministic CODE-REVIEW gate; task files have low conflict risk.

## Task workflow update - 2026-08-14T22:23:06.245Z
- Recorded fork run: 2yfjfh70vxyx
- Summary: Dispatched branch-sync fork to merge current origin/main before task-to-pr validation, preserving the reviewed task diff and resolving the unrelated stale-base getAppRoot blocker.

## Task workflow update - 2026-08-14T22:26:33.883Z
- Recorded fork run: 2yfjfh70vxyx
- Validation: PASS: castor test — 4472 tests, 17070 assertions; PASS: castor deptrac — 0 violations, 0 errors; PASS: castor phpstan — 0 errors; PASS: castor cs-check — 0 files fixed; PASS: castor test:llm-real — 13 tests, 144 assertions; PASS: castor test --filter=ForksConfigServiceTest after origin/main merge — 1 test, 3 assertions; Worktree clean at 336d7c03477d78f6284b59a519c1849796b27555; diff vs origin/main is 3 files, 53 insertions, 2 deletions
- Summary: Merged current origin/main cleanly as 336d7c03477d78f6284b59a519c1849796b27555; task-only diff remains exactly 3 scoped files. Reviewer APPROVED. Focused task-to-pr validation is green, including live LLM lane because the change affects fork model routing.
- Task-to-pr local validation completed after syncing origin/main. Six HATFIELD_*_TRANSPORT_DSN variables were unset only in the castor test shell per task/session caveat; active session identity was not touched.

## Task workflow update - 2026-08-14T22:28:52.725Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (122.2s).
- Pushed task/wire-forks-config-dto-from-appconfig to origin.
- branch 'task/wire-forks-config-dto-from-appconfig' set up to track 'origin/task/wire-forks-config-dto-from-appconfig'.
- Created PR: https://github.com/ineersa/agent-core/pull/383

## Task workflow update - 2026-08-14T22:28:59.797Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/383
- Updated PR Status: open
- Validation: PASS: deterministic castor check (122.2s)
- Summary: Moved to CODE-REVIEW after reviewer approval, clean origin/main sync, focused Castor validation, and deterministic castor check pass. Branch pushed and PR #383 created.

## Task workflow update - 2026-08-14T23:12:31.623Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/wire-forks-config-dto-from-appconfig.
- Merged task/wire-forks-config-dto-from-appconfig into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                               |  5 +++
 docs/settings-agents.md                            |  4 +-
 .../CodingAgent/Config/ForksConfigServiceTest.php  | 46 ++++++++++++++++++++++
 3 files changed, 53 insertions(+), 2 deletions(-)
 create mode 100644 tests/CodingAgent/Config/ForksConfigServiceTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/wire-forks-config-dto-from-appconfig.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/wire-forks-config-dto-from-appconfig.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #383 confirmed merged on GitHub at 2026-08-14T22:35:41Z (merge commit e819a49f98907f6ca8496fcfa0979900a8704688). Proceeding with integration merge/sync and worktree cleanup.

## Task workflow update - 2026-08-14T23:16:05.548Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/383
- Updated PR Status: merged
- Validation: PASS: LLM_MODE=true castor check (QA run qa-20260814-231236-540192-d159bf54); PASS lanes: deptrac; 4472 tests/17070 assertions; controller replay 12/165; TUI 36/300; llm-real 13/144; phpstan; cs-check; docs validation; PASS: QA artifact integrity, leak check, exact-run cache cleanup, and llama-proxy cache guard (261 → 261); Integration checkout clean and synchronized: main == origin/main at 6d6f110dd
- Summary: PR #383 merged and task integrated into main. Post-merge LLM_MODE=true castor check passed; integration checkout is clean and synchronized with origin/main at 6d6f110dd. Git worktree registration was removed and IDEA exclusions cleaned.
- Cleanup note: git worktree registration and tracked worktree content were removed. A 16KB `.hatfield/logs/agent-2026-08-14.log` directory was recreated at the old path by an exiting PHAR process and records `Revolt\EventLoop\UncaughtThrowable` missing during teardown; `lsof +D` found no remaining process handles. Preserved as lifecycle evidence rather than deleting it silently.

## Task workflow update - 2026-08-15T17:16:43.329Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
