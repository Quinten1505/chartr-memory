---
type: grilling
assignee: Quinten1505
claim_thread: 01a0de53-b221-7703-ac40-602ca353c477
claimed_at: 2026-09-26T15:39:17.032551+00:00
blocked_by: [01, 11]
undermined_by: []
assets: [support-contract-discussion.md]
---
# Choose the first-version provider and rollout contract

## Question

Which providers and operating systems must the first usable Chartr memory release support, and what must work in the manual-memory stage versus the automatic-extraction stage? Define observable cross-provider continuity scenarios, what happens for unsupported/custom agents, and whether already-running sessions must gain memory access. The agreed destination includes both stages; this decision fixes their acceptance boundary without treating every provider as equally capable.

Options for discussion (not decisions):

- Start with a small named provider set and expand through capability-based adapters
- Require manual memory for every MCP-capable Chartr provider, with automatic capture for a named subset
- Require manual memory and automatic capture for every currently supported Chartr provider

## Done when

The human selects the initial provider/OS matrix, stage gates, unsupported-provider behavior, and concrete success scenarios.

## Answer

Resolved with the operator on 2026-09-26.

### Required support and deployment

- Chartr clients target macOS and Linux. Native Windows Chartr support is deferred.
- Codex, Claude Code, and Grok Build are the required providers for explicit memory access and automatic capture. OpenCode remains researched but is not a required first-version provider.
- Use the operator's same existing Windows-hosted SurrealDB instance and its stored history. The verified development connection is `ws://127.0.0.1:8000`, namespace `harness`, database `memory`; this is not yet a reachable macOS/Linux deployment endpoint. The runtime decision must supply the remote connection and access boundary. No database rebind, provisioning, or credential changes were performed here.
- This remains one operator using one database instance. The earlier single-machine restriction is superseded because the operator explicitly requires macOS/Linux clients to use the Windows-hosted instance. Multi-user tenancy and database replication remain out of scope.
- Existing database history must be accessible according to the selected scope, lifecycle, and compatibility rules. Separate ingestion of old provider transcript files is not required. [Define compatibility with existing SurrealDB history](13-existing-surrealdb-history.md) owns the detailed mapping and coexistence contract.

### Rollout and acceptance

1. **Explicit-memory stage:** independently usable before automatic capture is ready. Each required provider must be able to write through the memory tools and retrieve eligible existing history. Verify cross-provider retrieval in fresh sessions after restart, visible provenance, and exclusion of unrelated project memory. Cover all six directed provider pairings on each target OS against the shared Windows-hosted instance.
2. **Automatic-capture stage:** may roll out provider by provider, but is complete only when all three required providers pass on macOS and Linux. Completed new work produces attributable typed memory under the agreed authority rules; another provider can retrieve eligible results; interruption/replay does not create duplicate memories; and skipped or failed capture is visible. Existing database history remains accessible alongside newly captured knowledge. This does not authorize automatic activation of hard rules; that policy belongs to the lifecycle ticket.
3. **Compatibility evidence:** use a declared compatibility contract and report inaccessible or unsupported history rather than silently claiming complete access. Provider capabilities documented in research are not a tested version matrix. Runtime acceptance must name the tested provider releases and existing database/schema compatibility. Exact adapters, extraction mechanisms, scope matching, retry rules, and budgets remain in their responsible tickets.

### Session and fallback behavior

- Access is guaranteed for supported newly started or explicitly resumed sessions after configuration; restart/resume may be required where a provider cannot reload. Seamless upgrade of every already-running session is not a requirement.
- Unsupported/custom agents remain usable as terminals. Show whether memory is available, unavailable, or unverified. Explicitly configured MCP access may be used without a full support guarantee; memory support is not a prerequisite for launching an unsupported agent.

### Evidence and boundaries

[Provider capability research](01-provider-memory-capabilities.md) and [Grok capability research](11-grok-memory-capabilities.md) establish candidate public interfaces. [Existing-instance evidence](../assets/existing-instance-context.md) establishes the current service/configuration, not its schema or remote reachability. Detailed database compatibility and network/runtime decisions remain open. The operator's stored-history clarification rules the separate provider-import ticket out of scope.
