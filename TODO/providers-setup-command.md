# Interactive providers:setup command (guided initial provider configuration)

## Goal
Interactive `bin/console providers:setup` command — the initial-setup on-ramp consuming the catalog from task `providers-catalog` (hard dependency; the catalog file is the preset data source).

## UX

```
Provider to set up:
  [0] Z.ai (GLM)            [1] DeepSeek
  [2] OpenAI Codex (OAuth)  [3] Grok / xAI (OAuth)
  [4] Custom OpenAI-compatible (llama.cpp, RunPod, LM Studio, …)
```
Loop-based (Symfony choice() has no multi-select): pick one → configure → "Add another?" → closing question.

## Flows

**API-key presets (zai, deepseek)** — catalog supplies everything (base_url, compat, models); asks only:
1. API key: "store via env var (recommended)" → `api_key: env:ZAI_API_KEY` (existing env: resolution) or paste raw
2. → SettingsOverrideWriter set `ai.providers.<id>` = { enabled: true, api_key } to USER layer (sparse overlay — that's the whole entry, per providers-catalog)

**OAuth presets (codex, grok)**:
1. Write provider entry `enabled: true`
2. "Log in now?" → yes: invoke existing auth command IN-PROCESS (CommandApplication find('auth:codex')->run()) — no shell-out, no new auth code
3. login skipped/failed → rewrite `enabled: false` + print the auth command hint

**Custom OpenAI-compatible wizard** (full definition — unknown provider id path is unchanged from today):
- provider id (slug), base URL, completions path (default /v1/chat/completions), API key optional (env: or raw)
- per model: id, display name, context window, max tokens, input [text]/[text,image], reasoning + thinking-level map (default passthrough), cost (default 0 = unknown)
- compat flags: supports_developer_role, thinking format

**Closing**: "Set default model?" → choice of just-configured models → set `ai.default_model`. Print files written.

## Implementation shape
- `src/CodingAgent/CLI/Providers/ProvidersSetupCommand.php` — #[AsCommand('providers:setup')], invokable (see AgentsInitCommand / CodexAuthCommand patterns)
- GOTCHA: bin/console has a hardcoded $visibleCommands allowlist (~line 117) — every new public command must be added there or it runs but never appears in `list`. Include 'providers:setup' in the allowlist in this task.
- Preset rendering reads config/ai-catalog.yaml (from providers-catalog) — NO duplicate preset data in PHP
- `--project` flag → write project layer instead of user layer; default user
- All writes via existing SettingsOverrideWriter (set/remove with empty-parent prune) — no new writer code

## Boundaries
- No provider enable/disable management UI beyond what setup writes, no model browsing/editing command, no usage/quota displays
- No key validation pings or silent fetches; the only network I/O is the explicit user-consented "refresh model catalog?" prompt delegating to providers:update (from task providers-catalog)
- Known-provider settings writes stay sparse; custom providers write full definitions (both are what the loader already accepts)

## Tests
Command tests via Symfony ConsoleTester: preset key flow (env: + raw), oauth in-process auth invocation (mock command), custom happy path, default-model write, --project layer. Follow .agents/skills/testing/SKILL.md + tests/AGENTS.md (state both were read in handoff). castor test / deptrac / phpstan / cs-check.

## Acceptance criteria
- bin/console providers:setup with provider picker loop covering known (catalog) + custom OpenAI-compatible providers
- API-key preset flow: 2 questions, writes sparse overlay to user settings (env: or raw key)
- OAuth preset flow: writes provider, offers in-process login via auth:codex/auth:grok, disabled + hint on skip/failure
- Custom wizard collects all AiProviderConfig-required fields with sane defaults; writes complete provider definition
- Default-model prompt with only just-configured models; writes ai.default_model
- SettingsOverrideWriter used for all writes (no new writer); user layer default, --project flag
- Tests per testing skill + tests/AGENTS.md; castor test / deptrac / phpstan / cs-check green

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
- Created: 2026-08-18T18:05:11.395Z
