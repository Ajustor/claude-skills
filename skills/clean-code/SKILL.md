---
name: clean-code
description: Use when writing, refactoring or reviewing code for readability — naming, function size and parameters, nesting, magic values, dead code, comments, immutability, error handling, input validation and unit tests (rules R-CC-01 to R-CC-24).
---

# Clean Code

Apply these rules to every line written or modified. Formatting is handled by the project's formatter and is never debated (R-CC-16) — run it, don't hand-format.

## Naming

- **R-CC-01** — Names reveal intent: `expirationDelayInDays`, not `d`.
- **R-CC-02** — Classes = nouns (`Invoice`), methods = verbs (`computeTotal`).
- **R-CC-03** — Booleans read like a question: `isActive`, `hasPermissions`.
- **R-CC-04** — No obscure abbreviations, no generic names (`data`, `info`, `manager`, `utils`).
- **R-CC-05** — One naming language per project, applied consistently. Check existing code before choosing.

## Functions

- **R-CC-06** — A function does **one thing**.
- **R-CC-07** — Aim for ≤ 20–30 lines; beyond that, consider splitting.
- **R-CC-08** — ≤ 3 parameters; beyond that, group them into an object.
- **R-CC-09** — No boolean flag parameters that change behavior (`do(x, true)`): write two functions.
- **R-CC-10** — Max 2–3 nesting levels; use **early returns** (guard clauses).
- **R-CC-11** — No hidden side effects: a read function modifies nothing.

## Structure and readability

- **R-CC-12** — No magic numbers or strings: use named constants.
- **R-CC-13** — No dead code or commented-out code: Git history is enough.
- **R-CC-14** — Comments explain the **why**, not the **what**.
- **R-CC-15** — Prefer immutability by default.
- **R-CC-16** — Formatting is automated.

## Error handling

- **R-CC-17** — Never swallow an exception silently (no empty `catch`, no `catch` that only logs and continues as if nothing happened unless justified).
- **R-CC-18** — Use explicit business errors rather than ambiguous return codes or `null`.
- **R-CC-19** — Validate input at system boundaries (API, UI, files).

## Tests

- **R-CC-20** — Every use case and business rule is covered by unit tests.
- **R-CC-21** — Tests follow **Arrange / Act / Assert** (or Given / When / Then).
- **R-CC-22** — A fixed bug = a regression test added (write it first, see it fail, then fix).
- **R-CC-23** — Tests are fast, independent and deterministic (no real clock, network, random without seed, shared state).
- **R-CC-24** — Respect the project's coverage target (e.g. 80% on domain and application layers).

## Refactoring procedure

1. Make sure tests cover the code you touch; add them first if missing.
2. Apply guard clauses to flatten nesting (R-CC-10).
3. Extract functions until each does one thing (R-CC-06/07) and rename to reveal intent (R-CC-01..04).
4. Replace flag parameters with separate functions (R-CC-09) and long parameter lists with an object (R-CC-08).
5. Replace literals with named constants (R-CC-12); delete dead and commented-out code (R-CC-13).
6. Run formatter, linter and tests.

Keep the refactor in the scope of the current goal; unrelated cleanup goes in a separate PR (R-REV-03), except small Boy Scout improvements on touched lines (R-VER-09).

## Review output

Report each finding as: `file:line — [level] R-CC-xx — problem → suggested fix`, with level `[blocking]`, `[suggestion]`, `[question]` or `[nit]`.
