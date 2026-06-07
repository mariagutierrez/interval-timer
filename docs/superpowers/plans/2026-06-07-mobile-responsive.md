# Mobile Responsive Design Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fix countdown overflow, iOS Safari viewport crush, and add PWA install support so the interval timer works correctly on iPhone in portrait and landscape.

**Architecture:** Pure CSS + one new JSON file + three icon PNGs. Portrait layout is unchanged. Landscape uses CSS Grid on `<body>` to split timer content (left column) from the rounds visualisation (right column) — no HTML restructuring needed. One small JS change hides the fullscreen button when the Fullscreen API is absent (iOS Safari).

**Tech Stack:** Vanilla HTML/CSS/JS, GitHub Pages (no build step), Python 3 (icon generation, built-in only).

---

### Task 1: Create feature branch

**Files:** none

- [ ] **Step 1: Create and checkout branch**

```bash
git checkout -b responsive-mobile
```

Expected: `Switched to a new branch 'responsive-mobile'`

---

### Task 2: Generate PWA icons

**Files:**
- Create: `icon-192.png`
- Create: `icon-512.png`
- Create: `apple-touch-icon.png`

These are solid `#2c2c2c` squares — matching the app background colour. iOS clips home screen icons to a rounded square automatically.

- [ ] **Step 1: Create icon generator script**

Create `create-icons.py` at repo root:

```python
import struct, zlib

def write_png(width, height, r, g, b, path):
    def chunk(kind, data):
        c = kind + data
        return struct.pack('>I', len(data)) + c + struct.pack('>I', zlib.crc32(c) & 0xffffffff)
    raw = b''.join(b'\x00' + bytes([r, g, b] * width) for _ in range(height))
    data = (
        b'\x89PNG\r\n\x1a\n'
        + chunk(b'IHDR', struct.pack('>IIBBBBB', width, height, 8, 2, 0, 0, 0))
        + chunk(b'IDAT', zlib.compress(raw))
        + chunk(b'IEND', b'')
    )
    with open(path, 'wb') as f:
        f.write(data)

write_png(192, 192, 44, 44, 44, 'icon-192.png')
write_png(512, 512, 44, 44, 44, 'icon-512.png')
write_png(180, 180, 44, 44, 44, 'apple-touch-icon.png')
print('Done.')
```

- [ ] **Step 2: Run it and confirm output**

```bash
python3 create-icons.py && ls -lh icon-*.png apple-touch-icon.png
```

Expected: three PNG files created, each a few KB.

- [ ] **Step 3: Remove the one-shot generator**

```bash
rm create-icons.py
```

- [ ] **Step 4: Commit icons**

```bash
git add icon-192.png icon-512.png apple-touch-icon.png
git commit -m "chore: add PWA home screen icons"
```

---

### Task 3: Create manifest.json and link it

**Files:**
- Create: `manifest.json`
- Modify: `index.html` `<head>` (add two `<link>` tags)

- [ ] **Step 1: Write manifest.json**

Create `manifest.json` at repo root:

```json
{
  "name": "Interval Timer",
  "short_name": "Interval",
  "description": "Japanese Interval Walking — 5 rounds of 3 min slow + 3 min fast",
  "display": "standalone",
  "orientation": "any",
  "background_color": "#2c2c2c",
  "theme_color": "#2c2c2c",
  "start_url": "./",
  "scope": "./",
  "icons": [
    { "src": "icon-192.png", "sizes": "192x192", "type": "image/png", "purpose": "any maskable" },
    { "src": "icon-512.png", "sizes": "512x512", "type": "image/png", "purpose": "any maskable" }
  ]
}
```

- [ ] **Step 2: Add manifest link + apple-touch-icon to index.html**

In `index.html`, find:

```html
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
```

Replace with:

```html
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<link rel="manifest" href="manifest.json">
<link rel="apple-touch-icon" sizes="180x180" href="apple-touch-icon.png">
```

- [ ] **Step 3: Commit**

```bash
git add manifest.json index.html
git commit -m "feat: add PWA manifest for Add to Home Screen install"
```

---

### Task 4: Fix iOS Safari viewport height

**Files:**
- Modify: `index.html` (CSS, `html, body` rule)

The problem: `height: 100%` on `<html>` resolves to `100vh`, which on iOS Safari includes the dynamic address bar. When the bar is visible, the content is taller than the visible area. `100svh` (small viewport height) is fixed to the *minimum* viewport — always excludes the toolbar — so the layout never overflows.

- [ ] **Step 1: Update the height declaration**

In `index.html`, find:

```css
  html, body { height: 100%; width: 100%; overflow: hidden; }
```

Replace with:

```css
  html, body { width: 100%; overflow: hidden; }
  html { height: 100vh; height: 100svh; }
  body { height: 100%; }
```

The duplicate `height` lines are intentional: browsers parse the second value last, so `100svh` wins if supported and `100vh` is the fallback.

- [ ] **Step 2: Verify in browser**

Open `index.html` in Chrome DevTools → device toolbar → iPhone 14 Pro, portrait. Confirm the layout fills the screen with no gap at the bottom and no overflow.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "fix: use 100svh to exclude iOS Safari dynamic toolbar"
```

---

### Task 5: Fix countdown font overflow

**Files:**
- Modify: `index.html` (CSS, `.countdown` rule)

The problem: `clamp(4rem, 22vh, 14rem)` sizes only by height. On a 844px-tall iPhone, `22vh ≈ 185px`. At that size, "30:00" is ~555px wide — wider than a 390px screen. Adding `min(20vh, 28vw)` caps by whichever axis is tighter.

- [ ] **Step 1: Update countdown font-size**

In `index.html`, find:

```css
    font-size: clamp(4rem, 22vh, 14rem);
```

Replace with:

```css
    font-size: clamp(3rem, min(20vh, 28vw), 10rem);
```

At 390px wide: `28vw = 109px` → "30:00" renders ~330px wide ✓  
At 1200px desktop: `28vw = 336px`, `20vh ≈ 160px` → `min` picks 160px, same as before ✓

- [ ] **Step 2: Verify**

Chrome DevTools → iPhone 14 Pro → portrait. Confirm "30:00" fully visible. Rotate to landscape. Confirm it still fits.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "fix: cap countdown font size by vw to prevent horizontal overflow"
```

---

### Task 6: Hide fullscreen button when unavailable

**Files:**
- Modify: `index.html` (one CSS rule, one JS block)

iOS Safari has no Fullscreen API. The button silently does nothing. Fix: hide it via CSS when running as a standalone PWA, and via JS when `requestFullscreen` doesn't exist at all.

- [ ] **Step 1: Add CSS rule for standalone PWA mode**

In `index.html`, find:

```css
  .fs-toggle:active { background: rgba(255, 255, 255, 0.18); }
```

Add immediately after:

```css
  @media (display-mode: standalone) {
    .fs-toggle { display: none; }
  }
```

- [ ] **Step 2: Add JS guard for missing Fullscreen API**

In `index.html`, find:

```js
  fsBtn.addEventListener('click', toggleFullscreen);
  document.addEventListener('fullscreenchange', updateFsBtn);
  document.addEventListener('webkitfullscreenchange', updateFsBtn);
  updateFsBtn();
```

Replace with:

```js
  const fsSupported = !!(document.documentElement.requestFullscreen || document.documentElement.webkitRequestFullscreen);
  if (!fsSupported) {
    fsBtn.classList.add('hidden');
  } else {
    fsBtn.addEventListener('click', toggleFullscreen);
    document.addEventListener('fullscreenchange', updateFsBtn);
    document.addEventListener('webkitfullscreenchange', updateFsBtn);
    updateFsBtn();
  }
```

- [ ] **Step 3: Verify**

Chrome DevTools → iPhone simulation: "Full screen" button should be absent (no Fullscreen API). Desktop Chrome: button should appear.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "fix: hide fullscreen button on iOS Safari and standalone PWA"
```

---

### Task 7: Landscape two-column layout

**Files:**
- Modify: `index.html` (add `@media (orientation: landscape)` block to CSS)

In landscape, `<body>` switches from `display: flex` to `display: grid` with two columns. Left column (55%): phase → countdown/done-card → controls, vertically centred via `1fr auto 1fr` rows. Right column (45%): rounds visualisation spanning all rows, with cards rotated to horizontal (SLOW | FAST side-by-side per row). The tagline is hidden — no room.

No HTML changes needed — grid placement is assigned by CSS using the existing element structure.

- [ ] **Step 1: Add landscape media query**

In `index.html`, find:

```css
</style>
```

Insert the entire block below immediately before `</style>`:

```css
  @media (orientation: landscape) {
    body {
      display: grid;
      grid-template-columns: 55% 45%;
      grid-template-rows: 1fr auto 1fr;
      align-items: center;
      justify-items: center;
      gap: 0;
      padding:
        max(1vh, env(safe-area-inset-top))
        max(1.5vw, env(safe-area-inset-right))
        max(1vh, env(safe-area-inset-bottom))
        max(1.5vw, env(safe-area-inset-left));
    }
    .phase {
      grid-column: 1; grid-row: 1;
      align-self: end;
      padding-bottom: 0.5vh;
      font-size: clamp(1.5rem, 8vh, 4rem);
    }
    .countdown {
      grid-column: 1; grid-row: 2;
      margin: 0;
      font-size: clamp(2rem, min(16vh, 14vw), 7rem);
    }
    .done-card {
      grid-column: 1; grid-row: 2;
      gap: clamp(0.6rem, 2vh, 1.2rem);
      padding: 0;
    }
    .done-title { font-size: clamp(1.5rem, 6vh, 3rem); }
    .done-list  { font-size: clamp(0.8rem, 2.2vh, 1.2rem); }
    .tagline   { display: none; }
    .controls {
      grid-column: 1; grid-row: 3;
      align-self: start;
      margin-top: 0;
      padding-top: 1vh;
    }
    button.primary {
      font-size: clamp(0.9rem, 3.5vh, 1.5rem);
      padding: 0.65rem 1.8rem;
      min-width: 7rem;
    }
    button.secondary {
      font-size: clamp(0.75rem, 2.5vh, 1.1rem);
      padding: 0.6rem 1rem;
    }
    .rounds {
      grid-column: 2; grid-row: 1 / 4;
      align-self: stretch;
      justify-self: stretch;
      flex-direction: column;
      height: auto;
      width: 100%;
      margin:
        max(1.5vh, env(safe-area-inset-top))
        0
        max(1.5vh, env(safe-area-inset-bottom))
        0;
      gap: clamp(3px, 0.8vh, 8px);
    }
    .round-card { flex-direction: row; }
  }
```

- [ ] **Step 2: Verify portrait is unchanged**

Chrome DevTools → iPhone 14 Pro → portrait. Confirm: READY → 30:00 → rounds grid → Japanese Interval Walking → START. Nothing displaced.

- [ ] **Step 3: Verify landscape idle state**

Rotate DevTools to landscape. Confirm:
- Left column: "READY" at top-centre, "30:00" in middle, "START" below
- Right column: 5 rows of SLOW|FAST blocks filling the height
- No tagline
- No overflow

- [ ] **Step 4: Verify landscape running state**

In the browser console, start the timer (click START). Confirm SLOW/FAST + countdown display in left column, round progress fills animate in right column.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: two-column landscape layout for iPhone"
```

---

### Task 8: Push and test on device

- [ ] **Step 1: Push branch to GitHub**

```bash
git push -u origin responsive-mobile
```

- [ ] **Step 2: Serve locally for on-device testing**

```bash
python3 -m http.server 8080
```

On the same WiFi, open `http://<your-mac-local-ip>:8080` in iPhone Safari. Find your Mac's IP with `ipconfig getifaddr en0`.

- [ ] **Step 3: Test portrait on device**

Confirm: countdown fully visible, layout fills screen, START reachable, screen stays on after tapping START.

- [ ] **Step 4: Test landscape on device**

Rotate phone. Confirm two-column layout, no overflow.

- [ ] **Step 5: Add to Home Screen**

Safari → Share → Add to Home Screen. Open from home screen. Confirm: no browser chrome, "Full screen" button absent.

- [ ] **Step 6: Open PR**

```bash
gh pr create --title "feat: mobile responsive layout + PWA install" --body "$(cat <<'EOF'
## Summary
- Fixes countdown font overflow on iPhone (min of vh and vw cap)
- Fixes iOS Safari layout crush via 100svh
- Adds PWA manifest + apple-touch-icon for Add to Home Screen
- Hides fullscreen button on iOS Safari / standalone PWA mode
- Two-column landscape layout: timer left, rounds viz right

## Test plan
- [ ] Portrait iPhone: countdown visible, no overflow, screen stays on
- [ ] Landscape iPhone: two-column layout, no overflow
- [ ] Add to Home Screen: launches standalone, no browser chrome
- [ ] Desktop: layout unchanged
EOF
)"
```
