# Authoring rules for agent plans

Calibration guide for writing the Agent Instructions section.
The mechanical rules (guardrail per step, eval before step, 1-3 checkpoints, error budget defaults) live in TEMPLATE.md; this file only calibrates judgment.
Grounded in "AI Engineering" (O'Reilly); the concepts themselves are assumed known.

## Evals

- Prefer the highest seam available: unit < integration < e2e < browser.
- Existing tests are the regression eval; every plan must run them.

## Guardrails

- Prefer commands the repo already has (typecheck, tests, lint) over new tooling.
- Chain order: types first, then implementation, then tests, then visual verification.

## Instructions vs context

- Instructions say WHAT; context (files to read, types, examples) says HOW.
- Compact instructions plus rich context beats long step-by-step prose, which is brittle.
- If a step needs more than two sentences to describe, split it.

## Human-in-the-Loop

- Place checkpoints before structural changes and before merge.
- Every checkpoint must state exactly what the user is confirming.

## Error budget

- The defaults live in TEMPLATE.md; override them only with a reason stated in the plan.
