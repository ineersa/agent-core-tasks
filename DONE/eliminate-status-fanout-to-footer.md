# Eliminate keyed status fan-out to the TUI footer

## Goal
`ChatScreen::setStatus()` currently projects every keyed status into both the status panel and `FooterDataProvider`. This causes panel-oriented messages, including observational-memory progress, to appear twice.

Change the status contract so keyed statuses have one presentation surface: the status panel. The footer should remain dedicated to its explicit footer data/segment providers rather than mirroring generic status entries.

Relevant areas identified during reconnaissance:
- `src/Tui/Screen/ChatScreen.php::setStatus()`
- `src/Tui/Footer/FooterDataProvider.php`
- `src/Tui/Footer/FooterBarWidget.php`
- `src/Tui/Status/StatusPanelWidget.php`
- `src/CodingAgent/ExtensionApi/Tui/TuiExtensionContextInterface.php`
- `docs/tui-architecture.md`
- `tests/Tui/Footer/FooterBarWidgetTest.php`
- `.hatfield/extensions/observational-memory/tests/Tui/OmBackgroundStatusVirtualRenderTest.php`

The fan-out was introduced intentionally with the original layout, so implementation must review existing callers such as question prompts, picker feedback, Ctrl-C feedback, runtime status updates, and extension statuses. They should remain visible in the status panel; do not add caller-specific footer compatibility paths or key-based exceptions.

Test thesis: a keyed status submitted through the real `ChatScreen`/extension status path is rendered once in the status panel and is absent from the footer. This is virtual/in-process TUI behavior; prove it at that layer rather than adding a tmux journey test.

## Acceptance criteria
- `ChatScreen::setStatus()` no longer mirrors keyed status entries into the footer.
- Keyed statuses from core callers and extensions remain visible in the status panel.
- Observational-memory progress is displayed once, in the status panel, and not duplicated in the footer.
- Obsolete generic footer status-entry storage/rendering is removed or narrowed so the footer contains only explicit footer data and segment-provider content.
- Current TUI architecture documentation is updated to describe the single-surface status contract.
- Virtual/in-process TUI tests prove the real status routing and rendering behavior; no tmux-only proof or extension-key special case is used.
- Focused Castor validation passes, followed by the required full `castor check` before CODE-REVIEW.

## Workflow metadata
Status: DONE
Branch: task/eliminate-status-fanout-to-footer
Worktree: /home/ineersa/projects/agent-core-worktrees/eliminate-status-fanout-to-footer
Fork run: 7vrdntcx53sq
PR URL: https://github.com/ineersa/agent-core/pull/358
PR Status: merged
Started: 2026-08-04T20:00:16.769Z
Completed: 2026-08-04T20:35:22.162Z

## Work log
- Created: 2026-08-04T19:09:29+00:00

## Task workflow update - 2026-08-04T19:11:18+00:00
- Validation: Reconnaissance only; no code changed and no QA run.; Both scouts read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before investigating TUI behavior.
- Summary: Reconnaissance is complete. The duplication has one producer and an intentional generic host fan-out: extension `setStatus()` reaches `ChatScreen::setStatus()`, which writes the same keyed value to both the status panel and footer. The task should remove the generic footer projection globally—not add an observational-memory key exception—while preserving keyed statuses in the panel and explicit footer providers/segments.
- ## Complete current flow and root cause

Observational memory has a single TUI producer, not duplicate reporters. `.hatfield/extensions/observational-memory/src/Tui/OmBackgroundStatusPoller.php` reads `om_current_activity`, formats it through `Runtime/OmActivityStatusText.php`, caches the last string, and calls `TuiExtensionContextInterface::setStatus('om-background', $text)` only when the text changes. Registration occurs once from `ObservationalMemoryExtension.php`.

The bridge is `src/Tui/Runtime/BridgeTuiExtensionContext.php::setStatus()`, which forwards to `ChatScreen::setStatus()`. In `src/Tui/Screen/ChatScreen.php` (reconnaissance snapshot lines 456-468), `setStatus()` updates all of: `TuiSlotRegistry`, `StatusPanelWidget`, and `FooterDataProvider`, then invalidates both panel and footer widgets. This is the exact duplication point.

Panel rendering path: `ChatScreen` → `src/Tui/Status/StatusPanelWidget.php`. Footer rendering path: `ChatScreen` → `src/Tui/Footer/FooterDataProvider.php::{setStatus,getStatusEntries}` → `src/Tui/Footer/FooterBarWidget.php`, which appends status values to the footer's right side. `WorkingStatusWidget` is unrelated; it renders generic working state independently.
- ## Historical intent and why global removal is now requested

The fan-out was intentional in the original layout rather than an accidental double render. Commit `137363698d27e540c6ea7d4dda10d86e0e25069a` (`Introduce TUI layout rendering system with slot-based extension support, footer data providers, and comprehensive test coverage`) introduced footer status storage/consumption and made `ChatScreen::setStatus()` write to both surfaces. Commit `d3696df9d370cccf756c687fefefa06c60bbadbf` retained keyed status entries while formalizing footer extensibility; current bridge behavior was carried through commit `5302f0de637c9bf537e1ab216b3ee72941662838`.

The dual projection is currently documented in `docs/tui-architecture.md` (reconnaissance snapshot around lines 345-356 and 581-587) and protected directly by `tests/Tui/Footer/FooterBarWidgetTest.php::testStatusEntriesAppearInFooter()`.

There is precedent showing the abstraction leaks panel-oriented state: commit `c82683aab9166a9e9302b03b872fade894fbb15e` routed reasoning through `ChatScreen::setStatus()` and unintentionally put it in the footer; commit `1ad4d68a59124b01fec433003460fc3cb9bb67e` (`fix(tui): correct reasoning display — clamp non-thinking models, panel-only status, no startup seed`) bypassed that bridge for panel-only reasoning. OM arrived later in `d9c04cb3d8f85d6f15a99eb419d5d4ffa9abd56e` as a TUI “status row” notice, inheriting footer display only because the public status API fans out.

Product decision finalized in this task: eliminate the generic fan-out globally. `setStatus()` means keyed panel status; footer content must come from explicit footer data/segment-provider APIs.
- ## Caller and impact inventory implementor should verify, not rediscover

Known `ChatScreen::setStatus()` callers include question action prompts (`src/Tui/Question/QuestionController.php`), picker feedback/errors (`src/Tui/Picker/ModelPickerController.php`, `SessionPickerController.php`, `TreePickerController.php`, `FavoritePickerController.php`), Ctrl-C feedback (`src/Tui/Listener/CtrlCInputInterceptor.php`), generic runtime status updates (`src/Tui/Listener/SubmitListener.php`), subagent status cleanup (`src/Tui/Runtime/SubagentLiveAttention.php`, `SubagentLiveMainReturn.php`), and extensions through `BridgeTuiExtensionContext`.

After this change these values must continue to appear in the status panel but must no longer be mirrored into the footer. This visible change is intentional. Do not retain footer copies for selected existing callers, add compatibility shims, add a new setting, or special-case the string key `om-background`.

Direct, explicit footer providers and footer segment providers are outside the removal target and must continue rendering. Inspect references before deleting `FooterDataProvider` APIs: remove/narrow only the generic keyed-status storage and rendering path; retain model/context/token or other explicit footer responsibilities.
- ## Smallest implementation shape and documentation updates

Start at `src/Tui/Screen/ChatScreen.php::setStatus()`: keep registry/status-panel update and panel invalidation; remove footer status update and status-triggered footer invalidation if no longer needed. Then remove the now-obsolete generic keyed-status collection/accessors from `src/Tui/Footer/FooterDataProvider.php` and its consumption from `src/Tui/Footer/FooterBarWidget.php`, subject to semantic reference verification.

Update `src/CodingAgent/ExtensionApi/Tui/TuiExtensionContextInterface.php` documentation if it currently implies dual display, and update both relevant passages in `docs/tui-architecture.md` so `setStatus()` is explicitly panel-only and footer extension content is supplied through footer-specific APIs. Preserve architecture boundaries and do not introduce a second panel-only API: the finalized requirement changes the existing status contract itself.

Review comments explaining non-obvious layout intent; update them to the new invariant rather than silently deleting rationale.
- ## Automated proof and validation requirements

Before touching tests or running QA, implementor must read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` and follow their conventions.

Lowest correct proof layer is virtual/in-process TUI, not tmux. Existing extension proof `.hatfield/extensions/observational-memory/tests/Tui/OmBackgroundStatusVirtualRenderTest.php` already exercises the real poller → extension bridge → `ChatScreen` → virtual terminal path. Strengthen it so the OM activity string occurs exactly once and/or prove the footer row does not contain it. Also replace/update `tests/Tui/Footer/FooterBarWidgetTest.php::testStatusEntriesAppearInFooter()` because that assertion protects the behavior being removed. Prefer one focused routing/render regression plus necessary adjustment of the direct footer unit test; avoid broad snapshot churn.

Test thesis: submitting a keyed status through the real `ChatScreen`/extension path leaves it visible in the status panel exactly once and absent from the footer, while explicit footer content still renders.

Run all QA through Castor. Focused implementation validation should include the relevant filtered virtual tests, then `castor test`, `castor deptrac`, `castor phpstan`, and `castor cs-check`. Because this touches TUI behavior, the mandatory full `castor check` gate is required before CODE-REVIEW; no new tmux journey is needed for this purely virtual routing/render behavior.

## Task workflow update - 2026-08-04T19:12:39+00:00
- ## User-visible impact and manual acceptance checklist

The affected surface is every caller of `ChatScreen::setStatus()` and every extension caller of `TuiExtensionContextInterface::setStatus()`: all of those messages will intentionally stop appearing in the footer and must remain functional in the status panel. Before CODE-REVIEW, perform focused exploratory verification of the following representative flows in addition to automated tests:

1. **Observational-memory background activity:** while observer/reflector/dropper progress is active, the `Observational memory: …` text appears exactly once in the status panel, never in the footer, updates when stage/token progress changes, and disappears when activity is cleared.
2. **Question/HITL action prompt:** open a question/approval interaction and confirm its keyed action/status guidance remains readable in the status panel, is not copied into the footer, and clears after answering/cancelling.
3. **Pickers:** exercise representative model and session picker feedback, plus an error/empty-state path if practical. Confirm messages from model/session/tree/favorite picker controllers still appear and clear in the status panel without footer copies.
4. **Ctrl-C feedback:** trigger the first Ctrl-C/confirmation hint and verify the feedback remains visible in the status panel, does not enter the footer, and follows its existing clear/expiry behavior.
5. **Generic runtime status updates:** exercise a normal submit/run transition that produces `StatusUpdate`/submit-listener status text. Confirm updates and cleanup still occur in the panel and no stale footer text remains.
6. **Subagent live status cleanup:** if a readily available flow produces keyed subagent status, confirm return/completion removes the panel entry and does not leave stale content in either surface. This is a cleanup regression check, not a requirement to add a new subagent feature test.
7. **Explicit footer content regression:** verify footer-owned content such as model/context/token information and registered footer segments still renders and updates. The footer must not become empty merely because generic keyed statuses were removed.
8. **Multiple/cleared keyed statuses:** where covered by the virtual harness, set more than one keyed status and then clear one with `setStatus($key, null)`; unaffected entries must remain in the panel and cleared text must not remain in the footer.

Automated minimum: virtual/in-process proof for real keyed-status routing (panel present, footer absent, clear behavior) plus retained direct proof that explicit footer providers/segments render. The manual checklist is focused exploratory acceptance, not justification for broad tmux journey tests. Record which flows were exercised and any impractical items in task validation notes. The mandatory `castor check` remains the final regression gate.

## Task workflow update - 2026-08-04T20:00:16.769Z
- Moved TODO → IN-PROGRESS.
- Created branch task/eliminate-status-fanout-to-footer.
- Created worktree /home/ineersa/projects/agent-core-worktrees/eliminate-status-fanout-to-footer.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/eliminate-status-fanout-to-footer.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/eliminate-status-fanout-to-footer.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/eliminate-status-fanout-to-footer.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/eliminate-status-fanout-to-footer.
- Summary: Task claimed for implementation. Loaded task-workflow, testing, and ponytail skills and read tests/AGENTS.md before planning TUI changes.

## Task workflow update - 2026-08-04T20:02:49.731Z
- Recorded fork run: 7vrdntcx53sq
- Summary: Implementation fork launched in `/home/ineersa/projects/agent-core-worktrees/eliminate-status-fanout-to-footer`. Existing task reconnaissance was sufficient, so no redundant scout was dispatched. Fork scope removes generic footer status storage/rendering, preserves panel routing and explicit footer segments, updates docs/caller tests, adds primary virtual OM proof plus the explicitly requested minimal replay-backed TmuxHarness OM path, runs focused Castor validation (not `castor check`), and commits the result.
- Specification fidelity: no new setting/API/storage/compatibility behavior; existing `setStatus()` contract becomes panel-only and footer remains explicit-provider-only.
- Testing instructions loaded by orchestrator: `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`. Fork was instructed to read both before TUI/test work and confirm in handoff.
- Fork run 7vrdntcx53sq launched in background; task-start stops before review/PR/full gate.

## Task workflow update - 2026-08-04T20:08:27.674Z
- Recorded fork run: 7vrdntcx53sq
- Validation: Focused virtual/unit Castor filters: OK — 18 tests, 67 assertions.; `castor test:tui --filter=TuiOmCommandsE2eTest`: OK — 2 tests, 7 assertions; real replay-backed TmuxHarness feature path, no live LLM.; `castor test`: OK — 4424 tests, 16430 assertions.; `castor deptrac`: OK — 0 violations.; `castor phpstan`: OK — 0 errors.; `castor cs-check`: OK — clean.; `castor check` intentionally not run during task-start; required in task-to-pr before CODE-REVIEW.; Parent verification: worktree clean; HEAD `c71ae2eb1579ba64d738b636999fd094fde5218c`; diff stat 16 files, 180 insertions, 112 deletions.
- Summary: Implementation completed and committed as `c71ae2eb1579ba64d738b636999fd094fde5218c` (`fix(tui): keep keyed statuses panel-only, drop footer fan-out`). Verified the worktree is clean, the commit exists, and its 16-file diff removes ChatScreen footer projection and FooterDataProvider status storage/rendering while preserving registry/status-panel routing and explicit footer segments. Model reasoning now uses the panel-only `ChatScreen::setStatus()` path; extension API docs and TUI architecture docs were updated. Primary virtual OM proof asserts exact-once panel rendering, footer absence, and clear behavior. A real replay-backed `TmuxHarness` OM test seeds `ActivityRepository` for the live session, exercises poller → bridge → ChatScreen, asserts exact-once current-viewport rendering and footer-line absence, then clears it. Fork confirmed it read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`. Manual HITL/picker/Ctrl-C exploratory checklist was not run; this remains optional pre-PR verification.

## Task workflow update - 2026-08-04T20:16:29.409Z
- Summary: Reviewer verdict on `c71ae2eb1`: APPROVE WITH SUGGESTIONS. Required proofs and specification fidelity passed, but actionable findings will be fixed before re-review: preserve same-frame footer reasoning color invalidation, make the tmux footer assertion fail-closed, remove now-redundant status-panel per-entry mutation/dead method after reference verification, and clarify the virtual test name's retained idle-cadence contract. Non-actionable notes about replay prompt wording and footer-format coupling are intentionally not changes.
- Reviewer confirmed it read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`.
- Reviewer explicitly verified real virtual OM poller→bridge→ChatScreen proof and real replay-backed TmuxHarness OM feature proof; no lingering removed footer-status API references.
- Specification fidelity passed: no unmapped external surface, shims, settings, or key exceptions.

## Task workflow update - 2026-08-04T20:18:45.817Z
- Validation: Focused Castor unit/virtual filters: OK — 42 tests, 151 assertions.; `castor test:tui --filter=TuiOmCommandsE2eTest`: OK — 2 tests, 9 assertions; real replay-backed TmuxHarness feature path.; `castor phpstan`: OK — 0 errors.; `castor cs-check`: OK — clean.; Fix fork read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before TUI/test/QA work.
- Summary: Review fixup committed as `45ebd78a9` (`fix(tui): restore same-frame reasoning footer refresh`). Verified clean worktree and 6-file fixup diff. Addressed all actionable reviewer findings: explicit same-frame footer invalidation for reasoning color, fail-closed tmux footer assertion, removal of redundant/dead status-panel per-entry mutation, and idle-safe virtual test naming. Non-actionable NTH notes were not changed.

## Task workflow update - 2026-08-04T20:27:50.135Z
- Validation: Reviewer: APPROVED on `45ebd78a907283d338244a97de4c9841254d47b3`; testing skill and `tests/AGENTS.md` read.; `castor test`: OK — 4424 tests, 16430 assertions (28.3s).; `castor deptrac`: OK — 0 violations, 0 errors.; `castor phpstan`: OK — 0 errors.; `castor cs-check`: OK — 0 files changed.; `castor test:tui`: OK — 33 tests, 211 assertions, 0 errors/failures/skips (96.3s); replay-backed, no live LLM.; Post-validation `git status --short --branch`: clean on `task/eliminate-status-fanout-to-footer`.
- Summary: Final reviewer verdict on current HEAD `45ebd78a907283d338244a97de4c9841254d47b3`: APPROVED with no actionable findings. Reviewer confirmed all prior findings resolved, specification fidelity passed, removed footer-status APIs have no references, primary virtual OM proof is real and complete, and the replay-backed no-live-LLM TmuxHarness OM feature proof is real and fail-closed. Worktree remains clean after full focused validation. No standalone `castor test:llm-real` was required because this task does not change provider/LLM-visible code.

## Task workflow update - 2026-08-04T20:29:50.776Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (105.1s).
- Pushed task/eliminate-status-fanout-to-footer to origin.
- branch 'task/eliminate-status-fanout-to-footer' set up to track 'origin/task/eliminate-status-fanout-to-footer'.
- Created PR: https://github.com/ineersa/agent-core/pull/358
- Validation: `castor test`: OK — 4424 tests, 16430 assertions.; `castor deptrac`: OK — 0 violations/errors.; `castor phpstan`: OK — 0 errors.; `castor cs-check`: OK — clean.; `castor test:tui`: OK — 33 tests, 211 assertions; real replay-backed TmuxHarness OM path included.; Reviewer: APPROVED; specification fidelity and required TUI proofs verified.
- Summary: Reviewer APPROVED current HEAD `45ebd78a907283d338244a97de4c9841254d47b3` after one fixup iteration. Focused Castor suite, Deptrac, PHPStan, CS check, and full replay-backed TUI suite all passed; worktree is clean. Preparing PR for panel-only keyed status routing and explicit-provider-only footer content.

## Task workflow update - 2026-08-04T20:29:55.727Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/358
- Updated PR Status: open
- Validation: Automatic deterministic `castor check`: PASSED — 105.1s.; PR: https://github.com/ineersa/agent-core/pull/358
- Summary: Task moved to CODE-REVIEW. Deterministic `castor check` passed in 105.1s, branch pushed, and PR #358 created.

## Task workflow update - 2026-08-04T20:35:22.162Z
- Moved CODE-REVIEW → DONE.
- Merged task/eliminate-status-fanout-to-footer into integration checkout.
- Merge made by the 'ort' strategy.
 .../Tui/OmBackgroundStatusVirtualRenderTest.php    |  42 ++++++--
 .../tests/Tui/TuiOmCommandsE2eTest.php             | 117 +++++++++++++++++++++
 docs/tui-architecture.md                           |   8 +-
 .../Tui/TuiExtensionContextInterface.php           |   6 ++
 src/Tui/Extension/TuiExtensionContext.php          |   2 +-
 src/Tui/Footer/FooterBarWidget.php                 |  39 ++-----
 src/Tui/Footer/FooterDataProvider.php              |  24 -----
 src/Tui/Listener/ModelControlListener.php          |   9 +-
 src/Tui/Screen/ChatScreen.php                      |   6 +-
 src/Tui/Status/StatusPanelWidget.php               |   9 --
 tests/Tui/Footer/FooterBarWidgetTest.php           |  13 ---
 .../Tui/Listener/AgentsMainCommandHandlerTest.php  |   5 +-
 tests/Tui/Listener/TickPollListenerTest.php        |   5 +-
 tests/Tui/Question/QuestionControllerTest.php      |   4 +-
 tests/Tui/Runtime/SubagentLiveAttentionTest.php    |   6 +-
 tests/Tui/Screen/TuiReasoningCycleTest.php         |   4 +-
 tests/Tui/Support/SubagentLiveScenarioHarness.php  |   6 +-
 17 files changed, 183 insertions(+), 122 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/eliminate-status-fanout-to-footer.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/eliminate-status-fanout-to-footer.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR state: MERGED — https://github.com/ineersa/agent-core/pull/358
- Summary: PR #358 confirmed merged on GitHub at 2026-08-04T20:34:58Z (merge commit `65ecc7f3bd8ee30a1e7f35f2c3bdb37a9ee8a934`). Moving task to DONE and syncing the integration checkout.

## Task workflow update - 2026-08-04T20:37:39.036Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/358
- Updated PR Status: merged
- Validation: Post-merge `LLM_MODE=true castor check`: PASSED.; Unit/integration: 4434 tests, 16492 assertions.; Controller replay: 11 tests, 160 assertions.; TUI replay: 33 tests, 213 assertions.; Live LLM: 13 tests, 144 assertions.; Deptrac, PHPStan, CS check, QA artifact integrity, leak check, and llama-proxy cache guard: PASSED.; Integration checkout clean; task worktree removed.; Testing skill and `tests/AGENTS.md` read before post-merge QA.
- Summary: Task completed. PR #358 was merged, the task branch was integrated into the main checkout, remote changes were pulled, the task worktree and IDEA exclusions were removed, and the integration checkout is clean.
