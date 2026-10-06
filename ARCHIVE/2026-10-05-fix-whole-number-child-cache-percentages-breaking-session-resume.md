# Fix whole-number child cache percentages breaking session resume

## Goal
User resumed session75 on main after PR550 and received runtime transport error: SubagentProgressSingleSnapshotDTO cacheReadHitPercentage expectedfloat|null but gotint. JSON drops fractional zeros, while progress payloads are denormalized from arrays without JSON format context. Main owns narrow wire-contract correction and regression proof for single snapshots and parallel child rows. No forks, no subagents except existing read-only reviewer. Do not modify session75 data or signal active/root-owned workers. Preserve fractional values, explicit zero and missing telemetry; keep strict rejection of nonnumeric data. No settings or storage changes.

## Acceptance criteria
- Whole-number percentages survive production JSONL encode/decode and progress DTO hydration for single and parallel snapshots.
- Fractional percentages and missing telemetry retain their values; nonnumeric percentages remain invalid.
- Add a deterministic failing regression through JSONL transport and existing serializer/projection facilities, not direct render-only proof.
- Focused Castor validation during implementation; reviewer and full transition-owned gate before PR submission.

## Workflow metadata
Status: DONE
Branch: task/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/551
PR Status: merged
Started: 2026-10-05T02:07:56+00:00
Completed: 2026-10-05T13:31:17+00:00

## Work log
- Created: 2026-10-05T02:07:42+00:00

## Task workflow update - 2026-10-05T02:07:56+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume/.idea.
- Summary: Investigate and fix the PR550 transport regression: JSON whole-number cache percentages must remain valid numeric values when hydrated into progress DTOs. Main owns; no session data edits or worker operations.

## Task workflow update - 2026-10-05T02:08:08+00:00
- Summary: Applicable runtime/protocol instructions, testing skill and tests/AGENTS loaded. JsonlCodec encodes whole-valued floats as JSON integers; consumers hydrate progress arrays without format context. Smallest candidate is numeric int|float|null types on the two new wire fields, leaving domain arithmetic and serializer enforcement unchanged. Reproduce with existing serializer and actual JSONL codec before editing production.
- Ownership: owner=main; fork_run=none; revision=cfd26aa61; scope=cache percentage JSON wire contract and round-trip regression; outcome=assigned; commit=none

## Task workflow update - 2026-10-05T02:23:07+00:00
- Validation: RED on baselinecfd26aa61: actual JSONL round trip of0.0 producedexact user exception cacheReadHitPercentage expectedfloat|null gotint. Durable reportvar/reports/cache-percentage-red/.; GREEN:159 focused serializer/codec/projection/catalog/rendering tests,792 assertions,1.2s; maximumcase0.028197s.; GREEN:14 deterministic controller-replay tests,261 assertions,51.4s; maximumcase5.61247s.; Full PHPStan,style check,Deptrac andgitdiff--check passed. No tests deleted or relaxed; numeric strings stayinvalid.; Task worktree clean after821c2f475. Fullcastorcheck deferred toCODE-REVIEW transition bytask-start procedure.
- Summary: Implemented narrow transport correction in821c2f475: both new cache percentage wire fields accept int|float|null. Domain arithmetic, JSON encoding, serializer type enforcement, keys, storage and settings remain unchanged. Added actual builder→normalizer→JsonlCodec encode/decode→discriminator hydration coverage for single and parallel snapshots:0%,100%,fractional and absent telemetry; separate targeted negative controls reject numeric strings. The old virtual-render tests bypassed JSON conversion, missing whole-valued float→integer loss. Main's session75 data and active workers untouched. Implementation complete; task-to-pr is next, reviewer/full transition gate not yet run.
- Ownership: owner=main; fork_run=none; revision=cfd26aa61; scope=cache percentage JSON wire contract and round-trip regression; outcome=completed; commit=821c2f475

## Task workflow update - 2026-10-05T13:12:49+00:00
- Summary: User requested PR creation. Task-to-pr procedure loaded; clean worktree and three-file diff confirmed. Resume existing reviewer context for this follow-up to merged PR550; reuse applicable focused proof and let CODE-REVIEW transition own full gate.
- Reviewer: role=independent read-only specification-fidelity review; artifact=agent_7828ac10c55bcd87; revision=821c2f475; scope=whole-number cache percentage transport fix and numeric wire regressions; outcome=assigned

## Task workflow update - 2026-10-05T13:21:50+00:00
- Validation: 159 focused cases792assertions and14controller-replay261assertions remain valid on821c2f475; static/style/architecture green. Full castorcheck runs viaCODE-REVIEW transition.; Reviewer inspected red report and confirmed production resume no-format denormalization; widened numeric wire fields preserve values and reject numericstrings.
- Summary: Independent reviewer APPROVE at821c2f475 with no blockers; verified actual resume consumer and numeric union call-site compatibility. User repeated the same error on still-unpatched main. Proceed directly to transition-owned full gate and PR creation; no production scope changes or worker operations.
- Reviewer continuation: prior artifact=agent_7828ac10c55bcd87 could not be resumed because it belongs to a previous parent lifetime. Created reviewer-only context in current lifetime, no implementation delegation.
- Reviewer: role=independent read-only specification-fidelity review; artifact=agent_d23ff674748cd2f2; revision=821c2f475; scope=whole-number percentage hydration, transport regressions and consumer compatibility; verdict=APPROVE; testing skill and tests/AGENTS.md read and followed; no blockers.

## Task workflow update - 2026-10-05T13:23:51+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (98.4s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume/var/reports/qa-20261005-132213-632-a4adf2d4.
- Session/run: 79.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-10-05T13:23:54+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume/var/reports/qa-20261005-132213-632-a4adf2d4.
- Session/run: 79.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-10-05T13:24:04+00:00
- castor check passed (98.4s).
- Pushed task/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume to origin.
- Created PR: <url>
- Session/run: 79.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-10-05T13:24:04+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (98.4s).
- Pushed task/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/551
- Summary: Reviewer-approved821c2f475 fixes session-resume transport crash from whole-number cache percentages. Run required fullgate, push and create hotfixPR.

## Task workflow update - 2026-10-05T13:24:57+00:00
- Updated PR URL: https://github.com/ineersa/agent-core/pull/551
- Updated PR Status: open
- Validation: castorcheck PASS in98.4s; reportsvar/reports/qa-20261005-132213-632-a4adf2d4/.; JUnit5044 cases,maximum6.180611s,noneover10s.; Reviewer agent_d23ff674748cd2f2 APPROVE at821c2f475,testing prerequisites confirmed.; GitHub PR551 OPEN,head821c2f4751f7ee1396554ac2425f06c4498f43b3; clean taskworktree.
- Summary: Created PR551 for approved821c2f475 after full transition-owned gate passed. GitHub head verified exact revision; worktree clean. Main remains unchanged until merge and deployment.

## Task workflow update - 2026-10-05T13:31:17+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume into integration checkout.
- Merge made by the 'ort' strategy.
 src/CodingAgent/Runtime/Contract/SubagentProgress/SubagentProgressChildRowDTO.php       |  3 ++-
 src/CodingAgent/Runtime/Contract/SubagentProgress/SubagentProgressSingleSnapshotDTO.php |  3 ++-
 tests/CodingAgent/Runtime/Contract/SubagentProgressSnapshotSerializerTest.php           | 62 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 3 files changed, 66 insertions(+), 2 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-10-05-fix-whole-number-child-cache-percentages-breaking-session-resume.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Summary: GitHub confirms PR #551 merged as b4f2401a7113d5f57fd01d34c5c33feb30c5f8e6. Integrate and pull main, remove task worktree, then run required post-merge Castor gate.

## Task workflow update - 2026-10-05T13:33:13+00:00
- Updated PR Status: merged
- Validation: Post-merge integration castorcheck PASS:all11 lanes. Reportsvar/reports/qa-20261005-133128-5829-7c70a37a/.; JUnit5044 passingcases;maximum9.673869s;noneover10s.; QA-owned process/tmux leakcheck clean;proxycache stable406→406.; IntegrationGitstatus clean;worktreeremovalverified;canonicalPHARsmokepassed afterrebuild.
- Summary: PR551 merged on GitHub as b4f2401a7113d5f57fd01d34c5c33feb30c5f8e6. Integration main updated and post-merge validated at2a9368f3a. Task worktree removed and Git status clean. Canonical PHAR rebuilt by Castor; active session copies/workers untouched.
