# Interactive providers:setup command (guided initial provider configuration)

## Goal
Interactive `hatfield providers:setup` — the initial-setup on-ramp consuming the bundled AI catalog (from merged task providers-catalog). Hatfield ships with **zero enabled providers**; this command is the comfortable way to enable what you need.

## Core design decisions (user, 2026-08-19)
- Ship without any enabled provider by default (catalog already ships `enabled: false` everywhere — verify nothing else enables by default).
- API-key providers (zai, deepseek): ask the user **where the API key is** — env var name (recommended, `api_key: env:VAR`) or pasted raw key.
- OAuth providers (codex, grok): enable the provider, then at the end print a clear message telling the user to run the appropriate login command (`hatfield auth:codex` / `hatfield auth:grok`). NO in-process auth invocation, no shell-out — just the message.
- Comfortable, user-friendly Symfony CLI: SymfonyStyle, coloring, clear labels and next-step hints.
- HARD RULE from providers-catalog review: never write `models:` maps for known/catalog providers — catalog inheritance would break (8-grok-models pin regression). Sparse writes only: `enabled`, `api_key` (+ nothing else for presets). Custom providers still write full definitions (no catalog entry exists for them).

## UX

```
 AI Provider Setup
 =================

Provider to set up:
  [0] Z.ai (GLM)            [1] DeepSeek
  [2] OpenAI Codex (OAuth)  [3] Grok / xAI (OAuth)
  [4] Custom OpenAI-compatible (llama.cpp, RunPod, LM Studio, …)
  [q] Done
```
Loop-based (Symfony choice() has no multi-select): pick one → configure → "Add another?" → closing. Show enabled state next to each provider (from resolved config). Use SymfonyStyle section()/info()/success()/warning() for comfortable output.

## Flows

**API-key presets (zai, deepseek)** — catalog supplies base_url, compat, models; ask only:
1. "Where is your API key?" → env var name (default suggestion e.g. ZAI_API_KEY, validate [A-Z0-9_]+) → `api_key: env:ZAI_API_KEY` (existing env: resolution), or paste raw key
2. Sparse write to USER layer: `ai.providers.<id> = { enabled: true, api_key }` — nothing else

**OAuth presets (codex, grok)**:
1. Sparse write: `ai.providers.<id> = { enabled: true }`
2. Closing output prints colored notice: `Next step: run \`hatfield auth:grok\` to log in` (auth_command comes from the catalog)

**Custom OpenAI-compatible wizard** (full definition — unknown provider ids pass through unchanged):
- provider id (slug), base URL, completions path (default /v1/chat/completions), API key optional (env: or raw)
- per model: id, display name, context window, max tokens, input [text]/[text,image], reasoning + thinking-level map (default passthrough), cost (default 0 = unknown)
- compat flags: supports_developer_role, thinking format

**Closing**: "Set default model?" → choice of models from just-enabled providers (catalog models for presets) → `ai.default_model`. Print files written + any pending auth commands.

## Implementation shape
- `src/CodingAgent/CLI/Providers/ProvidersSetupCommand.php` — #[AsCommand('providers:setup')], invokable (see AgentsInitCommand / CodexAuthCommand patterns)
- GOTCHA: bin/console has a hardcoded $visibleCommands allowlist (~line 117) — must add 'providers:setup' there or it runs but never appears in `list`
- Preset list reads config/ai-catalog.yaml via AiCatalog/provider metadata — NO duplicate preset data in PHP
- `--project` flag → write project layer instead of user layer; default user
- All writes via existing SettingsOverrideWriter — no new writer code
- Zero network I/O in this command

## Startup gate (user, 2026-08-19)
- Agent startup (run:agent / headless agent paths) with ZERO enabled providers must fail fast with a friendly, colored message: no providers configured → `hatfield providers:setup` hint → exit non-zero. No weird downstream bugs.
- CRITICAL: the gate covers agent-starting commands ONLY — `providers:setup`, `auth:codex`, `auth:grok`, `providers:update`, and all other console commands must work with zero providers enabled (chicken-and-egg).
- Gate where AiConfig is first needed for an agent run (AgentCommand entry / InteractiveMode boot — find the single choke point), not inside AiCatalog/loader (loader stays permissive; tests and setup read config with zero providers fine).
- Test: zero enabled → command exits with error message containing providers:setup; one enabled → boots (can assert at command layer with mocked kernel/agent if full boot isn't testable; follow existing AgentCommand test patterns).
- TUI/runtime-touching → full castor check required on this task.

## Boundaries
- No provider management UI beyond what setup writes, no model browsing/editing command, no usage/quota displays, no key validation pings
- Known-provider settings writes stay sparse (enabled + api_key only); custom providers write full definitions
- No models: writes for known providers, ever

## Tests
Command tests via Symfony ConsoleTester: preset env-key flow, preset raw-key flow, oauth flow (enabled write + auth-command message in output), custom happy path, default-model write, --project layer, no models: key in written settings for presets. Follow .agents/skills/testing/SKILL.md + tests/AGENTS.md (state both were read in handoff). castor test / deptrac / phpstan / cs-check.

## Acceptance criteria
- `hatfield providers:setup` picker loop covering known (catalog) + custom OpenAI-compatible providers, comfortable SymfonyStyle output
- API-key preset: asks where the key is (env var name or raw), writes sparse user-layer entry (enabled + api_key only)
- OAuth preset: writes enabled entry, prints colored `hatfield auth:codex`/`hatfield auth:grok` next-step message; no auth execution
- Custom wizard collects AiProviderConfig-required fields with sane defaults; writes complete definition
- Default-model prompt from just-enabled providers; writes ai.default_model
- No models: map written for known providers (asserted in test)
- 'providers:setup' added to bin/console $visibleCommands
- SettingsOverrideWriter for all writes; user layer default, --project flag
- Tests per testing skill; castor test / deptrac / phpstan / cs-check green

## Workflow metadata
Status: DONE
Branch: task/providers-setup-command
Worktree: /home/ineersa/projects/agent-core-worktrees/providers-setup-command
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/417
PR Status: merged
Started: 2026-08-19T19:55:29.607Z
Completed: 2026-08-21T02:51:38.773Z

## Work log
- Created: 2026-08-18T18:05:11.395Z
- 2026-08-19: Redesigned per user decisions: ship zero-enabled default; OAuth = message only (auth:codex/auth:grok hint, no in-process invocation); ask key location (env var name vs raw); HARD RULE no models: writes for known providers; dropped refresh-catalog prompt (catalog is local now); explicit user-friendly/coloring requirement.

## Task workflow update - 2026-08-19T19:55:29.607Z
- Moved TODO → IN-PROGRESS.
- Created branch task/providers-setup-command.
- Created worktree /home/ineersa/projects/agent-core-worktrees/providers-setup-command.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/providers-setup-command.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/providers-setup-command.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/providers-setup-command.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/providers-setup-command.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/providers-setup-command/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/providers-setup-command.

## Task workflow update - 2026-08-19T20:06:31.336Z
- 2026-08-19: Fork f5qpeiw061kb done (commit 8d2e6a570): ProvidersSetupCommand 463 LOC + 6 tests + allowlist — sparse preset writes (no models:), OAuth hint-only, custom wizard, default-model closer. Gate fork hw5dipk4zzo9 running (zero-enabled startup error). USER ADDITION: picker must support DISABLE — selecting an enabled provider offers enable/disable; disable writes {enabled: false} sparse (or removes override to fall back to catalog default false — implementer's choice, keep it simple).

## Task workflow update - 2026-08-19T20:10:45.596Z
- Disable support landed in commit 0761375ae (not pushed): configure/disable/cancel for already-enabled providers; sparse {enabled:false}; [disabled] badge; no auth hints for disabled; warn if default_model points at disabled provider (no rewrite). Tests: 9/76 green. Concurrent gate fork left uncommitted AgentCommand.php dirty (untouched by this fork).

## Task workflow update - 2026-08-19T20:21:14.004Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (153.9s).
- Pushed task/providers-setup-command to origin.
- branch 'task/providers-setup-command' set up to track 'origin/task/providers-setup-command'.
- Created PR: https://github.com/ineersa/agent-core/pull/417

## Task workflow update - 2026-08-19T20:48:27.845Z
- Updated PR Status: open
- Summary: Re-review of blocker fix 45b21d433 by reviewer subagent: APPROVED, zero issues. Collision guard uses the same raw catalog array as presets (covers user-copy + bundled ids, case-insensitive), all four settingsWriter call sites traced — no path writes models: under catalog ids, regression test asserts rejection + no zai entry + my-llm write, spec-fidelity exact (+49/−3, 2 files). PR #417 at 45b21d433, full castor check green, MERGEABLE — awaiting user merge. NTH notes: optional message polish for pre-existing colliding entries (preset pick silently replaces old custom definition under same id — delete manually first).

## Task workflow update - 2026-08-19T21:55:19.903Z
- Moved CODE-REVIEW → IN-PROGRESS.

## Task workflow update - 2026-08-19T22:16:53.573Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (147.9s).
- Pushed task/providers-setup-command to origin.
- branch 'task/providers-setup-command' set up to track 'origin/task/providers-setup-command'.
- PR already exists: https://github.com/ineersa/agent-core/pull/417

## Task workflow update - 2026-08-19T22:23:12.845Z
- Moved CODE-REVIEW → IN-PROGRESS.

## Task workflow update - 2026-08-20T00:11:26.969Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (157.2s).
- Pushed task/providers-setup-command to origin.
- branch 'task/providers-setup-command' set up to track 'origin/task/providers-setup-command'.
- PR already exists: https://github.com/ineersa/agent-core/pull/417

## Task workflow update - 2026-08-20T02:35:46.720Z
- Moved CODE-REVIEW → IN-PROGRESS.

## Task workflow update - 2026-08-20T03:13:01.236Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (147.5s).
- Pushed task/providers-setup-command to origin.
- branch 'task/providers-setup-command' set up to track 'origin/task/providers-setup-command'.
- PR already exists: https://github.com/ineersa/agent-core/pull/417

## Task workflow update - 2026-08-20T15:31:38.703Z
- Moved CODE-REVIEW → IN-PROGRESS.

## Task workflow update - 2026-08-20T15:32:21.247Z
- UX round 2 follow-up queued (after fork 4u2ql30jzbb2 lands): persistent footer bar with keybind hints per phase — picker: ↑/↓ select, Enter confirm, Esc exit; input: Enter submit, Esc back; list: Enter select, Esc back. Also make Ctrl+D behave as quit (SIGINT-safe tui->stop path) since user expected it alongside Esc. Must verify how Symfony TUI InputWidget/SelectListWidget handle ctrl+d keybinding before committing to it.

## Task workflow update - 2026-08-20T16:59:02.581Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (154.0s).
- Pushed task/providers-setup-command to origin.
- branch 'task/providers-setup-command' set up to track 'origin/task/providers-setup-command'.
- PR already exists: https://github.com/ineersa/agent-core/pull/417

## Task workflow update - 2026-08-20T18:40:01.843Z
- Validation: castor check: quality ok (448.6s) — 4747 tests/19062 asserts, replay 13/252, tui 40/333, llm-real 13/144, phpstan 0, cs clean, docs ok, catalog:version-check ok; focused: castor test --filter='ProvidersSetup|SetupSetup' 28/170 OK; live PTY: custom wizard Step1→Step2→paste URL→Enter → Step 3 confirmed by fix fork; reviewer: APPROVED on 0675a2413 (blocker resolved exactly, single-registration by construction grep-verified)
- Summary: Enter-key bug fixed (vendor detach wipes listeners): root cause traced live (VirtualTerminal repro — paste+\r never advanced), fix fork 66cc573bf + ponytail simplification 0675a2413 (unconditional wire-after-add in applyPhaseLayout, −14 LOC net). Reviewer cycle: REQUEST CHANGES (sentinel guards provably unnecessary) → fix → APPROVED. Merged origin/main (a2212b009, conflict was identical display-normalization assert, took main's). Full castor check green 448.6s (4747 unit tests, phpstan 0 after result-cache warm-up for the 90s lane timeout, cs clean, deptrac 0). PR #417 OPEN/MERGEABLE at a2212b009.

## Task workflow update - 2026-08-20T20:19:32.962Z
- Validation: castor check: quality ok (531.7s); focused: 29 tests / 180 assertions OK; phpstan --path=src/Tui/Setup: 0 errors; cs-check clean, deptrac 0 (polish commit); reviewer: APPROVED on 24fad39f9 — all four findings resolved exactly, preset startApiKey hint verified live; VirtualTerminal render proof: panel borders + Step 2 help/example visible (fork qc3vngmw47bs)
- Summary: UI polish round landed: bordered panel (ContainerWidget Border::all + Padding::xy, list/input nested inside), bold cyan title, plain-English help + example lines for all 13 wizard steps via customStepHelp()/formatStepHelp(). Reviewer cycle: REQUEST CHANGES (wrong api_raw_key example contradicting empty-key rejection; redundant setStyle; 7 dead hint writes) → fixed in 6369287c5 + 24fad39f9 → APPROVED. Full castor check green 531.7s. PR #417 OPEN/MERGEABLE at 24fad39f9 (tip: 25a36a6bb panels+help, 6369287c5 example fixes, 24fad39f9 dead-hint cleanup).

## Task workflow update - 2026-08-20T20:52:24.672Z
- Validation: castor check: quality ok (536.0s); focused: 41 tests / 265 assertions OK; phpstan src/Tui/Setup + src/CodingAgent/CLI/Providers: 0 errors; cs-check clean, deptrac 0; reviewer: APPROVED on ae67664cf — loop deletion verified dead in every reachable path; oauth No-path, submenu routing, ghost-guard, edit prefill, full-definition preservation all verified; VirtualTerminal dumps: picker (catalog-only rows), Your servers submenu (runpod/llama-local + url + glyphs), server action menu (Edit/Disable/Remove/Cancel)
- Summary: Round 3 (user-reported UX fixes) landed at ae67664cf: (1) another-model list uses action labels 'Add another model'/'Finish'; (2) OAuth enable gets Yes/No confirm before writing (No = nothing written); (3) Reconfigure removed from OAuth action menu entirely (API-key keeps it); (4) 'Other server' opens Your-servers submenu (id+url+status glyph rows, Add-new, Back) with per-server Edit (prefilled wizard, skips id step)/Disable|Enable/Remove(confirm); custom toggles use full-definition rewrites (sparse write would destroy config — invariant documented in interface docblock after review caught a wrong comment); SettingsOverrideWriter::remove() reused. Reviewer cycle: REQUEST CHANGES (wrong docblock, dead loop, dup docblock) → ae67664cf → APPROVED. Full castor check green 536.0s. PR #417 OPEN/MERGEABLE at ae67664cf.

## Task workflow update - 2026-08-20T21:56:26.727Z
- Updated PR Status: open
- Validation: castor check quality: ok (553.2s) at 27e6538ab — 4780 unit, controller-replay, tui, llm-real, phpstan, cs, docs, catalog:version-check; Focused suites: 50/307 (setup), 129/455 (AppConfig|LoadedResources|AgentCommand), phpstan 0, cs clean, deptrac 0; PR #417 updated to 27e6538ab, OPEN/MERGEABLE
- 5f3f8caf0: Continue?/Exit confirm (no default-model nag), Set default model picker row, Done→summary direct
- 69aaef17b: seed $configured from already-enabled settings providers — row shows on reopen, no fabricated auth hints
- b9e6917ff: review fixes — defaultModelWarningFor uses currentDefaultModel() (stale-default bug), ~36 dead lines cut, re-enable no-dup test
- d89a419ea: boot fallback — unavailable ai.default_model no longer crashes boot; first-available model + ⚠ startup warning; malformed still throws; AgentCommand gate owns zero-provider
- 27e6538ab: review fix — fallback no longer mutates $raw (false /settings 'Restart required' regression); dead branches cut
- Review cycle: REQUEST CHANGES → fix → APPROVED ×3; full castor check green; pushed ae67664cf..27e6538ab

## Task workflow update - 2026-08-21T02:46:19.286Z
- Validation: castor check quality: ok (493.9s) at 11d41faff (phpstan warm-cache retry after 90s lane timeout — known cold-cache flake); Focused: 49 tests/323 assertions, phpstan 0, cs clean, deptrac 0; PR #417 pushed to 11d41faff, OPEN/MERGEABLE
- b23a21b66: custom wizard → one-screen SettingsListWidget form (SettingsTextInputWidget bridge for vendor SelectEvent contract); line count ~flat but UX is single-screen edit-any-order
- 180902a15: review fixes — orphaned widget stacking dead (remove-before-reassign + updateValue), 3 real-input submenu tests through the actual SelectEvent bridge, UTF-8 fix, ~50 lines cut; castor wall 180→210 per user
- 11d41faff: ponytail — unreachable save branch + dead ?? fallbacks deleted, Provider-id exactly-once orphan assert; reviewer APPROVED
- SettingsList refactor cycle: REQUEST CHANGES → fix → REQUEST CHANGES (2 residues) → APPROVED; pushed 27e6538ab..11d41faff

## Task workflow update - 2026-08-21T02:51:38.773Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/providers-setup-command.
- Merged task/providers-setup-command into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/process.php                                |    2 +-
 .castor/tasks.php                                  |   10 +-
 bin/console                                        |    1 +
 config/ai-catalog.yaml                             |    4 +-
 depfile.yaml                                       |    9 +
 src/CodingAgent/CLI/AgentCommand.php               |   27 +
 .../CLI/Providers/ProvidersSetupCommand.php        |   72 ++
 .../CLI/Providers/ProvidersSetupFlow.php           |  616 ++++++++++
 src/CodingAgent/Config/AppConfig.php               |   65 +-
 .../LoadedResourcesSummaryBuilder.php              |   35 +
 src/Tui/Setup/ProvidersSetupFlowInterface.php      |  122 ++
 src/Tui/Setup/SettingsTextInputWidget.php          |   97 ++
 src/Tui/Setup/SetupScreen.php                      | 1264 ++++++++++++++++++++
 .../CLI/AgentCommandProvidersGateTest.php          |  186 +++
 .../CLI/Providers/ProvidersSetupCommandTest.php    |   84 ++
 .../CLI/Providers/ProvidersSetupFlowTest.php       |  655 ++++++++++
 tests/CodingAgent/Config/AppConfigTest.php         |   32 +-
 .../LoadedResourcesSummaryBuilderTest.php          |   59 +
 tests/Tui/Setup/SetupScreenVirtualRenderTest.php   |  946 +++++++++++++++
 19 files changed, 4243 insertions(+), 43 deletions(-)
 create mode 100644 src/CodingAgent/CLI/Providers/ProvidersSetupCommand.php
 create mode 100644 src/CodingAgent/CLI/Providers/ProvidersSetupFlow.php
 create mode 100644 src/Tui/Setup/ProvidersSetupFlowInterface.php
 create mode 100644 src/Tui/Setup/SettingsTextInputWidget.php
 create mode 100644 src/Tui/Setup/SetupScreen.php
 create mode 100644 tests/CodingAgent/CLI/AgentCommandProvidersGateTest.php
 create mode 100644 tests/CodingAgent/CLI/Providers/ProvidersSetupCommandTest.php
 create mode 100644 tests/CodingAgent/CLI/Providers/ProvidersSetupFlowTest.php
 create mode 100644 tests/Tui/Setup/SetupScreenVirtualRenderTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/providers-setup-command.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/providers-setup-command.
- Pulled integration checkout: Merge made by the 'ort' strategy..
