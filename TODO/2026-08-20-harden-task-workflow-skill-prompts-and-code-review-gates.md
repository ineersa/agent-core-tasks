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
Status: TODO
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-08-20T16:08:43+00:00
