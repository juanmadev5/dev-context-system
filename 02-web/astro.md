---
tags: [web, astro]
---

# Astro

Default for medium/large landing pages that need some interactivity but are fundamentally content-first — see [[tech-stack-map]]. Use [[html-tailwind]] instead when the page has little to no functionality.

## Project structure

```
src/
  components/         # .astro components, mostly static
  layouts/
  pages/              # file-based routing
  islands/            # interactive components (framework-backed: e.g. a Vue/React/Svelte island), hydrated selectively
```

- No architecture pattern beyond this — see [[architecture-principles]]. Astro projects are content/presentation-heavy; don't introduce Clean Architecture layering here, it's the wrong tool.
- Ship zero JS by default. Reach for an island (`client:load` / `client:visible` / `client:idle`) only for the specific interactive widget that needs it — never hydrate a whole page.

## Conventions

- File names: `kebab-case`.
- Prefer Astro's native templating for anything static; only drop into a UI framework island when real client-side interactivity/state is required.
- Constants and enums per [[coding-standards]] still apply to any TS/JS logic in the project (e.g. shared content config, island logic).
- **Package manager: pnpm**, never `npm`/`yarn` — see [[tech-stack-map]].

## Styling

- [[tailwind-css]].

## Static analysis

Mandatory before considering any task done — see [[coding-standards]].

- `astro check` — the official diagnostic command; validates `.astro` files and embedded TS/JS. Must pass clean.

## See also

- [[coding-standards]], [[tailwind-css]], [[responsive-design]]
- [[html-tailwind]]
