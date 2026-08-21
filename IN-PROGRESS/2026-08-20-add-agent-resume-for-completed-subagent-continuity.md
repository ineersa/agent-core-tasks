# Add agent_resume for completed subagent continuity

## Goal
Design locked after grill-me on token-efficiency / review-iterate waste, then explain lock-ins.

## Problem
Cold-start re-reviews and re-scouts redo Phase-1 context gathering. Parent often has the artifact id + a short delta task, but today’s `subagent` always launches a fresh child. Live-view already can `follow_up` a selected child; orchestrators need the same continuity via a tool.

## Product surface (v1)
- New parent-orchestrator tool: **`agent_resume`**
- Not an extension of `subagent` launch params
- Humans keep using `/agents-live` follow_up unchanged
- Fork is out of scope for v1 (separate later)

## Locked decisions (post-explain)

### Entry / API
- Dedicated tool: `agent_resume` (not an extension of `subagent` launch params)
- Single: `{artifact_id, task}` (optional `agent_run_id` like retrieve; artifact_id primary)
- Parallel: `{tasks:[{artifact_id, task}, ...]}` same shape as `subagent`, capped by `agents.max_agents`
- Model does not emit parallel tool-calls; parallelism is via `tasks[]` on the tool

### Lifetime / scope
- Current parent session only; must **not** survive parent `/resume`
- Parent compaction is irrelevant: resume uses child run_id/artifact from current parent registry

### Continuity / transport
- True continuity: `follow_up` into the **existing child session/run**
- Child continues on messenger transport; parent tool uses deferred completion (reuse/extend deferred subagent batch machinery)
- Do not rebuild/relaunch a new child with seeded handoff in v1

### Blocking / concurrency
- Block until finished (same foreground deferred semantics as today’s `subagent`)
- In-flight: reject if artifact status is `running` or `needs_clarification`; allow if terminal
- Reject same-artifact concurrent resume; allow different artifacts concurrently
- Parent steering of running subagents is a separate future task

### Eligibility
- All non-fork subagent roles (scout/reviewer/researcher/architect/browser/etc.)
- Fork out of scope for v1
- Eligible statuses: completed + failed + cancelled when child run/session is still usable
- Failed/cancelled: **try resume**; refuse with clear error only if unusable
- Context-size guardrail: refuse resume when `latestInputTokens >= max(0.75 * contextWindow, 200_000)`; if `contextWindow` missing, use absolute `200_000`
- No frontmatter `resumeAllowed` gate in v1

### Artifact identity / handoffs / parent result
- Keep the **same artifact id**
- Parent-visible result **mirrors `subagent`**: single = full **latest** handoff inline; parallel = bounded summaries
- Handoff storage: keep `handoff.md` as latest; on each finalize, archive prior content to `handoffs/<n>.md` and update `handoffs/index.json`
- `agent_retrieve`: `mode=handoff` remains latest only; add `mode=handoff_history` (list by default; pass index/n to fetch one older body)
- Compact-always parent result was rejected (parent won’t reliably retrieve)

### Non-goals (v1)
- No parent `/resume` survival
- No fork resume
- No changing human live-view UX
- No parent-steer-of-running-children API
- No relaxing “main agent never edits files” rule (revisit after resume ships)
- No requirement that resume rewrite reviewer prompts; workflow/prompt delta-verify rules stay in harden-task-workflow task

## Suggested acceptance focus
- Parent can `agent_resume` a just-finished reviewer with a short “verify these fixes / review delta only” task
- Child retains prior transcript/tool history
- Same artifact id updated; parent gets full latest handoff on single resume (mirrors subagent)
- Prior handoffs archived and retrievable via `handoff_history`
- Rejects in-flight same-artifact resume; allows different artifacts concurrently via `tasks[]`
- Works for completed/failed/cancelled terminal children in current parent session only (try-resume)
- Live-view follow_up behavior unchanged
- Docs/skills updated for launch-vs-continue usage

## Acceptance criteria
- New parent tool `agent_resume` exists (single + `tasks[]` parallel) and is orchestrator-facing
- Resume sends `follow_up` to the existing child run_id and waits via deferred/messenger batch completion until terminal again
- Resume is limited to the current parent session artifact registry and does not survive parent `/resume`
- All non-fork subagent roles can be resumed; fork remains out of scope
- Terminal children in completed, failed, or cancelled status are eligible when the child run/session is still usable; unusable sessions refuse clearly
- Same artifact id is preserved across resumes
- Parent-visible resume result mirrors `subagent` (single full latest handoff; parallel bounded summaries)
- On finalize, previous `handoff.md` is archived under `handoffs/<n>.md` with `handoffs/index.json`; `handoff.md` remains latest
- `agent_retrieve` keeps `mode=handoff` as latest; adds `mode=handoff_history` list + index/n body fetch
- In-flight same-artifact resume (`running` / `needs_clarification`) is rejected; different artifacts may resume concurrently
- Context-size guardrail refuses oversized children per locked formula
- Human `/agents-live` follow_up path remains unchanged in v1
- Docs/skills describe when to use `agent_resume` vs launching a fresh `subagent`

## Workflow metadata
Status: IN-PROGRESS
Branch: task/2026-08-20-add-agent-resume-for-completed-subagent-continuity
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity
Fork run:
PR URL:
PR Status:
Started: 2026-08-20T20:13:01+00:00
Completed:

## Work log
- Created: 2026-08-20T16:57:22+00:00

## Task workflow update - 2026-08-20T17:28:45+00:00
- Summary: Clarified after Claude Code scout + user feedback: dedicated agent_resume (not on subagent launch); parallel via tasks[] like subagent (not model multi-tool-calls); resumed children continue on messenger/deferred transport; no frontmatter gate in v1; add context-size resume guardrails; borrow Claude Code launch-vs-continue prompting.
- Clarification: parent meant messenger/deferred transport semantics (like subagent), not putting resume onto the subagent launch tool.
- API remains dedicated agent_resume (Option A). Parallelism: support tasks[] the same way as subagent because the model does not emit parallel tool calls; cap with agents.max_agents.
- Implementation: follow_up existing child run_id; child continues on its messenger transport; parent tool uses deferred completion and waits for terminal outcome (same foreground deferred pattern as subagent).
- Frontmatter resumeAllowed deferred; all non-fork subagents resumable in v1.
- New guardrail candidate: refuse resume when child context is near limit (e.g. ~75% of window or absolute ~200k+ tokens) — exact threshold TBD during implement/explain; purpose is to stop parent abusing one fat child forever.
- Prompting inspiration from Claude Code Agent/SendMessage: launch tool stays fresh-only; continue tool preserves full context; tell model when to continue vs spawn fresh; for Hatfield, encode that as subagent vs agent_resume docs/skill text.
- Result shape still compact delta to parent by default; full via agent_retrieve.

## Task workflow update - 2026-08-20T18:30:00+00:00
- Summary: Explain locked: dedicated agent_resume with tasks[] parallel; follow_up + deferred batch; parent results mirror subagent (single=full latest handoff); try failed/cancelled; in-flight reject on running/needs_clarification; context refuse at max(75% window, 200k); handoff.md latest + handoffs/<n>.md archive + index.json; retrieve mode=handoff latest, mode=handoff_history list/fetch by index.
- Explain complete with user lock-ins.
- Parent resume result mirrors subagent: single full latest handoff inline; parallel bounded summaries. Compact-always rejected.
- Handoff history: keep handoff.md as latest; archive prior to handoffs/<n>.md + handoffs/index.json.
- agent_retrieve: mode=handoff stays latest; new mode=handoff_history lists by default and fetches one body by index/n.
- In-flight: reject running/needs_clarification; allow terminal.
- Failed/cancelled: try resume; refuse only if session/run unusable.
- Context guardrail: refuse at max(75% contextWindow, 200k); missing window -> absolute 200k.
- Deferred wait: reuse/extend deferred subagent batch machinery.
- No frontmatter resumeAllowed; no parent-steer; no fork resume; live-view unchanged.

## Task workflow update - 2026-08-20T20:13:01+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-20-add-agent-resume-for-completed-subagent-continuity.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-20-add-agent-resume-for-completed-subagent-continuity.
