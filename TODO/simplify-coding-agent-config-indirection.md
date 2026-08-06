# Simplify verified CodingAgent config indirection and dead APIs

## Goal
Deep ponytail audit of `src/CodingAgent/Config` verified these cuts with IDE references plus YAML string-consumer checks. Estimated net reduction: ~220 lines, no dependency changes.

Scope:
- Fold the production-single-caller `ModelSettingsPersister` into `ModelSelectionService`; remove the dedicated class and redundant dedicated test coverage while preserving persistence behavior in `ModelSelectionServiceTest`.
- Remove deprecated, test-only `TuiConfig::fromArray()` and update tests to use the production constructor.
- Remove test-only `SettingsPathResolver::resolveList()` and `getAppRoot()` plus implementation-mirroring tests.
- Remove unreferenced `AppResourceLocator::getAppRoot()`.
- Remove unused `ModelSelectionService::cycleReasoning()` and tests-only `ModelResolver::cycleReasoning()` plus its test block; preserve `ModelResolver::LEVELS` and `cycleReasoningForCurrentModel()`.

Explicit exclusions:
- Do not modify any Subagent or Fork production/test path; another task is active there.
- Do not change model selection, persistence, settings resolution, TUI, or provider behavior.
- Do not pursue rejected audit candidates: keep `HatfieldModelCatalog::config()` because `AppConfig::$ai` and `$catalog` can be independently supplied; keep the two private numeric/env parsers rather than adding a shared helper abstraction.
- Do not touch generated `config/reference.php` or Composer dependencies.

## Acceptance criteria
- `ModelSettingsPersister` is removed and its behavior is owned directly by `ModelSelectionService` with no persistence or validation change.
- Deprecated/test-only and unreferenced methods listed in scope are removed, with callers/tests minimally updated.
- No file whose path or subject is Subagent/Fork is modified.
- Existing rationale comments for persistence, in-memory config synchronization, path resolution, and model-level handling are preserved or updated.
- Focused Castor tests for CodingAgent config/model selection pass; `castor deptrac`, `castor phpstan`, and `castor cs-check` pass.
- Final branch diff is behavior-preserving and removes approximately 200 lines without adding abstractions, settings, APIs, or dependencies.

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
- Created: 2026-08-06T14:57:22.257Z
