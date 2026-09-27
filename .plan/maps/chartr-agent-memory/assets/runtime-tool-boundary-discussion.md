# Runtime and tool boundary discussion

Questions were answered by the operator on 2026-09-26. The canonical resolution is in [Choose the memory runtime and agent tool boundary](../tickets/06-runtime-and-tool-boundary.md); this file preserves the exchange and its rationale.

| Question | Operator answer |
| --- | --- |
| Service placement | Independent service on the Windows host for now, designed to move to its own server later. |
| Initial bootstrap | The provider calls memory_bootstrap through its configured tool; Chartr checks availability and supplies a short instruction. |
| Service unreachable at session start | Launch the coding terminal with memory visibly unavailable; tool calls fail explicitly and can be retried after recovery. |
| Cross-machine route | Private routed network now; Tailscale access later. The offered choice included authenticated HTTPS and keeping the database on loopback. |
| Session credential, first pass | The operator initially selected a long-lived installation credential with server inference from provider-native IDs. After the identity-spoofing and unverified-ID problem was explained, the operator revised this to per-session scoped tokens. |
| Agent versus operator tools | Agents get scoped read, write, and proposal tools. Chartr uses a separate privileged control API for confirmation, publication, and binding changes. |
| Configuration | Chartr manages its provider MCP entries and connection checks, preserves other entries, and the memory service owns database credentials. |
| Continuing terminals | A Herdr terminal keeps memory access for its registered conversation after Chartr closes. Explicit session end or revocation stops access. |
| Embedder failure | Typed writes continue without embeddings; scoped lexical retrieval remains available with degraded health. |
| Token renewal | A per-session local MCP bridge renews scoped tokens using a protected client-installation credential, and outlives the UI. |
| Write applicability | The agent explicitly chooses a bound project, repository, worktree, or conversation scope; no implicit project-wide write. |
| Tool count | Keep the seven draft tools and correct the draft's “five tools” wording. Human-only actions use control APIs. |
| Operator-wide preference exception | An agent may propose one within its selected work/personal boundary; the operator must confirm it before ordinary access. |
| Agent status transitions | Only evidence-backed, non-authority transitions; accepting or activating instruction-bearing knowledge remains operator-only. |

The final proposed resolution was shown in a review asset and the operator replied: “Yes, this is the agreed contract.” A subsequent editorial correction restricted project and repository identity rows to trusted registration, consistent with the already resolved scope contract. No production service, database, provider configuration, or credential was changed.

## Evidence consulted

- [Provider capability report](../../../../docs/research/2026-09-26-chartr-memory-provider-capabilities.md) and [Grok capability report](../../../../docs/research/2026-09-26-chartr-memory-grok-capabilities.md).
- [Existing SurrealDB history decision](../tickets/13-existing-surrealdb-history.md) and its [read-only audit](./existing-surrealdb-readonly-audit.md).
- [Official Codex MCP configuration](https://learn.chatgpt.com/docs/extend/mcp?surface=cli), [Claude Code MCP configuration](https://code.claude.com/docs/en/mcp), and [Grok Build MCP configuration](https://docs.x.ai/build/features/mcp-servers).
- Chartr native plugin services and host: crates/chartr-plugin/src/lib.rs, crates/chartr-plugin/src/services.rs, crates/chartr-plugin-host/src/lib.rs. Chartr agent launch and Herdr lifecycle: plugins/agent/src/lib.rs, crates/chartr/src/session/launch.rs, crates/chartr-herdr/src/control.rs.
