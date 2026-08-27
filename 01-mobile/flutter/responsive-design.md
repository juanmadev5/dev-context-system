---
tags: [flutter, responsive-design]
---

# Responsive Design — Flutter

## Principle

- Never design or implement a screen/widget assuming one fixed device size. The realistic range of phone/tablet sizes is in scope **by default** — unlike i18n (see [flutter.md](flutter.md)'s "Internationalization" section), this isn't a per-project question to ask, it's a standing requirement for any UI work.
- "It looks right at the one size I happened to test" doesn't count as done. Before considering a UI task finished, verify it against the platform's real size range, not a single device.

## Flutter

- Use `LayoutBuilder` / `MediaQuery` (or a breakpoints abstraction built on top of them) to adapt layout between phone and tablet form factors and to handle orientation changes — never hardcode pixel/logical-pixel dimensions that assume one specific device.
- Respect safe areas/system insets, and don't assume portrait-only unless the project has explicitly scoped out landscape.

