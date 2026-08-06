# Inspect agent --controller memory usage (2.7GB PSS observed)

## Goal
## Symptom

During investigation of the occupancy-guard task (2026-07-01-prevent-resume-into-an-occupied-session-dual-hatfield-guard), a live `agent --controller` process was measured at **2.78 GB RSS / 2.75 GB PSS** — a single PHP process:

```
PID 400650  hatfield.phar agent --controller --cwd=.../subagent-live-03-child-hitl-cancellation
            RSS: 2,787,860 KB   PSS: 2,749,012 KB   CPU: 47.1%
            etime: ~27min (observed still growing)
```

Context: this was a subagent-heavy HITL (human-in-the-loop) cancellation session in a sibling worktree. The companion TUI (`agent`, PID 396295) was 100MB; 8 `messenger:consume` workers were ~70MB each (~560MB total). So the controller alone is ~4x the rest of the session combined and is the dominant memory consumer.

PSS ≈ RSS here (processes barely share pages), so this is not double-counting shared libraries. It is real resident heap.

## Why this matters

- A controller is a long-lived orchestrator (subprocess spawning, message dispatch, event polling). 2.7GB suggests unbounded accumulation, not steady-state working set.
- On memory-constrained machines this causes OOM kills, which interact with the crash-safety story of session storage / locks.
- May correlate with CPU pressure observed during concurrent gate runs (though that's unproven — see the occupancy task).

## Investigation scope (diagnosis first, fix later)

1. **Reproduce & profile**: run `agent --controller` under a representative subagent + HITL workload, capture memory growth over time (PSS curve, not a single point). Use `memory_get_usage(true)` instrumentation, `smaps_rollup` sampling, or a heap dump if available.
2. **Identify the accumulator**: candidates to inspect for unbounded retention:
   - Event store / transcript projection in memory (`events.jsonl` replay, `TranscriptProjector`)
   - Messenger worker message buffers / enqueued but un-acked jobs
   - Controller-side subprocess management (`AgentChildRunStore`, child run bookkeeping)
   - Subagent artifact registry (`AgentArtifactRegistry`)
   - LLM streaming response accumulation
   - Any per-run or per-turn cache that grows with session length
3. **Distinguish leak vs. large working set**: does memory plateau or climb linearly with turns/subagents? Does it release on session end / GC, or only on process exit?
4. **Report findings** before implementing a fix — root cause may be a deliberate cache, a missing unset, a circular reference, or an actual leak. Fix strategy depends on the cause.

## Out of scope

- The messenger:consume workers (~70MB each) are normal — do not investigate those unless profiling shows they also grow.
- This is NOT related to the occupancy-guard branch's flock/lock work.

## Notes

- Observed 2026-07-03 ~20:28–20:55 UTC, PHP 8.5.5, hatfield.phar build from the subagent-live-03-child-hitl-cancellation worktree.
- The PID has since changed/cycled (was 3361, then 3337, then 400650 across observations) — this is the known persistent controller, not a one-off.

## Acceptance criteria
- Memory growth curve captured for a representative subagent + HITL controller run (PSS over time, not single point)
- Dominant accumulator(s) identified with code-level evidence (which class/service retains the memory)
- Classification: genuine leak (unbounded) vs. large-but-bounded cache vs. transient spike — with the turn/subagent count at which it stabilizes
- If a fix is warranted: targeted change with a regression test that fails before the fix under the same workload
- No broad refactors or speculative 'while we're here' changes — diagnosis drives scope

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
- Created: 2026-07-04T01:03:57.828Z

## Task workflow update - 2026-07-21T17:06:08.531Z
- Summary: Cancellation requested by user on 2026-07-21. The previously observed controller memory issue has already been addressed sufficiently for now, and the stale 2.7 GB PSS observation no longer justifies a separate diagnosis task unless the symptom recurs. Do not start this task; reopen or create a fresh evidence-based investigation only if memory growth is observed again.
- 2026-07-21: User cancelled this investigation as superseded by subsequent memory fixes. Kept in TODO only because the current task-workflow tool does not yet support a CANCELLED status; move it to CANCELLED once task-workflow-toon-archive-cancelled is implemented.
