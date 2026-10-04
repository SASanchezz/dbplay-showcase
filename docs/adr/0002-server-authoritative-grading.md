# ADR-0002: Instant client grading, authoritative server grading

- **Status:** Accepted
- **Date:** 2026-09

## Context

Players want feedback in milliseconds while they iterate. Honor, levels and the leaderboard need a result nobody can forge. Grading only on the server adds latency and server load to every keystroke-level iteration. Grading only on the client means anyone can POST "3 stars".

## Decision

- The browser grades locally on PGlite for **instant, untrusted feedback**.
- **Submit for grading** sends only the SQL. The server re-runs it on real Postgres in a sandbox ([ADR-0003](0003-untrusted-sql-defence-in-depth.md)) and computes stars, cost and honor from its own run. The client never asserts a grade.
- Both sides use the **same grader and scoring code** from `quest-core` ([ADR-0004](0004-framework-free-shared-core.md)), so the client's "+N honor" preview matches the server's award.
- A browser-side verifier (`quests:verify:browser`) runs every quest through PGlite grading to keep the two engines in agreement.

## Consequences

- Cheating requires defeating the server sandbox, not editing a request.
- Server load scales with submissions, not with experimentation. Submissions are rate-limited (10 per minute) and the sandbox pool is capped.
- Two engines (PGlite and Postgres 16) can disagree on edge cases, which the dual verifier exists to catch.
- Once all stars on a quest are banked, further submissions return `409`, which removes pointless load.
