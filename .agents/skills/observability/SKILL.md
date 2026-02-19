---
name: observability
description: >
  Guides metrics, alerting, logging, and dashboard setup for production changes. Use
  for Tier 3 changes affecting production behavior where tests alone cannot confirm
  the change is working correctly.
---

# Observability

## Related Skills
- **deployment-strategy:** Monitoring during rollout.
- **failure-analysis:** Detecting failure modes.
- **data-migration:** Tracking migration progress.

## Checklist

- [ ] Key metrics identified (what numbers prove this works?)
- [ ] Baseline captured before deployment
- [ ] Alerting thresholds defined
- [ ] Dashboard updated or created
- [ ] Structured logging at key decision points (targeted, not verbose)
- [ ] Correlation IDs for request tracing across services
- [ ] Runbook updated (what to do when alert fires)

## Patterns

### Four signals
For any service change, monitor:
1. **Rate** — requests per second (traffic flowing?)
2. **Errors** — error rate and types (breaking?)
3. **Duration** — P50/P95/P99 latency (slow?)
4. **Saturation** — CPU, memory, connections, queue depth (overwhelmed?)

### What to log at decision points
- Input received (sanitized — no PII/secrets)
- Decision made and why
- External call: latency, status
- Output produced

### Alerting
- Alert on symptoms (error rate, latency), not causes
- Thresholds from baselines, not guesses
- Runbook link in alert message
- Page for user-facing impact only

## Anti-Patterns

- **Logging everything** — noise, cost, harder to find issues.
- **No baseline** — "errors went up" means nothing without before.
- **Non-actionable alerts** — if response is "ignore it," alert is wrong.
- **Dashboard nobody checks** — integrate into deploy workflow.
- **Logging PII** — structured logging doesn't exempt from security rules.
