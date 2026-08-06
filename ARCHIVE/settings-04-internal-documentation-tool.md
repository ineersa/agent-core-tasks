# SETTINGS-04: Bundle maintained docs and add internal documentation tool

## Goal
Make maintained Hatfield documentation available through a read-only parent-agent tool in source and PHAR runs. Add one-sentence YAML frontmatter descriptions to maintained documentation files so list output explains each document. The tool lists logical document IDs and loads bounded chunks by ID; it never accepts arbitrary filesystem paths.

Delete `docs/archive/` from the working tree, repair remaining references to archived paths, and include maintained `docs/` in the PHAR. The documentation tool is excluded from subagents through the child exclusion policy from SETTINGS-03.

Dependencies: SETTINGS-03 for child exclusion semantics.

## Acceptance criteria
- Maintained documentation files have consistent frontmatter containing a one-sentence description used by the catalog.
- The documentation tool lists document IDs/titles/descriptions and loads a selected document with bounded offset/limit behavior.
- Only catalogued files below the maintained docs root can be loaded; traversal, arbitrary paths, unsupported files, and archive access are rejected.
- The same resource service works from a source checkout and through `phar://` at PHAR runtime.
- Maintained `docs/` are included in the PHAR; `docs/archive/` is deleted and remaining repository references are repaired or intentionally removed.
- The documentation tool is available to the primary agent and excluded from all subagents by default.
- Focused catalog/resource/tool and PHAR behavior is covered through Castor-based validation.

## Workflow metadata
Status: ARCHIVE
Branch: task/settings-04-internal-documentation-tool
Worktree: /home/ineersa/projects/agent-core-worktrees/settings-04-internal-documentation-tool
Fork run: qwsgri9r6bsi
PR URL: https://github.com/ineersa/agent-core/pull/300
PR Status: merged
Started: 2026-07-18T17:15:14.830Z
Completed: 2026-07-18T21:45:34.426Z

## Work log
- Created: 2026-07-16T17:35:03.663Z

## Task workflow update - 2026-07-17T18:06:32.628Z
- MANDATORY implementation constraints: REUSE SYMFONY COMPONENTS and existing project extension points; do not invent custom discovery/loading/metadata infrastructure when Symfony components or existing resource-location APIs suffice. DO NOT OVERENGINEER; choose the simplest viable implementation and avoid speculative abstractions, extra DTOs/services, or generalized frameworks. MINIMIZE CHANGES AND TESTS: no unrelated renames/refactors/churn; add only the smallest tests proving catalog/load safety and PHAR resource access.

## Task workflow update - 2026-07-18T17:09:01.790Z
- Summary: Revised scope approved: do not bundle the complete `docs/` directory. Keep `docs/` as canonical documentation and add a curated `internal-docs/` symlink manifest containing only agent-relevant Hatfield documents. Bundle only materialized regular-file copies of `internal-docs/` in the PHAR. Rename the tool from broad `documentation` to `hatfield_docs`, and update the default subagent exclusion accordingly.
- Approved curated documentation layout: `internal-docs/` contains symlinks to selected canonical files under `docs/`; no duplicated maintained content.
- Initial curated set: settings.md, agents.md, compaction.md, mcp.md, prompt-templates.md, background-processes.md, file-rewind.md, hitl-and-approvals.md. Explicitly exclude maintainer/development docs such as datadog, QA metrics, LLM replay, PHAR packaging, TUI/runtime architecture/testing, POCs, and archive files.
- Box does not support symlinks. Source runs may read the symlink manifest directly, but Castor PHAR staging must dereference `internal-docs/` into regular files before Box compilation. `box.json` must include only `internal-docs/`, not `docs/`.
- Tool contract name is `hatfield_docs` with narrow list/read semantics over catalogued document IDs only; no arbitrary filesystem paths.
- SETTINGS-03 left `documentation` in the default `agents.subagent_excluded_tools`; SETTINGS-04 must replace it with `hatfield_docs` in defaults/config documentation and related tests.
- Archive deletion/reference repair is removed from SETTINGS-04 scope because archives are outside the curated package root; handle repository archive cleanup separately if desired.
- Acceptance interpretation superseding the original wording: frontmatter is required only on curated canonical documents; source and PHAR behavior must use the same curated catalog; PHAR contains regular `internal-docs/*.md` entries and does not bundle `docs/` or archives; focused safety/resource/tool/PHAR proof remains required.

## Task workflow update - 2026-07-18T17:11:20.961Z
- Summary: Scope correction: deleting `docs/archive/` and repairing/removing remaining references stays in SETTINGS-04. Archived docs are safe to delete and are never part of the curated `internal-docs/` catalog or PHAR bundle.
- User explicitly rejected removing archive cleanup from scope: SETTINGS-04 must delete `docs/archive/` and repair or intentionally remove all remaining repository references to archived paths.

## Task workflow update - 2026-07-18T17:14:32.597Z
- Summary: Final approved built-in Hatfield documentation catalog contains exactly eight documents: agents, background-processes, compaction, hitl-and-approvals, mcp, prompt-templates, session-storage, and settings.
- Curated `internal-docs/` symlink set finalized as: `agents.md`, `background-processes.md`, `compaction.md`, `hitl-and-approvals.md`, `mcp.md`, `prompt-templates.md`, `session-storage.md`, `settings.md`. This replaces the earlier proposed set: include `session-storage.md`; do not include `file-rewind.md`.

## Task workflow update - 2026-07-18T17:15:14.830Z
- Moved TODO → IN-PROGRESS.
- Created branch task/settings-04-internal-documentation-tool.
- Created worktree /home/ineersa/projects/agent-core-worktrees/settings-04-internal-documentation-tool.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/settings-04-internal-documentation-tool.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/settings-04-internal-documentation-tool.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/settings-04-internal-documentation-tool.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/settings-04-internal-documentation-tool.
- Summary: Starting implementation with the approved curated `internal-docs/` symlink catalog, `hatfield_docs` tool name, PHAR materialization, and archive deletion/reference repair.

## Task workflow update - 2026-07-18T17:31:13.431Z
- Summary: Architecture scout completed. Chosen minimal design: one `HatfieldDocsTool` using a fixed approved ID list, existing AppResourceLocator, MarkdownFrontmatterExtractor, Symfony Yaml, ToolRuntime, and TOON; no DTO/catalog service/interface. Source symlinks are dereferenced only during PHAR staging. Focused tool, exclusion, and PHAR proof only; no TUI work.
- Worktree: `/home/ineersa/projects/agent-core-worktrees/settings-04-internal-documentation-tool`.
- Scout confirmed Box filters/rejects symlinks; `.castor/helpers.php` must materialize `internal-docs/` with `cp -aL`, while `box.json` includes only `internal-docs`.
- Scout confirmed archive references outside `docs/archive/` are limited to README.md and `.pi/plans/extension-tool-hooks-safeguard-plan.md`.
- Scout read mandatory testing docs. Test thesis: only the eight approved IDs can be listed/read; chunk bounds preserve metadata/content semantics; PHAR has regular internal-doc files readable through AppResourceLocator and excludes docs/archive.

## Task workflow update - 2026-07-18T17:32:14.739Z
- Recorded fork run: asco74ksuf06
- Summary: Implementation fork launched in task worktree with exact minimal scope: one HatfieldDocsTool, fixed eight-ID catalog, source symlinks, PHAR dereference staging, default child exclusion rename, frontmatter, archive deletion/reference repair, and focused Castor proof only.

## Task workflow update - 2026-07-18T17:44:29.392Z
- Recorded fork run: asco74ksuf06
- Validation: castor test --filter='HatfieldDocsToolTest|AgentsConfigTest|AgentToolPolicyResolverTest' — OK (32 tests, 97 assertions); castor phar:build — OK (9.2 MB); HATFIELD_BINARY_PATH=... castor test --filter=PharSmokeTest — OK (5 tests, 57 assertions); source links materialized as regular PHAR files, no docs/archive; castor deptrac — 0 violations; castor phpstan — 0 errors; castor cs-check — clean; castor test:llm-real — not fully green: new tool catalog warmed proxy cache 471→493; focused sequential tool/controller tests passed 5/51, but full parallel RewindBranchLiveE2eTest repeatedly returned `crimson` instead of expected `apple` with cache stable 493→493. No SETTINGS-04 product failure observed, but this may block the later deterministic gate.
- Summary: Implementation completed and committed at 8b6440c4b4e96b265fb6bbe6ff560fed9bb6f684. Verified clean worktree, expected 63-file diff (+564/-3936), exactly eight source symlinks, archive deletion, and no remaining external `docs/archive` references except the intentional PHAR absence assertion. Product scope is complete. No review, push, PR, full castor check, or task transition performed.

## Task workflow update - 2026-07-18T17:57:01.001Z
- Recorded fork run: yx8vaa8ked6l
- Summary: Reviewer returned APPROVE WITH SUGGESTIONS with zero blockers/actionable findings; NTH polish skipped under minimal-diff mandate. Focused castor test/deptrac/phpstan/cs-check passed. Full llm-real remains blocked by a stable bad RewindBranch cassette (`crimson`). Launched a one-line marker-bump fork to re-record that unchanged live scenario under the new permanent tool catalog.

## Task workflow update - 2026-07-18T18:00:13.042Z
- Validation: castor test — OK (4500 tests, 15306 assertions); castor deptrac — 0 violations; castor phpstan — 0 errors; castor cs-check — clean; castor test:llm-real after marker refresh — OK (10 tests, 122 assertions); second consecutive full run OK with llama-proxy cache stable 501→501; castor phar:build — OK (9.2 MB); HATFIELD_BINARY_PATH=... castor test --filter=PharSmokeTest — OK (5 tests, 57 assertions)
- Summary: Final review at HEAD f0a8374caa76ee48bf5a605ac07f723112e2f2c0 returned APPROVED with zero actionable findings. Initial product review found no blockers/actionable issues; optional polish was intentionally skipped under the minimal-diff mandate. The one-line llm-real marker refresh is validated and current cache is stable.
- Reviewer decision at product HEAD 8b6440c4b: APPROVE WITH SUGGESTIONS, with zero blockers/actionable findings and only optional NTH polish.
- Focused llm-real initially replayed a bad `crimson` response after the permanent tool catalog changed. Fork yx8vaa8ked6l bumped the unique RewindBranch marker v1→v2 at commit f0a8374caa76ee48bf5a605ac07f723112e2f2c0; full lane passed and a second warm run confirmed cache stability.
- Final re-review at f0a8374ca: APPROVED, no issues.

## Task workflow update - 2026-07-18T18:02:25.041Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (118.1s).
- Pushed task/settings-04-internal-documentation-tool to origin.
- branch 'task/settings-04-internal-documentation-tool' set up to track 'origin/task/settings-04-internal-documentation-tool'.
- Created PR: https://github.com/ineersa/agent-core/pull/300
- Validation: castor test — OK (4500 tests, 15306 assertions); castor deptrac/phpstan/cs-check — clean; castor test:llm-real — OK (10 tests, 122 assertions), second run cache stable 501→501; castor phar:build + PharSmokeTest — OK (9.2 MB; 5 tests, 57 assertions)
- Summary: SETTINGS-04 complete at f0a8374caa76ee48bf5a605ac07f723112e2f2c0: curated eight-document `internal-docs` catalog, `hatfield_docs` list/read tool, parent-only exclusion, PHAR materialization, archive deletion/reference repair, focused source/PHAR tests, and warmed live-LLM catalog cassettes. Final reviewer decision APPROVED.

## Task workflow update - 2026-07-18T18:28:11.514Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #300 has four inline owner comments, all actionable: replace the fixed ID constant with a lazily built/cached directory catalog; read full documents rather than offset/limit chunks; return Markdown document context directly; remove duplicated hand-written argument validation already represented by the tool schema/toolbox. Beginning review iteration in the existing worktree.

## Task workflow update - 2026-07-18T18:35:28.981Z
- Summary: Classified all four PR #300 owner comments as actionable. Chosen minimal revision: lazy cached directory-derived catalog (no fixed ID constant), full cached Markdown body returned directly for read, remove offset/limit and TOON read envelope, remove bespoke schema-duplicate validators while retaining catalog-membership lookup as the security boundary. Source and phar:// enumeration will use native FilesystemIterator; schema remains static and I/O-free.
- Read all four inline comments: discussion_r3608926322 (dynamic cached catalog), r3608927047 (full-file reads), r3608929009 (return Markdown directly), r3608930752 (remove duplicate validation). No external documentation research required.
- Scout confirmed FilesystemIterator works for both source symlinks and materialized phar:// entries; glob fails for PHAR, Symfony Finder works but is unnecessary/transitive. Existing PHAR staging/proof needs no change.
- Test thesis for iteration: list discovers and caches directory docs; read returns complete cached Markdown without frontmatter; non-catalog IDs never become filesystem paths. Rewrite the existing focused HatfieldDocsToolTest only; no TUI or additional PHAR test.

## Task workflow update - 2026-07-18T19:02:25.056Z
- Recorded fork run: qwsgri9r6bsi
- Validation: castor test --filter=HatfieldDocsToolTest — OK (6 tests, 48 assertions); castor deptrac — 0 violations; castor phpstan — 0 errors; castor cs-check — clean; castor test:llm-real — OK (10 tests, 122 assertions), proxy cache stable 31→31
- Summary: PR #300 feedback iteration complete at HEAD 7a07a7768350387a5d5c2c2ad8cea02cef0ac886. Commit 3944c8036 replaced the fixed catalog/chunked TOON read with a lazy cached discovered catalog and raw full-Markdown read; commit 7a07a7768 applied the user's correction to use Symfony Finder and added the AppTool→SymfonyFinder deptrac allowance. Final reviewer verdict: APPROVED, all four owner comments resolved.
- Implementation fork sobcsxib4fh4 committed 3944c803697992319121fcb5a01fcdfe084bf677: dynamic process-lifetime catalog cache, full raw Markdown reads, no offset/limit or fixed IDs, reduced schema-duplicate validation, focused test rewrite.
- User rejected native FilesystemIterator discovery and requested Symfony Finder, noting PHAR support. Correction fork qwsgri9r6bsi committed 7a07a7768350387a5d5c2c2ad8cea02cef0ac886: Finder-based depth-0 `*.md` discovery plus AppTool SymfonyFinder dependency.
- Live proxy cache was cleared/rewarmed after schema changes; no marker edits remain from this iteration. Full llm-real passed repeatedly; parent confirmation passed 10/122 with cache stable 31→31.
- Reviewer re-read testing standards, all four PR comments, task context, full feedback diff, Finder/PHAR/deptrac paths, and returned APPROVED with no actionable issues. Optional missing-directory path sanitization and redundant cached id field were explicitly accepted under the simpler/minimal-churn mandate.

## Task workflow update - 2026-07-18T19:04:46.651Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (116.1s).
- Pushed task/settings-04-internal-documentation-tool to origin.
- branch 'task/settings-04-internal-documentation-tool' set up to track 'origin/task/settings-04-internal-documentation-tool'.
- PR already exists: https://github.com/ineersa/agent-core/pull/300
- Validation: castor test --filter=HatfieldDocsToolTest — OK (6 tests, 48 assertions); castor deptrac/phpstan/cs-check — clean; castor test:llm-real — OK (10 tests, 122 assertions), cache stable 31→31
- Summary: Addressed all four PR #300 inline comments and the follow-up Symfony Finder correction at HEAD 7a07a7768350387a5d5c2c2ad8cea02cef0ac886. `hatfield_docs` now discovers and caches curated docs once, lists cached metadata, returns full raw Markdown for reads, has no fixed ID list or offset/limit machinery, and keeps user IDs out of filesystem paths. Final reviewer verdict APPROVED.

## Task workflow update - 2026-07-18T21:45:34.426Z
- Moved CODE-REVIEW → DONE.
- Merged task/settings-04-internal-documentation-tool into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/helpers.php                                |   9 +
 .hatfield/settings.yaml                            |   2 +-
 .pi/plans/extension-tool-hooks-safeguard-plan.md   |   5 -
 README.md                                          |   1 -
 box.json                                           |   1 +
 config/hatfield.defaults.yaml                      |   2 +-
 depfile.yaml                                       |   3 +
 docs/agents.md                                     |   4 +
 docs/archive/agent-core-architecture.md            |  83 ---
 docs/archive/api-and-streaming.md                  |  58 --
 docs/archive/control-plane.md                      |  45 --
 docs/archive/events-and-traces.md                  |  54 --
 docs/archive/flex-recipe.md                        |  68 ---
 docs/archive/hooks.md                              |  30 --
 docs/archive/implementation/00-bundle-setup.md     | 153 ------
 .../implementation/01-js-parity-contracts.md       | 129 -----
 .../02-runtime-domain-and-reducer.md               | 114 ----
 .../03-persistence-hot-cold-storage.md             | 179 ------
 .../implementation/04-orchestrator-and-workers.md  |  93 ----
 .../implementation/05-symfony-ai-integration.md    |  80 ---
 .../06-tool-execution-hitl-and-parallelism.md      |  84 ---
 .../07-steering-cancel-continue-resume.md          |  95 ----
 .../implementation/08-api-and-mercure-streaming.md |  69 ---
 .../09-testing-observability-and-debugging.md      |  78 ---
 .../10-rollout-operations-and-retention.md         |  72 ---
 .../11-reference-schemas-and-event-examples.md     | 166 ------
 .../12-refactor-run-orchestrator-deepening.md      | 287 ----------
 .../13-refactor-tool-execution-symfony-ai.md       | 226 --------
 .../14-refactor-message-hydration.md               |  94 ----
 .../15-pi-mono-hooks-events-report.md              | 136 -----
 docs/archive/implementation/README.md              |  41 --
 docs/archive/implementation/arrays_replacement.md  | 341 ------------
 .../implementation/events_and_hooks_report.md      | 119 ----
 docs/archive/implementation/symfony-ai.md          | 598 ---------------------
 docs/archive/local-dev-symfony-app-setup.md        | 148 -----
 .../archive/operations/agent-loop-alert-rules.yaml |  47 --
 .../agent-loop-observability-dashboard.md          |  50 --
 .../operations/agent-loop-oncall-runbook.md        |  94 ----
 docs/archive/request-flow.md                       |  56 --
 docs/archive/tool-execution.md                     |  32 --
 docs/background-processes.md                       |   4 +
 docs/compaction.md                                 |   4 +
 docs/hitl-and-approvals.md                         |   4 +
 docs/mcp.md                                        |   4 +
 docs/prompt-templates.md                           |   4 +
 docs/session-storage.md                            |   4 +
 docs/settings.md                                   |   8 +-
 internal-docs/agents.md                            |   1 +
 internal-docs/background-processes.md              |   1 +
 internal-docs/compaction.md                        |   1 +
 internal-docs/hitl-and-approvals.md                |   1 +
 internal-docs/mcp.md                               |   1 +
 internal-docs/prompt-templates.md                  |   1 +
 internal-docs/session-storage.md                   |   1 +
 internal-docs/settings.md                          |   1 +
 .../Agent/Execution/AgentToolPolicyResolver.php    |   2 +-
 src/CodingAgent/Config/AgentsConfig.php            |   4 +-
 src/CodingAgent/Config/AppResourceLocator.php      |   8 +
 src/CodingAgent/Tool/HatfieldDocsTool.php          | 188 +++++++
 .../Execution/AgentToolPolicyResolverTest.php      |   6 +-
 tests/CodingAgent/Config/AgentsConfigTest.php      |   2 +-
 tests/CodingAgent/Phar/PharSmokeTest.php           |  45 ++
 .../Controller/E2E/RewindBranchLiveE2eTest.php     |   2 +-
 tests/CodingAgent/Tool/HatfieldDocsToolTest.php    | 175 ++++++
 64 files changed, 481 insertions(+), 3937 deletions(-)
 delete mode 100644 docs/archive/agent-core-architecture.md
 delete mode 100644 docs/archive/api-and-streaming.md
 delete mode 100644 docs/archive/control-plane.md
 delete mode 100644 docs/archive/events-and-traces.md
 delete mode 100644 docs/archive/flex-recipe.md
 delete mode 100644 docs/archive/hooks.md
 delete mode 100644 docs/archive/implementation/00-bundle-setup.md
 delete mode 100644 docs/archive/implementation/01-js-parity-contracts.md
 delete mode 100644 docs/archive/implementation/02-runtime-domain-and-reducer.md
 delete mode 100644 docs/archive/implementation/03-persistence-hot-cold-storage.md
 delete mode 100644 docs/archive/implementation/04-orchestrator-and-workers.md
 delete mode 100644 docs/archive/implementation/05-symfony-ai-integration.md
 delete mode 100644 docs/archive/implementation/06-tool-execution-hitl-and-parallelism.md
 delete mode 100644 docs/archive/implementation/07-steering-cancel-continue-resume.md
 delete mode 100644 docs/archive/implementation/08-api-and-mercure-streaming.md
 delete mode 100644 docs/archive/implementation/09-testing-observability-and-debugging.md
 delete mode 100644 docs/archive/implementation/10-rollout-operations-and-retention.md
 delete mode 100644 docs/archive/implementation/11-reference-schemas-and-event-examples.md
 delete mode 100644 docs/archive/implementation/12-refactor-run-orchestrator-deepening.md
 delete mode 100644 docs/archive/implementation/13-refactor-tool-execution-symfony-ai.md
 delete mode 100644 docs/archive/implementation/14-refactor-message-hydration.md
 delete mode 100644 docs/archive/implementation/15-pi-mono-hooks-events-report.md
 delete mode 100644 docs/archive/implementation/README.md
 delete mode 100644 docs/archive/implementation/arrays_replacement.md
 delete mode 100644 docs/archive/implementation/events_and_hooks_report.md
 delete mode 100644 docs/archive/implementation/symfony-ai.md
 delete mode 100644 docs/archive/local-dev-symfony-app-setup.md
 delete mode 100644 docs/archive/operations/agent-loop-alert-rules.yaml
 delete mode 100644 docs/archive/operations/agent-loop-observability-dashboard.md
 delete mode 100644 docs/archive/operations/agent-loop-oncall-runbook.md
 delete mode 100644 docs/archive/request-flow.md
 delete mode 100644 docs/archive/tool-execution.md
 create mode 120000 internal-docs/agents.md
 create mode 120000 internal-docs/background-processes.md
 create mode 120000 internal-docs/compaction.md
 create mode 120000 internal-docs/hitl-and-approvals.md
 create mode 120000 internal-docs/mcp.md
 create mode 120000 internal-docs/prompt-templates.md
 create mode 120000 internal-docs/session-storage.md
 create mode 120000 internal-docs/settings.md
 create mode 100644 src/CodingAgent/Tool/HatfieldDocsTool.php
 create mode 100644 tests/CodingAgent/Tool/HatfieldDocsToolTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/settings-04-internal-documentation-tool.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/settings-04-internal-documentation-tool.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #300 was merged on GitHub at 2026-07-18T21:45:03Z (merge commit c3a7417ef309c5d2158c4999425004f66234d239). Completing task workflow and cleaning the worktree.

## Task workflow update - 2026-07-18T21:47:51.393Z
- Updated PR Status: merged
- Validation: Post-merge `LLM_MODE=true castor check` — PASS in 280.0s; test — OK (4491 tests, 15250 assertions); controller-replay — OK (8 tests, 112 assertions); TUI replay — OK (37 tests, 193 assertions); llm-real — OK (10 tests, 122 assertions); deptrac/phpstan/cs-check — clean; llama-proxy cache stable 54→54; artifact integrity and leak checks passed
- Summary: SETTINGS-04 completed. PR #300 merged at c3a7417ef309c5d2158c4999425004f66234d239; task branch merged into integration checkout, remote main pulled, task worktree removed, and IDEA exclusions cleaned up. Integration checkout is clean.

## Task workflow update - 2026-08-06T20:59:35.251Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
