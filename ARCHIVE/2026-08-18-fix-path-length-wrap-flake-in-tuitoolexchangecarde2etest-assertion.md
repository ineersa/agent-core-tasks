# Fix path-length wrap flake in TuiToolExchangeCardE2eTest assertion

## Goal
TuiToolExchangeCardE2eTest::testEditToolExchangeCardShowsCompactedBodyWithoutFileContextLeak (tests/Tui/E2E/TuiToolExchangeCardE2eTest.php, from tui-04 45a6a97c1) fails on the integration checkout but passes in task worktrees.

Root cause (verified 2026-08-18): the compacted edit-success body line "Applied patch to <checkout>/var/tmp/tui-e2e-tool-exchange-XXXX/target.txt (1 addition, 1 deletion)" is 137 chars on integration; the 120-col tmux pane soft-wraps it EXACTLY between "1" and "addition", splitting the asserted phrase. Wrap position depends on checkout absolute path length (temp dir lives under <checkout>/var/tmp/), so worktrees with longer paths wrap inside the path harmlessly.

Fix (user-specified): strip/normalize line breaks in the captured pane text before assertStringContainsString — e.g. collapse \s+ to single spaces once for the full capture, then assert all phrases against the normalized string. Apply to all phrase assertions in this test that can be wrapped ('1 addition, 1 deletion' etc.).

Check first whether a whitespace-normalizing capture helper already exists (TmuxHarness::capturePlain*, TuiE2eTestCase helpers, or a similar pattern in sibling TuiE2e tests) and reuse it; otherwise add the smallest local helper in the test.

Repro: HATFIELD_TUI_E2E=1 vendor/bin/phpunit --configuration phpunit.xml.dist --filter testEditToolExchangeCardShowsCompactedBodyWithoutFileContextLeak tests/Tui/E2E/TuiToolExchangeCardE2eTest.php (run from /home/ineersa/projects/agent-core — the long-path checkout where it fails; note it may PASS in the task worktree by path-length luck, so prove via normalization reasoning + integration run at task-done).

Scope: this test file only. Do NOT sweep other tests.

## Acceptance criteria
- testEditToolExchangeCardShowsCompactedBodyWithoutFileContextLeak passes on the integration checkout (long path) and in worktrees — repro command provided in task file
- Normalization is applied at assertion time only; no production code changes
- Reuse existing helper if one already exists (TmuxHarness/TuiE2eTestCase); else add smallest local helper
- castor test:tui green; castor test, cs-check, phpstan green

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion
Fork run: 96rpc6hp1cqh
PR URL: https://github.com/ineersa/agent-core/pull/416
PR Status: merged
Started: 2026-08-18T22:53:51.446Z
Completed: 2026-08-18T23:43:05.780Z

## Work log
- Created: 2026-08-18T22:53:41.672Z

## Task workflow update - 2026-08-18T22:53:51.446Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion.

## Task workflow update - 2026-08-18T23:02:49.128Z
- Recorded fork run: 96rpc6hp1cqh
- Validation: castor test:tui: PASS 40/339 (136.2s); castor test: PASS 4696/18762; castor phpstan: errors=0; castor cs-check: clean; Wrap-proof php -r demo: PASS; Reviewer: APPROVED
- Summary: Fix via fork 96rpc6hp1cqh: commit cd0e76584, 1 file +8/−4 (tests/Tui/E2E/TuiToolExchangeCardE2eTest.php). preg_replace('/\s+/',' ') collapse of pane capture before all 4 phrase assertions (3 positive + 1 negative), with why-comment; raw capture still feeds saveAnsiSnapshot. No existing reusable helper (TmuxHarness::normalizeSnapshot is snapshot scrubbing, wrong semantics). Reviewer APPROVED (0 blockers; notes normalization strengthens the leak negative-assert against wrap false-pass). NTH reports: TuiRichTranscriptProductValidationE2eTest same latent class with short tokens (safe now), waitForCallback gates raw-captured (pre-existing).
- 2026-08-18: fork 96rpc6hp1cqh verified (cd0e76584, 1 file +8/−4); reviewer APPROVED; moving to CODE-REVIEW

## Task workflow update - 2026-08-18T23:05:29.469Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (149.3s).
- Pushed task/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion to origin.
- branch 'task/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion' set up to track 'origin/task/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion'.
- Created PR: https://github.com/ineersa/agent-core/pull/416

## Task workflow update - 2026-08-18T23:05:32.662Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/416
- Updated PR Status: open
- 2026-08-18: CODE-REVIEW — deterministic castor check passed (149.3s), branch pushed (cd0e76584), PR #416 created

## Task workflow update - 2026-08-18T23:43:05.781Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion.
- Merged task/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion into integration checkout.
- Merge made by the 'ort' strategy.
 tests/Tui/E2E/TuiToolExchangeCardE2eTest.php | 12 ++++++++----
 1 file changed, 8 insertions(+), 4 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-18-fix-path-length-wrap-flake-in-tuitoolexchangecarde2etest-assertion.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-08-18T23:46:03.797Z
- Updated PR Status: merged
- Validation: PR #416 merged; task branch merged into integration (1 file +8/−4), worktree + branch cleaned, JetBrains project closed; Integration checkout repro (previously failing): OK (1 test, 4 assertions) — flake dead on the long-path checkout; Post-merge LLM_MODE=true castor check: ALL 8 LANES OK — test 4696/18762, controller-replay 13/252, test:tui 40/333, llm-real 13/144, deptrac, phpstan, cs-check, docs:validate — exit 0
- 2026-08-18: task-done — merged; integration repro passes; full castor check 8/8 green, exit 0

## Task workflow update - 2026-08-19T18:16:23.090Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
