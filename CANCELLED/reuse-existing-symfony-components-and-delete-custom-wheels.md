# Reuse existing Symfony components and delete custom wheels

## Goal
## Goal
Replace only high-confidence custom infrastructure that duplicates already-installed Symfony 8.1 components. This is a deletion-first cleanup prompted by PR #381: use components directly at the owning boundary; do not invent Codec/Decoder/Mapper/adapter/factory wrappers.

## Evidence from preliminary parallel scout audit

### Event/orchestration deletion
- Verify external extensions do not reference `AgentCore/Contract/Extension/EventSubscriberInterface`, then delete the unused contract.
- Verify no listener depends on the `BoundaryHookEvent` / `after_turn_commit` Symfony bridge, then remove the inert Serializer round-trip/EventDispatcher branch from `HookDispatcher`; retain the real typed `HookSubscriberInterface` aggregation and failure isolation.

### Persistence primitive
- Replace `SessionRunStore::compareAndSwap()` in-place `state.json` write with `Symfony\Component\Filesystem\Filesystem::dumpFile()` so unlocked readers see the old or complete new file, never partial JSON. Preserve the existing lock, CAS contract, paths, and failure behavior.

### Direct Serializer boundaries
- Replace `ForksConfigDTO::fromRaw()` / `ChildExtensionsConfigDTO::fromRaw()` plumbing with the already-injected Symfony Serializer/Validator, preserving the explicit forbidden `agents.extensions.enabled` semantic rule.
- Replace MCP catalog DTO `fromArray()/toArray()` graphs with direct shared Serializer usage in `SessionFileMcpToolCatalogStore`; preserve documented camelCase keys via explicit metadata and fail clearly on malformed canonical data.
- Replace `CodexAuthRecord::fromArray()/toArray()` with direct shared Serializer use in `CodexAuthStorage`; preserve exact `accountId` public format, credential permissions, atomic writes, and fail-closed behavior.
- No new Codec, Decoder, Mapper, custom normalizer, or duplicate Serializer stack unless a proven Symfony limitation is presented to the user first.

### Symfony TUI reuse
- Replace the bespoke ANSI/box-drawing `/hotkeys` pipeline with a simple Markdown table returned as a normal `TranscriptMessage(style: markdown)` and rendered by the existing Symfony `MarkdownWidget`, which already supports tables. Delete `HotkeyTableRenderer`, `HotkeyTableData`, and the special `SubmitListener` branch. Preserve grouping, empty-state copy, footer notes, and readable key labels; keep any label formatter tiny and do not retain width/padding/table code.
- Replace the exact copied UTF-8/control-byte implementation in `EditorState::fromText()` with Symfony TUI `Widget\Util\StringUtils::sanitizeUtf8()` and `stripControlBytes()`, retaining CRLF/CR normalization order.
- Let `SelectListWidget` own completion Up/Down selection, wrapping, visible-window scrolling, and selection events; retain Hatfield completion lifecycle, provider results, replacement ranges, and editor focus.
- Replace `WelcomeTranscriptWidget` with native `TextWidget` only if normal and pathological narrow-width virtual proofs remain identical enough for the finalized UX.

## Explicit non-goals / keep custom
- Do not import Symfony TUI `Widget\Util\Line` directly: it is internal and `EditorWidget` already uses it; Hatfield has no duplicate line-editing engine to remove.
- Do not replace read-only `/settings-show` with `SettingsListWidget`. `SettingsListWidget` is an interactive value-cycling/submenu editor; `/settings-show` owns nested values, source provenance, descriptions, and restart warnings. A future interactive `/settings` is a separate product task.
- No broad migration of Hatfield widgets to experimental `AbstractWidget`, no custom table widget, no Console `Table` inside transcript, no Finder-for-every-scandir cleanup, no Symfony Config tree duplicating DTO validation, no runtime JSONL protocol rewrite, no AI config rewrite in this task.
- Do not remove the `ext:` `CommandRouter` path without a separate explicit public-surface decision.

## Implementation discipline
Implement as sequential, independently reviewable slices on the task worktree. Before each slice, trace all callers and prove the Symfony API is already installed and reduces net runtime/config LOC. If a candidate needs more custom glue than it deletes, skip it and record why rather than forcing Symfony usage.

## Acceptance criteria
- Runtime/config LOC decreases overall; no new dependency and no new wrapper abstraction around Symfony.
- Unused AgentCore event contract and inert boundary-hook bridge are removed only after repository and extension reference proof; typed hook behavior remains unchanged.
- SessionRunStore state writes are atomic for unlocked readers, with a runnable concurrent reader/writer regression test.
- Fork, MCP catalog, and Codex auth boundaries use the configured shared Serializer/Validator directly; public key casing, strict malformed-data behavior, permissions, and atomic storage are proven.
- `/hotkeys` renders through the existing MarkdownWidget table support; bespoke table renderer/result/special listener path and implementation-mirroring width tests are deleted.
- EditorState delegates exact UTF-8/control-byte sanitation to Symfony TUI StringUtils; normalization behavior remains covered.
- Completion navigation delegates to SelectListWidget without breaking editor focus, acceptance, cancellation, replacement ranges, or changing-suggestion selection behavior.
- TUI changes have virtual behavior proof at the lowest correct layer; persistence/Serializer changes have focused boundary tests.
- `castor test`, `castor test:controller-replay`, `castor test:tui`, `castor deptrac`, `castor phpstan`, `castor cs-check`, and final `castor check` pass.
- Final reviewer inventories every retained custom abstraction and rejects any replacement that adds more glue than it removes; Ponytail verdict is lean.

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
- Created: 2026-08-14T23:14:39.192Z

## Task workflow update - 2026-08-14T23:20:53.465Z
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Superseded before implementation because the hotkeys requirement was wrong: preserve the existing charming themed view and implement a real Symfony TUI AbstractWidget rather than replacing it with MarkdownWidget/TextWidget output.
