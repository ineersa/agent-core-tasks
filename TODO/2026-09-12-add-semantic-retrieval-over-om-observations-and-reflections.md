# Add semantic retrieval over OM observations and reflections

## Goal
Follow-up to 2026-09-12-add-cross-session-om-text-search-and-recall. Add meaning-based discovery of prior work, for example finding session 32 from 'where we upstreamed flat DTO arguments' without knowing PR #2510 or MapToolArguments.

Reuse the base task's result provenance and cross-session recall. Investigate existing Symfony AI retrieval, embeddings, and vector-store facilities before proposing a design. The fact OM already invokes an agent does not by itself supply an embedding index. Avoid a second memory-summary pipeline.

Finalize provider/model selection, index storage, backfill and incremental updates, retention handling, and ranking integration after the base task is implemented. This task is deferred; no semantic implementation in the base task.

## Acceptance criteria
- Inspect Symfony AI facilities and propose the smallest compatible semantic retrieval design before implementation.
- Retrieve observations and reflections by meaning while preserving exact-text lookup for identifiers.
- Return source-session and memory references usable by cross-session recall.
- Define indexing ownership, backfill, updates, privacy implications, and failure behavior explicitly.
- Demonstrate retrieval quality and cost on representative prior-session queries, including the Symfony AI PR #2510 example.

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
- Created: 2026-09-12T22:14:09+00:00
