---
type: grilling
blocked_by: [01, 03, 04, 05, 06]
undermined_by: []
assets: []
---
# Define automatic capture and extraction semantics

## Question

For the selected providers, what counts as a completed capture unit, how is its source obtained, and how does extraction resume without omissions or duplicates? Decide hook/event versus transcript polling boundaries, cursors, replay, compaction, interrupted turns, subagents, concurrency, and operation while Chartr is closed. Define observation-to-knowledge promotion and evidence requirements, extraction model ownership/configuration, resource bounds, and behavior when evidence is partial. Specify what may be read and retained without turning SurrealDB into a transcript archive.

Options for discussion (not decisions):

- Provider completion hooks/events with durable reconciliation
- Incremental provider transcript ingestion with bounded polling
- A capability-based combination with a common ingestion contract

## Done when

The human agrees to provider capture paths, deduplication and restart semantics, provenance/retention rules, extractor/promoter boundaries, budgets, and degraded behavior.
