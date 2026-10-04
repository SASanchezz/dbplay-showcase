# ADR-0006: Single-server Docker Compose, Postgres as the only stateful dependency

- **Status:** Accepted
- **Date:** 2026-09

## Context

dbplay is pre-revenue and operated by one engineer. The infrastructure has to be cheap, understandable at 2 a.m., and easy to move between providers.

## Options considered

| Option | Pros | Cons |
|---|---|---|
| PaaS (e.g. Render) | Zero ops | Higher cost at steady load, less control over Postgres roles and extensions needed for the grading sandbox |
| Kubernetes | Scales, standard | Massive overkill for one service and one database |
| **One VM + Docker Compose** | Cheap, portable, full control of Postgres | Single point of failure, manual scaling |

## Decision

- One Linux server running Docker Compose: `db` (Postgres 16, not internet-facing), `migrate` (migrations and idempotent seed, runs before every API start), `api` (NestJS), `caddy` (automatic HTTPS, both SPAs, reverse proxy).
- **No Redis:** rate-limit counters live in a Postgres table with an hourly purge job. One stateful dependency to back up and reason about.
- Daily database backups copied off the server. Release blockers (legal pages, email verification, backups) are tracked as a checklist in `DEPLOY.md`.

## Consequences

- Hosting cost is a single small VM. The whole stack can be rebuilt on a new host from the compose file and a backup.
- Vertical scaling only, which is acceptable at current scale. The API is stateless, so moving to multiple instances behind a load balancer needs no code changes. Rate limiting already lives in the shared database.
- Migrations run as a separate step before the API starts, and CI fails on entity/migration drift, so schema changes are explicit and reviewable.
