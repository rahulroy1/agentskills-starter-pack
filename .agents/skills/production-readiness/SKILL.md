---
name: production-readiness
description: >
  Guides production-impacting changes: cross-service coordination, observability
  (metrics, alerting, logging), and infrastructure modifications (IaC, databases,
  networking). Use for Tier 3 changes requiring multi-service deployment ordering,
  production monitoring, or infrastructure provisioning.
activation-triggers:
  - "Tier 3 change requiring multiple services to update"
  - "Change affecting production behavior that needs monitoring"
  - "Infrastructure change (DB, queues, networking, IaC)"
related-skills: [deployment-strategy, failure-analysis, data-migration]
---

# Production Readiness

## Related Skills
- **deployment-strategy:** Rollout strategy and rollback triggers.
- **failure-analysis:** Intermediate state safety and partial failure scenarios.
- **data-migration:** Database schema changes and backfill strategies.
- **api-contracts:** Contract versioning when multiple services consume an API.

---

## Cross-Service Coordination

### Checklist
- [ ] All consuming services identified (contract dependency graph mapped)
- [ ] Deployment order defined (which ships first, second, etc.)
- [ ] Intermediate compatibility verified (each step is a valid system state)
- [ ] Contract versioning strategy chosen
- [ ] Feature flags configured for progressive rollout
- [ ] Rollback procedure defined per service independently
- [ ] Communication plan for service owners
- [ ] End-to-end integration test covering transition path

### Patterns

**Expand-Contract (safest)**
1. **Expand:** Add new fields/endpoints alongside old. All services accept both.
2. **Migrate:** Update consumers one by one.
3. **Contract:** Remove old once all consumers migrated.

Each step independently deployable and rollback-safe.

**Deployment ordering**
- **Producer first:** Ship provider of new contract first (backward compatible). Then consumers.
- **Consumer first:** Only if new consumer handles both old and new producer.
- **Simultaneous:** Only with atomic deployment (rare). Avoid.

**Contract negotiation**
- Producer advertises supported versions
- Consumer requests preferred version
- Fallback to lowest common version
- Sunset old versions with monitoring + timeline

### Anti-Patterns
- **Assuming simultaneous deploy** — design for mixed-version operation.
- **Breaking changes without expand-contract** — removing a field consumers read is an outage.
- **No intermediate state testing** — test the rollout, not just the end state.
- **Tight temporal coupling** — design for hours/days of mixed versions, not minutes.
- **Shared database across services** — use APIs.

---

## Observability

### Checklist
- [ ] Key metrics identified (what numbers prove this works?)
- [ ] Baseline captured before deployment
- [ ] Alerting thresholds defined
- [ ] Dashboard updated or created
- [ ] Structured logging at key decision points (targeted, not verbose)
- [ ] Correlation IDs for request tracing across services
- [ ] Runbook updated (what to do when alert fires)

### Patterns

**Four signals** — for any service change, monitor:
1. **Rate** — requests per second (traffic flowing?)
2. **Errors** — error rate and types (breaking?)
3. **Duration** — P50/P95/P99 latency (slow?)
4. **Saturation** — CPU, memory, connections, queue depth (overwhelmed?)

**What to log at decision points**
- Input received (sanitized — no PII/secrets)
- Decision made and why
- External call: latency, status
- Output produced

**Alerting**
- Alert on symptoms (error rate, latency), not causes
- Thresholds from baselines, not guesses
- Runbook link in alert message
- Page for user-facing impact only

### Anti-Patterns
- **Logging everything** — noise, cost, harder to find issues.
- **No baseline** — "errors went up" means nothing without before.
- **Non-actionable alerts** — if response is "ignore it," alert is wrong.
- **Dashboard nobody checks** — integrate into deploy workflow.
- **Logging PII** — structured logging doesn't exempt from security rules.

---

## Infrastructure Changes

### Checklist
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

### Patterns

**Environment promotion**
1. Apply to dev → validate
2. Apply to staging → validate with production-like traffic
3. Apply to production → monitor closely
4. Never skip staging for infra changes

**Database changes**
- Use migration tools (Flyway, Alembic, Knex, etc.)
- Additive first (add column → backfill → enforce)
- Never DROP in same deploy as code change
- Test on production-sized dataset

**IaC workflow**
1. `plan` / `diff` — review what changes
2. Peer review plan output
3. `apply` to lower environment → validate
4. `apply` to production → confirm with monitoring

### Anti-Patterns
- **Manual infra changes** — undocumented, unreproducible, un-rollbackable.
- **Apply directly to prod** — always go through lower environments.
- **Ignoring plan output** — "it'll destroy 3 resources but probably fine."
- **Shared resources without coordination** — affects all consumers.
- **No cost estimation** — small config changes can 10x your bill.

---

## Verification

```bash
# Cross-service: verify deployment order
curl -s "https://metrics/api/versions?service=all" | jq '.[] | {service, version}'

# Observability: compare baseline vs current
curl -s "https://metrics/api/error_rate?service=api&window=1h" | jq '.rate'

# Infrastructure: review IaC plan before apply
terraform plan -out=plan.tfplan && terraform show plan.tfplan
```
