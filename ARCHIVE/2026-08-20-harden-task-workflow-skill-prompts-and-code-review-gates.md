# Harden task-workflow skill, prompts, and CODE-REVIEW gates

## Goal
Follow-up from the session-picker-delete run. Implement the agreed workflow improvements so orchestrators hit fewer human interrupts and conflicting instructions.

Scope is task-workflow skill/prompts/tooling (and related worktree bootstrap if needed). No product feature work.

## Agreed updates

### A. Align prompt templates with the skill (high)
In `task-to-pr` / review prompts, replace blanket TmuxHarness rejection with the TUI proof pyramid already in the skill + AGENTS.md:
- virtual for local picker/render/input
- controller-replay for runtime JSONL
- tmux only when pty/process boot is the contract

Same wording across skill, AGENTS.md, and slash/prompt templates.

### B. Make `move_task` failures actionable (high)
On CODE-REVIEW failure, return at least:
- failing lane name (`test`, `phpstan`, …)
- first failure snippet
- path to `var/reports/qa-<id>/check-*.log`

Do not return only “An error occurred while executing tool `move_task`”.

### C. Policy for unrelated gate failures (high)
Document one clear policy for when `move_task → CODE-REVIEW` fails on a flake clearly outside the task diff:
1. Short-term preferred: allowlisted flake triage — orchestrator may fix the minimal unrelated test assert on the branch, or
2. Escalation — stop and ask user before landing unrelated fixes on a feature PR, or
3. Later: gate split — CODE-REVIEW move runs task-relevant check first; full check advisory/post-merge.

Implement/document the chosen short-term policy in skill + prompts. Prefer (1) + better error reporting unless task-explain decides otherwise with user.

### D. Soften pre-move validation (medium)
Change `task-to-pr` local validation to:
- focused filters for touched areas
- `deptrac` / `phpstan` / `cs-check`
- `test:tui` only if TUI/tmux layer required
- do **not** mandate full `castor test` when `move_task` will run full `castor check` anyway

### E. Reviewer verdict rubric (medium)
Add to skill / reviewer instructions:
- CRITICAL/BUG/SEC/missing required proof → REQUEST CHANGES
- pure ponytail micro-shrink / NTH / naming → APPROVE WITH SUGGESTIONS unless correctness is affected
- “all previous findings fixed; only −3 line shrink left” → do **not** REQUEST CHANGES

### F. Worktree vendor symlink check (medium)
On worktree create (or first fork bootstrap), verify path packages like `ineersa/hatfield-extension-api` are symlinks/present. Fail loudly with “vendor package link broken” instead of later DI class-not-found.

### G. Skill cleanup + small runbooks (low/medium)
- Fix the orphaned `#` / leaked-workers placement in `task-workflow/SKILL.md`
- Add a short CODE-REVIEW failure runbook: where logs live, how to classify unrelated vs task regressions
- In `task-review-iterate`: prefer `gh api repos/.../pulls/<n>/comments` for inline review threads
- One-liner on status styling: style by key in `StatusPanelWidget`, keep `setStatus` text plain

### H. Prompt/skill split (low)
Keep `WorkflowPrompt` as a short system primer. Put phase procedures only in the skill. Slash prompts should say “load task-workflow skill, then follow phase X” instead of pasting a conflicting long checklist.

## Out of scope
- Product/TUI feature changes
- Reworking Castor QA lanes themselves beyond what is needed for actionable move_task errors / worktree vendor checks
- Changing the external task board layout

## Acceptance criteria
- task-to-pr / review prompt templates no longer require blanket TmuxHarness proof; they match the skill TUI pyramid (virtual → controller-replay → minimal tmux)
- move_task CODE-REVIEW failures return failing Castor lane, a short failure snippet, and the qa report/log path
- Skill + prompts document a clear unrelated-gate-failure policy (preferred short-term: minimal allowlisted flake fix on branch, or escalate to user)
- task-to-pr local validation no longer mandates full castor test before move_task; focused filters + deptrac/phpstan/cs-check, with test:tui only when needed
- Reviewer verdict rubric added: REQUEST CHANGES for critical/bug/sec/missing proof; APPROVE WITH SUGGESTIONS for NTH/ponytail micro-shrink
- Worktree create/bootstrap verifies path packages like ineersa/hatfield-extension-api and fails loudly if the vendor link is broken
- task-workflow SKILL.md orphaned heading/leaked-workers fragment fixed; CODE-REVIEW failure runbook + inline PR comment discovery + StatusPanel error-styling one-liner added
- WorkflowPrompt stays short; slash/prompt templates defer phase procedures to the skill instead of duplicating conflicting checklists

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates
Fork run: 6n969a3k8hos
PR URL: https://github.com/ineersa/agent-core/pull/435
PR Status: merged
Started: 2026-08-27T17:07:42.032Z
Completed: 2026-08-27T21:36:06.380Z

## Work log
- Created: 2026-08-20T16:08:43+00:00

## Task workflow update - 2026-08-27T17:01:56.309Z
- Summary: Task-explain decisions finalized: update both Pi/TypeScript and Hatfield/PHP workflow implementations, while preserving intentional platform differences. Replace any allowlist/known-flake policy with ruthless root-cause elimination of every flaky test. Keep common proof, validation, verdict, and error-reporting policy aligned without requiring prompt/skill files to be identical. Hatfield reviews should resume the prior reviewer with the delta when possible; Pi continues launching reviewers. Worktree bootstrap should proactively repair Composer path-package links during TODO→IN-PROGRESS, then verify them and fail clearly only if repair did not produce a usable package. CODE-REVIEW setup failures that have no lane log must report the actual setup failure and available QA directory without inventing a lane/log.
- Task-explain clarification: both native PHP/Hatfield and Pi TypeScript implementations remain supported and must be updated for parity in shared policy.
- Flake policy: no allowlists, quarantine, blind retries, timeout increases, or known-flake exceptions. Any flaky test exposed by the gate must receive a deterministic root-cause fix, be documented, re-reviewed, and rerun. Escalate only when the proper fix requires a broader product/design decision outside task authority.
- Worktree vendor handling: TODO→IN-PROGRESS should self-heal dependencies through Composer rather than merely detect a copied broken symlink. The existing `composer install -d .hatfield/extensions` repairs only the extension subproject vendor; implementation should also reconcile the root worktree Composer dependencies/path package as needed, then verify `vendor/ineersa/hatfield-extension-api` is usable. If Composer cannot repair it, fail loudly with `vendor package link broken` and clean up the partial worktree.
- CODE-REVIEW failure reporting: named lane failures return lane, bounded first useful snippet, and `var/reports/qa-<id>/check-*.log`. Failures before/after lanes (lock/setup/preflight/finalizer) return the real bounded error and QA directory when available, without fabricating a lane or log.
- Prompt/skill split: `.pi` and `.hatfield` files are intentionally not identical; do not add whole-file equality tests. Keep shared policies aligned while retaining runtime-specific procedures.
- Reviewer continuity: Pi launches a reviewer according to its available workflow. Hatfield records the reviewer identity and uses `agent_resume` with the new commit/diff, previous findings, and resolution delta for subsequent review rounds; launch a new reviewer only when no resumable reviewer exists. Keep WorkflowPrompt short and place platform-specific phase procedures in each platform skill.

## Task workflow update - 2026-08-27T17:07:30.425Z
- Summary: Additional finalized workflow decision: delegation is a context-ownership decision, not mandatory permission to edit. Replace the blanket rule that main agents may never edit with a context-aware ownership model. The agent holding the detailed implementation model should normally finish cohesive work; delegate only when a worker can own a bounded slice and the handoff reduces total context/rereading. Preserve independent review even when main implements.
- Implementation ownership policy to add across AGENTS.md, both workflow skills, and platform-appropriate prompts: main may implement when it already owns substantial code context, the change is cohesive, requirements are evolving, abstractions are tightly connected, transfer cost approaches implementation cost, or main would need to reread the whole area afterward.
- Worker delegation policy: delegate complete bounded slices such as isolated modules, mechanical migrations, independently testable task items, investigation-plus-implementation, disjoint parallel work, or work whose internal detail would consume significant main context. Size alone does not decide ownership.
- Ownership guardrails: one owner per implementation slice; no concurrent main/worker edits to the same files; ownership changes require explicit handoff; do not split investigation from implementation when that forces rediscovery; resume the same worker/reviewer for follow-ups where supported.
- Handoff efficiency: workers return compact evidence (commit, changed paths, validation, unresolved risks), not long implementation narratives. Main reviews the diff and evidence rather than reconstructing the worker's full context.
- Revised task-start procedure: scout/research as needed, build the implementation model, explicitly choose main-owned implementation or worker-owned bounded slices, implement without overlap, validate and record evidence, then stop before task-to-pr. Independent reviewer remains required before CODE-REVIEW regardless of implementation owner.

## Task workflow update - 2026-08-27T17:07:42.032Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates.
- Summary: Task-explain complete. Starting implementation with finalized decisions covering both workflow implementations, ruthless flake elimination, actionable gate failures, Composer-based worktree dependency repair, platform-specific reviewer continuity, and context-aware implementation ownership.

## Task workflow update - 2026-08-27T17:09:02.932Z
- Recorded fork run: fjnpms3rqw11
- Implementation ownership assigned as one cohesive worker slice in worktree `/home/ineersa/projects/agent-core-worktrees/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates`; fork run `fjnpms3rqw11`. Current repository rules still require fork-owned edits until this task changes them.

## Task workflow update - 2026-08-27T17:14:16.059Z
- Summary: Additional finalized rule: do not preserve dead code or unrequested fallback paths. When a change makes code, branches, prompts, adapters, compatibility paths, or tests obsolete/unreachable, they must be deleted rather than retained “just in case.” Reviewers must treat dead-code preservation and uncited fallback behavior as change-request findings.
- New mandatory policy to add to root/workflow guidance and reviewer instructions: no dead-code preservation. If code is dead, unreachable, superseded, or no longer has a supported caller, delete it in the same change. Do not retain old branches, adapters, prompt procedures, compatibility shims, fallback behavior, or tests for removed behavior unless an explicit finalized requirement or published compatibility contract requires them.
- Review gate addition: reviewers must search for obsolete paths left behind by the diff and REQUEST CHANGES for dead code or fallback behavior that is not mapped to an explicit requirement. This complements the existing specification-fidelity and no-backward-compatibility rules.

## Task workflow update - 2026-08-27T17:18:03.577Z
- Recorded fork run: 5fxn4omvcqd6
- Summary: Initial implementation commit `89476089c30e1bc5df84f05ea942ca46ce13122a` passed focused validation but was not accepted yet. A corrective implementation run is addressing the late no-dead-code/no-fallback requirement plus discovered ownership/TUI wording contradictions, realistic Castor lane parsing, no-lane error coverage, and dangling vendor-link repair/cleanup coverage.
- Initial fork `fjnpms3rqw11` committed `89476089c30e1bc5df84f05ea942ca46ce13122a`; focused tests, deptrac, phpstan, cs-check, and docs:validate passed. Parent inspection found remaining acceptance gaps, so implementation remains IN-PROGRESS.
- Corrective fork `5fxn4omvcqd6` owns the delta: remove residual mandatory-fork wording, remove unconditional test:tui language, add dead-code/no-uncited-fallback review policy, parse real Castor summaries without including `(exit code 1)` in lane filenames, test no-lane setup errors, detect dangling source package symlinks in PHP/TS, and add deterministic Composer repair/failure cleanup tests.
- An initial validation command appears to have accidentally modified the integration checkout copy of native WorkflowPrompt. Separate cleanup fork `gv0qee8f3t5r` was assigned only to verify and restore that duplicate uncommitted task change; no integration commit is authorized.

## Task workflow update - 2026-08-27T17:18:24.659Z
- Cleanup verification `gv0qee8f3t5r` correctly stopped because integration contamination was not textually identical to task-branch HEAD. Parent comparison established it is the semantically identical pre-cs-fix WorkflowPrompt shortening produced by the task run, not an independent user change. Narrow cleanup run `k5n4gyxs6q7n` is authorized to restore only that file if it remains the sole integration modification.

## Task workflow update - 2026-08-27T17:18:41.040Z
- Cleanup run `k5n4gyxs6q7n` restored only the confirmed accidental integration `WorkflowPrompt.php` modification. Integration checkout `git status --short` is clean; no commit or other file change was made.

## Task workflow update - 2026-08-27T17:28:11.605Z
- Recorded fork run: v7n6n4bip3ad
- Summary: Corrective commit `4abd05644` fixed gate parsing, dangling vendor repair/tests, dead-code policy, ownership procedure, and TUI validation. Parent inspection found the slash templates still duplicate long phase procedures and several fork-only phrases remain, contrary to the finalized prompt/skill split and context-aware ownership policy. Final focused cleanup run is thinning both prompt sets and removing those contradictions.
- Corrective fork `5fxn4omvcqd6` committed `4abd05644`; focused tests passed (21 tests, 147 assertions), plus deptrac/phpstan/cs-check/docs validation. It also unintentionally invoked `castor test:tui` through shell backtick substitution; the lane passed, but that invocation was not required evidence.
- Parent acceptance inspection after `4abd05644` found residual `fork MUST`/`Fork instructions`/`fork fixes` wording in both skills and Pi prompts, and the slash templates still contain full duplicated phase checklists plus a repeated shared-policy paragraph despite the requirement that skills own phase procedures.
- Final cleanup fork `v7n6n4bip3ad` owns only prompt thinning, generic implementation-owner wording, and Hatfield reviewer-resume procedure alignment. No production PHP/TS behavior change is authorized.

## Task workflow update - 2026-08-27T17:32:11.844Z
- Recorded fork run: v7n6n4bip3ad
- Validation: castor test --filter='MoveTaskHandlerTest|WorktreeManagerCopyTest' — PASS (21 tests, 147 assertions, 2.3s); castor deptrac — PASS (violations=0, errors=0); castor phpstan — PASS (errors=0, file_errors=0); castor cs-check — PASS (files_fixed=0); castor docs:validate — PASS (16 built-in documents); castor test:tui — PASS (8 tests, 59 assertions; accidentally invoked during prompt-text search, not required evidence); git diff --check — PASS; Parent targeted search for mandatory-fork and unconditional test:tui phrases — no matches; Task worktree git status — clean; Integration checkout git status — clean after restoring accidental validation contamination
- Summary: Implementation complete in commits `89476089c`, `4abd05644`, and `ac45e3463` (HEAD). Both Pi/TypeScript and Hatfield/PHP workflow paths now have actionable Castor gate errors, Composer repair plus fail-loud path-package verification, deterministic native coverage, ruthless flake/dead-code/no-uncited-fallback policy, context-aware implementation ownership, correct TUI proof pyramid, reviewer verdict rubric, Hatfield reviewer resume, and thin platform slash prompts that defer phase procedures to skills. Parent inspection confirmed the task worktree and integration checkout are clean and prohibited mandatory-fork/unconditional-test:tui wording is absent.
- Final cleanup fork `v7n6n4bip3ad` committed `ac45e3463`, reducing all ten Pi/Hatfield slash prompts to thin phase dispatchers, generalizing remaining ownership language, and making Hatfield re-review resume the prior reviewer at the actual re-review step.
- Parent final inspection confirmed HEAD `ac45e3463`, expected 22-file cumulative diff, clean worktree, thin prompt content, Hatfield agent_resume procedure, and no targeted contradictory mandatory-fork/unconditional-test:tui phrases. Implementation is task-start complete and intentionally remains IN-PROGRESS pending user-run task-to-pr.

## Task workflow update - 2026-08-27T17:58:31.948Z
- Summary: User approved a final workflow architecture expansion before PR: one implementation worker means one fork; persist delegated run/artifact identity in task work logs; localize module-specific root instructions into nested AGENTS files with a root routing map; make authority explicit across root/nested AGENTS, skills, thin slash prompts, WorkflowPrompt, and tool semantics; make scouting proportional; and remove the misleading configurable Castor gate timeout in favor of the fixed Castor wall plus outer cleanup grace. Explicitly out of scope: transition graph changes, lifecycle side-effect hardening, and Pi/TypeScript test-framework work.
- Terminology clarification: a worker is a fork. One bounded worker-owned implementation slice is owned by one fork. Scouts, researchers, and reviewers are subagents, not workers. Main may own cohesive implementation it already understands; fork ownership remains disjoint and explicit.
- Delegation identity: after launching implementation forks or reviewers during tracked phases, record role, fork/subagent artifact or run identity, target revision, and scope in task metadata/work log. Keep existing fork/reviewer-specific handoff formats authoritative; do not impose a universal handoff schema. Persist only identity, revision, outcome/validation, and unresolved blockers into task metadata. Hatfield `agent_resume` remains same-parent-session only; launch a new reviewer when no resumable reviewer exists.
- Instruction locality: keep global safety, Castor triggers, architecture constraints, specification fidelity/dead-code policy, logging/privacy, and task routing in root AGENTS. Move detailed TUI/runtime/session/Extension API invariants to nearest nested AGENTS files and add a concise root area→instructions map. Skills remain opt-in procedures, not substitutes for module-local invariants.
- Instruction authority: root AGENTS owns global invariants/routing; nested AGENTS owns module-local invariants; platform skills own phase procedures; slash prompts only validate arguments and dispatch phases; WorkflowPrompt only provides discoverability; tool definitions describe executable parameters/preconditions/side effects/errors rather than orchestration checklists. Root should reference the active runtime task-workflow skill, not hardcode Pi.
- Scouting policy: zero scouts when main already owns enough context; one scout for unfamiliar bounded scope; parallel scouts only for genuinely independent lenses or high-risk cross-module work; do not re-scout research main already performed. If noisy investigation should retain context into implementation, assign one fork the complete investigation+implementation slice. Scouts remain read-only; independent review remains required.
- Timeout contract: remove the public `castorCheckTimeoutSeconds` option from both PHP/Hatfield and Pi/TypeScript move_task surfaces and delete obsolete resolver/dead compatibility paths. Castor owns the 210-second absolute wall; use one fixed outer timeout with bounded termination/kill and host-process grace consistently in both implementations. Update native tests and guidance. Do not add Pi/TypeScript behavioral tests.
- Explicitly deferred by user: task transition state-machine enforcement, additional GitHub/worktree lifecycle hardening, and a Pi/TypeScript test harness.

## Task workflow update - 2026-08-27T17:59:26.689Z
- Recorded fork run: rry1p0jj4xnq
- Delegation: implementation fork; fork_run=rry1p0jj4xnq; target_revision=ac45e3463; scope=workflow instruction locality/authority, proportional scouting, delegation identity persistence guidance, and fixed Castor gate timeout across both implementations. One fork is the sole worker/owner for this bounded cohesive slice. Expected outcome: committed delta plus focused Castor validation; unresolved blockers to be recorded from the fork handoff.

## Task workflow update - 2026-08-27T18:09:18.761Z
- Ownership handoff after fork `rry1p0jj4xnq`: main owns a narrow final consistency correction based on its completed acceptance inspection. Scope is limited to removing the remaining root Pi-specific skill pointer and making both PHP/TypeScript `forkRun` tool descriptions match the finalized implementation-fork-only contract. Files: `AGENTS.md`, `.pi/extensions/task-workflow/index.ts`, `.hatfield/extensions/task-workflow/src/TaskWorkflowExtension.php`. No overlap with an active fork; the fork has completed.

## Task workflow update - 2026-08-27T18:10:24.083Z
- Recorded fork run: rry1p0jj4xnq
- Validation: castor test --filter='MoveTaskHandlerTest|WorktreeManagerCopyTest' — PASS (21 tests, 147 assertions, 2.3s); castor deptrac — PASS (violations=0, errors=0); castor phpstan — PASS (errors=0, file_errors=0; rerun after final main correction); castor cs-check — PASS (files_fixed=0; rerun after final main correction); castor docs:validate — PASS (16 built-in documents; rerun after final main correction); git diff --check — PASS; Search for removed castorCheckTimeoutSeconds — no matches; Search for root hardcoded .pi task-workflow skill pointer — no matches; Search for stale Fork/subagent forkRun descriptions — no matches; Task worktree git status — clean; Integration checkout git status — clean
- Summary: Final workflow architecture implementation is complete at HEAD `a23bc3bb5`. Fork `rry1p0jj4xnq` implemented instruction locality/authority, proportional scouting, delegation identity persistence, one-worker/one-fork terminology, and fixed Castor gate timeouts in commit `958ab89b2`. Parent acceptance inspection then applied and committed a narrow main-owned consistency correction in `a23bc3bb5`, removing the last root Pi-specific skill pointer and aligning both tool schemas so `forkRun` is explicitly reserved for implementation forks. Worktree and integration checkout are clean.
- Fork `rry1p0jj4xnq` completed commit `958ab89b2` with no blockers and passed focused tests, deptrac, phpstan, cs-check, docs validation, and diff check.
- Main-owned final consistency correction committed as `a23bc3bb5`: root docs map now references the active runtime task-workflow skill, and PHP/TypeScript move_task/update_task schema descriptions reserve `forkRun` for implementation fork identities. Main reran docs:validate, cs-check, and phpstan successfully. Implementation remains IN-PROGRESS pending task-to-pr.

## Task workflow update - 2026-08-27T19:04:07.990Z
- Recorded fork run: 5xxrf6sbo5jm
- Implementation fork `5xxrf6sbo5jm` owns the user-requested terminology correction at revision `a23bc3bb5`: remove agent/delegation uses of “worker” and use “fork” directly across root workflow guidance, both runtime task-workflow skills, packaged agent/subagent guidance, and docs. Legitimate QA/ParaTest/Messenger/process worker terminology and `cleanup:workers` commands are explicitly out of scope.

## Task workflow update - 2026-08-27T19:04:36.400Z
- Latest terminology requirement: do not introduce “worker” as an alias for an implementation fork. Agent/delegation guidance must say “fork” directly everywhere. Preserve “worker” only where it names an actual runtime/process concept (Messenger, ParaTest, QA cleanup/stale-worker guards, command names).

## Task workflow update - 2026-08-27T19:06:55.346Z
- Recorded fork run: 5xxrf6sbo5jm
- Validation: castor docs:validate — PASS (16 built-in documents); castor cs-check — PASS (files_fixed=0); git diff --check / git show --check — PASS; Scoped search for worker/fork, worker-owned, delegated-worker, a worker is one fork, new fork is a new worker, and not workers — no matches; Task worktree git status — clean
- Summary: Applied the final terminology correction in commit `7fce0975d`: implementation-agent guidance now uses “fork” directly and no longer introduces “worker” as an alias. Updated root AGENTS, both runtime task-workflow skills, docs/agents, packaged subagents skill, and scout prompt. Legitimate QA/ParaTest/Messenger/process worker terminology remains unchanged. Worktree is clean and implementation remains IN-PROGRESS pending task-to-pr.
- Fork `5xxrf6sbo5jm` completed commit `7fce0975d61be4dc8c7171ad5595f92cdd673bfd` with no blockers. Parent verified branch HEAD, clean status, six-file diff, and zero stale implementation-role terminology matches.

## Task workflow update - 2026-08-27T19:08:25.611Z
- User explicitly directed transition to CODE-REVIEW without a reviewer. Reviewer dispatch/verdict is skipped by user instruction; existing implementation and validation evidence remains recorded.

## Task workflow update - 2026-08-27T19:09:54.093Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (79.2s).
- Pushed task/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates to origin.
- branch 'task/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates' set up to track 'origin/task/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates'.
- Created PR: https://github.com/ineersa/agent-core/pull/435
- Validation: Focused MoveTaskHandler/WorktreeManager tests PASS (21 tests, 147 assertions); castor deptrac PASS; castor phpstan PASS; castor cs-check PASS; castor docs:validate PASS; git diff --check PASS; Worktree clean before transition
- Summary: Implementation complete at `7fce0975d`. User explicitly waived reviewer dispatch and directed transition to CODE-REVIEW. Final scope includes hardened failure diagnostics, worktree dependency repair, thin runtime-specific prompts, context-aware fork ownership, instruction locality/authority, proportional scouting, fixed Castor timeout contract, and fork-only implementation terminology.

## Task workflow update - 2026-08-27T20:11:31.926Z
- External review feedback received: REQUEST CHANGES. Parent verification agrees Castor lane parsing is broken for actual multiline `quality failed:\n- <lane>: exit code N` output; agrees ownership routing happens too late, implementation-fork handoff is excessively verbose, same-worktree write concurrency is underspecified, and Hatfield skill documents Pi configuration incorrectly. Ownership metadata concern is valid, but a canonical append-only work-log record is preferred over adding a new YAML ownership schema without a product decision. PR history is polluted by redundant merge commits, although the current net diff against origin/main contains only the 33 task-scope files; cleanup still improves reviewability. Local move_task Castor gate passed in 79.2s even though GitHub exposes only GitGuardian. No fixes started pending user decision.

## Task workflow update - 2026-08-27T20:15:15.214Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Address accepted review feedback. Implement: real multiline Castor lane parsing/tests in both runtimes, shallow ownership routing before deep exploration, canonical append-only ownership work-log records without new schema, sequential same-worktree writers, Hatfield-specific task-board configuration docs, and merge origin/main. Defer fork handoff-format changes per user decision.

## Task workflow update - 2026-08-27T20:16:10.676Z
- Recorded fork run: 3p01b2t0f6t1
- ownership: owner=fork; run=3p01b2t0f6t1; revision=7fce0975d; outcome=assigned; scope=merge origin/main and implement accepted review fixes for Castor multiline lane parsing, shallow ownership routing, append-only ownership records, write serialization, and Hatfield configuration docs
- Finalized review decisions: use canonical append-only ownership records in existing workLog with no new schema; defer all fork handoff-profile changes; merge origin/main normally rather than rewriting branch history; implement all other accepted findings. Fork `3p01b2t0f6t1` owns the complete correction and focused validation.

## Task workflow update - 2026-08-27T20:21:53.616Z
- Recorded fork run: 3p01b2t0f6t1
- Validation: castor test --filter=MoveTaskHandlerTest — PASS (16 tests, 107 assertions); castor deptrac — PASS (violations=0, errors=0); castor phpstan — PASS (errors=0, file_errors=0); castor cs-check — PASS (files_fixed=0); castor docs:validate — PASS (16 built-in documents); git diff --check and git diff --check origin/main...HEAD — PASS; Parent verified HEAD `463bd6301`, clean worktree, real multiline fixture, colon-preserving lane parser, canonical ownership format, shallow routing, sequential writers, and Hatfield configuration
- Summary: Accepted review fixes are complete at `463bd6301`. Current `origin/main` was merged normally in `9564e6291` without conflicts. The correction fixes real multiline Castor lane parsing in PHP/TypeScript with a production-shaped native regression fixture; routes ownership after shallow exploration; standardizes append-only ownership work-log records; serializes same-worktree writers; and corrects Hatfield task-root configuration guidance. Fork handoff-profile changes and Castor JSON summaries remain deliberately deferred. Worktree is clean and remains IN-PROGRESS pending transition back to CODE-REVIEW.
- Ownership: owner=fork; fork_run=3p01b2t0f6t1; revision=7fce0975d; scope=merge origin/main and implement accepted Castor parsing, ownership routing/recording, write serialization, and Hatfield configuration fixes; outcome=assigned; commit=none
- Ownership: owner=fork; fork_run=3p01b2t0f6t1; revision=7fce0975d; scope=merge origin/main and implement accepted Castor parsing, ownership routing/recording, write serialization, and Hatfield configuration fixes; outcome=completed; commit=463bd6301
- Fork `3p01b2t0f6t1` completed merge commit `9564e6291872e1994e7ce3a72fac4a197fbfae18` and correction commit `463bd6301e9630aeb1b93cb48f9376775a4d3ca2` with no blockers. It read and followed the testing skill and tests/AGENTS.md before test work. Parent acceptance inspection found no additional issue.

## Task workflow update - 2026-08-27T20:52:34.387Z
- Recorded fork run: a1q9mccrojxw
- Ownership: owner=fork; fork_run=a1q9mccrojxw; revision=463bd6301; scope=replace Hatfield exhaustive fork handoff with finalized compact delta format and update native prompt tests; outcome=assigned; commit=none
- Ownership: owner=fork; fork_run=0bm5fc3b5ui8; revision=my-pi-dirty-main; scope=replace Pi fork task prompt with finalized compact delta format, preserve all pre-existing dirty changes, validate, and globally install the local extensions package; outcome=assigned; commit=none
- Latest finalized requirement: replace fork handoff format in both Hatfield and Pi. Hatfield changes belong to PR 435; Pi lives in `/home/ineersa/claw/my-pi`, is already dirty, must preserve existing changes without staging/committing, and must install only the local extensions package globally. The full my-pi installer is intentionally avoided because non-interactive mode can overwrite bundled global settings.

## Task workflow update - 2026-08-27T20:57:18.624Z
- Recorded fork run: a1q9mccrojxw
- Validation: castor test --filter=ForkTaskPromptBuilderTest — PASS (4 tests, 27 assertions); castor deptrac — PASS (0 violations); castor phpstan — PASS (0 errors); castor cs-check — PASS after cs-fix; castor docs:validate — PASS (16 documents); git diff --check — PASS; Targeted obsolete prompt wording search — PASS; Builder heredoc vs scratch draft exact comparison — PASS
- Summary: Hatfield compact fork handoff format implemented and committed as b85e6aa334184846e514f7309f4b213c0e18baf1. Builder prompt now matches finalized scratch draft; obsolete exhaustive 11-section contract replaced with compact delta-oriented format and native prompt tests. Scratch draft received both approved micro-edits. Worktree reported clean.
- Ownership: owner=fork; fork_run=a1q9mccrojxw; revision=463bd6301; scope=replace Hatfield exhaustive fork handoff with finalized compact delta format and update native prompt tests; outcome=completed; commit=b85e6aa334184846e514f7309f4b213c0e18baf1

## Task workflow update - 2026-08-27T20:58:10.295Z
- Recorded fork run: 0bm5fc3b5ui8
- Validation: npm run typecheck — PASS; git diff --check -- packages/extensions/extensions/fork/runner.ts — PASS; Targeted prompt marker/obsolete wording search — PASS; Final target diff inspection preserving pre-existing PATH-wrapper hunk — PASS; pi install /home/ineersa/claw/my-pi/packages/extensions — PASS; pi list global local-package registration — PASS
- Summary: Pi compact fork handoff format implemented in the pre-existing dirty `/home/ineersa/claw/my-pi` checkout without staging or committing. Only `buildForkTaskPrompt()` in runner.ts was changed by this fork; adjacent pre-existing PATH-wrapper and all other dirty changes were preserved. Local extensions bundle installed globally by path and verified in `pi list`.
- Ownership: owner=fork; fork_run=0bm5fc3b5ui8; revision=my-pi-dirty-main; scope=replace Pi fork task prompt with finalized compact delta format, preserve all pre-existing dirty changes, validate, and globally install the local extensions package; outcome=completed; commit=none

## Task workflow update - 2026-08-27T21:18:24.032Z
- Validation: Reviewer focused castor test for MoveTaskHandlerTest|WorktreeManagerCopyTest|ForkTaskPromptBuilderTest — PASS (25 tests, 174 assertions); Reviewer castor deptrac — PASS; Reviewer castor phpstan — PASS; Reviewer castor cs-check — PASS; Reviewer castor docs:validate — PASS; Reviewer git diff --check — PASS; Reviewer castor check — FAIL deterministically in test lane: stale assertions at ForkChildStartRunInputCompositionTest.php:330-331; report var/reports/qa-20260827-211540-519843-d1a0de3f/check-test.log
- Summary: Independent re-review at b85e6aa334184846e514f7309f4b213c0e18baf1 returned REQUEST CHANGES. Merge blocker: `ForkChildStartRunInputCompositionTest` still asserts the superseded 11-section fork prompt, causing deterministic focused and full-gate failure. Reviewer also found one remaining implementation-role `Worker can run composer install` comment in Pi worktree bootstrap, contrary to finalized fork-only terminology. PR #435 remote head is still 7fce0975d and will update only after fixes pass and move_task returns it to CODE-REVIEW.
- Review: role=reviewer; artifact_run_id=not-exposed-by-subagent-tool; revision=b85e6aa334184846e514f7309f4b213c0e18baf1; scope=full PR #435 re-review against finalized requirements and prior blockers; decision=REQUEST CHANGES; blocker=stale 11-section assertions in ForkChildStartRunInputCompositionTest deterministically fail castor check

## Task workflow update - 2026-08-27T21:18:46.580Z
- Recorded fork run: 6n969a3k8hos
- Ownership: owner=fork; fork_run=6n969a3k8hos; revision=b85e6aa334184846e514f7309f4b213c0e18baf1; scope=fix stale compact-handoff composition assertions, retire obsolete filler heading, and remove remaining implementation-role worker comment; outcome=assigned; commit=none

## Task workflow update - 2026-08-27T21:20:20.237Z
- Recorded fork run: 6n969a3k8hos
- Validation: castor test --filter='Fork.*Test|OutputCapToolResultProcessorContractTest' — PASS (40 tests, 208 assertions); castor deptrac — PASS (0 violations); castor phpstan — PASS (0 errors); castor cs-check — PASS; castor docs:validate — PASS (16 documents); git diff --check — PASS; Targeted obsolete prompt/worker wording search — PASS
- Summary: Re-review blocker corrected in commit 1653a7b82df7026d5297a76d04d5944a84e958c0. Stale 11-section composition assertions now protect the compact contract; obsolete output-cap filler heading retired; remaining implementation-role worker comment changed to fork terminology. Worktree reported clean.
- Ownership: owner=fork; fork_run=6n969a3k8hos; revision=b85e6aa334184846e514f7309f4b213c0e18baf1; scope=fix stale compact-handoff composition assertions, retire obsolete filler heading, and remove remaining implementation-role worker comment; outcome=completed; commit=1653a7b82df7026d5297a76d04d5944a84e958c0

## Task workflow update - 2026-08-27T21:23:26.486Z
- Summary: Focused final re-review at 1653a7b82df7026d5297a76d04d5944a84e958c0 returned APPROVED. Previous stale-test blocker is resolved; compact handoff markers are verified at builder and composition layers without whole-prompt duplication; correction scope is clean and exact.
- Review: role=reviewer; artifact_run_id=not-exposed-by-subagent-tool; revision=1653a7b82df7026d5297a76d04d5944a84e958c0; scope=focused final re-review of previous blocker correction and compact handoff contract; decision=APPROVED; blockers=none

## Task workflow update - 2026-08-27T21:23:46.440Z
- Validation: castor test:llm-real — PASS (5 tests, 30 assertions, 11.2s)
- Summary: Pre-CODE-REVIEW focused live LLM validation passed for the LLM-visible fork prompt changes.

## Task workflow update - 2026-08-27T21:25:33.296Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (92.9s).
- Pushed task/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates to origin.
- branch 'task/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates' set up to track 'origin/task/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates'.
- PR already exists: https://github.com/ineersa/agent-core/pull/435
- Validation: Independent reviewer — APPROVED at 1653a7b82df7026d5297a76d04d5944a84e958c0; castor test --filter='Fork.*Test|OutputCapToolResultProcessorContractTest' — PASS (40 tests, 208 assertions); castor deptrac — PASS (0 violations); castor phpstan — PASS (0 errors); castor cs-check — PASS; castor docs:validate — PASS (16 documents); castor test:llm-real — PASS (5 tests, 30 assertions, 11.2s); git diff --check — PASS
- Summary: Final re-review approved at 1653a7b82df7026d5297a76d04d5944a84e958c0 after correcting the stale compact-handoff composition test. Focused unit/static/docs and live LLM validation passed. Transitioning back to CODE-REVIEW updates existing PR #435.

## Task workflow update - 2026-08-27T21:36:06.380Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates: ide_close_project returned isError.
- Merged task/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates into integration checkout.
- Merge made by the 'ort' strategy.
 .hatfield/extensions/extension-api/AGENTS.md       |   6 +
 .../task-workflow/skills/task-workflow/SKILL.md    |  89 +++--
 .../task-workflow/src/Prompt/WorkflowPrompt.php    |  12 +-
 .../src/Settings/TaskWorkflowSettings.php          |   7 +-
 .../task-workflow/src/TaskWorkflowExtension.php    |  14 +-
 .../task-workflow/src/Tool/MoveTaskHandler.php     |  74 ++--
 .../task-workflow/src/Worktree/WorktreeManager.php |  63 +++-
 .../task-workflow/tests/MoveTaskHandlerTest.php    |  57 ++-
 .../tests/WorktreeManagerCopyTest.php              |  92 +++++
 .hatfield/prompts/simplify.md                      |  25 +-
 .hatfield/prompts/task-done.md                     |  44 +--
 .hatfield/prompts/task-explain.md                  |  51 +--
 .hatfield/prompts/task-review-iterate.md           |  67 +---
 .hatfield/prompts/task-start.md                    |  57 +--
 .hatfield/prompts/task-to-pr.md                    |  48 +--
 .hatfield/settings.yaml                            |   1 -
 .pi/extensions/task-workflow/index.ts              |  44 +--
 .pi/extensions/task-workflow/prompt.ts             |  17 +-
 .pi/extensions/task-workflow/worktrees.ts          |  45 ++-
 .pi/prompts/simplify.md                            |  25 +-
 .pi/prompts/task-done.md                           |  44 +--
 .pi/prompts/task-explain.md                        |  50 +--
 .pi/prompts/task-review-iterate.md                 |  67 +---
 .pi/prompts/task-start.md                          |  56 +--
 .pi/prompts/task-to-pr.md                          |  48 +--
 .pi/skills/task-workflow/SKILL.md                  |  87 +++--
 AGENTS.md                                          |  47 ++-
 docs/agents.md                                     |   6 +-
 .../Agent/Fork/ForkTaskPromptBuilder.php           | 411 +++++++++++----------
 src/CodingAgent/Resources/agents/reviewer.md       |   6 +-
 src/CodingAgent/Resources/agents/scout.md          |   2 +-
 .../Resources/skills/subagents/SKILL.md            |   4 +-
 src/CodingAgent/Runtime/AGENTS.md                  |   7 +
 src/Tui/AGENTS.md                                  |  11 +
 .../Fork/ForkChildStartRunInputCompositionTest.php |   9 +-
 .../Agent/Fork/ForkTaskPromptBuilderTest.php       | 106 ++----
 .../OutputCapToolResultProcessorContractTest.php   |   2 +-
 37 files changed, 754 insertions(+), 1047 deletions(-)
 create mode 100644 .hatfield/extensions/extension-api/AGENTS.md
 create mode 100644 src/CodingAgent/Runtime/AGENTS.md
 create mode 100644 src/Tui/AGENTS.md
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-harden-task-workflow-skill-prompts-and-code-review-gates.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #435 state — MERGED; CODE-REVIEW deterministic castor check — PASS (92.9s); Final independent reviewer — APPROVED at 1653a7b82df7026d5297a76d04d5944a84e958c0
- Summary: PR #435 was merged on GitHub at 2026-08-27T21:35:42Z as merge commit 58d16fe7277195df8b9299d4b9489716f9ca10ec. Moving task to DONE and synchronizing the clean integration checkout.

## Task workflow update - 2026-08-27T21:37:36.646Z
- Validation: LLM_MODE=true castor check on integration checkout — PASS (153.3s); Unit/integration lane — PASS (4906 tests, 20075 assertions); Controller replay — PASS (7 tests, 105 assertions); TUI replay — PASS (8 tests, 59 assertions); Live LLM — PASS (5 tests, 30 assertions); Deptrac/phpstan/cs-check/docs/catalog/PHAR smoke — PASS; QA artifact integrity/leak check/cache guard — PASS
- Summary: Post-merge integration validation completed successfully. Integration checkout is synchronized; task worktree and IDEA exclusions were removed. JetBrains close reported a degraded/non-fatal IDE close error before filesystem cleanup.

## Task workflow update - 2026-08-29T16:09:38.964Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
