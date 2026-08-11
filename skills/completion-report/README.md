# completion-report

Produce the smallest user-facing report that preserves every material fact about the outcome.

## When to use

- At the end of an execution run, whether it completed, stalled, or failed
- "What was the outcome?"
- "Summarize what was accomplished"

## What it does

It is not a summarizer of the execution trace. It is a projection of **specification × final state × evidence**.

1. Classifies the terminal state: `COMPLETED`, `PARTIAL`, `BLOCKED`, `FAILED`, `NO_OP`.
2. Checks each acceptance criterion against actual evidence — so the implementer is not the sole judge of its own success.
3. Filters activity out and keeps consequences.
4. Verifies that no fact on the must-report list was lost to compression.
5. Renders in controlled technical prose, only the sections that carry material information.

Two rules generate the rest:

> No material fact may be lost merely because the answer must be concise.
>
> No execution detail survives merely because it happened.

A simple successful task produces two sentences with no headings. A partial one says exactly what remains unverified, in the same breath as what works.

## Prevents

Chronicle reporting, semantic amputation ("tests pass" when integration tests never ran), success inflation, file-list reporting, context echo, ritual "Next steps", and hidden deviation from the spec.

## Artifacts

Reads `spec-interview/<slug>/SPEC.md` and `LEDGER.md`. Renders the report in the conversation and writes `REPORT.md`.

## Related

Part of `SPECIFY → PLAN → EXECUTE ↔ REPLAN → REPORT`, with [spec-from-scratch](../spec-from-scratch), [plan-from-spec](../plan-from-spec), and [execute-plan](../execute-plan). [spec-to-done](../spec-to-done) is the entry point if you would rather not name the stage yourself.

Full design rationale, including the semantic-ledger model and validation criteria: [docs/completion-reporter-design.md](../../docs/completion-reporter-design.md).

## Install

```bash
npx skills add giuice/giuice-agent-skills --skill completion-report
```
