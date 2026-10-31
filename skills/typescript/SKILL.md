---
name: typescript
description: Use when writing, reviewing or setting up tooling for TypeScript or JavaScript code (Node, Bun, browser, monorepos) — strict tsconfig, ESLint/typescript-eslint or Biome, Prettier, package manager and lockfile, typing practices, async and error handling, runtime validation, modules, testing with Vitest/bun test, CI and security (rules R-TS-01 to R-TS-30). Also the base for the vue, react and svelte skills.
---

# TypeScript / JavaScript

Language-specific complement to `clean-code`, `clean-architecture` and `quality-verification`. Framework skills (`vue`, `react`, `svelte`) build on this one. Before changing anything, read `package.json`, the lockfile, `tsconfig*.json`, lint/format configs and CI to reuse the project's choices (runtime Bun/Node, package manager, Nx/workspaces).

## Project setup (tooling to put in place)

- **R-TS-01** — New code is **TypeScript**, not JavaScript. Plain JS (config files, scripts) uses `// @ts-check` + JSDoc types.
- **R-TS-02** — One package manager per repo (bun, pnpm or npm), declared in `packageManager` and `engines`; lockfile committed; CI installs with the frozen lockfile (`bun install --frozen-lockfile`, `pnpm install --frozen-lockfile`, `npm ci`).
- **R-TS-03** — Strict `tsconfig` (shared base in monorepos, `extends` everywhere):
  ```jsonc
  {
    "compilerOptions": {
      "strict": true,
      "noUncheckedIndexedAccess": true,
      "exactOptionalPropertyTypes": true,
      "noImplicitOverride": true,
      "noFallthroughCasesInSwitch": true,
      "noPropertyAccessFromIndexSignature": true,
      "useUnknownInCatchVariables": true,
      "verbatimModuleSyntax": true,
      "isolatedModules": true,
      "forceConsistentCasingInFileNames": true,
      "skipLibCheck": true
    }
  }
  ```
- **R-TS-04** — Lint: ESLint flat config (`eslint.config.js`) with `typescript-eslint` **type-checked** presets (`strictTypeChecked`, `stylisticTypeChecked`), or Biome if the project uses it. Key rules: `no-floating-promises`, `no-misused-promises`, `no-explicit-any`, `switch-exhaustiveness-check`, `consistent-type-imports`, `no-unnecessary-condition`.
- **R-TS-05** — Format: Prettier (or Biome formatter) — never both linting style rules and a formatter fighting each other (`eslint-config-prettier`).
- **R-TS-06** — Pre-commit: husky (or lefthook / simple-git-hooks) + lint-staged running format + lint on staged files. CI runs `typecheck` (`tsc --noEmit` / `tsc --build`), lint, tests, build.
- **R-TS-07** — Tests: Vitest (or `bun test` in Bun projects, Jest only if already present) with a blocking coverage threshold; Playwright for E2E; MSW to mock HTTP.
- **R-TS-08** — Architecture boundaries: `dependency-cruiser`, Nx `enforce-module-boundaries` or `eslint-plugin-boundaries`.
- **R-TS-09** — Security & deps: `npm audit` / `pnpm audit` / `bun audit` (or osv-scanner) in CI, Renovate/Dependabot, gitleaks; `knip` periodically for unused files, exports and deps.

## Language practices

### Typing
- **R-TS-10** — No `any`. Use `unknown` and narrow it. `as` casts only at validated boundaries; never `as unknown as T` to force a type; no `!` non-null assertion without a justifying comment.
- **R-TS-11** — `@ts-ignore` forbidden; `@ts-expect-error` only with a description.
- **R-TS-12** — Let inference work for locals; annotate exported function signatures and return types of public APIs.
- **R-TS-13** — Model state with **discriminated unions** and check exhaustiveness (`satisfies never` / `assertNever` in the default branch).
- **R-TS-14** — Prefer `type` unions / `as const` objects over `enum`; use `satisfies` to check literals without widening.
- **R-TS-15** — Readonly by default: `readonly` properties, `ReadonlyArray<T>` / `readonly T[]` parameters, `as const`.
- **R-TS-16** — Branded types for identifiers that must not be mixed (`type UserId = string & { readonly __brand: 'UserId' }`).

### Boundaries
- **R-TS-17** — Every external input (HTTP body, env vars, `JSON.parse`, localStorage, files, messages) is validated at runtime with a schema (Zod, Valibot, TypeBox/ArkType) and the static type is **derived** from the schema.
- **R-TS-18** — Environment variables are parsed once at startup into a typed config object; the app fails fast if invalid.

### Async and errors
- **R-TS-19** — Every promise is awaited, returned, or explicitly `void`-ed with a comment; no `async` callback passed where a sync one is expected (`forEach`).
- **R-TS-20** — Parallelise independent awaits with `Promise.all` / `Promise.allSettled`; no `await` inside loops unless sequencing is required.
- **R-TS-21** — Throw `Error` subclasses (never strings/objects); in `catch (error: unknown)` narrow before use; rethrow with `{ cause }`. Use a Result type for expected business failures if the project has one.
- **R-TS-22** — Network calls have a timeout/cancellation (`AbortSignal.timeout()`).

### Syntax and modules
- **R-TS-23** — `const` by default, `let` when reassigned, never `var`; `===` only; `??` and `?.` instead of `||` defaults on possibly falsy values.
- **R-TS-24** — ES modules only; named exports (no default exports except where a framework requires them); no barrel files re-exporting everything in apps (they hurt tree-shaking and create cycles); no circular imports.
- **R-TS-25** — `import type` for type-only imports (enforced by `verbatimModuleSyntax`).
- **R-TS-26** — Use the platform first (`fetch`, `structuredClone`, `Intl`, `URL`, `crypto.randomUUID`) before adding a dependency; evaluate size and maintenance of every new dependency.
- **R-TS-27** — No `console.log` left in committed code; use the project's logger with structured fields.

### Naming
- **R-TS-28** — camelCase for variables/functions, PascalCase for types/classes/components, UPPER_SNAKE_CASE for true constants; file names in the project's convention (kebab-case by default); no `I` prefix on interfaces.

### Tests
- **R-TS-29** — Test behaviour through the public API; fake timers (`vi.useFakeTimers`) instead of real waits; no snapshot tests on large unstable outputs.
- **R-TS-30** — Type-level tests (`expectTypeOf`, `tsd`) for reusable generic utilities and library APIs.

## Checklist before commit

1. Format check (`prettier --check .` / `biome check`)
2. Lint (`eslint .` — zero warnings: `--max-warnings 0`)
3. Typecheck (`tsc --noEmit` or `tsc --build`)
4. Tests with coverage
5. Build

## Review output

Report each finding as: `file:line — [level] R-TS-xx — problem → suggested fix`, with level `[blocking]`, `[suggestion]`, `[question]` or `[nit]`.
