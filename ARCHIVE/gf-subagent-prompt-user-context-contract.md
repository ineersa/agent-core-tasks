# GF-05: Correct subagent system prompt and user-context contract

## Goal
Fix the effective prompt/message contract for existing subagents independently of the fork tool. Start from current origin/main. Historical fork-MVP changes are forensic references only; do not cherry-pick their deleted tests or commits wholesale.

Confirmed current-main behavior:
- Parent startup emits a SYSTEM.md system message followed by separate `agents_context`, `skills_context`, and `agents_definitions_context` user-context messages.
- Subagents use `AgentPromptBuilder`: project/AGENTS content is incorporated through child prompt construction rather than represented consistently with the parent user-context path; child messages then add skills context, child contract, and task.
- `SystemPromptBuilder::buildToolsListForNames()` falls back from an empty `promptLine` to `- <dynamic/MCP name>: <full definition description>`. MCP/dynamic definitions intentionally have empty promptLine and are already delivered to the provider through the active tool schema path. This unwinds MCP descriptions into child system text in addition to provider schemas.
- APPEND_SYSTEM/prompt-contributor content can contain parent-scope/disallowed tool documentation unless filtered against the exact child allowlist.
- Historical child contracts listed `Allowed tools:` text even though provider tool scope is the authoritative tool contract.

Relevant current-main production surface:
- `src/CodingAgent/Agent/Execution/AgentPromptBuilder.php`
- `src/CodingAgent/Agent/Execution/SubagentExecutionService.php`
- `src/CodingAgent/SystemPrompt/SystemPromptBuilder.php`
- `src/CodingAgent/SystemPrompt/AgentsContextRenderer.php`
- `src/CodingAgent/Skills/SkillsContextBuilder.php`
- `src/CodingAgent/Agent/Context/AgentsContextBuilder.php`
- `src/CodingAgent/Agent/Execution/AgentToolPolicyResolver.php`
- `src/CodingAgent/Agent/Execution/SubagentToolSetResolver.php`
- `src/CodingAgent/Tool/ToolRegistry.php`
- `src/AgentCore/Infrastructure/SymfonyAi/DynamicToolDescriptionProcessor.php`

Historical references only:
- `230c7b851` — removed `Allowed tools:` and Pi-specific text from child contracts; contains both shared subagent and fork-only changes.
- `bf6fb0906` — filtered child append/contributor content against allowed tools; contains fork-oriented wording/logic that must not be copied blindly.
- `f4927c422` — stopped expanding empty MCP/dynamic promptLine into textual descriptions; also contains fork-only composer/layout changes.

Required design direction:
- Provider-visible active tool schemas are the authoritative dynamic/MCP tool catalog.
- Textual system tool listings contain only explicit, non-empty promptLine/guideline content for tools actually allowed to that child; never synthesize MCP schema descriptions into the system prompt.
- Child APPEND_SYSTEM and contributor output must not mention disallowed/parent-only tools.
- Project instructions, skill context, available-agent context (only when the child's capabilities need it), and child contract must use an explicit reviewed user-context layout rather than being concatenated opportunistically into system text.
- The first red specification commit must define the exact consolidated-versus-multiple user-context message layout. Do not implement until the user manually approves that message ordering/content.
- Parent effective prompt/message behavior must be captured as a regression contract and remain unchanged unless the reviewed red specification explicitly changes it.
- Exported/canonical RunStarted messages and provider-visible messages must agree on the effective context; do not prove only an internal builder result.

## Acceptance criteria
- First commit/push contains only reviewed RED specification tests; no production implementation. Accepted tests are immutable during implementation.
- Red test launches a real existing subagent path and inspects canonical RunStarted plus provider-visible messages, not only AgentPromptBuilder return values.
- The reviewed test fixes the exact message order and whether applicable project/skills/available-agent/child-contract content is consolidated or split; implementation cannot later change that test layout.
- Subagent system text contains no actual AGENTS/project body, skills catalog, available-agents catalog, or synthesized full MCP/dynamic schema descriptions.
- Every MCP/dynamic tool allowed to the child remains present in provider tool schemas and callable even though its description is not duplicated into textual system prompt.
- Disallowed tools, including parent-only/fork-only tools, appear in neither provider schemas nor child system/appended contributor text.
- Only tools with explicit non-empty promptLine/guidelines may appear in textual child system tool documentation.
- Project/AGENTS instructions and configured child skills appear exactly once in the reviewed user-context layout.
- Available-agent context appears only when the child has the capability to launch subagents; ordinary subagents must not receive misleading unavailable-agent instructions.
- Child contract does not duplicate an `Allowed tools:` list; runtime tool scope remains authoritative.
- A parent-run regression test captures current parent context/tool behavior and proves subagent fixes do not silently alter it.
- Canonical RunStarted/exported messages and actual provider input agree on role, ordering, and context content.
- All QA uses Castor; include focused live/controller validation because this changes LLM-visible prompts and tool schemas; no historical branch tests are copied.

## Workflow metadata
Status: ARCHIVE
Branch: task/gf-subagent-prompt-user-context-contract
Worktree: /home/ineersa/projects/agent-core-worktrees/gf-subagent-prompt-user-context-contract
Fork run: 4qmv3vgnga3v
PR URL: https://github.com/ineersa/agent-core/pull/282
PR Status: merged
Started: 2026-07-11T21:25:22.929Z
Completed: 2026-07-12T17:15:43.714Z

## Work log
- Created: 2026-07-11T21:12:57.339Z

## Task workflow update - 2026-07-11T21:25:22.929Z
- Moved TODO → IN-PROGRESS.
- Created branch task/gf-subagent-prompt-user-context-contract.
- Created worktree /home/ineersa/projects/agent-core-worktrees/gf-subagent-prompt-user-context-contract.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/gf-subagent-prompt-user-context-contract.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/gf-subagent-prompt-user-context-contract.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/gf-subagent-prompt-user-context-contract.
- Summary: Claimed for task-start. Acceptance criteria require the first commit to contain only RED specification tests and require manual approval of exact child message ordering/content before production implementation.

## Task workflow update - 2026-07-11T21:28:14.757Z
- Summary: Task claimed and worktree created. Read task-workflow/testing skills and tests/AGENTS.md. Scout traced current parent/child prompt paths. Implementation is intentionally paused before the first RED specification commit because acceptance requires manual approval of exact message ordering/content.
- Proposed reviewed child layout: system; optional user-context source=agents_context; optional user-context source=skills_context; optional user-context source=agents_definitions_context only when the exact child allowlist includes subagent capability; user-context source=agent_child_contract; final user task. Omit absent optional contexts rather than emitting empty messages.
- Proposed system contract: child system text contains only child instructions/harness plus explicit non-empty promptLine/guideline content for allowed tools; no project body, skills catalog, available-agent catalog, synthesized dynamic/MCP descriptions, or disallowed-tool contributor/append documentation.
- Proposed canonical contract: keep StartRunInput.systemPrompt populated and equal to the leading system message content (current shape), and require canonical RunStarted messages to exactly match provider-visible message roles/order/content. Parent remains system → agents_context → skills_context → agents_definitions_context → user.
- Blocker/approval needed before launching implementation fork: user must approve consolidated-vs-split child user-context layout and systemPrompt duplication choice. No files or tests have been modified.

## Task workflow update - 2026-07-11T21:43:24.273Z
- Blame result: root/ancestor `<dir>/AGENTS.md` discovery was accidentally removed by commit `fde4cb9896b6bc38b1d5f4d4270891e3d0a6d21f` (`SYSTEM-02: Address PR comments — .agents/ support, proper XML escaping, remove SYSTEM.md additions, clarify architecture`, Illia Vasylevskyi, 2026-05-25). The change replaced direct `<dir>/AGENTS.md` checks with only `<dir>/.hatfield/AGENTS.md` and `<dir>/.agents/AGENTS.md`; tests were rewritten to subdirectory fixtures, masking the regression. Root discovery was originally introduced in `3b7a67128`. No alternate root discovery exists. Child inheritance itself did not change; children copy the parent's `agents_context`, so they inherit the parent discovery omission.

## Task workflow update - 2026-07-11T21:50:42.860Z
- Located original archived task: `/home/ineersa/projects/agent-core-tasks/ARCHIVE/system-02-agents-context-discovery.md`, implemented by PR #52 (`https://github.com/ineersa/agent-core/pull/52`). Original plan: `.pi/plans/toolbox-design-plan.md` §6. Acceptance criteria say discovery checks `~/.hatfield/` first, then walks upward from `{cwd}` to filesystem root; supported filenames `AGENTS.md`/`AGENTS.MD`, first match per directory; global-first then nearest-to-farthest; render one `<project_context>` user-context; never insert AGENTS content into system prompt. Initial commit `3b7a67128` implemented ancestor `<dir>/AGENTS.md`. Review commit `fde4cb989` added `.agents/` support but replaced direct ancestor checks with `.hatfield/.agents` subdirectory-only checks. The archived work log records adding `.agents` precedence, but no requirement explicitly authorizes dropping root `<dir>/AGENTS.md`.

## Task workflow update - 2026-07-11T21:52:47.679Z
- Summary: User approved restoring root/ancestor AGENTS.md discovery and preserving the parent message order for child context: system → agents_context (AGENTS.md project instructions) → skills_context → agents_definitions_context only when child has subagent capability → agent_child_contract → user task. Retain current systemPrompt/leading-system-message shape. Proceeding with the required first RED-specification-only commit; production remains blocked until that commit is reviewed.

## Task workflow update - 2026-07-11T21:55:28.139Z
- User explicitly requires a regression test reproducing the missing AGENTS.md in effective context. RED proof must place a sentinel in bare project-root `AGENTS.md`, start a real parent/subagent path, and assert the sentinel appears exactly once in canonical RunStarted and provider-visible `agents_context` for both parent and inheriting child—not merely test `AgentsContextDiscovery::discover()` output.

## Task workflow update - 2026-07-11T22:00:08.405Z
- Recorded fork run: vpueqgx78rmg
- Validation: Verified commit and stat: 5 test/support files, 1088 insertions, no production files.; Fork reported `castor test --filter=...` exits on first expected failure.; Fork used raw PHPUnit to enumerate failures after Castor stop-on-failure; this is recorded as diagnostic isolation, not gate evidence.
- Summary: RED-only commit 6208d57faf8e47e0f0632b584cee788324175ac1 exists and worktree is clean, but the specification is not accepted yet. Review found material gaps requiring a tests-only correction commit before user approval.
- RED review rejection reasons: parent effective-context test writes `.hatfield/AGENTS.md` rather than the requested bare root `AGENTS.md` and currently fails despite `.hatfield` being supported, indicating a cwd/test-harness mismatch rather than the production regression; child test manually seeds parent `agents_context` instead of proving bare AGENTS discovery flows through a real parent into the child; provider-visible assertions call `AgentMessageConverter` helpers rather than capture actual provider invocation; MCP schema test manually filters a toolbox and passes on current code, so it does not catch the known synthesized-description bug through the real provider processor; no RED coverage proves disallowed APPEND_SYSTEM/contributor tool documentation is removed. Tests remain unapproved/mutable until corrected.

## Task workflow update - 2026-07-11T22:05:50.325Z
- Recorded fork run: 3o8mcp07dmxm
- Validation: Verified HEAD 5076d313b stacked on 6208d57fa; worktree clean; six test files changed in correction.; Effective bare-root integration now writes AGENTS.md before kernel boot and checks canonical parent/child RunStarted plus LlmPlatformAdapter provider capture.; ProviderBoundaryCaptureSupport invokes real LlmPlatformAdapter and DynamicToolDescriptionProcessor with a fake external ModelClient.
- Summary: Correction commit 5076d313b adds real adapter/provider capture and effective bare-root parent→child coverage, but RED set remains unapproved due two test-spec defects requiring one narrow tests-only correction.
- Remaining defects: `testParentStartRunPreservesMessageOrderAndProviderRepresentation` fails because isolated fixtures contain no skills context, an unrelated harness failure; it must provision deterministic skills/agent registries before asserting the parent order or otherwise isolate the current contract correctly. `Gf05ChildAppendContributorLeakContractTest` expects arbitrary contributor text `GF05_CONTRIBUTOR_FORK_LEAK_MARKER` to be entirely removed, which overstates the requirement and would reject benign contributor content; it must emit identifiable disallowed fork promptLine/guideline/catalog documentation plus benign/allowed content, then assert only disallowed documentation is removed. Tests are still not accepted/immutable.

## Task workflow update - 2026-07-11T22:08:09.695Z
- Recorded fork run: gv9plj8h4yxp
- Validation: `castor test --filter='testParentStartRunPreservesMessageOrderAndProviderRepresentation'` — PASS, 15 assertions.; `castor test --filter='testChildAppendModeSelectivelyFiltersDisallowedToolDocumentation'` — expected RED: disallowed fork catalog contributor leaks.; `castor test --filter='testBareRootAgentsMdInParentAndInheritingChildEffectiveContext'` — expected RED: bare AGENTS sentinel absent.; `castor test --filter='testParentStartInjectsBareRootAgentsMdIntoEffectiveContext'` — expected RED: bare AGENTS sentinel absent.; Verified clean worktree at cc0b0564a stacked on tests-only commits 5076d313b and 6208d57fa; latest stat 2 test files, +131/-44.
- Summary: Final tests-only correction commit cc0b0564a verified. RED specification now has deterministic parent context fixtures, exact parent ordering, effective bare-root parent→child coverage, real LlmPlatformAdapter/provider schema capture, and selective contributor filtering expectations. Awaiting user's explicit acceptance before production implementation; accepted test expectations will then be immutable.

## Task workflow update - 2026-07-11T22:19:32.996Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/282
- Updated PR Status: draft
- Summary: At user request, pushed the tests-only RED specification branch and opened draft PR #282 for inline review. Task remains IN-PROGRESS; no CODE-REVIEW transition or Castor gate was attempted because tests are intentionally red and production implementation has not started.
- Draft review PR created outside the normal final task-to-pr transition at explicit user request: https://github.com/ineersa/agent-core/pull/282. The PR body clearly marks it tests-only and intentionally red. Production implementation remains blocked pending review/approval of immutable expectations.

## Task workflow update - 2026-07-11T23:41:01.053Z
- Updated PR Status: draft
- Summary: User reviewed draft PR #282 and approved proceeding. RED expectations in commits 6208d57fa, 5076d313b, and cc0b0564a are now accepted and immutable. Launching production implementation; tests must not be edited to obtain green results.
- Manual RED-spec approval received: “looks okayish, let's go forward”. Production implementation may now begin. Draft PR remains open for review; implementation commit will not be pushed during task-start handoff.

## Task workflow update - 2026-07-11T23:49:40.782Z
- Recorded fork run: p57mt2ht7m8a
- Validation: Verified commit 05fb998d8, 6 files, +215/-47, not pushed.; Reported green: 10-test GF-05 subset, 51 focused regressions, controller replay, llm-real, deptrac, phpstan, changed-path cs-check.; Reported red: child provider-role assertion due Symfony Role enum normalization in test support; parent no-AGENTS freeze due temp directory living beneath newly discoverable worktree AGENTS.md.
- Summary: Production commit 05fb998d8 verified but handoff is not acceptable yet because two approved GF-05 tests remain red. Launching a narrow correction fork for test-harness representation/isolation fixes plus two production architecture corrections; no expectation changes allowed.
- Required correction: normalize Symfony provider Role enum to string in test capture support without changing expected contract; isolate parent no-AGENTS fixture outside repo ancestors while retaining exact assertions. Production cleanup: AgentsContextBuilder must be a required dependency, not optional solely to preserve manually constructed tests; update test harness constructors. Available-agent context decision must be based on the final resolved child allowlist containing `subagent`, not only the definition's requested tool names.

## Task workflow update - 2026-07-11T23:52:55.810Z
- Recorded fork run: 70icl74rq9aj
- Validation: `castor test --filter='Gf05\|AgentsContextDiscoveryBareRootRegressionTest\|ParentPromptUserContextRegressionTest\|SubagentPromptUserContextContractTest'` — PASS, 11 tests / 68 assertions.; `castor test --filter='AgentPromptBuilderTest'` — PASS, 6 tests.; `castor test --filter='SubagentExecutionServiceTest'` — PASS, 22 tests.; `castor test --filter='AgentsContextDiscoveryTest'` — PASS, 15 tests.; `castor test --filter='AgentToolPolicyResolverTest'` — PASS, 3 tests.; `castor test --filter='SubagentToolSetResolverTest'` — PASS, 5 tests.; `castor test:controller-replay` — PASS, 8 tests; teardown PID warnings reported by harness.; `castor test:llm-real` — PASS, 10 tests on production commit 05fb998d8; not rerun after DI/harness/final-allowlist correction 1652b3882.; `castor deptrac` — PASS, 0 violations.; `castor phpstan` — PASS.; `castor cs-check` — PASS after `castor cs-fix`.; Verified HEAD 1652b3882, clean worktree; branch is two production commits ahead of the draft PR remote branch.
- Summary: Implementation complete and committed in worktree. Production commits: 05fb998d8 restores bare AGENTS discovery, child user-context ordering, tool-text/schema contract, and selective append filtering; 1652b3882 completes required AgentsContextBuilder DI, gates agent definitions on final resolved tools, and corrects representation/isolation test harnesses without changing approved expectations. Worktree is clean. Draft PR #282 remains intentionally unupdated; task-to-pr should push/review/gate.

## Task workflow update - 2026-07-12T02:11:32.260Z
- Validation: Reviewer read root AGENTS.md, testing skill, tests/AGENTS.md, task, all 14 changed files, and related production collaborators.; Reviewer verdict: APPROVE WITH SUGGESTIONS; no critical issues.; Reviewer verified bare AGENTS precedence/global exclusion, exact message ordering, project context absent from child system, dynamic/MCP provider schema behavior, final allowlist gating, selective append filtering, parent freeze, and canonical/provider proof.; CODE-REVIEW hygiene fork removed sole untracked manual artifact `hatfield-child-agent_cd21a141f3b5fb65.html`; final worktree clean.
- Summary: CODE-REVIEW reviewer completed with APPROVE WITH SUGGESTIONS. No critical issues or blockers. Reviewer verified all GF-05 contracts; noted nonblocking sanitizer false-positive risk for natural prose shaped like `<disallowed-tool>:` and documentation/test-structure suggestions. Manual-test HTML artifact was confirmed untracked/unreferenced and removed; worktree is clean.

## Task workflow update - 2026-07-12T02:13:07.011Z
- Recorded fork run: 4r0d293socmr
- Validation: `castor test` — PASS, 4,250 tests / 13,898 assertions (~24s).; `castor deptrac` — PASS, 0 violations.; `castor phpstan` — PASS, no errors.; `castor cs-check` — PASS, 0/1,352 files need fixes.; `castor test:llm-real` — PASS, 10 tests / 121 assertions (~29s), generation preflight OK.; `castor clean:cleanup:workers:list` — no stale QA worker candidates; no processes touched.; Verified clean worktree at HEAD 1652b3882 before and after validation.
- Summary: Focused CODE-REVIEW validation completed at HEAD 1652b3882 with all required lanes green and clean worktree. Proceeding to deterministic gate/push via CODE-REVIEW transition; existing draft PR #282 should be updated with the two production commits.

## Task workflow update - 2026-07-12T02:15:16.442Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (111.4s).
- Pushed task/gf-subagent-prompt-user-context-contract to origin.
- branch 'task/gf-subagent-prompt-user-context-contract' set up to track 'origin/task/gf-subagent-prompt-user-context-contract'.
- PR already exists: https://github.com/ineersa/agent-core/pull/282
- Validation: Reviewer: APPROVE WITH SUGGESTIONS, no critical/blocking issues.; Focused `castor test`: PASS (4,250 tests, 13,898 assertions).; Focused `castor test:llm-real`: PASS (10 tests, 121 assertions).; `castor deptrac`, `castor phpstan`, `castor cs-check`: PASS.; GF-05 focused contract: PASS (11 tests, 68 assertions).; `castor test:controller-replay`: PASS (8 tests).
- Summary: Reviewer APPROVE WITH SUGGESTIONS; no blockers. GF-05 implementation complete at 1652b3882 with accepted RED contract green. Focused unit/integration, architecture, static analysis, style, controller replay, and live LLM validation passed. Push local production commits to existing PR #282 and run deterministic Castor gate.

## Task workflow update - 2026-07-12T02:15:32.435Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/282
- Updated PR Status: open
- Validation: Deterministic `castor check` gate — PASS in 111.4s.; Remote PR head verified at 1652b388257494d53015a576bbe769d5ae22022a.; PR #282 marked ready for review; GitGuardian check currently successful.
- Summary: CODE-REVIEW transition complete. Deterministic `castor check` passed in 111.4s, branch pushed at 1652b3882, existing PR #282 updated and marked ready for review (no longer draft).

## Task workflow update - 2026-07-12T02:32:29.871Z
- Summary: PR #282 received two inline owner comments, both classified as blockers. Feedback rejects post-hoc string sanitization because changing prompt lines/guidelines can leak excluded tools, and clarifies the intended architecture: only permanent/static tools belong in textual system prompt; dynamic/MCP tools belong only in provider-visible schemas. User requested recovery of the original system-prompt/tools task for its exact contract before revising implementation.
- Blocker discussion r3565496284: excluded tools must never enter child system prompt; post-render filtering tied to current descriptions/guidelines is not a sound boundary.
- Blocker discussion r3565503032: distinguish permanent/static prompt tools from dynamic tools; MCP/dynamic tools must not be passed into system prompt. Recover archived task/history for authoritative design before implementation.

## Task workflow update - 2026-07-12T02:32:32.783Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Reclaimed for review iteration after owner feedback. Investigating archived system-prompt/tool contract before replacing the incorrect post-hoc sanitizer with the intended permanent-vs-dynamic tool boundary.

## Task workflow update - 2026-07-12T02:45:48.011Z
- Recorded fork run: 4qmv3vgnga3v
- Validation: Correction reviewer: APPROVE; both owner discussions r3565496284 and r3565503032 resolved.; `castor test --filter=ToolRegistryTest` — PASS, 51 tests / 117 assertions.; `castor test --filter=SystemPromptBuilderTest` — PASS, 22 / 74.; Corrected GF-05 structural append contract — PASS, 1 / 6.; GF-05/context filter — PASS, 11 / 69.; `castor test` — PASS, 4,254 tests / 13,908 assertions.; `castor deptrac` — PASS, 0 violations.; `castor phpstan` — PASS.; `castor cs-check` — PASS after `castor cs-fix`.; `castor test:llm-real` — PASS, 10 / 121.; Verified clean worktree at c614e04c1; not pushed before transition.
- Summary: Owner review blockers resolved in commit c614e04c1. Removed all post-render child prompt scrubbing; added permanent-only ToolRegistry subset snapshots for child placeholders; dynamic/MCP/provider descriptions are structurally excluded; opaque APPEND_SYSTEM/contributor markdown remains unparsed. Corrected the superseded GF-05 test premise. Re-review verdict: APPROVE.

## Task workflow update - 2026-07-12T02:48:17.176Z
- Validation: Deterministic gate `test:llm-real` log: tests functionally OK (10/121), exit 1 due 2 PHP warnings at TestDirectoryIsolation.php:124/131.; Warning isolated path: `var/tmp/test-subagent-parallel-32d1e74bb962a457`; `.hatfield` and root still non-empty during teardown.; No cleanup/kill performed; production correction c614e04c1 remains clean and unpushed.
- Summary: CODE-REVIEW transition gate failed only in `test:llm-real`: all 10 tests and 121 assertions passed, but `SubagentParallelLiveE2eTest::testParallelSubagentsReturnArtifactReport` triggered two teardown warnings because `.hatfield` and isolated root remained non-empty during `TestDirectoryIsolation::removeDirectory`. Treating this as lifecycle/teardown bug per project rules; task remains IN-PROGRESS.

## Task workflow update - 2026-07-12T03:48:04.820Z
- Validation: `castor clean:cleanup:workers:list` — no stale QA workers.; `castor clean:cleanup` — removed generated var/tmp artifacts; tracked/untracked git status remained clean.; `castor check` — all lanes passed except controller-replay timeout exit 124; llm-real passed without warnings; cache guard/artifact integrity/leak check passed.; `castor test:controller-replay` immediately afterward — PASS, 8 tests / 112 assertions in 65.3s.; Cross-worktree JUnit timings: GF-05 64.9–68.8s; main recent 67.1–68.6s; atomic worktree up to 70.85s against fixed 75s gate timeout.
- Summary: Post-cleanup gate rerun: prior live-LLM teardown warning did not recur; llm-real passed 10/121. Gate instead timed out controller replay at the fixed 75s lane limit. Standalone controller replay immediately passed 8/112 in 65.3s. Cross-worktree JUnit evidence shows this is repository-wide narrow timeout headroom (recent main/other worktree runs ~61–71s), not GF-05 code or fork residue.

## Task workflow update - 2026-07-12T03:49:19.773Z
- Summary: Merged current `origin/main` (`d1ca0cf23`) into GF-05 without conflicts; merge commit `a68993529`, worktree clean. Main independently includes the repository-wide controller-replay gate correction observed here: timeout increased 75s→90s with documented measured runtime/headroom.
- User requested merge of origin/main. `git fetch origin main` + non-conflicting ort merge completed. Main brought PR #280 image paste, PR #283 session repair, and Castor controller-replay timeout update. No manual conflict resolution or file edits.

## Task workflow update - 2026-07-12T16:55:35.131Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (120.1s).
- Pushed task/gf-subagent-prompt-user-context-contract to origin.
- branch 'task/gf-subagent-prompt-user-context-contract' set up to track 'origin/task/gf-subagent-prompt-user-context-contract'.
- PR already exists: https://github.com/ineersa/agent-core/pull/282
- Validation: Correction reviewer: APPROVE.; Pre-merge full `castor test`: PASS (4,254 tests / 13,908 assertions).; Pre-merge `castor test:llm-real`: PASS (10 / 121).; Pre-merge `castor deptrac`, `castor phpstan`, `castor cs-check`: PASS.; Origin/main merged without conflicts at a68993529; worktree clean.
- Summary: Review correction c614e04c1 and conflict-free origin/main merge a68993529 are ready. Removed untracked manual export `hatfield-session-2.html`; worktree clean. Re-run deterministic gate with main's updated 90s controller-replay budget, then push to existing PR #282.

## Task workflow update - 2026-07-12T16:55:55.268Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/282
- Updated PR Status: open
- Validation: Post-merge deterministic `castor check` — PASS in 120.1s.; Remote PR head verified at a6899352934f62d3f9858636cd02e55855efc24a.; PR remains open and ready for review, not draft.; Both inline owner review threads received resolution replies referencing c614e04c1.
- Summary: PR #282 is now updated at merge HEAD a68993529 and task returned to CODE-REVIEW. Deterministic castor check passed in 120.1s with origin/main's 90s controller-replay budget. Replied to both owner review threads with the c614e04c1 structural resolution.

## Task workflow update - 2026-07-12T17:15:43.714Z
- Moved CODE-REVIEW → DONE.
- Merged task/gf-subagent-prompt-user-context-contract into integration checkout.
- Merge made by the 'ort' strategy.
 .../Agent/Execution/AgentPromptBuilder.php         |  54 +--
 .../Agent/Execution/SubagentExecutionService.php   |  39 +-
 .../SystemPrompt/AgentsContextDiscovery.php        |  36 +-
 .../SystemPrompt/SystemPromptBuilder.php           |  58 +--
 src/CodingAgent/Tool/ToolRegistry.php              |  75 ++++
 src/CodingAgent/Tool/ToolRegistryInterface.php     |  25 ++
 .../Agent/Execution/AgentPromptBuilderTest.php     |   6 +-
 ...05BareAgentsEffectiveContextIntegrationTest.php | 204 ++++++++++
 ...AppendStructuralPermanentSubsetContractTest.php | 138 +++++++
 .../Execution/SubagentExecutionServiceTest.php     |  21 +-
 .../SubagentPromptUserContextContractTest.php      | 438 +++++++++++++++++++++
 .../Support/PipelineCapturingAgentRunner.php       | 132 +++++++
 .../Support/PromptContractTestSupport.php          | 238 +++++++++++
 .../Support/ProviderBoundaryCaptureSupport.php     | 289 ++++++++++++++
 .../ParentPromptUserContextRegressionTest.php      | 296 ++++++++++++++
 ...gentsContextDiscoveryBareRootRegressionTest.php |  96 +++++
 tests/CodingAgent/Tool/ToolRegistryTest.php        |  61 +++
 17 files changed, 2111 insertions(+), 95 deletions(-)
 create mode 100644 tests/CodingAgent/Agent/Execution/Gf05BareAgentsEffectiveContextIntegrationTest.php
 create mode 100644 tests/CodingAgent/Agent/Execution/Gf05ChildAppendStructuralPermanentSubsetContractTest.php
 create mode 100644 tests/CodingAgent/Agent/Execution/SubagentPromptUserContextContractTest.php
 create mode 100644 tests/CodingAgent/Agent/Execution/Support/PipelineCapturingAgentRunner.php
 create mode 100644 tests/CodingAgent/Agent/Execution/Support/PromptContractTestSupport.php
 create mode 100644 tests/CodingAgent/Agent/Execution/Support/ProviderBoundaryCaptureSupport.php
 create mode 100644 tests/CodingAgent/Runtime/InProcess/ParentPromptUserContextRegressionTest.php
 create mode 100644 tests/CodingAgent/SystemPrompt/AgentsContextDiscoveryBareRootRegressionTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/gf-subagent-prompt-user-context-contract.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/gf-subagent-prompt-user-context-contract.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #282 state: MERGED at 2026-07-12T17:15:17Z.; Merged PR head: a6899352934f62d3f9858636cd02e55855efc24a.; Integration checkout on main and clean before DONE transition.; Pre-merge deterministic castor check passed in 120.1s.
- Summary: PR #282 confirmed merged on GitHub at merge commit 48aa14189743d2932ac00c6d8bddad13097813cc. Complete DONE transition, sync integration checkout, and clean task worktree.

## Task workflow update - 2026-07-12T17:19:26.997Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/282
- Updated PR Status: merged
- Validation: PR #282 merged as 48aa14189743d2932ac00c6d8bddad13097813cc.; Focused `castor test:tui --filter=BashCancelFollowUpE2eTest` after transient timeout — PASS (1 test / 1 assertion, 12.9s).; Post-merge `LLM_MODE=true castor check` rerun — PASS (265.5s): test 4,294/14,161; controller replay 8/112; TUI 36/186; llm-real 10/121; deptrac/phpstan/cs-check pass; cache guard/artifact integrity/leak check pass.; Integration checkout clean; task worktree removed.
- Summary: DONE completed. PR #282 merged, integration checkout synced, task worktree removed, IDEA exclusions cleaned, and post-merge deterministic validation passed. First post-merge gate had a transient BashCancelFollowUp TUI settle timeout; the exact focused test passed immediately, and the required full gate rerun passed all lanes.

## Task workflow update - 2026-08-06T20:59:11.458Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
