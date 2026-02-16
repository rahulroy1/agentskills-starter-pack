---
name: infrastructure-changes
description: >
  Guides infrastructure changes including database, queue, networking, and IaC
  modifications. Use for Tier 3 changes involving database schema, connection pools,
  messaging infrastructure, DNS, load balancers, or Terraform/CloudFormation/Pulumi.
---

# Infrastructure Changes

## Checklist

- [ ] Change documented as IaC (not manual)
- [ ] Environment promotion order defined (dev → staging → prod)
- [ ] Blast radius per environment assessed
- [ ] Rollback procedure defined and tested
- [ ] Resource limits and quotas checked
- [ ] Cost impact estimated
- [ ] Security review of new resources (public access, IAM, network)
- [ ] Monitoring/alerting for new resources configured
- [ ] DNS TTL considered for networking changes
- [ ] Dry run / plan output reviewed before apply

## Patterns

### Environment promotion
1. Apply to dev → validate
2. Apply to staging → validate with production-like traffic
3. Apply to production → monitor closely
4. Never skip staging for infra changes

### Database changes
- Use migration tools (Flyway, Alembic, Knex, etc.)
- Additive first (add column → backfill → enforce)
- Never DROP in same deploy as code change
- Test on production-sized dataset

### IaC workflow
1. `plan` / `diff` — review what changes
2. Peer review plan output
3. `apply` to lower environment → validate
4. `apply` to production → confirm with monitoring

## Anti-Patterns

- **Manual infra changes** — undocumented, unreproducible, un-rollbackable.
- **Apply directly to prod** — always go through lower environments.
- **Ignoring plan output** — "it'll destroy 3 resources but probably fine."
- **Shared resources without coordination** — affects all consumers.
- **No cost estimation** — small config changes can 10x your bill.
