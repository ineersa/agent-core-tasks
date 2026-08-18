# Split AI provider catalog from settings (config/ai-catalog.yaml + runtime models.dev refresh)

## Goal
Split reference data (provider/model catalog) from user settings. Today known-provider definitions live as commented 60-line YAML templates in config/hatfield.defaults.yaml (~:427 deepseek, :481 zai, :536 openai-codex, plus the grok block added by task grok-cli-provider-support) that users copy-paste and hand-maintain — model churn (glm-5.1→5.3, grok-4.x→4.6) makes this stale immediately.

REVISION 2026-08-18: composer package (symfony/models-dev) approach DROPPED. Hatfield ships as PHAR/static binary — vendor is frozen inside the artifact, end users have no composer. Catalog freshness must come from a runtime fetch of https://models.dev/api.json with a bundled fallback snapshot. (Verified: full api.json 3.8MB but filtered to our providers ~50KB; ETag supported → conditional GET.)

## Architecture

**`config/models-dev.snapshot.json`** (committed) — models.dev data FILTERED to catalog providers only (~50KB: zai, deepseek, xai, openai). The offline/fallback layer baked into the binary. Refreshed by maintainers via `bin/console providers:update --refresh-snapshot` (same fetch code, writes repo file) or CI — no manual curation.

**`bin/console providers:update`** — fetches https://models.dev/api.json via Symfony HttpClient (already a dep), ETag conditional GET (`If-None-Match`; server supports etag + must-revalidate), keeps ONLY catalog provider ids (mapping: grok→xai, codex→openai, zai→zai, deepseek→deepseek), writes filtered subset atomically to `~/.hatfield/cache/models-dev.json` (0600, tmp+rename). Offline/304 → keep existing cache, exit 0 with status note. Never fails hard on network errors — stale cache + snapshot is a supported state.

GOTCHA: add 'providers:update' to bin/console's hardcoded $visibleCommands allowlist (~line 117) or it runs but never appears in `list`.

**`config/ai-catalog.yaml`** (committed) — the CURATED layer. Per known provider:
```yaml
providers:
    zai:
        label: 'Z.ai (GLM)'
        kind: apikey            # apikey | oauth
        base_url: https://api.z.ai/api/coding/paas/v4
        api: openai-completions
        completions_path: /chat/completions
        compatibility: { supports_developer_role: false, thinking_format: zai }
        auth_command: null      # or 'auth:codex' / 'auth:grok'
        models:                             # FULL curated entries — yaml is the authority for model PRESENCE
            glm-5.3: { name: GLM 5.3, context_window: 1000000, max_tokens: 131072, input: [text], tool_calling: true, reasoning: true, thinking_level_map: {...}, cost: {...} }
            glm-5v-turbo: { ... }
            # grok: all models from the grok-cli-provider-support task catalog (incl. grok-composer-2.5-fast) — curated, zero upstream dependence
```

**Field split — presence vs refresh**:
- `config/ai-catalog.yaml` `models:` entries are COMPLETE and authoritative for which models exist. Every shipped model (glm-5.3, all 8 grok models incl. composer, deepseek line) is a full yaml entry — nothing requires models.dev to resolve. No include/extra split, no "unresolvable id" degradation path.
- models.dev data (`~/.hatfield/cache/models-dev.json` fresh → `config/models-dev.snapshot.json` fallback) only REFRESHES volatile metadata for ids already in yaml: cost, context_window, max_tokens, input modalities, reasoning, tool_call. Never adds, never removes, never touches base_url/compat/thinking_level_map (Hatfield quirks stay curated).
- `providers:update` output includes a DISCOVERY hint: upstream models not present in yaml get listed ("upstream has new zai models: glm-5.4, …") — opt-in by a maintainer/user adding them to yaml. No auto-add.

**Fetch policy**: network ONLY inside `providers:update` (and its optional call from `providers:setup` — task providers-setup-command — which offers "refresh model catalog?" when cache is missing or >7 days old). NEVER fetch during agent sessions or model resolution.

**Merge layering** (new lowest layer, below defaults): models.dev metadata (for `include` ids) + catalog yaml (connection settings, quirks, `extra` models win over upstream) → existing AiProviderConfig::fromArray → then existing user/project settings as sparse overlay.

**Overlay semantics** (settings stay sparse; for known providers `{ enabled: true, api_key: env:ZAI_API_KEY }` is now a complete user entry):
- scalar keys from settings win (enabled, api_key, base_url overrides)
- `models:` key in settings replaces catalog models wholesale (pin/trim mechanism)
- unknown provider ids in settings pass through as full definitions, unchanged from today

**models.dev → Hatfield mapping** (verified against live api.json 2026-08-18): `limit.context`→context_window, `limit.output`→max_tokens, `modalities.input`→input (filter to text/image), `reasoning`→reasoning, `tool_call`→tool_calling, `cost.input/output`→cost (per million — VERIFY Hatfield AiModelDefinition cost unit matches and document in code comment). `reasoning_options.values` can seed thinking_level_map only where semantics are clear; otherwise default passthrough map and keep curated map in yaml.

## Hard security invariant
Connection settings (base_url, api, paths, auth) are NEVER sourced from models.dev — only per-model metadata is. Upstream data must never redirect where API keys are sent. models.dev does carry an `api` URL field; do not read it. The filter step (keep only catalog provider ids + model fields in the allowlist) enforces this structurally.

## Why yaml is authoritative (verified against live api.json 2026-08-18)
- models.dev zai has no glm-5.3 (has glm-5, glm-5.1, glm-5.2) — the newest flagship, exactly what users want, would be missing if upstream gated presence
- xai CLI-gated models absent: grok-composer-2.5-fast missing upstream — Grok ships a fully curated catalog (the 8-model list from task grok-cli-provider-support)
- Upstream freshness still pays for costs/context/limits of models that DO exist upstream — that's the refresh role

## Cleanup
- Delete commented provider templates from config/hatfield.defaults.yaml (deepseek/zai/openai-codex/grok blocks) → one-line pointer: see config/ai-catalog.yaml + bin/console providers:setup
- Migrate grok template in (depends on grok-cli-provider-support having landed)

## Boundaries
- NO composer dependency for catalog data; no vendored snapshot package
- No background/auto fetch outside providers:update (and the explicit prompt in providers:setup)
- No per-model delete/merge syntax (wholesale models: replace is the whole mechanism)
- No new settings keys in AiProviderConfig — catalog is input to the same fromArray shape
- Do not touch Codex/Grok auth or bridge code

## Docs
docs/settings.md: known-provider settings become sparse overlays; catalog file + providers:update documented.

## Acceptance criteria
- config/ai-catalog.yaml shipped with complete curated model lists for zai (incl. glm-5.3), deepseek, openai-codex, and grok (all 8 models from grok-cli-provider-support, incl. grok-composer-2.5-fast) + connection settings + quirks
- config/models-dev.snapshot.json committed (filtered to catalog providers), regenerable via providers:update --refresh-snapshot
- bin/console providers:update: ETag fetch → filtered cache at ~/.hatfield/cache/models-dev.json; offline/304 keeps existing data; no hard failure on network error
- Network I/O exists ONLY in providers:update / its providers:setup prompt; agent runtime and model resolution never fetch
- Sparse overlay merge: catalog < defaults < user < project; scalar settings win; settings models: replaces catalog models wholesale; unknown provider ids unchanged
- models.dev NEVER gates model presence: with cache and snapshot both absent, every yaml model still resolves (yaml entries are complete)
- Connection settings never sourced from models.dev — structurally enforced by allowlist filter
- Hatfield cost units vs models.dev units documented with test
- Commented provider templates removed from hatfield.defaults.yaml, replaced by short pointer
- docs/settings.md updated; tests per .agents/skills/testing/SKILL.md + tests/AGENTS.md (fork states both were read)
- castor test / deptrac / phpstan / cs-check green

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
- Created: 2026-08-18T18:04:57.372Z
- 2026-08-18: Dropped symfony/models-dev composer dep (binary distribution freezes vendor; users have no composer). Replaced with runtime providers:update fetch + committed filtered snapshot fallback. Verified api.json size/ETag live.
- 2026-08-18: Flipped presence semantics — yaml models: entries are complete and authoritative (user decision: grok ships fully curated; glm-5.3 missing upstream is unacceptable as a gate). models.dev downgraded to metadata refresh (cost/context/max_tokens/modalities) for yaml-listed ids + discovery hints. include/extra split removed.
