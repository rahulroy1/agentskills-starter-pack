---
name: cross-service-coordination
description: >
  Guides coordinated changes across multiple services including deployment ordering,
  contract versioning, and intermediate state safety. Use for Tier 3 changes requiring
  multiple services to update, shared schema evolution, or cross-service data migration.
---

# Cross-Service Coordination

## Related Skills
- **api-contracts:** Contract versioning strategy.
- **deployment-strategy:** Deployment ordering and rollback.
- **failure-analysis:** Intermediate state safety.

## Checklist

- [ ] All consuming services identified (contract dependency graph mapped)
- [ ] Deployment order defined (which ships first, second, etc.)
- [ ] Intermediate compatibility verified (each step is a valid system state)
- [ ] Contract versioning strategy chosen
- [ ] Feature flags configured for progressive rollout
- [ ] Rollback procedure per service defined independently
- [ ] Communication plan for service owners
- [ ] End-to-end integration test covering transition path
- [ ] Monitoring for cross-service health during rollout

## Patterns

### Expand-Contract (safest)
1. **Expand:** Add new fields/endpoints alongside old. All services accept both.
2. **Migrate:** Update consumers one by one.
3. **Contract:** Remove old once all consumers migrated.

Each step independently deployable and rollback-safe.

### Deployment ordering
- **Producer first:** Ship provider of new contract first (backward compatible). Then consumers.
- **Consumer first:** Only if new consumer handles both old and new producer.
- **Simultaneous:** Only with atomic deployment (rare). Avoid.

### Contract negotiation
- Producer advertises supported versions
- Consumer requests preferred version
- Fallback to lowest common version
- Sunset old versions with monitoring + timeline

## Anti-Patterns

- **Assuming simultaneous deploy** — design for mixed-version operation.
- **Breaking changes without expand-contract** — removing a field consumers read is an outage.
- **No intermediate state testing** — test the rollout, not just the end state.
- **Tight temporal coupling** — design for hours/days of mixed versions, not minutes.
- **Shared database across services** — use APIs.
