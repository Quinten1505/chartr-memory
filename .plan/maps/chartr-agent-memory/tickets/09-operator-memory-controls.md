---
type: prototype
blocked_by: [05, 06, 07]
undermined_by: []
assets: []
---
# Find the minimal operator memory controls

## Question

What minimal Chartr interaction lets the operator configure access to the existing memory service/database, understand its health, and confirm or reject proposed hard rules with enough evidence to make the decision? Prototype the connection setup, unavailable/degraded state, and one proposed-to-confirmed interaction using the chosen runtime, existing-history compatibility, and lifecycle. Include recovery actions and where confirmation authority is conveyed to the server. Keep full memory browsing/editing and separate provider-transcript import outside this effort; the artifact exists to resolve behavior, not ship UI.

Options for discussion (not decisions):

- A compact native settings surface and proposal confirmation flow
- A small plugin pane with setup, health, and confirmations

## Done when

A throwaway artifact is linked and the human responds to it; accepted behavior and remaining questions are recorded. Generating an artifact alone does not resolve this ticket.
