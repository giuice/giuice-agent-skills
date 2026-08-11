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

### plan-from-spec

Turns a SPEC or stated goal into a plan of outcome-shaped tasks, each with an observable postcondition, and regenerates that plan when execution proves it wrong. Replanning may change the strategy, never the definition of success.

### execute-plan

Executes a plan task by task as an orchestrator: dispatches each task to a subagent, to itself, or to a person, verifies the postcondition itself against observable state, and records what became true in a semantic ledger that doubles as the resume point.

### completion-report

Produces the smallest user-facing report that preserves every material fact about the outcome — a projection of specification × final state × evidence, not a summary of the execution trace.

## The long-horizon workflow

The last four skills compose into one loop:

```
SPECIFY → PLAN → EXECUTE ↔ REPLAN → REPORT
```

Each stage has a strict boundary: the SPEC defines success, the plan is mutable strategy, the ledger is the observable record, and the report is the user-facing projection of that record. A plan may change; the contract may not change silently because the plan did.

The plan/execute/replan separation follows Erdogan et al., [PLAN-AND-ACT](https://arxiv.org/abs/2503.09572); the specification gate, the incremental ledger, and the reporting stage are additions.

`plan-from-spec`, `execute-plan`, and `completion-report` are domain-neutral — they carry no assumption of web, code, or a repository, and they accept a plain stated goal when no SPEC exists. `spec-from-scratch` is narrower: its interview and output are shaped for product work. For non-product domains, start at `plan-from-spec` with the goal stated directly.

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
npx skills add giuice/giuice-agent-skills --skill plan-from-spec
npx skills add giuice/giuice-agent-skills --skill execute-plan
npx skills add giuice/giuice-agent-skills --skill completion-report
```
