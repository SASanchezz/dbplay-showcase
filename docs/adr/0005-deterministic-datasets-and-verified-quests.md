# ADR-0005: Deterministic datasets and machine-verified quests

- **Status:** Accepted
- **Date:** 2026-09

## Context

With 237 built-in quests plus community-authored ones, a quest that is unsolvable, or that accepts a wrong answer, silently breaks the game. Cost ceilings only work if planner costs are reproducible.

## Decision

- **Deterministic data:** datasets are generated with arithmetic over `generate_series`, never `random()`, and analysed in the same way every time. Costs are reproducible across machines.
- **Quest families:** a technique family (lookup, covering, top-N, partial, expression, full-text, BRIN, keyset, lateral, workload tuning, …) fixes the teaching text and the win conditions. A per-dataset spec supplies the story, the query, a reference solution and plausible **near misses**.
- **Verifier** (`quests:verify`): on real Postgres, as `quest_runner`, under production limits, it proves for every quest that
  1. the untouched quest **fails**,
  2. the reference solution **passes with full stars**,
  3. every near miss **fails**.

  It prints costs and write multipliers so thresholds can be set with at least ~25% margin on both sides. If right and wrong answers cost the same, the quest is redesigned or dropped.
- **Browser verifier** (`quests:verify:browser`): the same checks through PGlite.
- **Community quests:** publishing requires a solution SQL that the server grades against the saved quest. Editing any grading field sends the quest back to draft until it is re-verified.

## Consequences

- Every published quest is provably solvable and provably rejects the obvious wrong answers.
- Reference solutions and near misses live in code only and are never written to the database.
- Calibration is an explicit, repeatable step rather than guesswork.
