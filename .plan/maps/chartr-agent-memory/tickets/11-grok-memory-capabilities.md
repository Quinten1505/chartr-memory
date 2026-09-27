---
type: research
assignee: Quinten1505
research_branch: research/chartr-memory-grok
blocked_by: []
undermined_by: []
assets: [grok-capabilities.md]
---
# Verify Grok memory and capture interfaces

## Question

For the Grok CLI/Grok Build provider registered by Chartr, which primary-source-documented interfaces support explicit MCP memory reads/writes and automatic capture on macOS/Linux? Identify the exact product rather than confusing it with the xAI inference API or unrelated third-party Grok CLIs. Inspect Chartr and its pinned Herdr integration, then verify MCP configuration, session/evidence identity, completion signals, transcript/export access, and supported version/platform limits. State unknowns and distinguish existing integration, documented capability, and untested behavior. The operator selected Grok alongside Codex and Claude Code as required first-version providers; do not substitute OpenCode or decide to drop Grok.

## Done when

A cited report records feasible documented seams or precisely bounded missing evidence, identifies any prerequisite experiment/access requirement, and explains implications for provider support and capture decisions without choosing policy. Preserve the report on its research branch and link it here.

## Answer

Official Grok Build documents STDIO/HTTP MCP access, hooks, native session identity, session persistence, and Markdown export. Current source includes optional transcript/prompt identity and cancellation events. Pinned Herdr only reports session-start identity, and Chartr locates Grok metadata/logs without parsing its transcript. Public docs/source disagree on some resume and stop-hook semantics; no installed release or runtime behavior was tested.

[The Grok capability report](../../../../docs/research/2026-09-26-chartr-memory-grok-capabilities.md) identifies candidate interfaces and bounded synthetic verification on macOS/Linux. Preserved on `research/chartr-memory-grok` at `3517399e39b8cd70c9a0f72175f373e42ddd3461`. No provider policy or capture mechanism is selected by this research. Grok remains a required provider; missing runtime evidence does not substitute OpenCode.
