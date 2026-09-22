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
Status: ARCHIVE
Branch: task/providers-catalog
Worktree: /home/ineersa/projects/agent-core-worktrees/providers-catalog
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/414
PR Status: merged
Started: 2026-08-18T20:54:09.158Z
Completed: 2026-08-19T19:50:00.028Z

## Work log
- Created: 2026-08-18T18:04:57.372Z
- 2026-08-18: Dropped symfony/models-dev composer dep (binary distribution freezes vendor; users have no composer). Replaced with runtime providers:update fetch + committed filtered snapshot fallback. Verified api.json size/ETag live.
- 2026-08-18: Flipped presence semantics — yaml models: entries are complete and authoritative (user decision: grok ships fully curated; glm-5.3 missing upstream is unacceptable as a gate). models.dev downgraded to metadata refresh (cost/context/max_tokens/modalities) for yaml-listed ids + discovery hints. include/extra split removed.

## Task workflow update - 2026-08-18T20:54:09.158Z
- Moved TODO → IN-PROGRESS.
- Created branch task/providers-catalog.
- Created worktree /home/ineersa/projects/agent-core-worktrees/providers-catalog.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/providers-catalog.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/providers-catalog.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/providers-catalog.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/providers-catalog.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/providers-catalog/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/providers-catalog.

## Task workflow update - 2026-08-18T21:29:43.158Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (168.0s).
- Pushed task/providers-catalog to origin.
- branch 'task/providers-catalog' set up to track 'origin/task/providers-catalog'.
- Created PR: https://github.com/ineersa/agent-core/pull/414
- Summary: Implementation 0d636b86a (18 files) + TUI wrap-tolerance hardening b9a5d4405. Focused review APPROVED. First CODE-REVIEW attempt: phpstan lane timeout 90s (cold result cache in fresh worktree) — warmed with standalone castor phpstan (0 errors, 68s), retrying.

## Task workflow update - 2026-08-18T21:36:11.441Z
- 2026-08-18: User decision (post-PR): trim shipped grok-cli catalog to grok-4.6 + grok-composer-2.5-fast only — users extend via settings models: wholesale override. Supersedes 'all 8 models' acceptance wording. Micro-fork idjs702ow8n1 on same branch updates PR #414.

## Task workflow update - 2026-08-18T22:28:57.136Z
- Moved CODE-REVIEW → IN-PROGRESS.

## Task workflow update - 2026-08-18T22:29:01.621Z
- 2026-08-18: USER REVIEW on PR #414 (REQUEST CHANGES, simplification): 1.4k LOC way too much; 5 classes → 1 service + 1 parser; snapshot file in config questioned (one-time download when updating — drop it; yaml-only resolution already complete so snapshot is redundant by design); ETag machinery over-built for '1 request'; AppConfigLoader seam too complicated; docs block word-salad. Simplification fork dispatched: collapse AiCatalogLoader/AiCatalogMerge/ModelsDevCache/ModelsDevMetadataFilter/ModelsDevProviderIdMap into single AiCatalog service + slim ProvidersUpdateCommand; delete snapshot + --refresh-snapshot + ETag; keep security allowlist (apply-time, single location), overlay semantics, discovery hints, tests consolidated.

## Task workflow update - 2026-08-18T22:40:52.597Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (164.1s).
- Pushed task/providers-catalog to origin.
- branch 'task/providers-catalog' set up to track 'origin/task/providers-catalog'.
- PR already exists: https://github.com/ineersa/agent-core/pull/414
- Summary: User PR review (REQUEST CHANGES: too much code) addressed by simplification commit a36917d92: 5 classes + snapshot + ETag → AiCatalog (259) + ProvidersUpdateCommand (96) = 355 LOC; invariants preserved (hostile-upstream allowlist test on final AiProviderConfig, yaml-only presence, soft-fail update); focused 71/217 green, deptrac/phpstan/cs green. Open: sparsify project .hatfield/settings.yaml grok list.

## Task workflow update - 2026-08-19T00:06:27.802Z
- Moved CODE-REVIEW → IN-PROGRESS.

## Task workflow update - 2026-08-19T00:06:31.756Z
- 2026-08-18: FINAL user decision (third signal, docs unchanged complaint): KILL the models.dev cache entirely. providers:update becomes advisory-only (fetch upstream, print new-model/cost/limit deltas vs catalog, write nothing). AiCatalog becomes pure yaml parser (~60-80 LOC) — upstream data structurally never enters config, allowlist unnecessary. Docs drop all cache/metadata-overlay wording. Fresh upstream costs land only via shipped catalog updates (release/manual yaml edit).

## Task workflow update - 2026-08-19T00:08:01.669Z
- 2026-08-18: SUPERSEDES prior entry — advisory-only rejected by user ('how do we update?'). FINAL design: providers:update updates the repo catalog file itself (lockfile-bump model: add new upstream models, apply cost/limit deltas, never touch connection settings/quirks/level maps, write formatted json, git diff + commit = the update path). Catalog becomes config/ai-catalog.json (machine-rewritable, no YAML comment loss). End users get fresh catalogs via releases; extend via settings models: override. AiCatalog = pure parser, no cache, no runtime allowlist (upstream data enters only via reviewed commit). Fork jh11m8rm1ecu.

## Task workflow update - 2026-08-19T00:18:11.049Z
- 2026-08-18: FOURTH design approved+dispatched (fork s8x94xbz1n9d): bundled config/ai-catalog.yaml gets version field; copied to ~/.hatfield/ai-catalog.yaml on first load (user's catalog, runtime parses with bundled fallback); startup warning when bundled version > user copy suggesting providers:update; providers:update = REBASE to bundled default (preserve user-added model ids, adopt version+connection fields) + SYNC models.dev (add new models, cost/limit deltas only, whitelist in command, never connection fields) → atomic write of USER catalog; runtime AiCatalog = pure parser + version check ≤120 LOC, zero cache/overlay/allowlist; docs purge cache wording.

## Task workflow update - 2026-08-19T00:52:37.723Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (162.7s).
- Pushed task/providers-catalog to origin.
- branch 'task/providers-catalog' set up to track 'origin/task/providers-catalog'.
- PR already exists: https://github.com/ineersa/agent-core/pull/414
- Summary: Final shape after 4 user-directed redesigns (cache→advisory→repo-edit→user-copy lifecycle): ed65c6178 (lifecycle redesign, review verdict on prior shape had 2 blockers) → 473af0b5a (blockers fixed + first TUI notice attempt at wrong seam) → 4158406e0 (seam corrected to LoadedResources startup header per user: '⚠ AI Catalog: update available — run bin/console providers:update', castor check 446.8s green) → ecd83bb94 (empty-item sections drop '(none)' noise). Reviewer approved runtime purity, whitelist isolation, rebase rules, soft-fail ordering, loader semantics; its two blockers closed in 473af0b5a. Open notes: PSR-3 + TUI dual signal intentional; Yaml::dump strips user-catalog comments after first update (disclosed); project .hatfield/settings.yaml still pins 8 grok models (separate decision).

## Task workflow update - 2026-08-19T18:15:40.628Z
- Moved CODE-REVIEW → IN-PROGRESS.

## Task workflow update - 2026-08-19T18:16:02.386Z
- 2026-08-18: LIVE-RUN fix (fork 7x8dsthwu4bq): first real providers:update added 68 models (incl. gpt-3.5-turbo, embeddings, image/video models) and re-added the 11 grok models the user trimmed — auto-add defeats curation. New semantics: sync = metadata deltas for existing models ONLY; new upstream ids printed as availability hints, never added. Command shrinks. Live user catalog reset + re-run verification included.

## Task workflow update - 2026-08-19T18:20:53.863Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (150.7s).
- Pushed task/providers-catalog to origin.
- branch 'task/providers-catalog' set up to track 'origin/task/providers-catalog'.
- PR already exists: https://github.com/ineersa/agent-core/pull/414

## Task workflow update - 2026-08-19T19:50:00.029Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/providers-catalog.
- Merged task/providers-catalog into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/catalog.php                                |  32 ++
 .castor/tasks.php                                  |   6 +
 .hatfield/settings.yaml                            |  67 ----
 AGENTS.md                                          |   1 +
 bin/console                                        |   1 +
 castor.php                                         |   1 +
 config/ai-catalog.yaml                             | 184 +++++++++++
 config/hatfield.defaults.yaml                      | 276 +---------------
 config/services.yaml                               |   5 +
 docs/ai-catalog.md                                 | 103 ++++++
 docs/settings-models.md                            |  11 +
 docs/settings.md                                   |  15 +-
 .../CLI/Providers/ProvidersUpdateCommand.php       | 305 +++++++++++++++++
 src/CodingAgent/Config/Ai/AiCatalog.php            | 189 +++++++++++
 .../Config/Ai/AiCatalogVersionGuard.php            | 162 +++++++++
 src/CodingAgent/Config/AppConfigLoader.php         |  61 +++-
 src/CodingAgent/Config/AppResourceLocator.php      |   8 +
 .../LoadedResourcesSummaryBuilder.php              |  36 +-
 src/Tui/Startup/LoadedResourcesWidget.php          |   7 +-
 .../CLI/Providers/ProvidersUpdateCommandTest.php   | 363 +++++++++++++++++++++
 tests/CodingAgent/Config/Ai/AiCatalogTest.php      | 258 +++++++++++++++
 .../Config/Ai/AiCatalogVersionGuardTest.php        | 149 +++++++++
 tests/CodingAgent/Config/AppConfigLoaderTest.php   |  65 ++++
 .../LoadedResourcesSummaryBuilderTest.php          |  78 +++++
 .../Screen/TuiSkillReadCardVirtualRenderTest.php   |   8 +-
 25 files changed, 2045 insertions(+), 346 deletions(-)
 create mode 100644 .castor/catalog.php
 create mode 100644 config/ai-catalog.yaml
 create mode 100644 docs/ai-catalog.md
 create mode 100644 src/CodingAgent/CLI/Providers/ProvidersUpdateCommand.php
 create mode 100644 src/CodingAgent/Config/Ai/AiCatalog.php
 create mode 100644 src/CodingAgent/Config/Ai/AiCatalogVersionGuard.php
 create mode 100644 tests/CodingAgent/CLI/Providers/ProvidersUpdateCommandTest.php
 create mode 100644 tests/CodingAgent/Config/Ai/AiCatalogTest.php
 create mode 100644 tests/CodingAgent/Config/Ai/AiCatalogVersionGuardTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/providers-catalog.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/providers-catalog.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-08-21T21:49:46+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
