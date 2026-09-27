# Harness-memory save status

The local tracker and research reports are saved. Cross-thread harness-memory persistence is pending.

On 2026-09-26, `memory_map_get` and `memory_map_apply` returned `backend_unavailable` after resolving [Define memory scope and provenance identities](../tickets/04-scope-and-provenance-identity.md). A new `memory_map_apply` attempt after resolving [Set the knowledge lifecycle and authority rules](../tickets/05-knowledge-lifecycle-contract.md) also returned `backend_unavailable`. The current full payload in [memory-map-save.json](./memory-map-save.json) includes every ticket's full question and body, the confirmed lifecycle resolution, and the other local ticket context. Both decisions are resolved and indexed in the map. No revision was returned, so the skill's memory-save completion condition has not been met.

Retry the exact payload with its existing idempotency key when the backend is available. If membership or wording changes, use a new key. On `revision_conflict`, retrieve the current map revision and reconcile before applying. Do not claim this JSON file is itself a successful harness-memory save.

After resolving [Define compatibility with existing SurrealDB history](../tickets/13-existing-surrealdb-history.md), the same payload was retried because map membership and question wording had not changed. `memory_map_apply` again returned `backend_unavailable`. The resolution is recorded in the local ticket and indexed in the local map; no harness-memory revision has been returned.

During [Choose the memory runtime and agent tool boundary](../tickets/06-runtime-and-tool-boundary.md), `memory_bootstrap` and `memory_map_get` again returned `backend_unavailable`. That ticket changed only its answer and the map's decision index, not the map's destination, notes, question membership/wording, fog, or exclusions, so no new `memory_map_apply` payload was needed. The local tracker remains canonical and cross-thread harness-memory persistence is still pending.
