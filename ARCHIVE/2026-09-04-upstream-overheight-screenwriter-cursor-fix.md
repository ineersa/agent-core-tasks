# Upstream the overheight ScreenWriter cursor fix to Symfony TUI

## Goal
Prepare a Symfony TUI issue and pull request for the terminal presentation defect fixed locally by agent-core PR #459.

Hatfield reproduced the defect in both GNOME Terminal and Kitty without tmux. A differential frame taller than the physical terminal can remain partially presented until another cursor command arrives. The renderer has already emitted the complete frame; a later cursor-only write makes the terminal display settle. Hatfield currently works around this by copying Symfony TUI ScreenWriter at revision `94c5b0ecce2c5ce41ab9460b84f48b2ab1366baf`, then deferring one final cursor commit to the next Revolt event-loop turn after an overheight changed frame.

Review Symfony issue #64941, which requested a replaceable ScreenWriter but did not add an injection seam. Reproduce against the latest Symfony TUI revision before proposing a patch. Prefer fixing Symfony's own ScreenWriter if the behavior still reproduces. If maintainers prefer customization instead, propose the smallest constructor or factory seam that removes Hatfield's `class_alias` workaround.

Reference Hatfield PR #459 and its investigation evidence, but reduce the upstream report to a minimal Symfony-only reproduction. Do not include Hatfield prompts, transcripts, or session content.

## Acceptance criteria
- Confirm whether the defect reproduces against the latest Symfony TUI revision in at least one real terminal emulator.
- Create a minimal Symfony-only reproduction that emits an overheight differential frame and demonstrates that a later cursor-only write settles the display.
- Open or update a Symfony issue with the reproduction, terminal versions, expected behavior, actual behavior, and root-cause evidence.
- Submit a minimal Symfony pull request that either fixes the overheight cursor commit in ScreenWriter or adds an approved ScreenWriter injection seam.
- Add deterministic automated coverage without arbitrary sleeps or timing-window assertions.
- Link the Symfony issue and pull request in the Hatfield task and in agent-core PR #459 or its follow-up record.
- After an upstream release contains the fix or seam, create a Hatfield cleanup task to delete DeferredCursorCommitScreenWriter, its alias installer, drift guard, and dead-code exception.

## Workflow metadata
Status: DONE
Branch:
Worktree:
Fork run:
PR URL: https://github.com/symfony/symfony/pull/65966
PR Status: merged
Started:
Completed: 2026-09-10T23:53:20+00:00

## Work log
- Created: 2026-09-04T16:42:52+00:00

## Task workflow update - 2026-09-07T01:48:25+00:00
- Summary: Upstream plan revised after user-confirmed no-defer fix in PR #475. GitHub #460 now targets synchronized cursor restoration, not the deferred timer workaround. This update supersedes the original deferred-commit implementation guidance.
- Current local candidate: ec590f39b / https://github.com/ineersa/agent-core/pull/475, CODE-REVIEW with full gate passed. New class SynchronizedCursorScreenWriter hides cursor while painting and restores final position/visibility before ending synchronized output. Deferred scheduling/cancellation and old tests removed.
- When starting upstream work, reproduce against latest Symfony and propose minimal frame-ordering fix with deterministic output tests. Do not upstream the next-turn timer. After upstream release, remove SynchronizedCursorScreenWriter, SynchronizedCursorScreenWriterAliasInstaller, drift guard, and dead-code exception. Historical GNOME/Kitty evidence remains historical; current user manual confirmation is not universal emulator proof.
- Updated issue: https://github.com/ineersa/agent-core/issues/460. Keep open for upstream fix and later cleanup.

## Task workflow update - 2026-09-07T15:55:10+00:00
- Summary: User observed recurring partial rendering on a large main session after synchronized-only fix and explicitly requested restoring deferred cursor commit. Restored original overheight deferred scheduling/cancellation alongside synchronized restoration in main working copy. Earlier conclusion that synchronization makes deferred commit unnecessary is withdrawn. Upstream work must account for both behaviors; do not remove deferred workaround based solely on short-session or virtual output evidence.

## Task workflow update - 2026-09-09T21:58:30+00:00
- Summary: User requested work directly in /home/ineersa/projects/symfony, with no commits or PR. Created fix/tui-screenwriter-cursor-sync and fast-forwarded it to upstream 8.2 at 4a4b29d999b47094664fbebd9a04566414c8d0bb. Uncommitted candidate moves synchronized-output completion after cursor restoration in all four repaint paths. No deferred workaround added. Issue-template draft saved to agent-core/.hatfield/tmp/symfony-screenwriter-cursor-issue.md. Tests and fresh real-terminal reproduction remain outstanding; this is not a validated resolution of the historical recurring partial-presentation symptom. Board transition was not performed because the standard transition creates an agent-core worktree rather than using the user-designated Symfony checkout.
- Ownership: owner=main; fork_run=none; revision=4a4b29d999b47094664fbebd9a04566414c8d0bb; scope=Symfony ScreenWriter upstream candidate and issue draft in /home/ineersa/projects/symfony; outcome=assigned; commit=none

## Task workflow update - 2026-09-09T22:07:07+00:00
- Summary: Expanded the uncommitted Symfony branch with cursor hiding across all repaint paths, deferred cursor-only commits for overheight repaints with markers, callback unreferencing, and cancellation on incoming frames/reset/getState. Cancellation occurs before processing a new frame, including the unchanged-frame fast path and failures, to avoid stale callbacks. Updated issue markdown with user-reported several-day experience, subjective >90% reduction, residual symptoms, and the distinction between empirical mitigation and root-cause proof. No commits, pushes, PRs, or issue publication. Automated tests and fresh upstream terminal validation still pending.

## Task workflow update - 2026-09-09T22:24:35+00:00
- Validation: ScreenWriter suite: 53 tests, 120 assertions passed.; Full TUI component suite: 1999 tests, 5876 assertions passed; 3 existing PHPUnit config/deprecation-trigger warnings and 1 pre-existing PHPUnit deprecation.; CS Fixer changed-file check passed, no fixes; PHP 8.5 runtime advisory.; PHPStan ScreenWriter level 7 passed; not monorepo baseline analysis.; Reproduce: cd /home/ineersa/projects/symfony && castor --castor-file=/home/ineersa/projects/agent-core/.hatfield/tmp/symfony-qa/castor.php test-tui
- Summary: Added deterministic upstream ScreenWriter tests via fork: synchronization ordering on repaint paths including nonvirtual redrawViewport, cursor-only deferred write, latest cursor replacement, reset/getState/short/unfocused/unchanged cancellation, unreferenced callback. Existing cursor visibility expectation updated for hide-during-paint. Symfony root contains only two intended modified files plus pre-existing .idea. No commit/PR. Package checks passed; full Symfony monorepo CI not run because sparse checkout lacks root dependency/bootstrap installation. Temporary reproducible Castor wrapper moved to agent-core/.hatfield/tmp/symfony-qa/castor.php.
- Ownership: owner=fork; fork_run=agent_3b43582b431ddd3d; revision=4a4b29d999b47094664fbebd9a04566414c8d0bb; scope=Symfony ScreenWriter deterministic tests and package validation; outcome=completed; commit=none

## Task workflow update - 2026-09-10T00:04:51+00:00
- Updated PR URL: https://github.com/symfony/symfony/pull/65966
- Updated PR Status: open
- Summary: Upstream submission completed: Symfony issue #65965 and PR #65966. Branch fix/tui-screenwriter-cursor-sync pushed as one commit 31bb920a71 with explicit force-with-lease after user authorization. PR is currently open with no approval recorded. Standard DONE transition cannot apply yet and would incorrectly invoke an agent-core integration merge for this external Symfony contribution. Await upstream review/merge; retain Hatfield workaround until a released upstream fix is adopted. Hatfield issue #460 remains open for tracking.

## Task workflow update - 2026-09-10T23:53:15+00:00
- Updated PR Status: merged
- Summary: User requests closure following merge of Symfony PR65966 (merge1e00f6cd04e33c622bee9215e5c91f75e9b80792). Maintainer revision fe41d6c retains synchronized cursor restoration and replaces deferred commits with overheight viewport redraw. Upstream contribution complete; Hatfield custom ScreenWriter retained pending adoption/validation, tracked separately in #460.

## Task workflow update - 2026-09-10T23:53:20+00:00
- Moved TODO → DONE.
- No Branch/Worktree metadata found; moved task without git merge.
- Summary: External Symfony contribution merged as PR65966. No agent-core task branch or worktree: close board record only. Keep local ScreenWriter cleanup tracked separately in #460.
