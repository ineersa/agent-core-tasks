# Fix runaway recursive search output and absurd token estimates

## Goal
Session 37 produced capped bash outputs reported as ~26,984,072 chars (~6,746,018 tokens) and ~3,395,579 chars (~848,895 tokens) from repository grep commands. Analyze why searches traversed massive generated/runtime/worktree/vendor content despite output capping, why the harness buffered/scanned that much before capping, and whether token estimates/cap reporting are accurate. The user-visible behavior is unacceptable even though output was eventually saved under `.hatfield/tmp/output-cap/`. Investigate command guidance, filesystem topology (including sibling worktrees exposed under the project), streaming/capping implementation, and recursive-search safeguards.

## Acceptance criteria
- Recover the exact offending session 37 commands and explain which paths/files caused the output explosion
- Verify whether reported character/token counts are accurate and how much data was buffered/read
- Prevent or terminate runaway textual output before multi-megabyte ingestion while preserving useful capped diagnostics
- Add regression coverage for huge command output and recursive-search edge cases

## Workflow metadata
Status: DONE
Branch: task/fix-runaway-recursive-search-output-and-token-estimates
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-runaway-recursive-search-output-and-token-estimates
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/349
PR Status: merged
Started: 2026-08-01T00:08:48.482Z
Completed: 2026-08-01T02:33:55.953Z

## Work log
- Created: 2026-07-31T20:28:30+00:00

## Task workflow update - 2026-08-01T00:08:48.482Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-runaway-recursive-search-output-and-token-estimates.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-runaway-recursive-search-output-and-token-estimates.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-runaway-recursive-search-output-and-token-estimates.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-runaway-recursive-search-output-and-token-estimates.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-runaway-recursive-search-output-and-token-estimates.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-runaway-recursive-search-output-and-token-estimates.
- Summary: Starting prioritized task 3. User emphasized that session 37 output-cap reporting appeared catastrophically large—possibly gigabyte-scale—and recalls settings-tool involvement. Investigation must first recover exactly what commands/tool calls were capped, which paths produced the bytes, and whether the reported character/token estimates represented buffered content, recursive metadata, or incorrect accounting before choosing a fix.

## Task workflow update - 2026-08-01T01:26:22.500Z
- Summary: User redirected the fix toward two concrete root causes: (1) `.hatfield/cache*` directories are proliferating and must be automatically cleaned up rather than left as recursive-search fodder; (2) audit/fix OutputCap path/category selection because Markdown and built-in settings/docs output appear to be receiving the 20k default cap instead of the intended 50k doc cap. Do not add a bash hard-stop threshold at this stage. Exact cache ownership/lifecycle and cap classification must be traced before implementation.

## Task workflow update - 2026-08-01T01:36:21.396Z
- Summary: Forensics finalized implementation scope. Current checkout has 61 `.hatfield/cache*` trees using 503.63 MiB: one persistent base cache (50.57 MiB) plus 60 leaked per-check QA/ParaTest caches (453.06 MiB) from 10 checks; no observable live process references them. Session 37's 26.98 MiB grep got 134/150 lines from five such cache trees. Required fix: automatically delete only the exact current QA run cache root and its `-paraT*` workers after Castor's exact-run leak check; preserve persistent `.hatfield/cache`, other runs, external absolute paths, and active/unverified caches. Also stop BashTool full-log ingestion by using its existing 20k bounded log read and referencing the existing background log instead of duplicating it; no process hard-stop/new threshold. OutputCap audit: direct `.md/.txt/.toon` paths already receive 50k, but `hatfield_docs read` incorrectly receives 20k, and the late hook re-caps fork/subagent reports at 20k. Centralize/fix semantic document classification for both cap stages; classify hatfield docs reads and agent handoff/retrieve reports as docs, keep task_list/bash/native settings generic, and prevent dotted native settings keys ending `.md` from being misclassified as files. No new setting or public API.

## Task workflow update - 2026-08-01T01:43:29.026Z
- Validation: Fork confirmed `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md` read/followed; Focused Castor tests — OK (61 tests, 254 assertions); castor deptrac — 0 violations; castor phpstan --path=src/CodingAgent/Tool — 0 errors; castor cs-check — clean after Castor cs-fix; castor test:llm-real — OK (13 tests, 144 assertions); Worktree clean; HEAD 278e2e9cd; 9 files changed, 635 insertions, 109 deletions; castor check intentionally not run in task-start phase
- Summary: Implementation committed as 278e2e9cd5e95af943b95d0164fa62a1fa6a3ce0. Future `castor check` runs now remove only their exact QA cache root/ParaTest worker roots after the exact-run leak assertion succeeds, preserving base/neighbor caches. Bash completion uses the existing bounded 20k log read and reports exact bytes plus the original background-log artifact rather than loading/duplicating multi-megabyte output. A shared cap-path resolver fixes 50k classification for `hatfield_docs read` and successful fork/subagent/agent_retrieve reports in both cap stages; native settings and generic tools remain 20k. No process hard-stop, broad stale-cache deletion, new settings, or my-pi extension change.

## Task workflow update - 2026-08-01T01:46:08.443Z
- Validation: Pre-delete: 60 exact cache-qa-* directories, 475,065,015 file bytes; 0 accessible active QA processes; Deleted exactly 60 regex-validated QA/ParaTest cache roots; Post-delete: 0 cache-qa-* directories; `.hatfield/cache` preserved
- Summary: With explicit user approval, removed the existing stale QA cache trees from the integration checkout before task-to-PR. Deletion was restricted by exact QA-run basename regex under `/home/ineersa/projects/agent-core/.hatfield`; persistent base cache was preserved. Accessible `/proc` environments showed zero active processes with HATFIELD_QA_RUN_ID/HATFIELD_CACHE_DIR before deletion.

## Task workflow update - 2026-08-01T02:07:23.619Z
- Validation: Reviewer initial: REQUEST CHANGES (simplification only; no correctness/security blockers); Reviewer re-review after dd7fb8fb9: APPROVED; Final delta review after 944a9c1a: APPROVED; Final castor test — OK (4417 tests, 16307 assertions); castor deptrac — 0 violations/errors; Final full castor phpstan — 0 errors; castor cs-check — clean; castor test:llm-real — OK (13 tests, 144 assertions); Final HEAD 944a9c1a718089179fcaf99ccf6d9573066c1a33; worktree clean; Existing integration stale QA caches: 60 exact dirs / 475,065,015 file bytes deleted with user approval; base cache preserved
- Summary: Task-to-PR review iteration complete. Initial reviewer REQUEST CHANGES were simplification-only and resolved by dd7fb8fb9; final full-PHPStan blocker (unreachable return after `fail_quality(): never`) was resolved by 944a9c1a with a natural void assertion and assert-then-cleanup control flow. Final reviewer APPROVED the full branch and delta; specification fidelity passed.

## Task workflow update - 2026-08-01T02:10:37.918Z
- Summary: First CODE-REVIEW transition was blocked by llama-proxy cache growth (221→223). Warm `castor test:llm-real` then passed at 223 entries. The failed gate exposed a real cleanup-order gap: cache guard failure occurs before `finalize_castor_check_run`, leaving six exact QA cache directories. This violates the automatic-cleanup intent and must be fixed before retrying PR transition; task remains IN-PROGRESS.

## Task workflow update - 2026-08-01T02:18:33.682Z
- Validation: castor test:llm-real warmup — OK (13 tests, 144 assertions), proxy entries=223; Cleanup-order reviewer: REQUEST CHANGES only for stale diagnostic wording; Final diagnostic delta reviewer: APPROVED; Full castor phpstan — 0 errors; cs-check clean after final deltas; Stranded failed-gate caches: deleted 6 exact inactive cache-qa-* dirs; remaining 0; Unrelated active QA id tui-20260801-021705-756448-a417d1cb was not signaled or modified; Final HEAD 915e664c179c861e9d2a6994623bc9c300e83dfd
- Summary: Post-lane cleanup ordering bug fixed in 5624d6c8 and stale diagnostic fixed in 915e664c; reviewers APPROVED both deltas. Warm live lane now passes with llama-proxy stable at 223 entries. With user approval, removed the six cache roots stranded by the first failed gate after confirming the only accessible active QA id was an unrelated `tui-*` run and did not match any `cache-qa-*` target.

## Task workflow update - 2026-08-01T02:20:39.369Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 480s)...
- castor check passed (109.3s).
- Pushed task/fix-runaway-recursive-search-output-and-token-estimates to origin.
- branch 'task/fix-runaway-recursive-search-output-and-token-estimates' set up to track 'origin/task/fix-runaway-recursive-search-output-and-token-estimates'.
- Created PR: https://github.com/ineersa/agent-core/pull/349

## Task workflow update - 2026-08-01T02:20:52.696Z
- Validation: move_task deterministic castor check — passed (109.3s); PR #349: https://github.com/ineersa/agent-core/pull/349; Post-gate task worktree cache-qa-* count: 0; Post-gate integration cache-qa-* count: 0
- Summary: Second CODE-REVIEW transition succeeded after cache warmup and cleanup-order correction. Deterministic castor check passed, branch pushed, PR #349 created. The successful gate left zero cache-qa-* directories in both task worktree and integration checkout, directly validating automatic exact-run cleanup.

## Task workflow update - 2026-08-01T02:33:55.953Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-runaway-recursive-search-output-and-token-estimates into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/helpers.php                                | 103 +++++++++
 .castor/tasks.php                                  |  43 +++-
 src/CodingAgent/Tool/BashTool.php                  |  42 +++-
 src/CodingAgent/Tool/OutputCapLlmTransformHook.php |  41 +---
 src/CodingAgent/Tool/OutputCapPathResolver.php     | 101 +++++++++
 .../Tool/OutputCapToolResultProcessor.php          |  81 +------
 .../Castor/ExactQaRunCacheCleanupTest.php          |  76 +++++++
 tests/CodingAgent/Tool/BashToolTest.php            |  51 +++++
 .../CodingAgent/Tool/OutputCapPathResolverTest.php | 236 +++++++++++++++++++++
 9 files changed, 655 insertions(+), 119 deletions(-)
 create mode 100644 src/CodingAgent/Tool/OutputCapPathResolver.php
 create mode 100644 tests/CodingAgent/Castor/ExactQaRunCacheCleanupTest.php
 create mode 100644 tests/CodingAgent/Tool/OutputCapPathResolverTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-runaway-recursive-search-output-and-token-estimates.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-runaway-recursive-search-output-and-token-estimates.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #349 state confirmed MERGED via gh; Integration checkout clean before DONE transition
- Summary: PR #349 confirmed merged on GitHub at 2026-08-01T02:33:28Z (merge commit 64d49380eb2ba44972d62c91a8af187899a27532). Moving task to DONE and cleaning worktree.

## Task workflow update - 2026-08-01T02:36:16.774Z
- Validation: LLM_MODE=true castor check — quality OK (371.7s); deptrac OK; test OK (4418 tests, 16315 assertions); controller-replay OK (11 tests, 160 assertions); TUI OK (30 tests, 181 assertions); llm-real OK (13 tests, 144 assertions); phpstan 0 errors; cs-check clean; llama-proxy cache guard stable 223→223; artifact integrity/leak checks OK; Automatic QA cache cleanup removed 6 exact-run roots; post-check cache-qa-* count 0; Integration checkout clean; task worktree removed
- Summary: Post-merge validation completed on main. Automatic exact-run cache cleanup executed and removed all six current QA cache roots; integration checkout remains clean and task worktree is removed.
