# Move mounted transcript to semantic widgets and incremental reconciliation

## Goal
Follow-up to PR #323 (`migrate-transcript-to-mounted-symfony-tui-widgets`). The first migration mounts a real Symfony widget subtree and fixes live WidgetContext/Markdown theming, but `TranscriptMountedWidget` still combines presentation projection, tool pairing/suppression, full-history fingerprinting, ordering, and widget reconciliation.

Refine the architecture so Symfony owns render caching and complex transcript concepts own their widget state. Ordinary streaming/tail updates must not scan, serialize, or hash the entire historical transcript, and unchanged mounted widgets must keep identity without application-level rendered-line caches.

## Intended direction

- Move canonical-block → visual-node projection out of `TranscriptMountedWidget` into a typed TUI presentation model/projector.
- Replace associative visual-item arrays with a typed immutable visual-node value carrying stable key, source block references and presentation dependencies.
- Replace full text/meta fingerprints with immutable source identity or explicit source/presentation revisions. Dirty detection remains application-owned; rendered-output caching remains Symfony-owned.
- Keep one transcript root container, but give complex mutable visual concepts coherent semantic widgets (for example message/reasoning, tool exchange, question and structured subagent views). Continue using plain Symfony `TextWidget`/`MarkdownWidget` for trivial static content; do not create a class per label.
- Make `TranscriptMountedWidget` a small keyed structural reconciler rather than the owner of block-specific presentation policy.
- Introduce an explicit incremental change path for normal append/update/remove operations. Preserve full replacement for bootstrap, resume, rewind, branch replacement and genuine reorder/non-tail insertion.
- Remove redundant transcript array copies and avoid retained duplicate source/render state.
- Profile the end-to-end large-session path, including `RuntimeEventPoller::synchronizeProjectedBlocks()`; replace repeated linear ID searches if they remain O(B²).
- Remove obsolete all-in-one/offscreen transcript adapters if their remaining tests/consumers can use the mounted production path directly; do not retain fallback rendering modes.

## Performance and memory contract

Use deterministic structural counters/identity assertions rather than flaky wall-clock thresholds. Record representative Castor-backed measurements for small and large transcripts. Ordinary tail streaming should touch only the changed visual dependency set and must not reserialize or rehash finalized historical text/metadata. Document the unavoidable O(number of mounted visual nodes) memory/layout ceiling while eliminating avoidable duplicate arrays, superseded widget trees and full-history temporary allocations.

## Preserve

Canonical ordering; tool call/result exchange grouping; duplicate-result, HITL/question and empty-assistant suppression; turn separators; preview expansion; explicit removal; branch replacement; stable Markdown WidgetContext/theming; native terminal scrollback; stock Symfony `Tui` and `ScreenWriter`.

## Explicit non-goals

- No viewport virtualization, clipping compositor, fixed layout, terminal scroll region or transcript pagination.
- No ScreenWriter override/alias, terminal telemetry experiment, tmux configuration detection or ANSI writer work.
- No manual rendered-line cache parallel to Symfony's widget cache.
- No feature flag, dual transcript renderer or compatibility fallback.
- No hard history cap without a separate product decision.

## Validation expectations

Load the testing skill and follow `tests/AGENTS.md`. Add the smallest virtual/in-process structural proofs for semantic widget identity and incremental update scope, preserve replay-backed transcript product behavior, and run all QA through Castor. Tmux remains integration smoke only; do not add a broad new tmux journey when virtual identity/counter assertions prove the contract. Full `castor check` is required before CODE-REVIEW because runtime/TUI paths are touched.

## Acceptance criteria
- `TranscriptMountedWidget` is primarily a keyed container reconciler; block grouping, suppression, separator and presentation-policy logic lives in a typed presentation model outside the widget.
- Routine streaming/tail updates do not serialize/hash every historical block and do not rebuild or replace unchanged semantic widgets.
- Complex mutable transcript concepts use mounted semantic Symfony widget components; trivial static content continues to use native `TextWidget`/`MarkdownWidget` without needless wrappers or one-class-per-kind scaffolding.
- Dirty detection uses immutable source identity or explicit revisions; Symfony widget revision/render caches remain the only rendered-output cache.
- Normal append/update/remove operations have an incremental path; bootstrap/resume/rewind/branch reorder retain an explicit full-replacement path.
- The large-session runtime synchronization path is linear or better per snapshot; repeated linear block-ID lookup inside a full loop is removed if confirmed.
- Avoidable duplicate transcript arrays, superseded widget trees and full-history temporary fingerprint allocations are removed; remaining linear mounted-tree memory behavior is measured and documented.
- Existing transcript behavior remains covered: ordering, tool exchanges, suppression, separators, preview expansion, removals, branch replacement and live Markdown theming.
- No viewport, scroll-region, custom ScreenWriter, terminal telemetry, compatibility mode, feature flag or manual rendered-line cache is introduced.
- Focused Castor validation and deterministic `castor check` pass; automated proof asserts operation/identity scope rather than unstable timing thresholds.

## Workflow metadata
Status: DONE
Branch: task/semantic-transcript-widgets-incremental-reconciliation
Worktree: /home/ineersa/projects/agent-core-worktrees/semantic-transcript-widgets-incremental-reconciliation
Fork run: m9cgzl9ar8o4
PR URL: https://github.com/ineersa/agent-core/pull/325
PR Status: merged
Started: 2026-07-27T15:50:58.727Z
Completed: 2026-07-27T19:53:38.453Z

## Work log
- Created: 2026-07-27T15:28:53.068Z

## Task workflow update - 2026-07-27T15:50:58.727Z
- Moved TODO → IN-PROGRESS.
- Created branch task/semantic-transcript-widgets-incremental-reconciliation.
- Created worktree /home/ineersa/projects/agent-core-worktrees/semantic-transcript-widgets-incremental-reconciliation.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/semantic-transcript-widgets-incremental-reconciliation.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/semantic-transcript-widgets-incremental-reconciliation.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/semantic-transcript-widgets-incremental-reconciliation.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/semantic-transcript-widgets-incremental-reconciliation.
- Summary: Starting implementation after PR #323 merged. Scope: simplify the mounted transcript architecture around semantic widgets and incremental reconciliation while preserving stock Symfony TUI/ScreenWriter and native terminal behavior.

## Task workflow update - 2026-07-27T15:59:37.476Z
- Recorded fork run: 2yu7g6vqx68x
- Task-start prerequisites completed: loaded task-workflow, testing, and ponytail skills; read tests/AGENTS.md.
- Three read-only scouts traced runtime projection, semantic widget lifecycle/Symfony caching, and the virtual/runtime test strategy. All confirmed they read the required testing docs.
- Implementation fork 2yu7g6vqx68x launched in the task worktree with explicit incremental-dataflow, semantic-widget, no-fallback, test-thesis, and focused-Castor constraints.

## Task workflow update - 2026-07-27T16:23:14.167Z
- Recorded fork run: 0ui0yabor2uw
- Validation: Initial fork read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`; Initial focused Castor tests: 159 tests/710 assertions plus 69 related tests passed; Initial `castor deptrac`: 0 violations; Initial scoped `castor phpstan`: 0 errors; Initial `castor cs-check`: clean; Parent verified commit 3c871faa50e2b624487116b86811de3a18a1373e exists and worktree was clean
- Summary: Initial implementation commit 3c871faa5 established canonical TranscriptChangeSet dirty tracking, propagated removals, removed the O(B²) poller lookup, extracted presentation policy, and replaced text/meta fingerprints with object identity. Parent verification found the ordinary mounted path still rebuilt its ID index and scanned the full visual history, and the single generic semantic wrapper did not yet meet the complex-widget decomposition requirement. A focused acceptance-fix fork is now addressing those blockers.
- Initial implementation fork 2yu7g6vqx68x completed commit 3c871faa5 (24 files, +1175/-640).
- Acceptance gap found: TranscriptProjectionState::drainChanges and TranscriptMountedWidget/TranscriptVisualProjector still walked full history on ordinary updates; mounted widget retained duplicate canonical block list.
- Acceptance gap found: SemanticTranscriptNodeWidget remained a generic catch-all wrapper rather than semantic components for tool exchange/question/subagent with native primitives for trivial nodes.
- Focused follow-up implementation fork 0ui0yabor2uw launched to produce dependency-bounded visual patches, semantic components, and batch-linear removal handling.

## Task workflow update - 2026-07-27T16:40:23.602Z
- Recorded fork run: ey7a9g3zhkqj
- Validation: Second fork required testing docs read and focused Castor-only validation confirmed; Second focused transcript/runtime filters passed (65 tests, 349 assertions aggregate); Second `castor deptrac`: 0 violations; Second scoped `castor phpstan`: 0 errors; Second `castor cs-check`: clean; Parent verified commit f0731489494145e4bbe263fc1d05ba9823f25272 exists and worktree is clean
- Summary: Second implementation commit f07314894 added stateful visual patches and dedicated semantic widgets. Parent verification accepted the deliberate O(B) low-frequency structural-policy ceiling but found the pure streaming mounted consumer still scanned the full order, a test-only production accessor, and a generic-widget replacement ordering bug. Final narrow fork is correcting those without expanding into a speculative dependency graph.
- Second fork 0ui0yabor2uw completed commit f07314894 (+1357/-374): O(changes) canonical drain, stateful visual model, typed visual patches, dedicated streaming/tool/question/subagent widgets, batch-linear session removals.
- Accepted ponytail ceiling: rare structural presentation-policy changes remain O(B) internally and may emit bounded mount patches; a per-exchange/neighbor dependency graph is deferred unless profiling shows these low-frequency scans matter.
- Final narrow fork ey7a9g3zhkqj launched to make pure stream mounted apply truly O(changes), remove test-only lastVisualPatch API, preserve generic-node order on update, and correct docs.

## Task workflow update - 2026-07-27T16:49:07.572Z
- Recorded fork run: w94rni2gbn8s
- Validation: Required testing skill, tests/AGENTS.md, root AGENTS.md, and task file read by implementation forks; Focused projection/runtime/TUI aggregate: 65 tests, 365 assertions passed; Final focused `castor test --filter='TranscriptVisualProjectorTest|TuiMountedTranscriptVirtualTest'`: 6 tests, 82 assertions passed; Focused RuntimeEventPoller, TranscriptProjectionChangeSet, and TuiTranscriptBlocksVirtualRender tests passed: 59 tests, 283 assertions; `castor deptrac`: 0 violations; Scoped `castor phpstan`: 0 errors; `castor cs-check`: clean; Parent verified commits 3c871faa5, f07314894, 44a249efd, b750b3a76 exist; final worktree clean; expected 29-file task diff
- Summary: Task-start implementation complete at b750b3a767f3d8a8b062337b6fc0edcaf6fc905a. Canonical projector changes now flow as TranscriptChangeSet deltas through indexed TUI session state into typed visual patches. High-frequency content updates are upsert-only O(changes), use immutable source identity instead of full text/meta fingerprints, and mutate stable semantic/native widgets in place. Full replacement remains explicit for bootstrap/resume/branch/preview/reorder; low-frequency structural tool/question/user policy currently reprojects O(B) then emits a bounded mounted patch when safe. Dedicated streaming Markdown, tool exchange, question, and subagent widgets replace the catch-all visual wrapper. Worktree is clean; no push/PR/full gate performed in task-start.
- Final cleanup fork w94rni2gbn8s committed b750b3a76 (-52/+13): content-only visual patches are now upserts-only with no removals/order scan; unused TranscriptVisualPatch::incremental compatibility alias deleted.
- Task-start stopped after implementation verification per workflow. Full castor check, reviewer, push, and PR transition are deferred to task-to-pr.
- Deliberate ceiling recorded with ponytail comment: structural tool/question/user/separator policy remains O(B); add per-exchange/neighbor dependency graph only if profiling shows low-frequency structural scans matter.

## Task workflow update - 2026-07-27T18:04:16.490Z
- Recorded fork run: 82h8rrqha8p6
- Validation: Fork 82h8rrqha8p6 confirmed root/task/testing docs read and Castor-only QA; castor test --filter='TuiMountedTranscriptVirtualTest|TranscriptVisualProjectorTest|TuiTranscriptBlocksVirtualRenderTest|RuntimeEventPollerTest': 63 tests, 360 assertions OK; castor phpstan --path=src/Tui/Transcript/TranscriptVisualProjector.php: 0 errors; castor cs-check: clean; castor test:tui --filter=TuiFileRewindE2eTest: 2 tests, 13 assertions OK; Worktree HEAD b1b589b83; full castor check still pending
- Summary: Task-to-PR focused TUI validation exposed and fixed a deterministic branch regression at b1b589b83: an assistant content update adjacent to a completed tool exchange expanded the unchanged ToolResult secondary, reclassified it as standalone GENERIC at the stable exchange key, and crashed ToolExchangeTranscriptWidget. The content-only projector now skips an expanded block when it is only the identical secondary source of the existing semantic node; direct dirty ToolResult updates still take the structural path. Added one virtual regression proving exchange kind/order/widget identity and assistant update. Reviewer subagent was relaunched four times after the fix but its configured provider returned a 429 usage-limit error/no verdict; independent read-only fork review jd18anv666xv is running as fallback.

## Task workflow update - 2026-07-27T18:11:13.617Z
- Recorded fork run: jd18anv666xv
- Validation: Reviewer fallback jd18anv666xv: APPROVED, no blockers; castor test: 4539 tests, 15901 assertions OK; castor deptrac: 0 violations, 0 errors; castor phpstan: 0 errors; castor cs-check: clean; castor test:tui: 37 tests, 190 assertions OK
- Summary: Final independent read-only review at b1b589b83 APPROVED with no blocking findings. The configured reviewer subagent was attempted four times but its provider returned a 429 usage-limit error/no verdict; fallback fork reviewer verified the full origin/main...HEAD diff, the exchange-secondary root fix, prior cleanup findings, architecture/performance acceptance, and test pyramid. Full focused task-to-PR validation is green; proceeding to deterministic move_task CODE-REVIEW gate.

## Task workflow update - 2026-07-27T18:13:30.547Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (119.4s).
- Pushed task/semantic-transcript-widgets-incremental-reconciliation to origin.
- branch 'task/semantic-transcript-widgets-incremental-reconciliation' set up to track 'origin/task/semantic-transcript-widgets-incremental-reconciliation'.
- Created PR: https://github.com/ineersa/agent-core/pull/325
- Validation: Review APPROVED (fallback independent fork; reviewer subagent provider quota documented); castor test: 4539 tests / 15901 assertions OK; castor test:tui: 37 tests / 190 assertions OK; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean; No stale QA worker candidates before gate
- Summary: Approved full-diff review and focused task-to-PR validation completed at b1b589b83. Root-cause fix for adjacent assistant update corrupting completed tool exchange included and regression-proven. Moving through deterministic Castor gate, push, and PR creation.

## Task workflow update - 2026-07-27T19:37:35.281Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User requested a PR iteration before further review: simplify TranscriptMountedWidget to the minimum keyed adapter over Symfony ContainerWidget. Do not launch reviewer yet. Remove duplicated projector reconciliation state/logic, preserve only key→widget ownership and patch application, push the committed change to the existing PR for user testing, and keep task IN-PROGRESS until user confirms.

## Task workflow update - 2026-07-27T19:41:11.696Z
- Recorded fork run: m9cgzl9ar8o4
- Validation: Fork m9cgzl9ar8o4 confirmed task-workflow/testing/tests docs read and Castor-only QA; castor test --filter='TuiMountedTranscriptVirtualTest|TranscriptVisualProjectorTest|TuiTranscriptBlocksVirtualRenderTest': 37 tests, 195 assertions OK; castor phpstan --path=src/Tui/Transcript/TranscriptMountedWidget.php: 0 errors; castor cs-check: clean; castor test:tui --filter=TuiFileRewindE2eTest: 2 tests, 13 assertions OK; Pushed b1b589b83..9e7e140f9 to origin/task/semantic-transcript-widgets-incremental-reconciliation
- Summary: User-requested PR iteration pushed at 9e7e140f9. TranscriptMountedWidget was reduced from 362 to 296 LOC and now retains only stable key→Symfony widget ownership. Removed duplicated applied visual-node state, mounted order state, survivor/non-tail detection, and complex ordered reconciliation already guaranteed by TranscriptVisualProjector/TranscriptVisualPatch. Content patches bind changed keys; structural patches remove/update/tail-append; full patches clear and ordered re-add while retaining compatible widget instances. Task remains IN-PROGRESS and no reviewer was launched, per user request, while the user tests PR #325.

## Task workflow update - 2026-07-27T19:52:49.392Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/325
- Updated PR Status: merged
- Summary: User confirmed the simplified implementation works and PR #325 is merged on GitHub at merge commit 716d034650461c7ec17587d6bf0a25d17232446d. Proceeding directly from IN-PROGRESS to DONE because the task was temporarily moved back for the requested pre-merge iteration and the user merged the updated PR before a metadata-only return to CODE-REVIEW.

## Task workflow update - 2026-07-27T19:53:38.453Z
- Moved IN-PROGRESS → DONE.
- Merged task/semantic-transcript-widgets-incremental-reconciliation into integration checkout.
- Auto-merging docs/tui-architecture.md
Auto-merging src/Tui/Application/InteractiveMode.php
Merge made by the 'ort' strategy.
 docs/tui-architecture.md                           |  66 +-
 .../Contract/TranscriptProjectorInterface.php      |  15 +-
 .../Runtime/Projection/TranscriptChangeSet.php     |  72 +++
 .../Projection/TranscriptProjectionState.php       | 131 +++-
 .../ProjectionPipeline/TranscriptProjector.php     |   6 +
 src/Tui/Application/InteractiveMode.php            |  14 +-
 .../ImagePaste/PastedImageSubmissionService.php    |   4 +-
 src/Tui/Listener/CancelListener.php                |   2 +-
 src/Tui/Listener/ImagePasteInputListener.php       |   4 +-
 src/Tui/Listener/SubmitListener.php                |  20 +-
 src/Tui/Listener/TickPollListener.php              |   8 +-
 src/Tui/Runtime/RuntimeEventPoller.php             | 122 ++--
 src/Tui/Runtime/TuiRuntimeEventApplier.php         |  12 +
 src/Tui/Runtime/TuiSessionState.php                | 168 ++++-
 src/Tui/Screen/ChatScreen.php                      |  19 +-
 src/Tui/Transcript/QuestionTranscriptWidget.php    |  60 ++
 .../StreamingMarkdownTranscriptWidget.php          |  74 +++
 src/Tui/Transcript/SubagentTranscriptWidget.php    |  60 ++
 .../Transcript/ToolExchangeTranscriptWidget.php    |  67 ++
 .../Transcript/TranscriptBlockWidgetFactory.php    |   5 +
 src/Tui/Transcript/TranscriptMountedWidget.php     | 646 ++++++-------------
 src/Tui/Transcript/TranscriptVisualNode.php        |  45 ++
 src/Tui/Transcript/TranscriptVisualNodeWidget.php  |  41 --
 src/Tui/Transcript/TranscriptVisualPatch.php       | 104 ++++
 src/Tui/Transcript/TranscriptVisualProjector.php   | 692 +++++++++++++++++++++
 .../TranscriptProjectionChangeSetTest.php          | 183 ++++++
 tests/Tui/Runtime/RuntimeEventPollerTest.php       | 110 ++--
 .../Tui/Screen/TuiMountedTranscriptVirtualTest.php | 225 ++++++-
 .../Transcript/TranscriptVisualProjectorTest.php   |  72 +++
 29 files changed, 2337 insertions(+), 710 deletions(-)
 create mode 100644 src/CodingAgent/Runtime/Projection/TranscriptChangeSet.php
 create mode 100644 src/Tui/Transcript/QuestionTranscriptWidget.php
 create mode 100644 src/Tui/Transcript/StreamingMarkdownTranscriptWidget.php
 create mode 100644 src/Tui/Transcript/SubagentTranscriptWidget.php
 create mode 100644 src/Tui/Transcript/ToolExchangeTranscriptWidget.php
 create mode 100644 src/Tui/Transcript/TranscriptVisualNode.php
 delete mode 100644 src/Tui/Transcript/TranscriptVisualNodeWidget.php
 create mode 100644 src/Tui/Transcript/TranscriptVisualPatch.php
 create mode 100644 src/Tui/Transcript/TranscriptVisualProjector.php
 create mode 100644 tests/CodingAgent/Runtime/Projection/TranscriptProjectionChangeSetTest.php
 create mode 100644 tests/Tui/Transcript/TranscriptVisualProjectorTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/semantic-transcript-widgets-incremental-reconciliation.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/semantic-transcript-widgets-incremental-reconciliation.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #325 merged at 716d034650461c7ec17587d6bf0a25d17232446d; User manual testing confirmed simplified implementation works; Pre-PR deterministic castor check passed in 119.4s; Final simplification focused virtual tests: 37 tests / 195 assertions OK; Final simplification focused TUI rewind E2E: 2 tests / 13 assertions OK; Final simplification scoped phpstan: 0 errors; cs-check clean; Worktree clean after user-approved restore of .hatfield/settings.yaml
- Summary: PR #325 merged after user tested and confirmed the simplified TranscriptMountedWidget works. Final implementation provides incremental canonical changes, stateful semantic visual projection, stable mounted Symfony widgets, and a minimal keyed patch adapter with no custom renderer/cache/viewport path. User approved restoring the test-only .hatfield/settings.yaml theme change so the worktree could be cleaned safely.

## Task workflow update - 2026-07-27T20:00:24.740Z
- Validation: Focused castor test --filter=SubagentLivePickerControllerTest: 17 tests / 56 assertions OK; castor test:controller-replay rerun: 10 tests / 135 assertions OK; Final LLM_MODE=true castor check: quality OK in 332.4s; Final gate unit: 4530 tests / 15832 assertions OK; Final gate controller replay: 10 tests / 135 assertions OK; Final gate TUI: 37 tests / 190 assertions OK; Final gate llm-real: 13 tests / 175 assertions OK; Final gate deptrac/phpstan/cs-check OK; cache guard, artifact integrity, and leak check OK
- Summary: Post-merge validation completed on integration checkout. First full gate had two unrelated load-sensitive failures (SubagentLivePickerController stale export feedback fixture and LLM worker-pool startup observing only two of three consumers); both passed immediately in focused/sequential reruns, no stale workers were present, and the deterministic full gate then passed completely. Task is DONE and worktree/IDE exclusions are removed.
