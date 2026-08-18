# Audit and tighten repository AGENTS.md

## Goal
Review the repository AGENTS.md for duplication, ambiguity, stale guidance, and unnecessarily broad prose. Make it shorter and more precise without weakening mandatory safety, testing, architecture, task-workflow, or runtime/TUI requirements.

Keep detailed procedures in their canonical skills/docs where possible and leave concise triggers/pointers in AGENTS.md. Include the JetBrains IDE guidance consistently, without duplicating the dedicated behavior fix task.

## Acceptance criteria
- Identify duplicated, contradictory, stale, or imprecise instructions before editing.
- Reduce repetition and clarify requirements while preserving all enforceable project constraints.
- Keep canonical detail in existing skills/docs and retain clear instructions for when agents must load them.
- Ensure fork/subagent, JetBrains IDE, Castor/testing, task workflow, safety, and architecture rules remain unambiguous.
- Validate links, paths, command names, and terminology against the current repository.
- Provide a concise before/after summary of material instruction changes.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-06-audit-and-tighten-repository-agents-md
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-06-audit-and-tighten-repository-agents-md
Fork run: bcp4pjxr4e2j
PR URL: https://github.com/ineersa/agent-core/pull/375
PR Status: merged
Started: 2026-08-12T22:27:02.588Z
Completed: 2026-08-13T01:49:09.143Z

## Work log
- Created: 2026-08-06T22:03:30.431Z

## Task workflow update - 2026-08-12T22:27:02.588Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-06-audit-and-tighten-repository-agents-md.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-06-audit-and-tighten-repository-agents-md.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-06-audit-and-tighten-repository-agents-md.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-06-audit-and-tighten-repository-agents-md.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-06-audit-and-tighten-repository-agents-md.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-06-audit-and-tighten-repository-agents-md.
- Summary: Claimed task for orchestrated AGENTS.md audit and tightening.

## Task workflow update - 2026-08-12T22:34:54.853Z
- Recorded fork run: 5m8ar106xpgl
- Read root AGENTS.md, testing skill, tests/AGENTS.md, task-workflow skill, task context, and related JetBrains task before delegation.
- Dispatched three scouts to audit duplication/contradictions, validate current paths/symbols/commands, and map canonical instruction ownership.
- Launched implementation fork in task worktree with root-AGENTS-only scope and exact preservation/correction requirements.

## Task workflow update - 2026-08-12T22:37:31.551Z
- Recorded fork run: mwqsnb1qbmyi
- Verified fork commit b6ed6c4e8 exists, worktree is clean, and only AGENTS.md changed (282→151 lines). Requested a narrow amended commit because 'never edit production files' weakened the orchestrator rule; it must forbid direct main-agent edits to all files.

## Task workflow update - 2026-08-12T22:38:21.906Z
- Recorded fork run: mwqsnb1qbmyi
- Validation: Verified commit 88eaa6fcdbedaa91a776c1ee748148df87766ff5 exists on task branch.; git show --check --stat HEAD: clean; AGENTS.md only, 76 insertions/207 deletions.; git status --short: clean.; Verified amended orchestrator wording forbids all direct main-agent file edits.; Fork verified retained paths and Castor task names against repository sources; no tests/castor check run because change is docs-only and task-start stops before gate.
- Summary: Implementation complete in commit 88eaa6fcdbedaa91a776c1ee748148df87766ff5. Tightened root AGENTS.md from 282 to 151 lines, removing duplicated operational runbooks while preserving mandatory Castor/testing, safety, runtime/TUI proof, architecture, Extension API, session, JetBrains, and fork-only workflow rules. Corrected stale report path, kernel description, architecture wording, task statuses, theme types, layout, and environment-specific PID guidance. Only AGENTS.md changed; worktree is clean.

## Task workflow update - 2026-08-13T00:23:16.334Z
- Validation: Reviewer: APPROVED; read testing skill and tests/AGENTS.md; validated paths, Castor commands, symbols, boundaries, and preserved constraints.; castor deptrac: PASS (0 violations, 0 errors).; castor phpstan: PASS (0 errors).; castor cs-check: PASS (0 files needing fixes).; castor test first run: FAIL, unrelated SubagentLivePickerControllerTest::testArrowNavigationMovesSingleNativeHighlight selection assertion.; Focused retry via castor test --filter=SubagentLivePickerControllerTest::testArrowNavigationMovesSingleNativeHighlight: PASS (1 test, 45 assertions).; Full castor test retry: PASS (4407 tests, 16549 assertions).
- Summary: Reviewer APPROVED amended commit 6ae78f688cf5b3cc10cfd814b2d3bff68695649a after two accuracy corrections: AgentCore framework prohibition is absolute, and src/Platform is explicitly not separately Deptrac-layered. Specification fidelity gate passed; no unmapped surface or weakened constraints.

## Task workflow update - 2026-08-13T00:28:13.782Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (126.5s).
- Pushed task/2026-08-06-audit-and-tighten-repository-agents-md to origin.
- branch 'task/2026-08-06-audit-and-tighten-repository-agents-md' set up to track 'origin/task/2026-08-06-audit-and-tighten-repository-agents-md'.
- Created PR: https://github.com/ineersa/agent-core/pull/375

## Task workflow update - 2026-08-13T00:28:21.435Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/375
- Updated PR Status: open
- Validation: First CODE-REVIEW transition attempt timed out after 60s waiting for active sibling-worktree castor check lock (holder PID 453083); no process was signaled or killed.; Retried after holder exited: deterministic castor check PASS (126.5s).; PR created: https://github.com/ineersa/agent-core/pull/375
- Summary: Moved to CODE-REVIEW after reviewer approval and focused validation. Deterministic gate passed; branch pushed and PR #375 created.

## Task workflow update - 2026-08-13T00:34:10.479Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User expanded scope during review: tighten all project-owned AGENTS.md files, not only root. Exclude dependency-owned files under vendor directories.

## Task workflow update - 2026-08-13T00:34:41.688Z
- Recorded fork run: wp2c6ln0a0bx
- Expanded implementation scope to all nine project-owned AGENTS.md files. Dependency-owned AGENTS.md files under vendor directories remain excluded.
- Launched fork to audit canonical ownership, tighten nested/test instructions, validate retained paths/symbols, and amend the existing single commit.

## Task workflow update - 2026-08-13T00:39:32.570Z
- Recorded fork run: wp2c6ln0a0bx
- Validation: Fork read testing skill and tests/AGENTS.md before documentation/test-guidance changes.; git diff --check: PASS.; Retained paths, symbols, Messenger routing, Doctrine locations, executable locator chain, protocol types, and test helpers validated against current sources.; Worktree reported clean; no Castor suites run by implementation fork because changes are documentation-only.
- Summary: Expanded implementation complete in amended commit bec8c9dfbe8c60e82d961ac2d8e15acd93c315bd. Audited/tightened all nine project-owned AGENTS.md files; excluded dependency-owned vendor copies. Total scoped instructions reduced from 799 to 487 lines, with stale message/event/Doctrine/protocol claims and broken nested paths corrected.

## Task workflow update - 2026-08-13T00:47:05.210Z
- Recorded fork run: syiyd3rtgso8
- Summary: Expanded-scope reviewer requested changes because the amended commit message still described root-only scope. Also requested broader, accurate event-ordering wording. Direct-PHPUnit environment guidance will not be restored because root mandates Castor-only QA.

## Task workflow update - 2026-08-13T00:58:02.117Z
- Recorded fork run: syiyd3rtgso8
- Validation: Final reviewer: APPROVED; specification fidelity passed; vendor-owned AGENTS.md excluded.; castor test: PASS (4407 tests, 16549 assertions).; castor deptrac: PASS (0 violations, 0 errors).; castor phpstan: PASS (0 errors).; castor cs-check: PASS (0 files needing fixes).
- Summary: Final expanded-scope commit 5a081a11229a4ce076944022d6fc9bef22e520ff approved. Commit message now accurately covers all project-owned AGENTS.md files; event ordering wording corrected. Nine project-owned instruction files reduced by 447 net lines while preserving local invariants and global constraints.

## Task workflow update - 2026-08-13T01:11:49.967Z
- Recorded fork run: bcp4pjxr4e2j
- User approved merging current origin/main into the amended task branch, then updating PR history with force-with-lease.
- Launched fork to perform a non-destructive merge of origin/main, preserve expanded AGENTS.md changes, resolve any conflicts, and leave a clean unpushed merge commit.

## Task workflow update - 2026-08-13T01:29:12.211Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (118.9s).
- Pushed task/2026-08-06-audit-and-tighten-repository-agents-md to origin.
- branch 'task/2026-08-06-audit-and-tighten-repository-agents-md' set up to track 'origin/task/2026-08-06-audit-and-tighten-repository-agents-md'.
- Skipped PR creation (pushOnly: true).
- Validation: Post-merge reviewer: APPROVED; task diff vs origin/main remains exactly nine project-owned AGENTS.md files.; castor test: PASS (4410 tests, 16559 assertions).; castor deptrac: PASS.; castor phpstan: PASS.; castor cs-check: PASS.; castor check: PASS; all lanes, artifact integrity, leak check, and llama-proxy cache guard passed.; Force-with-lease verified expected remote tip 6ae78f688cf5b3cc10cfd814b2d3bff68695649a before updating to 72a1ced3cd01b86b060a9d0da5b6464e96ad7657.; PR #375 open at updated HEAD 72a1ced3cd01b86b060a9d0da5b6464e96ad7657.
- Summary: Merged current origin/main at 2d2c4b861 into the task branch without conflicts, re-reviewed and revalidated, then safely updated existing PR #375 with force-with-lease after verifying the remote tip. PR title/body updated for expanded all-project-AGENTS scope.

## Task workflow update - 2026-08-13T01:29:18.008Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/375
- Updated PR Status: open
- Validation: CODE-REVIEW transition deterministic castor check also PASS (118.9s).
- Summary: PR #375 updated to expanded scope and current main. Task is ready for review/merge.

## Task workflow update - 2026-08-13T01:49:09.144Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-08-06-audit-and-tighten-repository-agents-md into integration checkout.
- Merge made by the 'ort' strategy.
 AGENTS.md                                       | 283 +++++++-----------------
 src/AgentCore/Application/AGENTS.md             | 150 ++++++-------
 src/AgentCore/Domain/AGENTS.md                  |   8 +-
 src/AgentCore/Domain/Event/AGENTS.md            |  38 ++--
 src/AgentCore/Domain/Message/AGENTS.md          |  31 ++-
 src/AgentCore/Infrastructure/Doctrine/AGENTS.md |  11 +-
 src/CodingAgent/Runtime/Process/AGENTS.md       |  37 ++--
 src/CodingAgent/Runtime/Protocol/AGENTS.md      | 259 +++-------------------
 tests/AGENTS.md                                 | 224 ++++++-------------
 9 files changed, 297 insertions(+), 744 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-06-audit-and-tighten-repository-agents-md.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-06-audit-and-tighten-repository-agents-md.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #375 state: MERGED.; Post-merge castor check: NOT RUN per explicit user constraint.; Pre-merge evidence remains recorded: focused Castor commands passed and deterministic castor check passed twice during CODE-REVIEW preparation.
- Summary: PR #375 confirmed merged on GitHub at 2026-08-13T01:37:54Z (merge commit b7832aa774a8793ebc117955c0af1f298b96a43b). User explicitly stated post-merge castor check cannot be run.

## Task workflow update - 2026-08-14T19:53:33+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
