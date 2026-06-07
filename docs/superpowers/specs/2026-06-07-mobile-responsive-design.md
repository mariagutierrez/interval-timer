---
name: mobile-responsive-design
description: Make interval timer responsive for iPhone portrait+landscape, add PWA home screen install, fix countdown overflow and iOS viewport height
metadata:
  type: project
---

# Mobile Responsive Design

## Problem

The app is hosted as a GitHub Page and primarily used on iPhone on a treadmill. Three bugs observed from screenshots:

1. Countdown font (`clamp(4rem, 22vh, 14rem)`) is sized only by viewport height — overflows screen width in both portrait and landscape
2. iOS Safari's dynamic toolbar (address bar + nav bar) reduces available height but `100vh` doesn't account for it, squashing the layout
3. The fullscreen button calls `requestFullscreen()` which iOS Safari does not support — it silently fails

The Wake Lock API is already implemented correctly and works on iOS 16.4+.

## Goals

- Fix countdown overflow in portrait and landscape
- Fix iOS Safari viewport height crush
- Support both portrait and landscape orientations on iPhone
- Install as a PWA from the home screen (launches fullscreen, no browser chrome)
- Hide fullscreen button in contexts where it doesn't work (iOS Safari, standalone PWA)

## PWA Install

Add `manifest.json` at repo root with:
- `name`: "Interval Timer"
- `short_name`: "Interval"
- `display`: `standalone`
- `background_color` / `theme_color`: `#2c2c2c`
- `start_url`: `./`
- `icons`: 192×192 and 512×512 PNG generated from a simple SVG (dark background `#2c2c2c`, white text "IT" or a timer glyph)

Link from `<head>`: `<link rel="manifest" href="manifest.json">`.

No service worker needed — the app has no offline requirement.

**Fullscreen button visibility:**
- Hide via CSS `@media (display-mode: standalone)` — not needed when running as PWA
- Hide on iOS Safari in browser mode by checking at startup: if neither `document.documentElement.requestFullscreen` nor `document.documentElement.webkitRequestFullscreen` exists, hide the button (this is false on iOS Safari, true on desktop and Android Chrome)

## Viewport Height Fix

Replace all `vh`-based layout heights with a `dvh` fallback chain:

```css
height: 100vh; /* fallback */
height: 100dvh; /* overrides on supporting browsers — excludes dynamic toolbar */
```

Apply to `html, body` height declarations.

## Countdown Font Fix

Current: `font-size: clamp(4rem, 22vh, 14rem)`

Problem: at 22vh on a tall phone (844px), font = ~185px. "30:00" at 5 chars × ~0.6 ratio = ~555px wide — overflows 390px screen.

Fix: cap by both axes: `font-size: clamp(3rem, min(20vh, 28vw), 10rem)`

At 390px wide phone: `min(20vh, 28vw)` = `min(~169px, ~109px)` = 109px → "30:00" renders ~330px wide ✓
At 1200px desktop: `min(20vh, 28vw)` = `min(~160px, ~336px)` = 160px → same as before ✓

## Portrait Layout

No structural changes. Existing stack (phase → countdown → rounds viz → tagline → controls) works correctly once font and dvh fixes are applied.

## Landscape Layout

`@media (orientation: landscape)` — split into two columns:

```
┌──────────────────┬─────────────────┐
│   SLOW / FAST    │  [rounds viz]   │
│   02:57          │  5 round cards  │
│                  │  filling height │
│  [RESET] [PAUSE] │                 │
│     [SKIP]       │                 │
└──────────────────┴─────────────────┘
```

- Left column ~55% width: phase + countdown + controls, vertically centred
- Right column ~45% width: rounds viz fills available height
- Tagline ("Japanese Interval Walking") hidden in landscape — space too limited
- Font sizes reduced for landscape (heights are ~350px after safe areas)
- Phase: ~`clamp(1.5rem, 8vh, 4rem)` in landscape
- Countdown: `clamp(2.5rem, min(16vh, 14vw), 7rem)` in landscape
- Controls: tighter padding

## Scope

One file: `index.html`. Add `manifest.json`. No JS logic changes. No service worker.
