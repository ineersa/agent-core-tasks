# remove-dead-registry-status-mutation-seam

## Goal
Follow-up to PR #403 (tui-05): after native widget migration, TuiSlotRegistry status/working mutators are silent no-ops visually — only ChatScreen::setStatus()/setWorkingMessage() sync the native StatusPanelWidget/LoaderWidget (registry has no per-render producer anymore). Production routes correctly; the hazard is future contributors/tests writing registry->setStatus() directly and nothing painting (no error).

Root-cause fix = delete the lying API, not add sync machinery:
- ChatScreen becomes sole owner/writer of status entries (and evaluate the same for working message): setStatus mutates state and pushes setEntries() to StatusPanelWidget; screen-level read getter for tests replacing registry->getStatusEntries() reads.
- TuiSlotRegistry drops setStatus/getStatusEntries (and setWorkingMessage if moved; getWorkingMessage read used by UsageCommandHandler.php:99 and TuiUsageCommandVirtualTest can move to a screen getter or stay if working state remains registry-owned — fork decides by smallest honest seam).
- Mechanical test updates: ~20 test files call $screen->registry()->getStatusEntries() (SubagentLiveAttentionTest, ResumeSession/RenameSessionCommandHandlerTest, SessionPickerControllerTest, QuestionControllerTest, AgentsMainCommandHandlerTest, TickPollListenerTest, SubagentLiveHitlScenarioTest, TickPollListenerChildHitlTest, SubmitListenerReasoningNoticeClearTest — the latter writes registry->setStatus directly then calls refresh(); replace with screen->setStatus and delete the refresh() safety net if it becomes dead).
- Keep input-handler registry paths (real native listeners) untouched.

Scope guards: no new abstraction/callback/sync layer; no behavior or layout change; net-negative or neutral production diff expected. Not part of PR #403 — do after merge.

## Acceptance criteria
- TuiSlotRegistry exposes no status/working mutation that silently fails to paint; ChatScreen is the sole writer
- Registry direct-writers in tests updated to the screen seam; refresh() safety-net hack removed if it becomes dead
- All focused castor test + test:tui lanes green; no production behavior/layout change; public ExtensionApi untouched

## Workflow metadata
Status: ARCHIVE
Branch: task/remove-dead-registry-status-mutation-seam
Worktree: /home/ineersa/projects/agent-core-worktrees/remove-dead-registry-status-mutation-seam
Fork run: zkxdutbfw7fb
PR URL: https://github.com/ineersa/agent-core/pull/404
PR Status: merged
Started: 2026-08-18T00:09:30.581Z
Completed: 2026-08-18T03:49:12.561Z

## Work log
- Created: 2026-08-18T00:02:19.642Z

## Task workflow update - 2026-08-18T00:09:30.581Z
- Moved TODO → IN-PROGRESS.
- Created branch task/remove-dead-registry-status-mutation-seam.
- Created worktree /home/ineersa/projects/agent-core-worktrees/remove-dead-registry-status-mutation-seam.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/remove-dead-registry-status-mutation-seam.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/remove-dead-registry-status-mutation-seam.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/remove-dead-registry-status-mutation-seam.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/remove-dead-registry-status-mutation-seam.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/remove-dead-registry-status-mutation-seam/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/remove-dead-registry-status-mutation-seam.

## Task workflow update - 2026-08-18T00:10:56.565Z
- Recorded fork run: zkxdutbfw7fb
- Summary: Task claimed; worktree created on 5431f2d56 (post-tui-05 main). Orchestrator collected seam map: registry sole production ctor is ChatScreen:103; SlotBased fallbacks production-dead (all three closures wired); only external production reader is UsageCommandHandler:99 + RunActivityStateEnum docblock; ~17 test files + one installed-extension TEST use the seam. Implementation fork launched: move status/working state into ChatScreen private state, registry stripped to input-handler API only, SlotBasedTuiExtensionContext closures required (registry dropped or input-only), screen read getters statusEntries()/workingMessage(), refresh() safety-net deleted if dead, SubmitListenerReasoningNoticeClearTest workaround removed. Existing virtual + replay-backed real TUI status-row proofs stay as the user-visible proof; no new fixture.

## Task workflow update - 2026-08-18T00:20:46.200Z
- Recorded fork run: zkxdutbfw7fb
- Validation: Fork states testing skill + tests/AGENTS read/followed; Focused castor test filter: PASS 126 tests / 584 assertions; castor test full: PASS 4577 tests / 18236 assertions (delta -3 = exactly the 3 deleted TuiSlotRegistry tests); castor deptrac: PASS 0 violations (uncovered 2284, allowed 2604 — expected drop from removed unused edges); castor phpstan: PASS 0 errors; castor cs-check: PASS 0 files fixed; castor test:tui: PASS 38 tests / 317 assertions (125.8s real tmux, replay-backed); git diff --check clean; castor clean:cleanup:workers:list no stale workers; Extension guard: .hatfield/extensions diff contains only the OmBackgroundStatusVirtualRenderTest test file; extension production byte-unchanged; Not run by design: castor check, test:llm-real, reviewer, push/PR
- Summary: Implementation complete at verified commit c8d71373912e9c07720eeb5b6e93fa3cfb74518d (`refactor(tui): own slot state in screen`), clean tree. 24 files +127/-200 (net -73; production net -67). Status entries + working message/visibility now private ChatScreen state; setStatus/setWorkingMessage/setWorkingVisible keep exact dedup semantics and sync native widgets directly; TuiSlotRegistry stripped to input-handler storage only (6 mutators + 3 props deleted); SlotBasedTuiExtensionContext closures now required (dead registry-fallback branches deleted, FooterDataProvider required non-null, registry param kept solely for addInputHandler); public ChatScreen::registry() getter removed; screen getters statusEntries()/workingMessage() added (workingMessage has the UsageCommandHandler production caller); refresh() status re-push deleted after tracing all 8 callers; SubmitListenerReasoningNoticeClearTest refresh() workaround removed; TuiSlotRegistryTest stripped to input-only; SlotBasedTuiExtensionContextTest fallback assertions replaced by closure-routing behavior tests. Parent verified SHA/type/clean tree/diff stat, extension diff limited to observational-memory TEST file, zero remaining registry() status/working callers outside the unrelated AgentToolPolicyResolverTest fixture, and all setWorkingMessage/setWorkingVisible production call sites go through $screen.

## Task workflow update - 2026-08-18T00:31:14.732Z
- Validation: Reviewer (read/followed testing skill + tests/AGENTS): APPROVED at c8d713739, zero blockers; castor test: PASS 4577 tests / 18236 assertions (30.3s); castor deptrac: PASS 0 violations (2284 uncovered, 2604 allowed); castor phpstan: PASS 0 errors; castor cs-check: PASS 0 files fixed; castor test:tui: PASS 38 tests / 317 assertions (129.5s real tmux, replay-backed); castor clean:cleanup:workers:list: no stale workers; git diff --check: clean; worktree clean at verified HEAD c8d713739
- Summary: Task-to-PR reviewer verdict: APPROVED with zero blockers. All nine verification points confirmed: dedup/early-return semantics byte-equivalent; zero remaining callers of deleted registry APIs (only unrelated AgentToolPolicyResolverTest fixture uses ->registry(); docblock mentions updated); SlotBased ctor honest with required closures and input path intact via addInputHandler/registerSlotInputListeners; refresh() re-push deletion proven dead across all 8 callers; syncWorkingSlot ctor-time defaults safe; test edits mechanical with closure-routing coverage replacing fallback assertions and SubmitListenerReasoningNoticeClearTest now exercising the real setStatus path (stronger than before); extension guard holds (one test file only); deptrac allowed drop 2612->2604 expected; no new public API beyond the two sanctioned getters. NTHs (non-blocking): statusEntries() is test-only but task-sanctioned and strictly better than reflection in 13+ files; work-log production-net figure corrected to -46 (src/ +64/-110), total diff -73. Parent re-ran full focused battery green.

## Task workflow update - 2026-08-18T00:34:10.314Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (163.7s).
- Pushed task/remove-dead-registry-status-mutation-seam to origin.
- branch 'task/remove-dead-registry-status-mutation-seam' set up to track 'origin/task/remove-dead-registry-status-mutation-seam'.
- Created PR: https://github.com/ineersa/agent-core/pull/404

## Task workflow update - 2026-08-18T02:08:35.427Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/404
- Updated PR Status: open
- Validation: Commit 22def1a9e verified (rev-parse + cat-file = commit), clean tree, 1 file +1/-1; castor docs:validate: PASS (changed doc is in the docs catalog); git diff --check: clean; Fork states testing skill + tests/AGENTS read/followed (no test changes; docs-only does not trigger TUI/runtime castor check); Pushed: c8d713739..22def1a9e on task branch -> PR #404 updated
- Summary: PR #404 hatfield review gap addressed: docs-only commit 22def1a9e83e8c0ab278b56c146ff905902def3e (`docs(tui): registry stores input handlers only`) fixes the stale TuiSlotRegistry row in docs/tui-architecture.md Key-types table — now reads "Native terminal input-handler registration (priority + listener) consumed by ChatScreen mount", consistent with the class docblock and registerSlotInputListeners. Verified SHA/type/clean tree; castor docs:validate PASS; git diff --check clean; pushed to origin so PR #404 now contains c8d713739 + 22def1a9e. Both review nits (reflection probe in TickPollListenerTest, test-only statusEntries() getter) deliberately left per user decision. Reviewer had APPROVED the code; docs commit is 1 line, no re-review requested.

## Task workflow update - 2026-08-18T03:49:12.561Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/remove-dead-registry-status-mutation-seam.
- Merged task/remove-dead-registry-status-mutation-seam into integration checkout.
- Merge made by the 'ort' strategy.
 .../Tui/OmBackgroundStatusVirtualRenderTest.php    |  4 +-
 docs/tui-architecture.md                           |  2 +-
 src/Tui/Extension/SlotBasedTuiExtensionContext.php | 50 ++++++------------
 src/Tui/Layout/TuiSlotRegistry.php                 | 60 ++--------------------
 src/Tui/Listener/UsageCommandHandler.php           |  2 +-
 src/Tui/Runtime/RunActivityStateEnum.php           |  2 +-
 src/Tui/Screen/ChatScreen.php                      | 60 +++++++++++++++-------
 .../Extension/SlotBasedTuiExtensionContextTest.php | 55 ++++++++++++++------
 tests/Tui/Layout/TuiSlotRegistryTest.php           | 31 -----------
 .../Tui/Listener/AgentsMainCommandHandlerTest.php  |  2 +-
 tests/Tui/Listener/CancelListenerTest.php          |  2 +-
 .../Listener/RenameSessionCommandHandlerTest.php   |  2 +-
 .../Listener/ResumeSessionCommandHandlerTest.php   |  2 +-
 .../SubmitListenerReasoningNoticeClearTest.php     |  8 ++-
 .../SubmitListenerSubagentLiveInputTest.php        |  6 +--
 .../Tui/Listener/TickPollListenerChildHitlTest.php |  2 +-
 ...ickPollListenerSubagentLivePickerExportTest.php |  5 +-
 tests/Tui/Listener/TickPollListenerTest.php        | 10 ++--
 tests/Tui/Picker/SessionPickerControllerTest.php   |  2 +-
 .../Picker/SubagentLivePickerControllerTest.php    | 10 +---
 tests/Tui/Question/QuestionControllerTest.php      |  2 +-
 tests/Tui/Runtime/SubagentLiveAttentionTest.php    |  2 +-
 .../Tui/Scenario/SubagentLiveHitlScenarioTest.php  |  2 +-
 tests/Tui/Screen/TuiUsageCommandVirtualTest.php    |  4 +-
 tests/Tui/Support/SubagentLiveScenarioHarness.php  |  2 +-
 25 files changed, 128 insertions(+), 201 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/remove-dead-registry-status-mutation-seam.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/remove-dead-registry-status-mutation-seam.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-08-19T18:17:18.381Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
