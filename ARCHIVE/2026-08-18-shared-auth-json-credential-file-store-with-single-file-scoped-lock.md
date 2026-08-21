# Shared auth.json credential file store with single file-scoped lock

## Goal
From Grok review of PR #409 (2026-08-18). Problem: CodexAuthStorage and GrokAuthStorage lock different keys (codex-auth-* vs grok-auth-*) but read-modify-write the same ~/.hatfield/auth.json. Concurrent token refresh from both providers can drop one provider's entry (last-writer-wins on the whole file). Recoverable via re-login but it's a real data-loss race.

Fix (architect-endorsed): extract a shared AuthCredentialFileStore (src/CodingAgent/Auth/) with ONE file-scoped lock covering the whole auth.json RMW, plus the shared atomic-write/permission (0600) logic. Codex and Grok storage classes become thin key-prefixed wrappers over it — kills ~150 lines of duplicated Codex/Grok storage I/O and closes the race. Deliberate divergence stays deliberate: auth record shapes and refresh policies remain separate.

Explicitly deferred (per same review): shared PKCE login loop (~80 lines, medium abstraction risk — revisit only if a third OAuth provider lands); merging auth records or token refreshers (opposite policies); shared 401-retry helper (rebuild paths diverge); sanitizeWireBody factory extraction; Deptrac Platform layer for Ineersa\Platform (policy question).

Reference: PR #409, review notes in task grok-cli-provider-support.

## Acceptance criteria
- Single file-scoped lock for all RMW on ~/.hatfield/auth.json (keyed per store, race impossible)
- Codex + Grok storage layers use the shared store; ~150 lines duplicated I/O removed
- Existing Codex/Grok auth test suites pass unchanged
- castor test, castor deptrac, castor phpstan, castor cs-check green

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/412
PR Status: merged
Started: 2026-08-18T20:20:46.683Z
Completed: 2026-08-18T20:51:34.384Z

## Work log
- Created: 2026-08-18T20:05:14.021Z

## Task workflow update - 2026-08-18T20:20:46.683Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock.

## Task workflow update - 2026-08-18T20:35:37.763Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (172.7s).
- Pushed task/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock to origin.
- branch 'task/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock' set up to track 'origin/task/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock'.
- Created PR: https://github.com/ineersa/agent-core/pull/412
- Summary: Reviewer APPROVED e45b7811b with no blockers: race closed under single 'auth.json' lock (verified no provider-derived createLock remains in src), public storage APIs identical, existing tests byte-identical, file format/0600/atomic write unchanged, unlocked mutators + lock-held network refresh are pre-existing/documented semantics. Non-blocking notes: public unlocked set/remove (future caller discipline), sequential-not-parallel lock test, consider locked-session object API if a third provider lands. First CODE-REVIEW attempt hit check-lock contention (user's integration castor:check); holder finished, clean retry.

## Task workflow update - 2026-08-18T20:51:34.384Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock.
- Merged task/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock into integration checkout.
- Merge made by the 'ort' strategy.
 src/CodingAgent/Auth/AuthCredentialFileStore.php   | 134 ++++++++++++++++
 src/CodingAgent/Auth/CodexAuthStorage.php          | 121 +++------------
 src/CodingAgent/Auth/GrokAuthStorage.php           | 115 +++-----------
 .../Auth/AuthCredentialFileStoreTest.php           | 168 +++++++++++++++++++++
 4 files changed, 350 insertions(+), 188 deletions(-)
 create mode 100644 src/CodingAgent/Auth/AuthCredentialFileStore.php
 create mode 100644 tests/CodingAgent/Auth/AuthCredentialFileStoreTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-shared-auth-json-credential-file-store-with-single-file-scoped-lock.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: Merged by user via GitHub PR #412 (commit e45b7811b). Delivered: AuthCredentialFileStore with single file-scoped 'auth.json' flock for whole-file RMW, closing the concurrent Codex+Grok token-refresh last-writer-wins race; CodexAuthStorage/GrokAuthStorage thinned to wrappers with identical public APIs; existing tests byte-identical; new race-repro/lock-key/0600/remove tests. castor check 172.7s green, reviewer APPROVED no blockers. Deferred (documented in task): shared PKCE loop, locked-session object API if a third provider lands.

## Task workflow update - 2026-08-19T18:16:29.377Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
