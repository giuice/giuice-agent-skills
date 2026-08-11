# plan-from-spec

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

A SPEC is optional. With a plain stated goal, the union of every task's `Done when` becomes the acceptance contract — which also makes that contract mutable by replanning, the one thing a SPEC prevents. The skill says so out loud rather than hiding it.

## Related

Part of `SPECIFY → PLAN → EXECUTE ↔ REPLAN → REPORT`, with [spec-from-scratch](../spec-from-scratch), [execute-plan](../execute-plan), and [completion-report](../completion-report). [spec-to-done](../spec-to-done) is the entry point if you would rather not name the stage yourself.

The plan/execute/replan separation follows Erdogan et al., [PLAN-AND-ACT: Improving Planning of Agents for Long-Horizon Tasks](https://arxiv.org/abs/2503.09572), where dynamic replanning was the single largest ablation gain — evidence that plan quality, not action execution, is the bottleneck on long-horizon tasks.

## Install

```bash
npx skills add giuice/giuice-agent-skills --skill plan-from-spec
```
