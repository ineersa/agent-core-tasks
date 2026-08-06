# SETTINGS-06: Add /reload full-process settings reload for the current session

## Goal
Add `/reload` as a true Hatfield process reload, not another iteration of the existing same-process `/resume` loop. `AppConfig` and many derived container services are fixed at kernel boot, so reusing `InteractiveMode` or merely restarting the controller would leave TUI/extensions/tools/providers partially stale.

Proposed lifecycle:
1. `/reload` records a typed process-reload intent for the current session and stops the TUI event loop.
2. `InteractiveMode` returns the reload intent to `AgentCommand` rather than starting another same-process session iteration.
3. After terminal restoration, the current `AgentSessionClient` is shut down synchronously so the controller and Messenger consumers have exited.
4. A relaunch service uses the existing `AppExecutableLocator` chain and `pcntl_exec` to replace the current process image, preserving stdio/PID/shell attachment and canonical CWD.
5. The new command boots a fresh Kernel/container and passes `--resume=<current-session-id>` when a persisted session exists; a draft reload starts a fresh draft.

Reconstruct relaunch arguments from normalized AgentCommand options rather than blindly replaying raw argv. Preserve persistent launch policy such as transport, model/reasoning CLI overrides, tool filters, skills, and prompt-template flags; replace cwd/resume and drop one-shot prompt input so it cannot execute twice.

The command must refuse reload while a run is active, waiting for human input, cancelling, or compacting, and when transient queued/paste state would be lost. No overlapping replacement process may be spawned.

Dependencies: useful after SETTINGS-03/05 but architecturally independent. SETTINGS-05 should direct users to `/reload` after successful settings mutations once this task lands.

## Acceptance criteria
- `/reload` is registered as a built-in local slash command and cannot be shadowed by prompt templates.
- Reload creates a typed process-reload result/intent carrying the current persisted session ID when available; it does not use the same-process `/resume` loop as the reload boundary.
- The TUI stops and restores the terminal before relaunch, then the session client/controller/consumer tree is shut down synchronously before process replacement.
- Relaunch uses `AppExecutableLocator` and `pcntl_exec` for both source and PHAR execution; it does not spawn an overlapping replacement process or require an external shell wrapper.
- The relaunched command uses canonical CWD, resumes the same persisted session, reloads a fresh Symfony Kernel/container and settings, and does not replay a one-shot prompt.
- Persistent CLI policy is preserved deliberately; obsolete resume/cwd values are replaced and unsafe/one-shot mode arguments are not copied blindly.
- Reload is rejected with an actionable local message during nonterminal activity or when transient queued/paste/question state would be discarded.
- Unsupported `pcntl_exec` or relaunch failure is reported clearly without pretending settings were reloaded.
- Focused contract coverage proves intent/argument construction and lifecycle guards. A minimal tmux/process integration proof changes a visible setting, runs `/reload`, and verifies the same session transcript resumes under the new setting using shared isolation conventions.
- Because this touches TUI/runtime/process lifecycle, focused Castor validation and the full `castor check` gate are required before review; leaked controller/consumer processes are treated as lifecycle bugs.

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
- Created: 2026-07-16T17:41:37.212Z

## Task workflow update - 2026-07-16T17:43:36.282Z
- Summary: CANCELLED by user during design review. Do not implement self-relaunch via `pcntl_exec`, wrapper relaunch, or replacement-process spawning. Recreating the complete Symfony container/TUI/runtime safely inside the existing process is not justified; settings changes should continue to report that the user must restart Hatfield manually.

## Task workflow update - 2026-07-16T17:53:52.775Z
- Summary: REOPENED with a revised authoritative design. The original `pcntl_exec`/replacement-process proposal and cancellation note are superseded. Implement `/reload` as resume-like session reconstruction across a fresh application bootstrap: stop the TUI, return a typed reload intent containing the current session ID, synchronously shut down the AgentSessionClient/controller/consumers and old Symfony Kernel/container, reset process-global TUI/Revolt lifecycle state as required, then create a fresh Kernel + Console Application inside an outer `bin/console` bootstrap loop and resume the same session. The PHP process, PID, terminal, and shell attachment stay unchanged. Do not use `pcntl_exec`, spawn a replacement Hatfield process, mutate the frozen container in place, or rely only on the existing same-container `/resume` loop. The fresh container must reread settings and recreate extensions/providers/tools/TUI services. Preserve the safety guard that reload must not discard active or transient work. Validate explicit controller/consumer teardown, no leaked event-loop callbacks/shutdown handlers, same-session transcript resume, and a visibly changed setting through the required virtual/process/tmux Castor lanes and full `castor check`.
- 2026-07-16 design correction: user confirmed `/reload` should mirror `/resume` conceptually while additionally rebuilding configuration/container and restarting controller/consumers. Keep task in TODO for later implementation.

## Task workflow update - 2026-07-17T18:06:32.632Z
- MANDATORY implementation constraints: REUSE SYMFONY Console/Kernel/container lifecycle components and existing TUI/controller/session-switch paths; do not invent custom lifecycle infrastructure where framework/project mechanisms suffice. DO NOT OVERENGINEER; choose the simplest viable in-process reload design and reject scope expansion. MINIMIZE CHANGES AND TESTS: no unrelated renames/refactors/churn; add only the smallest tests that prove kernel/container/controller teardown and same-session resume.
