# Agent Skills Starter Pack

A ready-to-use operational framework for AI coding agents. Drop it into any repo and every agent — Claude Code, Codex CLI, Cursor, Kiro, and others — starts with shared rules, risk awareness, spec discipline, security guardrails, and cross-session memory.

Read the full rationale: [The Structure Engineering Orgs Need for AI Coding Agents](https://rahulroyz.medium.com/the-structure-engineering-orgs-need-for-ai-coding-agents-c05d7af6a49e)

## The Problem

The models are built to write good code — that isn't the problem. The problem is ungoverned execution: no risk assessment, no spec discipline, no security positioning, no cross-session memory, no operational boundaries. Every agent starts from zero and moves at inference speed.

At org scale — fifteen engineers, multiple services, Claude Code and Codex and Cursor — all prompting agents with no shared rules. The cost compounds fast.

Five structural gaps account for most agent-related rework:

| # | Gap | What Goes Wrong |
|---|-----|----------------|
| 1 | **No risk assessment** | A typo fix and a schema migration get the same level of attention |
| 2 | **No spec discipline** | Agents code first, hit walls, produce something that works but doesn't fit the architecture |
| 3 | **Security as afterthought** | A "simple bug fix" introduces injection; a "minor refactor" leaks API keys to stdout |
| 4 | **No cross-session memory** | Developer A learns a lesson Tuesday with Claude Code; Developer B hits the same wall Wednesday with Codex |
| 5 | **No operational boundaries** | Agents delete files, modify prod config, commit and push without review |

## What's In This Repo

Two options depending on where you are in your journey:

### Option A: Single File (Start Here)

`AGENTS-single.md` — A self-contained v6 file (~440 lines) with all process rules and domain knowledge inlined. No dependencies, no skill system. Copy it into your repo as `AGENTS.md` and you're running.

```
your-repo/
├── AGENTS.md          # Rename from AGENTS-single.md
└── tasks/
    ├── todo.md        # Specs, plans, completion state
    └── lessons.md     # Cross-session memory
```

### Option B: Modular with Agent Skills (When One File Outgrows Itself)

`AGENTS.md` — A lean v8 process layer (~350 lines) that references [Agent Skills](https://agentskills.io) by name. Domain knowledge lives in `.agents/skills/` and loads on demand — dozens of skills available, zero context cost until the task activates them.

```
your-repo/
├── AGENTS.md                              # Process — tiers, workflow, security, permissions
├── tasks/
│   ├── todo.md                            # Specs, plans, completion state
│   └── lessons.md                         # Cross-session memory
└── .agents/skills/                        # Domain knowledge — loaded on demand
    ├── security-baseline/SKILL.md
    ├── code-quality/SKILL.md
    ├── testing-strategy/SKILL.md
    ├── api-contracts/SKILL.md
    ├── migration-deprecation/SKILL.md
    ├── data-migration/SKILL.md
    ├── deployment-strategy/SKILL.md
    ├── cross-service-coordination/SKILL.md
    ├── infrastructure-changes/SKILL.md
    ├── observability/SKILL.md
    ├── failure-analysis/SKILL.md
    └── performance-analysis/SKILL.md
```

**One file → maturity → modularize.** Don't over-engineer day one.

## How It Works

### Risk Tiers — Classify Before You Code

Every change is classified by **blast radius** before any code is written:

| Tier | Blast Radius | Example |
|------|-------------|---------|
| **1 — Contained** | One function, file, or module | Bug fix, typo, log change |
| **2 — Local Ripple** | Multiple internal modules | Feature touching 3+ files, internal API extension |
| **3 — Contract/System** | Crosses a trust boundary | API contract change, schema migration, cross-service update |

Same table works whether you're in a monolith, microservices, or a CLI tool — it classifies by impact, not architecture. When in doubt, tier up.

### Explore → Spec → Implement

Depth scales with tier. Sequence is always: **understand → define success → build.**

- **Tier 1:** Read the code. Confirm it's contained. One-line spec. Implement.
- **Tier 2:** Explore with subagents (recommended). Write a spec with acceptance criteria before coding. Implement per spec.
- **Tier 3:** Explore with subagents (mandatory). Write a full spec with contract diffs, rollback strategy, and conditional sections. Get explicit user approval. Implement per approved spec.

The spec is the source of truth. Implementation is measured against it. If the spec is wrong, fix the spec first, then fix the code.

### Security as Principle #1

Not a review step — a hard floor. Applies at all tiers, all times. Hard rules on secrets, input validation, injection prevention, authentication/authorization, cryptography, error hygiene, and dependencies. When "just ship it" conflicts with "validate the inputs," security wins.

### Cross-Session Memory

`tasks/lessons.md` captures systemic failures with a structured format:

```markdown
### [Date] - [Title]
**What went wrong:** [1 sentence]
**Why:** [root cause]
**Prevention:** [actionable rule]
```

Three rules make it work:
1. **High threshold** — Systemic failures and high-impact misses only. Not every trivial correction.
2. **Structured format** — Root cause and actionable prevention, not just "the test failed."
3. **Session start protocol** — Every agent reads it before starting work.

The file lives in the repo, not in any tool's memory. That's why it works across tools.

### Operational Boundaries

Explicit permissions table. Agents can read any file and edit source freely at Tier 1. Destructive operations — deleting files, modifying prod config, committing, pushing — always require explicit confirmation.

## Included Agent Skills

Each skill is a folder with a `SKILL.md` containing metadata and instructions. Agents load only the name and description at startup (~50 tokens per skill) and read full instructions only when the task activates them.

| Skill | When It Activates |
|-------|------------------|
| **security-baseline** | Every change (all tiers) |
| **code-quality** | Code quality and refactoring review lenses |
| **testing-strategy** | Test coverage review lens |
| **api-contracts** | Backward compatibility, versioning, sunset policies |
| **migration-deprecation** | Changing a contract with active consumers |
| **data-migration** | Schema, format, or storage changes |
| **deployment-strategy** | Can't deploy atomically or carries prod risk |
| **cross-service-coordination** | Multiple services must update |
| **infrastructure-changes** | DB, queues, networking, config changes |
| **observability** | Production behavior needs monitoring |
| **failure-analysis** | Partial rollout, async, or distributed changes |
| **performance-analysis** | Latency, throughput, or resource usage changes |

## Getting Started

1. **Pick your starting point.** Single file (`AGENTS-single.md`) or modular (`AGENTS.md` + skills).
2. **Copy into your repo root.** Include `tasks/` with empty `todo.md` and `lessons.md`.
3. **Customize the Tier 3 triggers** for your architecture — the risk tier table works universally, but the architecture-specific triggers should reflect your system boundaries.
4. **Trim skills you don't need.** Solo service with no migrations? Drop `cross-service-coordination` and `data-migration`. Add them back when complexity grows.
5. **Migrating from v6?** See `MIGRATION-v6-to-v8.md`.
6. **Start working.** Your AI coding agent reads `AGENTS.md` automatically, classifies risk, follows the workflow, and activates skills as needed.
7. **Log lessons as you go.** When an agent makes a systemic mistake, add it to `tasks/lessons.md`. Every future session benefits. When an agent makes a systemic mistake, add it to `tasks/lessons.md`. Every future session benefits.

## Comparison: Single File vs. Modular

| | Single File (v6) | Modular + Skills (v8) |
|---|---|---|
| **Files** | 1 file + tasks/ | 1 file + tasks/ + 12 skills |
| **Lines** | ~440 | ~350 (process) + skills load on demand |
| **Domain knowledge** | Inlined | Extracted into `.agents/skills/` |
| **Context cost** | Full cost upfront | ~50 tokens/skill at startup; full load only when activated |
| **Review lenses** | 6 | 7 (adds Architecture) |
| **Best for** | Getting started, smaller projects | Teams at scale, complex architectures |

## Tool Compatibility

`AGENTS.md` is read automatically by every major AI coding agent:

- [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview)
- [Codex CLI](https://github.com/openai/codex)
- [Cursor](https://www.cursor.com/)
- [Kiro](https://kiro.dev/)

Agent Skills (`.agents/skills/`) are an [open standard](https://agentskills.io) supported across these tools.

## License

MIT
