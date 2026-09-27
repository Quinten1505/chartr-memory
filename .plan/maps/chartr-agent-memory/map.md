# Shared agent memory for Chartr

## Destination

An implementation-ready design for shared semantic agent memory in macOS/Linux Chartr, with explicit memory access followed by automatic capture for Codex, Claude Code, and Grok Build. Serve one operator across projects and worktrees using the same existing Windows-hosted SurrealDB instance and its stored history, with minimal setup, health, and confirmation controls; completion means all material contracts and acceptance criteria are settled.

## Notes

- Source baseline: [SurrealDB memory ontology draft](../../../surrealdb-agent-memory-ontology-design.md). Its locked decisions are a baseline, not immutable: each reopening must name the decision and reason.
- Scope and rollout are recorded in [Choose the first-version provider and rollout contract](./tickets/03-first-version-support-contract.md). Its resolution supersedes the initial single-machine assumption and clarifies existing database history versus separate provider-log import.
- Use [the shared-memory glossary](../../../CONTEXT.md) for established terminology.
- This map plans the work. Do not implement the production destination or silently resolve human decisions. Every session consults wayfinder, grilling, and domain-modeling; research and prototype tickets additionally use their matching skills.
- Follow [the local tracker contract](../../../plugins/wayfinder/TRACKER-CONVENTION.md). The directory is the parent map; ticket type represents its wayfinder label. Answers live only in tickets and are indexed here.
- Chartr owns terminal sessions and a conversation identity/archive index. Provider stores own transcripts. T3 thread IDs, turn events, MCP forwarding, and a turn-end reactor must not be assumed to exist.
- Keep established terms in a glossary when resolved; do not turn a glossary into the implementation specification. Keep all baseline changes and reasons in the responsible decision ticket.
- Research reports belong in docs/research and are captured on research branches; ticket asset pointers identify their reports. During this charting session research assignees are recorded explicitly; no Chartr terminal claim identity will be fabricated.
- Harness memory bootstrap returned backend_unavailable during charting. The local tracker remains canonical; the full save payload is retained under assets for retry if the memory service remains unavailable.

## Decisions so far

- [Establish provider memory and capture capabilities](./tickets/01-provider-memory-capabilities.md): Representative providers expose relevant MCP and capture interfaces, but Chartr needs explicit identity, configuration, and durable capture contracts.
- [Verify SurrealDB support for the proposed memory model](./tickets/02-surrealdb-schema-capabilities.md): The principal capabilities are documented; schema invariants, query plans, and restore behavior require explicit contracts and validation.
- [Verify Grok memory and capture interfaces](./tickets/11-grok-memory-capabilities.md): Official MCP, hook, identity, and export interfaces provide candidate seams; installed-release behavior and capture durability remain to be verified.
- [Choose the first-version provider and rollout contract](./tickets/03-first-version-support-contract.md): Both stages require Codex, Claude Code, and Grok on macOS/Linux, using the existing Windows-hosted database and its history with explicit compatibility and continuity checks.
- [Define memory scope and provenance identities](./tickets/04-scope-and-provenance-identity.md): Stable cross-machine identities, operator-bound work/personal access, private opt-in, reviewed sharing snapshots, and durable provider-neutral attribution define the scope/filter contract.
- [Set the knowledge lifecycle and authority rules](./tickets/05-knowledge-lifecycle-contract.md): Operator acceptance governs instructions and decisions; attributed facts and lessons can surface, while scoped revisions, supersession, evidence, and publication follow explicit conflict and audit rules.
- [Define compatibility with existing SurrealDB history](./tickets/13-existing-surrealdb-history.md): Operator-reviewed legacy scope mapping and an additive adapter preserve stored history, trusted cutover statuses, and current-client coexistence.
- [Choose the memory runtime and agent tool boundary](./tickets/06-runtime-and-tool-boundary.md): An independent Windows-hosted service serves seven scoped MCP tools through per-session bridges, with Chartr-controlled authority and visible degraded behavior.

## Not yet specified

- **Provider-specific exceptions.** Capability research and the selected support matrix may expose incompatible identity, capture, or MCP behaviors. The concrete exceptions needing their own decisions are not yet known; revisit after provider support and capture are chosen.

- **Quality and performance experiments.** The retrieval and extraction contracts may need empirical calibration against representative operator work. Workloads, failure thresholds, and the experiments that could change a decision are not yet established.

- **Cross-contract gaps.** Reconcile the final ontology, capture, runtime, confirmation, and recovery choices. New questions may emerge from their interaction; graduate only those that can be stated precisely.

## Out of scope

- [Define historical session import semantics](./tickets/12-historical-session-import.md): Separate provider-log import was not intended; historical access means reusing the existing database and its stored history.

- **Production implementation.** Building and shipping the memory service, database migrations, provider wiring, and UI is outside this planning map. The agreed destination is an implementation-ready design; throwaway research and prototypes only support decisions.

- **Multiple operators and replicated memory.** Team ACLs, multi-tenant hosting, replicated databases, and mobile synchronization are excluded. One operator's macOS/Linux clients connecting to the same existing Windows-hosted database are in scope; replication is not required.

- **Native Windows support.** Deferred from this release; the operator selected macOS and Linux for the first-version memory contract.

- **Full memory browsing and editing.** A general memory explorer, graph editor, and comprehensive editing UI are excluded. The user selected minimal setup, health, and confirmation controls.

- **Replacing source and conversation stores.** Replacing git or provider transcript stores, rewriting Chartr's terminal UI, storing raw source/full transcripts in SurrealDB, and building a full code graph are excluded. These are baseline non-goals and are unnecessary for shared semantic memory.

- **Automatic ontology expansion and formal reasoning.** Agent-created tables/edge types, OWL/SHACL reasoning, and multi-embedder indexing are excluded. Preserves the draft's bounded first-version model; reopening requires a recorded scope decision.
