---
name: llm-integration
description: >
  Guides how to use LLM calls inside automated workflows — when to use them,
  how to validate their output, and how to maintain transparency. Use when
  building or reviewing any system that invokes an LLM as a processing step.
related-skills: audit-trail, testing-strategy
---

# LLM Integration

## Checklist

### Separation of Concerns
- [ ] Structure and existence decisions are deterministic — LLM never decides *what exists*, only *what it means*
- [ ] LLM is used for interpretation, classification, or enrichment — not for authoritative data
- [ ] Every LLM call site has a defined fallback if the model is unavailable or returns garbage

### Post-Validation
- [ ] Every LLM output passes deterministic validation before entering the system
- [ ] Validation checks structure (schema, types, required fields) AND domain constraints (value must exist in known set, reference must resolve)
- [ ] Invalid LLM output degrades to safe default or "unresolved" — never silently accepted

### Evidence and Provenance
- [ ] LLM output is never promoted to authoritative data without traceable source evidence
- [ ] Each resolved item carries: source input, model output, validation result, evidence reference
- [ ] Unresolved items are explicitly logged with reason — not silently dropped

### Transparency
- [ ] Every LLM invocation logs: model, tokens in, tokens out, latency, cost (or cost estimate)
- [ ] Aggregate token/cost telemetry available per run, per stage, per session
- [ ] Quality and accuracy are non-negotiable — transparency exists so teams make informed decisions, not to cut corners

### Prompt Design
- [ ] Prompts are precise about what role the LLM plays (classify, resolve, extract — not "figure it out")
- [ ] Prompts distinguish input from output when both appear in the source material (e.g., "which column is the lookup key" vs "which column is the result")
- [ ] Prompts include the domain vocabulary the model needs — don't assume the model shares your terminology
- [ ] Batch multiple items per call when items are independent — validate each item individually

### Merge Discipline
- [ ] LLM output is merged as union with deterministic baseline — LLM refines, never reduces
- [ ] If deterministic extraction and LLM disagree, the union is kept and the conflict is flagged — not silently resolved
- [ ] Re-running with different model versions or parameters produces auditable diff, not silent drift

## Patterns

### Validate-then-accept
```
llm_result = call_llm(prompt)
validated = validate_against_known_constraints(llm_result)
if validated:
    accept(validated, evidence=prompt_input)
else:
    mark_unresolved(reason="failed validation", raw=llm_result)
```

### Batch with per-item validation
Send N items in one call. Validate each returned item independently. Accept valid ones, mark invalid ones unresolved. Never accept or reject the batch as a whole.

### Transparency log entry
Each LLM call logs: `{timestamp, model, tokens_in, tokens_out, latency_ms, cost_estimate, call_site, batch_size}`

## Anti-Patterns

- **LLM as source of truth** — treating model output as authoritative without validation. The model suggests; the system decides.
- **Silent acceptance** — skipping validation because "the model is usually right." Usually right is not always right.
- **Prompt ambiguity** — asking the model to "figure out the right column" instead of "given this evidence, which column is the INPUT key." Ambiguous prompts produce confidently wrong answers.
- **Suppression by LLM** — letting LLM results replace deterministic baseline instead of merging. Baseline coverage must never regress.
- **Cost opacity** — running LLM calls without logging token usage. Teams cannot govern what they cannot measure.
- **One-shot for everything** — using the same prompt complexity and reasoning depth for trivial and complex items alike. Match prompt effort to item complexity.
