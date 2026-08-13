# Handoff: composite `spec-to-done` skill

Date: 2026-08-13

## Objective

Create a new experimental repository for a single composite `spec-to-done` skill that can take substantial work from an unclear request through specification, planning, execution, replanning, verification, and reporting.

The immediate reliability target is less-capable agents. They currently fail at two points:

1. **Weak implicit activation.** A natural request such as "create a task-management website" may not activate the existing `spec-to-done`, because its description emphasizes phrases such as "idea to done", "full workflow", and "with a plan and evidence" instead of ordinary build/create/implement requests.
2. **Fragile cross-skill handoffs.** Even when `spec-to-done` activates and correctly determines that `spec-from-scratch` is required, a less-capable agent may only say that the other skill should be invoked. It does not actually invoke and execute it.

Skill availability is not the problem in the reproduced case: the relevant skills were installed.

## Decision

Build a **new repository containing one composite skill**, not a collection of five independent runtime skills.

Proposed repository:

```text
giuice/spec-to-done
```

Proposed layout:

```text
spec-to-done/
├── README.md
├── LICENSE
├── skills/
│   └── spec-to-done/
│       ├── SKILL.md
│       ├── agents/
│       │   └── openai.yaml
│       ├── references/
│       │   ├── specify.md
│       │   ├── plan.md
│       │   ├── execute.md
│       │   └── report.md
│       └── scripts/
│           └── detect-stage.py
└── evals/
    ├── cases.yaml
    ├── rubric.md
    └── fixtures/
```

Keep the skill under `skills/spec-to-done/`, even if it is initially the repository's only skill. This is a conventional layout for the `skills` CLI and ensures that the installed skill includes its supporting files. Avoid a root-level `SKILL.md`, because root-skill installation has had limitations around installing sibling resources.

## Why one composite skill

Installing several skills from one repository helps distribution but does not fix cross-skill activation. They remain independent skills, so the model still has to discover and activate the next one.

The new execution model is:

```text
one activation of spec-to-done
  → read references/specify.md and execute it
  → read references/plan.md and execute it
  → read references/execute.md and execute it
  → read references/report.md and execute it
```

It replaces:

```text
activate spec-to-done
  → try to activate spec-from-scratch
  → try to activate plan-from-spec
  → try to activate execute-plan
  → try to activate completion-report
```

The files under `references/` are internal stage instructions, not nested skills. Do not put additional `SKILL.md` files below the composite skill.

## Protect the existing `spec-from-scratch`

The existing `skills/spec-from-scratch` is tested, distributed through skills.sh, and must remain unchanged.

For the new composite repository:

- copy its relevant workflow into `references/specify.md`;
- treat that copy as an intentional, pinned snapshot;
- record the upstream repository and commit in a comment at the beginning of the reference;
- do not automatically synchronize it with the standalone skill;
- let the composite version evolve only when composite-workflow evaluations justify a change.

Suggested provenance header:

```markdown
<!--
Derived from giuice/giuice-agent-skills skills/spec-from-scratch.
Upstream snapshot: <commit SHA>
This embedded version belongs to the composite spec-to-done workflow.
-->
```

The standalone and composite versions have different responsibilities:

- standalone `spec-from-scratch`: produce a SPEC as an independent deliverable;
- composite `spec-to-done`: run an end-to-end workflow whose first stage happens to specify the product.

## Activation requirement

The new description must front-load ordinary user language. It should activate for substantial product or project work even when the user does not mention a spec, plan, workflow, or "idea to done".

Starting draft:

```yaml
description: Take substantial product and project work from an unclear request
  through requirements, planning, execution, replanning, verification, and final
  reporting. Use when the user asks to create, build, implement, launch, redesign,
  migrate, or handle a site, app, feature, system, workflow, or other multi-step
  outcome end to end—even when they do not mention a spec or plan. Also use when
  resuming previous spec-to-done work. Skip only small, reversible changes or when
  the user explicitly requests a single stage.
```

This description is only a starting hypothesis. Test it against realistic prompts and refine it based on activation failures.

## Root `SKILL.md` responsibility

Keep the root skill concise. It should own:

- activation boundary;
- new-work size gate;
- persisted-state detection;
- the main stage loop;
- cross-stage continuation;
- escalation to the user;
- the invariant that a stage boundary is not a user-facing stopping point.

It should route to internal references with language like:

```markdown
Determine the current stage from the persisted artifacts.

- Specify: read `references/specify.md` completely and follow it.
- Plan: read `references/plan.md` completely and follow it.
- Execute or replan: read `references/execute.md` completely and follow it.
- Report: read `references/report.md` completely and follow it.

A stage boundary is not a user-facing stopping point. Continue immediately to
the next stage unless user input, permission, credentials, or an external action
is required.
```

The stage reference owns its detailed procedure. Do not duplicate stage rules in the root skill.

## Deterministic routing script

Use `scripts/detect-stage.py` to remove fragile artifact-state interpretation from the model. It should inspect one work folder and emit structured data such as:

```json
{
  "stage": "specify",
  "reason": "No SPEC.md or interview state exists",
  "next_reference": "references/specify.md"
}
```

Initial supported states should be deliberately small:

```text
empty folder                   → specify
interview state, no SPEC       → specify
SPEC, no PLAN                  → plan
PLAN with pending tasks        → execute
all tasks terminal             → report
current REPORT                 → complete
ambiguous or unsafe state      → needs-user
```

Do not implement advanced lineage, replan exhaustion, or reopening in the first version unless needed for a minimal replan evaluation.

## MVP scope

The first version must prove only the happy path:

```text
vague product request
  → requirements interview
  → SPEC.md
  → PLAN.md
  → simple execution
  → verification and LEDGER.md
  → REPORT.md
```

It must also prove that the workflow resumes after user answers during the interview.

Defer until the happy path works reliably on target models:

- blocker identities;
- continuation lineages;
- hard attempt ceilings;
- `replan exhausted`;
- post-terminal reopening;
- elaborate gate reconciliation;
- rare interruption windows.

Add those only in response to reproduced evaluation failures.

## Evaluation plan

Keep evaluations outside the installed skill under repository-level `evals/`.

Start with these prompt classes:

### Activation

Should activate:

```text
Create a task-management website for a small agency.
Build an app where a family can organize household chores.
Implement a customer-support triage workflow from start to finish.
Take this migration from the idea through a verified result.
Resume the task-management app work from yesterday.
```

Should not activate:

```text
Fix this typo in README.md.
Rename this local variable.
Explain what this function does.
Just write a plan; do not execute it.
```

### Handoff/continuity

For a vague website request, the agent must begin the specification interview. It must not:

- start implementing the site;
- stop after saying a SPEC is missing;
- tell the user to invoke another skill;
- fabricate a SPEC without interviewing;
- terminate at the SPEC or PLAN boundary when no user input is required.

### Test matrix

- one strong model as a control;
- at least two less-capable models intended to be supported;
- fresh session and clean fixture for every run;
- at least three runs per prompt/model pair.

Measure:

- correct implicit activation;
- correct stage selection;
- actual continuation rather than merely naming the next stage;
- expected artifact creation;
- no implementation before requirements are ready;
- no user-facing stop at an internal stage boundary;
- honest final status and evidence.

Do not evaluate primarily by prose style.

## Relationship to PLAN-AND-ACT

The architecture should retain the paper's core:

- high-level Planner separated from low-level execution;
- plan grounded in the observed current state;
- tasks as meaningful high-level units rather than raw actions;
- replan only future work when reality diverges;
- let the evolving plan carry verified discoveries forward.

SPEC, ledger, independent verification, and reporter are deliberate extensions. They should support the core rather than dominate the first implementation.

Relevant local source:

- `docs/plan-and-act-paper.md`

## Conventions and references

- Agent Skills specification: https://agentskills.io/specification
- OpenAI skill authoring: https://developers.openai.com/plugins/build/skills
- `skills` CLI repository and discovery conventions: https://github.com/vercel-labs/skills

Key conventions:

- one skill directory contains one required `SKILL.md`;
- name and description are the initial activation surface;
- keep `SKILL.md` concise and use `references/` for detailed stage procedures;
- use `scripts/` for fragile decisions that benefit from deterministic behavior;
- include `agents/openai.yaml` for UI metadata;
- do not place README or development/evaluation material inside the installed skill folder;
- validate the skill and forward-test it with fresh, minimally informed agents.

## Current repository state

At handoff time, the current repository has uncommitted work unrelated to creating the new repository:

```text
M  README.md
M  skills/execute-plan/SKILL.md
M  skills/plan-from-spec/SKILL.md
M  skills/spec-to-done/SKILL.md
?? docs/why-multi-agent-pipelines-fail.md
```

Do not discard, reset, commit, or move those changes as part of creating the new repository. The existing `skills/spec-from-scratch` is not modified in this working tree.

Do not create a nested Git repository inside `giuice-agent-skills`. Create the new repository in a separate local directory chosen by the user.

## First actions in the next session

1. Read this handoff completely.
2. Confirm the separate local path and final repository name with the user.
3. Use the `skill-creator` workflow and initialize the new skill with its provided initializer.
4. Create only the repository skeleton and minimal composite `SKILL.md` first.
5. Vendor the pinned specification-stage snapshot without changing the existing standalone skill.
6. Implement the minimal deterministic stage detector.
7. Add the happy-path evaluation fixtures before adding advanced recovery protocol.
8. Validate the skill directory and run forward tests against the target less-capable agents.

## Suggested new-session prompt

```text
Read docs/spec-to-done-composite-handoff.md completely. Help me create the new
standalone giuice/spec-to-done repository described there. Keep the existing
spec-from-scratch skill untouched. Start with the repository skeleton, the single
composite skill, the broad activation description, the minimal happy-path state
detector, and evaluations for less-capable agents. Do not bring the advanced
lineage/reopening protocol into the MVP unless an evaluation demonstrates that it
is needed.
```
