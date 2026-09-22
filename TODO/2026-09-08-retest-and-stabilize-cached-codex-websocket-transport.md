# Retest and stabilize cached Codex WebSocket transport

## Goal
Cached WebSocket transport previously caused enough issues that the user stopped using it. Retest the existing websocket_cached implementation and fix reproducible correctness or lifecycle failures so it is reliable for normal sessions. Do not assume the prior failures are understood or that fixture-only success proves live stability.

Start by gathering existing failure evidence and exercising the actual Codex transport. Compare with plain WebSocket as a control. Cover multi-turn conversations, tool loops, reasoning changes, cancellation and subsequent requests, resume, model changes, socket expiry/disconnects, and rejected continuation. Include Astra baseline/configuration_update behavior from PR #482, task 2026-09-07-preserve-astra-prompt-cache-on-reasoning-change.

Keep the existing transport opt-in. Do not change the default transport, add speculative retry/fallback behavior, or introduce new settings without approval. Here, durable means reliable across supported lifecycle and failure conditions, not persisting live sockets or continuation state across process restarts.

For each reproduced defect, identify its cause, make the smallest supported fix, and retain deterministic regression proof at the lowest correct layer. Use a bounded live provider smoke for contracts unavailable below. Cache-read metrics support investigation but do not by themselves prove correct continuation. Record wire-shape and response/connection lifecycle evidence without credentials or raw session content. Follow Castor-only QA, test ownership/teardown rules, and the task workflow. Investigate failures rather than retrying until green.

## Acceptance criteria
- Record a reproducible baseline of cached versus plain WebSocket behavior and identify which historical issues can still be reproduced.
- Demonstrate multi-turn and tool-loop correctness with no lost, duplicated, or reordered model-visible input or response events.
- Verify Astra reasoning changes preserve the initial baseline and eligible continuation; resume and incompatible model changes establish the required fresh state.
- Verify cancellation, disconnects, expiry, and rejected continuation follow supported recovery behavior without hangs, stale state reuse, leaked sockets, or leaked workers.
- Add deterministic regression tests for reproduced defects and run a bounded live Codex verification with documented results and remaining limitations.
- Keep cached transport opt-in and avoid unsupported persistence, compatibility paths, settings, or default changes.
- Required Castor validation passes, including relevant concurrent lanes when contention is a reproduced failure mode; independent review approves.

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
- Created: 2026-09-08T17:23:06+00:00
