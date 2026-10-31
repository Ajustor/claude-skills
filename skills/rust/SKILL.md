---
name: rust
description: Use when writing, reviewing or setting up tooling for Rust code — workspace and Cargo.toml lints, rustfmt, clippy, cargo-deny/audit, error handling (thiserror/anyhow), ownership and API design, unsafe, async, testing, benchmarks and CI (rules R-RS-01 to R-RS-30).
---

# Rust

Language-specific complement to `clean-code`, `clean-architecture` and `quality-verification`. Before changing anything, read the root `Cargo.toml`, `rust-toolchain.toml`, `rustfmt.toml`, `clippy.toml`, `deny.toml` and CI to reuse the project's choices.

## Project setup (tooling to put in place)

- **R-RS-01** — Use the latest stable edition; declare `edition` and `rust-version` (MSRV) once in `[workspace.package]` and inherit them (`edition.workspace = true`). Pin the toolchain in `rust-toolchain.toml` (with `rustfmt`, `clippy` components).
- **R-RS-02** — Multi-crate projects use a **workspace**; shared dependency versions in `[workspace.dependencies]`; `Cargo.lock` committed for binaries/apps.
- **R-RS-03** — Lints declared in Cargo, inherited by every crate (`[lints] workspace = true`):
  ```toml
  [workspace.lints.rust]
  unsafe_code = "forbid"          # or "deny" + justified #[allow] where unavoidable
  missing_debug_implementations = "warn"
  unused_must_use = "deny"

  [workspace.lints.clippy]
  all = { level = "deny", priority = -1 }
  pedantic = { level = "warn", priority = -1 }
  unwrap_used = "deny"
  expect_used = "warn"
  dbg_macro = "deny"
  todo = "deny"
  print_stdout = "warn"
  ```
- **R-RS-04** — `cargo fmt --all -- --check` and `cargo clippy --workspace --all-targets --all-features -- -D warnings` run in a pre-commit hook and in CI.
- **R-RS-05** — Supply chain: `cargo deny check` (advisories, licenses, bans, sources) in CI, `cargo audit` or Dependabot/Renovate; `cargo machete`/`cargo udeps` periodically for unused deps.
- **R-RS-06** — Tests with `cargo nextest` (or `cargo test`), coverage with `cargo llvm-cov` and a blocking threshold; `cargo test --doc` runs too (nextest skips doctests).
- **R-RS-07** — Benchmarks on critical paths with `criterion` (or `divan`); `cargo hack --feature-powerset check` for crates with features; check MSRV in CI.
- **R-RS-08** — Release profile tuned explicitly where it matters (`lto`, `codegen-units = 1`, `panic`), not by default copy-paste.

## Language practices

### Errors
- **R-RS-09** — No `unwrap()` / `expect()` in non-test code paths that can fail at runtime; `expect("reason")` only for invariants, with the invariant in the message.
- **R-RS-10** — Libraries define typed errors (`thiserror`, one enum per module/crate boundary); applications may use `anyhow`/`eyre` with `.context("what was being done")`.
- **R-RS-11** — Propagate with `?`; never discard a `Result` (`let _ = ...` requires a comment).
- **R-RS-12** — `panic!` only for programmer bugs, never for input or I/O errors.

### Types and API design
- **R-RS-13** — Make illegal states unrepresentable: enums over boolean flags, newtypes over raw primitives (`UserId(u64)`), builder or typestate for complex construction.
- **R-RS-14** — Accept the most general borrowed type (`&str`, `&[T]`, `impl AsRef<Path>`, `impl IntoIterator`); return owned types.
- **R-RS-15** — Derive the common traits (`Debug`, `Clone`, `PartialEq`, `Eq`, `Hash`, `Default`) when meaningful; implement `Display` for user-facing types; `From`/`TryFrom` for conversions.
- **R-RS-16** — `#[must_use]` on functions whose result must not be ignored; `#[non_exhaustive]` on public enums/structs that may grow.
- **R-RS-17** — Minimal visibility: `pub(crate)` by default, `pub` only for the real API.
- **R-RS-18** — Follow the Rust API Guidelines naming: `as_`/`to_`/`into_` conversions, `iter`/`iter_mut`/`into_iter`, `new`/`with_*`, no `get_` prefix on getters.

### Ownership and performance
- **R-RS-19** — No `.clone()` to silence the borrow checker without thinking; prefer borrowing, `Cow`, `Arc` for shared ownership.
- **R-RS-20** — Prefer iterators over index loops; `Vec::with_capacity` when the size is known; avoid allocating in hot loops.
- **R-RS-21** — Interior mutability (`RefCell`, `Mutex`) only when ownership can't express it; never hold a `std::sync::Mutex` guard across `.await`.

### Unsafe
- **R-RS-22** — `unsafe` is isolated in small modules behind a safe API; every `unsafe` block has a `// SAFETY:` comment stating the invariants (enforce `clippy::undocumented_unsafe_blocks`). Run `cargo miri test` on that code.

### Async
- **R-RS-23** — One runtime per application (usually tokio); no blocking calls inside async (`spawn_blocking` for CPU/blocking I/O).
- **R-RS-24** — Spawned tasks are tracked (`JoinSet`, handles) and cancellation-safe; timeouts on all network calls.

### Logging and docs
- **R-RS-25** — Use `tracing` (structured fields, spans) instead of `println!`/`eprintln!` in libraries and services.
- **R-RS-26** — Every public item has a `///` doc comment; fallible functions document `# Errors`, panicking ones `# Panics`, unsafe ones `# Safety`. Doc examples compile (doctests).

### Modules and style
- **R-RS-27** — snake_case for functions/modules/variables, PascalCase for types/traits, SCREAMING_SNAKE_CASE for consts/statics.
- **R-RS-28** — Modules by feature/domain, not by kind; `mod.rs`-less layout (`foo.rs` + `foo/`) consistently.
- **R-RS-29** — No wildcard imports except preludes and `use super::*` in tests.

### Tests
- **R-RS-30** — Unit tests in `#[cfg(test)] mod tests` next to the code; integration tests in `tests/` against the public API; property tests (`proptest`) for parsers/serialization; snapshot tests (`insta`) for complex outputs.

## Checklist before commit

1. `cargo fmt --all -- --check`
2. `cargo clippy --workspace --all-targets --all-features -- -D warnings`
3. `cargo nextest run --workspace` (+ `cargo test --doc`)
4. `cargo deny check`

## Review output

Report each finding as: `file:line — [level] R-RS-xx — problem → suggested fix`, with level `[blocking]`, `[suggestion]`, `[question]` or `[nit]`.
