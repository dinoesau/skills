# Reference: Architecture, Resilience, and Design

A quick catalog of patterns, qualities, and principles for production system
design.

## Architectural Patterns

| Pattern | When to Use | Trade-offs |
|---|---|---|
| **Monolith** | Small team, simple domain, rapid iteration | Easy to deploy; hard to scale independently |
| **Microservices** | Multiple teams, independent deployability, polyglot needs | Operational complexity; network latency |
| **Service-Oriented Architecture (SOA)** | Enterprise integration, shared services | Heavier governance; reusable but rigid |
| **Event-Driven Architecture** | Loose coupling, async workflows, real-time processing | Harder to debug; eventual consistency |
| **Serverless / FaaS** | Variable traffic, low ops overhead | Cold starts; vendor lock-in |
| **Layered Architecture** | CRUD apps, clear separation of concerns | Can become rigid; leaks between layers |
| **Hexagonal Architecture** | Testability, domain-centric design | Steeper learning curve; more files |
| **CQRS** | Read/write asymmetry, complex query needs | Complexity; eventual consistency |
| **Event Sourcing** | Audit trails, temporal queries | Complexity; storage growth |

## Resilience Patterns

| Pattern | Purpose |
|---|---|
| **Redundancy / Replication** | Survive node or zone failures |
| **Failover** | Automatically switch to healthy instances |
| **Circuit Breaker** | Stop calling a failing dependency temporarily |
| **Bulkhead** | Isolate failures to a subset of resources |
| **Retry with Backoff + Jitter** | Recover from transient failures without amplifying load |
| **Timeout** | Avoid waiting indefinitely for slow dependencies |
| **Rate Limiting** | Protect services from overload or abuse |
| **Load Balancing** | Distribute traffic evenly across instances |
| **Cache-Aside / Write-Through** | Reduce load on primary data stores |
| **Queue-Based Load Leveling** | Absorb traffic spikes with async queues |
| **Idempotency** | Make retries safe for write operations |
| **Graceful Degradation** | Maintain partial functionality during failures |
| **Health Checks** | Detect unhealthy instances quickly |
| **Observability (Logs/Metrics/Traces)** | Understand and alert on system behavior |
| **Backups / Disaster Recovery** | Recover from data loss or region failures |
| **Feature Flags** | Roll out and roll back changes safely |

## Design Patterns

### Creational

| Pattern | Use Case |
|---|---|
| **Factory** | Create objects without exposing construction logic |
| **Builder** | Construct complex objects step by step |
| **Singleton** | One instance only; use sparingly |

### Structural

| Pattern | Use Case |
|---|---|
| **Repository** | Abstract data access |
| **Adapter** | Bridge incompatible interfaces |
| **Facade** | Simplify a complex subsystem |
| **Decorator** | Add behavior dynamically |

### Behavioral

| Pattern | Use Case |
|---|---|
| **Strategy** | Swap algorithms at runtime |
| **Observer** | React to state changes |
| **Command** | Encapsulate requests as objects |
| **Saga** | Manage distributed transactions |

### Integration

| Pattern | Use Case |
|---|---|
| **API Gateway** | Single entry point for clients |
| **Outbox** | Reliably publish events from a database transaction |
| **CQRS** | Separate read and write models |
| **Event Sourcing** | Store state as a sequence of events |

## Quality Attributes

| Attribute | Question to Answer |
|---|---|
| **Availability** | What uptime target (e.g., 99.9%, 99.99%)? |
| **Scalability** | How does the system grow with load? |
| **Latency** | What are acceptable p50/p95/p99 response times? |
| **Durability** | How much data loss is acceptable? |
| **Consistency** | Strong or eventual? |
| **Security** | Authentication, authorization, encryption? |
| **Maintainability** | How easy is it to change and deploy? |
| **Testability** | How easy is it to verify behavior? |
| **Observability** | Can we understand failures quickly? |
| **Cost Efficiency** | What is the operational cost per request? |

## Design Principles

| Principle | Summary |
|---|---|
| **SOLID** | Single responsibility, Open/closed, Liskov substitution, Interface segregation, Dependency inversion |
| **DRY** | Don't repeat yourself, but prefer duplication over wrong abstraction |
| **KISS** | Keep it simple and straightforward |
| **YAGNI** | You aren't gonna need it — avoid speculative features |
| **Composition over Inheritance** | Favor composing behaviors over deep class hierarchies |
| **Law of Demeter** | Talk to friends, not strangers — limit object navigation |
| **Fail Fast** | Detect and surface errors as early as possible |
| **Defensive Programming** | Validate inputs and assumptions at boundaries |

## Useful Heuristics

- **Start simple, scale when measured.** Do not microservice a system without
  evidence it is needed.
- **Design for failure.** Assume every dependency will fail eventually.
- **Make state explicit.** Implicit state is harder to debug and scale.
- **Prefer idempotent operations.** They make retries, recovery, and scaling
  easier.
- **Optimize for readability.** Code is read more often than it is written.
- **Measure before optimizing.** Use load tests and traces to find real
  bottlenecks.
