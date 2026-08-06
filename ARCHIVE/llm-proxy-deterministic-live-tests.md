# Stabilize live LLM test prompts for llama-proxy replay cache

## Goal
User has created `/home/ineersa/projects/llama-proxy`, a request/response recording+replay proxy for the llama.cpp test model. Goal: live-LLM tests that currently use `llama_cpp_test/test` on port 9052 should route transparently through the proxy, record once, and replay quickly/deterministically on subsequent runs.

Read first:
- `/home/ineersa/projects/llama-proxy/README.md` and proxy implementation/config.
- `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before any test work.

Initial scout findings:
- Proxy cache key includes the full HTTP request body: method, path, query, JSON body (`messages`, `tools`, `model`, `temperature`, `stream`, etc.). Any prompt/tool/schema difference causes a miss.
- Agent-core live tests/config already target `http://192.168.2.38:9052/v1` for `llama_cpp_test/test`; proxy can listen on the same port and forward to upstream llama.cpp.
- Existing replay-backed controller/TUI tests use DI-level MockHttpClient fixtures and do not go through HTTP, so this task is specifically about live LLM smoke/recording paths (`castor test:llm-real`, `castor test:controller`, `castor llm:fixtures:record`) and any tests intentionally using the live llama_cpp provider.
- Prompt instability sources likely include system prompt `{date}` and `{cwd}`, AGENTS.md user-context blocks, skills registry/user-context, agents registry/catalog, APPEND_SYSTEM/prompt contributors, tool list/guidelines, and ordering of registries/discovery results.

Implementation intent:
- Make the portion of live-test LLM requests before the first real user message deterministic in test/proxy/recording mode, including replacing/removing date and cwd variability in the system prompt as requested.
- Make AGENTS.md, skills registry/context, and agents registry/catalog contributions deterministic enough that identical test scenarios produce identical request bodies across repeated runs and worktrees where practical.
- Preserve production/runtime behavior unless explicitly running in a test/recording/proxy deterministic mode; avoid broad compatibility shims or production-only APIs for tests.
- Document expected cache-miss boundaries: prompt text changes, tool schema changes, intentional system/developer prompt changes, and fixture/test scenario changes should still miss.

Potential affected areas from scout:
- `src/CodingAgent/SystemPrompt/SystemPromptBuilder.php` (`date`, `cwd`, tool/guideline/appends content)
- AGENTS discovery/context (`src/CodingAgent/SystemPrompt/AgentsContextDiscovery.php` and message construction paths)
- Skills/agents registry discovery and prompt/context contributors
- Live test provider settings under `.hatfield/settings.yaml` and test-local settings in TUI/controller E2E helpers
- Castor live LLM readiness/preflight/cleanup helpers under `.castor/`
- Existing live tests around `#[Group('llm-real')]`, `ControllerE2eTestCase`, `ReplayRecordingTest`

## Acceptance criteria
- A deterministic live-test/proxy mode exists for llama_cpp test traffic so repeated identical `llama_cpp_test/test` requests produce byte-identical JSON request bodies before the first real user message across repeated runs; date and cwd no longer vary in that prefix.
- AGENTS.md context, skills registry/context, and agents registry/catalog content/order are deterministic in that mode, or explicitly normalized/replaced with stable fixtures/placeholders where appropriate.
- The proxy remains transparent on port 9052 and live LLM tests still target `llama_cpp_test/test`; first run records through `/home/ineersa/projects/llama-proxy`, second identical run replays from cache and is observably faster.
- Add focused tests or diagnostics proving the request-prefix stability/caching contract without over-mirroring implementation details.
- Update project docs/testing guidance (for example `docs/llm-replay.md`, `tests/AGENTS.md`, or relevant Castor help) with how to run the proxy, record cache entries, inspect stats, clear cache, and known cache-miss causes.
- Run required validation through Castor only: at minimum focused tests for changed code, `LLAMA_CPP_SMOKE_TEST=1 castor test:llm-real` twice with proxy stats/timing evidence, and deterministic `castor check` if runtime/prompt/TUI/controller flow is touched; record any blockers such as proxy/model availability.

## Workflow metadata
Status: DONE
Branch: task/llm-proxy-deterministic-live-tests
Worktree: /home/ineersa/projects/agent-core-worktrees/llm-proxy-deterministic-live-tests
Fork run: kffyfc9l6u9y
PR URL: https://github.com/ineersa/agent-core/pull/202
PR Status: merged
Started: 2026-06-23T21:40:55.663Z
Completed: 2026-06-23T23:33:29.900Z

## Work log
- Created: 2026-06-23T21:39:39.330Z

## Task workflow update - 2026-06-23T21:40:55.663Z
- Moved TODO → IN-PROGRESS.
- Created branch task/llm-proxy-deterministic-live-tests.
- Created worktree /home/ineersa/projects/agent-core-worktrees/llm-proxy-deterministic-live-tests.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/llm-proxy-deterministic-live-tests.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/llm-proxy-deterministic-live-tests.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/llm-proxy-deterministic-live-tests.

## Task workflow update - 2026-06-23T21:41:25.929Z
- Recorded fork run: onpwn8zad338
- Moved TODO → IN-PROGRESS and launched implementation fork on worktree `/home/ineersa/projects/agent-core-worktrees/llm-proxy-deterministic-live-tests`. Fork instructed to include user clarification: exclude `.hatfield/APPEND_SYSTEM.md` / local append-system prompt content from deterministic live-test prompts.

## Task workflow update - 2026-06-23T21:45:33.428Z
- Recorded fork run: onpwn8zad338
- Validation: Fork: `castor test --filter=LlmProxyDeterministicPromptContractTest` — OK (2 tests, 7 assertions).; Fork: `castor test --filter='LlmProxyDeterministicPromptContractTest\|SystemPromptBuilderTest'` — tests reportedly OK (23 tests) but Castor wrapper reported FAILED; needs parent/follow-up verification.; Fork: `castor test:llm-real` twice with proxy stats — not run.; Fork: scoped `castor phpstan` / `castor cs-check` — attempted with wrong CLI; not valid.
- Summary: Implementation fork completed partially and committed core deterministic live-test/proxy prompt behavior on branch `task/llm-proxy-deterministic-live-tests`, commit `25689ae1f`. Fork reports: read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`; added `LlmProxyDeterministicPromptMode`; deterministic mode removes date, preserves real cwd, excludes APPEND_SYSTEM/prompt contributors, stabilizes AGENTS and skills discovery/display paths, and sets `HATFIELD_LLM_PROXY_DETERMINISTIC=1` for live controller E2E. Docs updated with llama-proxy section. Gaps: no double `castor test:llm-real` proxy replay proof; no full phpstan/cs-check due wrong CLI attempt; possible remaining cwd-based cache misses across isolated test dirs are intentional/open.

## Task workflow update - 2026-06-23T21:53:36.141Z
- Recorded fork run: ctlgjewc6v5s
- User implemented cache-side prompt stripping in llama-proxy and requested reverting prior app-side deterministic prompt changes, then running live LLM tests multiple times to verify proxy cache speedup. Launched follow-up fork `ctlgjewc6v5s` in worktree with instructions to non-destructively revert commit `25689ae1f`, commit the revert, and run `LLAMA_CPP_SMOKE_TEST=1 castor test:llm-real` multiple times with proxy health/stats/timing evidence.

## Task workflow update - 2026-06-23T22:00:47.869Z
- Recorded fork run: ctlgjewc6v5s
- Validation: Fork read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` before test work.; `git revert --no-edit 25689ae1f` — OK; created commit `9c5b3d796`; working tree clean.; Proxy health on `127.0.0.1:9052` and `192.168.2.38:9052` — OK; `cache_normalize_messages: true`.; Proxy stats baseline/attempts showed entries growing on first passes (`0 → 4 → 10–12`) and stable entry counts on repeated identical filtered controller smoke runs.; `LLAMA_CPP_SMOKE_TEST=1 castor test:llm-real` with temporary 600s timeout — FAIL after ~94s/~69s/~66s due live failures (`WriteFileToolE2eTest`, `OutputCapReadFileControllerTest`); timeout tweak was restored and not committed.; `LLAMA_CPP_SMOKE_TEST=1 castor test:llm-real --filter=ControllerSmokeTest` twice — OK; Castor wall time improved ~9.5s → ~6.7s with proxy stats unchanged at 12 entries, indicating cache hits.; Earlier `LlamaCppSmokeTest::testRealLlamaCppInvocation` repeated 3x — OK ~24.5s each; stats stable, showing cache hit but most time is harness/PHAR/preflight overhead.; Current-user stale worker process check — none killed.
- Summary: Follow-up fork completed. It non-destructively reverted app-side deterministic prompt implementation commit `25689ae1f` with revert commit `9c5b3d796`, restoring agent-core behavior to `origin/main` for system prompt, AGENTS/skills discovery, controller E2E env, docs, and tests. Fork verified llama-proxy is healthy with cache-side message normalization enabled (`cache_normalize_messages: true`), stripping leading system/developer and `[user-context]` prologue for cache keys. Conclusion: cache-side normalization is sufficient; no app-side deterministic prompt mode should be reintroduced unless future evidence shows insufficiency. Full `castor test:llm-real` remains blocked by live test failures/timeouts unrelated to the revert.

## Task workflow update - 2026-06-23T22:02:33.555Z
- Recorded fork run: jyej936q1r5m
- User requested fixing failing live LLM tests and optimizing them for faster runtime with llama-proxy cache normalization. Launched fork `jyej936q1r5m` to investigate/fix `WriteFileToolE2eTest`, `OutputCapReadFileControllerTest`, and other llm-real failures; optimize tests by using exact prompts and narrow event waits; run focused and full `LLAMA_CPP_SMOKE_TEST=1 castor test:llm-real` with proxy stats/timing; and commit changes.

## Task workflow update - 2026-06-23T22:10:31.283Z
- Recorded fork run: jyej936q1r5m
- Validation: Fork `jyej936q1r5m` reported two full `castor test:llm-real` runs with stable proxy entries/bytes on the second pass (cache hits) but wall-clock variance (~74s vs ~92s).; Fork `jyej936q1r5m` reported earlier filtered repeat evidence (`ControllerSmokeTest`) with unchanged entries and lower Castor time (~9.5s → ~6.7s).; Fork `jyej936q1r5m` handoff did not include a commit SHA for the optimization changes; parent inspection found the worktree dirty, so final validation/commit remains pending in fork `p096l1iaa0yu`.
- Summary: Fork `jyej936q1r5m` reported analysis/validation for live LLM test optimization but left the worktree with uncommitted changes. Reported conclusion: stable proxy cache stats between two full `castor test:llm-real` runs means no new cassettes were recorded on the second pass; slower wall time on second full run is explained by non-LLM overhead (PHAR ensure, preflight, controller/worker startup, tool execution, polling, session I/O, teardown, replay chunking, and run-to-run variance). Reported optimization direction: fewer LLM turns, narrow tool-completion waits, unique first user prompts to avoid normalized-key collisions, and avoid clearing cache during repeated CI/debug runs. Because the fork did not commit and the handoff lacked exact commit/status, a finalization fork `p096l1iaa0yu` was launched to inspect, validate, and commit or fix the uncommitted changes.
- Observed uncommitted changes after fork `jyej936q1r5m`: `.castor/e2e.php`, `tests/AGENTS.md`, `tests/AgentCore/Infrastructure/SymfonyAi/LlamaCppSmokeTest.php`, and several live controller E2E tests. Changes appear to add unique `[llm-real:*]` prompt prefixes, add `liveLlmToolWaitTimeout(): 25.0`, use that helper for tool completion waits, and raise llm-real Castor timeout to 180s. Launched finalization fork `p096l1iaa0yu` to review, run focused Castor validation, and commit finalized changes.

## Task workflow update - 2026-06-23T22:14:29.381Z
- User noted cache-hit proxy replays are fast but `castor test:llm-real` remains slow, likely due sleeps/test design. Ran read-only scout performance audit while finalization fork `p096l1iaa0yu` is active; did not launch a conflicting edit fork against the same dirty worktree. Key audit findings to feed into next iteration: (P0) controller live subprocess uses `APP_ENV=dev`/`APP_DEBUG=1`, so `services_test.yaml` 5s HTTP timeout and test-mode performance are not applied; likely safe to use `APP_ENV=test`, `APP_DEBUG=0` because replay HTTP client only activates when `HATFIELD_LLM_REPLAY_FIXTURE_PATH` is set. (P1) `check_llm_generation_ready()` runs a full curl generation before every `test:llm-real`; add short TTL cache to avoid repeated ~4-7s preflight cost in back-to-back runs. (P2) individual live wait budgets are broad: proposed lower `liveLlmToolWaitTimeout` from 25s toward ~12s, compaction 30s loops toward ~15s, controller smoke 15s toward ~8s once stability verified. (P3) `ShellFollowUpLiveE2eTest::collectRaw()` waits full timeout and causes fixed delay; replace with early-exit helpers such as `collectEventsUntilToolCompleted('ls', ...)` and `collectEventsUntil('assistant.text_started', ...)`. (P4) sequential controller subprocess startup dominates; larger refactor to shared controller process is possible but higher risk. (P5) PHAR ensure/preflight overhead can be optimized for filters/lazy paths.

## Task workflow update - 2026-06-23T22:14:54.096Z
- Recorded fork run: p096l1iaa0yu
- Validation: Parent inspection: `git status --short` clean in worktree.; Parent inspection: HEAD `d60f81717` on branch `task/llm-proxy-deterministic-live-tests`.; Parent inspection: net diff vs `origin/main` is 10 files, 23 insertions, 12 deletions.; Fork reported proxy healthy; cache still has 15 entries (~2.7MB).; Fork recommended sanity check: `LLAMA_CPP_SMOKE_TEST=1 castor test:llm-real` after zero-latency proxy restart; not yet captured by parent.
- Summary: Finalization fork committed the previously uncommitted live LLM stabilization changes as `d60f81717` (`test: stabilize live llm smoke tests for proxy replay`). Worktree is clean. Net diff vs `origin/main` now includes unique `[llm-real:*]` prompt prefixes, `liveLlmToolWaitTimeout()`, updated tool waits, `test:llm-real` Castor timeout to 180s, and tests guidance. Fork output was minimal, but parent inspection confirms commit and clean tree. Proxy was reported healthy with 15 warm cache entries (~2.7MB) and zero-latency replay restart (`LLAMA_PROXY_REPLAY_TPS=0`).

## Task workflow update - 2026-06-23T22:15:17.237Z
- Recorded fork run: com5oikbqfj1
- Launched performance-fix fork `com5oikbqfj1` from clean commit `d60f81717` to address remaining slow live tests despite fast cache-hit proxy replays. Fork instructed to implement targeted fixes from audit: run controller subprocess under `APP_ENV=test`/`APP_DEBUG=0` if safe, add TTL cache for `check_llm_generation_ready()`, reduce broad waits where stable, replace `ShellFollowUpLiveE2eTest::collectRaw()` fixed full-timeout waits with early-exit helpers, validate with focused/full `castor test:llm-real`, proxy stats, and commit changes.

## Task workflow update - 2026-06-23T22:16:15.295Z
- User corrected performance scope: do NOT switch controller live subprocess from `APP_ENV=dev`/`APP_DEBUG=1` to `APP_ENV=test`/`APP_DEBUG=0`; keep current dev/debug env because it is not expected to materially affect runtime. If fork `com5oikbqfj1` makes this env change, parent/follow-up must revert it before accepting the work. Focus should remain on real sleeps/broad waits/preflight/replay latency/test design.

## Task workflow update - 2026-06-23T22:19:45.232Z
- Recorded fork run: com5oikbqfj1
- Validation: Fork reported proxy health OK and cache normalization enabled.; Fork reported `LLAMA_CPP_SMOKE_TEST=1 castor test:llm-real --filter=ControllerSmokeTest` — OK ~4.8s wall.; Fork reported `LLAMA_CPP_SMOKE_TEST=1 castor test:llm-real --filter=ShellFollowUpLiveE2eTest` — OK ~11.5s wall.; Fork reported `LLAMA_CPP_SMOKE_TEST=1 castor test:llm-real --filter=CompactionLiveSmokeTest` — OK with 25s compaction wait.; Fork reported full `LLAMA_CPP_SMOKE_TEST=1 castor test:llm-real` — OK 8 tests, 92 assertions, ~48s Castor wall.; Fork reported `castor cs-check` — OK.; Caveat: validation includes disallowed env/source-console change, so follow-up validation is needed after reverting that portion.
- Summary: Performance fork `com5oikbqfj1` completed and committed `d29cdb5f6`, reporting full `LLAMA_CPP_SMOKE_TEST=1 castor test:llm-real` green in ~48s and `castor cs-check` OK. However, the fork ignored/misread the user's correction: user explicitly said to keep controller live subprocess `APP_ENV=dev` / `APP_DEBUG=1`, but the fork changed live controller E2E to source `bin/console` with `APP_ENV=test` / `APP_DEBUG=0` and removed PHAR ensure for `test:llm-real`. Parent should not accept that env/source/PHAR portion as-is; launch follow-up to revert those parts while preserving valid speed fixes (preflight TTL, unique prompts, narrower waits, ShellFollowUp early-exit) and revalidate.

## Task workflow update - 2026-06-23T22:21:57.722Z
- User clarified latest position on controller subprocess env: `APP_ENV=test`/source console is acceptable if it works, but keep `APP_DEBUG=1` so future failures expose useful errors. Prior task note saying to reject any APP_ENV change is superseded only for APP_ENV; debug must be restored to 1. User also flagged that ~48s is still too slow with instant proxy replay and expected parallel test execution; investigate/optimize actual llm-real parallelism and fixed waits.

## Task workflow update - 2026-06-23T22:26:12.971Z
- User specifically flagged compaction live test as ~30s and way too slow. Fork `z1ay3hiosjqx` should prioritize optimizing `CompactionLiveSmokeTest`: measure where time is spent, reduce/replace broad 25–30s waits with narrower early-exit/event proof if possible, and only keep a long timeout if actual measured async worker latency proves it is necessary.

## Task workflow update - 2026-06-23T22:28:33.430Z
- Recorded fork run: z1ay3hiosjqx
- Validation: Fork reported baseline sequential full `castor test:llm-real` ~54s wall / PHPUnit ~52s.; Fork reported per-test sequential JUnit: `CompactionLiveSmokeTest` ~26s; others ~3–10s.; Fork reported full `LLAMA_CPP_SMOKE_TEST=1 castor test:llm-real` using ParaTest --processes=4 — OK 8/8, ~22.1s wall / PHPUnit ~21.0s.; Fork reported warm repeat full `LLAMA_CPP_SMOKE_TEST=1 castor test:llm-real` — OK ~24.7s wall.; Fork reported filtered `ControllerSmokeTest` — OK ~4.7s wall.; Fork reported filtered `CompactionLiveSmokeTest` — OK with compaction wait restored to 25s after 18s flaked under ParaTest.; Fork reported proxy health OK; cache entries 18→27; `cache_normalize_messages: true`; preflight cache showed `ok (cached, ttl=120s)`.; Fork reported `castor cs-check` — OK.
- Summary: Performance/parallelization fork `z1ay3hiosjqx` completed and committed `e9b9c9d12`. It restored live controller subprocess `APP_DEBUG=1` while keeping the working `APP_ENV=test` source-console path; made full `castor test:llm-real` use ParaTest with 4 processes while keeping filtered runs sequential; trimmed a redundant compaction event drain but retained 25s compaction wait because 18s flaked under parallel load; documented the behavior in `tests/AGENTS.md`. Fork confirmed the main slowness was sequential PHPUnit plus compaction async latency: prior full group was a single phpunit process (~48–54s); after ParaTest warm full runs are ~22–25s. Testing skill and `tests/AGENTS.md` were read per fork handoff.

## Task workflow update - 2026-06-23T22:30:46.521Z
- User accepted current ~22–25s `test:llm-real` result and requested putting the `llm-real` group back into `castor check`. User also asked whether replay infrastructure should be removed now that llama-proxy caching exists. Parent assessment: do not remove replay infrastructure wholesale in this task; proxy cache accelerates live-provider smoke tests but is not equivalent to committed replay fixtures/offline deterministic CI. Consider pruning/retiring specific redundant replay layers only in a separate follow-up after `castor check` with `llm-real` proves stable.

## Task workflow update - 2026-06-23T22:34:28.977Z
- Recorded fork run: 5j6enrqbal76
- Validation: Fork reported proxy health OK, `cache_normalize_messages: true`, cache 28 entries (~8.7MB).; Fork reported `castor cs-check` — OK.; Fork reported `castor check` — all lanes including new `test:llm-real` OK (`test:llm-real` 33.6s), but overall failed due phpstan at `.castor/helpers.php:936`.; Fork reported `castor test:llm-real` — OK 8/8, 24.2s wall.
- Summary: Fork `5j6enrqbal76` completed request to add `llm-real` back into `castor check` and committed `69aaf34d8`. It extracted `build_test_llm_real_phpunit_command(?string $filter)` in `.castor/e2e.php`, added a `test:llm-real` lane to `.castor/tasks.php` using the same ParaTest --processes=4 command with 180s timeout, and runs `check_llm_generation_ready()` once before parallel check lanes. Docs updated in AGENTS/testing/test docs to reflect that `castor check` now requires live generation on 9052. Testing skill and `tests/AGENTS.md` were read per fork handoff. Caveat: full `castor check` did not exit green because phpstan failed on `.castor/helpers.php:936`, reported as pre-existing branch debt; follow-up needed before CODE-REVIEW gate.

## Task workflow update - 2026-06-23T22:34:52.175Z
- Recorded fork run: wf4dc1o4yzai
- Launched follow-up fork `wf4dc1o4yzai` from `69aaf34d8` to unblock CODE-REVIEW validation: fix phpstan failure in `.castor/helpers.php:936`, update stale `docs/qa-metrics.md` claims about `castor check` being no-live-LLM, preserve `APP_DEBUG=1` and ParaTest `llm-real` lane, run `castor phpstan`, `castor cs-check`, and full `castor check` if feasible, then commit.

## Task workflow update - 2026-06-23T22:37:14.739Z
- Recorded fork run: wf4dc1o4yzai
- Validation: Fork reported `castor phpstan` — OK, no errors.; Fork reported `castor cs-check` — OK, 0 files to fix.; Fork reported full `castor check` — quality: ok (196.0s).; Fork reported check lane timings: deptrac 0.9s OK; test 19.3s OK (3482 tests); test:controller-replay 60.2s OK (7 tests); test:tui 81.5s OK (16 tests); test:llm-real 29.0s OK (8 tests, 92 assertions); phpstan 2.0s OK; cs-check 3.2s OK.; Fork reported preflight `llama.cpp generation: ok` (not cached).
- Summary: Follow-up fork `wf4dc1o4yzai` completed and committed `b1cfd9e85`, fixing the phpstan blocker introduced by the LLM readiness TTL cache and aligning docs with current behavior: `castor check` now includes a live `test:llm-real` lane on port 9052/proxy. It preserved `APP_DEBUG=1`, ParaTest llm-real behavior, and did not remove replay infrastructure. It updated `.castor/helpers.php`, `docs/qa-metrics.md`, `docs/compaction.md`, and `.agents/skills/testing/SKILL.md`. Testing skill and `tests/AGENTS.md` were read per fork handoff.

## Task workflow update - 2026-06-23T22:42:19.496Z
- Read-only scout investigated slow `castor check` lanes after user flagged `test:tui` (~81.5s) and `test:controller-replay` (~60.2s). Findings: controller-replay is dominated by per-test controller subprocess lifecycle, especially `ControllerReplayE2eTestCase::stopProcessWithOwnership()` 3s SIGTERM grace + 100ms reap across 7 tests (~20s+ teardown, plus Symfony/controller boot). Several replay compaction tests have broad 25–30s event windows and 250ms post-death sleeps, though many exit early. TUI is dominated by 16 separate tmux/Symfony app sessions; fixed `usleep()` calls across TUI tests are only ~2–3s, while process startup/replay/user-flow work dominates. Quick wins: shorten controller replay teardown grace if safe, remove 250ms post-death sleeps, add idle-drain early-exit/narrow compaction replay waits, replace small TUI fixed sleeps with polling. Bigger win: shared/reused controller/TUI process would save more but is higher-risk due isolation.

## Task workflow update - 2026-06-23T22:44:12.203Z
- Recorded fork run: yglmnkdtf6px
- User approved trying low-risk speed optimizations for slow `castor check` lanes after scout findings. Launched fork `yglmnkdtf6px` from `b1cfd9e85` to implement/measure safe wins: shorten controller replay teardown grace if robust, remove fixed 250ms post-death sleeps, improve/narrow compaction replay drain idle exits, optionally replace obvious TUI fixed sleeps with polling, validate with `castor test:controller-replay`, `castor test:tui`, `castor check` if feasible, and `castor cs-check`, then commit.

## Task workflow update - 2026-06-23T22:46:18.151Z
- Recorded fork run: gxe2iemex8az
- User requested a second concurrent fork to update knowledge/docs about the new llama-proxy workflow while performance fork `yglmnkdtf6px` runs. Launched documentation-only fork `gxe2iemex8az` to update `AGENTS.md`, `tests/AGENTS.md`, and `.agents/skills/testing/SKILL.md` with verified proxy health/stats/reset/warm/regenerate commands, `castor check` live `llm-real` behavior, proxy cache normalization, replay-vs-proxy distinction, and preflight TTL notes. Fork instructed to inspect `/home/ineersa/projects/llama-proxy` for exact endpoint/source evidence and avoid clobbering concurrent performance-fork edits.

## Task workflow update - 2026-06-23T22:47:29.835Z
- Recorded fork run: gxe2iemex8az
- Validation: Fork reported `castor cs-check` — OK, 0 files to fix.; Fork skipped full `castor check` because changes were docs-only and concurrent performance fork had uncommitted test edits.; Proxy endpoint evidence verified by fork: `GET /__llama_proxy/health`, `GET /__llama_proxy/cache/stats`, `POST /__llama_proxy/cache/clear`, and `DELETE /__llama_proxy/cache`.; Fork noted replay responses set `x-llama-proxy-cache: hit` and cache normalization drops leading system/developer and `[user-context]` messages from cache key while upstream miss uses full body.
- Summary: Documentation fork `gxe2iemex8az` completed and committed `d23e3388a`, updating `AGENTS.md`, `tests/AGENTS.md`, and `.agents/skills/testing/SKILL.md` with verified llama-proxy workflow: admin health/stats/clear endpoints, cache normalization behavior, reset/warm/regenerate guidance, `castor check` live `llm-real` lane behavior, preflight TTL, and proxy-cache vs committed replay fixture distinctions. Fork verified endpoints from `/home/ineersa/projects/llama-proxy/README.md` and `llama_proxy/app.py`. It intentionally did not touch uncommitted performance-test edits from concurrent fork `yglmnkdtf6px`.

## Task workflow update - 2026-06-23T22:51:54.157Z
- Recorded fork run: yglmnkdtf6px
- Validation: Fork reported baseline `castor test:controller-replay` — OK, PHPUnit 56.492s / wall 57.58s.; Fork reported optimized `castor test:controller-replay` — OK, PHPUnit 41.216s / wall 42.24s (~15s faster).; Fork reported baseline `castor test:tui` — OK, PHPUnit 76.046s / wall 77.02s.; Fork reported optimized `castor test:tui` — OK, PHPUnit 78.062s / wall 79.07s (no material improvement).; Fork reported `castor cs-check` — OK, 0 fixes.; Fork reported full `castor check` partial: deptrac/test/controller-replay/tui/phpstan/cs-check OK; `test:llm-real` failed in 10.6s due 3× `runtime.ready` 5s timeout under full parallel check.; Fork reported immediate retry `castor test:llm-real` — OK 8/8, 24.699s.; Open validation: rerun full `castor check` after this commit stack to confirm green before CODE-REVIEW; if llm-real flake repeats, investigate runtime.ready timeout under full check parallelism.
- Summary: Performance fork `yglmnkdtf6px` completed and committed `26143032e`, implementing low-risk check-lane speedups. Controller replay teardown is now zombie-aware, SIGTERM grace reduced from 3.0s to 1.0s, and pre-`proc_close` 100ms sleep removed. Replay compaction drain helpers removed fixed 250ms post-death sleeps and gained 0.8s idle-drain exit only after run terminal state is seen, preserving compaction semantics after an initial too-aggressive attempt failed. `TuiJourneyE2eTest` replaced one 500ms startup sleep with polling, but TUI timing stayed within noise. Testing skill and `tests/AGENTS.md` were read per fork handoff.

## Task workflow update - 2026-06-23T22:59:06.253Z
- Recorded fork run: 62pevwvk0jog
- User said flaking is not acceptable and questioned why `test:tui` remains slow. Launched fork `62pevwvk0jog` from `26143032e` to robustly fix the full-check `test:llm-real` `runtime.ready` flake (likely by replacing hard-coded 5s live controller ready waits with a dedicated early-exit helper using a larger startup budget, without increasing run/tool waits), rerun full `castor check`, and produce concrete TUI per-test/class timing breakdown with any safe low-risk TUI speed wins.

## Task workflow update - 2026-06-23T23:08:17.790Z
- Recorded fork run: 62pevwvk0jog
- Validation: Fork reported `castor test:llm-real` — OK 8/8, ~22.2s wall.; Fork reported `castor test:tui` — OK 16/16, ~71.1–71.4s wall after TUI edits.; Fork reported `castor cs-check` — OK, 0 fixes.; Fork reported full `castor check` #1 — failed in `test:tui` due `CancelStickinessE2eTest` tight timing, while `test:llm-real` was OK 26.4s; fork fixed cancel waits.; Fork reported full `castor check` #2 — quality ok (173.0s): test:tui 80.2s, test:llm-real 26.6s, controller-replay 40.2s.; Fork reported full `castor check` #3 — quality ok (171.5s): test:tui 76.4s, test:llm-real 30.3s, controller-replay 39.2s.; Fork reported preflight cached on later checks (`ok (cached, ttl=120s)`).
- Summary: Fork `62pevwvk0jog` completed and committed `4723a7733`, fixing the unacceptable full-check `test:llm-real` flake and producing a concrete TUI timing breakdown. Live controller E2E tests now use `liveControllerReadyTimeout()` = 12s for `runtime.ready` (early-exit, so passing tests are not slowed) instead of hard-coded 5s; replay tests keep their 5s ready wait. TUI got `TmuxHarness::waitForTuiReadyAfterLogo()` and 9 TUI tests now poll visible ready state instead of a fixed 500ms post-logo sleep; `CancelStickinessE2eTest` waits were widened after a first full-check TUI flake under parallel load. Fork found TUI slowness is mostly per-test/class tmux + Symfony boot and real replay-driven user flows, not one large sleep; top filtered classes were `TuiResumeSessionSwitchE2eTest` (~13.3s PHPUnit / ~14.4s wall) and `TuiCompactCommandE2eTest` (~11.6s PHPUnit / ~12.7s wall). Testing skill and `tests/AGENTS.md` were read per fork handoff.

## Task workflow update - 2026-06-23T23:15:11.104Z
- Recorded fork run: kffyfc9l6u9y
- User asked to try ParaTest for TUI to address remaining `test:tui` slowness. Launched fork `kffyfc9l6u9y` from `4723a7733` to evaluate/implement a conservative TUI ParaTest path if safe: inspect TUI isolation (tmux sessions, var/tmp dirs, DB/cache/TEST_TOKEN), keep filtered runs sequential, start with small process count (default 2 unless 4 is proven stable), validate `castor test:tui` repeatedly and full `castor check`, and commit only if robust. Flaking is explicitly unacceptable; fork instructed to revert/no-commit if ParaTest is unsafe or flaky.

## Task workflow update - 2026-06-23T23:28:03.822Z
- Recorded fork run: kffyfc9l6u9y
- Validation: Fork reported baseline sequential `castor test:tui` — OK, ~76.2s wall.; Fork reported `castor test:tui` with ParaTest 2 — OK in 39.3s / 40.6s / 40.4s wall.; Fork reported `HATFIELD_TUI_PARATEST_PROCESSES=4 castor test:tui` — FAIL in `CancelStickinessE2eTest`; 4 workers not adopted.; Fork reported `castor test:tui --filter=CancelStickinessE2eTest` — OK 13.3s.; Fork reported `castor phpstan` — OK after env-processes fix.; Fork reported `castor cs-check` — OK.; Fork reported full `castor check` after hardening — quality ok (145.4s), test:tui 45.7s, llm-real 30.0s.; Fork reported full `castor check` #2 — quality ok (158.9s), test:tui 49.4s, llm-real 35.8s.
- Summary: Fork `kffyfc9l6u9y` completed and committed `da1ac96db`, enabling TUI E2E ParaTest for full `castor test:tui` / check lane at a conservative default of 2 workers. It verified TUI isolation through per-worker `paratest-bootstrap.php` DB/cache env, unique tmux session names based on worker PID, and per-test isolated temp dirs. Filtered `castor test:tui --filter=...` remains sequential. Check lane now reuses `build_test_tui_phpunit_command(null)` with ParaTest. 4 workers were explicitly rejected because `CancelStickinessE2eTest` failed under `HATFIELD_TUI_PARATEST_PROCESSES=4`; default 2 is documented with env override clamped 1–4. CancelStickiness waits were widened for parallel check load. Testing skill and `tests/AGENTS.md` were read per fork handoff.

## Task workflow update - 2026-06-23T23:32:27.644Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (0.2s).
- Pushed task/llm-proxy-deterministic-live-tests to origin.
- branch 'task/llm-proxy-deterministic-live-tests' set up to track 'origin/task/llm-proxy-deterministic-live-tests'.
- Created PR: https://github.com/ineersa/agent-core/pull/202
- Validation: Final fork `kffyfc9l6u9y`: `castor test:tui` with ParaTest 2 — OK in 39.3s / 40.6s / 40.4s wall (baseline ~76s).; Final fork `kffyfc9l6u9y`: `HATFIELD_TUI_PARATEST_PROCESSES=4 castor test:tui` — failed in `CancelStickinessE2eTest`; default kept at 2 workers.; Final fork `kffyfc9l6u9y`: `castor phpstan` — OK.; Final fork `kffyfc9l6u9y`: `castor cs-check` — OK.; Final fork `kffyfc9l6u9y`: full `castor check` — quality ok (145.4s), test:tui 45.7s, llm-real 30.0s.; Final fork `kffyfc9l6u9y`: full `castor check` #2 — quality ok (158.9s), test:tui 49.4s, llm-real 35.8s.; Earlier fork `62pevwvk0jog`: full `castor check` green twice after live runtime.ready hardening (173.0s and 171.5s).; Earlier fork `yglmnkdtf6px`: `castor test:controller-replay` improved from ~57.6s to ~42.2s and passed.; Earlier fork `5j6enrqbal76`/`wf4dc1o4yzai`: `test:llm-real` added to check and phpstan blocker fixed; full check passed before later speedups.
- Summary: Implementation complete and ready for review. Final branch HEAD `da1ac96db` includes: llama-proxy workflow docs; live `test:llm-real` restored into `castor check`; full `test:llm-real` ParaTest parallelization; robust live controller ready timeout; controller replay teardown/compaction drain speedups; TUI E2E ParaTest default 2 workers; and docs for proxy cache reset/warm workflow. Worktree was reported clean before transition. Full `castor check` was green twice in the final fork after TUI ParaTest hardening.

## Task workflow update - 2026-06-23T23:33:29.900Z
- Moved CODE-REVIEW → DONE.
- Merged task/llm-proxy-deterministic-live-tests into integration checkout.
- Merge made by the 'ort' strategy.
 .agents/skills/testing/SKILL.md                    |  88 ++++++++++++---
 .castor/e2e.php                                    | 124 ++++++++++++++-------
 .castor/helpers.php                                |  17 +++
 .castor/tasks.php                                  |  59 +++++-----
 AGENTS.md                                          |  29 ++++-
 docs/compaction.md                                 |   2 +-
 docs/qa-metrics.md                                 |  18 ++-
 tests/AGENTS.md                                    |  46 +++++++-
 .../Infrastructure/SymfonyAi/LlamaCppSmokeTest.php |   2 +-
 .../Controller/E2E/CompactionLiveSmokeTest.php     |  18 +--
 .../Controller/E2E/ControllerE2eTestCase.php       |  39 ++++++-
 ...ControllerReplayAutoCompactionMultiTurnTest.php |  22 +++-
 ...ReplayAutoCompactionRepeatedReplicationTest.php |  20 +++-
 ...ControllerReplayAutoCompactionToolCycleTest.php |  20 +++-
 .../Controller/E2E/ControllerReplayE2eTestCase.php |  38 +++++--
 .../E2E/ControllerReplaySummaryOnlyGuardTest.php   |   6 +-
 .../Runtime/Controller/E2E/ControllerSmokeTest.php |   6 +-
 .../E2E/OutputCapReadFileControllerTest.php        |   6 +-
 .../Controller/E2E/ShellFollowUpLiveE2eTest.php    |  59 +++-------
 .../Controller/E2E/ViewImageToolE2eTest.php        |   6 +-
 .../Controller/E2E/WriteFileToolE2eTest.php        |   6 +-
 tests/CodingAgent/Support/AgentTestExecutable.php  |  16 +++
 tests/Tui/E2E/CancelStickinessE2eTest.php          |   6 +-
 tests/Tui/E2E/TmuxHarness.php                      |  17 +++
 tests/Tui/E2E/TuiAutoCompactionCancelE2eTest.php   |   2 +-
 tests/Tui/E2E/TuiAutoCompactionE2eTest.php         |   2 +-
 tests/Tui/E2E/TuiCompactCommandE2eTest.php         |   4 +-
 tests/Tui/E2E/TuiJourneyE2eTest.php                |   5 +-
 tests/Tui/E2E/TuiOutputCapNoticeE2eTest.php        |   2 +-
 tests/Tui/E2E/TuiProviderErrorE2eTest.php          |   4 +-
 tests/Tui/E2E/TuiToolOutputE2eTest.php             |   4 +-
 31 files changed, 497 insertions(+), 196 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/llm-proxy-deterministic-live-tests.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/llm-proxy-deterministic-live-tests.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR: https://github.com/ineersa/agent-core/pull/202 merged by user.
- Summary: PR #202 was merged by user. Moving task to DONE and merging/cleaning up task workflow state.
