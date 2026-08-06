# llm-real full-suite cache grows +1 on first warm after cold

## Goal
## Symptom
When warming the full `llm-real` suite for the llama-proxy cache (AGENTS.md procedure: clear → cold-warm → warm), the **first warm run after cold grows +1 entry**, then subsequent warm runs are +0. Observed deterministically twice:

```
clear → cold full (HATFIELD_CHECK_LLM_REAL_PARATEST_PROCESSES=1): 0 → 22
warm #1: 22 → 23   (+1)
warm #2: 23 → 23   (+0)
warm #3: 23 → 23   (+0)
```

This does NOT block `castor check` if you warm until stable (the gate's own baseline is taken before preflight, and a stable 23→23 passes the guard). It IS a friction + a latent determinism smell worth fixing so warmup converges in a single repeat pass.

## Already ruled out
- **RewindBranchLiveE2eTest** — was the prior culprit (`random_bytes` marker), fixed in session-07 commit `fe87fe59d`. Independently re-verified +0 on warm after the fix (filtered clear→cold 0→6, warm +0, warm +0). NOT this bug.
- **WriteFileToolE2eTest** — verified +0 stable (fixed `[llm-real:write-file]` marker).

## Likely cause
One of the remaining `llm-real` tests emits a volatile value in a request position OUTSIDE the llama-proxy's normalization window. The proxy (at `/home/ineersa/projects/llama-proxy/llama_proxy/app.py`, `_normalize_body_for_cache_key`) only:
1. strips the leading prologue (`system`/`developer` + leading `[user-context]` user messages), and
2. templates `parent_run_id`, `agent_run_id`, test-subagent cwd paths, and `artifact_id` fields.

So any OTHER volatile value (bare session UUID, timestamp, non-`[user-context]`-prefixed context message, nondeterministic tool-call argument ordering, a temp-dir path not matching the test-subagent regex, etc.) in a user/assistant/tool message will change the cache key run-to-run. The fact that it stabilizes after one extra warm pass suggests it may also be an ordering/timing-dependent first-request effect.

## Candidate tests to diff (start here)
The 8 remaining `#[Group('llm-real')]` tests beyond the two ruled out:
- `tests/CodingAgent/Runtime/Controller/E2E/SubagentParallelLiveE2eTest.php` (`[llm-real:subagent-parallel-v1]`)
- `tests/CodingAgent/Runtime/Controller/E2E/SubagentRetrieveLiveE2eTest.php` (`[llm-real:subagent-retrieve-chain-v1]`, `…-v1-step2`)
- `tests/CodingAgent/Runtime/Controller/E2E/ShellFollowUpLiveE2eTest.php` (`[llm-real:shell-followup-no-shell]`, `…-with-shell`)
- `tests/CodingAgent/Runtime/Controller/E2E/OutputCapReadFileControllerTest.php` (`[llm-real:output-cap-read]`)
- `tests/CodingAgent/Runtime/Controller/E2E/ViewImageToolE2eTest.php` (`[llm-real:view-image]`)
- `tests/CodingAgent/Runtime/Controller/E2E/ControllerSmokeTest.php` (`[llm-real:controller-smoke]`)
- `tests/AgentCore/Infrastructure/SymfonyAi/LlamaCppSmokeTest.php`

The subagent tests are prime suspects — they spawn child runs with their own run IDs/cwds and multiple agents, so a non-templated id/path leaking into a child request is the most plausible +1.

## How to isolate (binary-search the offending test)
1. Clear cache: `curl -X DELETE http://127.0.0.1:9052/__llama_proxy/cache`
2. Cold-warm each candidate in isolation (sequential, 1 process): `HATFIELD_CHECK_LLM_REAL_PARATEST_PROCESSES=1 castor test:llm-real --filter <TestName>`
3. Run the same filtered test again warm and check delta. The offending test shows +N on first warm (like RewindBranch did before its fix).
4. Cache records are root-owned under `/var/cache/llama-proxy/` — not directly readable. To see the volatile content, either (a) ask the user to diff two cache records for the offending test, or (b) capture the raw outbound request body by temporarily pointing a test at a request-logging stub, or (c) read the proxy admin endpoints (`/__llama_proxy/cache/stats`) and reason from the cache-key inputs.

## Acceptance criteria
- [ ] Offending test(s) identified with a reproducer showing +N on warm.
- [ ] Root cause of the volatile request content named precisely (which field, which message position, why it escapes proxy normalization).
- [ ] Fix applied at the **lowest correct layer** — prefer normalizing the test's volatile content (stable per-scenario marker/fixed paths, matching the `[llm-real:…]` convention) over adding proxy-side normalization. Only touch the proxy if the volatile value is inherent and shared across many tests.
- [ ] Proven: clear → cold full-suite → warm full-suite = **+0 on the first warm** (single-pass convergence), with the full `llm-real` suite still green (10 tests / 121 assertions).
- [ ] `castor check` green with a single clear+cold+warm warmup (no double-warm needed).
- [ ] No new `@phpstan-ignore` annotations; no history rewrite.

## Notes
- Warm SEQUENTIALLY (`HATFIELD_CHECK_LLM_REAL_PARATEST_PROCESSES=1`) for any cold recording — default 4-process parallelism can degrade llama.cpp on cold misses and poison cassettes (burned real time on session-07 this way).
- This is a test-determinism / DX task, not a production code change. Keep scope narrow.

## Acceptance criteria
- Offending llm-real test identified with reproducer showing +N cache entries on first warm run
- Root cause of volatile request content named precisely (field + message position + why it escapes proxy normalization)
- Fix applied at lowest correct layer (prefer stable per-scenario test content over proxy changes)
- Proven: clear → cold full-suite → warm full-suite = +0 on FIRST warm (single-pass convergence), full llm-real still 10/121 green
- castor check green with single clear+cold+warm warmup (no double-warm needed)
- No new @phpstan-ignore annotations; no history rewrite

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
- Created: 2026-07-01T00:01:49.021Z

## Task workflow update - 2026-07-10T22:55:31.462Z
- Summary: Root cause now confirmed exactly from root-owned llama-proxy cassette comparison. Offending scenario is OutputCapReadFileControllerTest post-read request (`[llm-real:output-cap-read]`) with message roles user→assistant→tool. Assistant tool_call_id, read arguments, model, tools, and normalized user prompt are stable. The tool result embeds two volatile path components: isolated test root `var/tmp/test-output-cap-<random>` and output-cap filename `.hatfield/tmp/output-cap/<Ymd>-<random16>.txt`. Four otherwise-identical post-tool requests produced distinct keys ab5eba13, c9e12289, 4c3e4731, and 4e93d0a. This is why warmup/check keeps adding entries. The value is inherent to safe test isolation/output-cap persistence; correct layer is llama-proxy request/response path templating, not a fixed shared test path or disabling the follow-up. Key-only stripping is insufficient because replayed model output could reference the stale path; proxy should extract current `Saved full output:` path, template it in key and recorded response, and substitute current path on replay, analogous to existing agent artifact ID templating WIP. Cassette evidence saved at /tmp/out.json.

## Task workflow update - 2026-07-10T23:19:40.305Z
- Validation: castor test:llm-real --filter=OutputCapReadFileControllerTest x2: 1 test/11 assertions OK; cache delta 0 each; castor test:llm-real x2: 10 tests/121 assertions OK; cache delta 0 each; llama-proxy health: cache_template_output_cap_paths=true
- Summary: Proxy output-cap path templating implemented/deployed and repeat-run validation passed. Running health reports cache_template_output_cap_paths=true. Exact OutputCapReadFileControllerTest passed twice with cache 286→286 each run (1 test/11 assertions). Full castor test:llm-real passed twice with cache 286→286 each run (10 tests/121 assertions, ~22.7s and ~21.4s). This confirms the previously volatile Saved full output path no longer creates a new normalized key on repeated runs. Cold cache clear→record proof and final castor check remain outstanding; proxy WIP remains uncommitted in /home/ineersa/projects/llama-proxy.

## Task workflow update - 2026-07-11T01:20:11.801Z
- Summary: Implementation mapping confirmed for this task: llama-proxy 72cacca templates dynamic artifact/output-cap paths; 11f3927 templates `Successfully wrote N bytes to <path>` request/response paths. Agent-core temporary caller-relative write result was reverted at 4796cd05c, preserving canonical absolute-path product behavior while proxy owns test cache normalization. Latest normalized write cassette key 9c9e066f contains `Successfully wrote 12 bytes to {{write_result_path_0}}`. Root causes in task are fixed; remaining acceptance proof is full castor check with cache delta 0 (and optional destructive cold-clear convergence proof if still required).

## Task workflow update - 2026-07-11T01:22:51.590Z
- Summary: Warm-cache acceptance now proven in full deterministic gate qa-20260711-012030-101-c858b378: full llm-real lane passed 10 tests/121 assertions and llama-proxy cache guard stayed 289→289. Both cache_template_output_cap_paths and cache_template_write_result_paths enabled. Gate failed only an unrelated TUI assertion. Root-cause fixes are proxy commits 72cacca and 11f3927; latest normalized cassettes contain {{output_cap_path_0}} and {{write_result_path_0}}. Destructive clear→cold→warm proof was not run, but the original recurring warm-growth symptom is no longer present in standalone repeated suites or full check.

## Task workflow update - 2026-07-11T01:29:14.769Z
- Summary: Additional intermittent warm-growth source confirmed from full gate cache 289→290: ViewImageToolE2eTest post-tool request embeds random absolute isolated path three times (top-level view_image path, attachment_refs[].path, and `[Tool result image: ...]`). Cassette key 1653dd2d, evidence /tmp/cache-290.json. This operational attachment path should remain canonical in Hatfield and be request/response-templated proxy-side. Implementor prompt saved at /tmp/llama-proxy-view-image-path-normalization-prompt.md.

## Task workflow update - 2026-07-11T01:33:20.425Z
- Validation: ViewImageToolE2eTest run 1: OK 1 test/13 assertions ~6.8s, cache 290→290; ViewImageToolE2eTest run 2: OK 1 test/13 assertions ~6.8s, cache 290→290
- Summary: View-image proxy normalization implemented at llama-proxy 7216da0 (`Normalize view image paths`) with cache_template_view_image_paths=true. Focused `castor test:llm-real --filter=ViewImageToolE2eTest` passed twice (1 test/13 assertions, ~6.8s each), cache stable 290→290→290. This closes the third confirmed volatile path source; final proof is full gate Δ0.

## Task workflow update - 2026-07-11T01:35:48.821Z
- Summary: Final acceptance proof passed in full deterministic gate qa-20260711-013335-100-1911597b: proxy commits 72cacca (artifact/output-cap), 11f3927 (write-result), 7216da0 (view-image) enabled; llm-real 10 tests/121 assertions passed; cache stable 290→290 across full check. Original first-warm cache-growth symptom is resolved for all three confirmed volatile path sources.

## Task workflow update - 2026-07-11T02:08:53.882Z
- Summary: Correction after post-merge cache 290→291 forensic: cassette 019d46d2 is correctly templated. Structural comparison against legacy view-image cassettes 1653dd2d/c69b7f8a/23f293d8 found exactly one difference: messages[2].content concrete absolute image path → `{{view_image_path_0}}`; no model/tool/schema/parameter difference. Therefore +1 was one-time canonical templated cassette materialization while legacy path-keyed cassettes remained on disk, not a fourth volatile source. Prior focused 290→290 proof against uncleared legacy cache was insufficient. Original destructive cold→warm→warm convergence acceptance remains the authoritative missing proof before closing this task.
