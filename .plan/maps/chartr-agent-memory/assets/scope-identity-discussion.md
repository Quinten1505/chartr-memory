# Memory scope and provenance discussion

Discussion history for [Define memory scope and provenance identities](../tickets/04-scope-and-provenance-identity.md). All three rounds were confirmed on 2026-09-26. The ticket's Answer holds the canonical resolution; these notes retain the questions and alternatives.

## Round one: answered

1. **Project identity:** What should define a memory project across machines and folders?
   - Stable project identity across machines and folders (recommended; selected). Suggest matches from repository information and explicitly link when ambiguous; folder names and paths only help locate the project.
   - Repository identity defines each project.
   - Each Chartr folder space has separate memory.
2. **Work/personal separation:** What separation should exist between work and personal knowledge?
   - Separate work/personal boundaries; explicitly share selected knowledge (recommended; selected). General preferences can be marked safe for both.
   - Share across work/personal, exclude records marked private.
   - Keep every project isolated, explicitly share selected knowledge.
3. **Global preferences:** Should sessions receive relevant operator-wide preferences alongside project knowledge by default?
   - Include eligible operator-wide preferences by default (recommended; selected), respecting the selected memory boundary.
   - Use only project knowledge unless broader recall is requested.
   - Choose separately for each project.

## Baseline corrections and verified facts

- The ticket's single-machine wording is corrected to match the resolved provider/rollout contract. This is not a new scope expansion.
- The selected stable memory-project identity replaces the draft's path/basename-derived project assumption. The selected work/personal boundary requires more than the draft's caller-selected sensitivity filter; round two selects an operator-controlled binding enforced on all access paths.
- Chartr folder-space keys are path based (`crates/chartr/src/space.rs`); foreground cwd is presentation metadata, not ownership. The established UI space model does not itself provide cross-machine memory identity.
- Current conversation keys qualify native session IDs by provider and local data namespace (`crates/chartr-conversations/src/store.rs`). Grok uses `local:grok`; these keys lack a host discriminator. Source identities must not be assumed globally unique.
- Provider stores own transcripts. Durable source pointers and universal cross-client access to source contents are different requirements.

## Round two: answered

The operator selected every recommended answer below. Stable identity linking, project membership, worktree qualification, Free-session behavior, operator-controlled scope, and local-only evidence availability were agreed in this round; the remaining edge cases were settled in round three.

4. **Linking another checkout:** On a second machine, should an exact match to a previously linked repository automatically reuse its memory project?
   - Automatically reuse an unambiguous registered repository identity inside the selected memory boundary; ask for ambiguous matches and forks. Never match solely by basename or path (recommended).
   - Require confirmation for each new checkout, including exact matches.
   - Require manual selection for every new checkout, with no repository-based matching.
5. **Project membership:** Can a memory project contain several repositories or non-git folders?
   - Yes, explicitly link them; keep repository-specific knowledge qualified to its repository (recommended).
   - One repository per memory project; non-git folders get separate projects.
6. **Worktree knowledge:** Should linked worktrees share project knowledge while checkout-specific facts remain local to that worktree?
   - Yes; share project knowledge, keep worktree-specific facts local unless deliberately generalized (recommended).
   - Share all knowledge across the project's worktrees by default.
   - Isolate worktree memory unless explicitly shared.
7. **Free sessions:** What memory should a session without an assigned project access?
   - Select a work/personal boundary and receive its eligible global preferences; keep new non-global knowledge session-local until assigned to a project. Changing cwd does not silently change its memory binding (recommended).
   - Require assigning a memory project before any memory access.
   - Infer the memory project from the current directory and change it when cwd changes.
8. **Scope authority:** May an agent broaden access by passing another project or boundary to a memory tool?
   - No; Chartr's operator-selected binding constrains all reads/writes, direct record lookup, and linked evidence. Agents may narrow scope or request an explicit operator-approved switch/share (recommended).
   - Let agents search all projects within the selected boundary, but require operator approval to cross work/personal boundaries.
9. **Evidence availability:** Must original transcript contents be accessible from every client, or is durable attribution with a visible unavailable-source state sufficient?
   - Durable provider/source attribution is required; evidence may remain readable only on the originating machine. Closing Chartr retains attribution; absent logs are reported as unavailable (recommended).
   - Require original evidence to be readable from every client, adding a remote evidence-access requirement without storing full transcripts in memory.

## Round three: answered

The operator answered "All recommended" to this final round, confirming the proposed scope/provenance contract and its scope/filter examples. Lifecycle status, knowledge deduplication, capture replay, runtime credentials, and legacy-data mapping remain owned by their existing tickets.

10. **Stable identifiers and collisions:** Use opaque, stable identifiers for memory projects, repositories, client installations, checkouts/worktrees, conversations, and source runs. Names, absolute paths, and normalized repository remotes are aliases or locators, never sole durable identities. A registered repository alias may select a project only when unique within the selected boundary. A renamed/moved checkout retains identity when its registration can be verified; otherwise ask to relink. New clones get their own checkout identity while sharing the registered repository/project. Forks, conflicting aliases, duplicate installation registrations, and ambiguous matches never silently merge. Identical knowledge slugs in different projects/boundaries remain distinct; within-scope write conflict policy belongs to the lifecycle ticket.
    - Adopt these identity/collision rules (recommended).
    - Use a different identity or automatic-merge policy, to be specified by the operator.
11. **Meaning of private:** Treat private as an additional opt-in restriction within either work or personal, not as a synonym for personal. Private records are excluded from default recall, bootstrap, direct lookup, and graph/evidence traversal until the operator explicitly enables private access for that binding. Unknown or unmapped scope never becomes global or normal by inference; exact legacy mapping belongs to the existing-history compatibility ticket.
    - Adopt private opt-in and exclusion of unknown scope (recommended).
    - Do not add a separate private restriction; work/personal separation is enough.
12. **Changing scope during a conversation:** Previously retrieved text cannot be removed from a provider's context merely by changing a filter. Switching project or work/personal boundary therefore starts a fresh provider conversation with the new binding. Reusing an existing conversation preserves its old binding; moving a UI pane or changing cwd does not authorize new memory access. Explicit assignment/sharing can move selected Free-session knowledge into a project, but does not silently reclassify that session's existing contents.
    - Start a fresh conversation when changing project/boundary (recommended).
    - Allow an explicit in-conversation scope switch and accept that prior context may be carried into the destination scope.
13. **Explicit sharing:** Share a selected record through a destination-scoped publication linked internally to its origin. The operator reviews the content being exposed; hidden source text, restricted titles, neighboring records, and transcript access do not become shared implicitly. The publication is a snapshot, so future source edits are not published automatically. Withdrawing it stops future memory retrieval but cannot erase content already returned to an agent. Exact publication lifecycle is for the lifecycle ticket; this ticket fixes the visibility requirement.
    - Share reviewed snapshots with destination-specific visibility (recommended).
    - Share the live record so future source updates automatically appear at the destination.
14. **Source provenance:** Every new memory write carries a service-assigned source-run identity and the bound project/boundary, originating installation, provider, conversation, agent role/model when known, creation/observation time, and capture method. Keep native provider session ID, native event/turn locator, original transcript locator, and code pointers (repository plus commit/path/symbol) when available; missing fields are explicitly unknown, never fabricated. Explicit writes without a verified native session may use provisional local provenance; later verification adds a mapping while preserving the original run identity. A resumed verified native session on its registered installation stays the same conversation with new source runs; a different provider or an unverified imported/copied session remains distinct until explicitly linked. Missing transcripts affect evidence availability, not record visibility or acceptance by themselves. Capture replay/idempotency remains for the capture ticket.
    - Adopt this provenance contract, including provisional explicit writes and visible missing evidence (recommended).
    - Block memory writes until a native provider session and exact source location are verified.

## Confirmed scope/filter examples

The operator confirmed the proposed matrix with round three. It now lives in the canonical [ticket resolution](../tickets/04-scope-and-provenance-identity.md#scopefilter-acceptance-matrix).
