# Refine documentation and build a visual architecture guide

## Goal
User requests a full maintained-documentation audit and refinement against current implementation, a representative user-facing README, and a polished diagram-first HTML architecture guide under root architecture/. Cover request-to-response lifecycle, run management, startup/shutdown, background tasks, tools, MCP, extensions, infrastructure, queues, and messages, plus connected subsystems.

Use scouts for cross-module evidence gathering. Main retains editorial integration and architecture coherence. Read all maintained project docs, package docs, and applicable instruction references; inventory historical plans/reports and imported upstream references as historical/reference material rather than rewriting their history. Correct factual documentation drift without changing production behavior or weakening project safety/workflow rules. No settings files or secrets are to be read outside the settings tool. Keep public/model-visible documentation package-safe and within existing validator constraints. Use technical-writing and unslop skills throughout.

Baseline 9f744008c. Prior architecture report .pi/reports/architecture-review-20260905-190304.html is orientation only; current source is evidence. Known docs defect: TUI lane invokes phar_ensure even though testing skill says no PHAR prerequisite.

Implementation should be sequential in the task worktree; independent read-only research can run concurrently. Targeted Castor documentation validation and browser inspection of diagrams/navigation at desktop/mobile are required. Full gate and independent reviewer belong to task-to-pr, not task-start. Do not push/PR without the next phase authorization.

## Acceptance criteria
- Every maintained first-party documentation file has a recorded audit disposition, and factual claims are checked against current owning code/configuration.
- README presents Hatfield to users with accurate installation, first-run setup, supported capabilities, and focused links instead of an internal architecture dump.
- architecture/index.html provides polished navigable HTML documentation with diagrams for startup, request lifecycle, run state/recovery, queues/messages, tool and MCP execution, extensions/background work, storage, providers, and TUI projection.
- Architecture diagrams distinguish process ownership, bus routing, durable events, transient deltas, and operational storage, with source references and explicit failure/cancellation paths.
- Maintained Markdown uses precise consistent language and package-safe links; docs:validate passes without weakening the validator or documentation authority.
- HTML navigation and diagrams render correctly at desktop and mobile sizes; no local absolute source links or secrets are published.
- No production behavior, runtime configuration, or safety policy changes are introduced.

## Workflow metadata
Status: DONE
Branch: task/2026-09-05-refine-documentation-and-build-visual-architecture-guide
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-05-refine-documentation-and-build-visual-architecture-guide
Fork run: agent_a043858d18c7b796
PR URL: https://github.com/ineersa/agent-core/pull/473
PR Status: merged
Started: 2026-09-06T15:49:48+00:00
Completed: 2026-09-06T22:42:07+00:00

## Work log
- Created: 2026-09-06T15:49:28+00:00

## Task workflow update - 2026-09-06T15:49:48+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-05-refine-documentation-and-build-visual-architecture-guide.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-refine-documentation-and-build-visual-architecture-guide.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-05-refine-documentation-and-build-visual-architecture-guide.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-05-refine-documentation-and-build-visual-architecture-guide.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-05-refine-documentation-and-build-visual-architecture-guide.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-05-refine-documentation-and-build-visual-architecture-guide.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-05-refine-documentation-and-build-visual-architecture-guide/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-refine-documentation-and-build-visual-architecture-guide.

## Task workflow update - 2026-09-06T15:56:05+00:00
- Ownership: owner=main; fork_run=none; revision=9f744008c; scope=editorial integration, README, architecture visual guide, audit inventory and targeted validation; outcome=assigned; commit=none
- Read-only scouts returned at 9f744008c: agent_55e110d9060558d2 core/run/tool/storage docs; agent_4e548902da0a2c24 runtime/queues/observability docs; agent_733dd902c825ce1f integration/config/extension docs; agent_58652e0b4f1adc43 TUI/build/testing/workflow docs. Follow-up verification required for scout overstatements and incomplete instruction audit.
- Previous-session scout artifact resumes were rejected by runtime lifetime scoping; fresh read-only scouts were launched instead.

## Task workflow update - 2026-09-06T15:59:03+00:00
- Ownership: owner=fork; fork_run=pending; revision=9f744008c; scope=maintained Markdown documentation refinement excluding root README and architecture directory; outcome=assigned; commit=none
- Research verification corrected unsafe draft claims: fork routes to tool, subagent/agent_resume to agent; scheduler_default is non-durable recurring scheduler, deferred deadlines use durable run_control DelayStamp; compaction failure preserves original messages and resumes Running/Completed unless cancelled; result lookup is not concurrent execution exclusion; extension lifecycle failure isolation remains a separate TODO.

## Task workflow update - 2026-09-06T16:43:53+00:00
- Recorded fork run: agent_a043858d18c7b796
- Ownership: owner=fork; fork_run=agent_a043858d18c7b796; revision=9f744008c; scope=maintained Markdown refinement excluding README and architecture; outcome=completed; commit=740d92fa66dfc4763c27e43ed46135c68da4f944
- Fork read testing skill/tests AGENTS and passed castor docs:validate. Audit inventory /tmp/hatfield-docs-audit-markdown.json. Main now owns README/HTML and integration.

## Task workflow update - 2026-09-06T16:45:35+00:00
- Ownership: owner=fork; fork_run=pending; revision=740d92fa6; scope=root architecture/ diagram-first offline HTML guide and documentation audit inventory; outcome=assigned; commit=none
- Main source spot checks found remaining Markdown wording to correct during integration: ChatScreen does not wire listener registrars, runtime flow needs mapping before stdout, extension jobs are not all Execute* or result messages, MCP per-call cancellation is not generally enforced.

## Task workflow update - 2026-09-06T17:51:49+00:00
- Ownership: owner=fork; fork_run=agent_2a37854a3f8d698d; revision=740d92fa6; scope=architecture draft; outcome=completed; commit=ce05166e8
- Ownership: owner=main; fork_run=none; revision=ce05166e8; scope=README user experience, diagram corrections and coverage, reproducible guide inputs, prose integration and focused validation; outcome=assigned; commit=none

## Task workflow update - 2026-09-06T19:44:39+00:00
- Summary: Added user-supplied GIF and VHS tape unchanged under docs/assets and embedded GIF in README. User declined media inspection; browser child cancelled. Builtin selection reassessment recommends keeping existing 14 core selections, extracting terminal usage/install references and trimming internal session detail; no selection changes yet. Added explicit remote PHAR and native installer examples to README and distribution.

## Task workflow update - 2026-09-06T19:49:46+00:00
- Validation: castor docs:validate PASS: 19 selected documents total, including 15 core docs and 4 Extension API docs; package-safe links and size gate passed.
- Summary: Implemented approved builtin documentation changes. Added bundled installation.md and terminal-usage.md, preserved all existing selected docs, moved session replay measurements and full transition identity matrix to repository-only session-runtime-internals.md, retained user-facing recovery limits and repair safety, and linked new guides from README and owning maintainer docs. Other architecture/build/testing/tool-execution docs remain unbundled. Terminal cancellation/exit wording checked against actual CancelListener and CtrlCInputInterceptor rather than stale hotkey metadata.

## Task workflow update - 2026-09-06T21:33:52+00:00
- User supersedes HTML acceptance criteria: replace architecture HTMLs with Markdown and detailed Mermaid flow/sequence diagrams, using .pi/reports/architecture.md as visual depth/style reference, not source authority. Preserve user-edited README, GIF, licensing, and bundled-doc refinements. Main owns replacement; no new product behavior.

## Task workflow update - 2026-09-06T22:11:14+00:00
- Validation: Browser Mermaid 11.12.2: first pass 6 syntax failures of 40; fixed sequence semicolons. Second pass 40/40 rendered, no overlaps/clipping; all 92 architecture relative links resolved.; After second pass, improved wide layouts: overview simplified, sequence wrapping, end-to-end/startup width60 margin10, logging producer fan-in consolidated, routing/tool registration vertical, scheduler split into two diagrams. Latest 41-diagram revision still needs final browser validation and Castor docs:validate.
- Summary: Replaced rejected architecture HTML/CSS/Python renderer with Markdown Mermaid guide: README, request-lifecycle, processes-and-queues, state-and-recovery, tools-and-mcp, extensions-and-agents, backgrounding, context-and-projection, build-and-observability, logging, documentation-audit. Added bundled docs/tools.md catalog and README link. Preserved user README edits, demo files, LICENSE with Illia Vasylevskyi attribution, new bundled installation/terminal guides and session split. Logging evidence scout agent_24eb151543702b78 read-only; browser agent_d13be7f923a3067c validates Mermaid, not GIF. No commit/push/PR.
- Ownership: owner=main; fork_run=none; revision=ce05166e8; scope=Markdown Mermaid architecture replacement plus tools/logging/subagent/background coverage; outcome=blocked; commit=none
- Remaining: final rendering/legibility and docs validation; source-fidelity spot-checks needed for worker hook ordering and abbreviated event-publication arrows; audit inventory explicitly flags active .pi runtime instructions pending audit. No new delegated work after context-stop request. Main worktree contains all uncommitted edits and HTML deletions; original user GIF/tape in integration checkout untouched.

## Task workflow update - 2026-09-06T22:35:19+00:00
- Validation: git diff --cached --check passed before commit.; Earlier Mermaid revision rendered 40/40; subsequent readability edits not browser-revalidated. Audit inventory retains explicitly documented pending instruction audit.
- Summary: User approved submission and explicitly waived reviewer invocation. Staged documentation, diagrams, demo assets, and LICENSE; committed b81924966. Worktree clean. Proceeding to CODE-REVIEW transition-owned QA gate.
- Reviewer waived by explicit user instruction: "no reviewer needed". No independent approval claimed for b81924966.
- Ownership: owner=main; fork_run=none; revision=b81924966; scope=final documentation submission; outcome=completed; commit=b81924966

## Task workflow update - 2026-09-06T22:38:01+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (141.4s).
- Pushed task/2026-09-05-refine-documentation-and-build-visual-architecture-guide to origin.
- branch 'task/2026-09-05-refine-documentation-and-build-visual-architecture-guide' set up to track 'origin/task/2026-09-05-refine-documentation-and-build-visual-architecture-guide'.
- Created PR: https://github.com/ineersa/agent-core/pull/473
- Summary: Submit documentation revision b81924966. User explicitly requested no reviewer. Worktree committed and clean.

## Task workflow update - 2026-09-06T22:42:07+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-05-refine-documentation-and-build-visual-architecture-guide: ide_close_project returned isError.
- Merged task/2026-09-05-refine-documentation-and-build-visual-architecture-guide into integration checkout.
- Merge made by the 'ort' strategy.
 .agents/skills/castor/SKILL.md                           |  30 ++++++++++--------
 .agents/skills/testing/SKILL.md                          |  32 ++++++++++++++------
 .hatfield/extensions/extension-api/docs/extension-api.md |   5 +++
 LICENSE                                                  |  21 +++++++++++++
 README.md                                                | 264 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++---------------------------------------------------------------------------------------
 architecture/README.md                                   |  82 ++++++++++++++++++++++++++++++++++++++++++++++++++
 architecture/backgrounding.md                            | 145 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 architecture/build-and-observability.md                  |  71 +++++++++++++++++++++++++++++++++++++++++++
 architecture/context-and-projection.md                   | 115 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 architecture/documentation-audit.md                      | 181 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 architecture/extensions-and-agents.md                    | 166 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 architecture/logging.md                                  | 214 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 architecture/processes-and-queues.md                     | 165 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 architecture/request-lifecycle.md                        | 167 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 architecture/state-and-recovery.md                       | 177 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 architecture/tools-and-mcp.md                            | 208 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 config/packages/messenger.yaml                           |   6 ++--
 docs/agents.md                                           |   4 ++-
 docs/ai-catalog.md                                       |  14 ++++++++-
 docs/assets/demo-recorded.tape                           |  16 ++++++++++
 docs/assets/demo.gif                                     | Bin 0 -> 6389771 bytes
 docs/async-runtime-architecture.md                       |  73 ++++++++++++++++++++++++++++++--------------
 docs/background-processes.md                             |   4 +--
 docs/compaction.md                                       |  12 +++++++-
 docs/distribution.md                                     |  42 +++++++++++++++++++++++++-
 docs/installation.md                                     | 111 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 docs/llm-replay.md                                       |   6 ++--
 docs/mcp.md                                              |   2 ++
 docs/session-runtime-internals.md                        |  30 ++++++++++++++++++
 docs/session-storage.md                                  |  51 +++++++++++++++++--------------
 docs/settings-models.md                                  |   9 +++---
 docs/settings.md                                         |   7 +++--
 docs/terminal-usage.md                                   | 110 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 docs/tool-execution.md                                   |  38 ++++++++++++++++-------
 docs/tools.md                                            |  78 +++++++++++++++++++++++++++++++++++++++++++++++
 docs/tui-architecture.md                                 |  20 +++++++++---
 docs/tui-testing.md                                      |  23 ++++++++++----
 src/CodingAgent/Runtime/AGENTS.md                        |   6 ++--
 src/Tui/AGENTS.md                                        |   2 +-
 39 files changed, 2452 insertions(+), 255 deletions(-)
 create mode 100644 LICENSE
 create mode 100644 architecture/README.md
 create mode 100644 architecture/backgrounding.md
 create mode 100644 architecture/build-and-observability.md
 create mode 100644 architecture/context-and-projection.md
 create mode 100644 architecture/documentation-audit.md
 create mode 100644 architecture/extensions-and-agents.md
 create mode 100644 architecture/logging.md
 create mode 100644 architecture/processes-and-queues.md
 create mode 100644 architecture/request-lifecycle.md
 create mode 100644 architecture/state-and-recovery.md
 create mode 100644 architecture/tools-and-mcp.md
 create mode 100644 docs/assets/demo-recorded.tape
 create mode 100644 docs/assets/demo.gif
 create mode 100644 docs/installation.md
 create mode 100644 docs/session-runtime-internals.md
 create mode 100644 docs/terminal-usage.md
 create mode 100644 docs/tools.md
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-refine-documentation-and-build-visual-architecture-guide.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-refine-documentation-and-build-visual-architecture-guide.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR #473 merged as 44cfd29f9630ed6d3141f9ca73c932d45958a6f0. Integration checkout has only the user's original untracked demo.gif and demo-recorded.tape at root; preserve both. Task tracks copies under docs/assets, so no path collision. Allow non-clean integration checkout solely for these known user files.

## Task workflow update - 2026-09-06T22:43:54+00:00
- Updated PR Status: merged
- Validation: LLM_MODE=true castor check: PASS, 162.8s, all 10 lanes including docs:validate, PHAR smoke, replay, TUI, and llm-real.; Reports: var/reports/qa-20260906-224214-8432-318fcbb6. QA leak check and cache guard passed.
- Summary: Post-merge validation passed on integrated revision 20e041ceac9ee1fabcab84ab2f3d7c675f671319. Task worktree removed. Tracked working tree clean; original root demo.gif and demo-recorded.tape remain untracked and untouched. JetBrains close reported degradation during cleanup, but filesystem worktree removal succeeded.
