# Extract shared Markdown rendering support from recovery draft

## Goal
User approved splitting standalone improvements out of draft PR541 before replacing its shared-state architecture. Extract only shared Symfony Markdown parser/highlighter support and its focused deterministic regression from source commit932ce8500. Base new work on main, not shared cache architecture. Preserve rendering behavior and existing public APIs; no runtime/state/cache/claim-recovery/logging changes. Parent recovery task retains redesign scope. This is first independently reviewable slice, not a complete memory fix.

## Acceptance criteria
- Per-screen renderers reuse Markdown parser/highlighter through existing Symfony facilities.
- Focused regression proves sharing reduces allocation without requiring shared RunState/history cache architecture.
- No unrelated changes to session recovery, canonical ownership, metadata policies, worker claims or memory limits.
- Focused Castor tests, static/architecture/style checks pass; full gate remains task-to-pr.

## Workflow metadata
Status: CODE-REVIEW
Branch: task/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/542
PR Status: open
Started: 2026-10-01T21:11:00+00:00
Completed:

## Work log
- Created: 2026-10-01T21:09:13+00:00

## Task workflow update - 2026-10-01T21:11:00+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft/.idea.
- Summary: User rolled back the eight integration-main test edits; verified both main and recovery worktree clean. Proceed with independent Markdown extraction from932ce8500 onto main4f107b034.

## Task workflow update - 2026-10-01T21:11:33+00:00
- Summary: Main owns bounded nine-file extraction. Read source diff: six Transcript production paths plus standalone subprocess memory fixture/test and SubagentResultRenderer test adaptation. No shared projection/runtime dependency needed; source reference932ce8500, targetmain4f107b034. QuestionOverlay comment-only change omitted. Existing renderers/factory, Symfony MarkdownWidget constructor and focused regression are the proof boundary.
- Ownership: owner=main; fork_run=none; revision=4f107b034; scope=extract independent Markdown parser/highlighter sharing and focused allocation regression; outcome=assigned; commit=none

## Task workflow update - 2026-10-01T21:14:58+00:00
- Validation: Focused Castor70tests393assertions PASS,0.650s PHPUnit/1.1s runner.; Counterfactual Castor memory case fails75497472bytes vs50331648budget; restored source passes29360128bytes at128M hardlimit.; Castor phpstan PASS0errors; deptrac PASS0violations; cs-check PASS; gitdiffcheck PASS.; No actual-session128M fix or long-term plateau claim. No original live session or integration-main edits.
- Summary: Independent Markdown extraction implemented on clean main baseline and committed32c8c51df: nine files only, no shared cache/state/runtime ownership changes. Corrected misleading historical fixture claims: synthetic render load is not actual production session2 window. Repeated counterfactual removing shared parser/highlighter fails at72MiB against48MiB budget; restored sharing passes at28MiB. Not pushed or moved toCODE-REVIEW; independent review/full transition gate still required.
- Ownership: owner=main; fork_run=none; revision=32c8c51df; scope=independent shared Markdown renderer extraction and allocation regression; outcome=completed; commit=32c8c51df

## Task workflow update - 2026-10-01T23:52:32+00:00
- Summary: Independent reviewer=agent_d4e9f04be34471e0; target=32c8c51df; scope=entire independent extraction; specification-fidelity verdict=APPROVE WITH SUGGESTIONS.

## Task workflow update - 2026-10-01T23:57:58+00:00
- Summary: Independent review approved by agent_d4e9f04be34471e0; reviewed tree committed at 32c8c51df. Specification fidelity confirmed for independent Change1 scope; no unresolved blocking review findings. Ready for transition-owned full QA gate.

## Task workflow update - 2026-10-02T00:00:01+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (100.0s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft/var/reports/qa-20261001-235821-10968-e15b9fea.
- Session/run: 75.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-02T00:00:03+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft/var/reports/qa-20261001-235821-10968-e15b9fea.
- Session/run: 75.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-02T00:00:07+00:00
- castor check passed (100.0s).
- Pushed task/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft to origin.
- Created PR: <url>
- Session/run: 75.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-02T00:00:07+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (100.0s).
- Pushed task/2026-10-01-extract-shared-markdown-rendering-support-from-recovery-draft to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/542
