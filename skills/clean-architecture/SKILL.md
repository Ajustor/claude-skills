---
name: clean-architecture
description: Use when designing, placing or reviewing code structure — creating a feature, adding a use case, deciding which layer code belongs to, wiring dependency injection, organizing folders, or checking the dependency rule and SOLID principles (rules R-ARCH-01 to R-ARCH-06).
---

# Clean Architecture

Apply the project's binding architecture rules. Any deviation must be justified in writing (PR or code comment, R-VER-10).

## The dependency rule

Dependencies **always point inward**. The business core knows nothing about frameworks, database, UI or external APIs.

```
Infrastructure (DB, HTTP, UI, frameworks)
  └─ Adapters (controllers, repositories, presenters, gateways)
       └─ Application (use cases)
            └─ Domain (entities, rules)
```

## Layers

| Layer | Contains | Forbidden |
|---|---|---|
| **Domain** | Entities, value objects, business rules, domain interfaces | Any framework, ORM, HTTP or I/O import |
| **Application** | Use cases, inbound/outbound ports (interfaces), DTOs | Presentation logic, SQL, direct network calls |
| **Adapters** | Controllers, concrete repositories, mappers, presenters | Business rules |
| **Infrastructure** | Configuration, framework, drivers, dependency injection | Business rules |

## Rules

- **R-ARCH-01** — The domain is testable without a database, network or framework.
- **R-ARCH-02** — Use cases depend on **interfaces** (ports), never on concrete implementations.
- **R-ARCH-03** — Objects crossing layer boundaries are DTOs or domain objects, never ORM entities or HTTP request objects.
- **R-ARCH-04** — One use case = one business intent (`CreateOrder`, `CancelSubscription`). No catch-all "services".
- **R-ARCH-05** — Dependency injection is wired in the outermost layer (composition root).
- **R-ARCH-06** — Organize folders **by feature** first, then by layer.

## SOLID

- **S** — One reason to change per class/module.
- **O** — Open for extension, closed for modification.
- **L** — A subtype can replace its parent without breaking behavior.
- **I** — Many small specific interfaces over one large one.
- **D** — Depend on abstractions, not concretions.

## Procedure when adding a feature

1. Name the business intent → one use case per intent (R-ARCH-04).
2. Model the domain first: entities, value objects, rules — no framework imports.
3. Define ports (interfaces) the use case needs: repositories, gateways, notifiers.
4. Implement adapters for those ports in the adapters layer; map ORM/HTTP objects to DTOs or domain objects at the boundary (R-ARCH-03).
5. Wire everything in the composition root (R-ARCH-05).
6. Write unit tests for the domain and use case with in-memory fakes of the ports (R-ARCH-01).

Suggested layout (adapt names to the project's language):

```
src/
  orders/                 # feature first
    domain/
    application/          # use cases, ports, DTOs
    adapters/             # controllers, repositories, mappers
  billing/
    ...
  infrastructure/         # config, DI composition root, drivers
```

## Checking existing code

Look for these violations and cite the rule ID:

- Imports of framework/ORM/HTTP/I/O modules inside `domain/` → R-ARCH-01.
- A use case instantiating or importing a concrete repository/client → R-ARCH-02.
- ORM entities or request objects returned from or passed into use cases → R-ARCH-03.
- Classes named `*Service`/`*Manager` exposing many unrelated operations → R-ARCH-04.
- `new` of infrastructure classes outside the composition root → R-ARCH-05.
- Top-level folders like `controllers/`, `models/`, `services/` with no feature grouping → R-ARCH-06.
- Business rules inside controllers, repositories or config → layer table.

If the project has no automated architecture test, recommend one (R-VER-04): ArchUnit (Java/Kotlin), dependency-cruiser (JS/TS), import-linter (Python), NetArchTest (.NET), or the language equivalent.
