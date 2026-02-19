---
name: failure-analysis
description: >
  Analyzes failure modes, unsafe intermediate states, and partial rollout risks. Use
  for Tier 3 changes involving distributed systems, async operations, multi-step
  migrations, or any change where partial deployment creates inconsistent state.
---

# Failure Mode Analysis

## Related Skills
- **data-migration:** For multi-step data migrations.
- **deployment-strategy:** For partial rollout scenarios.
- **cross-service-coordination:** For distributed changes.

## Checklist

- [ ] All intermediate states identified (system states during rollout)
- [ ] Each intermediate state evaluated: safe or unsafe?
- [ ] Unsafe states documented with mitigation
- [ ] Partial failure scenarios enumerated (what if step 3 of 5 fails?)
- [ ] Rollback from each intermediate state possible and documented
- [ ] Timeout/retry behavior analyzed (does retry make it worse?)
- [ ] Data consistency at each step (no orphans, no partial writes)
- [ ] Monitoring detects each failure mode

## Patterns

### Intermediate state analysis
For each step of a multi-step change:
1. What is the system state after this step?
2. Is this state valid? Can the system operate here?
3. What happens if we stop here (crash, timeout)?
4. Can consumers handle this intermediate state?

### Failure categories
- **Crash:** Process dies mid-op. Resumable or must restart?
- **Timeout:** Does retry duplicate work? Is it idempotent?
- **Partial:** Some instances updated, others not. Mixed-version safe?
- **Data inconsistency:** Write succeeded in one store, failed in another.
- **Cascading:** One failure causes downstream failures. Circuit breakers?

### Safe intermediate states
Design so every step is valid:
- New code handles both old and new data formats
- Old code ignores new fields
- Missing data has sensible defaults
- No operation requires atomic cross-service coordination

### Pre-mortem questions
- "What if this ships to half the fleet?"
- "What if migration fails midway?"
- "What if we rollback after some data migrated?"
- "What if a dependency is down during deploy?"

## Anti-Patterns

- **"It won't fail"** — it will. Plan for it.
- **Non-idempotent without protection** — retries duplicate data.
- **Ignoring mixed-version** — both versions will run simultaneously.
- **Rollback ignoring data** — code rollback is easy; data rollback is not.
- **Testing only happy path** — failures are where incidents live.
