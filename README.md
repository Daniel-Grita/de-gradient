# De-Gradient

*(formerly Gritaflux — renamed 2026-09-02)*


A local, single-file browser app for making generative visuals — animated
gradients, dither fields, ASCII, pixel dissolves — and exporting them as
PNGs or seamless video loops, at a chosen aspect ratio.

**Runs offline. No build step, no server, no npm, no dependencies.**

## Run it

Double-click [`index.html`](index.html). That's it.

## What's inside

- **Six effects**, two families:
  - *Generative* (animated, loop-exportable): Gradient, Dots, Chars, Dissolve
  - *From image* (static, PNG only): Dither, ASCII
- **Presets** — built-in per effect, plus save/delete your own to `localStorage`.
- **Random** — one click rolls every parameter of the current effect into a
  new plausible combination (colours in HSL with clamped saturation/lightness,
  ranges snapped to their step).
- **Export**
  - PNG at delivery sizes — 2048² · 3840×2160 · 2048×2560 · 2160×3840 per
    ratio.
  - MP4 (or webm fallback) loops at 1080p equivalents — 1080² · 1920×1080 ·
    1080×1350 · 1080×1920. Exactly one loop cycle, seamless by construction
    (4D simplex noise, or a whole-number-cycle motion for Dissolve).
- **Import** from a shadergradient.co customize URL — pulls colours, speed,
  strength, density, grain.
- **Two-level nav** — mode tab (Generative / From image) + a `<select>` for
  effects within that mode. Per-mode last-active effect is remembered.

## The load-bearing constraint

The owner is not a coder. Every feature is reachable from the UI. If you
find yourself needing to edit code to use something, that's a bug.

## Repository layout

| Path | What |
| --- | --- |
| [`index.html`](index.html) | The whole app. Authoritative. |
| [`CLAUDE.md`](CLAUDE.md) | Contributor / AI-assistant instructions. Read before changing UX. |
| [`.design/de-gradient-app/DESIGN_BRIEF.md`](.design/de-gradient-app/DESIGN_BRIEF.md) | Why it is the way it is. |
| [`.design/de-gradient-app/STATUS.md`](.design/de-gradient-app/STATUS.md) | Current state, known risks, what's next. |
| [`.design/de-gradient-app/TASKS.md`](.design/de-gradient-app/TASKS.md) | Itemised task history through Phase 7. |
| [`.design/de-gradient-app/REVIEW_GATE.md`](.design/de-gradient-app/REVIEW_GATE.md) | End-of-Phase-7 review report (2026-09-02). |
| `.design/de-gradient-app/screenshots/` | Rendered screenshots across the app's states. |

## Design system

Light neutral instrument grey. **Nippo** for the wordmark, **Quantico** for
everything else (both embedded, both SIL OFL). Accent split into two tokens:
`--color-accent #b0d000` for fills only, `--color-accent-ink #647500` for
lines and text. `#ff5005` orange is *artwork* and never used for chrome.

## Status

Phase 7 complete as of 2026-09-02. UX-heuristics **10/10**, Norman **10/10**,
Steve Jobs review **INSANELY GREAT 9/10**. Last row waiting on Daniel to
open the current build and confirm by eye, plus a real-Chrome MP4 run at
the fixed 1080p sizes (bundled headless Chromium can't produce the file
to inspect). Full details in
[`.design/de-gradient-app/REVIEW_GATE.md`](.design/de-gradient-app/REVIEW_GATE.md).

## Licence

Personal project — no licence declared. Fonts are SIL OFL (Quantico by
Bold Monday, Nippo by Indian Type Foundry).
