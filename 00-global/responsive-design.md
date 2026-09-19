---
tags: [global, responsive-design]
---

# Responsive Design

Applies to every UI stack, web and mobile. The mechanics live elsewhere: [tailwind-css](../02-web/tailwind-css.md) for web, the `## Responsiveness` section of the mobile stack's own note.

## Principle

- Never design or implement a screen/component assuming one fixed viewport or device size. The realistic range of screen sizes for the target platform is in scope **by default** — unlike i18n (covered in the stack's own note), this isn't a per-project question to ask, it's a standing requirement for any UI work.
- "It looks right at the one size I happened to test" doesn't count as done. Before considering a UI task finished, verify it against the platform's real size range, not a single resolution/device.

## Mobile

- Respect safe areas/system insets, and don't assume portrait-only unless the project has explicitly scoped out landscape.
