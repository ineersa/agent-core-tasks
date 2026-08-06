# Add package READMEs for Extension API and File Rewind mirrors

## Goal
## Goal

Add canonical package documentation so the `ineersa/hatfield-extension-api` and `ineersa/hatfield-ext-file-rewind` split repositories no longer publish without README files.

## Scope

Create:

- `.hatfield/extensions/extension-api/README.md`
- `.hatfield/extensions/file-rewind/README.md`

Each README should be concise and package-specific, sourced from current code and existing Hatfield documentation.

### Extension API README

Document the package purpose, preserved namespace, intended Hatfield extension-author audience, Composer installation, approved Symfony TUI public dependency, compatibility/boundary expectations, and that `agent-core` is the authoritative source while the GitHub repository is a release mirror.

### File Rewind README

Document the extension’s user-visible purpose, extension class, Composer installation, minimal Hatfield settings enablement, storage/safety behavior already implemented, and the authoritative-monorepo/read-only-mirror policy. Do not invent configuration keys or behavior.

## Constraints

- Edit only canonical monorepo package directories; never edit split mirrors directly.
- No new settings, APIs, commands, compatibility paths, workflows, or release behavior.
- Reuse existing wording and facts from package source and project docs.
- Keep both documents useful on GitHub/Packagist without duplicating the full Hatfield manual.
- The next normal `v*` release will distribute them; do not create a release solely for these files.

## Validation

Documentation-only change: verify package names, namespaces, extension class, settings snippets, relative links and Markdown rendering against current source. Run only the smallest existing Castor documentation/style validation if one applies; do not run unrelated runtime/E2E tests.

## Acceptance criteria
- `.hatfield/extensions/extension-api/README.md` exists and accurately explains purpose, installation, public boundary, namespace, and mirror policy.
- `.hatfield/extensions/file-rewind/README.md` exists and accurately explains purpose, installation, extension enablement, implemented storage/safety behavior, and mirror policy.
- All documented package names, class names, settings and dependencies match current manifests/source.
- No generated mirror repository is edited directly and no new product or release behavior is introduced.
- The next release split will place both README files at the root of their respective mirrors.

## Workflow metadata
Status: CANCELLED
Branch: task/add-extension-api-file-rewind-readmes
Worktree: /home/ineersa/projects/agent-core-worktrees/add-extension-api-file-rewind-readmes
Fork run: 22993a52851af736fd9e7a6368e0c8e07aa7a435
PR URL:
PR Status:
Started: 2026-08-05T01:47:29.695Z
Completed:

## Work log
- Created: 2026-08-05T01:47:21.794Z

## Task workflow update - 2026-08-05T01:47:29.695Z
- Moved TODO → IN-PROGRESS.
- Created branch task/add-extension-api-file-rewind-readmes.
- Created worktree /home/ineersa/projects/agent-core-worktrees/add-extension-api-file-rewind-readmes.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/add-extension-api-file-rewind-readmes.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/add-extension-api-file-rewind-readmes.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/add-extension-api-file-rewind-readmes.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/add-extension-api-file-rewind-readmes.
- Summary: User requested canonical READMEs for the two split mirrors that currently lack them. Scope is documentation-only in the monorepo package directories.

## Task workflow update - 2026-08-05T01:49:56.242Z
- Recorded fork run: 22993a52851af736fd9e7a6368e0c8e07aa7a435
- Validation: Fact-checked package names, namespaces, File Rewind extension class, settings keys/defaults, storage/safety behavior, and mirror policy against current manifests/source/docs.; Final diff: 2 files, 124 insertions; worktree clean.; No runtime QA run because this is documentation-only and changes no executable behavior.
- Summary: Documentation implementation complete and committed as `22993a52851af736fd9e7a6368e0c8e07aa7a435`. Verified the worktree is clean and exactly two canonical package README files were added; no mirror, code, config, manifest, workflow, or release changes.

## Task workflow update - 2026-08-05T01:53:17.931Z
- Moved IN-PROGRESS → CANCELLED.
- Removed worktree /home/ineersa/projects/agent-core-worktrees/add-extension-api-file-rewind-readmes.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/add-extension-api-file-rewind-readmes.
- Summary: Cancelled the unnecessary task/PR workflow at the user's request. The documentation commit will be fast-forwarded directly onto the local integration `main`; no PR or separate review cycle.
