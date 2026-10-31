---
name: lua
description: Use when writing, reviewing or setting up tooling for Lua code (LÖVE, LuaJIT, ComputerCraft/CC:Tweaked, Neovim, embedded scripting) — luacheck or selene, StyLua, lua-language-server annotations, locals and globals, modules, OOP with metatables, error handling, performance and testing with busted (rules R-LUA-01 to R-LUA-24).
---

# Lua

Language-specific complement to `clean-code`, `clean-architecture` and `quality-verification`. First identify the **runtime** (Lua 5.1/5.4, LuaJIT, LÖVE, CC:Tweaked, Neovim…) — it defines the standard library, available globals and performance model. Read `.luacheckrc` / `selene.toml`, `stylua.toml`, `.luarc.json` and CI to reuse the project's choices.

## Project setup (tooling to put in place)

- **R-LUA-01** — Static analysis with **luacheck** (`.luacheckrc`, `std` set to the runtime, e.g. `luajit`, plus the host globals like `love`) or **selene** (`selene.toml`); runs in CI with zero warnings. Muted categories are listed with the reason.
- **R-LUA-02** — Formatting with **StyLua** (`stylua.toml`), `stylua --check .` in pre-commit and CI.
- **R-LUA-03** — **lua-language-server** configured (`.luarc.json`: runtime version, globals, library paths) and type annotations (`---@param`, `---@return`, `---@class`, `---@type`) on public functions and classes; `lua-language-server --check` in CI when practical.
- **R-LUA-04** — Tests with **busted** (or the runtime's harness, e.g. headless LÖVE); coverage with luacov on core logic.
- **R-LUA-05** — Dependencies through LuaRocks with a rockspec (or vendored with version noted) — no unversioned copy-pasted libraries.

## Language practices

### Scope and globals
- **R-LUA-06** — Everything is `local`: variables, functions, module tables. Never create accidental globals; host globals are whitelisted in the linter config.
- **R-LUA-07** — Cache frequently used globals/library functions as locals in hot modules (`local floor = math.floor`), only where it matters.
- **R-LUA-08** — No shadowing of outer locals with the same name in nested scopes (luacheck W4xx).

### Modules
- **R-LUA-09** — One module per file, returning a single table: `local M = {} … return M`; no side effects at `require` time beyond defining things.
- **R-LUA-10** — Private helpers are `local function`s not exported; `require` at the top of the file; no circular requires.

### Tables and OOP
- **R-LUA-11** — Classes through metatables with a consistent pattern (`Class.__index = Class`, `Class.new(...)` returning `setmetatable(o, Class)`); method calls with `:`; annotate with `---@class`.
- **R-LUA-12** — Remember arrays are 1-based; `#t` is only defined for sequences without holes — track counts explicitly otherwise.
- **R-LUA-13** — `ipairs` for sequences, `pairs` for maps; never rely on `pairs` order; don't add keys to a table while iterating it with `pairs`.
- **R-LUA-14** — Don't use `nil` as a meaningful value inside tables; use a sentinel or explicit field.

### Errors
- **R-LUA-15** — Expected failures return `nil, err_message` (the Lua convention); programmer errors use `error("message", level)` or `assert(cond, "message")` at function entry for argument validation.
- **R-LUA-16** — Wrap calls that may fail at system boundaries (file I/O, user scripts, mods, network) in `pcall`/`xpcall` with `debug.traceback`; never swallow the error — log or propagate it.
- **R-LUA-17** — Validate external data (save files, config, mod data, network messages) before use.

### Style and naming
- **R-LUA-18** — snake_case for variables/functions (or the project's existing convention), PascalCase for classes, UPPER_CASE for constants; `_` for intentionally unused values.
- **R-LUA-19** — Use `==`/`~=` with care on `nil` and `false` (both falsy); `x = x or default` only when `false` is not a valid value.
- **R-LUA-20** — String building in loops with `table.concat`, not repeated `..`.

### Performance (games, real-time)
- **R-LUA-21** — No table/closure allocation in per-frame hot paths (`update`, `draw`); reuse tables and pools; measure with a profiler before optimising.
- **R-LUA-22** — LuaJIT: avoid NYI (non-compiled) functions in hot loops; use FFI only behind a small, documented module.
- **R-LUA-23** — Frame logic is time-based (`dt`), never frame-count-based.

### Tests
- **R-LUA-24** — Game/engine logic is separated from the host API (`love.*`, `turtle.*`, `vim.*`) so it can be unit-tested without the runtime; the host API is injected or stubbed in tests.

## Checklist before commit

1. `stylua --check .`
2. `luacheck .` (or `selene .`)
3. `busted` (or the project's test runner)

## Review output

Report each finding as: `file:line — [level] R-LUA-xx — problem → suggested fix`, with level `[blocking]`, `[suggestion]`, `[question]` or `[nit]`.
