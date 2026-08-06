# Widget slot priority ordering + TUI compact-header (pinned above editor)

## Goal

Deliver **two** things together:

1. **A general priority-ordering capability** for the TUI widget slot system: `TuiSlotRegistry::setWidget()` (and the `TuiExtensionContext` seam) gain an `order` field, and `getWidgetsByPlacement()` sorts by it. This lets **any** widget pin itself to a stable position within the above/below-editor region — a first-class fix for what Pi achieves only via a fragile re-registration hack (see Pi reference below).

2. **The compact-header bar** (the user-visible feature that motivated this): a permanent, fixed capability bar rendered directly above the editor separator, showing what is loaded in the session (prompts/skills/agents/mcp). It is the **first consumer** of the new priority mechanism — registered at the pinned (max) order so it is **guaranteed to render last in the above-editor merge block** (i.e. adjacent to the editor), no matter what other extensions place in `AboveEditor` or when.

**Why priority instead of a dedicated hardcoded row:** Pi upstream has *no* pinning API — its built-in chrome is hardcoded layout rows and its extension `aboveEditor` region is one merged block ordered by Map insertion order; the my-pi `compact-header` only "wins" by re-`setWidget`-ing on every `turn_end` (delete+set → key moves to Map end → renders last). That is a race-to-be-last, not real pinning. Hatfield currently mirrors Pi's weakness: `aboveEditorWidget` is one merged `LiveTextWidget` whose producer loops `getWidgetsByPlacement(AboveEditor)` in raw insertion order with **no priority**. This task makes pinning a real, reusable, guaranteed property of the slot system, then builds compact-header on top of it.

## Visual target (what compact-header should look like)

Rendered directly above the editor separator, e.g. (labels dim/muted, values accent-colored):

```
prompts   /new  /resume  /rename  /compact  /hotkeys  ...
skills    skill:castor  skill:testing  skill:task-workflow  ...
agents    5 available • /agents-live
available architect  browser  researcher  reviewer  scout
mcp       ✓ browser (3): connected  ○ jetbrains-index (18): cached
─────────────────────────────────────────────────────────
<the existing editorSepWidget renders directly below this>
```

- Label column fixed width = **9** (matches Pi; fits `available`).
- Each section renders only when non-empty.
- Items wrap to subsequent indented lines at terminal width (re-wrap per item so color codes survive line breaks).
- Empty bar (nothing loaded yet / draft) → render **zero lines** (no blank vertical space) until populated.
- Trailing `─`: **rely on the existing `editorSepWidget`** (do NOT render the bar's own separator — avoids a double line). The bar is already visually delimited by sitting directly above `editorSepWidget`. (Pi renders its own `─` because its editor separator is separate; Hatfield already has one.)

## Pi reference (the behavior to mirror + the corrected understanding)

**Read these two files first** (derivative source of truth for the *look* + MCP icons):

- **`/home/ineersa/claw/my-pi/packages/extensions/extensions/compact-header.ts`** — the widget (JS).
- **`/home/ineersa/claw/my-pi/packages/extensions/extensions/mcp-shared-state.ts`** — MCP status→icon/status mapping (`getMcpServerStatus()`).

**Pi upstream core mechanics (from a pi-mono scout — the *corrected* understanding, do NOT copy):**

- Pi's `setWidget` API (`packages/coding-agent/src/core/extensions/types.ts`) accepts only `{ placement: "aboveEditor" | "belowEditor" }`. **No priority / pin / z-index / order field exists upstream.**
- `InteractiveMode` (`packages/coding-agent/src/modes/interactive/interactive-mode.ts`) stores above-editor widgets in **one `Map<string, Component>`** (`extensionWidgetsAbove`) and renders them in **Map insertion order** inside a single `widgetContainerAbove` child (`renderWidgetContainer`, ~1905–1926).
- Built-in chrome (header/chat/status/editor/footer) are **hardcoded `ui.addChild(...)` rows** (~653–662), **not** `setWidget` widgets. So Pi has *two separate mechanisms* — hardcoded chrome vs merged extension slot — and **no unified pinned-widget concept**.
- Widgets are **NOT cleared per turn**. They are cleared only on **session invalidate** (`resetExtensionUI` → `clearExtensionWidgets`, ~1861–1876) via `beforeSessionInvalidate` (session switch/reload/dispose).
- **Therefore my-pi's `turn_end` re-registration is an ordering hack, not a survival mechanism:** `setWidget` does `map.delete(key); map.set(key, …)` → the re-inserted key moves to the **end** of the Map → it renders **last** in the merged block → **closest to the editor**. "Sticky" really means "last in merge order." If another extension re-registered *after* compact-header's handler ran, it would slip in below. It is a race-to-be-last, not a guarantee.

**This task's job is to turn that hack into a real, guaranteed priority property in Hatfield.** Port the *look* and *data* from Pi, but implement pinning via explicit ordering, not re-registration.

**MCP icons/status (from `mcp-shared-state.ts` — adapt to Hatfield DTO):**
- `✓` connected (with tool count) → `✓ ${name} (${count}): connected`
- `✗` failed → `✗ ${name}: failed` (optionally append short `$errorMessage`)
- `○` cached (metadata present, not live-connected) → `○ ${name} (${count}): cached`
- `⏸` disabled (Hatfield DTO may not carry disabled — see data sources)
- catalog `null` (broker hasn't written yet) → **omit the whole `mcp` line** (avoid blank-line churn)

## Hatfield current state (what exists today)

- **`src/Tui/Layout/TuiSlotRegistry.php`** — central slot store. `setWidget(string $key, TuiWidget $widget, WidgetPlacementEnum $placement = AboveEditor): void`; storage `array<string, array{widget: TuiWidget, placement: WidgetPlacementEnum}>`; `getWidgetsByPlacement(WidgetPlacementEnum): array` returns widgets in **raw insertion (PHP array) order, no priority**. Same key replaces (order unchanged). **No pinned/reserved/priority concept exists today** — this task adds it.
- **`src/Tui/Widget/WidgetPlacementEnum.php`** — `AboveEditor = 'above_editor'`, `BelowEditor = 'below_editor'`. Priority is orthogonal; the enum stays unchanged.
- **`src/Tui/Extension/TuiExtensionContext.php`** + **`SlotBasedTuiExtensionContext.php`** — `setWidget($key, ?$content, $placement)` delegates to the registry. This is the TUI extension seam (`src/Tui/Extension/`), **NOT** the public `src/CodingAgent/ExtensionApi/` package, so adding an `order` param here does not touch the public compatibility surface.
- **`src/Tui/Screen/ChatScreen.php`** — `mount()` adds widgets in fixed order (top→bottom). Current `mount()` order (source of truth — architecture doc is slightly stale, omits one widget):
  1. `topMarginWidget`
  2. `headerWidget` (ASCII logo via `HeaderWidget`)
  3. `headerSepWidget`
  4. `loadedResourcesWidget` ← the *transient* startup block (`LoadedResourcesWidget`); populated once on first tick, skipped on resume
  5. `transcriptWidget`
  6. `pendingWidget`
  7. `workingWidget`
  8. `statusPanelWidget`
  9. `aboveEditorWidget` ← **single merged `LiveTextWidget`**: producer loops `getWidgetsByPlacement(AboveEditor)` and concatenates each `TuiWidget::render()` lines with `implode("\n")`. **This is where compact-header will appear, last in the sorted order.**
  10. `editorSepWidget`
  11. `promptEditor->getWidget()` (the editor)
  12. `belowEditorWidget`
  13. `footerSepWidget`
  14. `footerWidget`
- **Key consequence of Option B:** compact-header does **NOT** need a new `ChatScreen` mount row. It flows through the existing `aboveEditorWidget` merge (#9), which now sorts by `order`. compact-header registers at the pinned order → renders last in the merge → sits directly above `editorSepWidget`. Extensions placing widgets at default order render **above** compact-header (closer to the status panel). That is the correct, guaranteed pin.
- **Built-in permanent widget pattern (header/footer/loaded-resources):** instantiate a `TuiWidget` renderable, wrap in a `LiveTextWidget` producer. compact-header is **not** a `ChatScreen`-owned row — it is a built-in widget registered into the registry by `CompactHeaderRegistrar` (so it is permanent via the registrar, not via hardcoded mount).
- **Refresh / invalidation:** `LiveTextWidget` caches output keyed on `revision × columns × rows`; resize → automatic re-render; data change → `invalidate()`. `ChatScreen::refresh()` **already invalidates `aboveEditorWidget`** (among others), and `FooterStateListener` calls `$screen->refresh()` every tick — so compact-header in the merge gets free per-tick freshness once the snapshot is updated. (Note: extension `setWidget` has **no auto-invalidate** hook — the registrar must invalidate `aboveEditorWidget` / rely on `refresh()` + `requestRender()` after snapshot changes.)
- **`TuiWidget` interface:** `src/Tui/Widget/TuiWidget.php` — `render(TuiRenderContext $context): list<string>`. `TuiRenderContext` carries `terminalWidth`, `terminalHeight`, `theme`.
- **`LiveTextWidget`:** `src/Tui/Widget/LiveTextWidget.php` — `__construct(callable $producer /* (RenderContext $symfonyCtx): string */, bool $truncate = false)`.
- **Theme:** `TuiTheme` (`accent()`, `muted()`, `color(ThemeColorEnum, $text)`). Tokens: `ThemeColorEnum::Muted` (dim labels), `ThemeColorEnum::Accent` (values), `ThemeColorEnum::Separator` (the `─`). See `src/Tui/Theme/`.
- **Stable sort:** Symfony 8.1 ⇒ PHP 8.x ⇒ `usort`/`array_multisort` are **stable** since PHP 8.0, so equal `order` values preserve insertion order. Rely on this for the tiebreak (document it).

## Data sources (Hatfield services — all autowireable, all read-only/safe from the TUI process)

A new snapshot provider aggregates these. **Do NOT call `McpConnectionManager` from the TUI** (wrong process — it runs in the MCP messenger consumer and would block STDIO clients).

| Header line | Hatfield source today | Method | Notes |
|---|---|---|---|
| `prompts` | `Ineersa\Tui\Command\SlashCommandRegistry` | `allMetadata(): list<CommandMetadata>` | Unified registry of all slash commands. Populated by `TuiListenerRegistrar`s during `InteractiveMode` boot → **must snapshot on/after first tick**, not in constructor. **No `source` field on `CommandMetadata`** — see open decisions. |
| `skills` | `Ineersa\CodingAgent\Skills\SkillDiscovery` | `discover(): list<SkillDefinition>` | Filesystem scan, cached on instance after first call. Format names with `skill:` prefix (Pi convention). `SkillDefinition` has `name`, `description`, `modelInvocationEnabled`. |
| `agents` (`N available`) | `Ineersa\CodingAgent\Agent\Definition\AgentDefinitionDiscovery` | `discover(): AgentDefinitionCatalog` → `enabled(): list<AgentDefinitionDTO>` | Count = `count(enabled())`. Use `enabled()` (not `all()`). `AgentDefinitionCatalog` is a factory service (`factory: ['@AgentDefinitionDiscovery', 'discover']`). |
| `agents` (`available <names>`) | same | `enabled()` names | Sort names; format duplicates as `name×N` (Pi convention). |
| `mcp` | `Ineersa\CodingAgent\Mcp\Catalog\McpToolCatalogStoreInterface` (alias → `SessionFileMcpToolCatalogStore`) | `read(string $runId): ?McpToolCatalogDTO` | `runId === TuiSessionState::$sessionId`. Returns `McpToolCatalogDTO` with `servers: array<string, McpServerCatalogEntryDTO>`; each entry: `serverName`, `transport`, `status` (`McpServerCatalogStatusEnum`: connected\|failed), `errorMessage`, `tools[]` (count via `count($entry->tools)`). **`null` until broker writes `.hatfield/sessions/<id>/mcp-tools.json`** → omit the `mcp` line. |

Closest existing aggregate pattern: `src/CodingAgent/Runtime/LoadedResources/LoadedResourcesSummaryBuilder.php` (skills/prompts/agents/themes/extensions, **omits** slash registry + MCP) + `src/Tui/Listener/LoadedResourcesStartupRegistrar.php` (deferred first-tick population pattern).

## Implementation plan (detailed, file by file)

Implementor: load the `testing` skill and read `tests/AGENTS.md` before writing tests. This is a **purely virtual / local** TUI feature (slot ordering + widget render) → proof at the **virtual layer** (`castor test`). No tmux/controller-replay required.

### Part A — General priority ordering (the reusable capability)

**A1. `src/Tui/Layout/TuiSlotRegistry.php`**
- Change storage to `array<string, array{widget: TuiWidget, placement: WidgetPlacementEnum, order: int}>`.
- `setWidget(string $key, TuiWidget $widget, WidgetPlacementEnum $placement = WidgetPlacementEnum::AboveEditor, int $order = 0): void` — store `order`. Same key replaces widget **and** updates its `order`.
- `getWidgetsByPlacement(WidgetPlacementEnum $placement): array` — return widgets **sorted by `order` ascending** (lower order rendered first = top of block; higher order rendered last = editor-adjacent). Use a **stable** sort (PHP 8.0+ `usort` is stable) so equal orders preserve insertion order. Document this in a docblock comment (why stable matters: insertion order is the deterministic tiebreak).
- Expose named order constants so pinning is discoverable (not magic numbers). Recommend:
  ```php
  /** Default order: insertion order among equals. */
  public const ORDER_DEFAULT = 0;
  /** Pin a widget to render last within its placement (adjacent to editor for AboveEditor). */
  public const ORDER_PINNED_LAST = PHP_INT_MAX;
  ```
  (Constant names/direction are an open decision — see Open decisions.)

**A2. `src/Tui/Extension/TuiExtensionContext.php` + `SlotBasedTuiExtensionContext.php`**
- Thread `int $order = 0` through `setWidget($key, ?$content, $placement = AboveEditor, $order = 0)`. Nullable content (removal) ignores order. This is the TUI extension seam (`src/Tui/Extension/`), **not** `src/CodingAgent/ExtensionApi/`, so no public-compat-package concern. (Still: update any docs/examples that show the old signature.)

**A3. Tests — `tests/Tui/Layout/TuiSlotRegistryTest.php` (extend)**
- Assert `getWidgetsByPlacement` returns widgets sorted by `order` ascending.
- Assert **stable tiebreak**: equal `order` preserves insertion order (register A(0), B(0), C(0) → [A,B,C]; register A(5), B(0) → [B,A]).
- Assert re-setting an existing key updates its `order` and re-sorts (A(0), B(5) → [A,B]; re-set A(10) → [B,A]).
- Assert ordering applies to `BelowEditor` too (symmetry).
- Update any existing test that asserts raw insertion order for distinct widgets (e.g. `testMultipleWidgetsInSamePlacementOrder`) — equal-order behavior is unchanged, but add explicit priority cases.

### Part B — compact-header (first pinned consumer)

**B1. New renderable: `src/Tui/CompactHeader/CompactHeaderWidget.php` (implements `TuiWidget`)**
- Holds a mutable `?CompactHeaderSnapshot $snapshot = null`.
- `setSnapshot(?CompactHeaderSnapshot $snapshot): void`.
- `render(TuiRenderContext $context): list<string>`:
  - If snapshot null or all sections empty → `return []` (zero lines).
  - For each non-empty section, emit `wrapLabel(dim(label), accentItems)`; label column width = 9; wrap items across lines at `$context->terminalWidth` (re-wrap per item so color codes survive line breaks).
  - **No trailing `─`** (rely on `editorSepWidget`).
- Reuse an existing ANSI-aware width/truncate util if present (check what `FooterBarWidget` uses, e.g. Symfony `AnsiUtils`); do not reinvent.

**B2. New DTOs: `src/Tui/CompactHeader/CompactHeaderSnapshot.php` + `McpServerHeaderEntry.php`**
- Immutable DTOs: `prompts: list<string>`, `skills: list<string>` (prefix `skill:` in the provider or widget — pick one, document), `agentCount: int`, `agentNames: list<string>`, `mcpServers: list<McpServerHeaderEntry>` (`name`, `icon`, `toolCount:?int`, `statusText`). Add value equality for change detection (compare fields).

**B3. New `src/Tui/CompactHeader/CompactHeaderSnapshotProvider.php` (DI service)**
- Depends on `SlashCommandRegistry`, `SkillDiscovery`, `AgentDefinitionDiscovery`, `McpToolCatalogStoreInterface`.
- `build(string $sessionId): CompactHeaderSnapshot` — aggregate the four sources per the data-source table. MCP: `$this->catalogStore->read($sessionId)`; `null` → empty `mcpServers`. Map DTO status→icon/text: connected→`✓ (${count}): connected`, failed→`✗: failed`, else→`○ (${count}): cached`.

**B4. New `src/Tui/Listener/CompactHeaderRegistrar.php` (implements `TuiListenerRegistrar`, tag `app.tui_listener` via `_instanceof`)**
- Owns one `CompactHeaderWidget` instance.
- Register a tick handler that:
  1. On **first tick** (regardless of `$state->resuming`) → build snapshot, `$widget->setSnapshot(...)`, register into the slot at pinned order:
     `$context->extensionContext()->setWidget('compact-header', $this->widget, WidgetPlacementEnum::AboveEditor, TuiSlotRegistry::ORDER_PINNED_LAST);`
     then `$screen->refresh()` (invalidates `aboveEditorWidget`) + `$tui->requestRender()`.
  2. **Throttled rebuild** (~2–3s, matching Pi's 2s subagent TTL) → rebuild snapshot; only if changed (DTO value equality) → `$widget->setSnapshot(...)` + invalidate `aboveEditorWidget` (via `$screen->refresh()`) + `$tui->requestRender()`. This picks up late MCP catalog writes and agent/skill discovery without needless re-renders.
- This **replaces Pi's `turn_end` re-registration**: Hatfield has no `turn_end` lifecycle event; the pinned `order` guarantees position (no need to re-assert last-in-Map), and the tick throttle keeps data fresh. Registering the widget once (first tick) is sufficient because the registry entry persists for the session; only the snapshot is refreshed.

**B5. `ChatScreen` — NO structural mount change needed.** compact-header renders through the existing `aboveEditorWidget` merge (#9), now sorted. (Optional: if `setWidget` should auto-invalidate `aboveEditorWidget`, add a registry→screen hook — but simpler to let the registrar's `$screen->refresh()` handle it. Keep `ChatScreen` untouched unless the invalidate ergonomics demand it.)

**B6. DI** — `CompactHeaderSnapshotProvider` + `CompactHeaderRegistrar` autowire normally (no manual `services.yaml` beyond the existing `_instanceof: TuiListenerRegistrar` tag). Confirm `McpToolCatalogStoreInterface` is aliased to `SessionFileMcpToolCatalogStore` in `config/services.yaml`.

### Part C — Docs
- Update `docs/tui-architecture.md`:
  - **Slot table:** add `order` to `setWidget(key, ?TuiWidget, placement, order=0)`; document priority ordering (lower order = top of merge block; higher = editor-adjacent; stable insertion-order tiebreak); mention `ORDER_DEFAULT` / `ORDER_PINNED_LAST`.
  - **Layout diagram / widget tree:** note compact-header is a registry entry at pinned order (not a dedicated row) — it appears at the bottom of the above-editor region. Fix the existing 13→14 doc drift (`loadedResourcesWidget` is missing); compact-header does **not** add a new row (still 14, since it lives in the merge).
  - Note the new general pinning capability is available to any widget.

## Open design decisions (flag for parent before/as you implement)

1. **Ordering field naming + direction (confirm):** Recommend a dedicated `int $order` (ascending render; lower = top of block, higher = editor-adjacent) with named constants `ORDER_DEFAULT=0` / `ORDER_PINNED_LAST=PHP_INT_MAX`. Alternative: reuse Hatfield's existing `priority` vocabulary (listener priority 100/98/50; footer segment priority) where **higher priority = processed/rendered first/top** — under that convention compact-header would pin with **min** priority, which is less intuitive for "pin to editor." **Recommend `order`** to avoid colliding semantics with listener priority and to make "high = editor-adjacent" read naturally; confirm with parent.
2. **Pinned-order constant exposure (confirm):** expose `TuiSlotRegistry::ORDER_PINNED_LAST` / `ORDER_DEFAULT` (recommended — discoverable general capability) vs magic `PHP_INT_MAX`.
3. **Trailing separator:** rely on `editorSepWidget` (recommended — no double line).
4. **`prompts` line scope:** show (a) full `SlashCommandRegistry` list, or (b) only prompt-template commands to mirror Pi's source-filtered line. Hatfield has no `source` on `CommandMetadata`. **Recommend:** full registry (most useful overview). Adding `source` is a separate follow-up, out of scope here.
5. **`agents` hint command:** Pi shows `/agents or /subagents-status`. Hatfield core has `/agents-live` and `/agents-main`, not `/agents`. **Recommend:** show the hint that matches a real command (e.g. `N available • /agents-live`) or make it generic (`N available`). Confirm which command exists at implement time.
6. **Relationship to `LoadedResourcesWidget`:** keep both (compact-header = permanent pinned summary; loaded-resources = expandable ctrl+r detail block with source paths). **Recommend:** keep both. (Follow-up: maybe collapse loaded-resources later.)
7. **Resume sessions:** `LoadedResourcesStartupRegistrar` skips population on resume. compact-header should populate on first tick **regardless of resume** (it is permanent). Confirm.
8. **Auto-invalidate ergonomics:** whether `setWidget` should invalidate the relevant merge widget automatically (nicer extension DX) vs leaving the registrar responsible. **Recommend:** leave as-is (registrar calls `refresh()`) to keep this task scoped; auto-invalidate is a separate DX improvement.

## Non-goals / out of scope

- Do NOT add a `source` field to `CommandMetadata` (separate concern).
- Do NOT replace or remove `LoadedResourcesWidget`.
- Do NOT add a dedicated hardcoded `compactHeaderWidget` mount row (that was Option A; this task is Option B — pinning via the registry).
- Do NOT call `McpConnectionManager` from the TUI process.
- Do NOT add backward-compat shims for the old 3-arg `setWidget` (active development; update callers/tests instead).
- No `--plain-icons`/plain-icons setting work (Pi bundles that into the same extension; not needed here).

## Files (expected)

New:
- `src/Tui/CompactHeader/CompactHeaderWidget.php`
- `src/Tui/CompactHeader/CompactHeaderSnapshot.php` (DTO)
- `src/Tui/CompactHeader/McpServerHeaderEntry.php` (DTO)
- `src/Tui/CompactHeader/CompactHeaderSnapshotProvider.php`
- `src/Tui/Listener/CompactHeaderRegistrar.php`
- `tests/Tui/Layout/TuiSlotRegistryPriorityTest.php` (or extend `TuiSlotRegistryTest`)
- `tests/Tui/CompactHeader/CompactHeaderWidgetTest.php` (virtual render proof)
- `tests/Tui/CompactHeader/CompactHeaderSnapshotProviderTest.php`
- `tests/Tui/CompactHeader/CompactHeaderPinnedOrderTest.php` (virtual: compact-header renders last in the above-editor merge regardless of other AboveEditor widgets)

Modified:
- `src/Tui/Layout/TuiSlotRegistry.php` (order storage + sort + constants)
- `src/Tui/Extension/TuiExtensionContext.php` + `SlotBasedTuiExtensionContext.php` (thread `order`)
- `docs/tui-architecture.md` (slot table + layout diagram + fix 13→14 drift)
- Possibly `tests/Tui/Layout/TuiSlotRegistryTest.php` and `tests/Tui/Extension/SlotBasedTuiExtensionContextTest.php` (new param)

## Pi reference files (read these first)
- `/home/ineersa/claw/my-pi/packages/extensions/extensions/compact-header.ts`
- `/home/ineersa/claw/my-pi/packages/extensions/extensions/mcp-shared-state.ts`
- (Upstream mechanics, for understanding only): `/home/ineersa/claw/pi-mono/packages/coding-agent/src/modes/interactive/interactive-mode.ts` (`extensionWidgetsAbove`, `renderWidgetContainer`, `resetExtensionUI`) and `.../core/extensions/types.ts` (`WidgetPlacement`).

## Acceptance criteria

**Priority mechanism (general capability):**
- `TuiSlotRegistry::setWidget()` accepts an `int $order` (default `ORDER_DEFAULT`); `getWidgetsByPlacement()` returns widgets sorted by `order` ascending; equal orders preserve insertion order (stable); applies symmetrically to `AboveEditor` and `BelowEditor` — proven by virtual tests.
- Re-setting an existing key updates its widget and its `order` and re-sorts accordingly — proven by test.
- `ORDER_PINNED_LAST` (or agreed constant) is exposed and documented; any widget can pin itself last in its placement.
- `TuiExtensionContext::setWidget()` threads `order` through to the registry (TUI extension seam, not the public `ExtensionApi` package).

**compact-header (pinned consumer):**
- compact-header registers at pinned order and renders **last in the above-editor merge block** (adjacent to `editorSepWidget`) regardless of what/when other extensions place in `AboveEditor` — proven by a virtual test registering a lower-order widget and asserting compact-header's lines appear after it in the merged render.
- The bar displays prompts (slash commands), skills (skill:-prefixed), agents (count + names), and mcp (per-server icon + tool count + status) sections, each only when non-empty.
- MCP status maps DTO enum to icons/text: connected→✓ (count), failed→✗, cached/pending-metadata→○ (count); null catalog omits the mcp line without leaving blank lines.
- Empty/unpopulated snapshot renders zero lines (no blank vertical space) until first-tick population.
- Snapshot is populated on the first tick **regardless of resume** and refreshed on a throttle (~2–3s) with change detection so MCP/agent changes appear without needless re-renders; participates in `ChatScreen::refresh()` via `aboveEditorWidget` invalidation.
- Labels use Muted theme color and values use Accent; label column width 9; items wrap at terminal width with per-item color preserved; resize re-renders correctly.

**Cross-cutting:**
- No new backward-compat shims; no calls to `McpConnectionManager` from the TUI; the four named services are the only data sources.
- `docs/tui-architecture.md` updated (slot table gains `order`; layout diagram notes compact-header as a pinned registry entry; fix the 13-vs-14 widget doc drift).
- Passes focused Castor QA: `castor test` (virtual TUI proof) and `castor phpstan` + `castor deptrac` + `castor cs-check` for the touched layers (TuiLayout/TuiExtension/TuiCompactHeader/TuiListener).

## Workflow metadata
Status: DONE
Branch: task/2026-07-02-tui-compact-header-above-editor
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-02-tui-compact-header-above-editor
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/253
PR Status: merged
Started: 2026-07-02T23:09:20.447Z
Completed: 2026-07-03T18:11:10.385Z

## Work log
- Created: 2026-07-02T18:26Z (Option A — dedicated row)
- Revised: 2026-07-02 (→ Option B: general priority ordering in `TuiSlotRegistry`/`TuiExtensionContext`; compact-header is the first pinned consumer via `ORDER_PINNED_LAST`, rendered through the existing `aboveEditorWidget` merge — no dedicated mount row. Rationale: Pi upstream has no pin API; my-pi `turn_end` re-register is an ordering hack (delete+set → Map end → last in merge). This makes pinning a real, guaranteed, reusable property instead of a race-to-be-last.)

## Task workflow update - 2026-07-02T23:09:20.447Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-07-02-tui-compact-header-above-editor.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-07-02-tui-compact-header-above-editor.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-07-02-tui-compact-header-above-editor.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-07-02-tui-compact-header-above-editor.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-07-02-tui-compact-header-above-editor.

## Task workflow update - 2026-07-02T23:10:32.484Z
- task-start: claimed task → IN-PROGRESS. Branch task/2026-07-02-tui-compact-header-above-editor. Worktree: /home/ineersa/projects/agent-core-worktrees/2026-07-02-tui-compact-header-above-editor
- Orchestrator-resolved open decisions (fork to follow, not re-decide): (1) ordering field = `int $order`, ascending render (low=top of merge block, high=editor-adjacent); constants ORDER_DEFAULT=0, ORDER_PINNED_LAST=PHP_INT_MAX on TuiSlotRegistry. (2) trailing separator: rely on existing editorSepWidget, render NO own ─. (3) prompts line = PROMPT TEMPLATES via PromptTemplateCatalogInterface::allPromptTemplateCommands() (matches Pi semantics; sidesteps missing CommandMetadata::source). skills line = SkillDiscovery. (4) keep LoadedResourcesWidget. (5) populate compact-header on first tick regardless of resume. (6) no auto-invalidate hook; registrar calls screen->refresh().
- Key impl facts for fork: TuiRuntimeContext exposes $screen (ChatScreen), not registry directly; reach registry/extension context via $context->screen->extensionContext() (ChatScreen:597) / ->registry() (ChatScreen:592). Registrar registers widget via extensionContext()->setWidget('compact-header', $widget, AboveEditor, ORDER_PINNED_LAST). aboveEditorWidget producer already invalidates via ChatScreen::refresh() each tick (FooterStateListener).

## Task workflow update - 2026-07-03T16:32:48.390Z
- review-pass-1: Reviewer verdict APPROVE WITH SUGGESTIONS. Static QA green (phpstan 0, deptrac 0 violations, cs-check clean, castor test 4041 pass at virtual layer). E2E BLOCKER found: TuiCompactHeaderE2eTest AND known-good TuiStartupSnapshotTest both crash ("can't find pane" → agent dies after logo). Root cause reproduced by direct boot: Uncaught RuntimeException in event loop — SessionFileMcpToolCatalogStore::read('') throws 'MCP catalog run ID must not be empty'. CompactHeaderRegistrar fires on first tick during DRAFT session (sessionId === ''), passes empty id → read('') throws → uncaught in Revolt event loop → whole agent crashes. Virtual suite missed it (stubbed catalogStore with non-empty id). CRITICAL regression: registrar auto-registers for ALL TUI boots, so it breaks every TUI E2E.
- Actionable review findings to fix: (1) CRITICAL empty-sessionId crash — provider must guard read() on empty id (draft→omit mcp line per spec); registrar must wrap build in try/catch+logger so a widget failure NEVER crashes the agent loop. (2) DOC widget tree still says (13 widgets), missing loadedResourcesWidget — required by plan Part C. (3) provider unreachable 'cached' else branch (enum exhaustive CONNECTED/FAILED) — remove. (4) depfile TuiCompactHeader unused TuiTheme grant — drop. (5) formatAgentNameItems impossible dedup (enabled() keyed by unique name) — simplify. (6) registrar redundant instanceof guard — simplify. (7) add @see ORDER_DEFAULT + cadence comment.

## Task workflow update - 2026-07-03T16:40:09.743Z
- Updated PR Status: approved (post-fix); moving to CODE-REVIEW
- Validation: castor test => 4044 tests OK (13033 assertions); castor test:tui --filter=TuiCompactHeaderE2eTest => OK (1 test, 2 assertions); castor test:tui --filter=TuiStartupSnapshotTest => OK (1 test, 3 assertions); castor phpstan => 0 errors; castor deptrac => 0 violations; castor cs-check => 0 files fixed
- review-pass-1 RESOLVED: Fix fork b47526faf addressed all actionable findings + the critical E2E blocker. CRITICAL fix: CompactHeaderSnapshotProvider now guards read() on empty sessionId (draft→omit mcp line); CompactHeaderRegistrar wraps both tick-path builds in try/catch with LoggerInterface so a widget failure NEVER crashes the agent event loop. Also removed: unreachable 'cached' MCP branch, unused TuiTheme deptrac grant, impossible agent-name ×N dedup; fixed redundant instanceof guard; added @see ORDER_DEFAULT + cadence doc. Docs: widget tree 13→14 + loadedResourcesWidget + above-editor pinning note.
- Doc-consistency micro-fix 6a52e7ee2: mount() API table row 13→14 to match widget tree (same file). Branch is now 3 commits: a2ff6c43d (impl), b47526faf (review fixes), 6a52e7ee2 (doc). HEAD=6a52e7ee2.
- FINAL VALIDATION (Castor, on HEAD 6a52e7ee2): castor test=4044 pass; castor test:tui --filter=TuiCompactHeaderE2eTest=OK(1 test,2 assertions); castor test:tui --filter=TuiStartupSnapshotTest=OK(1 test,3 assertions); castor test:tui (both)=OK(2 tests,5 assertions) — the regression that crashed ALL TUI E2E is fully resolved; castor phpstan=0 errors; castor deptrac=0 violations; castor cs-check=clean. tmux 3.4 available, E2E actually executed (not skipped). Task does NOT touch LLM/provider code → test:llm-real not required.
- Reviewer verdict: APPROVED (post-fix). All pass-1 findings (DOC/EDGE-CASE-DEAD-CODE/SIMPLIFY/DEAD-CODE/REDUNDANT/NTH) addressed; E2E proof verdict PASSES locally (reviewer's E2E verdict was contingent on this local run, now confirmed green). No re-review needed — changes were surgical and exactly targeted the findings.

## Task workflow update - 2026-07-03T16:41:54.945Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (80.0s).
- Pushed task/2026-07-02-tui-compact-header-above-editor to origin.
- branch 'task/2026-07-02-tui-compact-header-above-editor' set up to track 'origin/task/2026-07-02-tui-compact-header-above-editor'.
- Created PR: https://github.com/ineersa/agent-core/pull/253

## Task workflow update - 2026-07-03T16:42:04.106Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/253
- Updated PR Status: open
- CODE-REVIEW: move_task succeeded. Deterministic castor check passed (80.0s). Branch pushed to origin. PR created: https://github.com/ineersa/agent-core/pull/253

## Task workflow update - 2026-07-03T17:38:37.099Z
- Updated PR Status: open (beautify iteration pushed)
- beautify iteration (commit 703e5d2b2, pushed to PR #253): per user visual review feedback — (1) added dim │ separator + clear spacing between labels and values (label col 9 + │ + space = value start col 11); (2) theme colors matching header: labels Accent(cyan, same as header), prefixes muted, names accent; (3) MCP always-loaded vs specific differentiation via McpConfigLoader availability: global=green ✓, specific=blue ◈ (MarkdownLink), failed=red ✗; (4) simplified agents to just names (dropped 'N available • /agents-live' count line + separate 'available' names line). DTO changes: McpServerHeaderEntry now (name,toolCount,isConnected,isGlobal); CompactHeaderSnapshot dropped agentCount. Provider: +McpConfigLoader 5th dep for availability map (try/catch degrade). depfile: +AppMcpConfig, +TuiTheme (re-added, widget imports ThemeColorEnum).
- VALIDATION: castor test=4046 OK; castor test:tui --filter=TuiCompactHeaderE2eTest=OK(1 test,2 assertions); castor phpstan=0; castor deptrac=0 violations; castor cs-check=clean. Real render sample verified with oh-p-dark palette (cyan labels, green ✓ global, blue ◈ specific, red ✗ failed). test:llm-real not required.

## Task workflow update - 2026-07-03T17:59:35.248Z
- Updated PR Status: open (style iteration 2 pushed — commit 5fe59a6c1)
- style iteration 2 (commit 5fe59a6c1, pushed to PR #253): per user feedback — (1) labels switched Accent(cyan)→MarkdownHeading(orange #d19a66) to match [Skills] in LoadedResourcesWidget; (2) values changed to white italic via new Style(italic:true) helper (italicValue()); (3) removed skill: prefix entirely (skills now render bare names); (4) prompts render as solid /prompt unit (one italic string, no muted/accent split). MCP status icons keep semantic colors (✓/◈/✗), server names italic, counts muted. Added use Symfony\Component\Tui\Style\Style import (no depfile change — AnsiUtils already uses same vendor layer).
- TEST FALLOUT from skill: prefix removal (required, not scope creep): CompactHeaderWidgetTest (assert NOT skill:), TuiCompactHeaderE2eTest (skill:e2e-compact-header → e2e-compact-header), CompactHeaderPinnedOrderTest (skill:pinned-skill → pinned-skill), CompactHeaderRegistrarTest (skill:reg-skill → reg-skill).
- VALIDATION: castor test=4046 OK/13051 assertions; castor test:tui --filter=TuiCompactHeaderE2eTest=OK(1 test,2 assertions) re-confirmed on final HEAD; castor phpstan=0; castor deptrac=0 violations; castor cs-check=clean. Live render verified with oh-p-dark palette: orange labels, white italic values.

## Task workflow update - 2026-07-03T18:04:21.710Z
- Updated PR Status: open (style iteration 3 pushed — commit fb924aad6)
- style iteration 3 (commit fb924aad6, pushed to PR #253): per user feedback — values changed from white-italic to blue non-italic (ThemeColorEnum::MarkdownLink #5b9bf5, same token as the ◈ MCP icon). Replaced all 4 italicValue() call sites (prompts/skills/agents/mcp names) with $theme->color(ThemeColorEnum::MarkdownLink, ...). Removed now-dead italicValue() helper + use Symfony\Component\Tui\Style\Style import. Orange MarkdownHeading labels unchanged. cs-fix applied minor formatting to agents array_map block.
- VALIDATION: castor test=4046 OK; castor test:tui --filter=TuiCompactHeaderE2eTest=OK; castor phpstan=0; castor deptrac=0; castor cs-check=clean. No test changes needed (tests assert plain-text names, not color/italic). 1 file changed: src/Tui/CompactHeader/CompactHeaderWidget.php (+4/−10).

## Task workflow update - 2026-07-03T18:11:10.385Z
- Moved CODE-REVIEW → DONE.
- Merged task/2026-07-02-tui-compact-header-above-editor into integration checkout.
- Merge made by the 'ort' strategy.
 depfile.yaml                                       |  21 ++
 docs/tui-architecture.md                           |   9 +-
 src/Tui/CompactHeader/CompactHeaderSnapshot.php    |  48 +++
 .../CompactHeaderSnapshotProvider.php              |  83 ++++++
 src/Tui/CompactHeader/CompactHeaderWidget.php      | 156 ++++++++++
 src/Tui/CompactHeader/McpServerHeaderEntry.php     |  24 ++
 src/Tui/Extension/SlotBasedTuiExtensionContext.php |   7 +-
 src/Tui/Extension/TuiExtensionContext.php          |   3 +-
 src/Tui/Layout/TuiSlotRegistry.php                 |  30 +-
 src/Tui/Listener/CompactHeaderRegistrar.php        | 101 +++++++
 .../CompactHeader/CompactHeaderPinnedOrderTest.php |  78 +++++
 .../CompactHeaderSnapshotProviderTest.php          | 332 +++++++++++++++++++++
 .../Tui/CompactHeader/CompactHeaderWidgetTest.php  | 113 +++++++
 tests/Tui/E2E/TuiCompactHeaderE2eTest.php          | 146 +++++++++
 .../Extension/SlotBasedTuiExtensionContextTest.php |  12 +
 tests/Tui/Layout/TuiSlotRegistryPriorityTest.php   |  90 ++++++
 tests/Tui/Listener/CompactHeaderRegistrarTest.php  | 203 +++++++++++++
 17 files changed, 1443 insertions(+), 13 deletions(-)
 create mode 100644 src/Tui/CompactHeader/CompactHeaderSnapshot.php
 create mode 100644 src/Tui/CompactHeader/CompactHeaderSnapshotProvider.php
 create mode 100644 src/Tui/CompactHeader/CompactHeaderWidget.php
 create mode 100644 src/Tui/CompactHeader/McpServerHeaderEntry.php
 create mode 100644 src/Tui/Listener/CompactHeaderRegistrar.php
 create mode 100644 tests/Tui/CompactHeader/CompactHeaderPinnedOrderTest.php
 create mode 100644 tests/Tui/CompactHeader/CompactHeaderSnapshotProviderTest.php
 create mode 100644 tests/Tui/CompactHeader/CompactHeaderWidgetTest.php
 create mode 100644 tests/Tui/E2E/TuiCompactHeaderE2eTest.php
 create mode 100644 tests/Tui/Layout/TuiSlotRegistryPriorityTest.php
 create mode 100644 tests/Tui/Listener/CompactHeaderRegistrarTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-07-02-tui-compact-header-above-editor.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-07-02-tui-compact-header-above-editor.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Final HEAD: fb924aad6 (3 style iterations on top of review-fix b47526faf); castor test: OK 4046 tests / 13051 assertions; castor test:tui --filter=TuiCompactHeaderE2eTest: OK (1 test, 2 assertions) — real TmuxHarness replay-backed proof; castor phpstan: 0 errors; castor deptrac: 0 violations; castor cs-check: clean; PR #253: MERGED into main at 2026-07-03T18:10:03Z
- Summary: Merged via PR #253. Delivered (Part A) general priority ordering in TuiSlotRegistry (order field, ORDER_DEFAULT=0 / ORDER_PINNED_LAST=PHP_INT_MAX, stable ascending sort, symmetry for AboveEditor+BelowEditor) and (Part B) the compact-header as its first pinned consumer — prompts/skills/agents/mcp rendered above the editor via the existing aboveEditorWidget merge slot, refreshed on first tick + 2.5s throttle with change detection. (Part C) docs/tui-architecture.md widget count 13→14 + setWidget order param. Final styling: orange MarkdownHeading labels, blue MarkdownLink values, MCP semantic icons (✓ global / ◈ specific / ✗ failed), simplified agents line. MCP global-vs-specific differentiation wired through McpConfigLoader availability. CompactHeaderRegistrar hardens against empty sessionId + wraps build in try/catch so a widget failure never crashes the event loop.
