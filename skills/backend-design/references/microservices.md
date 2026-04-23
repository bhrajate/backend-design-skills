# Microservices Architecture — Reference

Microservices is an architectural style where an application is built as a collection of small,
independently deployable services, each running its own process and communicating via APIs.

---

## When to Use Microservices

### ✅ Good fit:
- Large teams (Conway's Law: team structure → system structure)
- Multiple bounded contexts with distinct scalability needs
- Different tech stacks needed per domain
- Independent deployment cadences per team
- Proven domain model (you've already built the monolith)

### ❌ Bad fit (use a monolith first):
- Small team (< 10 engineers)
- Unclear domain boundaries
- Early-stage product (requirements still changing)
- Low traffic / no scale requirements
- First version of a product

> **Golden Rule**: Start with a well-structured modular monolith. Extract microservices when you have a concrete reason (scaling, team independence, technology mismatch).

---

## Core Principles

### 1. Single Responsibility per Service
Each service owns one business capability.

```
✅ Good boundaries:
- OrderService     → order lifecycle
- InventoryService → stock management
- NotificationService → emails, SMS, push

❌ Bad boundaries:
- UserOrderInventoryService (too much)
- OrderItemService (too granular — just a table)
```

### 2. Own Your Data
Each service has its own database. No shared databases.

```
✅ Correct:
OrderService    → orders_db (PostgreSQL)
InventoryService → inventory_db (PostgreSQL)
SearchService   → search_index (Elasticsearch)

❌ Wrong:
All services → shared_db  (tight coupling, one team's schema change breaks others)
```

### 3. Design for Failure
Any service can be down. Design with that assumption.

- Circuit breakers (e.g., Resilience4j, Hystrix)
- Timeouts and retries with exponential backoff
- Graceful degradation (return cached data, partial results)
- Bulkhead pattern (isolate failures)

### 4. Loose Coupling, High Cohesion
- Services communicate via APIs or events — not shared code or DB
- Internal implementation details never exposed

### 5. Decentralized Governance
Teams choose their own tech stack per service.

---

## Communication Patterns

### Synchronous (Request/Response)
- **REST**: Simple, widely understood. Use for queries and simple commands.
- **gRPC**: High performance, typed contracts. Use for internal service-to-service calls.

```
Client → [HTTP GET /orders/123] → OrderService → Response
```

### Asynchronous (Event-Driven)
- **Message Queue** (RabbitMQ, SQS): Point-to-point, guaranteed delivery
- **Event Bus / Pub-Sub** (Kafka, SNS): One publisher, many subscribers

```
OrderService --[OrderPlaced event]--> Kafka
                                       ├─→ InventoryService (reserve stock)
                                       ├─→ NotificationService (send email)
                                       └─→ AnalyticsService (track metrics)
```

**When to use async**: When the caller doesn't need an immediate response, or when multiple services need to react to the same event.

---

## Data Consistency Patterns

### Saga Pattern
Manages distributed transactions across services without a 2-phase commit.

**Choreography Saga** (event-driven):
```
OrderService        → publishes OrderCreated
InventoryService    → listens, reserves stock → publishes StockReserved
PaymentService      → listens, charges card   → publishes PaymentProcessed
OrderService        → listens, confirms order
```
If any step fails → compensating events are published to undo previous steps.

**Orchestration Saga** (central coordinator):
```
SagaOrchestrator → calls InventoryService.reserve()
                 → calls PaymentService.charge()
                 → calls OrderService.confirm()
                 → if failure: calls compensating actions
```

### CQRS (Command Query Responsibility Segregation)
Separate the read model from the write model.

```
Write side: Command → Domain Logic → Write DB (normalized, consistent)
Read side:  Query  → Read DB (denormalized, optimized for queries)

Sync via: Domain Events → Update Read Model
```

Use CQRS when: read/write patterns are very different, or heavy read load needs separate scaling.

---

## Service Decomposition Strategies

### By Business Capability (recommended)
Map to what the business does:
- Catalog, Orders, Payments, Shipping, Notifications

### By DDD Bounded Context (best for complex domains)
Each bounded context = one or a few services

### Strangler Fig Pattern (for migrating monoliths)
1. Build new service alongside monolith
2. Route some traffic to new service
3. Gradually strangle the monolith piece by piece

---

## Infrastructure Concerns

### API Gateway
- Single entry point for all clients
- Handles: auth, rate limiting, routing, SSL termination
- Examples: Kong, AWS API Gateway, nginx

### Service Discovery
- Services register themselves; clients look them up dynamically
- Examples: Consul, Kubernetes DNS, Eureka

### Distributed Tracing
- Trace a request across multiple services
- Examples: Jaeger, Zipkin, Datadog APM
- Requires: correlation IDs propagated in headers

### Centralized Logging
- Aggregate logs from all services
- Examples: ELK Stack, Datadog, Splunk
- Tag with: service name, trace ID, environment

### Health Checks
Every service exposes `/health` endpoint. Orchestrators (Kubernetes) use it for routing.

---

## Common Microservices Anti-Patterns

| Anti-Pattern | Problem | Fix |
|---|---|---|
| Distributed Monolith | Services tightly coupled, must deploy together | Fix boundaries via DDD |
| Chatty Services | Service A calls B, B calls C, C calls D per request | Aggregate data, use events |
| Shared Database | Two services share the same DB schema | Give each service its own DB |
| Synchronous Everything | No async, every failure cascades | Introduce event-driven communication |
| Nano-Services | Too granular (one service per DB table) | Merge into cohesive services |
| Premature Extraction | Splitting before domain is understood | Start monolith, extract later |

---

## Microservices vs Modular Monolith

| | Modular Monolith | Microservices |
|---|---|---|
| Deployment | Single unit | Independent |
| Scalability | Whole app scales | Per-service scaling |
| Complexity | Lower | Higher |
| Team size | Small/medium | Large, multiple teams |
| Latency | In-process calls | Network calls |
| Data isolation | Logical | Physical |
| Good for | Most products | Large orgs, proven domains |

**Recommended path**: Modular Monolith → extract services when there's a real reason.