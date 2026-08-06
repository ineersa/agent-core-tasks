# SETTINGS-03: Add simple settings tool with SafeGuard mutation approval

## Goal
Add one parent-agent settings tool with a deliberately singular schema rather than batch changes. Proposed shape: `operation` (`read`, `set`, `remove`), dotted `path`, optional `scope`, and optional `value`. `read` is allowed without approval. SafeGuard understands this built-in tool's arguments and requires confirmation for every `set` or `remove` call.

There is currently a global CLI `--tools-excluded` option but no child-specific denylist: child definitions with no `tools:` list inherit everything. Add a generic child exclusion setting (for example `agents.subagent_excluded_tools`) and exclude both `settings` and the later `documentation` tool by default, even when a child definition would otherwise inherit/request them.

Dependencies: SETTINGS-01. Unblocks SETTINGS-04/05 child-visibility and settings-update work.

## Acceptance criteria
- The settings tool performs exactly one `read`, `set`, or `remove` operation per call using a dotted setting path.
- Reads can inspect effective, user, or project settings without SafeGuard approval.
- `set` writes one explicit sparse user/project override; `remove` removes one override so inheritance resumes; built-in defaults are never writable.
- Mutation writes are atomic, reject malformed arguments/invalid scope, and return the resulting value/source or a clear diagnostic.
- Hatfield SafeGuard classifies the settings tool by operation: `read` is allowed, while `set` and `remove` require Allow once/Block confirmation with no persistent Always allow bypass.
- Settings mutations fail closed when no approval channel is available.
- A child-specific excluded-tools policy exists; `settings` and `documentation` are excluded from every subagent by default, including agents whose omitted tool list would otherwise inherit all parent tools.
- Tool schemas and child/SafeGuard behavior have focused contract coverage following shared test conventions; LLM-visible tool schema validation uses the required Castor lane.

## Workflow metadata
Status: ARCHIVE
Branch: task/settings-03-settings-tool-safeguard
Worktree: /home/ineersa/projects/agent-core-worktrees/settings-03-settings-tool-safeguard
Fork run: s2arpelpjovf
PR URL: https://github.com/ineersa/agent-core/pull/299
PR Status: merged
Started: 2026-07-17T23:02:35.404Z
Completed: 2026-07-18T02:38:51.454Z

## Work log
- Created: 2026-07-16T17:35:03.662Z

## Task workflow update - 2026-07-17T18:06:32.612Z
- MANDATORY implementation constraints: REUSE SYMFONY COMPONENTS and existing project extension points; do not invent custom parsers/resolvers/routers/policy infrastructure when Symfony or the existing SafeGuard/tool APIs already provide the capability. DO NOT OVERENGINEER; choose the simplest viable implementation and avoid speculative abstractions, extra DTOs/services, or generalized frameworks. MINIMIZE CHANGES AND TESTS: no unrelated renames/refactors/churn; reuse existing paths and tests; add only the smallest tests proving read/set/remove and approval boundaries.

## Task workflow update - 2026-07-17T23:02:35.404Z
- Moved TODO → IN-PROGRESS.
- Created branch task/settings-03-settings-tool-safeguard.
- Created worktree /home/ineersa/projects/agent-core-worktrees/settings-03-settings-tool-safeguard.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/settings-03-settings-tool-safeguard.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/settings-03-settings-tool-safeguard.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/settings-03-settings-tool-safeguard.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/settings-03-settings-tool-safeguard.
- Summary: Starting SETTINGS-03 under the mandatory constraints: reuse existing Symfony/tool/SafeGuard seams, singular read/set/remove operations only, no overengineering, minimal production changes and focused tests.

## Task workflow update - 2026-07-17T23:11:42.221Z
- Recorded fork run: 2spn8ncmwltb
- Summary: Architecture scouts completed. Chosen minimal design: one singular SettingsTool plus narrow Symfony Yaml/Filesystem/PropertyAccess writer; explicit mutation scope; specialized SafeGuard classification using existing CustomDangerous kind with one-time-only approval; child exclusions applied at AgentToolPolicyResolver; no new visibility framework, slash command, TUI/controller tests, or raw settings access. Implementation fork launched with strict minimal test budget.

## Task workflow update - 2026-07-17T23:31:10.762Z
- Recorded fork run: 3l09renfsdqh
- Summary: Rejected partial implementation handoff at e0fd29ce4. Commit exists (18 files, +982/-13), but live LLM lane failed on view_image/write smokes and static verification found actionable issues: test-driven default AgentsConfig constructor arg, associative exclusion config accepted as list, hand-rolled recursive removal walker, incomplete settings approval spec, composite source mislabeled, and test isolation helper violation. Launched narrow correction fork preserving product behavior.

## Task workflow update - 2026-07-17T23:36:52.292Z
- Recorded fork run: 3l09renfsdqh
- Validation: castor clean:cleanup:workers:list — no stale candidates; castor test --filter=SettingsToolTest — 8 tests / 31 assertions OK; castor test --filter=SafeGuardClassifierTest — 25 / 33 OK; castor test --filter=SafeGuardToolCallHookTest — 38 / 115 OK; castor test --filter=AgentToolPolicyResolverTest — 5 / 15 OK; castor test --filter=AgentsConfigTest — 17 / 35 OK; castor test --suite=coding-agent — 2626 / 8659 OK; castor deptrac — 0 violations; castor phpstan — 0 errors; castor cs-check — clean; castor test:llm-real — 10 / 122 OK; proxy cache stable at 420 entries; no schema rejection or worker leaks
- Summary: Implementation complete at local HEAD 47dc3d824 (parent e0fd29ce4). SETTINGS-03 adds one sequential settings read/set/remove tool, Symfony YAML/Filesystem/strict PropertyAccess sparse writer with empty-parent pruning, SafeGuard Allow once/Block mutation approval and no-channel fail-closed behavior, and configurable child exclusions defaulting to settings/documentation. Correction removed test-driven production defaults, enforced actual list config, completed approval operation context, aligned composite source to mixed, and fixed test isolation. Worktree clean; cumulative diff vs origin/main: 21 files, +1020/-19. Not pushed/reviewed; task remains IN-PROGRESS per task-start workflow.

## Task workflow update - 2026-07-18T00:07:27.283Z
- Recorded fork run: 20v0xs1hm58r
- Summary: Task-to-PR reviewer at HEAD 47dc3d824 returned APPROVE WITH SUGGESTIONS. All acceptance criteria verified. One actionable low-severity bug: no-op remove returned changed=false but restart_required=true. Launched narrow two-line correction fork. Reviewer explicitly classified dotted empty segments and remaining documentation/cosmetic/test observations as non-actionable or optional; scope not expanded.
- Reviewer run during task-to-PR: APPROVE WITH SUGGESTIONS at 47dc3d824; one actionable no-op restart flag defect selected for correction.

## Task workflow update - 2026-07-18T00:13:48.835Z
- Recorded fork run: 20v0xs1hm58r
- Validation: Reviewer at ddc42fb18 — APPROVED, no actionable findings; castor test --filter=SettingsToolTest — 8 tests / 31 assertions OK; castor test --filter=SubagentLivePickerControllerTest — 17 / 56 OK after unrelated parallel failure; castor test --filter=MessengerSqliteImmediateTransactionMiddlewareTest — 4 / 19 OK after unrelated parallel timing failures; castor test:llm-real — 10 / 122 OK; castor deptrac — 0 violations; castor phpstan — 0 errors; castor cs-check — clean; castor test default attempt 1 — unrelated SubagentLivePickerControllerTest parallel failure; castor test default attempts 2/3 and coding-agent suite retry — unrelated MessengerSqliteImmediateTransactionMiddlewareTest 140ms timing threshold; focused class green
- Summary: Task-to-PR re-review at HEAD ddc42fb18 returned APPROVED with no actionable findings. The sole prior defect is fixed: missing override removal now returns changed=false without restart_required; real removals still require restart. Worktree clean. Focused feature/live/static validation passed. Default full `castor test` was attempted three times but hit only two previously observed unrelated parallel flakes: SubagentLivePickerControllerTest export feedback race once, and MessengerSqliteImmediateTransactionMiddlewareTest 140ms contention threshold twice; both complete affected classes pass sequentially. Proceeding to deterministic CODE-REVIEW gate, which uses its isolated 4-worker unit lane.
- Correction commit ddc42fb18: no-op settings remove no longer claims restart required (2 files, +2/-2).
- Final reviewer verdict at ddc42fb18: APPROVED, no actionable findings.

## Task workflow update - 2026-07-18T00:16:16.626Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (133.1s).
- Pushed task/settings-03-settings-tool-safeguard to origin.
- branch 'task/settings-03-settings-tool-safeguard' set up to track 'origin/task/settings-03-settings-tool-safeguard'.
- Created PR: https://github.com/ineersa/agent-core/pull/299
- Validation: Reviewer APPROVED at ddc42fb18; SettingsToolTest 8/31 OK; SafeGuardClassifierTest 25/33 OK; SafeGuardToolCallHookTest 38/115 OK; AgentToolPolicyResolverTest 5/15 OK; AgentsConfigTest 17/35 OK; llm-real 10/122 OK; deptrac 0 violations; phpstan 0 errors; cs-check clean
- Summary: Reviewer APPROVED at ddc42fb18. SETTINGS-03 implements singular read/set/remove settings operations, sparse Symfony-backed override writes, SafeGuard one-time mutation approval, and default child exclusion for settings/documentation. Focused feature, static, and live LLM validation passed; unrelated high-concurrency unit flakes were isolated with passing focused classes.

## Task workflow update - 2026-07-18T00:16:21.899Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/299
- Updated PR Status: open
- Validation: deterministic castor check — passed in 133.1s
- Summary: Moved to CODE-REVIEW at ddc42fb18. Deterministic castor check passed in 133.1s; branch pushed and PR #299 created.
- PR created: https://github.com/ineersa/agent-core/pull/299

## Task workflow update - 2026-07-18T01:10:06.613Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Reopened PR #299 to apply user-approved review changes: remove PropertyPathBuilder allocation in favor of a validated Symfony bracket path string; offer only Allow once/Block and remove settings-specific forged Always allow handling/tests; reduce LLM-visible settings prompting to the minimal tool contract plus one raw-file safety guideline.

## Task workflow update - 2026-07-18T01:10:55.508Z
- Recorded fork run: j2u5hgjsn25i
- Summary: Implementation fork launched for the three approved minimal review changes. Scope excludes generic enum validation and unrelated refactors; focused tests will be reduced rather than expanded, followed by Castor coding-agent/static gates and llm-real validation because LLM-visible tool metadata changes.

## Task workflow update - 2026-07-18T01:26:31.682Z
- Recorded fork run: j2u5hgjsn25i
- Validation: Fork read .agents/skills/testing/SKILL.md and tests/AGENTS.md before test work.; castor test: focused affected tests — 81 tests / 247 assertions OK.; castor test --suite=coding-agent — 2626 tests / 8660 assertions OK after unrelated SQLite timing flake; focused flake class 4/19 OK.; Parent: castor test — 4489 tests / 15207 assertions OK.; Parent: castor deptrac — 0 violations.; Parent: castor phpstan — 0 errors.; Parent: castor cs-check — 0 files to fix.; Parent: initial castor test:llm-real had one parallel timing failure in WriteFileToolE2eTest after assistant.message_started; focused sequential WriteFileToolE2eTest passed 1/12; full retry passed 10/122.
- Summary: Review iteration implemented at de8681d368e75d7a2bf524cb7462d979b4a4efa9 (4 files, +34/-52): PropertyPathBuilder removed for validated bracket-string conversion; settings approvals expose only Allow once/Block with non_persistent/forged-answer branches removed; SettingsTool prompt metadata reduced to one concise contract and one raw-file safety guideline. Reviewer APPROVED with zero actionable findings and confirmed all six inline comments resolved.

## Task workflow update - 2026-07-18T01:30:19.466Z
- Validation: CODE-REVIEW transition castor check: failed only cache-growth guard (442→444); no code/test failure reported.; castor test:llm-real warmup — 10/122 OK, cache 444→445.; castor test:llm-real stability verification — 10/122 OK, cache remained 445→445.
- Summary: First CODE-REVIEW transition gate passed functional lanes but failed deterministic llama-proxy cache guard because the LLM-visible prompt change introduced uncached requests (cache 442→444). Warmed with full castor test:llm-real runs until stable: 10/122 OK, cache 445→445. Ready to rerun the deterministic transition.

## Task workflow update - 2026-07-18T01:44:35.769Z
- Validation: castor check — quality OK in 307.2s: test 4485/15195, controller-replay 8/112, TUI 37/193, llm-real 10/122, deptrac/phpstan/cs clean, cache guard 446→446, artifact integrity OK, leak check OK. Report: var/reports/qa-20260718-014218-583795-ab88354f.
- Summary: Manual deterministic castor check requested by user passed completely after prior check-specific cache warmup. Cache remained stable at 446→446, confirming the second transition had populated the final check-only request key.

## Task workflow update - 2026-07-18T01:52:13.607Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (127.8s).
- Pushed task/settings-03-settings-tool-safeguard to origin.
- branch 'task/settings-03-settings-tool-safeguard' set up to track 'origin/task/settings-03-settings-tool-safeguard'.
- PR already exists: https://github.com/ineersa/agent-core/pull/299
- Validation: castor check — quality OK in 307.2s: test 4485/15195, controller-replay 8/112, TUI 37/193, llm-real 10/122, deptrac/phpstan/cs clean, cache guard 446→446, artifact integrity and leak checks OK.
- Summary: PR #299 feedback revision complete at de8681d368e75d7a2bf524cb7462d979b4a4efa9. Reviewer APPROVED with zero actionable findings. User-requested deterministic castor check passed with stable proxy cache 446→446; branch is ready to push and PR update.

## Task workflow update - 2026-07-18T02:03:18.671Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Reopened PR #299 for one user-approved prompt refinement: replace the abstract settings safety guideline with explicit MUST/NEVER wording naming the settings tool, raw settings files, generic file tools, and common bash inspection commands.

## Task workflow update - 2026-07-18T02:03:34.162Z
- Recorded fork run: mqlsw2zmyx0g
- Summary: Launched one-line implementation fork for the approved explicit MUST/NEVER settings-tool guideline. No schema/runtime/test/docs changes; prompt-compatible Castor validation requested.

## Task workflow update - 2026-07-18T02:06:54.361Z
- Recorded fork run: mqlsw2zmyx0g
- Summary: Prompt guideline strengthened in one-line commit 8683152dc. Worktree `.hatfield/settings.yaml` currently has user-owned uncommitted changes from manual SettingsTool testing; explicitly do not restore, edit, stash, or otherwise touch that file. CODE-REVIEW transition must wait until the user finishes testing and makes the worktree clean.

## Task workflow update - 2026-07-18T02:21:21.511Z
- Recorded fork run: s2arpelpjovf
- Summary: Launched minimal implementation fork to encode successful SettingsTool results as TOON using the existing library and adapt only SettingsToolTest via decoding. User-owned `.hatfield/settings.yaml` changes are explicitly protected and excluded from all fork operations/commit.

## Task workflow update - 2026-07-18T02:25:08.406Z
- Validation: castor test --filter=SettingsToolTest — OK (8 tests, 38 assertions); castor phpstan — 0 errors; castor cs-check — clean; castor test:llm-real — OK (10 tests, 122 assertions)
- Summary: Implemented user-requested TOON output at commit 2883cf2bce06bf43f9ed1637195fea56ff9ac0be. Successful SettingsTool results encode once at the outer boundary with existing HelgeSverre\Toon\Toon; internal arrays and exception behavior remain unchanged. Existing tests decode TOON through one test-local helper. No reviewer will be run per user direction. User-owned `.hatfield/settings.yaml` remains dirty and untouched; do not transition until user testing is complete/clean.

## Task workflow update - 2026-07-18T02:30:12.931Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (129.1s).
- Pushed task/settings-03-settings-tool-safeguard to origin.
- branch 'task/settings-03-settings-tool-safeguard' set up to track 'origin/task/settings-03-settings-tool-safeguard'.
- PR already exists: https://github.com/ineersa/agent-core/pull/299
- Validation: castor test --filter=SettingsToolTest — OK (8 tests, 38 assertions); castor phpstan — 0 errors; castor cs-check — clean; castor test:llm-real — OK (10 tests, 122 assertions)
- Summary: Final user-requested revisions committed through 2883cf2bce06bf43f9ed1637195fea56ff9ac0be: strengthened mandatory settings-tool guidance and changed successful SettingsTool output to TOON using the existing library. User explicitly requested CODE-REVIEW transition without an additional reviewer.

## Task workflow update - 2026-07-18T02:38:51.455Z
- Moved CODE-REVIEW → DONE.
- Merged task/settings-03-settings-tool-safeguard into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/settings.yaml                            |   5 +
 config/hatfield.defaults.yaml                      |   7 +
 config/services.yaml                               |   8 +
 docs/settings.md                                   |  19 +-
 .../Agent/Execution/AgentToolPolicyResolver.php    |  12 +
 src/CodingAgent/Config/AgentsConfig.php            |  49 +++-
 src/CodingAgent/Config/SettingsOverrideWriter.php  | 200 ++++++++++++++++
 src/CodingAgent/Config/SettingsValueResolver.php   |  20 +-
 .../SafeGuard/Classifier/SafeGuardClassifier.php   |  38 ++-
 .../Builtin/SafeGuard/SafeGuardConfig.php          |   3 +
 .../Builtin/SafeGuard/SafeGuardExtension.php       |   1 +
 .../Builtin/SafeGuard/SafeGuardToolCallHook.php    |  50 +++-
 src/CodingAgent/Tool/SettingsTool.php              | 266 +++++++++++++++++++++
 .../Execution/AgentToolPolicyResolverTest.php      |  35 ++-
 ...05BareAgentsEffectiveContextIntegrationTest.php |   2 +-
 .../Execution/SubagentExecutionServiceTest.php     |   2 +-
 .../SubagentPromptUserContextContractTest.php      |   2 +-
 tests/CodingAgent/Config/AgentsConfigTest.php      |  28 +++
 .../Classifier/SafeGuardClassifierTest.php         |  27 +++
 .../SafeGuard/SafeGuardToolCallHookTest.php        |  82 +++++++
 tests/CodingAgent/Tool/SettingsToolTest.php        | 197 +++++++++++++++
 21 files changed, 1027 insertions(+), 26 deletions(-)
 create mode 100644 src/CodingAgent/Config/SettingsOverrideWriter.php
 create mode 100644 src/CodingAgent/Tool/SettingsTool.php
 create mode 100644 tests/CodingAgent/Tool/SettingsToolTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/settings-03-settings-tool-safeguard.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/settings-03-settings-tool-safeguard.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #299 merged; completing SETTINGS-03 and syncing the integration checkout.

## Task workflow update - 2026-07-18T02:41:14.326Z
- Validation: LLM_MODE=true castor check — OK in 306.5s: test 4485/15202, controller-replay 8/112, TUI 37/193, llm-real 10/122, deptrac/phpstan/cs-check clean, llama-proxy cache stable 452→452, artifact integrity and leak checks OK
- Summary: Post-merge integration validation passed; integration checkout synced and clean.

## Task workflow update - 2026-08-06T20:59:35.148Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
