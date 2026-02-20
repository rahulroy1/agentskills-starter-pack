# Agent Skills Authoring Guide

## Structure

Each skill should follow this structure:

1. **Frontmatter:** `name`, `description`, optionally `activation-triggers`, `related-skills`
2. **Related Skills:** Cross-references to skills commonly used together
3. **Checklist:** Actionable items, organized by category
4. **Patterns:** Concrete examples and recommended approaches
5. **Verification:** Commands or examples to verify checklist items (when applicable)
6. **Anti-Patterns:** What to avoid and why

## Naming Convention

- **Format:** kebab-case
- **Descriptive:** Not generic (e.g. `api-contracts`, not `api`)
- **Folder name:** Must match `name` in frontmatter

## Frontmatter Example

```yaml
---
name: skill-name
description: >
  One-line when to use this skill. Use for Tier X changes involving...
activation-triggers:
  - "Tier 3 change modifying..."
  - "Review lens: Architecture (when...)"
related-skills: [other-skill-name]
---
```

## Testing a New Skill

1. Verify activation triggers are clear from the description
2. Check cross-references are accurate and bidirectional where appropriate
3. Ensure examples are runnable or clearly illustrative
4. Add the skill to the AGENTS.md §3 table if it's a Tier 3 conditional concern
