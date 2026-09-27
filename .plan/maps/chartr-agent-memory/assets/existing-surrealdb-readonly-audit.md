# Existing SurrealDB history: read-only audit

Observed 2026-09-26 for [Define compatibility with existing SurrealDB history](../tickets/13-existing-surrealdb-history.md). This is evidence for a planning decision, not a migration or compatibility decision. Only `/version`, `INFO FOR DB/TABLE`, and `SELECT` were sent to the database. Credential values, record IDs, scope suffixes, titles, bodies, paths, and URIs are omitted.

## Connection and database shape

- The Windows process bound to `127.0.0.1:8000` responds as **SurrealDB 3.2.4** (`surrealdb-3.2.4+20260803.93ab219`). The configured `harness` namespace and `memory` database accept read-only SQL using the already present environment credentials. The storage launch script uses SurrealKV. This confirms database access from this Windows shell only; remote macOS/Linux reachability remains untested.
- `INFO FOR DB` lists **47 tables**: typed knowledge and entity tables, relation tables, and the later Wayfinder tables. The populated entity tables are `agent` 3, `artifact` 12, `constraint` 4, `convention` 1, `decision` 26, `fact` 6, `lesson` 7, `preference` 2, `project` 1, `provider` 7, `repo` 2, `schema_proposal` 1, and `thought` 7. The populated relation tables are `about` 18, `in_project` 14, and `supersedes` 2. All other tables are empty, including `run`, `observation`, `person`, `evidenced_by`, `work_map`, `question`, and `map_apply`.
- Knowledge card tables have `title`, `body`, `status`, `scope`, `tags`, `confidence`, `authored_by`, `source_run`, `valid_from`, `valid_until`, and timestamps. `decision` also has `alternatives`; `thought` has `facet` and `about`; `lesson` has `kind`. Fields are schema-defined, with full-text title/body indexes, scope/status/tag indexes, and vector indexes. All **54 existing knowledge cards** have title, body, a 4,096-element embedding, `confidence = 0.6`, and `valid_from`; none has `valid_until`. Tag sets vary from empty to five entries. No card content was copied into this report.

## Lifecycle, scope, and provenance actually stored

| Type | Observed status counts |
| --- | --- |
| `decision` | proposed 15; accepted 10; superseded 1 |
| `constraint` | proposed 2; active 2 |
| `convention` | active 1 |
| `fact` | asserted 6 |
| `lesson` | active 7 |
| `preference` | active 2 |
| `schema_proposal` | proposed 1 |
| `thought` | inbox 7 |

- `scope` is a `set<string>`. Every existing knowledge card has one or two scope tags. Their observed prefixes are `project:` and, on some facts, `domain:`; suffixes were not recorded. Entity scope is also present on projects, repositories, artifacts, and agents, with some entities having an empty set. Existing tags alone do not prove the new glossary's work/personal, private, publication, or worktree access semantics.
- Across 54 cards there are **58 `project:` tag assignments with 8 distinct suffixes**. Anonymized bucket sizes are 17, 16, 9, 8, 5, 1, 1, and 1; four cards carry two project tags. Only the 16-card bucket matches the one extant `project` record ID and that record's own scope tag. The other 38 cards have no project tag resolving to an extant project record. Of the 16 matching cards, 9 have an `in_project` edge to that record and 7 do not. Five other `in_project` edges target absent project records, although each edge target matches a project tag on its source card. In total, 45 cards have no `in_project` edge to the extant project. These counts distinguish tag identity from materialized project records and graph links; the current schema does not by itself establish that every scope tag must have a project row.
- Every existing knowledge card has an `authored_by` reference of type `agent`. There are 3 agent records and 7 provider records. All 54 knowledge cards have **no `source_run`**, and the `run` table is empty. Thus author attribution exists, but durable source-run, source-installation, original-session, and evidence provenance required by the new design cannot be recovered from these fields alone.
- Existing links are `about`: thought→artifact 14, thought→schema_proposal 3, thought→thought 1; `in_project`: decision→project 9, constraint→project 1, fact→project 4; and `supersedes`: decision→decision 1, lesson→lesson 1. One supersession link has a reason. `evidenced_by` has no records. This is retained history, not a record of raw provider transcripts.
- Status constraints differ by type: decisions allow proposed/accepted/superseded/rejected; thoughts inbox/clustered/promoted/discarded; facts asserted/contradicted/expired; lessons active/superseded; maps open/closed; questions open. Existing status labels must keep their current meaning. The proposed new lifecycle may need a mapping rather than a blanket status rewrite.

## Current client convention and adapter result

- The nearby T3 server source at `C:/Users/Quinten/Documents/git/memorycode/apps/server/src/memory/MemoryConfig.ts`, `MemoryService.ts`, and `SurrealMemoryStore.ts` configures `SURREAL_URL`, `SURREAL_NS`, `SURREAL_DB`, credentials, and embedding width. It uses the SurrealDB SDK and `RecordId`, centralizes typed writes and relations behind MCP memory tools, and renders schema plus Wayfinder migration when constructing its store. `SurrealMemoryStore` uses `UPSERT`/`RELATE` and status transitions; consumers should not assume direct table writes are the public contract. The source's default embedding width is 4,096, matching the stored vectors. The exact revision running behind this thread's MCP adapter was not established.
- The harness `memory_bootstrap` call returned `{"error":"backend_unavailable"}` during this audit, while direct read-only database queries succeeded. `MemoryService.ts` returns a disabled store if no backend is configured or if connection/schema initialization fails; `SurrealMemoryStore.ts` also maps query failures to the same error. The response therefore **does not identify the underlying cause**. We did not restart or reconfigure the server, run its schema initializer, or mutate the database to diagnose it. A read-only adapter health/log check in the server's actual runtime environment is still needed before attributing the failure to configuration, network, schema, or query behavior.

## Consequences to decide

The existing instance holds meaningful semantic history and an active schema. A compatible design must preserve access to the 54 existing knowledge cards, distinguish existing `project:`/`domain:` tags from newly defined binding and privacy concepts, retain type-specific statuses and supersession edges, and mark absent source-run/evidence provenance as unknown rather than fabricating it. It must also account for current T3 clients and their startup schema behavior before choosing direct reuse, an additive adapter, or an in-place mapping. No compatibility write has yet been tested.
