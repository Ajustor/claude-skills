# Development rules skills

Agent skills derived from the team's *Development Rules* document, one per work domain. Each folder under `skills/` holds a `SKILL.md` following the [Agent Skills](https://agentskills.io) format, deployable with [pollen](https://github.com/groupbees/pollen).

| Skill | Domain | Rules |
|---|---|---|
| [clean-architecture](skills/clean-architecture/SKILL.md) | Layers, dependency rule, SOLID, folder structure | R-ARCH-01..06 |
| [design-patterns](skills/design-patterns/SKILL.md) | When and which pattern, anti-patterns, YAGNI | R-PAT-01..04 |
| [clean-code](skills/clean-code/SKILL.md) | Naming, functions, errors, tests | R-CC-01..24 |
| [performance](skills/performance/SKILL.md) | Measurement, DB, caching, network, front-end | R-PERF-01..15 |
| [code-review](skills/code-review/SKILL.md) | PR process, review procedure, comment style, PR checklist | R-REV-01..07 |
| [quality-verification](skills/quality-verification/SKILL.md) | CI/hooks tooling, periodic audits, technical debt | R-VER-01..10 |

### Language skills

Per-language practices and the tooling to set up (lint, format, type-check, tests, CI, security). They complement the generic skills above; framework skills apply on top of `typescript`.

| Skill | Scope | Rules |
|---|---|---|
| [csharp](skills/csharp/SKILL.md) | C# / .NET: build props, analyzers, nullability, async, DI, xUnit | R-CS-01..30 |
| [rust](skills/rust/SKILL.md) | Workspace lints, clippy, cargo-deny, errors, API design, unsafe, async | R-RS-01..30 |
| [typescript](skills/typescript/SKILL.md) | TS/JS: strict tsconfig, ESLint/Prettier, typing, runtime validation, async | R-TS-01..30 |
| [vue](skills/vue/SKILL.md) | Vue 3 / Nuxt: script setup, reactivity, composables, Pinia | R-VUE-01..26 |
| [react](skills/react/SKILL.md) | React / Next.js: hooks rules, state, effects, data fetching, Testing Library | R-REACT-01..26 |
| [svelte](skills/svelte/SKILL.md) | Svelte 5 runes / SvelteKit: load, form actions, server-only code | R-SVELTE-01..24 |
| [lua](skills/lua/SKILL.md) | Lua / LuaJIT / LÖVE / CC:Tweaked: luacheck, StyLua, LuaLS, busted | R-LUA-01..24 |
| [gdscript](skills/gdscript/SKILL.md) | Godot 4: static typing, gdtoolkit, GUT/gdUnit4, scenes and signals | R-GD-01..28 |

Rule IDs are kept identical to the source document so findings can be traced back to it. When the rules document changes, update the matching skill.

## Install with pollen

In a project's `pollen.yaml` (or the user-level one, `pollen -g`):

```yaml
repos:
  - repo: <git URL of this repository>
    revision: <full commit SHA>  # vX.Y.Z
    paths:
      - path: skills
```

Then run `pollen update`. To pick only some domains, add `exclude: '^(performance|quality-verification)$'` under the path entry.

Check the skills locally before releasing:

```sh
pollen validate   # with a pollen.yaml using `repo: local` and `path: skills`
```
