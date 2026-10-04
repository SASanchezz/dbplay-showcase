# ADR-0003: Defence in depth for executing untrusted SQL

- **Status:** Accepted
- **Date:** 2026-09

## Context

The core feature is running arbitrary SQL from anonymous internet users against a Postgres instance that also hosts the application's data. Any single protection layer can have bugs.

## Decision

Stack independent layers, so a failure in any one of them is contained by the others:

| Layer | Mechanism | Stops |
|---|---|---|
| 1. Privileges (the real boundary) | Dedicated `quest_runner` role: no superuser, createdb, createrole or bypassrls, and **no grants on app tables**. The API checks this at boot and **refuses to start** if the role is over-privileged | Reading or modifying user data, cluster-level actions |
| 2. Isolation | Every grade runs in a fresh `quest_run_<uuid>` schema on its own pooled connection, inside a transaction that is **always rolled back**. The schema is then dropped and the session reset (or the connection destroyed if it can't be reset) | Cross-submission leakage, leftover objects |
| 3. SQL policy | Parse with the real Postgres parser (`libpg-query`) and allow only the statement kinds the quest permits. Reject schema-qualified names and functions that sleep, touch the server, or execute SQL from strings (`pg_*`, `lo_*`, `dblink`, `set_config`, sequence functions, …) | Escaping the sandbox via `search_path` tricks, smuggled dynamic SQL |
| 4. Resource limits | Per-connection `statement_timeout` and `idle_in_transaction_session_timeout`, also set on the role as a backstop. Pool capped at 5 concurrent sandboxes | Runaway queries, connection exhaustion |
| 5. Abuse limits | Per-user submission and test-run rate limits with Postgres-backed counters | Flooding, brute-force cost probing |
| 6. Observability hygiene | Sentry configured not to capture request bodies or locals | Leaking passwords or submitted SQL into telemetry |

## Consequences

- Parsing on the AST rather than with regexes means comments, quoting and casing can't bypass the policy.
- Layer 1 is designed so that a bug in layers 2–3 is an annoyance, not a breach.
- Building a per-grade schema and running `VACUUM` adds latency to each submission. That's acceptable for an explicit submit action and is the price of real isolation.
- Seeding uses `ANALYZE` on its own tables only. A bare `ANALYZE;` would deadlock concurrent grading.
