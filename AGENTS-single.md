# AGENTS.md (v6 - Spec-Driven, Risk-Tiered)

This file defines how agents should think, plan, execute, verify, and communicate when working in this repository.

**Goal:** Maximize quality per unit time — keep rigor where risk is high, avoid process overhead where risk is low.

**Philosophy:** Spec-driven development. Understand before building. Every non-trivial change starts with exploration, produces a spec, and verifies against that spec. Code is the last step, not the first.

---

## 1) Core Principles (Priority Order)

When principles conflict, higher rank wins.

1. **Security and safety** — Non-negotiable. Applies at all tiers, all times. Hard floor.
2. **Correctness and contract integrity** — Outputs satisfy their contracts. Deterministic outputs stay deterministic. Tests pass.
3. **Understand before building** — Explore, validate assumptions, write a spec, *then* implement. Never code from assumptions.
4. **Outcome over ceremony** — Use the lightest process that still satisfies #1, #2, and #3.
5. **Risk-proportional rigor** — Scale process to blast radius.
6. **No silent assumptions** — State and validate critical assumptions with code, tests, or data.

---

## 2) Risk Tiers (Required First Step)

Before implementation, classify the task by **blast radius** — how far the damage reaches if you get it wrong.

### Boundary test

| Question (evaluate top-down, first YES wins) | Tier |
|-----------------------------------------------|------|
| Could this break other systems, services, environments, or require coordinated deployment? | **3** |
| Does this change a public contract — output schemas, APIs, CLI behavior, shared interfaces — that external consumers depend on? | **3** |
| Does this cross module/component boundaries or change internal interfaces other modules depend on? | **2** |
| Is the change fully contained — one module, one file, or one function — with no effect beyond its boundary? | **1** |

When in doubt, tier up.

### Tier 1 — Contained
Blast radius: one function, file, or module. Nothing else breaks if you get it wrong.
- Examples: single-function bug fix, internal refactor, typo fix, log message change, script flag default adjustment.

### Tier 2 — Local Ripple
Blast radius: multiple modules or components within one codebase. Internal interfaces change, but nothing outside the repo/service is affected.
- Examples: feature touching 3+ internal files, internal data model change, new shared utility, internal API extension, adding a CLI flag with no default behavior change.

### Tier 3 — Contract / System Boundary
Blast radius: crosses a trust boundary. Consumers, downstream systems, other services, or users depending on your output will be affected. Includes any change that can't be rolled back trivially.

This tier covers different architectures with different concerns:

| Architecture | What makes it Tier 3 |
|-------------|---------------------|
| **Script/CLI** | Changing output format, exit codes, or argument behavior that other scripts or pipelines depend on |
| **Monolith** | Changing a shared module's public interface that multiple domains consume, DB schema migration, changing the API contract |
| **Modulith** | Breaking a module boundary contract, changing shared event/message schemas, altering module initialization order |
| **Microservices** | Changing a service API contract, shared schema/protobuf evolution, cross-service data migration, infrastructure changes |

### Reclassification
If during work you discover the change has a larger blast radius than classified, **stop and reclassify**. Apply the rules for the new tier before continuing.

---

## 3) Explore → Spec → Implement

Core workflow. Depth scales with tier, but the sequence is always: **understand first, define success, then build.**

---

### Tier 1 — Lightweight

**Explore:** Read the relevant code. Confirm the change is truly contained.

**Spec:** One or two sentences: what you're changing, what the expected outcome is.

**Implement:** Make the change.

---

### Tier 2 — Structured

**Explore** (use subagents if 3+ files — see §7):
- Read affected modules and their tests.
- Trace data flow through the change boundary.
- Identify callers, consumers, and side effects.
- Validate assumptions about current behavior with evidence.

**Spec** — write before coding:
- **Goal:** What problem this solves, in one sentence.
- **Changes:** Files/modules affected and what changes in each.
- **Contracts preserved:** What existing behavior stays the same (explicit).
- **Acceptance criteria:** Concrete, testable conditions that define "done."
- **Tests:** Which tests to add or update.
- **Verification:** Commands to run and expected results.

**Implement:** Code to the spec. If reality diverges, update the spec first.

---

### Tier 3 — Full Spec

**Explore** (subagents mandatory — see §7):
- All of Tier 2 exploration, plus:
- Map all downstream consumers and contracts affected.
- Research alternatives and trade-offs.
- Assess rollback feasibility.
- Identify the applicable Tier 3 concerns from the table below.

**Spec** — write and present for **explicit user approval** before implementation:

Core spec (always required):
- **Goal:** Problem statement and desired outcome.
- **Approach:** Chosen solution with alternatives considered and why rejected.
- **Changes:** Complete list of files, modules, contracts, and configs affected.
- **Contract diff:** Explicit before/after for every public interface or output changing.
- **Acceptance criteria:** Numbered, testable conditions.
- **Rollback strategy:** How to revert if something goes wrong.
- **Tests:** New tests, updated tests, regression coverage.
- **Verification gates:** Commands and expected outputs.

Conditional sections — include when applicable:

| Concern | When to include | What to specify |
|---------|----------------|-----------------|
| **Migration/deprecation** | Changing an existing contract that has active consumers | Migration steps, deprecation timeline, backward compatibility window, consumer communication plan |
| **Data migration** | Schema changes, data format changes, storage restructuring | Migration script, data validation, backfill strategy, rollback plan for data (not just code), dry-run results |
| **Cross-service coordination** | Change requires multiple services to update | Deployment order, intermediate compatibility states, feature flags, contract versioning strategy |
| **Deployment strategy** | Change can't be deployed atomically or carries production risk | Blue-green / canary / rolling strategy, rollback triggers, health check criteria, monitoring plan |
| **Backward compatibility** | Consumers can't all update simultaneously | Additive-only contract changes, version negotiation, sunset timeline for old contract |
| **Infrastructure** | Infra changes (DB, queues, networking, config) | Terraform/IaC plan, environment promotion order, blast radius per environment, rollback procedure |
| **Observability** | Change affects production behavior in ways that need monitoring | Key metrics to watch, alerting thresholds, dashboards to update, how to detect the change is working (or failing) in production |
| **Failure mode analysis** | Distributed or async changes where partial rollout is possible | What happens if only some consumers have updated? What happens if the migration is half-complete? Identify unsafe intermediate states and how to avoid or recover from them |

**Implement:** Code to the approved spec. If reality diverges, stop → update spec → get re-approval → continue.

---

### Spec is the source of truth
- Implementation is measured against the spec.
- Verification confirms the spec's acceptance criteria are met.
- Review checks whether the spec was followed.
- If the spec was wrong, fix the spec first, then fix the code.

---

## 4) Execution Rules

1. Change only what the spec calls for. No drive-by refactors, no scope creep.
2. Preserve existing conventions, patterns, and style.
3. Prefer simple, explicit, readable solutions over clever ones.
4. Never expose secrets in code, logs, tests, prompts, or output.
5. Never perform destructive operations (delete files, drop data, remove config) without explicit user confirmation.
6. Verify write scope before writing. Before any mutation, compute what will actually change and confirm it matches intended scope. Unintended side effects are the #1 source of silent data corruption.

---

## 5) Verification (Always Required)

Verify against the spec's acceptance criteria. Every change must include evidence: commands run, pass/fail results, and key outputs.

### Tier 1
- Lint/compile (if applicable) + targeted tests covering the change.

### Tier 2
- Tier 1 + impacted integration tests.
- Confirm each acceptance criterion is met with evidence.

### Tier 3
- Tier 2 + full regression suite and required gate reports.
- For deterministic pipelines: before/after artifact diff checks.
- **Derived artifact consistency:** When a single logical change produces multiple outputs, verify all outputs reflect the change.
- Walk through every acceptance criterion with evidence.
- If conditional spec sections apply: verify each (migration tested, deployment strategy validated, rollback tested, observability confirmed).

### When tests don't exist
- **Tier 1:** Add at least a targeted test. If not feasible, document why and provide manual verification evidence.
- **Tier 2/3:** Add tests. Untested Tier 2/3 changes are not complete.

No change is done without verification evidence.

---

## 6) Quality Review (Risk-Scaled)

Six review lenses:
1. **Spec conformance** — Does implementation match the spec? All acceptance criteria satisfied?
2. **Code quality** — Design, clarity, duplication, naming, error handling.
3. **Test coverage** — Edge cases, error paths, integration points, test quality.
4. **Security** — Input validation, injection risks, secrets, auth/authz, log exposure, dependency vulnerabilities.
5. **Documentation** — Public API docs, README, inline comments for complex logic, architecture docs.
6. **Refactoring** — Long functions, repeated patterns, tight coupling, magic values.

### Required per tier
| Tier | Spec | Security | Code | Tests | Docs | Refactoring |
|------|------|----------|------|-------|------|-------------|
| 1 | — | Required (quick check) | Optional | Optional | Optional | Optional |
| 2 | Required | Required | Required | Required | If affected | Optional |
| 3 | Required | Required | Required | Required | Required | Required |

For Tier 3, spawn a review subagent per lens in parallel (§7).

### Findings resolution
- **Critical/High:** Fix before marking complete, or explicitly log as approved debt with owner and timeline.
- **Medium/Low:** Note and address or defer with justification.

---

## 7) Subagent Strategy

Use subagents to keep the main context window clean, parallelize work, and — most importantly — **drive thorough exploration before spec writing.**

One focused task per subagent. Name descriptively.

### When to use

| Phase | Tier 1 | Tier 2 | Tier 3 |
|-------|--------|--------|--------|
| **Exploration** | Not needed | Recommended | **Mandatory** |
| **Implementation** | Not needed | Optional (3+ files) | Recommended |
| **Quality review** | Not needed | Optional | Recommended (one per lens) |

### Exploration subagents (most important use)

Before writing a Tier 2/3 spec, spawn exploration subagents to build a complete picture:

**Codebase analysis:** "Trace module X's public interface, internal structure, callers, and dependencies."

**Behavior validation:** "Run [test/command] and confirm assumption X about current behavior."

**Impact assessment:** "Identify everything depending on [function/interface/schema]. List all files, tests, docs, and downstream consumers that would need updating."

**Alternatives research:** "Compare approaches A, B, C for [problem] — trade-offs, complexity, risk, migration cost."

**Dependency/compatibility check:** "Check version compatibility for [library]. Identify breaking changes between current and target."

**Cross-system mapping** (Tier 3, distributed): "Identify all services that consume [API/schema/event]. Map the contract dependency graph. List what breaks if [proposed change] ships."

**Failure mode analysis** (Tier 3, distributed): "If service A deploys the change but service B hasn't updated yet, what happens? Identify unsafe intermediate states."

Synthesize subagent findings in the main thread, then write the spec from validated knowledge — not assumptions.

### Implementation subagents

Split large features by layer or component. Main thread orchestrates, integrates, and resolves conflicts. **No two subagents modify the same file** — if overlap is unavoidable, main thread applies changes sequentially after review.

### Review subagents

For Tier 3, spawn one subagent per review lens (§6). Each returns: structured findings, severity (Critical/High/Medium/Low), locations (file:line), reasoning, and suggested fix.

### When NOT to use subagents
Trivial changes, reading a single file, simple questions, quick checks.

---

## 8) Autonomous Bug Fixing

When given a bug report: **fix it.** Don't ask for reproduction steps, which file to check, how to fix it, or permission to investigate.

If the report is too vague to act on, ask **one** clarifying question about what is broken — not how to fix it.

Bug fixing still respects §16 restricted operations.

### Process
1. **Explore:** Trace the reported behavior. Read errors, logs, tests, related code. Use subagents for complex investigations.
2. **Reproduce:** Write a failing test or isolate minimum reproduction.
3. **Diagnose:** Root cause, not symptoms. Check for related bugs.
4. **Classify:** Determine the tier of the *fix*. If it requires contract changes, it's Tier 3 — write a spec and get approval.
5. **Spec:** Tier 2/3 fixes get a spec (§3). Tier 1: state what you're changing and why.
6. **Fix:** Address root cause. Consider edge cases.
7. **Verify:** Reproduction test passes, all existing tests pass, no regressions.
8. **Prevent:** Regression test, docs update, check for similar issues.

### Report format
```markdown
## Bug Fix: [Brief Description]
**Tier:** [1/2/3]
**Issue:** [What was broken]
**Root Cause:** [Why it was broken]
**Fix:** [What changed and why]
**Verification:** [Commands run, pass/fail evidence]
**Files Changed:** [list with descriptions]
```

---

## 9) Error Recovery

### When stuck
1. Stop. Do not retry the same approach more than twice.
2. Re-read error output carefully.
3. Re-assess: right approach, or different path needed?
4. If blocked after 3 attempts: **escalate to user** with what you tried, what failed, and your hypothesis.

### When scope changes mid-task
1. Stop current work.
2. Reclassify tier (§2).
3. Write or update the spec for the new tier (§3).
4. Present findings and new spec to user before continuing.

---

## 10) Security Baseline (All Tiers — Non-Negotiable)

### Secrets
- Never hardcode, commit, log, or expose credentials, tokens, or keys.
- Use environment variables. Fail fast at startup if required secrets are missing.

### Input validation
- Validate all external inputs at system boundaries.
- Use allowlists over blocklists.

### Injection prevention
- Use parameterized queries / prepared statements for database access.
- Never construct commands, queries, or templates from unsanitized input.
- Avoid dynamic code execution (eval, exec) unless sandboxed.

### Auth
- Authentication ≠ Authorization. Always check both.
- Least-privilege: minimum necessary permissions.

### Cryptography
- Established libraries only. Never roll custom crypto.
- No deprecated algorithms (MD5, SHA-1, DES, RC4).
- Passwords: bcrypt, scrypt, or Argon2 with salt.

### Error/log hygiene
- Never expose stack traces, internal paths, DB schemas, or keys in user-facing output.
- Never log passwords, tokens, PII, or secrets.

### Dependencies
- Minimal and auditable. Pin in production. Remove unused. Run scanners when available.

---

## 11) Policies

### Migration and Deprecation
Completion requires same-change alignment across: runtime, modules, tests, docs, contracts, command examples, and baselines. Remove deprecated paths once new path is accepted.

### Cleanup/Deletion Safety
Before deleting/moving: define scope allowlist, define preservation set, verify no cross-scope impact. Never delete outside approved scope.

### Documentation Drift Control
Contract/interface/architecture changes must update owning docs **in the same changeset**. Maintain a Key Design Decisions log: decision, rationale, date, impact.

---

## 12) Lessons Learned

Update `tasks/lessons.md` only for systemic failures, repeated patterns, or high-impact misses.

```markdown
### [Date] - [Brief Title]
**What went wrong:** [1 sentence]
**Why:** [root cause]
**Prevention:** [actionable rule]
```

**Session start:** Read `tasks/lessons.md` and apply relevant prevention rules to the current task.

---

## 13) Task Tracking

- `tasks/todo.md` — specs, execution plans, completion state
- `tasks/lessons.md` — durable learning (§12)

For Tier 2/3, write specs and track progress in `todo.md`.

---

## 14) Context Window Management

- Don't load everything. Use search, grep, targeted reads.
- Read architecture docs and READMEs first.
- **Use exploration subagents** (§7) for unfamiliar modules — keep main thread for synthesis.
- On exhaustion: offload to subagents, write findings to `tasks/notes.md`, summarize state to file before continuing.

---

## 15) Communication

Structure every response as:
1. **What changed** — files, behavior, scope
2. **Why** — reasoning, trade-offs, spec reference
3. **Evidence** — verification commands and results
4. **Open items** — residual risks, assumptions, limitations

Concise and factual. No filler, no hype.

---

## 16) Permissions and Safety Boundaries

### Allowed
- Read any file
- Edit source (Tier 1 freely; Tier 2/3 after spec approval)
- Run build, test, lint, pipeline commands
- Create files following project structure
- Install dependencies (after approval)

Spec approval = explicit user approval in-chat.

### Restricted — require explicit user confirmation
- Delete files or directories
- Modify production credentials or config
- Commit or push to version control
- Modify `.git/` metadata
- Destructive commands (DROP, TRUNCATE, rm -rf, etc.)

---

## 17) Quick Checklists

### Tier 1
- [ ] Confirmed change is contained (Tier 1)
- [ ] Change implemented
- [ ] Targeted tests pass (add test if none exist)
- [ ] Security self-check (secrets, injection, log leaks)
- [ ] No unintended file changes
- [ ] Summary: what, why, evidence

### Tier 2
- [ ] Exploration complete (subagents if 3+ files)
- [ ] Spec written with acceptance criteria
- [ ] Implemented per spec
- [ ] Unit + integration tests pass
- [ ] Spec conformance, security, code, test lenses reviewed
- [ ] Affected docs updated
- [ ] Summary with spec reference and evidence

### Tier 3
- [ ] Exploration complete (subagents mandatory)
- [ ] Full spec with applicable conditional sections
- [ ] Spec approved by user
- [ ] Implemented per approved spec
- [ ] Full test/gate suite passes
- [ ] Conditional verifications complete (migration, deployment, rollback, observability — as applicable)
- [ ] Deterministic artifact diffs reviewed (if applicable)
- [ ] All 6 review lenses completed
- [ ] Runtime, tests, docs, contracts aligned
- [ ] Key design decisions updated
- [ ] Final report: spec reference, evidence, residual risks, open items
