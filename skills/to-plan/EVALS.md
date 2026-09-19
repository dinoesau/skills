# to-plan fault localization evals

Three scenarios that test the Fault Localization Report.
Run them against a sample failed wave before changing the report template.
Each eval states inputs, expected layer, and expected action.

## E1 - Ambiguous spec forces failure

Inputs: lane retry log shows two evidence-backed attempts, Tier 1 fails with `wave diff contradicts wave guardrail`, wave guardrail text is vague about file ownership.
Expected layer: P3 Plan/Spec with high confidence.
Expected action: suggest plan-critique-loop on the plan path, wait for approval or apply the minimal unblocking mutation with the pending critique noted.
Fail if: the coordinator mutates the DAG without a report, or asks a bare proceed-or-fix question.

## E2 - Lane typo ignores clear spec

Inputs: spec states the correct outcome in one sentence, lane diff mistypes a symbol on one touched line, validator output cites that line, retry evidence exists.
Expected layer: P4 Agent execution with high confidence.
Expected action: respawn the lane with the minimal DAG mutation, state `no critique needed`, continue without suggesting plan-critique-loop.
Fail if: the coordinator suggests plan-critique-loop for this failure.

## E3 - Superficial fix misses root cause

Inputs: diff passes lane-local typecheck, Tier 1 fails with `symptom fixed, root cause untouched`, Problem Statement names the root cause explicitly.
Expected layer: P2 Solution with medium or high confidence.
Expected action: suggest plan-critique-loop on the plan path, reclassify one layer up if the same wave fails three times.
Fail if: the coordinator replans the same P4 layer a third time without reclassifying.
