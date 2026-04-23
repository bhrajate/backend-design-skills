# Domain-Driven Design (DDD) — Reference

DDD is an approach to software development that centers the design around the **business domain**,
using a shared language between developers and domain experts.

---

## Two Levels of DDD

### Strategic Design (the big picture)
How to divide and organize the overall system.

### Tactical Design (the code-level patterns)
How to model the domain inside a single bounded context.

---

## Strategic Design

### Ubiquitous Language
- A shared vocabulary used by both developers and business stakeholders
- Terms appear in code, conversations, documentation, and tests
- If a business expert says "Order" and the code says "PurchaseTransaction", that's a problem

### Bounded Context
- An explicit boundary within which a domain model applies
- The same concept can mean different things in different contexts
  - "Customer" in Sales context ≠ "Customer" in Support context
- Each bounded context typically maps to a microservice or a module

```
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│   Sales Context  │    │ Shipping Context │    │ Billing Context  │
│                  │    │                  │    │                  │
│  Customer        │    │  Recipient       │    │  Payer           │
│  Order           │    │  Shipment        │    │  Invoice         │
│  Product         │    │  Address         │    │  Payment         │
└──────────────────┘    └──────────────────┘    └──────────────────┘
```

### Context Map
Defines the relationships between bounded contexts:
- **Shared Kernel**: Two contexts share a subset of the model
- **Customer/Supplier**: One context depends on another
- **Anti-Corruption Layer (ACL)**: Translates between two incompatible models (common when integrating legacy systems)
- **Published Language**: Context exposes a well-defined API/schema for others

### Subdomain Types
| Type | Description | Example |
|------|-------------|---------|
| Core Domain | The primary business differentiator | Recommendation engine at Netflix |
| Supporting Subdomain | Supports core, but not differentiating | User management |
| Generic Subdomain | Common problem, use off-the-shelf | Authentication, billing |

---

## Tactical Design Patterns

### Entity
- Has a unique identity that persists over time
- Identity matters more than attributes
- Example: `User`, `Order`, `Product`

```python
class Order:
    def __init__(self, order_id: str, customer_id: str):
        self.order_id = order_id   # identity
        self.customer_id = customer_id
        self.items: list[OrderItem] = []
        self.status = OrderStatus.PENDING

    def add_item(self, item: OrderItem):
        if self.status != OrderStatus.PENDING:
            raise DomainException("Cannot add items to a confirmed order")
        self.items.append(item)
```

### Value Object
- Defined by its attributes, not identity
- Immutable — no setters
- Example: `Money`, `Address`, `Email`, `DateRange`

```python
from dataclasses import dataclass

@dataclass(frozen=True)   # frozen = immutable
class Money:
    amount: Decimal
    currency: str

    def add(self, other: 'Money') -> 'Money':
        if self.currency != other.currency:
            raise ValueError("Currency mismatch")
        return Money(self.amount + other.amount, self.currency)
```

### Aggregate
- A cluster of entities and value objects treated as a single unit
- Has one **Aggregate Root** — the only entry point from outside
- Enforces invariants (business rules) across the whole cluster
- Other aggregates reference it only by ID

```python
class Order:  # Aggregate Root
    def __init__(self, order_id: str):
        self.order_id = order_id
        self._items: list[OrderItem] = []   # private — accessed only through root

    def add_item(self, product_id: str, quantity: int, price: Money):
        # Enforce invariant: max 50 items per order
        if len(self._items) >= 50:
            raise DomainException("Order cannot have more than 50 items")
        self._items.append(OrderItem(product_id, quantity, price))

    def total(self) -> Money:
        return sum(item.subtotal() for item in self._items)
```

### Domain Service
- Logic that doesn't naturally belong to a single entity or value object
- Stateless
- Example: `PricingService`, `ShippingCalculator`, `FraudDetector`

```python
class PricingService:
    def calculate_discount(self, order: Order, customer: Customer) -> Money:
        # Involves both Order and Customer — doesn't belong to either alone
        ...
```

### Domain Event
- Something that happened in the domain, past tense
- Used to communicate across bounded contexts
- Example: `OrderPlaced`, `PaymentFailed`, `UserRegistered`

```python
@dataclass
class OrderPlaced:
    order_id: str
    customer_id: str
    total: Money
    occurred_at: datetime
```

### Repository
- Abstracts data persistence for aggregates
- Looks like an in-memory collection to the domain
- One repository per aggregate root

```python
from abc import ABC, abstractmethod

class OrderRepository(ABC):
    @abstractmethod
    def find_by_id(self, order_id: str) -> Optional[Order]: ...

    @abstractmethod
    def save(self, order: Order) -> None: ...

    @abstractmethod
    def find_by_customer(self, customer_id: str) -> list[Order]: ...
```

### Application Service
- Orchestrates domain objects to fulfill a use case
- No business logic — just coordination
- Handles transactions, publishes events

```python
class OrderApplicationService:
    def place_order(self, command: PlaceOrderCommand) -> str:
        customer = self.customer_repo.find_by_id(command.customer_id)
        order = Order.create(customer)
        for item in command.items:
            order.add_item(item.product_id, item.quantity, item.price)
        self.order_repo.save(order)
        self.event_bus.publish(OrderPlaced(order.id, customer.id, order.total()))
        return order.id
```

---

## Anemic Domain Model Anti-Pattern

The most common DDD violation: entities are just data bags with no behavior, and all logic sits in services.

```python
# ❌ Anemic — Order is just a struct
class Order:
    status: str
    items: list

class OrderService:
    def confirm_order(self, order: Order):
        if order.status != "pending": raise Exception(...)
        order.status = "confirmed"    # logic lives here, not in Order

# ✅ Rich domain model — behavior lives on the entity
class Order:
    def confirm(self):
        if self.status != OrderStatus.PENDING:
            raise DomainException("Only pending orders can be confirmed")
        self.status = OrderStatus.CONFIRMED
        self._raise_event(OrderConfirmed(self.id))
```

---

## DDD Summary

| Pattern | Belongs to | Purpose |
|---------|------------|---------|
| Entity | Tactical | Identity-based object with behavior |
| Value Object | Tactical | Immutable, equality by value |
| Aggregate | Tactical | Consistency boundary, enforces invariants |
| Domain Service | Tactical | Stateless logic spanning multiple objects |
| Domain Event | Tactical | Record of something that happened |
| Repository | Tactical | Persistence abstraction for aggregates |
| Application Service | Tactical | Use case orchestration |
| Bounded Context | Strategic | Explicit model boundary |
| Ubiquitous Language | Strategic | Shared vocabulary |
| Context Map | Strategic | Relationship between contexts |