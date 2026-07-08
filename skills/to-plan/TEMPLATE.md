# Plan: <feature-slug>

<!-- Skeleton for docs/plan-<slug>.md.
Replace every <placeholder>.
Delete sections marked (conditional) when they do not apply.
Delete every HTML comment, including this one, from the final plan. -->

> Generated: <YYYY-MM-DD> | Base commit: <output of `git rev-parse --short HEAD`>

## Required skills

Files the executing agent must READ at session start.
List file paths, not skill names: some skills set `disable-model-invocation: true`, which blocks the Skill tool, so the agent must Read the files directly.

| Skill | Path | Why |
|-------|------|-----|
| <name> | .agents/skills/<name>/SKILL.md | <why the executor needs it> |

---

## Docs for Humans

### Problem Statement

### Solution

### User Stories

1. As an <actor>, I want a <feature>, so that <benefit>

### Implementation Decisions

### Testing Decisions

### Out of Scope

### Further Notes

---

## Agent Instructions

> This section is for the agent executing the implementation.
> Read the skill files listed above before starting.

### Context to load first

- Skill files listed above
- Files to be modified (read before editing)
- Key types and interfaces
- Existing tests as style and pattern reference

**Do NOT:**

- Modify files outside those listed
- Refactor unrelated code
- Assume conventions that are not present in the codebase

### If an instruction cannot be executed as written

Detailed instructions are brittle, and the codebase may have moved past the plan's base commit (see header).
If a step cannot be executed exactly as written (file renamed, API changed, missing dependency):

1. Do NOT improvise an alternative silently.
2. If the mismatch is trivial (e.g. a slightly different file path), adapt, continue, and note it.
3. Otherwise stop, report the discrepancy, and ask how to proceed.

### Project commands

<!-- Author: verify these by running them against package.json, pyproject.toml, or Makefile, and fill in the project's actual commands.
Do not assume a stack: this repo has both a TypeScript frontend and a Python backend. -->

| Purpose | Command |
|---------|---------|
| Typecheck | <e.g. `make -C source typecheck` or `npx tsc --noEmit`> |
| Tests | <e.g. `make -C source test` or `npx vitest run`> |
| Lint | <e.g. `make -C source lint` or `npm run lint`> |
| Dev server (only if browser validation applies) | <e.g. `npm run dev`> |

### Implementation steps

Each step has a **guardrail**: a verifiable condition that must hold before moving to the next step.
If a guardrail fails, stop and report.

<!-- Example row (delete):
| 1 | Add `carpeta_id` to the Ticket model | source/app/models.py | `make -C source typecheck` reports no new errors |
-->

| # | Step | Files | Guardrail |
|---|------|-------|-----------|
| 1 | <short description> | <files to touch> | <verifiable condition> |
| 2 | ... | ... | ... |

### Evals (Evaluation-Driven Development)

Define the eval before implementing the step.
Each step has one or more evals proving the change works.

<!-- Example row (delete):
| 1 | A ticket saved with a carpeta_id reads back with the same carpeta_id | integration | `make -C source test` |
-->

| Step | Eval | Type | Command |
|------|------|------|---------|
| 1 | <test description> | unit / integration / e2e | <command> |
| 2 | ... | ... | ... |

### Browser validation (conditional: only if the change affects UI)

Use the agent's Playwright MCP browser tools: `browser_navigate`, `browser_snapshot`, `browser_click`, `browser_take_screenshot`.
Do NOT use `@playwright/test`.
If the MCP browser tools are not available in the session, report it and skip this section - do not substitute another mechanism.

1. Start the dev server
2. Navigate to the relevant page with `browser_navigate`
3. Check that the expected elements exist with `browser_snapshot`
4. Interact with `browser_click` if applicable
5. Capture evidence with `browser_take_screenshot`

Success criterion:

```
<what the snapshot must show>
```

### Human-in-the-Loop checkpoints

At these points the agent must PAUSE and ask the user before continuing.
Keep it to 1-3 checkpoints: each one interrupts the executor's autonomous flow.

1. **After step <N>:** <what the user must confirm>
2. **Before merge:** <final validation>

### Error budget

Single source of truth for failure tolerance in this plan:

| Event | Limit | Action when exceeded |
|-------|-------|----------------------|
| New or failing test | 2 fix attempts | Stop and report; never continue or declare done with failing tests |
| Type errors in touched files | 0 | Stop and fix before continuing |
| Pre-existing type errors | Not counted | Ignore, they predate the change |
| New lint errors | 0 | Stop and fix |
| Browser validation failure | 1 retry | Stop, report, and ask |
| File not found | - | If a trivial rename, adapt and note it; otherwise stop and report |
| Ambiguous instruction | 0 | Stop and ask; never assume |

### Completion checklist

The agent must complete this before declaring the work done:

- [ ] All implementation steps done
- [ ] New tests pass
- [ ] Existing tests still pass
- [ ] Typecheck passes with no new errors
- [ ] Lint passes
- [ ] Browser validation passed (only if applicable)
- [ ] No out-of-scope files modified

---

## Risks

| Risk | Mitigation |
|------|-----------|
| <risk> | <mitigation> |
