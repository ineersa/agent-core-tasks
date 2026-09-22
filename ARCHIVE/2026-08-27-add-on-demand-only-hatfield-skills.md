# Add on-demand-only Hatfield skills hidden from model discovery

## Goal
## Goal

Add a skill metadata option/convention so selected Hatfield skills do not appear in the model-visible skill catalog and are not autonomously selected, while remaining loadable explicitly through `/skill:<name>` and through an agent definition's `skills` frontmatter (for example, the Datadog skill attached to `datadog-logs`).

## Context

The user observed the equivalent capability in Matt Pocock's skills (for example `skill:improve-codebase-architecture`). During task-explain, verify the upstream field name and exact semantics rather than guessing or inventing an incompatible frontmatter key.

Desired use case:
- Large, narrow, or agent-specific skills should not spam the general model skill catalog.
- A user can still explicitly load one on demand with `/skill:<name>`.
- A specialist agent can still receive it through `skills: [<name>]`.
- Ordinary discoverable skills retain current behavior by default.

## Scope

- Skill frontmatter parsing/DTO/schema and discovery/catalog projection.
- Explicit `/skill:<name>` resolution and agent-bound skill loading.
- Documentation for discoverable versus on-demand-only skills.
- Deterministic tests at the lowest correct layer.
- Consider whether bundled, project, and user skills should share identical semantics; prefer one consistent rule.

## Boundaries

- Do not rename or redesign existing skill loading without need.
- Do not silently hide existing skills; the new behavior must be opt-in unless an explicit migration is approved.
- Do not introduce aliases or fallback parsing for speculative compatibility.
- Do not couple the feature to Datadog; Datadog is only the motivating example.
- If upstream conventions distinguish model invocation from user invocation, preserve that distinction and discuss any Hatfield-specific mapping before implementation.

## Open questions for task-explain

1. What exact frontmatter field and behavior does the referenced upstream skill system use?
2. Should an on-demand-only skill be omitted entirely from the model-visible catalog, or represented by a minimal name-only entry?
3. Should direct agent attachment always override hidden/discovery metadata? (Expected: yes.)
4. Does `/skill:<name>` already resolve hidden skills by name, or does command routing need adjustment?
5. Should Hatfield support separate controls for model discovery and user invocation if the upstream convention has both?

## Acceptance criteria
- A documented opt-in skill metadata option prevents a selected skill from appearing in the model-visible discovery catalog and from autonomous model selection.
- The same hidden/on-demand-only skill remains explicitly loadable via `/skill:<name>`.
- An agent definition using `skills: [name]` still receives the skill even when it is hidden from general model discovery.
- Skills without the option preserve current discovery and loading behavior.
- The implementation follows the verified upstream convention where compatible, or documents an explicitly approved Hatfield-specific difference.
- Tests cover discovery omission, explicit slash loading, agent-attached loading, default behavior, malformed metadata, and precedence/duplicate discovery where relevant.
- Any user-visible slash-command behavior is proven at the lowest correct TUI layer; do not default to tmux when virtual/in-process proof is sufficient.
- Documentation explains when to use discoverable versus on-demand-only skills and includes a specialist-agent example.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-27-add-on-demand-only-hatfield-skills
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-27-add-on-demand-only-hatfield-skills
Fork run: 5u1s9536gzfn
PR URL: https://github.com/ineersa/agent-core/pull/443
PR Status: merged
Started: 2026-08-29T16:55:16.594Z
Completed: 2026-08-29T23:50:20.715Z

## Work log
- Created: 2026-08-27T23:57:36.666Z

## Task workflow update - 2026-08-29T16:55:16.594Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-27-add-on-demand-only-hatfield-skills.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-27-add-on-demand-only-hatfield-skills.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-27-add-on-demand-only-hatfield-skills.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-27-add-on-demand-only-hatfield-skills.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-27-add-on-demand-only-hatfield-skills.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-27-add-on-demand-only-hatfield-skills.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-27-add-on-demand-only-hatfield-skills/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-27-add-on-demand-only-hatfield-skills.
- Summary: Started task-start phase. Preparing a shallow routing pass and implementation ownership assignment.

## Task workflow update - 2026-08-29T16:55:58.558Z
- Summary: Verified the upstream convention: Claude Code uses `disable-model-invocation: true` to omit a skill description from model context while preserving explicit user invocation. Hatfield will use the same field. Per the task acceptance criteria, explicit agent `skills` attachment remains an authoritative load path even though current Claude Code documentation says the flag prevents subagent preloading.
- Ownership: owner=fork; fork_run=pending; revision=d3c879397ad9a802a80359b025b935568d64086e; scope=implement `disable-model-invocation` skill metadata across parsing, model catalog omission, explicit slash loading, agent-attached loading, documentation, and deterministic focused tests; outcome=assigned; commit=none

## Task workflow update - 2026-08-29T16:56:16.168Z
- Recorded fork run: 5u1s9536gzfn
- Ownership: owner=fork; fork_run=5u1s9536gzfn; revision=d3c879397ad9a802a80359b025b935568d64086e; scope=implement `disable-model-invocation` skill metadata across parsing, model catalog omission, explicit slash loading, agent-attached loading, documentation, and deterministic focused tests; outcome=assigned; commit=none

## Task workflow update - 2026-08-29T17:06:18.195Z
- Recorded fork run: 5u1s9536gzfn
- Validation: castor test --filter='SkillDiscoveryTest|SkillsContextBuilderTest|SkillRegistryTest|SkillCommandRegistrarTest|CommandParserTest::testSlashSkillColonName|PromptTemplateExpansionInProcessTest::testStartExpandsOnDemandOnlySkillCommand' — PASS; 53 tests, 169 assertions; castor phpstan (scoped Skills/SkillCommandRegistrar/InProcess/Contract) — PASS; 0 errors; castor cs-fix (touched paths) — PASS; castor docs:validate — PASS; includes skills; castor deptrac — PASS; 0 violations; castor check — NOT RUN; task-start forbids it
- Summary: Implemented disable-model-invocation: true across discovery/catalog, /skill: registration+expansion, agent buildFor attachment, docs/skills.md, and focused tests. Committed on task branch.
- Ownership: owner=fork; fork_run=5u1s9536gzfn; revision=d3c879397ad9a802a80359b025b935568d64086e; scope=implement disable-model-invocation skill metadata across parsing, model catalog omission, explicit slash loading, agent-attached loading, documentation, and deterministic focused tests; outcome=completed; commit=pending-after-verify

## Task workflow update - 2026-08-29T17:06:22.237Z
- Ownership: owner=fork; fork_run=5u1s9536gzfn; revision=d3c879397ad9a802a80359b025b935568d64086e; scope=implement disable-model-invocation skill metadata across parsing, model catalog omission, explicit slash loading, agent-attached loading, documentation, and deterministic focused tests; outcome=completed; commit=c1a33cfc79d02f58bdc8860c4ce2d362160e19e7

## Task workflow update - 2026-08-29T17:07:00.541Z
- Recorded fork run: 5u1s9536gzfn
- Validation: Fork confirmed it read and followed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before test work.; `castor test --filter='SkillDiscoveryTest|SkillsContextBuilderTest|SkillRegistryTest|SkillCommandRegistrarTest|CommandParserTest::testSlashSkillColonName|PromptTemplateExpansionInProcessTest::testStartExpandsOnDemandOnlySkillCommand'` — PASS, 53 tests, 169 assertions.; Focused `castor phpstan --path=…` for touched Skills, Runtime Contract, TUI listener, and in-process client paths — PASS, 0 errors.; Focused `castor cs-fix --path=…` for touched paths — PASS.; `castor docs:validate` — PASS.; `castor deptrac` — PASS, 0 violations.; Parent verified clean worktree, expected commit c1a33cfc79d02f58bdc8860c4ce2d362160e19e7, and zero IDE error diagnostics in SkillDiscovery.php and InProcessAgentSessionClient.php.; `castor check` not run because task-start forbids it.
- Summary: Implemented and committed opt-in `disable-model-invocation: true` semantics. On-demand-only skills are absent from the model-visible catalog but remain available through `/skill:<name>` and explicit agent `skills` attachment. Default discovery, precedence, and malformed-metadata fallback behavior remain intact. Added documentation and deterministic discovery, command, attachment, parser, and in-process expansion tests. Worktree is clean at c1a33cfc79d02f58bdc8860c4ce2d362160e19e7.
- Ownership: owner=fork; fork_run=5u1s9536gzfn; revision=d3c879397ad9a802a80359b025b935568d64086e; scope=implement `disable-model-invocation` skill metadata across parsing, model catalog omission, explicit slash loading, agent-attached loading, documentation, and deterministic focused tests; outcome=completed; commit=c1a33cfc79d02f58bdc8860c4ce2d362160e19e7

## Task workflow update - 2026-08-29T17:20:11.401Z
- Validation: Pi reviewer at c1a33cfc79d02f58bdc8860c4ce2d362160e19e7: REQUEST CHANGES.; Reviewer confirmed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` were read and followed.; Reviewer reran focused tests — PASS, 44 tests, 153 assertions; `castor deptrac` — PASS; focused `castor phpstan` — PASS.
- Summary: Pi reviewer requested changes at c1a33cfc79d02f58bdc8860c4ce2d362160e19e7. Blocking correctness issue: mixed-case skill names register commands that the lowercase TUI parser cannot resolve. The reviewer also found argument splitting only handled ASCII spaces rather than general whitespace. Main owns the bounded correction and deterministic regression tests before re-review.
- Review: role=reviewer; artifact=inline-subagent-review-c1a33cfc7; target_revision=c1a33cfc79d02f58bdc8860c4ce2d362160e19e7; scope=specification fidelity, correctness, architecture, command routing, and lowest-layer TUI proof; decision=REQUEST CHANGES; blockers=mixed-case `/skill:` dispatch/expansion mismatch and ASCII-space-only argument splitting
- Ownership: owner=main; fork_run=none; revision=c1a33cfc79d02f58bdc8860c4ce2d362160e19e7; scope=normalize slash skill command names and lookup consistently, accept general whitespace before arguments, add deterministic mixed-case and whitespace regression tests, update docs only if behavior wording requires it; outcome=assigned; commit=none

## Task workflow update - 2026-08-29T17:23:21.470Z
- Validation: Post-fix focused `castor test --filter='SkillsContextBuilderTest|SkillCommandRegistrarTest|CommandParserTest::testSlashSkillColonName|PromptTemplateExpansionInProcessTest::testStartExpandsOnDemandOnlySkillCommand'` — PASS, 20 tests, 67 assertions.; Post-fix `castor deptrac` — PASS, 0 violations.; Post-fix full `castor phpstan` — PASS, 0 errors.; Post-fix `castor cs-check` — PASS, 0 changed files.; Post-fix `castor docs:validate` — PASS, 17 built-in documents.; IDE error diagnostics for SkillsContextBuilder.php and SkillCommandRegistrarTest.php — none.
- Summary: Resolved the reviewer blockers. `/skill:` command registration and expansion now use consistent case-insensitive matching, command DTO names are lowercase, and argument separation accepts general whitespace. Added mixed-case TUI dispatch and runtime expansion regressions, and clarified unknown-command behavior in the docs. Committed as 3135b9b360c05c65048d057ec4d24ec0da2715ae.
- Ownership: owner=main; fork_run=none; revision=c1a33cfc79d02f58bdc8860c4ce2d362160e19e7; scope=normalize slash skill command names and lookup consistently, accept general whitespace before arguments, add deterministic mixed-case and whitespace regression tests, update docs only if behavior wording requires it; outcome=completed; commit=3135b9b360c05c65048d057ec4d24ec0da2715ae

## Task workflow update - 2026-08-29T17:30:46.684Z
- Validation: Pi reviewer re-review at 3135b9b360c05c65048d057ec4d24ec0da2715ae: APPROVE WITH SUGGESTIONS.; Reviewer focused `castor test --filter='SkillDiscoveryTest|SkillsContextBuilderTest|SkillCommandRegistrarTest|CommandParserTest|PromptTemplateExpansionInProcessTest'` — PASS, 79 tests, 240 assertions.; Reviewer `castor deptrac` — PASS, 0 violations.; Reviewer focused `castor phpstan` — PASS, 0 errors.; Reviewer `castor cs-check` — PASS, 0 files.; Reviewer `castor docs:validate` — PASS.; Lowest correct TUI proof accepted: parser plus real slash catalog/registry plus in-process kernel expansion; tmux-specific proof is not needed for this local slash-command contract.
- Summary: Pi re-review approved revision 3135b9b360c05c65048d057ec4d24ec0da2715ae with non-blocking suggestions. Prior mixed-case routing and whitespace-argument blockers are resolved. No unresolved blockers remain before the deterministic CODE-REVIEW gate.
- Review: role=reviewer; artifact=var/reports/reviews/2026-08-27-add-on-demand-only-hatfield-skills-3135b9b36.md; target_revision=3135b9b360c05c65048d057ec4d24ec0da2715ae; scope=specification fidelity, prior blocker verification, correctness, architecture, DI, collisions, runtime ordering, docs, and lowest-layer TUI proof; decision=APPROVE WITH SUGGESTIONS; blockers=none; suggestions=optional case-collision agreement test, document single-word skill names or validate later, minor docs/usage consistency

## Task workflow update - 2026-08-29T17:32:12.990Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (69.8s).
- Pushed task/2026-08-27-add-on-demand-only-hatfield-skills to origin.
- branch 'task/2026-08-27-add-on-demand-only-hatfield-skills' set up to track 'origin/task/2026-08-27-add-on-demand-only-hatfield-skills'.
- Created PR: https://github.com/ineersa/agent-core/pull/443
- Summary: Implementation and reviewer iteration complete at 3135b9b360c05c65048d057ec4d24ec0da2715ae. Pi reviewer verdict: APPROVE WITH SUGGESTIONS, no blockers. Preparing deterministic full Castor gate, branch push, and PR creation.

## Task workflow update - 2026-08-29T17:32:20.682Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/443
- Updated PR Status: open
- Validation: CODE-REVIEW transition `castor check` — PASS in 69.8s.; Worktree clean at transition; branch pushed to `origin/task/2026-08-27-add-on-demand-only-hatfield-skills`.; PR created: https://github.com/ineersa/agent-core/pull/443
- Summary: CODE-REVIEW transition completed. Deterministic `castor check` passed in 69.8s, the branch was pushed, and PR #443 was created.

## Task workflow update - 2026-08-29T18:09:54.194Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #443 entered review iteration for the requested simplify pass. PR has no GitHub review or inline comments. Preparing exactly three independent read-only scout audits before any edits.

## Task workflow update - 2026-08-29T18:15:24.967Z
- Summary: Simplify audit completed with exactly three successful independent scouts. Two findings survived verification: lowercased slash projection emits redundant case-colliding command DTOs before the registrar discards them, and unknown `/skill:` attempts build a full SkillRegistry before confirming a matching skill exists. Main owns the bounded simplification and regression proof.
- Simplify scouts: 3/3 completed in one parallel subagent call at revision 3135b9b360c05c65048d057ec4d24ec0da2715ae; roles=reuse/architecture, simplicity/code quality, runtime/resource efficiency; all read-only and successful
- Ownership: owner=main; fork_run=none; revision=3135b9b360c05c65048d057ec4d24ec0da2715ae; scope=deduplicate case-folded slash command projection with first-discovered semantics, defer SkillRegistry construction until after a matching skill is found, and add focused collision regression proof; outcome=assigned; commit=none

## Task workflow update - 2026-08-29T18:17:01.987Z
- Validation: `castor test --filter=SkillsContextBuilderTest` — PASS, 14 tests, 45 assertions.; `castor deptrac` — PASS, 0 violations.; `castor phpstan --path=src/CodingAgent/Skills/SkillsContextBuilder.php` — PASS, 0 errors.; `castor cs-check` — PASS, 0 files.; IDE error diagnostics for production and test files — none.; Final simplify diff contains only `SkillsContextBuilder.php` and its focused test.
- Summary: Applied the verified simplify findings and committed revision 1566f6a66e5d4d7d1c06e115b14867b844e46544. Case-folded slash command projection now emits one first-discovered command instead of redundant DTOs later discarded by the registrar. `/skill:` expansion now avoids constructing SkillRegistry and collision data until a skill match exists. Added deterministic first-discovered case-collision regression proof.
- Ownership: owner=main; fork_run=none; revision=3135b9b360c05c65048d057ec4d24ec0da2715ae; scope=deduplicate case-folded slash command projection with first-discovered semantics, defer SkillRegistry construction until after a matching skill is found, and add focused collision regression proof; outcome=completed; commit=1566f6a66e5d4d7d1c06e115b14867b844e46544

## Task workflow update - 2026-08-29T18:23:14.114Z
- Summary: User clarified the authoritative dependency direction: TUI may depend on CodingAgent; CodingAgent must not depend on TUI. The skill catalog interface and DTO are unnecessary boundary workarounds. Main owns their removal and direct SkillDiscovery injection into the TUI registrar.
- Ownership: owner=main; fork_run=none; revision=1566f6a66e5d4d7d1c06e115b14867b844e46544; scope=remove SkillCatalogInterface and SkillCommand, inject SkillDiscovery directly into SkillCommandRegistrar, relocate normalized command projection there, update DI/deptrac/tests, preserve all skill behavior; outcome=assigned; commit=none

## Task workflow update - 2026-08-29T18:26:14.062Z
- Validation: Focused skill registrar/builder/in-process tests — PASS, 20 tests, 62 assertions.; `castor deptrac` — PASS, 0 violations.; Full `castor phpstan` — PASS, 0 errors.; `castor cs-check` — PASS, 0 files.; Semantic search found no remaining `SkillCatalogInterface` references.; Worktree clean after commit 1bb79203333551e27278c8f28788e2a7b4eaa660.
- Summary: Removed the unnecessary skill catalog interface and DTO under the clarified dependency direction. SkillCommandRegistrar now depends directly on CodingAgent SkillDiscovery; normalized command projection lives in the TUI registrar. Removed the DI alias and reversed the relevant Deptrac allowance to TuiListener → AppSkills. Committed as 1bb79203333551e27278c8f28788e2a7b4eaa660.
- Ownership: owner=main; fork_run=none; revision=1566f6a66e5d4d7d1c06e115b14867b844e46544; scope=remove SkillCatalogInterface and SkillCommand, inject SkillDiscovery directly into SkillCommandRegistrar, relocate normalized command projection there, update DI/deptrac/tests, preserve all skill behavior; outcome=completed; commit=1bb79203333551e27278c8f28788e2a7b4eaa660
- Follow-up task created: TODO/2026-08-29-remove-useless-boundary-interfaces-and-dtos.md

## Task workflow update - 2026-08-29T18:31:59.827Z
- Validation: Reviewer at 1bb79203333551e27278c8f28788e2a7b4eaa660: APPROVE WITH SUGGESTIONS, no code blockers.; Reviewer confirmed testing skill and tests/AGENTS.md were read and followed.; Reviewer verified zero remaining interface/DTO aliases, references, or orphaned Deptrac allowances.; Required full `castor check` remains pending for the CODE-REVIEW transition.
- Summary: Independent reviewer approved revision 1bb79203333551e27278c8f28788e2a7b4eaa660 with suggestions and no code blockers. Removal of the unnecessary interface/DTO, direct TUI → CodingAgent dependency, DI cleanup, Deptrac direction, behavior preservation, and deterministic tests were verified. Ready for the CODE-REVIEW full gate and PR update.
- Review: role=reviewer; artifact=var/reports/reviews/2026-08-27-direct-skill-discovery-1bb792033.md; target_revision=1bb79203333551e27278c8f28788e2a7b4eaa660; scope=specification fidelity, unnecessary boundary-type removal, direct TUI-to-CodingAgent dependency, DI/deptrac, behavior preservation, tests, and prior registry deferral; decision=APPROVE WITH SUGGESTIONS; blockers=none

## Task workflow update - 2026-08-29T18:33:29.818Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (77.5s).
- Pushed task/2026-08-27-add-on-demand-only-hatfield-skills to origin.
- branch 'task/2026-08-27-add-on-demand-only-hatfield-skills' set up to track 'origin/task/2026-08-27-add-on-demand-only-hatfield-skills'.
- PR already exists: https://github.com/ineersa/agent-core/pull/443
- Summary: Simplification and architecture correction complete at 1bb79203333551e27278c8f28788e2a7b4eaa660. Reviewer approved with suggestions and no blockers. Running the deterministic full gate, pushing the two follow-up commits, and updating PR #443.

## Task workflow update - 2026-08-29T18:33:37.640Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/443
- Updated PR Status: open
- Validation: CODE-REVIEW transition `castor check` — PASS in 77.5s.; Branch pushed through 1bb79203333551e27278c8f28788e2a7b4eaa660 and PR #443 updated.; Scouts: 3/3 completed.; Independent reviewer: APPROVE WITH SUGGESTIONS, no blockers.
- Summary: Architecture correction pushed to PR #443. Full deterministic `castor check` passed in 77.5s at 1bb79203333551e27278c8f28788e2a7b4eaa660. Follow-up audit task created as TODO/2026-08-29-remove-useless-boundary-interfaces-and-dtos.md.

## Task workflow update - 2026-08-29T18:44:19.819Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User requested removal of the overgrown runtime `expandSkillCommand()` path. Re-entering implementation to expand the already-parsed slash command in its TUI handler using existing `SkillsContextBuilder::buildFor()`, eliminating duplicate parsing and runtime-client special cases.

## Task workflow update - 2026-08-29T18:45:22.597Z
- Summary: Read all five current inline comments on PR #443. Required corrections: simplify `expandSkillCommand`, use a lowercase name index instead of repeated case-insensitive scans, remove narration comments, and restore the prior boolean cast for `disable-model-invocation`. No files have been edited for this review iteration yet.
- PR feedback: reviewer=ineersa; target_revision=1bb79203333551e27278c8f28788e2a7b4eaa660; comments=3887346504,3887348498,3887351160,3887354848,3887356093; classification=blocking implementation feedback
- Ownership: owner=main; fork_run=none; revision=1bb79203333551e27278c8f28788e2a7b4eaa660; scope=address all five PR comments by simplifying skill command parsing/lookup, indexing normalized names once, removing narration comments, and restoring boolean-cast metadata semantics; outcome=assigned; commit=none

## Task workflow update - 2026-08-29T20:19:56.653Z
- Validation: Focused Castor tests — PASS, 52 tests, 163 assertions.; `castor phpstan` — PASS, 0 errors.; `castor deptrac` — PASS, 0 violations.; `castor cs-fix` then `castor cs-check` — PASS, 0 files.; `castor docs:validate` — PASS, 17 documents.; IDE diagnostics — 0 errors in changed production/test files.
- Summary: Addressed all PR comments at 46989eb4403a181633463ccd5fbd6068f1f9c68b. Deleted `expandSkillCommand()` and removed skill checks from every runtime start/send path. The registered `/skill:<name>` TUI handler now performs the expansion only when that command is invoked. Skill commands accept no arguments. Discovery maintains a lowercase command-name index for direct lookup. Restored boolean-cast frontmatter semantics and removed the narration comment.
- Ownership: owner=main; fork_run=none; revision=1bb79203333551e27278c8f28788e2a7b4eaa660; scope=address all five PR comments; outcome=completed; commit=46989eb4403a181633463ccd5fbd6068f1f9c68b

## Task workflow update - 2026-08-29T20:25:55.450Z
- Validation: Reviewer: APPROVE WITH SUGGESTIONS, no blockers.; Reviewer confirmed testing skill and tests/AGENTS.md were read and followed.; Reviewer verified no remaining generic skill preprocessing or `expandSkillCommand` references.; Full `castor check` pending CODE-REVIEW transition.
- Summary: Independent reviewer approved 46989eb4403a181633463ccd5fbd6068f1f9c68b with suggestions and no blockers. Confirmed all five PR comments are addressed and expansion now occurs only in the TUI skill command handler.
- Review: role=reviewer; artifact=var/reports/reviews/2026-08-27-skill-command-handler-46989eb44.md; target_revision=46989eb4403a181633463ccd5fbd6068f1f9c68b; scope=all five PR comments, command-only expansion, direct indexed lookup, no-args contract, DI/deptrac, deterministic proof; decision=APPROVE WITH SUGGESTIONS; blockers=none

## Task workflow update - 2026-08-29T20:27:23.166Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (78.6s).
- Pushed task/2026-08-27-add-on-demand-only-hatfield-skills to origin.
- branch 'task/2026-08-27-add-on-demand-only-hatfield-skills' set up to track 'origin/task/2026-08-27-add-on-demand-only-hatfield-skills'.
- PR already exists: https://github.com/ineersa/agent-core/pull/443
- Summary: PR feedback iteration complete at 46989eb4403a181633463ccd5fbd6068f1f9c68b. Skill expansion is now exclusively command-triggered in TUI; generic runtime preprocessing and the oversized method were deleted. Reviewer approved with no blockers.

## Task workflow update - 2026-08-29T20:27:27.944Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/443
- Updated PR Status: open
- Validation: CODE-REVIEW transition `castor check` — PASS in 78.6s.; Branch and PR #443 updated through 46989eb4403a181633463ccd5fbd6068f1f9c68b.; Independent reviewer: APPROVE WITH SUGGESTIONS, no blockers.
- Summary: Revision 46989eb4403a181633463ccd5fbd6068f1f9c68b pushed to PR #443. `expandSkillCommand()` is deleted; `/skill:` expansion occurs only when the TUI command handler is invoked.

## Task workflow update - 2026-08-29T23:40:23.616Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-27-add-on-demand-only-hatfield-skills: ide_close_project returned isError.
- Merged task/2026-08-27-add-on-demand-only-hatfield-skills into integration checkout.
- Auto-merging AGENTS.md
Merge made by the 'ort' strategy.
 AGENTS.md                                          |   1 +
 depfile.yaml                                       |   1 +
 docs/agents.md                                     |   2 +-
 docs/settings-agents.md                            |   2 +
 docs/settings.md                                   |   1 +
 docs/skills.md                                     | 113 ++++++++++++
 .../InProcess/InProcessAgentSessionClient.php      |   6 +-
 src/CodingAgent/Skills/SkillDiscovery.php          |  12 ++
 src/Tui/Listener/SkillCommandRegistrar.php         |  64 +++++++
 tests/CodingAgent/Skills/SkillDiscoveryTest.php    |  68 +++++++
 .../Skills/SkillsContextBuilderTest.php            |  17 ++
 tests/Tui/Command/CommandParserTest.php            |  10 ++
 tests/Tui/Listener/SkillCommandRegistrarTest.php   | 197 +++++++++++++++++++++
 13 files changed, 488 insertions(+), 6 deletions(-)
 create mode 100644 docs/skills.md
 create mode 100644 src/Tui/Listener/SkillCommandRegistrar.php
 create mode 100644 tests/Tui/Listener/SkillCommandRegistrarTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-27-add-on-demand-only-hatfield-skills.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-27-add-on-demand-only-hatfield-skills.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #443 was merged on GitHub at 2026-08-29T23:40:00Z as merge commit 6142373c32e4a3a6c2222580f8622fb7ae238b04. Moving task to DONE and cleaning the task worktree.

## Task workflow update - 2026-08-29T23:41:48.271Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/443
- Updated PR Status: merged
- Validation: PR #443 confirmed MERGED at 2026-08-29T23:40:00Z.; Pre-merge CODE-REVIEW `castor check` passed in 78.6s at final task revision 46989eb4403a181633463ccd5fbd6068f1f9c68b.; Post-merge `LLM_MODE=true castor check`: 8/9 lanes passed; `test` lane failed on unrelated `ActiveRunContextTest::testCacheMissReplaysOnceAndPersistsTheResult` SQLite `database is locked`; leak check passed and no retry-until-green was performed.; Post-merge controller-replay, TUI, llm-real, Deptrac, PHPStan, CS, docs, and catalog checks passed.; Integration checkout clean; task worktree absent.
- Summary: Task moved to DONE after GitHub merge of PR #443 (merge commit 6142373c32e4a3a6c2222580f8622fb7ae238b04). Task worktree removed and integration checkout is clean.

## Task workflow update - 2026-08-29T23:44:30.494Z
- Moved DONE → IN-PROGRESS.
- Summary: Post-merge required `LLM_MODE=true castor check` failed in the unit lane with an SQLite `database is locked` error. Completion was premature. Reopening to investigate the deterministic parallel-isolation/lifecycle failure and require a passing full gate before DONE.

## Task workflow update - 2026-08-29T23:50:20.715Z
- Moved IN-PROGRESS → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-27-add-on-demand-only-hatfield-skills: ide_close_project returned isError.
- Merged task/2026-08-27-add-on-demand-only-hatfield-skills into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-27-add-on-demand-only-hatfield-skills.
- Pulled integration checkout: Already up to date..
- Summary: Investigated the post-merge SQLite lock rather than retrying blindly. The failing class passed against a fresh isolated DB (3 tests, 9 assertions), the complete ParaTest unit lane passed with fresh per-worker QA databases at 16-worker concurrency (4,825 tests, 19,512 assertions), and the required post-merge `LLM_MODE=true castor check` passed all nine lanes in 183.3s. Leak and cache guards were clean.

## Task workflow update - 2026-08-29T23:50:30.838Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/443
- Updated PR Status: merged
- Validation: Focused `ActiveRunContextTest` with fresh isolated databases — PASS, 3 tests, 9 assertions.; Full ParaTest unit lane with fresh per-worker QA databases at 16-worker concurrency — PASS, 4,825 tests, 19,512 assertions.; Post-merge `LLM_MODE=true castor check` — PASS all 9 lanes in 183.3s.; QA artifact integrity, process leak check, cache cleanup, and llama-proxy cache guard — PASS.; Integration checkout clean; task worktree removed.
- Summary: Task is DONE after deterministic investigation and a passing post-merge full gate. Integration checkout is clean and task worktree removed.

## Task workflow update - 2026-09-06T15:40:34+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
