# Logic Examples

## 1. Everyday claim to deductive check

Input: "If it rains the match is cancelled. The match was cancelled, so it rained."

Analysis:

- P1: If it rains, the match is cancelled.
- P2: The match was cancelled.
- C: It rained.
- Form: affirming the consequent, invalid. Other causes such as field works could explain P2.
- Repair: add "Only rain cancels matches" or weaken C to "rain is one possible cause."

## 2. Technical claim to inductive check

Input: "The new cache made staging faster in three manual tests, so production will be 50 percent faster."

Analysis:

- P1: Three staging runs were faster with the cache.
- P2 `[implicit]`: Staging matches production load and data.
- C: Production will be 50 percent faster.
- Verdict: weak induction. Sample is small, selection is manual, P2 is unsupported.
- Repair: report N, median and spread, production-like load test, and state confidence as range plus next check.

## 3. Loose idea to article outline, only on request

Input: "I think code review should require two reviewers. Turn this into an article outline."

Outline:

1. Thesis: two reviewers for high-risk changes because one misses context-specific defects.
2. Support: P1 escaped defects cluster in areas with single reviewer, P2 second reviewer adds domain check, C defect escape rate drops.
3. Objection: slower throughput. Reply: scope the rule to high-risk paths, async second pass elsewhere.
4. Limits: small teams, trivial changes, emergencies.
5. Closing: restate rule plus rollout step.
