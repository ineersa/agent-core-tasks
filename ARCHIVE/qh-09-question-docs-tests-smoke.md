# QH-09 Question/HITL docs, deterministic tests, and manual smoke (thin v1)

## Goal
Plan: .pi/plans/tui-question-hitl-plan.md

**V1 decision (2026-06-28):** Thin v1 only — docs/prompt guidance for `ask_human`, minimal deterministic tests for the live-session flow, manual smoke. No resume tests.

Scope:
- Update tool prompt/docs to teach ask_human usage and schema subset, noting that resume support is deferred.
- Add deterministic tests for the thin v1 ask_human flow (tool interrupt, payload shape, TUI overlay rendering, answer_human routing).
- Do **not** add tests for pending-HITL resume — that path is deferred.
- Add manual smoke steps using castor run:agent with a model/tool call to ask_human.
- Record known limitations, especially no full JSON Schema renderer and no resume support in v1.

Exclusions:
- Do not add live-model tests to castor check.
- Do not implement missing production APIs solely for tests.
- Do not broaden to safety policy/approval guard implementation.
- Do not add resume-pending-HITL tests.

Dependencies: QH-04, QH-05, QH-06.
Parallelizable with: none.

## Acceptance criteria
- Docs/prompt guidance explain when to use ask_human and note that resume support is deferred.
- Tests cover thin v1 flow: ask_human → interrupt → human_input.requested → TUI overlay → answer_human → run continues.
- Manual smoke verifies ask_human -> TUI question -> answer_human -> run continues.
- Known limitations documented (no full JSON Schema renderer, no resume).
- Relevant castor test filters and castor deptrac pass.

## Workflow metadata
Status: DONE
Branch: task/qh-09-question-docs-tests-smoke
Worktree: /home/ineersa/projects/agent-core-worktrees/qh-09-question-docs-tests-smoke
Fork run: 4be83322a
PR URL: https://github.com/ineersa/agent-core/pull/236
PR Status: merged
Started: 2026-06-29T18:33:22.642Z
Completed: 2026-06-29T19:27:50.953Z

## Work log
- Created: 2026-05-18T00:05:00.782Z

## Task workflow update - 2026-06-29T18:33:22.642Z
- Moved TODO → IN-PROGRESS.
- Created branch task/qh-09-question-docs-tests-smoke.
- Created worktree /home/ineersa/projects/agent-core-worktrees/qh-09-question-docs-tests-smoke.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/qh-09-question-docs-tests-smoke.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/qh-09-question-docs-tests-smoke.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/qh-09-question-docs-tests-smoke.

## Task workflow update - 2026-06-29T18:40:17.720Z
- Recorded fork run: 4be83322a
- Validation: castor test: OK 3812 tests / 12150 assertions (docs-only, trivially green); castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean (0 files fixed); independent verification: DTO table matches QuestionRequest.php source exactly; secret removed; cancel signal + escape hatch + v1 limitations all source-accurate
- Summary: Fork 4be83322a rewrote docs/hitl-and-approvals.md (+431/−51, single file). Documents the previously-undocumented second HITL path (ask_human → human_input.requested → WaitingHuman → answer_human) alongside the existing SafeGuard/ToolQuestion approval path. No production/test code changed.

Independently verified:
- Doc structure: two paths clearly delineated (Path A extension approvals / Path B agent-driven questions), 11 major sections
- QuestionRequest DTO table matches QuestionRequest.php exactly (every field/type/default); stale `secret` row removed (QH-06); allowOther semantics corrected (always-on HITL / false ToolQuestion); default clarified (pass-through only, no widget auto-select in v1)
- Cancel signal correctly documents 'Cancelled by user' (HITL) vs 'cancel' (ToolQuestion) with rationale
- Escape hatch correctly scoped to Choice-only, always-on for HITL
- v1 limitations section: 5 honest items (no resume, no JSON Schema renderer, default no auto-select, allowOther always-on, secret removed)
- Manual smoke section covers full surface

Test coverage: NO new test added (fork decision, sound per AGENTS.md test-value guidance): layer tests cover every link of the thin v1 flow (AskHumanToolTest, ToolExecutorTest, ToolCallExtractorTest, RuntimeEventMapperTest, TranscriptProjectorTest, TickPollListenerTest, AnswerHumanHandlerTest), user already live-smoke-tested. A controller-replay test would require non-trivial fixture infra and risk gate flakiness for no distinct-contract protection beyond what layer tests + live smoke provide.

Validation: castor test 3812/12150, deptrac 0, phpstan 0, cs-check clean (docs-only, trivially green).

## Task workflow update - 2026-06-29T18:41:55.166Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (58.5s).
- Pushed task/qh-09-question-docs-tests-smoke to origin.
- branch 'task/qh-09-question-docs-tests-smoke' set up to track 'origin/task/qh-09-question-docs-tests-smoke'.
- Created PR: https://github.com/ineersa/agent-core/pull/236

## Task workflow update - 2026-06-29T19:27:50.954Z
- Moved CODE-REVIEW → DONE.
- Merged task/qh-09-question-docs-tests-smoke into integration checkout.
- Merge made by the 'ort' strategy.
 docs/hitl-and-approvals.md | 482 ++++++++++++++++++++++++++++++++++++++++-----
 1 file changed, 431 insertions(+), 51 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/qh-09-question-docs-tests-smoke.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/qh-09-question-docs-tests-smoke.
- Pulled integration checkout: Merge made by the 'ort' strategy..
