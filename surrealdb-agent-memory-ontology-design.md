# SurrealDB Memory Ontology for a Multi-Provider Coding-Agent Harness

**Status:** Draft for review  
**Date:** 2026-09-12  
**Scope:** Shared semantic memory for every coding agent loaded into a T3 Code fork  
**Non-scope:** T3 UI rewrite, full codebase graph, multi-tenant SaaS

---

## 1. Problem

T3 Code is a control plane over foreign harnesses (Codex, Claude Code, Cursor, Grok Build, OpenCode, Antigravity, and others). Each provider has its own session memory. Nothing durable and typed is shared across providers, projects, or days.

The operator’s work spans software, hardware, verification, and ML. The knowledge that dies when a thread ends is not source code (git already has that). It is:

- why a design was chosen
- hard constraints (standards, pin limits, safety)
- conventions and operator preferences
- failures and their blast radius
- reusable procedures
- which component, tool, vendor, or datasheet a fact is about

This spec defines a SurrealDB ontology and the agent-facing contract so every harness process can read and write that knowledge.

---

## 2. Goals

1. One database of record for *knowledge about work*, accessible to every agent the harness launches.
2. Typed capture: decisions, constraints, conventions, lessons, incidents, skills, preferences, facts.
3. Graph traversal plus hybrid (FTS + vector) retrieval in a single query language.
4. Provenance and lifecycle on every durable record.
5. A small MCP tool surface. Agents never write SurrealQL.
6. Budgeted retrieval so bootstrap context stays small.
7. Safe writes: upsert by stable id, supersede instead of silent rewrite, proposed-until-confirmed for hard rules.

### 2.1 Non-goals (v1)

- Replacing T3’s SQLite event store, threads, or checkpoints.
- Ingesting a full Tree-sitter / call-graph index of every repo.
- Storing raw source files or full chat transcripts in SurrealDB.
- OWL/SHACL reasoning.
- Multi-user ACLs beyond `scope` + `sensitivity`.
- Per-project Surreal namespaces or databases.
- Letting agents invent new table or edge types. Unfitted ideas go to `thought`; new tables require a `schema_proposal` and a spec migrate.

---

## 3. Decisions locked in this spec

| ID | Decision | Rationale |
|---|---|---|
| D1 | Layered multi-model (episodic pointers → atoms → durable graph), not wiki-only and not ADR-only. | Matches both “what happened” and “what is true.” |
| D2 | Single NS/DB: `harness` / `memory`. Isolation is `scope`, not separate databases. | Lessons must travel across repos. |
| D3 | Closed node and edge vocabulary for *queryable* types. Anything that does not fit is a `thought` (inbox), not a new table. | Prevents ontology drift from LLM writers while refusing to drop novel ideas. |
| D11 | Capture is open; schema change is closed. Stacked similar thoughts become a `schema_proposal` plus optional reclassify onto an existing type. New tables ship only as a spec+migrate. | Lets the graph learn *content* continuously without letting agents mutate the ontology. |
| D4 | Git remains source of truth for code. Memory stores pointers (`repo`, path, commit, symbol name). | Avoids a stale second tree. |
| D5 | **L0 turns are not mirrored.** Durable memory points at T3 `thread_id` / `turn_id`. Optional L0 mirror is v2. | T3 already stores chat. Dual-write is a sync bug. |
| D6 | Agents speak MCP tools only. SurrealQL is an implementation detail of the memory server. | Providers differ; MCP is the common plug. |
| D7 | Constraints and preferences extracted from a single turn land as `proposed`. Promotion to `active` requires confirm or a second independent run. | Matches “no auto-apply.” |
| D8 | One embedder, one frozen dimension, configured at server boot. Missing embeddings are allowed (`option`). | HNSW cannot mix dimensions. |
| D9 | Record ids are slugs the writer controls (`decision:te0820-uio-not-mmap`). Upsert, do not append twins. | Identity is the product. |
| D10 | T3 integration is a sidecar MCP server + turn-end reactor. Do not fork provider adapters to embed memory. | Survives provider churn. |

---

## 4. Architecture

```
┌─────────────────────────────────────────────────────────────┐
│ T3 Code fork (control plane)                                │
│  threads, projects, checkpoints, provider subprocesses      │
│  SQLite event store unchanged                               │
└───────────────┬───────────────────────────┬─────────────────┘
                │ launches + MCP stdio      │ turn/checkpoint
                │ forwards .mcp.json        │ reactor hook
                ▼                           ▼
┌──────────────────────────┐    ┌────────────────────────────┐
│ Provider agent process   │    │ memory-extractor job       │
│ Claude / Codex / …       │    │ (same MCP or internal API) │
│ calls memory_* tools     │    │ distill L1 → L3            │
└────────────┬─────────────┘    └─────────────┬──────────────┘
             │ MCP                            │
             ▼                                ▼
        ┌─────────────────────────────────────────┐
        │ memory-server (this component)          │
        │ typed tools, embedder, promotion rules  │
        └─────────────────┬───────────────────────┘
                          ▼
        ┌─────────────────────────────────────────┐
        │ SurrealDB  NS harness  DB memory        │
        │ document + graph + FTS + HNSW           │
        └─────────────────────────────────────────┘
```

T3 remains system of record for conversation UI. SurrealDB is system of record for *semantic* memory.

A thin `harness/control` database may later hold agent/provider registry. v1 keeps `agent` / `provider` / `run` tables inside `memory` so the first schema is one file.

---

## 5. Layer model

| Layer | Stored in v1? | Contents | Retention |
|---|---|---|---|
| L0 Episodic | No (pointer only) | T3 thread/turn ids on `run` and `evidenced_by` | T3’s own policy |
| L1 Atomic | Yes | `observation`, `mention` | Compact after promotion; raw rows optional GC after 30 days |
| L2 Situational | Yes | `episode`, `incident` | Durable |
| L3 Semantic | Yes | decisions, constraints, conventions, lessons, skills, preferences, facts, topology | Durable; supersede, do not delete |

L1 exists so an extractor can write cheap structured notes without pretending they are accepted architecture. Promoter lifts a subset to L3.

---

## 6. Scope and identity

### 6.1 Scope tokens

Every L1–L3 record carries `scope: set<string>`. Tokens are `key:value` with a closed key set:

| Key | Example | Meaning |
|---|---|---|
| `project` | `project:te0820-hil` | T3 project slug |
| `repo` | `repo:owner/name` | GitHub-style repo id |
| `domain` | `domain:hw` \| `sw` \| `ml` \| `ops` \| `personal` | Coarse facet |
| `sensitivity` | `sensitivity:normal` \| `work` \| `private` | Retrieval filter |
| `_global` | `project:_global` | Operator-wide conventions and preferences |

A record may have several tokens. Retrieval ANDs caller filters against this set (requested project matches if `project:<slug>` OR `project:_global` is present, unless the caller sets `include_global=false`).

v1 does not implement Spectron-style OR-of-AND `scope_sets`. If leakage between `work` and `personal` becomes real, add `scope_sets` without changing node types.

### 6.2 Record identifiers

```
<table>:<kebab-slug>
```

Slugs are lowercase ASCII `[a-z0-9-]`, max 80 chars. The memory server rejects collision across *different* tables using the same slug only if a tool call omits the table prefix; tools always require `type` + `slug` and compose the id.

### 6.3 Common spine

All L2/L3 knowledge tables share this field set. Topology tables (`project`, `repo`, `component`, …) use the subset marked ●.

| Field | Type | ● | Notes |
|---|---|---|---|
| `title` | string | ● | Required, ≤ 200 chars |
| `body` | string | | Markdown, target ≤ 2000 chars. Longer content is an `artifact` pointer. |
| `status` | string | ● | Per-table enum, see §8 |
| `confidence` | float 0–1 | | Default 0.6 |
| `scope` | set\<string\> | ● | §6.1 |
| `tags` | set\<string\> | ● | Free, lowercase |
| `embedding` | option\<array\<float\>\> | | Dimension = server config |
| `created_at` | datetime READONLY | ● | `time::now()` |
| `updated_at` | datetime | ● | Server-maintained |
| `valid_from` | option\<datetime\> | | Inclusive |
| `valid_until` | option\<datetime\> | | Exclusive; null = current |
| `authored_by` | record\<agent \| person\> | | Required on L3 writes |
| `source_run` | option\<record\<run\>\> | | Provenance |

`DEFINE FIELD` assertions reject empty `title` and out-of-range `confidence`.

---

## 7. Node types

### 7.1 Actors and provenance

#### `provider`

Foreign harness T3 wraps.

- `slug` unique: `codex` `claude` `cursor` `grok` `opencode` `antigravity` `other`
- `title`

#### `agent`

Logical writer identity, not a provider process. One human operator may have many agents (`fpga-planner`, `verification-scribe`, `operator`).

- spine ● + `provider` option\<record\<provider\>\>
- `kind`: `human` \| `harness` \| `extractor` \| `promoter`

#### `run`

One T3 turn, extractor job, or manual MCP session that wrote memory.

- `provider` record\<provider\>
- `agent` record\<agent\>
- `project` option\<record\<project\>\>
- `t3_thread_id` option\<string\>
- `t3_turn_id` option\<string\>
- `started_at` `ended_at` option\<datetime\>
- `status`: `running` \| `ok` \| `error`

This is the L0 pointer. No turn text is stored.

### 7.2 Work topology

#### `project`

T3 workspace.

- `slug` (record id)
- `root_path` option\<string\> — local absolute path as last seen
- `default_repo` option\<record\<repo\>\>
- `title`, `status` (`active` \| `archived`), `scope`, `tags`

#### `repo`

- `full_name` string unique (`owner/name`)
- `default_branch` string default `"main"`
- `origin_url` option\<string\>

#### `component`

Module, crate, IP block, PCB, SoM, service, test fixture — anything that can be named and affected.

- spine ●
- `kind`: `software` \| `firmware` \| `fpga` \| `pcb` \| `mechanical` \| `service` \| `testbench` \| `other`
- `primary_repo` option\<record\<repo\>\>
- `primary_path` option\<string\> — path inside that repo

#### `interface`

API, protocol, pin bank, bus, schema.

- spine ●
- `kind`: `api` \| `abi` \| `bus` \| `pin` \| `schema` \| `cli` \| `rpc` \| `other`

#### `artifact`

Pointer to an external object. Not a blob store.

- `uri` string — `file://`, `https://`, `repo://owner/name@sha1/path`, `t3://thread/...`
- `media` option\<string\> mime or `"markdown"` `"pdf"` `"datasheet"`
- `content_hash` option\<string\>
- `title`, `scope`, `tags`

#### `symbol`

Sparse. Only symbols memory talks about.

- `qualified_name` string
- `language` option\<string\>
- `repo` option\<record\<repo\>\>
- `path` option\<string\>
- `title` (usually same as qualified_name)

### 7.3 Knowledge types (L3)

Status enums are closed. Writers cannot invent statuses.

#### `decision`

Architectural or implementation choice.

- spine
- `status`: `proposed` \| `accepted` \| `superseded` \| `rejected`
- `alternatives` option\<array\<string\>\> — names of options not chosen

#### `constraint`

Hard rule. Standards, safety, physical limits, compliance.

- spine
- `status`: `proposed` \| `active` \| `relaxed` \| `obsolete`
- `severity`: `blocker` \| `high` \| `medium`
- `authority` option\<string\> — e.g. `IEEE 1547 Table 23`, `UL1741`, `pin-count`

#### `convention`

Soft style. Team or operator taste that is not a physical law.

- spine
- `status`: `proposed` \| `active` \| `deprecated`

#### `lesson`

Distilled learning, including anti-patterns. Body states context → what happened → what to do instead.

- spine
- `status`: `active` \| `superseded`
- `kind`: `practice` \| `anti-pattern` \| `gotcha`

#### `incident`

A concrete failure with a time and a blast radius.

- spine
- `status`: `open` \| `mitigated` \| `closed`
- `detected_at` option\<datetime\>
- `resolved_at` option\<datetime\>

#### `skill`

Reusable procedure.

- spine
- `status`: `draft` \| `active` \| `retired`
- `trigger` string — when to use it
- `steps` array\<string\>
- `validate` option\<string\> — how to know it worked

#### `preference`

Operator taste. Scoped; `project:_global` is the default home.

- spine
- `status`: `proposed` \| `active` \| `withdrawn`

#### `fact`

Dated claim that can be wrong. Use for world-state (“Malanka app requires BYN prepaid”), not for decisions.

- spine
- `status`: `asserted` \| `contradicted` \| `expired`
- Default `valid_until` may be set by promoter (e.g. +90 days for volatile facts).

#### `concept`

Named idea used as a wiki node and as a tag target. Use when the thing is *stable enough to name* but is not a decision/constraint/lesson/etc.

- spine
- `status`: `active` \| `merged`

#### `thought`

Inbox for anything that does not fit a closed type yet: half-formed problems, questions, analogies, “we should think about X”, stray observations the extractor is unwilling to promote.

This is the open capture valve. Agents **must** write a `thought` rather than invent a table or overload `fact`.

- spine
- `status`: `inbox` \| `clustered` \| `promoted` \| `discarded`
- `facet` string — writer’s guess, free but lowercase kebab (`open-question`, `problem`, `analogy`, `risk`, `research-lead`, `smell`). Not a schema type.
- `about` option\<array\<string\>\> — record ids this is in the neighbourhood of, denormalized for recall; edges are authoritative

Default status: `inbox`. Not injected by `memory_bootstrap` unless the caller sets `include_inbox: true`. `memory_recall` includes thoughts when `types` is omitted or lists `thought`.

Promotion: `memory_reclassify` copies/moves a thought onto an existing closed type, sets status `promoted`, and writes `thought ->promoted_to-> <new record>`. The thought row stays as provenance.

#### `schema_proposal`

Deliberate request to extend the ontology after similar thoughts stack. Not applied automatically.

- spine
- `status`: `proposed` \| `accepted` \| `rejected` \| `shipped`
- `proposed_table` string — kebab name the operator would add
- `proposed_kind` `node` \| `edge`
- `justification` string — why closed types are insufficient
- `example_ids` array\<string\> — thought/concept ids that motivated it
- `suggested_fields` option\<array\<string\>\>

`accepted` means “we will spec and migrate.” `shipped` means the `DEFINE TABLE` exists and a backfill ran. Agents cannot `DEFINE` anything.

Cluster rule (promoter, not a trigger that invents types):

- If ≥ 5 live `thought` rows share the same `facet` **and** pairwise embedding cosine ≥ 0.82 (or the same `tagged` concept), promoter upserts one `schema_proposal:<facet>` with those ids in `example_ids`, status `proposed`.
- It also upserts a `concept:<facet>` and `tagged` edges so the cluster is already queryable as a concept before any schema change.
- It never creates a new table.

### 7.4 Directory types

#### `person`

- `title` (display name)
- `kind`: `operator` \| `collaborator` \| `vendor-contact` \| `other`
- `handles` option\<object\> FLEXIBLE

#### `vendor`

- `title`
- `url` option\<string\>

#### `tool`

Compilers, lab instruments, agent CLIs, embedders.

- `title`
- `kind`: `cli` \| `ide` \| `eda` \| `lab` \| `model` \| `service` \| `other`
- `current_version` option\<string\>

### 7.5 L1 / L2 working types

#### `observation`

Extractor output. Not shown in bootstrap unless promoted.

- `body` string
- `kind`: `note` \| `quote` \| `hypothesis`
- `scope`, `source_run`, `confidence`, timestamps

#### `mention`

Links an observation or run to a topology/knowledge node without promoting.

- Implemented as edge `mentions` (see §8). No extra table required if the edge carries `quote`. v1 uses the edge only.

#### `episode`

A bounded stretch of work (“TE0820 UIO bring-up, 2026-08”).

- spine
- `status`: `open` \| `closed`
- `started_at` `ended_at` option\<datetime\>

`incident` is L2 that can stand alone; `episode` groups work that may contain zero or more incidents.

---

## 8. Edge types

All edges are `DEFINE TABLE … TYPE RELATION SCHEMAFULL` with `in` / `out` union types as listed. Shared edge fields:

- `created_at` datetime READONLY
- `source_run` option\<record\<run\>\>
- `confidence` option\<float\>

| Table | IN | OUT | Extra fields | Cardinality notes |
|---|---|---|---|---|
| `in_project` | knowledge \| thought \| schema_proposal \| topology except project | `project` | | Required for L3 knowledge |
| `promoted_to` | `thought` | any L3 knowledge \| `concept` | `reason` option\<string\> | One current promotion per thought |
| `motivated` | `schema_proposal` | `thought` \| `concept` | | |
| `about` | `thought` \| `concept` | any node | | Soft topical link when no stronger verb fits |
| `in_repo` | `component` \| `artifact` \| `symbol` \| `interface` | `repo` | `path` option\<string\>, `commit` option\<string\> | |
| `implements` | `component` | `interface` | | |
| `depends_on` | `component` | `component` | `kind`: `build` \| `runtime` \| `test` \| `docs` | |
| `affects` | `decision` \| `constraint` \| `lesson` \| `incident` \| `convention` | `component` \| `interface` \| `project` | `severity`: `low` \| `medium` \| `high` | |
| `constrains` | `constraint` | `component` \| `interface` \| `project` \| `skill` | | |
| `supersedes` | same-type knowledge | same-type knowledge | `reason` string | OUT is the old record |
| `contradicts` | `fact` \| `lesson` \| `decision` | `fact` \| `decision` \| `constraint` | `reason` option\<string\> | |
| `learned_in` | `lesson` \| `decision` \| `constraint` \| `skill` | `episode` \| `incident` \| `run` | | |
| `evidenced_by` | any L3 knowledge | `artifact` \| `symbol` \| `run` | `quote` option\<string\>, `loc` option\<string\> | |
| `uses_tool` | `episode` \| `skill` \| `incident` \| `run` | `tool` | `version` option\<string\> | |
| `owned_by` | `component` \| `project` \| `artifact` | `person` | `role` option\<string\> | |
| `tagged` | any L3 | `concept` | | |
| `mentions` | `observation` \| `run` | any node | `quote` option\<string\> | L1 only |
| `occurred_in` | `incident` | `episode` \| `project` | | |

There is no `related_to`. If the writer cannot pick a verb, they `tagged` a `concept`.

Unique indexes:

- `(in, out)` unique on every edge table except `mentions` and `evidenced_by` (those may repeat with different quotes).
- `supersedes`: a record has at most one *current* outgoing `supersedes` to the record it replaces; incoming many is allowed (history chain).

---

## 9. Indexes and search

```surql
-- Applied on every table that has `body` + `title`
DEFINE ANALYZER memory_en TOKENIZERS blank, class
  FILTERS lowercase, snowball(english);

DEFINE INDEX idx_body_fts ON decision FIELDS body
  FULLTEXT ANALYZER memory_en BM25 HIGHLIGHTS;
DEFINE INDEX idx_title_fts ON decision FIELDS title
  FULLTEXT ANALYZER memory_en BM25;

DEFINE INDEX idx_embedding ON decision FIELDS embedding
  HNSW DIMENSION $EMBED_DIM DIST COSINE TYPE F32;

DEFINE INDEX idx_scope ON decision FIELDS scope;
DEFINE INDEX idx_status ON decision FIELDS status;
DEFINE INDEX idx_tags ON decision FIELDS tags;
```

Repeat FTS + HNSW + scope/status for: `constraint`, `convention`, `lesson`, `incident`, `skill`, `preference`, `fact`, `concept`, `thought`, `schema_proposal`, `component`, `interface`, `episode`, `observation`. Also index `thought.facet`.

`$EMBED_DIM` is not SurrealQL. The memory server renders the schema at boot from config (`EMBED_DIM`, default `1536` for a local embedder; override if the chosen model differs). Changing dimension requires rebuild of every HNSW index — treat as a migration.

Retrieval recipe (`memory_recall`):

1. Filter `status` to live values (accepted/active/asserted/open/draft-excluded unless asked).
2. Filter `scope` as in §6.1.
3. Run FTS and KNN in parallel (`search::rrf()` if available on the deployed Surreal version; otherwise merge in the server with Reciprocal Rank Fusion, k=60).
4. Take top `k` (default 8, max 20).
5. Expand **one hop** on `affects`, `constrains`, `in_project`, `supersedes` (incoming only, to surface “this was replaced by”), `tagged`, `about`, `promoted_to`.
6. Render cards. Hard cap 3 000 tokens for `recall`, 2 000 tokens for `bootstrap`.

Live status sets:

| Type | Live |
|---|---|
| decision | proposed, accepted |
| constraint | proposed, active |
| convention | proposed, active |
| lesson | active |
| incident | open, mitigated |
| skill | draft, active |
| preference | proposed, active |
| fact | asserted |
| concept | active |
| thought | inbox, clustered |
| schema_proposal | proposed, accepted |
| episode | open |
| project | active |

`memory_get` ignores live filters and returns the record as stored.

---

## 10. Write path and promotion rules

### 10.1 Tool write (`memory_remember`)

1. Validate `type` ∈ closed set.
2. Validate slug; compose id `type:slug`.
3. If id exists: patch `body`/`title`/`tags`/`scope` only when `status` is `proposed` or `draft`. If status is live (`accepted`/`active`/`asserted`) and the new body *changes meaning*, reject with `use memory_status to supersede`. Cosmetic edits (typos) allowed.
4. If id does not exist: insert with `status` from §10.3 defaults.
5. Apply declared `links[]` via the edge tables. Unknown verb → reject.
6. Enqueue embed job for `title + body`.
7. Return the record card.

### 10.2 Extractor write (reactor)

On T3 turn/checkpoint receipt (text lives in T3; reactor is given thread id + transcript by T3 or reads T3 userdata):

1. Create `run` pointing at `t3_thread_id` / `t3_turn_id`.
2. Extractor produces zero or more `observation` rows + `mentions` edges. Schema-constrained JSON. No L3 yet.
3. Promoter reads observations + nearby L3 (recall k=5 on the observation text) and may emit `memory_remember` calls. If nothing closed fits, it writes a `thought` with a `facet` guess rather than forcing a type.
4. Contradiction pass: if a new `fact`/`decision` has cosine ≥ 0.88 and different polarity vs an existing live record, create `contradicts` and leave both; do not auto-expire.
5. Cluster pass: apply the `schema_proposal` rule in §7.3. Never `DEFINE TABLE`.

### 10.3 Default statuses on create

| Type | Default status |
|---|---|
| decision | `proposed` |
| constraint | `proposed` |
| convention | `proposed` |
| preference | `proposed` |
| lesson | `active` |
| incident | `open` |
| skill | `draft` |
| fact | `asserted` |
| concept | `active` |
| thought | `inbox` |
| schema_proposal | `proposed` |
| episode | `open` |
| component / interface / … | `active` |

### 10.4 `memory_status`

Allowed transitions:

```
decision:    proposed → accepted | rejected
             accepted → superseded   (requires successor id)
             proposed → superseded   (allowed if successor exists)
constraint:  proposed → active | obsolete
             active → relaxed | obsolete
             relaxed → active | obsolete
convention:  proposed → active | deprecated
             active → deprecated
preference:  proposed → active | withdrawn
             active → withdrawn
lesson:      active → superseded     (requires successor)
skill:       draft → active | retired
             active → retired
fact:        asserted → contradicted | expired
incident:    open → mitigated → closed
             open → closed
episode:          open → closed
thought:          inbox → clustered | promoted | discarded
                  clustered → promoted | discarded | inbox
schema_proposal:  proposed → accepted | rejected
                  accepted → shipped
```

`superseded` always creates `from=successor ->supersedes-> old`. The old record’s status becomes `superseded`. Bootstrap and recall drop it from the live set.

### 10.7 Reclassify (`memory_reclassify`)

Moves inbox content onto the closed vocabulary without losing the original.

```
input: {
  from: string,                 // thought:…
  to_type: string,              // existing closed type, not a new name
  to_slug: string,
  title?: string,
  body?: string,                // default: copy thought.body
  extra?: object,
  links?: { verb, to, meta? }[],
  confirm?: boolean
}
```

Rules:

1. `from` must be a `thought` in `inbox` or `clustered`.
2. `to_type` must already exist in this spec. Reject `unknown_type` — that is what `schema_proposal` is for.
3. Upsert the target as `memory_remember` would. Constraint/preference confirm rule still applies.
4. RELATE `from ->promoted_to-> target`. Set thought status `promoted`.
5. Copy `scope`/`tags` if the caller omitted them.

Batch reorg (operator or a later maintenance job, not the extractor): given a `schema_proposal` that is `accepted`, a script maps each `example_id` through `memory_reclassify` onto the new table *after* that table has been migrated (`shipped`). Until then, the cluster is queryable as `concept:<facet>` plus `tagged` thoughts.

### 10.5 Confirm rule (D7)

`constraint` and `preference` may move `proposed → active` only if one of:

- tool call sets `confirm: true` (operator or an agent that was explicitly told to confirm), or
- a second `run` with a different `provider` record asserts the same slug.

Extractor/promoter never pass `confirm: true`.

### 10.6 Deletes

No user-facing delete in v1. Use status. A `REMOVE` is an operator maintenance action on the DB, not an MCP tool.

---

## 11. MCP tool contract

Server name: `harness-memory`. Five tools.

Shared card shape returned by recall/get/remember/status:

```json
{
  "id": "decision:te0820-uio-not-mmap",
  "type": "decision",
  "title": "…",
  "body": "…",
  "status": "accepted",
  "confidence": 0.9,
  "scope": ["project:te0820-hil", "domain:hw"],
  "tags": ["zynq", "uio"],
  "edges": [
    {"verb": "affects", "to": "component:uio_map", "meta": {"severity": "high"}}
  ],
  "evidence": [
    {"to": "artifact:docs/uio-map.md", "quote": null}
  ]
}
```

### 11.1 `memory_bootstrap`

```
input:  { project: string, max_tokens?: number = 2000, include_inbox?: boolean = false }
output: {
  project: card,
  constraints: card[],
  conventions: card[],
  preferences: card[],
  open_incidents: card[],
  recent_lessons: card[],   // valid_from last 90 days, max 5
  inbox?: card[]            // only if include_inbox; max 5 thoughts
}
```

Selection: live records whose scope contains `project:<slug>` or `project:_global`. Sort constraints by severity then confidence. Truncate to `max_tokens`.

Injected by T3 at thread start into the provider session (system fragment or skill file). Agents may also call it.

### 11.2 `memory_recall`

```
input: {
  query: string,
  project?: string,
  types?: string[],          // default = L3 knowledge + thought + concept
  k?: number,                // default 8, max 20
  include_proposed?: boolean // default false
}
output: { cards: card[], tokens_est: number }
```

### 11.3 `memory_get`

```
input:  { id: string }
output: { card: card } | { error: "not_found" }
```

Includes non-live records and both directions of `supersedes`.

### 11.4 `memory_remember`

```
input: {
  type: "decision"|"constraint"|"convention"|"lesson"|"incident"|"skill"|"preference"|"fact"|"concept"|"thought"|"schema_proposal"|"component"|"interface"|"episode"|"artifact"|"person"|"vendor"|"tool"|"project"|"repo"|"symbol",
  slug: string,
  title: string,
  body?: string,
  scope: string[],
  tags?: string[],
  extra?: object,            // type-specific fields; server validates
  links?: { verb: string, to: string, meta?: object }[],
  confirm?: boolean
}
output: { card: card } | { error: string, hint?: string }
```

`extra` maps to table-specific fields (`severity`, `steps`, `trigger`, `uri`, …). Unknown keys rejected.

### 11.5 `memory_link`

```
input: { from: string, verb: string, to: string, meta?: object }
output: { ok: true } | { error: string }
```

`verb` ∈ closed edge list. Server checks IN/OUT types.

### 11.6 `memory_status`

```
input: { id: string, status: string, successor?: string, reason?: string }
output: { card: card } | { error: string }
```

### 11.7 `memory_reclassify`

See §10.7. Same error vocabulary as `memory_remember`, plus `not_a_thought` and `already_promoted`.

### 11.8 Errors

Machine-readable `error` strings: `not_found`, `unknown_type`, `unknown_verb`, `invalid_slug`, `invalid_transition`, `successor_required`, `confirm_required`, `immutable_live_body`, `type_mismatch`, `scope_required`.

No raw Surreal errors leak to the model.

---

## 12. T3 fork integration

### 12.1 Process

`memory-server` is a long-lived process next to the T3 backend (same machine as `npx t3` / the forked server).

- Speaks MCP over stdio for provider sessions.
- Speaks a local HTTP or Unix socket to T3 reactors (same handlers as MCP tools, plus `ingest_turn`).

### 12.2 Wiring

1. Fork adds or extends `.mcp.json` so every provider adapter that already forwards MCP also forwards `harness-memory`.
2. A checked-in agent contract lives at `harness-memory/AGENTS.md` (copy into `.agents/skills/harness-memory/SKILL.md` or the fork root `AGENTS.md` as needed). It contains:
   - the closed type/verb list
   - “call `memory_bootstrap` at start of project work”
   - “call `memory_remember` when a decision, constraint, lesson, or incident is established”
   - “never invent types or verbs; if it does not fit, write a `thought` with a `facet`”
   - “call `memory_reclassify` only onto an existing type; stack similar thoughts via `schema_proposal`”
3. T3 reactor on turn end / checkpoint:
   - payload: `{ thread_id, turn_id, project_slug, transcript_ref }`
   - memory-server `ingest_turn` runs extractor+promoter asynchronously
   - failures emit a receipt; they do not block the T3 turn
4. Project slug mapping: T3 project directory basename or an explicit field in T3 project settings. First bootstrap for an unknown slug creates `project:<slug>` with `root_path`.

### 12.3 What not to touch

- Provider protocol adapters
- T3 event-sourcing decider / projector
- Checkpoint git refs
- Client UI, except an optional later “memory” pane that reads the same HTTP API

### 12.4 Config

Environment / config file on the memory server:

```
SURREAL_URL=
SURREAL_NS=harness
SURREAL_DB=memory
SURREAL_USER=
SURREAL_PASS=
EMBED_MODEL=          # local or API
EMBED_DIM=1536
T3_USERDATA=          # path to ~/.t3/userdata or worktree .t3, for ingest_turn
```

---

## 13. Schema file layout (implementation, not this spec’s job to execute)

When implemented, one SurrealQL file is source of truth:

```
schema/
  00_namespace.surql
  01_analyzers.surql
  10_actors.surql
  20_topology.surql
  30_knowledge.surql
  40_working.surql
  50_edges.surql
  60_indexes.surql
  70_functions.surql      # optional DEFINE FUNCTION wrappers
```

`DEFINE TABLE … SCHEMAFULL` everywhere except `person.handles` and `run` metadata object if needed (`FLEXIBLE` on that field only).

Example definitions (normative shape; full file is an implementation task):

```surql
DEFINE TABLE decision SCHEMAFULL;
DEFINE FIELD title ON decision TYPE string ASSERT string::len($value) > 0 AND string::len($value) <= 200;
DEFINE FIELD body ON decision TYPE string DEFAULT "";
DEFINE FIELD status ON decision TYPE string
  ASSERT $value IN ["proposed", "accepted", "superseded", "rejected"]
  DEFAULT "proposed";
DEFINE FIELD confidence ON decision TYPE float ASSERT $value >= 0 AND $value <= 1 DEFAULT 0.6;
DEFINE FIELD scope ON decision TYPE set<string> DEFAULT [];
DEFINE FIELD tags ON decision TYPE set<string> DEFAULT [];
DEFINE FIELD embedding ON decision TYPE option<array<float>>;
DEFINE FIELD alternatives ON decision TYPE option<array<string>>;
DEFINE FIELD created_at ON decision TYPE datetime DEFAULT time::now() READONLY;
DEFINE FIELD updated_at ON decision TYPE datetime DEFAULT time::now();
DEFINE FIELD valid_from ON decision TYPE option<datetime>;
DEFINE FIELD valid_until ON decision TYPE option<datetime>;
DEFINE FIELD authored_by ON decision TYPE record<agent | person>;
DEFINE FIELD source_run ON decision TYPE option<record<run>>;

DEFINE TABLE affects SCHEMAFULL TYPE RELATION
  IN decision | constraint | lesson | incident | convention
  OUT component | interface | project;
DEFINE FIELD severity ON affects TYPE string
  ASSERT $value IN ["low", "medium", "high"] DEFAULT "medium";
DEFINE FIELD created_at ON affects TYPE datetime DEFAULT time::now() READONLY;
DEFINE FIELD source_run ON affects TYPE option<record<run>>;
DEFINE FIELD confidence ON affects TYPE option<float>;
DEFINE INDEX affects_pair ON affects FIELDS in, out UNIQUE;
```

---

## 14. Worked example

Operator works on TE0820 HIL. An OpenCode turn settles the UIO question. Promoter writes:

```surql
CREATE decision:te0820-uio-not-mmap CONTENT {
  title: "Expose TE0820 PL registers via UIO + JSON map, not raw /dev/mem",
  body: "PS Linux userspace talks to PL through UIO. Pin map is versioned JSON. Avoids mmap footguns when agents drive HIL.",
  status: "proposed",
  confidence: 0.82,
  scope: ["project:te0820-hil", "domain:hw", "repo:example/te0820"],
  tags: ["zynq", "uio", "safety"],
  authored_by: agent:promoter,
  source_run: run:abc
};

RELATE decision:te0820-uio-not-mmap->affects->component:uio_map SET severity = "high";
RELATE decision:te0820-uio-not-mmap->constrains->interface:gpio-json;
RELATE decision:te0820-uio-not-mmap->evidenced_by->run:abc
  SET quote = "don't mmap /dev/mem from the agent";
```

Operator later: `memory_status({ id: "decision:te0820-uio-not-mmap", status: "accepted" })`.

A Claude Code thread next week on the same project calls `memory_bootstrap({ project: "te0820-hil" })` and receives this card plus active constraints (no `/dev/mem`, JSON map versioning) without rereading the old transcript.

---

## 15. Failure modes and mitigations

| Failure | Mitigation |
|---|---|
| Agents invent types | Server reject `unknown_type` / `unknown_verb`; skill file lists the closed set |
| Duplicate decisions | Slug upsert; recall-before-write in promoter; contradiction pass |
| Stale facts | `valid_until` + `expired`; no silent overwrite of live bodies |
| Context flood | Token caps on bootstrap/recall; live-status filter; one-hop only |
| Embedder down | Writes succeed with `embedding = NONE`; recall falls back to FTS |
| Surreal unreachable | Tools return `error: backend_unavailable`; T3 turn still completes |
| Transcript too large for extractor | Extractor sees tail N tokens + T3 summary if present; observations may be incomplete; promoter must not invent |
| Scope leak personal ↔ work | `sensitivity` filter on bootstrap default `normal,work`; `private` only when project scope says so |
| Provider does not forward MCP | Skill still documents manual fallback; reactor can still ingest after the fact |

---

## 16. Testing requirements

Implementation must cover, before calling v1 done:

1. Schema apply is idempotent (`DEFINE … OVERWRITE` or guarded migrate).
2. Every closed type can be created, fetched, and scoped-filtered.
3. Every legal status transition succeeds; illegal ones return `invalid_transition`.
4. Live-body mutation without supersede returns `immutable_live_body`.
5. Constraint promotion without `confirm` stays `proposed`.
6. `supersedes` hides the old record from bootstrap live sets and surfaces it on `memory_get`.
7. Recall with empty embedding index still returns FTS hits.
8. Bootstrap token estimate ≤ cap with a fixture of 50 constraints.
9. Unknown verb / type / slug characters rejected.
10. `in`/`out` type mismatch on `memory_link` rejected.

---

## 17. Implementation phases (for a later plan, not this spec)

Phase A — schema + memory-server + five tools against a local Surreal instance, no T3.  
Phase B — skill file + `.mcp.json` in the T3 fork; manual agent use.  
Phase C — `ingest_turn` reactor + extractor/promoter.  
Phase D — optional UI pane; optional L0 mirror; optional code-index sidecar.

Each phase is independently usable.

---

## 18. Out of scope follow-ups (explicitly parked)

- Full code knowledge graph (callers, blast radius).
- Multi-embedder / per-language vectors.
- Spectron-style hierarchical `scope_sets` and API keys.
- Automatic confidence decay job.
- Replication / remote Surreal for phone clients.
- Writing this ontology back into git as markdown wiki pages.

---

## 19. Review checklist

- [x] Closed type list enumerated
- [x] Inbox `thought` + `schema_proposal` reorg path specified
- [x] Closed edge list enumerated
- [x] L0 decision locked (pointers only)
- [x] MCP tools named and shaped
- [x] Promotion and confirm rules specified
- [x] T3 integration is sidecar + reactor, not adapter rewrite
- [x] Retrieval budget specified
- [x] No placeholder “TBD” requirements

---

## 20. Open questions that do not block v1

These can be answered during implementation without schema redesign:

1. Embedder choice (local on RTX 3090 vs API) — only `EMBED_DIM` and boot config.
2. Whether T3 userdata is readable enough for `ingest_turn` or the reactor must pass the transcript inline.
3. Exact skill-file path per provider in the current T3 fork.
4. Whether `project` slug is directory basename or an explicit T3 setting.

Anything that would add a table, an edge verb, or a tool is a spec change, not an implementation detail.
