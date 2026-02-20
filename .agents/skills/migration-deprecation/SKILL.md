---
name: migration-deprecation
description: >
  Guides version migrations and contract deprecation across runtime, tests, docs, and
  consumers. Use for Tier 3 changes that modify contracts with active consumers, version
  migrations (v1 to v2), or feature deprecation.
---

# Migration & Deprecation

## Related Skills
- **deployment-strategy:** Use together for production rollouts.
- **production-readiness:** Required for monitoring migration progress and cross-service coordination.
- **failure-analysis:** Essential for multi-step migrations.
- **api-contracts:** For contract versioning and sunset policies.

## Checklist

### Pre-Migration
- [ ] All consumers of current contract identified
- [ ] New contract fully specified (before/after diff)
- [ ] Backward compatibility window defined
- [ ] Consumer communication plan in place
- [ ] Migration tested in non-production environment

### Execution
- [ ] Same-change alignment: runtime, modules, tests, docs, contracts, commands, baselines
- [ ] Old and new paths coexist during transition (if needed)
- [ ] Feature flag or version negotiation controls rollout (if applicable)
- [ ] Monitoring confirms new path works before old is removed
- [ ] Rollback procedure tested

### Post-Migration
- [ ] Deprecated paths removed — no permanent dual contracts
- [ ] All docs reflect new path only
- [ ] Deprecation warnings removed from code
- [ ] Migration artifacts cleaned up

## Patterns

### Safe contract evolution
1. **Additive changes** — add new alongside old
2. **Migrate consumers** — update to new contract
3. **Verify** — confirm all migrated (logs, metrics)
4. **Remove old** — delete deprecated paths

### Deprecation timeline
1. Mark deprecated in code
2. Add deprecation warnings in responses/logs
3. Communicate sunset date
4. Monitor usage of deprecated path
5. Remove after sunset + grace period

## Anti-Patterns

- **Big bang migration** — always have a transition period.
- **Permanent dual contracts** — set sunset date and enforce.
- **Migration without monitoring** — measure before removing old.
- **Forgetting docs** — update in same changeset.
- **No rollback plan** — "we'll figure it out" is not a plan.
