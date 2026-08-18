# tui-04: Harden TUI input, editor, and runtime infrastructure

## Goal
## Goal
Combine the input-pipeline, editor-operation, and small Symfony-reuse candidates into one bounded infrastructure task. Prefer native APIs; retain documented workarounds when Symfony lacks the needed seam.

## Architecture report evidence (candidates 6–8)

### Input pipeline
Input semantics depend on scattered numeric priorities (`CompletionListener`, `CtrlCInputInterceptor`, `ImagePasteInputListener`, `ModelControlListener`, extension slot handlers) and raw byte literals for Ctrl-C, Ctrl-D, Shift-Tab, arrows, and alternate arrow encodings. `PromptHistoryListener` already demonstrates Symfony TUI `Key`/`Keybindings::matches()` usage. Two extension mechanisms coexist: native propagatable `InputEvent` listeners and slot handlers fixed at priority 50 that cannot stop propagation.

### Editor text operations
`PromptEditor::acceptCompletion()` deletes text by sending Backspace events per grapheme; `typeText()` moves the cursor through synthetic DOWN/END keystrokes. This is fragile coupling to Symfony TUI editor internals, but the report found no cursor-aware `replaceRange()` API. The correct long-term solution is upstream; do not replace it with more local simulation. Characterize current behavior and prepare/upstream the smallest required Symfony TUI API when feasible.

### Small infrastructure reuse
Audit these exact sites and replace only when behavior is equivalent and net code falls:
- `RuntimeEventPoller::isFatalPollingError()` message-substring classification → typed runtime/transport exceptions,
- `FooterStateInitializer::detectGitBranch()` raw `proc_open` → Symfony Process in an allowed utility owner,
- `ExportCommandHandler::parsePathArg()` manual quotes → Symfony Console `StringInput` if escaping semantics match,
- direct clear-screen/cursor ANSI writes in `InteractiveMode` → upstream/native Symfony TUI API when available.
The report also noted tiny custom tick/lifecycle subscriber dispatchers; do not swap them for EventDispatcher unless the session-scope task proves a concrete simplification and updates Deptrac deliberately.

## Smallest viable direction
- Centralize named input-priority constants and route key recognition through Symfony `Keybindings` where it supports the exact sequences.
- Upgrade extension slot input handling to prioritized, stoppable native event participation without changing public extension behavior; if public API change is unavoidable, stop for explicit approval.
- Keep editor simulation as a documented, regression-covered workaround until an actual upstream API exists; no vendor patch or reflection.
- Convert runtime failures to typed exceptions at the producing boundary rather than adding more substring lists.

## Scope boundaries
- Preserve key precedence, completion/history/picker/editor behavior, extension input order, cancellation, paste handling, focus, and propagation.
- Preserve quoted export-path semantics exactly; skip `StringInput` if backslash/quote rules differ.
- Preserve terminal clearing behavior and native scrollback rationale; no ScreenWriter replacement or raw vendor modification.
- No new user setting, keybinding surface, command, or compatibility layer.

## Test thesis
Virtual input tests should prove precedence, propagation stop, raw/alternate key forms, completion acceptance/replacement, multiline typing, history, paste, and extension handlers. Typed transport failure tests should exercise fatal vs degradable polling behavior. Use tmux only for terminal semantics not provable virtually.

## Acceptance criteria
- Input priorities have named ownership and key matching uses Symfony TUI `Keybindings` wherever semantics are equivalent.
- Extension input handlers participate in one prioritized, propagation-aware pipeline without breaking public extension behavior.
- Prompt editor completion and multiline insertion are regression-covered; synthetic keystrokes are removed only after a real cursor-aware native API is available.
- Fatal runtime polling classification uses typed exceptions rather than broad message substrings.
- Git branch detection uses Symfony Process if behavior remains equivalent; export parsing and terminal clearing use Symfony APIs only when exact semantics are proven.
- Rejected native substitutions are recorded as KEEP with the concrete mismatch; no glue-heavy wrappers are introduced.
- The fork and handoff state that `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` were read and followed.
- Focused virtual tests, `castor test`, `castor test:controller-replay`, `castor test:tui`, `castor deptrac`, `castor phpstan`, `castor cs-check`, and final `castor check` pass.

## Workflow metadata
Status: ARCHIVE
Branch: task/tui-04-harden-input-editor-and-runtime-infrastructure
Worktree: /home/ineersa/projects/agent-core-worktrees/tui-04-harden-input-editor-and-runtime-infrastructure
Fork run: 0cfb58f7f3a8
PR URL: https://github.com/ineersa/agent-core/pull/401
PR Status: merged
Started: 2026-08-17T02:57:50.352Z
Completed: 2026-08-17T17:56:09.787Z

## Work log
- Created: 2026-08-15T23:16:16.558Z

## Task workflow update - 2026-08-15T23:19:03.248Z
- Full architect report: [`/home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md`](file:///home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md) — TUI report, candidates 6–8.

## Task workflow update - 2026-08-17T02:57:50.352Z
- Moved TODO → IN-PROGRESS.
- Created branch task/tui-04-harden-input-editor-and-runtime-infrastructure.
- Created worktree /home/ineersa/projects/agent-core-worktrees/tui-04-harden-input-editor-and-runtime-infrastructure.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/tui-04-harden-input-editor-and-runtime-infrastructure.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/tui-04-harden-input-editor-and-runtime-infrastructure.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/tui-04-harden-input-editor-and-runtime-infrastructure.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/tui-04-harden-input-editor-and-runtime-infrastructure.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/tui-04-harden-input-editor-and-runtime-infrastructure/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/tui-04-harden-input-editor-and-runtime-infrastructure.
- Summary: Starting bounded TUI infrastructure hardening for architecture-report candidates 6–8: unify prioritized/stoppable input handling, characterize editor simulations, introduce typed fatal runtime errors at producer boundaries, and adopt native Symfony Process/StringInput/terminal APIs only where semantics are exactly equivalent.

## Task workflow update - 2026-08-17T03:07:49.043Z
- Summary: Three scouts completed after reading testing skill + tests/AGENTS. Input: priorities are 105/100/98/96/95/90/50; public ExtensionApi exposes no terminal-input registration, so the internal TuiExtensionContext/slot path may safely become native prioritized InputEvent listeners without public API change. Strict raw key comparisons must remain where Keybindings would also accept Kitty forms and therefore broaden behavior; existing history/picker Keybindings stay native. Editor: installed Symfony TUI exposes no public cursor/range API, so keep documented synthetic navigation/backspace workaround; characterize cursor-at-end, multiline and byte-offset behavior and strengthen existing real-tmux slash-completion phase. Runtime: introduce typed RuntimeTransportException at process/stream producer boundaries and classify only that type as immediately fatal; replace raw git proc_open via a concrete TuiUtility Symfony Process owner; KEEP export parser because StringInput changes backslash/quote/empty/option semantics; route terminal clearing through Tui terminal APIs while retaining scrollback clear. Do not touch tick/lifecycle dispatchers or ExtensionApi.

## Task workflow update - 2026-08-17T03:08:43.864Z
- Recorded fork run: xlwnz6vrpfq3
- Summary: Launched single implementation fork on exact worktree. Scope: named input priorities + internal native prioritized/stoppable InputEvent seam (no ExtensionApi change), editor byte-offset/multiline/history hardening while retaining documented Symfony workaround, typed RuntimeTransportException producer ownership, Symfony Process git detector, terminal API clearing, explicit KEEP for raw keys/StringInput mismatch, and mandatory replay-backed TuiJourney real-Tmux completion/cursor proof. Fork owns focused/full Castor lanes except final castor check, commit, and clean handoff.

## Task workflow update - 2026-08-17T03:46:11.473Z
- Recorded fork run: xlwnz6vrpfq3
- Validation: Read/followed .agents/skills/testing/SKILL.md and tests/AGENTS.md; castor test: PASS — 4503 tests, 17707 assertions; castor test:controller-replay: PASS — 12 tests, 165 assertions; castor test:tui --filter=TuiJourneyE2eTest: PASS — 1 test, 19 assertions, real TmuxHarness; castor test:tui: PASS twice consecutively — 38 tests, 295 then 297 assertions; castor deptrac: PASS — 0 violations; castor phpstan: PASS — 0 errors; castor cs-check: PASS — 0 files fixed; git diff --check: clean; castor clean:cleanup:workers:list: no stale workers; IDE diagnostics on key changed production files: no problems; castor check: NOT RUN — reserved for task-to-pr transition
- Summary: Implementation complete and committed as b260f16c23daafb54e9bb055807d3098664f9263 (`refactor(tui): harden input and runtime infrastructure`), verified via git rev-parse + git cat-file; worktree clean. 31 files changed (+562/-159). Added named input priorities and native prioritized/stoppable internal slot InputEvent listeners; hardened byte-offset/multiline/history editor behavior while retaining the documented Symfony cursor workaround; added typed RuntimeTransportException classification at transport producers; moved git branch detection to Symfony Process utility; routed switch clear/exit newline through Terminal APIs. ExtensionApi byte-unchanged; tick/lifecycle dispatchers untouched. Mandatory replay-backed real TmuxHarness journey now proves typing lands after accepted slash completion. KEEP decisions: strict raw keys because Keybindings broadens via Kitty CSI-u matching; manual export path parser because StringInput semantics differ. One full TUI run transiently failed before two consecutive clean reruns; final castor check remains for task-to-pr.

## Task workflow update - 2026-08-17T16:00:18.842Z
- Summary: Task-to-PR review at b260f16c2 returned REQUEST CHANGES with one concrete blocker: GitBranchDetector returns an identity ternary after trim; replace with direct return. Reviewer otherwise approved architecture, specification fidelity, typed transport mapping, editor/history behavior, native priority/propagation seam, Symfony Process/Terminal substitutions, test quality, ExtensionApi unchanged, tick/lifecycle untouched, and mandatory real replay-backed TmuxHarness proof. Also noted a small doc wording clarification ('public ExtensionApi') as sensible cleanup. Skipping timeout NTH: adding a new timeout policy changes behavior and is outside finalized exact-equivalence scope.
- Reviewer read testing skill/tests AGENTS and reviewed all 31 files. Verdict REQUEST CHANGES: shrink GitBranchDetector identity ternary; no correctness/security blockers. Real TmuxHarness journey proof accepted. Launching minimal fork for blocker plus adjacent public-API wording clarification.

## Task workflow update - 2026-08-17T16:01:36.553Z
- Recorded fork run: 0cfb58f7f3a8
- Validation: castor phpstan: PASS — 0 errors; castor cs-check: PASS — 0 files fixed; git diff --check: clean; ExtensionApi and tick/lifecycle dispatchers unchanged; git rev-parse + git cat-file verified commit; worktree clean
- Summary: Review-fix commit 7c5d9d15ee61843fcfa783ede7f1653de306c7ab verified and clean. Removed GitBranchDetector identity ternary and clarified ChatScreen late-registration docblock refers to public ExtensionApi. No behavior change; timeout NTH intentionally skipped as out-of-scope policy change.
- Review fix fork produced commit 7c5d9d15e (+4/-4 across 2 files), resolving the sole reviewer blocker and adjacent doc ambiguity. Re-review required on current HEAD.

## Task workflow update - 2026-08-17T16:01:43.438Z
- Summary: Metadata correction: the synchronous review-fix fork response did not expose a fork run ID. Disregard the previously recorded `0cfb58f7f3a8`; commit 7c5d9d15ee61843fcfa783ede7f1653de306c7ab is the authoritative verified artifact.
- Correction: synchronous fork returned no visible run ID; do not treat 0cfb58f7f3a8 as a real fork run. Commit SHA verification is authoritative.

## Task workflow update - 2026-08-17T16:03:13.112Z
- Summary: Targeted re-review at HEAD 7c5d9d15e returned APPROVED. Reviewer confirmed sole blocker removed, public ExtensionApi wording clarified, no behavior drift/scope expansion, timeout NTH correctly omitted, mandatory real TmuxHarness proof intact, ExtensionApi/tick/lifecycle unchanged, and clean worktree.
- Reviewer outcome: APPROVED on current HEAD 7c5d9d15e. Proceeding to required focused Castor validation before CODE-REVIEW transition.

## Task workflow update - 2026-08-17T16:06:11.729Z
- Validation: castor test: PASS — 4503 tests, 17707 assertions, 33.4s; castor deptrac: PASS — 0 violations; castor phpstan: PASS — 0 errors; castor cs-check: PASS — 0 files fixed; castor test:tui: PASS — 38 tests, 301 assertions, 114.4s, replay-backed real TmuxHarness; castor clean:cleanup:workers:list: no stale workers; git diff --check: clean; git status: clean
- Summary: Current clean HEAD 7c5d9d15ee61843fcfa783ede7f1653de306c7ab is reviewer APPROVED and passed all required focused validation. No provider/LLM-visible changes, so test:llm-real is not required.
- Final focused task-to-PR validation passed at 7c5d9d15e. Reviewer APPROVED. Ready for move_task CODE-REVIEW deterministic castor check, push, and PR creation.

## Task workflow update - 2026-08-17T16:08:35.136Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 600s)...
- castor check passed (128.3s).
- Pushed task/tui-04-harden-input-editor-and-runtime-infrastructure to origin.
- branch 'task/tui-04-harden-input-editor-and-runtime-infrastructure' set up to track 'origin/task/tui-04-harden-input-editor-and-runtime-infrastructure'.
- Created PR: https://github.com/ineersa/agent-core/pull/401
- Validation: castor test: PASS — 4503 tests, 17707 assertions; castor deptrac: PASS — 0 violations; castor phpstan: PASS — 0 errors; castor cs-check: PASS — clean; castor test:tui: PASS — 38 tests, 301 assertions; workers clean; git diff/status clean
- Summary: Reviewer APPROVED current HEAD 7c5d9d15e after one minimal cleanup commit. Named/stoppable native input pipeline, editor byte/multiline/history hardening, typed transport failures, Symfony Process git detection, and Terminal API reuse complete. Mandatory real TmuxHarness completion/cursor path proof present. ExtensionApi and tick/lifecycle dispatchers unchanged.

## Task workflow update - 2026-08-17T16:08:41.643Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/401
- Updated PR Status: open
- Validation: move_task deterministic castor check: PASS — 128.3s
- Summary: PR #401 created after deterministic castor check passed in 128.3s: https://github.com/ineersa/agent-core/pull/401. Branch pushed; task now CODE-REVIEW.
- IN-PROGRESS → CODE-REVIEW complete. Pushed task branch and opened PR #401.

## Task workflow update - 2026-08-17T17:07:57.101Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Addressing PR #401 comments. One code change accepted: make stateless GitBranchDetector::detect() static and remove DI constructor churn. Three comments need evidence/replies rather than changes: UnicodeString is incorrect because completion offsets are bytes; InputPriority cannot be a backed enum because priority 90 is intentionally shared and Symfony requires raw ints; history cursor issue is reproducible and covered by failing-before tests.

## Task workflow update - 2026-08-17T17:10:23.129Z
- Validation: castor test --filter='GitBranchDetectorTest|FooterStateListenerTest': PASS — 4 tests, 14 assertions; castor deptrac: PASS — 0 violations; castor phpstan: PASS — 0 errors; castor cs-check: PASS — 0 files fixed; git diff --check: clean; git rev-parse + git cat-file verified commit; worktree clean
- Summary: PR #401 iteration commit b0e790facbcb217f413de048156c413e08f085c3 verified and clean. Made stateless GitBranchDetector::detect() static, removed FooterStateInitializer DI dependency, and updated only affected tests. All semantic/grep references migrated; no instance use remains. Other three comments require evidence replies, not code changes.
- PR feedback fix: b0e790fac (+4/-7 across 4 files) makes branch detection static and removes constructor churn. Awaiting re-review and full task-to-PR validation.

## Task workflow update - 2026-08-17T17:17:30.309Z
- Summary: Reviewer APPROVED current HEAD b0e790fac. All four PR comments adjudicated: byte substr is required by documented byte offsets; constants class is smaller/correct because priority 90 is duplicated and Symfony requires ints; branch detector is now static; history cursor issue is reproducible against vendor setText cursor reset and failing-before tests. Reviewer confirmed real TmuxHarness proof and architecture boundaries remain intact.
- Re-review APPROVED at b0e790fac. Skipping private-constructor NTH: existing static Tui Utility pattern (Clipboard) does not add one, and accidental instantiation has no correctness cost. Proceeding to full focused task-to-PR validation.

## Task workflow update - 2026-08-17T17:20:48.037Z
- Validation: castor test: PASS — 4503 tests, 17707 assertions, 34.5s; castor deptrac: PASS — 0 violations; castor phpstan: PASS — 0 errors; castor cs-check: PASS — 0 files fixed; castor test:tui: PASS — 38 tests, 301 assertions, 133.9s; castor clean:cleanup:workers:list: no stale workers; git diff --check: clean; worktree clean; branch ahead of origin by one expected feedback commit
- Summary: PR feedback iteration at clean HEAD b0e790fac is reviewer APPROVED and all required focused lanes pass. Ready to update PR #401 through CODE-REVIEW transition.
- Full PR-iteration validation passed at b0e790fac after static branch detector change. Moving back to CODE-REVIEW will run deterministic castor check and push/update PR #401.

## Task workflow update - 2026-08-17T17:32:56.043Z
- Validation: castor check attempt: all reported blocker is test:tui; failure repeated twice at TuiResumeSessionSwitchE2eTest.php:157; focused castor test:tui --filter=TuiResumeSessionSwitchE2eTest: PASS — 3 tests, 14 assertions; no stale QA workers
- Summary: Deterministic castor check on b0e790fac failed TUI twice at the same race: TuiResumeSessionSwitchE2eTest typed `/new` and immediately sent Enter under parallel gate load, but capture showed the completion overlay still open and no switch. Focused class rerun passed 3/14, while both full parallel gate attempts reproduced the failure. This is now actionable test synchronization, not an acceptable one-off flake; launching a narrow TmuxHarness test fix to wait for `/new` completion readiness before Enter.
- CODE-REVIEW transition remains blocked. Root cause evidence: pane contains `/new` plus Completions overlay and old transcript, proving Enter raced completion readiness. Minimal fix should synchronize on the actual completion row before Enter, then preserve existing welcome+idle-ready waits.

## Task workflow update - 2026-08-17T17:38:00.096Z
- Validation: focused castor test:tui --filter=TuiResumeSessionSwitchE2eTest: PASS — 3 tests, 14 assertions; full castor test:tui: PASS — 38 tests, 295 assertions; castor cs-check: PASS — 0 files fixed; git diff --check: clean; workers list: no stale workers
- Summary: Committed deterministic gate race fix as 44f5ab16763230c4e348450e2116678559d3d92e, verified clean. Test-only change waits for the live `/new` completion overlay/row before sending Enter, using existing TUI_GATE_CALLBACK_TIMEOUT_PARALLEL and no sleep; existing post-switch assertions remain unchanged.
- Gate-flake fix commit 44f5ab167 adds one readiness wait (+16 lines) to the existing real TmuxHarness isolation test. Re-review required before another CODE-REVIEW attempt.

## Task workflow update - 2026-08-17T17:42:11.344Z
- Summary: Reviewer APPROVED gate synchronization commit 44f5ab167. Confirmed it is the smallest correct readiness barrier, preserves real Enter accepts+submits path, uses no sleep/new timeout, cannot stale-match in this test, and leaves all post-switch assertions unchanged.
- Targeted re-review APPROVED at 44f5ab167. Proceeding with remaining required focused Castor lanes before final CODE-REVIEW attempt.

## Task workflow update - 2026-08-17T17:43:11.220Z
- Validation: castor test: PASS — 4503 tests, 17707 assertions; castor deptrac: PASS — 0 violations; castor phpstan: PASS — 0 errors; castor cs-check: PASS — clean; castor test:tui: PASS — 38 tests, 295 assertions at current HEAD; focused TuiResumeSessionSwitchE2eTest: PASS — 3 tests, 14 assertions; workers/diff/status clean
- Summary: Current clean HEAD 44f5ab167 is reviewer APPROVED and all focused validation passes, including corrected full real-Tmux lane. Ready for deterministic castor check and PR #401 update.
- Final PR-iteration validation passed at 44f5ab167 after synchronization fix; task ready for CODE-REVIEW gate.

## Task workflow update - 2026-08-17T17:45:43.492Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 600s)...
- castor check passed (143.8s).
- Pushed task/tui-04-harden-input-editor-and-runtime-infrastructure to origin.
- branch 'task/tui-04-harden-input-editor-and-runtime-infrastructure' set up to track 'origin/task/tui-04-harden-input-editor-and-runtime-infrastructure'.
- Skipped PR creation (pushOnly: true).
- Validation: castor test: PASS — 4503/17707; castor deptrac: PASS — 0 violations; castor phpstan: PASS — 0 errors; castor cs-check: PASS; castor test:tui: PASS — 38/295; focused session-switch Tmux test: PASS — 3/14; workers/diff/status clean
- Summary: PR #401 comments addressed at current HEAD 44f5ab167. GitBranchDetector is static; evidence replies support byte offsets, constants class, and reproducible history fix. Repeated gate-only `/new` completion race was fixed with a reviewed real-Tmux readiness barrier. Reviewer APPROVED and all focused lanes pass. Existing PR URL preserved; push-only avoids prior transient GitHub GraphQL 503.

## Task workflow update - 2026-08-17T17:46:08.541Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/401
- Updated PR Status: open
- Validation: deterministic castor check: PASS — 143.8s; branch pushed at 44f5ab167; all four PR inline comments replied
- Summary: PR #401 comments addressed. Static detector change and Tmux gate synchronization commits pushed through HEAD 44f5ab167. Replied to all four inline discussions with code/vendor/test evidence. Reviewer APPROVED; deterministic castor check passed in 143.8s; task returned to CODE-REVIEW.
- PR iteration complete: b0e790fac makes branch detector static; 44f5ab167 stabilizes `/new` real-Tmux completion readiness. Full gate passed and all owner comments have replies.

## Task workflow update - 2026-08-17T17:56:09.787Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/tui-04-harden-input-editor-and-runtime-infrastructure.
- Merged task/tui-04-harden-input-editor-and-runtime-infrastructure into integration checkout.
- Merge made by the 'ort' strategy.
 .../Runtime/Contract/RuntimeTransportException.php | 17 +++++
 .../Runtime/Controller/RuntimeEventEmitter.php     |  3 +-
 .../Process/JsonlProcessAgentSessionClient.php     | 17 ++---
 .../Runtime/Stream/StdoutRuntimeEventSink.php      |  3 +-
 src/Tui/Application/InteractiveMode.php            | 42 +++---------
 src/Tui/Editor/PromptEditor.php                    | 39 +++++------
 src/Tui/Extension/SlotBasedTuiExtensionContext.php | 10 ++-
 src/Tui/Extension/TuiExtensionContext.php          | 13 +++-
 src/Tui/Layout/InputPriority.php                   | 41 ++++++++++++
 src/Tui/Layout/TuiSlotRegistry.php                 | 30 +++++++--
 src/Tui/Listener/CompletionListener.php            | 21 +++---
 src/Tui/Listener/CtrlCInputInterceptor.php         |  5 +-
 src/Tui/Listener/FooterStateInitializer.php        | 39 +----------
 src/Tui/Listener/ImagePasteInputListener.php       |  3 +-
 .../Listener/LoadedResourcesStartupRegistrar.php   |  3 +-
 src/Tui/Listener/ModelControlListener.php          |  5 +-
 src/Tui/Listener/PreviewExpansionInputListener.php |  5 +-
 src/Tui/Listener/PromptHistoryListener.php         | 34 +++++++++-
 .../Listener/SubagentLiveToggleInputListener.php   |  3 +-
 src/Tui/Runtime/RuntimeEventPoller.php             | 19 ++----
 src/Tui/Screen/ChatScreen.php                      | 21 ++++++
 src/Tui/Utility/GitBranchDetector.php              | 39 +++++++++++
 .../Runtime/Stream/StdoutRuntimeEventSinkTest.php  | 26 ++++++++
 tests/Tui/E2E/TuiJourneyE2eTest.php                | 21 +++++-
 tests/Tui/E2E/TuiResumeSessionSwitchE2eTest.php    | 16 +++++
 tests/Tui/Editor/PromptEditorTest.php              | 32 +++++++++
 .../Extension/SlotBasedTuiExtensionContextTest.php | 76 +++++++++++++++++++++-
 tests/Tui/Layout/TuiSlotRegistryTest.php           | 14 ++--
 tests/Tui/Listener/PromptHistoryListenerTest.php   | 37 +++++++++++
 tests/Tui/Runtime/RuntimeEventPollerTest.php       | 30 +++++++--
 tests/Tui/Utility/GitBranchDetectorTest.php        | 70 ++++++++++++++++++++
 31 files changed, 575 insertions(+), 159 deletions(-)
 create mode 100644 src/CodingAgent/Runtime/Contract/RuntimeTransportException.php
 create mode 100644 src/Tui/Layout/InputPriority.php
 create mode 100644 src/Tui/Utility/GitBranchDetector.php
 create mode 100644 tests/Tui/Utility/GitBranchDetectorTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/tui-04-harden-input-editor-and-runtime-infrastructure.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/tui-04-harden-input-editor-and-runtime-infrastructure.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #401 merged: confirmed via GitHub REST API; pre-merge deterministic castor check: PASS — 143.8s; final focused validation: castor test 4503/17707; deptrac 0; phpstan 0; cs clean; TUI 38/295
- Summary: PR #401 confirmed merged on GitHub at 2026-08-17T17:55:32Z with merge commit c4d7fe43daceba31acb7f18a25c24b3dfa4dc8f4. Final task HEAD 44f5ab167; reviewer approved and deterministic castor check passed before merge.

## Task workflow update - 2026-08-17T17:59:13.354Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/401
- Updated PR Status: merged
- Validation: Post-merge QA run qa-20260817-175633-409098-6c6d9020: PASS; deptrac PASS; test PASS — 4553 tests, 17867 assertions; test:controller-replay PASS — 12 tests, 165 assertions; test:tui PASS — 38 tests, 297 assertions; test:llm-real PASS — 13 tests, 144 assertions; phpstan PASS — 0 errors; cs-check PASS; docs:validate PASS; QA leak check PASS; llama-proxy cache stable 348→348
- Summary: Task complete. PR #401 merged, branch integrated, worktree/IDE exclusions removed, and post-merge LLM_MODE=true castor check passed all 8 lanes on integration HEAD a31f499f9.
- DONE validation complete: LLM_MODE=true castor check passed in 511.7s after merge; integration checkout clean and task worktree removed.

## Task workflow update - 2026-08-18T00:06:36.283Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
