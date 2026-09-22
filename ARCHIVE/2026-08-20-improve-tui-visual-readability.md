# Improve TUI visual rhythm and readability

## Goal
The TUI currently feels cramped and is difficult to scan during active agent work. Use the captured ANSI snapshots in `/home/ineersa/projects/agent-core/.hatfield/tmp/tui/snapshots/` as the starting reference, especially the dense handoff, tool-call, subagent, working-status, and chrome regions.

Improve visual hierarchy using Symfony TUI's existing/native styling capabilities and the current theme system. Favor modest whitespace, margins/padding, typography, and alignment over introducing card-style containers.

Specific observations from the snapshots:
- Adjacent content blocks need clearer separation so handoffs, reasoning/status text, tool calls, subagents, and working state are easier to track.
- Collapsed-content text such as `… 113 more lines` can use subdued italic styling where supported.
- Leading glyphs/spinners and tool-call markers appear poorly aligned with their text; improve perceived vertical alignment/rhythm using supported terminal layout/styling (not CSS-style line-height assumptions).
- Preserve information density and the existing single-column architecture.

Out of scope:
- Full card treatment, pervasive backgrounds, or heavy borders around every block.
- A new styling framework, dependency, settings surface, or broad TUI redesign.
- Changing runtime behavior or the information shown.

## Acceptance criteria
- Distinct transcript/runtime blocks have consistent, modest spacing that makes the active flow easier to scan without materially wasting terminal space.
- Collapsed-content indicators (for example, `… 113 more lines`) are visually secondary and italicized when Symfony TUI/terminal capabilities support it.
- Glyphs, spinners, and tool-call markers have consistent spacing/alignment with their associated text and no longer look vertically detached.
- The result uses existing Symfony TUI and Hatfield theme/layout APIs; no card-style backgrounds, pervasive boxed panels, or new dependency/settings surface is introduced.
- The single-column layout and all existing content/actions remain intact across narrow and wide terminal sizes.
- Relevant automated TUI rendering proof is updated at the lowest correct layer, and focused Castor validation passes.
- Before/after ANSI snapshots demonstrate improved readability for the handoff/tool-call/subagent flow represented by the referenced captures.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-20-improve-tui-visual-readability
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-20-improve-tui-visual-readability
Fork run: agent_864ebfe9fa952eb0
PR URL: https://github.com/ineersa/agent-core/pull/422
PR Status: merged
Started: 2026-08-21T13:55:23+00:00
Completed: 2026-08-21T19:31:36+00:00

## Work log
- Created: 2026-08-20T15:45:09+00:00

## Task workflow update - 2026-08-21T13:55:23+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-20-improve-tui-visual-readability.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-improve-tui-visual-readability.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-20-improve-tui-visual-readability.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-20-improve-tui-visual-readability.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-20-improve-tui-visual-readability.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-20-improve-tui-visual-readability.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-20-improve-tui-visual-readability/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-improve-tui-visual-readability.
- Summary: Starting TUI readability task: modest block spacing, secondary italic collapsed indicators, marker/glyph alignment, preserved single-column density, and replay-backed TmuxHarness before/after proof.

## Task workflow update - 2026-08-21T13:58:58+00:00
- Summary: Recon complete. Central render path is TranscriptBlockWidgetFactory → TranscriptBlockRenderer / TranscriptToolRenderer / SubagentResultRenderer, with collapsed previews from TranscriptLinePreviewService and working spinner in ChatScreen. Symfony TUI 8.1 provides native ContainerWidget Style gap/padding, italic/dim Style, and ANSI-aware visibleWidth. Existing replay journeys TuiRichTranscriptProductValidationE2eTest and TuiSubagentProgressE2eTest are the required real-terminal proof paths.
- Specification mapping: modest inter-block spacing → existing container gap/padding; collapsed indicators → existing preview ellipsis styled dim/muted+italic; glyph/tool/spinner alignment → shared prefixes/native width-aware layout; no cards/backgrounds/settings/dependencies/runtime changes.
- Testing plan: virtual render assertions at narrow and wide widths plus replay-backed TmuxHarness rich-transcript and subagent journeys; capture explicit before/after ANSI artifacts from identical checkpoints.

## Task workflow update - 2026-08-21T14:26:27+00:00
- Recorded fork run: agent_43dc605aa7e23895
- Validation: PASS focused virtual: 48 tests, 244 assertions; PASS replay-backed TmuxHarness rich transcript + subagent: 2 tests, 28 assertions at 120x60; PASS replay-backed queued steer: 1 test, 5 assertions; PASS castor deptrac/phpstan/cs-check; PASS castor check qa-20260821-141939-3822-2ed8bbfe; all 9 lanes; No stale QA workers; JetBrains diagnostics: no errors in new ellipsis helper or transcript mounted widget
- Summary: Implemented and committed TUI visual rhythm/readability at ba50f18b03888d174515e63d56dfa5f14e758106. Uses one native vertical gap between distinct transcript nodes, shared italic secondary preview ellipsis styling, and ANSI-width-aligned progress/approval glyph prefixes. Preserves single-column content/actions and adds no settings/dependencies/runtime changes.
- Real before artifacts: worktree var/tmp/tui-visual-readability/before/snapshot-ansi-20260820-*.ansi.
- Real after artifacts: worktree var/tmp/tui-visual-readability/after/rich-transcript-product-validation-history-*.{ansi,txt} and subagent-progress-resume-history-*.{ansi,txt}.
- Verified committed real TmuxHarness proofs in TuiRichTranscriptProductValidationE2eTest and TuiSubagentProgressE2eTest; generated ANSI artifacts remain ignored/uncommitted as intended.

## Task workflow update - 2026-08-21T16:01:54+00:00
- Summary: User extended finalized scope: make tool argument keys brighter with theme-specific colors that remain distinct from each theme's tool-name/title accent. Background treatment remains out of scope for now.
- Implementation must first fix ToolArgumentColoredFormatter to honor existing ToolArgumentKey/ToolArgumentValue theme tokens; then assign harmonious existing theme variables per built-in theme rather than using accent/tool-title or introducing new settings/tokens.

## Task workflow update - 2026-08-21T16:18:24+00:00
- Recorded fork run: agent_b9c39037f8e6eb71
- Validation: PASS focused formatter/theme/virtual: 27 tests, 99 assertions; PASS replay-backed TmuxHarness rich transcript at 120x60: 1 test, 16 assertions; PASS deptrac/phpstan/cs-check; PASS castor check qa-20260821-161238-26662-da8a854e; all 9 lanes; No stale QA workers; JetBrains diagnostics: no errors in formatter or tool renderer
- Summary: Added brighter theme-specific tool argument key colors at cd8092f3781651ea1be3870e3fd653dc08eb2785. Fixed runtime formatter to honor existing ToolArgumentKey/Value tokens and made edit/write path labels use the same semantics. Tool titles remain distinct; backgrounds remain untouched.
- Theme choices: cyberpunk neon magenta vs electric cyan; tokyo-night magenta vs blue; nord nord13 yellow vs nord8 cyan; gruvbox orange vs aqua; catppuccin peach vs mauve; oh-p-dark yellow vs cyan.
- AFTER ANSI: worktree var/tmp/tui-visual-readability/after/rich-transcript-product-validation-history-20260821-160839.ansi; generated artifact intentionally ignored.

## Task workflow update - 2026-08-21T17:20:55+00:00
- Summary: task-to-pr review started from clean worktree at cd8092f3781651ea1be3870e3fd653dc08eb2785. Reviewer returned REQUEST CHANGES: subagent handoff italic ANSI is stripped by MarkdownWidget; tool-result ellipsis styling relies on two content-sniffing heuristics that misstyle legitimate ellipses; view_image exchange arguments bypass ToolArgumentKey. Review fixes required before PR transition.
- Background experiments and local theme override are fully rolled back; branch contains only committed finalized task scope.
- Reviewer artifact: agent_e5d586b1932714f8. Spacing, glyph alignment, theme mapping, architecture, and Tmux/virtual proof were otherwise judged sound.

## Task workflow update - 2026-08-21T17:36:04+00:00
- Recorded fork run: agent_d112c4bcee6e3925
- Validation: PASS focused unit/virtual: 42 tests, 209 assertions; PASS focused replay-backed TmuxHarness rich transcript + subagent: 2 tests, 35 assertions; PASS deptrac; focused phpstan; cs-check; No stale QA workers
- Summary: Review blockers fixed at 99194ffe291881c97964a5d50409fe4d22105579: subagent handoff ellipsis now renders as a styled sibling outside MarkdownWidget; tool result preview keeps generated ellipsis structurally separate from legitimate U+2026 content; view_image path uses ToolArgumentKey/Value.
- Fix fork preserved finalized scope and did not reintroduce rejected backgrounds. Working tree clean; awaiting re-review.

## Task workflow update - 2026-08-21T17:51:16+00:00
- Recorded fork run: agent_fe27f03cbb45075d
- Validation: PASS focused revalidation: 20 tests, 133 assertions; PASS full castor check qa-20260821-174413-43764-47ba9666 on final commit: all 9 lanes; Unit 4795/19448; controller replay 13/252; TUI 41/361; llm-real 13/144; deptrac/phpstan/cs/docs/catalog PASS; No stale QA workers
- Summary: Final review cleanup completed despite accidental cancellation: dead ViewImageTranscriptFormatter::formatToolCallLines and its sole test removed at 0ac06b67632cfd5076c5146e17cac9c817db9a95; tree clean.

## Task workflow update - 2026-08-21T17:57:32+00:00
- Summary: Final reviewer APPROVED at 0ac06b67632cfd5076c5146e17cac9c817db9a95. All prior blockers resolved; full final-tree castor check green; rejected backgrounds absent; worktree clean.
- Final reviewer artifact: agent_c233fc2db5715144. Non-blocking future cleanup only: deduplicate ANSI artifact persistence helper when those E2Es are next touched.

## Task workflow update - 2026-08-21T18:10:54+00:00
- Recorded fork run: agent_864ebfe9fa952eb0
- Validation: Focused failing TUI test PASS 3 consecutive runs (1/5 each); Full parallel castor test:tui PASS 41 tests / 365 assertions in 165.7s; cs-check PASS; no stale QA workers
- Summary: First CODE-REVIEW transition auto-check failed at a proven hardcoded parallel budget mismatch in TuiResumeSessionSwitchE2eTest post-/new readiness. Fixed test-only at 9db3536c08cf1168798b74475215b0070bcc91fa by using existing 20s parallel callback budget; no product changes.
- Failed gate evidence: qa-20260821-175746-56235-ab70795f, same boundary as prior qa-20260821-160906. Scouts found no stale workers/resource holder or product-code path; timeout was the concrete mismatch.

## Task workflow update - 2026-08-21T18:16:21+00:00
- Summary: Focused re-review APPROVED for test-only parallel budget fix at 9db3536c08cf1168798b74475215b0070bcc91fa; predicate, choreography, and isolation assertions unchanged.
- Reviewer artifact agent_3bb89f872e3bf766. Final tree clean and ready for CODE-REVIEW transition.

## Task workflow update - 2026-08-21T18:19:43+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (181.3s).
- Pushed task/2026-08-20-improve-tui-visual-readability to origin.
- branch 'task/2026-08-20-improve-tui-visual-readability' set up to track 'origin/task/2026-08-20-improve-tui-visual-readability'.
- Created PR: https://github.com/ineersa/agent-core/pull/422
- Validation: Final full castor check qa-20260821-174413-43764-47ba9666 PASS before one-line test budget fix; Post-fix focused failing TUI test PASS x3; Post-fix full parallel castor test:tui PASS 41 tests / 365 assertions; Focused final reviewer APPROVED (agent_3bb89f872e3bf766); No stale QA workers
- Summary: Final reviewer APPROVED. Visual rhythm, italic collapsed indicators, ANSI-aware glyph alignment, and brighter distinct theme argument keys complete; background experiments rejected and rolled back. Final SHA 9db3536c08cf1168798b74475215b0070bcc91fa.

## Task workflow update - 2026-08-21T19:31:36+00:00
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-improve-tui-visual-readability.
- Merged task/2026-08-20-improve-tui-visual-readability into integration checkout.
- Merge made by the 'ort' strategy.
 config/themes/catppuccin-mocha.yaml                         |   2 +-
 config/themes/cyberpunk.yaml                                |   2 +-
 config/themes/gruvbox-dark.yaml                             |   2 +-
 config/themes/nord.yaml                                     |   2 +-
 config/themes/oh-p-dark.yaml                                |   2 +-
 config/themes/tokyo-night.yaml                              |   2 +-
 src/Tui/Transcript/EditToolCallDiffRenderer.php             |   2 +-
 src/Tui/Transcript/PendingMessagesWidget.php                |   2 +-
 src/Tui/Transcript/SubagentResultRenderer.php               |  46 +++++++++++++-----------
 src/Tui/Transcript/ToolArgumentColoredFormatter.php         |  11 +++---
 src/Tui/Transcript/TranscriptGlyphs.php                     |   8 ++---
 src/Tui/Transcript/TranscriptMountedWidget.php              |   4 +++
 src/Tui/Transcript/TranscriptPreviewEllipsis.php            |  34 ++++++++++++++++++
 src/Tui/Transcript/TranscriptToolRenderer.php               | 142 ++++++++++++++++++++++++++++++++++++++++++++------------------------------
 src/Tui/Transcript/ViewImageTranscriptFormatter.php         |  15 --------
 src/Tui/Transcript/WriteToolCallContentRenderer.php         |   2 +-
 tests/Tui/E2E/TmuxHarness.php                               |  16 +++++++++
 tests/Tui/E2E/TuiQueuedSteerE2eTest.php                     |   2 +-
 tests/Tui/E2E/TuiResumeSessionSwitchE2eTest.php             |   2 +-
 tests/Tui/E2E/TuiRichTranscriptProductValidationE2eTest.php |  63 ++++++++++++++++++++++++++++++++-
 tests/Tui/E2E/TuiSubagentProgressE2eTest.php                |  48 ++++++++++++++++++++++++-
 tests/Tui/Screen/TuiTranscriptBlocksVirtualRenderTest.php   | 298 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 tests/Tui/Support/SubagentProgressEventsFixture.php         |  20 ++++++++++-
 tests/Tui/Theme/ThemePaletteTest.php                        |   5 +--
 tests/Tui/Theme/ThemeRegistryTest.php                       |  28 +++++++++++++++
 tests/Tui/Transcript/SubagentResultRendererTest.php         |  28 +++++++++++++++
 tests/Tui/Transcript/ToolArgumentColoredFormatterTest.php   |  49 ++++++++++++++++++++++++++
 tests/Tui/Transcript/ViewImageTranscriptFormatterTest.php   |  12 -------
 28 files changed, 720 insertions(+), 129 deletions(-)
 create mode 100644 src/Tui/Transcript/TranscriptPreviewEllipsis.php
 create mode 100644 tests/Tui/Transcript/ToolArgumentColoredFormatterTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-improve-tui-visual-readability.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-improve-tui-visual-readability.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #422 state verified MERGED
- Summary: PR #422 merged on GitHub at 17c1605faad4a41cc4fdb671f568393f7abf9a6e. Moving task to DONE and cleaning task worktree.

## Task workflow update - 2026-08-21T19:34:55+00:00
- Validation: LLM_MODE=true castor check qa-20260821-193147-161210-423c3065 PASS all 9 lanes; Unit 4795/19448; controller replay 13/252; TUI 41/365; llm-real 13/144; deptrac/phpstan/cs-check/docs/catalog PASS; QA leak check PASS; llama-proxy cache unchanged
- Summary: Post-merge integration validation passed on main at 9ab93ef881dc6c4242edaf1c2853ca0d89962b3f.

## Task workflow update - 2026-08-21T21:49:23+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
