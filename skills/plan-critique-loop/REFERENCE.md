# Plan-Critique-Loop - Reference

Lane rubrics for the 3 counter agents. Each lane reviews only the filtered `C1..Cn` code change list from `SKILL.md`.

## Contents

- 12-factor for plans
- Language quality lane
- Seams lane
- Output format and merge rules

## 2. 12-factor for plans

Apply to each `Cn`. Flag BLOCKING when the plan bakes in a violation. Flag NIT when the risk is theoretical.

- Config: no hardcoded secrets, endpoints, or model names in code. Must come from env or config. Watch `.env`, `credentials.json`, `*.pem` in diffs.
- Dependencies: declare explicit versions. No unpinned installs at runtime.
- Backing services: treat DB, queue, LLM endpoint as attached resources. Plan must name them and allow swap without code change.
- Stateless processes: no local disk or memory as source of truth between steps. State goes to DB or artifact.
- Logs: logs are event streams. No PII or secrets in logs. Domain never imports logging directly if a language skill forbids it.
- Parity: dev, staging, prod stay close. Note any `works locally` shortcut as BLOCKING.
- Port binding and concurrency: service startup is explicit. Background workers have retry and idempotency notes where the plan implies at-least-once delivery.
- Disposability: fast startup and graceful shutdown. Migrations are reversible or guarded.

If a factor does not apply, say so in one line instead of forcing a finding.

## 3. Language quality lane

Load via the `skill` tool before reviewing. Load only what the diff touches.

- Python touched: load `good-python`. Check parse-at-edge, frozen value objects, `Result` over `raise`, stratified errors, `mypy --strict` plus `ruff` plus `pytest` loop.
- TypeScript touched: load `good-typescript`. Check branded types, ADTs, total functions, stratified errors.
- Always load `tdd` and `bloodhound-antipatterns` in this lane. Check red-green-refactor seam per code change and name exact smells or anti-patterns by canonical name.

Do not apply Python rules to TypeScript or vice versa. If neither language is touched, limit this lane to `tdd` plus `bloodhound-antipatterns` and log the language skip.

## 4. Seams lane

Load `improve-codebase-architecture` via the `skill` tool before reviewing.

- Name the seam where each `Cn` will be tested. Prefer existing seams over new ones.
- Ideal is one seam for the whole change. If parallel lanes need one seam each, document the exception vs ideal equals 1.
- New seams go as high as possible. Unit-only coverage for an integration change is BLOCKING.
- Respect ADRs in the touched area. Conflicts are BLOCKING with file and ADR cited.

## Output format and resolution rules

Each counter proposes, never applies. The orchestrator decides via the resolution subagent. Each counter returns one finding block per issue:

```
BLOCKING C2 src/billing/invoice.py: hardcoded API key, move to env
BEFORE:
stripe_key = "sk-live-123"
AFTER:
stripe_key = os.environ["STRIPE_KEY"]
NIT C1 src/auth/login.py: long method, consider extract
APPROVE: no blocking findings
```

Rules for counters:

- BLOCKING without BEFORE plus AFTER is invalid.
- BEFORE quotes the exact plan text or code under review.
- AFTER is the literal replacement, paste-ready.
- NITs and APPROVE carry no BEFORE/AFTER.

Resolution subagent (one per wave, spawned by the orchestrator):

- Input: C1..Cn list, plan text, all BLOCKINGs with BEFORE/AFTER.
- Output per BLOCKING: `ACCEPT <Cn>: apply AFTER as-is`, `REJECT <Cn>: <1-line reason>`, or `MODIFIED <Cn>` plus a superseding AFTER block.
- Merge rules for the resolver:
- Deduplicate identical `Cn` plus reason pairs across lanes.
- Blocking beats nit on the same `Cn`.
- Conflicting lane advice is kept as two open items, never silently dropped.
- A wave passes only when all 3 lanes return `APPROVE` lines or all BLOCKINGs are REJECTed with reason logged.
- Only ACCEPT and MODIFIED AFTERs reach the plan rewrite. REJECTs are logged, never applied.
