# Interval Timer

A simple timer for treadmill interval training:
**3 min slow walk → 3 min fast walk, repeated 5 times** (30 min total).

## Run it

Open `index.html` in any modern browser:

```sh
open index.html
```

Hit **START**. The screen turns green for slow walks and magenta for fast walks. A row of 10 dots tracks progress — the active interval fills like a pie chart, completed ones show a ✓. Audio cues:

- Two high beeps when it's time to **speed up**.
- One low beep when it's time to **slow down**.
- Three ascending beeps when the workout is **done**.

While running, the app requests a screen wake lock (where supported) so the display stays on.

## Install on iPhone

Open in Safari → Share → **Add to Home Screen**. Launches full-screen with no browser chrome.

## Dev / testing

Append `?fast=1` to the URL to use 5-second intervals instead of 3-minute ones, so you can verify all 10 transitions in under a minute:

```sh
open "index.html?fast=1"
```

## Roadmap

- **v0** — single HTML file, browser-only.
- **v1** ✓ — mobile responsive, dot progress indicator, PWA install (Add to Home Screen).
- **M2** — investigate iOS Safari background audio so cues fire while watching videos in another app.
- **M3** — wrap as a native iOS app for reliable background behavior + Live Activity glance.
