# Make file-mention indexing bounded and non-blocking during startup and resume

## Goal
User reproduction on main agent-core repo: launching installed `hatfield` exhausts the 128 MiB PHP limit in Symfony Finder `RecursiveDirectoryIterator` from `FileMentionIndexBuilder::scanEntries()`. `castor run:agent` takes roughly 10 seconds and selecting `/resume` is also very slow.

Investigation found these are connected: `FileMentionIndexStartupListener::register()` synchronously calls `FileMentionIndexBuilder::build()` during every `InteractiveMode` TUI iteration, including initial startup, CLI `--resume`, and the new iteration after selecting `/resume`. The agent-core tree currently exposes roughly 577,908 files / 3.4 GB to the scan because generated `.hatfield/cache-*` variants and nested dependency/tooling trees are not reliably excluded. The current index is only ~2,278 entries / 172 KB, so scanning the generated tree is waste.

The `/resume` picker query itself is not the bottleneck (36 DB rows; repeated metadata queries ~0.01s). Event replay is currently modest (largest log ~1.8 MB). Controller stop/start can independently add latency, but first remove the proven synchronous file scan; create a separate resume task only if latency remains afterward.

Root paths: `src/CodingAgent/CLI/FileMentionIndexBuilder.php`, `FileMentionIndexStartupListener.php`, `src/Tui/Application/InteractiveMode.php`, `CompletionFileIndexRefreshCommand`, and existing `FileMentionIndexBuilderTest`. Preserve atomic replacement, non-blocking lock, and functional `@` completion. No arbitrary PHP memory-limit increase.

## Acceptance criteria
- Launching Hatfield in the agent-core repository under the normal 128 MiB PHP memory limit does not OOM while preparing file-mention completion.
- Initial TUI startup, CLI `--resume`, and the post-selection `/resume` TUI iteration do not synchronously perform a full repository index rebuild before becoming usable.
- The scan excludes Hatfield runtime/generated trees including every `.hatfield/cache*` variant, sessions/tmp, and nested dependency/build trees such as `vendor`, `node_modules`, and `var` wherever they occur; a focused fixture proves these paths are not traversed/indexed.
- Index refresh remains bounded on pathological repositories and cannot retain/traverse hundreds of thousands of excluded entries merely to produce the 50,000-entry cap; fail/degrade without taking down the TUI.
- Existing valid indexes remain immediately usable while refresh happens outside the TUI registration/input path; missing/stale index behavior is explicit and does not silently reintroduce startup blocking.
- `@` file mention completion, atomic index replacement, and non-blocking concurrent-build locking continue to work.
- A regression test covers the actual root cause: cache-suffixed Hatfield directories and nested generated dependency trees plus repeated startup/resume registration; it would fail with the current synchronous broad scan.
- Measure startup and resume-selection behavior on agent-core after the fix. If `/resume` remains materially slow once index scanning is removed, record phase evidence and create a separate controller/replay latency task rather than broadening this fix.

## Workflow metadata
Status: ARCHIVE
Branch: task/fix-file-mention-index-startup-oom-resume-latency
Worktree: /home/ineersa/projects/agent-core-worktrees/fix-file-mention-index-startup-oom-resume-latency
Fork run: evezhshxjumg
PR URL: https://github.com/ineersa/agent-core/pull/338
PR Status: merged
Started: 2026-07-30T01:40:20.555Z
Completed: 2026-07-30T02:48:36.998Z

## Work log
- Created: 2026-07-30T01:25:37.074Z

## Task workflow update - 2026-07-30T01:29:59.790Z
- Summary: Correction after verifying actual Symfony Finder behavior: `.hatfield/cache-*` is correctly Git-ignored and omitted from the final index. The failure is subtler: Finder constructs `RecursiveDirectoryIterator` → explicit `ExcludeDirectoryFilterIterator` → `RecursiveIteratorIterator`, and only afterward wraps it in `VcsIgnoredFilterIterator`. Therefore Git-ignore filtering happens after recursive traversal and does not prune ignored directories. Only explicit `exclude()` entries prune before descent; current explicit list has `.hatfield/cache` but not `.hatfield/cache-*`. The ~578k count is traversal scope, not indexed output (~2,278 entries). Task acceptance already requires excluded trees not be traversed/indexed.

## Task workflow update - 2026-07-30T01:35:43.477Z
- Summary: Further verification: Git ignore rules are correct, but Symfony Finder applies `VcsIgnoredFilterIterator` after `RecursiveIteratorIterator`, so ignored paths are filtered only after descent. Explicit `exclude()` pruning does happen before recursion and already prunes every directory named `vendor`, `node_modules`, or `var` at any depth; the missing prune is chiefly `.hatfield/cache-*`. Preferred implementation direction is native Git (`git ls-files --cached --others --exclude-standard -z`, deriving parent directories) for Git repos, because it honors nested `.gitignore`, `.git/info/exclude`, and global excludes without adding a low-adoption parser dependency; retain a bounded Finder fallback for non-Git dirs. Current disk measurements: project `.hatfield/cache*` totals ~4.9GB and is not removed by `castor clean:cleanup`; installed global `~/.cache/hatfield` is ~15MB across 3 identities and has no automatic GC.

## Task workflow update - 2026-07-30T01:40:20.555Z
- Moved TODO → IN-PROGRESS.
- Created branch task/fix-file-mention-index-startup-oom-resume-latency.
- Created worktree /home/ineersa/projects/agent-core-worktrees/fix-file-mention-index-startup-oom-resume-latency.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/fix-file-mention-index-startup-oom-resume-latency.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/fix-file-mention-index-startup-oom-resume-latency.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/fix-file-mention-index-startup-oom-resume-latency.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/fix-file-mention-index-startup-oom-resume-latency.
- Summary: Starting user-approved implementation. Keep existing `.hatfield/cache-*` trees in place as the real regression fixture; do not clear them before validation. Preferred path: Git-native nonignored file enumeration for Git repos, bounded Finder fallback for non-Git roots, and no synchronous full rebuild during startup/resume.

## Task workflow update - 2026-07-30T01:50:53.789Z
- Recorded fork run: l5fndxgjvukf
- Summary: Implementation fork launched. It will use streamed `git ls-files --cached --others --exclude-standard -z` with inferred directories and explicit noisy-path filtering in Git repos, a bounded/pre-pruned Finder fallback for non-Git roots, and delete the synchronous startup registrar so scheduler refresh is the sole owner. Existing 4.9GB `.hatfield/cache-*` trees remain untouched for real validation.

## Task workflow update - 2026-07-30T01:58:41.154Z
- Recorded fork run: l5fndxgjvukf
- Validation: castor test --filter=FileMentionIndexBuilderTest|CompletionFileIndexRefreshCommandTest|FileMentionIndexReader => OK 22 tests / 251 assertions; castor test => OK 4470 tests / 16415 assertions; castor deptrac => 0 violations; castor phpstan => 0 errors; castor cs-fix + castor cs-check => files_fixed=0; castor clean:cleanup:workers:list => no stale workers; Real measurement from parent checkout: APP_ENV=prod APP_DEBUG=0 worktree bin/console completion:file-index:refresh => elapsed 0.21s peak_rss 74MB index 2275 lines, parent .hatfield/cache* count remained 452
- Summary: Implemented Git-native file-mention indexing and removed synchronous startup rebuild. Commit 80cc72ba8 on task/fix-file-mention-index-startup-oom-resume-latency. Real parent-repo refresh against retained 4.9GB cache trees: 0.21s, peak RSS ~74MB, 2275 index entries, zero noisy path hits, 452 cache dirs untouched.

## Task workflow update - 2026-07-30T02:00:28.720Z
- Recorded fork run: l5fndxgjvukf
- Validation: Focused Castor tests: 22 tests / 251 assertions passed; castor test: 4470 tests / 16415 assertions passed; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: files_fixed=0; castor clean:cleanup:workers:list: no stale workers; Real retained-cache measurement: 0.21s, 74MB peak RSS, 2,275 index entries, 452 cache directories unchanged
- Summary: Implementation complete at commit 80cc72ba86e24481a88fcd915cffff276ecd6dc6 (6 files, +532/-150). Git worktrees now stream `git ls-files --cached --others --exclude-standard -z`, infer parent directories, enforce the 50k combined cap, and filter explicit noisy trees. Non-Git/parent-ignored roots use Finder with pre-recursion prune for `.hatfield/cache-*` and existing noisy paths. Deleted FileMentionIndexStartupListener and DI registration; scheduler is sole refresh owner, so existing indexes are immediate and first-ever missing index remains empty until the first 30s refresh. Parent cache trees retained; real parent scan completed in 0.21s/74MB RSS with 2,275 entries and all 452 cache directories unchanged. Worktree clean; task remains IN-PROGRESS pending user-requested task-to-PR phase.

## Task workflow update - 2026-07-30T02:04:15.921Z
- Recorded fork run: 4ijy4m1b0obe
- Summary: Task-to-PR phase started. Sync fork is merging current origin/main into the clean task branch before correctness/minimality review; no push or PR yet.

## Task workflow update - 2026-07-30T02:04:18.336Z
- Validation: git fetch origin; git merge origin/main --no-ff (clean, no conflicts); castor cs-check skipped: merge did not touch task files and no conflict resolution
- Summary: Pre-PR sync: normal merge of origin/main (ed28ffc23, PR #337 subagent LLM step count) into task branch. Merge commit c9444ab8b parents 80cc72ba8 + ed28ffc23. Zero conflicts; merge only brought subagent progress files. Task behavior intact (Git-native enumeration, Finder prune, startup listener deleted, scheduler-only refresh). Worktree clean. No push/PR.

## Task workflow update - 2026-07-30T02:13:25.265Z
- Recorded fork run: evezhshxjumg
- Summary: Pre-PR reviewer verdict: APPROVE WITH SUGGESTIONS. No correctness, security, specification-fidelity, lifecycle, or test blockers; all acceptance criteria considered proven. One adjudicated dead-code cleanup is being applied: inline trim of already-bounded Git stderr and delete redundant truncateDiagnostic(). Focused full unit/static/TUI replay validation is running in the correction fork before final re-review.
- Reviewer verified no remaining startup-listener references, unconditional scheduler_default consumer launch, streamed/NUL-safe bounded Git enumeration, Finder pre-pruning, process teardown, argv safety, and atomic old-index preservation.
- Non-blocking reviewer notes left unchanged: deleted tracked files can transiently appear from git --cached; custom $excludeDirs affects Finder only but has no production callers; first missing index waits for scheduler by finalized design.

## Task workflow update - 2026-07-30T02:21:17.777Z
- Recorded fork run: evezhshxjumg
- Validation: Focused file-index tests: 22 tests / 251 assertions passed; castor test: 4472 tests / 16439 assertions passed; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: files_fixed=0; castor test:tui: 38 tests / 194 assertions passed (replay-backed); castor clean:cleanup:workers:list: no stale QA workers; Real retained-cache refresh: 0.21s, 74MB peak RSS, 2275 entries, 452 cache directories untouched
- Summary: Final task-to-PR review APPROVE at HEAD 4260425034850736cdc73b98d5bc5e113a99e5a8. Reviewer confirmed the post-review deletion preserves the 512-byte streamed stderr bound and found no correctness, security, specification-fidelity, minimality, lifecycle, or test blockers. Ponytail verdict: Lean already. Ship. Branch is clean and contains current origin/main via merge c9444ab8b.

## Task workflow update - 2026-07-30T02:24:33.247Z
- Validation: First automatic castor check: blocked by llama-proxy cache-growth guard (201→202), not code failure; castor test:llm-real warmup: 15 tests / 186 assertions passed
- Summary: First CODE-REVIEW transition was blocked only by deterministic gate hygiene: llama-proxy cache grew 201→202 during castor check. No code/test lane failure. Warmed the new cassette with focused `castor test:llm-real`; all 15 tests / 186 assertions passed in 31.6s. Retrying the deterministic gate with stable cache.

## Task workflow update - 2026-07-30T02:27:01.106Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (132.9s).
- Pushed task/fix-file-mention-index-startup-oom-resume-latency to origin.
- branch 'task/fix-file-mention-index-startup-oom-resume-latency' set up to track 'origin/task/fix-file-mention-index-startup-oom-resume-latency'.
- Created PR: https://github.com/ineersa/agent-core/pull/338
- Validation: Focused file-index tests: 22/251 passed; castor test: 4472/16439 passed; castor deptrac: 0 violations; castor phpstan: 0 errors; castor cs-check: clean; castor test:tui: 38/194 passed; castor test:llm-real warmup: 15/186 passed; No stale QA workers; Final reviewer: APPROVE
- Summary: Final reviewer APPROVE. Replaced synchronous startup/resume Finder scans with scheduler-owned refresh, streamed Git-native nonignored file enumeration, and pre-pruned bounded Finder fallback. Real agent-core refresh against retained 4.9GB cache trees is 0.21s at 74MB RSS; no caches removed. First gate warmed one new llama-proxy cassette; focused live lane then passed 15/186.

## Task workflow update - 2026-07-30T02:27:12.791Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/338
- Updated PR Status: open
- Summary: PR #338 opened at HEAD 4260425034850736cdc73b98d5bc5e113a99e5a8. GitHub reports MERGEABLE; GitGuardian passed. Deterministic castor check passed in 132.9s after warming the single new llama-proxy cassette.

## Task workflow update - 2026-07-30T02:48:36.998Z
- Moved CODE-REVIEW → DONE.
- Merged task/fix-file-mention-index-startup-oom-resume-latency into integration checkout.
- Merge made by the 'ort' strategy.
 config/services.yaml                               |  10 +-
 .../CLI/CompletionFileIndexRefreshCommand.php      |  16 +-
 src/CodingAgent/CLI/FileMentionIndexBuilder.php    | 372 ++++++++++++++++++---
 .../CLI/FileMentionIndexStartupListener.php        |  67 ----
 src/Tui/Completion/FileMentionIndexReader.php      |   5 +-
 .../CLI/FileMentionIndexBuilderTest.php            | 202 +++++++++--
 6 files changed, 522 insertions(+), 150 deletions(-)
 delete mode 100644 src/CodingAgent/CLI/FileMentionIndexStartupListener.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/fix-file-mention-index-startup-oom-resume-latency.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/fix-file-mention-index-startup-oom-resume-latency.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: PR #338 state: MERGED; PR merge commit: 8a03026ac8f638bc2d2083f5783e67253f82f3d7
- Summary: PR #338 was merged on GitHub at 8a03026ac8f638bc2d2083f5783e67253f82f3d7. Final reviewer approved; deterministic castor check passed before merge. Proceeding with integration merge/pull and worktree cleanup.

## Task workflow update - 2026-07-30T02:53:39.152Z
- Updated PR URL: https://github.com/ineersa/agent-core/pull/338
- Updated PR Status: merged
- Validation: Initial post-merge castor check had one unrelated/transient SubagentLivePickerControllerTest assertion failure; focused rerun passed 1 test / 5 assertions; Final LLM_MODE=true castor check passed in 356.1s; Unit/integration: 4472 tests / 16439 assertions; Controller replay: 10 tests / 135 assertions; TUI replay: 38 tests / 194 assertions; Live LLM: 15 tests / 186 assertions; Deptrac/PHPStan/CS: passed; llama-proxy cache guard stable 202→202; QA artifact integrity and leak checks passed; Integration git status clean; task worktree removed
- Summary: Task completed. PR #338 merged on GitHub as 8a03026ac8f638bc2d2083f5783e67253f82f3d7, integrated locally, current origin/main pulled, task worktree and IDEA exclusions removed. Integration checkout is clean.

## Task workflow update - 2026-08-06T20:58:59.597Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
