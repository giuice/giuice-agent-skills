# Completion Reporter Skill — Design Proposal

> A deterministic post-task reporting layer for Plan-and-Act style agent workflows.

## 1. Purpose

The **Completion Reporter** is the final communication layer between an autonomous agent workflow and the user.

Its job is **not to summarize the execution trace**.

Its job is to produce the **smallest user-facing report that preserves every material fact about the task outcome**.

The core problem it solves is common in coding agents and long-horizon agents:

- verbose narration of actions that do not matter to the user;
- omission of important context during compression;
- claims of success without verification;
- reports that list changed files but do not explain the behavioral consequence;
- reports that repeat the original task instead of reporting its outcome;
- summaries that are short but semantically incomplete.

The Reporter therefore operates as a **semantic projection layer**, not as a generic summarizer.

---

## 2. Position in the Workflow

This model extends a Plan-and-Act style architecture with an explicit reporting stage.

```text
USER REQUEST
     │
     ▼
┌──────────────────────┐
│ SPECIFICATION SKILL  │
│                      │
│ Goal                 │
│ Constraints          │
│ Acceptance criteria  │
│ Non-goals            │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ PLANNER              │
│                      │
│ Produces high-level  │
│ executable plan      │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ EXECUTOR             │◄─────────────────────┐
│                      │                      │
│ Executes plan steps  │                      │
│ Collects evidence    │                      │
│ Updates task state   │                      │
└──────────┬───────────┘                      │
           │                                  │
           │ replan required                  │
           ▼                                  │
┌──────────────────────┐                      │
│ REPLANNING           │                      │
│                      │                      │
│ Rebuilds remaining   │──────────────────────┘
│ plan from new state  │
└──────────────────────┘

           │ task boundary reached
           ▼
┌──────────────────────┐
│ SEMANTIC LEDGER      │
│                      │
│ Outcome              │
│ State deltas         │
│ Evidence             │
│ Unresolved items     │
│ Material events      │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ COMPLETION REPORTER  │
│                      │
│ Completion gate      │
│ Materiality filter   │
│ Coverage check       │
│ Language renderer    │
└──────────┬───────────┘
           │
           ▼
         USER
```

The complete runtime model becomes:

```text
SPECIFY → PLAN → EXECUTE ↔ REPLAN → REPORT
```

The reporting stage is intentionally isolated from execution.

---

## 3. Architectural Principle

The system separates five different concerns.

| Layer | Responsibility |
|---|---|
| Specification | Define what must be true when the task is complete |
| Planner | Decide the high-level path toward that state |
| Executor | Perform concrete actions in the environment |
| Replanning | Adapt the remaining plan when reality diverges from assumptions |
| Reporter | Communicate the resulting state to the user |

These responsibilities must not leak into each other.

### Critical invariant

```text
SPECIFICATION = task contract
PLAN          = mutable strategy
EXECUTION     = observable actions
LEDGER        = observable task state
REPORT        = user-facing projection of that state
```

A plan may change.

The task contract must not silently change because the plan changed.

Replanning may modify **how** the goal is achieved. It must not silently redefine **what counts as success**.

---

## 4. Why the Reporter Must Be Separate

A generic Executor is optimized for local action selection.

During a long task it may need to reason about:

- files;
- commands;
- browser state;
- tools;
- intermediate errors;
- temporary hypotheses;
- retries;
- environment state;
- implementation details.

Most of this information should never reach the user.

However, asking the same Executor to simply:

```text
Summarize what you did concisely.
```

creates a lossy compression problem.

The model must simultaneously decide:

1. what happened;
2. what matters;
3. what can be omitted;
4. whether completion was actually verified;
5. how much context the user requires;
6. how to phrase the result.

This makes omission and narration likely.

The Completion Reporter isolates these decisions and gives them an explicit protocol.

---

# 5. Semantic Ledger

The Reporter should not reconstruct task history from scratch.

The workflow maintains a **Semantic Ledger** while execution occurs.

The ledger is structured state.

It is not prose.

It is not chain-of-thought.

It should contain only externally useful facts, observations, decisions, state changes, and evidence.

Example:

```yaml
task:
  id: TASK-42
  objective: Add authenticated user lookup by email
  status: completed

contract:
  acceptance_criteria:
    - endpoint returns matching authenticated user
    - public API remains backward compatible
    - project builds
    - relevant tests pass

state_delta:
  - type: behavior
    change: authenticated users can now be resolved by email
  - type: interface
    change: IUserRepository gained an email lookup operation
    consequence: repository implementations must support the operation

verification:
  - check: dotnet build
    status: passed
  - check: relevant test suite
    status: passed
    evidence: 38 tests passed

unresolved: []

failures_recovered:
  - event: nullable warning detected
    consequence: none
    final_state: resolved

user_action_required: []
```

The final report is generated from this representation.

It is **not generated from a raw action transcript** unless the ledger is missing required evidence.

---

## 6. Ledger Rules

### 6.1 Incremental state

The ledger must be updated during execution.

Do not regenerate the ledger only at the end from memory.

```text
BAD

100 actions
   ↓
LLM reconstructs what mattered
   ↓
final summary
```

```text
BETTER

action
  ↓
observable state delta
  ↓
ledger update
  ↓
next action
  ↓
observable state delta
  ↓
ledger update
  ...
```

This reduces semantic loss in long-horizon tasks.

### 6.2 Evidence, not hidden reasoning

The ledger may store:

- observed facts;
- decisions;
- constraints;
- test results;
- errors;
- state changes;
- artifacts;
- user-visible consequences.

It should not require private reasoning traces.

Store:

```yaml
decision:
  selected: preserve existing API
  consequence: added repository operation without changing endpoint contract
```

Do not require:

```yaml
chain_of_thought:
  ...
```

### 6.3 State over activity

Prefer:

```yaml
change:
  authentication now rejects expired tokens before user lookup
```

over:

```yaml
activity:
  edited AuthService.cs
```

The file may be useful evidence, but the **state change is the semantic fact**.

---

# 7. Reporting Boundary

The Reporter runs at a **user-meaningful task boundary**, not after every low-level action.

Examples of low-level actions:

```text
read file
click element
run grep
call API
edit method
run compiler
scroll page
retry command
```

These are Executor concerns.

Examples of reporting boundaries:

```text
requested implementation completed
planned task completed
task partially completed
task blocked
task failed
user decision required
verification materially changes confidence
```

This prevents the workflow from turning every execution cycle into user-visible noise.

---

# 8. Reporter Input Contract

The Reporter receives a structured handoff.

Minimum conceptual schema:

```yaml
report_context:

  task:
    original_request:
    specification:
    acceptance_criteria:
    constraints:

  execution:
    status:
    current_state:
    state_delta:
    artifacts:

  verification:
    checks:
    evidence:

  exceptions:
    unresolved:
    failed_checks:
    blockers:
    assumptions_changed:

  communication:
    user_action_required:
    requested_output_style:
```

The Reporter should not need to infer the original goal from an execution transcript.

---

# 9. Completion Gate

Before writing anything, classify the task.

```text
COMPLETED
PARTIAL
BLOCKED
FAILED
NO_OP
```

## COMPLETED

All required acceptance criteria are satisfied, or the available evidence supports that conclusion.

## PARTIAL

A meaningful subset is complete, but one or more requirements remain unsatisfied.

## BLOCKED

Execution cannot continue without external information, permission, access, dependency, or user choice.

## FAILED

The attempted task did not achieve the required state and no valid continuation is currently available.

## NO_OP

The requested state was already true or no change was necessary.

The Reporter must never turn `PARTIAL`, `BLOCKED`, `FAILED`, or `UNVERIFIED` into `COMPLETED` merely to produce a cleaner response.

---

# 10. Materiality Filter

The central reporting rule is:

> Report consequences, not activity.

A fact is **material** when omitting it could change the user's understanding of:

- what outcome was achieved;
- how system behavior changed;
- whether the specification was satisfied;
- whether the result was verified;
- what risk remains;
- what remains incomplete;
- what the user must do next;
- whether compatibility or external behavior changed.

A fact is normally **non-material** when it only describes internal work with no consequence.

Examples:

```text
NON-MATERIAL
- opened five files
- searched the repository
- inspected an interface
- ran grep
- retried a command
- fixed formatting
- navigated through three pages
```

Unless one of those actions produced a consequence the user needs to know.

Material version:

```text
The public interface changed, so other implementations must add the new method.
```

Not:

```text
I modified IUserRepository.cs.
```

The filename can be included when it improves traceability, but the consequence comes first.

---

# 11. Semantic Coverage

Conciseness is subordinate to semantic completeness.

Before rendering the final answer, every material ledger fact must receive exactly one disposition:

```text
REPORT
MERGE
OMIT_WITH_JUSTIFICATION
```

### REPORT

The fact appears explicitly.

### MERGE

The fact is represented by another sentence without losing its consequence.

### OMIT_WITH_JUSTIFICATION

Allowed only when:

1. another reported fact fully implies it; or
2. it has no user-visible consequence.

The following facts may not be silently omitted:

```text
MUST_REPORT_IF_PRESENT

- unsatisfied acceptance criterion
- failed verification
- unverified completion claim
- changed external behavior
- changed public contract
- changed user data
- destructive or irreversible action
- unresolved blocker
- remaining risk
- required user action
- material deviation from specification
- assumption that materially affected the result
```

This is the **semantic checksum** of the report.

The question is not:

```text
Is the report concise?
```

The question is:

```text
Does the concise report preserve all material task state?
```

---

# 12. Completion Contract

The Reporter itself follows a contract.

```yaml
completion_report:

  required:
    - outcome

  conditional:
    material_changes: when_nonempty
    verification: when_available_or_materially_missing
    unresolved: when_nonempty
    user_action: when_required
    compatibility_effect: when_changed
    risk: when_material

  forbidden:
    - execution_narration_without_consequence
    - unsupported_success_claims
    - invented_next_steps
    - repeated_task_description
    - duplicate_semantic_information
    - hidden_failure
    - hidden_unverified_state
```

This contract controls **what must be communicated**.

Language rules control **how it is communicated**.

These are separate concerns.

---

# 13. Evidence Before Confidence

The Reporter must distinguish:

```text
implemented
verified
inferred
not verified
```

These are not synonyms.

Examples:

```text
Implemented the parser change. The relevant tests pass.
```

is stronger than:

```text
Implemented the parser change. I did not run the integration tests.
```

Both can represent useful work.

The Reporter must not convert the second state into:

```text
Everything is working correctly.
```

### Success claims

A success claim should be traceable to one or more of:

- acceptance criterion satisfied;
- test passed;
- build passed;
- observable environment state;
- tool confirmation;
- explicit artifact inspection;
- externally returned success state.

If verification is unavailable, say so briefly.

---

# 14. Information Architecture

The default report order is:

```text
1. OUTCOME
2. MATERIAL CHANGE
3. VERIFICATION
4. RESIDUAL STATE
5. USER ACTION
```

Only sections containing material information are rendered.

The Reporter should not generate empty headings.

For a simple successful task, the result may therefore be only two sentences.

Example:

```text
Implemented authenticated user lookup by email without changing the endpoint contract.
Build and 38 relevant tests pass.
```

For a task with a compatibility consequence:

```text
Implemented authenticated user lookup by email. The repository interface now requires
an email lookup operation, so other implementations of that interface must add it.
Build and 38 relevant tests pass.
```

For a partial task:

```text
The service implementation is complete and builds successfully, but the integration
test could not run because the test database is unavailable. The database-dependent
behavior remains unverified.
```

---

# 15. Language Renderer

Only after material facts have been selected should the Reporter render prose.

The language layer should use controlled technical writing principles inspired by systems such as Simplified Technical English, without claiming formal STE compliance unless the implementation actually validates the complete standard.

Recommended rules:

```text
- Use one stable term for one concept.
- Prefer concrete nouns and verbs.
- Prefer active voice when the actor matters.
- Avoid vague pronouns when the referent can be ambiguous.
- Avoid decorative transitions.
- Avoid synonyms merely for stylistic variation.
- Keep cause and consequence close together.
- Keep verification claims explicit.
- Use one proposition per sentence when combining propositions creates ambiguity.
- Prefer observable state over subjective confidence.
- Do not use generic success language.
```

Avoid:

```text
Everything looks good.
The task was successfully completed.
Several improvements were made.
The code was updated accordingly.
```

Prefer:

```text
The requested endpoint is implemented.
The build passes.
The integration test remains blocked by the unavailable database.
```

---

# 16. Compression Policy

The Reporter does not use a hard word limit by default.

Hard limits encourage omission.

Instead:

```text
COVERAGE FIRST
COMPRESSION SECOND
STYLE THIRD
```

Compression algorithm:

```text
1. Collect material facts.
2. Remove semantic duplicates.
3. Merge facts that share the same consequence.
4. Remove implementation details with no consequence.
5. Order remaining facts by user relevance.
6. Render the shortest wording that preserves all remaining facts.
```

Adaptive length follows the amount of material information.

A trivial task should produce a trivial report.

A complex task may require a longer report.

The Reporter must never add prose merely to make the response appear substantial.

---

# 17. Reporter Invariants

The skill should enforce the following invariants.

## R1 — Outcome first

The first statement communicates the result or current terminal state.

## R2 — No activity dump

Do not narrate actions merely because they occurred.

## R3 — Material consequence preservation

Every material consequence must survive compression.

## R4 — No unsupported completion

Do not claim completion when required verification or acceptance criteria contradict it.

## R5 — No silent residual work

Known incomplete work must be visible.

## R6 — Stable terminology

One concept uses one canonical term throughout the report.

## R7 — No redundant context

Do not restate information the user already supplied unless required to disambiguate the result.

## R8 — No invented next steps

Only report a next action when one is actually required or materially useful.

## R9 — Plan changes do not rewrite the contract

Replanning may alter strategy, not silently redefine success.

## R10 — Evidence survives compression

Verification results that materially affect confidence cannot disappear from the final message.

## R11 — Failures that changed the final state survive compression

A recovered transient error may be omitted when it has no residual consequence.

An error with residual consequence must be reported.

## R12 — Report state, not self

Prefer:

```text
The build passes.
```

over:

```text
I successfully ran the build.
```

---

# 18. Relationship With Replanning

In this workflow, the Executor decides when the current plan is no longer adequate and invokes replanning.

Conceptually:

```text
Executor observation
       │
       ├── plan still valid ─────► continue execution
       │
       └── plan invalid/incomplete
                    │
                    ▼
               REPLAN REQUEST
                    │
                    ▼
                 PLANNER
                    │
                    ▼
              revised plan
                    │
                    ▼
                 EXECUTOR
```

Examples of replanning triggers:

```text
- expected state does not exist;
- tool or environment returns an unexpected result;
- newly discovered information invalidates a future step;
- acceptance criterion requires work not represented in the plan;
- a planned action is impossible;
- a more direct valid path becomes available;
- execution discovers a material constraint.
```

The Reporter is **not** the replanner.

If a task is still actionable internally, control should return to Planner/Executor.

The Reporter becomes active when execution reaches a reporting boundary.

---

# 19. Reporter Must Compare Result Against Specification

The Reporter should not ask:

```text
Did the Executor say it finished?
```

It should ask:

```text
What does the specification require?

What does the final observable state show?

Which acceptance criteria have evidence?

What remains unresolved?
```

The final report is the projection of:

```text
SPECIFICATION
      ×
FINAL STATE
      ×
EVIDENCE
```

not merely:

```text
EXECUTOR SUMMARY
```

This prevents the Executor from becoming both the implementer and the sole judge of its own success.

---

# 20. Example: Coding Task

## User request

```text
Add caching to the product endpoint without changing its public API.
```

## Bad agent report

```text
Done! I inspected the product controller and service, found the relevant code,
created a cache abstraction, updated ProductService.cs, configured dependency
injection, added some tests, fixed a few issues, and ran the project. Everything
is now working correctly.
```

Problems:

- narrates execution;
- does not say what caching behavior changed;
- "everything is working" is unsupported;
- important compatibility information is hidden inside generic prose;
- verification is vague.

## Completion Reporter output

```text
Added caching to the product endpoint without changing its public API.
Cache behavior is covered by the relevant tests, and the project builds successfully.
```

If an important residual state exists:

```text
Added caching to the product endpoint without changing its public API.
Unit tests and the build pass, but cache behavior under the production Redis
configuration was not verified.
```

---

# 21. Example: Replanning

Initial specification:

```text
Find the repository's top contributor and follow that account.
```

Initial plan:

```text
1. Open contributors.
2. Identify the top contributor.
3. Follow the account.
```

During execution:

```text
top_contributor = alice
```

The Executor requests replanning because the abstract identity is now concrete.

Revised plan:

```text
1. Open Alice's profile.
2. Follow Alice.
3. Verify the follow state.
```

Ledger:

```yaml
objective:
  follow_top_contributor: true

discovered:
  top_contributor: alice

state_delta:
  - alice is now followed

verification:
  - profile shows following state

unresolved: []
```

Report:

```text
Followed the repository's top contributor, Alice. The profile now shows the account as followed.
```

The user does not need the click trajectory.

The newly discovered identity is reported because it is material to understanding the result.

---

# 22. Example: NO_OP

User:

```text
Enable nullable reference types in this project.
```

Final state:

```text
Nullable is already enabled globally.
```

Report:

```text
No change was needed. Nullable reference types are already enabled globally for the project.
```

Do not create fake work to satisfy the appearance of execution.

---

# 23. Example: BLOCKED

Specification:

```text
Deploy version 2.4.0 to production.
```

Execution state:

```yaml
deployment:
  status: blocked

blocker:
  production credentials unavailable

changes:
  production: none

verification:
  staging: passed
```

Report:

```text
Production deployment is blocked because production credentials are unavailable.
Version 2.4.0 passed the staging checks; production was not changed.
```

If the user must act:

```text
Production deployment is blocked because production credentials are unavailable.
Version 2.4.0 passed staging checks. Production was not changed.
Provide production deployment access to continue.
```

---

# 24. Failure Modes the Skill Is Designed to Prevent

## Chronicle reporting

The agent reports its journey instead of the result.

```text
First I inspected...
Then I changed...
After that I ran...
```

### Correction

Report final state and consequences.

---

## Semantic amputation

The summary becomes shorter by deleting an important condition.

Example:

```text
Tests pass.
```

when the real state is:

```text
Unit tests pass. Integration tests were not run.
```

### Correction

Coverage validation before compression.

---

## Success inflation

The agent treats implementation as proof of correctness.

### Correction

Separate implementation state from verification state.

---

## File-list reporting

```text
Changed:
- A.cs
- B.cs
- C.cs
```

without explaining why these changes matter.

### Correction

Describe behavioral or contract changes first.

Files are optional traceability metadata.

---

## Context echo

The agent repeats the user's request before reporting the outcome.

### Correction

Outcome first.

---

## Ritual next steps

The agent invents a "Next steps" section although no next action exists.

### Correction

Render user action only when required or materially useful.

---

## Hidden deviation

The Executor solves a different problem than the specification and reports success.

### Correction

Reporter compares final state to the specification contract.

---

# 25. Suggested Internal Algorithm

Pseudo-code:

```text
function report(specification, ledger, preferences):

    status = classify_completion(
        specification.acceptance_criteria,
        ledger.final_state,
        ledger.verification,
        ledger.unresolved
    )

    candidates = collect_reportable_facts(
        status,
        ledger.state_delta,
        ledger.verification,
        ledger.unresolved,
        ledger.blockers,
        ledger.user_action_required
    )

    material = []

    for fact in candidates:
        if changes_user_understanding(fact):
            material.append(fact)

    assert_required_facts_present(material, status, ledger)

    material = semantic_deduplicate(material)
    material = merge_by_consequence(material)

    ordered = order(
        material,
        [
            OUTCOME,
            BEHAVIOR,
            CONTRACT_CHANGE,
            VERIFICATION,
            RISK,
            UNRESOLVED,
            USER_ACTION
        ]
    )

    draft = controlled_render(ordered, preferences)

    validate_no_unsupported_claims(draft, ledger.evidence)
    validate_semantic_coverage(draft, material)
    validate_no_activity_dump(draft)

    return draft
```

The important part is that **rendering happens after semantic selection and validation**.

---

# 26. Machine-Checkable Report State

For stronger deterministic workflows, the Reporter can internally produce an intermediate representation before prose.

Example:

```yaml
report_ir:
  status: COMPLETED

  outcome:
    - authenticated email lookup implemented

  consequences:
    - public endpoint contract unchanged
    - repository implementations must support email lookup

  verification:
    - build: passed
    - tests:
        status: passed
        count: 38

  residual: []

  required_user_action: []

  prohibited_claims:
    - production behavior verified
```

Then the natural-language renderer may only verbalize facts in `report_ir`.

This creates a useful boundary:

```text
SEMANTIC LEDGER
      ↓
REPORT IR
      ↓
PROSE
```

The intermediate representation makes the system easier to test.

---

# 27. Deterministic Validation

The skill becomes much more reliable if important rules are testable.

Possible validators:

```text
MUST_REPORT(condition)
MUST_NOT_CLAIM(condition)
OMIT_IF_NO_CONSEQUENCE(condition)
REQUIRE_EVIDENCE(claim)
REQUIRE_USER_ACTION(blocked_by_user)
FORBID_DUPLICATE_SEMANTICS()
FORBID_EMPTY_SECTION()
FORBID_ACTIVITY_ONLY_SENTENCE()
```

Examples:

```text
IF failed_verification exists
THEN report MUST NOT have status COMPLETED
UNLESS specification explicitly permits that verification to be absent.
```

```text
IF public_contract_changed == true
THEN compatibility consequence MUST be represented.
```

```text
IF user_action_required is empty
THEN do not generate a "Next steps" section.
```

```text
IF verification.status == not_run
THEN do not use language equivalent to "verified", "confirmed", or
"everything works".
```

---

# 28. Testing the Skill

Do not evaluate the Reporter primarily by style preference.

Evaluate it by **semantic preservation**.

Recommended dimensions:

## Coverage

Did all material facts survive?

```text
material facts represented / total material facts
```

## Unsupported claim rate

Did the report claim anything not supported by the ledger?

Target:

```text
0
```

## Activity leakage

How many reported statements describe internal activity without consequence?

Target:

```text
0
```

## Redundancy

How many semantic facts are stated more than once?

Target:

```text
0
```

## Residual-state recall

Were all known blockers, incomplete requirements, and material risks represented?

Target:

```text
100%
```

## Compression

Among reports with full semantic coverage, prefer the shorter one.

This establishes the correct optimization order:

```text
1. correctness
2. coverage
3. relevance
4. compression
5. style
```

not:

```text
1. shortness
```

---

# 29. Relationship to Controlled Technical English

Controlled Technical English is useful at the rendering layer.

It helps with:

- lexical consistency;
- sentence clarity;
- predictable syntax;
- reduced ambiguity;
- technical terminology.

But it does not by itself solve the complete reporting problem.

The architecture therefore separates:

```text
INFORMATION MODEL
        ↓
INFORMATION SELECTION
        ↓
SEMANTIC COVERAGE
        ↓
INFORMATION ORDER
        ↓
CONTROLLED LANGUAGE
```

This distinction is fundamental.

A perfectly written sentence can still omit the most important fact.

---

# 30. Proposed Skill Identity

Suggested name:

```text
completion-reporter
```

Alternative names:

```text
semantic-reporter
task-completion-report
agent-completion-protocol
outcome-reporter
```

Recommended conceptual description:

> Produce a minimal, evidence-grounded, semantically complete user report at a task boundary. Report outcomes and consequences rather than execution activity. Preserve all material state, verification failures, residual work, risks, and required user actions during compression.

---

# 31. Proposed Skill Responsibility

The skill SHOULD:

```text
- classify terminal task state;
- compare final state against the specification;
- select material facts;
- preserve semantic coverage;
- distinguish implementation from verification;
- compress redundant information;
- render concise controlled technical prose;
- expose unresolved state;
- identify required user action.
```

The skill SHOULD NOT:

```text
- create the original specification;
- generate the execution plan;
- execute tools;
- decide low-level actions;
- perform replanning;
- repair implementation;
- invent missing evidence;
- reinterpret failed requirements as success;
- summarize private reasoning;
- narrate the action trajectory.
```

If execution must continue, control returns to the Executor or Planner instead of allowing the Reporter to solve the task.

---

# 32. Workflow Contract

A clean integration contract is:

```text
SPECIFICATION
    outputs:
      task_contract

PLANNER
    inputs:
      task_contract
    outputs:
      executable_plan

EXECUTOR
    inputs:
      task_contract
      executable_plan
      environment_state

    outputs:
      environment_actions
      semantic_ledger_updates

    may invoke:
      replanning

REPLANNING
    inputs:
      task_contract
      current_plan
      semantic_ledger
      current_environment_state

    outputs:
      revised_plan

COMPLETION REPORTER
    inputs:
      task_contract
      semantic_ledger
      final_environment_state
      verification_evidence

    outputs:
      user_report
```

This gives each stage a strict semantic boundary.

---

# 33. Central Design Rule

The entire skill can be reduced to one invariant:

> **No material fact may be lost merely because the final answer must be concise.**

And one selection rule:

> **No execution detail should survive merely because it happened.**

Together:

```text
REPORT = MINIMUM TEXT
         THAT PRESERVES
         MAXIMUM MATERIAL STATE
```

That is the goal of the Completion Reporter.

---

# 34. Reference Architecture

This workflow is inspired by the separation of planning and execution in:

Erdogan et al., **PLAN-AND-ACT: Improving Planning of Agents for Long-Horizon Tasks**, arXiv:2503.09572.

The proposed workflow extends that architecture with:

1. an explicit Specification contract before planning;
2. Executor-triggered replanning;
3. an incremental Semantic Ledger;
4. an explicit post-task Completion Reporter;
5. semantic coverage validation before natural-language compression.

The Reporter is therefore not a replacement for Plan-and-Act.

It is the **communication boundary that Plan-and-Act style execution leaves implicit**.

