---
type: grilling
blocked_by: [03, 04, 05, 06, 07, 11]
undermined_by: []
assets: []
---
# Define historical session import semantics

## Question

How should v1 derive typed semantic memories from pre-existing Codex, Claude Code, and Grok sessions on macOS/Linux while retaining provider transcripts as the source of record? Given the provider/rollout contract and common ingestion semantics, decide session discovery and selection, supported historical formats, import provenance and authority, overlap with live capture, stable deduplication, progress and cancellation, checkpoint/restart behavior, incomplete evidence handling, and resource budgets. Distinguish historical session import from adopting a pre-existing semantic memory database. Do not silently drop Grok or turn SurrealDB into a transcript archive.

Options for discussion (not decisions):

- Feed historical sessions through a common capture/extraction contract with import-specific discovery and checkpoints.
- Use a dedicated batch-import pipeline with an explicit compatibility boundary to live capture.

## Done when

The human agrees to import scope, compatibility/error reporting, provenance and authority invariants, replay/deduplication semantics, operational controls, and acceptance examples for every required provider. Create bounded research or prototype tickets for unresolved factual or behavioral questions rather than guessing.

## Ruled out

The operator clarified that historical access means reusing the already-running SurrealDB instance and its stored memories/history. Separate import of older provider sessions not already stored there is not required in v1. This ticket was opened from an overly broad interpretation of "old sessions" and sits beyond the clarified destination. Existing-history compatibility remains in [Define compatibility with existing SurrealDB history](13-existing-surrealdb-history.md); new-work extraction remains in [Define automatic capture and extraction semantics](07-automatic-capture-contract.md).
