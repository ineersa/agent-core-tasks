# Keep malformed command output from breaking durable tool results

## Goal
Session74 on main350d4d9e2 failed at2026-10-03T21:00:58Z. Bash exited2; raw log .hatfield/tmp/bg/e0dfc7e5edaa0332.log has14597bytes and invalidUTF8 atoffset7011 (d0 followed by0a). ToolCallResult snapshot serialization raised NotEncodableValueException, wrapped as snapshot write failed. Retries exhausted and agent_end(failed) seq1417 appended. Fix text presentation boundary while preserving raw log bytes and strict storage validation. Do not repair/reexecute live session or modify owned workers. Separate from unfinished SPEC03.

## Acceptance criteria
- Malformed command output produces persistable UTF-8 tool text for success and failure without losing exit status.
- Original background log bytes remain unchanged; any replacement is disclosed in returned text.
- Configured serializer and durable tool-batch regression proves malformed output cannot reproduce snapshot failure.
- Focused validation, independent review and full gate pass.

## Workflow metadata
Status: DONE
Branch: task/2026-10-03-fix-invalid-utf8-command-output-result-persistence
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-10-03-fix-invalid-utf8-command-output-result-persistence
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/549
PR Status: merged
Started: 2026-10-03T21:10:21+00:00
Completed: 2026-10-04T02:49:20+00:00

## Work log
- Created: 2026-10-03T21:10:08+00:00

## Task workflow update - 2026-10-03T21:10:21+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-10-03-fix-invalid-utf8-command-output-result-persistence.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-10-03-fix-invalid-utf8-command-output-result-persistence.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-10-03-fix-invalid-utf8-command-output-result-persistence.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-10-03-fix-invalid-utf8-command-output-result-persistence.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-10-03-fix-invalid-utf8-command-output-result-persistence.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-10-03-fix-invalid-utf8-command-output-result-persistence/.idea.
- Summary: Urgent isolated main regression fix. Main owns bounded process-log presentation normalization and real persistence regression. Keep SPEC03 draft untouched.

## Task workflow update - 2026-10-03T21:14:32+00:00
- Ownership: owner=main; fork_run=none; revision=350d4d9e2; scope=normalize process log text views without changing raw bytes, prove bash success/failure snapshot round-trip; outcome=assigned; commit=none

## Task workflow update - 2026-10-03T21:16:37+00:00
- Validation: castor test focused10/73 PASS; phpstan/deptrac/diff-check PASS; cs-fix complete.; Initial bash tool supervision returned unknown prematurely; original command kept running. Its immutable status file later recorded exit0 with passing full output. No duplicate QA execution.
- Summary: Main implemented5220297dd: normalize only process-log display text, preserve raw file, disclose replacement, keep successful/error status. Added real bash success/nonzero outcome -> ToolCallResult -> configured JSON batch save/load regression and malformed whole/tail log tests.
- Ownership: owner=main; fork_run=none; revision=350d4d9e2; scope=process log display normalization and persistence regression; outcome=completed; commit=5220297dd
- Reviewer continuation: agent_c3013579ac3ba666; target=5220297dd; scope=isolated3-file main hotfix; review pending.

## Task workflow update - 2026-10-03T21:20:17+00:00
- Validation: Reviewer independently ran ProcessLifecycleTest|BashToolTest:44 tests/252 assertions PASS; all cases <=10s.
- Summary: Independent reviewer agent_c3013579ac3ba666 APPROVE at5220297dd. Verified bash success/error, background status/completion text, strict serializer, raw-byte preservation and bounded display expansion. No blockers.

## Task workflow update - 2026-10-03T21:22:11+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (96.7s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-03-fix-invalid-utf8-command-output-result-persistence/var/reports/qa-20261003-212034-39332-4b347002.
- Session/run: 75.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-03T21:22:12+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-10-03-fix-invalid-utf8-command-output-result-persistence to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-03-fix-invalid-utf8-command-output-result-persistence/var/reports/qa-20261003-212034-39332-4b347002.
- Session/run: 75.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-03T21:22:16+00:00
- castor check passed (96.7s).
- Pushed task/2026-10-03-fix-invalid-utf8-command-output-result-persistence to origin.
- Created PR: <url>
- Session/run: 75.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-03T21:22:16+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (96.7s).
- Pushed task/2026-10-03-fix-invalid-utf8-command-output-result-persistence to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/549
- Summary: Fix session74 root cause: malformed bash output becomes disclosed UTF8-safe text while raw logs, exit status and strict snapshot storage remain unchanged. Independent review APPROVE5220297dd. Run fullgate and publish isolated hotfix.

## Task workflow update - 2026-10-03T21:47:53+00:00
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: User rejected PR549 as root-cause resolution. Explicit override: implement on main first; no malformed-byte reproductions, no tests now, no SPEC03 work. Existing PR worktree retained, not merged. Main owns source-only correction of representation boundaries.

## Task workflow update - 2026-10-03T21:49:34+00:00
- Summary: User explicitly requires source fix on main before tests. No reproductions or runtime/test commands. Preserve prior PR and SPEC03 drafts separately. Replace unchecked whole-log reads and byte-cut guidance; enforce complete terminal result representation across direct, factory, error and deferred results without blanket serializer substitution.
- Ownership: owner=main; fork_run=none; revision=350d4d9e2; scope=main-checkout result representation contract and bounded process log decoding; outcome=assigned; commit=none

## Task workflow update - 2026-10-03T21:52:33+00:00
- Summary: Main has a partial, unvalidated source-only draft in the main checkout. User requests fork ownership for this larger correction. Explicit handoff: main stops editing; one fork owns completion and source-only audit. No tests, reproductions, malformed-byte execution, session artifact reads, or SPEC03 work.
- Ownership: owner=main; fork_run=none; revision=350d4d9e2+dirty; scope=result representation and bounded process-log draft; outcome=blocked; commit=none

## Task workflow update - 2026-10-03T22:07:37+00:00
- Summary: Final user-approved behavior supersedes previous plans: any malformed UTF8 in tool result means FAILED tool call with safe explicit error; no scrub/base64/success-with-warning. Persist same finalized payload delivered to model; eliminate unchecked hidden output. Apply equally to exceptions, worker fallback and deferred results. User prohibits reproductions/tests now. Work only on PR549 branch. Main restored clean; abandoned nine-file draft preserved at /tmp/hatfield-abandoned-utf8-draft-fSpRsR, not applied to PR.
- Ownership: owner=fork; fork_run=agent_7c5f8ea748ff19c4; revision=5220297dd; scope=PR549 strict malformed-output tool failure and unified finalization, source-only; outcome=assigned; commit=none

## Task workflow update - 2026-10-03T22:30:23+00:00
- Validation: Source-only independent review APPROVE; git diff --check PASS.; NO tests, reproductions, runtime/static QA, PHAR work or incident artifact reads performed. No validation claim for execution behavior.
- Summary: Fork implemented approved fail-on-malformed policy in PR549 worktree, replacing scrub behavior. Complete result text inspection covers metadata and keys; malformed results become fixed nonretryable failed results. Shared success/error finalization plus deferred/fallback/owner admission paths; invalid hidden raw_result discarded. Source reviewer agent_c3013579ac3ba666 APPROVED corrected dirty revision after depth-throw blocker, diagnostic fallback and capped error metadata were fixed. Main clean; SPEC03 paused. Not committed/pushed/merged. Existing PR tests still expect obsolete scrub behavior and remain untouched per user instruction.
- Ownership: owner=fork; fork_run=agent_7c5f8ea748ff19c4; revision=5220297dd+16dirtypaths; scope=PR549 strict malformed-output failure and consistent finalization; outcome=completed; commit=none

## Task workflow update - 2026-10-03T23:01:39+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/549
- Validation: git diff --check PASS; worktree clean after commit/push. No reproductions or tests executed.
- Summary: Committed and pushed ac3faacfb to existing PR549 at user's explicit request for advisor review. Updated PR title/body to replace obsolete scrub claims and disclose no tests/QA for this revision; old tests still need authorized updates. Task remains IN-PROGRESS, not merged; source-only approval does not constitute full validation.
- Ownership: owner=fork; fork_run=agent_7c5f8ea748ff19c4; revision=ac3faacfb; scope=failed malformed tool results; outcome=completed; commit=ac3faacfb

## Task workflow update - 2026-10-03T23:14:28+00:00
- Summary: Advisor requests changes at ac3faacfb. Accepted four findings: object-property inspection misses internal/serialized data; post-execution processor throws escape before remembering; code_mode IPC silently substitutes malformed args/errors; live logs can end inside a character. Main routing pass inspected exact sources. Fork owns corrections sequentially in PR549 worktree, no main/SPEC03 changes. Prior no-reproduction/no-test constraint remains in force; tests must be replaced before eventual merge, not now.
- Ownership: owner=fork; fork_run=agent_7c5f8ea748ff19c4; revision=ac3faacfb; scope=advisor four findings: closed terminal representation, contained processor errors, strict code_mode IPC, live read boundaries; outcome=assigned; commit=none

## Task workflow update - 2026-10-03T23:59:55+00:00
- Validation: Source-only review covered all changed production paths; git diff --check PASS.; No tests, reproductions, runtime/static QA, PHAR or incident output reads. Existing scrub-era tests still need replacement before merge.
- Summary: Pushed advisor corrections0345c7811 to PR549: closed terminal values, explicit supported conversion, contained+remembered processor failures, strict code_mode IPC/name/argument/error handling, live/final bounded log reads. Source-only reviewer agent_c3013579ac3ba666 covered first13paths; could not resume at206083tokens. Replacement agent_faa5b7425270dd19 reviewed log2files+deadbranch cleanup, found stale status/fread guards, approved corrections. Main clean; SPEC03 paused. PR body updated with precise limits; no merge/statuspromotion.
- Ownership: owner=fork; fork_run=agent_7c5f8ea748ff19c4; revision=0345c7811; scope=all four advisor findings and source-review corrections; outcome=completed; commit=0345c7811

## Task workflow update - 2026-10-04T00:31:42+00:00
- Summary: Advisor REQUEST CHANGES at0345c7811 accepted: processors now run after cancellation and lose extension-owned outcomes; fragment depth512 leaves no space for snapshot wrappers. Main inspected executor cancellation, extension flag processor, both validators and snapshot writer. Fork assigned two bounded source-only fixes; no tests/reproductions yet.
- Ownership: owner=fork; fork_run=agent_7c5f8ea748ff19c4; revision=0345c7811; scope=restore processor-before-cancellation ordering and conservative consistent depth budget; outcome=assigned; commit=none

## Task workflow update - 2026-10-04T00:42:41+00:00
- Validation: git diff --check PASS; independent source-only review APPROVE.; No tests, reproductions or runtime/static QA executed.
- Summary: Pushed eedd86f43 to PR549 addressing both advisor P2s: validated/contained processors now run once before post-execution cancellation, then metadata/finalization/remember; both validators use named64-depth budget incl keys/string leaves, leaving wrapper headroom. Source reviewer agent_faa5b7425270dd19 APPROVED four-path correction. Existing tests still deferred and obsolete; no QA or reproduction. Not merged. Main unchanged; SPEC03 paused.
- Ownership: owner=fork; fork_run=agent_7c5f8ea748ff19c4; revision=eedd86f43; scope=processor ordering and conservative data depth; outcome=completed; commit=eedd86f43

## Task workflow update - 2026-10-04T02:43:10+00:00
- Summary: User explicitly approved deleting the obsolete scrubbing tests now and deferring replacement coverage. Follow-up TODO 2026-10-03-cover-rejected-tool-results-and-output-boundaries records acceptance checks and safety restrictions. No equivalent rejection tests claimed. Main owns this narrow two-file deletion, not a large implementation slice; no production changes or runtime validation.
- Ownership: owner=main; fork_run=none; revision=eedd86f43; scope=delete two obsolete scrub test methods/providers and unused imports; outcome=assigned; commit=none
- Deletion evidence: BashToolTest::testMalformedOutputCanBePersistedWithoutRepeatingCommand and ProcessLifecycleTest::malformedLogBytesAreDisplaySafeWithoutChangingArtifact assert rejected substitution/success behavior. Their four provider rows are removed. Replacement rejection/persistence/raw-artifact coverage is explicitly deferred with user approval to TODO/2026-10-03-cover-rejected-tool-results-and-output-boundaries.md.

## Task workflow update - 2026-10-04T02:47:12+00:00
- Validation: Source-only deletion review APPROVE; git diff --check PASS; no runtime QA.
- Summary: Committed/pushed386a0d0b4 deleting exactly the two obsolete scrub test methods/providers and three now-unused imports. Both files match pre-PR350d4d9e2; no pre-existing coverage removed. Reviewer agent_faa5b7425270dd19 source-only APPROVE. User-approved future coverage task2026-10-03-cover-rejected-tool-results-and-output-boundaries created. PR body discloses removal/coverage gap. No tests or reproductions run, not merged.
- Ownership: owner=main; fork_run=none; revision=386a0d0b4; scope=delete obsolete scrubbing tests and track deferred coverage; outcome=completed; commit=386a0d0b4

## Task workflow update - 2026-10-04T02:49:20+00:00
- Moved IN-PROGRESS → DONE.
- Merged task/2026-10-03-fix-invalid-utf8-command-output-result-persistence into integration checkout.
- Merge made by the 'ort' strategy.
 src/AgentCore/Application/AGENTS.md                                              |  15 +++++++++
 src/AgentCore/Application/Handler/ToolBatchCollector.php                         |   1 +
 src/AgentCore/Application/Handler/ToolCallResultFactory.php                      |  16 +++++-----
 src/AgentCore/Application/Handler/ToolExecutor.php                               | 147 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++---------
 src/AgentCore/Application/Pipeline/ToolCallResultHandler.php                     |   1 +
 src/AgentCore/Application/Pipeline/ToolExecutionEndPayloadCodec.php              |  30 ++++++++++++++++--
 src/AgentCore/Contract/Tool/MalformedToolResultException.php                     |  15 +++++++++
 src/AgentCore/Domain/Message/AgentMessageNormalizer.php                          |   5 +++
 src/AgentCore/Domain/Message/ToolCallResult.php                                  |  36 ++++++++++++++++++++++
 src/AgentCore/Domain/Tool/ToolResultText.php                                     |  69 ++++++++++++++++++++++++++++++++++++++++++
 src/CodingAgent/Runtime/Controller/BackgroundProcessCompletionPoller.php         |  11 +++++++
 src/CodingAgent/Runtime/Controller/CommandHandler/ExecuteShellToolCallWorker.php |   4 +--
 src/CodingAgent/Tool/BackgroundProcess/ProcessLifecycle.php                      |  99 +++++++++++++++++++++++++++++++++++++++++++-----------------
 src/CodingAgent/Tool/BackgroundProcessManager.php                                |  16 ++++++++--
 src/CodingAgent/Tool/BashTool.php                                                |  10 +++---
 src/CodingAgent/Tool/BgStatusTool.php                                            |   3 ++
 src/CodingAgent/Tool/CodeMode/CodeModeDiagnostics.php                            |   5 +++
 src/CodingAgent/Tool/CodeMode/CodeModeDiagnosticsToolResultProcessor.php         |  13 +++-----
 src/CodingAgent/Tool/CodeMode/CodeModeHostBridge.php                             |  20 ++++++++----
 src/CodingAgent/Tool/CodeMode/CodeModeIpc.php                                    |   3 +-
 src/CodingAgent/Tool/CodeMode/CodeModeValueCodec.php                             | 106 +++++++++++++++++++---------------------------------------------
 src/CodingAgent/Tool/CodeMode/Resources/bootstrap.php.inc                        |  12 +++++---
 src/CodingAgent/Tool/CodeModeTool.php                                            |  10 +++++-
 src/CodingAgent/Tool/OutputCapToolResultProcessor.php                            |   2 +-
 24 files changed, 493 insertions(+), 156 deletions(-)
 create mode 100644 src/AgentCore/Contract/Tool/MalformedToolResultException.php
 create mode 100644 src/AgentCore/Domain/Tool/ToolResultText.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-10-03-fix-invalid-utf8-command-output-result-persistence.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-10-03-fix-invalid-utf8-command-output-result-persistence.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub merge verified; both integration checkout and task worktree clean before transition.; Post-merge runtime validation remains unperformed by explicit user restriction.
- Summary: User merged PR549; GitHub confirms MERGED at4ab72e083598693ec0e76c7cc867f6214fccf1e9. Mark done and clean up task worktree. Replacement tests remain tracked separately. Do not run tests or reproductions; user's prohibition remains active.
