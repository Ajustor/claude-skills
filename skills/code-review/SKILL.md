---
name: code-review
description: Use when reviewing a pull request or diff, self-reviewing before submitting, writing a PR description, or checking a PR against the team checklist — covers review process rules, what to check (functional, architecture, design, clean code, performance, tests, security) and how to word comments (rules R-REV-01 to R-REV-07).
---

# Code Review

## Process rules

- **R-REV-01** — No merge into the main branch without at least **one approved review**.
- **R-REV-02** — Short PRs: ideally < 400 changed lines. Split otherwise.
- **R-REV-03** — One PR = one goal. No unrelated refactoring mixed with a feature.
- **R-REV-04** — PR description states: context, what changes, how to test, risks.
- **R-REV-05** — The author reviews their own PR before submitting it.
- **R-REV-06** — Build, tests and lint must pass **before** human review.
- **R-REV-07** — Target review turnaround: within 24 business hours.

## Review procedure

1. **Gate checks**: CI green (R-REV-06)? Size < 400 lines (R-REV-02)? Single goal (R-REV-03)? Description complete (R-REV-04)? If not, say so first.
2. Read the description, then the diff, then the surrounding code needed to understand it.
3. Check in this order:
   1. **Functional** — does it do what is asked? Edge cases (empty, null, limits, concurrency, errors)?
   2. **Architecture** — dependency rule respected? Right layer? (see `clean-architecture` skill)
   3. **Design** — patterns justified? Over-engineering? (see `design-patterns` skill)
   4. **Clean Code** — naming, function size, duplication, readability (see `clean-code` skill)
   5. **Performance** — N+1, expensive loops, unnecessary I/O, missing pagination (see `performance` skill)
   6. **Tests** — present, relevant, error cases covered, regression test for bug fixes (R-CC-22)
   7. **Security** — input validation, no secrets in code, injections (SQL, command, XSS), access control
4. Note what is done well.
5. Conclude with a verdict: **approve**, **approve with suggestions**, or **request changes** (any `[blocking]` item).

## Comment style

- Comment on the **code**, never the person.
- Prefix every comment with its level:
  - `[blocking]` — must be fixed before merge (rule violation, bug, security issue).
  - `[suggestion]` — improvement, author decides.
  - `[question]` — need clarification.
  - `[nit]` — minor, non-blocking.
- Explain the **why** and cite the rule ID when one applies; propose an alternative when possible.
- An exception to a rule is acceptable if justified in writing (R-VER-10) — check for that justification before blocking.

Format:

```
path/to/file.ext:42 — [blocking] R-ARCH-03 — The controller passes the ORM entity `OrderEntity` to the use case. Map it to `OrderDto` in the adapter so the application layer stays ORM-free.
```

## Writing a PR description (author side)

```markdown
## Context
Why this change is needed (ticket link).

## What changes
- ...

## How to test
Steps, commands, data.

## Risks
Migration, performance, backward compatibility, rollout.
```

Then append the checklist below and tick it honestly after a self-review (R-REV-05).

## PR checklist

```markdown
### Architecture
- [ ] The domain depends on no framework or infrastructure
- [ ] Use cases depend on interfaces
- [ ] Code is placed in the right layer

### Design
- [ ] Every pattern used addresses a real need
- [ ] No duplicated business logic

### Clean Code
- [ ] Explicit names, short single-responsibility functions
- [ ] No dead code, commented-out code or magic values
- [ ] Errors handled explicitly

### Performance
- [ ] No N+1 queries or I/O inside loops
- [ ] Paginated lists, indexed queries
- [ ] Non-trivial optimizations measured and documented

### Tests & quality
- [ ] Tests added or updated (including regression test for bugs)
- [ ] Green CI: build, lint, tests, static analysis, architecture tests
- [ ] Self-review done, complete PR description
```
