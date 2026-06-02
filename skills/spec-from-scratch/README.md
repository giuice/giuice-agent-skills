# spec-from-scratch

Create a complete product SPEC from an unclear idea through an exhaustive requirements interview before planning or implementation.

## When to use

Use this skill when you want to create, define, clarify, scope, or write a SPEC/PRD/requirements document from scratch and want to avoid hidden assumptions.

Good triggers include:

- "Help me write a spec for this idea"
- "Let's define requirements before planning"
- "I want to start at step zero"
- "Create a PRD/SPEC, but ask me questions first"

The interview runs in exhaustive rounds: each round delivers a clickable batch of questions (native question UI when the agent supports it, otherwise a self-contained HTML form you fill, click **Copy answers**, and paste back). Answers are stored, a coverage gate forces follow-up rounds until every domain is clear, then the final SPEC is written.

## What it does

The skill interviews the user across:

- Goals and success metrics
- Users and stakeholders
- Scope and non-goals
- Functional requirements
- Business rules
- Constraints
- Edge cases and failure modes
- Acceptance criteria and validation
- Test strategy (only when the project produces code)

Every question lets you pick an option or write your own answer/note. The acceptance criteria are TDD-ready (mapped to functional requirement IDs), and code projects also get a Test Strategy section.

It does not draft the final SPEC until the readiness gate passes.

## Install

```bash
npx skills add giuice/giuice-agent-skills --skill spec-from-scratch
```
