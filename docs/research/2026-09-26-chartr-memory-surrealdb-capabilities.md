# SurrealDB capabilities for Chartr memory

Date: 2026-09-26. Research baseline: **SurrealDB 3.3.0**, released September 24, 2026 and identified as latest stable by the official [release notes](https://surrealdb.com/releases/3.3). This is a capability assessment, not a decision to adopt that release or a deployment architecture. Input: the primary checkout's `surrealdb-agent-memory-ontology-design.md`, especially sections 8–10, 13, and 16.

Method: official documentation and release evidence only. **No database was installed, started, queried, migrated, or restored.** All query behavior below is documented, inferred, or explicitly awaiting an experiment. Live documentation can evolve independently of the named release; the final executable contract must be checked against the pinned binary.

## Findings and corrections

| Area | Evidence and consequence for the draft |
| --- | --- |
| Closed schema | `SCHEMAFULL` is supported. Since 3.0, undeclared fields cause errors instead of being silently discarded. Enumerate every persisted field, including queue/revision metadata; do not treat the example schema as complete. Typed relation endpoints can be unions. [DEFINE TABLE](https://surrealdb.com/docs/reference/query-language/statements/define/table) |
| Optional data | `option<T>` permits `NONE`, not automatically `NULL`. Correct the draft's “null = current” wording for `valid_until`, or explicitly widen its type. `option<array<float>>` permits absent embeddings; exact-length arrays are also supported. `TYPE option<object> FLEXIBLE` supports arbitrary object keys while defined subfields retain validation. [DEFINE FIELD](https://surrealdb.com/docs/reference/query-language/statements/define/field) |
| Timestamps | Plain `DEFAULT time::now()` supplies a creation default; it does not keep `updated_at` current. The draft already calls this server-maintained: that responsibility must appear in every write path. `READONLY` and assertions are documented field mechanisms. [DEFINE FIELD](https://surrealdb.com/docs/reference/query-language/statements/define/field) |
| Relation integrity | Endpoint table restrictions do not imply endpoint existence. `ENFORCED` adds that admission check. It is absent from the draft's `affects` example. Decide whether nonexistent targets are permissible. [DEFINE TABLE](https://surrealdb.com/docs/reference/query-language/statements/define/table) |
| Relation cardinality | Composite `UNIQUE` indexes enforce unique field combinations. Thus `(in,out)` prevents duplicate pairs but permits one source with several targets. **Inference:** the draft's one-outgoing-`supersedes` rule needs a separate invariant, such as uniqueness on `in` if it applies to all edges. “Current” conditional cardinality and incoming-many history semantics need clarification before choosing enforcement. [DEFINE INDEX](https://surrealdb.com/docs/reference/query-language/statements/define/indexes) |
| Full-text/vector syntax | The draft's `FULLTEXT ANALYZER … BM25` is correct for 3.x; older documentation uses `SEARCH ANALYZER`. HNSW, cosine distance, and F32 are supported. HNSW requires memory capacity planning. Treat `$EMBED_DIM` as the draft explicitly intends: a schema-template placeholder rendered to a validated integer, not a demonstrated bind parameter. [DEFINE INDEX](https://surrealdb.com/docs/reference/query-language/statements/define/indexes) |
| Hybrid fusion | `search::rrf(lists, limit, k)` exists; `k` defaults to 60. Inputs must already be ordered most relevant first. It merges candidates by ID and returns `rrf_score`. The draft need not retain “if available” for this baseline, but must specify per-table/title/body ranking, candidate limits, deduplication, and deterministic ties. [Search functions](https://surrealdb.com/docs/reference/query-language/functions/database-functions/search) |
| Filtered retrieval | 3.3 combines eligible indexed predicates through bitmap operations and can prefilter vector searches. It still rechecks the full predicate and permissions. Indexes built on 3.2 need rebuilding to participate. This does **not** establish that Chartr's set membership, project-or-global condition, sensitivity, status, validity time, and graph expansion all receive the intended plan. Use `EXPLAIN` and selective fixtures. [Query optimisation](https://surrealdb.com/docs/learn/querying/concepts-and-guides/query-optimisation) |

The vendor's [filtered vector search investigation](https://surrealdb.com/blog/why-does-my-vector-search-return-nothing-when-i-add-a-filter-2), published before 3.3, illustrates why postfiltering can underfill results and why tests must inspect the plan and actual caller credentials. It also documents literal K/EF operands for the investigated version. Do not extrapolate its exact planner behavior to 3.3; validate the generated query for configurable `k`. Scope checks must also cover one-hop expansion and direct reads, an application contract beyond ANN ranking.

## Atomic writes and concurrent lifecycle changes

`BEGIN … COMMIT` groups statements atomically; individual statements otherwise have their own transactions. A failed statement rolls the group back. **Design consequence:** changing lifecycle status, writing supersession/reclassification edges, and recording durable follow-up work can share a transaction. Atomicity cannot include an external embedding API call. [BEGIN](https://surrealdb.com/docs/reference/query-language/statements/begin)

In 3.3, `SELECT … FOR UPDATE` registers record IDs for commit-time conflict checks; it does not take a waiting row lock. Records written by the transaction already receive write-conflict protection. Read-only decision inputs need explicit protection when their concurrent change would invalidate the write. Even missing IDs can be registered, but predicates/ranges are not protected against new matching records. **Inference:** “count independent runs, then promote” cannot rely on a plain count and assume serializable behavior. Define a common record/invariant that competing transactions must check or write, plus bounded retry and idempotency behavior. [SELECT: FOR UPDATE](https://surrealdb.com/docs/reference/query-language/statements/select)

Required race cases remain unrun: two superseders; two promotions using overlapping evidence; duplicate ingest delivery; two reclassifications; and an embedding result arriving after its source body changed. The latter needs a content/revision association regardless of the database's vector support.

## Schema evolution and local operation

`IF NOT EXISTS` and `OVERWRITE` exist, but creation guards do not update stale definitions. Overwriting definitions is not a complete data migration. The index page has contradictory prose about the existing-index result of `IF NOT EXISTS`; verify the repeat-apply result directly rather than treating prose as a passed idempotency test. [DEFINE INDEX](https://surrealdb.com/docs/reference/query-language/statements/define/indexes)

The vendor distinguishes engine upgrades from application migrations: 3.3 applies datastore migrations automatically at first startup; its [upgrade guide](https://surrealdb.com/docs/build/migrating/from-old-surrealdb-versions/32-to-33) covers compatibility and index considerations. Chartr still needs a schema version, serialized migration ownership, populated-data backfills, interruption handling, and model/dimension migration rules. Rebuilding an HNSW index does not itself regenerate embeddings; a change of embedding model can require new vectors even when dimensions match. These are design consequences, not supplied migration code.

Available local interfaces leave packaging open:

- The CLI starts a separate server with an explicit storage path. RocksDB and SurrealKV are documented persistent backends; sync policy is configurable. Default startup storage is memory, so durable deployment must specify its intended backend/path. Bind, authentication, and capability flags are available. [start](https://surrealdb.com/docs/reference/cli/surrealdb-cli/commands/start)
- The official Docker image includes server and CLI; persistence requires a mounted directory and an on-disk backend. [Docker](https://surrealdb.com/docs/running/docker)
- Embedded Node.js/browser engines are also released with 3.3 (`@surrealdb/node-native` and `@surrealdb/wasm-native`). This establishes availability, not compatibility with Chartr's packaging or every target OS/architecture. [3.3 release notes](https://surrealdb.com/releases/3.3)
- `surreal export` produces a database-scoped SurrealQL file; import consumes it. Preserve generated `OPTION IMPORT`: it controls replay side effects and is required on the documented import path. [export](https://surrealdb.com/docs/reference/cli/surrealdb-cli/commands/export)
- Import commits statements incrementally, so failure can leave a partial target. Restore validation must use a fresh target and compare graph edges as well as vertices. [Import and export](https://surrealdb.com/docs/manage/instances/import-and-export) The 3.3 notes specifically fix earlier export restores dropping `ENFORCED` edges. [3.3 release notes](https://surrealdb.com/releases/3.3)

## Remaining acceptance experiments and decisions

Before implementation readiness, specify acceptance evidence for: generated schema apply twice; migration over populated data; omission versus `NONE`/`NULL`; absent-to-present and removed embeddings under HNSW; wrong dimensions; duplicate edges and endpoint absence; the lifecycle races above; filtered ranking against an exact baseline; FTS-only fallback; and export/restore including edge counts, provenance, and search rebuild readiness. None has passed in this research session.

Sharp questions for existing schema/runtime/recovery tickets:

1. Does `supersedes` prohibit multiple outgoing edges for all time, or only simultaneous current edges, and can several successors replace one predecessor?
2. Which backend, pinned release, process owner, and migration owner serve all worktrees, with what conflict/retry contract?
3. Which recall shortfall and latency are acceptable under narrow scopes, and when should exact filtered search replace or supplement ANN?
4. What restore verification, interruption recovery, and embedding-model transition must complete before memory becomes available again?

No new ontology, deployment architecture, or lifecycle policy is chosen here. Dataset size and platform packaging behavior remain fog until those decisions supply concrete constraints.
