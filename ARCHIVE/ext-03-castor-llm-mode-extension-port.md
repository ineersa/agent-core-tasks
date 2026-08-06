# EXT-03 Port castor-llm-mode extension to Hatfield project extension

## Goal
## Context

Port the pi `castor-llm-mode` extension (`.pi/extensions/castor-llm-mode.ts`, TypeScript, single file) to a native Hatfield project-level Composer extension. The pi extension is **NOT being decommissioned**. Read the pi source for exact behavior — it's short (~60 lines).

**What castor-llm-mode does** (from `.pi/extensions/castor-llm-mode.ts`):
- Hooks every `bash` tool call.
- If the command matches `(^|\s)(?:vendor/bin/)?castor(?=\s|$)`:
  - Prepends env exports the command is missing: `export LLM_MODE=true`, `export CASTOR_DISABLE_VERSION_CHECK=1`, `export NO_COLOR=1`, `export CLICOLOR=0`.
  - Rewrites bare `castor list` (and `&&`/`||`/`;`-separated occurrences) to `castor list --format=md --short --no-ansi`.
- Mutates the bash tool's `command` argument in place, then lets execution proceed.

**Depends on EXT-01** — specifically Deliverable 4: the `ToolCallDecisionKindEnum::Rewrite` case + `ToolCallDecisionDTO::rewrite(array $arguments)` + subscriber applying rewritten args and proceeding. The current Hatfield ToolCallHook can only Allow/Block/ReplaceResult/RequireApproval, which cannot express "mutate the command, then run it".

## Packaging

Composer package at `.hatfield/extensions/castor-llm-mode/`, autoloaded via `.hatfield/extensions/vendor/autoload.php`. Namespace `Ineersa\HatfieldExt\CastorLlmMode`.

```
.hatfield/extensions/castor-llm-mode/
  composer.json        # PSR-4 autoload Ineersa\\HatfieldExt\\CastorLlmMode\\ => src/
  src/
    CastorLlmModeExtension.php   # implements HatfieldExtensionInterface
    CastorCommandRewriter.php     # pure transformation logic (testable without hooks)
    CastorLlmModeToolCallHook.php # implements ToolCallHookInterface
```

## Registration (`CastorLlmModeExtension::register`)

1. Optionally read settings via `$api->getSettings('castor_llm_mode')` (enable/disable, custom env vars — keep minimal, mirror pi defaults).
2. Register `CastorLlmModeToolCallHook` via `$api->registerToolCallHook()`.

## Behavior to port faithfully

`CastorLlmModeToolCallHook::onToolCall(ToolCallContextDTO $context): ToolCallDecisionDTO`:
1. If `$context->toolName !== 'bash'` → `allow()`. (The pi extension uses `isToolCallEventType("bash", event)`; confirm the bash tool name against the host tool registry — SafeGuard's `tool_names.bash` default is `bash`.)
2. `$command = $context->arguments['command']` (string).
3. If command does not match the castor pattern → `allow()`.
4. Run `CastorCommandRewriter::rewrite(string $command): string`:
   - Build the env-export prefix from any of `LLM_MODE`, `CASTOR_DISABLE_VERSION_CHECK`, `NO_COLOR`, `CLICOLOR` not already present (regex `\bLLM_MODE\s*=`, etc.).
   - Apply `castor list` → `castor list --format=md --short --no-ansi` for every `castor list` occurrence including `&&`/`||`/`;`-separated.
5. Return `ToolCallDecisionDTO::rewrite(['command' => $newCommand], reason: 'castor-llm-mode: prepended LLM env exports and normalized castor list')`.

`CastorCommandRewriter` is a pure class with the two regex transformations — unit-test it exhaustively without touching the hook. This is the core logic; port the exact regexes from the pi file.

## Semantics to confirm against EXT-01

- The rewrite decision must **proceed to execution** with the modified `command` — not short-circuit. EXT-01's subscriber is responsible for applying the rewritten args and continuing.
- If another hook (e.g. SafeGuard) returns RequireApproval on the same call, confirm composition: the rewritten command should be what SafeGuard evaluates. (If EXT-01 applies Rewrite before SafeGuard sees the call, this works naturally; confirm in EXT-01 and document the ordering here.)

## Settings schema (optional, add to `.hatfield/settings.yaml` + `docs/settings.md`)

```yaml
extensions:
  enabled:
    - Ineersa\HatfieldExt\CastorLlmMode\CastorLlmModeExtension
  settings:
    castor_llm_mode:
      enabled: true   # default true
```

Keep the schema minimal — pi has no settings, so defaults are hard-coded. Expose `enabled` for convenience only.

## QA gates (run before CODE-REVIEW)

- `castor phpstan`, `castor cs-check` on the extension package.
- `castor test` — unit tests for `CastorCommandRewriter` (table of inputs → expected outputs: no-castor passthrough, bare `castor list`, `vendor/bin/castor list`, `castor list && castor test`, commands already containing env exports, commands missing each combination of exports). Plus one hook test asserting `onToolCall` returns `allow()` for non-bash and non-castor, and `rewrite()` for a castor command.
- No live LLM needed — pure transformation tests. Load the `testing` skill and read `tests/AGENTS.md` before writing tests.

## Out of scope

- EXT-01 API additions (must already be merged, including the Rewrite decision).
- task-workflow port (EXT-02).
- Decommissioning the pi castor-llm-mode extension (explicitly not doing this).

## Dependencies

- **Requires:** EXT-01 merged, specifically the `ToolCallDecisionKindEnum::Rewrite` deliverable.

## Acceptance criteria
- Composer package at .hatfield/extensions/castor-llm-mode/ with PSR-4 namespace Ineersa\HatfieldExt\CastorLlmMode, autoloadable via .hatfield/extensions/vendor/autoload.php
- CastorLlmModeExtension implements HatfieldExtensionInterface and registers a ToolCallHook
- CastorLlmModeToolCallHook returns ToolCallDecisionDTO::rewrite() with the transformed command for bash+castor calls, allow() otherwise
- CastorCommandRewriter is pure logic porting the exact pi regexes: env-export prefixing (LLM_MODE, CASTOR_DISABLE_VERSION_CHECK, NO_COLOR, CLICOLOR) and castor list normalization
- Unit tests cover CastorCommandRewriter transformation table + hook allow/rewrite branching
- castor phpstan + cs-check + test pass; testing skill + tests/AGENTS.md conventions followed
- pi castor-llm-mode extension left in place (not decommissioned)

## Workflow metadata
Status: DONE
Branch: task/ext-03-castor-llm-mode-extension-port
Worktree: /home/ineersa/projects/agent-core-worktrees/ext-03-castor-llm-mode-extension-port
Fork run: dkiq2w9inxn9
PR URL: https://github.com/ineersa/agent-core/pull/229
PR Status: merged
Started: 2026-06-25T20:45:08.155Z
Completed: 2026-06-28T16:07:58.428Z

## Work log
- Created: 2026-06-22T22:13:40.106Z

## Task workflow update - 2026-06-25T20:45:08.155Z
- Moved TODO → IN-PROGRESS.
- Created branch task/ext-03-castor-llm-mode-extension-port.
- Created worktree /home/ineersa/projects/agent-core-worktrees/ext-03-castor-llm-mode-extension-port.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/ext-03-castor-llm-mode-extension-port.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/ext-03-castor-llm-mode-extension-port.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/ext-03-castor-llm-mode-extension-port.

## Task workflow update - 2026-06-25T20:50:30.365Z
- Context gathered: read pi castor-llm-mode.ts source, EXT-02 task-workflow extension (reference structure), EXT-01 ExtensionApi contracts, host RegistryBackedToolbox rewrite-phase application, settings.yaml, docs/settings.md, phpunit.xml.dist, phpstan.dist.neon, root composer.json autoload, testing skill + tests/AGENTS.md.
- CRITICAL API DIVERGENCE: task body anticipates a `ToolCallDecisionKindEnum::Rewrite` case + `ToolCallDecisionDTO::rewrite()`, but EXT-01 implemented this DIFFERENTLY and better — as a separate `ToolCallRewriteHookInterface` (rewriteArguments(): ?array) registered via `ExtensionApiInterface::registerToolCallRewriteHook(string $toolName, $hook)`. There is NO `Rewrite` decision kind. The rewrite hook runs in a pre-event phase BEFORE SafeGuard/policy hooks (confirmed in RegistryBackedToolbox::execute), so SafeGuard naturally evaluates the rewritten command. The fork MUST use ToolCallRewriteHookInterface, not a non-existent rewrite() decision.
- USER CONSTRAINT (hard, non-negotiable): REGEX MUST STAY SAME. The 6 pi regexes must be ported byte-for-byte into PHP PCRE — only the JS /literal/ delimiters become PHP '/pattern/' delimiters. Pattern content, lookahead (?=\s|$), \b boundaries, alternations, and global replace-all semantics must NOT be changed/improved. CastorCommandRewriter tests must assert exact string outputs derived from these regexes.

## Task workflow update - 2026-06-25T20:53:41.685Z
- Recorded fork run: dkiq2w9inxn9
- Fork dkiq2w9inxn9 launched in worktree to implement the extension. Instructions encode the corrected API (ToolCallRewriteHookInterface, NOT the stale rewrite() decision) and the hard constraint REGEX MUST STAY SAME (6 pi regexes ported byte-for-byte). Validation scoped to composer autoload refresh, castor phpstan, castor cs-check, castor test (focused + extensions suite + main suite sanity). Fork told explicitly NOT to run castor check, push, or create a PR.

## Task workflow update - 2026-06-25T20:59:17.815Z
- Recorded fork run: dkiq2w9inxn9
- Validation: INDEPENDENTLY RE-VERIFIED by orchestrator in worktree:; Programmatic byte-for-byte regex check (python diff of 6 pi patterns vs PHP constants): ALL 6 MATCH byte-for-byte (REGEX MUST STAY SAME constraint satisfied; only JS /literal/ -> PHP '/pattern/' delimiter + g-flag dropped since preg_replace replaces all); castor phpstan: errors=0, file_errors=0; castor test --filter=CastorCommandRewriterTest: OK (15 tests, 15 assertions); castor test --filter=CastorLlmModeToolCallHookTest: OK (5 tests, 9 assertions); castor cs-check --path=.hatfield/extensions/castor-llm-mode: files_fixed=0, exit 0; Fork also reported (not re-run by orchestrator): full castor test 3589 OK / 11374 assertions; castor test --suite=extensions 37 OK — no regression with extension globally enabled in settings.yaml; API confirmed correct: ToolCallRewriteHookInterface + registerToolCallRewriteHook('bash', $hook) (NOT the stale ToolCallDecisionDTO::rewrite() predicted by task body); rewrite runs in host pre-event phase before SafeGuard
- Summary: Implementation complete and verified. Commit 64a9228a127c51fd1859763707e4083e302bcd32 on branch task/ext-03-castor-llm-mode-extension-port (worktree agent-core-worktrees/ext-03-castor-llm-mode-extension-port). 13 files changed, 383 insertions, 4 deletions. Created package .hatfield/extensions/castor-llm-mode/ (composer.json, CastorCommandRewriter, CastorLlmModeToolCallHook, CastorLlmModeExtension, README, 2 test classes). Wired into root composer.json PSR-4, .hatfield/extensions/composer.json (require+path repo), phpunit.xml.dist extensions suite, phpstan.dist.neon excludePaths, .hatfield/settings.yaml (enabled + castor_llm_mode.enabled:true), docs/settings.md (new section before safe_guard).
- Fork report VERIFIED. Commit 64a9228a confirmed; working tree clean; git diff --stat = 13 files / 383 insertions.
- REGEX MUST STAY SAME: programmatically diffed all 6 pi regex bodies against PHP constants — ALL byte-for-byte identical. Constraint satisfied.
- API: CastorLlmModeToolCallHook implements ToolCallRewriteHookInterface (rewriteArguments(): ?array), registered via registerToolCallRewriteHook('bash'). No stale rewrite() decision used. CastorLlmModeExtension is final readonly, implements HatfieldExtensionInterface, gates on settings.enabled (default true).
- Tests verified correct: isCastorCommand boundary cases (xcastor->false, castorfoo->false, echo castor->true) derive from (^|\s)(?:vendor/bin/)?castor(?=\s|$); rewrite table covers bare list, vendor/bin/list, &&-separated, global replace-all (castor list && castor list -> both normalized), all-exports-present (keeps existing export lines + list normalization only, faithful to pi), single-export-present cases, and castor test (no list transform). Hook test: non-bash/null, bash non-castor/null, bash castor/rewrite, missing command/null, non-string command/null — builds real ToolCallContextDTO via public constructor (no mocking).
- Fork made one CORRECT correction to orchestrator's test-case-e instruction: when all 4 env exports already present, pi keeps the existing export lines and only normalizes the list (exports array empty => no new prefix block). Stripping them would diverge from pi. Fork's test expectation is faithful.
- Wiring independently re-reviewed: root composer.json (PSR-4 autoload+autoload-dev), .hatfield/extensions/composer.json (require + path repo), phpunit.xml.dist (extensions suite dir), phpstan.dist.neon (excludePaths test dir), .hatfield/settings.yaml (enabled + castor_llm_mode.enabled:true), docs/settings.md (section before safe_guard) — all correct.
- STOPPING per task-start workflow. castor check NOT run, no push, no PR, no task move. Handing to user for task-to-pr.

## Task workflow update - 2026-06-26T23:30:35.356Z
- Merged origin/main (51 commits incl. PR #213 issue-205 bash background/cancel/parallel-exec fixes) into task branch: 35a6aef59 → ee70b0ebc. Clean ort merge; only .hatfield/settings.yaml + docs/settings.md auto-merged. Extension tree byte-identical to 64a9228a1 (git diff empty). Focused validation: phpstan 0 errors, CastorCommandRewriterTest 15/15, CastorLlmModeToolCallHookTest 5/5, cs-check clean. Bash-tool fix now on branch; awaiting user retest of `castor test` through Hatfield to confirm the kill is resolved.

## Task workflow update - 2026-06-28T16:05:51.115Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (57.2s).
- Pushed task/ext-03-castor-llm-mode-extension-port to origin.
- branch 'task/ext-03-castor-llm-mode-extension-port' set up to track 'origin/task/ext-03-castor-llm-mode-extension-port'.
- Created PR: https://github.com/ineersa/agent-core/pull/229

## Task workflow update - 2026-06-28T16:07:58.428Z
- Moved CODE-REVIEW → DONE.
- Merged task/ext-03-castor-llm-mode-extension-port into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/run.php                                    |  32 ++++++-
 .castor/shared.php                                 |   6 +-
 .hatfield/extensions/castor-llm-mode/README.md     |  22 +++++
 .hatfield/extensions/castor-llm-mode/composer.json |  19 ++++
 .../castor-llm-mode/src/CastorCommandRewriter.php  |  51 ++++++++++
 .../castor-llm-mode/src/CastorLlmModeExtension.php |  22 +++++
 .../src/CastorLlmModeToolCallHook.php              |  37 ++++++++
 .../tests/CastorCommandRewriterTest.php            | 105 +++++++++++++++++++++
 .../tests/CastorLlmModeToolCallHookTest.php        |  93 ++++++++++++++++++
 .hatfield/extensions/composer.json                 |   9 +-
 .hatfield/settings.yaml                            |   3 +
 composer.json                                      |   6 +-
 docs/settings.md                                   |  18 ++++
 phpstan.dist.neon                                  |   1 +
 phpunit.xml.dist                                   |   1 +
 15 files changed, 415 insertions(+), 10 deletions(-)
 create mode 100644 .hatfield/extensions/castor-llm-mode/README.md
 create mode 100644 .hatfield/extensions/castor-llm-mode/composer.json
 create mode 100644 .hatfield/extensions/castor-llm-mode/src/CastorCommandRewriter.php
 create mode 100644 .hatfield/extensions/castor-llm-mode/src/CastorLlmModeExtension.php
 create mode 100644 .hatfield/extensions/castor-llm-mode/src/CastorLlmModeToolCallHook.php
 create mode 100644 .hatfield/extensions/castor-llm-mode/tests/CastorCommandRewriterTest.php
 create mode 100644 .hatfield/extensions/castor-llm-mode/tests/CastorLlmModeToolCallHookTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/ext-03-castor-llm-mode-extension-port.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/ext-03-castor-llm-mode-extension-port.
- Pulled integration checkout: Merge made by the 'ort' strategy..
