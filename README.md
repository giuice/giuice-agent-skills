# Giuice Agent Skills

Reusable Open Agent Skills for AI coding agents and agentic workflows.

## Skills

### explorar-planejar-executar

A pt-BR workflow that turns vague goals into concrete execution through three explicit phases:

- `/explorar` — guided discovery and context gathering
- `/planejar` — approach selection and atomic task planning
- `/executar` — controlled step-by-step execution

Use it when a user has an open-ended objective that needs exploration, planning, or structured execution.

### spec-from-scratch

A requirements-discovery workflow for creating a complete product SPEC from an unclear idea.

It runs an exhaustive interview before drafting, covering goals, users, scope, requirements, business rules, constraints, edge cases, and acceptance criteria. Use it when a user wants to create, clarify, scope, or write a SPEC/PRD without hidden assumptions.

## Install

List available skills:

```bash
npx skills add giuice/giuice-agent-skills --list
```

Install all skills:

```bash
npx skills add giuice/giuice-agent-skills
```

Install a specific skill:

```bash
npx skills add giuice/giuice-agent-skills --skill explorar-planejar-executar
npx skills add giuice/giuice-agent-skills --skill spec-from-scratch
```
