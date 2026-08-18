# Render direct shell commands as complete colored bash transcript cards

## Goal
Investigation found that the broken `● bash` entry is an orphan `ToolResult`, not a fork-specific renderer. Direct `!command` execution bypasses the normal streamed tool-call lifecycle: `ApplyShellCommandHandler` creates `ExecuteShellToolCall`, and `ExecuteShellToolCallWorker` emits execution start/end events without preserving the command as tool-call arguments. `ToolProjectionSubscriber` therefore creates only a result block. `TranscriptVisualProjector` cannot pair it with a `ToolCall`, so `TranscriptBlockWidgetFactory` uses the standalone generic result renderer, producing `● bash` plus output with no command block and without the normal exchange-card styling.

Fix the shared direct-shell event/projection path so direct shell executions reuse the existing complete bash exchange renderer. Do not add a fork/subagent-specific rendering path.

## Acceptance criteria
- Direct `!command` execution renders as the existing complete, colored bash exchange card rather than an output-only generic result.
- The card shows the executed command as a `command:` argument; include `timeout:` only when that value actually exists.
- Normal LLM-generated bash tool cards remain unchanged.
- Replay/resume and fork/subagent child live view render the same complete direct-shell card.
- Projection does not create a duplicate `ToolCall` when a normal matching call block already exists.
- Add the smallest regression proof at the lowest appropriate runtime/TUI layer, following the repository testing conventions.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-06-render-direct-shell-commands-as-complete-colored-bash-transcript-card
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-06-render-direct-shell-commands-as-complete-colored-bash-transcript-card
Fork run: o46wb7tfj1mg
PR URL: https://github.com/ineersa/agent-core/pull/368
PR Status: merged
Started: 2026-08-06T23:08:10.924Z
Completed: 2026-08-07T02:55:33.712Z

## Work log
- Created: 2026-08-06T22:09:41.408Z

## Task workflow update - 2026-08-06T23:08:10.924Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-06-render-direct-shell-commands-as-complete-colored-bash-transcript-card.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-06-render-direct-shell-commands-as-complete-colored-bash-transcript-card.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-06-render-direct-shell-commands-as-complete-colored-bash-transcript-card.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-06-render-direct-shell-commands-as-complete-colored-bash-transcript-card.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-06-render-direct-shell-commands-as-complete-colored-bash-transcript-card.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-06-render-direct-shell-commands-as-complete-colored-bash-transcript-card.
- Summary: Task started. Loaded task-workflow and testing guidance plus tests/AGENTS.md before TUI/runtime test planning. User explicitly requires a replay-backed real TmuxHarness proof for this task, in addition to the lowest-layer focused regression.

## Task workflow update - 2026-08-06T23:16:54.065Z
- Recorded fork run: o46wb7tfj1mg
- Summary: Implementation fork launched in `/home/ineersa/projects/agent-core-worktrees/2026-08-06-render-direct-shell-commands-as-complete-colored-bash-transcript-card`. Scope includes shared direct-shell event/projection fix, duplicate guard, child replay proof, and the user-required replay-backed TmuxHarness card/color proof.
- Fresh-context scout completed read-only reconnaissance and confirmed it read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`; IDE tools were unavailable, so it used targeted fallback. Implementation delegated to fork run `o46wb7tfj1mg`.

## Task workflow update - 2026-08-06T23:23:25.410Z
- Recorded fork run: o46wb7tfj1mg
- Validation: PASS: `castor test --filter='ExecuteShellToolCallWorkerTest\|RuntimeEventMapperTest::testNormalizesToolExecutionStart\|TranscriptProjectorTest::testToolExecutionStartedWithArguments\|ChildRunTranscriptSnapshotProviderTest'` — 9 tests, 77 assertions.; PASS: `castor test:tui --filter=TuiJourneyE2eTest` — 1 test, 15 assertions; real direct `!ls -1` card text and ANSI styling asserted.; PASS: `castor deptrac` — 0 violations.; PASS: focused `castor phpstan` on 3 changed production files — 0 errors.; PASS: `castor cs-check` after `castor cs-fix`.; FAIL (reported unrelated/pre-existing): `castor test:controller-replay` — `ChildExtensionSelectionControllerReplayTest` received `command.rejected` and no `run.started`; no touched code in that path. Full `castor check` intentionally not run during task-start.
- Summary: Implementation completed and committed as `bcac06726e67da8094adacd28693a63380b6ec61` (`fix direct !shell as complete colored bash exchange card`). Verified worktree is clean and commit changes exactly 8 expected files: 3 shared production event/projection files plus focused worker/mapper/projector/child replay tests and the required real replay-backed `TmuxHarness` journey proof. Direct shell start events now carry command arguments; translation preserves them; projection creates one missing finalized ToolCall before the result while guarding existing normal LLM calls.

## Task workflow update - 2026-08-07T00:04:04.682Z
- Validation: REVIEW: APPROVED WITH SUGGESTIONS; no blockers; specification fidelity and required TmuxHarness proof confirmed.; PASS: `castor test` — 4396 tests, 16471 assertions.; PASS: `castor test:tui` — 34 tests, 226 assertions.; PASS: `castor deptrac` — 0 violations/errors.; PASS: `castor phpstan` — 0 errors.; PASS: `castor cs-check` — no files require fixes.; FAIL: `castor test:controller-replay` in task worktree — `ChildExtensionSelectionControllerReplayTest`, `command.rejected`, missing `run.started`.; BASELINE CONFIRMATION: same `castor test:controller-replay` failure reproduced on main at `f0116fb48`; therefore unrelated/pre-existing. `move_task(to=CODE-REVIEW)` not attempted because its mandatory `castor check` gate would deterministically fail.
- Summary: task-to-pr review completed: reviewer APPROVED WITH SUGGESTIONS, with no blocking correctness, security, architecture, specification-fidelity, or test-proof findings. Verified the real replay-backed TmuxHarness journey enters `!ls -1` and asserts complete card text plus command-key ANSI styling. Worktree remains clean at `bcac06726e67da8094adacd28693a63380b6ec61`. PR transition is blocked by a deterministic controller-replay failure reproduced unchanged on both the task worktree and current main: `ChildExtensionSelectionControllerReplayTest` receives `command.ack, command.rejected` and no `run.started`. This is outside the 8-file task diff; do not broaden this implementation without explicit authorization.

## Task workflow update - 2026-08-07T00:25:01.176Z
- Validation: PASS after main sync: `castor test:controller-replay` — 12 tests, 165 assertions. Previously failing child-extension replay now passes.
- Summary: Baseline gate blocker fixed directly on main as authorized: commit `b84e4d6ff` pushed to `origin/main`. Task branch merged updated main at `c48159bd1`; `origin/main...HEAD` remains exactly the original 8-file direct-shell diff, and the worktree is clean.

## Task workflow update - 2026-08-07T00:28:51.378Z
- Validation: PASS: `castor test:llm-real` — 13 tests, 144 assertions.; PASS warm-cache confirmation: second `castor test:llm-real` — 13 tests, 144 assertions; proxy entries remained exactly 243 before and after.
- Summary: First CODE-REVIEW transition gate completed code/test lanes but failed the deterministic llama-proxy cache-growth guard (225 → 241 entries). Warmed the live lane intentionally and confirmed stable cache before retry.

## Task workflow update - 2026-08-07T00:31:13.341Z
- Validation: Gate cache guard correctly failed on one new entry; no code/test failure reported.
- Summary: Second deterministic gate found one check-mode-only cold proxy cassette (243 → 244) despite the standalone live lane being stable. The exact check request is now cached; retrying without disabling the guard or using stress overrides.

## Task workflow update - 2026-08-07T00:33:45.556Z
- Validation: PASS focused rerun: `castor test --filter=TuiTranscriptBlocksVirtualRenderTest::testUserMessageRendersMarkdownAndInlineHtmlAsCode` — 1 test, 4 assertions.
- Summary: Third gate attempt passed non-unit lanes but hit one unrelated transient virtual-render assertion in `TuiTranscriptBlocksVirtualRenderTest::testUserMessageRendersMarkdownAndInlineHtmlAsCode`. The exact test passes immediately in focused Castor rerun; no task-touched file participates in that assertion.

## Task workflow update - 2026-08-07T00:35:52.515Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (113.9s).
- Pushed task/2026-08-06-render-direct-shell-commands-as-complete-colored-bash-transcript-card to origin.
- branch 'task/2026-08-06-render-direct-shell-commands-as-complete-colored-bash-transcript-card' set up to track 'origin/task/2026-08-06-render-direct-shell-commands-as-complete-colored-bash-transcript-card'.
- Created PR: https://github.com/ineersa/agent-core/pull/368
- Validation: Reviewer APPROVED; no blockers.; `castor test`, `castor test:tui`, `castor test:controller-replay`, `castor test:llm-real`, `castor deptrac`, `castor phpstan`, and `castor cs-check` passed.; Transient unrelated virtual-render gate assertion passed exact focused rerun.; Deterministic proxy safeguards remain enabled.
- Summary: Reviewer approved with no blockers. Direct `!command` renders through the existing complete colored bash exchange across main/replay/child paths, with duplicate protection. Full focused validation passes; prior gate-only proxy misses are warmed and an unrelated transient virtual-render assertion passed its focused rerun.

## Task workflow update - 2026-08-07T01:50:01.041Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User clarified the original defect is not TUI `!command`: normal bash tool calls are missing arguments/styling only while watching a fork/subagent live. After resuming the same failed fork, all bash cards render correctly. PR #368 therefore fixes the wrong path and must be revised from actual live child artifacts/events before review resumes.

## Task workflow update - 2026-08-07T02:49:28.315Z
- Summary: User explicitly approved keeping PR #368 as the direct `!command` fix and will merge it separately. The distinct intermittent live fork/subagent bash-card defect is split into a new task; no changes to PR #368 are requested.

## Task workflow update - 2026-08-07T02:51:53.709Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/368
- Updated PR Status: merged
- Summary: PR #368 merged on GitHub at 2026-08-07T02:49:23Z as merge commit `969edd7c7e69da5f0b4cb3004d9c12a91d4e3dbe`. Status transition raced the user merge and its PR-create step correctly found no remaining commits.

## Task workflow update - 2026-08-07T02:55:33.712Z
- Moved IN-PROGRESS → DONE.
- Merged task/2026-08-06-render-direct-shell-commands-as-complete-colored-bash-transcript-card into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-06-render-direct-shell-commands-as-complete-colored-bash-transcript-card.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-06-render-direct-shell-commands-as-complete-colored-bash-transcript-card.
- Pulled integration checkout: Already up to date..
- Validation: PR #368 merged at `969edd7c7e69da5f0b4cb3004d9c12a91d4e3dbe`.; Deterministic `castor check` passed before PR creation/merge.
- Summary: User merged PR #368. Task is complete as the approved direct `!command` card fix; the distinct live child buffering race is now tracked and in progress as `2026-08-07-preserve-complete-bash-cards-in-live-child-event-view`.

## Task workflow update - 2026-08-14T19:53:34+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
