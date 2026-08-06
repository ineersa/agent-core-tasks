# Implement TUI /export command for session export

## Goal
## Goal

Add a TUI slash command `/export` to agent-core, modeled after pi-mono's `/export` command.

The parent/orchestrator intentionally did not read source directly for this planning task; this task is based on scout reports only.

## Scout findings: pi-mono behavior to emulate

Pi-mono registers `/export` as a built-in slash command with description: `Export session (HTML default, or specify path: .html/.jsonl)`.

User-visible behavior:
- `/export` with no args exports the current session to standalone HTML.
- `/export <path>` exports to the requested path.
- If the path ends in `.jsonl`, pi-mono writes a JSONL session export; otherwise it writes HTML.
- Path parsing supports quoted paths with spaces, unquoted paths stopping at first whitespace, and no-arg default behavior.
- Success is surfaced as a TUI status message: `Session exported to: <path>`.
- Errors are caught and surfaced as friendly TUI errors, not uncaught exceptions.
- Default HTML filename in pi-mono is `pi-session-<session-file-basename>.html` in the current working directory.
- Default JSONL filename in pi-mono is `session-<timestamp>.jsonl`.

Important pi-mono implementation lessons:
- HTML export is standalone: session data is embedded into the HTML, with CSS/JS/template assets included.
- The HTML export must escape user/session/tool content. Pi-mono has explicit XSS coverage for markdown links/images, entry IDs, model names, tree labels, data attributes, and raw HTML passthrough.
- `.jsonl` export is machine-readable; pi-mono exports the current branch and re-chains `parentId` linearly.
- Pi-mono handles missing/empty/in-memory sessions with friendly errors.

## Scout findings: agent-core integration points

Agent-core slash commands should be added through the existing TUI command infrastructure:
- Register commands via a `TuiListenerRegistrar` implementation in `src/Tui/Listener/`.
- Handlers implement `SlashCommandHandler` and return a `CommandResult` such as `TranscriptMessage`.
- Do not add session-aware command logic under `src/Tui/Command/`; Deptrac keeps that layer dependency-free.
- Existing patterns to follow: `CopyCommandRegistrar`/`CopyCommandHandler` and `SessionCommandRegistrar` for `/new`, `/resume`, `/rename`.
- Registration should be idempotent: if command exists, call `setHandler`; otherwise `register(new CommandMetadata(...), $handler)`.
- `HatfieldSessionStore::resolveSessionsBasePath()` is public; scout reported session directory can be computed as `<sessionsBase>/<state->sessionId>`. Canonical events are in `.hatfield/sessions/<id>/events.jsonl`.
- `getSessionDir()` is private, so do not call it.

Suggested metadata:
```php
new CommandMetadata(
    name: 'export',
    aliases: ['exp'],
    description: 'Export the current session transcript to a file',
    usage: '/export [path]',
    acceptsArguments: true,
)
```

## Implementation outline

1. Add `src/Tui/Listener/ExportCommandRegistrar.php`.
   - Implement `TuiListenerRegistrar`.
   - Register `/export` and optional alias `/exp` using the idempotent registrar pattern.
   - Instantiate `ExportCommandHandler` with the current `TuiRuntimeContext` state and session-store/config dependencies needed to locate the current session.

2. Add `src/Tui/Listener/ExportCommandHandler.php`.
   - Implement `SlashCommandHandler`.
   - Parse `SlashCommand->args` with pi-mono-compatible rules:
     - empty args => default HTML output path;
     - quoted single/double string => full path including spaces;
     - unquoted => first token only;
     - malformed quotes => friendly `TranscriptMessage` error.
   - Resolve output paths consistently with project/session cwd conventions discovered during implementation; document the chosen default in code/tests.
   - If output path ends in `.jsonl`, export/copy the current session's canonical `events.jsonl` to that path.
   - Otherwise generate a standalone `.html` export of the current session.
   - Return a `TranscriptMessage` on success containing the written path.
   - Return friendly `TranscriptMessage` errors for no current session, missing/empty `events.jsonl`, invalid path, unsupported extension if any, and write failures.
   - Do not throw uncaught exceptions into the TUI submission path.

3. HTML export requirements.
   - First implementation may be simpler than pi-mono's full SPA, but must be a useful standalone browser-viewable transcript/session export.
   - Include enough session information for a user to inspect the conversation after the command runs.
   - Escape all untrusted session content before inserting into HTML.
   - Do not allow raw HTML passthrough from user/assistant/tool content.
   - If rendering markdown, sanitize URLs with an allowlist rather than trusting arbitrary schemes.
   - Preserve canonical session data; export must be read-only with respect to `.hatfield/sessions/<id>/events.jsonl`.

4. JSONL export requirements.
   - Use canonical `events.jsonl` as the source of truth.
   - Do not mutate the source events file.
   - If later branch-aware exports are required, implement them deliberately rather than adding compatibility shims.

5. Help/completion.
   - `/export` must appear in `/help` and slash-command completion via registry metadata.

## Test thesis

Without this change, a user cannot export an agent-core TUI session from the interactive UI; regressions would include the command not being registered, silently doing nothing, crashing the TUI, writing no file, writing unsafe HTML, or failing in the real TUI flow.

## Required tests

Per project rules, this TUI-visible feature is not complete without a real `TmuxHarness` E2E proof.

Add focused tests:
- `tests/Tui/Listener/ExportCommandHandlerTest.php`
  - populated session writes an export file and returns `TranscriptMessage` containing the path;
  - `.jsonl` path exports/copies canonical events;
  - empty/missing session produces friendly `TranscriptMessage` error;
  - quoted path with spaces is parsed correctly;
  - HTML content escapes user-controlled content.
- `tests/Tui/Listener/ExportCommandRegistrarTest.php`
  - registrar registers `/export` metadata and handler;
  - idempotent re-registration updates handler rather than duplicating metadata.
- TUI E2E proof with `TmuxHarness`, preferably by adding a small phase to `tests/Tui/E2E/TuiJourneyE2eTest.php` after a replay-backed session exists:
  - submit `/export`;
  - assert the TUI displays an export confirmation;
  - if practical, assert the exported file exists in the isolated test directory.

Do not add broad implementation-mirroring tests or enum/list coverage.

## Validation

Before touching tests or running QA, the worker must read:
- `.agents/skills/testing/SKILL.md`
- `tests/AGENTS.md`

Use Castor only:
- focused iteration: `castor test --filter=ExportCommandHandlerTest`
- TUI proof: `castor test:tui`
- architecture: `castor deptrac`
- final gate: `castor check`

Do not run raw `vendor/bin/*` commands. Do not run live LLM tests for this task unless implementation unexpectedly changes provider/LLM-visible behavior.

## Acceptance criteria
- `/export` is registered in the TUI, appears in `/help`, and is available to slash-command completion.
- `/export` with no args writes a standalone HTML export for the current session and shows a success `TranscriptMessage` with the path.
- `/export <path>.html` writes HTML to the requested path; quoted paths with spaces work.
- `/export <path>.jsonl` writes a JSONL export/copy from canonical session events without mutating the source session.
- Missing, empty, invalid, or unwritable sessions/paths produce friendly TUI transcript messages and do not crash the TUI.
- HTML export escapes untrusted session content; at least one regression test proves user-controlled HTML/script content is not emitted unsafely.
- Implementation stays within Deptrac boundaries: session-aware command logic lives under `src/Tui/Listener/`, not `src/Tui/Command/`.
- A real `TmuxHarness` E2E test proves `/export` works in the interactive TUI flow.
- Focused unit/registrar tests and final `castor check` pass.

## Workflow metadata
Status: DONE
Branch: task/tui-export-command
Worktree: /home/ineersa/projects/agent-core-worktrees/tui-export-command
Fork run: doxdbqp7e0mt
PR URL: https://github.com/ineersa/agent-core/pull/186
PR Status: merged
Started: 2026-06-21T17:43:15.616Z
Completed: 2026-06-21T20:09:24.357Z

## Work log
- Created: 2026-06-21T17:38:40.519Z

## Task workflow update - 2026-06-21T17:43:15.616Z
- Moved TODO → IN-PROGRESS.
- Created branch task/tui-export-command.
- Created worktree /home/ineersa/projects/agent-core-worktrees/tui-export-command.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/tui-export-command.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/tui-export-command.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/tui-export-command.
- Summary: Starting implementation phase per user request. Orchestrator loaded task-workflow and testing skills plus tests/AGENTS.md; implementation will be delegated to a fork in the task worktree.

## Task workflow update - 2026-06-21T17:44:23.571Z
- Recorded fork run: hyfurhi8xyr2
- Validation: Verification command: git -C /home/ineersa/projects/agent-core-worktrees/tui-export-command show --quiet --oneline 62f0843 => fatal: unknown revision; Verification command: ls src/Tui/Listener/ExportCommandHandler.php src/Tui/Listener/ExportCommandRegistrar.php => files missing; Verification command: git status --porcelain => clean
- Summary: Fork hyfurhi8xyr2 reported success, but orchestrator verification found no commit 62f0843, no ExportCommandHandler/Registrar files in the worktree, and a clean worktree still at the starting branch. Treating the fork result as invalid and relaunching implementation.

## Task workflow update - 2026-06-21T17:49:58.838Z
- Recorded fork run: 12hg5bra02f4
- Validation: git status --short in /home/ineersa/projects/agent-core-worktrees/tui-export-command shows partial uncommitted changes and generated hatfield-session-1.html; git log --oneline -3 shows branch still at 04b4a9f9c; no task commit created
- Summary: Fork 12hg5bra02f4 terminated before producing result.json. Orchestrator inspection shows partial uncommitted implementation artifacts remain in the worktree: modified tests/Tui/E2E/TuiJourneyE2eTest.php, untracked ExportCommandHandler/Registrar and tests, plus untracked generated hatfield-session-1.html. No commit was created. Awaiting user direction before launching any further fork or cleanup.

## Task workflow update - 2026-06-21T18:08:26.308Z
- Validation: git -C /home/ineersa/projects/agent-core-worktrees/tui-export-command status --short --branch => ## task/tui-export-command (clean); git -C /home/ineersa/projects/agent-core-worktrees/tui-export-command show --stat --oneline --summary 7a01c77e01962c3aeca4db50847249cdbc9f989b => 5 files changed, 1471 insertions(+), 1 deletion(-); Fork reported validation: handler/registrar tests passed, full non-E2E PHPUnit passed, TUI E2E group passed, phpstan/deptrac passed; caveat: fork report lists raw vendor/bin commands rather than Castor commands, so Castor validation remains for the task-to-pr phase.
- Summary: Delayed completion from fork hyfurhi8xyr2 arrived after earlier premature verification. Re-verified in the task worktree: commit 7a01c77e0 exists on task/tui-export-command, expected ExportCommandHandler/Registrar files exist, diff stat matches the fork handoff, and worktree is clean. Fork 12hg5bra02f4 is no longer active and left no dirty work after the final commit state. Implementation phase is complete; per task-start workflow, stop here until user requests task-to-pr/review.

## Task workflow update - 2026-06-21T18:18:24.136Z
- Summary: User feedback after trying /export: current HTML export is incomplete. It must provide a complete representation of events.jsonl, including user messages, AGENTS.md/instruction/skills registry content, assistant content, and any other event payloads rather than only selected rendered event types. Need iteration before PR.

## Task workflow update - 2026-06-21T18:39:49.426Z
- Recorded fork run: rhna4v5c4zp5
- Validation: git status --short --branch => ## task/tui-export-command (clean); git log --oneline -5 => HEAD 0a296d760 fix(tui): include complete event representation in HTML export; previous 7a01c77e0 feat(tui): implement /export slash command for session export; Fork reported Castor validation: castor test --filter=ExportCommandHandlerTest passed 19/19; castor test --filter=ExportCommandRegistrarTest passed 3/3; castor test:tui --filter=TuiJourneyE2eTest passed; castor deptrac passed; castor phpstan --path=src/Tui/Listener/ExportCommandHandler.php passed; castor cs-fix no fixes; castor test full suite passed 3030/3030.
- Summary: Iteration completed for user feedback that HTML export was incomplete. Commit 0a296d760 adds complete event-card rendering for every events.jsonl event, including metadata and full escaped raw JSON/details blocks while preserving friendly summaries for known events. This should expose user messages, assistant content, AGENTS.md/instruction/skills registry payloads, unknown event types, tool details, and any other event fields present in canonical events. Worktree verified clean with two task commits on branch.

## Task workflow update - 2026-06-21T18:43:49.543Z
- Summary: User feedback after second iteration: export is complete in raw JSON but readable visibility is still poor. Need user-facing readable sections for system prompt/instructions, user messages, tool call params, tool call results, token/usage stats, thinking (expandable), and similar high-value payload fields instead of forcing users to inspect raw JSON.

## Task workflow update - 2026-06-21T18:53:08.037Z
- Recorded fork run: 71ay1o65k6ii
- Validation: git status --short --branch => ## task/tui-export-command (clean); git log --oneline -6 => HEAD 3160c9d17 fix(tui): surface all key payload fields in readable HTML export; previous 0a296d760 and 7a01c77e0; Fork reported Castor validation: castor test --filter=ExportCommandHandlerTest passed 27/27; castor test --filter=ExportCommandRegistrarTest passed 3/3; castor test:tui --filter=TuiJourneyE2eTest passed; castor deptrac passed; castor phpstan --path=src/Tui/Listener/ExportCommandHandler.php passed; castor cs-fix no fixes.
- Summary: Iteration completed for readability feedback. Commit 3160c9d17 improves HTML export readable sections while keeping full raw JSON per event. It fixes real events.jsonl message extraction from payload.payload.messages, surfaces system/developer/user messages, assistant text/thinking, usage token stats, tool call arguments, tool execution result context, agent_command_applied messages, model notifications, and generic high-value fields for unknown events. Worktree verified clean with three task commits.

## Task workflow update - 2026-06-21T19:22:45.510Z
- Validation: Removed untracked /home/ineersa/projects/agent-core-worktrees/tui-export-command/hatfield-session-1.html; git status --short --branch => ## task/tui-export-command (clean); git rev-list --left-right --count origin/main...main => 0 16; git diff --stat main...HEAD => only 5 /export task files; git diff --stat origin/main...HEAD => includes unrelated src/CodingAgent/Tool/Edit* and tests/CodingAgent/Tool/EditFileToolTest.php plus /export files
- Summary: Preparing PR per user request without reviewer. Cleaned generated untracked export artifact from worktree. Found PR base blocker: local integration `main` is 16 commits ahead of `origin/main`, and task branch was created from local main. A PR against GitHub/default main would include unrelated edit-tool changes in addition to /export. Need user decision before creating PR: either approve rebasing/cherry-picking task commits onto origin/main, sync/push integration main first, or allow PR with unrelated local-main commits.

## Task workflow update - 2026-06-21T19:49:01.171Z
- Recorded fork run: doxdbqp7e0mt
- Validation: castor cs-check passed; castor phpstan --path=src/Tui/Listener/ExportCommandHandler.php passed; git status --short clean; HEAD 7767d9a0b style(tui): fix export handler formatting
- Summary: Final formatting fix after CODE-REVIEW gate failure. Fork doxdbqp7e0mt confirmed diff was CS-fixer formatting only, committed 7767d9a0b `style(tui): fix export handler formatting`, and left worktree clean.

## Task workflow update - 2026-06-21T19:53:58.361Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 1200s)...
- castor check passed (53.6s).
- Pushed task/tui-export-command to origin.
- branch 'task/tui-export-command' set up to track 'origin/task/tui-export-command'.
- Created PR: https://github.com/ineersa/agent-core/pull/186
- Validation: castor phpstan passed with errors=0,file_errors=0; castor test:controller-replay passed: OK (3 tests, 41 assertions); castor cs-check passed after formatting fix; worktree clean
- Summary: User requested PR without reviewer. Previous CODE-REVIEW gate encountered transient controller replay failure (only command.ack/run.started collected); immediately reran focused deterministic controller replay and it passed. Retrying CODE-REVIEW move.

## Task workflow update - 2026-06-21T20:09:24.357Z
- Moved CODE-REVIEW → DONE.
- Merged task/tui-export-command into integration checkout.
- Merge made by the 'ort' strategy.
 src/Tui/Listener/ExportCommandHandler.php         | 1345 +++++++++++++++++++++
 src/Tui/Listener/ExportCommandRegistrar.php       |   51 +
 tests/Tui/E2E/TuiJourneyE2eTest.php               |   37 +-
 tests/Tui/Listener/ExportCommandHandlerTest.php   |  943 +++++++++++++++
 tests/Tui/Listener/ExportCommandRegistrarTest.php |  107 ++
 5 files changed, 2482 insertions(+), 1 deletion(-)
 create mode 100644 src/Tui/Listener/ExportCommandHandler.php
 create mode 100644 src/Tui/Listener/ExportCommandRegistrar.php
 create mode 100644 tests/Tui/Listener/ExportCommandHandlerTest.php
 create mode 100644 tests/Tui/Listener/ExportCommandRegistrarTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/tui-export-command.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/tui-export-command.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR merged by user: https://github.com/ineersa/agent-core/pull/186
- Summary: User reports PR #186 was merged. Moving task to DONE to merge/sync integration checkout and clean up worktree.
