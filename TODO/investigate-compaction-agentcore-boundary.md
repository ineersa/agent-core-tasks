# Investigate compaction ownership and dependency boundaries

## Goal
Separate architecture investigation prompted by FORK-MVP-03 review. Existing compaction concerns currently live partly in AgentCore, including ExecuteCompactionStepWorker and compaction contracts/messages. Observability dependencies are nullable and LoggerInterface defaults to NullLogger. Determine the correct ownership boundary without mixing this architectural debt into the fork MVP cleanup.

Investigation only first: map current compaction call graph and module dependencies; distinguish domain/core contracts from CodingAgent runtime/provider orchestration; assess whether the worker and provider invocation belong in AgentCore or CodingAgent; assess required DI for RunMetrics, RunTracer, and LoggerInterface. Produce a proposed migration sliced independently from fork behavior.

## Acceptance criteria
- Document the current compaction call graph and AgentCore/CodingAgent dependency rationale.
- Recommend the target ownership of compaction contracts, orchestration, worker, provider invocation, metrics, tracing, and logging.
- Identify Deptrac and Symfony Messenger implications of the proposed boundary.
- Provide a staged migration plan that does not change fork behavior.
- Do not implement the migration until the architecture proposal is reviewed and approved.

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
- Created: 2026-07-17T20:35:20.811Z
