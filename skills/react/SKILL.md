---
name: react
description: Use when writing, reviewing or setting up tooling for React code (Vite, Next.js, React Native) — eslint-plugin-react-hooks, React Compiler, function components and hooks rules, state placement, effects, data fetching, forms, performance, accessibility and testing with Testing Library (rules R-REACT-01 to R-REACT-26). Apply together with the typescript skill.
---

# React

Applies on top of the `typescript` skill (all R-TS rules hold). Before changing anything, read `package.json`, the framework config (Vite / Next.js / Expo), lint config and existing components to reuse the project's conventions.

## Project setup (tooling to put in place)

- **R-REACT-01** — Current React major with a framework or Vite (no Create React App); TSX only.
- **R-REACT-02** — ESLint with `eslint-plugin-react-hooks` (recommended, rules as **errors**), `eslint-plugin-react` / `@eslint-react`, `eslint-plugin-jsx-a11y`; Next.js projects add `@next/eslint-plugin-next`.
- **R-REACT-03** — Enable the React Compiler when the stack supports it; then don't add manual `useMemo`/`useCallback`/`memo` unless measured.
- **R-REACT-04** — Tests: Vitest + `@testing-library/react` + `@testing-library/user-event`; MSW for HTTP; Playwright for E2E; Storybook for component catalog if the project has a design system.
- **R-REACT-05** — `<StrictMode>` enabled in development.

## Components

- **R-REACT-06** — Function components only; PascalCase names, one exported component per file; props typed with a `type XxxProps`.
- **R-REACT-07** — Components are pure: same props/state → same JSX; no side effects during render, no mutation of props or state (create new objects/arrays).
- **R-REACT-08** — Small, focused components; extract when JSX exceeds ~100 lines or mixes concerns. Prefer composition (`children`, slots as props) over configuration flags.
- **R-REACT-09** — Lists use a stable unique `key` (never the index on reorderable/mutable lists, never `Math.random()`).
- **R-REACT-10** — No component defined inside another component's body.
- **R-REACT-11** — `dangerouslySetInnerHTML` only with sanitised content (DOMPurify).

## Hooks and state

- **R-REACT-12** — Rules of Hooks: called at top level, only from components/custom hooks; custom hooks named `useXxx`.
- **R-REACT-13** — Keep state minimal: derive values during render instead of syncing state; lift state only as far as needed; colocate otherwise.
- **R-REACT-14** — `useReducer` for complex state transitions; Context for low-frequency global values (theme, auth), split by concern; a store (Zustand, Redux Toolkit…) only when Context isn't enough.
- **R-REACT-15** — **You might not need an effect**: no `useEffect` to derive data, handle events or reset state (use `key`). Effects are for synchronising with external systems, always with complete dependencies and a cleanup.
- **R-REACT-16** — Never silence `react-hooks/exhaustive-deps`; restructure the code instead.
- **R-REACT-17** — Refs (`useRef`) for DOM access and mutable values not used in rendering; don't read/write `ref.current` during render.

## Data and forms

- **R-REACT-18** — Server state through a data layer (TanStack Query, SWR, RTK Query, framework loaders, Server Components) — not hand-written `useEffect` + `fetch`. Handle loading, error and empty states (Suspense + error boundaries).
- **R-REACT-19** — Forms: React 19 actions (`useActionState`, `useFormStatus`) or React Hook Form, with schema validation (Zod) shared with the server.
- **R-REACT-20** — Next.js / RSC: Server Components by default, `'use client'` only at the leaves that need interactivity; never import server-only code (secrets, DB) into client components (`server-only` package).

## Performance and accessibility

- **R-REACT-21** — Measure with the React DevTools Profiler before optimising; lazy-load routes/heavy components (`lazy` + `Suspense`); virtualise long lists.
- **R-REACT-22** — Stable identities for context values passed to many consumers (memoised or compiler-handled).
- **R-REACT-23** — Semantic HTML first; every interactive element is keyboard-accessible; images have `alt`; form fields have labels; enforced by `jsx-a11y`.
- **R-REACT-24** — Error boundaries around independent UI regions; user-facing errors are actionable.

## Tests

- **R-REACT-25** — Test what the user sees and does: query by role/label (`getByRole`), interact with `user-event`, assert on the rendered result; no testing of internal state or implementation details; avoid `data-testid` when a role works.
- **R-REACT-26** — Custom hooks are tested with `renderHook`; async UI with `findBy*` / `waitFor`, never arbitrary timeouts.

## Checklist before commit

1. `eslint . --max-warnings 0`
2. `tsc --noEmit`
3. `vitest run --coverage`
4. Build (`vite build` / `next build`)

## Review output

Report each finding as: `file:line — [level] R-REACT-xx — problem → suggested fix`, with level `[blocking]`, `[suggestion]`, `[question]` or `[nit]`.
