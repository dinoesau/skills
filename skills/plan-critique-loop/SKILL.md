---
name: plan-critique-loop
description: Critiques implementation plans and revises code change requests through 3 parallel counter agents covering 12-factor, language quality, and architecture seams. Use when user says critique plan, critica plan, revisa plan, or asks for adversarial review of a plan or gh issue until approve.
license: MIT
allowed-tools: Bash
---

# Plan-Critique-Loop

Adversarial review loop for plans. Resolves a plan file or gh issue, filters to code change requests only, then spawns 3 counter agents in parallel, applies fixes in place, and repeats until all approve or the cycle cap hits.

Plan to review: $ARGUMENTS

If no arguments were given, derive the plan from the current conversation or ask for the plan path or gh issue number.

## Scope

Only code change requests are critiqued and rewritten. Ignore prose, PRD narrative, user stories, and timelines unless they change a code decision. If a section has no file change, no command, and no API contract change, skip it.

Full rubrics: [REFERENCE.md](./REFERENCE.md). Evaluation cases: [EVALS.md](./EVALS.md).

## Workflow

Copy this checklist into your response and mark progress.

```
Progress Plan-Critique-Loop:
- [ ] 1. Resolve input to reviewable text
- [ ] 2. Filter to code change requests
- [ ] 3. Spawn counter wave
- [ ] 4. Resolve via resolution subagent and rewrite plan in place
- [ ] 5. Re-loop until approve or cap
```

### 1. Resolve input to reviewable text

Accept one of:

- A plan path such as `docs/plan-<slug>.md`.
- A gh issue reference such as `#123` or a pasted issue body.
- Raw plan text in `$ARGUMENTS`.

If given a path, read that file. If given a gh issue number, run `gh issue view <number> --comments` to get body plus comments. If given raw text, treat it as cycle 1 input and note the source plan path for in-place rewrite. If the rewrite target is unclear, ask before spawning counters.

### 2. Filter to code change requests

List each proposed code change as `C1, C2, ...` with file path or new-file marker. Drop non-code sections. State the filtered list explicitly, for example `C1: src/auth/login.py, C2: new src/billing/invoice.py`. Counters review only this list. If the list is empty, stop and report no code changes to critique.

### 3. Spawn counter wave

Spawn all 3 counters in the same turn with the same filtered list and the same plan text. Each counter loads its skills via the `skill` tool, never by `Read` alone. Counters propose, never apply. Require this output format per finding:

```
BLOCKING C2 src/billing/invoice.py: reason
BEFORE:
<exact current plan text or code snippet under review>
AFTER:
<proposed replacement text, concrete enough to paste in>
NIT C1 src/auth/login.py: reason
APPROVE: no blocking findings
```

Rules for counters:

- Every BLOCKING must carry BEFORE plus AFTER. A BLOCKING without both is invalid, re-spawn the lane.
- BEFORE is the exact text being criticized, AFTER is the literal replacement.
- NITs need no BEFORE/AFTER. APPROVE needs no further detail.

Lane prompts:

- Counter 1, 12-factor: `You are a counter reviewer for code changes C1..Cn in the attached plan. Load no extra skills. Apply REFERENCE.md section 2 (12-factor for plans). Return only BLOCKING with BEFORE/AFTER, NIT, or APPROVE lines. Propose, never apply.`
- Counter 2, language quality: `You are a counter reviewer for C1..Cn. Load via skill tool: good-python if Python is touched, good-typescript if TypeScript is touched, plus tdd and bloodhound-antipatterns. Apply REFERENCE.md section 3. Return only BLOCKING with BEFORE/AFTER, NIT, or APPROVE lines. Propose, never apply.`
- Counter 3, seams: `You are a counter reviewer for C1..Cn. Load via skill tool: improve-codebase-architecture. Apply REFERENCE.md section 4. Prefer one seam. Return only BLOCKING with BEFORE/AFTER, NIT, or APPROVE lines. Propose, never apply.`

If a required skill ID is not installed, log the skip with reason and continue with the remaining lanes. Never invent skill content.

### 4. Merge via resolution subagent and rewrite plan in place

The orchestrator is the brain. It never applies counter AFTERs directly. It spawns one resolution subagent per wave to decide.

Resolution prompt shape:

```
You are the resolution subagent for wave <N> of <plan-path>.
Input: filtered C1..Cn list, current plan text, all counter BLOCKINGs with BEFORE/AFTER.
Output one decision per BLOCKING: ACCEPT (apply AFTER as-is), REJECT (drop with 1-line reason), or MODIFIED (give new AFTER that supersedes).
Rules: deduplicate identical Cn plus reason pairs, blocking beats nit on the same Cn, keep conflicting lane advice as two open items, never silently drop.
```

Apply only ACCEPT and MODIFIED AFTERs by editing the source plan file in place. Touch only code change sections. Do not reword unrelated prose. After editing, re-read the changed hunks to confirm only intended sections moved. Log every REJECT with reason in the final report.

### 5. Re-loop until approve or cap

Re-spawn the full wave on the updated plan. Repeat until all 3 return `APPROVE`, or cycle count reaches 10, or 2 consecutive cycles produce identical blocking sets.

On stop, report:

- Cycles run and per-cycle approve count.
- Files rewritten.
- Resolution decisions per BLOCKING: ACCEPT, REJECT with reason, or MODIFIED with new AFTER.
- Unresolved blocking findings, if any, with lane name and reason.
- Explicit verdict: `APPROVED` or `CAPPED with open items`.

Max 10 cycles is a hard cap. Prefer early exit on approve. Do not start cycle 11.

## Conditional decisions

If input has no code changes, stop after step 2 and report why.
If only one language is touched, load only that language skill in Counter 2.
If the plan touches UI, add a browser validation note but keep the 3-lane structure.
If a lane skill is missing, log the skip and continue instead of blocking the wave.
If blocking sets repeat twice, stop early and report the stable dissent instead of burning remaining cycles.

## Verification checklist

Before finishing, confirm:

- [ ] Filtered `C1..Cn` list exists and counters reviewed only that list
- [ ] All 3 counters were spawned per wave with loading evidence or logged skip
- [ ] Every BLOCKING carries BEFORE plus AFTER, invalid ones were re-spawned
- [ ] One resolution subagent ran per wave with ACCEPT, REJECT, or MODIFIED per BLOCKING
- [ ] Rewrite applied only accepted AFTERs and touched only code change sections
- [ ] Every blocking finding was accepted, modified, rejected with reason, or listed as unresolved
- [ ] Loop stopped on all-approve, repeat dissent, or cycle 10, never later
- [ ] Final report states cycles, resolution decisions, files rewritten, and APPROVED vs CAPPED

## Notifications

Send shell notifications via the `notify` service at lifecycle events.
Why: the user relies on the notification service to know when to return, not on polling chat.
Never let a `notify` failure block the loop: always append `|| true` and continue, logging the failure.
Every message must be max 300 characters in total: always pipe through `cut -c1-300`.

- Before pausing for human input (unclear rewrite target, no code changes to critique, CAPPED items needing a decision, any stop-and-ask):
  `notify "$(printf '%s' "Human input needed for critique <plan-path>: <reason>" | cut -c1-300)" || true`
- After the final report (APPROVED or CAPPED with open items):
  `notify "$(printf '%s' "Critique loop finished: <plan-path> - APPROVED|CAPPED" | cut -c1-300)" || true`
- On any blocking failure (gh fetch fails, rewrite fails, unexpected error):
  `notify "$(printf '%s' "Something went wrong with critique <plan-path>: <1-line cause>" | cut -c1-300)" || true`

## One-level references

Read only what the current wave needs. Do not follow nested links.

- Lane rubrics and output format: [REFERENCE.md](./REFERENCE.md).
- Test scenarios and trigger phrases: [EVALS.md](./EVALS.md).
