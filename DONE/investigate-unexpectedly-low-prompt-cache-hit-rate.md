# Investigate unexpectedly low prompt-cache hit rate

## Goal
Investigate the session/runtime cache metrics showing approximately `1883.6k/11.2k $10.52 ↻ 69% 31% 83.3k/272.0k`, where the displayed prompt-cache hit/reuse rate of 69% is unexpectedly low for a long Hatfield coding session. Determine exactly what each metric represents, verify the calculation and denominator, and identify which request prefixes are invalidating cache reuse. Analyze session 37 and representative child runs, including volatile system/developer context, injected project/skills/agent catalogs, date/CWD/session metadata, tool schemas and ordering, observational-memory/context updates, compaction, child launch snapshots, provider transport behavior, and model/provider changes. Distinguish genuine provider cache misses from incorrect telemetry or attribution. Avoid optimizing by removing required safety/context; preserve semantic correctness and privacy.

## Acceptance criteria
- Document the exact meaning and calculation of every displayed cache/token metric, including the 69% value
- Recover request-level cache-read/cache-write/input-token evidence for session 37 and reconcile it with the TUI totals
- Identify the dominant cache-busting request changes with quantified token impact
- Determine whether parent, scout, and fork requests are incorrectly aggregated across models/providers or transports
- Implement or propose the smallest safe stabilization of reusable prompt prefixes without changing required context semantics
- Add regression coverage for cache metric accounting and deterministic prompt-prefix stability across turns and child launches

## Workflow metadata
Status: DONE
Branch: task/investigate-unexpectedly-low-prompt-cache-hit-rate
Worktree: /home/ineersa/projects/agent-core-worktrees/investigate-unexpectedly-low-prompt-cache-hit-rate
Fork run: 7hoq78lgdyi0
PR URL: https://github.com/ineersa/agent-core/pull/357
PR Status: merged
Started: 2026-08-04T16:59:57.087Z
Completed: 2026-08-04T19:59:05.978Z

## Work log
- Created: 2026-07-31T20:48:51+00:00

## Task workflow update - 2026-08-04T16:59:57.087Z
- Moved TODO → IN-PROGRESS.
- Created branch task/investigate-unexpectedly-low-prompt-cache-hit-rate.
- Created worktree /home/ineersa/projects/agent-core-worktrees/investigate-unexpectedly-low-prompt-cache-hit-rate.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/investigate-unexpectedly-low-prompt-cache-hit-rate.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/investigate-unexpectedly-low-prompt-cache-hit-rate.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/investigate-unexpectedly-low-prompt-cache-hit-rate.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/investigate-unexpectedly-low-prompt-cache-hit-rate.
- Summary: User requested task be taken IN-PROGRESS and explained before implementation. Begin with read-only investigation of observed session/provider cache metrics and end-to-end prompt construction/caching seams; no code changes until findings and minimal scope are explained.

## Task workflow update - 2026-08-04T17:15:28.855Z
- Summary: Read-only forensic/code/provider audit complete; no implementation yet per user request to explain first. Session-37 checkpoint seq 787 exactly reconciles: 1,883,645 cumulative parent input → 1883.6k; 11,159 output → 11.2k; cache reads 1,296,384 / input 1,883,645 = 68.823% → ↻69%; latest input 83,346 / 272,000 = 30.642% → 31% 83.3k/272.0k. The adjacent 31% is context usage, not cache misses. Footer is parent-only; children are not aggregated. Latest checkpoint request itself was 41,472 / 83,346 = 49.8% cached, and parent cached prefix plateaued at 41,472 tokens after seq 241 while prompt grew to ~84k, so parent reuse really was limited. Child reuse was substantially higher: Codex children 77.0–93.7%, DeepSeek children 89.1–96.6%. Session 37 had no OM compaction; date/CWD/project context were stable; MCP catalog was empty. Different parent/scout/fork system prompts, tool policies, and model/provider namespaces intentionally cannot share one cache. Historical websocket continuation state is distinct from provider prompt caching; one-shot websocket still supports provider prefix caching. GPT-5.6 official semantics identify a likely actionable seam: implicit caching uses the latest user/tool breakpoint and does not fall back to the longest unmarked prefix; Hatfield sends stable prompt_cache_key but no explicit prompt_cache_breakpoint after its stable launch prologue. A separate definite telemetry bug was found: AiCostCalculator charges all input at full input price and then charges cached tokens again at cache-read price, inflating $10.52; correct billing must partition total input into uncached/read/write token classes. Compaction usage and failed/aborted attempt usage are also omitted, but broad accounting redesign is not yet authorized. Proposed minimal scope for approval: (1) correct cached-input cost partitioning; (2) add a Codex GPT-5.6 explicit breakpoint after each run archetype's stable system/user-context prologue while retaining default implicit caching as the longer-prefix opportunity; (3) document footer formula/scope and add focused accounting + deterministic breakpoint/prefix tests. Do not sort tools, freeze mutable safety context, merge provider caches, include children in parent footer, or alter required prompts without evidence.

## Task workflow update - 2026-08-04T17:40:08.465Z
- Summary: User rejected an unproven cache-breakpoint recommendation. Follow-up forensic narrowed historical facts: session 37 ran commit 1810fc1e7 with websocket-cached transport. At seq 241, after a 509s parallel-subagent turn, provider cached_tokens reached 41,472 of 42,186 input; every later parent request through seq 811 reported exactly 41,472 while logical input grew to 83,908. This is raw provider usage, not TUI clamping, global model ceiling (children exceeded it), child aggregation, model switching, or one-shot transport. Cached WebSocket compatible turns send previous_response_id + only delta input and deliberately remove prompt_cache_key; continuation mismatch/reset sends full input. Four later send failures reset sockets but successful subsequent calls still reported 41,472. Decisive historical transport logs/frames/response IDs did not survive, so the exact first provider-visible divergent segment after token 41,472 cannot be recovered retroactively. Treat this missing evidence as an observability defect, not an acceptable final conclusion. Revised investigation path before any cache optimization: privacy-safe HMAC fingerprints and byte boundaries for final full request instructions, ordered tool schemas, and ordered input items at existing Codex request-prepared seam, correlated with response input/cached tokens; compare current one-shot and opt-in cached transport under the same controlled live scenario. This will distinguish continuation/prompt_cache_key behavior from true message/tool-prefix instability without logging raw prompts. Do not add explicit breakpoints until evidence identifies the divergence. Cost-partition fix remains independently approved, but implementation should wait until user confirms whether to proceed alongside diagnostic reproduction.

## Task workflow update - 2026-08-04T17:55:59.591Z
- Recorded fork run: 7hoq78lgdyi0
- Summary: User finalized implementation scope: (1) websocket-cached continuation delta frames must retain the stable prompt_cache_key alongside previous_response_id instead of removing it; no explicit breakpoint/settings/transport redesign, (2) AiCostCalculator must partition total input into uncached, cache-read, and cache-creation/write tokens and bill each exactly once. Existing subagent/fork/agent_retrieve handoffs already receive the 50k document cap via OutputCapPathResolver, so no follow-up task was created. Implementation fork launched in task worktree; preserve unrelated untracked hatfield-session-1.html; focused existing-test modifications only, plus Castor phpstan/cs-check/test:llm-real; no castor check during task-start.

## Task workflow update - 2026-08-04T18:00:15.330Z
- Recorded fork run: 7hoq78lgdyi0
- Validation: castor test --filter=CodexWebSocketCachedModelClientTest: OK (7 tests, 45 assertions); castor test --filter=AiCostCalculatorTest: OK (7 tests, 7 assertions); castor phpstan: 0 errors; castor cs-check: clean after castor cs-fix; castor test:llm-real: OK (13 tests, 144 assertions)
- Summary: Implementation complete and committed as e248b81fa75d3c4e1534c6ad8c49615174870f96. Cached WebSocket continuation frames now retain the stable prompt_cache_key alongside previous_response_id and delta input. AiCostCalculator now partitions total input into clamped uncached/cache-read/cache-creation classes and bills each once, with explicit cache_read_tokens preferred over cached_tokens. Exactly 4 intended files changed (+54/-29): two production files and their existing tests; no docs/config/settings/breakpoints/new APIs. Required testing skill and tests/AGENTS.md were read. Unrelated untracked hatfield-session-1.html remains untouched and uncommitted.

## Task workflow update - 2026-08-04T19:44:48.871Z
- Validation: Reviewer: APPROVE WITH SUGGESTIONS at e248b81fa; no critical/bug/security issues; Fork follow-up castor test --filter=AiCostCalculatorTest: OK (8 tests, 8 assertions); Fork follow-up castor phpstan: 0 errors; Fork follow-up castor cs-check: clean
- Summary: Reviewer at e248b81fa returned APPROVE WITH SUGGESTIONS: no correctness/security blockers; requested qualifying the OpenAI-compatible token convention, restoring focused cached_tokens-alias fallback coverage, and improving test naming. Researcher verified official OpenAI Responses schema permits prompt_cache_key and previous_response_id together with no documented mutual exclusion, while noting no explicit behavioral guarantee/example for their combination. Fork addressed all actionable findings in commit 003ef02c79ee3ed8e6fd090f65ebc91020ebb245 (comment/test only); preserved untracked HTML at /tmp/hatfield-session-1.html with SHA-256 d0ce2de3fa8f5901e04422acc638e3c60e13b0d348b937fef4f2e07554ed80bd; worktree clean.

## Task workflow update - 2026-08-04T19:50:10.065Z
- Validation: Reviewer re-review: APPROVED at 003ef02c7; castor test: OK (4425 tests, 16428 assertions); castor deptrac: 0 violations, 0 errors; castor phpstan: 0 errors; castor cs-check: clean (0 files fixed); castor test:llm-real: OK (13 tests, 144 assertions)
- Summary: Final re-review at HEAD 003ef02c79ee3ed8e6fd090f65ebc91020ebb245: APPROVED. Reviewer confirmed finalized scope fidelity, mathematically sound clamped cost partitioning, official/current provider token convention, prompt_cache_key retention regression coverage, cached_tokens fallback coverage, no privacy/security issue, and lean 4-file diff. Branch is ready for deterministic CODE-REVIEW gate.
- Task-to-PR review cycle: e248b81fa received APPROVE WITH SUGGESTIONS; all actionable comment/test findings addressed by fork in 003ef02c7; re-review returned APPROVED. Official OpenAI schema research found prompt_cache_key and previous_response_id valid together with no documented mutual exclusion.

## Task workflow update - 2026-08-04T19:52:08.526Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (102.9s).
- Pushed task/investigate-unexpectedly-low-prompt-cache-hit-rate to origin.
- branch 'task/investigate-unexpectedly-low-prompt-cache-hit-rate' set up to track 'origin/task/investigate-unexpectedly-low-prompt-cache-hit-rate'.
- Created PR: https://github.com/ineersa/agent-core/pull/357
- Validation: Reviewer: APPROVED; castor test: OK (4425 tests, 16428 assertions); castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean; castor test:llm-real: OK (13 tests, 144 assertions)
- Summary: PR-ready at 003ef02c79ee3ed8e6fd090f65ebc91020ebb245. Cached WebSocket continuation deltas retain prompt_cache_key with previous_response_id; input cost accounting partitions uncached/cache-read/cache-write tokens exactly once. Reviewer APPROVED after one suggestion-fix iteration. Unrelated HTML snapshot preserved at /tmp/hatfield-session-1.html and worktree is clean.

## Task workflow update - 2026-08-04T19:52:14.334Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/357
- Updated PR Status: open
- Validation: move_task deterministic castor check: PASS (102.9s)
- Summary: Task-to-PR complete. Deterministic CODE-REVIEW gate passed in 102.9s; branch pushed and PR #357 created.

## Task workflow update - 2026-08-04T19:59:05.978Z
- Moved CODE-REVIEW → DONE.
- Merged task/investigate-unexpectedly-low-prompt-cache-hit-rate into integration checkout.
- Merge made by the 'ort' strategy.
 src/CodingAgent/Config/Ai/AiCostCalculator.php     | 61 ++++++++++++++--------
 .../OpenAICodex/CodexWebSocketModelClient.php      |  3 +-
 .../CodingAgent/Config/Ai/AiCostCalculatorTest.php | 36 ++++++++++---
 .../CodexWebSocketCachedModelClientTest.php        |  2 +
 4 files changed, 72 insertions(+), 30 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/investigate-unexpectedly-low-prompt-cache-hit-rate.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/investigate-unexpectedly-low-prompt-cache-hit-rate.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #357 state: MERGED
- Summary: PR #357 confirmed merged on GitHub at 2026-08-04T19:58:45Z (merge commit 88431c5187f2062009bb055f743b2b4190781813). Moving task to DONE and synchronizing integration checkout.

## Task workflow update - 2026-08-04T20:01:05.658Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/357
- Updated PR Status: merged
- Validation: Post-merge LLM_MODE=true castor check: PASS; unit/integration: 4425 tests, 16428 assertions; controller replay: 11 tests, 160 assertions; TUI replay: 32 tests, 204 assertions; llm-real: 13 tests, 144 assertions; deptrac: 0 violations; phpstan: 0 errors; cs-check: clean; llama-proxy cache stable: 223 → 223; QA leak check: OK; 6 exact-run cache roots removed; PHAR rebuilt and smoke tests passed
- Summary: DONE. PR #357 merged; task branch merged into integration checkout, remote changes synchronized, worktree removed, and post-merge full LLM_MODE Castor gate passed.
