# coding-agent-01: Consolidate CodingAgent JSONL storage and runtime encoding

## Goal
## Goal
Remove duplicate canonical JSONL mechanics across CodingAgent session/child-run persistence and runtime stdout sinks without changing the persisted or wire protocol.

## Architecture report evidence

### Duplicate event stores (candidate 1)
- `src/CodingAgent/Session/SessionRunEventStore.php` (~284 lines) and `src/CodingAgent/Agent/Artifact/AgentChildRunEventStore.php` (~281 lines) are near-identical.
- Both duplicate lock acquisition, `FileRunSequenceAllocator::allocateNext()`, `EventLogMaxSeqBootstrapReader`, `EventPayloadNormalizer`, append/appendMany, decode/read, and sequence sorting.
- Differences are path resolution and the child store's embedded-run-ID assertion.
- `depfile.yaml` already permits the relevant reuse direction (`AppAgent → AppSession`); do not invent a stricter boundary.
- The canonical event log is versioned replay data, so format and concurrency fixes should have one owner.

### Runtime codec drift (candidate 4)
- `src/CodingAgent/Runtime/Protocol/JsonlCodec.php` already owns `encodeEvent()`.
- `src/CodingAgent/Runtime/Stream/StdoutRuntimeEventSink.php` and `CommittedRuntimeEventStdoutSink.php` bypass it with direct `json_encode()`.
- The sinks omit `JSON_UNESCAPED_SLASHES`, producing multiple encodings of the canonical runtime wire format.
- Both sinks also duplicate stdout handle/pipe detection.

## Smallest viable direction
- Deepen one existing event-log component or introduce one minimal internal event-log engine parameterized only by path resolution and the child embedded-ID assertion.
- Keep public store contracts and path ownership unchanged.
- Route both runtime sinks through `JsonlCodec::encodeEvent()` and one existing/shared stdout-pipe primitive.
- Prefer deletion over a hierarchy of interfaces/factories; no speculative storage abstraction.

## Scope boundaries
- No event schema, ordering, path, locking, sequence allocation, replay, or public runtime protocol changes.
- Preserve the child store's stricter embedded run-ID validation.
- Do not mix unrelated session-storage redesign or test refactoring into this task.

## Test impact noted by the report
The report suggested consolidating duplicated coverage from `SessionRunEventStoreTest`, `AgentChildRunEventStoreTest`, and `ChildAwareEventStoreTest` into one boundary suite with path-strategy cases. Follow `tests/AGENTS.md`: retain behavior-focused coverage and combine only where it reduces duplication without obscuring one-class-per-production-class ownership. Runtime sink proof must assert codec output and pipe behavior.

## Dependencies
Coordinate with the active `reuse-symfony-components-outside-tui` task if it changes adjacent session persistence; rebase rather than duplicate its changes.

## Acceptance criteria
- One implementation owns shared JSONL append, sequence allocation/bootstrap, decode, and sort mechanics for session and child-run event stores.
- Session and child-run paths, locking semantics, event ordering, malformed-record behavior, and child embedded-run-ID validation remain unchanged.
- Both runtime stdout event sinks use `JsonlCodec::encodeEvent()`; canonical flags no longer drift.
- Duplicated stdout handle/pipe detection is removed or delegated to an existing primitive without adding a broad abstraction.
- A focused regression check covers both store path/validation variants and proves sink output matches the codec.
- The fork and handoff state that `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` were read and followed.
- Focused Castor tests, `castor test:controller-replay`, `castor deptrac`, `castor phpstan`, `castor cs-check`, and final `castor check` pass.

## Workflow metadata
Status: ARCHIVE
Branch: task/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding
Worktree: /home/ineersa/projects/agent-core-worktrees/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding
Fork run: s2v11wtdp1kt
PR URL: https://github.com/ineersa/agent-core/pull/397
PR Status: merged
Started: 2026-08-16T20:46:42.700Z
Completed: 2026-08-16T22:02:05.305Z

## Work log
- Created: 2026-08-15T23:06:31.077Z

## Task workflow update - 2026-08-15T23:19:03.265Z
- Full architect report: [`/home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md`](file:///home/ineersa/.hatfield/dumps/agent-core-architecture-report-2026-08-15.md) — CodingAgent report, candidates 1 and 4.

## Task workflow update - 2026-08-16T20:46:42.700Z
- Moved TODO → IN-PROGRESS.
- Created branch task/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding.
- Created worktree /home/ineersa/projects/agent-core-worktrees/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding.
- Summary: Claimed for implementation. Scope: consolidate duplicated session/child-run JSONL mechanics and route both runtime stdout sinks through JsonlCodec without protocol or storage behavior changes.

## Task workflow update - 2026-08-16T21:00:22.252Z
- Summary: Current-code scouting complete. Architecture report is stale in two important ways: SessionRunEventStore now has size+mtime decode caching, while AgentChildRunEventStore has recovery-only readAfterSeq(), streaming reads, and child-specific logging. The earlier reuse-symfony-components task is already merged. Implementation will consolidate append/allocation/bootstrap plus primitive decode/denormalize/schema/sort mechanics while retaining each wrapper's distinct read/cache/log/validation policy. Runtime sinks will reuse StdoutRuntimeEventSink as the shared stdout pipe writer and preserve transient-throw vs committed-log-and-swallow behavior.
- Task-start scouts read root/runtime instructions, testing skill, and tests/AGENTS.md. No tests or QA run during scouting.
- External surface mapping: persisted event schema/paths/locks/order unchanged; runtime wire bytes intentionally align slash encoding with existing JsonlCodec per acceptance; no settings, commands, interfaces, or provider behavior added.
- Focused validation plan: store/runtime sink tests, castor test:controller-replay, deptrac, phpstan, cs-check. llm-real and TUI lanes are not focused task-start requirements; final castor check remains task-to-pr.

## Task workflow update - 2026-08-16T21:00:26.372Z
- Recorded fork run: a8tju48g9mj1
- Implementation fork launched in exact task worktree with current-code storage/runtime design, strict behavior-preservation boundaries, focused Castor validation, and explicit no-castor-check/no-PR stop conditions.

## Task workflow update - 2026-08-16T21:17:34.766Z
- Validation: Fork: focused Castor filter PASS (50 tests, 147 assertions) twice; Fork: castor test PASS (4496 tests, 17671 assertions) on three reruns; first run had one non-reproduced flaky failure; Fork: castor test:controller-replay PASS (12 tests, 165 assertions); Fork: castor deptrac PASS (0 violations); Fork: castor phpstan PASS (0 errors); Fork: castor cs-check PASS (0 files fixed after cs-fix); Parent: git diff --check clean; IDE diagnostics 0 problems in JsonlRunEventLog and StdoutRuntimeEventSink
- Summary: Implementation fork completed at 1a106d95f: shared JsonlRunEventLog composition, preserved divergent parent cache/child streaming policies, canonical JsonlCodec stdout bytes for transient+committed sinks, subprocess byte proofs, and child bootstrap proof. Initial validation is green. Parent inspection found two trivial minimality follow-ups before accepting task-start: remove an unnecessary default-constructed stdout collaborator compatibility shim, and remove the shared engine's one-line append convenience in favor of its sole contiguous-block primitive; also repair the now-unqualified FileRunSequenceAllocator doc reference.

## Task workflow update - 2026-08-16T21:17:38.453Z
- Recorded fork run: s2v11wtdp1kt
- Minimality follow-up fork launched: remove constructor compatibility default, delete one-line shared append convenience, require new engine dependency explicitly, and repair one doc reference. No scope or behavior changes.

## Task workflow update - 2026-08-16T21:20:34.290Z
- Validation: Focused affected-class filter PASS twice on implementation (50 tests, 147 assertions); follow-up focused filter PASS (32 tests, 96 assertions); castor test PASS (4496 tests, 17671 assertions) on three clean reruns; first attempt had one non-reproduced flaky failure whose overwritten report did not retain the test name; castor test:controller-replay PASS (12 tests, 165 assertions); castor deptrac PASS (0 violations); castor phpstan PASS (0 errors), including after minimality follow-up; castor cs-check PASS (0 files fixed), including after minimality follow-up; git diff --check clean; JetBrains diagnostics: 0 problems in JsonlRunEventLog, SessionRunEventStore, AgentChildRunEventStore, StdoutRuntimeEventSink, and CommittedRuntimeEventStdoutSink
- Summary: Task-start implementation complete and committed as 1a106d95f + 3085b065e on task/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding. Cumulative diff: 8 files, +357/-246; production src +238/-245 (net -7), tests +119/-1. Added internal JsonlRunEventLog as the single append/sequence/bootstrap/normalize/decode/denormalize/schema/sort owner while retaining SessionRunEventStore's size+mtime cache and AgentChildRunEventStore's bound-ID/path/0755/streaming/readAfterSeq/logging policies. StdoutRuntimeEventSink now owns shared pipe detection/handle/write and encodes through JsonlCodec; CommittedRuntimeEventStdoutSink delegates while preserving log-and-swallow behavior. Added child sequence-bootstrap and exact slash-sensitive subprocess wire-byte proofs for both sinks. Minimality follow-up removed the engine's one-line append convenience and an unnecessary sink collaborator default. Worktree clean, branch not pushed, no PR, castor check deferred to task-to-pr.

## Task workflow update - 2026-08-16T21:38:19.265Z
- Validation: Reviewer APPROVED cumulative 1a106d95f + 3085b065e; testing skill and tests/AGENTS.md read/followed; Task-to-pr castor test PASS (4496 tests, 17671 assertions); Task-to-pr castor deptrac PASS (0 violations, 0 errors); Task-to-pr castor phpstan PASS (0 errors); Task-to-pr castor cs-check PASS (0 files fixed)
- Summary: Task-to-pr review APPROVED with no blockers. Reviewer verified exact storage-path/lock/allocation/bootstrap/cache/streaming/schema/malformed/error semantics; stdout tri-state/failure behavior and canonical codec bytes; autowiring; test isolation; and specification fidelity. No unmapped external surface or unnecessary abstraction found. Non-blocking notes only: appendMany's non-empty precondition could be documented, and dead-pipe failure-path tests remain a pre-existing gap; neither merits scope expansion.

## Task workflow update - 2026-08-16T21:40:46.156Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 600s)...
- castor check passed (135.7s).
- Pushed task/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding to origin.
- branch 'task/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding' set up to track 'origin/task/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding'.
- Created PR: https://github.com/ineersa/agent-core/pull/397

## Task workflow update - 2026-08-16T21:40:50.789Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/397
- Updated PR Status: open
- Validation: Deterministic castor check PASS (135.7s)
- Summary: Task-to-pr complete: branch pushed, deterministic castor check passed in 135.7s, PR #397 opened, task moved to CODE-REVIEW.

## Task workflow update - 2026-08-16T22:02:05.305Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding.
- Merged task/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding into integration checkout.
- Merge made by the 'ort' strategy.
 .../Agent/Artifact/AgentChildRunEventStore.php     | 130 ++++-------------
 .../Stream/CommittedRuntimeEventStdoutSink.php     |  51 +------
 .../Runtime/Stream/StdoutRuntimeEventSink.php      |  28 +++-
 src/CodingAgent/Session/JsonlRunEventLog.php       | 159 +++++++++++++++++++++
 src/CodingAgent/Session/SessionRunEventStore.php   | 115 +++------------
 .../Agent/Artifact/AgentChildRunEventStoreTest.php |  19 +++
 .../Stream/CommittedRuntimeEventStdoutSinkTest.php |  38 ++++-
 .../Runtime/Stream/StdoutRuntimeEventSinkTest.php  |  63 ++++++++
 8 files changed, 357 insertions(+), 246 deletions(-)
 create mode 100644 src/CodingAgent/Session/JsonlRunEventLog.php
 create mode 100644 tests/CodingAgent/Runtime/Stream/StdoutRuntimeEventSinkTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/coding-agent-01-consolidate-jsonl-storage-and-runtime-encoding.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #397 state MERGED at 2026-08-16T22:01:43Z
- Summary: PR #397 merged on GitHub as e62b7a8fa6cc022b4a75353d9320a18f9fc5d8bb. Moving task to DONE and syncing integration checkout.

## Task workflow update - 2026-08-16T22:04:39.415Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/397
- Updated PR Status: merged
- Validation: Post-merge LLM_MODE=true castor check PASS — qa-20260816-220208-652363-a21dc89f; deptrac PASS; test PASS (4496 tests, 17671 assertions); test:controller-replay PASS (12 tests, 165 assertions); test:tui PASS (37 tests, 296 assertions); test:llm-real PASS (13 tests, 144 assertions); phpstan PASS (0 errors); cs-check PASS; docs:validate PASS; QA artifact integrity and leak check PASS; llama-proxy cache stable 326 → 326
- Summary: Task DONE. PR #397 merged, integration checkout synchronized, task worktree/IDE exclusions removed, and post-merge integration QA passed all eight lanes.

## Task workflow update - 2026-08-18T00:06:36.237Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
