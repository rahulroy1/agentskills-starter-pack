---
name: code-quality
description: >
  Reviews code for design quality, clarity, duplication, naming, error handling, and
  refactoring opportunities. Use during Tier 2/3 code quality and refactoring review
  lenses, or when evaluating implementation quality.
---

# Code Quality & Refactoring

## Checklist

### Design
- [ ] Single responsibility — each function/class does one thing
- [ ] Clear interfaces — public API obvious, internals hidden
- [ ] No circular dependencies
- [ ] Error handling complete — no swallowed exceptions, no silent failures
- [ ] Naming is precise — describes what, not how

### Complexity
- [ ] Functions under 30 lines (guideline)
- [ ] Cyclomatic complexity under 10
- [ ] Max 3 levels of nesting
- [ ] No more than 4 parameters per function
- [ ] No boolean flag parameters that change behavior

### Duplication
- [ ] Same logic doesn't appear in 3+ places
- [ ] Shared patterns extracted to utilities
- [ ] No copy-paste with minor variations

### Readability
- [ ] Code self-documenting — good names, clear structure
- [ ] Comments explain *why*, never *what*
- [ ] No magic numbers or strings — named constants
- [ ] Consistent formatting with project conventions

## Patterns

### Extract when you see
- Function doing two things → split
- Repeated if/else chains → strategy pattern or lookup table
- Long parameter lists → parameter object
- Nested callbacks → extract named functions

### Naming
- Functions: verb phrases (`calculateTotal`, `validateInput`)
- Variables: noun phrases (`userCount`, `activeConnections`)
- Booleans: question form (`isValid`, `hasPermission`)

## Anti-Patterns

- **Premature abstraction** — don't extract until you see the pattern 3 times.
- **Clever code** — if it needs a comment to explain *what* it does, rewrite it.
- **Over-engineering** — don't build for extensibility you don't need.
- **Drive-by refactoring** — don't refactor unrelated code in the same changeset.
