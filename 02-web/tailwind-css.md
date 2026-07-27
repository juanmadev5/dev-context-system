---
tags: [web, css, tailwind]
---

# Tailwind CSS

Used for styling on **every** web project regardless of framework — [[angular]], [[vuejs]], [[react]], [[astro]], [[html-tailwind]] — see [[tech-stack-map]].

## Conventions

- Utility classes in markup are the default. Extract a component/partial when the same utility combination repeats across multiple places (DRY — see [[coding-standards]]), not by writing custom CSS classes to "clean up" the markup.
- Use Tailwind's `theme` config (colors, spacing, font sizes) as the single source of design tokens — never hardcode raw hex colors or dimensions in markup.
- **No arbitrary-value utilities** (`w-[323px]`, `text-[#1a2b3c]`, `mt-[17px]`) as a way to sidestep the theme — if the value you need doesn't exist as a token, add it to `theme.extend` in the Tailwind config first, then use the token. Arbitrary values are only acceptable for a genuinely one-off case (e.g. matching an exact third-party asset's pixel dimensions) — never for anything that could plausibly repeat.
- `@apply` sparingly, only for a handful of truly repeated, non-componentizable utility clusters — prefer extracting a component in the framework layer first.
- Keep the Tailwind config itself DRY: shared design tokens (brand colors, spacing scale) defined once, not duplicated per project unless the project genuinely has its own brand.

## Responsiveness

- Mobile-first ordering: unprefixed utilities = smallest viewport, then `sm:`/`md:`/`lg:`/`xl:`/`2xl:` add overrides for larger ones. See [[responsive-design]] for the full cross-platform principle and the minimum breakpoint set every layout must be verified against.
- No fixed pixel widths/heights on layout containers — use `flex`/`grid` with relative sizing, `max-w-*`/`min-w-0`. Media always constrained (`max-w-full h-auto` or `aspect-*`).

## Color tokens

- Never use Tailwind's raw palette (`blue-500`, `red-600`, ad-hoc hex) directly in markup/components. Every color used in the app goes through a **semantic token** — `primary`, `secondary`, `accent`, `success`, `warning`, `danger`, `background`, `surface`, `text`, `text-muted`, `border`, etc. — referenced by that name (`bg-primary`, `text-danger`), never by the underlying palette value.
- Semantic tokens are defined once as **CSS custom properties** (e.g. in a `:root` block in the project's global stylesheet) and wired into Tailwind's `theme.extend.colors` so utilities resolve to them. This isn't a workaround — it's Tailwind v4's own native approach (`@theme` blocks are CSS variables under the hood), so it's the idiomatic setup, not an extra layer on top.
- Naming is by **role**, not by hue: `primary`/`danger`, never `blue`/`red` — a token named after a color breaks the moment the actual color changes (e.g. rebranding `primary` from blue to purple shouldn't leave a token called `blue` pointing at purple).
- Payoff: rebranding, per-client white-labeling, or adding dark mode becomes a change to the token definitions in one place, never a project-wide find-and-replace across components.

## See also

- [[coding-standards]], [[responsive-design]]
