# Fix 10 silent-degradation catch blocks found in repo audit (add warning+ logs, no behavior change)

## Goal
From the 2026-08-18 repo-wide silent-catch audit (301 catch blocks reviewed; no HIGH findings — all write paths clean). Fix the MEDIUM and LOW-MEDIUM silent-degradation catches. Each is a 1–3 line warning/error log before the existing fallback; NO behavior changes, NO new config, NO refactors. If a site already sits under a caller that logs warning+ (verify first), prefer plain comment documenting intent over double-logging.

MEDIUM (silent loss of user-visible functionality):
1. src/CodingAgent/Mcp/Catalog/SessionFileMcpToolCatalogStore.php:62 — read() JsonException → return null. Corrupt mcp-tools.json indistinguishable from absent → MCP tools silently vanish. Log warning (run_id, path) then return null.
2. src/CodingAgent/Extension/ExtensionToolHookEventSubscriber.php:439 — runResultHooks() hook throws → bare continue. Log error matching sibling hook handlers (:84, :159), then continue.
3. src/Platform/Bridge/Generic/DurableResultConverter.php:442 — JsonException on tool-arg JSON → $arguments = []. Log warning (block id), keep fallback.
4. src/AgentCore/Infrastructure/SymfonyAi/LlmPlatformAdapter.php:929 — parseArguments() malformed JSON → []. Log warning (toolCallId), keep fallback.
5. src/CodingAgent/Tool/ImageProcessing/ImageAttachmentProcessor.php:222 — outer Throwable → self::original(...). Log warning (file, exception class) before fallback.
6. src/CodingAgent/CLI/FileMentionIndexBuilder.php:579 — isCwdVcsIgnored() git failure → return false. Log warning (cwd, git error). Sibling :548 same pattern — apply if same fix fits.

LOW-MEDIUM:
7. src/Tui/Export/SessionEventsExportService.php:1378 — parseEvents() skips unparseable lines. Log warning with line offset.
8. ImageAttachmentProcessor.php:476 — applyExifOrientationWithGd() Throwable → un-oriented image. Log warning (file, orientation).
9. src/AgentCore/Application/Pipeline/ApplyCommandHandler.php:822 — resolveToolCallContinuationRef() InvalidArgumentException → null. Log warning (questionId, ref).
10. src/CodingAgent/Infrastructure/SymfonyAi/Http/LlmHttpRetryPolicy.php:151 — empty catch (comment-only) on Retry-After parse. Log warning (raw header value) or debug if genuinely expected-often — judge from traffic reality; fill the catch.

Audit notes for the implementer: audit performed on integration checkout 2026-08-18 (line numbers may drift slightly — rg for the pattern if moved). Verify each logger exists in the class already; where none exists, inject via constructor following class conventions (most audit targets already have loggers). GenericPlatformResultConverter/LlmPlatformAdapter sites must not log tool arguments content itself — identifiers only.

## Acceptance criteria
- All 10 listed catch sites log at warning+ (or rethrow where caller already logs) with correlation context (run_id/session_id/file path where available) before degrading; no new behavior changes beyond logging — degraded fallbacks themselves stay as-is.
- SessionFileMcpToolCatalogStore::read() corrupt-JSON path: warning with run_id + file path (top priority — silent MCP tool loss today).
- ExtensionToolHookEventSubscriber::runResultHooks() extension-hook failure: error-level log matching siblings (:84, :159); then continue.
- LlmHttpRetryPolicy:151 empty catch filled — no comment-only empty catch remains in src/.
- One runnable check per touched path or group where feasible (existing test class updated to assert log emission); no behavioral test churn.
- Focused validation green: castor test, castor phpstan, castor deptrac, castor cs-check; no castor check needed unless runtime paths demand it (logging-only change).
- Out of scope: pattern groups A/B (display-only skips, documented best-effort probes) and the info/debug-level table from the audit — do not touch them in this task.

## Workflow metadata
Status: ARCHIVE
Branch: task/cleanup-silent-catch-degradation
Worktree: /home/ineersa/projects/agent-core-worktrees/cleanup-silent-catch-degradation
Fork run: b42a2c6df
PR URL: https://github.com/ineersa/agent-core/pull/413
PR Status: merged
Started: 2026-08-18T20:43:54.167Z
Completed: 2026-08-18T21:31:25.794Z

## Work log
- Created: 2026-08-18T20:06:59.577Z

## Task workflow update - 2026-08-18T20:43:54.167Z
- Moved TODO → IN-PROGRESS.
- Created branch task/cleanup-silent-catch-degradation.
- Created worktree /home/ineersa/projects/agent-core-worktrees/cleanup-silent-catch-degradation.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/cleanup-silent-catch-degradation.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/cleanup-silent-catch-degradation.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/cleanup-silent-catch-degradation.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/cleanup-silent-catch-degradation.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/cleanup-silent-catch-degradation/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/cleanup-silent-catch-degradation.

## Task workflow update - 2026-08-18T20:53:59.993Z
- Recorded fork run: b42a2c6df
- Validation: Focused filter (7 affected test classes): 118 OK / 753 assertions; castor test: 4688 OK / 18629 assertions; castor phpstan: 0 errors; castor deptrac: 0 violations; castor cs-fix + cs-check: clean; No castor check per task scope (logging-only); gate runs at CODE-REVIEW transition
- Summary: Implementation complete via fork. Commit b42a2c6df (13 files, +188/−16): all 10 audit sites now log warning+ (site 2 at error level, matching sibling hook handlers) before unchanged fallbacks; no comment-only sites remained (no caller already logged). Sibling empty catch at FileMentionIndexBuilder:548 (isInsideGitWorkTree) also filled — same pattern, flagged in task as apply-if-fits. Sites 3/10 were factory-constructed: SymfonyAiProviderFactory now passes its logger to DurableResultConverter + LlmHttpRetryPolicy (without this, prod would never log). Optional ?LoggerInterface trailing args where classes lacked loggers (avoids 20+ test constructor churn); DI autowires in prod. Site 4: parseArguments gained toolCallId param for identifier-only logging (never arg content). 3 log-emission tests added to existing classes (sites 1, 2, 10); sites 3–9 untested per no-churn acceptance. Known residual: ImageAttachmentProcessor/SessionEventsExportService/ApplyCommandHandler manual-new paths (if any) would omit logger — DI paths covered. NOTE: local main advanced (auth-credential-store merge) after worktree branched; file sets disjoint, no conflict expected.

## Task workflow update - 2026-08-18T21:03:09.623Z
- Updated PR Status: approved
- Validation: Reviewer: APPROVED, 0 blockers, 0 nits requiring change; From fork b42a2c6df: castor test 4688 OK/18629, phpstan 0, deptrac 0, cs-check clean
- Summary: Reviewer APPROVED with zero findings (NTHs only, not requested): spec-fidelity PASS — strictly logging-only, no scope creep; all 10 + sibling fallbacks byte-identical; identifiers-only logging verified (sites 3/4; site 9 continuation_ref confirmed ids-only in both construction paths); prod logger reachability verified per class (6 autowired via _defaults + monolog alias, 2 factory-wired via SymfonyAiProviderFactory autowired logger; no manual-new path exists in src or extensions); retry-after warning spam bounded (≤3/request, garbage header is precisely worth surfacing); structural claim (no caller pre-logs) spot-checked at 6 sites. Left as NTHs: LlmHttpRetryStrategy docblock 'headers never logged' now contradicts retry_after logging; error_message context inconsistency between sites.

## Task workflow update - 2026-08-18T21:06:02.420Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (165.4s).
- Pushed task/cleanup-silent-catch-degradation to origin.
- branch 'task/cleanup-silent-catch-degradation' set up to track 'origin/task/cleanup-silent-catch-degradation'.
- Created PR: https://github.com/ineersa/agent-core/pull/413

## Task workflow update - 2026-08-18T21:31:25.794Z
- Moved CODE-REVIEW → DONE.
- Closed JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/cleanup-silent-catch-degradation.
- Merged task/cleanup-silent-catch-degradation into integration checkout.
- Merge made by the 'ort' strategy.
 .../Application/Pipeline/ApplyCommandHandler.php   | 14 +++++++-
 .../SymfonyAi/LlmPlatformAdapter.php               | 13 ++++++--
 src/CodingAgent/CLI/FileMentionIndexBuilder.php    | 20 ++++++++++--
 .../Extension/ExtensionToolHookEventSubscriber.php | 12 ++++++-
 .../SymfonyAi/Http/LlmHttpRetryPolicy.php          | 13 ++++++--
 .../SymfonyAi/SymfonyAiProviderFactory.php         |  2 ++
 .../Mcp/Catalog/SessionFileMcpToolCatalogStore.php | 13 +++++++-
 .../ImageProcessing/ImageAttachmentProcessor.php   | 23 ++++++++++++--
 .../Bridge/Generic/DurableResultConverter.php      | 11 ++++++-
 src/Tui/Export/SessionEventsExportService.php      | 15 +++++++--
 .../ExtensionToolHookEventSubscriberTest.php       | 37 ++++++++++++++++++++++
 .../SymfonyAi/Http/LlmHttpRetryPolicyTest.php      | 12 +++++++
 .../Catalog/SessionFileMcpToolCatalogStoreTest.php | 19 +++++++++++
 13 files changed, 188 insertions(+), 16 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/cleanup-silent-catch-degradation.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/cleanup-silent-catch-degradation.
- Pulled integration checkout: Merge made by the 'ort' strategy..

## Task workflow update - 2026-08-18T21:31:36.436Z
- Summary: PR #413 merged to main; merged into integration checkout (ort, 13 files +188/−16); worktree removed. All 10 audit MEDIUM/LOW-MEDIUM silent-degradation catches + pre-authorized sibling now log warning+/error before unchanged fallbacks; prod logger reachability verified (autowired + factory-wired); 3 log-emission tests. Audit follow-through complete — no HIGH findings existed; groups A/B + info/debug table remain deliberately untouched.

## Task workflow update - 2026-08-19T18:16:44.379Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
