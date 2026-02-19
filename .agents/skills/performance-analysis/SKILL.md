---
name: performance-analysis
description: >
  Guides performance optimization, bottleneck identification, and scaling strategies.
  Use for Tier 2/3 changes affecting latency, throughput, or resource usage.
---

# Performance Analysis

## Related Skills
- **observability:** Baseline metrics and monitoring.
- **testing-strategy:** Performance regression tests.

## Checklist

- [ ] Baseline metrics captured (latency, throughput, resource usage)
- [ ] Bottleneck identified (CPU, memory, I/O, network, database)
- [ ] Optimization target defined (e.g., "reduce P95 latency by 50%")
- [ ] Profiling data collected (CPU profiles, memory dumps, trace logs)
- [ ] Change impact measured (before/after comparison)
- [ ] Regression tests for performance added
- [ ] Monitoring alerts for performance degradation configured

## Patterns

### Profiling
- **CPU:** py-spy, perf, pprof
- **Memory:** Heap dumps, allocation tracking
- **I/O:** Database query analysis, network latency
- **End-to-end:** Distributed tracing (Jaeger, Zipkin)

### Optimization Strategies
- **Caching:** Add where reads >> writes
- **Batching:** Reduce round-trips
- **Indexing:** Database query optimization
- **Connection pooling:** Reuse connections
- **Async processing:** Move work off critical path

## Verification

```bash
# Capture baseline (example: HTTP latency)
curl -w "@curl-format.txt" -o /dev/null -s https://api/endpoint

# Load test
ab -n 1000 -c 10 https://api/endpoint

# Profile (Python)
python -m cProfile -o profile.stats script.py
```

## Anti-Patterns

- **Optimizing without profiling** — guesswork wastes time.
- **Premature optimization** — measure first.
- **Ignoring production data** — local != production.
- **No baseline** — can't measure improvement.
