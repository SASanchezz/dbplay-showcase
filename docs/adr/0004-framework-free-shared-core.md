# ADR-0004: A framework-free `quest-core` package shared by browser and server

- **Status:** Accepted
- **Date:** 2026-09

## Context

Plan parsing, constraint evaluation, scoring and levels are needed in the React app (instant feedback, "+N honor" previews) and in the NestJS API (authoritative grading). Duplicating the logic would guarantee drift between what the UI promises and what the server awards.

## Decision

Extract the domain logic into `packages/quest-core`:

- plain TypeScript with **zero runtime dependencies**, and no React, NestJS or `pg` imports
- `EXPLAIN` JSON parser and plan summariser
- an evaluator registry with one module per constraint kind
- grader, result comparison, write-cost model, scoring constants and level thresholds
- unit tests (vitest) with plan fixtures, run in CI

Both apps consume it through npm workspaces.

## Consequences

- One source of truth for rules. Changing a constant changes client and server together.
- New constraint kinds are added by registering one evaluator, without touching the apps.
- Pure functions over plan JSON are fast and trivial to unit test without a database.
- Scoring constants feed stored values (honor, level), so changing them needs a data migration. This is documented next to the constants.
