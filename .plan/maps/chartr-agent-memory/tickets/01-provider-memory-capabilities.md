---
type: research
assignee: Quinten1505
research_branch: research/chartr-memory-providers
blocked_by: []
undermined_by: []
assets: [provider-capabilities.md]
---
# Establish provider memory and capture capabilities

## Question

Which documented MCP configuration, session identity, transcript access, and completion signals can Chartr use for shared memory without replacing its terminal-based agent workflow? Inspect Chartr and its pinned Herdr integration, then verify current primary sources for Codex, Claude Code, and OpenCode as representative providers. Inventory the remaining Chartr providers from local source, explicitly marking capabilities not externally verified. Distinguish stable public interfaces from private log formats, and desktop lifecycle from agent lifecycle. This is evidence gathering, not a choice of supported providers.

## Done when

A dated, cited capability matrix identifies usable integration seams, unsupported assumptions from the T3 draft, version-sensitive interfaces, and unknowns. Findings live on a research branch with a linked Markdown report. No provider support or capture policy is decided.

## Answer

Codex, Claude Code, and OpenCode document MCP access and capture hooks/plugins that can preserve terminal workflows. Existing Chartr/Herdr integration does not provide the draft's durable T3 turn stream; conversation, native-session, pane, and terminal identities differ, and history readers are bounded. Other catalog providers are inventoried with external capabilities explicitly unverified. Installed versions and runtime behavior were not tested.

See [the provider capability report](../../../../docs/research/2026-09-26-chartr-memory-provider-capabilities.md), preserved on `research/chartr-memory-providers` at `9b11dcf9`. Provider tiers, platform scope, identity, configuration ownership, capture replay, and worker lifetime remain human decisions in the existing tickets; no support policy is selected here.
