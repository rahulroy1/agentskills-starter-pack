# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is the **Agent Skills Starter Pack** — an operational framework that defines how AI coding agents should work. It is not an application with build/test commands. It is a set of Markdown-based guidelines and modular skills designed to be read by AI coding tools (Claude Code, Codex CLI, Cursor, Kiro) at session start.

Based on the open standard at agentskills.io.

## Repository Structure

- **AGENTS.md** — Core operational framework (risk tiers, workflow, security, verification, bug fixing, review lenses)
- **.agents/skills/** — 13 modular domain-specific skill files (SKILL.md format with YAML frontmatter)
- **tasks/todo.md** — Task tracking template (Tier 2/3 specs, progress, status)
- **tasks/lessons.md** — Cross-session learnings (systemic failures and prevention rules)

## Key Concepts

### Risk Tiers (classify BEFORE coding)
- **Tier 1 — Contained:** Single function/file. Read → brief spec → implement.
- **Tier 2 — Local Ripple:** Multiple internal modules. Explore → spec with acceptance criteria → implement. Mandatory gates.
- **Tier 3 — Contract/System Boundary:** Affects consumers, APIs, schemas. Mandatory subagent exploration → full spec → user approval → implement. All 7 review lenses required.

### Core Workflow: Explore → Spec → Implement
Code is always last. Spec depth scales with tier. Tier 3 specs require explicit user approval before implementation begins.

### Skills Loading
Skills in `.agents/skills/` activate conditionally based on the type of change (e.g., `api-contracts` for API changes, `data-migration` for schema changes, `security-baseline` for all tiers). Each skill has: checklist, patterns, verification steps, and anti-patterns.

### Skill File Format
```yaml
---
name: kebab-case-name
description: >
  One-line activation description
---
```
Followed by: Related Skills, Checklist, Patterns, Verification, Anti-Patterns sections.

## Working in This Repo

Changes here are to the framework itself (Markdown content). When editing:

- Preserve the tiered structure — guidance must remain classifiable by blast radius
- Skills must remain self-contained with standard sections (checklist, patterns, verification, anti-patterns)
- Keep YAML frontmatter in skill files valid
- Cross-references between skills (Related Skills sections) must stay consistent
- `tasks/lessons.md` entries use the format: date, category, what went wrong, why, prevention rule
- `tasks/todo.md` entries use the format: title, tier, status, date, spec, approval, progress, notes
