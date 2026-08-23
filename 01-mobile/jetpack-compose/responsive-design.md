---
tags: [jetpack-compose, responsive-design]
---

# Responsive Design — Jetpack Compose

## Principle

- Never design or implement a screen/Composable assuming one fixed device size. The realistic range of phone/tablet/foldable sizes is in scope **by default** — unlike i18n (see [[01-mobile/jetpack-compose/jetpack-compose|jetpack-compose.md]]'s "No hardcoded strings or dimensions" section), this isn't a per-project question to ask, it's a standing requirement for any UI work.
- "It looks right at the one size I happened to test" doesn't count as done. Before considering a UI task finished, verify it against the platform's real size range, not a single device.

## Jetpack Compose

- Use Material 3's `WindowSizeClass` (adaptive layouts) and `BoxWithConstraints` to adapt layout across Android's phone/tablet/foldable range — never a fixed `dp` layout that assumes one screen size.
- Respect safe areas/system insets, and don't assume portrait-only unless the project has explicitly scoped out landscape.

## See also

- [[01-mobile/jetpack-compose/jetpack-compose|jetpack-compose]]
