# Run same-turn tools concurrently by default, including forks and subagents

## Goal
The user reports that independent tools requested together in one parallel turn still execute sequentially: a `fork` completes before a scout `subagent` starts, and similar serialization occurs with `bash` plus child-agent tools. Investigate actual execution overlap and remove unintended serialization. The reported behavior is not yet independently verified.

## Required behavior

- All tool types are eligible for parallel execution by default, including `fork`, `subagent`, `agent_resume`, `bash`, `write`, and `edit`, plus extension and MCP tools.
- Independent calls submitted together must start without waiting for a sibling call to finish, subject to actual available capacity and existing explicit concurrency limits.
- Do not classify tools as exclusive merely because they launch children, block until completion, or mutate files. The user does not want blanket exclusions for any current tool type, including `write` and `edit`.
- Any intentional tool-level exclusion must be explicit and discoverable, not inferred from tool names or hidden special cases. Audit current exclusions and remove unnecessary ones. Bring any proposed retained exclusion to the user with concrete evidence before implementing it.
- Parallel eligibility does not make dependent operations independent. Preserve declared dependencies, approval boundaries, and existing same-file and worktree ownership rules without serializing unrelated calls across the whole turn. Do not introduce a new conflict-resolution or last-writer-wins contract without approval.

## Investigation and implementation

1. Reproduce `fork` plus scout `subagent`, `bash` plus `subagent`, and `bash` plus `fork` submitted as independent sibling calls. Separate native same-turn batches from explicit parallel-wrapper execution and child-tool batch execution. Distinguish intentionally sequential code inside `code_mode` from runtime serialization of parallel submissions.
2. Trace dispatch, admission, scheduling, queue consumption, worker availability, locks, child-run startup, and result waiting. Identify the exact boundary that serializes each affected combination. A blocking tool handler must not prevent independent siblings from being dispatched.
3. Audit parallel eligibility and exclusion metadata for built-in, extension, and MCP tools. Make eligibility default-on and update registration, settings, instructions, and docs where necessary through existing project facilities.
4. Fix the actual execution path. Removing an exclusion flag or displaying overlapping tool cards is insufficient if handlers or child runs still execute sequentially.
5. Preserve per-call result correlation, error reporting, cancellation, timeouts, session/run identity, child-agent limits, and resource ownership. One slow, failed, or approval-waiting call must not impose an unintended execution barrier on independent siblings. Required batch result gathering may still wait for the batch to finish.
6. Add deterministic overlap tests with synchronization barriers or equivalent execution evidence. Include child-agent combinations, ordinary tools, independent writes and edits to distinct files, and extension/MCP eligibility. Validate dependencies and explicit exclusions separately.
7. Measure actual admission, execution-start, child-start, and completion times. Verify worker activity and elapsed time under available capacity, rather than relying on UI timestamps alone. Record remaining capacity constraints and any approved exclusions.

## Related work

Inspect TODO task `2026-09-25-integrate-measured-messenger-concurrency-and-correct-tool-timing` and IN-PROGRESS task `2026-09-08-discover-a-launch-time-background-option-for-subagents-and-forks` before implementation. Coordinate scope and ownership; do not create a second competing scheduler or conflate background launch with concurrent execution.

This task records the request only. Implementation has not started.

## Acceptance criteria
- Under available capacity, independent `fork` plus scout `subagent`, `bash` plus `subagent`, and `bash` plus `fork` calls submitted together demonstrably overlap in actual execution.
- Parallel eligibility defaults to enabled for every tool type, including `write` and `edit`. No hidden or blanket child-agent or mutation-tool exclusion remains.
- Any retained tool-level exclusion is explicit, documented, covered by tests, and approved by the user. Capacity limits are distinguished from exclusions.
- Independent calls are dispatched without waiting for sibling completion. Mixed batches preserve result correlation, errors, approval boundaries, cancellation, timeouts, and existing child-agent limits.
- Independent writes and edits to distinct files overlap. Declared dependencies and existing same-file/worktree safety rules remain intact without global serialization.
- Deterministic regression tests and recorded execution/child-start timing prove overlap across the relevant submission paths. Visual overlap or changes to reported durations alone do not count as proof.
- Focused Castor validation passes in the implementation checkout, independent review is completed, and the CODE-REVIEW transition runs the full `castor check` gate.

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
- Created: 2026-10-12T20:50:59+00:00
