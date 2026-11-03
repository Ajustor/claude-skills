---
name: dotnet-sqlserver-pitfalls
description: Use when an EF Core / SQL Server / ASP.NET Core behavior looks impossible — retry not replaying a 500, enum value 0 not saved, removed child row still in DB, migration full of unrelated columns, CS0121 on EF.Functions, slow OR with EXISTS, Msg 1934 / 468 / 245 / 102 / 16202, multipart 400 bypassing exception filters, Swagger info leaking between hosts, authorization attribute silently ignored, green tests but 403 in production.
---

# .NET / SQL Server field pitfalls

Empirical gotchas, each reproduced: **symptom → cause → fix**.

## EF Core

- **Retry does not replay a connection killed mid-transaction.** It surfaces as `SqlException` **number 0, severity 20**, absent from EF's transient list (40197, 40613, 10928, 10054, -2 are in). → `EnableRetryOnFailure(..., errorNumbersToAdd: [0])`. Trade-off: a drop right after a server-accepted commit replays the operation.
- **Enum member valued 0 never inserted when the column has `HasDefaultValue`.** EF treats the CLR default as "unset"; the DB default wins, silently, on INSERT only. Only hint: `RelationalEventId.BoolWithDefaultWarning` ("…no configured sentinel value"), visible with EF logs at `Warning`. → `HasSentinel(<non-member value>)` or a nullable property.
- **`DbSet.Update(graph)` does not delete a child removed from an `Include`d collection.** The removed `OrderLine` stays. Mocks never show it. → Delete removed children explicitly, inside the execution-strategy lambda.
- **EF Core 10 `Contains(collection)`:** 3 items → `IN (@p1,@p2,@p3)`; 2 500+ → `OPENJSON`, zero discrete parameters, so the 2 100-parameter limit is never hit. Remaining risk: `OPENJSON` has no statistics (fixed estimate).
- **Prove translatability without a database:** `UseSqlServer("Server=none;Database=none;Trusted_Connection=True;")` then `query.ToQueryString()` — no connection opened. In-memory/mocked tests accept shapes EF rejects.
- **`dotnet ef migrations remove` regenerates the snapshot from the `.Designer.cs` chain.** If that chain is inconsistent (e.g. after a squash), the next migration has dozens of unrelated columns and "An operation was scaffolded that may result in the loss of data". `--force` does not help. → Delete the migration files by hand, `git checkout main -- <path>/<Ctx>ModelSnapshot.cs`, re-add, `grep "table:"` the result.
- **CS0121 "The call is ambiguous between SqlServerDbFunctionsExtensions and MySqlDbFunctionsExtensions"** with two providers referenced: `EF.Functions.DateDiff*`, also `IndexBuilder.IncludeProperties` (vs Npgsql). → Compute in memory on projected columns, or call the static class: `SqlServerIndexBuilderExtensions.IncludeProperties(idx, e => new { ... })`.

## SQL Server / Azure SQL

- **Correlated `EXISTS` under `OR` never becomes a semi-join** (measured 31× logical reads). → Uncorrelated `Select(x => x.Id)` per branch, `Union`, one `ids.Contains(o.Id)`. `NOT EXISTS` belongs under `AND`. Bad shape in `ToQueryString()`: `SELECT MAX(` wrapped in `COALESCE(` in the `WHERE`; good shape shows only `GROUP BY … HAVING MAX(`.
- **Msg 1934 "…incorrect settings: 'QUOTED_IDENTIFIER'"** writing via `sqlcmd` to a table with a filtered index: sqlcmd defaults it OFF. → `-I`, or `SET QUOTED_IDENTIFIER ON; SET ANSI_NULLS ON;` at the top.
- **Msg 102 "Incorrect syntax near '?'"** on a sane line: sqlcmd reads files as ANSI. → `-f 65001`; ASCII identifiers and `PRINT` literals.
- **Msg 468 collation conflict** with a table variable (tempdb collation); `COLLATE DATABASE_DEFAULT` fails when column collations are mixed locally. → Explicit `COLLATE` on **both** operands.
- **Diffing a `MERGE` outside SQL on a `_CI_AI_` key:** fold case + accents (NFD, strip diacritics, casefold), never ordinal — else a case-only change becomes insert + deactivate, and the deactivation kills the row SQL updated in place. `MERGE` keeps the old key spelling: compare keys in native collation, business columns in `_BIN2`, `EXCEPT` both ways.
- **`bcp -c` writes `0x00` for empty strings.** Normalize before diffing.
- **Msg 16202: `BULK INSERT` rejects `CODEPAGE` on Linux.** → `VARCHAR` staging with a UTF-8 collation (never `nvarchar`).
- **Msg 245 "Conversion failed when converting the nvarchar value '…' to data type int" after a deployment** changing a column type: migrations ran while the old revision still drained (endpoints and hosted `BackgroundService`s). Errors carry the old replica name. → Single revision or drain first.

## ASP.NET Core

- **Multipart action with any bound parameter:** the form is read **before** the action; "Multipart body length limit N exceeded" goes to ModelState and `[ApiController]` returns a 400 exception filters never see. → `ApiBehaviorOptions.InvalidModelStateResponseFactory`. `MultipartBodyLengthLimit` is **per section**; the total is Kestrel's (30 000 000 bytes).
- **Swashbuckle `PreSerializeFilters`: `document.Info` is shared** across requests; writing it leaks the first value. → `document.Info = new OpenApiInfo(document.Info) { ... }` (Microsoft.OpenApi 1.6, Swashbuckle 6.6.2); cache swagger.json per host + path.
- **`AuthorizationPolicy.CombineAsync` ignores `IAuthorizationRequirementData`** (net10.0); only the endpoint pipeline collects it. Tests through `CombineAsync` lie, and the attribute on a **SignalR hub method** enforces nothing (class-level is copied by `MapHub`). → Check in the hub or use a named policy.
- **A custom `IAuthorizationPolicyProvider` disables policy caching** (`AllowsCachingPolicies`): one `GetRequirements` per request instead of one per ~22 000.
- **Class + method role attributes evaluated with `All(...)` intersect:** a method attribute only narrows; adding a role there yields a silent 403. → Dedicated controller. Reflection tests must compute class ∩ method. A **deny** rule checked early against a role superset (union across tenants) rejects legitimate users.

## Testing

- **A fake supplying a claim production lacks** (user id on a `client_credentials` token) hides the real failure. → Model fakes on a decoded real token, per surface; comment every `null`.
- **Deterministic race under RCSI:** a separate connection holds an uncommitted concurrent write (plus a no-op `UPDATE` locking the endpoint's target), fire the HTTP call unawaited, assert still pending after ~3 s, commit, re-read. Run on a `COPY_ONLY` restore.
