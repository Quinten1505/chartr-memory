---
type: grilling
claimed_by: 01a0de85-8bf0-78b2-87e8-13ce1827d2c3
claimed_at: 2026-09-26T18:22:18.8750236+02:00
claim_kind: codex_session
blocked_by: [04]
undermined_by: []
assets: [knowledge-lifecycle-discussion.md]
---
# Set the knowledge lifecycle and authority rules

## Question

Which parts of the draft's closed ontology, default statuses, immutable-live-body rule, supersession, reclassification, and confirmation policy should become the Chartr contract? Resolve the tension between explicit human confirmation and promotion after a second provider asserts a slug; distinguish independent evidence from an agent repeating retrieved memory. Decide deterministic update/idempotency behavior, concurrent writes, stale facts, and entity/edge invariants. Audit the draft's conflicting live-status descriptions, unsupported supersession cases, and example edges. Record a reason for each reopened baseline decision.

Apply [Define memory scope and provenance identities](04-scope-and-provenance-identity.md), including its scoped-identity and memory-publication contract. Set publication status, update, and withdrawal mechanics without reopening the agreed visibility rules.

Options for discussion (not decisions):

- Require explicit human confirmation for hard rules; agents may propose and attach evidence
- Allow narrowly defined independent corroboration to activate hard rules
- Retain the draft's second-provider activation rule with explicit safeguards

## Done when

The human agrees to a complete lifecycle/authority matrix, update and conflict invariants, closed vocabulary corrections, and examples that distinguish capture from accepted knowledge.

## Answer

Confirmed by the operator on 2026-09-26. [Discussion history](../assets/knowledge-lifecycle-discussion.md) records the choices and baseline changes.

### Authority and visibility

| Knowledge | Initial state | Ordinary agent access | Transition to instruction authority |
| --- | --- | --- | --- |
| Decision | `proposed` | Excluded | Operator accepts it. |
| Constraint and operator preference | `proposed` | Excluded | Operator explicitly confirms it; another provider's assertion only adds evidence. |
| Convention | `proposed` | Excluded | Operator activates it. |
| Skill | `draft` | Excluded from ordinary instructions | Operator activates it. |
| Fact | `asserted` | Eligible with source and validity labels | No rule authority; a fact cannot masquerade as a constraint. |
| Lesson | `active` | Eligible with source and lesson label | Advisory learning, not a hard rule. |
| Incident | `open` | Eligible with source and incident state | `mitigated` or `closed` requires observed evidence or operator confirmation. |
| Concept | `active` | Eligible as a named idea/tag | No rule authority. |
| Thought | `inbox` | Excluded by default | Reclassification creates a typed target under that type's normal authority. |
| Schema proposal | `proposed` | Excluded by default | Operator accepts the proposal; `shipped` additionally requires evidence of a migrated schema. |
| Episode and topology | `open` for an episode; trusted registration for identities | Scope-filtered according to type | Episode status follows source work; identity registration follows the trusted binding contract. |

Ordinary bootstrap and recall exclude `proposed`, `draft`, `inbox`, rejected, withdrawn, expired, and superseded records. An explicitly requested proposal/history view may return them, always labeled and scoped. Direct record lookup does not bypass those access checks. Numeric confidence, another provider, and repeated exposure never confer acceptance. The extraction path may produce observations, facts, lessons, thoughts, and proposals, but capture itself never accepts a decision or activates an instruction.

### Record identity, revision, and replacement

A knowledge key is its trusted home scope plus closed type and writer-controlled slug. The home scope includes the work/personal boundary and the applicable memory project or Free-session identity, with repository/worktree qualifications when the claim is that narrow. A slug or path alone cannot merge records across scopes. A destination publication is a separate scoped record, not a second home for its source.

Every semantic mutation has a monotonic server revision and audit entry with actor/source run, time, previous revision, and reason when meaning or authority changes. Proposed/draft text may be intentionally amended through a revision-checked edit with the same audit trail; a conflicting `remember` call does not count as such an edit. On accepted or otherwise active knowledge, a strictly cosmetic typo correction may revise in place; changing meaning, scope, or authority creates a new key and a `supersedes` lineage edge. The old record becomes terminal `superseded` and leaves ordinary retrieval. Add `superseded` to every replaceable semantic type, including decision, constraint, convention, lesson, incident, skill, preference, fact, concept, and episode; keep their other type-specific states. Rejected and withdrawn records remain historical. An old record has at most one direct successor. A successor may replace several old records. A split marks the old item `superseded` with a split reason and links the new items as related outcomes without claiming a single direct successor.

The edge direction remains `successor ->supersedes-> predecessor`. The invariant is at most one incoming `supersedes` edge per predecessor, not one outgoing edge per successor as the draft says. A `supersedes` edge requires compatible type and authorized scope, an existing predecessor, an existing successor, and a terminal predecessor state; no cycles. The new record, old status, and lineage edge commit together. A split has no `supersedes` edge; its reason and related-outcome links commit with the terminal status instead.

An identical retried operation on a scoped key returns its prior result. “Identical” means the same canonical operation and input fields, not a model's judgment that two texts mean the same thing. Independently observed evidence is attached as a separate, provenance-bearing assertion. A different body, scope, status intent, or authority on the same key returns a conflict requiring an explicit successor or a new slug. Every body, status, scope, or successor mutation checks an expected server revision; a losing concurrent writer receives a revision conflict and must re-read before deciding. Status and edge updates are atomic. Idempotency keys protect delivery retries, while source-run attribution and capture fingerprints are specified by the capture ticket.

### Evidence, validity, and thought promotion

An evidence assertion records whether its source run independently observed the claim or derived it from retrieved memory. Different providers alone do not establish independence. Derived repetitions may be retained as provenance but do not count as corroboration. They never activate a hard rule. Evidence pointers remain access-controlled and may report the original source as unavailable without changing an eligible record's status.

Facts have inclusive `valid_from` and exclusive `valid_until` only when supported by the source or an explicit operator decision. There is no invented 90-day default. An expired fact leaves ordinary retrieval and remains in history. Conflicting current facts retain both claims, carry a contradiction link and visible conflict label, and are presented together for review when retrieved; a newer timestamp or higher confidence does not silently choose a winner. A source correction uses supersession, not an unrecorded overwrite.

Reclassifying a thought keeps the original as provenance and creates or identifies one current typed target. The target receives its type's default state and acceptance policy. Promotion does not widen the thought's trusted scope without operator review, and an existing same-key target still follows the conflict rule. The thought's `promoted` state and `promoted_to` edge commit with the target. A failed transaction leaves the thought unpromoted. Reclassification does not invent a new node or edge type.

### Publication and graph invariants

A publication starts as a draft destination-scoped snapshot and becomes active only after the operator reviews its visible content and destination. Source edits never update it implicitly. A changed snapshot or destination requires a new reviewed version; activation supersedes the previous published version atomically. Withdrawal changes an active publication to terminal `withdrawn`, stopping future retrieval while preserving audit history. An internal origin link does not grant access to the source record, source title, evidence, or neighbors. Publication cannot bypass work/personal, private, repository, or worktree binding rules.

Node and edge types remain closed. Unfitted content becomes a `thought`; an agent may suggest a schema proposal, but only a deliberate schema decision can accept it and only a migration can ship it. The draft's automatic “five similar thoughts” proposal trigger is removed. Every graph edge must have existing endpoints, a permitted typed direction, and a scope check on creation and traversal. Relations do not confer access to a restricted endpoint. Duplicate edge operations are idempotent where the relation is unique; repeated evidence assertions retain their own provenance. The draft's `decision ->constrains-> interface` example is invalid for its declared edge types: link the decision with `affects`, or create an actual constraint before using `constrains`.

### Baseline corrections and acceptance examples

- Draft D3 and the closed vocabulary remain. D11's agent-created schema-proposal similarity trigger is dropped because it can manufacture taxonomy pressure from repeated or derived content; a human schema decision remains required.
- Draft D7's second-provider activation path is removed because provider diversity does not establish authority or independent evidence. Explicit operator confirmation controls constraints and preferences. Convention and skill activation follows the same instruction-authority principle.
- Draft D9's writer-controlled slug remains, but its identity is qualified by trusted scope; otherwise identical slugs across projects or boundaries collide. Its upsert rule becomes exact idempotent retry plus explicit conflict and supersession.
- Draft section 9's proposed-as-live list is corrected to exclude proposals from ordinary retrieval, matching the selected visibility rule. Draft sections 10.4 and 8 gain cross-type supersession states and the corrected edge cardinality. Draft section 10.1's cosmetic-edit exception becomes an audited revision with a narrow definition.
- Agent A captures “never use raw `/dev/mem`” as a proposed constraint. Agent B reads and repeats it: the repetition is derived evidence; neither agent activates it. It enters normal bootstrap only after operator confirmation.
- Two agents independently record the same dated fact: the first fact stays asserted, and the second source is independent evidence. If they assert conflicting current values, recall shows both with a conflict marker. A source-grounded expiry removes an old value from ordinary recall without deleting it.
- An accepted decision changes scope from one worktree to its memory project: create a reviewed successor with a new key, atomically supersede the old decision, and do not let the old record's evidence or links widen access by inheritance.
- Two writers attempt different successors of the same decision: one atomic revision-checked transition wins; the other receives a revision conflict. An exact replay of the winner returns its recorded result.
