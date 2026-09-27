---
type: grilling
blocked_by: [03, 04, 05]
undermined_by: []
assets: [existing-instance-context.md, existing-surrealdb-readonly-audit.md, existing-history-discussion.md]
claimed_by: 01a0de99-4232-7d93-b77e-ef197a66014f
claimed_at: 2026-09-26T16:43:56Z
---
# Define compatibility with existing SurrealDB history

## Question

How should Chartr use the operator's already-running SurrealDB instance and make its stored history available without losing provenance, changing existing meanings, or disrupting current consumers? First establish read-only evidence of the actual database/namespace, schema, record lifecycle and scope conventions, and what stored history means. Then decide compatible read/write boundaries, identity and scope mapping, representation of records outside the draft's vocabulary, coexistence with current clients, and whether any explicit migration or adapter is necessary. Reusing the existing instance/history is a requirement; provisioning an unrelated fresh database is not an equivalent outcome. Distinguish database history access from importing older provider transcript files.

Options for discussion (not decisions):

- Reuse existing schema/contracts directly wherever compatible, with narrow adapters for differences.
- Maintain an explicitly mapped Chartr representation in the existing instance while retaining access to original history and current consumers.

## Done when

The human agrees to a compatibility and coexistence contract grounded in the actual existing data model, with explicit history-access acceptance examples and any migration prerequisites separately ticketed. Do not treat a failed harness-memory adapter as proof that SurrealDB is unavailable; do not guess schema or alter the database during discovery.

## Answer

Confirmed by the operator on 2026-09-26. The [read-only audit](../assets/existing-surrealdb-readonly-audit.md) is the evidence; [discussion history](../assets/existing-history-discussion.md) records the choices. No database, service, or credential change was made in this session.

### Existing-history baseline

The existing Windows SurrealDB 3.2.4 instance at `harness/memory` is reachable from this Windows shell. It contains 47 tables and 54 semantic knowledge cards with typed statuses, embeddings, and some graph links. Those cards use eight distinct `project:` scope tags; only one tag corresponds to the one stored project record. Four cards carry two project tags. All cards have an `agent` author, but no `source_run`; the run and evidence-link tables are empty. The harness memory adapter's `backend_unavailable` response has an unproven cause and is not evidence that the database or history is absent. Remote macOS/Linux reachability remains for the runtime decision.

### Identity, scope, and access

- Treat the existing `project:` values as **legacy project tags**, not stable memory-project identities or proof of work/personal or private authorization. An operator-reviewed mapping links each tag to a stable memory project, work/personal boundary, and private classification where applicable. Several legacy tags may map to one memory project. A missing project row or `in_project` edge does not silently invalidate a tag or create a project identity. The `domain:` values remain coarse descriptive facets; they do not grant access.
- The trusted memory service owns and enforces the mapping on every access path, including bootstrap, recall, direct ID lookup, graph expansion, and evidence lookup. Agents cannot choose or widen it. A tag or individual card whose boundary or sensitivity cannot be classified remains outside ordinary agent retrieval and is available only in an operator-restricted review view. Per-card review may narrow a tag-level classification; uncertainty never defaults to global, work, personal, or non-private access.
- Each of the four cards with two project tags requires individual review. The operator selects one home project and may approve a separate publication snapshot into another destination under the already selected publication rules. Until reviewed, these cards remain outside ordinary retrieval. Preserve their original rows and both original tags as history; the adapter governs new visibility.
- Preserve the old record IDs, bodies, types, edges, and status meanings. Existing statuses remain as stored unless a later guarded lifecycle transition is agreed under the correction rule below. Chartr reads old records through an explicit compatibility adapter in the **same existing instance**, alongside newly written memory. The adapter provides a collision-safe identity bridge and applies the new binding filters without rewriting legacy scope strings. Separate provider-transcript import is not part of this contract.

### Lifecycle, provenance, and coexistence

- Once a legacy card is mapped and eligible under its binding, trust its existing `accepted` or `active` status according to its type. Accepted decisions and active constraints, conventions, and preferences retain their instruction authority. Existing proposed, draft, inbox, superseded, withdrawn, expired, and other non-live states stay out of ordinary retrieval; scoped proposal/history access can show them with their original labels. Facts and lessons follow the selected lifecycle's advisory and validity rules. Do not promote a card based on its numeric confidence or an inferred source run.
- Grandfather authority only for a **cutover manifest** of original record IDs and content/status/scope fingerprints. The audit observed 54 cards, but the manifest is generated from the actual state at cutover. A new legacy write or semantic edit by the existing T3 client after cutover requires review before Chartr grants it accepted/active authority, even when its tag is already mapped. A changed fingerprint also suspends any grandfathered authority until reviewed.
- Expose eligible legacy cards with their original type, status, known agent author, and an explicit `source details unavailable` marker where source run, source installation, native session, or evidence are absent. Do not invent provenance or hide an otherwise eligible card solely for missing source detail. Preserve old supersession and other links, subject to the destination's access checks.
- Existing T3 clients keep their current record meanings and may continue to use the database. New identity/provenance data and Chartr writes use additive, versioned schema changes and a compatibility adapter; no blanket rewrite of legacy rows or statuses. Before rollout, test the exact running client revision, its startup schema initialization, and the additive migration against an isolated snapshot of the existing database. A failed compatibility check blocks rollout, not access to the original history through its existing client.
- When Chartr semantically replaces a legacy card, create a new scoped successor and use a guarded, atomic transition of the old card to a **semantically correct status supported by its existing type**, so old clients also stop treating it as live. Verify the original fingerprint/revision and use an idempotent operation with audit and lineage. If the type lacks a compatible terminal transition, do not claim cross-client correction or silently hide it only in Chartr: defer that replacement until a versioned, old-client-compatible migration is available. The lifecycle ticket's successor cardinality, scope, and authority checks still apply.

### History-access acceptance examples

| Case | Expected result |
| --- | --- |
| A mapped, grandfathered accepted decision requested by a fresh authorized session on another provider | Returned as accepted with the legacy author and missing-source-details marker. |
| A mapped active constraint or preference in the authorized boundary | Available as an instruction under its original active status. |
| An unmapped tag, unknown privacy, or a two-project card awaiting individual review | Absent from ordinary retrieval, direct lookup, and graph expansion; visible only to operator review. |
| A mapped proposed decision or inbox thought | Absent from ordinary retrieval; available in a scoped explicit proposal/history view with its original status. |
| An unrelated project, wrong work/personal boundary, or private access disabled | Excluded even when the record ID is known. |
| A new accepted record written by the old T3 client after cutover | Held for review in Chartr; a pre-cutover card whose fingerprint changed likewise loses grandfathered authority. |
| A corrected legacy decision with a compatible terminal transition | Old and new clients stop serving the old decision as live; the new scoped successor carries lineage and no wider inherited access. |

The runtime ticket owns the remote service and authentication boundary, including the unresolved adapter-health cause. Retrieval owns ranking and embedding policy for the existing 4,096-dimensional vectors. The operational handoff must retain the cutover inventory, operator mapping/review workflow, isolated-snapshot compatibility gate, and an explicit count of unmapped or inaccessible history; it cannot report complete history access while those remain unresolved.
