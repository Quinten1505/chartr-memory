---
type: grilling
blocked_by: [02, 04, 05, 13]
undermined_by: []
assets: []
---
# Choose retrieval, embedding, and context budgets

## Question

What should bootstrap and recall return, under what scope/lifecycle filters, and within what enforceable token and latency budgets? Decide lexical/vector fusion, graph expansion, ranking of constraints versus other knowledge, treatment of proposed records and contradictions, and evidence presentation. Select the embedding configuration strategy, offline behavior, backfill/rebuild policy, and rules preventing graph expansion from bypassing scope. Use concrete retrieval scenarios to decide quality criteria rather than adopting the draft's numerical thresholds without evidence.

Options for discussion (not decisions):

- Local embeddings plus lexical fallback and bounded graph expansion
- Lexical-first rollout with embeddings added at a defined quality gate
- Configured local or hosted embedder behind one fixed-dimensional contract

## Done when

A retrieval/filter contract, embedding lifecycle, token accounting policy, and representative quality/failure examples are agreed; any required performance or quality experiment is separately ticketed.
