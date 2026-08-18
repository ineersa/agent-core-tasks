# Reuse Symfony components outside the TUI

## Goal
## Goal
Delete non-TUI custom wheels where installed Symfony 8.1 components already own the primitive. This task covers AgentCore, CodingAgent, Platform, persistence, config, events, processes, filesystem, IDs, and serialization. TUI work is tracked separately in `replace-text-wrappers-with-proper-symfony-tui-widgets`.

## Candidate slices

### Events, handlers, and Messenger
- Prove external extensions do not reference `AgentCore/Contract/Extension/EventSubscriberInterface`, then delete the unused custom subscriber contract.
- Prove no listener depends on `BoundaryHookEvent` / `after_turn_commit`, then delete the inert Serializer round-trip/EventDispatcher bridge from `HookDispatcher`; retain the real typed `HookSubscriberInterface` aggregation, priorities, and failure isolation.
- Re-audit `CommandRouter` / `CommandHandlerInterface` / `CommandMailboxPolicy` and the `ext:` route. Delete only after proving no public or extension behavior depends on it; otherwise record KEEP with callers.
- Keep custom run CAS/idempotency, commit-before-effects, runtime projections, dynamic extension registries, and failure recovery where they encode domain semantics rather than framework plumbing.

### Filesystem, locking, discovery, IDs, append
- Replace `SessionRunStore::compareAndSwap()` in-place `state.json` write with `Filesystem::dumpFile()` so unlocked readers see old-or-complete state. Preserve lock/CAS/path/failure semantics.
- Assess `TaskBoardStore` discovery for direct `Finder` replacement, and duplicate non-recursive Markdown discovery in prompt/agent loaders for one existing Symfony primitive. Apply only when filtering/order/precedence remain explicit and net code decreases.
- Replace `uniqid()` command IDs with Symfony `Uid` where those IDs require collision-safe identity; preserve externally observed string contracts.
- Assess `Filesystem::appendToFile()` for JSONL idempotency/event stores. Keep custom append logic where locking, fsync, recovery, or partial-record semantics exceed the component primitive.
- Keep `TaskBoardLock`, `FileRunSequenceAllocator`, process lifecycle, and JSONL process client when audit confirms they own concurrency/protocol behavior absent from Symfony.

### Serializer, Validator, Config, and public formats
- Replace `ForksConfigDTO::fromRaw()` / nested raw parsing with configured Serializer/Validator use while preserving the forbidden `agents.extensions.enabled` semantic rule.
- Replace MCP catalog DTO `fromArray()/toArray()` graphs with direct shared Serializer use at `SessionFileMcpToolCatalogStore`; preserve documented camelCase keys and strict malformed-data failures.
- Replace `CodexAuthRecord::fromArray()/toArray()` with direct shared Serializer use at `CodexAuthStorage`; preserve `accountId`, file permissions, atomic writes, and fail-closed behavior.
- Audit AI config manual parsing and runtime JSONL casts against Symfony Config/Serializer, but migrate only a complete owning boundary that removes code without duplicating DTO validation or changing the public protocol.
- No Codec, Decoder, Mapper, custom normalizer, duplicate Serializer stack, or compatibility shim unless a concrete Symfony limitation and public compatibility requirement are demonstrated first.

## Discipline
Implement sequential independently reviewable slices. Trace all callers and extension references first. A candidate that adds more glue than it deletes is a documented KEEP, not a forced migration.

## Acceptance criteria
- TUI files and behavior are not changed by this task.
- Runtime/config LOC decreases overall; no new Composer dependency and no wrapper abstraction around Symfony.
- Unused event/subscriber and inert boundary-hook machinery are deleted only after repository plus extension reference proof; live typed hooks remain unchanged.
- SessionRunStore writes are atomic for unlocked readers and retain CAS/lock behavior, with a runnable concurrent regression proof.
- Direct Serializer/Validator migrations preserve public key casing, strict malformed-data behavior, permissions, and atomic persistence.
- Filesystem/Finder/Uid replacements preserve ordering, precedence, identity, concurrency, and recovery contracts; unsuitable candidates are explicitly recorded as KEEP.
- No public `ExtensionApi` or `ext:` behavior is removed without explicit evidence and user approval.
- Focused boundary tests plus `castor test`, `castor deptrac`, `castor phpstan`, `castor cs-check`, and final `castor check` pass.
- Final reviewer inventories retained custom primitives and verifies each encodes domain behavior absent from Symfony; Ponytail verdict is lean.

## Workflow metadata
Status: ARCHIVE
Branch: task/reuse-symfony-components-outside-tui
Worktree: /home/ineersa/projects/agent-core-worktrees/reuse-symfony-components-outside-tui
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/389
PR Status: merged
Started: 2026-08-15T22:44:44.552Z
Completed: 2026-08-15T23:37:55.975Z

## Work log
- Created: 2026-08-14T23:40:43.263Z

## Task workflow update - 2026-08-15T22:44:44.552Z
- Moved TODO → IN-PROGRESS.
- Created branch task/reuse-symfony-components-outside-tui.
- Created worktree /home/ineersa/projects/agent-core-worktrees/reuse-symfony-components-outside-tui.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/reuse-symfony-components-outside-tui.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/reuse-symfony-components-outside-tui.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/reuse-symfony-components-outside-tui.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/reuse-symfony-components-outside-tui.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/reuse-symfony-components-outside-tui/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/reuse-symfony-components-outside-tui.
- Summary: Finalized reduced scope: implement only atomic SessionRunStore writes via Symfony Filesystem::dumpFile() and delete proven-dead custom event subscriber/boundary-hook bridge. Explicitly exclude Finder, appendToFile, UID, Serializer/config/auth/MCP, command routing, TUI, and other candidates.

## Task workflow update - 2026-08-15T23:02:52.126Z
- Validation: castor test --filter='SessionRunStoreTest\|HookDispatcherContractTest\|RunCommitAfterTurnCommitPersistedSeqTest\|ToolBatchSnapshotCleanupHookSubscriberTest\|AggregateResumeTest' — OK (17 tests, 102 assertions); Regression mutation proof with old in-place file_put_contents restored temporarily — expected failure with CORRUPT_JSON; implementation restored afterward; castor deptrac — 0 violations; castor phpstan — 0 errors; castor cs-check — clean; git diff --check HEAD^..HEAD — clean; castor check intentionally not run during task-start phase
- Summary: Implementation committed as ae84e7903. Reduced scope delivered: SessionRunStore now uses Symfony Filesystem::dumpFile() for atomic old-or-complete unlocked-reader visibility; dead custom EventSubscriberInterface and inert BoundaryHookEvent/BoundaryHookName Serializer/EventDispatcher bridge removed; typed HookSubscriberInterface aggregation, registration order, RunCommit ordering/failure isolation, and public ExtensionApi hooks retained. Added one bounded concurrent regression test proven to fail against the old in-place write. Explicitly skipped all other original candidates. Net change: 14 files, 154 insertions, 392 deletions. Worktree clean; not pushed; no PR.

## Task workflow update - 2026-08-15T23:25:50.592Z
- Validation: Final HEAD: 6b0e5ce77; Reviewer: APPROVED (no actionable findings); castor test — OK (4471 tests, 17513 assertions; 32.6s); castor deptrac — 0 violations, 0 errors; castor phpstan — 0 errors, 0 file errors; castor cs-check — clean (0 files fixed); JetBrains diagnostics — 0 errors in SessionRunStore.php, HookDispatcher.php, and SessionRunStoreTest.php; Worktree clean before CODE-REVIEW transition
- Summary: Final reviewer verdict APPROVED on HEAD 6b0e5ce77 after two review iterations. Reviewer confirmed exact reduced-scope fidelity, atomicity/permissions/CAS correctness, complete dead-reference removal, intact typed hooks/RunCommit failure isolation/public ExtensionApi, and lean net deletion. Review fixes: 790516ed2 requires at least one successful concurrent read and removes redundant test mkdir; 6b0e5ce77 deletes an implementation-mirroring registration-order test that only tested PHP foreach semantics.
- Review iteration 1: APPROVE WITH SUGGESTIONS; fork commit 790516ed2 strengthened concurrency proof (`OK reads` must be positive) and removed redundant mkdir.
- Review iteration 2: REQUEST CHANGES; fork commit 6b0e5ce77 removed registration-order test that violated tests/AGENTS.md by testing PHP array/foreach insertion order.
- Final re-review of full origin/main...HEAD: APPROVED. Final diff: 14 files, 126 insertions, 402 deletions; no TUI/public API/other broad candidate changes.

## Task workflow update - 2026-08-15T23:28:08.823Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (124.8s).
- Pushed task/reuse-symfony-components-outside-tui to origin.
- branch 'task/reuse-symfony-components-outside-tui' set up to track 'origin/task/reuse-symfony-components-outside-tui'.
- Created PR: https://github.com/ineersa/agent-core/pull/389

## Task workflow update - 2026-08-15T23:28:13.479Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/389
- Updated PR Status: open
- Validation: move_task deterministic castor check — passed (124.8s); PR: https://github.com/ineersa/agent-core/pull/389
- Summary: Moved to CODE-REVIEW after final reviewer approval and full focused validation. Deterministic transition gate passed; branch pushed and PR #389 created.

## Task workflow update - 2026-08-15T23:37:55.975Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/reuse-symfony-components-outside-tui.
- Merged task/reuse-symfony-components-outside-tui into integration checkout.
- Merge made by the 'ort' strategy.
 src/AgentCore/Application/AGENTS.md                |   2 +-
 .../Application/Handler/HookDispatcher.php         |  59 +------
 .../Extension/EventSubscriberInterface.php         |  19 ---
 src/AgentCore/Domain/Event/AGENTS.md               |   2 +-
 src/AgentCore/Domain/Event/BoundaryHookEvent.php   |  19 ---
 src/AgentCore/Domain/Event/BoundaryHookName.php    |  10 --
 .../Extension/AfterTurnCommitEventSummary.php      |   4 +-
 src/CodingAgent/Session/SessionRunStore.php        |  14 +-
 .../AfterTurnCommitSerializerRegressionTest.php    | 171 ---------------------
 .../Handler/HookDispatcherContractTest.php         |  69 +--------
 .../RunCommitAfterTurnCommitPersistedSeqTest.php   |  24 +--
 tests/CodingAgent/Session/AggregateResumeTest.php  |   5 +-
 tests/CodingAgent/Session/SessionRunStoreTest.php  | 102 ++++++++++++
 .../ToolBatchSnapshotCleanupHookSubscriberTest.php |  28 +---
 14 files changed, 126 insertions(+), 402 deletions(-)
 delete mode 100644 src/AgentCore/Contract/Extension/EventSubscriberInterface.php
 delete mode 100644 src/AgentCore/Domain/Event/BoundaryHookEvent.php
 delete mode 100644 src/AgentCore/Domain/Event/BoundaryHookName.php
 delete mode 100644 tests/AgentCore/Application/Handler/AfterTurnCommitSerializerRegressionTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/reuse-symfony-components-outside-tui.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/reuse-symfony-components-outside-tui.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #389 state — MERGED; Integration checkout clean before DONE transition
- Summary: PR #389 confirmed merged on GitHub at 2026-08-15T23:37:27Z (merge commit 0b4fbe0233e7fb2d007d82523eb19a00e686a77b). Moving task to DONE and syncing integration checkout.

## Task workflow update - 2026-08-15T23:44:28.739Z
- Updated PR Status: merged
- Validation: Post-merge `LLM_MODE=true castor check` final rerun — PASS (quality: ok); Final unit lane — 4471 tests, 17513 assertions; Controller replay — 12 tests, 165 assertions; TUI replay — 37 tests, 296 assertions; LLM-real — 13 tests, 144 assertions; Deptrac, PHPStan, CS, docs validation, artifact integrity, leak check, and llama-proxy cache guard — PASS; Initial post-merge check had one isolated TuiResumeSessionSwitchE2eTest timing failure; worker diagnostics found no stale candidates, focused `castor test:tui --filter=TuiResumeSessionSwitchE2eTest` passed (2 tests, 9 assertions), and full check rerun passed; Integration working tree clean; task worktree removed
- Summary: Task completed from merged PR #389. Worktree and IDEA exclusions removed; integration checkout synced and clean. Net implementation deletes 276 lines while adding atomic session-state persistence.

## Task workflow update - 2026-08-18T00:06:36.241Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
