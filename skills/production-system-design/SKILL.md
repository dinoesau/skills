---
name: production-system-design
description: >
  Use when the user asks to design, architect, build, or refactor a production
  system, service, or application. Triggers on: "design a system",
  "diseñar un sistema", "system design", "software architecture",
  "production system", "resilient system", "scalable system",
  "high availability", "microservices", "url shortener",
  "distributed system", "cloud architecture", "failure resilience",
  "production-grade", "escalable", "sistema de producción".
  Enforces a strict two-phase workflow: resilient Software Architecture FIRST,
  Software Design SECOND. Never writes implementation code during the
  architecture phase.
---

# Production System Design

Design production-grade systems by separating **what the system is** from
**how the code works**.

- **Software Architecture** = high-level structure, quality attributes,
  resilience strategy, technology decisions.
- **Software Design** = component internals, interfaces, design patterns,
  code skeletons.

**Golden rule:** never write classes, interfaces, or implementation code during
Phase A. If the user asks for code before architecture is validated, remind
them of the two-phase workflow and ask whether to proceed.

## When to Use This Skill

Use this skill when the user wants to:

- Design or architect a new system or service.
- Refactor an existing system for production.
- Evaluate scalability, availability, or resilience.
- Prepare for a system-design interview with a production mindset.
- Translate business requirements into a technical solution.

## Phase A: Software Architecture

Produce the architecture artifact before touching design details.

### A.1 Requirements

Capture both functional and non-functional requirements:

- **Functional:** what the system must do (features, user flows, APIs).
- **Non-functional:** scalability, availability, latency, durability,
  security, maintainability, cost, compliance.

### A.2 Constraints and Assumptions

List explicit and implicit constraints:

- Expected traffic (QPS, peak, growth).
- Data volume and retention.
- Budget and team size.
- Existing tech stack and integrations.
- Compliance (GDPR, HIPAA, SOC2, etc.).

### A.3 Quality Attributes

Prioritize the top 3–5 quality attributes for this system. Examples:

- Availability (99.9%, 99.99%)
- Scalability (horizontal, vertical)
- Latency (p50, p95, p99)
- Durability (data loss tolerance)
- Consistency (strong, eventual)
- Security (authentication, authorization, encryption)
- Maintainability (deploy frequency, MTTR)

### A.4 Back-of-the-Envelope Estimation

Provide rough numbers to justify architectural choices:

- Requests per second (read vs write).
- Storage per year.
- Bandwidth (ingress/egress).
- Memory needed for caches.
- Number of nodes / shards.

### A.5 Architectural Style

Choose and justify one or a combination:

- Monolith
- Microservices
- Service-oriented architecture (SOA)
- Event-driven architecture
- Serverless / FaaS
- Layered architecture
- CQRS / Event sourcing
- Hexagonal / Clean architecture

### A.6 Components and Data Flow

Define high-level components:

- Clients / gateways / load balancers
- Services / workers / processors
- Databases (primary, replicas)
- Caches
- Message queues / event buses
- External integrations
- CDNs / object storage

Include a **Mermaid diagram** showing the architecture (C4 Container or
Component diagram is preferred).

### A.7 Data Model

Describe the main entities and their relationships at a high level:

- Read-heavy vs write-heavy patterns.
- Indexes and partitioning strategy.
- Hotspots and mitigation.

### A.8 Resilience Checklist

Mark and justify each item that applies:

- [ ] **Redundancy / replication** — multi-node, multi-zone, multi-region.
- [ ] **Failover** — automatic promotion of replicas.
- [ ] **Circuit breaker** — fail fast on unhealthy dependencies.
- [ ] **Bulkhead** — isolate failures to a subset of resources.
- [ ] **Retry with backoff + jitter** — avoid thundering herds.
- [ ] **Timeouts** — fail fast instead of hanging.
- [ ] **Rate limiting / throttling** — protect services from overload.
- [ ] **Load balancing** — distribute traffic evenly.
- [ ] **Caching strategy** — cache-aside, write-through, TTL, invalidation.
- [ ] **Async processing / queues** — decouple producers and consumers.
- [ ] **Idempotency** — safe retries for writes.
- [ ] **Graceful degradation** — partial functionality under stress.
- [ ] **Health checks / observability** — logs, metrics, traces, alerts.
- [ ] **Backups / disaster recovery** — RPO, RTO targets.
- [ ] **Feature flags** — safe rollout and rollback.

### A.9 Technology Decisions

Present options without imposing a specific stack:

| Concern | Option A | Option B | Option C | Recommendation |
|---|---|---|---|---|
| Database | SQL | NoSQL | NewSQL | justify |
| Cache | In-memory | Redis | CDN | justify |
| Queue | Kafka | RabbitMQ | SQS | justify |

Explain trade-offs (scalability, operational complexity, team expertise,
cost).

### A.10 Architecture Decision Record (ADR)

Summarize the most important decision in ADR format:

- **Context:** what forces are at play?
- **Decision:** what was chosen?
- **Consequences:** what improves and what gets harder?

### Phase A Output

A Markdown architecture document containing:

1. Requirements and constraints
2. Quality attributes
3. Estimations
4. Architectural style and rationale
5. Component diagram (Mermaid)
6. Data model
7. Resilience checklist
8. Technology decisions
9. ADR summary

## Gate: Validate Before Designing

Before moving to Phase B, explicitly ask:

> Architecture is ready. Do you want to adjust anything before we move to
> Software Design (component breakdown, design patterns, and code skeletons)?
> Reply `proceed` or tell me what to change.

Do not proceed until the user confirms or refines the architecture.

## Phase B: Software Design

Only after architecture is validated, design the internals.

### B.1 Component Decomposition

Break each architectural component into concrete modules:

- Responsibilities (single responsibility per module).
- Public API surface.
- Dependencies between modules.

### B.2 Interfaces and Contracts

Define:

- Service APIs (REST, gRPC, GraphQL, events).
- Request/response schemas.
- Error contracts.
- Event schemas (if event-driven).

### B.3 Design Patterns

Pick patterns justified by the problem, not by fashion:

- **Creational:** Factory, Builder, Singleton (use sparingly).
- **Structural:** Adapter, Repository, Facade, Decorator.
- **Behavioral:** Strategy, Observer, Command, Saga.
- **Integration:** Outbox, API Gateway, CQRS, Event Sourcing.

Include a **Mermaid class diagram** or **sequence diagram**.

### B.4 Design Principles

Apply deliberately:

- **SOLID**
- **DRY** — avoid duplication, but prefer duplication over wrong abstraction.
- **KISS** — simplest solution that works.
- **YAGNI** — don't add speculative features.
- **Composition over inheritance**
- **Law of Demeter**

### B.5 Code Skeleton

Provide language-agnostic pseudocode plus a concrete skeleton in the user's
preferred language. Include only structure and signatures — no full
implementation.

Example structure:

```python
class ShortUrlService:
    def __init__(self, repository: UrlRepository, cache: Cache, hasher: HashGenerator):
        ...

    async def create_short_url(self, original_url: str) -> Result[ShortUrl, Error]:
        ...

    async def resolve(self, short_code: str) -> Result[str, Error]:
        ...
```

### B.6 Test Strategy

Outline the test pyramid:

- Unit tests for domain logic.
- Integration tests for repositories and external clients.
- Contract tests for service boundaries.
- Load / chaos tests for resilience assumptions.

### Phase B Output

A Markdown design document containing:

1. Component decomposition
2. Interfaces and contracts
3. Design patterns with rationale
4. Applied principles
5. Class / sequence diagram (Mermaid)
6. Code skeletons
7. Test strategy

## Final Output Structure

Create these artifacts in the project:

```
docs/
├── architecture/
│   ├── 01-requirements.md
│   ├── 02-architecture-overview.md
│   ├── 03-resilience-plan.md
│   └── 04-technology-decisions.md
└── design/
    ├── 01-component-design.md
    └── 02-code-skeleton.md
```

If the project is small, combine into two files:

- `docs/ARCHITECTURE.md`
- `docs/DESIGN.md`

## Language

Respond in the same language the user is using. Generate all artifacts in that
language.

## Examples

See [EXAMPLES.md](./EXAMPLES.md) for a complete walkthrough using a URL
shortener.

## Reference

See [REFERENCE.md](./REFERENCE.md) for a catalog of architectural patterns,
resilience patterns, design patterns, and quality attributes.
