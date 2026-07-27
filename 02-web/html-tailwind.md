---
tags: [web, html]
---

# Plain HTML (+ Tailwind)

Default for simple landing pages that don't need meaningful functionality — see [[tech-stack-map]]. Use [[astro]] instead once the page grows past a single static page or needs any real interactivity/content structure.

## Structure

- Flat, simple: `index.html` (+ additional pages as plain `.html` files), a `css/` output from Tailwind's build, minimal `js/` only if strictly needed (e.g. a mobile nav toggle).
- No framework, no build-step bundler beyond Tailwind's CLI/build.
- No architecture pattern applies here — see [[architecture-principles]]. If the project needs enough JS logic to warrant one, it has outgrown plain HTML; move to [[astro]], [[vuejs]], or [[react]].

## Conventions

- Semantic HTML elements over generic `<div>` soup where a semantic tag exists (`<nav>`, `<header>`, `<main>`, `<section>`, `<footer>`).
- Any inline JS still follows [[coding-standards]] (no magic strings, meaningful naming) even in a small script.
- **Package manager: pnpm**, never `npm`/`yarn`, for the Tailwind CLI/build tooling — see [[tech-stack-map]].

## Styling

- [[tailwind-css]] — no other CSS unless Tailwind truly can't express something.

## See also

- [[coding-standards]], [[responsive-design]]
- [[astro]], [[tailwind-css]]
