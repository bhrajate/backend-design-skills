---
name: backend-design
description: >
  Expert backend system design guidance applying industry Golden Principles: S.O.L.I.D, DRY, KISS,
  YAGNI, Domain-Driven Design (DDD), Microservices Architecture, Clean Architecture, and more.
  Use this skill whenever the user asks about: backend architecture, system design, API design,
  service decomposition, domain modeling, code structure, design patterns, refactoring for
  maintainability, or any question involving how to organize and design backend systems.
  Also trigger for questions like "how should I structure my project?", "is this good design?",
  "how do I split my services?", or "what pattern should I use here?". Always consult this skill
  before giving backend architecture advice — even if the question seems simple.
---

# Backend Design Skill

This skill provides structured, principle-driven guidance for backend system design. Apply the
relevant principles based on the user's context (language, scale, team size, domain complexity).

---

## Golden Principles Overview

| Principle | Scope | Core Idea |
|-----------|-------|-----------|
| S.O.L.I.D | Class/Module level | OOP design rules for maintainable code |
| DRY | Any level | Eliminate duplication |
| KISS | Any level | Prefer simplicity |
| YAGNI | Feature level | Don't build what you don't need yet |
| Clean Architecture | App level | Decouple business logic from infrastructure |
| DDD | Domain level | Model software around the business domain |
| Microservices | System level | Decompose by business capability |
| 12-Factor App | Deployment level | Build portable, scalable cloud services |

---

## When to Read Reference Files

- **S.O.L.I.D deep dive** → `references/solid.md`
- **DDD (Domain-Driven Design)** → `references/ddd.md`
- **Microservices architecture** → `references/microservices.md`
- **Clean Architecture / Hexagonal** → `references/clean-architecture.md`
- **API Design principles** → `references/api-design.md`

Read only the reference(s) relevant to the user's question. Don't load all files at once.

---

## Quick Principle Guides

### DRY — Don't Repeat Yourself
Every piece of knowledge must have a **single, authoritative representation**.

- Extract repeated logic into shared functions/services
- Use configuration over hardcoded values
- Beware of "wrong DRY": don't merge things that look similar but represent different concepts

### KISS — Keep It Simple, Stupid
The simplest solution that works is usually the best.

- Avoid over-engineering for hypothetical future needs
- Prefer readable code over clever code
- A junior developer should be able to understand it

### YAGNI — You Aren't Gonna Need It
Don't add functionality until it's actually needed.

- Resist building "just in case" abstractions
- Works hand in hand with KISS
- Applies to: APIs, DB schemas, abstraction layers, microservice splits

### Separation of Concerns
Each module/layer/service handles one logical concern.

- Routes handle HTTP; services handle business logic; repositories handle data
- Don't let HTTP concepts leak into domain logic

### Fail Fast
Validate inputs early, throw errors loudly, don't silently swallow failures.

---

## How to Give Design Advice

1. **Identify the scope**: Is this a class-level, service-level, or system-level question?
2. **Identify violated principles**: What's wrong with the current design?
3. **Recommend the relevant principle(s)**: Name them explicitly
4. **Show concrete examples**: Always include before/after code or diagrams
5. **Consider tradeoffs**: Every pattern has a cost — mention it
6. **Calibrate to context**: A startup MVP needs different advice than a large enterprise system

---

## Common Anti-Patterns to Flag

| Anti-Pattern | Violates | Fix |
|---|---|---|
| God Class / God Service | SRP, SoC | Split by responsibility |
| Anemic Domain Model | DDD | Move logic into domain objects |
| Spaghetti dependencies | DIP, Clean Arch | Introduce interfaces/abstractions |
| Premature microservices | YAGNI, KISS | Start monolith, extract later |
| Shared DB across services | Microservices | Each service owns its data |
| HTTP calls in domain logic | Clean Arch | Use ports & adapters |
| Copy-paste code | DRY | Extract to shared module |
| Feature envy | SRP | Move method to the class it uses most |

---

## Language-Specific Notes

Adapt advice to the user's language:

- **Java/Kotlin**: Spring patterns, interface-based DI, JPA repositories
- **Python**: duck typing means interfaces are less formal; use ABCs or protocols when needed
- **TypeScript/Node**: leverage type system for domain modeling; beware of callback hell in infra
- **Go**: prefer composition over inheritance; explicit error handling aligns with Fail Fast
- **C#**: .NET DI container, repository pattern, CQRS with MediatR

---

## Scale-Aware Advice

| Stage | Recommended Architecture |
|-------|--------------------------|
| MVP / Early startup | Modular monolith, clean layering |
| Growing product | Modular monolith or mini-services |
| Large team / multi-domain | Microservices with DDD boundaries |
| Hyper-scale | Event-driven, CQRS, eventual consistency |

> ⚠️ The most common mistake: jumping to microservices too early. Start with a well-structured monolith.