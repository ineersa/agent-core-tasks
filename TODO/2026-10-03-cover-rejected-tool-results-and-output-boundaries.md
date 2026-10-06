# Add regression coverage for rejected tool results and output boundaries

## Goal
Follow-up to PR #549, approved by the user after advisor review of eedd86f43. The user explicitly requested deleting obsolete scrubbing tests now and deferring replacement coverage because exercising malformed output in the active Hatfield session could break it. This task tracks the acknowledged coverage gap, not completed validation.

Do not start malformed-output reproductions or tests during the current session. Before future execution, confirm that the user's restriction has been lifted and use isolated test resources. Do not return raw malformed bytes through Hatfield tool output or read real session artifacts. All validation must use Castor and follow the testing skill and tests/AGENTS.md.

Removed obsolete coverage: BashToolTest::testMalformedOutputCanBePersistedWithoutRepeatingCommand with malformedOutputExitCodes success/failure rows; ProcessLifecycleTest::malformedLogBytesAreDisplaySafeWithoutChangingArtifact with invalidLogReadBudgets whole-log/tail rows. These asserted substitution and successful malformed output, which contradict the approved rejection policy. No replacement rejection coverage is claimed. User explicitly approved this temporary coverage loss.

Scope is the result contract and its execution/persistence/read boundaries. Session 74's original byte producer and restart-safe exactly-once execution remain outside this task; the latter belongs to SPEC-03.

## Acceptance criteria
- Malformed success output, exception text, nested values, and keys become safe non-retryable failed tool results; rejected bytes and unchecked copies are absent from persisted and model-visible results.
- Configured snapshot save/load preserves the same finalized failure. Unsupported objects and processor-produced invalid data cannot bypass the closed terminal representation.
- Through the real executor path, processor failure is contained and remembered. Same-process dedupe reruns neither external execution nor the failing processor chain.
- Extension-owned cancellation and timeout outcomes survive when cancellation becomes requested during the tool call. Use deterministic synchronization, not sleeps or timing windows.
- Valid Unicode survives bounded and live reads, including split two-, three-, and four-byte characters. Legal incomplete live suffixes are withheld; finalized incomplete or genuinely malformed output fails. Original artifacts remain unchanged.
- The deepest payload accepted at final admission round-trips through the configured snapshot wrappers; excessive nesting becomes a safe tool failure before collection. Cover the corresponding code-mode depth rule and strict IPC argument/error paths.
- Use the lowest correct test layer, deterministic isolation and teardown, and cases within the project duration limit. Record actual Castor validation; do not infer runtime correctness from source review.

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
- Created: 2026-10-04T02:42:52+00:00
