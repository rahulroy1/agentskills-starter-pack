---
name: audit-trail
description: >
  Guides integrity verification, manual correction workflows, and change
  provenance tracking. Use when agents and humans both modify the same
  artifacts, or when outputs must be auditable.
related-skills: llm-integration, data-migration
---

# Audit Trail

## Checklist

### Integrity Verification
- [ ] Every write operation records hash before and hash after
- [ ] Integrity checks are automated — not left to manual spot-checking
- [ ] Corrupted or unexpected state is detected before downstream consumption

### Scope Control
- [ ] Before any mutation, compute the actual set of changes and compare to intended scope
- [ ] Out-of-scope changes are a hard failure — blocked, not just reported
- [ ] Scope verification is a gate in the workflow, not post-hoc telemetry

### Manual Corrections
- [ ] Manual corrections are logged with the same rigor as automated changes
- [ ] Each correction records: what changed, before/after values, reason, who, when, evidence
- [ ] Manual corrections are replayable — if the source is re-processed, the same corrections can be reapplied

### Derived Artifact Sync
- [ ] When one artifact is modified, all derived representations are updated in the same operation
- [ ] Never ship a primary artifact without verifying its derivatives are consistent
- [ ] Sync verification is automated — not dependent on the author remembering

### Provenance
- [ ] Consumers can distinguish: auto-generated vs human-corrected vs hybrid
- [ ] Each data point carries its origin: source reference, extraction method, validation status
- [ ] Provenance metadata survives through all transformations — not stripped in intermediate steps

## Patterns

### Hash-gated write
```
hash_before = hash(artifact)
apply_changes(artifact)
hash_after = hash(artifact)
log(patch_id, hash_before, hash_after, reason, changes)
```

### Manual correction log entry
```
{
  "id": "patch_001",
  "date": "YYYY-MM-DD",
  "target": "path/to/artifact",
  "changes": [
    {"field": "...", "before": "...", "after": "...", "reason": "..."}
  ],
  "evidence": "reference to source that confirms correction",
  "hash_before": "...",
  "hash_after": "..."
}
```

### Scope diff gate
Before write: compute `intended_changes` vs `actual_changes`. If `actual - intended` is non-empty, abort with report of unintended mutations.

## Anti-Patterns

- **Telemetry-only scope checks** — reporting out-of-scope changes without blocking them. If you only log it, you will ship it.
- **Patch without log** — fixing data silently. Future debugging becomes impossible when you cannot tell what the machine produced vs what a human corrected.
- **Primary-only updates** — patching the main artifact but forgetting to regenerate derived outputs. Consumers receive inconsistent views of the same data.
- **Provenance stripping** — intermediate processing steps that discard origin metadata. By the time the output reaches consumers, nobody knows where a value came from.
- **Unreplayable corrections** — manual fixes applied as one-off edits with no record. When the source is re-processed, the same errors reappear and nobody remembers the fix.
