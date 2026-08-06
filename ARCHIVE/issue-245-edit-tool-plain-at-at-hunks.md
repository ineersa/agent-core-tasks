# Issue #245: Verify/fix edit tool plain @@ hunk matching on PHP files

## Goal
GitHub issue: https://github.com/ineersa/agent-core/issues/245

Open issue reports plain `@@` edit-tool hunks failing with `E_PATCH_STALE` on a PHP file while numbered `diff -u` hunk headers work. Historical archived work in `ARCHIVE/edit-tool-failure-context-and-guidelines.md` appears related, but there is no active TODO/IN-PROGRESS/CODE-REVIEW/DONE task.

Reproduce against current main first. If existing edit-tool changes already fixed this, add/confirm regression evidence and close/update the GitHub issue. If not, fix the relaxed hunk matcher/normalizer while preserving stale/duplicate/truncation safety.

## Acceptance criteria
- Plain `@@` hunks with exact PHP context apply successfully on current edit tool behavior.
- Numbered standard unified-diff hunks continue to work.
- Stale/no-match and duplicate/ambiguous contexts still fail safely with clear model-facing guidance.
- Focused regression tests cover the issue scenario or existing tests are identified as sufficient with evidence.
- GitHub issue #245 is updated or closed with the outcome.

## Workflow metadata
Status: DONE
Branch: task/issue-245-edit-tool-plain-at-at-hunks
Worktree: /home/ineersa/projects/agent-core-worktrees/issue-245-edit-tool-plain-at-at-hunks
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/266
PR Status: merged
Started: 2026-07-07T18:32:10.817Z
Completed: 2026-07-07T21:03:09.413Z

## Work log
- Created: 2026-07-07T18:08:35.713Z

## Task workflow update - 2026-07-07T18:32:10.817Z
- Moved TODO → IN-PROGRESS.
- Created branch task/issue-245-edit-tool-plain-at-at-hunks.
- Created worktree /home/ineersa/projects/agent-core-worktrees/issue-245-edit-tool-plain-at-at-hunks.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/issue-245-edit-tool-plain-at-at-hunks.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/issue-245-edit-tool-plain-at-at-hunks.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/issue-245-edit-tool-plain-at-at-hunks.
- Summary: task-explain complete. Plan: reproduce-first against current main using the issue's diagnostic artifacts, then fix narrow root cause (likely in PatchNormalizer relaxed hunk path) if still failing; do NOT jump to fuzzy matching. Launching reproduction fork.

## Task workflow update - 2026-07-07T18:37:32.656Z
- Validation: castor test --filter=Issue245ReproductionTest::testSeq58PlainHunkOnCurrentMain → FAIL (reproduces #245); castor test --filter='Issue245ReproductionTest::testSeq202NumberedHunkOnFileWithFarewell|Issue245ReproductionTest::testSeq76WrongIndentPlainHunkFailsAsStaleOnCurrentMain' → PASS (2 tests); ad-hoc PatchNormalizer::normalize() on seq-58 patch → succeeds, emits @@ -17,6 +17,11 @@; ad-hoc patch --dry-run on normalized seq-58 → FAIL at 17 (isolates failure to GNU patch stage); castor check / full suite: NOT run (reproduce-only fork)
- Summary: Reproduction fork complete. Verdict: REPRODUCES on current main (be7877d56).

KEY FINDING — root cause is NOT the relaxed @@ matcher. The matcher (findExactBlockMatch) WORKS: it locates the seq-58 old block at file line 17 and rewrites the hunk to @@ -17,6 +17,11 @@ correctly. The failure happens AFTER, in GNU patch: the normalized numbered hunk FAILS dry-run with "Hunk #1 FAILED at 17" → surfaces to user as [E_PATCH_STALE].

So the model-visible "plain @@ fails, numbered succeeds" is misleading: the seq-202 numbered success was on a DIFFERENT file state (farewell() already added via bash), not the same file. Apples vs oranges in the original report.

Root-cause hypothesis (for fix fork): relaxed-hunk body emission for a blank CONTEXT line that the model emitted as a truly empty line (no leading space). PatchNormalizer rewrites only the @@ header and leaves the body blank line as a bare empty line; findExactBlockMatch treats '' as matching '' (lenient) so it matches, but GNU patch (esp. dry-run uses --posix) does not accept the bare-empty-line context identically → off-by-one in old-side context → "FAILED at 17". Pointers: src/CodingAgent/Tool/Edit/PatchNormalizer.php resolveRelaxedHunks() ~411-450 (header rewrite) + body preservation; blank-line parsing ~316-330.

Evidence:
- Repro test: tests/CodingAgent/Tool/Issue245ReproductionTest.php in worktree (testSeq58PlainHunkOnCurrentMain FAILS = reproduces; testSeq202NumberedHunkOnFileWithFarewell PASSES; testSeq76WrongIndent... PASSES as expected stale).
- Artifacts recovered from /home/ineersa/.hatfield/dumps/20260630-164030/session-1/events.jsonl (PR worktree report dir gone). Exact patches extracted from message_end events.
- Direct PatchNormalizer::normalize() on seq-58 patch succeeds and emits @@ -17,6 +17,11 @@; standalone `patch --dry-run` on that normalized output FAILS at 17 — isolating the failure to the GNU-patch stage, not PHP matching.

Validation: castor test --filter=Issue245ReproductionTest::testSeq58PlainHunkOnCurrentMain → FAIL (expected, reproduces bug). No castor check run. Fork read testing SKILL.md + tests/AGENTS.md.

NEXT (not started, awaiting user): implementation fork to fix relaxed→numbered body emission (blank context line normalization) so GNU patch accepts the hunk; promote a regression test into EditFileToolTest.php; preserve stale/duplicate/ambiguous fail-safety. Do NOT add fuzz to findExactBlockMatch.

NOTE: repro test Issue245ReproductionTest.php has one misleading debug method (testSeq58PlainHunkOldBlockIsByteExactInFile) with a simplified parser that does not mirror production prefix stripping — treat as debug only. Clean up or replace during fix fork.

## Task workflow update - 2026-07-07T20:39:45.705Z
- Validation: GNU patch 2.7.6; real normalized.patch min fuzz=4 (applier uses 3); Real patch at -F4 -l -N --posix --dry-run → 'succeeded at 17 with fuzz 4' (Option A verified); Rebalanced patch (3 lead + 1 trail) at -F3 -l -N --posix --dry-run → 'succeeded at 19 with fuzz 2' (Option B verified); git apply --check rejects same patch independently; Re-emit from exact bytes: no effect (Option C ruled out); Lead/trail matrix: 3L+3T ok, 5L+1T fail, 5L+3T ok, 3L+1T ok, 4L+1T fail; Reproduction test tests/CodingAgent/Tool/Issue245ReproductionTest.php still in worktree (testSeq58PlainHunkOnCurrentMain FAILS = reproduces)
- Summary: GNU PATCH BISECTION COMPLETE — root cause confirmed, NOT a PatchNormalizer byte bug.

ROOT CAUSE: PatchNormalizer output is BYTE-CORRECT (exact context bytes, correct @@ counts). The failure is a GNU patch 2.7.6 matching behavior: the model's plain-@@ patches have UNBALANCED context (heavy leading: 4-5 lines before the change, light trailing: 1 line after). GNU patch 2.7.6 + the applier's `-F3` (fuzz=3) CANNOT apply unbalanced hunks with indented+blank-line context. `diff -u`-style BALANCED context (3 leading + 3 trailing) applies cleanly at fuzz 0.

EVIDENCE (all via direct GNU patch / git apply, byte-verified with xxd):
- Real normalized.patch: minimum fuzz to apply = 4. Applier uses 3 → FAILS. At -F4 → "succeeded at 17 with fuzz 4".
- git apply (independent impl) ALSO rejects the same patch (--ignore-whitespace doesn't help).
- Context lines are byte-exact after marker strip (5-space patch line = 1 marker + 4-space content = file's 4-space). Re-emitting context from exact file bytes did NOT fix it → rules out normalizer whitespace bug.
- Lead/trail matrix on real file at -F3 -l -N --posix:
  * 3 lead + 3 trail (diff-u style) → SUCCESS (fuzz 0)
  * 5 lead + 1 trail (model/PatchNormalizer style) → FAIL
  * 5 lead + 3 trail → SUCCESS
  * 3 lead + 1 trail → SUCCESS
  * 4 lead + 1 trail → FAIL
  → Rule: with only 1 trailing context line, leading must be ≤3. Blank context line adds +1 to the fuzz requirement.
- BLANK LINE confirmed as compounding factor: same patch with no blank line in file succeeds at fuzz 3; with blank line needs fuzz 4.

FIX OPTIONS (both verified against the real patch):
A) PatchApplier: bump fuzz 3→4 (1-line change). Real patch → "succeeded at 17 with fuzz 4". CON: globally looser matching, weakens stale/duplicate fail-safety the task requires preserving. Probably NOT acceptable.
B) PatchNormalizer: rebalance/trim context to ≤3 leading AND ≤3 trailing (diff-u style) when rewriting relaxed→numbered hunks, keeping lines nearest the change, with a uniqueness check so trimming doesn't create ambiguity. Real patch trimmed to 3 lead + 1 trail → "succeeded at 19 with fuzz 2"; leading context `return sprintf('Hello...')` is unique in file. PRO: keeps fuzz=3, preserves safety. This is the recommended direction.
C) Re-emit context from exact file bytes: RULED OUT (tested, no effect).

The original issue report's "numbered succeeds, plain fails" was apples-vs-oranges (seq-202 numbered success was on a different file state with farewell() already present). But the underlying plain-@@ bug is REAL and reproducible.

Awaiting user decision on fix direction (A vs B) before launching implementation fork.
- Main agent ran direct GNU patch + git apply bisection in worktree var/tmp/issue245-diag/ (diagnostic scripts only, no src/ changes)
- Confirmed: root cause is unbalanced context (heavy leading/light trailing) + GNU patch 2.7.6 fuzz/whitespace handling, NOT a PatchNormalizer byte bug
- Both fix options verified against real patch; awaiting user direction before implementation fork

## Task workflow update - 2026-07-07T20:52:13.648Z
- Validation: castor test --filter='EditFileToolTest' → OK (61 tests, 303 assertions); castor cs-check PatchApplier + EditFileToolTest → files_fixed=0 (clean); castor phpstan PatchApplier → no errors; Regression test testRelaxedHunkWithHeavyLeadingContextAndBlankLineApplies: FAILS at F3, PASSES at F5 (guard-verified); All existing safety tests (duplicate/stale/ambiguous/unmatched) pass at F5 — verified at F5 and F6 prior to implementation
- Summary: IMPLEMENTATION COMPLETE (Option A — fuzz 3→5). Commit c2957c67b on task/issue-245-edit-tool-plain-at-at-hunks.

PRODUCTION CHANGE: src/CodingAgent/Tool/Edit/PatchApplier.php — '-F3' → '-F5' in both runPatchDryRun (line ~200) and runPatchApply (line ~211). Docblock added explaining why fuzz 5 (unbalanced LLM plain-@@ context; safety stays upstream in findExactBlockMatch).

REGRESSION TEST: tests/CodingAgent/Tool/EditFileToolTest.php — testRelaxedHunkWithHeavyLeadingContextAndBlankLineApplies (PHP class, plain-@@ insert of farewell() between methods, 5 leading context + blank line + 1 trailing context — the exact issue-245 shape). Verified: FAILS at F3 (E_PATCH_STALE), PASSES at F5.

CLEANUP: removed untracked throwaway tests/CodingAgent/Tool/Issue245ReproductionTest.php.

VALIDATION (Castor, focused — non-TUI/non-runtime/non-LLM-visible change):
- castor test --filter='EditFileToolTest' → OK (61 tests, 303 assertions)
- castor cs-check on touched files → clean (files_fixed=0)
- castor phpstan on PatchApplier → no errors
- new test guard-verified: F3 revert → test errors; F5 → passes

KNOWN CEILING: fuzz 5 fixes issue #245 and the realistic unbalanced-context range; not provably every conceivable patch (would need fuzz >5). Deterministic alternative (Option B rebalance) kept in reserve.
- Implementation fork: F3→F5 bump in PatchApplier, regression test added, throwaway repro test removed, committed 72d5adf3b
- Parent: cs-check found single-quote style on 'Goodbye' assert; applied cs-fix, re-validated 61/61 + cs clean, amended to c2957c67b
- Moving to CODE-REVIEW

## Task workflow update - 2026-07-07T20:54:08.557Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 600s)...
- castor check passed (98.5s).
- Pushed task/issue-245-edit-tool-plain-at-at-hunks to origin.
- branch 'task/issue-245-edit-tool-plain-at-at-hunks' set up to track 'origin/task/issue-245-edit-tool-plain-at-at-hunks'.
- Created PR: https://github.com/ineersa/agent-core/pull/266

## Task workflow update - 2026-07-07T21:03:09.414Z
- Moved CODE-REVIEW → DONE.
- Merged task/issue-245-edit-tool-plain-at-at-hunks into integration checkout.
- Merge made by the 'ort' strategy.
 src/CodingAgent/Tool/Edit/PatchApplier.php  | 11 ++++-
 tests/CodingAgent/Tool/EditFileToolTest.php | 71 +++++++++++++++++++++++++++++
 2 files changed, 80 insertions(+), 2 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/issue-245-edit-tool-plain-at-at-hunks.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/issue-245-edit-tool-plain-at-at-hunks.
- Pulled integration checkout: Already up to date..
