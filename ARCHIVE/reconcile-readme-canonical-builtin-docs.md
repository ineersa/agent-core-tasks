# Rewrite the README and reconcile canonical and built-in Hatfield documentation

## Goal
## Goal

Perform a source-backed documentation overhaul across:

- `README.md`;
- canonical repository documentation under `docs/`;
- the curated model-visible built-in documentation exposed by `hatfield_docs` through `internal-docs/`.

The result should accurately describe the current modular-monolith application and, after the packaging task lands, its supported binary/PHAR installation and release flow.

## Dependency and scope boundary

Coordinate with `TODO/ship-hatfield-phar-static-binaries-installer.md`. That task owns implementation of binary/PHAR builds, release workflows, checksums, and the installer. This task owns the complete user/developer documentation pass and must document the behavior that actually lands. Do not duplicate or invent packaging behavior ahead of that implementation.

This is primarily a documentation task. If documentation reveals a runtime defect, record a focused follow-up instead of silently broadening this task into unrelated production refactoring. Small documentation-validation/build-freshness fixes directly required to keep built-in docs correct are in scope.

## Current source-of-truth model

- `docs/*.md` is canonical prose.
- `internal-docs/*.md` is currently a curated set of symlinks into `docs/`.
- PHAR staging dereferences those symlinks into regular bundled files.
- `HatfieldDocsTool` discovers only the bundled `internal-docs` files and requires frontmatter plus an H1.
- Repository-only `docs/` is intentionally not bundled into the PHAR.

Preserve one canonical prose source rather than copying Markdown into `internal-docs`.

## Known drift to correct

### README

The current README is almost entirely obsolete:

- it documents removed `packages/` and `apps/` workspaces instead of `src/AgentCore`, `src/CodingAgent`, and `src/Tui`;
- it references nonexistent `castor install`, `castor lib:*`, `castor tui:validate`, and `castor app:check` tasks;
- it lacks current CLI usage, configuration precedence, feature overview, architecture boundaries, documentation index, and release installation guidance.

### Canonical/built-in docs

Known stale or contradictory material includes:

- `docs/settings.md`: wrong tool-timeout default and missing `agents.retrieve.*` settings;
- `docs/session-storage.md`: obsolete process-mode TODO/API, stale `/tree` future claim, attachment claims, stale Messenger middleware gap, misleading SQLite heading, and a missing `.pi` plan link;
- `docs/agents.md`: claims no dedicated child overlay despite `/agents-live` and `/agents-main` behavior;
- `docs/mcp.md`: marks dynamic registration and shutdown behavior as future although implemented;
- `docs/phar-packaging.md`: incomplete staging/Box examples, stale test-lane claims, raw `vendor/bin/phpunit`, inconsistent extension requirements, and a static-binary non-goal that conflicts with the new distribution task;
- packaged built-in docs link to unbundled `.pi/plans/`, uncurated docs, or missing files.

Other files must still be audited against current source/config rather than assuming this list is exhaustive.

## Acceptance criteria
- Rewrite `README.md` as a user-facing Hatfield entry point with an accurate product summary, current features, supported installation paths, basic invocation examples, settings precedence, real repository layout, architecture overview, development/QA commands through Castor, and a navigable documentation index.
- After the binary distribution task lands, document latest and pinned installation, static-versus-PHAR selection, supported OS/architecture targets, PHP/extension requirements, checksums, upgrades/reinstallation, local builds, and troubleshooting without duplicating unstable implementation details.
- Audit every top-level `docs/*.md` file against current code, configuration defaults, CLI help, AGENTS.md architecture rules, and actual Castor tasks. Remove stale future/TODO statements for completed behavior and clearly label genuinely unfinished behavior.
- Correct all known settings, session-storage, agents/subagents, MCP, PHAR, command, runtime, extension, and testing discrepancies identified in the task body.
- Use Castor equivalents in all QA/testing examples; remove normal-workflow instructions that invoke raw `vendor/bin/*` tools.
- Keep canonical prose in `docs/` and `internal-docs/` as a curated projection. Do not introduce copied, independently maintained versions of the same documentation.
- Define and document the curated built-in document set, including why documents are model-visible or repository-only. Ensure additions/removals have one clear update path rather than duplicated undocumented inventories.
- Ensure every built-in document has valid catalog frontmatter, a single useful H1/title, an accurate description, and content suitable for model consumption without assuming access to the source repository.
- Remove or replace links from built-in docs to `.pi/`, missing plans, unbundled files, local absolute paths, or other repository-only resources. All links/references in the packaged projection must remain meaningful from an installed PHAR/static binary.
- Add or strengthen automated documentation validation for internal-doc source existence, frontmatter/H1 validity, curated-set consistency, relative-link validity, and package-safe references.
- Ensure PHAR freshness/build inputs include canonical content used by `internal-docs`, so changing a built-in document cannot leave `phar:ensure` serving an old packaged copy.
- Verify the packaged artifact contains exactly the intended materialized built-in docs as regular files and does not accidentally bundle the full repository `docs/` tree.
- Reconcile `docs/phar-packaging.md`, `src/CodingAgent/Runtime/Process/AGENTS.md`, and packaging-related tests/checklists with the actual post-binary pipeline.
- Do not change production behavior merely to make stale documentation true. Record separately scoped implementation defects discovered during the audit.
- Run focused documentation/catalog/PHAR validation through Castor and mandatory applicable project checks before CODE-REVIEW.

## Workflow metadata
Status: ARCHIVE
Branch: task/reconcile-readme-canonical-builtin-docs
Worktree: /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs
Fork run: iuu5j1lvnj81
PR URL: https://github.com/ineersa/agent-core/pull/382
PR Status: merged
Started: 2026-08-13T17:04:42.946Z
Completed: 2026-08-14T02:14:38.999Z

## Work log
- Created: 2026-07-22T14:50:47.154Z

## Task workflow update - 2026-07-22T14:58:34.865Z
- Summary: User added a hard concision/size requirement: each maintained README, canonical documentation file, and built-in document should stay at or below 25,000 characters. Larger documents must be made more precise or split by coherent topic rather than retained as oversized catch-all files.
- Authoritative documentation size rule: measure file content in characters and keep each Markdown document <= 25,000 characters. Add a Castor-backed validation check so regressions fail deterministically.
- For oversized files, remove repetition/stale narrative first. If still over the limit, split along stable user-facing domains with clear names, focused frontmatter/H1 descriptions, and explicit navigation links from the original/index document.
- Splits must preserve one canonical source under `docs/`; update the `internal-docs` curated projection only for model-useful documents, and update all repository/package-safe cross-links and catalog expectations.
- Do not game the limit by minifying prose, removing useful rationale, or creating arbitrary numbered fragments. Prefer concise procedures, tables, and links to focused documents.

## Task workflow update - 2026-08-13T17:01:58.206Z
- Moved TODO → IN-PROGRESS.
- Created branch task/reconcile-readme-canonical-builtin-docs.
- Created worktree /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Summary: Claimed for documentation reconciliation implementation. Packaging dependency ship-hatfield-phar-static-binaries-installer is merged and archived; implementation will document actual landed behavior.

## Task workflow update - 2026-08-13T17:02:30.834Z
- Moved IN-PROGRESS → TODO.
- Summary: Returned to TODO at user request before implementation began.

## Task workflow update - 2026-08-13T17:04:32.715Z
- Moved TODO → CANCELLED.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs: ide_close_project returned isError.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Summary: Cleaning the stale clean worktree left by the earlier TODO rollback so the task can be reclaimed safely.

## Task workflow update - 2026-08-13T17:04:36.225Z
- Moved CANCELLED → TODO.
- Summary: Restored after stale worktree cleanup; task remains approved for immediate reclaim.

## Task workflow update - 2026-08-13T17:04:42.946Z
- Moved TODO → IN-PROGRESS.
- Created branch task/reconcile-readme-canonical-builtin-docs.
- Created worktree /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Summary: Reclaimed for implementation at user request after updating integration main and cleaning the stale prior worktree.

## Task workflow update - 2026-08-13T17:43:48.678Z
- Summary: Planning decision: remove internal-docs; canonical core docs stay under docs/ and use frontmatter builtin: true for hatfield_docs discovery and PHAR selection. Initial scope is minimal: hatfield_docs exposes curated core docs plus curated public Extension API docs; per-extension docs remain package-local. A future Extension API mechanism may let installed extensions expose docs, but that runtime surface is explicitly deferred. docs/file-rewind.md belongs to the file-rewind extension rather than core Hatfield documentation.

## Task workflow update - 2026-08-13T17:51:14.375Z
- Summary: User approved the initial model-visible catalog: core docs for settings/settings-models/settings-agents, agents, background-processes, compaction, human-input, approvals, MCP, prompt templates, session storage, and session history only if needed after concise cleanup; Extension API docs extension-api, extension-api-tools, extension-api-runtime, and extension-api-tui under .hatfield/extensions/extension-api/docs/. Both approved roots use builtin: true, extension-api-* IDs are reserved, duplicate IDs fail validation, installed extension docs remain package-local and undiscovered, and every maintained Markdown file must remain <=25,000 characters. docs/file-rewind.md moves into the file-rewind extension source of truth. User also requested a draft PR after implementation so progress can be tracked.

## Task workflow update - 2026-08-13T17:51:57.842Z
- Recorded fork run: u65h7s1x2cdt
- Summary: Implementation delegated to fork u65h7s1x2cdt in the exact task worktree. Brief covers removing internal-docs, strict builtin:true discovery across core and Extension API roots, exact PHAR packaging/freshness, the approved 12-or-fewer core and four API catalog, file-rewind ownership transfer, complete README/docs audit, <=25k validation, package-safe links, focused Castor tests, and a clean committed handoff. Draft PR will be created by the orchestrator after implementation verification, per user request.

## Task workflow update - 2026-08-13T18:06:38.932Z
- Recorded fork run: u65h7s1x2cdt
- Updated PR URL: https://github.com/ineersa/agent-core/pull/382
- Updated PR Status: draft
- Validation: Fork read and followed .agents/skills/testing/SKILL.md and tests/AGENTS.md.; castor docs:validate: PASS (15 built-ins).; castor test --filter=HatfieldDocsToolTest: PASS (8 tests).; castor phar:build: PASS.; castor test --filter=PharSmokeTest: PASS (7 tests).; Combined focused catalog/PHAR filter: PASS (15 tests, 219 assertions).; castor deptrac: PASS (0 violations).; Focused castor phpstan: PASS (0 errors).; castor cs-check: PASS.; Full castor check not run; reserved for task-to-pr.
- Summary: Implementation verified at f0164c454: worktree clean; one task commit against origin/main; 51 files changed (+1727/-8328); internal-docs deleted; exactly 15 builtin:true docs across the two approved roots; no maintained Markdown file exceeds 25,000 characters; diff --check clean. Pushed the task branch and opened draft PR #382 for progress tracking at user request. Task intentionally remains IN-PROGRESS; full reviewer workflow and castor check are deferred to task-to-pr.

## Task workflow update - 2026-08-13T18:12:24.405Z
- Summary: User requested that `castor docs:validate` run as part of the standard `castor check` gate. Preparing a focused follow-up implementation on the existing draft PR branch.

## Task workflow update - 2026-08-13T18:15:42.329Z
- Recorded fork run: r7npnb28t3dt
- Validation: Fork read and followed .agents/skills/testing/SKILL.md and tests/AGENTS.md.; php -l .castor/tasks.php: PASS.; castor docs:validate: PASS (15 built-ins).; Simulated check lane invocation with QA environment and timeout: PASS.; castor cs-check: PASS.; git diff --check origin/main...HEAD: PASS.; Full castor check remains deferred to task-to-pr.
- Summary: Added `docs:validate` as a first-class 30-second parallel lane in standard `castor check`, reusing the existing Castor task and normal lane failure/report aggregation. Updated README, AGENTS.md, and testing skill command inventories. Verified commit f3415e08e, pushed it to draft PR #382, and left the task worktree clean.

## Task workflow update - 2026-08-13T18:37:04.241Z
- Summary: Starting a source-backed documentation quality/correctness review of draft PR #382. Review will separately cover user-facing core docs, developer/architecture/QA docs, and Extension API/catalog/packaging docs, then actionable findings will be fixed before formal task-to-pr.

## Task workflow update - 2026-08-13T19:30:02.465Z
- Summary: User approved iterative source-grounding with 2–4 audit/fix passes and requested it be completed in this task rather than deferred to follow-ups. First remediation pass will address all verified correctness, packaging, validation, command, and API-doc findings; runtime defects discovered will be documented accurately rather than silently changed unless directly required by the docs/catalog contract.

## Task workflow update - 2026-08-13T20:01:22.409Z
- Recorded fork run: u6frhvplb6k2
- Validation: castor docs:validate: PASS (15 built-ins).; Focused catalog/tool/PHAR suite: PASS (25 tests, 300 assertions).; castor phar:build: PASS; unmarked Extension API doc exclusion manually proved.; castor deptrac: PASS (0 violations).; Focused castor phpstan: PASS (0 errors).; castor cs-check: PASS.; No maintained scoped Markdown exceeds 25,000 characters.; git diff --check origin/main...HEAD: PASS.
- Summary: Grounding pass 1 complete and pushed to draft PR #382 at 3a3715bfd. Fixed verified core/API documentation errors, strengthened exact catalog packaging, link/anchor validation, intended-marker failure, README size coverage, PHAR freshness/exclusion proof, and Castor-only command guidance. Worktree clean; beginning pass 2 audit for residual correctness, completeness, and prose quality.

## Task workflow update - 2026-08-13T20:11:56.013Z
- Summary: Grounding pass 2 found additional source-backed gaps in operational docs, testing commands, Extension API/catalog link contracts, PHAR build identity freshness, and test isolation. Launching an in-task remediation pass; no follow-up tasks will be created. Optional prose preferences will be skipped unless they restore a concrete invariant or correct a verifiable claim.

## Task workflow update - 2026-08-13T20:21:43.008Z
- Recorded fork run: yzj7qa8q86sx
- Validation: castor docs:validate: PASS (15 built-ins).; Focused docs/catalog/tool/staging/PHAR suite: PASS (28 tests, 353 assertions).; castor deptrac: PASS (0 violations).; Focused castor phpstan: PASS (0 errors).; castor cs-check: PASS.; git diff --check origin/main...HEAD: PASS.
- Summary: Grounding pass 2 complete and pushed to draft PR #382 at 1a6362b70. Corrected remaining model-visible operational semantics and runnable commands, centralized docs validation on CommonMark-backed production QA, fixed resolved-build-identity freshness, removed unselected/vendor API docs from PHAR, and made packaging tests isolated. Worktree clean; starting pass 3 convergence audit.

## Task workflow update - 2026-08-13T20:31:48.784Z
- Summary: Pass 3 requested changes. Final remediation will fix the remaining two model-facing human-input claims, stale/wrong tracked Markdown commands and storage claims, catalog root-containment/frontmatter fail-closed gaps, complete vendor-duplicate verification, production-helper staging proof, tracked README scope, and CommonMark-consistent H1 parsing. No new runtime/public product surface.

## Task workflow update - 2026-08-13T20:39:56.004Z
- Recorded fork run: a3wvok6jr679
- Validation: castor docs:validate: PASS (15 built-ins).; Focused docs/catalog/tool/staging suite: PASS (28 tests, 183 assertions).; PharSmokeTest: PASS (7 tests, 8206 assertions).; castor deptrac: PASS (0 violations).; Focused castor phpstan: PASS (0 errors).; castor cs-check: PASS.; git diff --check origin/main...HEAD: PASS.; No scoped maintained Markdown exceeds 25,000 characters.
- Summary: Grounding pass 3 complete and pushed to draft PR #382 at 7aa2e2368. Fixed final known model-facing and maintainer-doc inaccuracies, catalog symlink/root-containment and malformed-frontmatter gaps, tracked README scope, CommonMark-consistent H1/link handling, complete vendor-doc exclusion verification, and production-helper packaging proof. Beginning final convergence audit; no broad scope expansion.

## Task workflow update - 2026-08-13T20:47:03.163Z
- Summary: Final convergence found five narrow blockers: OpenAI base URL example duplication, one stale plan prerequisite, approved-root symlink escape, malformed trailing marker omission, and three unused AppResourceLocator docs methods. Running one final targeted remediation; no further broad audit pass afterward.

## Task workflow update - 2026-08-13T20:54:08.196Z
- Recorded fork run: is2u3n5112yz
- Validation: castor docs:validate: PASS (15 built-ins).; Focused catalog/validator/tool/staging/PHAR suite: PASS (39 tests, 8395 assertions).; castor deptrac: PASS (0 violations).; Focused castor phpstan for Docs and AppResourceLocator: PASS (0 errors).; castor cs-check: PASS.; Manual filesystem and PHAR catalog discovery: 15 entries each.; git diff --check origin/main...HEAD: PASS.; Final grep confirms removed locator methods and broken OpenAI base URL are absent.; No scoped maintained Markdown exceeds 25,000 characters.
- Summary: Four iterative source-grounding cycles are complete. Final targeted commit f12e8cd88 is pushed to draft PR #382. It fixes the last OpenAI example, historical plan reference, approved-root containment, malformed marker omission, and removes test-only AppResourceLocator docs methods. Final branch diff is clean: 63 files, +3285/-8347; no scoped maintained Markdown exceeds 25,000 characters. Implementation remains IN-PROGRESS pending explicit task-to-pr/full gate.

## Task workflow update - 2026-08-14T00:33:53.454Z
- Recorded fork run: iuu5j1lvnj81
- Validation: castor test: PASS (4441 tests, 16827 assertions).; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS.; castor docs:validate: PASS (15 built-ins).; git diff --check origin/main...HEAD: PASS.; Worktree clean at 5f213e3e8.
- Summary: Task-to-PR focused QA found and fixed three PHPStan issues in changed Castor files at 5f213e3e8. Full focused gate is now green. Formal reviewer subagent was attempted three times (plus one read-only review fork) but returned no usable report due agent/tool degradation. Review evidence therefore uses the completed multi-pass source-grounding audits: final user/model docs audit approved; all concrete repository and adversarial catalog findings were fixed and revalidated. No unresolved review blockers remain.

## Task workflow update - 2026-08-14T00:36:21.552Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (133.2s).
- Pushed task/reconcile-readme-canonical-builtin-docs to origin.
- branch 'task/reconcile-readme-canonical-builtin-docs' set up to track 'origin/task/reconcile-readme-canonical-builtin-docs'.
- PR already exists: https://github.com/ineersa/agent-core/pull/382
- Validation: castor test: PASS (4441 tests, 16827 assertions).; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS.; castor docs:validate: PASS (15 built-ins).; Focused catalog/validator/tool/staging/PHAR suite: PASS (39 tests, 8395 assertions).; git diff --check origin/main...HEAD: PASS.
- Summary: Documentation reconciliation is complete after four iterative source-grounding passes plus final task-to-PR QA remediation. Canonical docs now drive a strict 15-document built-in catalog, internal-docs is removed, Extension API docs and file-rewind ownership are reconciled, PHAR packaging/freshness is exact and validated, README/docs are source-grounded and <=25k, and docs:validate is part of castor check.

## Task workflow update - 2026-08-14T00:36:40.873Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/382
- Updated PR Status: open
- Validation: Deterministic castor check: PASS (133.2s).; GitHub PR #382: open, ready for review, mergeable.; Remote head: 5f213e3e826da2b1af23db62129cca1faad0e395.
- Summary: PR #382 is now ready for review at commit 5f213e3e8. Deterministic castor check passed in 133.2 seconds during CODE-REVIEW transition; branch pushed and PR is mergeable.

## Task workflow update - 2026-08-14T02:14:38.999Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Merged task/reconcile-readme-canonical-builtin-docs into integration checkout.
- Auto-merging AGENTS.md
Auto-merging src/CodingAgent/Config/AppResourceLocator.php
Auto-merging tests/CodingAgent/Phar/PharSmokeTest.php
Merge made by the 'ort' strategy.
 .../going-further/extending-castor/repack.md       |    2 -
 .agents/skills/testing/SKILL.md                    |   20 +-
 .castor/distribution.php                           |   67 +-
 .castor/docs.php                                   |   38 +
 .castor/helpers.php                                |  115 +-
 .castor/tasks.php                                  |   15 +-
 .../extension-api/docs/extension-api-runtime.md    |   64 +
 .../extension-api/docs/extension-api-tools.md      |   68 +
 .../extension-api/docs/extension-api-tui.md        |   44 +
 .../extensions/extension-api/docs/extension-api.md |   72 +
 .hatfield/extensions/file-rewind/README.md         |   47 +-
 .../extensions/observational-memory/README.md      |    8 +-
 .pi/plans/agents-subagents-implementation-plan.md  |   15 +-
 .pi/plans/async-headless-messenger-plan.md         |    9 +
 AGENTS.md                                          |    6 +-
 README.md                                          |  226 ++-
 box.json                                           |    3 +-
 castor.php                                         |    1 +
 docs/agents-hidden-run-control-poc.md              |  526 ------
 docs/agents.md                                     |  445 +----
 docs/approvals.md                                  |   84 +
 docs/async-runtime-architecture.md                 |  806 +--------
 docs/background-processes.md                       |  137 +-
 docs/compaction.md                                 |  308 +---
 docs/datadog.md                                    |  214 +--
 docs/distribution.md                               |  200 +--
 docs/file-rewind.md                                |   74 -
 docs/hitl-and-approvals.md                         |  799 ---------
 docs/human-input.md                                |   73 +
 docs/llm-replay.md                                 |  213 +--
 docs/mcp.md                                        |  349 +---
 docs/phar-packaging.md                             |  144 +-
 docs/prompt-templates.md                           |    1 +
 docs/qa-metrics.md                                 |   81 -
 docs/session-storage.md                            |  680 +-------
 docs/settings-agents.md                            |   97 ++
 docs/settings-models.md                            |   94 ++
 docs/settings.md                                   | 1780 ++------------------
 docs/static-packaging.md                           |  119 +-
 docs/tool-execution.md                             |  463 +----
 docs/tui-architecture.md                           | 1011 +----------
 docs/tui-testing.md                                |  330 +---
 internal-docs/agents.md                            |    1 -
 internal-docs/background-processes.md              |    1 -
 internal-docs/compaction.md                        |    1 -
 internal-docs/hitl-and-approvals.md                |    1 -
 internal-docs/mcp.md                               |    1 -
 internal-docs/prompt-templates.md                  |    1 -
 internal-docs/session-storage.md                   |    1 -
 internal-docs/settings.md                          |    1 -
 src/CodingAgent/Config/AppResourceLocator.php      |   11 +-
 src/CodingAgent/Docs/BuiltinDocsCatalog.php        |  388 +++++
 .../Docs/BuiltinDocsCatalogException.php           |   12 +
 .../Docs/BuiltinDocsMarkdownScanner.php            |  154 ++
 src/CodingAgent/Docs/BuiltinDocsValidator.php      |  254 +++
 src/CodingAgent/Runtime/Protocol/AGENTS.md         |    4 +-
 src/CodingAgent/Tool/HatfieldDocsTool.php          |   78 +-
 tests/AGENTS.md                                    |    4 +-
 tests/CodingAgent/Docs/BuiltinDocsCatalogTest.php  |  224 +++
 .../CodingAgent/Docs/DocsValidateContractTest.php  |  209 +++
 .../Docs/PharDocsStagingContractTest.php           |  213 +++
 tests/CodingAgent/Phar/PharSmokeTest.php           |  120 +-
 tests/CodingAgent/Tool/HatfieldDocsToolTest.php    |  110 +-
 63 files changed, 3290 insertions(+), 8347 deletions(-)
 create mode 100644 .castor/docs.php
 create mode 100644 .hatfield/extensions/extension-api/docs/extension-api-runtime.md
 create mode 100644 .hatfield/extensions/extension-api/docs/extension-api-tools.md
 create mode 100644 .hatfield/extensions/extension-api/docs/extension-api-tui.md
 create mode 100644 .hatfield/extensions/extension-api/docs/extension-api.md
 delete mode 100644 docs/agents-hidden-run-control-poc.md
 create mode 100644 docs/approvals.md
 delete mode 100644 docs/file-rewind.md
 delete mode 100644 docs/hitl-and-approvals.md
 create mode 100644 docs/human-input.md
 delete mode 100644 docs/qa-metrics.md
 create mode 100644 docs/settings-agents.md
 create mode 100644 docs/settings-models.md
 delete mode 120000 internal-docs/agents.md
 delete mode 120000 internal-docs/background-processes.md
 delete mode 120000 internal-docs/compaction.md
 delete mode 120000 internal-docs/hitl-and-approvals.md
 delete mode 120000 internal-docs/mcp.md
 delete mode 120000 internal-docs/prompt-templates.md
 delete mode 120000 internal-docs/session-storage.md
 delete mode 120000 internal-docs/settings.md
 create mode 100644 src/CodingAgent/Docs/BuiltinDocsCatalog.php
 create mode 100644 src/CodingAgent/Docs/BuiltinDocsCatalogException.php
 create mode 100644 src/CodingAgent/Docs/BuiltinDocsMarkdownScanner.php
 create mode 100644 src/CodingAgent/Docs/BuiltinDocsValidator.php
 create mode 100644 tests/CodingAgent/Docs/BuiltinDocsCatalogTest.php
 create mode 100644 tests/CodingAgent/Docs/DocsValidateContractTest.php
 create mode 100644 tests/CodingAgent/Docs/PharDocsStagingContractTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/reconcile-readme-canonical-builtin-docs.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #382 state: MERGED.; Merged at: 2026-08-14T02:14:12Z.; Merge commit: e1dd342d09d49634f6e76281ec4e5663dda8c7e8.
- Summary: PR #382 was merged on GitHub at e1dd342d09d49634f6e76281ec4e5663dda8c7e8. Moving task to DONE and synchronizing/cleaning the task worktree.

## Task workflow update - 2026-08-14T02:18:34.047Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/382
- Updated PR Status: merged
- Validation: PR #382 merged at e1dd342d09d49634f6e76281ec4e5663dda8c7e8.; DONE transition merged/synced integration checkout and removed worktree.; Post-merge LLM_MODE=true castor check: FAILED only test + phpstan.; Test failure: HatfieldDocsToolTest calls runtime path where HatfieldDocsTool.php:129 invokes undefined AppResourceLocator::getAppRoot().; PHPStan failure: undefined AppResourceLocator::getAppRoot() at HatfieldDocsTool.php:129.; deptrac, controller replay, TUI, llm-real, cs-check, docs:validate, artifact integrity, leak check, and llama-proxy cache guard: PASS.; QA report: var/reports/qa-20260814-021443-935398-950036b4.
- Summary: Task moved to DONE and worktree cleaned after PR #382 merged. Post-merge integration validation exposed a semantic conflict with concurrently merged PR #380: AppResourceLocator::getAppRoot() was removed there while HatfieldDocsTool from #382 calls it. The merged origin/main currently fails one HatfieldDocsTool test and PHPStan for that undefined method. This was not present in the task worktree gate and requires a small mainline hotfix; no workers leaked and all other check lanes passed.

## Task workflow update - 2026-08-14T19:53:40+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
