# Fix missing subagent handoff and final telemetry in TUI

## Goal
Observed during manual testing of `2026-08-24-fix-critical-runtime-event-performance-and-delivery` after merging `origin/main`: a scout completed and its handoff/result reached the parent model (the model could consume it), but the final handoff was not visible in the TUI transcript/live view.

Correction from manual reproduction: while a `fork` or `scout` is live, the parent transcript widget already shows its running summary and detailed telemetry. The defect is in the final historical result view, which drops the latest tool/token/duration summary, LLM usage/model/reasoning line, and context-window line. Preserve those live values when the widget transitions to its final result through the same canonical child metadata/runtime projection path rather than adding final-widget-only model or usage state.

Treat this as a separate small investigation/fix; do not fold it into the runtime-performance task without evidence that the performance changes caused it. Reproduce on the process transport and trace the completed scout result from child terminal event/artifact through parent tool result, runtime protocol mapping, subagent live catalog, transcript projection, and mounted TUI block. Check whether the issue is specific to entering/leaving child live view, terminal progress checkpointing, or replay versus live projection.

Keep the fix minimal and reuse the existing canonical terminal/tool-result projection path. Do not add another handoff event, transport, or presentation abstraction. Coordinate with the completed `2026-08-20-add-agent-resume-for-completed-subagent-continuity` work and the active runtime-performance task.

## Acceptance criteria
- A completed scout/fork handoff that reaches the parent model is also visible once in the TUI through the existing result/projection path.
- The final historical result widget for both `scout` and `fork` retains the latest live summary (`tools`, aggregate tokens, duration), detailed LLM statistics (step count, input/output/reasoning tokens, cost, effective model and reasoning level), and context-window usage.
- The existing live widget presentation remains unchanged.
- Live process mode and resume/replay produce the same final visible handoff and retained telemetry without duplicates.
- Root cause is identified with deterministic regression coverage at the lowest correct layer; runtime/TUI changes pass `castor check`.
- No new event type, storage field, setting, transport, or duplicate terminal-status/projection logic is introduced.

## Workflow metadata
Status: CANCELLED
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-08-24T18:44:10.822Z

## Task workflow update - 2026-08-31T15:40:53.103Z
- Summary: Resolved by PR #446 (`eliminate-hot-path-child-run-metadata-filesystem-io`). Manual live-process verification now shows completed parallel scout handoffs once, plus LLM steps, input/output/reasoning tokens, cost, model/reasoning, recent tools, and context-window usage. The only desired presentation follow-up—compact final tool count/token count/elapsed time—was moved to `2026-08-27-show-live-tool-call-duration-in-tui`.
- Root cause: terminal `subagent_progress` is a canonical `tool_execution.output_delta` with seq > 0. `ConsumerStdoutPoller` incorrectly passed it through `RuntimeEventPerRunCompactBuffer`, which intentionally discards replayable seq>0 non-checkpoint coalescable events. The TUI therefore retained the preceding `running` snapshot.
- Why the handoff disappeared: the later `tool_execution.completed` result containing the handoff still arrived, but `SubagentResultRenderer::resolveHandoffMarkdown()` renders handoff Markdown only when the structured progress card is terminal. Because terminal progress had been dropped, the card remained `running` and hid the existing result.
- Fix in PR #446: `ConsumerStdoutPoller` now flushes transient buffered events and forwards canonical seq>0 runtime events directly. The terminal progress snapshot reaches `ToolProjectionSubscriber`/`TuiRuntimeEventApplier`, changes the card to `completed`, preserves final telemetry, and allows the already-delivered handoff result to render through the existing projection path.
- Proof: `ConsumerStdoutPollerTest::testForwardsCanonicalToolProgressInsteadOfDroppingItAsTransientBacklog` covers running seq=0 followed by completed seq=53; existing projection and resume TUI tests cover terminal handoff, model/reasoning, context, and no stale progress spam. User manually verified the live parallel-scout output.
- No new event type, storage field, transport, or final-widget-only state was introduced.

## Task workflow update - 2026-08-31T15:41:03.279Z
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Validation: Manual live parallel-scout output verified handoff, LLM statistics, model/reasoning, recent tools, and context usage; PR #446 exact-HEAD castor check passed; Deterministic consumer forwarding, terminal projection, and resume TUI coverage recorded in task work log
- Summary: Closed as resolved/absorbed by PR #446. The missing handoff and final model/token/context telemetry were consequences of the dropped canonical terminal progress event and are fixed by direct seq>0 forwarding. Remaining compact final duration presentation moved to TODO `2026-08-27-show-live-tool-call-duration-in-tui`.
