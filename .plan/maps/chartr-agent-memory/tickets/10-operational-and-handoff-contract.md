---
type: grilling
blocked_by: [03, 04, 05, 06, 07, 08, 09]
undermined_by: []
assets: []
---
# Set recovery guarantees and implementation handoff criteria

## Question

What operational guarantees and verification evidence make the selected design ready for implementation? Decide ownership and guarantees for database/schema upgrade, backup/restore, corrupt or missing evidence handling, embedding migration, capture replay, and staged rollback for one operator whose macOS/Linux clients use the same existing Windows-hosted database. Preserve existing history and current consumers. Reconcile all selected contracts into acceptance scenarios, including concurrent writes, wrong-scope retrieval, provider shutdown, network interruption, and backend failure. Identify unresolved design contradictions or experiments as new decision tickets rather than closing them with implementation guesses.

Options for discussion (not decisions):

- Local recoverability with explicit operator-triggered backup/restore and stage gates
- Managed local backups and automated recovery checks as first-version requirements

## Done when

The human agrees to recovery and migration guarantees, stage-specific acceptance criteria, and the specification handoff structure. The map closes only after newly exposed questions and fog are resolved.
