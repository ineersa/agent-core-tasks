# Revisit sensitive-data handling in tool failure diagnostics

## Goal
Follow-up to PR #483 and task 2026-09-08-make-tool-execution-status-and-partial-workflow-failures-reliable.

The user explicitly chose to leave the current implementation unchanged and revisit it separately. Do not block or modify PR #483 for this follow-up.

Review DiagnosticMessageSanitizer in src/AgentCore/Contract/Tool and its callers. Credential-pattern filtering does not belong in AgentCore. Regex redaction cannot guarantee that arbitrary exception text is safe. Prefer keeping credentials, auth headers, and sensitive argument dumps out of exception messages and logging fields at their source while preserving actionable tool errors.

Investigate credential-bearing parameters where PHP SensitiveParameter can protect stack traces. The attribute does not redact exception messages, explicit logs, or serialized tool results. Trace relevant diagnostics into logs, canonical events.jsonl, and model-visible results. Local execution does not imply these outputs remain local when sent to a provider or exported.

Keep scope proportionate to this local, user-operated application. Prioritize auth headers and tokens, not a general data-loss prevention system. Existing MCP-specific redaction should not be removed without inspecting its purpose. A small global logger processor for common credential patterns is an option to evaluate, not an approved implementation requirement; it would not protect events or tool results.

Preserve the bash supervision and partial-workflow reporting fixes. No automatic retry, new operation database, or unrelated error-policy redesign.

## Acceptance criteria
- Identify the producers and consumers of sensitive diagnostic data, including logs, events.jsonl, and model-visible tool results.
- Propose a minimal replacement for AgentCore-owned credential-pattern filtering that keeps actionable errors and existing failure semantics.
- Keep credentials and auth headers out of diagnostics at their owning tools or adapters; assess SensitiveParameter for trace arguments without treating it as message or serialization protection.
- Explicitly decide which existing adapter-local safeguards remain and whether a basic logger processor adds value. Do not introduce a broad sanitization framework.
- Add deterministic regression proof for any finalized changes at the lowest correct layer and follow Castor validation requirements.
- Leave PR #483 unchanged; perform implementation only after the follow-up scope is finalized.

## Workflow metadata
Status: TODO
Branch:
Worktree:
Fork run:
PR URL:
PR Status:
Started:
Completed:

## Work log
- Created: 2026-09-08T21:13:19+00:00
