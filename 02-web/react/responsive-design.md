---
tags: [react, responsive-design]
---

# Responsive Design — React

## Principle

- Never design or implement a screen/component assuming one fixed viewport or device size. The realistic range of screen sizes for the target platform is in scope **by default** — unlike i18n (see [react.md](react.md)'s "Internationalization" section), this isn't a per-project question to ask, it's a standing requirement for any UI work.
- "It looks right at the one size I happened to test" doesn't count as done. Before considering a UI task finished, verify it against the platform's real size range, not a single resolution.

## Web

- **Mobile-first**: base (unprefixed) Tailwind utilities target the smallest supported viewport; larger viewports are layered on top with `sm:`/`md:`/`lg:`/`xl:`/`2xl:` overrides — never the reverse (build for desktop, then cram it into mobile with overrides).
- Minimum verification set: **~375px** (mobile), **~768px** (tablet), **~1280px+** (desktop). A layout that only works at one of these isn't finished.
- Layouts use `flex`/`grid` with relative sizing (`%`, `fr`, `max-w-*`, `min-w-0`) — not fixed pixel widths/heights on containers. Media (`img`, `video`) is always constrained (`max-w-full h-auto` or an aspect-ratio utility) so it can't blow out a narrow viewport.
- See [tailwind-css](../tailwind-css.md) for the utility-level mechanics (breakpoint prefixes, tokens) that implement this.

