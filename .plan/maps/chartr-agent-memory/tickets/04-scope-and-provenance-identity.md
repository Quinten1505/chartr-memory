---
type: grilling
assignee: Quinten1505
claim_thread: 01a0de71-d384-7bc1-9fa1-2eefeb4bb5d7
claimed_at: 2026-09-26T16:00:44.8364478+00:00
blocked_by: []
undermined_by: []
assets: [scope-identity-discussion.md]
---
# Define memory scope and provenance identities

## Question

How should one operator's memory identify projects, repositories, worktrees, folder spaces, Free sessions, agents, and source runs across macOS/Linux clients using the existing Windows-hosted database? Decide the sharing and filtering rules across renamed directories, identical basenames, multiple clones, worktree-specific facts, global preferences, and private/work knowledge. Replace the draft's T3 thread/turn assumptions with explicit provenance requirements; preserve original-provider evidence pointers even when Chartr closes. Define identity collision handling and which trusted component supplies scope.

Options for discussion (not decisions):

- Repository identity with explicit workspace/worktree overrides and operator-global scope
- Chartr folder-space identity with explicit links between spaces
- An explicit memory-project identity independent of folders and repositories

## Done when

Canonical terms, stable identifiers, provenance fields, and a worked scope/filter matrix cover same-basename repos, clones, worktrees, Free sessions, and global/private records.

## Answer

Resolved with the operator on 2026-09-26. The operator selected the recommended choices in all three rounds and confirmed the final proposed contract. [Discussion history](../assets/scope-identity-discussion.md) retains the questions and alternatives; this Answer is the canonical resolution.

### Stable identity and project membership

A memory project is independent of a directory, Chartr folder space, repository remote, and provider. It can explicitly contain several repositories and non-git folders. Its knowledge is shared across the operator's macOS/Linux clients using the existing Windows-hosted database, subject to the binding and restrictions below.

| Identity | Contract |
| --- | --- |
| Memory project | Stable opaque identity; title, slug, and directory basename are not identity. |
| Repository | Stable opaque identity registered with repository aliases. Normalized remotes help identify a registered repository; they are not its sole durable identity. |
| Client installation | Stable opaque identity distinguishing source installations even when paths and native provider IDs coincide. Duplicate installation registrations require reconciliation, not silent merging. |
| Checkout/worktree | Stable opaque identity for one registered checkout. New clones/worktrees receive distinct identities and may share the same repository/project. |
| Folder space | Chartr's UI ownership/location concept, explicitly linked to memory identity; it does not define memory identity. |
| Conversation | Stable opaque identity linked to installation-qualified provider/native-session evidence when verified. A provisional local conversation is allowed before verification. |
| Source run | Stable service-assigned identity attributing a memory write or capture unit to its originating work. Capture-unit granularity and replay rules belong to the capture contract. |
| Agent | Provider plus conversation/source-run context, with role and model when known. A model name or transient terminal ID does not identify durable source work. |

An unambiguous previously registered repository match automatically selects its memory project only within the operator-selected work/personal boundary. Multiple candidate projects, conflicting aliases, forks, and ambiguous matches require explicit linking. Basenames and paths alone never merge projects. A verified move/rename preserves registration and identity; if continuity cannot be verified, ask to relink. Identical knowledge slugs in different projects or boundaries remain distinct. Within-scope write conflicts and exact record keys belong to the lifecycle/tool contracts.

Project-wide knowledge is shared across linked repositories and worktrees. Repository-qualified knowledge remains qualified to its repository; worktree-specific facts remain local to that checkout unless deliberately shared or generalized. Non-git folders can be linked explicitly without inventing a repository identity.

### Memory binding and authority

The operator selects a session's memory binding: work/personal boundary, optional memory project, applicable repository and checkout/worktree, conversation, and private-access permission. Chartr supplies this binding through trusted integration; the memory service enforces it on reads, writes, bootstrap, search, direct record lookup, graph traversal, and evidence access. Agent-supplied parameters may narrow access but cannot authorize another project or boundary. Authentication and transport mechanisms belong to the runtime ticket.

Work and personal knowledge are separate. Eligible operator-wide preferences accompany project knowledge by default within their allowed boundary; the caller may exclude them. A preference explicitly marked safe for both can be available in both. Global scope never bypasses private, repository, worktree, or session restrictions.

Private is an additional opt-in restriction within either work or personal, not a synonym for personal. Private content is excluded on every access path until the operator enables private access for the binding. Unknown/unmapped scope or sensitivity is not inferred to be global or normal. Existing-history mappings remain owned by the compatibility ticket.

A Free session has no memory project. After selecting its work/personal boundary it receives eligible global preferences, while other new knowledge stays local to that conversation until explicitly assigned or published. Its cwd does not supply a project implicitly.

Changing cwd or moving a UI pane never changes memory authority. Switching memory project or work/personal boundary starts a fresh provider conversation: already-retrieved context cannot be removed by changing a filter. Resuming an existing conversation preserves its binding. Assigning selected Free-session knowledge to a project does not reclassify the conversation's other contents.

### Explicit sharing

Sharing publishes an operator-reviewed snapshot into a destination scope, with an internal origin link. It does not expose restricted source titles, transcript contents, neighboring records, or original evidence access. Future source edits require publication again. Withdrawing a publication stops future retrieval but cannot remove information already returned to an agent.

The snapshot's destination visibility is governed by the same binding checks. Internal provenance links do not grant source access. The lifecycle ticket owns publication status/update/withdrawal mechanics; this resolution fixes the visibility contract.

### Source provenance and evidence availability

Every new memory write has source attribution, including explicit writes. Fields unavailable from the source remain explicitly unknown; they are never fabricated.

| Attribution | Required meaning |
| --- | --- |
| Source-run identity | Stable service-assigned identity, retained when provisional provenance is later verified. |
| Binding | Bound boundary and project (project absent for Free sessions), plus applicable repository/worktree/session restrictions at the time of the write. |
| Origin | Registered client installation, provider, and internal conversation identity. |
| Agent details | Role/model when known; absence is explicit. |
| Timing and method | Write creation time, observation time when known, and explicit-write or capture method. |
| Native evidence | Provider-native session ID, native event/turn locator, and original transcript locator when available; preserve original-provider attribution. |
| Code evidence | Registered repository and available commit, path, and symbol pointers; no raw source mirror. |
| Verification and availability | Distinguish provisional/verified identity from readable, unavailable, or ambiguous evidence. |

Native provider IDs and data-directory paths are installation-qualified aliases, not globally unique conversation identities. Explicit memory writes may proceed with provisional local provenance before native session verification; later verification adds mappings while preserving the original source-run identity.

Resuming a verified native session on its registered installation retains its conversation identity and creates new source runs for new work. A different provider or an unverified copied/imported session remains distinct until explicitly linked. Capture replay/idempotency and independent-evidence rules belong to their existing capture and lifecycle tickets.

Closing Chartr preserves attribution. Original transcripts may remain accessible only on their originating machine; remote transcript delivery is not required. Missing or ambiguous source evidence is reported visibly, and does not by itself change record eligibility or acceptance. Provider stores remain transcript authorities; SurrealDB does not become a full transcript archive.

### Scope/filter acceptance matrix

These cases determine scope eligibility; the lifecycle ticket separately determines which statuses are eligible. Tests must exercise both explicit access and capture-derived records where applicable.

| Record and requesting session | Scope eligibility |
| --- | --- |
| Project-wide record, same project and boundary on another machine/provider | Include. |
| Eligible operator-wide preference, allowed for the current boundary | Include by default; caller may narrow it away. |
| Repository-specific record, same project and same registered repository | Include. |
| Repository-specific record, another repository in the same project | Exclude unless explicitly shared/generalized. |
| Worktree-specific record, another worktree or clone of the same repository | Exclude unless explicitly shared/generalized. |
| Unrelated project with an identical basename or slug | Exclude. |
| Work record requested from personal, including project/global matches | Exclude unless a destination-authorized publication exists. |
| Private record in the correct project and boundary | Exclude until operator enables private access for the binding. |
| Free-session non-global record | Only the owning session until explicit assignment/publication. |
| Free session seeking project records inferred only from cwd | Exclude. |
| Direct record ID, graph neighbor, or evidence reference outside the binding | Exclude without exposing restricted record content or metadata. |
| Missing/ambiguous source transcript for otherwise eligible knowledge | Include eligible knowledge with evidence unavailable/ambiguous. |
| Legacy record with unmapped project/boundary/sensitivity | Exclude from ordinary agent retrieval pending explicit compatibility mapping. |

Filters compose conjunctively: allowed work/personal boundary and private access, then project or eligible global preference (or the Free session's own records), then every applicable repository, worktree, and session restriction. The global alternative cannot bypass narrower restrictions. A destination publication is eligible only through its authorized destination scope.

Identity acceptance also covers a verified rename, a second-machine clone, a fork requiring linking, duplicate installation/native session identifiers, a resumed conversation, and a fresh conversation after a project/boundary switch. Neither provider changes nor restarted clients widen access.

### Baseline changes and remaining ticket ownership

- Draft sections 6.1, 7.2, and 13's T3-project/basename mapping are replaced by stable memory-project and registered repository/checkout identities. Reason: cross-machine continuity, rename safety, and collision isolation.
- Draft section 6.1's caller-selected sensitivity filters and default `normal,work` behavior are replaced by operator-bound work/personal separation plus private opt-in. Section 11's direct-get filter bypass cannot bypass scope. Reason: agent parameters and record IDs must not expand authority.
- Draft D5 and section 7.1's T3 thread/turn pointers are replaced with provider-neutral, installation-qualified source attribution. D5's no-full-transcript-mirror principle is retained. Reason: Chartr owns conversations/terminal sessions while provider stores own transcripts; there is no assumed T3 turn stream.
- Draft D2's single existing namespace/database is retained; logical boundaries do not require separate databases. D4's source-code pointers and git authority are retained.
- The initial single-machine premise was already superseded by [Choose the first-version provider and rollout contract](03-first-version-support-contract.md); this ticket applies that decision rather than reopening deployment scope.

[Set the knowledge lifecycle and authority rules](05-knowledge-lifecycle-contract.md) owns publication lifecycle, scoped record conflicts, and acceptance/independence rules. [Choose the memory runtime and agent tool boundary](06-runtime-and-tool-boundary.md) owns trusted binding delivery, credential/configuration ownership, concrete schemas, and identity registration mechanisms. [Define automatic capture and extraction semantics](07-automatic-capture-contract.md) owns run granularity, replay, and provider-specific attribution adapters. [Choose retrieval, embedding, and context budgets](08-retrieval-and-embedding-contract.md) applies the matrix to all retrieval paths. [Find the minimal operator memory controls](09-operator-memory-controls.md) owns the binding/link/share interaction. [Define compatibility with existing SurrealDB history](13-existing-surrealdb-history.md) maps existing records without silently assigning scope. These are already-ticketed decisions; no additional fog patch needs a duplicate ticket.
