# Stop reusing idle Codex WebSockets after cross-session disconnects

## Goal
Sessions 32, 64, and 65 repeatedly retry after cached WebSocket reuse. On 2026-09-25, 19 send failures occurred across three sessions and multiple workers; all threw Amp\\Websocket\\WebsocketClosedException after 63–285 seconds without a completed response, below the configured 300-second idle TTL. Other reused sockets closed before response.completed. In a bounded log comparison, 55 reuses under 60 seconds completed; three completed between 60–74 seconds; no reuses at 75+ seconds completed. This points to idle sockets closing before Hatfield expires them; the provider, a network intermediary, or local socket state could be responsible. First change the existing idle limit conservatively, not the retry policy, and validate that healthy short-idle requests still reuse connections. Do not signal live or root-owned workers or make paid provider requests without authorization.

## Acceptance criteria
- Use the existing cache idle TTL to retire sockets before the observed failure interval; keep the catalog and platform defaults consistent.
- Preserve successful short-idle reuse, in-flight requests, and retry/fail-loud semantics; no new setting, fallback, or keepalive.
- Prove the expiry boundary deterministically with the cache's injected clock and run focused Castor validation.
- Report that the change only affects new processes and live provider behavior remains unproven until measured after deployment.

## Workflow metadata
Status: DONE
Branch: task/2026-09-25-stop-reusing-idle-codex-websockets
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-25-stop-reusing-idle-codex-websockets
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/529
PR Status: merged
Started: 2026-09-25T13:48:28+00:00
Completed: 2026-09-25T14:16:27+00:00

## Work log
- Created: 2026-09-25T13:48:12+00:00

## Task workflow update - 2026-09-25T13:48:28+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-25-stop-reusing-idle-codex-websockets.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-25-stop-reusing-idle-codex-websockets.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-25-stop-reusing-idle-codex-websockets.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-25-stop-reusing-idle-codex-websockets.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-25-stop-reusing-idle-codex-websockets.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-25-stop-reusing-idle-codex-websockets/.idea.
- Summary: Read-only cross-session log analysis found 19 send failures on cached sockets in sessions 32/64/65; idle ages begin at 63.7s, and no observed 75s+ reuse completed. Main owns the small existing TTL/default change in an isolated worktree; no paid provider requests or live-session restarts.

## Task workflow update - 2026-09-25T13:48:56+00:00
- Summary: Routing: Platform cache settings and config/ai-catalog.yaml are the only production changes; CodexWebSocketConnectionCache already retires idle sockets on acquire using injected Clock. Existing cache test owns boundary proof. Main owns this small cohesive change. Session 32/64/65 aggregate from current logs shows 19 send failures and other stream closures after stale reuse; 55 successes under 60 seconds, 3 successes at 60–74 seconds, none at 75+ seconds. 60-second cutoff is the conservative existing-config change.
- Ownership: owner=main; fork_run=none; revision=agent-core task@fb4096b65; scope=Shorten existing Codex cached-socket idle expiration below first observed failure; align catalog and platform defaults; deterministic boundary proof without retry or live-session changes; outcome=assigned; commit=none

## Task workflow update - 2026-09-25T13:54:52+00:00
- Validation: Existing deterministic cache test now asserts reuse after 59s idle and fresh connection at 60s idle with MockClock. Test failed RED with default 300, then passed GREEN with 60. Focused Castor 45 tests/215 assertions plus AiCatalogTest 7 tests/63 assertions pass. Scoped castor phpstan 0 errors, castor deptrac 0 violations, castor dead-code 0 errors, castor cs-check 0 changes, castor docs:validate passed; git diff --check clean; worktree clean at e2a8a5bcd.
- Summary: Implemented at task@e2a8a5bcd: shortened the existing platform default and catalog Codex WebSocket idle TTL from 300 to 60 seconds. The existing cache releases and expires sockets by idleSince on acquire; no retry logic, API, keepalive, socket writes, or transport default changed. The shorter TTL affects new processes only. No paid provider trial or live-session restart. The observed server/intermediary close cause remains unproven; this conservatively retires sockets before the observed 63.7s minimum failure interval.
- Ownership: owner=main; fork_run=none; revision=agent-core task@e2a8a5bcd; scope=Shorten existing Codex cached-socket idle expiration below observed failures; align catalog/platform defaults; deterministic boundary proof; outcome=completed; commit=e2a8a5bcd

## Task workflow update - 2026-09-25T13:58:33+00:00
- Summary: Task-to-pr review requested on worktree revision e2a8a5bcd; branch and integration checkout clean.
- Review: role=reviewer; artifact=agent_9beb29b0d5edbba1; revision=agent-core task@e2a8a5bcd; scope=Independent specification-fidelity and reliability review of 60-second idle TTL, default/catalog alignment, and boundary test; outcome=assigned

## Task workflow update - 2026-09-25T13:58:52+00:00
- Review: role=reviewer; artifact=agent_9beb29b0d5edbba1; revision=agent-core task@e2a8a5bcd; scope=Resume previous Cached Codex reviewer; outcome=blocked; reason=artifact belongs to previous parent lifetime after /resume. Assigning fresh reviewer.

## Task workflow update - 2026-09-25T14:06:10+00:00
- Summary: Independent review agent_26f78d80f04ffb4b requested changes at e2a8a5bcd: bundled catalog version must bump or castor catalog:version-check fails; existing user ~/.hatfield/ai-catalog.yaml v7 still contains TTL 300 and shadows new default until providers:update; AiProviderConfig docblock still says default 300. Main owns bounded fixes, no user catalog edit or running-worker change.
- Review: role=reviewer; artifact=agent_26f78d80f04ffb4b; revision=agent-core task@e2a8a5bcd; scope=Specification-fidelity, effective config precedence and deterministic TTL boundary; decision=REQUEST CHANGES; blockers=catalog version bump, user catalog precedence operational follow-up, stale default docblock.
- Ownership: owner=main; fork_run=none; revision=agent-core task@e2a8a5bcd; scope=Resolve version guard and stale docblock, verify catalog update path without editing user config or restarting sessions; outcome=assigned; commit=none

## Task workflow update - 2026-09-25T14:08:11+00:00
- Validation: castor catalog:version-check pass; docs:validate pass; castor test filtered catalog/cache/provider 41 tests/143 assertions and ProvidersUpdateCommandTest 7/57 pass; cs-check clean; git diff --check clean; prior Platform focused 45/215, phpstan/deptrac/dead-code passed on unchanged production logic.
- Summary: Review blockers resolved at a7f54b226: bumped bundled AI catalog version to 8 and updated docs and AiProviderConfig default comment. Castor catalog:version-check now passes. Existing user catalog remains at v7 and TTL 300; after merge, providers:update must refresh it, then new worker processes must load the updated config. Deliberately did not edit ~/.hatfield/ai-catalog.yaml or restart live workers.
- Ownership: owner=main; fork_run=none; revision=agent-core task@a7f54b226; scope=Resolve catalog version guard, stale comment/docs, and record user-catalog operational update; outcome=completed; commit=a7f54b226
- Review: role=reviewer; artifact=agent_26f78d80f04ffb4b; revision=agent-core task@a7f54b226; scope=Re-review fixes for catalog version check, config precedence and documentation at new revision; outcome=assigned

## Task workflow update - 2026-09-25T14:10:08+00:00
- Validation: castor catalog:version-check passed; castor docs:validate passed; focused Castor tests 45/215 + 41/143 + ProvidersUpdateCommandTest 7/57 passed; scoped phpstan/deptrac/dead-code/cs-check passed; castor test:llm-real --filter=LlamaCppSmokeTest 1/8 passed; worktree clean at a7f54b226. Full castor check reserved for CODE-REVIEW transition.
- Summary: Independent reviewer agent_26f78d80f04ffb4b APPROVED a7f54b226 after catalog v8/default/comment fixes; operational condition recorded here and in forthcoming PR: existing ~/.hatfield/ai-catalog.yaml v7 has TTL 300 and takes precedence. Once merged, run `hatfield providers:update` to rebase user catalog to v8/TTL 60, then start fresh agent/controller processes to pick up new settings. No user catalog or running process touched; until that sequence, live sessions retain old TTL. No paid provider proof yet.
- Review: role=reviewer; artifact=agent_26f78d80f04ffb4b; revision=agent-core task@a7f54b226; scope=Re-review catalog version guard, effective user config and stale docs; decision=APPROVE conditional on operational note in task and PR; blockers=none after note.

## Task workflow update - 2026-09-25T14:11:41+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (72.2s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-25-stop-reusing-idle-codex-websockets/var/reports/qa-20260925-141029-3373-66a02fd4.
- Session/run: 65.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-25T14:11:42+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-25-stop-reusing-idle-codex-websockets to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-25-stop-reusing-idle-codex-websockets/var/reports/qa-20260925-141029-3373-66a02fd4.
- Session/run: 65.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-25T14:11:45+00:00
- castor check passed (72.2s).
- Pushed task/2026-09-25-stop-reusing-idle-codex-websockets to origin.
- Created PR: <url>
- Session/run: 65.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-25T14:11:45+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (72.2s).
- Pushed task/2026-09-25-stop-reusing-idle-codex-websockets to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/529
- Validation: MockClock boundary regression failed under 300s and passes with 60s. Focused Castor 45/215, 41/143, 7/57; catalog:version-check, phpstan, deptrac, dead-code, cs-check, docs:validate passed; local LlamaCppSmokeTest 1/8; full castor check via transition.
- Summary: Shortened Codex cached-socket idle TTL from 300 to 60 seconds, matching the observed cross-session failure boundary; bumped bundled catalog to v8, updated docs/comment, and added MockClock boundary proof. Independent reviewer agent_26f78d80f04ffb4b APPROVED a7f54b226 after blocking fixes. No retry/default-transport change or paid provider trial. Existing user catalog and active processes remain unchanged pending the documented post-merge update.

## Task workflow update - 2026-09-25T14:16:27+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-25-stop-reusing-idle-codex-websockets into integration checkout.
- Merge made by the 'ort' strategy.
 config/ai-catalog.yaml                                                  |  4 ++--
 docs/ai-catalog.md                                                      |  2 +-
 src/CodingAgent/Config/Ai/AiProviderConfig.php                          |  2 +-
 src/Platform/Bridge/OpenAICodex/CodexWebSocketCacheSettings.php         |  2 +-
 tests/Platform/Bridge/OpenAICodex/CodexWebSocketConnectionCacheTest.php | 13 +++++++++++--
 5 files changed, 16 insertions(+), 7 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-25-stop-reusing-idle-codex-websockets.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-25-stop-reusing-idle-codex-websockets.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Pre-merge CODE-REVIEW castor check passed (72.2s); post-merge integration castor check pending.
- Summary: PR #529 merged on GitHub at 4d984700487d2cee994d5e3d1a69cfa58e7dea94. Independent review approved a7f54b226; CODE-REVIEW castor check passed. Post-merge integration validation follows. Deployment reminder: existing user AI catalog v7 still specifies TTL 300; run `hatfield providers:update` to rebase catalog to v8 and start fresh processes before expecting the 60-second limit; no live workers were touched.

## Task workflow update - 2026-09-25T14:18:14+00:00
- Validation: Post-merge castor check passed in integration checkout (151.9s; QA run qa-20260925-141638-8544-82f1983b): unit/integration 5075 tests/22803 assertions; controller replay 13/218; TUI 6/40; local llm-real 5/30; deptrac, phpstan, lsp:check, dead-code, cs-check, docs:validate, catalog:version-check all passed; QA leak check clean; llama-proxy cache entries unchanged 406→406.
- Summary: Post-merge integration main at 1f6f753a9; PR #529 merge 4d9847004 integrated through task merge 7de91a54b and git pull merge 1f6f753a9. Worktree removed and integration working tree clean. User AI catalog still v7/TTL 300; `hatfield providers:update` and fresh agent/controller processes remain an operational follow-up, not performed in this task.

## Task workflow update - 2026-09-25T14:20:20+00:00
- Summary: Operational follow-up performed with merged checkout's `bin/console providers:update --no-interaction` (installed global `hatfield` v0.0.24 was older than the bundled catalog v8). User catalog ~/.hatfield/ai-catalog.yaml now version 8, openai-codex idle TTL 60, max age 3300, mode 0600. Command refreshed 8 metadata entries from models.dev; 73 upstream model IDs listed but not added. No settings override for TTL in user/project settings, no repo edits. Existing agent/controller workers were not restarted; new processes will load updated user catalog.
