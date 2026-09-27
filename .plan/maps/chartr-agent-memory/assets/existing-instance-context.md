# Existing SurrealDB connection context

Read-only environment discovery on 2026-09-26 for [Define compatibility with existing SurrealDB history](../tickets/13-existing-surrealdb-history.md).

- A native Windows SurrealDB process owns listener `127.0.0.1:8000`.
- The current environment declares `SURREAL_URL=ws://127.0.0.1:8000`, `SURREAL_NS=harness`, and `SURREAL_DB=memory`.
- Credential variables are present. Their values were neither exposed nor copied into these artifacts.
- `C:/Users/Quinten/.t3/memory/start.ps1` independently matches the listener, namespace and database and specifies SurrealKV storage.
- Persistent storage exists under `C:/Users/Quinten/.t3/memory/db`.

These facts establish a configured existing service, not schema compatibility or successful database access. Database version, schema, stored history, access permissions, and the cause of harness-memory's `backend_unavailable` response remain unverified. No database queries, private-history reads, service changes, or credential changes were performed.

The verified endpoint is Windows loopback. Reachability from the selected macOS/Linux application targets is not established and remains an explicit deployment-scope question. Do not copy this machine's paths into a portable runtime contract as defaults.
