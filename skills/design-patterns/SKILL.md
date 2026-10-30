---
name: design-patterns
description: Use when choosing whether to introduce a design pattern or abstraction (Repository, Factory, Strategy, Adapter, Decorator, CQRS, etc.), when refactoring toward one, or when spotting over-engineering and anti-patterns such as God Objects, anemic domain models or premature interfaces (rules R-PAT-01 to R-PAT-04).
---

# Design Patterns

A pattern is justified by a **real problem**, never by anticipation. Apply **YAGNI**: no abstraction without at least one concrete need, ideally two.

## Rules

- **R-PAT-01** — Every pattern introduced is named explicitly (class name, comment or PR description).
- **R-PAT-02** — Prefer **composition over inheritance**.
- **R-PAT-03** — Do not use a pattern when a simple function is enough.
- **R-PAT-04** — No global Singletons; use a single instance managed by dependency injection.

## Decision procedure

Before introducing a pattern, answer in order:

1. **What concrete problem exists today?** If you cannot point to code that hurts now, stop (YAGNI).
2. **Would a plain function or a small module solve it?** If yes, use that (R-PAT-03).
3. **Which need from the table below matches?** Pick that pattern, not a fancier one.
4. **Name it** in the type name or a short comment, and in the PR description (R-PAT-01).

## Pattern by need

| Need | Pattern |
|---|---|
| Isolate data access | **Repository** |
| Create complex or variable objects | **Factory**, **Builder** |
| Swap an algorithm at runtime | **Strategy** |
| React to events without coupling | **Observer**, **Domain Events** |
| Adapt an external API to our interfaces | **Adapter** |
| Simplify a complex subsystem | **Facade** |
| Add behavior without modifying the class | **Decorator** |
| Encapsulate an action (undo, queue, log) | **Command** |
| Manage states and transitions | **State** |
| Separate reads and writes | **CQRS** (only when complexity justifies it) |
| Group writes into a transaction | **Unit of Work** |
| Immutable, validated business values | **Value Object** |

## Anti-patterns to flag

- **God Object** — classes of several hundred lines with multiple responsibilities. Split by responsibility.
- **Anemic Domain Model** — entities are bags of getters/setters while business rules live in services. Move rules onto the entities/value objects.
- **Spaghetti / circular coupling** between modules. Break cycles with a port or a domain event.
- **Copy-pasted business logic** (DRY violation). Extract once, in the domain.
- **Premature abstraction** — an interface with a single implementation and no testing or evolution reason. Inline it. (Ports at layer boundaries are justified by testing — see the `clean-architecture` skill.)
- **Inheritance used for reuse** → replace with composition (R-PAT-02).
- **Global singletons / static mutable state** → inject a single instance (R-PAT-04).

## Output

When recommending or reviewing a pattern, state: the problem, the chosen pattern, why a simpler option is not enough, and the rule IDs involved.
