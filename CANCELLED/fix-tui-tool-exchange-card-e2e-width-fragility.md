# Fix TuiToolExchangeCardE2eTest width-fragile assertion (terminal wrap splits stats string on integration checkout)

## Goal
Post-merge regression: PR #407's new E2E test fails on the integration checkout (castor check → test:tui exit 1) while it passed in the task worktree and the CODE-REVIEW gate.

## Root cause (verified, deterministic)

Assertion `assertStringContainsString('1 addition, 1 deletion', $fullCapture)` at tests/Tui/E2E/TuiToolExchangeCardE2eTest.php:114 is width-fragile:

- EditFileTool result body = `Applied patch to <ABSOLUTE path> (1 addition, 1 deletion)` — path comes from PathResolver::resolve() against the test's temp project dir under the checkout's var/tmp.
- Pane width 120. In the integration checkout the line is ~137 chars → tmux wraps INSIDE the stats: `...(1` / `addition, 1 deletion)` — the space is consumed by the wrap, so neither the raw capture nor newline-stripped capture contains the asserted substring.
- In the worktree the path prefix was longer (agent-core-worktrees/tui-04-deepen-transcript-renderer-boundaries), moving the wrap point mid-path; stats landed intact on line 2 → test passed by path-length luck.

Evidence: var/tmp/tui-e2e-tool-exchange-2019d20b9aaccc75/.hatfield/tmp/tui/smoke/tool-exchange-card-FAILURE-20260818-141618.ansi shows the wrapped card; card otherwise renders correctly (edit card, diff, marker correctly absent).

## Fix (test-only, one test file)

Normalize whitespace before the long-token assertions: collapse runs of whitespace (incl. the wrap boundary) to single spaces, e.g. `$normalized = (string) preg_replace('/\s+/', ' ', $fullCapture);` and run the '1 addition, 1 deletion' + 'Updated file context:' (and any other wrap-prone multi-word) assertions against $normalized. Short tokens (path:, target.txt) are wrap-safe but may as well use the normalized string if simpler. Do not change waits, fixture, isolation, or snapshot behavior.

Sibling TuiRichTranscriptProductValidationE2eTest is unaffected (asserts short tokens like -before/+after) — verify with grep, no changes needed unless it has multi-word assertions near wrap boundaries.

## Validation

- castor test:tui --filter=TuiToolExchangeCardE2eTest in the INTEGRATION checkout (the failing environment) — but implement on task worktree and ALSO verify: the worktree has the LONG path that previously passed, so to prove the fix addresses wrap-splitting, run the filter test from the integration checkout after merge, or temporarily assert in-worktree against normalized capture (mutation-style: confirm the test would fail on the old capture by keeping one un-normalized long assertion? Not required — root cause is proven from the .ansi evidence).
- castor test:tui (full group) green on worktree.

## Acceptance criteria
- TuiToolExchangeCardE2eTest passes on the integration checkout (short-path) and in a worktree (long-path): no width-fragile string assertions remain
- Fix is test-only; no production changes
- castor test:tui --filter=TuiToolExchangeCardE2eTest green in integration checkout; full castor test:tui green

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
- Created: 2026-08-18T14:17:43.011Z

## Task workflow update - 2026-08-18T21:20:51.191Z
- 2026-08-18: Related class observed + fixed on task/providers-catalog branch (separate commit): TuiSkillReadCardVirtualRenderTest failed on long worktree paths (path wraps 'SKILL.md' mid-token; fails at base commit, passes on integration 34-char root). Fix pattern there: normalize whitespace before substring assertion. Same normalization approach likely applies to this task's stats-string wrap.

## Task workflow update - 2026-08-19T00:15:07.701Z
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.

## Task workflow update - 2026-08-19T00:15:16.445Z
- Summary: Duplicate of 2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion — same test, same assertion, same root cause (path-length wrap), same fix. Resolved by PR #416 (commit cd0e76584, merged; integration repro green; castor check 8/8).
- 2026-08-18: cancelled as duplicate — resolved by PR #416 / task 2026-08-18-fix-path-length-wrap-flake (DONE)
