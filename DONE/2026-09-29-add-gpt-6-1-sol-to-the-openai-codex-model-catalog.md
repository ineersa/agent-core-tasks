# Add GPT-6.1 Sol to the OpenAI Codex model catalog

## Goal
Add newly released gpt-6.1-sol to Hatfield's curated OpenAI Codex catalog. Check OpenAI's model documentation and pi v0.99.1 for model-specific behavior. Keep existing default selection unless the request requires a change.

## Acceptance criteria
- Bundled Codex catalog includes gpt-6.1-sol with supported reasoning levels, costs, and capped context metadata.
- Focused tests prove catalog metadata and supported effort mapping.
- Documentation and catalog version updated; focused Castor validation run.

## Workflow metadata
Status: DONE
Branch: task/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog
Fork run:
PR URL: https://github.com/ineersa/agent-core/pull/540
PR Status: merged
Started: 2026-09-29T19:11:46+00:00
Completed: 2026-09-29T22:43:33+00:00

## Work log
- Created: 2026-09-29T19:11:39+00:00

## Task workflow update - 2026-09-29T19:11:46+00:00
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog/.idea.

## Task workflow update - 2026-09-29T19:11:57+00:00
- Ownership: owner=main; fork_run=none; revision=b6cbf4d35; scope=Codex model catalog entry, focused metadata test and documentation; outcome=assigned; commit=none

## Task workflow update - 2026-09-29T19:23:53+00:00
- Validation: castor test --filter='AiCatalogTest|ReasoningOptionsResolverTest' (37 tests, 146 assertions); castor docs:validate (pass); castor catalog:version-check (pass); castor cs-check --path=tests/CodingAgent/Config (pass); git diff --check (pass)
- Summary: Added gpt-6.1-sol to the bundled OpenAI Codex catalog with 272k pinned context, 128k max output, official short-context pricing, low through max reasoning and configuration updates; bumped catalog version to 12, updated docs, and added focused catalog/resolver assertions. Compared OpenAI model documentation and pi v0.99.1; no new transport logic or default-model change needed. Independent reviewer approved.
- Ownership: owner=main; fork_run=none; revision=b6cbf4d35; scope=Codex model catalog entry, focused metadata test and documentation; outcome=completed; commit=63fb63299

## Task workflow update - 2026-09-29T19:29:32+00:00
- Validation: castor test:llm-real (5 tests, 30 assertions; pass); Prior focused tests, docs:validate, catalog:version-check, and cs-check passed on same revision. Full castor check reserved for CODE-REVIEW transition.
- Summary: Final independent specification-fidelity review approved committed revision 63fb6329928bf2eaf396f0c70f9721d097b45aae; no blockers. Reviewer artifact agent_eab31b6f98ce64dc. Worktree clean.
- Review: role=reviewer; artifact=agent_eab31b6f98ce64dc; revision=63fb6329928bf2eaf396f0c70f9721d097b45aae; scope=final diff against origin/main and specification fidelity; verdict=APPROVE; blockers=none

## Task workflow update - 2026-09-29T19:31:33+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed (112.1s).
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog/var/reports/qa-20260929-192941-2444-905c2f87.
- Session/run: 76.
- Task remains IN-PROGRESS pending push/PR.

## Task workflow update - 2026-09-29T19:31:35+00:00
- Attempted IN-PROGRESS → CODE-REVIEW.
- Completed: castor check passed; pushed task/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog to origin.
- QA reports: /home/ineersa/projects/agent-core-worktrees/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog/var/reports/qa-20260929-192941-2444-905c2f87.
- Session/run: 76.
- Task remains IN-PROGRESS pending PR creation.

## Task workflow update - 2026-09-29T19:31:40+00:00
- castor check passed (112.1s).
- Pushed task/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog to origin.
- Created PR: <url>
- Session/run: 76.
- Task remains IN-PROGRESS pending final metadata move.

## Task workflow update - 2026-09-29T19:31:40+00:00
- Moved IN-PROGRESS → CODE-REVIEW.
- castor check passed (112.1s).
- Pushed task/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog to origin.
- Created PR: https://github.com/ineersa/agent-core/pull/540

## Task workflow update - 2026-09-29T22:43:33+00:00
- Moved CODE-REVIEW → DONE.
- Merged task/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog into integration checkout.
- Merge made by the 'ort' strategy.
 config/ai-catalog.yaml                                    | 15 ++++++++++++++-
 docs/ai-catalog.md                                        |  2 +-
 docs/settings-models.md                                   | 15 ++++++++++-----
 tests/CodingAgent/Config/Ai/AiCatalogTest.php             | 17 +++++++++++++++++
 tests/CodingAgent/Config/ReasoningOptionsResolverTest.php | 19 +++++++++++++++++++
 5 files changed, 61 insertions(+), 7 deletions(-)
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-09-29-add-gpt-6-1-sol-to-the-openai-codex-model-catalog.
- Pulled integration checkout: Merge made by the 'ort' strategy..
