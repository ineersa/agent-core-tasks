# Remove unnecessary boundary interfaces and DTOs between TUI and CodingAgent

## Goal
Audit interfaces and DTOs added solely to work around an incorrect assumption that TUI could not depend on CodingAgent. The authoritative dependency direction is: TUI may depend on CodingAgent; CodingAgent must not depend on TUI. Replace unnecessary catalog/pass-through contracts with direct dependencies where doing so removes indirection without changing behavior or stable published APIs. Start with analogous Runtime/Contract catalog DTO seams such as prompt-template command projection, but inspect provenance and usages rather than deleting legitimate runtime protocol or public extension contracts. Update Deptrac rules to express the real direction. Do not broaden into unrelated architecture redesign.

## Acceptance criteria
- Inventory candidate interfaces/DTOs with references, implementations, callers, and whether each is a stable published/runtime protocol contract.
- Remove only interfaces/DTOs whose sole purpose is enforcing the nonexistent TUI-to-CodingAgent restriction.
- Use direct TUI dependencies on the owning CodingAgent service or model where semantics match.
- Preserve user-visible behavior, command precedence, runtime protocol, and published ExtensionApi contracts.
- Update DI wiring, Deptrac rules, tests, and architecture documentation to state TUI may depend on CodingAgent while CodingAgent may not depend on TUI.
- Validate through focused Castor tests, Deptrac, PHPStan, CS checks, and the required full gate for touched TUI/runtime behavior.

## Workflow metadata
Status: ARCHIVE
Branch: task/2026-08-29-remove-useless-boundary-interfaces-and-dtos
Worktree: /home/ineersa/projects/agent-core-worktrees/2026-08-29-remove-useless-boundary-interfaces-and-dtos
Fork run: 5lbess3jmhfy
PR URL: https://github.com/ineersa/agent-core/pull/445
PR Status: merged
Started: 2026-08-29T23:54:17.471Z
Completed: 2026-08-30T23:29:06.261Z

## Work log
- Created: 2026-08-29T18:23:14.114Z

## Task workflow update - 2026-08-29T23:54:17.471Z
- Moved TODO → IN-PROGRESS.
- Created branch task/2026-08-29-remove-useless-boundary-interfaces-and-dtos.
- Created worktree /home/ineersa/projects/agent-core-worktrees/2026-08-29-remove-useless-boundary-interfaces-and-dtos.
- Copied vendor directory into /home/ineersa/projects/agent-core-worktrees/2026-08-29-remove-useless-boundary-interfaces-and-dtos.
- Installed extensions vendor into /home/ineersa/projects/agent-core-worktrees/2026-08-29-remove-useless-boundary-interfaces-and-dtos.
- Copied .vera index into /home/ineersa/projects/agent-core-worktrees/2026-08-29-remove-useless-boundary-interfaces-and-dtos.
- Updated parent IDEA worktree exclusions for /home/ineersa/projects/agent-core-worktrees/2026-08-29-remove-useless-boundary-interfaces-and-dtos.
- Created worktree .idea module at /home/ineersa/projects/agent-core-worktrees/2026-08-29-remove-useless-boundary-interfaces-and-dtos/.idea.
- Opened JetBrains project for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-29-remove-useless-boundary-interfaces-and-dtos.
- Summary: Starting implementation. Scope is to audit TUI↔CodingAgent workaround boundaries and remove unnecessary interfaces/DTOs where the owning dependency direction permits direct concrete dependencies.

## Task workflow update - 2026-08-29T23:54:46.390Z
- Recorded fork run: 0t4rb0tebf98
- Summary: Implementation assigned to one fork owner for the cohesive audit-and-removal slice. The fork will inventory candidate TUI↔CodingAgent interfaces/DTOs, remove only proven workaround seams, preserve legitimate runtime/ExtensionApi contracts, update wiring/rules/tests/docs, run focused validation, and commit. Full `castor check` is forbidden in task-start.
- Ownership: owner=fork; fork_run=0t4rb0tebf98; revision=40598db949c0d46336a28f578894b6058bf498b1; scope=audit and remove unnecessary TUI-to-CodingAgent boundary interfaces/DTOs, starting with prompt-template command projection; preserve runtime protocol and ExtensionApi; update DI/deptrac/tests/docs; focused Castor validation only; outcome=assigned; commit=none

## Task workflow update - 2026-08-29T23:59:14.009Z
- Summary: User clarified that this task must also document the authoritative dependency hierarchy so future agents do not invent boundaries: AgentCore depends on none of CodingAgent, TUI, or extensions; CodingAgent may depend on AgentCore but not TUI or extensions; TUI may depend on CodingAgent, and direct TUI→AgentCore is technically possible but discouraged as a likely design smell; extension implementation/API are separate from AgentCore/CodingAgent/TUI, those product layers may not depend on extension implementation, while extensions may depend inward through the public ExtensionApi surface. Record this in root architecture guidance and appropriate extension documentation/instructions, aligned with authoritative Deptrac rules.
- Specification clarification: document dependency direction in root AGENTS.md and appropriate extension architecture docs/instructions. AgentCore !-> CodingAgent/TUI/extensions; CodingAgent -> AgentCore, !-> TUI/extensions; TUI -> CodingAgent allowed, TUI -> AgentCore discouraged/design smell; product layers !-> extension implementation; extensions may depend through public ExtensionApi. Exact wording must match depfile.yaml and existing published ExtensionApi constraints.

## Task workflow update - 2026-08-30T00:02:31.784Z
- Recorded fork run: 0t4rb0tebf98
- Validation: castor test --filter='PromptTemplateCommandRegistrarTest|PromptTemplateServiceTest|CompactHeaderSnapshotProviderTest|CompactHeaderRegistrarTest' PASS (27 tests, 60 assertions); castor deptrac PASS (0 violations); castor phpstan PASS (0 errors) for PromptTemplateService, PromptTemplateCommandRegistrar, CompactHeaderSnapshotProvider; full castor phpstan PASS; castor cs-fix applied; castor cs-check PASS; castor docs:validate PASS (17 documents)
- Summary: Fork completed prompt-template catalog/DTO removal. TUI now injects PromptTemplateService directly; PromptTemplateCatalogInterface and PromptTemplateCommand deleted; Deptrac/docs updated for TUI→CodingAgent direction. Other Runtime/Contract seams inventoried and retained as legitimate protocol/provider boundaries. Focused validation passed; full castor check not run (task-start).
- Ownership: owner=fork; fork_run=0t4rb0tebf98; revision=40598db949c0d46336a28f578894b6058bf498b1; scope=audit and remove unnecessary TUI-to-CodingAgent boundary interfaces/DTOs, starting with prompt-template command projection; preserve runtime protocol and ExtensionApi; update DI/deptrac/tests/docs; focused Castor validation only; outcome=completed; commit=pending

## Task workflow update - 2026-08-30T00:02:37.764Z
- Recorded fork run: 0t4rb0tebf98
- Ownership: owner=fork; fork_run=0t4rb0tebf98; revision=de833de43f8faca56e1675a283e7b25e36dec34b; scope=audit and remove unnecessary TUI-to-CodingAgent boundary interfaces/DTOs, starting with prompt-template command projection; preserve runtime protocol and ExtensionApi; update DI/deptrac/tests/docs; focused Castor validation only; outcome=completed; commit=de833de43f8faca56e1675a283e7b25e36dec34b

## Task workflow update - 2026-08-30T00:03:07.276Z
- Summary: User corrected the extension dependency rule after reviewing the existing ExtensionApi design: concrete extensions should depend almost entirely on the published ExtensionApi (plus approved Symfony TUI public types for TUI extensions), not arbitrarily on product internals. ExtensionApi is a legitimate stable boundary and is out of removal scope. Its small host-side bridge/DTO mapping overhead is intentional. Root architecture guidance should distinguish this from the unnecessary internal TUI↔CodingAgent workaround seams.
- Specification correction (supersedes prior 'extensions may depend inward on product layers' wording): concrete extensions depend on ExtensionApi public contracts and approved Symfony TUI public types only; they must not depend on AgentCore, CodingAgent internals, or in-repo Ineersa\Tui. Product host code implements/consumes ExtensionApi but must not depend on concrete extension implementations. ExtensionApi DTO/adapter overhead is intentional published-boundary cost and must be preserved.

## Task workflow update - 2026-08-30T00:03:27.229Z
- Recorded fork run: 0t4rb0tebf98
- Validation: Fork read `.agents/skills/testing/SKILL.md` and `tests/AGENTS.md`.; Focused prompt-template/TUI tests — PASS, 27 tests, 60 assertions.; `castor deptrac` — PASS, 0 violations.; `castor phpstan` — PASS, 0 errors.; `castor cs-fix` then `castor cs-check` — PASS.; `castor docs:validate` — PASS, 17 documents.; IDE diagnostics on changed production files — 0 errors.; Worktree clean at de833de43f8faca56e1675a283e7b25e36dec34b.
- Summary: Fork implementation completed at de833de43f8faca56e1675a283e7b25e36dec34b. Removed only the workaround-only prompt-template catalog interface/DTO, rewired TUI directly to PromptTemplateService, preserved legitimate runtime/provider and published ExtensionApi contracts, and updated DI/Deptrac/tests/docs. A sequential main-owned documentation clarification remains to encode the complete dependency hierarchy requested after fork assignment.
- Ownership: owner=fork; fork_run=0t4rb0tebf98; revision=40598db949c0d46336a28f578894b6058bf498b1; scope=audit and remove unnecessary TUI-to-CodingAgent boundary interfaces/DTOs, starting with prompt-template command projection; preserve runtime protocol and ExtensionApi; update DI/deptrac/tests/docs; focused Castor validation only; outcome=completed; commit=de833de43f8faca56e1675a283e7b25e36dec34b

## Task workflow update - 2026-08-30T00:03:33.738Z
- Summary: Sequential follow-up assigned to main: document the explicit product/ExtensionApi dependency graph in root and extension architecture guidance, correcting the earlier ambiguity without changing ExtensionApi behavior or broadening production scope.
- Ownership: owner=main; fork_run=none; revision=de833de43f8faca56e1675a283e7b25e36dec34b; scope=document explicit AgentCore/CodingAgent/TUI/concrete-extension/ExtensionApi dependency direction in root and appropriate extension guidance, aligned with depfile.yaml; no production behavior changes; outcome=assigned; commit=none

## Task workflow update - 2026-08-30T00:05:22.536Z
- Validation: `castor docs:validate` after dependency documentation — PASS, 17 documents.; `git diff --check` — PASS.; No remaining `PromptTemplateCatalogInterface` references.; IDE diagnostics: PromptTemplateService 0 problems; PromptTemplateCommandRegistrar 0 errors and one parameter-name warning to resolve.
- Summary: Documentation clarification committed at 86c9ada38d0663e5244884824171d8af07f1aaaf. Final semantic verification found one non-behavioral inherited-method parameter-name warning in PromptTemplateCommandRegistrar; assigning a main-owned micro-fix before completion.
- Ownership: owner=main; fork_run=none; revision=de833de43f8faca56e1675a283e7b25e36dec34b; scope=document explicit AgentCore/CodingAgent/TUI/concrete-extension/ExtensionApi dependency direction in root and appropriate extension guidance, aligned with depfile.yaml; no production behavior changes; outcome=completed; commit=86c9ada38d0663e5244884824171d8af07f1aaaf
- Ownership: owner=main; fork_run=none; revision=86c9ada38d0663e5244884824171d8af07f1aaaf; scope=align PromptTemplateCommandRegistrar inherited parameter name with SlashCommandCatalogRegistrar contract and rerun focused validation; outcome=assigned; commit=none

## Task workflow update - 2026-08-30T00:06:27.008Z
- Validation: Fork focused prompt-template/TUI suite — PASS, 27 tests, 60 assertions.; Final focused PromptTemplateCommandRegistrarTest — PASS, 9 tests, 27 assertions.; `castor deptrac` — PASS, 0 violations.; `castor phpstan` — PASS, 0 errors.; `castor cs-check` — PASS, 0 files.; `castor docs:validate` — PASS, 17 documents.; IDE diagnostics on final PromptTemplateService and PromptTemplateCommandRegistrar — 0 problems.; No remaining `PromptTemplateCatalogInterface` references.; `git diff --check` — PASS.; Worktree clean at 239417e79a8ed62d26618ee15e534c1743a086dc.; Full `castor check` not run because task-start forbids it.
- Summary: Implementation complete at 239417e79a8ed62d26618ee15e534c1743a086dc. Removed the workaround-only PromptTemplateCatalogInterface and PromptTemplateCommand DTO; TUI prompt-template command/header consumers now use PromptTemplateService directly. Preserved legitimate runtime/projection/provider contracts and the published ExtensionApi. DI and Deptrac now express TUI → AppPromptTemplate. Root, Runtime, TUI, and ExtensionApi guidance now documents the explicit dependency graph and distinguishes intentional public ExtensionApi mapping from unnecessary internal wrappers. Worktree is clean. Ready for task-to-pr when requested.
- Ownership: owner=main; fork_run=none; revision=86c9ada38d0663e5244884824171d8af07f1aaaf; scope=align PromptTemplateCommandRegistrar inherited parameter name with SlashCommandCatalogRegistrar contract and rerun focused validation; outcome=completed; commit=239417e79a8ed62d26618ee15e534c1743a086dc

## Task workflow update - 2026-08-30T00:10:19.468Z
- Summary: Reopened implementation scope after parent provenance review found the fork stopped too early. Three additional Runtime/Contract interfaces were introduced explicitly to keep TUI from depending on owning CodingAgent modules and are therefore the same invalid workaround: LoadedResourcesSummaryProviderInterface, ProviderQuotaProbeServiceInterface, and TuiExtensionRegistryInterface. Their returned data DTOs remain semantically useful and are not removed solely with the interfaces. ThemeLoadedResourcesProviderInterface remains a legitimate reverse-direction seam because CodingAgent cannot depend on TUI. History/session/child transcript interfaces remain real replay/projection capability seams with broad substitution and protocol DTO contracts.
- Ownership: owner=main; fork_run=none; revision=239417e79a8ed62d26618ee15e534c1743a086dc; scope=remove additional provenance-confirmed workaround interfaces LoadedResourcesSummaryProviderInterface, ProviderQuotaProbeServiceInterface, and TuiExtensionRegistryInterface; wire TUI directly to LoadedResourcesSummaryBuilder, ProviderQuotaProbeService, and ExtensionManager; preserve semantic DTOs and legitimate reverse/replay contracts; update DI/deptrac/tests/docs; focused validation only; outcome=assigned; commit=none

## Task workflow update - 2026-08-30T00:14:07.566Z
- Ownership: owner=main; fork_run=none; revision=239417e79a8ed62d26618ee15e534c1743a086dc; scope=remove additional provenance-confirmed workaround interfaces LoadedResourcesSummaryProviderInterface, ProviderQuotaProbeServiceInterface, and TuiExtensionRegistryInterface; wire TUI directly to owning CodingAgent services; outcome=blocked; commit=none

## Task workflow update - 2026-08-30T00:14:31.975Z
- Recorded fork run: xa5dyswoacdu
- Ownership: owner=fork; fork_run=xa5dyswoacdu; revision=239417e79a8ed62d26618ee15e534c1743a086dc; scope=remove LoadedResourcesSummaryProviderInterface, ProviderQuotaProbeServiceInterface, and TuiExtensionRegistryInterface; wire TUI directly to owning CodingAgent services; update DI/deptrac/tests/docs; focused validation and commit; outcome=assigned; commit=none

## Task workflow update - 2026-08-30T00:31:04.646Z
- Recorded fork run: xa5dyswoacdu
- Validation: Fork confirmed .agents/skills/testing/SKILL.md and tests/AGENTS.md were read and followed before test work.; Focused Castor tests PASS: 24 tests, 110 assertions.; Focused provider-throw case PASS: 1 test, 5 assertions.; castor deptrac PASS: 0 violations.; castor phpstan PASS: 0 errors.; castor cs-fix then castor cs-check PASS: 0 files.; castor docs:validate PASS: 17 documents.; IDE diagnostics PASS: 0 errors.; castor check not run; reserved for task-to-pr gate.
- Summary: Fork completed additional audit correction at e07c2843b: deleted LoadedResourcesSummaryProviderInterface, ProviderQuotaProbeServiceInterface, and TuiExtensionRegistryInterface; rewired TUI directly to LoadedResourcesSummaryBuilder, ProviderQuotaProbeService, and ExtensionManager; removed aliases; added only direct Deptrac edges; retained semantic DTOs and legitimate reverse/replay/ExtensionApi contracts; replaced fake quota service with real probe plus deterministic MockHttpClient test wiring.
- Ownership: owner=fork; fork_run=xa5dyswoacdu; revision=239417e79a8ed62d26618ee15e534c1743a086dc; scope=remove LoadedResourcesSummaryProviderInterface, ProviderQuotaProbeServiceInterface, and TuiExtensionRegistryInterface; wire TUI directly to owning CodingAgent services; update DI/deptrac/tests/docs; focused validation and commit; outcome=completed; commit=e07c2843ba9c74aed15c847807956990df7fa28f

## Task workflow update - 2026-08-30T00:42:14.867Z
- Validation: Independent reviewer read root/module AGENTS.md, .agents/skills/testing/SKILL.md, and tests/AGENTS.md before review/testing.; Reviewer castor deptrac PASS: 0 violations.; Reviewer focused castor phpstan PASS: 0 errors.; Reviewer castor cs-check PASS.; Reviewer castor docs:validate PASS.; Reviewer focused tests PASS: 33 cases.; Reviewer SettingsValueResolverTest PASS (test-container boot).; Reviewer castor test:tui --filter=TuiJourneyE2eTest PASS.; Reviewer worker leak diagnostic clean.; Full castor check not run; required at task-to-pr gate.
- Summary: Independent review at e07c2843b: APPROVE WITH SUGGESTIONS; no blocking correctness, architecture, DI, security, test-quality, or specification-fidelity issues. Reviewer confirmed all five workaround-only Runtime/Contract types are removed across the full branch and no retained Runtime/Contract interface appears workaround-only. Remaining suggestions are nonblocking naming/test-helper cleanup and one now-redundant ExtensionManager result type guard.
- Review: revision=e07c2843ba9c74aed15c847807956990df7fa28f; reviewer=independent subagent; verdict=APPROVE WITH SUGGESTIONS; blockers=none; retained-interface audit=no remaining workaround-only Runtime/Contract interfaces found; suggestions=remove redundant instanceof guard in TuiProjectExtensionRegistrar, follow-up naming/helper cleanup, pre-existing HOME restoration issue in CompactHeaderSnapshotProviderTest is out of scope

## Task workflow update - 2026-08-30T02:04:57.445Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (85.2s).
- Pushed task/2026-08-29-remove-useless-boundary-interfaces-and-dtos to origin.
- branch 'task/2026-08-29-remove-useless-boundary-interfaces-and-dtos' set up to track 'origin/task/2026-08-29-remove-useless-boundary-interfaces-and-dtos'.
- Created PR: https://github.com/ineersa/agent-core/pull/445
- Validation: Focused tests PASS (24 tests, 110 assertions); independent reviewer focused suite PASS (33 cases).; Focused provider-throw test PASS (1 test, 5 assertions).; castor test:tui --filter=TuiJourneyE2eTest PASS.; castor deptrac PASS (0 violations).; castor phpstan PASS (0 errors).; castor cs-check PASS.; castor docs:validate PASS (17 documents).; IDE diagnostics PASS (0 errors).; Independent review verdict: APPROVE WITH SUGGESTIONS; no blockers.
- Summary: Removed five Runtime/Contract types created solely to enforce a nonexistent TUI→CodingAgent boundary: PromptTemplateCatalogInterface, PromptTemplateCommand, LoadedResourcesSummaryProviderInterface, ProviderQuotaProbeServiceInterface, and TuiExtensionRegistryInterface. TUI now depends directly on owning CodingAgent services. Preserved semantic DTOs, legitimate replay/projection contracts, the reverse ThemeLoadedResourcesProviderInterface seam, and published ExtensionApi contracts. Independent reviewer approved with suggestions and no blockers.

## Task workflow update - 2026-08-30T19:14:40.963Z
- Moved CODE-REVIEW → IN-PROGRESS.
- Summary: PR #445 is conflicting with origin/main (mergeStateStatus DIRTY); no review comments. Reopened for a non-destructive merge of origin/main and conflict resolution.

## Task workflow update - 2026-08-30T19:14:45.047Z
- Ownership: owner=main; fork_run=none; revision=e07c2843ba9c74aed15c847807956990df7fa28f; scope=merge current origin/main into PR #445 branch, resolve conflicts preserving both main changes and task requirements, focused validation, commit merge; outcome=assigned; commit=none

## Task workflow update - 2026-08-30T19:15:30.958Z
- Validation: After merge: git diff --check PASS.; After merge: castor docs:validate PASS (17 documents).; After merge: castor deptrac PASS (0 violations).
- Summary: Merged origin/main into PR #445 branch and resolved the sole conflict in AGENTS.md by preserving main's current workflow text together with the task's explicit dependency-direction rules. Merge commit 78f4bbb42. Worktree clean.
- Ownership: owner=main; fork_run=none; revision=e07c2843ba9c74aed15c847807956990df7fa28f; scope=merge current origin/main into PR #445 branch, resolve conflicts preserving both main changes and task requirements, focused validation, commit merge; outcome=completed; commit=78f4bbb42a8ad27264e6241111a65e88433b6be3

## Task workflow update - 2026-08-30T19:20:10.746Z
- Validation: Post-merge reviewer read testing skill and tests/AGENTS.md before QA.; Post-merge reviewer castor docs:validate PASS.; Post-merge reviewer castor deptrac PASS (0 violations/errors/warnings).; Post-merge conflict-marker scan PASS.; Post-merge worktree clean.
- Summary: Post-merge independent review at 78f4bbb42: APPROVE WITH SUGGESTIONS, no blockers. Reviewer verified the product/task tree is byte-identical to prior approved revision, origin/main workflow updates are preserved, only AGENTS.md required manual resolution, no task changes or conflict markers were lost, and dependency rules remain specification-faithful.
- Review: revision=78f4bbb42a8ad27264e6241111a65e88433b6be3; reviewer=independent subagent; scope=origin/main merge result, sole AGENTS.md conflict resolution, task preservation, workflow preservation, dependency specification fidelity; verdict=APPROVE WITH SUGGESTIONS; blockers=none

## Task workflow update - 2026-08-30T19:23:34.224Z
- Validation: Mandatory castor check FAILED: test:tui lane, TuiJourneyE2eTest follow-up stuck Working after 15s.; Failure artifact: var/reports/qa-20260830-192019-151232-9db6c7a1/check-test:tui.log.; Session evidence: events.jsonl ends at run_started sequence 9; no assistant/error event.; Structured log evidence: StartRun failed after canonical append in ActiveRunContext::remember -> RunOperationalProjectionRepository::replace with SQLSTATE HY000 database is locked; Messenger retry then handled without dispatching the missing effect.; castor clean:cleanup:workers:list PASS: no stale QA workers.
- Summary: origin/main merge is complete locally at 78f4bbb42 and independently approved, but CODE-REVIEW transition did not push because mandatory castor check failed in test:tui. Failure is a pre-existing runtime durability bug, not a merge conflict: StartRun appended canonical run_started, then RunOperationalProjectionRepository::replace() hit SQLite database-is-locked; Messenger retried after the canonical transition, treated StartRun as already applied, and never dispatched the LLM effect, leaving TUI stuck Working. No leaked workers. PR remains conflicting remotely until this gate blocker is resolved or an explicitly approved workflow exception is chosen.
- Gate blocker: revision=78f4bbb42a8ad27264e6241111a65e88433b6be3; lane=test:tui; root_cause=non-atomic canonical append plus disposable projection write allows projection lock failure after run_started, and Messenger retry observes transition already applied without replaying missing effect; outcome=blocked; commit=none

## Task workflow update - 2026-08-30T22:50:31.704Z
- Ownership: owner=fork; fork_run=pending; revision=78f4bbb42a8ad27264e6241111a65e88433b6be3; scope=investigate and fix intermittent TuiJourneyE2eTest dead run after SQLite projection lock, distinguish harness isolation defect from product retry/idempotency defect, add deterministic lowest-layer regression proof, focused contention validation, commit; outcome=assigned; commit=none

## Task workflow update - 2026-08-30T22:50:56.821Z
- Recorded fork run: 5lbess3jmhfy
- Ownership: owner=fork; fork_run=5lbess3jmhfy; revision=78f4bbb42a8ad27264e6241111a65e88433b6be3; scope=investigate and fix intermittent TuiJourneyE2eTest dead run after SQLite projection lock, distinguish harness isolation defect from product retry/idempotency defect, add deterministic lowest-layer regression proof, focused contention validation, commit; outcome=assigned; commit=none

## Task workflow update - 2026-08-30T23:03:33.311Z
- Recorded fork run: 5lbess3jmhfy
- Validation: Fork confirmed testing skill and tests/AGENTS.md read and followed.; Focused StartRun/isolation tests PASS: 11 tests, 78 assertions.; Deterministic PDO same-token/different-lane contention proof PASS.; castor deptrac PASS: 0 violations.; castor phpstan PASS: 0 errors.; castor cs-fix/cs-check PASS.; castor docs:validate PASS: 17 docs.; Worker leak diagnostic PASS: no stale QA workers.; IDE diagnostics PASS: 0 errors.; Full castor check pending parent transition gate.
- Summary: Fork completed SQLite/TUI gate flake fix at 71559ae39. Root cause had two layers: concurrent castor-check ParaTest lanes reused identical TEST_TOKEN DB/cache names, causing cross-lane SQLite lock contention; after run_started append, StartRun redelivery no-op skipped the initial AdvanceRun kickoff. Fix adds HATFIELD_QA_LANE to per-worker isolation and re-arms AdvanceRun only for the narrow started-but-never-advanced state. Added deterministic isolation and redelivery regression tests.
- Ownership: owner=fork; fork_run=5lbess3jmhfy; revision=78f4bbb42a8ad27264e6241111a65e88433b6be3; scope=investigate and fix intermittent TuiJourneyE2eTest dead run after SQLite projection lock, distinguish harness isolation defect from product retry/idempotency defect, add deterministic lowest-layer regression proof, focused contention validation, commit; outcome=completed; commit=71559ae39e57d3fca1bc113834dd02efdb78bc94

## Task workflow update - 2026-08-30T23:14:43.856Z
- Validation: Reviewer focused StartRun/isolation tests PASS.; Reviewer castor test --suite=platform PASS (170 tests).; Reviewer deptrac PASS (0 violations).; Reviewer phpstan PASS (0 errors).
- Summary: Independent review of 71559ae39 requested changes for one blocker: exact-run cache cleanup still matches only legacy cache-<qaRunId>-paraT* roots, so new cache-<qaRunId>-<lane>-T<token> roots leak after every check. Root-cause and StartRun fixes otherwise approved as necessary, minimal, deterministic, and state-machine safe.
- Review: revision=71559ae39e57d3fca1bc113834dd02efdb78bc94; reviewer=independent subagent; scope=ParaTest lane isolation, cleanup ownership, StartRun redelivery state machine, deterministic regression; verdict=REQUEST CHANGES; blocker=update exact-run cache cleanup and regression test for lane-scoped roots

## Task workflow update - 2026-08-30T23:14:47.622Z
- Ownership: owner=main; fork_run=none; revision=71559ae39e57d3fca1bc113834dd02efdb78bc94; scope=fix exact-run cleanup for lane-scoped ParaTest cache roots, update cleanup regression and standalone cache docs, focused validation, commit; outcome=assigned; commit=none

## Task workflow update - 2026-08-30T23:16:40.333Z
- Validation: castor test --filter=ExactQaRunCacheCleanupTest PASS: 1 test, 16 assertions.; castor phpstan PASS: 0 errors.; castor cs-fix/cs-check PASS.; castor docs:validate PASS: 17 docs.; git diff --check PASS.
- Summary: Resolved reviewer blocker at 97ab59ac0: exact-run finalizer now removes both legacy and lane-scoped unit/tui/llm-real worker cache roots while preserving unknown lanes and neighboring caches; cleanup regression covers all owned root shapes; standalone cache docs corrected.
- Ownership: owner=main; fork_run=none; revision=71559ae39e57d3fca1bc113834dd02efdb78bc94; scope=fix exact-run cleanup for lane-scoped ParaTest cache roots, update cleanup regression and standalone cache docs, focused validation, commit; outcome=completed; commit=97ab59ac0aa780b80c431629d3f8125fa5e53f39

## Task workflow update - 2026-08-30T23:19:57.484Z
- Validation: Re-review read testing skill and tests/AGENTS.md before QA.; Re-review castor test --filter=ExactQaRunCacheCleanupTest PASS: 1 test, 16 assertions, 0.335s.; Re-review verdict: APPROVE WITH SUGGESTIONS; no blockers.
- Summary: Focused re-review at 97ab59ac0: APPROVE WITH SUGGESTIONS, no blockers. Reviewer confirmed all run-owned primary, legacy, and unit/tui/llm-real lane cache roots are removed; unknown lanes, neighboring runs, persistent caches, and symlink containment safeguards remain protected; regression is deterministic and exact.
- Review: revision=97ab59ac0aa780b80c431629d3f8125fa5e53f39; reviewer=independent subagent; scope=lane-scoped exact-run cache cleanup blocker and over-deletion safeguards; verdict=APPROVE WITH SUGGESTIONS; blockers=none

## Task workflow update - 2026-08-30T23:21:36.604Z
- Moved IN-PROGRESS → CODE-REVIEW.
- Running deterministic castor check in worktree (Castor wall 210s; outer cleanup guard 240s)...
- castor check passed (85.1s).
- Pushed task/2026-08-29-remove-useless-boundary-interfaces-and-dtos to origin.
- branch 'task/2026-08-29-remove-useless-boundary-interfaces-and-dtos' set up to track 'origin/task/2026-08-29-remove-useless-boundary-interfaces-and-dtos'.
- PR already exists: https://github.com/ineersa/agent-core/pull/445
- Validation: Focused StartRun/isolation tests PASS: 11 tests, 78 assertions.; Deterministic PDO same-token/different-lane contention proof PASS.; Exact-run cache cleanup test PASS: 1 test, 16 assertions.; castor test --suite=platform PASS: 170 tests.; castor deptrac PASS: 0 violations.; castor phpstan PASS: 0 errors.; castor cs-check PASS.; castor docs:validate PASS: 17 docs.; Worker leak diagnostic PASS.; Independent implementation review blockers resolved; focused re-review APPROVE WITH SUGGESTIONS.
- Summary: Merged origin/main and fixed the intermittent TUI gate failure. Castor check ParaTest pools now isolate DB/cache resources by QA lane, StartRun redelivery re-arms the initial AdvanceRun only when the run started but never advanced, and exact-run finalization removes all lane-scoped worker caches. Deterministic regressions cover lane isolation, projection-failure redelivery, and cache cleanup. Independent reviews approved with no remaining blockers.

## Task workflow update - 2026-08-30T23:29:06.261Z
- Moved CODE-REVIEW → DONE.
- JetBrains project close degraded for /home/ineersa/projects/agent-core-worktrees/2026-08-29-remove-useless-boundary-interfaces-and-dtos: ide_close_project returned isError.
- Merged task/2026-08-29-remove-useless-boundary-interfaces-and-dtos into integration checkout.
- Auto-merging config/services.yaml
Merge made by the 'ort' strategy.
 .agents/skills/testing/SKILL.md                    |  24 +--
 .castor/e2e.php                                    |   4 +-
 .castor/helpers.php                                |  16 +-
 .castor/phpunit.php                                |  12 +-
 .hatfield/extensions/extension-api/AGENTS.md       |   4 +-
 .../extensions/extension-api/docs/extension-api.md |   5 +-
 AGENTS.md                                          |   9 +-
 config/services.yaml                               |  11 +-
 config/services_test.yaml                          |   9 +-
 depfile.yaml                                       |   6 +-
 docs/async-runtime-architecture.md                 |   2 +-
 docs/tui-architecture.md                           |   2 +
 src/AgentCore/Application/AGENTS.md                |   1 +
 .../Application/Pipeline/StartRunHandler.php       |  17 +-
 src/CodingAgent/Extension/ExtensionManager.php     |   3 +-
 .../ProviderQuota/ProviderQuotaProbeService.php    |   3 +-
 .../PromptTemplate/PromptTemplateService.php       |  23 +--
 src/CodingAgent/Runtime/AGENTS.md                  |   4 +-
 .../LoadedResourcesSummaryProviderInterface.php    |  16 --
 .../Contract/PromptTemplateCatalogInterface.php    |  22 ---
 .../Runtime/Contract/PromptTemplateCommand.php     |  23 ---
 .../ProviderQuotaProbeServiceInterface.php         |  15 --
 .../Contract/TuiExtensionRegistryInterface.php     |  16 --
 .../LoadedResourcesSummaryBuilder.php              |   3 +-
 src/Tui/AGENTS.md                                  |   2 +
 .../CompactHeaderSnapshotProvider.php              |   4 +-
 .../Listener/LoadedResourcesStartupRegistrar.php   |   6 +-
 .../Listener/PromptTemplateCommandRegistrar.php    |  12 +-
 src/Tui/Listener/TuiProjectExtensionRegistrar.php  |   6 +-
 src/Tui/Listener/UsageCommandHandler.php           |   8 +-
 src/Tui/Listener/UsageCommandRegistrar.php         |   4 +-
 tests/AGENTS.md                                    |   2 +-
 .../Application/Pipeline/StartRunHandlerTest.php   |  35 +++-
 .../StartRunProjectionFailureRedeliveryTest.php    | 134 +++++++++++++++
 .../Castor/ExactQaRunCacheCleanupTest.php          |  22 ++-
 .../FakeProviderQuotaHttpClientFactory.php         |  57 ++++++
 .../FakeProviderQuotaProbeService.php              |  29 ----
 .../PromptTemplate/PromptTemplateServiceTest.php   |   6 +-
 .../Support/ParaTestWorkerIsolation.php            |  56 ++++++
 .../Support/ParaTestWorkerIsolationTest.php        |  62 +++++++
 .../CompactHeaderSnapshotProviderTest.php          |  77 +++++----
 tests/Tui/Listener/CompactHeaderRegistrarTest.php  |  31 +++-
 .../LoadedResourcesStartupRegistrarTest.php        |  33 ----
 .../PromptTemplateCommandRegistrarTest.php         | 123 +++++++------
 tests/Tui/Screen/TuiUsageCommandVirtualTest.php    | 191 ++++++++++++++++-----
 tests/paratest-bootstrap.php                       |  31 ++--
 46 files changed, 766 insertions(+), 415 deletions(-)
 delete mode 100644 src/CodingAgent/Runtime/Contract/LoadedResourcesSummaryProviderInterface.php
 delete mode 100644 src/CodingAgent/Runtime/Contract/PromptTemplateCatalogInterface.php
 delete mode 100644 src/CodingAgent/Runtime/Contract/PromptTemplateCommand.php
 delete mode 100644 src/CodingAgent/Runtime/Contract/ProviderQuotaProbeServiceInterface.php
 delete mode 100644 src/CodingAgent/Runtime/Contract/TuiExtensionRegistryInterface.php
 create mode 100644 tests/AgentCore/Application/Pipeline/StartRunProjectionFailureRedeliveryTest.php
 create mode 100644 tests/CodingAgent/Infrastructure/ProviderQuota/FakeProviderQuotaHttpClientFactory.php
 delete mode 100644 tests/CodingAgent/Infrastructure/ProviderQuota/FakeProviderQuotaProbeService.php
 create mode 100644 tests/CodingAgent/Support/ParaTestWorkerIsolation.php
 create mode 100644 tests/CodingAgent/Support/ParaTestWorkerIsolationTest.php
- Removed worktree /home/ineersa/projects/agent-core-worktrees/2026-08-29-remove-useless-boundary-interfaces-and-dtos.
- Removed IDEA exclusions for worktree /home/ineersa/projects/agent-core-worktrees/2026-08-29-remove-useless-boundary-interfaces-and-dtos.
- Pulled integration checkout: Merge made by the 'ort' strategy..
- Validation: Pre-merge full castor check PASS (85.1s).; Independent implementation and post-fix reviews approved with no blockers.
- Summary: PR #445 merged at 1731c6c0ce387623a9d2ef5a88d603711f18ada5. Removed workaround-only TUI→CodingAgent boundary types, documented actual dependency direction, isolated concurrent QA lane resources, and made StartRun redelivery recover its initial kickoff after projection failure.

## Task workflow update - 2026-08-30T23:31:26.544Z
- Validation: Post-merge LLM_MODE=true castor check PASS in 173.9s after 39.3s lock wait.; test PASS: 4815 tests, 19485 assertions.; test:controller-replay PASS: 6 tests, 88 assertions.; test:tui PASS: 8 tests, 60 assertions.; test:llm-real PASS: 5 tests, 30 assertions.; deptrac, phpstan, cs-check, docs:validate, catalog:version-check PASS.; QA artifact integrity PASS; leak check PASS; llama-proxy cache guard PASS.; Exact-run cache cleanup removed 11 owned cache roots.; Integration checkout Git status clean; task worktree removed.
- Summary: Post-merge completion verified. Integration checkout contains merged PR #445, worktree removed, Git status clean, and mandatory LLM_MODE=true castor check passed all nine lanes.

## Task workflow update - 2026-09-06T15:40:36+00:00
- Moved DONE → ARCHIVE.
- Archived task without git, worktree, PR, or branch side effects.
