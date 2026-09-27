# Chartr provider memory and capture capabilities

Date: 2026-09-26. Research for **Establish provider memory and capture capabilities**. This report establishes integration facts; it does not select supported providers, capture policy, or an architecture.

## Findings

Explicit shared-memory tools can use documented MCP clients in Codex, Claude Code, and OpenCode while their terminal interfaces remain intact. Automatic capture also has documented hooks/plugins to investigate. Neither fact means that Chartr currently implements memory capture. Its existing provider integration supplies native identity and presentation state, not a durable, complete stream of turns.

Evidence levels below distinguish **local implementation**, **documented capability**, and **inference**. No provider was launched, no private transcript was inspected, and no installed provider version was established. Documentation was fetched on the date above; compatibility with the operator's installed binaries remains untested.

## Local baseline and pinned Herdr

Inspected Chartr commit `32f5977db187034c4b920a58152a0996b234bc06`. The canonical catalog contains nine providers: Codex, Claude, Kimi, Pi, OMP, Cursor, Antigravity, Grok, and OpenCode. Icon assets for other brands do not add catalog support. Codex/Claude declare terminal-prompt transport, OpenCode declares API transport, and the remaining entries declare terminal-only transport. These declarations are not MCP capability claims. [Catalog](../../crates/chartr-agent/src/lib.rs)

Chartr pins Herdr `0.8.2-chartr.5a2dee700eee`, upstream revision `5a2dee700eeeea68267a4d16777307632f77172f`, protocol 22. It owns a private daemon namespace; native PTYs can outlive a window. Chartr's `PaneId`, `TerminalId`, and Herdr workspace are distinct from provider-native session identity. The control client installs provider integrations, verifies their installed state, and launches ordinary CLIs with `agent.start`. [Pin and ownership](../../crates/chartr-herdr/src/lib.rs), [control client](../../crates/chartr-herdr/src/control.rs), [namespace](../../crates/chartr-herdr/src/namespace.rs)

Pinned upstream integration scripts were fetched read-only from the exact revision. Codex's script accepts `SessionStart`, requires a transcript path, and reports native session identity; Claude's likewise accepts `SessionStart`, excludes subagents, and reports identity/path. Neither inspected script forwards a turn-completed event or transcript content. OpenCode's plugin reports root-session identity and working/blocked/idle state from provider events, guarding against child-session identity replacing the root. These are source findings, not live tests. [Codex script](https://github.com/herdrdev/herdr/blob/5a2dee700eeeea68267a4d16777307632f77172f/src/integration/assets/codex/herdr-agent-state.sh), [Claude script](https://github.com/herdrdev/herdr/blob/5a2dee700eeeea68267a4d16777307632f77172f/src/integration/assets/claude/herdr-agent-state.sh), [OpenCode plugin](https://github.com/herdrdev/herdr/blob/5a2dee700eeeea68267a4d16777307632f77172f/src/integration/assets/opencode/herdr-agent-state.js)

Chartr exposes `agent_session {source, agent, kind, value}` and coarse status. Its conversation store is SQLite rows containing serialized `Conversation` objects; it is not T3's thread/turn event store. Conversations hold optional native identity, messages, and a transient runtime route. Runtime reconciliation checks pane, terminal, provider, and native identity to avoid routing a reused pane to an old conversation. [Wire types](../../crates/chartr-herdr/src/protocol.rs), [conversation model](../../crates/chartr-conversations/src/lib.rs), [store](../../crates/chartr-conversations/src/store.rs), [identity checks](../../crates/chartr/src/app/conversations.rs)

Existing history adapters are bounded display readers: JSONL input is limited to 32 MiB and message retention to 300. Several parsed messages are marked `complete: true` without a universal turn event. Codex/Claude use provider files; OpenCode uses its SQLite schema. They are not an established durable capture queue. An independent OpenCode export path already executes `opencode export <id>` and verifies returned identity. [Readers](../../crates/chartr-conversations/src/transcripts.rs), [export](../../crates/chartr-conversations/src/session_logs.rs)

## Representative provider matrix

| Provider | Documented explicit memory configuration | Identity, transcript, and completion seams | Relationship to existing Chartr |
| --- | --- | --- | --- |
| Codex | MCP supports STDIO and HTTP; `mcp_servers` configuration supports command/environment or URL configuration. [MCP documentation](https://learn.chatgpt.com/docs/extend/mcp?surface=cli) | Hooks expose `session_id`, nullable `transcript_path`, and event-specific `turn_id`; `Stop` includes latest assistant message and continuation state. `Interrupt` is separate. Non-managed hooks require trust. The docs expressly say transcript format is unstable. [Hooks](https://learn.chatgpt.com/docs/hooks) | Pinned Herdr already reports session start identity, but automatic extraction and durable completion records are absent from the inspected Chartr interfaces. Modern hook availability on installed Codex needs a version check and a focused test. |
| Claude Code | `claude mcp add` supports STDIO/HTTP. User scope uses `~/.claude.json`; project scope uses `.mcp.json` with interactive approval behavior. [MCP documentation](https://code.claude.com/docs/en/mcp) | Hook inputs provide session identity and transcript path. `Stop` gives latest assistant message and continuation state; `SubagentStop` has separate agent identity/path. `StopFailure` is a distinct failure signal. These are documented hook envelopes; a path alone does not establish a permanently stable transcript schema. [Hooks reference](https://code.claude.com/docs/en/hooks) | Existing integration identifies the root session; local history parsing is a separate adapter. New capture hooks would need coexistence with managed Herdr hooks and version verification. |
| OpenCode | `mcp` entries in OpenCode config support local command/environment and remote URL servers. [MCP documentation](https://opencode.ai/docs/mcp-servers/) | Plugins receive `session.idle`, `session.status`, message events and other lifecycle events. Public server API exposes session/message IDs, message reads, and SSE events. CLI supports session export. [Plugins](https://opencode.ai/docs/plugins/), [server API](https://opencode.ai/docs/server/), [CLI](https://opencode.ai/docs/cli/) | Herdr's plugin already translates lifecycle into coarse state. Chartr has a private DB reader and a CLI export path. Public message API/export are alternatives to relying on DB internals, subject to endpoint discovery and version testing. |

Codex also documents app-server `thread/read` and `turn/completed`, including completed/interrupted/failed states. That is a richer integration surface, but these docs do not establish a passive event subscription to every independently launched terminal process. Replacing terminal ownership with app-server orchestration is not a prerequisite proven by this research. [App server](https://learn.chatgpt.com/docs/app-server)

**Inference:** use separate capability dimensions for tool access, native identity, readable evidence, and reliable capture triggering. A provider supporting MCP does not thereby expose its transcript or completion events through MCP. A hook firing at a stop does not by itself prove all child/background activity has ended or that a transcript is durably flushed. Those semantics need adapter tests.

## Remaining catalog: local inventory only

All external MCP, hook, and transcript compatibility claims for these six providers remain **unverified**. Paths and adapters below describe Chartr code, not a promise from the provider.

| Provider | Local evidence |
| --- | --- |
| Kimi | Terminal-only catalog entry; dedicated history/session-log adapter, default `.kimi-code` with `KIMI_CODE_HOME` override. |
| Pi | Terminal-only; native path identity and JSONL reader under `.pi/agent/sessions`; managed plugin compatibility shim handles legacy `agent_end` and newer `agent_settled`. |
| OMP | Terminal-only; path identity and JSONL reader; `.omp/agent` default with config/environment overrides. |
| Cursor | Terminal-only; possible session-log lookup under projects, but history reader deliberately returns no transcript. |
| Antigravity | Terminal-only; possible log lookup under Antigravity brain directories, but history reader deliberately returns no transcript. |
| Grok | Terminal-only; possible session-log lookup under sessions, but history reader deliberately returns no transcript. |

Sources: [catalog](../../crates/chartr-agent/src/lib.rs), [paths/readers](../../crates/chartr-conversations/src/transcripts.rs), [Kimi adapter](../../crates/chartr-conversations/src/transcripts/kimi.rs), [Pi/OpenCode installation adjustments](../../crates/chartr-herdr/src/integration.rs). Successful discovery of a provider in Inbox is therefore not evidence of complete capture support.

## Draft assumptions needing explicit decisions

These findings justify reopening specific draft assumptions; they do not themselves change the baseline decisions.

- **D5 / L0 provenance:** T3 `thread_id`/`turn_id` cannot be substituted mechanically. Decide the mapping between Chartr conversation, provider session, optional provider turn/message, workspace/worktree, and evidence locator. The current store does not guarantee a complete replayable transcript. [Model and reader evidence above](../../crates/chartr-conversations/src/lib.rs)
- **D10 / turn-end reactor:** Chartr's observed idle state is not a durable turn-end event. Decide whether automatic capture uses provider hooks/plugins, public exports/APIs, private log adapters, or a combination with reconciliation. Preserve documented loss/duplication behavior rather than assuming an exactly-once callback. [Current protocol](../../crates/chartr-herdr/src/protocol.rs)
- **One `.mcp.json` forwarding mechanism:** the three provider configuration interfaces differ. Decide managed installation scope, coexistence, launch environment binding, and health detection per provider. MCP access can be staged independently of automatic extraction. [Codex](https://learn.chatgpt.com/docs/extend/mcp?surface=cli), [Claude](https://code.claude.com/docs/en/mcp), [OpenCode](https://opencode.ai/docs/mcp-servers/)
- **Desktop lifetime:** decide who owns capture when Chartr closes and PTYs continue, and how later startup reconciles missed work. Window closure, process exit, and completed agent turn are different facts. [Control lifecycle](../../crates/chartr-herdr/src/control.rs)
- **Host platform:** current Herdr bridge is Unix-gated and its fetch script only targets macOS/Linux. A Windows checkout does not establish Windows runtime support; whether Windows belongs in this memory design needs an explicit scope answer if not already settled. [Bridge](../../crates/chartr-herdr/src/lib.rs), [supported build targets](../../vendor/herdr/fetch.sh)

## Remaining validation and fog

Sharp follow-up questions: Which providers receive which initial capability tier? What is the canonical evidence identity? Which capture adapter contract owns cursors, retries, deduplication, cancellation, and degraded health? Who owns the background worker and MCP connection when no Chartr window is open?

Before implementation, test selected provider versions for hook installation/trust, resume/fork/subagent identity, interrupted turns, duplicate callbacks, transcript availability, and public export/API completeness. None of those tests was performed here. Broader unknowns include long-session compaction/retention behavior and coexistence with arbitrary user plugins; their detailed tickets depend on the selected capture contract. Private on-disk schemas must remain version-sensitive adapters rather than be represented as stable provider APIs.
