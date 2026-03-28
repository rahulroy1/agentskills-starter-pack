---
name: code-quality
description: >
  Review code for design quality, clarity, duplication, naming, error handling, and
  refactoring opportunities. Activate during Tier 2/3 code quality and refactoring
  review lenses, or when evaluating implementation quality.
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

## Gotchas

- Extracting a helper after seeing it twice is premature — wait for 3 occurrences. The third reveals the real abstraction.
- Boolean parameters (`process(data, verbose=True)`) almost always mean the function does two things. Split it.
- "Self-documenting code" doesn't excuse missing *why* comments on business rules. `if balance < 0` is clear; *why* negative balances are allowed is not.
- Parallelized work merged in completion order produces non-deterministic output. Always merge in source encounter order.

## Patterns

### Output Determinism
When same inputs must produce same outputs:
- Use content-derived identifiers over random IDs
- Preserve source encounter order — don't sort by convenience
- Parallelize work, but merge results in deterministic order (source order, not completion order)

## Anti-Patterns

- **Premature abstraction** — don't extract until you see the pattern 3 times.
- **Clever code** — if it needs a comment to explain *what* it does, rewrite it.
- **Drive-by refactoring** — don't refactor unrelated code in the same changeset.
- **Non-deterministic output** — random IDs, completion-order merges, or unstable sorts that make diffing between runs impossible.
