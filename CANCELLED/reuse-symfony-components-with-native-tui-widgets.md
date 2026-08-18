# Reuse Symfony components with native TUI widgets

## Goal
## Goal
Replace only high-confidence custom infrastructure that duplicates installed Symfony 8.1 components. Deletion-first: use components directly at the owning boundary; do not invent Codec/Decoder/Mapper/adapter/factory wrappers.

This supersedes the cancelled `reuse-existing-symfony-components-and-delete-custom-wheels` task. Its Markdown hotkeys requirement was rejected: preserve the existing themed/charming view and implement a real Symfony TUI widget rather than pre-rendering into MarkdownWidget or TextWidget.

## Event/orchestration deletion
- Verify external extensions do not reference `AgentCore/Contract/Extension/EventSubscriberInterface`, then delete the unused contract.
- Verify no listener depends on `BoundaryHookEvent` / `after_turn_commit`, then remove the inert Serializer round-trip/EventDispatcher branch from `HookDispatcher`; retain typed `HookSubscriberInterface` aggregation and failure isolation.

## Persistence primitive
- Replace `SessionRunStore::compareAndSwap()` in-place `state.json` write with `Symfony\Component\Filesystem\Filesystem::dumpFile()` so unlocked readers see the old or complete new file. Preserve locking, CAS, paths, and failure behavior.

## Direct Serializer boundaries
- Replace `ForksConfigDTO::fromRaw()` / `ChildExtensionsConfigDTO::fromRaw()` plumbing with the configured Symfony Serializer/Validator while preserving the forbidden `agents.extensions.enabled` semantic rule.
- Replace MCP catalog DTO `fromArray()/toArray()` graphs with direct shared Serializer usage in `SessionFileMcpToolCatalogStore`; preserve documented camelCase keys and strict malformed-data failures.
- Replace `CodexAuthRecord::fromArray()/toArray()` with direct shared Serializer use in `CodexAuthStorage`; preserve exact `accountId` format, credential permissions, atomic writes, and fail-closed behavior.
- No new Codec, Decoder, Mapper, custom normalizer, or duplicate Serializer stack unless a proven Symfony limitation is presented first.

## Native Symfony TUI reuse
- Preserve `/hotkeys` grouping, themed borders/headers/keys/descriptions, empty-state copy, footer notes, and overall visual character.
- Replace `HotkeyTableRenderer` and pre-rendered ANSI transcript text with a proper `HotkeyTableWidget extends Symfony\Component\Tui\Widget\AbstractWidget`. It must own structured rows and return rendered lines directly from `render(RenderContext)`; do not route table output through `TextWidget` or `MarkdownWidget`.
- Use Symfony TUI style elements and `AnsiUtils` (`visibleWidth()`, `truncateToWidth(..., pad: true)`) instead of duplicating ANSI width/truncation/padding utilities. Layout must recompute from `RenderContext` columns and remain correct after terminal resize.
- Keep this hotkey-specific unless a second real table consumer is proven; do not build a speculative generic table framework.
- Replace copied UTF-8/control-byte logic in `EditorState::fromText()` with Symfony TUI `Widget\Util\StringUtils::sanitizeUtf8()` and `stripControlBytes()`, retaining CRLF/CR normalization order.
- Let `SelectListWidget` own completion Up/Down selection, wrapping, visible-window scrolling, and selection events; retain Hatfield completion lifecycle, provider results, replacement ranges, and editor focus.
- Replace `WelcomeTranscriptWidget` with a native widget only if normal and pathological narrow-width virtual proofs preserve the finalized UX.

## Explicit non-goals
- Do not import Symfony TUI `Widget\Util\Line` directly: it is internal and `EditorWidget` already uses it; Hatfield has no duplicate line-editing engine to remove.
- Do not replace read-only `/settings-show` with `SettingsListWidget`. It is an interactive value-cycling/submenu editor, while `/settings-show` owns nested values, source provenance, descriptions, and restart warnings. A future interactive `/settings` is separate.
- No broad widget hierarchy migration, Console `Table` buffered into `TextWidget`, Finder-for-every-scandir cleanup, Symfony Config tree duplicating DTO validation, runtime JSONL rewrite, AI config rewrite, or `ext:` CommandRouter deletion in this task.

## Discipline
Implement sequential, independently reviewable slices. Before each slice, trace all callers and prove the Symfony API reduces net runtime/config LOC or removes duplicated behavior. If a candidate requires more glue than it deletes, skip it and record why.

## Acceptance criteria
- Runtime/config LOC decreases overall; no new dependency or wrapper abstraction around Symfony.
- Unused AgentCore event contract and inert boundary-hook bridge are removed only after repository and extension reference proof; typed hook behavior remains unchanged.
- SessionRunStore state writes are atomic for unlocked readers, with a runnable concurrent reader/writer regression test.
- Fork, MCP catalog, and Codex auth boundaries use configured shared Serializer/Validator services directly; public key casing, strict malformed-data behavior, permissions, and atomic storage are proven.
- `/hotkeys` uses a real `AbstractWidget` with structured input and direct `RenderContext` rendering; no MarkdownWidget/TextWidget table output or pre-rendered ANSI transcript payload.
- Hotkey visual character is preserved, and narrow/resize proofs cover column sizing, ANSI-aware truncation, and styling through Symfony TUI primitives.
- EditorState delegates exact UTF-8/control-byte sanitation to Symfony TUI StringUtils with behavior coverage.
- Completion navigation delegates to SelectListWidget without breaking focus, acceptance, cancellation, replacement ranges, or changing-suggestion selection behavior.
- TUI changes have virtual proof at the lowest correct layer; persistence/Serializer changes have focused boundary tests.
- `castor test`, `castor test:controller-replay`, `castor test:tui`, `castor deptrac`, `castor phpstan`, `castor cs-check`, and final `castor check` pass.
- Final reviewer inventories retained custom abstractions and rejects replacements that add more glue than they remove; Ponytail verdict is lean.

## Workflow metadata
Status: CANCELLED
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-08-14T23:21:34.236Z

## Task workflow update - 2026-08-14T23:39:59.886Z
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Superseded before implementation because this draft incorrectly combined independent TUI and non-TUI Symfony reuse work. Replaced by two focused TODO tasks as requested.
