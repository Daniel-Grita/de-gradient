# De-Gradient — Status

Last updated 2026-09-02.

*Renamed from **Gritaflux** on 2026-09-02. Local folder still `C:\Code\Gritaflux\`;
localStorage keys still `gritaflux.*` for preset preservation. Design folder
moved from `.design/gritaflux-app/` to `.design/de-gradient-app/`.*

## Where it stands (current)

`index.html` is 158 KB, single file, offline-genuine — three fonts embedded
as base64 (Quantico 400 + 700, Nippo 700, all SIL OFL), zero external
requests verified with the network killed. Six effects (Gradient, Dots,
Chars, Dissolve, Dither, ASCII) in the variant-B two-level nav (mode tabs
+ per-tab `<select>`). Export: PNG at delivery sizes (2048–4K) per ratio,
MP4/webm loop at fixed 1080p equivalents per ratio.

Phase 7 landed 2026-09-01 → 2026-09-02 as batches 1–8 plus a review-gate
pass and a small post-gate correction (hide-not-dim + Random button) and
a simplify pass. Scores at the gate:

| Lens | Score |
| --- | --- |
| UX Heuristics (Krug + Nielsen) | **10/10** |
| Design review (against brief) | Every must-fix closed |
| Norman (Design of Everyday Things) | **10/10** |
| Steve Jobs review | **INSANELY GREAT 9/10** — row 7 (Daniel using it daily) is the one waiting on him |

Full trail in [`REVIEW_GATE.md`](REVIEW_GATE.md), itemised in [`TASKS.md`](TASKS.md).

**Still open — needs Daniel:**

1. Open the current build by eye and confirm.
2. Real Chrome MP4 run — capture a full loop at the fixed 1920×1080 sizes
   and confirm playback is seamless. Bundled Chromium can't produce one.
3. Decide two remaining cuts: 5 charsets → 3? Fold "Wave scale" into presets?

---

## History

### 2026-08-01 — foundation

`index.html` is built and complete — six effects, presets with `localStorage`,
PNG and video export at four aspect ratios, URL importer, responsive layout,
accessibility baseline. Daniel has opened it and confirmed the Gradient effect
looks right.

**Dissolve landed 2026-08-01**, from a reference poster Daniel brought in (blue
over off-white, split across the middle, the boundary broken into scattered
square pixels). New effect `dissolve` in the Generative group: two flat colours,
direction, pixel size, position, spread, falloff, seed, edge wave, and three
motion controls (loop duration, drift, boil). Four presets — Signal, Ember, Hard
Cut, Tideline.

**Variable colour count landed the same day.** Gradient, Dots and Chars take a
`colors` select (3 or 2); Dissolve takes 1 or 2 per side, the second scattered
into the first with a `Mix` amount. New presets: *Duo* (Gradient), *Two Tone*
(Dots, Chars), *Static* (Dissolve). Two supporting fixes came with it, both
worth keeping: `applyPreset` now backfills from `defaults` so a preset saved
before a param existed applies as a complete snapshot, and the URL importer
forces `colors` back to 3, since a shadergradient URL always describes three.

**Inactive controls were then dimmed and disabled** (`activeWhen` on the control,
`syncControlStates()` applying `.is-inert` plus `disabled`). This was a fix, not
a plan: the colour-count work shipped with inactive swatches left live, and
Daniel hit it straight away — Dissolve opens on *Signal* at one colour per side,
so Color A2 and Mix sat there doing nothing with no explanation. Mix also moved
directly under the Colors select, which puts it above the fold.

**Reversed 2026-09-02 to hide, not dim** — presence *is* meaning; if the user
said "2 colors", a third one dimmed reads as broken. `syncControlStates` now
toggles `hidden`. Full reasoning trail in the brief.

It is the first effect added since the "effects are data" claim was made, and it
required no UI code — only the `EFFECTS` entry plus the three wiring points
(`EFFECT_ORDER`, `EFFECT_GROUPS`, `render()`/`exportStill()`). Two things worth
keeping:

- **No `Math.random()`.** The scatter is `hashCell(x, y, seed)`, deterministic
  per cell, or the field would fizz between frames instead of holding still.
- **The loop stays exact without a noise field.** Drift and boil run at 1× the
  loop rate, the edge wave at 1× and 2×, so phase 0 and phase 1 are the same
  image — verified byte-for-byte, see below.

**Variable export aspect landed 2026-08-01**, closing Open Questions #1 in the
brief and risk 0 below. A `1:1 / 16:9 / 4:5 / 9:16` segmented control sits in the
Export footer; the ratio is global, persisted under its own `localStorage` key,
and kept out of presets. Every draw function now takes `w, h` rather than a
single `size`, the frame is sized to the ratio so preview and export agree, and
the shader samples noise in short-edge units so nothing stretches. Details in
`CLAUDE.md`; verified by rendering, see below.

**The theme went light on 2026-08-01**, explored in Paper as two artboards;
Daniel picked *Light A (Instrument Grey)*. What changed in `index.html`:

- The `:root` block is now light neutral grey, with the accent split into
  `--color-accent` `#b0d000` (fills only) and `--color-accent-ink` `#647500`
  (lines, text, focus). Every place the accent was a hairline or text was moved
  to the ink token; the active preset became a fill instead of lime text.
- New `--color-canvas-backdrop` `#fff` — deliberately off the grey ramp, since
  it sits behind the artwork rather than beside it.
- `--radius-sm` 4→3px, `--radius-md` 8→4px.
- The frame stopped being a fixed centred square. This was first done with
  `object-fit:cover` on a still-square canvas, which made the preview a crop of
  the file rather than the file; the aspect-ratio work above then replaced it
  with a frame sized to the chosen ratio. See risk 0.
- The panel was tightened twice (`.panel__body` gap 22→13, `.section` gap 12→8,
  `.ctrl` gap 7→5, `.switch` padding 16→14) to keep Export above the fold. These
  are off-scale one-off values; `.panel__body` still scrolls, so the tightening
  buys headroom rather than replacing the mechanism. Expect to revisit it when a
  sixth effect or a longer section arrives.
- The narrow (≤900px) rule was rewritten — the old `width:min(100%,560px)` square
  no longer applies; the frame takes an explicit 4:3 there, matching roughly the
  shape it takes on a wide screen.

A cleanup pass then wired `.preview`/`.stage` back onto the spacing scale
(`--space-6`, `--space-4`), tokenised the canvas backdrop, and removed four dead
declarations. Comments were added at the two places that cannot be tokenised: the
`.select` arrow data-URI, where `var()` cannot be interpolated into a `url()` and
`%236e6e6e` must track `--color-legend-muted` by hand, and the narrow-layout
`4:3`, so it does not read as an arbitrary guess.

Verified by static analysis only: CSS braces balance, JS parses clean, no
undeclared CSS variables, no hardcoded chrome colours remain outside `:root`,
all artwork hexes untouched.

## Verified vs unverified — read this first

**The app has now been rendered.** Chrome is still absent from this machine, but
Playwright's bundled Chromium is cached at
`%LOCALAPPDATA%\ms-playwright\chromium-1228\chrome-win64\chrome.exe` — note
`chrome-win64`, not `chrome-win` — and driving it directly works. Launch it with
`--use-gl=angle --use-angle=swiftshader --enable-unsafe-swiftshader` so WebGL is
available. That is how everything below was checked, on 2026-08-01.

**The Playwright MCP server cannot do this for you.** It is configured for
channel `chrome` and fails with "Chromium distribution 'chrome' is not found"
before it ever opens a page. Skip it and drive the cached Chromium over CDP
instead: spawn it with `--remote-debugging-port`, read
`http://127.0.0.1:<port>/json/version` for the socket, and talk to it with
Node's built-in `WebSocket` (no `ws` package, browser-style `addEventListener`,
not `.on`). `Target.attachToTarget` with `flatten:true`, then
`Runtime.evaluate` and `Page.captureScreenshot`.

**Verified by rendering:**
- **The light theme renders correctly.** Accent reads as a fill throughout, the
  panel clears 900px, and the Export block sits above the fold with the new
  ratio row in place.
- **All six effects display**, at all four aspect ratios, with no console
  errors. `sampleGradientGrid()` reads back from the hidden WebGL canvas fine.
- **Dissolve's loop is exact, and proved rather than assumed.** Rendering it at
  phase 0 and phase 1 into a 320² buffer and comparing gives **0 differing
  bytes**; the same phase twice also gives 0 (deterministic); phase 0 vs 0.5
  gives 17,955 (it genuinely moves). Loop export is enabled for it and
  `captureStream()` matches the preview buffer.
- **Dissolve costs nothing.** Full-size still export measured 1ms at 2048²,
  2ms at 3840×2160, 3ms at 2048×2560, 4ms at 2160×3840 — two orders of
  magnitude under Dither. It writes one ImageData at cell resolution and
  nearest-neighbour scales it up, the same trick as the Dots block path.
- **The sixth effect did not push Export below the fold.** Measured with
  Dissolve active: Export's bottom edge lands exactly on the viewport bottom,
  and all six switcher labels fit without ellipsis. See the corrected note in
  `CLAUDE.md` — a new *group* is what would cost height, not a new effect.
- **Per-effect param state survives switching** — set a Dissolve seed, went to
  Gradient and back, seed held.
- **The colour-count control degrades exactly, not approximately.** Dissolve at
  2 colours per side with Mix 0 is **byte-identical** to 1 colour per side, and
  Mix 1 is byte-identical to the palette with the second colours swapped in.
- **Dissolve's mix hash is genuinely independent of its scatter hash.** Over
  40,000 cells the joint probability is 0.1776 against 0.1772 for true
  independence — a delta of 0.0003. Sharing the hash would have tinted every
  surviving straggler the same way.
- **Export stays above the fold in all six effects** with the new colour
  controls in place, measured per effect.
- **Old presets backfill.** A synthetic user preset with no `colors` key applies
  as the default rather than inheriting the current value.
- **Inactive controls really are inert.** Mix, Color A2 and Color B2 all report
  `is-inert` and `disabled` at one colour per side and flip back on the select
  change; Color 3 does the same on the gradient family. Focus stays on the
  select after the change. A disabled swatch and a disabled range both ignore a
  programmatic `input` event, and enabled ones still work.
- **Every renderer runs at full export size** at all four ratios — dimensions
  exact, both axes even, timings in risk 1 below.
- **Loop capture works.** `captureStream()` track dimensions match the preview
  buffer exactly, so the recorded file is the shape the frame shows.
- **The narrow layout (≤900px) renders**, frame ratio holding at 430px wide.
- The preview frame and its backing store agree on aspect at every ratio.

**Still unverified:**
- **MP4 specifically.** The bundled Chromium only offers webm, so the MP4 branch
  of `MIME_CANDIDATES` has never actually produced a file. Real Chrome would.
- **A full 6s recording end to end** — capture was proved with a 1.2s stub, not
  by running `startRecording()` through to a download.
- The URL importer has never been run against a real shadergradient.co link.
- Daniel has not yet seen any of this by eye.

## Known risks

0. ~~**The preview canvas now renders more than you can see.**~~ **Resolved
   2026-08-01** by the export-aspect work, which was always the same decision.
   The frame is now sized to the chosen ratio and the backing store matches it,
   so nothing is rendered off the edge and `captureStream()` records exactly the
   visible frame. `object-fit` is `contain` rather than `cover` and now only
   absorbs the sub-pixel rounding from keeping the buffer even-numbered.

1. **Dither/ASCII on large images.** `luminanceGrid()` averages every source
   pixel per cell, and `paintGrid()` then writes every output pixel. This is the
   slowest path in the app and the 16:9 and 9:16 ratios export at 4K, roughly
   double the pixels of the old square. Measured on the bundled Chromium:
   Dither 107ms at 2048², 219ms at 3840×2160; ASCII 51ms / 93ms. Still a
   one-off cost per still, but it is the first thing to feel slow if export
   sizes grow again.
2. **Halftone in Dots** draws one arc per cell at full resolution every frame —
   much heavier than the block path, which scales a tiny ImageData up.
3. ~~**Fonts are CDN-loaded**~~ **Resolved 2026-09-02** — both Quantico weights and
   Nippo 700 embedded as base64 `@font-face` rules. File is 158 KB, verified offline
   with zero external requests.

## Phase 7 landed (2026-09-01 → 2026-09-02)

The full audit → fix → review-gate cycle happened in this window. All eight batches
shipped; the review gate passed all four lenses (ux-heuristics **10/10**, design-review
clean, Norman **10/10**, Jobs **INSANELY GREAT 9/10**). See `REVIEW_GATE.md` and
`TASKS.md` for the itemised trail. Structural changes worth carrying in the head:

- **Panel-nav restructure** — the two-group `.switch` widget is gone. Now a mode tab
  (`Generative / From image`) + a `<select class="fx-select">` for effects within the
  mode. `state.mode` and `state.lastEffectInMode` are independent of `state.effect` and
  remember the last-active effect per mode.
- **Status primitive** — `#statusLine` between `.panel__body` and `.export`, `role="status"`
  / `aria-live="polite"`. `showStatus(text, kind)` / `hideStatus()` / `clearErrorStatus()`.
  Info + success auto-dismiss at 3.2s; error persists until the next action clears it.
  Every previously silent output/failure path now routes through it.
- **Loop output size** — fixed per-ratio 1080p equivalents (`RATIOS[i].lw/lh`), decoupled
  from the still delivery size. `startRecording` swaps the preview buffer to loop dims
  before `captureStream`; `finishRecording` restores; `onResize` short-circuits during
  recording so a window resize can't clobber the buffer.
- **Fonts embedded** — three `@font-face` blocks (Quantico 400 + 700, Nippo 700) using
  base64 data-URIs. Five CDN link/preconnect tags gone.

## Next up

1. **Open it and look.** The full Phase-7 rewrite has been rendered and audited but
   not seen by Daniel. This is the last Jobs-row that isn't code.
2. **Real Chrome MP4 run** — capture a full loop at the new fixed 1080p sizes and
   confirm playback is seamless. Bundled Chromium can't produce one.
3. **Two remaining cuts** — decide: 5 charsets → 3? Fold Wave scale into presets?
4. **Draw the narrow layout in Paper.** Still engineering guesswork rather than a
   design decision.
5. **Tune the gradient.** Strength / Density ranges may want rebalancing.

## Deliberately not doing

3D, user-composed effect chains, GIF export, panning/zooming, anything cloud.
See Out of Scope in the brief.
