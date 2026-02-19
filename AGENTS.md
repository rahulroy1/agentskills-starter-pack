# AGENTS.md (v8 - Spec-Driven, Risk-Tiered, Skills-Compatible)

This file defines **how** agents operate — process, workflow, and safety boundaries.

Domain expertise is packaged as Agent Skills in `.agents/skills/`. This file references them by name.

### Skill Activation Protocol
1. **Discovery:** Agent scans `.agents/skills/*/SKILL.md` at session start.
2. **Metadata load:** Read frontmatter (name, description) only — ~50 tokens per skill.
3. **Activation triggers:** Tier 3 exploration (identify from §3 table); review phase (per lens §6); explicit reference in AGENTS.md.
4. **Full load:** Read complete SKILL.md only when activated.
5. **Missing skill:** Log warning, proceed with AGENTS.md guidance only.

**Goal:** Maximize quality per unit time. Rigor where risk is high, speed where risk is low.

**Philosophy:** Spec-driven development. Explore → Spec → Implement. Code is the last step.

---

## 1) Core Principles (Priority Order)

When principles conflict, higher rank wins.

1. **Security and safety** — Non-negotiable hard floor. All tiers, all times. Activate: `security-baseline`
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

### Tier Classification Decision Tree
- **Q1:** Does this change observable behavior outside the repo? → YES: Tier 3.
- **Q2:** Does this change a contract (API, schema, CLI output)? → YES: Tier 3.
- **Q3:** Does this affect multiple modules/components? → YES: Tier 2. Else: Tier 1.

| Scenario | Tier | Rationale |
|----------|------|-----------|
| Add optional API field | 3 | Contract change (even if backward compatible) |
| Change internal error handling that affects error messages | 3 | Observable behavior change |
| Refactor internal function, same inputs/outputs | 1 | No observable change |
| Change shared utility used by 5 modules | 2 | Multiple modules affected |

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
- **Spec** — write and get **explicit user approval** (see Tier 3 Spec Approval Protocol below):
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

#### Tier 3 Spec Approval Protocol
1. **Present spec** as markdown with clear sections (§3).
2. **Request approval:** "Approve this spec? (yes/no/modify)"
3. **Wait for response.** Do not proceed without explicit approval.
4. **Partial approval:** If user requests changes → update spec → re-present → re-approve.
5. **Approval record:** Log approval in `tasks/todo.md` with timestamp.

#### Tier 3 — Skill-Based Conditional Sections

Include in the spec when the change involves these concerns. Activate the relevant skill for detailed patterns and checklists.

| Concern | When to include | Skill | Phase |
|---------|----------------|-------|-------|
| Migration / deprecation | Changing a contract with active consumers | `migration-deprecation` | Exploration |
| Data migration | Schema, format, or storage changes | `data-migration` | Exploration |
| Cross-service coordination | Multiple services must update | `cross-service-coordination` | Exploration |
| Deployment strategy | Can't deploy atomically or carries prod risk | `deployment-strategy` | Spec |
| Backward compatibility | Consumers can't all update simultaneously | `api-contracts` | Exploration |
| Infrastructure | DB, queues, networking, config changes | `infrastructure-changes` | Exploration |
| Observability | Production behavior needs monitoring | `observability` | Spec |
| Failure mode analysis | Partial rollout, async, distributed changes | `failure-analysis` | Exploration |

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
| 7 | Architecture | — (see Architecture Review Lens below) |

### Required per tier
| Tier | Spec | Security | Code | Tests | Docs | Refactoring | Architecture |
|------|------|----------|------|-------|------|-------------|--------------|
| 1 | — | Required | Optional | Optional | Optional | Optional | — |
| 2 | Required | Required | Required | Required | If affected | Optional | — |
| 3 | Required | Required | Required | Required | Required | Required | Required |

For Tier 3, spawn a review subagent per lens (§7).

**Findings:** Critical/High → fix or log approved debt with owner. Medium/Low → note and justify.

### Architecture Review Lens
- [ ] Change aligns with system architecture (monolith/modulith/microservices).
- [ ] No new coupling introduced (circular dependencies, shared state).
- [ ] Boundaries respected (module/service boundaries, data ownership).
- [ ] Scalability impact assessed (bottlenecks, resource usage).
- [ ] Technology choices justified (why this library/framework/pattern).

---

## 7) Subagent Strategy

Keep main context clean. Parallelize work. **Drive exploration before spec writing.**

One task per subagent. Name descriptively.

**Naming convention:** `[phase]-[scope]-[task]` (e.g. `explore-api-contracts-consumers`, `implement-user-service-layer`, `review-security-authn-authz`). Avoid generic names like `explore` or `fix-stuff`.

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
- **Cross-system mapping** (Tier 3): Contract dependency graph. Activate: `cross-service-coordination`
- **Failure mode analysis** (Tier 3): Unsafe intermediate states. Activate: `failure-analysis`

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

**Escalation template:**
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

**Scope changed:** Stop → reclassify tier → update spec → present to user.

---

## 10) Security (Non-Negotiable — All Tiers)

Activate: `security-baseline` for full checklists and patterns.

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

**Migration/deprecation:** Same-change alignment across runtime, modules, tests, docs, contracts, commands, baselines. Remove deprecated paths once new is accepted. Activate: `migration-deprecation`

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
- [ ] Security self-check. Activate: `security-baseline`
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

---

## Appendix: Quick Reference

| Task | Tier | Required Skills | Spec Approval? |
|------|------|-----------------|----------------|
| Fix typo in error message | 1 | security-baseline | No |
| Add new API endpoint | 3 | api-contracts, security-baseline | Yes |
| Refactor shared utility | 2 | code-quality | No |
| Database schema change | 3 | data-migration, failure-analysis | Yes |
| Deploy to production | 3 | deployment-strategy, observability | Yes |
