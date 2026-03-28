# Agent Skills Authoring Guide

## Structure

Each skill should follow this structure:

1. **Frontmatter:** `name`, `description` (required). Optional: `license`, `compatibility`, `allowed-tools`, `metadata`
2. **Related Skills:** Cross-references to skills commonly used together
3. **Gotchas:** Environment-specific facts that defy agent assumptions — highest-value content
4. **Checklist:** Actionable items, organized by category
5. **Patterns:** Pick a default approach, mention alternatives briefly. Favor procedures over declarations.
6. **Validation Loop / Plan-Validate-Execute:** Instruct agent to check its own work (when applicable)
7. **Anti-Patterns:** What to avoid and why

## Naming Convention

- **Format:** Lowercase, alphanumeric + hyphens only (1-64 chars)
- **No consecutive hyphens**, cannot start/end with hyphens
- **Descriptive:** Not generic (e.g. `api-contracts`, not `api`)
- **Folder name:** Must exactly match `name` in frontmatter
- **Description:** Max 1024 chars, imperative phrasing, include activation keywords

## Frontmatter Example

```yaml
---
name: skill-name
description: >
  Imperative verb + what the skill does + when to activate. Keep under 1024 chars.
  Include concrete keywords agents will search for.
---
```

## Content Best Practices

- **Add what the agent lacks, omit what it knows.** Ask: "Would the agent get this wrong without this instruction?" If no, cut it.
- **Gotchas are highest value.** Concrete corrections to mistakes agents will make without being told.
- **Defaults, not menus.** When multiple approaches work, pick one default and mention alternatives briefly.
- **Procedures over declarations.** Teach *how to approach* a class of problems, not *what to produce*.
- **Validation loops.** Instruct the agent to validate its own work before moving on.
- **Keep under 500 lines / 5000 tokens.** Move detailed reference material to `references/` with clear loading triggers.

## Testing a New Skill

1. Verify activation triggers are clear from the description
2. Check cross-references are accurate and bidirectional where appropriate
3. Ensure examples are runnable or clearly illustrative
4. Add the skill to the AGENTS.md §3 table if it's a Tier 3 conditional concern
