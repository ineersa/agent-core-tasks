# Keep failed tool output collapsed until Ctrl+O expands it

## Goal
## Bug report

The user reports that tool results shown in red after a nonzero exit status dump large output directly into the transcript instead of supporting the normal collapsed and expanded tool-result display. The failure path has not yet been independently reproduced.

## Expected behavior

Use the same collapsed and expanded presentation as regular tool calls. Keep the failure indication visible in the collapsed summary. Show the full available result only in the Ctrl+O expanded view, and collapse it again when toggled back. Do not duplicate the full output in a separate error block.

This is a display change, not a change to tool failure semantics or output retention limits. Investigate the failed-result projection and rendering path before choosing edits.

Related completed work: 2026-09-18-keep-code-mode-diagnostics-in-the-normal-expandable-tool-result, PR #509. Check for reusable behavior without assuming the same cause.

## Acceptance criteria
- A tool call with a nonzero exit status and large output uses the normal bounded collapsed tool-result display.
- The collapsed result still clearly indicates failure.
- Ctrl+O reveals the full available failed-tool result and toggling back restores the collapsed display.
- No separate error block duplicates the large failed-tool output.
- Successful tool-result display and tool failure semantics remain unchanged.
- Automated regression coverage proves collapsed and expanded failure rendering at the lowest correct TUI layer.

## Workflow metadata
Status: DONE
Branch: task/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it
Fork run: 99fa579e-ab6a-5cbe-b400-0ebd20e64cc3
PR URL: https://github.com/ineersa/agent-core/pull/514
PR Status: merged
Started: 2026-09-19T22:13:09+00:00
Completed: 2026-09-20T00:11:14+00:00

## Work log
- Created: 2026-09-19T22:10:47+00:00

## Task workflow update - 2026-09-19T22:13:09+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it/.idea.

## Task workflow update - 2026-09-19T22:14:38+00:00
- Summary: Routing pass found explicit error/cancel/timeout full-render bypass in TranscriptToolRenderer and TranscriptToolResultPreviewWidget. Failure predicate also controls pairing and successful-edit compaction, which must remain unchanged. Runtime already projects failure as ToolResult; no runtime event changes needed. Main owns this cohesive TUI rendering change.
- Ownership: owner=main; fork_run=none; revision=14cca43578d434822636aecb0ed41240194a85f6; scope=failed-tool preview rendering and virtual Ctrl+O regression proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-19T22:20:07+00:00
- Validation: castor test --filter='Transcript|TuiCollapsedToolCard|TuiSkillReadCard|PreviewExpansion|SubagentResultRenderer': PASS, 290 tests / 1850 assertions, suite 1.061s.; Virtual Ctrl+O regressions cover paired and standalone failed bash, generic code_mode, read, skill read, and paired and standalone view_image. Assert bounded collapsed output, error ANSI color, all result lines exactly once when expanded, and re-collapse after second Ctrl+O.; castor phpstan --path=src/Tui: PASS, 0 errors.; castor deptrac: PASS, 0 violations / errors.; castor cs-fix --path=<each changed PHP file>: clean, no changes.; Full castor check and independent review deferred to task-to-pr, as required by task-start.
- Summary: Implemented failed-tool preview behavior in the task worktree. Error results now use bounded normal previews with unchanged error coloring; Ctrl+O expands and collapses the full available result. Failed view_image results use the shared preview renderer instead of unbounded metadata diagnostics. Preserved failure pairing priority, ask_human suppression rules, successful-edit compaction, and successful result behavior. Renamed the failure predicate to toolResultIsUnsuccessful and removed the now-unused fullRender bypass and constant-false arguments. No runtime or tool failure semantics changed.
- Ownership: owner=main; fork_run=none; revision=14cca43578d434822636aecb0ed41240194a85f6; scope=failed-tool preview rendering and virtual Ctrl+O regression proof; outcome=completed; commit=none

## Task workflow update - 2026-09-19T22:31:23+00:00
- Validation: Reviewer independently ran castor test --filter='failedOutputUsesBoundedPreviewAndCtrlOTogglesFullResult': PASS, 7 cases / 133 assertions. Confirmed testing skill and tests/AGENTS.md read and followed. Existing focused validation reused for unchanged executable diff.
- Summary: Independent reviewer APPROVE WITH SUGGESTIONS, no blockers. Accepted stale-comment correction in TranscriptBlockWidgetFactory; no executable changes after reviewed implementation. Other optional suggestions deferred to keep scope small.
- Review: role=reviewer; artifact=agent_7c920c9a3d23d453; target=uncommitted diff atop 14cca43578d434822636aecb0ed41240194a85f6; scope=failed-tool bounded previews, Ctrl+O, preserved failure semantics, specification fidelity and virtual regression proof; verdict=APPROVE WITH SUGGESTIONS; blockers=none.

## Task workflow update - 2026-09-19T22:31:44+00:00
- Summary: Committed reviewed implementation as 17ee45defbe1a0514d27f4bdbdae0f8ed0e7354a. Worktree clean. Review target finalized at this revision; sole post-review change corrects the stale comment identified by reviewer. No unresolved blockers. Ready for transition-owned full QA gate.

## Task workflow update - 2026-09-19T22:34:07+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it.md.
- Session/run: 60.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it/var/reports/qa-20260919-223158-39468-d89bfbfb.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-19T22:34:46+00:00
- Summary: Transition gate failed deterministically in BashToolTest::testCommandOutcomeRendersWithMatchingStatus, exit-42 dataset. Old assertion expects the exit-status heading in collapsed mode, but four-row bash tail correctly omits that first row. No push or PR occurred. Keep actual output/status assertions and expanded full-content proof; collapsed mode should assert the visible output tail and matching status color. QA reports: var/reports/qa-20260919-223158-39468-d89bfbfb.
- Ownership: owner=main; fork_run=none; revision=17ee45defbe1a0514d27f4bdbdae0f8ed0e7354a; scope=align existing BashToolTest render expectations with bounded failed previews; outcome=assigned; commit=none

## Task workflow update - 2026-09-19T22:38:08+00:00
- Validation: castor test --filter='testCommandOutcomeRendersWithMatchingStatus|failedOutputUsesBoundedPreviewAndCtrlOTogglesFullResult': PASS, 12 tests / 180 assertions, 1.989s.; Scoped castor cs-fix clean. Reviewer independently verified focused cases.
- Summary: Fixed stale BashToolTest expectation in 6d1cb1b375392cc840dbd01814cb92d53a081ec4. Retains actual execution/status, runtime projection, expanded full-output, and status-color proof. Removed obsolete collapsed full-output assertions; TuiCollapsedToolCardVirtualRenderTest owns bounded preview and real Ctrl+O proof, with mapping recorded in test comment. Resumed reviewer APPROVE, no blockers. Worktree clean.
- Ownership: owner=main; fork_run=none; revision=17ee45defbe1a0514d27f4bdbdae0f8ed0e7354a; scope=align existing BashToolTest render expectations with bounded failed previews; outcome=completed; commit=6d1cb1b375392cc840dbd01814cb92d53a081ec4
- Review: role=reviewer; artifact=agent_7c920c9a3d23d453; target=6d1cb1b375392cc840dbd01814cb92d53a081ec4; scope=gate-failure correction, specification fidelity, lower-layer proof mapping; verdict=APPROVE; blockers=none.

## Task workflow update - 2026-09-19T22:39:16+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Failed step: castor check (exit code 1).
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it.md.
- Session/run: 60.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it/var/reports/qa-20260919-223826-43962-6c2c12f6.
- Next: fix the failures, re-validate with focused Castor commands, then retry move_task(to="CODE-REVIEW").

## Task workflow update - 2026-09-19T22:40:26+00:00
- Validation: Failure log: var/reports/qa-20260919-223826-43962-6c2c12f6/check-test:tui.log. TmuxHarness.php:580 via TuiProviderErrorE2eTest.php:39. Session tui-provider-error-qa-20260919-223826-43962-6c2c12f6-44624-0, pane_pid=278170, local_untagged_leftovers empty.; castor clean:cleanup:workers:list: no stale QA worker candidates in this checkout.; Read-only follow-up: PID 278170 absent and tmux session no longer exists. This does not establish the underlying shutdown cause. Working tree clean.
- Summary: Second CODE-REVIEW gate blocked by TuiProviderErrorE2eTest teardown: owned tmux session still reported a live pane after Ctrl+D shutdown wait. No push or PR occurred; task remains IN-PROGRESS at reviewed commit 6d1cb1b375392cc840dbd01814cb92d53a081ec4. Failure is in existing provider-error terminal smoke shutdown, not failed-tool preview assertions. Did not retry the gate, increase timeouts, signal processes, or alter teardown safety.
- Blocker: diagnose provider-error E2E shutdown ownership/protocol before another CODE-REVIEW gate. No PR exists. Do not treat transient disappearance as a fix or retry until green.

## Task workflow update - 2026-09-19T22:43:32+00:00
- Summary: User authorized shutdown fix. Routing evidence: failing provider-error project log records controller EOF at 22:38:45.827 and run_control consumer shutdown escalation after shared grace at 22:38:50.843. Successful prior run has EOF without escalation. This points to controller/consumer lifecycle, not preview rendering. Delegate bounded lifecycle diagnosis and fix because it needs substantial runtime/process iteration; main retains task transitions and review.
- Ownership: owner=fork; fork_run=pending; revision=6d1cb1b375392cc840dbd01814cb92d53a081ec4; scope=diagnose and fix provider-error E2E Ctrl+D shutdown blocker with deterministic lifecycle proof; outcome=assigned; commit=none

## Task workflow update - 2026-09-19T22:59:26+00:00
- Recorded fork run: 99fa579e-ab6a-5cbe-b400-0ebd20e64cc3
- Validation: Focused provider-error and OM tests: 3 tests / 15 assertions PASS; individual times 3.873s, 3.065s, 3.142s.; castor test:tui two-worker ParaTest lane: PASS, 9 tests / 69 assertions, 18.9s. Slowest case 5.354s; changed cases 4.175s, 3.103s, 3.146s.; Scoped castor cs-fix clean. Main inspected exact diff: three added waitUntilPaneExits calls across two files. Fork confirmed testing skill and tests/AGENTS read/followed.
- Summary: Diagnosed incomplete E2E shutdown synchronization, not a proven production worker defect. Provider-error and two OM tests sent Ctrl+D then returned into two-second fail-closed teardown. Existing supported controller/client shutdown can take five/seven seconds. Added existing waitUntilPaneExits after Ctrl+D in those three journeys, matching other TUI tests. No timeout, production lifecycle, or safety changes. Fork also ruled out unacknowledged final Messenger work using a reverted isolated probe that waited for idle plus empty messenger table before quit.
- Ownership: owner=fork; fork_run=99fa579e-ab6a-5cbe-b400-0ebd20e64cc3; revision=6d1cb1b375392cc840dbd01814cb92d53a081ec4; scope=provider-error and newly evidenced OM E2E shutdown synchronization; outcome=completed; commit=none

## Task workflow update - 2026-09-19T23:02:06+00:00
- Summary: Reviewer APPROVE on shutdown synchronization fix. Committed dc0becd041951c7fe6e1c7a0e90708456c7899bf, worktree clean; no blockers before transition gate. Existing two additional unmodified TUI journeys lack the same explicit wait; reviewer marked this optional follow-up, outside evidenced blocker correction.
- Review: role=reviewer; artifact=agent_7c920c9a3d23d453; target=dc0becd041951c7fe6e1c7a0e90708456c7899bf; scope=positive E2E shutdown synchronization, fail-closed safety, specification fidelity; verdict=APPROVE; blockers=none.
- Ownership: owner=fork; fork_run=99fa579e-ab6a-5cbe-b400-0ebd20e64cc3; revision=6d1cb1b375392cc840dbd01814cb92d53a081ec4; scope=provider-error and OM E2E shutdown synchronization; outcome=completed; commit=dc0becd041951c7fe6e1c7a0e90708456c7899bf

## Task workflow update - 2026-09-19T23:03:13+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (50.4s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it/var/reports/qa-20260919-230223-60300-e52be8d4.
- Session/run: 60.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-19T23:03:14+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it/var/reports/qa-20260919-230223-60300-e52be8d4.
- Session/run: 60.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-19T23:03:17+00:00
- castor check passed (50.4s).
- Pushed task/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it to origin.
- Created PR: <url>
- Session/run: 60.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-19T23:03:17+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (50.4s).
- Pushed task/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/514

## Task workflow update - 2026-09-20T00:11:14+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/extensions/observational-memory/tests/Tui/TuiOmCommandsE2eTest.php |  2 ++
 src/Tui/Transcript/EditToolCallDiffRenderer.php                              |  1 -
 src/Tui/Transcript/SubagentResultRenderer.php                                |  1 -
 src/Tui/Transcript/TranscriptBlockWidgetFactory.php                          |  2 +-
 src/Tui/Transcript/TranscriptLinePreviewService.php                          |  5 -----
 src/Tui/Transcript/TranscriptToolPresentationPolicy.php                      |  6 +++---
 src/Tui/Transcript/TranscriptToolRenderer.php                                | 53 +++++++++++++++++++---------------------------------
 src/Tui/Transcript/TranscriptToolResultFacts.php                             | 11 +++++------
 src/Tui/Transcript/TranscriptToolResultPreviewWidget.php                     |  2 --
 src/Tui/Transcript/WriteToolCallContentRenderer.php                          |  1 -
 tests/CodingAgent/Tool/BashToolTest.php                                      | 43 +++++++++++++++++++++----------------------
 tests/Tui/E2E/TuiProviderErrorE2eTest.php                                    |  1 +
 tests/Tui/Screen/TuiCollapsedToolCardVirtualRenderTest.php                   | 88 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Transcript/TranscriptToolPresentationPolicyTest.php                |  6 +++---
 tests/Tui/Transcript/TranscriptToolResultFactsTest.php                       |  8 ++++----
 15 files changed, 147 insertions(+), 83 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-19-keep-failed-tool-output-collapsed-until-ctrl-o-expands-it.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: Verified PR #514 merged on GitHub at 2026-09-19T23:59:25Z, merge commit 2c0a7e9f7412c99931c63a1ff362236efde9e923. Proceeding with integration and required post-merge validation.

## Task workflow update - 2026-09-20T00:12:33+00:00
- Updated PR Status: merged
- Validation: Post-merge castor check PASS, 137.1s, all 11 lanes. Unit/integration 5123 tests / 22206 assertions; controller replay 11 / 175; TUI 9 / 69; llm-real 5 / 30. PHPStan, Deptrac, LSP, dead-code, formatting, docs and catalog checks passed.; QA process/tmux leak guard and cache guard passed. Reports: var/reports/qa-20260920-001121-364-379463d7.
- Summary: Post-merge validation complete at integration revision a3a5b620ebde1384f5772f9e14fcac6f7b2ed709. Git status clean. Task worktree removed and absent from git worktree list.
