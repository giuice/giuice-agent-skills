---
name: spec-from-scratch
description: Create a complete SPEC from scratch through an exhaustive requirements interview before any planning or implementation. Use this skill whenever the user asks to create, define, clarify, scope, or write a spec/SPEC/PRD/requirements document from an idea, especially when they want to avoid assumptions, start at "step zero," or prepare input for later planning workflows. This skill must question goals, requirements, constraints, edge cases, business rules, and acceptance criteria before drafting the final spec.
---

# Spec From Scratch

Use this skill to turn an unclear idea into a complete, assumption-light Product SPEC. The core job is not writing first; it is discovering what must be true before a useful spec can exist.

This workflow is portable across coding agents and planning systems. Treat it as a pre-planning gate: after the SPEC is complete, the user may hand it to any planning method, implementation workflow, or project-management process.

## Operating principle

Do not draft the final SPEC until the user's goals, requirements, constraints, edge cases, business rules, and acceptance criteria are fully understood.

If critical information is missing, ask more questions. A polished spec built on hidden assumptions is worse than no spec because it makes uncertainty look settled.

## Workflow

### 1. Start with scope capture

Begin by restating the raw idea in one or two sentences, then ask the first interview batch.

Use questions with answer options. Mark exactly one option as `(Recommended)` unless the question truly has no sensible default.

Default question format:

```markdown
## Question batch N — [theme]

1. **[Question]?**
   - A. [Option] `(Recommended)` — [short reason]
   - B. [Option] — [short reason]
   - C. [Option] — [short reason]
   - D. Other — [ask user to specify]

2. **[Question]?**
   - A. ...
```

Ask 7-10 questions in the first batch unless the idea is extremely narrow. The first batch should quickly expose the shape of the problem, so include at least one question about each of these areas: goals, primary users, scope boundaries, business rules or constraints, edge cases/failure modes, and acceptance criteria or success signals. Later batches may be smaller and focused only on remaining gaps. Keep each batch answerable in one pass.

### 2. Interview exhaustively by domain

Cover these domains before drafting:

1. **Problem and goals**
   - Problem being solved
   - Desired outcome
   - Primary and secondary users
   - Jobs to be done
   - Success metrics
   - Non-goals

2. **Scope and deliverables**
   - Product surface or system boundary
   - Must-have vs should-have vs later
   - User journeys
   - Inputs and outputs
   - Environments, platforms, or channels
   - Dependencies

3. **Requirements**
   - Functional requirements
   - Data requirements
   - Permission and access rules
   - Integrations
   - Reporting, notifications, or automation
   - Admin or support needs

4. **Business rules**
   - Eligibility rules
   - Pricing, billing, quotas, or limits
   - Workflow states and transitions
   - Approval rules
   - Compliance or policy constraints
   - Exceptions and overrides

5. **Constraints**
   - Technical constraints
   - Time, budget, staffing, and operational constraints
   - Existing systems that must be preserved
   - Security, privacy, legal, and regulatory constraints
   - Performance and reliability expectations
   - Localization, accessibility, and device constraints

6. **Edge cases and failure modes**
   - Empty, invalid, duplicate, stale, missing, or conflicting data
   - Partial completion and retry behavior
   - Offline or degraded dependencies
   - Abuse cases
   - Race conditions or concurrent edits
   - User mistakes and recovery paths

7. **Acceptance criteria and validation**
   - Observable behavior for each major flow
   - Given/When/Then scenarios
   - Definition of done
   - Test data or examples
   - UAT checklist
   - Launch or rollout criteria

### 3. Track knowns and unknowns

After each answer batch, summarize progress in a compact state table:

```markdown
## Current understanding

| Area | Status | Notes |
|---|---|---|
| Goals | Clear / Partial / Missing | ... |
| Users | Clear / Partial / Missing | ... |
| Requirements | Clear / Partial / Missing | ... |
| Constraints | Clear / Partial / Missing | ... |
| Edge cases | Clear / Partial / Missing | ... |
| Business rules | Clear / Partial / Missing | ... |
| Acceptance criteria | Clear / Partial / Missing | ... |

## Remaining unknowns

- [Unknown]
- [Unknown]
```

Then ask the next batch focused only on the weakest areas.

### 4. Apply the hard readiness gate

Before writing the final SPEC, explicitly run this gate:

```markdown
## SPEC readiness check

- Goals are specific and measurable: Pass / Missing
- Users and stakeholders are identified: Pass / Missing
- In-scope and out-of-scope boundaries are explicit: Pass / Missing
- Functional requirements are testable: Pass / Missing
- Business rules are explicit: Pass / Missing
- Constraints are explicit: Pass / Missing
- Edge cases and failure modes are covered: Pass / Missing
- Acceptance criteria are observable: Pass / Missing
- Open questions are non-blocking or intentionally deferred: Pass / Missing

Verdict: Ready / Not ready
```

If any item is `Missing`, the verdict is `Not ready` and you must ask more questions instead of drafting the final SPEC.

If the user asks to draft anyway, refuse the final SPEC politely and offer one of these alternatives:

- Continue the interview
- Produce a clearly labeled `Discovery Notes` summary, not a SPEC
- Narrow the scope so the readiness gate can pass

### 5. Write the final SPEC only when ready

When the readiness gate passes, produce this default Product SPEC format:

```markdown
# SPEC: [Name]

## 1. Summary
[Concise description of what is being built and why.]

## 2. Problem statement
[The problem, who has it, and why it matters.]

## 3. Goals and success metrics
### Goals
- ...

### Success metrics
- ...

## 4. Users and stakeholders
### Primary users
- ...

### Secondary users
- ...

### Stakeholders
- ...

## 5. Scope
### In scope
- ...

### Out of scope
- ...

## 6. User journeys
- ...

## 7. Functional requirements
Use stable requirement IDs:

| ID | Requirement | Priority | Acceptance signal |
|---|---|---|---|
| FR-001 | ... | Must | ... |

## 8. Business rules
Use stable rule IDs:

| ID | Rule | Rationale |
|---|---|---|
| BR-001 | ... | ... |

## 9. Data and integrations
### Data inputs
- ...

### Data outputs
- ...

### Integrations
- ...

## 10. Constraints and assumptions
### Constraints
- ...

### Assumptions
Only include assumptions explicitly confirmed by the user.
- ...

## 11. Edge cases and failure modes
| Case | Expected behavior |
|---|---|
| ... | ... |

## 12. Security, privacy, compliance, and abuse considerations
- ...

## 13. Accessibility, localization, and usability considerations
- ...

## 14. Acceptance criteria
Use Given/When/Then where helpful:

- Given ..., when ..., then ...

## 15. Validation and launch checklist
- ...

## 16. Open questions
Only include non-blocking questions. If any question blocks requirements, return to the interview instead.
- ...
```

## Question design rules

- Provide options that teach tradeoffs, not generic choices.
- Include `Other — specify` for questions where the user may have domain-specific answers.
- Recommend the safest or most scope-preserving option when uncertain.
- Prefer concrete language over abstract product jargon.
- Ask about negative space: what the product must not do.
- Ask about bad data, bad actors, failed dependencies, and user mistakes.
- Ask for examples whenever a rule or workflow could be interpreted multiple ways.
- Do not invent domain facts. If a detail matters and is unknown, ask.

## Handling different user styles

If the user is brief, offer defaults but keep the hard readiness gate.

If the user is domain-expert, ask sharper business-rule and edge-case questions instead of explaining basics.

If the user is overwhelmed, reduce batch size to 3-5 questions and tell them they can answer with option letters.

If the user gives a large existing brief, extract what is already answered first, then ask only for gaps.

## Portable output note

When the user plans to use the result in another tool or agent, keep the final SPEC self-contained. Avoid references like "as discussed above" unless the referenced content is included in the SPEC itself.
