# tui-03: Deepen the TUI transcript widget architecture

## Goal
## Goal
Combine the two transcript architecture candidates: replace concrete-class dispatch ladders with one semantic visual-widget contract and split the 1,286-line block factory along real presentation-policy seams.

## Architecture report evidence (candidates 3 + 4)

### Concrete dispatch ladder
`TranscriptMountedWidget::bind()` dispatches across `StreamingMarkdownTranscriptWidget`, `ToolExchangeTranscriptWidget`, `QuestionTranscriptWidget`, `SubagentTranscriptWidget`, `WelcomeTranscriptWidget`, `TurnSeparatorWidget`, and `TextWidget` via `instanceof`. `createAndBind()` repeats the type checks. This is a hand-written router over classes that already expose an `apply(TranscriptVisualNode)`-shaped operation.

### Mega-factory
`TranscriptBlockWidgetFactory` is ~1,286 lines with ~50 private methods and eight renderer collaborators. It combines:
- block-kind dispatch,
- tool call/result pairing and standalone-result suppression,
- preview truncation policy,
- YAML-like argument rendering,
- hotkey/question rendering,
- specialized tool paths (subagent, view-image, skill-read, edit, write, generic).
The public interface is small, but unrelated policies cannot be tested or changed independently.

## Existing architecture constraints
Preserve the archived semantic transcript work: `TranscriptVisualProjector`/patches own presentation projection and incremental identity; `TranscriptMountedWidget` remains the keyed mounted adapter; high-frequency content updates stay O(changes); low-frequency structural policy may retain its documented O(B) ceiling. Do not recreate removed all-in-one renderers, fingerprints, caches, or alternate transcript paths.

## Smallest viable direction
- Introduce one narrow typed contract implemented by mutable semantic visual widgets so mounted binding does not know every concrete class.
- Keep generic native `TextWidget` handling explicit if wrapping it would add more code than one contained special case.
- Extract only coherent factory responsibilities: tool-exchange pairing/suppression policy and per-kind/specialized rendering groups. Do not create one renderer/interface per trivial label or one-use method.
- Keep dispatch typed; avoid replacing the `instanceof` ladder with string `supports()` routers or service-locator indirection unless it demonstrably reduces code.

## Scope boundaries
- No transcript visuals, ordering, grouping, suppression, preview, theme, hotkey, question, tool, streaming, replay, or stable-key behavior changes.
- No viewport, pagination, manual rendered-line cache, feature flag, compatibility renderer, or ScreenWriter work.
- Do not mix session scoping or dual top-level widget migration into this task.

## Test thesis
Use existing virtual mounted-transcript tests to prove semantic widget identity, in-place updates, structural patches, tool pairing/suppression, and unchanged output. Tests should target extracted policy boundaries rather than mirror private methods. Tmux remains integration smoke, not the primary proof.

## Acceptance criteria
- `TranscriptMountedWidget::bind/createAndBind` no longer duplicate concrete semantic-widget dispatch ladders.
- Mutable semantic transcript widgets share one narrow typed update contract; trivial native widgets are not wrapped without net simplification.
- `TranscriptBlockWidgetFactory` is materially smaller and delegates tool pairing/suppression plus coherent rendering groups to focused owners.
- Existing incremental patch ownership, stable keys/widget identity, high-frequency O(changes) path, and documented low-frequency O(B) ceiling remain intact.
- All transcript visual, pairing, suppression, ordering, preview, theme, question, tool, replay, and streaming behavior is preserved.
- No one-class-per-label scaffolding, generic renderer framework, service locator, compatibility path, or manual render cache is introduced.
- The fork and handoff state that `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` were read and followed.
- Focused virtual transcript tests, `castor test`, `castor test:controller-replay`, `castor test:tui`, `castor deptrac`, `castor phpstan`, `castor cs-check`, and final `castor check` pass.

## Workflow metadata
Status: ARCHIVE
Branch: task/tui-03-deepen-transcript-widget-architecture
Worktree: /home/ineersa/projects/agent-core-worktrees/tui-03-deepen-transcript-widget-architecture
Fork run: u1dg0nzyj4d0
PR URL: https://github.com/ineersa/agent-core/pull/399
PR Status: merged
Started: 2026-08-17T00:40:03.051Z
Completed: 2026-08-17T02:20:09.846Z

## Work log
- Created: 2026-08-15T23:16:16.557Z

## Task workflow update - 2026-08-15T23:19:03.251Z
- Full architect report: [`/home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md`](file:///home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md) — TUI report, candidates 3 and 4.

## Task workflow update - 2026-08-17T00:40:03.051Z
- Moved TODO → IN-PROGRESS.
- Created branch task/tui-03-deepen-transcript-widget-architecture.
- Created worktree /home/ineersa/projects/agent-core-worktrees/tui-03-deepen-transcript-widget-architecture.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/tui-03-deepen-transcript-widget-architecture.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/tui-03-deepen-transcript-widget-architecture.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/tui-03-deepen-transcript-widget-architecture.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/tui-03-deepen-transcript-widget-architecture.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/tui-03-deepen-transcript-widget-architecture/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/tui-03-deepen-transcript-widget-architecture.
- Summary: Starting tui-03 implementation phase: deepen transcript widget architecture while preserving all user-visible transcript behavior and incremental patch semantics.

## Task workflow update - 2026-08-17T00:47:36.978Z
- Summary: Three read-only scouts mapped the change. Candidate 3: four mutable semantic widgets already share apply(TranscriptVisualNode); one narrow typed contract can collapse both mounted-widget ladders while leaving Welcome/TurnSeparator immutable and native TextWidget explicit. Candidate 4: minimum coherent split is a tool pairing/suppression policy owner, a specialized tool renderer, and a non-tool block renderer, with TranscriptBlockWidgetFactory retained as the small compatibility facade and no Deptrac changes. Existing virtual mounted/output tests already cover identity, in-place updates, structural changes, pairing/suppression, previews, and streaming; add only one stable structural-patch characterization if needed, with no implementation-existence tests or new tmux/controller cases. Implementation will be split into bounded sequential forks to avoid another sprawling high-token run.

## Task workflow update - 2026-08-17T00:47:53.837Z
- Recorded fork run: 67xh287bj4gk
- Summary: Launched bounded implementation phase 1: introduce the narrow mutable semantic transcript-widget contract and collapse TranscriptMountedWidget's duplicate bind/createAndBind ladders only. Factory decomposition intentionally deferred to dependent phases.

## Task workflow update - 2026-08-17T00:51:21.306Z
- Recorded fork run: 67xh287bj4gk
- Summary: Phase-1 fork ended without a usable handoff or commit. The worktree contains the intended 7-file uncommitted diff: new MutableTranscriptWidget contract, four implementations, collapsed mounted dispatch, and one wrong-kind stable-key behavioral regression test. Git diff --check is clean. A narrow continuation fork is required to inspect minimality, validate, and commit; no work was lost.

## Task workflow update - 2026-08-17T00:51:40.209Z
- Recorded fork run: 9qrxj87cldhn
- Summary: Launched narrow phase-1 recovery fork to inspect the existing diff, enforce minimality/test standards, run focused Castor validation, and produce a verified clean commit only.

## Task workflow update - 2026-08-17T00:54:22.865Z
- Recorded fork run: 9qrxj87cldhn
- Summary: Recovery fork validated phase 1 cleanly but again exited before committing; worktree remains at base HEAD with the same uncommitted 7-file diff. Launching a commit-only recovery with no edits/tests.

## Task workflow update - 2026-08-17T00:55:44.506Z
- Recorded fork run: 9qrxj87cldhn
- Summary: Phase 1 committed cleanly as 96d1ea4b7216c382850f8a4f771da8817351741a (`refactor(tui): unify semantic transcript widget updates`), 7 files +107/-38. MutableTranscriptWidget now owns kind/apply; four mutable semantic widgets implement it; TranscriptMountedWidget has one contract branch while native TextWidget and immutable widgets remain explicit. One public mounted-behavior test preserves wrong-kind stable-key re-creation. Recovery fork reported focused test/PHPStan/CS/diff validation clean; commit-only fork verified SHA/type and clean status.

## Task workflow update - 2026-08-17T00:56:03.509Z
- Recorded fork run: lamagf0jrnuc
- Summary: Launched bounded phase 2: extract one tool pairing/suppression/result-presentation policy owner while preserving the factory constructor and public callers. Renderer extraction remains deferred.

## Task workflow update - 2026-08-17T01:00:26.482Z
- Recorded fork run: lamagf0jrnuc
- Summary: Phase 2 completed at verified commit 92f88bd0881768660ff5a4afb296b76c2a90beee. Added TranscriptToolPresentationPolicy and reduced TranscriptBlockWidgetFactory 1286→1083 lines while preserving its constructor/public forwarders. Existing 68 virtual tests (769 assertions), scoped PHPStan, CS, syntax, and diff checks passed; worktree clean.

## Task workflow update - 2026-08-17T01:00:48.857Z
- Recorded fork run: ocp61bx5bk9k
- Summary: Launched bounded phase 3: extract one concrete TranscriptToolRenderer and move only the complete specialized tool-rendering cluster. Non-tool rendering remains deferred to the final extraction phase.

## Task workflow update - 2026-08-17T01:09:16.626Z
- Recorded fork run: ocp61bx5bk9k
- Summary: Phase 3 completed at verified commit b75c5810d678a4c535af9dea4ffd8d9a828c8f72. Added one concrete TranscriptToolRenderer (599 lines) and reduced TranscriptBlockWidgetFactory 1082→539 lines; exact constructor/API and subagent fallback preserved. Focused 68 tests/769 assertions passed twice, Deptrac 0 violations, scoped PHPStan 0 errors, CS/diff/syntax clean; worktree clean.

## Task workflow update - 2026-08-17T01:09:37.500Z
- Recorded fork run: u1dg0nzyj4d0
- Summary: Launched final bounded phase 4: extract one concrete non-tool TranscriptBlockRenderer, reduce the factory to orchestration/forwarders, then run the complete implementation-phase Castor battery except final castor check.

## Task workflow update - 2026-08-17T01:17:14.585Z
- Recorded fork run: u1dg0nzyj4d0
- Validation: Mandatory testing skill and tests/AGENTS.md read and followed by every test-running fork.; Focused virtual transcript suite: PASS (68 tests, 769 assertions).; castor test: PASS (4493 tests, 17688 assertions).; castor test:controller-replay: PASS (12 tests, 165 assertions).; castor test:tui: PASS (38 tests, 295 assertions; replay-backed real tmux).; castor deptrac: PASS (0 violations).; castor phpstan (scoped during phases, full final phase as instructed): PASS (0 errors).; castor cs-check: PASS (0 files fixed).; castor clean:cleanup:workers:list: no stale QA workers.; git diff --check clean; commit d354d7da24d52d89371f8573bf3495b8c7b6e950 verified via git cat-file; worktree clean.; Final castor check intentionally not run during task-start; task-to-pr owns that gate.
- Summary: Implementation complete at verified clean HEAD d354d7da24d52d89371f8573bf3495b8c7b6e950 across four bounded commits. TranscriptMountedWidget now uses one MutableTranscriptWidget contract; TranscriptBlockWidgetFactory is reduced from 1,286 to 182 lines and delegates to TranscriptToolPresentationPolicy, TranscriptToolRenderer, and TranscriptBlockRenderer. Constructor/public API, visual output, stable keys, incremental patch ownership, pairing/suppression, previews, themes, question/system/tool/streaming/replay behavior remain unchanged. Cumulative diff: 11 files +1453/-1186; only test growth is one wrong-kind stable-key mounted behavior case.

## Task workflow update - 2026-08-17T01:37:17.658Z
- Summary: Task-to-PR reviewer returned REQUEST CHANGES with one blocker only: TranscriptBlockWidgetFactory's class docblock still claims moved rendering responsibilities and references unimported MarkdownWidget, resolving to a nonexistent same-namespace class. All architecture, behavior, specification fidelity, ownership, by-reference semantics, contract, identity/performance paths, and virtual proof were otherwise approved. Smallest fix: delete the stale responsibility paragraph; retain only facade/delegation documentation.

## Task workflow update - 2026-08-17T01:43:17.446Z
- Validation: Reviewer final verdict: APPROVED at HEAD 03fd8e79424edc5a145e6234c0bc0006badac463.; castor test: PASS (4493 tests, 17688 assertions).; castor deptrac: PASS (0 violations).; castor phpstan: PASS (0 errors).; castor cs-check: PASS (0 files fixed).; castor test:tui: PASS (38 tests, 297 assertions; replay-backed real tmux).; castor clean:cleanup:workers:list: no stale QA workers.; git diff --check clean; worktree clean.
- Summary: Task-to-PR review cycle complete. Initial reviewer REQUEST CHANGES on one stale/unresolved factory PHPDoc reference; fixed by docs-only commit 03fd8e79424edc5a145e6234c0bc0006badac463 (+6/-17), then targeted re-review APPROVED. Reviewer confirmed architecture/specification fidelity, no behavior drift, correct typed contract, stable-key identity/O(changes), coherent three-owner split, preserved pairing/suppression/by-ref semantics, no external surface changes, and appropriate virtual proof.

## Task workflow update - 2026-08-17T01:45:35.687Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (125.8s).
- Pushed task/tui-03-deepen-transcript-widget-architecture to origin.
- branch 'task/tui-03-deepen-transcript-widget-architecture' set up to track 'origin/task/tui-03-deepen-transcript-widget-architecture'.
- Created PR: https://github.com/ineersa/agent-core/pull/399
- Summary: Reviewer APPROVED after one documentation-only fix. Focused Castor lanes and replay-backed real tmux passed on clean HEAD 03fd8e79424edc5a145e6234c0bc0006badac463. Running deterministic castor check, then pushing and opening the PR.

## Task workflow update - 2026-08-17T02:05:28.073Z
- Summary: Post-PR architect audit of PR #399 verdict: PARTIAL/MOSTLY RELOCATION. Candidate 3 is a real improvement (one MutableTranscriptWidget contract replaces two duplicated four-class dispatch ladders and owns wrong-kind rebinding). Candidate 4 improved naming/readability and extracted pairing/suppression into a genuine policy owner, but preserved the exact factory constructor/public API/call graph; the three new collaborators are private, hard-wired, absent from direct tests, and TranscriptToolRenderer (599 lines) plus TranscriptBlockRenderer (409 lines) are mostly verbatim method relocation. Architect still recommends MERGE because this satisfies the finalized behavior-preserving decomposition task and the renderer classes have small cohesive interfaces (3 and 1 public entry points), while clearly describing it as decomposition rather than deep architectural change. Suggested follow-up only if desired: separate result-presentation facts from pairing policy, remove dead facade surface, and add one policy-boundary behavior test; avoid per-tool classes/generic renderer frameworks.

## Task workflow update - 2026-08-17T02:20:09.846Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/tui-03-deepen-transcript-widget-architecture.
- Merged task/tui-03-deepen-transcript-widget-architecture into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Transcript/MutableTranscriptWidget.php     |   36 +
 src/Tui/Transcript/QuestionTranscriptWidget.php    |    7 +-
 .../StreamingMarkdownTranscriptWidget.php          |    7 +-
 src/Tui/Transcript/SubagentTranscriptWidget.php    |    7 +-
 .../Transcript/ToolExchangeTranscriptWidget.php    |    7 +-
 src/Tui/Transcript/TranscriptBlockRenderer.php     |  409 +++++++
 .../Transcript/TranscriptBlockWidgetFactory.php    | 1203 +-------------------
 src/Tui/Transcript/TranscriptMountedWidget.php     |   37 +-
 .../TranscriptToolPresentationPolicy.php           |  294 +++++
 src/Tui/Transcript/TranscriptToolRenderer.php      |  599 ++++++++++
 .../Tui/Screen/TuiMountedTranscriptVirtualTest.php |   44 +
 11 files changed, 1453 insertions(+), 1197 deletions(-)
 create mode 100644 src/Tui/Transcript/MutableTranscriptWidget.php
 create mode 100644 src/Tui/Transcript/TranscriptBlockRenderer.php
 create mode 100644 src/Tui/Transcript/TranscriptToolPresentationPolicy.php
 create mode 100644 src/Tui/Transcript/TranscriptToolRenderer.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/tui-03-deepen-transcript-widget-architecture.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/tui-03-deepen-transcript-widget-architecture.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #399 confirmed merged on GitHub at 2026-08-17T02:19:42Z with merge commit 27e354026f10b82841a9fe09e0d065b94e3dace6. Closing task and cleaning worktree.

## Task workflow update - 2026-08-17T02:22:48.097Z
- Validation: LLM_MODE=true castor check: PASS, QA run qa-20260817-022014-950239-38d9be2f.; deptrac PASS; unit/integration PASS (4492 tests, 17679 assertions); controller replay PASS (12 tests, 165 assertions); TUI replay/tmux PASS (38 tests, 301 assertions); llm-real PASS (13 tests, 144 assertions); phpstan PASS (0 errors); cs-check PASS; docs:validate PASS.; QA artifact integrity PASS; QA leak check PASS; llama-proxy cache guard stable 326→326.; castor clean:cleanup:workers:list: no stale QA workers.
- Summary: Post-merge validation complete on integration checkout. PR #399 merged, task worktree and IDEA exclusions removed, integration checkout synced. Follow-up architecture task remains TODO as tui-04-deepen-transcript-renderer-boundaries.

## Task workflow update - 2026-08-18T00:06:36.255Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
