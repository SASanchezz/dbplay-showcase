# ADR-0001: Grade on query plans, not wall-clock time

- **Status:** Accepted
- **Date:** 2026-09

## Context

dbplay teaches indexing and query tuning. The obvious success metric is "the query got faster", but timings are a poor grading signal:

- The browser engine (PGlite) is single-user Postgres in WASM, so wall-clock time is noisy and depends on the device.
- On a shared server, timings vary with load, cache state and neighbours.
- A grade that changes between two identical submissions destroys trust in a game with points and levels.

## Decision

Grade on the **query plan** and planner-reported numbers, never on elapsed time. Each quest declares win conditions that are evaluated against `EXPLAIN (ANALYZE)` output:

- shape checks: `no_seq_scan`, `require_index_scan`, `require_index_only_scan`, `no_sort`
- work checks: `max_rows_examined`, `max_heap_fetches`, `max_cost`, `max_total_cost`
- budget checks: `max_indexes`, `max_write_multiplier` (write amplification the indexes cause)
- correctness: `matches_reference` (a rewrite must return exactly the reference rows)

Timings are shown in the labs but explicitly labelled **illustrative**.

## Consequences

- Grades are deterministic and explainable: the UI can say *which* condition failed and why.
- Conditions teach real concepts (covering indexes, sort avoidance, write cost) instead of "make the number smaller".
- Cost ceilings depend on planner statistics, so datasets must be deterministic and analysed the same way every time (see [ADR-0005](0005-deterministic-datasets-and-verified-quests.md)).
- Index Only Scans report heap fetches unless the visibility map is set, so sandboxes `VACUUM` the seeded tables before grading.
