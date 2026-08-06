# Smoke test: IDEA worktree exclusions after restart

## Goal
Temporary smoke task created after Pi restart to verify external task workflow behavior:

- task board operations write only to /home/ineersa/projects/agent-core-tasks
- moving TODO -> IN-PROGRESS creates a git branch/worktree without committing task metadata to agent-core
- new workflow does NOT copy .idea/ into the task worktree
- new workflow adds parent worktree IDEA module sentinel exclusions
- DONE cleanup removes the sentinel block after removing the worktree

This task can be moved directly to DONE after smoke verification because it has no code changes.

## Acceptance criteria
- code repo git status remains clean after create/move
- worktree has vendor/.vera if available but no copied .idea directory
- parent agent-core-worktrees IDEA module contains pi-task-workflow sentinel block while task is IN-PROGRESS
- DONE cleanup removes worktree and sentinel block
- no task metadata commit appears in agent-core history

## Workflow metadata
Status: DONE
Branch: task/workflow-smoke-ide-exclusions-after-restart
Worktree: /home/ineersa/projects/agent-core-worktrees/workflow-smoke-ide-exclusions-after-restart
Fork run:
PR URL:
PR Status: merged
Started: 2026-06-18T18:03:48.673Z
Completed: 2026-06-18T18:04:11.411Z

## Work log
- Created: 2026-06-18T18:03:44.502Z

## Task workflow update - 2026-06-18T18:03:48.673Z
- Moved TODO → IN-PROGRESS.
- Created branch task/workflow-smoke-ide-exclusions-after-restart.
- Created worktree /home/ineersa/projects/agent-core-worktrees/workflow-smoke-ide-exclusions-after-restart.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/workflow-smoke-ide-exclusions-after-restart.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/workflow-smoke-ide-exclusions-after-restart.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/workflow-smoke-ide-exclusions-after-restart.
- Summary: Smoke step 1 after Pi restart: verify new external task workflow creates worktree without copying .idea and updates parent worktree IDEA module sentinel exclusions instead. No implementation changes will be made.

## Task workflow update - 2026-06-18T18:04:11.411Z
- Moved IN-PROGRESS → DONE.
- Merged task/workflow-smoke-ide-exclusions-after-restart into integration checkout.
- Already up to date.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/workflow-smoke-ide-exclusions-after-restart.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/workflow-smoke-ide-exclusions-after-restart.
- Deleted branch task/workflow-smoke-ide-exclusions-after-restart.
- Pulled integration checkout: Already up to date..
- Validation: After TODO→IN-PROGRESS: agent-core status remained clean (## main...origin/main); Worktree existed at /home/ineersa/projects/agent-core-worktrees/workflow-smoke-ide-exclusions-after-restart; No copied worktree .idea directory existed; vendor/ and .vera/ were copied; Parent /home/ineersa/projects/agent-core-worktrees/.idea/agent-core-worktrees.iml contained pi-task-workflow sentinel block with expected excludes
- Summary: Smoke step 2 after Pi restart: verified TODO→IN-PROGRESS created worktree without copied .idea, copied vendor/.vera, kept agent-core git status clean, and added parent worktree IDEA sentinel exclusions. Moving directly to DONE to verify worktree cleanup and sentinel removal.
