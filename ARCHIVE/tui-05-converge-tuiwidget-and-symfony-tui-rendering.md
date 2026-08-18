# tui-05: Converge TuiWidget and Symfony TUI rendering

## Goal
## Goal
Retire the remaining internal line-array rendering bridge in favor of mounted Symfony TUI widgets, but only after proving the current ExtensionApi compatibility surface. This is intentionally the last/lowest-priority TUI architecture task.

## Architecture report evidence (candidate 9)
- Internal `TuiWidget` renders `list<string>` through `TuiRenderContext`.
- `ChatScreen` wraps these renderers in `LiveTextWidget` producer closures and `implode()` bridges while transcript, editor, overlays, pickers, completion, Markdown, loader, keybindings, and styles already use native Symfony TUI widgets.
- Remaining line renderers include `HeaderWidget`, `StatusPanelWidget`, `PendingMessagesWidget`, `LoadedResourcesWidget`, `CompactHeaderWidget`, and `FooterBarWidget`.
- `CompactHeaderWidget::wrapLabel()` maintains custom ANSI-aware column wrapping while Symfony `AnsiUtils`/`TextWrapper` primitives already exist elsewhere.
- The hybrid model creates two rendering mental models and makes parts of `TuiRenderContext` loose/dead.

## Prior task history
Archived PR #386 deliberately deferred internal `TuiWidget`/`LiveTextWidget` migration after proving the public ExtensionApi accepts Symfony `AbstractWidget`; it converted semantic hotkey/subagent surfaces and retained simple labels/native paths. Earlier mounted-transcript work already moved transcript rendering into a first-class Symfony subtree. Re-read those decisions and current references; do not redo completed transcript/widget work.

## Required decision before implementation
Inventory all production and `.hatfield/extensions` references and classify `TuiWidget`, `TuiRenderContext`, slot adapters, and custom extension widgets as internal, public compatibility surface, or dead. If removal changes a published `ExtensionApi` contract or third-party extension behavior, obtain explicit user approval and a documented deprecation decision before editing. No automatic compatibility shim.

## Smallest viable direction
- Migrate only the six remaining meaningful line renderers to direct `AbstractWidget`/native widget composition when each migration deletes its `LiveTextWidget` adapter and reduces total code.
- Use Symfony `AnsiUtils`/text wrapping/style primitives instead of copied width math where output remains exact.
- Delete obsolete bridge/context APIs after references are gone.
- Keep plain static text as `TextWidget`; do not create a custom class for every line.

## Scope boundaries
- No transcript architecture, input pipeline, viewport, ScreenWriter, terminal scrolling, theme appearance, layout order, extension behavior, or user-visible text changes.
- No dual renderer, feature flag, adapter compatibility layer, or one-widget-per-label scaffolding.
- If native migration increases glue or loses extension functionality, record KEEP and stop rather than forcing completion.

## Test thesis
Use virtual rendering for exact layout/order/style/width/resize behavior and existing extension slot tests for compatibility. Minimal tmux proof covers only real terminal integration. Compare output and resize behavior rather than class existence.

## Acceptance criteria
- All `TuiWidget`, `TuiRenderContext`, `LiveTextWidget`, ChatScreen bridge, and extension references are inventoried before implementation.
- Public ExtensionApi and installed extension compatibility is explicitly proven or a user-approved compatibility decision is recorded.
- Remaining eligible header/status/pending/resources/compact-header/footer regions mount native Symfony TUI widgets directly and delete more bridge code than they add.
- ANSI width/wrapping/style logic reuses Symfony primitives when output is equivalent; plain static text remains `TextWidget`.
- Obsolete internal line-rendering/context APIs and producer closures are deleted after semantic reference proof.
- No transcript rewrite, compatibility renderer, feature flag, layout/visual change, or shallow widget proliferation is introduced.
- The fork and handoff state that `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` were read and followed.
- Focused virtual/extension tests, `castor test`, `castor test:tui`, `castor deptrac`, `castor phpstan`, `castor cs-check`, and final `castor check` pass.

## Workflow metadata
Status: ARCHIVE
Branch: task/tui-05-converge-tuiwidget-and-symfony-tui-rendering
Worktree: /home/ineersa/projects/agent-core-worktrees/tui-05-converge-tuiwidget-and-symfony-tui-rendering
Fork run: m54repgf7du0
PR URL: https://github.com/ineersa/agent-core/pull/403
PR Status: merged
Started: 2026-08-17T18:02:08.512Z
Completed: 2026-08-18T00:04:53.545Z

## Work log
- Created: 2026-08-15T23:16:16.558Z

## Task workflow update - 2026-08-15T23:19:03.250Z
- Full architect report: [`/home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md`](file:///home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md) — TUI report, candidate 9.

## Task workflow update - 2026-08-17T18:02:08.512Z
- Moved TODO → IN-PROGRESS.
- Created branch task/tui-05-converge-tuiwidget-and-symfony-tui-rendering.
- Created worktree /home/ineersa/projects/agent-core-worktrees/tui-05-converge-tuiwidget-and-symfony-tui-rendering.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/tui-05-converge-tuiwidget-and-symfony-tui-rendering.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/tui-05-converge-tuiwidget-and-symfony-tui-rendering.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/tui-05-converge-tuiwidget-and-symfony-tui-rendering.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/tui-05-converge-tuiwidget-and-symfony-tui-rendering.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/tui-05-converge-tuiwidget-and-symfony-tui-rendering/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/tui-05-converge-tuiwidget-and-symfony-tui-rendering.
- Summary: Starting final bounded TUI rendering convergence task. First gate is read-only inventory/classification of all TuiWidget/TuiRenderContext/LiveTextWidget/slot/extension references and archived PR #386 decisions; implementation will stop for user approval if any published ExtensionApi or installed-extension behavior must change.

## Task workflow update - 2026-08-17T18:15:04.586Z
- Validation: Three read-only scouts completed; IDE index ready; semantic references collected with no truncation for TuiWidget/TuiRenderContext/LiveTextWidget; Testing skill and tests/AGENTS read by proof scout; No edits/tests performed during inventory
- Summary: Required pre-implementation compatibility gate passed. Semantic IDE inventory: TuiWidget 51 project refs / six internal implementors; TuiRenderContext 40 refs; LiveTextWidget only ChatScreen plus one test; no `.hatfield/extensions` production source references any of them. Published ExtensionApi already uses Symfony AbstractWidget and BridgeTuiExtensionContext; installed file-rewind uses native Container/Text/SelectList, observational-memory uses public setStatus/onTick. No public API or installed-extension behavior change is required, so no user approval blocker. Migration plan: convert six internal line renderers to native widgets, direct-mount compact header, delete obsolete internal widget slot methods/types/context bridge pieces after refs, retain LiveTextWidget only for blank margin/dynamic separators where TextWidget is not equivalent, preserve public bridge unchanged.
- Classification: internal active — six renderers, TuiWidget/TuiRenderContext/LiveTextWidget bridge; internal dead — header/footer/editor replacement methods and widget placement APIs after compact-header direct mount; public preserved — ExtensionApi AbstractWidget bridge; KEEP — LiveTextWidget for whitespace margin and responsive separators, CompactHeader/Footer custom atomic packing using Symfony AnsiUtils because TextWrapper is not equivalent.

## Task workflow update - 2026-08-17T18:16:05.271Z
- Recorded fork run: ls69vb4a18k5
- Summary: Launched single implementation fork on exact worktree. Scope: native AbstractWidget migration for six chrome regions; delete TuiWidget/TuiRenderContext and internal widget slot bridge after zero refs; direct-mount compact header; preserve public ExtensionApi/Bridge and TUI-04 input pipeline; bounded KEEP for LiveTextWidget blank margin/three separators and custom compact/footer packing; behavior-first virtual proofs plus mandatory replay-backed real TmuxHarness journey. Fork owns full focused/full Castor lanes except final castor check, then one verified clean commit.

## Task workflow update - 2026-08-17T18:43:51.290Z
- Recorded fork run: ls69vb4a18k5
- Validation: Fork states `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` read/followed before test work and validation; Focused widget/extension/screen: PASS 53 tests / 500 assertions; All TUI unit/virtual: PASS 887 tests / 3831 assertions; castor test: PASS 4543 tests / 18096 assertions; Focused replay-backed tmux group: PASS 5 tests / 58 assertions; castor test:tui: PASS 38 tests / 311 assertions; castor deptrac: PASS 0 violations (2283 uncovered, 2612 allowed); castor phpstan: PASS 0 errors; castor cs-check: PASS 0 files fixed after targeted cs-fix; git diff --check: clean; castor clean:cleanup:workers:list: no stale workers; Not run by design: castor check, test:llm-real
- Summary: Implementation complete at verified commit 3fcaaf69ae3116d03b26f81015f3cf054b8fbc74 (`refactor(tui): converge chrome on native widgets`), clean worktree. 28 files +601/-883 (net -282; fork reports production/config net -266), 5 deletions, no new files. Six chrome regions now extend/mount Symfony AbstractWidget directly; ChatScreen line-array adapters and duplicate renderable fields removed; compact header direct-mounted; obsolete TuiWidget/TuiRenderContext/WidgetPlacementEnum and internal widget-slot order/replacement APIs deleted. LiveTextWidget remains only for blank top margin and three responsive separators. Public ExtensionApi, BridgeTuiExtensionContext, installed extension production source, transcript/input/viewport behavior unchanged. Mandatory real replay-backed TuiJourneyE2eTest now submits/interacts and verifies chrome structure/order after assistant output. Parent independently verified SHA/type/clean tree, expected diff/deletions, and no `.hatfield/extensions` diff. Legacy symbol search has no code references; only the intentional LiveTextWidget docblock names the deleted bridge as historical rationale.
- Verified commit with git rev-parse HEAD and git cat-file -t HEAD; branch worktree clean.
- Compatibility proof: git diff for `.hatfield/extensions` is empty; Bridge/public ExtensionApi untouched; file-rewind overlay and observational-memory status paths preserved by fork validation.
- Review note for task-to-pr: sanity-check six narrow deptrac additions, especially TuiScreen -> TuiCompactHeader. Fork reports it is required by direct mounting and matches existing TuiScreen -> other chrome-owner edges.
- KEEP rationales: LiveTextWidget only where Symfony TextWidget drops whitespace-only margin or lacks exact responsive separator behavior; CompactHeader/Footer retain AnsiUtils atomic packing because TextWrapper would word-wrap and change layout.

## Task workflow update - 2026-08-17T21:11:34.891Z
- Summary: Task-to-PR reviewer verdict at 3fcaaf69a: REQUEST CHANGES. Blocking bug: native FooterBarWidget lost old LiveTextWidget(truncate:true) outer width guarantee, so columns <=11 can emit over-wide lines and Symfony Renderer throws RenderException. Blocking stale docs: root AGENTS.md and docs/tui-architecture.md still describe deleted TuiWidget/widget slots; ChatScreen overlay order docblocks still name deleted above/belowEditorWidget. Reviewer approved all other migration decisions, compatibility proof, KEEP rationales, direct compact-header ownership/deptrac edge, status sync, overlay behavior, and virtual+real-tmux test pyramid.
- Reviewer read/followed testing skill and tests/AGENTS; read-only review against base a31f499f9.
- Fix plan: restore final AnsiUtils truncation to terminal columns for every FooterBarWidget return path; add pathological narrow-width regression proof; update stale architecture/overlay docs; remove only stale TuiWidget deptrac allow-edges from layers with zero remaining Widget dependency. No production/API/layout changes beyond restoring prior width safety.

## Task workflow update - 2026-08-17T21:11:59.694Z
- Recorded fork run: m54repgf7du0
- Summary: Launched bounded review-fix fork for FooterBarWidget narrow-width truncation regression, one failing-before pathological-width test, stale architecture/overlay docs, and stale TuiWidget deptrac allow-edge deletion only. No public API/extension/input/layout changes; focused Castor validation and one commit, then targeted re-review.

## Task workflow update - 2026-08-17T22:14:38.370Z
- Recorded fork run: m54repgf7du0
- Validation: Fork states testing skill + tests/AGENTS read/followed; castor test --filter=FooterBarWidgetTest: PASS 7 tests / 93 assertions; failing-before proven with pre-fix widget (RenderException width 12 vs 4 cols), then passing after restore; castor deptrac: PASS 0 violations 0 errors (allowed flat at 2612); castor phpstan: PASS 0 errors; castor cs-check: PASS 0 files fixed; castor docs:validate: PASS; git diff --check: clean; castor clean:cleanup:workers:list: no stale workers; Parent verified SHA/type/clean tree; rg sweep shows no code refs to deleted symbols beyond intentional LiveTextWidget docblock and depfile layer definition
- Summary: Review-fix complete at verified commit 3a44ded73f343cc8d578af3a57fd099c5436af45 (`fix(tui): preserve narrow footer rendering`), clean worktree, 6 files +62/-23. Blockers resolved: (1) FooterBarWidget restores exact old LiveTextWidget(truncate:true) outer width guarantee via AnsiUtils::truncateToWidth to getColumns() on both return paths — byte-identical at fitting widths, ellipsis behavior matches old bridge; regression test proven failing-before with the exact predicted RenderException (width 12 vs 4 columns) and passing after; (2) root AGENTS.md + docs/tui-architecture.md now describe native AbstractWidget chrome and surviving status/working/footer/input surfaces, no deleted symbols; (3) ChatScreen overlay docblocks rewritten to actual mount order (comments only); (4) 11 stale TuiWidget deptrac allow-edges removed (9 named + TuiApplication/AppCli also mechanically proven zero-usage; parent adjudicated acceptable), 4 kept for real users (Screen/Listener/Question/Picker), 0 violations with allowed-count flat.

## Task workflow update - 2026-08-17T22:22:51.101Z
- Validation: Reviewer (read/followed testing skill + tests/AGENTS): APPROVED at 3a44ded73; castor test: PASS 4544 tests / 18150 assertions (35.2s); castor deptrac: PASS 0 violations, 0 errors (2285 uncovered, 2612 allowed); castor phpstan: PASS 0 errors; castor cs-check: PASS 0 files fixed; castor test:tui: PASS 38 tests / 317 assertions (121.9s real tmux); castor clean:cleanup:workers:list: no stale workers; git diff --check: clean; worktree clean at verified HEAD
- Summary: Targeted re-review at HEAD 3a44ded73 returned APPROVED: all four blockers resolved (footer outer truncation byte-identical restoration with failing-before RenderException proof; docs/AGENTS/overlay-docblocks accurate with zero deleted symbols; 11 stale deptrac edges mechanically confirmed zero-usage; narrow-width regression test at correct virtual layer). Reviewer found no scope creep, no new API/abstraction, ExtensionApi/extensions/Bridge untouched. One NTH deferred as pre-existing out-of-scope cleanup (redundant inner footer truncation guards, ~8 lines). Parent-adjudicated extra TuiApplication/AppCli edge removals accepted by reviewer (no evidence needed). Focused validation re-run by parent at same HEAD: all green.

## Task workflow update - 2026-08-17T22:25:19.569Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (133.4s).
- Pushed task/tui-05-converge-tuiwidget-and-symfony-tui-rendering to origin.
- branch 'task/tui-05-converge-tuiwidget-and-symfony-tui-rendering' set up to track 'origin/task/tui-05-converge-tuiwidget-and-symfony-tui-rendering'.
- Created PR: https://github.com/ineersa/agent-core/pull/403

## Task workflow update - 2026-08-18T00:04:53.545Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/tui-05-converge-tuiwidget-and-symfony-tui-rendering.
- Merged task/tui-05-converge-tuiwidget-and-symfony-tui-rendering into integration checkout.
- Auto-merging depfile.yaml
Merge made by the 'ort' strategy.
 AGENTS.md                                          |   4 +-
 depfile.yaml                                       |  17 +-
 docs/tui-architecture.md                           |   7 +-
 src/Tui/CompactHeader/CompactHeaderWidget.php      |  24 ++-
 src/Tui/Extension/SlotBasedTuiExtensionContext.php |  40 +---
 src/Tui/Extension/TuiExtensionContext.php          |  45 +---
 src/Tui/Footer/FooterBarWidget.php                 |  33 ++-
 src/Tui/Header/HeaderWidget.php                    |  24 ++-
 src/Tui/Layout/TuiSlotRegistry.php                 | 102 +--------
 src/Tui/Listener/CompactHeaderRegistrar.php        |  39 ++--
 src/Tui/Screen/ChatScreen.php                      | 232 ++++++---------------
 src/Tui/Startup/LoadedResourcesWidget.php          |  32 ++-
 src/Tui/Status/StatusPanelWidget.php               |  32 +--
 src/Tui/Transcript/PendingMessagesWidget.php       |  24 ++-
 src/Tui/Widget/LiveTextWidget.php                  |   9 +
 src/Tui/Widget/TuiRenderContext.php                |  22 --
 src/Tui/Widget/TuiWidget.php                       |  30 ---
 src/Tui/Widget/WidgetPlacementEnum.php             |  17 --
 .../CompactHeader/CompactHeaderPinnedOrderTest.php |  78 -------
 .../Tui/CompactHeader/CompactHeaderWidgetTest.php  |  63 ++++--
 tests/Tui/E2E/TuiJourneyE2eTest.php                |  72 +++++++
 .../Extension/SlotBasedTuiExtensionContextTest.php |  85 ++------
 tests/Tui/Footer/FooterBarWidgetTest.php           | 117 +++++++++--
 tests/Tui/Layout/TuiSlotRegistryPriorityTest.php   |  90 --------
 tests/Tui/Layout/TuiSlotRegistryTest.php           |  96 ---------
 tests/Tui/Listener/CompactHeaderRegistrarTest.php  |  17 +-
 tests/Tui/Listener/FooterStateListenerTest.php     |   6 +-
 tests/Tui/Screen/TuiStartupVirtualRenderTest.php   | 108 ++++++++++
 tests/Tui/Startup/LoadedResourcesWidgetTest.php    |  68 +++++-
 .../Tui/Transcript/SubagentResultRendererTest.php  |  34 ++-
 30 files changed, 662 insertions(+), 905 deletions(-)
 delete mode 100644 src/Tui/Widget/TuiRenderContext.php
 delete mode 100644 src/Tui/Widget/TuiWidget.php
 delete mode 100644 src/Tui/Widget/WidgetPlacementEnum.php
 delete mode 100644 tests/Tui/CompactHeader/CompactHeaderPinnedOrderTest.php
 delete mode 100644 tests/Tui/Layout/TuiSlotRegistryPriorityTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/tui-05-converge-tuiwidget-and-symfony-tui-rendering.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/tui-05-converge-tuiwidget-and-symfony-tui-rendering.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: castor check passed in worktree (133.4s) at CODE-REVIEW transition; Full focused battery green at 3a44ded73: castor test 4544/18150, test:tui 38/317, deptrac 0 violations, phpstan 0, cs-check 0, docs:validate pass
- Summary: PR #403 merged. tui-05 complete: six chrome regions migrated to native Symfony AbstractWidgets, TuiWidget/TuiRenderContext/WidgetPlacementEnum and dead slot APIs deleted (net ~-283 lines), footer narrow-width crash fixed with failing-before regression proof, docs/deptrac cleaned. Reviewer APPROVED after fixes; castor check passed (133.4s).

## Task workflow update - 2026-08-18T00:06:36.267Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.

## Task workflow update - 2026-08-18T00:08:24.650Z
- Summary: Post-merge validation on integration checkout: LLM_MODE=true castor check passed all 8 lanes (deptrac, test 4580/18247, controller-replay 12/165, test:tui 38/313, llm-real 13/144, phpstan 0, cs-check, docs:validate; 399.8s, no leaked workers, proxy cache stable 348→348). Integration pushed to origin main; worktree removed. Task closed.
