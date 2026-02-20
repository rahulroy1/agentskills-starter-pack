# Review: AGENTS.md and Skills - Improvement Recommendations

## Executive Summary

The framework is well-structured with clear risk tiers, spec-driven workflow, and modular skills. The following improvements address gaps in clarity, consistency, discoverability, and edge cases.

---

## 1. Structural & Organizational Improvements

### 1.1 AGENTS.md: Skill Discovery Mechanism

**Issue:** AGENTS.md states "the agent's skill discovery system handles activation" but doesn't specify:
- How agents discover skills (file system scan? registry? explicit activation?)
- When to activate (before spec? during exploration? during review?)
- What happens if a skill is missing or malformed

**Recommendation:** Add explicit skill activation protocol:

```markdown
## Skill Activation Protocol

1. **Discovery:** Agent scans `.agents/skills/*/SKILL.md` at session start
2. **Metadata Load:** Read frontmatter (name, description) only — ~50 tokens per skill
3. **Activation Triggers:**
   - Tier 3 exploration: Identify applicable skills from §3 table
   - Review phase: Activate per lens (§6)
   - Explicit reference: When AGENTS.md mentions "→ skill: `name`"
4. **Full Load:** Read complete SKILL.md only when activated
5. **Missing Skill:** Log warning, proceed with AGENTS.md guidance only
```

### 1.2 AGENTS.md: Missing Architecture Lens Guidance

**Issue:** §6 lists "Architecture" as review lens #7 but provides no guidance on what to review.

**Recommendation:** Add architecture review criteria:

```markdown
### Architecture Review Lens
- [ ] Change aligns with system architecture (monolith/modulith/microservices)
- [ ] No new coupling introduced (circular dependencies, shared state)
- [ ] Boundaries respected (module/service boundaries, data ownership)
- [ ] Scalability impact assessed (bottlenecks, resource usage)
- [ ] Technology choices justified (why this library/framework/pattern)
```

### 1.3 Skills: Standardize Structure

**Issue:** Skills are consistent but could benefit from explicit sections.

**Recommendation:** Enforce standard structure in skill template:

```markdown
---
name: skill-name
description: One-line when to use
activation-triggers: [list of explicit triggers]
related-skills: [cross-references]
---

# Skill Title

## When to Use
[Explicit conditions beyond description]

## Checklist
[Actionable items]

## Patterns
[Concrete examples]

## Anti-Patterns
[What to avoid]

## Integration Points
[How this skill interacts with others, e.g., "Use with deployment-strategy for rollback"]
```

---

## 2. Content Gaps & Enhancements

### 2.1 AGENTS.md: Tier Classification Edge Cases

**Issue:** Risk tier table doesn't handle ambiguous cases:
- Adding a new optional field to an API (breaking? non-breaking?)
- Changing internal behavior that affects external output (Tier 2 or 3?)
- Refactoring that changes error messages (Tier 1 or 2?)

**Recommendation:** Add decision tree with examples:

```markdown
### Tier Classification Decision Tree

**Q1: Does this change observable behavior outside the repo?**
- YES → Tier 3 (even if "internal" refactor affects output)
- NO → Continue

**Q2: Does this change a contract (API, schema, CLI output)?**
- YES → Tier 3
- NO → Continue

**Q3: Does this affect multiple modules/components?**
- YES → Tier 2
- NO → Tier 1

### Edge Case Examples

| Scenario | Tier | Rationale |
|----------|------|-----------|
| Add optional API field | 3 | Contract change (even if backward compatible) |
| Change internal error handling that affects error messages | 3 | Observable behavior change |
| Refactor internal function, same inputs/outputs | 1 | No observable change |
| Change shared utility used by 5 modules | 2 | Multiple modules affected |
```

### 2.2 AGENTS.md: Spec Approval Mechanism

**Issue:** §3 says "get explicit user approval" for Tier 3 but doesn't specify:
- Format of approval request
- What constitutes approval (explicit "yes"? implicit continuation?)
- How to handle partial approval (approve approach but not implementation?)

**Recommendation:** Add approval protocol:

```markdown
### Tier 3 Spec Approval Protocol

1. **Present Spec:** Format as markdown with clear sections (§3)
2. **Request Approval:** Explicit question: "Approve this spec? (yes/no/modify)"
3. **Wait for Response:** Do not proceed without explicit approval
4. **Partial Approval:** If user approves approach but requests changes:
   - Update spec → re-present → re-approve
5. **Approval Record:** Log approval in `tasks/todo.md` with timestamp
```

### 2.3 Skills: Missing Cross-References

**Issue:** Skills don't reference each other, leading to:
- Missing `observability` when using `deployment-strategy`
- Missing `failure-analysis` when using `data-migration`
- Missing `api-contracts` when using `migration-deprecation`

**Recommendation:** Add "Related Skills" section to each skill:

```markdown
## Related Skills

- **deployment-strategy:** Use together for production rollouts
- **observability:** Required for monitoring migration progress
- **failure-analysis:** Essential for multi-step migrations
```

### 2.4 Skills: Missing Verification Patterns

**Issue:** Skills provide checklists but not verification commands/examples.

**Recommendation:** Add "Verification" section to applicable skills:

```markdown
## Verification

### How to verify this skill's checklist items:

**Migration progress:**
```bash
# Check migration status
psql -c "SELECT COUNT(*) FROM schema_migrations WHERE applied_at IS NULL;"

# Verify data integrity
python scripts/verify_migration.py --dry-run
```

**Deployment health:**
```bash
# Check canary error rate
curl -s "https://metrics/api/error_rate?service=api&version=canary" | jq '.rate'

# Compare baseline vs canary
./scripts/compare_metrics.sh baseline canary
```
```

---

## 3. Consistency Issues

### 3.1 AGENTS.md: Inconsistent Skill References

**Issue:** Some sections reference skills with `→ skill: name`, others just mention names.

**Recommendation:** Standardize format:

```markdown
# Current (inconsistent):
- Security and safety → skill: `security-baseline`
- Migration/deprecation → skill: `migration-deprecation`
- Use skill: `api-contracts` for...

# Standardized:
- Security and safety → activate: `security-baseline`
- Migration/deprecation → activate: `migration-deprecation`
- Use `api-contracts` skill for... → activate: `api-contracts`
```

### 3.2 Skills: Inconsistent Checklist Format

**Issue:** Some skills use `- [ ]`, others use `- [ ]` with sub-bullets inconsistently.

**Recommendation:** Standardize checklist hierarchy:

```markdown
## Checklist

### Category Name
- [ ] Top-level item
  - [ ] Sub-item (indented 2 spaces)
  - [ ] Sub-item with code example:
    ```python
    # Example
    ```
```

### 3.3 AGENTS.md vs AGENTS-single.md: Version Drift

**Issue:** AGENTS-single.md (v6) has some content not in AGENTS.md (v8):
- More detailed security baseline (§10 in single vs reference in v8)
- Different review lens count (6 vs 7)

**Recommendation:** 
- Document migration path from v6 → v8
- Ensure all v6 content is either in v8 or delegated to skills
- Add version compatibility notes

---

## 4. Clarity & Usability

### 4.1 AGENTS.md: Subagent Naming Convention

**Issue:** §7 says "Name descriptively" but doesn't provide examples.

**Recommendation:** Add naming convention:

```markdown
### Subagent Naming Convention

Format: `[phase]-[scope]-[task]`

Examples:
- `explore-api-contracts-consumers`
- `implement-user-service-layer`
- `review-security-authn-authz`
- `validate-migration-dry-run`

Avoid:
- Generic: `explore`, `implement`, `review`
- Vague: `check-things`, `fix-stuff`
```

### 4.2 Skills: Missing Prerequisites

**Issue:** Skills don't specify prerequisites (e.g., `data-migration` requires `failure-analysis` understanding).

**Recommendation:** Add prerequisites section:

```markdown
## Prerequisites

Before using this skill, ensure:
- [ ] `failure-analysis` skill activated (for intermediate state safety)
- [ ] `observability` skill activated (for migration monitoring)
- [ ] Backup strategy defined (see `infrastructure-changes` skill)
```

### 4.3 AGENTS.md: Missing Quick Reference

**Issue:** No quick lookup table for common scenarios.

**Recommendation:** Add quick reference appendix:

```markdown
## Appendix: Quick Reference

### Common Scenarios

| Task | Tier | Required Skills | Spec Approval? |
|------|------|----------------|----------------|
| Fix typo in error message | 1 | security-baseline | No |
| Add new API endpoint | 3 | api-contracts, security-baseline | Yes |
| Refactor shared utility | 2 | code-quality | No |
| Database schema change | 3 | data-migration, failure-analysis | Yes |
| Deploy to production | 3 | deployment-strategy, observability | Yes |
```

---

## 5. Integration & Discovery

### 5.1 Skill Metadata: Missing Activation Triggers

**Issue:** Skill frontmatter doesn't specify when to activate.

**Recommendation:** Add `activation-triggers` to frontmatter:

```yaml
---
name: api-contracts
description: Guides API contract versioning...
activation-triggers:
  - "Tier 3 change modifying public API"
  - "Review lens: Architecture (when API changes)"
  - "Explicit mention in spec: 'API contract change'"
related-skills: [migration-deprecation, cross-service-coordination]
---
```

### 5.2 AGENTS.md: Skill Activation Table Enhancement

**Issue:** §3 table lists skills but doesn't specify activation timing.

**Recommendation:** Add activation phase:

```markdown
| Concern | When to include | Skill | Activation Phase |
|---------|----------------|-------|------------------|
| Migration / deprecation | Changing a contract with active consumers | `migration-deprecation` | During exploration |
| Data migration | Schema, format, or storage changes | `data-migration` | During exploration |
| Cross-service coordination | Multiple services must update | `cross-service-coordination` | During exploration |
| Deployment strategy | Can't deploy atomically or carries prod risk | `deployment-strategy` | During spec writing |
| Backward compatibility | Consumers can't all update simultaneously | `api-contracts` | During exploration |
| Infrastructure | DB, queues, networking, config changes | `infrastructure-changes` | During exploration |
| Observability | Production behavior needs monitoring | `observability` | During spec writing |
| Failure mode analysis | Partial rollout, async, distributed changes | `failure-analysis` | During exploration |
```

---

## 6. Missing Capabilities

### 6.1 Missing Skill: Performance Analysis

**Issue:** No skill for performance-critical changes (optimizations, scaling, bottlenecks).

**Recommendation:** Create `performance-analysis` skill:

```markdown
---
name: performance-analysis
description: >
  Guides performance optimization, bottleneck identification, and scaling strategies.
  Use for Tier 2/3 changes affecting latency, throughput, or resource usage.
---

# Performance Analysis

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
- CPU: Use profilers (py-spy, perf, pprof)
- Memory: Heap dumps, allocation tracking
- I/O: Database query analysis, network latency
- End-to-end: Distributed tracing (Jaeger, Zipkin)

### Optimization Strategies
- Caching: Add where reads >> writes
- Batching: Reduce round-trips
- Indexing: Database query optimization
- Connection pooling: Reuse connections
- Async processing: Move work off critical path

## Anti-Patterns
- **Optimizing without profiling** — guesswork wastes time
- **Premature optimization** — measure first
- **Ignoring production data** — local != production
- **No baseline** — can't measure improvement
```

### 6.2 Missing Skill: Documentation Standards

**Issue:** No skill for documentation requirements (API docs, README, inline comments).

**Recommendation:** Create `documentation-standards` skill:

```markdown
---
name: documentation-standards
description: >
  Guides documentation requirements for code, APIs, and architecture. Use during
  Tier 2/3 documentation review lens or when adding new public interfaces.
---

# Documentation Standards

## Checklist
- [ ] Public API documented (parameters, return types, errors)
- [ ] README updated for user-facing changes
- [ ] Architecture decisions documented (ADR format)
- [ ] Inline comments explain *why*, not *what*
- [ ] Examples provided for complex APIs
- [ ] Breaking changes documented in CHANGELOG
- [ ] Migration guides for contract changes

## Patterns
### API Documentation
- OpenAPI/Swagger for REST APIs
- JSDoc/TSDoc for TypeScript/JavaScript
- Docstrings for Python (Google/NumPy style)

### Architecture Documentation
- ADR (Architecture Decision Records) for significant decisions
- Sequence diagrams for complex flows
- Data flow diagrams for ETL/pipelines

## Anti-Patterns
- **"Code is self-documenting"** — complex logic needs explanation
- **Outdated docs** — update in same changeset
- **Missing examples** — docs without examples are incomplete
```

### 6.3 Missing: Error Recovery Escalation Protocol

**Issue:** §9 mentions escalation but doesn't specify format.

**Recommendation:** Add escalation template:

```markdown
### Escalation Template

When escalating after 3 failed attempts:

```markdown
## Escalation: [Task Name]

**Attempts:** 3
**Approach 1:** [What you tried, why it failed]
**Approach 2:** [What you tried, why it failed]
**Approach 3:** [What you tried, why it failed]

**Current State:** [What's working, what's not]
**Hypothesis:** [Your best guess at root cause]
**Blockers:** [What's preventing progress]
**Requested Help:** [Specific question or permission needed]
```
```

---

## 7. Specific Skill Improvements

### 7.1 security-baseline: Add Threat Modeling

**Recommendation:** Add threat modeling section:

```markdown
## Threat Modeling (Tier 3)

For Tier 3 changes, consider:
- [ ] Attack surface increased? (new endpoints, file uploads, external integrations)
- [ ] Data flow mapped? (where does sensitive data go?)
- [ ] Trust boundaries identified? (what can attackers control?)
- [ ] Mitigations documented? (how are threats addressed?)
```

### 7.2 testing-strategy: Add Property-Based Testing

**Recommendation:** Add property-based testing patterns:

```markdown
### Property-Based Testing

For complex logic, consider property-based tests:
- **Invariants:** Properties that always hold (e.g., "sort is idempotent")
- **Round-trips:** Encode → decode returns original
- **Relationships:** Function composition properties

Tools: Hypothesis (Python), QuickCheck (Haskell), jqwik (Java)
```

### 7.3 failure-analysis: Add Chaos Engineering

**Recommendation:** Add chaos testing section:

```markdown
### Chaos Testing (Tier 3, Production)

For distributed systems:
- [ ] Failure injection tested (network partitions, service failures)
- [ ] Retry behavior validated (does retry make it worse?)
- [ ] Circuit breakers tested (do they prevent cascading failures?)
- [ ] Partial failures handled (some instances down, others up)

Tools: Chaos Monkey, Litmus, Gremlin
```

---

## 8. Implementation Priority

### High Priority (Immediate)
1. Add skill activation protocol (§1.1)
2. Add architecture review lens guidance (§1.2)
3. Add Tier 3 spec approval protocol (§2.2)
4. Standardize skill references (§3.1)
5. Add quick reference table (§4.3)

### Medium Priority (Next Sprint)
6. Add decision tree for tier classification (§2.1)
7. Add cross-references to skills (§2.3)
8. Add verification patterns to skills (§2.4)
9. Create performance-analysis skill (§6.1)
10. Add subagent naming convention (§4.1)

### Low Priority (Backlog)
11. Create documentation-standards skill (§6.2)
12. Add threat modeling to security-baseline (§7.1)
13. Add property-based testing to testing-strategy (§7.2)
14. Add chaos engineering to failure-analysis (§7.3)
15. Resolve version drift between v6 and v8 (§3.3)

---

## 9. Testing the Framework

### Recommended Validation

1. **New Agent Test:** Give a fresh agent AGENTS.md + skills, ask it to:
   - Classify a Tier 3 API change
   - Write a spec with applicable skills
   - Execute the workflow

2. **Skill Discovery Test:** Verify agents can:
   - Discover all skills
   - Activate correct skills for given scenarios
   - Handle missing skills gracefully

3. **Edge Case Test:** Present ambiguous scenarios:
   - "Add optional field to API" (Tier 2 or 3?)
   - "Refactor that changes error messages" (Tier 1 or 3?)
   - "Change that affects observability but not functionality" (Tier?)

---

## 10. Documentation Improvements

### 10.1 Add Skill Authoring Guide

Create `.agents/skills/README.md`:

```markdown
# Agent Skills Authoring Guide

## Structure
- Frontmatter: name, description, activation-triggers, related-skills
- When to Use: Explicit conditions
- Checklist: Actionable items
- Patterns: Concrete examples
- Anti-Patterns: What to avoid
- Integration Points: How this skill interacts with others

## Naming Convention
- kebab-case
- Descriptive (not generic)
- Matches folder name

## Testing
- Verify activation triggers are clear
- Check cross-references are accurate
- Ensure examples are runnable
```

### 10.2 Add Migration Guide

Create `MIGRATION-v6-to-v8.md`:

```markdown
# Migration Guide: v6 (Single File) → v8 (Modular)

## What Changed
- Security baseline moved to `security-baseline` skill
- Review lenses: 6 → 7 (added Architecture)
- Skill activation: Explicit protocol added

## Migration Steps
1. Copy AGENTS.md (v8) to your repo
2. Copy `.agents/skills/` directory
3. Update any custom rules to reference skills
4. Test with a Tier 3 change
```

---

## Summary

The framework is solid but would benefit from:
1. **Explicit protocols** (skill activation, spec approval, escalation)
2. **Better edge case handling** (tier classification, ambiguous scenarios)
3. **Cross-skill integration** (references, prerequisites, related skills)
4. **Missing capabilities** (performance, documentation standards)
5. **Consistency improvements** (naming, formatting, references)

Most improvements are additive and don't break existing functionality. Priority should be on clarity and discoverability to reduce agent confusion and improve adherence to the framework.
