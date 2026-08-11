# execute-plan

Execute a plan task by task as an orchestrator: delegate, verify, record, decide whether to replan.

## When to use

- "Execute the plan"
- "Work through PLAN.md"
- "Resume where we stopped"

## What it does

The agent running this skill orchestrates rather than performs. It:

1. picks the next task whose dependencies are met;
2. checks whether the postcondition already holds — which also makes an interrupted run safe to resume;
3. dispatches the task with a self-contained brief and a structured return contract;
4. **verifies the postcondition itself** against observable state — the performer is not the judge of its own success;
5. appends an entry to `LEDGER.md` as the sole writer;
6. runs a replan gate after every task, and immediately when a divergence is already fatal;
7. exits through `completion-report`, never silently.

## Three dispatch modes

**Delegated** (default) — one subagent per task. **Inline** — when the host has no subagents or the plan is trivial. **Human handoff** — when the task needs a person: a physical action, an approval, an access grant. Verification and the ledger stay with the orchestrator in all three.

Delegation costs more tokens, not fewer — the subagent re-reads context the orchestrator already holds. It buys two things worth more: the orchestrator's context stays clean, so a long run never hits lossy compaction; and the ledger becomes the only memory channel between tasks, which is what keeps it honest. Written inline, a ledger duplicates context and quietly rots.

## Artifacts

Reads the contract (`SPEC.md`, or `PLAN.md` when no SPEC exists) and `PLAN.md`. Writes `LEDGER.md` — the semantic record of what became true, entry by entry, which doubles as the resume point.

The ledger records **state, not activity**: "requests over 30s now fail", not "edited the HTTP client". Each entry carries the plan's `Covers` IDs, so evidence still joins to acceptance criteria after the task leaves the plan. Deviations, risks, and required user actions are recorded explicitly, because a silent one is a hidden contract change.

## Related

Part of `SPECIFY → PLAN → EXECUTE ↔ REPLAN → REPORT`, with [spec-from-scratch](../spec-from-scratch), [plan-from-spec](../plan-from-spec), and [completion-report](../completion-report).

## Install

```bash
npx skills add giuice/giuice-agent-skills --skill execute-plan
```
