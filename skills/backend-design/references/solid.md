# S.O.L.I.D Principles — Reference

## S — Single Responsibility Principle (SRP)

> A class/module should have only one reason to change.

**What it means**: One class = one job. If a class handles both business logic AND persistence AND email sending, it has three reasons to change.

**Signs of violation**:
- Class/function name contains "And" (e.g., `SaveUserAndSendEmail`)
- Large files with 500+ lines
- Testing one feature requires understanding many others

**Example (Python)**:
```python
# ❌ Violates SRP
class UserService:
    def create_user(self, data): ...
    def save_to_db(self, user): ...       # DB concern
    def send_welcome_email(self, user): ... # Email concern
    def generate_pdf_report(self): ...    # Report concern

# ✅ Correct
class UserService:
    def create_user(self, data): ...

class UserRepository:
    def save(self, user): ...

class EmailService:
    def send_welcome(self, user): ...
```

---

## O — Open/Closed Principle (OCP)

> Software entities should be open for extension but closed for modification.

**What it means**: Adding new behavior = adding new code, not changing existing tested code.

**Signs of violation**:
- Long `if/elif` or `switch` chains that grow with each new type
- Adding a new feature requires editing core logic

**Example (TypeScript)**:
```typescript
// ❌ Violates OCP — adding new payment type requires editing this function
function processPayment(type: string, amount: number) {
  if (type === 'credit') { ... }
  else if (type === 'paypal') { ... }
  // Must edit here for every new type
}

// ✅ Correct — new payment type = new class, no editing existing code
interface PaymentProcessor {
  process(amount: number): void;
}

class CreditCardProcessor implements PaymentProcessor { ... }
class PayPalProcessor implements PaymentProcessor { ... }
class CryptoProcessor implements PaymentProcessor { ... } // Just add this
```

---

## L — Liskov Substitution Principle (LSP)

> Subtypes must be substitutable for their base types without altering correctness.

**What it means**: If `Bird` has `fly()`, then every subclass of `Bird` must also be able to fly. A `Penguin extends Bird` that throws `UnsupportedOperationException` on `fly()` violates LSP.

**Signs of violation**:
- Subclass throws exceptions for inherited methods
- Subclass overrides a method to do nothing (empty override)
- You check `instanceof` before calling a method

**Fix**: Redesign the hierarchy. Use composition or separate interfaces.

```java
// ❌ Violates LSP
class Bird { void fly() {...} }
class Penguin extends Bird {
    void fly() { throw new UnsupportedOperationException(); }
}

// ✅ Correct
interface Flyable { void fly(); }
class Sparrow implements Flyable { ... }
class Penguin { /* no fly method */ }
```

---

## I — Interface Segregation Principle (ISP)

> Clients should not be forced to depend on interfaces they don't use.

**What it means**: Prefer many small, specific interfaces over one large general-purpose interface.

**Signs of violation**:
- Classes that implement an interface but leave many methods empty or throwing errors
- Interface has methods that belong to completely different roles

**Example**:
```typescript
// ❌ Violates ISP — PrinterScanner forces all implementors to support both
interface PrinterScanner {
  print(doc: string): void;
  scan(): string;
  fax(doc: string): void;  // not all printers can fax!
}

// ✅ Correct — segregated interfaces
interface Printer { print(doc: string): void; }
interface Scanner { scan(): string; }
interface Fax { fax(doc: string): void; }

class SimplePrinter implements Printer { ... }
class AllInOne implements Printer, Scanner, Fax { ... }
```

---

## D — Dependency Inversion Principle (DIP)

> High-level modules should not depend on low-level modules. Both should depend on abstractions.

**What it means**: Your business logic should not know about MySQL, SendGrid, or S3. It should depend on interfaces. The concrete implementations are injected.

**Signs of violation**:
- `new MySQLDatabase()` or `new SendGridClient()` inside a service class
- Testing a service requires setting up a real DB connection
- Changing from MySQL to Postgres requires editing business logic

**Example (Python)**:
```python
# ❌ Violates DIP — OrderService depends on concrete MySQL
class OrderService:
    def __init__(self):
        self.db = MySQLDatabase(host="localhost")  # hardcoded!

    def create_order(self, order):
        self.db.save(order)

# ✅ Correct — depends on abstraction, concrete class injected
from abc import ABC, abstractmethod

class OrderRepository(ABC):
    @abstractmethod
    def save(self, order): ...

class MySQLOrderRepository(OrderRepository):
    def save(self, order): ...

class InMemoryOrderRepository(OrderRepository):  # for testing!
    def save(self, order): ...

class OrderService:
    def __init__(self, repo: OrderRepository):
        self.repo = repo

    def create_order(self, order):
        self.repo.save(order)
```

**DIP enables**:
- Easy unit testing (inject mocks)
- Swappable implementations (DB, email provider, etc.)
- Clean Architecture layers

---

## S.O.L.I.D Summary

| Letter | Name | One-liner | Key smell |
|--------|------|-----------|-----------|
| S | Single Responsibility | One class, one job | Class name has "And" |
| O | Open/Closed | Extend don't modify | Long if/switch chains |
| L | Liskov Substitution | Subclass = drop-in replacement | Empty overrides / throws |
| I | Interface Segregation | Small focused interfaces | Implementing unused methods |
| D | Dependency Inversion | Depend on abstractions | `new ConcreteService()` in constructors |