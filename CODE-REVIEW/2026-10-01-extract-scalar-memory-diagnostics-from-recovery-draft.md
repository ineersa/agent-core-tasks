# Extract scalar memory diagnostics from recovery draft

## Goal
Implementation specification v1.0 Change1, /home/ineersa/projects/agent-core-tasks/reports/implementation-specification.md. Independently extract memory logging from draft932ce8500 onto main, without shared projections/replay/state/worker-recovery or Markdown changes. Process sampler identity, live/allocated/peak/limit, retained transcript counts/bytes at resume/compaction/boundary/idle/shutdown. Existing INFO logging configuration only, no contents or per-delta spam. Original draft has known weak reflection shutdown test: do not copy it. Keep logged phase labels honest and parent vs child view counts correctly labeled.

## Acceptance criteria
- No shared RunState/history/transcript cache dependency or delivery semantics change.
- Retained counts are computed from already-resident data; no archive scans or new retained graphs.
- Preserve existing pid correlation while adding sampler_pid for actual process heap.
- Deterministic lifecycle logging tests exercise actual wiring; no constructor bypass or reflection-only false proof.
- Castor focused/static/architecture checks pass; document measured versus unproven memory properties.

## Workflow metadata
Status: CODE-REVIEW
Branch: task/2026-10-01-extract-scalar-memory-diagnostics-from-recovery-draft
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-scalar-memory-diagnostics-from-recovery-draft
Fork run: agent_a1f1ce95828fbd9a
PR URL: https://github.com/ineersa/agent-core/pull/544
PR Status: open
Started: 2026-10-01T21:43:54+00:00
Completed:

## Work log
- Created: 2026-10-01T21:43:40+00:00

## Task workflow update - 2026-10-01T21:43:54+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-10-01-extract-scalar-memory-diagnostics-from-recovery-draft.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-scalar-memory-diagnostics-from-recovery-draft.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-scalar-memory-diagnostics-from-recovery-draft.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-scalar-memory-diagnostics-from-recovery-draft.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-scalar-memory-diagnostics-from-recovery-draft.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-scalar-memory-diagnostics-from-recovery-draft/.idea.
- Summary: Independent specificationChange1 observability slice on main4f107b034. Separate writer/worktree from streaming reader slice. Integration order Markdown→streaming→memory diagnostics; no concurrent shared directory/index writes.

## Task workflow update - 2026-10-01T21:44:08+00:00
- Ownership: owner=fork; fork_run=pending; revision=4f107b034; scope=independent scalar process/retained-memory lifecycle diagnostics with real wiring proof; outcome=assigned; commit=none

## Task workflow update - 2026-10-01T22:00:40+00:00
- Recorded fork run: agent_a1f1ce95828fbd9a
- Summary: Fork extracted eleven-path scalar diagnostics draft and reports focused60tests256assertions plus actual packaged shutdown1test9assertions, static/architecture/style/docs pass. Parent review NOT accepted yet: LogContextProcessor changes baseline pid precedence. Main4f107b034 injects actual pid BEFORE ambient merge; fork moves fallback after ambient and claims historical semantics. Restore baseline precedence and keep additive sampler_pid unless separately authorized. Test currently freezes incorrect claimed old behavior. No integration/commit/push; no shared cache/state changes.
- Ownership: owner=fork; fork_run=agent_a1f1ce95828fbd9a; revision=4f107b034+memory-diagnostics-draft; scope=independent scalar lifecycle memory logs; outcome=blocked; commit=none
- Next eligible continuation: same fork at /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-scalar-memory-diagnostics-from-recovery-draft. Inspect main baseline actual pid injection, preserve it, adjust regression to distinguish explicit call-site pid from ambient fallback. Recheck ordinary payload-free logs and true shutdown/one-shot idle proof.

## Task workflow update - 2026-10-01T22:41:30+00:00
- Ownership: owner=fork; fork_run=agent_a1f1ce95828fbd9a; revision=4f107b034+existing-draft; scope=preserve baseline PID precedence; outcome=assigned; commit=none

## Task workflow update - 2026-10-01T22:47:39+00:00
- Validation: Parent castor test --filter=(LogContextProcessorTest|ProcessMemorySnapshotLoggerTest|CompactionProjectionSubscriberTest|TickPollListenerTest): PASS61tests260assertions; git diff --check: PASS
- Summary: Parent reviewed corrected baseline PID precedence and scalar-only logger. Parent focused validation PASS61tests260assertions. Fork packaged Ctrl+D proof PASS1test9assertions. Lifecycle metrics extracted without cache/state/recovery changes. Implementation slice ready for task-to-pr independent review/full gate, not merged; no user-ready PHAR claim.
- Ownership: owner=fork; fork_run=agent_a1f1ce95828fbd9a; revision=4f107b034+corrected-memory-diagnostics-draft; scope=additive memory diagnostics preserving baseline PID precedence; outcome=completed; commit=none

## Task workflow update - 2026-10-01T23:52:32+00:00
- Summary: Independent reviewer=agent_f31ff5f2df7b696b; target=4f107b034+dirty; scope=entire independent extraction; specification-fidelity verdict=REQUEST CHANGES: unreachable retention-floor trigger; shutdown logging must not prevent teardown.

## Task workflow update - 2026-10-01T23:57:59+00:00
- Summary: Independent review approved by agent_f31ff5f2df7b696b; reviewed tree committed at 774b0fe89. Specification fidelity confirmed for independent Change1 scope; no unresolved blocking review findings. Ready for transition-owned full QA gate.

## Task workflow update - 2026-10-02T00:04:20+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (102.4s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-scalar-memory-diagnostics-from-recovery-draft/var/reports/qa-20261002-000237-20757-9d30124e.
- Session/run: 75.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-02T00:04:21+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-10-01-extract-scalar-memory-diagnostics-from-recovery-draft to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-01-extract-scalar-memory-diagnostics-from-recovery-draft/var/reports/qa-20261002-000237-20757-9d30124e.
- Session/run: 75.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-02T00:04:24+00:00
- castor check passed (102.4s).
- Pushed task/2026-10-01-extract-scalar-memory-diagnostics-from-recovery-draft to origin.
- Created PR: <url>
- Session/run: 75.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-02T00:04:24+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (102.4s).
- Pushed task/2026-10-01-extract-scalar-memory-diagnostics-from-recovery-draft to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/544
