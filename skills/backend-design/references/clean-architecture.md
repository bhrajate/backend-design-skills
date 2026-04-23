# Clean Architecture & Hexagonal Architecture — Reference

Both patterns share the same core goal: **isolate business logic from infrastructure concerns**
(databases, HTTP, message queues, third-party APIs).

---

## The Core Idea

```
┌─────────────────────────────────────────────┐
│              Infrastructure                  │  HTTP, DB, Email, Queue...
│  ┌───────────────────────────────────────┐  │
│  │           Application Layer           │  │  Use cases, orchestration
│  │  ┌─────────────────────────────────┐  │  │
│  │  │         Domain Layer            │  │  │  Entities, business rules
│  │  │   (no external dependencies)   │  │  │
│  │  └─────────────────────────────────┘  │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘

Dependency rule: outer layers depend on inner layers. NEVER the reverse.
```

---

## Layers

### Domain Layer (innermost)
- Entities, Value Objects, Aggregates
- Domain Services
- Repository interfaces (abstractions, not implementations)
- Domain Events
- **Zero external dependencies** — no framework imports, no DB drivers

### Application Layer
- Use Cases / Application Services
- Commands and Queries (if using CQRS)
- Orchestrates domain objects
- Defines ports (interfaces for infrastructure)

### Infrastructure Layer (outermost)
- Repository implementations (SQL, MongoDB, Redis...)
- HTTP controllers / REST handlers
- Message queue consumers/producers
- Third-party API clients (Stripe, SendGrid...)
- Framework-specific code

---

## Ports & Adapters (Hexagonal Architecture)

Robert Martin's Clean Architecture and Alistair Cockburn's Hexagonal Architecture are closely related.

- **Port**: An interface defined by the application (what it needs)
- **Adapter**: Concrete implementation that satisfies the port

```
                     ┌──────────────────────────┐
HTTP Request ──────► │   REST Adapter (in)       │
                     │                           │
                     │    ┌────────────────┐     │
DB Query ◄───────────│    │  Application   │     │
                     │    │  + Domain      │     │
Kafka Event ────────►│    └────────────────┘     │
                     │                           │
Email ◄──────────────│   Email Adapter (out)      │
                     └──────────────────────────┘

Driving Adapters (in):  REST, gRPC, CLI, Test
Driven Adapters (out):  DB, Email, Queue, External APIs
```

---

## Example Structure (Python / FastAPI)

```
src/
├── domain/
│   ├── order.py              # Order entity + business rules
│   ├── money.py              # Value Object
│   └── repositories.py       # Abstract interfaces
│
├── application/
│   ├── place_order.py        # Use case
│   └── get_order.py          # Query use case
│
└── infrastructure/
    ├── http/
    │   └── order_router.py   # FastAPI routes
    ├── persistence/
    │   └── sql_order_repo.py # SQLAlchemy implementation
    └── messaging/
        └── kafka_publisher.py
```

---

## Example (TypeScript / Node)

```typescript
// domain/order.ts — no imports from outside domain
export class Order {
  private items: OrderItem[] = [];

  addItem(item: OrderItem): void {
    if (this.items.length >= 50) throw new DomainError('...');
    this.items.push(item);
  }
}

// domain/repositories.ts — interface only
export interface OrderRepository {
  findById(id: string): Promise<Order | null>;
  save(order: Order): Promise<void>;
}

// application/place-order.ts — orchestrates domain
export class PlaceOrderUseCase {
  constructor(private orderRepo: OrderRepository) {}  // depends on abstraction

  async execute(cmd: PlaceOrderCommand): Promise<string> {
    const order = Order.create(cmd.customerId);
    cmd.items.forEach(i => order.addItem(i));
    await this.orderRepo.save(order);
    return order.id;
  }
}

// infrastructure/persistence/prisma-order-repo.ts — concrete implementation
export class PrismaOrderRepository implements OrderRepository {
  async findById(id: string): Promise<Order | null> { ... }
  async save(order: Order): Promise<void> { ... }
}

// infrastructure/http/order-controller.ts — HTTP layer
export async function placeOrderHandler(req: Request, res: Response) {
  const useCase = new PlaceOrderUseCase(new PrismaOrderRepository());
  const orderId = await useCase.execute(req.body);
  res.status(201).json({ orderId });
}
```

---

## Benefits

| Benefit | How |
|---------|-----|
| Testable domain | Domain has no DB/HTTP deps — test with pure unit tests |
| Swappable infrastructure | Switch MySQL → Postgres without touching domain |
| Framework independence | Domain doesn't know Express/FastAPI/Spring exists |
| Clear boundaries | New developer knows exactly where business logic lives |

---

## Common Mistakes

1. **Leaking infrastructure into domain**: `@Column`, `@Entity` JPA annotations on domain objects
2. **Use cases doing too much**: HTTP parsing, DB transaction management all in one place
3. **Skipping the domain layer**: Just services calling repositories, no rich domain model
4. **Circular dependencies**: Infrastructure importing from Application importing from Infrastructure

---

## 12-Factor App (Deployment Principles)

Companion principles for cloud-native backends:

| Factor | Principle |
|--------|-----------|
| Codebase | One codebase, many deploys |
| Dependencies | Explicitly declare all dependencies |
| Config | Store config in environment, not code |
| Backing services | Treat DB, queue, cache as attached resources |
| Build/Release/Run | Strictly separate build and run stages |
| Processes | Stateless processes, share nothing |
| Port binding | Export services via port binding |
| Concurrency | Scale out via process model |
| Disposability | Fast startup, graceful shutdown |
| Dev/Prod parity | Keep environments as similar as possible |
| Logs | Treat logs as event streams |
| Admin processes | Run admin tasks as one-off processes |