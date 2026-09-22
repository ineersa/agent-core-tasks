# Decouple Hatfield observability from the Datadog PHP extension

## Goal
User disabled the Datadog PHP extension after severe memory/CPU pressure associated with datadog-ipc-helper. Host telemetry captured ~93.55 GiB used, nearly exhausted swap and load >400. Screenshots showed helper RES ~32 GiB even after rollback to 1.18.0 and disabling profiling. Disabling the extension reduced CPU/memory use and improved perceived responsiveness, but the user reports missing logs and a half-empty dashboard. Exact root cause and which telemetry depended on the extension remain unverified.

Keep the extension disabled. Investigate missing logs and dashboard data, then use existing structured logging and Datadog Agent facilities to restore required observability without the PHP extension. Distinguish actual log loss from missing trace correlation or dashboard filters. Inventory dashboard dependencies before deciding what to replace.

User suggests publishing application metrics explicitly when Datadog is configured, potentially through DogStatsD/UDP. Evaluate existing project/framework facilities and Agent-supported transports before selecting an implementation. No finalized requirement for a new setting, transport, custom client, or replacement of all APM/profiling features. Ask about any unresolved product scope after the inventory.

Incident reference: https://github.com/DataDog/dd-trace-php/issues/4042 . Screenshots: .hatfield/sessions/1/attachments/pasted-image-1.png and .hatfield/sessions/7/attachments/pasted-image-1.png in the integration checkout. Prior exported telemetry: /tmp/hatfield-memory-20260905-1315-1321-edt/ (temporary, may disappear). Existing mitigation commit 50062bac7 disables profiling for castor run:agent; system PHP extension was subsequently disabled by user.

Do not re-enable the extension or reproduce the resource-exhausting workload as part of initial investigation.

## Acceptance criteria
- Inventory missing logs, metrics, trace correlation and affected dashboard widgets; identify their actual dependencies on the PHP extension.
- Preserve structured, privacy-safe application logging without requiring the Datadog PHP extension, using existing logging and Agent collection facilities.
- Propose the smallest explicit metric-publishing approach for configured Datadog environments, evaluating existing clients and DogStatsD support rather than inventing a transport.
- Keep Datadog optional: absent or unavailable telemetry infrastructure must not block application work or cause unbounded buffering, retries or resource growth.
- Implement only agreed telemetry requirements and document retained visibility and any intentional loss of APM/profiling features.
- Validate extension-free behavior with focused, bounded checks; do not re-enable the extension or rerun the exhausting workload merely to reproduce the incident.

## Workflow metadata
Status: DONE
Branch: task/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/492
PR Status: merged
Started: 2026-09-11T12:53:13+00:00
Completed: 2026-09-11T15:31:58+00:00

## Work log
- Created: 2026-09-05T18:08:16+00:00

## Task workflow update - 2026-09-11T12:53:13+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension.

## Task workflow update - 2026-09-11T12:53:33+00:00
- Summary: task-explain completed; user agreed to: (1) remove ddtrace-specific code paths (git history is the archive), (2) evaluate log-based application metrics derived from existing JSONL logs instead of spans/APM. New metric set requested: tool calls distribution, tool call error rate, edit tool error rate, LLM turns count/error rate, retries.
- Ownership: owner=main; fork_run=none; revision=6d59a21f1; scope=repo decoupling: remove ddtrace span provider, span contract, RunTracer provider plumbing, dd.* log correlation, APM launcher/QA env machinery, castor Datadog diagnostics cleanup, docs rewrite with log-based metric definitions; outcome=assigned; commit=none

## Task workflow update - 2026-09-11T13:07:44+00:00
- Validation: castor test --filter='RunTracerTest|LogContextProcessorTest' -> OK (10 tests, 37 assertions); castor test --filter='QaSessionEnvSanitizationTest|QaStandaloneTestCacheIsolationTest|DeadCodeBaselineRegenerationTest' -> OK (15 tests, 151 assertions); castor cs-check -> files_fixed=0; castor deptrac -> violations=0, errors=0; castor docs:validate -> OK; castor phpstan -> errors=0, file_errors=0; castor dead-code -> errors=0, file_errors=0; stale-reference grep: no ddtrace/SpanProvider/HATFIELD_DATADOG refs outside retirement notes and the QA env guard
- Summary: Repo decoupling implemented and committed. Deleted ddtrace integration paths (SpanProviderInterface, DdtraceSpanProvider, RunTracer provider plumbing, dd.* log injection), removed Castor APM auto-enable/HATFIELD_DATADOG machinery and ddtrace diagnostics, kept the QA guard against host ddtrace tracing/log injection, and rewrote docs/datadog.md + architecture/logging.md with retained visibility, intentional APM/profiling loss, and validated log-based metric definitions (tool calls, tool failures, edit tool error rate, LLM completed/failed, retries, durations). Datadog-side evaluation: log attributes and numeric duration measures verified against indexed logs; counts match local log baselines within ingestion/worktree variance; MCP publishes no log-based metric or facet creation tools, so metric publication needs Datadog UI or Logs Metrics API. Dashboard widget repointing remains pending and requires explicit authorization.
- Ownership: owner=main; fork_run=none; revision=6d59a21f1; scope=repo decoupling: remove ddtrace span provider, span contract, RunTracer provider plumbing, dd.* log correlation, APM launcher/QA env machinery, castor Datadog diagnostics cleanup, docs rewrite with log-based metric definitions; outcome=completed; commit=2a4b52dbd

## Task workflow update - 2026-09-11T13:35:34+00:00
- Validation: castor docs:validate -> exit 0 (revision 52de9bec8); castor cs-check -> exit 0, files_fixed=0 (revision 52de9bec8); castor test --filter='LogContextProcessorTest|RunTracerTest' -> OK (10 tests, 37 assertions) (revision 4d77cca22); Independent reviewer: APPROVE at revision 52de9bec8, specification fidelity holds, no unresolved blockers
- Summary: Review complete at revision 52de9bec8: independent reviewer verdict APPROVE after two fix rounds. Proceeding to the CODE-REVIEW transition gate (full castor check, push, PR).
- Review: role=reviewer; artifact=agent_a925f0176f46b0af; target_revision=52de9bec8; scope=origin/main...HEAD (12 files) specification-fidelity, correctness, test-quality, and docs-accuracy review in the task worktree; verdict=APPROVE; blockers=none. Three blocking findings fixed across revisions 4d77cca22 (verification command, llm.duration model grouping, turn-count metric) and 52de9bec8 (request-vs-turn record semantics, per-turn error-rate label).
- Ownership: owner=main; fork_run=none; revision=52de9bec8; scope=review fixes in docs/datadog.md, tests/CodingAgent/Logging/LogContextProcessorTest.php, src/CodingAgent/Logging/LogContextProcessor.php, .hatfield/skills/datadog/references/hatfield-telemetry.md; outcome=completed; commit=52de9bec8

## Task workflow update - 2026-09-11T13:36:33+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (52.9s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension/var/reports/qa-20260911-133540-3262-6bc5944c.
- Session/run: 40.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-11T13:36:35+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension/var/reports/qa-20260911-133540-3262-6bc5944c.
- Session/run: 40.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-11T13:36:41+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension/var/reports/qa-20260911-133540-3262-6bc5944c.
- Failed step: PR creation.
- Cause: gh is not authenticated: github.com
  X Failed to log in to github.com account ineersa (default)
  - Active account: true
  - The token in default is invalid.
  - To re-authenticate, run: gh auth login -h github.com
  - To forget about this account, run: gh auth logout -h github.com -u ineersa.
- Task remains IN-PROGRESS: IN-PROGRESS/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension.md.
- Session/run: 40.
- Next: restore GitHub authentication and inspect existing PRs before retrying. CODE-REVIEW retries run mandatory QA again; pushOnly skips PR creation, not QA.

## Task workflow update - 2026-09-11T13:36:50+00:00
- Summary: CODE-REVIEW transition partial failure at revision 52de9bec8. Completed: castor check passed (52.9s) and the task branch was pushed to origin. Failed: PR creation, because the gh CLI token for github.com account ineersa is invalid (re-authentication required). Task stays IN-PROGRESS. QA reports: var/reports/qa-20260911-133540-3262-6bc5944c in the task worktree. Session/run: 40. Safe next action: the user restores gh auth, then the transition is retried; a retry re-runs the mandatory QA gate and the push is idempotent, so nothing already completed is repeated blindly.
- Transition to CODE-REVIEW (revision 52de9bec8): partial failure. Completed steps: castor check passed (52.9s); branch task/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension pushed to origin. Failed step: PR creation. Bounded cause: gh CLI is not authenticated for github.com (invalid token for account ineersa; auth output redacted). Current task path/status: IN-PROGRESS/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension.md, IN-PROGRESS. QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension/var/reports/qa-20260911-133540-3262-6bc5944c. Session/run: 40. Safe next action: restore gh authentication (gh auth login -h github.com), then retry move_task(to=CODE-REVIEW); the retry re-runs the mandatory QA gate and re-pushes idempotently, and PR creation is the only remaining step.

## Task workflow update - 2026-09-11T13:42:49+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (52.6s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension/var/reports/qa-20260911-134156-8018-8ea7c78a.
- Session/run: 40.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-11T13:42:50+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension/var/reports/qa-20260911-134156-8018-8ea7c78a.
- Session/run: 40.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-11T13:42:52+00:00
- castor check passed (52.6s).
- Pushed task/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension to origin.
- Created PR: <url>
- Session/run: 40.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-11T13:42:52+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (52.6s).
- Pushed task/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/492
- Validation: gh auth status -> logged in as ineersa (keyring), no pre-existing PR for the branch; previous attempt: castor check passed (52.9s), branch pushed; this retry: mandatory castor check gate + PR creation
- Summary: CODE-REVIEW transition retried after GitHub auth was restored. Revision 52de9bec8, worktree clean, no pre-existing PR for the branch. Previous attempt had already passed castor check (52.9s) and pushed the branch; this retry re-runs the mandatory QA gate and creates the PR.

## Task workflow update - 2026-09-11T14:01:37+00:00
- Summary: Post-review Datadog follow-up completed with user authorization: dashboard xza-j7r-e4s rebuilt over MCP to log-query widgets, so no log-based metric publication is required for the dashboard. Repo doc sync for the new widget definitions is pending and needs a decision because the PR revision is frozen in CODE-REVIEW.
- Datadog dashboard update (authorized by user): dashboard xza-j7r-e4s rebuilt via MCP datadog_upsert_datadog_dashboard. Dropped the two widgets with no log source: 4309650939374206 (Database operations) and 6792767117984366 (Messenger consume). Replaced the six span-based widgets and added five new ones as log-query widgets: tool throughput grouped by tool, tool latency avg/pc95, LLM turns, LLM failures, LLM-step error rate, LLM latency p95, tool failures by tool, tool error rate, edit tool error rate, LLM failures by category, LLM retries. Retained the four working widgets with unchanged IDs/definitions (1954057201076375, 3448242938512668, 5473361968423792, 3097066923179713), plus title, description, tags, $env template variable, and fixed layout. All 11 changed/new widgets passed datadog_validate_dashboard_widget and the read-back verified the final inventory. Runtime data confirmed for 8 widgets; LLM-step errors, LLM failures by category, and LLM retries are empty in a now-1h window because those events are rare (4-9 per 24h), and read-only checks over now-24h/7d confirmed the queries work. Pending repo sync: .hatfield/skills/datadog/references/hatfield-dashboard.md and docs/datadog.md still describe the pre-rebuild dashboard state; the PR revision 52de9bec8 stays frozen in CODE-REVIEW, so the sync needs a follow-up task or a task-review-iterate pass.

## Task workflow update - 2026-09-11T14:58:18+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Review iteration: fix repo docs that still describe the pre-rebuild dashboard after the authorized Datadog-side widget rebuild. Files: docs/datadog.md and .hatfield/skills/datadog/references/hatfield-dashboard.md. Owner: main (docs-only, cohesive).

## Task workflow update - 2026-09-11T14:58:22+00:00
- Ownership: owner=main; fork_run=none; revision=52de9bec8 plus Datadog-side rebuild; scope=docs sync for the log-query dashboard rebuild in docs/datadog.md and .hatfield/skills/datadog/references/hatfield-dashboard.md; outcome=assigned; commit=none

## Task workflow update - 2026-09-11T15:06:02+00:00
- Validation: castor docs:validate -> exit 0 (revision 653bf76e5); castor cs-check -> exit 0 (revision 653bf76e5); Independent reviewer: APPROVE at revision 653bf76e5, specification fidelity holds, no unresolved blockers; Datadog-side: dashboard xza-j7r-e4s corrected and verified; six span queries now filter message:"agent_loop.trace.finish"; counts halved exactly (tool.call 4,404 -> 2,201; llm.call 3,174 -> 1,586); changed widgets validated, upserted, read back, and rendered
- Ownership: owner=main; fork_run=none; revision=653bf76e5; scope=docs sync for the log-query dashboard rebuild in docs/datadog.md and .hatfield/skills/datadog/references/hatfield-dashboard.md; outcome=completed; commit=653bf76e5
- Review iteration: role=reviewer; artifact=agent_a925f0176f46b0af; target_revision=653bf76e5; scope=git diff 52de9bec8..HEAD (2 docs files, no code) with specification-fidelity and factual-accuracy review against the recorded Datadog evidence; verdict=APPROVE; blockers=none. Non-blocking notes: the Tool execution latency widget title still says p50/p95 while it measures avg/p95 (flagged in the doc itself for the next dashboard touch); the archived .pi architecture report still lists the deleted SpanProviderInterface.

## Task workflow update - 2026-09-11T15:07:00+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (53.2s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension/var/reports/qa-20260911-150607-13690-e33f10cd.
- Session/run: 40.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-11T15:07:02+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension/var/reports/qa-20260911-150607-13690-e33f10cd.
- Session/run: 40.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-11T15:07:03+00:00
- castor check passed (53.2s).
- Pushed task/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension to origin.
- PR already exists: <url>
- Session/run: 40.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-11T15:07:03+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (53.2s).
- Pushed task/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/492
- Validation: castor docs:validate -> exit 0 (revision 653bf76e5); castor cs-check -> exit 0 (revision 653bf76e5); Independent reviewer: APPROVE at revision 653bf76e5; transition gate: full castor check + push + update PR #492
- Summary: Review iteration complete at revision 653bf76e5: docs synced to the log-query dashboard rebuild, independent reviewer APPROVE. Re-running the CODE-REVIEW transition gate to push and update PR #492.

## Task workflow update - 2026-09-11T15:31:21+00:00
- Process Check question investigated and closed without changes (user declined). Findings, for the record: Datadog has live system.processes.* for service:hatfield (16 processes, CPU 60-75%, RSS ~1.8 GB), but the series is a single aggregate per check instance tagged process_name:hatfield (the instance name, not an OS process name) with no command tag, so no per-process identity exists in the data. hatfield-safe is a 61-byte launcher script (exec $HOME/bin/pi-bwrap hatfield agent) and never persists as a process. The check's search strings (hatfield.phar, bin/console agent, bin/console messenger:consume) do not match the current launcher argv (/home/ineersa/.local/bin/hatfield ...) and a local replay of the check's matching rules found zero matches against this session's 15 processes, while Datadog counts 16; the covered process set could not be attributed from inside this bwrap sandbox. Proposed alignment of the check config (and optional per-role instances) was declined.

## Task workflow update - 2026-09-11T15:31:58+00:00
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension: ide_close_project returned isError.
- Merged task/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/env.php                                           | 119 +++-------------------------------------------
 .castor/run.php                                           |  21 +++-----
 .hatfield/skills/datadog/references/hatfield-dashboard.md | 413 ++++++++++++++++++++++++++++++++++++++++++--------------------------------------------------------------------------------------------------------------------
 .hatfield/skills/datadog/references/hatfield-telemetry.md |   7 +++
 architecture/logging.md                                   |  62 +++++++++++-------------
 docs/datadog.md                                           | 161 ++++++++++++++++++++++++++++++++++++++++++++++++++++++--------
 src/AgentCore/Application/Handler/RunTracer.php           |  48 +------------------
 src/AgentCore/Contract/SpanProviderInterface.php          |  41 ----------------
 src/CodingAgent/Logging/DdtraceSpanProvider.php           |  89 ----------------------------------
 src/CodingAgent/Logging/LogContextProcessor.php           |  19 +-------
 tests/AgentCore/Application/Handler/RunTracerTest.php     | 105 ++++++++--------------------------------
 tests/CodingAgent/Logging/LogContextProcessorTest.php     |   8 ++--
 12 files changed, 323 insertions(+), 770 deletions(-)
 delete mode 100644 src/AgentCore/Contract/SpanProviderInterface.php
 delete mode 100644 src/CodingAgent/Logging/DdtraceSpanProvider.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #492 state MERGED at 2026-09-11T15:31:34Z by ineersa, head ref task/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension; PRE-merge gate (CODE-REVIEW transition, revision 653bf76e5): castor check passed (53.2s); Independent reviewer APPROVE at revision 653bf76e5 (artifact agent_a925f0176f46b0af), specification fidelity holds, no unresolved blockers; Post-merge gate: castor check in the integration checkout (pending)
- Summary: Merged through PR #492. Delivers: removal of the Datadog PHP extension integration (SpanProviderInterface, DdtraceSpanProvider, RunTracer span plumbing, dd.trace_id/span_id log injection, HATFIELD_DATADOG launcher machinery), docs rewrite for extension-free observability, and the log-query dashboard rebuild on the Datadog side (no log-based metrics needed).

## Task workflow update - 2026-09-11T15:33:07+00:00
- Validation: Post-merge gate: castor check in integration checkout /home/ineersa/projects/agent-core -> exit 0, quality ok (151.9s); lanes all OK: test (4940 tests, 20746 assertions), test:controller-replay (11 tests), test:tui (9 tests), test:llm-real (5 tests), deptrac, phpstan (0 errors), dead-code (0), cs-check, docs:validate, catalog:version-check; QA run leak check ok and llama-proxy cache guard ok; Integration checkout HEAD 324b6a1a8 contains the task merge (88bb857ba, PR #492 merge 735159dc4); git status clean; Task worktree /home/ineersa/projects/agent-core-worktrees/2026-09-05-decouple-hatfield-observability-from-the-datadog-php-extension removed; IDEA exclusions removed; git worktree list no longer shows it; Minor degraded step: JetBrains project close for the worktree returned an error during the DONE transition; the worktree and exclusions were still removed
- task-done complete: PR #492 merged by ineersa at 2026-09-11T15:31:34Z; branch merged into the integration checkout, worktree removed, post-merge castor check green (exit 0).
