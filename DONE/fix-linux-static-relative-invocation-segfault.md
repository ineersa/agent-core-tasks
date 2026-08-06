# Fix Linux static binary relative-invocation startup segfault

## Goal
v0.0.2 release run 30386020360 built and passed topology on all four runners, but final publish failed when invoking `./var/tmp/dist/hatfield.linux-amd64 --version` (SIGSEGV 139). Downloaded exact workflow artifact reproduces locally: relative invocation always segfaults, while absolute path and PATH-resolved invocation succeed. Core/strace prove startup crash in musl `realpath()` called by pinned phpmicro `micro_get_filename()` on `getauxval(AT_EXECFN)` when relative exec filename lies at a page boundary; vectorized scan crosses into unmapped memory. Not artifact corruption, CPU ISA, or shutdown regression. v0.0.2 release has zero assets.

## Acceptance criteria
- Patch the exact pinned phpmicro source deterministically so Linux resolves self through `/proc/self/exe` rather than passing page-boundary AT_EXECFN storage to realpath; patch content and SHA are repository-pinned and fail closed before build.
- Static source checkout remains pinned to phpmicro commit fb6d497b; cached/dirty checkouts are reset and patched idempotently before SPC build.
- Native artifact smoke executes through a relative path/symlink from isolated CWD so the exact v0.0.2 regression fails during each static build, before artifact upload.
- Absolute/native topology, version/commit identity, canonical PHAR handoff, checksums, release-only trigger, and four-platform gates remain unchanged.
- Document the pinned Linux phpmicro patch rationale and removal condition concisely in static packaging docs/pin notes.
- Focused Castor QA and reviewer pass; release validation requires a new tag after merge and all six assets publish.

## Workflow metadata
Status: DONE
Branch: task/fix-linux-static-relative-invocation-segfault
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-linux-static-relative-invocation-segfault
Fork run: u2wquh386unu
PR URL: https://github.com/ineersa/agent-core/pull/331
PR Status: merged
Started: 2026-07-28T18:49:15.954Z
Completed: 2026-07-29T17:36:56.054Z

## Work log
- Created: 2026-07-28T18:49:06.652Z

## Task workflow update - 2026-07-28T18:49:15.954Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-linux-static-relative-invocation-segfault.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-linux-static-relative-invocation-segfault.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-linux-static-relative-invocation-segfault.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-linux-static-relative-invocation-segfault.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-linux-static-relative-invocation-segfault.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-linux-static-relative-invocation-segfault.
- Validation: Release run 30386020360: PHAR + all four static build/topology jobs PASS; aggregate relative-path version smoke FAIL exit 139; publish skipped; Downloaded artifact sha256 475a97dfd2d8747823cc9c0227bc558fb850542b0193515f4111ff0cdf820922 reproduces: `./hatfield.linux-amd64 --version` exit 139, absolute path exit 0, PATH invocation exit 0; Core backtrace: realpath; rsi points to `./hatfield.linux-amd64` at page end; vector load crosses page; Pinned phpmicro fb6d497b Linux micro_get_filename uses realpath(getauxval(AT_EXECFN))
- Summary: Starting root fix from v0.0.2 final publish SIGSEGV. Exact downloaded artifact reproduced locally; core points to phpmicro Linux realpath(AT_EXECFN) with relative exec filename at page boundary.

## Task workflow update - 2026-07-28T18:50:02.502Z
- Recorded fork run: u2wquh386unu
- Validation: Local exact artifact: relative invocation 10/10 SIGSEGV; absolute and PATH invocation PASS; strace: crash immediately after exec startup; core: realpath -> vector scan of AT_EXECFN at page boundary; All v0.0.2 build/topology jobs passed; publish skipped at relative-path host smoke; release has zero assets
- Summary: Implementation fork launched for deterministic one-line pinned phpmicro Linux self-path patch plus relative-path build smoke. Rejected CPU flags/workflow absolute-path workaround because exact core proves AT_EXECFN page-boundary realpath crash and users must be able to run `./hatfield`.

## Task workflow update - 2026-07-28T18:56:00.769Z
- Recorded fork run: u2wquh386unu
- Validation: Production patch preparation invoked twice: idempotent; patched source contains /proc/self/exe and no getauxval(AT_EXECFN); castor distribution:info PASS; castor test --filter='CanonicalPharHandoffTest|BuildDistributionScriptTest' PASS (5 tests, 53 assertions); castor deptrac PASS (0 violations); castor phpstan PASS (0 errors); castor cs-check PASS; castor clean:cleanup:workers:list PASS (none); Host fused binary build not run: missing re2c/flex/gperf; CI release remains required for artifact-level proof
- Summary: Implementation complete at 17ebb05d0: pinned one-line phpmicro Linux self-path patch (`/proc/self/exe`, SHA-256 4b9b1937…), fail-closed idempotent patch preparation, relative-path native artifact smoke, focused regression contract and concise docs. 6 files +133/-11; clean worktree, not pushed.

## Task workflow update - 2026-07-28T19:21:56.515Z
- Validation: Reviewer verdict APPROVE — no critical issues, no issues, no security blockers; Ponytail: Lean already. Ship.; castor test PASS (4415 tests, 15523 assertions); castor deptrac PASS (0 violations); castor phpstan PASS (0 errors); castor cs-check PASS (files_fixed=0); castor clean:cleanup:workers:list PASS (no stale QA workers); git status clean at 17ebb05d0f6dc30a9a0c20ddf6493ed36d7a8fda
- Summary: Task-to-PR review APPROVE on 17ebb05d0. Reviewer confirmed root-cause fidelity, patch/SHA and shell/path safety, CRLF applicability to pinned upstream blob, idempotent cached checkout handling, relative symlink exec semantics, and lean scope. No critical/actionable findings; worktree clean.

## Task workflow update - 2026-07-28T19:24:10.273Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (119.7s).
- Pushed task/fix-linux-static-relative-invocation-segfault to origin.
- branch 'task/fix-linux-static-relative-invocation-segfault' set up to track 'origin/task/fix-linux-static-relative-invocation-segfault'.
- Created PR: https://github.com/ineersa/agent-core/pull/331
- Validation: Reviewer APPROVE; castor test PASS (4415 tests, 15523 assertions); castor deptrac PASS; castor phpstan PASS; castor cs-check PASS; No stale QA workers; Worktree clean at 17ebb05d0
- Summary: Approved root fix ready for PR: pinned phpmicro Linux `/proc/self/exe` patch, fail-closed idempotent application, and relative native smoke that catches the v0.0.2 startup crash before upload.

## Task workflow update - 2026-07-29T17:36:56.054Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-linux-static-relative-invocation-segfault into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-linux-static-relative-invocation-segfault.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-linux-static-relative-invocation-segfault.
- Pulled integration checkout: Already up to date..
- Validation: PR #331 state MERGED; Worktree clean and branch matches origin; v0.0.3 release artifact previously validated successfully
- Summary: Found as an orphaned clean worktree/task after PR #331 had already merged on 2026-07-28 as ce7671a627099d88ca29c75ea4a34e69c12902bd. Marking DONE and cleaning worktree.
