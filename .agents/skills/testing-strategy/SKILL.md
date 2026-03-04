---
name: testing-strategy
description: >
  Guides test coverage, test quality, and edge case identification. Use during Tier 2/3
  test coverage review lens, when writing specs that define test strategy, or when
  adding tests to code with no existing coverage.
---

# Testing Strategy

## Checklist

### Coverage
- [ ] Happy path tested
- [ ] Edge cases: empty input, null, max/min, boundary conditions
- [ ] Error paths: invalid input, missing resources, timeouts, permission denied
- [ ] Integration points: API calls, DB queries, file I/O, external services
- [ ] Regression test for every bug fix (fails on old code, passes on new)

### Quality
- [ ] Tests independent — no shared mutable state, no order dependency
- [ ] Tests deterministic — same result every run
- [ ] Tests fast — unit <100ms, integration <5s
- [ ] Test names describe scenario and expected outcome
- [ ] Tests verify behavior, not implementation
- [ ] Each test asserts one concept

### Structure
- [ ] Arrange-Act-Assert pattern
- [ ] Test data explicit in the test — no hidden fixtures
- [ ] Mocks only at boundaries (external services, I/O)

## Patterns

### Edge cases to always consider
- Empty string, empty list, empty object
- Null / None / undefined
- Zero, negative, MAX_INT
- Single element vs many
- Unicode, special characters, very long strings
- Concurrent access (if applicable)
- Time zones, DST (if date-related)

### Mock vs test directly
- **Mock:** External APIs, databases, file systems, network, clocks
- **Don't mock:** Business logic, data transformations, validation, pure functions

### When tests don't exist
- Tier 1: Add at least one targeted test. If not feasible, document why.
- Tier 2/3: Tests required. No exceptions.

### Trace verification
When outputs derive from sources, test the traceability:
- Every output value should trace back to a source input with evidence
- Test that derived artifacts are consistent with each other (if you produce two representations of the same data, they must agree)
- Regression diff: compare before/after with explicit change counts — not just "tests pass"

## Anti-Patterns

- **Testing implementation** — test outputs, not method calls.
- **Giant fixtures** — 50 lines setup, 1 line assertion → break it down.
- **Flaky tests ignored** — a flaky test is a bug. Fix or remove.
- **Coverage as a goal** — 100% bad tests < 60% good tests.
- **Shape-only validation** — verifying output structure without checking that values trace to their source. Structurally correct but semantically wrong is the hardest bug to find.
