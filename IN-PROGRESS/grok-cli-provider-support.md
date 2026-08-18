# Add Grok CLI provider (xAI OAuth + cli-chat-proxy) to Hatfield

## Goal
## Goal

Basic Grok support: OAuth login via console command (same UX as `auth:codex`) and a usable `grok-cli` provider in Hatfield for models like `grok-composer-2.5-fast`. **No** usage checks, multi-account, quota rotation, dashboard, or image generation.

## Reference implementation (scouted)

Unofficial Pi extension at `~/claw/pi-grok-cli` (TypeScript). Read these files before implementing:

| File | Contents |
|---|---|
| `src/auth/config.ts` | Endpoints + client ID |
| `src/auth/oauth.ts` | Full OIDC PKCE login, device-code flow, refresh |
| `src/provider/stream.ts` | Spoofed client headers (version gate) |
| `src/models/catalog.ts` | Model catalog with costs/context windows |
| `src/payload/sanitize.ts` | xAI Responses dialect quirks (required reading) |
| `src/provider/register.ts` | How pieces fit together (skip account/rotation logic) |

## What the Grok CLI surface is

xAI ships an official `grok` CLI binary. It does NOT use `api.x.ai`; it talks to a separate endpoint `https://cli-chat-proxy.grok.com/v1` (xAI infrastructure) that fronts the chat backend and maps an X Premium/SuperGrok subscription to weekly allowances instead of per-token billing. The endpoint is gated to xAI's own client: it parses `User-Agent` for a grok client version and returns HTTP 426 when absent. pi-grok-cli reverse-engineered the required headers from captured traffic.

## Constants (from pi-grok-cli)

```
OIDC issuer:              https://auth.x.ai
OIDC discovery:           https://auth.x.ai/.well-known/openid-configuration
Token endpoint:           https://auth.x.ai/oauth2/token
OAuth client ID:          b1a00492-073a-47ea-816f-4c329264a828
Scope:                    openid profile email offline_access grok-cli:access api:access
API base URL:             https://cli-chat-proxy.grok.com/v1
Responses path:           /responses  (full: https://cli-chat-proxy.grok.com/v1/responses)
Auth:                     Bearer access_token
```

NOTE: xAI's OIDC is *more* standard than OpenAI's Hydra — `CodexOAuthProvider.php` exists only to strip Hydra quirks (`approval_prompt`, empty `client_secret`). The grok OAuth provider subclass should start from stock `League\OAuth2\Client\Provider\GenericProvider` and only add quirks if the first login attempt reveals them (verify what a plain `GenericProvider` sends: it may inject `approval_prompt` too — if so, reuse the Codex strip, but do not blindly copy).

### Required spoofed headers (every request)

```
User-Agent:                  grok-pager/0.2.91 grok-shell/0.2.91 (macos; aarch64)
x-grok-client-identifier:    grok-pager
x-grok-client-version:       0.2.91
x-xai-token-auth:            xai-grok-cli
x-grok-model-override:       <modelId>      # static per model
x-grok-conv-id:              <sessionId>    # dynamic per request
```

`0.2.91` must track the official grok CLI's current version (maintenance tax; make it a class constant with a comment referencing pi-grok-cli `src/provider/stream.ts`).

### Payload dialect quirks (xAI Responses ≠ stock OpenAI)

From `sanitize.ts` — required, this is not optional polish:

- Rejects `role: "system"` and `role: "developer"` in the input array → must be hoisted to top-level `instructions` (Hatfield `compatibility: supports_developer_role: false` covers the developer role part; verify how system role is handled by the Open Responses contract and whether `instructions` hoisting is needed at the bridge).
- Uses `text.format` instead of `response_format` (vendor ModelClient already does this — see below).
- Uses `prompt_cache_key` for conversation caching; does not support `prompt_cache_retention`.
- Replayed `reasoning` items must drop output-only `status` and carry typed content (`reasoning_text`); `reasoning.encrypted_content` include used.
- Empty-string content items cause validation failures → drop.
- `function_call_output.output` cannot contain image arrays.
- `image_url` parts must be normalized to `input_image` with base64 data URIs.
- `reasoning.effort` only valid on subset of models (see catalog `reasoning`/thinkingLevelMap).

## Model catalog (hardcode as Hatfield defaults)

| Model ID | Reasoning | Input | Cost in/out/cacheR/cacheW ($/M) | Ctx | MaxTokens |
|---|---|---|---|---|---|
| grok-composer-2.5-fast | false (thinkingLevelMap: off→none, rest null) | text+image | 3 / 15 / 0.5 / 0 | 200k | 30k |
| grok-build | true | text+image | 1 / 2 / 0.2 / 0.2 | 500k | 30k |
| grok-4.3 | true | text+image | 1.25 / 2.5 / 0.2 / 0 | 1M | 30k |
| grok-4.5 | true | text+image | 2 / 6 / 0.5 / 0 | 500k | 30k |
| grok-4.6 | true | text+image | 2 / 6 / 0.5 / 0 | 500k | 30k |
| grok-4.20-0309-reasoning | true | text+image | 1.25 / 2.5 / 0.2 / 0 | 2M | 30k |
| grok-4.20-0309-non-reasoning | false (thinkingLevelMap as composer) | text+image | 1.25 / 2.5 / 0.2 / 0 | 2M | 30k |
| grok-4.20-multi-agent-0309 | true | text+image | 1.25 / 2.5 / 0.2 / 0 | 2M | 30k |

## Hatfield implementation plan

Mirror the Codex OAuth + provider-builder flow 1:1 (Codex is the template):

### 1. Auth (clone of `src/CodingAgent/Auth/*Codex*`)

- `src/CodingAgent/Auth/GrokOAuthConfig.php` — constants above, auth storage key (e.g. `grok-cli`), default callback port (pick one ≠ Codex's; pi uses 56122).
- `src/CodingAgent/Auth/GrokOAuthProvider.php` — league GenericProvider configured for auth.x.ai, PKCE.
- `src/CodingAgent/Auth/GrokOAuthService.php` — login (browser loopback + manual code paste fallback, same as Codex) and `refreshCredentials()`.
- Reuse as-is: `LocalCallbackServer.php`, `BrowserLauncher.php`, `ManualCodeParser.php`, auth storage shape (where does Codex store? check `CodexAuthStorage.php` / `~/.hatfield/auth.json` — store grok tokens under key `grok-cli` the same way).
- NO device-code flow (Codex doesn't have one; parity is enough).
- `src/CodingAgent/CLI/Auth/GrokAuthCommand.php` — `bin/console auth:grok`, clone of `CodexAuthCommand` (login/logout/status subactions as Codex has them). Register in `config/services.yaml` + command registration the same way.

### 2. Provider builder (clone of `src/CodingAgent/Infrastructure/SymfonyAi/Codex/CodexSymfonyAiProviderBuilder.php`)

- `src/CodingAgent/Infrastructure/SymfonyAi/Grok/GrokSymfonyAiProviderBuilder.php` — `supports(): type === 'grok'`.
- Uses `vendor/symfony/ai-open-responses-platform` (already a composer dep): `ResponsesModel`, `ResultConverter`, `Factory::createProvider()` (accepts baseUrl/apiKey/httpClient/modelCatalog/eventDispatcher).
- `ProjectedSymfonyModelCatalog` with `ResponsesModel::class` for the model catalog projection.
- Access-token refresher closure wired to `GrokOAuthService::refreshCredentials()` (Codex builder does exactly this pattern).
- **Headers**: the vendor `ModelClient` hardcodes headers and only sets `auth_bearer`. Do NOT modify vendor. The Hatfield pattern for full control is `src/Platform/Bridge/OpenAICodex/CodexModelClient.php` — a `ModelClientInterface` impl that builds `headers` itself and reads `run_id` from invocation options. Create `src/Platform/Bridge/Grok/GrokModelClient.php` implementing `ModelClientInterface` (or extending the vendor ModelClient and overriding `request()`), which:
  - sets all spoofed headers incl. `x-grok-conv-id` from the `run_id` invocation option (session_id === run_id in Hatfield),
  - strips Hatfield-internal option keys before wire merge (same reason as `SanitizedGenericModelClient`; note the vendor Responses ModelClient merges `$options` into the JSON body, so `run_id`/`tools_ref`/`turn_no` MUST be stripped or they 400),
  - sets `x-grok-model-override` from the model,
  - applies payload dialect quirks listed above (or as many as the vendor ResultConverter/Contract already handles — verify before duplicating: `supports_developer_role: false` compatibility, `text.format` handling in vendor ModelClient).
  - URL: exactly `https://cli-chat-proxy.grok.com/v1/responses` — beware vendor default path is `/v1/responses` appended to rtrim'd baseUrl, so configure baseUrl without `/v1` or pass path `/responses`; do not double it.
- Deptrac: `src/Platform/` placement keeps boundaries clean; the builder in CodingAgent may depend on Platform (verify `depfile.yaml` — Codex builder already imports Platform classes, follow same edges).

### 3. Config / defaults

- `config/hatfield.defaults.yaml`: add commented-out `grok-cli` provider template (same style as the `openai-codex` block at ~line 536) with:
  ```yaml
  #         grok-cli:
  #             type: grok
  #             enabled: false
  #             base_url: 'https://cli-chat-proxy.grok.com'
  #             api: openai-responses
  #             completions_path: /v1/responses
  #             supports_completions: true
  #             supports_embeddings: false
  #             supports_thinking_levels: true
  #             compatibility: { supports_developer_role: false }
  #             models: { grok-composer-2.5-fast: {...}, ... }
  # ```
  User enables it manually in their own `.hatfield/settings.yaml` — no code-level enabled-by-default.
- Model catalog defaults live in this YAML template (user copies); ALSO hardcode a fallback catalog constant if the codebase has a default-models-in-code pattern — check how Codex models default before inventing one. If settings-only is the pattern, settings-only is correct.

### 4. Out of scope (do NOT build)

- Usage/quota checks (`/grok-cli-usage`), multi-account vault/rotation, billing, browser dashboard, image generation (`imagine`), device-code login, payload sanitize features beyond the dialect quirks above (e.g. pi's local-image-path resolution — Hatfield already has its own image handling).

## Constraints & risks

- Unofficial, undocumented endpoint; header spoof version will break when xAI bumps the client version — keep version a named constant, comment the maintenance procedure (bump `GROK_CLI_VERSION` to match current official CLI).
- Quota exhaustion will surface as raw 4xx; no friendly handling required (accepted scope).
- OAuth creds in `~/.hatfield/auth.json` under `grok-cli` key, 0600, same as Codex storage.
- Logs must never include tokens (AGENTS.md privacy rules).

## Validation

- Per testing skill (`castor` only, reports under `var/reports/`).
- Unit tests for: OAuth service login/refresh (mocked HTTP), GrokModelClient header injection + internal-key stripping + payload quirks (fixtures from pi-grok-cli sanitize tests if shapes match — see `~/claw/pi-grok-cli/tests/`).
- `castor test`, `castor deptrac`, `castor phpstan`, `castor cs-check`.
- LLM-visible provider change → `castor test:llm-real` is the focused live gate, but this provider has no fixtures/creds in CI; do a one-off manual live smoke with real creds (`bin/console auth:grok` then one session on `grok-cli/grok-composer-2.5-fast`) and record the result in the task. Manual smoke is not sufficient alone; unit + castor check are the gate.
- Docs: update `docs/settings.md` if new settings keys are introduced (keep `.hatfield/settings.yaml` example in sync).

## Acceptance criteria

- `bin/console auth:grok` performs browser PKCE login against `https://auth.x.ai` (with manual code paste fallback), stores access/refresh tokens under the `grok-cli` key with Codex-style storage hygiene (0600, same file).
- Expired access tokens are refreshed automatically during provider build/request (same refresher pattern as Codex builder).
- A provider entry with `type: grok` in Hatfield AI settings builds via `symfony/ai-open-responses-platform` and serves requests to `https://cli-chat-proxy.grok.com/v1/responses`.
- Every request carries the full spoofed header set (`User-Agent` grok-pager/grok-shell version, `x-grok-client-identifier`, `x-grok-client-version`, `x-xai-token-auth`, `x-grok-model-override`, `x-grok-conv-id`); Hatfield-internal option keys never reach the wire body.
- xAI dialect quirks (no system/developer role in input, reasoning item normalization, empty-content drop) are handled; a session on `grok-cli/grok-composer-2.5-fast` completes a real turn with tool use.
- `config/hatfield.defaults.yaml` documents the commented-out `grok-cli` provider template incl. model catalog; enabling is user-side settings only.
- Unit tests cover login/refresh (mocked HTTP), header injection, internal-key stripping, and dialect normalization; `castor test`, `castor deptrac`, `castor phpstan`, `castor cs-check` pass. One live manual smoke with real creds is run and recorded; no usage/multi-account/imagine features exist in the diff.

## Workflow metadata
Status: IN-PROGRESS
Branch: task/grok-cli-provider-support
Worktree: /home/ineersa/projects/agent-core-worktrees/grok-cli-provider-support
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/409
PR Status: open
Started: 2026-08-18T17:47:40.831Z
Completed:

## Work log
- Created: 2026-08-18T17:34:42.040Z

## Task workflow update - 2026-08-18T17:34:46.417Z
- Task created after scouting ~/claw/pi-grok-cli and Hatfield Codex auth/provider-builder seams. Scope confirmed with user: login + provider only, no usage/quota/accounts/imagine, hardcoded default catalog.

## Task workflow update - 2026-08-18T17:47:40.831Z
- Moved TODO → IN-PROGRESS.
- Created branch task/grok-cli-provider-support.
- Created worktree /home/ineersa/projects/agent-core-worktrees/grok-cli-provider-support.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/grok-cli-provider-support.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/grok-cli-provider-support.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/grok-cli-provider-support.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/grok-cli-provider-support.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/grok-cli-provider-support/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/grok-cli-provider-support.

## Task workflow update - 2026-08-18T18:06:02.106Z
- Validation: castor test — OK (4627 tests); castor deptrac — 0 violations; castor phpstan — 0 errors; castor cs-check — clean; castor test --filter=Grok — OK (25 tests)
- Summary: Implemented Grok CLI provider: auth:grok OAuth, GrokModelClient with spoofed headers, type:grok builder, defaults YAML catalog, unit tests. Commit 93a9a6ce9. Validation: castor test/deptrac/phpstan/cs-check all green.
- Read testing skill + tests/AGENTS.md before writing tests.
- Implemented auth/storage/CLI/model client/builder + defaults + docs + tests.
- LocalCallbackServer: optional callbackPath (Grok /callback).
- Reused CodexOAuthProvider for xAI (no clone).
- GrokModelClient uses bare HttpClient (not EventSource) like Codex — mock SSE content-type.
- Committed 93a9a6ce9 on task/grok-cli-provider-support.

## Task workflow update - 2026-08-18T18:15:50.291Z
- Summary: Review 1 (reviewer subagent): REQUEST CHANGES. Constants verified against live auth.x.ai OIDC discovery; security posture clean; OAuth provider reuse validated. Blockers: (1) BUG GrokModelClient include overwrites caller value and sent without reasoning; (2) missing GrokOAuthServiceTest + GrokTokenRefresherTest (task acceptance criterion); (3) live smoke with real creds unmet (needs interactive user login — deferred to user); (4) storage test makes real HTTP (stub it). Fix fork zz5qwcqqlab5 dispatched. Known-gap notes recorded: reasoning.effort shaping unimplemented (inert with shipped template — all thinking_level_map values null; if non-null map added later, resolver would emit chat-completions-style reasoning_effort instead of Responses reasoning.effort); content:null passthrough unverified until live smoke; auth.json cross-provider lock namespace (grok-auth vs codex-auth) is a pre-existing pattern, file-scoped lock would be safer follow-up.
- 2026-08-18: Implementation fork ymcsb7gm8352 complete @ 93a9a6ce9 — castor test 4627 OK, deptrac 0, phpstan 0, cs clean; testing skill + tests/AGENTS.md read.
- 2026-08-18: Reviewer REQUEST CHANGES (include bug, 2 missing test files, live smoke pending) → fix fork zz5qwcqqlab5 launched.

## Task workflow update - 2026-08-18T18:25:04.453Z
- Validation: castor test: 2545 OK (ConsumerSupervisorTest flaky under ParaTest, green on sequential retry, pre-existing); castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean; Focused Grok suite: 29 tests / 95 assertions OK
- Summary: Fix fork zz5qwcqqlab5 @ d9e2c688c resolved all 5 blockers: include guard (reasoning-conditional, caller-preserving) + 3-case test matrix; GrokTokenRefresherTest (success/omit-keeps-old/empty-keeps-old/failure-hint via Guzzle MockHandler); GrokOAuthServiceTest (Codex essentials mirrored); storage auto-refresh success path + network-free stubs; CodexOAuthProvider dual-provider docblock. Re-review 2: APPROVED — include guard spec-verbatim, test matrices complete, Guzzle seam ruled acceptable DI injection (league collaborators mechanism, CodexOAuthProviderTest precedent, zero production delta), no token leakage, no other wire-body changes. Validation: castor test green (one pre-existing flaky ConsumerSupervisor race under ParaTest, green on retry — unrelated), deptrac 0, phpstan 0, cs-check clean. REMAINING ACCEPTANCE GAP: live smoke with real creds — needs interactive user browser login (auth:grok), deferred to user post-PR or post-merge.
- 2026-08-18: Fix fork zz5qwcqqlab5 @ d9e2c688c — all 5 review blockers resolved, optional Guzzle test seam on GrokTokenRefresher (ruled acceptable in re-review).
- 2026-08-18: Re-review 2 APPROVED. Live smoke remains the only open acceptance item (needs user's interactive xAI login).

## Task workflow update - 2026-08-18T18:27:39.881Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (137.7s).
- Pushed task/grok-cli-provider-support to origin.
- branch 'task/grok-cli-provider-support' set up to track 'origin/task/grok-cli-provider-support'.
- Created PR: https://github.com/ineersa/agent-core/pull/409

## Task workflow update - 2026-08-18T18:58:50.028Z
- Moved CODE-REVIEW → IN-PROGRESS.
