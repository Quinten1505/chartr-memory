---
type: research
assignee: Quinten1505
research_branch: research/chartr-memory-surrealdb
blocked_by: []
undermined_by: []
assets: [surrealdb-capabilities.md]
---
# Verify SurrealDB support for the proposed memory model

## Question

Against a named current SurrealDB release and primary documentation, which parts of the draft's schema, relation constraints, optional embeddings, full-text/vector fusion, filtered retrieval, transactions, and migration strategy are supported as written, require correction, or need an experiment? Check unique relation cardinalities, SCHEMAFULL field behavior, parameterized dimensions, idempotent schema changes, and concurrent lifecycle updates. Identify available local deployment and backup interfaces without selecting the application's packaging. Separate documented capability from executed validation; no live database changes are required.

## Done when

A dated report names the checked version, cites primary sources, lists concrete corrections and unresolved experiments, and explains consequences for the later schema/runtime decisions. It does not claim unrun queries passed.

## Answer

Documented SurrealDB 3.3.0 capabilities cover the principal storage and retrieval needs, with corrections for NONE versus NULL, endpoint existence, supersession cardinality, timestamp maintenance, filtered-search planning, and migration/restore behavior. Transactions do not remove the need for application-level conflict and idempotency rules. No database queries or restores were executed. These are research findings, not adoption of a version or architecture.

See [the capability report](../../../../docs/research/2026-09-26-chartr-memory-surrealdb-capabilities.md), preserved on `research/chartr-memory-surrealdb` at `0a285fc9`. Schema and lifecycle choices remain in [Set the knowledge lifecycle and authority rules](05-knowledge-lifecycle-contract.md); deployment, search, and recovery choices remain in their existing tickets.
