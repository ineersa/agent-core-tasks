# Reset ChatGPT reasoning-transition baseline when context refresh rewrites anchored history

## Goal
## Defect

Main sessions on `openai-codex/gpt-6.1-sol` deterministically fail with upstream HTTP 503 (text/plain "ChatGPT is temporarily unavailable") after a context refresh, while the same model works from forks/subagents (explicit model => machinery skipped) and other models work from the same session with the same ~155-177k-token bag.

## Evidence (session 75, 2026-10-10)

- `context_refreshed` at 17:00:39; first 503 at 17:01:46. Failures at 17:01, 17:07, 22:33, 23:23 — 4/4 deterministic, retry storm cannot recover.
- Same session on gpt-5.6-sol: 155k-177k input tokens, all succeed (22:51-23:13). gpt-6.1-sol succeeded earlier the same day at 176k tokens (03:44; 1708 lifetime successes).
- Probe fork on gpt-6.1-sol inheriting parent context: succeeded first try (forks pass explicit model, so `SessionAwareModelResolver` skips the transition block).

## Mechanism

`SessionAwareModelResolver` (~lines 100-120) enables reasoning transitions only for mutable main sessions on ChatGPT providers where `supports_reasoning_configuration_updates` is set — currently only `gpt-6.1-sol`. For those requests:

- `HatfieldSessionStore::claimReasoningBaseline()` (src/CodingAgent/Session/HatfieldSessionStore.php:224) validates the durable baseline by model match, ROOT anchor match, and transition message_key presence.
- `ChatGPTReasoningTransitionRequestHook` (src/CodingAgent/Agent/Execution/ChatGPTReasoningTransitionRequestHook.php) forces historical baseline effort and injects control-only `AssistantMessage(new ReasoningConfiguration($effort))` envelopes before anchored messages.

A context refresh preserves the root key while rewriting/moving mid-history anchors, so validation passes and envelopes are replayed against rewritten history; the ChatGPT Responses backend rejects the item sequence with a generic 503 instead of a 4xx.

## Entry points

- src/CodingAgent/Session/HatfieldSessionStore.php — claimReasoningBaseline, listReasoningTransitions, rememberReasoningTransition
- src/CodingAgent/Agent/Execution/ChatGPTReasoningTransitionRequestHook.php
- src/CodingAgent/Agent/Execution/ChatGPTReasoningTransitionTransformHook.php (re-applies anchors onto surviving history)
- src/CodingAgent/Agent/Execution/SessionAwareModelResolver.php (transition enable block)
- Tests: tests/CodingAgent/Agent/Execution/ChatGPTReasoningTransitionRequestHookTest.php, tests/CodingAgent/Session/SessionHistoryProviderTest.php, ChatGPTServiceWiringTest

## Constraints

- Smallest solution; existing facilities; no BC shims; no new settings/public APIs.
- Keep comments that explain the validation invariants; update them with the change.

## Acceptance criteria
- Baseline claiming detects refresh-rewritten history beyond root-key survival: any change to the ordered anchor sequence or anchor context invalidates the baseline and resets it (empty transitions, no forced historical effort, no injected envelopes)
- Regression test reproduces the session-75 shape (root anchor survives refresh, transition message_keys still present, but positions/context rewritten) and asserts reset + envelope-free request bag
- Existing ChatGPTReasoningTransition* and SessionHistoryProvider tests keep passing; dead code removed per specification fidelity
- Focused validation in the task worktree (touched suites; phpstan if touched files are in its path); full castor check reserved for the CODE-REVIEW transition

## Workflow metadata
Status: DONE
Branch: task/fix-reasoning-transition-baseline-after-context-refresh
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-reasoning-transition-baseline-after-context-refresh
Fork run: agent_c272e6f049ed7f6b
PR URL: https://github.com/ineersa/agent-core/pull/561
PR Status: merged
Started: 2026-10-11T00:24:11+00:00
Completed: 2026-10-15T18:34:33+00:00

## Work log
- Created: 2026-10-11T00:23:51+00:00

## Task workflow update - 2026-10-11T00:24:11+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-reasoning-transition-baseline-after-context-refresh.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-reasoning-transition-baseline-after-context-refresh.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-reasoning-transition-baseline-after-context-refresh.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-reasoning-transition-baseline-after-context-refresh.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-reasoning-transition-baseline-after-context-refresh.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/fix-reasoning-transition-baseline-after-context-refresh/.idea.

## Task workflow update - 2026-10-11T00:32:29+00:00
- Recorded fork run: agent_c272e6f049ed7f6b
- Validation: castor test --filter='ChatGPTReasoningTransition|SessionHistoryProvider|ChatGPTServiceWiring|SessionAwareModelResolver|HatfieldSessionStore' → 72 tests / 493 assertions passed; castor phpstan → 0 errors; castor cs-check → passed (after cs-fix); git diff --check → clean; castor check reserved for CODE-REVIEW
- Ownership: owner=fork; fork_run=agent_c272e6f049ed7f6b; revision=main@974aec772; scope=baseline validation (HatfieldSessionStore), resolver fingerprints (SessionAwareModelResolver), request-hook regression tests; outcome=assigned; commit=none
- Ownership: owner=fork; fork_run=agent_c272e6f049ed7f6b; revision=main@974aec772; scope=baseline validation (HatfieldSessionStore), resolver fingerprints (SessionAwareModelResolver), request-hook regression tests; outcome=completed; commit=1fc69204bb6ef53d569976256eb8a32e912d8035
- Fork confirmed reads: .agents/skills/testing/SKILL.md and tests/AGENTS.md before test work.
- Fork caveat: applyContextRefreshed replaces generated context (instructions) rather than reordering conversational anchors; tests prove the reset contract, not that this defect alone produced the reported upstream 503. Real-world confirmation pending (session 75 on gpt-6.1-sol after fix).

## Task workflow update - 2026-10-11T02:14:56+00:00
- Validation: Follow-up commit 8078c59fc (review EDGE fixes): castor test --filter='ChatGPTReasoningTransition|SessionAwareModelResolver' → 29 tests / 237 assertions OK; castor phpstan → 0 errors; castor cs-check → clean; Skipped castor test:llm-real per task-to-pr step 5: no schema/prompt/streaming/auth change; wire delta is withholding injected controls after rewritten history, asserted deterministically (byte-identical bag vs explicit-model path); live repro requires a real session's ChatGPT history — reviewer assessment concurs (agent_5b2a0006dd91d96e)
- Review: role=reviewer; artifact=agent_5b2a0006dd91d96e; target=1fc69204b vs main@974aec772; scope=resolver fingerprints, baseline claim validation, transition tests; verdict=APPROVE WITH SUGGESTIONS (0 CRITICAL/BUG/SEC; 2 EDGE, 5 NTH)
- Post-review: owner=main applied both EDGE findings (tolerant fingerprint encoding matching TransformHook::stringify + key-order assumption comment) in 8078c59fc
- NTH deferred (recorded, not blocking): filter anchor keys symmetrically in tests; direct legacy-baseline one-time-reset test; boundary comment for rewrites after last anchor; docs/session-storage.md wording drift; per-turn baseline flush write amplification noted
- Unresolved: live confirmation that the fixed invariant removes the reported upstream 503 for an affected session on gpt-6.1-sol — close only on empirical confirmation after merge

## Task workflow update - 2026-10-11T02:17:12+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (114.0s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/fix-reasoning-transition-baseline-after-context-refresh/var/reports/qa-20261011-021518-7025-48f32780.
- Session/run: 84.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-11T02:17:14+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/fix-reasoning-transition-baseline-after-context-refresh to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/fix-reasoning-transition-baseline-after-context-refresh/var/reports/qa-20261011-021518-7025-48f32780.
- Session/run: 84.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-11T02:17:16+00:00
- castor check passed (114.0s).
- Pushed task/fix-reasoning-transition-baseline-after-context-refresh to origin.
- Created PR: <url>
- Session/run: 84.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-11T02:17:16+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (114.0s).
- Pushed task/fix-reasoning-transition-baseline-after-context-refresh to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/561

## Task workflow update - 2026-10-11T03:14:21+00:00
- Root-cause correction (2026-10-11): the reported session-75 gpt-6.1-sol 503s are NOT caused by reasoning-transition replay. Live experiments: (1) supports_reasoning_configuration_updates=false at provider level with full restart — failures unchanged; (2) session 83 on gpt-6.1-sol with its own provider_cache_key works concurrently; (3) probe fork on gpt-6.1-sol works. Root cause isolated to a corrupted OpenAI server-side prompt-cache entry for (session 75 prompt_cache_key 01a0ed6f…, gpt-6.1-sol) — the key is sent as prompt_cache_key on every main-session request (SessionAwareModelResolver:199, ModelClient ALLOWED_FIELDS) and is stable since session creation. Immediate remediation: rotated session 75 provider_cache_key to 01a128f3-bd4c-7511-87b1-c0fd16395777; live confirmation pending. PR #561 body updated to reflect the corrected root cause; PR remains a correctness fix for stale baseline replay.
- Follow-up needed (separate task): rotate provider_cache_key when a cache-keyed provider burns through retry exhaustion on 5xx storms, so one corrupted upstream cache entry cannot wedge a session permanently.

## Task workflow update - 2026-10-12T01:11:58+00:00
- ROOT CAUSE FOUND (2026-10-12, supersedes all prior theories): OpenAI ChatGPT edge gateway rejects request bodies > 1,048,576 bytes (1 MiB) with HTTP 503 text/plain 'upstream connect error or disconnect/reset before headers. reset reason: remote connection failure' (Envoy edge error, cf-ray present, __oailb affinity JWT names chat.gateway.unified-209 backend). Proven by byte-exact replay bisect: session-75 fixture 1,277,257B -> 503 (6/6 in-TUI + curl replay); drop-oldest-turn 1,000,480B -> 200; same content padded +79KB dummy user msg -> 503; pinpoint: 1,048,402B -> 200, 1,048,702B -> 503. Content is innocent (dummy padding flips the result), additional_tools item innocent (dropping it at 1,197,899B still fails), backend healthy (G fixture routed to same unified-209 host -> 200).
- Why it looked model/session-specific: gpt-6.1-sol retains native history (session-75 F body: function_call_output 530KB + reasoning encrypted_content 382KB + function_call 132KB = native bloat); cross-model (gpt-5.6-sol) conversion strips thinking/signatures and shrinks below 1MiB; forks rebuild history; smaller sessions under limit. Context refresh at 17:00:39 Oct 10 grew retained history past 1MiB; first failure 17:01:46. Cache-key rotation and reasoning-transition flag had no effect because the bytes stayed over the cap. PR #561 fix is valid hardening but NOT the root cause.
- Evidence: var/tmp/llm-capture/ (F=73cc279d..b1f5c271 6x503, G=9a97bc90 200, replay .replay.json files for every probe); tools/diagnostics/llm_capture_{manifest,replay}.php; capture decorator LlmRequestCaptureHttpClient (temporary, to be removed with fixtures at close).
- Durable fix (new task needed): enforce a serialized-byte budget (~1MiB minus headroom) when assembling ChatGPT /v1/responses bodies — trim oldest retained native history groups at safe user-turn boundaries preserving call_id pairing until the body fits; surface a clear error when instructions+tools+current-turn alone exceed the cap.

## Task workflow update - 2026-10-12T01:19:54+00:00
- pi-mono comparison + compression probes (2026-10-12): pi-mono (~/claw/pi-mono, 6fb2e7815) has NO byte-level request guard either — it escapes the byte cap by (a) targeting chatgpt.com/backend-api/codex/responses (DEFAULT_CODEX_BASE_URL) instead of api.openai.com/v1/responses, and (b) zstd-compressing request bodies (compressRequestBodyZstd, Content-Encoding: zstd; openai-codex-responses.ts:203-377, 'The Codex backend accepts zstd-compressed request bodies'); its compaction is token-based (shouldCompact: contextTokens > contextWindow - reserveTokens, default 16384) like ours.
- Compression NOT available on our route: api.openai.com/v1/responses rejects Content-Encoding zstd AND gzip with 400 {"detail":"Unsupported content type"} (full 1.28MB F body compressed to 434KB zstd / 527KB gzip — both rejected). Switching to chatgpt.com/backend-api/codex/responses with our OAuth token: 401 rejected_by_access_enforcement regardless of originator/OpenAI-Beta/account-id headers — our token audience is api.openai.com; the Codex backend needs the Codex CLI OAuth client. pi-mono's escape hatch requires new OAuth plumbing, not a URL change.
- Token-vs-byte accounting for the record: F body 1,277,257B carried ~176k input tokens ≈ 7.3 bytes/token (vs ~4 for prose) — encrypted reasoning base64 (382KB, billed as original reasoning tokens not base64 bytes), tool outputs (530KB, JSON-escaped terminal text), 298 items of framing. Context budgeter enforced the 272k TOKEN window; the edge cap is on BYTES; at 7.3B/token the byte cap binds first with ~140k tokens of this content mix. Fix must check serialized bytes every request.

## Task workflow update - 2026-10-15T17:59:45+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: Address user review P2 at reviewed head8078c59fcb: fingerprint provider-relevant tool_calls/source_model metadata while excluding transient transition markers; add metadata-only rewrite and unchanged assistant/tool cold-replay regressions.

## Task workflow update - 2026-10-15T18:00:02+00:00
- Summary: Main owns this bounded two-file correction. Previous implementation/reviewer artifacts belong to session84 and cannot resume from this parent; a new independent reviewer will be needed after focused validation. No production metadata API or unrelated 503 diagnosis changes are in scope.
- Ownership: owner=main; fork_run=none; revision=8078c59fcb44880c9cee30f721e59fcf70260b23; scope=two-file P2 fix in SessionAwareModelResolver and request-hook metadata/cold-replay regressions; outcome=assigned; commit=none

## Task workflow update - 2026-10-15T18:08:38+00:00
- Validation: Focused Castor suite ChatGPTReasoningTransition|SessionHistoryProvider|ChatGPTServiceWiring|SessionAwareModelResolver|HatfieldSessionStore:76 tests/605 assertions PASS.; castor phpstan:0 errors; castor cs-check:clean; git diff --check PASS.; Full castor check reserved for CODE-REVIEW transition after independent review.
- Summary: P2 corrected in676bbf3e30ad26358db1a51d05ddfb83042e4c57. Resolver fingerprints explicit tool_calls/source_model projection, excluding request-local markers. Canonical fixture now optionally commits assistant/tool exchange. New regressions cover arguments/name/source rewrites resetting selected effort and clearing ledger, unchanged assistant/tool cold replay, metadata order and transient-marker stability. Original command-only regression failed before production fix (expected high, actual low). Removed an attempted invalid orphaned-call-ID data row because sequence validation correctly rejects it before resolution; valid metadata-only rewrites remain covered.
- Ownership: owner=main; fork_run=none; revision=8078c59fcb44880c9cee30f721e59fcf70260b23; scope=two-file P2 fix in SessionAwareModelResolver and request-hook metadata/cold-replay regressions; outcome=completed; commit=676bbf3e30ad26358db1a51d05ddfb83042e4c57
- Review assignment: role=reviewer; prior_artifact=agent_5b2a0006dd91d96e (session84, not resumable from current parent); target=676bbf3e30ad26358db1a51d05ddfb83042e4c57 versus8078c59fcb; scope=user P2 metadata fingerprint correction, cold replay and specification fidelity; status=awaiting new independent reviewer.

## Task workflow update - 2026-10-15T18:16:50+00:00
- Validation: Reviewer independently ran request-hook12tests304assertions, resolver/reasoning/tool-sequence44tests369assertions, converter/conversion/replay/baseline28tests179assertions; all PASS.; castor test:llm-real:5tests30assertions PASS on local llama_cpp_test/test.; Main focused suite76tests605assertions, phpstan/cs-check/diff clean.
- Summary: Independent reviewer approved676bbf3e30ad26358db1a51d05ddfb83042e4c57. No correctness/security/spec-fidelity blockers. Ready for mandatory transition gate and existing PR561 update; no merge authorized. Live local model smoke also passed5tests30assertions; this does not claim live ChatGPT upstream confirmation.
- Review: role=reviewer; artifact=agent_a606a13194e15a68; run=c3acae03-6ab4-5082-97f5-ec1bdfa741a6; target=676bbf3e30ad26358db1a51d05ddfb83042e4c57 versus8078c59fcb; scope=P2 provider metadata fingerprint, canonical assistant/tool cold replay, transient-marker stability and specification fidelity; decision=APPROVE; commit=676bbf3e30ad26358db1a51d05ddfb83042e4c57; unresolved_blockers=none.
- Nonblocking suggestions deferred: explicit outgoing tool-name assertion; release note for one-time baseline invalidation. No extra release file is needed for this narrow correction.

## Task workflow update - 2026-10-15T18:19:11+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (115.6s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/fix-reasoning-transition-baseline-after-context-refresh/var/reports/qa-20261015-181716-8159-2927d3dc.
- Session/run: 88.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-15T18:19:13+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/fix-reasoning-transition-baseline-after-context-refresh to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/fix-reasoning-transition-baseline-after-context-refresh/var/reports/qa-20261015-181716-8159-2927d3dc.
- Session/run: 88.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-15T18:19:14+00:00
- castor check passed (115.6s).
- Pushed task/fix-reasoning-transition-baseline-after-context-refresh to origin.
- PR already exists: <url>
- Session/run: 88.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-15T18:19:14+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (115.6s).
- Pushed task/fix-reasoning-transition-baseline-after-context-refresh to origin.
- PR already exists: https://github.com/ineersa/agent-core/pull/561
- Validation: Regression-first command-only rewrite failed before fix (selected high, stale low retained).; Focused suite76tests605assertions PASS; independent reviewer supplementary suites PASS.; PHPStan0errors, formatting and diff clean; live local LLM5tests30assertions PASS.
- Summary: Address user P2 metadata-only history rewrites in676bbf3e3. Fixed provider metadata projection; canonical assistant/tool rewrite and cold replay regressions pass. Independent reviewer APPROVE at exact revision. Run mandatory full gate and update existing PR561; do not merge.

## Task workflow update - 2026-10-15T18:20:38+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/561
- Updated PR Status: open
- Validation: Full CODE-REVIEW castor check PASS115.6s at676bbf3e3.; gh pr view confirms OPEN, head676bbf3e30ad26358db1a51d05ddfb83042e4c57, P2 description updated.; git status --short empty.
- Summary: P2 fix submitted at676bbf3e30ad26358db1a51d05ddfb83042e4c57 after independent APPROVE. Mandatory transition castor check passed115.6s; branch pushed. Existing-PR transition did not update its body, so explicitly updated PR561 description and verified remote OPEN/head/body match. Worktree clean. No merge performed; no unresolved correctness blockers.

## Task workflow update - 2026-10-15T18:34:33+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/fix-reasoning-transition-baseline-after-context-refresh into integration checkout.
- Auto-merging tests/CodingAgent/Agent/Execution/ChatGPTReasoningTransitionRequestHookTest.php
Merge made by the 'ort' strategy.
 src/CodingAgent/Agent/Execution/SessionAwareModelResolver.php                   |  21 +++++++++++++-
 src/CodingAgent/Session/HatfieldSessionStore.php                                |  25 ++++++++---------
 tests/CodingAgent/Agent/Execution/ChatGPTReasoningTransitionRequestHookTest.php | 142 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++--
 3 files changed, 172 insertions(+), 16 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-reasoning-transition-baseline-after-context-refresh.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-reasoning-transition-baseline-after-context-refresh.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Pre-merge full castor check passed115.6s at independently approved676bbf3e3.; GitHub merge verified2026-10-15T18:33:26Z; no uncommitted work in either checkout.
- Summary: User merged PR561. GitHub confirms MERGED at14d8364177fee7c844f46193a60904670521ae53 with reviewed head676bbf3e30ad26358db1a51d05ddfb83042e4c57. Main and task worktree clean before transition. Sync integration and remove task worktree; post-merge full Castor validation follows.

## Task workflow update - 2026-10-15T18:38:18+00:00
- Updated PR Status: merged
- Validation: Post-merge castor check: PASS, exit status 0, reported duration 249.5 seconds. QA run qa-20261015-183509-13315-199ca578.; Unit/integration: 5,161 tests, 26,981 assertions. Controller replay: 15 tests, 922 assertions. TUI replay: 7 tests, 50 assertions. Live LLM: 5 tests, 30 assertions.; All 11 QA lanes passed; artifact integrity, exact-run process/tmux leak checks, and llama-proxy cache guard passed.; Git status clean; GitHub merge commit is an ancestor of main; task worktree absent.
- Summary: PR #561 is merged. Integration checkout synchronized at 2a4971f07765de94f66bae678cdd19aa9af29ee1, including GitHub merge 14d8364177fee7c844f46193a60904670521ae53. Required post-merge castor check passed. Task worktree and IDEA exclusion removed; main is clean.
- Initial command supervision returned without an exit status while the original Castor process continued. No rerun was launched. Waited for that process to finish and verified recorded exit status 0 and final quality: ok log.
