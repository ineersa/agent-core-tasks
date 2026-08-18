# run:agent: launch sessions from a session-owned PHAR copy

## Goal
Follow-up to PR #406 (make-castor-qa-safe-to-run-from-hatfield-sessions). The landed in-use guard makes QA safe but skips PHAR rebuilds while a live session holds the canonical artifact (var/tmp/phar/hatfield.phar), so in-session `castor check` boots a stale artifact in the TUI lane. This implements the task's original option (c): sessions resolve to their own stable copy at startup.

## Design

- `agent_phar_invocation()` (.castor/run.php): keep `phar_ensure()` against the canonical path, then materialize a session-owned copy under `var/tmp/phar/sessions/<run-id or timestamp>/hatfield.phar` and exec from the copy. Sessions never rebuild under themselves; concurrent run:agent sessions are isolated by construction. Copy must be atomic (build to temp + rename) so a crash cannot leave a truncated binary.
- GC: at each launch, remove session copies older than a generous threshold (e.g. 24h by mtime); never touch the copy currently being launched. No background daemon — piggyback on launch.
- Keep the existing phar_in_use_pids()/skip guard as defense in depth (phar:build misuse, future producers). Do not remove it.
- Cost target: ~30-40 lines + one helper-level test + testing-skill note. No new public surface, no changes to distribution/canonical handoff.

## Motivation

- In-session castor check currently boots a stale artifact (visible skip message) — packaging regressions in a diff are only caught at the PR gate, not in casual in-session runs.
- Two concurrent run:agent sessions in one worktree still collide on canonical (guard skips, second session boots stale).

## Workflow metadata
Status: CANCELLED

## Acceptance criteria
- run:agent launches from var/tmp/phar/sessions/<id>/hatfield.phar, not the canonical artifact; byte-identical copy verified
- Concurrent run:agent sessions in one worktree do not affect each other's binaries
- In-session castor check rebuilds canonical freely (TUI artifact lane boots a fresh PHAR) while a run:agent session runs in the same tree
- Old session copies are GC'd at launch (age-based), never the running one; no unbounded growth
- Existing phar guard tests remain green; new helper-level test for copy+GC; testing skill updated
- castor check fully green on implementing branch

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
- Created: 2026-08-18T01:57:05.774Z

## Task workflow update - 2026-08-18T01:58:56.685Z
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
