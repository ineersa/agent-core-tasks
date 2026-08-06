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
Status: TODO
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-07-22T14:50:47.154Z

## Task workflow update - 2026-07-22T14:58:34.865Z
- Summary: User added a hard concision/size requirement: each maintained README, canonical documentation file, and built-in document should stay at or below 25,000 characters. Larger documents must be made more precise or split by coherent topic rather than retained as oversized catch-all files.
- Authoritative documentation size rule: measure file content in characters and keep each Markdown document <= 25,000 characters. Add a Castor-backed validation check so regressions fail deterministically.
- For oversized files, remove repetition/stale narrative first. If still over the limit, split along stable user-facing domains with clear names, focused frontmatter/H1 descriptions, and explicit navigation links from the original/index document.
- Splits must preserve one canonical source under `docs/`; update the `internal-docs` curated projection only for model-useful documents, and update all repository/package-safe cross-links and catalog expectations.
- Do not game the limit by minifying prose, removing useful rationale, or creating arbitrary numbered fragments. Prefer concise procedures, tables, and links to focused documents.
