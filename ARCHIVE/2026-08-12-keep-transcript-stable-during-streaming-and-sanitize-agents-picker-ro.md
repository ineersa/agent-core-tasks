# Keep transcript stable during model streaming with agents picker open

## Goal
Two related TUI defects in the fork/subagent agents picker:

1. When the `/agents` picker is open while the model is actively streaming, transcript rendering breaks/corrupts. Investigate overlay redraw, transcript projection, streamed delta application, and picker lifecycle rather than suppressing streaming.

2. Picker rows can become multiline because fork/subagent task summaries preserve embedded newlines. Observed rendering:

```text
→ fork [completed] agent_3c6b551435a85a82 run:f893b1c3-2c9… — Fork tool interactive test.

  Your task, in or... · 3% 33.0k/1000.0k deepseek-v4-flash
  fork [completed] agent_e0dfaa7b7e894d53 run:ca257ced-10a… — PING-PONG test: This is a test of the fork to... · 2% 23.7k/1000.0k deepseek-v4-flash
```

Picker row labels should be single-line. Normalize all line breaks and repeated whitespace at the shared row-label/summary formatting boundary before width truncation, for both fork and normal subagent entries.

Keep the fix scoped to picker/live-overlay rendering. Do not change persisted task prompts or child transcript content.

## Acceptance criteria
- Opening or keeping the agents picker visible while assistant/model output streams does not corrupt, erase, duplicate, or misplace transcript content.
- Streaming continues normally behind/alongside the picker; closing the picker reveals the complete correctly ordered transcript.
- Every fork and subagent picker entry renders on exactly one terminal row, even when the original task summary contains LF, CRLF, tabs, or repeated whitespace.
- Whitespace normalization occurs before width-aware truncation and applies through one shared picker-row formatting path.
- Selection/highlight and navigation remain aligned with the logical child entry after redraws and streaming updates.
- Automated TUI proof reproduces concurrent streaming with the picker open at the lowest correct layer; include minimal tmux proof only if the defect depends on real overlay/terminal behavior.
- Focused Castor validation passes, including the required TUI lane for the chosen proof layer, plus deptrac, PHPStan, and cs-check.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-12-keep-transcript-stable-during-streaming-and-sanitize-agents-picker-ro
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-12-keep-transcript-stable-during-streaming-and-sanitize-agents-picker-ro
Fork run: 0arbxkx4or6f
PR URL: https://github.com/ineersa/agent-core/pull/378
PR Status: merged
Started: 2026-08-13T13:43:46.200Z
Completed: 2026-08-13T15:58:28.381Z

## Work log
- Created: 2026-08-13T01:33:02.176Z

## Task workflow update - 2026-08-13T13:43:46.200Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-12-keep-transcript-stable-during-streaming-and-sanitize-agents-picker-ro.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-12-keep-transcript-stable-during-streaming-and-sanitize-agents-picker-ro.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-12-keep-transcript-stable-during-streaming-and-sanitize-agents-picker-ro.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-12-keep-transcript-stable-during-streaming-and-sanitize-agents-picker-ro.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-12-keep-transcript-stable-during-streaming-and-sanitize-agents-picker-ro.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-12-keep-transcript-stable-during-streaming-and-sanitize-agents-picker-ro.
- Summary: Started TUI task. Scope: preserve transcript correctness while model output streams with /agents picker open; normalize fork/subagent picker labels to one row before truncation; require replay-backed real TmuxHarness proof per user instruction.

## Task workflow update - 2026-08-13T13:52:32.763Z
- Summary: Scout traced root cause to multiline task summaries violating SelectListWidget's one-logical-row contract. Embedded physical lines desynchronize ScreenWriter row accounting when transcript deltas change geometry behind the picker; canonical transcript projection itself remains ordered. Minimal production fix is to reuse PickerListLabelFormatter::sanitizeTitle() in SubagentLivePickerController::buildItems() before existing summary truncation. Both fork and subagent rows share this catalog/buildItems path. Required proof will extend replay-backed TmuxHarness flow to keep /agents-live open across delayed assistant SSE chunks, verify single-row/highlight alignment, then close and assert complete ordered transcript.

## Task workflow update - 2026-08-13T14:21:58.781Z
- Recorded fork run: wk5boznvcz8q
- Validation: Failing-before proof PASS: `castor test --filter=buildItemsSanitizesMultilineTaskSummaryBeforeTruncation` failed without production sanitization because picker label contained LF.; Incomplete: TmuxHarness scenario reached open single-row picker with both fork/subagent entries but timed out waiting for first stream marker due stale `sequence.cursor` in overwritten test session fixture.
- Summary: Initial implementation fork hit its output/token ceiling before completing or committing. It left a dirty candidate implementation and established failing-before unit proof: without sanitization, buildItems emits LF and the regression fails. Mandatory TmuxHarness scenario was added but follow-up streaming stalled because the fixture helper overwrote events.jsonl after creating a live session while leaving the session's sequence.cursor at the previous sequence, so the follow-up never produced STREAM_MARK_A. A continuation fork will repair the test fixture lifecycle, minimize the candidate diff, complete real overlap proof, validate, and commit.

## Task workflow update - 2026-08-13T14:42:30.371Z
- Recorded fork run: 8ajoni7lk4h4
- Validation: Focused mandatory TmuxHarness test still fails: timed out waiting for STREAM_MARK_A after picker opened; 1 test, 12 assertions, 1 error.; Worktree remains uncommitted with candidate diff; production behavior is still the intended one-line sanitize-before-truncate change.
- Summary: Second continuation fork also hit its output/token ceiling before committing. It made the overwritten live session more canonical (tool result/message/batch/end events, synchronized sequence.cursor, removed stale state.json), but the resumed follow-up still never reached STREAM_MARK_A. Its final diagnosis/direction was to avoid mutating an already-open live session entirely: seed a completed session and launch the TUI directly with `agent --resume=<id>`, reusing the existing TuiRepairCommandE2eTest DB/session setup pattern.

## Task workflow update - 2026-08-13T15:07:03.471Z
- Recorded fork run: 2adpjkl27yyr
- Validation: Failing-before: focused picker regression failed without production sanitize because LF remained in the built picker label.; Focused picker: 1 test, 12 assertions PASS.; Focused replay-backed TmuxHarness: 1 test, 41 assertions PASS; STREAM_MARK_A through STREAM_MARK_FINAL arrived while picker remained open, Down moved the sole native highlight, and final markers were ordered after Escape.; castor test: 4414 tests, 16680 assertions PASS.; castor test:tui: 36 tests, 284 assertions PASS.; castor deptrac: 0 violations/errors.; castor phpstan: 0 errors.; castor cs-check: clean.
- Summary: Implementation completed and committed as 9c39aca96f856bad7b66d7c2a40739c7a504432c. Picker task summaries now use existing PickerListLabelFormatter::sanitizeTitle() before truncation, keeping fork/subagent entries to one physical row. Added deterministic test-only SSE chunk pacing and a preseeded completed --resume TmuxHarness scenario proving assistant markers continue while /agents-live remains open, native selection moves one logical row, and closing the picker reveals complete ordered non-duplicated output. Worktree is clean; 6 files changed.

## Task workflow update - 2026-08-13T15:31:58.318Z
- Validation: Reviewer verdict: REQUEST CHANGES (1 blocker).; Production picker-open/clear/redraw code changed: none.; Default non-paced replay SSE bytes remain equivalent after current refactor, but paced iterable is ineffective for incremental delivery.
- Summary: task-to-pr review returned REQUEST CHANGES. Confirmed production diff does not modify picker open/close/overlay clear/redraw behavior; it only sanitizes taskSummary before truncation. The observed full-screen blank flash is therefore not introduced by this branch and appears to be a separate/pre-existing picker-open redraw issue. Blocking review finding: test-only sse_chunk_delay_ms generator is synchronously prebuffered by Symfony MockResponse, so it delays the whole response rather than delivering chunks across TUI ticks; mandatory streaming-overlap proof must use a real incremental MockResponse callback/native mechanism or remove the ineffective seam.

## Task workflow update - 2026-08-13T15:44:47.974Z
- Recorded fork run: 0arbxkx4or6f
- Validation: Focused factory+picker unit tests: 4 tests, 24 assertions PASS.; Focused replay-backed TmuxHarness: 1 test, 43 assertions PASS.; castor test: 4415 tests, 16686 assertions PASS.; castor test:tui: 36 tests, 282 assertions PASS.; castor deptrac: 0 violations.; castor phpstan: 0 errors.; castor cs-check: clean.
- Summary: Addressed blocking review finding and committed as 1f1fb8f94. Replaced ineffective prebuffered-body sleeps with a test-only HttpClient stream decorator that introduces real wall-clock gaps between replay chunks. Strengthened TmuxHarness proof: early marker arrives while picker is open and final marker is absent; native highlight moves Down before later/final marker arrives; final marker arrives with picker still open; Escape reveals ordered unique transcript. Production remains only sanitize-before-truncate; blank picker-open flash remains separate/pre-existing.

## Task workflow update - 2026-08-13T15:51:55.322Z
- Validation: Final reviewer: APPROVED, no blocking findings.; Production picker-open clear/flash code changed: no.
- Summary: Final reviewer verdict APPROVED at HEAD 1f1fb8f94 with no blockers. Reviewer verified test-only stream decorator contract and real incremental Tmux proof, fixture/session coherence and isolation, unchanged ordinary replay path, and strict specification fidelity. Explicitly confirmed no production picker-open/clear/flash/redraw code changed; sole production change remains task-summary sanitization before truncation.

## Task workflow update - 2026-08-13T15:54:10.825Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 240s)...
- castor check passed (120.5s).
- Pushed task/2026-08-12-keep-transcript-stable-during-streaming-and-sanitize-agents-picker-ro to origin.
- branch 'task/2026-08-12-keep-transcript-stable-during-streaming-and-sanitize-agents-picker-ro' set up to track 'origin/task/2026-08-12-keep-transcript-stable-during-streaming-and-sanitize-agents-picker-ro'.
- Created PR: https://github.com/ineersa/agent-core/pull/378
- Validation: Focused unit: 4 tests, 24 assertions PASS.; Focused TmuxHarness: 1 test, 43 assertions PASS.; castor test: 4415 tests, 16686 assertions PASS.; castor test:tui: 36 tests, 282 assertions PASS.; castor deptrac: 0 violations.; castor phpstan: 0 errors.; castor cs-check: clean.; Reviewer: APPROVED.
- Summary: Implementation and review complete at 1f1fb8f94. Sole production behavior change normalizes picker task-summary whitespace before truncation. Deterministic TmuxHarness proves incremental streaming continues across picker-open interaction and final transcript remains ordered. Final reviewer APPROVED. User-observed blank-screen flash on picker open is not changed by this branch and remains separate/pre-existing.

## Task workflow update - 2026-08-13T15:58:28.381Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-08-12-keep-transcript-stable-during-streaming-and-sanitize-agents-picker-ro into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Picker/SubagentLivePickerController.php    |   2 +-
 .../Replay/ControllerReplayHttpClientFactory.php   |  62 ++++-
 .../ControllerReplayHttpClientFactoryTest.php      |  55 +++++
 .../E2E/Replay/StreamPacingHttpClient.php          |  63 +++++
 tests/Tui/E2E/TuiSubagentLiveViewE2eTest.php       | 260 ++++++++++++++++++++-
 .../fixtures/tui-agents-picker-stream-paced.json   |  41 ++++
 .../Picker/SubagentLivePickerControllerTest.php    |  38 +++
 .../Tui/Support/SubagentProgressEventsFixture.php  | 161 +++++++++++++
 8 files changed, 670 insertions(+), 12 deletions(-)
 create mode 100644 tests/CodingAgent/Runtime/Controller/E2E/Replay/StreamPacingHttpClient.php
 create mode 100644 tests/Tui/E2E/fixtures/tui-agents-picker-stream-paced.json
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-12-keep-transcript-stable-during-streaming-and-sanitize-agents-picker-ro.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-12-keep-transcript-stable-during-streaming-and-sanitize-agents-picker-ro.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #378 state: MERGED at 2026-08-13T15:58:09Z.; Pre-merge integration checkout clean on main.
- Summary: PR #378 merged on GitHub as d73641e493bef36b953549c570cf9a9e027a5a4a. Moving task to DONE and cleaning the task worktree.

## Task workflow update - 2026-08-13T16:00:58.229Z
- Validation: LLM_MODE=true castor check PASS: 4415 unit/integration tests (16686 assertions), 12 controller-replay tests (165 assertions), 36 TUI tests (288 assertions), 13 llm-real tests (144 assertions), deptrac 0 violations, PHPStan 0 errors, cs-check clean.; llama-proxy cache stable 244→244; QA artifact integrity and process/tmux leak checks clean.
- Summary: Post-merge integration validation passed on main after PR #378 merge. Task worktree removed and IDEA exclusions cleaned.

## Task workflow update - 2026-08-14T19:53:38+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
