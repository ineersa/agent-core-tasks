# Use global artifact-scoped cache and disposable installer smokes

## Goal
Fix the v0.0.4 installed-artifact regression where `hatfield` fails with `Theme "cyberpunk" is not registered. Available themes: (none)` after installation.

Root cause: the v0.0.4 console visibility loop calls `Application::all()` before `run()`, so the installer's pre-move `--version` smokes now boot FrameworkBundle and compile Symfony's DI container. The container embeds `%kernel.project_dir%` as a literal `phar://<physical executable>/...` resource root. Candidate and install-dir smokes run under temporary filenames, then the installer atomically renames the artifact. The existing PHAR/native cache key includes artifact content only, so the final executable reuses a container whose `AppResourceLocator` points into the deleted temporary filename. Themes are packaged correctly; cached service wiring points at the wrong outer archive path.

Implement the complete cache model approved by the user:

```text
${XDG_CACHE_HOME:-$HOME/.cache}/hatfield/
  <environment>/
    <artifact-content-hash>-<canonical-installation-path-hash>/
```

Installed PHAR and fused native artifacts use this global, namespaced Symfony container cache by default. Project settings, sessions, extension data and other project runtime state remain under project `.hatfield/`. Source-checkout cache behavior remains project-local. `HATFIELD_CACHE_DIR` remains an authoritative explicit override for tests/operations and is still namespaced safely for environment/artifact identity.

Installer candidate and install-directory temporary smokes must use disposable cache and log roots under the installer's existing temporary tree (or equivalent `mktemp -d`) and remove them through the existing cleanup trap. Temporary artifact execution must never write generated Symfony state into either the caller project's `.hatfield/` or the user's persistent global Hatfield cache.

Reproduction evidence: actual v0.0.3/v0.0.4 PHAR and linux-amd64 assets contain identical themes. Running identical artifact bytes at path A with a shared cache, removing A, then running at path B reproduces the empty registry. Generated `getThemeRegistryService.php` constructs `AppResourceLocator('phar://<deleted-path-A>')`. A fresh `HATFIELD_CACHE_DIR` fixes it. v0.0.3 had the latent cache-identity defect, but its installer `--version` did not boot the kernel; v0.0.4's unconditional `Application::all()` makes the standard install flow trigger it.

## Acceptance criteria
- Installed PHAR and fused native executions default Symfony's compiled container cache to `${XDG_CACHE_HOME}/hatfield` when `XDG_CACHE_HOME` is set, otherwise `$HOME/.cache/hatfield`; they no longer create/use project `.hatfield/cache` by default.
- Global installed-artifact cache entries are isolated by Symfony environment plus stable SHA-256-derived identity for both artifact contents and the canonical physical executable/archive path. Same bytes at different paths do not reuse a compiled container; different versions at the same path do not reuse one either.
- Canonical artifact path handling works for PHAR, fused PHP-micro native binaries, symlinked invocations, paths containing spaces, and macOS/Linux. Reuse existing executable/PHAR resolution code rather than adding a second locator algorithm where practical.
- Source-checkout execution keeps its current project-local cache behavior; project settings, sessions, extension data and logs are not moved by this task. `HATFIELD_CACHE_DIR` remains authoritative and receives the same environment/artifact isolation needed to prevent cross-path reuse.
- If an installed artifact cannot determine a valid XDG/HOME cache root or create its cache directory, fail with a clear diagnostic rather than silently writing to an unexpected working directory.
- Both installer `--version` smokes (download candidate path and install-directory temporary path) run with disposable `HATFIELD_CACHE_DIR` and `HATFIELD_LOG_DIR` values under a trap-cleaned temporary root. They leave no cache/log files in the caller project or persistent global Hatfield cache, while preserving checksum verification, pre-replacement smoke validation and atomic final `mv`.
- Add a regression proof for the original defect: run identical PHAR bytes from path A using a shared cache, make path A unavailable, then run from path B and prove agent construction/resource loading still finds the bundled `cyberpunk` theme. Prefer a bounded non-interactive `agent --headless` EOF path over a new Tmux journey if it constructs the real ThemeRegistry and exits cleanly.
- Add focused cache-contract tests proving: same artifact+same canonical path is stable; same bytes+different canonical paths differ; different bytes+same path differ; source and installed artifact caches do not collide; explicit `HATFIELD_CACHE_DIR` remains authoritative.
- Extend the existing bash installer behavioral test to prove smoke cache/log isolation and cleanup without inspecting implementation text or mutating real user cache directories.
- Document the installed cache location, XDG fallback, identity/isolation behavior, `HATFIELD_CACHE_DIR` override, multi-version coexistence and safe cache-clearing procedure in the appropriate distribution/static/PHAR documentation without moving project-owned runtime data.
- Run focused Castor tests, `castor test`, `castor deptrac`, `castor phpstan`, `castor cs-check`, artifact PHAR regression proof, and full deterministic `castor check` before PR. A release artifact is still required after merge to prove fused native behavior on the four-platform matrix.

## Workflow metadata
Status: ARCHIVE
Branch: task/fix-global-artifact-cache-and-isolate-installer-smokes
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-global-artifact-cache-and-isolate-installer-smokes
Fork run: b6p99l4soumt
PR URL: https://github.com/ineersa/agent-core/pull/336
PR Status: merged
Started: 2026-07-28T23:50:18.881Z
Completed: 2026-07-29T17:36:31.053Z

## Work log
- Created: 2026-07-28T23:49:50.601Z

## Task workflow update - 2026-07-28T23:50:18.881Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-global-artifact-cache-and-isolate-installer-smokes.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-global-artifact-cache-and-isolate-installer-smokes.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-global-artifact-cache-and-isolate-installer-smokes.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-global-artifact-cache-and-isolate-installer-smokes.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-global-artifact-cache-and-isolate-installer-smokes.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-global-artifact-cache-and-isolate-installer-smokes.
- Summary: Starting implementation of the user-approved complete fix: XDG global installed-artifact cache keyed by content+canonical path, source cache unchanged, and disposable cache/log roots for both installer temporary smokes.

## Task workflow update - 2026-07-29T00:00:29.872Z
- Recorded fork run: 0wnhdedo54zi
- Summary: Implementation fork launched. It will first merge origin/main normally because the integration checkout had diverged before v0.0.4, then implement the global XDG content+canonical-path cache identity, disposable installer smoke environment, real PHAR relocation regression, installer behavioral proof and scoped docs.

## Task workflow update - 2026-07-29T00:12:45.752Z
- Recorded fork run: 0wnhdedo54zi
- Validation: castor phar:build PASS; Focused affected tests PASS (16 tests, 191 assertions); castor test PASS (4459 tests, 16226 assertions); castor deptrac PASS (0 violations); castor phpstan PASS (0 errors); castor cs-check PASS; castor clean:cleanup:workers:list PASS
- Summary: Primary implementation completed at 28bf7f5ef after normal origin/main merge 29fa41e6f: 11 files, global installed XDG content+canonical-path cache, source cache unchanged, disposable installer smokes, scoped docs and tests. A narrow correction fork p75b0wcswyth is replacing the relocation test's lazy `agent --help`/generated-container inspection with the real bounded `agent --headless` EOF path that constructs ThemeRegistry and catches the original failure.

## Task workflow update - 2026-07-29T00:33:39.020Z
- Recorded fork run: p75b0wcswyth
- Validation: Diagnostic: PHAR agent --headless with invalid theme exits 0, proving headless is insufficient; Diagnostic: PHAR agent --transport=in-process with invalid theme exits 1 at ThemeRegistry; cyberpunk paints successfully; castor phar:build PASS; Focused relocation test PASS (1 test, 27 assertions); Focused affected suite PASS (16 tests, 191 assertions before proof correction); castor test PASS (4459 tests, 16226 assertions before proof correction); castor deptrac PASS (0 violations); castor phpstan PASS (0 errors); castor cs-check PASS; castor clean:cleanup:workers:list PASS (no stale workers)
- Summary: Task-start implementation complete and clean at 6cd3178bd (on implementation 28bf7f5ef and origin/main merge 29fa41e6f). Diagnostic disproved the proposed `agent --headless` proof: headless constructs ThemeRegistry but never calls getOrThrow, so an invalid theme exits 0. Relocation regression now uses bounded `agent --transport=in-process`, which reaches InteractiveMode→ThemeRegistry::getOrThrow without controller/Messenger workers, proves cyberpunk paint after path A is removed and identical PHAR moves to path B, and retains content/canonical-path/symlink cache identity assertions. No production API was added for tests.

## Task workflow update - 2026-07-29T00:44:46.022Z
- Validation: Reviewer: APPROVE WITH SUGGESTIONS at 6cd3178bd; Specification-fidelity inventory: all changed external behavior mapped; no unmapped surface; Proof assessment: relocation, cache identity, source behavior, and disposable installer smoke tests are valid; Security/process review: clear fail-closed roots, race-safe directory creation, shell quoting/traps accepted
- Summary: Reviewer verdict at clean HEAD 6cd3178bd: APPROVE WITH SUGGESTIONS. No critical, correctness, security, process, spec-fidelity, documentation, or proof blockers. Reviewer confirmed the in-process TUI relocation regression reaches ThemeRegistry::getOrThrow and would fail the old content-only cache. Non-blocking cleanup: replace fragile source-cache assertion containing `/hatfield/`; inline Kernel::isInstalledArtifact() and resolveCanonicalArtifactPath() one-call wrappers (~18 lines).

## Task workflow update - 2026-07-29T00:50:10.416Z
- Recorded fork run: cpy0sh5zd40z
- Validation: castor test --filter=KernelCacheIsolationTest PASS (2 tests, 4 assertions); castor test PASS (4459 tests, 16225 assertions; ParaTest 16); castor deptrac PASS (0 violations); castor phpstan PASS (0 errors); castor cs-check PASS (files_fixed=0); castor clean:cleanup:workers:list PASS (no stale workers); No additional reviewer per explicit user instruction
- Summary: User-approved reviewer cleanups completed without re-review. HEAD e2c573b8c is clean: removed two one-call Kernel wrappers, replaced fragile source-cache assertion with exact override-aware contract, then fixed its ParaTest-only relative HATFIELD_CACHE_DIR handling discovered by full castor test. Ready for deterministic CODE-REVIEW gate.

## Task workflow update - 2026-07-29T00:52:35.029Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (131.0s).
- Pushed task/fix-global-artifact-cache-and-isolate-installer-smokes to origin.
- branch 'task/fix-global-artifact-cache-and-isolate-installer-smokes' set up to track 'origin/task/fix-global-artifact-cache-and-isolate-installer-smokes'.
- Created PR: https://github.com/ineersa/agent-core/pull/336
- Validation: castor phar:build PASS; Focused PHAR relocation and installer tests PASS; castor test PASS (4459 tests, 16225 assertions); castor deptrac PASS (0 violations); castor phpstan PASS (0 errors); castor cs-check PASS; No stale QA workers
- Summary: Approved implementation at e2c573b8c: globally artifact/path-scoped installed cache, source cache unchanged, disposable installer smokes, real relocated-PHAR active-theme regression proof, scoped docs, and reviewer cleanups. Existing reviewer verdict APPROVE WITH SUGGESTIONS; suggestions applied; no re-review per user instruction.

## Task workflow update - 2026-07-29T16:57:35.771Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User reproduced `castor run:agent` failure after global cache change: bubblewrap sandbox does not allow writing `${XDG_CACHE_HOME:-$HOME/.cache}/hatfield`. Reopening task to inspect sandbox mounts and minimally permit the Hatfield cache root.

## Task workflow update - 2026-07-29T17:03:47.462Z
- Recorded fork run: b6p99l4soumt
- Summary: Root cause confirmed outside repository: `/home/ineersa/bin/pi-bwrap` read-only binds HOME and overlays projects/claw/.pi/.hatfield, but not the new exact Hatfield XDG cache root. Applying a local secure wrapper fix to create/bind only the effective Hatfield cache directory writable (not all ~/.cache), honoring HATFIELD_CACHE_DIR/XDG precedence and spaces.

## Task workflow update - 2026-07-29T17:06:48.346Z
- Validation: bash -n /home/ineersa/bin/pi-bwrap PASS; pi-bwrap write/delete probe under $HOME/.cache PASS; pi-bwrap built PHAR --version PASS (Hatfield dev, prod); Expected ~/.cache/hatfield/prod/<content>-<path> directory present
- Summary: User-local Bubblewrap wrapper fixed directly at `/home/ineersa/bin/pi-bwrap`: `$HOME/.cache` added as a writable overlay after read-only HOME bind, per explicit user approval to share the whole cache. Repository branch unchanged. Verified bash syntax, writable file creation inside sandbox, and the built PHAR `--version` boot inside pi-bwrap creates/reuses the expected global Hatfield cache identity successfully.

## Task workflow update - 2026-07-29T17:08:54.288Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (118.5s).
- Pushed task/fix-global-artifact-cache-and-isolate-installer-smokes to origin.
- branch 'task/fix-global-artifact-cache-and-isolate-installer-smokes' set up to track 'origin/task/fix-global-artifact-cache-and-isolate-installer-smokes'.
- PR already exists: https://github.com/ineersa/agent-core/pull/336
- Validation: Local pi-bwrap syntax PASS; Sandbox ~/.cache write probe PASS; Sandboxed built PHAR --version PASS with global artifact cache; Prior focused repository validation PASS
- Summary: Returned to CODE-REVIEW after fixing the user-local pi-bwrap writable-cache mount. Repository branch remains clean/unchanged at e2c573b8c; PR #336 implementation is intact.

## Task workflow update - 2026-07-29T17:36:31.053Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-global-artifact-cache-and-isolate-installer-smokes into integration checkout.
- Auto-merging docs/settings.md
Merge made by the 'ort' strategy.
 .castor/helpers.php                                |  45 +++--
 docs/distribution.md                               |  14 ++
 docs/phar-packaging.md                             |  28 ++-
 docs/settings.md                                   |  13 +-
 docs/static-packaging.md                           |  13 +-
 installer/bash-installer                           |  30 ++-
 src/CodingAgent/Kernel.php                         | 157 ++++++++++------
 .../CodingAgent/Distribution/BashInstallerTest.php | 126 +++++++++++++
 .../CodingAgent/Phar/KernelCacheIsolationTest.php  |  70 +++----
 .../Phar/PharArtifactCacheRelocationTest.php       | 209 +++++++++++++++++++++
 tests/CodingAgent/Phar/PharSmokeTest.php           | 122 ++++++------
 11 files changed, 633 insertions(+), 194 deletions(-)
 create mode 100644 tests/CodingAgent/Phar/PharArtifactCacheRelocationTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-global-artifact-cache-and-isolate-installer-smokes.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-global-artifact-cache-and-isolate-installer-smokes.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #336 state MERGED at 2026-07-29T17:14:34Z; CODE-REVIEW deterministic castor check passed before merge; Sandboxed PHAR global-cache boot passed
- Summary: PR #336 merged on GitHub as 7347304a4e2064434ff0c11f6a72b1f0bb5b6d2a. Completing tracked task and cleaning its worktree.

## Task workflow update - 2026-07-29T17:39:25.015Z
- Validation: LLM_MODE=true castor check PASS (314.6s); 4468 unit/integration tests, 16260 assertions PASS; controller replay 10 tests/135 assertions PASS; TUI replay 38 tests/193 assertions PASS; llm-real 15 tests/186 assertions PASS; deptrac/phpstan/cs-check PASS; llama-proxy cache stable 195→195; QA artifact integrity PASS; QA leak check PASS
- Summary: Post-merge integration validation completed on main commit 57835eaa7. Also reconciled two older merged-but-unclosed tasks/worktrees (PRs #331 and #334): both clean worktrees removed and tasks moved CODE-REVIEW→DONE. Remaining worktrees are active OM-05 and TUI architecture spike only.

## Task workflow update - 2026-08-06T20:58:59.614Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
