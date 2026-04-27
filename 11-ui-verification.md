# UI Verification Protocol

> **SCOPE: Web/frontend projects ONLY.**
> Active when project has `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.css` files.
> **SKIP ENTIRELY for:** Telegram bots, CLI tools, backend APIs, Python scripts.
> If `CLAUDE.md` defines the project as non-web → this entire file is inactive.

## Trigger

Activate after ANY change to: components, templates, stylesheets, layout files, CSS-in-JS, assets.

## Verification Sequence

After every visual change:

1. **Ensure dev server is running** (port 3000/5173/8080)
2. **Navigate to affected page** — wait for full load
3. **Screenshot desktop** (1440×900) — check layout, spacing, colors, typography
4. **Screenshot mobile** (375×812) — check no horizontal scroll, readable text, 44px touch targets
5. **Screenshot tablet** (768×1024) — check layout reflows correctly
6. **Accessibility snapshot** — missing alt, labels, heading hierarchy, focus order
7. **Console errors** — must be zero before showing to user
8. **Present screenshots + ask:** "ნახე შედეგი — კარგად გამოიყურება?"

## Key Checks Per Viewport

- No horizontal scrollbar
- Text ≥ 14px mobile / ≥ 16px desktop
- Touch targets ≥ 44×44px on mobile
- Images scale proportionally (max-width: 100%)
- No overlapping or overflowing elements

## Before/After Comparison

When modifying existing UI: screenshot before changes, then after, show both.

## Accessibility Minimums

- Every `<img>` has `alt` attribute
- Single `<h1>` per page, no skipped heading levels
- Normal text contrast ≥ 4.5:1, large text ≥ 3:1
- All interactive elements reachable by keyboard Tab
- Modals trap focus; Escape closes them

## Performance Targets

| Metric | Target |
|--------|--------|
| Page load | < 3s |
| LCP | < 2.5s |
| CLS | < 0.1 |

Zero 4xx/5xx on network requests. No broken images or fonts.
