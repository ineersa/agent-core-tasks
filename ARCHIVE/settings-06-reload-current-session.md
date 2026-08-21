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
Status: ARCHIVE
Branch: task/settings-06-reload-current-session
Worktree: /home/ineersa/projects/agent-core-worktrees/settings-06-reload-current-session
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/408
PR Status: merged
Started: 2026-08-18T15:36:15.790Z
Completed: 2026-08-18T18:12:00.452Z

## Work log
- Created: 2026-07-16T17:41:37.212Z

## Task workflow update - 2026-07-16T17:43:36.282Z
- Summary: CANCELLED by user during design review. Do not implement self-relaunch via `pcntl_exec`, wrapper relaunch, or replacement-process spawning. Recreating the complete Symfony container/TUI/runtime safely inside the existing process is not justified; settings changes should continue to report that the user must restart Hatfield manually.

## Task workflow update - 2026-07-16T17:53:52.775Z
- Summary: REOPENED with a revised authoritative design. The original `pcntl_exec`/replacement-process proposal and cancellation note are superseded. Implement `/reload` as resume-like session reconstruction across a fresh application bootstrap: stop the TUI, return a typed reload intent containing the current session ID, synchronously shut down the AgentSessionClient/controller/consumers and old Symfony Kernel/container, reset process-global TUI/Revolt lifecycle state as required, then create a fresh Kernel + Console Application inside an outer `bin/console` bootstrap loop and resume the same session. The PHP process, PID, terminal, and shell attachment stay unchanged. Do not use `pcntl_exec`, spawn a replacement Hatfield process, mutate the frozen container in place, or rely only on the existing same-container `/resume` loop. The fresh container must reread settings and recreate extensions/providers/tools/TUI services. Preserve the safety guard that reload must not discard active or transient work. Validate explicit controller/consumer teardown, no leaked event-loop callbacks/shutdown handlers, same-session transcript resume, and a visibly changed setting through the required virtual/process/tmux Castor lanes and full `castor check`.
- 2026-07-16 design correction: user confirmed `/reload` should mirror `/resume` conceptually while additionally rebuilding configuration/container and restarting controller/consumers. Keep task in TODO for later implementation.

## Task workflow update - 2026-07-17T18:06:32.632Z
- MANDATORY implementation constraints: REUSE SYMFONY Console/Kernel/container lifecycle components and existing TUI/controller/session-switch paths; do not invent custom lifecycle infrastructure where framework/project mechanisms suffice. DO NOT OVERENGINEER; choose the simplest viable in-process reload design and reject scope expansion. MINIMIZE CHANGES AND TESTS: no unrelated renames/refactors/churn; add only the smallest tests that prove kernel/container/controller teardown and same-session resume.

## Task workflow update - 2026-08-18T15:36:15.790Z
- Moved TODO → IN-PROGRESS.
- Created branch task/settings-06-reload-current-session.
- Created worktree /home/ineersa/projects/agent-core-worktrees/settings-06-reload-current-session.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/settings-06-reload-current-session.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/settings-06-reload-current-session.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/settings-06-reload-current-session.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/settings-06-reload-current-session.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/settings-06-reload-current-session/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/settings-06-reload-current-session.

## Task workflow update - 2026-08-18T16:54:46.750Z
- Validation: castor test --filter Reload* -- 42 tests, 136 assertions OK; castor test ×2 — 4621 tests, 18344 assertions OK; castor test:tui --filter=TuiReloadSettingsE2eTest — 1 test, 18 assertions OK (same pane PID, same session id, visible settings change cap 500→20000, controller lock acquired ×2 no conflict, graceful shutdowns); castor test:tui full — 40 tests, 335 assertions OK; castor deptrac — 0 violations; castor phpstan — 0 errors; castor cs-fix + cs-check — clean; castor clean:cleanup:workers:list — no stale workers; Full castor check NOT run (runs at CODE-REVIEW gate)
- Summary: Implementation complete via fork. Commit 4de592c8e on task/settings-06-reload-current-session (30 files, +1593/−48). Design: /reload → ReloadCommandHandler guards → TuiSessionSwitchService::requestReload → InteractiveMode post-loop branch (reload consumed before switch, SessionEnded endReason=Reload) → ProcessReloODState::set + AgentSessionClient::shutdown() (synchronous controller teardown) → exit 75 → outer bin/console loop (setAutoExit(false)) rebuilds Kernel+Application in-process → ReloadArgvBuilder argv → --resume=<id> → fresh settings. Two flagged deviations (both correctness-driven, pending user confirmation): (1) ReloadArgvBuilder captures argv AFTER the early --cwd block, not raw startup argv — raw would re-validate a relative --cwd against already-changed CWD and fail on second boot; (2) bin/console setAutoExit(false) + explicit exit($exitCode) — required because Application::run() default autoExit exits the process inside run(), killing the outer loop. E2E teardown proof uses agent.log session-owner-lock lines instead of /proc (bwrap namespace makes host processes invisible).

## Task workflow update - 2026-08-18T17:13:52.526Z
- Validation: Reviewer: REQUEST CHANGES — 1 NIT (dead -p code), 0 BLOCKERs; Fix commit a885e0758: castor test --filter=ReloadArgvBuilderTest — 4 tests OK; castor test --filter=ReloadCommandHandlerTest|SessionSwitchServiceTest — 35 tests OK; castor phpstan — 0 errors; castor cs-check — clean
- Summary: Reviewer verdict: REQUEST CHANGES with single NIT — speculative -p handling in ReloadArgvBuilder (AgentCommand --prompt has no shortcut; -p can never reach builder). All else approved: "smallest viable in-process re-bootstrap, guards exhaustive, ordering correct, E2E proves the visible-settings-change claim". Fix applied via fork as commit a885e0758: deleted -p clauses, dead option-looking guard for --prompt, matching test assertions; reachable behavior unchanged (--prompt= and --prompt <v> still stripped). Re-review skipped: reviewer pre-approved exactly this deletion. Head now a885e0758.

## Task workflow update - 2026-08-18T17:16:37.992Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (148.8s).
- Pushed task/settings-06-reload-current-session to origin.
- branch 'task/settings-06-reload-current-session' set up to track 'origin/task/settings-06-reload-current-session'.
- Created PR: https://github.com/ineersa/agent-core/pull/408

## Task workflow update - 2026-08-18T17:37:19+00:00
- Validation: Post-reload scout subagent launch completed on updated model (artifact agent_a9567b46ad5d320d, run 641361ba-9a40-5938-b126-e92f98bc34b7); Pre-reload scout run confirmed stale model persisted until reload (negative case)
- Summary: Live verification of /reload: changed scout agent model runpod/Qwen3.8-27B → llama_cpp/flash in ~/.hatfield/agents/scout.md, ran /reload, launched a scout ping-pong smoke test. Pre-reload run used the stale model; post-reload scout run (agent_a9567b46ad5d320d, completed) confirmed the updated llama_cpp/flash model was picked up by new subagent launches.

## Task workflow update - 2026-08-18T17:39:24.470Z
- Moved CODE-REVIEW → IN-PROGRESS.

## Task workflow update - 2026-08-18T17:41:35.961Z
- Validation: castor test:tui --filter=TuiReloadSettingsE2eTest — 1 test, 18 assertions OK; castor phpstan — 0 errors; castor cs-check — clean; castor test --filter=SessionSwitchServiceTest|ReloadCommandHandlerTest — 35 tests OK
- Summary: User live-tested /reload (agent model change scenario) — works. Requested follow-up: clear screen on reload like /resume. Commit c89bf4f84: extracted shared clearScreenForNextSession() (clearScreen + \x1b[3J) used by both reload and switch branches; reload branch clears after SessionEnded dispatch, before client shutdown — same cooked-mode condition as /resume. E2E needed no assertion changes (all markers are post-reload re-rendered content).

## Task workflow update - 2026-08-18T17:44:12.645Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (146.1s).
- Pushed task/settings-06-reload-current-session to origin.
- branch 'task/settings-06-reload-current-session' set up to track 'origin/task/settings-06-reload-current-session'.
- PR already exists: https://github.com/ineersa/agent-core/pull/408

## Task workflow update - 2026-08-18T18:12:00.452Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/settings-06-reload-current-session.
- Merged task/settings-06-reload-current-session into integration checkout.
- Merge made by the 'ort' strategy.
 bin/console                                        | 147 ++++---
 src/CodingAgent/CLI/ReloadArgvBuilder.php          |  73 ++++
 .../Runtime/Contract/AgentSessionClient.php        |   7 +
 .../Runtime/Contract/ProcessReloadIntentDTO.php    |  19 +
 .../Runtime/Contract/ProcessReloadState.php        |  44 +++
 .../InProcess/InProcessAgentSessionClient.php      |  11 +
 .../Process/JsonlProcessAgentSessionClient.php     |  13 +
 src/Tui/Application/InteractiveMode.php            |  91 +++--
 src/Tui/Application/TuiSessionSwitchService.php    |  31 ++
 src/Tui/Listener/ReloadCommandHandler.php          | 118 ++++++
 src/Tui/Listener/SessionCommandRegistrar.php       |  16 +-
 .../Contract/TuiSessionSwitchServiceInterface.php  |  10 +
 .../Runtime/TuiSessionLifecycleEndReasonEnum.php   |   3 +
 tests/CodingAgent/CLI/ReloadArgvBuilderTest.php    |  79 ++++
 .../BackgroundProcessCompletionPollerTest.php      |   7 +
 .../CommandHandler/AnswerHumanHandlerTest.php      |   7 +
 .../CommandHandler/CompactHandlerTest.php          |   7 +
 .../CommandHandler/ResumeHandlerTest.php           |   7 +
 .../Runtime/Controller/RuntimeEventEmitterTest.php |   7 +
 .../JsonlProcessAgentSessionClientShutdownTest.php | 139 +++++++
 tests/Tui/Application/SessionSwitchServiceTest.php |  63 +++
 tests/Tui/E2E/TuiReloadSettingsE2eTest.php         | 425 +++++++++++++++++++++
 tests/Tui/Listener/CompactCommandHandlerTest.php   |   7 +
 .../Tui/Listener/NewSessionCommandHandlerTest.php  |   4 +
 tests/Tui/Listener/ReloadCommandHandlerTest.php    | 326 ++++++++++++++++
 .../Listener/ResumeSessionCommandHandlerTest.php   |   4 +
 .../Listener/TickPollListenerSubagentLiveTest.php  |   7 +
 tests/Tui/Picker/SessionPickerControllerTest.php   |   4 +
 .../SubagentLivePickerObservationLifecycleTest.php |   7 +
 tests/Tui/Support/RecordingAgentSessionClient.php  |   7 +
 30 files changed, 1612 insertions(+), 78 deletions(-)
 create mode 100644 src/CodingAgent/CLI/ReloadArgvBuilder.php
 create mode 100644 src/CodingAgent/Runtime/Contract/ProcessReloadIntentDTO.php
 create mode 100644 src/CodingAgent/Runtime/Contract/ProcessReloadState.php
 create mode 100644 src/Tui/Listener/ReloadCommandHandler.php
 create mode 100644 tests/CodingAgent/CLI/ReloadArgvBuilderTest.php
 create mode 100644 tests/CodingAgent/Runtime/Process/JsonlProcessAgentSessionClientShutdownTest.php
 create mode 100644 tests/Tui/E2E/TuiReloadSettingsE2eTest.php
 create mode 100644 tests/Tui/Listener/ReloadCommandHandlerTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/settings-06-reload-current-session.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/settings-06-reload-current-session.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-08-18T18:12:04.372Z
- Summary: PR #408 merged to main. Merged into integration checkout (ort, 30 files +1612/−78), worktree removed, task complete. Live-verified by user: multiple sequential reloads via castor run:agent incl. agent-model-change scenario; clear-on-reload added post-review (c89bf4f84). Known untested corners accepted: PHAR-mode reload loop, InProcess transport explicit teardown (documented no-op).

## Task workflow update - 2026-08-19T18:17:25.197Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
