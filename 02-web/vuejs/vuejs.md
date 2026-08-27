---
tags: [web, vue, typescript]
---

# Vue.js

Default for medium/small web apps — see [tech-stack-map](../../00-global/tech-stack-map.md). Pairs with **Vertical Slice** or a plain feature-first structure (see [architecture-principles](architecture-principles.md)) — reserved for projects that don't carry the complexity of a heavier, fully layered architecture. Equivalent alternative: [react](../react/react.md) — pick per team/project preference.

## Project structure

```
src/
  shared/              # reusable components, composables, utils with no feature-specific logic
  features/
    <feature>/
      components/
      composables/      # feature-specific logic, extracted from components (business rules live here, not in templates)
      services/          # API calls for this feature
      types/
```

- Composition API + `<script setup>` by default (not the Options API), TypeScript throughout.
- Business logic goes in composables, not inside component `<script setup>` blocks directly — keeps components thin and logic testable/reusable.

## Conventions

- File names: `PascalCase.vue` for components, `camelCase.ts` for composables/utils (`useThing.ts`).
- Props/emits explicitly typed (`defineProps<T>()`, `defineEmits<T>()`), no implicit `any`.
- Constants and enums per [coding-standards](coding-standards.md).
- **Package manager: pnpm**, never `npm`/`yarn` — see [tech-stack-map](../../00-global/tech-stack-map.md).

## Styling

- [tailwind-css](../tailwind-css.md) — avoid `<style scoped>` blocks except for the rare case Tailwind can't express.

## State management

- **Pinia** for global state. Feature-scoped stores (one store per feature/domain concern, not one giant app-wide store).

## Routing

- **Vue Router**, with auth/permission guards centralized in one place (a single guards module applied to routes via meta fields) rather than duplicated inline per route.

## Testing

- **Vitest** + **Vue Test Utils** for unit/component tests. Add Playwright for e2e when a project needs it.

## Internationalization (i18n)

- **Before starting a new Vue project (or a significant new app within one), the agent must explicitly ask whether the app needs to support multiple languages** — never assume either way.
- **If yes**: use **vue-i18n**, the de facto standard for the ecosystem. Translation keys in per-locale JSON files, lazy-loaded per feature where the project is large enough for that to matter. No literal UI strings in templates — every user-facing string goes through a translation key from the start, even for a single-language initial release.
- **If no**: UI strings stay hardcoded (the one exception to English-only in [coding-standards](coding-standards.md)), but centralized in a shared constants file (or one per feature) instead of duplicated across components, per DRY.

## Static analysis

Mandatory before considering any task done — see [coding-standards](coding-standards.md).

- `vue-tsc --noEmit` — plain `tsc` can't parse `.vue` Single File Components, so `vue-tsc` is the standard type-checker for Vue+TS projects.
- `eslint .` (with `eslint-plugin-vue`) — must pass clean.

