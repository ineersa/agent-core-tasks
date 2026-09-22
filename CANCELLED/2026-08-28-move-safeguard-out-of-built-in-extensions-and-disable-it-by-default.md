# Move safeguard out of built-in extensions and disable it by default

## Goal
Refactor the safeguard extension so it is no longer shipped or registered as a built-in extension. Keep it available as an explicit extension that users can opt into, and change the default configuration so safeguard is disabled unless explicitly enabled.

Determine the correct non-built-in package/location during task planning and update extension discovery, packaging, configuration examples, and documentation consistently.

## Acceptance criteria
- Safeguard is no longer included or registered as a built-in extension.
- Safeguard remains available through the project’s supported explicit extension installation/enabling mechanism.
- Fresh/default configuration does not enable safeguard.
- Existing settings examples and user documentation reflect the opt-in behavior.
- Relevant Castor validation passes.

## Workflow metadata
Status: CANCELLED
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-08-28T21:32:05.486Z

## Task workflow update - 2026-08-30T23:14:21+00:00
- Moved TODO → CANCELLED.
- No Worktree metadata; cancelled without git worktree cleanup.
- Summary: Cancelled at user request. Verified before transition that the task had not started and had no branch, worktree, or PR.
