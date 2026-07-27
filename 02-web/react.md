---
tags: [web, react, typescript, vite]
---

# React

Alternative default to [[vuejs]] for medium/small web apps — see [[tech-stack-map]]. Pick whichever of the two matches the team/project's existing preference; neither is a "smaller" or "bigger" choice than the other, same reasoning as [[aspnet-core]] vs [[spring-boot]] on the backend.

## Scaffolding

- **Vite**, always — `pnpm create vite@latest` to scaffold, **TypeScript** template every time, same as [[angular]]. Never Create React App (unmaintained) or a custom webpack setup.

## Project structure

Pairs with **Vertical Slice** or a plain feature-first structure (see [[architecture-principles]]) — same tier as [[vuejs]], not Angular-level layering.

```
src/
  shared/              # reusable components, hooks, utils with no feature-specific logic
  features/
    <feature>/
      components/
      hooks/            # feature-specific logic extracted from components (business rules live here, not in JSX)
      services/          # API calls for this feature
      types/
```

- Business logic goes in custom hooks (`useThing.ts`), not inline in component bodies — keeps components thin and logic testable/reusable, same principle as Vue's composables.
- Function components only, no class components.

## Conventions

- File names: `PascalCase.tsx` for components, `camelCase.ts` for hooks/utils/services (`useThing.ts`).
- Props typed via an explicit `interface`/`type`, no implicit `any`. No `React.FC` (its implicit `children` typing is imprecise) — type props directly on the function signature.
- Constants and enums per [[coding-standards]].
- **Package manager: pnpm**, never `npm`/`yarn` — see [[tech-stack-map]].

## Styling

- [[tailwind-css]] — no separate CSS Modules/styled-components unless Tailwind genuinely can't express something (rare).

## State management

- **Zustand** for global state — feature-scoped stores (one store per feature/domain concern), same reasoning as Vue's Pinia and Angular's signals-only rule: avoid Redux-level ceremony unless a specific feature's state is genuinely Redux-shaped (many interdependent actions, need for time-travel debugging), which is a deliberate, documented exception, not the default.
- Local component state (`useState`/`useReducer`) for anything that doesn't need to be shared across features — don't reach for the global store by default.

## Routing

- **React Router**, with auth/permission guards centralized in one place (a loader/wrapper applied at the route level) rather than duplicated inline per component — same principle as Vue Router's guards.

## Forms

- **React Hook Form** + **Zod** (via `@hookform/resolvers/zod`) for schema-based validation — keeps validation rules declarative and decoupled from the component's render logic, same spirit as [[vuejs]]'s reusable `ValidatorFn`s.

## Testing

- **Vitest** + **React Testing Library** for unit/component tests. Add Playwright for e2e when a project needs it — same stack shape as [[vuejs]].

## Internationalization (i18n)

- **Before starting a new React project (or a significant new app within one), the agent must explicitly ask whether the app needs to support multiple languages.** Same rule as [[vuejs]], [[angular]], [[flutter]] — never assume either way.
- **If yes**: use **react-i18next**, the de facto standard for the ecosystem. Translation keys in per-locale JSON files, lazy-loaded per feature where the project is large enough for that to matter. No literal UI strings in JSX — every user-facing string goes through a translation key from the start, even for a single-language initial release.
- **If no**: UI strings stay hardcoded (the one exception to English-only in [[coding-standards]]), but centralized in a shared constants file (or one per feature) instead of duplicated across components, per DRY.

## Static analysis

Mandatory before considering any task done — see [[coding-standards]].

- `tsc --noEmit` — type-checking. Unlike Vue's SFCs, plain `tsc` parses `.tsx` directly, no wrapper tool needed.
- `eslint .` (with `eslint-plugin-react-hooks`) — must pass clean.

## See also

- [[coding-standards]], [[architecture-principles]], [[tailwind-css]], [[responsive-design]]
- [[vuejs]] (equivalent frontend, pick by team preference)
- [[supabase]], [[aspnet-core]], [[spring-boot]] (typical backend pairings)
