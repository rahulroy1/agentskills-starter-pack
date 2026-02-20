---
name: data-migration
description: >
  Guides database schema changes, data format changes, storage restructuring, and
  backfill strategies. Use for Tier 3 changes involving schema changes, data
  transformations, or storage migrations that require rollback-for-data planning.
---

# Data Migration

## Related Skills
- **failure-analysis:** Essential for intermediate state safety.
- **production-readiness:** Required for migration monitoring and infrastructure changes.

## Checklist

### Planning
- [ ] Current and new schema documented (before/after diff)
- [ ] Volume assessed (data size, migration duration estimate)
- [ ] Migration script written and version-controlled
- [ ] Dry run executed in non-production environment
- [ ] Rollback-for-data plan defined (not just rollback-for-code)
- [ ] Backfill strategy defined for existing records

### Execution
- [ ] Backup taken before migration
- [ ] Migration runs idempotently (safe to re-run)
- [ ] Progress tracking in place
- [ ] Data validation after migration (row counts, checksums, spot checks)
- [ ] Application compatible with both schemas during migration window
- [ ] Zero or minimal downtime (online migration preferred)

### Post-Migration
- [ ] Data integrity verified (no orphans, corruption, missing records)
- [ ] Old schema artifacts cleaned up
- [ ] Compatibility shims removed from code
- [ ] Monitoring confirms normal operation

## Patterns

### Online migration (zero downtime)
1. **Expand:** Add new columns/tables alongside old. Write to both.
2. **Backfill:** Migrate existing data old → new.
3. **Validate:** Checksums, counts confirm match.
4. **Switch:** Read from new, stop writing to old.
5. **Contract:** Remove old after confirmation period.

### Rollback-for-data
- Keep old columns/tables during transition
- Write reverse migration script if data was transformed
- Track what data was written by new vs old code
- Point-in-time backup for catastrophic scenarios

### Idempotent migrations
- Upserts instead of blind inserts
- Track progress (last processed ID, checkpoint)
- Re-running produces same result

## Verification

```bash
# Check migration status (example: PostgreSQL)
psql -c "SELECT * FROM schema_migrations ORDER BY applied_at DESC LIMIT 5;"

# Verify row counts match
psql -c "SELECT COUNT(*) FROM old_table;" 
psql -c "SELECT COUNT(*) FROM new_table;"

# Dry-run migration script
python scripts/migrate_data.py --dry-run --limit 100
```

## Anti-Patterns

- **DROP COLUMN in same deploy** — old code can't find it on rollback.
- **Single transaction** — long locks cause outages. Batch it.
- **No dry run** — 10 rows local ≠ 10M rows production.
- **No data validation** — always verify counts and checksums.
- **Assuming fast** — estimate from volume. Plan for 10x longer.
