# AGENTS.md (v8 - Spec-Driven, Risk-Tiered, Skills-Compatible)

This file defines **how** agents operate — process, workflow, and safety boundaries.

Domain expertise is packaged as Agent Skills in `.agents/skills/`. This file references them by name; the agent's skill discovery system handles activation.

**Goal:** Maximize quality per unit time. Rigor where risk is high, speed where risk is low.

**Philosophy:** Spec-driven development. Explore → Spec → Implement. Code is the last step.

---

## 1) Core Principles (Priority Order)

When principles conflict, higher rank wins.

1. **Security and safety** — Non-negotiable hard floor. All tiers, all times. → skill: `security-baseline`
2. **Correctness and contract integrity** — Outputs satisfy contracts. Tests pass. Deterministic stays deterministic.
3. **Understand before building** — Explore, validate assumptions, spec, *then* implement.
4. **Outcome over ceremony** — Lightest process that satisfies #1, #2, #3.
5. **Risk-proportional rigor** — Scale process to blast radius.
6. **No silent assumptions** — State and validate with code, tests, or data.

---

## 2) Risk Tiers (Required First Step)

Classify by **blast radius** — how far damage travels if you get it wrong.

| Question (top-down, first YES wins) | Tier |
|--------------------------------------|------|
| Breaks other systems/services/environments, or requires coordinated deployment? | **3** |
| Changes a public contract (output schema, API, CLI behavior, shared interface)? | **3** |
| Crosses module/component boundaries or changes internal interfaces others depend on? | **2** |
| Fully contained — one module, one file, one function? | **1** |

When in doubt, tier up.

### Tier 1 — Contained
Nothing else breaks. Single-function fix, internal refactor, typo, log change.

### Tier 2 — Local Ripple
Multiple internal modules affected, but nothing outside the repo/service boundary.

### Tier 3 — Contract / System Boundary
Consumers, downstream systems, or users depending on your output will be affected.

| Architecture | Tier 3 trigger |
|-------------|----------------|
| **Script/CLI** | Output format, exit codes, argument behavior change |
| **Monolith** | Shared module public interface, DB schema migration, API contract |
| **Modulith** | Module boundary contract, shared event schemas, initialization order |
| **Microservices** | Service API contract, shared schema evolution, cross-service migration, infra |

### Reclassification
If blast radius turns out larger than classified: **stop → reclassify → apply new tier rules.**

---

## 3) Explore → Spec → Implement

Depth scales with tier. Sequence is always: **understand → define success → build.**

### Tier 1
- **Explore:** Read relevant code. Confirm change is contained.
- **Spec:** 1-2 sentences — what changes, expected outcome.
- **Implement.**

### Tier 2
- **Explore** (subagents recommended — §7):
  - Read affected modules, tests, data flow.
  - Identify callers, consumers, side effects.
  - Validate assumptions with evidence.
- **Spec** — write before coding:
  - **Goal** — one sentence.
  - **Changes** — files/modules and what changes in each.
  - **Contracts preserved** — what stays the same (explicit).
  - **Acceptance criteria** — concrete, testable.
  - **Tests** — add/update.
  - **Verification** — commands and expected results.
- **Implement** per spec. Divergence → update spec first.

### Tier 3
- **Explore** (subagents mandatory — §7):
  - All Tier 2 exploration, plus:
  - Map downstream consumers and contracts.
  - Research alternatives and trade-offs.
  - Assess rollback feasibility.
  - Identify applicable skills (see below).
- **Spec** — write and get **explicit user approval**:
  - **Goal** — problem and desired outcome.
  - **Approach** — chosen solution, alternatives rejected with rationale.
  - **Changes** — all files, modules, contracts, configs.
  - **Contract diff** — explicit before/after for every public interface changing.
  - **Acceptance criteria** — numbered, testable.
  - **Rollback strategy.**
  - **Tests** — new, updated, regression.
  - **Verification gates** — commands and expected outputs.
  - **Applicable skill sections** — include relevant guidance from activated skills.
- **Implement** per approved spec. Divergence → stop → update spec → re-approve.

#### Tier 3 — Skill-Based Conditional Sections

Include in the spec when the change involves these concerns. Activate the relevant skill for detailed patterns and checklists.

| Concern | When to include | Skill |
|---------|----------------|-------|
| Migration / deprecation | Changing a contract with active consumers | `migration-deprecation` |
| Data migration | Schema, format, or storage changes | `data-migration` |
| Cross-service coordination | Multiple services must update | `cross-service-coordination` |
| Deployment strategy | Can't deploy atomically or carries prod risk | `deployment-strategy` |
| Backward compatibility | Consumers can't all update simultaneously | `api-contracts` |
| Infrastructure | DB, queues, networking, config changes | `infrastructure-changes` |
| Observability | Production behavior needs monitoring | `observability` |
| Failure mode analysis | Partial rollout, async, distributed changes | `failure-analysis` |

### Spec is the source of truth
Implementation is measured against the spec. Verification confirms acceptance criteria. Review checks conformance. Wrong spec → fix spec first, then code.

---

## 4) Execution Rules

1. Change only what the spec calls for. No scope creep.
2. Preserve existing conventions and style.
3. Simple and readable over clever.
4. Never expose secrets in code, logs, tests, prompts, or output.
5. Never perform destructive operations without explicit user confirmation.

---

## 5) Verification (Always Required)

Verify against the spec's acceptance criteria. Evidence required: commands, results, outputs.

| Tier | Required |
|------|----------|
| 1 | Lint/compile + targeted tests |
| 2 | Tier 1 + integration tests + acceptance criteria confirmed |
| 3 | Tier 2 + full regression + gate reports + conditional skill verifications |

For deterministic pipelines: before/after artifact diff checks.

**No tests exist?** Tier 1: add targeted test or document why with manual evidence. Tier 2/3: add tests — untested changes are not complete.

---

## 6) Quality Review (Risk-Scaled)

Seven review lenses:

| # | Lens | Skill |
|---|------|-------|
| 1 | Spec conformance | — |
| 2 | Code quality | `code-quality` |
| 3 | Test coverage | `testing-strategy` |
| 4 | Security | `security-baseline` |
| 5 | Documentation | — |
| 6 | Refactoring | `code-quality` |
| 7 | Architecture | — |

### Required per tier
| Tier | Spec | Security | Code | Tests | Docs | Refactoring | Architecture |
|------|------|----------|------|-------|------|-------------|--------------|
| 1 | — | Required | Optional | Optional | Optional | Optional | — |
| 2 | Required | Required | Required | Required | If affected | Optional | — |
| 3 | Required | Required | Required | Required | Required | Required | Required |

For Tier 3, spawn a review subagent per lens (§7).

**Findings:** Critical/High → fix or log approved debt with owner. Medium/Low → note and justify.

---

## 7) Subagent Strategy

Keep main context clean. Parallelize work. **Drive exploration before spec writing.**

One task per subagent. Name descriptively.

| Phase | Tier 1 | Tier 2 | Tier 3 |
|-------|--------|--------|--------|
| Exploration | — | Recommended | **Mandatory** |
| Implementation | — | Optional (3+ files) | Recommended |
| Review | — | Optional | Recommended (per lens) |

### Exploration subagents (most important use)

Spawn before writing Tier 2/3 specs:

- **Codebase analysis:** Trace interface, callers, dependencies of module X.
- **Behavior validation:** Run command, confirm assumption X.
- **Impact assessment:** What depends on this? What breaks if it changes?
- **Alternatives research:** Compare approaches — trade-offs, cost, risk.
- **Dependency check:** Version compatibility, breaking changes.
- **Cross-system mapping** (Tier 3): Contract dependency graph. → skill: `cross-service-coordination`
- **Failure mode analysis** (Tier 3): Unsafe intermediate states. → skill: `failure-analysis`

Synthesize findings in main thread → write spec from validated knowledge.

### Implementation subagents
Split by layer/component. Main thread orchestrates. **No two subagents modify the same file.**

### Review subagents
One per lens (§6). Returns: findings, severity, locations, reasoning, suggested fix.

---

## 8) Autonomous Bug Fixing

**Fix it.** Don't ask for repro steps, which file, or permission. If report is too vague, ask **one** question about *what* is broken.

Respects §15 restricted operations.

1. **Explore** — trace behavior, errors, logs, code. Subagents for complex cases.
2. **Reproduce** — write failing test or isolate minimum case.
3. **Diagnose** — root cause, not symptoms.
4. **Classify** — tier the *fix*. Tier 3 fix → spec + approval.
5. **Spec** — Tier 2/3 per §3. Tier 1: state change and rationale.
6. **Fix** — root cause, edge cases.
7. **Verify** — repro test passes, all tests pass, no regressions.
8. **Prevent** — regression test, docs, check for similar issues.

```markdown
## Bug Fix: [Brief Description]
**Tier:** [1/2/3]
**Issue:** [What was broken]
**Root Cause:** [Why]
**Fix:** [What changed and why]
**Verification:** [Commands, evidence]
**Files Changed:** [list]
```

---

## 9) Error Recovery

**Stuck:** Stop after 2 failed retries → re-assess approach → escalate to user after 3 attempts with what you tried and your hypothesis.

**Scope changed:** Stop → reclassify tier → update spec → present to user.

---

## 10) Security (Non-Negotiable — All Tiers)

→ **Activate skill: `security-baseline` for full checklists and patterns.**

Hard rules (always apply):
- Never hardcode, commit, log, or expose secrets.
- Validate external inputs at boundaries. Allowlists over blocklists.
- Parameterized queries — never build SQL/commands from unsanitized input.
- Auth ≠ authz — check both. Least privilege.
- Established crypto only. No MD5/SHA-1/DES/RC4.
- Never leak stack traces, paths, schemas, or keys to users.
- Never log passwords, tokens, PII, or secrets.
- Dependencies: minimal, pinned, auditable, scanned.

---

## 11) Policies

**Migration/deprecation:** Same-change alignment across runtime, modules, tests, docs, contracts, commands, baselines. Remove deprecated paths once new is accepted. → skill: `migration-deprecation`

**Cleanup/deletion:** Scope allowlist → preservation set → verify no collateral → execute. Never outside approved scope.

**Doc drift:** Contract/interface/architecture changes update owning docs **in same changeset**. Maintain Key Design Decisions log.

---

## 12) Lessons Learned

`tasks/lessons.md` — systemic failures, repeated patterns, high-impact misses only.

```markdown
### [Date] - [Title]
**What went wrong:** [1 sentence]
**Why:** [root cause]
**Prevention:** [actionable rule]
```

**Session start:** Read `tasks/lessons.md` → apply relevant rules.

---

## 13) Task Tracking

- `tasks/todo.md` — specs, plans, completion state.
- `tasks/lessons.md` — durable learning (§12).

Tier 2/3: track progress in `todo.md`.

---

## 14) Context Management

- Don't load everything. Search, grep, targeted reads.
- Architecture docs and READMEs first.
- Exploration subagents for unfamiliar modules (§7).
- On exhaustion: subagent summaries, write to `tasks/notes.md`, summarize state to file.

---

## 15) Permissions

**Allowed:** Read any file. Edit source (Tier 1 freely; Tier 2/3 after spec approval in-chat). Run build/test/lint. Create files per structure. Install deps (after approval).

**Restricted (explicit confirmation required):** Delete files. Modify prod credentials/config. Commit/push. Modify `.git/`. Destructive commands (DROP, rm -rf, etc.).

---

## 16) Communication

Every response:
1. **What changed** — files, behavior, scope
2. **Why** — reasoning, spec reference
3. **Evidence** — commands, results
4. **Open items** — risks, assumptions, limitations

Concise, factual. No filler.

---

## 17) Quick Checklists

### Tier 1
- [ ] Confirmed contained (Tier 1)
- [ ] Implemented
- [ ] Tests pass (add if none exist)
- [ ] Security self-check → skill: `security-baseline`
- [ ] No unintended changes
- [ ] Summary: what, why, evidence

### Tier 2
- [ ] Exploration complete
- [ ] Spec with acceptance criteria
- [ ] Implemented per spec
- [ ] Unit + integration tests pass
- [ ] Spec, security, code, test lenses reviewed
- [ ] Docs updated
- [ ] Summary with spec reference + evidence

### Tier 3
- [ ] Exploration complete (subagents mandatory)
- [ ] Full spec with applicable skill-based sections
- [ ] Spec approved by user
- [ ] Implemented per spec
- [ ] Full test/gate suite passes
- [ ] Conditional verifications per applicable skills
- [ ] Artifact diffs reviewed (if deterministic)
- [ ] All 7 review lenses completed
- [ ] Runtime/tests/docs/contracts aligned
- [ ] Design decisions updated
- [ ] Final report: spec, evidence, risks, open items
