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
- For parallel waves, apply chain order per wave (types -> impl -> tests per lane), then a global barrier does integration and visual verification.

## Concurrency

- Waves are the unit of delegation. A wave is a set of steps with no inter-dependency and no file conflict; sequential waves run one lane at a time with Max parallelism 1. Every step runs inside exactly one lane, never directly by the coordinator.
- Leaf-first: start from tasks and files with zero `depends_on`, build the tree upward. If a dependency is uncertain, mark it as sequential (pessimistic).
- File-conflict rule: if two steps in the same wave touch the same file, they must be sequentialized into sub-waves or the file split. Detect with `grep` on the Files column.
- Degrees of freedom: narrow bridge (DB migration, schema, `graph`) = low freedom, exact script per worker; open field (UI copy, tests, `sequenceDiagram`) = high freedom, heuristic prompt. Match specificity to fragility.
- Chain order per wave: types -> implementation -> tests. Global barrier after the last wave does integration and visual verification.
- Error budget per wave: a failure in lane A isolates; lane B continues until the barrier, then the coordinator decides. Document `isolation` vs `fail-fast` per plan.
- Context scoping: each worker gets minimal context - its wave's files, key types, and one example test - not the full repo. Shared context is only merged at barriers.
- State is king: edges of the graph transport state. Persist the state schema (files, types, env) in `docs/plan-<slug>-state.md` per wave to avoid polluting shared context. Lanes never write outside their own section; only the coordinator merges at barriers.

## Coordinator loop

- The coordinator NEVER implements steps itself and NEVER edits source files. It only reads, spawns lane and counter subagents, merges the state file, runs read-only barrier checks, and decides proceed / replan / stop.
- The coordinator is the only writer at barriers. Barrier order is fixed: collect lane outputs, merge state file, run barrier guardrails, spawn Tier 1 counter review, then decide proceed / replan / stop.
- Replan is autonomous and logged. The coordinator may add, remove, split, or move steps across waves without pausing, as long as leaf-first order and the file-conflict rule still hold.
- Every replan appends a DAG version entry (v1, v2, ...) to the state file with reason, DAG diff, and affected waves. The plan file itself stays frozen; the state file is the live record.
- If an instruction cannot be executed as written, the coordinator logs the discrepancy in the state file, applies the smallest DAG mutation that unblocks progress, and continues. It only stops and asks when guardrails, evals, and counter review all fail to produce a safe path.

## DAG mutation

- Allowed ops: add step, remove step, split wave into sub-waves, move step across waves, fan-out conditional lanes from one lane result.
- Forbidden ops: merging two lanes that touch the same file into one parallel wave, dropping a guardrail or eval to make a lane pass, skipping Tier 2 counter review.
- Conditional fan-out must declare its trigger in the state file (for example: `if lane A reports >3 call sites, fan-out to 2 lanes`). Untriggered branches stay dormant and cost no context.
- Version every mutation. A wave running on stale DAG v1 while the coordinator published v2 must stop and reload state before continuing.

## State store

- One Markdown file per plan: `docs/plan-<slug>-state.md`. Initialized at DAG v1 with one section per wave plus slots for counter verdicts and DAG log.
- Per-wave section records: inputs, files touched, key type surface, commands run with output summary, eval results, retry count, counter verdict.
- Markdown over JSON: diffable in `git diff`, readable by humans and agents, matches the existing `docs/plan-<slug>.md` pattern.
- Barrier summaries append to the state file. Never rewrite history; append v2, v3 entries so the full run stays auditable.

## Lane kickoff

- Require the readiness gate for every lane that edits code. Skip it for read-only `explore` lanes, `counter` reviews, and the trivial single-step exception.
- Judge readiness on four signals: correct problem summary, correct skill IDs loaded via the skill tool, supporting files read (REFERENCE.md, EVALS.md), and at least one concrete edge case. A verdict or readiness reply without loading evidence is invalid; re-spawn rather than coaching across rounds.
- The GO must be explicit. `Proceed`, `GO`, or a corrected re-spawn counts. Silence or an unrelated message does not count.
- Cost control: one extra roundtrip per editing lane is cheap compared to rework from a misunderstood scope. Do not add a second confirmation round inside the lane.

## Retry loop

- Each lane runs act -> eval -> reflect -> fix, max 2 fix attempts per the error budget. The third failure isolates the lane until the barrier.
- Each attempt must log validator output (typecheck, test, lint) into its state file section before retrying. Retries without evidence are not allowed.
- Lane-local evals (typecheck per lane) may run before the barrier. Wave-level evals run after the barrier merge.

## Adversarial review

- Two tiers, both blocking. Tier 1 runs after each wave barrier on the wave diff. Tier 2 is the final pre-merge gate with three parallel lanes: 2a correctness, 2b language standard, 2c 12-factor. Any `fail` blocks the next wave or the merge until the coordinator replans with fix waves.
- The coordinator spawns one `counter` subagent per review with the wave or branch `git diff` plus the Problem Statement and Solution copied from Docs for Humans.
- Tier 2b spawns one lane per touched language loading the skill via the skill tool: Python loads good-python, TypeScript loads good-typescript, Rust loads rusty, then reads REFERENCE.md and EVALS.md from the skill base directory. Findings on diff-touched lines block; pre-existing outside the diff is advisory.
- Tier 2c applies the 12-factor checklist in TEMPLATE.md. Required for deploy, runtime, config, or backing-service changes; skipped with a logged reason for pure library changes.
- Tier 2 fix loop is capped at 4 cycles. After the fourth failed re-review the coordinator stops and asks instead of spawning more waves.
- Verdict format is `pass / fail + findings`, stored in the state file. Findings must cite file paths and line numbers.
- Keep the review narrow: refute the diff against the stated problem and solution. Do not redesign scope; file scope questions as findings for the coordinator.

## Instructions vs context

- Instructions say WHAT; context (files to read, types, examples) says HOW.
- Compact instructions plus rich context beats long step-by-step prose, which is brittle.
- If a step needs more than two sentences to describe, split it.
- For parallel plans, scope context per wave and lane (its files + key types + one example test). Shared context merges only at barriers. Sequential waves follow the same rule with one lane.

## Human-in-the-Loop

- Place checkpoints before structural changes and before merge.
- Every checkpoint must state exactly what the user is confirming.

## Error budget

- The defaults live in TEMPLATE.md; override them only with a reason stated in the plan.
