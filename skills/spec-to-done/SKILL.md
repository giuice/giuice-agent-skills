---
name: spec-to-done
description: Route a piece of work through the full long-horizon loop — specify, plan, execute, replan, report — by deciding whether the work warrants the loop, detecting which stage it is already in, and handing off to the right skill. Use this skill whenever the user asks to take something from idea to done, to handle a goal end to end, to run the full workflow, to work on something properly with a plan and evidence, or to resume, continue, or check where a previous run of this workflow stopped.
metadata:
  author: "Giuliano Lemes <giuice@gmail.com>"
---

# Spec to Done

Route work through `SPECIFY → PLAN → EXECUTE ↔ REPLAN → REPORT`.

This skill decides and hands off. It holds **no** planning, execution, or reporting rules — those live in the skills it routes to, and duplicating them here would create a second source of truth that drifts. If you catch yourself explaining how to write a task or a report, stop and hand off instead.

## Step 0 — Does this work need the loop at all?

Most work does not. The loop costs a requirements interview, a plan file, a ledger entry per task, a verification per task, and a replan checkpoint after each one. That overhead buys traceability and resilience over a long horizon. On a small change it buys nothing and just slows the work down.

Run the loop when **at least two** of these hold:

- the work is more than a handful of tasks;
- it will not finish in one sitting, or will outlive one context window;
- getting the requirements wrong is expensive to undo;
- execution is likely to discover things that change the approach;
- someone needs to audit afterwards what was done and on what evidence;
- the work will be handed between people, sessions, or agents.

Otherwise just do the work. Say in one line that you are skipping the loop and why — do not run a ceremony to look thorough.

Between those extremes, offer the middle: run `plan-from-spec` and `execute-plan` on a stated goal, skipping the SPEC interview.

## Step 1 — Locate the work

Derive the slug from the goal (short kebab-case) and look at `spec-interview/<slug>/`. If the user is resuming and the slug is unknown, list the folders and ask which one.

## Step 2 — Route on what exists

| State of `spec-interview/<slug>/` | Hand off to |
|---|---|
| Nothing yet, and the work is product-shaped with unclear requirements | `spec-from-scratch` |
| Nothing yet, and the goal is already clear, or the domain is not product work | `plan-from-spec` — state the goal directly, no SPEC |
| `SPEC.md` exists, no `PLAN.md` | `plan-from-spec`, initial mode |
| `PLAN.md` exists with tasks that have no ledger entry | `execute-plan` |
| Every task is `done` or `no_op`, or a blocker survived replanning | `completion-report` |
| `REPORT.md` exists and matches the ledger | Nothing — state the outcome and ask what is next |

Replanning is not a route. `execute-plan` owns its replan gate and calls `plan-from-spec` itself; do not intercept that loop.

## Step 3 — Hand off cleanly

One stage at a time. Name the skill you are invoking and why, then let it own its own contract, gates, and output. Do not pre-chew its work, and do not second-guess its gates from here — a readiness gate that fails means another round in that stage, not an override from this one.

When a stage escalates to the user — an unreachable acceptance criterion, a destructive action, a missing credential — surface it and stop. Escalations are the user's decision, and routing around one defeats the point of having it.

## Resuming

When the user asks where things stand, read the folder and state it plainly before routing:

```
Goal:        <from SPEC.md or PLAN.md>
Plan:        version <N>, <M> tasks
Progress:    <k> done, <k> unresolved
Last entry:  <task id and status>
Blockers:    <from the ledger, or none>
Next:        <the stage from the routing table>
```

The ledger is the source of truth for progress, not memory of the conversation.

## When the user wants one stage only

Honor it. "Just write me a plan" ends at `plan-from-spec`. "Just tell me what happened" is `completion-report` alone. Do not drag someone through the whole loop because they touched one part of it.
