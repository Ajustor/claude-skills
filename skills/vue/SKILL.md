---
name: vue
description: Use when writing, reviewing or setting up tooling for Vue 3 / Nuxt code — vue-tsc, eslint-plugin-vue, Composition API with script setup, props/emits typing, reactivity pitfalls, composables, Pinia stores, component design, performance, accessibility and testing with Vitest + Vue Test Utils (rules R-VUE-01 to R-VUE-26). Apply together with the typescript skill.
---

# Vue

Applies on top of the `typescript` skill (all R-TS rules hold). Before changing anything, read `package.json`, `vite.config.*` / `nuxt.config.*`, lint config and existing components to reuse the project's conventions.

## Project setup (tooling to put in place)

- **R-VUE-01** — Vue 3 with Vite (or Nuxt); no new Vue 2 / Options API code.
- **R-VUE-02** — Type-checking with `vue-tsc --noEmit` (or `nuxi typecheck`) in CI; `@vue/tsconfig` as base.
- **R-VUE-03** — ESLint flat config with `eslint-plugin-vue` (`flat/recommended`) + `@vue/eslint-config-typescript` (type-checked) + `@vue/eslint-config-prettier`.
- **R-VUE-04** — Tests: Vitest + `@vue/test-utils` (or `@testing-library/vue`), `happy-dom`/`jsdom`; Playwright for E2E. Vue DevTools for debugging.
- **R-VUE-05** — State: Pinia (never Vuex for new code); routing: Vue Router with typed routes (`unplugin-vue-router`) when available.

## Components

- **R-VUE-06** — Single-File Components with `<script setup lang="ts">`; block order `<script>`, `<template>`, `<style>`.
- **R-VUE-07** — Component names are multi-word PascalCase (`UserCard.vue`), base components prefixed (`BaseButton`), one component per file.
- **R-VUE-08** — Props typed with `defineProps<{...}>()` and defaults through reactive props destructure (Vue ≥ 3.5) or `withDefaults`; emits typed with `defineEmits<{ change: [id: string] }>()`; `v-model` via `defineModel()`.
- **R-VUE-09** — Props are never mutated; data flows down through props, up through events (or `v-model`).
- **R-VUE-10** — Small components: if a template exceeds ~100 lines or mixes concerns, extract sub-components or composables. Logic out of templates: no complex expressions, use `computed`.
- **R-VUE-11** — `v-for` always has a stable unique `:key` (never the index on mutable lists); never `v-if` and `v-for` on the same element.
- **R-VUE-12** — Styles are `scoped` (or CSS modules); no global style leaks from components.
- **R-VUE-13** — `v-html` only with sanitised content (DOMPurify) — XSS risk.

## Reactivity

- **R-VUE-14** — `ref()` by default; `reactive()` only for grouped state you never reassign or destructure; use `toRefs`/`storeToRefs` to destructure.
- **R-VUE-15** — `computed` for derived state (pure, no side effects); `watch`/`watchEffect` only for side effects, with explicit sources and cleanup (`onWatcherCleanup`).
- **R-VUE-16** — `shallowRef` / `markRaw` for large immutable data or third-party instances (maps, charts).
- **R-VUE-17** — Side effects registered in lifecycle hooks are cleaned up in `onUnmounted` (listeners, timers, subscriptions).

## Composables and state

- **R-VUE-18** — Reusable stateful logic lives in composables named `useXxx` in `composables/`; they accept `MaybeRefOrGetter` inputs (`toValue`) and return refs.
- **R-VUE-19** — `provide`/`inject` with typed `InjectionKey<T>`; no prop drilling beyond 2 levels.
- **R-VUE-20** — Pinia stores are setup stores, one per domain; components don't call APIs directly — they go through stores/composables/services.
- **R-VUE-21** — Server state (fetching, caching) through a dedicated layer (TanStack Query for Vue, Nuxt `useFetch`/`useAsyncData`) with loading and error states handled.

## Performance and accessibility

- **R-VUE-22** — Lazy-load routes and heavy components (`defineAsyncComponent`, dynamic imports); `v-memo`/`v-once` only after measuring; virtualise long lists.
- **R-VUE-23** — Semantic HTML, labels on form controls, keyboard navigation and focus management; `eslint-plugin-vuejs-accessibility`.
- **R-VUE-24** — Texts go through i18n (`vue-i18n`) when the app is translated; no hard-coded user-facing strings.

## Tests

- **R-VUE-25** — Test components through their public contract (props, emitted events, rendered output, user interactions), not internal state.
- **R-VUE-26** — Composables and stores have unit tests (`createTestingPinia`, `setActivePinia(createPinia())`).

## Checklist before commit

1. `eslint . --max-warnings 0`
2. `vue-tsc --noEmit`
3. `vitest run --coverage`
4. Build (`vite build` / `nuxi build`)

## Review output

Report each finding as: `file:line — [level] R-VUE-xx — problem → suggested fix`, with level `[blocking]`, `[suggestion]`, `[question]` or `[nit]`.
