# spec-to-done

## Current workflow

For new work, install the self-contained `spec-to-done` from [giuice/spec-to-done](https://github.com/giuice/spec-to-done) ([skills.sh](https://www.skills.sh/giuice/spec-to-done/spec-to-done)):

```bash
npx skills add giuice/spec-to-done --skill spec-to-done
```

One installation includes specification, planning, execution/replanning, and reporting. You do not need separate `plan-from-spec`, `execute-plan`, or `completion-report` installations. For an end-to-end request, use the current composite workflow. For an explicitly requested single stage, preserve that stopping boundary.

This directory retains the legacy router skill for existing users. Before planning or execution, the current composite checks SPEC readiness and user approval; existing specifications must satisfy its prerequisites. It uses `TRACK.md` / `SNAPSHOT.md` instead of this workflow's `LEDGER.md`; do not assume an existing run is directly interchangeable. Preserve its artifacts and review the current workflow before switching.

## Legacy skill

The entry point to the long-horizon workflow. Decides whether the work warrants it, finds which stage the work is already in, and hands off.

## When to use

- "Take this from idea to done"
- "Handle this end to end, properly"
- "Run the full workflow on this"
- "Where did we stop?" / "Resume that work"

## What it does

Three things, and deliberately nothing else:

1. **State detection, first.** Reads `spec-interview/<slug>/` and works out which stage the work is in — including whether a run died mid-loop and whether the folder's stored goal is actually the goal you are asking about. None of the four stage skills can do this; each only knows its own exit.
2. **A YAGNI gate, for new work only.** Most new work does not deserve a spec interview, a plan file, and a ledger entry per task. The skill says so and steps aside, or offers the middle path — plan and execute against a stated goal, no SPEC. Work already in progress never gets gated out; finishing it outside its own contract would strand the record it was keeping.
3. **Routing.** Hands off to `spec-from-scratch`, `plan-from-spec`, `execute-plan`, or `completion-report`, then gets out of the way.

## What it deliberately does not do

It contains no planning, execution, or reporting rules. Those live in the stage skills. A router that restates the rules it routes to becomes a second source of truth and drifts from the first one.

It also does not intercept replanning — `execute-plan` owns its replan gate and calls the planner itself — and it does not override a stage's readiness gate or route around an escalation.

One exception it does own: a **non-product** goal that is still vague. `spec-from-scratch` would force user journeys and launch criteria onto research or operational work, so the router asks the few questions needed to produce one checkable outcome condition, then hands to `plan-from-spec`.

## Related

The router for [spec-from-scratch](../spec-from-scratch), [plan-from-spec](../plan-from-spec), [execute-plan](../execute-plan), and [completion-report](../completion-report).

## Legacy installation

```bash
npx skills add giuice/giuice-agent-skills --skill spec-to-done
```
