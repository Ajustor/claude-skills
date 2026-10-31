---
name: svelte
description: Use when writing, reviewing or setting up tooling for Svelte 5 / SvelteKit code — svelte-check, eslint-plugin-svelte, runes ($state, $derived, $effect, $props), snippets, component design, SvelteKit load functions, form actions and server-only code, accessibility and testing (rules R-SVELTE-01 to R-SVELTE-24). Apply together with the typescript skill.
---

# Svelte

Applies on top of the `typescript` skill (all R-TS rules hold). Before changing anything, read `package.json`, `svelte.config.js`, `vite.config.*`, lint config and existing components to reuse the project's conventions.

## Project setup (tooling to put in place)

- **R-SVELTE-01** — Svelte 5 with **runes mode**; SvelteKit for applications (scaffolded with `npx sv create`). No new Svelte 4 syntax (`export let`, `$:`, `on:click`, stores for local state).
- **R-SVELTE-02** — `svelte-check --fail-on-warnings` in CI (types + a11y + compiler warnings); `<script lang="ts">` in every component.
- **R-SVELTE-03** — ESLint flat config with `eslint-plugin-svelte` + typescript-eslint; Prettier with `prettier-plugin-svelte`.
- **R-SVELTE-04** — Tests: Vitest + `@testing-library/svelte` (or `vitest-browser-svelte`); Playwright for E2E.

## Components and runes

- **R-SVELTE-05** — Props through `let { a, b = default, ...rest }: Props = $props()` with a `Props` type; two-way binding only when explicitly needed (`$bindable()`).
- **R-SVELTE-06** — `$state` for local mutable state; `$state.raw` for large data replaced wholesale; classes with `$state` fields for reusable reactive logic in `.svelte.ts` files.
- **R-SVELTE-07** — `$derived` / `$derived.by` for every computed value — never `$effect` to sync one state to another.
- **R-SVELTE-08** — `$effect` only for side effects with external systems (DOM APIs, subscriptions, analytics); return a cleanup function; no state writes inside unless unavoidable (justify).
- **R-SVELTE-09** — Events as callback props (`onclick`, `onchange={...}`), not `createEventDispatcher`.
- **R-SVELTE-10** — Content composition with snippets (`{#snippet}` / `{@render}`, typed `Snippet<[T]>`), not slots.
- **R-SVELTE-11** — Keyed each blocks `{#each items as item (item.id)}` on any list that changes.
- **R-SVELTE-12** — Component files PascalCase; keep components small, extract logic into `.svelte.ts` modules.
- **R-SVELTE-13** — Shared state through context (`setContext`/`getContext` with typed keys or `createContext`) rather than module-level `$state` on the server (shared between requests → data leak).
- **R-SVELTE-14** — `{@html}` only with sanitised content.
- **R-SVELTE-15** — Styles are scoped by default; `:global` only in a few justified places.

## SvelteKit

- **R-SVELTE-16** — Data loading in `load` functions (`+page.ts` / `+page.server.ts`), typed via the generated `./$types` (`PageLoad`, `PageServerLoad`, `PageProps`).
- **R-SVELTE-17** — Secrets and DB access only in server modules (`+page.server.ts`, `+server.ts`, `$lib/server/`) and `$env/static/private` / `$env/dynamic/private`; never in universal code.
- **R-SVELTE-18** — Mutations through form actions with progressive enhancement (`use:enhance`) and schema validation (Zod/Valibot, e.g. Superforms); return `fail(400, ...)` for validation errors.
- **R-SVELTE-19** — Expected errors with `error(status, message)`, redirects with `redirect()`; `+error.svelte` pages; `handleError` hook for logging unexpected errors.
- **R-SVELTE-20** — Auth/authorization in `hooks.server.ts` and re-checked in each server `load`/action that needs it (layouts alone don't protect actions).
- **R-SVELTE-21** — Choose rendering per route explicitly (`prerender`, `ssr`, `csr`) and the right adapter.

## Accessibility and tests

- **R-SVELTE-22** — Zero a11y compiler warnings: semantic elements, labels, keyboard handlers with click handlers, `alt` on images.
- **R-SVELTE-23** — Component tests through rendered output and user interactions; logic in `.svelte.ts` tested directly (wrap in `$effect.root` when it uses effects).
- **R-SVELTE-24** — `load` functions and form actions have unit tests; critical flows have Playwright tests.

## Checklist before commit

1. `prettier --check . && eslint . --max-warnings 0`
2. `svelte-kit sync && svelte-check --fail-on-warnings`
3. `vitest run --coverage`
4. `vite build`

## Review output

Report each finding as: `file:line — [level] R-SVELTE-xx — problem → suggested fix`, with level `[blocking]`, `[suggestion]`, `[question]` or `[nit]`.
