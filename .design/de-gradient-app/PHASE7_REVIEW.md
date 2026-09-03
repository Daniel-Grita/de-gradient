# De-Gradient — Phase 7 Review (audits 2–5)

Generated 2026-09-01, against `index.html` as built (2932 lines).
Companion to `TASKS.md` (the ux-heuristics audit from the same day). This covers the
other four Phase-7 lenses of `/design-workflow`:

| # | Lens | Verdict |
| --- | --- | --- |
| 2 | `design-review` (brief + aesthetic fidelity) | Strong. 2 must-fix, mostly a11y. |
| 3 | `web-design-guidelines` (Web Interface Guidelines) | ~20 findings, all small; 4 worth doing now. |
| 4 | `design-everyday-things` (Norman) | **4/10** — evaluation and recovery gulfs open. |
| 5 | `steve-jobs-design-review` | **NOT DONE, 6/10** — plywood failure states, nothing cut, offline promise broken. |

`grill-me` (the 6th lens) is interactive and has not been run — see end.

Screenshots captured to `screenshots/`:
`review-desktop-1280`, `review-tablet-768`, `review-mobile-375`, `review-dissolve-desktop`,
`review-dither-empty-desktop`, `review-focus-desktop`, `review-ratio-169-desktop`.
Rendered via CDP-driven Chromium (`--use-angle=swiftshader`), the method in `STATUS.md`.

---

## 2. Design review

Reviewed against `DESIGN_BRIEF.md`. Philosophy: *quiet instrument, light neutral grey,
the canvas is the only loud thing.*

### Summary

The build is faithful to the brief. Instrument-grey system reads exactly as intended:
recessed wells, hairline seams, accent restricted to four places (active effect, active
preset, filled track, Loop button). The gradient dominates the frame; the panel stays
quiet. Export sits above the fold at 1280px with the ratio row in place, as the brief's
caveat required. The biggest gaps are accessibility: no heading structure, no `main`
landmark, and two contrast failures.

### Must fix

1. **No heading hierarchy, no landmarks.** `index.html:496` — the wordmark is a
   `<span>`, every section legend is a `<span>`, there is no `<h1>` and no `<main>`.
   A screen-reader user gets no document outline and can't jump to content.
   _Fix: `<h1 class="wordmark">`, wrap the preview in `<main>`, make `.section__legend`
   / `.switch__glabel` real `<h2>`/`<h3>` (style stays identical)._
2. **Two contrast failures on small text.**
   - `.switch__seg` inactive labels: `--color-legend-muted #6e6e6e` on
     `--color-surface-raised #e4e4e4` ≈ 3.9:1 — fails AA. Visible in
     `review-mobile-375` ("Dots / Chars / Dissolve" are faint). `index.html:208`.
   - `.is-inert{opacity:.4}` (`index.html:263`) on already-muted label text lands
     near 1.8:1. Seen on `review-dissolve-desktop` (Mix / Color A2 / Color B2).
   _Fix: darken the segment idle colour to `--color-text`; replace `.is-inert` opacity
   with a defined dim token. (Both already in `TASKS.md` "Contrast + micro-text pass".)_

### Should fix

1. **`<canvas>` has no accessible name.** `index.html:456-457` — neither canvas carries
   `role="img"` or `aria-label`. _Fix: `role="img"` + `aria-label` on `#artboard` that
   `syncStage()` updates to the active effect name._
2. **Touch targets below 44px** at the ≤900px layout the app supports: transport 26px,
   toggle 30×17, swatch chip 22px, preset rows 30px. `review-mobile-375`. (In `TASKS.md`.)
3. **Focus ring is faint** — `--color-accent-ink #647500` 2px on the grey panel is legible
   but quiet (`review-focus-desktop`). Acceptable per the brief's "quiet chrome", but
   consider 2px offset + a subtle inset companion for range inputs specifically.
4. **`--radius-lg` (12px) and `--space-1` (4px) are declared and never used**
   (`index.html:59,62`). Dead tokens. _Fix: delete, or use `--space-1` where 4px literals
   appear._

### Could improve

- The active preset as a full lime fill (`review-desktop-1280`, "Halo") is the loudest
  thing in the panel — brief-legal (accent = fill) but it pulls the eye more than the
  active *effect* does. Consider matching their weights.
- "+ Save" sits alone on its own row whenever an effect has an even preset count
  (`review-desktop-1280`). Minor rhythm break.

### What works well

- Aesthetic fidelity is high — this genuinely reads as studio equipment, not a creative
  app. The restraint is holding.
- The Dither empty state (`review-dither-empty-desktop`) is hero-quality: dashed zone,
  icon, plain-language title, one primary button.
- Ratio reshaping is direct and immediate (`review-ratio-169-desktop`) — the frame *is*
  the file.
- `prefers-reduced-motion` handled properly (starts paused, not disabled).

---

## 3. Web Interface Guidelines

Grouped `file:line`. Terse. Nothing here is severe; the top four are worth doing with the
`TASKS.md` pass.

### Worth doing now

```
index.html:5   - no <meta name="theme-color"> (add content="#d9d9d9")
index.html:28  - :root missing `color-scheme: light` (fixes native select/color-input/scrollbar rendering)
index.html:10  - <link> to api.fontshare.com has no matching <link rel="preconnect">
index.html:73  - no `touch-action: manipulation` on interactive controls (300ms tap delay on touch)
```

### Accessibility

```
index.html:496       - no <h1>; headings not hierarchical (see Design review must-fix #1)
index.html:456-457   - <canvas> has no role/aria-label (see Design review should-fix #1)
index.html:2121      - import report updates with no aria-live="polite"
index.html:2199      - preset status ("Edited — N values differ") updates with no aria-live
index.html:2270      - recording progress value updates with no aria-live
index.html:2916      - WebGL-failure message just swaps metaDims text; no role="alert", no explanation
index.html:2027      - toggle label is a <span>, not linked to the control — clicking the word does nothing
index.html:508       - #fileInput has no label/aria-hidden
```

### Forms

```
index.html:2110      - import URL input is type="text"; should be type="url" + inputmode="url"
index.html:2112      - placeholder "shadergradient.co URL" — no trailing "…"
index.html:2109      - import URL input: no name, no autocomplete="off", no spellcheck=false
index.html:291,391   - .swatch__hex / .input use :focus not :focus-visible (ring shows on click)
```

### Touch & interaction

```
index.html:73        - no `touch-action: manipulation`
index.html:73        - no `-webkit-tap-highlight-color` set
index.html:218       - .panel__body scroll: no `overscroll-behavior: contain` (scroll chains to page < 900px)
index.html:314       - .toggle__switch has no :hover state
```

### Typography / copy

```
index.html:2467      - "this app doesn't have" — straight apostrophe in UI copy (curly ’)
index.html:2415,2461 - "That URL could not be read." / "Nothing usable found" — problem, no next step
index.html:234       - .ctrl__value: no `font-variant-numeric: tabular-nums` (brief accepts this tradeoff — optional)
```

### Images / performance

```
index.html:2071      - .source__thumb <img> created without width/height attributes (minor CLS)
index.html:496       - "DE-GRADIENT" wordmark + hex fields: no translate="no" (auto-translate can mangle)
```

### Navigation & state

```
index.html (whole)   - no URL/state sync. The app imports a shadergradient URL but can't
                       round-trip its own look to a shareable link. Defensible for a
                       double-click file:// tool; note as a deliberate non-goal or add it.
```

### Passes cleanly

- No `user-scalable=no`, no `transition: all`, no `onPaste` block, no `<div onClick>`,
  no `outline:none` without a replacement, no autoFocus, no animated GIF.
- `prefers-reduced-motion` media query + paused start.
- Long-content truncation is handled on presets, source name, meta file, switch segments.
- `preconnect` present for Google Fonts; `display=swap` on both font URLs.

---

## 4. Design of Everyday Things (Norman)

**Score: 4/10.** Two of the five diagnostic rows pass.

| Row | Pass? | Why |
| --- | --- | --- |
| Discoverability | ~ | Core loop (switch effect, drag slider, click preset, export) is instantly usable. But: hex fields carry no persistent affordance so nobody discovers they're typeable; `.transport__time` "0:00" looks like a draggable scrubber and isn't (false affordance); dimmed controls give no reason; the non-coder owner gets **no hint text** on what Strength vs Density / Falloff / Gamma / Wave scale do — and "every feature reachable from the UI" is the brief's load-bearing constraint. Marked failing. |
| **Evaluation** | ✗ | Canvas updates <0.1s on every drag — excellent. But **Still export gives zero in-app feedback** (file just appears via browser chrome; `toBlob` null → total silence), and `startRecording` / `pickMime` / WebGL-init all fail with bare `return`s. The single most important output action is a black hole. |
| **Error recovery** | ✗ | No undo anywhere. Switching effect / applying a preset / importing a URL discards unsaved edits with no confirm and no undo. `saveCurrentAsPreset` silently overwrites an existing name — including built-ins like "Halo" — unrecoverably (it's in localStorage now). No delete/rename for user presets. |
| Mapping | ✓ | Sliders go up = more; "A on top / bottom / left / right" is explicit; controls sit by what they affect; the frame reshapes to the chosen ratio in real time. |
| Constraints | ✗ | Loop button disables correctly. But the **Still button stays primary-green and clickable on an image effect with no image** (`review-dither-empty-desktop`) — click does nothing. Dimmed controls are disabled but unexplained. |

### Failing paths, concretely

- **Gulf of evaluation — Still export.** Goal "get the PNG" → click Still → *perceive* nothing.
  Fix: `showStatus("Exporting…")` → `showStatus("Saved de-gradient-…​.png")`; handle null blob.
- **Gulf of evaluation — silent failures.** Every bare `return` on a user action needs a
  visible `showStatus(…, "error")`.
- **Error recovery — destructive preset overwrite.** Confirm on name collision; block or
  rename against built-in names; add delete.
- **Error recovery — lost edits.** Confirm "Discard N edited values?" before `applyPreset`
  / `setEffect` when `dirtyKeys().length`.
- **Constraint — dead Still button.** Disable both export buttons when a `needsImage`
  effect has no image; label the reason.

(All five are already in `TASKS.md` — this lens confirms they are not cosmetic; they are
the two open gulfs.)

---

## 5. Steve Jobs design review

```
# Design Review: De-Gradient
Verdict: NOT DONE (score 6/10)
The One Thing: Open a file, drag until the generative visual looks right, take out a
               PNG or a seamless loop — no code, no account, no build step.
Keeps its promise?  Partly. The tuning loop is genuinely insanely-great — live canvas,
                    direct manipulation, no apply button. The *output* half and the
                    *offline* half don't hold.
```

Passes 4 of 7 diagnostic rows: the One Thing is stateable, core value is ≤3 steps
(launch → drag → Still), it was reviewed cold on a real running build, and the demo is
real. Fails:

**Row 5 — nothing was cut.** The `TASKS.md` fix list is 20 items and every one is an
*addition*. A focused cycle kills something.

**Row 6 — the back of the fence is plywood.** The Dither empty state is held to the
hero bar; the failure states are not. Silent Still export, silent capture failure, and a
WebGL dead-end where Gradient/Dots/Chars stay clickable and render blank with a
three-word readout. Error copy states the problem and stops.

**Row 7 — would the team sign it and use it daily?** Not yet:
- **Fonts are CDN-loaded.** The product's whole promise is "double-click the file, no
  dependencies" — and offline it silently drops to system sans. The signature feature
  doesn't survive its own use case.
- **Loop exports at a window-dependent mystery size** (`review-ratio-169-desktop` meta:
  "MP4 912×514" while the still is 3840×2160). A daily user is surprised every time.
- The owner has confirmed Gradient by eye but has **not** seen the light theme, Dissolve,
  or the ratio control by eye (`STATUS.md` "Next up #1").

### Cut list

- One of the two "add an image" affordances — the dropzone button and the Source-section
  button do the same thing. Keep the dropzone; drop `makeSource`'s button, or vice versa.
- `--radius-lg`, `--space-1` — declared, unused.
- Consider: 5 charsets → 3 (Classic / Blocks / Binary covers the range; Minimal and Dots
  are near-duplicates of Classic). "Wave scale" — a control a non-coder will never touch;
  fold into presets.

### Fix list (ranked)

1. **Make Still export speak.** Working state + "Saved …" + null-blob error. This is the
   product's output and it's currently silent.
2. **Surface every silent failure** (capture, mime, WebGL). WebGL failure gets a sentence:
   what happened, what still works.
3. **Embed the fonts as base64.** The offline promise is the product. The woff2 files
   already exist in the portfolio repo (brief, Open Questions #3).
4. **Decide the loop-size story.** Either render the loop to the ratio's real `ew/eh`, or
   state the actual pixels plainly and that the window controls them.
5. **Confirm before discarding edits**; **confirm before overwriting a preset name**.
6. **Disable Still when there's no source image.**
7. **Cut one thing** (see cut list) so this cycle is a focusing cycle, not an accreting one.

### Back of the fence

Empty state: signed. Error/failure states: plywood. Settings: n/a. The `EFFECTS` data
structure itself is beautifully kept — that's the inside of the case, and it's signed.

---

## 6. grill-me — not yet run

`grill-me` is an interactive interview and needs Daniel in the loop. Suggested focus for
that session, given the above:

- Is offline-genuine (embedded fonts) in scope for *this* pass, or deferred again?
- Loop-size: fix the capture, or fix the expectation?
- The non-coder-hint gap: per-section hint text, or is the owner now fluent enough that
  it's not needed?
- What comes *out* of the app on a failure — how loud should De-Gradient be allowed to get,
  given "the canvas is the only loud thing"?

---

## Panel variants (artifact `630b10da`, "De-Gradient Panel Variants", 2026-08-31)

A static mockup exploring one structural change: replace today's **flat switcher**
(two always-visible labelled groups, all six effects on screen) with a **two-level nav** —
a top "Generative / From image" mode tab, then only that mode's effects in a sub-nav.
Screenshots: `panel-variant-A-button-row`, `-B-dropdown`, `-C-pill-row`,
`-variant-A-fromimage-tab`.

### Why this is worth taking seriously

- It removes the always-visible second effect group. `CLAUDE.md` is explicit that a *third*
  `EFFECT_GROUPS` entry is what costs real height (label + row each); this design removes
  even the second, and buys the headroom the panel was hand-tightened twice to find.
- "From image" effects stop advertising themselves when you're doing generative work — the
  dropzone/upload only appears in its own tab.
- The mockups also quietly fix things the audit flagged: `font-variant-numeric: tabular-nums`
  on every value, and the panel gets a full `--r-lg` border + radius instead of a bare
  `border-left`.

### The cost

- **Cross-mode switching becomes two clicks** (mode tab → effect). Today any effect is one
  click. For someone who hops Gradient → Dither → Gradient a lot, that's friction the brief's
  "direct manipulation" principle would notice.
- **Effect state + the animation loop must survive the tab swap** — `state.params[id]` per
  effect already exists, but `syncStage()` / `render()` / the RAF `tick()` all assume the
  active effect is always reachable. Real wiring work, not just markup.
- The mockup is static — preset sets shown ("Classic / Stark / Warm / Ink" for Dither) don't
  match the real `EFFECTS` data; treat the layout as the proposal, not the content.

### The three sub-nav treatments

| | Read | Verdict |
|---|---|---|
| **A — button row** | Identical segmented style to today's switcher, just scoped to the tab. Zero new visual language. | Safest. The mode tab and the effect row look near-identical though (`panel-variant-A`), so it can read as "why two rows of the same thing." |
| **B — dropdown** | Effect collapses to a `<select>`. Saves one row. | Costs discoverability — the other effects vanish behind a click, and a non-coder owner scanning for "what can this do" now has to open it. Fights the brief's "every feature reachable". Weakest for this product. |
| **C — pills** | Small rounded pills, dark-fill active state, sit visually *under* the tab as a clear subordinate. | Best hierarchy — you can see it's a second-level choice, not a repeat of the first. The dark `--color-legend` active fill is a *new* active-state colour though (today active = accent fill everywhere); that's a design-system decision, not a freebie. |

### Decision (2026-09-01)

Daniel picked **B as drawn**, rejecting C's dark pills as off-brand (correct — the app's
active state is accent-fill everywhere; C invents a second active language). A "refined B"
(single grouped `<select>` with `<optgroup>`) was proposed and dropped: it changed the
structure Daniel chose without asking.

B keeps its two-level shape: a segmented control **Generative / From image** on top,
then the current mode's effects in a `<select>` inside that tab. Cross-mode switching is
two clicks (accepted trade-off: "direct manipulation" is about tuning a look, not about
changing effect). The From-image tab always shows the dropzone at the top of its pane when
no image is loaded, regardless of Dither/ASCII selection.

Full task breakdown and the items it reprioritises are in `TASKS.md` §"Panel-nav
restructure — DECIDED". Artifact:
`https://claude.ai/code/artifact/630b10da-dd7f-4775-b772-25e2aaae6cfb`.

---

## Consolidated priority (this review + `TASKS.md`)

The four lenses converge hard on **two themes** already in `TASKS.md`:

1. **The status-message primitive** unblocks the evaluation gulf (Norman row 2), the
   plywood failure states (Jobs row 6), and 4 of the WIG a11y findings. Build it first.
2. **Dirty-state + preset-overwrite guards** close the recovery gulf (Norman row 3).

New from this review, not in `TASKS.md`:

- Heading hierarchy + `<main>` landmark + `<canvas role="img">` (design-review must-fix).
- `color-scheme: light`, `<meta name="theme-color">`, `api.fontshare.com` preconnect,
  `touch-action: manipulation` (WIG, 5-minute fixes).
- Embed fonts as base64 (Jobs fix #3 — promotes `STATUS.md` "Next up #5" to a blocker).
- Cut one feature (Jobs row 5).
- `type="url"` on the import field; curly apostrophe in the ignored-params copy.
