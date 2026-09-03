# Design Brief: De-Gradient

> Supersedes `Thought.md`. Written after a full grilling pass on 2026-07-31.
> All open questions from that document are resolved here.

## Problem

Daniel wants distinctive generative visuals — animated gradients, dithered
halftones — for social posts, video backgrounds, hero sections, and wallpapers.
He can see exactly the look he wants in other people's tools, but can't get it
out on his own terms.

Existing options each fail differently. Shader tools on the web are built for
developers embedding a component, not for someone who wants a finished file.
Design tools don't do live shader work at all. And writing the shader himself
isn't on the table — he's not a coder.

The friction is the gap between *seeing the look* and *having the file*. Every
route to closing that gap currently runs through code.

## Solution

A single double-clickable HTML file that opens to a live, playing canvas and a
panel of sliders. Drag things until it looks right; the canvas responds
immediately. When it does look right, freeze it to a PNG or record a seamless
loop to video.

No project files, no build step, no server, no account. Open it, tune it, take
the output, close it.

## Experience Principles

1. **Direct manipulation over configuration** — Every parameter is a control you
   drag against a canvas that's already running. No apply buttons, no forms, no
   confirmation steps. Numeric values are shown beside each slider so the
   immediacy doesn't cost precision.

2. **The canvas is the only loud thing** — The chrome is deliberately quiet so
   the artwork is the only saturated element on screen. When UI identity and
   artwork legibility conflict, the artwork wins.

3. **What you see is what you export** — The preview renders at the true export
   aspect and scales to fit, rather than filling whatever space the window
   happens to give it. Layout flexes; output stays predictable.

## Aesthetic Direction

> **Settled.** Explored in Paper (file `01KWYYT0M4R02GE4W4S2CA02VM`, three
> artboards: Gradient, Dither, Missing States) and then built.

> **Revised 2026-08-01.** The theme was inverted from near-black to light
> neutral grey. See "Correction: the theme went light" below — the principles
> below still hold, the palette carrying them does not.

- **Philosophy**: Quiet tool. Restrained type, low-contrast labels, no
  decorative chrome. The instrument disappears; the output doesn't.
- **Tone**: Calm, focused, instrument-like. Closer to a piece of studio equipment
  than a consumer creative app.
- **Reference points**: Paper's inspector panel (stacked labeled sections, label +
  slider + numeric value per row); the dithering app screenshot (grouped
  parameters, paired color swatches); shadergradient.co's customizer.
- **Anti-references**: Playful/illustrated consumer creative apps. Gradient-heavy
  SaaS marketing UI. Anything where the chrome competes with the canvas for
  attention. Free-panning infinite canvases — this is a single fixed preview.

### Color note

The UI accent is **`#b0d000`**, one step down from the portfolio's `#cff500`
chartreuse so it seats on a light panel instead of glaring off it. It appears in
only four places — the active effect, the active preset, filled slider tracks,
and the Loop button — which is what keeps the canvas the loudest thing on screen.

**It is a fill colour, not a line colour.** Anything filled with `#b0d000`
carries `--color-on-accent` `#191d1a` text on top (13.6:1 at `#cff500`, 9.6:1 at
`#b0d000`). Where the accent must be a hairline, a focus ring or text — the
Revert link, the source Remove action, input focus — it uses
`--color-accent-ink` `#647500`, the same hue dark enough to clear 4.5:1 against
the panel. This split is forced by physics, not taste: a lime bright enough to
read as lime cannot also be legible as text on light grey.

Both are distinct from **orange `#ff5005`**, which is *artwork*: the dither ink
default and the gradient's `color1`. UI accent and artwork colour are separate
concerns and must never share a token.

## Existing Patterns

De-Gradient descends from Daniel's own design system
(`github.com/Daniel-Grita/daniel-grita-portfolio`, `src/styles/global.css`) but
no longer uses its dark palette. The type, spacing, radius scale and focus rule
are still the portfolio's; the neutrals are De-Gradient's own.

| De-Gradient token | Value | Role |
| --- | --- | --- |
| `--color-ground` | `#d9d9d9` | Workspace surrounding the frame |
| `--color-surface` | `#f2f2f2` | Panel |
| `--color-surface-raised` | `#e4e4e4` | Control wells, segments, secondary buttons |
| `--color-seam` | `#c9c9c9` | Hairline dividers and borders |
| `--color-legend` | `#1a1a1a` | Headings and numeric values |
| `--color-text` | `#3d3d3d` | Control labels |
| `--color-legend-muted` | `#6e6e6e` | Section legends, hints (4.55:1 on panel) |
| `--color-disabled` | `#a3a3a3` | Disabled button text |
| `--color-accent` | `#b0d000` | Fills only |
| `--color-accent-ink` | `#647500` | Lines, text, focus (4.59:1 on panel) |
| `--color-accent-dim` | `#cdcdcd` | Unfilled slider track |
| `--color-on-accent` | `#191d1a` | Text on any accent fill |
| `--color-canvas-backdrop` | `#fff` | Behind the artwork — deliberately off the grey ramp |

- **Typography**: Nippo (display, wordmark only) + Quantico (everything else).
  **Quantico ships only 400 and 700** — the portfolio's `global.css` asks for 500
  and 600 in places, which the browser cannot honour. De-Gradient uses real weights.
- **Numerals**: Quantico, no monospace. Its squared construction gives near-uniform
  digit widths, so values don't jitter while a slider is dragged. The one
  exception is the ASCII canvas, which needs a true monospace — but that is
  *artwork*, not chrome, so it sits outside the system's type rules.
- **Spacing / radius**: portfolio scale (`--radius-sm` 4px, `--radius-md` 8px).
- **Focus**: the portfolio's `:focus-visible` rule (2px accent, 2px offset) is
  reused verbatim.

All values live in one `:root` block in `index.html`. There is one theme; the
`data-theme` attribute on `<html>` reads `light` and exists only as a hook if a
second theme is ever wanted.

## Component Inventory

| Component | Status | Notes |
| --- | --- | --- |
| App shell | Built | Two-pane layout: preview area (~80%) + control panel (~20%) |
| Preview artboard | Built | Square canvas, scales to fit the responsive preview area |
| Effect switcher | Built | Two labelled groups — see Effects below |
| Panel section | Built | Labeled group heading + stacked control rows |
| Slider row | Built | Native `input[type=range]`, so keyboard and focus come free |
| Color swatch | Built | Native `input[type=color]` + hex readout |
| Select | Built | Dither/ASCII pattern, charset, and colour mode |
| Toggle | Built | Grain; play/pause lives in the transport |
| Preset grid | Built | Named presets + save-current, with dirty tracking and Revert |
| Image dropzone | Built | Drag-and-drop or file picker, with an inviting empty state |
| Export controls | Built | Still (PNG) and Loop (MP4/webm), Loop disabled when static |
| URL importer | Built | shadergradient.co URL, reports exactly what it imported |
| Recording indicator | Built | REC badge, accent canvas border, progress + Cancel |

## Effects

Six effects in two families. Effects are declared as data (`EFFECTS` in
`index.html`) — params, sections and presets — and the panel builds itself from
that declaration. Dissolve, added 2026-08-01, was the first effect built after
that claim and needed no UI code, which settles it.

**Generative** — no upload, animated, Loop export available:

| Effect | What it is |
| --- | --- |
| Gradient | WebGL, 4D simplex noise, three colours, grain |
| Dots | The gradient rendered as an ordered-dither or halftone cell field |
| Chars | The gradient rendered through a character ramp |
| Dissolve | Two flat colours meeting at a band that breaks into square pixels |

**From image** — needs an upload, static, Loop export disabled:

| Effect | What it is |
| --- | --- |
| Dither | Ordered 2×2/4×4/8×8, Floyd–Steinberg, or halftone dots |
| ASCII | Five character ramps |

Dots and Chars each offer a **colour mode**: ink-and-paper duotone, or the
gradient's own colours posterised.

### Palettes can be smaller than the effect allows

> **Added 2026-08-01.** Every generative effect now says how many colours it is
> actually using, rather than forcing its maximum.

Three colours is the gradient's ceiling, not its requirement — a two-colour
field is a distinct and often better look, and before this the only way to fake
one was to set two swatches to the same hex, which is a workaround rather than
a control. Gradient, Dots and Chars take a **Colors** select (3 or 2); Dissolve
takes **1 or 2 per side**.

Dissolve's second colour per side is **scattered into the first on the same
pixel grid**, with a Mix amount — not ramped. A ramp would have introduced a
soft gradient into the one effect whose whole character is flat fields and hard
pixels; scattering keeps both fields flat and speaks the same language as the
dissolve band itself. Mix at 0 is byte-identical to one colour per side, so the
control degrades to exactly the old behaviour rather than approximating it.

**Controls that do not apply are hidden.** This has been through two reversals
worth recording because the reasoning trail matters.

First shipped as *left live* — inactive swatches simply present, on the theory
that a row costing nothing beat controls appearing and disappearing. That was
wrong in practice: Dissolve opens on *Signal*, one colour per side, so the
first thing you meet was a Color A2 swatch and a Mix slider that did nothing
and gave no reason. Silence read as breakage.

Second version *dimmed and disabled* the inactive rows, keeping the panel's
fixed shape as the goal, with the **Colors** select directly above as the
explanation. That was better but still wrong. **Reversed 2026-09-02:** if the
user has explicitly said "2 colors", showing a third one dimmed reads as state
they've already ruled out. Presence is meaning — the extra row is now hidden
entirely (`hidden` attribute set by `syncControlStates`). The panel changes
shape when the Colors select changes; that shape change is now the signal that
the choice took effect.

The general mechanism: `activeWhen: { key, value }` on a control declares
which parent value makes it active; anything else hides it. Gradient, Dots and
Chars hide Color 3 at "2 colors"; Dissolve hides Color A2, Color B2 and Mix at
"1 per side". Mix stays directly under the Colors select rather than after the
swatches, so the two controls that answer the same question are grouped
together — this held through both reversals.

**Dissolve is the odd one in the Generative group.** It reads no gradient and
needs no upload — it is a pixel dissolve, the poster device where a flat field
breaks into scattered squares before giving way to a second flat field. It is
grouped as Generative because it animates and needs no source, not because it
shares the gradient pipeline. Its direction (which side Color A sits on), band
position, spread, falloff, edge wave and scatter seed are all controls, so the
same effect covers a hard split, a soft shatter, and a wave-broken tideline.

### The loop is exact

The gradient uses **4D** simplex noise, with two of the four dimensions tracing
a full circle over the loop duration. The field therefore returns exactly to its
starting state — no cross-fade, no amplitude dip, no seam. This is a step past
shadergradient's own 4-sample blend approximation, and it is why loop export can
simply record one cycle and stop.

## Key Interactions

**Tuning a parameter.** Dragging any slider updates the canvas on the next frame
with no intermediate commit step. The numeric value updates as you drag. The
animation keeps playing throughout — you're always tuning a moving image, never
a frozen one.

**Switching effects.** Selecting a different effect swaps the entire control
section beneath the switcher. Each effect holds its own parameter state, so
returning to a previously-used effect restores what you left it at rather than
resetting to defaults.

**Loading a source image.** Dither and ASCII require an uploaded image. Until one
is provided, the canvas shows an empty state that reads as an invitation rather
than an error. Drag-and-drop onto the canvas works as well as the file picker.

**Applying a preset.** Clicking a preset snaps all parameters to that snapshot
and the canvas updates immediately. Saving prompts for a name and adds it to the
grid, persisted to `localStorage`.

**Exporting a still.** One click. The current frame renders at the export
resolution and downloads as PNG. The animation is unaffected — it keeps playing.

**Exporting a loop.** Recording runs for exactly one loop duration, so the
result is seamless by construction rather than by luck. The UI must show that
recording is in progress and roughly how long remains, since this is the only
action in the app that isn't instant.

**Importing a URL.** Pasting a shadergradient.co customize URL pulls in three
colors, speed, strength, and density. Everything else in such a URL describes 3D
parameters this app doesn't have, so the importer must state plainly what it
brought over rather than silently ignoring most of the input.

## Responsive Behavior

Desktop-first — this is a tool used on a real screen, and shader tuning on a
phone isn't a use case worth designing for.

- **Wide (default)**: panel fixed-width on the right, preview takes the rest. The
  square artboard centers in that space and scales to fit.
- **Narrow**: the panel drops below the preview and becomes a full-width scrolling
  column. The artboard still scales to fit the now-shorter preview area.
- The artboard never crops or letterboxes its own content — it always renders the
  full square and scales it. What's visible is always exactly what exports.

## Accessibility Requirements

A personal tool, so this is a sane baseline rather than a compliance target:

- Sliders operable by keyboard (arrow keys to adjust, Home/End for range limits).
- Visible focus indicators throughout, using the accent token.
- Label and value text at ≥4.5:1 contrast against the panel background. Low
  contrast is an aesthetic goal for the chrome, but not at the cost of legibility.
- Never rely on color alone — the active effect and selected preset need a
  non-color indicator too.
- **`prefers-reduced-motion`**: the canvas is inherently animated, so it can't
  simply be disabled. Honor the preference by starting paused rather than
  playing, with an obvious play control. This matters more here than in most
  interfaces.

## Out of Scope

Explicitly not in v1:

- **All 3D.** No geometry types, camera controls, position/rotation, lighting, or
  environment maps. No Three.js dependency. (Env HDR maps would also require
  network fetches, breaking the offline requirement.)
- **User-composed effect chains.** Dots and Chars process the gradient, but that
  pairing is fixed in the effect declaration — there is no UI for stacking
  arbitrary effects onto arbitrary sources. See the correction below.
- **GIF export.** MP4/webm only — GIF is both heavier and lower quality for smooth
  gradient content.
- **Multiple canvases, panning, zooming, or layers.** One fixed preview.
- **Any cloud, account, sharing, or collaboration feature.**
- **Mobile-first design.**
- **A dark theme.** Reversed on 2026-08-01 — the app is light-only now. See the
  correction below.

### Corrections to this brief

Two things this document originally asserted turned out to be wrong once the
design and build existed. They are recorded rather than quietly edited, because
both were decisions taken during grilling.

**"Mutually exclusive modes, no layering" — partly reversed.** Grilling chose
separate modes over a generate→process pipeline. Dots and Chars are exactly that
pipeline, arriving as *named effects* rather than as user-composed layers. That
turned out to be the better shape: it keeps the mental model flat (pick one
effect) while delivering the layered result.

**Correction: the theme went light.** Explored in Paper on 2026-08-01 as two
artboards — *Light A (Instrument Grey)* and *Light B (Paper White)* — and A was
chosen. The reference shifted from "quiet dark tool" to a light neutral panel in
the register of a desktop illustration app: grey workspace, lighter panel,
controls in recessed wells, hairline seams, tighter rows, less corner radius.
Principle 2 ("the canvas is the only loud thing") survives the inversion, but it
now depends on the accent being restricted to fills rather than on the chrome
being dark.

Two knock-on changes came with it:

- **The frame fills the preview area on both axes** rather than being a centred
  square. The canvas backing store is still square; `object-fit:cover` crops it
  to the frame. **The preview is therefore no longer a literal picture of the
  export** — this knowingly relaxes principle 3, and the meta bar under the frame
  carries the true output dimensions instead.
- **The panel needed tightening twice** to keep the Export footer above the fold
  at 900px, because the five-effect switcher in two labelled groups is 64px
  taller than the three-segment row the original artboard drew.

**"Export is a constant pair" — false.** Loop export is meaningless for a static
image, so the Still/Loop pair is conditional on the active effect: Loop is
disabled in Dither and ASCII, and Still is promoted to primary there. Export
state depends on effect state.

## Open Questions

1. ~~**Variable export aspect.**~~ **Built 2026-08-01**, as agreed and drawn:
   a `1:1 / 16:9 / 4:5 / 9:16` control under EXPORT, global rather than
   per-effect, not stored in presets, at 2048², 3840×2160, 2048×2560 and
   2160×3840. All five draw paths, `exportStill` and the capture path now take
   width and height separately. The noise field is aspect-corrected by sampling
   in short-edge units, so a wide frame shows a wider view of the same field
   rather than a stretched one — at 1:1 the output is unchanged. This also
   settled principle 3 above literally: the frame is now sized to the ratio, so
   the preview *is* the export rather than a crop of it.
2. **Loop capture is realtime.** Recording a 6-second loop takes 6 seconds. If
   capture ever becomes faster than realtime, the progress readout should switch
   from seconds to frames or a percentage.
3. **Fonts are CDN-loaded**, so a genuinely offline launch falls back to system
   sans. Embedding the Quantico and Nippo woff2 files as base64 would close this;
   the files already exist in the portfolio repo.
