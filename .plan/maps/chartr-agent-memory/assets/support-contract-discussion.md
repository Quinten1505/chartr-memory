# Provider and rollout contract discussion

Working notes for [Choose the first-version provider and rollout contract](../tickets/03-first-version-support-contract.md). These are working questions and partial operator answers, not a ticket resolution. Started 2026-09-26.

Terminology proposed for this discussion: manual memory means explicit agent calls to memory tools; automatic extraction means background derivation of memory from completed work. No glossary terms have been confirmed yet.

## Round one: answered

Operator answers: macOS/Linux initially; Windows deferred. Required explicit-memory providers are Codex, Claude Code, and Grok (replacing the recommended OpenCode). Restart/resume may be required. Unsupported/custom agents stay usable with memory capability clearly shown; explicitly configured MCP access is permitted without a full support guarantee.

1. **Platforms:** Which operating systems must the first usable Chartr memory release support? Current Chartr/Herdr runtime targets macOS and Linux; the checkout is on Windows.
   - macOS and Linux; Windows deferred (recommended to match existing runtime targets).
   - Native Windows must also work in v1.
   - Linux/WSL first; other platforms later (a scope preference, not a claim that WSL has been verified).
2. **Initial providers:** Which providers must have verified explicit memory reads/writes in the first usable release? Automatic-capture coverage follows after this selection.
   - Codex, Claude Code, and OpenCode (recommended; researched representative set).
   - Codex and Claude Code first.
   - Every MCP-capable provider registered in Chartr.
3. **Existing sessions:** After configuration, must already-running sessions gain memory without restarting?
   - Restart or resume may be required (recommended; guarantee newly started/resumed sessions).
   - Existing sessions must gain access without restarting.
4. **Unsupported/custom agents:** What happens when an agent lacks verified memory support?
   - Keep terminal usable, clearly report unavailable/unverified memory, and permit explicitly configured MCP access without a full support guarantee (recommended).
   - Require working memory before launching an agent.
   - Keep terminal usable but offer no memory integration until verified.

The operator may supply a different answer to any question. Later rounds must settle automatic-capture coverage, stage acceptance scenarios, and remaining branches of this ticket without deciding the runtime, schema, or capture mechanisms owned by other tickets.

## Round two: answered

Operator answers: automatic capture is required for Codex, Claude Code, and Grok, with incremental rollout. Require the full cross-provider explicit-memory acceptance matrix on both macOS/Linux. Importing old sessions is also required in v1; the recommendation to defer it was rejected.

5. **Automatic-capture coverage:** Once the automatic-extraction stage is complete, must it work for Codex, Claude Code, and Grok?
   - All three required, with incremental rollout and independently usable explicit memory (recommended; Grok feasibility is under research).
   - Codex and Claude Code required; Grok may remain explicit-only.
   - One provider initially; choose after capability research.
6. **Explicit-memory acceptance:** Require every selected provider to write memory, another selected provider to retrieve it in a fresh session after restart, visible origin, and exclusion of unrelated project memory, across all three providers on both macOS/Linux? Exact scope rules belong to the identity ticket.
   - Require this cross-provider test matrix (recommended).
   - Validate one provider pair first and expand later.
7. **Historical capture:** Begin with work after activation or include old-session import?
   - New work plus restart recovery since activation; older-session import deferred (recommended).
   - Importing old sessions is required in v1.

Grok capability evidence is being gathered in [Verify Grok memory and capture interfaces](../tickets/11-grok-memory-capabilities.md). The provider choice remains the operator's requirement; absent evidence must not silently substitute a different provider.

## Round three: answered with an import-source clarification pending

Operator answers: use the already-running SurrealDB instance and access its existing history. This corrects the assumption that setup should provision a new database. Declared historical compatibility with explicit unsupported-session reporting is accepted. The automatic/import acceptance matrix is accepted. It remains unclear whether a separate import of provider sessions absent from that database is also required; do not equate stored database history with provider transcript files.

8. **Import selection:** For all three providers on both target platforms, should the operator choose old sessions/projects or should setup import all available history? Import derives typed memories with source references; provider transcripts remain in their existing stores.
   - Explicit selection, including an optional import-all action (recommended).
   - Automatically import all discoverable history during setup.
9. **Historical compatibility:** Must v1 understand every historical format, or support a declared provider/version matrix with explicit reporting of missing, unreadable, or unsupported evidence?
   - Declared compatibility with visible unsupported-session reporting (recommended).
   - Every discoverable historical session must import before v1 is complete.
10. **Automatic/import acceptance:** Require completed new work and selected history to produce attributable typed memories under the agreed authority rules; cross-provider retrieval of eligible results; interruption replay without duplicate memories; and visible skipped/failed capture/import, for all three providers on macOS/Linux?
   - Require the full automatic-capture and import test matrix (recommended).
   - Start with basic extraction/retrieval and defer replay/failure reporting.

Exact mechanisms belong to the capture, historical-import, authority, and runtime tickets. Grok research is now complete and identifies candidate public interfaces with released-binary validation still required.

## Round four: answered

Operator answer to the history question: reuse the existing database and its stored history; no separate old-provider-session import is required. This supersedes the earlier interpretation of the old-session requirement. The historical-import ticket is ruled out; existing-history compatibility remains required. The automatic-stage acceptance scenarios therefore cover new-work extraction and access to existing database history, not batch ingestion of provider log files.

11. **Existing history versus additional import:** V1 must reuse the existing SurrealDB instance and its stored history. Is that the entire historical-data requirement, or must v1 also import older provider sessions not already stored there?
   - Reuse the existing database and stored history; no separate old-provider-session import required.
   - Reuse the existing database/history and additionally import older provider sessions not yet stored.

Read-only discovery is checking available connection context. An unavailable harness-memory adapter is not evidence that the existing SurrealDB instance is stopped. No service or database changes are authorized by this planning investigation.

12. **Database location:** Read-only discovery found a Windows SurrealDB process listening on `127.0.0.1:8000`. Is this same instance intended for the macOS/Linux Chartr release, or is it development state while deployment uses an existing local instance on the target machine?
   - The same Windows-hosted instance must serve Chartr too; this adds a network-connection requirement, not replication.
   - This is development state; deployment uses an existing local instance on the target machine.

The existing-database fog has graduated into [Define compatibility with existing SurrealDB history](../tickets/13-existing-surrealdb-history.md). Database identity and schema are not inferred from the baseline document or process port.

Operator answer to the placement question: the same Windows-hosted instance must serve Chartr's macOS/Linux clients. This revises the earlier single-machine boundary; it does not require replication or multiple operators. All questions in this support-contract discussion are answered. The canonical resolution is the ticket's Answer; these notes retain the question/response trail, including superseded interpretations.
