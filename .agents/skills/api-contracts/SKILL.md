---
name: api-contracts
description: >
  Guides API contract versioning, backward compatibility, and consumer-driven contract
  testing. Use for Tier 3 changes modifying public APIs, output schemas, or shared
  interfaces where consumers cannot all update simultaneously.
---

# API Contracts & Versioning

## Related Skills
- **migration-deprecation:** For contract changes with active consumers.
- **cross-service-coordination:** When multiple services consume the contract.

## Checklist

- [ ] Contract change documented (before/after diff)
- [ ] Change classified: additive (safe) vs breaking (requires migration)
- [ ] Consumer impact assessed
- [ ] Versioning strategy chosen and applied
- [ ] Backward compatibility preserved during transition
- [ ] Sunset timeline defined for old version
- [ ] Consumer-driven contract tests added (if applicable)
- [ ] Documentation updated (API docs, schemas, examples)

## Patterns

### Safe (non-breaking) changes
- Adding optional fields to responses
- Adding new endpoints
- Adding optional query parameters
- Widening accepted input types
- Adding new enum values (if consumers handle unknown)

### Breaking changes (require versioning)
- Removing or renaming fields
- Changing field types or semantics
- Making optional fields required
- Changing error response format
- Removing endpoints

### Versioning strategies
- **URL path:** `/v1/users` → `/v2/users` — simple, explicit
- **Header:** `Accept: application/vnd.api.v2+json` — cleaner URLs
- **Additive only:** Only non-breaking changes. Simplest when feasible.

### Sunset process
1. Announce deprecation with timeline
2. Add deprecation headers (`Sunset`, `Deprecation`)
3. Monitor usage of deprecated version
4. Return 410 Gone or redirect after sunset
5. Remove code after grace period

## Anti-Patterns

- **Removing fields without versioning** — consumers get undefined/null/crash.
- **Changing semantics without names** — invisible breakage.
- **No sunset timeline** — old versions accumulate forever.
- **Testing only your own consumers** — third parties exist.
