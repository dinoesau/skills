---
name: to-plan
description: Generate an agent-ready implementation plan as docs/plan-<slug>.md - a human-facing PRD plus executable Agent Instructions (steps, evals, guardrails, error budget).
disable-model-invocation: true
argument-hint: "[feature-description]"
---

# to-plan

Feature to plan: $ARGUMENTS

If no arguments were given, derive the feature from the current conversation.

Produces a `docs/plan-<slug>.md` with two audiences:

- **Docs for Humans** - a PRD with problem, solution, user stories, implementation decisions, testing decisions
- **Agent Instructions** - an executable spec another agent can follow without access to this conversation

The slug is the feature name in kebab-case, 2-5 words.
If `docs/plan-<slug>.md` already exists, this run replaces it - say so when reporting.

The plan file is the only output: never publish anything to the issue tracker.

## Seams

Sketch the seams at which the feature will be tested before writing the plan.
Prefer existing seams over new ones; propose new ones at the highest point possible.
The fewer seams across the codebase, the better - the ideal number is one.

## Process

Decide first: if you could not fill the Implementation Decisions section without guessing (scope, test seams, or constraints are ambiguous), run **Clarify** first.
Otherwise go straight to **Synthesize**.

### Clarify (feature is fuzzy)

1. Ask the user 2-4 focused questions covering scope, test seams, and constraints.
2. Confirm the answers back and wait for the user's approval before synthesizing.

### Synthesize (feature is clear)

1. Explore the codebase until you can name the files and seams the feature touches.
   Respect any ADRs in the touched area.
2. Write the **Docs for Humans** section using the template in [PRD-TEMPLATE.md](PRD-TEMPLATE.md), in the project's domain glossary vocabulary, with Mermaid diagrams for structural or sequential concepts.
3. Read [REFERENCE.md](REFERENCE.md) to calibrate evals, guardrails, and checkpoints, then write the **Agent Instructions** section following the skeleton in [TEMPLATE.md](TEMPLATE.md).
4. Fill the Required skills table by scanning `.agents/skills/` for skills the executing agent needs in the touched area.
5. Write the result to `docs/plan-<slug>.md`.
6. Verify the plan against the checklist below.

## Verification checklist

Before declaring the plan done, confirm:

- [ ] Every implementation step has a verifiable guardrail and at least one eval
- [ ] Every command in the plan was run once and works (a dev server only needs to start); pre-existing failures are noted in the plan, not fixed
- [ ] Every file path referenced in the plan exists, or is explicitly marked as new
- [ ] Required skills are listed as readable file paths, not just skill names
- [ ] The browser validation section is present only if the change affects UI
- [ ] Structural or sequential concepts in Docs for Humans are explained with Mermaid diagrams (or the plan has none because the feature is trivial)
- [ ] Every template placeholder is replaced and every HTML comment deleted
- [ ] The plan is self-contained: an agent with no access to this conversation can execute it
