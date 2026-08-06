# Eliminate multi-second agent startup and session resume latency

## Goal
Session 33 (about 1.8 MiB / 445 canonical events at inspection time) takes multiple seconds to start or resume. Read-only evidence found repeated full-history work in the resume path: `SessionInitializer::replayFromEvents()` loads the event file, `SessionTurnTreeProvider::forSession()` loads/projects it again, and `SessionTranscriptProvider::transcriptForLeaf()` loads it a third time before initial transcript rendering. Process-mode attach also waits for controller `runtime.ready`, so cold startup/controller boot must be timed separately rather than assuming all delay is event replay.

Profile the real cold-start and resume phases first, then remove redundant canonical reads/tree reconstruction/projection/render work. Preserve the canonical event store and branch semantics; do not introduce a second session-history format or stale cache as a shortcut.

Relevant areas:
- `src/Tui/Application/SessionInitializer.php`
- `src/CodingAgent/Session/SessionTranscriptProvider.php`
- `src/CodingAgent/Session/SessionTurnTreeProvider.php`
- `src/CodingAgent/Session/SessionRunEventStore.php`
- `src/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClient.php`
- initial `ChatScreen` / transcript render path

## Acceptance criteria
- Add privacy-safe phase timing evidence for cold startup and resume that separates controller boot/readiness, DB/session lookup, canonical event read/decode, branch projection, transcript projection, and first interactive render.
- Resume reads/decodes the canonical event stream once per initialization and shares the immutable snapshot through turn-tree and transcript projection, unless profiling proves a different dominant cause and the task records that evidence.
- Preserve active-leaf branch filtering, transcript ordering/content, `lastSeq`, usage replay, passive activity replay, and controller resume protocol semantics.
- Add a representative large-session benchmark or bounded performance regression based on an isolated synthetic fixture; it must detect repeated full-history parsing and demonstrate a material improvement without relying only on wall-clock sleeps.
- Validate cold new-session startup and resume separately through the real runtime/TUI path; the result must not merely optimize a unit fixture while controller-ready or first-render latency remains multi-second.
- Use structured startup diagnostics with `session_id`, `component`, and `event_type`; never log transcript content, prompts, tool output, or environment values.
- Run all QA through Castor, including the lowest-correct virtual/runtime proof and full `castor check` before CODE-REVIEW.

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
- Created: 2026-07-23T18:52:18.410Z

## Task workflow update - 2026-07-23T19:15:46.856Z
- Summary: Root cause confirmed for resume: branch-aware initialization reads, decodes, denormalizes, and sorts the same parent events.jsonl three times (`SessionInitializer`, `SessionTurnTreeProvider`, `SessionTranscriptProvider`), then repeats tree/filter/projection work before first paint. Session 33's 445 events therefore become 1,335 decodes plus three sorts. Fresh startup does not replay history; its remaining multi-second cost is controller process/kernel/consumer boot before `runtime.ready`, but that phase is currently uninstrumented. MCP discovery accounts for only ~0.4s, so startup needs phase timing before a more specific boot root cause can be claimed.
- Root-cause scout inspected only 8 bounded files. `InteractiveMode` keeps replay and controller attach/readiness sequential on the first-interactive-frame critical path. No existing log times SessionInitializer or runtime.ready wait.

## Task workflow update - 2026-07-23T19:57:05.228Z
- Summary: User impact clarified: the blank screen for seconds is itself unacceptable. Fix should not only reduce repeated replay/controller readiness time; it must paint a responsive startup shell immediately and surface explicit loading phases while history/controller attachment continues, without accepting stale interaction or losing canonical ordering.
- Task acceptance already requires phase timing and real TUI startup/resume validation; implementation should treat first paint separately from fully interactive readiness.

## Task workflow update - 2026-07-23T20:07:55.699Z
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Cancelled as over-scoped after user review. The proposed async first-paint/instrumentation work is not justified for the confirmed resume defect. Replaced by a minimal SessionRunEventStore memoization task.
