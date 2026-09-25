---
name: to-plan
description: Generate an agent-ready implementation plan as docs/plan-<slug>.md - a human-facing PRD plus executable Agent Instructions (steps, evals, guardrails, error budget) with dependency graph and parallel waves for concurrent sub-agents when tasks are independent. Use when planning a feature with multiple files or independent tasks.
disable-model-invocation: true
argument-hint: "[feature-description]"
---

# to-plan

Feature to plan: $ARGUMENTS

If no arguments were given, derive the feature from the current conversation.

Produces a `docs/plan-<slug>.md` with two audiences plus a runtime state companion:

- **Docs for Humans** - a PRD with problem, solution, user stories, implementation decisions, testing decisions
- **Agent Instructions** - an executable spec another agent can follow without access to this conversation, including the coordinator loop
- **State file** - `docs/plan-<slug>-state.md` initialized at DAG v1, updated at each wave barrier

The slug is the feature name in kebab-case, 2-5 words.
If `docs/plan-<slug>.md` already exists, this run replaces it - say so when reporting.
If the state file already exists, reset it to DAG v1 for the new plan.

The plan file and state file are the only outputs: never publish anything to the issue tracker.

## Seams

Sketch the seams at which the feature will be tested before writing the plan.
Prefer existing seams over new ones; propose new ones at the highest point possible.
The fewer seams across the codebase, the better - the ideal number is one.
When decomposing for parallel execution, multiple seams may be justified - one per lane. Document the exception vs ideal=1.

## Process

Decide first: if you could not fill the Implementation Decisions section without guessing (scope, test seams, or constraints are ambiguous), run **Clarify** first.
Otherwise go straight to **Synthesize**.

### Clarify (feature is fuzzy)

1. Ask the user 2-4 focused questions covering scope, test seams, and constraints.
2. Confirm the answers back and wait for the user's approval before synthesizing.
3. Before each pause for human input, send a notification per the Notifications section below.

### Synthesize (feature is clear)

1. Explore the codebase until you can name the files and seams the feature touches.
   Respect any ADRs in the touched area.
2. Build the dependency graph: list all files and tasks, identify leaf nodes with zero dependencies, construct the full tree, and mark file-conflict edges (two tasks touching the same file must be sequentialized or the file split). Tag it DAG v1. See [REFERENCE.md](REFERENCE.md) Concurrency and DAG mutation.
3. Partition into waves: group leaf nodes into Wave 1, then successive waves by `depends_on`. Every wave gets a parallelizable flag and required sub-agent assignment, even sequential single-lane waves with Max parallelism 1. Render as a Mermaid DAG in Implementation Decisions. Declare the coordinator barrier procedure (merge state, run guardrails, spawn counter review, run fault localization on failure) per wave. The coordinator itself implements zero steps.
4. Write the **Docs for Humans** section using the template in [PRD-TEMPLATE.md](PRD-TEMPLATE.md), in the project's domain glossary vocabulary, with Mermaid diagrams for structural or sequential concepts.
5. Read [REFERENCE.md](REFERENCE.md) to calibrate evals, guardrails, checkpoints, retry loop, state store, adversarial review, and fault localization, then write the **Agent Instructions** section following the skeleton in [TEMPLATE.md](TEMPLATE.md).
6. Fill the Required skills table by scanning `.agents/skills/` for skills the executing agent needs in the touched area. Include good-python, good-typescript, or rusty when the diff touches that language, so Tier 2b lanes can load them by skill ID. The skill ID is the directory name; lanes invoke it via the skill tool, never by `Read` alone.
7. Estimate line-change scope from the Implementation steps Files column (new files count fully, edits count partially; when uncertain, state the assumption). Write the result to `docs/plan-<slug>.md` with the frozen `Est. scope: +<ins> -<del> in <N> files (~<churn> churn)` header, and initialize `docs/plan-<slug>-state.md` at DAG v1 with per-wave sections, empty results, the live `Actual vs base` header slot, and the counter review slots.
8. Verify the plan against the checklist below.

## Verification checklist

Before declaring the plan done, confirm:

- [ ] Every implementation step has a verifiable guardrail and at least one eval
- [ ] Every command in the plan was run once and works (a dev server only needs to start); pre-existing failures are noted in the plan, not fixed
- [ ] Every file path referenced in the plan exists, or is explicitly marked as new
- [ ] Required skills are listed as readable file paths, not just skill names
- [ ] The browser validation section is present only if the change affects UI
- [ ] Structural or sequential concepts in Docs for Humans are explained with Mermaid diagrams (or the plan has none because the feature is trivial)
- [ ] Every template placeholder is replaced and every HTML comment deleted
- [ ] Header carries the frozen `Est. scope` in `+<ins> -<del> in <N> files (~<churn> churn)` format (or `TBD` with reason)
- [ ] The plan is self-contained: an agent with no access to this conversation can execute it
- [ ] Dependency graph is present with leaf-first tree and file-conflict matrix
- [ ] Each wave declares sub-agent assignment and barrier guardrail, even sequential single-lane waves; no two parallel steps touch the same file; coordinator implements zero steps itself
- [ ] Coordinator loop is present with barrier merge, counter BEFORE/AFTER, resolution subagent decisions, DAG mutation log, and autonomous replan rules
- [ ] Fault Localization Report section is present with strict template, P1-P5 layers, and plan-critique-loop suggestion rule (P2/P3 only)
- [ ] State file is initialized at `docs/plan-<slug>-state.md` with per-wave sections, live `Actual vs base` header slot, counter verdict slots, resolution decision slots, and DAG v1
- [ ] Each lane declares its retry loop (act -> eval -> reflect -> fix, max 2 fix attempts) with validator output logged to the state file plus BEFORE/AFTER proposals for the resolver
- [ ] Each lane declares skill IDs to load via the skill tool with loading evidence (REFERENCE.md, EVALS.md); editing lanes add the kickoff gate (readiness reply + explicit GO before any edit)
- [ ] Final gate declares Tier 2a correctness, Tier 2b language standard per touched language, and Tier 2c 12-factor (or logged skip with reason); all blocking with counter BEFORE/AFTER, resolution decisions, and fix-wave iteration capped at 4 cycles

## Notifications

Send shell notifications via the `notify` service at lifecycle events.
Why: the user relies on the notification service to know when to return, not on polling chat.
Never let a `notify` failure block planning: always append `|| true` and continue, logging the failure.
Every message must be max 300 characters in total: always pipe through `cut -c1-300`.

- Before pausing for human input (Clarify questions, answer confirmation, any stop-and-ask):
  `notify "$(printf '%s' "Human input needed: <slug> - <reason>" | cut -c1-300)" || true`
- After verification passes and both files are written:
  `notify "$(printf '%s' "Plan creation completed: docs/plan-<slug>.md" | cut -c1-300)" || true`
- On any blocking failure (cannot synthesize, verification fails, unexpected error):
  `notify "$(printf '%s' "Something went wrong with <slug>: <1-line cause>" | cut -c1-300)" || true`
