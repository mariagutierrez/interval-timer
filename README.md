# Interval Timer

A simple timer for treadmill interval training:
**3 min slow walk → 3 min fast walk, repeated 5 times** (30 min total).

## Run it

Open `index.html` in any modern browser:

```sh
open index.html
```

Hit **START**. The screen turns blue for slow walks and magenta for fast walks. Audio cues:

- Two high beeps when it's time to **speed up**.
- One low beep when it's time to **slow down**.
- Three ascending beeps when the workout is **done**.

While running, the app requests a screen wake lock (where supported) so the display stays on.

## Dev / testing

Append `?fast=1` to the URL to use 5-second intervals instead of 3-minute ones, so you can verify all 10 transitions in under a minute:

```sh
open "index.html?fast=1"
```

## Roadmap

- **v0** (this) — single HTML file, browser-only.
- **M1** — PWA: add to iPhone home screen, run full-screen.
- **M2** — investigate iOS Safari background audio so cues fire while watching TikTok / videos in another app.
- **M3** — wrap as a native iOS app for reliable background behavior + Live Activity glance.
