# Prevent Symfony TUI viewport redraws from filling tmux scrollback

## Goal
Track and fix agent-core issue #512. Prepare the smallest upstream Symfony TUI ScreenWriter change that preserves correct bottom-viewport rendering without using a whole-screen erase that tmux archives into history. Work against Symfony TUI 8.2 and verify the patch in isolation before updating Agent Core. Do not commit a vendor patch or restore a full application-owned ScreenWriter copy.

## Acceptance criteria
- Keep `redrawViewport()` and its existing bottom-slice, synchronized-output, cursor, and state behavior.
- Replace the `CSI 2J` whole-screen erase with in-place repainting: position the cursor at each terminal row, erase that row with `CSI 2K`, and write the corresponding viewport line or leave the row blank.
- Preserve the viewport correctness cases covered by Symfony PR #65861, including one-line and three-line trailing shrinkage.
- Add upstream tests that verify the visible viewport and emitted ANSI behavior without duplicating implementation details unnecessarily.
- Add one deterministic tmux proof that repeated grow-and-shrink cycles do not add a screenful of history.
- Prepare the upstream Symfony monorepo and standalone TUI changes with focused validation. Do not add an Agent Core compatibility copy or Composer patch.
- Reference agent-core issue #512 and record upstream branch or PR status.

## Workflow metadata
Status: DONE
Branch: task/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/515
PR Status: merged
Started: 2026-09-19T19:23:59+00:00
Completed: 2026-09-19T23:58:17+00:00

## Work log
- Created: 2026-09-19T19:22:34+00:00

## Task workflow update - 2026-09-19T19:23:59+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback/.idea.
- Validation: Standalone Symfony TUI reproduction already confirmed 20 grow/shrink cycles add 425 tmux history lines in a 100x20 pane.
- Summary: Starting implementation from agent-core issue #512 and the user-approved in-place repaint candidate.

## Task workflow update - 2026-09-19T19:24:46+00:00
- Summary: Routing pass complete. The change belongs in upstream Symfony TUI `ScreenWriter::redrawViewport()`. Existing `ScreenWriterTest` already proves the bottom-slice cases from Symfony PR #65861 and uses `ScreenBuffer` to model terminal state. The new proof must also reject `CSI 2J` and verify that repeated shrink redraws do not create scrollback. Agent Core should not carry a copied writer or vendor patch.
- Ownership: owner=main; fork_run=none; revision=Symfony upstream-gh/8.2 11aca3b1630da02e852a794a28c2a5d92212d66b; scope=upstream ScreenWriter in-place viewport repaint, focused upstream tests, standalone tmux proof, and split-repository synchronization; outcome=assigned; commit=none

## Task workflow update - 2026-09-19T19:37:01+00:00
- Validation: Red proof: the four existing overheight-shrink data cases failed after adding the no-`CSI 2J` and five-row erase assertions.; Focused upstream proof: 4 cases, 16 assertions passed.; Full standalone Symfony TUI component suite: 2,036 tests, 6,269 assertions passed; one existing PHPUnit deprecation remained.; Symfony PHP-CS-Fixer dry run on both changed files: 0 files to fix.; Standalone tmux probe at 100x20: 20 grow/shrink cycles reduced history growth from the prior 425-line reproduction to 25 lines; the final viewport was correct.; Agent Core task worktree: `castor test:tui --filter=TuiSubagentProgressE2eTest` passed with 1 test and 18 assertions. PHAR staging `ScreenWriter.php` matches the patched monorepo source byte-for-byte.
- Summary: Implemented the candidate fix in both local Symfony repositories. `ScreenWriter::redrawViewport()` now keeps the existing bottom slice, synchronized output, cursor restoration, and writer state, but repaints each terminal row with absolute positioning plus `CSI 2K` instead of `CSI 2J`. The task Agent Core worktree uses the monorepo component through an uncommitted Composer path override, and its PHAR contains the patched writer. No branch was pushed and no upstream PR was opened in this phase.
- Ownership: owner=main; fork_run=none; revision=Symfony 249d39e77a2e62c9b32483bb6666b57ae6771e4a + standalone TUI 5c7261c6629e5e74914a723d3faf7f2d710abbc6; scope=upstream ScreenWriter in-place viewport repaint, focused upstream tests, standalone tmux proof, and Agent Core local dependency trial; outcome=completed; commit=249d39e77a2e62c9b32483bb6666b57ae6771e4a

## Task workflow update - 2026-09-19T20:04:35+00:00
- Validation: `git verify-commit HEAD` reports a good signature from Illia Vasylevskyi for Symfony commit `8168691ca0` and standalone TUI commit `fd870c9`.; Pushed `fix/tui-screenwriter-in-place-viewport` to `ineersa/symfony` and `ineersa/tui`. The committed trees are unchanged from the previously validated revisions.
- Summary: Re-signed both unchanged implementation commits after pinentry was restored and pushed matching feature branches to the two forks. No force push was needed because neither branch existed remotely.
- Ownership: owner=main; fork_run=none; revision=Symfony 8168691ca0 + standalone TUI fd870c9; scope=sign and push the completed upstream renderer fix to both forks; outcome=completed; commit=8168691ca0

## Task workflow update - 2026-09-19T20:51:33+00:00
- Validation: Focused ScreenWriter history tests: 9 tests, 74 assertions passed.; Full standalone Symfony TUI suite: 2,041 tests, 6,327 assertions passed (1 existing PHPUnit deprecation).; PHP CS Fixer dry-run: 0 files to fix.; Real 100×20 tmux reproduction: history_size=6 total (four initial transcript rows plus completion-output scrolling), with no repeated accumulation across 20 oscillations.; Agent Core patched-PHAR smoke: castor test:tui --filter=TuiSubagentProgressE2eTest passed (1 test, 18 assertions); PHAR rebuilt and source matched patched ScreenWriter.
- Summary: Extended the initial in-place repaint patch after advisor review. ScreenWriter now separates viewport repainting from native history progress: overheight height changes stage only previously uncommitted prefix rows into scrollback, then repaint the bottom viewport. This prevents grow-after-shrink cycles from repeatedly archiving the restored top row while preserving genuine history growth, including 24→23→25 crossover and growth larger than one viewport. Signed follow-up commits pushed: ineersa/symfony ee5824eef645f4663130199d0d8baa864811683a and ineersa/tui 396010011d26070273381b8157d814ddd28593db.

## Task workflow update - 2026-09-19T21:57:50+00:00
- Validation: Focused ScreenWriter tests: 12 tests, 84 assertions passed.; Full standalone Symfony TUI suite: 2,044 tests, 6,337 assertions passed (1 existing PHPUnit deprecation).; PHP CS Fixer dry-run: 0 files to fix.; Real 100×20 tmux reproduction: history_size=6 after 20 oscillations; Transcript line 04 appears only as the documented history/viewport boundary overlap, not repeated accumulation.; Agent Core patched PHAR rebuilt successfully; castor test:tui --filter=TuiSubagentProgressE2eTest passed (1 test, 18 assertions).; Remote branch heads verified at Symfony 4ed4649413e5f1b6bbd4f6633174364f8863f08d and TUI b9bc279d3c2081897ce1f124d4f64d23d2653715.
- Summary: Addressed advisor review regressions. Ordinary first overflow now remains on the natural differential path, preserving terminal output that predates ScreenWriter. Changes to an already committed prefix invalidate the history watermark; subsequent growth uses the existing clear-and-rebuild fallback rather than archiving incorrect indexes. Added exact history tests for pre-existing shell output, insertion into archived prefix, and truncate→replace→regrow. Unsigned commits pushed per user instruction: ineersa/symfony 4ed4649413e5f1b6bbd4f6633174364f8863f08d; ineersa/tui b9bc279d3c2081897ce1f124d4f64d23d2653715. User plans to sign later.

## Task workflow update - 2026-09-19T22:16:40+00:00
- Validation: Focused ScreenWriter tests: 13 tests, 89 assertions passed.; Full standalone Symfony TUI suite: 2,045 tests, 6,342 assertions passed (1 existing PHPUnit deprecation).; PHP CS Fixer dry-run: 0 files to fix.; Real 100×20 tmux oscillation reproduction remains at history_size=6.; Agent Core worktree vendor refreshed from patched component; PHAR rebuilt; castor test:tui --filter=TuiSubagentProgressE2eTest passed (1 test, 18 assertions).
- Summary: Addressed the final advisor P2: differential rendering now keeps historyCommittedThrough monotonic instead of lowering it to the current viewport top. Added the exact overheight grow→shrink→same-height edit→regrow regression sequence. Unsigned commits pushed per prior user instruction: ineersa/symfony 0bbda534567e and ineersa/tui f1e8dd5.

## Task workflow update - 2026-09-19T23:46:22+00:00
- Summary: Squashed and GPG-signed the upstream fixes, then force-pushed with explicit user approval. Symfony monorepo branch now df32269abb22c28293fe201ecb77b8850b4ce9ca; standalone TUI branch now e682d162f31aa82832d8255f4c9c3fd17977e903. Agent Core Composer configuration now resolves symfony/tui from the ineersa/tui fix branch instead of an absolute path repository.
- Ownership: owner=main; fork_run=none; revision=e682d162f31aa82832d8255f4c9c3fd17977e903; scope=point Agent Core Composer dependency at signed ineersa/tui fix branch; outcome=completed; commit=none

## Task workflow update - 2026-09-19T23:49:57+00:00
- Summary: Independent reviewer APPROVE at Agent Core revision a0be908876526c775647513142e73ea18796ac34. No correctness, security, dependency, lock-consistency, or specification-fidelity findings. Reviewer verified the fork branch head, valid GPG signature, exact parent matching the previously locked upstream revision, minimal two-file upstream delta, Composer validation, and installed vendor identity.
- Review: role=reviewer; artifact=agent_bdde82f83bb105d3; revision=a0be908876526c775647513142e73ea18796ac34; scope=Composer fork pin, lock consistency, supply-chain provenance, specification fidelity; verdict=APPROVE

## Task workflow update - 2026-09-19T23:52:27+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback.md.
- Session/run: 59.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback/var/reports/qa-20260919-235019-1134-139823af.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-19T23:55:20+00:00
- Validation: Failed CODE-REVIEW gate: test lane, stale PickerOverlayTest CSI 2J expectation; report var/reports/qa-20260919-235019-1134-139823af/check-test.log.; castor test --filter=PickerOverlayTest: 12 tests, 139 assertions passed.; castor cs-check --path=tests/Tui/Picker/PickerOverlayTest.php: 0 files fixed.
- Summary: CODE-REVIEW gate failed only because PickerOverlayTest still required the old CSI 2J behavior on overheight close. Updated the test to require no whole-screen clear, retain strict no-repaint assertions for fits-viewport transitions, permit in-place repaint for overheight transitions, and verify final screen contents. Independent re-review APPROVE at 7d38fb415.
- Review: role=reviewer; artifact=agent_bdde82f83bb105d3; revision=7d38fb415; scope=PickerOverlayTest adaptation after failed CODE-REVIEW gate; verdict=APPROVE

## Task workflow update - 2026-09-19T23:56:24+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (50.4s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback/var/reports/qa-20260919-235534-5620-1c37f142.
- Session/run: 59.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-19T23:56:26+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback/var/reports/qa-20260919-235534-5620-1c37f142.
- Session/run: 59.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-19T23:56:28+00:00
- castor check passed (50.4s).
- Pushed task/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback to origin.
- Created PR: <url>
- Session/run: 59.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-19T23:56:28+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (50.4s).
- Pushed task/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/515

## Task workflow update - 2026-09-19T23:58:17+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback into integration checkout.
- Merge made by the 'ort' strategy.
 composer.json                          |  6 +++++-
 composer.lock                          | 45 ++++++++++++++++-----------------------------
 tests/Tui/Picker/PickerOverlayTest.php | 14 +++++++-------
 3 files changed, 28 insertions(+), 37 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-19-prevent-symfony-tui-viewport-redraws-from-filling-tmux-scrollback.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #515 merged on GitHub at 6b9a8fdb75a1109e51527ea36ae48deb676dd3f7. Agent Core now resolves symfony/tui from signed fork commit e682d162f31aa82832d8255f4c9c3fd17977e903.

## Task workflow update - 2026-09-20T00:01:34+00:00
- Validation: Initial post-merge castor check failed: stale vendor/symfony/tui caused PickerOverlayTest to exercise upstream CSI 2J; TuiProviderErrorE2eTest teardown also reported one live pane. QA: var/reports/qa-20260919-235828-10945-f2b654fb.; COMPOSER_ROOT_VERSION=dev-main composer install installed symfony/tui e682d162f31aa82832d8255f4c9c3fd17977e903; installed ScreenWriter matched the signed fork.; castor test --filter=PickerOverlayTest: 12 tests, 139 assertions passed.; castor test:tui --filter=TuiProviderErrorE2eTest: 1 test, 6 assertions passed.; Integrated castor check passed in 158.5s: 5,116 unit tests/22,097 assertions; 11 controller replay tests/175 assertions; 9 TUI tests/69 assertions; 5 llm-real tests/30 assertions; all static, architecture, docs, style, dead-code, LSP, catalog, artifact-integrity, leak, and cache guards passed. QA: var/reports/qa-20260920-000021-15440-8e248993.; Integration checkout clean; task worktree removed.
- Summary: Post-merge validation complete on integrated main revision 669a3de8782adb43325b74486e679c4c3016c4b8. The first run exposed stale installed vendor code because the merge changed composer.lock without updating vendor; after composer install resolved signed fork commit e682d162, the focused picker and TUI provider-error tests passed and the full integrated gate passed. Main is clean and the task worktree is removed.
