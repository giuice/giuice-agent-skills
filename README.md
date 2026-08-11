# Giuice Agent Skills

Reusable Open Agent Skills for AI coding agents and agentic workflows.

The main thing here is a **long-horizon workflow**: five skills that take a piece of work from an unclear idea to an evidence-grounded, reported outcome without losing the thread halfway through. Sometimes that outcome is "done and verified"; sometimes it is "blocked here, and this is why" — the workflow is built so both arrive honestly.

```
SPECIFY → PLAN → EXECUTE ↔ REPLAN → REPORT
```

---

## Quick start

Install the workflow:

```bash
npx skills add giuice/giuice-agent-skills --skill spec-to-done spec-from-scratch plan-from-spec execute-plan completion-report
```

Skill names are space-separated, not comma-separated.

Then just say what you want, in your own words:

```
Take this from idea to done: our support inbox needs auto-triage by topic.
```

`spec-to-done` picks it up, decides whether the work is big enough to deserve the loop, and routes you through the stages. **You do not need to know the other skill names.** They also work standalone if you want only one of them.

To pick up work later:

```
Where did we stop on the auto-triage work?
```

---

## What actually happens

| Stage | Skill | What you see |
|---|---|---|
| 0. Route | `spec-to-done` | It tells you whether this work needs the loop, and where you already are in it |
| 1. Specify | `spec-from-scratch` | Rounds of clickable questions until every requirement domain is clear, then `SPEC.md` |
| 2. Plan | `plan-from-spec` | It inspects your actual current state, then writes `PLAN.md` — tasks with observable "done when" conditions |
| 3. Execute | `execute-plan` | One task at a time, each verified against reality, each recorded in `LEDGER.md` |
| 4. Replan | `plan-from-spec` again | After every task: is the rest of the plan still true? Usually yes, and nothing changes |
| 5. Report | `completion-report` | What became true, what was verified, what remains — and nothing else |

Each piece of work gets **its own folder**, named after a slug derived from your goal. Features never share a folder and never collide:

```
spec-interview/
  auto-triage/          ← derived from "auto-triage the support inbox"
    SPEC.md      what counts as success        (optional — a stated goal also works)
    PLAN.md      how to get there              (rewritten as reality intervenes)
    LEDGER.md    what actually became true     (the resume point)
    REPORT.md    what you are told at the end
  billing-export/       ← a different feature, its own independent run
    SPEC.md
    PLAN.md
```

`spec-interview/` is just the parent directory the workflow uses. `auto-triage` is an example slug, not a fixed name — start a new feature and you get a new folder beside it.

Because the state lives in files rather than in the conversation, you can close the session, come back tomorrow, and continue.

---

## When not to use it

Most work does not need this. The loop costs an interview, a plan file, a verification and a ledger entry per task. On a small, reversible change that overhead buys nothing, and `spec-to-done` will tell you so instead of running the ceremony.

The middle path is usually the right one: skip the SPEC interview, state the goal directly, and run plan → execute → report.

---

## The design in one paragraph

Each stage has a strict boundary. The SPEC defines success. The plan is mutable strategy. The ledger is the observable record, written as work happens rather than reconstructed from memory afterwards. The report is a projection of contract × final state × evidence — not a summary of what the agent did. The load-bearing invariant is that **a plan may change, but the contract may not change silently because the plan did**: if execution makes an acceptance criterion unreachable, the workflow stops and asks you, rather than quietly satisfying a weaker goal.

The plan/execute/replan separation follows Erdogan et al., [PLAN-AND-ACT](https://arxiv.org/abs/2503.09572), where dynamic replanning was the single largest ablation gain — evidence that plan quality, not action execution, is the bottleneck on long tasks. The specification gate, the incremental ledger, and the reporting stage are additions.

---

## Skills

### spec-to-done

The entry point. Decides whether work warrants the full loop, detects which stage it is already in by reading the working folder, and routes. Holds no planning or reporting rules of its own.

### spec-from-scratch

Creates a complete product SPEC from an unclear idea through an exhaustive requirements interview — goals, users, scope, business rules, constraints, edge cases, acceptance criteria — and refuses to draft until a readiness gate passes. Useful on its own, whatever you plan to do with the SPEC afterwards.

### plan-from-spec

Turns a SPEC or a stated goal into a plan of outcome-shaped tasks, each with an observable postcondition, and regenerates that plan when execution proves it wrong. Replanning may change the strategy, never the definition of success.

### execute-plan

Executes a plan task by task as an orchestrator: dispatches each task to a subagent, to itself, or to a person, verifies the postcondition itself against observable state, and records what became true in a semantic ledger that doubles as the resume point.

### completion-report

Produces the smallest user-facing report that preserves every material fact about the outcome. No activity narration, no unsupported success claims, no hidden unverified state.

### explorar-planejar-executar

A separate pt-BR workflow, not part of the loop above. Turns vague goals into concrete execution through `/explorar`, `/planejar`, and `/executar`. Lighter weight and more conversational — it keeps its own planning file rather than the four-artifact folder, and has no spec contract, ledger, or verification step.

---

## Domain neutrality

`spec-to-done`, `plan-from-spec`, `execute-plan`, and `completion-report` assume no web, no code, and no repository. They work for research, writing, operations, and physical-world tasks, and they accept a plain stated goal when no SPEC exists.

`spec-from-scratch` is narrower: its interview and output are shaped for product work. For non-product domains, let the router send you straight to `plan-from-spec` with the goal stated directly.

---

## Install

List what is available:

```bash
npx skills add giuice/giuice-agent-skills --list
```

Install everything:

```bash
npx skills add giuice/giuice-agent-skills
```

Install one skill:

```bash
npx skills add giuice/giuice-agent-skills --skill spec-to-done
```
