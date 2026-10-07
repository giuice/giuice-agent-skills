# plan-from-spec

## Current workflow

For new work, install the self-contained `spec-to-done` from [giuice/spec-to-done](https://github.com/giuice/spec-to-done) ([skills.sh](https://www.skills.sh/giuice/spec-to-done/spec-to-done)):

```bash
npx skills add giuice/spec-to-done --skill spec-to-done
```

One installation includes specification, planning, execution/replanning, and reporting. You do not need separate `plan-from-spec`, `execute-plan`, or `completion-report` installations. Ask the current composite for planning only: it satisfies missing prerequisites, writes the plan, and stops before execution.

This directory retains the legacy planning skill for existing users. Before planning or execution, the current composite checks SPEC readiness and user approval; existing specifications must satisfy its prerequisites. It uses `TRACK.md` / `SNAPSHOT.md` instead of this workflow's `LEDGER.md`; do not assume an existing run is directly interchangeable. Preserve its artifacts and review the current workflow before switching.

## Legacy skill

Turn a SPEC or a stated goal into a plan of outcome-shaped tasks, and regenerate that plan when execution proves it wrong.

## When to use

- "Break this spec into a plan"
- "Plan the work for this goal"
- "The plan is wrong, redo it from here"
- Automatically, when `execute-plan` hits a replan trigger

## What it does

Two modes, one output format.

**Initial** — observes the actual current state first (never plans from the goal alone), then decomposes into tasks. Each task states WHAT must become true, with a `Done when` postcondition and a `Verify by` check. That postcondition is what lets an executor detect divergence in domains where state is not observable for free.

**Replan** — reflects on what actually happened according to the ledger, then makes remaining tasks specific with newly discovered information, drops what is no longer needed, and adds what the current state revealed. Task IDs stay stable across versions so the ledger keeps joining to the plan.

The hard rule: **replanning changes the strategy, never the definition of success.** If a goal has become unreachable as specified, the skill stops and escalates rather than quietly planning toward a weaker goal.

## Artifacts

Writes `spec-interview/<slug>/PLAN.md`, alongside the `SPEC.md` that `spec-from-scratch` produces.

A SPEC is optional. With a plain stated goal, the union of the tasks' `Done when` conditions — satisfied ones living on in the ledger — becomes the acceptance contract, which leaves that contract extendable by replanning, the one thing a SPEC prevents. The skill says so out loud rather than hiding it.

## Related

Part of `SPECIFY → PLAN → EXECUTE ↔ REPLAN → REPORT`, with [spec-from-scratch](../spec-from-scratch), [execute-plan](../execute-plan), and [completion-report](../completion-report). [spec-to-done](../spec-to-done) is the entry point if you would rather not name the stage yourself.

The plan/execute/replan separation follows Erdogan et al., [PLAN-AND-ACT: Improving Planning of Agents for Long-Horizon Tasks](https://arxiv.org/abs/2503.09572), where dynamic replanning was the single largest ablation gain — evidence that plan quality, not action execution, is the bottleneck on long-horizon tasks.

## Legacy installation

```bash
npx skills add giuice/giuice-agent-skills --skill plan-from-spec
```
