---
name: deployment-strategy
description: >
  Guides deployment planning including blue-green, canary, rolling, and feature flag
  strategies. Use for Tier 3 changes that cannot be deployed atomically, carry production
  risk, or require coordinated rollout across environments.
---

# Deployment Strategy

## Related Skills
- **observability:** Required for monitoring during rollout.
- **failure-analysis:** For partial rollout and rollback scenarios.

## Checklist

- [ ] Deployment strategy chosen and documented in spec
- [ ] Rollback trigger defined (what condition triggers rollback?)
- [ ] Rollback procedure tested
- [ ] Health checks defined
- [ ] Monitoring dashboard identified or created
- [ ] Canary/rollout percentage planned (if gradual)
- [ ] Feature flag configured (if progressive delivery)
- [ ] Environment promotion order defined (dev → staging → prod)
- [ ] Blast radius per environment assessed

## Patterns

### Blue-Green
Two identical environments. Deploy to inactive, verify, switch traffic.
- **Use when:** Zero-downtime required, full environment testing needed.
- **Rollback:** Switch traffic back.

### Canary
Route small percentage of traffic to new version. Increase gradually.
- **Use when:** Validate in production with limited risk.
- **Key metric:** Error rate, latency, business metrics at each percentage.

### Rolling
Update instances incrementally. Each batch verified before next.
- **Use when:** Stateless services with health checks.

### Feature Flags
Deploy code everywhere, activate via flag for specific users/percentages.
- **Use when:** Decoupling deploy from release, gradual rollout.
- **Rollback:** Disable flag. Instant.

## Verification

```bash
# Check canary error rate (example)
curl -s "https://metrics/api/error_rate?service=api&version=canary" | jq '.rate'

# Compare baseline vs canary metrics
./scripts/compare_metrics.sh baseline canary

# Health check
curl -f https://api/health || echo "Health check failed"
```

### Rollback triggers
- Error rate > X% above baseline
- P99 latency > Yms
- Business metric drops by Z%
- Health check failures > N within M minutes

## Anti-Patterns

- **"We'll just redeploy"** — if rollback isn't tested, it doesn't work.
- **Friday 5pm deploy** — deploy when team is available to monitor.
- **No canary for breaking changes** — 0% to 100% is gambling.
- **Feature flags left forever** — set cleanup date when creating.
- **Skipping staging** — promote through environments.
