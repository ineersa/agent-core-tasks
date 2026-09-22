# Show and highlight the exact Safeguard approval trigger

## Goal
Safeguard approval prompts currently identify an operation as destructive or otherwise guarded without clearly showing the exact input that triggered the decision. Before approving or denying, the user should see both what will run and why Safeguard intercepted it.

For guarded tool operations, show the relevant triggering input in the approval UI—for example, the complete shell command for a guarded `bash` call. When the guard was triggered by one or more regex matches, visually emphasize the exact matched substring(s) inside that input using the existing theme (for example warning/error color and bold), while keeping the surrounding content readable.

The display must be derived from Safeguard's actual decision evidence, not by independently re-running or approximating policy matching in the TUI. Preserve original input exactly apart from presentation styling, safely escape markup/control content, wrap long input without truncation, and retain the existing approve/deny behavior.

Keep the change minimal: extend the existing Safeguard approval presentation and decision contract only as needed. Do not add policy configuration, expose unrelated/raw tool payload fields, or build a general-purpose syntax highlighter.

## Acceptance criteria
- Every Safeguard approval prompt shows the exact relevant operation input that caused interception, such as the complete command for a guarded shell operation.
- When Safeguard's decision includes regex match evidence, every actual matched span is distinctly highlighted within the displayed input using existing theme styles; surrounding text remains legible.
- The UI consumes match evidence produced by the Safeguard decision path and does not duplicate policy evaluation or rerun regex rules in presentation code.
- Long or multiline triggering input wraps without ellipsis/truncation, preserves meaningful whitespace, and remains associated with the approval reason.
- Overlapping, repeated, Unicode, and multiple-rule match spans render deterministically without corrupting text or ANSI styling.
- Untrusted command/tool text is escaped so it cannot inject Symfony Console markup or terminal control sequences through the approval display.
- Approval, denial, cancellation, tool execution, and audit/logging behavior remain unchanged; no sensitive unrelated payload data is displayed or logged.
- Automated tests cover decision evidence propagation and virtual TUI rendering for plain, matched, multiple-match, multiline, and escaped trigger input; required focused Castor validation passes.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-20-show-safeguard-trigger-context
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-20-show-safeguard-trigger-context
Fork run: nh5wwdtcadwb
PR URL: https://github.com/ineersa/agent-core/pull/439
PR Status: merged
Started: 2026-08-28T14:21:55.930Z
Completed: 2026-08-28T19:15:43.571Z

## Work log
- Created: 2026-08-20T16:39:29+00:00

## Task workflow update - 2026-08-28T14:21:55.930Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-20-show-safeguard-trigger-context.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-show-safeguard-trigger-context.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-20-show-safeguard-trigger-context.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-20-show-safeguard-trigger-context.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-20-show-safeguard-trigger-context.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-20-show-safeguard-trigger-context.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-20-show-safeguard-trigger-context/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-show-safeguard-trigger-context.

## Task workflow update - 2026-08-28T14:23:48.682Z
- Recorded fork run: dbwlsyg85s5f
- Ownership: owner=fork; fork_run=dbwlsyg85s5f; revision=task branch baseline at 2026-08-28 task-start; scope=complete SafeGuard trigger-evidence propagation, approval presentation, virtual/unit proof, focused validation; outcome=assigned; commit=none

## Task workflow update - 2026-08-28T14:24:26.129Z
- Ownership: owner=fork; fork_run=dbwlsyg85s5f; revision=a9f1cbefd; scope=complete SafeGuard trigger-evidence propagation, approval presentation, virtual/unit proof, focused validation; outcome=assigned; commit=none

## Task workflow update - 2026-08-28T14:36:48.271Z
- Validation: Fork owner: castor test --filter='(SafeGuardCommandMatcherTest|SafeGuardToolCallHookTest|QuestionControllerTest)' — PASS (75 tests, 207 assertions).; Parent verification: castor test --filter='(SafeGuardCommandMatcherTest|SafeGuardToolCallHookTest|QuestionControllerTest)' — PASS (75 tests, 207 assertions; 1.5s test task after PHAR ensure).; castor deptrac — PASS (0 violations/errors).; castor phpstan — PASS (0 errors).; castor cs-check — PASS (no files requiring changes).; git show --check 5c62a7379c88be9f1705a37292a2cca3f9402159 — PASS; worktree clean.; Full castor check intentionally not run during task-start; CODE-REVIEW transition owns the full gate.
- Summary: Implementation complete on commit 5c62a7379c88be9f1705a37292a2cca3f9402159. SafeGuard decisions now carry exact trigger input and byte-offset match evidence through approval details into the TUI question overlay. The overlay renders literal, wrapped, non-truncated input; merges repeated/overlapping spans; highlights matches with theme warning/bold styling; and visibly escapes terminal controls without interpreting Symfony-style markup. Non-bash approvals carry only their relevant path/settings operation input. Approval resolution, execution, audit, settings, commands, persistence, and Extension API contracts remain unchanged. Worktree is clean.
- Ownership: owner=fork; fork_run=dbwlsyg85s5f; revision=a9f1cbefd; scope=complete SafeGuard trigger-evidence propagation, approval presentation, virtual/unit proof, focused validation; outcome=completed; commit=5c62a7379c88be9f1705a37292a2cca3f9402159

## Task workflow update - 2026-08-28T15:49:20.972Z
- Summary: task-to-pr reviewer at revision 5c62a7379c88be9f1705a37292a2cca3f9402159 returned REQUEST CHANGES. Blockers: byte-mode control sanitization corrupts valid UTF-8 continuation bytes; raw-input fallback can bypass sanitization on PCRE failure; the runtime-event-to-QuestionRequest propagation hop lacks direct proof; overlap coverage is not a real overlap; and the trigger label should cross the same sanitization boundary. Verified approval/audit semantics, markup non-interpretation, decision-origin evidence, and lowest-layer virtual rendering design were otherwise sound.
- Review: role=reviewer; reviewer_run=pi-subagent-result-id-not-exposed; revision=5c62a7379c88be9f1705a37292a2cca3f9402159; scope=full specification-fidelity, security/correctness, evidence-path, and test-layer review; decision=REQUEST CHANGES
- Ownership: owner=main; fork_run=none; revision=5c62a7379c88be9f1705a37292a2cca3f9402159; scope=review blockers only: UTF-8/control-safe literal rendering, label sanitization, real overlap proof, runtime-event propagation proof, focused validation; outcome=assigned; commit=none

## Task workflow update - 2026-08-28T15:52:38.738Z
- Validation: castor test --filter='(SafeGuardCommandMatcherTest|SafeGuardToolCallHookTest|QuestionControllerTest|TickPollListenerTest)' — PASS (111 tests, 340 assertions).; castor deptrac — PASS (0 violations/errors).; castor phpstan — PASS (0 errors).; castor cs-check — PASS (no files requiring changes).; git diff --check — PASS before commit.
- Summary: Addressed reviewer blockers in commit cfa3c30f1. Trigger text and labels now sanitize malformed UTF-8 through Symfony StringUtils and escape Unicode control code points in Unicode mode without misclassifying valid UTF-8 continuation bytes; sanitizer failure throws instead of passing raw text. Virtual rendering now proves uppercase Greek/CJK preservation and label/control escaping. Matcher evidence now proves genuine repeated, overlapping, multi-rule spans. Existing RuntimeQuestionEventHandler coverage now proves trigger input/label/span propagation and malformed-span filtering.
- Ownership: owner=main; fork_run=none; revision=5c62a7379c88be9f1705a37292a2cca3f9402159; scope=review blockers only: UTF-8/control-safe literal rendering, label sanitization, real overlap proof, runtime-event propagation proof, focused validation; outcome=completed; commit=cfa3c30f1

## Task workflow update - 2026-08-28T16:05:06.367Z
- Validation: Reviewer re-review at cfa3c30f1: APPROVE WITH SUGGESTIONS; UTF-8/control safety, runtime propagation proof, genuine overlap proof, label sanitization, specification fidelity, and unchanged approval/audit behavior verified.; Focused validation retained: 111 tests/340 assertions; deptrac 0; phpstan 0; cs-check clean.; Pre-transition git status — clean.
- Summary: Re-review of cfa3c30f1 returned APPROVE WITH SUGGESTIONS. All prior blockers were verified resolved. Remaining notes are non-blocking documentation/minor simplification or theoretical invalid/custom-case-folding edges; no additional surface or behavior is justified by the finalized task.
- Review: role=reviewer; reviewer_run=pi-subagent-result-id-not-exposed; revision=cfa3c30f1; scope=full re-review plus prior blocker verification and specification-fidelity gate; decision=APPROVE WITH SUGGESTIONS

## Task workflow update - 2026-08-28T16:06:42.060Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (80.0s).
- Pushed task/2026-08-20-show-safeguard-trigger-context to origin.
- branch 'task/2026-08-20-show-safeguard-trigger-context' set up to track 'origin/task/2026-08-20-show-safeguard-trigger-context'.
- Created PR: https://github.com/ineersa/agent-core/pull/439
- Validation: Focused SafeGuard/TUI tests: PASS (111 tests, 340 assertions).; castor deptrac: PASS (0 violations/errors).; castor phpstan: PASS (0 errors).; castor cs-check: PASS (clean).; Reviewer: APPROVE WITH SUGGESTIONS at cfa3c30f1.
- Summary: Implementation and review complete at cfa3c30f1. SafeGuard approval prompts now display the exact classified operation input, highlight decision-produced match spans, wrap multiline content without truncation, and neutralize markup/control injection while preserving valid Unicode. Reviewer approved with suggestions after one blocker-fix round.

## Task workflow update - 2026-08-28T17:48:39.628Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #439 received seven inline comments. Required iteration themes: remove wrapper/double regex complexity in SafeGuardCommandMatcher; prefer Symfony Unicode string handling for custom substring evidence; keep the generic question/HITL presentation contract free of SafeGuard-specific naming/coupling so SafeGuard can move out of built-ins; and replace opaque low-level UTF-8/control/span logic with clearer platform-native or simpler code.

## Task workflow update - 2026-08-28T17:51:50.840Z
- Summary: PR feedback architecture pass confirms transport and Extension API are already generic (`ToolCallDecisionDTO::details` and waiting_human payload passthrough); coupling exists in the TUI/wire vocabulary and SafeGuard-specific comments. Before revising, user confirmation is needed on whether to use a new neutral question `presentation` block or encode everything into the existing prompt, whether custom substring policies should remain unhighlighted because they are not regex evidence, and whether terminal controls should be visibly escaped or stripped.
- Scout: role=scout; run=pi-subagent-result-id-not-exposed; revision=cfa3c30f1; scope=generic question/HITL contract, SafeGuard matcher simplification, Symfony String/TUI API availability; outcome=completed
- Ownership: owner=main; fork_run=none; revision=cfa3c30f1; scope=PR #439 feedback redesign and simplification after user resolves generic presentation/control-display behavior; outcome=blocked; commit=none

## Task workflow update - 2026-08-28T17:58:41.344Z
- Summary: User clarified the desired seam: use the existing generic question prompt and Symfony TUI Markdown renderer rather than adding SafeGuard-shaped question presentation fields. HTML is not interpreted by Symfony MarkdownWidget, but Markdown strong/code styling is supported; MarkdownWidget already sanitizes UTF-8, strips terminal controls, and wraps. Revision will generate safely escaped Markdown from SafeGuard decision evidence, remove all TUI/question contract additions, and keep the question stack generic.
- Ownership: owner=main; fork_run=none; revision=cfa3c30f1; scope=PR #439 feedback: collapse evidence into existing Markdown approval prompt, revert SafeGuard-shaped TUI/question surface, simplify matcher/custom Unicode substring handling, focused proof; outcome=assigned; commit=none

## Task workflow update - 2026-08-28T18:03:56.373Z
- Validation: castor test --filter='(SafeGuardCommandMatcherTest|SafeGuardToolCallHookTest)' — PASS (49 tests, 108 assertions).; Virtual TUI proof uses the real SafeGuard-generated Markdown prompt through QuestionController/VirtualTuiHarness; multiline Unicode, literal markup, terminal-control stripping, wrapping, and bold matched ranges verified.; castor deptrac — PASS (0 violations/errors).; castor phpstan — PASS (0 errors).; castor cs-check — PASS (clean).; git diff --check — PASS; worktree clean after commit and origin/main merge.
- Summary: PR feedback revision implemented. SafeGuard now embeds the exact escaped operation input and regex-evidence emphasis directly into its existing approval Markdown prompt. Generic QuestionRequest/runtime/TUI renderer additions were fully removed; Symfony MarkdownWidget supplies UTF-8/control sanitization, Markdown styling, and wrapping. Matcher wrappers/double evaluation and custom-substring regex approximation were removed; built-in regex spans are collected once and merged, while custom patterns use Symfony UnicodeString normalized substring matching and remain unhighlighted because they are not regex evidence. Branch merged latest origin/main cleanly.
- Ownership: owner=main; fork_run=none; revision=cfa3c30f1; scope=PR #439 feedback: collapse evidence into existing Markdown approval prompt, revert SafeGuard-shaped TUI/question surface, simplify matcher/custom Unicode substring handling, focused proof; outcome=completed; commit=347bc1a9a
- Integration: merged origin/main revision 1c658b71b into task branch; merge_commit=925fc2efb; conflicts=none

## Task workflow update - 2026-08-28T18:16:34.462Z
- Validation: castor test --filter='(SafeGuardCommandMatcherTest|SafeGuardToolCallHookTest)' — PASS (50 tests, 110 assertions).; castor deptrac — PASS (0 violations/errors).; castor phpstan — PASS (0 errors).; castor cs-check — PASS (clean).; git diff --check — PASS.
- Summary: Followed reviewer suggestions before repush: switched matched ranges from CommonMark strong delimiters to inline-code styling, which is flanking-safe and uses the existing MarkdownCode theme color; dynamically sizes backtick delimiters and keeps boundary whitespace outside the code span. Encoded plain spaces/tabs as Markdown entities so leading indentation cannot become a code block while rendering the exact whitespace. Settings evidence is labeled Operation, and empty custom patterns retain prior fail-closed match-all behavior.
- Ownership: owner=main; fork_run=none; revision=925fc2efb; scope=non-blocking review edge fixes for CommonMark delimiter flanking, leading whitespace, settings label, and empty custom patterns; outcome=completed; commit=b3a7a444c

## Task workflow update - 2026-08-28T18:24:35.238Z
- Validation: Final reviewer at b3a7a444c: APPROVE.; Focused tests: PASS (50 tests, 110 assertions).; deptrac/phpstan/cs-check: PASS.; Pre-transition worktree: clean.
- Summary: Final reviewer at b3a7a444c returned APPROVE. Confirmed all PR comments resolved, no SafeGuard-specific generic TUI/Extension API surface remains, Markdown rendering and themed inline-code evidence are safe for tested arbitrary input, classification/approval semantics remain intact, and focused virtual proof is at the lowest correct layer.
- Review: role=reviewer; reviewer_run=pi-subagent-result-id-not-exposed; revision=b3a7a444c; scope=final PR-feedback re-review, security/specification fidelity, Markdown rendering and evidence path; decision=APPROVE

## Task workflow update - 2026-08-28T18:26:01.910Z
- Validation: Failed gate: qa-20260828-182446-284643-e42d632d, lane=test, check-test.log.; Failure: RunMessageProcessorLogComponentTest::testCompactionHandlersRetainCompactionComponentThroughProductionRegisteredHandlers; compiled proxy absolute path crossed isolated temp roots in ParaTest worker 3.
- Summary: CODE-REVIEW transition full gate failed in unrelated unit lane after 11.253s: RunMessageProcessorLogComponentTest loaded a compiled PlatformInterface proxy from a different TestDirectoryIsolation root in the same ParaTest worker (`hatfield-test-b395...` referenced from container under `hatfield-test-a276...`). This is a deterministic cache/CWD isolation defect, not a SafeGuard assertion. Per workflow, no blind retry; investigate and fix root cause before re-review.

## Task workflow update - 2026-08-28T18:26:18.342Z
- Recorded fork run: nh5wwdtcadwb
- Ownership: owner=fork; fork_run=nh5wwdtcadwb; revision=b3a7a444c; scope=unrelated CODE-REVIEW unit-gate cache/CWD isolation failure root cause, minimal fix, regression proof, focused/parallel validation; outcome=assigned; commit=none

## Task workflow update - 2026-08-28T18:31:17.605Z
- Recorded fork run: nh5wwdtcadwb
- Validation: castor test --filter=RunMessageProcessorLogComponentTest — PASS (3 tests, 23 assertions).; Relevant RunMessageProcessorLogComponentTest passed in 16-worker ParaTest lane after fix.; castor cs-fix --path=tests/CodingAgent/TestCase — PASS/clean.; castor cs-check — PASS.; castor phpstan — PASS (0 errors).; git diff --check — PASS; worktree clean.
- Summary: Fork nh5wwdtcadwb completed deterministic root-cause fix for the CODE-REVIEW unit gate failure. Both custom isolated-kernel bases bypassed Symfony KernelTestCase teardown but left its shared static kernel/class references alive after deleting the isolated CWD, allowing the next class in one ParaTest worker to mix compiled container/proxy paths from different roots. Commit 60a9d7e51 clears kernel, class, and booted state before removing the isolated tree; no product or SafeGuard code changed.
- Ownership: owner=fork; fork_run=nh5wwdtcadwb; revision=b3a7a444c; scope=unrelated CODE-REVIEW unit-gate cache/CWD isolation failure root cause, minimal fix, regression proof, focused/parallel validation; outcome=completed; commit=60a9d7e511fcda0158dbccb8e31b1f24352fb6c8

## Task workflow update - 2026-08-28T18:39:45.315Z
- Validation: Reviewer at 60a9d7e51 vs b3a7a444c: APPROVE WITH SUGGESTIONS; no blockers.
- Summary: Independent reviewer approved the isolated-kernel teardown fix at 60a9d7e51 with suggestions only. Reviewer confirmed the stale worker-wide KernelTestCase static references exactly explain the mixed removed/current cache roots and that clearing them before directory deletion matches Symfony teardown semantics without changing DAMA or exception-handler behavior. No product/public surface changed.
- Review: role=reviewer; reviewer_run=pi-subagent-result-id-not-exposed; revision=60a9d7e511fcda0158dbccb8e31b1f24352fb6c8; scope=CODE-REVIEW gate-failure isolated-kernel static-state teardown fix and specification-fidelity inventory; decision=APPROVE WITH SUGGESTIONS

## Task workflow update - 2026-08-28T18:40:14.240Z
- Validation: Post-review focused test at 60a9d7e51: PASS (53 tests, 133 assertions) covering RunMessageProcessorLogComponentTest and both SafeGuard suites.; Post-review castor deptrac — PASS (0 violations/errors).; Post-review castor phpstan — PASS (0 errors).; Post-review castor cs-check — PASS (clean).; git diff --check — PASS; worktree clean.

## Task workflow update - 2026-08-28T18:41:34.975Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (68.5s).
- Pushed task/2026-08-20-show-safeguard-trigger-context to origin.
- branch 'task/2026-08-20-show-safeguard-trigger-context' set up to track 'origin/task/2026-08-20-show-safeguard-trigger-context'.
- PR already exists: https://github.com/ineersa/agent-core/pull/439
- Validation: Focused SafeGuard + regression suite — PASS (53 tests, 133 assertions).; castor deptrac — PASS.; castor phpstan — PASS.; castor cs-check — PASS.; Final SafeGuard reviewer — APPROVE at b3a7a444c.; Kernel isolation fix reviewer — APPROVE WITH SUGGESTIONS at 60a9d7e51; no blockers.
- Summary: PR #439 feedback resolved and unrelated CODE-REVIEW gate isolation defect fixed. SafeGuard now embeds exact operation input and actual built-in regex evidence directly in the existing approval Markdown prompt using Symfony TUI themed code styling; generic question/runtime/TUI contracts remain unchanged. Kernel-test harness fix 60a9d7e51 clears stale Symfony KernelTestCase statics before isolated CWD deletion, preventing cross-root compiled proxy reuse under ParaTest. Independent reviews approved both revisions.

## Task workflow update - 2026-08-28T18:41:40.313Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/439
- Updated PR Status: open
- Validation: CODE-REVIEW deterministic castor check — PASS (68.5s) at 60a9d7e51.; Branch pushed; PR #439 updated.

## Task workflow update - 2026-08-28T19:15:43.571Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-20-show-safeguard-trigger-context: ide_close_project returned isError.
- Merged task/2026-08-20-show-safeguard-trigger-context into integration checkout.
- Merge made by the 'ort' strategy.
 .../SafeGuard/Classifier/SafeGuardClassifier.php   |   5 +
 .../Classifier/SafeGuardCommandMatcher.php         | 111 +++++++++++++++------
 .../Builtin/SafeGuard/Policy/SafeGuardDecision.php |   9 ++
 .../Builtin/SafeGuard/SafeGuardToolCallHook.php    |  94 ++++++++++++++++-
 .../Classifier/SafeGuardCommandMatcherTest.php     |  32 ++++++
 .../SafeGuard/SafeGuardToolCallHookTest.php        |  68 +++++++++++++
 .../TestCase/IsolatedKernelTestCase.php            |   9 +-
 .../TestCase/PerMethodIsolatedKernelTestCase.php   |   9 +-
 8 files changed, 306 insertions(+), 31 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-show-safeguard-trigger-context.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-show-safeguard-trigger-context.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #439 state: MERGED.; Pre-merge CODE-REVIEW castor check: PASS (68.5s).
- Summary: PR #439 merged on GitHub at 2026-08-28T19:15:12Z as 8263949a71f749dcf55360a3cc708b56cae7139a. SafeGuard approval prompts now show exact triggering input with themed Markdown highlighting for actual built-in regex evidence; generic question/TUI contracts remain unchanged. The deterministic isolated-kernel cache leak found during the review gate was also fixed.

## Task workflow update - 2026-08-28T19:17:29.781Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/439
- Updated PR Status: merged
- Validation: Post-merge `LLM_MODE=true castor check` — PASS (175.7s), QA run qa-20260828-191547-346091-f528cf9d.; Unit/integration: 4907 tests, 20039 assertions — PASS.; Controller replay: 6 tests, 88 assertions — PASS.; TUI: 8 tests, 59 assertions — PASS.; LLM real: 5 tests, 30 assertions — PASS.; deptrac/phpstan/cs-check/docs/catalog — PASS.; QA leak check and llama-proxy cache guard — PASS.; Integration checkout clean; task worktree removed.; JetBrains project close degraded during cleanup (`ide_close_project` error), but filesystem worktree cleanup succeeded.
- Summary: Task completed. PR #439 merged; task worktree removed. Post-merge integration validation passed all deterministic QA lanes.

## Task workflow update - 2026-08-29T16:09:38.995Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
