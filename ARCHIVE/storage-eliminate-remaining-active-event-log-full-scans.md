# STORAGE- Eliminate remaining active event-log full scans

## Goal
Narrow follow-up to completed storage audit PR #430. Two confirmed active/recovery paths still scan complete canonical logs:

1. `InProcessAgentSessionClient::events()` calls `EventStoreInterface::allFor()` whenever `RuntimeEventEmitter` or TUI callbacks poll. The parent store's process-local cache avoids repeat decoding while a file is unchanged, but every canonical append invalidates the cache and causes another complete read/decode of accumulated history.
2. `AgentChildRunEventStore::readAfterSeq()` reads the child file from byte zero and filters `seq > cursor`. It is called by `DeferredSubagentBatchRecoveryService` after observed child turns and on run-control worker startup, so a near-tail cursor still rereads the entire child log.

Already resolved or owned elsewhere:
- Child `latestSequenceFor()` / `reverseFor()` now use lazy reverse-tail reads from PR #430.
- Canonical child payload duplication belongs exclusively to `STORAGE-optimize-child-canonical-event-payloads`.
- Event/state schema redesign, payload deletion, and retention are out of scope.

Desired invariant: active polling and cursor recovery read/decode only unseen canonical events. A deliberate initial attach/resume/history reconstruction may read historical events once; repair, inspection, export, and explicit history operations may still perform full replay.

## Acceptance criteria
- Map the exact call/lifecycle paths for `InProcessAgentSessionClient::events()` through `RuntimeEventEmitter` and TUI callbacks, including new start, passive attach/resume, follow-up after terminal state, history-position selection, cancellation, transient streaming, and controller shutdown. Do not assume one watermark policy fits all projection resets.
- Replace repeated `allFor()` in active in-process event polling with bounded unseen-event delivery using existing EventStore range/tail capabilities and the minimum per-run sequence watermark. Do not add a generic queue, cache layer, sidecar index, new protocol field, or new user setting.
- Preserve the ordering contract that transient stream deltas are delivered before the canonical completion/checkpoint events that finalize them. Preserve canonical newest sequence exactly once per observer lifetime without dropping follow-up events.
- On explicit attach/resume or another projection-reset boundary, allow one intentional canonical history reconstruction, establish the watermark from durable events, then switch to bounded unseen-event polling. Do not repeatedly replay history after the initial boundary.
- Handle `select_history_turn` / history-position changes explicitly: projector reset and selected-history reconstruction must remain correct even when ordinary active polling uses a watermark. Do not silently skip required replay or emit duplicate transcript blocks.
- Define observer/watermark lifetime and cleanup for new run, attach, `/new`, `/resume`, cancellation, terminal state with queued follow-up, controller reconnect, and shutdown. Do not release a watermark at a boundary that can cause a legitimate follow-up to be dropped.
- Rewrite child `readAfterSeq(cursor)` to avoid scanning from byte zero when the cursor is near the tail. Reuse the existing reverse-line JSONL primitive and durable append-order invariant where appropriate; collect unseen tail records and return them in chronological order expected by `DeferredSubagentBatchRecoveryService`.
- Do not use `sequence.cursor` as durable event-tail truth because allocation precedes append and can leave valid gaps. Correctness must derive from durable JSONL records and tolerate non-contiguous sequence numbers.
- Preserve child run-ID integrity, incompatible-schema handling, malformed JSON behavior within the inspected unseen range, missing/mismatched child behavior, and append-order semantics. Document that records older than the supplied recovery cursor are intentionally not decoded by a bounded tail read.
- Keep process-mode live-child behavior unchanged: one canonical snapshot on live-view entry followed by runtime-pipe events. Do not reintroduce canonical child polling.
- Add privacy-safe before/after evidence using deterministic fixture logs: physical bytes/chunks read where observable, decoded records, and returned events for small and large prefixes with near-tail cursors. Do not log paths, run IDs, prompts, tool output, or payloads.
- Add deterministic lowest-layer tests for: new run polling, passive attach/resume initial replay then deltas, repeated empty poll, canonical append, transient-before-canonical order, terminal follow-up, history selection/reset, sequence holes, trailing incompatible schema, malformed unseen tail, malformed prefix older than cursor, missing/mismatched child, worker-start recovery, and child-turn recovery. No arbitrary sleeps, timing windows, retries-until-green, test-only production APIs, or cases over 10 seconds.
- Update `.pi/reports/session-storage-file-io-audit.md` and `docs/session-storage.md` with the remaining-read fixes, watermark lifecycle, and bounded child recovery semantics. Do not modify canonical payload ownership or storage-retention sections beyond cross-references.
- Use existing `EventStoreInterface`, `JsonlRunEventLog`, runtime event mapper/sink, and session lifecycle seams. No new abstraction unless an unavoidable architectural boundary is demonstrated and approved by the user first.
- Because this touches `AgentSessionClient`, runtime polling, event ordering, and deferred recovery, run focused Castor tests, `castor deptrac`, `castor phpstan`, `castor cs-check`, controller replay, and full `castor check` before CODE-REVIEW. Inspect JUnit and remediate any individual case over 10 seconds.

## Workflow metadata
Status: ARCHIVE
Branch: task/storage-eliminate-remaining-active-event-log-full-scans
Worktree: /home/ineersa/projects/agent-core-worktrees/storage-eliminate-remaining-active-event-log-full-scans
Fork run: eyoc1s81ehji
PR URL: https://github.com/ineersa/agent-core/pull/431
PR Status: merged
Started: 2026-08-25T20:36:47.472Z
Completed: 2026-08-25T22:44:44.214Z

## Work log
- Created: 2026-08-25T20:36:18.397Z

## Task workflow update - 2026-08-25T20:36:47.472Z
- Moved TODO → IN-PROGRESS.
- Created branch task/storage-eliminate-remaining-active-event-log-full-scans.
- Created worktree /home/ineersa/projects/agent-core-worktrees/storage-eliminate-remaining-active-event-log-full-scans.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/storage-eliminate-remaining-active-event-log-full-scans.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/storage-eliminate-remaining-active-event-log-full-scans.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/storage-eliminate-remaining-active-event-log-full-scans.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/storage-eliminate-remaining-active-event-log-full-scans.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/storage-eliminate-remaining-active-event-log-full-scans/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/storage-eliminate-remaining-active-event-log-full-scans.
- Summary: User approved implementation. Scope remains exactly two bounded-read changes: in-process active event polling must stop replaying allFor() after initial/reset boundaries, and child readAfterSeq() must read unseen durable tail records without scanning from byte zero. No payload/schema/retention work.

## Task workflow update - 2026-08-25T20:40:43.816Z
- Summary: Read-only scout completed. Correct minimal design avoids a client-owned watermark because that could advance before RuntimeEventEmitter stdout delivery succeeds and lose retries. Instead extend the internal AgentSessionClient::events contract with an `afterSeq` argument supplied by existing owning cursors: RuntimeEventEmitter runEventCursors, RuntimeEventPoller state.lastSeq, and SubagentLiveChildViewPoller childLastSeq. InProcess client drains transients first, reverse-reads durable events until seq <= afterSeq, reverses the unseen list, and maps chronologically. It remains stateless, preserving multiple-observer and retry semantics. JsonlProcess client accepts the cursor without protocol changes. Child readAfterSeq uses the same reverse-tail/stop/reverse pattern. No new storage index, sidecar, queue, cache, setting, or protocol field.

## Task workflow update - 2026-08-25T20:41:12.154Z
- Recorded fork run: hv6oqredft3k
- Summary: Implementation fork hv6oqredft3k launched with finalized observer-owned afterSeq design and bounded reverse-tail child recovery. Fork will implement, test, document, validate through focused Castor lanes, and commit; no full check or PR transition in this phase.

## Task workflow update - 2026-08-25T20:59:17.996Z
- Recorded fork run: hv6oqredft3k
- Validation: Focused Castor tests: PASS — 74 tests, 389 assertions in 4.7s (InProcess events, child store, deferred recovery, emitter, TUI pollers/callbacks).; castor test:controller-replay: PASS — 6 tests, 92 assertions in 21.0s.; castor deptrac: PASS — 0 violations/errors.; castor phpstan: PASS — 0 errors.; castor cs-check: PASS — 0 files fixed.; castor docs:validate: PASS — 16 built-in documents.; git diff --check: PASS.; Commit 8d50fd0709b9805ab374ab8e4d5a76484a051fcc verified; worktree clean.
- Summary: Implementation complete at commit 8d50fd070 (`perf(storage): bound active event log reads`). Observer-owned `afterSeq` now flows through AgentSessionClient polling from RuntimeEventEmitter/TUI cursors; InProcessAgentSessionClient reverse-reads and chronologically maps only unseen canonical events after draining transients. RuntimeEventEmitter advances cursors only after successful stdout writes, preserving retry safety. Child readAfterSeq now reverse-reads only the unseen durable tail and returns chronological events. No local cursor cache, index, sidecar, queue, setting, protocol field, payload/schema, or retention change. 24 touched files are mostly the internal interface's mechanical test doubles; production behavior is a small bounded-read patch. Worktree is clean and task remains IN-PROGRESS pending user-requested review/PR phase.

## Task workflow update - 2026-08-25T21:18:08.353Z
- Validation: Reviewer read root/task workflow/testing/tests instructions and completed read-only review.; Reviewer found no new setting/protocol/storage/schema/retention surface and no correctness/security blocker.; Reviewer confirmed runtime observer cursor/retry/history/transient ordering and child integrity semantics are correct.; Reviewer REQUEST CHANGES only for missing large-prefix quantified evidence required by task acceptance criteria.
- Summary: Task-to-PR reviewer verdict: REQUEST CHANGES with one specification blocker, no correctness/security blockers. Missing finalized acceptance evidence: deterministic large-prefix near-tail fixtures and quantified before/after read/decode evidence for both in-process polling and child readAfterSeq. Required fix is test-only measurement at lowest layer plus narrow audit note; reviewer also requested the child method docblock state that prefixes older than the cursor boundary are intentionally not decoded. Implementation/API design itself was approved as correct/minimal; branch's two preceding merge commits are tree-neutral and not a blocker.

## Task workflow update - 2026-08-25T22:01:38.148Z
- Recorded fork run: p75o5az3j47h
- Summary: Reviewer-fix fork aborted before commit after attempting an unacceptable broad cleanup command. Forensic inspection confirms the exact attempted command was `cd <task-worktree> && rm -rf home` — a relative worktree-local `home/`, not `/home` or the user's actual home directory. The command was blocked/aborted and did not run. No fork process remains. Worktree currently has the four expected reviewer-fix files modified plus an untracked 40 KiB `home/ineersa/.../var/reports/reviewer-large-prefix` shadow report tree created by the fork's absolute HATFIELD_QA_REPORTS_DIR invocation. No cleanup will be performed without explicit user approval; task remains IN-PROGRESS.

## Task workflow update - 2026-08-25T22:04:27.273Z
- Summary: User removed the generated worktree-local `home/` shadow directory. Verified it is gone; worktree now contains only the four expected uncommitted reviewer-fix files. Root cause of shadow directory is confirmed: testing skill instructed an absolute `HATFIELD_QA_REPORTS_DIR`, while `.castor/helpers.php::reports_dir()` strips the leading slash and prefixes `project_root_dir()`, turning `/home/...` into `<worktree>/home/...`. The fork's attempted broad `rm -rf home` response was unacceptable and remains aborted.

## Task workflow update - 2026-08-25T22:07:38.686Z
- Summary: User explicitly added task-to-PR corrective scope after aborted fork: fix `.castor/helpers.php::reports_dir()` so absolute `HATFIELD_QA_REPORTS_DIR` values remain absolute instead of being stripped and nested under the worktree; preserve project-relative behavior for relative values; add deterministic regression proof. Then finish/revalidate the four inherited reviewer-evidence edits and relaunch implementation fork. Safety constraint: fork must issue no filesystem cleanup/deletion commands (`rm`, `rmdir`, unlink, find -delete, git clean, or equivalents); leave any artifact and report it rather than deleting.

## Task workflow update - 2026-08-25T22:13:56.822Z
- Recorded fork run: eyoc1s81ehji
- Validation: Focused Castor tests: PASS — 33 tests, 127 assertions in 3.0s.; Full castor test: PASS — 4,853 tests, 19,792 assertions in 30.0s.; New case timings: 0.005226s, 0.013982s, 0.000707s, 0.000702s; full lane max 3.600607s.; castor phpstan: PASS — 0 errors.; castor cs-check: PASS — 0 files fixed.; castor docs:validate: PASS — 16 documents.; git diff --check: PASS.; No deletion/cleanup command used; worktree clean and no untracked artifacts.
- Summary: Replacement fork completed safely with two focused commits: 8ae4f0456 (`test(storage): quantify bounded event tail reads`) and ee430a429 (`fix(castor): preserve absolute report paths`). Large-prefix reviewer evidence now quantifies 2,000-record near-tail reads and one 8 KiB child tail chunk; Castor absolute report paths now remain absolute while relative paths remain project-rooted. Fork explicitly used no cleanup/deletion commands and reports clean worktree/no untracked artifacts.

## Task workflow update - 2026-08-25T22:21:57.920Z
- Validation: Delta reviewer: APPROVED; testing skill/tests instructions read; no cleanup/deletion commands used.; castor test: PASS — 4,853 tests, 19,790 assertions in 30.4s.; castor test:controller-replay: PASS — 6 tests, 92 assertions in 20.3s.; castor test:tui: PASS — 8 tests, 59 assertions in 20.3s.; castor deptrac: PASS — 0 violations/errors.; castor phpstan: PASS — 0 errors.; castor cs-check: PASS — 0 files fixed.; castor docs:validate: PASS — 16 documents.; JUnit timing audit: unit 4,853 cases max 4.471541s; controller 6 max 3.874431s; TUI 8 max 4.416420s; zero cases >10s.; git diff --check and git status: PASS/clean.
- Summary: Delta reviewer APPROVED with no blockers. Prior evidence blocker is closed: deterministic 2,000-record fixtures prove suffix+boundary decoding and one 8 KiB child reverse chunk. Reviewer also approved user-authorized Castor report-path correction as exact/minimal; absolute custom paths remain absolute, relative/default behavior remains unchanged. Original runtime cursor/retry/history/transient and child integrity semantics remain approved. Two tree-neutral ancestry merges are nonblocking.

## Task workflow update - 2026-08-25T22:23:18.005Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 600s)...
- castor check passed (66.4s).
- Pushed task/storage-eliminate-remaining-active-event-log-full-scans to origin.
- branch 'task/storage-eliminate-remaining-active-event-log-full-scans' set up to track 'origin/task/storage-eliminate-remaining-active-event-log-full-scans'.
- Created PR: https://github.com/ineersa/agent-core/pull/431

## Task workflow update - 2026-08-25T22:23:38.500Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/431
- Updated PR Status: open
- Validation: Automated move_task castor check: PASS in 66.4s.; Full-gate JUnit: 4,872 cases, zero >10s; max 6.521554s (TuiResumeSessionSwitchE2eTest).; PR #431 open: https://github.com/ineersa/agent-core/pull/431; Worktree git status clean; task branch divergence from origin 0/0.
- Summary: Task-to-PR complete. Branch pushed and PR #431 opened after reviewer approval and deterministic CODE-REVIEW gate. Worktree is clean and synchronized with origin.

## Task workflow update - 2026-08-25T22:44:44.214Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/storage-eliminate-remaining-active-event-log-full-scans: ide_close_project returned isError.
- Merged task/storage-eliminate-remaining-active-event-log-full-scans into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/storage-eliminate-remaining-active-event-log-full-scans.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/storage-eliminate-remaining-active-event-log-full-scans.
- Pulled integration checkout: Already up to date..
- Validation: GitHub PR #431 state: MERGED at 2026-08-25T22:27:56Z.; Integration main clean; origin/main divergence 0/0 before transition.
- Summary: PR #431 confirmed merged at 66edd8464. Integration main was clean and synchronized with origin before DONE transition.

## Task workflow update - 2026-08-25T22:46:27.536Z
- Updated PR Status: merged
- Validation: Post-merge `LLM_MODE=true castor check`: PASS in 145.3s; all 9 lanes green.; Post-merge JUnit: 4,872 cases, zero >10s; max 5.996136s.; QA artifact integrity: PASS; exact-run leak check: PASS; llama-proxy cache guard stable 391→391.; Integration git status clean; origin/main divergence 0/0.; Task worktree removal verified.
- Summary: Post-merge completion verified. PR #431 merged, task branch integrated, task worktree removed, integration main clean and synchronized with origin. JetBrains close reported a degraded isError during cleanup, but worktree and IDEA exclusions were removed successfully.

## Task workflow update - 2026-08-29T16:09:39.044Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
