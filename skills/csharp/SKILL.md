---
name: csharp
description: Use when writing, reviewing or setting up tooling for C# / .NET code — project settings (nullable, warnings as errors, analyzers), .editorconfig and dotnet format, async/await, nullability, records and immutability, exceptions, dependency injection, logging, testing with xUnit, CI and security scanning (rules R-CS-01 to R-CS-30).
---

# C# / .NET

Language-specific complement to `clean-code`, `clean-architecture` and `quality-verification`. Before changing anything, read the existing `Directory.Build.props`, `.editorconfig`, `global.json` and CI to reuse the project's choices.

## Project setup (tooling to put in place)

- **R-CS-01** — Target the current **LTS** .NET; pin the SDK in `global.json` (`rollForward: latestFeature`).
- **R-CS-02** — Centralise settings in `Directory.Build.props` at the repo root:
  ```xml
  <PropertyGroup>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
    <AnalysisLevel>latest-recommended</AnalysisLevel>
    <EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <Deterministic>true</Deterministic>
  </PropertyGroup>
  ```
- **R-CS-03** — Central package management (`Directory.Packages.props`, `ManagePackageVersionsCentrally`) and a lock file (`RestorePackagesWithLockFile`, `dotnet restore --locked-mode` in CI).
- **R-CS-04** — An `.editorconfig` holds naming and style rules with severities; `dotnet format --verify-no-changes` runs in a pre-commit hook (Husky.Net) and in CI.
- **R-CS-05** — Extra analyzers where relevant: `Microsoft.VisualStudio.Threading.Analyzers` (async), `SonarAnalyzer.CSharp`, `Meziantou.Analyzer`. Suppressions are local (`#pragma` / `[SuppressMessage]`) with a written justification (R-VER-10).
- **R-CS-06** — Tests: xUnit (or NUnit if already used) + an assertion library, coverage via coverlet with a blocking threshold; architecture tests with NetArchTest/ArchUnitNET.
- **R-CS-07** — Security: `dotnet list package --vulnerable --include-transitive` fails CI; Dependabot/Renovate; gitleaks.

## Language practices

### Nullability and types
- **R-CS-08** — Nullable reference types are on everywhere; no `!` (null-forgiving) without a comment explaining why it cannot be null.
- **R-CS-09** — Validate arguments at public boundaries with `ArgumentNullException.ThrowIfNull`, `ArgumentException.ThrowIfNullOrEmpty`, `ArgumentOutOfRangeException.ThrowIf*`.
- **R-CS-10** — Prefer `record` / `record struct` for value objects and DTOs; `init`-only or `required` properties; `readonly struct` for small value types.
- **R-CS-11** — Expose collections as `IReadOnlyList<T>` / `IReadOnlyCollection<T>`; never return `null` for a collection, return empty.
- **R-CS-12** — `sealed` by default for classes not designed for inheritance.
- **R-CS-13** — Use pattern matching and `switch` expressions; exhaustive switches over enums/closed hierarchies with a default arm that throws.

### Async
- **R-CS-14** — Async all the way: no `.Result`, `.Wait()`, `GetAwaiter().GetResult()` on async code.
- **R-CS-15** — No `async void` except event handlers.
- **R-CS-16** — Every async API accepts a `CancellationToken` (last parameter) and passes it down.
- **R-CS-17** — `ConfigureAwait(false)` in library code; not needed in ASP.NET Core app code.
- **R-CS-18** — Return `Task`/`ValueTask` directly instead of `async`/`await` only when there is no `using`/`try` around it.

### Resources, errors, logging
- **R-CS-19** — `using` declarations for every `IDisposable`/`IAsyncDisposable` (`await using`).
- **R-CS-20** — Throw specific exceptions; never `catch (Exception)` without rethrowing (`throw;`, never `throw ex;`) except at the top-level boundary. Use a `Result` type for expected business failures if the project has one.
- **R-CS-21** — Structured logging through `ILogger<T>` with message templates (`"Order {OrderId} created"`), never string interpolation; prefer `[LoggerMessage]` source generators on hot paths.
- **R-CS-22** — `HttpClient` through `IHttpClientFactory` / typed clients, never `new HttpClient()` per call.

### Design
- **R-CS-23** — Constructor injection via the built-in DI container; no service locator, no `static` mutable state.
- **R-CS-24** — Configuration through the Options pattern (`IOptions<T>` + `ValidateDataAnnotations().ValidateOnStart()`).
- **R-CS-25** — Time and randomness are injected (`TimeProvider`, not `DateTime.Now`) so tests are deterministic.
- **R-CS-26** — EF Core: `AsNoTracking()` for reads, projections with `Select`, no lazy loading in loops (N+1), migrations versioned.

### Naming and style
- **R-CS-27** — PascalCase for types/members/constants, camelCase for locals/parameters, `_camelCase` for private fields, `I` prefix for interfaces, `Async` suffix for async methods.
- **R-CS-28** — File-scoped namespaces, one public type per file, file name = type name.
- **R-CS-29** — `var` when the type is obvious from the right-hand side; explicit otherwise (enforced in `.editorconfig`).

### Tests
- **R-CS-30** — Test names `Method_State_ExpectedResult`; AAA layout; no `Thread.Sleep`; use `FakeTimeProvider`, in-memory fakes or Testcontainers for integration tests.

## Checklist before commit

1. `dotnet format --verify-no-changes`
2. `dotnet build -warnaserror`
3. `dotnet test --collect:"XPlat Code Coverage"` (threshold reached)
4. `dotnet list package --vulnerable --include-transitive`

## Review output

Report each finding as: `file:line — [level] R-CS-xx — problem → suggested fix`, with level `[blocking]`, `[suggestion]`, `[question]` or `[nit]`.
