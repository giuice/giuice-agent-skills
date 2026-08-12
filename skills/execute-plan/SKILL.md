---
name: execute-plan
description: Execute a PLAN.md task by task as an orchestrator, dispatching each task, verifying its postcondition against observable state, recording outcomes in a semantic ledger, and triggering replanning when reality diverges from the plan. Use this skill whenever the user asks to execute, run, carry out, work through, or implement a plan, whenever work should proceed task by task with evidence and state tracking, or whenever resuming a partially executed plan. Requires a PLAN.md produced by plan-from-spec, or any plan with checkable per-task outcomes.
metadata:
  author: "Giuliano Lemes <giuice@gmail.com>"
---

# Execute Plan

Carry out a plan one task at a time, keeping an honest record of what actually became true.

You are the **orchestrator**. You dispatch each task, verify the result against observable state, write the ledger, and decide whether the plan still holds. Verification and the ledger are always yours, whoever performs the work.

This skill is domain-neutral: it executes code work, research, writing, operational, and physical-world plans alike.

## Working directory

```
spec-interview/<slug>/
  SPEC.md      the contract      (read-only here; may be absent — then PLAN.md is the contract)
  PLAN.md      the strategy      (read; replaced by plan-from-spec)
  LEDGER.md    the record        (you are the only writer)
  REPORT.md    written by completion-report
```

## Execution modes

Pick per task, in this order:

**Delegated (default).** One subagent per task. Costs more total tokens — the subagent re-reads context you already hold — and buys two things worth more: your context stays clean, so a long run never hits lossy compaction; and the ledger becomes the only memory channel between tasks, which is what keeps it honest. A ledger written inline duplicates your context and quietly rots.

**Inline.** When the host provides no subagents, when the user asks for it, or when the whole plan is two trivial tasks. Everything else in this skill is unchanged — including verifying the postcondition as a separate act from doing the work.

**Human handoff.** When the task must be performed by a person or outside your reach — a physical action, an approval, an access grant. Present the task, its `Done when`, and what evidence you need in plain language; never hand a person the structured return contract below. Ask only: did it happen, what is now true, and anything unexpected. **You** turn that answer into the ledger entry, recording it as `attested`. Never mark such a task done on the assumption it happened.

---

## The loop

```
read contract + PLAN + LEDGER
   │
   ▼
select next task ──── none left ────► invoke completion-report
   │
   ▼
does its postcondition already hold? ── yes ──┐
   │ no                                       │
   ▼                                          │
dispatch (delegated / inline / human)         │
   │                                          │
   ▼                                          │
verify against observable state               │   ◄── you do this, not the performer
   │                                          │
   ▼                                          ▼
append ledger entry ◄─────────────────────────┘
   │
   ▼
replan gate ──── plan still true ────► next task
   │
   └── plan invalid ──► invoke plan-from-spec in replan mode

Every path reaches the ledger and then the gate. No status skips either.
```

### 1. Select the next task

The first task in `PLAN.md` with **no ledger entry at all**, and whose `Depends on` tasks are all `done` or `no_op`.

**Never redispatch a task that already has any ledger entry.** A `partial`, `blocked`, or `failed` task is unfinished work whose remainder is not yet planned; re-running it repeats side effects. The replanner returns that remainder as a new task with a new ID, so the ledger and the plan never disagree about what has been attempted.

If a dependency is `blocked` or `failed`, do not proceed to its dependents. Go to the replan gate.

### 2. Check before doing

Before dispatching, check whether `Done when` already holds.

This costs one cheap check and closes the gap where a previous run was interrupted after its side effects but before the ledger was written. It also catches work someone else already did.

Which status you record depends on **why** the condition holds:

- Nothing ever attempted this task — it was already true → `no_op`.
- A previous run may have acted and died before recording → `done`, with the observed state delta and a note that it was reconstructed after an interruption. Recording that as `no_op` would erase a real change from the record.

When you cannot tell the two apart, assume the second. Losing a state delta is worse than over-reporting one.

### 3. Dispatch

The performer starts cold, so the brief must stand alone. Send exactly this:

```markdown
## Task
<the full task block from PLAN.md — T-id, Task, Done when, Verify by, Covers>

## Goal context
<the one-sentence goal from PLAN.md>

## Constraints that apply
<constraints, business rules, and non-goals from the contract that bear on this task —
not the whole contract>

## Established facts
<the Discovered entries from LEDGER.md that this task depends on — identities,
paths, versions, decisions already made — plus any fact from the task's Reasoning
the performer needs; for T1, the grounding observation>

## Return contract
Do the work, then verify your own postcondition and return ONLY this block:

status: done | partial | blocked | failed | no_op
state_delta:
- <what is observably different now, and its consequence>
evidence:
- <check performed> → <result>
discovered:
- <fact that later tasks need; omit the section if none>
unresolved:
- <what remains undone; omit the section if none>
risk:
- <what could still go wrong as a result of this work; omit if none>
user_action:
- <what the user must do; omit if none>
deviation: <how the executed work differs from the task as written, and why; omit if none>

Rules:
- Report state, not activity. "Requests over 30s now fail" — not "I edited the client".
- Do not claim a check you did not run.
- Use `no_op` if the postcondition was already satisfied before you started.
- If the task is impossible as written, return `blocked` or `failed` with the reason.
  Do not improvise a different task.
- Stop and return `blocked` before any destructive or irreversible action the user
  has not explicitly asked for.
- Do not write to LEDGER.md, PLAN.md, or the contract.
```

Give the performer the smallest slice of the contract that matters. Dumping the whole SPEC into every brief defeats the purpose of dispatching.

The brief deliberately omits the task's `Reasoning` — that is the planner's justification, not an instruction. If the reasoning contains a fact the performer needs, that fact belongs in `Established facts`.

### 4. Verify — do not take the performer's word

The performer is not the judge of its own success. Before writing the ledger, check the postcondition yourself against observable state:

- Run the `Verify by` check, or confirm the reported evidence is real and current.
- Confirm the `Done when` condition actually holds now.
- If `done` was claimed but the postcondition does not hold, record `failed` with the discrepancy. Do not soften it.

Label the verification, because the reporter depends on the distinction:

| Label | Meaning |
|---|---|
| `verified` | You ran or confirmed a check that observes the postcondition |
| `attested` | A person confirmed it; valid evidence, not machine-checked |
| `unverified` | No check was possible, or confirmation is still pending |

`attested` is a satisfied criterion. `unverified` is not.

**Never record `done` with `unverified`.** If the postcondition cannot be observed at all, the status is `partial` and the unobservable condition goes in `Unresolved`. Otherwise the executor would call a task finished that the reporter must then downgrade, and the run would end claiming a completion the evidence does not support.

### 5. Write the ledger entry

**You are the only writer.** Performers return structured blocks; you validate and append. One writer means one format, no concurrent writes, and a natural point to decide on replanning.

Append to `spec-interview/<slug>/LEDGER.md`:

```markdown
# LEDGER: <slug>

## T1 — Payment client timeout  [done]
Plan version: 1
Covers: FR-004, AC-002
State delta:
- Outbound payment requests now fail after 30 seconds instead of hanging indefinitely.
Evidence:
- integration test PaymentTimeoutTests → passed
Verification: verified
Gate: plan holds

## T2 — Product endpoint cache  [partial]
Plan version: 1
Covers: FR-007, AC-005
State delta:
- The cache abstraction exists and the product endpoint reads through it.
Evidence:
- unit tests → passed (14)
- production Redis behavior → not checked, no access to that environment
Verification: unverified
Discovered:
- The product endpoint is also called by the reporting job, which expects fresh data.
Unresolved:
- Cache invalidation for the reporting path.
Risk:
- The reporting job may read stale data until invalidation is added.
Deviation: TTL set to 60s rather than the planned 300s, because the reporting job
tolerates at most one minute of staleness.
Gate: replan required
```

Ledger rules:

- **Close every entry with `Gate:`** — `plan holds`, `replan required`, or `replan done (plan version N)`. Write `replan required` the moment the gate decides a replan is needed — before invoking the replanner — then overwrite that same line with `replan done (plan version N)` once the new plan is written. If the replanner instead concludes that no valid plan exists — an unroutable blocker, or a repair that would change the contract — overwrite it with `replan exhausted` and exit through the reporter. The `Gate:` line is the one field updated in place; the append-only rule below does not apply to it. This is the only durable record that the loop reached its checkpoint; without it, a run interrupted between a failed task and its replan looks identical to one that simply stopped, and nothing can tell which.
- **On a run with no `Covers`** — no SPEC, so the contract is the union of the plan's `Done when` — copy the task's `Done when` into the entry instead, along with its `Restates:` line when present. Completed tasks leave the plan, so this is the only place the satisfied part of the contract survives.

- **Write it after every task, never reconstruct it at the end.** A ledger rebuilt from memory at the end of a long run is exactly the semantic loss this workflow exists to prevent.
- **Copy `Covers` verbatim from the plan.** It is the only path from task evidence back to acceptance criteria once the task leaves the plan. Omit when the plan has no `Covers`.
- **State over activity.** "Authentication now rejects expired tokens" — not "edited AuthService". Filenames are optional traceability, appended after the consequence.
- **Evidence, not reasoning.** Record what was checked and what it returned. Do not record deliberation.
- **Record deviations honestly.** A silent deviation becomes a hidden contract change.
- **Destructive or irreversible actions and changes to user data are always state deltas.** Never leave one implicit.
- A recovered transient error with no residual consequence may be omitted. An error that changed the final state must be recorded.
- Append only. Never rewrite a past entry — its `Gate:` line excepted, per the rule above. If a later task changes an earlier outcome, write a new entry saying so, naming the `Covers` ID or `Done when` it undoes.

### 6. Status transitions

**This table is normative.** Where the diagram, the prose, or the anti-patterns seem to say otherwise, the table wins. Every status has exactly one continuation, and no status leaves the loop undefined.

| Status | Ledger | Then |
|---|---|---|
| `done` | record with verification label | replan gate → next task |
| `no_op` | record with the evidence that it was already true | replan gate → next task |
| `partial` | record, with `Unresolved` filled | replan gate **must** run — the remainder needs a new task |
| `blocked` | record, with the blocker and any `user_action` | replan gate **must** run; if it cannot route around it → report |
| `failed` | record, with the discrepancy | replan gate **must** run |

### 7. Replan gate

Run this after **every** task, without exception. It is a checkpoint, not a rewrite: most gates should pass without a replan.

Ask: *is the remaining plan still true, given what is now known?*

Invoke `plan-from-spec` in replan mode when any of these holds:

```
- the task returned partial, blocked, or failed
- a postcondition failed and a retry will not fix it
- an expected file, resource, person, capability, or state does not exist
- something discovered invalidates a later task
- a task turned out to be impossible as written
- a materially shorter valid path became available
- a new constraint surfaced during execution
- an acceptance criterion requires work no task covers
- a deviation changed what later tasks can assume
```

Otherwise state that the plan still holds and continue.

**Mid-task trigger.** Do not wait for the task to end when the divergence is already fatal. If a performer reports `blocked` or `failed` with a reason that invalidates the plan, replan immediately rather than dispatching the next task into a plan you know is wrong.

### 8. Exit

Stop the loop and invoke `completion-report` when:

- every task is `done` or `no_op`; or
- the plan has zero tasks because the goal was already satisfied; or
- a task is `blocked` and replanning cannot route around it; or
- the plan cannot be repaired without changing the contract; or
- the user asks to stop.

Never stop silently. Every exit goes through the reporter.

---

## Escalation

Stop and ask the user — do not decide alone — when:

- an acceptance criterion has become unreachable;
- an action would be destructive or irreversible and the user has not explicitly asked for it;
- the work has drifted far enough from the contract that finishing it would satisfy a different goal;
- a blocker needs access, credentials, or a decision only the user has.

Report the state through `completion-report` rather than improvising a new objective.

---

## Resuming

`LEDGER.md` is the resume point. On restart, read the contract, `PLAN.md`, and `LEDGER.md`. **First look at the last entry's `Gate:` line**: if it is missing, the run died before its checkpoint — run the replan gate for that entry now; if it says `replan required`, compare the entry's `Plan version` to `PLAN.md`'s — a higher plan version means the replan already completed and only the Gate was left open, so just overwrite it with `replan done (plan version N)`; otherwise invoke the replanner now. The same comparison reconciles a replan the user ran standalone. Only then continue from the first task with no ledger entry — running the step 2 pre-check before dispatching it, since a prior run may have acted without recording.

Do not re-run completed tasks. Re-verify a completed task only when a later discovery may have invalidated its postcondition.

---

## Anti-patterns

**Trusting the return.** Accepting `status: done` without checking the postcondition. The performer is optimistic about its own work; that is the whole reason for the verification step.

**Redispatching unfinished tasks.** Re-running a `partial` task from the top repeats its side effects. The remainder is new work and needs a new task.

**Batch ledger writing.** Running five tasks and then writing five entries from memory. The details that mattered are already gone.

**Activity ledgers.** Entries that list what was touched instead of what became true. The report cannot be built from those.

**Skipping the replan gate when things are going well.** The plan is most often wrong exactly when execution feels smooth — because nothing has forced you to look at it.

**Fixing the plan inline.** When the plan is wrong, invoke the replanner. Silently doing something other than what the plan says produces a run nobody can audit.

**Reinventing the todo list.** `PLAN.md` is the durable artifact. Any ephemeral task-tracking UI is a view of it, never a second source of truth.
