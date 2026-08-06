# Refactor Castor helpers to use Symfony components instead of reinvented primitives

**CANCELLED 2026-06-28** — Audit-driven refactor with no reachable behavior change; not worth the churn. Re-analyzed against AGENTS.md test-value rules ("protects a user-visible behavior / stable contract / safety boundary / observed bug"):

- `phar_build_with_lock()` → Symfony Lock: the only acceptance-mandatory item, but it replaces working `flock(LOCK_EX|LOCK_NB)` + 60s deadline + double-check-after-acquire code that has **no defect**. Pure style churn.
- shell `curl` → HttpClient (LLM proxy stats, generation-ready probe): robust already (`timeout`/`-m`/`-f`), no bug.
- TTL `touch`+`filemtime` → Symfony Cache: 4 lines of dead-simple code; swapping adds moving parts for no gain (arguably worse).
- `latest_file_mtime()` / test-detection → Symfony Finder: `RecursiveIteratorIterator` is fine.
- One latent defect exists — `cleanup.php::rmtree()` calls `rmdir()` on a symlink-to-dir (fails) — but it's **unreachable**: all targets are `var/tmp/test-*` / `var/tmp/tui-e2e-*` dirs freshly created by the test harness, never containing symlinks-to-dirs.
- Process/session reaping + `/proc` stale-worker detection stays custom (Symfony Process can't do PGID/session/`/proc`) — unchanged either way.

Net estimate ~−130/+100 (≈−30 net) for zero reachable behavior change. AGENTS.md explicitly discourages broadening tasks and coverage-only churn. Task's own acceptance criteria are full of "or document rationale" escape hatches, which is the audit author hedging because there's no compelling case.

If a reachable defect ever surfaces (e.g. a symlink genuinely lands under a cleanup target), file a focused bug task instead of reviving this broad audit.

## Goal
Follow-up from ISSUE-208 audit. Castor currently has several homegrown helpers where Symfony components are already installed and available in Castor context. Refactor the appropriate low-risk areas to use Symfony components, while preserving custom process/session handling where Symfony Process is not suitable.

Audit highlights:
- `symfony/lock`, `symfony/http-client`, `symfony/filesystem`, `symfony/finder`, `symfony/cache`, `symfony/process`, `symfony/stopwatch` are installed.
- Immediate candidate: `.castor/helpers.php::phar_build_with_lock()` still uses manual `flock()`; should use Symfony Lock like the `castor check` lock.
- Follow-up candidates: curl-based LLM/proxy HTTP helpers -> Symfony HttpClient; `cleanup.php::rmtree()` -> Symfony Filesystem; recursive mtime discovery -> Symfony Finder; generation-ready TTL file -> Symfony Cache.
- Keep custom: `.castor/process.php` process-group/session reaping and stale-worker `/proc` detection are deliberate because Symfony Process does not provide session-level cleanup for Hatfield controller/messenger descendants.

Do not mix this broad refactor into ISSUE-208 unless the touched code is already part of that fix. This task should be a focused cleanup with validation.

## Acceptance criteria
- `phar_build_with_lock()` uses Symfony Lock rather than manual `flock()` or the task documents why it cannot.
- curl/shell HTTP calls in LLM readiness and llama-proxy stats are replaced with Symfony HttpClient or explicitly justified with tests/docs.
- Simple recursive filesystem cleanup uses Symfony Filesystem/Finder where appropriate, without changing cleanup semantics.
- Generation-ready TTL cache is either migrated to Symfony Cache or intentionally kept with rationale.
- Custom process/session reaping remains custom with comments explaining why Symfony Process is insufficient.
- Castor validation passes: `castor cs-check`, `castor phpstan --path=.castor/`, and focused Castor tests relevant to changed helpers.

## Workflow metadata
Status: CANCELLED
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-06-24T21:27:34.630Z
- 2026-06-28: Re-analyzed during task-explain. Scout-confirmed all targets (lock/curl/TTL/Finder/rmtree) replace working code with no reachable defect; only latent bug (rmtree symlink-to-dir) is unreachable in practice. Cancelled as not-worth-the-churn per AGENTS.md test-value rules. Symfony components confirmed available in castor v1.5.0 runtime (http-client/filesystem/finder/cache/lock; stopwatch NOT available).
