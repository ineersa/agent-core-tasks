# Add durable session prompt-cache diagnostics and inspector command

## Goal
## Goal
Add a Symfony console command (working name: `session:cache:inspect <session-id>`) that explains prompt-cache behavior for a completed or running Hatfield session: request-level and cumulative cache numbers, which parent/child/model/transport each request belongs to, and the first Hatfield-controlled prefix component that changed between requests.

A post-hoc reader of `events.jsonl` is insufficient: session 37 preserved provider usage but lost the final request shape and cached-WebSocket continuation decisions when structured logs rotated. Persist privacy-safe request diagnostics durably with the session so future invalidations are attributable rather than guessed.

## Requested report
- Session summary: input, output, thinking, cache-read, cache-write/creation, uncached input, cache ratio, cost, and request count.
- Keep parent, scout/subagent, and fork runs separated by run/model/provider/transport; optionally summarize them, never silently aggregate incompatible caches.
- Per request: event/step/time, model, transport, full-context versus continuation delta, presence of stable prompt-cache key and previous-response state, provider usage, and cache ratio.
- Prefix comparison against the previous request in the same run/cache family: stable/changed instructions; ordered tool names/schema/order; ordered provider-input item type/role/tool name; longest common prefix and first changed/inserted/removed component with byte sizes (token boundary only when provider telemetry supports it).
- Explicitly distinguish provider-reported cache telemetry from local inference.

## Durable privacy boundary
Persist fingerprints/lengths/structural metadata, not raw prompts, tool output, headers, environment values, access tokens, account identifiers, or API keys. Use a run-scoped keyed digest for exact provider-visible component comparison so short predictable content is not stored as a plain digest. Diagnostics must survive normal log rotation and remain colocated with the canonical session/child artifact lifecycle.

## Cache Hunter reference
Reviewed https://github.com/co-l/cache-hunter (`/tmp/cache-hunter`, 2026-08-04 clone). Reuse concepts only: its request-column/message-row hash grid, separate tools hash, thread separation, token/latency timeline, and first-divergence visualization are useful. Do not copy its architecture or code: it is an HTTP-only external proxy, stores full raw request bodies and headers in SQLite, uses collision-prone 4-character MD5 hashes, does not cover Hatfield's cached WebSocket continuation path, and its checked-in source/README are partially inconsistent.

## Scope constraints
- Native Hatfield/Symfony console command; no Node service, web UI, proxy, SQLite sidecar service, or new dependency.
- Instrument the shared Hatfield prompt/request seam for provider-neutral instructions/tools/input structure; additionally record Codex full-vs-delta continuation metadata at its final wire-body seam.
- Historical sessions without durable fingerprints should still show their existing provider usage and clearly mark prefix attribution unavailable; do not fabricate causes.
- No required prompt, safety context, tool, or provider behavior changes in this observability task.
- Human-readable console output first; machine-readable export is a separate decision unless explicitly approved.

## Related evidence
Session 37 cached prefix froze at 41,472 tokens after seq 241 while input grew to ~84k. It used `websocket-cached`; continuation frames sent delta input plus `previous_response_id` and previously removed `prompt_cache_key`. Existing events could reconcile usage totals but could not recover per-request full/delta choice or first provider-visible changed component.

## Acceptance criteria
- A console command inspects a session by ID and reports request-level plus cumulative input/output/thinking/cache-read/cache-write/uncached/cost metrics with exact formulas.
- Parent and child runs are attributed separately by model/provider/transport; incompatible cache families are never silently combined.
- New sessions durably record privacy-safe structural fingerprints sufficient to identify the longest stable prefix and first changed instruction/tool/input component between adjacent requests.
- Codex diagnostics show full-context versus cached-WebSocket delta mode and whether prompt_cache_key/previous_response_id were present, without persisting their values.
- No raw prompts, tool outputs, request headers, secrets, account identifiers, or environment values are persisted or printed.
- Historical sessions lacking fingerprints degrade honestly to usage-only output with an explicit attribution-unavailable notice.
- Focused automated coverage proves one stable-prefix case, one tool/input mutation case, usage arithmetic, child attribution, and privacy exclusions at the lowest correct layer.
- No external proxy/runtime/dependency or Cache Hunter source is introduced.

## Workflow metadata
Status: ARCHIVE
Branch: task/add-session-prompt-cache-diagnostics-command
Worktree: /home/ineersa/projects/agent-core-worktrees/add-session-prompt-cache-diagnostics-command
Fork run: bi3s3qgewfri
PR URL: https://github.com/ineersa/agent-core/pull/363
PR Status: merged
Started: 2026-08-04T23:16:27.368Z
Completed: 2026-08-05T20:06:11.304Z

## Work log
- Created: 2026-08-04T19:26:49.130Z

## Task workflow update - 2026-08-04T23:16:27.368Z
- Moved TODO → IN-PROGRESS.
- Created branch task/add-session-prompt-cache-diagnostics-command.
- Created worktree /home/ineersa/projects/agent-core-worktrees/add-session-prompt-cache-diagnostics-command.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/add-session-prompt-cache-diagnostics-command.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/add-session-prompt-cache-diagnostics-command.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/add-session-prompt-cache-diagnostics-command.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/add-session-prompt-cache-diagnostics-command.

## Task workflow update - 2026-08-04T23:30:09.151Z
- Recorded fork run: funo4u3xhphr
- Summary: Task-start investigation complete. Three scouts traced shared Symfony AI invocation/usage persistence, Codex SSE/WebSocket continuation wire seams, and command/session/child artifact readers. Final implementation scope: optional privacy-safe diagnostics on existing LLM step events; provider-neutral recorder shared by generic/Codex clients; full logical fingerprints plus Codex wire-mode booleans; result/message plumbing; per-family parent/child inspector command with honest historical fallback. No new event type, migration, settings, machine export, sidecar, proxy, compaction instrumentation, TUI behavior, or provider behavior change. Implementation fork launched.
- Scouts read root AGENTS.md, testing skill, tests/AGENTS.md, and task spec fully; no edits or QA.
- Specification decisions: logical PlatformInterface invocation records; inner auth/HTTP retry remains one record; thinking-only second invocation preserves both; per-family only; persisted cost authority; unknown telemetry/transport remains unknown; existing provider_cache_key/run_id used only as HMAC key source and never printed.
- Implementation fork run funo4u3xhphr launched in `/home/ineersa/projects/agent-core-worktrees/add-session-prompt-cache-diagnostics-command`.

## Task workflow update - 2026-08-05T00:09:12.983Z
- Recorded fork run: bi3s3qgewfri
- Summary: Initial implementation fork funo4u3xhphr terminated mid-edit with an incomplete handoff after accidentally deleting the inspection service. No commit was created; intended changes remain safely in the dirty task worktree. Parent inspected without editing and launched continuation fork bi3s3qgewfri to audit/recover the partial implementation, recreate the service under the correct Session\Diagnostics namespace, validate, and commit without destructive operations.
- Fork funo4u3xhphr incomplete: dirty worktree contains provider recorder/plumbing/model-client/command/test edits; missing SessionPromptCacheInspectionService and inconsistent command import.
- Continuation fork bi3s3qgewfri launched with exact finalized scope, privacy/data-lifecycle constraints, constructor/call-site audit, focused Castor + live LLM validation, and no-reset instruction.

## Task workflow update - 2026-08-05T00:19:47.709Z
- Recorded fork run: bi3s3qgewfri
- Validation: Fork confirmed it read root AGENTS.md, `.agents/skills/testing/SKILL.md`, `tests/AGENTS.md`, and task spec before tests/QA.; `castor test --filter='PromptCacheRequestDiagnosticsRecorderTest|SanitizedGenericModelClientTest|CodexWebSocketCachedModelClientTest|SessionCacheInspectCommandTest'` — OK, 14 tests / 107 assertions.; `castor test:llm-real` — OK, 13 tests / 144 assertions.; `castor deptrac` — OK, 0 violations.; `castor phpstan` — OK, 0 errors.; `castor cs-check` — clean.; Parent verified HEAD/commit, exact 23-file stat, and clean worktree. `origin/main...HEAD` currently warns of multiple merge bases; intended task delta is still exactly the same 23 files. Repair topology non-destructively before task-to-PR review/gate.
- Summary: Implementation complete at commit 1a5df992952c693e0347cc1d330e740c90241b28 (23 files, +1663/-14), worktree clean. Adds privacy-safe provider-neutral request fingerprint recorder; generic/Codex SSE/WebSocket final-seam capture; optional diagnostics plumbing through invocation/result/event lifecycle including thinking-only retry merge; native `session:cache:inspect` command; parent/registered-child per-family inspection with persisted usage arithmetic, local-only prefix diffs, and honest historical fallback. Inspection service lives under AppAgent (`CodingAgent/Agent/Diagnostics`) because it consumes child artifact APIs and AppSession→AppAgent is forbidden; minimal AppCli→AppAgent Deptrac allowance added. No new event type, DB/migration, setting, export, UI, dependency, provider behavior, or compaction instrumentation. Task-start complete; no reviewer/PR/full gate run.
- Initial fork funo4u3xhphr ended mid-edit without commit after deleting the report service; partial changes were preserved.
- Continuation fork bi3s3qgewfri audited/recovered the dirty worktree without destructive operations, recreated the service, fixed namespace/dependency direction, validated, and committed as 1a5df9929.
- Task-start stopped after implementation/verification per workflow; task remains IN-PROGRESS.

## Task workflow update - 2026-08-05T00:38:15.744Z
- Summary: Task-to-PR review fallback returned REQUEST CHANGES at HEAD 98cda8d24: one blocking repository-policy violation in SessionPromptCacheInspectionService child-artifact degradation catch—Throwable is swallowed to empty events with a comment but without required structured diagnostic logging. All specification fidelity, privacy, wire invariance, arithmetic, family separation, attribution, command UX, and main test theses were otherwise judged sound.
- Reviewer subagent tool was attempted four times but returned no usable artifact/output; independent read-only review fork used as fallback.
- Single blocker: add privacy-safe structured warning logging for missing/corrupt child event reads at SessionPromptCacheInspectionService.php catch. No product-surface decision required.

## Task workflow update - 2026-08-05T02:17:15.937Z
- Validation: Reviewer re-review: APPROVED at task code HEAD 13a71c289; AppAgent service/AppCli dependency, event field, command, recorder/wire seams, arithmetic, family separation, history fallback, privacy, tests all accepted.; `castor test` — OK, 4447 tests / 16650 assertions.; `castor test:llm-real` — OK, 13 tests / 144 assertions.; `castor deptrac` — 0 violations.; `castor phpstan` — 0 errors.; `castor cs-check` — clean.; Post-validation origin/main docs-only advancement merged conflict-free; no task-code changes. Final topology HEAD 67d6d84b5; clean; exact 23-file two-dot/three-dot set.
- Summary: Task-to-PR review complete. Initial review REQUEST CHANGES found one silent child-artifact degradation catch; fixed in commit 13a71c289 with required privacy-safe structured warning and production-store corrupt-child regression. Reviewer re-review APPROVED with blocker resolved and no remaining scope/correctness/privacy issues. Branch synchronized non-destructively with current origin/main; final HEAD 67d6d84b50846ef208700f117ed12b7c015915b2, clean, one merge base, origin/main ancestor, exact 23-file task delta (+1740/-14). Ready for deterministic CODE-REVIEW gate and PR.
- Required reviewer subagent initially failed to return usable output across multiple attempts; read-only reviewer fork found one blocker. After fix, reviewer subagent returned a complete APPROVED re-review.
- Review fix commit 13a71c289: inject LoggerInterface, structured `session.cache_inspect.child_events_unavailable` warning, corrupt-child regression; 2 files +80/-3.
- Final origin/main topology sync commit 67d6d84b5 was conflict-free and docs-only.

## Task workflow update - 2026-08-05T02:19:29.080Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (121.5s).
- Pushed task/add-session-prompt-cache-diagnostics-command to origin.
- branch 'task/add-session-prompt-cache-diagnostics-command' set up to track 'origin/task/add-session-prompt-cache-diagnostics-command'.
- Created PR: https://github.com/ineersa/agent-core/pull/363

## Task workflow update - 2026-08-05T02:19:35.742Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/363
- Updated PR Status: open
- Validation: Deterministic `castor check` gate — passed in 121.5s.
- Summary: Moved to CODE-REVIEW. Deterministic castor check passed in 121.5s; branch pushed; PR #363 opened.

## Task workflow update - 2026-08-05T16:28:21.567Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User rejected PR #363 architecture as too intrusive. Approved lean redesign: CodingAgent-only low-priority Symfony AI InvocationEvent observer plus CodingAgent-owned privacy-safe diagnostics storage; inspector joins diagnostics with existing canonical usage events. Remove all AgentCore diagnostics/result/event plumbing and all provider model-client instrumentation. Exact Codex cached-WebSocket delta/previous_response_id/wire_input_count diagnostics explicitly dropped as acceptable while cache transport remains unreliable.

## Task workflow update - 2026-08-05T16:38:16.138Z
- Summary: User approved lean redesign and explicitly accepted dropping Codex cached-WebSocket `previous_response_id`, wire input count, and full/delta mode diagnostics. Final rewrite: no AgentCore source changes, no Platform/provider model-client changes, no diagnostics object threaded through metadata/results/messages. A CodingAgent Symfony AI InvocationEvent subscriber observes final shaped requests, persists only privacy-safe structural HMAC records to CodingAgent-owned sidecar JSONL, and the inspector joins those records with existing canonical LLM usage events.
- Single redesign scout confirmed InvocationEvent priority -100 sees final shaped MessageBag/options; priority +100 can retain existing invocation correlation before AgentCore strips metadata without mutating the event.
- Existing AgentChildRunDirectory + SessionAgentArtifactPathResolver can route parent/child sidecars under canonical session/artifact directories; no orphan global store required.
- Rewrite will remove all 7 AgentCore diffs, all provider/model-client diffs, mutable PromptCacheRequestDiagnosticsRecorder, DTO fields, worker/result-handler plumbing, and provider-specific tests.

## Task workflow update - 2026-08-05T18:22:20.745Z
- Summary: Lean rewrite completed locally at e0ec0d678 with zero AgentCore/Platform diffs and 8 changed files, but independent review returned REQUEST CHANGES for remaining dead/redundant diagnostic code. Current +1833 LOC is still larger than the rejected PR despite better ownership, so it is not acceptable yet under explicit user minimality feedback.
- Reviewer blockers/simplifications: delete six persisted-but-never-read record fields and whole-request canonical/HMAC work; remove redundant Symfony Lock around FILE_APPEND|LOCK_EX; remove duplicate read-path wrapper; remove speculative impossible UserMessage content branches; compare same-family components by HMAC only.
- Follow-up must also compact repetitive component construction/test setup so the final diff is materially smaller, while preserving privacy, parent/child routing, usage joins, and prefix comparison.

## Task workflow update - 2026-08-05T18:41:55.668Z
- Validation: Reviewer re-review: APPROVED; all prior dead-code/redundant-lock/correlation/minimality findings resolved.; `castor test` — OK, 4444 tests / 16631 assertions.; `castor test:llm-real` — OK, 13 tests / 144 assertions.; `castor deptrac` — 0 violations.; `castor phpstan` — 0 errors.; `castor cs-check` — clean.; Topology: origin/main ancestor; one merge base; two-dot=three-dot=8 intended files; worktree clean.
- Summary: User-approved CodingAgent-only rewrite complete at a0f98b1ca. Final delta is 8 files/+1446 (down from rejected 23 files/+1740): four production classes, one integration test, command allowlist/test, and three minimal Deptrac edges. Zero diffs under src/AgentCore, src/Platform, provider factories, model clients, DTOs, workers, or result handlers. Diagnostics are isolated to a dual-priority InvocationEvent subscriber and parent/child sidecar JSONL; exact Codex wire-mode fields remain intentionally dropped. Reviewer APPROVED after minimality iterate.
- Rewrite commit 1c9816641 removed all intrusive AgentCore/Platform/provider plumbing and replaced it with CodingAgent InvocationEvent observer + sidecars.
- Minimality commit a0f98b1ca removed dead record fields, whole-request HMAC work, redundant Symfony lock/read wrapper, speculative normalization branches, and compacted subscriber/service/test from +1833 to +1446.
- Final data flow: InvocationEvent +100 captures existing correlation, -100 fingerprints final shaped messages/tools without mutation, sidecar store routes parent/child, inspector joins with canonical usage by step_id and assigns usage only to last multi-record invocation.

## Task workflow update - 2026-08-05T18:43:54.300Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (106.4s).
- Pushed task/add-session-prompt-cache-diagnostics-command to origin.
- branch 'task/add-session-prompt-cache-diagnostics-command' set up to track 'origin/task/add-session-prompt-cache-diagnostics-command'.
- PR already exists: https://github.com/ineersa/agent-core/pull/363

## Task workflow update - 2026-08-05T18:44:17.681Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/363
- Updated PR Status: open
- Validation: Deterministic `castor check` after lean rewrite — passed in 106.4s.
- Summary: PR #363 rewritten and updated in place with the user-approved CodingAgent-only architecture. Deterministic castor check passed in 106.4s. Remote HEAD a0f98b1ca; PR title/body corrected to remove all stale claims about AgentCore events/provider seams.

## Task workflow update - 2026-08-05T20:06:11.304Z
- Moved CODE-REVIEW → DONE.
- Merged task/add-session-prompt-cache-diagnostics-command into integration checkout.
- Merge made by the 'ort' strategy.
 bin/console                                        |   1 +
 depfile.yaml                                       |   3 +
 .../PromptCacheDiagnosticsInvocationSubscriber.php | 293 +++++++++++++
 .../Diagnostics/PromptCacheDiagnosticsStore.php    | 113 +++++
 .../SessionPromptCacheInspectionService.php        | 457 +++++++++++++++++++++
 .../CLI/Session/SessionCacheInspectCommand.php     | 191 +++++++++
 tests/CodingAgent/CLI/ConsoleEntrypointUxTest.php  |   1 +
 .../CLI/Session/SessionCacheInspectCommandTest.php | 387 +++++++++++++++++
 8 files changed, 1446 insertions(+)
 create mode 100644 src/CodingAgent/Agent/Diagnostics/PromptCacheDiagnosticsInvocationSubscriber.php
 create mode 100644 src/CodingAgent/Agent/Diagnostics/PromptCacheDiagnosticsStore.php
 create mode 100644 src/CodingAgent/Agent/Diagnostics/SessionPromptCacheInspectionService.php
 create mode 100644 src/CodingAgent/CLI/Session/SessionCacheInspectCommand.php
 create mode 100644 tests/CodingAgent/CLI/Session/SessionCacheInspectCommandTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/add-session-prompt-cache-diagnostics-command.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/add-session-prompt-cache-diagnostics-command.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: PR #363 merged on GitHub as a898b8d412e9447d0dc4aeaba748b04dad5d811b. Moving task to DONE and synchronizing integration checkout.

## Task workflow update - 2026-08-05T20:08:45.485Z
- Updated PR Status: merged
- Validation: Post-merge `LLM_MODE=true castor check` passed: unit/integration 4444 tests/16631 assertions; controller replay 12/165; TUI 34/216; live LLM 13/144; Deptrac 0 violations; PHPStan 0 errors; CS clean; PHAR rebuild/smokes passed; QA artifact integrity, exact-run leak check/cache cleanup, and llama-proxy guard all passed (224→224).
- Summary: DONE. PR #363 merged as a898b8d412e9447d0dc4aeaba748b04dad5d811b; integration checkout synchronized; task worktree and IDEA exclusions removed; integration checkout clean.

## Task workflow update - 2026-08-06T20:58:47.480Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
