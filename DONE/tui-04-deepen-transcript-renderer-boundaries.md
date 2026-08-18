# tui-04: Deepen transcript renderer boundaries after factory split

## Goal
## Goal
Turn PR #399's mostly mechanical factory decomposition into clearer ownership boundaries where it produces real value, without replacing the old god class with a renderer framework or one class per tool.

## Context
PR #399 reduced `TranscriptBlockWidgetFactory` and introduced `MutableTranscriptWidget`, `TranscriptToolPresentationPolicy`, `TranscriptToolRenderer`, and `TranscriptBlockRenderer`. A post-PR architect audit found:
- the mutable-widget contract is a genuine improvement;
- pairing/suppression is a meaningful extracted policy;
- the two renderer extractions are mostly verbatim relocation;
- the factory constructor/public API/call graph stayed unchanged;
- `TranscriptToolPresentationPolicy` mixes pairing/suppression policy with result-body presentation facts used by `TranscriptToolRenderer`;
- extracted boundaries have no direct behavioral proof.

Reference: PR https://github.com/ineersa/agent-core/pull/399 and task `tui-03-deepen-transcript-widget-architecture`.

## Smallest viable direction
- Trace all production callers before changing the facade or any public method.
- Keep `TranscriptToolRenderer` as one cohesive tool-rendering module unless concrete coupling evidence supports a smaller owner; do not split per tool name.
- Separate tool pairing/suppression decisions from result presentation/formatting facts so the policy no longer serves as a mixed helper bag for the renderer.
- Remove dead or accidental facade surface only after semantic reference checks prove no callers remain.
- Add the smallest behavior-level proof at the real policy boundary for scoring/selection/suppression invariants not already protected through mounted transcript tests.
- Prefer concrete owners. Do not add interfaces, optional injection seams, DTOs, or production APIs solely for tests or hypothetical alternate implementations.

## Scope boundaries
- No transcript visual, ordering, grouping, suppression, pairing, preview, theme, question, tool, streaming, replay, or stable-key behavior changes.
- No one-class-per-tool/label decomposition, generic renderer registry/framework, service locator, compatibility facade, DI expansion, setting, command, or ExtensionApi change.
- Do not mix viewport/pagination, session scope, projector rewrite, or top-level widget migration into this task.
- Deletion and direct ownership are preferred over preserving unused internal APIs.

## Acceptance criteria
- Production references and call hierarchy are checked before changing or deleting facade/policy methods.
- Pairing/suppression policy owns only pairing/suppression decisions; renderer-specific formatting/result-body facts have one coherent owner.
- Any dead `TranscriptBlockWidgetFactory` public/accessor surface is removed only when no production callers remain; remaining facade methods each have a real orchestration purpose.
- No per-tool renderer classes, generic supports/router framework, service locator, compatibility shim, or test-only production seam is introduced.
- At least one focused behavior test protects a real extracted policy invariant not already adequately covered; no implementation-existence or trivial delegation tests.
- All transcript visuals, pairing/suppression, ordering, previews, themes, questions, tools, streaming, replay, stable keys, and incremental identity remain unchanged.
- The fork and handoff state that `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` were read and followed.
- Focused virtual transcript tests, `castor test`, `castor test:controller-replay`, `castor test:tui`, `castor deptrac`, `castor phpstan`, `castor cs-check`, and final `castor check` pass.

## Workflow metadata
Status: DONE
Branch: task/tui-04-deepen-transcript-renderer-boundaries
Worktree: /home/ineersa/projects/agent-core-worktrees/tui-04-deepen-transcript-renderer-boundaries
Fork run: 1qaii92gg3af
PR URL: https://github.com/ineersa/agent-core/pull/407
PR Status: merged
Started: 2026-08-18T02:04:00.503Z
Completed: 2026-08-18T14:12:33.194Z

## Work log
- Created: 2026-08-17T02:19:42.553Z

## Task workflow update - 2026-08-18T02:04:00.503Z
- Moved TODO → IN-PROGRESS.
- Created branch task/tui-04-deepen-transcript-renderer-boundaries.
- Created worktree /home/ineersa/projects/agent-core-worktrees/tui-04-deepen-transcript-renderer-boundaries.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/tui-04-deepen-transcript-renderer-boundaries.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/tui-04-deepen-transcript-renderer-boundaries.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/tui-04-deepen-transcript-renderer-boundaries.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/tui-04-deepen-transcript-renderer-boundaries.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/tui-04-deepen-transcript-renderer-boundaries/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/tui-04-deepen-transcript-renderer-boundaries.

## Task workflow update - 2026-08-18T02:23:37.562Z
- Recorded fork run: 1qaii92gg3af
- Validation: castor test — OK 4599 tests, 18285 assertions; castor test:controller-replay — OK 12 tests, 165 assertions (110s); castor test:tui --filter=TuiToolExchangeCardE2eTest — OK 1 test (9.2s); castor test:tui full group — OK 39 tests, 321 assertions (159s); castor deptrac — 0 violations; castor phpstan — 0 errors; castor cs-check --path (7 changed files) — clean; php -l all 7 files — OK; castor check deferred to CODE-REVIEW transition gate (as with prior tui tasks)
- Summary: Implementation complete (commit 45a6a97c1, 7 files +762/−95). Extracted TranscriptToolResultFacts (toolResultIsFullRender/toolResultBodyText/metaIsTruthy + edit-truncation, verbatim moves) — policy now owns pairing/suppression only, renderer has zero policy dep, both share one facts instance composed in the factory. Deleted dead factory displayConfig() accessor after repo-wide zero-caller verification. New behavior tests: TranscriptToolPresentationPolicyTest (scoring: full-render bonus dominance, streaming penalty, seq tiebreak; tool-name compatibility; ask_human error bypass; empty-assistant-placeholder suppression) + TranscriptToolResultFactsTest (body fallbacks, edit marker truncation, metaIsTruthy edges). New replay-backed TmuxHarness E2E TuiToolExchangeCardE2eTest (edit fixture) proves moved facts on live terminal path — non-vacuous: raw events.jsonl contains 'Updated file context' ×2, rendered capture does not.

## Task workflow update - 2026-08-18T02:34:47.779Z
- Validation: Reviewer subagent: APPROVED (spec fidelity gate pass, verbatim-move pass, behavior-preservation pass, dead-surface pass, test-quality pass, project-rules pass, diff-scope pass); castor deptrac — violations=0, errors=0; castor cs-check — files_fixed=0; Fork-validated on same commit 45a6a97c1: castor test 4599/18285 OK, castor test:controller-replay 12/165 OK, castor test:tui 39/321 OK, castor phpstan 0 errors
- Summary: Reviewer APPROVED (subagent review on worktree): verbatim moves verified byte-identical vs origin/main; 12 rewired call sites semantically identical; displayConfig() confirmed dead since introduction (f9afd9021) with zero callers across src/tests/config/extensions/docs; diff confined to exactly 7 claimed files; tests assert behavior at correct layers. Two NTH notes deferred: literal expected string in YAML-dump facts test; shared E2E settings builder (test-infra cleanup, out of scope). Orchestrator sanity re-run: castor deptrac 0 violations, castor cs-check files_fixed=0.

## Task workflow update - 2026-08-18T02:37:53.084Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (158.7s).
- Pushed task/tui-04-deepen-transcript-renderer-boundaries to origin.
- branch 'task/tui-04-deepen-transcript-renderer-boundaries' set up to track 'origin/task/tui-04-deepen-transcript-renderer-boundaries'.
- Created PR: https://github.com/ineersa/agent-core/pull/407

## Task workflow update - 2026-08-18T14:12:33.194Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/tui-04-deepen-transcript-renderer-boundaries: ide_close_project returned isError.
- Merged task/tui-04-deepen-transcript-renderer-boundaries into integration checkout.
- Merge made by the 'ort' strategy.
 .../Transcript/TranscriptBlockWidgetFactory.php    |  20 +-
 .../TranscriptToolPresentationPolicy.php           |  80 +------
 src/Tui/Transcript/TranscriptToolRenderer.php      |  26 +--
 src/Tui/Transcript/TranscriptToolResultFacts.php   |  83 +++++++
 tests/Tui/E2E/TuiToolExchangeCardE2eTest.php       | 255 +++++++++++++++++++++
 .../TranscriptToolPresentationPolicyTest.php       | 212 +++++++++++++++++
 .../Transcript/TranscriptToolResultFactsTest.php   | 181 +++++++++++++++
 7 files changed, 762 insertions(+), 95 deletions(-)
 create mode 100644 src/Tui/Transcript/TranscriptToolResultFacts.php
 create mode 100644 tests/Tui/E2E/TuiToolExchangeCardE2eTest.php
 create mode 100644 tests/Tui/Transcript/TranscriptToolPresentationPolicyTest.php
 create mode 100644 tests/Tui/Transcript/TranscriptToolResultFactsTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/tui-04-deepen-transcript-renderer-boundaries.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/tui-04-deepen-transcript-renderer-boundaries.
- Pulled integration checkout: Merge made by the 'ort' strategy..
