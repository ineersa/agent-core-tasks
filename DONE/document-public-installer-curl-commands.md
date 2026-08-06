# Document public installer curl commands in README

## Goal
README install section currently shows `bash installer/bash-installer`, which only works inside a source checkout. Add copy/paste public HTTPS curl-pipe commands for default PHAR and `--static`, retain explicit version/custom install-dir examples, and state PHAR vs native requirements. Keep packaging detail linked rather than expanding README.

## Acceptance criteria
- README shows a copy/paste curl command for PHAR installation from raw.githubusercontent.com.
- README shows a copy/paste curl command for native/static installation using --static.
- Examples make latest/default versus --version=vX.Y.Z clear and retain ~/.local/bin/PATH guidance.
- No installer or release behavior changes; concise README-only diff.

## Workflow metadata
Status: DONE
Branch: task/document-public-installer-curl-commands
Worktree: /home/ineersa/projects/agent-core-worktrees/document-public-installer-curl-commands
Fork run: 0nejunucd7sj
PR URL: https://github.com/ineersa/agent-core/pull/333
PR Status: merged
Started: 2026-07-28T18:33:40.332Z
Completed: 2026-07-28T20:33:33.297Z

## Work log
- Created: 2026-07-28T18:33:35.035Z

## Task workflow update - 2026-07-28T18:33:40.332Z
- Moved TODO → IN-PROGRESS.
- Created branch task/document-public-installer-curl-commands.
- Created worktree /home/ineersa/projects/agent-core-worktrees/document-public-installer-curl-commands.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/document-public-installer-curl-commands.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/document-public-installer-curl-commands.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/document-public-installer-curl-commands.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/document-public-installer-curl-commands.
- Summary: User identified README only documents repo-local installer invocation; implementing concise public curl commands.

## Task workflow update - 2026-07-28T18:50:13.014Z
- Recorded fork run: 0m4oyknoc3k4
- Validation: Manual Markdown/diff inspection PASS; Single README.md file changed; No Castor QA needed for prose-only command documentation; Worktree clean at 555bac73063536796d23e882fbd0904c86ca4997
- Summary: README public install docs implemented at 555bac730: replaced repo-local installer examples with HTTPS curl-pipe commands for default PHAR and --static, plus ~/.local/bin PATH and optional version pin guidance. README-only +16/-5.

## Task workflow update - 2026-07-28T20:30:17.089Z
- Recorded fork run: 0nejunucd7sj
- Validation: Merge origin/main ce7671a62: PASS, zero conflicts; git diff origin/main...HEAD: README.md only (+16/-5); Reviewer APPROVE — no critical issues or issues; Ponytail: Lean already. Ship.; Manual command/installer contract verification PASS; Worktree clean at 83bf67f399bd286baddff8583d7c14c56c18bfa2
- Summary: Merged current origin/main into docs branch cleanly at 83bf67f39; diff remains README-only +16/-5. Reviewer APPROVE: public curl commands, installer flags, raw URL, PATH/default path, PHP/static distinction and generic pin syntax all verified; no unrelated changes.

## Task workflow update - 2026-07-28T20:30:58.559Z
- Validation: First move_task castor check: FAIL in phar:ensure writable-dir isolation (temporary smoke path); Immediate castor phar:ensure: PASS — build + list/about/agent-help/version/writable-dir/cache-isolation smokes all green
- Summary: First CODE-REVIEW gate failed during PHAR preflight because one temporary smoke run did not observe `.hatfield/cache`; no task code implicated. Immediate focused `castor phar:ensure` rebuilt the branch PHAR and all smokes passed, including writable-dir isolation and cache isolation. Retrying deterministic gate.

## Task workflow update - 2026-07-28T20:33:16.100Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (timeout 900s)...
- castor check passed (127.0s).
- Pushed task/document-public-installer-curl-commands to origin.
- branch 'task/document-public-installer-curl-commands' set up to track 'origin/task/document-public-installer-curl-commands'.
- Created PR: https://github.com/ineersa/agent-core/pull/333
- Validation: Reviewer APPROVE; README-only diff (+16/-5); Manual installer command contract verification PASS; castor phar:ensure PASS including all PHAR smokes; Worktree clean at 83bf67f39
- Summary: README public curl installer commands approved; focused PHAR retry passed after one transient preflight failure.

## Task workflow update - 2026-07-28T20:33:33.297Z
- Moved CODE-REVIEW → DONE.
- Merged task/document-public-installer-curl-commands into integration checkout.
- Merge made by the 'ort' strategy.
 .castor/distribution.php                           | 70 ++++++++++++++++++++--
 README.md                                          | 21 +++++--
 docs/static-packaging.md                           | 16 +++--
 .../Distribution/CanonicalPharHandoffTest.php      | 16 +++++
 tools/static/README.md                             | 16 ++++-
 tools/static/phpmicro-linux-self-path.patch        | 22 +++++++
 tools/static/pin.json                              |  4 +-
 7 files changed, 149 insertions(+), 16 deletions(-)
 create mode 100644 tools/static/phpmicro-linux-self-path.patch
- Removed worktree /home/ineersa/projects/agent-core-worktrees/document-public-installer-curl-commands.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/document-public-installer-curl-commands.
- Pulled integration checkout: Already up to date..
- Validation: PR #333 mergeable; Reviewer APPROVE; castor check PASS (127.0s); README-only diff (+16/-5)
- Summary: User explicitly requested the README installer fix be pushed to main. PR #333 is mergeable, reviewer-approved, README-only, and deterministic castor check passed.

## Task workflow update - 2026-07-28T20:34:04.459Z
- Validation: git push origin main PASS (ce7671a62..0a3fc616e); PR #333 state MERGED; origin/main and local main synchronized; git status clean
- Summary: README installer docs are now on origin/main. Direct main push completed per user request; GitHub recognized PR #333 as merged at 0a3fc616ef133fbb056e9295d7620aa20263ce7e. Integration checkout clean and synchronized.
