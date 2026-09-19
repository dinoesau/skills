# Plan-Critique-Loop - Evaluations

Run each scenario in a fresh session with the skill installed. Compare against the baseline: same prompt without the skill. The skill wins only if it meets all expected behaviors. Re-run when the lane rubrics change, not only when `SKILL.md` changes.

## Scenario 1: hardcoded secrets plus missing tests

Input: a plan that adds `src/billing/invoice.py` with a hardcoded Stripe key, no env config, and no test seam.

Expected:

- [ ] Step 2 filters to `C1: src/billing/invoice.py` and skips PRD prose.
- [ ] Counter 1 returns BLOCKING on config factor with env fix.
- [ ] Counter 2 returns BLOCKING on missing TDD seam.
- [ ] Rewrite moves the key to env and adds a test step in place.
- [ ] Final report states cycles run and files rewritten.

## Scenario 2: clean plan approves fast

Input: a small plan that adds one pure function with existing tests and env-based config.

Expected:

- [ ] All 3 lanes return APPROVE on wave 1 or 2.
- [ ] No unrelated prose is rewritten.
- [ ] Loop exits early without burning remaining cycles.
- [ ] Final verdict is APPROVED with cycle count 1 or 2.

## Scenario 3: gh issue input with mixed prose

Input: gh issue `#99` with background story plus two code changes in `src/auth/login.py` and a new worker file.

Expected:

- [ ] Step 1 resolves the issue via `gh issue view` or pasted body.
- [ ] Step 2 lists exactly `C1` and `C2`, ignoring background story.
- [ ] Counter 3 names the test seam per `Cn` and flags unit-only coverage as BLOCKING where applicable.
- [ ] Rewrite touches only the two code sections in the source file.

## Trigger test

Must trigger on: "critique this plan", "critica este plan", "revisa este plan", "adversarial review of gh issue #12 until approve".
Must not trigger on: prose polishing requests, logic puzzles without a plan, general 12-factor questions with no plan attached.
If it overtriggers, narrow the description. If it undertriggers, add the user phrase.
