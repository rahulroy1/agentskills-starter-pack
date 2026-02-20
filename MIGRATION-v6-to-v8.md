# Migration Guide: v6 (Single File) → v8 (Modular)

## What Changed

| Area | v6 (AGENTS-single.md) | v8 (AGENTS.md + skills) |
|------|------------------------|-------------------------|
| Security baseline | Inlined in §10 | Moved to `security-baseline` skill |
| Review lenses | 6 lenses | 7 lenses (added Architecture) |
| Skill activation | N/A | Explicit protocol added |
| Tier classification | Boundary test only | Added decision tree and examples |
| Spec approval | "Explicit approval" | Spec approval protocol with steps |
| Error escalation | Brief mention | Escalation template added |
| Production skills | N/A | `cross-service-coordination`, `observability`, `infrastructure-changes` consolidated into `production-readiness` |
| Task file format | Unstructured | Templates with defined schema in `tasks/todo.md` and `tasks/lessons.md` |

## Migration Steps

1. **Copy AGENTS.md (v8)** to your repo root.
2. **Copy `.agents/skills/`** directory with all skills.
3. **Create `tasks/`** with empty `todo.md` and `lessons.md`.
4. **Update custom rules** to use "Activate: `skill-name`" format.
5. **Test** with a Tier 3 change to verify workflow.

## Choosing Your Starting Point

- **Use v6 (single file)** if: New repo, small team, no Tier 3 changes yet.
- **Use v8 (modular)** if: Multiple agents, Tier 3 changes common, context budget matters.

You can start with v6 and migrate to v8 when the single file exceeds ~500 lines or skill activation becomes valuable.
