---
name: quality-verification
description: Use when setting up or auditing quality tooling and governance — pre-commit hooks, CI pipeline (lint, tests, coverage threshold, static analysis, architecture tests, dependency and secret scanning, performance tests), periodic quality reviews, technical debt tracking and rule exceptions (rules R-VER-01 to R-VER-10).
---

# Rule Verification

Rules that are not checked stop being followed. Automate as much as possible, complement with periodic reviews.

## Automated checks (every commit / PR)

- **R-VER-01** — Linter and formatter run as pre-commit hooks **and** in CI; CI fails on errors.
- **R-VER-02** — Unit and integration tests run in CI; the coverage threshold is blocking.
- **R-VER-03** — Static analysis (SonarQube or equivalent) with a blocking quality gate: cyclomatic complexity, duplication, code smells.
- **R-VER-04** — Automated architecture tests enforce the dependency rule.
- **R-VER-05** — Vulnerable dependency scanning and secret detection.
- **R-VER-06** — Benchmarks or performance tests on critical paths, alerting on regression.

## Setting up a project

1. Detect the stack (package manager, language, existing CI config) before proposing tools; reuse what is already there.
2. Fill the gaps for each rule, choosing the ecosystem's usual tool. Examples:

| Rule | JS/TS | Python | Java/Kotlin | .NET |
|---|---|---|---|---|
| R-VER-01 hooks | husky + lint-staged, ESLint, Prettier | pre-commit, ruff | Spotless, Checkstyle | dotnet format, Husky.Net |
| R-VER-02 coverage | Vitest/Jest coverage threshold | pytest-cov `--cov-fail-under` | JaCoCo rules | coverlet threshold |
| R-VER-03 static analysis | SonarQube / SonarCloud | SonarQube, radon | SonarQube | SonarQube |
| R-VER-04 architecture | dependency-cruiser | import-linter | ArchUnit | NetArchTest |
| R-VER-05 deps & secrets | npm audit / Dependabot, gitleaks | pip-audit, gitleaks | OWASP dependency-check, gitleaks | dotnet list package --vulnerable, gitleaks |
| R-VER-06 perf | benchmark job (e.g. tinybench, k6) | pytest-benchmark | JMH | BenchmarkDotNet |

3. Make every check **blocking** in CI (non-zero exit fails the pipeline).
4. Order the pipeline cheapest first: format/lint → build → unit tests + coverage → architecture tests → static analysis → security scans → integration/perf tests.
5. Document in the README how to run each check locally.

## Auditing a project

For each R-VER-01..06, report: present? blocking? where configured? Output a table `Rule | Status (ok / missing / non-blocking) | Evidence (file) | Action`.

## Periodic checks

| Frequency | Action |
|---|---|
| Every sprint / 2 weeks | Review quality indicators (coverage, technical debt, static analysis alerts). |
| Monthly | Audit a sample of merged PRs against the PR checklist (see `code-review` skill). |
| Monthly | Review production performance metrics (response times, slow queries, errors). |
| Quarterly | Architecture review: layer drift, circular dependencies, modules to refactor. |
| Quarterly | Update the rules document: obsolete rules, new rules, team feedback. |

When asked to run one of these, gather the data (CI reports, coverage, Sonar, git log of merged PRs, dependency graph) and produce a short report with findings ranked by priority.

## Handling deviations

- **R-VER-07** — Every deviation found is tracked as a technical debt ticket with a priority.
- **R-VER-08** — Reserve 10–20% of each iteration for debt reduction.
- **R-VER-09** — Boy Scout rule: leave code a little cleaner than you found it.
- **R-VER-10** — An exception to a rule is allowed only if **justified in writing** in the PR or the code.

For each deviation, draft a ticket: title, rule ID, location, impact, proposed fix, priority.
