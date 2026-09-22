# Discover a launch-time background option for subagents and forks

## Goal
## Goal
Investigate the smallest design that lets main choose foreground or background execution when launching a subagent or fork, so main can continue independent work while a child runs.

## Agreed direction
Main chooses through a launch-time option. Reuse the existing bash background completion principle: launch returns an identity, and completion arrives as a follow-up notification. Existing agent retrieval metadata can support inspection and recovery, but routine status polling must not be the operating model.

User-triggered conversion of an already-running child to background is excluded from v1. Child-to-parent messaging, SendMessage-style tools, message boards, scheduling, and cross-session resume are deferred.

## Discovery questions
Trace existing foreground child execution, deferred results, retrieval metadata, cancellation, and bash completion delivery. Determine which facilities can be shared without creating another execution system.

Specify completion delivery when the parent is busy or idle, correlation and result bounds, error delivery, and interruption/recovery behavior. Preserve child-output provenance even if the delivery mechanism uses a user-follow-up envelope. Examine exactly what an active session means for child lifetime, parent exit, and shutdown.

Account for human input and approvals while a child runs in the background. Keep write ownership explicit: background execution must not permit concurrent writers to share a checkout. Foreground versus background should ideally change waiting behavior rather than create different child semantics.

## Deliverable
A cited discovery report with recommended minimal interface, lifecycle behavior, alternatives, unresolved product decisions, implementation slices, and deterministic validation plan. Discovery only; no production feature implementation or new public API without approval.

## Related work
`2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable`: reliable execution status, actionable errors, and result recovery are prerequisites to assess, not duplicate here.

## Acceptance criteria
- Map current subagent/fork and bash background execution paths with source references; identify reusable notification, status, artifact, and cancellation facilities.
- Recommend a launch-time foreground/background contract controlled by main, including completion notification without polling and bounded inspectable results.
- Specify parent busy/idle behavior, child failures, human approvals/input, cancellation, session shutdown, and result-delivery recovery; flag unresolved decisions instead of inventing behavior.
- Explain writer isolation and how completion notifications retain child provenance rather than masquerading as human instructions.
- Provide a bounded implementation and validation plan, including dependencies on reliable execution reporting. Keep user-triggered mid-flight backgrounding, messaging, scheduling, and cross-session operation out of v1.

## Workflow metadata
Status: IN-PROGRESS
Branch: task/2026-09-08-discover-a-launch-time-background-option-for-subagents-and-forks
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-08-discover-a-launch-time-background-option-for-subagents-and-forks
Fork run:
PR URL:
PR Status:
Started: 2026-09-08T22:06:30+00:00
Completed:

## Work log
- Created: 2026-09-08T16:32:35+00:00

## Task workflow update - 2026-09-08T22:06:30+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-08-discover-a-launch-time-background-option-for-subagents-and-forks.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-08-discover-a-launch-time-background-option-for-subagents-and-forks.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-08-discover-a-launch-time-background-option-for-subagents-and-forks.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-08-discover-a-launch-time-background-option-for-subagents-and-forks.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-09-08-discover-a-launch-time-background-option-for-subagents-and-forks.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-08-discover-a-launch-time-background-option-for-subagents-and-forks.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-08-discover-a-launch-time-background-option-for-subagents-and-forks/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-08-discover-a-launch-time-background-option-for-subagents-and-forks.

## Task workflow update - 2026-09-08T22:09:54+00:00
- Ownership: owner=main; fork_run=none; revision=0175446ec; scope=cited discovery report only, no feature or API changes; outcome=assigned; commit=none
- Scout agent_169b09e38c0504ae examined deferred child lifecycle and retrieval at baseline 0175446ec, read-only. Main owns bash follow-up trace, source verification, recommendations, and report.

## Task workflow update - 2026-09-08T22:19:35+00:00
- Validation: castor docs:validate passed for the existing 20-document built-in catalog. The discovery report is intentionally not a built-in document, so this is not a report content check.; Main verified source evidence and reviewed the report against acceptance criteria. git diff --cached --check passed; committed worktree clean.; Testing skill and tests/AGENTS.md read by main and scout. No tests or live probes run for this source-cited discovery. Full castor check and independent review deferred to task-to-pr.
- Summary: Discovery report committed as 1fab3db2f in docs/background-agents-discovery.md. No production feature, tool argument, setting, or test changes. Recommends a default-foreground whole-call background option backed by existing durable batches and command mailbox. Identifies deferred-registration coupling, bash send/mark duplicate window, cancellation wake behavior, controller shutdown ordering, child provenance, and shared checkout risks. Product decisions remain explicit in the report; no approval needed to finish discovery. Next phase is task-to-pr.
- Ownership: owner=main; fork_run=none; revision=0175446ec; scope=cited discovery report only, no feature or API changes; outcome=completed; commit=1fab3db2f
- Scout agent_169b09e38c0504ae continuation verified CWD inheritance, cancellation and registration gates, and main-selected child HITL. Read-only handoff completed; main checked key source paths.
- Skill defect reported, not repaired: /home/ineersa/.hatfield/skills/subagents/SKILL.md says 'Fork children are not resumable: each new fork requires an explicit ownership handoff.' Current bundled source supports same-session terminal fork resume.

## Task workflow update - 2026-09-08T22:56:39+00:00
- Summary: User-requested relocation: report moved unchanged from docs/background-agents-discovery.md to architecture/background-agents-discovery.md in the task worktree. Rename is staged, not committed.

## Task workflow update - 2026-09-08T22:59:46+00:00
- Ownership: owner=main; fork_run=none; revision=1fab3db2f; scope=rewrite discovery as proposed architecture decision record with Mermaid flows and architecture index link; outcome=assigned; commit=none

## Task workflow update - 2026-09-08T23:05:35+00:00
- Validation: castor docs:validate passed for built-in documentation catalog. It does not validate this repository-only ADR or Mermaid rendering.; git diff --check and staged diff whitespace checks passed. Mermaid source inspected but not rendered; no Mermaid CLI installed.; No production behavior or tests changed. Full gate and independent review remain in task-to-pr.
- Summary: Rewrote architecture/background-agents-discovery.md as a proposed ADR matching the architecture collection: navigation links, current and proposed Mermaid flows, delivery and child state diagrams, shutdown, HITL, provenance, and checkout ownership. Added proposed-ADR entry to architecture/README.md. Commit 177514eb0; clean worktree. Loaded unslop, technical-writing, domain-modeling, and ADR format. Kept architecture location as explicitly requested rather than the skill's default docs/adr location. Product choices remain proposed, not approved.
- Ownership: owner=main; fork_run=none; revision=1fab3db2f; scope=rewrite discovery as proposed architecture decision record with Mermaid flows and architecture index link; outcome=completed; commit=177514eb0

## Task workflow update - 2026-09-09T00:38:39+00:00
- Validation: castor --castor-file=var/tmp/mermaid-validate/castor-mermaid.php mermaid:validate --file=architecture/background-agents-discovery.md PASS: 11 blocks, 0 API parse failures, 0 real-browser mmdc render failures.; SVGs, report.json and gallery.pdf under worktree var/tmp/mermaid-validate/out. Temporary npm dependencies and validator are untracked; no project dependencies changed.; git diff --check passed; committed worktree clean.
- Summary: Fixed actual Mermaid parsing error at architecture/background-agents-discovery.md:292. A semicolon in sequence note text was parsed as a statement separator. Replaced it with a comma; commit 1717609ec. All 11 diagrams now parse and render with Mermaid 11.17.2 via a temporary Castor validation task.
- Browser agent_3fa02893245137a7 reproduced 1 failing diagram out of 11, identified semicolon lexer behavior, and verified minimal correction on an untracked copy. Main applied correction and reran all parsing and rendering.
- Ownership: owner=main; fork_run=none; revision=177514eb0; scope=Mermaid parsing correction and full document diagram validation; outcome=completed; commit=1717609ec
