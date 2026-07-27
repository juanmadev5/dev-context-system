---
tags: [global, frontend, responsive-design]
---

# Responsive Design

Applies to **every** frontend surface — web ([[angular]], [[vuejs]], [[react]], [[astro]], [[html-tailwind]]) and mobile ([[flutter]], [[jetpack-compose]]) — see [[tech-stack-map]].

## Principle

- Never design or implement a screen/component assuming one fixed viewport or device size. The realistic range of screen sizes for the target platform is in scope **by default** — unlike i18n (see each stack note's "Internationalization" section), this isn't a per-project question to ask, it's a standing requirement for any UI work.
- "It looks right at the one size I happened to test" doesn't count as done. Before considering a UI task finished, verify it against the platform's real size range, not a single resolution.

## Web

- **Mobile-first**: base (unprefixed) Tailwind utilities target the smallest supported viewport; larger viewports are layered on top with `sm:`/`md:`/`lg:`/`xl:`/`2xl:` overrides — never the reverse (build for desktop, then cram it into mobile with overrides).
- Minimum verification set: **~375px** (mobile), **~768px** (tablet), **~1280px+** (desktop). A layout that only works at one of these isn't finished.
- Layouts use `flex`/`grid` with relative sizing (`%`, `fr`, `max-w-*`, `min-w-0`) — not fixed pixel widths/heights on containers. Media (`img`, `video`) is always constrained (`max-w-full h-auto` or an aspect-ratio utility) so it can't blow out a narrow viewport.
- See [[tailwind-css]] for the utility-level mechanics (breakpoint prefixes, tokens) that implement this.

## Mobile

- **Flutter**: use `LayoutBuilder` / `MediaQuery` (or a breakpoints abstraction built on top of them) to adapt layout between phone and tablet form factors and to handle orientation changes — never hardcode pixel/logical-pixel dimensions that assume one specific device. See [[flutter]].
- **Jetpack Compose**: use Material 3's `WindowSizeClass` (adaptive layouts) and `BoxWithConstraints` for the same purpose across Android's phone/tablet/foldable range. See [[jetpack-compose]].
- Both: respect safe areas/system insets, and don't assume portrait-only unless the project has explicitly scoped out landscape.

## See also

- [[tailwind-css]], [[flutter]], [[jetpack-compose]]
- [[tech-stack-map]]
