# Show full ask_human option text with multiline wrapping

## Goal
The TUI presentation for `ask_human` choice questions appears to impose a limited option width and truncates option labels. Users must be able to read every choice in full before selecting it.

Remove width-based text truncation from choice labels. When an option does not fit on one terminal line, wrap it across as many lines as needed within the available TUI width. Preserve the visual association between continuation lines and their option marker/selection state, and keep keyboard navigation/select behavior unchanged.

Use the existing Symfony TUI layout/text-wrapping capabilities and current theme system. Do not add settings, dependencies, horizontal scrolling, or a new choice widget unless the existing implementation cannot support correct wrapping.

## Acceptance criteria
- `ask_human` choice labels are never ellipsized or width-truncated by the TUI; their complete text is visible.
- Options fitting within the available width remain on one line; longer options wrap naturally onto multiple lines.
- Wrapped continuation lines are clearly associated with the correct option and do not collide with option markers, neighboring choices, borders, or surrounding question text.
- The focused/selected style applies coherently to a wrapped option, and existing keyboard navigation, confirmation, cancellation, and returned option value remain unchanged.
- Rendering responds correctly when the terminal is resized and works at representative narrow and wide widths, including Unicode and ANSI-styled labels.
- Automated virtual TUI rendering proof covers full one-line and multiline option labels at constrained widths, and focused Castor validation passes.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-20-wrap-ask-human-options
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-20-wrap-ask-human-options
Fork run: agent_b4410b65e8d3c436
PR URL: https://github.com/ineersa/agent-core/pull/423
PR Status: merged
Started: 2026-08-21T20:58:33+00:00
Completed: 2026-08-23T19:18:52+00:00

## Work log
- Created: 2026-08-20T16:36:38+00:00

## Task workflow update - 2026-08-21T20:58:33+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-20-wrap-ask-human-options.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-wrap-ask-human-options.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-20-wrap-ask-human-options.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-20-wrap-ask-human-options.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-20-wrap-ask-human-options.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-20-wrap-ask-human-options.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-20-wrap-ask-human-options/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-wrap-ask-human-options.
- Summary: Starting implementation of full multiline wrapping for ask_human option labels, preserving selection/input behavior and proving the real TUI path.

## Task workflow update - 2026-08-21T21:02:42+00:00
- Summary: Recon complete: ask_human routes through QuestionController to Symfony SelectListWidget; native widget deliberately truncates every label and exposes no wrapping flag/render callback. Its renderItem and state are private, so styling/subclassing cannot implement true multiline rows. Smallest valid seam is a focused question-only selectable widget using Symfony TextWrapper/ANSI visible widths while preserving logical item-index navigation and existing events/keybindings.
- Implementation scope mapping: full labels + multiline association + coherent selected style → question-only wrapping list; resize → render-time wrapping from RenderContext width; navigation/value unchanged → item-index state and existing SelectEvent/CancelEvent/keybindings; Unicode/ANSI → Symfony TextWrapper/AnsiUtils; no settings/dependencies/protocol changes.
- Required proof: virtual wide→narrow resize with full ANSI/Unicode labels and exact answer selection, plus replay-backed real TmuxHarness ask_human choice journey at narrow width selecting a wrapped option and proving the exact returned value.

## Task workflow update - 2026-08-21T21:27:25+00:00
- Recorded fork run: agent_deff88db0b65827f
- Validation: PASS focused virtual/controller tests: 31 tests / 178 assertions; PASS replay-backed real TmuxHarness ask_human choice journey twice initially: 1 test / 7 assertions; PASS corrected real TmuxHarness journey with canonical exact answer assertion: 1 test / 11 assertions; PASS deptrac; focused phpstan; cs-check; No stale QA workers; JetBrains diagnostics clean
- Summary: Implemented ask_human Choice multiline wrapping at 5de8ae7a3d21b7acaee4e24cc4c894277f15a8d8 using a question-only Symfony TUI selectable widget; native SelectListWidget truncates by design and exposes no wrapping extension point. Follow-up exact-selection proof committed at c52dc7cb56ad7babfeb9018e005df39eea440433.
- Changed 7 files, +757/-13. Added QuestionChoiceListWidget, controller wiring, focused virtual tests, real narrow-width ask_human E2E, and two deterministic replay fixtures.
- Real Tmux proof starts at 56 columns, verifies full head/tail without label ellipsis, selects second wrapped option via Down+Enter, waits for continuation, then structurally asserts canonical events.jsonl agent_command_applied/human_response payload.answer equals the entire long option value.
- Implementation and follow-up fork artifacts: agent_deff88db0b65827f, agent_ec3e593c29ec0ac5. Worktree clean at c52dc7cb56ad7babfeb9018e005df39eea440433.

## Task workflow update - 2026-08-21T21:37:21+00:00
- Summary: User-tested visual follow-up added to finalized scope: distinguish ask_human question text from answers using the existing theme Prompt color, add one row of spacing between overlay blocks/question and answers, and add one blank row between logical answer options while keeping wrapped continuation/description rows attached to their option.
- Recon: QuestionOverlayPromptRenderer already receives TuiTheme but only applies padding; all built-in themes define semantic ThemeColorEnum::Prompt. Symfony TUI ContainerWidget supports native vertical gap. Individual choices are physical lines inside QuestionChoiceListWidget, so their separator must be emitted between logical item groups rather than between wrapped rows. No theme YAML, settings, API, dependency, or card/background changes needed.

## Task workflow update - 2026-08-21T21:49:20+00:00
- Recorded fork run: agent_195d3da8455f753c
- Validation: PASS focused virtual/controller tests (final 8 tests / 151 assertions); PASS replay-backed TmuxHarness exact ask_human journey twice (1 test / 11 assertions); PASS castor deptrac (0 violations); PASS focused castor phpstan and castor cs-check; No stale QA workers; JetBrains diagnostics clean
- Summary: Implemented user visual follow-up at 67a2a988a94ee139209ac14fcda8219988483283: ask_human question text now uses the active theme's existing Prompt color; overlay blocks use native Symfony TUI vertical gap 1; visible logical answers have exactly one blank separator while wrapped continuation/description rows remain attached.
- No new theme tokens/YAML/settings/dependencies or cards/backgrounds. Header remains Accent, question uses Prompt, answers retain list styling.
- ANSI artifact: worktree var/tmp/ask-human-choice-wrap-visual/ask-human-choice-wrap-overlay-20260821-214515.ansi. Worktree clean at 67a2a988a94ee139209ac14fcda8219988483283.

## Task workflow update - 2026-08-22T00:30:21+00:00
- Summary: Task-to-pr reviewer returned REQUEST CHANGES: custom question list accidentally removed inherited Left/Right and Ctrl+B/Ctrl+F page navigation, violating unchanged keyboard behavior; also identified three unused speculative overrides (setItems, setSelectedIndex, getSelectedItem) and redundant default keybinding override. Physical height growth from full wrapping + separators is an accepted consequence of the finalized full-label requirement, not a blocker.
- Reviewer artifact agent_1e6581fc0e236664. Minimal blocker fix: restore parent cursor_left/cursor_right paging fallbacks, remove redundant keybinding/default and unused state API overrides, add navigation regression proof, then focused re-review.

## Task workflow update - 2026-08-22T00:41:31+00:00
- Recorded fork run: agent_53e8b964171fd311
- Validation: PASS castor test focused QuestionChoiceListWidgetTest|QuestionControllerTest: 34 tests / 246 assertions; PASS replay-backed castor test:tui TuiAskHumanChoiceWrapE2eTest: 1 test / 11 assertions; PASS castor deptrac, focused phpstan, cs-check; No stale QA workers; Reviewer APPROVED at f99a3e71a203fd9438a49a8a10d4e2ff3f659bf9
- Summary: Reviewer blockers fixed at f99a3e71a203fd9438a49a8a10d4e2ff3f659bf9: restored inherited Left/Right/Ctrl+B/Ctrl+F paging parity and removed dead widget overrides. Focused re-review APPROVED (agent_930a3337b2630ff5).
- Initial reviewer REQUEST CHANGES artifact agent_1e6581fc0e236664; fix fork agent_53e8b964171fd311; re-review artifact agent_930a3337b2630ff5. Worktree clean and ready for full task-to-pr validation.

## Task workflow update - 2026-08-22T00:47:11+00:00
- Validation: PASS full castor test: 4804 tests / 19608 assertions; PASS full castor test:tui: 42 tests / 370 assertions in 178.7s; PASS castor deptrac: 0 violations; PASS castor phpstan: 0 errors; PASS castor cs-check: 0 files; No stale QA workers
- Summary: Task-to-pr focused/full validation passed at f99a3e71a203fd9438a49a8a10d4e2ff3f659bf9; ready for deterministic Castor gate and PR transition.

## Task workflow update - 2026-08-22T01:07:13+00:00
- Recorded fork run: agent_9c71b2a4bc629e02
- Validation: Target TuiResumeSessionSwitchE2eTest passed 3 consecutive runs (1 test / 7 assertions each); Full parallel castor test:tui passed: 42 tests / 374 assertions in 165.4s; castor cs-check PASS; no stale QA workers; Reviewer APPROVED test synchronization fix
- Summary: First CODE-REVIEW transition gate exposed and root-fixed an existing stale-positive /new E2E wait at test-only commit 85cbda5fbe87e8315e576579bd3ad1675ca912ba. Focused re-review APPROVED (agent_29e4b686fedccbf2); retrying transition only after proof.
- Failed QA: qa-20260822-004724-866238-7c1020c0; all non-TUI lanes passed. Root cause was old welcome/idle/footer satisfying post-/new wait before command transition; fix atomically requires old completion/transcript/session markers gone and fresh idle draft markers present.

## Task workflow update - 2026-08-22T01:10:29+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (176.7s).
- Pushed task/2026-08-20-wrap-ask-human-options to origin.
- branch 'task/2026-08-20-wrap-ask-human-options' set up to track 'origin/task/2026-08-20-wrap-ask-human-options'.
- Created PR: https://github.com/ineersa/agent-core/pull/423
- Validation: Full reviewer APPROVED at f99a3e71a203fd9438a49a8a10d4e2ff3f659bf9; Gate-fix reviewer APPROVED at 85cbda5fbe87e8315e576579bd3ad1675ca912ba; castor test: 4804 tests / 19608 assertions; castor test:tui after synchronization fix: 42 tests / 374 assertions; castor deptrac/phpstan/cs-check PASS; No stale QA workers
- Summary: Implemented full multiline ask_human Choice labels, exact logical selection, theme Prompt question styling, and visual spacing. Reviewer approved after restoring inherited page navigation/removing dead overrides and hardening the proven stale-positive /new E2E transition wait.

## Task workflow update - 2026-08-22T01:32:53+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Addressing three PR #423 review comments: explain keybinding handling, reassess native SelectListWidget visual preservation/clamping concern, and minimize or remove custom widget rewrite/answer spacing if native styling cannot support it.

## Task workflow update - 2026-08-22T02:25:04+00:00
- Summary: User finalized the styling direction for PR #423: delete the custom QuestionChoiceListWidget and restore native Symfony SelectListWidget behavior; retain native ContainerWidget gap only between header, question, and answer-list blocks; keep header accent and question Prompt color; add a question-specific style class and native pseudo-element stylesheet rules for selected/label/description/scroll-info as appropriate; give up blank rows between individual answers because native options are internal rows. Multiline wrapping remains unresolved and is being investigated separately, including why this overlay receives unexpectedly narrow columns.
- Locked decision: no custom select-list state/input/render rewrite for styling.
- Locked decision: no per-answer blank spacing; preserve native picker semantics and appearance.
- Multiline solution must be decided independently and must not reintroduce duplicated native widget behavior.

## Task workflow update - 2026-08-22T02:33:15+00:00
- Summary: Multiline investigation found the concrete cause of the unexpectedly narrow ask_human options: QuestionController always includes `description` even when it is the empty default string. Native SelectListWidget treats any present description as non-null and enters its label+description table path, where `labelColumnWidth = min(30, maxLabelWidth)`. Therefore an empty description silently clamps every choice label to 30 columns at both 80- and 120-column terminals. Omitting the description key when empty restores native full-row width (terminal columns minus selection/chrome, approximately 76 at 80 and 116 at 120). This is separate from true wrapping beyond one terminal row.
- Root cause proven in pinned Symfony SelectListWidget render path: empty-string descriptions activate the 30-column description layout.
- Smallest Hatfield root fix: include `description` only when non-empty; do not change renderer/state/input.
- True multiline for labels wider than the restored full row remains a separate upstream SelectListWidget capability question.

## Task workflow update - 2026-08-22T02:39:06+00:00
- Summary: Finalized reduced scope after PR review: this task will restore native Symfony SelectListWidget and make ask_human choices use the full available row by omitting empty description keys, which currently trigger Symfony's 30-column label+description layout. Keep locked native styling improvements and block-level spacing; remove the local multiline widget and per-answer spacing. True multiline select options deferred to TODO task 2026-08-22-upstream-multiline-select-options.
- Created follow-up TODO task 2026-08-22-upstream-multiline-select-options for a minimal upstream Symfony TUI API/patch; Hatfield-local state-machine duplication is forbidden.
- Current task acceptance is superseded where necessary: full labels now means full available native row without the accidental 30-column clamp; labels exceeding terminal width may remain natively truncated until upstream support exists.

## Task workflow update - 2026-08-22T02:58:29+00:00
- Recorded fork run: agent_63867627e540a35a
- Validation: PASS castor test --filter='QuestionControllerTest|ThemeStyleSheetFactoryTest' — 28 tests, 123 assertions; PASS castor test:tui --filter=TuiAskHumanChoiceFullWidthE2eTest — replay-backed real TmuxHarness, 1 test, 11 assertions; PASS castor deptrac — 0 violations; PASS focused castor phpstan — 0 errors; PASS castor cs-check — 0 files; PASS castor clean:cleanup:workers:list — no stale QA workers; JetBrains diagnostics: 0 errors in QuestionController.php, ThemeStyleSheetFactory.php, and TuiAskHumanChoiceFullWidthE2eTest.php
- Summary: Implemented finalized reduced scope at commit 9c1bfb45e8cc75d41e9057d429551f130cf557d4: deleted custom QuestionChoiceListWidget and its 301-line test, restored native Symfony SelectListWidget, omitted empty descriptions to remove accidental 30-column clamp, retained Prompt/header/block styling, added question-scoped native pseudo-element styles, and replaced multiline E2E with full-width + exact canonical selection proof. Net cleanup in final commit: 202 insertions, 582 deletions; worktree clean and branch one commit ahead of PR.

## Task workflow update - 2026-08-23T18:44:02+00:00
- Recorded fork run: agent_2560e5266ca35831
- Validation: Read/followed merged AGENTS.md, .agents/skills/testing/SKILL.md, and tests/AGENTS.md; PASS focused two QuestionController proofs — 2 tests, 13 assertions, PHPUnit 0.629s / Castor wall 5.59s; PASS full QuestionControllerTest — 26 tests, 96 assertions, PHPUnit 0.656s / Castor wall 5.63s; PASS focused castor phpstan — 0 errors; PASS castor cs-check; PASS castor deptrac — 0 violations; PASS castor clean:cleanup:workers:list — no stale QA workers; JetBrains diagnostics: 0 errors in QuestionController.php and QuestionControllerTest.php
- Summary: Merged origin/main bfbbe11b2 at merge commit 58de7dc55, read the new testing rules/policies, and audited task tests. At commit 621d0334e, deleted the 251-line tmux journey, its two fixtures, and private-rule stylesheet test; slimmed QuestionController coverage to fast deterministic virtual proofs. Task test delta from the pre-audit commit: 14 insertions, 434 deletions. Current complete task diff vs origin/main is 5 files, +117/-8; no task-added E2E/process/timing tests remain.

## Task workflow update - 2026-08-23T19:11:57+00:00
- Recorded fork run: agent_b4410b65e8d3c436
- Validation: PASS focused proof — 2 tests, 18 assertions, Castor wall 5.527s; PASS full QuestionControllerTest — 26 tests, 101 assertions, Castor wall 5.523s; PASS focused castor phpstan — 0 errors; PASS castor cs-check; PASS castor deptrac — 0 violations; PASS worker list — no stale QA workers; Reviewer agent_2166484dd3821d3b: APPROVED
- Summary: Reviewer blockers fixed at 78cd6b494: actual in-process rendered proof now verifies scoped native selected Accent+bold styling; Prompt/Accent ANSI prefixes have non-vacuous guards; whitespace-only descriptions are omitted without changing meaningful values; redundant vertical direction and selected:focus rule removed. Focused re-review at 78cd6b494 returned APPROVED with no issues. Full current task diff reviewed against origin/main; slow task E2E remains deleted.

## Task workflow update - 2026-08-23T19:13:30+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (74.1s).
- Pushed task/2026-08-20-wrap-ask-human-options to origin.
- branch 'task/2026-08-20-wrap-ask-human-options' set up to track 'origin/task/2026-08-20-wrap-ask-human-options'.
- PR already exists: https://github.com/ineersa/agent-core/pull/423
- Validation: Focused QuestionController proofs: 2 tests, 18 assertions, 5.527s Castor wall; QuestionControllerTest: 26 tests, 101 assertions, 5.523s Castor wall; Focused PHPStan, CS check, Deptrac pass; no stale QA workers; Final reviewer APPROVED
- Summary: Merged latest origin/main, adopted the new hard test policies, deleted slow/duplicate task E2E coverage, retained fast deterministic virtual proof, and received final reviewer approval at 78cd6b494. Updating PR #423.

## Task workflow update - 2026-08-23T19:18:52+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-20-wrap-ask-human-options: ide_close_project returned isError.
- Merged task/2026-08-20-wrap-ask-human-options into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Question/QuestionController.php            | 23 ++++++++++++++++++-----
 src/Tui/Question/QuestionOverlayPromptRenderer.php |  9 +++++++--
 src/Tui/Screen/ChatScreen.php                      |  1 +
 src/Tui/Transcript/ThemeStyleSheetFactory.php      | 17 +++++++++++++++++
 tests/Tui/Question/QuestionControllerTest.php      | 92 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++--
 5 files changed, 133 insertions(+), 9 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-wrap-ask-human-options.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-wrap-ask-human-options.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #423 state MERGED; mergedAt 2026-08-23T19:17:36Z; PR merge commit 1462f445c80d5b8d4122ed54b3ce931e91e95d76
- Summary: Verified PR #423 merged on GitHub at 1462f445c80d5b8d4122ed54b3ce931e91e95d76. Moving task to DONE and cleaning its worktree. Integration checkout contains unrelated untracked hatfield-session-41.html, which is preserved untouched.

## Task workflow update - 2026-08-23T19:20:57+00:00
- Validation: PASS LLM_MODE=true castor check qa-20260823-191919-8215-07691500 — all 9 lanes, 143.6s; Unit: 4795 tests, 19526 assertions; Controller replay: 6 tests, 92 assertions; TUI: 8 tests, 59 assertions; llm-real: 5 tests, 30 assertions; Deptrac/PHPStan/CS/docs/catalog all PASS; QA leak check PASS; llama-proxy cache unchanged 383→383; Task worktree removal confirmed
- Summary: Post-merge integration validation passed on main. Task worktree and IDEA exclusions were removed. Unrelated untracked hatfield-session-41.html remains untouched; integration checkout is therefore not fully clean for unrelated reasons.

## Task workflow update - 2026-08-29T16:09:38.942Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
