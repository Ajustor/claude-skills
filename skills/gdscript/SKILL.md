---
name: gdscript
description: Use when writing, reviewing or setting up tooling for Godot 4 projects in GDScript — static typing and warnings as errors, gdtoolkit (gdformat/gdlint), GUT or gdUnit4 tests, headless CI exports, scene and node architecture, signals, resources, autoloads, the official style guide and performance (rules R-GD-01 to R-GD-28).
---

# GDScript (Godot 4)

Language-specific complement to `clean-code`, `clean-architecture` and `quality-verification`. Before changing anything, read `project.godot` (Godot version, autoloads, debug/gdscript warnings), `export_presets.cfg`, addons and CI to reuse the project's choices.

## Project setup (tooling to put in place)

- **R-GD-01** — Godot 4 stable, version pinned for the whole team and CI (same editor version as in `project.godot` `config/features`).
- **R-GD-02** — In *Project Settings → Debug → GDScript*: `untyped_declaration`, `inferred_declaration` (optional), `unsafe_*` warnings set to **Warn** or **Error**; treat warnings as errors for new code. Every `@warning_ignore` has a comment explaining why.
- **R-GD-03** — **gdtoolkit**: `gdformat --check .` and `gdlint .` (with a `gdlintrc` committed) in a pre-commit hook and in CI.
- **R-GD-04** — Tests with **GUT** or **gdUnit4**, run headless in CI (`godot --headless -s addons/gut/gut_cmdln.gd -gexit` or the gdUnit4 CLI).
- **R-GD-05** — CI also runs a headless import/check (`godot --headless --import`, then the test run) and exports builds (`godot --headless --export-release <preset> <path>`), e.g. with a pinned Godot container image.
- **R-GD-06** — `.gitignore` excludes `.godot/` and export outputs; `.gitattributes` normalises line endings and marks binary assets (LFS for large ones). Commit `.import` files' sources, not the cache.

## Typing and language

- **R-GD-07** — Static typing everywhere: typed variables (`var speed: float = 10.0` or `:=`), typed parameters and return types (`-> void`), typed arrays and dictionaries (`Array[Enemy]`, `Dictionary[String, int]`).
- **R-GD-08** — `class_name` for reusable scripts so they are usable as types; no string-based `has_method`/`call` duck typing when a type or interface-like base class works.
- **R-GD-09** — `@export` for designer-tunable values with hints (`@export_range`, `@export_enum`, `@export_group`); no magic numbers in logic.
- **R-GD-10** — Node references via `@onready var sprite: Sprite2D = $Sprite2D` or `%UniqueName`, cached once — never `get_node()` in `_process`.
- **R-GD-11** — `enum` for states; `match` with an explicit default branch.
- **R-GD-12** — Check `is_instance_valid()` before using nodes that may have been freed; free with `queue_free()`.

## Architecture

- **R-GD-13** — **Call down, signal up**: parents call methods on children; children emit signals to parents. Siblings never reach each other via `get_parent().get_node(...)` path chains.
- **R-GD-14** — Signals are typed (`signal health_changed(new_value: int)`), connected in code with callables (`button.pressed.connect(_on_pressed)`), named in past tense.
- **R-GD-15** — Scenes are self-contained and reusable: they work when instanced alone (dependencies exported or injected), one responsibility per scene; composition over deep inheritance (component nodes).
- **R-GD-16** — Data in custom `Resource` classes (`.tres`) rather than hard-coded dictionaries; duplicate resources (`duplicate()` / `resource_local_to_scene`) before mutating shared ones.
- **R-GD-17** — Autoloads only for truly global services (event bus, save system, audio); no game logic dumped in a singleton; access through typed names.
- **R-GD-18** — Game logic separated from presentation where testable (plain `RefCounted` classes for rules, nodes for display/input).
- **R-GD-19** — Input through the Input Map actions (`Input.is_action_pressed("jump")`), never hard-coded keys.

## Style (official GDScript style guide)

- **R-GD-20** — snake_case for files, functions and variables; PascalCase for `class_name` and nodes; CONSTANT_CASE for constants and enum members; private members prefixed `_`; signal handlers `_on_<node>_<signal>`.
- **R-GD-21** — Script member order: `@tool`, `class_name`, `extends`, doc comment, signals, enums, constants, `@export` vars, public vars, private vars, `@onready` vars, `_init`, `_ready`, other built-in virtuals, public methods, private methods.
- **R-GD-22** — Doc comments (`##`) on public classes, methods and exported properties.

## Performance

- **R-GD-23** — Use `_physics_process` for physics/movement, `_process` for visuals; disable processing (`set_process(false)`) on idle nodes; prefer signals/timers to polling.
- **R-GD-24** — Movement and timing scale with `delta`.
- **R-GD-25** — Avoid allocations and node creation in per-frame code; pool frequently spawned objects; preload (`preload`) known assets, `ResourceLoader.load_threaded_request` for large ones.
- **R-GD-26** — Profile with the built-in profiler and monitors before optimising; move proven hot spots to C# or GDExtension only after measuring.

## Tests and saves

- **R-GD-27** — Gameplay rules, state machines and save/load are unit-tested; tests don't depend on frame timing (simulate with explicit `delta`).
- **R-GD-28** — Save data is versioned and validated on load; use `FileAccess`/`ConfigFile`/JSON, never `var_to_str`/`bytes_to_var` with objects on untrusted files (code execution risk).

## Checklist before commit

1. `gdformat --check .`
2. `gdlint .`
3. Headless test run (GUT / gdUnit4) with no new GDScript warnings
4. Export of the main preset succeeds (CI)

## Review output

Report each finding as: `file:line — [level] R-GD-xx — problem → suggested fix`, with level `[blocking]`, `[suggestion]`, `[question]` or `[nit]`.
