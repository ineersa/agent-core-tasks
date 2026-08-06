# Spike: replace turn tree with linear history rewind/forward

## Goal
Investigate replacing Hatfield's exposed `/tree` branching model with a simpler single active linear history and cursor-based rewind/forward navigation.

Context:
- `/tree` currently works, but turn ancestry, command-to-child-turn mapping, branch-filtered replay, rewind command suppression, and compaction checkpoints make the model difficult to reason about.
- The desired everyday workflow is linear history with the ability to move backward and forward.
- `events.jsonl` should remain append-only for recovery and audit; physically truncating canonical events is not preferred.
- Candidate semantics: back/forward moves an active cursor without executing; submitting new input behind the tip explicitly abandons the forward timeline and appends a new timeline generation/reset marker. The old future remains audit-only rather than an exposed selectable sibling branch.
- Observational memory is deliberately session-global for MVP and must remain independent of this spike.

This is an architecture/design spike only. Do not implement the replacement until the proposed user semantics and migration boundary are approved.

## Acceptance criteria
- Map the current `/tree`, rewind, turn-tree projection, replay filtering, command seeding, resume/fork, compaction, runtime protocol, and TUI dependencies with concrete file/symbol references.
- Verify how ordinary and hook-provided `context_compacted` events behave across rewind and branch selection, including the shared-ancestor checkpoint edge case and current test coverage gaps.
- Specify exact linear navigation semantics for back, forward, cursor-at-tip, submitting input behind the tip, abandoning forward history, and whether abandoned history remains inspectable outside normal navigation.
- Propose an append-only event/projection model for one active timeline, including cursor and timeline generation/reset events, stable source identity, replay rules, crash recovery, and state persistence.
- Define behavior for in-flight LLM steps, tool calls, shell-only activity, queued commands, HITL requests, failed/cancelled turns, manual/automatic compaction, resume, and fork.
- Compare retaining the current tree, hiding tree UX over existing internals, and fully replacing turn-tree internals; recommend the smallest coherent option and explain its complexity/risk.
- Identify affected public/runtime/TUI contracts and a migration/removal strategy that avoids compatibility shims unless a published surface requires them.
- Provide a focused implementation task breakdown and test thesis at the lowest correct layers, without implementing production changes during the spike.
- State explicitly that OM remains session-global and non-branch-aware regardless of the selected history model.

## Workflow metadata
Status: IN-PROGRESS
Branch: task/spike-linear-history-rewind
Worktree: /home/ineersa/projects/agent-core-worktrees/spike-linear-history-rewind
Fork run: 2pwh21cl670l
PR URL: https://github.com/ineersa/agent-core/pull/366
PR Status: open
Started: 2026-08-06T01:25:06.640Z
Completed:

## Work log
- Created: 2026-07-21T17:04:24.553Z

## Task workflow update - 2026-08-06T01:24:59.875Z
- Summary: User approved implementation; the original architecture-only spike scope is superseded by the finalized linear undo/redo history requirements below. Implement the smallest clean replacement with no tree compatibility layer.
- Approved semantics (2026-08-05): replace exposed turn-tree branching with linear undo/redo history. `/history` replaces `/tree` and shows only user prompts (plus the minimal start/current indication needed by the picker); assistant messages, tool calls, shell events, and compaction events are internal context, not selectable rows.
- Selecting user prompt N is non-destructive: position conversation context immediately before prompt N and populate the editor with prompt N's text. Existing forward history remains available until a context-affecting action is submitted. Selecting another retained user prompt provides both backward and forward navigation; do not add separate speculative navigation UX.
- Any context-affecting action while positioned behind the tip abandons the forward tail first. This includes a new/edited user message, shell command, manual compaction, and other commands that mutate conversation context. The selected prompt and everything after it are abandoned when its edited replacement is submitted.
- Keep canonical `events.jsonl` append-only. Do not physically rewrite/truncate it. Append one minimal tail-discard/reset marker referencing the retained turn boundary; normal state, hot prompt, transcript, replay, resume, and `/history` projection must completely ignore discarded turns. Discarded bytes remain audit-only.
- Reuse existing stable `turnNo` and current selected turn/leaf semantics. Do not introduce generation IDs, a new cursor subsystem, compatibility readers, dual tree/linear paths, or speculative storage/configuration. Remove parent/child ancestry, sibling selection, branch rendering, command-to-child inference, and branch-specific replay logic where no longer needed.
- Compaction must operate only on the active retained history. A context-affecting compaction behind the tip first abandons the forward tail, eliminating shared-ancestor checkpoint behavior. Preserve ordinary and hook-provided compaction semantics on the retained linear history.
- Keep existing in-flight/cancellation behavior unless a minimal change is required by removal of tree internals; do not add new navigation policy or UX without explicit approval. Resume rebuilds the active retained timeline; fork uses the currently selected retained context. OM remains session-global and non-history-aware.
- Hatfield `.hatfield/extensions/file-rewind` remains a separate file-restoration feature. Do not combine `/rewind` with conversational `/history`; that is another ticket. Preserve its hidden-Git backend and `runId + turnNo` checkpoints. Adapt only the minimum host row/projection integration needed for flat active user-turn history. Do not touch `my-pi`.
- Active-development cutover: follow repository policy—replace old tree behavior and update tests/docs without backward-compatibility shims or dual-format support. Existing tree-format sessions need not remain navigable through a compatibility path.
- Validation requirements: read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`; state test theses before changes. Add minimal focused proof for linear replay/tail discard and user-visible `/history` selection/editor population at the lowest correct layers, plus controller replay for runtime protocol changes where needed. Run focused Castor commands only; implementation handoff must include results. Full `castor check` is deferred to task-to-PR per workflow.

## Task workflow update - 2026-08-06T01:25:06.640Z
- Moved TODO → IN-PROGRESS.
- Created branch task/spike-linear-history-rewind.
- Created worktree /home/ineersa/projects/agent-core-worktrees/spike-linear-history-rewind.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/spike-linear-history-rewind.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/spike-linear-history-rewind.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/spike-linear-history-rewind.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/spike-linear-history-rewind.
- Summary: Finalized scope: clean replacement of tree branching with append-only linear undo/redo history and `/history` user-prompt picker; no compatibility shim or file-rewind combination.

## Task workflow update - 2026-08-06T01:26:27.252Z
- Recorded fork run: ueyxgp1ohxnd
- Implementation fork `ueyxgp1ohxnd` launched in `/home/ineersa/projects/agent-core-worktrees/spike-linear-history-rewind` with finalized linear-history semantics, minimal-design constraints, focused test theses, Castor-only QA, and explicit prohibition on PR/push/status transition.

## Task workflow update - 2026-08-06T01:52:00.068Z
- Recorded fork run: n2pm01itczj9
- Fork `ueyxgp1ohxnd` ended prematurely without commit, leaving a dirty implementation; its artifact only reported an alleged unrelated skill-card display flake and beginning deptrac, with no usable validation handoff. Continuation fork `n2pm01itczj9` launched to audit/finish the existing diff, verify failures, remove stale tree complexity, run focused Castor validation, and commit.

## Task workflow update - 2026-08-06T02:02:23.474Z
- Recorded fork run: tvdc535mdsl3
- Verified commit `59c0df80adc71996ce52577226cea2d425e4793e` is clean with 50 files, +1231/-3036 (net -1805). Found one specification violation before accepting handoff: explicit deprecated `turn_branched` compatibility handling/tests/docs remained despite no-compatibility requirement. Narrow cleanup fork `tvdc535mdsl3` launched to remove only that legacy path and commit.

## Task workflow update - 2026-08-06T02:04:35.542Z
- Recorded fork run: u5zga7ejrq5k
- Verified cleanup commit `1ffb229dc8c3d3ad50dcd570184912ea2c084579`; branch clean, cumulative diff 51 files +1231/-3061 (net -1830), no `turn_branched` references. Final scan found one false `.hatfield/extensions/file-rewind/README.md` statement that `/tree` remains available plus two stale replacement comments. Tiny docs-only fork `u5zga7ejrq5k` launched to remove all remaining `/tree` mentions and commit.

## Task workflow update - 2026-08-06T02:05:34.783Z
- Recorded fork run: u5zga7ejrq5k
- Validation: Focused history/TUI/runtime filters: OK — 131 tests, 729 assertions.; castor test:controller-replay: OK — 12 tests, 165 assertions.; castor deptrac: OK — 0 violations.; castor phpstan: OK — 0 errors.; castor cs-check: OK — clean.; Legacy cleanup focused filters: OK — 97 tests, 399 assertions.; Full castor test: one path-wrap failure in unchanged TuiSkillReadCardVirtualRenderTest under the long task worktree path; identical test passes on integration main. Exact cause recorded by fork: expected contiguous `docs/unrelated`, VirtualTerminal wrapped long path as `docs/un` + newline + `related/SKILL.md`. Full castor check intentionally deferred to task-to-PR.
- Summary: Implementation complete and committed through `7db7840ba1a6417d58e9ef29a2c9aa30d33f4349`. Replaced exposed branching with append-only linear undo/redo history: `/history` lists user prompts, selection positions before the prompt and fills the editor, context mutation behind the tip appends `history_tail_discarded`, and discarded events disappear from active replay/state/prompt/transcript/resume while remaining audit bytes. Removed `turn_branched` compatibility and all `/tree` references. File `/rewind` remains separate. Cumulative diff: 52 files, +1232/-3062, net -1830 LOC. Worktree is clean.

## Task workflow update - 2026-08-06T02:48:37.688Z
- Recorded fork run: yt29f5pxci7a
- Summary: Reviewer returned REQUEST CHANGES: `/history` repurposed shared `flattenTurnOrder()` to user-only, causing public extension `turnRowsInDisplayOrder()` and file `/rewind` to hide assistant/tool-cycle checkpoints. Fix fork launched; also removing verified dead tree contracts and small stale complexity identified by review.
- Reviewer verdict: REQUEST CHANGES. Blocking scenario: `BridgeTuiExtensionContext::turnRowsInDisplayOrder()` routes through user-only history rows, so file-rewind checkpoints captured on assistant advance-after-tools turns disappear from `/rewind`. Minimal fix is bridge iteration over all active retained turns plus focused extension-row regression proof. Fix fork `yt29f5pxci7a` launched; no PR transition until re-review approves.

## Task workflow update - 2026-08-06T02:52:08.053Z
- Recorded fork run: bsujwq8d8cps
- Summary: User rejected reviewer blocker and clarified intended contract: both `/history` and file `/rewind` show user-turn checkpoints only; assistant/tool-cycle turns remain internal. Correction fork is reverting the all-turn bridge behavior/test while preserving valid dead-code and simplification cleanup.
- Authoritative clarification after reviewer: user-only filtering in `turnRowsInDisplayOrder()` is intentional for both `/history` and `/rewind`; assistant/tool-cycle checkpoint rows must stay hidden. Reviewer finding was based on the wrong product assumption. Commit `b42a611c` changed this incorrectly. Correction fork `bsujwq8d8cps` launched to restore user-only rows, document the public contract accordingly, and invert the regression test while preserving unrelated valid cleanup.

## Task workflow update - 2026-08-06T03:12:30.774Z
- Recorded fork run: hu4l8x37tdfy
- Validation: Reviewer re-review: APPROVED; no blockers.; castor deptrac: OK — 0 violations.; castor phpstan: OK — 0 errors.; castor cs-check: OK — clean.; castor test:controller-replay: OK — 12 tests, 165 assertions.; castor test:tui: OK — 34 tests, 218 assertions.; castor test: FAILED only TuiSkillReadCardVirtualRenderTest contiguous path assertion; actual long worktree path wraps `docs/unrelated` across terminal line, matching previously documented noncausal failure.
- Summary: Re-review APPROVED with user-only `/history` and `/rewind` semantics. Focused task-to-PR validation passed except full `castor test`, blocked by a pre-existing path-wrap assertion that fails only in long worktree paths. Narrow test-only portability fix fork launched before final review/gate.
- Task-to-PR validation confirmed the known path-wrap test blocks the required gate in this worktree. Fork `hu4l8x37tdfy` launched for the smallest wrap-tolerant test assertion and full `castor test` proof; production history code is frozen.

## Task workflow update - 2026-08-06T03:19:03.399Z
- Recorded fork run: hu4l8x37tdfy
- Validation: Final reviewer: APPROVED — no blockers; test-only wrap regex is narrower/more coherent than prior independent substring assertions.; castor test: OK — 4,410 tests, 16,418 assertions.; castor test:controller-replay: OK — 12 tests, 165 assertions.; castor test:tui: OK — 34 tests, 218 assertions.; castor deptrac: OK — 0 violations.; castor phpstan: OK — 0 errors.; castor cs-check: OK — clean.; git diff --check: clean.
- Summary: Final reviewer APPROVED at `a58127c93c3b0bc7408139b865b86565f4826ee9`. Production behavior approved with explicit user-only `/history` and `/rewind` contract; test-only long-path portability fix also approved. Branch clean and ready for deterministic gate/PR.

## Task workflow update - 2026-08-06T03:21:03.364Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (108.7s).
- Pushed task/spike-linear-history-rewind to origin.
- branch 'task/spike-linear-history-rewind' set up to track 'origin/task/spike-linear-history-rewind'.
- Created PR: https://github.com/ineersa/agent-core/pull/366

## Task workflow update - 2026-08-06T03:21:10.641Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/366
- Updated PR Status: open
- Validation: Deterministic castor check: PASSED in 108.7s during CODE-REVIEW transition.
- Summary: Moved to CODE-REVIEW after final approval and deterministic gate. Branch pushed; PR #366 opened.

## Task workflow update - 2026-08-06T14:48:27.326Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR review requested a complete semantic rewrite: remove all remaining Tree/Leaf/Branch naming and tree-shaped DTO/logic, simplify around ordered linear history, and keep only the necessary AgentCore history interface for now. File `/rewind` remains separate; user-only picker semantics unchanged.

## Task workflow update - 2026-08-06T14:58:13.262Z
- Summary: PR review iteration scope finalized: rewrite the implementation as genuinely linear history, not tree-shaped machinery with new comments. Remove all conversational Tree/Leaf/Branch names, namespaces, DTO fields, ancestry walks, runtime event names, command/class names, tests, docs, and service wiring. Replace them with compact ordered history turns plus selected position. Keep the necessary AgentCore history boundary interface for now, as explicitly allowed.
- PR comments classified as blocking architectural feedback: remove `Contract/TurnTree` namespace; keep necessary AgentCore `HistoryTailDiscardInterface` coupling for now despite acknowledged boundary discomfort; rename/remove `LeafSet`, `RunLeafChangedEventFactory`, `run.leaf_changed`, all `TurnTree*` views/providers/projectors, and `TreeCommand*`/picker names. User explicitly requested proper rewrite and simplification, not cosmetic aliases.
- Approved target model: `HistoryProjector` produces one ordered `HistoryDTO` containing retained `HistoryTurnDTO` entries and `positionTurnNo`; no parent/children/root/path/leaf fields or ancestry walking. `HistoryReplayFilter` filters an ordered prefix and preserves only the command-seeding logic required by issue #183. `HistorySelectionService` selects the predecessor of a chosen user prompt and emits `history_position_set`; runtime emits `run.history_position_changed`; `/history` consumes a flat user-prompt `HistoryView`.
- Remove the Core `Contract/TurnTree` branch replay interface/DTO and adapter because replay filtering is CodingAgent session-local. Keep `HistoryTailDiscardInterface`, `RunStateRebuilderInterface` (rename leaf method to position terminology), and the necessary conversation history selection boundary, renamed away from file `/rewind` terminology where practical. Preserve file `/rewind`, user-only picker semantics, hidden-Git backend, OM session-global behavior, compaction semantics, in-flight behavior, and issue #183 command/tool resolution safeguards.
- Zero-stale-reference expectation applies only to conversational history: no `TurnTree`, `TurnBranch`, `BranchReplay`, `RunLeaf`, `LeafChanged`, `leaf_set`, `rewind_to_turn`, current/root/parent/child/path leaf/tree fields, `/tree`, or corresponding filenames/classes in AgentCore history contracts, CodingAgent session/runtime, TUI, their tests/docs/config. Legitimate Git trees, widget trees, session/subagent parent-child topology, and Git branches remain unchanged.

## Task workflow update - 2026-08-06T14:59:35.665Z
- Recorded fork run: lssn0fmfwusi
- Implementation fork `lssn0fmfwusi` launched for full semantic rewrite: flat ordered history DTO/projector/filter, history position event/selection/runtime protocol, history-named provider/views/TUI, removal of conversational TurnTree/Leaf/Branch namespaces/classes/fields/tests/docs/wiring, preservation of necessary AgentCore history interface and issue #183 safeguards.

## Task workflow update - 2026-08-06T17:32:21.940Z
- Recorded fork run: jmpca6fu7o1v
- Summary: First semantic-rewrite fork completed the production direction but timed out before test migration, Castor validation, and commit; dirty worktree preserved. Continuation fork launched to finish fixtures/tests/docs, remove the temporary old payload-key fallback, enforce zero stale conversational Tree/Leaf/Branch surface, run focused/full Castor lanes, and commit.
- Fork `lssn0fmfwusi` left 83-file dirty diff (+475/-2826 tracked, plus new flat History files) with no commit. Production rewrite substantially present: `HistoryProjector`, `HistoryReplayFilter`, `HistorySelectionService`, `history_position_set`, `select_history_turn`, `run.history_position_changed`, History-named provider/views/TUI, and deleted conversational TurnTree/Rewind production classes.
- Continuation fork `jmpca6fu7o1v` launched in the same worktree. It must preserve dirty work, finish all half-migrated tests/fixtures/docs, remove `turn_no` payload compatibility fallback, validate through Castor (`test`, deptrac, phpstan, cs-check, controller-replay, tui), and commit without push/status move.

## Task workflow update - 2026-08-06T17:45:39.140Z
- Validation: `castor test`: OK — 4,407 tests, 16,390 assertions (second full run; first had unrelated unchanged VirtualTerminal wrap flake, filtered and full rerun green).; `castor deptrac`: OK — 0 violations.; `castor phpstan`: OK — 0 errors.; `castor cs-check`: OK after `castor cs-fix` normalized 16 files.; `castor test:controller-replay`: OK — 12 tests, 165 assertions.; `castor test:tui`: OK — 34 tests, 222 assertions.; `git diff --check`: clean.; Conversational stale-name search (`TurnTree|TurnBranch|BranchReplay|RunLeaf|LeafChanged|LeafSet|leaf_set|rewind_to_turn|...|/tree`) across scoped src/tests/config/docs: zero hits.; Old position payload fallback search in src: zero hits.
- Summary: Semantic rewrite completed and committed at `1e2ab8c554999a6f1a280a1b40505abc10c49b89`. Conversational history is now flat ordered retained turns plus position; old Tree/Leaf/Branch production surface, payload keys, compatibility readers, runtime protocol names, TUI names, tests, docs, and wiring are removed. Worktree clean; cumulative branch diff is 118 files, +3251/-5613 (net -2362 LOC).
- Continuation fork `jmpca6fu7o1v` read testing skill and `tests/AGENTS.md`, finished the dirty test/fixture/docs migration, removed temporary `turn_no` compatibility fallback, validated both transports and TUI paths, and committed `1e2ab8c55`.
- Final architecture: `HistoryProjector` → `HistoryDTO{turns, positionTurnNo}`; `HistoryReplayFilter` retained-prefix replay + issue #183 command-seeding suppression; `HistorySelectionService::selectPrompt()` appends `history_position_set`; runtime command/event are `select_history_turn` / `run.history_position_changed`; flat user-only `HistoryView` feeds `/history` and file `/rewind`.
- Known non-blocking test environment note: an unchanged VirtualTerminal render test wrapped `<system-reminder>` once under ParaTest; filtered rerun and subsequent full suite passed. No production or test-file diff implicated.

## Task workflow update - 2026-08-06T18:06:50.674Z
- Summary: Re-review at `1e2ab8c55` requested changes: functionality is sound, but stale Tree/Leaf/Branch/Rewind terminology remains in comments/test names/fixture names; user-role filtering is duplicated; and the unmatched pending-command safeguard is split between replay filter and state replay service.
- Reviewer verdict: REQUEST CHANGES. No correctness/security/data-loss blocker in the flat history flow; AgentCore history interfaces explicitly accepted. Blocking cleanup is mechanical stale terminology plus one replay simplification.
- Independent scout proved `filterAbandonedLaunchCommandsAtPosition()` is not wholly redundant today: `HistoryReplayFilter` suppresses commands mapped to a discarded `TurnAdvanced`, while the service helper uniquely suppresses queued/applied commands after completion before `history_select` when no next `TurnAdvanced` exists (including compaction/interrupted cases). However, keeping it service-local leaves `rebuildIfStale()` crash recovery unprotected. Fix direction: move unmatched pending-command suppression into `HistoryReplayFilter`, delete the service helper/call, and add one minimal regression proving the no-next-turn sequence is excluded.
- Additional hardening selected from reviewer observations: make `HistoryReplayFilter` a required dependency of state and hot-prompt replay services, eliminating nullable unfiltered production fallback; remove duplicate user-role filtering from bridge/picker because `SessionHistoryProvider` guarantees user-only `HistoryView`; rename stale E2E fixture files/temp prefixes and all conversational comments/test names.

## Task workflow update - 2026-08-06T18:07:22.447Z
- Recorded fork run: dd5pk8sb85bo
- Fix fork `dd5pk8sb85bo` launched to address re-review: clean all stale conversational terminology including fixture filenames, consolidate mapped+unmatched pending-command suppression in `HistoryReplayFilter`, delete service-local duplicate helper, make replay filters required (no unfiltered fallback), remove duplicate user-role filtering, add focused unmatched-command and middle-selection proofs, run Castor lanes, and commit.

## Task workflow update - 2026-08-06T18:20:36.868Z
- Validation: Focused history/runtime/TUI filters: OK — 118 tests, 596 assertions.; `castor test`: OK — 4,409 tests, 16,401 assertions.; `castor deptrac`: OK — 0 violations.; `castor phpstan`: OK — 0 errors.; `castor cs-check`: OK after `castor cs-fix`.; `castor test:controller-replay`: OK — 12 tests, 165 assertions.; `castor test:tui`: OK — 34 tests, 222 assertions.; `git diff --check origin/main...HEAD`: clean.; Stale implementation search (`filterAbandoned|userPromptRows|?HistoryReplayFilter|postLeaf|leaf_set|TurnTree|TreePicker|tui-tree-rewind|RunLeaf|LeafChanged|rewind_to_turn|BranchReplay`) across src/tests/config/docs: zero hits.
- Summary: Re-review fixes completed at `68af8a3e80023de69f33a5d3333695a1444bddd9`; worktree clean. Pending-command suppression is consolidated in required `HistoryReplayFilter` for both normal and crash-recovery rebuilds; service-local duplicate removed; user-only role filtering single-sourced; stale conversational terminology and fixture names removed.
- Fork `dd5pk8sb85bo` committed `68af8a3e8` (43 files, +472/-285). `HistoryReplayFilter` now suppresses both commands mapped to discarded turns and unmatched commands persisted after completion before `history_select`; `SessionRunStateReplayService` and `SessionHotPromptReplayService` require the filter and never fall back to unfiltered canonical events.
- Removed redundant bridge/picker user-role filters; `SessionHistoryProvider` is the sole user-prompt boundary. Renamed stale E2E fixtures to `tui-history-select-*`, updated comments/test names, and added focused unmatched-command crash-recovery plus middle-prompt selection proofs.
- Cumulative branch vs origin/main: 136 files, +3608/-5783 (net -2175 LOC). Legitimate remaining tree/branch/rewind wording is restricted to Git/source/widget trees, session/subagent topology, Git branch display, OM non-branch-aware wording, and separate file `/rewind`.

## Task workflow update - 2026-08-06T18:30:12.996Z
- Recorded fork run: 01b2t6rcnmz4
- Summary: Final re-review approved `68af8a3e8` with no blockers. One last deletion-focused cleanup is running before PR update: remove four cosmetic conversational leftovers, delete malformed no-turn production fallback, and collapse duplicated boundary-0 replay into the shared filter path.
- Final reviewer verdict at `68af8a3e8`: APPROVED. Confirmed unmatched command recovery, required filtering, single user-role gate, middle selection behavior, payload clean cut, and no stale banned compound terms. AgentCore history interfaces explicitly accepted.
- Cleanup fork `01b2t6rcnmz4` launched for reviewer-observed minimality: canonicalize invalid no-`turn_advanced` test fixtures instead of production pass-through, replace manual boundary-0 replay branch with the shared filter path while forcing selected turn number, and remove remaining conversational `rewind`/`branch`/`sampleTree` wording.

## Task workflow update - 2026-08-06T18:38:56.567Z
- Validation: Focused history/replay/pipeline/TUI filters: OK — 83 tests, 477 assertions.; `castor test`: OK — 4,409 tests, 16,401 assertions.; `castor deptrac`: OK — 0 violations.; `castor phpstan`: OK — 0 errors.; `castor cs-check`: OK — 0 files fixed.; `castor test:controller-replay`: OK — 12 tests, 165 assertions.; `castor test:tui`: OK — 34 tests, 222 assertions.; `git diff --check origin/main...HEAD`: clean.; Cleanup search (`sampleTree|testTurnAllocationAfterRewind|tui-tree-rewind|streamHasTurnAdvanced|the rewind is observed|?HistoryReplayFilter|filterAbandoned`): zero hits.
- Summary: Post-approval deletion cleanup committed at `fca333cb55661b80393dbdbf5e126cdb226af9ad`; worktree clean. Removed malformed no-`TurnAdvanced` production fallback, unified boundary-0/nonzero position rebuild through `HistoryReplayFilter`, canonicalized tests, and removed final reviewer-observed conversational wording.
- Fork `01b2t6rcnmz4` committed `fca333cb5` (12 files, +128/-100). Invalid hot-prompt fixtures now emit canonical `TurnAdvanced` events rather than requiring a production pass-through.
- `SessionRunStateReplayService::rebuildAtPosition()` now has one filter/reducer/state-result path for all positions; rebuilt `turnNo` is explicitly the requested position and `lastSeq` remains canonical max.
- Cumulative branch vs origin/main: 136 files, +3666/-5813 (net -2147 LOC).

## Task workflow update - 2026-08-06T18:43:47.671Z
- Validation: Final reviewer: APPROVED — pass-through removal, unified position rebuild, canonical fixtures, replay safety, both transports, history/file-rewind behavior, terminology scope, and dead-code/compatibility checks all verified.; Focused/full Castor evidence remains green: test 4,409/16,401; deptrac 0; phpstan 0; cs-check clean; controller-replay 12/165; TUI 34/222.
- Summary: Final reviewer APPROVED HEAD `fca333cb55661b80393dbdbf5e126cdb226af9ad`. The cumulative rewrite is ready to update PR #366.
- Final gate reviewer traced the 12-file cleanup plus cumulative history path and returned APPROVED with no blockers or nice-to-have changes required. Confirmed valid pre-first-turn streams still pass normal filtering, position-0 output is equivalent, selected turn/lastSeq invariants hold, and issue #183 suppression is unchanged.

## Task workflow update - 2026-08-06T18:45:38.957Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 600s)...
- castor check passed (102.5s).
- Pushed task/spike-linear-history-rewind to origin.
- branch 'task/spike-linear-history-rewind' set up to track 'origin/task/spike-linear-history-rewind'.
- PR already exists: https://github.com/ineersa/agent-core/pull/366
- Validation: Final reviewer APPROVED.; Focused history/runtime/TUI tests: 83 tests, 477 assertions OK.; `castor test`: 4,409 tests, 16,401 assertions OK.; `castor deptrac`: 0 violations.; `castor phpstan`: 0 errors.; `castor cs-check`: clean.; `castor test:controller-replay`: 12 tests, 165 assertions OK.; `castor test:tui`: 34 tests, 222 assertions OK.; Stale conversational naming and compatibility/fallback searches: clean.
- Summary: PR review iteration complete: replaced residual tree/leaf/branch conversational architecture with flat ordered linear history, consolidated replay filtering and recovery safety, removed compatibility/fallback paths and duplicate role/position filtering, preserved user-only `/history` and file `/rewind` behavior, and received final reviewer approval at `fca333cb5`.

## Task workflow update - 2026-08-06T20:14:45.599Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #366 has a merge conflict. Reopened to merge current `origin/main` into the task branch, resolve conflicts without dropping the linear-history rewrite or unrelated upstream changes, validate, and update the PR.

## Task workflow update - 2026-08-06T20:15:10.719Z
- Recorded fork run: n1qs8jarjsmc
- Merge-up fork `n1qs8jarjsmc` launched to fetch and merge current `origin/main` into the task branch, semantically resolve renamed/deleted History conflicts, preserve upstream changes plus approved linear-history behavior, run focused/full Castor lanes, and commit without push/status transition.

## Task workflow update - 2026-08-06T20:22:24.911Z
- Validation: Focused history/replay/skill/file-rewind tests: OK — 110 tests, 465 assertions.; `castor test`: OK — 4,405 tests, 16,394 assertions.; `castor deptrac`: OK — 0 violations.; `castor phpstan`: OK — 0 errors.; `castor cs-check`: OK — 0 files fixed.; `castor test:controller-replay`: OK — 12 tests, 165 assertions.; `castor test:tui`: OK — 34 tests, 222 assertions.; `git diff --check origin/main...HEAD`: clean.; Conflict-marker search: zero markers.; Stale conversational tree-symbol search in scoped src/tests/config/docs/file-rewind: zero hits.
- Summary: Merged current `origin/main` (`4b4f32906`) into the task branch at two-parent merge commit `d8426566e25b097f4d000aab39032bb76fe85453`; worktree clean. Resolved two conflicts semantically without restoring deleted tree DI or losing upstream cleanup.
- Merge fork `n1qs8jarjsmc` read testing skill and `tests/AGENTS.md`, merged `origin/main` at `4b4f32906`, and committed `d8426566e` with parents `fca333cb5` + `4b4f32906`.
- Conflict `config/services.yaml`: preserved History selection/tail-discard/replay wiring and `SessionEventsExportService`; rejected obsolete upstream TurnTree/BranchReplay aliases whose implementations are deleted on this branch.
- Conflict `TuiSkillReadCardVirtualRenderTest.php`: accepted upstream newline-stripping path assertion, which is more robust than the branch's split-point regex. Upstream PR #365 cleanup and file-rewind null-check removal otherwise auto-merged intact.

## Task workflow update - 2026-08-06T20:28:03.887Z
- Validation: Reviewer APPROVED merge commit structure, both semantic conflict resolutions, complete upstream PR #365 integration, History DI/runtime/replay behavior, both transports, stale symbol absence, and cumulative specification fidelity.
- Summary: Merge-up re-review APPROVED at `d8426566e`; ready to update PR #366.
- Merge reviewer confirmed all 40 upstream PR #365 files are present (39 byte-equivalent; only expected History-wired services.yaml difference), no deleted tree aliases/classes returned, and the upstream VirtualTerminal assertion is stronger than the prior branch regex.

## Task workflow update - 2026-08-06T20:43:45.309Z
- Summary: Latest PR comments confirm the prior pass still renamed the old projector instead of reconsidering it. Architecture analysis completed: separate retained replay turn numbers from sparse actual user prompts, delete assistant/title/role inference and presentation fields, fix explicit position 0, collapse replay wrappers/fallbacks/duplicate scans, and use existing immutable state copying.
- New PR comments at `fca333cb5` target `HistoryProjector` one-line helpers, duplicated text helpers, role classification, and assistant-row/title logic. User explicitly requested self-directed full architecture reconsideration rather than another naming pass.
- Root design flaw: current `HistoryProjector` builds a DTO for every internal turn, guesses `displayRole`, extracts assistant titles, then `SessionHistoryProvider` filters assistant rows out. Target model instead stores `list<int> retainedTurnNos` for replay plus sparse `array<int,string> promptsByTurnNo` for actual human prompts; internal tool/assistant/shell turns remain replay boundaries but never become presentation DTOs.
- Delete `HistoryTurnDTO`. Reduce `HistoryPromptView` to `turnNo + promptText`; remove `title`, `displayRole`, and `isPosition`. `HistoryView` becomes `prompts + int positionTurnNo`; remove unused runId. Public ExtensionApi keeps its published primitive row shape by deriving a sanitized title and hardcoding `displayRole=user` only at the bridge.
- Rewrite `HistoryProjector` as one ordered forward scan: capture initial RunStarted user prompt; capture only applied human `follow_up`/`steer` text (exclude generated `append_message`); attach latest applied prompt to next TurnAdvanced; retain every TurnAdvanced internally; prune turns/prompts on tail discard; preserve pending prompt through compaction; clear on discard/anchor. Remove backward anchor rescans, assistant text/title extraction, role/step-id heuristics, fixed 80-char truncation, placeholders, and one-line wrappers. Preserve current latest-applied policy when multiple commands precede one anchor.
- Fix explicit boundary 0: position is an int with 0 meaning before first, never nullable. TurnAdvanced moves it to tip; history_position_set can set 0; tail discard sets retained boundary. This fixes the current bug where explicit 0 becomes null then incorrectly defaults back to tip.
- Simplify `HistoryReplayFilter`: pure event-array API without runId, delete diagnostics-only `HistoryReplayResultDTO`, build projection once per call, sort once for filter helper scans, preserve both mapped discarded-turn and unmatched post-completion command suppression. Callers log canonical counts locally.
- Use `RunState::with()` instead of repeated full constructor copies in RunMessageProcessor/history replay/selection. Tail discard compares selected turn to linear tip without remapping DTOs. Keep necessary AgentCore history interfaces and shared discard choke point as explicitly allowed.
- Remove SessionInitializer's full canonical replay fallback after history projection failure; valid resume always uses retained-prefix transcript at int position (including 0), and projection failure propagates/fails closed rather than resurrecting discarded content. No compatibility or dual behavior.
- Test budget: rewrite existing projector/filter/provider/picker/selection/resume proofs; specifically prove internal turns retained but absent from prompt rows, initial/follow_up/steer prompts preserved exactly, generated append_message/assistant text absent, explicit position 0 survives projection/resume, selecting internal turn fails, discard/new tail prunes sparse prompts, ExtensionApi/file-rewind remains user-only. Preserve issue #183 tests and both transport/controller/TUI proof.

## Task workflow update - 2026-08-06T20:44:56.937Z
- Recorded fork run: 2pwh21cl670l
- Architecture rewrite fork `2pwh21cl670l` launched from merged HEAD `d8426566e`. It must replace the role/title/assistant projector with retained turn numbers + sparse human prompts, fix explicit position 0, simplify runtime views/filter results/state copies/resume fallback, preserve issue #183/internal replay anchors/file-rewind API, run all focused/full Castor lanes, and commit without push/status move.
