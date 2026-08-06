# Design distribution and lifecycle for extension-owned skills

## Goal
## Goal

Determine the smallest supported way for a Hatfield extension Composer package to distribute agent skills alongside its PHP extension code, using task-workflow and its separate `task-workflow` skill as the first concrete case.

## Problem

The extension packages can now be released and installed independently, but related skills are still managed separately. Installing an extension elsewhere should not require undocumented manual copying, and uninstall/update behavior must not leave stale skill files.

## Investigation scope

1. Trace current Hatfield skill discovery, precedence, loading and project/user locations.
2. Inventory the task-workflow PHP extension and task-workflow skill relationship: duplicated guidance, ownership, version coupling and runtime assumptions.
3. Evaluate minimal distribution options, including:
   - skills stored inside the Composer package and discovered in place;
   - Composer install/update hooks copying skills into a project location;
   - an explicit Hatfield extension install/sync command;
   - keeping skills independent when coupling is not actually required.
4. Define lifecycle semantics for install, update and removal without stale files or hidden mutation.
5. Define naming/collision and precedence behavior between built-in, user, project and extension-owned skills.
6. Check compatibility with source installs, split repositories, PHAR/static distribution and project-local extensions.
7. Recommend one minimal model and a migration plan for task-workflow. Identify any required ExtensionApi or package-manifest surface explicitly; avoid speculative general plugin infrastructure.

## Non-goals

- Implementing the chosen model in this task unless the user separately finalizes and starts implementation.
- Migrating every existing skill.
- Building a general marketplace, registry or remote skill updater.
- Adding compatibility copies or dual discovery paths without an explicit migration requirement.

## Output

A decision document or task plan containing current-state evidence, option comparison, recommended package layout/discovery flow, lifecycle and collision rules, migration steps, risks, and exact validation strategy.

## Acceptance criteria
- Current skill discovery and precedence are documented with concrete paths and code references.
- The task-workflow extension/skill coupling and duplicated responsibilities are identified.
- At least the in-package discovery, Composer-copy, explicit-sync, and independent-skill options are compared against install/update/remove safety.
- A single minimal recommendation is made for source, PHAR/static, and externally installed extension packages.
- Any new public API, manifest field, command, setting, or user-visible behavior is listed as an explicit decision requiring user approval.
- A follow-on implementation plan includes migration boundaries and focused validation without expanding into a marketplace or general package manager.

## Workflow metadata
Status: ARCHIVE
Branch: task/design-extension-owned-skill-distribution
Worktree: /home/ineersa/projects/agent-core-worktrees/design-extension-owned-skill-distribution
Fork run: a8hqb62j7hu9
PR URL: https://github.com/ineersa/agent-core/pull/364
PR Status: merged
Started: 2026-08-05T18:11:39.235Z
Completed: 2026-08-05T21:57:21.931Z

## Work log
- Created: 2026-08-04T23:37:36.088Z

## Task workflow update - 2026-08-05T18:11:39.235Z
- Moved TODO → IN-PROGRESS.
- Created branch task/design-extension-owned-skill-distribution.
- Created worktree /home/ineersa/projects/agent-core-worktrees/design-extension-owned-skill-distribution.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/design-extension-owned-skill-distribution.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/design-extension-owned-skill-distribution.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/design-extension-owned-skill-distribution.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/design-extension-owned-skill-distribution.
- Summary: Design finalized for implementation: add ExtensionApiInterface::registerSkill(string $absoluteSkillDirectory); bridge directly to SkillDiscovery; extension skills are enabled-extension-only, lowest precedence, suppressed by --no-skills, and use existing collision handling. Move the task-workflow skill from .agents into two deliberate copies: .pi/skills/task-workflow for Pi and package-local .hatfield/extensions/task-workflow/skills/task-workflow for Hatfield distribution. No Composer hooks, sync command, manifest metadata, InstalledVersions scanning, .pi settings changes, or optional provider interface.

## Task workflow update - 2026-08-05T18:13:41.468Z
- Recorded fork run: a8hqb62j7hu9
- Summary: Implementation fork launched with finalized registerSkill design and package/Pi skill-copy migration boundaries.

## Task workflow update - 2026-08-05T18:26:58.295Z
- Recorded fork run: a8hqb62j7hu9
- Validation: Read .agents/skills/testing/SKILL.md and tests/AGENTS.md before test work; castor test --filter=SkillDiscoveryTest: PASS (18 tests); castor test --filter=ExtensionToolRegistryBridgeTest: PASS (30 tests); Combined focused extension/skill filter: PASS (56 tests, 154 assertions); castor deptrac: PASS (0 violations); castor phpstan: PASS (0 errors); castor cs-check: PASS; cmp .pi/skills/task-workflow/SKILL.md .hatfield/extensions/task-workflow/skills/task-workflow/SKILL.md: identical; Verified clean worktree and expected 15-file commit diff
- Summary: Implementation complete at 30d95e39992bdeb8133a2750326284368afac37d. Added ExtensionApiInterface::registerSkill(), direct bridge-to-SkillDiscovery registration with cache invalidation, lowest-precedence/--no-skills behavior, task-workflow package registration, moved the shared skill into the extension package plus a byte-identical .pi copy, removed .agents copy, updated focused docs/tests and Deptrac allowance. Worktree is clean; no push/PR/full gate yet.

## Task workflow update - 2026-08-05T20:15:04.212Z
- Recorded fork run: a8hqb62j7hu9
- Validation: Reviewer re-review: APPROVED; specification fidelity satisfied; Verified vendor/ineersa/hatfield-extension-api resolves to worktree interface containing registerSkill; castor test: PASS on full rerun (4444 tests, 16592 assertions); first run had one unrelated virtual TUI flake that passed solo and on full rerun; castor deptrac: PASS (0 violations); castor phpstan: PASS (0 errors); castor cs-check: PASS; Skill copies cmp: byte-identical; Testing skill and tests/AGENTS.md read by implementation/fix forks and reviewers
- Summary: Task-to-PR review iteration complete. Initial reviewer requested changes for 8 observational-memory ExtensionApiInterface test doubles masked by a stale parent-checkout path-package symlink. Follow-up commit d792756a6c65f11fe3b0593004e1412746348ee0 added all missing no-op registerSkill implementations, corrected the test-bridge comment, and refreshed vendor resolution to the worktree API. Re-review verdict: APPROVED with no blockers; full diff matches finalized minimal design.

## Task workflow update - 2026-08-05T20:17:06.868Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 240s)...
- castor check passed (107.8s).
- Pushed task/design-extension-owned-skill-distribution to origin.
- branch 'task/design-extension-owned-skill-distribution' set up to track 'origin/task/design-extension-owned-skill-distribution'.
- Created PR: https://github.com/ineersa/agent-core/pull/364
- Validation: Full castor test PASS on rerun: 4444 tests / 16592 assertions; castor deptrac PASS; castor phpstan PASS; castor cs-check PASS; Reviewer APPROVED; Skill copies byte-identical
- Summary: Implementation and review complete at d792756a6. Adds direct extension registerSkill API/discovery, distributes task-workflow skill in package plus Pi copy, and preserves minimal lifecycle/precedence semantics. Initial review blocker for stale-autoload-masked API doubles was fixed; re-review APPROVED.

## Task workflow update - 2026-08-05T21:57:21.931Z
- Moved CODE-REVIEW → DONE.
- Merged task/design-extension-owned-skill-distribution into integration checkout.
- Auto-merging depfile.yaml
Merge made by the 'ort' strategy.
 .../extension-api/src/ExtensionApiInterface.php    |  11 ++
 ...bservationalMemoryExtensionRegistrationTest.php |   4 +
 .../tests/ObserveBoundaryJobHandlerTest.php        |   4 +
 .../tests/ObserveBoundaryThresholdDispatchTest.php |   4 +
 .../tests/OmBeforeCompactionHookTest.php           |   4 +
 .../tests/OmQueryServiceTest.php                   |   4 +
 .../tests/OmSessionContextCommandTest.php          |   4 +
 .../tests/RecallToolHandlerTest.php                |   4 +
 .../tests/ReflectGenerationJobHandlerTest.php      |   4 +
 .hatfield/extensions/task-workflow/README.md       |   4 +-
 .../task-workflow}/skills/task-workflow/SKILL.md   |   0
 .../task-workflow/src/TaskWorkflowExtension.php    |   3 +
 .pi/skills/task-workflow/SKILL.md                  | 151 +++++++++++++++++++++
 depfile.yaml                                       |   1 +
 docs/settings.md                                   |  11 +-
 .../Extension/ExtensionToolRegistryBridge.php      |   7 +
 src/CodingAgent/Skills/SkillDiscovery.php          |  25 +++-
 .../ChildExtensionSelectionServiceTest.php         |   4 +
 .../Extension/ExtensionOwnerFilteringTest.php      |  14 +-
 .../Extension/ExtensionToolRegistryBridgeTest.php  |  14 +-
 .../FileRewindExtensionIntegrationTest.php         |  10 ++
 .../Extension/InMemoryExtensionApiBridge.php       |   5 +
 tests/CodingAgent/Skills/SkillDiscoveryTest.php    |  52 +++++++
 23 files changed, 337 insertions(+), 7 deletions(-)
 rename {.agents => .hatfield/extensions/task-workflow}/skills/task-workflow/SKILL.md (100%)
 create mode 100644 .pi/skills/task-workflow/SKILL.md
- Removed worktree /home/ineersa/projects/agent-core-worktrees/design-extension-owned-skill-distribution.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/design-extension-owned-skill-distribution.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: GitHub PR #364 state: MERGED at 2026-08-05T21:56:55Z
- Summary: PR #364 was merged on GitHub as bfaff23c423cd2545ea0a26f8c44437fb2cc4ac5. Proceeding with task-done integration and worktree cleanup.

## Task workflow update - 2026-08-05T22:00:30.343Z
- Validation: Post-merge `LLM_MODE=true castor check`: PASS (all 7 lanes); deptrac OK; test OK: 4447 tests / 16639 assertions; test:controller-replay OK: 12 tests / 165 assertions; test:tui OK: 34 tests / 222 assertions; test:llm-real OK: 13 tests / 144 assertions; phpstan OK: 0 errors; cs-check OK; llama-proxy cache guard stable: 224 → 224; QA leak assertion OK; Integration working tree clean; local main ahead of origin/main by 4 merge commits
- Summary: Task-done completed: PR #364 merged, task branch integrated, worktree and IDEA exclusions removed. Post-merge validation passed on integration main.

## Task workflow update - 2026-08-06T20:58:47.623Z
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
